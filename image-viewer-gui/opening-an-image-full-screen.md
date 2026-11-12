# Ein Bild im Vollbildmodus öffnen

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Ein im Vollbildmodus geöffnetes Bild mit der Ebenenauswahl oben rechts</p></figcaption></figure>

Der Chloros Bildbetrachter ist die Vollbild-Oberfläche zum Anzeigen, Prüfen und Messen Ihrer Bilder. Hier können Sie **tatsächliche Pixelwerte** — DN pro Kanal, Reflexionsgrad in Prozent oder Strahlungsintensität in W/m²/sr/nm — ablesen können und nicht die gestreckte Vorschau, die der Bildschirm anzeigt.

## Aufrufen des Bildbetrachters

### Über den Dateibrowser

1. Öffnen Sie die Registerkarte **Dateibrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klicken Sie auf eine beliebige **Miniaturansicht** im [Bildraster](image-grid.md)
3. Das Bild wird im Vollbildmodus auf der Registerkarte **Bildbetrachter** geöffnet

Das Bild wird in dem Produkt geöffnet, das gerade im Raster angezeigt wurde. Ist das Raster auf `RAW (Reflectance)` eingestellt, gelangen Sie zu dieser Ebene.

### Die Seitenleiste des Bildbetrachters öffnen

Klicken Sie in der linken Seitenleiste auf das Symbol **Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">, um das Analysefenster auszuklappen. Es enthält von oben nach unten:

* den Bildnamen und das Kameramodell
* die Schaltfläche **Bild(er) exportieren/speichern** (nur wenn ein Index oder eine LUT aktiv ist)
* die Kontrollkästchen **Index**und**LUT** sowie das Konfigurationsfenster für den Index – siehe [Index/LUT-Sandbox](index-lut-sandbox.md)
* das Fenster **Cursor-Werte**: kanalweise Auslesung, Ebenenhistogramm und die GSD-Steuerung***

## Navigieren und Zoomen

### Durch Bilder blättern

* **Nächstes Bild**: die Schaltfläche → oder die Taste**→** (Pfeil nach rechts)
* **Vorheriges Bild**: die Schaltfläche ← oder die Taste**←** (Pfeil nach links)
* **Zu einem bestimmten Bild springen**: Zurück zum Raster gehen und auf dessen Miniaturansicht klicken

Zoom und Verschiebung bleiben beim Wechsel zwischen den Bildern erhalten, sodass Sie eine Bilderserie durchgehen können, während Sie auf demselben Teil des Bildausschnitts bleiben.

### Zoom

Das Zoomen erfolgt über das **Mausrad** in 15-prozentigen Schritten und ist am Cursor verankert – der Punkt unter dem Zeiger bleibt unter dem Zeiger. Der Bereich wird durch die Bild- und Fenstergröße begrenzt: Sie können nicht über die Option „An Fenster anpassen“ hinaus herauszoomen, und die Obergrenze wird durch die native Auflösung des Bildes festgelegt.

Im Vollbild-Viewer gibt es keine speziellen Zoom-Tasten. (Im Raster ändern **Strg + `+` / `−`** die Größe der Miniaturansichten – dies ist eine andere Funktion.)

### Verschieben im gezoomten Zustand

Klicken Sie mit der linken Maustaste auf das Bild, halten Sie die Taste gedrückt und ziehen Sie. Das Verschieben ist begrenzt, sodass das Bild nicht aus dem Bildschirmbereich herausgezogen werden kann.

### Pixelgenaue Untersuchung bei hoher Vergrößerung

Sobald die effektive Vergrößerung **60×** überschreitet, zeichnet Chloros einen hervorgehobenen Rahmen um das einzelne angezeigte Pixel unter dem Cursor und zeigt daneben einen Wert an.

Die „effektive“ Vergrößerung berücksichtigt die GSD-Blockgröße: Bei einer Blockgröße von 8 erscheint die Markierung bereits bei einer 7,5-fachen Vergrößerung statt bei 60-facher, da ein angezeigtes Pixel bereits 8 × 8 Quellpixel umfasst. Wenn Sie den Zoom wieder unter den Schwellenwert zurücknehmen, verschwindet die Markierung.

### Tastaturkürzel

| Taste                             | Wo       | Aktion                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Vollbild | Nächstes Bild                          |
| **←**                           | Vollbild | Vorheriges Bild                      |
| **Strg + R**                    | Vollbild | Index/LUT-Sandbox zurücksetzen         |
| **Strg + `+`**/**Strg + `=`** | Raster        | Größere Miniaturansichten (4 px pro Tastendruck)  |
| **Strg + `−`**                  | Raster        | Kleinere Miniaturansichten (4 px pro Tastendruck) |

***

## Cursorwerte

Bewegen Sie den Cursor über das Bild, und das Fenster **Cursorwerte** zeigt den Wert jedes darunterliegenden Kanals an.

{% hint style="success" %}
**Dies sind die tatsächlichen Werte der Datei.** Die Leinwand auf dem Bildschirm ist eine gestreckte 8-Bit-Vorschau und kann diese Werte nicht liefern, daher nimmt Chloros für die Anzeige eine Stichprobe aus der eigentlichen Produktdatei. Aus diesem Grund zeigt ein 12-Bit-Raw-Frame Werte über 255 an, und eine Float32-Radiance-Ebene gibt physikalische Einheiten an.
{% endhint %}

### Bedeutung der Spalten

Das Fenster passt sich der gerade angezeigten Ebene an:

| Aktuell angezeigte Ebene              | Angezeigte Spalten    | Anmerkungen                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Reflexionsgrad                        | **DN**und**%** | Der Prozentsatz wird anhand der dateispezifischen Skalierung berechnet – siehe unten                                      |
| Strahlungsintensität                        | **W/m²/sr/nm**   | Physikalische Werte als Gleitkommazahlen; keine DN-Spalte, da ein DN-Wert hier keine Aussagekraft hat                           |
| Rohdaten / Debayert / Vorschau / JPG    | **DN**           | Digitale Ganzzahlwerte                                                                         |
| 32-Bit-Exporte der Reflexionsrate in Prozent | nur **%**       | Der gespeicherte Gleitkommawert ist kein DN, daher würde eine Rundung auf eine Ganzzahl einen sinnlosen Wert wie `0` oder `1` ergeben |

Jede Zeile ist mit dem Kanalnamen für den Filter Ihrer Kamera beschriftet – `Red / Green / NIR` für RGN, `Orange / Cyan / NIR` für OCN, `NIR / Green / Blue` für NGB, `Red / Green / Blue` für RGB sowie den Namen des einzelnen Bandes für RE-, NIR sowie für Mono-M3M-Kameras. Jede Beschriftung ist mit einem farbigen Punkt versehen, der den im Indexformel-Editor verwendeten Kanalkreisen entspricht.

Gespeicherte **Index- und LUT**-Bilder stellen einen Sonderfall dar: Sie enthalten Farbkartenkomponenten anstelle von Spektralbändern, daher sind ihre Zeilen mit „`Red / Green / Blue`“ (oder „`Index`“ bei einer einkanaligen Indexdatei) beschriftet und nicht mit den Filternamen der Kamera.

Wenn ein Index in der Sandbox aktiv ist, erscheint unterhalb der Kanäle eine zusätzliche Zeile, die den **Indexwert** an der Cursorposition anzeigt, zusammen mit dem Namen des Index und einem weißen Punkt, der seiner Markierung im Histogramm entspricht.

### Der Reflexionsgrad in Prozent verwendet die dateispezifische Skala

{% hint style="warning" %}
**Gehen Sie nicht davon aus, dass 65535 = 100 % ist.** Chloros speichert die Reflexion in unterschiedlichen Skalen, je nachdem, von welcher Kamera sie stammt, und der Viewer ermittelt für jede Datei die richtige Skala.
{% endhint %}

| Quelle                  | DN, der dem Reflexionsgrad 1,0 entspricht | Wie er identifiziert wird                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP-Tag `Chloros:PixelScale=32768`, das in jeden LATTICE-Reflexionsgrad-Export geschrieben wird. Der 2-fache Spielraum ermöglicht es, dass die Datei ρ-Werte über 1,0 enthält, ohne dass es zu Abschneidungen kommt |
| **Survey3**|**65535**                      | Kein XMP-Skalierungs-Tag „Chloros“ – die Kalibrierung „Survey3“ schreibt ρ × dtype-max und beschneidet bei 1,0                                                               |

Der Viewer, die Index-/LUT-Sandbox und der Indexexport ermitteln die Skalierung alle über dieselbe einzige Implementierung, sodass ein Wert, den Sie am Cursor ablesen, derselbe Wert ist, den die Indexberechnung verwendet hat.

Zwei wissenswerte Konsequenzen:

* Ein **32-Bit-Prozentwert**(TIFF) speichert DN/65535 als Gleitkommazahl, und ein**8-Bit-** PNG/JPG-Export speichert DN × 255/65535 – der Viewer rechnet beide Werte vor der Anzeige eines Prozentwerts wieder zurück.
* Ein Fall lässt sich nicht beheben: Ein **8-Bit-TIFF-Export einer Aufnahme aus einer 8-Bit-Quelle** wird auf 0–255 beschnitten statt neu skaliert und enthält bewusst kein Skalierungs-Tag. Bei diesen Dateien zeigt das Bedienfeld nur den DN an, ohne Prozent-Spalte. Dies ist die ehrliche Antwort, kein Fehler.

***

## Das Ebenenhistogramm

Unterhalb der Cursorzeilen befindet sich ein Live-Histogramm der angezeigten Ebene in **256 Bins**. Standardmäßig wird eine kombinierte Kurve gezeichnet, gewichtet nach `(R + 2G + B) / 4` – demselben Messraum, den auch die LATTICE-Kamera-Histogramme verwenden. Wenn Sie**RGB** aktivieren, wird diese durch kanalbezogene Kurven in den Kanalfarben ersetzt, die additiv gemischt werden, damit Überlappungen lesbar bleiben. Mono-Ebenen zeichnen immer die einzelne Kurve.

Die horizontale Achse ist in der eigenen Einheit der Ebene angegeben:

| Ebene       | Achsen-Einheit  | Achsen-Maximalwert                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Reflexionsgrad | Prozent    | 125 % – der Headroom des Produkts erlaubt ρ über 1,0           |
| Strahlungsintensität | W/m²/sr/nm | Der frameneigene Spitzenwert, auf zwei signifikante Stellen aufgerundet |
| 8-Bit-Daten  | DN         | 255                                                        |
| 12-Bit-Daten | DN         | 4095                                                       |
| 16-Bit-Daten | DN         | 65535                                                      |

Wenn die Achse auf DN eingestellt ist und auf einem dieser drei Höchstwerte liegt, erkennt Chloros auch die Bittiefe des angezeigten Bildmaterials.

Über dem Histogramm befinden sich drei Schaltflächen:

| Schaltfläche     | Standardwert | Wirkung                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSOR** | Ein      | Zeichnet Markierungslinien auf dem Histogramm genau an den in den obigen Zeilen angegebenen Werten ein, sodass Sie sehen können, wo sich das Pixel unter Ihrem Cursor in der Verteilung des Bildes befindet. Im Modus RGB gibt es pro Kanal eine Markierung in einer eigenen Farbe; andernfalls eine einzelne weiße Markierung am kombinierten Wert |
| **INDEX**| Ein      | Wird nur angezeigt, solange ein Index aktiv ist. Schaltet das Histogramm von den Quellbändern auf die**Indexwertverteilung** um, wobei die beiden Clip-Schwellenwerte als orangefarbene gestrichelte Linien und der Indexwert des Cursors als weiße Linie dargestellt werden                                                          |
| **RGB**| Aus     | Schaltet von der kombinierten Kurve zu kanalspezifischen Kurven um. Bei einem Mono-Sensor lautet die Beschriftung dieser Schaltfläche**MONO** und sie ist deaktiviert – es gibt nur einen Kanal zur Anzeige                                                                                                                                  |

Das Histogramm wird anhand der **sichtbaren Blöcke** berechnet, nicht anhand der dahinter liegenden Quellpixel: Wenn Sie die GSD-Blockgröße ändern, wird die Verteilung neu berechnet, sodass das Histogramm, die Cursormarkierung und das angezeigte Bild stets übereinstimmen.

***

## GSD-Blockgröße

Am unteren Rand des Fensters befindet sich das Steuerelement **GSD (px)**: ein Zahleneingabefeld, ein Schieberegler von**1 bis 256**und eine**RESET**-Schaltfläche.

Es vergröbert das _angezeigte_ Bild, indem es einen N × N-Block von Quellpixeln zu einem angezeigten Pixel mittelt. `1` ist die native Auflösung.

* Dies wirkt sich auf **die Vollbildansicht, die Raster-Miniaturansichten, die Cursoranzeige und beide Histogramme** aus – alles, was das Bild anzeigt, basiert auf derselben Bodenauflösung.
* Dies gilt **nur für die Anzeige**. Die Verarbeitung und der Export bleiben davon unberührt. Die einzige Ausnahme ist beabsichtigt: Ein Export über die [Index/LUT-Sandbox](index-lut-sandbox.md) speichert das, was Sie gerade sehen, übernimmt also die aktuelle Blockgröße, und das Exportfenster warnt Sie, wenn die Blockgröße über 1 liegt.
* Der Wert wird **pro Projekt** als `viewer_display.gsd_bin` in `project.json` gespeichert, sodass er beim Schließen und erneuten Öffnen erhalten bleibt.
* Die Cursoranzeige gibt den Blockwert und nicht den Wert des Quellpixels an, sobald die Blockgröße größer als 1 ist – der angezeigte Wert ist der Mittelwert des Blocks unter Ihrem Cursor.

{% hint style="info" %}
**Warum „Blockgröße“ und nicht Zentimeter pro Pixel?** Ein Wert in cm/px erfordert eine Höhe über dem Boden. Die EXIF-Daten eines einzelnen Bildes enthalten die GPS-Höhe über dem mittleren Meeresspiegel, nicht über dem Gelände, auf das die Kamera gerichtet war; daher gibt Chloros keine Bodenentfernung an, die es nicht zuverlässig ableiten kann. Die Blockgröße in Quellpixeln ist dieselbe Ausweichlösung, die die MAPIR-Cloud-Tools verwenden, wenn der Bodenabstand unbekannt ist.
{% endhint %}

***

## Bildtypen, die Sie anzeigen können

Das Ebenen-Dropdown-Menü oben rechts im Viewer listet alle Versionen des aktuellen Bildes auf. Welche Einträge angezeigt werden, hängt von der Kamera und den durchgeführten Verarbeitungsschritten ab – siehe [Bildebenen](image-layers.md) für die vollständige Liste und Informationen zum Verhalten des Dropdown-Menüs.

### Survey3

* **JPG** – die kameraneigene Vorschaudatei
* **RAW (Original)** – die `.RAW`-Quelldatei, zur Anzeige debayert, ohne Korrekturen
* **RAW (Ziel)** – ein Bild, das als Kalibrierungsziel identifiziert wurde
* **RAW (Reflexionsgrad)** – das kalibrierte Reflexionsgrad-Produkt (65535 = ρ 1,0)
* **Vignettierungskorrektur**/**Sensorkennlinie** – das unkalibrierte Fallback-Produkt
* **Weißabgleich** — das weißabgeglichene Produkt
* **RAW (`<INDEX>`-Index)**und**`<INDEX>`-LUT** — berechnete Indexbilder

### LATTICE

LATTICE-Aufnahmen verwenden dasselbe Dropdown-Menü mit den Namen der Pipeline-Ebenen:

| Ebene                 | Inhalt                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Original)**    | Das erfasste RAW-Quellbild                                     |
| **RAW (Debayered)**   | Das linear debayered Bild                                           |
| **RAW (Vorschau)**     | Die Anzeigevorschau – Falschfarben-Streckung für Multispektralkameras |
| **Weißabgleich**    | Die Anzeigevorschau für RGB-Masterkameras (Weißabgleich + Gamma)   |
| **RAW (Strahlungsintensität)**    | Spektrale Strahlungsintensität als Float32 in W/m²/sr/nm                              |
| **RAW (Reflexionsgrad)** | Reflexionsgrad als uint16, 32768 = ρ 1,0                                    |

Strahlungsintensität und Reflektanz sind ausschließlich multispektral: Eine RGB-Masterkamera verfügt über keine bandweise Radiometrie, daher werden diese Ebenen für sie nicht erzeugt.

***

## Index und LUT-Anwendung

Wenden Sie multispektrale Indizes und Farb-Look-Up-Tabellen über die Seitenleiste an:

1. Öffnen Sie den **Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> Seitenleiste
2. Aktivieren Sie **Index**

3. Wählen Sie den Filter Ihrer Kamera und eine Indexformel aus und ziehen Sie dann die Kanalkreise auf die Felder der Formel
4. Fügen Sie eine LUT hinzu und wählen Sie einen Farbverlauf, Schwellenwerte und einen Clipping-Modus aus
5. Lesen Sie die Werte am Cursor ab und speichern Sie das Ergebnis mit **Exportieren/Bild(er) speichern**Die vollständige Anleitung finden Sie unter [Index/LUT-Sandbox](index-lut-sandbox.md) für die vollständige Anleitung.

***

## Fehlerbehebung

### Das Bild lässt sich nicht öffnen

**Mögliche Ursachen**: Die Datei wurde nach dem Import verschoben oder gelöscht; das Produkt wurde nie geschrieben; nicht genügend Speicherplatz für ein sehr großes Bild.**Was tun**:

1. Überprüfen Sie, ob die Datei der Ebene noch im Ausgabeverzeichnis des Projekts vorhanden ist
2. Öffnen Sie die Datei in einem externen Bildbetrachter, um sicherzustellen, dass sie intakt ist
3. Schließen Sie andere Anwendungen, um Speicherplatz freizugeben

### Das Bild ist schwarz, weiß oder stark verfärbt

**Mögliche Ursachen**: Die Bildstreckung hat keine Daten zur Verfügung (ein nahezu konstanter Frame); eine „float32“-Ebene mit ungewöhnlichen Werten; ein Index, der keine gültigen Daten erzeugt hat.**Vorgehensweise**:

1. Lesen Sie die Cursorwerte aus – wenn jeder Kanal bei oder nahe Null liegt, liegt das Problem bei den Daten, nicht bei der Anzeige
2. Überprüfen Sie das Histogramm: Ein einzelner Ausschlag an einem Ende weist darauf hin, dass das Bild abgeschnitten oder leer ist
3. Überprüfen Sie das Verarbeitungsprotokoll für den Lauf, der die Ebene erzeugt hat

### Die Werte sehen falsch aus

**Mögliche Ursachen**: Sie befinden sich auf einer anderen Ebene als angenommen; Sie vergleichen einen Prozentwert mit einem rohen DN; Sie vergleichen eine LATTICE-Datei mit einer Survey3-Datei unter Verwendung desselben Divisors.**Was zu tun ist**:

1. Überprüfen Sie die ausgewählte Ebene im Dropdown-Menü – die Einheiten im Fenster richten sich nach der Ebene.
2. Verwenden Sie für die Reflexanz die Spalte **%**, anstatt den DN selbst zu dividieren; falls Sie dividieren müssen, verwenden Sie den Wert der jeweiligen Datei: `Chloros:PixelScale` (32768 für LATTICE, fehlt dieser Wert, gilt 65535 für Survey3)
3. Setzen Sie die GSD-Blockgröße wieder auf 1 zurück – bei Werten über 1 lesen Sie einen Blockmittelwert aus, kein einzelnes Pixel
4. Überprüfen Sie, ob die Reflektanzkalibrierung für diesen Frame tatsächlich durchgeführt wurde; ein unkalibriertes Fallback-Produkt (Sensor Response / Vignette Corrected) ist keine Reflektanz

***

## Nächste Schritte

* [**Bildebenen**](image-layers.md) – alle Ebenennamen (sofern vorhanden) und die Bedeutung ihrer Werte
* [**Index/LUT-Sandbox**](index-lut-sandbox.md) – Erstellen, Anpassen und Exportieren von Indexvisualisierungen
* [**Kartenmarkierungen**](map-markers.md) – derselbe Bildsatz auf einer Karte
* [**Multispektrale Indexformeln**](../project-settings/multispectral-index-formulas.md) – die Indexreferenz

Informationen zum Verarbeitungsworkflow finden Sie unter [Bilder verarbeiten (GUI)](../processing-images-gui/adding-files-to-a-project.md).
