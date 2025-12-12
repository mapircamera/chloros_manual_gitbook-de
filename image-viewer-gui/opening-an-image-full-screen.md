# Bild im Vollbildmodus öffnen

Der Chloros Image Viewer bietet eine spezielle Vollbildschnittstelle zum Anzeigen, Analysieren und Bearbeiten Ihrer Multispektralbilder. Unabhängig davon, ob Sie Originalbilder oder verarbeitete Ausgaben anzeigen, bietet der Image Viewer leistungsstarke Tools für die Überprüfung und Analyse.

## Aufrufen des Bildbetrachters

### Über den Dateibrowser

Die gängigste Methode zum Öffnen eines Bildes im Bildbetrachter:

1. Vergewissern Sie sich, dass Sie sich auf der Registerkarte **Dateibrowser** befinden. <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klicken Sie auf eine beliebige **Bildminiaturansicht** im Bildraster.
3. Das Bild wird im **Hauptvorschau-Bereich** (Mitte des Bildschirms) geöffnet.
4. Das Bild ist nun geladen und kann im Vollbildmodus angezeigt werden.

### Öffnen der Registerkarte „Bildbetrachter“

Sobald ein Bild im Vorschau-Bereich geladen ist:

1. Klicken Sie auf das Symbol „**Bildbetrachter**“ <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> in der linken Seitenleiste.
2. Die Registerkarte „Bildbetrachter“ wird geöffnet und zeigt das ausgewählte Bild im Vollbildmodus an.
3. In der linken Seitenleiste stehen erweiterte Anzeige- und Analysewerkzeuge zur Verfügung.

***

## Übersicht über die Benutzeroberfläche des Bildbetrachters

### Hauptanzeigebereich

Der größte Teil des Bildschirms zeigt Ihr Bild an:

* **Volle Auflösung**: Bilder werden in ihrer nativen Auflösung angezeigt.
* **Zoombar**: Verwenden Sie die Steuerelemente oder das Mausrad, um zu zoomen
* **Verschiebbar**: Klicken und ziehen Sie, um sich im gezoomten Bild zu bewegen
* **Seitenverhältnis beibehalten**: Bilder werden proportional skaliert

***

## Anzeigeoptionen

### Grundlegende Bildnavigation

#### Durch Bilder blättern

Navigieren Sie mit Tastaturkürzeln oder Schaltflächen durch Ihren Bildsatz:

* **Nächstes Bild**: Klicken Sie auf die Schaltfläche → oder drücken Sie die Taste **→** (Pfeil nach rechts).
* **Vorheriges Bild**: Klicken Sie auf die Schaltfläche ← oder drücken Sie die Taste **←** (Pfeil nach links).
* **Zu einem bestimmten Bild springen**: Kehren Sie zum Dateibrowser zurück und klicken Sie auf die gewünschte Miniaturansicht.

#### Zoomsteuerung

Passen Sie die Vergrößerung an, um Bilddetails zu betrachten:

**Vergrößern:**

* Klicken Sie auf die Schaltfläche **+** (Plus).
* Drücken Sie die Taste **+** oder **=**.
* Scrollen Sie mit dem Mausrad **nach oben**.

**Verkleinern:**

* Klicken Sie auf die Schaltfläche **−** (Minus).
* Drücken Sie die Taste **−** (Minus).
* Scrollen Sie mit dem Mausrad **nach unten**.

**An Bildschirm anpassen:**

* Klicken Sie auf die Schaltfläche **↔** (Anpassen).
* Drücken Sie die Taste **0** (Null).
* Doppelklicken Sie auf das Bild.

#### Schwenken beim Zoomen

Wenn Sie über die Bildschirmgröße hinaus gezoomt haben:

1. Bewegen Sie den Mauszeiger über das Bild.
2. Klicken Sie und **halten Sie die linke Maustaste gedrückt**.
3. **Ziehen** Sie, um das Bild zu verschieben
4. Lassen Sie die Taste los, um das Verschieben zu beenden

**Alternative**: Verwenden Sie die Pfeiltasten, um in kleinen Schritten zu verschieben

***

## Pixelwertprüfung

### Anzeigen der Pixelwerte am Cursor

Wenn Sie den Mauszeiger über das Bild bewegen, werden die Pixelwerte in Echtzeit angezeigt:

**Anzeigeort des Werts:**

* **Gleitkommawert und rote Linie in der LUT-Gradientenlegende auf der rechten Seite**
* **Bei weiterer Vergrößerung: Gleitkommawert in der Nähe des Cursors und hervorgehobenes Pixel**
* Zeigt Werte für Pixel **unter dem Cursor oder hervorgehoben** an
* Wird aktualisiert, wenn Sie die Maus bewegen

***

## Bildtypen, die Sie anzeigen können

### Originalbilder (vor der Verarbeitung)

**RAW- und JPG-Bilder von der Kamera:**

* Anzeige der RAW-Daten als Vorschau
* Anzeige der ursprünglichen, unkorrigierten Werte
* Nützlich zur Überprüfung der Bildqualität vor der Verarbeitung

### Kalibrierte Reflexionsbilder

**Nach der Verarbeitung:**

* Vignettierung korrigiert
* Reflexion kalibriert
* Multiband TIFF (Red, Green, NIR usw.)
* Wissenschaftliche Daten bereit für die Analyse

### Indexbilder

**NDVI, NDRE, GNDVI usw. (\_NDVI.tif-Dateien):**

* Einband-Graustufenbilder
* Pixelwerte stellen Indexberechnungsergebnisse dar
* Bereich typischerweise -1 bis +1 für normalisierte Indizes
* Kann Farblookup-Tabellen (LUTs) zur Visualisierung anwenden

***

## Index- und LUT-Anwendung

Wenden Sie multispektrale Indizes und Farblookup-Tabellen an:

1. Suchen Sie **Index/LUT Sandbox** in der **Bildanzeige** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> Seitenleiste
2. Wählen Sie den Vegetationsindex (NDVI, NDRE usw.)
3. Wählen Sie eine multispektrale Formel oder erstellen Sie eine eigene benutzerdefinierte Formel (nur Chloros+)
4. Wenden Sie einen Farb-LUT-Farbverlauf zur Visualisierung an
5. Passen Sie Wertebereiche und Schwellenwerte an

Ausführliche Anweisungen finden Sie unter [Index/LUT-Sandbox](index-lut-sandbox.md).

***

## Tastaturkürzel

### Navigation

* **→** (Pfeil nach rechts): Nächstes Bild
* **←** (Pfeil nach links): Vorheriges Bild
* **Home**: Erstes Bild in der Liste
* **Ende**: Letztes Bild in der Liste

### Zoomen

* **+** oder **=**: Vergrößern
* **−**: Verkleinern
* **0** (Null): An Bildschirm anpassen
* **Mausrad**: Vergrößern/Verkleinern

### Ansichtssteuerung

* **P**: Pixelprozentmodus umschalten
* **L**: Ebenenbedienfeld umschalten
* **Esc**: Vollbildmodus schließen oder zum Dateibrowser zurückkehren

### Sonstiges

* **Strg+S**: Aktuelles Bild speichern
* **F**: Vollbildmodus (falls verfügbar)

***

### Überprüfen der Indexberechnungen

Überprüfen Sie, ob die Indizes korrekt berechnet wurden:

1. Öffnen Sie NDVI oder ein anderes Indexbild.
2. Überprüfen Sie die Vegetationsflächen:
   * **NDVI**: Sollte für gesunde Pflanzen einen Wert zwischen 0,4 und 0,9 anzeigen.
   * **NDRE**: Höhere Werte für kräftiges Wachstum
   * **GNDVI**: Ähnlich wie NDVI, jedoch chlorophyllempfindlich
3. Nichtvegetation überprüfen:
   * **Boden**: Nahe 0 oder leicht negativ
   * **Wasser**: Negative Werte (-0,5 bis 0)

***

## Fehlerbehebung bei Anzeigefehlern

### Bild lässt sich nicht öffnen

**Mögliche Ursachen:**

* Datei während der Verarbeitung beschädigt
* Nicht unterstütztes Dateiformat
* Unzureichender Speicher für großes Bild

**Lösungen:**

1. Versuchen Sie, die Datei in einem externen Viewer zu öffnen, um die Dateiintegrität zu überprüfen.
2. Überprüfen Sie, ob das Dateiformat mit dem erwarteten Typ übereinstimmt.
3. Schließen Sie andere Anwendungen, um Speicherplatz freizugeben.
4. Versuchen Sie es mit einem kleineren/anderen Bild.

### Schwarz-Weiß-Bildanzeige

**Mögliche Ursachen:**

* Wertebereich außerhalb der Anzeigefähigkeit
* 32-Bit-Float-Bild mit ungewöhnlichen Werten
* Fehler bei der Indexberechnung

**Lösungen:**

1. Überprüfen Sie die Pixelwerte – wenn alle sehr niedrig oder sehr hoch sind, passen Sie den Anzeigebereich an.
2. Versuchen Sie, die Datei in QGIS oder einem ähnlichen Programm mit automatischer Bereichsanpassung zu öffnen.
3. Überprüfen Sie das Debug-Protokoll der Verarbeitung auf Fehler.

### Pixelwerte scheinen falsch zu sein

**Mögliche Ursachen:**

* Anzeige eines falschen Bildes (Original vs. verarbeitet)
* Kalibrierung wurde nicht korrekt angewendet
* Lichtsensordaten wurden nicht in die Eingabe einbezogen
* Prozentmodus wurde falsch umgeschaltet

**Lösungen:**

1. Überprüfen Sie, ob Sie die verarbeitete Ausgabe anzeigen (überprüfen Sie die Dateinamenerweiterung).
2. Überprüfen Sie den Status der Prozentmodus-Schaltfläche.
3. Vergleichen Sie mit bekanntermaßen guten Bildern aus demselben Datensatz.

***

## Nächste Schritte

Nachdem Sie nun Bilder im Vollbildmodus anzeigen können:

* [**Bildlayer**](image-layers.md) – Erfahren Sie mehr über die Multiband-Visualisierung.
* [**Index/LUT-Sandbox**](index-lut-sandbox.md) – Wenden Sie benutzerdefinierte Indizes und Farbabbildungen an.
* [**Multispektrale Indexformeln**](../project-settings/multispectral-index-formulas.md) – Erfahren Sie mehr über verfügbare Indizes

Informationen zum Verarbeitungsworkflow finden Sie unter:

* [**Bilder verarbeiten (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Vollständige Verarbeitungsanleitung
