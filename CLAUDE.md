# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projekt

**HEIM.Tone.lab** — mobile-first Tongenerator/Frequenz-Werkzeug für Musiker (primär iPhone, 390×844 pt). Die gesamte App lebt in **einer einzigen, selbständigen Datei: `index.html`** (Inline-CSS + Inline-JS, keine Abhängigkeiten ausser Google Fonts, kein Build-Schritt, kein Framework).

Massgebliche Referenzen im Ordner:
- `HEIM-Tone-lab-2b Design-Spezifikation.md` — verbindliche Design-/Build-Spec (Farb-Tokens, Typografie, Layout, Verhalten). Bei Design-Fragen gilt diese Datei.
- `HEIM-Tone-lab-2b.png` — Referenz-Screenshot des Zielzustands.
- `tmpBild*.png` — Wegwerf-Screenshots aus Nutzer-Feedback, keine Referenzen.

Kommunikation mit dem Nutzer auf Deutsch; angezeigte Zahlen mit deutschem Dezimal-Komma (Helfer `de()` im Code).

## Befehle

Es gibt kein Node, keine Tests, keinen Linter. Arbeitsablauf:

```bash
# App im Browser öffnen
open "index.html"

# JS-Syntaxcheck ohne Node (Inline-Script extrahieren, per JavaScriptCore nur parsen):
sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/app.js
osascript -l JavaScript -e 'ObjC.import("Foundation"); var src = $.NSString.stringWithContentsOfFileEncodingError("/tmp/app.js", $.NSUTF8StringEncoding, null).js; try { new Function(src); "SYNTAX_OK" } catch(e){ "SYNTAX_ERROR: " + e }'

# Für Tests auf dem iPhone im WLAN:
python3 -m http.server 8000
```

Nach jeder Änderung: Syntaxcheck laufen lassen, dann mit `open` neu öffnen.

## Architektur (alles in `index.html`)

**Zustandsmodell — die Frequenz ist die einzige Quelle der Wahrheit.** `state.freq` (Hz) wird von zwei gleichwertigen Eingabewegen gesetzt (bewusste Spec-Entscheidung, „zwei Wege, ein Zustand"):
1. *Tonmodus:* Klaviatur + Oktavwahl setzen exakte Notenfrequenzen (`freqOfMidi`).
2. *Frequenzmodus:* FREQ-Scrollrad (Canvas `#dial`, Ziehen/Mausrad, 3 Cents/Pixel, Doppelklick rastet auf die nächste Note). Bereich `FREQ_MIN` (C0 ≈ 16,35 Hz) bis `FREQ_MAX` (20 000 Hz).

Alles andere wird in `render()` **abgeleitet**: nächste Note (= Anzeige-Tonname und SOLL-Frequenz), Δ in Cents, sowie die Live-Nachführung der Klaviatur (`state.viewOct` blättert automatisch; C/C#/D der Folgeoktave bleiben auf der aktuellen Ansicht sichtbar). Klaviaturtasten setzen die exakte Notenfrequenz — eine per Rad eingestellte Cent-Abweichung geht dabei bewusst verloren.

**Audio (Web Audio API):** Graph `OscillatorNode → gate (GainNode) → masterGain → destination`. Die Klangfarbe ist eine `PeriodicWave` aus der gewichteten Fourier-Summe von Sinus/Dreieck/Sägezahn (`buildWave`, 64 Harmonische); `mixSample()` ist das Zeitbereichs-Pendant für den Trace. Alle hörbaren Übergänge laufen über `setTargetAtTime`-Rampen (kein Knacken); `gate` schaltet bei Mischung 0 % stumm. AudioContext entsteht erst beim ersten START (iOS-Pflicht); Ton kommt grundsätzlich erst auf START, Werte rechnen aber immer sofort.

**Rendering:** Zwei manuell gezeichnete Canvases mit devicePixelRatio-Skalierung — `#trace` (Mischwellenform im Display, animiert nur bei laufendem Ton via rAF) und `#dial` (Skala: Hz-Beschriftung oben, Tonnamen unten, nur Stammtöne). `render()` ist der zentrale Neuaufbau; Ausnahme: `setHotWave()` schaltet die aktive Wellenform-Zeile direkt per Klassen-Toggle um, **ohne** vollen Render.

## Gelernte Stolperfallen (nicht rückgängig machen)

- **Slider nie während eines Drags programmatisch beschreiben.** `render()` setzt `input.value` nur bei tatsächlicher Abweichung; die Hot-Zeilen-Umschaltung läuft absichtlich am vollen Render vorbei. Sonst unterbricht der Browser die native Bedienung scheinbar zufällig.
- **Layout-Höhenbudget:** Der Inhalt muss auf 844 pt passen. `.content > *` hat `flex-shrink:0` (Panels dürfen nie zusammengequetscht werden — das hatte das Mess-Display unlesbar gemacht); bei Überlauf scrollt `.content`. Wer einen Block hinzufügt, muss an anderer Stelle Höhe einsparen.
- **Akzentfarbe nur über `--ac`** und `color-mix`-Ableitungen — keine hartkodierten `#FFB020` streuen (Spec-Regel, Farbe muss zentral austauschbar bleiben).

## Konventionen

- Versionsnummer im Header (`v0.x · REC`) bei funktionalen Ausbauschritten hochzählen.
- Touch-Ziele ≥ 44 px; iPhone-Safe-Areas (`env(safe-area-inset-*)`) beachten; `#dial` braucht `touch-action:none`.
- Roadmap läuft in Phasen; Phase 2 („Bedienung": Klaviatur, Frequenzmodus, Live-Nachführung) ist umgesetzt. Die Roadmap selbst liegt beim Nutzer, nicht im Ordner — bei Unklarheit zum Umfang nachfragen.
