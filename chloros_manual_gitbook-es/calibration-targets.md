---
description: Im Labor gemessene Panels zur Kalibrierung der erfassten Daten in der Nachbearbeitung
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrierungsziele

MAPIR bietet verschiedene Kalibrierungsziele, um eine Reihe von Anwendungen abzudecken. Der unten abgebildete kompakte T4-R50 enthält 4 Panels, deren Lichtreflexion im Bereich von 250 bis 2.500 nm gemessen wurde.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Die diffusen T4-Referenzziele haben die folgenden Reflexionskurven, [Daten-Download hier](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflexionsgrad :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflexionsgrad :: 400–1000 nm</p></figcaption></figure>

Wenn Sie sich das Reflexionsdiagramm ansehen, können Sie sehen, dass die Werte die Wellenlänge (X-Achse) im Vergleich zum Reflexionsprozentsatz (Y-Achse) darstellen. Wenn wir ein Bild des Kalibrierungsziels aufnehmen, erstellen wir eine Beziehung zwischen Pixelwert und Reflexionsprozentsatz innerhalb des Spektrums, für das jedes Sensorband der Kamera empfindlich ist.

Das bedeutet, dass Sie bei jedem Bild, das Sie mit unseren Kameras aufnehmen, ein Foto unserer Reflexionsziele, wie z. B. [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) oder [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), verwenden können, um die Bilder hinsichtlich des Reflexionsvermögens zu kalibrieren. Nach der Kalibrierung entspricht jedes Pixel im Bild dem prozentualen Reflexionsgrad.

Wenn Sie die kalibrierten Bilder in Chloros als typisches JPG oder TIFF ausgeben, wird der Reflexionsgrad berechnet, indem der Pixelwert durch die Bittiefe des Bildformats dividiert wird. Teilen Sie also für JPG durch 255 und für TIFF durch 65.535. Sie können in Chloros auch das PROZENT-Format für die Ausgabe wählen, und jedes Pixel hat dann einen Prozentwert von 0,0 bis 1,0 (Reflexionsgrad von 0 % bis 100 %). Bedenken Sie jedoch, dass einige Bildanwendungen die Prozentbilder (Gleitkommabilder) nicht akzeptieren können und dass sie speichermäßig sehr groß sind.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
