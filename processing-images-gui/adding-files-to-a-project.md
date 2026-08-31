# Dateien zu einem Projekt hinzufügen

Nachdem Sie ein Projekt in Chloros erstellt oder geöffnet haben, müssen Sie im nächsten Schritt Ihre Multispektralbilder hinzufügen, um mit der Verarbeitung zu beginnen. Über die Registerkarte „Dateibrowser“ <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> können Sie Bilder ganz einfach importieren und Ihren Datensatz verwalten.

## Auf den Dateibrowser zugreifen

1. Öffnen oder erstellen Sie ein Projekt in Chloros.
2. Klicken Sie in der linken Seitenleiste auf das Symbol **Dateibrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">.
3. Im Dateibrowser-Fenster wird die Dateiliste Ihres Projekts angezeigt.

{% hint style="info" %}
**Unterstützte Dateiformate**:

* **Survey3W / Survey3N**: RAW+JPG-Paare und JPG-Bilder (RAW+JPG empfohlen)
* **LATTICE**: `.tif` / `.tiff`-Aufnahmen – aufgezeichnet über die Chloros-Kamerasteuerung oder einen LATTICE-Hub
* **Lichtsensordaten**: `.daq`-Aufzeichnungen (DAQ-U/M/E) und DAQ-M `.csv`-Downwelling-Protokolle – zusammen mit Bildmaterial importiert, um die Reflektanzkalibrierung durchzuführen
{% endhint %}

***

## Hinzufügen von Bildern zu Ihrem Projekt

Es gibt zwei Hauptmethoden, um Bilder zu Ihrem Projekt hinzuzufügen:

### Methode 1: Dateien hinzufügen

Verwenden Sie diese Option, um einzelne Bilddateien oder eine kleine Auswahl von Dateien zu importieren.

1. Klicken Sie oben im Dateibrowser-Fenster auf die Schaltfläche **„Dateien hinzufügen“** (<img src="../.gitbook/assets/image (3).png" alt="" data-size="line">).
2. Navigieren Sie zu dem Ordner, der Ihre Bilder enthält.
3. Wählen Sie eine oder mehrere Bilddateien aus (halten Sie die **Strg**-Taste gedrückt, um mehrere Dateien auszuwählen).
4. Klicken Sie auf **„Öffnen“**, um die ausgewählten Dateien zu importieren.

### Methode 2: Ordner hinzufügen

Verwenden Sie diese Option, um alle Bilder aus einem Ordner auf einmal zu importieren. Sie können **mehrere Ordner** in einem Dialogfeld auswählen.

1. Klicken Sie oben im Dateibrowser-Fenster auf die Schaltfläche **„Ordner hinzufügen“** (<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">).
2. Navigieren Sie zu den Ordnern, die Ihre Bilder aus der Aufnahmesitzung enthalten, und wählen Sie diese aus.
3. Klicken Sie auf **„Ordner auswählen“**, um alle unterstützten Bilder zu importieren.

{% hint style="info" %}
**Dateien, die nicht geladen werden können, werden gemeldet.** Wenn ein Ordner Dateien enthält, die Chloros zwar erkennt, aber nicht laden kann, werden Sie durch eine Warnung darauf hingewiesen – Bilder verschwinden nicht unbemerkt aus der Tabelle.
{% endhint %}

***

## Importieren von LATTICE-Aufnahmeordnern

LATTICE-Aufnahmen werden mit **einem Unterordner pro Exportstufe** gespeichert – zum Beispiel `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` – mit der entsprechenden „downwelling“-Datei `.daq` im Stammverzeichnis:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Wählen Sie „Ordner hinzufügen“ im Stammverzeichnis der Aufnahmen** (`output/` oben). Wenn der ausgewählte Ordner selbst keine Bilder enthält, aber Unterordner hat, durchläuft Chloros diese automatisch – die Unterordner dieser Ebene und das Stammverzeichnis `.daq` werden alle auf einmal erfasst.**So werden Aufnahmen importiert:*** Jede Aufnahme wird als **einzelnes Bild** importiert, gruppiert nach Aufnahme (nicht ein Eintrag pro Ebene). Die anderen Ebenen derselben Aufnahme erscheinen als Ansichtsmodi dieses einen Bildes.
* **Die Verarbeitung beginnt immer mit dem Rohbild.** Die anderen Ebenen sind zwar sichtbar, aber nur „`raw`“ wird jemals durch die Pipeline geleitet – eine erneute Verarbeitung eines bereits verarbeiteten Produkts würde Korrekturen doppelt anwenden, daher wird „Chloros“ abgelehnt. Ein erneut importierter Export kann niemals den Rohdaten-Slot einer Aufnahme einnehmen.
* Ein Aufnahmeordner, der **ohne** Rohdaten gespeichert wurde, wird normal angezeigt, aber die Verarbeitung überspringt ihn und vermerkt dies im Protokoll. (Das Flag CLI kann in diesem Fall einen Einstiegspunkt erzwingen – siehe [die CLI-Referenz](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**LATTICE-Hub-Sitzungen** werden auf dieselbe Weise importiert: Wählen Sie unter „Add Folder“ den vom Hub kopierten Sitzungsordner aus (er enthält `raw/` sowie `previews/`) zusammen mit einem beliebigen DAQ-M-`.csv`-Downwelling-Protokoll. Falls die Kalibrierung der Kamera oder des DAQ noch nicht auf Ihrem Rechner zwischengespeichert ist, ruft Chloros diese beim Import automatisch anhand der Seriennummer ab (erfordert einmalig eine Internetverbindung).***

## Die Tabelle im Dateibrowser verstehen

Sobald die Bilder importiert sind, werden sie in einer Tabelle mit den folgenden Spalten angezeigt:

### Dateiname

* Ursprünglicher Dateiname der Kamera
* Behält die Namenskonvention der Kamera bei (z. B. IMG\_0001.RAW oder capture\_20260816\_101500\_SN213800234\_raw.tif)

### Zeitstempel

* Datum und Uhrzeit der Aufnahme
* Aus den EXIF-Metadaten des Bildes extrahiert
* Wird für den Lichtsensorabgleich, die PPK-Synchronisation und die Planung von Kalibrierungszielen verwendet

### Kameramodell

* Automatisch erkannte Kamera- und Filterkonfiguration
* Survey3-Beispiele: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* LATTICE-Beispiele: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Dient zur Anwendung der richtigen Verarbeitungsprofile

### Zielspalte (Kontrollkästchen)

* Aktivieren Sie dieses Kontrollkästchen für Bilder, die Kalibrierungsziele enthalten
* Wenn mindestens ein Bild markiert ist, werden **nur die markierten Bilder** nach Zielen durchsucht
* Weitere Informationen finden Sie unter [Auswahl von Zielbildern](choosing-target-images.md)

### Bildmetadaten anzeigen

Durch Klicken auf die Umschaltfläche in der oberen rechten Ecke über der Tabelle werden die Metadaten des ausgewählten Bildes im Bildrasterbereich angezeigt.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Lichtsensor-Dateien in Ihrem Projekt

* Die Dateien „`.daq`“ und „`.csv`“ erscheinen in der Liste des Dateibrowsers, sind jedoch keine anklickbaren Bilder – sie liefern die nach unten gerichtete Bestrahlungsstärke für die Reflektanzkalibrierung.
* Jede importierte `.daq`-/`.csv`-Datei wird unter **Projekteinstellungen → DAQ-Lichtsensor** aufgeführt, wo Sie die für jede Datei geltende Diffusorkappenkorrektur überprüfen können. Siehe [Anpassen der Projekteinstellungen](adjusting-project-settings.md).
* Aufzeichnungen, die Sie auf der Registerkarte **Lichtsensoren** vornehmen, werden automatisch zum geöffneten Projekt hinzugefügt – ein manueller Import ist nicht erforderlich.***

## Verwalten von Dateien in Ihrem Projekt

### Dateien entfernen

So entfernen Sie unerwünschte Bilder aus Ihrem Projekt:

1. Wählen Sie ein oder mehrere Bilder in der Tabelle des Dateibrowsers aus.
2. Klicken Sie auf die Schaltfläche **„Ausgewählte entfernen“** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">.
3. Bestätigen Sie den Vorgang (die Dateien werden nicht von der Festplatte gelöscht, sondern nur aus dem Projekt entfernt).

### Sortieren und Filtern

* **Nach Spalte sortieren**: Klicken Sie auf eine beliebige Spaltenüberschrift, um die Bilder zu sortieren
* **Sortierung nach Zeitstempel**: Nützlich zum Ordnen chronologischer Aufnahmesequenzen
* **Filter nach Kameramodell**: Gruppieren Sie Bilder nach Kameratyp, wenn Sie mehrere Kameras verwenden***

## Bildvorschau

### Vollbild anzeigen

Klicken Sie auf eine beliebige Bildminiaturansicht im Dateibrowser, um sie im Hauptvorschau-Bereich anzuzeigen:

1. Das Bild erscheint im zentralen Vorschaufenster
2. Verwenden Sie die Zoom-Steuerelemente, um Bilddetails zu betrachten
3. Navigieren Sie mit den Pfeiltasten zwischen den Bildern

### Schnelle Navigation

* **Vorheriges Bild**: Klicken Sie auf den Pfeil nach links oder drücken Sie die Taste ←
* **Nächstes Bild**: Klicken Sie auf den Pfeil nach rechts oder drücken Sie die Taste →
* **Vergrößern/Verkleinern**: Verwenden Sie das Mausrad oder die Zoom-Schaltflächen
* **Verschieben**: Klicken und ziehen Sie auf dem Bild, wenn es vergrößert ist***

## Umgang mit doppelten Dateien

Chloros erkennt doppelte Dateien automatisch und ignoriert sie:

* Dateien mit identischen Dateinamen werden übersprungen
* Verhindert versehentliche doppelte Verarbeitung
* Bei Erkennung von Duplikaten wird eine Warnmeldung angezeigt

{% hint style="warning" %}
**Wichtig**: Benennen Sie Ihre Originalbilddateien vor dem Import nicht um und nehmen Sie keine Änderungen daran vor. Chloros ist für die korrekte Verarbeitung auf die Originaldateinamen und Metadaten angewiesen.
{% endhint %}

***

## Gemischte Kameradatensätze

Wenn Ihr Projekt Bilder von mehreren MAPIR-Kameras enthält:

1. Chloros erkennt automatisch jedes Kameramodell – Survey3, LATTICE oder eine Mischung
2. Jeder Kameratyp wird mit dem entsprechenden Kalibrierungsprofil verarbeitet
3. Der Dateibrowser zeigt das Kameramodell in der Spalte „Kameramodell“ an
4. Jede Kamera erhält bei der Verarbeitung eine eigene Ausgabeordnerstruktur

**Beispielszenarien**: Survey3W RGN + Survey3N OCN-Konfiguration mit zwei Kameras, oder ein LATTICE-Array mit einem RGB-Master und mehreren Schmalbandmodulen***

## Bewährte Vorgehensweisen

### Vor dem Import organisieren

* Speichern Sie die Kalibrierungszielbilder im selben Ordner wie die Vermessungsbilder
* Speichern Sie die Lichtsensordateien `.daq` / `.csv` jeder Aufnahmesitzung zusammen mit den Bildern dieser Sitzung
* Behalten Sie die ursprüngliche Ordnerstruktur Ihrer Kamera/SD-Karte/des Hubs bei
* Mischen Sie keine Datensätze aus verschiedenen Aufnahmesitzungen in einem Projekt

### Dateibenennung

* Behalten Sie die ursprünglichen Kameradateinamen bei (IMG\_0001.RAW, capture\_..., usw.)
* Benennen Sie Dateien vor dem Import nicht um
* Die ursprünglichen Namen enthalten wichtige Metadaten

### Kalibrierungszielbilder

* Fügen Sie pro Sitzung immer 1–2 Kalibrierungszielbilder hinzu (Survey3; bei LATTICE kann eine DAQ-Aufzeichnung als Ersatz dienen – siehe [Auswahl von Zielbildern](choosing-target-images.md))
* Nehmen Sie die Zielbilder vor und nach der Aufnahmesitzung auf
* Platzieren Sie die Zielbilder unter denselben Lichtbedingungen wie den Aufnahmebereich
* Markieren Sie die Zielbilder mit dem Kontrollkästchen „Ziel“

***

## Häufige Probleme und Lösungen

### Bilder werden nach dem Import nicht angezeigt

**Mögliche Ursachen:**

* Dateiformat wird nicht unterstützt (siehe Liste der unterstützten Dateiformate oben auf dieser Seite)
* Die Bilder stammen von Kameras, die nicht mit MAPIR kompatibel sind (siehe [Unterstützte Kameras](../supported-cameras.md))
* Beschädigte Datei oder unvollständige Übertragung von der SD-Karte

**Lösung**: Überprüfen Sie das Dateiformat und die Kompatibilität des Kameramodells und sehen Sie sich die Warnmeldung beim Laden der Dateien an, um festzustellen, bei welchen Dateien genau der Fehler aufgetreten ist

### Kameramodell nicht erkannt

**Mögliche Ursachen:**

* Geänderte EXIF-Metadaten
* In externer Software bearbeitete Bilder
* Unvollständige Dateiübertragung

**Lösung**: Importieren Sie die ursprünglichen, unveränderten Dateien erneut von der Kamera/SD-Karte.

### Fehlende Zeitstempel

**Mögliche Ursachen:**

* Die Uhr der Kamera ist nicht richtig eingestellt
* EXIF-Daten wurden durch externe Software entfernt

**Lösung**: Überprüfen Sie, ob die Zeiteinstellungen der Kamera während der Aufnahme korrekt waren

### Beim erneuten Öffnen des Projekts werden fehlende Dateien gemeldet

Wenn Quelldateien seit dem letzten Öffnen des Projekts verschoben oder gelöscht wurden, zeigt Ihnen Chloros an, **welche** Dateien fehlen, anstatt ein leeres Raster zu öffnen. Stellen Sie die Dateien an ihren ursprünglichen Pfaden wieder her oder entfernen Sie die fehlenden Einträge und importieren Sie die Dateien erneut.***

## Nächste Schritte

Sobald Ihre Dateien importiert sind:

1. **Überprüfen Sie die Dateiliste** – Stellen Sie sicher, dass alle Bilder korrekt geladen wurden
2. **Überprüfen Sie die Kameramodelle** – Vergewissern Sie sich, dass die Kameras korrekt erkannt wurden
3. **Zielbilder markieren** – Siehe [Zielbilder auswählen](choosing-target-images.md)
4. **Einstellungen anpassen** – Konfigurieren Sie die Verarbeitungsoptionen in den [Projekteinstellungen](adjusting-project-settings.md)
5. **Verarbeitung starten** – Siehe [Verarbeitung starten](starting-the-processing.md)

Ausführliche Informationen zur Projektkonfiguration finden Sie unter [Projekteinstellungen anpassen](adjusting-project-settings.md).
