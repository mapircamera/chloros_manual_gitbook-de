---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formeln für multispektrale Indizes

Die folgenden Indexformeln verwenden eine Kombination aus den durchschnittlichen Transmissionsbereichen der Survey3-Filter:

<table><thead><tr><th align="center">Survey3-Filterfarbe</th><th width="196.199951171875" align="center">Survey3-Filtername</th><th width="159.800048828125" align="center">Transmissionsbereich (FWHM)</th><th align="center">Durchschnittliche Transmission</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Bei Verwendung dieser Formeln kann der Name auf „\_1“ oder „\_2“ enden, was angibt, welcher NIR-Filter – entweder NIR1 oder NIR2 – verwendet wurde.

Bei LATTICE M3C-Kameras (Bayer-Dreifach-Bandpass) verwendet dieselbe Index-Engine die M3C-Filterbänder:

| M3C-Filter | Band 1 (Mitte/FWHM) | Band 2 (Mitte/FWHM) | Band 3 (Mitte/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M-Kameras sind Einbandkameras (ein Schmalbandfilter pro Kamera), daher werden für ein einzelnes M3M-Bild keine Multiband-Indizes berechnet. Um Indizes mit M3M zu berechnen, kombinieren Sie zwei oder mehr Kameras zu einem ausgerichteten Multiband-Stack und verwenden Sie die LATTICE-Index-Engine (`chloros-cli lattice index` oder den Live-Index-Rechner der GUI).

***

## Wo die einzelnen Indexnamen funktionieren

Chloros verfügt über **drei** Indexoberflächen, deren voreingestellte Listen nicht identisch sind. In diesem Abschnitt können Sie überprüfen, ob ein Name an der von Ihnen geplanten Stelle funktioniert.

| Wo Sie sich befinden | Welche Liste gilt | Anzahl |
| --- | --- | --- |
| Projekteinstellungen → Index → Index hinzufügen (GUI) | Oberfläche 1 | 27 |
| Bildbetrachter [Index/LUT-Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Oberfläche 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Oberfläche 2 | 22 |
| SDK `process_folder(indices=[...])` | Oberfläche 2 | 22 |
| `chloros-cli lattice index --preset` | Oberfläche 3 | 22 (eine andere 22) |
| Registerkarte „Kameras“ – Live-Index-Rechner | Oberfläche 3 | 22 (eine andere 22) |

Surface 1 und 2 verarbeiten **jeweils ein Bild von einer Kamera**, wobei die Symbol-Slots `x`/`y`/`z`(/`a`) an die Filterkanäle dieser Kamera gebunden sind. Oberfläche 3 arbeitet mit einem**ausgerichteten Multiband-Stack** – mehreren LATTICE-Kameras, die zu einem Cube ko-registriert wurden – und bezieht sich auf Kanäle anhand ihrer Namen in Kleinbuchstaben.

### 1. GUI-Projekteinstellungen / Dropdown-Menü „Sandbox“ im Bildbetrachter – 27 Formeln

Das Dropdown-Menü listet sie in dieser Reihenfolge auf (es handelt sich um die Reihenfolge der Einfügung, nicht um eine alphabetische Sortierung):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

In der GUI ziehen Sie die Filterkanäle Ihrer Kamera auf die Band-Slots der Formel, sodass jede Formel mit jeder von Ihrer Kamera unterstützten Bandzuordnung verwendet werden kann. Von Ihnen gespeicherte benutzerdefinierte Formeln werden unterhalb dieser Liste angehängt.

Die **fünf ausschließlich in der GUI verfügbaren** Formeln – jene, die die CLI-/SDK-Liste „`--indices`“ nicht akzeptiert – sind wie folgt implementiert:

| Nur in der GUI verfügbare Voreinstellung | Formel (wie implementiert) | Slots |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (vier Slots) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Die jeweilige Zuordnung ist weiter unten auf dieser Seite in einem eigenen Abschnitt angegeben (zum Beispiel erwartet GARI x=Green, y=NIR, z=Blue, a=Red). GARI ist die einzige Formel in Chloros, die einen vierten Platz verwendet.

### 2. CLI- / SDK-`--indices`-Namenserweiterung – 22 Voreinstellungen

Die Option `chloros-cli process --indices` (und der SDK-Parameter `indices`) akzeptiert folgende Voreinstellungsnamen:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Unbekannte Indexnamen werden ohne Meldung übersprungen.** Ein Name, der nicht in dieser Liste enthalten ist (einschließlich der fünf ausschließlich in der GUI verfügbaren Formeln `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` sowie alle benutzerdefinierten Formeln, die Sie in der GUI gespeichert haben) wird lediglich mit einem Protokollhinweis verworfen – der Lauf wird ohne diesen Index fortgesetzt und wird weiterhin als erfolgreich gemeldet. Der Hinweis wird wie folgt ausgegeben:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Beim Abgleich der Namen wird auf Groß- und Kleinschreibung nicht geachtet, nachdem Leerzeichen entfernt wurden; daher sind `ndvi`, `NDVI` und ` NDVI ` dieselbe Voreinstellung. Eine Voreinstellung wird ebenfalls übersprungen, wenn sie ein Band benötigt, das der Filter Ihrer Kamera nicht bereitstellt.
{% endhint %}

Die genauen Formeln wie implementiert (die Symbole `x`/`y`/`z` sind Band-Slots; die Standardzuordnung wird pro Voreinstellung angezeigt):

| Voreinstellung | Formel (wie implementiert) | Standardfilter | Slots (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | 

Wenn Sie einen bloßen Namen wie `NDVI` übergeben, muss Chloros entscheiden, welchen Kanal welcher Datei jedes Symbol ausliest. Dazu verwendet es diese Tabelle, die einen Filtercode der Array-Position des jeweiligen Kanals zuordnet:

| Filtercode | Kanal → Array-Index |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` wird als Alias für Orange akzeptiert, ebenfalls 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Der **Standardfilter** der Voreinstellung (die Spalte „Standardfilter“ oben) wird verwendet, wenn das Projekt Bilder mit diesem Filter enthält. Ist dies nicht der Fall, durchsucht Chloros die tatsächlich im Projekt vorhandenen Filter in der Reihenfolge `RGN, OCN, NGB, RGB, RE, NIR` und wählt den ersten aus, der alle Kanäle bereitstellen kann, die die Voreinstellung benötigt. Ist dies bei keinem der Fall, wird die Voreinstellung für diesen Durchlauf verworfen. Aus diesem Grund liefert „`NDVI`“, das für einen reinen OCN-Datensatz angefordert wird, dennoch ein sinnvolles Ergebnis – es wird an die Orange- und NIR-Positionen von OCN gebunden.

LATTICE-M3C-Modellzeichenfolgen enthalten den Filter mit dem Präfix „`F`“ (`LATT-M3C-L41-FRGN`), doch das Präfix wird beim Auslesen des Filtercodes aus dem Bild entfernt, sodass eine FRGN-Kamera die obige Zeile `RGN` auflöst und keine besondere Behandlung erfordert.

### 3. LATTICE-Index-Engine (`lattice index --preset`, Live-Index-Rechner) – 22 Voreinstellungen

Die LATTICE-Engine arbeitet mit ausgerichteten Multiband-Stacks (Live-Arrays oder exportierte Multiband-TIFFs) und verwendet Kanalnamen in Kleinbuchstaben (`red`, `green`, `blue`, `red_edge`, `nir`). Ihre Voreinstellungsliste unterscheidet sich von den beiden oben genannten:

| Voreinstellung | Formel | Kanäle |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | rot, NIR |
| GNDVI | `(nir - green) / (nir + green)` | grün, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | blau, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | Rotkante, NIR |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | Blau, Grün, Nahinfrarot |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | Rot, Nahinfrarot |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | rot, NIR |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | rot, NIR |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | blau, rot, NIR |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | rot, NIR |
| CVI | `nir * red / green**2` | rot, grün, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | rot, NIR |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | rot, NIR |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | blau, rot, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | rot, grün, blau |
| NGRDI | `(green - red) / (green + red)` | rot, grün |
| VARI | `(green - red) / (green + red - blue)` | rot, grün, blau |
| TGI | `green - 0.39*red - 0.61*blue` | rot, grün, blau |
| EXG | `2*green - red - blue` | rot, grün, blau |
| CIRE | `(nir / red_edge) - 1` | rot\_Kante, NIR |
| CIGREEN | `(nir / green) - 1` | grün, NIR |
| NDWI | `(green - nir) / (green + nir)` | grün, nir |

Führen Sie `chloros-cli lattice index --list-presets` aus, um diese Tabelle aus Ihrer installierten Version auszudrucken, und `--list-gradients` für die verfügbaren Farbverläufe. Bei den Kanalsymbolen wird zwischen Groß- und Kleinschreibung unterschieden; sie müssen mit den Namen der Voreinstellungen in Kleinbuchstaben übereinstimmen (z. B. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI – Chlorophyll-Vegetationsindex

Dieser Index reagiert empfindlich auf den Chlorophyllgehalt der Blätter über einen weiten Bereich von Kronendichten hinweg. Er nutzt die NIR-, Rot- und Grün-Bänder.

$$
CVI = NIR * {Red \over Green^{2}}
$$

In der GUI sowie in der CLI-/SDK-Voreinstellungsliste lautet die Symbolformel `(z/y)*(x/y)` mit x=Red, y=Green, z=NIR (die Bänder einer RGN-Kamera); die Voreinstellung der LATTICE-Index-Engine lautet `nir * red / green**2`. Jede Fläche berechnet denselben Index.

_Quelle: Vincini, M., E. Frazzi und P. D’Alessio. „A Broad-Band Leaf Chlorophyll Vegetation Index at the Canopy Scale.“ Precision Agriculture 9 (2008): 303–319._

***

## ENDVI – Enhanced Normalized Difference Vegetation Index (erweiterter normalisierter Differenzvegetationsindex)

Dieser Index nutzt zusätzlich zu NIR und Grün den Blaukanal und ist besonders bei Kameras mit NGB-Filter beliebt, bei denen das Blauband das Rotband ersetzt.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Die Implementierung erfolgt über die Symbolformel `((x+y)-(2*z))/((x+y)+(2*z))` – weisen Sie die NIR- und Green-Kanäle Ihrer Kamera den x/y-Slots und Blue dem z-Slot zu (bei einer NGB-Kamera: x=NIR, y=Green, z=Blue).

***

## EVI – Enhanced Vegetation Index

Dieser Index wurde ursprünglich für die Verwendung mit MODIS-Daten als Verbesserung gegenüber dem NDVI entwickelt, indem das Vegetationssignal in Gebieten mit hohem Blattflächenindex (LAI) optimiert wurde. Er ist besonders nützlich in Regionen mit hohem LAI, in denen der NDVI möglicherweise gesättigt ist. Er nutzt den blauen Reflexionsbereich, um Bodenhintergrundsignale zu korrigieren und atmosphärische Einflüsse, einschließlich Aerosolstreuung, zu reduzieren.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-Werte sollten für Vegetationspixel im Bereich von 0 bis 1 liegen. Helle Objekte wie Wolken und weiße Gebäude sowie dunkle Objekte wie Wasser können zu anomalen Pixelwerten in einem EVI-Bild führen. Bevor Sie ein EVI-Bild erstellen, sollten Sie Wolken und helle Objekte aus dem Reflektanzbild ausblenden und optional die Pixelwerte auf einen Bereich von 0 bis 1 beschränken.

_Quelle: Huete, A., et al. „Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Waldbedeckungsindex 1

_Nur GUI – nicht als CLI-/SDK-Voreinstellung „`--indices`“ verfügbar._

Dieser Index unterscheidet Waldbestände von anderen Vegetationstypen mithilfe multispektraler Reflektanzbilder, die ein „Red-Edge“-Band enthalten.

$$
FCI1 = Red * RedEdge
$$

Bewaldete Gebiete weisen aufgrund der geringeren Reflektanz von Bäumen und der Schattenbildung innerhalb des Baumkronendachs niedrigere FCI1-Werte auf.

_Quelle: Becker, Sarah J., Craig S.T. Daughtry und Andrew L. Russ. „Robust forest cover indices for multispectral images.“ Photogrammetric Engineering & Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Waldbedeckungsindex 2

_Nur GUI – nicht als CLI- oder SDK-Voreinstellung „`--indices`“ verfügbar._

Dieser Index unterscheidet Waldkronen von anderen Vegetationstypen anhand von multispektralen Reflektanzbildern, die kein „Red-Edge“-Band enthalten.

$$
FCI2 = Red * NIR
$$

Bewaldete Gebiete weisen aufgrund der geringeren Reflektanz von Bäumen und des Vorhandenseins von Schatten innerhalb des Baumkronendachs niedrigere FCI2-Werte auf.

_Quelle: Becker, Sarah J., Craig S.T. Daughtry und Andrew L. Russ. „Robust forest cover indices for multispectral images.“ Photogrammetric Engineering & Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Global Environmental Monitoring Index

_Nur GUI – nicht als CLI- oder SDK-Voreinstellung „`--indices`“ verfügbar._

Dieser nichtlineare Vegetationsindex wird zur globalen Umweltüberwachung anhand von Satellitenbildern verwendet und versucht, atmosphärische Effekte zu korrigieren. Er ähnelt dem NDVI, reagiert jedoch weniger empfindlich auf atmosphärische Einflüsse. Da er durch kahlen Boden beeinflusst wird, wird seine Verwendung in Gebieten mit spärlicher oder mäßig dichter Vegetation nicht empfohlen.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Dabei gilt:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Quelle: Pinty, B., und M. Verstraete. „GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites“. Vegetation 101 (1992): 15–20._

***

## GARI – Green Atmospherically Resistant Index

_Nur GUI – nicht als CLI-/SDK-Voreinstellung `--indices` verfügbar._

Dieser Index reagiert empfindlicher auf einen breiten Bereich von Chlorophyllkonzentrationen und weniger empfindlich auf atmosphärische Einflüsse als der NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Die Gamma-Konstante ist eine Gewichtungsfunktion, die von den Aerosolbedingungen in der Atmosphäre abhängt. ENVI verwendet einen Wert von 1,7, was dem von Gitelson, Kaufman und Merzylak empfohlenen Wert entspricht (1996, Seite 296).

_Literaturhinweis: Gitelson, A., Y. Kaufman und M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.“ Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green-Chlorophyll-Index

Dieser Index wird verwendet, um den Chlorophyllgehalt der Blätter bei einer Vielzahl von Pflanzenarten abzuschätzen.

$$
GCI = {NIR \over Green} - 1
$$

Die Verwendung breiter Wellenlängenbereiche im nahen NIR- und Grünbereich ermöglicht eine bessere Vorhersage des Chlorophyllgehalts und sorgt gleichzeitig für eine höhere Empfindlichkeit sowie ein besseres Signal-Rausch-Verhältnis.

_Literaturhinweis: Gitelson, A., Y. Gritz und M. Merzlyak. „Zusammenhänge zwischen dem Chlorophyllgehalt der Blätter und der spektralen Reflexion sowie Algorithmen zur zerstörungsfreien Chlorophyllbestimmung in Blättern höherer Pflanzen.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Leaf Index

Dieser Index wurde ursprünglich für den Einsatz mit einer digitalen RGB-Kamera zur Messung der Weizenbedeckung entwickelt, wobei die digitalen Werte (DNs) für Rot, Grün und Blau im Bereich von 0 bis 255 liegen.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-Werte reichen von -1 bis +1. Negative Werte stehen für Boden und nicht lebende Objekte, während positive Werte grüne Blätter und Stängel darstellen.

_Quelle: Louhaichi, M., M. Borman und D. Johnson. „Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.“ Geocarto International 16, Nr. 1 (2001): 65–70._

***

## GNDVI – Greenr normalisierter Differenzvegetationsindex

Dieser Index ähnelt dem NDVI, misst jedoch anstelle des roten Spektrums das grüne Spektrum von 540 bis 570 nm. Dieser Index reagiert empfindlicher auf die Chlorophyllkonzentration als der NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Quelle: Gitelson, A., und M. Merzlyak. „Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.“ Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green-Optimierter, bodenangepasster Vegetationsindex

Dieser Index wurde ursprünglich mit Farb-Infrarot-Aufnahmen entwickelt, um den Stickstoffbedarf von Mais vorherzusagen. Er ähnelt dem OSAVI, ersetzt jedoch das rote Band durch das grüne.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Quelle: Sripada, R., et al. „Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.“ Doktorarbeit, North Carolina State University, 2005._

***

## GRVI – Green-Ratio-Vegetationsindex

Dieser Index reagiert empfindlich auf die Photosyntheseraten im Walddach, da die Reflexionswerte für Grün und Rot stark von Veränderungen der Blattpigmente beeinflusst werden.

$$
GRVI = {NIR \over Green }
$$

_Quelle: Sripada, R., et al. „Luftbildaufnahmen im Farb-Infrarotbereich zur Bestimmung des Stickstoffbedarfs von Mais zu Beginn der Vegetationsperiode.“ Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Soil Adjusted Vegetation Index

Dieser Index wurde ursprünglich mithilfe von Farb-Infrarot-Fotografie entwickelt, um den Stickstoffbedarf von Mais vorherzusagen. Er ähnelt dem SAVI, ersetzt jedoch das Rotband durch das Grünband.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Quelle: Sripada, R. et al. „Bestimmung des Stickstoffbedarfs von Mais während der Vegetationsperiode mittels Luftbild-Farb-Infrarotfotografie.“ Doktorarbeit, North Carolina State University, 2005._

***

## LAI – Blattflächenindex

Dieser Index wird zur Abschätzung der Blattbedeckung sowie zur Vorhersage des Pflanzenwachstums und des Ertrags verwendet. ENVI berechnet den grünen LAI anhand der folgenden empirischen Formel von Boegh et al. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Dabei ist EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hohe LAI-Werte liegen typischerweise im Bereich von etwa 0 bis 3,5. Wenn die Szene jedoch Wolken und andere helle Merkmale enthält, die gesättigte Pixel erzeugen, können die LAI-Werte 3,5 überschreiten. Idealerweise sollten Sie Wolken und helle Merkmale aus Ihrer Szene ausblenden, bevor Sie ein LAI-Bild erstellen.

_Quelle: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde und A. Thomsen. „Luftgestützte Multispektraldaten zur Quantifizierung des Blattflächenindex, der Stickstoffkonzentration und der Photosyntheseeffizienz in der Landwirtschaft.“ Remote Sensing of Environment 81, Nr. 2–3 (2002): 179–193._

***

## LCI – Blattchlorophyllindex

_Nur GUI – nicht als CLI-/SDK-Voreinstellung `--indices` verfügbar._

Dieser Index wird zur Abschätzung des Chlorophyllgehalts in höheren Pflanzen verwendet und reagiert empfindlich auf Schwankungen der Reflexion, die durch die Absorption von Chlorophyll verursacht werden.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Quelle: Datt, B. „Remote Sensing of Water Content in Eucalyptus Leaves.“ Journal of Plant Physiology 154, Nr. 1 (1999): 30–36._

***

## MNLI – Modifizierter nichtlinearer Index

Dieser Index ist eine Weiterentwicklung des nichtlinearen Index (NLI), bei der der bodenbereinigte Vegetationsindex (SAVI) einbezogen wird, um den Bodenhintergrund zu berücksichtigen. ENVI verwendet einen Wert von 0,5 für den Faktor zur Anpassung an den Kronendachhintergrund (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Quelle: Yang, Z., P. Willis und R. Mueller. „Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.“ Tagungsband des Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modifizierter bodenangepasster Vegetationsindex 2

Dieser Index ist eine vereinfachte Version des von Qi et al. vorgeschlagenen MSAVI-Index (1994) vorgeschlagen wurde und eine Weiterentwicklung des Soil Adjusted Vegetation Index (SAVI) darstellt. Er reduziert Bodenrauschen und erhöht den Dynamikbereich des Vegetationssignals. MSAVI2 basiert auf einer induktiven Methode, die keinen konstanten _L_-Wert (wie bei SAVI) verwendet, um gesunde Vegetation hervorzuheben.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Quelle: Qi, J., A. Chehbouni, A. Huete, Y. Kerr und S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.“ Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Modified Simple Ratio

Dieser Index ist eine Modifikation des einfachen NIR/Red-Verhältnisses, die darauf abzielt, dessen Zusammenhang mit biophysikalischen Parametern zu linearisieren, und ist bei höheren Vegetationsdichten empfindlicher als der NDVI.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Quelle: Chen, J. „Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.“ Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normalisierte Differenz RedEdge

Dieser Index ähnelt dem NDVI, vergleicht jedoch den Kontrast zwischen NIR und RedEdge anstelle von Red, wodurch Vegetationsstress oft früher erkannt wird.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalisierter Differenz-Vegetationsindex

Dieser Index ist ein Maß für gesunde, grüne Vegetation. Die Kombination aus seiner Formulierung als normalisierte Differenz und der Nutzung der Bereiche mit der höchsten Absorption und Reflexion von Chlorophyll macht ihn unter einer Vielzahl von Bedingungen robust. Er kann jedoch bei dichter Vegetation, wenn der LAI-Wert hoch ist, zu einer Sättigung kommen.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Der Wert dieses Indexes reicht von -1 bis 1. Der übliche Bereich für grüne Vegetation liegt zwischen 0,2 und 0,8.

_Quelle: Rouse, J., R. Haas, J. Schell und D. Deering. „Monitoring Vegetation Systems in the Great Plains with ERTS“. Drittes ERTS-Symposium, NASA (1973): 309–317._

***

## NLI – Nichtlinearer Index

Dieser Index geht davon aus, dass die Beziehung zwischen vielen Vegetationsindizes und biophysikalischen Oberflächenparametern nichtlinear ist. Er linearisiert Beziehungen zu Oberflächenparametern, die tendenziell nichtlinear sind.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Literaturhinweis: Goel, N., und W. Qin. „Einflüsse der Kronenarchitektur auf die Beziehungen zwischen verschiedenen Vegetationsindizes und LAI sowie Fpar: Eine Computersimulation.“ Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimierter bodenangepasster Vegetationsindex

Dieser Index basiert auf dem bodenangepassten Vegetationsindex (SAVI). Er verwendet einen Standardwert von 0,16 für den Hintergrundanpassungsfaktor des Kronendachs. Rondeaux (1996) stellte fest, dass dieser Wert bei geringer Vegetationsbedeckung eine größere Bodenvariation ermöglicht als der SAVI, während er gleichzeitig eine erhöhte Empfindlichkeit gegenüber einer Vegetationsbedeckung von mehr als 50 % aufweist. Dieser Index eignet sich am besten für Gebiete mit relativ spärlicher Vegetation, in denen der Boden durch das Kronendach hindurch sichtbar ist.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Quelle: Rondeaux, G., M. Steven und F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.“ Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalisierter Differenz-Vegetationsindex

Dieser Index nutzt die Differenz zwischen den Wellenlängen im nahen Infrarot und im roten Bereich in Verbindung mit dem NDVI, um gesunde Vegetation hervorzuheben. Er ist unempfindlich gegenüber den Auswirkungen des Bodens und der Sonneneinstrahlungsgeometrie.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Quelle: Roujean, J., und F. Breon. „Schätzung der von der Vegetation absorbierten PAR anhand bidirektionaler Reflektanzmessungen.“ Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Bodenbereinigter Vegetationsindex

Dieser Index ähnelt dem NDVI, unterdrückt jedoch die Auswirkungen von Bodenpixeln. Er verwendet einen Hintergrundkorrekturfaktor für das Kronendach, _L_, der eine Funktion der Vegetationsdichte ist und häufig Vorkenntnisse über die Vegetationsmenge erfordert. Huete (1988) schlägt einen optimalen Wert von _L_ = 0,5 vor, um Bodenhintergrundschwankungen erster Ordnung zu berücksichtigen. Dieser Index eignet sich am besten für Gebiete mit relativ spärlicher Vegetation, in denen der Boden durch das Kronendach hindurch sichtbar ist.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Quelle: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).“ Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – Transformed Difference Vegetation Index

Dieser Index eignet sich zur Überwachung der Vegetationsbedeckung in städtischen Gebieten. Er zeigt keine Sättigung wie der NDVI und der SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Quelle: Bannari, A., H. Asalhi und P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping“ In: Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS '02, IEEE International, Band 5 (2002)._

***

## VARI – Visible Atmospherically Resistant Index

Dieser Index basiert auf dem ARVI und wird verwendet, um den Vegetationsanteil in einer Szene mit geringer Empfindlichkeit gegenüber atmosphärischen Einflüssen abzuschätzen.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Literaturhinweis: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction“. International Journal of Remote Sensing 23 (2002): 2537–2562._

***

## WDRVI – Vegetationsindex mit großem Dynamikbereich

Dieser Index ähnelt dem NDVI, verwendet jedoch einen Gewichtungskoeffizienten (_a_), um die Diskrepanz zwischen den Beiträgen der Nahinfrarot- und Rot-Signale zum NDVI zu verringern. Der WDRVI ist besonders effektiv in Szenen mit mittlerer bis hoher Vegetationsdichte, wenn der NDVI den Wert 0,6 überschreitet. Der NDVI neigt dazu, sich abzuflachen, wenn der Vegetationsanteil und der Blattflächenindex (LAI) zunehmen, während der WDRVI empfindlicher auf einen größeren Bereich von Vegetationsanteilen und auf Änderungen des LAI reagiert.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Der Gewichtungskoeffizient (_a_) kann zwischen 0,1 und 0,2 liegen. Ein Wert von 0,2 wird von Henebry, Viña und Gitelson (2004) empfohlen.

_Literaturverzeichnis_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.“ Journal of Plant Physiology 161, Nr. 2 (2004): 165–173._

_Henebry, G., A. Viña und A. Gitelson. „Der Vegetationsindex mit großem Dynamikbereich und sein potenzieller Nutzen für die Lückenanalyse.“ Gap Analysis Bulletin 12: 50–56._
