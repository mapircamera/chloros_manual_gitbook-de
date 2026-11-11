# API: Python-SDK

{% hint style="info" %}
**Suchen Sie nach der vollständigen API?** Diese Seite ist ein praktisches Tutorial. Alle öffentlichen Klassen, Methoden, genauen Signaturen und kopierbaren Beispiele finden Sie in der [SDK-Referenz](reference/sdk-reference.md), die für KI-Assistenten optimiert ist.**Arbeiten Sie mit einem KI-Assistenten?** Fügen Sie diese URL in den Chat ein, damit dieser Zugriff auf die vollständige, aktuelle Chloros-1.3.0-API von Chloros hat:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Jede Seite dieses Handbuchs ist als Markdown-Rohdatei unter ihrem Slug in Kleinbuchstaben + `.md` verfügbar, und das gesamte Handbuch ist unter `https://mapir.gitbook.io/chloros/llms.txt` indexiert.
{% endhint %}

Das **Chloros Python SDK** (`chloros-sdk` auf PyPI) steuert alle Funktionen der Desktop-App über Python: die Stapelverarbeitung von Bildern, die Live-Steuerung von LATTICE-Kameras und -Arrays, DAQ-Lichtsensor-Sitzungen sowie die Automatisierung gespeicherter Projekte. Es handelt sich um eine dünne Schicht über demselben lokalen Backend, das auch die GUI und die CLI nutzen (HTTP unter `127.0.0.1:5000`), sodass das Verhalten auf allen drei Oberflächen identisch ist.

## Installation

Das SDK ist im Chloros-Desktop-Paket enthalten, das auch das Verarbeitungs-Backend und die Hardware-Laufzeitumgebungen bereitstellt und stets mit dessen Version übereinstimmt.

**Schritt 1 – Chloros installieren.** Windows: Führen Sie das Desktop-Installationsprogramm (Standardpfad `C:\Program Files\Chloros\`) von der [Download-Seite](download.md) aus. Linux: Installieren Sie das Paket `.deb` mit `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Linux-Installation](linux/linux-installation.md)).**Schritt 2 – Überprüfen Sie das SDK** (Python 3.7+). In der Regel muss nichts installiert werden:

* **Windows:** Das Installationsprogramm installiert das mitgelieferte SDK-Wheel in Ihr System-Python (falls kein Python installiert ist, wird dieser Schritt übersprungen).
* **Linux:** Das `.deb` installiert das SDK für das System-`python3` in `/usr/lib/python3/dist-packages`. Die Voraussetzungen, `python3-numpy` und `python3-requests`, werden vom Paket empfohlen: `apt install` enthält sie, `dpkg -i` hingegen nicht.
* **Eine virtualenv** erkennt die System-Pakete nicht; installieren Sie das mitgelieferte Wheel darin: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` unter Linux oder das `chloros_sdk-*.whl` im Ordner `sdk` des Windows-Installationsordners (standardmäßig `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Auf einem Rechner ohne Chloros-Paket, dessen SDK ein Backend auf einem anderen Rechner steuert (siehe [Was eine reine pip-Installation bietet](#what-a-pip-only-install-gets)), installieren Sie es über PyPI: `pip install chloros-sdk`. PyPI wird bei Release-Builds aktualisiert, sodass es mit der neuesten stabilen Version übereinstimmt.

**Schritt 3 – Einmal pro Rechner anmelden:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Anmeldedaten werden in `~/.chloros/` zwischengespeichert (beide Plattformen). Unter Windows können Sie sich alternativ über die Registerkarte „Benutzer“ <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> der Desktop-App anmelden. Das SDK erfordert einen kostenpflichtigen Chloros+-Tarif – siehe [Lizenzanforderungen](#license-requirement) weiter unten.

| Anforderung | Details |
| --- | --- |
| **Chloros installiert** | Windows: Desktop-Installationsprogramm; Linux: `.deb`-Paket (enthält die Backend-Binärdatei und das SDK) |
| **Python** | 3.7 oder höher (entwickelt/getestet mit 3.10) |
| **Betriebssystem** | Windows 10/11 64-Bit, Ubuntu 22.04 LTS oder neuer oder NVIDIA Jetson (JetPack 6) |
| **Lizenz** | Aktives Chloros+-Konto, beliebige kostenpflichtige Stufe (Copper oder höher) |

## Erfolg in 60 Sekunden

Ein einziger Aufruf erstellt ein Projekt, importiert einen Ordner, konfiguriert die Verarbeitung und führt die Pipeline aus – dabei wird das Backend automatisch gestartet, falls noch keines läuft:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(Unter Linux verwenden Sie Linux-Pfade: `/home/user/drone_images/flight001`. Das SDK funktioniert auf beiden Plattformen identisch.)

Verarbeiten Sie einen LATTICE-Aufnahmeordner? Verwenden Sie den LATTICE-freundlichen Wrapper – er wendet die richtigen Standardeinstellungen an (keine Erkennung des Panel-Ziels, Standard-Debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` – vollständige Kontrolle über die Pipeline

Für alles, was über einen Einzeiler hinausgeht, verwenden Sie `ChlorosLocal`. Es nutzt ein bereits laufendes Backend (das der Desktop-App) oder startet bei der ersten Verwendung das installierte Backend (`auto_start_backend=True`; unter Windows aus dem Ordner „`backend`“ des Chloros-Installationsordners, standardmäßig `C:\Program Files\Chloros\backend\`), erstellt und konfiguriert Projekte, überwacht den Fortschritt und gibt nach Abschluss eine Zusammenfassung aus.

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
Behalten Sie den Standardwert `http://127.0.0.1:5000` bei, anstatt ihn durch `localhost` zu ersetzen — unter Windows wird `localhost` zunächst zu `::1` aufgelöst und kostet bei einem reinen IPv4-Backend etwa 2 Sekunden pro Anfrage.
{% endhint %}

Verwenden Sie es als Kontextmanager. Das Verlassen des Blocks beendet die Nutzung des Backends durch diese Instanz, nicht das Backend selbst: Es bleibt für das nächste Skript aktiv und stoppt sich nach 30 Minuten Inaktivität von selbst. `shutdown_backend()` beendet eine bereits gestartete Instanz.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` akzeptiert folgende Schlüsselwörter: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` und `custom_settings`. Die wichtigsten Werte:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Die LATTICE-spezifischen Regler (`input_level`, `radiometric_output`, die `array_alignment*`-Familie) sind mit ihren vollständigen Wertetabellen in der [SDK-Referenz](reference/sdk-reference.md#supported-values) dokumentiert.

### Verfolgen des Fortschritts

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Auslesen der Zusammenfassung nach dem Lauf – und Erkennen leerer Läufe

Nach Abschluss des Laufs fügt `process()` die Verarbeitungszusammenfassung des Backends als `result["summary"]` an. Jeder Eintrag in `summary["hints"]` ist ein vollständiger Satz, der alles Bemerkenswerte erklärt – zum Beispiel, warum ein Lauf keine Ausgabe erzeugt hat – und jeder Hinweis wird zusätzlich als Python-Hint `UserWarning` ausgegeben, sodass leere Läufe selbstdiagnostisch sind, selbst wenn Sie das Dict nie überprüfen:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` wird nicht ausgelöst, wenn ein Lauf keine Bilder erzeugt.** Dies ist der einzige Punkt, an dem sich das SDK und die CLI bewusst unterscheiden: `chloros-cli process` behandelt „Produkte wurden angefordert, es wurden keine geschrieben“ als Fehler und beendet den Lauf mit einem Wert ungleich Null, während das SDK normal zurückkehrt und den Zustand über `summary` bzw. Hinweise meldet. Sollte Ihre Pipeline bei einem leeren Lauf anhalten, überprüfen Sie dies selbst – überprüfen Sie `summary` (oder zählen Sie die Dateien im Projektordner), anstatt sich auf eine Ausnahme zu verlassen.
{% endhint %}

## Smart Connect – Live-Hardware

Drei Hilfsprogramme öffnen dauerhafte Sitzungen im Hardware-Pool des Backends – demselben Pool, den die CLI, andere Skripte und die Kamera-Registerkarten der GUI nutzen, sodass SDK-Skripte mit diesen koexistieren können, ohne um serielle Schnittstellen oder Netzwerkbandbreite zu konkurrieren. Alle drei starten automatisch ein lokales Backend, falls noch keines läuft. Das DAQ-Fenster der GUI ist nicht Teil des Pools: Schließen Sie einen Lichtsensor in der GUI, bevor Sie ihn über ein Skript öffnen.

### Einzelne LATTICE-Kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Relative Pfade beziehen sich auf den Ort, an dem Ihr Skript ausgeführt wird (geändert in 1.3.0).** Bei einem lokalen Backend ist ein relativer Pfad `output_dir` (einschließlich des Standardpfads `"output"`) — und die anderen Pfade, die das Backend öffnet (`burst_dir`, `profile_path`, `save_path`, `mask_path`) – werden relativ zum Arbeitsverzeichnis Ihres Skripts aufgelöst, sodass `cam.capture("output/")` in `<current folder>/output/` schreibt. In Version 1.2.0 hat ein installiertes Backend diese Namen in seinem eigenen Installationsordner aufgelöst und die Fehlermeldung „Zugriff verweigert“ ausgegeben. Bei einem Remote-Backend werden die Pfade so übermittelt, wie sie eingegeben wurden.
{% endhint %}

### Synchronisiertes Array – `connect_array`

`connect_array` ist der empfohlene Einstiegspunkt für Mehrkamera-Rigs. Es führt denselben Smart-Prep-Ablauf wie die GUI aus: Netzwerkanalyse, automatische Auswahl der Synchronisationsstufe, PTP-Zeitsynchronisation, Auswahl des Pixelformats pro Kamera, AE-Seeding und Aktivierung des GPIO-Triggers. Die **erste serielle Nummer ist der Master** (er löst den Hardware-Triggerimpuls aus); die übrigen sind Slaves.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Fügen Sie `smart=True` zu jeder Array-Aufnahme hinzu, um vor dem Auslösen abzuwarten, bis sich die automatische Belichtung bei allen Kameras eingependelt hat.

Neu in Version 1.3.0: Ein `ArraySession` liefert Ihnen außerdem direkt synchronisierte Bilder, ohne Dateien zu schreiben:

* `arr.grab()` gibt ein Bild pro Kamera zurück, alle aus einem einzigen Trigger, als NumPy-Arrays (`{serial: array}`); `.group_id` ist der gemeinsame Auslöser und `.errors` benennt jede ausgelassene Kamera.
* `arr.frame_stream(fps=5.0)` liefert pro Tick einen solchen Satz für die Live-Verarbeitung in Ihrer eigenen Pipeline.
* `arr.continuous("output/")` führt die Backend-eigene kontinuierliche Aufnahme durch (den „Continuous“-Modus der GUI) und gibt ein `ContinuousHandle` mit `status()` zurück, `wait()` und `stop()` – verwenden Sie dies anstelle einer Aufzeichnungsschleife, wenn jeder Trigger mit hoher Rate gespeichert werden soll.

Informationen zu den Erfassungsmodi (Einzeln / Kontinuierlich / Intervall / Schnellstmöglich), Rekordern, „Burst-to-Video“ und Array-Ausrichtung finden Sie in der [SDK-Referenz](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ-Lichtsensor – `connect_daq_sensor`

Ohne Argumente erkennt `connect_daq_sensor()` automatisch das Transportprotokoll (Priorität: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Jeder Frame enthält den 135-Punkte-Wert `spectrum` (W/m²/nm nach Kalibrierung), ein `is_saturated`-Flag sowie CIE `x`, `y` sowie `z`. Um einen bestimmten Sensor oder Transport festzulegen, übergeben Sie einen expliziten Hinweis:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Beachten Sie, dass Cap-Korrekturprofile (`cap_id`) **kein** SDK-Regler sind – wählen Sie diese stattdessen über `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` aus.

### Gespeicherte Projekte – `open_project`

Ein gespeichertes Chloros-Projekt behält seine angeschlossene Hardware bei (`cameras.json` + `sensors.json` sowie `project.json`), und `chloros_sdk.open_project(path)` kann seine Kameras, Arrays und Lichtsensoren auf einmal wieder verbinden und Aufnahmen anhand des Gerätenamens auslösen. Lichtsensoren stellen über ein laufendes Chloros-Backend wieder eine Verbindung her, wobei ihre gespeicherten Einstellungen und Kapazitäten beibehalten werden. Siehe [Projektautomatisierung](reference/sdk-reference.md#project-automation--chlorosproject) in der Referenz.

## Was eine reine pip-Installation bietet

Überprüfen Sie die Verfügbarkeitsflags auf Modulebene, bevor Sie Hardware-Oberflächen verwenden:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Auf einem Host, auf dem **nur** `pip install chloros-sdk` und kein Chloros-Desktop-Paket installiert ist:

* `ChlorosLocal`, `process_folder` und `process_lattice_capture` funktionieren **nicht** – sie benötigen die Backend-Binärdatei, die im Desktop-Installationsprogramm enthalten ist.
* Die Smart-Connect-Helfer (`connect_camera`, `connect_array`, `connect_daq_sensor`) sind reine HTTP-Clients und funktionieren daher mit einem Backend auf einem anderen Rechner – die mitgelieferten Backends sind jedoch nur für Loopback gebunden, sodass Sie den Port selbst weiterleiten müssen (z. B. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) und `backend_url="http://127.0.0.1:5000"` zusammen mit `auto_start_backend=False` übergeben müssen. Siehe [Remote-Backend-Modus](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Die direkt auf der Hardware laufenden LATTICE-Klassen (`LatticeCamera`, `CameraPool`, …) lassen sich zwar importieren, benötigen jedoch die Arena-SDK-Laufzeitumgebung aus dem Desktop-Paket – ohne diese ist `CAMERA_AVAILABLE` gleich `False`.
* DAQ-Sensoren werden ausschließlich über ein (getunneltes) Backend angesteuert – `connect_daq_sensor()` oder das `proj.sensors` eines Projekts; das Backend öffnet den Sensor auf seinem eigenen Rechner.

## KI-Assistenten – `chloros-mcp`

Das SDK enthält `chloros-mcp`, einen MCP-Server, über den Claude Desktop, Claude Code, Cursor und andere MCP-Clients Chloros steuern können: Bilder verarbeiten und LATTICE-Kameras, Arrays sowie DAQ-Lichtsensoren betreiben. Seine Tools decken die Funktionen dieses SDKs ab. Es wird zusammen mit dem SDK installiert, sodass die Einrichtung für Claude Code aus einer einzigen Zeile besteht:

```bash
claude mcp add chloros -- chloros-mcp
```

Andere Clients: [Verwendung von Chloros mit KI-Assistenten](ai-assistants.md). Optionen und alle Tools: [MCP-Server (KI-Assistenten)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Lizenzanforderungen

Für den Zugriff auf das SDK ist ein aktives Chloros+-Login in einem beliebigen kostenpflichtigen Tarif erforderlich – **Copper oder höher**(Copper / Bronze / Silver / Gold); die kostenlose Stufe „Iron“ bietet keinen SDK-/CLI-Zugriff. Die Überprüfung erfolgt**serverseitig**: Jede SDK-Anfrage muss sowohl eine aktive Sitzung als auch einen kostenpflichtigen Tarif enthalten, andernfalls gibt das Backend die Fehlercodes `403` / `PLAN_UPGRADE_REQUIRED` zurück (ausgelöst als `ChlorosLicenseError` durch `ChlorosLocal` und als `ChlorosConnectError` durch die `connect_*`-Helfer). Ein abgemeldeter Aufrufer erhält stattdessen `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) – eine erneute Ausführung von `chloros-cli login` behebt den ersten Fall, nicht jedoch den zweiten.

Die Offline-Nutzung funktioniert innerhalb der Karenzzeit des Tarifs: Die Tarifstufe wird aus dem Server-Validierungs-Cache (5 Minuten) oder dem signierten, maschinengebundenen Lizenz-Cache (30 Tage bei monatlichen Tarifen; bis zum Ablauf des Abonnements bei jährlichen Tarifen) gelesen. Nach Ablauf der Karenzzeit wird der Tarif auf „kostenlos“ zurückgesetzt und der SDK-Zugriff wird unterbrochen, bis der Rechner einmal eine Verbindung zum Server hergestellt hat. `chloros-cli status` bleibt in der kostenlosen Stufe erreichbar, sodass der Grund immer sichtbar ist. Siehe [Chloros+-Anmeldung](chloros+-login.md).

## Ausnahmen

Fangen Sie die Basisklasse ab, um „alle Fehler bei Chloros“ zu behandeln:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

Alle Pipeline-Ausnahmen (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) leiten sich von `ChlorosError` ab. Eine Besonderheit: `ChlorosConnectError` – wird nur von `connect_camera` / `connect_array` / `connect_daq_sensor` ausgelöst — leitet sich vom einfachen `Exception` ab, **nicht** von `ChlorosError`, sodass `except ChlorosError` diesen Fehler nicht abfängt. Die vollständige Hierarchie finden Sie in der [SDK-Referenz](reference/sdk-reference.md#exceptions).

## Siehe auch

* [SDK-Referenz](reference/sdk-reference.md) – die vollständige API-Oberfläche, optimiert für KI-Assistenten.
* [CLI-Referenz](reference/cli-reference.md) – jeder CLI-Unterbefehl entspricht einem SDK-Aufruf.
* [Verwendung von Chloros mit KI-Assistenten](ai-assistants.md) – `chloros-mcp`-Einrichtungs- und Prompt-Rezepte.
* [Download](download.md) – Installationsprogramme für Windows und Linux.
