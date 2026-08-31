# Abschluss der Verarbeitung

Sobald Chloros die Verarbeitung abgeschlossen hat, ist es an der Zeit, Ihre Ergebnisse zu überprüfen, die Ausgabequalität zu kontrollieren und Ihre verarbeiteten Bilder für die Verwendung in Ihrem Workflow vorzubereiten. Diese Seite führt Sie durch die letzten Schritte und die nächsten Maßnahmen.

## Anzeige „Verarbeitung abgeschlossen“

Wenn die Verarbeitung erfolgreich abgeschlossen ist, werden mehrere Anzeigen eingeblendet:

* ✅ **Fortschrittsbalken**: Erreicht 100 % Fertigstellung
* ✅ **Debug-Protokoll**: Zeigt die letzten Zeilen von `[RUN-SUMMARY]` mit den jeweiligen Zahlen an (Bilder, Kameragruppen, Ziele, kalibrierte Bilder, geschriebene Dateien)
* ✅ **Start-Schaltfläche**: Wird wieder aktiviert (bereit für den nächsten Verarbeitungslauf)
* ✅ **Ausgabedateien**: Alle verarbeiteten Bilder werden im Ausgabeverzeichnis des Projekts (unten) gespeichert

{% hint style="warning" %}
**Ein Lauf, bei dem keine Bilder geschrieben werden, gilt als Fehler.** Wenn Sie Bildprodukte angefordert haben und der Durchlauf keine geschrieben hat, meldet Chloros dies als Fehler – `[RUN-SUMMARY]` deutet im Protokollnamen auf die wahrscheinliche Ursache hin (nichts importiert, kein Ziel erkannt, oder alle angeforderten Produkte wurden als nicht anwendbar übersprungen). Das Äquivalent CLI beendet den Lauf mit einem Wert ungleich Null. Ein absichtlich nur auf Metadaten ausgerichteter Lauf (alle Exportprodukte deaktiviert, keine Indizes) gilt weiterhin als erfolgreich. Siehe [die CLI-Referenz](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## So finden Sie Ihre verarbeiteten Bilder

### Öffnen des Ausgabeordners

1. Klicken Sie im **Hauptmenü** auf das Symbol „<img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">“ (oben links)
2. Wählen Sie **„Projektordner öffnen“**

3. Ihr Datei-Explorer öffnet sich im Projektverzeichnis
4. Suchen Sie Ihr Projekt anhand des Namens

### Die Ausgabestruktur

Die Produkte werden **im Projektordner gespeichert, gruppiert nach Kamera und anschließend nach Dateiformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Kameraordner**: `LATT-<sensor>-<lens>-F<filter>` für LATTICE (entspricht dem EXIF-Eintrag der Aufnahme `Model`), `<model>_<filter>` für Survey3 (z. B. `Survey3N_RGN`). Zwei Kameras, die denselben Sensor und Filter verwenden, sich jedoch im Objektiv unterscheiden, behalten separate Verzeichnisstrukturen bei – Vignettierung, Sichtfeld und Verzeichnung unterscheiden sich.
* **Format-Ordner**: richtet sich nach Ihrer Einstellung für das Exportformat – `tiff16`, `tiff8`, `png8`, `jpg8` oder `tiff32` für TIFF (32-Bit, Prozent). Die Strahlungsintensität ist immer vom Typ „float32“ und wird stets unter `tiff32` gespeichert.
* **Produktordner**:
  * `Reflectance_Calibrated_Images/` – kalibrierte Reflexion
  * `Debayered_Images/` – linear debayert (LATTICE)
  * `Preview_Images/` – Anzeige-Vorschau (LATTICE)
  * `Radiance_Images/` – spektrale Strahlungsintensität (Float32), W/m²/sr/nm (LATTICE multispektral)
  * `Vignette_Corrected_Images/` **oder** `Sensor_Response_Images/` — der unkalibrierte Fallback für Bilder ohne Reflektanzreferenz; pro Durchlauf existiert genau einer der beiden, ausgewählt durch die Einstellung „Vignettenkorrektur“
  * `<INDEX>_Index_Images/` — ein Ordner pro ausgewähltem Index (z. B. `NDVI_Index_Images`)

{% hint style="info" %}
**Jedes exportierte Produkt behält den Namen der QUELLDATEI bei.**Ein Radiance-Export von `capture_..._raw.tif` heißt weiterhin `capture_..._raw.tif` – er befindet sich lediglich in `tiff32/Radiance_Images/`.**Der Ordner identifiziert das Produkt, nicht der Dateiname**; daher liefert die Suche nach „`*radiance*.tif`“ keine Ergebnisse; suchen Sie stattdessen nach dem Verzeichnis.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Wie viele Dateien sollten vorhanden sein?

Zählen Sie nicht nach einer Formel – die Anzahl der Ausgabedateien hängt davon ab, welche Produkte aktiviert wurden und welche für die jeweilige Kamera gelten (z. B. erhalten Kameras vom Typ RGB keine Strahlungs- oder Reflexionsdaten). Die maßgebliche Anzahl findet sich im Protokoll: Die letzte Zeile „`[RUN-SUMMARY]`“ gibt genau an, wie viele Dateien geschrieben wurden, und in den Hinweszeilen wird erläutert, was übersprungen wurde.

***

## Überprüfen der verarbeiteten Bilder

### Schnellvorschau im Datei-Explorer

**Windows integrierte Vorschau:**

1. Navigieren Sie in einen Produktordner (z. B. `tiff16/Reflectance_Calibrated_Images/`)
2. Wählen Sie eine Bilddatei aus
3. Die Vorschau wird im Vorschau-Fenster des Windows-Explorers angezeigt
4. Blättern Sie mit den Pfeiltasten durch die Bilder

### Vorschau in externen Bildbetrachtern

**Empfohlene Bildbetrachter:*** **QGIS** – Kostenlose GIS-Software (ideal für georeferenzierte Multispektralanalysen)
* **IrfanView** – Schneller, ressourcenschonender Bildbetrachter (unterstützt TIFF)
* **Adobe Photoshop** – Professionelle Bildbearbeitung (TIFF-Unterstützung)
* **GIMP** – Kostenlose Alternative zu Photoshop
* **Windows Photos** – Einfache Bildanzeige (unterstützt möglicherweise kein 16-Bit-TIFF)

### Vorschau im Chloros-Bildbetrachter

Verwenden Sie den in Chloros integrierten Bildbetrachter für eine erweiterte Visualisierung:

1. Klicken Sie im Dateibrowser auf eine Bildminiaturansicht
2. Das Bild wird im Hauptvorschaubereich geöffnet
3. Klicken Sie in der linken Seitenleiste auf die Registerkarte **Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
4. Verwenden Sie [Index/LUT-Sandbox](../image-viewer-gui/index-lut-sandbox.md) für interaktive Analysen

Ausführliche Anweisungen finden Sie unter [Bildbetrachter](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Auslesen der Reflexions-Pixelwerte (GIS / Pix4D / Skripte)

Die Reflexion wird als ganzzahliger DN-Wert gespeichert, und **der DN-Wert, der ρ = 1,0 entspricht, hängt von der Quellkamera ab**:

| Quelle          | ρ = 1,0 entspricht | Wie lässt sich dies feststellen                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (Spielraum bis zu ρ 2,0) | Das XMP-Tag `Chloros:PixelScale=32768` ist in die Datei eingetragen |
| Survey3         | **65535** (bei ρ 1,0 abgeschnitten)     | Keine `Chloros:*`-XMP-Tags – dieses Fehlen ist das Signal |

**Lies das `Chloros:PixelScale`-Tag und teile durch diesen Wert**, anstatt pauschal von 65535 auszugehen – die Division der LATTICE-Reflexion durch 65535 halbiert stillschweigend jeden Wert. Ein Sonderfall enthält bewusst kein Skalierungs-Tag: Eine als 8-Bit-Ausgabe geschriebene Aufnahme mit 8-Bit-Quelle wird abgeschnitten, nicht neu skaliert, und erhält absichtlich kein Skalierungs-Tag – exportieren Sie stattdessen mit 16 Bit oder 32 Bit, anstatt den Wert zu dividieren. Siehe [Ausgabebildformate](../output-image-formats.md) für die vollständige Erklärung.***

## In die Exporte übernommene Metadaten

Jedes Produkt behält den **GPS-Block**der Quellaufnahme und dessen**EXIF-Sub-IFD** bei, sodass ein
Export die Daten `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` und
`CameraSerialNumber` sowie die Georeferenzierung.

{% hint style="warning" %}
**Wenn ein Orthomosaik in einem absurden Maßstab ausgegeben wird, überprüfen Sie zunächst `FocalLength`.**
Pix4D berechnet den Ground Sample Distance (GSD) anhand der Brennweite und der Flughöhe. Ohne das Tag
greift das Programm auf einen völlig falschen Maßstab zurück – bei einem gemessenen Flug mit 49 Aufnahmen wurde ein 411 m × 160 m
Orangenhain als 47,8 km × 13 km rekonstruiert, was ein 455-Megapixel-Orthofoto von überwiegend
leerem Raum ergab. Langsames Tiling und eine unerwartet große Datei sind Symptome dafür, keine separaten
Probleme.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Es wird nicht *jedes* Tag kopiert. Die Struktur-Tags von IFD0 werden bewusst zurückgelassen (das Kopieren
dieser Tags führt zu einer Beschädigung der LATTICE-Ausgabe), und `ExifImageWidth` / `ExifImageHeight` werden ausgeschlossen,
da sie die ursprüngliche Aufnahme beschreiben – ein Export mit geänderter Größe würde andernfalls
Abmessungen beanspruchen, die im Widerspruch zu seinem eigenen Raster stehen.

***

## Überprüfen des Debug-Protokolls

### Auf Warnungen oder Fehler prüfen

1. Öffnen Sie das **Debug-Protokoll** auf der Registerkarte „<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">“
2. Blättern Sie durch die Meldungen
3. Suchen Sie nach gelben Warnungen oder roten Fehlern
4. Lesen Sie die Zeilen „`[RUN-SUMMARY]`“ und etwaige Hinweise
5. Wenden Sie sich an den MAPIR-Support, um Unterstützung zu erhalten

### Speichern des Protokolls

Um einen Nachweis über die Verarbeitung zu führen oder das Protokoll an den MAPIR-Support zu senden:

1. Klicken Sie auf die Schaltfläche **„Kopieren“**oder**„Herunterladen“**

2. Speichern Sie das Protokoll als Textdatei im Projektordner
3. Fügen Sie es der Projektdokumentation bei
4. Senden Sie es bei auftretenden Problemen an den MAPIR-Support

***

## Häufige Probleme bei der Ausgabe und deren Lösungen

### Problem: Fehlende Ausgabedateien

**Mögliche Ursachen:**

* Das Produkt ist für diese Kamera nicht geeignet (z. B. Strahlungsintensität/Reflexionsgrad für RGB-Kameras – dies wird im Protokoll angegeben)
* Eine erforderliche Referenz fehlte (z. B. Reflektanz ohne Ziel und ohne `.daq`-Downwelling)
* Das Kontrollkästchen für den Export des Produkts war in den Projekteinstellungen deaktiviert
* Der Speicherplatz reichte während des Exports nicht aus

**Lösungen:**

1. Überprüfen Sie die Zeilen „`[RUN-SUMMARY]`“ und „`[EXPORT-CHECK]`“ im Debug-Protokoll – sie erklären die kameraspezifischen Überspringungen
2. Überprüfen Sie die Kontrollkästchen für die Exportprodukte in den [Projekteinstellungen](adjusting-project-settings.md)
3. Stellen Sie sicher, dass ausreichend Speicherplatz vorhanden war
4. Führen Sie die Verarbeitung nach Behebung der Ursache erneut durch

### Problem: Dunkle oder helle Ränder (Vignettierung weiterhin sichtbar)

**Mögliche Ursachen:**

* Vignettierungskorrektur deaktiviert
* Kamera/Objektiv nicht in der Chloros-Profil-Datenbank enthalten
* Extreme Vignettierung, die die Korrekturmöglichkeiten übersteigt

**Lösungen:**

1. Überprüfen Sie, ob die Vignettierungskorrektur in den Projekteinstellungen aktiviert war
2. Überprüfen Sie, ob das Kameramodell korrekt erkannt wurde
3. Wenden Sie sich an den MAPIR-Support, wenn die Vignettierung weiterhin auftritt

### Problem: Falsche Farben oder Werte

**Mögliche Ursachen:**

* Keine Kalibrierungsziele erkannt
* Falsches Kalibrierungszielmodell ausgewählt
* Reflexionskalibrierung deaktiviert
* Zielbilder von schlechter Qualität

**Lösungen:**

1. Überprüfen Sie, ob die Reflektanzkalibrierung aktiviert war.
2. Überprüfen Sie die Meldungen „Ziel gefunden“ im Debug-Protokoll.
3. Überprüfen Sie die Qualität der Zielbilder.
4. Führen Sie die Verarbeitung erneut durch, nachdem Sie die richtigen Ziele markiert haben.

### Problem: NDVI-Werte scheinen falsch zu sein

**Erwartete NDVI-Bereiche:*** **Wasser, Felsen, Boden**: -0,1 bis 0,2
* **Spärliche/kranke Vegetation**: 0,2 bis 0,4
* **Mäßige Vegetation**: 0,4 bis 0,6
* **Gesunde, dichte Vegetation**: 0,6 bis 0,9**Falls die Werte außerhalb dieser Bereiche liegen:**

1. Überprüfen Sie, ob die Reflektanzkalibrierung angewendet wurde
2. Überprüfen Sie, ob das Lichtsensorprotokoll enthalten ist
3. Überprüfen Sie, ob Kalibrierungsziele erkannt wurden
4. Stellen Sie sicher, dass das richtige Kameramodell erkannt wurde
5. Überprüfen Sie den Zeitpunkt und die Bedingungen der Zielbildaufnahme
6. Wenn Sie Indizes selbst aus Reflektanzdateien berechnen, vergewissern Sie sich, dass Sie durch den Wert `Chloros:PixelScale` der Datei geteilt haben (siehe oben)

***

## Verwendung Ihrer verarbeiteten Bilder

### Für Photogrammetrie / Erstellung von Orthomosaiken

**Empfohlener Arbeitsablauf:**

1.**Importieren Sie kalibrierte Reflektanzbilder** in eine Photogrammetrie-Software:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Behalten Sie die EXIF-Metadaten bei**: Stellen Sie sicher, dass die GPS-Daten für die Geotag-Zuordnung erhalten bleiben
3. **Kalibrierte Arbeitsabläufe**: Verwenden Sie Reflektanzbilder für wissenschaftliche Genauigkeit – LATTICE-Reflektanzbilder enthalten die XMP-Kalibrierungs-Tags, die Pix4D liest
4. **Indexmosaike verarbeiten**: Erstellen Sie NDVI-Orthomosaike aus einzelnen Indexbildern
5. **Georeferenzierte GeoTIFF-Dateien exportieren**: Zur Verwendung in GIS-Anwendungen

### Für die GIS-Analyse

**Empfohlener Arbeitsablauf:**

1.**In QGIS, ArcGIS oder ähnliche Programme laden**

2.**Verwenden Sie 16-Bit-Reflexionsbilder (TIFF)** für die Multiband-Analyse (durch den Wert der Datei `Chloros:PixelScale` dividieren)
3. **Verwenden Sie Indexbilder** (NDVI, NDRE) als gebrauchsfertige Vegetationsebenen
4. **Rasterrechner**: Bänder für benutzerdefinierte Analysen kombinieren
5. **Exportieren**: Klassifizierungskarten, Änderungserkennung und Karten zum Vegetationszustand erstellen

### Für direkte Analyse / Berichterstellung

**Empfohlener Arbeitsablauf:**

1.**Verwenden Sie Indexbilder mit LUT-Farben** für visuelle Berichte
2. **Statistiken extrahieren**: Mittelwert von NDVI pro Feld/Parzelle
3. **Zeitreihen**: Indizes über mehrere Erfassungszeiträume hinweg vergleichen
4. **Berichte erstellen**: Karten, Statistiken und Visualisierungen einbinden***

## Archivierung und Sicherung

### Empfohlene Sicherungsstrategie

**Was gesichert werden sollte:*** ✅ **Original-RAW-/JPG-Bilder oder LATTICE-Rohaufnahmen** – Auf einem separaten Laufwerk/in der Cloud archivieren; die Rohdaten sind die Quelle der Pipeline, und alles andere kann daraus neu generiert werden
* ✅ **`.daq` / `.csv`-Lichtsensordateien** – werden benötigt, um die Reflektanz später erneut abzuleiten
* ✅ **Verarbeitete Ergebnisse** – kalibrierte Bilder und Indizes aufbewahren
* ✅ **Projektordner** (`project.json` und zugehörige Dateien) – Enthält alle Einstellungen für eine eventuelle erneute Verarbeitung
* ✅ **Debug-Protokoll** – Dokumentiert Details der Verarbeitung
* ✅ **Kalibrierungszielbilder** – Zur Überprüfung und erneuten Verarbeitung**Empfehlungen zur Speicherung:*** **Sofortige Sicherung**: Externe Festplatte
* **Langzeitarchiv**: Cloud-Speicher (Google Drive, Dropbox usw.)
* **Kritische Daten**: Bewahren Sie 2–3 Kopien an verschiedenen Orten auf***

## Nächste Verarbeitungsläufe

### Wiederverwendung von Projekteinstellungen

Falls Sie in Zukunft ähnliche Datensätze verarbeiten möchten:

1. **Projekvorlage speichern** (falls noch nicht geschehen)
2. **Neues Projekt erstellen** unter Verwendung der gespeicherten Vorlage
3. **Neue Bilder importieren**

4.**Verarbeiten**Sie die Bilder mit identischen Einstellungen, um Konsistenz zu gewährleisten

### Stapelverarbeitung mehrerer Sitzungen

Bei mehreren Sitzungen/Datensätzen:**Option 1: GUI – Mehrere Projekte**

* Erstellen Sie für jede Sitzung ein separates Projekt
* Verwenden Sie einheitliche Vorlageneinstellungen
* Verarbeiten Sie die Projekte nacheinander

**Option 2: Chloros CLI (nur Chloros+)**

* Stapelverarbeitung automatisieren
* Mehrere Ordner mit Skripten verarbeiten
* Siehe [CLI-Dokumentation](../CLI.md) und die [CLI-Referenz](../reference/cli-reference.md)

**Option 3: Python SDK (nur Chloros+)**

* Programmgesteuerte Steuerung
* Integration in Analyse-Pipelines
* Siehe [API-Dokumentation](../api-python-sdk.md) und die [SDK-Referenz](../reference/sdk-reference.md)

***

## Fehlerbehebung bei der Nachbearbeitung

### Erneute Verarbeitung mit anderen Einstellungen

Wenn die Ergebnisse nicht zufriedenstellend sind:

1. Bewahren Sie die Originalbilder auf (niemals löschen)
2. Öffnen Sie dasselbe Projekt in Chloros
3. Passen Sie die Einstellungen im Bereich „Projekteinstellungen“ an
4. Führen Sie die Verarbeitung erneut durch – die Ergebnisse werden in denselben Produktordnern abgelegt, sodass Dateien mit demselben Namen aus dem vorherigen Durchlauf ersetzt werden

### Verarbeitung einer Teilmenge von Bildern

So verarbeiten Sie nur bestimmte Bilder erneut:

1. Erstellen Sie ein neues Projekt
2. Importieren Sie nur die Bilder, die erneut verarbeitet werden müssen
3. Verwenden Sie dieselbe Einstellungsvorlage
4. Verarbeiten Sie den kleineren Datensatz

### Hilfe erhalten

Falls Probleme auftreten:

* 📧 **E-Mail**: info@mapir.camera (bitte Debug-Protokoll beifügen)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Häufig gestellte Fragen](../faq.md)
* 📖 **Dokumentation**: [Chloros-Handbuch](../)***

## Zusammenfassung: Vollständiger Arbeitsablauf

Sie haben nun den gesamten Chloros-Verarbeitungsworkflow abgeschlossen:

1. ✅ **Projekt erstellt** – Siehe [Projekte](../projects.md)
2. ✅ **Dateien hinzugefügt** – Siehe [Dateien hinzufügen](adding-files-to-a-project.md)
3. ✅ **Einstellungen angepasst** – Siehe [Projekteinstellungen anpassen](adjusting-project-settings.md)
4. ✅ **Ziele markiert** – Siehe [Zielbilder auswählen](choosing-target-images.md)
5. ✅ **Verarbeitung gestartet** – Siehe [Verarbeitung starten](starting-the-processing.md)
6. ✅ **Fortschritt überwacht** – Siehe [Verarbeitung überwachen](monitoring-the-processing.md)
7. ✅ **Ergebnisse überprüft** – Diese Seite**Ihre kalibrierten, reflektanzkorrigierten Multispektralbilder sind bereit für die Analyse!**

***

## Weitere Ressourcen

### Erweiterte Funktionen

* [**Bildbetrachter**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktive Visualisierung und Analyse
* [**Index-/LUT-Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testen benutzerdefinierter Indizes
* [**Multispektrale Indexformeln**](../project-settings/multispectral-index-formulas.md) – Vollständiges Index-Nachschlagewerk

### Automatisierung &amp; Integration

* [**CLI-Dokumentation**](../CLI.md) – Batch-Verarbeitung über die Befehlszeile
* [**Python SDK**](../api-python-sdk.md) – Programmgesteuerte Automatisierung
* [**Chloros+ Funktionen**](../#chloros) – Erweiterte Verarbeitungsmöglichkeiten

### Support &amp; Schulungen

* [**FAQ**](../faq.md) – Antworten auf häufig gestellte Fragen
* [**Kalibrierungsziele**](../calibration-targets.md) – Grundlagen der Reflexionskalibrierung
* [**Unterstützte Kameras**](../supported-cameras.md) – Kompatible Hardware
