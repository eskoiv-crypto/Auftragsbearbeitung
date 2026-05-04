# ROADMAP — Prioritized Improvements

What to tackle when bringing this tool to a higher level. Ordered by **leverage per effort**, not by ambition.

---

## Tier 1 — Quick Wins (start here)

### 1.1 Modularize the source (keep single-file deploy)
**Problem:** 2,400-line single HTML is hard to navigate, diff, and reason about.

**Approach:** Split into logical files in `src/`, build to a single deployable HTML.

```
src/
├── index.html          # shell
├── styles.css          # all CSS (extracted from <style>)
├── data.js             # const JTL_DATA = ... (or import from JSON)
├── state.js            # S object + helpers (esc, lg, sfR, ...)
├── tags.js             # tgs() + tag definitions
├── jtl.js              # findJTLText, generateJTLText
├── stages/
│   ├── stage0_erfassung.js    # sub0–sub6
│   ├── stage1_rechnung.js
│   ├── stage2_kommi.js
│   └── stage3_amm.js
├── render.js           # main render() loop
└── main.js             # boot

build.js                # concatenates → dist/elvinci_auftragssteuerung.html
```

A simple Node build script (50 lines, no bundler needed) reads each file, concatenates CSS into `<style>`, JS into `<script>`, embeds the JSON, and writes the single HTML output.

**Effort:** ~1 focused session.  
**Win:** future edits become 10x easier; data lives in `data/jtl_data.json` and gets read at build time → no more keeping HTML and XLSX manually in sync.

### 1.2 Extract design tokens into a CSS theme
**Problem:** Inline `style="..."` is everywhere in template strings. Hard to keep visual consistency.

**Approach:** Convert most inline styles to utility classes (`.mt-12`, `.text-orange`, `.flex-row-gap-10`) or to per-component classes (`.flag-card`, `.tag`, etc.). The tool already uses CSS vars for colors — extend the same approach to spacing, typography, sizing.

**Effort:** ~2 sessions.  
**Win:** can make UI changes (colors, spacing) without touching JS template strings.

### 1.3 Auto-save draft to localStorage
**Problem:** browser refresh = lost work. A long Erfassung means losing 5 minutes.

**Approach:** Subscribe to `S` mutations (or just call `save()` after every `render()`), serialize to `localStorage['elvinci_draft_v1']`, restore on boot if present. Add a small "Draft restored — clear?" banner.

**Edge case:** Multiple browser tabs. Either lock to one tab, or auto-merge on focus. Pick one.

**Effort:** ~half a session.  
**Win:** Dustin and Mirko stop losing work to accidental tab closes.

---

## Tier 2 — Medium Investments

### 2.1 Multi-Auftrag list (Path B from earlier sessions)
**Problem:** tool currently handles one order at a time. Real workflow processes 5–10 per session.

**Approach:**
- Add a sidebar list of saved drafts (localStorage-backed)
- "New order" button creates a fresh `S`
- Click an existing entry → swap into `S`
- Status indicator per draft: in-progress / generated / archived

**Risk:** state-isolation bugs if `S` gets confused between drafts. Add a clear identity field (`S.id = uuid()`) at boot.

**Effort:** 1–2 sessions.  
**Win:** matches the operational reality.

### 2.2 Mobile / narrow-viewport layout
**Problem:** unclear how the tool behaves on a tablet or phone. Dustin sometimes works in the warehouse with a tablet.

**Approach:** Audit on a 768px and 375px viewport. Likely needs:
- Sidebar collapsible on narrow screens
- Larger tap targets for checkboxes
- Stacked grid layouts

**Effort:** 1 session if foundation is good (after 1.2).  
**Win:** tool becomes usable on the warehouse floor.

### 2.3 Keyboard accessibility audit
**Problem:** unverified. Tab order, Enter-to-advance, Escape-to-cancel, Arrow keys in selects — all unknown.

**Approach:** Tab through every stage, document broken nav, fix.

**Effort:** ~half a session.  
**Win:** power users (Mirko especially) gain speed.

---

## Tier 3 — Bigger Bets

### 3.1 Replace Vanilla render() with reactive layer
**Tension:** the global-`S` + `innerHTML` pattern is simple but limits responsiveness. Reactive frameworks (Preact, Solid, Lit) would help but add dependency.

**Approach options:**
- **Stay vanilla, get smarter:** add a tiny pub/sub on `S`, only re-render affected components.
- **Adopt Preact:** ~10 KB, single CDN import, JSX optional. Most of the existing code translates 1:1.
- **Adopt Lit:** Web Components style, more verbose but standards-aligned.

**Effort:** 3–5 sessions for a clean rewrite of render layer.  
**Win:** smooth UX, fewer focus bugs, smaller mental model per change.

**This is a Rule-5 decision.** Don't pick without explicit user approval.

### 3.2 Type checking via JSDoc
Add `/** @typedef */` blocks for `S`, `JTLEntry`, parameter shapes. Run `tsc --noEmit` in CI. Catches param-name typos and field misses without a TS rewrite.

**Effort:** 1 session.  
**Win:** the whole class of "I forgot S has 'jtlCounter' instead of 'counter'" bugs goes away.

### 3.3 Unit tests for findJTLText + generateJTLText
A 50-line test file that simulates every reachable parameter combination, asserts a unique match, asserts the prepend behavior. Prevents regressions from data or code edits.

**Effort:** ~half a session.  
**Win:** the matrix can be edited without fear.

---

## Anti-Patterns (don't do this)

- ❌ **Don't add npm dependencies that require build tools the user can't run.** Anything that needs `npm install` only at build time on the developer's side is fine. But the deployed file must be self-contained.
- ❌ **Don't translate `jtl_text` fields without review.** They are the legal/commercial language going to customers.
- ❌ **Don't change the JTL_DATA shape without verifying matrix sync.** 45 entries, exact structure as defined in ARCHITECTURE.md.
- ❌ **Don't add `oninput="...;render()"` to text inputs.** Use `sfR()` or no refresh.
- ❌ **Don't add localStorage as a primary store and break the "open it from SharePoint" workflow.** localStorage is per-browser-per-machine. The HTML must still work standalone, with localStorage as enhancement.

---

## Verification (run after any data change)

```python
# Save as verify.py and run with: python3 verify.py
import re, json
from openpyxl import load_workbook

with open('elvinci_auftragssteuerung.html') as f: html = f.read()
m = re.search(r'const JTL_DATA = (\[.*?\n\]);\s*\n', html, re.DOTALL)
data = json.loads(m.group(1))
assert len(data) == 45, f"Expected 45, got {len(data)}"
assert all('counter' not in e for e in data), "counter field present (should not be)"

wb = load_workbook("data/JTL_Rechnungstexte.xlsx", data_only=True)
# ... key extraction, see verify.py in handover for full version
print(f"✓ {len(data)} entries, no counter, parses cleanly")
```

A full version of this script (with HTML↔XLSX field-level diff) lives in the project history. Re-derive it from `ARCHITECTURE.md` if you need to verify after a data edit.

---

## Suggested First Session

**Pick one of:**

**Option A — Architecture First:** do 1.1 (modularize source). Sets foundation for everything else.

**Option B — User-Facing First:** do 1.3 (auto-save) + 2.1 (multi-Auftrag list). Immediate operational win, no big refactor.

**Option C — Polish First:** do 1.2 (CSS theme) + 2.3 (keyboard audit). Lowest risk, most visible improvement.

**Recommendation: A.** Modular structure unblocks everything else. The data extraction alone removes the recurring HTML↔XLSX sync chore that has eaten time across multiple sessions.

If you disagree, pick another and run with it. But pick **one**, finish it, then move to the next. Don't fan out across all three.
