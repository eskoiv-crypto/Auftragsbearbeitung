# DECISIONS — Architektur-Sperren

Stand: 2026-05-04. Diese Entscheidungen sind verbindlich, bis explizit revidiert. Begründung pro Entscheidung — wer eine ändern will, muss die Begründung aushebeln.

---

## D1 — Output: Single-File HTML
**Entscheidung:** `elvinci_auftragssteuerung.html` bleibt eine einzelne, doppelklickbare Datei. Kein Build-Step. CDN-Skripte (Chart.js, jsPDF, neu: SheetJS) sind erlaubt.
**Quelle:** L1 bestätigt 2026-05-04.
**Konsequenz:** Modularisierung mit Build-System (alter ROADMAP §1.1) bleibt **nicht** umgesetzt. Code-Organisation durch Section-Kommentare im selben File.

## D2 — Persistenz: localStorage + JSON Export/Import
**Entscheidung:**
- **Primär:** Browser-`localStorage` mit Versions-Key (`elvinci_S_v1`, `elvinci_drafts_v1`).
- **Backup:** Tool exportiert auf Knopfdruck eine `elvinci_backup_YYYY-MM-DD.json` mit `S` + Drafts-Liste. Janna lädt diese Datei manuell in SharePoint hoch.
- **Restore:** Drag-and-Drop oder „Backup laden"-Button.
**Begründung:** Single-User pro Browser (L6) macht localStorage sicher. PDF-Anhänge **nicht** in localStorage (5–10 MB Quota) — stattdessen Filename-Referenz + Download-Hinweis. Manuelle Backup-Datei ist die Brücke zu Multi-Device-Resilienz, ohne Backend.
**Konsequenz:** Tabletwechsel = manueller Backup-Export/-Import. Kein Auto-Sync.

## D3 — Externe Systeme: Bridges, kein API-Call
**Entscheidung:** Tool generiert Outputs (`mailto:`, `tel:`, `https://wa.me/...`, Planner-Tab-URL, SharePoint-Pfad) — der Mensch klickt, das System öffnet sich nativ. Kein OAuth, keine MS-Graph-Calls, keine Backend-Hops.
**Begründung:** Single-File-Constraint (D1) + simple > clever. Bridges sind robust, transparent, debuggbar.
**Konsequenz:** AMM-Anmeldemail kommt als vorbereiteter `mailto:`-Link mit prefilled Body. Anhänge muss Janna manuell anhängen (mailto kann's nicht). Tool erinnert sie aktiv daran.

## D4 — Workflow-Modell: 7 Planner-Buckets als Sidebar, 6 Phasen als Klammer
**Entscheidung:** Sidebar zeigt **7 Buckets** in Planner-Reihenfolge:
1. `bereit für Rechnung (VERKAUF)`
2. `Offene Rechnungen` (Janna)
3. `Kommissionierungsplanung (DUSTIN)`
4. `Versandplanung JANNA`
5. `Abholplan. / Kommunikation (JANNA)`
6. `Angemeldet (JANNA)`
7. `Beendet (Versendet/Abgeholt)`

Pro Bucket sichtbar: zugehörige Stellenbeschreibungs-Phase (1–6) als Hinweis-Banner. Bucket = operative Aktion. Phase = logische Klammer für Reporting + Schulung.
**Begründung:** Buckets sind die echte Realität (Planner = Source of Truth). Stellenbeschreibungs-Phasen 1–6 sind die abstrakte Sicht — nützlich, aber nicht das Arbeitsraster.
**Konsequenz:** „Bucket-Wechsel" ersetzt „Phase-Übergang". Audit-Log trackt Bucket-Changes mit Timestamp.

## D5 — Tag-System: 1:1 zu Planner-Labels
**Entscheidung:** Tool-Tags werden auf **16 Labels** erweitert, Namen exakt wie im Planner. Schema in `data/planner_schema.json`. Kategorien:
- versandart: ABHOLUNG
- kundentyp: Privatkunde
- auftrag-flag: KOMBI-AUFTRAG, SONDERFALL, Kleingeräte, Loose Beladung!, Fixtermin, Zoll, UIT-Code
- status: STORNIERT
- zahlung-flag: Vorkasse (FLEX), Barzahlung (PRIO), ZZ (PRIO)
- zahlung-status: Warte auf Zahlung, Bezahlt
- kommunikation: Kontaktiert
**Begründung:** Eindeutige Übereinstimmung mit Planner-Realität. Keine Übersetzungs-Layer.
**Konsequenz:** `S.isKombi`, `S.isSonder` etc. bleiben semantisch, werden aber durch eine `S.labels: Set<string>` ergänzt. Bestehende Boolean-Flags bleiben für JTL-Match-Logik (`findJTLText`) — Tag-Anzeige kommt aus `S.labels`.

## D6 — Persona: Janna ist die alleinige Stelleninhaberin
**Entscheidung:** Alle Code- und UI-Vorkommen von „Mirko" werden durch „Janna" ersetzt. Dustin bleibt Vorgesetzter (Eskalation, Kommissionierungsplanung-Bucket). Mirko-References werden komplett entfernt, nicht aliassed.
**Begründung:** L5 bestätigt — Mirko nicht mehr bei elvinci.
**Konsequenz:** Single global rename. Audit-Log historische Mirko-Einträge bleiben (Datenintegrität); UI-Beschriftungen aktualisiert.

## D7 — Wochenreport: PDF + Email-Body, datengetrieben
**Entscheidung:** „Wochenreport"-Knopf erzeugt:
1. **PDF** via jsPDF (vorhandene Lib): Header (KW, Stand-Datum), Tabelle aller laufenden Aufträge nach Bucket gruppiert, Eskalations-/Klärfall-Liste, Engpässe (manuell editierbar), Kennzahlen-Block (abgeschlossen-diese-Woche, in-Bearbeitung, eskaliert).
2. **Email-Body** via `mailto:dustin@…?subject=Wochenreport KW NN` mit den Kennzahlen als Klartext im Body. Hinweis: PDF separat anhängen (mailto-Limit).
**Begründung:** „Sinnvoll und Logisch aufbauen" delegiert Design — PDF = lesbar/archivierbar, Email = Versand-Bridge.
**Konsequenz:** Tool aggregiert aus localStorage; Janna braucht nichts manuell zu sammeln.

## D8 — Tagescheckliste: persistente Top-Sidebar
**Entscheidung:** Pinnbar/ausklappbar oben in der Sidebar, immer sichtbar. Inhalt:
- 2× tägl. Sync mit Dima (Slot-Reminder vormittags + nachmittags, Browser-Notification opt-in)
- Outlook-Inbox-Check (Erinnerung nur, kein Outlook-Zugriff)
- Saleskärtchen-Check (öffnet Planner-Tab-Bridge)
- Pipeline-Update (öffnet SharePoint-Pipeline-URL, sobald geliefert)
- Wochenreport (Reminder am Freitag)

Erledigt-Status pro Tag in localStorage; resettet sich um Mitternacht.
**Begründung:** Nutzungsmuster „Tagescheckliste = Tagescheckliste" — sie muss präsent sein, sonst wird sie ignoriert.
**Konsequenz:** Permission-Prompt für Browser-Notifications beim ersten Tageswechsel.

## D9 — Eskalation: Email primär, WhatsApp als Co-Channel bei „Kunde nicht erreichbar"
**Entscheidung:** Pro Eskalations-Situation aus Kapitel 8 ein Knopf:
- Default: `mailto:dustin@…` mit prefilled Subject/Body.
- Sondersituation „Kunde nicht erreichbar": **zwei** Knöpfe parallel (Email + WhatsApp), wie Stellenbeschreibung verlangt („schriftlich UND telefonisch").
**Begründung:** Stellenbeschreibung explizit (Phase 3, Eskalation).
**Konsequenz:** Telefon-Eskalation ist außerhalb des Tool-Scopes (kein VoIP-Call); aber `tel:`-Link auf der gleichen Karte vereinfacht es.

## D10 — Verify-Pipeline: browserbasiert (`verify.html`)
**Entscheidung:** `verify.py` wird **nicht** ausgeführt. Stattdessen `verify.html`: lädt `data/jtl_data.json`, parsed `JTL_DATA` aus dem Tool-HTML, vergleicht. XLSX-Diff via SheetJS-CDN.
**Begründung:** „keep it simple" — User hat weder Python noch Node real installiert. Browser ist da. Verify läuft mit Doppelklick.
**Konsequenz:** `verify.py` bleibt im Repo als Referenz, läuft aber nicht. README dokumentiert beides.

## D11 — Acceptance-Tests: parallel zur Feature-Entwicklung, nicht erst am Ende
**Entscheidung:** Jedes neu gebaute Feature in Tier 1+2+3 bekommt einen Smoke-Test in `verify.html`. Mini-DSL: pro Test eine Funktion, die `S` simuliert und ein erwartetes Ergebnis prüft.
**Begründung:** 4 Versionen in 2 Wochen + keine Tests = Risiko. Tests ab jetzt parallel.
**Konsequenz:** Roadmap-Tier 6.2 wird über alle Tiers verteilt, nicht am Ende konsolidiert.

---

## Was bleibt offen (Daten, nicht Architektur)

Diese Items sind kein Architektur-Block — sie sind **Befüllungs-Daten** für die fertigen Bridges. Tool kann ohne sie gebaut werden, aber die Bridges sind erst „scharf", wenn die Daten da sind.

- **AMM-Mail-Vorlage:** Bestehende Original-Mail (eine kürzliche Anmeldung an Stefan Dehm / Björn Stadler) → ich derive das Template daraus. Bitte 1 Beispiel-Mail teilen.
- **Zollagent:** Email + Pflichtfelder + ggf. eine Beispiel-Mail.
- **SharePoint-Pipeline-URL:** Direktlink zur Excel-Datei.
- **Single-Card-Planner-Deeplink:** Eine Karte im Planner öffnen, Browser-Adresszeile kopieren. Format ist meist `https://tasks.office.com/<tenant>/Home/Task/<task-id>`.
- **Wochenreport-Empfänger:** Email-Adresse Dustin.
- **Eskalation Siyad:** Email-Adresse.
- **Pipeline-Excel-Sharepoint-Pfad-Schema** für SharePoint-Ablage pro Auftrag.

Kein Architektur-Veto — Tool wird mit Platzhaltern gebaut, Janna ersetzt sie später.
