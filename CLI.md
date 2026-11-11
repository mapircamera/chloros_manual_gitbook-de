# CLI: Befehlszeile

> **Vollständige Referenz:**Die [CLI-Referenz](reference/cli-reference.md) dokumentiert**jedes Flag jedes Unterbefehls** und ist für KI-Assistenten optimiert – fügen Sie die URL in Ihren Assistenten ein und fragen Sie nach einem funktionierenden Befehl: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tipp für KI-Tools:** Jede Seite dieses Handbuchs ist als reines Markdown verfügbar, indem man `.md` an die URL anhängt (z. B. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), und „`https://mapir.gitbook.io/chloros/llms.txt`“ indiziert das gesamte Handbuch für die Verwendung durch LLMs.

> **Möchtest du, dass dein Assistent Chloros selbst ausführt?** `chloros-mcp`, ein mit dem Python-SDK installierter MCP-Server, funktioniert mit Claude Desktop, Claude Code, Cursor und anderen MCP-Clients: siehe [Verwendung von Chloros mit KI-Assistenten](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Was ist die CLI
?

`chloros-cli` ist das Befehlszeilen-Frontend für dieselbe Verarbeitungs-Engine, die auch die Chloros
-Desktop-App nutzt. Es handelt sich um einen schlanken HTTP
-Client, der auf das Chloros
-Backend (ein lokaler Server auf `127.0.0.1:5000`) zugreift – die meisten Befehle starten das Backend automatisch, sodass ein einziger `chloros-cli process …`-Aufruf für ein Skript ausreicht. Das Backend bleibt dann für den nächsten Befehl aktiv: Es wird bei `chloros-cli logout` beendet (es sei denn, die App ist geöffnet oder ein Lauf bzw. ein Build ist im Gange), bei `chloros-cli backend stop` oder von selbst nach 30 Minuten ohne Client, Lauf oder verbundenes Gerät. `chloros-cli backend status` zeigt dies an.

Es läuft unter **Windows
10/11 (x64)**und**Linux
(x86_64 sowie NVIDIA Jetson arm64 auf JetPack 6)**, in jedem Terminal und ohne dass eine grafische Benutzeroberfläche erforderlich ist. Überprüfen Sie Ihre Installation mit:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Die Befehlsgruppen im Überblick:

* **Verarbeitung &amp; Konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 Sprachen – siehe [Unterstützte Sprachen](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (nur Linux
/Jetson)
* **Live-Hardware** — `lattice` (LATTICE-Kamerasteuerung, über 45 Unterbefehle), `daq pool-*` (DAQ-Lichtsensoren), `time-sync` (PTP)
* **Automatisierung** — `project` (Ausführung eines gespeicherten Chloros
-Projekts im Headless-Modus, einschließlich YAML-Erfassungsrezepten)

Wichtige globale Optionen: `--port N` (Backend-Port, Standard: `5000`), `-v/--verbose`, `--restart` (Backend zwangsweise neu starten), `--backend-exe PATH`. Eine vollständige Liste finden Sie in der [CLI
-Referenz](reference/cli-reference.md).

***

## Installation

Die CLI
**ist auf jeder Plattform im Chloros
-Installationsprogramm enthalten** – es gibt keinen separaten CLI
-Download. Laden Sie das Installationsprogramm von der [Download](download.md)-Seite herunter.

### Windows


Das Installationsprogramm installiert die CLI
unter:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

und fügt diesen Ordner einmalig zu Ihrem System `PATH` hinzu (ein Upgrade entfernt auch doppelte Einträge, die von früheren Installationsprogrammen hinterlassen wurden) – **öffnen Sie nach der Installation ein neues Terminal**, damit das aktualisierte `PATH` erkannt wird. Der Installer legt außerdem Startskripte (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) im Installationsstammverzeichnis sowie eine**Chloros
-CLI
** Verknüpfung im Startmenü ab, die jeweils ein Terminal mit dem einsatzbereiten `chloros-cli` öffnen.

### Linux


Installieren Sie die für Ihre Architektur geeignete Version von `.deb` mit `apt` (nicht `dpkg -i`, da dieses die empfohlenen Anforderungen des Python
-SDK
überspringt):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Dadurch wird `chloros-cli` bis `/usr/bin/chloros-cli` (bereits in `PATH` enthalten), das Backend und die CLI
-Programme, die in `/usr/lib/chloros/backend/` und `/usr/lib/chloros/cli/` entpackt sind (`/usr/lib/chloros/chloros-backend` ist ein Launcher, der das Backend startet), die Arena-SDK
-Laufzeitumgebung, die das Backend für LATTICE-Kameras benötigt, sowie das Python
-SDK
für das System `python3`. Weitere Informationen finden Sie unter [Linux
-Installation](linux/linux-installation.md).

### Überprüfen

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Anmeldung &amp; Lizenzierung

Der Zugriff über die CLI
(und das Python
-SDK
) erfordert einen **kostenpflichtigen Chloros
+-Tarif**– dieser ist in allen kostenpflichtigen Tarifen enthalten, im kostenlosen Tarif jedoch nicht. Die Beschränkung wird**serverseitig** durch das Backend durchgesetzt, nicht durch die CLI
-Binärdatei: Ein Aufruf ohne Anmeldung wird mit `401 AUTH_REQUIRED` abgelehnt, und ein Aufruf im angemeldeten Zustand in der kostenlosen Stufe mit dem Fehlercode `403 PLAN_UPGRADE_REQUIRED`, unabhängig davon, ob er von `chloros-cli`, dem SDK
oder einem selbst entwickelten HTTP
-Client stammt. Führen Sie ein Upgrade unter [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) durch.

Melden Sie sich **einmal pro Rechner** an:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Passwörter mit Sonderzeichen**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` wird von der Shell verfälscht (die CLI
erkennt dies anhand eines 401-Fehlers und versucht es automatisch erneut, aber einfache Anführungszeichen vermeiden das Problem vollständig).
{% endhint %}

Die Sitzung wird unter `~/.chloros/user_session.json` zwischengespeichert und funktioniert während der Nachfrist des Tarifs weiterhin offline (30 Tage bei monatlichen Tarifen, bis zum Ablauf bei jährlichen Tarifen). `chloros-cli status` funktioniert auch ohne kostenpflichtigen Tarif, sodass der Grund für eine Ablehnung immer sichtbar ist.

{% hint style="danger" %}
**Headless-Aufträge planen? Melden Sie sich zuerst an.**Ein Befehl zum Starten eines Backends (`process`, `status`, `export-status`, …) wird ohne**zwischengespeicherte Sitzung**ausgeführt, schlägt nicht sofort fehl – es wechselt zu einer interaktiven `Email:`-/`Password:`-Eingabeaufforderung über stdin. Ein unbeaufsichtigter Cron-Job oder CI-Schritt**hängt daher beim Warten auf Eingaben**. Führen Sie `chloros-cli login EMAIL 'PASSWORD'` einmal auf dem Rechner aus, bevor Sie irgendetwas planen.
{% endhint %}

***

## Ihr erster Verarbeitungslauf

Richten Sie `process` auf einen Ordner mit Erfassungen – es erkennt automatisch Survey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` oder eine Mischung daraus:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Der Fortschritt wird live pro Pipeline-Thread (Erkennen, Analysieren, Verarbeiten, Exportieren) angezeigt, und ein erfolgreicher Durchlauf endet mit der Meldung, wie viele Bildprodukte geschrieben wurden: `Image products written: N`, gefolgt von `(plus M clamp masks)`, wenn bei dem Durchlauf auch [Clamp-Masken](output-image-formats.md#clamp-masks) geschrieben wurden (diese werden nicht als Produkte gezählt).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Wenn bereits ein Chloros
-Backend läuft (z. B. das der Desktop-App), nutzt `process` dieses und öffnet sein Projekt darin. Solange die App ein von ihr geöffnetes Projekt geöffnet hat, lehnt es den Vorgang ab: Führen Sie den Vorgang über die App aus oder schließen Sie dieses Projekt. Wenn ein vorheriger Lauf noch nicht abgeschlossen ist, wartet es bis zu 15 Sekunden darauf, bevor es den Vorgang ablehnt. `--restart` erzwingt ein neues Backend. Unter Linux
wird ein Backend, das als systemd-Dienst läuft, niemals neu gestartet: `process` nutzt es so, wie es ist, und wo es andernfalls neu gestartet würde (`--restart` oder ein Import, der nichts gefunden hat), gibt es `sudo systemctl restart <unit>` aus und beendet sich mit `1`.

### Wo die Ausgaben gespeichert werden

`process` schreibt in einen **Projektordner**, nicht in Ihren Eingabeordner:

* Ohne `-o`: Das Projekt wird in Ihrem Standard-Projektordner angelegt (gemeinsam mit der GUI; verwalten Sie ihn mit `get-project-folder` / `set-project-folder`; wenn keiner festgelegt ist, `Documents/MAPIR/Chloros Projects` in Ihrem Benutzerordner), benannt nach `-n/--project-name` oder einem Zeitstempel (`YYYYMMDD_HHMMSS`), wenn dieser weggelassen wird.
* Bei `-o PATH`: Dieser Ordner **ist** der Projektordner. Wenn er bereits eine Datei mit dem Namen `project.json` enthält, wird anstelle eines Überschreibens eine Datei mit dem Suffix `_1`/`_2`… mit diesem Suffix erstellt, anstatt die Datei zu überschreiben. Ein relativer Pfad `PATH` wird relativ zu dem Ordner interpretiert, aus dem Sie `chloros-cli` ausführen.

{% hint style="info" %}
**Relative Pfade (geändert in 1.3.0).** Jeder Pfad, den das Backend öffnet – `-o`/`--output` bei jedem Befehl (einschließlich des Standardbefehls `output` der Erfassungsbefehle), `--input`, `--burst-dir`, `--profile`, `--save` und `--mask` — wird anhand des Ordners aufgelöst, aus dem Sie `chloros-cli` ausführen, sodass `-o captures/` in `<current folder>/captures/` schreibt. In Version 1.2.0 hat das Backend sie in seinem eigenen Installationsordner aufgelöst, was mit der Fehlermeldung „Zugriff verweigert“ fehlschlug. Bei einem entfernten Backend (`CHLOROS_BACKEND_URL`) werden die Pfade so gesendet, wie sie eingegeben wurden, und auf diesem Rechner aufgelöst.
{% endhint %}

{% hint style="warning" %}
**Windows
: Halten Sie den Projektordner kurz, es sei denn, lange Pfade sind aktiviert.** Wenn unter Windows
lange Pfade deaktiviert sind (Windows
-Standard), darf kein Dateipfad länger als 260 Zeichen sein, und Produkte befinden sich bis zu etwa 130 Zeichen unterhalb des Projektordners. `process` prüft dies vor dem Start: Ein zu tief verschachtelter Projektordner wird mit der Meldung **„Der Projektordner ist für diesen Computer zu tief“** abgelehnt, der längste zulässige Projektordner wird angegeben und es wird erklärt, wie lange Pfade aktiviert werden können – es wird nichts weiter verarbeitet. Verwenden Sie eine kürzere Version von `-o` (oder `set-project-folder`) oder aktivieren Sie lange Windows
-Pfade (die Option **„Windows
-Langpfade aktivieren“** des Installationsprogramms, die standardmäßig aktiviert ist) und starten Sie Chloros
neu.
{% endhint %}

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

Der Kameraordner lautet `LATT-<sensor>-<lens>-F<filter>` für LATTICE (entsprechend dem EXIF-Datumsformat der Aufnahme `Model`) und `<model>_<filter>` (z. B. `Survey3N_RGN`) für Survey3
. Der Formatordner folgt auf `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` oder `tiff32` für `TIFF (32-bit, Percent)`.

Bei einer multispektralen M3C-Kamera kann neben den Produktordnern auch ein Ordner „`clamp_masks/`“ in einem Formatordner erscheinen (zum Beispiel „`tiff16/clamp_masks/Reflectance_Calibrated_Images/`“ und „`tiff32/clamp_masks/Radiance_Images/`“). Er enthält die pixelweisen Masken der Strahlungs- und Reflexionsprodukte, bei denen die Bandentmischung einen negativen Wert auf Null begrenzt hat – siehe [Begrenzungsmasken](output-image-formats.md#clamp-masks). Kein Produktordner enthält jemals eine Maske.

{% hint style="info" %}
**Jedes exportierte Produkt behält den Namen der QUELLDATEI bei.**Ein Strahlungs-Export von `capture_..._raw.tif` heißt weiterhin `capture_..._raw.tif` – er befindet sich lediglich in `tiff32/Radiance_Images/`.**Der Ordner identifiziert das Produkt, nicht der Dateiname**, suchen Sie also nach dem Verzeichnis und nicht nach dem Suffix „`*radiance*`“.
{% endhint %}

### Die Optionen, die Sie tatsächlich verwenden werden

| Flag | Standard | Funktion |
| --- | --- | --- |
| `-o, --output PATH` | Standard-Projektordner | Speicherort des Projektordners (siehe oben). |
| `-n, --project-name NAME` | Zeitstempel | Projektname. |
| `--format FMT` | `TIFF (16-bit)` | Eine der folgenden Optionen: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros
führt keine Hochskalierung durch: Bei reinen 8-Bit-Bildern wird ein tieferes Format als `TIFF (8-bit)` geschrieben, und dies wird einmal angegeben. |
| `--indices NAME [NAME ...]` | keine | Zu exportierende Vegetationsindizes (siehe [Vegetationsindizes](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuronales Debayering, langsamer, höchste Qualität (Chloros
+, benötigt eine CUDA-GPU: Ohne diese wird `standard` verwendet; wird für LATTICE-Aufnahmen abgelehnt, die immer `standard` erhalten). |
| `--vignette / --no-vignette` | ein | Vignettierungskorrektur. |
| `--reflectance / --no-reflectance` | ein | Reflexionskalibrierung; bei LATTICE dient dies auch als Umschalter für das Reflexionsprodukt. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Erzwingt den Pipeline-Einstiegspunkt für LATTICE-TIFFs. |

Für alles andere – Feinabstimmung der Zielerkennung, PPK, Belichtungspins, Flags zur Array-Ausrichtung – siehe den [Abschnitt „`process`“ der CLI
-Referenz](reference/cli-reference.md).

***

## Auswahl der zu exportierenden Daten (LATTICE-Produkte)

Die LATTICE-Verarbeitung wird **in einem Durchgang auf alle anwendbaren Produkte verteilt**. Vier produktbezogene Schaltflächen sind**standardmäßig aktiviert**; verwenden Sie das Formular `--no-`, um eine davon zu deaktivieren:

| Schalter | Produkt |
| --- | --- |
| `--debayered` | Lineares Demosaicing → `Debayered_Images/`: Entfernung des Dunkeloffset des Kalibrierungspakets, anschließend ein kantensensitives Demosaicing ohne Linearitätskorrektur, Flat-Field-Korrektur oder Unmix – dieselbe Datei, die bei einer Aufnahme auf der Stufe `debayered` gespeichert wird. Beibehaltung der nativen Ausrichtung des Sensors (in XMP gespeichert). |
| `--preview` | Vorschau anzeigen (Weißabgleich + Gamma; Falschfarben-Streckung für Multispektralbilder) → `Preview_Images/` |
| `--radiance` | float32 Strahlungsintensität, W/m²/sr/nm → `Radiance_Images/` (immer `tiff32/`) |
| `--reflectance` | uint16 Reflexionsgrad, Pix4D-kompatibel → `Reflectance_Calibrated_Images/` |

RGB
-Master-Kameras geben stets nur debayered + Vorschau aus – Radiance/Reflexionsgrad pro Band sind für einen Breitbandsensor nicht aussagekräftig, daher haben diese Umschaltoptionen für sie keine Auswirkung. Survey3
`.raw` ignoriert die Umschaltoptionen und folgt dem Standardpfad für Reflexionsgrad/Ziel.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (Standard: `auto`) wählt die Reflektanzreferenz aus: `auto` macht ein QA-konformes, im Bildfeld befindliches [Kalibrierungsziel](calibration-targets.md) zur absoluten Referenz und greift auf die DAQ-Lichtsensor-Abwärtsstrahlungsgrenze zurück (ρ = π·L/E) zurück, wenn kein Ziel vorhanden ist (ein Ziel, das die Qualitätssicherung mit einem Flag besteht, konkurriert mit dem DAQ hinsichtlich des gemessenen Werts); `target` ist streng (keine DAQ-Ersetzung); `daq` ist DAQ-maßgebend. Gemessene Zielscans pro Einheit können mit `--target-reflectance-dir` bereitgestellt werden.

{% hint style="info" %}
**Auslesen von Reflexionspixeln:**Der DN-Wert, der ρ = 1,0 bedeutet, gilt**pro Quelle** – LATTICE-Dateien fügen `Chloros:PixelScale=32768` in die XMP-Datei ein; Survey3
-Dateien verwenden 65535 und enthalten kein `Chloros:PixelScale`-Tag (sie enthalten jedoch andere `Chloros:*`-Verarbeitungs-Tags). Lesen Sie das Tag aus und dividieren Sie damit, anstatt von einer Konstante auszugehen. Details und der eine absichtliche Randfall ohne Skalierung finden Sie in der [CLI
-Referenz](reference/cli-reference.md).
{% endhint %}

**Die Verarbeitung beginnt immer bei `raw`.** Abgeleitete Produkte (Debayering-/Strahlungs-/Reflexions-Exporte) werden niemals wieder in die Pipeline zurückgeführt – ein erneuter Import und eine erneute Verarbeitung würden die Kalibrierungsberechnungen doppelt anwenden, daher überspringt Chloros
sie und weist darauf hin. `--input-level` ist die bewusst eingebaute Notausstiegsmöglichkeit, wenn Sie wirklich einen Einstiegspunkt erzwingen müssen.

***

## Wenn ein Durchlauf fehlschlägt

Ab Version 1.2.0 meldet `process` einen deutlichen Fehler, anstatt scheinbar „erfolgreich“ zu sein, ohne dass etwas angezeigt wird:

* Ein Lauf, der **Produkte angefordert, aber keine geschrieben hat**– nur `project.json` und `calibration_data.json` –, gibt `Processing finished but wrote no image products.` aus und**beendet den Lauf mit einem Wert ungleich Null**, sodass Skripte dies erkennen können. Die üblichen Ursachen: Der Eingabeordner wurde nicht als Aufzeichnung erkannt (überprüfen Sie das Layout und `--input-level`), oder jedes angeforderte Produkt war für diese Kameras nicht anwendbar (z. B. die Abfrage von Radianz/Reflexionsgrad bei reinen RGB
-Kameras).
* Ein **absichtlich nur auf Metadaten ausgerichteter Lauf** (alle Produkte deaktiviert, kein `--indices`) gilt weiterhin als erfolgreich – eine leere Bildausgabe ist in diesem Fall das korrekte Ergebnis.
* Führen Sie den Lauf mit `--verbose` erneut durch und überprüfen Sie das Backend-Protokoll auf Zeilen mit den Codes `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, die die kameraspezifischen Übersprünge erklären.
* Unter Windows
, bei dem lange Pfade deaktiviert sind, wird ein Projektordner, der zu tief liegt und die Pfadlänge von 260 Zeichen überschreitet, **vor** Beginn der Verarbeitung abgelehnt; dabei wird der längste Projektordner verwendet, der noch passt (siehe [Wo die Ausgaben gespeichert werden](#where-the-outputs-land)).
* Aufnahmebefehle (`lattice capture`, `lattice array-capture` und die anderen Aufnahmefamilien) listen alle Dateien auf, die sie nicht schreiben konnten, und beenden den Vorgang mit dem Code `1`; wiederholte Aufnahmen werden beendet, wenn das Laufwerk voll ist.

Beendigungscodes: `0` Erfolg · `1` Allgemeiner Fehler · `2` Argumentfehler · `130` Durch Strg+C unterbrochen.

***

## Vegetationsindizes

Rufen Sie `--indices` mit einem oder mehreren voreingestellten Namen auf; jeder Index wird in einem eigenen `<INDEX>_Index_Images/`-Ordner abgelegt:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

Die 22 voreingestellten Namen, die `process --indices` akzeptiert:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Ein Name, der noch nicht im Projekt aufgeführt ist, wird hinzugefügt. Ein Name, der keine Voreinstellung ist oder den die Kamerabänder nicht erzeugen können, wird übersprungen, wobei eine Warnung mit dem Namen ausgegeben wird. Wenn die CLI
-Schnittstelle die aktuelle Indexliste des Projekts nicht lesen kann, lässt sie die Liste unverändert und gibt eine Warnung aus, anstatt sie zu ersetzen.

{% hint style="warning" %}
**Es gibt drei Indexlisten – verwechseln Sie diese nicht.**Das Dropdown-Menü „Projekteinstellungen“ der GUI enthält 27 Formeln (fügt `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – diese fünf sind nur in der GUI verfügbar und gelten**nicht** für `--indices`). Der Live-/Offline-Befehl `lattice index --preset` verwendet eine eigene, separate Liste mit 22 Voreinstellungen. Formeln und Bandberechnungen sind unter [Multispektrale Indexformeln](project-settings/multispectral-index-formulas.md) dokumentiert.
{% endhint %}

***

## DAQ-Lichtsensoren: Ein kurzer Überblick

Die `daq pool-*`-Familie steuert MAPIR
-DAQ-Spektralsensoren (DAQ-U über USB, DAQ-M über BLE, DAQ-E über Ethernet) über den persistenten Pool des Backends an – GUI, CLI
und SDK
nutzen alle denselben Live-Handle. **`pool-*` ist der unterstützte DAQ-Pfad in der mitgelieferten CLI
**; andere `daq`-Unterbefehle, auf die möglicherweise verwiesen wird, sind eine rein MAPIR
-interne Oberfläche und werden mit einem expliziten Fehler beendet, der Sie auf `pool-*` verweist.

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

`pool-record` ohne `--duration` läuft bis `pool-record --stop`; das Standardausgabeverzeichnis ist `~/Documents/DAQ Live View/` **auf dem Rechner des Backends**. Das Cap-Korrekturprofil wird bei der Verbindung ausgewählt (`--cap-id`, Backend-Standard `sunshine_cosine`) und kann live mit `pool-set-cap` — Cap-Profile und der kalibrierte Messbereich des Sensors werden in den DAQ-Kapiteln dieses Handbuchs behandelt.

{% hint style="warning" %}
**DAQ-E auf einem Host mit mehreren Netzwerkkarten:** Die erste automatische Erkennung von `pool-connect --eth` nach dem Systemstart kann selbst bei einem fehlerfreien Sensor fehlschlagen. `--eth-host <ip-or-hostname>` ist die zuverlässige Variante – greifen Sie darauf zurück, wenn die Erkennung erfolglos bleibt.
{% endhint %}

***

## LATTICE-Kameras, PTP und Projektautomatisierung

Die `lattice`-Familie (über 45 Unterbefehle) deckt den gesamten Arbeitsablauf mit LATTICE-Kameras ab: Erkennung, Einzelaufnahmen, persistente synchronisierte Arrays mit dem „Smart-Prep“-Verbindungsablauf der GUI, Live-Vorschau im Browser, Ausrichtung, Indexberechnungen und Host-NIC-Diagnose. Ein kleiner Vorgeschmack:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Daneben: `chloros-cli time-sync` liefert Informationen über den PTP-Grandmaster, den der Chloros
-Host betreibt (LATTICE-Kameras und DAQ-E-Sensoren sind diesem als Slaves zugeordnet, um geräteübergreifende Zeitstempel zu erhalten), und `chloros-cli project` öffnet ein gespeichertes Chloros
-Projekt und steuert dessen Kameras, Arrays und Sensoren im Headless-Modus – einschließlich skriptgesteuerter YAML-Aufnahmerezepte.

Diese drei Familien (`lattice`, `project`, `daq pool-*`) sind zudem die einzigen, die `CHLOROS_BACKEND_URL` zur Steuerung eines **remoten** Backends unterstützen; die Kernbefehle zielen immer auf den lokalen Rechner ab.

Ausführliche Anleitungen finden Sie in den LATTICE-Kapiteln dieses Handbuchs; alle Flags sind in der [CLI
-Referenz](reference/cli-reference.md) aufgeführt.

***

## Fehlerbehebung: Die 5 häufigsten Probleme

| Symptom | Lösung |
| --- | --- |
| `Login required` oder ein geplanter Job bleibt an einer `Email:`-Eingabeaufforderung hängen | Führen Sie `chloros-cli login EMAIL 'PASSWORD'` einmal auf diesem Rechner aus – Befehle ohne zwischengespeicherte Sitzung werden interaktiv ausgeführt, anstatt sofort fehlzuschlagen. |
| `backend unreachable` | Starten Sie die Chloros
-Desktop-App oder starten Sie das Backend selbst: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` unter Windows
(im Ordner `backend` Ihres Installationsordners; dieser befindet sich nicht auf `PATH`), `chloros-backend` unter Linux
. `chloros-cli backend status` zeigt an, ob eines läuft; `lattice`/`project`/`daq pool-*` starten niemals einen. Wenn Sie sie auf ein entferntes Backend verweisen, überprüfen Sie `CHLOROS_BACKEND_URL`. |
| Array-Verbindung blockiert: `FRAMES WILL DROP` / `Reduce ROI to enable` | Empfangsring der Host-Netzwerkkarte auf Standardwerte zurückgesetzt – die häufigste Ursache dafür, dass ein zuvor funktionierendes System keine Verbindung mehr herstellt, typischerweise nach einer Aktualisierung des Netzwerkkartentreibers. Führen Sie `chloros-cli lattice network --fix` in einem **Terminal mit erhöhten Rechten** aus (oder setzen Sie `ReceiveBufferLen=256`, `PendingReceives=64`); siehe den Abschnitt *Einrichtung und Optimierung der Host-Netzwerkkarte* im Handbuch. |
| Der `daq`-Unterbefehl bricht ab: „erfordert das vollständige DAQ-Paket…“ | Verwenden Sie die Befehle `daq pool-*`, die denselben Sensor über das Backend steuern: Verbindung herstellen, Streamen, Aufzeichnen, CAP-Auswahl sowie für ein DAQ-E dessen Netzwerkeinstellungen, einen Neustart und Firmware-Updates (oder `chloros_sdk.connect_daq_sensor()` aus Python
). |
| Jetson gibt vor großen Ordnern eine Swap-Warnung aus | Fügen Sie dateibasierten Swap hinzu – die CLI
gibt die genauen auszuführenden Befehle `fallocate`/`swapon` aus. |

***

## Hilfe erhalten

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Alle Flags, alle Unterbefehle:** [CLI
-Referenz](reference/cli-reference.md)
* **Python
-Äquivalent:** [Python
SDK
](api-python-sdk.md) und die [SDK
-Referenz](reference/sdk-reference.md)
* **Support:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
