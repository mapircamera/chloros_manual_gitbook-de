# LATTICE-Kameras

LATTICE ist das modulare Multispektralkamerasystem von MAPIR für die landwirtschaftliche und wissenschaftliche Bildgebung. Jede LATTICE-Kamera basiert auf dem Global-Shutter-Sensor Sony IMX265 (**3,1 MP, 3,45 µm Pixel**) und wird über Ethernet als**GigE Vision**-Gerät angeschlossen.

Chloros 1.2.0 steuert LATTICE-Kameras in Echtzeit – Erkennung, Live-Vorschau, Aufnahme und synchronisierte Mehrkamera-Arrays – über drei Benutzeroberflächen:

| Benutzeroberfläche    | Wo                                                          | Plattformen                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Registerkarte **Kameras** in der Chloros-Seitenleiste                         | Windows 10/11 x64                                        |
| CLI        | Befehlsfamilie `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |

> **Sie suchen nach der Hardware?**Kameramodule, Objektive, Filter und Bandbreiten, Rahmen und Befestigungen, Kabel, PoE- und Trigger-Verkabelung sind im [**LATTICE-Benutzerhandbuch**](https://mapir.gitbook.io/lattice-camera) dokumentiert. Dieses Kapitel behandelt die Ansteuerung der Kameras über Chloros.

LATTICE-Aufnahmen sind Standarddateien im Format `.tif`/`.tiff`, und Chloros verarbeitet diese stets ausgehend von der Rohaufnahme. Siehe die [CLI-Referenz](../reference/cli-reference.md) und die [SDK-Referenz](../reference/sdk-reference.md) für die vollständigen Befehle und die API-Oberfläche.

## Zwei Sensorkonfigurationen

| Konfiguration | Sensor       | Filter                                | Was eine Kamera liefert                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-Farbe | Dreifach-Bandpassfilter                |**Drei kalibrierte Bänder aus einer einzigen Belichtung**                 |
| **M3M**| Monochrom   | Ein einzelner Schmalband-Interferenzfilter |**Ein kalibriertes Band**; mehrere M3M-Kameras für Indizes kombinieren |

Da eine M3M-Kamera hinter einem einzigen Filter monochrom arbeitet, erhält jedes Band eine eigene Belichtung. Eine M3C-Kamera deckt alle drei ihrer Bänder mit einer einzigen Sensorbelichtung ab.

## Modellzeichenfolgen und Benennung

Jede Kamera speichert ihre Identität in GenICam als Modellzeichenfolge „`DeviceUserID`“:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros zeigt sie mit dem Präfix `LATT-` an (zum Beispiel `LATT-M3M-L87-F450`). Derselbe String „`LATT-…`“ wird in den EXIF-Tag „`Model`“ jedes Exports geschrieben und dient in bearbeiteten Projekten als Name des Ausgabeordners der Kamera.

| Komponente | Werte                                                   | Bedeutung                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Bayer-Farbe / Monochrom                                                                          |
| Objektiv      | `L41` / `L87`                                            | Die Zahl gibt das **horizontale Sichtfeld in Grad** an: L41 = schmal (41°), L87 = breit (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) oder `F<nm>` (M3M) | Siehe [Filter &amp; Spektralbänder](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Die Modellzeichenfolge bestimmt alle nachfolgenden Schritte: Chloros ermittelt das Sensorprofil, die Bandanordnung und die Werkskalibrierung aus `DeviceUserID` + `DeviceSerialNumber`. Pro Kamera gibt es nichts zu konfigurieren – siehe [Anschluss von Kameras](connecting.md).

## Filter und Bänder

Bandmitten, FWHM-Bandgrenzen und der vollständige M3M-Katalog mit 23 Artikelnummern sind Produktspezifikationen und daher im Hardware-Handbuch zu finden: [**Filter &amp; Spektralbänder**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Was auf der Softwareseite wichtig ist: Der Filtercode in der Modellzeichenfolge bestimmt, welche Produkte Chloros erstellen kann. RGB-Filterkameras (`FRGB`) geben ausschließlich debayerte und Vorschau-Produkte aus – Radianz und Reflektanz pro Band sind für einen Breitbandsensor nicht aussagekräftig, daher überspringt Chloros diese Werte und gibt dies an. Jeder andere Filter liefert die vollständige Kette von Radianz → Reflektanz → Index.

## Radiometrische Kalibrierung auf einen Blick

Jede LATTICE-Kamera wird werkseitig individuell anhand einer auf das NIST rückführbaren Kette kalibriert und wird mit einem kameraspezifischen Zertifikat ausgeliefert. Was dies umfasst, wie es gemessen wird und welche Genauigkeit Sie angeben können, finden Sie im Hardware-Handbuch: [**Werkseitige radiometrische Kalibrierung**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Auf der Softwareseite ist entscheidend, dass Chloros beim Anschließen einer Kamera die richtige Kalibrierung ermittelt und die angewendeten Koeffizienten bei jedem Export festschreibt – siehe [Kameras anschließen](connecting.md).

## In diesem Kapitel

* [Anschließen von Kameras](connecting.md) – automatische Erkennung, der Verbindungsdialog in der Benutzeroberfläche, CLI/SDK-Entsprechungen sowie die Ermittlung der Werkskalibrierung (kamerainternes Paket vs. Cloud) beim Anschließen einer Kamera.

Weitere LATTICE-Themen – Kameraeinstellungen und Live-Steuerung, Aufnahmemodi, Mehrkamera-Arrays sowie Mono- (M3M-)Verarbeitung und Indizes – werden in eigenen Abschnitten dieses Handbuchs behandelt, und die vollständige Befehlsoberfläche finden Sie in der [CLI-Referenz](../reference/cli-reference.md) und [SDK-Referenz](../reference/sdk-reference.md).
