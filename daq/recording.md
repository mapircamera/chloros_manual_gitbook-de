# Aufzeichnung und das .daq-Format

Eine `.daq`-Datei ist das Aufzeichnungsformat des Lichtsensors von Chloros: eine **SQLite-Datenbank** mit kalibrierten Spektralbildern von einem DAQ-Sensor. Wenn Sie während einer Aufnahmesitzung eine solche Datei aufzeichnen, kann die Reflexions-Pipeline später jedes Bild durch die genau in diesem Moment gemessene nach unten gerichtete Bestrahlungsstärke dividieren.

## Was eine .daq-Datei enthält

| Eigenschaft | Wert |
| --- | --- |
| Container | SQLite-Datenbank, eine Datei pro Sensor pro Aufzeichnung |
| Dateiname | Enthält die **Sensor-ID**und einen**Zeitstempel**, z. B. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum pro Bild | 135 Punkte, 340–1010 nm in 5-nm-Schritten, plus CIE-XYZ-Dreikomponentenwerte |
| Einheiten | Kalibrierte spektrale Bestrahlungsstärke, **W/m²/nm** (werkseitiges Kalibrierungspaket + angewandtes Kap-Profil) |
| Eingetragene Metadaten | Sensor-ID (der Schlüssel zum Abrufen der Werkskalibrierung dieses Geräts) und das aktuell gültige Kappenprofil – siehe [Kappenprofile &amp; kalibrierter Bereich](caps-and-range.md) |

Das Format ist bei DAQ-U, DAQ-M und DAQ-E identisch, sodass es für die nachgelagerte Verarbeitung keine Rolle spielt, welches Transportmodul die Daten aufgezeichnet hat.

Für kalibrierte Aufzeichnungen ist das Werkskalibrierungspaket des Sensors erforderlich. Bei DAQ-U und DAQ-M ruft das Backend das Paket anhand der Sensor-ID aus der Cloud von MAPIR ab (die Aufzeichnung wird verweigert, wenn dies nicht möglich ist); DAQ-E-Geräte sind davon ausgenommen, da sie ihre Kalibrierung auf dem Gerät selbst speichern.

## Aufzeichnung über die GUI

Für die Aufzeichnung über die GUI muss ein **Projekt geöffnet** sein (andernfalls sind die Aufzeichnungsschaltflächen deaktiviert):

* **Alle aufzeichnen / Alle stoppen** — oben in der Seitenleiste „Lichtsensoren“; startet oder stoppt gleichzeitig eine `.daq`-Aufzeichnung auf allen angeschlossenen Sensoren.
* **Aufzeichnen / Aufzeichnung beenden** – pro Sensor, im Einstellungsmodal mit dem Zahnrad-Symbol. Während der Aufzeichnung wird in den Live-Info-Zeilen des Sensors eine rote „REC“-Anzeige angezeigt.

Die Dateien werden in `<project>/light_sensor/` gespeichert, und wenn eine Aufzeichnung beendet wird – sei es über „Stopp“, „Alle stoppen“ oder durch das Trennen eines Aufzeichnungssensors –, wird die fertige Datei `.daq` **automatisch dem geöffneten Projekt hinzugefügt**. Sie erscheint in der Dateiliste des Projekts, ohne dass ein manueller Hinzufügungsschritt erforderlich ist, und steht bereits für die Reflexionsverarbeitung bereit.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Aufzeichnung vom CLI

Das CLI zeichnet über den Sensorpool des Backends auf (das Backend muss laufen – diese Befehle sind „schlanke“ HTTP-Clients):

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

Den Wert von `--sensor-id` von `chloros-cli daq pool-list` abrufen. Zwei wissenswerte Standardwerte:

| Option | Standardwert |
| --- | --- |
| `--duration` | `0` – aufzeichnen bis `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` auf dem Dateisystem des **Backends**, nicht auf dem von CLI |

Die Unterscheidung des Ausgabeverzeichnisses ist von Bedeutung, wenn das CLI auf ein Backend auf einem anderen Rechner verweist: Die Datei wird dort abgelegt, wo das Backend läuft.

## Aufzeichnung von Python

`DAQSensorSession` (zurückgegeben von `chloros_sdk.connect_daq_sensor()`) stellt dieselbe gepoolte Aufzeichnung bereit: `record_start(output_dir=None, device_name=None)` gibt den Dateipfad zurück, `record_stop()` gibt `{path, rows}` zurück. Die vollständige Sitzung API finden Sie in der [SDK-Referenz](../reference/sdk-reference.md). Die Direkt-Hardware-Klassen von SDK (nur Desktop-Installationen) schreiben Aufzeichnungen standardmäßig in `~/Documents/DAQ/`; für veröffentlichte Builds ist der oben angegebene Pool-Pfad der unterstützte Weg.

## Verwendung einer .daq-Datei bei der Verarbeitung

Um aus Bilddaten die Reflexion zu berechnen, benötigt Chloros die für jede Belichtung angepasste nach unten gerichtete Strahlungsintensität:

* **Bewahren Sie die `.daq`-Datei zusammen mit den Bilddaten auf.**Bei der Verarbeitung ermittelt die Pipeline automatisch die**zeitstempelabgestimmte nach unten gerichtete Strahlung** aus einer aufgezeichneten `.daq`-Datei (beliebiges DAQ-Modell) – oder aus einer DAQ-M-eigenen `.csv`-Datei —, das sich neben den Bildern befindet. GUI-Aufzeichnungen erfüllen diese Voraussetzung automatisch, da sie dem Projekt sofort nach Beendigung hinzugefügt werden.
* **Die Kalibrierung wird bei Bedarf abgerufen.** Wenn ein werkseitiges Kalibrierungspaket pro Kamera oder pro DAQ noch nicht lokal zwischengespeichert ist, ruft Chloros es bei der ersten Verwendung automatisch aus der Cloud von MAPIR ab (einmalige Internetverbindung erforderlich; zwischengespeichert unter `~/.chloros/`).
* **Live-Aufnahmen schreiben ihre eigene Begleitdatei.** Für jedes live erfasste Reflexionsbild wird der tatsächlich verwendete DAQ-Messwert als `.daq`-Begleitdatei neben dem Bild gespeichert, sodass die Aufnahme später ohne die ursprüngliche Aufzeichnung erneut verarbeitet werden kann.

## Die Bestrahlungsstärke wiederherstellen

Bei der Verarbeitung eines Projekts werden außerdem alle darin enthaltenen Lichtsensoraufzeichnungen in einen
Ordner „`Light Sensor/`“ neben den Bildprodukten exportiert. Dafür werden **keine** Bilddaten benötigt: Ein
allein geflogener Lichtsensor stellt eine vollständige Aufnahme dar, und ein Ordner, der ausschließlich `.daq`-
Dateien enthält, ist eine gültige Eingabe. Der Lauf berichtet, wie viele Lichtsensor-Produkte geschrieben wurden.

| Produkt | Was es ist |
| --- | --- |
| `<name>_calibrated.daq` | Ein nachbearbeitbares Archiv im gleichen Schema wie eine Live-Aufzeichnung, das nun das Kalibrierungsbundle angibt, mit dem es erstellt wurde. Ein erneuter Import führt **nicht** zu einer zweiten Kalibrierung. |
| `<name>_calibrated.csv` | Spektrale Bestrahlungsstärke in W/m²/nm auf dem sensoreigenen Wellenlängengitter, eine Zeile pro Messwert, sowie photometrische Spalten: Gesamtleistung, photopische und skotopische Lux, PPFD mit Aufschlüsselung nach Blau/Grün/Rot sowie Spitzenwellenlänge. |

Ein DAQ-U oder DAQ-M, dessen Kalibrierungsbundle nicht abgerufen werden kann – Sie sind offline oder
für diesen Sensor liegt keine Kalibrierung vor –, wird **mit einer Begründung übersprungen** und niemals
als „kalibrierte“ Datei mit Rohzählwerten ausgegeben. Stellen Sie eine Verbindung zum Internet her und führen Sie den Vorgang erneut aus. Ein DAQ-E
verfügt über eine eigene Kalibrierung, sodass diese nur benötigt wird, wenn das Gerät nicht verbunden ist und
lokal nichts zwischengespeichert ist.

### DAQ-A: Rohzählwerte und warum das die richtige Lösung ist

Der **DAQ-A** stammt aus der Zeit vor dem System der serienbezogenen Kalibrierungspakete und verfügt über kein Paket, das
abgerufen werden könnte. Das ist kein Versehen: Ein DAQ-A wird vor Ort anhand eines
Reflexionsziels kalibriert, und bei der zielbasierten Kalibrierung wird nur die *relative*
Empfindlichkeit des Sensors benötigt – und genau das sind seine Rohzählwerte. Chloros kalibriert heute mit ihnen.

Eine DAQ-A-Aufzeichnung lässt sich also exportieren, allerdings unter einem anderen Namen:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, nicht `_calibrated` – ein anderer Dateiname statt eines Kennzeichens innerhalb der Datei,
da die Angabe auch dann erhalten bleiben muss, wenn die Datei als reiner Dateiname per E-Mail weitergeleitet wird. Der Header „`.csv`“
lautet „`raw spectral sensor counts (NOT irradiance)`“ und weist darauf hin, dass die Werte
nur **innerhalb** der Datei vergleichbar sind, nicht jedoch sensorübergreifend. Die Spalten, die nur
für die tatsächliche Bestrahlungsstärke von Bedeutung sind – Gesamtleistung, Lux, PPFD – werden leer gelassen, anstatt
aus Zählwerten berechnet zu werden.

Ältere DAQ-A-SD-Aufzeichnungen (Schema v1.01 / v1.02) zeichnen nur den Schreibzeitpunkt der Datei auf, nicht einen
Zeitstempel pro Messwert. Chloros gleicht die Bilder nicht mit diesen ab – die Zuordnung eines Bildes zu einem
Schreibzeitpunkt wäre falsch, ohne dass dies jemals auffallen würde –, aber der Export liest sie einwandfrei ein und
die Datei CSV gibt an, auf welcher Uhr sie basiert.

Für den vollständigen Überblick über die Reflektanzmessung – Einzelsensor mit Kamera und Doppelsensor für Umgebungslicht/Objekt – siehe [Reflektanz-Workflows](reflectance.md).
