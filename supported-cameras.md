---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Unterstützte Kameras

Chloros verarbeitet Bilddaten von zwei MAPIR-Kameraserien auf **allen Plattformen** (Windows, Linux amd64 und Linux arm64/Jetson):

* **Survey3** – Kameras der Typen Survey3W (breit) und Survey3N (schmal). Eingabe: `RAW+JPG`.
* **LATTICE**– M3C- und M3M-Multispektralkameramodule. Eingabe: `.tif`/`.tiff`-Aufnahmen. LATTICE-Kameras können auch**live** von Chloros aus gesteuert werden – über die Registerkarte „Kameras“ der Benutzeroberfläche (Windows) oder über `chloros-cli lattice` / Python und SDK (Windows und Linux) – einschließlich synchronisierter Mehrkamera-Arrays. Siehe den [LATTICE-Leitfaden](lattice/).

Die Verarbeitungspipeline akzeptiert auch `.dng`-Eingabedateien.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Hersteller</th><th width="250">Kameramodell</th><th width="138">Filtermodell</th><th width="187">Bildtyp</th></tr></thead><tbody><tr><td><strong>Hersteller</strong></td><td><strong>Kameramodell</strong></td><td><strong>Filtermodell</strong></td><td><strong>Bildtyp</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

Die LATTICE-Reihe ist ein modulares Multispektralkamerasystem, das auf dem Global-Shutter-Sensor IMX265 von Sony (3,1 MP, 3,45 µm Pixel) basiert. Jede Kamera speichert ihre Identität als Modellzeichenfolge:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros zeigt sie mit dem Präfix `LATT-` an (zum Beispiel `LATT-M3M-L41-F550`), und die Modellzeichenfolge steuert alle nachgelagerten Prozesse – Sensorprofil, Bandanordnung und Kalibrierung werden automatisch ermittelt; es muss pro Kamera nichts konfiguriert werden. Die Objektivnummer entspricht dem **horizontalen Sichtfeld in Grad**: `L41` = schmal 41°, `L87` = weit 87°.

Es gibt zwei Sensorkonfigurationen:

| Konfiguration | Sensor      | Filtertyp                           | Bänder pro Kamera                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayer-Farbe | Dreifach-Bandpass                       | 3 Spektralbänder aus einer einzigen Belichtung                                 |
| **M3M**       | Monochrom  | Einfacher Schmalband-Interferenzfilter | 1 kalibriertes Band – Kombination mehrerer M3M-Kameras für Vegetationsindizes |

### M3C (Bayer)-Filteroptionen

| Filter | Bänder (Bezeichnung bei Mittenwellenlänge in nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### M3M (Mono)-Filterkatalog – 23 Artikelnummern

Die F-Zahl ist die Artikelnummer; das gemessene Band (auf jedem kalibrierten Export eingeprägt) ist der Filterscan pro Charge:

| Artikelnummer    | Mitte (nm, gemessen) | FWHM Kanten (nm) | Breite (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominal)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominal)           | 867–893         | 26         |
| F905   | — (nominal)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_„Die Bandkanten werden als Werte der Halbwertsbreite aus den Filterscans pro Charge von MAPIR gemessen – dieselben Werte, die Chloros in jeden kalibrierten Export einfügt.“_ „— (nominal)“ = noch kein Chargenscan; bei diesen SKUs entspricht der angegebene Mittelpunkt der SKU-Nummer und die Breite dem Herstellerwert.

† „Die Reflexion von F988 wird mithilfe einer Reflexionsplatte in der Szene kalibriert: Da das Band außerhalb des kalibrierten Bereichs des DAQ-Lichtsensors liegt, wendet Chloros Ihre letzte Erfassung der Reflexionsplatte an und behält diese zwischen den Messungen bei.“ Siehe [Kalibrierungsziele](calibration-targets.md).

Informationen zur Live-Kamerasteuerung, zu Arrays, zur Netzwerkeinrichtung und zur radiometrischen Verarbeitungskette finden Sie im [LATTICE-Leitfaden](lattice/).
