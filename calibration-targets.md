---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrierungsstandards

MAPIR bietet verschiedene Kalibrierungsstandards für eine Vielzahl von Anwendungen an. Der unten abgebildete kompakte T4-R50 enthält vier Platten, deren Lichtreflexion im Bereich von 250 bis 2.500 nm gemessen wurde.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Die diffusen Referenztargets vom Typ T4 weisen die folgenden Reflexionskurven auf, [Daten hier herunterladen](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4-Reflexionsgrad :: 250–2.500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4-Reflexionsgrad :: 400–1.000 nm</p></figcaption></figure>Die diffusen T4P-Referenztargets weisen die folgenden Reflexionskurven auf, [Daten hier herunterladen](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P-Reflexionsgrad :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P Reflexionsgrad :: 400–1000 nm</p></figcaption></figure>Ein Blick auf das Reflexionsdiagramm zeigt, dass die Werte die Wellenlänge (x-Achse) im Verhältnis zum Reflexionsprozentsatz (y-Achse) darstellen. Wenn wir ein Bild des Kalibrierungsziels aufnehmen, erstellen wir eine Beziehung zwischen Pixelwert und Reflexionsprozentsatz innerhalb des Spektrums, für das die einzelnen Sensorbereiche der Kamera empfindlich sind.

Das bedeutet, dass Sie bei jedem Bild, das Sie mit unseren Kameras aufnehmen, ein Foto unserer Reflexionsziele wie beispielsweise das [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) oder [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), verwenden, um die Bilder hinsichtlich der Reflexion zu kalibrieren. Nach der Kalibrierung entspricht jedes Pixel im Bild dem Reflexionsprozentsatz.

Bei den **Survey3** gilt: Wenn Sie die kalibrierten Bilder in Chloros als typisches JPG oder TIFF ausgeben, wird der Reflexionsgrad in Prozent berechnet, indem der Pixelwert durch die Bittiefe des Bildformats geteilt wird. Bei JPG dividieren Sie also durch 255, bei TIFF durch 65.535. Sie können auch die Ausgabe im PERCENT-Format in Chloros wählen; dann liegt der Wert jedes Pixels im Bereich von 0,0 bis 1,0 Prozent (0 % bis 100 % Reflexionsgrad). Beachten Sie jedoch, dass einige Bildbearbeitungsprogramme keine Bilder im Prozent-Format (Gleitkomma) verarbeiten können und diese zudem viel Speicherplatz beanspruchen.

{% hint style="info" %}
**Die LATTICE-Reflexionswerte verwenden einen anderen Pixelskalierungsfaktor.** LATTICE-Reflexionswerte werden mit DN 32768 = 100 % Reflexion (nicht 65535) gespeichert, und jede Datei enthält einen XMP-Tag „`Chloros:PixelScale`“, der den Skalierungsfaktor angibt. Lesen Sie das Tag aus und dividieren Sie den Wert durch diesen, anstatt von einer Konstante auszugehen – siehe [Ausgabebildformate](output-image-formats.md).
{% endhint %}

## Kalibrierungsziele mit LATTICE-Kameras

Bei LATTICE-Kameras ist ein Kalibrierungsziel für die Reflexion **optional**: Chloros kann die Reflexion stattdessen auf die von einem DAQ-Lichtsensor gemessene nach unten gerichtete Bestrahlungsstärke beziehen (ρ = π·L/E). Die Referenz wird über die Einstellung „Reflexionsquelle“ ausgewählt (Projekteinstellungen in der GUI; `--reflectance-source` im CLI; `reflectance_source` im SDK) ausgewählt:

| Wert | Verhalten |
| --- | --- |
| `auto` *(Standard)* | Ein QA-konformes Ziel im Bild ist die **absolute Referenz**; wenn kein Ziel vorhanden ist oder die QA fehlschlägt, greift Chloros auf die DAQ-Downwelling-Teilung zurück. |
| `target` | Strikt zielorientiert – keine DAQ-Ersetzung. |
| `daq` | DAQ-maßgeblich – die Downwelling-Messung ist immer die Referenz. |

Zusätzliches Zielverhalten für LATTICE:

* **Zielgeometrien** – ArUco-markierte Platten, Platten mit festem ROI und Streifenziele werden alle unterstützt; die Geometrie stammt aus der Zielkonfiguration des Projekts.
* **Pro Einheit gemessene Ziel-Daten** — `--target-reflectance-dir DIR` verweist auf ein Verzeichnis mit pro Einheit gemessenen Zielreflexions-Scans (`<serial>.csv`, die anhand der Seriennummer/des QR-Codes der Zieleinheit abgerufen werden). Bei einem Fehltreffer greift Chloros auf die nominalen T3/T4P-Spektren zurück.
* **Zeitliche Verankerung** — Ein erkanntes Ziel kalibriert die umgebenden Bilder und wird zwischen den Zielsichtungen beibehalten.

Die vollständige Flag-Semantik und Beispiele finden Sie in der [CLI-Referenz](reference/cli-reference.md) (siehe „Export-Schalter pro Produkt“).

### F988

„Die Reflektanz von F988 wird mithilfe einer Reflektanzplatte vor Ort kalibriert: Da das Band außerhalb des kalibrierten Bereichs des DAQ-Lichtsensors liegt, wendet Chloros Ihre letzte Plattenerfassung an und behält diese zwischen den Plattenerfassungen bei.“

Wird F988 mit einer reinen DAQ-Kalibrierung ausgeführt, lehnt Chloros die DAQ-basierte Reflexion für dieses Band ab und gibt den Grund dafür an (Überspringgrund `dls-uncalibrated-band-988`); der Panel-Workflow ist der unterstützte Weg.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
