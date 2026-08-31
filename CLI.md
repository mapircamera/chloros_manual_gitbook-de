# CLI: Befehlszeile

> **Vollständige Referenz:**[CLI Reference](reference/cli-reference.md) dokumentiert**jedes Flag jedes Unterbefehls** und ist für KI-Assistenten optimiert – fügen Sie den URL in Ihren Assistenten ein und fragen Sie nach einem funktionierenden Befehl: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tipp für KI-Tools:** Jede Seite dieses Handbuchs ist als reines Markdown verfügbar, indem man `.md` an die entsprechende URL anhängt (z. B. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), und „`https://mapir.gitbook.io/chloros/llms.txt`“ indiziert das gesamte Handbuch für die Verwendung durch LLMs.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Was ist „CLI
“?

`chloros-cli` ist das Befehlszeilen-Frontend für dieselbe Verarbeitungs-Engine, die auch die Desktop-App „Chloros
“ verwendet. Es handelt sich um einen Thin-HTTP
-Client, der über das Backend „Chloros
“ (ein lokaler Server auf `127.0.0.1:5000`) läuft – die meisten Befehle starten das Backend automatisch, sodass ein einziger Aufruf von `chloros-cli process …` für ein Skript ausreicht.

Sie läuft unter **Windows
10/11 (x64)**und**Linux
(x86_64 sowie NVIDIA Jetson arm64 auf JetPack 6)** in jedem Terminal, ohne dass eine grafische Benutzeroberfläche erforderlich ist. Überprüfen Sie Ihre Installation mit:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Die Befehlsfamilien im Überblick:

* **Verarbeitung &amp; Konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 Sprachen – siehe [Unterstützte Sprachen](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (nurLinux
/Jetson)
* **Live-Hardware** — `lattice` (LATTICE-Kamerasteuerung, über 45 Unterbefehle), `daq pool-*` (DAQ-Lichtsensoren), `time-sync` (PTP)
* **Automatisierung** — `project` (Ausführung eines gespeicherten „Chloros
“-Projekts im Headless-Modus, einschließlich YAML-Erfassungsrezepten)

Wichtige globale Optionen: `--port N` (Backend-Port, Standard: `5000`), `-v/--verbose`, `--restart` (Neustart des Backends erzwingen), `--backend-exe PATH`. Eine vollständige Liste finden Sie in der [CLI
-Referenz](reference/cli-reference.md).

***

## Installation

CLI
**ist auf jeder Plattform imChloros
-Installationsprogramm enthalten** – es gibt keinen separaten Download fürCLI
. Laden Sie das Installationsprogramm von der [Download-Seite](download.md) herunter.

###Windows


Das Installationsprogramm legtCLI
unter folgendem Pfad ab:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

und fügt diesen Ordner zu Ihrem System hinzu `PATH` – **öffnen Sie nach der Installation ein neues Terminal**, damit das aktualisierte `PATH` erkannt wird. Das Installationsprogramm legt außerdem Startskripte (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) im Installationsstammverzeichnis sowie eine**Chloros
CLI
** Startmenü-Verknüpfung ab, die jeweils ein Terminal mit dem einsatzbereiten `chloros-cli` öffnen.

###Linux


Installieren Sie die für Ihre Architektur geeignete Version von `.deb`:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Dadurch wird `chloros-cli` bis `/usr/bin/chloros-cli` (bereits auf `PATH`) und das Backend auf `/usr/lib/chloros/chloros-backend`, zusammen mit der für LATTICE-Kameras erforderlichen Arena-SDK
-Laufzeitumgebung. Weitere Informationen finden Sie unter [Linux
Installation](linux/linux-installation.md).

### Überprüfen

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Anmeldung &amp; Lizenzierung

CLI
(sowiePython
SDK
) erfordern einen **kostenpflichtigenChloros
+-Tarif**– dieser ist in allen kostenpflichtigen Tarifen enthalten, nicht jedoch im kostenlosen Tarif. Die Beschränkung wird**serverseitig** durch das Backend durchgesetzt, nicht durch dieCLI
-Binärdatei: Ein Aufruf im abgemeldeten Zustand wird mit dem Fehlercode `401 AUTH_REQUIRED` abgelehnt, und ein Aufruf im angemeldeten Zustand in der kostenlosen Stufe mit dem Fehlercode `403 PLAN_UPGRADE_REQUIRED`, unabhängig davon, ob er von `chloros-cli`, demSDK
oder einem selbst entwickeltenHTTP
-Client stammt. Upgrade unter [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Melden Sie sich **einmal pro Rechner** an:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Passwörter mit Sonderzeichen**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` wird von der Shell verfälscht (CLI
erkennt dies anhand eines 401-Fehlers und versucht es automatisch erneut, aber einfache Anführungszeichen vermeiden das Problem vollständig).
{% endhint %}

Die Sitzung wird unter `~/.chloros/user_session.json` zwischengespeichert und funktioniert offline für die Nachfrist des Tarifs weiter (30 Tage bei monatlichen Tarifen, bis zum Ablauf bei jährlichen Tarifen). `chloros-cli status` funktioniert auch ohne kostenpflichtigen Tarif, sodass der Grund für eine Ablehnung immer sichtbar ist.

{% hint style="danger" %}
**Headless-Aufträge planen? Melden Sie sich zuerst an.**Ein Befehl zum Starten eines Backends (`process`, `status`, `export-status`, …) ohne zwischengespeicherte Sitzung**nicht sofort fehlschlägt**– es wechselt stattdessen zu einer interaktiven `Email:`-/`Password:`-Eingabeaufforderung über stdin. Ein unbeaufsichtigter Cron-Job oder CI-Schritt**hängt daher beim Warten auf Eingaben**. Führen Sie `chloros-cli login EMAIL 'PASSWORD'` einmal auf dem Rechner aus, bevor Sie irgendetwas planen.
{% endhint %}

***

## Ihr erster Verarbeitungsdurchlauf

Richten Sie `process` auf einen Ordner mit Erfassungen – es erkennt automatisch „Survey3
“ (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` oder eine Mischung aus beidem:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Der Fortschritt wird live pro Pipeline-Thread (Erkennen, Analysieren, Verarbeiten, Exportieren) angezeigt, und ein erfolgreicher Durchlauf endet mit der Meldung, wie viele Bildprodukte geschrieben wurden (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Wo die Ergebnisse gespeichert werden

`process` schreibt in einen **Projektordner**, nicht in Ihren Eingabeordner:

* Ohne `-o`: Das Projekt wird unter Ihrem Standard-Projektordner angelegt (gemeinsam mit der GUI; verwalten Sie es mit `get-project-folder` / `set-project-folder`, Fallback `~/Chloros Projects`) und wird, falls nicht angegeben, nach `-n/--project-name` oder einem Zeitstempel (`YYYYMMDD_HHMMSS`) benannt.
* Bei Verwendung von `-o PATH`: Dieser Ordner **ist** der Projektordner. Wenn er bereits eine Datei `project.json` enthält, wird anstelle eines Überschreibens eine Schwesterdatei mit dem Suffix `_1`/`_2`… erstellt.

Innerhalb des Projekts werden die Produkte **nach Kamera und anschließend nach Dateiformat** gruppiert:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Der Kameraordner lautet `LATT-<sensor>-<lens>-F<filter>` für LATTICE (entsprechend dem EXIF-Datums- und Uhrzeitstempel der Aufnahme `Model`) und `<model>_<filter>` (z. B. `Survey3N_RGN`) für „Survey3
“. Der Formatordner folgt nach `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` oder `tiff32` für `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Jedes exportierte Produkt behält den Namen der QUELLDATEI bei.**Ein Radiance-Export von `capture_..._raw.tif` heißt weiterhin `capture_..._raw.tif` – er befindet sich lediglich im Ordner `tiff32/Radiance_Images/`.**Der Ordner identifiziert das Produkt, nicht der Dateiname**, suchen Sie also nach dem Verzeichnis und nicht nach dem Suffix „`*radiance*`“.
{% endhint %}

### Die Optionen, die Sie tatsächlich verwenden werden

| Flag | Standard | Funktion |
| --- | --- | --- |
| `-o, --output PATH` | Standard-Projektordner | Speicherort des Projektordners (siehe oben). |
| `-n, --project-name NAME` | Zeitstempel | Projektname. |
| `--format FMT` | `TIFF (16-bit)` | Eine der folgenden Optionen: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | keine | Zu exportierende Vegetationsindizes (siehe [Vegetationsindizes](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuronales Debayering, langsamer, höchste Qualität (Chloros
+, NVIDIA-GPU). |
| `--vignette / --no-vignette` | ein | Vignettierungskorrektur. |
| `--reflectance / --no-reflectance` | ein | Reflektanzkalibrierung; bei LATTICE dient dies auch als Umschalter für das Reflektanzprodukt. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Erzwingt den Pipeline-Einstiegspunkt für LATTICE-TIFFs. |

Für alles andere – Feinabstimmung der Zielerkennung, PPK, Belichtungspins, Flags zur Array-Ausrichtung – siehe den Abschnitt [`process` der „CLI
“-Referenz](reference/cli-reference.md).

***

## Auswahl der zu exportierenden Daten (LATTICE-Produkte)

Die LATTICE-Verarbeitung verzweigt sich **in einem Durchgang auf jedes anwendbare Produkt**. Vier produktbezogene Schaltflächen sind**standardmäßig aktiviert**; verwenden Sie das Formular `--no-`, um eine davon zu deaktivieren:

| Schaltfläche | Produkt |
| --- | --- |
| `--debayered` | Lineares Demosaicing → `Debayered_Images/` |
| `--preview` | Vorschau anzeigen (Weißabgleich + Gamma; Falschfarben-Streckung für Multispektralbilder) → `Preview_Images/` |
| `--radiance` | float32 Strahlungsintensität, W/m²/sr/nm → `Radiance_Images/` (immer `tiff32/`) |
| `--reflectance` | uint16 Reflexionsgrad, Pix4D-kompatibel → `Reflectance_Calibrated_Images/` |

RGB
Master-Kameras geben ausschließlich debayered + Vorschau aus – bandweise Radianz/Reflexionsgrad sind für einen Breitbandsensor nicht aussagekräftig, daher haben diese Umschaltoptionen für sie keine Auswirkung.Survey3
`.raw` ignoriert die Umschaltoptionen und folgt dem Standardpfad für Reflexionsgrad/Ziel.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (Standard: `auto`) wählt die Reflektanzreferenz aus: `auto` erstellt ein QA-konformes [Kalibrierungsziel](calibration-targets.md) innerhalb des Bildausschnitts als absolute Referenz und greift auf die vom DAQ-Lichtsensor ermittelte Downwelling-Teilung (ρ = π·L/E) zurück, wenn kein Ziel vorhanden ist; `target` ist streng (keine DAQ-Ersetzung); `daq` ist DAQ-maßgeblich. Gemessene Zielscans pro Einheit können mit `--target-reflectance-dir` bereitgestellt werden.

{% hint style="info" %}
**Auslesen von Reflexionspixeln:**Der DN-Wert, der ρ = 1,0 bedeutet, ist**pro Quelle** — LATTICE-Dateien fügen `Chloros:PixelScale=32768` in die XMP-Datei ein; „Survey3
“-Dateien verwenden 65535 (und enthalten keine `Chloros:*`-Tags). Lesen Sie das Tag aus und dividieren Sie damit, anstatt eine Konstante anzunehmen. Details und der eine bewusste Sonderfall ohne Skalierung finden Sie in der [CLI
-Referenz](reference/cli-reference.md).
{% endhint %}

**Die Verarbeitung beginnt immer bei `raw`.** Abgeleitete Produkte (Debayering-/Strahlungs- bzw. Reflexions-Exporte) werden niemals wieder in die Pipeline zurückgeführt – ein erneuter Import und eine erneute Verarbeitung würden die Kalibrierungsberechnungen doppelt anwenden, daher überspringtChloros
diese und gibt eine entsprechende Meldung aus. `--input-level` ist die bewusst eingeplante Ausweichmöglichkeit, wenn Sie wirklich einen Einstiegspunkt erzwingen müssen.

***

## Wenn ein Durchlauf fehlschlägt

Ab Version 1.2.0 meldet `process` einen deutlichen Fehler, anstatt „erfolgreich“ zu sein, ohne dass etwas angezeigt wird:

* Ein Lauf, der **Produkte angefordert, aber keine geschrieben hat**– nur `project.json` und `calibration_data.json` –, gibt `Processing finished but wrote no image products.` aus und**beendet den Lauf mit einem Wert ungleich Null**, sodass Skripte dies erkennen können. Die üblichen Ursachen: Der Eingabeordner wurde nicht als Aufzeichnung erkannt (überprüfen Sie das Layout und `--input-level`), oder jedes angeforderte Produkt war für diese Kameras nicht anwendbar (z. B. die Abfrage von Radianz/Reflexionsgrad bei Kameras, die ausschließlichRGB
-Daten liefern).
* Ein **absichtlich nur auf Metadaten ausgerichteter Durchlauf** (alle Produkte deaktiviert, kein `--indices`) gilt weiterhin als erfolgreich – eine leere Bildausgabe ist hier das korrekte Ergebnis.
* Führen Sie den Lauf mit `--verbose` erneut durch und überprüfen Sie das Backend-Protokoll auf Zeilen mit den Codes `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, die die kameraspezifischen Übersprünge erklären.

Exit-Codes: `0` Erfolg · `1` Allgemeiner Fehler · `2` Argumentfehler · `130` Durch Strg+C unterbrochen.

***

## Vegetationsindizes

Rufen Sie `--indices` mit einem oder mehreren voreingestellten Namen auf; jeder Index wird in einem eigenen `<INDEX>_Index_Images/`-Ordner abgelegt:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

Die 22 voreingestellten Namen, die `process --indices` akzeptiert:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Es gibt drei Indexlisten – verwechseln Sie diese nicht.**Das Dropdown-Menü „Projekteinstellungen“ der Benutzeroberfläche enthält 27 Formeln (ergänzt um `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – diese fünf sind ausschließlich in der GUI verfügbar und gelten**nicht** für `--indices`). Der Live-/Offline-Befehl `lattice index --preset` verwendet eine eigene, separate Liste mit 22 Voreinstellungen. Formeln und Bandberechnungen sind unter [Multispektrale Indexformeln](project-settings/multispectral-index-formulas.md) dokumentiert.
{% endhint %}

***

## DAQ-Lichtsensoren: Ein kurzer Überblick

Die `daq pool-*`-Familie steuert die spektralen DAQ-Sensoren „MAPIR
“ (DAQ-U über USB, DAQ-M über BLE, DAQ-E über Ethernet) über den persistenten Pool des Backends – die GUI,CLI
undSDK
nutzen alle denselben Live-Handle. **`pool-*` ist der unterstützte DAQ-Pfad in der ausgelieferten Version „CLI
“**; andere `daq`-Unterbefehle, auf die möglicherweise verwiesen wird, sind eine rein interne Oberfläche von „MAPIR
“ und enden mit einem expliziten Fehler, der auf `pool-*` verweist.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` ohne `--duration` läuft bis `pool-record --stop`; das Standardausgabeverzeichnis ist `~/Documents/DAQ Live View/` **auf dem Rechner des Backends**. Das Cap-Korrekturprofil wird bei der Verbindungsherstellung ausgewählt (`--cap-id`, Backend-Standard `sunshine_cosine`) und kann live mit `pool-set-cap` — Cap-Profile und der kalibrierte Messbereich des Sensors werden in den DAQ-Kapiteln dieses Handbuchs behandelt.

{% hint style="warning" %}
**DAQ-E auf einem Host mit mehreren Netzwerkkarten:** Die erste automatische Erkennung von `pool-connect --eth` nach dem Booten kann selbst bei einem fehlerfreien Sensor fehlschlagen. `--eth-host <ip-or-hostname>` ist die zuverlässige Variante – greifen Sie darauf zurück, wenn die Erkennung keine Ergebnisse liefert.
{% endhint %}

***

## LATTICE-Kameras, PTP und Projektautomatisierung

Die `lattice`-Familie (über 45 Unterbefehle) deckt den gesamten Arbeitsablauf mit LATTICE-Kameras ab: Erkennung, Einzelaufnahmen, persistente synchronisierte Arrays mit dem „Smart-Prep“-Verbindungsablauf der GUI, Live-Vorschau im Browser, Ausrichtung, Indexberechnungen und Host-Netzwerkkarten-Diagnose. Ein kleiner Vorgeschmack:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Daneben: `chloros-cli time-sync` liefert Informationen über den PTP-Grandmaster, den der „Chloros
“-Host betreibt (LATTICE-Kameras und DAQ-E-Sensoren sind diesem als Slaves zugeordnet, um geräteübergreifende Zeitstempel zu erhalten), und `chloros-cli project` öffnet ein gespeichertes „Chloros
“-Projekt und steuert dessen Kameras, Arrays und Sensoren im Headless-Modus an – einschließlich skriptgesteuerter YAML-Aufnahmerezepte.

Diese drei Familien (`lattice`, `project`, `daq pool-*`) sind zudem die einzigen, die `CHLOROS_BACKEND_URL` zur Steuerung eines **remoten** Backends unterstützen; die Kernbefehle zielen immer auf den lokalen Rechner ab.

Ausführliche Anleitungen finden Sie in den LATTICE-Kapiteln dieses Handbuchs; alle Flags sind in der [CLI
-Referenz](reference/cli-reference.md) aufgeführt.

***

## Fehlerbehebung: Die 5 häufigsten Probleme

| Symptom | Lösung |
| --- | --- |
| `Login required` oder ein geplanter Job bleibt bei einer `Email:`-Eingabeaufforderung hängen | Führen Sie `chloros-cli login EMAIL 'PASSWORD'` einmal auf diesem Rechner aus – Befehle ohne zwischengespeicherte Sitzung werden interaktiv ausgeführt, anstatt sofort abzubrechen. |
| `backend unreachable` | Starten Sie die Desktop-Anwendung „Chloros
“ oder führen Sie die Backend-Binärdatei direkt aus (`chloros-backend`). Wenn Sie `lattice`/`project`/`daq pool-*` auf ein Remote-Backend verweisen, überprüfen Sie `CHLOROS_BACKEND_URL`. |
| Array-Verbindung blockiert: `FRAMES WILL DROP` / `Reduce ROI to enable` | Empfangsring der Host-Netzwerkkarte auf Standardwerte zurückgesetzt – die häufigste Ursache dafür, dass ein zuvor funktionierendes Rig keine Verbindung mehr herstellt, typischerweise nach einer Aktualisierung des Netzwerkkartentreibers. Führen Sie `chloros-cli lattice network --fix` in einem **Terminal mit erhöhten Rechten** aus (oder setzen Sie `ReceiveBufferLen=256`, `PendingReceives=64`); siehe den Abschnitt *Einrichtung und Optimierung der Host-Netzwerkkarte* im Handbuch. |
| Der `daq`-Unterbefehl bricht ab: „erfordert das vollständige DAQ-Paket…“ | Bei ausgelieferten Builds zu erwarten – das kompilierte „CLI
“ enthält nur die `daq pool-*`-Familie, die die Funktionen „Connect“, „Stream“, „Record“ und die CAP-Auswahl abdeckt. Verwenden Sie `pool-*` (oder `chloros_sdk.connect_daq_sensor()` vonPython
). |
| Jetson gibt vor großen Ordnern eine Swap-Warnung aus | Dateibasierten Swap hinzufügen – unterCLI
werden die genauen Befehle `fallocate`/`swapon` angezeigt, die ausgeführt werden müssen. |

***

## Hilfe erhalten

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Jedes Flag, jeder Unterbefehl:** [CLI
Referenz](reference/cli-reference.md)
* **Äquivalent zuPython
:** [Python
SDK
](api-python-sdk.md) und die [SDK
Referenz](reference/sdk-reference.md)
* **Support:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
