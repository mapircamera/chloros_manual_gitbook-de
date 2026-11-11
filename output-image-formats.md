---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Ausgabe-Bildformate

Chloros exportiert verarbeitete Produkte in vier Dateiformaten. Wählen Sie das Format in den Projekteinstellungen (GUI), mit `--format` (CLI) oder mit `export_format` (SDK) aus. Die CLI und das SDK akzeptieren genau die unten aufgeführten Zeichenfolgen.

| Formatzeichenfolge | Dateiendung | Pixeltyp | Pixelbereich | Anmerkungen |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(Standard)* | `.tif` | uint16-Zahl | 0 – 65535 | Empfohlen für Photogrammetrie / GIS. |
| `TIFF (8-bit)` | `.tif` | uint8-Zahl | 0 – 255 | Für Aufnahmen, die vollständig in 8 Bit erfolgen und nur 8-Bit-Formate bieten: Chloros führt keine Hochskalierung der Bittiefe durch. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % Reflexionsgrad. Einige Anwendungen können TIFF-Dateien mit Gleitkommadaten nicht lesen; die Dateien sind größer. |
| `PNG (8-bit)` | `.png` | uint8-Zahl | 0 – 255 | Verlustfreie Komprimierung, geeignet für die Anzeige im Web und zur Visualisierung. |
| `JPG (8-bit)` | `.jpg` | uint8-Zahl | 0 – 255 | Verlustbehaftete Komprimierung, kleinste Dateien. |

## Speicherort der Ausgabedateien

Die Dateien werden im Projektordner abgelegt, gruppiert nach Kamera und anschließend nach Dateiformat:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Der Kameraordner lautet `LATT-<sensor>-<lens>-F<filter>` für LATTICE und `<model>_<filter>` (z. B. `Survey3N_RGN`) für Survey3. **Jedes exportierte Produkt behält den Namen der Quelldatei bei – der Ordner identifiziert das Produkt, nicht eine Dateinamenerweiterung.** Die vollständigen Regeln finden Sie unter [Speicherort der Ausgabedateien](reference/cli-reference.md) in der CLI-Referenz.

## LATTICE-Produkte (Erfassungs- und Exportstufen)

Ein LATTICE-Rohbild wird in einem einzigen Durchlauf in alle angeforderten Produkte aufgeteilt. Jeder Produkttyp verfügt über eigene Umschaltoptionen (GUI-Kontrollkästchen oder CLI-Befehle `--debayered` / `--preview` / `--radiance` / `--reflectance`, standardmäßig alle aktiviert):

| Ebene | Inhalt | Datentyp |
| --- | --- | --- |
| `raw` | Bayer-Daten direkt vom Sensor (Monochromkameras: das einzelne Band). Die Verarbeitung beginnt immer mit den Rohdaten. | Wie aufgenommen |
| `debayered` | Lineares Demosaicing – 3-Kanal für M3C, 1-Kanal-Graustufen für M3M: Der Dunkeloffset des Kalibrierungspakets wird entfernt, anschließend erfolgt ein kantensensitives Demosaicing ohne Linearitätskorrektur, Flatfield oder Unmix. Ein `process`-Export schreibt in jedem Verarbeitungsmodus dieselbe Datei, die bei einer Aufnahme auf dieser Stufe gespeichert wird. | Lineares DN |
| `radiance` | Absolute spektrale Strahlungsintensität aus der vollständigen radiometrischen Kette, in **W/m²/sr/nm**. Wird unabhängig vom gewählten Exportformat immer als 32-Bit-TIFF-Datei (`tiff32/Radiance_Images/`) geschrieben. | float32 |
| `reflectance` | Reflexionsgrad ρ, wobei **DN 32768 = ρ 1,0 (100 %)** gilt, mit Spielraum bis ρ 2,0. Pix4D-kompatibel. | uint16 |
| `preview` | Anzeigegerechter Render: RGB = Weißabgleich + Gamma; multispektral = Falschfarben-Streckung. | 8-Bit-Anzeige |

### Ausrichtung

`raw`- und `debayered`-Dateien behalten stets die **native** Ausrichtung des Sensors bei; die Spiegelungs-/Drehungseinstellung einer Kamera wird nur auf die anderen Produkte (Vorschau, Strahlungsintensität, Reflexionsgrad, Index) angewendet. Bei Array-Aufnahmen bleiben die Dateien aller Elemente in ihrer nativen Ausrichtung, es sei denn, Sie geben eine Ausrichtung an (`--orient` in der CLI, `orient=True` im SDK). Jede Datei speichert die zu ihren Pixeln gehörende Ausrichtung im XMP – `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` – und `ChlorosOrientationApplied` gibt an, ob die eigenen Pixel diese Information enthalten (`false` bei Raw- und debayered-Dateien). Der Chloros Image Viewer passt RAW- und debayerte Dateien ausschließlich zur Anzeige an diesen Datensatz an; die Dateien selbst werden niemals überschrieben. Eine Datei ohne diesen Datensatz (die vor Version 1.3.0 oder mit einem anderen Tool erstellt wurde) wird so angezeigt, wie sie gespeichert ist.

## Clamp-Masken

Die Radiance- und Reflektanz-Produkte einer multispektralen M3C-Kamera durchlaufen eine 3×3-Band-Entmischung. Wenn die Entmischung in einem Band einen negativen Wert erzeugt, wird dieser Wert auf Null gekürzt (Live-Ansichten werden auf genau dieselbe Weise gekürzt). Neu in Version 1.3.0: Ein Produkt, in dem mindestens ein Pixel abgeschnitten wurde, erhält eine zugehörige **Clamp-Maske**, die genau angibt, um welche Pixel und Bänder es sich handelt:

* **Wo:**in einem Ordner „`clamp_masks/`“**neben** den Produktordnern, niemals innerhalb eines solchen: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` und `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Was:** eine einbandige `uint8`-TIFF-Datei auf dem produkt-eigenen Pixelraster (einschließlich Ausrichtung, ROI, Binning und Array-Ausrichtung). Bit 0 (`mask & 1`) ist gesetzt, wenn das Band im R-Kanal des Produkts beschnitten wurde, Bit 1 (`& 2`) für den G-Kanal, Bit 2 (`& 4`) für den B-Kanal. Der Anteil der beschnittenen Pixel pro Band wird ebenfalls als `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` und `Chloros:BandBUnmixClampFraction` im Produkt vermerkt.
* **So wird dies im Produkt angegeben:** Das XMP-Tag `Chloros:UnmixClampMask` enthält den Pfad der Maske relativ zum Ordner des Produkts (`../clamp_masks/<product folder>/<name>_clampmask.tif`, immer durch `/` getrennt), `none`, wenn kein Pixel abgeschnitten wurde (es wird keine Maske geschrieben), oder `unavailable`, wenn keine Maske geschrieben werden konnte. Bei einem Produkt ohne Tag wurde die Entmischung nie durchgeführt (bei RGB- und Mono-Kameras, Raw- und debayered-Ebenen).

Da kein Produktordner eine Maske enthält, können Sie einen Produktordner wie beispielsweise `Reflectance_Calibrated_Images/` unverändert – einschließlich aller Unterordner – an Pix4D, Metashape oder andere Photogrammetrie-Software übergeben – übergeben Sie den Produktordner, nicht den darüber liegenden Formatordner. Der Chloros-eigene Import überspringt Clamp-Masken. Die CLI zählt sie separat: `Image products written: N (plus M clamp masks)`.

## Indexprodukte

Jeder Index landet in einem eigenen `<INDEX>_Index_Images/`-Ordner (z. B. `NDVI_Index_Images/`).

* **Ganzzahlformate**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) speichern eine**lineare Dehnung pro Bild** und keine −1…1-Zuordnung: DN 0 ist der niedrigste Indexwert dieses Frames und der Vollbereich der höchste. Die Datei enthält `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` und `Chloros:IndexMax`, also auch `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Da jedes Bild seine eigenen Endpunkte hat, sollten Bilder als Werte und nicht als DN verglichen werden.
* **`TIFF (32-bit, Percent)`** speichert die Indexwerte selbst als float32 ohne Skalierung – die beste Wahl für quantitative Auswertungen über einen Flug hinweg.

Neu in Version 1.3.0: Ein Indexprodukt beschreibt **sich selbst**, anstatt die Tags der Rohdatenquelle zu wiederholen. Seine GPS- und Kameratags werden aus dem Produkt kopiert, aus dem es berechnet wurde (dem Reflektanzprodukt, sofern vorhanden), und dann:

| Tag | Wert |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (sodass die Datei niemals erneut als Aufnahme importiert wird) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / die verwendete Formel |
| `Chloros:IndexLevel` | `reflectance` oder `radiance` – woraus der Index berechnet wurde (fehlt, wenn keines von beiden zutrifft) |
| `Chloros:IlluminationInvariant` | `True` für einen aus der Reflexion berechneten Index, `False` aus der Strahlungsintensität oder aus Sensorwerten |
| `Chloros:ChlorosPipelineVersion` | die Chloros-Version, die den Index erstellt hat |

`Chloros:PixelScale` wurde aus den Indexprodukten entfernt – verwenden Sie stattdessen die oben genannten Werte `IndexMin` bzw. `IndexMax`. Das vollständige Rezept zum Zurücklesen finden Sie unter „Ein ganzzahliges Indexraster ist eine bildspezifische Kontraststreckung“ in der [CLI-Referenz](reference/cli-reference.md).

## Auslesen von Reflexions-Pixelwerten

Die Reflexion wird als ganzzahliger Digitalwert gespeichert, und **der DN-Wert, der ρ = 1,0 (100 % Reflexion) entspricht, hängt von der Quellkamera ab**:

| Quellkamera | ρ = 1,0 entspricht DN | Erkennungsmerkmal |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (Spielraum bis zu ρ 2,0) | Das XMP-Tag `Chloros:PixelScale=32768` ist in die Datei eingebettet. |
| Survey3 | `65535` (abgeschnitten bei ρ 1,0) | Kein `Chloros:PixelScale`-Tag – das Fehlen ist das Erkennungsmerkmal. (Survey3-Produkte enthalten ab Version 1.3.0 in jedem Verarbeitungsmodus weitere `Chloros:*`-Verarbeitungs-Tags, wie beispielsweise `Chloros:CalibrationApplied`.) |

**Lesen Sie das `Chloros:PixelScale`-XMP-Tag aus und dividieren Sie durch dessen Wert**, anstatt von einer Konstante auszugehen. Das Tag ist im uint16-Bereich definiert, daher bleibt es über alle Ausgabeformate hinweg, die eine Neuskalierung vornehmen, bei `32768` – normalisieren Sie den gespeicherten Datentyp zunächst wieder auf uint16 (×257 von 8-Bit, ×65535 von float32).

{% hint style="warning" %}
**8-Bit-Aufnahmen.** Die Exportdaten einer 8-Bit-Aufnahme sind 8-Bit-Daten. Ihre Reflexanz wird neu skaliert und mit `Chloros:PixelScale=32768` gekennzeichnet, sodass die Skalierung erhalten bleibt; 8-Bit-DN-Produkte und vor Version 1.3.0 erstellte Reflektanzdateien werden auf 0–255 beschnitten und enthalten kein Skalierungs-Tag. Wenn das Tag in einer LATTICE-Reflektanzdatei fehlt, sollten Sie nicht von einer Skalierung ausgehen; exportieren Sie stattdessen mit 16 Bit oder 32 Bit neu.
{% endhint %}

Die vollständigen Regeln (einschließlich der MicaSense-kompatiblen Tags) finden Sie unter **„Reflexionspixel lesen“** in der [CLI-Referenz](reference/cli-reference.md).
