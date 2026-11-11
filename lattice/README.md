# LATTICE-Kameras

LATTICE ist das modulare Multispektralkamerasystem vMAPIR für die landwirtschaftliche und wissenschaftliche Bildgebung. Jede LATTICE-Kamera basiert auf dem Global-Shutter-Sensor Sony IMX265 (**3,1 MP, 3,45 µm Pixel**) und wird über Ethernet als**GigE Vision**-Gerät angeschlossen.

Chloros 1.3.0 steuert LATTICE-Kameras in Echtzeit – Erkennung, Live-Vorschau, Aufnahme und synchronisierte Mehrkamera-Arrays – über drei Benutzeroberflächen:

| Benutzeroberfläche    | Wo                                                          | Plattformen                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Registerkarte **Kameras** in der Chloros-Seitenleiste                         | Windows 10/11 x64                                        |
| CLI        | Befehlsfamilie `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python-SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Auf der Suche nach der Hardware?**Kameramodule, Objektive, Filter und Bandbreiten, Rahmen und Befestigungen, Kabel, PoE- und Trigger-Verkabelung sind im [**LATTICE-Benutzerhandbuch**](https://mapir.gitbook.io/lattice-camera) dokumentiert. Dieses Kapitel behandelt die Ansteuerung der Kameras über Chloros.

LATTICE-Aufnahmen sind Standarddateien vom Typ `.tif`/`.tiff`, und Chloros verarbeitet diese stets ausgehend von der Rohaufnahme. Die vollständige Befehls- und API-Oberfläche finden Sie in der [CLI-Referenz](../reference/cli-reference.md) und der [SDK-Referenz](../reference/sdk-reference.md).

## Zwei Sensorkonfigurationen

| Konfiguration | Sensor       | Filter                                | Was eine Kamera liefert                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-Farbe | Dreifach-Bandpassfilter                |**Drei kalibrierte Bänder aus einer einzigen Belichtung**                 |
| **M3M**| Monochrom   | einzelner Schmalband-Interferenzfilter |**Ein kalibriertes Band**; Kombination mehrerer M3M-Kameras für Indizes |

Da eine M3M-Kamera hinter einem einzigen Filter monochrom ist, erhält jedes Band seine eigene Belichtung. Eine M3C-Kamera deckt alle drei ihrer Bänder mit einer einzigen Sensorbelichtung ab.

## Modellzeichenfolgen und Namensgebung

Jede Kamera speichert ihre Identität in GenICam als Modellzeichenfolge „`DeviceUserID`“:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros zeigt diese mit dem Präfix „`LATT-`“ an (zum Beispiel „`LATT-M3M-L87-F450`“). Derselbe String „`LATT-…`“ wird in das EXIF-Tag „`Model`“ jedes Exports geschrieben und dient in verarbeiteten Projekten als Name des Ausgabeordners der Kamera.

| Komponente | Werte                                                   | Bedeutung                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Bayer-Farbfilter / Monochrom                                                                          |
| Objektiv      | `L41` / `L87`                                            | Die Zahl gibt das **horizontale Sichtfeld in Grad** an: L41 = schmal (41°), L87 = breit (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) oder `F<nm>` (M3M) | Siehe [Filter &amp; Spektralbänder](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Die Modellzeichenfolge steuert alle nachfolgenden Schritte: Chloros ermittelt das Sensorprofil, die Bandanordnung und die Werkskalibrierung aus `DeviceUserID` + `DeviceSerialNumber`. Pro Kamera muss nichts konfiguriert werden – siehe [Anschließen von Kameras](connecting.md).

## Filter und Spektralbänder

Bandmitten, FWHM-Kanten und der vollständige M3M-Katalog mit 23 Artikelnummern sind Produktspezifikationen und daher im Hardware-Handbuch zu finden: [**Filter &amp; Spektralbänder**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Was auf der Softwareseite zählt: Der Filtercode in der Modellzeichenfolge bestimmt, welche Produkte Chloros erstellen kann. RGB-Filterkameras (`FRGB`) liefern ausschließlich debayerte und Vorschau-Produkte – Radiance- und Reflektanzwerte pro Band sind für einen Breitbandsensor nicht aussagekräftig, daher lässt Chloros diese weg und weist darauf hin. Jeder andere Filter liefert die vollständige Kette von Radiance über Reflektanz bis hin zum Index.

## Radiometrische Kalibrierung auf einen Blick

Jede LATTICE-Kamera wird werkseitig individuell anhand einer NIST-rückführbaren Kette kalibriert und wird mit einem kameraspezifischen Zertifikat ausgeliefert. Was dies umfasst, wie es gemessen wird und welche Genauigkeit Sie angeben können, finden Sie im Hardware-Handbuch: [**Radiometrische Werkskalibrierung**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Auf der Softwareseite ist entscheidend, dass Chloros die richtige Kalibrierung ermittelt, sobald eine Kamera verbunden wird, und die angewendeten Koeffizienten bei jedem Export festschreibt – siehe [Kameras verbinden](connecting.md).

## In diesem Kapitel

* [Kameras verbinden](connecting.md) – automatische Erkennung, der Verbindungsdialog in der GUI, entsprechende CLI- und SDK-Funktionen sowie die Art und Weise, wie die werkseitige Kalibrierung (Kamera-Paket vs. Cloud) ermittelt wird, wenn eine Kamera verbunden wird.
* [Kameraeinstellungen](camera-settings.md) – alle Steuerelemente auf der Registerkarte „Kameras“: die Kameraliste, der Einstellungsbereich pro Kamera und der Bereich für Array-Einstellungen (einschließlich ROI-Editor und Ausrichtung).
* [Aufnahmeeinstellungen und -modi](capture.md) – „Alles aufnehmen“, Einzel-/Serien-/Intervallaufnahme, Exportformate und die Struktur des Aufnahmeordners.
* [Mehrkamera-Arrays](arrays.md) – der Array-Verbindungsassistent, die Synchronisationskabel-Prüfung, Bandbreite, PTP und die Live-Ansicht des Arrays.
* [Mono-Kameras &amp; Vegetationsindizes](mono-indices.md) – M3M-Kameras und der Workflow „Ausrichten → Stapeln → Indexieren“.

Die vollständige Befehlsoberfläche finden Sie in der [CLI-Referenz](../reference/cli-reference.md) und der [SDK-Referenz](../reference/sdk-reference.md).
