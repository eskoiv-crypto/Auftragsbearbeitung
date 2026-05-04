# ROADMAP v2 — Auftragsbearbeitung 100%

**Ziel:** Tool, das die Arbeitsplatzbeschreibung *Auftragsbearbeitung v1.1* (Janna, 04.05.2026) **vollständig** abbildet — keine Phase, keine Pflicht, kein Eskalationspfad fehlt. „100 % funktionierend" heißt: jede Pflicht aus der Stellenbeschreibung hat ihren Platz im Tool, jede Übergabe ist ein Klick, jede Eskalation ein Knopf.

**Datenstand:** 2026-05-04 — basierend auf Handover v4 + `2026-05-04_Arbeitsplatzbeschreibung_Auftragsbearbeitung_v1.1.docx`.

**Diese Roadmap ersetzt nicht** `ROADMAP.md` (= technisches Refactoring-Backlog). Sie ist die **fachliche** Roadmap. Tech-Items aus dem alten Backlog werden hier wo relevant in Tiers eingehängt.

---

## Reality-Check — was das aktuelle Tool kann vs. was die Stelle verlangt

| Phase laut Stellenbeschreibung | Tool aktuell (v4) | Coverage |
|---|---|---|
| 1 — Saleskärtchen & Auftragseingang | Stage 0 „Erfassung" | ~70 % (Saleskärtchen-Konzept fehlt explizit) |
| 2 — Fulfilment Pipeline pflegen (Excel B/C/D/G/L/M/N/E/H/I/N/Q) | **fehlt komplett** | **0 %** |
| 3 — Kundenkommunikation (WhatsApp/Email/Anruf) | nicht modelliert | **0 %** |
| 4 — Rechnungsstellung (JTL-Abgleich, Anmerkungsfeld, PDF an Teamskärtchen) | Stage 1 „Rechnung" — aber nur Textgenerator, kein Abgleich, kein Anhangs-Zwang | ~40 % |
| 5 — Anmeldung Spedition / Zoll (AMM-Mail, Zollagent-Paket) | Stage 3 „AMM-Übergabe" — Stub | ~20 % |
| 6 — Nachverfolgung & Dokumentation (Reporting, SharePoint) | Audit-Log nur In-Memory, kein Wochenreport | ~15 % |
| Kapitel 7 — Tägl./wöchentl. Pflichten (2× Sync mit Dima, Wochenreport, SharePoint-Aufräumen) | nicht abgebildet | **0 %** |
| Kapitel 8 — Eskalationspfade | nicht abgebildet | **0 %** |

**Aggregat: ~20 % der Stellenbeschreibung im Tool. 80 % fehlt.**

---

## Strategische Leitplanken (Rule-5-Entscheidungen)

Die Roadmap setzt diese Voraussetzungen voraus. Bevor wir Tier 1 starten, **muss** die User-Bestätigung zu jeder Leitplanke da sein.

| # | Leitplanke | Default-Annahme | User-Bestätigung nötig? |
|---|---|---|---|
| L1 | Single-File-HTML bleibt Hard-Constraint (Deploy ohne Build) | ja | ja — sonst Build-Step + Modular-Code |
| L2 | Persistenz via `localStorage` (kein Backend, kein OAuth) | ja | ja — wenn SharePoint-Live-Sync gewünscht, ist Backend Pflicht |
| L3 | Externe Systeme via `mailto:`/`tel:`/`whatsapp://`/SharePoint-URL — kein API-Call | ja | ja |
| L4 | Phase-Anzahl im Tool: **6** (statt aktuell 4) | ja | ja — sonst bleibt Phase 2/3/6 unsichtbar |
| L5 | Owner Phase 4 (Rechnung) = **Janna** (laut Stellenbeschreibung), NICHT Mirko (laut Handover) | ja | ja — Diskrepanz im Handover, muss aufgelöst werden |
| L6 | Tool ist Single-User pro Browser/Tablet (Janna), keine Multi-User-Konkurrenz | ja | ja |

---

## Tier 0 — Klärung (kein Code, ~30 min)

**Ziel:** Open Questions schließen, bevor wir bauen. Ohne diese Antworten ist jeder Code Schätzung.

| # | Frage | Warum |
|---|---|---|
| 0.1 | Mirko vs. Janna — wer stellt Rechnungen? | Phase-4-UI-Owner |
| 0.2 | Saleskärtchen + Teamskärtchen — welches System? Trello, MS Planner, eigenes? URL-Schema? | Damit Tool Deeplinks bauen kann |
| 0.3 | Fulfilment Pipeline — SharePoint-URL, Spalten B/C/D/G/L/M/N/E/H/I/N/Q exakt: was steht in jeder Spalte? | Für Pipeline-Form im Tool |
| 0.4 | AMM-Anmeldemail — gibt es eine bestehende Vorlage? Stefan & Björn beide cc, oder nur einer? | Für Mail-Template |
| 0.5 | Zollagent — Email-Adresse, Pflichtfelder im Datenpaket | Für Zoll-Paket-Generator |
| 0.6 | Wochenreport an Dustin — Format (PDF/Email/Word/Karte)? Inhalt-Template? | Für Report-Generator |
| 0.7 | Tagescheckliste — soll Tool sie pro Browser-Session zeigen, oder als persistente "Today"-View? | Für Daily-Modus |
| 0.8 | Eskalation an Dustin/Siyad — bevorzugter Kanal: Email, WhatsApp, Telefon, Karte? | Für Eskalations-Buttons |
| 0.9 | Bleibt Single-File-Constraint? Oder Build-Step erlaubt? (L1-Entscheidung) | Architektur-Weiche |
| 0.10 | Tooling installierbar? (Python+openpyxl ODER Node) | Sonst Verify-Pipeline browser-portieren (Tier 6) |

**Output:** beantwortete Fragen werden zu `OPEN_QUESTIONS.md` im Repo, dann nach jeder Klärung gestrichen.

---

## Tier 1 — Operatives Fundament (2 Sessions)

**Ohne Tier 1 ist „geführtes Arbeiten" nur Theorie.** Janna verliert jede Unterbrechung Arbeit.

### 1.1 Auto-Save in localStorage
- `S` nach jedem `confirmSub`/`render()` serialisieren → `localStorage['elvinci_S_v1']`.
- Boot: bei vorhandenem Draft → "Stand vom HH:MM wiederherstellen?"-Banner.
- Edge: mehrere Tabs → Lock auf erstem Tab, Warnhinweis im zweiten.
- **Akzeptanz:** Browser-Refresh → 100 % State zurück, inkl. AuditLog.
- **Aufwand:** ½ Session.

### 1.2 Multi-Order-Liste
- Drafts in `localStorage['elvinci_drafts_v1']` (Array von `{id, kunde, au, status, lastUpdate}`).
- Sidebar-Liste zeigt alle laufenden Aufträge mit Status-Tag (Erfassung/Rechnung/Kommi/AMM/abgeschlossen).
- Klick auf Eintrag → Swap in `S`. Aktueller Eintrag bekommt automatisch eine `id = uuid()` beim Anlegen.
- "Neuer Auftrag"-Button reset'd `S`.
- **Akzeptanz:** 5 parallele Aufträge, frei zwischen ihnen wechseln, jeder behält seinen Stand.
- **Aufwand:** 1 Session.

### 1.3 Phase-Übergabe-Modell
- Pro Phase ein **Lock**-Status (`offen` / `in_arbeit` / `abgeschlossen`).
- Beim Übergang: Phase wird abgeschlossen → nächste freigegeben. Snapshot der Daten ins Audit-Log mit Timestamp + (vorerst) hard-coded Verantwortlichen.
- Read-only-Anzeige aller bereits abgeschlossenen Phasen mit Diff-Hinweis bei späterer Korrektur.
- **Akzeptanz:** Janna kann Auftrag in „Erfassung abgeschlossen" Status setzen; bei Wiederaufruf ist Phase 1 read-only, Phase 2 aktiv.
- **Aufwand:** ½ Session.

---

## Tier 2 — Phase-Coverage 1:1 zur Stellenbeschreibung (3 Sessions)

**Ziel:** Tool hat **6 Phasen** (statt aktuell 4), 1:1 zur Stellenbeschreibung benannt.

### 2.1 Phase-Schema umbauen
Sidebar-Stages neu:
1. **Saleskärtchen & Auftragseingang** (war: Erfassung)
2. **Fulfilment Pipeline** (NEU)
3. **Kundenkommunikation** (NEU)
4. **Rechnungsstellung** (war: Rechnung)
5. **Anmeldung Spedition / Zoll** (war: AMM-Übergabe)
6. **Nachverfolgung & Dokumentation** (NEU als first-class)

### 2.2 Phase 1 — Saleskärtchen-Vollständigkeitscheck
- Pflichtfelder als Checklist: Kundendaten, Mengen, Preise, Versandart, Zahlungsziel.
- Bei Lücken: roter Block mit Button **„Saleskärtchen an Verkauf zurück"** → erzeugt Outlook-Mail mit fehlenden Feldern.
- "Bearbeitung beginnen"-Button **disabled**, solange ein Pflichtfeld leer ist (Maxime: 100 % Vollständigkeit).
- **Akzeptanz:** unvollständiges Kärtchen kann nicht zu Phase 2 fortschreiten.

### 2.3 Phase 4 — Rechnungsstellung anreichern
- Schritte 4.1–4.5 explizit als Sub-Stages:
  - 4.1 Auftragsnummer als einzige ID erzwingen (Tool zeigt Warnung wenn AU-Feld leer).
  - 4.2 JTL-Abgleich-Form: Mengen, Versandart, Zahlungsziel — Janna haket ab, Diskrepanz-Feld dokumentiert.
  - 4.3 Rechnungsdetails-Form: Land/MwSt./Zoll/Versand-vs-Abholung.
  - 4.3.1 Rechnungstext aus Vorlage (= bestehender JTL-Generator) — ✓ vorhanden.
  - 4.4 Speichern-Button → Schritt-Marker im Audit.
  - 4.5 Rechnungs-PDF-Anhang **Pflicht-Upload** (sonst Phase 4 nicht abschließbar) → Drag-and-Drop in Browser, base64 in `S`, in localStorage. Plus Hinweis "PDF auch ins Teamskärtchen anhängen" mit Deeplink (sobald 0.2 geklärt).
- **Akzeptanz:** Phase 4 nicht abschließbar ohne PDF-Anhang.

### 2.4 Phase 6 — Doku-Klammer
- "Erreichbarkeit"-Banner: WhatsApp / Email / Telefon-Status sichtbar.
- "Wochenreport-Eintrag erzeugen"-Button (Inhalt für Tier 5).
- Audit-Log persistent (kommt aus 1.1).

**Aufwand Tier 2 gesamt:** 3 Sessions.

---

## Tier 3 — External-System-Bridges (2 Sessions)

Single-File ohne Backend → wir bauen **Bridges** statt Integrationen. Das Tool generiert fertige Outputs, der Mensch führt sie aus.

### 3.1 mailto:-Generator für jeden Kommunikations-Touchpoint
- **AMM-Anmeldung (Phase 5.4):** Button erzeugt `mailto:stefan.dehm@…?cc=bjoern.stadler@…&subject=Anmeldung%20{AU}&body=…` mit allen Pflichtdaten (Palettenanzahl, Warenwert, Hebebühne, Fixtermin) prefilled.
- **Zollagent (Phase 5.2.1):** analog mit Zollagent-Email + Datenpaket.
- **Saleskärtchen-Rückgabe (Phase 1):** mailto an Verkauf mit Lücken-Liste.
- **Eskalation an Dustin/Siyad (Kapitel 8):** pro Eskalations-Situation ein Button mit vorgefertigter Mail.

### 3.2 Telefon- und WhatsApp-Bridges
- `tel:`-Links zu Kunde / Spedition.
- `https://wa.me/{kundenNummer}`-Link für WhatsApp-Web.
- Logging: jeder Klick auf Bridge erzeugt Eintrag im Audit-Log + Pflichtfeld „Was war das Ergebnis?".

### 3.3 SharePoint-Pfad-Generator
- Tool baut Vorschlags-URL nach Schema (sobald 0.2 geklärt): z. B. `…/Auftragsbearbeitung/{AU}/`. Click-to-Open.
- "Ablage prüfen"-Button öffnet den Ordner im Browser.

**Aufwand:** 2 Sessions.

---

## Tier 4 — Fulfilment-Pipeline-Integration (2–3 Sessions)

Phase 2 ist heute eine **fremde Excel-Datei**. Tool muss sie nicht ersetzen, aber den User durch die Pflichtspalten lotsen.

### 4.1 Pflichtspalten-Form pro Schritt
- Schritt 2.1 (Spalten B+C+D), 2.2 (G+L+M+N), 2.3 (E+H+I), 2.4 (N+Q) — jeweils dedizierter Sub-Stage mit den Spaltennamen als Form-Feldern (sobald 0.3 geklärt).
- Tool-State spiegelt die Excel-Realität.

### 4.2 Excel-Export für Copy-Paste
- "Pipeline-Zeile kopieren"-Button → erzeugt eine Tab-getrennte Zeile in Spalten-Reihenfolge der SharePoint-Excel. Janna paste'd direkt in die Excel.
- Alternativ: Download als 1-Zeilen-CSV.

### 4.3 Optional Tier 4-Stretch — bidirektionaler Sync
- Falls L2 fällt (Backend erlaubt) → MS-Graph-API Live-Sync. **Kein Default**, weil Riesen-Aufwand und Auth-Komplexität.

**Aufwand:** 2 Sessions (4.1 + 4.2).

---

## Tier 5 — Reporting + Compliance (2 Sessions)

### 5.1 Tagescheckliste-Modus (Kapitel 7)
- "Heute"-View, oben fixiert: laufende Pflichten als Haken-Liste.
  - 2× tägl. Sync mit Dima — mit Reminder-Slots vormittags/nachmittags.
  - Outlook-Inbox prüfen.
  - Saleskärtchen-Eingang prüfen.
  - Pipeline-Update.
- Browser-Notification (Permission opt-in) zur Sync-Zeit.

### 5.2 Wochenreport-Generator (Kapitel 6.5)
- Knopf in Phase 6 / Sidebar: „Wochenreport erstellen".
- Aggregiert aus localStorage: Status aller Aufträge, Klärfälle, Engpässe, abgeschlossene Aufträge.
- Output: PDF via jsPDF + `mailto:dustin@…` mit PDF-Anhang-Hinweis (mailto kann keine Anhänge → Tool zeigt PDF und sagt „bitte anhängen").

### 5.3 Eskalations-Pad (Kapitel 8)
- Pro Eskalations-Situation aus Kapitel 8 ein vorbereiteter Button mit korrektem Empfänger:
  - Kunde nicht erreichbar → Dustin/Siyad
  - Saleskärtchen unvollständig → Verkauf
  - JTL-Diskrepanz → Siyad
  - AMM/Spedition/Zoll → Dustin
  - Reklamation → Dustin
- Button öffnet vorbereitete Email mit Auftragskontext.

**Aufwand:** 2 Sessions.

---

## Tier 6 — Härtung & Verifikation (2 Sessions)

### 6.1 Verify-Pipeline browser-tauglich machen
- `verify.py` braucht Python — nicht installiert. Logik nach `verify.html` portieren: Browser-JS, lädt `data/jtl_data.json` + parsed `JTL_DATA` aus dem HTML, prüft Shape + Cross-Match.
- XLSX-Diff via SheetJS-CDN (kein Node nötig).
- **Akzeptanz:** Janna doppelklickt `verify.html`, sieht grünes "ALL CHECKS PASSED" oder roten Diff.

### 6.2 Acceptance-Suite pro Phase
- Pro Phase 1–6 ein **Smoke-Test** als Browser-Test (in `verify.html` integriert):
  - simuliert State, prüft, ob Phase abschließbar / blockiert wie spezifiziert.
- Verhindert Regression bei Refactoring.

### 6.3 JSDoc-Types (ROADMAP_v1 §3.2)
- `@typedef` für `S`, `JTLEntry`, Phase-Schema. Keine TS-Migration.

**Aufwand:** 2 Sessions.

---

## Tier 7 — Stretch (optional, je nach Zeit)

- **Onboarding-Modus** für neue Mitarbeiter — Tooltip-Tour pro Phase.
- **Dustin-Dashboard** — Read-only-View aller laufenden Aufträge, Engpass-Hinweise, Eskalations-Liste.
- **Aging-Logik** — Aufträge in Phase X länger als N Tage → roter Hinweis im Dashboard.
- **CSS-Theme & Mobile-Layout** (= Tier 1.2 + 2.2 aus altem ROADMAP.md).
- **Modularisierung mit Build-Step** (= Tier 1.1 aus altem ROADMAP.md). Nur wenn L1 fällt.

---

## Definition of „100 % funktionierendes Tool"

Das Tool ist **nicht** fertig, wenn der Code sauber ist oder UI poliert. Es ist fertig, wenn:

1. **Coverage:** jede der 6 Phasen + Kapitel 7 (Pflichten) + Kapitel 8 (Eskalation) hat einen first-class Platz im Tool. Keine Phase im Kopf des Mitarbeiters, keine Pflicht „nebenbei".
2. **Persistenz:** Browser-Crash, Tab-Schließen, Power-Loss → 0 Datenverlust.
3. **Übergabe:** Phase-Wechsel ist explizit, nachvollziehbar, mit Audit-Eintrag.
4. **Bridges:** jeder externe System-Touchpoint (Outlook/WhatsApp/Telefon/SharePoint/AMM/Zollagent) ist 1 Klick.
5. **Reporting:** Tagescheckliste täglich, Wochenreport wöchentlich — beide aus dem Tool, ohne manuelle Aggregation.
6. **Eskalation:** jede der 7 Eskalations-Situationen aus Kapitel 8 hat ihren Button.
7. **Verify:** browserbasierte Verify-Suite läuft grün vor jedem Release.
8. **Constraint-Treue:** Single-File-HTML (oder erlaubter Build-Step nach L1-Entscheidung), offline lauffähig.

---

## Reihenfolge — was machen wir wann

```
0  Klärung           ← BLOCKIEREND. Macht User. ~30 min.
1  Operatives Fundament    ← muss vor allem anderen. 2 Sessions.
2  Phase-Coverage          ← strukturelle Basis. 3 Sessions.
3  External-Bridges        ← parallelisierbar mit 4. 2 Sessions.
4  Pipeline-Integration    ← parallelisierbar mit 3. 2 Sessions.
5  Reporting + Compliance  ← braucht 1+2. 2 Sessions.
6  Härtung                 ← begleitet alle Tiers, dediziert am Ende. 2 Sessions.
7  Stretch                 ← optional.
```

**Gesamt-Schätzung bis 100 %:** ~13 Sessions.

**Erste Session (sobald Tier 0 geklärt):** Tier 1.1 + 1.3 — Auto-Save + Phase-Lock-Modell. Das ist das absolute Minimum, ohne das alles andere wackelig steht.

---

## Risiken (sortiert nach Wahrscheinlichkeit × Schaden)

| Risiko | Wahrsch. | Schaden | Mitigation |
|---|---|---|---|
| Tier 0 wird nicht beantwortet → Roadmap blockiert | hoch | hoch | Sofortige User-Konversation, Antworten zu `OPEN_QUESTIONS.md` |
| L1-Entscheidung fällt zu Build-Step → Tooling-Setup nötig (Node) | mittel | mittel | Erst Verify-Pipeline browser-portieren (6.1), dann Build-Diskussion |
| Diskrepanz Mirko/Janna nicht aufgelöst → falscher Phase-Owner | hoch | mittel | L5-Frage in Tier 0, vor 2.3 |
| SharePoint-Pipeline-Spalten ändern sich | mittel | mittel | Spalten-Schema in `data/pipeline_schema.json` zentralisieren, eine Stelle zum Ändern |
| Datenverlust bei localStorage-Quota → 5–10 MB Limit | niedrig | hoch | PDF-Anhänge nicht in localStorage, sondern user-driven Export |
| 4 Versionen in 2 Wochen → Iterationsgeschwindigkeit produziert Bugs | mittel | mittel | Tier 6.2 Acceptance-Suite **nicht erst am Ende**, sondern parallel ab Tier 2 |
