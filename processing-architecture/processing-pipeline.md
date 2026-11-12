# Verarbeitungspipeline

Chloros 1.3.0 nutzt eine Verarbeitungspipeline mit vier Threads, die wie ein stufenweises Fließband funktioniert. Jeder Thread übernimmt eine bestimmte Phase des Workflows, sodass mehrere Bilder gleichzeitig in verschiedenen Phasen bearbeitet werden können.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architektur der Pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Jedes Bild durchläuft nacheinander alle vier Threads. Bei der Multithread-Verarbeitung vChloros+ belegen mehrere Bilder gleichzeitig verschiedene Threads – während Thread 3 ein Bild verarbeitet, kann Thread 1 bereits das nächste erkennen, Thread 2 ein weiteres kalibrieren und Thread 4 ein fertiges Bild auf die Festplatte schreiben.

Der Fortschritt wird pro Thread gemeldet und über „Server-Sent Events“ übertragen (das Backend veröffentlicht diese auf `/api/events`). In der Live-Fortschrittsanzeige der CLI werden die vier Phasen mit **Erkennung, Analyse, Verarbeitung, Export** bezeichnet.

***

## Thread-Details

### Thread 1: Erkennung

**Zweck**: Bilder laden und Kalibrierungsziele erkennen.

* Liest Bilddateien von der Festplatte – Survey3-Paare `.raw`+`.jpg`, LATTICE-Aufnahmen `.tif`/`.tiff` sowie `.dng`
* Extrahiert EXIF-Metadaten (GPS, Kameramodell, Zeitstempel, Belichtung)
* Erkennt Kalibrierungsziele: ArUco-markierte Zielgeometrien für LATTICE-Aufnahmen sowie den klassischen Panel-Detektor für Survey3-Kalibrierungszielfotos
* Ausgabe: Bilddaten + Metadaten + Ergebnisse der Zielerkennung

In erster Linie ein I/O- und CPU-gebundener Thread.

### Thread 2: Kalibrierung

**Zweck**: Berechnung der Kalibrierungsparameter aus den erkannten Zielen.

* Berechnet Reflexionskalibrierungskoeffizienten aus Zielbildern
* Berechnet Vignettierungskorrekturparameter
* Ermittelt bandweise Kalibrierungskurven
* Ausgabe: Kalibrierungsparameter für jedes Bild

Ein CPU-gebundener Berechnungs-Thread. Bei einem Survey3- oder gemischten Durchlauf wartet Thread 3 darauf, wenn die Reflektanzkalibrierung aktiviert ist, sodass dessen Koeffizienten bereitstehen, bevor ein Bild verarbeitet wird. Bei einem reinen „LATTICE“-Durchlauf läuft Thread 3 während der Erkennung: Jeder Frame beginnt, sobald sein Zielanker endgültig feststeht. Ein Bild nimmt das zeitlich nächstgelegene Ziel auf beiden Seiten, daher wartet es, bis jedes Bild seiner Kamera, das zeitlich näher liegt als sein aktuelles nächstgelegenes Ziel, gescannt wurde. Die Ergebnisse sind dieselben, nur früher verfügbar.

### Thread 3: Verarbeitung (GPU)

**Zweck**: Korrekturen anwenden und Vegetationsindizes berechnen.

**Dies ist der rechenintensivste Thread.**

* **Debayering**: Konvertiert RAW-Bayer-Daten in Mehrkanalbilder
  * Standard (schnell, mittlere Qualität) – Standard, `--debayer standard`
  * Texturorientiert (langsam, höchste Qualität) – nur „Chloros+“, `--debayer texture-aware`, verwendet ein KI/ML-Rauschunterdrückungsmodell
  * LATTICE-Mono-Aufnahmen (M3M) sind einkanalig: Die Schritte zur Demosaikierung und zum Weißabgleich werden bei ihnen übersprungen (mit einer einzeiligen Protokollmeldung), während alle M3C-/Bayer-Bilder im selben Durchlauf diese Schritte weiterhin durchlaufen
* **Vignettierungskorrektur**: Wendet eine Korrektur der Objektivvignettierung auf das gesamte Bild an
* **Reflexionskalibrierung**: Wendet Kalibrierungskoeffizienten an, um die Werte in Reflexionswerte umzuwandeln
* **Indexberechnung**: Berechnet Vegetationsindizes (NDVI, NDRE, GNDVI, …)
* Ausgabe: verarbeitete Bilddaten, bereit zum Export

Dieser Thread profitiert am meisten von der GPU-Beschleunigung, und er ist der Thread, den die [Dynamische Rechenanpassung](dynamic-compute-adaptation.md) optimiert.

### Thread 4: Export

**Zweck**: Speichern der verarbeiteten Bilder auf der Festplatte.

* Schreibt Ausgabedateien im ausgewählten Format – `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Betten Metadaten in die Ausgabedateien ein (GPS, Zeitstempel, Verarbeitungsparameter) – einschließlich der `Chloros:*`-Verarbeitungs-Tags, die ab Version 1.3.0 in jedem Verarbeitungsmodus geschrieben werden, sowohl für Survey3 als auch für „LATTICE“
* Organisiert die Ausgabe im Projektordner als `<camera>/<format>/<Product>_Images/` – zum Beispiel `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Exportierte Dateien behalten den Namen der Quelldatei bei; der Ordner identifiziert das Produkt.**
* Bei LATTICE-Aufnahmen kann ein Quellbild in mehrere Produkte (Debayered, Preview, Radiance, Reflectance, Index) aufgeteilt werden, die jeweils in einem eigenen Produktordner abgelegt werden. Indexprodukte werden aus dem vom Lauf geschriebenen Reflektanzprodukt berechnet (falls kein solches vorhanden ist, aus einem anderen vom Lauf geschriebenen Produkt, wie z. B. Radiance). Strahlungs- und Reflexionsprodukte von multispektralen M3C-Kameras können in `<format>/clamp_masks/` auch eine pixelweise Clamp-Maske erhalten (siehe [Ausgabebildformate](../output-image-formats.md#clamp-masks))
* Ausgaben: endgültige Dateien auf der Festplatte

In erster Linie ein I/O-gebundener Thread – SSD-Speicher verbessert die Leistung spürbar.

***

## Hinter den Kulissen: Executor-Prozesse

Innerhalb von Thread 3 wird die bildspezifische Arbeit wie folgt parallelisiert:

* **GPU-Strategien**(`GPU_SINGLE`, `GPU_PARALLEL`) nutzen einen Pool von**Worker-Prozessen**, bei denen es sich jeweils um separate Betriebssystemprozesse mit eigenem CUDA-Kontext handelt. Chloros startet jeden Worker, indem es dessen eigenes installiertes Backend-Programm erneut ausführt – nicht über Pythons `multiprocessing`, dessen Worker-Start innerhalb der gepackten Anwendung nicht funktioniert
* **`CPU_PARALLEL`** nutzt einen Thread-Pool (Pythons `ThreadPoolExecutor`) – NumPy und OpenCV geben den GIL frei, sodass Threads ausreichen
* Jetson-Geräte mit 8 GB oder weniger gemeinsam genutztem RAM überspringen den Executor vollständig und verarbeiten die Aufgaben prozessintern und sequenziell
* „Texture Aware“ auf einer GPU mit weniger als 7 GB VRAM läuft ebenfalls sequenziell – das Denoiser-Modell passt nicht mehr als einmal hinein

Chloros verwendet kein verteiltes Framework von Drittanbietern (wie beispielsweise Ray). Siehe [Dynamische Rechenanpassung](dynamic-compute-adaptation.md) zur Auswahl der Strategie und der Anzahl der Worker.

***

## Sequenzielle vs. pipelinierte Verarbeitung

### Freier Modus (sequenziell)

In der kostenlosen Version von Chloros werden Bilder **einzeln** nacheinander durch alle vier Phasen verarbeitet:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Die Benutzeroberfläche zeigt im kostenlosen Modus einen vereinfachten Fortschrittsbalken an; die einzelnen Phasen werden als **Zielerkennung**und anschließend als**Verarbeitung** angezeigt.

### Chloros+-Modus (Pipelined)

Mit einer Chloros+-Lizenz arbeiten alle vier Threads **parallel** an verschiedenen Bildern:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Der Fortschrittsbalken der GUI zeigt die vier Phasen an; wenn Sie mit der Maus darüberfahren, sehen Sie den Fortschritt pro Thread. In der CLI werden dieselben vier Phasen live als **Erkennung, Analyse, Verarbeitung, Export** angezeigt.

{% hint style="info" %}
**Eine Bezeichnung, zwei Namen.** Die CLI bezeichnet Phase 3 als _Verarbeitung_. Der Fortschritts-Feed des Backends im Premium-Modus – derjenige, den der Fortschrittsbalken der GUI anzeigt – bezeichnet dieselbe Phase als _Kalibrierung_. Es handelt sich um denselben Thread, der dieselbe Arbeit ausführt (Thread 3: Debayering, Korrekturen, Indizes).
{% endhint %}

{% hint style="success" %}
**Die Pipeline-Verarbeitung mit „Chloros+“** kann je nach Hardware und Datensatzgröße 3- bis 5-mal schneller sein als die sequenzielle Verarbeitung. Die Beschleunigung ist auf Systemen mit schnellen GPUs und SSDs am größten.
{% endhint %}

***

## Fortschritt des Export-Threads 4

Der Export-Thread verfügt über eine eigene Fortschrittsanzeige, die Sie separat abfragen können:**Über die CLI**:

```bash
chloros-cli export-status
```

**Über das SDK**:

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Die Verarbeitung ist abgeschlossen, wenn Thread 4 100 % erreicht hat.

{% hint style="info" %}
**Ein Durchlauf, bei dem keine Images geschrieben werden, gilt als Fehler.**Bei Erfolg meldet `chloros-cli process`, wie viele Image-Produkte geschrieben wurden (`Image products written: N`). Wenn Produkte angefordert wurden und**keine**geschrieben wurden – nur `project.json` und `calibration_data.json` –, gibt die CLI `Processing finished but wrote no image products.` aus und**beendet den Vorgang mit einem Wert ungleich Null**, wobei der Projektordner und die üblichen Ursachen genannt werden (der Eingabeordner wurde nicht als Aufnahme erkannt – überprüfen Sie das Layout und `--input-level` – oder jedes angeforderte Produkt war für diese Kameras nicht anwendbar). Skripte können sich auf den Beendigungscode verlassen.
{% endhint %}

***

## Zusammenhang mit der dynamischen Rechenanpassung

[Dynamische Rechenanpassung](dynamic-compute-adaptation.md) wirkt sich in erster Linie auf **Thread 3 (Verarbeitung)** aus:

* **`GPU_PARALLEL`**: Thread 3 verarbeitet mehrere Bilder gleichzeitig über die GPU mithilfe der `fused_gpu`-Pipeline
* **`GPU_SINGLE`**: Thread 3 serialisiert den GPU-Zugriff mit einem Semaphor, während Worker-Prozesse die E/A-Vorgänge überlappen, wobei entweder die `fused_gpu`-Pipeline oder die speichereffiziente `tiled_gpu`-Pipeline verwendet wird
* **`CPU_PARALLEL`**: Thread 3 nutzt CPU-basierte Verarbeitung mit Multithread-Parallelität

Die GPU-Speicherzuweisung von Thread 3 wächst zudem, sobald die Threads 1 und 2 ihre Aufgaben abgeschlossen haben – siehe [Dynamische GPU-Speicherzuweisung](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Nächste Schritte

* [Dynamische Rechenanpassung](dynamic-compute-adaptation.md) – Wie Chloros die optimale Strategie für Ihre Hardware auswählt
* [NVIDIA Jetson-Leitfaden](../linux/nvidia-jetson-guide.md) – Plattformspezifisches Pipeline-Verhalten auf Jetson
* [Überwachung der Verarbeitung](../processing-images-gui/monitoring-the-processing.md) – Fortschrittsanzeige über die grafische Benutzeroberfläche
* [CLI-Referenz](../reference/cli-reference.md) – `process`, `export-status`, Exit-Codes und Ausgabeformat
