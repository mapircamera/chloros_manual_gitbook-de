# Linux-Installation

Chloros wird für Linux als `.deb`-Pakete bereitgestellt, die die CLI, den Backend-Server und das Python-SDK installieren (für das System `python3` sowie ein versionskompatibles Wheel für virtualenvs).

Die Dateinamen der Pakete enthalten die Versionsnummer und die Architektur: `chloros_1.3.0_amd64.deb` für x86_64, und `chloros_1.3.0_arm64_jp6.deb` für arm64 (NVIDIA Jetson, JetPack 6). Ersetzen Sie in den folgenden Befehlen den Namen durch den der Datei, die Sie tatsächlich heruntergeladen haben.

{% hint style="info" %}
**Neu in 1.3.0: Die Programme werden unkomprimiert ausgeliefert.** Das Backend und die CLI werden als Ordner installiert, `/usr/lib/chloros/backend/` und `/usr/lib/chloros/cli/`, anstelle einzelner selbstextrahierender Dateien. Sie entpacken sich nicht mehr bei jedem Start in einen temporären Ordner, sodass sie deutlich schneller starten (auf x86_64 benötigte das Backend früher bei jedem Start etwa 100 Sekunden zum Entpacken) und benötigen nicht mehr mehrere GB freien Speicherplatz in „`/tmp`“. Auch die amd64-Version „`.deb`“ ist nur noch etwa halb so groß wie zuvor.
{% endhint %}

***

## Linux-amd64 (x86_64)

### Systemanforderungen

| Anforderung | Minimum | Empfohlen |
| --- | --- | --- |
| **Distribution** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Prozessor** | x86_64 (Intel/AMD) | Intel Core i7 oder besser |
| **Arbeitsspeicher (RAM)** | 8 GB | 16 GB oder mehr |
| **Grafikkarte** | Keine (CPU-Verarbeitung) | NVIDIA-GPU mit mindestens 4 GB VRAM (ab 12 GB wird `GPU_PARALLEL` freigeschaltet, ab 7 GB bleibt „Texture Aware“ im Einzelbild-Pfad deaktiviert) |
| **Speicherplatz** | 2 GB freier Speicherplatz | SSD mit mindestens 10 GB freiem Speicherplatz |
| **Python** | Python 3.7+ (für das SDK) | Python 3.10+ |

> **Ubuntu 20.04 und Debian 11 werden nicht unterstützt.** Die Abhängigkeitsliste von `.deb`
> leitet sich aus den tatsächlichen Verknüpfungen des Chloros-Backends ab, und dazu gehört eine
> `libc6`, das neuer ist als die in „focal“ und „bullseye“ enthaltene glibc 2.31; daher lehnt `apt` die
> Installation von vornherein ab, anstatt sie später zur Laufzeit scheitern zu lassen.

### Installation

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Verwenden Sie `apt install ./…`, nicht `dpkg -i`.** `apt` löst die Abhängigkeiten des Pakets auf und installiert zudem die empfohlenen Komponenten: die Anforderungen des Python-SDK, `python3-numpy` und `python3-requests`. `dpkg -i` installiert keines von beiden. Die CLI und das Backend sind eigenständig und laufen auch ohne diese Komponenten, aber `import chloros_sdk` benötigt sie; die Installationsausgabe listet alle fehlenden Komponenten auf. Behalten Sie `./` bei, damit `apt` die Datei installiert, anstatt nach einem Paket mit diesem Namen zu suchen. Das Paket deklariert außerdem `libgl1`, das vom mitgelieferten OpenCV geladen wird und in einem minimalen Server-Image möglicherweise fehlt.
{% endhint %}

Überprüfen Sie die Installation:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Systemanforderungen

| Anforderung | Minimum | Empfohlen |
| --- | --- | --- |
| **Plattform** | NVIDIA Jetson mit JetPack 6 | Jetson Orin NX 16 GB oder AGX Orin |
| **JetPack** | JetPack 6.x | Aktuelles JetPack 6 |
| **Arbeitsspeicher (RAM)** | 8 GB (gemeinsam genutzt von GPU und CPU) | 16 GB+ gemeinsam genutzt (12 GB+ sind die Mindestanforderung für die parallele Verarbeitung) |
| **Speicherplatz** | 2 GB freier Speicherplatz | NVMe-SSD mit mindestens 10 GB freiem Speicherplatz |
| **Python** | Python 3.7+ (für das SDK) | Python 3.10+ |

### Installation

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Gleiches Layout wie beim amd64 `.deb` (einschließlich Python-SDK, Empfangs-Puffer und Backend-Dienste) für Jetson Orin / Orin NX / Orin Nano auf **JetPack 6** (L4T R36). Das enthaltene PyTorch ist der CUDA-Build von JetPack 6 für Orin, sodass die GPU-Verarbeitung – einschließlich des „Texture Aware“-Debayers – auf der GPU des Jetson läuft. Das Paket enthält CUDA selbst nicht: Es nutzt die CUDA- und cuDNN-Bibliotheken, die JetPack 6 installiert (`sudo apt install nvidia-jetpack` auf einem Image, auf dem diese fehlen). Wenn keine CUDA-GPU verfügbar ist, läuft die Verarbeitung auf der CPU, und `--debayer texture-aware` verwendet den Standard-Debayer (das Backend protokolliert eine Zeile `[DEBAYER] … needs a CUDA GPU …`, in der der Grund dafür angegeben wird). Informationen zum Speicher, zur Wärmeentwicklung und zum Einsatzverhalten von Jetson im Feld finden Sie im [NVIDIA Jetson-Handbuch](nvidia-jetson-guide.md).***

## Installation des Python-SDKs (alle Linux-Systeme)**Das `.deb` installiert das SDK für Sie.** Beide Pakete (amd64 und Jetson) installieren es für das System `python3`, sodass `import chloros_sdk` direkt nach der Installation funktioniert – ohne pip und ohne Netzwerkverbindung. Seine Pakete (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` sowie die Projektlader) werden in `/usr/lib/python3/dist-packages` integriert, das zum Paket gehört: Bei einem Upgrade werden sie ersetzt, und beim Entfernen des Pakets werden sie entfernt. Die Version stimmt immer mit der von Ihnen installierten CLI und dem Backend überein.

Die eigenen Anforderungen des SDKs, `python3-numpy` und `python3-requests`, sind **empfohlen, aber nicht erforderlich** vom Paket: `sudo apt install ./chloros_<version>_<arch>.deb` installiert sie, `dpkg -i` hingegen nicht. Die Installationsausgabe gibt die von `python3` importierte SDK-Version an, sowie alle fehlenden Anforderungen sowie alle `chloros_sdk`-Dateien an anderen Stellen im Pfad von `python3`, die die Kopie des Pakets überschreiben (z. B. eine zuvor mit `pip install --user` installierte Version).**In einer virtualenv**, die die Systempakete nicht sieht, installieren Sie das mitgelieferte Wheel darin (vollständig offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Von PyPI** – nur für einen Rechner, auf dem kein Chloros-Paket installiert ist und dessen SDK ein Backend auf einem anderen Rechner steuert (siehe [Remote-Backend-Modus](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Optionale Extras für eine pip-Installation (in einer virtualenv oder über PyPI):

| Extra | Befehl | Fügt hinzu |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` für Live-Fortschritts-Streaming |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` für BLE (DAQ-M)-Übertragung |

Überprüfen Sie das SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Das Python-SDK kommuniziert mit dem Backend, das `.deb` über eine lokale HTTP-API (`http://127.0.0.1:5000`) installiert. `ChlorosLocal` und die Verbindungshelfer starten es bei Bedarf automatisch; das `connect_all()` eines gespeicherten Projekts tut dies jedoch nie, daher sollten Sie auf einem Headless-Rechner das Backend als [Dienst](#always-on-ptp-for-headless-hosts) ausführenaus. Verwenden Sie immer die literale IPv4-Adresse anstelle von `localhost` – `localhost` kann zu `::1` aufgelöst werden und kostet etwa zwei Sekunden pro Anfrage.
{% endhint %}

***

## Ersteinrichtung

### 1. Anmelden

Für den Zugriff über CLI und SDK ist ein kostenpflichtiges Chloros+-Abo (**Copper** oder höher) erforderlich, das serverseitig durchgesetzt wird: Ein abgemeldeter Aufrufer erhält `401 AUTH_REQUIRED`, und ein Aufrufer mit kostenlosem Abo (Iron) erhält `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Anmeldedaten werden in `~/.chloros/user_session.json` zwischengespeichert.

{% hint style="info" %}
**Bei Upgrades bleibt Ihre Anmeldung erhalten; bei einer Deinstallation wird sie gelöscht.** Die Installation einer neueren Version von `.deb` über eine bestehende Installation (auch über `chloros-cli update`) behält `~/.chloros/user_session.json` und den zwischengespeicherten Lizenz-Cache bei, sodass Headless-CLI- und SDK-Jobs weiterhin funktionieren. Durch das Entfernen des Pakets (`sudo apt remove chloros`) werden die gespeicherten Anmeldedaten und der Lizenz-Cache für jeden Benutzer auf dem Rechner gelöscht, sodass für eine Neuinstallation erneut `chloros-cli login` erforderlich ist.
{% endhint %}

### 2. Überprüfen Sie Ihren Lizenzstatus

```bash
chloros-cli status
```

`chloros-cli status` funktioniert auf jeder Stufe (einschließlich der kostenlosen), sodass Sie jederzeit sehen können, warum der Zugriff verfügbar ist oder nicht.

### 3. Systemdiagnose ausführen

```bash
chloros-cli selftest
```

Es werden nacheinander sieben Prüfungen durchgeführt, und der Befehl beendet sich mit einem Ergebnis ungleich Null, wenn eine davon fehlschlägt:

| # | Prüfung | Was wird überprüft? |
| --- | --- | --- |
| 1 | **Version** | Die CLI gibt ihre Version an (`v1.3.0`). |
| 2 | **Port verfügbar** | Port 5000 ist frei *oder* wird bereits von einem funktionsfähigen Chloros-Backend beantwortet (was als bestanden gilt). |
| 3 | **Backend-Start** | Die Backend-Binärdatei wird gestartet. |
| 4 | **Test der API (`/api/test`)** | Das Backend antwortet mit `status: ok`. |
| 5 | **Systeminformationen** | Gibt `GPU: <name>, CUDA: <bool>, PyTorch: <version>` aus `/api/system-info` aus. |
| 6 | **Denoiser-Modelle** | Findet `*.pth.enc`-Modelle (unter Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Rauschunterdrückung**| „Texture Aware“ ist tatsächlich einsetzbar – benötigt CUDA**und** mindestens eine Modelldatei. |

Der Lauf endet mit `N/7 checks passed`, wobei alle Fehler namentlich aufgelistet werden.

### 4. Verarbeiten Sie Ihren ersten Datensatz

```bash
chloros-cli process ~/datasets/flight001
```

***

## Dateien und Verzeichnisse

### Benutzerbezogen

Chloros speichert seine Anmeldedaten und die CLI-Konfiguration in einem einzigen plattformübergreifenden Verzeichnis, **`~/.chloros/`** (unter Windows: `%USERPROFILE%\.chloros\`). Zwei Linux-spezifische Caches folgen stattdessen den XDG-Konventionen – diese verwenden bei Einrichtung die Namen `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`.

| Pfad | Zweck |
| --- | --- |
| `~/.chloros/user_session.json` | Anmeldesitzungs-Cache, geschrieben von `chloros-cli login` (bleibt bei Paket-Upgrades erhalten; wird gelöscht, wenn das Paket entfernt wird) |
| `~/.chloros/working_directory.txt` | Überschreibung des Standard-Projektordners (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI-Spracheinstellung (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Spracheinstellung, die mit der Windows-GUI geteilt wird – ein `language` hat hier Vorrang vor `cli_language.json` |
| `~/.chloros/update_cache.json` | Einstündiger Cache für die Überprüfung auf Updates beim Start von Linux/Jetson |
| `~/.chloros/backend.log` | Backend-Protokoll, wenn das Backend über die CLI gestartet wurde |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Zwischengespeicherte LATTICE-Kalibrierungspakete pro Kamera, sortiert nach Seriennummer und Bundle-Hash |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Optionale benutzerdefinierte Überschreibungen für DAQ-Cap-Korrekturprofile |
| `~/.config/chloros/system_config.json` | Zwischengespeichertes Hardwareprofil aus der dynamischen Rechenanpassung – löschen Sie es, um eine neue Hardwareerkennung zu erzwingen |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-Serverprotokolle, eine Datei pro Start |
| `~/Documents/MAPIR/Chloros Projects/` | Standard-Projektordner, wenn keine Überschreibung festgelegt ist |

### Systemweit

| Pfad | Zweck |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-Skript – legt `LD_LIBRARY_PATH` für die gebündelten nativen Bibliotheken fest (und `TMPDIR` auf `/mnt/ssd/tmp`, wenn dieser Ordner existiert), führt anschließend die eigentliche Binärdatei aus |
| `/usr/bin/chloros-backend` | Wrapper-Skript – dasselbe für das Backend, das es als installierten Produktions-Build markiert |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Die kompilierte CLI und das Backend, entpackt installiert (neu in 1.3.0); die Programme heißen `cli/chloros-cli` und `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Zweizeilige Launcher-Skripte, die die oben genannten eigentlichen Programme starten – sie werden an diesen Pfaden beibehalten, da das SDK, die CLI und ältere Skripte hier nach ihnen suchen |
| `/usr/lib/chloros/arena_runtime/` | Von LATTICE-Kameras benötigte Arena-SDK-Laufzeitumgebung |
| `/usr/lib/chloros/models/*.pth.enc` | Verschlüsselte Denoiser-Modelle, die vom „Texture Aware“-Debayer verwendet werden |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (sowie `lattice_sdk`, `lattice_cam`, `arena_api`, die Projektlader) | Das Python-SDK, das für das System `python3` installiert wurde und zum Paket gehört |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python-SDK-Wheel, das genau zu diesem Build passt, für eine virtualenv |
| `/usr/lib/chloros/exiftool` | Mitgelieferter ExifTool-Launcher (nur dann per Symlink mit `/usr/local/bin/exiftool` verknüpft, wenn kein systemeigenes ExifTool vorhanden ist). Ab Version 1.3.0 wird das ExifTool-Skript mit einer eigenen Perl-Bibliothek in `/usr/lib/chloros/exiftool-dist/` ausgeliefert, sodass es ohne das Systempaket `libimage-exiftool-perl` läuft |
| `/etc/chloros/update.conf` | Konfiguration des Update-Kanals, die von `chloros-cli update` gelesen wird |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Setzt `net.ipv4.ip_unprivileged_port_start = 319` so, dass das Backend die PTP-Ports ohne Root-Rechte binden kann |
| `/etc/sysctl.d/60-chloros-gige.conf` | Erhöht die Socket-Empfangspuffer des Kernels (`net.core.rmem_max`, `net.core.rmem_default`) für LATTICE-Kamerastreams auf 32 MB und wendet diese bei der Installation an (ein bereits höher eingestellter Wert bleibt unverändert). Bei der Kernel-Standardeinstellung werden Kamerapakete im Kernel verworfen, und ein Array verliert Frames und Bildrate; `chloros-cli lattice network` meldet die Puffer, wenn sie fast voll sind. Siehe [Linux-Hosts – Socket-Empfangspuffer](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Verweist den dynamischen Loader auf `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Gewährt dem angemeldeten Benutzer Zugriff auf die DAQ-U-USB-Seriellbrücke (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Opt-in-Dienst für den ständig aktiven Backend-Dienst, der das Backend als den von Ihnen angegebenen Benutzer ausführt (installiert, **nicht aktiviert**) – diesen Dienst müssen Sie aktivieren, siehe [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Derselbe Dienst, der das Backend als **root**ausführt (installiert,**nicht aktiviert**); wird beibehalten, damit ein Rechner, auf dem er aktiviert wurde, weiterläuft |
| `/var/log/chloros/` | Das detaillierte Protokoll des Root-Dienstes, lesbar für die Gruppe `adm` (wird beim Löschen des Pakets entfernt) |
| `/usr/share/applications/chloros-cli.desktop` | „Chloros CLI“-Eintrag im Anwendungsmenü, der ein Terminal öffnet |

**Firewall (neu in 1.3.0).** Wenn `ufw` oder `firewalld` zum Zeitpunkt der Installation aktiv ist, öffnet das Paket die UDP-Ports `5002` (DAQ-E-Daten), `5353` (mDNS-Erkennung), `319` und `320` (PTP) und entfernt diese Regeln, wenn das Paket deinstalliert wird. Ist eine Firewall installiert, aber nicht aktiv, nimmt das Paket keine Änderungen vor und gibt die zuzulassenden Ports aus, falls Sie die Firewall später aktivieren sollten. Es öffnet keinen TCP-Port: Ein DAQ-E-Firmware-Update, bei dem der Sensor über TCP eine Verbindung zu diesem Computer herstellt, um das Bild zu erfassen, schlägt mit dem Fehler „`… device never connected back on TCP <port> …`“ fehl, wenn eine aktive Firewall diese Verbindung blockiert.

## Speicherort der Backend-Ausführungsdatei

Die CLI und das SDK erkennen das Backend automatisch:

| Komponente | Pfad |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (ein Starter für `/usr/lib/chloros/backend/chloros-backend`) |

Überschreiben Sie den Backend-Pfad mit dem CLI-Flag „`--backend-exe`“ oder dem SDK-Konstruktorparameter „`backend_exe`“, und den Port mit `--port` (Standard: `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` verweist auf die Befehlsfamilien **`lattice`**,**`project`**und**`daq pool-*`** auf ein entferntes Backend. Die Kernbefehle (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignorieren diesen Wert bewusst und zielen immer auf `http://127.0.0.1:<port>` ab.
{% endhint %}

***

## LATTICE-Kameras und DAQ-Lichtsensoren unter Linux

Die Befehlsfamilien von „live-hardware“ funktionieren alle unter Linux (amd64 und Jetson):

* **`chloros-cli lattice`** – Erkennen, Verbinden, Konfigurieren und Aufzeichnen von LATTICE-Kameras und synchronisierten Arrays. Der Befehl `.deb` bündelt die dafür erforderliche Arena-SDK-Laufzeitumgebung und registriert sie beim dynamischen Loader.
* **`chloros-cli daq pool-*`** — Verbinden Sie DAQ-U/M/E-Lichtsensoren über den Backend-Pool, streamen Sie kalibrierte Spektren, zeichnen Sie `.daq`-Dateien auf und tauschen Sie CAP-Profile aus; bei einem DAQ-E können Sie zudem dessen Netzwerkeinstellungen ändern, das Gerät neu starten und die Firmware aktualisieren. `chloros-cli daq discover` sucht nach Sensoren, die Sie anschließen könnten, und `chloros-cli daq --help` listet die `pool-*`-Unterbefehle auf.
* **`chloros-cli project`** — ein gespeichertes Projekt (mit seinen Kameras, Sensoren und Verarbeitungseinstellungen) im Headless-Modus ausführen.
* **`chloros-cli time-sync`** — Überprüfen Sie den PTP-Grandmaster, den das Chloros-Backend für LATTICE-Kameras und DAQ-E-Sensoren betreibt.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` wird von `pool-latest` benötigt, `pool-stream`, `pool-record` und `pool-set-cap`; `pool-list` zeigt die derzeit im Pool befindlichen IDs an.

{% hint style="info" %}
**`--eth-host` überspringt die Erkennung.** `pool-connect --eth` findet einen DAQ-E über mDNS, mit einem ARP-Fallback, der auch bei einem leeren ARP-Cache funktioniert; durch Übergabe der IP-Adresse oder des Hostnamens des Sensors mit `--eth-host` wird eine direkte Verbindung hergestellt.
{% endhint %}

**Serielle Berechtigungen für DAQ-U** werden durch die installierte udev-Regel verwaltet (`uaccess` + Gruppe `dialout`). Sollte ein bereits angeschlossener Sensor weiterhin nicht erreichbar sein, laden Sie die Regeln neu oder schließen Sie ihn erneut an:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Den vollständigen Befehlssatz finden Sie in der [CLI-Referenz](../CLI.md).

### Ständig aktives PTP für Headless-Hosts

Das Paket installiert zwei systemd-Dienste für das Backend, die beide nicht aktiviert sind. Auf einem Jetson ohne Bildschirm oder einem Server, auf dem die PTP-Zeitsynchronisation für DAQ-E-Sensoren und LATTICE-Kameras kontinuierlich laufen soll, oder wenn SDK-Skripte gespeicherte Projekte verbinden (`connect_all()` startet niemals ein Backend), aktivieren Sie den Dienst **als Ihr Benutzer**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` führt das Backend als diesen Benutzer aus: Die von ihm geschriebenen Aufzeichnungen gehören diesem Benutzer, sein detailliertes Protokoll befindet sich in dessen `~/.cache/chloros/logs`, und er verwendet dessen `chloros-cli login` (`~/.chloros`) – melden Sie sich daher zunächst einmal als dieser Benutzer an. Bei der Installation des Pakets wird dieser Befehl mit Ihrem Benutzernamen ausgegeben.

Ohne einen Dienst läuft das Backend (und damit der PTP-Grandmaster) nur, solange es von etwas gestartet wurde – einem Befehl „`chloros-cli`“ oder einem SDK-Skript.

Die Einheit bindet das Backend an die Umgebungseinstellungen „`127.0.0.1:5000`“ (Umgebungseinstellungen „`CHLOROS_HOST`“ / „`CHLOROS_PORT`“ innerhalb der Unit; Überschreiben mit „`sudo systemctl edit chloros-backend@.service`“) und startet es bei einem Fehler nach 5 Sekunden neu.

**Der Root-Dienst.**`chloros-backend.service` führt das Backend als**root** aus. Es ist weiterhin installiert, damit ein Rechner, auf dem es aktiviert wurde, weiterläuft, aber alles, was ein Root-Backend schreibt, gehört root, sodass ein Benutzer seine eigenen Aufzeichnungen nicht löschen kann; sein Protokoll befindet sich in `/var/log/chloros` und ist für die Gruppe `adm` lesbar. Um zu Ihrem Benutzer zu wechseln:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Das Backend eines Dienstes wird gemeinsam genutzt und niemals von einem Client angehalten.** `chloros-cli` und SDK-Skripte nutzen es so, wie sie es vorfinden: `process` läuft darauf und lädt sein eigenes Projekt anstelle des letzten. Wo die CLI-Schnittstelle andernfalls das Backend neu starten würde (`--restart` oder ein `process`, dessen Import nichts gefunden hat), nennt sie die Einheit, gibt „`sudo systemctl restart <unit>`“ aus und beendet „`1`“, da ein auf diese Weise gestopptes Backend inaktiv bleiben würde. Starten Sie es selbst mit „`sudo systemctl restart chloros-backend@$USER.service`“ neu. Das Backend eines Dienstes stoppt sich niemals selbst im Leerlauf, und `chloros-cli backend stop` lehnt dies ab und nennt den Befehl `systemctl stop`.**Wie PTP seine Ports erhält.** PTP verwendet UDP 319/320, die beide unterhalb der normalen Grenze von 1024 privilegierten Ports liegen. Das Paket `postinst` schreibt `/etc/sysctl.d/60-chloros-ptp.conf` mit `net.ipv4.ip_unprivileged_port_start = 319`, wodurch das Backend diese Ports binden kann, während es als Ihr Benutzer läuft. Ab Version 1.3.0 setzt das Paket keine Dateiberechtigungen (`setcap`) für das Backend, und `libcap2-bin` ist keine Abhängigkeit mehr: Auf dem entpackten Backend würde eine Dateiberechtigung dessen Start verhindern, fügen Sie daher selbst keine hinzu.***

## Beispiele für Bash-Skripte

{% hint style="info" %}
**Skriptfreundliche Exit-Codes.**`chloros-cli process` beendet `0` bei Erfolg und**mit einem Wert ungleich Null bei einem Fehler – einschließlich eines Laufs, bei dem Bildprodukte angefordert, aber keine geschrieben wurden** (es gibt „`Processing finished but wrote no image products.`“ aus und nennt den Projektordner sowie die üblichen Ursachen). Erfolgreiche Durchläufe melden, wie viele Bildprodukte geschrieben wurden (`Image products written: N`, plus `(plus M clamp masks)`, wenn Masken geschrieben wurden). Exit-Codes: `0` – Erfolg, `1` – Fehler, `2` – Argumentfehler, `130` – unterbrochen.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Es gibt genau vier gültige `--format`-Werte, die Leerzeichen enthalten – setzen Sie diese daher immer in Anführungszeichen:

| `--format`-Wert | Ausgabeordner |
| --- | --- |
| `TIFF (16-bit)` *(Standard)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` akzeptiert `standard` (Standard) oder `texture-aware` (Chloros+).

### Automatisierte Verarbeitung mit Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Beispiel für das Python-SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend lässt sich nach Hinzufügen von Dateiberechtigungen nicht starten

Führen Sie `setcap` nicht auf dem Backend aus. Ab Version 1.3.0 legt das Paket keine Dateifähigkeiten fest, und die PTP-Ports 319/320 stammen aus der Einstellung `sysctl`, die unter [Wie PTP seine Ports erhält](#always-on-ptp-for-headless-hosts) beschrieben ist. Eine Dateifähigkeit versetzt das entpackte Backend in einen sicheren Ausführungsmodus, in dem es seine eigenen Bibliotheken nicht finden kann und daher nicht startet. Falls Sie eine hinzugefügt haben, entfernen Sie diese:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP startet nicht / kann Port 319 nicht binden

Vergewissern Sie sich, dass die Untergrenze für nicht privilegierte Ports gesenkt wurde, und wenden Sie diese gegebenenfalls erneut für den aktuellen Boot-Vorgang an:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Überprüfen Sie anschließend den Grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice`-Befehle schlagen mit der Meldung „LATTICE-Kameratreiber nicht gefunden“ fehl

Das Backend konnte die LATTICE-Kamera-Laufzeitumgebung, mit der es installiert wurde, nicht laden. Jeder `lattice`-Befehl wird auf dem Backend ausgeführt; die CLI selbst enthält keine Kameratreiber. Installieren Sie Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`) neu: Der Befehl `.deb` installiert die Laufzeitumgebung neben dem Backend (in `/usr/lib/chloros/arena_runtime`) und registriert sie beim dynamischen Lader. So überprüfen Sie diese Registrierung:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Die Backend-Protokolle zum fehlgeschlagenen Start befinden sich in `~/.cache/chloros/logs/`. Wenn Port 5000 zu einem Backend-Dienst gehört (`systemctl status chloros-backend@$USER.service`), starten Sie den Dienst neu, anstatt dessen Prozess zu beenden.

### CUDA nicht erkannt

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` meldet dasselbe in einer Zeile: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Fehlende gemeinsam genutzte Bibliotheken

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Langsamer Start auf SD-Karten-Systemen

Ab Version 1.3.0 werden das Backend und die CLI-Anwendung im entpackten Zustand installiert, sodass sie sich nicht mehr bei jedem Start in ein temporäres Verzeichnis entpacken. Temporäre Dateien werden automatisch in `/mnt/ssd/tmp` abgelegt, sofern dieser Ordner existiert; andernfalls können Sie sie auf ein schnelles Dateisystem verweisen:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Chloros-Update unter Linux

Der Befehl „`update`“ ist nur unter Linux/Jetson verfügbar. Er überprüft die Version, die im unter „`/etc/chloros/update.conf`“ konfigurierten Update-Kanal veröffentlicht wurde, und bietet an, die passende Version „`.deb`“ herunterzuladen und zu installieren:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Unter Linux und Jetson führt die CLI außerdem bei jedem Start eine nicht blockierende Aktualisierungsprüfung durch (das Ergebnis wird eine Stunde lang in `~/.chloros/update_cache.json` zwischengespeichert) und gibt `Update available: vX.Y.Z` aus, wenn eine neuere Version vorliegt. Ihre Einstellungen, Projekte und Anmeldedaten bleiben bei einem Update erhalten.

Bei der Aktualisierung des Pakets werden während der Installation alle Backend-Dienste angehalten, sodass ein laufendes Skript den Zugriff auf seine Kameras und Sensoren verliert und anschließend jeden aktivierten Dienst erneut startet (ein manuell gestarteter, aber nie aktivierter Dienst bleibt angehalten, wie nach einem Neustart).

## Deinstallation

```bash
sudo apt remove chloros
```

Die Deinstallation stoppt und deaktiviert alle Backend-Dienste, stellt die Standardgrenze für nicht privilegierte Ports (1024) wieder her, entfernt die Einstellungsdatei für den Empfangspuffer (die aktuellen Werte bleiben bis zum Neustart erhalten), den Symlink zum mitgelieferten „exiftool“, die Arena-Loader-Konfiguration, die vom Paket hinzugefügten Firewall-Regeln und das installierte Python-SDK sowie die zwischengespeicherten Anmeldedaten. Ihre Projekte und `~/.chloros/`-Datendateien bleiben unberührt. `sudo apt purge chloros` löscht außerdem `/var/log/chloros`.

***

## Nächste Schritte

* [NVIDIA Jetson-Handbuch](nvidia-jetson-guide.md) — Jetson-spezifische Optimierung und Bereitstellung
* [CLI: Befehlszeile](../CLI.md) — das CLI-Handbuch
* [API: Python-SDK](../api-python-sdk.md) — das SDK-Handbuch
* [CLI-Referenz](../reference/cli-reference.md) und [SDK-Referenz](../reference/sdk-reference.md) – umfassende Befehls- und API-Listen für Version 1.3.0
* [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md) – So passt sich Chloros an Ihre Hardware an
