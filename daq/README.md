# DAQ-Lichtsensoren

> **Sie suchen nach Informationen zur Hardware?**Die Sensoren selbst – Modelle, Befestigung, Abdeckungen, Anschlüsse, Stromversorgung und die SCANNER-App – sind im**[DAQ-Benutzerhandbuch](https://mapir.gitbook.io/daq)**dokumentiert. Dieses Kapitel behandelt deren Verwendung in Chloros. Die**DAQ**-Lichtsensoren von

MAPIR messen das Umgebungslicht als radiometrisch kalibrierte Spektren. In Chloros erfüllen sie zwei Funktionen:

* **Ein eigenständiges Spektralmessgerät** – Live-Spektrumdiagramme, kolorimetrische Daten und `.daq`-Aufzeichnungen, alles über die [Registerkarte „Lichtsensoren“](gui.md), der [CLI](cli-quick-start.md) oder dem Python-SDK.
* **Eine Quelle für die nach unten gerichtete Bestrahlungsstärke zur Reflexionsberechnung** – während der Verarbeitung interpoliert Chloros Ihre `.daq`-Messwerte auf den Belichtungszeitstempel jeder Aufnahme und verwendet das gemessene nach unten gerichtete Licht, um die Kamerastrahlungsintensität in Reflektanz (`--reflectance-source daq`) umzuwandeln; für kalibrierte Bänder ist kein In-Scene-Panel erforderlich.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Drei Modelle, ein Datenformat

| Modell | Übertragung | Erkennung |
| --- | --- | --- |
| **DAQ-U** | USB (seriell) | Scan über serielle Schnittstelle |
| **DAQ-M** | Bluetooth Low Energy | BLE-Scan nach Namen |
| **DAQ-E** | Ethernet (IPv4, PoE-gespeist) | mDNS `_daq-e._tcp` (Hostname `daq-e-<id>.local`) |**DAQ-E-S (neu in 1.3.0).** Ein DAQ-E mit festem Diffusor und Neigungssensor. Chloros listet ihn als `DAQ-E-S` auf und betreibt ihn genau wie einen DAQ-E, mit der Ausnahme, dass er keine Kappenkorrektur vornimmt und jeder Messwert die Neigung, Roll- und Nickwinkel des Geräts enthält. Siehe [DAQ-E-S: fester Diffusor und Neigung](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Alle drei verwenden dasselbe Kommunikationsprotokoll und liefern identische Daten:

* Ein **135-Punkt-Spektrum von 340 bis 1010 nm in 5-nm-Schritten** sowie CIE-XYZ-Dreistimuluswerte in jedem Frame.
* **Radiometrisch kalibrierte spektrale Bestrahlungsstärke in W/m²/nm** – das werkseitige Kalibrierungspaket jedes Geräts (zuzüglich seines aktiven Kappenkorrekturprofils) wird angewendet, bevor die Daten bei Ihnen eintreffen.
* Das gleiche **`.daq`-Aufzeichnungsformat** (eine SQLite-Datei). Die nachgelagerte Verarbeitung ist identisch, unabhängig davon, welcher Übertragungsweg die Datei erzeugt hat.

Die Transportstacks (USB-Seriell, BLE, mDNS/zeroconf) sind im Chloros-Backend gebündelt – es muss nichts installiert werden, um über die GUI oder die `pool-*`-Befehle der CLI mit einem der drei Modelle zu kommunizieren.

***

## Kalibrierter Bereich: 340–1010 nm gemeldet, ~374–974 nm kalibriert

Der Sensor meldet das gesamte Raster von 340–1010 nm, aber die NIST-rückführbare radiometrische Verstärkung erstreckt sich über etwa **374–974 nm**. Chloros lehnt die Aufteilung nach absoluter Reflexion für jedes Kameraband ab, dessen spektrales Gewicht weniger als die Hälfte innerhalb dieses kalibrierten Bereichs ausmacht; das übersprungene Band wird mit dem Überspringgrund `dls-uncalibrated-band-<nm>` gemeldet.

Unter den lieferbaren LATTICE-Filter-SKUs ist nur **F988** betroffen:

Die Reflexanz von F988 wird mithilfe einer Reflexanzplatte vor Ort kalibriert: Da das Band außerhalb des kalibrierten Bereichs des DAQ-Lichtsensors liegt, wendet Chloros Ihre letzte Erfassung der Reflexanzplatte an und behält diese zwischen den Messungen bei.

Wird eine F988-Erfassung verarbeitet, für die nur DAQ-Daten verfügbar sind, lehnt Chloros die DAQ-basierte Reflexion für dieses Band mit dem Überspringgrund `dls-uncalibrated-band-988` ab ab – der [Reflexionspanel-Workflow](../calibration-targets.md) ist der für F988 unterstützte Weg.

***

## Sensor-IDs

Jedes DAQ meldet eine stabile Sensor-ID. Deren Format unterscheidet sich je nach Modell:

| Modell | ID-Format | Beispiel |
| --- | --- | --- |
| DAQ-U | 5 Oktette mit Bindestrichen | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 Oktette mit Bindestrichen | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Die Sensor-ID ist:

* in jede von ihm aufgezeichnete `.daq`-Datei eingetragen,
* der Schlüssel, den Chloros verwendet, um das Werkskalibrierungspaket dieses Geräts abzurufen,
* der Wert, den Sie in den CLI-Befehlen an `--sensor-id` übergeben, und
* beim DAQ-E zusätzlich dessen mDNS-Hostname (`daq-e-def330.local`) – der Wert, den `--eth-host` akzeptiert.

***

## Werkskalibrierung und die Cloud

Jedes DAQ-Gerät wird einzeln im Werk mit einer auf das NIST rückführbaren radiometrischen Kette kalibriert, und Chloros lädt für jedes Gerät das auf dessen Sensor-ID abgestimmte Kalibrierungspaket. Der Kalibrierungsbericht pro Gerät (PDF) kann über die Sensoreinstellungen auf der [Registerkarte „Lichtsensoren“](gui.md) heruntergeladen werden.

{% hint style="warning" %}
**DAQ-U und DAQ-M benötigen für die Kalibrierung einen Cloud-Zugang.**Keines der beiden Modelle speichert Daten lokal: Ihre werkseitigen Kalibrierungsdatenpakete befinden sich in der MAPIR-Cloud und werden anhand der Sensor-ID abgerufen (und anschließend lokal zwischengespeichert). Chloros benötigt eine Internetverbindung, um kalibrierte W/m²/nm-Daten von einem DAQ-U oder DAQ-M zu liefern.**DAQ-E bildet hier eine Ausnahme** – es speichert seine Kalibrierung auf dem Gerät selbst.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Wo die Aufzeichnungen gespeichert werden

| Oberfläche | Standard-Speicherort für `.daq` |
| --- | --- |
| GUI – Registerkarte „Lichtsensoren“ | `<project folder>/light_sensor/` (fertige Aufzeichnungen werden automatisch zum Projekt hinzugefügt) |
| CLI – `daq pool-record` | `~/Documents/DAQ Live View/` auf dem Rechner, auf dem das Backend läuft (ein relativer Pfad `-o` wird relativ zu dem Ordner aufgelöst, in dem Sie den Befehl ausgeführt haben, wenn sich das Backend auf demselben Computer befindet) |

Jeder Dateiname `.daq` enthält die Sensor-ID und einen Zeitstempel.

***

## In diesem Kapitel

* [**Die Registerkarte „DAQ“ in Chloros**](gui.md) – eine vollständige Anleitung zur Benutzeroberfläche: Anschließen der einzelnen Modelle, sensorbezogene Einstellungen, Spektraldiagramme, kolorimetrische Live-Daten, Reflexionsmessung mit zwei Sensoren und Aufzeichnung.
* [**Schnellstart für die CLI (pool-\*)**](cli-quick-start.md) – Ansteuerung von DAQ-Sensoren über `chloros-cli daq pool-*`, den unterstützten Befehlszeilenpfad.
* [**Cap-Profile & kalibrierter Bereich**](caps-and-range.md) – welche Caps es pro Modell gibt, wie man sie deklariert und der kalibrierte Spektralbereich im Detail.
* [**Aufzeichnung & das .daq-Format**](recording.md) – das SQLite-Format von `.daq` und Aufzeichnungsabläufe.
* [**DAQ-E-Vernetzung & Zeitsynchronisation**](ethernet-ptp.md) – DAQ-E-Übertragungsmodi und PTP-Zeitsynchronisation.
* [**Reflexions-Workflows**](reflectance.md) – Verwendung von DAQ-Downwelling-Daten zur Ermittlung der Reflexion.
* Eine vollständige Dokumentation auf Flag-Ebene finden Sie in der [CLI-Referenz](../reference/cli-reference.md) (Abschnitt `chloros-cli daq`) und in der [SDK-Referenz](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), die beide so verfasst sind, dass sie direkt von KI-Assistenten verarbeitet werden können.
