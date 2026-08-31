# Bildebenen

Über das **Ebenen-Dropdown-Menü** oben rechts im Bildbetrachter können Sie zwischen allen Versionen des angezeigten Bildes wechseln – von der Originalaufnahme über jedes verarbeitete Produkt bis hin zu den berechneten Indexbildern –, ohne den Betrachter verlassen zu müssen.

## Was sind Bildebenen?

Eine „Ebene“ in Chloros ist eine **Produktdatei**, die einem Quellbild zugeordnet ist. Beim Import erhalten Sie die Quelldateien; bei der Verarbeitung wird für jedes vom Lauf erstellte Produkt eine Ebene hinzugefügt. Exportierte Dateien behalten den Quelldateinamen bei – es ist der**Ordner**, der das Produkt identifiziert, und der Ebenenname ist die Bezeichnung von Chloros für diesen Ordner.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Die Ebenenliste

### Immer vorhanden

| Ebene | Was ist das? |
| --- | --- |
| **JPG**(oder**PNG**/**TIFF**) | Die Basisdatei, die mit der Erfassung eingegangen ist. Survey3 importiert neben jedem `.RAW` eine `.JPG`; LATTICE-Aufnahmen liefern eine PNG oder TIFF-Vorschau mit. Beschriftet entsprechend dem tatsächlich importierten Inhalt |
| **RAW (Original)** | Das ursprüngliche RAW-Bild, für die Anzeige debayert, ohne dass Korrekturen angewendet wurden. Verfügbar ab dem Zeitpunkt des Imports – es bedarf keiner weiteren Verarbeitung |

Eine LATTICE-Aufnahme, deren Basisdatei **der** RAW-Frame ist, hat keinen separaten Basiseintrag: `RAW (Original)` deckt dies bereits ab.

### Survey3-Verarbeitungsprodukte

| Ebene | Geschrieben in | Vorhanden, wenn |
| --- | --- | --- |
| **RAW (Target)** | — | Das Bild wurde als Kalibrierungsziel identifiziert |
| **RAW (Reflectance)** | `Reflectance_Calibrated_Images/` | Die Reflexionskalibrierung wurde für dieses Bild erfolgreich durchgeführt |
| **Vignettenkorrektur**| `Vignette_Corrected_Images/` | Das Bild konnte nicht reflektanzkalibriert werden**und** die *Vignettenkorrektur* war aktiviert |
| **Sensorantwort**| `Sensor_Response_Images/` | Das Bild konnte nicht reflektiv kalibriert werden**und** die *Vignettierungskorrektur* war deaktiviert |
| **Weißabgleich** | `White_Balanced_Images/` | Ein Produkt mit Weißabgleich wurde geschrieben |

{% hint style="info" %}
**Vignettenkorrektur und Sensorantwort sind Alternativen, niemals beides.** Pro Durchlauf existiert genau ein unkalibriertes Fallback-Produkt für jedes Kameramodell, und der Schalter *Vignettenkorrektur* wählt aus, welches. Siehe [Projekteinstellungen](../project-settings/project-settings.md).
{% endhint %}

### LATTICE-Ebenen

LATTICE erfasst die Aufteilung in diese Ebenen in einem einzigen Verarbeitungsdurchlauf. Welche Ebenen vorhanden sind, hängt von den produktbezogenen Exportoptionen in den Projekteinstellungen und von den für die jeweilige Kamera geltenden Einstellungen ab.

| Ebene | Wird geschrieben in | Gilt für |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB und Multispektral |
| **RAW (Vorschau)** | `Preview_Images/` | Multispektral (Falschfarben-Streckung) |
| **Weißabgleich** | `Preview_Images/` | RGB-Masterkameras — die Vorschau „RGB“ ist unter diesem Namen registriert, damit sie mit der gleichnamigen Ebene „Survey3“ übereinstimmt |
| **RAW (Strahlungsintensität)** | `Radiance_Images/` | Nur multispektral |
| **RAW (Rezidenz)** | `Reflectance_Calibrated_Images/` | Nur multispektral und nur, wenn ein passender `.daq`-Downwelling-Datensatz oder ein QA-konformes In-Frame-Ziel das Bild abdeckt |

RGB-Masterkameras verfügen über keine bandweise Radiometrie, daher werden Strahlungsintensität und Reflektanz für sie als **nicht anwendbar** übersprungen – dies wird im Protokoll vermerkt, anstatt dass der Vorgang stillschweigend fehlschlägt.

### Index-, LUT- und Sandbox-Ebenen

| Ebenenmuster | Beispiel | Herkunft |
| --- | --- | --- |
| **RAW (`<INDEX>`-Index)** | `RAW (NDVI Index)` | Je einen pro in den Projekteinstellungen konfiguriertem Index, berechnet während der Verarbeitung |
| **`<INDEX>` LUT** | `NDVI LUT` | Die farbabgebildete Version eines Indexes |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Eine pro [Index/LUT-Sandbox](index-lut-sandbox.md)-Exportdurchlauf |

Wenn derselbe Indexname mehrmals mit unterschiedlichen Einstellungen konfiguriert wird, erhalten der zweite und alle folgenden Einträge eine Nummer im Namen (`RAW (NDVI2 Index)`), damit die Ebenen unterscheidbar bleiben.

***

## Verwendung der Ebenenauswahl

1. Öffnen Sie ein Bild im Vollbildmodus, indem Sie auf eine Miniaturansicht im Raster klicken.
2. Klicken Sie auf das **Ebenen-Dropdown-Menü** oben rechts im Viewer.
3. Wählen Sie eine Ebene aus – das Bild wird sofort aktualisiert.

Das Dropdown-Menü listet **JPG, RAW (Original), RAW (Ziel), RAW (Reflexionsgrad)** in dieser Reihenfolge an erster Stelle auf und zeigt alle weiteren Einträge in der Reihenfolge an, in der die Produkte registriert wurden.

### Ebenenpräferenz beim Navigieren

Durch Drücken von **←**/**→** gelangst du zum nächsten Bild, wobei versucht wird, dich auf derselben Ebene zu belassen:

1. **Exakte Übereinstimmung zuerst** – wenn das nächste Bild eine Ebene mit demselben Namen hat, wird diese angezeigt. Dadurch bleiben Sie auf der Ebene „`RAW (NDVI Index)`“, während Sie einen ganzen Satz durchgehen
2. **Dann Übereinstimmung nach Typ** – eine Indexebene sucht nach einer beliebigen Indexebene, eine LUT nach einer beliebigen LUT, Reflektanz nach Reflektanz, Ziel nach Ziel, Original nach Original, Basis nach Basis
3. **Dann, nur für Export-Ebenen** – der Name wird beibehalten, auch wenn die Ebenenliste noch nicht auf dem neuesten Stand ist, da die Datei bereits auf der Festplatte vorhanden ist. Dadurch können Sie Produkte überprüfen, während ein Lauf sie noch schreibt
4. **Andernfalls** – die erste verfügbare Ebene, bei der es sich normalerweise um das Basisbild handelt

Die Sidecar-Dateien „`.daq`“ und „`.csv`“ im Projekt werden bei der Navigation mit den Pfeiltasten übersprungen, sodass man beim Durchblättern der Bilder niemals auf eine Lichtsensoraufnahme stößt.

Zoom und Schwenken werden auch über Bilder hinweg beibehalten, was den Vorher-Nachher-Vergleich derselben Bildposition vereinfacht.

***

## Pixelwerte nach Ebene verstehen

Das [Fenster „Cursorwerte“](opening-an-image-full-screen.md#cursor-values) zeigt den tatsächlichen Wert pro Kanal unter Ihrem Cursor in der Einheit an, in der die Ebene gespeichert ist. Die Spalten ändern sich je nach Ebene:

| Ebene | Angegebene Einheit | Anmerkungen |
| --- | --- | --- |
| Basis (JPG / PNG / TIFF-Vorschau) | DN, 0–255 | Anzeigewerte, gammakorrigiert in RGB. Nur zur visuellen Überprüfung |
| RAW (Original) | DN | Digitale Rohwerte des Sensors. Die Histogrammachse gibt die Bit-Tiefe an: 255 (8 Bit), 4095 (12 Bit) oder 65535 (16 Bit) |
| RAW (Debayered) | DN | Linear, keine Bildstreckung |
| RAW (Vorschau) / Weißabgleich | DN | Anzeigeergebnis – gestreckt oder gamma-korrigiert. Nicht zur Messung geeignet |
| RAW (Strahlungsintensität) | **W/m²/sr/nm** | Physikalische Strahlungsintensität als Float32. Keine DN-Spalte |
| RAW (Reflexionsgrad) | DN **und %** | Prozentwert, berechnet anhand der dateieigenen Skalierung – siehe unten |
| Index-/LUT-/Sandbox-Exporte | Indexwert oder RGB-Komponenten | Eine einkanalige Indexdatei gibt den Indexwert an; eine farbabgebildete LUT-Datei gibt die Komponenten Red/Green/Blue an |

### Reflexionsgrad: Die Skalierung gilt pro Datei

{% hint style="warning" %}
**„Durch 65.535 teilen“ ist nur für Survey3 korrekt.** Die Reflektanz von LATTICE wird in einem anderen Maßstab gespeichert, und die Verwechslung der beiden Divisoren ist der häufigste Grund dafür, dass Reflektanzwerte genau halb so groß ausfallen, wie sie eigentlich sein sollten.
{% endhint %}

| Quelle | DN, der einer Reflektanz von 1,0 entspricht | Identifiziert durch |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Das XMP-Tag `Chloros:PixelScale=32768`, das in jeden LATTICE-Reflexionsgrad-Export eingefügt wird. Der 2-fache Headroom bedeutet, dass ρ über 1,0 darstellbar ist und nicht abgeschnitten wird |
| **Survey3**|**65535** | Kein Chloros-XMP-Skalierungs-Tag – die Kalibrierung Survey3 schreibt ρ × dtype-max und schneidet bei 1,0 ab |

Für GIS und Skripte: Lesen Sie `Chloros:PixelScale` aus der Datei aus und dividieren Sie durch diesen Wert. Fehlt das Tag, ist die Datei Survey3-skaliert (65535). Der Viewer, die Index-/LUT-Sandbox und der Indexexport berechnen den Maßstab alle auf dieselbe Weise, sodass der am Cursor angezeigte Wert dem Wert entspricht, der bei der Indexberechnung verwendet wurde.

Formatspezifische Speicherung zusätzlich zu diesem Maßstab:

* **TIFF (32-Bit, Prozent)** speichert DN / 65535 als Gleitkommazahl
* **PNG (8 Bit)**und**JPG (8 Bit)** speichern DN × 255 / 65535
* Ein **8-Bit-TIFF-Export einer 8-Bit-Quellaufnahme** wird auf 0–255 begrenzt, anstatt neu skaliert zu werden, und enthält bewusst kein Skalierungs-Tag. Das Bedienfeld zeigt für diese Dateien nur den DN an, ohne Prozent-Spalte

### Indexwertbereiche

| Indexfamilie | Typischer Bereich | Lesewert |
| --- | --- | --- |
| Normalisierte Differenz (NDVI, GNDVI, NDRE, ENDVI…) | −1 bis +1 | Gesunde Vegetation üblicherweise 0,4–0,9; nackter Boden nahe 0; Wasser negativ |
| Bodenbereinigt (SAVI, OSAVI, MSAVI2…) | etwa −1 bis +1,5 | Ähnlicher Messwert wie NDVI, wobei der Bodenhintergrund unterdrückt ist |
| Verhältnis (GRVI, GCI, MSR, CIRE…) | nach oben unbegrenzt | Die Verhältnisse steigen unbegrenzt an, wenn der Nenner gegen Null geht |
| EVI / LAI | 0 bis ~1, 0 bis ~3,5 | Wolken und andere gesättigte Pixel führen dazu, dass beide Werte den Bereich überschreiten – maskieren Sie diese zunächst |

Die genauen Formeln hinter jeder Voreinstellung finden Sie unter [Formeln für multispektrale Indizes](../project-settings/multispectral-index-formulas.md).

***

## Gängige Arbeitsabläufe

### Vorher-Nachher-Vergleich

1. Wählen Sie **RAW (Original)** aus und beachten Sie die Vignettierung sowie die unkalibrierten Werte
2. Wechseln Sie zu **RAW (Reflexion)**

3. Vergleichen Sie – Vignettierung entfernt, Werte kalibriert. Zoom und Verschiebung bleiben erhalten, sodass Sie denselben Boden betrachten

### Einen Index über einen gesamten Satz hinweg überprüfen

1. Öffnen Sie das erste verarbeitete Bild und wählen Sie die Indexebene aus
2. Drücken Sie wiederholt **→** – die Indexebene folgt Ihnen von Bild zu Bild
3. Beobachten Sie dabei das Histogramm in der Seitenleiste: Ein Bild, dessen Verteilung sprunghaft abweicht, sollte genauer untersucht werden

### Kalibrierungsziele überprüfen

1. Wählen Sie **RAW (Ziel)** in einem Zielbild aus
2. Vergewissern Sie sich, dass das Ziel deutlich sichtbar ist und erkannt wird
3. Wechseln Sie zum nächsten Zielbild – die Ziel-Ebene folgt Ihnen

### Überprüfen Sie die Genauigkeit der Reflexionswerte

1. Wählen Sie **RAW (Reflexion)**

2. Lesen Sie die Spalte**%** im Fenster „Cursor-Werte“ ab – sie ist für diese Datei bereits korrekt skaliert
3. Führen Sie eine Plausibilitätsprüfung anhand bekannter Materialien im Bild durch: Gesunde Vegetation weist hohe Werte bei NIR und niedrige Werte im Rotbereich auf; ein Kalibrierungsziel sollte Werte nahe seinem veröffentlichten Reflexionsgrad aufweisen

***

## Fehlerbehebung

### Eine erwartete Ebene ist nicht in der Auswahlliste enthalten

**Mögliche Ursachen**

* Das Bild wurde nie verarbeitet – es existieren nur die Basisebene und die `RAW (Original)`-Ebene
* Der Export-Schalter für das Produkt ist in den Projekteinstellungen deaktiviert
* Das Produkt ist für diese Kamera nicht verfügbar (Strahlungsintensität und Reflektanz bei einer RGB-Master-Kamera; beliebiger Index bei einer einkanaligen M3M-Mono-Kamera)
* Für die Reflektanzkalibrierung standen keine Daten zur Verfügung – keine `.daq`-Abdeckung in Abwärtsrichtung und kein QA-konformes Zielobjekt im Bild –, sodass das Bild auf „Vignettenkorrektur“ oder „Sensorreaktion“ zurückfiel

**Vorgehensweise**

1. Überprüfen Sie das Protokoll des Laufs: Chloros gibt an, wann ein angefordertes Exportprodukt nicht erstellt werden konnte und warum
2. Überprüfen Sie die produktbezogenen Export-Schaltflächen in den [Projekteinstellungen](../project-settings/project-settings.md)
3. Vergewissern Sie sich, dass der Produktordner im Ausgabebaum des Projekts vorhanden ist
4. Führen Sie die Verarbeitung erneut durch, wobei das Produkt aktiviert ist

### Die Ebenenliste scheint veraltet zu sein

Chloros scannt die Produktordner des Projekts während eines laufenden Durchlaufs erneut und gleicht fehlende Ebenenzuordnungen anhand der tatsächlich auf der Festplatte vorhandenen Daten an. Daher erscheint eine Ebene, deren Export normal abgeschlossen wurde, von selbst innerhalb einer Abfrage. Das Wechseln weg vom Bild und zurück erzwingt eine neue Auflösung.

### Die Reflexionswerte scheinen nur halb so hoch zu sein, wie sie sein sollten

Sie dividieren mit ziemlicher Sicherheit eine LATTICE-Datei durch 65535. Verwenden Sie `Chloros:PixelScale` (32768) oder lesen Sie die Spalte **%**, in der dieser Wert bereits angewendet wurde.

### Die Indexebene ist vorhanden, das Bild ist jedoch leer

Der Index benötigt Bänder, über die Ihre Ebene nicht verfügt – beispielsweise ein Index, der einen dritten Kanal ausliest, der auf eine Ein- oder Zweikanaldatei angewendet wird. Wechseln Sie zu einer Mehrbandebene (Reflexionsgrad oder debayert) oder wählen Sie einen Index, der zum Filter der Kamera passt.

***

## Nächste Schritte

* [**Ein Bild im Vollbildmodus öffnen**](opening-an-image-full-screen.md) – Cursoranzeige, Histogramm und GSD-Steuerung
* [**Index/LUT-Sandbox**](index-lut-sandbox.md) – interaktive Indexvisualisierung und Export
* [**Multispektrale Indexformeln**](../project-settings/multispectral-index-formulas.md) – die Indexreferenz
* [**Abschluss der Verarbeitung**](../processing-images-gui/finishing-the-processing.md) – die Ausgabeverzeichnisstruktur, auf die diese Ebenen verweisen
