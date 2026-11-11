# Projekteinstellungen

Über die Seitenleiste „Projekteinstellungen“ <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

in Chloros können Sie alle Aspekte der Bildverarbeitung, der Erkennung von Kalibrierungszielen, der Berechnung multispektraler Indizes sowie der Exportoptionen für Ihr Projekt konfigurieren. Diese Einstellungen werden zusammen mit Ihrem Projekt gespeichert und können als Vorlagen gespeichert werden, um sie in mehreren Projekten wiederzuverwenden.

## Aufrufen der Projekteinstellungen

So rufen Sie die Projekteinstellungen auf:

1. Öffnen Sie ein Projekt in Chloros.
2. Klicken Sie in der linken Seitenleiste auf die Registerkarte **Projekteinstellungen** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">.
3. Im Einstellungsfenster werden alle verfügbaren Konfigurationsoptionen nach Kategorien geordnet angezeigt.



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Einstellungen, die von anderen Einstellungen abhängen, sind ausgegraut.** Wenn eine übergeordnete Option eine Einstellung unmöglich macht (beispielsweise macht das Deaktivieren von *Reflexionskalibrierung / Weißabgleich* den *Export der Reflexion* unmöglich), wird das davon abhängige Steuerelement deaktiviert und sein Tooltip nennt die Option, die geändert werden muss.
{% endhint %}

***

## Anzeige

### Auflösung der Bildminiaturansichten

* **Typ**: Dropdown-Auswahl
* **Optionen**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Standard**: Standard (512 px)
* **Beschreibung**: Auflösung (längste Kante, in Pixeln), mit der die Miniaturansichten im Bildraster dargestellt werden. Höhere Werte wirken beim Vergrößern schärfer, werden jedoch langsamer geladen und beanspruchen mehr Speicherplatz. Die volle Auflösung entspricht der Originalbildgröße.
* **Hinweis**: Nur zur Anzeige – dies hat keinerlei Auswirkungen auf die Verarbeitung oder exportierte Dateien.***

## Zielerkennung

Diese Einstellungen steuern, wie Chloros Kalibrierungsziele in Ihren Bildern erkennt und verarbeitet. Beide sind nur aktiv, solange **Reflexionskalibrierung / Weißabgleich** aktiviert ist (ansonsten sind sie ausgegraut, da die Zielerkennung vollständig übersprungen wird).

### Mindestfläche für Kalibrierungsmuster (px)

* **Typ**: Zahl
* **Bereich**: 0 bis 10.000 Pixel
* **Standard**: 25 Pixel
* **Beschreibung**: Legt die Mindestfläche (in Pixeln) fest, die erforderlich ist, damit ein erkannter Bereich als gültige Kalibrierungszielprobe gilt. Kleinere Werte erkennen kleinere Ziele, können jedoch zu mehr Fehlalarmen führen. Größere Werte erfordern größere, deutlichere Zielbereiche für die Erkennung.
* **Wann anpassen**:
  * Erhöhen Sie den Wert, wenn Sie bei kleinen Bildartefakten falsche Erkennungen erhalten.
  * Verringern Sie den Wert, wenn Ihre Kalibrierungsziele in Ihren Bildern klein erscheinen und nicht erkannt werden.

### Minimale Zielclusterung (0–100)

* **Typ**: Zahl
* **Bereich**: 0 bis 100
* **Standardwert**: 60
* **Beschreibung**: Steuert den Clustering-Schwellenwert für die Gruppierung ähnlich gefärbter Bereiche bei der Erkennung von Kalibrierungszielen. Höhere Werte erfordern, dass mehr ähnliche Farben zusammengefasst werden, was zu einer konservativeren Zielerkennung führt. Niedrigere Werte lassen mehr Farbabweichungen innerhalb einer Zielgruppe zu.
* **Wann anpassen**:
  * Erhöhen Sie den Wert, wenn Kalibrierungsziele in mehrere Erkennungen aufgeteilt werden.
  * Verringern Sie den Wert, wenn Kalibrierungsziele mit Farbabweichungen nicht vollständig erkannt werden.

***

## Verarbeitung

Diese Einstellungen steuern, wie Chloros Ihre Bilder verarbeitet und kalibriert.

### Vignettierungskorrektur

* **Typ**: Kontrollkästchen
* **Standard**: Aktiviert (markiert)
* **Beschreibung**: Wendet eine Vignettierungskorrektur an, um die Objektivverdunkelung an den Bildrändern auszugleichen. Vignettierung ist ein häufiges optisches Phänomen, bei dem die Ecken und Ränder eines Bildes aufgrund der Objektiveigenschaften dunkler erscheinen als die Bildmitte.
* **Nebenwirkung**: Mit diesem Schalter wird auch ausgewählt, welches *unkalibrierte Fallback-Produkt* bei einem Durchlauf geschrieben wird (siehe unten).

### Reflexionskalibrierung / Weißabgleich

* **Typ**: Kontrollkästchen
* **Standard**: Aktiviert (markiert)
* **Beschreibung**: Aktiviert die Reflektanzkalibrierung – basierend auf erkannten Kalibrierungszielen im Bildausschnitt und/oder Downwelling-Daten des DAQ-Lichtsensors, je nach Kamera und verfügbaren Daten. Dies normalisiert die Reflektanzwerte in Ihrem Datensatz und gewährleistet konsistente Messungen unabhängig von den Lichtverhältnissen.
* **Wenn deaktiviert**: Die Zielerkennung wird vollständig übersprungen, und**es kann von keiner Kamera ein Reflexionsprodukt erstellt werden** – weder bei der zielgesteuerten Survey3-Methode noch bei der DAQ-gesteuerten „LATTICE“-Methode. Die davon abhängigen Einstellungen (*Reflexion exportieren*, *Mindestintervall für Neukalibrierung* und die Schwellenwerte für die Zielerkennung) sind ausgegraut.

### Fallback-Produkte bei fehlender Kalibrierung: Sensorantwort exportieren / Vignettierungskorrektur exportieren

* **Typ**: Zwei Kontrollkästchen
* **Standardeinstellungen**: Beide aktiviert (markiert)
* **Beschreibung**: Wenn ein Bild nicht reflektanzkalibriert werden kann (es wurde kein Kalibrierungsziel gefunden oder die Reflektanzkalibrierung ist deaktiviert), wird es stattdessen als *unkalibriertes Fallback-Produkt* gespeichert. **Pro Durchlauf gibt es genau eines der beiden Fallback-Produkte für jedes Kameramodell**, das über den Schalter *Vignettenkorrektur* ausgewählt wird:
  * Vignettenkorrektur **ein**→ `Vignette_Corrected_Images/` (abhängig von**Export vignettenkorrigiert**)
  * Vignettenkorrektur **aus**→ `Sensor_Response_Images/` (abhängig von**Export Sensorantwort**)
* Das nicht verwendete Fallback-Produkt ist ausgegraut. Wird das Häkchen bei dem verwendeten Produkt entfernt, wird die entsprechende Datei gar nicht erst geschrieben.

### LATTICE-Exportprodukte

Bei Projekten, die LATTICE-Aufnahmen enthalten, wird jedes importierte LATTICE-Bild in einem einzigen Verarbeitungsdurchlauf auf alle aktivierten **und anwendbaren**Produkte aufgeteilt. Vier Kontrollkästchen steuern die Aufteilung (alle standardmäßig**aktiviert**):

| Einstellung | Ausgabeordner | Was exportiert wird |
| --- | --- | --- |
| **Debayered exportieren** | `Debayered_Images/` | Das linear debayered Bild: der Dunkeloffset des Kalibrierungspakets wurde entfernt, anschließend erfolgte ein kantensensitives Demosaicing ohne Linearitätskorrektur, Flat-Field-Korrektur oder Unmix – dieselbe Datei, die eine Aufnahme auf ihrer debayered-Ebene speichert. Wird in der nativen Ausrichtung des Sensors beibehalten (in XMP gespeichert). Gilt für RGB- und Multispektralkameras. |
| **Vorschau exportieren** | `Preview_Images/` | Die Anzeigevorschau. RGB = Weißabgleich (DAQ-Lichtart, sofern verfügbar, andernfalls „Gray-World“) + Gamma; multispektral = Falschfarben-Streckung. |
| **Export-Strahlungsintensität** | `Radiance_Images/` | Spektrale Strahlungsintensität als Float32 in W/m²/sr/nm. Nur multispektral (M3C/M3M) – gilt nicht für RGB-Master. Wird unabhängig von der Einstellung *Kalibriertes Bildformat* immer als 32-Bit-TIFF geschrieben. |
| **Reflexionsgrad exportieren**| `Reflectance_Calibrated_Images/` | Uint16-Reflexionsgrad, so skaliert, dass**32768 = Reflexionsgrad 1,0** (als XMP `Chloros:PixelScale` gekennzeichnet). Nur multispektral, wird geschrieben, wenn ein passender `.daq`-Downwelling-Datensatz (oder ein QA-konformes In-Frame-Ziel) das Bild abdeckt. |

* RGB-Masterkameras geben debayerte Daten + Vorschau aus; Strahlungsintensität/Reflexionsgrad werden bei ihnen übersprungen, da sie nicht zutreffen.
* Die Bittiefe der debayierten Daten/Vorschau richtet sich nach der Einstellung *Kalibriertes Bildformat*; die Strahlungsintensität ist immer float32.
* Die Survey3-Verarbeitung wird von diesen vier Schaltoptionen nicht beeinflusst.
* Strahlungs- und Reflexionsprodukte von multispektralen M3C-Kameras können zudem eine pixelweise **Clamp-Maske** in einen Ordner „`clamp_masks/`“ neben den Produktordnern schreiben, wobei aufgezeichnet wird, wo die Bandentmischung einen negativen Wert auf Null abgeschnitten hat. Siehe [Ausgabebildformate](../output-image-formats.md#clamp-masks).

Die gleichen vier Schalter existieren auch in der Headless-Variante als `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Sie ersetzen das alte CLI-Flag `--radiometric-output`, das nicht mehr existiert.

{% hint style="warning" %}
**Das Deaktivieren aller anwendbaren Produkte führt zum Fehlschlagen des Durchlaufs.** Ab Version 1.2.0 meldet ein Verarbeitungsdurchlauf, bei dem Produkte angefordert wurden, aber kein Bildprodukt geschrieben wurde, einen Fehler, und die CLI wird mit einem Wert ungleich Null beendet, anstatt einen stillen Erfolg zu melden. Das Protokoll nennt das Produkt, das nicht geschrieben werden konnte, sowie den Grund dafür. Ein absichtlich nur auf Metadaten beschränkter Durchlauf (nichts angefordert) gilt weiterhin als erfolgreich.
{% endhint %}

### Reflektanzquelle (Projekteinstellung, über die CLI

festgelegt)

Das Projekt speichert außerdem, welche **Reflektanzreferenz** das LATTICE-Reflektanzprodukt verwendet. Es gibt keine eigene Steuerung im Einstellungsfenster; der Wert wird in der Projektkonfiguration als `Processing → "Target reflectance source"` gespeichert und mit `chloros-cli process --reflectance-source {auto,target,daq}` (das SDK-Element `configure()` verfügt über kein entsprechendes Schlüsselwort):

* **`auto`** (Standard): Ein QA-konformes Kalibrierungsziel im Bild wird zur absoluten Referenz; ist kein Ziel vorhanden oder scheitert die QA, wird auf die DAQ-Downwelling-Teilung (ρ = πL/E) zurückgegriffen. Ein Ziel, das die Qualitätssicherung mit einem Flag besteht (ein Panel ist ausgefallen oder ein Band ist außer Betrieb), wird nicht automatisch bevorzugt: Die Ketten „Ziel“, „Ziel+DAQ“ und „nur DAQ“ werden in der Reihenfolge ihrer gemessenen Punktzahl auf den zurückgehaltenen Panels ausprobiert.
* **`target`**: strikte zielgesteuerte Reflexion – keine DAQ-Ersetzung.
* **`daq`**: DAQ-maßgebliche Reflexion; Ziele im Bild werden nicht als Referenz verwendet.

Der gespeicherte Wert wird ohne Berücksichtigung der Groß-/Kleinschreibung abgeglichen, und einige Schreibweisen werden als Aliase akzeptiert: `target`, `target_image`, `empirical` und `empirical_line` bedeuten alle **Ziel**; `daq`, `dls`, `light_sensor` und `sensor` bedeuten alle**daq**. Alles andere – einschließlich eines fehlenden Schlüssels – wird zu**auto** aufgelöst.**Gemessene** Ziel-Scans pro Einheit werden anhand der Seriennummer/des QR-Codes der Zieleinheit, z. B. `<serial>.csv`, an drei Stellen nachgeschlagen: dem mit `--target-reflectance-dir` angegebenen Verzeichnis (gespeichert als `Processing → "Target reflectance dir"`), dem projektspezifischen Ordner `target_reflectance/` sowie im Pfad der Umgebungsvariablen „`CHLOROS_TARGET_REFLECTANCE_DIR`“. Wenn für diese Einheit kein gemessener Scan vorhanden ist, wird stattdessen die veröffentlichte Nennkurve für das Zielmodell verwendet.

### Debayer-Methode

* **Typ**: Auswahl aus Dropdown-Menü
* **Optionen**:
  * Standard (schnell, mittlere Qualität)
  * Texturorientiert (langsam, höchste Qualität) \[Chloros+]
* **Standard**: Standard (schnell, mittlere Qualität)
* **Beschreibung**: Wählt den Demosaicing-Algorithmus aus, der zur Umwandlung der Rohdaten des Bayer-Muster-Sensors in Vollfarbbilder verwendet wird. Die Methode „Standard (schnell, mittlere Qualität)“ bietet ein optimales Gleichgewicht zwischen Verarbeitungsgeschwindigkeit und Bildqualität. Die Methode „Texturorientiert (langsam, höchste Qualität)“ \[Chloros+] verwendet ein hochwertiges, kantenbewusstes Demosaicing in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Demosaicing-Rauschen entfernt. Das „Texture Aware“-Modell benötigt zur Ausführung GPU-Speicher (VRAM). Wir empfehlen die Verwendung, wenn Ihnen mehr als 4 GB VRAM zur Verfügung stehen, um eine schnellere Verarbeitung zu erzielen.
* **Wenn die Zeile überhaupt ein Dropdown-Menü ist**: Das Dropdown-Menü mit zwei Optionen erscheint nur, wenn**beide** Bedingungen erfüllt sind – Sie sind mit einem berechtigten Chloros+-Abonnement angemeldet, **und** das Projekt enthält keine LATTICE-Aufnahmen. Andernfalls wird die Zeile als reiner Text mit dem Wortlaut „`Standard (Fast, Medium Quality)`“ dargestellt, ohne dass eine Auswahlmöglichkeit besteht.
* **Hinweis zu LATTICE**: Es gibt kein LATTICE-trainiertes „Texture Aware“-Modell, und die Pipeline erzwingt unabhängig vom gespeicherten Wert das Standard-Demosaicing für LATTICE-Frames. Wenn Sie einen LATTICE-Ordner zu einem Projekt hinzufügen, für das bereits „Texture Aware“ ausgewählt war, setzt Chloros die Einstellung wieder auf „Standard“ zurück, anstatt einen veralteten Wert in `project.json`.

### Minimales Neukalibrierungsintervall

* **Typ**: Zahl
* **Bereich**: 0 bis 3.600 Sekunden
* **Standardwert**: 0 Sekunden
* **Beschreibung**: Legt das minimale Zeitintervall (in Sekunden) zwischen der Verwendung von Kalibrierungszielen fest. Bei der Einstellung 0 verwendet Chloros jedes erkannte Kalibrierungsziel. Bei einem höheren Wert verwendet Chloros nur Kalibrierungsziele, die mindestens diesen Zeitabstand voneinander haben, wodurch sich die Verarbeitungszeit für Datensätze mit häufigen Kalibrierungszielerfassungen verkürzt.
* **Wann anpassen**:
  * Stellen Sie den Wert auf 0 ein, um bei wechselnden Lichtverhältnissen maximale Kalibrierungsgenauigkeit zu erzielen.
  * Erhöhen Sie den Wert (z. B. auf 60–300 Sekunden), um die Verarbeitung zu beschleunigen, wenn die Lichtverhältnisse konstant sind und Sie häufig Bilder von Kalibrierungszielen erhalten.

### Zeitzonenversatz des Lichtsensors

* **Typ**: Zahl
* **Bereich**: -12 bis +12 Stunden
* **Standardwert**: 0 Stunden
* **Beschreibung**: Legt den Zeitzonen-Offset (in Stunden gegenüber UTC) für die Zeitstempel der Lichtsensordaten fest, der beim Abgleich von Lichtsensor-Protokollen mit den Bildaufnahmezeiten verwendet wird. Neuere `.daq`-Aufzeichnungen enthalten ihre eigene Zeitzonenangabe, daher ist diese Einstellung hauptsächlich für ältere, in Ortszeit aufgezeichnete Protokolle erforderlich.

### PPK-Korrekturen anwenden

* **Typ**: Kontrollkästchen
* **Standard**: Deaktiviert (nicht markiert)
* **Beschreibung**: Aktiviert die Verwendung von PPK-Korrekturen (Post-Processed Kinematic) aus MAPIR-DAQ-Rekordern, die über ein GPS (GNSS) verfügen. Wenn diese Option aktiviert ist, verwendet Chloros alle .daq-Protokolldateien in Ihrem Projektverzeichnis, die Belichtungs-Pin-Daten enthalten, und wendet präzise Geolokalisierungs-Korrekturen auf Ihre Bilder an.
* **Voraussetzung**: In Ihrem Projektverzeichnis muss eine .daq-Protokolldatei mit Belichtungs-Pin-Einträgen vorhanden sein
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
  * Eine Kamera + ein Pin: Die Kamera wird automatisch ausgewählt
  * Eine Kamera + zwei Pins: Pin 1 wird der Kamera automatisch zugewiesen
  * Mehrere Kameras: Manuelle Auswahl erforderlich

### Belichtungs-Pin 2

* **Typ**: Auswahl aus Dropdown-Menü
* **Sichtbarkeit**: Nur sichtbar, wenn „PPK-Korrekturen anwenden“ aktiviert ist UND Belichtungsdaten für Pin 2 verfügbar sind
* **Optionen**:
  * Im Projekt erkannte Kameramodellnamen
  * „Nicht verwenden“ – Dieser Belichtungs-Pin wird ignoriert
* **Standard**: Wird basierend auf der Projektkonfiguration automatisch ausgewählt
* **Beschreibung**: Weist dem Belichtungs-Pin 2 eine bestimmte Kamera für die PPK-Zeitsynchronisation zu, wenn eine Konfiguration mit zwei Kameras verwendet wird.
* **Verhalten bei der automatischen Auswahl**:
  * Eine Kamera + ein Pin: Pin 2 wird automatisch auf „Nicht verwenden“ gesetzt
  * Eine Kamera + zwei Pins: Pin 2 wird automatisch auf „Nicht verwenden“ gesetzt
  * Mehrere Kameras: Manuelle Auswahl erforderlich
* **Hinweis**: Dieselbe Kamera kann nicht gleichzeitig sowohl Pin 1 als auch Pin 2 zugewiesen werden.***

## DAQ-Lichtsensor

Dieser Abschnitt erscheint in den Projekteinstellungen und listet alle DAQ-Downwelling-Dateien im Projekt auf – `.daq`-Aufzeichnungen und DAQ-M-`.csv`-Downwelling-Protokolle. Aufzeichnungen, die auf der Registerkarte „Lichtsensoren“ erstellt wurden, werden automatisch zum geöffneten Projekt hinzugefügt.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Jede Zeile zeigt die Datei, das Sensormodell und die für diese Datei tatsächlich geltende Diffusorkappenkorrektur an. Über den Zeilen befindet sich ein einziges projektweites Steuerelement:

### Kappenüberschreibung (alle Dateien)

* **Typ**: Dropdown-Auswahl
* **Optionen**: `Auto` sowie die für die im Projekt vorhandenen Sensortypen gültigen Diffusorkappen-Korrekturprofile
* **Standard**: Auto
* **Gespeichert als**: `Processing → "DAQ cap id"` (Standard: `auto`)
* **Beschreibung**: `Auto` verwendet die in jeder Datei aufgezeichnete Kappe (wird nichts aufgezeichnet, wird die Sunshine-Kappe angenommen; DAQ-U, DAQ-M und DAQ-E werden jedoch ohne Kappe ausgeliefert, und Chloros kann nicht erkennen, welche Kappe montiert ist, daher gibt die Verarbeitung eine Warnung bezüglich einer angenommenen Kappe aus: Wählen Sie die tatsächlich montierte Kappe aus). Die Auswahl einer bestimmten Kappe überschreibt **alle** Downwelling-Dateien im Projekt mit Ausnahme derjenigen eines DAQ-E-S, dessen Diffusor Teil seiner Kalibrierung ist; ein Projekt, das ausschließlich aus DAQ-E-S-Dateien besteht, zeigt kein Dropdown-Menü an: Rohaufzeichnungen werden mit dieser Korrektur korrigiert, und Aufnahmen, die bereits eine Kappe enthalten, werden neu referenziert (die aufgezeichnete Korrektur wird rückgängig gemacht und die ausgewählte angewendet).
* **Wichtig**: Die ausgewählte Kappe muss mit der Kappe übereinstimmen, die während der Aufnahme physisch angebracht war. Weder der Sensor noch die Software können die physische Kappe erkennen – eine nicht übereinstimmende Kappen-ID führt zu einer fehlerhaften Korrektur der Spektren.

Es gibt bewusst **nur eine** projektweite Steuerung anstelle von Dropdown-Menüs pro Datei: Die Einstellung wirkt sich auf jede nach unten strahlende Quelle im Projekt aus.***

## Array-Ausrichtung

Dieser Abschnitt erscheint **nur**, wenn mindestens ein Bild im Projekt die Modul-zu-Modul-Ausrichtungstransformation enthält, die LATTICE-Arrays zum Zeitpunkt der Aufnahme einfügen (XMP-Tags `Chloros:Alignment*`). Er zeigt an, wie viele Bilder Ausrichtungs-Tags enthalten, welche Kamera als Referenz dient (`REF`-Kennzeichnung) und enthält eine kameraspezifische Tabelle mit der Anzahl der Bilder.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Array-Ausrichtung anwenden

* **Typ**: Kontrollkästchen
* **Standard**: Aktiviert (markiert)
* **Gespeichert als**: `Processing → "Array alignment"`
* **Beschreibung**: Verzerrt jedes verarbeitete Produkt (Debayering / Vorschau / Radiance / Reflektanz / Index) unter Verwendung der bei der Aufnahme festgehaltenen Transformation in die gemeinsame Referenzgeometrie des Arrays. Aus = Export in nativer, sensorspezifischer Geometrie.

### Auf gemeinsame Überlappung zuschneiden

* **Typ**: Kontrollkästchen (nur aktiv, wenn *Array-Ausrichtung anwenden* aktiviert ist)
* **Standard**: Aktiviert (markiert)
* **Gespeichert als**: `Processing → "Array alignment crop"`
* **Beschreibung**: Schneidet ausgerichtete Exporte auf den Bereich zu, den alle Kameramodule gemeinsam nutzen, sodass jedes Band denselben Abdruck hat. Bei „Aus“ bleibt die gesamte Sensorfläche erhalten (schwarze Füllung außerhalb der Quelle).

### Neuberechnung

* **Typ**: Dropdown-Auswahl (nur aktiv, wenn *Array-Ausrichtung anwenden* aktiviert ist)
* **Optionen**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Standard**: Bilinear
* **Gespeichert als**: `Processing → "Array alignment interpolation"`
* **Beschreibung**: Von der Ausrichtungs-Verzerrung verwendete Interpolation. „Nearest“ bewahrt die exakten Quellwerte (keine Vermischung zwischen Pixeln) für eine strenge radiometrische Analyse; „Bilinear“ eignet sich am besten für Kartierungen und visuelle Anwendungen.

Die gleichen drei Optionen sind auch ohne Präfix als `chloros-cli process --array-alignment`, `--array-alignment-crop` und `--array-alignment-interp {bilinear,nearest,cubic}` verfügbar.

***

## Indizes

Mit diesen Einstellungen können Sie multispektrale Indizes für die Analyse und Visualisierung konfigurieren.

### Index hinzufügen

* **Typ**: Spezielles Konfigurationsfenster für Indizes
* **Beschreibung**: Öffnet ein interaktives Fenster, in dem Sie multispektrale Vegetationsindizes (NDVI, NDRE, EVI usw.) auswählen und konfigurieren können, die während der Bildverarbeitung berechnet werden sollen. Sie können mehrere Indizes hinzufügen, wobei jeder seine eigenen Visualisierungseinstellungen hat.
* **Verfügbare Indizes**: Das Dropdown-Menü der Benutzeroberfläche enthält**27** vordefinierte multispektrale Indexformeln (siehe [Multispektrale Indexformeln](multispectral-index-formulas.md) für die vollständige Liste, einschließlich der Namen, die auch von der CLI-/SDK-Option `--indices`-Option) akzeptiert werden.
* **Funktionen**:
  * Auswahl aus vordefinierten Indexformeln
  * Ziehen Sie die Filterkanäle Ihrer Kamera auf die Bandplätze der Formel
  * Konfigurieren Sie Farbverläufe für die Visualisierung (LUT – Look-Up-Tabellen)
  * Schwellenwerte und Clipping-Modi festlegen
  * Benutzerdefinierte Indexformeln erstellen
* **Hinweis**: Für einbandige LATTICE M3M-Monochromkameras werden keine Indizes berechnet – Multiband-Indizes sind bei einem Band undefiniert. Survey3 und LATTICE M3C sind davon nicht betroffen.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Jeder Index, den Sie hinzufügen, stellt seine Formel als mathematischen Ausdruck dar, mit einem farbigen Kreis pro Bandplatz: rot = Red, grün = Green, blau = Blue, orange = Orange, Cyan = Cyan, Violett = NIR, Magenta = RE. Ziehen Sie einen Kreis aus der Zeile über der Formel auf einen Slot, um ihn zuzuordnen; doppelklicken Sie auf einen zugeordneten Slot, um ihn zu löschen. Der Index wird nur berechnet, wenn jeder von der Formel verwendete Slot über einen Kanal verfügt.

### Benutzerdefinierte Formeln (Chloros+-Funktion)

* **Typ**: Array mit Definitionen benutzerdefinierter Formeln
* **Verfügbarkeit**: Erfordert die Anmeldung mit einem berechtigten Chloros+-Abonnement.
* **Beschreibung**: Ermöglicht es Ihnen, benutzerdefinierte multispektrale Indexformeln mithilfe von Bandmathematik zu erstellen und zu speichern. Benutzerdefinierte Formeln werden mit Ihren Projekteinstellungen gespeichert und können genau wie integrierte Indizes verwendet werden.
* **So erstellen Sie eine Formel**:
  1. Öffnen Sie im Konfigurationsbereich „Index“ den Rechner für benutzerdefinierte Formeln.
  2. Geben Sie die Formel unter Verwendung der **Band-Slot-Symbole** ein, nicht der Bandnamen.
  3. Speichern Sie die Formel unter einem aussagekräftigen Namen – sie erscheint dann am Ende der Formel-Auswahlliste, und Sie ziehen die Kanalkreise Ihrer Kamera genau wie bei einer integrierten Voreinstellung auf die entsprechenden Slots
* **Formelsyntax**:
  * Band-Slots: `x`, `y`, `z`, `a`, `b`, `c` – sechs Positionen, die Sie durch Ziehen auf reale Kanäle zuordnen
  * Operatoren: `+`, `-`, `*`, `/`, `^` und `()` zum Gruppieren
  * Funktionen: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Warum Symbole statt Bandnamen**: Eine als `(y-x)/(y+x)` geschriebene Formel funktioniert auf jeder Kamera, da die Drag-and-Drop-Zuordnung entscheidet, ob `y` das 850-nm-NIR eines RGN-Filters oder das 808-nm-NIR eines OCN-Filters ist. Die integrierten Voreinstellungen werden auf dieselbe Weise gespeichert – siehe [Multispektralindex-Formeln](multispectral-index-formulas.md) für die genaue Symbolform aller 27.
* **Wo sie funktionieren**: Benutzerdefinierte Formeln werden mit den Projekteinstellungen gespeichert und können sowohl in der [Index/LUT-Sandbox](../image-viewer-gui/index-lut-sandbox.md) als auch bei der Bildverarbeitung verwendet werden. Sie werden von der CLI-/SDK-Namensliste `--indices` **nicht** akzeptiert, die lediglich die 22 Namen der integrierten Voreinstellungen auflistet.***

## Export

Diese Einstellungen steuern das Format und die Qualität der exportierten, verarbeiteten Bilder.

### Kalibriertes Bildformat

* **Typ**: Dropdown-Auswahl
* **Optionen**:
  * **TIFF (16-Bit)** – Unkomprimiertes 16-Bit-TIFF-Format
  * **TIFF (8-Bit)** – Unkomprimiertes 8-Bit-TIFF-Format
  * **TIFF (32-Bit, Prozent)** – 32-Bit-TIFF mit Gleitkomma-Reflexionswerten in Prozent
  * **PNG (8-Bit)** – Komprimiertes 8-Bit-PNG-Format
  * **JPG (8-Bit)** – Komprimiertes 8-Bit-JPEG-Format
* **Standard**: TIFF (16-Bit); bei einem reinen 8-Bit-Projekt wird TIFF (8-Bit) angezeigt und es werden nur die 8-Bit-Formate angeboten
* **Beschreibung**: Wählt das Dateiformat zum Speichern der verarbeiteten und kalibrierten Bilder aus. Die exportierten Dateien werden in einem formatbezogenen Unterordner innerhalb des jeweiligen Kameraordners abgelegt (`tiff16`, `tiff32`, `png8`, `jpg8`), wobei pro Produkt ein Ordner „`<Product>_Images/`“ vorhanden ist. Exportierte Dateien behalten den Quelldateinamen bei – das Produkt wird durch den Ordner und nicht durch eine Dateiendung identifiziert.
* **Formatempfehlungen**:
  * **TIFF (16-Bit)**: Empfohlen für wissenschaftliche Analysen und professionelle Arbeitsabläufe. Bewahrt maximale Datenqualität ohne Kompressionsartefakte. Am besten geeignet für multispektrale Analysen und die Weiterverarbeitung in GIS-Software.
  * **TIFF (32-Bit, Prozent)**: Am besten geeignet für Arbeitsabläufe, die Reflexionswerte als Prozentsätze (0–100 %) erfordern. Bietet höchste Präzision für radiometrische Messungen.
  * **PNG (8-Bit)**: Gut geeignet für die Anzeige im Web und allgemeine Visualisierung. Kleinere Dateigrößen durch verlustfreie Komprimierung, jedoch reduzierter Dynamikbereich.
  * **JPG (8-Bit)**: Kleinste Dateigrößen, nur für Vorschauen und die Darstellung im Web geeignet. Verwendet verlustbehaftete Komprimierung, die für wissenschaftliche Analysen nicht geeignet ist.
* **Hinweis**: LATTICE-Radiance-Daten werden unabhängig von dieser Einstellung immer als 32-Bit-Float-TIFF exportiert.

***

## Projektvorlage speichern

Mit dieser Funktion können Sie Ihre aktuellen Projekteinstellungen als wiederverwendbare Vorlage speichern.

* **Typ**: Texteingabe + Schaltfläche „Speichern“
* **Beschreibung**: Geben Sie einen aussagekräftigen Namen für Ihre Einstellungsvorlage ein und klicken Sie auf das Speichersymbol. Die Vorlage speichert alle Ihre aktuellen Projekteinstellungen (Zielerkennung, Verarbeitungsoptionen, Indizes, sowie das Exportformat) und können so in zukünftigen Projekten einfach wiederverwendet werden. Vorlagen werden im Ordner „`Project Templates/`“ innerhalb Ihres Projekt-Speicherordners abgelegt und können auch über das Hauptmenü ausgewählt oder exportiert werden (*Vorlage auswählen* / *Vorlage speichern* / *Vorlage exportieren*).
* **Anwendungsfälle**:
  * Erstellen Sie Vorlagen für verschiedene Kamerasysteme (RGB, multispektral, NIR)
  * Speichern Sie Standardkonfigurationen für bestimmte Kulturarten oder Analyse-Workflows
  * Stellen Sie einheitliche Einstellungen für das gesamte Team bereit
* **Vorgehensweise**:
  1. Konfigurieren Sie alle gewünschten Projekteinstellungen
  2. Geben Sie einen Vorlagennamen ein (z. B. „RedEdge Survey3 NDVI Standard“)
  3. Klicken Sie auf das Speichersymbol
  4. Die Vorlage kann nun beim Erstellen neuer Projekte geladen werden

***

## Projektordner speichern

Diese Einstellung legt fest, wo neue Projekte standardmäßig gespeichert werden.

* **Typ**: Anzeige des Verzeichnispfads + Schaltfläche „Bearbeiten“
* **Standard (unter Windows)**: `Documents\MAPIR\Chloros Projects` in Ihrem Ordner „Dokumente“ – in der Regel `C:\Users\[Username]\Documents\MAPIR\Chloros Projects` oder an dem Ort, an den OneDrive oder ein Roaming-Profil den Ordner „Dokumente“ verschoben hat
* **Standard (unter Linux)**: `~/Documents/MAPIR/Chloros Projects`
* **Beschreibung**: Zeigt das aktuelle Standardverzeichnis an, in dem neue Chloros-Projekte erstellt werden. Klicken Sie auf das Bearbeitungssymbol, um ein anderes Verzeichnis auszuwählen. Die Überschreibung wird als einzelne Textzeile in `~/.chloros/working_directory.txt` gespeichert – unter Windows ist dies `C:\Users\<Username>\.chloros\working_directory.txt`. Fehlt diese Datei oder verweist sie auf einen Pfad, der nicht mehr existiert, greift Chloros auf die oben genannte Standardeinstellung zurück. Die CLI-Schnittstelle liest und schreibt dieselbe Datei, sodass sich `chloros-cli` und die GUI stets darüber einig sind, wo sich die Projekte befinden.
* **Projektvorlagen** befinden sich in einem Unterordner namens `Project Templates/` dieses Verzeichnisses.
* **Wann eine Änderung erforderlich ist**:
  * Legen Sie ein Netzlaufwerk fest, um die Zusammenarbeit im Team zu ermöglichen
  * Wechseln Sie zu einem Laufwerk mit mehr Speicherplatz für große Datensätze
  * Organisieren Sie Projekte nach Jahr, Kunde oder Projekttyp in verschiedenen Ordnern
* **Hinweis**: Eine Änderung dieser Einstellung wirkt sich nur auf NEUE Projekte aus. Bestehende Projekte verbleiben an ihren ursprünglichen Speicherorten.
* **Windows-Pfadlänge**: Sofern lange Windows-Pfade nicht aktiviert sind (Option **„Enable Windows long paths“** im Installationsprogramm), wird ein Projektordner, dessen Pfadlänge mehr als etwa 130 Zeichen beträgt, vor der Verarbeitung abgelehnt, da dessen Ergebnisse die Windows-Grenze von 260 Zeichen überschreiten würden. Siehe [Starten der Verarbeitung](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Speicherung der Einstellungen

Ein Chloros-Projekt ist ein **Ordner**. Alle Projekteinstellungen werden darin unter `project.json` gespeichert; angeschlossene Hardware wird zusammen mit diesen in den Dateien „`cameras.json`“ und „`sensors.json`“ gespeichert, sodass beim erneuten Öffnen eines Projekts auch dessen Kameras und Lichtsensoren wieder verbunden werden. Wenn Sie ein Projekt erneut öffnen, werden alle Einstellungen genau so wiederhergestellt, wie Sie sie hinterlassen haben. Gespeicherte Projekte können auch im Headless-Modus mit „`chloros-cli project`“ oder der -Datei „`open_project`“ des SDK gesteuert werden.

### Einstellungshierarchie

Einstellungen werden in der folgenden Reihenfolge angewendet:

1. **Systemstandardwerte** – Von Chloros definierte integrierte Standardwerte 2. **Vorlageneinstellungen** – Wenn Sie beim Erstellen eines Projekts eine Vorlage laden
3. **Gespeicherte Projekteinstellungen** – Mit der Projektdatei gespeicherte Einstellungen
4. **Manuelle Anpassungen** – Alle Änderungen, die Sie während der aktuellen Sitzung vornehmen

### Einstellungen und Bildverarbeitung

Die Verarbeitungseinstellungen werden beim Start eines Verarbeitungslaufs geladen. Das Ändern einer Einstellung wirkt sich nicht rückwirkend auf bereits auf der Festplatte gespeicherte Ergebnisse aus – führen Sie die Verarbeitung erneut durch, um die neuen Einstellungen anzuwenden. Einige Einstellungen haben keinerlei Auswirkungen auf die Verarbeitung:

* Auflösung der Bildminiaturansichten (nur zur Anzeige)
* Projektvorlage speichern
* Projektordner speichern

***

## Referenz zu Konfigurationsschlüsseln

Für die Automatisierung (CLI: `--config`, SDK: `configure` oder beim direkten Auslesen von `project.json`) sind dies die genauen Schlüssel unter `Project Settings`:

| Schlüsselpfad | Typ | Standardwert |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | Zahl zwischen 0 und 10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | Zahl zwischen 0 und 100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | Zahl 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | Zahl -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | Cap-Profil-ID oder `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | Liste der Indexkonfigurationen | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Die Schlüssel `Array alignment` werden beim ersten Rendern des Abschnitts „Array Alignment“ oder bei einem Automationsaufruf geschrieben, der sie setzt. Solange sie fehlen, verwendet die Pipeline dieselben Werte wie oben angegeben (`true`, `true`, bilinear), sodass sich ein Projekt.json ohne diese Schlüssel genauso verhält wie eines mit ihnen.

### In „`project.json`“ gespeicherte Schlüssel ohne Steuerelemente im Einstellungsfenster

Diese befinden sich unter derselben `Project Settings`-Struktur und werden bei der Verarbeitung gelesen, aber in der Seitenleiste gibt es kein Widget dafür:

| Schlüsselpfad | Typ | Standardwert | Festgelegt durch |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Überschreibt die Interpretation von LATTICE-Eingabe-TIFFs; `auto` leitet dies aus dem XMP-Tag jeder Datei sowie der Kanalanzahl ab. Wird bei Survey3-Aufnahmen ignoriert. `.raw`. Absichtlich keine GUI-Einstellung – „auto“ ist in jedem normalen Fall korrekt. |
| `Processing → Target reflectance dir` | Pfadzeichenfolge | `""` | `chloros-cli process --target-reflectance-dir` oder die Projektziel-API |
| `Processing → Target reflectance config` | nach Kameraseriennummer indiziertes Wörterbuch | `{}` | Registrierung eines Ziels innerhalb des Bildausschnitts (Modus `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | Pfadzeichenfolge | `""` | SDK `process_folder(daq_log_path=…)`. Verweist auf eine `.daq`-Aufzeichnung oder einen Ordner mit solchen Aufzeichnungen |
| `Target Detection → Minimum calibration target squares` | Zahl | `4` | Veralteter Standardwert; keine Steuerung und kein CLI-Flag |
| `UI → Grid thumbnail size` | Zahl | `160` | Der eigene Miniaturansicht-Zoomregler des Bildrasters |

Zwei Viewer-Einstellungen werden **auf oberster Ebene in `project.json`** gespeichert, vollständig außerhalb von `Project Settings`, da es sich um Anzeigezustände und nicht um Verarbeitungseinstellungen handelt:

| Schlüsselpfad | Typ | Standardwert | Festgelegt durch |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | Ganzzahl 1–256 | `1` | Das GSD-Steuerelement (px) der Bildregisterkarte – siehe [Bild im Vollbildmodus öffnen](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Bewährte Vorgehensweisen

1. **Mit den Standardeinstellungen beginnen**: Die Standardeinstellungen eignen sich gut für die meisten MAPIR-Kamerasysteme und typische Arbeitsabläufe.
2. **Vorlagen erstellen**: Sobald Sie die Einstellungen für einen bestimmten Arbeitsablauf oder eine bestimmte Kamera optimiert haben, speichern Sie diese als Vorlage, um die Konsistenz über alle Projekte hinweg zu gewährleisten.
3. **Vor der vollständigen Verarbeitung testen**: Wenn Sie mit neuen Einstellungen experimentieren, testen Sie diese zunächst an einer kleinen Auswahl von Bildern, bevor Sie Ihren gesamten Datensatz verarbeiten.
4. **Dokumentieren Sie Ihre Einstellungen**: Verwenden Sie aussagekräftige Vorlagennamen, die das Kamerasystem, die Art der Verarbeitung und den Verwendungszweck angeben (z. B. „Survey3\_RGB\_NDVI\_Agriculture“).
5. **Auswahl des Exportformats**: Wählen Sie Ihr Exportformat entsprechend Ihrer Endanwendung aus:
   * Wissenschaftliche Analyse → TIFF (16-Bit oder 32-Bit)
   * GIS-Verarbeitung → TIFF (16-Bit)
   * Schnelle Visualisierung → PNG (8-Bit)
   * Web-Freigabe → JPG (8-Bit)

***

Weitere Informationen zu multispektralen Indizes in Chloros finden Sie auf der Seite [Formeln für multispektrale Indizes](multispectral-index-formulas.md).
