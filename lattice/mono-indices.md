# Monochromkameras und Vegetationsindizes

## Eine Kamera = ein Band

Eine **M3M**-Kamera ist das Monochrom-Pendant zur Bayer-Kamera**M3C**: ein monochromer IMX265-Sensor hinter einem einzigen Schmalband-Interferenzfilter. Der Modellstring benennt das Band – `M3M-<lens>-F<wavelength>`, z. B. `M3M-L87-F685` (in Chloros als `LATT-M3M-L87-F685` angezeigt). Der Sensor liefert ein**einziges Graustufenband** ohne Bayer-Mosaik: Es gibt nichts zu demosaiken, kein kanalübergreifendes Übersprechen, das entmischt werden müsste, und keinen Weißabgleich, der eingestellt werden müsste.

Wichtige Konsequenzen, die Sie kennen sollten, bevor Sie ein Monochrom-System planen:

* **Strahlungsintensität und Reflexionsgrad sind pro Band vollständig definiert.**Es handelt sich um bandweise radiometrische Karten, sodass eine M3M-Kamera kalibrierte Float32-Strahlungswerte (W/m²/sr/nm) und uint16-Reflexionswerte (`32768` = ρ 1,0) liefert, genau wie ein M3C-Band. Mono-Bilder enthalten eine**identische** Sensorantwortmatrix – eine 3×3-Entmischung ist nicht erforderlich und wird auch nicht angewendet.
* **Eine einzelne Mono-Kamera kann keinen Vegetationsindex erzeugen.** NDVI, NDRE und ähnliche Indizes benötigen mindestens zwei Bänder. Um Indizes mit Mono-Hardware zu berechnen, kombiniert man mehrere M3M-Kameras – siehe unten.
* M3M-Kameras streamen **Mono12** (12 Bit, 2 Byte/Pixel über die Leitung), was für die [Bandbreitenplanung des Arrays](arrays.md#bandwidth-the-rules-of-thumb) von Bedeutung ist.

## Was Chloros bei Mono überspringt – und wie es Ihnen dies mitteilt

Die Stufen der Farb-Pipeline sind bei einem Einband-Sensor schlichtweg nicht anwendbar. Chloros **überspringt sie mit einer einzeiligen Meldung**, anstatt einen Fehler auszugeben, und führt sie dennoch normal für jede M3C-Kamera (Bayer) in derselben Sitzung aus:

| Stufe | Verhalten bei Mono (M3M) | Verhalten bei M3C |
| --- | --- | --- |
| Demosaic / Debayer | Übersprungen – die Exportstufe von `debayered` ist ein 1-Kanal-Graustufenbild. | 3-Kanal-Demosaic. |
| Weißabgleich (`lattice white-balance`) | Wird mit einer einzeiligen Meldung übersprungen. | Wird normal ausgeführt. |
| Farbprofil (`lattice color-profile`) | Wird mit einer einzeiligen Meldung übersprungen. | Wird normal ausgeführt. |
| Sättigung/Kontrast (`lattice color`) | Wird mit einer einzeiligen Meldung übersprungen. | Läuft normal ab. |
| Spektrale Entmischung | Identität (keine 3×3-Matrix). | 3×3-Matrix pro Kamera angewendet. |
| Strahlungsintensität / Reflexionsgrad | **Wird ausgeführt** — pro Band, vollständig kalibriert. | Wird pro Band ausgeführt. |

Die Benutzeroberfläche wendet dieselbe Filterung an: Bei einer Mono-Kamera blendet der Einstellungsbereich pro Kamera die Zeilen aus, die ausschließlich für RGB gelten (Weißabgleich, Gamma, Farbprofil, Sättigung, Kontrast, Kanalaufteilung), und das Live-Histogramm ist auf eine einzige **MONO**-Kurve festgelegt. Das Unterscheidungsmerkmal im gesamten Stapel ist das `M3M`-Token in der Modellzeichenfolge, das in der Benutzeroberfläche als SDK angezeigt wird.

## Indizes erfordern ≥ 2 Bänder: Ausrichten → Stapeln → Indizieren

Der Mono-Index-Workflow besteht immer aus denselben drei Schritten:

1. **Ausrichten** – Richten Sie mehrere M3M-Kameras auf verschiedene Wellenlängen aus (z. B. eine F650 „Red“ und eine F850 „NIR“), verbinden Sie diese zu einem [Multikamera-Array](arrays.md) und lassen Sie Chloros die Co-Registrierungs-Verzerrung zwischen den Kameras berechnen.
2. **Stack** – Die ausgerichteten Einzelbilder werden zu einem Multiband-Bild zusammengefügt (jede Kamera trägt ein benanntes Band bei).
3. **Index** – Berechnen Sie eine Indexformel über die Bänder des Stacks und rendern Sie diese optional über eine LUT.

In der Benutzeroberfläche entspricht diese gesamte Kette dem Anzeigemodus **„Combined Cameras“**: Die Live-Komposition ist bereits ausgerichtet, und der Index-Rechner des Arrays (unten) definiert die Formel, nach der sie gerendert wird. Aufgezeichnete Exporte können mit der Aufzeichnungsoption**„Aligned“** auf dieselbe Ausrichtung warpen lassen.

## Der Index-Rechner

Der Index-Rechner erstellt den Indexausdruck, der von der Live-Ansicht und den kameraspezifischen Index-Exporten verwendet wird. Es handelt sich um eine gemeinsame Oberfläche, die an zwei Stellen in der Seitenleiste der Registerkarte „Kameras“ geöffnet werden kann:

* **Pro Kamera**— Live-Vorschau →**Index**-Zahnrad (nur RGN/OCN/NGB-Bayer-Kameras; eine einzelne Mono-Kamera verfügt über keine Indexsteuerung, da ein Band keinen Index bilden kann).
* **Pro Array**— Array-Einstellungen → Live-Vorschau →**Index**-Zahnrad. Dies ist der Mono-Pfad: Die Bandliste umfasst**alle zugehörigen Kameras**, sodass ein Mono-Paar hier seine beiden Bänder beisteuert.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Die Steuerelemente, von oben nach unten:

* **Band-Chips** („Bänder – zum Hinzufügen zum Ausdruck anklicken“) – eine Schaltfläche pro verfügbarem Band, beschriftet mit Farbnamen + Wellenlänge in nm (doppelte Farbnamen werden zur Unterscheidung z. B. als „Farbe 850“ angezeigt). Durch Klicken wird das Band-Token an der Cursorposition eingefügt. Bänder von Kameras, die keine Strahlungsintensität pro Band erzeugen können (RGB/FRGB), werden herausgefiltert.
* **Operator- und Funktionsschaltflächen** – `+ - * / ( ) ^ ,` sowie `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Textfeld „Ausdruck“** — frei einzugebende Formel; der Platzhalter zeigt die klassische NDVI-Form `(NIR - Red) / (NIR + Red)` an. Eine schreibgeschützte, tokenisierte Vorschau darüber rendert Bandchips, Zahlen und Flags als unbekannte Token.
* **Gültigkeitsbanner**— grau „Leer – es wird kein Index angewendet“; grün „Gültiger Ausdruck“; rot mit dem spezifischen Parsing-Fehler (unbekanntes Band, mehrdeutiges Band, das von mehreren Kameras erfasst wird, fehlende Klammer, …); oder gelb, wenn der Ausdruck gültig, aber**konstant** ist (z. B. `X/X`, oder ein NDVI-Nenner, der mit `−` anstelle von `+` eingegeben wurde) – eine Konstante ordnet dem gesamten Bild eine einzige Farbe zu.
* Eine separate gelbe Warnung erscheint, wenn der angewendete Ausdruck in Ordnung ist, der **Live-Frame jedoch einheitlich** ist (flache oder gesättigte Szene) – der Histogramm-Zusammenbruch wird automatisch erkannt.
* **LUT anwenden**(standardmäßig aktiviert; deaktiviert = Graustufen-Streckung),**Stufe**2/3/5/7-Stop (standardmäßig 7-Stop) sowie die Eingabefelder**Min / Max**, die den Verlaufsbalken flankieren. „Min“ ist standardmäßig auf**0,2**eingestellt — dies zoomt die Farbskala in den für die Vegetation relevanten Bereich, während Werte darunter als Graustufen durchgelassen werden; setzen Sie „Min“ auf −1, um den vollen Indexbereich zu erhalten (die Schaltfläche**Zurücksetzen** stellt den Bereich von −1 bis +1 wieder her). „Max“ ist standardmäßig auf 1 eingestellt.
* **Live-Histogramm** der Indexverteilung – quadratwurzel-skalierte Balken, bernsteinfarbene p2-/p98-Perzentillinien, eine weiße Medianlinie und Anzeigen für Werte außerhalb des Bereichs („◀ N % &lt; lo“ / „hi &lt; N % ▶“), die oberhalb von 1 % bernsteinfarben werden und damit signalisieren, dass das Min/Max-Fenster erweitert werden sollte.
* Mit **Anwenden**wird der Ausdruck auf den Live-Stream angewendet; LUT-Anpassungen werden live übernommen, ohne dass „Anwenden“ gedrückt werden muss. Ausdrücke sind bewusst**nur sitzungsspezifisch** — sie werden zwischen den Sitzungen nicht beibehalten.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## Der CLI-Pfad

Dieselbe Kette aus „Align“ → „Stack“ → „Index“, durchgängig skriptfähig:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` ordnet die Symbole einer Voreinstellung den Bandnamen des Stacks zu. Zwei Regeln ersparen Ihnen einen fehlgeschlagenen Durchlauf:

* **Bei Symbolen wird die Groß-/Kleinschreibung beachtet** und sie müssen exakt mit den Kanalnamen des Presets übereinstimmen – Presets verwenden Kleinbuchstaben (NDVI lautet `red`,`nir`; siehe `--list-presets`). `--channel red=Red_660` funktioniert; `--channel RED=660` schlägt mit einem `channel_map missing entries`-Fehler fehl.
* Die Bandseite muss ein Band im ausgerichteten Stapel benennen (`lattice align-info --profile align.json` listet diese auf). Der Offline-Modus akzeptiert auch bandbasierte Indizes, z. B. `--channel red=0 --channel nir=1`.

`lattice index` läuft auch vollständig offline mit einem gespeicherten, ausgerichteten Mehrband-TIFF:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Index-Voreinstellungen

`lattice index --preset` (sowie die [Index/LUT-Sandbox](../image-viewer-gui/index-lut-sandbox.md) auf der Registerkarte „Bild“, die dieselbe Engine verwendet) enthält diese **22 Voreinstellungen**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Führen Sie `chloros-cli lattice index --list-presets` für die Formeln und Kanalsymbole der einzelnen Voreinstellungen und `--list-gradients` für die verfügbaren Farbverläufe aus. Benutzerdefinierte Formeln verwenden `--formula EXPR` mit derselben Syntax wie der Index-Rechner. Beachten Sie, dass diese Voreinstellungsliste spezifisch für die LATTICE-Index-Engine ist – das Dropdown-Menü „Verarbeitung“ in den Projekteinstellungen für importierte Bilddaten enthält eine andere Liste (siehe [Multispektrale Indexformeln](../project-settings/multispectral-index-formulas.md)).

Der vollständige Flagsatz (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, Ausrichtungs- und Verzerrungsregler für `--live`, und mehr) ist in der [CLI-Referenz § Index / Vegetationsmathematik](../reference/cli-reference.md#index--vegetation-maths) dokumentiert; SDK-Entsprechungen finden sich in der [SDK-Referenz](../reference/sdk-reference.md).

## Erfassen von Indexprodukten aus einem Mono-Array

Bei angeschlossenem Array und angewendeter Indexausdrucksform speichert `array-capture` (oder die GUI-Funktion **„Alle erfassen“**) die Exportstufen pro Kamera *sowie* das Index-Rendering — `--index`/`--no-index` schaltet diese Funktion auf dem CLI um, und die Erfassung umfasst standardmäßig alle anwendbaren Ebenen. Der Beitrag einer Monokamera zu jeder Aufnahmegruppe besteht aus ihrem einen Band auf den Ebenen „Raw“, „Debayered (Graustufen)“, „Radiance“ und „Reflectance“ sowie aus dem gemeinsamen kombinierten Index-Composite, wenn das Array im kombinierten Modus läuft. Siehe [Multi-Kamera-Arrays § Erfassung](arrays.md#capturing-monitoring-vs-analysis).
