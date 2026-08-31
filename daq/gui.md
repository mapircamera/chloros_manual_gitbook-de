# Die Registerkarte „DAQ“ in Chloros

Die Registerkarte „DAQ“ – in der Seitenleiste von Chloros als **Lichtsensoren** bezeichnet – ist die Live-Bedienoberfläche für [DAQ-U-, DAQ-M- und DAQ-E-Lichtsensoren](README.md): Schließen Sie Sensoren über beliebige Übertragungsprotokolle an, beobachten Sie kalibrierte Spektren in Echtzeit, berechnen Sie die Live-Reflexionswerte eines Sensorpaares und zeichnen Sie `.daq`-Dateien direkt in Ihr Projekt auf.

Die Registerkarte ist verfügbar, sobald der Start des Chloros-Backends abgeschlossen ist. Die Diagramme auf der Registerkarte werden vom DAQ-Dienst von Chloros über eine Live-Verbindung gespeist, die bei einer Unterbrechung automatisch wiederhergestellt wird (2–10 s Wartezeit); solange der Dienst nicht erreichbar ist, wird in der Statuszeile eines Sensors **Kein Server** angezeigt.

Das Layout besteht aus einer **Sensor-Seitenleiste**(eine Zeile pro verbundenem Sensor) sowie einem**Diagrammbereich** (eine Diagrammkachel pro Sensor oder Gruppe).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Einen Sensor verbinden

Klicken Sie oben in der Seitenleiste auf **Sensor verbinden**. Der Verbindungsdialog öffnet sich im Hauptbereich (oder als Overlay, wenn ein weiterer Sensor hinzugefügt wird – in diesem Fall erscheint eine Schaltfläche „Abbrechen“).

| Steuerung | Verhalten |
| --- | --- |
| **Gerätetyp** | `DAQ-U (USB)` (Standard), `DAQ-M (Bluetooth)` oder `DAQ-E (Ethernet)`. Durch Umschalten wird der Suchvorgang für den neu ausgewählten Transport neu gestartet. |
| **Port / BLE-Gerät / Hostname / IP** | Listet erkannte Geräte als „`device - description`“ auf; der erste als Sensor erkannte Eintrag wird automatisch ausgewählt. Während des Suchvorgangs werden `Scanning...` (USB), `Scanning (N)...` mit einem 8-Sekunden-Countdown (BLE) oder `Discovering ethernet sensors (N)...` mit einem 5-Sekunden-Countdown (Ethernet). Leere Ergebnisse werden als `No ports` / `No BLE devices` / `No ethernet sensors found` angezeigt. |
| **↻ Aktualisieren** | Führt sofort einen erneuten Scan des ausgewählten Transports durch (während eines laufenden BLE-/Ethernet-Scans deaktiviert). |
| **Verbinden** | Wird aktiviert, sobald ein Gerät ausgewählt ist; die Bezeichnung ändert sich zu `Connecting...`, während die Verbindung hergestellt wird. |

Die Erkennung läuft nur **solange der Verbindungsdialog auf dem Bildschirm angezeigt wird** und wird alle 15 Sekunden ausschließlich für den ausgewählten Transport wiederholt – das bloße Öffnen der Registerkarte löst keinen Scan aus. Bei einem Fehler zeigt der Dialog Folgendes an: *„Verbindung fehlgeschlagen. Trennen Sie den Sensor vom Gerät, schließen Sie ihn erneut an und klicken Sie dann erneut auf ‚Verbinden‘.“*

Die Seitenleiste öffnet sich automatisch, sobald sich Ihr erster Sensor verbindet.

{% hint style="info" %}
**DAQ-E wird nicht angezeigt?** Der DAQ-E verfügt über keine Status-LED – überprüfen Sie die PoE-/Verbindungsanzeige am Switch oder Injektor-Port, an den er angeschlossen ist, und warten Sie nach dem Einschalten einige Sekunden, bis er hochgefahren ist. Das Chloros-Gerät muss sich in derselben Broadcast-Domäne befinden (mDNS durchläuft keine Router). Akzeptieren Sie auf dem Windows die Defender-Firewall-Abfrage, sobald das Chloros zum ersten Mal seine Multicast-Sockets bindet (mDNS UDP 5353, DAQ-E-Daten UDP 5002, PTP UDP 319/320) bindet. Zwei DAQ-E-Geräte in einem LAN werden separat erkannt, jedes unter seinem eigenen `daq-e-<id>.local`-Hostnamen.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Gerätetyp bietet DAQ-U (USB), DAQ-M (Bluetooth) und DAQ-E (Ethernet)</figcaption></figure>***

## Die Sensor-Seitenleiste

Jeder angeschlossene Sensor erhält eine Zeile (zuzüglich einer Zeile pro „Ambient+Object“-Gruppe). Die Reihenfolge der Zeilen lässt sich per Drag &amp; Drop ändern, wodurch sich auch die Reihenfolge der Diagrammkacheln ändert. Klicken Sie auf eine Zeile, um diesen Sensor bzw. diese Gruppe im Diagramm der Listenansicht aktiv zu machen.

| Element | Bedeutung |
| --- | --- |
| Farbiger linker Rand | Die Farbe des Sensordiagramms. |
| Transport-Symbol | `DAQ-U` / `DAQ-M` / `DAQ-E` oder ein grünes `REF`-Symbol für eine „Ambient+Object“-Reflexionsgruppe. |
| Gerätename | Standardmäßig die Seriennummer des Sensors (seine feste Kennung für die Kalibrierung, `.daq`-Dateinamen und den Importabgleich); benutzerdefinierte Namen bleiben projektbezogen erhalten. |
| **Kalibriert**-Pille (grün) | Wird angezeigt, wenn das werkseitige Kalibrierungspaket des Sensors geladen ist, d. h. die Spektren sind echte W/m²/nm. |
| **Update verfügbar**-Symbol (bernsteinfarben, nur DAQ-E) | Die aktuell ausgeführte Firmware ist älter als das mit diesem Chloros-Build mitgelieferte Image. Während eines Updates wird der Fortschritt in Echtzeit angezeigt (`Flashing… N%`, `Restarting sensor…`, dann `Updated X → Y` oder `Failed`). |
| Auge | Schaltet die Sichtbarkeit dieses Sensors in seinem Diagramm ein oder aus. |
| Zahnrad | Öffnet das Einstellungsfenster für diesen Sensor (unten). |
| ✕ (rot) | Trennt die Verbindung zum Sensor oder entfernt eine „Ambient+Object“-Gruppe. |

Über den Zeilen befinden sich zwei Schaltflächen:

* **Sensor verbinden** – öffnet den Verbindungsdialog (wird während der Verbindung in „`Connecting...`“ umbenannt).
* ****Alle aufzeichnen / Alle stoppen**— Startet oder stoppt eine `.daq`-Aufzeichnung auf**jedem**angeschlossenen Sensor. Erfordert mindestens einen Sensor**und ein geöffnetes Projekt** (Tooltip: „Projekt öffnen, um aufzuzeichnen“); die Schaltfläche wird rot, solange eine Aufzeichnung läuft.

Im leeren Zustand wird „Keine Sensoren angeschlossen“ angezeigt.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Einstellungen pro Sensor (das Zahnrad-Fenster)

Öffnen Sie es über das Zahnrad-Symbol in einer Sensorzeile. Inhalte in der Reihenfolge:

* **Info-Zeilen** — Gerätetyp (DAQ-U/M/E), Anschluss (`Serial (USB)` / `Bluetooth` / `Ethernet`), Anschluss (COM-Port, BLE-Adresse oder Host) und Seriennummer.
* **Kalibrierungsbericht: Herunterladen** – Ruft das NIST-rückverfolgbare Kalibrierungszertifikat (PDF) dieses Geräts ab und öffnet es in Ihrem PDF-Viewer. Verfügbar, sobald die Seriennummer bekannt ist; das Zertifikat wird bei der ersten Verbindung zwischengespeichert.
* **Gerätename** – Klicken Sie auf den Bleistift, um den Namen zu ändern; wird pro Projekt beibehalten.
* **Farbe der Diagrammlinie** – Farbfeld; wird pro Projekt beibehalten.
* **Integrationszeit (ms)**— Schieberegler + Zahl,**1–500 ms**, Standard**32 ms**. Deaktiviert, solange AE eingeschaltet ist.
* **Frame-Durchschnitt**— Schieberegler + Zahl,**1–50 Frames**, Standard**20**.
* **AE: EIN/AUS**– Umschalter für automatische Belichtung;**standardmäßig EIN** bei Verbindungsaufbau. Schalten Sie ihn aus, um die Belichtungszeit manuell einzustellen.
* **Streaming stoppen / Streaming starten** – den Live-Stream pausieren oder fortsetzen.
* **Aufnahme / Aufnahme beenden** — sensorbezogene `.daq`-Aufnahme (erfordert ein geöffnetes Projekt).
* **Cap** — das Cap-Korrekturprofil (nächster Abschnitt).
* **Live-Info-Zeilen** — Integrationszeit (ms), FPS, Samples, Aufzeichnung (rot `REC` oder `Off`) und Status (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Nur DAQ-E: Zeilen zu Netzwerk, Firmware und PTP

* **Hostname / IP** – die aktuelle Adresse des Geräts.
* **Firmware** – aktuelle Firmware-Version sowie eine Aktionszelle: Eine<version\>

Schaltfläche</version\>

„**Update to \<version\>**“ erscheint</version\>

,<version\>

wenn dieser Chloros-Build ein neueres DAQ-E-Firmware-Image enthält. Das Update wird in etwa 30 Sekunden über das Netzwerk übertragen; der Sensor startet neu und stellt automatisch die Verbindung wieder her, und bei einer unterbrochenen Übertragung bleibt die aktuelle Firmware erhalten. Der Fortschritt wird live angezeigt (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), und die Zelle zeigt „`Up to date`“ an, sobald der aktuelle Stand erreicht ist.
* **PTP-Synchronisation** – der aktuelle PTP-Status (fällt auf „`unknown`“ zurück). Die DAQ-E-Firmware ab Version 1.2.0 nimmt am IEEE 1588 PTPv2 als reine Slave-Uhr teil; das Backend des Hosts „Chloros“ ist der PTP-Grandmaster, und alle DAQ-E- und LATTICE-Kameras im LAN sind in Domäne 0 dessen Slaves, wobei die Zeitstempel innerhalb von etwa 1 ms gehalten werden.

Bei einer „Ambient+Object“-Gruppe zeigt das Zahnrad-Modalfenster nur die Quellsensoren der Gruppe, den Gerätenamen und die Farbe der Diagrammlinie an.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Auswahl der Kappe

Das Dropdown-Menü **Cap** teilt Chloros mit, welche physische Kappe auf dem Diffusor des Sensors angebracht ist, und wendet das werkseitig gemessene Korrekturprofil dieser Kappe auf jedes Spektrum an. Die Auswahlmöglichkeiten hängen vom Modell ab:

| Modell | Kappenauswahl |
| --- | --- |
| DAQ-U | Keine (unverkleideter Sensor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (Kosinus-Korrektor) |
| DAQ-M | Keine (unverkleideter Sensor), Sunshine (Kosinus-Korrektor) |
| DAQ-E | Keine (unverkleideter Sensor), Sichtfeld 15°, Sichtfeld 45°, Sichtfeld 90°, Sunshine (Kosinus-Korrektor) |

**Die Standardeinstellung für jedes Modell ist „Sunshine“ (Kosinus-Korrektor)** — MAPIR liefert jedes DAQ-Gerät mit installierter „Sunshine“-Kappe aus; dies ist die Standardkonfiguration für den Außenbereich: ein 180°-Halbkugelsichtfeld mit einem Kosinusfehler von ≤ ±4 % bis 60° und ≤ ±4,5 % bis 70° (nicht empfohlen bei einem Sonnenhöhenwinkel unter ~15°), konstruktionsbedingt gedämpft (~12×). Ihre Auswahl bleibt im Projekt erhalten.

{% hint style="warning" %}
**Die Auswahl der Kappe muss mit der tatsächlich montierten Kappe übereinstimmen.**Weder der Sensor noch die Software können erkennen, welche Kappe montiert ist. Die Auswahl bestimmt sowohl die Echtzeitkorrektur als auch den in jede `.daq`-Datei geschriebenen Stempel – bei der ~12-fachen Dämpfung der Sunshine-Kappe führt ein nicht deklarierter Kappenwechsel zu einer um etwa diesen Faktor verzerrten Spektralkorrektur. (Das Entfernen und erneute Anbringen derselben Kappe wiederholt sich auf etwa 1,5 %.) Wählen Sie**„Keine“ (unbedeckter Sensor)** nur, wenn die Kappe physisch entfernt wurde; bei einem DAQ-E wird bei der Auswahl „Keine“ weiterhin ein werkseitiges Geometrieprofil für den versenkten Glasdiffusor angewendet – es handelt sich nicht um eine No-Op-Funktion – und ein unbedeckter DAQ-E ist eine Laborkonfiguration, keine unterstützte Feldkonfiguration.
{% endhint %}

{% hint style="info" %}
Aktualisierung gegenüber einem früheren Handbuch: Der browserbasierte Schalter „Sunshine Diffuser Installed“ aus Version 1.1.0 ist nicht mehr vorhanden. Die Handhabung der Abdeckkappe erfolgt nun über dieses sensorspezifische Kappe-Profil, das serverseitig angewendet wird.
{% endhint %}

***

## Der Diagrammbereich

Eine feststehende Leiste am oberen Rand enthält einen **Umschalter zwischen Listen- und Rasteransicht**sowie einen Schieberegler für den**Diagrammzoom** (Kachelgröße 200–2000 px). Die Ansicht wechselt automatisch zur Rasteransicht, wenn mehr als eine Diagrammgruppe vorhanden ist, und zurück zur Listenansicht, wenn nur eine oder keine vorhanden ist. Der Anzeigemodus und die Diagrammgröße bleiben pro Projekt erhalten.

Das **Spektrumdiagramm** für jeden Sensor zeigt:

* **X-Achse** — Wellenlänge (nm). Das Sensorraster umfasst 340–1010 nm in 5-nm-Schritten (135 Punkte) und wird für die Anzeige auf 1 nm interpoliert.
* **Y-Achse** – Leistung (W/m²), mit einem automatischen SI-Präfix (m/µ/n), das anhand des Spitzenwerts ausgewählt wird. Die Spektren sind radiometrisch kalibrierte spektrale Bestrahlungsstärken (W/m²/nm) für alle drei Transportarten.
* Eine regenbogenfarbige Spektralfüllung unter einer einzelnen Kurve; mehrere Sensoren auf einem Diagramm werden als farbige Linien mit abgeblendeter Füllung überlagert.
* **Mauszeiger**— ein vertikaler Cursor mit Wellenlänge und Wert pro Sensor;**Ziehen** zum Zoomen (im gezoomten Zustand erscheint eine Schaltfläche zum Verkleinern).
* Eine **+**-Schaltfläche (nur in der Rasteransicht), um diesem Diagramm einen Sensor hinzuzufügen oder eine Gruppe zu erstellen (siehe unten).
* Der Gerätename ist oben zentriert, und es wird ein Ladekreisel angezeigt, bis der erste Datenpunkt eintrifft.

**Sättigung** wird im Diagramm selbst nicht markiert: Ein gesättigter Sensor zeigt den roten Statustext „`SATURATED`“ und eine rote Zeile „`Saturated: Yes`“ in der Live-Datentabelle an. Verringern Sie die Integrationszeit oder aktivieren Sie AE erneut, um dies zu beheben.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Live-Datentabelle (Listenansicht)

Unterhalb des Diagramms in der Listenansicht, die alle 500 ms aktualisiert wird:

* **Alle Modelle**: Farbfeld (sRGB aus CIE XYZ), gesättigt (Ja/Nein), CIE 1931 X/Y/Z, Farbwert x/y, CIE u′/v′, CCT (K), CRI (Ra), dominante Wellenlänge (nm), Spitzenwellenlänge (nm), Anregungsreinheit, Duv, CIE L\*/a\*/b\* und Munsell H/V/C.
* **Nur kalibrierte Sensoren**(jedes der Modelle DAQ-U / DAQ-M / DAQ-E, sobald das Werkskalibrierungspaket geladen ist – erkennbar am grünen**Kalibriert**-Symbol in der Sensorzeile): Gesamtleistung (W/m²), photopische Lux (lx), skotopische Lux (lx), S/P-Verhältnis, PPFD sowie PPFD Red/Green/Blue (µmol/m²/s), sowie die opischen Bestrahlungsstärken – S-Zapfen, Melanopik, Rhodopik, M-Zapfen, L-Zapfen (alle in W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Reflexionsgruppen (Umgebungslicht + Objekt)

Zwei miteinander verbundene Sensoren lassen sich zu einer Live-Reflexionsanzeige kombinieren – ganz ohne Kamera:

1. Klicken Sie in der Rasteransicht auf **+**in einem Diagrammfeld und wählen Sie**Umgebung + Objekt kombinieren**.
2. Wählen Sie einen **Umgebungslichtquelle**-Sensor und einen**Objektscanner**-Sensor (zwei unterschiedliche Sensoren) aus und klicken Sie dann auf**Erstellen**.

Chloros berechnet R(λ) = Objekt(λ) / Umgebungslicht(λ) pro Wellenlänge aus den beiden Live-Datenströmen (0, wenn Umgebungslicht ≤ 0). Die Bezeichnung der Gruppe richtet sich nach der Kalibrierungsklasse der Sensoren:

* Beide Sensoren kalibriert (Bundle geladen) → **„Scheinbare Reflektanz“**.
* Ein Sensor nicht kalibriert → **„Relative Reflektanz“**.

Die Gruppe erscheint als grüne `REF`-Zeile in der Seitenleiste und in einem eigenen Diagramm (Regenbogenfüllung, Werte bei Mauszeiger auf 4 Dezimalstellen, Zoomen per Ziehen).

Das **+**-Menü bietet außerdem die Option**Neuen Sensor hinzufügen** mit drei Platzierungsmöglichkeiten: *Neuen Sensor kombinieren* (diesem Diagramm hinzufügen), *Vorhandenen Sensor hierher verschieben* oder *Neuen Sensor anzeigen* (eigenes Diagramm).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Vegetationsindex-Tabelle

In der Listenansicht befindet sich unterhalb des Diagramms einer Reflektanzgruppe eine Vegetationsindex-Tabelle, die aus der Live-Reflektanz an den Bandmitten **Blau 450 / Grün 550 / Rot 670 / NIR 800 nm** berechnet wird (Werte auf 4 Dezimalstellen, `---`, wenn nicht berechenbar; für den vollständigen Namen mit der Maus über einen Indexnamen fahren):

* **Immer angezeigt** (skaleninvariant, beliebige Sensorkombination): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Nur wenn beide Sensoren leistungskalibriert sind** (beide Pakete geladen): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

.***

## Aufzeichnen von `.daq`-Dateien

* Für die Aufzeichnung ist ein **geöffnetes Projekt** erforderlich – andernfalls sind sowohl die Schaltfläche „Alle aufzeichnen“ (Seitenleiste) als auch die Aufzeichnungsschaltfläche für die einzelnen Sensoren deaktiviert.
* Die Dateien werden unter **`<project folder>/light_sensor/`** gespeichert; die Dateinamen enthalten die Sensor-ID und einen Zeitstempel, und der Gerätename wird zusammen mit der Aufzeichnung gespeichert.
* Wenn eine Aufzeichnung beendet wird („Stopp“, „Alle stoppen“ oder eine Unterbrechung der Verbindung während der Aufzeichnung), wird die fertige Datei „`.daq`“ **automatisch dem geöffneten Projekt hinzugefügt** — sie erscheint in der Dateiliste des Projekts, ohne dass sie manuell hinzugefügt werden muss, und steht als Daten zur einfallenden Strahlung für die [Reflexionsverarbeitung](README.md) bereit.
* Während der Aufzeichnung wird in den Live-Zeilen des Einstellungsmodals eine rote `REC`-Anzeige angezeigt.

Für quantitative Bestrahlungsstärkewerte sollten Daten von mindestens 15 Sekunden gemittelt werden – dies ist eine Eigenschaft des Geräts und kein Fehler.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Multisensor-Layouts und Projektpersistenz

* Kombinieren Sie mehrere Sensoren in einem Diagramm (gemeinsame Achsen), behalten Sie separate Diagramme bei (automatisches Rasterlayout), verschieben Sie Sensoren zwischen Diagrammen, ordnen Sie Zeilen/Kacheln per Ziehen neu an und blenden Sie einzelne Sensoren mit dem Augen-Schalter aus.
* Pro Projekt bleiben für Chloros folgende Einstellungen erhalten: Gerätenamen, Diagrammfarben, Diagrammgröße, Ansichtsmodus sowie die Einstellungen jedes Sensors (Integrationszeit, Bildmittelung, AE-Status, Obergrenzenauswahl).
* **Beim erneuten Öffnen eines Projekts werden dessen Sensoren automatisch** über die Adresse wieder verbunden – COM-Port für DAQ-U, BLE-Gerät für DAQ-M, mDNS-Hostname bei DAQ-E (wird auch dann aufgelöst, wenn sich die IP-Adresse des Geräts geändert hat) – und wendet das gespeicherte Cap-Profil, die Bildmittelung, den AE-Status und die manuelle Integrationszeit jedes Sensors erneut an.***

## Kamera-Kopplung (DLS)

Es muss nichts gekoppelt werden. Im Gegensatz zu DLS-Workflows bei Drohnen, bei denen ein Lichtsensor von vornherein an eine Kamera gebunden wird, ordnet Chloros die DAQ-Daten nachträglich den Bilddaten zu: Beim Import bzw. bei der Verarbeitung werden die `.daq`-Messwerte auf den Belichtungszeitstempel der jeweiligen Aufnahme interpoliert. Nehmen Sie mit einem beliebigen angeschlossenen Sensor auf (der `.daq` wird automatisch dem Projekt hinzugefügt), und die Reflexionsverarbeitung ermittelt die richtigen Messwerte anhand der Zeit – unter [DAQ-Lichtsensoren](README.md) erfahren Sie, wie die Daten zur nach unten gerichteten Strahlung verwendet werden.</version\>
