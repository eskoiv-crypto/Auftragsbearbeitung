# OPEN QUESTIONS — Tier 0 vor Roadmap-Start

Stand: 2026-05-04. Antworten bitte direkt unter jeder Frage eintragen, dann gestrichen.

Diese Antworten sind **blockierend** für die Roadmap (siehe `ROADMAP_v2_workplace_alignment.md` Tier 0).

---

## L1 — Single-File-Constraint

**Frage:** Bleibt das Tool ein einzelnes, doppelklickbares HTML-File? Oder darf ein Build-Step rein (Node, Modularisierung mit Bundling auf 1 HTML)?

**Antwort:**

---

## L2 — Persistenz-Strategie

**Frage:** Reicht `localStorage` (pro Browser, pro Gerät), oder muss bei einem Tabletwechsel der Auftragsstand übernommen werden (= Backend nötig)?

**Antwort:**

---

## L3 — Externe Systeme

**Frage:** OK, wenn das Tool nur **Bridges** baut (mailto/tel/whatsapp/SharePoint-Link) und der Mensch die Aktion ausführt? Oder wirklich automatisierter API-Aufruf gewünscht (= OAuth + Backend)?

**Antwort:**

---

## L4 — Phase-Anzahl

**Frage:** Tool wird auf 6 Phasen umgebaut (1 Saleskärtchen, 2 Pipeline, 3 Kundenkommunikation, 4 Rechnung, 5 Spedition/Zoll, 6 Doku) — bestätigt?

**Antwort:**

---

## ~~L5 — Owner Phase 4~~ ✅ BEANTWORTET (2026-05-04)

**Frage:** Stellenbeschreibung sagt **Janna** macht Rechnung. Handover sagt **Mirko**. Was stimmt?

**Antwort:** Mirko gibt es nicht mehr bei elvinci. **Janna** ist alleinige Stelleninhaberin Auftragsbearbeitung. **Dustin** ist alleiniger Vorgesetzter. Tool-Code muss alle „Mirko"-Beschriftungen zu „Janna" ändern.

---

## L6 — Multi-User

**Frage:** Tool ist Single-User pro Browser/Tablet (Janna). Oder soll Mirko/Dustin gleichzeitig im selben Tool arbeiten können (= Synchronisation, vermutl. Backend)?

**Antwort:**

---

## ~~0.2 — Sales-/Teamskärtchen-System~~ ✅ BEANTWORTET (2026-05-04)

**Frage:** Was ist das Saleskärtchen / Teamskärtchen technisch?

**Antwort:** **Microsoft Teams Planner**. Tab-URL geliefert (siehe `data/planner_schema.json` → `tab_url`). 7 Buckets (verkauf → rechnung → kommi → versand/abholung → angemeldet → beendet). 16 Labels.

**Folge-Frage offen:** Single-Card-Deeplink-Format. Die Tab-URL öffnet das gesamte Planner-Board. Für Deeplink zu **einer** Karte brauche ich entweder:
- (a) das Format `https://tasks.office.com/<tenant>/Home/Task/<task-id>` — kannst du eine Beispiel-URL aus dem Planner kopieren? (Karte öffnen → Browser-Adresszeile)
- (b) die Antwort: „Tab-URL reicht, Janna sucht selbst die Karte" — dann bauen wir nur Tab-Bridge, keinen Card-Deeplink.

---

## ~~0.3 — Fulfilment Pipeline Spalten~~ ✅ BEANTWORTET (2026-05-04)

**Frage:** Die Pipeline-Excel hat Spalten B/C/D/G/L/M/N/E/H/I/N/Q. Was steht exakt drin?

**Antwort:** Datei geliefert (`docs/pipeline_export_2026-05-04.csv`). Schema dekodiert in `data/pipeline_schema.json`. Mapping:

| Phase-Schritt | Spalten | Bedeutung |
|---|---|---|
| 2.1 (Start) | B + C + D | Kunde, Auftragsnr., Artikel-Stückzahl |
| 2.2 (Kommi-Tag) | G + L + M + N | Kommissioniertag, Zahlungsziel, Versandart, Bezahlt? |
| 2.3 (Notizen + Versand) | E + H + I | NOTIZEN AMM, Versand-/Abholdatum, Versandanmeldung |
| 2.4 (Abwicklung) | N + Q | Bezahlt? (final), erledigt-Marker |

Pipeline hat zusätzlich Spalten S–AA (Zollpapiere, Paletten, Reste, Rechnungskorrektur, Kundeninfos K/L, Auftrag erledigt) — sind in der Stellenbeschreibung **nicht** als Pflichtspalten genannt. Annahme: Tool füllt Pflichtspalten + bietet diese als Optional-Block.

**Folge-Frage offen:** SharePoint-Direktlink zur Pipeline-Excel — bitte URL angeben.

---

## 0.4 — AMM-Anmeldemail

**Frage:** Gibt es eine bestehende Mail-Vorlage für die AMM-Anmeldung (Phase 5.4)?
- Empfänger: Stefan Dehm (`?@?`) UND Björn Stadler (`?@?`) — beide TO oder einer CC?
- Betreff-Schema?
- Pflicht-Inhalte (Palettenanzahl, Warenwert, …)?

Wenn vorhanden: bitte Mail im Repo ablegen unter `templates/amm_anmeldung.txt`.

**Antwort:**

---

## 0.5 — Zollagent

**Frage:** Wer ist der Zollagent? Email + Pflichtfelder im Datenpaket (Rechnung, Auftragsnummer, Warenwert, Kundendaten, Palettenzahl — vollständig?)

**Antwort:**

---

## 0.6 — Wochenreport

**Frage:** Wie soll der Wochenreport an Dustin aussehen?
- Format: PDF / Email-Body / Word / Karte im Teamskärtchen / anderes?
- Inhalt-Struktur: bitte Beispiel oder Template.
- Adressat-Email: `?@?`

**Antwort:**

---

## 0.7 — Tagescheckliste

**Frage:** Tagescheckliste-Modus im Tool — soll das eine eigene "Today"-View sein (immer sichtbar oben), oder eine separate Seite, oder ein Modal beim Browser-Start?

**Antwort:**

---

## 0.8 — Eskalation

**Frage:** Bevorzugter Kanal für Eskalation an Dustin/Siyad — Email, WhatsApp, Telefon, Karte? Wenn pro Situation unterschiedlich: bitte Tabelle.

**Antwort:**

---

## 0.10 — Tooling

**Frage:** Kannst du Python (mit `pip install openpyxl`) ODER Node.js installieren?
- Wenn ja: welches?
- Wenn nein: ich portiere die Verify-Pipeline in den Browser (Tier 6.1) — kein Tooling nötig.

**Antwort:**

---

## Sonstige

Jede zusätzliche Anforderung, die in der Stellenbeschreibung **nicht** steht, aber im Tool sein soll:

**Antwort:**
