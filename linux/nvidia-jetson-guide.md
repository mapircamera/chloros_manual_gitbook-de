# NVIDIA Jetson-Handbuch

Chloros auf NVIDIA Jetson ermöglicht die multispektrale Bildverarbeitung am Netzwerkrand – im Feld, auf Drohnen und in abgelegenen Anlagen. Chloros 1.2.0 erkennt Ihr Jetson-Modell beim Start und optimiert seine Verarbeitungsstrategie für die erkannte Hardware. **Eine manuelle Feinabstimmung ist nicht erforderlich.**

***

## Unterstützte Jetson-Modelle

| Modell                | RAM            | Verarbeitungsstrategie                                     | Empfohlene Verwendung                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB gemeinsam genutzt | `GPU_PARALLEL` (2 Worker)                              | Maximale Leistung, große Datensätze                      |
| **Jetson Orin NX**   | 8–16 GB gemeinsam genutzt  | `GPU_PARALLEL` (2 Worker, 16 GB) / `GPU_SINGLE` (8 GB)   | Erste Wahl für den Einsatz in Fluggeräten und im Feld                        |
| **Jetson Orin Nano** | 8 GB gemeinsam genutzt     | `GPU_SINGLE` (1 Worker, sequenziell)                     | Edge-Computing der Einstiegsklasse                                 |

{% hint style="info" %}
Das arm64-Paket Linux erfordert **JetPack 6**, das für die Jetson Orin-Familie verfügbar ist. Ältere Modelle (Nano, TX2, Xavier NX) können JetPack 6 nicht ausführen und werden vom aktuellen Paket nicht unterstützt.
{% endhint %}

***

## Voraussetzungen

* **JetPack 6.x** (neueste Version empfohlen)
* **NVIDIA CUDA** (in JetPack enthalten)
* **Kostenpflichtiger Chloros+-Tarif** – Stufe „Copper“ oder höher (erforderlich für den Zugriff auf alle CLI-/SDK-Funktionen; wird serverseitig durchgesetzt)

## Installation

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Allgemeine Informationen zur Installation von Linux, zu Dateipfaden und zur Fehlerbehebung finden Sie unter [Linux-Installation](linux-installation.md).

{% hint style="info" %}
**Legen Sie das Entpackungsverzeichnis auf einem schnellen Speichermedium an.** Die kompilierten Binärdateien entpacken sich bei jedem Start in ein temporäres Verzeichnis – was von einer SD-Karte aus äußerst langsam ist. Chloros verwendet automatisch `/mnt/ssd/tmp`, sofern dieses vorhanden ist; andernfalls legen Sie für `TMPDIR` einen Pfad auf Ihrem NVMe fest (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamische Rechenanpassung auf Jetson

### So funktioniert es

Beim Start erstellt Chloros ein Profil Ihres Systems:

1. **Erkennt das Jetson-Modell** über `/proc/device-tree/model`
2. **Liest den verfügbaren gemeinsam genutzten GPU-/CPU-Speicher aus** (Jetson verwendet Unified Memory)
3. **Wählt eine Verarbeitungsstrategie aus** (`GPU_PARALLEL`, `GPU_SINGLE` oder `CPU_PARALLEL`)
4. **Legt die Anzahl der Worker, den Pipeline-Typ und die Speicherzuweisung** automatisch fest

Die Entscheidung richtet sich nach dem **gesamten gemeinsam genutzten RAM**, nicht nach dem Modellnamen:

* **Bei weniger als 12 GB Gesamt-RAM**(alle Jetsons mit 8 GB): `GPU_SINGLE` mit**1 Worker – gezielte sequenzielle Verarbeitung**. Der Speicher ist für parallele Worker zu knapp, daher werden die Bilder nacheinander verarbeitet. Auf Jetsons mit**8 GB oder weniger** umgeht Thread 3 den Worker-Pool vollständig und führt die pro Bild anfallenden Aufgaben innerhalb des Prozesses aus.
* **12 GB oder mehr**(Orin NX 16 GB, AGX Orin): Der einheitliche Speicher erfüllt die Voraussetzungen für `GPU_PARALLEL`, allerdings ist die Anzahl der Worker**auf Jetson auf 2 begrenzt** – die GPU, der RAM der Worker-Prozesse und ihre CUDA-Kontexte pro Worker greifen alle auf denselben gemeinsamen Pool zu, sodass bei mehr Workern das Risiko von Speicherfehlern besteht.

Sie können die automatische Auswahl mit der Umgebungsvariablen `CHLOROS_STRATEGY` überschreiben – siehe [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Verhalten pro Modell

| Jetson-Modell                | Strategie       | Worker | Ausführung                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sequenzielle In-Process-Schleife (`tiled_gpu` bei Speicherengpässen) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sequenzielle In-Process-Schleife                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Parallele Worker-Prozesse, `fused_gpu`-Pfad  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Parallele Worker-Prozesse, `fused_gpu`-Pfad  |

Der wesentliche Unterschied zwischen den Plattformen liegt im **Arbeitsspeicher**. Ein Jetson mit 8 GB muss Bilder unter hoher Auslastung einzeln mit einem speichereffizienten, kachelbasierten Ansatz verarbeiten, während ein Orin mit 16 GB oder mehr mithilfe der Fused-Pipeline mit höherem Durchsatz zwei Bilder gleichzeitig über die GPU verarbeiten kann.

### GPU-Budget pro Modell

Jedes Jetson-Modell verfügt zudem über ein Hardwareprofil, das festlegt, wie viel vom gemeinsamen Pool die Verarbeitung beanspruchen darf, und die Batch-Größen skaliert:

| Modell | Obergrenze des GPU-Budgets | Batch-Größen-Multiplikator | Für System/Anzeige reserviert |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Der erkannte Arbeitsspeicher passt das Profil an: Bei einem Jetson, der **16 GB oder mehr** meldet, wird der Batch-Multiplikator auf ×1,2 erhöht. Die Basis-Batchgröße vor Anwendung der Multiplikatoren beträgt 8 Bilder.

Die vollständige Referenz zur Rechenanpassung finden Sie unter [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md).

***

## GPU-Frequenzbegrenzung für „Texture Aware“ auf Nano und Orin Nano

Der „Texture Aware“-Debayer führt eine Inferenz im neuronalen Netzwerk auf der GPU durch, was bei Jetson-Modellen mit geringem Stromverbrauch (10–15-W-Klasse) bei voller GPU-Taktfrequenz **Überstromwarnungen**auslösen kann. Vor der „Texture Aware“-Verarbeitung auf einem**Jetson Nano oder Orin Nano**überprüft Chloros die maximale Frequenz der GPU und begrenzt diese auf**510 MHz** (510000000), falls diese derzeit höher ist:

* Wenn der Befehl CLI den sysfs-Knoten für die GPU-Frequenz beschreiben kann, wird die Begrenzung **automatisch angewendet** und eine Bestätigung ausgegeben.
* Ist dies nicht der Fall (Root-Rechte erforderlich), gibt CLI den genauen Befehl `sudo` aus, um die Begrenzung manuell anzuwenden, wartet einen Moment, damit Sie ihn lesen können, und fährt dann fort – die Verarbeitung läuft weiter, es können jedoch Überstromwarnungen angezeigt werden.

So wenden Sie die Begrenzung vor der Verarbeitung selbst an:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modelle mit höherer Leistung (Orin NX 25 W, AGX Orin 60 W) laufen mit voller GPU-Geschwindigkeit; es wird keine Begrenzung angewendet. Der Standard-Debayer löst bei keinem Modell die Begrenzung aus.

{% hint style="info" %}
**„Texture Aware“ auf Jetson verarbeitet immer jeweils nur ein Bild.** Jeder Worker würde einen eigenen CUDA-Kontext (~1 GB) sowie eine eigene Kopie des Denoiser-Modells benötigen, was der einheitliche Speicher nicht leisten kann – daher ist der „Texture Aware“-Pfad auf Jetson an einen einzelnen Worker gebunden, wobei der GPU-Zugriff serialisiert wird. Es ist zu erwarten, dass „Texture Aware“ auf jedem Jetson deutlich langsamer ist als „Standard“.
{% endhint %}

***

## Wärmemanagement

Jetson-Geräte verfügen über begrenzten thermischen Spielraum, insbesondere bei Einsätzen in geschlossenen Räumen oder in Flugzeugen. Chloros überwacht die SoC-Temperatur und drosselt die Batch-Größen automatisch:

| Temperatur         | Maßnahme                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normalbetrieb – volle Verarbeitungsgeschwindigkeit          |
| **70 °C** (Warnung)  | Die Batchgröße wird schrittweise reduziert (100 % → 50 % zwischen 70 °C und 80 °C) |
| **80 °C** (kritisch) | Aggressive Drosselung (50 % → 0 % zwischen 80 °C und 90 °C) |
| **90 °C** (Herunterfahren) | GPU-Verarbeitung vollständig stoppen – Abkühlung erforderlich |

{% hint style="warning" %}
**Sorgen Sie für ausreichende Belüftung und Wärmeableitung**, um einen dauerhaften Betrieb zu gewährleisten, insbesondere in geschlossenen Feldgehäusen oder luftgestützten Systemen. Durch thermische Drosselung wird der Rechendurchsatz reduziert, um die Hardware zu schützen.
{% endhint %}

***

## Speicherverwaltung

Jetson-Geräte verwenden **Unified Memory** – GPU und CPU teilen sich denselben physischen RAM. Der angegebene VRAM (z. B. ~15,3 GB bei einem Orin NX mit 16 GB) ist kein dedizierter GPU-Speicher; es handelt sich um denselben RAM, den das Betriebssystem und alle anderen Prozesse nutzen.

### Swap-Warnung und Empfehlungen

Vor der Verarbeitung auf dem Jetson zählt das Skript „CLI“ die RAW-Bilder in Ihrem Eingabeordner (`.tif`, `.tiff`, `.raw`, `.dng` – JPG-Vorschauen werden nicht gezählt), schätzt den maximalen Speicherbedarf des Laufs und **warnt vor dem Start**, wenn RAM + Swap-Speicher voraussichtlich nicht ausreichen. Die Warnung trägt die Überschrift „`LOW MEMORY WARNING - Jetson Detected`“, gibt die Anzahl Ihrer Bilder, den Arbeitsspeicher, den aktuellen Auslagerungsspeicher und den geschätzten Spitzenbedarf an und liefert anschließend die genauen Befehle „`fallocate`“ / „`chmod`“ / `mkswap` / `swapon`-Befehle an, deren Größe auf Ihr Projekt abgestimmt ist (niemals kleiner als 8 GB). Es wird eine Pause von einigen Sekunden eingelegt, damit die Meldung nicht im Scrollback verloren geht, dann wird die Verarbeitung fortgesetzt.**Von der Warnung verwendete Speicherschätzungen:**

| Debayer-Modus | Basis | Pro Bild |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (Modell + Python-Laufzeit) | ~15 MB |

Die Warnung wird ausgelöst, wenn der geschätzte Spitzenwert den Wert aus „RAM + Swap“ abzüglich einer Sicherheitsmarge von 1 GB überschreitet, wobei nur **dateibasierter** Swap berücksichtigt wird – eine reine zram-Konfiguration wird dennoch gemeldet.

So fügen Sie Auslagerungsspeicher manuell hinzu (Beispiel: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```### OOM-Behandlung (Out of Memory)

Während der Verarbeitung überwacht Chloros den GPU-Speicher und schaltet nahtlos auf einen reduzierten Modus um, anstatt abzustürzen:

1. Wenn die GPU-Speicherauslastung **85 %** überschreitet, werden die Batch-Größen präventiv reduziert
2. Tritt dennoch ein Out-of-Memory-Ereignis auf, wird die Batch-Größe **halbiert** und bei jedem weiteren OOM erneut halbiert; jeder nachfolgende erfolgreiche Batch hebt diese Einschränkung um einen Schritt auf
3. Bei anhaltender Belastung wechselt die Pipeline von `fused_gpu` auf den speichereffizienten `tiled_gpu`-Pfad und als letztes Mittel auf die CPU-Verarbeitung

***

## Einsatz vor Ort

### Überlegungen zur Leistungsaufnahme

| Jetson-Modell     | Typische Leistungsaufnahme | Anmerkungen                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC-Barrel-Stecker          |
| Jetson Orin NX   | 10–25 W             | DC-Barrel-Stecker          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD oder Rundstecker |

Planen Sie Ihr Strombudget für den Dauerbetrieb – die maximale Leistungsaufnahme tritt während des GPU-intensiven Threads 3 (Verarbeitung) auf.

### Speicherempfehlungen

* **NVMe-SSD** wird für arm64-Bereitstellungen dringend empfohlen
* SD-Karten sind für die Verarbeitung zu langsam – verwenden Sie sie nur als Boot-Medium
* Planen Sie das 2- bis 3-fache der Größe Ihrer Rohbilddaten für die verarbeiteten Ergebnisse ein

### Headless-Betrieb über SSH

Chloros und CLI eignen sich ideal für Headless-Jetson-Bereitstellungen:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Ständig aktives Backend für die Zeitsynchronisation von LATTICE / DAQ-E

Wenn Ihr Jetson LATTICE-Kameras oder DAQ-E-Lichtsensoren im Headless-Modus steuert, aktivieren Sie den systemd-Dienst des Backends, damit der PTP-Grandmaster kontinuierlich läuft (die Komponente ist installiert, aber standardmäßig nicht aktiviert):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Weitere Informationen, einschließlich der Beschreibung, wie das Paket die PTP-Ports 319/320 ohne Root-Rechte bindbar macht, finden Sie unter [Linux Installation](linux-installation.md#always-on-ptp-for-headless-hosts).

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

`chloros-cli process` beendet sich mit einem Wert ungleich Null, wenn ein Lauf, der Produkte angefordert hat, keine Bilder schreibt; daher ist der Fehlerstatus von systemd für die Überwachung aussagekräftig.

Kombinieren Sie dies mit einem systemd-Timer für die zeitgesteuerte Verarbeitung:

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
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

### Python und SDK auf Jetson

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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Empfohlene Jetson-Systeme für den Feldeinsatz

Für den Einsatz im Feld und in der Luft sollten Sie folgende Jetson Orin NX 16 GB-Trägerplatinen in Betracht ziehen:

* **Luftfahrt/Drohnen**: Systeme mit Vibrationsfestigkeit (MIL-STD), geringem Gewicht (unter 300 g) und passiver Kühlung
* **Robuster Feldeinsatz**: Wasserdichte Gehäuse nach IP67/IP69K mit PoE-GigE-Kameraanschluss
* **Minimal/kostengünstig**: Entwickler-Kits mit aufsteckbaren Gehäusen

Wenden Sie sich an den [MAPIR-Support](https://www.mapir.camera/community/contact), um spezifische Hardware-Empfehlungen für Ihr Einsatzszenario zu erhalten.

***

## Nächste Schritte

* [Linux-Installation](linux-installation.md) – Allgemeine Details zur Linux-Installation
* [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md) – Vollständige Referenz zur Rechenstrategie
* [Verarbeitungspipeline](../processing-architecture/processing-pipeline.md) – Die 4-Thread-Pipeline verstehen
* [CLI: Befehlszeile](../CLI.md) – Das CLI-Handbuch
* [API: Python SDK](../api-python-sdk.md) – Das SDK-Handbuch
* [CLI-Referenz](../reference/cli-reference.md) und [SDK-Referenz](../reference/sdk-reference.md) — Vollständige Befehls-/API-Listen für 1.2.0
