# Überwachung der Verarbeitung

Sobald die Verarbeitung begonnen hat, bietet Chloros verschiedene Möglichkeiten, den Fortschritt zu überwachen, auf Probleme zu prüfen und zu verstehen, was mit Ihrem Datensatz geschieht. Auf dieser Seite wird erläutert, wie Sie Ihre Verarbeitung verfolgen und die von Chloros bereitgestellten Informationen interpretieren können.

## Übersicht über den Fortschrittsbalken

Der Fortschrittsbalken in der oberen Kopfzeile zeigt den Verarbeitungsstatus in Echtzeit und den Fertigstellungsgrad in Prozent an. Der Fortschritt wird live aus dem Backend über Server-Sent Events (SSE) übertragen, sodass der Balken genau widerspiegelt, was die Pipeline gerade tut.

### Fortschrittsbalken im kostenlosen Modus

Für Nutzer ohne Chloros+-Lizenz:

**2-stufige Fortschrittsanzeige:**

1.**Zielerkennung** – Kalibrierungsziele in Bildern finden
2. **Verarbeitung** – Korrekturen anwenden und exportieren**Die Fortschrittsleiste zeigt:**

* Gesamtfortschrittsprozentsatz (0–100 %)
* Name der aktuellen Phase
* Einfache horizontale Balkenvisualisierung

### Chloros+-Fortschrittsbalken

Für Nutzer mit einer Chloros+-Lizenz:

**4-stufige Fortschrittsanzeige:**

1.**Erkennen** – Kalibrierungsziele finden
2. **Analysieren** – Bilder untersuchen und Pipeline vorbereiten
3. **Kalibrierung** – Anwenden von Vignettierungs- und Reflexionskorrekturen
4. **Export** – Speichern der verarbeiteten Dateien

**Interaktive Funktionen:**

* **Bewegen Sie den Mauszeiger über** den Fortschrittsbalken, um das erweiterte 4-Stufen-Fenster anzuzeigen
* **Klicken Sie** auf den Fortschrittsbalken, um das erweiterte Fenster einzufrieren/anzuheften
* **Klicken Sie erneut**, um die Fixierung aufzuheben; das Fenster wird automatisch ausgeblendet, wenn Sie den Mauszeiger entfernen
* Jede Phase zeigt den individuellen Fortschritt an (0–100 %)

{% hint style="info" %}
**Parität mit der CLI**: Während eines `chloros-cli process`-Laufs melden dieselben vier Threads „Detecting“, „Analyzing“, „Processing“ und „Exporting“, und `chloros-cli export-status` zeigt den Live-Fortschritt des Thread-4-Exports von einem anderen Terminal aus an. Siehe die [CLI-Referenz](../reference/cli-reference.md).
{% endhint %}

***

## Die einzelnen Verarbeitungsphasen verstehen

{% hint style="info" %}
**Pipeline-Architektur**: Diese 4 GUI-Phasen entsprechen der [4-Thread-Verarbeitungspipeline](../processing-architecture/processing-pipeline.md). Auf Systemen mit GPU-Beschleunigung profitiert Thread 3 (Kalibrierung) von der [dynamischen Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md), die die Verarbeitung für Ihre spezifische Hardware optimiert.
{% endhint %}

### Stufe 1: Erkennung (Zielerkennung)

**Was passiert:**

* Chloros scannt die Bilder, die Sie mit dem Kontrollkästchen „Ziel“ markiert haben (alle Bilder, wenn keines markiert ist)
* Computer-Vision-Algorithmen identifizieren die Kalibrierungsfelder
* Reflexionswerte werden aus jedem Feld extrahiert
* Zeitstempel der Ziele werden für eine korrekte Kalibrierungsplanung aufgezeichnet

**Dauer:**

* Mit markierten Zielen: 10–60 Sekunden
* Ohne markierte Ziele: 5–30+ Minuten (alle Bilder werden gescannt)

**Fortschrittsanzeige:**

* Erkennung: 0 % → 100 %
* Anzahl der gescannten Bilder (zählt nur die tatsächlich gescannten Bilder)
* Anzahl der gefundenen Zielmarken

**Was zu beachten ist:**

* Sollte schnell abgeschlossen sein, wenn die Zielmarken korrekt markiert sind
* Wenn es zu lange dauert, sind die Zielmarken möglicherweise nicht markiert
* Überprüfen Sie das Debug-Protokoll auf „Ziel gefunden“-Meldungen

### Phase 2: Analyse

**Was passiert:**

* Einlesen der EXIF-Metadaten der Bilder (Zeitstempel, Belichtungseinstellungen)
* Festlegung der Kalibrierungsstrategie auf Basis der Zeitstempel der Ziele und der verfügbaren DAQ-Downwelling-Daten
* Organisation der Bildverarbeitungswarteschlange
* Vorbereitung der Worker für die parallele Verarbeitung (nur Chloros+)

**Dauer:** 5–30 Sekunden**Fortschrittsanzeige:**

* Analyse: 0 % → 100 %
* Schnelle Phase, ist in der Regel zügig abgeschlossen

**Worauf zu achten ist:**

* Der Fortschritt sollte gleichmäßig und ohne Unterbrechungen verlaufen
* Warnungen zu fehlenden Metadaten werden im Debug-Protokoll angezeigt

### Phase 3: Kalibrierung

**Was passiert:**

* **Debayering**: Umwandlung des RAW-Bayer-Musters in 3 Kanäle (wird bei LATTICE-Mono-Modulen übersprungen, mit einem Hinweis)
* **Vignettierungskorrektur**: Beseitigung der Randverdunkelung durch das Objektiv
* **Reflexionskalibrierung**: Normalisierung anhand von Zielwerten und/oder DAQ-Downwelling
* **Indexberechnung**: Berechnung multispektraler Indizes
* Verarbeitung jedes Bildes durch die gesamte Pipeline

**Dauer:** Der Großteil der gesamten Verarbeitungszeit (60–80 %)**Fortschrittsanzeige:**

* Kalibrierung: 0 % → 100 %
* Aktuelles Bild wird verarbeitet
* Fertige Bilder / Gesamtzahl der Bilder

**Verarbeitungsverhalten:**

* **Freier Modus**: Verarbeitet jeweils ein Bild nacheinander
* **„Chloros+“-Modus**: Lässt einen hardwareadaptiven Worker-Pool laufen – 1–4 parallele Worker auf GPU-Systemen (abhängig vom VRAM), einen Worker pro physikalischem Kern (minus eins) auf reinen CPU-Systemen. Siehe [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-Beschleunigung**: Beschleunigt diese Phase erheblich**Was zu beachten ist:**

* Gleichmäßiger Fortschritt bei der Bildanzahl
* Überprüfen Sie das Debug-Protokoll auf Abschlussmeldungen für jedes einzelne Bild
* Warnungen bezüglich Bildqualität oder Kalibrierungsproblemen

### Stufe 4: Export

**Was passiert:**

* Die verarbeiteten Bilder werden nach ihrer Fertigstellung im ausgewählten Format auf die Festplatte geschrieben
* **LATTICE**: Jedes Bild wird auf alle aktivierten Produkte verteilt (Debayering / Vorschau / Radiance / Reflektanz)
* Export von multispektralen Indexbildern mit LUT-Farben
* Erstellung des Ausgabeverzeichnisses `<project>/<camera>/<format>/<Product>_Images/` – exportierte Dateien behalten den Quelldateinamen bei; der Ordner identifiziert das Produkt

**Dauer:** 10–20 % der gesamten Verarbeitungszeit**Fortschrittsanzeige:**

* Export: 0 % → 100 %
* Dateien werden geschrieben
* Exportformat und Zielort

**Worauf zu achten ist:**

* Warnungen zum Speicherplatz
* Fehler beim Schreiben von Dateien
* Abschluss aller konfigurierten Ausgabevorgänge

***

## Registerkarte „Debug-Protokoll“

Das Debug-Protokoll liefert detaillierte Informationen zum Verarbeitungsfortschritt und zu eventuell aufgetretenen Problemen. Auch Startmeldungen des Backends werden in der Protokollkonsole wiedergegeben, sodass das Protokoll den gesamten Verlauf wiedergibt, selbst wenn Sie es erst später öffnen.

### Auf das Debug-Protokoll zugreifen

1. Klicken Sie in der linken Seitenleiste auf das Symbol **Debug-Protokoll** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">


2. Das Protokollfenster wird geöffnet und zeigt Verarbeitungsmeldungen in Echtzeit an
3. Es erfolgt ein automatischer Bildlauf, um die neuesten Meldungen anzuzeigen

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Protokollmeldungen verstehen Den Protokollzeilen von

Chloros sind Tags in Klammern vorangestellt, die das Subsystem benennen – zum Beispiel `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Am wichtigsten ist die **Laufzusammenfassung**, die am Ende jedes Laufs (einschließlich abgebrochener Läufe) ausgegeben wird:

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Zusätzliche Hinweszeilen vom Typ `[RUN-SUMMARY]` folgen immer dann, wenn etwas erklärt werden muss – zum Beispiel bei einem Lauf, der keine Ergebnisse lieferte, oder eine Kamera, deren angefordertes Produkt als nicht anwendbar übersprungen wurde. `[EXPORT-CHECK]`-Zeilen erläutern kameraspezifische Überspringungen (z. B. warum eine RGB-Kamera kein Radiance-Produkt erhalten hat).

Die allgemeinen Schweregrade der Meldungen (die folgenden Beispiele dienen der Veranschaulichung und sind nicht wörtlich zu verstehen):

#### Informationsmeldungen (Weiß/Grau)

Normale Verarbeitungsaktualisierungen: Verarbeitung gestartet, Ziele erkannt (mit Panel-Anzahl), Fortschritt der Kalibrierung pro Bild, Dateien exportiert, Verarbeitung abgeschlossen.

#### Warnmeldungen (Gelb)

Nicht kritische Probleme, die die Verarbeitung nicht unterbrechen – z. B. fehlende GPS-Daten in einem Bild, eine große Lücke im Zeitstempel zwischen Zielbildern oder geringer Kontrast in einem Kalibrierfeld.

**Maßnahme:** Überprüfen Sie die Warnungen nach der Verarbeitung, unterbrechen Sie diese jedoch nicht

#### Fehlermeldungen (in Red)

Kritische Probleme, die zum Fehlschlagen der Verarbeitung führen können – z. B. volle Festplatte, eine beschädigte Bilddatei oder keine erkannten Ziele, obwohl eine Reflexionskalibrierung angefordert wurde.

**Maßnahme:** Verarbeitung stoppen, Fehler beheben, neu starten

### Häufige Situationen im Protokoll

| Situation                             | Bedeutung                                       | Erforderliche Maßnahme                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Ziel in \[Dateiname] erkannt        | Kalibrierungsziel erfolgreich gefunden         | Keine – normal                                         |
| Fortschrittsbalken pro Bild              | Aktuelle Fortschrittsanzeige                       | Keine – normal                                         |
| Keine Ziele gefunden                      | Keine Kalibrierungsziele erkannt               | Zielbilder markieren oder Reflexionskalibrierung deaktivieren |
| Unzureichender Speicherplatz               | Nicht genügend Speicherplatz für die Ausgabe                 | Speicherplatz freigeben                                    |
| Beschädigte Datei wird übersprungen               | Bilddatei ist beschädigt                         | Datei erneut von der SD-Karte kopieren                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Eine Aufnahme ohne RAW-Frame kann nicht verarbeitet werden | Erneut mit RAW aufnehmen, oder verwenden Sie die CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Der Lauf hat keine Bildprodukte erzeugt – als Fehler mit Hinweisen gemeldet | Lesen Sie die Hinweszeilen; prüfen Sie, was übersprungen wurde und warum |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Lange Pfade unter Windows sind deaktiviert und der Projektordner ist zu tief für die Pfadlänge von maximal 260 Zeichen; es wurde nichts verarbeitet | Verwenden Sie einen kürzeren Projektordner (in der Meldung wird der längste zulässige Pfad angegeben) oder aktivieren Sie die langen Pfade unter Windows und starten Sie Chloros neu – siehe [Starten der Verarbeitung](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Die Clamp-Maske eines LATTICE-Produkts konnte nicht geschrieben werden; das Produkt selbst ist in Ordnung und zeigt `Chloros:UnmixClampMask=unavailable` an | Überprüfen Sie den Speicherplatz und die Ordnerberechtigungen und führen Sie die Verarbeitung erneut durch, falls Sie die Maske benötigen |

### Protokolldaten kopieren

So kopieren Sie das Protokoll zur Fehlerbehebung oder für den Support:

1. Öffnen Sie den Bereich „Debug-Protokoll“.
2. Klicken Sie auf die Schaltfläche **Kopieren**(kopiert das Protokoll in die Zwischenablage) oder auf**Herunterladen**, um es als Textdatei zu speichern.
3. Fügen Sie den Inhalt in eine Textdatei oder eine E-Mail ein.
4. Senden Sie es bei Bedarf an den MAPIR-Support

### Einen Fehler über die Registerkarte „Debug-Protokoll“ melden

Neu in 1.3.0: Über das Fehlersymbol (**Fehler melden**) oben auf der Registerkarte „Debug-Protokoll“ wird ein Bericht direkt an MAPIR gesendet.

1. Klicken Sie auf das Fehlersymbol oben auf der Registerkarte „Debug-Protokoll“
2. Beschreiben Sie das Problem: Was Sie gerade getan haben, was Sie erwartet haben und was stattdessen passiert ist (erforderlich)
3. Aktivieren Sie optional **Einige Bilder hinzufügen (bis zu 10)**und wählen Sie Bilder aus dem geöffneten Projekt aus oder verwenden Sie**Durchsuchen...** für beliebige andere Bilddateien
4. Klicken Sie auf **Senden**

Der Bericht enthält Ihre Beschreibung, alle von Ihnen angehängten Bilder, die Chloros-Protokolle sowie Systemdetails (Chloros-Version, Betriebssystem, GPU und Projekteinstellungen). Sie müssen bei Chloros angemeldet sein (Registerkarte „Benutzer“), um einen Bericht senden zu können. Sie können das Fenster während des Sendevorgangs schließen; Sobald der Vorgang abgeschlossen ist, wird eine Referenznummer wie z. B. `Bug report sent. Reference: BR-…` in das Protokoll geschrieben – geben Sie diese an, wenn Sie sich an MAPIR wenden. Falls der Bericht nicht hochgeladen werden kann (z. B. wegen fehlender Internetverbindung), speichert Chloros ihn auf Ihrem Computer und zeigt den Pfad an, damit Sie ihn per E-Mail an support@mapir.camera senden können.

***

## Überwachung der Systemressourcen

### CPU-Auslastung

**Freier Modus:**

* 1 CPU-Kern bei ~100 %
* Andere Kerne im Leerlauf oder verfügbar
* Das System bleibt reaktionsfähig

**Parallelmodus von Chloros+:**

* Mehrere Kerne mit hoher Auslastung – die Anzahl hängt von der durch [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) gewählten Strategie ab
* Das System wirkt möglicherweise weniger reaktionsschnell

**Zur Überwachung:**

* Windows-Task-Manager (Strg+Umschalt+Esc)
* Registerkarte „Leistung“ → Abschnitt „CPU“
* Suchen Sie nach „Chloros “- oder „chloros-backend“-Prozessen

### Speicherauslastung (RAM)

**Typische Auslastung:**

* Kleine Projekte (< 100 Bilder): 2–4 GB
* Mittlere Projekte (100–500 Bilder): 4–8 GB
* Große Projekte (über 500 Bilder): 8–16 GB
* Der Parallelmodus von „Chloros+“ beansprucht mehr RAM

**Bei geringem Arbeitsspeicher:**

* Verarbeiten Sie kleinere Stapel
* Schließen Sie andere Anwendungen
* Erweitern Sie den Arbeitsspeicher, wenn Sie regelmäßig große Datensätze verarbeiten

### GPU-Auslastung („Chloros+“ mit CUDA)

Wenn die GPU-Beschleunigung aktiviert ist:

* Die NVIDIA-GPU weist eine hohe Auslastung auf (60–90 %)
* Die VRAM-Auslastung steigt (erfordert mindestens 4 GB VRAM; mindestens 7 GB für gleichzeitiges „Texture Aware“-Debayering)
* Die Kalibrierungsphase verläuft deutlich schneller

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
* Vermeiden Sie Netzlaufwerke bei großen Datensätzen
* Stellen Sie sicher, dass die Festplatte nicht fast voll ist (beeinträchtigt die Schreibgeschwindigkeit)

***

## Probleme während der Verarbeitung erkennen

### Warnzeichen

**Der Fortschritt stockt (keine Veränderung seit mehr als 5 Minuten):**

* Überprüfen Sie das Debug-Protokoll auf Fehler
* Überprüfen Sie den verfügbaren Festplattenspeicher
* Überprüfen Sie im Task-Manager, ob Chloros läuft

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
* ❌ Wiederholte Fehler wegen beschädigter Bilddateien
* ❌ Das System ist vollständig eingefroren (reagiert nicht)
* ❌ Sie haben festgestellt, dass falsche Einstellungen konfiguriert wurden
* ❌ Falsche Bilder wurden importiert

**So brechen Sie den Vorgang ab:**

1. Klicken Sie auf die**Schaltfläche „Stopp“** (ersetzt die Schaltfläche „Start“) – einmal reicht aus
2. Der Fortschrittsbalken zeigt „Wird angehalten...“ an, während das gerade bearbeitete Image fertiggestellt wird; anschließend endet der Durchlauf im angehaltenen Zustand
3. Bereits exportierte Produkte bleiben auf der Festplatte erhalten; das Protokoll gibt einen aussagekräftigen `[RUN-SUMMARY]`-Eintrag darüber aus, was abgeschlossen wurde
4. Beheben Sie die Probleme und starten Sie den Vorgang neu – der Lauf beginnt von vorne

***

## Fehlerbehebung während der Verarbeitung

### Die Verarbeitung ist sehr langsam

**Mögliche Ursachen:**

* Nicht markierte Zielbilder (alle Bilder werden gescannt)
* Festplattenspeicher statt SSD
* Unzureichende Systemressourcen
* Zu viele konfigurierte Indizes
* Zugriff auf ein Netzlaufwerk

**Lösungen:**

1. Wenn der Vorgang gerade erst gestartet wurde und sich in der Erkennungsphase befindet: Anhalten, Ziele markieren, neu starten
2. Für die Zukunft: SSD verwenden, Indizes reduzieren, Hardware aufrüsten
3. Für die Stapelverarbeitung großer Datensätze die CLI-Eingabe in Betracht ziehen

### Warnungen zum „Speicherplatz“

**Lösungen:**

1. Sofort Speicherplatz freigeben
2. Projekt auf ein Laufwerk mit mehr Speicherplatz verschieben
3. Anzahl der zu exportierenden Indizes reduzieren
4. Nicht benötigte LATTICE-Exportprodukte deaktivieren (Projekteinstellungen → Verarbeitung)
5. JPG-Format anstelle von TIFF verwenden (kleinere Dateien)

### Häufige Meldungen „Beschädigte Datei“

**Lösungen:**

1. Kopieren Sie die Bilder erneut von der SD-Karte, um deren Integrität sicherzustellen
2. Überprüfen Sie die SD-Karte auf Fehler
3. Entfernen Sie beschädigte Dateien aus dem Projekt
4. Setzen Sie die Verarbeitung der verbleibenden Bilder fort

### Überhitzung / Drosselung des Systems

**Lösungen:**

1. Für ausreichende Belüftung sorgen
2. Staub aus den Lüftungsschlitzen des Computers entfernen
3. Die Verarbeitungslast reduzieren (den „Free“-Modus anstelle von „Chloros+“ verwenden)
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
