---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Ausgabe-Bildformate

Chloros exportiert bearbeitete Produkte in vier Dateiformaten. Wählen Sie das Format in den Projekteinstellungen (GUI) aus, entweder mit `--format` (CLI) oder mit `export_format` (SDK). Die Optionen CLI und SDK akzeptieren genau die unten aufgeführten Zeichenfolgen.

| Formatzeichenfolge | Erweiterung | Pixeltyp | Pixelbereich | Anmerkungen |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(Standard)* | `.tif` | uint16-Zahl | 0 – 65535 | Empfohlen für Photogrammetrie / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % Reflexionsgrad. Einige Anwendungen können TIFF-Dateien mit Gleitkommazahlen nicht lesen; die Dateien sind größer. |
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

Der Kameraordner lautet `LATT-<sensor>-<lens>-F<filter>` für LATTICE und `<model>_<filter>` (z. B. `Survey3N_RGN`) für Survey3. **Jedes exportierte Produkt behält den Namen der Quelldatei bei – der Ordner identifiziert das Produkt, nicht eine Dateinamenerweiterung.** Die vollständigen Regeln finden Sie unter [Speicherorte der Ausgabedateien](reference/cli-reference.md) in der CLI-Referenz.

## LATTICE-Produkte (Erfassungs- und Exportstufen)

Ein LATTICE-Rohframe wird in einem einzigen Durchlauf in alle angeforderten Produkte aufgeteilt. Jeder Produkttyp verfügt über einen eigenen Schalter (GUI-Kontrollkästchen oder CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, standardmäßig alle aktiviert):

| Ebene | Inhalt | Datentyp |
| --- | --- | --- |
| `raw` | Bayer-Daten direkt vom Sensor (Monochromkameras: das Einzelband). Die Verarbeitung beginnt immer mit den Rohdaten. | Wie aufgenommen |
| `debayered` | Lineare Demosaikierung – 3-Kanal für M3C, 1-Kanal-Graustufen für M3M. | Linear DN |
| `radiance` | Absolute spektrale Strahlungsintensität aus der vollständigen radiometrischen Kette, in **W/m²/sr/nm**. Wird unabhängig vom gewählten Exportformat stets als 32-Bit-Wert „TIFF“ (`tiff32/Radiance_Images/`) geschrieben. | float32 |
| `reflectance` | Reflexionsgrad ρ, wobei **DN 32768 = ρ 1,0 (100 %)** mit Spielraum bis ρ 2,0. Pix4D-kompatibel. | uint16 |
| `preview` | Anzeigegerechter Render: RGB = Weißabgleich + Gamma; multispektral = Falschfarben-Streckung. | 8-Bit-Anzeige |

## Auslesen der Reflektanz-Pixelwerte

Die Reflektanz wird als ganzzahliger Digitalwert gespeichert, und **der DN-Wert, der ρ = 1,0 (100 % Reflektanz) entspricht, hängt von der Quellkamera ab**:

| Quellkamera | ρ = 1,0 entspricht DN | Erkennungsmerkmal |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (Spielraum bis zu ρ 2,0) | Das XMP-Tag `Chloros:PixelScale=32768` ist in die Datei eingebettet. |
| Survey3 | `65535` (abgeschnitten bei ρ 1,0) | Keine `Chloros:*`-XMP-Tags – das Fehlen ist das Signal. |

**Lies den `Chloros:PixelScale`-XMP-Tag und dividiere durch diesen**, anstatt von einer Konstante auszugehen. Das Tag ist im uint16-Bereich definiert, daher bleibt der Wert über alle Ausgabeformate hinweg, die eine Neuskalierung vornehmen, unverändert – normalisieren Sie den gespeicherten Datentyp daher zunächst wieder auf uint16 (×257 von 8-Bit, ×65535 von float32).

{% hint style="warning" %}
**Ein Fall weist aufgrund der Konzeption keinen Skalierungsfaktor auf.** Wenn eine 8-Bit-Quellaufnahme (BayerRG8) als 8-Bit-TIFF geschrieben wird, beschränkt die Pipeline den Wert auf 0–255, anstatt ihn neu zu skalieren, sodass die Datei keinen Skalierungsfaktor enthält – Chloros lässt `Chloros:PixelScale` hier bewusst weg. Fehlt das Tag in einer LATTICE-Reflexionsdatei, sollte man nicht von einem Maßstab ausgehen; exportieren Sie die Datei stattdessen mit 16 Bit oder 32 Bit neu.
{% endhint %}

Die vollständigen Regeln (einschließlich der MicaSense-kompatiblen Tags) finden Sie unter **„Reflexionspixel lesen“** in der [CLI-Referenz](reference/cli-reference.md).
