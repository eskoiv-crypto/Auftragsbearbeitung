# Architecture Notes

Internal design of `elvinci_auftragssteuerung.html` (current = v4).

---

## File Layout (currently monolithic)

```
elvinci_auftragssteuerung.html         (~278 KB)
├── <head>
│   └── <style> ... ~600 lines of CSS, all design tokens as CSS vars
├── <body>
│   └── <div id="root"></div>
└── <script>
    ├── const JTL_DATA = [...]              ← 45-entry array, ~165 KB
    ├── const S = {...}                     ← single global state object
    ├── helpers (esc, fmtD, isFix, isSp, isZoll, lg, sfR)
    ├── tag system (tgs)
    ├── stage sub-functions (sub0..sub6)
    ├── JTL Generator (findJTLText, generateJTLText)
    ├── render() — innerHTML regeneration
    ├── ticket / PDF export
    └── boot
```

The script section is ~1,800 lines (excluding the 1,300-line JSON data).

---

## State

A single global `S` object holds **everything**. Direct mutation, no immutability, no event system.

Key fields (excerpt):
```js
{
    // Order metadata
    au: '',                    // AU number(s)
    kunde: '',                 // customer name
    
    // Versand & freight
    versandart: '',            // 'spedition' or 'selbstabholung'
    frachtkosten: '',
    
    // Payment
    za: '',                    // 'zahlungsziel' / 'vorkasse' / 'barzahler' / 'zoll'
    re: '',                    // invoice number
    
    // Order-level flags (show as tags in header)
    isZoll: false, zielland: '',
    isLoose: false, looseInfo: '',
    isKlein: false,
    isSonder: false, sonderText: '',
    isKombi: false,            // auto-detected from AU comma/+
    isSonderverein: false,     // NEW in v4 — Sondervereinbarung mit Kunden
    
    // Termine
    fixDatum: '',              // Fixtermin (Lieferung)
    fixOverride: false,        // override for tight buffer
    tpKommiDatum: '',          // Kommissionierungsdatum
    termin: '',                // Terminwünsche freitext
    besonderes: '',            // Besondere Vereinbarungen freitext (internal note)
    
    // JTL Generator state
    jtlZZ: '7',                // Zahlungsziel days (counter REMOVED in v3)
    jtlHasRamp: false,
    jtlHasAppointment: false,
    jtlAppointmentStatus: 'erfuellbar',
    jtlShowGenerator: false,
    jtlResult: null,           // matched template + computed prepends
    jtlTab: 0,                 // which tab to show (todos_vor/jtl/todos_nach/dashboard)
    
    // Stage progression
    sub: 0,                    // Erfassung sub-stage 0..6
    auditLog: [],              // chronological list of events
}
```

---

## Tag System (`tgs`)

Returns `[{x:'TAG-LABEL', c:'css-class'}, ...]` based on current `S` flags. Rendered as colored tags in the header. Currently 8+ tags including ZOLL, LOOSE, KLEIN, SONDERFALL, KOMBI, SONDERVEREINBARUNG, OVERRIDE.

To add a new tag:
1. Add the flag to `S` initializer
2. Add a line in `tgs()` returning the tag
3. Add a CSS class `.t-yourTag` if you want a unique color

---

## findJTLText — The Matching Logic

The tool's most critical function. Given a parameter object, it must return exactly one match from the 45-entry `JTL_DATA` matrix.

```js
function findJTLText(params) {
    const heberStr = params.hebebuehne ? 'mit' : 'ohne';
    const zzStr = params.zahlungsziel != null ? String(params.zahlungsziel) : null;
    return JTL_DATA.find(s => {
        if (s.versandart !== params.versandart) return false;
        if (s.terminwunsch !== params.terminwunsch) return false;
        if (params.terminwunsch === 'mit' && s.terminwunsch_status !== params.terminwunsch_status) return false;
        if (params.versandart === 'Lieferung' && s.hebebuehne !== heberStr) return false;
        if (s.zahlungsart !== params.zahlungsart) return false;
        if (params.zahlungsart === 'ZAHLUNGSZIEL' && String(s.zahlungsziel) !== zzStr) return false;
        return true;
    }) || null;
}
```

**Invariant:** Every reachable parameter combination must return exactly one match. Verified by simulation in the verification scripts.

**Hardened in v2:** explicit string comparisons (was: boolean coercion, fragile). 

**Simplified in v3:** counter check removed (counter dimension eliminated).

---

## generateJTLText — Prepend Pipeline

Calls `findJTLText`, then conditionally prepends customer-facing clauses to `jtl_text`:

```js
function generateJTLText() {
    const params = {/* derived from S */};
    S.jtlResult = findJTLText(params);
    
    // KOMBI: factual scope statement
    if (S.isKombi && S.jtlResult?.jtl_text) {
        S.jtlResult = { 
            ...S.jtlResult, 
            jtl_text: 'We have merged orders [KOMBI_AU_NUMMERN] in this invoice.\n\n' 
                      + S.jtlResult.jtl_text 
        };
    }
    
    // SONDERVEREINBARUNG: legal/contractual clause (added in v4)
    if (S.isSonderverein && S.jtlResult?.jtl_text) {
        S.jtlResult = { 
            ...S.jtlResult, 
            jtl_text: 'This invoice is a special agreement that explicitly pertains to this transaction. All regular agreements remain unaffected.\n\n' 
                      + S.jtlResult.jtl_text 
        };
    }
    
    S.jtlTab = 0;
    render();
}
```

**Rendering order in final text** (when both flags active): `Sondervereinbarung → KOMBI → body`. Sondervereinbarung is closest to the top because it's prepended last. Justified: customer reads the legal framing first.

To add another conditional clause: insert another `if (S.flag) { ... prepend ... }` block. Order matters — last to prepend ends up first.

---

## Rendering

`render()` calls every sub-function and concatenates HTML strings, then assigns to `root.innerHTML`. No virtual DOM, no diffing, no partial updates.

**Pros:** simple, no framework dependency, full control.

**Cons:** every keystroke on most inputs triggers full UI regeneration → focus loss. Mitigated by `sfR()` (soft refresh, 200 ms debounce + selectionRange preservation) for text inputs that affect tags or downstream UI.

**Anti-pattern to avoid:** adding `oninput="...;render()"` to text inputs. Use `sfR()` or no refresh at all.

---

## Audit Log (`lg`)

`lg(label, detail)` pushes `{ts, label, detail}` to `S.auditLog`. Rendered chronologically in the sidebar. Entries are added at sub-stage confirmations (`confirmSub`).

This is currently in-memory only. Lost on reload.

---

## CSS

All design tokens declared as CSS variables on `:root`:
- Colors: `--bg`, `--bg2`, `--fg`, `--fg2`, `--bd`, `--ok`, `--red`, `--orange`, `--blue`, `--purple`, etc.
- Typography: monospace font for AU/RE numbers via `.mono` class
- Spacing: 8/12/16 px grid roughly

Tags use class prefix `t-` (`.t-zoll`, `.t-loose`, `.t-klein`, `.t-sonder`, `.t-kombi`, `.t-p1`, `.t-p2`, `.t-ad`, `.t-ok`).

**Inline styles are heavy.** Many `style="..."` attributes embedded in template strings. Refactor opportunity (see ROADMAP).

---

## Dependencies

| Lib | Purpose | Loaded via |
|---|---|---|
| Chart.js | Charts (currently unused but loaded — leftover) | CDN |
| jsPDF | PDF export of audit ticket | CDN |
| (none other) | — | — |

No bundler, no transpiler. Plain ES modules-style code without `<script type="module">`.

---

## Cross-Cutting Concerns

### Data integrity (HTML ↔ XLSX)

The XLSX is the source-of-truth for invoice templates. The HTML embeds `JTL_DATA` as a parsed JSON array. Any change to one must be propagated to the other.

Verification script (Python, requires `openpyxl`):
```python
# Check sync — see ROADMAP.md § Verification
```

### Counter is gone

Past artifacts may reference `counter`, `jtlCounter`, "If you pay X more invoices on time...". These were removed in v3 (2026-05-04). If you find any residual: bug.

### Internal vs customer-facing text

- `todos_vor`, `todos_nach`, `dashboard_text` → **internal** (German), shown to staff
- `jtl_text` → **external** (English), goes to the customer's invoice

Don't accidentally show internal text to a customer. Don't translate `jtl_text` without explicit review.
