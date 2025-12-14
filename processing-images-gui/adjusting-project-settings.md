# Anpassen der Projekteinstellungen

Bevor Sie Ihre Bilder bearbeiten, ist es wichtig, die Projekteinstellungen entsprechend Ihren Arbeitsabläufen zu konfigurieren. Das Projektfenster <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> bietet umfassende Kontrolle über Kalibrierung, Verarbeitungsoptionen, multispektrale Indizes und Exportformate.

## Zugriff auf die Projekteinstellungen

1. Öffnen Sie Ihr Projekt in Chloros
2. Klicken Sie auf das Symbol **Projekteinstellungen** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in der linken Seitenleiste.
3. Das Fenster „Projekteinstellungen“ zeigt alle Konfigurationsoptionen an.

{% hint style=&quot;info&quot; %}
**Die Einstellungen werden automatisch** mit Ihrem Projekt gespeichert. Wenn Sie ein Projekt erneut öffnen, werden alle Einstellungen wiederhergestellt.
{% endhint %}

***

## Schnelle Einrichtung für gängige Arbeitsabläufe

### Standardeinstellungen (für die meisten Benutzer empfohlen)

Für typische MAPIR Survey3 Kamera-Workflows eignen sich die Standardeinstellungen gut:

* ✅ **Vignettenkorrektur**: Aktiviert
* ✅ **Reflexionskalibrierung**: Aktiviert (erfordert Bilder von MAPIR-Zielen)
* ✅ **Debayer-Verfahren**: Hohe Qualität (schneller)
* ✅ **Exportformat**: TIFF (16 Bit)

Importieren Sie einfach Ihre Bilder und beginnen Sie mit der Verarbeitung mit diesen Standardeinstellungen.

***

## Übersicht über die Projekteinstellungen

Das Fenster „Projekteinstellungen“ ist in mehrere Kategorien unterteilt. Nachfolgend finden Sie eine Zusammenfassung der einzelnen Abschnitte. Die vollständige Dokumentation finden Sie unter [Projekteinstellungen](../project-settings/project-settings.md).

### Zielerkennung

Steuert, wie Chloros Kalibrierungsziele in Ihren Bildern identifiziert.

**Wichtige Einstellungen:**

* **Minimale Kalibrierungsprobenfläche**: Größenschwelle für die Zielerkennung (Standard: 25 Pixel)
* **Minimale Zielclusterung**: Ähnlichkeitsschwelle für die Gruppierung von Zielbereichen (Standard: 60)

**Wann anpassen:**

* Vergrößern Sie die Probenfläche, wenn es zu Fehlerkennungen kommt.
* Verringern Sie sie, wenn Ziele nicht erkannt werden.
* Passen Sie die Clusterung an, wenn Ziele in mehrere Erkennungen aufgeteilt werden.

### Verarbeitung

Wichtigste Bildverarbeitungs- und Kalibrierungsoptionen.

**Wichtige Einstellungen:**

* **Vignettenkorrektur**: Kompensiert die Verdunkelung des Objektivs an den Rändern ✅ Empfohlen
* **Reflexionskalibrierung**: Normalisiert Werte mithilfe von Kalibrierungszielen ✅ Empfohlen
* **Debayer-Verfahren**: Algorithmus zur Konvertierung von RAW in 3-Kanal-Multispektral
* **Minimales Neukalibrierungsintervall**: Zeit zwischen der Verwendung von Kalibrierungszielen (0 = alle verwenden)

**Erweiterte Einstellungen:**

* **Zeitzonenversatz des Lichtsensors**: Für die PPK-Zeitsynchronisation (Standard: 0)
* **PPK-Korrekturen anwenden**: Verwendet GPS-/Belichtungs-Pin-Daten aus .daq-Dateien
* **Belichtungs-Pin 1/2**: Weist Kameras Belichtungs-Pins für Dual-Kamera-Setups zu

### Index (Multispektrale Indizes)

Konfigurieren Sie, welche Vegetationsindizes berechnet und exportiert werden sollen.

**So fügen Sie Indizes hinzu:**

1. Klicken Sie auf die Schaltfläche **„Index hinzufügen“**
2. Wählen Sie einen Index aus dem Dropdown-Menü aus (NDVI, NDRE, GNDVI usw.).
3. Konfigurieren Sie die Visualisierungseinstellungen (LUT-Farben, Wertebereiche).
4. Fügen Sie nach Bedarf mehrere Indizes hinzu.

**Beliebte Indizes:**

* **NDVI**: Allgemeiner Gesundheitszustand der Vegetation (am häufigsten)
* **NDRE**: Früherkennung von Stress mit RedEdge
* **GNDVI**: Empfindlich gegenüber Chlorophyllkonzentration
* **OSAVI**: Funktioniert gut bei sichtbarem Boden
* **EVI**: Regionen mit hohem Blattflächenindex (LAI)

**Benutzerdefinierte Formeln (nur Chloros+):**

* Erstellen Sie benutzerdefinierte Multispektralindexformeln.
* Verwenden Sie Bandmathematik mit allen Bildkanälen.
* Speichern Sie benutzerdefinierte Formeln zur Wiederverwendung.

Alle verfügbaren Indizes und Formeln finden Sie unter [Multispektralindexformeln](../project-settings/multispectral-index-formulas.md).

### Exportieren

Steuert das Ausgabeformat und die Qualität der Datei.

**Verfügbare Formate:**

* **TIFF (16 Bit)**: Empfohlen für GIS und wissenschaftliche Analysen (Bereich 0–65.535)
* **TIFF (32 Bit, Prozent)**: Fließkomma-Reflexionswerte (Bereich 0,0–1,0)
* **PNG (8 Bit)**: Verlustfreie Komprimierung für die Visualisierung (Bereich 0–255)
* **JPG (8 Bit)**: Kleinste Dateien, verlustbehaftete Komprimierung (Bereich 0–255)

***

## Speichern und Laden von Einstellungen

### Projektvorlage speichern

Erstellen Sie wiederverwendbare Vorlagen für konsistente Arbeitsabläufe:

1. Konfigurieren Sie alle gewünschten Einstellungen im Bereich „Projekteinstellungen“.
2. Scrollen Sie zum Abschnitt „**Projektvorlage speichern**“ unten.
3. Geben Sie einen aussagekräftigen Namen für die Vorlage ein (z. B. „Survey3N\_RGN\_Agriculture“).
4. Klicken Sie auf das Speichersymbol.

**Vorteile:**

* Wenden Sie identische Einstellungen auf mehrere Projekte an.
* Teilen Sie Konfigurationen mit Teammitgliedern.
* Sorgen Sie für Konsistenz bei wiederholten Umfragen.

### Vorlage in neues Projekt laden

Beim Erstellen eines neuen Projekts:

1. Wählen Sie **„Neues Projekt”** aus dem Hauptmenü.
2. Wählen Sie die Option **„Aus Vorlage laden”**.
3. Wählen Sie Ihre gespeicherte Vorlage aus.
4. Alle Einstellungen werden automatisch übernommen.

### Arbeitsverzeichnis

Die Einstellung **„Projektordner speichern”** legt fest, wo neue Projekte standardmäßig erstellt werden:

* **Standardort**: `C:\Users\[Username]\Chloros Projects`
* **Ort ändern**: Klicken Sie auf das Bearbeitungssymbol und wählen Sie einen neuen Ordner aus.
* **Wann ändern**:
  * Netzlaufwerk für die Zusammenarbeit im Team
  * Anderes Laufwerk mit mehr Speicherplatz
  * Organisierte Ordnerstruktur nach Jahr/Kunde

***

## PPK-Einrichtung (Post-Processed Kinematic)

Bei Verwendung von MAPIR DAQ-Rekordern mit GPS für präzise Geolokalisierung:

### Voraussetzungen

* MAPIR DAQ mit GPS-Modul (GNSS)
* .daq-Protokolldatei mit Einträgen zu den Belichtungsstiften
* Kamera während der Aufnahmesitzung an DAQ-Belichtungs-Pins angeschlossen

### Konfigurationsschritte

1. Legen Sie die .daq-Protokolldatei in Ihrem Projektordner ab.
2. Aktivieren Sie in den Projekteinstellungen das Kontrollkästchen **„PPK-Korrekturen anwenden“**.
3. Stellen Sie bei Bedarf **„Zeitzonenversatz des Lichtsensors“** ein (Standard: 0 für UTC).
4. Weisen Sie den Belichtungs-Pins Kameras zu:
   * **Einzelne Kamera**: Wird automatisch Pin 1 zugewiesen.
   * **Zwei Kameras**: Weisen Sie jede Kamera manuell dem richtigen Pin zu.

**Belichtungs-Pin-Zuweisung:**

* **Belichtungs-Pin 1**: Wählen Sie das Kameramodell aus der Dropdown-Liste aus.
* **Belichtungs-Pin 2**: Wählen Sie die zweite Kamera oder „Nicht verwenden”.
* Dieselbe Kamera kann nicht beiden Pins zugewiesen werden.

{% Hinweis style=&quot;warning&quot; %}
**Wichtig**: Belichtungs-Pins müssen den jeweiligen Kameras korrekt zugewiesen werden. Eine falsche Zuweisung führt zu falschen Geolokalisierungsdaten.
{% endhint %}

***

## Erweiterte Szenarien

### Projekte mit mehreren Kameras

Bei der Verarbeitung von Bildern aus mehreren MAPIR-Kameras in einem Projekt:

1. Chloros erkennt automatisch jedes Kameramodell.
2. Jede Kamera erhält ein geeignetes Verarbeitungsprofil.
3. PPK: Weisen Sie jeder Kamera manuell den richtigen Belichtungsstift zu.
4. Alle Kameras verwenden dasselbe Exportformat und dieselben Indizes.

**Beispiel**: Survey3W RGN + Survey3N OCN Dual-Kamera-Rig

### Zeitraffer- oder Mehrfachdatenerhebungen

Für wiederholte Erhebungen desselben Gebiets im Laufe der Zeit:

1. Erstellen Sie eine Vorlage mit Ihren Standardeinstellungen.
2. Verwenden Sie bei jeder Sitzung eine einheitliche Kalibrierungszielkonfiguration.
3. Verarbeiten Sie jedes Datum als separates Projekt.
4. Verwenden Sie identische Einstellungen für vergleichbare Ergebnisse.
5. Exportieren Sie im gleichen Format für die zeitliche Analyse.

### Große Datensätze

Für Projekte mit vielen Bildern (500+):

* Erwägen Sie, die Daten nach Datum oder Gebiet in kleinere Projekte aufzuteilen.
* Verwenden Sie die parallele Verarbeitung Chloros+ für schnellere Ergebnisse.
* Erwägen Sie CLI oder API für die Batch-Automatisierung.
* Passen Sie das minimale Neukalibrierungsintervall an, um die Zielerkennungszeit zu verkürzen.

***

## Überprüfen Ihrer Einstellungen

Überprüfen Sie vor Beginn der Verarbeitung diese wichtigen Einstellungen:

* [ ] Kameramodell im Dateibrowser korrekt erkannt
* [ ] Vignettenkorrektur aktiviert
* [ ] Reflektionskalibrierung aktiviert
* [ ] Mindestens ein Kalibrierungszielbild importiert
* [ ] Gewünschte multispektrale Indizes hinzugefügt
* [ ] Für Ihren Workflow geeignetes Exportformat
* [ ] PPK-Einstellungen konfiguriert (bei Verwendung von .daq mit Belichtungsereignissen)

***

## Nächste Schritte

Sobald Ihre Einstellungen konfiguriert sind:

1. **Markieren Sie Kalibrierungszielbilder** – Siehe [Auswählen von Zielbildern](choosing-target-images.md)
2. **Starten Sie die Verarbeitung** – Siehe [Starten der Verarbeitung](starting-the-processing.md)
3. **Überwachen Sie den Fortschritt** – Siehe [Überwachen der Verarbeitung](monitoring-the-processing.md)

Ausführliche Informationen zu allen verfügbaren Einstellungen finden Sie in der Referenzdokumentation [Projekteinstellungen](../project-settings/project-settings.md).
