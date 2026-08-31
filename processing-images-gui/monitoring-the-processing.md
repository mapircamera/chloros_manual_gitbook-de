# Überwachung der Verarbeitung

Sobald die Verarbeitung begonnen hat, bietet Chloros verschiedene Möglichkeiten, den Fortschritt zu überwachen, auf Probleme zu prüfen und zu verstehen, was mit Ihrem Datensatz geschieht. Auf dieser Seite wird erläutert, wie Sie Ihre Verarbeitung verfolgen und die von Chloros bereitgestellten Informationen interpretieren können.

## Übersicht über den Fortschrittsbalken

Der Fortschrittsbalken in der oberen Kopfzeile zeigt den Verarbeitungsstatus in Echtzeit und den Fertigstellungsgrad in Prozent an. Der Fortschritt wird live aus dem Backend über Server-Sent Events (SSE) übertragen, sodass der Balken genau widerspiegelt, was die Pipeline gerade tut.

### Fortschrittsbalken im kostenlosen Modus

Für Nutzer ohne Chloros+-Lizenz:

**2-stufige Fortschrittsanzeige:**

1.**Zielerkennung** – Erkennen von Kalibrierungszielen in Bildern
2. **Verarbeitung** – Anwenden von Korrekturen und Exportieren**Der Fortschrittsbalken zeigt:**

* Gesamtfortschrittsprozent (0–100 %)
* Name der aktuellen Phase
* Einfache horizontale Balkenvisualisierung

### Chloros+-Fortschrittsbalken

Für Benutzer mit einer Chloros+-Lizenz:

**4-stufige Fortschrittsanzeige:**

1.**Erkennen** – Kalibrierungsziele finden
2. **Analysieren** – Bilder untersuchen und Pipeline vorbereiten
3. **Kalibrieren** – Anwenden von Vignettierungs- und Reflexionskorrekturen
4. **Exportieren** – Speichern der verarbeiteten Dateien**Interaktive Funktionen:*** **Führen Sie den Mauszeiger über** den Fortschrittsbalken, um das erweiterte 4-Stufen-Fenster anzuzeigen
* **Klicken Sie** auf den Fortschrittsbalken, um das erweiterte Fenster einzufrieren/anzuheften
* **Klicken Sie erneut**, um die Fixierung aufzuheben; das Fenster wird automatisch ausgeblendet, wenn der Mauszeiger entfernt wird
* Jede Phase zeigt den individuellen Fortschritt an (0–100 %)

{% hint style="info" %}
**CLI Parität**: Während eines `chloros-cli process`-Laufs melden dieselben vier Threads „Detecting“, „Analyzing“, „Processing“ und „Exporting“, während `chloros-cli export-status` den Live-Fortschritt des Exports von Thread 4 auf einem anderen Terminal anzeigt. Siehe die [CLI-Referenz](../reference/cli-reference.md).
{% endhint %}

***

## Die einzelnen Verarbeitungsphasen verstehen

{% hint style="info" %}
**Pipeline-Architektur**: Diese vier GUI-Phasen entsprechen der [4-Thread-Verarbeitungspipeline](../processing-architecture/processing-pipeline.md). Auf Systemen mit GPU-Beschleunigung profitiert Thread 3 (Kalibrierung) von der [dynamischen Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md), die die Verarbeitung für Ihre spezifische Hardware optimiert.
{% endhint %}

### Stufe 1: Erkennung (Zielerkennung)

**Was geschieht:**

* Chloros scannt die Bilder, die Sie mit dem Kontrollkästchen „Ziel“ markiert haben (alle Bilder, wenn keines markiert ist)
* Bildverarbeitungsalgorithmen identifizieren die Kalibrierungsfelder
* Aus jedem Feld werden Reflexionswerte extrahiert
* Zeitstempel der Ziele werden für eine korrekte Kalibrierungsplanung aufgezeichnet

**Dauer:**

* Mit markierten Zielen: 10–60 Sekunden
* Ohne markierte Ziele: 5–30+ Minuten (alle Bilder werden gescannt)

**Fortschrittsanzeige:**

* Erkennung: 0 % → 100 %
* Anzahl der gescannten Bilder (zählt nur die tatsächlich gescannten Bilder)
* Anzahl der gefundenen Ziele

**Was zu beachten ist:**

* Sollte schnell abgeschlossen sein, wenn die Ziele ordnungsgemäß markiert sind
* Wenn es zu lange dauert, sind die Ziele möglicherweise nicht markiert
* Überprüfen Sie das Debug-Protokoll auf „Ziel gefunden“-Meldungen

### Stufe 2: Analyse

**Was geschieht:**

* Einlesen der EXIF-Metadaten der Bilder (Zeitstempel, Belichtungseinstellungen)
* Festlegung der Kalibrierungsstrategie auf Basis der Ziel-Zeitstempel und der verfügbaren DAQ-Downwelling-Daten
* Organisation der Bildverarbeitungswarteschlange
* Vorbereitung der Worker für die parallele Verarbeitung (nur Chloros+)

**Dauer:** 5–30 Sekunden**Fortschrittsanzeige:**

* Analyse: 0 % → 100 %
* Schnelle Phase, ist in der Regel zügig abgeschlossen

**Worauf zu achten ist:**

* Der Fortschritt sollte gleichmäßig und ohne Unterbrechungen verlaufen
* Warnungen zu fehlenden Metadaten werden im Debug-Protokoll angezeigt

### Phase 3: Kalibrierung

**Was passiert:*** **Debayering**: Umwandlung des RAW-Bayer-Musters in 3 Kanäle (wird bei LATTICE-Mono-Modulen übersprungen, mit einem Hinweis)
* **Vignettierungskorrektur**: Beseitigung der Randverdunkelung des Objektivs
* **Reflexionskalibrierung**: Normalisierung anhand von Zielwerten und/oder DAQ-Downwelling
* **Indexberechnung**: Berechnung multispektraler Indizes
* Verarbeitung jedes Bildes durch die gesamte Pipeline

**Dauer:** Der Großteil der gesamten Verarbeitungszeit (60–80 %)**Fortschrittsanzeige:**

* Kalibrierung: 0 % → 100 %
* Aktuelles Bild wird verarbeitet
* Fertige Bilder / Gesamtanzahl der Bilder

**Verarbeitungsverhalten:*** **Freier Modus**: Verarbeitet jeweils ein Bild nacheinander
* **Chloros+-Modus**: Lässt einen hardwareadaptiven Worker-Pool laufen – 1–4 parallele Worker auf GPU-Systemen (abhängig vom VRAM), einen Worker pro physischem Kern (minus eins) auf reinen CPU-Systemen. Siehe [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-Beschleunigung**: Beschleunigt diese Phase erheblich**Worauf zu achten ist:**

* Gleichmäßiger Fortschritt anhand der Bildanzahl
* Überprüfen Sie das Debug-Protokoll auf Abschlussmeldungen für jedes einzelne Bild
* Warnungen bezüglich Bildqualität oder Kalibrierungsproblemen

### Stufe 4: Export

**Was passiert:**

* Die verarbeiteten Bilder werden nach ihrer Fertigstellung im ausgewählten Format auf die Festplatte geschrieben
* **LATTICE**: Jedes Bild wird auf alle aktivierten Produkte verteilt (Debayering / Vorschau / Radiance / Reflektanz)
* Export von multispektralen Indexbildern mit LUT-Farben
* Erstellung des Ausgabeverzeichnisses `<project>/<camera>/<format>/<Product>_Images/` – exportierte Dateien behalten den Quelldateinamen; der Ordner identifiziert das Produkt

**Dauer:** 10–20 % der gesamten Verarbeitungszeit**Fortschrittsanzeige:**

* Export: 0 % → 100 %
* Dateien werden geschrieben
* Exportformat und Zielort

**Zu beachten:**

* Warnungen zum Speicherplatz
* Fehler beim Schreiben von Dateien
* Abschluss aller konfigurierten Ausgabevorgänge

***

## Registerkarte „Debug-Protokoll“

Das Debug-Protokoll liefert detaillierte Informationen zum Verarbeitungsfortschritt und zu eventuell aufgetretenen Problemen. Auch Startmeldungen des Backends werden in der Protokollkonsole wiedergegeben, sodass das Protokoll den gesamten Verlauf wiedergibt, selbst wenn Sie es erst später öffnen.

### Auf das Debug-Protokoll zugreifen

1. Klicken Sie in der linken Seitenleiste auf das **Debug-Protokoll**-<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

-Symbol
2. Das Protokollfenster wird geöffnet und zeigt Verarbeitungsmeldungen in Echtzeit an
3. Es wird automatisch gescrollt, um die neuesten Meldungen anzuzeigen

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Protokollmeldungen verstehen

Chloros-Protokollzeilen sind mit Tags in eckigen Klammern vorangestellt, die das Subsystem benennen – zum Beispiel `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Am wichtigsten ist die **Laufzusammenfassung**, die am Ende jedes Laufs (einschließlich abgebrochener Läufe) ausgegeben wird:

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Zusätzliche Hinweszeilen vom Typ `[RUN-SUMMARY]` folgen immer dann, wenn etwas erklärt werden muss – zum Beispiel bei einem Lauf, der keine Ergebnisse lieferte, oder bei einer Kamera, deren angefordertes Produkt als nicht anwendbar übersprungen wurde. Zeilen vom Typ `[EXPORT-CHECK]` erklären proKamera (z. B. warum eine RGB-Kamera kein Radiance-Produkt erhalten hat).

Die allgemeinen Schweregrade der Meldungen (die folgenden Beispiele dienen der Veranschaulichung und sind nicht wörtlich zu verstehen):

#### Informationsmeldungen (weiß/grau)

Normale Verarbeitungsaktualisierungen: Verarbeitung gestartet, Ziele erkannt (mit Panel-Anzahl), Fortschritt der Kalibrierung pro Bild, Dateien exportiert, Verarbeitung abgeschlossen.

#### Warnmeldungen (gelb)

Nicht kritische Probleme, die die Verarbeitung nicht unterbrechen – z. B. fehlende GPS-Daten in einem Bild, eine große Lücke im Zeitstempel zwischen Zielbildern oder geringer Kontrast in einem Kalibrierungsfeld.

**Maßnahme:** Überprüfen Sie die Warnungen nach der Verarbeitung, unterbrechen Sie diese jedoch nicht.

#### Fehlermeldungen (Red)

Kritische Probleme, die zum Abbruch der Verarbeitung führen können – z. B. volle Festplatte, eine beschädigte Bilddatei oder keine erkannten Ziele, obwohl eine Reflexionskalibrierung angefordert wurde.

**Maßnahme:** Verarbeitung stoppen, Fehler beheben, neu starten

### Häufige Protokollsituationen

| Situation                             | Bedeutung                                       | Erforderliche Maßnahme                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Ziel in \[Dateiname] erkannt        | Kalibrierungsziel erfolgreich gefunden         | Keine – normal                                         |
| Fortschrittsbalken pro Bild              | Aktuelle Fortschrittsanzeige                       | Keine – normal                                         |
| Keine Ziele gefunden                      | Keine Kalibrierungsziele erkannt               | Zielbilder markieren oder Reflektanzkalibrierung deaktivieren |
| Unzureichender Speicherplatz               | Nicht genügend Speicherplatz für die Ausgabe                 | Speicherplatz freigeben                                    |
| Beschädigte Datei wird übersprungen               | Bilddatei ist beschädigt                         | Datei erneut von der SD-Karte kopieren                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Eine Aufnahme ohne RAW-Frame kann nicht verarbeitet werden | Remit RAW-Frame neu oder verwenden Sie CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Der Lauf hat keine Bildprodukte erzeugt – als Fehler mit Hinweisen gemeldet | Lesen Sie die Hinweszeilen; überprüfen Sie, was übersprungen wurde und warum |

### Protokoll-Daten kopieren

So kopieren Sie das Protokoll zur Fehlerbehebung oder für den Support:

1. Öffnen Sie das Fenster „Debug-Protokoll“
2. Klicken Sie auf die Schaltfläche **„Protokoll kopieren“** (oder Rechtsklick → „Alles auswählen“)
3. Fügen Sie den Inhalt in eine Textdatei oder eine E-Mail ein
4. Senden Sie diese bei Bedarf an den MAPIR-Support

***

## Überwachung der Systemressourcen

### CPU-Auslastung

**Freier Modus:**

* 1 CPU-Kern bei ~100 %
* Andere Kerne im Leerlauf oder verfügbar
* Das System bleibt reaktionsfähig

**Chloros+ Parallelmodus:**

* Mehrere Kerne mit hoher Auslastung – die Anzahl hängt von der durch [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) gewählten Strategie ab
* Das System wirkt möglicherweise weniger reaktionsschnell

**Zur Überwachung:**

* Windows Task-Manager (Strg+Umschalt+Esc)
* Registerkarte „Leistung“ → Abschnitt „CPU“
* Suchen Sie nach den Prozessen „Chloros“ oder „chloros-backend“

### Speicher (RAM)-Auslastung

**Typische Auslastung:**

* Kleine Projekte (&lt; 100 Bilder): 2–4 GB
* Mittlere Projekte (100–500 Bilder): 4–8 GB
* Große Projekte (500+ Bilder): 8–16 GB
* Der Parallelmodus von Chloros+ benötigt mehr RAM

**Bei geringem Arbeitsspeicher:**

* Verarbeiten Sie kleinere Stapel
* Schließen Sie andere Anwendungen
* Erweitern Sie den Arbeitsspeicher, wenn Sie regelmäßig große Datensätze verarbeiten

### GPU-Auslastung (Chloros+ mit CUDA)

Wenn die GPU-Beschleunigung aktiviert ist:

* Die NVIDIA-GPU weist eine hohe Auslastung auf (60–90 %)
* Die VRAM-Auslastung steigt (erfordert mindestens 4 GB VRAM; mindestens 7 GB für gleichzeitiges „Texture Aware Debayering“)
* Die Kalibrierungsphase ist deutlich schneller

**Zur Überwachung:**

* NVIDIA-Symbol in der Taskleiste
* Task-Manager → Leistung → GPU
* GPU-Z oder ein ähnliches Überwachungstool

### Festplatten-E/A

**Zu erwarten:**

* Hohe Lesezugriffe auf die Festplatte während der Analysephase
* Hohe Schreibzugriffe auf die Festplatte während der Exportphase
* SSD deutlich schneller als HDD

**Leistungstipp:**

* Verwenden Sie nach Möglichkeit eine SSD für den Projektordner
* Vermeiden Sie Netzlaufwerke für große Datensätze
* Stellen Sie sicher, dass der Speicherplatz nicht fast erschöpft ist (beeinträchtigt die Schreibgeschwindigkeit)

***

## Probleme während der Verarbeitung erkennen

### Warnzeichen

**Fortschritt stockt (keine Veränderung seit mehr als 5 Minuten):**

* Überprüfen Sie das Debug-Protokoll auf Fehler
* Überprüfen Sie den verfügbaren Festplattenspeicher
* Überprüfen Sie im Task-Manager, ob „Chloros“ ausgeführt wird

**Es erscheinen häufig Fehlermeldungen:**

* Brechen Sie die Verarbeitung ab und überprüfen Sie die Fehler
* Häufige Ursachen: Festplattenspeicher, beschädigte Dateien, Speicherprobleme
* Siehe Abschnitt „Fehlerbehebung“ weiter unten

**Das System reagiert nicht mehr:**

* Der Parallelmodus von Chloros+ beansprucht zu viele Ressourcen
* Erwägen Sie, die Anzahl der gleichzeitig ausgeführten Aufgaben zu reduzieren oder die Hardware aufzurüsten
* Der Freimodus ist weniger ressourcenintensiv

### Wann die Verarbeitung abgebrochen werden sollte

Brechen Sie die Verarbeitung ab, wenn Folgendes auftritt:

* ❌ Fehlermeldungen wie „Festplatte voll“ oder „Datei kann nicht geschrieben werden“
* ❌ Wiederholte Fehler aufgrund beschädigter Bilddateien
* ❌ Das System ist vollständig eingefroren (reagiert nicht mehr)
* ❌ Sie haben festgestellt, dass falsche Einstellungen konfiguriert wurden
* ❌ Falsche Bilder wurden importiert

**So brechen Sie den Vorgang ab:**

1. Klicken Sie auf die**Schaltfläche „Stopp“** (ersetzt die Schaltfläche „Start“) – einmal reicht aus
2. Der Fortschrittsbalken zeigt „Wird angehalten...“ an, während das gerade verarbeitete Image fertiggestellt wird; anschließend endet der Durchlauf im angehaltenen Zustand
3. Bereits exportierte Produkte bleiben auf der Festplatte erhalten; das Protokoll gibt einen detaillierten Überblick über die abgeschlossenen Vorgänge (`[RUN-SUMMARY]`)
4. Beheben Sie die Probleme und starten Sie den Vorgang neu – er beginnt von vorne

***

## Fehlerbehebung während der Verarbeitung

### Die Verarbeitung ist sehr langsam

**Mögliche Ursachen:**

* Nicht markierte Zielbilder (es werden alle Bilder gescannt)
* HDD statt SSD als Speichermedium
* Unzureichende Systemressourcen
* Viele konfigurierte Indizes
* Zugriff auf ein Netzlaufwerk

**Lösungen:**

1. Wenn der Vorgang gerade erst gestartet wurde und sich in der Erkennungsphase befindet: Anhalten, Ziele markieren, neu starten
2. Für die Zukunft: SSD verwenden, Indizes reduzieren, Hardware aufrüsten
3. CLI für die Stapelverarbeitung großer Datensätze in Betracht ziehen

### Warnungen zum „Speicherplatz“

**Lösungen:**

1. Sofort Speicherplatz freigeben
2. Projekt auf ein Laufwerk mit mehr Speicherplatz verschieben
3. Reduzieren Sie die Anzahl der zu exportierenden Indizes
4. Deaktivieren Sie nicht benötigte LATTICE-Exportprodukte (Projekteinstellungen → Verarbeitung)
5. Verwenden Sie das JPG-Format anstelle von TIFF (kleinere Dateien)

### Häufige Meldungen über „beschädigte Dateien“

**Lösungen:**

1. Kopieren Sie die Bilder erneut von der SD-Karte, um die Integrität sicherzustellen
2. Prüfen Sie die SD-Karte auf Fehler
3. Entfernen Sie beschädigte Dateien aus dem Projekt
4. Setzen Sie die Verarbeitung der verbleibenden Bilder fort

### Überhitzung / Drosselung des Systems

**Lösungen:**

1. Sorgen Sie für ausreichende Belüftung
2. Staub aus den Lüftungsschlitzen des Computers entfernen
3. Verarbeitungslast reduzieren (den „Free“-Modus anstelle von Chloros+ verwenden)
4. Die Verarbeitung zu kühleren Tageszeiten durchführen

***

## Benachrichtigung über den Abschluss der Verarbeitung

Wenn die Verarbeitung abgeschlossen ist:

* Der Fortschrittsbalken erreicht 100 %
* Die Zeilen „`[RUN-SUMMARY]`“ erscheinen im Debug-Protokoll mit den endgültigen Zahlen
* Die Schaltfläche „Start“ wird wieder aktiv
* Alle Ausgabedateien befinden sich im kameraspezifischen Ausgabeverzeichnis des Projekts: `<project>/<camera>/<format>/<Product>_Images/`

***

## Nächste Schritte

Sobald die Verarbeitung abgeschlossen ist:

1. **Ergebnisse überprüfen** – Siehe [Abschluss der Verarbeitung](finishing-the-processing.md)
2. **Ausgabeordner überprüfen** – Vergewissern Sie sich, dass alle Dateien korrekt exportiert wurden
3. **Debug-Protokoll überprüfen** – Auf Warnungen oder Fehler prüfen
4. **Vorschau der verarbeiteten Bilder** – Verwenden Sie den Bildbetrachter oder eine externe Software

Informationen zum Überprüfen und Verwenden Ihrer verarbeiteten Ergebnisse finden Sie unter [Abschluss der Verarbeitung](finishing-the-processing.md).
