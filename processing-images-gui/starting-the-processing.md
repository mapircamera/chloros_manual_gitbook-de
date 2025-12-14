# Starten der Verarbeitung

Nachdem Sie Ihre Bilder importiert, Ihre Kalibrierungsziele markiert und Ihre Projekteinstellungen konfiguriert haben, können Sie mit der Verarbeitung beginnen. Diese Seite führt Sie durch die Initiierung der Chloros-Verarbeitungs-Pipeline.

## Checkliste vor der Verarbeitung

Bevor Sie auf die Schaltfläche „Start“ klicken, überprüfen Sie, ob alles bereit ist:

* [ ] **Dateien importiert** – Alle Bilder werden im Dateibrowser angezeigt
* [ ] **Zielbilder markiert** – Zielspalte für Kalibrierungsbilder überprüft
* [ ] **Kameramodelle erkannt** – In der Spalte „Kameramodell“ werden die richtigen Kameras angezeigt
* [ ] **Einstellungen konfiguriert** – Projekteinstellungen überprüft und angepasst
* [ ] **Indizes ausgewählt** – Gewünschte multispektrale Indizes hinzugefügt (falls erforderlich)
* [ ] **Exportformat ausgewählt** – Für Ihren Workflow geeignetes Ausgabeformat

{% hint style=&quot;info&quot; %}
**Tipp**: Klicken Sie sich durch einige Bilder im Dateibrowser, um zu überprüfen, ob sie vor der Verarbeitung korrekt geladen wurden.
{% endhint %}

***

## Starten der Verarbeitung

### Suchen Sie die Startschaltfläche

Die Schaltfläche „Start/Wiedergabe“ befindet sich in der oberen Kopfzeile von Chloros:

* Position: Oben in der Mitte des Fensters
* Symbol: **Schaltfläche „Wiedergabe/Start“** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Status: Die Schaltfläche ist aktiviert (hell), wenn die Verarbeitung bereit ist

### Zum Starten klicken

1. Klicken Sie auf die **Schaltfläche „Wiedergabe/Start“** in der oberen Kopfzeile
2. Die Verarbeitung beginnt sofort
3. Die Schaltfläche wird während der Verarbeitung deaktiviert (ausgegraut)
4. Der Fortschrittsbalken wird aktualisiert und zeigt den Verarbeitungsstatus an

{% hint style=&quot;success&quot; %}
**Verarbeitung gestartet**: Nach dem Klicken übernimmt Chloros automatisch alle Verarbeitungsschritte – Zielerkennung, Debayering, Kalibrierung, Indexberechnung und Export.
{% endhint %}

***

## Verarbeitungsmodi verstehen

Chloros arbeitet je nach Ihrer Lizenz in zwei verschiedenen Verarbeitungsmodi:

### Freier Modus (sequenzielle Verarbeitung)

**Für alle Benutzer verfügbar**

**So funktioniert es:**

* Verarbeitet Bilder nacheinander, sequenziell
* Single-Thread-Betrieb
* Geringerer Speicherverbrauch

**Der Fortschrittsbalken zeigt 2 Phasen an:**

1. **Zielerkennung** – Scannen nach Kalibrierungszielen
2. **Verarbeitung** – Anwenden der Kalibrierung und Exportieren der Bilder

**Verarbeitungszeit:**

* Deutlich langsamer als der parallele Modus von Chloros+
* Geeignet für kleine bis mittlere Datensätze (&lt; 200 Bilder)

### Modus Chloros+ (Parallelverarbeitung)

**Erfordert eine Lizenz für Chloros+**

**So funktioniert es:**

* Verarbeitet mehrere Bilder gleichzeitig
* Multithread-Betrieb (bis zu 16 parallele Worker)
* Nutzt mehrere CPU-Kerne
* Optionale GPU-Beschleunigung (CUDA) mit NVIDIA-Grafikkarten

**Der Fortschrittsbalken zeigt 4 Phasen an:**

1. **Erkennen** – Kalibrierungsziele finden
2. **Analysieren** – Bildmetadaten untersuchen und Pipeline vorbereiten
3. **Kalibrieren** – Korrekturen und Kalibrierungen anwenden
4. **Exportieren** – Verarbeitete Bilder und Indizes speichern

**Interaktion mit der Fortschrittsanzeige:**

* **Bewegen Sie die Maus** über die Leiste, um ein detailliertes Dropdown-Fenster mit den 4 Phasen anzuzeigen
* **Klicken Sie** auf die Fortschrittsanzeige, um das Dropdown-Fenster an dieser Stelle einzufrieren
* **Klicken Sie erneut**, um das Fenster wieder zu aktivieren und auszublenden.

**Verarbeitungszeit:**

* Deutlich schneller als im freien Modus.
* Skaliert mit der Anzahl der CPU-Kerne.
* GPU-Beschleunigung verbessert die Geschwindigkeit zusätzlich.

{% hint style=&quot;info&quot; %}
**Chloros+ Geschwindigkeit**: Die parallele Verarbeitung kann bei großen Datensätzen 5-10 Mal schneller sein als der sequentielle Modus. Ein Projekt mit 500 Bildern, das im kostenlosen Modus 2 Stunden dauert, kann mit Chloros+ in 15-20 Minuten abgeschlossen werden.
{% endhint %}

***

## Was passiert während der Verarbeitung?

### Stufe 1: Zielerkennung

**Was Chloros tut:**

* Scannt markierte Zielbilder (oder alle Bilder, wenn keine markiert sind)
* Identifiziert die 4 Kalibrierungsfelder in jedem Ziel
* Extrahiert Reflexionswerte aus den Zielfeldern
* Zeichnet Zielzeitstempel für die Kalibrierungsplanung auf

**Dauer:** 1–30 Sekunden (mit markierten Zielen), 5–30+ Minuten (unmarkiert)

### Stufe 2: Debayering (RAW-Konvertierung)

**Was Chloros leistet:**

* Konvertiert RAW-Bayer-Musterdaten in vollständige RGB-Bilder
* Wendet einen hochwertigen Demosaicing-Algorithmus an
* Erhält maximale Bildqualität und Detailgenauigkeit

**Dauer:** Variiert je nach Bildanzahl und CPU-Geschwindigkeit

### Stufe 3: Kalibrierung

**Was Chloros leistet:**

* **Vignettierungskorrektur**: Entfernt die Verdunkelung der Linsen an den Rändern
* **Reflexionskalibrierung**: Normalisiert anhand von Zielreflexionswerten
* Wendet Korrekturen auf alle Bänder/Kanäle an
* Verwendet für jedes Bild das geeignete Kalibrierungsziel basierend auf dem Zeitstempel

**Dauer:** Großteil der Verarbeitungszeit

### Stufe 4: Indexberechnung

**Was Chloros tut:**

* Berechnet konfigurierte multispektrale Indizes (NDVI, NDRE usw.)
* Wendet Bandmathematik auf kalibrierte Bilder an
* Erzeugt Indexbilder für jeden ausgewählten Index

**Dauer:** Einige Sekunden pro Bild

### Stufe 5: Export

**Was Chloros tut:**

* Speichert kalibrierte Bilder im ausgewählten Format
* Exportiert Indexbilder mit konfigurierten LUT-Farben
* Schreibt Dateien in Unterordner des Kameramodells
* Behält ursprüngliche Dateinamen mit Suffixen bei

**Dauer:** Variiert je nach Exportformat und Dateigröße

***

## Verarbeitungsverhalten

### Automatische Verarbeitungs-Pipeline

Nach dem Start läuft die gesamte Pipeline automatisch ab:

* Keine Benutzerinteraktion erforderlich
* Alle konfigurierten Schritte werden nacheinander ausgeführt
* Fortschrittsanzeige in Echtzeit

### Computernutzung während der Verarbeitung

**Freier Modus:**

* Relativ geringe CPU-Auslastung (Single-Threaded)
* Der Computer bleibt für andere Aufgaben reaktionsfähig
* Es ist sicher, Chloros zu minimieren und in anderen Anwendungen zu arbeiten

**Chloros+ Parallelmodus:**

* Hohe CPU-Auslastung (Multi-Threaded, bis zu 16 Kerne)
* Mit GPU-Beschleunigung: Hohe GPU-Auslastung
* Der Computer reagiert während der Verarbeitung möglicherweise weniger schnell.
* Vermeiden Sie es, andere CPU-intensive Aufgaben zu starten.

{% Hinweis style=&quot;warning&quot; %}
**Leistungstipp**: Um die beste Leistung von Chloros+ zu erzielen, schließen Sie andere Anwendungen und lassen Sie Chloros die gesamten Systemressourcen nutzen.
{% endhint %}

### Die Verarbeitung kann nicht angehalten werden

**Wichtige Einschränkungen:**

* Nach dem Start kann die Verarbeitung nicht angehalten werden.
* Sie können die Verarbeitung abbrechen, aber der Fortschritt geht verloren.
* Teilweise Ergebnisse werden nicht gespeichert.
* Bei Abbruch muss von vorne begonnen werden.

**Planungstipp:** Bei sehr großen Projekten sollten Sie eine Verarbeitung in Stapeln oder die Verwendung von CLI in Betracht ziehen, um eine bessere Kontrolle zu erzielen.

***

## Überwachen Ihrer Verarbeitung

Während der Verarbeitung können Sie:

* **Den Fortschrittsbalken beobachten** – Sehen Sie den Gesamtfortschritt in Prozent.
* **Die aktuelle Phase anzeigen** – Erkennen, Analysieren, Kalibrieren oder Exportieren.
* **Die Registerkarte „Protokoll“ überprüfen** – Sehen Sie detaillierte Verarbeitungsmeldungen und Warnungen.
* **Eine Vorschau der fertigen Bilder anzeigen** – Einige Exportdateien werden möglicherweise während der Verarbeitung angezeigt.

Ausführliche Informationen zur Überwachung finden Sie unter [Überwachen der Verarbeitung](monitoring-the-processing.md).

***

## Abbrechen der Verarbeitung

Wenn Sie die Verarbeitung stoppen müssen:

### So brechen Sie den Vorgang ab

1. Suchen Sie die **Schaltfläche „Stopp/Abbrechen“** (ersetzt während der Verarbeitung die Schaltfläche „Start“).
2. Klicken Sie auf die Schaltfläche „Stopp“.
3. Die Verarbeitung wird sofort angehalten.
4. Teilweise Ergebnisse werden verworfen.

### Wann sollte abgebrochen werden?

**Gültige Gründe für einen Abbruch:**

* Es wurde festgestellt, dass falsche Einstellungen verwendet wurden.
* Das Markieren der Zielbilder wurde vergessen.
* Es wurden falsche Bilder importiert.
* Das System läuft zu langsam oder reagiert nicht.

**Nach dem Abbruch:**

* Überprüfen und beheben Sie alle Probleme.
* Passen Sie die Einstellungen nach Bedarf an.
* Starten Sie die Verarbeitung von vorne.
* Für ein optimales Erlebnis schließen Sie Chloros vollständig und starten Sie es neu.

{% hint style=&quot;warning&quot; %}
**Keine Teilergebnisse**: Durch das Abbrechen werden alle Fortschritte verworfen. Chloros speichert keine teilweise verarbeiteten Bilder.
{% endhint %}

***

## Geschätzte Verarbeitungszeit

Die tatsächliche Verarbeitungszeit hängt stark von folgenden Faktoren ab:

* Anzahl der Bilder
* Bildauflösung
* Eingabeformat (RAW oder JPG)
* Verarbeitungsmodus (Free oder Chloros+)
* CPU-Geschwindigkeit und Anzahl der Kerne
* Verfügbarkeit der GPU (nur Chloros+)
* Anzahl der zu berechnenden Indizes
* Komplexität des Exportformats

### Grobe Schätzungen (Chloros+, 12-MP-Bilder, moderne CPU)

| Anzahl der Bilder | Kostenloser Modus | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 Bilder   | 15–20 Min. | 5–8 Min.        | 3–5 Min.        |
| 100 Bilder  | 30–40 Min. | 10–15 Min.      | 5–8 Min.        |
| 200 Bilder  | 1–1,5 Std. | 20–30 Min.      | 10–15 Min.      |
| 500 Bilder  | 2–3 Std.   | 45–60 Min.      | 20–30 Min.      |
| 1000 Bilder | 4–6 Std.   | 1,5–2 Std.      | 40–60 Min.      |

{% hint style=&quot;info&quot; %}
**Erster Durchlauf**: Die erste Verarbeitung kann länger dauern, da Chloros Caches und Profile erstellt. Die nachfolgende Verarbeitung ähnlicher Datensätze erfolgt schneller.
{% endhint %}

***

## Häufige Probleme beim Start

### Start-Schaltfläche deaktiviert (ausgegraut)

**Mögliche Ursachen:**

* Keine Bilder importiert
* Backend nicht vollständig gestartet
* Vorherige Verarbeitung läuft noch
* Projekt nicht vollständig geladen

**Lösungen:**

1. Warten Sie, bis das Backend vollständig initialisiert ist (überprüfen Sie das Symbol im Hauptmenü).
2. Überprüfen Sie im Dateibrowser, ob die Bilder importiert wurden.
3. Starten Sie Chloros neu, wenn die Schaltfläche weiterhin deaktiviert ist.
4. Überprüfen Sie das Debug-Protokoll auf Fehlermeldungen.

### Die Verarbeitung beginnt und schlägt dann sofort fehl

**Mögliche Ursachen:**

* Keine gültigen Bilder im Projekt
* Beschädigte Bilddateien
* Unzureichender Speicherplatz
* Unzureichender Arbeitsspeicher (RAM)

**Lösungen:**

1. Überprüfen Sie das Debug-Protokoll <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> auf Fehlermeldungen
2. Verfügbaren Speicherplatz überprüfen
3. Versuchen Sie, eine kleinere Teilmenge von Bildern zu verarbeiten
4. Überprüfen Sie, ob die Bilder nicht beschädigt sind

### Warnung „Keine Ziele erkannt”

**Mögliche Ursachen:**

* Zielbilder wurden nicht markiert
* Zielbilder enthalten keine sichtbaren Ziele
* Einstellungen für die Zielerkennung sind zu streng

**Lösungen:**

1. Überprüfen Sie die [Auswahl von Zielbildern](choosing-target-images.md).
2. Markieren Sie die entsprechenden Bilder in der Spalte „Ziel”.
3. Überprüfen Sie, ob die Ziele in den markierten Bildern sichtbar sind.
4. Passen Sie die Einstellungen für die Zielerkennung bei Bedarf an.

***

## Tipps für eine erfolgreiche Verarbeitung

### Vor dem Start

1. **Testen Sie zunächst mit einer kleinen Teilmenge** – Verarbeiten Sie 10 bis 20 Bilder, um die Einstellungen zu überprüfen.
2. **Überprüfen Sie den verfügbaren Speicherplatz** – Stellen Sie sicher, dass 2 bis 3 Mal so viel Speicherplatz wie die Datensatzgröße frei ist.
3. **Schließen Sie nicht benötigte Anwendungen** – Geben Sie Systemressourcen frei.
4. **Überprüfen Sie die Zielbilder** – Zeigen Sie eine Vorschau der markierten Ziele an, um die Qualität sicherzustellen.
5. **Projekt speichern** – Das Projekt wird automatisch gespeichert, aber es empfiehlt sich, es manuell zu speichern.

### Während der Verarbeitung

1. **Vermeiden Sie den Ruhezustand des Systems** – Deaktivieren Sie Energiesparmodi.
2. **Halten Sie Chloros im Vordergrund** – Oder zumindest in der Taskleiste sichtbar.
3. **Überwachen Sie gelegentlich den Fortschritt** – Achten Sie auf Warnungen oder Fehler.
4. **Keine anderen ressourcenintensiven Anwendungen laden** – insbesondere im Parallelmodus von Chloros+.

### Chloros+ GPU-Beschleunigung

Bei Verwendung der NVIDIA-GPU-Beschleunigung:

1. NVIDIA-Treiber auf die neueste Version aktualisieren.
2. Sicherstellen, dass die GPU über mindestens 4 GB VRAM verfügt.
3. Schließen Sie GPU-intensive Anwendungen (Spiele, Videobearbeitung).
4. Überwachen Sie die GPU-Temperatur (stellen Sie eine ausreichende Kühlung sicher).

***

## Nächste Schritte

Sobald die Verarbeitung begonnen hat:

1. **Überwachen Sie den Fortschritt** – siehe [Überwachen der Verarbeitung](monitoring-the-processing.md)
2. **Warten Sie, bis der Vorgang abgeschlossen ist** – Die Verarbeitung läuft automatisch.
3. **Überprüfen Sie die Ergebnisse** – Siehe [Beenden der Verarbeitung](finishing-the-processing.md).

Informationen dazu, was während der Verarbeitung zu tun ist, finden Sie unter [Überwachen der Verarbeitung](monitoring-the-processing.md).
