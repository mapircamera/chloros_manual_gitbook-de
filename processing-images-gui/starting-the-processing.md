# Starten der Verarbeitung

Sobald Sie Ihre Bilder importiert, Ihre Kalibrierungsmarken markiert und Ihre Projekteinstellungen konfiguriert haben, können Sie mit der Verarbeitung beginnen. Diese Seite führt Sie durch den Start der Chloros-Verarbeitungs-Pipeline.

## Checkliste für die Vorverarbeitung

Bevor Sie auf die Schaltfläche „Start“ klicken, vergewissern Sie sich, dass alles bereit ist:

* [ ] **Dateien importiert** – Alle Bilder werden im Dateibrowser angezeigt
* [ ] **Zielbilder markiert** – Spalte „Ziel“ auf Kalibrierungsbilder überprüft (oder eine `.daq`-Aufnahme für LATTICE importiert)
* [ ] **Kameramodelle erkannt** – Spalte „Kameramodell“ zeigt die richtigen Kameras an
* [ ] **Einstellungen konfiguriert** – Projekteinstellungen überprüft und angepasst
* [ ] **Indizes ausgewählt** – Gewünschte multispektrale Indizes hinzugefügt (falls erforderlich)
* [ ] **Exportformat ausgewählt** – Für Ihren Workflow geeignetes Ausgabeformat

{% hint style="info" %}
**Tipp**: Klicken Sie im Dateibrowser durch einige Bilder, um vor der Verarbeitung zu überprüfen, ob diese korrekt geladen wurden.
{% endhint %}

***

## Starten der Verarbeitung

### Die Schaltfläche „Start“ finden

Die Start-/Wiedergabetaste befindet sich in der oberen Kopfzeile von Chloros:

* Position: Oben in der Mitte des Fensters
* Symbol: **Wiedergabe-/Starttaste** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Status: Die Schaltfläche ist aktiviert (hell), sobald die Verarbeitung beginnen kann

### Zum Starten anklicken

1. Klicken Sie auf die **Wiedergabe-/Start-Schaltfläche** in der oberen Kopfzeile
2. Die Verarbeitung beginnt sofort
3. Die Schaltfläche wird während der Verarbeitung zu einer **Stopp**-Schaltfläche
4. Der Fortschrittsbalken wird aktualisiert und zeigt den Verarbeitungsstatus an

{% hint style="success" %}
**Verarbeitung gestartet**: Nach dem Klicken führt Chloros automatisch alle Verarbeitungsschritte durch – Zielerkennung, Debayering, Kalibrierung, Indexberechnung und Export. Es erkennt automatisch, ob es sich bei Ihrem Projekt um Survey3, LATTICE oder eine Mischung handelt, und wendet für jede Kamera die richtige Pipeline an.
{% endhint %}

***

## Verarbeitungsmodi verstehen

Chloros arbeitet je nach Ihrer Lizenz in zwei verschiedenen Verarbeitungsmodi:

### Kostenloser Modus (sequenzielle Verarbeitung)

**Für alle Benutzer verfügbar**

**So funktioniert es:**

* Verarbeitet Bilder nacheinander, eines nach dem anderen
* Single-Thread-Betrieb
* Geringerer Speicherbedarf

**Der Fortschrittsbalken zeigt 2 Phasen an:**

1.**Zielerkennung** – Suche nach Kalibrierungszielen
2. **Verarbeitung** – Anwenden der Kalibrierung und Exportieren der Bilder**Verarbeitungszeit:**

* Deutlich langsamer als der parallele Modus von Chloros+
* Geeignet für kleine bis mittlere Datensätze (&lt; 200 Bilder)

### Chloros+-Modus (parallele Verarbeitung)

**Erfordert eine Chloros+-Lizenz**

**So funktioniert es:**

* Verarbeitet mehrere Bilder gleichzeitig mithilfe einer [4-Thread-Verarbeitungspipeline](../processing-architecture/processing-pipeline.md)
* Die [dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md) wählt zu Beginn der Ausführung automatisch die optimale Strategie für Ihre Hardware aus
* GPU-Beschleunigung (CUDA) mit NVIDIA-Grafikkarten (Desktop und Jetson)
* **Die Anzahl der Worker passt sich der Hardware an**: Bei GPU-Strategien werden**1–4 Worker gleichzeitig** ausgeführt (skaliert nach VRAM – ein Jetson mit wenig Speicher führt 1 Worker aus, eine Desktop-GPU mit 12 GB oder mehr führt bis zu 4 Worker aus); Systeme, die ausschließlich mit CPU arbeiten, führen einen Worker pro physikalischem Kern aus, abzüglich eines**Der Fortschrittsbalken zeigt 4 Phasen an** (entsprechend den 4 Pipeline-Threads):

1. **Erkennung** (Thread 1) – Kalibrierungsziele finden
2. **Analyse** (Thread 2) – Bildmetadaten untersuchen und Kalibrierung berechnen
3. **Kalibrierung** (Thread 3) – Debayering, Vignettenkorrektur, Kalibrierung, Indexberechnung
4. **Export** (Thread 4) – Speichern der verarbeiteten Bilder und Indizes**Interaktion mit dem Fortschrittsbalken:*** **Bewegen Sie den Mauszeiger** über den Balken, um das detaillierte 4-stufige Dropdown-Fenster anzuzeigen
* **Klicken Sie** auf den Fortschrittsbalken, um das Dropdown-Fenster an Ort und Stelle zu fixieren
* **Klicken Sie erneut**, um die Fixierung aufzuheben und das Fenster auszublenden**Verarbeitungszeit:**

* Deutlich schneller als im kostenlosen Modus
* GPU-Beschleunigung verbessert die Geschwindigkeit zusätzlich

{% hint style="info" %}
**Chloros+ Geschwindigkeit**: Bei großen Datensätzen kann die parallele Verarbeitung 5- bis 10-mal schneller sein als im sequenziellen Modus. Ein Projekt mit 500 Bildern, das im kostenlosen Modus 2 Stunden dauert, kann mit Chloros+ in 15–20 Minuten abgeschlossen werden.
{% endhint %}

***

## Was geschieht während der Verarbeitung?

### Stufe 1: Zielerkennung

**Was Chloros tut:**

* Scannt die Bilder, die Sie in der Spalte „Ziel“ angekreuzt haben (alle Bilder, wenn keine angekreuzt sind)
* Identifiziert die Kalibrierungsfelder in jedem Ziel
* Extrahiert die Reflexionswerte aus den Zielfeldern
* Zeichnet die Zeitstempel der Ziele für die Kalibrierungsplanung auf

**Dauer:** 1–30 Sekunden (bei markierten Zielen), 5–30+ Minuten (bei unmarkierten Zielen)

### Stufe 2: Debayering (RAW-Konvertierung)

**Funktionen von Chloros:**

* Konvertiert RAW-Daten im Bayer-Muster in vollständige 3-Kanal-Bilder (LATTICE-Mono-Module bleiben einkanalig – bei ihnen wird das Debayering übersprungen und im Protokoll vermerkt)
* Wendet den ausgewählten Demosaicing-Algorithmus an
* Erhält maximale Bildqualität und Detailgenauigkeit

**Dauer:** Variiert je nach Bildanzahl und CPU-/GPU-Geschwindigkeit

### Stufe 3: Kalibrierung

**Was Chloros leistet:*** **Vignettierungskorrektur**: Beseitigt die Randverdunkelung des Objektivs
* **Reflexionskalibrierung**: Normalisiert anhand von Zielreflexionswerten und/oder DAQ-Downwelling-Daten
* Wendet Korrekturen auf alle Bänder/Kanäle an
* Verwendet für jedes Bild die passende Kalibrierungsreferenz basierend auf dem Zeitstempel

**Dauer:** Der Großteil der Verarbeitungszeit

### Stufe 4: Indexberechnung

**Funktionen von Chloros:**

* Berechnet konfigurierte Multispektralindizes (NDVI, NDRE usw.)
* Wendet Bandberechnungen auf kalibrierte Bilder an
* Erzeugt Indexbilder für jeden ausgewählten Index

**Dauer:** Einige Sekunden pro Bild

### Schritt 5: Export

**Funktionen von Chloros:**

* Speichert die verarbeiteten Bilder im ausgewählten Format
* **LATTICE-Fan-out**: Jeder rohe LATTICE-Frame wird in einem Durchgang als jedes aktivierte Produkt exportiert – debayert, Vorschau, Radiance (immer float32), Reflektanz
* Schreibt Dateien in den Projekt-Ausgabebaum: `<project>/<camera>/<format>/<Product>_Images/`
* **Behält den Quelldateinamen bei** – der Ordner identifiziert das Produkt, es wird kein Suffix hinzugefügt**Dauer:** Variiert je nach Exportformat und Dateigröße***

## Verarbeitungsverhalten

### Automatische Verarbeitungspipeline

Nach dem Start läuft die gesamte Pipeline automatisch ab:

* Keine Benutzerinteraktion erforderlich
* Alle konfigurierten Schritte werden nacheinander ausgeführt
* Fortschrittsanzeigen werden in Echtzeit angezeigt
* Exportierte Dateien werden nach Fertigstellung auf die Festplatte geschrieben – Sie können fertige Ergebnisse bereits öffnen, während der Lauf noch andauert

### Computerauslastung während der Verarbeitung

**Freier Modus:**

* Relativ geringe CPU-Auslastung (Single-Thread)
* Der Computer bleibt für andere Aufgaben reaktionsfähig
* Sie können Chloros gefahrlos minimieren und in anderen Anwendungen arbeiten

**Chloros+ Parallelmodus:**

* Hohe CPU-Auslastung über den gesamten Worker-Pool der Strategie hinweg
* Mit GPU-Beschleunigung: Hohe GPU-Auslastung
* Der Computer reagiert während der Verarbeitung möglicherweise weniger schnell
* Vermeiden Sie es, andere CPU-intensive Aufgaben zu starten

{% hint style="warning" %}
**Leistungstipp**: Für eine optimale Leistung von Chloros+ schließen Sie andere Anwendungen und lassen Sie Chloros die gesamten Systemressourcen nutzen.
{% endhint %}

### Die Verarbeitung kann nicht angehalten werden (ein sauberes Beenden ist jedoch möglich)

* Einmal gestartete Verarbeitung kann nicht angehalten und später fortgesetzt werden
* Ein Klick auf **Stopp** beendet den Lauf beim ersten Klick sauber
* Produkte, die bereits vor dem Stopp exportiert wurden, bleiben auf der Festplatte erhalten
* Ein gestoppter Lauf gibt ehrlich an, was abgeschlossen wurde (siehe die Zeilen „`[RUN-SUMMARY]`“ im Protokoll)
* Ein neuer Lauf startet die Pipeline von Anfang an

**Planungstipp:** Bei sehr großen Projekten sollten Sie eine Verarbeitung in Stapeln oder die Verwendung von CLI in Betracht ziehen, um eine bessere Kontrolle zu erhalten.***

## Überwachung Ihrer Verarbeitung

Während der Verarbeitung können Sie:

* **Den Fortschrittsbalken beobachten** – Den Gesamtfortschritt in Prozent anzeigen
* **Die aktuelle Phase anzeigen** – Erkennen, Analysieren, Kalibrieren oder Exportieren
* **Die Registerkarte „Protokoll“ überprüfen** – Detaillierte Verarbeitungsmeldungen und Warnungen anzeigen
* **Eine Vorschau der fertigen Bilder anzeigen** – Exportierte Dateien erscheinen während der Verarbeitung auf der Festplatte

Ausführliche Informationen zur Überwachung finden Sie unter [Überwachung der Verarbeitung](monitoring-the-processing.md).

***

## Verarbeitungsprozess anhalten

Wenn Sie die Verarbeitung anhalten müssen:

### So halten Sie den Prozess an

1. Suchen Sie die **Schaltfläche „Stopp“** (ersetzt während der Verarbeitung die Schaltfläche „Start“)
2. Klicken Sie einmal darauf – in der Leiste wird **„Wird angehalten...“** angezeigt, während das gerade verarbeitete Bild fertiggestellt wird
3. Der Durchlauf endet in einem endgültigen Stoppzustand und das Protokoll gibt einen genauen Überblick über die abgeschlossenen Schritte

### Wann man den Vorgang abbrechen sollte

**Gültige Gründe für einen Abbruch:**

* Es wurde festgestellt, dass falsche Einstellungen verwendet wurden
* Das Markieren der Zielbilder wurde vergessen
* Es wurden falsche Bilder importiert
* Das System läuft zu langsam oder reagiert nicht mehr

**Nach dem Abbruch:**

* Vor dem Abbruch exportierte Produkte bleiben auf der Festplatte erhalten
* Überprüfen und beheben Sie etwaige Probleme, passen Sie die Einstellungen nach Bedarf an
* Starten Sie die Verarbeitung neu – der Durchlauf beginnt von vorne

***

## Geschätzte Verarbeitungszeiten

Die tatsächliche Verarbeitungszeit variiert stark in Abhängigkeit von:

* Anzahl der Bilder
* Bildauflösung
* Eingabeformat (RAW vs. JPG)
* Verarbeitungsmodus (Free vs. Chloros+)
* CPU-Geschwindigkeit und Anzahl der Kerne
* Verfügbarkeit einer GPU (nur Chloros+)
* Anzahl der zu berechnenden Indizes
* Anzahl der aktivierten Exportprodukte (LATTICE)

### Grobe Schätzungen (Chloros+, 12-MP-Bilder, moderne CPU)

| Anzahl der Bilder | Kostenloser Modus | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 Bilder   | 15–20 Min. | 5–8 Min.        | 3–5 Min.        |
| 100 Bilder  | 30–40 Min. | 10–15 Min.      | 5–8 Min.        |
| 200 Bilder  | 1–1,5 Std. | 20–30 Min.      | 10–15 Min.      |
| 500 Bilder  | 2–3 Std.   | 45–60 Min.      | 20–30 Min.      |
| 1.000 Bilder | 4–6 Std.   | 1,5–2 Std.      | 40–60 Min.      |

{% hint style="info" %}
**Erster Durchlauf**: Die anfängliche Verarbeitung kann länger dauern, da Chloros Caches und Profile erstellt. Die nachfolgende Verarbeitung ähnlicher Datensätze erfolgt schneller.
{% endhint %}

***

## Häufige Probleme beim Start

### Start-Schaltfläche deaktiviert (ausgegraut)

**Mögliche Ursachen:**

* Es wurden keine Bilder importiert
* Das Backend ist noch nicht vollständig gestartet
* Die vorherige Verarbeitung läuft noch
* Das Projekt wurde noch nicht vollständig geladen

**Lösungen:**

1. Warten Sie, bis das Backend vollständig initialisiert ist (überprüfen Sie das Symbol im Hauptmenü)
2. Vergewissern Sie sich, dass die Bilder im Dateibrowser importiert sind
3. Starten Sie Chloros neu, wenn die Schaltfläche weiterhin deaktiviert ist
4. Überprüfen Sie das Debug-Protokoll auf Fehlermeldungen

### Die Verarbeitung startet und bricht dann sofort ab

**Mögliche Ursachen:**

* Keine gültigen Bilder im Projekt
* Beschädigte Bilddateien
* Unzureichender Speicherplatz
* Unzureichender Arbeitsspeicher (RAM)

**Lösungen:**

1. Überprüfen Sie das Debug-Protokoll unter <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> auf Fehlermeldungen
2. Überprüfen Sie den verfügbaren Speicherplatz
3. Versuchen Sie, eine kleinere Teilmenge der Bilder zu verarbeiten
4. Stellen Sie sicher, dass die Bilder nicht beschädigt sind

### Der Lauf wird beendet, es werden jedoch keine Bilder geschrieben

Ein Lauf, bei dem Bildprodukte angefordert, aber keine geschrieben wurden, wird als **Fehler und nicht als Erfolg** gewertet – Chloros meldet dies deutlich:

* Das GUI-Protokoll gibt Hinweise mit der Meldung „`[RUN-SUMMARY]`“ aus, die die wahrscheinliche Ursache nennen – keine Bilder importiert, kein Ziel erkannt oder alle angeforderten Produkte als nicht anwendbar übersprungen (z. B. bei der Anforderung von Radianz-/Reflexionsdaten von Kameras, die ausschließlich RGB unterstützen)
* Das CLI-Äquivalent (`chloros-cli process`) gibt `Processing finished but wrote no image products.` aus und **beendet den Vorgang mit einem Wert ungleich Null**, sodass Skripte dies erkennen können
* Ein absichtlich nur auf Metadaten beschränkter Durchlauf (alle Exportprodukte deaktiviert, keine Indizes) gilt weiterhin als erfolgreich

Die vollständige Semantik findest du in der [CLI-Referenz](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Warnung „Keine Ziele erkannt“

**Mögliche Ursachen:**

* Vergessen, Zielbilder zu markieren
* Zielbilder enthalten keine sichtbaren Ziele
* Einstellungen zur Zielerkennung sind zu streng

**Lösungen:**

1. Lesen Sie [Auswahl von Zielbildern](choosing-target-images.md)
2. Markieren Sie die entsprechenden Bilder in der Spalte „Ziel“
3. Vergewissern Sie sich, dass die Ziele in den markierten Bildern sichtbar sind
4. Passen Sie die Einstellungen für die Zielerkennung bei Bedarf an

***

## Tipps für eine erfolgreiche Verarbeitung

### Vor dem Start

1. **Testen Sie zunächst mit einer kleinen Teilmenge** – Verarbeiten Sie 10–20 Bilder, um die Einstellungen zu überprüfen
2. **Verfügbaren Speicherplatz prüfen** – Stellen Sie sicher, dass das 2–3-Fache der Datensatzgröße frei ist (mehr, wenn alle LATTICE-Produkte aktiviert sind)
3. **Nicht benötigte Anwendungen schließen** – Systemressourcen freigeben
4. **Zielbilder überprüfen** – Markierte Ziele in der Vorschau anzeigen, um die Qualität sicherzustellen
5. **Projekt speichern** – Das Projekt wird automatisch gespeichert, es empfiehlt sich jedoch, es manuell zu speichern

### Während der Verarbeitung

1. **Vermeiden Sie den Ruhezustand des Systems** – Deaktivieren Sie Energiesparmodi
2. **Halten Sie Chloros im Vordergrund** – Oder lassen Sie es zumindest in der Taskleiste sichtbar
3. **Überwachen Sie den Fortschritt gelegentlich** – Achten Sie auf Warnungen oder Fehler
4. **Laden Sie keine anderen ressourcenintensiven Anwendungen** – Insbesondere im Parallelmodus von Chloros+

### Chloros+ GPU-Beschleunigung

Bei Verwendung der NVIDIA-GPU-Beschleunigung:

1. NVIDIA-Treiber auf die neueste Version aktualisieren
2. Sicherstellen, dass die GPU über mindestens 4 GB VRAM verfügt (mindestens 7 GB für gleichzeitiges „Texture Aware“-Debayering)
3. GPU-intensive Anwendungen schließen (Spiele, Videobearbeitung)
4. GPU-Temperatur überwachen (für ausreichende Kühlung sorgen)

***

## Nächste Schritte

Sobald die Verarbeitung begonnen hat:

1. **Überwachen Sie den Fortschritt** – siehe [Überwachen der Verarbeitung](monitoring-the-processing.md)
2. **Warten Sie, bis der Vorgang abgeschlossen ist** – die Verarbeitung läuft automatisch
3. **Ergebnisse überprüfen** – Siehe [Abschluss der Verarbeitung](finishing-the-processing.md)

Informationen dazu, was während der Verarbeitung zu tun ist, finden Sie unter [Überwachung der Verarbeitung](monitoring-the-processing.md).
