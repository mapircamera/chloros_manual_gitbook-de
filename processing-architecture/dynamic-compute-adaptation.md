# Dynamische Anpassung der Rechenleistung

Chloros 1.1.0 bietet eine intelligente Hardwareerkennung und die automatische Auswahl der Verarbeitungsstrategie. Die Verarbeitungs-Engine passt sich Ihrer Hardware an – vom Jetson Nano bis zur Workstation mit mehreren GPUs – ohne dass eine manuelle Konfiguration erforderlich ist.

***

## So funktioniert es

Beim Start von Chloros wird Ihr System automatisch profiliert:

1. **Erkennt das Betriebssystem** – Windows oder Linux
2. **Ermittelt CPU-Kerne und Gesamtspeicher**

3.**Erkennt vorhandene GPUs** – NVIDIA CUDA-Fähigkeit, VRAM, Modell
4. **Ermittelt das Jetson-Modell** (falls zutreffend) – über `/proc/device-tree/model`
5. **Überprüft Temperatursensoren** (Jetson) – für temperaturbewusste Verarbeitung
6. **Wählt die optimale Rechenstrategie aus** — basierend auf der gesamten erkannten Hardware
7. **Konfiguriert die Anzahl der Worker, den Pipeline-Typ und die Speicherzuweisung** automatisch

Das Ergebnis wird zwischengespeichert, sodass nachfolgende Durchläufe schneller starten. Bei Hardwareänderungen (z. B. Hinzufügen einer GPU) führt Chloros beim nächsten Start eine Neuprofilierung durch.

***

## Rechenstrategien

Chloros wählt je nach Ihrer Hardware eine von drei Rechenstrategien aus:

| Strategie | GPU erforderlich | Worker | Pipeline | Am besten geeignet für |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Ja (12 GB+ VRAM oder 16 GB+ gemeinsam genutzt) | 3–4 | `fused_gpu` | Desktop-GPUs mit 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Ja (&lt; 12 GB VRAM) | 1–3 | `tiled_gpu` | Einsteiger-GPUs, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Nein | Kerne – 1 | `cpu_fallback` | Systeme ohne NVIDIA-GPU |

### Pipeline-Typen

* **`fused_gpu`** — Vollständiger GPU-Verarbeitungspfad. Alle Debayer-, Korrektur- und Indexoperationen werden auf der GPU in einem einzigen fusionierten Durchlauf ausgeführt. Höchster Durchsatz, erfordert jedoch mehr VRAM.
* **`tiled_gpu`** — Speichereffizienter GPU-Pfad. Verarbeitet Bilder in Kacheln, um sie an den begrenzten GPU-Speicher anzupassen. Geringerer Durchsatz, funktioniert jedoch auf Geräten mit begrenztem Speicher.
* **`cpu_fallback`** — Reine CPU-Verarbeitung unter Verwendung von Multithread-Parallelität. Wird verwendet, wenn keine NVIDIA-GPU verfügbar ist.***

## Plattformspezifisches Verhalten

| Plattform | Strategie | Worker | Pipeline | Anmerkungen |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (seriell) | Speichereffizienter Modus, verarbeitet jeweils ein Bild |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (parallel) | Empfohlenes Edge-Gerät – echte parallele GPU-Verarbeitung |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (parallel) | Maximale Edge-Leistung |
| **Desktop mit 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Gute Desktop-Leistung mit speichereffizienten Kacheln |
| **Desktop mit 12 GB+ GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimale Desktop-Leistung |
| **Reines CPU-System** | `CPU_PARALLEL` | Kerne – 1 | `cpu_fallback` | Keine GPU erforderlich, nutzt ThreadPool |

{% hint style="info" %}
**Jetson Unified Memory**: Jetson-Geräte teilen sich GPU- und CPU-Speicher. Ein Jetson Orin NX 16 GB meldet ~15,3 GB VRAM, jedoch handelt es sich hierbei um denselben physischen RAM, der vom Betriebssystem und den CPU-Prozessen genutzt wird. Chloros berücksichtigt dies bei der Festlegung von Schwellenwerten für die Speicherzuweisung.
{% endhint %}

***

## Dynamische GPU-Speicherzuweisung

Chloros verwendet eine [4-Thread-Verarbeitungspipeline](processing-pipeline.md):

* **Thread 1** (Erkennung) — Bildladen, EXIF-Analyse, Zielerkennung
* **Thread 2** (Kalibrierung) — Berechnung der Reflexionskalibrierung
* **Thread 3** (Verarbeitung) – GPU-Debayering, Vignettierungskorrektur, Indexberechnung
* **Thread 4** (Export) – Dateischreiben, Einbetten von Metadaten

Sobald frühere Pipeline-Threads ihre Arbeit abgeschlossen haben (z. B. wenn alle Bilder erkannt wurden), wird ihre GPU-Speicherzuweisung freigegeben und **auf die verbleibenden aktiven Threads umverteilt**. Das bedeutet, dass Thread 3 (die GPU-intensive Phase) im Verlauf der Pipeline zunehmend mehr Speicher erhält, wodurch der Durchsatz für die rechenintensivsten Aufgaben verbessert wird.

### Zuweisungsphasen

| Phase | Aktive Threads | GPU-Speicherverteilung |
| --- | --- | --- |
| **Früh** | 1, 2, 3, 4 | Auf alle Threads verteilt |
| **Frühe Mitte** | 2, 3, 4 | Speicher von Thread 1 neu verteilt |
| **Späte Mitte** | 3, 4 | Speicher von Threads 1+2 geht an 3+4 |
| **Spät** | 3 oder 4 | Maximaler Speicher für verbleibenden Thread |***

## Texture Aware-Verarbeitung

Die Texture Aware-Debayer-Methode (nur Chloros+) verbraucht aufgrund des AI/ML-Rauschunterdrückungsmodells deutlich mehr GPU-Speicher als die Standardmethode:

* Systeme mit **&lt; 7 GB VRAM** werden im Texture Aware-Modus in eine synchrone Verarbeitungsschleife gezwungen (jeweils ein Bild)
* Systeme mit **7 GB+ VRAM** können Texture Aware parallel verarbeiten, allerdings mit einer im Vergleich zum Standard reduzierten Anzahl an Workern***

## Wärmemanagement (Jetson)

Jetson-Geräte unterliegen thermischen Einschränkungen, insbesondere bei geschlossenen oder luftgestützten Einsätzen. Chloros überwacht die GPU- und CPU-Temperaturen und passt die Verarbeitung automatisch an:

| Temperatur | Reaktion |
| --- | --- |
| **&lt; 70 °C** | Normalbetrieb – volle Geschwindigkeit |
| **70 °C** (Warnung) | Batchgröße reduzieren |
| **80 °C** (Kritisch) | Aggressive Drosselung – geringere Parallelität und Worker-Anzahl |
| **90 °C** (Herunterfahren) | GPU-Verarbeitung vollständig stoppen |

Die Temperaturüberwachung erfolgt auf Jetson-Plattformen über `tegrastats`. Auf Desktop-Systemen mit ausreichender Kühlung wird die thermische Drosselung selten ausgelöst.

***

## Umgang mit Speicherauslastung

Chloros überwacht den Speicherbedarf des Systems während der Verarbeitung:

* **Speicherschwelle**: Eine Auslastung von 85 % löst ein konservatives Verhalten aus
* **OOM-Reduzierung**: Tritt ein Out-of-Memory-Ereignis auf, wird die Zuweisung um 25 % reduziert (Multiplikator 0,75)
* **Pipeline-Fallback**: Bei starkem Speicherengpass wechselt die Pipeline automatisch von `fused_gpu` zu `tiled_gpu`
* **Swap-Empfehlungen**: Auf Jetson warnt Chloros Sie, wenn der Swap-Speicher für die Größe Ihres Datensatzes nicht ausreicht***

## Überwachung der Rechenanpassung

### Statusausgabe von CLI

Zu Beginn der Verarbeitung zeigt CLI das erkannte Hardwareprofil an:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Systemdiagnose

Führen Sie `chloros-cli selftest` aus, um ein vollständiges Hardwareprofil anzuzeigen und die Rechenkapazitäten zu überprüfen:

```bash
chloros-cli selftest
```

Dadurch werden die CUDA-Verfügbarkeit, der GPU-Speicher, die Denoiser-Modelle und die Backend-Konnektivität überprüft.

***

## Nächste Schritte

* [Verarbeitungspipeline](processing-pipeline.md) – Die 4-Thread-Pipeline-Architektur verstehen
* [NVIDIA Jetson-Leitfaden](../linux/nvidia-jetson-guide.md) – Jetson-spezifische Bereitstellung und Optimierung
* [CLI : Befehlszeile](../CLI.md) – Vollständige CLI-Referenz
