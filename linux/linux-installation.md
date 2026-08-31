# Linux-Installation

Chloros wird für Linux als `.deb`-Pakete bereitgestellt, die CLI und den Backend-Server installieren. Das Python SDK ist ein separates pip-Paket (das auch im `.deb` als versionsangepasstes Wheel enthalten ist).

Die Dateinamen der Pakete enthalten die Version und die Architektur: `chloros_1.2.0_amd64.deb` für x86_64 und `chloros_1.2.0_arm64_jp6.deb` für JetPack 6 Jetson-Builds. Ersetzen Sie in den folgenden Befehlen den Namen durch den der Datei, die Sie tatsächlich heruntergeladen haben.

***

## Linux amd64 (x86_64)

### Systemanforderungen

| Anforderung | Minimum | Empfohlen |
| --- | --- | --- |
| **Distribution** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Prozessor** | x86_64 (Intel/AMD) | Intel Core i7 oder besser |
| **Arbeitsspeicher (RAM)** | 8 GB | 16 GB oder mehr |
| **Grafikkarte** | Keine (CPU-Verarbeitung) | NVIDIA-GPU mit mindestens 4 GB VRAM (ab 12 GB wird `GPU_PARALLEL` freigeschaltet, ab 7 GB bleibt „Texture Aware“ im Single-Image-Pfad deaktiviert) |
| **Speicherplatz** | 2 GB freier Speicherplatz | SSD mit mindestens 10 GB freiem Speicherplatz |
| **Python** | Python 3.7+ (für SDK) | Python 3.10+ |

> **Ubuntu 20.04 und Debian 11 werden nicht unterstützt.** Die Abhängigkeitsliste des `.deb` leitet sich
> aus den tatsächlichen Verknüpfungen des Chloros-Backends ab, darunter
> `libc6 (>= 2.34)`. Sowohl Focal als auch Bullseye werden mit glibc 2.31 ausgeliefert, daher lehnt `apt` die
> Installation von vornherein ab, anstatt sie später zur Laufzeit scheitern zu lassen.

### Installation

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` löst die Abhängigkeiten nicht auf. Wenn fehlende Pakete gemeldet werden, schließt `sudo apt-get install -f` (oder `sudo apt --fix-broken install`) die Installation ab – dies ist der normale Ablauf und kein Fehler.
{% endhint %}

Überprüfen Sie die Installation:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Systemanforderungen

| Anforderung | Minimum | Empfohlen |
| --- | --- | --- |
| **Plattform** | NVIDIA Jetson mit JetPack 6 | Jetson Orin NX 16 GB oder AGX Orin |
| **JetPack** | JetPack 6.x | Aktuelles JetPack 6 |
| **Arbeitsspeicher (RAM)** | 8 GB (gemeinsam von GPU und CPU genutzt) | 16 GB+ gemeinsam genutzt (12 GB+ sind die Mindestanforderung für parallele GPU-Worker) |
| **Speicherplatz** | 2 GB freier Speicherplatz | NVMe-SSD mit 10 GB+ freiem Speicherplatz |
| **Python** | Python 3.7+ (für SDK) | Python 3.10+ |

### Installation

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Gleiches Layout wie das amd64-Modell `.deb`, mit einem für Jetson Orin / Orin NX / Orin Nano optimierten CUDA-Build. Informationen zum Speicher-, Wärme- und Einsatzverhalten von Jetson finden Sie im [NVIDIA Jetson-Handbuch](nvidia-jetson-guide.md).

***

## Python SDK-Installation (Alle Linux)

Das SDK ist ein reiner-Python-HTTP-Client für das Backend, daher funktioniert dasselbe Paket sowohl auf amd64 als auch auf arm64. Zwei Quellen:**Von PyPI** – die veröffentlichte stabile Version:

```bash
pip install chloros-sdk
```

**Aus dem mitgelieferten Wheel** – passt garantiert zu dem CLI/Backend, das Sie gerade installiert haben (verwenden Sie dies, wenn Ihr `.deb` neuer ist als das auf PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668-Distributionen** (Ubuntu 23.10+, Debian 12+) lassen keine systemweiten pip-Installationen zu. Verwenden Sie `pip install --user …`, eine virtuelle Umgebung oder `sudo pip install --break-system-packages …`. Der Paketinstallierer installiert SDK niemals automatisch in Ihrem System Python – diese Entscheidung bleibt Ihnen überlassen.
{% endhint %}

Optionale Extras:

| Extra | Befehl | Fügt hinzu |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` für Live-Fortschrittsanzeige |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` für BLE (DAQ-M)-Übertragung |

Überprüfen Sie das SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Das `.deb` installiert das Chloros, das CLI und das Backend. Das Python, das SDK kommuniziert mit diesem Backend über ein lokales HTTP API (`http://127.0.0.1:5000`) und startet es bei Bedarf automatisch. Verwenden Sie immer die literale IPv4-Adresse anstelle von `localhost` – `localhost` kann zu `::1` aufgelöst werden und kostet etwa zwei Sekunden pro Anfrage.
{% endhint %}

***

## Ersteinrichtung

### 1. Anmelden

Der Zugriff auf CLI und SDK erfordert eine kostenpflichtige Chloros+-Stufe (**Copper** oder höher), die serverseitig durchgesetzt wird: Ein abgemeldeter Aufrufer erhält `401 AUTH_REQUIRED`, und ein Aufrufer der kostenlosen Stufe (Iron) erhält `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Anmeldedaten werden in `~/.chloros/user_session.json` zwischengespeichert.

{% hint style="warning" %}
**Sie müssen sich nach jeder Installation oder jedem Upgrade erneut anmelden.** Das Skript „`prerm`“ des Pakets löscht bewusst „`~/.chloros/user_session.json`“ und die zwischengespeicherte Lizenz für jeden Benutzer auf dem Rechner, sodass eine neue Version die Lizenz immer neu validiert, anstatt einem veralteten Cache zu vertrauen.
{% endhint %}

### 2. Überprüfen Sie Ihren Lizenzstatus

```bash
chloros-cli status
```

`chloros-cli status` funktioniert in jeder Stufe (einschließlich der kostenlosen), sodass Sie immer sehen können, warum der Zugriff verfügbar ist oder nicht.

### 3. Führen Sie eine Systemdiagnose durch

```bash
chloros-cli selftest
```

Es werden nacheinander sieben Prüfungen durchgeführt, und der Befehl beendet sich mit einem Ergebnis ungleich Null, wenn eine davon fehlschlägt:

| # | Prüfung | Was wird überprüft? |
| --- | --- | --- |
| 1 | **Version** | CLI gibt seine Version an (`v1.2.0`). |
| 2 | **Port verfügbar** | Port 5000 ist frei *oder* wird bereits von einem funktionsfähigen Chloros-Backend beantwortet (was als erfolgreich gilt). |
| 3 | **Backend-Start** | Die Backend-Binärdatei wird gestartet. |
| 4 | **API-Test (`/api/test`)** | Das Backend antwortet mit `status: ok`. |
| 5 | **Systeminformationen** | Gibt `GPU: <name>, CUDA: <bool>, PyTorch: <version>` aus `/api/system-info` aus. |
| 6 | **Denoiser-Modelle** | Findet `*.pth.enc`-Modelle (auf Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Rauschunterdrückung**| „Texture Aware“ ist tatsächlich einsetzbar – benötigt CUDA**und** mindestens eine Modelldatei. |

Der Lauf endet mit `N/7 checks passed`, wobei etwaige Fehler namentlich aufgelistet werden.

### 4. Verarbeiten Sie Ihren ersten Datensatz

```bash
chloros-cli process ~/datasets/flight001
```

***

## Dateien und Verzeichnisse

### Pro Benutzer

Chloros speichert seine Anmeldedaten und die Konfiguration von CLI in einem einzigen plattformübergreifenden Verzeichnis, **`~/.chloros/`** (auf Windows, `%USERPROFILE%\.chloros\`). Zwei Linux-spezifische Caches folgen stattdessen den XDG-Konventionen – diese berücksichtigen `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, sofern diese festgelegt sind.

| Pfad | Zweck |
| --- | --- |
| `~/.chloros/user_session.json` | Von `chloros-cli login` geschriebener Cache für Anmeldesitzungen (wird bei jeder Paketinstallation/-aktualisierung gelöscht) |
| `~/.chloros/working_directory.txt` | Überschreibung des Standard-Projektordners (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI-Spracheinstellung (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Spracheinstellung, die mit der Windows-Benutzeroberfläche geteilt wird – ein `language` hat hier Vorrang vor `cli_language.json` |
| `~/.chloros/update_cache.json` | Einstündiger Cache für die Start-Update-Prüfung von Linux/Jetson |
| `~/.chloros/backend.log` | Backend-Protokoll, wenn das Backend von CLI gestartet wurde |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Zwischengespeicherte LATTICE-Kalibrierungspakete pro Kamera, sortiert nach Seriennummer und Bundle-Hash |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Optionale Benutzereingriffe für DAQ-Cap-Korrekturprofile |
| `~/.config/chloros/system_config.json` | Zwischengespeichertes Hardwareprofil aus der dynamischen Rechenanpassung – löschen Sie es, um eine neue Hardwareerkennung zu erzwingen |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-Serverprotokolle, eine Datei pro Start |
| `~/Chloros Projects/` | Standard-Projektordner, wenn keine Überschreibung festgelegt ist |

### Systemweit

| Pfad | Zweck |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-Skript – legt `LD_LIBRARY_PATH` für die gebündelten nativen Bibliotheken fest und führt anschließend die eigentliche Binärdatei aus |
| `/usr/bin/chloros-backend` | Wrapper-Skript – wie oben, zusätzlich mit `CHLOROS_PRODUCTION=1`, damit sich das Authentifizierungs-Gateway des Backends niemals stillschweigend selbst deaktivieren kann |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Die kompilierten Binärdateien |
| `/usr/lib/chloros/arena_runtime/` | Arena-Laufzeitumgebung SDK, die von LATTICE-Kameras benötigt wird |
| `/usr/lib/chloros/models/*.pth.enc` | Verschlüsselte Denoiser-Modelle, die vom Texture-Aware-Debayer verwendet werden |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK – Wheel, das genau zu diesem Build passt |
| `/usr/lib/chloros/exiftool` | Mitgeliefertes exiftool (nur dann per Symlink mit `/usr/local/bin/exiftool` verknüpft, wenn kein systemeigenes exiftool vorhanden ist) |
| `/etc/chloros/update.conf` | Konfiguration des Update-Kanals, die von `chloros-cli update` gelesen wird |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Setzt `net.ipv4.ip_unprivileged_port_start = 319` so, dass das Backend die PTP-Ports ohne Root-Rechte binden kann |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Richtet den dynamischen Loader auf `/usr/lib/chloros/arena_runtime` aus |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Gewährt dem angemeldeten Benutzer Zugriff auf die DAQ-U-USB-Seriell-Brücke (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Opt-in für den ständig aktiven Backend-Dienst (installiert, **nicht aktiviert**) |
| `/usr/share/applications/chloros-cli.desktop` | Eintrag „Chloros CLI“ im Anwendungsmenü, der ein Terminal öffnet |

## Speicherort der Backend-Ausführungsdatei

Die Komponenten „CLI“ und „SDK“ erkennen das Backend automatisch:

| Komponente | Pfad |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Überschreiben Sie den Backend-Pfad mit dem Flag `--backend-exe` CLI oder dem Konstruktorparameter `backend_exe` SDK und den Port mit `--port` (Standard: `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` verweist auf die Befehlsfamilien **`lattice`**,**`project`**, und**`daq pool-*`**-Befehlsfamilien auf ein entferntes Backend. Die Kernbefehle (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignorieren dies bewusst und zielen immer auf `http://127.0.0.1:<port>` ab.
{% endhint %}

***

## LATTICE-Kameras und DAQ-Lichtsensoren auf Linux

Alle Befehlsfamilien für Live-Hardware funktionieren auf dem Linux (amd64 und Jetson):

* **`chloros-cli lattice`** – Erkennen, Verbinden, Konfigurieren und Aufzeichnen von LATTICE-Kameras und synchronisierten Arrays. `.deb` bündelt die dafür erforderliche Arena SDK-Laufzeitumgebung und registriert sie beim dynamischen Loader.
* **`chloros-cli daq pool-*`** – Verbinden von DAQ-U/M/E-Lichtsensoren über den Backend-Pool, Streamen kalibrierter Spektren und Aufzeichnen von `.daq`-Dateien. Die kompilierte Version CLI enthält ausschließlich die `pool-*`-Familie: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** – Führt ein gespeichertes Projekt (mit seinen Kameras, Sensoren und Verarbeitungseinstellungen) im Headless-Modus aus.
* **`chloros-cli time-sync`** — den PTP-Grandmaster überprüfen, für den das Chloros-Backend für LATTICE-Kameras und DAQ-E-Sensoren läuft.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` wird von `pool-latest`, `pool-stream`, `pool-record` sowie `pool-set-cap` benötigt; `pool-list` zeigt die derzeit im Pool befindlichen IDs an.

{% hint style="info" %}
**Verwenden Sie vorzugsweise `--eth-host` für die erste DAQ-E-Verbindung auf einem Rechner mit mehreren Netzwerkanschlüssen.** Die automatische Erkennung durchsucht mDNS und kann die Schnittstelle des Sensors aufgrund eines leeren ARP-Caches übersehen, sodass die erste `pool-connect --eth`-Verbindung nach dem Systemstart fehlschlagen kann, selbst wenn der Sensor einwandfrei funktioniert. Durch die Übergabe der IP-Adresse oder des Hostnamens des Sensors wird die Erkennung vollständig übersprungen.
{% endhint %}

**Die seriellen Berechtigungen für DAQ-U** werden durch die installierte udev-Regel (`uaccess` + Gruppe `dialout`) verwaltet. Sollte ein bereits angeschlossener Sensor weiterhin nicht erreichbar sein, laden Sie die Regeln neu oder schließen Sie ihn erneut an:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Den vollständigen Befehlssatz finden Sie in der [CLI-Referenz](../CLI.md).

### Ständig aktives PTP für Headless-Hosts

Bei der Erstinstallation wird die systemd-Einheit `chloros-backend.service` generiert, ist jedoch **nicht aktiviert**. Auf einem Jetson ohne Bildschirm oder einem Server, auf dem die PTP-Zeitsynchronisation für DAQ-E-Sensoren und LATTICE-Kameras kontinuierlich laufen soll, aktivieren Sie sie:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Ohne diese Unit läuft PTP nur, solange das Backend Chloros aktiv ist – also während einer aktiven CLI-/SDK-Sitzung.

Das Gerät bindet das Backend an `127.0.0.1:5000` (Umgebungseinstellungen `CHLOROS_HOST` / `CHLOROS_PORT` innerhalb des Geräts; Überschreiben mit `sudo systemctl edit chloros-backend.service`) und startet es bei einem Fehler nach 5 Sekunden neu.

**Wie PTP seine Ports erhält.** PTP verwendet UDP 319/320, beide unterhalb der normalen Grenze von 1024 für privilegierte Ports. Das `postinst` des Pakets schreibt `/etc/sysctl.d/60-chloros-ptp.conf` mit `net.ipv4.ip_unprivileged_port_start = 319`, wodurch das Backend diese Ports binden kann, während es als Ihr Benutzer läuft. Als zusätzliche Sicherheitsmaßnahme wird außerdem „`setcap cap_net_bind_service,cap_net_raw=+ep`“ auf die Backend-Binärdatei angewendet – deshalb ist „`libcap2-bin`“ eine deklarierte Abhängigkeit des Pakets.***

## Beispiele für Bash-Skripte

{% hint style="info" %}
**Skriptfreundliche Exit-Codes.**`chloros-cli process` beendet `0` bei Erfolg und**mit einem Wert ungleich Null bei einem Fehler – einschließlich eines Laufs, bei dem Bildprodukte angefordert, aber keine geschrieben wurden** (es gibt `Processing finished but wrote no image products.` aus und nennt den Projektordner sowie die üblichen Ursachen). Erfolgreiche Läufe melden, wie viele Bildprodukte geschrieben wurden (`Image products written: N`). Beendigungscodes: `0` – Erfolg, `1` – Fehler, `2` – Argumentfehler, `130` – unterbrochen.
{% endhint %}

### Mehrere Datensätze verarbeiten

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Mit benutzerdefinierten Einstellungen verarbeiten

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Es gibt genau vier gültige `--format`-Werte, die Leerzeichen enthalten – setzen Sie diese daher immer in Anführungszeichen:

| `--format`-Wert | Ausgabeordner |
| --- | --- |
| `TIFF (16-bit)` *(Standard)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` akzeptiert `standard` (Standard) oder `texture-aware` (Chloros+).

### Automatisierte Verarbeitung mit Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Beispiel für Python und SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Fehlerbehebung

### CLI nach der Installation nicht gefunden

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Zugriff verweigert

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### „setcap fehlgeschlagen“ während der Installation

`.deb` wendet `cap_net_bind_service` auf `/usr/lib/chloros/chloros-backend` an, damit es die PTP-Ports 319/320 ohne Root-Rechte binden kann. Falls `libcap2-bin` zum Zeitpunkt der Installation fehlte, wird der Aufruf übersprungen. Installieren Sie es und installieren Sie das Paket erneut:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP startet nicht / kann Port 319 nicht binden

Vergewissern Sie sich, dass die Untergrenze für nicht privilegierte Ports gesenkt wurde, und wenden Sie diese gegebenenfalls erneut auf den aktuellen Start an:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Überprüfen Sie anschließend den Grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### „LATTICE-Kameratreiber nicht gefunden“

Die Arena-Laufzeitumgebung SDK wird nicht aufgelöst. Vergewissern Sie sich, dass die vom Paket geschriebene Loader-Konfiguration vorhanden und aktualisiert ist:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Backend konnte nicht gestartet werden

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Die Backend-Protokolle zum fehlgeschlagenen Start befinden sich in `~/.cache/chloros/logs/`.

### CUDA nicht erkannt

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` meldet dasselbe in einer Zeile: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Fehlende gemeinsam genutzte Bibliotheken

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Langsamer Start auf SD-Karten-Systemen

Die kompilierten Binärdateien entpacken sich bei jedem Start in ein temporäres Verzeichnis. Falls `/mnt/ssd/tmp` vorhanden ist, verwendet Chloros dieses automatisch; andernfalls setzen Sie `TMPDIR` auf ein schnelles Dateisystem:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Aktualisierung von Chloros auf Linux

Der Befehl `update` ist nur für Linux/Jetson verfügbar. Er überprüft die Version, die im unter `/etc/chloros/update.conf` konfigurierten Update-Kanal veröffentlicht wurde, und bietet an, die passende Version `.deb` herunterzuladen und zu installieren:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Auf Linux/Jetson führt der Befehl CLI zudem bei jedem Start eine nicht blockierende Update-Prüfung durch (das Ergebnis wird eine Stunde lang in `~/.chloros/update_cache.json` zwischengespeichert) und gibt „`Update available: vX.Y.Z`“ aus, wenn eine neuere Version vorliegt. Ihre Einstellungen und Projekte bleiben bei einem Update erhalten; Sie müssen sich anschließend erneut anmelden.

## Deinstallation

```bash
sudo apt remove chloros
```

Die Deinstallation beendet `chloros-backend.service`, stellt die standardmäßige Untergrenze für nicht privilegierte Ports (1024) wieder her, entfernt den Symlink zum mitgelieferten „exiftool“ und die Arena-Loader-Konfiguration sowie löscht zwischengespeicherte Anmeldedaten. Ihre Projekte und `~/.chloros/`-Datendateien bleiben unberührt.

***

## Nächste Schritte

* [NVIDIA Jetson-Handbuch](nvidia-jetson-guide.md) – Jetson-spezifische Optimierung und Bereitstellung
* [CLI : Befehlszeile](../CLI.md) – das CLI-Handbuch
* [API : Python SDK](../api-python-sdk.md) – das SDK-Handbuch
* [CLI-Referenz](../reference/cli-reference.md) und [SDK-Referenz](../reference/sdk-reference.md) – vollständige Befehls-/API-Listen für 1.2.0
* [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md) – wie sich Chloros an Ihre Hardware anpasst
