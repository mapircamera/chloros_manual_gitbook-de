# Aufzeichnung und das .daq-Format

Eine `.daq`-Datei ist das Aufzeichnungsformat für Lichtsensoren vChloros

: eine **SQLite-Datenbank** mit kalibrierten Spektralbildern eines DAQ-Sensors. Wenn Sie während einer Erfassungssitzung eine solche Datei aufzeichnen, kann die Reflektanz-Pipeline später jedes Bild durch die genau in diesem Moment gemessene nach unten gerichtete Bestrahlungsstärke dividieren.

## Was eine .daq-Datei enthält

| Eigenschaft | Wert |
| --- | --- |
| Container | SQLite-Datenbank, eine Datei pro Sensor und Aufzeichnung |
| Dateiname | Enthält die **Sensor-ID**und einen**Zeitstempel**, z. B. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum pro Bild | 135 Punkte, 340–1010 nm in 5-nm-Schritten, plus CIE-XYZ-Dreistimulus |
| Einheiten | Kalibrierte spektrale Bestrahlungsstärke, **W/m²/nm** (werkseitiges Kalibrierungspaket + angewandtes Kappenprofil) |
| Eingetragene Metadaten | Sensor-ID (der Schlüssel zum Abrufen der Werkskalibrierung dieses Geräts) und das aktuell gültige Kappenprofil – siehe [Kappenprofile &amp; kalibrierter Bereich](caps-and-range.md) |

Das Format ist bei DAQ-U, DAQ-M, DAQ-E und DAQ-E-S identisch, sodass es für die nachgelagerte Verarbeitung keine Rolle spielt, welches Gerät die Aufzeichnung vorgenommen hat. Eine DAQ-E-S-Aufzeichnung speichert neben jedem Spektrum auch die Neigung, Roll- und Nickwinkel des Geräts.

**Aufzeichnungen bleiben auch bei einem Stromausfall erhalten (1.3.0).**Eine Live-Aufzeichnung wird bei jedem Commit in ein Journal geschrieben und mit der Festplatte synchronisiert, sodass bei einem Stromausfall oder einem abgebrochenen Prozess lediglich die letzten ca. 3 Sekunden verloren gehen, nicht die Datei. Eine Aufzeichnung, die während des Schreibvorgangs abgebrochen wurde, kann neben der Datei „`.daq`“ eine Datei „`<name>.daq-journal`“ hinterlassen:**Behalten Sie beide Dateien und kopieren Sie sie zusammen.** Chloros

setzt das Journal beim ersten Lesen der Datei zurück. Eine ausgelastete Festplatte verzögert die Datei, niemals die Messwerte des Sensors.

**Ein DAQ-E, das keine Daten mehr liefert, wird wieder verbunden (1.3.0).** Wenn der Datenstrom eines DAQ-E ausfällt (Stromausfall am Switch, unterbrochene Verbindung), stellt das Backend die Verbindung mit den bisherigen Einstellungen wieder her, und eine darauf laufende Aufzeichnung wird in derselben `.daq`-Datei fortgesetzt, wobei die Lücke in der Datei markiert wird.

Für kalibrierte Aufzeichnungen ist das werkseitige Kalibrierungspaket des Sensors erforderlich. Bei DAQ-U und DAQ-M ruft das Backend das Paket anhand der Sensor-ID aus der MAPIR

-Cloud ab (die Aufzeichnung wird abgelehnt, wenn dies nicht möglich ist); DAQ-E-Geräte sind davon ausgenommen, da sie ihre Kalibrierung auf dem Gerät speichern.

## Aufzeichnung über die GUI

Für die Aufzeichnung über die GUI muss ein **offenes Projekt** vorhanden sein (andernfalls sind die Aufzeichnungsschaltflächen deaktiviert):

* **Alle aufzeichnen / Alle stoppen** – oben in der Seitenleiste „Lichtsensoren“; startet oder stoppt gleichzeitig eine `.daq`-Aufzeichnung auf allen angeschlossenen Sensoren.
* **Aufzeichnen / Aufzeichnung beenden** – pro Sensor, im Einstellungsmodal mit dem Zahnrad-Symbol. Während der Aufzeichnung wird in den Live-Info-Zeilen des Sensors eine rote „REC“-Anzeige eingeblendet.

Die Dateien werden in `<project>/light_sensor/` geschrieben, und wenn eine Aufzeichnung beendet wird – sei es über „Stopp“, „Alle stoppen“ oder durch das Trennen eines Aufzeichnungssensors –, wird die fertige `.daq`-Datei **automatisch dem geöffneten Projekt hinzugefügt**. Sie erscheint in der Dateiliste des Projekts, ohne dass ein manueller Hinzufügungsschritt erforderlich ist, und steht somit bereits für die Reflexionsverarbeitung bereit.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Aufzeichnung über die CLI



Die CLI

führt Aufzeichnungen über den Sensorpool des Backends durch (das Backend muss laufen – diese Befehle sind „Thin HTTP

-Clients“):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Ruft den Wert von `--sensor-id` aus `chloros-cli daq pool-list` ab. Zwei wissenswerte Standardwerte:

| Option | Standardwert |
| --- | --- |
| `--duration` | `0` – Aufzeichnung bis `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` auf dem Dateisystem des **Backends**, nicht auf dem der CLI

|

Die Unterscheidung zwischen den Ausgabeverzeichnissen ist von Bedeutung, wenn die CLI

-Anwendung auf ein Backend auf einem anderen Rechner verweist: Die Datei wird dort abgelegt, wo das Backend läuft. Befindet sich das Backend auf demselben Rechner, wird ein relatives `-o` (wie beispielsweise das oben genannte `./out`) relativ zu dem Ordner aufgelöst, in dem Sie den Befehl ausgeführt haben.

## Aufzeichnung aus Python

: `DAQSensorSession` (zurückgegeben von `chloros_sdk.connect_daq_sensor()`) gibt dieselbe gepoolte Aufzeichnung preis: `record_start(output_dir=None, device_name=None)` gibt den Dateipfad zurück, `record_stop()` gibt `{path, rows}` zurück. Die vollständige Sitzungs-API

finden Sie in der [SDK

-Referenz](../reference/sdk-reference.md). Die Direkt-Hardware-Klassen des SDK

s (nur Desktop-Installationen) schreiben Aufzeichnungen standardmäßig in `~/Documents/DAQ/`; für veröffentlichte Builds ist der oben genannte gepoolte Pfad der unterstützte Weg.

## Verwendung einer .daq-Datei bei der Verarbeitung

Um aus den Bilddaten die Reflexion zu berechnen, benötigt Chloros

die auf jede Belichtung abgestimmte nach unten gerichtete Bestrahlungsstärke:

* **Bewahren Sie die Datei „`.daq`“ zusammen mit den Bilddaten auf.**Bei der Verarbeitung ermittelt die Pipeline automatisch die**zeitstempelabgestimmte nach unten gerichtete Strahlung** aus einer aufgezeichneten `.daq`-Datei (beliebiges DAQ-Modell) – oder aus einer DAQ-M-eigenen `.csv`-Datei —, das sich neben den Bildern befindet. GUI-Aufzeichnungen erfüllen diese Voraussetzung automatisch, da sie dem Projekt sofort nach Beendigung hinzugefügt werden.
* **Die Kalibrierung wird bei Bedarf abgerufen.** Falls ein werkseitiges Kalibrierungspaket pro Kamera oder pro DAQ noch nicht lokal zwischengespeichert ist, ruft Chloros

es bei der ersten Verwendung automatisch aus der MAPIR

-Cloud ab (einmalige Internetverbindung erforderlich; zwischengespeichert unter `~/.chloros/`).
* **Live-Aufnahmen schreiben ihre eigene Begleitdatei.** Für jedes live erfasste Reflexionsbild wird der tatsächlich verwendete DAQ-Messwert als `.daq`-Datei im Ordner `daq/` der Erfassung gespeichert, sodass die Erfassung später ohne die ursprüngliche Aufzeichnung erneut verarbeitet werden kann.
* **Aufzeichnungen von den `chloros_scripts`-Recordern (1.3.0).**Eine von den öffentlichen [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)-Aufzeichnungsgeräten erstellte `.daq`-Datei wird als das DAQ-Modell aufgeführt, das sie erstellt hat, und ihr Obergrenze kann wie bei jeder anderen Aufzeichnung unter**Projekteinstellungen → DAQ-Lichtsensor** wie bei jeder anderen Aufzeichnung eingestellt werden. Diese Aufzeichnungsgeräte schreiben die rohen Sensorwerte; Chloros

ruft die Werkskalibrierung des Sensors über die serielle Schnittstelle ab und wendet sie an.
* **Wem der Messwert gehört (1.3.0).**Neben jeder Aufnahme „`.daq`“ gibt ein „`.daq.cameras.json`“ die Kamera(s) an, deren Bilder dieser Messwert beschreibt, und die Verarbeitung ordnet ihn**nur Bildern dieser Kameras** zu. Eine eigenständige Kamera ohne eigenen Lichtsensor übernimmt daher nicht mehr den Messwert einer anderen Kamera – ordnen Sie ihr einen Sensor zu oder zeichnen Sie einen auf. Eine mit dem DAQ selbst erstellte Aufzeichnung (GUI-Aufzeichnung, `daq pool-record`) sowie jede `.daq`-Datei ohne diese Datei (einschließlich älterer Aufnahmen) dient weiterhin allen Kameras.

## Die Strahlungsintensität wiederherstellen

Bei der Verarbeitung eines Projekts werden außerdem alle darin enthaltenen Lichtsensoraufzeichnungen in einen
`Light Sensor/`-Ordner neben den Bildprodukten exportiert. Hierfür sind **keine** Bilddaten erforderlich: Eine
allein aufgezeichnete Lichtsensoraufzeichnung ist eine vollständige Erfassung, und ein Ordner, der ausschließlich `.daq`-
Dateien enthält, ist eine gültige Eingabe. Ein Projekt, das nur Lichtsensoraufzeichnungen enthält, listet diese auf und
verarbeitet sie sowohl über die App als auch über die CLI

, und sie bleiben nach dem Lauf in der Liste stehen. Der Lauf
gibt an, wie viele Lichtsensorprodukte geschrieben wurden.

| Produkt | Was es ist |
| --- | --- |
| `<name>_calibrated.daq` | Ein erneut verarbeitbares Archiv im gleichen Schema wie eine Live-Aufzeichnung, das nun das Kalibrierungsbündel angibt, mit dem es erstellt wurde. Ein erneuter Import führt **nicht** zu einer zweiten Kalibrierung. |
| `<name>_calibrated.csv` | Spektrale Bestrahlungsstärke in W/m²/nm auf dem sensoreigenen Wellenlängengitter, eine Zeile pro Messwert, sowie photometrische Spalten: Gesamtleistung, photopische und skotopische Lux, PPFD mit Aufteilung in Blau/Grün/Rot und Spitzenwellenlänge. |

Ein DAQ-U oder DAQ-M, dessen Kalibrierungspaket nicht abgerufen werden kann – Sie sind offline oder
für diesen Sensor liegt keine Kalibrierung vor –, wird **mit einer Begründung übersprungen** und niemals
als „kalibrierte“ Datei mit Rohzählwerten ausgegeben. Stellen Sie eine Verbindung zum Internet her und führen Sie den Vorgang erneut aus. Ein DAQ-E
verfügt über eine eigene Kalibrierung, sodass diese nur benötigt wird, wenn das Gerät nicht verbunden ist und
lokal nichts zwischengespeichert ist.

### DAQ-A: Rohzählwerte und warum das die richtige Lösung ist

Der **DAQ-A** stammt aus der Zeit vor dem System der serienbezogenen Kalibrierungspakete und verfügt über kein Paket, das
abgerufen werden könnte. Das ist kein Versehen: Ein DAQ-A wird vor Ort anhand eines
Reflexionsziels kalibriert, und für die zielbasierte Kalibrierung wird nur die *relative*
Empfindlichkeit des Sensors benötigt – und genau das sind seine Rohzählwerte. Chloros

kalibriert heute damit.

Eine DAQ-A-Aufzeichnung lässt sich also exportieren, allerdings unter einem anderen Namen:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, nicht `_calibrated` – ein anderer Dateiname statt eines Flags innerhalb der Datei,
da die Angabe erhalten bleiben muss, wenn die Datei als reiner Name per E-Mail weitergeleitet wird. Der Header „`.csv`“
lautet „`raw spectral sensor counts (NOT irradiance)`“ und weist darauf hin, dass die Werte
**innerhalb** der Datei vergleichbar sind, nicht jedoch sensorübergreifend. Die Spalten, die nur
für die tatsächliche Bestrahlungsstärke von Bedeutung sind – Gesamtleistung, Lux, PPFD – bleiben leer und werden nicht
aus Zählwerten berechnet.

Ältere DAQ-A-SD-Aufzeichnungen (Schema v1.01 / v1.02) erfassen nur den Schreibzeitpunkt der Datei, nicht jedoch einen
Zeitstempel pro Messwert. Chloros

gleicht die Bilder nicht mit diesen ab – die Zuordnung eines Bildes zu einem
Schreibzeitpunkt wäre falsch, ohne dass dies jemals auffallen würde –, aber der Export liest sie einwandfrei ein und
die CSV

-Datei gibt an, auf welcher Uhr sie basieren.

Für den vollständigen Überblick über die Reflektanzmessung – Einzelsensor mit Kamera und Doppelsensor für Umgebungslicht/Objekt – siehe [Reflektanz-Workflows](reflectance.md).
