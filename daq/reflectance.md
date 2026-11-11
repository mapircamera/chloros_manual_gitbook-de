# Arbeitsabläufe zur Reflektanzmessung

Ein DAQ-Lichtsensor wandelt radiometrische Bilddaten in Reflektanzwerte um. Es gibt zwei unterschiedliche Arbeitsabläufe:

1. **Einzel-Sensor** — Ein DAQ misst die nach unten gerichtete Bestrahlungsstärke, während eine Kamera Aufnahmen macht; Chloros
dividiert die von der Kamera erfasste Strahlungsintensität durch diesen Referenzwert.
2. **Zwei-Sensor-Verfahren** – Zwei DAQ-Sensoren, von denen einer den Himmel und einer ein Objekt erfasst, erzeugen eine spektrale Live-Reflexionskurve, ohne dass eine Kamera erforderlich ist.

## Ein Sensor + Kamera (Referenz für die nach unten gerichtete Strahlung)

Der DAQ fungiert als Sensor für die nach unten gerichtete Strahlung (DLS): Die Kamera misst die aufsteigende Strahlungsintensität **L**(W/m²/sr/nm), das DAQ misst die einfallende Bestrahlungsstärke**E** (W/m²/nm) und Chloros
berechnet die Reflektanz pro Band wie folgt:

> ρ = π · L / E

Der DAQ-Messwert ist stets **zeitlich auf die Belichtungszeit abgestimmt** – aus diesem Grund nutzen der DAQ und die Kameras eine gemeinsame, PTP-gesteuerte Uhr (siehe [DAQ-E-Vernetzung &amp; Zeitsynchronisation](ethernet-ptp.md)). Setzen Sie bei Arbeiten im Freien die „Sunshine“-Kosinuskappe auf und deklarieren Sie diese korrekt; die Deklaration der Kappe skaliert E direkt (siehe [Kappenprofile &amp; kalibrierter Bereich](caps-and-range.md)). Beachten Sie bei quantitativen Messungen die Gerätecharakteristik: Die quantitative Bestrahlungsstärke ergibt sich aus dem Mittelwert von Messwerten, die über mindestens 15 s erfasst wurden.

### Live-Erfassung

Verbinden Sie das DAQ auf der Registerkarte „Kameras“ mit einer Kamera: Das Einstellungsfeld jeder Kamera verfügt über ein Dropdown-Menü **Lichtsensor**, in dem alle angeschlossenen DAQs (DAQ-U/M/E) aus der Registerkarte „Lichtsensoren“ aufgelistet sind; bei einem synchronisierten Array wird die Auswahl des Lichtsensors für das gesamte Array auf jedes einzelne Element übertragen (einzelne Kameras können diese Einstellung jedoch weiterhin überschreiben). Nach der Verknüpfung werden die Spektren des Sensors in den DLS-Slot der Kamera eingespeist, und die Reflektanzwerte werden durch den entsprechenden Messwert dividiert. In einem Array verwendet ein Mitglied mit eigenem Sensor dessen Messwert – für seine Bänder im Live-Composite, seine Exportdaten und seine automatische Belichtung –, während die anderen Mitglieder die Werte des Arrays verwenden.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Zwei wichtige Verhaltensweisen:

* **Keine DAQ-Bindung → Reflektanz wird abgelehnt, nicht simuliert.** Chloros
lehnt das Reflektanzprodukt ab und protokolliert den Grund für das Überspringen, anstatt stillschweigend ein minderwertiges Produkt zurückzugeben.
* **Der verwendete Messwert bleibt erhalten.** Für jeden Reflexionsrahmen wird der tatsächlich angewendete DAQ-Messwert als `.daq`-Datei im Ordner `daq/` der Aufnahmen gespeichert, sodass die Aufnahme später erneut verarbeitet werden kann ([Aufzeichnung &amp; das .daq-Format](recording.md)). Eine daneben liegende `.daq.cameras.json`-Datei benennt die Kameras, auf die sie sich bezieht, und bei der erneuten Verarbeitung wird sie ausschließlich mit den Bildern dieser Kameras abgeglichen.

### Verarbeitung aufgezeichneter Bilder

Für die Nachbearbeitung nach dem Flug sollten Sie während der Sitzung eine `.daq`-Datei aufzeichnen und diese zusammen mit den Bilddaten aufbewahren – die Pipeline ordnet die zeitlich abgestimmten Downwelling-Daten automatisch zu und ruft bei der ersten Verwendung fehlende Werkskalibrierungen aus der MAPIR
-Cloud ab. GUI-Aufzeichnungen werden dem geöffneten Projekt automatisch hinzugefügt, sobald sie beendet sind.

Die Reflexionsreferenz ist zum Zeitpunkt der Verarbeitung wählbar – „`--reflectance-source`“ auf „`chloros-cli process`“ oder die Einstellung der Reflexionsquelle in den Projekteinstellungen der GUI:

| Wert | Verhalten |
| --- | --- |
| `auto` (Standard) | Ein QA-konformes Kalibrierungsziel innerhalb des Bildausschnitts dient als absolute Referenz; die DAQ-Downwelling-Messung (ρ = π·L/E) wird als Ausweichlösung verwendet. Ein Ziel, das die Qualitätssicherung mit einem Flag besteht (ein Panel ist ausgefallen oder ein Band ist außer Betrieb), wird nicht automatisch bevorzugt: Die Ketten „Ziel“, „Ziel+DAQ“ und „nur DAQ“ werden in der Reihenfolge ihrer gemessenen Punktzahl auf den zurückgehaltenen Panels ausprobiert. |
| `daq` | DAQ-maßgeblich |
| `target` | Strenges In-Frame-Ziel; keine DAQ-Ersetzung |

Siehe [Kalibrierungsziele](../calibration-targets.md) für Ziel-Workflows sowie das [Kapitel „LATTICE“](../lattice/README.md) sowie die [CLI
-Referenz](../reference/cli-reference.md) für die vollständige Verarbeitungspipeline. Verwenden Sie beim Einlesen exportierter Reflexionspixel die angegebene Skalierung (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3
: 65535) – siehe [Ausgabebildformate](../output-image-formats.md).

### Bänder außerhalb des kalibrierten Bereichs des DAQ

Der radiometrisch kalibrierte Bereich des DAQ liegt bei ~374–974 nm. Chloros
lehnt DAQ-basierte Reflexionswerte für alle Kamerabänder ab, bei denen weniger als die Hälfte des spektralen Gewichts innerhalb dieses Bereichs liegt, und meldet dabei den Überspringgrund `dls-uncalibrated-band-<nm>`. Unter den lieferbaren Artikeln betrifft dies nur die F988: Die Reflektanz der F988 wird mithilfe eines Reflektanzpanels vor Ort kalibriert: Da das Band außerhalb des kalibrierten Bereichs des DAQ-Lichtsensors liegt, wendet Chloros
Ihre letzte Panel-Erfassung an und behält diese zwischen den Panel-Messungen bei. Wird eine F988-Kamera ausschließlich im DAQ-Modus betrieben, lehnt Chloros
die DAQ-basierte Reflexion für dieses Band mit dem Überspringgrund `dls-uncalibrated-band-988` ab – der Panel-Workflow ist der unterstützte Weg.

## Zwei Sensoren (Umgebungslicht + Objekt)

Zwei DAQ-Sensoren – beliebige Kombination, unabhängig vom Transportmedium – liefern ein Live-Reflexionsspektrum ohne Kamera: Ein Sensor ist zum Himmel gerichtet (**Umgebungslichtquelle**), der andere zum Objekt (**Objektscanner**), und Chloros
berechnet pro Wellenlänge:

> R(λ) = Objekt(λ) / Umgebungslicht(λ)

(Null, wenn Umgebungslicht ≤ 0).

### In der Benutzeroberfläche

Wenn beide Sensoren auf der Registerkarte „Lichtsensoren“ verbunden sind, öffnen Sie das Overlay zum Hinzufügen von Sensoren (die Schaltfläche „+“ auf einer Diagrammkachel in der Rasteransicht) und wählen Sie **Umgebungslicht + Objekt kombinieren**. Wählen Sie die beiden Sensoren in den Dropdown-Menüs „Umgebungslichtquelle“ und „Objektscanner“ aus und klicken Sie auf „Erstellen“. Die Gruppe erscheint als eigenes Diagramm und als Zeile in der Seitenleiste mit einem grünen**REF**-Symbol.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Unterhalb des Reflexionsdiagramms (Listenansicht) berechnet eine Live-**Vegetationsindex-Tabelle** Indizes aus der Kurve unter Verwendung der Bandmitten bei Blau 450 / Grün 550 / Rot 670 / NIR
800 nm. Verhältnisbasierte Indizes, die die absolute Skala aufheben (NDVI
, GNDVI
, ENDVI, WDRVI
, GRVI
, CVI
, GCI
, MSR) werden immer angezeigt; Indizes, die absolute Reflektanz benötigen (EVI
, SAVI
, OSAVI
, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI
, NLI, MNLI, FCI, GEMI
), erscheinen nur, wenn es sich bei beiden Sensoren um leistungskalibrierte Modelle handelt.

### Scheinbare vs. relative Reflektanz – die Beschriftungsregel

Chloros
kennzeichnet die Ausgabe des Dual-Sensorsystems entsprechend dem, was das Sensorpaar tatsächlich leisten kann:

| Sensorpaar | Bezeichnung |
| --- | --- |
| Beide Sensoren kalibriert – Werkskalibrierungspaket geladen | **Scheinbare Reflektanz** |
| Ein Sensor unkalibriert | **Relative Reflektanz** |

Alle drei Modelle sind radiometrisch: Sobald das werkseitige Kalibrierungsbündel eines Sensors geladen ist, liegen seine Spektren als absolute Werte in W/m²/nm vor, sodass ein Paar kalibrierter Sensoren ein Verhältnis zu einer absoluten scheinbaren Reflektanz ergibt – der Transportfaktor bestimmt diese nicht. Ein Sensor, der weiterhin Rohzählwerte überträgt (kein Paket erreichbar), stuft das Ergebnis auf eine relative Kurve herab (die Spektralform bleibt jedoch gültig). Beide Sensoren sollten korrekt deklarierte Obergrenzen aufweisen ([Obergrenzenprofile &amp; kalibrierter Bereich](caps-and-range.md)).

### Aus Python


In der zusammengefassten SDK
-Oberfläche gibt es keinen speziellen Aufruf für zwei Sensoren: Öffnen Sie zwei Sitzungen mit `chloros_sdk.connect_daq_sensor()` und berechnen Sie das Verhältnis ihrer `latest()`-Spektren selbst, wobei Sie dieselbe Beschriftungskonvention anwenden. (Ein Tool zur Zweisensor-Aufzeichnung existiert auch auf der internen Direkt-Hardware-Oberfläche von MAPIR
und ist der Vollständigkeit halber in der [CLI
-Referenz](../reference/cli-reference.md) aufgeführt – es ist jedoch nicht Teil der ausgelieferten CLI
; der oben beschriebene GUI-Workflow ist der unterstützte Live-Pfad.)
