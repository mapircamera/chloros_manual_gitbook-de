# Kartenmarkierungen

Auf der Registerkarte „Karte“ werden Ihre Bilder anhand ihrer GPS-Koordinaten auf einer interaktiven 2D-Karte dargestellt. So erhalten Sie einen geografischen Überblick über eine Aufnahmesitzung und können Bilder, die Sie nicht bearbeiten möchten, direkt nach dem Import auf schnellstem Wege aussortieren.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Aufrufen der Registerkarte „Karte“

1. Öffnen oder erstellen Sie ein Projekt in Chloros.
2. Importieren Sie Bilder, die GPS-Metadaten enthalten.
3. Klicken Sie in der linken Seitenleiste auf die Registerkarte **Karte** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line">.
4. Auf der Karte wird an der GPS-Position jedes Bildes eine Markierung angezeigt.

{% hint style="info" %}
**GPS erforderlich**: Nur Bilder mit GPS-Koordinaten in ihren EXIF-Metadaten werden auf der Karte angezeigt. Ein Bild ohne Koordinaten ist weiterhin im Projekt enthalten und wird normal verarbeitet – es hat lediglich keine Markierung.
{% endhint %}

***

## Bilder über den Reiter „Karte“ bearbeiten

Der Reiter **Karte**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> verfügt über dieselben Schaltflächen zum Hinzufügen <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> und Entfernen <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> von Dateien wie der Reiter [**Dateibrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Er zeigt dieselbe Projektdateiliste mit geografischen Spalten an:

| Spalte        | Inhalt                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Name**      | Der Dateiname, wie er von der Kamera stammt                             |
| **Breitengrad** | Dezimalgrad, sechs Dezimalstellen                                |
| **Längengrad** | Dezimalgrad, sechs Dezimalstellen                                |
| **Höhe**  | Meter, eine Dezimalstelle – `-`, wenn das Bild keine Höhenangabe enthält |

{% hint style="info" %}
Klicken Sie auf eine beliebige Spaltenüberschrift, um nach dieser zu sortieren; klicken Sie erneut, um die Reihenfolge umzukehren.
{% endhint %}

{% hint style="warning" %}
**Die Höhe bezieht sich auf die Höhe über dem Meeresspiegel, nicht auf die Höhe über dem Boden.** Der Wert stammt aus dem EXIF-Tag `GPSAltitude` des Bildes, das sich auf den mittleren Meeresspiegel bezieht. Es handelt sich nicht um die Flughöhe über dem Gelände, und Chloros leitet daraus keinen Bodenauflösungsabstand ab – über einem Feld, das 300 m über dem Meeresspiegel liegt, zeichnet eine Drohne in 100 m Höhe über dem Boden (AGL) hier etwa 400 m auf. Verwenden Sie die Spalte, um Ausreißer zu erkennen und eine konsistente Flughöhe zu bestätigen, nicht als AGL-Messwert.
{% endhint %}

***

## Bildmarkierungen

Jedes Bild mit GPS-Daten erhält eine Markierung an seinen Koordinaten.

### Anzeige der Markierungen

* Markierungen befinden sich genau an den für jede Aufnahme aufgezeichneten Koordinaten
* Markierungen, die dicht beieinander liegen, können sich beim Herauszoomen optisch überlappen – zoomen Sie hinein, um sie voneinander zu trennen
* Ausgewählte und hervorgehobene Markierungen werden über den übrigen angezeigt

### Vorschau beim Bewegen des Mauszeigers

* Bewegen Sie den Mauszeiger über einen beliebigen Marker, um eine Miniaturansicht des Bildes mit seinem Dateinamen anzuzeigen
* **Klicken**Sie auf einen Marker, um das Bild auszuwählen und das Popup**anzuheften** – es bleibt sichtbar, bis Sie an eine andere Stelle klicken. Solange ein Popup angeheftet ist, wird es nicht durch das Bewegen des Mauszeigers über andere Marker verdeckt
* Dies ist der schnellste Weg, um ein bestimmtes Bild in einer umfangreichen Sitzung zu finden, ohne die Karte zu verlassen

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Auf der Registerkarte „Karte“ werden alle mit Geotags versehenen Bilder des Projekts dargestellt</p></figcaption></figure>### Super-Zoom

{% hint style="success" %}
**SUPER-ZOOM**: Wenn Sie den maximalen Zoom erreichen, für den der Kachelanbieter Bildmaterial bereitstellt, werden die Kacheln beim weiteren Vergrößern weiter vergrößert, anstatt zu stoppen, sodass Sie Markierungen unterscheiden können, die fast übereinander liegen.
{% endhint %}

* Der Super-Zoom wird nur aktiviert, wenn Sie sich **genau** am maximalen Zoomfaktor des Anbieters für diesen Standort befinden und die Kacheln vollständig geladen sind. Darunter verhält sich das Zoomen normal
* Der Bereich liegt **zwischen 1× und 32×** über dem maximalen Zoomfaktor des Anbieters
* Eine Anzeige in der Ecke zeigt den aktuellen Super-Zoom als Prozentsatz an, und eine **×**-Schaltfläche daneben versetzt dich mit einem Klick zurück zum normalen Zoom
* Das Herauszoomen wird immer an die Karte selbst weitergeleitet, sodass du niemals im Super-Zoom „steckenbleiben“ kannst
* Das Zoomen und Schwenken im Super-Zoom-Modus überträgt den resultierenden Versatz zurück auf die Karte, sodass der nicht zentrierte Bereich, in den Sie sich bewegt haben, weiterhin Kacheln anfordert, anstatt leer zu bleiben
* Markierungen werden als Vektorelemente und nicht gerastert dargestellt, sodass sie auf jeder Super-Zoom-Stufe scharf bleiben

***

## Anbieter von Kartenkacheln

{% hint style="success" %}
**Automatische Auswahl**: Chloros wählt den Kacheldienst aus, der die beste Zoomstufe für den jeweiligen Standort Ihrer Bilder bietet. Sie können jederzeit manuell wechseln.
{% endhint %}

| Anbieter        | Anmerkungen                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Weltweite Abdeckung; unterstützt alle vier Kacheltypen                                                                                                            |
| **Esri ArcGIS**| Oft höher aufgelöste Luftbilder in bestimmten Regionen. Der Kacheltyp**Terrain** wird für Esri nicht angeboten und die entsprechende Schaltfläche ist deaktiviert, solange Esri ausgewählt ist |***

## Kartenkachel-Typen

Wählen Sie den Kartenschichttyp mit den Schaltflächen (von links nach rechts) aus:

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Typ                 | Zeigt                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Gelände**          | Höhenschattierung mit Kartendetails (Straßen, Beschriftungen). Nur Google       |
| **Karte**              | Standard-Straßenkartenkacheln – die Option mit der geringsten Bandbreite              |
| **Satellit**        | Detaillierte Satellitenbilder, keine Beschriftungen – die Option mit der höchsten Bandbreite |
| **Hybrid** (Standard) | Satellitenbilder mit darüber eingeblendeten Straßen und Beschriftungen                |

Die Registerkarte „Karte“ wird auf **Hybrid** geöffnet. Ihre Auswahl wirkt sich auf den Anbieterwechsel aus, sofern der Anbieter dies unterstützt.***

## Kartennavigation

* **Zoom**: Maus-Scrollrad oder die Zoom-Schaltflächen auf der Karte
* **Verschieben**: Klicken und Ziehen
* **Vollbild**: Mit der Vollbildsteuerung wird die Karte auf das gesamte Fenster vergrößert***

## Anwendungsfälle

### Überprüfung der Flugbahn

* Den Erfassungsbereich einer Drohnenflug-Sitzung auf einen Blick erkennen
* Lücken erkennen, an denen ein Durchgang ausgelassen wurde
* Überprüfen, ob der Flug dem geplanten Muster folgte

### Überprüfung der Bodenaufnahmen

* Die Verteilung der bodengestützten Aufnahmen anzeigen
* Kalibrierungszielrahmen relativ zum Vermessungsgebiet lokalisieren
* Entscheiden, wo zusätzliche Aufnahmen erforderlich sind

### Qualitätskontrolle

* Finden Sie Bilder, die an unerwarteten Stellen aufgenommen wurden, und entfernen Sie diese vor der Verarbeitung
* Sortieren Sie nach Höhe, um Bilder zu erkennen, die in der falschen Höhe aufgenommen wurden oder bei denen die GPS-Ortung ungenau war
* Vergleichen Sie die Bildstandorte mit den Feldnotizen

***

## Fehlerbehebung

### Es werden keine Marker angezeigt

**Mögliche Ursachen**

* Die Bilder enthalten keine GPS-Metadaten
* Das GPS war während der Aufnahme an der Kamera deaktiviert
* Die EXIF-Daten wurden vor dem Import durch andere Software entfernt

**Was zu tun ist**: Vergewissern Sie sich, dass GPS an der Kamera aktiviert ist, und importieren Sie die Originaldateien erneut. Sie können überprüfen, ob eine bestimmte Datei Koordinaten enthält, indem Sie in der Dateitabelle auf der Registerkarte „Karte“ danach suchen – ein Bild ohne Koordinaten hat dort keine Zeile.

### Marker befinden sich an der falschen Stelle

**Mögliche Ursachen**: eine schlechte Satellitenortung zum Zeitpunkt der Aufnahme oder eine GPS-Drift während der Sitzung.**Vorgehensweise**: Hierbei handelt es sich um ein Problem zum Zeitpunkt der Aufnahme, das Chloros nachträglich nicht korrigieren kann. Verwenden Sie für präzise Arbeiten einen PPK/RTK-GPS-Workflow – siehe die Einstellung**„PPK-Korrekturen anwenden“** in den [Projekteinstellungen](../project-settings/project-settings.md).

### Die Karte ist leer oder das Laden der Kacheln bricht ab

Die Kachelanbieter sind Online-Dienste. Wenn keine Kacheln mehr eintreffen, überprüfen Sie die Netzwerkverbindung des Geräts und versuchen Sie anschließend, den Anbieter zu wechseln. Wenn Sie stark gezoomt hatten, drücken Sie die **×**-Taste zum Zurücksetzen, um zur normalen Zoomstufe zurückzukehren, und lassen Sie die Karte die Kacheln erneut anfordern.***

## Verwandte Seiten

* [**Bildraster**](image-grid.md) – derselbe Bildsatz wie bei den Miniaturansichten
* [**Ein Bild im Vollbildmodus öffnen**](opening-an-image-full-screen.md) – ein Bild im Detail betrachten
* [**Dateien zu einem Projekt hinzufügen**](../processing-images-gui/adding-files-to-a-project.md) – die Schaltflächen zum Hinzufügen und Entfernen von Dateien, die auf dieser Registerkarte zu finden sind
