# NVIDIA Jetson-Leitfaden

Chloros auf NVIDIA Jetson ermöglicht die Verarbeitung multispektraler Bilder am Edge – vor Ort, auf UAVs und in abgelegenen Anlagen. Chloros erkennt Ihr Jetson-Modell automatisch und optimiert die Verarbeitungsstrategie für Ihre Hardware.

***

## Unterstützte Jetson-Modelle

| Modell                | RAM            | Verarbeitungsstrategie                                   | Empfohlene Verwendung                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB gemeinsam genutzt | `GPU_PARALLEL` (4 Worker)                            | Maximale Leistung, große Datensätze                      |
| **Jetson Orin NX**   | 8–16 GB gemeinsam genutzt  | `GPU_PARALLEL` (3 Worker, 16 GB) / `GPU_SINGLE` (8 GB) | Primäre Empfehlung für den Einsatz in der Luft und im Feld |
| **Jetson Orin Nano** | 8 GB gemeinsam genutzt     | `GPU_SINGLE` (1 Worker)                               | Edge-Computing der Einstiegsklasse                                 |
| **Jetson Nano**      | 4–8 GB gemeinsam genutzt   | `GPU_SINGLE` (1 Worker)                               | Einstiegsmodell, begrenzte Speicherkapazität                          |

{% hint style="info" %}
**Ältere Jetson-Modelle** (TX2, TX1, Xavier NX) werden möglicherweise nicht unterstützt. Die Leistung hängt vom verfügbaren GPU-Speicher und den CUDA-Fähigkeiten ab.
{% endhint %}

***

## Anforderungen

* **JetPack 6.x** (neueste Version empfohlen)
* **NVIDIA CUDA** (in JetPack enthalten)
* **Chloros+-Lizenz** (erforderlich für den Zugriff auf CLI/SDK)

## Installation

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Allgemeine Informationen zur Installation von Linux finden Sie unter [Linux Installation](linux-installation.md).

***

## Dynamische Rechenanpassung auf Jetson

Chloros erkennt Ihr Jetson-Modell automatisch und wählt die optimale Verarbeitungsstrategie aus. **Eine manuelle Anpassung ist nicht erforderlich.**

### So funktioniert es

Beim Start erstellt Chloros ein Profil Ihres Systems:

1. **Erkennt das Jetson-Modell** über `/proc/device-tree/model`
2. **Liest den verfügbaren GPU-/gemeinsamen Speicher**

3.**Wählt eine Verarbeitungsstrategie** (`GPU_PARALLEL`, `GPU_SINGLE` oder `CPU_PARALLEL`)
4. **Legt die Anzahl der Worker, den Pipeline-Typ und die Speicherzuweisung** automatisch fest

### Verhalten pro Modell

| Jetson-Modell                | Strategie       | Worker | Pipeline                       | Parallelität |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (speichereffizient) | Serialisiert  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Serialisiert  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serialisiert  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (vollständiger GPU-Pfad)    | Parallel  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Parallel  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** ist der Sweet Spot für den Edge-Einsatz – es erhält die Strategie `GPU_PARALLEL` mit 3 parallelen Workern und liefert echte parallele GPU-Verarbeitung in einem kompakten Formfaktor.
{% endhint %}

Der entscheidende Unterschied zwischen den Plattformen ist der **Speicher**. Ein Jetson Nano mit 8 GB gemeinsam genutztem Speicher muss Bilder einzeln mit einem speichereffizienten Tiled-Ansatz verarbeiten, während ein Orin NX mit 16 GB mithilfe der Fused-Pipeline mit höherem Durchsatz drei Bilder gleichzeitig über die GPU ausführen kann.

Die vollständige Referenz zur Rechenanpassung finden Sie unter [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md).

***

## Wärmemanagement

Jetson-Geräte verfügen über begrenzte thermische Spielräume, insbesondere bei geschlossenen oder luftgestützten Einsätzen. Chloros umfasst eine automatische Temperaturüberwachung und Drosselung:

| Temperatur         | Maßnahme                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normalbetrieb – volle Verarbeitungsgeschwindigkeit          |
| **70 °C** (Warnung) | Batchgröße automatisch reduzieren                   |
| **80 °C** (Kritisch) | Aggressive Drosselung – geringere Parallelität         |
| **90 °C** (Herunterfahren) | GPU-Verarbeitung vollständig stoppen – Abkühlung erforderlich |

{% hint style="warning" %}
**Sorgen Sie für ausreichende Belüftung und Wärmeableitung** für einen dauerhaften Betrieb, insbesondere in geschlossenen Feldgehäusen oder luftgestützten Systemen. Durch thermische Drosselung wird der Verarbeitungsdurchsatz reduziert, um die Hardware zu schützen.
{% endhint %}

***

## Speicherverwaltung

Jetson-Geräte verwenden **Unified Memory** – GPU und CPU teilen sich denselben physischen RAM. Das bedeutet, dass der angegebene VRAM (z. B. 15,3 GB auf dem Orin NX 16 GB) kein dedizierter GPU-Speicher ist; er wird mit dem Betriebssystem und anderen Prozessen geteilt.

### Empfehlungen zum Swap-Speicher

Bei großen Datensätzen oder der Texture-Aware-Debayer-Verarbeitung empfiehlt Chloros möglicherweise die Einrichtung von Swap-Speicher:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Geschätzter Speicherbedarf pro Bild:**

* Standard-Debayer: ~10 MB pro Bild
* Textur-bewusster Debayer: ~15 MB pro Bild

Chloros berechnet den erforderlichen Speicher automatisch anhand der Größe Ihres Datensatzes und warnt Sie, wenn Swap empfohlen wird.

### OOM (Out of Memory)-Fallback

Wenn während der Verarbeitung ein Speicherengpass festgestellt wird:

1. Chloros reduziert automatisch die Anzahl der GPU-Worker
2. Wechselt von der `fused_gpu`- zur `tiled_gpu`-Pipeline (speichereffizienter)
3. Die Verarbeitung wird mit reduziertem Durchsatz fortgesetzt, anstatt abzustürzen

***

## Einsatz vor Ort

### Überlegungen zur Stromversorgung

| Jetson-Modell     | Typische Leistungsaufnahme | Anmerkungen                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 W              | USB-C oder Rundstecker    |
| Jetson Orin Nano | 7–15 W              | DC-Rundstecker          |
| Jetson Orin NX   | 10–25 W             | DC-Rundstecker          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD oder Rundstecker |

Planen Sie Ihr Strombudget für die Dauerverarbeitung – die höchste Leistungsaufnahme tritt während des GPU-intensiven Threads 3 (Verarbeitung) auf.

### Speicherempfehlungen

* **NVMe-SSD** wird für arm64-Bereitstellungen dringend empfohlen
* SD-Karten sind für die Verarbeitung zu langsam – verwenden Sie sie nur als Boot-Medium
* Planen Sie für die verarbeitete Ausgabe das 2- bis 3-fache der Größe Ihrer Rohbilddaten ein

### Headless-Betrieb über SSH

Chloros CLI ist ideal für Headless-Jetson-Bereitstellungen:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Automatisierte Verarbeitung mit systemd

Erstellen Sie einen systemd-Dienst für die automatisierte Verarbeitung:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Kombinieren Sie diesen mit einem systemd-Timer für die zeitgesteuerte Verarbeitung:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Beispiel-Workflows

### Grundlegende Jetson-Verarbeitung

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK auf Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Stapelverarbeitung mehrerer Flüge

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Empfohlene Jetson-Systeme für den Feldeinsatz

Für den Einsatz vor Ort und in der Luft sollten Sie diese Jetson Orin NX 16 GB Carrier-Board-Optionen in Betracht ziehen:

* **Luftfahrt/Drohnen**: Systeme mit Vibrationsfestigkeit (MIL-STD), geringem Gewicht (unter 300 g) und passiver Kühlung
* **Robuster Feldeinsatz**: Wasserdichte Gehäuse nach IP67/IP69K mit PoE-GigE-Kameraanschluss
* **Minimal/Budget**: Entwickler-Kits mit Zusatzgehäusen

Wenden Sie sich an den [MAPIR-Support](https://www.mapir.camera/community/contact), um spezifische Hardware-Empfehlungen für Ihr Einsatzszenario zu erhalten.

***

## Nächste Schritte

* [Linux Installation](linux-installation.md) — Allgemeine Details zur Linux-Installation
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — Vollständige Referenz zur Rechenstrategie
* [Verarbeitungspipeline](../processing-architecture/processing-pipeline.md) – Die 4-Thread-Pipeline verstehen
* [CLI : Befehlszeile](../CLI.md) – Vollständige CLI-Referenz
* [API : Python SDK](../api-python-sdk.md) — Vollständige SDK-Referenz
