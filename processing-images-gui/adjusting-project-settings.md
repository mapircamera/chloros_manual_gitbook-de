# Projekteinstellungen anpassen

Bevor Sie Ihre Bilder bearbeiten, sollten Sie die Projekteinstellungen an die Anforderungen Ihres Arbeitsablaufs anpassen. Über das Fenster „Projekteinstellungen“ <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> bietet umfassende Steuerungsmöglichkeiten für Kalibrierung, Verarbeitungsoptionen, multispektrale Indizes und Exportformate.

## Aufrufen der Projekteinstellungen

1. Öffnen Sie Ihr Projekt in Chloros
2. Klicken Sie auf das Symbol **Projekteinstellungen** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in der linken Seitenleiste
3. Das Fenster „Projekteinstellungen“ zeigt alle Konfigurationsoptionen an

{% hint style="info" %}
**Die Einstellungen werden automatisch** mit Ihrem Projekt gespeichert. Wenn Sie ein Projekt erneut öffnen, werden alle Einstellungen wiederhergestellt.
{% endhint %}

***

## Schnelleinrichtung für gängige Arbeitsabläufe

### Standardeinstellungen (für die meisten Benutzer empfohlen)

Für typische MAPIR Survey3-Kamera-Workflows eignen sich die Standardeinstellungen gut:

* ✅ **Vignettenkorrektur**: Aktiviert
* ✅ **Reflexionskalibrierung**: Aktiviert (erfordert Bilder von MAPIR-Testmustern)
* ✅ **Debayer-Methode**: Standard (schnell, mittlere Qualität)
* ✅ **Exportformat**: TIFF (16-Bit)

Importieren Sie einfach Ihre Bilder und beginnen Sie die Verarbeitung mit diesen Standardeinstellungen.

***

## Übersicht über die Projekteinstellungen

Das Fenster „Projekteinstellungen“ ist in mehrere Kategorien unterteilt. Nachfolgend finden Sie eine Zusammenfassung der einzelnen Abschnitte. Die vollständige Dokumentation finden Sie unter [Projekteinstellungen](../project-settings/project-settings.md).

### Zielerkennung

Steuert, wie Chloros Kalibrierungsziele in Ihren Bildern identifiziert.

**Wichtige Einstellungen:*** **Mindestfläche für Kalibrierungsmuster**: Größenschwelle für die Zielerkennung (Standard: 25 Pixel)
* **Mindestclustergröße für Ziele**: Ähnlichkeitsschwelle für die Gruppierung von Zielbereichen (Standard: 60)**Wann anpassen:**

* Erhöhen Sie die Musterfläche, wenn falsche Erkennungen auftreten
* Verringern Sie ihn, wenn keine Ziele erkannt werden
* Passen Sie die Clusterbildung an, wenn Ziele in mehrere Erkennungen aufgeteilt werden

### Verarbeitung

Wichtige Optionen für die Bildverarbeitung und Kalibrierung.

**Wichtige Einstellungen:*** **Vignettierungskorrektur**: Kompensiert die Abdunkelung der Bildränder durch das Objektiv ✅ Empfohlen
* **Reflexionskalibrierung**: Normalisiert Werte mithilfe von Kalibrierungszielen ✅ Empfohlen
* **Debayer-Methode**: Algorithmus zur Konvertierung von RAW in 3-Kanal-Multispektral
* **Minimales Neukalibrierungsintervall**: Zeit zwischen der Verwendung von Kalibrierungszielen (0 = alle verwenden)**Erweiterte Einstellungen:*** **Zeitzonen-Offset des Lichtsensors**: Für die PPK-Zeitsynchronisation (Standard: 0)
* **PPK-Korrekturen anwenden**: Verwendet GPS-/Belichtungspin-Daten aus .daq-Dateien
* **Belichtungspin 1/2**: Weist Kameras Belichtungspins für Dual-Kamera-Konfigurationen zu

### Debayer-Methode

Wir bieten derzeit 2 Debayering-Methoden in Chloros an:

#### Standard (Schnell, mittlere Qualität)

Der Standard-Debayer verarbeitet schnell, zeigt jedoch Farbrauschen beim Debayering, was zu weniger genauen und verrauschteren Bildern führt.

#### Texture Aware (langsam, höchste Qualität) \[Nur Chloros+]

„Texture Aware“ verwendet einen hochwertigen, kantenbewussten Debayer in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt. Das „Texture Aware“-Modell benötigt zur Ausführung GPU-Speicher (VRAM). Wir empfehlen die Verwendung, wenn Sie über mehr als 4 GB VRAM verfügen, um eine schnellere Verarbeitung zu gewährleisten.

### Index (Multispektrale Indizes)

Konfigurieren Sie, welche Vegetationsindizes berechnet und exportiert werden sollen.

**So fügen Sie Indizes hinzu:**

1. Klicken Sie auf die Schaltfläche**„Index hinzufügen“**

2. Wählen Sie einen Index aus dem Dropdown-Menü aus (NDVI, NDRE, GNDVI usw.)
3. Konfigurieren Sie die Visualisierungseinstellungen (LUT-Farben, Wertebereiche)
4. Fügen Sie nach Bedarf mehrere Indizes hinzu

**Beliebte Indizes:*** **NDVI**: Allgemeiner Vegetationszustand (am häufigsten verwendet)
* **NDRE**: Früherkennung von Stress in Kombination mit RedEdge
* **GNDVI**: Empfindlich gegenüber Chlorophyllkonzentration
* **OSAVI**: Funktioniert gut bei sichtbarem Boden
* **EVI**: Regionen mit hohem Blattflächenindex (LAI)**Benutzerdefinierte Formeln (nur Chloros+):**

* Erstellen Sie benutzerdefinierte multispektrale Indexformeln
* Verwenden Sie Bandberechnungen mit allen Bildkanälen
* Speichern Sie benutzerdefinierte Formeln zur Wiederverwendung

Alle verfügbaren Indizes und Formeln finden Sie unter [Multispektrale Indexformeln](../project-settings/multispectral-index-formulas.md).

### Export

Steuert das Ausgabeformat und die Qualität der Datei.

**Verfügbare Formate:*** **TIFF (16-Bit)**: Empfohlen für GIS und wissenschaftliche Analysen (Bereich 0–65.535)
* **TIFF (32-Bit, Prozent)**: Gleitkomma-Reflexionswerte (Bereich 0,0–1,0)
* **PNG (8-Bit)**: Verlustfreie Komprimierung für die Visualisierung (Bereich 0–255)
* **JPG (8-Bit)**: Kleinste Dateien, verlustbehaftete Komprimierung (Bereich 0–255)***

## Einstellungen speichern und laden

### Projektvorlage speichern

Erstellen Sie wiederverwendbare Vorlagen für konsistente Arbeitsabläufe:

1. Konfigurieren Sie alle gewünschten Einstellungen im Bereich „Projekteinstellungen“
2. Scrollen Sie nach unten zum Abschnitt **„Projektvorlage speichern“**

3. Geben Sie einen aussagekräftigen Namen für die Vorlage ein (z. B. „Survey3N\_RGN\_Landwirtschaft“)
4. Klicken Sie auf das Speichersymbol

**Vorteile:**

* Wenden Sie identische Einstellungen auf mehrere Projekte an
* Teilen Sie Konfigurationen mit Teammitgliedern
* Sorgen Sie für Konsistenz bei wiederholten Erhebungen

### Vorlage in neues Projekt laden

Beim Erstellen eines neuen Projekts:

1. Wählen Sie **„Neues Projekt“** aus dem Hauptmenü
2. Wählen Sie die Option **„Aus Vorlage laden“**

3. Wählen Sie Ihre gespeicherte Vorlage aus
4. Alle Einstellungen werden automatisch übernommen

### Arbeitsverzeichnis

Die Einstellung **„Projektordner speichern“** legt fest, wo neue Projekte standardmäßig erstellt werden:

* **Standardpfad**: `C:\Users\[Username]\Chloros Projects`
* **Pfad ändern**: Klicken Sie auf das Bearbeitungssymbol und wählen Sie einen neuen Ordner aus
* **Wann ändern**:
  * Netzlaufwerk für die Zusammenarbeit im Team
  * Anderes Laufwerk mit mehr Speicherplatz
  * Organisierte Ordnerstruktur nach Jahr/Kunde

***

## PPK-Einrichtung (Post-Processed Kinematic)

Bei Verwendung von MAPIR-DAQ-Recordern mit GPS für präzise Geolokalisierung:

### Voraussetzungen

* MAPIR DAQ mit GPS-Modul (GNSS)
* .daq-Protokolldatei mit Belichtungspin-Einträgen
* Kamera während der Aufnahmesitzung an die Belichtungspins des DAQ angeschlossen

### Konfigurationsschritte

1. Legen Sie die .daq-Protokolldatei in Ihrem Projektordner ab
2. Aktivieren Sie in den Projekteinstellungen das Kontrollkästchen **„PPK-Korrekturen anwenden“**

3. Stellen Sie bei Bedarf den**„Zeitzonen-Offset des Lichtsensors“** ein (Standard: 0 für UTC)
4. Weisen Sie den Belichtungspins Kameras zu:
   * **Einzelkamera**: Wird automatisch Pin 1 zugewiesen
   * **Zwei Kameras**: Weisen Sie jede Kamera manuell dem richtigen Pin zu**Belichtungspin-Zuweisung:*** **Belichtungspin 1**: Wählen Sie das Kameramodell aus der Dropdown-Liste aus
* **Belichtungspin 2**: Wählen Sie die zweite Kamera oder „Nicht verwenden“
* Dieselbe Kamera kann nicht beiden Pins zugewiesen werden

{% hint style="warning" %}
**Wichtig**: Die Belichtungspins müssen den jeweiligen Kameras korrekt zugewiesen werden. Eine falsche Zuweisung führt zu falschen Geolokalisierungsdaten.
{% endhint %}

***

## Erweiterte Szenarien

### Projekte mit mehreren Kameras

Bei der Verarbeitung von Bildern mehrerer MAPIR-Kameras in einem Projekt:

1. Chloros erkennt automatisch jedes Kameramodell
2. Jede Kamera erhält das passende Verarbeitungsprofil
3. PPK: Weisen Sie jede Kamera manuell dem richtigen Belichtungspin zu
4. Alle Kameras verwenden dasselbe Exportformat und dieselben Indizes

**Beispiel**: Survey3W RGN + Survey3N OCN Dual-Kamera-Rig

### Zeitraffer- oder Mehrzeitpunkt-Erhebungen

Für wiederholte Erhebungen desselben Gebiets im Zeitverlauf:

1. Erstellen Sie eine Vorlage mit Ihren Standardeinstellungen
2. Verwenden Sie bei jeder Sitzung ein einheitliches Kalibrierungsziel-Setup
3. Verarbeiten Sie jeden Zeitpunkt als separates Projekt
4. Verwenden Sie identische Einstellungen für vergleichbare Ergebnisse
5. Exportieren Sie im gleichen Format für zeitliche Analysen

### Große Datensätze

Für Projekte mit vielen Bildern (500+):

* Erwägen Sie eine Aufteilung in kleinere Projekte nach Datum oder Gebiet
* Verwenden Sie die parallele Verarbeitung mit Chloros+ für schnellere Ergebnisse
* Erwägen Sie CLI oder API für die Stapelverarbeitung
* Passen Sie das minimale Neukalibrierungsintervall an, um die Zielerkennungszeit zu verkürzen

***

## Überprüfen Ihrer Einstellungen

Überprüfen Sie vor Beginn der Verarbeitung diese wichtigen Einstellungen:

* [ ] Kameramodell im Dateibrowser korrekt erkannt
* [ ] Vignettenkorrektur aktiviert
* [ ] Reflexionskalibrierung aktiviert
* [ ] Mindestens ein Kalibrierungszielbild importiert
* [ ] Gewünschte multispektrale Indizes hinzugefügt
* [ ] Für Ihren Workflow geeignetes Exportformat
* [ ] PPK-Einstellungen konfiguriert (bei Verwendung von .daq mit Belichtungsereignissen)

***

## Nächste Schritte

Sobald Ihre Einstellungen konfiguriert sind:

1. **Kalibrierungszielbilder markieren** – Siehe [Auswahl von Zielbildern](choosing-target-images.md)
2. **Verarbeitung starten** – Siehe [Starten der Verarbeitung](starting-the-processing.md)
3. **Fortschritt überwachen** – Siehe [Überwachen der Verarbeitung](monitoring-the-processing.md)

Ausführliche Informationen zu allen verfügbaren Einstellungen finden Sie in der Referenzdokumentation unter [Projekteinstellungen](../project-settings/project-settings.md).
