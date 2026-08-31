# API : Python SDK

{% hint style="info" %}
**Suchen Sie nach dem vollständigen API?** Diese Seite ist ein praktisches Tutorial. Alle öffentlichen Klassen, Methoden, genauen Signaturen und kopierbaren Beispiele finden Sie in der [SDK-Referenz](reference/sdk-reference.md), die für KI-Assistenten optimiert ist.**Arbeiten Sie mit einem KI-Assistenten?** Fügen Sie diesen URL in den Chat ein, damit er über die vollständige, aktuelle Chloros 1.2.0 API verfügt:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Jede Seite dieses Handbuchs ist als Markdown-Rohdatei unter ihrem Slug in Kleinbuchstaben + `.md` verfügbar, und das gesamte Handbuch ist unter `https://mapir.gitbook.io/chloros/llms.txt` indexiert.
{% endhint %}

Das **Chloros Python SDK** (`chloros-sdk` auf PyPI) steuert alle Funktionen der Desktop-Anwendung ab Python: Stapelverarbeitung von Bildern, Live-Steuerung von LATTICE-Kameras und -Arrays, DAQ-Lichtsensor-Sitzungen sowie die Automatisierung gespeicherter Projekte. Es handelt sich um eine dünne Schicht über demselben lokalen Backend, das auch die GUI und CLI verwenden (HTTP auf `127.0.0.1:5000`), sodass das Verhalten auf allen drei Oberflächen identisch ist.

## Installation

Die Installation erfolgt in zwei Schritten: Zunächst das Desktop-Paket Chloros (es stellt das Verarbeitungs-Backend und die Hardware-Laufzeitumgebungen bereit), anschließend das Paket Python.

**Schritt 1 – Chloros installieren.** Windows: Führen Sie das Desktop-Installationsprogramm (Standardpfad `C:\Program Files\MAPIR\Chloros\`) von der [Download](download.md)-Seite aus. Linux: Installieren Sie das `.deb`-Paket ([Linux-Installation](linux/linux-installation.md)).**Schritt 2 – Installieren Sie SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Möglicherweise benötigen Sie nicht einmal pip: Jeder Installer enthält ein passendes SDK-Wheel. Der Windows-Installer installiert es automatisch in Ihrem System Python; der Linux `.deb` legt es unter `/usr/lib/chloros/sdk/` ab und gibt den genauen Befehl `pip install --user` aus. PyPI wird bei Release-Builds aktualisiert, sodass `pip install chloros-sdk` der neuesten stabilen Version entspricht.

**Schritt 3 – Einmal pro Rechner anmelden:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Die Anmeldedaten werden in `~/.chloros/` zwischengespeichert (beide Plattformen). Auf Windows können Sie sich alternativ über die Registerkarte „Benutzer<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">“ der Desktop-App anmelden. Für SDK ist ein kostenpflichtiger Chloros+-Tarif erforderlich – siehe [Lizenzanforderungen](#license-requirement) weiter unten.

| Voraussetzung | Details |
| --- | --- |
| **Chloros installiert** | Windows: Desktop-Installationsprogramm; Linux: `.deb`-Paket (stellt die Backend-Binärdatei bereit) |
| **Python** | 3.7 oder höher (entwickelt/getestet auf 3.10) |
| **Betriebssystem** | Windows 10/11 64-Bit, Ubuntu 22.04 LTS oder neuer oder NVIDIA Jetson (JetPack 6) |
| **Lizenz** | Aktives Chloros+-Konto, beliebige kostenpflichtige Stufe (Copper oder höher) |

## Erfolg in 60 Sekunden

Ein einziger Aufruf erstellt ein Projekt, importiert einen Ordner, konfiguriert die Verarbeitung und führt die Pipeline aus – dabei wird das Backend automatisch gestartet, falls noch keines läuft:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Verwenden Sie bei Linux die Pfade von Linux: `/home/user/drone_images/flight001`. Der Befehl SDK funktioniert auf beiden Plattformen identisch.)

Verarbeiten Sie einen LATTICE-Aufnahmeordner? Verwenden Sie den LATTICE-kompatiblen Wrapper – er wendet die richtigen Standardeinstellungen an (keine Erkennung des Panel-Ziels, Standard-Debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` – vollständige Steuerung der Pipeline

Für alles, was über einen Einzeiler hinausgeht, verwenden Sie `ChlorosLocal`. Es startet das Backend bei der ersten Verwendung (`auto_start_backend=True`), erstellt und konfiguriert Projekte, überwacht den Fortschritt und gibt nach Abschluss eine Zusammenfassung aus.

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
Behalten Sie den Standardwert `http://127.0.0.1:5000` bei, anstatt ihn durch `localhost` zu ersetzen – bei Windows wird „`localhost`“ zunächst zu „`::1`“ aufgelöst und kostet bei einem reinen IPv4-Backend etwa 2 Sekunden pro Anfrage.
{% endhint %}

Verwenden Sie es als Kontextmanager für eine garantierte Bereinigung:

```python
import chloros_sdk

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

`configure()` akzeptiert folgende Schlüsselwörter: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` und `custom_settings`. Die wichtigsten Werte:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Die LATTICE-spezifischen Regler (`input_level`, `radiometric_output`, die `array_alignment*`-Familie) sind mit ihren vollständigen Wertetabellen in der [SDK-Referenz](reference/sdk-reference.md#supported-values) dokumentiert.

### Fortschrittsanzeige

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

Nach Abschluss des Laufs fügt `process()` die Verarbeitungszusammenfassung des Backends als `result["summary"]` an. Jeder Eintrag in `summary["hints"]` ist ein vollständiger Satz, der alles Bemerkenswerte erklärt – zum Beispiel, warum ein Lauf keine Ausgabe erzeugt hat – und jeder Hinweis wird zusätzlich als Python `UserWarning` ausgegeben, sodass leere Läufe selbstdiagnostisch sind, selbst wenn Sie das Wörterbuch nie überprüfen:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` wird nicht ausgelöst, wenn ein Lauf keine Bilder erzeugt.** Dies ist der einzige Punkt, an dem sich SDK und CLI bewusst unterscheiden: `chloros-cli process` behandelt „Produkte wurden angefordert, es wurden keine geschrieben“ als Fehler und beendet den Lauf mit einem Wert ungleich Null, während SDK den Lauf normal beendet und den Zustand über `summary` bzw. Hinweise meldet. Sollte Ihre Pipeline bei einem leeren Durchlauf anhalten, überprüfen Sie dies selbst – sehen Sie sich `summary` an (oder zählen Sie die Dateien im Projektordner), anstatt sich auf eine Ausnahme zu verlassen.
{% endhint %}

## Smart Connect – Live-Hardware

Drei Hilfsprogramme öffnen dauerhafte Sitzungen im Hardware-Pool des Backends – demselben Pool, den auch die GUI nutzt, sodass die Skripte unter SDK neben der Desktop-Anwendung laufen können, ohne sich um serielle Schnittstellen oder Netzwerkbandbreite zu streiten. Alle drei starten automatisch ein lokales Backend, falls noch keines läuft.

### Einzelne LATTICE-Kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Synchronisiertes Array — `connect_array`

`connect_array` ist der empfohlene Einstiegspunkt für Mehrkamera-Rigs. Es führt denselben „Smart-Prep“-Ablauf wie die GUI aus: Netzwerkanalyse, automatische Auswahl der Synchronisationsstufe, PTP-Zeitsynchronisation, Auswahl des Pixelformats pro Kamera, AE-Seeding und Aktivierung des GPIO-Triggers. Die **erste serielle Schnittstelle ist der Master** (sie löst den Hardware-Triggerimpuls aus); die übrigen sind Slaves.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Fügen Sie `smart=True` zu jeder Array-Aufnahme hinzu, um vor dem Auslösen abzuwarten, bis sich die automatische Belichtung über alle Kameras hinweg eingependelt hat. Informationen zu Aufnahmemodi (Einzelaufnahme / Serienaufnahme / Intervallaufnahme / Schnellste Aufnahme), Rekordern, Burst-to-Video und Array-Ausrichtung finden Sie in der [SDK-Referenz](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ-Lichtsensor — `connect_daq_sensor`

Ohne Argumente erkennt `connect_daq_sensor()` automatisch das Übertragungsprotokoll (Priorität: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Jeder Frame enthält den 135-Punkte-Wert `spectrum` (W/m²/nm nach Kalibrierung), ein `is_saturated`-Flag sowie CIE `x`, `y`, `z`. Um einen bestimmten Sensor oder Transport festzulegen – die zuverlässige Wahl auf Hosts mit mehreren Netzwerkschnittstellen, wo die automatische Ethernet-Erkennung einen funktionsfähigen DAQ-E beim ersten Versuch übersehen kann –, übergeben Sie einen expliziten Hinweis:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Beachten Sie, dass Profile zur Cap-Korrektur (`cap_id`) **keine** SDK-Einstellung sind – wählen Sie diese stattdessen über `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` aus.

### Gespeicherte Projekte – `open_project`

Ein gespeichertes Chloros-Projekt behält die angeschlossene Hardware bei (`cameras.json` + `sensors.json` sowie `project.json`), und `chloros_sdk.open_project(path)` kann alle Verbindungen auf einmal wiederherstellen und Erfassungen anhand des Gerätenamens steuern. Siehe [Projektautomatisierung](reference/sdk-reference.md#project-automation--chlorosproject) in der Referenz.

## Was eine reine pip-Installation bietet

Überprüfen Sie die Verfügbarkeitsflags auf Modulebene, bevor Sie Hardware-Oberflächen verwenden:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Auf einem Host, auf dem **nur** `pip install chloros-sdk` und kein Chloros-Desktop-Paket installiert ist:

* `ChlorosLocal`, `process_folder` und `process_lattice_capture` funktionieren **nicht** – sie benötigen die Backend-Binärdatei, die im Desktop-Installationsprogramm enthalten ist.
* Die Smart-Connect-Helfer (`connect_camera`, `connect_array`, `connect_daq_sensor`) sind reine HTTP-Clients, funktionieren also mit einem Backend auf einem anderen Rechner – die mitgelieferten Backends binden jedoch nur auf Loopback, sodass Sie den Port selbst weiterleiten müssen (z. B. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) und `backend_url="http://127.0.0.1:5000"` zusammen mit `auto_start_backend=False` übergeben. Siehe [Remote-Backend-Modus](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Die LATTICE-Klassen für direkte Hardware (`LatticeCamera`, `CameraPool`, …) lassen sich zwar importieren, benötigen jedoch die Arena-Laufzeitumgebung SDK aus dem Desktop-Paket – ohne diese ist `CAMERA_AVAILABLE` gleich `False`.
* `daq_sdk` (die direkten DAQ-Klassen) ist im Desktop-Paket enthalten, nicht im PyPI-Paket, daher ist `DAQ_AVAILABLE` auf einem reinen pip-Host gleich `False` – steuern Sie DAQ-Sensoren stattdessen über `connect_daq_sensor()` gegen ein (getunneltes) Backend an.

## Lizenzvoraussetzungen

Für den Zugriff auf SDK ist ein aktives Chloros+-Konto in einer kostenpflichtigen Stufe erforderlich – **Copper oder höher**(Copper / Bronze / Silver / Gold); die kostenlose Stufe „Iron“ bietet keinen Zugriff auf SDK/CLI. Die Durchsetzung erfolgt**serverseitig**: Jede SDK-Anfrage muss sowohl eine aktive Sitzung als auch einen kostenpflichtigen Tarif enthalten, andernfalls gibt das Backend `403` / `PLAN_UPGRADE_REQUIRED` zurück (ausgelöst als `ChlorosLicenseError` durch `ChlorosLocal` und als `ChlorosConnectError` durch die `connect_*`-Helfer). Ein abgemeldeter Aufrufer erhält stattdessen `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) – die erneute Ausführung von `chloros-cli login` behebt den ersten Fall, nicht jedoch den zweiten.

Die Offline-Nutzung funktioniert innerhalb der Karenzzeit des Tarifs: Die Tarifstufe wird aus dem Server-Validierungs-Cache (5 Minuten) oder dem signierten, maschinengebundenen Lizenz-Cache (30 Tage bei monatlichen Tarifen; bis zum Ablauf des Abonnements bei jährlichen Tarifen) gelesen. Nach Ablauf der Karenzzeit wechselt der Tarif in die kostenlose Stufe, und der Zugriff über SDK wird unterbrochen, bis der Rechner einmal eine Verbindung zum Server hergestellt hat. `chloros-cli status` bleibt in der kostenlosen Stufe erreichbar, sodass der Grund immer sichtbar ist. Siehe [Chloros+ Anmeldung](chloros+-login.md).

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

Alle Pipeline-Ausnahmen (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) leiten sich von `ChlorosError` ab. Eine Besonderheit: `ChlorosConnectError` – wird nur von `connect_camera` / `connect_array` / `connect_daq_sensor` ausgelöst — leitet sich vom einfachen `Exception` ab, **nicht** von `ChlorosError`, sodass `except ChlorosError` diesen Fehler nicht abfängt. Die vollständige Hierarchie findet sich in der [SDK-Referenz](reference/sdk-reference.md#exceptions).

## Siehe auch

* [SDK-Referenz](reference/sdk-reference.md) – die vollständige API-Oberfläche, optimiert für KI-Assistenten.
* [CLI-Referenz](reference/cli-reference.md) – jeder CLI-Unterbefehl entspricht einem SDK-Aufruf.
* [Download](download.md) — Installationsprogramme für Windows und Linux.
