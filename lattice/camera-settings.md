# Kameraeinstellungen

Die Registerkarte **Kameras** ist die Live-Bedienoberfläche vChloros

für LATTICE-Kameras: ein Hauptbildbereich, in dem jede angeschlossene Kamera als Live-Kachel angezeigt wird, sowie eine Seitenleiste, die zwischen drei Seiten wechselt – der **Kameraliste**, einem**Einstellungsbereich**(Einstellungen pro Kamera, Array oder Aufnahme – jeweils einzeln) sowie der**Index-Rechner**. Diese Seite dokumentiert alle Steuerelemente in der Kameraliste, im Einstellungsbereich pro Kamera und im Array-Einstellungsbereich. Aufnahmemodi, die Auswahl des Exporttyps sowie der „Capture All“-Ablauf werden auf der zugehörigen Seite [Aufnahmeeinstellungen &amp; -modi](capture.md) behandelt.

Die Registerkarte „Kameras“ erscheint in der Seitenleiste, sobald das Chloros

-Backend bereit ist. Alle unten aufgeführten Steuerelemente kommunizieren über `127.0.0.1:5000` mit dem lokalen Backend; Änderungen werden sofort auf die Live-Kamera angewendet, sofern nicht anders angegeben.

## Auf dieser Seite verwendete Kameratypen

Die Steuerelemente werden je nach ausgewählter Kamerart ein- oder ausgeblendet. Im Handbuch werden durchgehend folgende Begriffe verwendet:

| Begriff | Bedeutung | Filterkanäle |
| --- | --- | --- |
| **RGB

-Kamera** | LATTICE M3C mit FRGB-Filter (Modell enthält `-FRGB`) | Red

/ Green

/ Blue

|
| **Bayer-Multispektral** | LATTICE M3C mit FRGN, FOCN oder FNGB | FRGN: Red

/ Green

/ NIR

· FOCN: Orange

/ Cyan

/ NIR

· FNGB: NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M – ein Schmalbandfilter, ein kalibriertes Band | Einzelband |
| **Array-Mitglied** | Eine Kamera, die als Teil eines synchronisierten Arrays angeschlossen ist (kombinierte oder separate Anzeige) | Je nach Filter |

Bei RGB

-Kameras erfolgt eine photometrische Verarbeitung (Weißabgleich, Farbprofile, Gamma); bei Multispektral- und Mono-Kameras wird die radiometrische Kette angewendet und die photometrischen Einstellungen werden übersprungen. Array-Mitglieder übergeben Einstellungen auf Stream-Ebene (Pixelformat, Auflösung, Binning, Trigger, Bildrate) an das Array weiter – diese Zeilen werden im kameraspezifischen Bereich schreibgeschützt und werden stattdessen in den Array-Einstellungsbereich verschoben.

## Der Hauptbereich für den Live-Feed

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Wenn keine Kameras angeschlossen sind, zeigt der Live-Feed-Bereich einen Startbildschirm mit der Aufforderung **„Schließen Sie eine Kamera an, um zu beginnen“**und zwei Schaltflächen an:**Kamera verbinden**(grün, öffnet den Verbindungsdialog für eine einzelne Kamera) und**Array verbinden** (blau, öffnet den Verbindungsdialog für ein Array). Die Verbindungsdialoge selbst sind unter [Kameras verbinden](connecting.md) dokumentiert; Array-Konzepte (Synchronisation, Ebenen, Bandbreite) unter [Multi-Kamera-Arrays](arrays.md). Wenn Sie ein gespeichertes Projekt öffnen, das Kameras enthält, zeigt der Begrüßungsbildschirm stattdessen einen Ladekreis mit der Meldung „N gespeicherte Kameras werden erneut geöffnet…“, während Chloros

die Streams aus der letzten Sitzung wiederherstellt.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Obere Leiste

| Steuerung | Funktion |
| --- | --- |
| **Umschalten des Ansichtsmodus**| Wechselt zwischen der**Rasteransicht**(alle Kacheln als Zellen) und der**Listenansicht**(Zeilen in voller Breite: jedes kombinierte Array als eine zusammengesetzte Zeile, jedes Element eines**separaten** Arrays als eigene Zeile mit eigenem Feed und eigener Kalibrierungszieltabelle, darunter EINE eigenständige Kamera – die aktive). Tooltips: „Zur Rasteransicht wechseln“ / „Zur Listenansicht wechseln“. |
| **Rastersperre**(Vorhängeschloss) | Standardmäßig**gesperrt** – Kacheln sind an ihrer Position fixiert. Entsperren, um Kacheln per Ziehen in einen beliebigen Platz zu verschieben (Lücken bleiben erhalten). Das Raster wird automatisch wieder gesperrt, sobald eine neue Kamera verbunden wird. Tooltips: „Raster entsperren (Kacheln verschieben)“ / „Raster sperren (Kacheln fixieren)“. |
| **Feed-Zoom**-Schieberegler | Kachelgröße, von 60 px bis zur vollen Containerbreite. Die Zellen behalten ein Seitenverhältnis von 4:3 bei. Bei einer Zellenbreite unter 200 px werden die Überlagerungen für Name und fps ausgeblendet, um die Kachel übersichtlich zu halten. |

### Feed-Kacheln

Jede Kamera rendert eine zusammengesetzte Live-Kachel; eine Kamera kann zusätzlich drei graustufige **kanalweise aufgeteilte** Kacheln anzeigen (siehe [Kanalaufteilungen](#display-overlays-drawn-over-the-live-feed)), und Arrays rendern eine kombinierte Kachel. Die aktive Kachel ist mit einem Auswahlring in der Farbe der Kamera (oder des Arrays) versehen.

Beim Bewegen des Mauszeigers über eine Kachel wird eine **X**-Schaltfläche zum Schließen angezeigt:

* Das Schließen einer **zusammengesetzten** Kachel, während ihre Kanalaufteilungen noch angezeigt werden, blendet lediglich die zusammengesetzte Kachel aus.
* Das Schließen der **letzten sichtbaren Kachel einer Einzelkamera** trennt die Verbindung zu dieser Kamera.
* **Aufgeteilte Kacheln von Array-Mitgliedern trennen niemals** die Verbindung zur Kamera – sie werden lediglich ausgeblendet.

Wenn das Raster entsperrt ist, ziehen Sie eine beliebige Kachel auf einen beliebigen Platz; das Layout wird mit dem Projekt gespeichert.

## Seitenleiste – Kameraliste

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Auf der ersten Seite der Seitenleiste werden alle verbundenen Kameras und Arrays aufgelistet:

* **Kamera verbinden**(grün) /**Array verbinden** (blau, zeigt „Wird erkannt…“ während des Scannens an). Beide sind deaktiviert, solange ein Verbindungsdialog geöffnet ist.
* **Alles aufnehmen** (rot) – nimmt jede aufgelistete Kamera mit den in den Aufnahmeeinstellungen ausgewählten Exporttypen auf. Erfordert ein geöffnetes Projekt. Ausführlich dokumentiert unter [Aufnahmeeinstellungen &amp; Modi](capture.md).
* **Zahnrad „Aufnahmeeinstellungen“** (neben „Alles aufnehmen“) – öffnet den [Bereich „Aufnahmeeinstellungen“](capture.md#the-capture-settings-pane). Ist ohne Projekt oder während der Aufnahme deaktiviert.

### Kamerareihen

Jede Kamerareihe zeigt einen farbcodierten Rahmen (die benutzerdefinierte Farbe der Kamera), eine „CAM“-Beschriftung – mit einem blauen **M**(Master) oder einem grünen**S** (Slave) als Rollenbuchstaben für Array-Mitglieder – sowie den Anzeigenamen. Der Standardname lautet `LATTICE-MODEL (serial)`; Sie können ihn im Einstellungsbereich der jeweiligen Kamera umbenennen. Zeilenschaltflächen:

| Schaltfläche | Wirkung |
| --- | --- |
| **Auge**| Sichtbarkeit umschalten. Ausgeblendete Kameras werden aus dem Raster entfernt und**von „Capture All“ ausgeschlossen**. |
| **Zahnrad** | Öffnet den Einstellungsbereich für die einzelne Kamera (nächster Abschnitt). |
| **Pause / Wiedergabe**| Friert die Live-Vorschau**nur auf der Anzeigeseite** ein – die Backend-Aufzeichnung läuft weiter. Pausierte Kameras können nicht aufzeichnen. |
| **X** | Verbindung trennen. Die Benutzeroberfläche wird sofort aktualisiert (im besten Fall); die Trennung der Backend-Verbindung selbst kann 10–30 s dauern. |

### Array-Zeilen

Eine Array-Zeile zeigt ein „ARRAY“-Symbol in der Farbe des Arrays, den Array-Namen (in den Array-Einstellungen umbenennbar) und eine **DAQ · Ein/Aus**-Schaltfläche an –**Ein**, wenn der Lichtsensor auf Array-Ebene aktiviert ist *oder* ein beliebiges Mitglied über einen kameraspezifischen Sensor verfügt; in der Tooltip-Anzeige wird genau aufgelistet, welcher Sensor was liefert. Die Kameras der Elemente werden darunter eingerückt in eigenen Zeilen aufgelistet. Schaltflächen in Array-Zeilen: **Auge**(blendet ALLE Mitglieder gemeinsam ein/aus),**Zahnrad**(Einstellungsfenster des Arrays),**X**(das gesamte Array trennen).

Der Status des Lichtsensors (DLS), der in Array-Zeilen und im Einstellungsfenster des Arrays verwendet wird, hat vier Zustände:**aus**,**wartend**(noch kein Spektrum),**aktiv**(ein Spektrum ist innerhalb der letzten 3 s eingetroffen) und**veraltet** – seit 3 s kein aktuelles Spektrum, aber der letzte Messwert wird *weiterhin verwendet* (DAQ-Messwerte verfallen auf dem Erfassungspfad nie).

Solange Sie die Reihenfolge nicht ändern, zeigt die Liste zuerst Arrays und dann Einzelkameras an, wobei die Kameras (und die Mitglieder jedes Arrays) in **Kamerareihenfolge** angeordnet sind: M3C (Bayer) vor M3M (Mono); unter M3C, FRGB, FRGN, FOCN, dann FNGB; M3M nach Filterwellenlänge, beginnend mit der niedrigsten. Das Standardlayout des Rasters folgt derselben Reihenfolge. Du kannst Einzelkameras und ganze Array-Gruppen in der Seitenleiste aneinander vorbeiziehen, um die Liste neu zu ordnen; Array-Elemente lassen sich nicht unabhängig voneinander verschieben.

## Einstellungsbereich pro Kamera

Öffnen Sie ihn über das **Zahnrad** in einer Kamerazeile. Der Bereich wird über die Kameraliste geschoben.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Kopfzeile**: das**Farbfeld**der Kamera (zum Öffnen eines nativen Farbwählers anklicken – legt die Farbe des Seitenleistenrahmens und des Auswahlrings für die Kacheln fest), der**Name**mit einem Bleistift-Button zum**Umbenennen**(wird ein leerer Name gespeichert, wird der Standardwert `MODEL (serial)` wiederhergestellt) und**×** zum Schließen.

### Live-Histogramm

Oben im Fenster befindet sich ein Live-Luma-Histogramm, das mit einer Frequenz von ca. 8 Hz aus der JPEG

-Vorschau berechnet wird. Der Mittelwert ist Bayer-gewichtet – (R+2G+B)/4 –, um der kamerainternen Belichtungsmessung zu entsprechen.

* **Orange

r gestrichelter Strich**= das Belichtungsziel.**Ziehen Sie ihn horizontal, um das Ziel neu festzulegen** – beim Loslassen wird ein Befehl gesendet, und durch das Ziehen wird der Belichtungszielmodus auf „Manuell“ umgeschaltet.
* **Green

e durchgezogene Linie** = der tatsächliche Luma-Mittelwert (das, was die Belichtungssteuerung derzeit liefert).
* **RGB

-Schaltfläche** (oben rechts): Schaltet bandweise überlagerte Histogramme um, die entsprechend dem Filter der Kamera eingefärbt sind (z. B. bei FRGN: graues NIR

, grün, rot). Bei Mono-Kameras (M3M) trägt die Schaltfläche die Bezeichnung „MONO“ und ist deaktiviert – bei Mono wird immer das Ein-Band-Luma-Histogramm angezeigt.
* Die Beschriftungen der X-Achse richten sich nach der Bit-Tiefe des Sensors des aktuellen Pixelformats: 0..255, 0..1023, 0..4095 oder 0..65535.

### Zeilen

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

mit Kamerainformationen

| Zeile | Verhalten |
| --- | --- |
| **Modell** | Schreibgeschützt (z. B. `LATT-M3C-L87-FRGN`). |
| **Radiometrische Kalibrierung** | Green

es **„Aktiv“**-Symbol mit einer Beschriftung, die die Kalibrierungsstufe, den Hash-Wert, das Kalibrierungsdatum und die Bandliste anzeigt, geladen aus dem Kalibrierungspaket der Kamera (siehe [Werksradiometrische Kalibrierung](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Bei RGB

-Kameras ausgeblendet** – diese verfügen über eine photometrische Weißabgleichskalibrierung, nicht über eine bandweise Strahlungskalibrierung. |
| **Kalibrierungsbericht**|**Download**-Schaltfläche – öffnet das seriennummernspezifische NIST-Kalibrierungszertifikat der Kamera als PDF in Ihrem Betriebssystem-Viewer. Wenn das Zertifikat noch nicht zwischengespeichert ist, zeigt Chloros

stattdessen einen Hinweis an. |
| **Seriennummer** | Schreibgeschützt. |
| **Firmware**| Zeigt die aktuelle Version an und ermittelt anschließend die für dieses Modell verfügbare Version (pro Modell zwischengespeichert – ein Array aus N Kameras fragt den Server einmal ab). Status: „Wird geprüft…“ → Schaltfläche**„Auf X aktualisieren“**→ „Wird geflasht…“ → „Aktualisiert von A auf B“ / „Fehlgeschlagen: …“ / „Übersprungen: …“ / grün**„Auf dem neuesten Stand“**. Tooltip der Update-Schaltfläche: „Werkseinstellung + Flashen + Neuprogrammierung von UserSet1. ~2–3 Minuten; Verbindung nicht trennen.“ |
| **IP** | Schreibgeschützt. |
| **Temperatur** | Schreibgeschützt, wird alle 3 s aktualisiert. Leuchtet orange bei ≥65 °C und rot mit einem ⚠ bei ≥75 °C. |
| **Kalibrierungsziel** | Aktiviert die ArUco-Reflexionszielerkennung mit einer panelbezogenen NDVI

-Validierungstabelle unterhalb der Live-Ansicht (Listenansicht) – siehe [Die Kalibrierungsziel-Tabelle](#the-calibration-target-table). Nur für die aktuelle Sitzung – wird immer deaktiviert geöffnet. |
| **Lichtsensor**-Dropdown | Bindet einen DAQ-Lichtsensor (DAQ-E/M/U, aus der Liste auf der Registerkarte „Lichtsensoren“) an diese Kamera für die Beleuchtungskorrektur bei einfallendem Licht (DLS) und die prädiktive automatische Belichtung. „„Keine“ löscht die Zuordnung. Sind keine Sensoren angeschlossen, zeigt das Dropdown-Menü „(keine Sensoren angeschlossen – DAQ-Registerkarte öffnen)“ an. Die Zuordnung wird mit dem Projekt gespeichert. |

#### Die Tabelle „Kalibrierungsziel“

Die Tabelle dient **ausschließlich zur Validierungsanzeige** („nur zur Validierung – ändert die Kalibrierung nicht“). In der Kopfzeile sind die Seriennummer der Kamera, der Filter, die Zielnummer und die Markergröße in Pixeln angegeben, und es gibt eine Zeile pro Panel:

* **Panel**, gefolgt von einer Spalte pro Band mit dem Wert des Panels – eine Bayer-Kamera zeigt ihre drei Bänder nach Name und Wellenlänge an (z. B. FRGN: Red

660 nm / Green

550 nm / NIR

850 nm); eine **Monochrom-Kamera (M3M) zeigt ihr einziges Band** an (z. B. `F750`) und hat keine NDVI

-Spalten.
* **Sat** – der Anteil der gesättigten Pixel des Panels (bernsteinfarben ab 1 %, rot ab 5 %).
* **NDVI

Meas**,**NDVI

Theo** (der Referenz-Reflexions-NDVI

des Panels) und **Δ**– nur bei Bayer-Multispektralkameras. Ein kleines Zeichen hinter „NDVI

Meas“ gibt an, woraus der gemessene NDVI

berechnet wurde:**ρ**(Reflexion – ein Lichtsensorwert ist verfügbar),**L**(Strahlungsintensität – kein Lichtsensorwert) oder**DN**(Rohwerte der Kamera). Nur ein**ρ**-NDVI

ist mit dem NDVI

Theo vergleichbar: Bei L und DN ist die Spalte „Δ“ ausgegraut und ein Hinweis weist darauf hin („Kein Lichtsensorwert: NDVI

basiert auf Radiance, daher ist Δ keine Kalibrierungsprüfung“ / „NDVI

basiert auf Rohwerten der Kamera, daher ist Δ keine Kalibrierungsprüfung“). Bei ρ ist Δ unter 0,02 grün, unter 0,05 gelb und ansonsten rot.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Belichtung &amp; Verstärkung

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Alle numerischen Eingaben hier verwenden Spinner, die durch Gedrückthalten beschleunigt werden: Tippen = ±1, länger als 1,5 s gedrückt halten = ±10, länger als 3 s gedrückt halten = ±100. Der Wert wird an die Kamera gesendet, sobald Sie loslassen.

| Steuerung | Bereich / Auswahlmöglichkeiten | Standardwert | Gilt für | Funktion |
| --- | --- | --- | --- | --- |
| **Belichtung (us)**| Min./Max. der Kamera im Live-Bild | Auto | Alle | Belichtungszeit in Mikrosekunden, mit einem**Auto/Manuell**Umschalter. „Auto“ = kontinuierliche kameraseitige Belichtungsautomatik. Bei Einstellung einer manuellen Belichtung bleibt**Verstärkung** auf „Auto“, sofern zuvor so eingestellt, sodass der gesamte Verstärkungsbereich der Kamera genutzt werden kann, bis Sie die Verstärkung bearbeiten oder auf „Manuell“ umstellen. |
| **Verstärkung (dB)**| Live-Min./Max.-Werte der Kamera (z. B. bis zu 48 dB) | Manuell (aus) | Alle | Analoge/digitale Verstärkung mit eigenem**Auto/Manuell**-Schalter. |
| **AE-Zielhelligkeit**| 0–255 | 80, Modus**Auto**| Alle (bearbeitbar, wenn AE oder automatische Verstärkung aktiviert sind) | Die Helligkeit, auf die die Belichtungsautomatik abzielt. Im**Auto**(Standardeinstellung) wählt ein histogrammbasierter Backend-Regler den Zielwert selbst aus und hält die Belichtung dabei bei 60–75 % des Sensormaximums. Durch Eingabe eines Werts oder Ziehen der orangefarbenen Linie im Histogramm wird auf**Manuell** umgeschaltet. |
| **AE-Glättung** | 0,5–40, Schrittweite 0,1 | 8,0 | Alle | AE-Dämpfung. Tooltip: „Niedriger = AE reagiert schneller (kann bei hohen Bildraten pulsieren). Höher = gleichmäßiger / langsamer.“ Werte, die weit unter dem Standardwert liegen, können bei hohen Bildraten zu einem Pulsieren der AE führen und das Streaming destabilisieren; 8,0 ist der stabile Standardwert. |
| **AE-Interessensbereich**| Kontrollkästchen „Aktivieren“ + Schaltfläche**Zielen**| Aus | Alle | Wenn aktiviert, misst AE nur den grün gestrichelten Bereich statt des gesamten Bildes.**„Aim“** aktiviert das Platzieren per Klick im Live-Feed: Ein Klick zentriert einen Bereich bei 30 % des Bildes; Klicken und Ziehen zeichnet ein benutzerdefiniertes Rechteck (mindestens 5 % × 5 %). „Aim“ deaktiviert sich nach einer Platzierung automatisch. Der Bereich wird unter Berücksichtigung aller von Ihnen festgelegten Drehungen/Spiegelungen auf die kamerainigenen Koordinaten zurückgemappt und zusammen mit dem Projekt gespeichert. |
| **AE-Anpassungsgeschwindigkeit** | 0,1–5, Schrittweite 0,1 | 1,0 | Nur für Array-Mitglieder | Wie schnell das automatische AE-Ziel Änderungen der Szenenhelligkeit nachverfolgt; bei 1,0× erfolgt eine erneute Überprüfung alle 2,5 s. |
| **Highlight-Schutz** | Streng (1 %) / Normal (5 %) / Locker (15 %) | Streng | Kameras, die diese Einstellung bereitstellen | Wie viel des Bildausschnitts darf in Weiß ausbleichen, bevor die Belichtungsautomatik das Bild abdunkelt. |

{% hint style="info" %}
**Lichtanforderungen für Bayer-Multispektralkameras (RGN

/ OCN

/ NGB

):** Die Szene muss in allen drei Kanälen über ausreichend Licht verfügen, sonst funktioniert die Kalibrierung nicht korrekt — eine einzige Sensorbelichtung deckt alle drei Spektren ab. Verwenden Sie einen DAQ-Lichtsensor, um die Lichtverhältnisse zu messen, oder wechseln Sie in den All-Mono-Modus (M3M), damit jedes Band seine eigene Belichtung erhält. Wenn eine Aufnahme diese Vorgabe nicht erfüllt, erkennt Chloros

dies und warnt Sie (die „Unmix-Clamp“-Benachrichtigung). Im Live-Modus zeigt eine Kanal-Split-Kachel **„Banddaten mit geringer Zuverlässigkeit · N %“** an, wenn der Unmix dieses Band bei 25 % oder mehr seiner Pixel geklemmt hat.
{% endhint %}

{% hint style="info" %}
**Wie die automatische Belichtung Belichtungszeit und Verstärkung einsetzt (1.3.0).** Bei automatischer Verstärkung hält Chloros

die Verstärkung bei 0 dB, bis die Belichtungszeit den maximal zulässigen Wert erreicht hat (die AE-Obergrenze, die Triggerperiode eines Arrays oder die DAQ-Belichtungsobergrenze); erst dann steigt die Verstärkung an. In einem Array hat die Triggerrate Vorrang: Die automatische Belichtung füllt die durch die Rate vorgegebene Periode aus, und erhöht dann den Verstärkungsfaktor, sodass eine dunkler werdende Szene die automatische Triggerrate nicht mehr verringert (in Version 1.2.0 war dies noch der Fall). Eine **manuelle**Belichtung an einem Array-Mitglied, die länger als die Triggerperiode dauert, verlangsamt den Trigger des Arrays entsprechend; die automatische Triggerrate wird wiederhergestellt, sobald jedes Element, das sie verlangsamt hat, wieder auf automatische Belichtung umgeschaltet ist. Wenn sowohl**Belichtung**als auch**AE-Zielhelligkeit** auf „Auto“ stehen und ein DAQ-Lichtsensor einen Messwert liefert, wird die Belichtung jeder Kamera beibehalten, sobald sich die automatische Belichtung eingependelt hat, solange das gemessene Licht innerhalb von etwa ±25 % bleibt; Eine größere, anhaltende Lichtänderung, ein Neustart der Einstellungen (z. B. eine Änderung des ROI) oder eine von Ihnen vorgenommene Belichtungsänderung löst die Belichtung erneut aus.
{% endhint %}

{% hint style="info" %}
**Verhalten der automatischen Belichtung (1.3.0).**

* Bei einer manuellen Belichtung bleibt die Verstärkung auf „Auto“, bis Sie die Verstärkung ändern.
* Kleine Glanzlichter verdunkeln die Szene nicht: Eine Spiegelung von bis zu 1 % des Bildausschnitts wird ignoriert, und bei den Feldern eines bekannten Kalibrierungsziels bis zu 0,5 % eines Feldes.
* Die DAQ-Belichtungsobergrenze gilt nur, solange ein Kalibrierungsziel im Blickfeld ist; ist kein Feld zu sehen, belichtet die Kamera entsprechend der Szene.
* Die Belichtung bleibt bei konstantem Licht stabil und pendelt sich nach einer starken Änderung innerhalb weniger Sekunden ein. Ein Clip begrenzt die Belichtung nicht mehr dauerhaft, und bei einer Änderung des ROI wird die Belichtung wiederhergestellt.
* Der Helligkeitswert überschreitet nicht den Bereich der KameraReichweite, sodass ein wiederkehrendes Licht keine Überbelichtung im Bild verursacht.
* Eine Kamera, deren automatische Belichtung bei starker Sonneneinstrahlung nicht mehr reagiert, wird auf dem Niveau der aktuellen Lichtverhältnisse gehalten, das vom Lichtsensor oder – falls kein Lichtsensor angeschlossen ist – anhand der eigenen Bilder der Kamera ermittelt wird, und bei einer Lichtänderung wieder losgelassen. Ein verdunkelter oder abgedeckter Lichtsensor stört dies nicht.
{% endhint %}

### Pixelformat &amp; Auflösung

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Array-Elemente** zeigen die schreibgeschützten Zeilen „Aktuell“ (Format + BxH) und „Binning“ mit dem Hinweis „In den Array-Einstellungen festgelegt“ an – ein Neustart des Streams bei einem Element würde die Synchronisation unterbrechen, daher werden diese im [Array-Einstellungsfenster](#array-settings-pane) verwaltet.**Einzelkameras** verfügen über:

| Steuerung | Auswahlmöglichkeiten | Funktion |
| --- | --- | --- |
| **Pixelformat** | 8-Bit / 10-Bit / 12-Bit / 16-Bit | Bittiefe des Sensors; die Formatfamilie entspricht der des Sensors. Die Standardeinstellung ist 12-Bit, und ein Projekt speichert die von Ihnen festgelegte Einstellung – einschließlich 8-Bit – und stellt die Verbindung entsprechend wieder her. |
| **Auflösung** | Voll / Halb / Viertel | Relativ zum aktuellen Binning: Voll = 2048/N × 1536/N bei N×N-Binning. |
| **Binning** | 1×1 (keines) / 2×2 / 4×4 | Hardware-N×N-Binning – größere Werte verringern die Auflösung, erhöhen jedoch das Signal-Rausch-Verhältnis (SNR) und die Bildrate. Eine Änderung führt zu einem Neustart des Streams und setzt alle ROIs auf das neue vollständige Sichtfeld zurück. |
| **Aktuell** | schreibgeschützt | Die tatsächlich geltenden Werte für WxH und (x, y)-Versatz. |

### Live-Vorschau

Alle Einstellungen in diesem Abschnitt wirken sich **nur auf die Anzeige**aus – sie verändern das, was Sie im Live-Feed sehen, während gespeicherte Aufnahmen linear und unverändert bleiben – mit einer Ausnahme:**Vignette** ist radiometrisch und wirkt sich auch auf Exporte aus (siehe unten).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Steuerung | Bereich / Auswahlmöglichkeiten | Standard | Gilt für | Funktion |
| --- | --- | --- | --- | --- |
| **Render-Auflösung** | 360p (am schnellsten) / 480p / 720p / 1080p / Native Sensorauflösung (langsamste) | 720p | Alle | Die Höhe, in der das Backend die radiometrische Vorschau-Kette ausführt. Eine niedrigere Einstellung erhöht die Bildrate, ohne das Sichtfeld zu verändern. |
| **Index**| Kontrollkästchen „Aktivieren“ + Zahnrad | Aus | Nur Bayer-Multispektral,**nicht** Mitglieder von Combined-Array | Live-Vorschau des Vegetationsindex. Das Zahnrad öffnet den gemeinsamen [Index-Rechner](#index-calculator-pane), der mit den natürlichen Filterbändern der Kamera vorbelegt ist (z. B. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Der benutzerdefinierte Ausdruck sowie die LUT (ein/aus, Standardstufe 7 Blendenstufen, Standardmin. 0,2, Standardmax. 1) wird bei jedem Vorschaubild berechnet. Mitglieder des kombinierten Arrays blenden diese Zeile aus – das Array verfügt über einen gemeinsamen Index. |
| **Weißabgleich** | Aus / Einmal / Kontinuierlich + Schaltfläche zur Neuerfassung | Kontinuierlich | Nur RGB

| Live-Weißabgleich. Die Schaltfläche „Aktualisieren“ erfasst den Weißabgleich erneut anhand des aktuellen DLS-Spektrums (deaktiviert, wenn der Modus auf „Aus“ steht). |
| **Gamma** | Ein / Aus | Ein | Nur RGB

| Anzeige des Gammawerts (γ = 2,2 LUT) in der Live-Vorschau. Gespeicherte Aufnahmen bleiben linear. |
| **Rauschunterdrückung** | Kontrollkästchen + Stärke 0–100 | Aus / 50 | Alle (pro Kamera, auch innerhalb von Arrays) | Bilateraler Filter in der Live-Vorschau. Höhere Werte = glatteres, aber weicheres Detailbild. |
| **Schärfe** | Kontrollkästchen + Stärke 0–100 | Aus / 30 | Alle | Unscharfmaskierung in der Live-Vorschau, wird zuletzt angewendet. Kann Rauschen verstärken. Nur in der Vorschau. |
| **Vignettierung**| Kontrollkästchen + Stärke 0–100 | Aus / 0 | Alle | Manuelle Restentfernungs-Vignettierung (hellt Ecken auf), die über die „Smart Vignette“-Schätzung des Arrays gelegt wird.**Radiometrisch – wirkt sich auf die Live-Ansicht UND Exporte aus**, im Gegensatz zu Rauschunterdrückung/Schärfe. |
| **Farbprofil** | Raw / Linear / Natürlich / Verbessert / Benutzerdefinierte Temperatur | Natürlich | Nur RGB

| Siehe unten. |
| **Farbtemperatur** | 2000–10000 K, Schritt 100 | 5500 K | Nur RGB

, benutzerdefiniertes Temperaturprofil | Legt den Weißabgleich auf eine feste korrelierte Farbtemperatur fest (DLS-Eingabe wird ignoriert). Der zuletzt gewählte Kelvin-Wert wird bei Profilwechseln beibehalten. |
| **Lichter** | −100 bis 0, Schrittweite 1 | 0 (aus) | Nur RGB

| Neu in 1.3.0. Verdunkelt die hellsten, weißesten Bereiche des Bildes (0 = unverändert, −100 = am stärksten). Gilt für die Live-Vorschau **sowie für Aufnahmen auf Anzeigeebene (Vorschau)**, die die Einstellung speichern (`Chloros:DisplayHighlights`); Strahlungs- und Reflexionswerte werden niemals beeinflusst. Es können keine abgeschnittenen Sensordaten wiederhergestellt werden – verringern Sie hierfür die Belichtung. Die Zebra-Einblendung passt sich dem abgedunkelten Bild an. Deaktiviert mit dem Raw-Farbprofil („Wird beim RAW-Farbprofil nicht angewendet.“). |
| **Sättigung** | 0–200 (100 = neutral) | 100 | Nur RGB

| HSV-Sättigung in der Live-Vorschau. |
| **Kontrast** | 0–200 (100 = neutral) | 100 | Nur RGB

| Linearer Kontrast um Mittelgrau in der Live-Vorschau. |
| **Horizontal spiegeln / Vertikal spiegeln** | Kontrollkästchen | Aus | Alle | Die Vorschau horizontal/vertikal spiegeln. |
| **Drehung**| 0° / 90° / 180° / 270° | 0° | Alle | Die Vorschau drehen. Die Ausrichtung wird am Ende der Backend-Vorschau-Kette angewendet.**Gespeicherte Aufnahmen (1.3.0):**Die Produkte „Vorschau“, „Radiance“, „Reflexion“ und „Index“ einer eigenständigen Kamera werden durch Spiegeln/Drehen gedreht, während**„Raw“ und „Debayered“ in der kameraneigenen Ausrichtung**; jede Aufnahme-Datei speichert die Einstellung in ihrem XMP, sodass bei der späteren Verarbeitung einer RAW-Datei diese auf dieselbe Weise angewendet wird.**Dateien von Array-Elementen bleiben in ihrer nativen Ausrichtung** (die Einstellung wird gespeichert, aber nicht angewendet), und zusammengesetzte Ansichten des Arrays ignorieren sie. |**Semantik der Farbprofile** (RGB

-Kameras):

* **Raw** — die Verarbeitungskette wird vollständig umgangen.
* **Linear** — Dunkelsignal + Flat-Field + Weißabgleich; keine Farbmatrix, kein Gamma.
* **Natürlich** *(Standard)* — linear plus die gemessene Farbkorrekturmatrix und eine an die Szene angepasste Tonwertkurve.
* **Verbessert**— „Natürlich“ plus Lebendigkeit und lokaler CLAHE-Kontrast. Der zusätzliche Aufwand gilt**nur für die Live-Vorschau** — gespeicherte Aufnahmen erhalten unabhängig vom Profil immer die vollständige Nachbearbeitung.
* ****Benutzerdefinierte Temperatur** — „Natürlich“ mit einem auf den von Ihnen gewählten Kelvin-Wert festgelegten Weißabgleich.

{% hint style="warning" %}
Bei „Natürlich“, „Verbessert“ und „Benutzerdefinierte Temperatur“ zeigt der Bereich einen Hinweis zur Tonwertkurve an: Die Bilder werden entsprechend ihrer jeweiligen Szene aufgehellt, sodass gespeicherte *Anzeigebilder* nicht Bild für Bild vergleichbar sind. **Exportieren Sie die Strahlungsintensität oder Reflektanz für Messungen.**
{% endhint %}

### Anzeige-Overlays (über den Live-Feed gelegt)

Diese sind nur im Frontend verfügbar – sie werden über das Video gelegt, ohne den Stream oder die Aufnahmen zu beeinflussen.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Overlay | Steuerelemente | Standard | Funktion |
| --- | --- | --- | --- |
| **Zebra** | Kontrollkästchen + Schwellenwert 200–255 | Aus / 250 | Magenta-Diagonalstreifen auf abgeschnittenen Pixeln. |
| **Fadenkreuz** | Kontrollkästchen | Aus | Markierung der Bildmitte. |
| **Raster** | Aus / 3 × 3 / 9 × 9 | Aus | Kompositionsraster. |
| **Histogramm** | Kontrollkästchen + Breite 0,10–0,90 des Bildes | Aus / 0,25 | Ein Histogrammstreifen im Live-Feed. |
| **Fokus-Peak** | Kontrollkästchen + Schwellenwert 20–200 + Farbfeld | Aus / 80 / `#ff5722` | Sobel-Kantenhervorhebung zur Fokussierung. |
| **Kanalaufteilung** | „Aufteilung anzeigen (Red

/ Green

/ NIR

)“ / Schaltfläche „Aufteilungen ausblenden“ | Ausgeblendet | Fügt neben dem zusammengesetzten Bild drei unabhängige Graustufen-Kacheln pro Kanal hinzu (die Beschriftung der Schaltfläche richtet sich nach den Filterkanälen der Kamera). Jede Kachel lässt sich verschieben und hat die Rahmenfarbe der Kamera. Nicht verfügbar bei Monochrom-Kameras. Wird mit dem Projekt gespeichert. |

### Spot-Belichtungsmesser

* Kontrollkästchen **„Zum Abtasten klicken“**: Klicken Sie auf das Live-Bild, um ein einzelnes Pixel abzutasten (ein Fadenkreuz markiert es), oder klicken und ziehen Sie einen Bereich, um einen Pixeldurchschnitt zu ermitteln.**„Löschen“**entfernt die Abtastung und das Fadenkreuz. Schließt den**„Aim“**-Modus von AE-ROI aus.
* **Dropdown-Menü**Anzeigen**:**Raw (Bittiefe)**– native digitale Werte in der Bittiefe des Sensors (z. B. 12 Bit → 0..4095) – oder**Anzeige (8 Bit)** (Standard). Wenn ein Live-Index aktiv ist, zeigt „Display“ stattdessen den berechneten Indexwert (z. B. NDVI

) an.
* Das Auslesefenster listet Pixelkoordinaten, Bildgröße, Pixelformat, Bittiefe und eine Kanaltabelle (Chan / Wert / %) mit Bandbezeichnungen und Wellenlängen auf; Bayer-Grünpaare werden gemittelt; Bereichswerte zeigen „N px avg“ an.

Der Status des Spotmessers gilt nur für die aktuelle Sitzung.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Vorausschauende automatische Belichtung (DLS-gesteuert)

Dieser Abschnitt wird nur angezeigt, wenn **mindestens ein DAQ-Lichtsensor angeschlossen ist** – der Solver benötigt ein Live-Spektrum der einfallenden Strahlung, um ihn zu steuern.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Steuerung | Bereich | Standard | Funktion |
| --- | --- | --- | --- |
| **Aktivieren** | Kontrollkästchen | Ein (Standalone-Kameras) | Ein Solver mit geschlossener Formel nutzt das DLS-Spektrum sowie die Skalare des Kalibrierungspakets der Kamera, um das hellste Band nahe der Sättigung zu platzieren und gleichzeitig das dunkelste Band über dem SNR-Mindestwert zu halten – ein einziger Belichtungsschreibvorgang pro Berechnung, keine Einstellschleife. Konzipiert für solarbetriebene Zeitrafferaufnahmen, bei denen jede Aufnahme korrekt belichtet sein muss. Das Backend weicht stillschweigend auf reaktive Belichtungssteuerung (AE) zurück, sobald der DLS-Messwert veraltet ist oder fehlt oder das Kalibrierungspaket nicht geladen ist. |
| **Glättung (α)** | 0,05–1,0, Schrittweite 0,05 | 0,3 | Glättung aufeinanderfolgender prädiktiver Lösungen (niedrigerer Wert = glatter). |
| **Szenenreflexionsgrad**| Schaltfläche**ρ neu kalibrieren** | — | Schätzt den vom Solver verwendeten Szenenreflexionsgrad neu. |

{% hint style="info" %}
**„Array connect“ deaktiviert standardmäßig die prädiktive Belichtungsautomatik** — bei Array-Konfigurationen übernehmen „Chloros

‘ Smart AE“ sowie die kameraseitige Belichtungsautomatik die Belichtung (mit Sättigungsschutz), und die Schätzung der Reflexion einer einzelnen Szene durch die prädiktive BelichtungsautomatikReflexionsschätzung der prädiktiven Belichtungsautomatik ist bei gemischten Szenen nicht zuverlässig. Sie können sie hier pro Kamera wieder aktivieren, wenn Sie ausdrücklich eine DLS-gesteuerte radiometrische Belichtung wünschen.
{% endhint %}

**DAQ-gesteuerte Belichtungsobergrenze und auf den einfallenden Lichtstrahl fixierte Belichtungsautomatik.** Unabhängig vom obigen Kontrollkästchen berechnet Chloros

– wenn ein DAQ-Lichtsensor einer RGB

-Kamera zugewiesen ist – anhand der gemessenen absoluten nach unten gerichteten Bestrahlungsstärke die maximale Belichtung × Verstärkung, bei der eine Oberfläche mit 100 % Reflexionsgrad unterhalb der Clipping-Grenze bleibt, und wendet diese als **Obergrenze**für die automatische Belichtung an. Seit Version 1.3.0 gilt die Obergrenze nur, solange sich ein Kalibrierungsziel im Sichtfeld dieser Kamera befindet (innerhalb der letzten 10 Sekunden im Bildausschnitt erfasst); ist kein Ziel im Blickfeld, greift stattdessen die normale automatische Belichtung der Kamera mit ihrem eigenen Schutz vor Überbelichtung, sodass der Rest der Szene nicht dunkler gehalten wird, als es notwendig ist. Während die Obergrenze aktiv ist, arbeitet die Kamera**auffalllichtgesteuert**: Sie arbeitet im offenen Regelkreis bei der auf das einfallende Licht gemessenen Belichtung mit einer Verstärkung von 0 dB – die Belichtung richtet sich nach dem gemessenen Licht, nicht nach dem Bildinhalt. Da der Oberwert die Belichtungszeit nur verkürzen kann, kann er selbst kein Clipping verursachen. Seit Version 1.3.0 wird die Obergrenze anhand der kameraspezifischen Belichtungskonstante aus dem Kalibrierungspaket berechnet (ein Flottendurchschnittswert nur dann, wenn das Paket keinen eigenen Wert enthält). Die Obergrenze wird automatisch deaktiviert – und die normale Szenen-AE wird wieder aufgenommen –, sobald kein Kalibrierungsziel im Blickfeld ist, der DAQ-Messwert fehlt, veraltet (&gt;30 s) oder dunkel ist oder wenn ≥15 % des Bildes bei der festgelegten Belichtung übersteuert werden (was bedeutet, dass Sensor und Kamera unterschiedliche Beleuchtungsstärken wahrnehmen). Es gibt keinen Schalter in der Benutzeroberfläche; dies ist das Standardverhalten, sobald eine RGB

-Kamera mit einem DAQ verbunden ist.

### Erfassung &amp; Trigger

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Array-Mitglieder zeigen zusätzlich die schreibgeschützten Zeilen **Rolle**(Master in Blau / Slave in Grün),**Synchronisationsleitung**und**Peers** an.

| Steuerung | Auswahlmöglichkeiten | Standard | Anmerkungen |
| --- | --- | --- | --- |
| **Triggermodus** | Aus / Ein | Ein | Für Array-Elemente deaktiviert (das Array verwaltet die Triggerung). |
| **Triggerquelle** | Software / Leitung 0 (M8) / Leitung 1 / Leitung 2 | Leitung 0 | Ausgeblendet, solange der Triggermodus auf „Aus“ steht; für Array-Mitglieder deaktiviert. Leitung 0 ist der optoisolierte externe Triggereingang M8. |
| **Bildrate**| Auto / Manuell + Wert | Auto |**Auto**: Die Bildratenbegrenzung der Kamera ist deaktiviert – die Belichtungszeit bestimmt die fps, und das Feld zeigt die aktuelle Live-Bildrate an.**Manuell**: Sie begrenzen die fps mit einem Schieberegler (1 bis zum durch die Bandbreite begrenzten Maximum), ausgehend von der aktuellen tatsächlichen Bildrate. Array-Mitglieder sehen einen schreibgeschützten Wert „N fps (live)“ mit dem Hinweis „In Array-Einstellungen festgelegt“. |

### Netzwerk / Transport

| Zeile | Verhalten |
| --- | --- |
| **Paketgröße**| 1500 (Standard) / 9000 (Jumbo) – Standard ist**Jumbo**. |
| **Durchsatz** | Schreibgeschützte Begrenzung des Link-Durchsatzes in MB/s. Das Backend verteilt diesen bei jeder Verbindung/Trennung neu auf alle angeschlossenen Kameras. |
| **Pufferverwaltung** | Schreibgeschützter Modus für die Pufferverwaltung. |

### Aufnahme

Der Bereich endet mit einer Schaltfläche **„Aufnahmeeinstellungen öffnen…“**, die zum [Bereich „Aufnahmeeinstellungen“](capture.md#the-capture-settings-pane) führt (deaktiviert, bis ein Projekt geöffnet ist – „Erstellen oder öffnen Sie ein Projekt, um Aufnahmen zu speichern“). Wenn die Kamera ausgeblendet oder angehalten ist, erinnert Sie ein Hinweis daran, sie vor der Aufnahme wieder einzublenden bzw. fortzusetzen.

## Bereich „Array-Einstellungen“

Öffnen Sie diesen Bereich über das **Zahnrad**in einer ARRAY-Zeile. Kopfzeile: Array-Name mit einem Bleistift-Symbol zum Umbenennen und**×** zum Schließen. Die unten mit *nur kombiniert* gekennzeichneten Abschnitte erscheinen nur bei Arrays, die im kombinierten Anzeigemodus verbunden sind.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synchronisation

Schreibgeschützte Zeilen für **Master**,**Slaves**und**Sync-Leitung**.

### Umgebungslichtsensor

Wird sowohl für kombinierte als auch für separate Arrays angezeigt:

* Dropdown-Menü **Lichtsensor** – bindet einen DAQ an das gesamte Array. Die Auswahl wird sofort übernommen, überträgt sich auf das jeweilige „Lichtsensor“-Dropdown-Menü jeder einzelnen Kamera (Sie können die Einstellung weiterhin pro Kamera überschreiben) und beginnt mit der Weiterleitung von Spektren an das Array.
* Live-Zeile **Status**: Aus · „Warten auf erstes Spektrum…“ · „Aktiv – alle Kameras im Array sind beleuchtungskorrigiert“ · „Kein neues Spektrum in den letzten 3 s – es wird weiterhin der letzte Messwert verwendet (kein Timeout für veraltete Daten)…“.
* Hinweis im Fenster: „Array-weite radiometrische Korrektur. Kamera-spezifische Einstellungen haben Vorrang.“

### Kalibrierungsziel

Eine eigene Gruppe direkt nach dem Umgebungslichtsensor, sowohl für kombinierte als auch für separate Arrays:

* Kontrollkästchen **Aktivieren** — „MAPIR

-ArUco-Ziel erkennen und NDVI

-LUT gegenüber der Panel-Reflexions-LUT validieren“; steuert die Zielüberlagerung und die Validierungstabellen der kombinierten Kachel. Bei einem **separaten** Array aktiviert es die eigene [Kalibrierungsziel-Tabelle](#the-calibration-target-table) (eine pro Kamerareihe in der Listenansicht).

### Aufnahme – einheitliche Sensoreinstellungen

Wird sowohl für kombinierte als auch für separate Arrays angezeigt (die Gruppenüberschrift lautet „Aufnahme (kameragleich)“). Diese Einstellungen gelten einheitlich für jedes Element (Änderungen pro Element würden die Synchronisation unterbrechen). Bearbeitungen werden zwischengespeichert und gemeinsam mit **Anwenden**übernommen – mit Ausnahme des Schalters**ROI ein / ROI aus**, der sofort wirksam wird.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Steuerung | Auswahlmöglichkeiten / Bereich | Funktion |
| --- | --- | --- |
| **Pixelformat** | 8-Bit / 10-Bit / 12-Bit / 16-Bit | Einheitliche Bittiefe für alle Mitglieder; jede Kamera übernimmt das Format dieser Bittiefe für ihren eigenen Sensor. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardware-Binning – bewahrt das volle Sichtfeld bei gleichzeitiger Steigerung des Signal-Rausch-Verhältnisses und der Bildrate. Eine Änderung setzt die ROI-Felder auf das neue gesamte Sichtfeld zurück. |
| **Auflösung** (Voreinstellung) | Voll / Halb / Viertel | Relativ zum Binning; füllt die ROI-Felder mit einem zentrierten Ausschnitt. |
| **ROI-Ausschnitt (px)**| Umschalter**ROI ein / ROI aus**, Schaltfläche**Auswählen**, Felder für Breite / Höhe / X / Y, Vollsensor-Karte | Sensorausschnitt – siehe [Der ROI-Editor](#the-roi-editor). Breite/Höhe rasten auf Vielfache von 16 (mindestens 64); Offsets rasten an Vielfachen von 4 ein. Ein Hinweis „max WxH“ zeigt den Höchstwert an, und**Zurücksetzen** setzt die Felder auf das volle Sichtfeld zurück. Während der Bearbeitung verfolgt der orangefarbene Ausschnittrahmen jede Feldänderung auf der Karte und auf der Array-Kachel. |
| **Triggerrate**| Umschalter „Auto“ / „Manuell“ + fps 0,5–10, Schrittweite 0,5 |**Auto**(Standard): Das Backend leitet die Auslösefrequenz aus Auflösung und Bandbreite ab – die Eingabe ist deaktiviert und zeigt den abgeleiteten Wert an.**Manuell**: Legt Ihren Wert beim Klicken auf „Übernehmen“ fest. Eine festgelegte Frequenz, die eine neue Bildgröße nicht unterstützen kann, wird auf die schnellste Frequenz begrenzt, die diese Größe zulässt, und kehrt zurück, sobald die Bildgröße dies wieder zulässt. |

Hinweis im Fenster: „Änderungen an Format/Auflösung führen zu einem kurzen Neustart aller Kameras. Die Triggerrate wird live angewendet.“ Die Schaltflächen **Übernehmen / Abbrechen** befinden sich am unteren Rand des Fensters.

#### Der ROI-Editor

Neu in 1.3.0. Die ROI-Ausschnittzeile und die darunter liegende Karte arbeiten zusammen:

* **ROI ein / ROI aus**– schaltet den Sensorausschnitt**sofort**ein oder aus (die Kameras werden kurz neu gestartet; die Array-Kachel zeigt „Einstellungen werden angewendet…“ an).**Aus**liest den gesamten Sensor aus und**merkt sich Ihren Ausschnitt**– die Felder zeigen ihn weiterhin an;**Ein** wendet ihn erneut an. Wenn Sie den ROI ohne festgelegten Ausschnitt aktivieren, wird ein zentrierter Ausschnitt in halber Größe angewendet („Es wurde kein Ausschnitt festgelegt, daher wurde ein zentrierter Ausschnitt in halber Größe angewendet. Passen Sie ihn über die Felder an.“).
* **Auswählen**— zeichnet den Ausschnitt auf der Karte: Ziehen Sie den Mauszeiger darüber oder klicken Sie auf zwei gegenüberliegende Ecken (Esc bricht den Vorgang ab). Das Rechteck wird in den Feldern W/H/X/Y angepasst an die oben genannten Schritte; es wird nichts angewendet, bis Sie auf**Übernehmen** klicken, sodass Sie es zunächst mit den Feldpfeilen feinjustieren können. Ein Rechteck pro Klick auf „Auswählen“.
* **Die Karte**zeigt den Ausschnitt auf dem gesamten Sensor an, sodass Sie ihn wieder erweitern können: „Vollsensor, live“, während die Kameras den gesamten Sensor auslesen, oder „Vollbild-Schnappschuss von {Zeitpunkt}. Die Kameras lesen nur den Ausschnitt“, während sie beschnitten sind. Wenn das Array bisher nur beschnittene Daten gestreamt hat, lautet die Anzeige „Noch keine Vollbildansicht: Dieses Array hat bisher nur beschnittene Daten gestreamt.“ mit einer Schaltfläche**Vollbild anzeigen**, die den ROI deaktiviert. Die Bezeichnung des Ausschnitts „`W×H @ X,Y`“ befindet sich außerhalb des Rahmens (darüber oder darunter, wenn der Rahmen den oberen Rand berührt), sowohl auf der Karte als auch auf der Live-Kachel.
* **Anwenden**übernimmt die Felder: Ein Ausschnitt aktiviert den ROI; der Vollbildmodus (nach**Zurücksetzen**) deaktiviert ihn, wobei der vorherige Ausschnitt weiterhin gespeichert bleibt; ein deaktivierter ROI, dessen Felder noch den gespeicherten Ausschnitt anzeigen, bleibt deaktiviert, wenn Sie eine weitere Änderung (Format, Binning, Bildrate) anwenden.
* **Bei einem ausgerichteten Array wird das Ausschnittfenster jedes Mitglieds durch die Ausrichtung festgelegt**, sodass jede Kamera denselben Ausschnitt der Szene erfasst. Ein Array, das vor der Ausrichtung beschnitten wurde, wird zunächst auf den gesamten Sensor ausgerichtet (der Ausschnitt wird aufgehoben, die Ausrichtung berechnet und der Ausschnitt mit kameraspezifischen Fenstern wiederhergestellt).
* Nach einer Änderung des Ausschnitts wird die automatische Belichtung an den neuen Bildausschnitt angepasst. Innerhalb eines Ausschnitts kann die Belichtung eines Mitglieds weiterhin bis zum Ende der Auslöseperiode laufen.
* Der Ein-/Aus-Status des ROI und der gespeicherte Ausschnitt werden mit dem Projekt gespeichert.

### Ausrichtung (Co-Registrierung)

Wird für **sowohl kombinierte als auch separate Arrays** angezeigt (in Version 1.2.0 nur für kombinierte Arrays); die Gruppenüberschrift lautet „Ausrichtung“.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Restwert**-Anzeige: „RMS x,xx px“ – grün unter 1 px, gelb unter 3 px, rot in allen anderen Fällen oder wenn eine Kamera versagt hat; „kein Profil“ vor der ersten Berechnung.
* Zusammenfassungszeile: „✓ Alle Kameras ausgerichtet (N)“ / „⚠ p/N Kameras ausgerichtet —  <serial (filter)="">fehlgeschlagen“ / „Zuschneiden aktiv — Zur Ausrichtung neu kalibrieren (nutzt den gesamten Sensor)“ / „Warten auf das Stabilisieren der Belichtung…“ / „Einstellungen werden angewendet…“ (die Kameras werden neu gestartet) / „Array liefert keine Bilder mehr – Array erneut verbinden“.
* Tabelle pro Kamera: Kamera (die letzten 4 Ziffern der Seriennummer + Filter), Reprojektionsfehler in px mit Übereinstimmungsanzahl („ref“ für die Master-Kamera) und der auf die 0,35-Pass-Schwelle normierte Kreuzkorrelationswert der Überlappung.
* **Ausrichtung neu kalibrieren** Schaltfläche (zeigt vor dem ersten Profil „Ausrichtung kalibrieren“ an) – führt die Co-Registrierung mit neuen Bildern erneut durch.
* **Kontrollkästchen „Kameras für die Ausrichtung automatisch belichten“** (standardmäßig aktiviert) – hellt dunkle oder flache Kameras vorübergehend auf (zuerst Belichtung, dann Verstärkung), damit sie eine Textur zum Abgleichen aufweisen, und stellt anschließend die automatische Belichtung wieder her.

Chloros richtet jedes Array nach dem Verbinden automatisch aus, und zwar in beiden Anzeigemodi, sobald sich die automatische Belichtung der einzelnen Elemente stabilisiert hat (es wird bis zu 5 Minuten lang versucht); die kombinierte Vorschau wird ebenfalls beim Öffnen ausgerichtet. Ein separates Array verfügt über keine zusammengesetzte Kachel, daher fragt dieser Abschnitt den Ausrichtungsstatus ab und führt „Neukalibrieren“ selbst aus. Neukalibrieren, wenn sich Fokus oder Szenentiefe geändert haben. Die Ausrichtung ist **konzeptionell auf die jeweilige Sitzung beschränkt** – sie wird niemals in einem Profil gespeichert, da sie von der aktuellen Szenentiefe abhängt. Aufnahmen können weiterhin pixelregistriert exportiert werden (siehe [Ausgerichtete Exporte](capture.md#per-array-controls)).

### Intelligente Vignettierung

* Kontrollkästchen **„Korrektur aktivieren“**– wendet die kameraspezifische Vignettierungsschätzung auf die radiometrische Kette an (Live-Anzeige**und** Exporte).
* ****Aus aktueller Ansicht kalibrieren**– richten Sie das Array zunächst auf ein einheitliches Ziel (Flachbildschirm, Wand oder Himmel); jede Kamera wird einzeln abgeflacht, und der Status meldet „n/N Kameras · −x,x %“ Abflachungsgewinn. Mit**Löschen** wird die Schätzung entfernt.
* Status: „Kalibriert“, „Nicht kalibriert“ oder „Teilweise kalibriert – erneut ausführen“, wenn nur die Schätzungen einiger Kameras bei ihrem aktuellen Bildausschnitt noch gelten (beispielsweise nachdem eine „Neukalibrierung“ das Ausschnittfenster einer Kamera verschoben hat) – führen Sie die Kalibrierung erneut durch.
* Führen Sie eine Feinabstimmung pro Kamera mit dem kameraspezifischen **Vignette**-Schieberegler in der [Live-Vorschau](#live-preview) durch.

### Live-Vorschau

* Kontrollkästchen **Histogramm**+ Schieberegler für die Breite (0,10–0,90 des Bildausschnitts) – der Histogrammstreifen wird auf allen Kacheln aller Mitglieder gleichzeitig angezeigt (einschließlich der Kacheln mit getrennten Kanälen), das [Histogramm-Überlagerung](#display-overlays-drawn-over-the-live-feed), die für das gesamte Array festgelegt ist. Sie wird als aktiviert angezeigt, wenn jedes Mitglied sie bereits hat. Bei einem**separaten** Array ist dies die einzige Zeile in der Gruppe.
* *Nur kombiniert:* **Index**: Kontrollkästchen aktivieren + Zahnrad – öffnet den gemeinsamen [Index-Rechner](#index-calculator-pane) mit Bändern, die aus**allen** Kameras der Mitglieder erstellt wurden. Eine Zeile zur Ausdrucksvorschau darunter zeigt den aktuellen Ausdruck an („Kein Ausdruck festgelegt – öffnen Sie den Rechner, um einen zu erstellen“), der jede Sekunde aktualisiert wird.
* *Nur kombiniert:* Dropdown-Menü **Render-Auflösung**(gleiche Voreinstellungen wie pro Kamera, Standard 720p): die Höhe des Live-View-Streams**und** die gespeicherte Größe des Composite-Exports. Hinweis im Fenster: „Vorschau + gespeicherte Composite-Größe. Bilder pro Kamera werden immer in voller Auflösung exportiert.“

### Anzeigeebenen *(nur kombiniert)** Kontrollkästchen **Aktivieren** (standardmäßig deaktiviert – die Master-Kamera wird direkt angezeigt; aktiviert = überlagerte Komposition).
* **Dropdown-Menüs**Vordergrund**/**Hintergrund**: jede Mitgliedskamera (nach Name) oder**Index**. Wenn „Vordergrund“ auf „Index“ gesetzt ist, zeigen Pixel außerhalb des LUT-Min/Max-Bereichs die Hintergrundebene an.

### Geteilte Ansicht *(nur kombiniert)*

**„Mitgliedskameras anzeigen“**– eine Schaltfläche**„Mitglieds-Kameras aufteilen / ausblenden“**, die den Live-Feed jeder einzelnen Mitglieds-Kamera als separate Rasterkacheln neben der Zusammensetzung hinzufügt. Die Kacheln lesen den vorhandenen Bildspeicher des Arrays aus (keine zusätzliche Kameraverbindung). Nur in der Rasteransicht; wird pro Array mit dem Projekt gespeichert.

### Funktionen

Ein schreibgeschütztes Fenster, das alle 5 s aktualisiert wird:

* **Stufenbezeichnung**: „Simultane Aufnahme“ (grün) · „Simultane Aufnahme (FTD-versetzte Emission)“ (grün) · „Versetzte Aufnahme (100 ms Drift)“ (gelb) · „Konfiguration zu groß“ (rot).
* **Frame-Zustand**: „x,xx % unvollständig“ – grün unter 1 %, gelb unter 5 %, rot ab 5 %.
* ****Verbindungszeile**: „NIC {mbps} Mbit/s – nachhaltig {MB/s} MB/s“.

Dies ist das Live-Bandbreitenbudget des Arrays. Informationen zum zugrunde liegenden fps- und Netzwerkmodell – sowie dazu, was zu ändern ist, wenn die Stufe gelb oder rot wird – finden Sie unter [Multi-Kamera-Arrays](arrays.md) und in der [CLI-Referenz](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Bereich „Index-Rechner“

Die dritte Seite in der Seitenleiste, die sowohl vom kameraspezifischen Index-Zahnrad als auch vom kombinierten Array-Index-Zahnrad genutzt wird (jeweils nur eines – die Überschrift lautet „Index-Rechner – <camera name="">“ oder „Index-Rechner – <array name="">“). Hier werden die Bandliste (die natürlichen Filterbänder der Kameranatürliche Bänder der Kamera oder alle Bänder aller Mitglieder des Arrays), den aktuellen Ausdruck und die LUT-Konfiguration (ein/aus, Stufe – Standard 7 Blendenstufen, min – Standard 0,2, max – Standard 1) sowie ein Live-Index-Histogramm. Mit **Anwenden** wird der Ausdruck übernommen; LUT-Änderungen werden live auf die Vorschau angewendet.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Kamera-spezifische vs. Array-verwaltete Einstellungen

Kurzanleitung dazu, was sich wo befindet, wenn eine Kamera Teil eines Arrays ist:

| Array-verwaltet (im Kamera-Fenster schreibgeschützt) | Nach wie vor kamera-spezifisch innerhalb eines Arrays |
| --- | --- |
| Pixelformat, Auflösung, Binning | Automatische Belichtung (Belichtung, Verstärkung, Zielwert, Glättung, ROI) |
| Auslösemodus/-quelle, Bildrate | Rauschunterdrückung, Schärfe, Vignettierung |
| | Ausrichtung (Spiegelung/Drehung), Anzeige-Overlays (Array-Einstellungen → „Live-Vorschau“ legt das Histogramm für alle Mitglieder gleichzeitig fest), Spotmessung |
| | Index (Arrays mit separater Anzeige), Zuordnung von Lichtsensoren |

Sonstiges übergreifendes Verhalten:

* **Kombinierte vs. separate Anzeige** wird beim Verbinden des Arrays ausgewählt: kombiniert = eine ausgerichtete zusammengesetzte Kachel (Mitglieder-Feeds nur über „Split View“); separat = jedes Mitglied rendert seine eigene synchronisierte Kachel. Eine Kamera zeigt niemals sowohl einen eigenständigen Feed als auch eine Array-Kachel an.
* ****Automatische Wiederverbindung**: Beim Öffnen eines gespeicherten Projekts werden dessen Kameras und Arrays wiederhergestellt und alle gespeicherten Einstellungen erneut auf das Backend angewendet, bevor die Streams fortgesetzt werden.
* **Aufnahme-Gating**: Ausgeblendete oder pausierte Kameras werden von „Capture All“ ausgeschlossen; ein Array wird nur dann vollständig blockiert, wenn ALLE Mitglieder ausgeblendet/pausiert sind. Siehe [Aufnahmeeinstellungen und -modi](capture.md).

## Wie Einstellungen beibehalten werden

Der Status der Registerkarte „Kamera“ wird **mit dem Projekt** gespeichert, nicht im Browser:

* Bei jeder reaktiven Änderung werden die Kameras und Arrays im Projekt unter `cameras.json` (mit einer Entprellzeit von 500 ms) gespeichert. Dies umfasst Kameranamen und -farben, Belichtungs-/Verstärkungs-/AE-Einstellungen, Pixelformat/Auflösung/Binning, Triggerrate, Vorschau-Einstellungen (Render-Auflösung, Rauschunterdrückung, Schärfe, Vignettierung, Farbprofil, Lichter, Sättigung/Kontrast), Ausrichtung, Überlagerungen, Kanalaufteilungen, Indexkonfiguration, Einstellungen für prädiktive Belichtungsautomatik, AE-ROI, Array-Namen, Anzeigemodus, Array-Aufnahmeeinstellungen (einschließlich des ROI-Ausschnitts, ob der ROI aktiviert ist und des gespeicherten Ausschnitts, wenn er deaktiviert ist), das „Wire Budget“ des Arrays sowie den Rasterblock (Feed-Zoom, Ansichtsmodus, Rastersperre, manuelle Kachelreihenfolge, ausgeblendete Kameras, geschlossene Kacheln, aktive Kamera).
* Lichtsensor-Zuordnungen werden in der Projektdatei „`sensors.json`“ gespeichert.
* Beim erneuten Öffnen des Projekts wird die Hardware wieder verbunden und alle Einstellungen erneut angewendet.
* **Kein Projekt geöffnet = nur sitzungsbezogen**: Ohne Projekt gehen beim Schließen von Chloros alle Einstellungen verloren.
* Nur sitzungsbezogen, unabhängig vom Projekt: Pausenzustand, Spotmess-Proben, das Kontrollkästchen „Kalibrierungsziel“ pro Kamera (ist standardmäßig deaktiviert) und das Array-Ausrichtungsprofil (wird konzeptionell pro Sitzung neu berechnet).
* Eine Ausnahme: Die Exportauswahlen unter **Aufnahmeeinstellungen** und der Aufnahmemodus bleiben pro Projekt im lokalen App-Speicher erhalten und nicht in `cameras.json` – siehe [Aufnahmeeinstellungen &amp; Modi](capture.md).</array></camera></serial>
