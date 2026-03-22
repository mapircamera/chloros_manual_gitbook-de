# Ein Bild im Vollbildmodus öffnen

Der Chloros Image Viewer bietet eine spezielle Vollbildoberfläche zum Anzeigen, Analysieren und Bearbeiten Ihrer Multispektralbilder. Ob Sie nun Originalbilder oder verarbeitete Ergebnisse betrachten – der Image Viewer bietet leistungsstarke Werkzeuge für die Inspektion und Analyse.

## Aufrufen des Bildbetrachters

### Über den Dateibrowser

Die gängigste Methode zum Öffnen eines Bildes im Bildbetrachter:

1. Vergewissern Sie sich, dass Sie sich auf der Registerkarte **Dateibrowser** befinden <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klicken Sie auf eine beliebige **Bildminiaturansicht** im Bildraster
3. Das Bild wird im **Hauptvorschau-Bereich** (Mitte des Bildschirms) geöffnet
4. Das Bild ist nun geladen und bereit für die Vollbildanzeige

### Öffnen der Registerkarte „Bildbetrachter“

Sobald ein Bild im Vorschau-Bereich geladen ist:

1. Klicken Sie auf das Symbol **Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> in der linken Seitenleiste
2. Die Registerkarte „Bildbetrachter“ wird geöffnet und zeigt das ausgewählte Bild im Vollbildmodus an
3. In der linken Seitenleiste stehen erweiterte Anzeige- und Analysewerkzeuge zur Verfügung

***

## Übersicht über die Benutzeroberfläche des Bildbetrachters

### Hauptanzeigebereich

Der größte Teil des Bildschirms zeigt Ihr Bild an:

* **Volle Auflösung**: Bilder werden in ihrer nativen Auflösung angezeigt
* **Zoomfähig**: Verwenden Sie die Steuerelemente oder das Mausrad zum Zoomen
* **Verschiebbar**: Klicken und ziehen Sie, um sich im gezoomten Zustand zu bewegen
* **Seitenverhältnis beibehalten**: Bilder werden proportional skaliert***

## Anzeigeoptionen

### Grundlegende Bildnavigation

#### Durch Bilder blättern

Navigieren Sie mithilfe von Tastaturkürzeln oder Schaltflächen durch Ihren Bildsatz:

* **Nächstes Bild**: Klicken Sie auf die Schaltfläche → oder drücken Sie die Taste**→** (Pfeil nach rechts)
* **Vorheriges Bild**: Klicken Sie auf die Schaltfläche ← oder drücken Sie die Taste**←** (Pfeil nach links)
* **Zu einem bestimmten Bild springen**: Kehren Sie zum Dateibrowser zurück und klicken Sie auf die gewünschte Miniaturansicht

#### Zoom-Steuerelemente

Passen Sie die Vergrößerung an, um Bilddetails zu betrachten:

**Vergrößern:*** Klicken Sie auf die Schaltfläche **+** (Plus)
* Drücken Sie die Taste **+**oder**=*** Scrollen Sie mit dem Mausrad **nach oben**

**Verkleinern:*** Klicken Sie auf die Schaltfläche **−** (Minus)
* Drücken Sie die Taste **−** (Minus)
* Scrollen Sie mit dem Mausrad **nach unten**

#### Schwenken bei vergrößerter Ansicht

Wenn über die Bildschirmgröße hinaus vergrößert wurde:

1. Bewegen Sie den Mauszeiger über das Bild
2. Klicken Sie und **halten Sie die linke Maustaste gedrückt**

3.**Ziehen** Sie, um das Bild zu verschieben
4. Lassen Sie die Taste los, um das Schwenken zu beenden

**Alternative**: Verwenden Sie die Pfeiltasten, um in kleinen Schritten zu schwenken***

## Überprüfung der Pixelwerte

### Anzeigen der Pixelwerte am Cursor

Wenn Sie den Mauszeiger über das Bild bewegen, werden die Pixelwerte in Echtzeit angezeigt:**Anzeigeposition der Werte:*** **Schwebende Zahl und rote Linie in der rechten Index-LUT-Gradientenlegende*** **Bei weiterer Vergrößerung: schwebender Wert in der Nähe des Cursors und des markierten Pixels*** Zeigt Werte für das Pixel **unter dem Cursor oder das markierte Pixel** an
* Aktualisierung beim Bewegen der Maus

***

## Bildtypen, die Sie anzeigen können

### JPG

**JPG-Bilder von der Kamera:**

* Anzeige der JPG-Daten als Vorschau
* Anzeige der ursprünglichen, unkorrigierten Werte
* Nützlich zur Überprüfung der Bildqualität vor der Bearbeitung

### RAW (Original)

### RAW (Reflexionsgrad)

**Nach der Verarbeitung:**

* Vignettierung korrigiert
* Reflexion kalibriert
* Multiband-TIFF (Red, Green, NIR usw.)
* Wissenschaftliche Daten bereit für die Analyse

### RAW (Index)

**NDVI, NDRE, GNDVI usw. (\_NDVI.tif-Dateien):**

* Einband-Graustufenbilder
* Pixelwerte stellen die Ergebnisse der Indexberechnung dar
* Bereich typischerweise -1 bis +1 für normalisierte Indizes
* Es können Farb-LUTs zur Visualisierung angewendet werden

***

## Anwendung von Indizes und LUTs

Anwendung von Multispektralindizes und Farb-Look-Up-Tabellen:

1. Suchen Sie **Index/LUT Sandbox**im**Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> Seitenleiste
2. Vegetationsindex auswählen (NDVI, NDRE usw.)
3. Multispektrale Formel auswählen oder eigene benutzerdefinierte Formel erstellen (nur Chloros+)
4. Farb-LUT-Verlauf zur Visualisierung anwenden
5. Wertebereiche und Schwellenwerte anpassen

Detaillierte Anweisungen finden Sie unter [Index/LUT-Sandbox](index-lut-sandbox.md).

***

## Tastaturkürzel

### Navigation

* **→** (Pfeil nach rechts): Nächstes Bild
* **←** (Pfeil nach links): Vorheriges Bild
* **Home**: Erstes Bild in der Liste
* **Ende**: Letztes Bild in der Liste

### Zoomen

* **+**oder**=**: Vergrößern
* **−**: Verkleinern
* **Mausrad**: Vergrößern/Verkleinern***

### Überprüfung der Indexberechnungen

Überprüfen Sie, ob die Indizes korrekt berechnet wurden:

1. Öffnen Sie NDVI oder ein anderes Indexbild
2. Überprüfen Sie die Vegetationsflächen:
   * **NDVI**: Sollte bei gesunden Pflanzen 0,4–0,9 anzeigen
   * **NDRE**: Höhere Werte bei kräftigem Wachstum
   * **GNDVI**: Ähnlich wie NDVI, jedoch chlorophyllsensitiv
3. Nicht-Vegetation überprüfen:
   * **Boden**: Nahe 0 oder leicht negativ
   * **Wasser**: Negative Werte (-0,5 bis 0)***

## Fehlerbehebung bei Anzeige-Problemen

### Bild lässt sich nicht öffnen

**Mögliche Ursachen:**

* Datei während der Verarbeitung beschädigt
* Nicht unterstütztes Dateiformat
* Unzureichender Speicher für großes Bild

**Lösungen:**

1. Versuchen Sie, die Datei in einem externen Bildbetrachter zu öffnen, um die Dateiintegrität zu überprüfen
2. Überprüfen Sie, ob das Dateiformat dem erwarteten Typ entspricht
3. Schließen Sie andere Anwendungen, um Speicherplatz freizugeben
4. Versuchen Sie es mit einem kleineren/anderen Bild

### Schwarz-Weiß-Anzeige des Bildes

**Mögliche Ursachen:**

* Wertebereich außerhalb der Anzeigekapazität
* 32-Bit-Float-Bild mit ungewöhnlichen Werten
* Fehler bei der Indexberechnung

**Lösungen:**

1. Überprüfen Sie die Pixelwerte – wenn alle sehr niedrig oder sehr hoch sind, passen Sie den Anzeigebereich an
2. Versuchen Sie, die Datei in QGIS oder einem ähnlichen Programm mit automatischer Bereichsanpassung zu öffnen
3. Überprüfen Sie das Debug-Protokoll der Verarbeitung auf Fehler

### Pixelwerte scheinen falsch zu sein

**Mögliche Ursachen:**

* Falsches Bild wird angezeigt (Original vs. verarbeitet)
* Kalibrierung wurde nicht korrekt angewendet
* Lichtsensordaten wurden nicht in die Eingabe einbezogen
* Prozentmodus wurde falsch umgeschaltet

**Lösungen:**

1. Vergewissern Sie sich, dass Sie die verarbeitete Ausgabe anzeigen (überprüfen Sie die Dateinamenerweiterung)
2. Überprüfen Sie den Status der Schaltfläche für den Prozentmodus
3. Vergleichen Sie mit bekanntermaßen korrekten Bildern aus demselben Datensatz

***

## Nächste Schritte

Nachdem Sie nun Bilder im Vollbildmodus anzeigen können:

* [**Bildebenen**](image-layers.md) – Erfahren Sie mehr über die Multiband-Visualisierung
* [**Index/LUT-Sandbox**](index-lut-sandbox.md) – Wenden Sie benutzerdefinierte Indizes und Farbabbildungen an
* [**Multispektrale Indexformeln**](../project-settings/multispectral-index-formulas.md) – Erfahren Sie mehr über verfügbare Indizes

Informationen zum Verarbeitungsworkflow finden Sie unter:

* [**Bilder verarbeiten (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Vollständige Anleitung zur Bildverarbeitung
