# Arbeitsabläufe zur Reflektanzmessung

Ein DAQ-Lichtsensor wandelt radiometrische Bilddaten in Reflektanzwerte um. Es gibt zwei unterschiedliche Arbeitsabläufe:

1. **Einzel-Sensor** – Ein DAQ-Sensor misst die nach unten gerichtete Bestrahlungsstärke, während eine Kamera Aufnahmen macht, und Chloros dividiert die von der Kamera erfasste Strahlungsintensität durch diesen Referenzwert.
2. **Zwei-Sensor-Verfahren** – Zwei DAQ-Sensoren, von denen einer den Himmel und einer ein Objekt beobachtet, erzeugen eine Live-Spektralreflexionskurve, ohne dass eine Kamera beteiligt ist.

## Ein Sensor + Kamera (abwärtsgerichtete Referenz)

Der DAQ fungiert als Sensor für die abwärtsgerichtete Strahlung (DLS): Die Kamera misst die aufsteigende Strahlungsintensität **L**(W/m²/sr/nm), der DAQ misst die abwärtsgerichtete Bestrahlungsstärke**E** (W/m²/nm), und Chloros berechnet die Reflexion pro Band wie folgt:

> ρ = π · L / E

Der DAQ-Messwert ist stets **zeitstempel-abgestimmt auf die Belichtungszeit** – deshalb nutzen das DAQ und die Kameras eine gemeinsame PTP-gesteuerte Uhr (siehe [DAQ-E-Vernetzung &amp; Zeitsynchronisation](ethernet-ptp.md)). Tragen Sie bei Arbeiten im Freien die „Sunshine“-Kosinus-Kappe und geben Sie diese korrekt an; die Angabe der Kappe skaliert E direkt (siehe [Kappenprofile &amp; kalibrierter Bereich](caps-and-range.md)). Beachten Sie bei quantitativen Messungen die Gerätekennlinie: Die quantitative Bestrahlungsstärke ergibt sich aus dem Mittelwert von Messwerten über mindestens 15 Sekunden.

### Live-Erfassung

Verbinden Sie das DAQ auf der Registerkarte „Kameras“ mit einer Kamera: Das Einstellungsfeld jeder Kamera verfügt über ein Dropdown-Menü **Lichtsensor**, in dem alle angeschlossenen DAQs (DAQ-U/M/E) aus der Registerkarte „Lichtsensoren“ aufgelistet sind; bei einem synchronisierten Array wird die Auswahl des Lichtsensors für das gesamte Array auf jedes einzelne Element übertragen (einzelne Kameras können diese Einstellung jedoch weiterhin überschreiben). Nach der Verknüpfung werden die Spektren des Sensors in den DLS-Slot der Kamera eingespeist, und die exportierten Reflexionswerte werden durch den entsprechenden Messwert dividiert.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Zwei wichtige Verhaltensweisen, die Sie kennen sollten:

* **Kein DAQ gebunden → Reflektanz wird abgelehnt, nicht simuliert.** Chloros lehnt das Reflektanzprodukt ab und protokolliert den Grund für den Übersprung, anstatt stillschweigend ein minderwertiges Produkt zurückzugeben.
* **Der verwendete Messwert bleibt erhalten.** Für jedes Reflektanzbild wird der tatsächlich verwendete DAQ-Messwert als `.daq`-Sidecar neben dem Bild gespeichert, sodass die Aufnahme später erneut verarbeitet werden kann ([Aufzeichnung und das .daq-Format](recording.md)).

### Verarbeitung aufgezeichneter Bilddaten

Für die Nachbearbeitung sollten Sie während der Sitzung einen `.daq` aufzeichnen und diesen zusammen mit den Bilddaten aufbewahren – die Pipeline ermittelt automatisch die zeitlich abgestimmte Downwelling-Daten, und ruft bei der ersten Verwendung fehlende Werkskalibrierungen aus der Cloud von MAPIR ab. GUI-Aufzeichnungen werden nach Beendigung automatisch dem geöffneten Projekt hinzugefügt.

Die Reflexionsreferenz ist zum Zeitpunkt der Verarbeitung wählbar – `--reflectance-source` auf `chloros-cli process` oder die Einstellung der Reflexionsquelle in den Projekteinstellungen der GUI:

| Wert | Verhalten |
| --- | --- |
| `auto` (Standard) | Ein QA-konformes Kalibrierungsziel innerhalb des Bildausschnitts dient als absolute Referenz; die DAQ-Downwelling-Werte (ρ = π·L/E) werden als Ausweichwert verwendet |
| `daq` | DAQ-maßgeblich |
| `target` | Striktes Ziel innerhalb des Bildausschnitts; keine DAQ-Ersetzung |

Siehe [Kalibrierungsziele](../calibration-targets.md) für Ziel-Workflows und das [LATTICE-Kapitel](../lattice/README.md) sowie die [CLI-Referenz](../reference/cli-reference.md) für die vollständige Verarbeitungspipeline. Verwenden Sie beim Einlesen exportierter Reflexionspixel die angegebene Skalierung (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) – siehe [Ausgabebildformate](../output-image-formats.md).

### Bänder außerhalb des kalibrierten Bereichs des DAQ

Der radiometrisch kalibrierte Bereich des DAQ liegt bei ~374–974 nm. Chloros lehnt die DAQ-basierte Reflexionsmessung für jedes Kameraband ab, bei dem weniger als die Hälfte des spektralen Gewichts innerhalb dieses Bereichs liegt, und meldet den Überspringungsgrund `dls-uncalibrated-band-<nm>`. Unter den lieferbaren Artikelnummern betrifft dies nur die F988: Die Reflexionsmessung der F988 wird mithilfe einer Reflexionsplatte vor Ort kalibriert: Da das Band außerhalb des kalibrierten Bereichs des DAQ-Lichtsensors liegt, wendet Chloros Ihre letzte Erfassung der Reflexionsplatte an und behält diese zwischen den Messungen bei. Wird eine F988-Kamera ausschließlich im DAQ-Modus betrieben, lehnt Chloros die DAQ-basierte Reflektanz für dieses Band mit dem Überspringgrund `dls-uncalibrated-band-988` ab – der Panel-Workflow ist der unterstützte Weg.

## Zwei Sensoren (Umgebungslicht + Objekt)

Zwei DAQ-Sensoren – beliebige Kombination, unabhängig vom Transportmedium – liefern ein Live-Reflexionsspektrum ohne Kamera: Ein Sensor ist zum Himmel gerichtet (**Umgebungslichtquelle**), einer zum Objekt (**Objektscanner**), und Chloros berechnet pro Wellenlänge:

> R(λ) = Objekt(λ) / Umgebungslicht(λ)

(Null, wenn Umgebungslicht ≤ 0).

### In der Benutzeroberfläche

Wenn beide Sensoren auf der Registerkarte „Lichtsensoren“ verbunden sind, öffnen Sie das Überlagerungsfenster zum Hinzufügen eines Sensors (die Schaltfläche „+“ auf einer Diagrammkachel in der Rasteransicht) und wählen Sie **Umgebungslicht + Objekt kombinieren**. Wählen Sie die beiden Sensoren in den Dropdown-Menüs „Umgebungslichtquelle“ und „Objektscanner“ aus und klicken Sie auf „Erstellen“. Die Gruppe wird als eigenes Diagramm und als Zeile in der Seitenleiste mit einem grünen**REF**-Symbol angezeigt.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Unterhalb des Reflexionsdiagramms (Listenansicht) berechnet eine Live-**Vegetationsindex-Tabelle** Indizes aus der Kurve unter Verwendung der Bandmitten bei Blau 450 / Grün 550 / Rot 670 / NIR 800 nm. Verhältnisbasierte Indizes, die die absolute Skala aufheben (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) werden immer angezeigt; Indizes, für die die absolute Reflexion benötigt wird (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) erscheinen nur, wenn es sich bei beiden Sensoren um stromkalibrierte Modelle handelt.

### Scheinbar vs. relativ – die Benennungsregel

Chloros kennzeichnet die Ausgabe des Dual-Sensors mit dem, was das Sensorpaar tatsächlich leisten kann:

| Sensorpaar | Bezeichnung |
| --- | --- |
| Beide Sensoren kalibriert – Werkskalibrierung geladen | **Scheinbare Reflektanz** |
| Ein Sensor unkalibriert | **Relative Reflektanz** |

Alle drei Modelle sind radiometrisch: Sobald das Werkskalibrierungspaket eines Sensors geladen ist, sind seine Spektren absolute Werte in W/m²/nm, sodass ein Paar kalibrierter Sensoren ein Verhältnis zu einer absoluten scheinbaren Reflektanz ergibt – der Transport entscheidet dies nicht. Ein Sensor, der weiterhin Rohzählwerte überträgt (kein Paket erreichbar), stuft das Ergebnis auf eine relative Kurve herab (die Spektralform bleibt jedoch gültig). Beide Sensoren sollten korrekt deklarierte Obergrenzen aufweisen ([Obergrenzenprofile &amp; kalibrierter Bereich](caps-and-range.md)).

### Aus Python

In der gepoolten Oberfläche SDK gibt es keinen speziellen Dual-Sensor-Aufruf: Öffnen Sie zwei Sitzungen mit `chloros_sdk.connect_daq_sensor()` und berechnen Sie das Verhältnis ihrer `latest()`-Spektren selbst, wobei Sie dieselbe Beschriftungskonvention anwenden. (Ein Tool zur Dual-Sensor-Aufzeichnung ist auch auf der internen, direkt an die Hardware gebundenen Oberfläche von MAPIR vorhanden und in der [CLI-Referenz](../reference/cli-reference.md) aufgeführt – es ist jedoch nicht Bestandteil des ausgelieferten CLI; der oben beschriebene GUI-Workflow ist der unterstützte Live-Pfad.)
