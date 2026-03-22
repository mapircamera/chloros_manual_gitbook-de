# Installation von Linux

Chloros wird für Linux als `.deb`-Pakete bereitgestellt, die CLI und das Backend installieren. Python und SDK werden separat über pip installiert.

***

## Linux amd64 (x86_64)

### Systemanforderungen

| Anforderung | Minimum | Empfohlen |
| --- | --- | --- |
| **Distribution** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Prozessor** | x86_64 (Intel/AMD) | Intel Core i7 oder besser |
| **Arbeitsspeicher (RAM)** | 8 GB | 16 GB oder mehr |
| **Grafikkarte** | Keine (CPU-Verarbeitung) | NVIDIA-GPU mit 4 GB+ VRAM |
| **Speicherplatz** | 2 GB freier Speicherplatz | SSD mit 10 GB+ freiem Speicherplatz |
| **Python** | Python 3.7+ (für SDK) | Python 3.10+ |

### Installation

Laden Sie das `.deb`-Paket herunter und installieren Sie es:

```bash
sudo dpkg -i chloros-amd64.deb
```

Überprüfen Sie die Installation:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Systemanforderungen

| Anforderung | Minimum | Empfohlen |
| --- | --- | --- |
| **Plattform** | NVIDIA Jetson mit JetPack 6 | Jetson Orin NX 16 GB oder AGX Orin |
| **JetPack** | JetPack 6.x | Aktuelles JetPack 6 |
| **Arbeitsspeicher (RAM)** | 8 GB (gemeinsam genutzt von GPU/CPU) | 16 GB+ gemeinsam genutzt |
| **Speicherplatz** | 2 GB freier Speicherplatz | NVMe-SSD mit 10 GB+ freiem Speicherplatz |
| **Python** | Python 3.7+ (für SDK) | Python 3.10+ |

### Installation

Laden Sie das JetPack 6 `.deb`-Paket herunter und installieren Sie es:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Überprüfen Sie die Installation:

```bash
chloros-cli --version
```

Ausführliche Informationen zur Jetson-Einrichtung einschließlich Wärmemanagement und Einsatz vor Ort finden Sie im [NVIDIA Jetson-Handbuch](nvidia-jetson-guide.md).

***

## Python SDK Installation (Alle Linux)

Das Python SDK wird separat über pip installiert und funktioniert sowohl auf amd64 als auch auf arm64:

```bash
pip install chloros-sdk
```

Um die optionale Unterstützung für Progress-Streaming einzubinden:

```bash
pip install chloros-sdk[progress]
```

Überprüfen Sie das SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Das Paket `.deb` installiert das Backend Chloros und CLI. Das Python SDK ist ein separates pip-Paket, das über ein lokales HTTP API mit dem Backend kommuniziert.
{% endhint %}

***

## Konfigurationsverzeichnisse

Chloros auf Linux folgt der [XDG-Basisverzeichnisspezifikation](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Zweck | Linux Pfad | Windows Äquivalent |
| --- | --- | --- |
| **Konfiguration** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Daten / Projekte** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Anmeldedaten** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Speicherorte der Backend-Ausführungsdateien

Das Paket `.deb` installiert das Backend an einem Standard-Speicherort. CLI und SDK erkennen den Backend-Pfad automatisch:

| Installationsmethode | Backend-Pfad |
| --- | --- |
| `.deb`-Paket | `/usr/lib/chloros/chloros-backend` |
| Manuell / benutzerdefiniert | `/opt/mapir/chloros/backend/chloros-backend` |

Sie können den Backend-Pfad mit dem Flag `--backend-exe` oder dem Konstruktorparameter `backend_exe` überschreiben.

***

## Ersteinrichtung

### 1. Aktivieren Sie Ihre Lizenz

Für den Zugriff auf CLI und SDK ist eine Chloros+-Lizenz erforderlich:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Überprüfen Sie Ihren Lizenzstatus

```bash
chloros-cli status
```

### 3. Verarbeiten Sie Ihren ersten Datensatz

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Führen Sie eine Systemdiagnose durch

Stellen Sie sicher, dass Ihr System korrekt konfiguriert ist:

```bash
chloros-cli selftest
```

Dies führt 7 Diagnosetests durch, darunter Version, Backend-Start, API-Konnektivität und CUDA/GPU-Verfügbarkeit.

***

## Beispiele für Bash-Skripte

### Mehrere Datensätze verarbeiten

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Mit benutzerdefinierten Einstellungen verarbeiten

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automatisierte Verarbeitung mit Cron

Fügen Sie dies zu Ihrer crontab hinzu (`crontab -e`), um neue Datensätze automatisch zu verarbeiten:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Beispiel

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

Wenn `chloros-cli` nach der Installation des Pakets `.deb` nicht gefunden wird:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Zugriff verweigert

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
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

### CUDA nicht erkannt

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Fehlende gemeinsam genutzte Bibliotheken

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Aktualisierung von Chloros auf Linux

Verwenden Sie den integrierten Update-Befehl, um nach Updates zu suchen und diese zu installieren:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Nächste Schritte

* [NVIDIA Jetson-Handbuch](nvidia-jetson-guide.md) – Jetson-spezifische Optimierung und Bereitstellung
* [CLI : Befehlszeile](../CLI.md) — Vollständige CLI-Befehlsreferenz
* [API : Python SDK](../api-python-sdk.md) — Vollständige SDK-Referenz
* [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md) — Wie sich Chloros an Ihre Hardware anpasst
