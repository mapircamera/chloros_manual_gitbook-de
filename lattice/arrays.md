# Mehrkamera-Arrays

Ein LATTICE-**Array**besteht aus zwei oder mehr LATTICE-Kameras, die zu einer synchronisierten Einheit verbunden sind. Eine Kamera fungiert als**Master**: Sie sendet einen Hardware-GPIO-Triggerimpuls über eine gemeinsame Synchronisationsleitung (standardmäßig**Line2**), sodass alle Kameras im selben Moment belichten. Chloros bietet zusätzlich PTP-Zeitsynchronisation, eine Live-Vorschau (Kamera-Kacheln oder ein einzelnes, ausgerichtetes Multiband-Komposit) sowie synchronisierte Aufnahmen – jeder Aufnahmedurchlauf erzeugt eine **Bildgruppe**, in der alle Kameras denselben Zeitstempel und dieselbe Bild-ID (in der Aufnahmeausgabe als `fid:N` angegeben).

Mithilfe von Arrays erzeugen Mono-Kameras (M3M) Vegetationsindizes – eine Kamera liefert ein Band, und das Array ordnet diese zu einem Multiband-Stack an. Siehe [Mono-Kameras & Vegetationsindizes](mono-indices.md).

Es gibt drei gleichwertige Möglichkeiten, ein Array zu verbinden, und bei allen wird derselbe „smart-prep“-Ablauf ausgeführt:

| Oberfläche | Einstiegspunkt |
| --- | --- |
| GUI | Registerkarte „Kameras“ → **Array verbinden** (blaue Schaltfläche) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (erste Seriennummer = Master) |
| Python-SDK | `connect_array(serials=[…])` → `ArraySession` (erste Seriennummer = Master) |

„Smart-Prep“ führt Folgendes durch: eine Netzwerkfähigkeitsprüfung (ICMP-DF-Ping + GVSP-Prüfung), die Auswahl der Synchronisationsstufe, eine Bildratenprognose bei der angeforderten Bildgröße, die Entscheidung für den gepackten 12-Bit-Transport (siehe [Gepackter 12-Bit-Transport](#packed-12-bit-transport)), PTP-Aktivierung, automatische Auswahl des Pixelformats pro Kamera, automatische Belichtungsvoreinstellung anhand des gespeicherten Zustands jeder Kamera sowie GPIO-Trigger-Konfiguration auf Line2.

**Standardmäßig voller Sensor (neu in 1.3.0).** Connect behält die von Ihnen angeforderte Bildgröße bei – den vollen 2048×1536-Sensor, kein Binning und kein Zuschneiden, sofern Sie nichts anderes wählen — und passt die Bildrate an die vom Host und Netzwerk bereitgestellte Rate an. Es wird niemals eigenständig gebinnt oder beschnitten (in Version 1.2.0 wurde das Binning automatisch unterhalb von 1,5 fps aktiviert). Ob Binning oder ein kleinerer ROI, bleibt dir überlassen: Wähle im „Connect“-Dialog eine Option aus, und unter „Projected Outcome“ wird die daraus resultierende Bildrate angezeigt; `array-connect` gibt die schnelleren Optionen mit ihren prognostizierten Bildraten aus.

{% hint style="info" %}
Die Kameras müssen über die Verbindung erreichbar sein, bevor all dies funktioniert – siehe [Kameras verbinden](connecting.md) zur Erkennung, Adressierung und zum Herunterladen der Kalibrierungsdaten bei der ersten Verbindung. Bei Mehrkamera-Setups sind die Einstellungen des Empfangsrings der Host-Netzwerkkarte ebenso wichtig wie die Verbindungsgeschwindigkeit; die vollständige Tabelle mit Symptomen und Lösungen finden Sie im Abschnitt [CLI-Referenz § Einrichtung und Optimierung der Host-Netzwerkkarte](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Der Dialog „Array verbinden“

Die Registerkarte „Kameras“ → **Array verbinden**öffnet einen dreistufigen Assistenten:**Auswählen → Anzeigemodus → Einstellungen**.

### Schritt 1 – Master und Slaves auswählen



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->

Der Dialog durchsuchst das Netzwerk, sobald er geöffnet wird („Netzwerk wird gescannt...“), und prüft anschließend die Synchronisationskabel („Synchronisationskabel werden geprüft...“), um festzustellen, welche Kameras an der M8-Synchronisationsleitung angeschlossen sind. Sie benötigen mindestens **2 Kameras**, um ein Array aufzubauen.

Die M8-Synchronisationskabel laufen an einem Knotenpunkt zusammen, sodass **jede verkabelte Kamera als Master fungieren kann**. Die Überprüfung füllt die Rollen vorab aus – die RGB-Kamera (`-FRGB`) als Master, wenn sie verkabelt ist, andernfalls die erste verkabelte Kamera und die anderen verkabelten Kameras als Slaves – und zeigt eines von drei Bannern an:

| Banner | Bedeutung |
| --- | --- |
| „Synchronisationskabel an {n} Kameras gefunden. Jede davon kann Master sein; die RGB-Kamera (ansonsten die erste) ist vorausgewählt.“ (grün) | Die Sonde hat die Kameras an der Synchronisationsleitung gefunden; die Kontrollkästchen für Master und Slave sind bereits angekreuzt. |
| „Kein Master erkannt – GPIO-Kabel prüfen“ (orange) | Keine Kamera hat einen Triggerimpuls empfangen; überprüfen Sie die Synchronisationsverkabelung. Die Rollen sind weiterhin voreingestellt (die RGB-Kamera, andernfalls die erste Kamera in der Liste, als Master) und können geändert werden. |
| „Kein Sync-Kabel: {serials}“ (orange) | An die aufgelisteten Kameras ist kein Sync-Kabel angeschlossen. |

Die Kameratabelle enthält die Spalten **Kamera / Seriell / IP / Master (Funk) / Slave (Kontrollkästchen)**. Die Zeilen sind in**Kamerareihenfolge** aufgelistet – M3C-Kameras (Bayer) vor M3M-Kameras (Mono); unter den M3C-Kameras folgen FRGB, FRGN, FOCN und dann FNGB; M3M-Kameras nach Filterwellenlänge, beginnend mit der niedrigsten (F450 vor F808). Die Kameraliste in der Seitenleiste und das Standardlayout des Rasters verwenden dieselbe Reihenfolge.

* Wählen Sie genau **einen Master**und**einen oder mehrere Slaves** aus. Ein erneuter Klick auf das Auswahlfeld des aktuellen Masters hebt die Auswahl auf. Die Auswahl eines anderen Masters erfolgt mit einem Klick: Der neue Master verlässt die Slaves und der vorherige Master wird zum Slave (es sei denn, er verfügt über kein Synchronisationskabel).
* Eine Kamera mit der Kennzeichnung **„Kein Synchronisationskabel“** kann niemals als Slave ausgewählt werden – ein Slave ohne Auslöseverkabelung würde ewig auf die Synchronisationsleitung warten und ein leeres Bild liefern. Schließen Sie diese Kamera stattdessen als eigenständige Kamera an.
* Kameras, die bereits als Einzelkameras angeschlossen sind, werden *nicht* deaktiviert: Durch den Array-Anschluss wird die Einzelkamera-Sitzung beendet und die Kamera innerhalb des Arrays erneut geöffnet.

**Weiter: Anzeigemodus →**wird aktiviert, sobald ein Master und mindestens ein Slave ausgewählt wurden.**Neu scannen** führt die Erkennung und die Verkabelungsprüfung erneut durch.

{% hint style="warning" %}
**Abbrechen** ist deaktiviert, solange ein Scan oder eine Verkabelungsprüfung läuft – ein Abbruch während der Verkabelungsprüfung kann das Kamera-SDK auf der LATTICE-Kamera-Firmware zum Absturz bringen. Warten Sie, bis der Ladekreis vollständig geladen ist.
{% endhint %}

### Schritt 2 – Anzeigemodus

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->



| Modus | Was Sie erhalten |
| --- | --- |
| **Separate Kameras** | Eine Live-Kachel pro Kamera, die alle gemeinsam ausgelöst werden, sodass die Bilder synchron bleiben. Jede Kamera behält ihre eigene Farbe und ihre eigenen Einstellungen bei. |
| **Kombinierte Kameras** *(Standard)* | Ein einzelnes Kachelbild, das die ausgerichtete Multiband-NDVI-/Index-Komposition darstellt. Die Kameras teilen sich die Array-Farbe. |

Der Anzeigemodus verändert lediglich die Darstellung der Live-Vorschau – das Aufnahmeverhalten ist in beiden Fällen identisch.

### Schritt 3 – Array-Einstellungen und das prognostizierte Ergebnis

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Beim Aufrufen dieser Szene fragt Chloros das Backend nach einer **Empfehlung**und wendet diese an: den**vollständigen Sensor – kein Binning, kein ROI-Ausschnitt – wann immer die Bildrate dies zulässt**, unabhängig davon, welche Rate der Host und die Verbindung liefern. Nur wenn keine Rate den vollständigen Sensor liefern kann, verkleinert die Empfehlung das Bild, und bevorzugt dann Binning gegenüber dem ROI-Ausschnitt, da durch Binning das gesamte Sichtfeld erhalten bleibt. Ab diesem Punkt tauschen Sie Auflösung gegen Bildrate ein. Bei jeder Änderung wird die Analyse live neu durchgeführt und das rechte Fenster**„Prognostiziertes Ergebnis“** aktualisiert.

Linke Spalte – Einstellungen:

| Steuerung | Auswahlmöglichkeiten | Standard | Anmerkungen |
| --- | --- | --- | --- |
| **ROI (Sichtfeld)** | Voll (2048×1536) / Halb (1024×768) / Viertel (512×384) | Voll | Sensorausschnitt: Halb-/Viertel-Ausschnitt auf einen kleineren Bereich bei nativem Pixelabstand. |
| **Binning** | 1× / 2× (Summe 2×2) / 4× (Summe 4×4) | 1× | Hardware-Binning: 2×2 = volles Sichtfeld bei einem Viertel der Leitungsbelegung; 4×4 = volles Sichtfeld bei 1/16. Ausgeblendet, wenn die Kameras kein Binning unterstützen. |
| **Leitungsseitiges Bild** (Auslesung) | — | — | Die nach dem Binning tatsächlich über die Leitung gesendete Breite × Höhe, auf Vielfache von 16 (mindestens 64) gerundet. |
| **Triggerrate** | 0,5–60 fps, Schrittweite 0,1 | leer = automatisch | Die Triggerrate des Masters. Leer lassen, damit Chloros den Wert ableiten kann. |
| **Leitungsbandbreite**| 20–2000 MB/s, Schrittweite 10 | leer = automatisch | Wie viel der Host tatsächlich verarbeiten kann, in MB/s —**die einzige Zahl, von der die gesamte Array-Zuweisung abhängt.** Wird vom Netzwerkadapter automatisch erkannt. Verringern Sie diesen Wert, wenn das Array beschädigte Frames meldet: Der erkannte Wert überschätzt USB-Adapter und gemeinsam genutzte Switches. Durch eine Änderung wird die Live-Prognose neu berechnet. Das Feld wird mit dem Budget geöffnet, mit dem das letzte Array verbunden war (zusammen mit dem Projekt gespeichert), sodass ein einmal korrigiertes Budget beibehalten wird. |

Das Kontrollkästchen **Pin-Auflösung** aus Version 1.2.0 ist entfernt worden: „Connect“ behält stets die von Ihnen gewählte Frame-Größe bei und senkt stattdessen die Bildrate.

Rechte Spalte — **Prognostiziertes Ergebnis**:

* **Synchronisationsstufe** – „Simultane Erfassung“ (grün), „Simultane Erfassung (FTD-versetzte Aussendung)“ (grün), „Versetzte Erfassung (100 ms Drift)“ (gelb) oder „Konfiguration zu groß“ (rot). Chloros wählt die Stufe aus: simultane Erfassung, wann immer die Leitung mit einer Rate gemeinsam genutzt werden kann, an die sich die Kameras anpassen lassen (die Bildrate sinkt, wenn der Anteil jeder Kamera schrumpft), und versetzte Erfassung nur dann, wenn dies nicht möglich ist – so lässt sich ein Array in jedem Netzwerk mit der Bildrate verbinden, die das Netzwerk zulässt.
* **fps-Prognose** – die Bildrate, mit der das Array voraussichtlich laufen wird. Bei der simultanen Aufnahme ist dies ein einziger Wert: Die Rate steht an erster Stelle, und die automatische Belichtung füllt den dadurch vorgegebenen Zeitraum aus und erhöht dann die Verstärkung, sodass eine dunkle Szene diese nicht verringert (siehe [Wie die automatische Belichtung Belichtungszeit und Verstärkung einsetzt](camera-settings.md)). Bei versetzter Aufnahme wird dies als Bereich („dunkel → hell“) angezeigt, da die Belichtungszeit jeder Kamera den Versatz in einer dunklen Szene verlängert.
* **NIC-Leitung** – Verbindungsgeschwindigkeit und Dauerdurchsatz („NIC {mbps} Mbps · Dauer {N} MB/s“).
* **Sim-Emit-Burst-Prüfung** — Kann der NIC-Ring des Hosts einen gleichzeitigen Burst von allen Kameras aufnehmen („Sim-Emit-Burst: X MB · nutzbarer NIC-Ring: Y MB ✓/✗“)?
* **Überprüfung des Leitungsbudgets** — aggregierter Bedarf im stationären Zustand im Vergleich zur kollisionssicheren Leitungsobergrenze („Leitungsbudget: {Bedarf} MB/s, gefordert von {n} Kameras · Obergrenze {Obergrenze} MB/s ✓/✗ überbelegt“).
* **„Max. Kameras auf dieser Leitung: {n} – festgelegt durch die Mindestbandbreite pro Kamera, sodass das Binning diesen Wert nicht erhöht.“** – wird angezeigt, wenn Sie nahe an der Obergrenze für die Kameranzahl sind (oder diese überschreiten).
* **„Bei diesen Einstellungen gehen Frames verloren.“**— rote Warnung mit der Begründung des Backends sowie einer Liste der Blockierer und blauen**Behebungsvorschlägen** („Um dieses Array ins Netzwerk einzupassen“ / „Um die gleichzeitige Aufnahme freizuschalten“).**Anwenden & Verbinden** ist gesperrt, bis eine Prognose vorliegt, und die Beschriftung erklärt, warum die Funktion abgelehnt wird:

| Beschriftung der Schaltfläche | Bedeutung | Was tatsächlich hilft |
| --- | --- | --- |
| „Wird analysiert...“ | Analyse noch im Gange. | Warten. |
| **„Zu viele Kameras für dieses Netzwerk“**| Die Leitung kann das Array nicht einmal mit einer Kamera nach der anderen übertragen (Aggregationsprüfung fehlgeschlagen): weit mehr Kameras, als sie bewältigen kann, oder andere Kameras, die über sie streamen. | Weniger Kameras, Jumbo-Frames durchgehend oder eine schnellere Netzwerkkarte.**Ein kleinerer ROI hilft NICHT** – siehe unten. |
| **„ROI reduzieren, um zu aktivieren“** | Bei diesen Einstellungen würden Frames verloren gehen (Burst-/Ring-Prüfung fehlgeschlagen). | ROI reduzieren, Binning erhöhen oder den Empfangsring der Netzwerkkarte reparieren. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Während des Verbindungsaufbaus kann ein grünes **Kalibrierungs-Download-Fenster** mit einem serienweisen Fortschrittsbalken erscheinen: Wenn eine Kamera zum ersten Mal an einen Rechner angeschlossen wird, lädt Chloros das ca. 3,8 MB große Werkskalibrierungspaket über GigE von der Kamera herunter (ca. 70 Sekunden pro Kamera). Bei bereits zwischengespeicherten Kameras wird dieses Fenster nicht angezeigt; bei mehr als sechs Kameras werden die Zeilen gescrollt. Siehe [Kameras anschließen](connecting.md).

### Gepackter 12-Bit-Transport

12-Bit-Formate (`BayerRG12`, `Mono12`) werden normalerweise als 16-Bit-Wörter übertragen. **Der**gepackte**Transport sendet dieselben 12 Bit in 1,5 Byte pro Pixel und entpackt sie auf dem Host, sodass die Übertragung jedes Bildes 25 % weniger Zeit in Anspruch nimmt – bei voller Sensorauslastung ergibt dies etwa ein Drittel mehr Bilder pro Sekunde.**Die gespeicherten Daten sind identisch.** „Array Connect“ entscheidet automatisch (`auto`): Es wird nur dann gepackt, wenn dies sinnvoll ist und der Host über genügend CPU-Leistung für das Entpacken verfügt. In der GUI gibt es hierfür keine Option; über die CLI verwenden Sie `array-connect --packed {auto,on,off}` (Standard: `auto`). 8-Bit- und 16-Bit-Formate werden niemals komprimiert.

## Bandbreite: Wie viele Kameras passen?

Was ein Array übertragen kann, hängt von der Leitung ab, nicht von Chloros. Die Planungswerte finden Sie daher im Hardware-Handbuch: **[Planung der Array-Bandbreite](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Was Chloros damit macht: Der Verbindungsdialog führt einen Netzwerk-Test durch, prognostiziert die erreichbare Bildrate und wählt eine passende Stufe aus. Ein Array lässt sich in jedes Netzwerk einbinden: Bei einer langsamen Verbindung wird jede Kamera an ihren Anteil an der Leitungsbandbreite angepasst und die Bildrate sinkt, anstatt dass das Array abgelehnt oder seine Frames verkleinert werden. Nur ein Array, bei dem die Verbindung nicht einmal eine Kamera gleichzeitig übertragen kann, wird abgelehnt (**Zu viele Kameras für dieses Netzwerk**), anstatt Pakete stillschweigend zu verwerfen – siehe das oben beschriebene Fenster mit den prognostizierten Ergebnissen.

## Wenn Frames fehlen

Eine Kamera kann aus zwei völlig unterschiedlichen Gründen in einer veröffentlichten Gruppe fehlen,
und diese erfordern gegensätzliche Lösungen. Chloros zählt sie separat, anstatt eine
„unvollständige“ Zahl zu melden, die keinen der beiden Fälle benennt:

| Was ist passiert? | Was bedeutet das? | Wo nachsehen? |
| --- | --- | --- |
| **Beschädigt**– das Bild ist angekommen, war aber strukturell fehlerhaft | GVSP-Paketverlust auf dem Netzwerkpfad | Das**Leitungsbudget**, der NIC-Empfangsring, Jumbo-Frames, der Switch |
| **Nie angekommen**– es kam überhaupt kein Frame | Die Kamera hat nicht ausgelöst oder es wurde nichts gesendet | Das**M8-Synchronisationskabel**, die Synchronisationsleitung, ob jedes Mitglied aktiviert ist |

Die Aufteilung wird alle 10 Sekunden neu bewertet, während das Array streamt. Bei Werten über 5 % wird dies
mit Angabe beider Zahlen protokolliert, und jeder beschädigte Puffer wird beim ersten Auftreten
pro Kamera gemeldet, anschließend einmal pro Minute zusammengefasst, damit eine lange Sitzung lesbar bleibt.

**Beschädigte Frames mit dem Wert „0“ bei „Nie angekommen“ bedeuten, dass Auslösung und Kabelsynchronisation einwandfrei sind**und jeder verlorene Frame auf dem Netzwerkpfad liegt. Die Lösung besteht darin, das**Wire Budget** zu senken und
die Verbindung neu herzustellen.

{% hint style="warning" %}
**Eine Verringerung der Triggerrate hilft nicht bei fehlerhaften Frames.** Das Paket-Pacing der Kamera
wird einmalig beim Verbindungsaufbau festgelegt. Eine Verringerung der Triggerrate ändert, wie oft ein Burst
auftritt, nicht, wie schnell der Burst selbst über die Leitung übertragen wird. Bei einem getesteten System mit 4 Kameras
hat eine 5-fache Reduzierung der Triggerrate nichts geändert, während die Senkung des Wire Budgets von 240 auf
200 MB/s senkte den Anteil fehlerhafter Frames bei derselben Anlage von 10,4 % auf null.
{% endhint %}

Ein laufendes Array kann sich nicht neu planen – trennen Sie die Verbindung und stellen Sie sie erneut her, damit der „Connect-Time“-Picker
mit dem neuen Budget arbeiten kann.

### USB-Netzwerkadapter sind begrenzt

Ein USB-Ethernet-Adapter gibt seine *Ethernet*-Verbindungsrate an, doch was er tatsächlich
aufrechterhalten kann, wird durch den USB-Bus und dessen Treiber begrenzt. Einem USB-10-GbE-Dongle wurden früher
etwa 1000 MB/s Durchsatz zugeschrieben — ein Wert, der noch nie gemessen worden war — und die Anpassung
von vier Kameras an diesen vermeintlichen Spielraum führte zu einer Verfälschung von 6–18 % der Bilder, während das Array
weiterhin eine einwandfreie Zielbildrate meldete. USB-Adapter sind nun auf
**200 MB/s** unter Windows und auf NVIDIA Jetson sowie auf **400 MB/s** unter Linux x86_64 begrenzt. Die Begrenzung ist ein absoluter Wert und kein Prozentsatz, da der
Bus die Grenze darstellt: Ein USB-1-GbE-Adapter erreicht etwa 80 MB/s und ist davon nicht betroffen.

Wenn Ihr Host tatsächlich schneller ist als die Obergrenze, erhöhen Sie das **Wire Budget** entsprechend.

## PTP-Zeitsynchronisation

Die *Synchronisation* der Frames erfolgt über den Hardware-Trigger; **PTP** (IEEE 1588 PTPv2) sorgt für vergleichbare *Zeitstempel* auf allen Geräten. Es ist standardmäßig beim Verbinden des Arrays aktiviert:

* Das **Host-Backend von Chloros läuft als PTP-Grandmaster**. LATTICE-Kameras und DAQ-E-Lichtsensoren sind in Domäne 0 daran gekoppelt, sodass Bildzeitstempel und DAQ-Spektren auf einer Taktfrequenz (~1 ms) liegen.
* `--no-ptp` (CLI) deaktiviert diese Funktion für Laborarbeiten – Zeitstempel verschiedener Kameras sind dann **nicht** vergleichbar.
* Überprüfen Sie den Synchronisationsstatus über die CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Die Registerkarte „Kameras“ selbst verfügt über keine PTP-Anzeige; dort werden pro Kamera die Synchronisationsdaten in den schreibgeschützten Feldern **Rolle**(Master/Slave),**Synchronisationsleitung** und die Fähigkeitenebene des Arrays angezeigt. Der PTP-Status von DAQ-E wird in den Sensordetails auf der Registerkarte „Lichtsensoren“ angezeigt.

## Die Live-Array-Ansicht

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

Der Hauptanzeigebereich bietet zwei Layouts (Umschaltung in der oberen Leiste): die **Rasteransicht**(jede Kachel ist eine Zelle; per Drag-and-Drop neu anordnen, wenn das Raster-Schloss entsperrt ist) und**Listenansicht**(Zeilen in voller Breite: Ein**kombiniertes**Array bildet eine Zeile – seine zusammengesetzte Kachel –, während ein**separates**Array**jedem Mitglied eine eigene Zeile**zuweist, mit eigenem Live-Feed und eigener Kalibrierungsziel-Tabelle; unterhalb der Arrays eine eigenständige Kamera – die aktive). Der Schieberegler**Feed-Zoom** passt die Größe der Kacheln an; bei einer Zellenbreite unter 200 px werden die Namens- und Bildraten-Einblendungen automatisch ausgeblendet. Die zugehörigen Kameras erscheinen in der Seitenleiste und im Standardlayout des Rasters in der Reihenfolge der Kameras (M3C vor M3M, FRGB zuerst, Mono-Kameras nach Wellenlänge); das Entsperren des Rasters und das Ziehen überschreibt weiterhin die Rasterreihenfolge.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



Im **separaten Modus** wird eine Kachel pro Kamera angezeigt. Jede Kachel enthält folgende Einblendungen:

* den Kameranamen (oben links),
* eine **fps-Anzeige** (oben rechts) – dies ist die vom Backend gemeldete *tatsächliche Bildaufnahmerate* der Kamera, nicht die Abfragerate der Vorschau (die Live-Vorschau ist unabhängig von der Bildaufnahmerate auf 30 fps begrenzt),
* einen Statuspunkt – grün (Streaming) / gelb (Laden) / rot (Fehler),
* einen **Spinner für veraltete Bilder**, wenn seit 2 s kein neues Bild eingegangen ist – normal für ca. 5 s nach jeder Verbindungsherstellung/Trennung, während das Backend das Bandbreitenbudget auf die Kameras neu verteilt.

Im **kombinierten Modus**wird eine einzige zusammengesetzte Kachel angezeigt: Das Backend führt Debayering, Skalierung, Ausrichtung und Rauschunterdrückung durch, wandelt die Daten in bandweise Strahlungsintensität um (zuzüglich DLS-Reflexionsgrad, wenn ein Lichtsensor gebunden ist), wertet den Indexausdruck des Arrays aus, wendet die LUT an und streamt das Ergebnis als MJPEG. Bis das erste ausgerichtete Bild gerendert wird, zeigt die Kachel ihren Status an: „Array wird vorbereitet…“, „Ausrichtung kalibrieren…“, „Warten auf erstes Bild…“ oder – falls die automatischen Ausrichtungsversuche der Kachel (ca. eine Minute) erschöpft sind – „Ausrichtung erforderlich“ mit einer Schaltfläche**Ausrichtung kalibrieren**.

Die FPS-Anzeige der Array-Kachel gibt zudem den Status des Streams an:**„Einstellungen werden angewendet…“**, während eine Änderung der Einstellungen (Format, Binning, ROI) die Kameras neu startet,**„Blockiert“**, wenn das Array aktiv ist, aber keine Bildgruppen mehr liefert, und**„Gestoppt“**, wenn es den Stream eingestellt hat – in diesem Fall das Array erneut verbinden. Wenn die spektrale Entmischung ein im Composite angezeigtes Band bei einem großen Anteil seiner Pixel (25 % oder mehr) begrenzen musste, zeigt die Kachel**„Banddaten mit geringer Zuverlässigkeit · N %“** an: Auf diesem Band basierende Indexwerte sind fragwürdig (in der Regel hat ein Band zu wenig Licht empfangen).

Nützliche Informationen zum Kombinationsmodus:

* Das Composite ist auf das Bild der **Master**-Kamera ausgerichtet. Die AE-ROI-Ausrichtung und die Spotmessung auf dem Composite sind für die Master-Kamera exakt und für die Slave-Kameras nur annähernd; verwende**Split View** (Array-Einstellungen → „Mitgliederkameras anzeigen“), um pixelgenaue Kacheln pro Kamera zu erhalten, ohne zusätzliche Kameraverbindungen zu öffnen.
* **Mit**Anzeigeebenen**(Array-Einstellungen; standardmäßig deaktiviert) können Sie eine Vordergrund- und eine Hintergrundebene auswählen – jede Mitgliedskamera oder**Index**. Bei „Vordergrund = Index“ zeigen Pixel außerhalb des LUT-Min/Max-Bereichs die Hintergrundebene an.
* **Render-Auflösung** (Standard 720p) legt die Höhe des Live-Streams *und* die Exportgröße des gespeicherten Composites fest. Bilder der einzelnen Kameras werden immer in voller Auflösung exportiert.
* Die Ausrichtung wird pro Sitzung berechnet und niemals gespeichert. Chloros richtet **jedes Array automatisch nach dem Verbindungsaufbau in beiden Anzeigemodi**aus, sobald sich die automatische Belichtung der Mitglieder stabilisiert hat (es wird bis zu 5 Minuten lang versucht). Der Abschnitt**Ausrichtung**im Array-Einstellungsfenster zeigt die RMS-Abweichungen und die Schaltfläche**Ausrichtung neu kalibrieren** an – auch für einzelne Arrays, die keine zusammengesetzte Kachel haben.

## Aufnahme: Überwachung vs. Analyse

Die Aufnahmeoberflächen der Arrays lassen sich klar in **Überwachungsqualität**(Aufzeichnung des Sichtbaren) und**Analysequalität** (Aufzeichnung von Rohdaten, spätere Kalibrierung) unterteilen:

| Workflow | Qualität | Was wird gespeichert? | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Aufnahme**(Standbilder) | Analyse | Eine synchronisierte Bildgruppe pro Durchgang; kameraspezifische Dateien auf jeder ausgewählten Exportebene (Rohdaten/Debayered/Radiance/Reflexion/Vorschau/Index) + `.daq`-Sidecar |**Schaltfläche „**Alles erfassen**“ + Erfassungseinstellungen | `lattice array-capture` |
| **Indexvideo aufzeichnen** | Überwachung | Das angezeigte Live-Composite des kombinierten Index – 8 Bit, Vorschauauflösung, LUT eingebrannt. Über die CLI, ohne dass eine GUI-Ansicht geöffnet ist, rendert das Backend den NDVI-Wert des Arrays (oder `--expr`), sobald das Array ausgerichtet ist; eine Aufzeichnung, bei der kein Frame geschrieben wird, schlägt fehl | ● Indexvideo aufzeichnen (kombinierte Arrays) | `lattice array-record` |
| **Rohdaten-Burst → Video erstellen**| Analyse | Rohdaten-Sensorframes bei voller Erfassungsrate + Manifest + `.daq`, anschließend Offline-Rekonstruktion zu kalibriertem Strahlungsintensitäts-/Reflexions-/Index-Video, zeitlich abgeglichen mit den DAQ-Messwerten | ⦿ Rohdaten-Burst aufzeichnen →**Video erstellen** | `lattice array-burst` → `lattice array-build-video` |

Faustregel: Wenn die Pixel *Messungen* liefern sollen, verwenden Sie „Capture“ oder „Burst“ (Analysequalität); wenn Sie lediglich *beobachten oder demonstrieren* möchten, was das Array gesehen hat, zeichnen Sie das Index-Video auf (Überwachungsqualität).

### Aufnahmeeinstellungen (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Das Zahnrad-Symbol neben **„Capture All“** öffnet den Bereich „Aufnahmeeinstellungen“ (erfordert ein geöffnetes Projekt – Aufnahmen werden darin gespeichert):

* **Aufnahmemodus**:**Einzelaufnahme**(ein Durchgang) /**Kontinuierlich**(aufeinanderfolgend; begrenzt durch eine Aufnahmemenge, Standard 1, oder eine Dauer, Standard 10 s) /**Intervall** (Zeitraffer: N Aufnahmen alle X Intervalle für insgesamt Y, standardmäßig 1 alle 5 s für 1 Minute).
* **Exporttypen pro Kamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index – alle zutreffenden Optionen sind standardmäßig aktiviert. Radiance/Reflectance sind bei Kameras mit RGB-Filter ausgeblendet; **„Reflectance“ erscheint nur, wenn die Kamera über einen DAQ-Lichtsensor verfügt** (einen eigenen oder vom Array geerbten); „Index“ erfordert einen konfigurierten Indexausdruck.
* **Ausgerichtet**(pro Array, standardmäßig**aktiviert**): Passt die Exporte der Elemente an das Ausrichtungsprofil des Arrays an, sodass die Exporte pixelgenau registriert sind. „Raw“ bleibt immer unverzerrt, enthält die Transformation jedoch in den Metadaten. Array-Mitgliedsdateien werden in der nativen Ausrichtung der jeweiligen Kamera gespeichert (etwaige Spiegelungs-/Rotationseinstellungen werden in der Datei aufgezeichnet, aber nicht angewendet).
* **Schnellste Erfassung** (Umschaltfunktion): Nur „Raw“ + der zugewiesene DAQ-Messwert + die freie kombinierte Index-Komposition; dabei wird die Kalibrierungsberechnung zum Zeitpunkt der Erfassung übersprungen, um die maximale Bildrate zu erreichen – Radiance/Reflexion/Index werden später aus der gespeicherten Datei „`.daq`“ neu berechnet.
* Auswahlen bleiben im Projekt erhalten. Ausgeblendete oder angehaltene Kameras werden übersprungen.
* **„Continuous“** bei Arrays läuft im Chloros-Backend (neu in 1.3.0): Wenn jede Kamera in der Liste zu einem Array gehört, führt das Backend die Aufnahmeschleife selbst aus und schreibt eine Gruppe, während es die nächste auswählt, ohne dass pro Aufnahme eine Anfrage gestellt wird. Ein volles Laufwerk stoppt den Lauf („Gestoppt: Laufwerk voll. N gespeichert, F fehlgeschlagen“). Siehe [Aufnahmeeinstellungen und -modi](capture.md#capture-modes) für den Hinweis von Microsoft Defender zu hohen Aufnahmeraten.

Die entsprechende CLI-Befehlszeile (derselbe Backend-Endpunkt, dieselbe Semantik):

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

Die TIFF-Komprimierung für Erfassungen ist `deflate` (verlustfrei, Standard) oder `none` – vollständige Flag-Tabellen, die Struktur des Erfassungsordners und Regeln zur Nachbearbeitung finden Sie in der [CLI-Referenz](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Koppeln eines DAQ-Lichtsensors

Reflexions- und beleuchtungskorrigierte Vorschauen benötigen Daten zum nach unten gerichteten Licht von einem DAQ-Sensor (angeschlossen auf der Registerkarte **Lichtsensoren**):

* Die **Array-Zeile**in der Seitenleiste zeigt eine**„DAQ · ein/aus“-Schaltfläche** an – *ein*, wenn ein Lichtsensor auf Array-Ebene eingestellt ist **oder** eine der zugehörigen Kameras über einen eigenen verfügt; der Tooltip listet genau auf, welcher Sensor welche Kamera versorgt.
* Die Zuweisung für das gesamte Array erfolgt in den Array-Einstellungen → **Umgebungslichtsensor**→ Dropdown-Menü**Lichtsensor**. Die Auswahl bleibt im Projekt erhalten, wird auf jede einzelne Kamera im Array übertragen, und einzelne Kameras können sie weiterhin durch ihren eigenen Sensor überschreiben.
* Die Statuszeile darunter zeigt den aktuellen Status an: **Aus**→ „Warten auf erstes Spektrum…“ →**„Aktiv – alle Kameras im Array sind beleuchtungskorrigiert“** → oder, falls in den letzten 3 Sekunden kein neues Spektrum eingegangen ist, ein Hinweis auf veraltete Daten – der letzte Messwert wird weiterhin verwendet (Messwerte verfallen im Aufnahmepfad nie).

Bei zugewiesenem Sensor: Der Exporttyp „Reflexionsgrad“ wird verfügbar, Live-Vorschauen sind beleuchtungskorrigiert, die prädiktive Belichtungsautomatik kann das Spektrum nutzen, und jede Reflexionsgrad-Aufnahme schreibt den tatsächlich verwendeten DAQ-Messwert als **`.daq`-Datei** in den `daq/`-Ordner der Erfassung gespeichert, sodass die Erfassung später erneut verarbeitet werden kann. Daneben benennt eine `.daq.cameras.json`-Datei die Kameras, deren Bilder dieser Messwert beschreibt; bei der erneuten Verarbeitung ordnet Chloros den Messwert nur Bildern dieser Kameras zu.

## `array-connect`-CLI-Optionen

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Alle LATTICE-Kameras automatisch erkennen (erfordert ≥2) | **Die erste Seriennummer ist die des MASTERS.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-Synchronisationsleitung. |
| `--target-fps F` | auto | Auslösefrequenz des Masters. |
| `--binning {1,2,4}` | auto | Hardware-Binning. „Auto“ bedeutet den vollen Sensor (kein Binning), wann immer eine Bildrate dies zulässt; „connect“ führt niemals Binning durch, um die Bildrate aufrechtzuerhalten. |
| `--packed {auto,on,off}` | `auto` | [Gepackter 12-Bit-Transport](#packed-12-bit-transport): `auto` komprimiert nur, wenn es sinnvoll ist und der Host die Dekomprimierung bewältigen kann. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Experten-Übersteuerung der Auswahl der Synchronisationsstufe. |
| `--wire-ceiling-mbps MB_PER_S` | automatisch erkannt | Host-Bandbreitenbudget in MB/s – die CLI-Darstellung des Feldes **Wire Budget**. Verringern Sie diesen Wert, wenn das Array beschädigte Frames meldet. Wird mit dem Projekt gespeichert, sodass er bei einer späteren erneuten Verbindung wiederhergestellt wird. |
| `--no-recommend` | aus | Den Schritt der Netzwerkanalyse überspringen. |
| `--no-ptp` | aus | PTP deaktivieren (Zeitstempel verschiedener Kameras sind dann nicht vergleichbar). |

`lattice array-list`, `array-status` und `array-disconnect` verwalten die persistente Sitzung. Die vollständige Referenz zu den Unterbefehlen, einschließlich der Ausrichtung (`align-calibrate` / `align-apply`) und der Netzwerk-Tools, finden Sie im [CLI-Referenz § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); die entsprechenden SDK-Befehle (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) finden Sie in der [SDK-Referenz](../reference/sdk-reference.md). In Python lautet das „Wire Budget“ `connect_array(..., wire_ceiling_mbps=120)`, und die Unterscheidung zwischen „live corrupt“ und „never-arrived“ findet sich unter [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
