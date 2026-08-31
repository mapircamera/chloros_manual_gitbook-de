# Chloros CLI Referenz

**Version:**

1.2.0**Erstellt:**

29.07.2026, 19:19 Uhr ·**Überarbeitet:**

30.08.2026**Zielgruppe:** Optimiert für die Verwendung durch LLMs; für Menschen lesbar.**Geltungsbereich:** Alle benutzerspezifischen Unterbefehle von `chloros-cli`, einschließlich Optionen und-Einfügebeispiele.

Dieses Dokument ist die vollständige Referenz für das Befehlszeilentool `chloros-cli`, das im Lieferumfang von MAPIR Chloros enthalten ist. Es ist bewusst umfassend gestaltet, damit ein LLM (oder ein Mensch) jeden unterstützten Workflow aus den folgenden Auflistungen zusammenstellen kann, ohne den Quellcode einsehen zu müssen.

Wenn Sie nur die wichtigsten Punkte benötigen, springen Sie zu:
- [Fünf-Minuten-Schnellstart](#five-minute-quickstart)
- [Workflow für die Erstverbindung mit einer LATTICE-Kamera](#lattice-camera-first-connect-workflow)
- [Workflow für die Erstverbindung mit einem DAQ-Sensor](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Aufnahmemodi, Recorder und Offline-Nachbearbeitung](#capture-modes-recorders--offline-reprocess)

---

## Konventionen

- Alle Befehle beginnen mit dem Präfix `chloros-cli`. Bei Windows lautet die Binärdatei `chloros-cli.exe`; bei Linux/Jetson lautet sie `chloros-cli`.
- Optionale Argumente werden als `--flag` dargestellt. Erforderliche Positionsargumente werden ohne Klammern dargestellt.
- Wenn ein Standardwert angegeben ist, wird bei Weglassen des Flags dieser Wert verwendet.
- CLI ist ein „Thin“-Client über das Chloros-Backend (Flask-Server auf `127.0.0.1:5000`). Das Backend wird von den meisten Befehlen automatisch gestartet. `CHLOROS_BACKEND_URL=<url>` verweist auf **`lattice`**,**`project`**sowie**`daq pool-*`** auf ein entferntes Backend – die Kernbefehle (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fixieren bewusst `http://127.0.0.1:<port>` und ignorieren es (das IPv4-Literal vermeidet Windows‘ `localhost`→`::1` ~2 s proAnfrage). Siehe [Umgebungsvariablen](#environment-variables).
- Für alle SDK-/CLI-Aufrufe ist eine Anmeldung mit einem Chloros+-Konto erforderlich (`chloros-cli login` einmal pro Rechner ausführen; wird in `~/.chloros/` zwischengespeichert).
- Die Beispiele verwenden Linux-Pfade; ersetzen Sie auf Windows `/home/user/...` durch `C:/Users/.../...` ersetzen.

---

## Übersicht auf oberster Ebene

```
chloros-cli [global options] COMMAND [command options]
```

### Globale Optionen

| Flag | Beschreibung |
| --- | --- |
| `--backend-exe PATH` | Die automatisch erkannte Backend-Ausführungsdatei überschreiben. |
| `--port N` | Port des Backends HTTP (Standard: `5000`). |
| `-v, --verbose` | Ausführliche Ausgabe aktivieren. |
| `--restart` | Erzwingt einen Neustart des Backends (beendet alle laufenden `backend_server.py`-Prozesse). |
| `--version` | Version anzeigen (`Chloros CLI 1.2.0`) anzeigen. |
| `--help` | Hilfe der obersten Ebene anzeigen. |

### Befehlsverzeichnis

| Befehl | Zweck |
| --- | --- |
| [`process`](#chloros-cli-process) | Einen Ordner mit Survey3- oder LATTICE-Aufzeichnungen durchgängig verarbeiten. |
| [`login`](#chloros-cli-login) | Authentifizierung dieses Rechners mit einem Chloros+-Konto. |
| [`logout`](#chloros-cli-logout) | Zwischengespeicherte Anmeldedaten löschen. |
| [`status`](#chloros-cli-status) | Aktuellen Lizenz-/Authentifizierungsstatus anzeigen. |
| [`export-status`](#chloros-cli-export-status) | Live-Anzeige des Thread-4-Exportfortschritts während eines `process`-Laufs. |
| [`language`](#chloros-cli-language) | CLI-Anzeigesprache festlegen oder auflisten (38 unterstützt). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#Projektordner-Befehle) | Standard-Projektordner (gemeinsam mit der GUI genutzt). |
| [`update`](#chloros-cli-update) | Nach CLI-Updates suchen und diese installieren (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Systemdiagnose + Funktionstests. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-Grandmaster-Status / -Steuerung. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-Kamerasteuerung und -Aufnahme (über 45 Unterbefehle). |
| [`daq`](#chloros-cli-daq) | Steuerung der DAQ-Spektralsensoren (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Ein gespeichertes Chloros-Projekt (Kameras + DAQs) öffnen und ausführen. |

---

## Installation

`chloros-cli` ist auf jeder unterstützten Plattform im Chloros-Desktop-Installationsprogramm enthalten – es gibt keinen separaten Download für CLI. Durch die Installation des Plattformpakets wird `chloros-cli` neben der Desktop-App und der von ihr gesteuerten Backend-Binärdatei zu Ihrem `PATH` hinzugefügt.

Aktuelle Downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Das Installationsprogramm enthält außerdem praktische Startskripte (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), die eine einsatzbereite CLI-Shell öffnen; diese werden im [CLI-Benutzerhandbuch Handbuch](../CLI.md) behandelt und werden hier nicht erneut beschrieben.

### Windows (.exe)

1. Laden Sie das Installationsprogramm Windows von der Download-Seite herunter.
2. Führen Sie `Chloros-Setup-x.y.z.exe` aus und folgen Sie den Anweisungen des Assistenten. Der Standardinstallationspfad ist `C:\Program Files\Chloros\` (die Datei CLI wird in `C:\Program Files\Chloros\cli\` installiert, das der Installer zur PATH-Variablen hinzufügt).
3. Öffnen Sie ein neues Terminal (`cmd.exe`, PowerShell oder oder Windows Terminal), damit das aktualisierte `PATH` erkannt wird.

```powershell
chloros-cli --version
```

Das Installationsprogramm fügt `chloros-cli.exe` automatisch zu Ihr System „`PATH`“ hinzu und bündelt die für LATTICE-Kameras erforderliche Arena-Laufzeitumgebung „SDK“.

### Linux amd64 (.deb)

Für Ubuntu 22.04 LTS oder neuer / Debian-basierte x86_64-Workstations.

> **Ubuntu 20.04 wird nicht unterstützt.** Die Abhängigkeitsliste des Pakets leitet sich aus
> den tatsächlichen Verknüpfungen des Backends ab, darunter auch `libc6 (>= 2.34)`;
> Focal liefert glibc 2.31 aus. `apt` verweigert die Installation, anstatt sie zur
> Laufzeit scheitern zu lassen.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Das .deb-Paket installiert:
- `chloros-cli` bis `/usr/bin/chloros-cli`
- Das kompilierte Backend auf `/usr/lib/chloros/chloros-backend`
- Die Arena-Laufzeitumgebung SDK (für LATTICE-Kameras)
- Denoiser-Modelle, Kalibrierungs-Bundles und die Konfiguration des Update-Kanals

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Gleicher Aufbau wie das amd64-.deb, mit einem für Jetson Orin / Orin NX / Orin Nano optimierten CUDA-Build.

### Einmalige Authentifizierung pro Gerät

Jede Plattform erfordert eine einmalige Anmeldung bei Chloros+, bevor SDK/CLI-Aufrufe funktionieren:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Die Anmeldedaten werden in `~/.chloros/user_session.json` zwischengespeichert.

### Installation überprüfen

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Chloros+-Abonnement erforderlich.**Für CLI ist ein aktiver Chloros+-Tarif erforderlich.**Copper**ist die Einstiegsstufe Chloros+-Stufe – jede kostenpflichtige Chloros+-Stufe verfügt über CLI/SDK-Zugang; nur die kostenlose Stufe**Iron** bietet diesen nicht. (Zuordnung der Tarif-IDs: `0`=Iron/kostenlos, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Upgrade unter [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing) durchführen.
>
> Diese Untergrenze wird vom Backend durchgesetzt, nicht nur durch den CLI: Eine mit SDK/CLI-gekennzeichnete Anfrage ohne kostenpflichtigen Tarif wird mit `403 PLAN_UPGRADE_REQUIRED` abgelehnt, unabhängig davon, ob sie von `chloros-cli`, dem Python SDK oder von einem selbst erstellten HTTP-Client stammt. Ein abgemeldeter Aufrufer erhält stattdessen den Fehlercode `401 AUTH_REQUIRED`. Der Zugriff funktioniert offline während der Nachfrist des Tarifs (30 Tage monatlich, bis zum Ablauf bei Jahresabonnements) und endet, sobald diese abgelaufen ist; `chloros-cli status` funktioniert weiterhin, damit der Grund sichtbar bleibt (es handelt sich um die einzige Route SDK/CLI, die von der Stufenbeschränkung ausgenommen ist – `GET /api/license-status`).

---

## Fünf-Minuten-Schnellstart

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Verarbeiten Sie einen Ordner mit Bildern durch die vollständige Chloros-Pipeline (Zielerkennung → Kalibrierung → Vignette → Reflexionsgrad → Index-Export).

### Übersicht

```
chloros-cli process INPUT [OPTIONS]
```

### Positionsargumente

| Argument | Beschreibung |
| --- | --- |
| `INPUT` | Pfad zum Eingabeordner, der `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) oder `.dng`-Dateien enthält. |

### Allgemeine Optionen

| Option | Standard | Beschreibung |
| --- | --- | --- |
| `-o, --output PATH` | Ein neuer Ordner mit Zeitstempel unter Ihrem Standard-Projektpfad (`~/Chloros Projects`, sofern nicht anders konfiguriert) | Projektordner, der erstellt oder wiederverwendet werden soll. Befindet sich in dem Ordner bereits eine Datei „`project.json`“, wird anstelle eines Überschreibens ein Ordner „`_1`“/`_2`-Unterordner erstellt, anstatt den bestehenden zu überschreiben. |
| `-n, --project-name NAME` | automatisch (Zeitstempel) | Projektname. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` verwendet einen neuronalen Debayer vom Typ Chloros+; langsamer, aber höhere Qualität. |
| `--vignette / --no-vignette` | `--vignette` | Vignettierungskorrektur. |
| `--reflectance / --no-reflectance` | `--reflectance` | Reflektanzkalibrierung (verwendet Panel-Target, falls vorhanden, NIST-Kalibrierung pro Seriennummer für LATTICE). Bei LATTICE-Multispektralbildern dient dies gleichzeitig als Umschalter für das **Produkt** der Reflexionskalibrierung – siehe [Umschalter für den produktbezogenen Export](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | aus | PPK-GNSS-Korrekturen aus Sidecar-Dateien anwenden. |
| `--exposure-pin-1 MODEL` | aus | Das „pin-1“-Modell eines `--format FORMAT`-Doppelkamera-Rigs fixieren. |
| `--exposure-pin-2 MODEL` | aus | Das „pin-2“-Modell fixieren. |
| `--recal-interval SECONDS` | 0 | Erzwinge die erneute Ausführung der Kalibrierungsberechnungen alle N Sekunden der Aufnahmzeit. |
| `--timezone-offset HOURS` | local | Überschreibe den in den Ausgabemetadaten festgelegten Zeitzonen-Offset. |
| `--format FORMAT` | `TIFF (16-bit)` | Einer der folgenden Werte: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | keine | Vegetationsindizes (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Erzwingt den Pipeline-Einstiegspunkt für LATTICE-TIFFs (Survey3 .raw bleibt davon unberührt). Ebenso die Ausweichmöglichkeit, die es ermöglicht, eine Aufnahme **ohne Raw-Daten** überhaupt zu verarbeiten – siehe [So sieht ein Captures-Ordner aus](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | ein | Das linear debayerte Ergebnis ausgeben (`Debayered_Images`). Siehe [Export-Schalter pro Produkt](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | ein | Die Anzeigevorschau ausgeben (`Preview_Images`): RGB = Weißabgleich (DAQ-Lichtart, falls verfügbar, andernfalls „Gray-World“) + Gamma; Multispektral = Falschfarben-Streckung. |
| `--radiance / --no-radiance` | ein | Strahlungsintensität als float32 ausgeben (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referenz für das LATTICE-Reflexionsprodukt: `auto` = QA-konformes In-Frame-Ziel ist die absolute Referenz, DAQ-Downwelling (ρ = π·L/E) als Ausweichlösung; `target` = streng (keine DAQ-Ersetzung); `daq` = DAQ-maßgeblich. Siehe [Export-Schaltflächen pro Produkt](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | keine | Verzeichnis der **gemessenen** Zielreflexions-Scans pro Einheit (`<serial>.csv`); bei Fehlschlag wird auf die nominalen T3/T4P-Spektren zurückgegriffen. |
| `--array-alignment / --no-array-alignment` | on | LATTICE-Arrays: Wendet die in der `Chloros:Alignment*`-XMP-Datei jeder Aufnahme vermerkte Modul-zu-Modul-Ausrichtung auf jedes verarbeitete Produkt an (Debayering / Vorschau / Radianz / Reflektanz / Index). Keine Aktion für Bilder ohne diese Tags. |
| `--array-alignment-crop / --no-array-alignment-crop` | crop | Ausgerichtete Exporte auf den gemeinsamen Überlappungsbereich des Arrays zuschneiden, sodass alle Module denselben Fußabdruck haben; `--no-…` behält die volle Sensorfläche bei (außerhalb der Quelle mit Schwarz gefüllt). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Neuberechnung der Abmessungen für die Ausrichtungsverzerrung. `nearest` bewahrt die exakten Quell-DNs (keine Vermischung radiometrischer Werte zwischen den Pixeln). |

### Optionen zur Zielerkennung

| Flag | Beschreibung |
| --- | --- |
| `--min-target-size PIXELS` | Minimale Panel-Zielgröße (px) für den Detektor. |
| `--target-clustering 0-100` | Clustering-Empfindlichkeit. |
| `--target / --targets` | Den Eingabeordner als reines Ziel-Panel behandeln (Erkennung von Umfragen überspringen). |

### Beispiele

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Export-Schalter pro Produkt (LATTICE Multispektral)

Die LATTICE-Verarbeitung verteilt sich auf **jedes anwendbare Produkt in einem Durchgang**auf. Die vier Schalter pro Typ – `--debayered`, `--preview`, `--radiance`, `--reflectance` — sind alle**standardmäßig aktiviert**; verwenden Sie die Form `--no-<type>`, um eine davon zu deaktivieren. Die Master-Kameras RGB geben immer nur debayerte Daten + Vorschau aus (keine Strahlungsintensität/Reflexionsgrad pro Band), daher sind `--radiance`/`--reflectance` für sie ohne Auswirkung. Die Umschaltoptionen werden für Survey3 und `.raw` (die dem Standardpfad für Reflektanz/Ziel folgen). *(Das alte `--radiometric-output {reflectance,radiance,sensor-response}`-Flag wurde **entfernt** und durch diese Schaltoptionen ersetzt; es gibt keine `sensor-response`-Stufe mehr.)*

| Produkt | Ausgabe | DAQ-Downwelling erforderlich? |
| --- | --- | --- |
| `--debayered` | Lineares Demosaicing (`Debayered_Images`). | Nein. |
| `--preview` | Vorschau anzeigen (`Preview_Images`): RGB = Weißabgleich + Gamma; Multispektral = Falschfarben-Streckung. | Nein. |
| `--radiance` | float32 W/m²/sr/nm aus der vollständigen radiometrischen Kette (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 Reflexionsgrad ρ (`32768` = 1,0), Pix4D-kompatibel. | **Ja**, es sei denn, ein QA-konformes In-Frame-Ziel verankert es (siehe unten). |

`--reflectance-source` wählt die Reflexionsreferenz aus:**`auto`**(Standard) macht ein QA-konformes Ziel innerhalb des Bildausschnitts zur**absoluten Referenz**– die zielverankerten empirischen Linienketten werden auf zurückgehaltenen Panels kreuzvermisst und der gemessene Gewinner angewendet –, wobei auf die DAQ-Downwelling-Trennlinie (ρ = π·L/E) zurückgegriffen wird, wenn kein Ziel vorhanden ist oder die QA fehlschlägt;**`target`**ist streng (keine DAQ-Ersetzung);**`daq`**wählt das DAQ-maßgebliche Verhalten. Die Zielgeometrie (ArUco / fester ROI / Streifen) stammt aus der Projektzielkonfiguration; `--target-reflectance-dir DIR` speichert die**gemessenen** Scans (`<serial>.csv`) pro Einheit, die anhand der Seriennummer/des QR-Codes der Ziel-Einheit nachgeschlagen werden, wobei die nominalen T3/T4P-Spektren als Fallback dienen.

Der DAQ-Reflexionspfad ermittelt die **zeitstempelabgeglichene Downwelling-Daten**automatisch aus einer aufgezeichneten**`.daq`**(DAQ-U/M/E)**oder einem nativen DAQ-M-`.csv`**, der zusammen mit den Bilddaten vorliegt. Wenn kein kameraspezifisches oder DAQ-Kalibrierungspaket lokal zwischengespeichert ist,**bei der ersten Verwendung automatisch von AWS** (erfordert einmalig eine Internetverbindung; wird unter `~/.chloros/` zwischengespeichert).

#### Auslesen von Reflexionspixelwerten (Pix4D / Metashape / eigene Skripte)

Die Reflexion wird als ganzzahliger DN-Wert gespeichert, und **der DN-Wert, der ρ = 1,0 entspricht, hängt von der Quellkamera ab**:

| Quelle | ρ = 1,0 entspricht | Erkennungsmerkmal |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (Spielraum bis ρ 2,0) | XMP `Chloros:PixelScale=32768` ist in die Datei eingestempelt. |
| Survey3 | `65535` (abgeschnitten bei ρ 1,0) | Keine `Chloros:*`-XMP-Tags – dieses Fehlen *ist* das Signal. |

**Lies den Wert `Chloros:PixelScale` ein und dividiere damit**, anstatt von einer Konstante auszugehen. Das Tag ist im uint16-Bereich definiert, daher bleibt es auch bei Ausgabeformaten, die eine Neuskalierung vornehmen – `32768`, `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` und `TIFF (32-bit, Percent)` sind alle selbsterklärend (normalisieren Sie den gespeicherten Datentyp zunächst wieder auf uint16: ×257 von 8-Bit, ×65535 von Float).

> **Ein Fall enthält keine Skalierung, was beabsichtigt ist.** Wenn eine 8-Bit-Quellaufnahme (BayerRG8) als 8-Bit-TIFF geschrieben wird, *beschneidet* die Pipeline den Wert auf 0..255, anstatt eine Neuskalierung vorzunehmen; daher wird jeder Wert über ρ ≈ 0,008 auf 255 abgeflacht, und die Datei enthält keine Skalierung. Chloros lässt dort bewusst sowohl das Tupel `Chloros:PixelScale` als auch das Tupel `MicaSense:RadiometricCalibration`-Tupel dort bewusst aus und protokolliert den Grund dafür. **Fehlt das Tag in einer LATTICE-Reflexionsdatei, darf keine Skalierung angenommen werden – führen Sie einen erneuten Export mit 16-Bit oder 32-Bit durch**, anstatt Pixel zu teilen, die nie teilbar waren.

#### EXIF-Daten werden beim Export übernommen

`process` kopiert den **GPS-Block und dessen ExifIFD** der Quellaufnahme in jedes Produkt, sodass ein
Export neben der Georeferenzierung auch `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` sowie
`CameraSerialNumber` neben den Georeferenzierungsdaten.

**`FocalLength` ist für die Photogrammetrie unverzichtbar.** Pix4D berechnet den Ground Sample Distance (GSD) aus
Brennweite und Höhe; fehlt das Tag, greift das Programm auf einen völlig falschen Maßstab zurück. Bei einem
Flug über einen Orangenhain mit 49 Aufnahmen verwandelte das fehlende Tag ein 411 m × 160 m großes Gelände in ein rekonstruiertes
47,8 km × 13 km – ein 455-MP-Orthobild mit überwiegend fehlenden Daten, was zunächst als Kachelungsproblem und
als BigTIFF-Problem interpretiert wurde, bevor jemand die GSD überprüft hatte. Wenn Ihr Orthofoto einen unplausiblen
Maßstab aufweist, führen Sie zunächst `exiftool -FocalLength` auf das exportierte Produkt an.

Die Kopie ist bewusst **nicht** `-all:all`: Die Struktur-Tags von IFD0 stören die LATTICE-Ausgabe beim
, und `ExifImageWidth` / `ExifImageHeight` werden ausgeschlossen, da sie die
*Quell*-Aufnahme beschreiben — ein Export, dessen Größe jemals geändert wurde, würde andernfalls Abmessungen enthalten,
die im Widerspruch zu seinem eigenen Raster stehen. XMP wird direkt geschrieben statt kopiert, da ExifTool
XMP-Tags desselben Aufrufs verworfen werden, wenn der XMP-Block kopiert wird (wodurch die MAPIR-
Kalibrierungs-Tags verloren gingen).

### Wo die Ausgabedateien gespeichert werden

Die Dateien werden **im Projektordner abgelegt, gruppiert nach Kamera und anschließend nach Dateiformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Der Kameraordner lautet `LATT-<sensor>-<lens>-F<filter>` für LATTICE (entsprechend dem EXIF-Datensatz der Aufnahme
`Model`) und `<model>_<filter>` für Survey3 – zwei Kameras, die sich einen Sensor und einen Filter teilen, sich jedoch
hinsichtlich Objektiv, Vignettierung, das Sichtfeld und die Verzerrung unterscheiden sich. Das Format
des Ordners folgt `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` oder `tiff32` für
`TIFF (32-bit, Percent)`.

> **Jedes exportierte Produkt behält den Namen der QUELLDATEI bei.** Ein Radiance-Export von
> `capture_…_raw.tif` heißt weiterhin `capture_…_raw.tif` – es befindet sich lediglich in
> `tiff32/Radiance_Images/`. **Der Ordner identifiziert das Produkt, nicht der Dateiname**, daher findet eine Globbing-Abfrage
> nach „`*radiance*.tif`“ liefert keine Ergebnisse; suchen Sie stattdessen nach dem Verzeichnis.

### Lichtsensoraufzeichnungen – kalibriert `.daq` + `.csv`

`process` verarbeitet auch die `.daq`-Aufzeichnungen in Ihrem Eingabeordner und benötigt dafür **keine**
Bilddaten: Ein eigenständig geflogener DAQ-U / DAQ-M / DAQ-E stellt eine vollständige
Aufzeichnung, und ein Ordner, der ausschließlich `.daq`-Dateien enthält, ist eine gültige Eingabe.

Ein DAQ kann **ohne** seine Kalibrierung aufgezeichnet werden – genau das ist es, was die öffentlichen
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) Rekorder
(`record_daq.py`) standardmäßig tun: Sie schreiben die rohen Sensorwerte und versehen die Datei mit einem Zeitstempel, sodass
Chloros die Werkskalibrierung dieses Sensors **über die serielle Schnittstelle** abruft (zuerst aus dem lokalen Cache,
dann aus der MAPIR-Cloud) und anwendet. `process` schreibt das Ergebnis zurück:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

Die Datei `.csv` enthält eine Zeile pro Messwert: UTC-Zeitstempel, Integrationszeit, Gesamtleistung,
photopische/skotopische Lux, PPFD (und deren Aufteilung in Blau/Grün/Rot), Spitzenwellenlänge, dann das
vollständige Spektrum auf dem sensoreigenen Wellenlängengitter. `.daq` importiert erneut, ohne
ein zweites Mal kalibriert zu werden.

Bei Erfolg meldet der Lauf `Light-sensor products written: N (calibrated .daq + .csv)`.
Die Angabe in Klammern beschreibt, was tatsächlich geschrieben wurde; so lautet die Ausgabe
`(RAW COUNTS — this sensor has no calibration bundle)` für einen Sensor ohne Bundle und
`(N calibrated, M raw counts)` für einen Ordner, der beides enthält. Die Backend-eigenen
Überschriften `[DAQ-EXPORT]` und `[RUN-SUMMARY]` leiten ihre Formulierung auf dieselbe Weise ab – keiner der
drei darf einen Rohdatenexport als kalibriert bezeichnen.

Eine DAQ-U-/DAQ-M-/DAQ-E-Aufzeichnung, deren Kalibrierungsbündel nicht abgerufen werden kann – weil Sie
offline sind oder für diesen Sensor keine Kalibrierung vorliegt –, wird **mit einer Begründung** in einer
`[DAQ-EXPORT]`-Zeile übersprungen und niemals als „kalibrierte“ Datei mit Rohzählwerten ausgegeben.
Stellen Sie eine Internetverbindung her und führen Sie den Vorgang erneut aus. Die Begründung ist diejenige, die der Leser tatsächlich
für diese Datei ermittelt hat (unlesbares Schema, kein Paket, ein Schreibfehler), und die Laufzusammenfassung
listet **einzelne** Gründe auf – zwanzig übersprungene Dateien aufgrund einer Ursache werden als eine
Ursache ausgewiesen, nicht als zwanzig Wiederholungen derselben.

#### DAQ-A-Aufzeichnungen werden als Rohzählwerte exportiert

Die **DAQ-A**-Familie stammt aus der Zeit vor dem seriennummernspezifischen Bundle-System und verfügt über kein Kalibrierungs-Bundle,
das abgerufen werden könnte – sie wird stattdessen vor Ort anhand eines Reflexionsziels kalibriert, weshalb
sie nie eines benötigte. Die Ablehnung dieser Aufzeichnungen ließ den Anwendern keine Möglichkeit, ihre
Werte überhaupt zu erhalten, daher werden sie unter einem **anderen Namen** exportiert:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Ein anderer Dateiname statt eines Flags innerhalb der Datei, da die Angabe den
Versand per E-Mail als reiner Name überstehen muss. Der Header „`.csv`“ gibt
„`raw spectral sensor counts (NOT irradiance)`“ an und weist darauf hin, dass die Werte
**innerhalb** der Datei vergleichbar sind – was genau dem Zweck entspricht, für den sie bei der zielbasierten Kalibrierung verwendet werden – und
nicht sensorübergreifend. Die leistungsabhängigen photometrischen Spalten (Gesamtleistung, photopische und
skotopische Lux, PPFD) werden mit **NULL** geschrieben, anstatt aus Zählwerten integriert zu werden, und die
lautet `RAW COUNTS`, sodass die in einem Protokoll „exportierten“ Werte nicht als Bestrahlungsstärke gelesen werden können.

Ältere **v1.01 / v1.02** (die von einem DAQ-A-SD geschrieben werden) enthalten keine Epochenangabe pro Messwert,
sondern nur den Schreibzeitpunkt der Datei. Der Bild↔Downwelling-Matcher lehnt sie weiterhin ab – der Abgleich eines
Bildes mit einem Schreibzeitpunkt wäre unsichtbar falsch — aber der Exporter liest sie, und
die Datei „CSV“ gibt „`clock=daq_created_on`“ aus, sodass das Produkt angibt, auf welcher Uhr es läuft.

### Hinweise

- „`process`“ erkennt automatisch, , ob Ihr Ordner aus Survey3, LATTICE oder einer Mischung besteht.
- Der Fortschritt wird über Server-Sent Events übertragen; das CLI zeigt den Live-Fortschritt pro Thread an (Erkennen, Analysieren, Verarbeiten, Exportieren).
- Bei Linux/Jetson überprüft das CLI den Swap-Speicher und gibt möglicherweise eine Warnung aus, bevor große Ordner verarbeitet werden. Der texturbasierte Debayer wendet außerdem automatisch eine GPU-Frequenzbegrenzung auf Jetson-Geräte mit geringer Leistungsaufnahme (Nano, Orin Nano) an.
- Bei erfolgreichem Abschluss meldet der Lauf, wie viele Bildprodukte geschrieben wurden (`Image products written: N`).

#### Ein Lauf, der keine Bilder schreibt, schlägt fehl

Wenn Sie Produkte angefordert haben und der Lauf **keine** geschrieben hat – nur `project.json` und
`calibration_data.json` –, behandelt `process` dies als Fehler: Es gibt
`Processing finished but wrote no image products.` aus und **beendet den Lauf mit einem Wert ungleich Null**, sodass ein Skript
dies erkennen kann. Die Meldung nennt den Projektordner und die üblichen Ursachen:

- Der Eingabeordner wurde nicht als Erfassungsordner erkannt (überprüfen Sie das Layout und `--input-level`), oder
- jedes angeforderte Produkt wurde als für diese Kameras nicht anwendbar übersprungen (z. B. bei der Abfrage von
  Strahlungsintensität/Reflexionsgrad von Kameras, die ausschließlich RGB unterstützen).

Führen Sie den Vorgang mit `--verbose` erneut aus und überprüfen Sie das Backend-Protokoll auf die Zeilen `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`-Zeilen,
die kameraspezifische Übersprünge erklären, die sonst nicht in die Ausgabe von CLI gelangen.

Ein absichtlich nur mit Metadaten durchgeführter Lauf – alle Produkte deaktiviert und kein `--indices` – ist dennoch ein
**Erfolg**, da eine leere Bildausgabe hier das korrekte Ergebnis ist.

Ebenso wie ein **leichterLauf nur mit Lichtsensor**: Ein Ordner mit `.daq`-Aufzeichnungen enthält per Definition keine Bilder zum Exportieren
, und der Durchlauf wird anhand der kalibrierten `.daq`- und `.csv`-Dateien bewertet, die stattdessen geschrieben wurden.

---

## `chloros-cli login`

Authentifizieren Sie diesen Rechner mit einem Chloros+-Cloud-Konto. Die Anmeldedaten werden sicher in `~/.chloros/user_session.json` zwischengespeichert.

```
chloros-cli login EMAIL PASSWORD
```

### Beispiele

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (Entfernen von Teilen des Passworts oder Duplizieren von Passwortteilen). Bei einem 401-Fehler versucht CLI automatisch einen erneuten Versuch, indem zunächst `$$` angehängt und anschließend die deduplizierte Hälfte des Passworts verwendet wird; wenn ein Wiederholungsversuch erfolgreich ist, werden Sie angemeldet und die korrekte Syntax mit einfachen Anführungszeichen wird angezeigt, die Sie beim nächsten Mal verwenden sollten.

> **Verwendung im Headless-Modus/per Skript: Keine zwischengespeicherte Sitzung bedeutet eine interaktive Eingabeaufforderung, keinen schnellen Fehler.** Jeder Backend-Befehl zum Starten eines Backends (`process`, `status`, `export-status`, `time-sync`, …) wird ohne zwischengespeicherte Lizenz ausgeführt/Sitzung führt zu einer interaktiven `Email:`- bzw. `Password:`-Eingabeaufforderung über stdin, bevor der Vorgang fortgesetzt wird. Ein unbeaufsichtigter Job ohne zwischengespeicherte Sitzung bleibt daher in der Warteschlange hängen und wartet auf eine Eingabe – führen Sie `chloros-cli login EMAIL PASSWORD` einmal pro Rechner , bevor Sie Headless-Aufträge einplanen.

---

## `chloros-cli logout`

Löscht die zwischengespeicherte Sitzung und erzwingt beim nächsten Aufruf eine neue Anmeldung.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Zeigt die aktuelle Lizenzstufe (Iron/Copper/Bronze/Silver/Gold), den authentifizierten Benutzer und die Anzahl der Gerätebindungen an.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Fragt den aktuellen Fortschritt des Thread-4-Exports ab. Kann **während** der Ausführung von `process` von einer anderen Shell aus sicher aufgerufen werden.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Legt die Anzeigesprache von CLI fest (38 Sprachen werden unterstützt, darunter CJK, RTL und indische Sprachen). Auf älteren Konsolen, die das Skript nicht darstellen können, wird nahtlos auf Englisch zurückgegriffen.

```
chloros-cli language [LANG_CODE] [--list]
```

### Beispiele

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Befehle für den Projektordner

Diese Befehle verwalten den Speicherort des Standard-Projektordners (gemeinsam mit der GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux / Nur Jetson. Überprüft `version_url` anhand von `/etc/chloros/update.conf` und bietet an, die passende Datei `.deb` herunterzuladen und Installation des passenden `.deb` an.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Auf Linux/Jetson führt das CLI außerdem bei jedem Start eine **automatische Update-Prüfung bei jedem Start** (nichtblockierend, verzögert den Befehl niemals): Es liest `/etc/chloros/update.conf` aus, speichert das Ergebnis für 1 Stunde in `~/.chloros/update_cache.json` im Cache und gibt `Update available: vX.Y.Z / Run: chloros-cli update` aus, wenn eine neuere Version vorhanden ist. Bei bei jedem Fehler sowie bei Windows.

---

## `chloros-cli selftest`

Führt einen 7-stufigen Smoke-Test durch: Version, Portverfügbarkeit, Backend-Start, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), Vorhandensein des Denoiser-Modells, CUDA+Denoiser-Bereitschaft.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-Grandmaster-Status und -Steuerung. Der Host Chloros führt den PTP-Grandmaster aus; LATTICE-Cams und DAQ-E-Einheiten sind diesem als Slaves zugeordnet, um geräteübergreifende Zeitstempel zu erhalten.

| Unterbefehl | Beschreibung |
| --- | --- |
| `status` | Grandmaster-Status, BMCA-Prioritäten und Taktidentität anzeigen. |
| `peers` | Über Delay_Req erkannte Slaves auflisten (Kameras + DAQ-E-Sensoren). |
| `cameras` | PTP-Zustand pro Kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Grandmaster-Prozess neu starten. |
| `set-priority --priority1 N --priority2 N` | BMCA-Prioritäten überschreiben. |

### Beispiele

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE-Kamerasteuerung. Jeder Unterbefehl wird über das Backend Chloros geleitet; das Backend verwaltet den Kamerapool, sodass nachfolgende CLI-Aufrufe dasselbe offene Handle wiederverwenden.

### Allgemeine Optionen (gemeinsam für die meisten Unterbefehle)

| Flag | Beschreibung |
| --- | --- |
| `-d, --device N` | Kameraindex (Standard: 0). |
| `-s, --serial SN` | Spezifische Seriennummer; überschreibt `--device`. |
| `--serials SN1,SN2,…` | Durch Kommas getrennte Seriennummern für denKamerabetrieb. |
| `--all` | Auf jeder erkannten Kamera ausführen. |
| `--exposure US` | Belichtungszeit in Mikrosekunden. |
| `--gain DB` | Verstärkung in dB. |
| `--pixel-format FMT` | z. B. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Bildabmessungen. |
| `--preset {default,high_quality,high_speed,triggered}` | Voreinstellung anwenden. Alle im Freilauf, außer `triggered`, das die Kamera auf einen Hardware-Flanke auf Zeile 2 – da diese Leitung nicht angesteuert wird, wartet die Kamera unendlich lange, anstatt eine Aufnahme zu machen. |
| `-o, --output DIR` | Ausgabeverzeichnis (Standard: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-Paketgröße. `auto` führt ICMP+GVSP-Tests durch; `jumbo` = 9000; `standard` = 1500. |

### LATTICE-Kamera: Workflow für die Erstverbindung

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referenz zu Unterbefehlen

#### Erkennung und Informationen

| Unterbefehl | Zweck |
| --- | --- |
| `lattice info` | Liste der verbundenen Kameras (Hersteller, Modell, Seriennummer, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Host-System für optimale Kamerakonfiguration analysieren. `--no-discover` überspringt die Kameraerkennung (schneller, Analyse nur über Netzwerkkarte). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Netzwerkkarteneinstellungen prüfen/korrigieren; Bandbreite/FPS abschätzen. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Netzwerkfähigkeiten des Backends mit stabilem Schema + Array-Empfehlung (gibt `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` behält die angeforderte Auflösung bei, begrenzt jedoch die Ziel-fps – lesen Sie `recommended.recommended_target_fps` aus und übergeben Sie es als Verbindungsziel; behandeln Sie dies als Erfolg, nicht als Fehler. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Was-wäre-wenn-Analyse ohne Kameras zu öffnen. **`--n-active` ist die Gesamtzahl der Kameras im Netzwerk, nicht nur die dieses Arrays**– erhöhen Sie diesen Wert, wenn eigenständige Kameras gleichzeitig streamen oder das Leitungsbudget anhand eines Bedarfs berechnet wird, der diese nicht vollständig berücksichtigt (Standard: `len(--models)`). Es wird immer der Gesamtwert `Wire budget:` (geforderte MB/s im Vergleich zur kollisionssicheren Obergrenze) und `Max cameras:`-Zeilen aus und markiert `** OVER-SUBSCRIBED**`, wenn das Array den Bus überbelegt – siehe [Array-FPS und Burst-Modell](#array-fps--burst-model). |
| `lattice gpu` | GPU-Status anzeigen. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Kamera-Firmware prüfen oder aktualisieren. Die lokale Auswahl `.fwa` ist fixiert: Die Datei in `firmware/<MODEL_PREFIX>/`, die mit dem Build von `MIN_FIRMWARE_VERSION` übereinstimmt, wird, sofern vorhanden, geflasht (nur die höchste Version als Fallback), sodass ein neueres, auf der Festplatte bereitgestelltes Hersteller-Image unwirksam bleibt, bis diese Festlegung geändert wird – bewusst neuere Versionen gelangen über das signierte AWS-Manifest auf die Geräte, was bevorzugt wird, wenn es neuer ist. |
| `lattice presets [--apply NAME]` | Kamera-Voreinstellungen auflisten oder anwenden. |
| `lattice status` | Live-Kamerastatus. |

#### Aufnahme

| Unterbefehl | Zweck |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Einzelbild. **Speichert standardmäßig jeden Exporttyp** (`--processing all`); siehe [Aufnahme-Exportstufen](#capture-export-levels-the-all-default). `--levels` speichert eine explizite Teilmenge (überschreibt `--processing`); `--force-daq` schreibt den zugewiesenen DAQ-Messwert als `.daq`-Sidecar-Datei, selbst bei einer reinen Rohdatenaufnahme. `--jpeg-quality` = JPEG-Qualität 1–100 (Standard 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Auf Festplatte streamen, bis Strg+C gedrückt wird. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Browserbasierte Live-MJPEG-Vorschau. `--ae-damping` legt die Dämpfung der automatischen Belichtung fest (0,4–100). |

#### Sensor-Optimierung

| Unterbefehl | Zweck |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Lesen/Schreiben eines beliebigen GenICam-Knotens. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Belichtung &amp; AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Verstärkung &amp; automatische Verstärkung. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Sensor-ROI und Binning. |
| `lattice format [--set FMT] [--list]` | Pixelformat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardware-/Software-Trigger. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (keine Flags = One-Shot-Weißabgleich) | Weißabgleich-Operationen. Nur RGB/Bayer-Kameras; bei Mono-M3M-Kameras wird diese Operation übersprungen. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB-Farbpipeline für die Anzeige. `natural` (Standard) ist die kostengünstige Live-Nachbearbeitung; `enhanced` fügt Defringing + Lebendigkeit + lokalen CLAHE-Kontrast hinzu, um den vollständigen Hub-Parity-Look zu erzielen – bei etwa doppelten Kosten pro Bild, was zu einer niedrigeren **Live**-Bildrate führt; gespeicherte Aufnahmen erhalten in jedem Fall die vollständige Nachbearbeitung. RGB/nur Bayer-Kameras; wird bei Mono-M3M übersprungen. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Bildschirmsättigung/Kontrast (RGB-Filterkameras). Wird bei Mono-M3M übersprungen. |
| `lattice filter [--set NAME] [--list]` | Filtermodell der Kamera festlegen (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Strom- und Thermoknoten der Sonde; Umschalten auf Energiesparmodus im Leerlauf. |

#### Kalibrierung &amp; Sensoren

| Unterbefehl | Zweck |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrierung anhand eines Reflexionsziels. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Befehle für den integrierten Sensor für einfallendes Licht. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Vignettierungskorrektur auf vorhandene Bilder anwenden. |

#### Mehrkamera (Transiente Sitzungen)

| Unterbefehl | Zweck |
| --- | --- |
| `lattice multi-info` | Alle Kameras mit Synchronisationsrollen auflisten. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Ein synchronisiertes Bild von jeder Kamera. Speichert **standardmäßig alle Exporttypen**, wenn ein persistentes Array angeschlossen ist; wird der vorübergehende Fallback ohne Array**nur debayert** (führen Sie für den Rest zuerst `array-connect` aus). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Synchronisierte Bilder streamen (vorübergehend). |
| `lattice multi-test [--count N]` | GPIO-Synchronisations-Timing-Test. |
| `lattice multi-detect [--line LINE] [--json]` | Automatische Erkennung der GPIO-Master/Slave-Verdrahtung. |

#### Ausrichtung

| Unterbefehl | Zweck |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — plus Detektor-/Matcher-Regler `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-Regler `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, Kombination aus mehreren-Frame-Kombination `[--averaging mean\|median\|inlier_weighted]`, geometrische Einschränkungen `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, räumliche Einschränkungen `[--roi X0,Y0,X1,Y1] [--mask PATH]` und Slave-spezifische Überschreibungen `[--per-cam-override SN:KEY=VALUE]` (wiederholbar) | Berechnung des Ausrichtungsprofils aus Live-Kameras. `--prefilter` verwendet standardmäßig `gradient` (Kantenkarte; entspricht dem GUI-/Array-Aligner – Kanten bleiben über Spektralbänder hinweg erhalten). `--matcher flann` zahlt sich ab ca. 5000 Merkmalen aus; `--averaging median` ist robust gegenüber einer fehlerhaften Aufnahme, `inlier_weighted` gewichtet nach der Anzahl der Übereinstimmungen; `--lock-scale` projiziert auf die nächstgelegene Rotation (keine Skalierung), `--lock-axis` setzt eine Translationskomponente auf Null; `--mask` gilt für jede Kamera (verwenden Sie `--per-cam-override` für kameraspezifische Einstellungen, z. B. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` verweigert das Speichern einer Kalibrierung, deren RMS-Wert bei der Reprojektion den Schwellenwert überschreitet. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Nimmt ein ausgerichtetes Multiband-Bild auf. `--bit-depth` passt sich standardmäßig der Kamera an; `--no-crop` behält das gesamte Bild bei (mit Schwarz aufgefüllt); `--interpolation` (Standard: `linear`) und `--border-mode`/`--border-value` (Standard: `constant`/0) steuern die CPU-Verzerrung – der GPU-Pfad ist in jedem Fall bilinear. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Stream-ausgerichtete Multiband-Frames (gleiche Warp-Regler wie bei `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Details zum Profil anzeigen. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Reihenfolge der Ebenen ändern. |

#### Index / Vegetationsberechnungen

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Vollständiger Flagsatz: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (wiederholbar), `--capture-level raw|debayered|radiance|reflectance|unknown` (Überschreiben der im Quelldatensatz TIFF gespeicherten Erfassungsstufe; Standard: aus den Metadaten von TIFF lesen), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Bei `--live` gelten die Regler für die Ausrichtungsverzerrung ebenfalls: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Bei den Symbolen „`--channel`“ wird zwischen Groß- und Kleinschreibung unterschieden.** Die Symbolbezeichnung muss exakt mit den Kanalnamen des Presets übereinstimmen (Presets verwenden Kleinbuchstaben, z. B. NDVI = `red`, `nir` – siehe `--list-presets`), und die Bandseite muss mit einem Bandnamen im angepassten Stapel übereinstimmen (oder im Offline-Modus ein 0-basierter Bandindex sein). `--channel red=Red_660 --channel nir=NIR_850` funktioniert; `--channel RED=660` schlägt mit einem `channel_map missing entries`-Fehler fehl.

#### Persistente Verbindungen (Smart-Prep, GUI-äquivalenter Ablauf)

Diese Befehle halten Kameras über mehrere CLI-Aufrufe hinweg im Backend-Pool offen.

| Unterbefehl | Zweck |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Eine Kamera zum Pool hinzufügen (Einzelkamera, kein Array). |
| `lattice cam-disconnect [--serial SN] [--all]` | Freigeben. |
| `lattice cam-list` | Kameras im Pool auflisten. |
| **`lattice array-connect`**|**Ein persistentes, synchronisiertes Array verbinden (DER empfohlene Einstiegspunkt).** Führt den vollständigen Smart-Prep-Ablauf über die GUI aus. |
| `lattice array-disconnect [--array-id ID] [--all]` | Ein Array freigeben. |
| `lattice array-list` | Verbundene Arrays auflisten. |
| `lattice array-status [--array-id ID]` | Live-FPS, PTP, letzter Fehler. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Eine synchronisierte Aufnahme aus dem Live-Array – Einzel / Kontinuierlich / Intervall / Schnellste. **Standardmäßig `all`** (eine Datei pro zutreffendem Exporttyp pro Kamera). Übersprungene Kameras (z. B. RGB, ausgeschlossen von Radianz/Reflexionsgrad) werden mit `Skipped: SN:<serial> (<reason>)` gemeldet; der für den Reflexionsgrad verwendete DAQ-Messwert wird separat gespeichert und mit `DAQ: <path>` gemeldet. Siehe [Aufnahmemodi, Rekorder &amp; Offline-Nachbearbeitung](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Die Live-Ansicht des kombinierten Index als Video/GIF aufzeichnen (in Überwachungsqualität; erfordert, dass der kombinierte Stream geöffnet ist). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Raw-Bayer-Burst mit hoher Bildrate (in Analysequalität; Offline-Nachbearbeitung). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Eine gespeicherte Raw-Serienaufnahme in kalibrierte Videos umwandeln. |

##### `array-connect`-Optionen

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Alle LATTICE-Kameras automatisch erkennen (erfordert ≥2) | Die erste Seriennummer ist der MASTER. Bei Weglassen wird die Erkennung auf LATTICE-Modelle (`TRI032*`) und verbindet alle miteinander. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-Synchronisationsleitung. |
| `--target-fps F` | auto | Auslösefrequenz des Masters. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Tier-Auswahl überschreiben. |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | **Das nachhaltige Bandbreitenbudget des Hosts in MB/s – der Wert, von dem die gesamte Array-Zuweisung abhängt.** Verringern Sie diesen Wert, wenn das Array GVSP-beschädigte Frames meldet: Der Wert „auto“ wird aus der vom Netzwerkadapter gemeldeten Verbindungsrate abgeleitet, die USB-Adapter, schmale PCIe-Lanes und stark ausgelastete gemeinsam genutzte Fabric-Strukturen überbewertet. Persim Array-Erfassungsblock des Projekts vermerkt, sodass ein „reopen“ / CLI / SDK „reconnect“ den Wert wiederherstellt. Siehe [Array-Zustand](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Hardware-Binning. |
| `--no-recommend` | off | Den Netzwerk-Analyseschritt überspringen. |
| `--no-ptp` | aus | PTP deaktivieren (Zeitstempel verschiedener Kameras sind dann **nicht** vergleichbar). |

### Smart-AE / Smart-Capture

LATTICE-Arrays führen im Hintergrund kontinuierlich eine AE durch, sobald sie verbunden sind, doch eine neu ausgerichtete Szene benötigt einen Moment, um zu konvergieren. `array-capture --smart` ist die **praktische Lösung**: Es wartet, bis sich die Belichtungssteuerung bei allen Kameras im Array eingependelt hat, und löst dann die Aufnahme aus. Verwenden Sie diese Funktion, wenn Sie während einer Sitzung die Szene wechseln.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Die Stabilisierungsrichtlinie ist standardmäßig konservativ eingestellt: 5 s Timeout, 1,5 s Stabilitätsfenster, ±5 % Toleranz für die Belichtungsstreuung. Passen Sie die Einstellungen über SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) an, wenn Sie ein anderes Verhalten der Automatisierung benötigen.

### Exportstufen für Aufnahmen (Standardwert: `all`)

Ab dieser Version sind `lattice capture`, `lattice multi-capture` und `lattice array-capture` **standardmäßig auf `--processing all` eingestellt** — eine gespeicherte Datei pro Exporttyp, die für jede Kamera gilt und dem Verhalten von „Capture All“ in der Benutzeroberfläche entspricht. Die Stufen sind:

| Stufe | Ausgabe | Gilt für |
| --- | --- | --- |
| `raw` | Ein-Kanal-Bayer (Mono-Kameras: das einzelne Band) direkt vom Sensor. | Alle Kameras. |
| `debayered` | 3-Kanal-BGR-Demosaik (Mono-Kameras: 1-Kanal-Graustufen). | Alle Kameras. |
| `radiance` | float32 W/m²/sr/nm über die vollständige radiometrische Kette. | Multispektral (M3C/M3M) – **wird bei Kameras mit RGB-Filter übersprungen**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-kompatibel. | Nur multispektral und **nur, wenn ein DAQ gebunden ist + die Kamera kalibriert ist**; andernfalls übersprungen. |
| `preview` / `display` | Vollständige GUI-Vorschau-Kette (CCM + WB + Gamma gemäß dem Profil der Kamera). `lattice capture` benennt dies als `preview`; `array-capture`/`multi-capture` verwenden `display`. | Alle Kameras. |

Geben Sie eine einzelne Stufe an, um nur diese zu speichern (`--processing debayered`). Wenn Sie nach `all` fragen, werden Pegel, die für eine bestimmte Kamera nicht zutreffen, übersprungen (und gemeldet), es wird jedoch kein Fehler ausgegeben – eine nicht angeschlossene oder nicht kalibrierte Kamera erhält weiterhin `raw` / `debayered` / `preview`.

Für jeden Reflexionsbildrahmen wird der tatsächlich verwendete DAQ-Downwelling-Messwert in ein **`.daq`**-Sidecar neben dem Bildmaterial (damit die Aufnahme später erneut verarbeitet werden kann) und in einer `DAQ:`-Zeile gemeldet.

### So sieht ein Aufnahmordner aus

Jeder Exporttyp landet in seinem **eigenen Unterordner** unter `-o` abgelegt, sodass sich bei einer mehrstufigen Aufnahme niemals verschiedene Typen vermischen:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` ist der Zeitstempel der Aufnahme und `<serial>` die Seriennummer der Kamera; somit teilt sich eine synchronisierte Gruppe einen
Zeitstempel über alle Kameras hinweg. **Beachten Sie die eine Asymmetrie:** Die Ebene `display` wird in einem Ordner
mit dem Namen `preview/` gespeichert, während die Dateien selbst `_display` im Namen behalten – Ordner und Suffix unterscheiden sich
nur bei dieser Ebene. Unbekannte Ebenen werden in einen Ordner mit ihrem eigenen Namen verschoben, und wenn der Unterordner
nicht erstellt werden kann, wird die Datei in das Ausgabeverzeichnis geschrieben, anstatt verloren zu gehen.

**Neuverarbeitung eines „captures“-Ordners:**Verweisen Sie `chloros-cli process` auf das**„captures“-Stammverzeichnis**
(`output/`). `process` importiert normalerweise nur den von Ihnen angegebenen Ordner; enthält dieser jedoch keine
Bilder enthält und Unterordner aufweist, durchsucht er diese automatisch – sodass die Unterordner auf der Ebene des Stammordners und der
Stammordner `.daq` alle auf einmal erfasst werden. Jede Ebene einer Erfassung wird als einzelnes Bild importiert, wobei
die anderen Ebenen als Modi verfügbar sind, anstatt als ein Bild pro Ebene.

Die direkte Benennung eines **Unterordners auf einer Ebene** (z. B. `output/raw/`) funktioniert ebenfalls. Dabei bleibt der Stammordner
`.daq` unberücksichtigt; kopieren Sie daher den DAQ-Wert oder verweisen Sie darauf, wenn Sie ein radiometrisches
Produkt aus `raw/` – andernfalls kann der Zeitstempelabgleich nicht zugeordnet werden.

**Die Verarbeitung beginnt immer bei `raw`.** Innerhalb jeder Erfassung ist das Rohbild die Pipeline-Quelle;
`debayered`, `radiance`, `reflectance` und `preview` dienen als Anzeigemodi, werden jedoch niemals
zurück in die Pipeline geleitet. Bei der erneuten Verarbeitung eines abgeleiteten Produkts würden Vignettierung, CCM und
die Strahlungsberechnungen, die bereits in seine Pixel eingebrannt sind; daher wird Chloros abgelehnt, anstatt
eine doppelte Verarbeitung durchzuführen. Zwei wichtige Konsequenzen:

- Die `index/`- und `composite/`-X-Renderings werden **niemals** verarbeitet. Es handelt sich um Ausgabedateien, nicht um Aufzeichnungen –
  ein NDVI-LUT-Rendering lässt keine aussagekräftige Radiance-Interpretation zu.
- Ein **ohne** `raw` exportierter „captures“-Ordner (z. B. `array-capture --processing reflectance`) hat
  keine gültige Pipeline-Quelle. Diese Aufnahmen lassen sich normal importieren und anzeigen, aber `process` überspringt
  und gibt dies auch an:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Wenn Sie ein abgeleitetes Produkt wirklich durchschleusen müssen – eine Hub-Sitzung, die mit
  aktiviertem `demosaic` aufgezeichnet wurde, oder einen Legacy-Ordner –, erzwingt `--input-level {raw,debayered,processed}` den Einstiegspunkt
  erzwingt und die Überspringung außer Kraft setzt. Dieses Flag ist die absichtlich vorgesehene Ausweichmöglichkeit; `auto` (die Standardeinstellung)
  verarbeitet niemals eine Aufnahme, die keine Rohdaten enthält.

### Übersprungene Aufnahmen in gemischten Filter-Arrays

Wenn Sie RGB- und Multispektralkameras in einem Array mischen, speichert `array-capture --processing radiance` (oder `reflectance`) speichert die Multispektral-Bilder und **überspringt** die RGB-Kameras – die Strahlungsintensität pro Bayer-Pixel ist für einen Breitbandsensor nicht aussagekräftig. CLI gibt jede gespeicherte Datei (mit ihrer Exportstufe) aus, jeden geschriebenen `.daq` sowie jedes Überspringen explizit aus, sodass die Dateianzahl nicht überraschend ist:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Token für Überspringgründe folgen dem Muster `<level>-not-applicable-to-rgb-cam`. ReflectReflexion kann ebenfalls mit `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` übersprungen werden sowie mit `dls-uncalibrated-band-<nm>`, wenn das Band größtenteils außerhalb des radiometrisch kalibrierten Bereichs des DAQ-Lichtsensors (~374–974 nm) – unter den lieferbaren Artikelnummern gilt dies nur für F988, dessen unterstützter Arbeitsablauf auf dem Reflexionspanel basiert.

Verwenden Sie `--processing debayered` (oder `display`), um jede Kamera unabhängig vom Filtertyp einzubeziehen, oder das Standardprogramm `all`, um alle anwendbaren Pegel pro Kamera in einem Durchgang zu erfassen.

---

## Aufnahmemodi, Recorder und Offline-Nachbearbeitung

Diese Funktionen arbeiten alle mit einem **persistenten Array** (führen Sie zuerst `array-connect` aus). Sie spiegeln das GUI-Aufnahmefenster wider.

### `array-capture`-Modi

`array-capture` ist ein einzelner Befehl mit vier Auslösemodi sowie einer Reihe von Export-Schaltflächen:

| Modus | Flag | Verhalten |
| --- | --- | --- |
| **Einzelaufnahme** *(Standard)* | (keine) | Eine synchronisierte Aufnahmegruppe, dann Beenden. |
| **Serienaufnahme** | `--continuous` | Aufeinanderfolgende Durchläufe bis `Ctrl+C`, `--count N` oder `--duration S`. |
| **Intervall** | `--interval S` | Ein Durchlauf alle `S` Sekunden (gemessen ab dem Beginn jedes Durchlaufs), gleiche Grenzen. |
| **Schnellste** | `--fastest` | Nur Rohdaten + der zugewiesene DAQ-Messwert + der zusammengesetzte Index; überspringt die Berechnungen für Strahlungsintensität, Reflektanz und Anzeige, damit das Bild schnell angezeigt wird. Impliziert `--processing raw --force-daq`. Die gespeicherten Daten `.daq` werden später zu kalibrierten Produkten nachbearbeitet. |

Export-Schaltflächen (kombinierbar mit jedem Modus; alle nutzen die GUI/SDK-Endpunkt):

| Flag | Wirkung |
| --- | --- |
| `--processing LEVEL` | Einzelne Exportebene oder `all` (Standard). |
| `--levels L1,L2,…` | Explizite Teilmenge von Exporttypen (z. B. `raw,radiance,reflectance`); **überschreibt `--processing`**. |
| `--aligned` / `--no-aligned` | Nicht-Raw-Exporte aller Elemente werden an das [Ausrichtungsprofil](#alignment) (co-registriert). Rohdaten bleiben unverzerrt, enthalten die Transformation jedoch in den Metadaten. Weicht auf nicht ausgerichtete Daten zurück (mit einer Warnung) , wenn das Array kein Profil hat. |
| `--index` / `--no-index` | Speichert bzw. überspringt die pro Kamera konfigurierte Vegetationsindex-Überlagerung, sofern eine konfiguriert ist. Standard: Renderen. |
| `--force-daq` | Speichert den zugewiesenen DAQ/DLS-Messwert als `.daq`-Sidecar-Datei, auch wenn keine ausgewählte Ebene dies benötigt (z. B. bei einer reinen Rohdaten-Aufnahme), damit die Bilder offline in Reflektanz-/Indexwerte nachbearbeitet werden können. |
| `--smart` | Warten, bis sich die AE über alle Kameras hinweg stabilisiert hat, bevor ausgelöst wird (siehe [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-Pixelkomprimierung. `deflate` (Standard) = verlustfreie zlib L1 + horizontaler Prädiktor, ~4,1 MB pro Bild in voller Auflösung; `none` = unkomprimiert, ~5× schnellerer Schreibvorgang bei ~6,3 MB pro Bild – für maximale Dauerrate verwenden, sofern die Festplatte dies zulässt. Beide sind verlustfrei und werden beim Import identisch gelesen beim Import. |

> **Einmaliges Schreiben TIFF + das Modell für die Dauerübertragungsrate.**Die Aufnahmen werden in**einem**Durchgang in eine TIFF-Datei geschrieben, die Pixel + XMP + IFD0 Hersteller/Modell enthält (gemessen bei voller Auflösung Mono12: 36 ms komprimiert / 6,5 ms unkomprimiert, gegenüber ca. 148 ms beim alten Verfahren „Erst schreiben, dann mit ExifTool neu schreiben“); die einzige verbleibende ExifTool-Aufgabe (Überarbeitung der EXIF-Sub-IFDs) läuft in einem asynchronen Hintergrund-Worker, und ein Bild ist vollständig und importbereit, selbst wenn dieser nie ausgeführt wird. Beachten Sie, dass die DEFLATE-Komprimierung den Python-GIL hält, sodass komprimierte Schreibvorgänge**nicht**über die Schreib-Threads der einzelnen Kameras parallelisiert werden – eine dauerhafte Aufnahme mit 8 Kameras in voller Auflösung bei Sensor-Rate (~10,4 fps) erfordert `--compression none`**sowie** eine Festplatte der NVMe-Klasse (~500 MB/s Dauer-Schreibgeschwindigkeit). Derselbe Regler wird auf `POST /api/camera/array/capture` als `compression` bereitgestellt.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` – Video/GIF mit kombiniertem Index (Überwachungs-)

Zeichnet alles auf, was die **Live-Kombinationsansicht** anzeigt, auf ein `.avi` (und optional ein `.gif`). Da der Live-Composite-Stream abgegriffen wird, muss der kombinierte Stream geöffnet sein (z. B. wird das Array in der GUI in der Vorschau angezeigt wird), damit Frames empfangen werden können. Es fragt alle 2 s den Fortschritt ab und stoppt bei `--duration`, `Ctrl+C` oder wenn der Recorder selbstbeendet.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--array-id ID` | nur Array | Ziel-Array (weglassen, wenn nur eines angeschlossen ist). |
| `-o, --output DIR` | `output` | Ausgabeverzeichnis (backend-lokal). |
| `--fps F` | `10` | Bildrate der Aufzeichnung. |
| `--duration S` | bis Strg+C | Automatischer Stopp nach `S` Sekunden. |
| `--gif` | aus | Auch ein animiertes GIF schreiben. |
| `--gif-only` | aus | Nur ein GIF erstellen (kein `.avi`). |

### `array-burst` – Raw-Bayer-Serienaufnahme mit hoher Bildrate (für Analysezwecke)

Liest direkt aus dem Synced-Group-Puffer der Aufnahmeschleife – **keine Kalibrierungskette, kein exiftool, keine Live-Ansicht erforderlich** – läuft somit mit der vollen Bildaufnahmerate der Kamera. Schreibt Rohbilder + ein Manifest pro Bild + ein `.daq` pro eindeutigem DLS-Messwert unter `<output>/bursts/<base>/`. Offline nachbearbeiten (nächster Befehl) oder übergebe `--build`, um dies sofort nach Beendigung durchzuführen.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--array-id ID` | nur Array | Ziel-Array. |
| `-o, --output DIR` | `output` | Ausgabeverzeichnis (Burst landet in `<DIR>/bursts/<base>/`). |
| `--duration S` | bis Strg+C | Automatischer Stopp nach `S` Sekunden. |
| `--max-frames N` | unbegrenzt | Automatischer Stopp nach `N` Rohbildern. |
| `--build` | aus | Nach dem Stopp die Bildserie sofort erneut verarbeiten (wie bei `array-build-video`). |
| `--products …` | `combined:index` | Mit `--build`: Welche Videos sollen erstellt werden (siehe unten)? |
| `--fps F` | `10` | Bei `--build`: Bildfrequenz des Ausgabevideos. |
| `--save-tiffs` | aus | Mit `--build`: zusätzlich pro Einzelbild kalibrierte TIFF-Dateien speichern. |
| `--gif` | aus | Mit `--build`: zusätzlich animierte GIFs schreiben. |

### `array-build-video` – Offline-Nachbearbeitung einer gespeicherten Bildserie

Ordnet jedes Rohbild zeitlich dem nächstgelegenen gespeicherten `.daq`-Messwert zu und leitet es durch die **gleiche Radianz-/Reflexionsgrad-/Index-Kette wie die Import-Pipeline**, wodurch ein oder mehrere Videos gerendert werden.

`--products` ist eine durch Kommas getrennte Liste von `kind:level`-Elementen, wobei `kind` ∈ `per_cam` | `combined` und `level` ∈ `radiance` | `reflectance` | `index`. Ein bloßer `level` (kein `kind:`) wird standardmäßig auf `per_cam` gesetzt. Der Standardwert ist `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--burst-dir DIR` | (erforderlich) | Pfad zum Burst-Ordner (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-Liste, wie oben. |
| `--fps F` | `10` | Bildrate (fps) des Ausgabevideos. |
| `--save-tiffs` | aus | Speichert zusätzlich kalibrierte TIFF-Dateien für jedes Einzelbild zusammen mit dem/den Video(s). |
| `--gif` | aus | Erstellt zusätzlich animierte GIFs. |

> **Wählen Sie den richtigen Recorder aus.** `array-record` ist für die *Überwachung* geeignet – es erfasst das angezeigte Live-Composite und benötigt einen offenen Stream. `array-burst` → `array-build-video` ist für die *Analyse* geeignet — er speichert Rohsensordaten mit voller Rate und rekonstruiert anschließend kalibrierte Radianz-/Reflexionsgrad-/Index-Videos, wobei keine Live-Ansicht erforderlich ist.

### Mono (M3M) Einbandkameras

Die **M3M**-Reihe ist das Mono-Pendant zur Bayer-**M3C**: ein Schmalband-Interferenzfilter pro Kamera (`M3M-<lens>-F<wavelength>`, z. B. `M3M-L87-F685`), sodass der Sensor ein**einziges Graustufenband** ohne Bayer-Mosaik liefert. Es gibt nichts zu demosaiken, kein kanalübergreifendes Übersprechen zum Entmischen und keinen Weißabgleich einzustellen – die gesamte Farbpipeline von RGB bis zur Anzeige kommt hier schlichtweg nicht zum Tragen.

Was das für das CLI bedeutet:

- **`lattice white-balance`, `lattice color-profile` und `lattice color`**erkennen eine Mono-Kamera und**überspringen den Vorgang mit einer einzeiligen Meldung**, anstatt sinnlose Einstellungen zu übertragen. Sie laufen in derselben Sitzung weiterhin normal mit einer RGB/Bayer M3C-Kamera.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** funktionieren weiterhin – Strahlungsintensität und Reflexion sind *bandweise* radiometrische Karten und für ein Band perfekt definiert. Mono-Bilder enthalten eine **identische** Sensor-Antwortmatrix (keine 3×3-Entmischung), sodass die Ebene die Kalibrierungsberechnungen unverändert durchläuft.
- **Eine einzelne Mono-Kamera kann keinen Vegetationsindex erzeugen.**NDVI/NDRE/etc. benötigen mindestens zwei Bänder (z. B. Red + NIR). Um einen Index aus Mono-Hardware zu erhalten, richten Sie**mehrere** M3M-Kameras mit unterschiedlichen Wellenlängen, fügen Sie diese zu einem Multiband-Stack zusammen und berechnen Sie *diesen* als Index:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

Die Symbole „`--channel`“ müssen **genau** mit den Kanalnamen der Voreinstellung übereinstimmen (Groß-/Kleinschreibung beachten; „NDVI“ sind Kleinbuchstaben, `red`,`nir` — siehe `--list-presets`), und der Bandseitenname bezeichnet ein Band im ausgerichteten Stapel (im Offline-Modus werden auch 0-basierte Bandindizes akzeptiert, z. B. `--channel red=0 --channel nir=1`).

Das Unterscheidungsmerkmal im gesamten Stapel ist das `M3M`-Token in der Modellzeichenfolge (es erscheint niemals in einer `M3C`-Zeichenfolge), das in der GUI als `is_mono` angezeigt wird.

---

## Einrichtung und Optimierung der Host-Netzwerkkarte (LATTICE-Arrays)

LATTICE-Kameras streamen GVSP über den Ethernet-Adapter des Hosts, daher sind bei Arrays mit mehreren Kameras der **Treiber**und die**Empfangsringgröße** des Adapters ebenso wichtig wie die Verbindungsgeschwindigkeit. Falsche Einstellungen werden im Bereich „Array-Einstellungen“ als `FRAMES WILL DROP`- bzw. `Reduce ROI to enable`-Gate angezeigt (sowie in `lattice network-analysis` bzw. im SDK als „`analyze_array_network()`“), selbst wenn die Kameras selbst einwandfrei funktionieren.

### USB-10-GbE-Adapter – Realtek RTL8157 („Realtek USB 10GbE Family Controller“)

| Element | Erforderlicher Wert | Warum dies wichtig ist |
| --- | --- | --- |
| **Treiberversion**|**≥ v10.67 (Jan. 2026)**, INF `rtump64x64sta.inf` | Der veraltete**2016**-Treiber (v10.65, `rtump64x64.inf`) behandelt das Herunterfahren und Bugchecks mit**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**beim Herunterfahren, Neustart oder in den Ruhezustand versetzen. Der Übergang hängt sich auf (~5 Minuten Timeout), der Benutzer schaltet das Gerät zwangsweise aus, und wiederholte unsaubere Herunterfahren**beschädigen das WMI-Repository**(PowerShell/Tools zeigen Fehler mit `Invalid class` an) und**blockieren den USB-Stack** beim nächsten Systemstart (Netzwerkkarte lässt sich nicht aktivieren; USB-Laufwerke werden nicht mehr erkannt). Führen Sie ein Update von realtek.com (oder vom Dongle-Hersteller) durch, bevor Sie auf saubere Neustarts setzen. |
| **Empfangspuffer**— Schlüsselwort `ReceiveBufferLen` |**256**(Treiber-Maximum) | Der RX-Ring der Netzwerkkarte. Die Treiber-Standardeinstellung von**32**lässt nur ~0,26 MB nutzbaren Ring übrig — viel zu klein für eine Serienaufnahme mit mehreren Kameras — daher meldet das Array-Panel `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` und blockiert die Verbindung. Bei**256**ist der Ring groß (**~13,5 MB, gemessen auf dem 10-GbE-Host im Labor**), was der RX-Pipeline echten Spielraum für Multi-Kamera-GVSP-Bursts bietet. (Ob eine bestimmte Konfiguration tatsächlich *eine Verbindung herstellt*, wird durch zwei Prüfungen entschieden — die **drain-aware**-Zulassungsprüfung und die**aggregate over-subscription** – nicht durch einen reinen Vergleich von Burst und Ring; siehe [Array-FPS- und Burst-Modell](#array-fps--burst-model).) |
| **Empfangs-URBs**– Schlüsselwort `PendingReceives` |**64** (max.) | USB-Anforderungsblöcke in der Übertragung; erhöhen Sie diesen Wert parallel zu den Empfangs-Puffern, um Bursts aufzunehmen. |
| **Jumbo-Frame** — Schlüsselwort `*JumboPacket` | **9014** | Erforderlich für 9000-Byte-GVSP-Pakete (6-mal weniger Pakete pro Frame als bei 1500). |

> ⚠️ **Eine Aktualisierung des Netzwerkkartentreibers SETZT diese erweiterten Eigenschaften auf die Standardwerte ZURÜCK.**Nach der Aktualisierung oder dem Austausch des Adaptertreibers müssen Sie**die Einstellungen erneut anwenden** `ReceiveBufferLen=256` und `PendingReceives=64` erneut an, da sonst das Array-Panel erneut sperrt, obwohl sich „an der Hardware nichts geändert hat“. Dies ist der häufigste Grund dafür, dass ein zuvor funktionierendes Rig plötzlich keine Verbindung mehr herstellt.

Wenden Sie die Befehle in einer **mit erhöhten Rechten ausgeführten** PowerShell an (ersetzen Sie den Namen Ihres Adapters, z. B. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` gilt für USB-10-GbE-Adapter.** Nun wird der Adaptertyp erkannt und das richtige „receive-ring“-Schlüsselwort eingestellt: `*ReceiveBuffers`→2048 für PCIe-Netzwerkkarten (Intel I219 usw.)oder `ReceiveBufferLen`→256 + `PendingReceives`→64 für den Realtek **USB**-10-GbE-Controller (der `*ReceiveBuffers` nicht bereitstellt). Die Zielwerte werden auf den vom jeweiligen Treiber gemeldeten Maximalwert (`NumericParameterMaxValue`) begrenzt, sodass niemals ein Wert außerhalb desBereichs geschrieben wird. Führen Sie das Skript in einem **Terminal mit erhöhten Rechten** aus; wie bei jeder registrierungsbasierten Optimierung wird die Änderung erst nach einem Neustart des Adapters oder eines Systemneustarts wirksam. Die oben genannten manuellen Befehle `Set-NetAdapterAdvancedProperty` bleiben eine gute Alternative – sie werden live angewendet (Neubindung des Adapters) ohne Neustart.

### Netzwerk-Grundlagen (alle LATTICE-Verbindungen)

- **Adressierung:** Link-Local `169.254.0.0/16` (GigE Vision LLA). Der Host erhält eine statische `169.254.x.x/16`; Kameras und DAQ-E weisen sich selbst Adressen im selben Bereich zu. Kein DHCP/Gateway erforderlich.
- **Paketgröße:**Jumbo (9000) wird bevorzugt, aber die automatische Erkennung soll die Größe ermitteln – die automatische Erkennung misst bei jeder Verbindung neu und umgeht die 1500-Byte-ICMP-Obergrenze der Kamera bereits über eine GVSP-Prüfung, sodass sie auf Jumbo-Pakete zurückgreift, wo immer die Leitung diese tatsächlich überträgt. Verwenden Sie die feste Einstellung mit `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` nur, wenn Sie es besser wissen als die automatische Erkennung, und bevorzuge „pro Befehl“ gegenüber „permanent“: Ein Pin umgeht die Probe; wenn der Pfad also tatsächlich keine 9000 übertragen kann, läuft**jede** Erfassung mit `SC_ERR_TIMEOUT -1011` in eine Zeitüberschreitung (siehe [Umgebungsvariablen](#environment-variables)).
- **Der RX-Ring skaliert mit `ReceiveBufferLen`:**Bei der Standardeinstellung `32` beträgt der nutzbare Ring ~0,26 MB (zu klein für jede Multi-Kamera-Burst-Aufnahme); bei maximalem `256` ist er groß (~13,5 MB, gemessen auf dem 10-GbE-Host im Labor), was echten Spielraum bietet. Ob eine Konfiguration verbunden wird, entscheidet sich dann durch die „Drain-Aware“-Zulassungsprüfung**und** die unten beschriebene aggregierte Überbelegungsprüfung – nicht durch einen reinen Vergleich von Burst- und Ringgröße.

### Array-FPS- und Burst-Modell

So lesen Sie das Fenster „Array-Einstellungen“ (sowie `lattice analyze-array` / das SDK des `analyze_array_network`):

- **Der Burst wird pro Kamera im jeweiligen realen Pixelformat der Kamera summiert.**Mono-**M3M**-Kameras streamen**Mono12 (2 B/px)**;**M3C**-Bayer-Kameras streamen 8- oder 12-Bit (TRI032S gibt stillschweigend BayerRG12 aus, selbst wenn BayerRG8 angefordert wird). Ein Vollauflösungs-Frame mit 4 Kameras ist also**~12,6 MB, wenn alle 8-Bit-Kameras verwendet werden, aber ~25 MB bei drei 12-Bit-Mono-Kameras**. Die Projektion ermittelt das Format jeder Kamera anhand ihres Modells (Identity-Cache), sodass der Burst dem tatsächlich übertragenen Datenfluss entspricht – und nicht einer pauschalen BayerRG8-Annahme.
- **Ein USB-Ethernet-Adapter ist unabhängig von den technischen Angaben auf 200 MB/s begrenzt.** Die Effizienz-Tabelle, die eine Verbindungsrate in einen dauerhaften Wert umrechnet, ist aus PCIeabgeleitet; eine USB-Netzwerkkarte gibt zwar ihre *Ethernet*-Verbindungsrate an, ist jedoch durch den USB-Bus und ihren Treiber begrenzt. Ein USB-10-GbE-Dongle erreichte früher ~1063 MB/s „Dauerleistung“ – ein Wert, der nie überprüft wurde –, und die daraus resultierende Taktrate führte zu einer Beschädigung von 6–18 % der Frames, während weiterhin eine einwandfreie Ziel-FPS-Rate gemeldet wurde. USB-gebundene Netzwerkkarten sind nun auf **200 MB/s** als absoluten (die Begrenzung liegt beim Bus, daher skaliert sie nicht mit der Nennleistung; ein USB-1-GbE-Adapter erreicht ca. 80 MB/s und ist davon nicht betroffen). `wire_ceiling_source` im Capability-Record besagt dies ausdrücklich, und `nic_is_usb` kennzeichnet es. Beides lässt sich mit `--wire-ceiling-mbps` außer Kraft setzen.
- **Die Zulassung ist drain-bewusst, nicht ganzheitlichBurst-vs.-Ring-Prinzip.** Ein gleichzeitiger Burst muss lediglich in den *transienten Rückstau* passen = `max(0, Σ per-cam arrival − host drain) × emit_window`, nicht der gesamte Burst. Bei einer Fabric mit schnellem Host und langsamen CAMs (ein **PCIe**-10G-Host + 4× 1-GbE-CAMs: Ankunft ≈ 320 MB/s, Drain ≈ 1063 MB/s) entleert der Host schneller, als die Kameras füllen; der Rückstau beträgt ≈ 0, sodass**Full-Res-Sim-Emit** **zulässt**, obwohl der 25-MB-Burst den 13,5-MB-Ring überschreitet. Schließt man dieselben vier Kameras an einen**USB**-10-GbE-Adapter an, beträgt die Abflussrate 200 MB/s statt 1063 — die Ankunft überholt ihn, und der Verlust zeigt sich als beschädigte Frames statt als niedrigere Bildrate. Auf einem 1-GbE-Host führt der DLThr-Mindestwert der Kameras von 31,25 MB/s dazu, dass die Ankunft die Abgabe überholt → es**blockiert** korrekt (für *diese* Blockklasse: ROI reduzieren oder Binning ≥ 2 verwenden). Die Zulassung erfolgt über eines von **zwei** Verbindungsgates – das andere ist die unten beschriebene Überprüfung der aggregierten Überbelegung.
- **Die prognostizierte Bildrate (fps) ist eine konservative Obergrenze für den seriellen Abruf.**Die Host-Abrufschleife zieht derzeit den Puffer jeder Kamera**seriell**ab (jeweils etwa ein Emit-Fenster pro Kamera), sodass der Zyklus durch `max(readout+emit, N × emit)` begrenzt ist, wobei das Emit pro Kamera auf die**Zugriffsverbindung**der Kamera geklemmt ist (1 GbE ≈ 80 MB/s) begrenzt, nicht durch den Host-Uplink. Bei einem 4Kameras mit voller Auflösung und 12-Bit-Array beträgt diese**~2,8 fps**, was den gemessenen ~2,7–3,0 fps entspricht. Die fps-Angabe ist bewusst**belichtungsunabhängig**, sodass in dunklen Szenen der tatsächliche Wert bei längerer Belichtungszeit leicht unter die Obergrenze sinken kann. Der serielle Abruf ist der eigentliche fps-Begrenzer; eine parallelewürde die Obergrenze in Richtung der Einzelausgaberate anheben.
- **Eine aggregierte Überbelegung ist ein hartes Verbindungshindernis.**Die Bandbreitenzuweisung pro Kamera liegt bei mindestens**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), sodass bei Erreichen dieser Untergrenze der Gesamtbedarf (`per_cam × N`) die**kollisionssichere Leitungsobergrenze**(`sustained × sim_emit_factor`) überschreiten. Praktische Obergrenzen bei voller Auflösung über 1 GbE:**6 Kameras bei 1500 MTU, 9 mit Jumbo**. Diese Obergrenze ist allein eine Eigenschaft der Leitung und der Untergrenze – sie ist**unabhängig von der Frame-Größe**, daher**helfen Binning und kleinere ROIs NICHT** (sie senken die Bytes pro *Frame*, nicht die GevSCPD-gesteuerten Bytes pro *Sekunde*); die einzigen Abhilfemaßnahmen sind weniger Kameras, Jumbo-Frames durchgehend oder eine schnellere Netzwerkkarte. Das Symptom wäre ein Paketverlust bei GVSP, keine sanfte Verringerung der Bildrate, daher setzt `analyze-array` die Werte für die erreichbaren fps auf Null und gibt `**OVER-SUBSCRIBED**` aus, und `array-connect` mit einer festgelegten Auflösung **verweigert die Verbindung** (der „Walk-Down“-Vorgang bindet Frames ansonsten herunter, was diese Art von Block ebenfalls nicht beseitigt). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` stuft die Verbindungsverweigerung bei Benchmark-Tests auf eine deutliche Warnung herab – siehe [Umgebungsvariablen](#environment-variables).

### Array-Zustand – welches Subsystem verliert Frames

Das `GET /api/camera/array/<array_id>/capability` eines verbundenen Arrays enthält einen aktiven
`health`-Block, der in einem rollierenden **10-Sekunden**-Fenster neu ausgewertet wird. Er unterteilt den Frame-Verlust
in die beiden Ursachen, die gegensätzliche Abhilfemaßnahmen erfordern, anstatt eine einzige „unvollständige“
Rate zu melden, die keine der beiden nennt:

| Feld | Bedeutung | Betroffenes Subsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pro serieller Schnittstelle) | Der Frame **kam an und wies strukturelle Fehler auf**— GVSP-Paketverlust. |**Netzwerk**: Leitungsbandbreite, Pacing, NIC-RX-Ring, MTU |
| `never_arrived_rate_pct` (pro serieller Schnittstelle) | Das Frame **ist überhaupt nicht angekommen**— die Kamera hat nicht ausgelöst, oder es wurde nichts gesendet. |**Trigger / Synchronisation**: M8-Kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Die schlechteste Ausfallrate der jeweiligen Kamera. | — |
| `per_cam_rate_pct` | Kombinierte Ausfallrate pro Kamera (beide Ursachen zusammen). | — |
| `stable_for_seconds` | Wie lange jede Kamera unter 0,01 % geblieben ist. | — |

Oberhalb von 5 % protokolliert das Backend eine Zeile „`[array-health <id>] WARN`“, in der die Aufteilung benannt wird – bei der
ersten Überschreitung, bei einer Änderung des Schweregrads, einmal pro Minute, solange sie andauert, und einmal, wenn
sie behoben ist. Die fehlerhafte Hälfte gibt „`[gvsp-corrupt <SN>]`“ beim ersten Auftreten pro Kamera und
Ursache aus, gefolgt von einer Zusammenfassung alle 60 Sekunden. Jede Auswertung landet weiterhin in der Backend-Protokolldatei;
die Zähler werden bei jedem Puffer weitergetreten, unabhängig davon, was ausgegeben wird.

Derselbe Datensatz gibt die Nummer an, unter der die gesamte Zuweisung hängt:

| Feld | Bedeutung |
| --- | --- |
| `wire_ceiling_mbps` | Das derzeit gültige, nachhaltige Bandbreitenbudget des Hosts, MB/s. |
| `wire_ceiling_source` | Woher dieser Wert stammt, in Worten – z. B. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` oder `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, wenn `--wire-ceiling-mbps` (oder das Feld **Wire Budget** der GUI) diesen Wert festlegt. |
| `nic_is_usb` | `true` für einen USB-Ethernet-Adapter — siehe die oben genannte Obergrenze von 200 MB/s. |

**Deutung:** Ein Wert ungleich Null für `gvsp_corrupt_rate_pct` bei einem Wert von 0 für `never_arrived_rate_pct`
bedeutet, dass Triggerung und Kabelsynchronisation einwandfrei funktionieren und 100 % des Verlusts auf dem Netzwerkpfad
liegt — Verringern Sie den Wert von `--wire-ceiling-mbps` und stellen Sie die Verbindung neu her. Das umgekehrte Muster deutet stattdessen auf das
Synchronisationskabel oder die Triggerleitung hin.

> **`--target-fps` ist nicht der Hebel für beschädigte Frames.** Das GevSCPD-Pacing wird
> einmalig bei der Verbindung geschrieben, daher verändert eine Verringerung der Triggerrate den Tastgrad und nicht die
> Burstrate bei simultaner Übertragung. Eine gemessene 5-fache Reduzierung der Anforderung brachte keine Verbesserung;
> die Senkung der Leitungsobergrenze von 240 auf 200 MB/s senkte bei derselben Anlage den Anteil der beschädigten Frames von 10,4 %
> auf 0,00 %.

> **Eine automatische Verkleinerung während des Betriebs ist bei der TRI032S-Firmware nicht verfügbar.** Ein laufendes Array
> kann dieses Problem nicht selbst beheben; trennen Sie die Verbindung und stellen Sie sie erneut her, damit der Verbindungszeit-Picker
> mit der neuen Obergrenze neu planen kann.

### Symptom → Behebung

| Symptom (Array-Einstellungen / Verbindung / `analyze_array_network`) | Ursache | Behebung |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` wird auf 32 zurückgesetzt (typischerweise nach einem Treiber-Update) | Setzen Sie `ReceiveBufferLen`→256, `PendingReceives`→64; das Panel erneut öffnen (das Backend neu starten, falls es die alte Ringgröße zwischengespeichert hat) |
| Neustart/Herunterfahren hängen sich auf; später `Invalid class` WMI-Fehler, Netzwerkkarte lässt sich nicht aktivieren, USB-Laufwerke fehlen | Alter Realtek USB-10-GbE-Treiber von 2016 → BSOD `0x9F` → erzwungene Ausschaltvorgänge | Adaptertreiber auf ≥ v10.67 (2026) aktualisieren, dann die oben genannten Empfangsring-Einstellungen erneut anwenden |
| Verbindung wird erfolgreich hergestellt, gibt jedoch eine Auflösung unterhalb der nativen Auflösung zurück | Smart-Prep verkleinert automatisch– hat den Frame verkleinert, um ihn an die Leitung anzupassen | Verbindung aufwerten / Verkleinerung akzeptieren / `--force-tier slip-emit-and-capture` |
| Das Array meldet eine intakte Ziel-FPS, liefert jedoch nur einen Bruchteil davon; `health.gvsp_corrupt_rate_pct` ungleich Null, `never_arrived_rate_pct` 0 | Das vom Host abgeleitete Leitungsbudget überschätzt die tatsächlich verfügbare Kapazität (typisch bei einem USB-Ethernet-Adapter, einer schmalen PCIe-Lane oder einer gemeinsam genutzten Fabric) | Stellen Sie die Verbindung mit einem niedrigeren `--wire-ceiling-mbps`-Wert wieder her und überprüfen Sie den Integritätsblock erneut. **Nicht** `--target-fps` – GevSCPD-Pacing ist bei der Verbindung festgesetzt |
| Kameras fehlen in veröffentlichten Gruppen; `health.never_arrived_rate_pct` ungleich Null, `gvsp_corrupt_rate_pct` 0 | Trigger-/ Synchronisationspfad – die Kameras lösen nicht aus, kein Netzwerkproblem | Überprüfen Sie das M8-Synchronisationskabel und `--line`; stellen Sie sicher, dass jedes Mitglied scharfgeschaltet ist (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` in `analyze-array` überschritten oder Verbindungsablehnung bei festgelegter Auflösung (`array over-subscribes the wire`) | Der aggregierte Bedarf pro Kamera (8 MB/s Mindestwert × N Kameras) überschreitet die kollisionssichere Leitungsgrenze – 6 Kameras in voller Auflösung bei 1 GbE mit 1500 MTU, 9 mit Jumbo-Frames | Weniger Kameras, Jumbo-Frames von-zu-End oder eine schnellere Netzwerkkarte. **ROI/Binning hilft NICHT** (die Obergrenze ist unabhängig von der Frame-Größe). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` überschreibt auf dem Teststand (akzeptiert den Paketverlust) |

---

## `chloros-cli daq`

Befehle für den Spektralsensor. Zwei Klassen:
- **`pool-*`**— „dünne“ HTTP-Clients, die den Sensor über den persistenten Pool des Backends ansteuern.**Dies ist der unterstützte Pfad und der einzige, der im ausgelieferten CLI vorhanden ist.** Das Backend verwaltet den Transport, sodass die GUI sowie die Skripte CLI und SDK sich alle ein einziges aktives Handle teilen, anstatt sich um die serielle Schnittstelle zu streiten.
- **Alles andere**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — direkter Hardwarezugriff, der der Vollständigkeit halber im Folgenden dokumentiert wird. Hierfür wird das Paket `daq` Python benötigt, das**in keinem ausgelieferten Artefakt enthalten ist**: Das kompilierte CLI schließt es aus (`scripts/Build-CLI.ps1` setzt `--nofollow-import-to=daq`, und die Transporte `pyserial` / `bleak` / `zeroconf` zusammen damit), und auch das PyPI-Paket SDK enthält es nicht. Sie laufen nur aus einem Quellcode-Checkout heraus, betrachten Sie sie daher eher als einen MAPIR-internen Entwicklungspfad und nicht als etwas, auf das man zurückgreifen sollte.
- **`discover` / `list`** liegen dazwischen: Es handelt sich um direkte Hardwarebefehle aus einem Quellcode-Checkout, aber bei einem ausgelieferten Build greifen sie auf `pool-discover` zurück, und das Backend führt den Scan durch. Das Scannen funktioniert also überall – was wichtig ist, da es die einzige Möglichkeit, die BLE-MAC-Adresse eines DAQ-M zu ermitteln.

> **`chloros-cli daq --help`** (sowie `-h` / `help`) listet die `pool-*`-Unterbefehle auf – die Hilfe wird bewusst an den Pool-Client weitergeleitet, damit sie die tatsächlich ausgeführten Befehle widerspiegelt. Wenn Sie einen „direct-hardware“-Unterbefehl in einer ausgelieferten Version auf, wird dieser mit einer expliziten Fehlermeldung beendet, die das fehlende Paket nennt und Sie auf `pool-*` verweist; nichts schlägt stillschweigend fehl. (`discover` / `list` bilden hier eine Ausnahme – sie leiten zu `pool-discover` weiter und funktionieren einfach.)
>
> **Alles, was ein Kunde benötigt, ist über `pool-*` erreichbar** — Verbinden, Streamen, Aufzeichnen kalibrierter `.daq`-Dateien und Austauschen von Kappenprofilen. Das DAQ kann auch über Python mit `chloros_sdk.connect_daq_sensor()` angesteuert werden, das denselben gepoolten Pfad verwendet.

### Arbeitsablauf für die erstmalige Verbindung mit einem DAQ-Sensor

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`-Referenz

| Unterbefehl | Zweck |
| --- | --- |
| `daq pool-connect` (smart-detect) | Einen Sensor im Backend-Pool öffnen. |
| `daq pool-connect --port PORT` | DAQ-U an einem bestimmten seriellen Port. |
| `daq pool-connect --ble` | DAQ-M über BLE, MAC automatisch gescannt. |
| `daq pool-connect --mac MAC` | DAQ-M über BLE mit bekanntem MAC (setzt `--ble` voraus). |
| `daq pool-connect --eth-host HOST` | DAQ-E über Ethernet bei bekanntem Host. |
| `daq pool-connect --eth` | DAQ-E über Ethernet, Host automatisch erkannt (mDNS + ARP-Fallback; funktioniert auch bei leerem ARP-Cache auf Windows und Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Integrationsfenster / AE-Zustand anpassen. |
| `daq pool-connect --no-stream` | Verbinden, aber noch nicht mit dem Streaming beginnen (Fortsetzung mit `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` |-Korrekturprofil. Standardwert im Backend ist `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Alle Transporte nach Sensoren durchsuchen, mit denen eine Verbindung hergestellt werden könnte, ohne eine Verbindung herzustellen. **So finden Sie die BLE-MAC-Adresse eines DAQ-M.** `daq discover` / `daq list` leiten in den ausgelieferten Builds automatisch hierher weiter. Bereits im Pool geöffnete Sensoren werden nicht aufgelistet – ein verbundenes DAQ-M beendet das Advertising — verwenden Sie für diese daher `pool-list`. |
| `daq pool-list` | Zeigt jeden Sensor im Backend-Pool an. |
| `daq pool-disconnect --sensor-id ID [--all]` | Freigabe. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Die neuesten N Spektrum-Frames. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Streaming fortsetzen / anhalten. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Eine .daq-Aufzeichnung starten / stoppen. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Das Cap-Korrekturprofil zur Laufzeit wechseln. |

### Direkte Hardware-Unterbefehle (nur im Quellcode-Checkout – nicht in ausgelieferten Builds enthalten)

> Der Vollständigkeit halber aufgeführt. Diese erfordern das Paket `daq` Python sowie `pyserial` / `bleak` / `zeroconf`, von denen keines im kompilierten CLI oder im PyPI-Build SDK enthalten ist – sie lassen sich nur aus einem MAPIR-Quellcode-Checkout ausführen. **Wenn Sie einen veröffentlichten Chloros-Build verwenden, nutzen Sie stattdessen die oben genannten `pool-*`-Befehle**; diese decken die Funktionen „connect“, „stream“, „record“ und die Cap-Auswahl ab.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Öffnen, verbinden und steuern Sie ein gespeichertes Chloros-Projekt (einen Ordner mit `cameras.json` + `sensors.json` + `project.json`). Alles wird über das Backend geleitet, sodass die GUI und CLI denselben Hardwarezustand erzeugen.

### Referenz zu Unterbefehlen

| Unterbefehl | Zweck |
| --- | --- |
| `project open PATH` | Das Gerätemanifest des Projekts ausgeben (Kameras, Arrays, Sensoren). |
| `project devices PATH [--reconnect]` | Erkennung auflisten oder erneut ausführen. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Alle gespeicherten Kameras / Arrays / Sensoren verbinden. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Einzelaufnahme von einer benannten Kamera oder Array. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | N-Frame-Burst von einer benannten Kamera oder einem Array (`-n/--count` Standardwert 5; `-i/--interval` Sekunden zwischen den Frames, Standardwert 0). Array-Serien entfernen Duplikate wiederholter synchronisierter Gruppen (Veraltungs-Watchdog), sodass ein Array mit Teilzyklus nicht N Kopien eines Einzelbildes zurückgeben kann; gibt die Ergebnisse pro Iteration aus. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Stream-toFestplatte über einen Backend-Job. `--poll-interval` = Sekunden zwischen `/stats`-Abfragen (Standardwert 2,0). |
| `project sensor read PATH NAME [--json]` | Neuester Spektrum-Frame. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | .daq-Datei aufzeichnen. |
| `project run PATH RECIPE.yaml` | Ein YAML/JSON-Erfassungsrezept ausführen. `--dry-run` validiert, ohne es auszuführen. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Berechne die Ausrichtung für ein Array – siehe [die untenstehende Flag-Tabelle](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Aktuelles Ausrichtungsprofil ausgeben. |
| `project align clear PATH NAME` | Das zwischengespeicherte Profil löschen. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Die Transformation eines Slaves verschieben. |
| `project align export PATH NAME --to FILE` | Das Profil in JSON speichern. |
| `project align import PATH NAME --from FILE [--no-validate]` | Ein gespeichertes Profil laden. |

#### `project align calibrate`-Optionen

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Ausrichtungsmethode. **Diese Schreibweisen unterscheiden sich von `lattice align-calibrate`**, die die Kurzformen `orb` / `akaze` / `phase` verwendet; die beiden Befehle sind bei diesem Flag nicht austauschbar. |
| `--model {translation, rigid, affine, homography}` | `affine` | Modell so transformieren, dass es passt. |
| `--frames N` | `1` | Synchronisierte Einzelbilder auf den Mittelwert anpassen. |
| `--reference SN` | Die Master- | Seriennummer der Referenzkamera; alle anderen Mitglieder werden darauf ausgerichtet. |
| `--max-features N` | `5000` | Obergrenze für die Anzahl der ORB-Merkmale. |
| `--ratio-threshold F` | `0.75` | Lowe-Verhältnis-Test. |
| `--ransac-threshold-px F` | `3.0` | RANSAC-Schwellenwert für Inlier. |
| `--min-matches N` | `15` | **Qualitätsschwelle** — Die Lösung wird abgelehnt, wenn die Anzahl der Inlier-Übereinstimmungen unter diesem Wert liegt. |
| `--max-reproj-err-px F` | `4.0` | **Qualitätsschwelle** — Die Lösung wird abgelehnt, wenn der RMS-Fehler bei der Reprojektion diesen Wert überschreitet. |
| `--checkerboard RxC` | — | Platinengeometrie für `--method checkerboard`, z. B. `9x6`. |
| `--name PROFILE` | leer | Profilname, eingebettet in das gespeicherte JSON eingebettet. **Nicht der Array-Name** – das ist der positionelle `NAME`. |

Die beiden Qualitätsprüfungen sind der Grund dafür, dass eine Kalibrierung zwar erfolgreich gelöst werden kann, das Speichern jedoch
verweigert wird: Ein Profil, das eine der beiden Prüfungen nicht besteht, würde jede
spätere Aufnahme stillschweigend falsch registrieren, daher wird es abgelehnt, anstatt es zu speichern.

### Beispiele

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Rezept-DSL

`project run RECIPE.yaml` akzeptiert eine YAML- oder JSON-Datei, die eine Abfolge von Aktionen beschreibt:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Unterstützte Aktionen: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Die Aktion `burst` benötigt `name` (erforderlich), `count` (Standardwert 5), `interval` (Sekunden, Standardwert 0), `output`, `format` und `settings` (gleiche kameraspezifische Einstellungsstruktur wie `apply`); Array-Bursts verwenden denselben Watchdog für frisch synchronisierte Gruppen wie `project burst`.

Ausführung:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Umgebungsvariablen

| Variable | Wirkung |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Überschreibt das Backend URL (Standard: `http://127.0.0.1:5000`) – **wird nur von den Befehlsfamilien `lattice`, `project` und `daq pool-*` berücksichtigt.** Die Kernbefehle (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) PIN `http://127.0.0.1:<port>` und diese Variable ignorieren (das IPv4-Literal umgeht die Windows `localhost`→`::1`-Strafe von ~2 s pro Anfrage), sodass sie immer den lokalen Rechner als Ziel anvisieren. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` stuft die Verbindungsablehnung bei Überbelegung des Arrays (aggregierte Nachfrage pro CAM &gt; kollisionssichere Leitungskapazitätsgrenze mit `pin_resolution`) auf eine deutliche Warnung mit Fortsetzung, wobei GVSP-Paketverluste akzeptiert werden. Nur für Benchmark-Zwecke – siehe [Array-fps- und Burst-Modell](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Wird vom CLI selbst gesetzt; weist das Backend an, die parallele Verarbeitung zu aktivieren. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` überspringt die GVSP-Fallback-Prüfung (nur ICMP-Ergebnisse). **Dies deaktiviert Jumbo-Pakete, es dämpft nicht nur die Protokollausgabe** – die Kamera antwortet auf DF-Pings nur bis zu 1500 auf jedem Pfad, daher ist dieser Test die einzige Möglichkeit, Jumbo-Pakete zu erkennen. Spart ca. 1 s pro Kamera und Verbindung; kostet ca. das 1,45-Fache der Leitungskapazität, wenn das Netzwerk Jumbo-Pakete hätte übertragen können. Der SDK gibt eine Warnung aus, wenn du diese Einstellung vornimmst. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Legt die GVSP-Paketgröße fest auf N Bytes; überspringt die Abfrage vollständig. Bevorzuge die Einstellung pro Befehl (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) gegenüber einer : Eine festgelegte Größe verhindert die Anpassung an das vorliegende Netzwerk, und die Festlegung von 9000 auf einem Pfad, der keine Jumbo-Pakete übertragen kann, führt dazu, dass **jeder** Erfassung mit `SC_ERR_TIMEOUT -1011` zu einem Timeout. |
| `TMPDIR` (Linux) | Nuitka-Verzeichnis für die „onefile“-Extraktion überschreiben. CLI verwendet automatisch `/mnt/ssd/tmp`, falls vorhanden. |

---

## Beendigungscodes

| Code | Bedeutung |
| --- | --- |
| `0` | Erfolgreich. |
| `1` | Allgemeiner Fehler (die meisten Unterbefehlsfehler). |
| `2` | Argumentfehler. |
| `130` | Durch Strg+C unterbrochen. |

---

## Hinweise zur Fehlerbehebung

- **„Anmeldung erforderlich“** → Führen Sie `chloros-cli login EMAIL PASSWORD` einmal auf diesem Rechner aus.
- **„Backend nicht erreichbar“** → Starten Sie die Desktop-App Chloros oder führen Sie die Backend-Binärdatei direkt aus (`chloros-backend`) aus, oder prüfen Sie `CHLOROS_BACKEND_URL`, falls es sich um einen Remote-Zugriff handelt.
- **`lattice`-Befehle schlagen mit der Meldung „LATTICE-Kameratreiber nicht gefunden“** → Die Arena SDK-Laufzeitumgebung istnicht installiert; die CLI wird mit `win32api` auf Windows gebündelt ausgeliefert, aber die C-Laufzeitumgebung ist Teil des GUI-Installationsprogramms.
- **In den Array-Verbindungs- / Array-Einstellungen zeigen „FRAMES WILL DROP“ oder „Reduce ROI to enable“ an** → Der Empfangsring der Host-Netzwerkkarte ist zu klein (wird nach einer Aktualisierung des Netzwerkkartentreibers üblicherweise auf 32 zurückgesetzt). Siehe [Einrichtung und Optimierung der Host-Netzwerkkarte](#host-nic-setup--tuning-lattice-arrays) – setzen Sie `ReceiveBufferLen=256` und `PendingReceives=64`.
- **Der Rechner hängt sich beim Neustart/Herunterfahren auf, dann WMI `Invalid class` / Netzwerkkarte lässt sich nicht aktivieren / USB-Laufwerke fehlen** → Veralteter Treiber für den USB-10-GbE-Adapter verursacht `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aktualisieren Sie den Adaptertreiber – siehe [Einrichtung und Optimierung der Host-Netzwerkkarte](#host-nic-setup--tuning-lattice-arrays).
- **Jetson-Swap-Warnung** → Fügen Sie einen dateibasierten Swap hinzu; der Befehl „CLI“ gibt genau die Befehle „`fallocate`“ / „`swapon`“ aus.
- **DAQ-Direktbefehle fehlen** → Erwartet: Das mitgelieferte `chloros-cli` schließt das `daq`-Paket bewusst aus, sodass nur `pool-*` vorhanden ist (das PyPI-Paket „SDK“ enthält es ebenfalls nicht). Verwenden Sie „`pool-*`“, das denselben Sensor über das Backend ansteuert, oder `chloros_sdk.connect_daq_sensor()` aus Python.

---

## Siehe auch

- [Python SDK-Referenz](sdk-reference.md) – programmatisches Äquivalent zu jedem CLI-Befehl.
- [DAQ-Sensorhandbuch](../daq/README.md) – sensorspezifische Verkabelung und Kalibrierung.
- Online-Dokumentation: `https://mapir.gitbook.io/chloros/cli`
