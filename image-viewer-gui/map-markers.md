# Kartenmarkierungen

Auf der Registerkarte „Karte“ werden Ihre Bilder anhand ihrer GPS-Koordinaten auf einer interaktiven 2D-Karte angezeigt. Dies bietet einen geografischen Überblick über Ihre Aufnahmesitzung und hilft Ihnen, die räumliche Abdeckung zu visualisieren. Dies ist auch beim ersten Import Ihrer Bilder nützlich, um Bilder, die Sie nicht bearbeiten müssen, schnell zu entfernen.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Auf die Registerkarte „Karte“ zugreifen

1. Öffnen oder erstellen Sie ein Projekt in Chloros
2. Importieren Sie Bilder, die GPS-Metadaten enthalten
3. Klicken Sie auf die Registerkarte **Karte** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> in der linken Seitenleiste
4. Die Karte zeigt Markierungen an den GPS-Standorten der einzelnen Bilder an

{% hint style="info" %}
**GPS erforderlich**: Auf der Karte werden nur Bilder angezeigt, deren EXIF-Metadaten GPS-Koordinaten enthalten. Stellen Sie sicher, dass die GPS-Funktion Ihrer Kamera während der Aufnahme aktiviert ist.
{% endhint %}

***

## Bilder über die Registerkarte „Karte“ bearbeiten

Die Registerkarte **Karte**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> verfügt über dieselben Schaltflächen zum Hinzufügen  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  und  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  Datei-Schaltflächen wie die Registerkarte [**Dateibrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Außerdem zeigt er dieselbe Projektdatei-Tabellenliste an, jedoch mit anderen Spaltenüberschriften:

### Dateiname

* Ursprünglicher Dateiname der Kamera
* Behält die Namenskonvention der Kamera bei (z. B. IMG\_0001.RAW)

### Breitengrad

* Der Breitengrad des Bildes

### Längengrad

* Der Längengrad des Bildes

### Höhe

* Die Höhe des Bildes

{% hint style="info" %}
Durch Klicken auf die Spaltenüberschriften der Tabelle werden die Daten in den Zeilen sortiert
{% endhint %}

***

## Bildmarkierungen

Jedes Bild mit GPS-Daten wird durch eine Markierung auf der Karte dargestellt:

### Anzeige der Markierungen

* Markierungen zeigen die genauen GPS-Koordinaten an, an denen jedes Bild aufgenommen wurde
* Bei starkem Herauszoomen können Markierungen zu Gruppen zusammengefasst werden
* Zoomen Sie heran, um die einzelnen Bildstandorte zu sehen

{% hint style="success" %}
SUPER-ZOOM: Wenn Sie die maximale Zoomstufe des Kartenkachel-Anbieters erreichen, wird die Kachel bei weiterem Heranzoomen vergrößert, sodass Sie Markierungen sehen können, die nahe beieinander liegen.
{% endhint %}

### Vorschau beim Bewegen des Mauszeigers

* **Bewegen Sie den Mauszeiger** über einen beliebigen Marker, um eine Miniaturansicht des Bildes zu sehen
* Dies ermöglicht eine schnelle visuelle Identifizierung, ohne die Kartenansicht zu verlassen
* Nützlich zum Auffinden bestimmter Bilder innerhalb einer großen Aufnahmeserie

***

## Kartenkachel-Anbieter

{% hint style="success" %}
**Automatische Auswahl**: Chloros wählt automatisch den Kacheldienst aus, der die beste Zoomstufe für Ihren aktuellen Kartenstandort bietet. Sie können bei Bedarf manuell zwischen den Anbietern wechseln.
{% endhint %}

Die Registerkarte „Karte“ unterstützt zwei Kachelanbieter für die Hintergrundkartenbilder:

### Google Maps

* Standard-Satelliten- und Kartenbilder von Google
* Am besten geeignet für allgemeine weltweite Abdeckung

### ESRI

* Satelliten- und Luftbilder von ESRI ArcGIS
* Bietet in bestimmten Regionen oft Bilder mit höherer Auflösung

***

## Kartenkachel-Typen

Sie können den Kartenschicht-Typ auswählen (von links nach rechts):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Gelände

Zeigt Höhenprofile und Kartenkacheln mit Details (Straßen usw.) an

### Karte

Zeigt Standard-Kartenkacheln (geringere Bandbreite) mit Details (Straßen usw.) an

### Satellit

Zeigt detaillierte Satellitenkartenkacheln (höhere Bandbreite) an

### Hybrid

Zeigt Satellitenkartenkacheln mit zusätzlichen Details (Straßen usw.) an

***

## Kartennavigation

### Zoom-Steuerung

* **Vergrößern/Verkleinern**: Verwenden Sie das Mausrad oder die Zoom-Schaltflächen
* **Vollbild**: Zeigt die Karte im Vollbildmodus an

### Schwenk-Steuerung

* **Schwenken**: Klicken und ziehen, um sich auf der Karte zu bewegen***

## Anwendungsfälle

### Visualisierung der Flugbahn

* Anzeigen des Erfassungsbereichs von Drohnenaufnahmen
* Identifizieren von Lücken in der Bildabdeckung
* Überprüfen der Flugbahnausführung

### Überprüfung der Bodenvermessung

* Anzeigen der räumlichen Verteilung bodengestützter Aufnahmen
* Lokalisieren von Kalibrierungszielbildern relativ zum Vermessungsgebiet
* Planen zusätzlicher Aufnahmestandorte

### Qualitätskontrolle

* Schnelle Identifizierung von Bildern, die an unerwarteten Orten aufgenommen wurden
* Überprüfung der GPS-Genauigkeit im gesamten Datensatz
* Abgleich der Bildstandorte mit Feldnotizen

***

## Fehlerbehebung

### Es werden keine Markierungen angezeigt

**Mögliche Ursachen:**

* Die Bilder enthalten keine GPS-Metadaten
* Das GPS der Kamera war während der Aufnahme deaktiviert
* Die EXIF-Daten wurden durch externe Software entfernt

**Lösung**: Überprüfen Sie, ob GPS auf Ihrer Kamera aktiviert ist, und importieren Sie die Originaldateien erneut

### Marker an falscher Position

**Mögliche Ursachen:**

* Das Kamera-GPS hatte einen schlechten Satellitenempfang
* GPS-Abweichung während der Aufnahme

**Lösung**: Dies ist in der Regel ein Problem zum Zeitpunkt der Aufnahme; erwägen Sie die Verwendung von PPK/RTK-GPS für Präzisionsanwendungen
