# HEIM.Tone.lab — Design- & Build-Spezifikation (Variante 2b)

> Referenz-Screenshot: `HEIM-Tone-lab-2b.png` (390×844, @2x = 780×1688).
> Diese Datei + Screenshot zusammen in den Claude-Code-Prompt geben.

## Zielbild
Mobile-first Web-App (primär iPhone, ~390×844 pt) — ein **Tongenerator / Frequenz-Werkzeug** für Musiker.
Ästhetik: **hochwertiges digitales Messinstrument** (Tuner / Studio-/Oszilloskop-Gerät), Dark Mode,
hoher Kontrast, reduziert, Klarheit vor Funktionsfülle. Kein verspieltes Consumer-App-Gefühl.
Charakter: technisch, monospace-lastig, Oszilloskop-Raster.

## Design-Tokens

### Farben
| Rolle | Hex |
|---|---|
| Akzent (primär) | `#FFB020` (Bernstein) |
| Akzent dunkel (Verläufe) | `color-mix(#FFB020 70%, #000)` ≈ `#B37A16` |
| App-Hintergrund | `#0A0C09` |
| Panel-Hintergrund | `#0D110C` |
| Rahmen / Divider | `#223021` |
| Oszilloskop-Rasterlinien | `#182417` |
| Text primär | `#E6F0E2` |
| Text sekundär / Labels | `#7C8C77` |
| Text disabled (Oktave 0/7/8) | `#556052` |
| Text auf Akzent (Buttons/aktive Taste) | `#0A0C09` |
| Weisse Tasten | Verlauf `#DDE7D8 → #B9C7B3`, Beschriftung `#2A3327` |
| Schwarze Tasten | Verlauf `#1A2418 → #050705`, Rand `#000` |
| Inaktive Slider-Füllung (Dreieck/Sägezahn) | `#7C8C77` auf Track `#182417` |

Akzent wird an EINER Stelle als CSS-Variable gesetzt (`--ac`); alle Akzent-Flächen leiten
Glow/Verläufe per `color-mix(in srgb, var(--ac) X%, …)` ab → Farbe zentral austauschbar.

### Typografie (Google Fonts)
- **Chivo Mono** — alle Zahlen, Mess-/Statuswerte, Labels, Tasten- und Wellenform-Beschriftung, Buttons. Gewichte 400/500/700.
- **Chivo** (Sans) — Grundtext / neutrale Beschriftung. Gewichte 300–700.
- Zahlen immer `font-variant-numeric: tabular-nums` bzw. Mono → keine springenden Ziffern.

### Form & Abstand
- Rundung: **eher scharf-technisch**. Panels/Display r=`12px`, Wellenform-/Master-Zeilen r=`6px`, Oktav-Zellen r=`3px`, STOP-Button r=`10px`. Gerätrahmen r=`44px`.
- Grid-Gap vertikal zwischen Blöcken: `14px`. Content-Padding: `8px 20px 20px`.
- Akzent-Glows durchweg dezent: `box-shadow` mit 12–55 % Akzent-Alpha.

## Layout (von oben nach unten)

1. **Statusleiste** — Uhrzeit links (`9:41`), Signal/Batterie rechts. Mono, `#7C8C77`.
2. **Kopfzeile** — `HEIM.TONE.LAB` (Mono 13px, letter-spacing 2px, Akzentfarbe, bold) links; `v0.3 · REC` rechts (`#7C8C77`). Untere 1px-Divider-Linie.
3. **Mess-Display (Oszilloskop-Panel)** — Panel `#0D110C`, Rahmen `#223021`, mit gekacheltem Rasterhintergrund (28×28px, `#182417`, opacity .6). Enthält die **vier synchronen Werte**:
   - Aktueller Ton **A3** — Mono 62px bold, Akzentfarbe, Glow.
   - Aktuelle Frequenz **220 Hz** — Mono 34px bold, Akzent, „Hz" 16px.
   - Dazwischen: **Wellenlinien-Trace** (SVG-Sinus, Akzent, opacity .85) — im echten Build durch live gerenderte Mischwellenform ersetzen.
   - Fusszeile im Panel: **SOLL 220,00 Hz** (weiss) links; **Δ +3 ct** (Akzent) rechts. Deutsche Dezimal-Kommas.
4. **Oktav-Auswahl** — Label `OCT` (Mono, `#7C8C77`) + 9 gleich breite Zellen **0–8** in einer Reihe. Aktive Oktave (`3`) = Akzent-Fläche, dunkler Text, bold. 0/7/8 dezent (`#556052`). Inaktive: Rahmen `#223021`, Text `#7C8C77`.
5. **Klaviatur** — Pianolayout, ~1,5 Oktaven **C3–D4** (9 weisse Tasten: C3 D3 E3 F3 G3 A3 B3 C4 D4). Schwarze Tasten als absolut positioniertes Overlay über den weissen (Positionen bei 11.11/22.22/44.44/55.55/66.66/88.88 % der Breite, Breite 24px). Weisse Tasten: heller Verlauf, Notenlabel unten (Mono 9px). **Aktive Taste A3** = Akzent-Verlauf, dunkles Label, Glow. Internationale Notation.
6. **Wellenform-Wähler (3 Zeilen)** — je Zeile: SVG-Glyph (Sinus-Kurve / Dreieck / Sägezahn) + Name (Mono, `SINUS`/`DREIECK`/`SÄGEZAHN`) + horizontaler **Mix-Regler** + **%-Wert** rechts. Aktive/gewählte Zeile (Sinus) hat Akzent-Rahmen, Akzent-Füllung & Akzent-%; inaktive grau. Beispielwerte 60 % / 30 % / 10 % (Summe frei, nicht zwingend 100 %).
7. **Master-Lautstärke** — Label `MASTER` + horizontaler Regler (Akzent-Füllung, heller quadratischer Knopf `#E6F0E2`) + %-Wert (Akzent). Beispiel 75 %.
8. **Start/Stop-Button** — grosser, gut erreichbarer Button am unteren Rand (`margin-top:auto`). Zustand STOP: transparentes Panel mit 2px Akzent-Rahmen, Akzent-Text `■ STOP` (Mono 17px, letter-spacing 4px), dezenter innerer + äusserer Akzent-Glow.

## Funktionales Verhalten (für den echten Build)
Das Mockup ist statisch — die reale App soll:
- **Web Audio API**: `OscillatorNode`(s) bzw. `PeriodicWave` aus der gewichteten Summe von Sinus/Dreieck/Sägezahn erzeugen; Master-Gain über `GainNode`.
- **Start/Stop** togglet die Klangausgabe; Zustand STOP↔START (Text & Icon wechseln, `■`↔`▶`).
- **Tastatur / Oktave** bestimmen die Grundfrequenz; gleichstufige Stimmung, `A4 = 440 Hz` (konfigurierbar). Ausgewählte Taste + Oktave setzen Ton, Ist- und Sollfrequenz.
- **Vier Werte synchron aktualisieren**: Ton-Name, Ist-Frequenz (Hz), Sollfrequenz (2 Nachkommastellen, Komma), Cent-Abweichung (`±N ct`, gerundet, Vorzeichen). Cent-Meter/Farbe kann bei grosser Abweichung visuell reagieren.
- **Wellenform-Regler** verändern die Mischung live; %-Werte spiegeln die Gewichte.
- **Master-Regler** steuert Ausgangspegel (0–100 %).
- **Wellen-Trace** im Display idealerweise als Live-Visualisierung (AnalyserNode oder berechnete Kurve).
- Touch-Ziele ≥ 44px; iPhone-Safe-Areas berücksichtigen.

## Wichtige Regeln
- Akzent zentral über `--ac` steuern (keine hartkodierten `#FFB020`-Streuungen).
- Deutsche Zahlenformatierung (Dezimal-Komma) für angezeigte Frequenzen.
- Kein zusätzliches UI-Rauschen; Ruhe im unteren Bereich ist bewusst.
