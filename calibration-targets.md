---
description: Lab-measured panels used to calibrate captured data in post-processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# Kalibrierungsziele

MAPIR bietet verschiedene Kalibrierungsziele für eine Vielzahl von Anwendungen. Das unten abgebildete kompakte T4-R50 enthält 4 Panels, deren Lichtreflexion im Bereich von 250 bis 2.500 nm gemessen wurde.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Die diffusen Referenzziele T4 weisen die folgenden Reflexionskurven auf, [Daten hier herunterladen](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflexion :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflexion :: 400–1000 nm</p></figcaption></figure>

Aus dem Reflexionsgrad-Diagramm geht hervor, dass die Werte die Wellenlänge (x-Achse) im Verhältnis zum Reflexionsgrad in Prozent (y-Achse) darstellen. Wenn wir ein Bild des Kalibrierungsziels aufnehmen, erstellen wir eine Beziehung zwischen dem Pixelwert und dem Reflexionsgrad in Prozent innerhalb des Spektrums, für das die einzelnen Sensorbereiche der Kamera empfindlich sind.

Das bedeutet, dass Sie bei jedem Bild, das Sie mit unseren Kameras aufnehmen, ein Foto unserer Reflektionsziels, wie beispielsweise [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) oder [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), verwenden können, um die Bilder hinsichtlich der Reflektionsfähigkeit zu kalibrieren. Nach der Kalibrierung entspricht jedes Pixel im Bild einem Prozentsatz der Reflexion.

Wenn Sie die kalibrierten Bilder in Chloros als typisches JPG oder TIFF ausgeben, wird der Reflexionsgrad in Prozent berechnet, indem der Pixelwert durch die Bittiefe des Bildformats dividiert wird. Für JPG dividieren Sie also durch 255 und für TIFF durch 65.535. Sie können auch das PERCENT-Format in Chloros auswählen, dann liegt jeder Pixelwert zwischen 0,0 und 1,0 Prozent (0 % bis 100 % Reflexion). Beachten Sie jedoch, dass einige Bildbearbeitungsprogramme keine Prozentbilder (Gleitkomma) akzeptieren und diese Bilder sehr viel Speicherplatz beanspruchen.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
