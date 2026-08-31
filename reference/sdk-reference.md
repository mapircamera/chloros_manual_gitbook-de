# Chloros Python SDK Referenz

**Version:**

1.2.0**Erstellt:**

29.07.2026 19:19 ·**Überarbeitet:**

30.08.2026**Paket:** `chloros-sdk` (PyPI)**Zielgruppe:** Optimiert für die Nutzung durch große Sprachmodelle (LLM); für Menschen lesbar.**Umfang:** Alle öffentlichen Klassen, Funktionen und Hilfsfunktionen, die von `import chloros_sdk` bereitgestellt werden, mit kopier- und einfügbaren Beispielen zu Bildverarbeitung, Steuerung einzelner Kameras, synchronisierten Arrays, DAQ-Sensoren und Projektautomatisierung.

Wenn Sie nur die wichtigsten Punkte benötigen, springen Sie zu:
- [Installation &amp; Schnellstart](#installation)
- [Smart-Connect für LATTICE-Arrays](#smart-connect-for-lattice-cameras)
- [DAQ-Sensor-Sessions](#daq-sensor-sessions)
- [Projektautomatisierung](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Architektur in 60 Sekunden

SDK ist eine Thin-Python-Schicht über dem Backend „Chloros“ (derselbe Flask-Server, den auch die Desktop-GUI und „CLI“ verwenden). Zur Automatisierung importieren Sie `chloros_sdk` und rufen übergeordnete Methoden auf; im Hintergrund wird jeder Aufruf zu einer HTTP-Anfrage an das lokale Backend auf Port 5000 — `http://127.0.0.1:5000/api/...` (bewusst nicht `localhost`, das zunächst auf Windows zu `::1` aufgelöst wird und bei einem reinen IPv4-Backend etwa 2 Sekunden pro Anfrage kostet). Das Backend verwaltet den Hardware-Pool – Kameras, DAQ-Sensoren, Ausrichtungsprofile, Bildpuffer –, sodass „SDK“-Skripte neben der GUI laufen können, ohne um serielle Schnittstellen oder Netzwerkbandbreite konkurrieren zu müssen.

Es gibt drei Oberflächen, die Sie verwenden werden:

1. **`ChlorosLocal` + freie Funktionen** (`process_folder`, `process_lattice_capture`) – Bildverarbeitungspipeline. Führen Sie für einen gesamten Ordner die Kalibrierung, den Debayering-Vorgang und den Index-Export mit einem einzigen Aufruf von „Python“ durch.
2. **Smart-Connect-Handles** (`connect_camera`, `connect_array`, `connect_daq_sensor`) – Öffnen einer persistenten Backend-Sitzung für Live-Hardware. Gleicher „Smart-Prep“-Ablauf wie in der GUI: Netzwerk-Probe, automatische Tier-Auswahl, PTP, AE-Seeding, GPIO-Trigger-Konfiguration.
3. **`ChlorosProject` / `open_project`** — Ein gespeichertes Projekt laden (Ordner mit `cameras.json` + `sensors.json` + `project.json`), alles auf einmal verbinden und Aufzeichnungen über benannte Handles steuern.

Die Oberflächen 1 und 2 **starten automatisch ein lokales Backend**, falls noch keines im Listening-Modus ist (dasselbe mitgelieferte Binärprogramm, das auch von der GUI/dem „CLI“ gestartet wird) – daher funktioniert ein einfaches Skript in einer neuen Shell, ohne dass Sie zuvor ein Backend starten müssen. Übergeben Sie `auto_start_backend=False`, um diese Funktion zu deaktivieren (z. B. wenn auf ein Remote-Backend verwiesen wird, das niemals gestartet wird). Siehe [Automatischer Start des Backends](#backend-auto-start). Surface 3 verhält sich anders: `open_project()` akzeptiert keinen `auto_start_backend`-Parameter, und `connect_all()` startet niemals ein Backend – es fragt einmal nach `http://127.0.0.1:5000` einmal ab und wechselt, falls keine Antwort erfolgt, stillschweigend zur direkten (backend-freien) `lattice_sdk`-Gerätesteuerung. Nur `proj.process()` und `stream(..., overlays=True)` erstellen verzögert ein `ChlorosLocal()` (das automatisch startet).

Alle drei sind authentifizierungsabhängig: Führen Sie `chloros-cli login` einmal auf dem Rechner aus oder melden Sie sich über die Desktop-GUI an. Aufrufe von „SDK“ ohne gültige Sitzung lösen den Fehler `ChlorosAuthenticationError` aus.

Voraussetzungen:
- „Python“ 3.7+ (wie im Paket angegeben; entwickelt/getestet unter 3.10)
- „Chloros“ Desktop lokal installiert (die Backend-Binärdatei ist im Installationsprogramm enthalten)
- Aktive Anmeldung bei Chloros+. Die Mindeststufe für SDK / CLI ist **Copper**oder höher (Copper / Bronze / Silver / Gold); die kostenlose Stufe**Iron**hat keinen Zugriff auf SDK / CLI. Dies wird**serverseitig** durchgesetzt: Jede Anfrage mit dem Flag SDK / CLI mit dem Flag „-“ muss sowohl eine aktive Sitzung als auch einen kostenpflichtigen Tarif enthalten, andernfalls gibt das Backend `403` zusammen mit `error_code: PLAN_UPGRADE_REQUIRED` zurück (das von den Helfern `ChlorosLocal` und als `ChlorosConnectError` von den `connect_*`-Helfern). Ein abgemeldeter Aufrufer erhält stattdessen `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) – die beiden unterscheiden sich, da die erneute Ausführung von `chloros-cli login` den ersten Fehler behebt, den zweiten jedoch nicht.
- Die Offline-Nutzung wird innerhalb der Nachfrist des Tarifs unterstützt: Die Tarifstufe wird aus dem Server-Validierungs-Cache (5 Minuten) oder dem signierten, maschinengebundenen Lizenz-Cache (30 Tage bei Monatsplänen, bis zum Ablauf des Abonnements bei Jahresplänen). Nach Ablauf dieser Nachfrist wechselt der Tarif auf die kostenlose Stufe, und der Zugriff auf SDK / CLI wird unterbrochen, bis der Rechner den Server einmal erreichen kann. `chloros-cli status` (`GET /api/license-status`) bleibt in der kostenlosen Stufe erreichbar, sodass der Grund ersichtlich ist – es ist die einzige Route SDK / CLI, die von der Stufenbeschränkung ausgenommen ist.
- Windows 10/11 64-Bit, **Ubuntu 22.04 LTS oder neuer**oder Jetson (JetPack 6). Ubuntu 20.04 wird**nicht** unterstützt: Die Abhängigkeiten von `.deb` leiten sich aus den Bibliotheken ab, gegen die das Backend verlinkt, einschließlich `libc6 (>= 2.34)`, und „focal“ liefert glibc 2.31 aus.

---

## Installation

Das „Python“ (SDK) ist eine dünne „Python“-Schicht über dem Backend „Chloros“. Für alles, was über einige wenige reine DAQ-Workflows hinausgeht, benötigen Sie das **Desktop-Paket „Chloros“, das lokal installiert sein muss** (Installationsprogramm unter Windows oder Linux `.deb`) – dieses stellt die Backend-Binärdatei, die Arena-Laufzeitumgebung SDK für LATTICE-Kameras sowie die Kalibrierungspakete bereit.

Aktuelle Downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Schritt 1 – Installieren Sie das „Chloros“-Plattformpaket

#### „Windows“ (.exe)

1. Laden Sie `Chloros-Setup-x.y.z.exe` von der Download-Seite herunter.
2. Führen Sie das Installationsprogramm aus und folgen Sie den Anweisungen des Assistenten. Der Standardinstallationspfad lautet `C:\Program Files\MAPIR\Chloros\`.
3. Starten Sie „Chloros“ mindestens einmal und melden Sie sich mit Ihrem „Chloros+“-Konto an.

#### „Linux“ amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### „Linux“ arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Schritt 2 – Installieren Sie das „Python“ SDK

**Das Installationsprogramm unter Chloros enthält ein passendes „SDK“-Wheel.** Jedes Installationsprogramm unter Windows und jede .deb-Datei unter Linux legt eine Datei namens `chloros_sdk-X.Y.Z-py3-none-any.whl` auf der Festplatte ab, die genau mit der Version der GUI / CLI / des Backends übereinstimmt. Sie müssen nicht PyPI im Auge behalten, um auf dem neuesten Stand zu bleiben.

#### Windows

Der Installer führt `pip install` automatisch mit dem mitgelieferten Wheel unter Verwendung Ihrer SystemPython aus (bevorzugt wird der `py.exe`-Launcher, andernfalls wird auf `python -m pip` zurückgegriffen). Es sind keine weiteren Schritte erforderlich – `import chloros_sdk` funktioniert nach erfolgreicher Installation in Ihrer „Python“-Umgebung. Falls kein „Python“ auf dem Rechner vorhanden ist, überspringt der Installer diesen Schritt stillschweigenddiesen Schritt im Hintergrund aus, und die GUI sowie „CLI“ funktionieren weiterhin.

#### Linux (.deb)

Die .deb-Datei legt das Wheel unter `/usr/lib/chloros/sdk/` ab. Die Datei `postinst` gibt den genauen Befehl aus – PEP-668-Distributionen verweigern standardmäßig globale pip-Schreibvorgänge, daher erfolgt keine automatische Installation:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Bei Jetson-Bereitstellungen mit Air-Gap erfolgt dies vollständig offline – das Wheel befindet sich bereits auf der Festplatte.

#### Öffentliches PyPI

Für reine pip-Hosts (kein „Chloros“-Desktop-Paket installiert; Remote--Backend oder reine DAQ-Workflows):

```bash
pip install chloros-sdk
```

PyPI wird bei Installationsbuilds der Release-Version aktualisiert, sodass das veröffentlichte Wheel mit der neuesten stabilen Version übereinstimmt. Dev-Builds (z. B. `1.1.4.dev1`) werden ausschließlich über das mitgelieferte Installer-Wheel.

#### Überprüfen

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ Abonnement erforderlich.** Alle SDK-Aufrufe erfordern eine aktive Chloros+ Anmeldung. Führen Sie `chloros-cli login user@example.com 'YourPassword'` einmal pro Rechner aus; die Anmeldedaten werden in `~/.chloros/` zwischengespeichert.

### Benötige ich das Desktop-Paket?

Das pip-Paket allein reicht **nicht** für die meisten Arbeitsabläufe ausreichend. Hier sehen Sie, was die einzelnen „SDK“-Oberflächen benötigen:

| „SDK“-Oberfläche | Benötigt das Desktop-Paket? | Warum |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ja** | Startet die Backend-Binärdatei automatisch unter `/usr/lib/chloros/chloros-backend` (Linux) oder bei `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ja**(lokal)**/ Nein**(remote) | Reine „HTTP“-Clients über das Backend. Lokales Backend → Desktop-Paket erforderlich. Remote-Backend → `backend_url=`**über einen Tunnel** (siehe Remote-Backend-Modus – mitgelieferte Backends binden nur Loopback). |
| `ChlorosProject` / `open_project` | **Ja** | Ruft gespeicherte Projekte über das Backend ab. |
| Direkte LATTICE-Klassen (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ja** | Benötigt die native Arena-SDK-Laufzeitumgebung, die im Desktop-Paket enthalten ist. Andernfalls entspricht `CAMERA_AVAILABLE` beim Import `False`. |
| Direkte DAQ-Klassen (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Nein** | Reines „Python“ über pyserial/bleak/zeroconf. Eine reine „pip“-Umgebung kann DAQs durchgängig ansteuern. |

### Remote-Backend-Modus (reiner „pip“-Host, über Tunnel)

> **Das mitgelieferte Backend ist über das LAN nicht erreichbar.** Produktions-
> Builds binden nur an Loopback (beide Loopback-Familien) und lehnen den
> einzigen Nicht-Loopback-Modus (`CHLOROS_CLOUD_MODE`), daher
> kann `backend_url="http://<lan-ip>:5000"` **nicht mit einem installierten
> Chloros** funktionieren – dieses Muster funktionierte bisher nur mit einem source/dev
> Backend. Um ein Backend auf einem anderen Rechner anzusteuern, leiten Sie dessen Loopback-
> Port selbst weiter und verweisen Sie den „SDK“ auf den Tunnel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Headless-/CI-/Robotik-Hosts können einen Rechner mit der vollständigen Desktop-Installation als „Chloros-Server“ nutzen und überall sonst `pip install chloros-sdk` einsetzen – der Transport zwischen ihnen erfolgt jedoch über den oben beschriebenen, vom Benutzer eingerichteten Tunnel und nicht über ein direktes LAN-URL.

> **Bekannte Einschränkung – `ChlorosLocal` unterstützt nicht ausschließlich pip.** `ChlorosLocal(backend_url=BACKEND)` löst derzeit eine lokale Backend-Binärdatei in seinem Konstruktor auf *bevor* die „URL“ abgefragt wird, und löst den Fehler `ChlorosBackendError` („Chloros-Backend nicht gefunden…“) aus, wenn kein Desktop-Paket installiert ist – selbst bei einem erreichbaren Remote-Backend. Nur die oben genannte Smart-Connect-Oberfläche (`connect_camera` / `connect_array` / `connect_daq_sensor`, sowie die Hilfsprogramme `analyze_array_network` und `list_*` / `discover_*`) funktioniert auf einem reinen pip-Host.

### DAQ-only-Workflow (reiner pip-Host)

Wenn Sie nur DAQ-Sensoren benötigen und weder LATTICE-Kameras noch Bildverarbeitung nutzen, ist das pip-Paket in sich geschlossen:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Kein Backend, kein .deb, keine Anmeldung unter Chloros+ erforderlich für die direkte Hardware-Datenerfassung.

---

## Schnellstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Index der Top-Level-API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Bildverarbeitung — `ChlorosLocal`

Die übergeordnete Pipeline-Klasse. Startet das Backend bei der ersten Verwendung, erstellt und konfiguriert Projekte, überwacht den Fortschritt und gibt Zusammenfassungen nach Abschluss der Ausführung zurück.

### Konstruktor

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

### Methoden

| Methode | Beschreibung |
| --- | --- |
| `create_project(project_name, camera=None)` | Erstellt ein neues Projekt (optional mit einer Kameravorlage wie `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importiert RAW-/TIF-/JPG-/DNG-Bilder **sowie `.daq`-Lichtsensoraufzeichnungen**. Gibt `count` (Bilder) und `scan_count` (Aufzeichnungen). Es wird nur eine Warnung ausgegeben, wenn der Ordner beides nicht enthält. |
| `export_light_sensor(daq=True, csv=True)` | Schreibt kalibrierte `.daq` + `.csv`-Daten für jede Lichtsensoraufzeichnung des Projekts in `<project>/Light Sensor/`. Siehe [Lichtsensoraufzeichnungen](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Verarbeitungsregler einstellen. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Die Pipeline ausführen. Gibt `{"status": "complete", "async": False}` zurück sowie einen `summary`-Schlüssel, sofern das Backend einen bereitstellt – siehe [Zusammenfassung und Hinweise nach dem Lauf](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Backend-Status überprüfen. |
| `logout()` | Zwischengespeicherte Anmeldedaten löschen. |
| `shutdown_backend()` | Das Backend beenden (falls SDK gestartet wurde). |
| `discover_cameras()` | LATTICE-Kameras **über das Backend dieser Instanz** ermitteln (`/api/camera/discover`). Gibt eine Liste von Dictionaries zurück (`serial`, `model`, `ip`, …) – in derselben Form, wie sie die GUI/CLI anzeigt. Leere Liste, wenn keine gefunden wurden oder das Backend nicht erreichbar ist. |
| `camera_capture(output_dir, format="tiff", **settings)` | Nimmt ein einzelnes Bild**über das Backend**auf (wird automatisch über dieses Handle gestartet), sodass es die gleiche Vorbereitung erhält wie die GUI/ CLI (Standard: 12 Bit, Pool-Wiederverwendung, eingebettete Kalibrierungsmetadaten). Das Ziel mit `serial=` oder `device_index=` auflösen; `exposure`/`gain`/`pixel_format`/`preset` als `**settings`. Gibt das Legacy-Metadaten-Dict zurück (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Liefert überlagerte, zusammengesetzte Vorschaubilder von einer gepoolten Kamera – schlanker MJPEG-Client über die `/api/camera/<serial>/stream-annotated`-Route des Backends (Zebra / Gitter / Fadenkreuz / Histogramm / Peaking / Spot serverseitig gezeichnet). `decode=True` liefert BGR-Arrays; `False` liefert RohdatenJPEG-Bytes. Auch erreichbar über-Projekt als `ChlorosProject.stream(overlays=True)` erreichbar. |

Verwendung als Kontextmanager für garantierte Bereinigung:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Lichtsensor-Aufzeichnungen – kalibriert `.daq` + `.csv`

Ein DAQ-U / DAQ-M / DAQ-E kann **ohne** sein Kalibrierungsbundle aufgezeichnet werden. Das ist
das, was die öffentlichen [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) standardmäßig tun: Sie schreiben die Rohdaten der Sensorzählwerte und versehen die
Datei mit einem Zeitstempel, sodass Chloros die Werkskalibrierung dieses Sensors **über die Seriennummer** abruft – zunächst aus dem lokalen Cache,
dann aus der MAPIR-Cloud – und diese beim Import anwendet.

Chloros schreibt das Ergebnis als zwei Produkte pro Aufzeichnung zurück unter
`<project>/Light Sensor/`:

| Produkt | Was es ist |
| --- | --- |
| `<name>_calibrated.daq` | Das wiederaufbereitbare Archiv – dasselbe Schema wie eine Live-Aufzeichnung, nun mit Angabe des Bundles, das es erzeugt hat. Ein erneuter Import führt ****nicht** zu einer zweiten Kalibrierung. |
| `<name>_calibrated.csv` | Spektrale Bestrahlungsstärke in W/m²/nm auf dem sensoreigenen Wellenlängengitter, eine Zeile pro Messwert, sowie photometrische Spalten (Gesamtleistung, photopische/skotopische Lux, PPFD und deren Aufteilung in Blau/Grün/Rot, Spitzenwellenlänge). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Nur sensoren ohne Bundle (DAQ-A).** Rohdaten der spektralen Sensorzählwerte – *keine* Bestrahlungsstärke. Siehe unten. |

`process()` führt diesen Export als einen seiner Schritte durch. Es sind **keine** Bilddaten erforderlich:
Ein eigenständig geflogener Lichtsensor stellt einen vollwertigen Arbeitsablauf dar, und ein solches Projekt enthält konstruktionsbedingt
keinerlei Bilder.

**DAQ-A-Aufzeichnungen werden als Rohwerte exportiert.** Die DAQ-A-Familie stammt aus der Zeit vor dem seriennummernbezogenen
Bundle-System und verfügt über kein abzurufendes Bundle – sie wird stattdessen vor Ort anhand eines
Reflexionsziels kalibriert, weshalb sie nie eines benötigte. Diese Aufzeichnungen werden
unter dem Stamm „`_raw`“ statt „XPROT“ exportiertX000219 exportiert: ein anderer Dateiname statt eines Flags
innerhalb der Datei, da die Angabe auch beim Weiterleiten per E-Mail als reiner Name erhalten bleiben muss. Der
`.csv`-Header gibt `raw spectral sensor counts (NOT irradiance)` an und warnt, dass die
Werte **innerhalb** der Datei vergleichbar sind – genau dafür werden sie bei der zielbasierten Kalibrierung
verwendet – und nicht sensorübergreifend. Die leistungsabhängigen photometrischen Spalten (Gesamtleistung,
photopische/skotopische Lux, PPFD) werden als **NULL** zurück, anstatt aus Zählwerten integriert zu werden.

Ein DAQ-U / DAQ-M / DAQ-E, dessen Bundle einfach nicht abgerufen werden konnte, wird weiterhin **übersprungen**,
nicht im Rohformat geschrieben: In diesem Fall existiert das Bundle, und „erneut verbinden und neu verarbeiten“ ist ein echter Ratschlag.

Ältere **v1.01 / v1.02**-Aufzeichnungen (ein DAQ-A-SD schreibt diese) enthalten keine proLese-Epoche,
sondern nur die Schreibzeit der Datei. Der Image↔Downwelling-Matcher lehnt sie weiterhin ab – das Abgleichen eines
Frames mit einer Schreibzeit wäre unsichtbar falsch –, aber der Exporter liest sie ein, und der
CSV gibt „`clock=daq_created_on`“ aus, sodass das Produkt angibt, auf welcher Uhr es läuft.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Eine Aufzeichnung, deren Kalibrierungsbündel nicht abgerufen werden kann (Offline-Modus oder ein Sensor ohne
Kalibrierung in der Datei), wird unter `skipped` **mit Angabe des Grundes** gemeldet. Es wird niemals
als „kalibrierte“ Datei mit Rohzählwerten ausgegeben – stellen Sie eine Internetverbindung her und
führen Sie den Vorgang erneut aus, dann wird der Export abgeschlossen.

### Fortschritts-Callbacks

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Zusammenfassung und Hinweise nach Abschluss

Nach Abschluss ruft `process()` die Datei `GET /api/processing-summary` ab und fügt den Hauptteil als `result["summary"]` an. Das Abrufen erfolgt nach bestem Bemühen und blockiert niemals eine erfolgreiche Rückgabe – wenn die Zusammenfassung nicht verfügbar ist, greift `process()` auf die einfache Form `{"status": "complete", "async": False}` zurück. Jeder Eintrag in `summary["hints"]` – vollständige Sätze mit dem vorgeschlagenen Abhilfemaß, z. B. warum ein Lauf keine Ausgabe erzeugt hat – wird ebenfallsals „Python“ (`UserWarning`) ausgegeben, sodass Läufe mit Null-Ausgabe selbstdiagnostizierend sind, selbst wenn man das Wörterbuch nie überprüft:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` ist die maschinelllesbare Hälfte:

| Schlüssel | Was gezählt wird |
| --- | --- |
| `models` | Kameragruppen im Durchlauf. |
| `images_in_groups` | Quellbilder über diese Gruppen hinweg. |
| `targets_found` | Erkannte Reflexionsziele. |
| `images_calibrated` | Bilder, die der Durchlauf kalibriert hat. |
| `exported_files` | **Vom Durchlauf erstellte Bildproduktdateien.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Licht-Sensoraufzeichnungen, die absichtlich separat gezählt werden – sie stammen aus einer anderen Phase und existieren auch bei Durchläufen ohne Bildmaterial; würden sie mit einbezogen, würde ein reiner DAQ-Durchlauf so aussehen, als hätte er Bilder exportiert. |

Daneben: `summary["output_dirs"]` (jedes Verzeichnis, in das geschrieben wurde),
`summary["light_sensor_export"]`, `summary["stopped"]` (wahr, wenn der Benutzer den
Lauf unterbrochen hat, damit Teilzählungen nicht als abgeschlossener Lauf mit zu geringer Ausbeute gewertet werden) sowie
`summary["groups"]` (die Aufschlüsselung nach Gruppen).

`exported_files` wird von der Pipeline **während des Schreibvorgangs** aufgezeichnet und nicht anschließend aus den
Projekt-Bildobjekten des Projekts nachträglich abgetastet. Die Parallel- und GPU-Strategien erstellen ihre eigenen Bildobjekte
(in Worker-Unterprozessen für die GPU-Pfade), sodass der alte Scan
`0 file(s) written` für jeden solchen Lauf meldete und anschließend den Hinweis „zero-exports“ ausgab – bei Läufen,
bei denen alles funktioniert hatte. Wenn Sie Ihr Skript auf diese Nummer ausrichten, meldet ein fehlerfreier Parallel-Lauf nun
einen Wert ungleich Null.

„Light-Sensor-Skips“ melden den Grund, den der Reader tatsächlich für jede Datei festgestellt hat – ein
unlesbares Schema, ein fehlendes Bundle, einen Schreibfehler – **dedupliziert**, sodass zwanzig Dateien,
, die aus einem Grund übersprungen wurden, als einen einzigen Grund und nicht als zwanzig Wiederholungen desselben.

> **`process()` wird nicht ausgelöst, wenn ein Lauf keine Bilder erzeugt.** Dies ist der einzige Punkt, an dem sich der „SDK“ und
> der „CLI“ bewusst unterscheiden: `chloros-cli process` behandelt „Produkte wurden angefordert, es wurden keine
> geschrieben“ als Fehler und beendet den Lauf mit einem Wert ungleich Null, während die SDK normal zurückkehrt und den
> Zustand über `summary` / hints meldet. Sollte Ihre Pipeline bei einem leeren Durchlauf anhalten, überprüfen Sie dies
> selbst – sehen Sie sich `summary` an (oder zählen Sie die Dateien im Projektordner), anstatt sich darauf zu verlassen, dass
> keine Ausnahme auftritt. Die üblichen Ursachen sind ein Eingabeordner, der nicht als
> Erfassung erkannt wurde, sowie Produkte, die als für die vorhandenen Kameras nicht anwendbar übersprungen wurden (z. B. Radiance von Kameras, die ausschließlich RGB
> verwenden).

### Komfortfunktionen

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Unterstützte Werte

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrische Ausgabe (LATTICE-Multispektral-Pipeline)

Die Exportstufen der LATTICE-Multispektral-Pipeline (M3C/M3M) der `process`-Pipeline – `reflectance` (Standard), `radiance`, `sensor-response` oder `all` (jeder anwendbare Modus pro Bild) – entspricht der **„Radiometrische Ausgabe“** des Projekts. Für `configure()` gibt es ein eigenes Schlüsselwort:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Der erweiterte Ausweg – das Schreiben des `"Radiometric output"`-Schlüssels des Projekts über `custom_settings` – funktioniert weiterhin, aber beachten Sie, dass dadurch der gesamte Einstellungsblock ersetzt wird (siehe die Warnung unten):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (die Standardeinstellung) dividiert die Kamerastrahlung durch die **zeitstempelabgeglichene DAQ-Downwelling-Werte**, die automatisch aus einem aufgezeichneten `.daq` (DAQ-U/M/E)**oder einem DAQ-M-eigenen `.csv`**ermittelt wird, der zusammen mit den Bilddaten vorliegt; fehlende kameraspezifische oder DAQ-Kalibrierungspakete werden**bei der ersten Verwendung automatisch von AWS abgerufen** bei der ersten Verwendung automatisch abgerufen. Die „CLI“ stellt dies als typbezogene Produkt-Schaltflächen auf `chloros-cli process` bereit: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **ersetzt** den gesamten Block mit den berechneten Einstellungen (es umgeht dabei bewusst die anderen Schlüsselwörter und die Validierung von `configure()`). Wenn Sie es verwenden, fügen Sie jeden `Project Settings`-Schlüssel ein, der für Sie von Bedeutung ist, wie im obigen Beispiel.

---

## Smart-Connect für LATTICE-Kameras

Persistente Backend-Sitzungen für Live-Hardware. Es werden dieselben Endpunkte wie von der GUI verwendet, sodass das Verhalten über SDK / CLI / GUI hinweg identisch ist.

### Einzelne Kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()`-Signatur

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession`-Methoden

| Methode | Beschreibung |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | GenICam-Knoten lesen; gibt `{nodes, errors, enums, device}` zurück. |
| `set_settings(**kwargs)` | Knoten unter ihrem Anzeigenamen schreiben (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Nimmt ein **einzelnen** Frame auf. Gibt eine Liste mit einem Element zurück, die aus Frame-Metadaten-Dictionaries besteht. (Die Burst-/Multi-Frame-Erfassung wurde entfernt – rufen Sie `capture()` in einer Schleife auf, wenn Sie eine Serie benötigen.) |
| `disconnect()` | Aus dem Pool freigeben. Keine Auswirkung, wenn wir an eine bereits geöffnete Sitzung angehängt sind. |

`capture()`-Exportsteuerung (gleiches Modell wie das Array + GUI):

- `processing` / `levels` — `processing="all"` speichert alle anwendbaren Exporttypen; `levels=["raw","radiance"]` speichert nur diese (überschreibt `processing`). Lassen Sie beide weg, um die Backend-Standardeinstellung zu verwenden.
- `force_daq=True` — Speichert den zugewiesenen DAQ/DLS-Messwert als `.daq`-Sidecar, selbst bei einer „Raw-only“-Aufnahme, damit der Frame später in Reflektanz/Index umgewandelt werden kann. Keine Auswirkung, wenn kein DAQ verknüpft ist.

### Synchronisiertes Array — `ArraySession` (Smart-Prep)

`connect_array` ist **der empfohlene Einstiegspunkt** fürKamera-Konfigurationen. Im Hintergrund wird der vollständige Smart-Prep-Ablauf über die GUI ausgeführt:

1. **Netzwerkanalyse** (`/api/camera/array/recommend`) – ermittelt die größte Bildgröße, die in die Sim-Emit-Stufe passt, ohne dass Bilder verloren gehen.
2. **Automatische Tier-Auswahl** – `sim-capture-sim-emit`, sofern die Leitung dies zulässt; andernfalls `sim-capture-ftd-stagger` oder `slip-emit-and-capture`.
3. **Automatische Verkleinerung**– verkleinert stillschweigend die Frame-Größe bzw. erhöht das Binning, wenn die Leitung die angeforderte Auflösung nicht aufrechterhalten kann.**Dieses Sicherheitsnetz deckt keine aggregierte Überbelegung ab**: Zu viele Kameras für die Leitung können nicht durch das Verkleinern von Frames behoben werden – siehe [Überbelegung](#over-subscription-the-per-cam-floor).
4. **PTP standardmäßig aktiviert** – Zeitstempel zwischen den Kameras sind auf Mikrosekunden genau vergleichbar.
5. **Automatische Auswahl des Pixelformats pro Kamera** — RGB-Kameras → `BayerRG8`, Multispec → `BayerRG12`.
6. **AE-Seeding** — Erfasst den aktuellen AE-Zustand jeder Kamera, damit die Verbindung die Belichtung während des Betriebs nicht zurücksetzt.
7. **GPIO-Trigger-Konfiguration** — `connect_array` versetzt jede Kamera in den Auslösezustand (`TriggerMode=On`, `TriggerSource=Line2`), sodass der Impuls des MastersImpuls die Slaves über das M8-Kabel ansteuert. Dies ist ein Schritt, der nur bei einem Array durchgeführt wird: Eine einzelne Kamera, die mit `LatticeCamera` geöffnet wurde, läuft stattdessen im Freilaufmodus.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()`-Signatur

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier`-Werte:
- `"sim-capture-sim-emit"` – echte Synchronität (alle Kameras lösen am selben Taktflankenübergang aus).
- `"sim-capture-ftd-stagger"` – flexible zeitliche Staffelung (Kameras senden zu leicht versetzten Zeitpunkten, sodass die Pakete auf der Leitung serialisiert werden).
- `"slip-emit-and-capture"` – sequenzielle Erfassung pro Kamera (keine zeitliche Synchronisation; einzige Option, wenn keine Rahmengröße zur Simulanz passt).

`wire_ceiling_mbps` überschreibt das **nachhaltige Leitungsbudget des Hosts** in MB/s – die einzige
Zahl, von der die gesamte Array-Zuweisung abhängt. Belassen Sie den Wert auf `None`, um den automatisch ermittelten
Wert zu verwenden. Verringern Sie ihn, wenn das Array GVSP-beschädigte Frames meldet: Der automatische Wert wird
aus der von der Netzwerkkarte gemeldeten Verbindungsrate, die USB-Adapter, schmale PCIe-Lanes und
stark ausgelastete gemeinsam genutzte Fabrics überbewertet – und diese Überschätzung zeigt sich eher in Form von beschädigten Frames als in einer
sichtbar langsame Verbindung. Der Wert wird im Array-Erfassungsblock des Projekts gespeichert, sodass ein
erneutes Öffnen oder ein späteres `connect_array` ihn wie jede andere Array-Einstellung wiederherstellt.
Siehe [Array-Zustand](#array-health--which-Subsystem-is-losing-frames).

#### Überbelegung (die Untergrenze pro Kamera)

Sim-Emit-Pacing weist jeder Kamera einen Anteil am kollisionssicheren Leitungsbudget zu, dessen Untergrenze bei **8 MB/s pro Kamera**liegt (`per_cam_floor_bps`). Sobald `N × floor` die kollisionssichere Obergrenze überschreitet,**überbelegt das Array die Leitung**– der Fehlermodus ist GVSP-Paketverlust, nicht eine niedrigere Bildrate – und es gibt keine Abhilfe hinsichtlich der Frame-Größe:**Binning und ROI verringern die Bytes pro Frame, nicht die durch das Pacing festgelegten Bytes pro Sekunde**, die bei der Aggregatprüfung verglichen werden. Praktische Obergrenzen bei voller Auflösung auf einem 1-GbE-Host:**6 Kameras bei 1500 MTU, 9 mit Jumbo-Frames** (`max_cams_collision_safe` in der Analyseantwort gibt die Obergrenze für Ihre Leitung an). Abhilfemaßnahmen: weniger Kameras, Jumbo-Frames von Ende zu-End oder eine schnellere Netzwerkkarte.

- Die Antworten `analyze_array_network()` und `/api/camera/array/connect` enthalten `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` und `per_cam_floor_bps`. Wenn `oversubscribed` wahr ist, setzt die Projektion **die fps-Felder auf Null** (`achievable_fps_max` / `fps_bright` / `fps_dark`) auf Null, anstatt eine irreführende, zwar langsame, aber funktionierende Rate zu melden.
- `POST /api/camera/array/connect` akzeptiert einen `pin_resolution`-Body-Parameter (**nur HTTP — kein „SDK“-kwarg**; `connect_array` macht ihn nicht sichtbar). Durch das Pinning wird das Sicherheitsnetz des „Binning Walk-Down“ entfernt, sodass eine überbelegte Verbindung mit gesetztem `pin_resolution`**kategorisch abgelehnt** wird mit einem Fehler, der jede Abhilfemaßnahme nennt. Ohne Pinning wird die Verbindung mit dem „Walk-down“ fortgesetzt, es wird jedoch gewarnt, dass durch die Verkleinerung das Aggregat nicht geleert werden kann.
- Notausgang für die Entwicklung: Setzen Sie `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` in der Umgebung des Backends, um die Ablehnung auf eine deutliche Warnung herabzustufen – Sie stellen die Verbindung trotzdem her und akzeptieren den Paketverlust .

#### Array-Zustand – welches Subsystem verliert Frames?

`GET /api/camera/array/<array_id>/capability` enthält einen aktiven `health`-Block auf einem
verbundenen Array, derin einem rollierenden **10-Sekunden**-Fenster neu ausgewertet. Es unterteilt den Frame-Verlust
in die beiden Ursachen, die gegensätzliche Korrekturen erfordern, anstatt einer einzigen „Unvollständigkeitsrate“, die
keine der beiden benennt:

| Feld | Bedeutung | Betroffenes Subsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pro Seriennummer) | Der Frame **kam an und war strukturell fehlerhaft**– GVSP-Paketverlust. |**Netzwerk**: Leitungsbandbreite, Pacing, NIC-RX-Ring, MTU |
| `never_arrived_rate_pct` (pro serieller Schnittstelle) | Der Frame **ist überhaupt nicht angekommen**– die Kamera hat nicht ausgelöst oder es wurde nichts gesendet. |**Trigger / Synchronisation**: M8-Kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Schlechteste Übertragungsrate der Kamera für jede. | — |
| `per_cam_rate_pct` | Kombinierte Unvollständigkeitsrate pro Kamera (beide Ursachen zusammen). | — |
| `stable_for_seconds` | Wie lange jede Kamera unter 0,01 % geblieben ist. | — |

Neben `health` gibt derselbe Datensatz die Zahl an, um die die gesamte Zuweisung überschritten wird:

| Feld | Bedeutung |
| --- | --- |
| `wire_ceiling_mbps` | Das derzeit gültige, nachhaltige Bandbreitenbudget des Hosts in MB/s. |
| `wire_ceiling_source` | Woher dieser Wert stammt, in Worten – z. B. `USB-capped 200 MB/s (was theoretical 1062; …)` oder `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, als `wire_ceiling_mbps=` diesen Wert festlegte. |
| `nic_is_usb` | `true` für einen USB-Ethernet-Adapter. |

Für diesen Endpunkt gibt es keinen „SDK“-Wrapper – lesen Sie ihn direkt aus:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Auswertung:** Ein Wert ungleich Null für `gvsp_corrupt_rate_pct` bei einem Wert von 0 für `never_arrived_rate_pct` bedeutet, dass
dass Triggerung und Kabelsynchronisation einwandfrei sind und 100 % des Verlusts auf dem Netzwerkpfad liegen – senken Sie
`wire_ceiling_mbps` und stellen Sie die Verbindung neu her. Das umgekehrte Muster deutet stattdessen auf das Synchronisationskabel oder die
Triggerleitung hin.

> **`target_fps` ist nicht der Hebel für beschädigte Frames.** Das GevSCPD-Pacing wird einmalig bei der
> Verbindung festgelegt, sodass eine Verringerung der Triggerrate den Tastgrad verändert, nicht jedoch die
> Burstrate bei simultaner Übertragung. Eine gemessene 5-fache Reduzierung der Anforderung brachte keine Verbesserung, während
> die Senkung der Leitungsobergrenze von 240 auf 200 MB/s die Fehlerquote desselben Systems von 10,4 % auf
> 0,00 % senkte.

> **Eine automatische Reduzierung während der Übertragung ist bei der TRI032S-Firmware nicht verfügbar.** Ein laufendes Array kann
> dieses Problem nicht selbst beheben; trennen Sie die Verbindung und stellen Sie sie erneut her, damit der „Connect-Time-Picker“ die Planung anhand
> der neuen Obergrenze neu durchführt.

Ein **USB-Ethernet-Adapter wird von der Sonde auf 200 MB/s begrenzt**, unabhängig von seiner
Typenschildangabe: Die Effizienztabelle, die eine Verbindungsrate in einen Dauerdurchsatz umrechnet, ist
PCIe-basiert, und eine USB-Netzwerkkarte gibt zwar ihre Ethernet-Verbindungsrate an, ist dabei jedoch durch den
USB-Bus und dessen Treiber begrenzt. Die Obergrenze ist ein absoluter Wert, kein Bruchteil – ein USB-1-GbE-Adapter
erzielt ~80 MB/s und ist davon nicht betroffen.

#### `ArraySession`-Methoden

| Methode | Beschreibung |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Eine synchronisierte Erfassungsgruppe. Gibt ein `CaptureResult` zurück (Liste von Frame-Dicts + `.skipped`). Exportsteuerung siehe unten. |
| `capture(..., smart=True)` | **Smart Capture** – wartet, bis sich der AE bei allen Kameras eingependelt hat, und löst dann die Aufnahme aus. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Schnellste Aufnahme: nur Rohdaten + der zugewiesene DAQ-Messwert (+ der freie kombinierte Index). Spiegelt die Schaltfläche „Schnellste Aufnahme““. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Einzel-/Serien-/Intervallaufnahme in einer begrenzten Schleife. Gibt `list[CaptureResult]` zurück.**Erfordert `count` und/oder `duration_s`** erforderlich, damit die Funktion beendet wird (die „SDK“ verfügt nicht über die Tastenkombination Strg+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Startet die Aufzeichnung der kombinierten Live-Indexansicht als Video/GIF → `RecorderHandle`. Ein Composite-Recorder pro Array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Startet einefps-Raw-Bayer-Burst → `RecorderHandle`. Offline-Nachbearbeitung mit `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Offline-Nachbearbeitung einer gespeicherten Raw-Serienaufnahme zu kalibrierten Videos. Blockiert bis zum Abschluss (`wait=True`) und gibt `{outputs, errors, combined}` zurück. |
| `build_video_status(job_id, timeout=15.0)` | Einen Offline-Build-Job abfragen: `{running, result, error, burst_dir}`. |
| `disconnect()` | Das gesamte Array freigeben. |

`capture()`-Exportsteuerung (derselbe Endpunkt wie bei der GUI/CLI):

- `processing` / `levels` — `processing="all"` (oder `levels=["raw","radiance",…]`) speichert jeden anwendbaren Exporttyp pro Kamera; ein einzelner `processing`-Wert speichert nur diese Ebene.
- `aligned=True` — verzerrt den Nicht-Raw-Export jedes Mitglieds auf das [Ausrichtungsprofil] des Arrays(#array-alignment) des Arrays an (ko-registriert); Rohdaten bleiben unverzerrt, enthalten die Transformation jedoch in den Metadaten. Weicht auf „nicht ausgerichtet“ zurück (mit einer Warnung im `alignment` des Ergebnisses), wenn das Array kein Profil hat.
- `render_index=False` — Überspringe dieKamera-Vegetationsindex-Überlagerung; standardmäßig wird sie an der konfigurierten Stelle gerendert.
- `force_daq=True` — Speichert den zugewiesenen DAQ/DLS-Messwert als `.daq`-Sidecar-Datei, auch wenn kein ausgewählter Level dies benötigt.

**TIFF-Komprimierung (Regler „HTTP“):**`ArraySession.capture()` sendet keinen `compression`-Schlüssel, daher gilt die Standard-Einstellung des Backends – `POST /api/camera/array/capture` liest einen `compression`-Body-Parameter, `"deflate"` standardmäßig (verlustfreie zlib L1 + horizontaler Prädiktor, ~4,1 MB pro Vollauflösungs-Frame). `"none"` schreibt unkomprimiert (~6,3 MB/Frame) mit einem**~5-mal schnelleren Schreibvorgang** – beide sind verlustfrei und werden beim Import identisch gelesen. Die `SDK` stellt hierfür keine `kwarg` zur Verfügung; die Ausweichmöglichkeit ist ``chloros-cli lattice array-capture --compression none`` oder die Rohdaten-`HTTP`. DEFLATE hält zudem den GIL von `Python`, sodass komprimierte Schreibvorgänge nicht über die Schreib-Threads der einzelnen Kameras hinweg parallelisiert werden – eine anhaltende 8-Kamera-Aufnahme in voller Auflösung bei Sensor-Rate erfordert `compression: "none"`. Details: [CLI Referenz → array-capture](cli-reference.md).**Export-Überschreibungen pro Element (nur HTTP):**Der gleiche Endpunkt akzeptiert auch `exclude_serials` (Liste – Elemente aus dem gespeicherten Satz entfernen; das Array wird weiterhin als eine synchronisierte Gruppe ausgelöst, und ausgeschlossene Elemente werden in `excluded` zurückgegeben), `serial_levels` (`{serial: [level tokens]}` – Übersteuerungen auf Kameraebene) sowie `serial_index` (`{serial: bool}` – Übersteuerungen für Index-Overlays pro Kamera). Hierbei handelt es sich um Parameter zur GUI-Parität und**nicht um „SDK“-kwArgumente**; in den Zuordnungen fehlende Elemente greifen auf die array-weiten Werte `levels` / `render_index` zurück.

##### Überprüfung übersprungener Cams — `CaptureResult.skipped`

`ArraySession.capture()` gibt ein `CaptureResult` zurück, bei dem es sich um eine Unterklasse von `list` handelt: Iterieren, indizieren, mit `len()` verarbeiten – alle bestehenden Muster funktionieren weiterhin. Neuer Code kann das Attribut `.skipped` überprüfen, um festzustellen, welche Kameras ausgeschlossen wurden und warum. Der häufigste Fall ist, dass RGB Kameras in einem gemischtenFilter-Array, wenn man nach `processing="radiance"` oder `"reflectance"` fragt – die Strahlungsintensität pro Bayer-Pixel ist für einen Breitbandsensor bedeutungslos, daher überspringt das Backend diese Kameras, anstatt unsinnige Ergebnisse zu liefern.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Die Grund-Tokens folgen dem Muster `<level>-not-applicable-to-rgb-cam` (ein Eintrag pro übersprungener Ebene, jeder enthält `level`). Die reflektanzspezifischen Übersprünge sind `reflectance-skipped-no-fresh-dls` (kein aktueller Messwert der nach unten gerichteten Strahlung verfügbar), `reflectance-skipped-bound-daq-unavailable (…)` (der gebundene DAQ konnte nicht erreicht werden) und `dls-uncalibrated-band-<nm>` – das Band liegt größtenteils außerhalb des radiometrisch kalibrierten Bereichs des DAQ-Lichtsensors (~374–974 nm), sodass die absolute, auf dem DAQ basierende Reflexionsgrad-Trennung abgelehnt wird und das Bild lautstark auf die Sensorreaktion zurückgestuft wird. Unter den ausgelieferten SKUs löst nur die F988 dies aus; der unterstützte Arbeitsablauf dieser Kamera ist der Reflexionsgrad-Panel-Workflow.

`processing`-Stufen:

| Stufe | Ausgabe |
| --- | --- |
| `"raw"` | Einkanal-Bayer (Mono-Kameras: das Einzelband) direkt vom Sensor. |
| `"debayered"` *(Standard bei SDK)* | 3-Kanal-BGR über bilineare Demosaikierung (Monochromkameras: 1-Kanal-Graustufen). |
| `"radiance"` | float32 W/m²/sr/nm über die vollständige radiometrische Kette. Nur multispektral – „RGB“-Kameras werden übersprungen. |
| `"reflectance"` | uint16 0..32768 (Pix4D-kompatibel); erfordert eine Live-DAQ-Kopplung für absolute Referenz. Nur multispektral. |
| `"display"` | Vollständige Kette entsprechend der GUI-Vorschau (CCM + WB + Gamma gemäß dem Profil der Kamera). |
| `"all"` | **Eine Datei pro anwendbarer Stufe** für jede Kamera (entspricht der GUI-Standardeinstellung „Capture All“/CLI). Die zurückgegebene Datei „`CaptureResult`“ enthält dann pro „`(cam, level)`“ ein Frame-Dict, wobei die Stufe in jedem Dict enthalten ist; nicht anwendbare Stufen erscheinen in „`.skipped`“. Der für jedes Reflexions-Frame verwendete DAQ-Messwert wird als `.daq`-Sidecar gespeichert. |

> **Hinweis – Die Standardeinstellung weicht von der in der „CLI“ ab.** `ArraySession.capture()` ist standardmäßig auf `processing="debayered"` gesetzt; der Befehl `chloros-cli lattice array-capture` ist standardmäßig auf `processing="all"` gesetzt. Übergeben Sie `processing="all"` explizit über die „SDK“, um die mehrstufige Speicherung von „CLI“ /GUI abzubilden.

### Aufnahmemodi &amp; Rekorder

Die Array-Oberfläche spiegelt das GUI-Aufnahmefenster wider: Einzel-/Daueraufnahme-/Intervall-/Schnellauslöser-Modi sowie zwei Rekorder (Live-Composite-Video und Rohdaten-Burst → Offline-Nachbearbeitung).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**ist die Serien-/Intervallschleife des „SDK“. Da es kein `Ctrl+C` gibt, um diese Schleife per Skript zu unterbrechen,**müssen** Sie `count` und/oder `duration_s` übergeben (die Schleife stoppt, sobald eine der beiden Bedingungen erfüllt ist). `interval_s` wird ab dem Beginn jedes Durchlaufs gemessen (entsprechend der GUI). Verbleibende kwArgumente werden direkt an `capture()` weitergeleitet.
- **`record`** ist *für Überwachungszwecke geeignet*: Er erfasst den angezeigten Live-Composite-Stream mit kombiniertem Index, daher muss der kombinierte Stream geöffnet sein, damit Frames empfangen werden können. Ein Composite-Recorder pro Array (löst einen Fehler aus, wenn bereits einer läuft).
- **`burst` → `build_video`** ist *Analysequalität*: `burst` schreibt Rohframes + ein Manifest pro Frame + einen `.daq` pro eindeutigem DLS-Messwert unter `<output>/bursts/<base>/` mit der vollen Rate der Erfassungsschleife (keine Verkettung, kein exiftool, keine Live-Ansicht). `build_video` gleicht jeden Frame zeitlich mit dem nächstgelegenen `.daq` ab und führt die Radiance-/Reflexionsgrad-/Index-Kette der Import-Pipeline erneut aus. `products` ist eine Liste von `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (Standard: der kombinierte Index). `burst().stop()` löst außerdem automatisch die Erstellung eines „Best-Effort“-kombinierten Index aus, der im Stop-Ergebnis als `build_job` zurückgegeben wird.

#### `RecorderHandle`

Wird von `ArraySession.record()` und `ArraySession.burst()` zurückgegeben. Verwenden Sie es als Kontextmanager, um den Vorgang beim Verlassen des Gültigkeitsbereichs automatisch zu beenden, oder steuern Sie ihn manuell.

| Element | Beschreibung |
| --- | --- |
| `job_id` | Backend-Job-ID (Zeichenkette). |
| `kind` | `"composite"` (aus `record`) oder `"raw"` (aus `burst`). |
| `start_stats` | Das vom Aufruf von `start` zurückgegebene Wörterbuch. |
| `result` | `None` während der Ausführung; das endgültige „stop-result“-Dict nach dem Stoppen. |
| `stats(timeout=10.0)` | Live-Auftragsstatistiken (geschriebene Frames, erreichte fps, verstrichene Zeit). |
| `stop(timeout=60.0)` | Den Recorder anhalten; gibt das Endergebnis zurück und speichert es im Cache. Idempotent (ein zweiter Aufruf gibt das im Cache gespeicherte Ergebnis zurück). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### An ein bereits verbundenes Array anhängen — `attach_array`

Wenn das Array bereits aktiv ist (die GUI hat es geöffnet oder eine vorherige SDK-Sitzung hat `connect_array` aufgerufen), verwende `attach_array`, um ein Handle dafür zu erhalten, anstatt die Verbindung neu herzustellen. `connect_array` gibt in dieser Situation immer die Fehlermeldung „Kamera  befindet sich<sn> bereits im Array <id>“, da der POST-Aufruf von `/array/connect` für ein Mitglied im Pool nicht idempotent ist; `attach_array` liest `/api/camera/array/list` und gleicht die Werte entweder anhand der Array-ID oder der Seriennummern ab.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Muster: SDK Skripte, die gemeinsam mit der Desktop-GUI ausgeführt werden, sollten zunächst `attach_array` versuchen und auf `connect_array` zurückgreifen, wenn sich noch kein Array im Pool befindet.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Wichtig – Der Beendigungsbefehl des Context-Managers trennt die Verbindung.**`ArraySession.disconnect()` sendet immer ein POST an `/array/disconnect`; es gibt keine „attached-not-owned“-Prüfung wie bei `CameraSession` / `DAQSensorSession`. Wenn Sie gemeinsam mit der GUI arbeiten und das Array beim Verlassen des Gültigkeitsbereichs nicht abbauen möchten,**verwenden Sie den `with`-Block nicht** – speichern Sie den Handle in einer normalen Variablen und überspringen Sie den expliziten `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Hilfsmittel zur Netzwerkanalyse

Nützlich vor dem Öffnen des Arrays – zeigt an, ob Ihre vorgeschlagenen Einstellungen passen:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` ist eines von `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (ansonsten `error`). `auto_capped_fps` bedeutet, dass die angeforderte Auflösung nur bei einer begrenzten Triggerrate zum RX-Ring passt – behalte die Auflösung bei und übergebe `target_fps=result["recommended"]["recommended_target_fps"]` an `connect_array` (siehe [Beispiel 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**So lesen Sie die Projektion** (gleiches Modell wie im GUI-Fenster „Array-Einstellungen“):

- **„Burst“ (`frame_bytes_total`) wird pro Kamera im tatsächlichen Pixelformat der jeweiligen Kamera summiert.**Mono-**M3M**-Kameras streamen Mono12 (2 B/px), unabhängig davon, welchen Wert Sie für `pixel_format` übergeben. Daher beträgt die Größe eines Vollauflösungs-Frames mit vier Kameras bei drei Mono-Kameras**~25 MB** und nicht die ~12,6 MB, die sich aus der Annahme einer durchgängigen 8-Bit-Auflösung ergeben würden. Das Backend ermittelt das Format anhand ihres Modells.
- **Die Admittanz (`burst_fits_nic_ring`) berücksichtigt die Entleerungsrate**, nicht nach „ganzer Burst vs. Ring“: „Sim-Emit“ passt, wenn der Host den RX-Ring schneller entleert, als die Kameras ihn füllen. Ein 10G-Host + 1-GbE-Kameras**lassen** die volle-Res, selbst wenn der Burst den Ring übersteigt; ein 1-GbE-Host blockiert (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` ist eine konservative Obergrenze für den seriellen Abruf** — `max(readout+emit, N×emit)` mit pro Kamera auf den 1-GbE-Camera-Link begrenztem Emit, belichtungsunabhängig. Z. B. ~2,8 fps für ein Array mit 4 Kameras bei voller Auflösung und 12 Bit (entspricht den in der Laufzeit gemessenen ~2,7–3,0). Vollständiges Modell: [CLI Referenz → Array-fps- und Burst-Modell](cli-reference.md#array-fps--burst-model).
- **Überbelegung (`oversubscribed: true`) bedeutet, dass N × der Mindestwert pro Kamera die kollisionssichere Obergrenze überschreitet** – die fps-Felder (`achievable_fps_max` / `fps_bright` / `fps_dark`) zeigen 0 an, und automatisches Verkleinern/Binning kann das Problem nicht beheben (sie reduzieren die Bytes pro Frame, nicht die Bytes pro Sekunde im Pacing). Abhilfemaßnahmen sind weniger Kameras, Jumbo-Frames oder eine schnellere Netzwerkkarte; `max_cams_collision_safe` meldet die Obergrenze (6 Kameras mit voller Auflösung bei 1 GbE @ 1500 MTU, 9 mit Jumbo). Die Antwort enthält außerdem `aggregate_demand_bps`, `collision_safe_ceiling_bps` und `per_cam_floor_bps` (8 MB/s). Siehe [Over-Subscription](#over-subscription-the-per-cam-floor).

### Erkennung und Auflistung

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

LATTICE-Arrays führen im Hintergrund kontinuierlich eine automatische Belichtung (AE) durch, sobald sie verbunden sind, aber eine frischausgerichtete Szene benötigt einen Moment, um zu konvergieren. **Smart-Capture** ist die komfortable Lösung: Es fragt die Belichtung jeder Kamera ab, wartet, bis das Array über ein Fenster hinweg stabil ist, und löst dann die Aufnahme aus. Es entspricht der GUI: Die „Smart“-Aufnahmetaste der Desktop-App denselben Backend-Endpunkt auf.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Bei der Steuerung über `ChlorosProject` (nächster Abschnitt) stehen weitere Einstellmöglichkeiten zur Verfügung:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Die „Smart-AE“-Richtlinie ist standardmäßig konservativ eingestellt. Verengen Sie `exposure_tolerance_pct` für anspruchsvolle radiometrische Arbeiten; erweitern Sie den Bereich für sich schnell ändernde Szenen, bei denen „nahe genug“ ausreicht.

---

## DAQ-Sensor-Sitzungen

Persistenter Backend-Pool für Spektralsensoren (DAQ-U über USB, DAQ-M über BLE, DAQ-E über Ethernet). Spiegelt die Kameraoberfläche wider: Smart-Detect, Pool-Wiederverwendung, idempotente Anbindung.

### Smart-Detect (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Priorität: Ethernet → BLE → USB. Übergeben Sie einen beliebigen expliziten Hinweis, um den Transport festzulegen.

### Festgelegter Transport

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession`-Methoden

| Methode | Beschreibung |
| --- | --- |
| `status(timeout=10.0)` | Zusammenfassung des Pool-Eintrags (Streaming-/Aufzeichnungsstatus, Wellenlängenbereich, Kalibrierungs-SHA, Integrationszeit, frame_avg, AE-Status). |
| `latest(n=1, timeout=10.0)` | Gibt die bis zu N neuesten Spektrum-Frames zurück. |
| `stream_start()` / `stream_stop()` | Streaming fortsetzen / anhalten (Handle bleibt offen). |
| `record_start(output_dir=None, device_name=None)` | Aufnahme einer .DAQ-Datei. Gibt den Dateipfad zurück. Wird bei DAQ-U/M ohne AWS-Kalibrierungsbundle abgelehnt (DAQ-E ist davon ausgenommen). |
| `record_stop()` | Beendet die Aufzeichnung. Gibt `{path, rows}` zurück. |
| `disconnect()` | Aus dem Pool freigeben. Keine Operation für angehängte, aber nicht im Besitz befindliche Handles. |

> **Cap-Korrekturprofile (`cap_id`) sind kein „SDK“-Regler.** `connect_daq_sensor()` / `DAQSensorSession` stellen weder den Parameter `cap_id` noch die Methode `set_cap` bereit. Wählen Sie ein Flotten-Cap-Korrekturprofil über die CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) oder über die `/api/daq`-HTTP-Routen des Backends (`/api/daq/connect` und `/api/daq/<id>/cap-id` akzeptieren `cap_id`).

### Discovery – Suche nach einer Adresse für die Verbindung

`discover_daq_sensors()` durchsucht USB / BLE / ETH nach Sensoren ab, die man *öffnen* könnte. Es ist das DAQ-Pendant zu `discover_lattice_cameras()` und die einzige Möglichkeit, eine **BLE-MAC-Adresse eines DAQ-M**zu erhaltenBLE-MAC** zu erhalten – ein DAQ-E hat einen Hostnamen und ein DAQ-U einen COM-Port, aber eine MAC-Adresse ist weder auf dem Gerät aufgedruckt noch wird sie vom Betriebssystem aufgelistet.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Feld | Beschreibung |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-Port / BLE-MAC / Hostname — an `connect_daq_sensor` als `port=` / `mac=` / `eth_host=` weitergeben. |
| `display` | Lesbare Bezeichnung. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` oder `None` für einen Port, den der Scan nicht identifizieren kann (USB-Seriell-Adapter sind ohne Sonde nicht unterscheidbar, daher werden unbekannte Einträge angezeigt statt ausgeblendet). |
| `extra` | Details pro Übertragungsart (BLE-Advertised-Name, USB-Hersteller, DAQ-E-IP/FW/…). Leere Werte werden weggelassen. |

| Parameter | Standardwert | Beschreibung |
| --- | --- | --- |
| `transports` | alle drei | Sequenz (oder CSV-Zeichenkette), die den Scan einschränkt. Es lohnt sich, diesen Parameter zu übergeben, wenn Sie wissen, was Sie wollen – BLE ist der langsame Teil. |
| `scan_timeout` | 5 | Scan-Fenster pro Transportart in Sekunden; das Backend begrenzt den Wert auf 1–20. |
| `timeout` | 60,0 | „HTTP“-Obergrenze für den gesamten Aufruf (wie auch an anderer Stelle im „SDK“). |
| `auto_start_backend` | `True` | Ein lokales Backend starten, falls noch keines läuft. Wird niemals für ein entferntes `backend_url` gestartet. |

> **Sensoren, die bereits im Pool werden nicht angezeigt.** Ein verbundenes BLE-Peripheriegerät stellt die Werbung ein, und ein geöffneter COM-Port kann nicht abgefragt werden; daher listet die Erkennung nur das auf, was *für eine Verbindung verfügbar* ist. Ein leeres Ergebnis unmittelbar nach dem Anschließen eines Geräts ist zu erwarten – verwende `list_daq_sensors()` für das, was du bereits hast. Transporte, deren Scan nicht ausgeführt werden kann (kein bleak / zeroconf installiert), werden übersprungen, anstatt einen Fehler auszulösen, sodass ein Rechner ohne Bluetooth dennoch seine USB- und ETH-Antworten erhält.

### Auflistung

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Gemeinsame Nutzung mit GUI / CLI

Wenn in der GUI bereits ein Sensor geöffnet ist, gibt der Aufruf von `connect_daq_sensor(port="COM3")` über „Python“ ein Handle mit der Kennung `already_connected=True` zurück. Das `disconnect()` der Sitzung ist dann eine No-Op-Operation, sodass Ihr Skript „SDK“ den Sensor bei Beendigung des Scopes nicht unter der GUI wegzieht .

### Direkt-Hardware-Klassen (ohne Backend)

`daq_sdk` wird von `chloros_sdk` erneut exportiert, sodass Sie Sensoren auch ohne Backend direkt im Prozess steuern können:

> **Verfügbarkeit:**`daq_sdk` ist im Lieferumfang der Desktop-Installation von „Chloros“ enthalten,**jedoch nicht** im PyPI-Paket – `pip install chloros-sdk` stellt Ihnen `lattice_sdk` zur Verfügung, lässt jedoch `chloros_sdk.DAQ_AVAILABLE == False`. Überprüfen Sie dieses Flag, bevor Sie diese Klassen verwenden; auf einem reinen pip-Host-Laufwerk sollten Sie stattdessen den Sensor über [`connect_daq_sensor()`](#daq-sensor-sessions) nutzen, der keine lokalen Transportbibliotheken benötigt.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Verwenden Sie vorzugsweise den Smart-Connect-Pfad (`connect_daq_sensor`), wenn Sie eine gemeinsame Nutzung mit der GUI wünschen; nutzen Sie die direkten Klassen für Headless-Skripte, die den Sensor exklusiv nutzen.

---

## Projektautomatisierung — `ChlorosProject`

Ein gespeichertes „Chloros“-Projekt ist ein Ordner, der `cameras.json` + `sensors.json` + `project.json` enthält. `open_project` lädt das Manifest, und `connect_all` schaltet jedes gespeicherte Gerät mit seinen gespeicherten Einstellungen online – im gleichen Hardwarezustand, den die GUI erzeugen würde.

### Minimalbeispiel

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Oder als Kontextmanager:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-Methoden

| Methode | Beschreibung |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Alle gespeicherten Geräte erkennen und verbinden. Gibt einen Verbindungsbericht pro Klasse zurück. Verwendet ein laufendes Backend, wenn eines auf `127.0.0.1:5000` im Listening-Modus ist; andernfalls weicht es stillschweigend auf die direkte (backend-freie) `lattice_sdk`-Gerätesteuerung – es wird niemals ein Backend gestartet. |
| `disconnect_all()` | Alle Verbindungen trennen. |
| `capture_all(output_dir=".")` | Ein Bild von jeder Kamera + Array + Spektrum von jedem Sensor. |
| `stream(camera, overlays=False, fps=10.0)` | Generator, der BGR-`numpy`-Bilder von einer benannten Kamera (oder Array). `overlays=False` ist eine direkte `lattice_sdk`-Grab-Schleife (Arrays liefern `{serial: frame}`-Dicts). `overlays=True` leitet über `ChlorosLocal.camera_stream()` → den MJPEG-Feed des Backends `/api/camera/<serial>/stream-annotated` weiter, wobei der gespeicherte Block der Kamera `ui.overlay` als Abfrageparameter weitergeleitet. Erfordert den Backend-Modus und eine **Standalone-Kamera**: Eine Kamera im Direktmodus löst `RuntimeError` aus (das Backend kanneine Kamera, die diesem Prozess gehört, nicht erfassen) und ein Array löst `NotImplementedError` aus (überlagert Composite-Bilder pro Kamera – streamt ein Element anhand des Namens). Einmalige Ausführung: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Führt die Ausrichtung für jedes derzeit verbundene Array durch. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Führt die Kalibrierungs-/Index-Pipeline für die Bilder des Projekts durch (umschließt `ChlorosLocal.process`; diese vier sind die **einzigen** akzeptierten kwargs – `indices=` usw. lösen `TypeError` aus; Indizes über `ChlorosLocal.configure()` festlegen). Erstellt verzögert ein `ChlorosLocal()`, das ein Backend automatisch startet. |

Attribute:
- `proj.cameras` — `Dict[str, CameraHandle]`, indiziert nach Name UND Seriennummer.
- `proj.arrays` — `Dict[str, ArrayHandle]`, indiziert nach Name UND array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, indiziert nach Name UND slot_id.
- `proj.config` — `project.json["config"]`-Dikt.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Verarbeitungsstufen.** `capture()`, `grab()` und `frame_stream()` verwenden alle dasselbe `processing`-Token,
und die Kette ist kumulativ – jede Ebene führt alle darüber liegenden aus:

| Ebene | Ausgabe | Anmerkungen |
| --- | --- | --- |
| `raw` | 1-Kanal-Bayer, sensornativ | Kein Demosaicing. Überlagerungen sind auf dieser Ebene nicht verfügbar. |
| `debayered` | 3-Kanal-BGR (**Standard**) | Bilineares Demosaicing. Die einzige Stufe, die ohne Backend-Modus funktioniert. |
| `radiance` | float32, W/m²/sr/nm | Vollständige radiometrische Kette: Demosaicing + 3×3-Entmischung (Multispektral) + DSNU + Flat-Field + NIST-Skala, wobei Belichtung × Verstärkung herausgerechnet werden, sodass die Werte absolut sind. |
| `reflectance` | uint16, 32768 = 1.0 | Strahlungsdichte geteilt durch die nach unten gerichtete Bestrahlungsstärke (ρ = π·L/E). Erfordert einen DLS/DAQ-Messwert – siehe Hinweis unten. |
| `display` | 8-Bit, sRGB-ähnlich | GUI-äquivalente Darstellung: CCM + Weißabgleich + Gamma über das aktive Farbprofil der Kamera. |

Alles andere als `debayered` erfordert den Backend-Modus; eine Kamera im Direktmodus löst
`NotImplementedError` aus. `reflectance` benötigt einen brauchbaren Messwert für dieWelling-Wert – der Frame-Endpunkt zieht
die gepoolte DAQ automatisch in den DLS-Slot der Kamera, aber ohne gebundene DAQ lehnt die Kette den
Reflexionsausgang ab und vermerkt die Herabstufung ehrlich in den zurückgegebenen Metadaten, anstatt stillschweigend
einfach ein minderwertiges Produkt zurückgibt.

> **Reflexions-DN-Skala – nicht fest einprogrammieren.** LATTICE-Reflexion verwendet `32768` = ρ 1,0 und kennzeichnet
> XMP `Chloros:PixelScale=32768`; Survey3-Reflexionsmessung verwendet `65535` = ρ 1,0 und enthält keine
> `Chloros:*`-Tags. Lies das Tag aus und dividiere durch diesen Wert. Er ist im uint16-Bereich definiert, bleibt also
> `32768` für jedes Format, das eine Skalierung vornimmt (16-Bit-TIFF, 8-Bit-PNG/JPG, 32-Bit-Prozent) – normalisiere
> den gespeicherten Datentyp zunächst zurück auf uint16 (×257 aus 8-Bit, ×65535 von Float). Die einzige Ausnahme:
> Eine als 8-Bit-TIFF gespeicherte 8-Bit-Quellaufnahme wird *beschnitten* und nicht skaliert, sodass keine Skalierung
> darauf zutrifft — Chloros lässt in diesem Fall `PixelScale` und das MicaSense-Tupel vollständig weg. Behandle ein fehlendes
> Tag in einer LATTICE-Reflexionsdatei als „keine gültige Skalierung“ und nicht als Standardwert.

> **EXIF-Daten werden beim Export übernommen.** `process()` kopiert den GPS-Block der Quellaufnahme
> **und dessen ExifIFD** in jedes Produkt, sodass Exporte `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` und `CameraSerialNumber` sowie die
> Georeferenzierung. `FocalLength` ist die Datei, anhand derer Pix4D den Ground-Sample-Abstand berechnet – ohne sie
> fällt die Rekonstruktion auf einen völlig falschen Maßstab zurück (in einem gemessenen Fall wurde ein 411 m großes Gelände
> in ein 47,8 km großes umgewandelt). Die Kopie ist bewusst nicht `-all:all`: Die Struktur-Tags von IFD0 stören
> die LATTICE-Ausgabe, und `ExifImageWidth`/`Height` werden ausgeschlossen, da sie die
> Quellaufnahme und nicht das exportierte Raster beschreiben.

Aufnahme-Stufen-Unterflags (gelten für die radiometrischen Ebenen – `radiance`, `reflectance`, `display`):

| Flag | Standard | Bedeutung |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + Flat-Field + 3x3-Unmix + radiometrische NIST-Skala. |
| `apply_white_balance` | `True` | WB-LUT. DLS-fähig, wenn ein DAQ mit der Kamera verbunden ist. |
| `apply_index` | `False` | Auswertung des Vegetationsindex. |
| `index_expression` | `None` | Formel überschreiben. Nichtleer → aktiviert den Index automatisch. |
| `annotated` | `False` | GUI-Dekorationen überlagern (Zebra/Raster/Peaking). Nicht verfügbar für `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Der Rückgabetyp ist `CapturePathMap`, nicht `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` ist `Dict[str, Union[str, List[str]]]`: Ein einstufiges
> `processing` weist jedem seriellen Element einen Pfad zu, während ein mehrstufiger Pfad (`"all"` oder eine
> explizite `levels`-Liste) ihm die **sortierte Liste** aller für diese
> Kamera gespeicherten Produkte. Ein Live-Komposit, falls eines gestreamt wird, wird unter dem zusätzlichen
> Schlüssel „`"combined"`“ statt unter einer Seriennummer bereitgestellt. Code, der von „`str`“ ausgeht, führt bei der
> Listenform zu einem Fehler, ohne dass der Typprüfer Einwände erhebt – in der Anmerkung stand nach der Veröffentlichung der Listenform eine Zeit lang „`Dict[str, str]`“,
> weshalb der Alias existiert. Normalisieren Sie
> die Form, wenn Sie die flache Form wünschen:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Array-Ausrichtung

`ArrayHandle` macht die gesamte Ausrichtungsfläche zugänglich. Profile sind standardmäßig nur sitzungsbezogen – rufen Sie `export_alignment()` explizit auf, um sie dauerhaft zu speichern.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Ausrichtung bei Verbindungsaufbau

`connect_all(align=...)` kannjedes Array bei der Verbindung ausrichten:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Greift auf `project.json["config"]["auto_align_on_connect"]` zurück, wenn nicht anders angegeben.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Direkte Hardware (ohne Backend)

Wenn Sie keinerlei Abhängigkeit vom Backend wünschen (CI, Headless-Roboter, Embedded), importieren Sie `lattice_sdk` und `daq_sdk` direkt — beide werden von `chloros_sdk` erneut exportiert. Hinweis zu `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` ist im PyPI-Paket enthalten (benötigt jedoch die Arena-Laufzeitumgebung SDK), während `daq_sdk` nur mit der Desktop-Installation ausgeliefert wird.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Voreinstellungen und der Auslöser

Drei der vier Voreinstellungen laufen im **Freilauf**-Modus: Die Kamera belichtet kontinuierlich und
`capture()` gibt das nächste Bild zurück. `triggered` bildet die Ausnahme – es schaltet die
Kamera für eine Hardware-Flanke auf Leitung 2 in den Aufnahmebereitschaftsmodus, sodass sie nichts aufnimmt, bis eine solche eintrifft.

| Voreinstellung | Auslöser | Verwendung |
| --- | --- | --- |
| `default` | Free-Run | Allgemeiner Gebrauch |
| `high_speed` | Free-Run | 8Bit, Begrenzung auf 60 fps, kurze Belichtungszeit |
| `high_quality` | Free-Run | 12 Bit, keine fps-Begrenzung – die übliche Wahl für Standbilder |
| `triggered` | **ausgelöst, Zeile 2** | die Kamera ist an ein M8-Synchronisationskabel angeschlossen und wird durch ein anderes Gerät ausgelöst |

Wenn Sie `triggered` auswählen (oder `trigger_mode="On"` selbst einstellen), ohne dass
an Leitung 2, kommt es bei jedem `capture()` zu einem Timeout – zu Recht, da Sie
die Kamera angewiesen haben, zu warten. Das „SDK“ erklärt dies, wenn es auftritt; siehe
[SC_ERR_TIMEOUT während der Aufnahme](#direct-hardware-backend-free).

> **Hinweis – „GVSP-Probe“ / `SC_ERR_TIMEOUT -1011`-Meldungen beim Verbindungsaufbau sind keine Fehler.**&gt; Beim Verbindungsaufbau versucht der „SDK“,**Jumbo-Frames** (9000-Byte-GVSP-Pakete) für einen höheren Durchsatz auszuhandeln. Bei einer direkten Punkt-zu-Punkt-Netzwerkverbindung (z. B. einer Verbindung-lokale `169.254.x.x`-Adresse) kann das Netzwerk in der Regel keine Jumbo-Frames übertragen, sodass diese Abfrage mit einer Zeitüberschreitung endet und Zeilen wie die folgenden protokolliert werden:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Dies ist der **vorgesehene Fallback**: Die „SDK“ wechselt automatisch zu Standardpaketen mit 1.500 Byte, und die Kamera stellt weiterhin eine normale Verbindung her (die folgenden Zeilen „`[chunk-enable …]`“ sind Teil der normalen Verbindungssequenz). Die Aufzeichnung funktioniert weiterhin.
>
> Sie können diese Probe überspringen, aber **sie dient nicht nur dazu, die Protokollierung zu unterdrücken – sie deaktiviert Jumbo-Frames.** Die Kamera antwortet mit „Don&#x27;t-Fragment-Pings nur bis zu einer Größe von 1500 Byte, ganz gleich, wie gut Ihr Netzwerk ist; daher kann der Ping-Test allein Jumbo-Frames niemals erkennen; diese Probe ist das einzige Mittel, das dies leisten kann. Deaktivieren Sie sie, und die Kamera verwendet in jedem Netzwerk dauerhaft Standardpakete mit 1500 Byte:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Das lohnt sich nur in einem Netzwerk, von dem Sie *wissen*, dass es keine Jumbo-Frames übertragen kann, wo dadurch pro Kamera etwa eine Sekunde Verbindungszeit eingespart wird. Da es sich um einen echten Kompromiss und nicht um eine rein kosmetische Änderung handelt, weist der „SDK“ nun bei Verwendung darauf hin:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Lassen Sie die Einstellung unverändert, es sei denn, Sie haben einen Grund dafür.** Bleibt die Option aktiviert, wird bei jeder Verbindung die tatsächliche Netzwerkkonfiguration neu ermittelt: Schließen Sie die Kamera an einen Jumbo-fähigen Switch an, und die nächste Verbindung erkennt Jumbo-Pakete automatisch – ganz ohne Konfiguration und ohne Neustart.
>
> Wenn Sie den Jumbo-Durchsatz *wünschen*, aktivieren Sie Jumbo durchgehend (NIC-MTU 9000 + ein Switch, der diese Pakete durchlässt) oder legen Sie die Größe mit `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` fest, wenn Sie wissen, dass die Verbindung dies unterstützt – allerdings ist die befehlsbezogene Einstellung mit `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` einer dauerhaften Festlegung vorzuziehen, da eine festgelegte Größe die Erkundung überspringt und die Anpassung an das vorangegangene Netzwerk verhindert. **Jedes** Gerät auf dem Pfad muss Jumbo-Pakete weiterleiten – einschließlich aller PoE-Splitter oder -Injektoren, was der übliche Grund dafür ist, dass eine ansonsten Jumbo--fähige Konfiguration diese nicht übertragen kann.

> **`SC_ERR_TIMEOUT -1011` während `capture()` / `grab*()` ist ein anderes Problem – das ist ein echter Fehler.**&gt; Der Hinweis oben bezieht sich nur auf `-1011`, das von der**Connect-Time-Probe**protokolliert wurde. Der gleiche Fehler, der bei einer**Erfassung** auftritt, bedeutet, dass die Kamera zwar erfolgreich verbunden wurde, aber keine Bilder sendet:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Ein eindeutiges Anzeichen ist eine Kamera, deren *Steuer*kanal einwandfrei funktioniert – die Erkennung klappt, Einstellungen und `[chunk-enable …]`-Schreibvorgänge sind alle erfolgreich –, während *jedes* Bild eine Zeitüberschreitung verursacht.
>
> **Die übliche Ursache ist, dass die Kamera für einen Hardware-Trigger aktiviert ist.** Bei `trigger_mode="On"` und `trigger_source="Line2"` sendet die Kamera überhaupt nichts, bis eine elektrische Flanke am M8-Synchronisationskabel anliegt. Wenn kein Kabel diese Leitung ansteuert, wartet jeder Grab-Vorgang unendlich lange. Die Kamera ist nicht defekt und das Netzwerk funktioniert einwandfrei – sie tut genau das, was ihr befohlen wurde.
>
> `CameraSettings()` sowie die Voreinstellungen `default` / `high_speed` / `high_quality` sind auf Freilauf eingestellt, und ein Grab, bei dem die Zeitüberschreitung auftritt, während aktiviert ist, erklärt sich von selbst, anstatt lediglich „`-1011`“ auszugeben. `PRESETS["triggered"]` aktiviert Line2, wie vorgesehen.
>
> So zwingen Sie eine beliebige Kamera in den Freilauf:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Wenn es mit `trigger_mode="Off"` immer noch zu einem Timeout kommt, liefert die Kamera tatsächlich keine Daten – senden Sie uns das Protokoll und `ip link show`.

#### Farbprofile (Live-Vorschau unter RGB) – `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` wählt das Farbprofil des Bildschirms für die **Live-Vorschau** bei „RGB“-Kameras aus (Multispec-Kameras ignorieren diese Einstellung):

| Profil | Bedeutung |
| --- | --- |
| `raw` | Die radiometrische Kette vollständig umgehen. |
| `linear` | DSNU + „Flat“ + Weißabgleich, kein CCM, kein Gamma. |
| `natural` | Linear + gemessenes CCM + sRGB-Gamma, nur mit der „Cheap“-Veredelung (Chroma-Glättung + Entsättigung der Lichter) – die realistische Standardeinstellung. |
| `enhanced` | `natural` plus die vollständige „Hub-Parity“-Nachbearbeitung (Farbstreifenentfernung, Lebendigkeit, lokaler CLAHE-Kontrast). Satteres Erscheinungsbild bei etwa **doppelten Bearbeitungskosten pro Bild**, daher eine niedrigere LIVE-Bildrate. |
| `custom_temp` | `natural`, jedoch mit auf `custom_cct_k` Kelvin (DLS wird ignoriert; backendseitig auf 2000–10000 K begrenzt). |

Das Profil ist ein **nur-Live-Vorschau**-Regler für Geschwindigkeit/Aussehen: Gespeicherte Aufnahmen erhalten immer die volle, reichhaltige Nachbearbeitung, unabhängig vom ausgewählten Profil; die Wahl von `natural`, um Frame-Zeit zurückzugewinnen, mindert also nicht die Qualität dessen, was auf der Festplatte landet. Ein unbekanntes Profil erhöht `ValueError`; wenn ein Chloros-Backend erreichbar ist, wird die Änderung auch an dieses gesendet, sodass der nächste Vorschaubild dies widerspiegelt (Benutzer von direct-SDK ohne Backend erhalten die Einstellungsänderung dennoch).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono- (M3M-)Kameras und `Calibration`

Eine Mono-**M3M**-Kamera (`M3M-<lens>-F<wavelength>`) ist einbandig: eine Graustufenebene, kein Bayer-Mosaik, keine 3×3-Spektral-Crosstalk-Matrix. `Calibration` erkennt dies und stellt ein `is_mono`-Flag. Die Reflektanz gilt weiterhin als radiometrische Karte pro Band (die Entmischung erfolgt über die Identitätsmatrix), aber multispektrale Berechnungen auf einer einzelnen Kamera liefern sinnvolle Ergebnisse statt Unsinn:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Um einen Vegetationsindex mit Monochrom-Hardware zu erstellen, kombinieren Sie mehrere M3M-Kameras mit unterschiedlichen Wellenlängen zu einem ausgerichteten Multiband-Stack (siehe [Array-Ausrichtung](#array-alignment)) und berechnen Sie den Index über diesen Stack hinweg statt auf einer einzelnen Kamera.

DAQ-Direktmodus:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` – zulässige Schlüssel**— genau `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; zugunsten von `cap_id` veraltet), `filter_model` (DAQ-M), sowie `cap_id` (alle DAQ-Typen; `None`/`""`/`"none"` = reiner Sensor, keine Kap-Korrektur). Unbekannte Schlüssel werden**stillschweigend ignoriert** – z. B. bewirkt `{"integration_time": 64}` nichts (es muss `integration_time_ms` sein). Gibt `{"applied": [...], "errors": {...}}` zurück und löst niemals eine Ausnahme aus.

`chloros_sdk` exportiert nur die oben verwendete Kernoberfläche erneut. Die vollständige öffentliche `daq_sdk` öffentliche „API“ (22 Namen) fügt Folgendes hinzu – importiere sie direkt aus `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Ausnahmen

Fangen Sie die Basisklasse ab, um „alles, was Chloros schiefgelaufen ist“ zu behandeln:

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

> `ChlorosAuthenticationError` und `ChlorosConfigurationError` werden zusammen mit den übrigen auf oberster Ebene exportiert; sie können, wie gezeigt, auch aus `chloros_sdk.exceptions` importiert werden.

Hierarchie:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## End-to-End-Beispiele

### 1. Einen Ordner mit einem benutzerdefinierten Fortschrittsbalken verarbeiten

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Live-LATTICE-Array → Reflexionsgrad + DAQ-Referenz

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektgesteuerte Erfassungskampagne

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Multi-Kamera-Frame-Stream → NumPy-Pipeline

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skript zur direkt auf der Hardware (ohne Backend) erfolgenden Erfassung ohne Benutzeroberfläche

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Funktionsprüfung vor dem Anschließen eines 4-Kamera-Arrays

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Äquivalent zum Aufnahmerezept (reines „Python“)

Die Rezept-DSL von „CLI“ hat ein direktes Äquivalent in „Python“:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Automatischer Start des Backends

Die Smart-Connect-Einstiegspunkte – `connect_camera`, `connect_array`, `connect_daq_sensor` und `discover_lattice_cameras` — sind Thin-HTTP-Clients, die davon ausgehen, dass ein Backend auf `127.0.0.1:5000` (dem Standard-URL der Smart-Connect-Oberfläche) auf Verbindungsanfragen wartet. Wenn die GUI oder „CLI“ bereits läuft, ist dies der Fall. Bei einem reinen Skript ist dies möglicherweise nicht der Fall – daher starten diese Funktionen **die mitgelieferte Backend-Binärdatei automatisch** (ohne Fenster, genau wie es `ChlorosLocal` tut) vor ihrem ersten Aufruf und warten dann bis zu `backend_startup_timeout` darauf, dass es hochfährt.

Regeln:

- **Es wird immer nur ein lokaler „URL“ gestartet.** Ein `backend_url`, der auf `localhost` / `127.0.0.1` / `[::1]` ist zulässig; jeder andere Host wird als Rechner eines anderen Benutzers angesehen und niemals gestartet.
- **Das Backend bleibt zur Wiederverwendung laufen** (genau wie bei „CLI“) – es erfolgt kein implizites Herunterfahren, wenn Ihr Skript beendet wird. Bei erneuter Ausführung des Skripts wird das aktive Backend wiederverwendet.
- **Deaktivieren Sie dies mit „`auto_start_backend=False`“** bei jedem dieser Aufrufe (z. B. wenn Sie auf ein Remote-Backend verwiesen haben, oder den Lebenszyklus des Backends selbst verwalten).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Wenn die mitgelieferte Binärdatei nicht gefunden oder gestartet werden kann, löst der nachfolgende Aufruf von `HTTP` einen bearbeitbaren, **plattformabhängigen** Fehler „`ChlorosConnectError`“ aus, anstatt einer bloßen– unter Windows verweist er Sie auf die Desktop-App oder einen Befehl „`chloros-cli`“; auf Linux (ohne GUI) verweist er auf einen Befehl vom Typ „`chloros-cli`“ oder auf „`.deb`“.

---

## Umgebung &amp; Header

Die „SDK“ kennzeichnet jeden Backend-HTTP-Aufruf mit „`X-Chloros-Client: sdk`“. Das Backend wendet die Lizenzierungsregeln von SDK / CLI an (Anmeldung **und** ein kostenpflichtiger Chloros+-Tarif erforderlich) anstelle des kostenlosen GUI-Zugangs. Dies wird beim Import automatisch festgelegt – Sie müssen nichts unternehmen.

`http://localhost` und `http://127.0.0.1` werden als lokales Backend erkannt. Aufrufe an andere Hosts (z. B. Ihren eigenen Analysedienst) bleiben unberührt.

Überschreiben Sie das Backend-URL, indem Sie `backend_url=` (oder `api_url=` auf `ChlorosLocal`) übergeben:

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Ein Nicht-Loopback-`backend_url` erreicht nur ein Source/Dev-Backend – mitgelieferte Backends binden nur an Loopback; siehe „Remote-Backend-Modus“ für das Tunnelmuster.)

---

## Versionierung und Kompatibilität

- Die Version „SDK“ wird als `chloros_sdk.__version__` bereitgestellt.
- „SDK“ bindet das Verhalten an die mitgelieferte Backend-Version. Die Kombination eines älteren „SDK“ mit einem neueren Backend funktioniert in der Regel (vorwärtskompatible Endpunkte), aber die Kombination eines neueren „SDK“ mit einem älteren Backend kann zu `404`-Fehler bei neuen Endpunkten – aktualisieren Sie die Desktop-App entsprechend.
- Die Smart-Connect-Oberfläche (`connect_camera` / `connect_array` / `connect_daq_sensor`) und der Endpunkt zur Netzwerkanalyse geben stabile Schemata von „JSON“ zurück; neue Felder werden zusätzlich bereitgestellt.

---

## Hinweise zur Fehlerbehebung

- **`ChlorosAuthenticationError: Login required`** → Führen Sie `chloros-cli login EMAIL PASSWORD` einmal auf diesem Rechner aus oder melden Sie sich über die Desktop-App „Chloros“ an.
- **`ChlorosConnectError: No Chloros backend is running …`** → Die Smart-Connect-Aufrufe starten automatisch ein lokales Backend, daher tritt diese Meldung nur auf, wenn die mitgelieferte Binärdatei nicht gefunden oder gestartet werden kann (z. B. bei einem reinen Pip-Host ohne Desktop-Paket). Die Meldung ist plattformabhängig: auf Windows die Desktop-App öffnen oder einen beliebigen `chloros-cli`-Befehl ausführen; auf Linux einen `chloros-cli`-Befehl ausführen (es gibt keine GUI) oder das `.deb` installieren. Für ein Remote-Backend übergeben Sie `backend_url=` (und `auto_start_backend=False`) übergeben.
- **`CAMERA_AVAILABLE == False`** beim Import → `lattice_sdk` konnte nicht geladen werden (in der Regel sind die Arena-SDK-Laufzeit-DLLs nicht installiert). Die Oberfläche ohne Kamera funktioniert weiterhin.
- **„Array connect“ gibt eine Auflösung unterhalb der nativen Auflösung zurück**→ Die „smart-Prep-Funktion des Backends verkleinert die Bildgröße automatisch, um sie an die Verbindung anzupassen. Verwenden Sie `analyze_array_network()`, um den Grund zu ermitteln, und aktualisieren Sie dann entweder die Verbindung, akzeptieren Sie die Verkleinerung oder übergeben Sie `force_tier="slip-emit-and-capture"` für die sequenzielle Erfassung. Das Sicherheitsnetz der Verkleinerung gilt**nicht** eine aggregierte Überbelegung abdecken (`oversubscribed: true`, fps-Felder 0): Zu viele Kameras für die Leitung lassen sich nicht durch Binning/ROI beheben – reduzieren Sie die Kameranzahl, aktivieren Sie Jumbo-Frames oder wechseln Sie zu einer schnelleren Netzwerkkarte (siehe [Überbelegung](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` meldet den NIC-Empfangsring als winzig (~0,26 MB) / Verbindungsgates mit „FRAMES WILL DROP“** → Der Empfangsring der Host-Netzwerkkarte befindet sich auf dem Standardwert (wird nach einer Aktualisierung des Netzwerkkartentreibers häufig auf 32 zurückgesetzt). Stellen Sie bei einem Realtek USB-10-GbE-Adapter die Werte `ReceiveBufferLen=256` und `PendingReceives=64` (erhöht) einund starten Sie anschließend das Backend neu, damit es den Ring erneut ausliest. Vollständige Vorgehensweise: [CLI Referenz → Einrichtung und Optimierung der Host-Netzwerkkarte](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Der Host hängt sich beim Neustart/Herunterfahren auf, später treten WMI-Fehler `Invalid class` auf / Netzwerkkarte lässt sich nicht aktivieren** → Veralteter USB-10-GbE-Treiber verursacht `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aktualisieren Sie den Adaptertreiber auf eine aktuelle Version (≥ 2026) und wenden Sie die Einstellungen für den Empfangsring erneut an. Siehe [CLI Referenz → Einrichtung und Optimierung der Host-Netzwerkkarte](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Reflexionsmessung abgelehnt** → Für die Reflexion im absoluten Maßstab muss ein aktives DAQ an die Kamera (oder das Array) gebunden sein. Führen Sie die Bindung entweder über die GUI durch oder verwenden Sie `processing="radiance"` (W/m²/sr/nm) verwenden, das keinen gekoppelten Sensor erfordert.
- **Die Erfassung mit `smart=True` dauert länger als erwartet** → Die AE-Konvergenz hängt von der Dynamik der Szene ab; erhöhen Sie den Wert für `exposure_tolerance_pct` oder verkürzen Sie den Wert für `stability_window_s`, wenn Sie einen schnelleren (weniger stabilen) Trigger wünschen.

---

## Siehe auch

- [CLI-Referenz](cli-reference.md) – Jeder Unterbefehl von „CLI“ entspricht einem Aufruf von „SDK“.
- [DAQ-Sensorhandbuch](../daq/README.md) – sensorspezifische Verkabelung, Kalibrierung und Aufzeichnungsregeln.
- Online-Dokumentation: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
