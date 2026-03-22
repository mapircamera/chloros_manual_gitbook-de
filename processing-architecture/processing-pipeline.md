# Verarbeitungspipeline

Chloros 1.1.0 nutzt eine 4-Thread-Verarbeitungspipeline, die wie ein stufenweiser Fließbandprozess funktioniert. Jeder Thread übernimmt eine bestimmte Phase des Verarbeitungsablaufs, sodass mehrere Bilder gleichzeitig in verschiedenen Phasen verarbeitet werden können.

***

## Pipeline-Architektur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Jedes Bild durchläuft nacheinander alle vier Threads. Mit der Multithread-Verarbeitung von Chloros+ können sich mehrere Bilder gleichzeitig in verschiedenen Threads befinden – während Thread 3 ein Bild verarbeitet, kann Thread 1 das nächste erkennen, Thread 2 ein weiteres kalibrieren und Thread 4 ein zuvor verarbeitetes Bild auf die Festplatte schreiben.

***

## Thread-Details

### Thread 1: Erkennung

**Zweck**: Laden von Bildern und Erkennen von Kalibrierungszielen.

* Liest Bilddateien von der Festplatte (RAW, JPG)
* Extrahiert EXIF-Metadaten (GPS, Kameramodell, Zeitstempel, Belichtung)
* Erkennt ArUco-Kalibrierungsziele in markierten Zielbildern
* Ausgabe: Bilddaten + Metadaten + Ergebnisse der Zielerkennung

Dies ist in erster Linie ein I/O- und CPU-gebundener Thread.

### Thread 2: Kalibrierung

**Zweck**: Berechnung der Kalibrierungsparameter aus den erkannten Zielen.

* Berechnet Reflexionskalibrierungskoeffizienten aus Zielbildern
* Berechnet Vignettierungskorrekturparameter
* Ermittelt bandweise Kalibrierungskurven
* Ausgabe: Kalibrierungsparameter für jedes Bild

Dies ist ein CPU-gebundener Rechen-Thread.

### Thread 3: Verarbeitung (GPU)

**Zweck**: Korrekturen anwenden und Vegetationsindizes berechnen.**Dies ist der rechenintensivste Thread.*** **Debayering**: Konvertiert RAW-Daten im Bayer-Muster in Mehrkanalbilder
  * Standard (schnell, mittlere Qualität) – Standard
  * Texturbewusst (langsam, höchste Qualität) – nur Chloros+, verwendet AI/ML-Rauschunterdrückung
* **Vignettierungskorrektur**: Wendet eine Objektiv-Vignettierungskorrektur auf das gesamte Bild an
* **Reflexionskalibrierung**: Wendet Kalibrierungskoeffizienten an, um in Reflexionswerte umzuwandeln
* **Indexberechnung**: Berechnet Vegetationsindizes (NDVI, NDRE, GNDVI usw.)
* Ausgabe: verarbeitete Bilddaten, bereit für den Export

Dieser Thread profitiert am meisten von der GPU-Beschleunigung. Das System [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optimiert in erster Linie das Verhalten dieses Threads.

### Thread 4: Export

**Zweck**: Schreiben der verarbeiteten Bilder auf die Festplatte.

* Schreibt Ausgabedateien im ausgewählten Format (TIFF 16-Bit, TIFF 32-Bit %, PNG, JPG)
* Betten von EXIF-Metadaten in Ausgabedateien ein (GPS, Zeitstempel, Verarbeitungsparameter)
* Organisieren der Ausgabe in Unterordner nach Kameramodellen
* Ausgabe: endgültige Dateien auf der Festplatte

Dies ist in erster Linie ein I/O-gebundener Thread. SSD-Speicher verbessert die Leistung von Thread 4 erheblich.

***

## Sequenzielle vs. Pipeline-Verarbeitung

### Freier Modus (Sequenziell)

In der kostenlosen Version von Chloros werden Bilder **einzeln** nacheinander durch alle vier Stufen verarbeitet:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Der Fortschrittsbalken in der Benutzeroberfläche zeigt zwei Stufen an: Zielerkennung und Verarbeitung.

### Chloros+-Modus (Pipelined)

Mit einer Chloros+-Lizenz arbeiten alle vier Threads **gleichzeitig** an verschiedenen Bildern:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Der Fortschrittsbalken in der Benutzeroberfläche zeigt 4 Phasen an: Erkennung, Analyse, Kalibrierung, Export. Bewegen Sie den Mauszeiger über den Fortschrittsbalken, um den Fortschritt pro Thread anzuzeigen.

{% hint style="success" %}
**Die Pipeline-Verarbeitung mit Chloros+** kann je nach Hardware und Datensatzgröße 3- bis 5-mal schneller sein als die sequenzielle Verarbeitung. Die Beschleunigung ist auf Systemen mit schnellen GPUs und SSDs am größten.
{% endhint %}

***

## Fortschritt des Export-Threads 4

In Chloros 1.1.0 verfügt der Export-Thread (Thread 4) über eine eigene Fortschrittsanzeige. Sie können den Exportfortschritt separat überwachen:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Die Verarbeitung ist abgeschlossen, wenn Thread 4 100 % erreicht.

***

## Zusammenhang mit der dynamischen Rechenanpassung

Das System der [Dynamischen Rechenanpassung](dynamic-compute-adaptation.md) wirkt sich in erster Linie auf **Thread 3 (Verarbeitung)** aus:

* **`GPU_PARALLEL`**-Strategie: Thread 3 verarbeitet mehrere Bilder gleichzeitig über die GPU unter Verwendung der `fused_gpu`-Pipeline
* **`GPU_SINGLE`**-Strategie: Thread 3 verarbeitet jeweils ein Bild mithilfe der speichereffizienten `tiled_gpu`-Pipeline
* **`CPU_PARALLEL`**-Strategie: Thread 3 nutzt CPU-basierte Verarbeitung mit Multithread-Parallelität

Die GPU-Speicherzuweisung von Thread 3 ändert sich zudem dynamisch, sobald die Threads 1 und 2 abgeschlossen sind – siehe [Dynamische GPU-Speicherzuweisung](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Nächste Schritte

* [Dynamische Rechenanpassung](dynamic-compute-adaptation.md) — Wie Chloros die optimale Strategie für Ihre Hardware auswählt
* [NVIDIA Jetson-Leitfaden](../linux/nvidia-jetson-guide.md) — Plattformspezifisches Pipeline-Verhalten auf Jetson
* [Überwachung der Verarbeitung](../processing-images-gui/monitoring-the-processing.md) — Fortschrittsüberwachung über die GUI
