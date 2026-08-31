# Projekteinstellungen

Über die Seitenleiste „Projekteinstellungen“<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

in Chloros können Sie alle Aspekte der Bildverarbeitung, der Erkennung von Kalibrierungszielen, der Berechnung multispektraler Indizes sowie der Exportoptionen für Ihr Projekt konfigurieren. Diese Einstellungen werden zusammen mit Ihrem Projekt gespeichert und können als Vorlagen gespeichert werden, um sie in mehreren Projekten wiederzuverwenden.

## Aufrufen der Projekteinstellungen

So rufen Sie die Projekteinstellungen auf:

1. Öffnen Sie ein Projekt in Chloros
2. Klicken Sie in der linken Seitenleiste auf die Registerkarte **Projekteinstellungen**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">


3. Im Einstellungsbereich werden alle verfügbaren Konfigurationsoptionen nach Kategorien geordnet angezeigt

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Einstellungen, die von anderen Einstellungen abhängen, sind ausgegraut.** Wenn eine übergeordnete Option eine Einstellung unmöglich macht (beispielsweise macht das Deaktivieren von *Reflexionskalibrierung / Weißabgleich* die Option *Reflexion exportieren* unmöglich), wird das abhängige Steuerelement deaktiviert und in seinem Tooltip wird die Option genannt, die geändert werden muss.
{% endhint %}

***

## Anzeige

### Auflösung der Bildminiaturansichten

* **Typ**: Dropdown-Auswahl
* **Optionen**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Standard**: Standard (512 px)
* **Beschreibung**: Auflösung (längste Kante, in Pixeln), mit der die Miniaturansichten im Bildraster gerendert werden. Höhere Werte sehen beim Vergrößern schärfer aus, werden jedoch langsamer geladen und beanspruchen mehr Speicher. Die volle Auflösung entspricht der Originalbildgröße.
* **Hinweis**: Nur zur Anzeige – dies hat keinerlei Auswirkungen auf die Verarbeitung oder exportierte Dateien.***

## Zielerkennung

Diese Einstellungen steuern, wie Chloros Kalibrierungsziele in Ihren Bildern erkennt und verarbeitet. Beide sind nur aktiv, solange **Reflexionskalibrierung / Weißabgleich** aktiviert ist (ansonsten sind sie ausgegraut, da die Zielerkennung vollständig übersprungen wird).

### Mindestfläche für Kalibrierungsmuster (px)

* **Typ**: Zahl
* **Bereich**: 0 bis 10.000 Pixel
* **Standardwert**: 25 Pixel
* **Beschreibung**: Legt die Mindestfläche (in Pixeln) fest, die erforderlich ist, damit ein erkannter Bereich als gültige Kalibrierungszielprobe gewertet wird. Kleinere Werte erkennen zwar kleinere Ziele, können jedoch zu mehr Fehlalarmen führen. Größere Werte erfordern größere, deutlichere Zielbereiche für die Erkennung.
* **Wann anpassen**:
  * Erhöhen Sie den Wert, wenn Sie bei kleinen Bildartefakten falsche Erkennungen erhalten.
  * Verringern Sie den Wert, wenn Ihre Kalibrierungsziele in Ihren Bildern klein erscheinen und nicht erkannt werden.

### Minimale Zielclusterbildung (0–100)

* **Typ**: Zahl
* **Bereich**: 0 bis 100
* **Standardwert**: 60
* **Beschreibung**: Steuert den Clustering-Schwellenwert für die Gruppierung ähnlich gefärbter Bereiche bei der Erkennung von Kalibrierungszielen. Höhere Werte erfordern, dass mehr ähnliche Farben zusammengefasst werden, was zu einer konservativeren Zielerkennung führt. Niedrigere Werte lassen größere Farbabweichungen innerhalb einer Zielgruppe zu.
* **Wann anpassen**:
  * Erhöhen Sie den Wert, wenn Kalibrierungsziele in mehrere Erkennungen aufgeteilt werden.
  * Verringern Sie den Wert, wenn Kalibrierungsziele mit Farbabweichungen nicht vollständig erkannt werden.

***

## Verarbeitung

Diese Einstellungen steuern, wie Chloros Ihre Bilder verarbeitet und kalibriert.

### Vignettierungskorrektur

* **Typ**: Kontrollkästchen
* **Standard**: Aktiviert (markiert)
* **Beschreibung**: Wendet eine Vignettierungskorrektur an, um die Abdunkelung an den Bildrändern durch das Objektiv auszugleichen. Vignettierung ist ein häufiges optisches Phänomen, bei dem die Ecken und Ränder eines Bildes aufgrund der Eigenschaften des Objektivs dunkler erscheinen als die Bildmitte.
* **Nebenwirkung**: Mit diesem Schalter wird auch ausgewählt, welches *unkalibrierte Fallback-Produkt* ein Lauf schreibt (siehe unten).

### Reflexionskalibrierung / Weißabgleich

* **Typ**: Kontrollkästchen
* **Standard**: Aktiviert (aktiviert)
* **Beschreibung**: Aktiviert die Reflektanzkalibrierung – basierend auf erkannten Kalibrierungszielen im Bildausschnitt und/oder Daten des DAQ-Lichtsensors zur Lichtintensität, je nach Kamera und verfügbaren Daten. Dies normalisiert die Reflektanzwerte in Ihrem Datensatz und gewährleistet konsistente Messungen unabhängig von den Lichtverhältnissen.
* **Wenn deaktiviert**: Die Zielerkennung wird vollständig übersprungen, und**es kann von keiner Kamera ein Reflexionsprodukt erzeugt werden** – weder bei Survey3-zielgesteuerten noch bei LATTICE-DAQ-gesteuerten Systemen. Die davon abhängigen Einstellungen (*Reflexion exportieren*, *Mindestintervall für die Neukalibrierung* und die Schwellenwerte für die Zielerkennung) sind ausgegraut.

### Fallback-Produkte ohne Kalibrierung: Sensorantwort exportieren / Vignettenkorrektur exportieren

* **Typ**: Zwei Kontrollkästchen
* **Standardeinstellungen**: Beide aktiviert (markiert)
* **Beschreibung**: Wenn ein Bild nicht reflektanzkalibriert werden kann (es wurde kein Kalibrierungsziel gefunden oder die Reflektanzkalibrierung ist deaktiviert), wird es stattdessen als *unkalibriertes Fallback-Produkt* gespeichert. **Pro Durchlauf gibt es genau eines der beiden Fallback-Produkte für jedes Kameramodell**, ausgewählt über den Schalter *Vignettenkorrektur*:
  * Vignettenkorrektur **ein**→ `Vignette_Corrected_Images/` (gesteuert durch**„Export vignette corrected“**)
  * Vignettenkorrektur **aus**→ `Sensor_Response_Images/` (gesteuert durch**„Export sensor response“**)
* Das nicht aktive Fallback-Produkt ist ausgegraut. Wird das Häkchen bei der nicht aktiven Option entfernt, wird die entsprechende Datei gar nicht erst geschrieben.

### LATTICE-Exportprodukte

Bei Projekten, die LATTICE-Aufnahmen enthalten, wird jedes importierte LATTICE-Bild in einem einzigen Verarbeitungsdurchlauf auf alle aktivierten **und anwendbaren**Produkte verteilt. Vier Kontrollkästchen steuern diese Verteilung (alle standardmäßig**aktiviert**):

| Einstellung | Ausgabeordner | Was wird exportiert? |
| --- | --- | --- |
| **Export debayered** | `Debayered_Images/` | Das linear debayered Bild. Gilt für RGB und Multispektralkameras. |
| **Vorschau exportieren** | `Preview_Images/` | Die Anzeigevorschau. RGB = Weißabgleich (DAQ-Lichtart, falls verfügbar, sonst Grauwelt) + Gamma; multispektral = Falschfarben-Streckung. |
| **Export-Strahlungsdichte** | `Radiance_Images/` | Spektrale Strahlungsdichte als Float32 in W/m²/sr/nm. Nur für Multispektral (M3C/M3M) – gilt nicht für RGB-Master. Wird unabhängig von der Einstellung *Kalibriertes Bildformat* immer als 32-Bit-Wert TIFF geschrieben. |
| **Reflexionsgrad beim Export**| `Reflectance_Calibrated_Images/` | Uint16-Reflexionsgrad, so skaliert, dass**32768 = Reflexionsgrad 1,0** (als XMP-Tag `Chloros:PixelScale` vermerkt). Nur multispektral, geschrieben, wenn ein passender `.daq`-Downwelling-Datensatz (oder ein QA-konformes In-Frame-Ziel) das Bild abdeckt. |

* RGB-Masterkameras geben debayered + Vorschau aus; Strahlungsintensität/Reflexionsgrad werden für sie übersprungen, da sie nicht zutreffen.
* Die Bittiefe von debayertem Bild/Vorschau richtet sich nach der Einstellung *Kalibriertes Bildformat*; die Strahlungsintensität ist immer float32.
* Die Verarbeitung von Survey3 wird von diesen vier Schaltern nicht beeinflusst.

Die gleichen vier Schalter existieren im Headless-Modus als `chloros-cli process --debayered / --preview / --radiance / --reflectance` sowie als entsprechende Parameter von SDK. Sie ersetzen das alte `--radiometric-output`-Flag, das nicht mehr existiert.

{% hint style="warning" %}
**Das Deaktivieren aller anwendbaren Produkte führt zum Fehlschlagen des Laufs.** Ab Version 1.2.0 meldet ein Verarbeitungslauf, der Produkte angefordert hat, aber keine Bildprodukte geschrieben hat, einen Fehler, und CLI beendet sich mit einem Wert ungleich Null, anstatt einen stillen Erfolg zu melden. Das Protokoll nennt das Produkt, das nicht geschrieben werden konnte, und den Grund dafür. Ein bewusst nur auf Metadaten ausgerichteter Lauf (keine Anfragen) ist weiterhin erfolgreich.
{% endhint %}

### Reflexionsquelle (Projekteinstellung, festgelegt über CLI/SDK)

Das Projekt speichert außerdem, welche **Reflexionsreferenz** das LATTICE-Reflexionsprodukt verwendet. Es gibt keine eigene Steuerung im Einstellungsfenster; der Wert wird in der Projektkonfiguration als `Processing → "Target reflectance source"` gespeichert und über `chloros-cli process --reflectance-source {auto,target,daq}` oder den Parameter `reflectance_source` von SDK festgelegt:

* **`auto`** (Standard): Ein QA-konformes Kalibrierungsziel im Bild wird zur absoluten Referenz; bei Fehlen eines Ziels oder bei fehlgeschlagener Qualitätssicherung (QA) wird auf die DAQ-Downwelling-Teilung (ρ = πL/E) zurückgegriffen, wenn kein Ziel vorhanden ist oder die Qualitätssicherung fehlschlägt.
* **`target`**: strikte zielgesteuerte Reflektanz – keine DAQ-Ersetzung.
* **`daq`**: DAQ-maßgebliche Reflektanz; Ziele innerhalb des Bildausschnitts werden nicht als Referenz verwendet.

Der gespeicherte Wert wird ohne Berücksichtigung der Groß-/Kleinschreibung abgeglichen, und einige Schreibweisen werden als Aliase akzeptiert: `target`, `target_image`, `empirical` und `empirical_line` bedeuten alle **Ziel**; `daq`, `dls`, `light_sensor` und `sensor` bedeuten alle**daq**. Alles andere – einschließlich eines fehlenden Schlüssels – wird zu**auto** aufgelöst.**Gemessene** Ziel-Scans pro Einheit werden anhand der Seriennummer/des QR-Codes der Zieleinheit nachgeschlagen, wie z. B. `<serial>.csv`, an drei Stellen: im mit `--target-reflectance-dir` angegebenen Verzeichnis (gespeichert als `Processing → "Target reflectance dir"`), im projektspezifischen Ordner `target_reflectance/` und im Pfad der Umgebungsvariable `CHLOROS_TARGET_REFLECTANCE_DIR`. Wenn für diese Einheit kein gemessener Scan vorliegt, wird stattdessen die veröffentlichte Nennkurve für das Zielmodell verwendet.

### Debayer-Methode

* **Typ**: Auswahl aus Dropdown-Menü
* **Optionen**:
  * Standard (schnell, mittlere Qualität)
  * Texturorientiert (langsam, höchste Qualität) \[Chloros+]
* **Standard**: Standard (Schnell, mittlere Qualität)
* **Beschreibung**: Wählt den Demosaicing-Algorithmus aus, der zur Umwandlung der Rohdaten eines Sensors mit Bayer-Muster in Vollfarbbilder verwendet wird. Die Methode „Standard (Schnell, mittlere Qualität)“ bietet ein optimales Gleichgewicht zwischen Verarbeitungsgeschwindigkeit und Bildqualität. Die Methode „Texturorientiert (Langsam, höchste Qualität)“ \[Chloros+] verwendet ein hochwertiges, kantenbewusstes Demosaicing in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Demosaicing-Rauschen entfernt. Das „Texture Aware“-Modell benötigt GPU-Speicher (VRAM). Wir empfehlen die Verwendung, wenn Ihnen &gt;4 GB VRAM zur Verfügung stehen, um eine schnellere Verarbeitung zu erzielen.
* **Wenn es sich bei der Zeile überhaupt um ein Dropdown-Menü handelt**: Das Dropdown-Menü mit zwei Optionen erscheint nur, wenn**beide**Bedingungen erfüllt sind – Sie sind mit einem berechtigten Chloros+-Abonnement angemeldet und**und** das Projekt keine LATTICE-Aufnahmen enthält. Andernfalls wird die Zeile als reiner Text mit dem Wortlaut „`Standard (Fast, Medium Quality)`“ dargestellt, ohne dass eine Auswahlmöglichkeit besteht.
* **Hinweis zu LATTICE**: Es gibt kein für LATTICE trainiertes „Texture Aware“-Modell, und die Pipeline erzwingt unabhängig vom gespeicherten Wert das Standard-Demosaicing für LATTICE-Frames. Wenn Sie einem Projekt, bei dem bereits „Texture Aware“ ausgewählt war, einen LATTICE-Ordner hinzufügen, setzt Chloros die Einstellung wieder auf „Standard“ zurück, anstatt einen veralteten Wert in `project.json` beizubehalten.

### Minimales Neukalibrierungsintervall

* **Typ**: Zahl
* **Bereich**: 0 bis 3.600 Sekunden
* **Standardwert**: 0 Sekunden
* **Beschreibung**: Legt das minimale Zeitintervall (in Sekunden) zwischen der Verwendung von Kalibrierungszielen fest. Bei der Einstellung 0 verwendet Chloros jedes erkannte Kalibrierungsziel. Bei der Einstellung eines höheren Werts verwendet Chloros nur Kalibrierungsziele, die mindestens diesen Zeitabstand voneinander haben, wodurch sich die Verarbeitungszeit für Datensätze mit häufigen Erfassungen von Kalibrierungszielen verkürzt.
* **Wann anpassen**:
  * Stellen Sie den Wert auf 0 ein, um bei wechselnden Lichtverhältnissen maximale Kalibrierungsgenauigkeit zu erzielen.
  * Erhöhen Sie den Wert (z. B. auf 60–300 Sekunden), um die Verarbeitung zu beschleunigen, wenn die Lichtverhältnisse konstant sind und Sie häufig Bilder von Kalibrierungszielen erhalten.

### Zeitzonenversatz des Lichtsensors

* **Typ**: Zahl
* **Bereich**: -12 bis +12 Stunden
* **Standardwert**: 0 Stunden
* **Beschreibung**: Gibt den Zeitzonen-Offset (in Stunden gegenüber UTC) für die Zeitstempel der Lichtsensordaten an, der beim Abgleichen von Lichtsensor-Protokollen mit den Bildaufnahmezeiten verwendet wird. Neuere `.daq`-Aufzeichnungen enthalten ihre eigene Zeitzonenangabe, sodass dies hauptsächlich für ältere, in Ortszeit aufgezeichnete Protokolle erforderlich ist.

### PPK-Korrekturen anwenden

* **Typ**: Kontrollkästchen
* **Standard**: Deaktiviert (nicht markiert)
* **Beschreibung**: Aktiviert die Verwendung von Post-Processed Kinematic (PPK)-Korrekturen von MAPIR-DAQ-Aufzeichnungsgeräten, die über ein GPS (GNSS) verfügen. Wenn diese Option aktiviert ist, verwendet Chloros alle .daq-Protokolldateien in Ihrem Projektverzeichnis, die Belichtungs-Pin-Daten enthalten, und wendet präzise Geolokalisierungs-Korrekturen auf Ihre Bilder an.
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
  * Eine Kamera + ein Pin: Die Kamera wird automatisch ausgewählt
  * Eine Kamera + zwei Pins: Pin 1 wird der Kamera automatisch zugewiesen
  * Mehrere Kameras: Manuelle Auswahl erforderlich

### Belichtungs-Pin 2

* **Typ**: Auswahl aus Dropdown-Menü
* **Sichtbarkeit**: Nur sichtbar, wenn „PPK-Korrekturen anwenden“ aktiviert ist UND Belichtungsdaten für Pin 2 verfügbar sind
* **Optionen**:
  * Im Projekt erkannte Kameramodellnamen
  * „Nicht verwenden“ – Diesen Belichtungs-Pin ignorieren
* **Standard**: Wird basierend auf der Projektkonfiguration automatisch ausgewählt
* **Beschreibung**: Weist dem Belichtungs-Pin 2 eine bestimmte Kamera für die PPK-Zeitsynchronisation zu, wenn eine Konfiguration mit zwei Kameras verwendet wird.
* **Verhalten bei der automatischen Auswahl**:
  * Eine Kamera + ein Pin: Pin 2 wird automatisch auf „Nicht verwenden“ gesetzt
  * Eine Kamera + zwei Pins: Pin 2 wird automatisch auf „Nicht verwenden“ gesetzt
  * Mehrere Kameras: Manuelle Auswahl erforderlich
* **Hinweis**: Dieselbe Kamera kann nicht gleichzeitig Pin 1 und Pin 2 zugewiesen werden.***

## DAQ-Lichtsensor

Dieser Abschnitt erscheint in den Projekteinstellungen und listet alle DAQ-Downwelling-Dateien im Projekt auf – `.daq`-Aufzeichnungen und DAQ-M `.csv`-Downwelling-Protokolle. Aufzeichnungen, die auf der Registerkarte „Lichtsensoren“ erstellt wurden, werden automatisch zum geöffneten Projekt hinzugefügt.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Jede Zeile zeigt die Datei, das Sensormodell und die für diese Datei tatsächlich geltende Diffusorkappenkorrektur an. Über den Zeilen befindet sich ein einziges projektweites Steuerelement:

### Kappenüberschreibung (alle Dateien)

* **Typ**: Dropdown-Auswahl
* **Optionen**: `Auto` sowie die für die im Projekt vorhandenen Sensortypen gültigen Korrekturprofile für die Diffusorkappe
* **Standard**: Auto
* **Gespeichert als**: `Processing → "DAQ cap id"` (Standard: `auto`)
* **Beschreibung**: `Auto` verwendet die in jeder Datei aufgezeichnete Obergrenze (wird nichts aufgezeichnet, wird die „Sunshine“-Obergrenze angenommen – alle MAPIR-Datenerfassungen werden mit dem „Sunshine“-Korrektor ausgeliefert). Die Auswahl einer bestimmten Kappe überschreibt**alle** Downwelling-Dateien im Projekt: Rohaufzeichnungen werden damit korrigiert, und Aufzeichnungen, die bereits eine Kappe enthalten, werden neu referenziert (die aufgezeichnete Korrektur wird rückgängig gemacht und die ausgewählte angewendet).
* **Wichtig**: Die ausgewählte Kappe muss mit der Kappe übereinstimmen, die während der Aufzeichnung physisch angebracht war. Weder der Sensor noch die Software können die physische Kappe erkennen – eine nicht übereinstimmende Kappen-ID führt zu einer fehlerhaften Korrektur der Spektren.

Es gibt bewusst **nur eine** projektweite Steuerung anstelle von Dropdown-Menüs pro Datei: Die Einstellung gilt für jede Downwelling-Quelle im Projekt.***

## Array-Ausrichtung

Dieser Abschnitt wird **nur** angezeigt, wenn mindestens ein Bild im Projekt die Modul-zu-Modul-Ausrichtungstransformation enthält, die LATTICE-Arrays zum Zeitpunkt der Aufnahme einfügen (XMP-Tags `Chloros:Alignment*`). Er zeigt an, wie viele Bilder Ausrichtungs-Tags enthalten, welche Kamera die Referenz ist (`REF`-Kennzeichnung) ist sowie eine kameraspezifische Tabelle mit der Anzahl der Bilder.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Array-Ausrichtung anwenden

* **Typ**: Kontrollkästchen
* **Standard**: Aktiviert (markiert)
* **Gespeichert als**: `Processing → "Array alignment"`
* **Beschreibung**: Verzerrt jedes verarbeitete Produkt (Debayering / Vorschau / Strahlungsintensität / Reflexionsgrad / Index) unter Verwendung der bei der Aufnahme eingeprägten Transformation in die gemeinsame Referenzgeometrie des Arrays. Aus = Export in der nativen Geometrie pro Sensor.

### Auf gemeinsame Überlappung zuschneiden

* **Typ**: Kontrollkästchen (nur aktiv, wenn *Array-Ausrichtung anwenden* aktiviert ist)
* **Standard**: Aktiviert (markiert)
* **Gespeichert als**: `Processing → "Array alignment crop"`
* **Beschreibung**: Schneidet ausgerichtete Exporte auf den Bereich zu, den alle Kameramodule gemeinsam nutzen, sodass jedes Band denselben Abdruck hat. Bei „Aus“ bleibt die vollständige Sensorfläche erhalten (schwarze Füllung außerhalb der Quelle).

### Neuberechnung

* **Typ**: Dropdown-Auswahl (nur aktiv, wenn *Array-Ausrichtung anwenden* aktiviert ist)
* **Optionen**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Standard**: Bilinear
* **Gespeichert als**: `Processing → "Array alignment interpolation"`
* **Beschreibung**: Vom Ausrichtungs-Warp verwendete Interpolation. „Nearest“ bewahrt die exakten Quellwerte (keine Vermischung zwischen Pixeln) für eine strenge radiometrische Analyse; „Bilinear“ eignet sich am besten für Kartierungen und visuelle Anwendungen.

Die gleichen drei Optionen sind auch ohne Vorfix als `chloros-cli process --array-alignment`, `--array-alignment-crop` und `--array-alignment-interp {bilinear,nearest,cubic}` verfügbar.

***

## Index

Mit diesen Einstellungen können Sie multispektrale Indizes für die Analyse und Visualisierung konfigurieren.

### Index hinzufügen

* **Typ**: Spezielles Konfigurationsfenster für Indizes
* **Beschreibung**: Öffnet ein interaktives Fenster, in dem Sie multispektrale Vegetationsindizes (NDVI, NDRE, EVI usw.) auswählen und konfigurieren können, die während der Bildverarbeitung berechnet werden sollen. Sie können mehrere Indizes hinzufügen, wobei jeder seine eigenen Visualisierungseinstellungen hat.
* **Verfügbare Indizes**: Das Dropdown-Menü der Benutzeroberfläche enthält**27** vordefinierte multispektrale Indexformeln (siehe [Multispektrale Indexformeln](multispectral-index-formulas.md) für die vollständige Liste, einschließlich der Namen, die auch von der Option CLI/SDK `--indices`-Option akzeptiert werden).
* **Funktionen**:
  * Auswahl aus vordefinierten Indexformeln
  * Ziehen Sie die Filterkanäle Ihrer Kamera auf die Bandplätze der Formel
  * Konfigurieren Sie Farbverläufe für die Visualisierung (LUT – Look-Up-Tabellen)
  * Legen Sie Schwellenwerte und Clipping-Modi fest
  * Erstellen Sie benutzerdefinierte Indexformeln
* **Hinweis**: Für einbandige LATTICE M3M-Monochromkameras werden keine Indizes berechnet – Multiband-Indizes sind bei einem Band undefiniert. Survey3 und LATTICE M3C sind davon nicht betroffen.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Jeder Index, den Sie hinzufügen, rendert seine Formel als mathematische Berechnung, mit einem farbigen Kreis pro Bandplatz: rot = Red, grün = Green, blau = Blue, orange = Orange, cyan = Cyan, lila = NIR, magenta = RE. Ziehen Sie einen Kreis aus der Zeile über der Formel auf einen Slot, um ihn zuzuordnen; doppelklicken Sie auf einen zugeordneten Slot, um ihn zu löschen. Der Index wird erst berechnet, wenn jeder von der Formel verwendete Slot über einen Kanal verfügt.

### Benutzerdefinierte Formeln (Chloros+-Funktion)

* **Typ**: Array mit Definitionen benutzerdefinierter Formeln
* **Verfügbarkeit**: Erfordert die Anmeldung mit einem berechtigten Chloros+-Abonnement.
* **Beschreibung**: Ermöglicht das Erstellen und Speichern benutzerdefinierter multispektraler Indexformeln mithilfe von Bandberechnungen. Benutzerdefinierte Formeln werden zusammen mit Ihren Projekteinstellungen gespeichert und können genau wie integrierte Indizes verwendet werden.
* **So erstellen Sie eine Formel**:
  1. Öffnen Sie im Konfigurationsfenster „Index“ den Rechner für benutzerdefinierte Formeln
  2. Geben Sie die Formel unter Verwendung der **Band-Slot-Symbole** ein, nicht der Bandnamen
  3. Speichern Sie die Formel unter einem aussagekräftigen Namen – sie erscheint dann am Ende des Formel-Dropdown-Menüs, und Sie ziehen die Kanalkreise Ihrer Kamera genau wie bei einer integrierten Voreinstellung auf die entsprechenden Slots
* **Formelsyntax**:
  * Band-Slots: `x`, `y`, `z`, `a`, `b`, `c` — sechs Positionen, die Sie durch Ziehen den tatsächlichen Kanälen zuordnen
  * Operatoren: `+`, `-`, `*`, `/`, `^` und `()` zum Gruppieren
  * Funktionen: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Warum Symbole statt Bandnamen**: Eine Formel in der Form `(y-x)/(y+x)` funktioniert bei jeder Kamera, da die Drag-and--Drop-Zuordnung entscheidet, ob `y` das 850-nm-NIR eines RGN-Filters oder das 808-nm-NIR eines OCN-Filters ist. Die integrierten Voreinstellungen werden auf dieselbe Weise gespeichert – siehe [Multispektrale Indexformeln](multispectral-index-formulas.md) für die genaue Symbolform aller 27.
* **Wo sie funktionieren**: Benutzerdefinierte Formeln werden mit den Projekteinstellungen gespeichert und können sowohl in der [Index/LUT-Sandbox](../image-viewer-gui/index-lut-sandbox.md) als auch bei der Bildverarbeitung verwendet werden. Sie werden**nicht** von der Namensliste CLI/SDK `--indices` akzeptiert, die lediglich die 22 integrierten Voreinstellungsnamen erweitert.***

## Export

Diese Einstellungen steuern das Format und die Qualität der exportierten, bearbeiteten Bilder.

### Kalibriertes Bildformat

* **Typ**: Auswahl aus der Dropdown-Liste
* **Optionen**:
  * **TIFF (16-Bit)** – Unkomprimiertes 16-Bit-TIFF-Format
  * **TIFF (32-Bit, Prozent)** – 32-Bit-Gleitkomma-TIFF mit Reflexionswerten in Prozent
  * **PNG (8-Bit)** – Komprimiertes 8-Bit-PNG-Format
  * **JPG (8-Bit)** – Komprimiertes 8-Bit-JPEG-Format
* **Standard**: TIFF (16-Bit)
* **Beschreibung**: Wählt das Dateiformat zum Speichern bearbeiteter und kalibrierter Bilder aus. Die exportierten Dateien werden in einem formatbezogenen Unterordner innerhalb des jeweiligen Kameraordners abgelegt (`tiff16`, `tiff32`, `png8`, `jpg8`), wobei pro Produkt ein Ordner „`<Product>_Images/`“ vorhanden ist. Exportierte Dateien behalten den Quelldateinamen bei – das Produkt wird durch den Ordner und nicht durch eine Dateiendung identifiziert.
* **Empfehlungen zum Format**:
  * **TIFF (16-Bit)**: Empfohlen für wissenschaftliche Analysen und professionelle Arbeitsabläufe. Bewahrt maximale Datenqualität ohne Kompressionsartefakte. Am besten geeignet für multispektrale Analysen und die Weiterverarbeitung in GIS-Software.
  * **TIFF (32-Bit, Prozent)**: Am besten geeignet für Arbeitsabläufe, die Reflexionswerte in Prozent (0–100 %) erfordern. Bietet maximale Präzision für radiometrische Messungen.
  * **PNG (8-Bit)**: Gut geeignet für die Webanzeige und allgemeine Visualisierung. Geringere Dateigrößen durch verlustfreie Komprimierung, jedoch reduzierter Dynamikbereich.
  * **JPG (8-Bit)**: Kleinste Dateigrößen, nur für Vorschauen und die Darstellung im Web geeignet. Verwendet verlustbehaftete Komprimierung, die für wissenschaftliche Analysen nicht geeignet ist.
* **Hinweis**: LATTICE-Radiance wird unabhängig von dieser Einstellung immer als 32-Bit-Float-TIFF exportiert.***

## Projektvorlage speichern

Mit dieser Funktion können Sie Ihre aktuellen Projekteinstellungen als wiederverwendbare Vorlage speichern.

* **Typ**: Texteingabefeld + Schaltfläche „Speichern“
* **Beschreibung**: Geben Sie einen aussagekräftigen Namen für Ihre Einstellungsvorlage ein und klicken Sie auf das Speichersymbol. Die Vorlage speichert alle Ihre aktuellen Projekteinstellungen (Zielerkennung, Verarbeitungsoptionen, Indizes und Exportformat), um sie in zukünftigen Projekten einfach wiederverwenden zu können. Vorlagen werden im Ordner „`Project Templates/`“ innerhalb Ihres Projekt-Speicherordners abgelegt und können auch über das Hauptmenü ausgewählt oder exportiert werden (*Vorlage auswählen* / *Vorlage speichern* / *Vorlage exportieren*).
* **Anwendungsbeispiele**:
  * Erstellen Sie Vorlagen für verschiedene Kamerasysteme (RGB, multispektral, NIR)
  * Speichern Sie Standardkonfigurationen für bestimmte Kulturpflanzenarten oder Analyse-Workflows
  * Gemeinsame Nutzung einheitlicher Einstellungen im Team
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
* **Standard (Linux)**: `~/Chloros Projects`
* **Beschreibung**: Zeigt das aktuelle Standardverzeichnis an, in dem neue Chloros-Projekte erstellt werden. Klicken Sie auf das Bearbeitungssymbol, um ein anderes Verzeichnis auszuwählen. Die Überschreibung wird als einzelne Textzeile in der Datei „`~/.chloros/working_directory.txt`“ gespeichert – bei „Windows“ ist dies „`C:\Users\<Username>\.chloros\working_directory.txt`“. Fehlt diese Datei oder einen Pfad angibt, der nicht mehr existiert, greift Chloros auf den oben genannten Standardwert zurück. Die Datei „CLI“ liest und schreibt dieselbe Datei, sodass sich „`chloros-cli`“ und die Benutzeroberfläche stets darüber einig sind, wo sich die Projekte befinden.
* **Projektvorlagen** befinden sich in einem Unterordner „`Project Templates/`“ dieses Verzeichnisses.
* **Wann ändern**:
  * Auf ein Netzlaufwerk einstellen für die Zusammenarbeit im Team
  * Wechseln Sie zu einem Laufwerk mit mehr Speicherplatz für große Datensätze
  * Organisieren Sie Projekte nach Jahr, Kunde oder Projekttyp in verschiedenen Ordnern
* **Hinweis**: Das Ändern dieser Einstellung wirkt sich nur auf NEUE Projekte aus. Bestehende Projekte verbleiben an ihren ursprünglichen Speicherorten.***

## Persistenz der Einstellungen

Ein Chloros-Projekt ist ein **Ordner**. Alle Projekteinstellungen werden darin unter `project.json` gespeichert; die angeschlossene Hardware wird zusammen mit dem Projekt in den Ordnern „`cameras.json`“ und „`sensors.json`“ gespeichert, sodass beim erneuten Öffnen eines Projekts auch dessen Kameras und Lichtsensoren wieder verbunden werden. Wenn Sie ein Projekt erneut öffnen, werden alle Einstellungen genau so wiederhergestellt, wie Sie sie hinterlassen haben. Gespeicherte Projekte können auch im Headless-Modus mit „`chloros-cli project`“ oder dem „`open_project`“ aus „SDK“ gesteuert werden.

### Einstellungshierarchie

Die Einstellungen werden in der folgenden Reihenfolge angewendet:

1. **Systemstandardwerte** – Integrierte Standardwerte, die von Chloros definiert werden
2. **Vorlageneinstellungen** – Wenn Sie beim Erstellen eines Projekts eine Vorlage laden
3. **Gespeicherte Projekteinstellungen** – Mit der Projektdatei gespeicherte Einstellungen
4. **Manuelle Anpassungen** – Alle Änderungen, die Sie während der aktuellen Sitzung vornehmen

### Einstellungen und Bildverarbeitung

Die Verarbeitungseinstellungen werden beim Start eines Verarbeitungslaufs geladen. Das Ändern einer Einstellung wirkt sich nicht rückwirkend auf bereits auf der Festplatte gespeicherte Ergebnisse aus – führen Sie die Verarbeitung erneut durch, um die neuen Einstellungen anzuwenden. Einige Einstellungen haben keinerlei Auswirkungen auf die Verarbeitung:

* Auflösung der Bildminiaturansichten (nur zur Anzeige)
* Projektvorlage speichern
* Projektordner speichern

***

## Referenz zu den Konfigurationsschlüsseln

Für die Automatisierung (CLI `--config`, SDK `configure`, oder beim direkten Auslesen von `project.json`) sind dies die genauen Schlüssel unter `Project Settings`:

| Schlüsselpfad | Typ | Standardwert |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | Zahl zwischen 0 und 10.000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | Zahl 0–100 | `60` |
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
| `Processing → Minimum recalibration interval` | Zahl 0–3600 | `0` |
| `Processing → Light sensor timezone offset` | Zahl -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | Cap-Profil-ID oder `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | Liste der Indexkonfigurationen | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Die Schlüssel `Array alignment` werden beim ersten Rendern des Abschnitts „Array-Ausrichtung“ geschrieben oder durch einen Automationsaufruf gesetzt werden. Solange sie fehlen, verwendet die Pipeline dieselben oben gezeigten Werte (`true`, `true`, bilinear). Daher verhält sich ein Projekt.json ohne diese Schlüssel genauso wie eines mit ihnen.

### In `project.json` gespeicherte Schlüssel ohne Steuerung im Einstellungsfenster

Diese befinden sich unter derselben `Project Settings`-Struktur und werden bei der Verarbeitung gelesen, aber in der Seitenleiste gibt es kein Widget dafür:

| Schlüsselpfad | Typ | Standardwert | Festgelegt durch |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Überschreibt die Interpretation der LATTICE-Eingabe-TIFFs; `auto` leitet dies aus dem XMP-Tag jeder Datei sowie der Kanalanzahl ab. Wird bei Survey3-Aufnahmen ignoriert. Absichtlich keine GUI-Einstellung – „auto“ ist in jedem normalen Fall korrekt. |
| `Processing → Target reflectance dir` | Pfadzeichenfolge | `""` | `chloros-cli process --target-reflectance-dir` oder das Projektziel API |
| `Processing → Target reflectance config` | nach Kameraseriennummer sortiertes Wörterbuch | `{}` | Registrierung eines Ziels innerhalb des Bildausschnitts (Modus `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | Pfadzeichenfolge | `""` | SDK `process_folder(daq_log_path=…)`. Verweist auf eine `.daq`-Aufzeichnung oder einen Ordner mit solchen Aufzeichnungen |
| `Target Detection → Minimum calibration target squares` | Zahl | `4` | Alte Standardeinstellung; keine Steuerung und kein CLI-Flag |
| `UI → Grid thumbnail size` | Nummer | `160` | Der eigene Miniaturansicht-Zoomregler des Bildrasters |

Zwei Viewer-Einstellungen werden **auf oberster Ebene in `project.json`** gespeichert, vollständig außerhalb von `Project Settings`, da es sich um Anzeigestatus und nicht um Verarbeitungseinstellungen handelt:

| Schlüsselpfad | Typ | Standardwert | Setzt durch |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | Ganzzahl 1–256 | `1` | Das GSD-Steuerelement (px) der Bildregisterkarte – siehe [Bild im Vollbildmodus öffnen](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Bewährte Vorgehensweisen

1. **Mit den Standardeinstellungen beginnen**: Die Standardeinstellungen eignen sich gut für die meisten MAPIR-Kamerasysteme und typische Arbeitsabläufe.
2. **Erstellen Sie Vorlagen**: Sobald Sie die Einstellungen für einen bestimmten Arbeitsablauf oder eine bestimmte Kamera optimiert haben, speichern Sie diese als Vorlage, um die Konsistenz über alle Projekte hinweg zu gewährleisten.
3. **Testen Sie vor der vollständigen Verarbeitung**: Wenn Sie mit neuen Einstellungen experimentieren, testen Sie diese zunächst an einer kleinen Auswahl von Bildern, bevor Sie Ihren gesamten Datensatz verarbeiten.
4. **Dokumentieren Sie Ihre Einstellungen**: Verwenden Sie aussagekräftige Vorlagennamen, die das Kamerasystem, die Verarbeitungsart und den Verwendungszweck angeben (z. B. „Survey3\_RGB\_NDVI\_Landwirtschaft“).
5. **Auswahl des Exportformats**: Wählen Sie Ihr Exportformat entsprechend Ihrer Endanwendung aus:
   * Wissenschaftliche Analyse → TIFF (16-Bit oder 32-Bit)
   * GIS-Verarbeitung → TIFF (16-Bit)
   * Schnelle Visualisierung → PNG (8-Bit)
   * Web-Freigabe → JPG (8-Bit)

***

Weitere Informationen zu multispektralen Indizes in Chloros finden Sie auf der Seite [Formeln für multispektrale Indizes](multispectral-index-formulas.md).
