# Abschluss der Verarbeitung

Sobald Chloros die Verarbeitung abgeschlossen hat, ist es an der Zeit, Ihre Ergebnisse zu überprüfen, die Ausgabequalität zu kontrollieren und Ihre verarbeiteten Bilder für die Verwendung in Ihrem Workflow vorzubereiten. Diese Seite führt Sie durch die letzten Schritte und die nächsten Maßnahmen.

## Anzeige „Verarbeitung abgeschlossen“

Wenn die Verarbeitung erfolgreich abgeschlossen ist, werden mehrere Anzeigen angezeigt:

* ✅ **Fortschrittsbalken**: Erreicht 100 % Fertigstellung
* ✅ **Debug-Protokoll**: Zeigt die Meldung „Verarbeitung abgeschlossen“ an
* ✅ **Start-Schaltfläche**: Wird wieder aktiviert (bereit für den nächsten Verarbeitungsdurchlauf)
* ✅ **Ausgabedateien**: Alle verarbeiteten Bilder werden im Unterordner des Kameramodells gespeichert***

## Ihre verarbeiteten Bilder finden

### Den Ausgabeordner öffnen

1. Klicken Sie auf das **Hauptmenü** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (oben links)
2. Wählen Sie **„Projektordner öffnen“**

3. Ihr Datei-Explorer öffnet sich im Projektverzeichnis
4. Suchen Sie Ihr Projekt anhand des Namens

***

## Überprüfen der verarbeiteten Bilder

### Schnellvorschau im Datei-Explorer

**Windows integrierte Vorschau:**

1. Navigieren Sie zum Unterordner des Kameramodells
2. Wählen Sie eine Bilddatei aus
3. Die Vorschau erscheint im Windows Explorer-Vorschaufenster
4. Blättern Sie mit den Pfeiltasten durch die Bilder

### Vorschau in externen Bildbetrachtern

**Empfohlene Bildbetrachter:*** **QGIS** – Kostenlose GIS-Software (ideal für georeferenzierte Multispektralanalyse)
* **IrfanView** – Schneller, ressourcenschonender Bildbetrachter (unterstützt TIFF)
* **Adobe Photoshop** – Professionelle Bildbearbeitung (TIFF-Unterstützung)
* **GIMP** – Kostenlose Alternative zu Photoshop
* **Windows Photos** – Einfache Bildanzeige (unterstützt möglicherweise kein 16-Bit-TIFF)

### Vorschau im Chloros-Bildbetrachter

Verwenden Sie den in Chloros integrierten Bildbetrachter für erweiterte Visualisierung:

1. Klicken Sie im Dateibrowser auf eine Bildminiaturansicht
2. Das Bild wird im Hauptvorschau-Bereich geöffnet
3. Klicken Sie auf die Registerkarte **Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> in der linken Seitenleiste
4. Verwenden Sie [Index/LUT-Sandbox](../image-viewer-gui/index-lut-sandbox.md) für die interaktive Analyse

Ausführliche Anweisungen finden Sie unter [Bildbetrachter](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Debug-Protokoll überprüfen

### Auf Warnungen oder Fehler prüfen

1. Öffnen Sie die Registerkarte **Debug-Protokoll** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> 2. Blättern Sie durch die Meldungen
3. Achten Sie auf gelbe Warnungen oder rote Fehler
4. Überprüfen Sie alle festgestellten Probleme
5. Wenden Sie sich an den MAPIR-Support, um Unterstützung zu erhalten

### Speichern des Protokolls

Um einen Nachweis der Verarbeitung zu führen oder das Protokoll an den MAPIR-Support zu senden:

1. Klicken Sie auf die Schaltfläche **„Kopieren“**oder**„Herunterladen“**

2. Speichern Sie das Protokoll als Textdatei im Projektordner
3. Fügen Sie es der Projektdokumentation bei
4. Senden Sie es an den MAPIR-Support, falls Probleme auftreten

***

## Häufige Probleme bei der Ausgabe und deren Lösungen

### Problem: Fehlende Ausgabedateien

**Mögliche Ursachen:**

* Dateien erfüllten die Verarbeitungskriterien nicht
* Nur-Ziel-Bilder (vom Export ausgeschlossen)
* Der Speicherplatz reichte während des Exports nicht aus
* Dateibeschädigung während der Verarbeitung

**Lösungen:**

1. Debug-Protokoll auf Überspring- oder Fehlermeldungen überprüfen
2. Überprüfen, ob ausreichend Speicherplatz vorhanden war
3. Dateien zählen: Die Anzahl sollte (ursprüngliche Anzahl – Zielanzahl) × (Indizes + 1) entsprechen
4. Fehlende Dateien erneut importieren und verarbeiten

### Problem: Dunkle oder helle Ränder (Vignettierung weiterhin sichtbar)

**Mögliche Ursachen:**

* Vignettierungskorrektur deaktiviert
* Kamera/Objektiv nicht in der Chloros-Profildatenbank enthalten
* Extreme Vignettierung, die die Korrekturkapazität übersteigt

**Lösungen:**

1. Überprüfen Sie, ob die Vignettierungskorrektur in den Projekteinstellungen aktiviert war
2. Überprüfen Sie, ob das Kameramodell korrekt erkannt wurde
3. Wenden Sie sich an den MAPIR-Support, wenn die Vignettierung weiterhin besteht

### Problem: Falsche Farben oder Werte

**Mögliche Ursachen:**

* Keine Kalibrierungsziele erkannt
* Falsches Kalibrierungszielmodell ausgewählt
* Reflexionskalibrierung deaktiviert
* Zielbilder von schlechter Qualität

**Lösungen:**

1. Überprüfen Sie, ob die Reflektanzkalibrierung aktiviert war
2. Überprüfen Sie die Meldungen „Ziel gefunden“ im Debug-Protokoll
3. Überprüfen Sie die Qualität der Zielbilder
4. Führen Sie die Verarbeitung erneut durch, nachdem Sie die richtigen Ziele markiert haben

### Problem: NDVI-Werte scheinen falsch zu sein

**Erwartete NDVI-Bereiche:*** **Wasser, Felsen, Boden**: -0,1 bis 0,2
* **Spärliche/ungesunde Vegetation**: 0,2 bis 0,4
* **Mäßige Vegetation**: 0,4 bis 0,6
* **Gesunde, dichte Vegetation**: 0,6 bis 0,9**Wenn Werte außerhalb dieser Bereiche liegen:**

1. Überprüfen Sie, ob die Reflektanzkalibrierung angewendet wurde
2. Überprüfen Sie, ob das Lichtsensorprotokoll enthalten war
3. Überprüfen Sie, ob Kalibrierungsziele erkannt wurden
4. Stellen Sie sicher, dass das richtige Kameramodell erkannt wurde
5. Überprüfen Sie den Zeitpunkt und die Bedingungen der Zielbildaufnahme

***

## Verwendung Ihrer verarbeiteten Bilder

### Für Photogrammetrie / Erstellung von Orthomosaiken

**Empfohlener Arbeitsablauf:**

1.**Importieren Sie kalibrierte Reflektanzbilder** in eine Photogrammetrie-Software:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Behalten Sie die EXIF-Metadaten**: Stellen Sie sicher, dass die GPS-Daten für die Geotagging-Funktion erhalten bleiben
3. **Kalibrierte Arbeitsabläufe**: Verwenden Sie Reflektanzbilder für wissenschaftliche Genauigkeit
4. **Verarbeiten Sie Indexmosaike**: Erstellen Sie NDVI-Orthomosaike aus einzelnen Indexbildern
5. **Exportieren Sie georeferenzierte GeoTIFF**: Zur Verwendung in GIS-Anwendungen

### Für die GIS-Analyse

**Empfohlener Arbeitsablauf:**

1.**In QGIS, ArcGIS oder ähnliche Programme laden**

2.**Verwenden Sie 16-Bit-Reflexionsbilder (TIFF)** für die Multiband-Analyse
3. **Verwenden Sie Indexbilder** (NDVI, NDRE) als gebrauchsfertige Vegetationsebenen
4. **Raster-Rechner**: Kombinieren Sie Bänder für benutzerdefinierte Analysen
5. **Exportieren**: Erstellen Sie Klassifizierungskarten, Erstellen Sie Änderungserkennungen und Karten zur Vegetationsgesundheit

### Für direkte Analyse / Berichterstellung

**Empfohlener Arbeitsablauf:**

1.**Verwenden Sie Indexbilder mit LUT-Farben** für visuelle Berichte
2. **Statistiken extrahieren**: Mittelwert von NDVI pro Feld/Parzelle
3. **Zeitreihen**: Vergleichen Sie Indizes über mehrere Sitzungen hinweg
4. **Erstellen Sie Berichte**: Fügen Sie Karten, Statistiken und Visualisierungen ein***

## Archivierung und Sicherung

### Empfohlene Sicherungsstrategie

**Was Sie speichern sollten:*** ✅ **Original-RAW-/JPG-Bilder** – Archivieren Sie diese auf einem separaten Laufwerk/in der Cloud
* ✅ **Verarbeitete Ergebnisse** – Kalibrierte Bilder und Indizes aufbewahren
* ✅ **Projektdatei** – Enthält alle Einstellungen für eine eventuelle erneute Verarbeitung
* ✅ **Debug-Protokoll** – Dokumentiert Details der Verarbeitung
* ✅ **Kalibrierungszielbilder** – Zur Überprüfung und erneuten Verarbeitung**Empfehlungen zur Speicherung:*** **Sofortige Sicherung**: Externe Festplatte
* **Langzeitarchiv**: Cloud-Speicher (Google Drive, Dropbox usw.)
* **Kritische Daten**: Bewahren Sie 2–3 Kopien an verschiedenen Orten auf***

## Nächste Verarbeitungsläufe

### Wiederverwendung von Projekteinstellungen

Wenn Sie in Zukunft ähnliche Datensätze verarbeiten:

1. **Projekvorlage speichern** (falls noch nicht geschehen)
2. **Neues Projekt erstellen** unter Verwendung der gespeicherten Vorlage
3. **Neue Bilder importieren**

4.**Verarbeiten**mit identischen Einstellungen zur Gewährleistung der Konsistenz

### Stapelverarbeitung mehrerer Sitzungen

Für mehrere Sitzungen/Datensätze:**Option 1: GUI – Mehrere Projekte**

* Erstellen Sie für jede Sitzung ein separates Projekt
* Verwenden Sie konsistente Vorlageneinstellungen
* Verarbeiten Sie jeweils nur ein Projekt

**Option 2: Chloros CLI (nur Chloros+)**

* Automatisieren Sie die Stapelverarbeitung
* Verarbeiten Sie mehrere Ordner mit Skripten
* Siehe [CLI-Dokumentation](../CLI.md)

**Option 3: Python SDK (nur Chloros+)**

* Programmgesteuerte Steuerung
* Integration in Analyse-Pipelines
* Siehe [API-Dokumentation](../api-python-sdk.md)

***

## Fehlerbehebung bei der Nachbearbeitung

### Erneute Verarbeitung mit anderen Einstellungen

Wenn die Ergebnisse nicht zufriedenstellend sind:

1. Bewahren Sie die Originalbilder auf (niemals löschen)
2. Öffnen Sie dasselbe Projekt in Chloros
3. Passen Sie die Einstellungen im Bereich „Projekteinstellungen“ an
4. Führen Sie die Verarbeitung erneut durch – die Ergebnisse überschreiben die vorherigen Ergebnisse

### Verarbeitung einer Teilmenge von Bildern

So verarbeiten Sie nur bestimmte Bilder erneut:

1. Erstellen Sie ein neues Projekt
2. Importieren Sie nur die Bilder, die erneut verarbeitet werden müssen
3. Verwenden Sie dieselbe Einstellungsvorlage
4. Verarbeiten Sie den kleineren Datensatz

### Hilfe erhalten

Wenn Probleme auftreten:

* 📧 **E-Mail**: info@mapir.camera (Debug-Protokoll beifügen)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Häufig gestellte Fragen](../faq.md)
* 📖 **Dokumentation**: [Chloros-Handbuch](../)***

## Zusammenfassung: Vollständiger Workflow

Sie haben nun den gesamten Chloros-Verarbeitungsworkflow abgeschlossen:

1. ✅ **Projekt erstellt** – Siehe [Projekte](../projects.md)
2. ✅ **Dateien hinzugefügt** – Siehe [Dateien hinzufügen](adding-files-to-a-project.md)
3. ✅ **Einstellungen angepasst** – Siehe [Projekteinstellungen anpassen](adjusting-project-settings.md)
4. ✅ **Ziele markiert** – Siehe [Zielbilder auswählen](choosing-target-images.md)
5. ✅ **Verarbeitung gestartet** – Siehe [Verarbeitung starten](starting-the-processing.md)
6. ✅ **Fortschritt überwacht** – Siehe [Überwachen der Verarbeitung](monitoring-the-processing.md)
7. ✅ **Ergebnisse überprüft** – Diese Seite**Ihre kalibrierten, reflektanzkorrigierten Multispektralbilder sind bereit für die Analyse!**

***

## Weitere Ressourcen

### Erweiterte Funktionen

* [**Bildbetrachter**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktive Visualisierung und Analyse
* [**Index/LUT-Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testen benutzerdefinierter Indizes
* [**Multispektrale Indexformeln**](../project-settings/multispectral-index-formulas.md) – Vollständige Indexreferenz

### Automatisierung &amp; Integration

* [**CLI Dokumentation**](../CLI.md) – Batch-Verarbeitung über die Befehlszeile
* [**Python SDK**](../api-python-sdk.md) – Programmatische Automatisierung
* [**Chloros+ Funktionen**](../#chloros) – Erweiterte Verarbeitungsfunktionen

### Support &amp; Schulungen

* [**FAQ**](../faq.md) – Antworten auf häufig gestellte Fragen
* [**Kalibrierungsziele**](../calibration-targets.md) – Grundlagen der Reflexionskalibrierung
* [**Unterstützte Kameras**](../supported-cameras.md) – Kompatible Hardware
