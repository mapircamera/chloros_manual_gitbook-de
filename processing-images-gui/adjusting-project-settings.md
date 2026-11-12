# Projekteinstellungen anpassen

Bevor Sie Ihre Bilder bearbeiten, sollten Sie die Projekteinstellungen entsprechend Ihren Arbeitsabläufen konfigurieren. Über das Fenster „Projekteinstellungen“ <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> haben Sie umfassende Kontrolle über Kalibrierung, Bearbeitungsoptionen, multispektrale Indizes und Exportformate.

## Aufrufen der Projekteinstellungen

1. Öffnen Sie Ihr Projekt in Chloros.
2. Klicken Sie in der linken Seitenleiste auf das Symbol **Projekteinstellungen** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">.
3. Im Fenster „Projekteinstellungen“ werden alle Konfigurationsoptionen angezeigt.

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Das Fenster „Projekteinstellungen“ – Anzeige, Zielerkennung und Bearbeitung</p></figcaption></figure>

{% hint style="info" %}
**Die Einstellungen werden automatisch** zusammen mit Ihrem Projekt gespeichert. Wenn Sie ein Projekt erneut öffnen, werden alle Einstellungen wiederhergestellt.
{% endhint %}

***

## Schnelle Einrichtung für gängige Arbeitsabläufe

### Standardeinstellungen (für die meisten Benutzer empfohlen)

Die Standardeinstellungen eignen sich gut für typische Survey3- und LATTICE-Arbeitsabläufe:

* ✅ **Vignettierungskorrektur**: Aktiviert
* ✅ **Reflexionskalibrierung / Weißabgleich**: Aktiviert (verwendet MAPIR-Targets und/oder DAQ-Lichtsensordaten)
* ✅ **Debayer-Methode**: Standard (schnell, mittlere Qualität)
* ✅ **Exportformat**: TIFF (16-Bit) (TIFF (8-Bit) für ein vollständig 8-Bit-Projekt)
* ✅ **Alle Exportprodukte**: Aktiviert (LATTICE erfasst automatisch den Fan-Out in Debayered, Vorschau, Radiance und Reflektanz)

Importieren Sie einfach Ihre Bilder und beginnen Sie mit der Verarbeitung unter Verwendung dieser Standardeinstellungen.

***

## Übersicht über die Projekteinstellungen

Der Bereich „Projekteinstellungen“ ist in die unten aufgeführten Abschnitte unterteilt. Zwei zusätzliche Abschnitte – **DAQ-Lichtsensor**und**Array-Ausrichtung** – werden automatisch angezeigt, wenn Ihr Projekt die entsprechenden Dateien enthält. Die vollständige Dokumentation finden Sie unter [Projekteinstellungen](../project-settings/project-settings.md).

### Anzeige

* **Auflösung der Bildminiaturen**: Auflösung der Miniaturansichten im Bildraster. Optionen:**Standard (512 px)**,**1024 px**,**2048 px**,**Volle Auflösung**. Nur zur Anzeige – hat keinerlei Auswirkungen auf die Verarbeitung. Höhere Werte sehen beim Vergrößern schärfer aus, werden jedoch langsamer geladen.

### Zielerkennung

Steuert, wie Chloros-Kalibrierungsziele in Ihren Bildern identifiziert.

**Wichtige Einstellungen:*** **Mindestfläche für Kalibrierungsproben (px)**: Größenschwelle für die Zielerkennung (Standard:**25**, Bereich 0–10000)
* **Mindestwert für Zielclusterbildung (0–100)**: Ähnlichkeitsschwellenwert für die Gruppierung von Zielbereichen (Standard:**60**)**Wann anpassen:**

* Erhöhen Sie die Probenfläche, wenn es zu Fehlerkennungen kommt
* Verringern Sie sie, wenn Ziele nicht erkannt werden
* Passen Sie die Clusterbildung an, wenn Ziele in mehrere Erkennungen aufgeteilt werden

{% hint style="info" %}
Diese Einstellungen sind ausgegraut, wenn **Reflexionskalibrierung / Weißabgleich** deaktiviert ist – bei deaktivierter Funktion wird die Zielerkennung überhaupt nicht ausgeführt.
{% endhint %}

### Verarbeitung

Wichtige Optionen zur Bildverarbeitung und Kalibrierung.

**Wichtige Einstellungen:*** **Vignettierungskorrektur**: Gleicht die Randverdunkelung des Objektivs aus ✅ Empfohlen
* **Reflexionskalibrierung / Weißabgleich**: Kalibriert Bilder anhand erkannter Zielmarken (Survey3) und/oder Daten des DAQ-Lichtsensors (LATTICE) ✅ Empfohlen
* **Debayer-Methode**: Algorithmus zur Konvertierung von RAW in 3-Kanal-Multispektralbilder
* **Mindestintervall für die Neukalibrierung**: Mindestzeit in Sekunden zwischen der Verwendung von Kalibrierungszielen (Standard:**0** = alle verwenden, Bereich 0–3600)**Nicht kalibrierte Fallback-Produkte:**Wenn ein Bild nicht reflektanzkalibriert werden kann (kein Ziel verfügbar oder Kalibrierung deaktiviert), wird es als eines von zwei Fallback-Produkten exportiert –**pro Durchlauf existiert genau eines der beiden**, ausgewählt durch den Schalter „Vignettenkorrektur“:

* **Sensorkennlinie exportieren**: schreibt `Sensor_Response_Images` – wird verwendet, wenn die Vignettierungskorrektur**ausgeschaltet** ist
* **Vignettenkorrektur exportieren**: schreibt `Vignette_Corrected_Images` – wird verwendet, wenn die Vignettenkorrektur**aktiv** ist

Das Kontrollkästchen, das nicht verwendet wird, ist ausgegraut. Wird das aktive Kontrollkästchen deaktiviert, wird die entsprechende Datei überhaupt nicht geschrieben.

**LATTICE-Exportprodukte** (werden für jedes Projekt angezeigt; sie gelten für LATTICE-Aufnahmen):

* **Debayert exportieren**: das linear debayerte Bild (`Debayered_Images`): Der Dunkeloffset des Kalibrierungspakets wird entfernt, anschließend erfolgt die Demosaikierung ohne Flat-Field-Bild oder Unmix – dieselbe Datei, die eine Aufnahme auf ihrer debayierten Ebene speichert – in der nativen Ausrichtung des Sensors. Gilt für RGB- und Multispektralmodule.
* **Vorschau exportieren**: die Anzeigevorschau (`Preview_Images`). RGB = Weißabgleich (DAQ-Lichtart, sofern verfügbar, sonst „Gray-World“) + Gamma; Multispektral = Falschfarben-Streckung.
* **Strahlungsintensität exportieren**: spektrale Strahlungsintensität als float32 (`Radiance_Images`, W/m²/sr/nm). Nur für multispektrale Module – gilt nicht für RGB-Master.
* **Reflexionsgrad exportieren**: uint16-Reflexionsgrad (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), wenn ein `.daq`-Downwelling-Messwert oder ein Zielobjekt im Bild den Bildausschnitt abdeckt. Nur für Multispektralmodule.

Alle vier sind **standardmäßig aktiviert**– ein importiertes LATTICE-Rohbild wird in einem einzigen Verarbeitungsdurchlauf auf alle aktivierten und anwendbaren Produkte verteilt. Das Kontrollkästchen**Reflexionsgrad exportieren** ist ausgegraut, wenn die Reflexionsgradkalibrierung deaktiviert ist. Einstellungen, die durch einen übergeordneten Schalter deaktiviert werden, sind immer ausgegraut und mit einem Tooltip versehen, der den zu ändernden Schalter nennt.**Erweiterte Einstellungen:*** **Zeitzonen-Offset des Lichtsensors**: Stunden ab UTC für die Zeitabgleichung des Lichtsensors (Standard: 0, Bereich −12 bis +12)
* **PPK-Korrekturen anwenden**: Verwendet GPS-/Belichtungs-Pin-Daten aus `.daq`-Dateien (Standard: deaktiviert)
* **Belichtungs-Pin 1/2**: Weist Kameras den Belichtungs-Pins für Dual-Kamera-Konfigurationen zu

{% hint style="info" %}
**Der LATTICE-Eingabepegel wird automatisch festgelegt.** LATTICE-Aufnahmen enthalten ihren Verarbeitungspegel in den XMP-Metadaten, und die Verarbeitung erfolgt immer auf Basis des Rohbildes – in der GUI gibt es nichts zu konfigurieren. (Das CLI-Flag `--input-level` dient als Überbrückungsoption für fortgeschrittene Benutzer bei Aufnahmen mit fehlenden Metadaten; siehe die [CLI-Referenz](../reference/cli-reference.md).)
{% endhint %}

### Debayer-Methode

Derzeit bieten wir in Chloros zwei Debayering-Methoden an:

#### Standard (schnell, mittlere Qualität)

Der Standard-Debayer arbeitet schnell, zeigt jedoch farbliches Rauschen beim Debayering, was zu weniger genauen und verrauschteren Bildern führt.

#### „Texture Aware“ (langsam, höchste Qualität) \[Nur Chloros+]

„Texture Aware“ verwendet einen hochwertigen, kantensensitiven Debayer in Kombination mit einem KI-/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt. Das Modell benötigt GPU-Speicher (VRAM) zur Ausführung: Mit **7 GB oder mehr VRAM** kann es mehrere Bilder gleichzeitig verarbeiten; unter 7 GB wird jeweils nur ein Bild verarbeitet (deutlich langsamer). Siehe [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-Aufnahmen verwenden immer das Standard-Demosaicing.**Es gibt kein für LATTICE trainiertes „Texture Aware“-Modell, daher wird diese Option für LATTICE-Bilder nicht angeboten –**Survey3**-Bilder im selben Projekt können sie jedoch weiterhin nutzen.
{% endhint %}

### Indizes (multispektrale Indizes)

Konfigurieren Sie, welche Vegetationsindizes berechnet und exportiert werden sollen. Das Dropdown-Menü der Benutzeroberfläche bietet **27 vordefinierte Indexformeln**.**So fügen Sie Indizes hinzu:**

1. Klicken Sie auf die Schaltfläche**„Index hinzufügen“**

2. Wählen Sie einen Index aus dem Dropdown-Menü aus (NDVI, NDRE, GNDVI usw.)
3. Konfigurieren Sie die Visualisierungseinstellungen (LUT-Farben, Wertebereiche)
4. Fügen Sie bei Bedarf mehrere Indizes hinzu

**Gängige Indizes:*** **NDVI**: Allgemeiner Vegetationszustand (am häufigsten verwendet)
* **NDRE**: Früherkennung von Stress mit RedEdge
* **GNDVI**: Empfindlich gegenüber der Chlorophyllkonzentration
* **OSAVI**: Eignet sich gut für sichtbaren Boden
* **EVI**: Regionen mit hohem Blattflächenindex (LAI)

**Benutzerdefinierte Formeln:**

* Erstellen Sie benutzerdefinierte multispektrale Indexformeln mit Bandberechnungen über alle Bildkanäle hinweg
* Speichern Sie benutzerdefinierte Formeln zur Wiederverwendung
* Benutzerdefinierte Formeln sind eine Funktion von Chloros+; die Verfügbarkeit hängt von Ihrer Tarifstufe ab

Eine Übersicht über alle verfügbaren Indizes und Formeln – einschließlich der Angabe, welche Namen nur in der GUI verfügbar sind und welche auch in der CLI bzw. im SDK funktionieren – finden Sie unter [Multispektrale Indexformeln](../project-settings/multispectral-index-formulas.md).

### Export

Steuert das Ausgabe-Dateiformat.

**Verfügbare Formate**(Einstellung:**Kalibriertes Bildformat**, Standard**TIFF (16-Bit)**):

* **TIFF (16-Bit)**: Empfohlen für GIS und wissenschaftliche Analysen
* **TIFF (8-Bit)**: 8-Bit-TIFF
* **TIFF (32-Bit, Prozent)**: Gleitkommawerte
* **PNG (8-Bit)**: Verlustfreie Komprimierung zur Visualisierung
* **JPG (8-Bit)**: Kleinste Dateien, verlustbehaftete Komprimierung

Die Liste zeigt, welche Bildformate das Projekt unterstützen kann: Chloros führt keine Hochskalierung der Bittiefe durch. Für reine 8-Bit-Bilder stehen nur TIFF (8-Bit), PNG (8-Bit) und JPG (8-Bit) zur Verfügung, wobei standardmäßig TIFF (8-Bit) verwendet wird. In einem gemischten Projekt wird jede Aufnahme mit ihrer eigenen Farbtiefe oder einer niedrigeren exportiert. Radiance bleibt 32-Bit-Float.

Die Ausgabedateien werden im Projektordner gespeichert, gruppiert nach Kamera und Format: `<project>/<camera>/<format>/<Product>_Images/`. Radiance wird unabhängig von dieser Einstellung **immer** als Float32 in den Ordner `tiff32` geschrieben. Exportierte Dateien behalten den Quelldateinamen bei – der Ordner identifiziert das Produkt. Multispektrale LATTICE-Exporte können außerdem pixelweise Clamp-Masken in einen Ordner „`clamp_masks/`“ neben den Produktordnern schreiben. Den vollständigen Ausgabebaum finden Sie unter [Abschluss der Verarbeitung](finishing-the-processing.md).

{% hint style="warning" %}
**Auslesen der Reflexionswerte**: Der DN-Wert, der ρ = 1,0 entspricht, hängt von der Quellkamera ab – LATTICE verwendet 32768 (als XMP-Tag `Chloros:PixelScale` hinterlegt), Survey3 verwendet 65535. Lesen Sie das Tag aus, anstatt von einem konstanten Wert auszugehen. Siehe [Ausgabebildformate](../output-image-formats.md).
{% endhint %}

### DAQ-Lichtsensor

In diesem Abschnitt werden alle DAQ-Downwelling-Dateien (`.daq` / `.csv`) in Ihrem Projekt aufgelistet, eine Zeile pro Datei, wobei das Sensormodell, der Dateiname und die für diese Datei geltende **Kappen**-Korrektur des Diffusors Korrektur, die für diese Datei gilt.

* **Korrektur-Überschreibung (alle Dateien)**: ein einziges projektweites Dropdown-Menü.**Auto** (Standard) verwendet die in jeder Datei aufgezeichnete Cap – wenn nichts aufgezeichnet wurde, wird Sonnenschein angenommen; DAQ-U, DAQ-M und DAQ-E werden jedoch ohne Cap ausgeliefert, und Chloros kann nicht erkennen, was angepasst wurde, sodass die Verarbeitung bei einer angenommenen Cap eine Warnung ausgibt: Wählen Sie die Cap aus, die angepasst wurde. Die Auswahl einer Kappe überschreibt alle Dateien mit Ausnahme derjenigen eines DAQ-E-S, dessen Diffusor Teil seiner Kalibrierung ist: Rohaufzeichnungen werden damit korrigiert, und Aufzeichnungen, die bereits eine Kappe enthalten, werden neu referenziert (die aufgezeichnete Korrektur wird rückgängig gemacht, die ausgewählte Kappe wird angewendet).
* In den Zeilen wird eine Warnung angezeigt, wenn eine aufgezeichnete Kappe die vom Hub angenommene Standardeinstellung war und nicht vom Bediener bestätigt wurde, sowie wenn für die ausgewählte Kappe kein Profil für dieses Gerätemodell vorhanden ist (die Überschreibung wird für diese Datei abgelehnt).

DAQ-Aufzeichnungen, die auf der Registerkarte „Lichtsensoren“ erstellt wurden, werden automatisch zum geöffneten Projekt hinzugefügt, und importierte `.daq`-/`.csv`-Dateien erscheinen hier, sobald sie hinzugefügt wurden.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Untere Projekteinstellungen – Index, Exportformat, der Abschnitt „DAQ-Lichtsensoren“ sowie die Steuerelemente für Projektvorlagen und Ordner</p></figcaption></figure>### Array-Ausrichtung

Dieser Abschnitt wird **nur** angezeigt, wenn mindestens ein Bild im Projekt die Modul-zu-Modul-Ausrichtungstransformation enthält, die LATTICE-Arrays bei der Erfassung einfügen (`Chloros:Alignment*` XMP). Hier wird angezeigt, wie viele Bilder Tags enthalten und welche Kamera als Referenz dient, mit folgenden Steuerelementen:

* **Array-Ausrichtung anwenden** (Standard: aktiviert): Jedes verarbeitete Produkt (Debayering / Vorschau / Radiance / Reflektanz / Index) wird in die gemeinsame Referenzgeometrie des Arrays transformiert. Deaktiviert = Export in nativer Sensor-Geometrie.
* **Auf gemeinsame Überlappung zuschneiden** (Standard: aktiviert): Schneidet ausgerichtete Exporte auf den Bereich zu, den alle Module gemeinsam haben, sodass jedes Band denselben Bildausschnitt hat. Deaktiviert bleibt die volle Sensorfläche erhalten (schwarze Füllung außerhalb der Quelle).
* **Neuberechnung**:**Bilinear (glatt, Standard)**,**Nearest (exakte Werte beibehalten)**– keine Vermischung zwischen Pixeln, für strenge radiometrische Analysen – oder**Cubic (schärfste)**.

***

## Einstellungen speichern und laden

### Projektvorlage speichern

Erstellen Sie wiederverwendbare Vorlagen für konsistente Arbeitsabläufe:

1. Konfigurieren Sie alle gewünschten Einstellungen im Bereich „Projekteinstellungen“
2. Scrollen Sie zum Abschnitt **„Projektvorlage speichern“** am unteren Rand
3. Geben Sie einen aussagekräftigen Namen für die Vorlage ein (z. B. „Survey3N\_RGN\_Agriculture“)
4. Klicken Sie auf das Speichersymbol

**Vorteile:**

* Identische Einstellungen in mehreren Projekten anwenden
* Konfigurationen mit Teammitgliedern teilen
* Konsistenz bei wiederholten Vermessungen gewährleisten

### Vorlage für ein neues Projekt laden

Beim Erstellen eines neuen Projekts:

1. Wählen Sie **„Neues Projekt“** aus dem Hauptmenü aus
2. Wählen Sie eine Projektvorlage in der optionalen Vorlagenauswahl aus
3. Alle Einstellungen aus der Vorlage werden automatisch übernommen

### Arbeitsverzeichnis

Die Einstellung **„Arbeitsverzeichnis“** legt fest, wo neue Projekte standardmäßig erstellt werden:

* **Standardpfad**: `Documents\MAPIR\Chloros Projects` in Ihrem Windows-Ordner „Dokumente“ (normalerweise `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; der Pfad folgt dem Ordner „Dokumente“, falls dieser durch OneDrive oder ein Roaming-Profil verschoben wird)
* **Speicherort ändern**: Klicken Sie auf das Bearbeitungssymbol und wählen Sie einen neuen Ordner aus
* **Gemeinsam mit der CLI genutzt**: `chloros-cli` verwendet dieselbe Standardeinstellung für den Projektordner
* **Halten Sie den Pfad unter Windows kurz**: Sofern lange Pfade unter Windows nicht aktiviert sind, wird ein Projektordner, dessen Pfad länger als etwa 130 Zeichen ist, vor der Verarbeitung abgelehnt – siehe [Starten der Verarbeitung](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Wann Sie den Pfad ändern sollten**:
  * Netzlaufwerk für die Zusammenarbeit im Team
  * Ein anderes Laufwerk mit mehr Speicherplatz
  * Organisierte Ordnerstruktur nach Jahr/Kunde

***

## PPK-Einrichtung (Post-Processed Kinematic)

Bei Verwendung von MAPIR-DAQ-Rekordern mit GPS zur präzisen Geolokalisierung:

### Voraussetzungen

* MAPIR-DAQ mit GPS-Modul (GNSS)
* .daq-Protokolldatei mit Belichtungs-Pin-Einträgen
* Kamera, die während der Aufnahmesitzung an die Belichtungs-Pins des DAQ angeschlossen war

### Konfigurationsschritte

1. Legen Sie die .daq-Protokolldatei in Ihrem Projektordner ab
2. Aktivieren Sie in den Projekteinstellungen das Kontrollkästchen **„PPK-Korrekturen anwenden“**

3. Stellen Sie bei Bedarf den**„Zeitzonen-Offset des Lichtsensors“** ein (Standard: 0 für UTC)
4. Weisen Sie den Belichtungspins Kameras zu:
   * **Einzelkamera**: Wird automatisch Pin 1 zugewiesen
   * **Zwei Kameras**: Weisen Sie jede Kamera manuell dem richtigen Pin zu**Zuordnung der Belichtungs-Pins:*** **Belichtungs-Pin 1**: Wählen Sie das Kameramodell aus der Dropdown-Liste aus
* **Belichtungs-Pin 2**: Wählen Sie die zweite Kamera oder „Nicht verwenden“
* Ein und dieselbe Kamera kann nicht beiden Pins zugewiesen werden

{% hint style="warning" %}
**Wichtig**: Die Belichtungs-Pins müssen den jeweiligen Kameras korrekt zugewiesen werden. Eine falsche Zuweisung führt zu falschen Geolokalisierungsdaten.
{% endhint %}

***

## Fortgeschrittene Szenarien

### Projekte mit mehreren Kameras

Bei der Verarbeitung von Bildern mehrerer MAPIR-Kameras in einem Projekt:

1. Chloros erkennt automatisch jedes Kameramodell (sowohl Survey3 als auch LATTICE)
2. Jede Kamera erhält die entsprechenden Verarbeitungsprofile und einen eigenen Ausgabeordnerbaum
3. PPK: Weisen Sie jede Survey3-Kamera manuell dem richtigen Belichtungs-Pin zu
4. Alle Kameras verwenden dasselbe Exportformat und dieselben Indizes

**Beispiele**: Survey3W-RGN- + Survey3N-OCN-Doppelkamerasystem oder ein LATTICE-Array, das einen RGB-Master mit Schmalbandmodulen kombiniert

### Zeitraffer- oder Mehrzeitpunkt-Erhebungen

Für wiederholte Erhebungen desselben Gebiets im Zeitverlauf:

1. Erstellen Sie eine Vorlage mit Ihren Standardeinstellungen
2. Verwenden Sie bei jeder Sitzung eine einheitliche Anordnung der Kalibrierungsziele
3. Verarbeiten Sie jeden Datumsstempel als separates Projekt
4. Verwenden Sie identische Einstellungen für vergleichbare Ergebnisse
5. Exportieren Sie im gleichen Format für zeitliche Analysen

### Große Datensätze

Für Projekte mit vielen Bildern (500+):

* Erwägen Sie eine Aufteilung in kleinere Projekte nach Datum oder Gebiet
* Nutzen Sie die Parallelverarbeitung von „Chloros+“ für schnellere Ergebnisse
* Ziehen Sie die Verwendung von CLI oder API für die Stapelverarbeitung in Betracht
* Passen Sie das minimale Neukalibrierungsintervall an, um die Zeit für die Zielerkennung zu verkürzen

***

## Überprüfung Ihrer Einstellungen

Überprüfen Sie vor Beginn der Verarbeitung diese wichtigen Einstellungen:

* [ ] Kameramodell im Dateibrowser korrekt erkannt
* [ ] Vignettierungskorrektur aktiviert
* [ ] Reflexionskalibrierung aktiviert
* [ ] Für Survey3: Mindestens ein Kalibrierungszielbild importiert und überprüft; für „LATTICE“: Ein Ziel und/oder eine `.daq`-Downwelling-Aufzeichnung vorhanden
* [ ] Gewünschte Multispektralindizes hinzugefügt
* [ ] Für Ihren Arbeitsablauf geeignetes Exportformat
* [ ] PPK-Einstellungen konfiguriert (bei Verwendung von .daq-Dateien mit Belichtungsereignissen)

***

## Nächste Schritte

Sobald Ihre Einstellungen konfiguriert sind:

1. **Kalibrierungszielbilder markieren** – Siehe [Auswahl von Zielbildern](choosing-target-images.md)
2. **Verarbeitung starten** – Siehe [Starten der Verarbeitung](starting-the-processing.md)
3. **Fortschritt überwachen** – Siehe [Überwachen der Verarbeitung](monitoring-the-processing.md)

Ausführliche Informationen zu allen verfügbaren Einstellungen finden Sie in der Referenzdokumentation unter [Projekteinstellungen](../project-settings/project-settings.md).
