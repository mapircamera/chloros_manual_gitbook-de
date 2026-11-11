# Dynamische Rechenanpassung

Chloros 1.3.0 nutzt Hardware-Erkennung und die automatische Auswahl der Verarbeitungsstrategie. Die Verarbeitungs-Engine passt sich Ihrer Hardware an – vom Jetson Orin Nano bis hin zu einer Workstation mit mehreren GPUs – ganz ohne manuelle Konfiguration.

***

## So funktioniert es

Beim Start von Chloros wird Ihr System profiliert:

1. **Erkennt das Betriebssystem** – Windows oder Linux
2. **Ermittelt CPU-Kerne und den gesamten Arbeitsspeicher**

3.**Erkennt das Vorhandensein einer GPU** – NVIDIA-CUDA-Fähigkeit, VRAM, Modell
4. **Ermittelt das Jetson-Modell** (falls zutreffend) – über `/proc/device-tree/model`
5. **Überprüft die Temperatursensoren** (Jetson) – für temperaturbewusste Verarbeitung
6. **Wählt die Rechenstrategie aus** – basierend auf der gesamten erkannten Hardware
7. **Konfiguriert die Anzahl der Worker, den Pipeline-Typ und die Speicherzuweisung** automatisch

Das erkannte Profil wird für die Sitzung im Arbeitsspeicher und auf der Festplatte zwischengespeichert, sodass spätere Ausführungen schneller starten:

| Plattform | Zwischengespeichertes Profil |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (berücksichtigt `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Löschen Sie diese Datei, um eine neue Erkennung zu erzwingen – nützlich nach dem Hinzufügen einer GPU oder von mehr RAM. Chloros führt außerdem automatisch eine erneute Erkennung durch, wenn der Cache von einer inkompatiblen älteren Version geschrieben wurde.

***

## Rechenstrategien

Chloros wählt je nach Ihrer Hardware eine von drei Rechenstrategien aus:

| Strategie | Ausgewählt, wenn | Worker | Executor | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA-GPU mit**12 GB+ VRAM**(auf Jetson Unified Memory, erfordert außerdem mindestens 12 GB gemeinsam genutzten RAM) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, mindestens 2 —**auf Jetson auf 2 begrenzt** | Worker-Prozesse (separate CUDA-Kontexte) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA-GPU mit**2–12 GB VRAM**| 3 (I/O-Überlappung; GPU-Zugriff durch einen Semaphor serialisiert).**1 (sequenziell) auf Jetsons mit weniger als 12 GB RAM** | Worker-Prozesse (separate CUDA-Kontexte); sequenziell innerhalb eines Prozesses auf Jetsons mit geringem RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Keine CUDA-GPU oder weniger als 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Anwendungsbeispiele für die `GPU_PARALLEL`-Worker-Formel: 12 GB VRAM → 2 Worker, 16 GB → 3 Worker, 20 GB+ → 4 Worker. Jedem Worker werden 3,5 GB zugewiesen – der gemessene Bedarf einer „Texture Aware“-Aufgabe –, und zwar im Rahmen von 90 % des verbleibenden VRAM nach Abzug einer Reserve von 0,5 GB.

{% hint style="info" %}
**Die GPU-Strategien erfordern eine PyTorch-Version, die CUDA nutzen kann.** Ein Rechner, dessen PyTorch über kein nutzbares CUDA-Gerät verfügt, erhält `CPU_PARALLEL`, unabhängig von seiner GPU. Das Jetson-Paket 1.3.0 verwendet den CUDA-Build von PyTorch aus JetPack 6, sodass ein Jetson die Voraussetzungen erfüllt, sobald die CUDA- und cuDNN-Bibliotheken von JetPack installiert sind. Ein Jetson mit weniger als 12 GB RAM verarbeitet immer jeweils ein Bild im laufenden Prozess (protokolliert als `GPU_SINGLE`).
{% endhint %}

GPU-Strategien führen einen Pool von Worker-Prozessen aus, bei denen es sich jeweils um separate Betriebssystemprozesse mit eigenem CUDA-Kontext handelt, die Chloros durch erneutes Starten seines eigenen installierten Backend-Programms in Gang setzt (nicht über Pythons `multiprocessing`, dessen Worker-Start innerhalb der gepackten Anwendung nicht funktioniert). Die CPU-Strategie nutzt einen Thread-Pool (Pythons `ThreadPoolExecutor`). Chloros verwendet kein verteiltes Framework von Drittanbietern (wie beispielsweise Ray).

### Pipeline-Typen

* **`fused_gpu`** – Vollständiger GPU-Verarbeitungspfad. Debayer-, Korrektur- und Indexoperationen werden auf der GPU in einem einzigen fusionierten Durchlauf ausgeführt. Höchster Durchsatz, benötigt am meisten VRAM.
* **`tiled_gpu`** – Speichereffizienter GPU-Verarbeitungsweg. Verarbeitet Bilder in Kacheln, um sie an den begrenzten GPU-Speicher anzupassen. Geringerer Durchsatz, funktioniert jedoch auf Geräten mit begrenztem Speicher.
* **`cpu_fallback`** – Ausschließlich CPU-basierte Verarbeitung unter Verwendung von Multithread-Parallelität. Wird verwendet, wenn keine NVIDIA-GPU verfügbar ist, sowie als Fallback-Lösung letzter Instanz, wenn beide GPU-Verarbeitungswege fehlschlagen.

Die Fallback-Kette zur Laufzeit lautet immer `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Manuelle Übersteuerung der Strategie

Setzen Sie die Umgebungsvariable `CHLOROS_STRATEGY`, um eine bestimmte Strategie zu erzwingen – eine Experten-Notfalllösung für den Fall, dass die automatische Erkennung eine für Ihre Situation ungeeignete Option auswählt (z. B. um die GPU für andere Aufgaben freizuhalten):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Bei der Abgleichung der Variablen wird die Groß-/Kleinschreibung nicht berücksichtigt; alles, was nicht einer der drei Namen ist, wird ignoriert, und die automatische Erkennung läuft normal weiter. Auch bei einer Übersteuerung wählt Chloros weiterhin die Anzahl der Worker für Sie aus:

| Übersteuerung | Verwendete Worker-Anzahl |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Eine Überschreibung durch „`GPU_SINGLE`“ oder „`GPU_PARALLEL`“ auf einem Rechner ohne verwendbares CUDA-Gerät wird mit einer Zeile „`[STRATEGY]`“ im Protokoll abgelehnt, und der Lauf verwendet „`CPU_PARALLEL`“.

Es wird empfohlen, die Einstellung pro Befehl statt dauerhaft vorzunehmen, damit sich normale Läufe weiterhin automatisch anpassen.

***

## Plattformspezifisches Verhalten

| Plattform | Strategie | Worker | Pipeline | Anmerkungen |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequenziell) | Speichereffizienter Modus, jeweils ein Bild |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequenziell) | Gemeinsamer Arbeitsspeicher unter 12 GB erzwingt sequenzielle Verarbeitung |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (parallel) | Empfohlenes Edge-Gerät – bei Jetson auf 2 Worker begrenzt |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (parallel) | Maximale Edge-Leistung (ebenfalls bei Jetson auf 2 Worker begrenzt) |
| **Desktop mit 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 Worker überlappen sich bei der E/A, während ein Semaphor den GPU-Zugriff serialisiert |
| **Desktop mit 12 GB+ GPU** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (gleichzeitig) | Optimale Desktop-Leistung: 12 GB → 2 Worker, 16 GB → 3, 20 GB+ → 4 |
| **Reines CPU-System** | `CPU_PARALLEL` | physische Kerne − 1 (mind. 2) | `cpu_fallback` | Keine GPU erforderlich, nutzt einen Thread-Pool |

{% hint style="info" %}
**Jetson Unified Memory**: Jetson-Geräte teilen sich GPU- und CPU-Speicher. Ein Jetson Orin NX mit 16 GB meldet ca. 15,3 GB VRAM, doch dabei handelt es sich um denselben physischen RAM, der auch vom Betriebssystem und den CPU-Prozessen genutzt wird. Aus diesem Grund erfüllen Jetson-Geräte mit 16 GB und mehr die Voraussetzungen für `GPU_PARALLEL` wie eine Desktop-GPU mit 12 GB und mehr, sind jedoch auf 2 Worker begrenzt – die GPU, die Worker-Prozesse und ihre CUDA-Kontexte pro Worker greifen alle auf denselben gemeinsamen Pool zu.
{% endhint %}

### GPU-Budget nach VRAM (diskrete GPUs)

Auf x86_64-Hosts mit einer diskreten NVIDIA-GPU bestimmt der erkannte VRAM auch, wie viel Rechenleistung die Karte beanspruchen darf und wie groß Batches werden können:

| Erkannter VRAM | Obergrenze des GPU-Budgets | Multiplikator für die Batch-Größe |
| --- | --- | --- |
| **8 GB+** | 90 % | ×2,0 |
| **6–8 GB** | 85 % | ×1,75 |
| **3,5–6 GB** | 80 % | ×1,5 |
| **2–3,5 GB** | 75 % | ×1,25 |
| **Unter 2 GB** | 70 % | ×1,0 |

Diskrete GPUs reservieren nur 0,5 GB für das System, da sie den System-RAM nicht gemeinsam nutzen. Jetson-Profile reservieren weitaus mehr und haben eine niedrigere Obergrenze – siehe den [NVIDIA Jetson-Leitfaden](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynamische GPU-Speicherzuweisung

Chloros verwendet eine [4-Thread-Verarbeitungspipeline](processing-pipeline.md):

* **Thread 1** (Erkennung) – Bildladen, EXIF-Analyse, Zielerkennung
* **Thread 2** (Kalibrierung) – Berechnung der Reflexionskalibrierung
* **Thread 3** (Verarbeitung) – GPU-Debayering, Vignettierungskorrektur, Indexberechnung
* **Thread 4** (Export) – Dateischreiben, Einbetten von Metadaten

Die Threads 1, 2 und 4 beanspruchen die GPU nur geringfügig; Thread 3 ist der rechenintensive Thread. Sobald frühere Pipeline-Threads abgeschlossen sind, wird ihr GPU-Budget **auf die verbleibenden aktiven Threads umverteilt**, sodass Thread 3 im Laufe der Ausführung zunehmend mehr Speicher erhält.

### Zuweisungsphasen

| Phase | Aktive Threads | GPU-Speicherverteilung |
| --- | --- | --- |
| **Früh** | 1, 2, 3, 4 | Auf alle Threads aufgeteilt, der größte Teil geht an Thread 3 |
| **Mitte-Früh** | 2, 3, 4 | Der Anteil von Thread 1 wird umverteilt |
| **Mitte-Spät** | 3, 4 | Die Anteile der Threads 1 und 2 gehen an 3 und 4 |
| **Spät** | 3 oder 4 | Der zuletzt aktive Thread erhält seine maximale Zuweisung |

Zwei Regeln bestimmen die Zahlen:

* Ein Thread, der der **einzige** aktive ist, erhält die maximale Zuweisung seines Profils.
* Wenn mehr als eine *ressourcenintensive* GPU-Aufgabe aktiv ist, wird die Basiszuweisung jeder ressourcenintensiven Aufgabe auf diese aufgeteilt (niemals unter das konfigurierte Minimum).

Der tatsächlich zur Laufzeit verwendete Wert ist der **niedrigere** Wert aus der Zuweisung des Plattformprofils und der aktuellen Empfehlung des GPU-Speichermonitors, sodass eine ausgelastete Karte immer Vorrang vor einem optimistischen Profil hat.***

## Textur-bewusste Verarbeitung

Der „Texture Aware“-Debayer (**nur Chloros+** – `--debayer texture-aware`) führt ein KI/ML-Rauschunterdrückungsmodell aus, das pro Kopie etwa 1,75 GB VRAM in FP16 benötigt; daher verbraucht er weitaus mehr GPU-Speicher als die Standardmethode:

* Systeme mit **weniger als 7 GB VRAM**verarbeiten „Texture Aware“ in einer**synchronen Schleife, Bild für Bild** – mehrere Modellkopien passen nicht hinein, und ein Worker-Pool würde nur zu Konflikten führen
* Systeme mit **7 GB oder mehr VRAM** können „Texture Aware“ parallel verarbeiten, allerdings mit einer im Vergleich zum Standardverfahren reduzierten Anzahl an Workern
* Auf **Jetson** wird „Texture Aware“ immer an einen einzelnen Worker gebunden
* Wenn **keine nutzbare CUDA-GPU** vorhanden ist, verwendet der Lauf den Standard-Debayer, und das Backend protokolliert eine Zeile „`[DEBAYER] … needs a CUDA GPU …`“, in der der Grund dafür angegeben wird
* Auf Jetson-Geräten mit geringer Leistungsaufnahme (Nano, Orin Nano) wendet die CLI eine GPU-Frequenzbegrenzung an, wenn Sie „Texture Aware“ anfordern – siehe den [NVIDIA Jetson-Leitfaden](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Wärmemanagement (Jetson)

Jetson-Geräte unterliegen thermischen Einschränkungen, insbesondere bei Einsätzen in geschlossenen Räumen oder in Flugzeugen. Chloros überwacht die integrierten Temperatursensoren des Jetson und skaliert die Batch-Größen automatisch:

| Temperatur | Reaktion |
| --- | --- |
| **< 70 °C** | Normalbetrieb – volle Geschwindigkeit |
| **70 °C** (Warnung) | Die Batch-Größe wird schrittweise verringert (100 % → 50 % zwischen 70 °C und 80 °C) |
| **80 °C** (kritisch) | Aggressive Drosselung (50 % → 0 % zwischen 80 °C und 90 °C) |
| **90 °C** (Herunterfahren) | GPU-Verarbeitung vollständig stoppen |

Auf Desktop-Systemen mit ausreichender Kühlung wird die thermische Drosselung selten ausgelöst.

***

## Umgang mit Speicherbelastung

Chloros überwacht den GPU-Speicher während der Verarbeitung kontinuierlich und reagiert auf drei Ebenen.

**Batch-Größe.** Ein Batch beginnt mit 8 Bildern multipliziert mit dem Plattform-Multiplikator aus den obigen Tabellen. Chloros prüft dann den freien VRAM, reserviert 20 % davon für den eigenen Overhead von PyTorch und geht von etwa 100 MB GPU-Speicher pro 12-MP-Bild aus – der Batch entspricht dem jeweils kleineren Wert aus der durch den Speicher abgeleiteten Grenze und der Plattform-Basis. Sie fällt niemals unter 1.

**Vorbeugende Reduzierung.**Bei einer**VRAM-Auslastung von über 85 %** werden die Batch-Größen reduziert, bevor es zu Ausfällen kommt.**Reduzierung der Zuweisung pro Thread.** Mit steigender aktueller Auslastung wird das GPU-Budget jedes Threads gekürzt: ×0,75 bei einer Auslastung über 80 %, ×0,5 bei einer Auslastung über 90 %. Die Überwachungsschwellen liegen bei 70 % (konservativ), 85 % (normale Betriebsgrenze) und 95 % (OOM-Risiko).**OOM-Backoff und Wiederherstellung.** Sollte es dennoch zu einem Out-of-Memory-Ereignis kommen:

* wird die Batchgröße **halbiert** und bei jedem weiteren „OOM“-Ereignis erneut halbiert – jeder nachfolgende erfolgreiche Batch versetzt diese Strafe um einen Schritt zurück
* werden die Zuweisungen für aktive Threads auf 70 % ihres aktuellen Werts gekürzt, und der Allokator wechselt zu seiner konservativen Strategie, die sich nach einer Reihe erfolgreicher Zuweisungen wieder lockert
* Bei starker Belastung wechselt die Pipeline von `fused_gpu` zu `tiled_gpu` und als letztes Mittel zu `cpu_fallback`

**Host-RAM (Jetson).** Vor der Verarbeitung schätzt die CLI den Spitzenbedarf an Host-Speicher anhand Ihrer Bildanzahl und des Debayer-Modus und warnt, wenn RAM plus dateibasierter Swap-Speicher voraussichtlich nicht ausreichen. Dabei werden die genauen Befehle zum Hinzufügen von Swap-Speicher ausgegeben – siehe den [NVIDIA Jetson-Leitfaden](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Überwachung der Rechenanpassung

### Systemdiagnose

`chloros-cli selftest` ist der schnellste Weg, um zu überprüfen, was die Rechenschicht erkennt:

```bash
chloros-cli selftest
```

Die 7 Prüfungen umfassen Version, Portverfügbarkeit, Backend-Start, `/api/test`, Systeminformationen, Vorhandensein des Denoiser-Modells sowie die Bereitschaft von CUDA und Denoiser. Prüfung 5 gibt die Hardware-Zeile direkt aus:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Prüfung 7 gibt `CUDA: <bool>, Denoiser: <bool>` aus – beide müssen wahr sein, damit „Texture Aware“ überhaupt genutzt werden kann.

### Backend-Protokolle

Die Strategie und die Anzahl der Worker werden zu Beginn jedes Durchlaufs im Backend festgelegt – es gibt kein CLI-Banner, das diese angibt. Wenn etwas unerwartet verläuft (ein GPU-Pfad, der auf einen Fallback zurückgreift, ein OOM, ein Denoiser, der nicht geladen werden kann), wird dies im Backend-Protokoll für diese Sitzung angezeigt:

| Plattform | Speicherort des Protokolls |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (eine Datei pro Start) |
| **Linux, über CLI gestartetes Backend** | ebenfalls `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Live-Fortschritt

Während eines Laufs zeigt die CLI den Live-Fortschritt pro Thread an (Erkennen, Analysieren, Verarbeiten, Exportieren), der über Server-Sent Events übertragen wird – ein praktischer Anhaltspunkt dafür, ob Thread 3 den Engpass darstellt. Siehe [Verarbeitungspipeline](processing-pipeline.md).

***

## Nächste Schritte

* [Verarbeitungspipeline](processing-pipeline.md) – Die Architektur der 4-Thread-Pipeline verstehen
* [NVIDIA Jetson-Handbuch](../linux/nvidia-jetson-guide.md) — Jetson-spezifische Bereitstellung und Optimierung
* [CLI: Befehlszeile](../CLI.md) — Das CLI-Handbuch
* [CLI-Referenz](../reference/cli-reference.md) — Vollständige Befehlsliste für Version 1.3.0
