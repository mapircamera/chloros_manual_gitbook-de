# Die Registerkarte „DAQ“ in Chloros

Die Registerkarte „DAQ“ – in der Chloros-Seitenleiste als **Lichtsensoren** bezeichnet – ist die Live-Bedienoberfläche für [DAQ-U-, DAQ-M- und DAQ-E-Lichtsensoren](README.md): Schließen Sie Sensoren über einen beliebigen Transport an, beobachten Sie kalibrierte Spektren in Echtzeit, berechnen Sie die Live-Reflexion eines Sensorpaares und zeichnen Sie `.daq`-Dateien direkt in Ihr Projekt auf.

Die Registerkarte wird verfügbar, sobald das Chloros-Backend vollständig gestartet ist. Die Diagramme der Registerkarte werden vom DAQ-Dienst von Chloros über eine Live-Verbindung gespeist, die bei einer Unterbrechung automatisch wiederhergestellt wird (Wartezeit von 2–10 s); solange der Dienst nicht erreichbar ist, wird in der Statuszeile eines Sensors **Kein Server** angezeigt.

Das Layout besteht aus einer **Sensor-Seitenleiste**(eine Zeile pro verbundenem Sensor) sowie einem**Diagrammbereich** (eine Diagrammkachel pro Sensor oder Gruppe).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Einen Sensor verbinden

Klicken Sie oben in der Seitenleiste auf **Sensor verbinden**. Der Verbindungsdialog öffnet sich im Hauptbereich (oder als Overlay, wenn Sie einen weiteren Sensor hinzufügen – in diesem Fall erscheint eine Schaltfläche „Abbrechen“).

| Steuerung | Verhalten |
| --- | --- |
| **Gerätetyp** | `DAQ-U (USB)` (Standard), `DAQ-M (Bluetooth)` oder `DAQ-E (Ethernet)`. Durch Umschalten wird die Suche nach dem neu ausgewählten Transport erneut gestartet. |
| **Port / BLE-Gerät / Hostname / IP** | Listet erkannte Geräte als „`device - description`“ auf; der erste als Sensor erkannte Eintrag wird automatisch ausgewählt. Während des Suchvorgangs werden „`Scanning...`“ (USB), „`Scanning (N)...`“ mit einem 8-Sekunden-Countdown (BLE) oder „`Discovering ethernet sensors (N)...`“ mit einem 5-Sekunden-Countdown (Ethernet). Leere Ergebnisse werden als `No ports` / `No BLE devices` / `No ethernet sensors found` angezeigt. |
| **↻ Aktualisieren** | Führt sofort einen erneuten Scan des ausgewählten Übertragungsprotokolls durch (deaktiviert während eines laufenden BLE-/Ethernet-Scans). |
| **Sensor verbinden** | Wird aktiviert, sobald ein Gerät ausgewählt ist; die Bezeichnung ändert sich während des Verbindungsaufbaus zu `Connecting...`. |

Die Erkennung läuft nur **solange der Verbindungsdialog auf dem Bildschirm angezeigt wird** und wiederholt sich alle 15 Sekunden ausschließlich für den ausgewählten Transport – das bloße Öffnen der Registerkarte löst keinen Scan aus. Bei einem Fehler zeigt der Dialog Folgendes an: *„Verbindung fehlgeschlagen. Ziehen Sie den Sensor ab und schließen Sie ihn erneut an, dann klicken Sie erneut auf ‚Verbinden‘.“*

Die Seitenleiste öffnet sich automatisch, sobald sich Ihr erster Sensor verbindet.

{% hint style="info" %}
**DAQ-E wird nicht angezeigt?**Das DAQ-E verfügt über keine Status-LED – überprüfen Sie die PoE-/Link-Anzeige am Switch oder Injektor-Port, an den es angeschlossen ist, und warten Sie nach dem Einschalten einige Sekunden, bis es hochgefahren ist. Der Chloros-Rechner muss sich in derselben Broadcast-Domäne befinden (mDNS durchquert keine Router). Die Firewall-Regel**„Chloros-Backend (UDP-In)“** des Installationsprogramms lässt mDNS (UDP 5353), DAQ-E-Daten (UDP 5002) und PTP (UDP 319/320) durch; bei einer Firewall eines Drittanbieters müssen dieselben Ports freigegeben sein. Zwei DAQ-E-Geräte in einem LAN werden separat erkannt, jedes unter seinem eigenen `daq-e-<id>.local`-Hostnamen.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Als Gerätetyp stehen DAQ-U (USB), DAQ-M (Bluetooth) und DAQ-E (Ethernet) zur Auswahl</figcaption></figure>

***

## Die Sensor-Seitenleiste

Jeder angeschlossene Sensor erhält eine Zeile (zuzüglich einer Zeile pro „Ambient+Object“-Gruppe). Die Reihen lassen sich per Drag & Drop neu anordnen, wobei sich durch die Änderung der Reihenfolge auch die Anordnung der Diagrammkacheln ändert. Klicken Sie auf eine Zeile, um diesen Sensor bzw. diese Gruppe in der Listenansicht zum aktiven Diagramm zu machen.

| Element | Bedeutung |
| --- | --- |
| Farbiger linker Rand | Die Farbe des Sensordiagramms. |
| Transport-Symbol | `DAQ-U` / `DAQ-M` / `DAQ-E` oder ein grünes `REF`-Symbol für eine „Ambient+Object“-Reflexionsgruppe. |
| Gerätename | Standardmäßig die Seriennummer des Sensors (seine feste Identität für die Kalibrierung, `.daq`-Dateinamen und den Importabgleich); benutzerdefinierte Namen bleiben projektbezogen erhalten. |
| **Kalibriert**-Pille (grün) | Wird angezeigt, wenn das werkseitige Kalibrierungspaket des Sensors geladen ist, d. h., die Spektren sind echte W/m²/nm. |
| **Update verfügbar**-Pille (bernsteinfarben, nur DAQ-E) | Die aktuell ausgeführte Firmware ist älter als das mit diesem Chloros-Build mitgelieferte Image. Während eines Updates wird der Fortschritt in Echtzeit angezeigt (`Flashing… N%`, `Restarting sensor…`, dann `Updated X → Y` oder `Failed`). |
| Auge | Schaltet die Sichtbarkeit dieses Sensors in seinem Diagramm um. |
| Zahnrad | Öffnet das Einstellungsfenster für den jeweiligen Sensor (unten). |
| ✕ (rot) | Trennt die Verbindung zum Sensor oder entfernt eine „Ambient+Object“-Gruppe. |

Über den Zeilen befinden sich zwei Schaltflächen:

* **Sensor verbinden** — öffnet den Verbindungsdialog (wird während der Ausführung in „`Connecting...`“ umbenannt).
* **Alle aufzeichnen / Alle stoppen**— startet oder stoppt eine `.daq`-Aufzeichnung an**jedem**angeschlossenen Sensor. Erfordert mindestens einen Sensor**und ein geöffnetes Projekt** (Tooltip: „Projekt zum Aufzeichnen öffnen“); die Schaltfläche wird rot, solange eine Aufzeichnung läuft.

Im leeren Zustand lautet die Anzeige „Keine Sensoren verbunden“.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Einstellungen pro Sensor (das Zahnrad-Modalfenster)

Öffnen Sie das Menü über das Zahnrad-Symbol in einer Sensorzeile. Inhalt in der Reihenfolge:

* **Info-Zeilen** — Gerätetyp (DAQ-U/M/E), Verbindung (`Serial (USB)` / `Bluetooth` / `Ethernet`), Anschluss (COM-Port, BLE-Adresse oder Host) und Seriennummer.
* **Kalibrierungsbericht: Herunterladen** – Ruft das NIST-rückverfolgbare Kalibrierungszertifikat (PDF) dieses Geräts ab und öffnet es in Ihrem PDF-Viewer. Verfügbar, sobald die Seriennummer bekannt ist; das Zertifikat wird bei der ersten Verbindung zwischengespeichert.
* **Gerätename** — Klicken Sie auf den Bleistift, um den Namen zu ändern; wird pro Projekt beibehalten.
* **Farbe der Diagrammlinie** — Farbfeld; wird pro Projekt beibehalten.
* **Integrationszeit (ms)**— Schieberegler + Zahl,**1–500 ms**, Standard**32 ms**. Deaktiviert, solange AE eingeschaltet ist.
* **Bilddurchschnitt**— Schieberegler + Zahl,**1–50 Bilder**, Standard**20**.
* **AE: EIN/AUS**– Umschalter für die automatische Belichtung;**standardmäßig EIN** bei der Verbindung. Schalten Sie ihn aus, um die Belichtungszeit manuell einzustellen.
* **Streaming stoppen / Streaming starten** – den Live-Stream anhalten oder fortsetzen.
* **Aufnahme / Aufnahme beenden** — sensorbezogene `.daq`-Aufnahme (erfordert ein geöffnetes Projekt).
* **Cap** — das Cap-Korrekturprofil (nächster Abschnitt).
* **Live-Info-Zeilen** — Integrationszeit (ms), FPS, Samples, Aufnahme (rot `REC` oder `Off`) und Status (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Nur DAQ-E: Zeilen zu Netzwerk, Firmware und PTP

* **Hostname / IP** – die aktuelle Adresse des Geräts.
* **Firmware** – aktuelle Firmware-Version sowie eine Aktionszelle: Eine<version\>

Schaltfläche</version\>

„**Auf \** aktualisieren“<version\>

wird angezeigt</version\>,<version\>

wenn dieser Chloros-Build ein neueres DAQ-E-Firmware-Image enthält. Das Update wird in etwa 30 Sekunden über das Netzwerk übertragen; der Sensor startet neu und stellt automatisch die Verbindung wieder her, wobei bei einer Unterbrechung der Übertragung die aktuelle Firmware erhalten bleibt. Der Fortschritt wird live übertragen (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), und die Zelle zeigt „`Up to date`“ an, sobald der aktuelle Stand erreicht ist.
* **PTP-Synchronisation** — der aktuelle PTP-Status (fällt auf `unknown` zurück). Die DAQ-E-Firmware ab Version 1.2.0 nimmt am IEEE 1588 PTPv2 als reine Slave-Uhr teil; das Backend des Chloros-Hosts ist der PTP-Grandmaster, und jede DAQ-E- sowie LATTICE-Kamera im LAN ist in Domäne 0 dessen Slave, wobei die Zeitstempel innerhalb von etwa 1 ms gehalten werden.
* **Netzwerk**– die Adresse, unter der das Gerät läuft, und wie es diese erhalten hat (DHCP, Link-Local oder statisch), sowie eine gespeicherte Einstellung, die nach einem Neustart gilt.**Ändern…**öffnet ein Formular:**Automatisch (DHCP)**oder**Statische IP**mit einer IP-Adresse, einer Subnetzmaske und einem optionalen Gateway.**Übernehmen und neu starten** speichert die Einstellungen, startet den Sensor neu und stellt die Verbindung wieder her. Eine statische Adresse außerhalb des Netzwerks dieses Computers wird abgelehnt, da Chloros den Sensor nach dem Neustart nicht erreichen könnte; es erscheint dann ein Kontrollkästchen, um dies dennoch zuzulassen – für einen Computer, der neu adressiert oder in dieses Netzwerk weitergeleitet wird. **Stream beim Einschalten**legt fest, ob der Sensor sofort nach dem Einschalten mit dem Streaming beginnt, noch bevor eine Software eine Verbindung herstellt.**Sensor neu starten** startet den Sensor neu und stellt die Verbindung mit den aktuellen Einstellungen wieder her. Ein Neustart wird abgelehnt, solange der Sensor aufzeichnet. Neu in 1.3.0.

Für eine „Ambient+Object“-Gruppe zeigt das Zahnrad-Modalfenster nur die Quellsensoren der Gruppe, den Gerätenamen und die Farbe der Diagrammlinie an.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Auswahl der Kappe

Das Dropdown-Menü **Kappe** teilt Chloros mit, welche physische Kappe auf dem Diffusor des Sensors angebracht ist, und wendet das werkseitig gemessene Korrekturprofil dieser Kappe auf jedes Spektrum an. Die Auswahlmöglichkeiten hängen vom Modell ab:

| Modell | Kappenauswahl |
| --- | --- |
| DAQ-U | Keine (unverkleideter Sensor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (Kosinus-Korrektor) |
| DAQ-M | Keine (unverkleideter Sensor), Sunshine (Kosinus-Korrektor) |
| DAQ-E | Keine (unverkleideter Sensor), Sichtfeld 15°, Sichtfeld 45°, Sichtfeld 90°, Sunshine (Kosinus-Korrektor) |

**Die Standardeinstellung für jedes Modell ist „Sunshine“ (Kosinus-Korrektor)** — MAPIR liefert jedes DAQ-Gerät mit aufgesetzter „Sunshine“-Kappe aus; dies ist die Standardkonfiguration für den Außenbereich: ein 180°-Halbkugelsichtfeld mit einem Kosinusfehler von ≤ ±4 % bis 60° und ≤ ±4,5 % bis 70° (nicht empfohlen bei einem Sonnenhöhenwinkel unter ~15°), konstruktionsbedingt gedämpft (~12×). Ihre Auswahl bleibt im Projekt erhalten.

{% hint style="warning" %}
**Die Auswahl der Kappe muss mit der tatsächlich verwendeten Kappe übereinstimmen.**Weder der Sensor noch die Software können erkennen, welche Kappe angebracht ist. Die Auswahl bestimmt sowohl die Live-Korrektur als auch den in jede `.daq`-Datei geschriebenen Stempel – bei einer ~12-fachen Dämpfung durch die „Sunshine“-Kappe führt ein nicht deklarierter Kappenwechsel zu einer um etwa diesen Faktor verzerrten Spektralkorrektur. (Das Entfernen und erneute Anbringen derselben Kappe wiederholt sich auf etwa 1,5 %.) Wählen Sie**„Keine“ (unverkleideter Sensor)** nur, wenn die Kappe physisch entfernt wurde; bei einem DAQ-E wird bei „None“ weiterhin ein werkseitiges Geometrieprofil für den versenkten Glasdiffusor angewendet – es handelt sich nicht um eine No-Op-Funktion – und ein ungeschützter DAQ-E ist eine Laborkonfiguration, keine unterstützte Feldkonfiguration.
{% endhint %}

{% hint style="info" %}
Aktualisierung gegenüber einem früheren Handbuch: Der browserbasierte Schalter „Sunshine Diffuser Installed“ aus Version 1.1.0 ist nicht mehr vorhanden. Die Handhabung der Abdeckung erfolgt nun über dieses sensorspezifische Abdeckungsprofil, das serverseitig angewendet wird.
{% endhint %}

***

## Der Diagrammbereich

Eine feststehende Leiste am oberen Rand enthält einen **Umschalter zwischen Listen- und Rasteransicht**sowie einen Schieberegler für den**Diagrammzoom** (Kachelgröße 200–2000 px). Die Ansicht wechselt automatisch zur Rasteransicht, wenn mehr als eine Diagrammgruppe vorhanden ist, und zurück zur Listenansicht, wenn nur eine oder keine vorhanden ist. Anzeigemodus und Diagrammgröße bleiben pro Projekt erhalten.

Das **Spektrumdiagramm** für jeden Sensor zeigt:

* **X-Achse** — Wellenlänge (nm). Das Sensorraster reicht von 340–1010 nm in 5-nm-Schritten (135 Punkte) und wird für die Anzeige auf 1 nm interpoliert.
* **Y-Achse**– bei einem kalibrierten Sensor (grüne**Kalibriert**-Pille),**Leistung (W/m²/nm)**– radiometrisch kalibrierte spektrale Bestrahlungsstärke für alle drei Transportarten – mit einem automatischen SI-Präfix (m/µ/n), das anhand des Spitzenwerts ausgewählt wird. Ein Sensor, dessen Kalibrierungspaket nicht geladen ist, stellt seine Rohmesswerte als**Intensität (Zählwerte)** dar, ohne Skalierung. Wenn ein Diagramm beide Arten überlagert, wird die Leistung auf der linken Achse und die Zählwerte auf der rechten Achse dargestellt.
* Eine spektrale Regenbogenfüllung unter einer einzelnen Kurve; mehrere Sensoren in einem Diagramm werden als farbige Linien mit abgeblendeten Füllungen überlagert.
* **Mauszeiger**– ein vertikaler Cursor mit Wellenlänge und Wert pro Sensor (W/m²/nm für einen kalibrierten Sensor, ansonsten Gesamtzählwerte);**Ziehen** zum Zoomen (im gezoomten Zustand erscheint eine Schaltfläche zum Verkleinern).
* Eine **+**-Schaltfläche (nur in der Rasteransicht), um diesem Diagramm einen Sensor hinzuzufügen oder eine Gruppe zu erstellen (siehe unten).
* Der Gerätename ist oben zentriert, und es wird ein Ladekreisel angezeigt, bis das erste Datenbild eintrifft.

**Sättigung** wird im Diagramm selbst nicht angezeigt: Ein gesättigter Sensor zeigt den roten Statustext „`SATURATED`“ und eine rote Zeile „`Saturated: Yes`“ in der Live-Datentabelle an. Verringern Sie die Integrationszeit oder aktivieren Sie AE erneut, um den Zustand zu beheben.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Live-Datentabelle (Listenansicht)

Unterhalb des Diagramms in der Listenansicht, die alle 500 ms aktualisiert wird:

* **Alle Modelle**: Lichtfarbfeld (sRGB aus CIE XYZ), gesättigt (Ja/Nein), CIE 1931 X/Y/Z, Farbwert x/y, CIE u′/v′, CCT (K), CRI (Ra), dominante Wellenlänge (nm), Spitzenwellenlänge (nm), Anregungsreinheit, Duv, CIE L\*/a\*/b\* und Munsell H/V/C.
* **Nur kalibrierte Sensoren**(jedes der Modelle DAQ-U / DAQ-M / DAQ-E, sobald das Werkskalibrierungspaket geladen ist – das grüne**Kalibriert**-Symbol in der Sensorzeile ist das Erkennungsmerkmal): Gesamtleistung (W/m²), photopische Lux (lx), skotopische Lux (lx), S/P-Verhältnis, PPFD sowie PPFD Red /Green /Blue (µmol/m²/s) und die opischen Bestrahlungsstärken – S-Zapfen, Melanopisch, Rhodopisch, M-Zapfen, L-Zapfen (alle in W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Reflektanzgruppen (Umgebungslicht + Objekt)

Zwei miteinander verbundene Sensoren können zu einer Live-Reflexionsanzeige kombiniert werden – ganz ohne Kamera:

1. Klicken Sie in der Rasteransicht auf **+**in einer Diagrammkachel und wählen Sie**Umgebungslicht + Objekt kombinieren**.
2. Wählen Sie einen **Umgebungslichtquelle**-Sensor und einen**Objektscanner**-Sensor (zwei unterschiedliche Sensoren) aus und klicken Sie dann auf**Erstellen**.

Chloros berechnet R(λ) = Objekt(λ) / Umgebung(λ) pro Wellenlänge aus den beiden Live-Streams (0, wenn Umgebung ≤ 0). Die Bezeichnung der Gruppe richtet sich nach der Kalibrierungsklasse der Sensoren:

* Beide Sensoren kalibriert (Bundle geladen) → **„Scheinbare Reflektanz“**.
* Ein Sensor nicht kalibriert → **„Relative Reflektanz“**.

Die Gruppe erscheint als grüne `REF`-Zeile in der Seitenleiste und in einem eigenen Diagramm (Regenbogenfüllung, Werte bei Mauszeiger auf 4 Dezimalstellen, Zoomen per Ziehen).

Das **+**-Menü bietet außerdem die Option**Neuen Sensor hinzufügen** mit drei Platzierungsmöglichkeiten: *Neuen Sensor kombinieren* (diesem Diagramm hinzufügen), *Vorhandenen Sensor hierher verschieben* oder *Neuen Sensor anzeigen* (eigenes Diagramm).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Vegetationsindex-Tabelle

In der Listenansicht befindet sich unterhalb des Diagramms einer Reflektanzgruppe eine Vegetationsindex-Tabelle, die aus der Live-Reflektanz an den Bandmitten **Blau 450 / Grün 550 / Rot 670 / NIR 800 nm** (Werte auf 4 Dezimalstellen, `---`, wenn nicht berechenbar; mit der Maus über einen Indexnamen fahren, um dessen vollständigen Namen anzuzeigen):

* **Immer angezeigt** (skaleninvariant, beliebige Sensorkombination): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Nur wenn beide Sensoren leistungskalibriert sind** (beide Bundles geladen): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Aufzeichnung von `.daq`-Dateien

* Für die Aufzeichnung ist ein **geöffnetes Projekt** erforderlich – andernfalls sind sowohl „Record All“ (Seitenleiste) als auch die Aufzeichnungsschaltfläche für die einzelnen Sensoren deaktiviert.
* Die Dateien werden unter **`<project folder>/light_sensor/`** gespeichert; die Dateinamen enthalten die Sensor-ID und einen Zeitstempel, und der Gerätename wird zusammen mit der Aufzeichnung gespeichert.
* Wenn eine Aufzeichnung beendet wird (über „Stopp“, „Alle stoppen“ oder eine Unterbrechung der Verbindung während der Aufzeichnung), wird die fertige `.daq`-Datei **automatisch dem geöffneten Projekt hinzugefügt** — es erscheint in der Dateiliste des Projekts, ohne dass es manuell hinzugefügt werden muss, und steht als Daten zur einfallenden Strahlung für die [Reflexionsverarbeitung](README.md) bereit.
* Während der Aufzeichnung wird in den Live-Zeilen des Einstellungsmodals ein roter `REC`-Indikator angezeigt.

Für quantitative Bestrahlungsstärkewerte sollten Daten von mindestens 15 Sekunden gemittelt werden – dies ist eine Geräteeigenschaft und kein Fehler.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Multisensor-Layouts und Projektpersistenz

* Kombinieren Sie mehrere Sensoren in einem Diagramm (gemeinsame Achsen), behalten Sie separate Diagramme bei (automatisches Rasterlayout), verschieben Sie Sensoren zwischen Diagrammen, ordnen Sie Zeilen/Kacheln per Ziehen neu an und blenden Sie einzelne Sensoren mit dem Augensymbol aus.
* Pro Projekt speichert Chloros folgende Daten: Gerätenamen, Diagrammfarben, Diagrammgröße, Ansichtsmodus sowie die Einstellungen jedes Sensors (Integrationszeit, Bildmittelung, AE-Status, Kapselauswahl).
* **Beim erneuten Öffnen eines Projekts werden die Sensoren automatisch wieder verbunden** – über die Adresse: COM-Port für DAQ-U, BLE-Gerät für DAQ-M, mDNS-Hostname für DAQ-E (wird auch dann aufgelöst, wenn sich die IP-Adresse des Geräts geändert hat) – und wendet das gespeicherte Cap-Profil, die Bildmittelung, den AE-Status und die manuelle Integrationszeit jedes Sensors erneut an.***

## Kamera-Kopplung (DLS)

Es muss nichts gekoppelt werden. Im Gegensatz zu DLS-Workflows bei Drohnen, bei denen ein Lichtsensor von vornherein an eine Kamera gebunden wird, ordnet Chloros die DAQ-Daten nachträglich den Bildern zu: Beim Import bzw. bei der Verarbeitung werden die `.daq`-Messwerte auf den Belichtungszeitstempel der jeweiligen Aufnahme interpoliert. Nehmen Sie mit einem beliebigen angeschlossenen Sensor auf (der `.daq` wird automatisch dem Projekt hinzugefügt), und die Reflexionsverarbeitung ermittelt die richtigen Messwerte anhand des Zeitstempels – unter [DAQ-Lichtsensoren](README.md) erfahren Sie, wie die Daten zur Sonneneinstrahlung verwendet werden. Für **Live**-LATTICE-Aufnahmen wählen Sie stattdessen den Sensor im Dropdown-Menü**Lichtsensor** auf der Registerkarte „Kameras“ aus; jede Aufnahme speichert dann den verwendeten Messwert, der ausschließlich auf die Kameras dieser Aufnahme abgestimmt ist (siehe [Reflexions-Workflows](reflectance.md)).</version\>
