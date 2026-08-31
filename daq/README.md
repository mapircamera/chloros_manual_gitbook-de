# DAQ-Lichtsensoren

> **Sie suchen Informationen zur Hardware?**Die Sensoren selbst – Modelle, Montage, Abdeckungen, Anschlüsse, Stromversorgung und die SCANNER-App – sind im**[DAQ-Benutzerhandbuch](https://mapir.gitbook.io/daq)** dokumentiert. Dieses Kapitel behandelt deren Verwendung ab Chloros.

Die **DAQ**-Lichtsensoren in MAPIR messen das Umgebungslicht als radiometrisch kalibrierte Spektren. In Chloros erfüllen sie zwei Funktionen:

* **Ein eigenständiges Spektralmessgerät** – Live-Spektrumdiagramme, kolorimetrische Daten und `.daq`-Aufzeichnungen, alles über die [Registerkarte „Lichtsensoren“](gui.md), dem [CLI](cli-quick-start.md) oder dem Python SDK.
* **Eine Quelle für die nach unten gerichtete Bestrahlungsstärke zur Reflektanzberechnung** — während der Verarbeitung interpoliert Chloros Ihre `.daq`-Messwerte auf den Belichtungszeitstempel jeder AufnahmeBelichtungszeitstempel und wandelt mithilfe des gemessenen nach unten gerichteten Lichts die Strahlungsintensität der Kamera in Reflexionsgrad um (`--reflectance-source daq`); für kalibrierte Bänder ist kein Panel vor Ort erforderlich.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Drei Modelle, ein Datenformat

| Modell | Übertragung | Erkennung |
| --- | --- | --- |
| **DAQ-U** | USB (seriell) | Scan über serielle Schnittstelle |
| **DAQ-M** | Bluetooth Low Energy | BLE-Scan nach Namen |
| **DAQ-E** | Ethernet (IPv4, PoE-gespeist) | mDNS `_daq-e._tcp` (Hostname `daq-e-<id>.local`) |

Alle drei verwenden dasselbe Kommunikationsprotokoll und liefern identische Daten:

* Ein **135-Punkt-Spektrum von 340 bis 1010 nm in 5-nm-Schritten** sowie CIE-XYZ-Dreikomponentenwerte in jedem Datenframe.
* **Radiometrisch kalibrierte spektrale Bestrahlungsstärke in W/m²/nm** – das werkseitige Kalibrierungspaket jedes Geräts (zuzüglich seines aktiven Cap-Korrekturprofils) wird angewendet, bevor die Daten bei Ihnen ankommen.
* Das gleiche **`.daq`-Aufzeichnungsformat** (eine SQLite-Datei). Die nachgelagerte Verarbeitung ist identisch, unabhängig davon, über welchen Übertragungsweg die Datei erzeugt wurde.

Die Transportstacks (USB-Seriell, BLE, mDNS/Zeroconf) sind im Chloros-Backend gebündelt – es muss nichts installiert werden, um mit einem der drei Modelle über die GUI oder die CLI-Befehle von `pool-*` zu kommunizieren.

***

## Kalibrierter Bereich: 340–1010 nm gemeldet, ~374–974 nm kalibriert

Der Sensor meldet das gesamte Raster von 340–1010 nm, aber die NIST-rückführbare radiometrische Verstärkung erstreckt sich über etwa **374–974 nm**. Chloros lehnt die Division durch die absolute Reflexion für jedes Kameraband ab, bei dem weniger als die Hälfte des spektralen Gewichts innerhalb dieses kalibrierten Bereichs liegt; das übersprungene Band wird mit dem Überspringungsgrund `dls-uncalibrated-band-<nm>` gemeldet.

Unter den lieferbaren LATTICE-Filter-SKUs ist nur **F988** betroffen:

Die Reflexion von F988 wird mithilfe einer Reflexionsplatte vor Ort kalibriert: Da das Band außerhalb des kalibrierten Bereichs des DAQ-Lichtsensors liegt, wendet Chloros Ihre letzte Erfassung der Reflexionsplatte an und behält diese zwischen den Messungen bei.

Wird eine F988-Erfassung verarbeitet, wobei nur DAQ-Daten verfügbar sind, lehnt Chloros die DAQ-basierte Reflexion für dieses Band mit dem Überspringgrund `dls-uncalibrated-band-988` ab – der [Reflexionspanel-Workflow](../calibration-targets.md) ist der für F988 unterstützte Pfad.

***

## Sensor-IDs

Jedes DAQ meldet eine stabile Sensor-ID. Deren Format unterscheidet sich je nach Modell:

| Modell | ID-Format | Beispiel |
| --- | --- | --- |
| DAQ-U | 5 Oktette mit Bindestrichen | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 Oktette, mit Bindestrichen | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Die Sensor-ID ist:

* in jede von ihr aufgezeichnete `.daq`-Datei eingetragen,
* der Schlüssel, den Chloros verwendet, um das Werkskalibrierungspaket dieses Geräts abzurufen,
* der Wert, den Sie in den Befehlen CLI und `pool-*` an `--sensor-id` übergeben, und
* beim DAQ-E zusätzlich dessen mDNS-Hostnamen (`daq-e-def330.local`) – den Wert, den `--eth-host` akzeptiert.

***

## Werkskalibrierung und die Cloud

Jedes DAQ-Gerät wird einzeln werkseitig mit einer NIST-rückführbaren radiometrischen Kette kalibriert, und Chloros lädt das Kalibrierungsbündel jedes Geräts, das anhand seiner Sensor-ID zugeordnet ist. Der Kalibrierungsbericht (PDF) für jedes einzelne Gerät kann über die Sensoreinstellungen auf der [Registerkarte „Lichtsensoren“](gui.md) heruntergeladen werden.

{% hint style="warning" %}
**DAQ-U und DAQ-M benötigen für die Kalibrierung einen Cloud-Zugang.**Keines der beiden Modelle speichert Daten lokal: Ihre werkseitigen Kalibrierpakete befinden sich in der Cloud von MAPIR und werden anhand der Sensor-ID abgerufen (und anschließend lokal zwischengespeichert). Chloros benötigt eine Internetverbindung, um kalibrierte W/m²/nm-Daten von einem DAQ-U oder DAQ-M zu liefern.**DAQ-E bildet hier eine Ausnahme** – es speichert seine Kalibrierung auf dem Gerät selbst.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Wo die Aufzeichnungen gespeichert werden

| Oberfläche | Standard-Speicherort für `.daq` |
| --- | --- |
| GUI – Registerkarte „Lichtsensoren“ | `<project folder>/light_sensor/` (fertige Aufzeichnungen werden automatisch zum Projekt hinzugefügt) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` auf dem Rechner, auf dem das Backend läuft |

Jeder `.daq`-Dateiname enthält die Sensor-ID und einen Zeitstempel.

***

## In diesem Kapitel

* [**Die Registerkarte „DAQ“ in Chloros**](gui.md) – eine vollständige Anleitung zur Benutzeroberfläche: Anschließen der einzelnen Modelle, sensorbezogene Einstellungen, Spektrumdiagramme, kolorimetrische Live-Daten, Reflexionsmessung mit zwei Sensoren und Aufzeichnung.
* [**CLI Schnellstart (pool-\*)**](cli-quick-start.md) – Ansteuerung von DAQ-Sensoren über `chloros-cli daq pool-*`, den unterstützten Befehlszeilenpfad.
* [**Cap-Profile &amp; kalibrierter Bereich**](caps-and-range.md) — welche Caps es pro Modell gibt, wie man sie deklariert und der kalibrierte Spektralbereich im Detail.
* [**Aufzeichnung &amp; das .daq-Format**](recording.md) – das SQLite-Format von `.daq` und Aufzeichnungsabläufe.
* [**DAQ-E-Vernetzung und Zeitsynchronisation**](ethernet-ptp.md) – DAQ-E-Übertragungsmodi und PTP-Zeitsynchronisation.
* [**Reflexions-Workflows**](reflectance.md) – Verwendung von DAQ-Downwelling-Daten zur Ermittlung der Reflexion.
* Eine vollständige Dokumentation auf Flag-Ebene finden Sie in der [CLI-Referenz](../reference/cli-reference.md) (Abschnitt `chloros-cli daq`) und in der [SDK-Referenz](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`); beide sind so verfasst, dass sie direkt von KI-Assistenten verarbeitet werden können.
