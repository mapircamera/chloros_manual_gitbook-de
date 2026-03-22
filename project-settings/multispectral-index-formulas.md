---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formeln für multispektrale Indizes

Die folgenden Indexformeln verwenden eine Kombination aus den durchschnittlichen Durchlässigkeitsbereichen des Filters Survey3:

<table><thead><tr><th align="center">Survey3 Filterfarbe</th><th width="196.199951171875" align="center">Survey3 Filter-Name</th><th width="159.800048828125" align="center">Transmissionsbereich (FWHM)</th><th align="center">Durchschnittliche Transmission</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN – Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN – NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Bei Verwendung dieser Formeln kann der Name auf „\_1“ oder „\_2“ enden, was angibt, welcher NIR-Filter verwendet wurde, entweder NIR1 oder NIR2.

***

## EVI – Erweiterter Vegetationsindex

Dieser Index wurde ursprünglich für die Verwendung mit MODIS-Daten entwickelt und stellt eine Verbesserung gegenüber NDVI dar, indem er das Vegetationssignal in Gebieten mit hohem Blattflächenindex (LAI) optimiert. Er ist besonders nützlich in Regionen mit hohem LAI-Wert, in denen NDVI möglicherweise gesättigt ist. Er nutzt den blauen Reflexionsbereich, um Bodensignale zu korrigieren und atmosphärische Einflüsse, einschließlich Aerosolstreuung, zu reduzieren.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-Werte sollten für Vegetationspixel im Bereich von 0 bis 1 liegen. Helle Objekte wie Wolken und weiße Gebäude sowie dunkle Objekte wie Wasser können zu anomalen Pixelwerten in einem EVI-Bild führen. Vor der Erstellung eines EVI-Bildes sollten Sie Wolken und helle Objekte aus dem Reflexionsbild ausblenden und optional die Pixelwerte auf einen Schwellenwert zwischen 0 und 1 beschränken.

_Referenz: Huete, A., et al. „Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Waldbedeckungsindex 1

Dieser Index unterscheidet Waldkronen von anderen Vegetationstypen unter Verwendung multispektraler Reflektanzbilder, die ein Red-Edge-Band enthalten.

$$
FCI1 = Red * RedEdge
$$

Bewaldete Gebiete weisen aufgrund der geringeren Reflektanz von Bäumen und des Vorhandenseins von Schatten innerhalb des Kronendachs niedrigere FCI1-Werte auf.

_Referenz: Becker, Sarah J., Craig S.T. Daughtry und Andrew L. Russ. „Robust forest cover indices for multispectral images.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Waldbedeckungsindex 2

Dieser Index unterscheidet Waldkronen von anderen Vegetationstypen unter Verwendung multispektraler Reflektanzbilder, die kein Red-Edge-Band enthalten.

$$
FCI2 = Red * NIR
$$

Bewaldete Gebiete weisen aufgrund der geringeren Reflektanz von Bäumen und des Vorhandenseins von Schatten innerhalb des Kronendachs niedrigere FCI2-Werte auf.

_Referenz: Becker, Sarah J., Craig S.T. Daughtry und Andrew L. Russ. „Robust forest cover indices for multispectral images.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Globaler Umweltüberwachungsindex

Dieser nichtlineare Vegetationsindex wird für die globale Umweltüberwachung anhand von Satellitenbildern verwendet und versucht, atmosphärische Einflüsse zu korrigieren. Er ähnelt NDVI, ist jedoch weniger empfindlich gegenüber atmosphärischen Einflüssen. Er wird durch kahle Böden beeinflusst; daher wird seine Verwendung in Gebieten mit spärlicher oder mäßig dichter Vegetation nicht empfohlen.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Wo:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referenz: Pinty, B., und M. Verstraete. GEMI: ein nichtlinearer Index zur globalen Vegetationsüberwachung aus Satellitendaten. Vegetation 101 (1992): 15–20._

***

## GARI – Green: Atmosphärenresistenter Index

Dieser Index reagiert empfindlicher auf einen breiten Bereich von Chlorophyllkonzentrationen und weniger empfindlich auf atmosphärische Einflüsse als NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Die Gamma-Konstante ist eine Gewichtungsfunktion, die von den Aerosolbedingungen in der Atmosphäre abhängt. ENVI verwendet einen Wert von 1,7, was dem von Gitelson, Kaufman und Merzylak (1996, Seite 296) empfohlenen Wert entspricht.

_Referenz: Gitelson, A., Y. Kaufman und M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.“ Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green Chlorophyllindex

Dieser Index wird verwendet, um den Chlorophyllgehalt der Blätter bei einer Vielzahl von Pflanzenarten zu schätzen.

$$
GCI = {NIR \over Green} - 1
$$

Die Verwendung breiter NIR- und Grünwellenlängen ermöglicht eine bessere Vorhersage des Chlorophyllgehalts und sorgt gleichzeitig für eine höhere Empfindlichkeit und ein besseres Signal-Rausch-Verhältnis.

_Referenz: Gitelson, A., Y. Gritz und M. Merzlyak. „Relationships Between Leaf Chlorophyll Content and Spectral Reflectance and Algorithms for Non-Destructive Chlorophyll Assessment in Higher Plant Leaves.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Blattindex

Dieser Index wurde ursprünglich für die Verwendung mit einer digitalen RGB-Kamera zur Messung der Weizenbedeckung entwickelt, wobei die digitalen Werte (DNs) für Rot, Grün und Blau im Bereich von 0 bis 255 liegen.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Die Werte von GLI reichen von -1 bis +1. Negative Werte stehen für Boden und nicht lebende Elemente, während positive Werte grüne Blätter und Stängel darstellen.

_Referenz: Louhaichi, M., M. Borman und D. Johnson. „Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.“ Geocarto International 16, Nr. 1 (2001): 65–70._

***

## GNDVI – Green Normalisierter Differenzvegetationsindex

Dieser Index ähnelt dem NDVI, misst jedoch das grüne Spektrum von 540 bis 570 nm anstelle des roten Spektrums. Dieser Index reagiert empfindlicher auf die Chlorophyllkonzentration als der NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referenz: Gitelson, A., und M. Merzlyak. „Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.“ Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimierter bodenangepasster Vegetationsindex

Dieser Index wurde ursprünglich mit Hilfe der Farb-Infrarot-Fotografie entwickelt, um den Stickstoffbedarf von Mais vorherzusagen. Er ähnelt dem OSAVI, ersetzt jedoch das rote Band durch das grüne.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referenz: Sripada, R., et al. „Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.“ Doktorarbeit, North Carolina State University, 2005._

***

## GRVI – Green-Verhältnis-Vegetationsindex

Dieser Index reagiert empfindlich auf die Photosyntheseraten im Walddach, da die Reflexionswerte für Grün und Rot stark von Veränderungen der Blattpigmente beeinflusst werden.

$$
GRVI = {NIR \over Green }
$$

_Referenz: Sripada, R., et al. „Aerial Color Infrared Photography for Determining Early In-season Nitrogen Requirements in Corn.“ Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Bodenbereinigter Vegetationsindex

Dieser Index wurde ursprünglich mit Hilfe der Farb-Infrarot-Fotografie entwickelt, um den Stickstoffbedarf von Mais vorherzusagen. Er ähnelt dem SAVI, ersetzt jedoch das rote Band durch das grüne.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referenz: Sripada, R., et al. „Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.“ Doktorarbeit, North Carolina State University, 2005._

***

## LAI – Blattflächenindex

Dieser Index wird verwendet, um die Blattbedeckung zu schätzen und das Pflanzenwachstum sowie den Ertrag zu prognostizieren. ENVI berechnet den grünen LAI unter Verwendung der folgenden empirischen Formel von Boegh et al. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Wobei EVI ist:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hohe LAI-Werte liegen typischerweise im Bereich von etwa 0 bis 3,5. Wenn die Szene jedoch Wolken und andere helle Merkmale enthält, die gesättigte Pixel erzeugen, können die LAI-Werte 3,5 überschreiten. Idealerweise sollten Sie Wolken und helle Merkmale aus Ihrer Szene ausblenden, bevor Sie ein LAI-Bild erstellen.

_Referenz: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde und A. Thomsen. „Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.“ Remote Sensing of Environment 81, Nr. 2-3 (2002): 179-193._

***

## LCI – Blattchlorophyllindex

Dieser Index wird zur Schätzung des Chlorophyllgehalts in höheren Pflanzen verwendet und reagiert empfindlich auf Schwankungen der Reflexion, die durch die Absorption von Chlorophyll verursacht werden.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referenz: Datt, B. „Remote Sensing of Water Content in Eucalyptus Leaves.“ Journal of Plant Physiology 154, Nr. 1 (1999): 30–36._

***

## MNLI – Modifizierter nichtlinearer Index

Dieser Index ist eine Erweiterung des Nichtlinearen Index (NLI), der den bodenbereinigten Vegetationsindex (SAVI) einbezieht, um den Bodenhintergrund zu berücksichtigen. ENVI verwendet einen Wert von 0,5 für den Faktor zur Anpassung des Kronendachhintergrunds (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referenz: Yang, Z., P. Willis und R. Mueller. „Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.“ Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modifizierter bodenangepasster Vegetationsindex 2

Dieser Index ist eine vereinfachte Version des von Qi et al. (1994) vorgeschlagenen MSAVI-Index, der den Boden-angepassten Vegetationsindex (SAVI) verbessert. Er reduziert Bodenrauschen und erhöht den Dynamikbereich des Vegetationssignals. MSAVI2 basiert auf einer induktiven Methode, die keinen konstanten _L_-Wert (wie bei SAVI) verwendet, um gesunde Vegetation hervorzuheben.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referenz: Qi, J., A. Chehbouni, A. Huete, Y. Kerr und S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.“ Remote Sensing of Environment 48 (1994): 119–126._

***

## NDRE – Normalisierte Differenz RedEdge

Dieser Index ähnelt dem NDVI, vergleicht jedoch den Kontrast zwischen NIR und RedEdge anstelle von Red, wodurch Vegetationsstress oft früher erkannt wird.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalisierter Differenzvegetationsindex

Dieser Index ist ein Maß für gesunde, grüne Vegetation. Die Kombination aus seiner normierten Differenzformulierung und der Nutzung der Bereiche mit der höchsten Absorption und Reflexion von Chlorophyll macht ihn unter einer Vielzahl von Bedingungen robust. Er kann jedoch bei dichter Vegetation sättigen, wenn LAI hohe Werte annimmt.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Der Wert dieses Indexes reicht von -1 bis 1. Der übliche Bereich für grüne Vegetation liegt zwischen 0,2 und 0,8.

_Referenz: Rouse, J., R. Haas, J. Schell und D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309-317._

***

## NLI – Nichtlinearer Index

Dieser Index geht davon aus, dass die Beziehung zwischen vielen Vegetationsindizes und biophysikalischen Oberflächenparametern nichtlinear ist. Er linearisiert Beziehungen zu Oberflächenparametern, die tendenziell nichtlinear sind.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referenz: Goel, N. und W. Qin. „Einflüsse der Kronenstruktur auf die Beziehungen zwischen verschiedenen Vegetationsindizes und LAI sowie Fpar: Eine Computersimulation.“ Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimierter bodenbereinigter Vegetationsindex

Dieser Index basiert auf dem bodenbereinigten Vegetationsindex (SAVI). Er verwendet einen Standardwert von 0,16 für den Hintergrundanpassungsfaktor des Kronendachs. Rondeaux (1996) stellte fest, dass dieser Wert bei geringer Vegetationsbedeckung eine größere Bodenvariation bietet als SAVI, während er gleichzeitig eine erhöhte Empfindlichkeit gegenüber einer Vegetationsbedeckung von mehr als 50 % aufweist. Dieser Index eignet sich am besten für Gebiete mit relativ spärlicher Vegetation, in denen der Boden durch das Kronendach sichtbar ist.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referenz: Rondeaux, G., M. Steven und F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.“ Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormierter Differenzvegetationsindex

Dieser Index nutzt die Differenz zwischen Nahinfrarot- und Rotwellenlängen zusammen mit dem NDVI, um gesunde Vegetation hervorzuheben. Er ist unempfindlich gegenüber den Auswirkungen von Bodenbeschaffenheit und Sonnenstand.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referenz: Roujean, J., und F. Breon. „Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.“ Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Bodenbereinigter Vegetationsindex

Dieser Index ähnelt NDVI, unterdrückt jedoch die Auswirkungen von Bodenpixeln. Er verwendet einen Faktor zur Anpassung des Hintergrunds des Kronendachs, _L_, der eine Funktion der Vegetationsdichte ist und oft Vorkenntnisse über die Vegetationsmenge erfordert. Huete (1988) schlägt einen optimalen Wert von _L_=0,5 vor, um Bodenhintergrundschwankungen erster Ordnung zu berücksichtigen. Dieser Index eignet sich am besten für Gebiete mit relativ spärlicher Vegetation, in denen der Boden durch das Kronendach sichtbar ist.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referenz: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).“ Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – Transformed Difference Vegetation Index

Dieser Index eignet sich zur Überwachung der Vegetationsbedeckung in städtischen Umgebungen. Er saturiert nicht wie NDVI und SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referenz: Bannari, A., H. Asalhi und P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping“ In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Band 5 (2002)._

***

## VARI – Sichtbarer atmosphärenresistenter Index

Dieser Index basiert auf dem ARVI und wird verwendet, um den Vegetationsanteil in einer Szene mit geringer Empfindlichkeit gegenüber atmosphärischen Einflüssen zu schätzen.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referenz: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.“ International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Vegetationsindex mit großem Dynamikbereich

Dieser Index ähnelt dem NDVI, verwendet jedoch einen Gewichtungskoeffizienten (_a_), um die Diskrepanz zwischen den Beiträgen der Nahinfrarot- und der Rotsignale zum NDVI zu verringern. Der WDRVI ist besonders effektiv in Szenen mit mittlerer bis hoher Vegetationsdichte, wenn der NDVI den Wert 0,6 überschreitet. NDVI neigt dazu, sich abzuflachen, wenn der Vegetationsanteil und der Blattflächenindex (LAI) zunehmen, während der WDRVI empfindlicher auf einen größeren Bereich von Vegetationsanteilen und auf Änderungen des LAI reagiert.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Der Gewichtungskoeffizient (_a_) kann zwischen 0,1 und 0,2 liegen. Ein Wert von 0,2 wird von Henebry, Viña und Gitelson (2004) empfohlen.

_Literaturhinweise_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.“ Journal of Plant Physiology 161, Nr. 2 (2004): 165–173._

_Henebry, G., A. Viña und A. Gitelson. „The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.“ Gap Analysis Bulletin 12: 50–56._
