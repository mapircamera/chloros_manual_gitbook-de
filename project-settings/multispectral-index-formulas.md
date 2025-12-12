---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektrale Indexformeln

Die folgenden Indexformeln verwenden eine Kombination aus den durchschnittlichen Transmissionsbereichen des Filters Survey3:

<table><thead><tr><th align="center">Survey3 Filterfarbe</th><th width="196.199951171875" align="center">Survey3-Filtername</th><th width="159.800048828125" align="center">Durchlässigkeitsbereich (FWHM)</th><th align="center">Durchschnittliche Transmission</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Bei Verwendung dieser Formeln kann der Name auf „\_1” oder „\_2” enden, was dem verwendeten NIR-Filter entspricht, entweder NIR1 oder NIR2.

***

## EVI – Erweiterter Vegetationsindex

Dieser Index wurde ursprünglich für die Verwendung mit MODIS-Daten als Verbesserung gegenüber NDVI entwickelt, indem das Vegetationssignal in Gebieten mit hohem Blattflächenindex (LAI) optimiert wurde. Er ist besonders nützlich in Regionen mit hohem LAI, in denen NDVI gesättigt sein kann. Er nutzt den blauen Reflexionsbereich, um Bodensignale zu korrigieren und atmosphärische Einflüsse, einschließlich Aerosolstreuung, zu reduzieren.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Die Werte von EVI sollten für Vegetationspixel im Bereich von 0 bis 1 liegen. Helle Merkmale wie Wolken und weiße Gebäude sowie dunkle Merkmale wie Wasser können zu anomalen Pixelwerten in einem EVI-Bild führen. Bevor Sie ein EVI-Bild erstellen, sollten Sie Wolken und helle Merkmale aus dem Reflexionsbild ausblenden und optional die Pixelwerte von 0 bis 1 schwellenwertieren.

_Referenz: Huete, A., et al. „Überblick über die radiometrische und biophysikalische Leistung der MODIS-Vegetationsindizes.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Waldbedeckungsindex 1

Dieser Index unterscheidet Waldkronen von anderen Vegetationstypen anhand multispektraler Reflexionsbilder, die ein rotes Randband enthalten.

$$
FCI1 = Red * RedEdge
$$

Waldgebiete weisen aufgrund der geringeren Reflexion von Bäumen und der Schattenbildung innerhalb des Kronendachs niedrigere FCI1-Werte auf.

_Referenz: Becker, Sarah J., Craig S.T. Daughtry und Andrew L. Russ. „Robuste Waldbedeckungsindizes für multispektrale Bilder.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 – Waldbedeckungsindex 2

Dieser Index unterscheidet Waldkronen von anderen Vegetationstypen anhand multispektraler Reflexionsbilder, die kein rotes Randband enthalten.

$$
FCI2 = Red * NIR
$$

Waldgebiete weisen aufgrund der geringeren Reflexion von Bäumen und des Vorhandenseins von Schatten innerhalb des Kronendachs niedrigere FCI2-Werte auf.

_Referenz: Becker, Sarah J., Craig S.T. Daughtry und Andrew L. Russ. „Robuste Waldbedeckungsindizes für multispektrale Bilder.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI – Globaler Umweltüberwachungsindex

Dieser nichtlineare Vegetationsindex wird für die globale Umweltüberwachung anhand von Satellitenbildern verwendet und versucht, atmosphärische Effekte zu korrigieren. Er ähnelt NDVI, ist jedoch weniger empfindlich gegenüber atmosphärischen Effekten. Er wird durch nackten Boden beeinflusst und ist daher nicht für den Einsatz in Gebieten mit spärlicher oder mäßig dichter Vegetation empfohlen.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Wo:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referenz: Pinty, B., und M. Verstraete. GEMI: ein nichtlinearer Index zur Überwachung der globalen Vegetation aus Satelliten. Vegetation 101 (1992): 15-20._

***

## GARI – Green Atmosphärisch resistenter Index

Dieser Index reagiert empfindlicher auf einen breiten Bereich von Chlorophyllkonzentrationen und weniger empfindlich auf atmosphärische Einflüsse als NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Die Gamma-Konstante ist eine Gewichtungsfunktion, die von den Aerosolbedingungen in der Atmosphäre abhängt. ENVI verwendet einen Wert von 1,7, der von Gitelson, Kaufman und Merzylak (1996, Seite 296) empfohlen wird.

_Referenz: Gitelson, A., Y. Kaufman und M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.” Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI – Green Chlorophyllindex

Dieser Index wird verwendet, um den Chlorophyllgehalt der Blätter einer Vielzahl von Pflanzenarten zu schätzen.

$$
GCI = {NIR \over Green} - 1
$$

Durch einen breiten NIR und grünen Wellenlängenbereich lässt sich der Chlorophyllgehalt besser vorhersagen, während gleichzeitig eine höhere Empfindlichkeit und ein besseres Signal-Rausch-Verhältnis erzielt werden.

_Referenz: Gitelson, A., Y. Gritz und M. Merzlyak. „Beziehungen zwischen dem Chlorophyllgehalt von Blättern und der spektralen Reflexion sowie Algorithmen für die zerstörungsfreie Chlorophyllbewertung in Blättern höherer Pflanzen.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Blattindex

Dieser Index wurde ursprünglich für die Verwendung mit einer digitalen RGB-Kamera zur Messung der Weizenbedeckung entwickelt, wobei die digitalen Zahlen (DNs) für Rot, Grün und Blau im Bereich von 0 bis 255 liegen.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Die Werte von GLI reichen von -1 bis +1. Negative Werte stehen für Boden und nicht lebende Merkmale, während positive Werte für grüne Blätter und Stängel stehen.

_Referenz: Louhaichi, M., M. Borman und D. Johnson. „Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.” Geocarto International 16, Nr. 1 (2001): 65-70._

***

## GNDVI – Green Normalisierter Differenzvegetationsindex

Dieser Index ähnelt NDVI, misst jedoch das grüne Spektrum von 540 bis 570 nm anstelle des roten Spektrums. Dieser Index reagiert empfindlicher auf die Chlorophyllkonzentration als NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referenz: Gitelson, A., und M. Merzlyak. „Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves” (Fernerkundung der Chlorophyllkonzentration in Blättern höherer Pflanzen). Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI – Green Optimierter bodenangepasster Vegetationsindex

Dieser Index wurde ursprünglich mit Hilfe von Farb-Infrarotfotografie entwickelt, um den Stickstoffbedarf von Mais vorherzusagen. Er ähnelt OSAVI, ersetzt jedoch das grüne Band durch das rote.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referenz: Sripada, R., et al. „Bestimmung des Stickstoffbedarfs von Mais während der Vegetationsperiode mithilfe von Luftbildaufnahmen im Farb-Infrarotbereich.“ Doktorarbeit, North Carolina State University, 2005._

***

## GRVI – Green Verhältnis-Vegetationsindex

Dieser Index reagiert empfindlich auf die Photosyntheserate in Waldkronen, da die Reflexion von grünem und rotem Licht stark von Veränderungen der Blattpigmente beeinflusst wird.

$$
GRVI = {NIR \over Green }
$$

_Referenz: Sripada, R., et al. „Aerial Color Infrared Photography for Determining Early In-season Nitrogen Requirements in Corn” (Luftbild-Farbinfrarotfotografie zur Bestimmung des Stickstoffbedarfs von Mais zu Beginn der Vegetationsperiode). Agronomy Journal 98 (2006): 968-977._

***

## GSAVI – Green Bodenangepasster Vegetationsindex

Dieser Index wurde ursprünglich mit Hilfe der Farb-Infrarotfotografie entwickelt, um den Stickstoffbedarf von Mais vorherzusagen. Er ähnelt SAVI, ersetzt jedoch das grüne Band durch das rote.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referenz: Sripada, R., et al. „Bestimmung des Stickstoffbedarfs von Mais während der Vegetationsperiode mithilfe von Luftbildaufnahmen im Farb-Infrarotbereich.“ Doktorarbeit, North Carolina State University, 2005._

***

## LAI – Blattflächenindex

Dieser Index wird verwendet, um die Blattbedeckung zu schätzen und das Wachstum und den Ertrag von Nutzpflanzen vorherzusagen. ENVI berechnet den grünen LAI anhand der folgenden empirischen Formel von Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Dabei ist EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hohe LAI-Werte liegen in der Regel zwischen etwa 0 und 3,5. Wenn die Szene jedoch Wolken und andere helle Merkmale enthält, die gesättigte Pixel erzeugen, können die LAI-Werte 3,5 überschreiten. Idealerweise sollten Sie Wolken und helle Merkmale aus Ihrer Szene maskieren, bevor Sie ein LAI-Bild erstellen.

_Referenz: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde und A. Thomsen. „Luftgestützte multispektrale Daten zur Quantifizierung des Blattflächenindex, der Stickstoffkonzentration und der Photosyntheseeffizienz in der Landwirtschaft.“ Remote Sensing of Environment 81, Nr. 2-3 (2002): 179-193._

***

## LCI – Blattchlorophyllindex

Dieser Index wird verwendet, um den Chlorophyllgehalt in höheren Pflanzen zu schätzen, die empfindlich auf Veränderungen der Reflexion reagieren, die durch die Absorption von Chlorophyll verursacht werden.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referenz: Datt, B. „Fernerkundung des Wassergehalts in Eukalyptusblättern.“ Journal of Plant Physiology 154, Nr. 1 (1999): 30-36._

***

## MNLI – Modifizierter nichtlinearer Index

Dieser Index ist eine Erweiterung des Nichtlinearen Index (NLI), der den Bodenangepassten Vegetationsindex (SAVI) einbezieht, um den Bodenhintergrund zu berücksichtigen. ENVI verwendet einen Wert von 0,5 für den Anpassungsfaktor für den Kronendachhintergrund (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referenz: Yang, Z., P. Willis und R. Mueller. „Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy” (Auswirkungen von bandverstärkten AWIFS-Bildern auf die Genauigkeit der Klassifizierung von Nutzpflanzen). Tagungsband des Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modifizierter bodenangepasster Vegetationsindex 2

Dieser Index ist eine vereinfachte Version des von Qi et al. (1994) vorgeschlagenen MSAVI-Index, der den bodenangepassten Vegetationsindex (SAVI) verbessert. Er reduziert Bodenrauschen und erhöht den Dynamikbereich des Vegetationssignals. MSAVI2 basiert auf einer induktiven Methode, die keinen konstanten _L_-Wert (wie bei SAVI) verwendet, um gesunde Vegetation hervorzuheben.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referenz: Qi, J., A. Chehbouni, A. Huete, Y. Kerr und S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE – Normalisierte Differenz RedEdge

Dieser Index ähnelt NDVI, vergleicht jedoch den Kontrast zwischen NIR und RedEdge anstelle von Red, wodurch Vegetationsstress oft früher erkannt wird.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalisierter Differenzvegetationsindex

Dieser Index ist ein Maß für gesunde, grüne Vegetation. Die Kombination aus seiner normalisierten Differenzformulierung und der Verwendung der Regionen mit der höchsten Absorption und Reflexion von Chlorophyll macht ihn unter einer Vielzahl von Bedingungen robust. Er kann jedoch bei dichter Vegetation gesättigt sein, wenn LAI hoch wird.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Der Wert dieses Index reicht von -1 bis 1. Der übliche Bereich für grüne Vegetation liegt zwischen 0,2 und 0,8.

_Referenz: Rouse, J., R. Haas, J. Schell und D. Deering. Überwachung von Vegetationssystemen in den Great Plains mit ERTS. Drittes ERTS-Symposium, NASA (1973): 309-317._

***

## NLI – Nichtlinearer Index

Dieser Index geht davon aus, dass die Beziehung zwischen vielen Vegetationsindizes und biophysikalischen Oberflächenparametern nichtlinear ist. Er linearisiert Beziehungen zu Oberflächenparametern, die tendenziell nichtlinear sind.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referenz: Goel, N., und W. Qin. „Einflüsse der Baumkronenarchitektur auf die Beziehungen zwischen verschiedenen Vegetationsindizes und LAI und Fpar: Eine Computersimulation.“ Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – Optimierter bodenangepasster Vegetationsindex

Dieser Index basiert auf dem bodenangepassten Vegetationsindex (SAVI). Er verwendet einen Standardwert von 0,16 für den Anpassungsfaktor für den Hintergrund des Blätterdachs. Rondeaux (1996) stellte fest, dass dieser Wert bei geringer Vegetationsbedeckung eine größere Bodenvariation als SAVI liefert und gleichzeitig eine erhöhte Empfindlichkeit gegenüber einer Vegetationsbedeckung von mehr als 50 % aufweist. Dieser Index eignet sich am besten für Gebiete mit relativ spärlicher Vegetation, in denen der Boden durch das Blätterdach hindurch sichtbar ist.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referenz: Rondeaux, G., M. Steven und F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Renormalisierter Differenzvegetationsindex

Dieser Index nutzt die Differenz zwischen nahen Infrarot- und roten Wellenlängen zusammen mit dem NDVI, um gesunde Vegetation hervorzuheben. Er ist unempfindlich gegenüber den Auswirkungen von Boden und Sonnengeometrie.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referenz: Roujean, J., und F. Breon. „Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.” Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI – Bodenbereinigter Vegetationsindex

Dieser Index ähnelt NDVI, unterdrückt jedoch die Auswirkungen von Bodenpixeln. Er verwendet einen Anpassungsfaktor für den Hintergrund der Baumkronen, _L_, der eine Funktion der Vegetationsdichte ist und häufig Vorkenntnisse über die Vegetationsmenge erfordert. Huete (1988) schlägt einen optimalen Wert von _L_=0,5 vor, um Bodenhintergrundschwankungen erster Ordnung zu berücksichtigen. Dieser Index eignet sich am besten für Gebiete mit relativ spärlicher Vegetation, in denen der Boden durch die Baumkronen sichtbar ist.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referenz: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).“ Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – Transformierter Differenzvegetationsindex

Dieser Index ist nützlich für die Überwachung der Vegetationsdecke in städtischen Umgebungen. Er sättigt nicht wie NDVI und SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referenz: Bannari, A., H. Asalhi und P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Band 5 (2002)._

***

## VARI – Sichtbarer atmosphärischer Resistenzindex

Dieser Index basiert auf dem ARVI und wird verwendet, um den Anteil der Vegetation in einer Szene mit geringer Empfindlichkeit gegenüber atmosphärischen Einflüssen zu schätzen.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referenz: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Vegetationsindex mit großem Dynamikbereich

Dieser Index ähnelt NDVI, verwendet jedoch einen Gewichtungskoeffizienten (_a_), um die Diskrepanz zwischen den Beiträgen der Nahinfrarot- und Rot-Signale zum NDVI zu verringern. Der WDRVI ist besonders effektiv in Szenen mit mittlerer bis hoher Vegetationsdichte, wenn NDVI 0,6 überschreitet. NDVI neigt dazu, sich zu stabilisieren, wenn der Vegetationsanteil und der Blattflächenindex (LAI) zunehmen, während WDRVI empfindlicher auf einen größeren Bereich von Vegetationsanteilen und auf Änderungen in LAI reagiert.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Der Gewichtungskoeffizient (_a_) kann zwischen 0,1 und 0,2 liegen. Ein Wert von 0,2 wird von Henebry, Viña und Gitelson (2004) empfohlen.

_Referenzen_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.” Journal of Plant Physiology 161, Nr. 2 (2004): 165-173._

_Henebry, G., A. Viña und A. Gitelson. „Der Vegetationsindex mit großem Dynamikbereich und sein potenzieller Nutzen für die Lückenanalyse.“ Gap Analysis Bulletin 12: 50-56._
