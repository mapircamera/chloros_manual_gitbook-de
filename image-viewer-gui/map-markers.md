# Kartenmarkierungen

Auf der Registerkarte „Karte“ werden Ihre Bilder anhand ihrer GPS-Koordinaten auf einer interaktiven 2D-Karte angezeigt. Dies bietet einen geografischen Überblick über Ihre Aufnahmesitzung und hilft Ihnen, die räumliche Abdeckung zu visualisieren. Dies ist auch beim ersten Importieren Ihrer Bilder nützlich, um nicht benötigte Bilder schnell zu entfernen.

## Auf die Registerkarte „Karte“ zugreifen

1. Öffnen oder erstellen Sie ein Projekt in Chloros.
2. Importieren Sie Bilder, die GPS-Metadaten enthalten.
3. Klicken Sie auf die Registerkarte „Karte“ <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> in der linken Seitenleiste.
4. Die Karte zeigt Markierungen an den GPS-Standorten der einzelnen Bilder an.

{% hint style=&quot;info&quot; %}
**GPS erforderlich**: Nur Bilder mit eingebetteten GPS-Koordinaten in ihren EXIF-Metadaten werden auf der Karte angezeigt. Stellen Sie sicher, dass Ihr Kamera während der Aufnahme GPS aktiviert hat.
{% endhint %}

***

## Bilder über die Registerkarte „Karte“ anpassen

Die Registerkarte „Karte“ <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> hat die gleichen Funktionen zum Hinzufügen  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  und Entfernen  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  Dateischaltflächen wie die Registerkarte [**Dateibrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Sie zeigt auch dieselbe Projektdateitabellenliste, jedoch mit anderen Spaltenüberschriften:

### Dateiname

* Originaldateiname der Kamera
* Behält die Namenskonvention der Kamera bei (z. B. IMG\_0001.RAW)

### Breitengrad

* Der Breitengrad des Bildes

### Längengrad

* Der Längengrad des Bildes

### Höhe

* Die Höhe des Bildes

{% hint style=&quot;info&quot; %}
Durch Klicken auf die Spaltenüberschriften der Tabelle werden auch die Zeilendaten sortiert.
{% endhint %}

***

## Bildmarkierungen

Jedes Bild mit GPS-Daten wird durch eine Markierung auf der Karte dargestellt:

### Markierungsanzeige

* Markierungen geben die genauen GPS-Koordinaten an, an denen jedes Bild aufgenommen wurde.
* Gruppierte Markierungen können beim Herauszoomen zusammengefasst werden.
* Zoomen Sie hinein, um die einzelnen Bildpositionen anzuzeigen.

{% hint style=&quot;success&quot; %}
SUPER-ZOOM: Wenn Sie die maximale Zoomstufe des Kartenkachelanbieters erreicht haben, wird die Kachel bei weiterem Zoomen vergrößert, sodass Sie Markierungen sehen können, die nahe beieinander liegen.
{% endhint %}

### Vorschau beim Darüberfahren

* **Bewegen Sie den Mauszeiger** über eine beliebige Markierung, um eine Miniaturvorschau des Bildes anzuzeigen.
* So können Sie Bilder schnell visuell identifizieren, ohne die Kartenansicht zu verlassen.
* Nützlich, um bestimmte Bilder innerhalb einer großen Aufnahmesitzung zu finden.

***

## Kartenkachelanbieter

{% hint style=&quot;success&quot; %}
**Automatische Auswahl**: Chloros wählt automatisch den Kachel-Dienst aus, der die beste Zoomstufe für Ihren aktuellen Kartenstandort bietet. Bei Bedarf können Sie manuell zwischen den Anbietern wechseln.
{% endhint %}

Die Registerkarte „Karte“ unterstützt zwei Kachelanbieter für die Hintergrundkartenbilder:

### Google Maps

* Standard-Satelliten- und Kartenbilder von Google
* Am besten geeignet für allgemeine weltweite Abdeckung

### ESRI

* Satelliten- und Luftbilder von ESRI ArcGIS
* Bietet oft Bilder mit höherer Auflösung in bestimmten Regionen

***

## Kartenkachel-Typen

Sie können den Kartenschicht-Typ auswählen (von links nach rechts):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Gelände

Zeigt Höhenprofile und Kartenkacheln mit Details (Straßen usw.) an.

### Karte

Zeigt Standard-Kartenkacheln (geringere Bandbreite) mit Details (Straßen usw.) an.

### Satellit

Zeigt detaillierte Satellitenkartenkacheln (höhere Bandbreite) an.

### Hybrid

Zeigt Satellitenkartenkacheln mit zusätzlichen Details (Straßen usw.) an.

***

## Kartennavigation

### Zoomsteuerung

* **Vergrößern/Verkleinern**: Verwenden Sie das Mausrad oder die Zoomtasten.
* **Vollbild**: Vollbildanzeige der Karte

### Schwenksteuerung

* **Schwenk**: Klicken und ziehen Sie, um sich auf der Karte zu bewegen.***

## Anwendungsfälle

### Visualisierung der Flugbahn

* Zeigen Sie den Erfassungsbereich von Drohnenaufnahmen an.
* Identifizieren Sie Lücken in der Bildabdeckung.
* Überprüfen Sie die Ausführung der Flugbahn.

### Überprüfung der Bodenvermessung

* Sehen Sie sich die räumliche Verteilung der bodengestützten Aufnahmen an.
* Lokalisieren Sie Kalibrierungszielbilder relativ zum Vermessungsgebiet.
* Planen Sie zusätzliche Aufnahmeorte.

### Qualitätskontrolle

* Identifizieren Sie schnell Bilder, die an unerwarteten Orten aufgenommen wurden.
* Überprüfen Sie die GPS-Genauigkeit im gesamten Datensatz.
* Vergleichen Sie Bildorte mit Feldnotizen.

***

## Fehlerbehebung

### Keine Markierungen werden angezeigt

**Mögliche Ursachen:**

* Bilder enthalten keine GPS-Metadaten.
* GPS war während der Aufnahme auf der Kamera deaktiviert.
* EXIF-Daten wurden durch externe Software entfernt.

**Lösung**: Überprüfen Sie, ob GPS auf Ihrer Kamera aktiviert ist, und importieren Sie die Originaldateien erneut.

### Markierungen an falscher Stelle

**Mögliche Ursachen:**

* Das GPS der Kamera hatte eine schlechte Satellitenortung.
* GPS-Drift während der Aufnahme.

**Lösung**: Dies ist in der Regel ein Problem der Aufnahmezeit. Erwägen Sie die Verwendung von PPK/RTK-GPS für Präzisionsanwendungen.
