# Überwachung der Verarbeitung

Sobald die Verarbeitung begonnen hat, bietet Chloros verschiedene Möglichkeiten, den Fortschritt zu überwachen, auf Probleme zu prüfen und zu verstehen, was mit Ihrem Datensatz geschieht. Auf dieser Seite wird erläutert, wie Sie Ihre Verarbeitung verfolgen und die von Chloros bereitgestellten Informationen interpretieren können.

## Übersicht über den Fortschrittsbalken

Der Fortschrittsbalken in der oberen Kopfzeile zeigt den Verarbeitungsstatus in Echtzeit und den Fertigstellungsgrad in Prozent an.

### Fortschrittsbalken im kostenlosen Modus

Für Benutzer ohne Chloros+-Lizenz:

**2-stufige Fortschrittsanzeige:**

1.**Zielerkennung** – Suchen von Kalibrierungszielen in Bildern
2. **Verarbeitung** – Anwenden von Korrekturen und Exportieren**Die Fortschrittsanzeige zeigt:**

* Gesamtfortschrittsprozentsatz (0–100 %)
* Name der aktuellen Phase
* Einfache horizontale Balkenvisualisierung

### Chloros+-Fortschrittsanzeige

Für Benutzer mit einer Chloros+-Lizenz:

**4-stufige Fortschrittsanzeige:**

1.**Erkennen** – Kalibrierungsziele finden
2. **Analysieren** – Bilder untersuchen und Pipeline vorbereiten
3. **Kalibrieren** – Vignetten- und Reflexionskorrekturen anwenden
4. **Exportieren** – Bearbeitete Dateien speichern**Interaktive Funktionen:*** **Bewegen Sie den Mauszeiger über** den Fortschrittsbalken, um das erweiterte 4-Stufen-Fenster anzuzeigen
* **Klicken Sie** auf den Fortschrittsbalken, um das erweiterte Fenster zu fixieren
* **Klicken Sie erneut**, um die Fixierung aufzuheben; das Fenster wird automatisch ausgeblendet, wenn Sie den Mauszeiger entfernen
* Jede Stufe zeigt den individuellen Fortschritt an (0–100 %)

***

## Die einzelnen Verarbeitungsstufen verstehen

{% hint style="info" %}
**Pipeline-Architektur**: Diese 4 GUI-Phasen entsprechen der [4-Thread-Verarbeitungspipeline](../processing-architecture/processing-pipeline.md). Auf Systemen mit GPU-Beschleunigung profitiert Thread 3 (Kalibrierung) von der [dynamischen Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md), die die Verarbeitung für Ihre spezifische Hardware optimiert.
{% endhint %}

### Stufe 1: Erkennung (Zielerkennung)

**Was passiert:**

* Chloros scannt Bilder, die mit dem Kontrollkästchen „Ziel“ markiert sind
* Computer-Vision-Algorithmen identifizieren die 4 Kalibrierungsfelder
* Reflexionswerte werden aus jedem Feld extrahiert
* Ziel-Zeitstempel werden für eine korrekte Kalibrierungsplanung aufgezeichnet

**Dauer:**

* Mit markierten Zielen: 10–60 Sekunden
* Ohne markierte Ziele: 5–30+ Minuten (scannt alle Bilder)

**Fortschrittsanzeige:**

* Erkennung: 0 % → 100 %
* Anzahl der gescannten Bilder
* Anzahl der gefundenen Ziele

**Was zu beachten ist:**

* Sollte schnell abgeschlossen sein, wenn die Ziele ordnungsgemäß markiert sind
* Wenn es zu lange dauert, sind die Ziele möglicherweise nicht markiert
* Überprüfen Sie das Debug-Protokoll auf „Ziel gefunden“-Meldungen

### Stufe 2: Analyse

**Was geschieht:**

* Lesen der EXIF-Metadaten der Bilder (Zeitstempel, Belichtungseinstellungen)
* Festlegung der Kalibrierungsstrategie basierend auf den Zeitstempeln der Ziele
* Organisation der Bildverarbeitungswarteschlange
* Vorbereitung der Worker für die parallele Verarbeitung (nur Chloros+)

**Dauer:** 5–30 Sekunden**Fortschrittsanzeige:**

* Analyse: 0 % → 100 %
* Schnelle Phase, wird in der Regel zügig abgeschlossen

**Worauf zu achten ist:**

* Der Fortschritt sollte ohne Unterbrechungen stetig voranschreiten
* Warnungen zu fehlenden Metadaten werden im Debug-Protokoll angezeigt

### Phase 3: Kalibrierung

**Was passiert:*** **Debayering**: Konvertierung des RAW-Bayer-Musters in 3 Kanäle
* **Vignettierungskorrektur**: Beseitigung der Randverdunkelung des Objektivs
* **Reflexionskalibrierung**: Normalisierung anhand von Zielwerten
* **Indexberechnung**: Berechnung multispektraler Indizes
* Verarbeitung jedes Bildes durch die gesamte Pipeline

**Dauer:** Der Großteil der gesamten Verarbeitungszeit (60–80 %)**Fortschrittsanzeige:**

* Kalibrierung: 0 % → 100 %
* Aktuell verarbeitetes Bild
* Fertige Bilder / Gesamtzahl der Bilder

**Verarbeitungsverhalten:*** **Freier Modus**: Verarbeitet nacheinander jeweils ein Bild
* **Chloros+ Modus**: Verarbeitet bis zu 16 Bilder gleichzeitig
* **GPU-Beschleunigung**: Beschleunigt diese Phase erheblich**Zu beachten:**

* Gleichmäßiger Fortschritt bei der Bildanzahl
* Debug-Protokoll auf Abschlussmeldungen pro Bild überprüfen
* Warnungen zu Bildqualitäts- oder Kalibrierungsproblemen

### Phase 4: Exportieren

**Was geschieht:**

* Schreiben der kalibrierten Bilder im ausgewählten Format auf die Festplatte
* Exportieren von Multispektral-Indexbildern mit LUT-Farben
* Erstellen von Unterordnern für Kameramodelle
* Beibehalten der ursprünglichen Dateinamen mit entsprechenden Suffixen

**Dauer:** 10–20 % der gesamten Verarbeitungszeit**Fortschrittsanzeige:**

* Exportieren: 0 % → 100 %
* Dateien werden geschrieben
* Exportformat und Zielort

**Worauf zu achten ist:**

* Warnungen zum Festplattenspeicher
* Fehler beim Schreiben von Dateien
* Abschluss aller konfigurierten Ausgaben

***

## Registerkarte „Debug-Protokoll“

Das Debug-Protokoll liefert detaillierte Informationen zum Verarbeitungsfortschritt und zu eventuell aufgetretenen Problemen.

### Auf das Debug-Protokoll zugreifen

1. Klicken Sie auf das Symbol **Debug-Protokoll** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> in der linken Seitenleiste
2. Das Protokollfenster wird geöffnet und zeigt Verarbeitungsmeldungen in Echtzeit an
3. Es wird automatisch gescrollt, um die neuesten Meldungen anzuzeigen

### Protokollmeldungen verstehen

#### Informationsmeldungen (weiß/grau)

Normale Verarbeitungsaktualisierungen:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Warnmeldungen (gelb)

Nicht kritische Probleme, die die Verarbeitung nicht unterbrechen:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Maßnahme:** Überprüfen Sie die Warnungen nach der Verarbeitung, unterbrechen Sie diese jedoch nicht

#### Fehlermeldungen (Red)

Kritische Probleme, die zum Fehlschlagen der Verarbeitung führen können:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Maßnahme:** Verarbeitung anhalten, Fehler beheben, neu starten

### Häufige Protokollmeldungen

| Meldung                          | Bedeutung                                | Erforderliche Maßnahme                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Ziel in \[Dateiname] erkannt“ | Kalibrierungsziel erfolgreich gefunden  | Keine – normal                                         |
| „Bild X von Y wird verarbeitet“        | Aktueller Fortschrittsbericht                | Keine – normal                                         |
| „Keine Ziele gefunden“               | Keine Kalibrierungsziele erkannt        | Zielbilder markieren oder Reflexionskalibrierung deaktivieren |
| „Nicht genügend Speicherplatz“        | Nicht genügend Speicherplatz für die Ausgabe          | Speicherplatz freigeben                                    |
| „Beschädigte Datei wird übersprungen“        | Bilddatei ist beschädigt                  | Datei erneut von der SD-Karte kopieren                             |
| „PPK-Daten angewendet“               | GPS-Korrekturen aus der .daq-Datei angewendet | Keine – normal                                         |

### Protokoll-Daten kopieren

So kopieren Sie das Protokoll zur Fehlerbehebung oder für den Support:

1. Öffnen Sie das Debug-Log-Fenster
2. Klicken Sie auf die Schaltfläche **„Protokoll kopieren“** (oder Rechtsklick → Alles auswählen)
3. Fügen Sie den Text in eine Textdatei oder eine E-Mail ein
4. Senden Sie ihn bei Bedarf an den MAPIR-Support

***

## Überwachung der Systemressourcen

### CPU-Auslastung

**Free-Modus:**

* 1 CPU-Kern bei ~100 %
* Andere Kerne im Leerlauf oder verfügbar
* System bleibt reaktionsfähig

**Chloros+ Parallel-Modus:**

* Mehrere Kerne bei 80–100 % (bis zu 16 Kerne)
* Hohe Gesamt-CPU-Auslastung
* Das System reagiert möglicherweise weniger schnell

**Zur Überwachung:**

* Windows Task-Manager (Strg+Umschalt+Esc)
* Registerkarte „Leistung“ → Abschnitt „CPU“
* Suchen Sie nach den Prozessen „Chloros“ oder „chloros-backend“

### Speicherauslastung (RAM)

**Typische Nutzung:**

* Kleine Projekte (&lt; 100 Bilder): 2–4 GB
* Mittlere Projekte (100–500 Bilder): 4–8 GB
* Große Projekte (500+ Bilder): 8–16 GB
* Der Parallelmodus von Chloros+ verbraucht mehr RAM

**Bei geringem Speicherplatz:**

* Verarbeiten Sie kleinere Stapel
* Schließen Sie andere Anwendungen
* Erweitern Sie den Arbeitsspeicher, wenn Sie regelmäßig große Datensätze verarbeiten

### GPU-Auslastung (Chloros+ mit CUDA)

Wenn die GPU-Beschleunigung aktiviert ist:

* Die NVIDIA-GPU weist eine hohe Auslastung auf (60–90 %)
* Die VRAM-Auslastung steigt (erfordert 4 GB+ VRAM)
* Die Kalibrierungsphase ist deutlich schneller

**Zur Überwachung:**

* NVIDIA-Symbol in der Taskleiste
* Task-Manager → Leistung → GPU
* GPU-Z oder ein ähnliches Überwachungstool

### Festplatten-E/A

**Zu erwarten:**

* Hohe Leseaktivität während der Analysephase
* Hohe Schreibaktivität während der Exportphase
* SSD deutlich schneller als HDD

**Leistungstipp:**

* Verwenden Sie nach Möglichkeit eine SSD für den Projektordner
* Vermeiden Sie Netzlaufwerke für große Datensätze
* Stellen Sie sicher, dass die Festplatte nicht fast voll ist (beeinträchtigt die Schreibgeschwindigkeit)

***

## Erkennen von Problemen während der Verarbeitung

### Warnzeichen

**Fortschritt stockt (keine Veränderung seit mehr als 5 Minuten):**

* Debug-Protokoll auf Fehler überprüfen
* Verfügbaren Speicherplatz überprüfen
* Im Task-Manager sicherstellen, dass Chloros ausgeführt wird

**Fehlermeldungen erscheinen häufig:**

* Verarbeitung anhalten und Fehler überprüfen
* Häufige Ursachen: Speicherplatz, beschädigte Dateien, Speicherprobleme
* Siehe Abschnitt „Fehlerbehebung“ weiter unten

**Das System reagiert nicht mehr:**

* Der Parallelmodus von Chloros+ beansprucht zu viele Ressourcen
* Erwägen Sie, die Anzahl der gleichzeitig ausgeführten Aufgaben zu reduzieren oder die Hardware aufzurüsten
* Der Freimodus ist weniger ressourcenintensiv

### Wann die Verarbeitung abgebrochen werden sollte

Brechen Sie die Verarbeitung ab, wenn Folgendes auftritt:

* ❌ Fehlermeldungen wie „Festplatte voll“ oder „Datei kann nicht geschrieben werden“
* ❌ Wiederholte Fehler wegen beschädigter Bilddateien
* ❌ System ist komplett eingefroren (reagiert nicht)
* ❌ Es wurde festgestellt, dass falsche Einstellungen konfiguriert wurden
* ❌ Falsche Bilder wurden importiert

**So brechen Sie die Verarbeitung ab:**

1. Klicken Sie auf die Schaltfläche**„Stopp/Abbrechen“** (ersetzt die Schaltfläche „Start“)
2. Die Verarbeitung wird angehalten, der Fortschritt geht verloren
3. Beheben Sie die Probleme und starten Sie von vorne

***

## Fehlerbehebung während der Verarbeitung

### Die Verarbeitung ist sehr langsam

**Mögliche Ursachen:**

* Nicht markierte Zielbilder (alle Bilder werden gescannt)
* HDD statt SSD-Speicher
* Unzureichende Systemressourcen
* Viele Indizes konfiguriert
* Zugriff auf Netzlaufwerk

**Lösungen:**

1. Wenn gerade gestartet und in der Erkennungsphase: Abbrechen, Ziele markieren, neu starten
2. Für die Zukunft: SSD verwenden, Indizes reduzieren, Hardware aufrüsten
3. CLI für die Stapelverarbeitung großer Datensätze in Betracht ziehen

### Warnungen zum „Speicherplatz“

**Lösungen:**

1. Sofort Speicherplatz freigeben
2. Projekt auf ein Laufwerk mit mehr Speicherplatz verschieben
3. Reduzieren Sie die Anzahl der zu exportierenden Indizes
4. Verwenden Sie das JPG-Format anstelle von TIFF (kleinere Dateien)

### Häufige Meldungen über „beschädigte Dateien“

**Lösungen:**

1. Kopieren Sie die Bilder erneut von der SD-Karte, um die Integrität sicherzustellen
2. Überprüfen Sie die SD-Karte auf Fehler
3. Entfernen Sie beschädigte Dateien aus dem Projekt
4. Setzen Sie die Verarbeitung der verbleibenden Bilder fort

### Überhitzung / Drosselung des Systems

**Lösungen:**

1. Für ausreichende Belüftung sorgen
2. Staub aus den Lüftungsschlitzen des Computers entfernen
3. Verarbeitungslast reduzieren (Free-Modus anstelle von Chloros+ verwenden)
4. Verarbeitung zu kühleren Tageszeiten durchführen

***

## Benachrichtigung über Abschluss der Verarbeitung

Wenn die Verarbeitung abgeschlossen ist:

* Der Fortschrittsbalken erreicht 100 %
* Die Meldung **„Verarbeitung abgeschlossen“** erscheint im Debug-Protokoll
* Die Schaltfläche „Start“ wird wieder aktiv
* Alle Ausgabedateien befinden sich im Unterordner des Kameramodells

***

## Nächste Schritte

Sobald die Verarbeitung abgeschlossen ist:

1. **Ergebnisse überprüfen** – Siehe [Abschluss der Verarbeitung](finishing-the-processing.md)
2. **Ausgabeordner überprüfen** – Vergewissern Sie sich, dass alle Dateien korrekt exportiert wurden
3. **Debug-Protokoll überprüfen** – Achten Sie auf Warnungen oder Fehler
4. **Vorschau der verarbeiteten Bilder** – Verwenden Sie den Bildbetrachter oder eine externe Software

Informationen zum Überprüfen und Verwenden Ihrer verarbeiteten Ergebnisse finden Sie unter [Abschluss der Verarbeitung](finishing-the-processing.md).
