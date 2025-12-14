# Projekteinstellungen

Die Projekteinstellungen <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in Chloros können Sie alle Aspekte der Bildverarbeitung, der Erkennung von Kalibrierungszielen, der Berechnung multispektraler Indizes und der Exportoptionen für Ihr Projekt konfigurieren. Diese Einstellungen werden mit Ihrem Projekt gespeichert und können als Vorlagen für die Wiederverwendung in mehreren Projekten gespeichert werden.

## Zugriff auf die Projekteinstellungen

So greifen Sie auf die Projekteinstellungen zu:

1. Öffnen Sie ein Projekt in Chloros
2. Klicken Sie auf die Registerkarte **Projekteinstellungen**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in der linken Seitenleiste.
3. Im Einstellungsfeld werden alle verfügbaren Konfigurationsoptionen nach Kategorien geordnet angezeigt.

***

## Zielerkennung

Mit diesen Einstellungen legen Sie fest, wie Chloros Kalibrierungsziele in Ihren Bildern erkennt und verarbeitet.

### Mindestfläche für Kalibrierungsproben (px)

* **Typ**: Zahl
* **Bereich**: 0 bis 10.000 Pixel
* **Standard**: 25 Pixel
* **Beschreibung**: Legt die Mindestfläche (in Pixel) fest, die erforderlich ist, damit ein erkannten Bereich als gültige Kalibrierungszielprobe gilt. Kleinere Werte erkennen kleinere Ziele, können jedoch zu mehr Fehlalarmen führen. Größere Werte erfordern größere, klarere Zielbereiche für die Erkennung.
* **Wann anpassen**:
  * Erhöhen Sie den Wert, wenn Sie falsche Erkennungen bei kleinen Bildartefakten erhalten.
  * Verringern Sie den Wert, wenn Ihre Kalibrierungsziele in Ihren Bildern klein erscheinen und nicht erkannt werden.

### Minimale Zielclusterung (0–100)

* **Typ**: Zahl
* **Bereich**: 0 bis 100
* **Standard**: 60
* **Beschreibung**: Steuert den Clustering-Schwellenwert für die Gruppierung ähnlich gefärbter Bereiche bei der Erkennung von Kalibrierungszielen. Höhere Werte erfordern, dass ähnlichere Farben gruppiert werden, was zu einer konservativeren Zielerkennung führt. Niedrigere Werte erlauben mehr Farbvariationen innerhalb einer Zielgruppe.
* **Wann anpassen**:
  * Erhöhen Sie den Wert, wenn Kalibrierungsziele in mehrere Erkennungen aufgeteilt werden.
  * Verringern Sie den Wert, wenn Kalibrierungsziele mit Farbvariationen nicht vollständig erkannt werden.

***

## Verarbeitung

Diese Einstellungen steuern, wie Chloros Ihre Bilder verarbeitet und kalibriert.

### Vignettierungskorrektur

* **Typ**: Kontrollkästchen
* **Standard**: Aktiviert (aktiviert)
* **Beschreibung**: Wendet eine Vignettierungskorrektur an, um die Verdunkelung des Objektivs an den Bildrändern auszugleichen. Vignettierung ist ein häufiges optisches Phänomen, bei dem die Ecken und Ränder eines Bildes aufgrund der Eigenschaften des Objektivs dunkler erscheinen als die Mitte.
* **Wann deaktivieren**: Nur deaktivieren, wenn Ihre Kamera-/Objektivkombination bereits eine Vignettierungskorrektur angewendet hat oder wenn Sie die Vignettierung in der Nachbearbeitung manuell korrigieren möchten.

### Reflektionskalibrierung / Weißabgleich

* **Typ**: Kontrollkästchen
* **Standard**: Aktiviert (aktiviert)
* **Beschreibung**: Aktiviert die automatische Reflektionskalibrierung anhand der in Ihren Bildern erkannten Kalibrierungsziele. Dadurch werden die Reflektionswerte in Ihrem Datensatz normalisiert und konsistente Messungen unabhängig von den Lichtverhältnissen gewährleistet.
* **Wann deaktivieren**: Deaktivieren Sie diese Option nur, wenn Sie rohe, unkalibrierte Bilder verarbeiten möchten oder wenn Sie einen anderen Kalibrierungsworkflow verwenden.

### Debayer-Verfahren

* **Typ**: Dropdown-Auswahl
* **Optionen**:
  * Hohe Qualität (schneller) – Derzeit die einzige verfügbare Option
* **Standard**: Hohe Qualität (schneller)
* **Beschreibung**: Wählt den Demosaicing-Algorithmus aus, der zur Konvertierung von rohen Bayer-Muster-Sensordaten in Vollfarbbilder verwendet wird. Die Methode „Hohe Qualität (schneller)” bietet ein optimales Gleichgewicht zwischen Verarbeitungsgeschwindigkeit und Bildqualität.
* **Hinweis**: In zukünftigen Versionen von Chloros können weitere Debayer-Methoden hinzugefügt werden.

### Mindestintervall für die Neukalibrierung

* **Typ**: Zahl
* **Bereich**: 0 bis 3.600 Sekunden
* **Standard**: 0 Sekunden
* **Beschreibung**: Legt das Mindestzeitintervall (in Sekunden) zwischen der Verwendung von Kalibrierungszielen fest. Bei einer Einstellung von 0 verwendet Chloros jedes erkannte Kalibrierungsziel. Bei einer höheren Einstellung verwendet Chloros nur Kalibrierungsziele, die mindestens diesen Zeitabstand voneinander haben, wodurch die Verarbeitungszeit für Datensätze mit häufigen Kalibrierungszielaufnahmen reduziert wird.
* **Wann anpassen**:
  * Stellen Sie den Wert auf 0 ein, um bei wechselnden Lichtverhältnissen eine maximale Kalibrierungsgenauigkeit zu erzielen.
  * Erhöhen Sie den Wert (z. B. auf 60–300 Sekunden), um die Verarbeitung zu beschleunigen, wenn die Lichtverhältnisse konstant sind und Sie häufig Kalibrierungszielbilder haben.

### Zeitzonenversatz des Lichtsensors

* **Typ**: Zahl
* **Bereich**: -12 bis +12 Stunden
* **Standard**: 0 Stunden
* **Beschreibung**: Gibt den Zeitzonenversatz (in Stunden gegenüber UTC) für Zeitstempel von Lichtsensordaten an. Dies wird bei der Verarbeitung von PPK-Datendateien (Post-Processed Kinematic) verwendet, um eine korrekte Zeitsynchronisation zwischen Bildaufnahmen und GPS-Daten sicherzustellen.
* **Wann anzupassen**: Stellen Sie diesen Wert auf Ihren lokalen Zeitzonenversatz ein, wenn Ihre PPK-Daten die Ortszeit anstelle von UTC verwenden. Beispiel:
  * Pazifische Zeit: -8 oder -7 (je nach Sommerzeit)
  * Ostküstenzeit: -5 oder -4 (je nach Sommerzeit)
  * Mitteleuropäische Zeit: +1 oder +2 (je nach Sommerzeit)

### PPK-Korrekturen anwenden

* **Typ**: Kontrollkästchen
* **Standard**: Deaktiviert (nicht markiert)
* **Beschreibung**: Ermöglicht die Verwendung von nachbearbeiteten kinematischen (PPK) Korrekturen aus MAPIR DAQ-Rekordern, die ein GPS (GNSS) enthalten. Wenn diese Option aktiviert ist, verwendet Chloros alle .daq-Protokolldateien, die Belichtungsstiftdaten in Ihrem Projektverzeichnis enthalten, und wendet präzise Geolokalisierungskorrekturen auf Ihre Bilder an.
* **Voraussetzung**: In Ihrem Projektverzeichnis muss eine .daq-Protokolldatei mit Belichtungs-Pin-Einträgen vorhanden sein.
* **Wann aktivieren**: Es wird empfohlen, die PPK-Korrektur immer zu aktivieren, wenn Ihre .daq-Protokolldatei Belichtungs-Feedback-Einträge enthält.

### Belichtungs-Pin 1

* **Typ**: Dropdown-Auswahl
* **Sichtbarkeit**: Nur sichtbar, wenn „PPK-Korrekturen anwenden” aktiviert ist UND Belichtungsdaten für Pin 1 verfügbar sind.
* **Optionen**:
  * Im Projekt erkannte Kameramodellnamen
  * „Nicht verwenden” – Diesen Belichtungs-Pin ignorieren
* **Standard**: Automatische Auswahl basierend auf der Projektkonfiguration
* **Beschreibung**: Weist dem Belichtungs-Pin 1 eine bestimmte Kamera für die PPK-Zeitsynchronisation zu. Der Belichtungs-Pin zeichnet den genauen Zeitpunkt auf, zu dem der Kameraverschluss ausgelöst wird, was für eine genaue PPK-Geolokalisierung entscheidend ist.
* **Verhalten bei automatischer Auswahl**:
  * Einzelne Kamera + einzelner Pin: Die Kamera wird automatisch ausgewählt.
  * Einzelne Kamera + zwei Pins: Pin 1 wird automatisch der Kamera zugewiesen.
  * Mehrere Kameras: Manuelle Auswahl erforderlich

### Belichtungs-Pin 2

* **Typ**: Dropdown-Auswahl
* **Sichtbarkeit**: Nur sichtbar, wenn „PPK-Korrekturen anwenden” aktiviert ist UND Belichtungsdaten für Pin 2 verfügbar sind
* **Optionen**:
  * Im Projekt erkannte Kameramodellnamen
  * „Nicht verwenden” – Dieser Belichtungs-Pin wird ignoriert
* **Standard**: Automatische Auswahl basierend auf der Projektkonfiguration
* **Beschreibung**: Weist dem Belichtungs-Pin 2 eine bestimmte Kamera für die PPK-Zeitsynchronisation zu, wenn eine Konfiguration mit zwei Kameras verwendet wird.
* **Verhalten bei automatischer Auswahl**:
  * Einzelne Kamera + einzelner Pin: Pin 2 wird automatisch auf „Nicht verwenden” gesetzt.
  * Einzelne Kamera + zwei Pins: Pin 2 wird automatisch auf „Nicht verwenden” gesetzt.
  * Mehrere Kameras: Manuelle Auswahl erforderlich
* **Hinweis**: Dieselbe Kamera kann nicht gleichzeitig Pin 1 und Pin 2 zugewiesen werden.

***

## Index

Mit diesen Einstellungen können Sie multispektrale Indizes für die Analyse und Visualisierung konfigurieren.

### Index hinzufügen

* **Typ**: Spezielles Indexkonfigurationsfeld
* **Beschreibung**: Öffnet ein interaktives Fenster, in dem Sie multispektrale Vegetationsindizes (NDVI, NDRE, EVI usw.) auswählen und konfigurieren können, die während der Bildverarbeitung berechnet werden sollen. Sie können mehrere Indizes hinzufügen, die jeweils eigene Visualisierungseinstellungen haben.
* **Verfügbare Indizes**: Das System umfasst mehr als 30 vordefinierte multispektrale Indizes, darunter:
  * NDVI (normalisierter Differenzvegetationsindex)
  * NDRE (normalisierter Differenzindex RedEdge)
  * EVI (Enhanced Vegetation Index)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Und viele mehr (siehe [Multispektrale Indexformeln](multispectral-index-formulas.md) für die vollständige Liste)
* **Funktionen**:
  * Auswahl aus vordefinierten Indexformeln
  * Konfiguration von Farbverläufen für die Visualisierung (LUT – Look-Up Tables)
  * Festlegung von Schwellenwerten für die Analyse
  * Erstellung benutzerdefinierter Indexformeln

### Benutzerdefinierte Formeln (Chloros+ Funktion)

* **Typ**: Array benutzerdefinierter Formeldefinitionen
* **Beschreibung**: Ermöglicht das Erstellen und Speichern benutzerdefinierter Multispektralindexformeln mithilfe von Bandmathematik. Benutzerdefinierte Formeln werden mit Ihren Projekteinstellungen gespeichert und können wie integrierte Indizes verwendet werden.
* **So erstellen Sie sie**:
  1. Suchen Sie im Indexkonfigurationsfeld nach der Option für benutzerdefinierte Formeln.
  2. Definieren Sie Ihre Formel mithilfe von Bandkennungen (z. B. NIR, Red, Green, Blue).
  3. Speichern Sie die Formel unter einem aussagekräftigen Namen.
* **Formelsyntax**: Es werden standardmäßige mathematische Operationen unterstützt, darunter:
  * Arithmetik: `+`, `-`, `*`, `/`
  * Klammern für die Reihenfolge der Operationen
  * Bandreferenzen: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Export

Diese Einstellungen steuern das Format und die Qualität der exportierten verarbeiteten Bilder.

### Kalibriertes Bildformat

* **Typ**: Dropdown-Auswahl
* **Optionen**:
  * **TIFF (16-Bit)** – Unkomprimiertes 16-Bit-TIFF-Format
  * **TIFF (32 Bit, Prozent)** – 32-Bit-Gleitkomma-TIFF mit Reflexionswerten in Prozent
  * **PNG (8 Bit)** – Komprimiertes 8-Bit-Format PNG
  * **JPG (8 Bit)** – Komprimiertes 8-Bit-Format JPEG
* **Standard**: TIFF (16 Bit)
* **Beschreibung**: Wählt das Dateiformat zum Speichern verarbeiteter und kalibrierter Bilder aus.
* **Formatempfehlungen**:
  * **TIFF (16 Bit)**: Empfohlen für wissenschaftliche Analysen und professionelle Arbeitsabläufe. Erhält maximale Datenqualität ohne Kompressionsartefakte. Am besten geeignet für multispektrale Analysen und die weitere Verarbeitung in GIS-Software.
  * **TIFF (32-Bit, Prozent)**: Am besten geeignet für Arbeitsabläufe, die Reflexionswerte als Prozentsätze (0–100 %) erfordern. Bietet maximale Präzision für radiometrische Messungen.
  * **PNG (8 Bit)**: Gut geeignet für die Anzeige im Internet und allgemeine Visualisierungen. Kleinere Dateigrößen mit verlustfreier Komprimierung, jedoch reduzierter Dynamikbereich.
  * **JPG (8 Bit)**: Kleinste Dateigrößen, am besten geeignet für Vorschauen und die Anzeige im Internet. Verwendet verlustbehaftete Komprimierung, die für wissenschaftliche Analysen nicht geeignet ist.

***

## Projektvorlage speichern

Mit dieser Funktion können Sie Ihre aktuellen Projekteinstellungen als wiederverwendbare Vorlage speichern.

* **Typ**: Texteingabe + Speichern-Schaltfläche
* **Beschreibung**: Geben Sie einen aussagekräftigen Namen für Ihre Einstellungsvorlage ein und klicken Sie auf das Speichern-Symbol. Die Vorlage speichert alle Ihre aktuellen Projekteinstellungen (Zielerkennung, Verarbeitungsoptionen, Indizes und Exportformat), sodass Sie diese in zukünftigen Projekten einfach wiederverwenden können.
* **Anwendungsfälle**:
  * Erstellen Sie Vorlagen für verschiedene Kamerasysteme (RGB, multispektral, NIR).
  * Speichern Sie Standardkonfigurationen für bestimmte Pflanzenarten oder Analyse-Workflows.
  * Teilen Sie einheitliche Einstellungen mit Ihrem Team.
* **Verwendung**:
  1. Konfigurieren Sie alle gewünschten Projekteinstellungen.
  2. Geben Sie einen Namen für die Vorlage ein (z. B. „RedEdge Survey3 NDVI Standard”).
  3. Klicken Sie auf das Speichersymbol.
  4. Die Vorlage kann nun beim Erstellen neuer Projekte geladen werden.

***

## Projektordner speichern

Diese Einstellung legt fest, wo neue Projekte standardmäßig gespeichert werden.

* **Typ**: Anzeige des Verzeichnispfads + Schaltfläche „Bearbeiten“
* **Standard**: `C:\Users\[Username]\Chloros Projects`
* **Beschreibung**: Zeigt das aktuelle Standardverzeichnis an, in dem neue Chloros-Projekte erstellt werden. Klicken Sie auf das Bearbeitungssymbol, um ein anderes Verzeichnis auszuwählen.
* **Wann ändern**:
  * Für die Zusammenarbeit im Team auf ein Netzlaufwerk einstellen.
  * Für große Datensätze auf ein Laufwerk mit mehr Speicherplatz ändern.
  * Projekte nach Jahr, Kunde oder Projekttyp in verschiedenen Ordnern organisieren.
* **Hinweis**: Das Ändern dieser Einstellung wirkt sich nur auf NEUE Projekte aus. Bestehende Projekte bleiben an ihrem ursprünglichen Speicherort.

***

## Speicherung der Einstellungen

Alle Projekteinstellungen werden automatisch mit Ihrer Projektdatei (`.mapir`-Projektformat) gespeichert. Wenn Sie ein Projekt erneut öffnen, werden alle Einstellungen genau so wiederhergestellt, wie Sie sie hinterlassen haben.

### Hierarchie der Einstellungen

Einstellungen werden in der folgenden Reihenfolge angewendet:

1. **Systemstandards** – Integrierte Standardeinstellungen, die von Chloros definiert wurden
2. **Vorlageneinstellungen** – Wenn Sie beim Erstellen eines Projekts eine Vorlage laden
3. **Gespeicherte Projekteinstellungen** – Mit der Projektdatei gespeicherte Einstellungen
4. **Manuelle Anpassungen** – Alle Änderungen, die Sie während der aktuellen Sitzung vornehmen

### Einstellungen und Bildverarbeitung

Die meisten Einstellungsänderungen (insbesondere in den Kategorien „Verarbeitung“ und „Export“) lösen eine erneute Verarbeitung der Bilder aus, um die neuen Einstellungen zu übernehmen. Einige Einstellungen sind jedoch „nur für den Export“ und erfordern keine sofortige erneute Verarbeitung:

* Projektvorlage speichern
* Arbeitsverzeichnis
* Kalibriertes Bildformat (gilt beim Exportieren)

***

## Bewährte Verfahren

1. **Mit Standardeinstellungen beginnen**: Die Standardeinstellungen eignen sich für die meisten MAPIR-Kamerasysteme und typischen Arbeitsabläufe.
2. **Vorlagen erstellen**: Wenn Sie die Einstellungen für einen bestimmten Arbeitsablauf oder eine bestimmte Kamera optimiert haben, speichern Sie sie als Vorlage, um die Konsistenz zwischen den Projekten sicherzustellen.
3. **Vor der vollständigen Verarbeitung testen**: Wenn Sie mit neuen Einstellungen experimentieren, testen Sie diese zunächst an einer kleinen Teilmenge von Bildern, bevor Sie Ihren gesamten Datensatz verarbeiten.
4. **Dokumentieren Sie Ihre Einstellungen**: Verwenden Sie aussagekräftige Vorlagennamen, die das Kamerasystem, die Verarbeitungsart und den Verwendungszweck angeben (z. B. „Survey3\_RGB\_NDVI\_Agriculture“).
5. **Auswahl des Exportformats**: Wählen Sie Ihr Exportformat entsprechend Ihrer Endanwendung:
   * Wissenschaftliche Analyse → TIFF (16-Bit oder 32-Bit)
   * GIS-Verarbeitung → TIFF (16-Bit)
   * Schnelle Visualisierung → PNG (8 Bit)
   * Webfreigabe → JPG (8 Bit)

***

Weitere Informationen zu Multispektralindizes in Chloros finden Sie auf der Seite [Multispektralindexformeln](multispectral-index-formulas.md).
