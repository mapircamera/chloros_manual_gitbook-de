# Starten der Verarbeitung

Sobald Sie Ihre Bilder importiert, Ihre Kalibrierungsziele markiert und Ihre Projekteinstellungen konfiguriert haben, können Sie mit der Verarbeitung beginnen. Diese Seite führt Sie durch den Start der Chloros-Verarbeitungs-Pipeline.

## Checkliste für die Vorverarbeitung

Bevor Sie auf die Schaltfläche „Start“ klicken, überprüfen Sie, ob alles bereit ist:

* [ ] **Dateien importiert** – Alle Bilder werden im Dateibrowser angezeigt
* [ ] **Zielbilder markiert** – Spalte „Ziel“ für Kalibrierungsbilder markiert
* [ ] **Kameramodelle erkannt** – Spalte „Kameramodell“ zeigt die richtigen Kameras an
* [ ] **Einstellungen konfiguriert** – Projekteinstellungen überprüft und angepasst
* [ ] **Indizes ausgewählt** – Gewünschte Multispektralindizes hinzugefügt (falls erforderlich)
* [ ] **Exportformat ausgewählt** – Für Ihren Workflow geeignetes Ausgabeformat

{% hint style="info" %}
**Tipp**: Klicken Sie im Dateibrowser durch einige Bilder, um vor der Verarbeitung zu überprüfen, ob sie korrekt geladen wurden.
{% endhint %}

***

## Starten der Verarbeitung

### Die Start-Schaltfläche finden

Die Start-/Wiedergabe-Schaltfläche befindet sich in der oberen Kopfzeile von Chloros:

* Position: Oben in der Mitte des Fensters
* Symbol: **Wiedergabe-/Start-Schaltfläche** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Status: Die Schaltfläche ist aktiviert (hell), wenn die Verarbeitung bereit ist

### Zum Starten klicken

1. Klicken Sie auf die **Wiedergabe-/Start-Schaltfläche** in der oberen Kopfzeile
2. Die Verarbeitung beginnt sofort
3. Die Schaltfläche wird während der Verarbeitung deaktiviert (ausgegraut)
4. Der Fortschrittsbalken wird aktualisiert und zeigt den Verarbeitungsstatus an

{% hint style="success" %}
**Verarbeitung gestartet**: Nach dem Klicken führt Chloros automatisch alle Verarbeitungsschritte durch – Zielerkennung, Debayering, Kalibrierung, Indexberechnung und Export.
{% endhint %}

***

## Verarbeitungsmodi verstehen

Chloros arbeitet je nach Ihrer Lizenz in zwei verschiedenen Verarbeitungsmodi:

### Kostenloser Modus (Sequentielle Verarbeitung)

**Für alle Benutzer verfügbar**

**So funktioniert es:**

* Verarbeitet Bilder nacheinander, sequenziell
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
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) wählt automatisch die optimale Strategie für Ihre Hardware aus
* GPU-Beschleunigung (CUDA) mit NVIDIA-Grafikkarten (Desktop und Jetson)
* Skalierbar von einem Jetson Nano (1 Worker) bis zu einem Desktop mit 12 GB+ GPU (3–4 Worker)

**Der Fortschrittsbalken zeigt 4 Phasen an** (entsprechend den 4 Pipeline-Threads):

1. **Erkennen** (Thread 1) – Kalibrierungsziele finden
2. **Analysieren** (Thread 2) – Bildmetadaten untersuchen und Kalibrierung berechnen
3. **Kalibrieren** (Thread 3) – GPU-Debayering, Vignettenkorrektur, Indexberechnung
4. **Exportieren** (Thread 4) – Speichern der verarbeiteten Bilder und Indizes**Interaktion mit dem Fortschrittsbalken:*** **Bewegen Sie den Mauszeiger** über den Balken, um das detaillierte Dropdown-Fenster mit den 4 Phasen anzuzeigen
* **Klicken Sie** auf den Fortschrittsbalken, um das Dropdown-Fenster an Ort und Stelle zu fixieren
* **Klicken Sie erneut**, um die Fixierung aufzuheben und das Fenster auszublenden**Verarbeitungszeit:**

* Deutlich schneller als im kostenlosen Modus
* Skaliert mit der Anzahl der CPU-Kerne
* GPU-Beschleunigung verbessert die Geschwindigkeit zusätzlich

{% hint style="info" %}
**Chloros+ Geschwindigkeit**: Bei großen Datensätzen kann die parallele Verarbeitung 5- bis 10-mal schneller sein als im sequenziellen Modus. Ein Projekt mit 500 Bildern, das im kostenlosen Modus 2 Stunden dauert, kann mit Chloros+ in 15 bis 20 Minuten abgeschlossen werden.
{% endhint %}

***

## Was passiert während der Verarbeitung

### Stufe 1: Zielerkennung

**Was Chloros tut:**

* Scannt markierte Zielbilder (oder alle Bilder, wenn keine markiert sind)
* Identifiziert die 4 Kalibrierungsfelder in jedem Ziel
* Extrahiert Reflexionswerte aus den Zielfeldern
* Zeichnet Zeitstempel der Ziele für die Kalibrierungsplanung auf

**Dauer:** 1–30 Sekunden (bei markierten Zielen), 5–30+ Minuten (unmarkiert)

### Stufe 2: Debayering (RAW-Konvertierung)

**Was Chloros macht:**

* Konvertiert RAW-Daten im Bayer-Muster in vollständige RGB-Bilder
* Wendet einen hochwertigen Demosaicing-Algorithmus an
* Bewahrt maximale Bildqualität und Detailgenauigkeit

**Dauer:** Variiert je nach Bildanzahl und CPU-Geschwindigkeit

### Stufe 3: Kalibrierung

**Was Chloros tut:*** **Vignettierungskorrektur**: Beseitigt die Randverdunkelung durch das Objektiv
* **Reflexionskalibrierung**: Normalisiert anhand von Zielreflexionswerten
* Wendet Korrekturen auf alle Bänder/Kanäle an
* Verwendet für jedes Bild das passende Kalibrierungsziel basierend auf dem Zeitstempel

**Dauer:** Der Großteil der Verarbeitungszeit

### Stufe 4: Indexberechnung

**Was Chloros tut:**

* Berechnet konfigurierte multispektrale Indizes (NDVI, NDRE usw.)
* Wendet Bandberechnungen auf kalibrierte Bilder an
* Erzeugt Indexbilder für jeden ausgewählten Index

**Dauer:** Einige Sekunden pro Bild

### Stufe 5: Export

**Was Chloros tut:**

* Speichert kalibrierte Bilder im ausgewählten Format
* Exportiert Indexbilder mit konfigurierten LUT-Farben
* Schreibt Dateien in Unterordner nach Kameramodell
* Behält ursprüngliche Dateinamen mit Suffixen bei

**Dauer:** Variiert je nach Exportformat und Dateigröße***

## Verarbeitungsverhalten

### Automatische Verarbeitungs-Pipeline

Nach dem Start läuft die gesamte Pipeline automatisch ab:

* Keine Benutzerinteraktion erforderlich
* Alle konfigurierten Schritte werden nacheinander ausgeführt
* Fortschrittsanzeigen werden in Echtzeit angezeigt

### Computerauslastung während der Verarbeitung

**Freier Modus:**

* Relativ geringe CPU-Auslastung (Single-Thread)
* Der Computer bleibt für andere Aufgaben reaktionsfähig
* Es ist unbedenklich, Chloros zu minimieren und in anderen Anwendungen zu arbeiten

**Chloros+ Parallelmodus:**

* Hohe CPU-Auslastung (Multi-Thread, bis zu 16 Kerne)
* Mit GPU-Beschleunigung: Hohe GPU-Auslastung
* Der Computer reagiert während der Verarbeitung möglicherweise weniger schnell
* Vermeiden Sie es, andere CPU-intensive Aufgaben zu starten

{% hint style="warning" %}
**Leistungstipp**: Für beste Chloros+-Leistung schließen Sie andere Anwendungen und lassen Sie Chloros die vollen Systemressourcen nutzen.
{% endhint %}

### Die Verarbeitung kann nicht angehalten werden

**Wichtige Einschränkungen:**

* Einmal gestartete Verarbeitung kann nicht angehalten werden
* Sie können die Verarbeitung abbrechen, der Fortschritt geht jedoch verloren
* Teilresultate werden nicht gespeichert
* Bei Abbruch muss von vorne begonnen werden

**Planungstipp:** Bei sehr großen Projekten sollten Sie eine Verarbeitung in Stapeln oder die Verwendung von CLI für eine bessere Kontrolle in Betracht ziehen.***

## Überwachung Ihrer Verarbeitung

Während der Verarbeitung können Sie:

* **Den Fortschrittsbalken beobachten** – Den Gesamtfortschrittsprozentsatz anzeigen
* **Die aktuelle Phase anzeigen** – Erkennen, Analysieren, Kalibrieren oder Exportieren
* **Registerkarte „Protokoll“ überprüfen** – Detaillierte Verarbeitungsmeldungen und Warnungen anzeigen
* **Vorschau der fertigen Bilder** – Einige Exportdateien werden möglicherweise bereits während der Verarbeitung angezeigt

Ausführliche Informationen zur Überwachung finden Sie unter [Überwachung der Verarbeitung](monitoring-the-processing.md).

***

## Abbruch der Verarbeitung

Wenn Sie die Verarbeitung stoppen müssen:

### So brechen Sie die Verarbeitung ab

1. Suchen Sie die **Schaltfläche „Stopp/Abbrechen“** (ersetzt während der Verarbeitung die Schaltfläche „Start“)
2. Klicken Sie auf die Schaltfläche „Stopp“
3. Die Verarbeitung wird sofort angehalten
4. Teilweise Ergebnisse werden verworfen

### Wann Sie abbrechen sollten

**Gültige Gründe für einen Abbruch:**

* Sie haben festgestellt, dass falsche Einstellungen verwendet wurden
* Sie haben vergessen, Zielbilder zu markieren
* Es wurden falsche Bilder importiert
* Das System läuft zu langsam oder reagiert nicht

**Nach dem Abbrechen:**

* Überprüfen Sie die Ergebnisse und beheben Sie etwaige Probleme
* Passen Sie die Einstellungen nach Bedarf an
* Starten Sie die Verarbeitung von vorne
* Für ein optimales Ergebnis schließen Sie Chloros vollständig und starten Sie

{% hint style="warning" %}
**Keine Teilergebnisse**: Durch das Abbrechen werden alle bisherigen Fortschritte verworfen. Chloros speichert keine teilweise verarbeiteten Bilder.
{% endhint %}

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
* Komplexität des Exportformats

### Grobe Schätzungen (Chloros+, 12-MP-Bilder, moderne CPU)

| Bildanzahl | Free-Modus | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 Bilder   | 15–20 Min. | 5–8 Min.        | 3–5 Min.        |
| 100 Bilder  | 30–40 Min. | 10–15 Min.      | 5–8 Min.        |
| 200 Bilder  | 1–1,5 Std. | 20–30 Min.      | 10–15 Min.      |
| 500 Bilder  | 2–3 Std.   | 45–60 Min.      | 20–30 Min.      |
| 1000 Bilder | 4–6 Std.   | 1,5–2 Std.      | 40–60 Min.      |

{% hint style="info" %}
**Erster Durchlauf**: Die anfängliche Verarbeitung kann länger dauern, da Chloros Caches und Profile erstellt. Die nachfolgende Verarbeitung ähnlicher Datensätze erfolgt schneller.
{% endhint %}

***

## Häufige Probleme beim Start

### Start-Schaltfläche deaktiviert (ausgegraut)

**Mögliche Ursachen:**

* Keine Bilder importiert
* Backend noch nicht vollständig gestartet
* Vorherige Verarbeitung läuft noch
* Projekt noch nicht vollständig geladen

**Lösungen:**

1. Warten, bis das Backend vollständig initialisiert ist (überprüfen Sie das Symbol im Hauptmenü)
2. Überprüfen Sie, ob die Bilder im Dateibrowser importiert sind
3. Starten Sie Chloros neu, wenn die Schaltfläche weiterhin deaktiviert ist
4. Überprüfen Sie das Debug-Protokoll auf Fehlermeldungen

### Die Verarbeitung startet und schlägt dann sofort fehl

**Mögliche Ursachen:**

* Keine gültigen Bilder im Projekt
* Beschädigte Bilddateien
* Unzureichender Speicherplatz
* Unzureichender Arbeitsspeicher (RAM)

**Lösungen:**

1. Debug-Protokoll <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> auf Fehlermeldungen
2. Verfügbaren Speicherplatz überprüfen
3. Versuchen Sie, eine kleinere Teilmenge der Bilder zu verarbeiten
4. Überprüfen Sie, ob die Bilder beschädigt sind

### Warnung „Keine Ziele erkannt“

**Mögliche Ursachen:**

* Vergessen, Zielbilder zu markieren
* Zielbilder enthalten keine sichtbaren Ziele
* Einstellungen für die Zielerkennung zu streng

**Lösungen:**

1. Lesen Sie [Auswahl von Zielbildern](choosing-target-images.md)
2. Markieren Sie die entsprechenden Bilder in der Spalte „Ziel“
3. Überprüfen Sie, ob die Ziele in den markierten Bildern sichtbar sind
4. Passen Sie die Einstellungen für die Zielerkennung bei Bedarf an

***

## Tipps für eine erfolgreiche Verarbeitung

### Vor dem Start

1. **Testen Sie zunächst mit einer kleinen Teilmenge** – Verarbeiten Sie 10–20 Bilder, um die Einstellungen zu überprüfen
2. **Überprüfen Sie den verfügbaren Speicherplatz** – Stellen Sie sicher, dass 2–3-mal so viel Speicherplatz wie die Datensatzgröße frei ist
3. **Schließen Sie nicht benötigte Anwendungen** – Geben Sie Systemressourcen frei
4. **Überprüfen Sie die Zielbilder** – Zeigen Sie eine Vorschau der markierten Ziele an, um die Qualität sicherzustellen
5. **Projekt speichern** – Das Projekt wird automatisch gespeichert, es empfiehlt sich jedoch, es manuell zu speichern

### Während der Verarbeitung

1. **System-Ruhezustand vermeiden** – Energiesparmodi deaktivieren
2. **Chloros im Vordergrund lassen** – Oder zumindest in der Taskleiste sichtbar halten
3. **Fortschritt gelegentlich überwachen** – Auf Warnungen oder Fehler prüfen
4. **Keine anderen ressourcenintensiven Anwendungen laden** – Insbesondere im Parallelmodus von Chloros+

### Chloros+ GPU-Beschleunigung

Bei Verwendung der NVIDIA-GPU-Beschleunigung:

1. NVIDIA-Treiber auf die neueste Version aktualisieren
2. Sicherstellen, dass die GPU über mindestens 4 GB VRAM verfügt
3. Schließen Sie GPU-intensive Anwendungen (Spiele, Videobearbeitung)
4. Überwachen Sie die GPU-Temperatur (sorgen Sie für ausreichende Kühlung)

***

## Nächste Schritte

Sobald die Verarbeitung begonnen hat:

1. **Überwachen Sie den Fortschritt** – Siehe [Überwachen der Verarbeitung](monitoring-the-processing.md)
2. **Warten Sie auf den Abschluss** – Die Verarbeitung läuft automatisch ab
3. **Überprüfen Sie die Ergebnisse** – Siehe [Abschluss der Verarbeitung](finishing-the-processing.md)

Informationen dazu, was während der Verarbeitung zu tun ist, finden Sie unter [Überwachung der Verarbeitung](monitoring-the-processing.md).
