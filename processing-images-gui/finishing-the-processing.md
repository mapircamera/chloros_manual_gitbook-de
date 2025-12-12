# Abschluss der Verarbeitung

Sobald Chloros die Verarbeitung abgeschlossen hat, ist es an der Zeit, Ihre Ergebnisse zu überprüfen, die Ausgabequalität zu verifizieren und Ihre verarbeiteten Bilder für die Verwendung in Ihrem Workflow vorzubereiten. Diese Seite führt Sie durch die letzten Schritte und die nächsten Aktionen.

## Anzeige „Verarbeitung abgeschlossen”

Wenn die Verarbeitung erfolgreich abgeschlossen wurde, werden mehrere Anzeigen angezeigt:

* ✅ **Fortschrittsbalken**: Erreicht 100 % Fertigstellung
* ✅ **Debug-Protokoll**: Zeigt die Meldung „Verarbeitung abgeschlossen” an
* ✅ **Start-Schaltfläche**: Wird wieder aktiviert (bereit für den nächsten Verarbeitungslauf)
* ✅ **Ausgabedateien**: Alle verarbeiteten Bilder werden im Unterordner des Kameramodells gespeichert

***

## Auffinden Ihrer verarbeiteten Bilder

### Öffnen des Ausgabeordners

1. Klicken Sie auf das Symbol **Hauptmenü** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (oben links)
2. Wählen Sie **„Projektordner öffnen”**
3. Ihr Datei-Explorer öffnet sich im Projektverzeichnis
4. Suchen Sie Ihr Projekt anhand des Namens

***

## Bearbeitete Bilder überprüfen

### Schnellvorschau im Datei-Explorer

**Windows integrierte Vorschau:**

1. Navigieren Sie zum Unterordner des Kameramodells.
2. Wählen Sie eine Bilddatei aus.
3. Die Vorschau wird im Vorschaufenster des Windows Explorers angezeigt.
4. Verwenden Sie die Pfeiltasten, um durch die Bilder zu blättern.

### Vorschau in externen Bildbetrachtern

**Empfohlene Bildbetrachter:**

* **QGIS** – Kostenlose GIS-Software (am besten geeignet für georeferenzierte Multispektralanalyse)
* **IrfanView** – Schneller, schlanker Bildbetrachter (unterstützt TIFF)
* **Adobe Photoshop** – Professionelle Bearbeitung (unterstützt TIFF)
* **GIMP** – Kostenlose Alternative zu Photoshop
* **Windows Photos** – Grundlegende Anzeige (unterstützt möglicherweise kein 16-Bit-TIFF)

### Vorschau im Chloros-Bildbetrachter

Verwenden Sie den integrierten Bildbetrachter von Chloros für eine erweiterte Visualisierung:

1. Klicken Sie im Dateibrowser auf eine Bildminiaturansicht.
2. Das Bild wird im Hauptvorschau-Bereich geöffnet.
3. Klicken Sie auf die Registerkarte **Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> in der linken Seitenleiste.
4. Verwenden Sie [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) für die interaktive Analyse.

Ausführliche Anweisungen finden Sie unter [Bildbetrachter](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Debug-Protokoll überprüfen

### Auf Warnungen oder Fehler prüfen

1. Öffnen Sie die Registerkarte **Debug-Protokoll** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> .
2. Scrollen Sie durch die Meldungen.
3. Achten Sie auf gelbe Warnungen oder rote Fehler.
4. Überprüfen Sie alle aufgeführten Probleme.
5. Wenden Sie sich an den Support von MAPIR, um Hilfe zu erhalten.

### Speichern des Protokolls

Um eine Aufzeichnung der Verarbeitung zu speichern oder an den Support von MAPIR zu senden:

1. Klicken Sie auf die Schaltfläche **„Kopieren”** oder **„Herunterladen”**.
2. Speichern Sie das Protokoll als Textdatei im Projektordner.
3. Fügen Sie es der Projektdokumentation bei.
4. Senden Sie es bei Problemen an den MAPIR-Support.

***

## Häufige Probleme bei der Ausgabe und Lösungen

### Problem: Fehlende Ausgabedateien

**Mögliche Ursachen:**

* Dateien erfüllten die Verarbeitungskriterien nicht.
* Nur Zielbilder (vom Export ausgeschlossen).
* Der Speicherplatz auf der Festplatte war während des Exports erschöpft.
* Dateibeschädigung während der Verarbeitung.

**Lösungen:**

1. Debug-Protokoll auf Überspring-/Fehlermeldungen überprüfen.
2. Überprüfen, ob ausreichend Speicherplatz auf der Festplatte vorhanden war.
3. Dateien zählen: Sollte übereinstimmen mit (ursprüngliche Anzahl – Zielanzahl) × (Indizes + 1)
4. Importieren Sie fehlende Dateien erneut und verarbeiten Sie sie erneut.

### Problem: Dunkle oder helle Ränder (Vignettierung noch sichtbar)

**Mögliche Ursachen:**

* Vignettierungskorrektur deaktiviert
* Kamera/Objektiv nicht in der Chloros-Profildatenbank enthalten
* Extreme Vignettierung, die über die Korrekturfähigkeit hinausgeht

**Lösungen:**

1. Überprüfen Sie, ob die Vignettierungskorrektur in den Projekteinstellungen aktiviert wurde.
2. Überprüfen Sie, ob das Kameramodell korrekt erkannt wurde.
3. Wenden Sie sich an den MAPIR-Support, wenn die Vignettierung weiterhin besteht.

### Problem: Falsche Farben oder Werte

**Mögliche Ursachen:**

* Keine Kalibrierungsziele erkannt.
* Falsches Kalibrierungszielmodell ausgewählt.
* Reflektionskalibrierung deaktiviert.
* Zielbilder von schlechter Qualität.

**Lösungen:**

1. Überprüfen Sie, ob die Reflektionskalibrierung aktiviert ist.
2. Überprüfen Sie die Meldungen „Ziel gefunden” im Debug-Protokoll.
3. Überprüfen Sie die Qualität der Zielbilder.
4. Führen Sie die Verarbeitung mit den richtigen markierten Zielen erneut durch.

### Problem: Die NDVI-Werte scheinen falsch zu sein.

**Erwartete NDVI-Bereiche:**

* **Wasser, Felsen, Erde**: -0,1 bis 0,2
* **Spärliche/ungesunde Vegetation**: 0,2 bis 0,4
* **Mäßige Vegetation**: 0,4 bis 0,6
* **Gesunde, dichte Vegetation**: 0,6 bis 0,9

**Wenn die Werte außerhalb dieser Bereiche liegen:**

1. Überprüfen Sie, ob die Reflektionskalibrierung angewendet wurde.
2. Überprüfen Sie, ob das Licht-Sensor-Protokoll enthalten ist.
3. Überprüfen Sie, ob die Kalibrierungsziele erkannt wurden.
4. Stellen Sie sicher, dass das richtige Kameramodell erkannt wurde.
5. Überprüfen Sie den Zeitpunkt und die Bedingungen der Zielbildaufnahme.

***

## Verwendung Ihrer verarbeiteten Bilder

### Für Photogrammetrie/Orthomosaik-Erstellung

**Empfohlener Arbeitsablauf:**

1. **Importieren Sie kalibrierte Reflexionsbilder** in die Photogrammetrie-Software:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Behalten Sie die EXIF-Metadaten bei**: Stellen Sie sicher, dass die GPS-Daten für die Geotagging-Funktion erhalten bleiben.
3. **Kalibrierte Arbeitsabläufe**: Verwenden Sie Reflexionsbilder für wissenschaftliche Genauigkeit.
4. **Verarbeiten Sie Indexmosaike**: Erstellen Sie NDVI-Orthomosaike aus einzelnen Indexbildern
5. **Exportieren Sie georeferenzierte GeoTIFF**: Zur Verwendung in GIS-Anwendungen

### Für die GIS-Analyse

**Empfohlener Arbeitsablauf:**

1. **In QGIS, ArcGIS oder ähnliches laden**
2. **Verwenden Sie 16-Bit-TIFF**-Reflexionsbilder für die Multiband-Analyse
3. **Verwenden Sie Indexbilder** (NDVI, NDRE) als gebrauchsfertige Vegetationsschichten
4. **Rasterrechner**: Kombinieren Sie Bänder für benutzerdefinierte Analysen
5. **Exportieren**: Erstellen Sie Klassifizierungskarten, Änderungserkennung und Vegetationsgesundheitskarten.

### Für direkte Analyse/Berichterstellung

**Empfohlener Arbeitsablauf:**

1. **Verwenden Sie Indexbilder mit LUT-Farben** für visuelle Berichte.
2. **Extrahieren Sie Statistiken**: Mittelwert NDVI pro Feld/Parzelle.
3. **Zeitreihen**: Indizes über mehrere Sitzungen hinweg vergleichen
4. **Berichte erstellen**: Karten, Statistiken und Visualisierungen einbeziehen

***

## Archivierung und Sicherung

### Empfohlene Sicherungsstrategie

**Was zu speichern ist:**

* ✅ **Original-RAW/JPG-Bilder** – Auf separatem Laufwerk/in der Cloud archivieren
* ✅ **Verarbeitete Ausgaben** – Kalibrierte Bilder und Indizes aufbewahren
* ✅ **Projektdatei** – Enthält alle Einstellungen für eine erneute Verarbeitung, falls erforderlich
* ✅ **Debug-Protokoll** – Dokumentiert Details zur Verarbeitung
* ✅ **Kalibrierungszielbilder** – Zur Überprüfung und erneuten Verarbeitung

**Empfehlungen zur Speicherung:**

* **Sofortige Sicherung**: Externe Festplatte
* **Langzeitarchivierung**: Cloud-Speicher (Google Drive, Dropbox usw.)
* **Kritische Daten**: 2–3 Kopien an verschiedenen Orten aufbewahren

***

## Nächste Verarbeitungsläufe

### Wiederverwendung von Projekteinstellungen

Wenn Sie in Zukunft ähnliche Datensätze verarbeiten möchten:

1. **Projektvorlage speichern** (falls noch nicht geschehen)
2. **Neues Projekt erstellen** unter Verwendung der gespeicherten Vorlage
3. **Neue Bilder importieren**
4. **Verarbeiten** mit identischen Einstellungen für Konsistenz

### Stapelverarbeitung mehrerer Sitzungen

Für mehrere Sitzungen/Datensätze:

**Option 1: GUI – Mehrere Projekte**

* Erstellen Sie für jede Sitzung ein separates Projekt.
* Verwenden Sie konsistente Vorlageneinstellungen.
* Verarbeiten Sie jeweils nur eine Sitzung.

**Option 2: Chloros CLI (nur Chloros+)**

* Automatisieren Sie die Stapelverarbeitung.
* Verarbeiten Sie mehrere Ordner mit Skripten.
* Siehe [CLI-Dokumentation](../CLI.md)

**Option 3: Python SDK (nur Chloros+)**

* Programmgesteuerte Steuerung
* Integration in Analyse-Pipelines
* Siehe [API-Dokumentation](../api-python-sdk.md)

***

## Fehlerbehebung bei der Nachbearbeitung

### Neubearbeitung mit anderen Einstellungen

Wenn die Ergebnisse nicht zufriedenstellend sind:

1. Behalten Sie die Originalbilder (niemals löschen).
2. Öffnen Sie dasselbe Projekt in Chloros.
3. Passen Sie die Einstellungen im Fenster „Projekteinstellungen“ an.
4. Führen Sie die Verarbeitung erneut durch – die Ergebnisse überschreiben die vorherigen Ergebnisse.

### Verarbeitung einer Teilmenge von Bildern

So verarbeiten Sie nur bestimmte Bilder erneut:

1. Erstellen Sie ein neues Projekt.
2. Importieren Sie nur die Bilder, die erneut verarbeitet werden müssen.
3. Verwenden Sie dieselbe Einstellungsvorlage.
4. Verarbeiten Sie einen kleineren Datensatz.

### Hilfe

Wenn Sie auf Probleme stoßen:

* 📧 **E-Mail**: info@mapir.camera (Debug-Protokoll beifügen)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Häufig gestellte Fragen](../faq.md)
* 📖 **Dokumentation**: [Chloros-Handbuch](../)

***

## Zusammenfassung: Vollständiger Workflow

Sie haben nun den gesamten Chloros-Verarbeitungs-Workflow abgeschlossen:

1. ✅ **Projekt erstellt** – Siehe [Projekte](../projects.md)
2. ✅ **Dateien hinzugefügt** – Siehe [Dateien hinzufügen](adding-files-to-a-project.md)
3. ✅ **Einstellungen angepasst** – Siehe [Anpassen der Projekteinstellungen](adjusting-project-settings.md)
4. ✅ **Ziele markiert** – Siehe [Auswählen der Zielbilder](choosing-target-images.md)
5. ✅ **Verarbeitung gestartet** – Siehe [Verarbeitung starten](starting-the-processing.md)
6. ✅ **Fortschritt überwacht** – Siehe [Verarbeitung überwachen](monitoring-the-processing.md)
7. ✅ **Ergebnisse überprüft** – Diese Seite

**Ihre kalibrierten, reflektionskorrigierten Multispektralbilder sind nun bereit für die Analyse!**

***

## Weitere Ressourcen

### Erweiterte Funktionen

* [**Bildbetrachter**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktive Visualisierung und Analyse
* [**Index/LUT-Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Benutzerdefinierte Indexprüfung
* [**Multispektrale Indexformeln**](../project-settings/multispectral-index-formulas.md) – Vollständige Indexreferenz

### Automatisierung und Integration

* [**CLI-Dokumentation**](../CLI.md) – Befehlszeilen-Stapelverarbeitung
* [**Python SDK**](../api-python-sdk.md) – Programmatische Automatisierung
* [**Chloros+ Funktionen**](../#chloros) – Erweiterte Verarbeitungsfunktionen

### Support &amp; Lernen

* [**FAQ**](../faq.md) – Antworten auf häufig gestellte Fragen
* [**Kalibrierungsziele**](../calibration-targets.md) – Reflektionskalibrierung verstehen
* [**Unterstützte Kameras**](../supported-cameras.md) – Kompatible Hardware
