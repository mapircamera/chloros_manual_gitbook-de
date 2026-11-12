# Starten der Verarbeitung

Sobald Sie Ihre Bilder importiert, Ihre Kalibrierungsziele markiert und Ihre Projekteinstellungen konfiguriert haben, können Sie mit der Verarbeitung beginnen. Diese Seite führt Sie durch den Start der Chloros-Verarbeitungs-Pipeline.

## Checkliste für die Vorverarbeitung

Bevor Sie auf die Schaltfläche „Start“ klicken, vergewissern Sie sich, dass alles bereit ist:

* [ ] **Dateien importiert** – Alle Bilder werden im Dateibrowser angezeigt
* [ ] **Zielbilder markiert** – Spalte „Ziel“ für Kalibrierungsbilder markiert (oder eine `.daq`-Aufnahme für LATTICE importiert)
* [ ] **Kameramodelle erkannt** – In der Spalte „Kameramodell“ werden die richtigen Kameras angezeigt
* [ ] **Einstellungen konfiguriert** – Projekteinstellungen überprüft und angepasst
* [ ] **Indizes ausgewählt** – Gewünschte multispektrale Indizes hinzugefügt (falls erforderlich)
* [ ] **Exportformat ausgewählt** – Für Ihren Arbeitsablauf geeignetes Ausgabeformat

{% hint style="info" %}
**Tipp**: Klicken Sie im Dateibrowser durch einige Bilder, um vor der Verarbeitung zu überprüfen, ob sie korrekt geladen wurden.
{% endhint %}

***

## Starten der Verarbeitung

### Die Start-Schaltfläche finden

Die Start-/Wiedergabe-Schaltfläche befindet sich in der oberen Kopfzeile von Chloros:

* Position: Oben in der Mitte des Fensters
* Symbol: **Wiedergabe-/Start-Schaltfläche** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Status: Die Schaltfläche ist aktiviert (hell), wenn die Verarbeitung bereit ist

### Zum Starten anklicken

1. Klicken Sie auf die **Wiedergabe-/Start-Schaltfläche** in der oberen Kopfzeile
2. Die Verarbeitung beginnt sofort
3. Während der Verarbeitung wird die Schaltfläche zu einer **Stopp**-Schaltfläche
4. Der Fortschrittsbalken wird aktualisiert und zeigt den Verarbeitungsstatus an

{% hint style="success" %}
**Verarbeitung gestartet**: Nach dem Klicken übernimmt Chloros automatisch alle Verarbeitungsschritte – Zielerkennung, Debayering, Kalibrierung, Indexberechnung und Export. Es erkennt automatisch, ob es sich bei Ihrem Projekt um Survey3, LATTICE oder eine Mischung handelt, und wendet für jede Kamera die richtige Pipeline an.
{% endhint %}

***

## Die Verarbeitungsmodi verstehen

Chloros arbeitet je nach Ihrer Lizenz in zwei verschiedenen Verarbeitungsmodi:

### Kostenloser Modus (sequenzielle Verarbeitung)

**Für alle Nutzer verfügbar**

**So funktioniert es:**

* Verarbeitet Bilder nacheinander, eines nach dem anderen
* Single-Thread-Betrieb
* Geringerer Speicherbedarf

**Der Fortschrittsbalken zeigt 2 Phasen an:**

1.**Zielerkennung** – Suche nach Kalibrierungszielen
2. **Verarbeitung** – Anwenden der Kalibrierung und Exportieren der Bilder**Verarbeitungszeit:**

* Deutlich langsamer als der Chloros+-Parallelmodus
* Geeignet für kleine bis mittlere Datensätze (< 200 Bilder)

### Chloros+-Modus (parallele Verarbeitung)

**Erfordert eine Chloros+-Lizenz**

**So funktioniert es:**

* Verarbeitet mehrere Bilder gleichzeitig mithilfe einer [4-Thread-Verarbeitungspipeline](../processing-architecture/processing-pipeline.md)
* Die [dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md) wählt zu Beginn der Ausführung automatisch die optimale Strategie für Ihre Hardware aus
* GPU-Beschleunigung (CUDA) mit NVIDIA-Grafikkarten
* **Die Anzahl der Worker passt sich der Hardware an**: Bei GPU-Strategien werden**2–4 Worker gleichzeitig** ausgeführt (skaliert nach VRAM – 3 bei einer GPU mit 2–12 GB, 2 bis 4 bei einer GPU mit 12 GB oder mehr); Systeme nur mit CPU führen einen Worker pro physischem Kern aus, minus eins**Der Fortschrittsbalken zeigt 4 Phasen an** (entsprechend den 4 Pipeline-Threads):

1. **Erkennung** (Thread 1) – Kalibrierungsziele finden
2. **Analyse** (Thread 2) – Untersuchung der Bildmetadaten und Berechnung der Kalibrierung
3. **Kalibrierung** (Thread 3) – Debayering, Vignettierungskorrektur, Kalibrierung, Indexberechnung
4. **Export** (Thread 4) – Speichern der verarbeiteten Bilder und Indizes**Interaktion mit dem Fortschrittsbalken:*** **Bewegen Sie den Mauszeiger** über den Balken, um das detaillierte Dropdown-Fenster mit den 4 Phasen anzuzeigen
* **Klicken Sie** auf den Fortschrittsbalken, um das Dropdown-Fenster an Ort und Stelle zu fixieren
* **Klicken Sie erneut**, um die Fixierung aufzuheben und das Fenster auszublenden**Verarbeitungszeit:**

* Deutlich schneller als im kostenlosen Modus
* GPU-Beschleunigung erhöht die Geschwindigkeit zusätzlich

{% hint style="info" %}
**Chloros+ Geschwindigkeit**: Bei großen Datensätzen kann die parallele Verarbeitung 5- bis 10-mal schneller sein als im sequenziellen Modus. Ein Projekt mit 500 Bildern, das im kostenlosen Modus 2 Stunden dauert, kann mit Chloros+ in 15 bis 20 Minuten abgeschlossen werden.
{% endhint %}

***

## Was geschieht während der Verarbeitung?

### Stufe 1: Zielerkennung

**Was Chloros tut:**

* Scannt die Bilder, die Sie in der Spalte „Ziel“ markiert haben (alle Bilder, wenn keine markiert sind)
* Identifiziert die Kalibrierungsfelder in jedem Ziel
* Extrahiert die Reflexionswerte aus den Zielfeldern
* Zeichnet die Zeitstempel der Ziele für die Kalibrierungsplanung auf

**Dauer:** 1–30 Sekunden (bei markierten Zielen), 5–30+ Minuten (bei unmarkierten Zielen)

### Stufe 2: Debayering (RAW-Konvertierung)

**Was Chloros tut:**

* Konvertiert RAW-Daten im Bayer-Muster in vollständige 3-Kanal-Bilder (LATTICE-Mono-Module bleiben einkanalig – bei ihnen wird das Debayering übersprungen und im Protokoll vermerkt)
* Wendet den ausgewählten Demosaicing-Algorithmus an
* Erhält maximale Bildqualität und Detailgenauigkeit

**Dauer:** Variiert je nach Bildanzahl und CPU-/GPU-Geschwindigkeit

### Stufe 3: Kalibrierung

**Was Chloros tut:*** **Vignettierungskorrektur**: Beseitigt die Randverdunkelung durch das Objektiv
* **Reflexionskalibrierung**: Normalisiert anhand von Zielreflexionswerten und/oder DAQ-Downwelling-Daten
* Wendet Korrekturen auf alle Bänder/Kanäle an
* Verwendet für jedes Bild die passende Kalibrierungsreferenz basierend auf dem Zeitstempel

**Dauer:** Der Großteil der Verarbeitungszeit

### Stufe 4: Indexberechnung

**Was Chloros tut:**

* Berechnet konfigurierte multispektrale Indizes (NDVI, NDRE usw.)
* Wendet Bandberechnungen auf kalibrierte Bilder an
* Erzeugt Indexbilder für jeden ausgewählten Index

**Dauer:** Einige Sekunden pro Bild

### Schritt 5: Export

**Was Chloros tut:**

* Speichert die verarbeiteten Bilder im ausgewählten Format
* **LATTICE-Fan-Out**: Jeder rohe LATTICE-Frame wird in einem Durchgang als jedes aktivierte Produkt exportiert – debayert, Vorschau, Strahlungsintensität (immer float32), Reflexionsgrad
* Schreibt Dateien in den Projekt-Ausgabebaum: `<project>/<camera>/<format>/<Product>_Images/`
* **Behält den Quelldateinamen bei** – der Ordner identifiziert das Produkt, es wird kein Suffix hinzugefügt**Dauer:** Variiert je nach Exportformat und Dateigröße***

## Verarbeitungsverhalten

### Automatische Verarbeitungspipeline

Nach dem Start läuft die gesamte Pipeline automatisch ab:

* Keine Benutzerinteraktion erforderlich
* Alle konfigurierten Schritte werden nacheinander ausgeführt
* Fortschrittsanzeigen werden in Echtzeit angezeigt
* Exportierte Dateien werden nach ihrer Fertigstellung auf die Festplatte geschrieben – Sie können fertige Ergebnisse öffnen, während der Lauf noch andauert

### Computerauslastung während der Verarbeitung

**Freier Modus:**

* Relativ geringe CPU-Auslastung (Single-Thread)
* Der Computer bleibt für andere Aufgaben reaktionsfähig
* Es ist unbedenklich, Chloros zu minimieren und in anderen Anwendungen zu arbeiten

**Chloros+ Parallelmodus:**

* Hohe CPU-Auslastung über den gesamten Worker-Pool der Strategie hinweg
* Bei GPU-Beschleunigung: Hohe GPU-Auslastung
* Der Computer reagiert während der Verarbeitung möglicherweise weniger schnell
* Vermeiden Sie es, andere CPU-intensive Aufgaben zu starten

{% hint style="warning" %}
**Leistungstipp**: Für eine optimale Leistung von Chloros+ schließen Sie andere Anwendungen und lassen Sie Chloros die gesamten Systemressourcen nutzen.
{% endhint %}

### Die Verarbeitung kann nicht angehalten werden (ein sauberes Beenden ist jedoch möglich)

* Einmal gestartete Verarbeitung kann nicht angehalten und später fortgesetzt werden
* Ein Klick auf **Stopp** beendet den Lauf sauber beim ersten Klick
* Bereits vor dem Stopp exportierte Produkte bleiben auf der Festplatte erhalten
* Ein gestoppter Lauf gibt ehrlich an, was abgeschlossen wurde (siehe die Zeilen `[RUN-SUMMARY]` im Protokoll)
* Ein neuer Durchlauf startet die Pipeline von Anfang an

**Planungstipp:** Bei sehr großen Projekten sollten Sie eine Verarbeitung in Stapeln oder die Verwendung der Befehlszeilenschnittstelle (CLI) in Betracht ziehen, um eine bessere Kontrolle zu erhalten.

***

## Überwachung Ihrer Verarbeitung

Während der Verarbeitung können Sie:

* **Den Fortschrittsbalken beobachten** – Den Gesamtfortschritt in Prozent anzeigen
* **Die aktuelle Phase anzeigen** – Erkennen, Analysieren, Kalibrieren oder Exportieren
* **Die Registerkarte „Protokoll“ überprüfen** – Detaillierte Verarbeitungsmeldungen und Warnungen anzeigen
* **Eine Vorschau der fertigen Bilder anzeigen** – Exportierte Dateien erscheinen bereits während der Verarbeitung auf der Festplatte

Ausführliche Informationen zur Überwachung finden Sie unter [Überwachung der Verarbeitung](monitoring-the-processing.md).

***

## Verarbeitung anhalten

Wenn Sie die Verarbeitung anhalten müssen:

### So halten Sie die Verarbeitung an

1. Suchen Sie die **Schaltfläche „Stopp“** (ersetzt während der Verarbeitung die Schaltfläche „Start“)
2. Klicken Sie einmal darauf – in der Leiste wird **„Wird angehalten...“** angezeigt, während das gerade verarbeitete Bild fertiggestellt wird
3. Der Durchlauf endet in einem endgültigen Stoppzustand und das Protokoll gibt einen genauen Überblick über die abgeschlossenen Schritte aus (`[RUN-SUMMARY]`)

### Wann sollte man die Verarbeitung anhalten?

**Gültige Gründe für ein Anhalten:**

* Es wurde festgestellt, dass falsche Einstellungen verwendet wurden
* Das Markieren der Zielbilder wurde vergessen
* Es wurden falsche Bilder importiert
* Das System läuft zu langsam oder reagiert nicht mehr

**Nach dem Anhalten:**

* Vor dem Anhalten exportierte Produkte bleiben auf der Festplatte erhalten
* Überprüfen und beheben Sie etwaige Probleme, passen Sie die Einstellungen bei Bedarf an
* Starten Sie die Verarbeitung neu – der Lauf beginnt von vorne

***

## Geschätzte Verarbeitungszeiten

Die tatsächliche Verarbeitungszeit hängt stark von folgenden Faktoren ab:

* Anzahl der Bilder
* Bildauflösung
* Eingabeformat (RAW oder JPG)
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
* Eine vorherige Verarbeitung läuft noch
* Das Projekt wurde noch nicht vollständig geladen

**Lösungen:**

1. Warten Sie, bis das Backend vollständig initialisiert ist (überprüfen Sie das Symbol im Hauptmenü)
2. Vergewissern Sie sich, dass die Bilder im Dateibrowser importiert sind
3. Starten Sie Chloros neu, wenn die Schaltfläche weiterhin deaktiviert ist
4. Überprüfen Sie das Debug-Protokoll auf Fehlermeldungen

### Die Verarbeitung startet, bricht dann aber sofort ab

**Mögliche Ursachen:**

* Keine gültigen Bilder im Projekt
* Beschädigte Bilddateien
* Unzureichender Festplattenspeicher
* Unzureichender Arbeitsspeicher (RAM)

**Lösungen:**

1. Überprüfen Sie das Debug-Protokoll <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> auf Fehlermeldungen
2. Überprüfen Sie den verfügbaren Speicherplatz
3. Versuchen Sie, eine kleinere Teilmenge der Bilder zu verarbeiten
4. Vergewissern Sie sich, dass die Bilder nicht beschädigt sind

### Der Lauf wird abgeschlossen, es werden jedoch keine Bilder geschrieben

Ein Lauf, bei dem Bildprodukte angefordert, aber keine geschrieben wurden, wird als **Fehlschlag und nicht als Erfolg** gewertet – Chloros meldet dies deutlich:

* Das GUI-Protokoll gibt Hinweise mit den Codes `[RUN-SUMMARY]` aus, die die wahrscheinliche Ursache benennen – keine Bilder importiert, kein Ziel erkannt oder jedes angeforderte Produkt als nicht anwendbar übersprungen (z. B. bei der Anforderung von Radianz-/Reflexionswerten von reinen RGB-Kameras)
* Das entsprechende CLI-Meldungsformat (`chloros-cli process`) gibt „`Processing finished but wrote no image products.`“ aus und **beendet den Vorgang mit einem Wert ungleich Null**, sodass Skripte dies erkennen können
* Ein absichtlich nur auf Metadaten ausgerichteter Lauf (alle Exportprodukte deaktiviert, keine Indizes) gilt weiterhin als Erfolg

Die vollständige Semantik finden Sie in der [CLI-Referenz](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Der Projektordner ist für diesen Computer zu tief

**Nur Windows, neu in 1.3.0.** Windows wird standardmäßig mit deaktivierten langen Pfaden ausgeliefert, sodass kein Dateipfad länger als 260 Zeichen sein darf. Chloros-Produkte befinden sich bis zu etwa 130 Zeichen unterhalb des Projektordners, sodass ein tiefer liegender Projektordner seine Produkte nicht erhalten könnte. Anstatt den Lauf auszuführen und nichts zu schreiben, prüft Chloros nun vor Beginn der Verarbeitung:

* Ein Klick auf „Start“ löst keinen Lauf aus: Der Fortschrittsbalken zeigt den Grund an, und im Debug-Protokoll wird „`[PROCESS] refused: The project folder is too deep for this computer…`“ angezeigt. Die Meldung gibt die Länge des Projektordners und den längsten Projektordner an, der noch passt.
* Das Erstellen oder Öffnen eines Projekts in einem Ordner, der selbst für die eigenen Dateien des Projekts zu tief liegt, wird ebenfalls abgelehnt. Ein Meldungsfeld erklärt den Grund, nennt den längsten zulässigen Projektordnernamen und erklärt, wie man lange Pfade aktivieren kann; es wird nichts erstellt oder geöffnet; im Debug-Protokoll wird dies als „`[PROJECT] refused: …`“ protokolliert. Auch das Duplizieren eines Projekts unter einem längeren Namen kann die Grenze überschreiten: Die Kopie wird erstellt, aber nicht geöffnet, und dieselbe Meldung erklärt den Grund.

**Lösungen:**

1. Speichern Sie Projekte in einem Ordner mit kürzerem Pfad – ändern Sie das**Arbeitsverzeichnis** unter [Projekteinstellungen](../project-settings/project-settings.md) und erstellen Sie das Projekt erneut, oder
2. Aktivieren Sie lange Pfade unter Windows und starten Sie Chloros neu: Führen Sie das Chloros-Installationsprogramm erneut aus, wobei die Option **„Lange Pfade unter Windows aktivieren“** aktiviert sein muss (sie ist standardmäßig aktiviert), oder verwenden Sie die Gruppenrichtlinie: *Computerkonfiguration > Administrative Vorlagen > System > Dateisystem > Lange Win32-Pfade aktivieren*.

### Warnung „Keine Ziele erkannt“

**Mögliche Ursachen:**

* Vergessen, Zielbilder zu markieren
* Zielbilder enthalten keine sichtbaren Ziele
* Einstellungen zur Zielerkennung sind zu streng

**Lösungen:**

1. Lesen Sie den Abschnitt [Auswahl von Zielbildern](choosing-target-images.md)
2. Markieren Sie die entsprechenden Bilder in der Spalte „Ziel“
3. Vergewissern Sie sich, dass die Ziele in den markierten Bildern sichtbar sind
4. Passen Sie die Einstellungen für die Zielerkennung bei Bedarf an

***

## Tipps für eine erfolgreiche Verarbeitung

### Vor dem Start

1. **Zunächst mit einer kleinen Teilmenge testen** – 10–20 Bilder verarbeiten, um die Einstellungen zu überprüfen
2. **Verfügbaren Speicherplatz prüfen** – Stellen Sie sicher, dass das 2- bis 3-Fache der Datensatzgröße frei ist (mehr, wenn alle LATTICE-Produkte aktiviert sind)
3. **Schließen Sie nicht benötigte Anwendungen** – Geben Sie Systemressourcen frei
4. **Überprüfen Sie die Zielbilder** – Sehen Sie sich eine Vorschau der markierten Ziele an, um die Qualität sicherzustellen
5. **Speichern Sie das Projekt** – Das Projekt wird automatisch gespeichert, es empfiehlt sich jedoch, es manuell zu speichern

### Während der Verarbeitung

1. **System-Ruhezustand vermeiden** – Energiesparmodi deaktivieren
2. **Chloros im Vordergrund lassen** – Oder zumindest in der Taskleiste sichtbar halten
3. **Fortschritt gelegentlich überwachen** – Auf Warnungen oder Fehler prüfen
4. **Keine anderen ressourcenintensiven Anwendungen laden** – Insbesondere im Chloros+ Parallelmodus

### Chloros+ GPU-Beschleunigung

Bei Verwendung der NVIDIA-GPU-Beschleunigung:

1. NVIDIA-Treiber auf die neueste Version aktualisieren
2. Sicherstellen, dass die GPU über mindestens 4 GB VRAM verfügt (mindestens 7 GB für gleichzeitiges „Texture Aware Debayering“)
3. GPU-intensive Anwendungen schließen (Spiele, Videobearbeitung)
4. GPU-Temperatur überwachen (für ausreichende Kühlung sorgen)

***

## Nächste Schritte

Sobald die Verarbeitung begonnen hat:

1. **Überwachen Sie den Fortschritt** – siehe [Überwachung der Verarbeitung](monitoring-the-processing.md)
2. **Warten Sie, bis der Vorgang abgeschlossen ist** – die Verarbeitung läuft automatisch ab
3. **Überprüfen Sie die Ergebnisse** – siehe [Abschluss der Verarbeitung](finishing-the-processing.md)

Informationen dazu, was während der Verarbeitung zu tun ist, finden Sie unter [Überwachung der Verarbeitung](monitoring-the-processing.md).
