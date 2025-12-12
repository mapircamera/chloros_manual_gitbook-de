# Überwachung der Verarbeitung

Sobald die Verarbeitung begonnen hat, bietet Chloros mehrere Möglichkeiten, den Fortschritt zu überwachen, auf Probleme zu überprüfen und zu verstehen, was mit Ihrem Datensatz geschieht. Auf dieser Seite wird erläutert, wie Sie Ihre Verarbeitung verfolgen und die von Chloros bereitgestellten Informationen interpretieren können.

## Übersicht über den Fortschrittsbalken

Der Fortschrittsbalken in der oberen Kopfzeile zeigt den Verarbeitungsstatus und den Prozentsatz der Fertigstellung in Echtzeit an.

### Fortschrittsbalken im freien Modus

Für Benutzer ohne Chloros+-Lizenz:

**2-stufige Fortschrittsanzeige:**

1. **Zielerkennung** – Auffinden von Kalibrierungszielen in Bildern
2. **Verarbeitung** – Anwenden von Korrekturen und Exportieren

**Die Fortschrittsanzeige zeigt:**

* Gesamtfertigstellungsgrad (0–100 %)
* Name der aktuellen Stufe
* Einfache horizontale Balkenvisualisierung

### Fortschrittsanzeige für Chloros+

Für Benutzer mit einer Chloros+-Lizenz:

**4-stufige Fortschrittsanzeige:**

1. **Erkennen** – Kalibrierungsziele finden
2. **Analysieren** – Bilder untersuchen und Pipeline vorbereiten
3. **Kalibrieren** – Vignetten- und Reflektionskorrekturen anwenden
4. **Exportieren** – Verarbeitete Dateien speichern

**Interaktive Funktionen:**

* **Bewegen Sie den Mauszeiger über** den Fortschrittsbalken, um das erweiterte 4-Stufen-Fenster anzuzeigen
* **Klicken Sie** auf die Fortschrittsleiste, um das erweiterte Fenster anzuhalten/anzuheften.
* **Klicken Sie erneut**, um die Anhaltung aufzuheben und das Fenster beim Verlassen mit der Maus automatisch auszublenden.
* Jede Stufe zeigt den individuellen Fortschritt (0–100 %) an.

***

## Die einzelnen Verarbeitungsstufen verstehen

### Stufe 1: Erkennen (Zielerkennung)

**Was passiert:**

* Chloros scannt Bilder, die mit dem Kontrollkästchen „Ziel“ markiert sind
* Computervisionsalgorithmen identifizieren die 4 Kalibrierungsfelder
* Aus jedem Feld werden Reflexionswerte extrahiert
* Zielzeitstempel werden für eine ordnungsgemäße Kalibrierungsplanung aufgezeichnet

**Dauer:**

* Mit markierten Zielen: 10–60 Sekunden
* Ohne markierte Ziele: 5–30+ Minuten (scannt alle Bilder)

**Fortschrittsanzeige:**

* Erkennung: 0 % → 100 %
* Anzahl der gescannten Bilder
* Anzahl der gefundenen Ziele

**Was zu beachten ist:**

* Sollte schnell abgeschlossen sein, wenn die Ziele ordnungsgemäß markiert sind.
* Wenn es zu lange dauert, sind die Ziele möglicherweise nicht markiert.
* Überprüfen Sie das Debug-Protokoll auf Meldungen „Ziel gefunden“.

### Stufe 2: Analyse

**Was passiert:**

* Lesen der EXIF-Metadaten der Bilder (Zeitstempel, Belichtungseinstellungen)
* Festlegen der Kalibrierungsstrategie basierend auf den Zeitstempeln der Ziele
* Organisieren der Bildverarbeitungswarteschlange
* Vorbereiten der parallelen Verarbeitungsprozesse (nur Chloros+)

**Dauer:** 5–30 Sekunden

**Fortschrittsanzeige:**

* Analyse: 0 % → 100 %
* Schnelle Stufe, in der Regel schnell abgeschlossen

**Zu beachten:**

* Sollte ohne Unterbrechungen stetig voranschreiten
* Warnungen zu fehlenden Metadaten werden im Debug-Protokoll angezeigt

### Stufe 3: Kalibrierung

**Was geschieht:**

* **Debayering**: Konvertierung des RAW-Bayer-Musters in 3 Kanäle
* **Vignettierungskorrektur**: Entfernen der Verdunkelung an den Rändern des Objektivs
* **Reflexionskalibrierung**: Normalisierung mit Zielwerten
* **Indexberechnung**: Berechnung multispektraler Indizes
* Verarbeitung jedes Bildes durch die gesamte Pipeline

**Dauer:** Großteil der gesamten Verarbeitungszeit (60–80 %)

**Fortschrittsanzeige:**

* Kalibrierung: 0 % → 100 %
* Aktuelles Bild wird verarbeitet
* Fertige Bilder / Gesamtzahl der Bilder

**Verarbeitungsverhalten:**

* **Freier Modus**: Verarbeitet jeweils ein Bild nacheinander
* **Chloros+-Modus**: Verarbeitet bis zu 16 Bilder gleichzeitig
* **GPU-Beschleunigung**: Beschleunigt diese Phase erheblich

**Was zu beachten ist:**

* Gleichmäßiger Fortschritt durch die Bildanzahl
* Debug-Protokoll auf Meldungen zur Fertigstellung einzelner Bilder überprüfen
* Warnungen zu Bildqualität oder Kalibrierungsproblemen

### Phase 4: Exportieren

**Was geschieht:**

* Schreiben kalibrierter Bilder im ausgewählten Format auf die Festplatte
* Exportieren multispektraler Indexbilder mit LUT-Farben
* Erstellen von Unterordnern für Kameramodelle
* Beibehalten der ursprünglichen Dateinamen mit entsprechenden Suffixen

**Dauer:** 10–20 % der gesamten Verarbeitungszeit

**Fortschrittsanzeige:**

* Exportieren: 0 % → 100 %
* Dateien werden geschrieben
* Exportformat und Ziel

**Was zu beachten ist:**

* Warnungen zum Speicherplatz
* Fehler beim Schreiben von Dateien
* Abschluss aller konfigurierten Ausgaben

***

## Registerkarte „Debug-Protokoll“

Das Debug-Protokoll enthält detaillierte Informationen zum Verarbeitungsfortschritt und zu eventuell aufgetretenen Problemen.

### Auf das Debug-Protokoll zugreifen

1. Klicken Sie auf das Symbol **Debug-Protokoll** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> in der linken Seitenleiste.
2. Das Protokollfenster wird geöffnet und zeigt Echtzeit-Verarbeitungsmeldungen an.
3. Es wird automatisch gescrollt, um die neuesten Meldungen anzuzeigen.

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

**Maßnahme:** Überprüfen Sie die Warnungen nach der Verarbeitung, aber unterbrechen Sie den Vorgang nicht.

#### Fehlermeldungen (Red)

Kritische Probleme, die zu einem Fehlschlagen der Verarbeitung führen können:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Maßnahme:** Verarbeitung stoppen, Fehler beheben, neu starten.

### Allgemeine Protokollmeldungen

| Meldung                          | Bedeutung                                | Erforderliche Maßnahme                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Ziel in \[Dateiname] erkannt“ | Kalibrierungsziel erfolgreich gefunden  | Keine – normal                                         |
| „Bild X von Y wird verarbeitet“        | Aktuelle Fortschrittsanzeige                | Keine – normal                                         |
| „Keine Ziele gefunden“               | Keine Kalibrierungsziele erkannt        | Zielbilder markieren oder Reflektionskalibrierung deaktivieren |
| „Nicht genügend Speicherplatz”        | Nicht genügend Speicherplatz für die Ausgabe          | Speicherplatz freigeben                                    |
| „Beschädigte Datei überspringen”        | Bilddatei ist beschädigt                  | Datei erneut von SD-Karte kopieren                             |
| „PPK-Daten angewendet”               | GPS-Korrekturen aus .daq-Datei angewendet | Keine – normal                                         |

### Kopieren von Protokolldaten

So kopieren Sie das Protokoll zur Fehlerbehebung oder für den Support:

1. Öffnen Sie das Debug-Protokollfenster.
2. Klicken Sie auf die Schaltfläche **„Protokoll kopieren“** (oder klicken Sie mit der rechten Maustaste → Alle auswählen).
3. Fügen Sie den Inhalt in eine Textdatei oder E-Mail ein.
4. Senden Sie ihn bei Bedarf an den MAPIR-Support.

***

## Überwachung der Systemressourcen

### CPU-Auslastung

**Freier Modus:**

* 1 CPU-Kern bei ~100 %
* Andere Kerne im Leerlauf oder verfügbar
* Das System bleibt reaktionsfähig

**Chloros+ Parallelmodus:**

* Mehrere Kerne bei 80–100 % (bis zu 16 Kerne)
* Hohe Gesamt-CPU-Auslastung
* Das System reagiert möglicherweise weniger schnell.

**Überwachung:**

* Windows Task-Manager (Strg+Umschalt+Esc)
* Registerkarte „Leistung“ → Abschnitt „CPU“
* Suchen Sie nach den Prozessen „Chloros“ oder „chloros-backend“.

### Speicher (RAM)-Auslastung

**Typische Nutzung:**

* Kleine Projekte (&lt; 100 Bilder): 2–4 GB
* Mittlere Projekte (100–500 Bilder): 4–8 GB
* Große Projekte (über 500 Bilder): 8–16 GB
* Der parallele Modus Chloros+ benötigt mehr RAM

**Bei geringem Speicherplatz:**

* Verarbeiten Sie kleinere Stapel.
* Schließen Sie andere Anwendungen.
* Erweitern Sie den RAM, wenn Sie regelmäßig große Datensätze verarbeiten.

### GPU-Nutzung (Chloros+ mit CUDA)

Wenn die GPU-Beschleunigung aktiviert ist:

* Die NVIDIA-GPU weist eine hohe Auslastung auf (60–90 %).
* Die VRAM-Nutzung steigt (erfordert 4 GB+ VRAM)
* Die Kalibrierungsphase ist deutlich schneller

**Zur Überwachung:**

* NVIDIA-Symbol in der Taskleiste
* Task-Manager → Leistung → GPU
* GPU-Z oder ähnliches Überwachungstool

### Festplatten-E/A

**Was zu erwarten ist:**

* Hohe Festplattenlesevorgänge während der Analysephase
* Hohe Festplattenschreibvorgänge während der Exportphase
* SSD deutlich schneller als HDD

**Leistungstipp:**

* Verwenden Sie nach Möglichkeit eine SSD für den Projektordner
* Vermeiden Sie Netzlaufwerke für große Datensätze
* Stellen Sie sicher, dass die Festplatte nicht fast voll ist (beeinträchtigt die Schreibgeschwindigkeit)

***

## Probleme während der Verarbeitung erkennen

### Warnzeichen

**Fortschritt kommt zum Stillstand (keine Veränderung für mehr als 5 Minuten):**

* Debug-Protokoll auf Fehler überprüfen
* Verfügbaren Festplattenspeicher überprüfen
* Im Task-Manager überprüfen, ob Chloros ausgeführt wird

**Fehlermeldungen werden häufig angezeigt:**

* Verarbeitung stoppen und Fehler überprüfen
* Häufige Ursachen: Festplattenspeicher, beschädigte Dateien, Speicherprobleme
* Siehe Abschnitt „Fehlerbehebung“ weiter unten

**Das System reagiert nicht mehr:**

* Chloros+ Parallelmodus verbraucht zu viele Ressourcen.
* Reduzieren Sie die Anzahl der gleichzeitigen Aufgaben oder rüsten Sie die Hardware auf.
* Der Freie Modus ist weniger ressourcenintensiv.

### Wann sollte die Verarbeitung gestoppt werden?

Stoppen Sie die Verarbeitung, wenn Sie Folgendes sehen:

* ❌ Fehler „Festplatte voll” oder „Datei kann nicht geschrieben werden”
* ❌ Wiederholte Fehler aufgrund beschädigter Bilddateien
* ❌ System ist vollständig eingefroren (reagiert nicht)
* ❌ Es wurden falsche Einstellungen konfiguriert
* ❌ Falsche Bilder wurden importiert

**So stoppen Sie die Verarbeitung:**

1. Klicken Sie auf die **Schaltfläche „Stopp/Abbrechen“** (ersetzt die Schaltfläche „Start“)
2. Die Verarbeitung wird angehalten, der Fortschritt geht verloren
3. Beheben Sie die Probleme und starten Sie von vorne

***

## Fehlerbehebung während der Verarbeitung

### Die Verarbeitung ist sehr langsam

**Mögliche Ursachen:**

* Nicht markierte Zielbilder (alle Bilder werden gescannt)
* HDD- statt SSD-Speicher
* Unzureichende Systemressourcen
* Viele Indizes konfiguriert
* Zugriff auf Netzlaufwerk

**Lösungen:**

1. Wenn gerade erst gestartet und in der Erkennungsphase: Abbrechen, Ziele markieren, neu starten
2. Für die Zukunft: SSD verwenden, Indizes reduzieren, Hardware aufrüsten
3. CLI für die Stapelverarbeitung großer Datensätze in Betracht ziehen

### Warnungen zum „Festplattenspeicher“

**Lösungen:**

1. Sofort Festplattenspeicher freigeben
2. Projekt auf Laufwerk mit mehr Speicherplatz verschieben
3. Reduzieren Sie die Anzahl der zu exportierenden Indizes.
4. Verwenden Sie das JPG-Format anstelle von TIFF (kleinere Dateien).

### Häufige Meldungen „Beschädigte Datei”

**Lösungen:**

1. Kopieren Sie die Bilder erneut von der SD-Karte, um die Integrität sicherzustellen.
2. Testen Sie die SD-Karte auf Fehler.
3. Entfernen Sie beschädigte Dateien aus dem Projekt.
4. Setzen Sie die Verarbeitung der verbleibenden Bilder fort.

### Überhitzung/Drosselung des Systems

**Lösungen:**

1. Sorgen Sie für ausreichende Belüftung.
2. Entfernen Sie Staub aus den Lüftungsöffnungen des Computers.
3. Reduzieren Sie die Verarbeitungslast (verwenden Sie den freien Modus anstelle von Chloros+).
4. Führen Sie die Verarbeitung zu kühleren Tageszeiten durch.

***

## Benachrichtigung über abgeschlossene Verarbeitung

Wenn die Verarbeitung abgeschlossen ist:

* Der Fortschrittsbalken erreicht 100 %.
* Die Meldung **„Verarbeitung abgeschlossen“** wird im Debug-Protokoll angezeigt.
* Die Schaltfläche „Start“ ist wieder aktiviert.
* Alle Ausgabedateien befinden sich im Unterordner des Kameramodells.

***

## Nächste Schritte

Nach Abschluss der Verarbeitung:

1. **Überprüfen Sie die Ergebnisse** – siehe [Abschluss der Verarbeitung](finishing-the-processing.md).
2. **Überprüfen Sie den Ausgabeordner** – Vergewissern Sie sich, dass alle Dateien korrekt exportiert wurden.
3. **Überprüfen Sie das Debug-Protokoll** – Achten Sie auf Warnungen oder Fehler.
4. **Zeigen Sie eine Vorschau der verarbeiteten Bilder an** – Verwenden Sie den Bildbetrachter oder eine externe Software.

Informationen zum Überprüfen und Verwenden Ihrer verarbeiteten Ergebnisse finden Sie unter [Beenden der Verarbeitung](finishing-the-processing.md).
