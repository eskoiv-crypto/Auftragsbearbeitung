# elvinci JTL Auftragssteuerung

Internal tool for elvinci.de GmbH that walks an order through its full processing lifecycle:
**Erfassung → Rechnung → Kommissionierung → AMM-Übergabe.**

Generates customer-facing JTL invoice texts from a 45-entry template matrix based on order parameters (versandart, hebebühne, zahlungsart, zahlungsziel, terminwunsch).

---

## Quick Start

```
elvinci_auftragssteuerung.html   ← double-click to open in any modern browser
```

That's it. No build, no install. Pure self-contained HTML + JS + Chart.js + jsPDF (CDN).

---

## File Inventory

| File | Purpose |
|---|---|
| `elvinci_auftragssteuerung.html` | The tool (current = v4, 2026-05-04) |
| `data/JTL_Rechnungstexte.xlsx` | Source-of-truth for all 45 invoice templates |
| `data/jtl_data.json` | Same data as JSON (extracted, in sync) |
| `CLAUDE.md` | Project rules for any AI assistant |
| `ARCHITECTURE.md` | How the tool is built internally |
| `ROADMAP.md` | What to improve next, with priority |

---

## Tool at a Glance

### The Workflow (left sidebar = stage progression)

1. **Stage 0 — Erfassung (Verkauf)** — Order entry: AU number, customer, freight, content, flags (Zoll/Loose/Kleingeräte/Sonderfall/Sondervereinbarung), termine. 7 sub-stages.
2. **Stage 1 — Rechnung (Mirko)** — JTL Generator: select versandart/hebebühne/terminwunsch_status/zahlungsziel, system finds matching template, prepends KOMBI/Sondervereinbarung clauses, outputs todos+text.
3. **Stage 2 — Kommissionierung (Dustin)** — Picking + packaging coordination.
4. **Stage 3 — AMM-Übergabe (Dustin)** — Handoff to AMM spedition.

### Core State Object `S`

A single global object holding everything: AU numbers, customer, freight, flags, JTL Generator selections, audit log. Mutated directly by handlers. Re-render via `render()`.

### Render Pattern

`render()` regenerates the entire UI via `innerHTML`. Soft refresh `sfR()` exists for inputs that need debounced re-render with focus preservation (200 ms + selectionRange restore).

### JTL Template Matching

`findJTLText({versandart, terminwunsch, terminwunsch_status, hebebuehne, zahlungsart, zahlungsziel})` searches the 45-entry `JTL_DATA` array for an exact match. Returns the matching `{todos_vor, jtl_text, todos_nach, dashboard_text}` or `null`.

### Two Customer-Facing Prepend Clauses

Both order-level flags trigger conditional prepends to the matched `jtl_text`:

- `S.isKombi` → `"We have merged orders [KOMBI_AU_NUMMERN] in this invoice.\n\n" + body`
- `S.isSonderverein` → `"This invoice is a special agreement that explicitly pertains to this transaction. All regular agreements remain unaffected.\n\n" + body`

When both are set, Sondervereinbarung ends up at the very top (it prepends after KOMBI), giving customers the legal framing first, scope second, content third.

---

## Version History (recent)

| Date | Version | Changes |
|---|---|---|
| 2026-05-04 | **v4** | Sondervereinbarungsklausel: order-level flag + tag + JTL-prepend, mirrors KOMBI architecture |
| 2026-05-04 | v3 | Counter dimension eliminated. JTL_DATA: 99→45 entries. Counter dropdown removed. Customers can no longer rise in ZZ — only fall on late payment. |
| 2026-05-04 | v2 | XLSX typo sweep (KOMISSIONIERUNG, bedeuted, AUFTRTAG, 21Tage); HTML pretty-print of JTL_DATA; findJTLText hardened (string compare for hebebuehne, String() coerce for counter) |
| ~2026-04 | v1 | Origin: previous monolithic HTML built across many sessions |

See `ARCHITECTURE.md` for a deeper look at design decisions.

---

## Testing the Tool

Open the HTML file. Then:

**Smoke test (full happy path):**
1. AU: enter `AU2026020513372`, customer: `Test GmbH`
2. Zahlungsart: ZAHLUNGSZIEL, ZZ: 14 Tage
3. Versandart: Lieferung, Hebebühne: ja, Termin: Erfüllbar
4. Click through Erfassung sub-stages
5. Stage 1 → JTL Generator → "Passenden Text finden"
6. Verify: matched text appears, todos shown, no console errors

**Sondervereinbarung test:**
1. In Stage 0 → sub 4 (Merkmale): tick "SONDERVEREINBARUNG"
2. Run JTL Generator
3. Verify text starts with `"This invoice is a special agreement..."`

**KOMBI test:**
1. AU field: `AU2026020513372, AU2026020513373` (comma-separated)
2. Verify auto-detected KOMBI tag in header
3. Run Generator → verify KOMBI line in output

**Combined test:**
1. KOMBI + Sondervereinbarung both set
2. Verify text order: Sondervereinbarung first, KOMBI second, body third

---

## Working with This Tool in Claude Code

- Read `CLAUDE.md` first.
- For any data change touching `JTL_DATA` or the XLSX: run sync verification (see `ROADMAP.md` § Verification).
- Single-file deployment is non-negotiable. If you split into modules, build a bundler step that produces a single deployable HTML.
- The German-language UI labels are the customer-facing API. Don't translate without ask.
- Customer-facing English text in `jtl_text` fields is the legal-effect API. Don't change without explicit content review.
