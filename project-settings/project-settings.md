# Projekteinstellungen

Über die Seitenleiste „Projekteinstellungen“ <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in Chloros ermöglicht es Ihnen, alle Aspekte der Bildverarbeitung, der Erkennung von Kalibrierungszielen, der Berechnung multispektraler Indizes und der Exportoptionen für Ihr Projekt zu konfigurieren. Diese Einstellungen werden mit Ihrem Projekt gespeichert und können als Vorlagen gespeichert werden, um sie in mehreren Projekten wiederzuverwenden.

## Aufrufen der Projekteinstellungen

So rufen Sie die Projekteinstellungen auf:

1. Öffnen Sie ein Projekt in Chloros
2. Klicken Sie auf die Registerkarte **Projekteinstellungen**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in der linken Seitenleiste
3. Im Einstellungsfenster werden alle verfügbaren Konfigurationsoptionen nach Kategorien geordnet angezeigt

***

## Zielerkennung

Diese Einstellungen steuern, wie Chloros Kalibrierungsziele in Ihren Bildern erkennt und verarbeitet.

### Mindestfläche für Kalibrierungsproben (px)

* **Typ**: Zahl
* **Bereich**: 0 bis 10.000 Pixel
* **Standard**: 25 Pixel
* **Beschreibung**: Legt die Mindestfläche (in Pixeln) fest, die erforderlich ist, damit eine erkannte Region als gültige Kalibrierungszielprobe gilt. Kleinere Werte erkennen kleinere Ziele, können jedoch zu mehr Fehlalarmen führen. Größere Werte erfordern größere, klarere Zielregionen für die Erkennung.
* **Wann anpassen**:
  * Erhöhen Sie den Wert, wenn Sie falsche Erkennungen bei kleinen Bildartefakten erhalten.
  * Verringern Sie den Wert, wenn Ihre Kalibrierungsziele in Ihren Bildern klein erscheinen und nicht erkannt werden.

### Minimale Zielclusterung (0–100)

* **Typ**: Zahl
* **Bereich**: 0 bis 100
* **Standard**: 60
* **Beschreibung**: Steuert den Clustering-Schwellenwert für die Gruppierung ähnlich gefärbter Bereiche bei der Erkennung von Kalibrierungszielen. Höhere Werte erfordern, dass mehr ähnliche Farben zusammengefasst werden, was zu einer konservativeren Zielerkennung führt. Niedrigere Werte lassen mehr Farbabweichungen innerhalb einer Zielgruppe zu.
* **Wann anpassen**:
  * Erhöhen Sie den Wert, wenn Kalibrierungsziele in mehrere Erkennungen aufgeteilt werden
  * Verringern Sie den Wert, wenn Kalibrierungsziele mit Farbabweichungen nicht vollständig erkannt werden

***

## Verarbeitung

Diese Einstellungen steuern, wie Chloros Ihre Bilder verarbeitet und kalibriert.

### Vignettierungskorrektur

* **Typ**: Kontrollkästchen
* **Standard**: Aktiviert (aktiviert)
* **Beschreibung**: Wendet eine Vignettierungskorrektur an, um die Verdunkelung an den Bildrändern durch das Objektiv auszugleichen. Vignettierung ist ein häufiges optisches Phänomen, bei dem die Ecken und Ränder eines Bildes aufgrund der Eigenschaften des Objektivs dunkler erscheinen als die Bildmitte.
* **Wann deaktivieren**: Deaktivieren Sie diese Option nur, wenn Ihre Kamera-/Objektivkombination bereits eine Vignettierungskorrektur angewendet hat oder wenn Sie die Vignettierung manuell in der Nachbearbeitung korrigieren möchten.

### Reflektanzkalibrierung / Weißabgleich

* **Typ**: Kontrollkästchen
* **Standard**: Aktiviert (markiert)
* **Beschreibung**: Aktiviert die automatische Reflektanzkalibrierung unter Verwendung der in Ihren Bildern erkannten Kalibrierungsziele. Dies normalisiert die Reflektanzwerte in Ihrem Datensatz und gewährleistet konsistente Messungen unabhängig von den Lichtverhältnissen.
* **Wann deaktivieren**: Deaktivieren Sie diese Option nur, wenn Sie rohe, unkalibrierte Bilder verarbeiten möchten oder wenn Sie einen anderen Kalibrierungs-Workflow verwenden.

### Demosaicing-Methode

* **Typ**: Dropdown-Auswahl
* **Optionen**:
  * Standard (schnell, mittlere Qualität)
  * Texturbewusst (langsam, höchste Qualität) \[Chloros+]
* **Standard**: Standard (schnell, mittlere Qualität)
* **Beschreibung**: Wählt den Demosaicing-Algorithmus aus, der verwendet wird, um rohe Daten von Sensoren mit Bayer-Muster in Vollfarbbilder umzuwandeln. Die Methode „Standard (schnell, mittlere Qualität)“ bietet ein optimales Gleichgewicht zwischen Verarbeitungsgeschwindigkeit und Bildqualität. Die Methode „Texture Aware (Langsam, Höchste Qualität)“ \[Chloros+] verwendet ein hochwertiges, kantenbewusstes Demosaicing in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Demosaicing-Rauschen entfernt. Das Texture-Aware-Modell benötigt zur Ausführung GPU-Speicher (VRAM). Wir empfehlen die Verwendung, wenn Ihnen &gt;4 GB VRAM für eine schnellere Verarbeitung zur Verfügung stehen.
* **Hinweis**: In zukünftigen Versionen von Chloros werden möglicherweise weitere Debayer-Methoden hinzugefügt.

### Minimales Neukalibrierungsintervall

* **Typ**: Zahl
* **Bereich**: 0 bis 3.600 Sekunden
* **Standard**: 0 Sekunden
* **Beschreibung**: Legt das minimale Zeitintervall (in Sekunden) zwischen der Verwendung von Kalibrierungszielen fest. Bei einer Einstellung von 0 verwendet Chloros jedes erkannte Kalibrierungsziel. Bei einer höheren Einstellung verwendet Chloros nur Kalibrierungsziele, die mindestens diesen Zeitabstand voneinander haben, wodurch die Verarbeitungszeit für Datensätze mit häufigen Kalibrierungszielerfassungen reduziert wird.
* **Wann anpassen**:
  * Setzen Sie den Wert auf 0, um bei wechselnden Lichtverhältnissen maximale Kalibrierungsgenauigkeit zu erzielen.
  * Erhöhen Sie den Wert (z. B. auf 60–300 Sekunden), um die Verarbeitung zu beschleunigen, wenn die Lichtverhältnisse konstant sind und Sie häufige Kalibrierungszielbilder haben.

### Zeitzonenversatz des Lichtsensors

* **Typ**: Zahl
* **Bereich**: -12 bis +12 Stunden
* **Standard**: 0 Stunden
* **Beschreibung**: Legt den Zeitzonen-Offset (in Stunden gegenüber UTC) für Zeitstempel von Lichtsensordaten fest. Dies wird bei der Verarbeitung von PPK-Datendateien (Post-Processed Kinematic) verwendet, um eine korrekte Zeitsynchronisation zwischen Bildaufnahmen und GPS-Daten sicherzustellen.
* **Wann anpassen**: Stellen Sie diesen Wert auf Ihren lokalen Zeitzonen-Offset ein, wenn Ihre PPK-Daten die Ortszeit anstelle von UTC verwenden. Beispiel:
  * Pazifikzeit: -8 oder -7 (je nach Sommerzeit)
  * Ostküstenzeit: -5 oder -4 (je nach Sommerzeit)
  * Mitteleuropäische Zeit: +1 oder +2 (je nach Sommerzeit)

### PPK-Korrekturen anwenden

* **Typ**: Kontrollkästchen
* **Standard**: Deaktiviert (nicht markiert)
* **Beschreibung**: Aktiviert die Verwendung von PPK-Korrekturen (Post-Processed Kinematic) aus MAPIR-DAQ-Recordern, die über ein GPS (GNSS) verfügen. Wenn diese Option aktiviert ist, verwendet Chloros alle .daq-Protokolldateien mit Belichtungs-Pin-Daten in Ihrem Projektverzeichnis und wendet präzise Geolokalisierungs-Korrekturen auf Ihre Bilder an.
* **Voraussetzung**: In Ihrem Projektverzeichnis muss eine .daq-Protokolldatei mit Belichtungspin-Einträgen vorhanden sein
* **Wann aktivieren**: Es wird empfohlen, die PPK-Korrektur immer zu aktivieren, wenn Ihre .daq-Protokolldatei Belichtungs-Feedback-Einträge enthält.

### Belichtungspin 1

* **Typ**: Dropdown-Auswahl
* **Sichtbarkeit**: Nur sichtbar, wenn „PPK-Korrekturen anwenden“ aktiviert ist UND Belichtungsdaten für Pin 1 verfügbar sind
* **Optionen**:
  * Im Projekt erkannte Kameramodellnamen
  * „Nicht verwenden“ – Diesen Belichtungspin ignorieren
* **Standard**: Wird automatisch basierend auf der Projektkonfiguration ausgewählt
* **Beschreibung**: Weist dem Belichtungs-Pin 1 eine bestimmte Kamera für die PPK-Zeitsynchronisation zu. Der Belichtungs-Pin zeichnet den genauen Zeitpunkt auf, zu dem der Kameraverschluss ausgelöst wird, was für eine genaue PPK-Geolokalisierung entscheidend ist.
* **Verhalten bei der automatischen Auswahl**:
  * Einzelne Kamera + einzelner Pin: Wählt automatisch die Kamera aus
  * Einzelne Kamera + zwei Pins: Pin 1 wird automatisch der Kamera zugewiesen
  * Mehrere Kameras: Manuelle Auswahl erforderlich

### Belichtungspin 2

* **Typ**: Dropdown-Auswahl
* **Sichtbarkeit**: Nur sichtbar, wenn „PPK-Korrekturen anwenden“ aktiviert ist UND Belichtungsdaten für Pin 2 verfügbar sind
* **Optionen**:
  * Im Projekt erkannte Kameramodellnamen
  * „Nicht verwenden“ – Diesen Belichtungs-Pin ignorieren
* **Standard**: Wird basierend auf der Projektkonfiguration automatisch ausgewählt
* **Beschreibung**: Weist dem Belichtungs-Pin 2 eine bestimmte Kamera für die PPK-Zeitsynchronisation zu, wenn eine Konfiguration mit zwei Kameras verwendet wird.
* **Verhalten bei der automatischen Auswahl**:
  * Einzelne Kamera + einzelner Pin: Pin 2 wird automatisch auf „Nicht verwenden“ gesetzt
  * Einzelne Kamera + zwei Pins: Pin 2 wird automatisch auf „Nicht verwenden“ gesetzt
  * Mehrere Kameras: Manuelle Auswahl erforderlich
* **Hinweis**: Dieselbe Kamera kann nicht gleichzeitig sowohl Pin 1 als auch Pin 2 zugewiesen werden.***

## Index

Mit diesen Einstellungen können Sie multispektrale Indizes für die Analyse und Visualisierung konfigurieren.

### Index hinzufügen

* **Typ**: Spezielles Konfigurationsfenster für Indizes
* **Beschreibung**: Öffnet ein interaktives Fenster, in dem Sie multispektrale Vegetationsindizes (NDVI, NDRE, EVI usw.) auswählen und konfigurieren können, die während der Bildverarbeitung berechnet werden sollen. Sie können mehrere Indizes hinzufügen, jeder mit eigenen Visualisierungseinstellungen.
* **Verfügbare Indizes**: Das System enthält über 30 vordefinierte multispektrale Indizes, darunter:
  * NDVI (Normalisierter Differenzvegetationsindex)
  * NDRE (Normalisierte Differenz RedEdge)
  * EVI (Enhanced Vegetation Index)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Und viele weitere (siehe [Multispektrale Indexformeln](multispectral-index-formulas.md) für die vollständige Liste)
* **Funktionen**:
  * Auswahl aus vordefinierten Indexformeln
  * Konfiguration von Farbverläufen für die Visualisierung (LUT – Look-Up-Tabellen)
  * Festlegen von Schwellenwerten für die Analyse
  * Erstellen benutzerdefinierter Indexformeln

### Benutzerdefinierte Formeln (Chloros+-Funktion)

* **Typ**: Array mit Definitionen benutzerdefinierter Formeln
* **Beschreibung**: Ermöglicht das Erstellen und Speichern benutzerdefinierter multispektraler Indexformeln mithilfe von Bandmathematik. Benutzerdefinierte Formeln werden mit Ihren Projekteinstellungen gespeichert und können genau wie integrierte Indizes verwendet werden.
* **So erstellen Sie sie**:
  1. Suchen Sie im Konfigurationsfenster „Index“ nach der Option für benutzerdefinierte Formeln
  2. Definieren Sie Ihre Formel unter Verwendung von Bandkennungen (z. B. NIR, Red, Green, Blue)
  3. Speichern Sie die Formel unter einem aussagekräftigen Namen
* **Formelsyntax**: Es werden standardmäßige mathematische Operationen unterstützt, darunter:
  * Arithmetik: `+`, `-`, `*`, `/`
  * Klammern zur Festlegung der Rechenfolge
  * Bandreferenzen: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Export

Diese Einstellungen steuern das Format und die Qualität der exportierten, bearbeiteten Bilder.

### Kalibriertes Bildformat

* **Typ**: Dropdown-Auswahl
* **Optionen**:
  * **TIFF (16-Bit)** – Unkomprimiertes 16-Bit-TIFF-Format
  * **TIFF (32-Bit, Prozent)** – 32-Bit-Gleitkomma-TIFF mit Reflexionswerten in Prozent
  * **PNG (8-Bit)** – Komprimiertes 8-Bit-PNG-Format
  * **JPG (8-Bit)** – Komprimiertes 8-Bit-JPEG-Format
* **Standard**: TIFF (16-Bit)
* **Beschreibung**: Wählt das Dateiformat zum Speichern bearbeiteter und kalibrierter Bilder aus.
* **Formatempfehlungen**:
  * **TIFF (16-Bit)**: Empfohlen für wissenschaftliche Analysen und professionelle Arbeitsabläufe. Bewahrt maximale Datenqualität ohne Kompressionsartefakte. Am besten geeignet für multispektrale Analysen und die Weiterverarbeitung in GIS-Software.
  * **TIFF (32-Bit, Prozent)**: Am besten geeignet für Arbeitsabläufe, die Reflexionswerte als Prozentsätze (0–100 %) erfordern. Bietet maximale Präzision für radiometrische Messungen.
  * **PNG (8-Bit)**: Gut geeignet für die Webanzeige und allgemeine Visualisierung. Geringere Dateigrößen durch verlustfreie Komprimierung, jedoch reduzierter Dynamikbereich.
  * **JPG (8-Bit)**: Kleinste Dateigrößen, am besten nur für Vorschauen und die Anzeige im Web geeignet. Verwendet verlustbehaftete Komprimierung, die für wissenschaftliche Analysen nicht geeignet ist.***

## Projektvorlage speichern

Mit dieser Funktion können Sie Ihre aktuellen Projekteinstellungen als wiederverwendbare Vorlage speichern.

* **Typ**: Texteingabe + Speichern-Schaltfläche
* **Beschreibung**: Geben Sie einen aussagekräftigen Namen für Ihre Einstellungsvorlage ein und klicken Sie auf das Speichern-Symbol. Die Vorlage speichert alle Ihre aktuellen Projekteinstellungen (Zielerkennung, Verarbeitungsoptionen, Indizes und Exportformat) zur einfachen Wiederverwendung in zukünftigen Projekten.
* **Anwendungsfälle**:
  * Erstellen Sie Vorlagen für verschiedene Kamerasysteme (RGB, Multispektral, NIR)
  * Speichern Sie Standardkonfigurationen für bestimmte Kulturarten oder Analyse-Workflows
  * Geben Sie einheitliche Einstellungen im Team weiter
* **Anwendung**:
  1. Konfigurieren Sie alle gewünschten Projekteinstellungen
  2. Geben Sie einen Vorlagennamen ein (z. B. „RedEdge Survey3 NDVI Standard“)
  3. Klicken Sie auf das Speichersymbol
  4. Die Vorlage kann nun beim Erstellen neuer Projekte geladen werden

***

## Projektordner speichern

Diese Einstellung legt fest, wo neue Projekte standardmäßig gespeichert werden.

* **Typ**: Anzeige des Verzeichnispfads + Schaltfläche „Bearbeiten“
* **Standard (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Standard (Linux)**: `~/.local/share/chloros/projects`
* **Beschreibung**: Zeigt das aktuelle Standardverzeichnis an, in dem neue Chloros-Projekte erstellt werden. Klicken Sie auf das Bearbeitungssymbol, um ein anderes Verzeichnis auszuwählen.
* **Wann ändern**:
  * Legen Sie ein Netzlaufwerk fest, um die Zusammenarbeit im Team zu ermöglichen
  * Wechseln Sie zu einem Laufwerk mit mehr Speicherplatz für große Datensätze
  * Organisieren Sie Projekte nach Jahr, Kunde oder Projekttyp in verschiedenen Ordnern
* **Hinweis**: Das Ändern dieser Einstellung wirkt sich nur auf NEUE Projekte aus. Bestehende Projekte verbleiben an ihren ursprünglichen Speicherorten.***

## Beibehaltung der Einstellungen

Alle Projekteinstellungen werden automatisch mit Ihrer Projektdatei (`.mapir`-Projektformat) gespeichert. Wenn Sie ein Projekt erneut öffnen, werden alle Einstellungen genau so wiederhergestellt, wie Sie sie hinterlassen haben.

### Einstellungshierarchie

Einstellungen werden in der folgenden Reihenfolge angewendet:

1. **Systemstandards** – Integrierte Standardwerte, definiert durch Chloros
2. **Vorlageneinstellungen** – Wenn Sie beim Erstellen eines Projekts eine Vorlage laden
3. **Gespeicherte Projekteinstellungen** – Mit der Projektdatei gespeicherte Einstellungen
4. **Manuelle Anpassungen** – Alle Änderungen, die Sie während der aktuellen Sitzung vornehmen

### Einstellungen und Bildverarbeitung

Die meisten Änderungen an den Einstellungen (insbesondere in den Kategorien „Verarbeitung“ und „Export“) lösen eine erneute Verarbeitung der Bilder aus, um die neuen Einstellungen zu berücksichtigen. Einige Einstellungen sind jedoch „nur für den Export“ bestimmt und erfordern keine sofortige Neuverarbeitung:

* Projektvorlage speichern
* Arbeitsverzeichnis
* Kalibriertes Bildformat (gilt beim Export)

***

## Bewährte Vorgehensweisen

1. **Beginnen Sie mit den Standardeinstellungen**: Die Standardeinstellungen eignen sich gut für die meisten MAPIR-Kamerasysteme und typische Arbeitsabläufe.
2. **Erstellen Sie Vorlagen**: Sobald Sie die Einstellungen für einen bestimmten Workflow oder eine bestimmte Kamera optimiert haben, speichern Sie diese als Vorlage, um die Konsistenz über Projekte hinweg sicherzustellen.
3. **Vor der vollständigen Verarbeitung testen**: Wenn Sie mit neuen Einstellungen experimentieren, testen Sie diese zunächst an einer kleinen Auswahl von Bildern, bevor Sie Ihren gesamten Datensatz verarbeiten.
4. **Dokumentieren Sie Ihre Einstellungen**: Verwenden Sie aussagekräftige Vorlagennamen, die das Kamerasystem, die Verarbeitungsart und den Verwendungszweck angeben (z. B. „Survey3\_RGB\_NDVI\_Landwirtschaft“).
5. **Auswahl des Exportformats**: Wählen Sie Ihr Exportformat entsprechend Ihrer Endanwendung:
   * Wissenschaftliche Analyse → TIFF (16-Bit oder 32-Bit)
   * GIS-Verarbeitung → TIFF (16-Bit)
   * Schnelle Visualisierung → PNG (8-Bit)
   * Web-Freigabe → JPG (8-Bit)

***

Weitere Informationen zu multispektralen Indizes in Chloros finden Sie auf der Seite [Formeln für multispektrale Indizes](multispectral-index-formulas.md).
