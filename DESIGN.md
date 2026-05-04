# DESIGN — Apple-like Workspace für die Auftragsbearbeitung

Stand: 2026-05-04. Vorschlag mit Varianten, Tokens und offenen Fragen. Nicht implementiert — entscheiden, dann bauen.

**Ziel:** Tool, das Janna durch ihren **gesamten Arbeitsplatz** führt — nicht nur durch einen Auftrag. Apple-Niveau in Ruhe, Klarheit, Polish.

---

## 1. Philosophie

| Apple-Prinzip | Übersetzt für die Auftragsbearbeitung |
|---|---|
| **Ruhe vor Inhalt** | Stille Hintergründe, viel Weißraum, Inhalt darf atmen |
| **Hierarchie durch Größe + Gewicht, nicht Farbe** | Farbe nur für Status & Aktion — der Rest ist neutral |
| **Direkte Manipulation** | Drag-and-Drop zwischen Buckets, Klick = Aktion |
| **Progressive Disclosure** | Detail erst, wenn ich danach frage. Keine Wand aus Feldern. |
| **Räumliche Konsistenz** | UI-Elemente bleiben am gleichen Ort. Lernkurve = einmal. |
| **Ambient Awareness** | Heute-Status auf einen Blick — ohne hinzustarren |
| **Polish** | Sanfte Schatten, weiche Übergänge, Frosted Glass, 60fps |
| **Multi-Device** | Funktioniert auf Tablet (Janna im Lager) wie auf Desktop |

**Kernprinzip:** Das Tool ist *nicht* ein Auftragsformular mit 50 Feldern. Es ist ein **Cockpit**. Was Janna *jetzt* tun muss, ist groß. Alles andere ist klein und am Rand.

---

## 2. Drei Layout-Varianten

### Variante A — „Apple Mail" (3-Pane)

```
┌─────────────────────────────────────────────────────────────────┐
│  ☀ Heute · 12 aktiv · 3 fällig · ⏰ Sync mit Dima 14:00          │ ← Heute-Bar
├──────────────┬─────────────────────┬────────────────────────────┤
│              │                     │                            │
│  Filter      │  Auftrags-Liste     │  Detail-Workspace          │
│  ─────       │  ───────────────    │  ───────────────────────   │
│  Heute       │  ⬤ Outlet Simex     │  Outlet Simex              │
│  Alle (12)   │    AU2026010813260  │  AU2026010813260           │
│              │    Bucket 3 · KOMBI │  ━━━━●─○─○─○─○─○─○         │
│  ──────      │    Vor 2h           │  Bucket 1 von 7            │
│  📥 verkauf  │                     │                            │
│  📋 rechnung │  ⬤ Kaldi Prime      │  [Hero: Kunde, AU, Tags]   │
│  📦 kommi    │    AU2026010913264  │                            │
│  🚚 versand  │    Bucket 5 · ZOLL  │  [Aktive Bucket-Form]      │
│  🚐 abholung │    Heute            │                            │
│  ✓ angemeld. │                     │  [Action Dock unten:       │
│  🏁 beendet  │  ⬤ ...              │   📧 AMM   📱 WhatsApp     │
│              │                     │   📋 Planner   ⚠ Eskal.]   │
│  ──────      │                     │                            │
│  + Tag-Filter│                     │                            │
│              │                     │                            │
└──────────────┴─────────────────────┴────────────────────────────┘
```

**Pro:** klassisches 3-Pane-Mailprogramm-Mental-Model, jeder weiß sofort wie's geht.
**Contra:** der Detail-Workspace ist relativ schmal; 50/50-Split verschwendet Platz auf großen Bildschirmen.

---

### Variante B — „Linear Kanban" (Planner-Mirror)

```
┌─────────────────────────────────────────────────────────────────┐
│  ☀ Heute · 12 aktiv · 3 fällig · ⏰ 14:00 Sync                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  verkauf  rechnung  kommi   versand  abholung angem.  beendet  │
│  (3)      (4)       (1)     (2)      (1)     (1)     (8)       │
│  ────     ────      ────    ────     ────    ────    ────      │
│  ┌──┐     ┌──┐      ┌──┐    ┌──┐     ┌──┐    ┌──┐    ┌──┐      │
│  │📋│     │📋│      │📋│    │📋│     │📋│    │📋│    │📋│      │
│  └──┘     └──┘      └──┘    └──┘     └──┘    └──┘    └──┘      │
│  ┌──┐     ┌──┐                                                  │
│  │📋│     │📋│      Klick auf Karte → Inspector slidet von      │
│  └──┘     └──┘      rechts rein und zeigt vollen Workspace.    │
│  ┌──┐     ┌──┐                                                  │
│  │📋│     │📋│                                                  │
│  └──┘     └──┘                                                  │
│           ┌──┐                                                  │
│           │📋│                                                  │
│           └──┘                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Pro:** 1:1 zum bekannten Planner-Layout. Drag-and-Drop fühlt sich natürlich an. Auftragsfluss visuell.
**Contra:** Detail-Arbeit findet immer im überlagerten Inspector statt — bricht den Lesefluss. Bei 12+ Aufträgen pro Spalte wird's eng.

---

### Variante C — „Cockpit" (Hybrid, EMPFOHLEN) ⭐

```
┌─────────────────────────────────────────────────────────────────┐
│  ⌘ Auftragssteuerung           ⏰ 14:00 Sync · 3 fällig · 🔔     │ ← Top
├─────────────────────────────────────────────────────────────────┤
│  HEUTE: ☐ Outlook · ☐ Saleskärtchen · ☐ Sync Dima · ☐ Pipeline  │ ← Tagescheckliste
├──────┬────────────────────────────────────────────────┬─────────┤
│      │                                                │         │
│ List │  ┌─────────────────────────────────────────┐  │  Inspe- │
│ ──── │  │  Outlet Simex D.O.O                     │  │  ctor   │
│ 🔍   │  │  AU2026010813260 · RE2026010813599      │  │  ────   │
│      │  │  ⬤⬤⬤⬤ KOMBI · ZZ (PRIO) · Privatkunde   │  │  Audit  │
│ ⬤Out…│  └─────────────────────────────────────────┘  │  Komm.  │
│ ⬤Kal…│                                                │  Eskal. │
│ ⬤Sim…│  ━━━━━━●─────○─────○─────○─────○─────○─────○ │  Backup │
│ …    │  verkauf rechng kommi vrsand abhlg angm beend │         │
│      │       Bucket 3 von 7 · Phase 4 · Dustin       │         │
│ + Neu│                                                │         │
│      │  ┌─────────────────────────────────────────┐  │         │
│      │  │  KOMMISSIONIERUNGSPLANUNG               │  │         │
│      │  │  ─────────────────────────              │  │         │
│      │  │  Kommi-Termin:    [Datepicker]          │  │         │
│      │  │  Verantwortlich:  Dustin                │  │         │
│      │  │  …                                      │  │         │
│      │  └─────────────────────────────────────────┘  │         │
│      │                                                │         │
│      │  ──── Action Dock ────                        │         │
│      │  📧 AMM-Mail   📱 WhatsApp   📞 Anruf         │         │
│      │  📋 Planner-Karte   📂 SharePoint   ⚠ Eskal.  │         │
│      │                                                │         │
│      │  ←  Bucket zurück       Bucket weiter →       │         │
│      │                                                │         │
└──────┴────────────────────────────────────────────────┴─────────┘
```

**Pro:**
- **Tagescheckliste** als persistente Top-Bar (Kapitel 7 der Stellenbeschreibung) — immer sichtbar, kein Versteck.
- **Auftragsliste links** (collapsible) zeigt 5–10 parallele Aufträge mit Status-Tag.
- **Center = Workspace** — der Auftrag bekommt den größten Raum. Hero, Bucket-Stepper, aktive Form, Action Dock.
- **Inspector rechts** (collapsible) sammelt Audit, Kommentare, Eskalations-Pad, Datensicherung — alles, was *nicht* dauerhafte Sicht braucht.
- Bucket-Stepper zeigt sieben Punkte horizontal — Apple-Reminders/iOS-Setup-style. Sofort verständlich wo der Auftrag steht.
- Action Dock gruppiert externe Bridges in einer fixen Zone — Muscle Memory entwickelt sich.

**Contra:**
- Maximale Bildschirmbreite empfohlen ≥ 1200 px — auf Tablet (768 px) Inspector & Liste collapsing.
- Mehr UI-Elemente sichtbar gleichzeitig als bei A oder B — Disziplin beim Styling nötig, sonst wird's busy.

**Empfehlung: Variante C.** Sie spiegelt am besten das Anforderungsprofil aus der Stellenbeschreibung wider (alle 6 Phasen + Kapitel 7+8 sichtbar) und liefert gleichzeitig Apple-Cockpit-Ruhe. A wäre simpler, B wäre verspielter — C ist *richtig*.

---

## 3. Apple Design Tokens

### 3.1 Farben — System-Palette (Light + Dark)

```css
/* LIGHT MODE — Default */
--bg-primary:    #f5f5f7;   /* macOS Window Background */
--bg-secondary:  #ffffff;   /* Cards, Surfaces */
--bg-tertiary:   #fbfbfd;   /* Hover-States, gentle elevation */
--separator:     rgba(60, 60, 67, 0.12);
--label:         #1d1d1f;   /* Primary text */
--label-2:       #6e6e73;   /* Secondary text */
--label-3:       #86868b;   /* Tertiary text, hints */
--accent:        #0071e3;   /* Apple blue (slightly warmer than iOS systemBlue) */
--accent-hover:  #0077ed;
--success:       #00c896;   /* Slightly more teal than iOS green for differentiation */
--warning:       #ff9500;   /* iOS systemOrange */
--error:         #ff3b30;   /* iOS systemRed */
--info:          #5856d6;   /* iOS systemPurple — for "open Planner", "external link" */

/* DARK MODE */
--bg-primary:    #1d1d1f;
--bg-secondary:  #2c2c2e;
--bg-tertiary:   #3a3a3c;
--separator:     rgba(255, 255, 255, 0.10);
--label:         #f5f5f7;
--label-2:       #a1a1a6;
--label-3:       #6e6e73;
--accent:        #0a84ff;   /* iOS systemBlue dark */
--success:       #30d158;
--warning:       #ff9f0a;
--error:         #ff453a;
--info:          #bf5af2;
```

**Bucket-Farb-Pillen** (subtil, nicht dominant):
- verkauf: `#5856d6` (purple, eingehend)
- rechnung: `#ff9500` (orange, in Bearbeitung)
- kommi: `#34c759` (green, geplant)
- versand: `#007aff` (blue, in Bewegung)
- abholung: `#ff3b30` (red, sofortige Aufmerksamkeit Kunde)
- angemeldet: `#5ac8fa` (light blue, wartend)
- beendet: `#86868b` (gray, archiv)

### 3.2 Typografie

```css
font-family: -apple-system, "SF Pro Display", "SF Pro Text", "Inter", system-ui, sans-serif;
font-feature-settings: "ss01", "cv01"; /* Apple-Stil: alternate forms wo verfügbar */

/* Skala (iOS Type Style nahegelegt) */
--type-large-title:  34px / 1.21 / 700;  /* z. B. Today-Header */
--type-title-1:      28px / 1.21 / 700;  /* Auftrags-Hero */
--type-title-2:      22px / 1.27 / 600;  /* Bucket-Sektion */
--type-title-3:      20px / 1.25 / 600;
--type-headline:     17px / 1.29 / 600;  /* Form-Labels */
--type-body:         17px / 1.41 / 400;  /* Default Read */
--type-callout:      16px / 1.31 / 400;
--type-subhead:      15px / 1.33 / 400;
--type-footnote:     13px / 1.38 / 400;  /* Audit-Einträge */
--type-caption-1:    12px / 1.33 / 400;
--type-caption-2:    11px / 1.18 / 500;  /* Tags, Badges */
```

Monospace nur für AU- und RE-Nummern: `"SF Mono", "JetBrains Mono", monospace`.

### 3.3 Spacing — 4-Punkt-Grid

```
4 / 8 / 12 / 16 / 20 / 24 / 32 / 40 / 56 / 80
```

Nichts dazwischen. Konsistenz schlägt Pixelfeintuning.

### 3.4 Radius

- **Card:** 14 px (iOS Card)
- **Pill:** 999 px
- **Input:** 10 px
- **Modal/Sheet:** 18 px
- **Top Bar Item / Button:** 8 px

### 3.5 Schatten

```css
--shadow-1: 0 1px 2px rgba(0,0,0,0.04), 0 1px 3px rgba(0,0,0,0.06);
--shadow-2: 0 4px 6px rgba(0,0,0,0.05), 0 2px 4px rgba(0,0,0,0.04);
--shadow-3: 0 10px 15px rgba(0,0,0,0.08), 0 4px 6px rgba(0,0,0,0.04);
--shadow-glass: 0 8px 24px rgba(0,0,0,0.12);
```

### 3.6 Frosted Glass (Top-Bar, Inspector-Header)

```css
background: rgba(255, 255, 255, 0.72);
backdrop-filter: saturate(180%) blur(20px);
border-bottom: 1px solid var(--separator);
```

### 3.7 Motion — alles fühlt sich „gefedert" an

```css
--ease-out:    cubic-bezier(0.32, 0.72, 0, 1);
--ease-spring: cubic-bezier(0.5, 1.25, 0.75, 1.25);
--dur-fast:    0.15s;
--dur-base:    0.25s;
--dur-slow:    0.4s;
```

- Hover-States: fade `0.15s ease-out`
- Bucket-Wechsel: slide + fade `0.25s ease-out`
- Sheet-Open: spring `0.4s ease-spring`
- Tag-Erscheinen: pop-in `0.2s ease-spring`

Wichtig: **`prefers-reduced-motion: reduce`** respektieren — alle Animationen auf 0s clampen.

---

## 4. Informations-Architektur

### 4.1 Top Bar (immer sichtbar)

- **Logo + Titel** „Auftragssteuerung"
- **Sync-Reminder:** „⏰ 14:00 Sync mit Dima" (orange wenn überfällig)
- **Tagesstats:** „12 aktiv · 3 fällig · 1 eskaliert"
- **🔔 Notifications** (Sammelpunkt für Eskalationen, neue Saleskärtchen)
- **Suche** (`⌘K`)
- **Einstellungen / Backup** (Zahnrad)

### 4.2 Tagescheckliste-Strip

Nur wenn die Top-Bar **expandiert** ist (Klick auf Sync-Pille). Default: zugeklappt.
- ☐ Outlook prüfen
- ☐ Saleskärtchen-Eingang
- ☐ Pipeline-Update vormittags
- ☐ Sync mit Dima 09:00
- ☐ Sync mit Dima 14:00
- ☐ Pipeline-Update nachmittags
- ☐ Wochenreport (Freitag)

Erledigt-Status pro Tag in localStorage; reset um Mitternacht.

### 4.3 Liste links (collapsible)

- Suchfeld oben
- Filter-Pillen: „Heute" / „Alle" / je Bucket
- Auftragskarte:
  ```
  ⬤ Outlet Simex D.O.O
    AU2026010813260
    📋📋 Bucket 3 · KOMBI
    Stand: vor 2h · ⚠ heute fällig
  ```
- Sortierung: Default „heute fällig zuerst", dann nach Anlegedatum
- „+ Neu" am Ende

### 4.4 Center — Workspace (das Herz)

```
┌────────────────────────────────────────────────────┐
│ Outlet Simex D.O.O                                 │
│ AU2026010813260 · RE2026010813599                  │
│ [KOMBI] [ZZ (PRIO)] [Privatkunde] [+]              │
└────────────────────────────────────────────────────┘
        ━━━━━●─────○─────○─────○─────○─────○─────○
        verk  rech  komm  vers  abh  angem beend
              Bucket 3 · Phase 4 · Dustin

[Bucket-spezifische Inhalts-Card]

──── Action Dock ────
Bridges horizontal in Reihenfolge:
[📧 AMM-Mail] [📋 Planner-Karte] [📂 SharePoint] [📱 WhatsApp]
[📞 Anruf] [⚠ Eskalation]

[← Bucket zurück]                  [Bucket weiter →]
```

**Bucket-spezifische Inhalts-Cards (pro Bucket):**

| Bucket | Inhalt |
|---|---|
| verkauf | Saleskärtchen-Vollständigkeitscheck, Pflichtfeld-Liste, Zurück-an-Verkauf-Button |
| rechnung | Bestehender JTL-Generator (Versandart/Hebebühne/Zahlungsziel + Templates) + JTL-Abgleich-Form (Phase 4.2) |
| kommi | Kommissionierungs-Datepicker, Verantwortlicher, Kapazitätshinweis |
| versand | AMM-Mail-Generator, Palettenanzahl, Hebebühne, Fixtermin |
| abholung | Termin-Vereinbarung, Kontakt-Status, WhatsApp-Bridge prominent |
| angemeldet | Wartet-Status, Anlieferzeitfenster, manueller Status-Update-Button |
| beendet | Audit-Snapshot, PDF-Ticket-Export, Wochenreport-Eintrag |

### 4.5 Inspector rechts (collapsible)

Tabs: **Audit** (default) · **Kommentare** · **Eskalation** · **Backup**

- **Audit:** chronologische Liste, jüngster oben, mit Bucket-Farbe-Punkt
- **Kommentare:** Threading nicht nötig — flache Liste, „Wer / Wann / Was". Rich-Text minimal.
- **Eskalation:** Pad mit 6 Knöpfen (siehe Kapitel 8 Stellenbeschreibung):
  - Kunde nicht erreichbar → Dustin/Siyad (Email + WhatsApp parallel)
  - Saleskärtchen unvollständig → Verkauf
  - JTL-Diskrepanz → Siyad
  - Logistik-Problem → Dustin
  - Reklamation → Dustin
  - Wiederkehrender Fehler → Dustin
- **Backup:** ☁ Backup speichern · 📂 Backup laden · ↺ Reset

---

## 5. Vergleich mit aktuellem Tool

| Bereich | Aktuell | Variante C |
|---|---|---|
| Layout | Single-Column, mittig 720 px | Drei Spalten + Top-Bar, voll genutzte Breite |
| Auftragsanzahl | 1 pro Browser | 5–10 parallel in Liste |
| Tagescheckliste | nicht vorhanden | Top-Strip + Permission-basierte Reminder |
| Eskalation | nicht vorhanden | Inspector-Tab, 6 Buttons |
| Bridges | nicht vorhanden | Action Dock unten im Workspace |
| Bucket-Stepper | Progress-Bar 7 Segmente (oben) | Horizontal Stepper unter Hero, Klick zum Wechsel |
| Tags | „Pillen" auf t-* Klassen | gleichlogisch, aber Apple-Pill-Stil mit System-Farben |
| Audit | Sidebar-Sektion | Inspector-Tab, jüngster oben, mit Bucket-Punkt |
| Theme | Dark only | Light default, Dark via OS-Pref |

---

## 6. Open Design Questions

| # | Frage | Default-Annahme |
|---|---|---|
| Q1 | Light Mode default oder Dark Mode default oder Auto (`prefers-color-scheme`)? | **Auto** |
| Q2 | Bucket-Stepper: horizontal (wie iOS Setup) oder vertikal (wie Mail-Sidebar)? | **horizontal** |
| Q3 | Drag-and-Drop zwischen Buckets aktivieren oder nur explizite „Weiter →" Buttons? | **Beide** — Drag als Power-Move, Button als Default |
| Q4 | Inspector-Tabs: 4 (Audit/Komm/Eskal/Backup) oder mehr/weniger? | **4** |
| Q5 | Top-Bar mit Suche `⌘K` jetzt schon oder später? | **später** (Tier 2.2+) |
| Q6 | Multi-Auftrag-Liste: Karten-Stil (Apple Mail) oder Tabellen-Stil? | **Karten** |
| Q7 | Sortierung Default: „heute fällig zuerst" oder „neueste oben"? | **heute fällig zuerst** |
| Q8 | Bucket-Farben (siehe 3.1) — fühlen sich richtig an oder anders? | **prüfen, dann lock** |
| Q9 | Sound bei Eskalation/Notification (Apple-Glocke) — gewünscht oder störend? | **aus** Default, opt-in |
| Q10 | Sprache: Bucket-Namen 1:1 wie Planner („bereit für Rechnung") oder verkürzt für UI („Eingang", „Rechnung", „Kommi", „Versand", „Abhol", „Angem.", „Done")? | **kurz im Stepper, voll im Hover/Tooltip** |

---

## 7. Roadmap fürs Bauen — sobald Variante gewählt

1. **Mockup HTML** (`mockups/design_v1_workspace.html`) — statisch, zum Anschauen + Kommentieren. ✓ kommt mit diesem Commit.
2. **Design-Tokens-CSS** (`design/tokens.css`) — Farben, Type, Spacing, Motion in 1 Datei.
3. **Layout-Skelett** ins `elvinci_auftragssteuerung.html` einziehen — Header + 3-Spalten + Inspector — ohne bestehende Logik anzufassen.
4. **Bestehende sub-Funktionen** in die Center-Workspace-Card umziehen — JTL-Generator, Erfassung, etc. funktioniert weiter.
5. **Liste & Inspector** mit echten Daten verdrahten (S, multi-order später).
6. **Tagescheckliste-Strip** + Sync-Reminder.
7. **Eskalations-Pad** + Bridge-Buttons.
8. **Polish-Pass:** Animationen, Frosted Glass, Reduced-Motion-Branch.

Jeder Schritt = 1 Commit. Jeder Commit testbar.

---

## 8. Anti-Patterns — was wir NICHT machen

- ❌ **Keine bunten Gradienten als Hintergrund.** Apple-Polish kommt aus Ruhe, nicht aus Effekt.
- ❌ **Keine 3D-Schatten / Depth-Stacking** wie auf Spotify. Flach + sanft.
- ❌ **Keine Emoji-Spam.** Pro UI-Element max 1 Symbol — und dann SF Symbol-Stil (mono, einfarbig).
- ❌ **Keine modale Dialog-Wand** für jede Aktion. Inline expand und sheet bevorzugen.
- ❌ **Keine 8 Schriftgrößen.** Nur die Skala oben.
- ❌ **Keine Border-Boxen um alles.** Trennung durch Whitespace und sehr feine Linien.
- ❌ **Keine externe Icon-Library** (Material, Font Awesome, Bootstrap Icons). Pure SVG inline oder System-Symbole. Bibliothek = Bruch des Single-File-Constraints.
