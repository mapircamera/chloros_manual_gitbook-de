# Kappenprofile und kalibrierter Messbereich

> Die Kappen selbst – welche Kappe zu welchem Sensor gehört, wie sie montiert werden und wie sie sich optisch verhalten – sind im **[DAQ-Benutzerhandbuch](https://mapir.gitbook.io/daq)** dokumentiert. Diese Seite befasst sich mit der *Angabe* der montierten Kappe an Chloros, wodurch die Korrektur korrekt erfolgt.

Die werkseitige radiometrische Kalibrierung jedes DAQ-Lichtsensors beschreibt den *unverkleideten* Sensor. Die physische Kappe, die über dem Diffusor angebracht ist, verändert das vom Sensor erfasste Licht, daher wendet Chloros zusätzlich zum Kalibrierungsbündel ein werkseitig gemessenes **Kappenkorrekturprofil** an. Die Angabe der richtigen Kappe ist Teil der Erfassung kalibrierter Daten – diese Seite behandelt, welche Kappen pro Modell existieren, wie sie anzugeben sind und wie der kalibrierte Spektralbereich des Sensors tatsächlich aussieht.

## Verfügbarkeit der Kappen nach Modell

| Kappenprofil (`cap_id`) | Physikalische Kappe | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | „Sunshine“-Kosinus-Korrekturkappe (**Standard bei jedem Modell**) | Ja | Ja | Ja |
| `fov_15` / `fov_45` / `fov_90` | Kegel zur Einschränkung des Sichtfelds (15° / 45° / 90°) | Ja | — | Ja |
| `fov_30` / `fov_60` | Sichtfeldbegrenzende Kegel (30° / 60°) | Ja | — | — |
| `none` | Keine Kappe montiert | — | — | Ja |

Modellspezifische Hinweise:

* **DAQ-M verfügt über ein Kappenprofil: `sunshine_cosine`.** „Bare-plus-Sunshine-Kappe“ ist die Produktbezeichnung, und ein „Bare“-DAQ-M benötigt kein Geometrieprofil.
* **Ein „Bare“-DAQ-U ist ein echtes „Bare“-Modell** – es benötigt überhaupt kein Geometrieprofil, weshalb es dafür auch kein `none`-Profil gibt.
* **`none` bei einem DAQ-E ist KEINE No-Op.** Der versenkte, glasabgedeckte Diffusor des DAQ-E verfügt über eine eigene Geometriekorrektur, sodass „keine Kappe“ bei diesem Modell selbst ein gemessenes Profil darstellt.
* Ein **unverkleideter DAQ-E kann bei keinem Sonnenstand direkte Sonneneinstrahlung messen** – die Sunshine-Kappe ist die Standardkonfiguration für den Feldeinsatz. Planen Sie keine Arbeiten im Freien mit einem unverkleideten DAQ-E ein.

In den sensorbezogenen Einstellungen der Benutzeroberfläche (Zahnradsymbol auf der Registerkarte „Lichtsensoren“) bietet das Dropdown-Menü **„Cap“** bei DAQ-U und DAQ-M auch die Option „Keine (unverhüllter Sensor)“ an – bei diesen beiden Modellen bedeutet „unverhüllt“ gemäß den obigen Hinweisen lediglich, dass keine Kappenkorrektur angewendet wird. Wählen Sie diese Option nur, wenn die Kappe physisch entfernt wurde.

## Angabe der Kappe – und warum dies wichtig ist

**Die angegebene Kennung `cap_id` muss mit der Kappe übereinstimmen, die physisch auf dem Sensor sitzt.** Weder der Sensor noch die Software können die aufgesetzte Kappe erkennen. Die Angabe hat zwei Auswirkungen:

1. Die **Live-Korrektur**, die auf jedes Spektrum angewendet wird.
2. Der **Kappenstempel, der in jede `.daq`-Aufzeichnung geschrieben wird** und auf den sich die nachgelagerte Reflexionsverarbeitung stützt.

Die Sunshine-Kappe dämpft **konstruktionsbedingt um etwa das 12-Fache**, sodass eine Aufzeichnung mit einer falsch deklarierten Kappe die Spektren um etwa diesen Faktor falsch skaliert. Geben Sie Änderungen der Kappe unverzüglich an.

### Einstellen der Kappe

GUI: Registerkarte „Lichtsensoren“ → Zahnrad-Symbol in der Sensorzeile → Dropdown-Menü **„Kappe“**. Die Standardeinstellung für jedes Modell ist `sunshine_cosine` (alle DAQ-Sensoren werden mit installiertem Kosinuskorrektor ausgeliefert), und die Auswahl bleibt im Projekt erhalten.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (Backend muss laufen):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

Das Modell CLI akzeptiert syntaktisch die gesamte `cap_id`-Liste (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); jedes Profil wird bei der Verbindung anhand des Sensormodells validiert, sodass eine nicht verfügbare Cap-ID (z. B. eine reine E-ID auf einem DAQ-U) mit einem eindeutigen Fehler fehlschlägt, anstatt eine falsche Korrektur vorzunehmen. Die Backend-Standardeinstellung, wenn nichts übergeben wird, ist `sunshine_cosine`.

Python SDK Hinweis: `cap_id` ist **kein** SDK-Regler – `connect_daq_sensor()` / `DAQSensorSession` stellen keinen Cap-Parameter bereit. Wählen Sie die Obergrenze über die oben genannten CLI-Befehle oder das Dropdown-Menü der Benutzeroberfläche aus; siehe [SDK-Referenz](../reference/sdk-reference.md).

Fortgeschritten: Profile werden im Rahmen der Chloros-Installation unter `daq/cap_profiles/<u|m|e>/<cap_id>.json` bereitgestellt und können pro Benutzer unter `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json` überschrieben werden.

Unabhängig von den Obergrenzen erhalten Sensoren, die noch nie neu kalibriert wurden, automatisch eine geringfügige, aus der Flotte abgeleitete Verfeinerung des Dunkel-Offsets – ohne dass ein Eingriff des Benutzers erforderlich ist.

## Leistung der Sonneneinstrahlungs-Obergrenze (Konfiguration für den Außenbereich)

Zahlen, auf denen Sie Verfahren aufbauen können:

| Eigenschaft | Wert |
| --- | --- |
| Sichtfeld | 180° halbkugelförmig |
| Kosinus-Reaktionsfehler | ≤ ±4 % bis zu einem Einfallswinkel von 60°; ≤ ±4,5 % bis zu 70° |
| Grenze bei tief stehender Sonne | Nicht empfohlen unter ~15° Sonnenhöhe |
| Dämpfung | ~12× (konstruktionsbedingt) |
| Wiederholgenauigkeit beim Wiederanbringen der Kappe | ≈ 1,5 % |
| Quantitative Bestrahlungsstärke | Durchschnitt aus **≥ 15 s** Messwerten (Geräteeigenschaft, kein Defekt) |

Verwenden Sie für alle quantitativen Bestrahlungsstärkewerte – einschließlich Reflexionsreferenzwerten – einen Durchschnitt aus mindestens 15 Sekunden Messwerten anstelle eines einzelnen Bildes.

## Kalibrierter Spektralbereich

| Eigenschaft | Wert |
| --- | --- |
| Spektrale Abtastung | 340–1010 nm in 5-nm-Schritten (135 Punkte) |
| Radiometrisch kalibrierter Bereich | **~374–974 nm** (durch die Software erzwungen) |

Der Sensor liefert das gesamte Raster von 340–1010 nm, aber die NIST-rückführbare radiometrische Verstärkung erstreckt sich über ~374–974 nm. Chloros **verweigert die Aufteilung nach absoluter Reflexion** für jedes Kameraband, bei dem weniger als die Hälfte des spektralen Gewichts innerhalb dieses Bereichs liegt, und meldet den Überspringgrund `dls-uncalibrated-band-<nm>`, anstatt ein unkalibriertes Produkt zu erzeugen. Unter den lieferbaren Kamera-SKUs liegt nur der Filter F988 außerhalb dieses Bereichs; er verwendet stattdessen den Reflexionspanel-Workflow – siehe [Reflexions-Workflows](reflectance.md).

Informationen zu Sensormodellen, Transporten und Sensor-IDs finden Sie in der [DAQ-Übersicht](README.md). Informationen dazu, wie der Cap-Stempel während der Verarbeitung verbraucht wird, finden Sie unter [Aufzeichnung und das .daq-Format](recording.md).
