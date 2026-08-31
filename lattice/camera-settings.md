# Kameraeinstellungen

Die Registerkarte **Kameras**ist die Live-Bedienoberfläche von Chloros für LATTICE-Kameras: Ein Hauptbereich, in dem jede angeschlossene Kamera als Live-Kachel angezeigt wird, sowie eine Seitenleiste, die zwischen drei Seiten wechselt – der**Kameraliste**, ein**Einstellungsbereich**(Einstellungen pro Kamera, für das Array oder für die Aufnahme – jeweils einzeln) sowie der**Index-Rechner**. Diese Seite dokumentiert alle Bedienelemente in der Kameraliste, im Einstellungsbereich pro Kamera und im Einstellungsbereich für das Array. Die Auswahl der Aufnahmemodi, der Export-Typen-Typauswahl sowie der „Capture All“-Ablauf werden auf der zugehörigen Seite [Aufnahmeeinstellungen &amp; -modi](capture.md) behandelt.

Die Registerkarte „Kameras“ wird in der Seitenleiste angezeigt, sobald das Chloros-Backend bereit ist. Alle unten aufgeführten Steuerelemente kommunizieren über `127.0.0.1:5000` mit dem lokalen Backend; Änderungen werden sofort auf die Live-Kamera angewendet, sofern nicht anders angegeben.

## Auf dieser Seite verwendete Kameratypen

Die Steuerelemente werden je nach ausgewählter Kamerart ein- oder ausgeblendet. Im Handbuch werden durchgehend folgende Begriffe verwendet:

| Begriff | Bedeutung | Filterkanäle |
| --- | --- | --- |
| **RGB-Kamera** | LATTICE M3C mit FRGB-Filter (Modell enthält `-FRGB`) | Red / Green / Blue |
| **Bayer-Multispektral** | LATTICE M3C mit FRGN, FOCN oder FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M – ein Schmalbandfilter, ein kalibriertes Band | Einzelband |
| **Array-Element** | Eine Kamera, die als Teil eines synchronisierten Arrays angeschlossen ist (kombinierte oder separate Anzeige) | Je nach Filter |

RGB-Kameras durchlaufen die photometrische Verarbeitung (Weißabgleich, Farbprofile, Gamma); Multispektral- und Mono-Kameras durchlaufen die radiometrische Kette und überspringen die photometrischen Einstellungen. Array-Mitglieder übergeben Einstellungen auf Stream-Ebene (Pixelformat, Auflösung, Binning, Trigger, Bildrate) an das Array weiter – diese Zeilen werden im kameraspezifischen Bereich schreibgeschützt und werden stattdessen in den Array-Einstellungsbereich verschoben.

## Der Hauptbereich

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

für den Live-Feed Wenn keine Kameras angeschlossen sind, zeigt der Live-Feed-Bereich einen Begrüßungsbildschirm mit der Aufschrift **„Schließen Sie eine Kamera an, um zu beginnen“**und zwei Schaltflächen an:**Kamera verbinden**(grün, öffnet den Verbindungsdialog für eine einzelne Kamera) und**Array verbinden** (blau, öffnet den Verbindungsdialog für ein Array). Die Verbindungsdialoge selbst sind unter [Kameras verbinden](connecting.md) dokumentiert; die Konzepte für Array-Konfigurationen (Synchronisation, Ebenen, Bandbreite) in [Multi-Kamera-Arrays](arrays.md). Wenn Sie ein gespeichertes Projekt mit Kameras öffnen, zeigt der Begrüßungsbildschirm stattdessen einen Ladekreis mit der Meldung „N gespeicherte Kameras werden erneut geöffnet…“ während Chloros die Streams aus der letzten Sitzung wiederherstellt.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Obere Leiste

| Steuerung | Funktion |
| --- | --- |
| **Umschalten zwischen Ansichtsmodi**| Wechselt zwischen**Rasteransicht**(alle Kacheln als Zellen) und**Listenansicht** (Arrays in voller Breite oben, EINE aktive Kamera darunter). Tooltips: „Zur Rasteransicht wechseln“ / „Zur Listenansicht wechseln“. |
| **Rastersperre**(Vorhängeschloss) | Standardmäßig**gesperrt** – Kacheln sind an ihrer Position fixiert. Entsperren, um Kacheln per Drag &amp; Drop in einen beliebigen Platz zu verschieben (Lücken bleiben erhalten). Das Raster wird automatisch wieder gesperrt, sobald sich eine neue Kamera verbindet. Tooltips: „Raster entsperren (Kacheln verschieben)“ / „Raster sperren (Kacheln fixieren)“. |
| **Feed-Zoom**-Schieberegler | Kachelgröße, von 60 px bis zur vollen Containerbreite. Die Kacheln behalten ein Seitenverhältnis von 4:3 bei. Bei einer Kachelbreite unter 200 px werden die Überlagerungen für Name und fps ausgeblendet, um die Kachel übersichtlich zu halten. |

### Feed-Kacheln

Jede Kamera rendert eine zusammengesetzte Live-Kachel; eine Kamera kann zusätzlich drei graustufige **kanalweise aufgeteilte** Kacheln anzeigen (siehe [Kanalaufteilungen](#display-overlays-drawn-over-the-live-feed)), und Arrays rendern eine kombinierte Kachel. Die aktive Kachel ist mit einem Auswahlring in der Farbe der Kamera (oder des Arrays) versehen.

Wenn man mit der Maus über eine Kachel fährt, wird eine **X**-Schaltfläche zum Schließen angezeigt:

* Das Schließen einer **zusammengesetzten** Kachel, während ihre Kanalaufteilungen noch angezeigt werden, blendet lediglich die zusammengesetzte Kachel aus.
* Das Schließen der **zuletzt sichtbaren Kachel einer eigenständigen Kamera** trennt diese Kamera.
* **Aufgeteilte Kacheln von Array-Mitgliedern trennen die Kamera niemals** – sie werden lediglich ausgeblendet.

Wenn das Raster entsperrt ist, ziehen Sie eine beliebige Kachel auf einen beliebigen Platz; das Layout wird mit dem Projekt gespeichert.

## Seitenleiste – Kameraliste

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Auf der ersten Seite der Seitenleiste werden alle angeschlossenen Kameras und Arrays aufgelistet:

* **Kamera verbinden**(grün) /**Array verbinden** (blau, zeigt „Erkennung läuft…“ während des Scannens an). Beide Optionen sind deaktiviert, solange ein Verbindungsdialog geöffnet ist.
* **Alles aufnehmen** (rot) – nimmt alle aufgelisteten Kameras mit den in den Aufnahmeeinstellungen gewählten Exporttypen auf. Erfordert ein geöffnetes Projekt. Vollständig dokumentiert unter [Aufnahmeeinstellungen &amp; -modi](capture.md) ausführlich dokumentiert.
* **Zahnrad „Aufnahmeeinstellungen“** (neben „Alles aufnehmen“) — öffnet den [Bereich „Aufnahmeeinstellungen“](capture.md#the-capture-settings-pane). Deaktiviert, wenn kein Projekt vorhanden ist oder während der Aufnahme.

### Kamera-Zeilen

Jede Kamerazelle zeigt einen farbcodierten Rand (die benutzerdefinierte Farbe der Kamera), die Bezeichnung „CAM“ – mit einem blauen **M**(Master) oder grünen**S** (Slave) als Rollenbuchstaben für Array-Mitglieder – sowie den Anzeigenamen. Der Standardname lautet `LATTICE-MODEL (serial)`; benennen Sie ihn im Einstellungsfenster der jeweiligen Kamera um. Zeilenschaltflächen:

| Schaltfläche | Wirkung |
| --- | --- |
| **Auge**| Sichtbarkeit umschalten. Ausgeblendete Kameras werden aus dem Raster entfernt und**von „Capture All“ ausgeschlossen**. |
| **Zahnrad** | Öffnet den Einstellungsbereich für die jeweilige Kamera (nächster Abschnitt). |
| **Pause / Wiedergabe**| Friert die Live-Vorschau**nur auf der Anzeigeseite** ein – die Backend-Aufzeichnung läuft weiter. Angehaltene Kameras können keine Aufnahmen machen. |
| **X** | Verbindung trennen. Die Benutzeroberfläche wird sofort aktualisiert (optimistisch); die Trennung der Verbindung im Backend kann 10–30 s dauern. |

### Array-Zeilen

Eine Array-Zeile zeigt ein „ARRAY“-Symbol in der Farbe des Arrays, den Array-Namen (in den Array-Einstellungen umbenennbar) und eine **DAQ · Ein/Aus**-Schaltfläche an –**Ein**, wenn der Lichtsensor auf Array-Ebene aktiviert ist *oder* ein beliebiges Mitglied über einen kameraspezifischen Sensor verfügt; in dessen Tooltip genau aufgeführt ist, welcher Sensor welche Daten liefert. Die zugehörigen Kameras werden darunter eingerückt in eigenen Zeilen aufgelistet. Schaltflächen in Array-Zeilen: **Auge**(blendet ALLE Mitglieder gemeinsam ein/aus),**Zahnrad**(Fenster „Array-Einstellungen“),**X**(die Verbindung zum gesamten Array trennen).

Der Status des Lichtsensors (DLS), der in den Array-Zeilen und im Array-Einstellungsbereich verwendet wird, hat vier Zustände:**aus**,**wartet**(noch kein Spektrum),**aktiv**(ein Spektrum ist innerhalb der letzten 3 s eingetroffen) und**veraltet** – seit 3 Sekunden kein aktuelles Spektrum, aber der letzte Messwert wird *weiterhin verwendet* (DAQ-Messwerte verfallen im Erfassungspfad niemals).

Du kannst einzelne Kameras und ganze Array-Gruppen in der Seitenleiste aneinander vorbeiziehen lassen, um die Liste neu zu ordnen; Array-Elemente lassen sich nicht unabhängig voneinander verschieben.

## Einstellungsbereich pro Kamera

Öffnen Sie diesen über das **Zahnrad** in einer Kamerazeile. Der Bereich wird über die Kameraliste geschoben.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Kopfzeile**: das**Farbfeld**der Kamera (per Klick öffnet sich ein nativer Farbwähler – legt die Farbe des Seitenleistenrandes und des Kachelauswahlrings fest), der**Name**mit einem Bleistift-Symbol zum**Umbenennen**(beim Speichern eines leeren Namens wird auf den Standardnamen „`MODEL (serial)`“ zurückgesetzt) sowie**×** zum Schließen.

### Live-Histogramm

Oben im Fenster befindet sich ein Live-Luma-Histogramm, das mit einer Frequenz von ca. 8 Hz aus der JPEG-Vorschau berechnet wird. Der Mittelwert ist Bayer-gewichtet – (R+2G+B)/4 –, um der kamerainternen Belichtungsmessung zu entsprechen.

* **Orange gestrichelte Linie**= das AE-Ziel.**Ziehen Sie sie horizontal, um das Ziel neu festzulegen** – beim Loslassen wird ein Befehl gesendet, und durch das Ziehen wird der AE-Zielmodus auf „Manuell“ umgeschaltet.
* **Green durchgezogene Linie** = der tatsächliche Mittelwert der Luminanz (das, was die Belichtungsautomatik derzeit liefert).
* **RGB-Schaltfläche** (oben rechts): Schaltet bandweise überlagerte Histogramme um, die entsprechend dem Filter der Kamera eingefärbt sind (z. B. bei FRGN: grau NIR, grün, rot). Bei Mono-Kameras (M3M) trägt die Schaltfläche die Bezeichnung „MONO“ und ist deaktiviert – im Mono-Modus wird immer das Ein-Band-Luma-Histogramm angezeigt.
* Die Beschriftungen der X-Achse richten sich nach der Bit-Tiefe des Sensors des aktuellen Pixelformats: 0..255, 0..1023, 0..4095 oder 0..65535.

### Zeilen

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

mit Kamerainformationen | Zeile | Verhalten |
| --- | --- |
| **Modell** | Schreibgeschützt (z. B. `LATT-M3C-L87-FRGN`). |
| **Radiometrische Kalibrierung**| Green**„Aktiv“**-Symbol mit einer Beschriftung, die die Kalibrierungsstufe, den Hash-Wert, das Kalibrierungsdatum und die Bandliste anzeigt, geladen aus dem Kalibrierungspaket der Kamera (siehe [Werksseitige radiometrische Kalibrierung](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Bei Kameras vom Typ RGB ausgeblendet** – diese verfügen über eine photometrische Weißabgleichskalibrierung, nicht über eine bandweise Strahlungskalibrierung. |
| **Kalibrierungsbericht**|**Download**-Schaltfläche – öffnet das seriennummernspezifische NIST-Kalibrierungszertifikat der Kamera im PDF-Format in Ihrem Betriebssystem-Viewer. Wenn das Zertifikat noch nicht im Cache gespeichert ist, zeigt Chloros stattdessen einen Hinweis an. |
| **Seriennummer** | Schreibgeschützt. |
| **Firmware**| Zeigt die aktuelle Version an und ermittelt anschließend die für dieses Modell verfügbare Version (pro Modell zwischengespeichert – ein Array aus N Kameras fragt den Server einmal ab). Zustände: „Wird geprüft…“ → Schaltfläche**„Auf X aktualisieren“**→ „Wird geflasht…“ → „Aktualisiert von A → B“ / „Fehlgeschlagen: …“ / „Übersprungen: …“ / grün**„Auf dem neuesten Stand“**. Tooltip der Update-Schaltfläche: „Werkseinstellung + Flashen + Neuprogrammierung von UserSet1. ~2–3 Minuten; Verbindung nicht trennen.“ |
| **IP** | Schreibgeschützt. |
| **Temperatur** | Schreibgeschützt, wird alle 3 s aktualisiert. Leuchtet orange bei ≥65 °C und rot mit einem ⚠ bei ≥75 °C. |
| Kontrollkästchen **Kalibrierungsziel** | Aktiviert die Erkennung von ArUco-Reflexionszielen mithilfe einer panelbezogenen NDVI-Validierungstabelle unterhalb der Live-Ansicht (Listenansicht). Nur für die aktuelle Sitzung – ist immer deaktiviert. |
| **Lichtsensor**-Dropdown-Menü | Verknüpft einen DAQ-Lichtsensor (DAQ-E/M/U, aus der Liste auf der Registerkarte „Lichtsensoren“) mit dieser Kamera zur Korrektur der Beleuchtung durch herabfallendes Licht (DLS) und zur prädiktiven automatischen Belichtung. „Keine“ hebt die Verknüpfung auf. Wenn keine Sensoren angeschlossen sind, zeigt das Dropdown-Menü „(keine Sensoren angeschlossen – DAQ-Registerkarte öffnen)“. Die Zuordnung wird mit dem Projekt gespeichert. |

### Belichtung &amp; Verstärkung

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Alle numerischen Eingaben hier verwenden Spinner mit „Halten zum Beschleunigen“: Tippen = ±1, länger als 1,5 s halten = ±10, länger als 3 s halten = ±100. Der Wert wird an die Kamera gesendet, sobald Sie loslassen.

| Steuerung | Bereich / Auswahlmöglichkeiten | Standardwert | Gilt für | Funktion |
| --- | --- | --- | --- | --- |
| **Belichtung (us)**| Live-Min./Max.-Werte der Kamera | Auto | Alle | Belichtungszeit in Mikrosekunden, mit einem Umschalter für**Auto/Manuell**. Auto = kontinuierliche kameraseitige Belichtungsautomatik. |
| **Verstärkung (dB)**| Live-Min/Max der Kamera (z. B. bis zu 48 dB) | Manuell (aus) | Alle | Analoge/digitale Verstärkung mit eigenem**Auto/Manuell**-Schalter. |
| **AE-Zielhelligkeit**| 0–255 | 80, Modus**Auto**| Alle (bearbeitbar, wenn AE oder automatische Verstärkung aktiviert sind) | Die Helligkeit, auf die die Belichtungsautomatik abzielt. Im Modus**Auto**(Standardeinstellung) wählt ein histogrammbasierter Backend-Regler den Zielwert selbst aus und hält die Belichtung dabei bei 60–75 % des Sensormaximums. Durch Eingabe eines Werts oder Ziehen der orangefarbenen Linie im Histogramm wird der Modus auf**Manuell** umgeschaltet. |
| **AE-Glättung** | 0,5–40, Schrittweite 0,1 | 8,0 | Alle | AE-Dämpfung. Tooltip: „Niedriger = AE reagiert schneller (kann bei hohen Bildraten pulsieren). Höher = gleichmäßiger / langsamer.“ Werte, die weit unter dem Standardwert liegen, können bei hohen Bildraten zu Aussetzern der Belichtungsautomatik führen und das Streaming destabilisieren; 8,0 ist der stabile Standardwert. |
| **Belichtungsautomatik-Zielbereich**| Kontrollkästchen „Aktivieren“ + Schaltfläche**Aim**| Aus | Alle | Wenn aktiviert, misst die Belichtungsautomatik nur den grün gestrichelten Bereich statt des gesamten Bildes.**Zielen** aktiviert das Platzieren per Klick im Live-Feed: Ein Klick zentriert einen Bereich bei 30 % des Bildes; Klicken und Ziehen zeichnet ein benutzerdefiniertes Rechteck (mindestens 5 % × 5 %). „Aim“ deaktiviert sich nach einer Platzierung automatisch. Der Bereich wird entsprechend der von Ihnen festgelegten Drehung/Spiegelung auf die kameraneigenen Koordinaten abgebildet und mit dem Projekt gespeichert. |
| **AE-Anpassungsgeschwindigkeit** | 0,1–5, Schrittweite 0,1 | 1,0 | Nur für Array-Mitglieder | Wie schnell das automatische AE-Ziel Änderungen der Szenenhelligkeit nachverfolgt; bei 1,0× erfolgt eine erneute Überprüfung alle 2,5 s. |
| **Highlight-Schutz** | Streng (1 %) / Normal (5 %) / Locker (15 %) | Streng | Kameras, die diese Einstellung unterstützen | Wie viel des Bildausschnitts darf in Weiß ausbleichen, bevor die Belichtungsautomatik das Bild abdunkelt. |

{% hint style="info" %}
**Lichtanforderungen für Bayer-Multispektralkameras (RGN / OCN / NGB):** Die Szene muss in allen drei Kanälen ausreichend beleuchtet sein, sonst funktioniert die Kalibrierung nicht korrekt – eine einzige Sensorbelichtung deckt alle drei Spektren ab. Verwenden Sie einen DAQ-Lichtsensor, um die Lichtstärke zu messen, oder wechseln Sie in den All-Mono-Modus (M3M), damit jedes Band seine eigene Belichtung erhält. Sollte eine Aufnahme gegen diese Vorgabe verstoßen, erkennt Chloros dies und warnt Sie (die „Unmix-Clamp“-Benachrichtigung).
{% endhint %}

### Pixelformat &amp;

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

Auflösung**Array-Elemente** zeigen die schreibgeschützten Zeilen „Aktuell“ (Format + B x H) und „Binning“ mit dem Hinweis „In den Array-Einstellungen festgelegt“ an – ein Neustart des Streams bei einem Element würde die Synchronisation unterbrechen, daher werden diese im [Array-Einstellungsbereich](#array-settings-pane) verwaltet.**Standalone-Kameras** verfügen über:

| Steuerung | Auswahlmöglichkeiten | Funktion |
| --- | --- | --- |
| **Pixelformat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Sensor-Pixelformat (Bittiefe). |
| **Auflösung** | Voll / Halb / Viertel | Relativ zum aktuellen Binning: Voll = 2048/N × 1536/N bei N×N-Binning. |
| **Binning** | 1×1 (keines) / 2×2 / 4×4 | Hardware-N×N-Binning – höhere Werte verringern die Auflösung, erhöhen jedoch das Signal-Rausch-Verhältnis und die Bildrate. Eine Änderung führt zu einem Neustart des Streams und setzt alle ROIs auf das neue volle Sichtfeld zurück. |
| **Aktuell** | schreibgeschützt | Die tatsächliche Breite × Höhe und (x, y)-Offset, die gerade wirksam sind. |

### Live-Vorschau

Alle Einstellungen in diesem Abschnitt wirken sich **nur auf die Anzeige**aus – sie verändern das, was Sie im Live-Feed sehen, während gespeicherte Aufnahmen linear und unverändert bleiben – mit einer Ausnahme:**Vignette** ist radiometrisch und wirkt sich auch auf Exporte aus (siehe unten).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Steuerung | Bereich / Auswahlmöglichkeiten | Standardwert | Gilt für | Funktion |
| --- | --- | --- | --- | --- |
| **Render-Auflösung** | 360p (am schnellsten) / 480p / 720p / 1080p / Native Sensorauflösung (am langsamsten) | 720p | Alle | Die Höhe, auf der das Backend die radiometrische Vorschau-Kette ausführt. Eine niedrigere Einstellung erhöht die Bildrate, ohne das Sichtfeld zu verändern. |
| **Index**| Kontrollkästchen „Aktivieren“ + Zahnrad | Aus | Nur Bayer-Multispektral,**nicht** Combined-Array-Modelle | Live-Vorschau des Vegetationsindex. Das Zahnrad öffnet das gemeinsam genutzte [Index-Rechenfenster](#index-calculator-pane), der mit den natürlichen Filterbändern der Kamera (z. B. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`) vorbelegt ist. Der benutzerdefinierte Ausdruck plus LUT (ein/aus, Standardwert für „Level“ 3, Standardwert für „min“ 0,2, Standardwert für „max“ 1) wird bei jedem Vorschaubild berechnet. Mitglieder von kombinierten Arrays blenden diese Zeile aus – das Array verfügt über einen gemeinsamen Index. |
| **Weißabgleich** | Aus / Einmal / Kontinuierlich + eine Schaltfläche zur Neuerfassung | Kontinuierlich | Nur RGB | Live-Weißabgleich. Die Schaltfläche „Aktualisieren“ erfasst den Weißabgleich erneut anhand des aktuellen DLS-Spektrums (deaktiviert, wenn der Modus auf „Aus“ steht). |
| **Gamma** | Ein / Aus | Ein | Nur RGB | Gamma (γ = 2,2 LUT) in der Live-Vorschau anzeigen. Gespeicherte Aufnahmen bleiben linear. |
| **Rauschunterdrückung** | Kontrollkästchen + Stärke 0–100 | Aus / 50 | Alle (pro Kamera, auch innerhalb von Arrays) | Bilateraler Filter in der Live-Vorschau. Höhere Werte = glatteres, aber weicheres Detail. |
| **Schärfe** | Kontrollkästchen + Stärke 0–100 | Aus / 30 | Alle | Unscharfmaskierung in der Live-Vorschau, wird zuletzt angewendet. Kann Rauschen verstärken. Nur in der Vorschau. |
| **Vignettierung**| Kontrollkästchen + Stärke 0–100 | Aus / 0 | Alle | Manuelle Rest-Vignettierungskorrektur (hellt die Ecken auf), die über die „Smart Vignette“-Schätzung des Arrays gelegt wird.**Radiometrisch – wirkt sich auf die Live-Ansicht UND Exporte aus**, im Gegensatz zu Rauschunterdrückung/Schärfe. |
| **Farbprofil** | Raw / Linear / Natürlich / Verbessert / Benutzerdefinierte Farbtemperatur | Natürlich | Nur RGB | Siehe unten. |
| **Farbtemperatur** | 2000–10000 K, Schrittweite 100 | 5500 K | Nur RGB, benutzerdefiniertes Temperaturprofil | Legt den Weißabgleich auf eine feste korrelierte Farbtemperatur fest (DLS-Eingabe wird ignoriert). Der zuletzt gewählte Kelvin-Wert bleibt beim Wechseln des Profils erhalten. |
| **Sättigung** | 0–200 (100 = neutral) | 100 | Nur RGB | HSV-Sättigung in der Live-Vorschau. |
| **Kontrast** | 0–200 (100 = neutral) | 100 | Nur RGB | Linearer Kontrast um Mittelgrau in der Live-Vorschau. |
| **Spiegeln H / Spiegeln V** | Kontrollkästchen | Aus | Alle | Die Vorschau horizontal / vertikal spiegeln. |
| **Drehung**| 0° / 90° / 180° / 270° | 0° | Alle | Die Vorschau drehen. Die Ausrichtung wird am Ende der Backend-Vorschau-Kette angewendet –**gespeicherte Aufnahmen behalten ihre kameraneutrale Ausrichtung** bei, und Array-Kompositansichten ignorieren sie. |**Semantik der Farbprofile** (RGB-Kameras):

* **Raw** – die Verarbeitungskette wird vollständig umgangen.
* **Linear** – Dunkelsignal + Flatfield + Weißabgleich; keine Farbmatrix, kein Gamma.
* **Natürlich** *(Standard)* – linear plus die gemessene Farbkorrekturmatrix und eine szenenadaptive Tonwertkurve.
* **Verbessert**— „Natürlich“ plus Lebendigkeit und lokaler CLAHE-Kontrast. Der Aufpreis gilt**nur für die Live-Vorschau** — gespeicherte Aufnahmen erhalten unabhängig vom Profil immer die vollständige Nachbearbeitung.
* **Benutzerdefinierte Farbtemperatur** — „Natürlich“ mit Weißabgleich, der auf den von Ihnen gewählten Kelvin-Wert festgelegt ist.

{% hint style="warning" %}
Bei „Natürlich“, „Verbessert“ und „Benutzerdefinierte Temperatur“ zeigt der Bereich einen Hinweis zur Tonwertkurve an: Die Bilder werden entsprechend ihrer jeweiligen Szene aufgehellt, sodass gespeicherte *Anzeige*-Bilder nicht Bild für Bild vergleichbar sind. **Exportieren Sie Radianz oder Reflektanz für Messungen.**
{% endhint %}

### Anzeige-Overlays (über den Live-Feed gelegt)

Diese sind nur im Frontend verfügbar – sie werden über das Video gelegt, ohne den Stream oder die Aufnahmen zu beeinflussen.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Overlay | Steuerelemente | Standard | Funktion |
| --- | --- | --- | --- |
| **Zebra** | Kontrollkästchen + Schwellenwert 200–255 | Aus / 250 | Magenta-farbige diagonale Streifen auf überbelichteten Pixeln. |
| **Fadenkreuz** | Kontrollkästchen | Aus | Markierung der Bildmitte. |
| **Raster** | Aus / 3 × 3 / 9 × 9 | Aus | Kompositionsraster. |
| **Histogramm** | Kontrollkästchen + Breite 0,10–0,90 des Bildes | Aus / 0,25 | Ein Histogrammstreifen im Live-Feed. |
| **Fokus-Peak** | Kontrollkästchen + Schwellenwert 20–200 + Farbfeld | Aus / 80 / `#ff5722` | Sobel-Kantenhervorhebung zur Fokussierung. |
| **Kanalaufteilung** | „Aufteilung anzeigen (Red / Green / NIR)“ / Schaltfläche „Aufteilungen ausblenden“ | Ausgeblendet | Fügt neben dem zusammengesetzten Bild drei unabhängige Graustufen-Kacheln pro Kanal hinzu (die Beschriftung der Schaltflächen entspricht den Filterkanälen der Kamera). Jede Aufteilungskachel ist verschiebbar und hat die Rahmenfarbe der Kamera. Nicht verfügbar bei Monochrom-Kameras. Wird mit dem Projekt gespeichert. |

### Spot-Belichtungsmesser

* Kontrollkästchen **„Zum Abtasten klicken“**: Klicken Sie auf das Live-Bild, um ein einzelnes Pixel abzutasten (ein Fadenkreuz markiert es), oder klicken und ziehen Sie einen Bereich, um einen Pixeldurchschnitt zu ermitteln. Mit**„Löschen“**werden die Abtastung und das Fadenkreuz entfernt. Schließt den**„Aim“**-Modus von AE-ROI aus.
* Dropdown-Menü **Anzeigen**:**Raw (Bittiefe)**– native digitale Werte in der Bittiefe des Sensors (z. B. 12 Bit → 0..4095) – oder**Anzeige (8 Bit)** (Standard). Wenn ein Live-Index aktiv ist, zeigt „Display“ stattdessen den berechneten Indexwert an (z. B. NDVI).
* Das Auslesefenster listet Pixelkoordinaten, Bildgröße, Pixelformat, Bittiefe und eine Kanaltabelle (Chan / Wert / %) mit Bandbezeichnungen und Wellenlängen auf; Bayer-Grünpaare werden gemittelt; Bereichsproben zeigen „N px avg“ an.

Der Status des Spotmessers gilt nur für die aktuelle Sitzung.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Vorausschauende Belichtungsautomatik (DLS-gesteuert)

Dieser Abschnitt erscheint nur, wenn **mindestens ein DAQ-Lichtsensor angeschlossen ist** – der Solver benötigt ein Live-Spektrum der einfallenden Strahlung, um ihn zu steuern.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Steuerung | Bereich | Standard | Funktion |
| --- | --- | --- | --- |
| **Aktivieren** | Kontrollkästchen | Ein (Standalone-Kameras) | Ein Solver mit geschlossener Form nutzt das DLS-Spektrum sowie die Skalare des Kalibrierungspakets der Kamera, um das hellste Band nahe der Sättigung zu platzieren und gleichzeitig das dunkelste Band über dem SNR-Mindestwert zu halten — ein einziger Belichtungsschreibvorgang pro Lösung, keine Einstellschleife. Konzipiert für solarbetriebene Zeitraffer, bei denen jede Aufnahme korrekt belichtet sein muss. Das Backend weicht stillschweigend auf reaktive Belichtungsautomatik (AE) zurück, wenn der DLS-Messwert veraltet ist/fehlt oder das Kalibrierungspaket nicht geladen ist. |
| **Glättung (α)** | 0,05–1,0, Schrittweite 0,05 | 0,3 | Glättung aufeinanderfolgender prädiktiver Lösungen (niedrigerer Wert = glatter). |
| **Szenenreflexionsgrad** | Schaltfläche „ρ neu kalibrieren“ | — | Schätzt den vom Solver verwendeten Szenenreflexionsgrad neu. |

{% hint style="info" %}
**Bei Array-Kameras ist die prädiktive Belichtungsautomatik standardmäßig deaktiviert** – bei Array-Kameras regeln „Chloros“ Smart AE sowie die kameraseitige Belichtungsautomatik die Belichtung (mit Sättigungsschutz), und die einzelne Szenenreflexionsschätzung der prädiktiven Belichtungsautomatik ist bei gemischten Szenen nicht sicher. Sie können sie hier pro Kamera wieder aktivieren, wenn Sie ausdrücklich eine DLS-gesteuerte radiometrische Belichtung wünschen.
{% endhint %}

**DAQ-gesteuerte Belichtungsobergrenze und auf den einfallenden Lichtstrahl fixierte Belichtungsautomatik.**Unabhängig vom obigen Kontrollkästchen berechnet Chloros – ausgehend von der gemessenen absoluten nach unten gerichteten Bestrahlungsstärke – die maximale Belichtung × Verstärkung, bei der eine Oberfläche mit 100 %-Reflexionsfläche unterhalb der Clipping-Grenze bleibt, und wendet diesen Wert als**Obergrenze**für die automatische Belichtung an. Solange die Obergrenze aktiv ist, arbeitet die Kamera**einfallslichtgesteuert**: Sie läuft im offenen Regelkreis mit der durch das einfallende Licht gemessenen Belichtung und einer Verstärkung von 0 dB – die Belichtung folgt dem gemessenen Licht, nicht dem Bildinhalt. Da die Obergrenze die Belichtungszeit nur verkürzen kann, kann sie selbst kein Clipping verursachen. Die Obergrenze wird automatisch deaktiviert – und die normale Szenen-AE wird wieder aufgenommen –, sobald der DAQ-Messwert fehlt, veraltet ist (&gt;30 s) oder dunkel ist, oder wenn ≥15 % des Bildes bei der festgelegten Belichtung clippen (was bedeutet, dass Sensor und Kamera unterschiedliche Beleuchtungsstärken wahrnehmen). Es gibt keinen Schalter in der Benutzeroberfläche; dies ist das Standardverhalten, sobald eine RGB-Kamera mit einem DAQ verbunden ist.

### Mitglieder des Erfassungs- und Trigger-Arrays

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

zeigen zusätzlich die schreibgeschützten Zeilen **Rolle**(Master in Blau / Slave in Grün),**Synchronisationsleitung**und**Peers** an.

| Steuerung | Auswahlmöglichkeiten | Standard | Anmerkungen |
| --- | --- | --- | --- |
| **Triggermodus** | Aus / Ein | Ein | Für Array-Mitglieder deaktiviert (das Array verwaltet die Triggerung). |
| **Triggerquelle** | Software / Leitung 0 (M8) / Leitung 1 / Leitung 2 | Leitung 0 | Ausgeblendet, solange der Triggermodus auf „Aus“ steht; für Array-Mitglieder deaktiviert. Leitung 0 ist der optoisolierte externe Triggereingang M8. |
| **Bildrate**| Auto / Manuell + Wert | Auto |**Auto**: Die Bildratenbegrenzung der Kamera ist deaktiviert – die Belichtungszeit bestimmt die fps, und das Feld zeigt die aktuelle Live-Bildrate an.**Manuell**: Sie begrenzen die fps mit einem Schieberegler (1 bis zum durch die Bandbreite begrenzten Maximum), ausgehend von der aktuellen tatsächlichen Bildrate. Array-Mitglieder sehen einen schreibgeschützten Wert „N fps (live)“ mit dem Hinweis „In den Array-Einstellungen festgelegt“. |

### Netzwerk / Transport

| Zeile | Verhalten |
| --- | --- |
| **Paketgröße**| 1500 (Standard) / 9000 (Jumbo) – Standardmäßig**Jumbo**. |
| **Durchsatz** | Schreibgeschützte Begrenzung des Link-Durchsatzes in MB/s. Das Backend verteilt diesen bei jeder Verbindung/Trennung neu auf alle verbundenen Kameras. |
| **Pufferverwaltung** | Schreibgeschützter Modus für die Pufferverwaltung. |

### Aufnahme

Der Bereich endet mit einer **„Aufnahmeeinstellungen öffnen…“**, der zum [Fenster „Aufnahmeeinstellungen“](capture.md#the-capture-settings-pane) führt (deaktiviert, bis ein Projekt geöffnet ist – „Erstellen oder öffnen Sie ein Projekt, um Aufnahmen zu speichern“). Wenn die Kamera ausgeblendet oder angehalten ist, erinnert Sie ein Hinweis daran, sie vor der Aufnahme wieder einzublenden bzw. fortzusetzen.

## Bereich „Array-Einstellungen“

Öffnen Sie diesen Bereich über das **Zahnrad**in einer ARRAY-Zeile. Kopfzeile: Array-Name mit einem Bleistift-Symbol zum Umbenennen und**×** zum Schließen. Die unten mit *nur kombiniert* gekennzeichneten Abschnitte erscheinen nur bei Arrays, die im kombinierten Anzeigemodus verbunden sind.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Synchronisation

Schreibgeschützte Zeilen **Master**,**Slaves**und**Synchronisationsleitung**.

### Umgebungslichtsensor

Wird sowohl für kombinierte als auch für separate Arrays angezeigt:

* **Kontrollkästchen**Kalibrierungsziel** — „MAPIR ArUco-Ziel erkennen und NDVI anhand der LUT für die Reflektanz des Panels validieren“; steuert die Zielüberlagerung und die Validierungstabelle der kombinierten Kachel.
* **Dropdown-Menü „**Lichtsensor**“ – ordnet einen DAQ dem gesamten Array zu. Die Auswahl wird sofort übernommen, überträgt sich auf das eigene Dropdown-Menü „Lichtsensor“ jeder einzelnen Kamera (Sie können die Einstellung weiterhin pro Kamera überschreiben) und beginnt mit der Weiterleitung von Spektren an das Array.
* Live-Zeile **Status**: Aus · „Warten auf erstes Spektrum…“ · „Aktiv – alle Kameras im Array sind beleuchtungskorrigiert“ · „Kein neues Spektrum in den letzten 3 s – es wird weiterhin der letzte Messwert verwendet (kein Timeout für veraltete Daten)…“.
* Hinweis im Fenster: „Array-weite radiometrische Korrektur. Einstellungen pro Kamera haben Vorrang.“

### Aufnahme – einheitliche Sensoreinstellungen *(nur kombiniert)*

Diese Einstellungen gelten einheitlich für jedes Element (Änderungen pro Element würden die Synchronisation unterbrechen). Änderungen werden zwischengespeichert und gemeinsam angewendet.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Steuerung | Auswahlmöglichkeiten / Bereich | Funktion |
| --- | --- | --- |
| **Pixelformat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Einheitliches Sensorformat für alle Mitglieder. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardware-Binning – bewahrt das volle Sichtfeld bei gleichzeitiger Steigerung des Signal-Rausch-Verhältnisses und der Bildrate. Eine Änderung setzt die ROI-Felder auf das neue gesamte Sichtfeld zurück. |
| **Auflösung**-Voreinstellung | Voll / Halb / Viertel | Relativ zum Binning; füllt die ROI-Felder mit einem zentrierten Ausschnitt. |
| **ROI-Ausschnitt (px)**| Numerische Felder für B / H / X / Y | Sensorausschnitt. Breite/Höhe rasten an Vielfachen von 16 (mindestens 64); Offsets richten sich nach Vielfachen von 4. Ein Hinweis „max WxH“ zeigt die Obergrenze an, und**Zurücksetzen** kehrt zum vollen Sichtfeld zurück. Während der Bearbeitung wird ein orangefarbener Live-Vorschaurahmen auf die Array-Kachel gezeichnet (einschließlich eines Schemas des gesamten Sensors, wenn der Ausschnitt nach außen erweitert wird). |
| **Auslösefrequenz**| Umschalter „Auto“ / „Manuell“ + fps 0,5–10, Schrittweite 0,5 |**Auto**(Standard): Das Backend leitet die Auslösefrequenz aus Auflösung und Bandbreite ab – die Eingabe ist deaktiviert und zeigt den abgeleiteten Wert an.**Manuell**: Setzt Ihren Wert beim Klicken auf „Übernehmen“ fest. |

Hinweis im Fenster: „Änderungen an Format/Auflösung führen zu einem kurzen Neustart aller Kameras. Die Triggerrate wird live angewendet.“ Die Schaltflächen **Übernehmen / Abbrechen** befinden sich am unteren Rand des Fensters.

### Ausrichtung (Co-Registrierung) *(nur kombiniert)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Restfehler**-Anzeige: „RMS x,xx px“ – grün unter 1 px, gelb unter 3 px, rot in allen anderen Fällen oder wenn eine Kamera fehlgeschlagen ist; „kein Profil“ vor der ersten Berechnung.
* Zusammenfassungszeile: „✓ Alle Kameras ausgerichtet (N)“ / „⚠ p/N Kameras ausgerichtet –  <serial (filter)="">fehlgeschlagen“ / „Ausschnitt aktiv – Zur Ausrichtung neu kalibrieren (nutzt den gesamten Sensor)“ / „Warten auf das Abklingen der Belichtung…“.
* Tabelle pro Kamera: Kamera (letzte 4 Ziffern der Seriennummer + Filter), Reprojektionsfehler in px mit Übereinstimmungsanzahl („ref“ für die Master-Kamera) und der normalisierte Kreuzkorrelationswert der Überlappung im Vergleich zum Mindestwert von 0,35 für das Bestehen.
* **Schaltfläche „Ausrichtung neu kalibrieren“** (zeigt vor dem ersten Profil „Ausrichtung kalibrieren“ an) – führt die Co-Registrierung mit neuen Bildern erneut durch.
* **Kontrollkästchen „Kameras für die Ausrichtung automatisch belichten“** (standardmäßig aktiviert) – hellt dunkle oder flache Kameras vorübergehend auf (zuerst Belichtung, dann Verstärkung), damit sie eine Textur zum Abgleichen haben, und stellt anschließend die automatische Belichtung wieder her.

Die kombinierte Vorschau wird beim Öffnen automatisch ausgerichtet; kalibrieren Sie sie neu, wenn sich der Fokus oder die Szenentiefe geändert haben. Die Ausrichtung ist **konzeptionell auf die jeweilige Sitzung beschränkt** – sie wird niemals in einem Profil gespeichert, da sie von der aktuellen Szenentiefe abhängt. Aufnahmen können weiterhin pixelregistriert exportiert werden (siehe [Ausgerichtete Exporte](capture.md#per-array-controls)).

### Intelligente Vignettierung

* Kontrollkästchen **Korrektur aktivieren**– wendet die kameraspezifische Vignettierungsschätzung auf die radiometrische Kette an (Live-Anzeige**und** Exporte).
* **Von aktueller Ansicht kalibrieren**– Richten Sie das Array zunächst auf ein einheitliches Ziel (Flachbildschirm, Wand oder Himmel); jede Kamera wird einzeln abgeflacht und der Status meldet „n/N Kameras · −x,x %“ als Abflachungsgewinn. Mit**Löschen** wird die Schätzung entfernt.
* Feinabstimmung pro Kamera mit dem kameraspezifischen Schieberegler **Vignettierung** in der [Live-Vorschau](#live-preview).

### Live-Vorschau *(nur kombiniert)** **Index**: Kontrollkästchen aktivieren + Zahnrad — öffnet den gemeinsamen [Index-Rechner](#index-calculator-pane) mit Bändern, die aus**allen** Mitgliedskameras. Eine Ausdrucksvorschauzeile darunter zeigt den aktuellen Ausdruck an („Kein Ausdruck festgelegt – öffnen Sie den Rechner, um einen zu erstellen“), der jede Sekunde aktualisiert wird.
* Dropdown-Menü **Render-Auflösung**(gleiche Voreinstellungen wie pro Kamera, Standard 720p): die Höhe des Live-View-Streams**und** die gespeicherte Exportgröße des Composites. Hinweis im Fenster: „Vorschau + gespeicherte Composite-Größe. Bilder pro Kamera werden immer in voller Auflösung exportiert.“

### Anzeigeebenen *(nur kombiniert)** Kontrollkästchen **Aktivieren** (Standard: deaktiviert – die Master-Kamera wird direkt angezeigt; aktiviert = überlagerter Composite).
* Dropdown-Menüs **Vordergrund**/**Hintergrund**: jede Einzelkamera (nach Name) oder**Index**. Wenn „Vordergrund“ auf „Index“ gesetzt ist, zeigen Pixel außerhalb der LUT-Min/Max-Grenzen die Hintergrundebene an.

### Geteilte Ansicht *(nur kombiniert)*

**„Mitglieds-Kameras anzeigen“**– eine Schaltfläche**„Mitglieds-Kameras aufteilen / ausblenden“**, die den Live-Feed jeder einzelnen Mitglieds-Kamera als separate Rasterkacheln neben der Zusammensetzung hinzufügt. Die Kacheln lesen den vorhandenen Bildspeicher des Arrays aus (keine zusätzliche Kameraverbindung). Nur in der Rasteransicht; wird pro Array mit dem Projekt gespeichert.

### Funktionen

Ein schreibgeschütztes Fenster, das alle 5 s aktualisiert wird:

* **Stufenbezeichnung**: „Simultane Erfassung“ (grün) · „Simultane Erfassung (FTD-versetzte Übertragung)“ (grün) · „Versetzte Erfassung (100 ms Drift)“ (gelb) · „Konfiguration zu groß“ (rot).
* **Frame-Zustand**: „x,xx % unvollständig“ – grün unter 1 %, gelb unter 5 %, rot bei 5 % oder mehr.
* **Verbindungsanzeige**: „NIC {mbps} Mbps – nachhaltig {MB/s} MB/s“.

Dies ist das aktuelle Bandbreitenbudget des Arrays. Informationen zu den zugrunde liegenden fps und dem Netzwerkmodell – sowie dazu, was zu ändern ist, wenn die Stufe gelb oder rot wird – finden Sie unter [Multi-Kamera-Arrays](arrays.md) und in der [CLI-Referenz](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Bereich „Index-Rechner“

Die dritte Seite in der Seitenleiste, die sowohl vom kameraspezifischen Index-Einstellungsmodul als auch vom kombinierten Array-Index-Einstellungsmodul genutzt wird (jeweils nur eines – die Überschrift lautet „Index-Rechner – <camera name="">“ oder „Index-Rechner –<array name="">

“). Hier werden die Bandliste (die kameraspezifischen natürlichen Filterbänder oder alle Bänder aller Array-Mitglieder), den aktuellen Ausdruck und die LUT-Konfiguration (ein/aus, Stufe – Standard 3, Min. – Standard 0,2, Max. – Standard 1) sowie ein Live-Index-Histogramm. Mit **Anwenden** wird der Ausdruck übernommen; LUT-Änderungen werden live auf die Vorschau angewendet.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Kamera-spezifische vs. Array-verwaltete Einstellungen

Kurzanleitung dazu, was wo zu finden ist, wenn eine Kamera Teil eines Arrays ist:

| Vom Array verwaltet (im Kamera-Fenster schreibgeschützt) | Innerhalb eines Arrays weiterhin kameraspezifisch |
| --- | --- |
| Pixelformat, Auflösung, Binning | Automatische Belichtung (Belichtung, Verstärkung, Zielwert, Glättung, ROI) |
| Auslösemodus/-quelle, Bildrate | Rauschunterdrückung, Schärfe, Vignettierung |
| | Ausrichtung (Spiegelung/Drehung), Anzeige-Overlays, Spotmessung |
| | Index (Arrays mit separater Anzeige), Zuordnung zu Lichtsensor |

Sonstiges übergreifendes Verhalten:

* **Kombinierte vs. separate Anzeige** wird beim Verbinden des Arrays ausgewählt: kombiniert = eine ausgerichtete zusammengesetzte Kachel (Mitglieder senden nur über „Split View“); separat = jedes Mitglied rendert seine eigene synchronisierte Kachel. Eine Kamera zeigt niemals sowohl einen eigenständigen Feed als auch eine Array-Kachel an.
* **Automatische Wiederverbindung**: Beim Öffnen eines gespeicherten Projekts werden dessen Kameras und Arrays wiederhergestellt und alle gespeicherten Einstellungen erneut auf das Backend angewendet, bevor die Streams fortgesetzt werden.
* **Aufnahme-Gating**: Ausgeblendete oder pausierte Kameras werden von „Capture All“ ausgeschlossen; ein Array wird erst dann vollständig blockiert, wenn ALLE Mitglieder ausgeblendet/pausiert sind. Siehe [Aufnahmeeinstellungen und -modi](capture.md).

## Wie Einstellungen gespeichert werden

Der Status der Registerkarte „Kamera“ wird **zusammen mit dem Projekt** gespeichert, nicht im Browser:

* Bei jeder reaktiven Änderung wird ein Snapshot der Kameras und Arrays im Projekt `cameras.json` (mit einer Entprellzeit von 500 ms). Dies umfasst Kameranamen und -farben, Belichtungs-/Verstärkungs-/AE-Einstellungen, Pixelformat/Auflösung/Binning, Triggerrate, Vorschau-Einstellungen (Render-Auflösung, Rauschunterdrückung, Schärfe, Vignettierung, Farbprofil, Sättigung/Kontrast), Ausrichtung, Überlagerungen, Kanalaufteilungen, Indexkonfiguration, Predictive-AE-Einstellungen, AE-ROI, Array-Namen, Anzeigemodus, Array-Aufnahmeeinstellungen (einschließlich ROI-Ausschnittposition) sowie den Rasterblock (Feed-Zoom, Ansichtsmodus, Rastersperre, manuelle Kachelreihenfolge, ausgeblendete Kameras, geschlossene Kacheln, aktive Kamera).
* Lichtsensor-Zuordnungen werden in der Projektdatei „`sensors.json`“ gespeichert.
* Beim erneuten Öffnen des Projekts wird die Hardware erneut verbunden und alle Einstellungen werden erneut angewendet.
* **Kein Projekt geöffnet = nur Sitzung**: Ohne Projekt geht beim Schließen von Chloros nichts verloren.
* Nur Sitzung, unabhängig vom Projekt: Pausenzustand, Spotmessungswerte, das Kontrollkästchen „Kalibrierungsziel“ pro Kamera (ist standardmäßig deaktiviert) und das Array-Ausrichtungsprofil (wird konzeptionell pro Sitzung neu berechnet).
* Eine Ausnahme: Die im Export ausgewählten **Aufnahmeeinstellungen** und der Aufnahmemodus bleiben pro Projekt im lokalen App-Speicher erhalten und nicht in `cameras.json` – siehe [Aufnahmeeinstellungen und -modi](capture.md).</array></camera></serial>
