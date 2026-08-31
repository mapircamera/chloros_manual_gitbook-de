# Mehrkamera-Arrays

Ein LATTICE-**Array**besteht aus zwei oder mehr LATTICE-Kameras, die als eine synchronisierte Einheit verbunden sind. Eine Kamera fungiert als**Master**: Sie sendet einen Hardware-GPIO-Triggerimpuls über eine gemeinsame Synchronisationsleitung (standardmäßig**Line2**), sodass alle Kameras zum selben Zeitpunkt eine Aufnahme erstellen. Chloros bietet zusätzlich PTP-Zeitsynchronisation, eine Live-Vorschau (Kacheln pro Kamera oder ein einzelnes, ausgerichtetes Multiband-Komposit) sowie synchronisierte Aufnahmen – jeder Aufnahmedurchgang erzeugt eine**Bildgruppe**, in der alle Kameras denselben Zeitstempel und dieselbe Bild-ID (in der Aufnahmeausgabe als `fid:N` angegeben).

Mithilfe von Arrays erzeugen Mono-Kameras (M3M) Vegetationsindizes – eine Kamera liefert ein Band, und das Array richtet diese zu einem Multiband-Stack aus. Siehe [Mono-Kameras &amp; Vegetationsindizes](mono-indices.md).

Es gibt drei gleichwertige Möglichkeiten, ein Array zu verbinden, und bei allen wird derselbe „Smart-Prep“-Ablauf ausgeführt:

| Oberfläche | Einstiegspunkt |
| --- | --- |
| GUI | Registerkarte „Kameras“ → **Array verbinden** (blaue Schaltfläche) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (erste Seriennummer = Master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (erste Seriennummer = Master) |

Smart-prep führt nacheinander folgende Schritte durch: eine Netzwerkfähigkeitsprüfung (ICMP-DF-Ping + GVSP-Prüfung), die Auswahl der Synchronisationsstufe, die automatische Verkleinerung der Bildgröße zur Anpassung an die Übertragungsleitung, die Aktivierung von PTP, die automatische Auswahl des Pixelformats pro Kamera, die automatische Belichtungsvoreinstellung anhand des gespeicherten Zustands jeder Kamera sowie die GPIO-Triggerkonfiguration auf Line2.

{% hint style="info" %}
Die Kameras müssen über die Verbindung erreichbar sein, bevor dies funktioniert – siehe [Kameras anschließen](connecting.md) für die Erkennung, Adressierung und den Kalibrierungs-Download bei der ersten Verbindung. Bei Mehrkamera-Setups sind die Einstellungen des Empfangsrings der Host-Netzwerkkarte ebenso wichtig wie die Verbindungsgeschwindigkeit; die vollständige Tabelle mit Symptomen und Lösungen findet sich im [CLI-Handbuch § Einrichtung und Optimierung der Host-Netzwerkkarte](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Der Dialog „Array Connect“

Auf der Registerkarte „Kameras“ → **„Array verbinden“**wird ein dreistufiger Assistent geöffnet:**Auswählen → Anzeigemodus → Einstellungen**.

### Schritt 1 – Master und Slaves

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

auswählen Der Dialog durchsuchst das Netzwerk, sobald er geöffnet wird („Netzwerk wird gescannt...“), und prüft anschließend die GPIO-Trigger-Verkabelung („GPIO-Verkabelung wird geprüft...“). Sie benötigen mindestens **2 Kameras**, um ein Array aufzubauen.

Die Verkabelungsprüfung füllt die Rollenauswahl nach Möglichkeit automatisch aus und zeigt eine der folgenden drei Meldungen an:

| Meldung | Bedeutung |
| --- | --- |
| „GPIO-Master erkannt – Auswahl bereits ausgefüllt“ (grün) | Die Überprüfung hat die Trigger-Topologie gefunden; die Kontrollkästchen für Master und Slave sind bereits markiert. |
| „Kein Master erkannt – GPIO-Kabel prüfen“ (orange) | Keine Kamera hat einen Triggerimpuls empfangen; überprüfen Sie die Synchronisationsverkabelung. Sie können die Rollen weiterhin manuell auswählen. |
| „Kein Synchronisationskabel: {serials}“ (orange) | An den aufgeführten Kameras ist kein Synchronisationskabel angeschlossen. |

Die Kameratabelle enthält die Spalten **Kamera / Seriell / IP / Master (Funk) / Slave (Kontrollkästchen)**:

* Wählen Sie genau **einen Master**und**einen oder mehrere Slaves** aus. Durch erneutes Klicken auf das Funkfeld des aktuellen Masters wird die Auswahl aufgehoben.
* Eine Kamera mit der Kennzeichnung **„Kein Synchronisationskabel“** kann niemals als Slave ausgewählt werden – ein Slave ohne Triggerverkabelung würde endlos auf die Synchronisationsleitung warten und ein leeres Bild liefern. Schließen Sie diese Kamera stattdessen als eigenständige Kamera an.
* Kameras, die bereits eigenständig angeschlossen sind, werden *nicht* deaktiviert: Durch den Array-Anschluss wird die eigenständige Sitzung beendet und die Kamera innerhalb des Arrays erneut geöffnet.

**Weiter: Anzeigemodus →**wird aktiviert, sobald ein Master und mindestens ein Slave ausgewählt wurden.**Neu scannen** führt die Erkennung und die Verdrahtungsprüfung erneut durch.

{% hint style="warning" %}
**Abbrechen** ist deaktiviert, solange ein Scan oder eine Verkabelungsprüfung läuft – ein Abbruch während der Verkabelungsprüfung kann bei der LATTICE-Kamera-Firmware zum Absturz der Kamera SDK führen. Warten Sie, bis der Ladekreis vollständig geladen ist.
{% endhint %}

### Schritt 2 – Anzeigemodus

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Modus | Was Sie erhalten |
| --- | --- |
| **Separate Kameras** | Ein Live-Kachel pro Kamera, alle werden gemeinsam ausgelöst, sodass die Bilder synchron bleiben. Jede Kamera behält ihre eigene Farbe und ihre eigenen Einstellungen bei. |
| **Kombinierte Kameras** *(Standard)* | Ein einzelnes Kachelbild, das die ausgerichtete Multiband-NDVI/index-Komposition darstellt. Die Kameras teilen sich die Array-Farbe. |

Der Anzeigemodus verändert lediglich die Darstellung der Live-Vorschau – das Aufnahmeverhalten ist in beiden Fällen identisch.

### Schritt 3 – Array-Einstellungen und das projizierte Ergebnis

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Beim Aufrufen dieser Szene fragt Chloros das Backend nach einer **Empfehlung**und wendet automatisch eine Kombination aus ROI und Binning an, die zum Empfangsring Ihres NIC passt (es wird Binning gegenüber ROI-Zuschneiden bevorzugt, da beim Binning das gesamte Sichtfeld erhalten bleibt). Bei jeder Änderung wird die Analyse live neu durchgeführt und der rechte Bereich**Prognostiziertes Ergebnis** aktualisiert.

Linke Spalte – Einstellungen:

| Steuerung | Auswahlmöglichkeiten | Standard | Anmerkungen |
| --- | --- | --- | --- |
| **ROI (Sichtfeld)** | Voll (2048×1536) / Halb (1024×768) / Viertel (512×384) | Voll | Sensor-Ausschnitt: Halb-/Viertel-Ausschnitt auf einen kleineren Bereich bei nativem Pixelabstand. |
| **Binning** | 1× / 2× (Summe 2×2) / 4× (Summe 4×4) | 1× | Hardware-Binning: 2×2 = volles Sichtfeld bei einem Viertel der Leitungs-Kosten; 4×4 = volles Sichtfeld bei 1/16. Ausgeblendet, wenn die Kameras kein Binning unterstützen. |
| **Leitungsseitiges Bild** (Auslesung) | — | — | Die nach dem Binning tatsächlich über die Leitung gesendete Breite × Höhe, auf Vielfache von 16 (mindestens 64). |
| **Pin-Auflösung**| Kontrollkästchen | aus | Chloros erhöht das Binning normalerweise automatisch beim Verbindungsaufbau, wenn die prognostizierte Bildrate unter**1,5 fps** fällt. Durch das Festlegen der Pin-Einstellung wird die von Ihnen gewählte Bildgröße beibehalten und die niedrigere Bildrate akzeptiert – wodurch eine überbelegte Konfiguration zu einer harten Verbindungsablehnung statt zu einer automatischen Herabstufung führt. |
| **Triggerrate** | 0,5–60 fps, Schritt 0,1 | leer = auto | Die Triggerrate des Masters. Leer lassen, damit Chloros sie ableiten kann. |
| **Wire-Budget**| 20–2000 MB/s, Schrittweite 10 | leer = auto | Wie viel der Host tatsächlich aufnehmen kann, in MB/s –**die einzige Zahl, von der die gesamte Array-Zuweisung abhängt.** Wird vom Netzwerkadapter automatisch erkannt. Verringern Sie diesen Wert, wenn das Array beschädigte Frames meldet: Der erkannte Wert überschätzt die Leistung von USB-Adaptern und gemeinsam genutzten Switches. Durch Ändern dieses Werts wird die Projektion live neu berechnet. |

Rechte Spalte – **Prognostiziertes Ergebnis**:

* **Synchronisationsstufe** — „Simultane Aufnahme“ (grün), „Simultane Aufnahme (FTD-versetzte Ausgabe)“ (grün), „Versetzte Aufnahme (100 ms Drift)“ (gelb) oder „Konfiguration zu groß“ (rot).
* **fps-Prognose** — wird als Bereich angezeigt („dunkel → hell“), da die Bildrate eines synchronisierten Arrays durch die Belichtungszeit der langsamsten Kamera begrenzt wird.
* **NIC-Leitung** — Verbindungsgeschwindigkeit und Dauerdurchsatz („NIC {mbps} Mbps · Dauer {N} MB/s“).
* **Sim-Emit-Burst-Prüfung** — Kann der Ring des Host-NICs einen gleichzeitigen Burst von allen Kameras aufnehmen („Sim-Emit-Burst: X MB · NIC-Ring nutzbar: Y MB ✓/✗“)?
* **Überprüfung des Leitungsbudgets** — aggregierter Bedarf im stationären Zustand im Vergleich zur kollisionssicheren Leitungsobergrenze („Leitungsbudget: {Bedarf} MB/s, gefordert von {n} Kameras · Obergrenze {Obergrenze} MB/s ✓/✗ überbelegt“).
* **„Max. Kameras auf dieser Leitung: {n} – festgelegt durch die Mindestbandbreite pro Kamera, sodass das Binning diesen Wert nicht erhöht.“** – wird angezeigt, wenn du dich der Obergrenze für die Kameranzahl näherst (oder diese überschreitest).
* **„Bei diesen Einstellungen werden FRAMES VERLOREN.“**– rote Warnung mit der Begründung des Backends sowie einer Liste der Blocker und blauen**Behebungsvorschlägen** („Um dieses Array ins Netzwerk zu integrieren“ / „Um die gleichzeitige Aufnahme freizuschalten“).**„Anwenden &amp; Verbinden“** ist gesperrt, bis eine Prognose vorliegt, und die Beschriftung des Schalters gibt an, warum die Aktion abgelehnt wird:

| Schaltflächenbeschriftung | Bedeutung | Was tatsächlich hilft |
| --- | --- | --- |
| „Wird analysiert...“ | Analyse noch im Gange. | Warten. |
| **„Zu viele Kameras für dieses Netzwerk“**| Das Array überlastet die Leitung (Aggregatprüfung fehlgeschlagen). | Weniger Kameras, Jumbo-Frames durchgehend oder eine schnellere Netzwerkkarte.**Ein kleinerer ROI hilft NICHT** – siehe unten. |
| **„ROI reduzieren, um zu aktivieren“** | Bei diesen Einstellungen würden Frames verloren gehen (Burst-/Ring-Prüfung fehlgeschlagen). | ROI reduzieren, Binning erhöhen oder den Empfangsring der Netzwerkkarte reparieren. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Während des Verbindungsaufbaus kann ein grünes **Kalibrierungs-Download-Fenster** mit einem seriennummernspezifischen Fortschrittsbalken angezeigt: Wenn eine Kamera zum ersten Mal an einen Rechner angeschlossen wird, lädt Chloros ihr ca. 3,8 MB großes Werkskalibrierungspaket über GigE von der Kamera herunter (ca. 70 Sekunden pro Kamera). Bei zwischengespeicherten Kameras wird dieses Fenster nie angezeigt. Siehe [Kameras anschließen](connecting.md).

## Bandbreite: Wie viele Kameras passen?

Was ein Array übertragen kann, hängt von der Leitung ab, nicht von Chloros; daher finden sich die Planungswerte im Hardware-Handbuch: **[Planung der Array-Bandbreite](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Was Chloros damit macht: Der Verbindungsdialog führt einen Netzwerk-Test durch, prognostiziert die erreichbare Bildrate und wählt eine passende Stufe aus. Wenn das Array die Leitung überlastet, lehnt es die Verbindung ab, anstatt Pakete stillschweigend zu verwerfen – siehe das oben beschriebene Fenster mit den prognostizierten Ergebnissen.

## Wenn Frames fehlen

Eine Kamera kann aus zwei völlig unterschiedlichen Gründen in einer veröffentlichten Gruppe fehlen,
und diese erfordern gegensätzliche Lösungen. Chloros zählt sie separat, anstatt eine
„unvollständige“ Zahl zu melden, die keinen der beiden Gründe benennt:

| Was ist passiert? | Was bedeutet das? | Wo muss nachgesehen werden? |
| --- | --- | --- |
| **Beschädigt**– das Frame ist angekommen, war aber strukturell fehlerhaft | GVSP-Paketverlust auf dem Netzwerkpfad | Das**Leitungsbudget**, der NIC-Empfangsring, Jumbo-Frames, der Switch |
| **Nie angekommen**– es kam überhaupt kein Frame | Die Kamera hat nicht ausgelöst oder es wurde nichts gesendet | Das**M8-Synchronisationskabel**, die Synchronisationsleitung, ob alle Teilnehmer scharfgeschaltet sind |

Die Aufteilung wird alle 10 Sekunden neu bewertet, während das Array streamt. Oberhalb von 5 % wird dies
mit Angabe beider Zahlen protokolliert, und jeder beschädigte Puffer wird beim ersten Auftreten
pro Kamera gemeldet, anschließend einmal pro Minute zusammengefasst, damit eine lange Sitzung lesbar bleibt.

**Beschädigte Frames mit dem Wert „0“ bei „Nie angekommen“ bedeuten, dass Auslösung und Kabelsynchronisation einwandfrei sind**und jeder verlorene Frame auf dem Netzwerkpfad liegt. Die Lösung besteht darin, das**Wire Budget** zu senken und
die Verbindung neu herzustellen.

{% hint style="warning" %}
**Eine Verringerung der Triggerrate hilft nicht bei fehlerhaften Bildern.** Das Paket-Pacing der Kamera
wird einmalig beim Verbindungsaufbau festgelegt. Eine Verringerung der Triggerrate verändert, wie oft ein Burst
auftritt, nicht, wie schnell der Burst selbst über die Leitung übertragen wird. Bei einem gemessenen System mit 4 Kameras
fünfmalige Verringerung der Triggerrate änderte nichts, während die Senkung des Wire Budgets von 240 auf
200 MB/s bei derselben Anlage den Anteil fehlerhafter Frames von 10,4 % auf null senkte.
{% endhint %}

Ein laufendes Array kann sich nicht neu planen – trennen Sie die Verbindung und stellen Sie sie erneut her, damit der „Connect-Time“-Picker
mit dem neuen Budget arbeiten kann.

### USB-Netzwerkadapter sind auf 200 MB/s begrenzt

Ein USB-Ethernet-Adapter gibt zwar seine *Ethernet*-Verbindungsrate an, doch was er tatsächlich
aufrechterhalten kann, wird durch den USB-Bus und seinen Treiber begrenzt. Einem USB-10-GbE-Dongle wurden früher
etwa 1000 MB/s Durchsatz zugeschrieben – ein Wert, der noch nie gemessen worden war –, und die
von vier Kameras auf diesen imaginären Spielraum führte zu einer Verfälschung von 6–18 % der Bilder, während das Array
weiterhin eine einwandfreie Zielbildrate meldete. USB-Adapter sind nun auf
**200 MB/s** begrenzt. Die Obergrenze ist ein absoluter Wert und kein Prozentsatz, da der Begrenzer der
Bus ist: Ein USB-1-GbE-Adapter erreicht etwa 80 MB/s und ist davon nicht betroffen.

Wenn Ihr Host tatsächlich schneller ist als die Obergrenze, erhöhen Sie das **Wire Budget** entsprechend.

## PTP-Zeitsynchronisation

Die *Synchronisation* der Frames erfolgt über den Hardware-Trigger; **PTP** (IEEE 1588 PTPv2) sorgt für vergleichbare *Zeitstempel* auf allen Geräten. Es ist standardmäßig beim Verbinden des Arrays aktiviert:

* Das **Chloros-Host-Backend läuft als PTP-Grandmaster**. LATTICE-Kameras und DAQ-E-Lichtsensoren sind in Domäne 0 diesem untergeordnet, sodass Bildzeitstempel und DAQ-Spektren auf einen Takt (~1 ms) abgestimmt sind.
* `--no-ptp` (CLI) deaktiviert dies für Laborarbeiten – Zeitstempel verschiedener Kameras sind dann **nicht** vergleichbar.
* Überprüfen Sie den Synchronisationsstatus mit dem CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Die Registerkarte „Kameras“ selbst verfügt über keine PTP-Anzeige; dort werden pro Kamera die synchronen Werte der schreibgeschützten Felder **Rolle**(Master/Slave),**Synchronisationsleitung** sowie die Leistungsstufe des Arrays angezeigt. Der DAQ-E-PTP-Status wird in den Sensordetails auf der Registerkarte „Lichtsensoren“ angezeigt.

## Die Live-Array-Ansicht

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

Der Hauptbereich bietet zwei Layouts (Umschaltung in der oberen Leiste): **Rasteransicht**(jede Kachel ist eine Zelle; durch Ziehen neu anordnen, wenn das Raster-Schloss entsperrt ist) und**Listenansicht**(Arrays in voller Breite oben, eine aktive Kamera darunter). Der Schieberegler**Feed-Zoom** passt die Größe der Kacheln an; bei einer Zellenbreite unter 200 px werden die Namens- und fps-Einblendungen automatisch ausgeblendet.

Im **Einzelmodus** wird eine Kachel pro Kamera angezeigt. Jede Kachel enthält folgende Einblendungen:

* den Kameranamen (oben links),
* eine **FPS-Anzeige** (oben rechts) – dies ist die vom Backend gemeldete *tatsächliche Aufnahmerate* der Kamera, nicht die Abtastrate der Vorschau (die Live-Vorschau ist unabhängig von der Aufnahmerate auf 30 fps begrenzt),
* einen Statuspunkt – grün (Streaming) / gelb (Ladevorgang) / rot (Fehler),
* einen **Spinner für veraltete Frames**, wenn seit 2 s kein neues Bild eingetroffen ist – normal für ca. 5 s nach jeder Verbindungsherstellung/Trennung, während das Backend die Bandbreitenverteilung zwischen den Kameras neu ausbalanciert.

Im **kombinierten Modus**wird eine einzige zusammengesetzte Kachel angezeigt: Das Backend führt Debayering, Skalierung, Ausrichtung und Rauschunterdrückung durch, wandelt die Daten in bandweise Strahlungsintensität um (zuzüglich DLS-Reflexionsgrad, wenn ein Lichtsensor gebunden ist), wertet den Indexausdruck des Arrays aus, wendet die LUT an und streamt das Ergebnis als MJPEG. Bis das erste ausgerichtete Bild gerendert wird, zeigt die Kachel ihren Status an: „Array wird vorbereitet…“, „Ausrichtung wird kalibriert…“, „Warten auf erstes Bild…“ oder – falls das Zeitlimit für automatische Neuausrichtungsversuche (~30 s) abgelaufen ist – „Ausrichtung erforderlich“ mit einer Schaltfläche**Ausrichtung kalibrieren**.

Nützliche Informationen zum Kombinationsmodus:

* Das Composite wird auf das Bild der **Master**-Kamera ausgerichtet. Die AE-ROI-Ausrichtung und die Spotmessung auf dem Composite sind für die Master-Kamera exakt und für die Slave-Kameras annähernd; verwenden Sie**Split View** (Array-Einstellungen → „Mitglieds-Kameras anzeigen“), um pixelgenaue Kacheln pro Kamera zu erhalten, ohne zusätzliche Kameraverbindungen zu öffnen.
* Mit **Ebenen anzeigen**(Array-Einstellungen; standardmäßig deaktiviert) können Sie eine Vordergrund- und eine Hintergrundebene auswählen – jede Mitgliedskamera oder**Index**. Bei Vordergrund = Index zeigen Pixel außerhalb des LUT-Min/Max-Bereichs die Hintergrundebene an.
* **Render-Auflösung** (Standard 720p) legt die Höhe des Live-Streams *und* die Exportgröße des gespeicherten Composites fest. Bilder der einzelnen Kameras werden immer in voller Auflösung exportiert.
* Die Ausrichtung wird pro Sitzung berechnet und niemals gespeichert – siehe den Abschnitt „Ausrichtung“ im Einstellungsfenster des Arrays für RMS-Residuen und die Schaltfläche „Neukalibrieren“.

## Aufzeichnung: Überwachung vs. Analyse

Die Array-Aufnahmeflächen lassen sich klar in **Überwachungsqualität**(Aufzeichnung des Sichtbaren) und**Analysequalität** (Aufzeichnung von Rohdaten, spätere Kalibrierung) unterteilen:

| Workflow | Qualität | Was wird gespeichert? | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Aufnahme**(Standbilder) | Analyse | Eine synchronisierte Bildgruppe pro Durchgang; kameraspezifische Dateien auf jeder ausgewählten Exportebene (Rohdaten/Debayering/Radiance/Reflexionsgrad/Vorschau/Index) + `.daq`-Sidecar | Schaltfläche**Alles aufnehmen** + Aufnahmeeinstellungen | `lattice array-capture` |
| **Indexvideo aufzeichnen** | Überwachung | Die angezeigte Live-Kompositansicht des kombinierten Indexes – 8-Bit, Vorschauauflösung, LUT eingebrannt; erfordert einen geöffneten Live-Stream | ● Indexvideo aufzeichnen (kombinierte Arrays) | `lattice array-record` |
| **Rohdaten-Burst → Video erstellen**| Analyse | Rohdaten-Sensorbilder bei voller Erfassungsrate + Manifest + `.daq`, anschließend Offline-Rekonstruktion zu kalibriertem Strahlungs-/Reflexions-/Index-Video, zeitlich abgeglichen mit den DAQ-Messwerten | ⦿ Rohdaten-Burst aufzeichnen →**Video erstellen** | `lattice array-burst` → `lattice array-build-video` |

Faustregel: Wenn die Pixel *Messwerte* liefern sollen, verwende „Aufnahme“ oder „Burst“ (Analysenqualität); wenn Sie lediglich *beobachten oder demonstrieren* möchten, was das Array erfasst hat, nehmen Sie das Index-Video auf (für Überwachungszwecke).

### Aufnahmeeinstellungen (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

Das Zahnrad-Symbol neben **„Capture All“** öffnet den Bereich „Aufnahmeeinstellungen“ (erfordert ein geöffnetes Projekt – Aufnahmen werden darin gespeichert):

* **Aufnahmemodus**:**Einzelaufnahme**(ein Durchgang) /**Kontinuierlich**(aufeinanderfolgend; begrenzt durch eine Aufnahmemenge, Standard 1, oder eine Dauer, Standard 10 s) /**Intervall** (Zeitraffer: N Aufnahmen alle X Intervalle für insgesamt Y, Standard 1 alle 5 s für 1 Minute).
* **Exporttypen pro Kamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index – alle zutreffenden Optionen sind standardmäßig aktiviert. Radiance/Reflectance sind bei Kameras vom Typ RGB-Filter ausgeblendet;**Reflectance wird nur angezeigt, wenn die Kamera über einen DAQ-Lichtsensor verfügt** (eigener oder vom Array geerbter); „Index“ erfordert einen konfigurierten Indexausdruck.
* **Ausgerichtet**(pro Array, standardmäßig**aktiviert**): Passen die Exporte der Elemente an das Ausrichtungsprofil des Arrays an, sodass die Exporte pixelgenau registriert sind. „Raw“ bleibt immer unverzerrt, enthält die Transformation jedoch in den Metadaten.
* **Schnellste Erfassung** (Umschaltfunktion): Nur Rohdaten + der zugewiesene DAQ-Messwert + die freie kombinierte Index-Zusammensetzung; dabei wird die Kalibrierungsberechnung zum Zeitpunkt der Erfassung übersprungen, um die maximale Rate zu erreichen – Radianz/Reflexionsgrad/Index werden später aus dem gespeicherten `.daq` neu berechnet.
* Auswahlen bleiben im Projekt erhalten. Ausgeblendete oder angehaltene Kameras werden übersprungen.

Das Äquivalent CLI (derselbe Backend-Endpunkt, dieselbe Semantik):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

Die Komprimierung für Aufnahmen mit TIFF erfolgt mit `deflate` (verlustfrei, Standard) oder `none` — vollständige Flag-Tabellen, die Struktur des Erfassungsordners und die Regeln für die Nachbearbeitung finden Sie in der [CLI-Referenz](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Koppeln eines DAQ-Lichtsensors

Für reflektanz- und beleuchtungskorrigierte Vorschauen werden Daten zum nach unten gerichteten Licht von einem DAQ-Sensor benötigt (angeschlossen auf der Registerkarte **Lichtsensoren**):

* Die **Array-Zeile**in der Seitenleiste zeigt eine**„DAQ · ein/aus“-Schaltfläche** an – *ein*, wenn ein Lichtsensor auf Array-Ebene eingestellt ist **oder** eine der zugehörigen Kameras über einen eigenen verfügt; der Tooltip listet genau auf, welcher Sensor welche Kamera versorgt.
* Die Zuweisung für das gesamte Array erfolgt in den Array-Einstellungen → **Umgebungslichtsensor**→ Dropdown-Menü**Lichtsensor**. Die Auswahl bleibt projektweit erhalten, wird auf jede einzelne Kamera im Array übertragen, und einzelne Kameras können sie weiterhin durch ihren eigenen Sensor überschreiben.
* Die Statuszeile darunter zeigt den aktuellen Status an: **Aus**→ „Warten auf erstes Spektrum …“ →**„Aktiv – alle Kameras im Array sind beleuchtungskorrigiert“** → oder, falls in den letzten 3 s kein neues Spektrum eingegangen ist, ein Hinweis auf veraltete Daten – der letzte Messwert wird weiterhin verwendet (Messwerte verfallen im Aufnahmepfad nie).

Bei zugewiesenem Sensor: Der Exporttyp „Reflexionsgrad“ wird verfügbar, Live-Vorschauen sind belichtungskorrigiert, die prädiktive automatische Belichtung kann das Spektrum nutzen, und bei jeder Reflexionsaufnahme wird der tatsächlich verwendete DAQ-Messwert als **`.daq`-Sidecar** neben dem Bild gespeichert, sodass die Aufnahme später erneut verarbeitet werden kann.

## `array-connect` CLI-Optionen

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Alle LATTICE-Kameras automatisch erkennen (erfordert ≥2) | **Die erste serielle Nummer ist der MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-Synchronisationsleitung. |
| `--target-fps F` | auto | Auslösefrequenz des Masters. |
| `--binning {1,2,4}` | auto | Hardware-Binning. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Experten-Übersteuerung des Sync-Tier-Pickers. |
| `--wire-ceiling-mbps MB_PER_S` | automatisch erkannt | Host-Wire-Budget in MB/s – die CLI-Form des Feldes **Wire-Budget**. Verringern Sie diesen Wert, wenn das Array beschädigte Frames meldet. Wird mit dem Projekt gespeichert, sodass es bei einer späteren Wiederverbindung wiederhergestellt wird. |
| `--no-recommend` | aus | Den Schritt der Netzwerkanalyse überspringen. |
| `--no-ptp` | aus | PTP deaktivieren (Zeitstempel verschiedener Kameras sind dann nicht vergleichbar). |

`lattice array-list`, `array-status` und `array-disconnect` verwalten die persistente Sitzung. Die vollständige Referenz zu den Unterbefehlen, einschließlich der Ausrichtung (`align-calibrate` / `align-apply`) und der Netzwerk-Tools, befindet sich im [CLI-Referenz § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); die SDK-Entsprechungen (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) befinden sich in der [SDK-Referenz](../reference/sdk-reference.md)enthalten. Ab Python beträgt das Leitungsbudget `connect_array(..., wire_ceiling_mbps=120)`, und die Aufteilung in „live“, „beschädigt“ und „nie angekommen“ erfolgt auf [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
