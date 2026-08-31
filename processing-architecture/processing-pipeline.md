# Verarbeitungs-Pipeline

Chloros1.2.0 nutzt eine 4-Thread-Verarbeitungs-Pipeline, die wie ein stufenweises Fließband funktioniert. Jeder Thread übernimmt eine bestimmte Phase des Workflows, sodass mehrere Bilder gleichzeitig in verschiedenen Phasen bearbeitet werden können.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architektur der Pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Jedes Bild durchläuft nacheinander alle vier Threads. Bei der multithreaded Verarbeitung von Chloros+ belegen mehrere Bilder gleichzeitig verschiedene Threads – während Thread 3 ein Bild verarbeitet, kann Thread 1 bereits das nächste erkennen, Thread 2 ein weiteres kalibrieren und Thread 4 ein fertiges Bild auf die Festplatte schreiben.

Der Fortschritt wird pro Thread gemeldet und über „Server-Sent Events“ übertragen (das Backend veröffentlicht diese auf `/api/events`). In der Live-Fortschrittsanzeige des „CLI“ sind die vier Phasen mit **Erkennung, Analyse, Verarbeitung, Export** beschriftet.***

## Thread-Details

### Thread 1: Erkennung

**Zweck**: Bilder laden und Kalibrierungsziele erkennen.

* Liest Bilddateien von der Festplatte – Paare aus den Dateien „Survey3“ und „`.raw`+`.jpg`“, LATTICE-Aufnahmen (`.tif`/`.tiff`) sowie `.dng`
* Extrahiert EXIF-Metadaten (GPS, Kameramodell, Zeitstempel, Belichtung)
* Erkennt Kalibrierungsziele: ArUco-markierte Zielgeometrien für LATTICE-Aufnahmen sowie den klassischen Panel-Detektor für Survey3-Kalibrierungszielfotos
* Ausgabe: Bilddaten + Metadaten + Ergebnisse der Zielerkennung

In erster Linie ein I/O- und CPU-gebundener Thread.

### Thread 2: Kalibrierung

**Zweck**: Berechnung der Kalibrierungsparameter aus den erkannten Zielmarken.

* Berechnet Reflexionskalibrierungskoeffizienten aus Zielbildern
* Berechnet Vignettierungskorrekturparameter
* Ermittelt bandweise Kalibrierungskurven
* Ausgaben: Kalibrierungsparameter für jedes Bild

Ein CPU-gebundener Berechnungs-Thread. Thread 3 wartet darauf, wenn die Reflexionskalibrierung aktiviert ist, damit die Koeffizienten bereitstehen, bevor ein Bild verarbeitet wird.

### Thread 3: Verarbeitung (GPU)

**Zweck**: Korrekturen anwenden und Vegetationsindizes berechnen.**Dies ist der rechenintensivste Thread.*** **Debayering**: Konvertiert RAW-Bayer-Daten in Mehrkanalbilder
  * Standard (schnell, mittlere Qualität) – Standard, `--debayer standard`
  * Texturorientiert (langsam, höchste Qualität) – nur „Chloros+“, `--debayer texture-aware`, verwendet ein KI/ML-Rauschunterdrückungsmodell
  * LATTICE-Mono-Aufnahmen (M3M) sind einkanalig: Die Schritte „Demosaicing“ und „Weißabgleich“ werden bei ihnen übersprungen (mit einer einzeiligen Protokollmeldung), während alle M3C-/Bayer-Bilder im selben Durchlauf diese Schritte weiterhin durchlaufen
* **Vignettierungskorrektur**: Wendet eine Korrektur der Objektivvignettierung auf das gesamte Bild an
* **Reflexionskalibrierung**: Wendet Kalibrierungskoeffizienten an, um die Werte in Reflexionswerte umzuwandeln
* **Indexberechnung**: Berechnet Vegetationsindizes (NDVI, NDRE, GNDVI, …)
* Ausgabe: Verarbeitete Bilddaten, bereit zum Export

Dieser Thread profitiert am meisten von der GPU-Beschleunigung und ist der Thread, den [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optimiert.

### Thread 4: Export

**Zweck**: Schreiben der verarbeiteten Bilder auf die Festplatte.

* Schreibt Ausgabedateien im ausgewählten Format – `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Betten Metadaten in die Ausgabedateien ein (GPS, Zeitstempel, Verarbeitungsparameter)
* Organisiert die Ausgabe im Projektordner als `<camera>/<format>/<Product>_Images/` – zum Beispiel `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Exportierte Dateien behalten den Namen der Quelldatei bei; der Ordner identifiziert das Produkt.**
* Bei LATTICE-Aufnahmen kann ein Quellbild in mehrere Produkte aufgeteilt werden (Debayered, Preview, Radiance, Reflectance, Index), die jeweils in einem eigenen Produktordner abgelegt werden
* Ausgabe: endgültige Dateien auf der Festplatte

In erster Linie ein I/O-gebundener Thread – SSD-Speicher verbessert die Leistung spürbar.

***

## Hinter den Kulissen: Executors

Innerhalb von Thread 3 wird die bildspezifische Verarbeitung mit dem Standard-`concurrent.futures` von „Python“ parallelisiert:

* **GPU-Strategien**(`GPU_SINGLE`, `GPU_PARALLEL`) verwenden eine `ProcessPoolExecutor` mit der**spawn** – jeder Worker ist ein separater Prozess mit eigenem CUDA-Kontext (`fork` würde den initialisierten CUDA-Zustand des Elternprozesses erben und die Kindprozesse beschädigen)
* **`CPU_PARALLEL`** verwendet ein `ThreadPoolExecutor` – NumPy und OpenCV geben den GIL frei, sodass Threads ausreichen
* Jetson-Geräte mit 8 GB oder weniger gemeinsam genutztem RAM überspringen den Executor vollständig und verarbeiten prozessintern sequenziell
* „Texture Aware“ auf einer GPU mit weniger als 7 GB VRAM läuft ebenfalls sequenziell – das Denoiser-Modell kann nicht mehr als einmal angepasst werden „

Chloros“ verwendet kein verteiltes Framework von Drittanbietern (wie z. B. Ray). Unter [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) erfahren Sie, wie die Strategie und die Anzahl der Worker ausgewählt werden.

***

## Sequenzielle vs. pipelinierte Verarbeitung

### Freier Modus (sequenziell)

In der kostenlosen Version von „Chloros“ werden Bilder **einzeln** und sequenziell durch alle vier Phasen verarbeitet:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Die Benutzeroberfläche zeigt im kostenlosen Modus einen vereinfachten Fortschrittsbalken an; die einzelnen Phasen werden als **Zielerkennung**und anschließend als**Verarbeitung** angezeigt.

### „Chloros+“-Modus (Pipelining)

Mit einer „Chloros+“-Lizenz arbeiten alle vier Threads **gleichzeitig** an verschiedenen Bildern:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Der Fortschrittsbalken in der Benutzeroberfläche zeigt die vier Phasen an; wenn Sie mit der Maus darüberfahren, sehen Sie den Fortschritt pro Thread. Im „CLI“ werden dieselben vier Phasen live als **Erkennung, Analyse, Verarbeitung, Export** angezeigt.

{% hint style="info" %}
**Eine Bezeichnung, zwei Namen.** Die „CLI“ bezeichnet Phase 3 als _Processing_. Der Fortschritts-Feed des Backends im Premium-Modus – derjenige, den der Fortschrittsbalken der Benutzeroberfläche anzeigt – bezeichnet dieselbe Phase als _Calibrating_. Es handelt sich um denselben Thread, der dieselbe Arbeit ausführt (Thread 3: Debayering, Korrekturen, Indizes).
{% endhint %}

{% hint style="success" %}
**Die pipelinierte Verarbeitung mit „Chloros“+** kann je nach Hardware und Datensatzgröße 3- bis 5-mal schneller sein als die sequenzielle Verarbeitung. Die Beschleunigung ist auf Systemen mit schnellen GPUs und SSDs am größten.
{% endhint %}

***

## Thread 4: Exportfortschritt

Der Export-Thread verfügt über eine eigene Fortschrittsanzeige, die Sie separat abfragen können:

**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Die Verarbeitung ist abgeschlossen, wenn Thread 4 100 % erreicht.

{% hint style="info" %}
**Ein Durchlauf, bei dem keine Bilder geschrieben werden, gilt als Fehlschlag.**Bei Erfolg meldet `chloros-cli process`, wie viele Bildprodukte geschrieben wurden (`Image products written: N`). Wenn Produkte angefordert wurden und**keine**geschrieben wurden – nur `project.json` und `calibration_data.json` –, gibt der „CLI“ `Processing finished but wrote no image products.` aus und**beendet den Vorgang mit einem Wert ungleich Null**, wobei der Projektordner und die üblichen Ursachen genannt werden (der Eingabeordner wurde nicht als Aufzeichnung erkannt – überprüfen Sie das Layout und `--input-level` – oder jedes angeforderte Produkt war für diese Kameras nicht anwendbar). Skripte können sich auf den Beendigungscode verlassen.
{% endhint %}

***

## Zusammenhang mit der dynamischen Rechenanpassung

[Dynamische Rechenanpassung](dynamic-compute-adaptation.md) wirkt sich in erster Linie auf **Thread 3 (Verarbeitung)** aus:

* **`GPU_PARALLEL`**: Thread 3 verarbeitet mehrere Bilder gleichzeitig über die GPU mithilfe der `fused_gpu`-Pipeline
* **`GPU_SINGLE`**: Thread 3 serialisiert den GPU-Zugriff mit einem Semaphor, während Worker-Prozesse die E/A-Vorgänge überlappen, wobei entweder die `fused_gpu`-Pipeline oder die speichereffiziente `tiled_gpu`-Pipeline verwendet wird
* **`CPU_PARALLEL`**: Thread 3 nutzt CPU-basierte Verarbeitung mit Multithread-Parallelität

Die GPU-Speicherzuweisung von Thread 3 wächst zudem, sobald die Threads 1 und 2 fertig sind – siehe [Dynamische GPU-Speicherzuweisung](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Nächste Schritte

* [Dynamische Rechenanpassung](dynamic-compute-adaptation.md) – Wie „Chloros“ die optimale Strategie für Ihre Hardware auswählt
* [NVIDIA Jetson-Leitfaden](../linux/nvidia-jetson-guide.md) – Plattformspezifisches Pipeline-Verhalten auf Jetson
* [Überwachung der Verarbeitung](../processing-images-gui/monitoring-the-processing.md) – Fortschrittsüberwachung über die GUI
* [Referenz zu CLI](../reference/cli-reference.md) – `process`, `export-status`, Exit-Codes und Ausgabelayout
