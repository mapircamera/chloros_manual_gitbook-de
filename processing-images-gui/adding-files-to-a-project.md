# Hinzufügen von Dateien zu einem Projekt

Nachdem Sie in Chloros ein Projekt erstellt oder geöffnet haben, müssen Sie im nächsten Schritt Ihre Multispektralbilder hinzufügen, um mit der Verarbeitung zu beginnen. Über die Registerkarte „Dateibrowser“<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> erleichtert das Importieren von Bildern und die Verwaltung Ihres Datensatzes.

## Aufrufen des Dateibrowsers

1. Öffnen oder erstellen Sie ein Projekt in Chloros
2. Klicken Sie auf das Symbol **Dateibrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> in der linken Seitenleiste
3. Im Dateibrowser-Fenster wird die Dateiliste Ihres Projekts angezeigt

{% hint style="info" %}
**Unterstützte Dateiformate**: Chloros unterstützt RAW+JPG- und JPG-Bilddateien von den Kameras MAPIR, Survey3W und Survey3N. Es werden ausschließlich RAW+JPG-Dateien empfohlen.
{% endhint %}

***

## Bilder zu Ihrem Projekt hinzufügen

Es gibt zwei Hauptmethoden, um Bilder zu Ihrem Projekt hinzuzufügen:

### Methode 1: Dateien hinzufügen

Verwenden Sie diese Option, um einzelne Bilddateien oder eine kleine Auswahl an Dateien zu importieren.

1. Klicken Sie auf die Schaltfläche **„Dateien hinzufügen“** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> oben im Dateibrowser-Fenster
2. Navigieren Sie zu dem Ordner, der Ihre Bilder enthält
3. Wählen Sie eine oder mehrere Bilddateien aus (halten Sie **Strg** gedrückt, um mehrere Dateien auszuwählen)
4. Klicken Sie auf **„Öffnen“**, um die ausgewählten Dateien zu importieren

### Methode 2: Ordner hinzufügen

Verwenden Sie diese Option, um alle Bilder aus einem Ordner auf einmal zu importieren.

1. Klicken Sie auf die Schaltfläche **„Ordner hinzufügen“** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> oben im Dateibrowser-Fenster
2. Navigieren Sie zu dem Ordner, der Ihre Aufnahmen enthält, und wählen Sie ihn aus
3. Klicken Sie auf **„Ordner auswählen“**, um alle unterstützten Bilder aus diesem Ordner zu importieren***

## Die Tabelle im Dateibrowser verstehen

Sobald die Bilder importiert sind, werden sie in einer Tabelle mit den folgenden Spalten angezeigt:

### Dateiname

* Ursprünglicher Dateiname der Kamera
* Behält die Namenskonvention der Kamera bei (z. B. IMG\_0001.RAW)

### Zeitstempel

* Datum und Uhrzeit der Bildaufnahme
* Aus den EXIF-Metadaten des Bildes extrahiert
* Wird für die PPK-Synchronisation und die Erkennung von Kalibrierungsmarken verwendet

### Kameramodell

* Automatisch erkannte Kamera- und Filterkonfiguration
* Beispiele: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Wird verwendet, um die richtigen Verarbeitungsprofile anzuwenden

### Zielspalte (Kontrollkästchen)

* Aktivieren Sie dieses Kontrollkästchen für Bilder, die Kalibrierungsziele enthalten
* Beschleunigt die Zielerkennung während der Verarbeitung erheblich
* Weitere Informationen finden Sie unter [Auswahl von Zielbildern](choosing-target-images.md)

### Anzeigen von Bildmetadaten

Durch Klicken auf die Umschaltfläche in der oberen rechten Ecke über der Tabelle werden die Metadaten des ausgewählten Bildes im Bildrasterbereich angezeigt.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Verwalten von Dateien in Ihrem Projekt

### Dateien entfernen

So entfernen Sie unerwünschte Bilder aus Ihrem Projekt:

1. Wählen Sie ein oder mehrere Bilder in der Tabelle des Dateibrowsers aus
2. Klicken Sie auf die Schaltfläche **„Ausgewählte entfernen“** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> Schaltfläche
3. Bestätigen Sie das Entfernen (die Dateien werden nicht von der Festplatte gelöscht, sondern nur aus dem Projekt entfernt)

### Sortieren und Filtern

* **Nach Spalte sortieren**: Klicken Sie auf eine beliebige Spaltenüberschrift, um die Bilder zu sortieren
* **Sortierung nach Zeitstempel**: Nützlich zum Organisieren chronologischer Aufnahmesequenzen
* **Filter nach Kameramodell**: Gruppieren Sie Bilder nach Kameratyp, wenn Sie mehrere Kameras verwenden***

## Bildvorschau

### Vollbild anzeigen

Klicken Sie auf eine beliebige Bildminiatur im Dateibrowser, um sie im Hauptvorschau-Bereich anzuzeigen:

1. Das Bild erscheint im mittleren Vorschaufenster
2. Verwenden Sie die Zoom-Steuerelemente, um Bilddetails zu betrachten
3. Navigieren Sie mit den Pfeiltasten zwischen den Bildern

### Schnelle Navigation

* **Vorheriges Bild**: Klicken Sie auf den Pfeil nach links oder drücken Sie die Taste ←
* **Nächstes Bild**: Klicken Sie auf den Rechtspfeil oder drücken Sie die Taste →
* **Vergrößern/Verkleinern**: Verwenden Sie das Mausrad oder die Zoom-Schaltflächen
* **Schwenken**: Klicken und ziehen Sie auf dem Bild, wenn es vergrößert ist***

## Umgang mit doppelten Dateien

Chloros erkennt und ignoriert automatisch doppelte Dateien:

* Dateien mit identischen Dateinamen werden übersprungen
* Verhindert versehentliche doppelte Verarbeitung
* Warnmeldung wird angezeigt, wenn Duplikate erkannt werden

{% hint style="warning" %}
**Wichtig**: Benennen Sie Ihre Originalbilddateien vor dem Import nicht um und ändern Sie sie nicht. Chloros ist für die korrekte Verarbeitung auf die Originaldateinamen und Metadaten angewiesen.
{% endhint %}

***

## Gemischte Kameradaten

Wenn Ihr Projekt Bilder von mehreren MAPIR-Kameras enthält:

1. Chloros erkennt automatisch jedes Kameramodell
2. Jeder Kameratyp wird mit dem entsprechenden Kalibrierungsprofil verarbeitet
3. Der Dateibrowser zeigt das Kameramodell in der Spalte „Kameramodell“ an
4. Bei der Verarbeitung werden die richtigen Einstellungen für jeden Kameratyp angewendet

**Beispielszenario**: Survey3W RGN + Survey3N OCN Dual-Kamera-Konfiguration***

## Bewährte Verfahren

### Vor dem Import organisieren

* Speichern Sie Kalibrierungszielbilder im selben Ordner wie die Vermessungsbilder
* Behalten Sie die ursprüngliche Ordnerstruktur Ihrer Kamera/SD-Karte bei
* Mischen Sie keine Datensätze aus verschiedenen Sitzungen in einem Projekt

### Dateibenennung

* Behalten Sie die ursprünglichen Kameradateinamen bei (IMG\_0001.RAW usw.)
* Benennen Sie Dateien vor dem Import nicht um
* Die ursprünglichen Namen enthalten wichtige Metadaten

### Kalibrierungszielbilder

* Fügen Sie immer 1–2 Kalibrierungszielbilder pro Sitzung hinzu
* Nehmen Sie die Ziele vor und nach der Aufnahmesitzung auf
* Platzieren Sie die Ziele unter denselben Lichtbedingungen wie den Aufnahmebereich
* Markieren Sie Zielbilder mit dem Kontrollkästchen „Ziel“, um die Verarbeitung zu beschleunigen

***

## Häufige Probleme und Lösungen

### Bilder werden nach dem Import nicht angezeigt

**Mögliche Ursachen:**

* Dateiformat wird nicht unterstützt (nur RAW+JPG und JPG von MAPIR-Kameras)
* Bilder stammen von Nicht-MAPIR-Kameras (siehe [Unterstützte Kameras](../supported-cameras.md))
* Beschädigte Datei oder unvollständige Übertragung von der SD-Karte

**Lösung**: Überprüfen Sie die Kompatibilität von Dateiformat und Kameramodell

### Kameramodell nicht erkannt

**Mögliche Ursachen:**

* Geänderte EXIF-Metadaten
* Bilder wurden in externer Software bearbeitet
* Unvollständige Dateiübertragung

**Lösung**: Importieren Sie die ursprünglichen, unveränderten Dateien erneut von der Kamera/SD-Karte

### Fehlende Zeitstempel

**Mögliche Ursachen:**

* Kamera-Uhr nicht richtig eingestellt
* EXIF-Daten durch externe Software entfernt

**Lösung**: Überprüfen Sie, ob die Zeiteinstellungen der Kamera während der Aufnahme korrekt waren***

## Nächste Schritte

Sobald Ihre Dateien importiert sind:

1. **Überprüfen Sie die Dateiliste** – Stellen Sie sicher, dass alle Bilder korrekt geladen wurden
2. **Kameramodelle überprüfen** – Überprüfen Sie, ob die Kamera korrekt erkannt wurde
3. **Zielbilder markieren** – Siehe [Zielbilder auswählen](choosing-target-images.md)
4. **Einstellungen anpassen** – Konfigurieren Sie die Verarbeitungsoptionen in den [Projekteinstellungen](adjusting-project-settings.md)
5. **Verarbeitung starten** – Siehe [Verarbeitung starten](starting-the-processing.md)

Ausführliche Informationen zur Projektkonfiguration finden Sie unter [Projekteinstellungen anpassen](adjusting-project-settings.md).
