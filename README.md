# Akustische Messkette — Modellmesstechnik 1:20

Interaktive Visualisierung der akustischen Messkette zur Bestimmung der Raumimpulsantwort (RIR) eines Modells des Dresdner Kulturpalasts im Maßstab 1:20.

**[→ Live-Demo](https://andrejzamsev.github.io/messkette-visualisierung/)**

---

## Zweck der Visualisierung

Die Visualisierung zeigt den vollständigen Signalfluss einer raumakustischen Messung in einem Architekturmodell. Ziel ist die Bestimmung der **Raumimpulsantwort (RIR)** und der daraus abgeleiteten **Nachhallzeit T₆₀** in Terzbändern.

Der Maßstab 1:20 erfordert eine Frequenzskalierung um den Faktor 20: Alle akustischen Frequenzen liegen im **Ultraschallbereich** (bis 126 kHz), weshalb spezielle Messtechnik eingesetzt wird.

Die Animation verdeutlicht:
- Die **Signaltransformation** von digital → analog → akustisch → analog → digital
- Den **Loopback-Referenzpfad** zur Zeitkorrektur
- Die **Farbcodierung** der Signaltypen (Digital, Analog, Akustisch, Loopback)

---

## Beschreibung der einzelnen Blöcke

### 1. MATLAB — Signalerzeugung
- **Funktion:** Erzeugt ein digitales Anregungssignal – einen logarithmischen Sinus-Sweep (ESS, Exponential Sine Sweep)
- **Frequenzbereich:** f_u = 559 Hz bis f_o = 126.491 Hz (Modellmaßstab)
- **Sweep-Dauer:** T_w = 0,5 s
- **Besonderheiten:**
  - Optional wird ein Linearisierungsfilter im Frequenzbereich angewendet, um den Frequenzgang des Gesamtsystems zu kompensieren
  - Das Signal wird mit 100 ms Startverzögerung und Ausschwingzeit am Ende versehen
  - Abtastrate: f_s = 500 kHz

### 2. NI-9262 — Digital-Analog-Wandler (DAC)
- **Funktion:** Wandelt das digitale Signal aus MATLAB in ein analoges elektrisches Signal
- **Hardware:** Steckmodul in Slot 3 des NI cDAQ-9174 Chassis
- **Auflösung:** 16 Bit (65.536 Quantisierungsstufen)
- **Abtastrate:** f_s = 500 kHz (erfüllt Nyquist: f_s > 2 · f_o)

### 3. BNC-T-Stück — Signalsplitter
- **Funktion:** Passiver Signalteiler, der das analoge Signal in zwei Pfade aufteilt
- **Pfad A:** Hauptpfad → weiter zum Verstärker und Lautsprecher
- **Pfad B:** Loopback-Referenz → direkt zurück zum ADC (NI-9222, Kanal ai3)
- **Zweck des Loopback:** Ermöglicht die exakte Zeitversatzkorrektur zwischen DAC und ADC sowie eine Qualitätskontrolle des Ausgangssignals

### 4. B&K 2706 — Leistungsverstärker
- **Funktion:** Verstärkt das elektrische Signal auf die benötigte Leistung für den Lautsprecher
- **Hersteller:** Brüel & Kjær
- **Einstellung:** Filter auf „LIN" (lineare Verstärkung, keine Filterung)

### 5. Mini-Dodekaeder — Schallquelle
- **Funktion:** Wandelt das elektrische Signal in Schallwellen um
- **Bauform:** 3D-gedrucktes Gehäuse mit 12 Lautsprecherflächen
- **Abstrahlcharakteristik:** Omnidirektional (Kugelcharakteristik) — strahlt in alle Raumrichtungen gleichmäßig ab
- **Maßstab:** 1:20, passend zum Architekturmodell

### 6. Modellraum — Akustisches Medium
- **Funktion:** Der Schall durchläuft das physische Architekturmodell des Kulturpalasts Dresden
- **Akustische Phänomene:**
  - Reflexionen an Wänden, Decke und Einrichtung
  - Absorption durch Materialien und Oberflächen
  - Streuung an geometrischen Details
  - Luftdämpfung — besonders stark im Ultraschallbereich
- **Ergebnis:** Im Raum entsteht die Raumimpulsantwort (RIR), die alle akustischen Eigenschaften des Raumes kodiert

### 7. GRAS 46DE — Druckmikrofon
- **Funktion:** Wandelt die Schallwellen zurück in ein elektrisches Signal
- **Typ:** ⅛-Zoll Druckmikrofon
- **Frequenzbereich:** 6,5 Hz – 140 kHz (±3 dB)
- **Abstrahlcharakteristik:** Kugelcharakteristik (omnidirektional empfindlich)
- **Besonderheit:** Ultraschall-tauglich für den Modellmaßstab

### 8. B&K 1708 — Mikrofonvorverstärker
- **Funktion:** Verstärkt das schwache Mikrofonsignal auf einen verwertbaren Pegel
- **Hersteller:** Brüel & Kjær
- **Versorgung:** ICP/CCLD (Konstantstromversorgung für das Mikrofon)
- **Einstellung:** Filter auf „LIN", Verstärkung so hoch wie möglich ohne Übersteuerung (Overload-LED darf nicht leuchten)

### 9. NI-9222 — Analog-Digital-Wandler (ADC)
- **Funktion:** Digitalisiert die analogen Signale für die Verarbeitung in MATLAB
- **Hardware:** Steckmodul in Slot 1 des NI cDAQ-9174 Chassis
- **Kanäle:**
  - **ai0:** Mikrofonsignal (Messsignal)
  - **ai3:** Loopback-Signal (Referenz)
- **Abtastrate:** f_s = 500 kHz

### 10. MATLAB — Signalverarbeitung
- **Funktion:** Verarbeitet die aufgenommenen Signale zur Bestimmung der Raumimpulsantwort und Nachhallzeit
- **Verarbeitungsschritte:**
  1. **Zeitkorrektur:** Peak-Erkennung im Loopback-Signal zur Synchronisation
  2. **Mittelung:** N_avg = 4 Messwiederholungen zur Rauschunterdrückung
  3. **Kreuzkorrelation:** Mikrofonsignal ↔ Loopback-Signal ergibt die RIR
  4. **Terzbandanalyse:** Aufteilung der RIR in Frequenzbänder
  5. **Schröder-Rückwärtsintegration:** Berechnung der Energieabklingkurve
  6. **Nachhallzeit T₆₀:** Bestimmung pro Terzband aus der Abklingkurve

---

## Loopback-Pfad

Der Loopback ist ein separater Referenzpfad, der **direkt** vom BNC-T-Stück (Block 3) zurück zum NI-9222 ADC (Block 9, Kanal ai3) führt — ohne den akustischen Pfad zu durchlaufen.

**Zweck:**
- Exakte **Zeitversatzkorrektur** zwischen DAC-Ausgabe und ADC-Aufnahme
- **Qualitätskontrolle** des ausgegebenen Signals (Verifikation der Signalintegrität)

---

## Dargestellte Formeln

| Formel | Beschreibung |
|--------|-------------|
| Sweep-Signal s(t) | Logarithmischer Sinus-Sweep mit exponentiell ansteigender Frequenz |
| Nyquist-Theorem | Nachweis, dass die Abtastrate ausreichend ist: f_s > 2·f_o |
| Kreuzkorrelation | Berechnung der RIR aus Mikrofon- und Loopback-Signal |
| Sabine-Formel | Zusammenhang zwischen Nachhallzeit, Raumvolumen und äquivalenter Absorptionsfläche |

---

## Frequenzbereichsdarstellung (Terzbandanalyse)

Der Balkendiagramm unten rechts zeigt die **Nachhallzeit T₆₀ aufgeschlüsselt nach Terzbändern**. Die obere Frequenzachse zeigt die Modell-Frequenzen (500 Hz – 10 kHz), die untere in Klammern die korrespondierenden Real-Frequenzen (25 Hz – 500 Hz) nach Rückskalierung mit Faktor 1:20.

Typisch für den Kulturpalast: höhere Nachhallzeiten bei tiefen Frequenzen, abfallend zu hohen Frequenzen durch stärkere Luftabsorption.

---

## Interaktivität

| Funktion | Beschreibung |
|----------|-------------|
| **Hover** | Bewegen Sie die Maus über eine Komponente für detaillierte Spezifikationen |
| **Play/Pause** | Startet oder stoppt die Signalfluss-Animation |
| **Geschwindigkeit** | Schaltet zwischen 0.5×, 1×, 2× und 3× um |
| **Reset** | Setzt die Animation auf den Anfangszustand zurück |

---

## Technologie

- Reines **HTML + CSS + JavaScript** (kein Framework)
- **Canvas API** für Partikel-Animationen und Signalpfade
- **KaTeX** für mathematische Formeln in LaTeX-Qualität
- **Google Fonts**: Space Grotesk (UI), JetBrains Mono (Formeln/Code)
- Optimiert für **1920×1080** (Präsentationsformat)
- Responsive Skalierung für kleinere Bildschirme

---

## Lokale Entwicklung

```bash
# Einfach im Browser öffnen:
open index.html

# Oder mit einem lokalen Server:
python3 -m http.server 8080
# → http://localhost:8080
```

## Lizenz

Erstellt für akademische Zwecke — Technische Universität Dresden, Modellmesstechnik.
