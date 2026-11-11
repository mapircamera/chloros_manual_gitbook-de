# Kappenprofile und kalibrierter Messbereich

> Die Abdeckungen selbst – welche Abdeckung zu welchem Sensor gehört, wie sie montiert werden und ihr optisches Verhalten – sind im **[DAQ-Benutzerhandbuch](https://mapir.gitbook.io/daq)** dokumentiert. Auf dieser Seite geht es um die *Angabe* der montierten Abdeckung gegenüber Chloros
, wodurch die Korrektur erst korrekt erfolgt.

Die werkseitige radiometrische Kalibrierung jedes DAQ-Lichtsensors beschreibt den *unverhüllten* Sensor. Die physische Kappe, die über den Diffusor gestülpt wird, verändert das vom Sensor erfasste Licht, weshalb Chloros
zusätzlich zum Kalibrierungsbündel ein werkseitig gemessenes **Kappenkorrekturprofil** anwendet. Die Angabe der richtigen Kappe ist Teil der Erfassung kalibrierter Daten – auf dieser Seite wird erläutert, welche Kappen es für die einzelnen Modelle gibt, wie man sie angibt und wie der kalibrierte Spektralbereich des Sensors tatsächlich aussieht.

## Verfügbarkeit der Kappen nach Modell

| Kappenprofil (`cap_id`) | Physikalische Kappe | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Sunshine-Kosinus-Korrekturkappe (**Standard bei jedem Modell**) | Ja | Ja | Ja |
| `fov_15` / `fov_45` / `fov_90` | Sichtfeldbegrenzende Kegel (15° / 45° / 90°) | Ja | — | Ja |
| `fov_30` / `fov_60` | Sichtfeldbegrenzende Kegel (30° / 60°) | Ja | — | — |
| `none` | Keine Kappe montiert | — | — | Ja |

Modellspezifische Hinweise:

* **DAQ-M verfügt über ein Kappenprofil: `sunshine_cosine`.** „Bare-plus-Sunshine-Kappe“ ist die Produktbezeichnung, und ein DAQ-M ohne Kappe benötigt kein Geometrieprofil.
* **Ein DAQ-U ohne Kappe ist ein echtes „Bare“-Modell** – es benötigt überhaupt kein Geometrieprofil, weshalb es dafür kein `none`-Profil gibt.
* **`none` auf einem DAQ-E ist KEINE No-Op.** Der versenkte, glasabgedeckte Diffusor des DAQ-E verfügt über eine eigene Geometriekorrektur, sodass „keine Kappe“ bei diesem Modell selbst ein gemessenes Profil darstellt.
* Ein **unverkleideter DAQ-E kann bei keinem Sonnenstand direktes Sonnenlicht messen** – die Sunshine-Kappe ist die Standardkonfiguration. Planen Sie keine Arbeiten im Freien mit einem unverkleideten DAQ-E.

In den sensorbezogenen Einstellungen der Benutzeroberfläche (Zahnradsymbol auf der Registerkarte „Lichtsensoren“) bietet das Dropdown-Menü **„Cap“** bei DAQ-U und DAQ-M auch die Option „Keine (unverhüllter Sensor)“ an – bei diesen beiden Modellen bedeutet „unverhüllt“ gemäß den obigen Hinweisen lediglich, dass keine Kappenkorrektur angewendet wird. Wählen Sie diese Option nur, wenn die Kappe physisch entfernt wurde.

## Angabe der Kappe – und warum dies wichtig ist

**Die angegebene Kennung `cap_id` muss mit der Kappe übereinstimmen, die physisch auf dem Sensor sitzt.** Weder der Sensor noch die Software können die aufgesetzte Kappe erkennen. Die Angabe hat zwei Auswirkungen:

1. Die **Live-Korrektur**, die auf jedes Spektrum angewendet wird.
2. Der **Kappenstempel, der in jede `.daq`-Aufzeichnung geschrieben wird** und auf den sich die nachgelagerte Reflexionsverarbeitung verlässt.

Die Sunshine-Kappe dämpft **konstruktionsbedingt um etwa das 12-Fache**, sodass eine Aufzeichnung mit einer falsch angegebenen Kappe die Spektren um etwa diesen Faktor falsch skaliert. Geben Sie Kapppenänderungen unverzüglich an.

### Einstellen der Kappe

GUI: Registerkarte „Lichtsensoren“ → Zahnrad-Symbol in der Sensorzeile → Dropdown-Menü **„Kappe“**. Die Standardeinstellung für jedes Modell ist `sunshine_cosine` – eine Standardannahme (DAQ-U, DAQ-M und DAQ-E werden ohne Kappe ausgeliefert, und eine Aufzeichnung geht davon aus, dass eine Kappe verwendet wurde, bis Sie eine auswählen) –, und die Auswahl bleibt für das Projekt erhalten.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(Backend muss laufen):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

Die CLI
akzeptiert syntaktisch die vollständige `cap_id`-Liste (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); Jedes Profil wird bei der Verbindung anhand des Sensormodells validiert, sodass eine nicht verfügbare Cap-ID (z. B. eine „E-only“-ID auf einem DAQ-U) mit einem eindeutigen Fehler fehlschlägt, anstatt eine falsche Korrektur vorzunehmen. Der Backend-Standardwert, wenn nichts übergeben wird, ist `sunshine_cosine`.

Python
-SDK
: Benennen Sie die Kapazität bei der Verbindung mit `connect_daq_sensor(cap_id="sunshine_cosine")` oder tauschen Sie sie zur Laufzeit mit `DAQSensorSession.set_cap("fov_45")` aus; siehe die [SDK
-Referenz](../reference/sdk-reference.md#daq-sensor-sessions).

Fortgeschritten: Profile sind in der Chloros
-Installation unter `daq/cap_profiles/<u|m|e>/<cap_id>.json` enthalten und können benutzerspezifisch unter `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json` überschrieben werden.

Unabhängig von den Obergrenzen erhalten Sensoren, die noch nie neu kalibriert wurden, automatisch eine geringfügige, aus der Flotte abgeleitete Korrektur des Dunkeloffsets – ohne dass ein Eingreifen des Benutzers erforderlich ist.

## DAQ-E-S: fester Diffusor und feste Neigung

Der **DAQ-E-S** ist ein DAQ-E mit zwei Unterschieden, die für Chloros
von Bedeutung sind. Alles andere (Erkennung, Verbindung, PTP, Aufzeichnung, Geräteeinstellungen) funktioniert genau wie bei einem DAQ-E.

* **Sein Diffusor ist fest eingebaut, sodass keine Kappenkorrektur erforderlich ist.** Der Diffusor war bereits am Gerät angebracht, als dessen Werkskalibrierung gemessen wurde, sodass diese Kalibrierung bereits die tatsächliche nach unten gerichtete Bestrahlungsstärke angibt. Chloros
wendet unabhängig von den Eingaben kein Cap-Profil darauf an: Ein dafür benanntes Cap wird mit einer Meldung ignoriert, in den Aufzeichnungen wird angegeben, dass das Cap aus dem Modell stammt, und die Warnung wegen eines angenommenen Caps wird dafür niemals ausgelöst. Unter „Projekteinstellungen“ → „DAQ-Lichtsensor“ wird in der entsprechenden Zeile kein Cap angezeigt, und die projektweite Cap-Überschreibung wirkt sich nicht auf dieses Gerät aus. Wenden Sie keinen Cap manuell auf dessen Daten an: Die darüber liegende Sunshine-Korrektur würde etwa das 11-Fache des tatsächlichen Werts anzeigen.
* **Es verfügt über einen 3-Achsen-Beschleunigungsmesser.** Die Kosinus-Charakteristik des Diffusors gilt nur, solange er waagerecht ausgerichtet ist; daher meldet ein geneigter Sensor eine falsche Bestrahlungsstärke, ohne dass andere Symptome auftreten. Jeder DAQ-E-S-Messwert enthält daher seine Neigung, Roll- und Nickwinkel, und ein mit einem solchen Sensor aufgezeichneter `.daq` speichert diese neben jedem Spektrum (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …) gespeichert, sodass man eine Wolke von einer verschobenen Halterung unterscheiden kann.

Informationen zu jedem Lagefeld und dazu, wie ein Fehler im Lagestrom gemeldet wird, finden Sie unter [DAQ-E-S in der CLI
-Referenz](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Leistung der Sunshine-Kappe (Konfiguration im Freien)

Zahlen, auf denen Sie Verfahren aufbauen können:

| Eigenschaft | Wert |
| --- | --- |
| Sichtfeld | 180° halbkugelförmig |
| Kosinus-Reaktionsfehler | ≤ ±4 % bis zu einem Einfallswinkel von 60°; ≤ ±4,5 % bis zu 70° |
| Grenze für tiefstehende Sonne | Nicht empfohlen unter ~15° Sonnenhöhe |
| Dämpfung | ~12× (konstruktionsbedingt) |
| Wiederholgenauigkeit beim Wiederaufsetzen der Kappe | ≈ 1,5 % |
| Quantitative Bestrahlungsstärke | Durchschnitt aus **≥ 15 s** Messwerten (Geräteeigenschaft, kein Defekt) |

Verwenden Sie für alle quantitativen Bestrahlungsstärkewerte – einschließlich Reflexionsreferenzwerten – einen Durchschnitt aus mindestens 15 Sekunden Messwerten anstelle eines einzelnen Bildes.

## Kalibrierter Spektralbereich

| Eigenschaft | Wert |
| --- | --- |
| Spektrale Abtastung | 340–1010 nm in 5-nm-Schritten (135 Punkte) |
| Radiometrisch kalibrierter Bereich | **~374–974 nm** (softwareseitig erzwungen) |

Der Sensor liefert das vollständige Raster von 340–1010 nm, aber die NIST-rückführbare radiometrische Verstärkung erstreckt sich über ~374–974 nm. Chloros
**lehnt die Aufteilung nach absoluter Reflexion** für jedes Kameraband ab, bei dem weniger als die Hälfte des spektralen Gewichts innerhalb dieses Bereichs liegt, und meldet den Überspringgrund `dls-uncalibrated-band-<nm>`, anstatt ein unkalibriertes Ergebnis zu liefern. Unter den im Handel erhältlichen Kamera-SKUs liegt nur der Filter F988 außerhalb dieses Bereichs; er nutzt stattdessen den Reflektanz-Panel-Workflow – siehe [Reflektanz-Workflows](reflectance.md).

Informationen zu Sensormodellen, Transporten und Sensor-IDs finden Sie in der [DAQ-Übersicht](README.md). Informationen dazu, wie der Cap-Stempel während der Verarbeitung verbraucht wird, finden Sie unter [Aufzeichnung und das .daq-Format](recording.md).
