# Chloros-CLI-Referenz

**Version:**

1.3.0**Erstellt:**

29.07.2026 19:19 ·**Überarbeitet:**

06.11.2026**Zielgruppe:** Optimiert für die Nutzung durch LLMs; für Menschen lesbar.**Geltungsbereich:** Alle benutzerspezifischen Unterbefehle von `chloros-cli`, einschließlich Optionen und Beispielen zum Kopieren und Einfügen.

Dieses Dokument ist die vollständige Referenz für das Befehlszeilentool `chloros-cli`, das im Lieferumfang von MAPIR Chloros enthalten ist. Es ist bewusst umfassend gestaltet, damit ein LLM (oder ein Mensch) jeden unterstützten Workflow anhand der folgenden Auflistungen zusammenstellen kann, ohne den Quellcode einsehen zu müssen.

Wenn Sie nur die wichtigsten Punkte benötigen, springen Sie zu:
- [Fünf-Minuten-Schnellstart](#five-minute-quickstart)
- [Workflow für die Erstverbindung mit der LATTICE-Kamera](#lattice-camera-first-connect-workflow)
- [Workflow für die Erstverbindung mit dem DAQ-Sensor](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Aufnahmemodi, Recorder und Offline-Nachbearbeitung](#capture-modes-recorders--offline-reprocess)

> **Chloros-Steuerung über einen KI-Assistenten?** Claude Desktop, Claude Code, Cursor und andere MCP-Clients können Processing, LATTICE-Kameras und -Arrays sowie DAQ-Lichtsensoren über `chloros-mcp` steuern, dem MCP-Server, der mit dem Python-SDK installiert wird (sowie mit dem Linux-.deb-Paket, in `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Einrichtung, Optionen und Tools finden Sie in der SDK-Referenz unter [MCP-Server (KI-Assistenten)](sdk-reference.md#mcp-server-ai-assistants) der SDK-Referenz. Ein Assistent mit einer Shell kann zudem jeden Befehl auf dieser Seite direkt ausführen.

> **Geändert in 1.3.0** (was ein bestehendes Skript oder ein Workflow der Version 1.2.0 bemerkt).
>
> **Änderungen, die ein Skript der Version 1.2.0 möglicherweise berücksichtigen muss:**
>
> - **`--preset` gilt.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` und `align-stream` wurden in Version 1.2 entfernt.0 entfernt und lief mit den kameraseitigen Einstellungen. Es legt nun das Format, die Belichtung, die Verstärkung und die Bildrate der Voreinstellung fest: `--preset high_speed` nimmt 8 Bit bei 5 ms, 6 dB und 60 fps. Auch `lattice cam-connect --preset` behält das Format der Voreinstellung bei (in Version 1.2.0 wurde die Kamera nach Anwendung der Voreinstellung auf 12 Bit eingestellt). Siehe [Allgemeine Optionen](#common-options-shared-by-most-subcommands).
> - **Ein Bayer-Pixelformat bei einer M3M-Kamera (Mono) entspricht deren Farbtiefe.** Die Voreinstellungen benennen Bayer-Formate, die ein Mono-Sensor nicht aufnehmen kann: In Version 1.2.0 lehnte die Kamera das Format ab und behielt das bisherige bei, sodass `--preset high_speed` dort nicht 8-Bit war. Es handelt sich nun um das Mono-Format mit derselben Tiefe: `high_speed` und `--pixel-format BayerRG8` ergeben `Mono8`, `high_quality`, `triggered` und `BayerRG12` ergeben `Mono12`. In einem gemischten M3C- + M3M-Array verwendet jede Kamera das gemeinsame Format ihrer eigenen Familie, und `network-analysis` / `analyze-array --pixel-format` projizieren jede Kamera auf diese Tiefe. Das Gleiche gilt auch umgekehrt: Eine M3C, für die ein Mono-Format angefordert wurde, nimmt das Bayer-Format dieser Tiefe an, sodass ein Array, dessen Master eine M3M ist, seine M3C-Kameras nicht mehr im Mono-Modus. Siehe [Mono (M3M)-Einbandkameras](#mono-m3m-single-band-cameras).
> - **`lattice array-capture`-Exporte sind standardmäßig ausgerichtet, genau wie die der App.** Die `debayered`-, `display`, `radiance`, `reflectance` und Indexdateien werden in den Bildausschnitt der Master-Kamera transformiert und auf die Überlappung der Mitglieder zugeschnitten (die um die Offsets der Kameras kleiner ist als der Sensor, `XMP-mapir:AlignmentJSON` `"applied": true`), sodass eine CLI-Aufnahme aus denselben Dateien besteht wie die Aufnahme derselben Szene durch die Chloros-App; `raw` bleibt nativ. Zuvor wurden bei einer Aufnahme ohne `--aligned` alle Dateien in ihrer nativen Größe und unverzerrt gespeichert. `--no-aligned` behält dies bei. Die gleiche Standardeinstellung gilt für `--smart`, `--continuous` und `--interval` sowie für das SDK und `chloros-mcp`. Siehe [`array-capture`-Modi](#array-capture-modes).
> - **`lattice array-capture --compression` ist standardmäßig auf `none` eingestellt.** Dateien werden unkomprimiert geschrieben (ca. 6,3 MB pro Bild in voller Auflösung), um die schnellste Dauerübertragungsrate zu erzielen; Version 1.2.0 schrieb im Modus `deflate` (ca. 4,1 MB). Geben Sie `--compression deflate` an, um kleinere Dateien zu erhalten. Beide sind verlustfrei.
> - **Für PNG und JPEG werden nur die Formate `display` / `preview` sowie Index-Dateien unterstützt.** Eine `--format png`- oder `jpg`-Erfassung mit einem Pegel, den eine 8-Bit-Datei nicht aufnehmen kann, wird abgelehnt, bevor überhaupt etwas erfasst wird (HTTP-Code 400, Beenden von `1`), ebenso wie die Standardeinstellung `--processing all` mit `--format png`/`jpg`. Siehe [Erfassungs-Exportstufen](#capture-export-levels-the-all-default).
> - **`lattice dls` benötigt `--mac`.**Es gibt keine Standard-MAC-Adresse mehr (es handelte sich um die Adresse einer Einheit); `daq pool-discover --only ble` ermittelt die eines Sensors.**`lattice dls --camera SERIAL`** ist neu.
> - **Weitere Fehler führen zum Beenden von `1`.** `process`, wenn die Exporte eines Durchlaufs nicht verifiziert werden konnten oder ein Image verloren ging, obwohl die anderen Produkte geschrieben wurden; ein Erfassungsbefehl mit einer beliebigen Datei, die nicht geschrieben werden konnte (die anderen Dateien werden weiterhin gespeichert); `lattice continuous` und `multi-stream` bei vollem Laufwerk; `array-burst` / `array-record`, wenn der Recorder einen Fehler meldet; `array-build-video`, wenn ein angefordertes Produkt übersprungen oder nicht wie gewünscht erstellt wurde; `project burst` / `stream`, wenn eine Erfassung teilweise fehlschlägt; `project sensor log`, wenn die Aufzeichnung keinen Messwert enthält; `lattice exposure --set`, `gain --set` und `configure --set` mit einem Wert, den die Kamera ablehnt. `lattice network-analysis --json` wird beendet, da die Zusammenfassung beendet wird (`1` für `needs_force_slip`). Umgekehrt: `daq pool-disconnect` eines Sensors, der sich nicht mehr im Pool befindet, beendet `0`. Siehe [Beendigungscodes](#exit-codes).
> - **Ein langsames `lattice array-connect` schlägt bei angeschlossenem Array niemals fehl.** Er wartet bis zu 600 s (180 s in 1.2.0; `project connect` 90 s), gibt dabei den Schritt des Backends aus, und eine bis dahin nicht abgeschlossene Verbindung wird vom Backend rückgängig gemacht – jede Kamera wird in den Zustand zurückversetzt, in dem sie vorgefunden wurde –, sodass ein `Error:` (Beenden von `1`) bedeutet, dass kein Array verbunden ist. In Version 1.2.0 konnte eine Verbindung, die unmittelbar nach dem Einschalten der Kameras hergestellt wurde, die Wartezeit überstehen und das Array verbinden, nachdem der Befehl fehlgeschlagen war. Strg+C bricht die Verbindung nun ab. Siehe [Persistente Verbindungen](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` behält den vollständigen Sensor bei.** Ohne `--width` / `--height` / `--binning` wird jede Kamera auf den vollen Sensor eingestellt und die Bildrate sinkt auf das Niveau, das dieser Computer und das Netzwerk liefern; die Verbindungsausgabe listet schnellere Optionen (Binning oder ein kleinerer ROI) mit den entsprechenden prognostizierten Raten auf. In Version 1.2.0 wurden die Bilder verkleinert oder gebinnt, um die Rate aufrechtzuerhalten. Siehe [`array-connect`-Optionen](#array-connect-options).
> - **Ein langsames Netzwerk verhindert nicht mehr den Betrieb eines Arrays.** Jede Kamera wird an ihre jeweilige Bandbreite angepasst, und die Bildrate sinkt entsprechend. Wenn die Kameras nicht so langsam angepasst werden können, läuft das Array im seriellen Emit-Modus (`slip-emit-and-capture`): Jeder Frame kommt an, aber die Kameras werden nicht gemeinsam belichtet. `--force-tier sim-capture-sim-emit` besteht auf gleichzeitiger Belichtung. Siehe [Array-fps &amp; Burst-Modell](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` geben vor, welche „connect“-Einstellung gilt**: Ohne `--pixel-format` werden die von „connect“ festgelegten Formate (Mono12 / BayerRG12, zuvor BayerRG8) sowie die Entscheidung bezüglich [gepackter 12-Bit-Daten](#packed-12-bit-transport---packed) übernommen. Derselbe Befehl kann daher eine andere Bildrate angeben als in 1.2.0.
> - **Relative Pfade beziehen sich auf den Ort, an dem der Befehl ausgeführt wird.** `-o` und die anderen Pfade, die das Backend öffnet (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), das `output` eines Rezeptschritts, sowie die Standardwerte `output/` und `lattice index --live` werden relativ zu dem Ordner aufgelöst, aus dem Sie `chloros-cli` ausführen. In Version 1.2.0 hat das Backend sie in seinem eigenen Installationsordner aufgelöst, was bei einer installierten Version mit der Fehlermeldung „Zugriff verweigert“ fehlschlug. Siehe [Konventionen](#conventions).
> - **`lattice index` läuft in jeder Form auf dem Backend**, genau wie die anderen `lattice`-Befehle; starten Sie daher zuerst das Backend. Der Offline-Index (`--input`) wird vom Backend berechnet, das die TIFF-Datei einliest und die Ergebnisse daneben oder unter `-o` speichert, während `--list-presets` und `--list-gradients` die Tabellen des Backends ausgeben. Unter Windows funktionieren diese nun: Zuvor antwortete die Windows-CLI auf jede Form von `lattice index`, einschließlich `--live`, mit `Error: lattice_sdk not available`. Unter Linux wurde der Offline-Index früher innerhalb der CLI ohne Backend und ohne Anmeldung ausgeführt; nun benötigt er beides, wie jeder andere CLI-Befehl auch. Siehe [Index / Vegetationsberechnungen](#index--vegetation-maths).
> - **Neue Dateien neben den Produkten.** Jedes Radiance-, Reflektanz- und Indexprodukt, bei dem die Bandentmischung ein Pixel geklemmt hat, erhält eine Maske in einem Ordner „`clamp_masks/`“ neben dem Produktordner, und der Befehl „capture print“ gibt eine Zeile „`clamp mask:`“ aus. Siehe [Unmix-Clamp-Maske](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Kameraausrichtung.** Bei einer Aufnahme mit einer einzigen Kamera werden die Dateien „`raw`“ und „`debayered`“ so gespeichert, wie der Sensor sie ausgelesen hat, und die Spiegelung bzw. Drehung der Kamera wird in der XMP-Datei aufgezeichnet; „`process`“ wendet diese Informationen auf die erstellten Produkte an. Die Dateien der Array-Elemente bleiben im nativen Format, es sei denn, Sie übergeben die neuen `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`) übergeben. Siehe [Kameraausrichtung](#what-a-captures-folder-looks-like).
> - **Ein Exportstufenname, der nicht „eins“ lautet, wird abgelehnt.** `lattice capture --processing` und jede `--levels`-Liste akzeptiert `raw`, `debayered`, `radiance`, `reflectance` oder `display` (oder einen Alias wie `preview`; `--processing` sowie `all`). Jeder andere Name wird abgelehnt, bevor die Kamera berührt wird (HTTP-Fehler 400 `UNKNOWN_LEVEL`, Beenden `1`); 1.2.0 hat dafür `raw` gespeichert und wurde beendet mit `0`.
> - **`daq pool-set-cap` benötigt `--cap-id`.** Ohne diesen wird der Befehl beendet `2`; 1.2.0 hat die Abdeckungskorrektur entfernt, sodass ein ungeschützter Sensor verbleibt. `--cap-id none` fragt nach dem ungeschützten Sensor.
> - **`lattice resolution --binning-mode` ist nicht mehr vorhanden.** Das Binning erfolgt immer als Mittelwert; das Flag wird weiterhin akzeptiert, es wird eine Warnung ausgegeben und es wird nichts unternommen. Siehe [Sensor-Tuning](#sensor-tuning).
> - **`process --debayer texture-aware` wird für LATTICE-Aufnahmen abgelehnt.** Bei einem Ordner mit LATTICE-Aufnahmen beendet `process` nach dem Import `1`, gibt die Anzahl der Aufnahmen an und führt anschließend keine weitere Verarbeitung durch. LATTICE-Aufnahmen werden immer standardmäßig demosaiziert, daher hat Version 1.2.0 die Option akzeptiert und ignoriert; auch die Chloros-App bietet sie für diese Aufnahmen nicht an. Siehe [`chloros-cli process`](#chloros-cli-process).
> - **Kameras werden in den Zustand zurückversetzt, in dem sie vorgefunden wurden — es sei denn, `--keep-settings`.** `lattice array-disconnect`, jeder transiente Befehl (`multi-capture` ohne Array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) sowie `lattice continuous` versetzen jede Kamera wieder in den ursprünglichen Zustand: die Verkabelung von Auslöser und Synchronisationsleitung, die PTP-Einstellungen sowie alle während der Sitzung geänderten Kameraeinstellungen (Belichtung und Verstärkung, AE-Zielwert, Grenzwerte und Dämpfung, Weißabgleich, Schwarzpegel, Pixelformat, Binning und ROI, Spiegelungen, Bildfrequenz). `--keep-settings` speichert stattdessen die Einstellungen jeder KameraEinstellungen in den eigenen Speicher der Kamera, sodass sie mit diesen Einstellungen hochfährt; `--clear-settings` setzt diesen Speicher auf die Werkseinstellungen zurück. In beiden Fällen laufen die Kameras im Freilaufmodus weiter. Ein Projekt ruft beim Anschließen der Kameras deren eigene gespeicherte Einstellungen ab. Siehe [Kameraspeicher](#camera-memory----keep-settings----clear-settings).
> - **Jeder Modus speichert ein Projekt auf dieselbe Weise.** Jede Änderung an einer Kamera, einem Array oder einem Lichtsensor, die im Rahmen eines Projekts verbunden sind – sei es über die App, die CLI, das SDK oder den MCP-Server – wird im `cameras.json` / `sensors.json` gespeichert, und das Öffnen des Projekts in einem beliebigen Modus nimmt die darin enthaltenen Geräteeinstellungen vor. Zuvor speicherte nur die App das Projekt, und ihr nächster Speichervorgang überschrieb alle anderen Daten. Siehe [Ein Projekteintrag, jeder Modus](#one-project-record-every-mode).
> - **Dateien, die aus mehreren Kameras erstellt wurden, benennen jede einzelne davon.** Ein Multiband-Stack (`align-apply`, `align-stream`, `index --live --save-multiband`), ein aus den Bändern mehrerer Kameras erstellter Live-Index sowie das Bild „`composite/`“ einer Array-Aufnahme enthalten jeweils die Seriennummer, das Modell und das Kalibrierungspaket der jeweiligen Quellkamera als Listen (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …) sowie die EXIF-Daten (`SerialNumber`), in denen die Seriennummern durch Kommas getrennt aufgeführt sind. Früher enthielten sie entweder gar keine Kameradaten oder nur die Seriennummer der ersten Kamera. Siehe [Dateien, die aus mehreren Kameras erstellt wurden](#files-made-from-several-cameras).
> - **Nur eine `raw`-Datei enthält die Kalibrierungs-Tags der Kamera.** Der MicaSense-kompatible Satz (`BlackLevel`, `RadiometricCalibration`, die kanalbezogenen `Band{R,G,B}RadiometricCalibration`-Sätze, die `MicaSenseVignetting*`-Sätze) sowie die Nutzdaten `CalibrationJSON` beschreiben die Rohwerte des Sensors und sind daher ausschließlich in der Aufnahmedatei `raw` enthalten. Alle anderen Dateien – die `debayered`-, `display`-, `radiance`-, `reflectance` sowie die Indexdateien, die `lattice continuous`- und `multi-stream`-Frames, sowie jedes `process`-Produkt – enthält die Identifikation der Kamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, …) sowie bei einer Aufnahme die aufnahmenspezifischen Daten des Einzelbilds (`XMP-mapir:FrameJSON`: Belichtung, Verstärkung, Temperatur, Pixelformat, Aufnahmezeit). In Version 1.2.0 enthielten sie auch den Rohdatensatz, den ein Tool (Pix4D, Metashape) auf Pixel anwendet, die keine Rohwerte sind. `process` ermittelt die Kalibrierung einer solchen Datei anhand ihrer Seriennummer und `CalBundleSha256`.
>
> **Weitere Änderungen:**
>
> - **Die automatische Bildrate eines Arrays richtet sich nach den vom Array gelieferten Werten.** Eine Bildrate, die mehr als 1 % ihrer Trigger verliert, wird herabgesetzt – dies wird anhand mehrerer längerer Beobachtungszeiträume statt nur eines beurteilt –, und eine Bildrate, von der sie herabgesetzt wurde, wird von Zeit zu Zeit erneut versucht. Eine von Ihnen selbst festgelegte Bildrate bleibt unverändert.
> - **Die automatische Belichtung ist stabiler.** Sie bleibt bei konstanter Lichtstärke konstant, gleicht sich innerhalb von Sekunden nach einer Änderung an, ignoriert kleine Überbelichtungen und bringt eine Kamera wieder zum Laufen, die bei starker Sonneneinstrahlung nicht mehr reagiert. Siehe [So verhält sich die automatische Belichtung](#how-auto-exposure-behaves).
> - **`process` zählt nur Ihre Bilder und Aufnahmen.** Bei erneuter Ausführung auf einem Ordner, der die `camera_cal/`-Dateien eines Projekts enthält, werden diese Kalibrierungsdateien nicht mehr gezählt („96 Bilder gefunden“ bei einem Satz von 90 Bildern), und ein Projekt, das sie bereits aufgenommen hat, verwirft sie.
> - **`lattice multi-test` übernimmt den Einschaltzustand der Kameras.** Es schaltet das PTP der Kameras für den Durchlauf ein, wartet vor dem ersten Auslöser darauf, dass sich ihre Uhren synchronisieren und stabilisieren – jede Kamera ist 120 s lang synchronisiert, etwa 2 Minuten nach dem Einschalten (`--ptp-settle SECONDS` setzt dies), da eine gerade synchronisierte Uhr etwa 2 Minuten lang eine Abweichung von bis zu ~200 µs aufweist – und stellt das PTP jeder Kamera wieder so ein, wie es vorgefunden wurde. Auf eine Kamera, die noch aus einem früheren Test synchronisiert ist, wird nicht erneut gewartet. Früher wurde PTP in einem Netzwerk mit nur einem Switch, in dem kein DAQ-E antwortete, deaktiviert, und der frei laufende Taktgeber jeder Kamera wurde als PTP-Takt interpretiert, der sich nicht synchronisieren ließ (`PTP is not locked`, bei jedem Durchlauf). Siehe [Mehrkamera (transiente Sitzungen)](#multi-camera-transient-sessions).
> - **Eine `.daq`-Aufzeichnung gibt an, ob die Kappe ausgewählt oder angenommen wurde.** DAQ-U, DAQ-M und DAQ-E werden ohne Kappe ausgeliefert, und Chloros kann nicht erkennen, ob eine angebracht ist; daher erhält ein Sensor, der ohne benannte Kappe angeschlossen ist, standardmäßig den Wert `sunshine_cosine` – eine *Annahme*. Jede Chloros-Aufzeichnung (Schema 1.30) speichert nun neben `cap_id` auch `cap_id_source`: `operator`, wenn Sie der Kappe einen Namen gegeben haben (einschließlich `none`), und `auto_default`, wenn es sich um den Standardwert handelte. Bei der Verarbeitung einer Aufzeichnung, bei der die Kappe nur angenommen wurde, und bei der die DAQ-Kappe des Projekts auf „Auto“ belassen wurde, gibt ein `[daq-dls] WARNING` aus, das den Dateinamen enthält: Ein reines DAQ-U unter der Sonnenlichtkorrektur liefert einen etwa 30-fach zu hohen Wert. **Benennen Sie die von Ihnen angepasste Obergrenze** beim Verbinden oder legen Sie die DAQ-Obergrenze des Projekts fest, dann verschwindet die Warnung.
> - **Keine PIP-Konsolenfenster mehr unter Windows.** Auf einem PC mit installiertem Python führten das Backend und jeder seiner Image-Worker beim Start einmal den Befehl „`pip`“ aus und jeder öffnete ein Konsolenfenster. Das ist nun nicht mehr der Fall.
> - **Live-Aufzeichnungen von `.daq` überstehen einen Stromausfall.**Eine Aufzeichnung wird bei jedem Commit hinter dem Sensor mit einem Journal und einer vollständigen Festplattensynchronisierung geschrieben, sodass eine langsame Festplatte die Datei verzögert, niemals aber die Messwerte. Eine Aufzeichnung, die mitten im Commit gestoppt wurde (Stromausfall, Abbruch), kann „`<name>.daq-journal`“ daneben hinterlassen:**Kopieren Sie die beiden zusammen.** Chloros rollt das Journal beim ersten Lesen der Datei zurück, sodass die Aufzeichnung mit allen vor dem Stopp festgeschriebenen Daten geöffnet wird (etwa die letzten 3 s gehen verloren); in 1.2.0 konnte ein Stromausfall im falschen Moment die gesamte Datei unlesbar machen („Datenbank-Festplattenabbild ist fehlerhaft“).
> - **Eine Aufzeichnung auf einer ausgelasteten Festplatte blockiert den Sensor nicht mehr.** `daq pool-record` stellt Messwerte in eine Warteschlange und schreibt sie in einem eigenen Thread; ein Messwert, der nicht geschrieben werden konnte, wird gezählt (`daq pool-list` zeigt `N write error(s)` mit dem letzten Fehler an), anstatt den Sensor zum Einfrieren zu bringen (ein Schreibvorgang dauerte 8,4 s neben vier Kameras, die TIFFs speicherten).
> - **`lattice multi-capture` ohne Array verliert nicht mehr die gespeicherten Bilder, wenn ein Trigger eine Zeitüberschreitung aufweist.** Ein Auslöser einer Kamera, dessen Zeitlimit abgelaufen ist (`SC_ERR_TIMEOUT -1011`, beobachtet bei vier Kameras an einem 1-GbE-Switch), beendet den Lauf bei dieser Bildgruppe: Die bereits gespeicherten Gruppen werden mit `Saved:` aufgelistet, gefolgt vom Fehler und Beenden von `1`. In Version 1.2.0 war dies ein HTTP-500-Fehler ohne Liste, obwohl die Dateien auf der Festplatte vorhanden waren.
> - **Softwaregesteuerte Kameras lassen nicht mehr jeden zweiten Trigger aus.** Eine Kamera im Software-Trigger-Modus stellt nun einen Trigger, der während ihrer Auslesung eintrifft, in die Warteschlange (oder Chloros wartet, bis sie bereit ist), wie es kabelgesynchronisierte Kameras bereits taten.
> - **`daq pool-discover` benennt ein DAQ-E-S `DAQ-E-S`** in ihrer Spalte „`MODEL`“ (und „`--json`“ in der Spalte „`model`“). Für jeden Ethernet-Sensor wurde „`DAQ-E`“ angegeben; einer, der die Identitätsprüfung nicht innerhalb von etwa 2 s beantwortet, wird weiterhin als `DAQ-E` aufgeführt.
> - **Verbesserte LATTICE-Indexprodukte** (NDVI und andere Indizes) ab `process` in jedem Verarbeitungsmodus.
> - **OCN-Kameras: verbesserte Radiance-, Reflektanz- und Indexprodukte bei Sonneneinstrahlung.** Nach der Bandentmischung wird eine Flottenkorrektur angewendet; RGN-Kameras bleiben unverändert, und die OCN-Werte weichen von denen der Version 1.2.0 ab. Dies gilt auch für eine mit einer früheren Version erstellte OCN-Aufnahme, wenn diese auf einem Computer verarbeitet wird, auf dem die Kalibrierung gespeichert ist, mit der die Aufnahme erstellt wurde (Chloros behält die Kalibrierung einer Kamera bei, sobald diese angeschlossen wurde).
> - **`lattice continuous` zeichnet das Rohbild auf.** Ein `tiff`-Lauf schrieb demosaizierte 3-Ebenen-Bilder, von denen `process` nicht ausgehen kann. Seine Bilder entsprechen nun der Datei `raw`, die eine Array-Aufnahme schreibt: eine Mosaik-Ebene mit den sensoreigenen Werten, unkomprimiert, mit dem Kalibrierungspaket. Ein Skript, das drei Ebenen aus diesen Dateien liest, liest nur eine.
>
> - **Jede Aufnahme speichert den verwendeten DAQ-Messwert, genau wie die GUI.** `lattice capture`, `lattice continuous` (ein `.daq` pro Bild), `array-capture` und `multi-capture` mit einem verbundenen Array (und dem SDK) schreiben die `.daq` und dessen `.match.json`, wann immer ein DAQ gebunden oder in den Pool aufgenommen wird, einschließlich einer reinen Rohdatenerfassung; in 1.2.0 wurde bei einer Raw- oder debayered-Aufnahme nur eine Datei mit `--force-daq` gespeichert. `--no-daq` deaktiviert diese Funktion, `--daq-tolerance-ms MS` legt die Übereinstimmungstoleranz für diese Aufnahme fest, und das neue `daq pool-bind` bindet eine Kamera oder ein Array über die CLI an ein DAQ. Mit `.daq` wird eine gespeicherte Aufnahme in `process` nur mit den darin genannten Kameras abgeglichen. Eine Aufnahme wartet nun auf den nächsten Messwert des DAQ, damit sie die Messwerte auf beiden Seiten der Belichtung zusammenführen kann. Siehe [Der DAQ-Messwert, den eine Aufnahme speichert](#the-daq-reading-a-capture-saves) und [Ein DAQ-Messwert weit entfernt von der Belichtung](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** fügt den Index für die Bänder jeder Kamera zum Projekt hinzu, wobei ein neues Projekt einbezogen wird. In Version 1.2.0 wurde ein Name, der noch nicht im Projekt aufgeführt war, mit `Unknown index name(s) ignored` ignoriert.
> - **`process` neben einem Live-Array:**Es wird ein Backend, das Kameras oder DAQs enthält, nicht mehr neu gestartet, und es wird bis zu 15 s auf einen Lauf gewartet, der sich dem Ende nähert.**Strg+C** stoppt den Lauf im Backend, nicht nur in der CLI. Siehe [`process`-Hinweise](#notes).
> - **Ein Projektordner, der zu tief liegt und damit Windows’ Begrenzung auf 260 Zeichen im Pfad überschreitet**, wird von vornherein abgelehnt, wobei der längste zulässige Ordnername verwendet wird, anstatt einen Lauf auszuführen, der nichts schreibt. Der Windows-Installer bietet nun die Optionen „Windows-Langpfade aktivieren“ und eine Microsoft Defender-Ausnahme für das Backend an, die beide standardmäßig aktiviert sind. Siehe [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** führt die Backend-eigene kontinuierliche Erfassung aus: alle 2 Sekunden eine Fortschrittsanzeige, eine Zusammenfassung anstelle einer Auflistung pro Datei und einen Beendigungscode ungleich Null bei nicht geschriebenen Dateien oder einem vollen Laufwerk. Siehe [`array-capture`-Modi](#array-capture-modes).
> - **Das erste `array-capture` nach `array-connect`** wartet bis zu 20 s auf die erste Bildgruppe des Arrays, anstatt mit dem HTTP-Fehler 503 „Array noch im Aufwärmprozess“ abzubrechen.
> - **Belichtung eines Array-Mitglieds innerhalb eines ROI:** Ein kleiner ROI begrenzt die Belichtung eines Array-Mitglieds nicht mehr auf die Freilaufzeit des Sensors (4,9 ms bei 768 × 304). Siehe [Belichtung eines Array-Mitglieds](#exposure-on-an-array-member).
> - **Die automatische Belichtung senkt die automatische Bildrate eines Arrays nicht mehr.** Ein kabelgesynchronisiertes Array läuft mit der Rate, die das Netzwerk und die Auslesegeschwindigkeit der Kameras zulassen; die automatische Belichtung füllt den durch diese Rate vorgegebenen Zeitraum aus und erhöht dann die Verstärkung. Eine manuelle Belichtung, die länger als die Periode dauert (`lattice exposure --set`, `configure --set`), verlangsamt den Auslöser, um sie anzupassen, anstatt sie abzulehnen, und die Bildrate kehrt zurück, sobald das Array-Element wieder auf „Auto“ umgeschaltet wird. Siehe [Belichtung bei einem Array-Element](#exposure-on-an-array-member).
> - **Arrays liefern weniger unvollständige Bildgruppen** und weniger 503-Fehler („Keine neue synchronisierte Bildgruppe“): Der Auslöseeingang jeder Slave-Kamera ist nun glitchfrei.
> - **Gepackter 12-Bit-Transport** (`--packed`, Standard: `auto`) erhöht die Bildraten bei Vollsensor-Arrays um etwa ein Drittel. `array-connect --packet-size` wird nun angewendet (in Version 1.2.0 wurde es akzeptiert, aber ignoriert). Siehe [Gepackter 12-Bit-Transport](#packed-12-bit-transport---packed).
> - **Ein Array, dessen Kameras sich nach einem Neustart von selbst wiederherstellen.** Nach einem Neustart des Switches öffnet das Array von `array-connect`- die Kameras erneut, sobald diese antworten, und streamt wieder, sobald PTP wieder aktiviert ist; in 1.2.0 wurde der Vorgang endgültig beendet und erforderte ein neues `array-connect`. `array-status` meldet `ptp_enabled: False` (und `ptp_off:`), während eine Kamera PTP als „aus“ anzeigt; zuvor wurde die Einstellung gemeldet, mit der das Array verbunden war. Siehe [Persistente Verbindungen](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` zeichnet auf, ohne dass eine GUI geöffnet ist.** Wenn die kombinierte Ansicht der GUI geschlossen ist, rendert das Backend die Ansicht selbst: denNDVI-Wert oder den Index, den Sie mit dem neuen `--expr` übergeben. Eine Aufzeichnung, die kein Bild schreibt, schlägt nun fehl (`no-frames: …`, Beenden von `1`). Zuvor wurde nur aufgezeichnet, solange die kombinierte Ansicht der GUI geöffnet war, und eine Aufzeichnung, die nichts schrieb, beendete dennoch `0`. Siehe [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Eine Kamera, die Chloros bereits hält, wird nicht ein zweites Mal geöffnet.** Ein Befehl, der sie selbst öffnen würde, gibt stattdessen den HTTP-Status 409 `CAMERA_HELD` zurück und nennt dabei, wer sie hält und welcher Befehl sie freigibt. Dies gilt auch für eine Kamera, die gerade von einem anderen Befehl verwendet wird (ein `multi-test` oder `multi-capture`, der seine Kameras öffnet, ein `lattice capture`): Der zweite Befehl wird abgelehnt, bis der erste abgeschlossen ist. Ein erneutes Öffnen könnte den Stream des Inhabers unterbrechen oder das Backend zum Absturz bringen. Siehe [Persistente Verbindungen](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Transiente Mehrkamera-Befehle liefern jede Kamera oder schlagen fehl.** `lattice multi-capture` ohne angeschlossenes Array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` und `index --live` beenden den Vorgang; `1` nennt eine Kamera, die kein Bild geliefert hat. Die Streams überspringen eine Bildgruppe, in der eine Kamera fehlt, und stoppen nach drei aufeinanderfolgenden; `multi-test` gibt pro Trigger eine Zeile aus, `(no frame)`, wenn eine Kamera ein Bild ausgelassen hat; `align-stream` beendet ebenfalls `1` bei einem Auftragsfehler. Jeder Durchlauf versetzt anschließend jede Kamera wieder in den Zustand, in dem sie vorgefunden wurde: ihre Trigger- und Sync-Line-Verkabelung, ihre PTP (`PtpEnable`) sowie alle Kameraeinstellungen, die der Durchlauf geändert hat – Belichtung und Verstärkung, AE-Ziel, Grenzwerte und Dämpfung, Weißabgleich, Schwarzpegel, Pixelformat, Binning und ROI, Spiegelungen, Bildrate –, es sei denn, `--keep-settings` / `--clear-settings` (siehe [Kameraspeicher](#camera-memory----keep-settings----clear-settings)). Zuvor behielten diese Befehle alle eingehenden Bilder bei und beendeten `0` (`align-stream` gab einen Auftragsfehler aus als `Note:`), und ließen jede Kamera in `TriggerMode` `On`. Außerdem stellten sie die Belichtung jeder Kamera anhand der Szene vor dem Speichern eines Bildes ein (höchstens einige Sekunden), wobei ein angeschlossenes Array jede Kamera auf folgende Werte startet: 40 Multispektral, 60 RGB (`--brightness N` legt dies fest) und behält diese Werte für den gesamten Durchlauf bei; ebenso verhält es sich mit einem einzelnen `lattice capture` einer Kamera, die Chloros nicht hält. Zuvor wurde das erste Bild jeder Kamera bei der kameraspezifischen Belichtungsgrenze von etwa 100 ms aufgenommen, unabhängig von den Lichtverhältnissen, und war oft überbelichtet. Siehe [Multi-Kamera (Transient-Sitzungen)](#multi-camera-transient-sessions).
> - **Transient-Multi-Kamera-Befehle teilen sich das Netzwerk auf die gleiche Weise wie `array-connect` und öffnen nur `--serials`.** Jede Kamera wird an ihren Anteil an der Host-Verbindung angepasst, sodass vier Kameras über eine 1-Gbit/s-Verbindung laufen; zuvor forderten die vier zusammen 125 MB/s an, alles über 1-GbE, und bei jedem Frame gingen Pakete verloren. Ein Array, das das Netzwerk nicht übertragen kann, wird abgelehnt, bevor eine Kamera mit dem Streamen beginnt. `--serials` wählt nun die Kameras aus, wobei die erste den Trigger auslöst; zuvor öffnete jeder Befehl alle Kameras im Netzwerk, unabhängig davon, was `--serials` angab. Ein Fehler, bei dem es sich um einen GVSP-Paketverlust handelt, wird nun als solcher angezeigt, anstatt auf das M8-Synchronisationskabel hinzuweisen. Siehe [Multi-Kamera (vorübergehende Sitzungen)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` lässt die Kameras im freien Betrieb, wie vorgefunden.** `TriggerMode`, `Off` und alles andere so, wie es bei der Verbindung festgestellt wurde – die Trigger- und Sync-Leitungs-Einstellungen (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP sowie die von der Sitzung geänderten Kameraeinstellungen – sodass ein einzigerKamerabefehl für ein ehemaliges Mitglied ruft alle Bilder auf einmal ab, und der Master bleibt nicht mehr auf `TriggerSource`, `Software`. `--keep-settings` speichert die Einstellungen der Sitzung im Speicher jeder einzelnen Kamera, `--clear-settings` setzt diesen Speicher zurück (siehe [Kameraspeicher](#camera-memory----keep-settings----clear-settings)). `lattice continuous` stellt außerdem eine Kamera, die es als aktiviert erkennt, für den Lauf auf „Free-Run“ ein (und stellt sie danach wieder zurück), und ein Lauf ohne Frame beendet `1`.
> - **`lattice configure --get` / `lattice exposure`** geben einen `note:` mit dertatsächlichen Belichtung oder Verstärkung der Kamera aus, während deren Automatikmodus läuft. Ein abgelehntes `lattice exposure --set` / `configure --set` gibt den von der Kamera akzeptierten Bereich an und schaltet den Automatikmodus wieder ein, anstatt die automatische Belichtung ausgeschaltet zu lassen. Siehe [Sensor-Tuning](#sensor-tuning).
> - **Neue Kamerasteuerungen:**`lattice highlights` (hellweiße Oberflächen in der RGB-Anzeige abdunkeln) und `lattice exposure --brightness-auto`.**`lattice multi-detect`** ermittelt die verkabelten Kameras in einem Abtastrundenlauf und legt die RGB-Kamera standardmäßig als Master fest.
> - **Neue DAQ-Befehle:** `daq pool-bind`; `pool-enable-ptp`; sowie für die eigenen Einstellungen eines DAQ-E: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` und `pool-update-firmware`. Siehe [DAQ-E-Geräteeinstellungen](#daq-e-device-settings).
> - **Ein DAQ-E, das keine Daten mehr liefert, wird vom Backend erneut verbunden.** Wenn der Datenstrom eines im Pool befindlichen DAQ-E ausfällt (z. B. durch einen Neustart des Switches, der es mit Strom versorgt, eine unterbrochene Kameraverbindung), stellt das Backend die Verbindung mit den bisherigen Einstellungen wieder her und versucht dies in zunehmenden Intervallen erneut, solange es nicht erreichbar ist. Eine darauf laufende Aufzeichnung wird im selben `.daq` fortgesetzt, wobei die Lücke in der Datei markiert wird, und `pool-record --stop` listet jede Unterbrechung auf. In Version 1.2.0 blieb der Sensor so lange still, bis `pool-disconnect` + `pool-connect` aufgerufen wurden, wodurch auch die Aufzeichnung beendet wurde. Siehe [Ein DAQ-E, der keine Daten mehr liefert](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** meldet, ob die Pakete des PTP-Grandmasters das Netzwerk erreichen (`health`); wenn dies nicht der Fall ist, wird ein `WARNING:` auf stderr ausgegeben. Siehe [`time-sync`](#chloros-cli-time-sync).
> - **Linux-`.deb`-Paket:** Es vergrößert die vom Kernel verwendeten Socket-Empfangspuffer, die für Kamerastreams benötigt werden (bei der Kernel-Standardeinstellung gehen Bilder und Bildrate verloren), fügt einen Backend-Dienst hinzu, der unter Ihrem Benutzerkonto läuft, und `lattice network` überprüft die Puffer. Die CLI stoppt niemals ein Backend, das als Dienst läuft, und bei einem Upgrade werden die aktivierten Backend-Dienste erneut gestartet. Siehe [Ausführen des Backends als Dienst](#running-the-backend-as-a-service-headless) und [Linux-Hosts – Socket-Empfangspuffer](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` gibt an, wann keine Ausrichtung möglich war:**Jeder Grund, aus dem eine Erfassung nicht ausgerichtet gespeichert wurde (kein Ausrichtungsprofil, Elemente mit unterschiedlichen Auflösungen, ein Array, das nicht kabel-synchronisiert ist, ein Frame, auf den das Profil nicht abgebildet werden kann) wird als `Alignment:`-Zeile ausgegeben.**`chloros-cli daq help`** ist `daq --help` (es wurde mit `2` beendet), und unter Linux listet die Hilfe die direkten Hardware-Befehle `daq` auf, auf, die in diesem Build enthalten sind.
> - **Jetson:** Das Paket ist `chloros_1.3.0_arm64_jp6.deb` für JetPack 6, und die GPU-Verarbeitung läuft auf dem Orin mit JetPacks CUDA-PyTorch. Siehe [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konventionen

- Alle Befehle haben das Präfix `chloros-cli`. Unter Windows lautet die Binärdatei `chloros-cli.exe`; unter Linux/Jetson lautet sie `chloros-cli`.
- Optionale Argumente werden als `--flag` dargestellt. Erforderliche Positionsargumente werden ohne Klammern dargestellt.
- Wenn ein Standardwert angegeben ist, wird dieser verwendet, wenn das Flag weggelassen wird.
- Die CLI ist ein schlanker HTTP-Client über das Chloros-Backend (der Server auf `127.0.0.1:5000`, den auch die Desktop-App nutzt). **Welche Befehle starten sie:**
  - Die Kernbefehle – `process`, `login`, `logout`, `status`, `export-status`, `time-sync` und `selftest` – nutzen ein bereits laufendes Backend oder starten eines, wenn keines läuft, und beenden es wieder, sobald sie fertig sind.
  - **Die Befehle `lattice`, `project` und `daq` starten niemals ein Backend.** Sie benötigen ein bereits laufendes Backend: das der Windows-Desktop-App, den Linux-[Backend-Dienst](#running-the-backend-as-a-service-headless) oder eines, das Sie selbst starten (siehe [Backend starten](#start-the-backend)). Ohne ein solches Backend geben sie „`Error: Chloros backend not reachable at http://127.0.0.1:5000`“ zusammen mit dem Befehl aus, der es startet, und beenden sich mit „`1`“.
- `CHLOROS_BACKEND_URL=<url>` verweist die Befehlsfamilien **`lattice`**,**`project`**und**`daq pool-*`** Befehlsfamilien auf ein anderes Backend (`lattice network-analysis` verweist stattdessen auf `--backend-url`). Die Kernbefehle fixieren `http://127.0.0.1:<port>` bewusst und ignorieren es (das IPv4-Literal vermeidet die von Windows verursachte Verzögerung von etwa 2 Sekunden pro Anfrage bei der Umwandlung von `localhost` in `::1`). Siehe [Umgebungsvariablen](#environment-variables).
- **Relative Pfade werden relativ zu dem Ordner aufgelöst, aus dem Sie `chloros-cli` ausführen.** Das Backend ist der Prozess, der die Ausgabe- und Eingabeordner öffnet; daher wandelt die CLI, bevor sie einen Pfad übermittelt, diesen in einen absoluten Pfad um: `-o` / `--output` (einschließlich dessen Standardwert, `output`), `--input`, `--burst-dir`, `--profile`, `--save` und `--mask` sowie `output` (Standard: `output`) eines [Rezept](#recipe-dsl)-Schrittssowie der Standardordner `output` von `lattice index --live`. `-o captures/` schreibt daher in `<current folder>/captures/`, und die ausgegebenen Pfade sind absolut. `~` wird aufgelöst. Bei einem Remote-Backend (`CHLOROS_BACKEND_URL`, das einen anderen Rechner bezeichnet) werden Pfade so gesendet, wie sie eingegeben wurden, und auf diesem Rechner aufgelöst.
- Für alle SDK- und CLI-Aufrufe ist eine Anmeldung mit einem Chloros+-Konto erforderlich Aufrufe erforderlich (`chloros-cli login` einmal pro Rechner ausführen; wird in `~/.chloros/` zwischengespeichert).
- Die Beispiele verwenden Linux-Pfade; unter Windows ersetzen Sie `/home/user/...` durch `C:/Users/.../...`. Die serielle Schnittstelle eines DAQ-Userielle Schnittstelle hat unter Windows das Format `COM3` und unter Linux das Format `/dev/ttyUSB0` bzw. `/dev/ttyACM0`.
- In den Synopsen werden sich gegenseitig ausschließende Flags als `{--a | --b}` und optionale Flags als `[...]`.

---

## Synopsis der obersten Ebene

```
chloros-cli [global options] COMMAND [command options]
```

### Globale Optionen

| Flag | Beschreibung |
| --- | --- |
| `--backend-exe PATH` | Die automatisch erkannte Backend-Ausführungsdatei überschreiben. |
| `--port N` | HTTP-Port des Backends (Standard: `5000`). Wird nur von den Kernbefehlen verwendet (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` und `daq` verwenden `CHLOROS_BACKEND_URL` (Standard: `http://127.0.0.1:5000`). |
| `-v, --verbose` | Ausführliche Ausgabe aktivieren. |
| `--restart` | Erzwingt einen Neustart des Backends (beendet alle laufenden `backend_server.py`-Prozesse). Ein Backend, das von einem systemd-Dienst (Linux .deb) wird niemals gestoppt: Der Befehl gibt stattdessen `sudo systemctl restart <unit>` aus und beendet sich mit `1`. |
| `--version` | Version anzeigen (`Chloros CLI 1.3.0`). |
| `--help` | Hilfe der obersten Ebene anzeigen. |

### Befehlsverzeichnis

| Befehl | Zweck |
| --- | --- |
| [`process`](#chloros-cli-process) | Einen Ordner mit Survey3- oder „LATTICE“-Aufzeichnungen durchgängig verarbeiten. |
| [`login`](#chloros-cli-login) | Diesen Rechner mit einem Chloros+-Konto authentifizieren. |
| [`logout`](#chloros-cli-logout) | Zwischengespeicherte Anmeldedaten löschen. |
| [`status`](#chloros-cli-status) | Aktuellen Lizenz-/Authentifizierungsstatus anzeigen. |
| [`export-status`](#chloros-cli-export-status) | Live-Fortschrittsanzeige des Thread-4-Exports während eines `process`-Laufs. |
| [`language`](#chloros-cli-language) | CLI-Anzeigesprache festlegen oder auflisten (38 Sprachen werden unterstützt). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Standard-Projektordner (gemeinsam mit der GUI genutzt). |
| [`update`](#chloros-cli-update) | Nach Chloros-Updates suchen und diese installieren (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Systemdiagnose + Smoke-Tests. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-Grandmaster-Status / -Steuerung. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-Kamerasteuerung und -Aufnahme (über 45 Unterbefehle). |
| [`daq`](#chloros-cli-daq) | Steuerung der DAQ-Spektralsensoren (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Ein gespeichertes Chloros-Projekt (Kameras + DAQs) öffnen und ausführen. |

---

## Installation

`chloros-cli` ist auf jeder unterstützten Plattform im Chloros-Desktop-Installationsprogramm enthalten – es gibt keinen separaten CLI-Download. Durch die Installation des Plattformpakets wird `chloros-cli` neben der Desktop-App und der von ihr gesteuerten Backend-Binärdatei zu Ihrem `PATH` hinzugefügt.

Aktuelle Downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Das Installationsprogramm enthält außerdem praktische Startskripte (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), die eine einsatzbereite CLI-Shell öffnen.

### Windows (.exe)

1. Laden Sie das Windows-Installationsprogramm von der Download-Seite herunter.
2. Führen Sie `Chloros Installer 1.3.0.exe` aus und folgen Sie den Anweisungen des Assistenten. Der Standardinstallationspfad ist `C:\Program Files\Chloros\` (die CLI-Anwendung wird unter `C:\Program Files\Chloros\cli\` abgelegt, das der Installer zur PATH-Variablen hinzufügt).
3. Öffnen Sie ein neues Terminal (`cmd.exe`, PowerShell oder Windows-Terminal), damit das aktualisierte `PATH` erkannt wird.

```powershell
chloros-cli --version
```

Das Installationsprogramm fügt `chloros-cli.exe` automatisch zu Ihrem Systemordner `PATH` hinzu und bündelt die LATTICE-Kameratreiber. Das Backend ist `C:\Program Files\Chloros\backend\chloros-backend.exe` (der Ordner „`backend`“ in Ihrem Installationsordner), das auf `PATH` nicht vorhanden ist.

**Systemeinstellungen.** Die Seite *„Zusätzliche Aufgaben auswählen“* des Installationsprogramms enthält unter **Systemeinstellungen**zwei Optionen, die beide**standardmäßig aktiviert** sind:

| Option (wie im Installationsprogramm formuliert) | Funktion | Deaktivierung der unbeaufsichtigten Installation |
| --- | --- | --- |
| **Lange Windows-Pfade aktivieren** (ermöglicht Projektordner, die tiefer als 260 Zeichen liegen; empfohlen) | Setzt den systemweiten Windows-Wert „`LongPathsEnabled`“ auf 1. Windows wird standardmäßig mit deaktivierter Einstellung ausgeliefert, und dann darf kein Pfad länger als 260 Zeichen sein; Chloros-Produkte befinden sich bis zu etwa 130 Zeichen unterhalb des Projektordners, sodass ein tiefer liegender Projektordner abgelehnt wird (siehe [Fehlerbehebung](#troubleshooting-pointers)). Tritt für Programme in Kraft, die nach der Installation gestartet werden. Bleibt bei der Deinstallation aktiviert, da andere Programme möglicherweise darauf angewiesen sind. | `/MERGETASKS="!longpaths"` |
| **Das Chloros-Backend vom Microsoft Defender-Scan ausschließen**(deutlich schnellere Kameraaufnahmen; empfohlen) | Fügt eine Defender-**Prozess**-Ausnahme für `chloros-backend.exe` im Installationsordner hinzu, sodass der Echtzeitschutz nicht jede Datei scannt, die das Backend schreibt, unabhängig davon, wohin es sie schreibt. Es wird kein anderes Programm ausgeschlossen; die Ausnahme wird bei der Deinstallation entfernt. Wenn Defender nicht vorhanden ist oder durch Richtlinien gesperrt ist, schlägt der Schritt stillschweigend fehl und die Installation wird fortgesetzt. | `/MERGETASKS="!defenderexclusion"` |**Warum die Defender-Option wichtig ist:** Bei hohen Erfassungsraten kostet das Echtzeit-Scannen der Erfassungsdateien etwa 30 % der Erfassungsrate (gemessen bei einer kontinuierlichen Erfassung mit 9 Kameras: 77,7 Aufnahmen/s ausgeschlossen, 52,7 gescannt). `lattice array-capture --continuous` gibt eine Meldung aus, wenn es feststellt, dass Defender einen oder mehrere CPU-Kerne für die Aufnahme beansprucht; führen Sie das Installationsprogramm erneut aus, wobei die Option aktiviert ist.

Bei einer automatischen Installation (`/SILENT` oder `/VERYSILENT`) werden beide Optionen angewendet, sofern sie nicht deaktiviert wurden, z. B. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Windows-Firewall.** Das Installationsprogramm fügt in jedem Netzwerkprofil stets zwei eingehende Regeln für `chloros-backend.exe` hinzu, und das Deinstallationsprogramm entfernt diese: `Chloros Backend (UDP-In)` für Kamerastreams, PTP, mDNS und den DAQ-E-Datenstrom sowie `Chloros Backend (TCP-In)`, beschränkt auf das lokale Subnetz, für ein DAQ-E-Firmware-Update, bei dem der Sensor eine Verbindung zu diesem Computer herstellt, um das Bild abzurufen (siehe [DAQ-E-Geräteeinstellungen](#daq-e-device-settings)). Die HTTP-API des Backends bleibt an `127.0.0.1` gebunden.

### Linux amd64 (.deb)

Für Ubuntu 22.04 LTS oder neuer / Debian-basierte x86_64-Workstations.

> **Ubuntu 20.04 wird nicht unterstützt.** Die Abhängigkeitsliste des Pakets leitet sich davon ab,
> womit das Backend tatsächlich verknüpft ist, und dazu gehört auch `libc6 (>= 2.35)`;
> Focal liefert glibc 2.31 aus. `apt` verweigert die Installation, anstatt sie zur
> Laufzeit scheitern zu lassen. Die Liste nennt auch `libgl1`, das vom mitgelieferten OpenCV geladen wird und in einem minimalen
> Server-Image möglicherweise fehlt; `apt install` stellt es bereit.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

Das .deb installiert:
- `chloros-cli` bis `/usr/bin/chloros-cli` sowie `chloros-backend` (startet das Backend) bis `/usr/bin/chloros-backend`
- Das kompilierte Backend und die CLI werden entpackt in `/usr/lib/chloros/backend/` und `/usr/lib/chloros/cli/`
- Die LATTICE-Kameratreiber
- Denoiser-Modelle, Kalibrierungs-Bundles und die Konfiguration des Update-Kanals
- Das Python-SDK für das System `python3` (`import chloros_sdk`). `apt install` bringt zudem seine eigenen Voraussetzungen mit: `python3-numpy` und `python3-requests` (empfohlen, aber nicht erforderlich: Die oben genannten Programme sind eigenständig; `dpkg -i` überspringt diese, und die Installationsausgabe nennt, was fehlt). In einer virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
– `/etc/sysctl.d/60-chloros-gige.conf`, das die Socket-Empfangspuffer des Kernels für Kamerastreams Streams auf 32 MB erhöht und diese sofort anwendet (ein bereits höher eingestellter Wert bleibt unverändert). Durch das Entfernen des Pakets wird die Datei entfernt. Siehe [Linux-Hosts – Socket-Empfangspuffer](#linux-hosts--socket-receive-buffers).
- Zwei optionale systemd-Dienste für das Backend, von denen keiner aktiviert ist (nächster Abschnitt)

#### Das Backend als Dienst ausführen (headless)

Ohne einen Dienst läuft das Backend (und damit der PTP-Grandmaster) nur, solange es von etwas gestartet wurde: `chloros-backend` in einem Terminal, ein SDK-Skript oder ein core-`chloros-cli`-Befehl für die Dauer seiner Ausführung. Für ein ständig aktives Backend auf einem headless-Rechner aktivieren Sie den Dienst **als Ihr Benutzer**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` führt das Backend als diesen Benutzer aus. Die von ihm geschriebenen Aufzeichnungen gehören diesem Benutzer, sein detailliertes Protokoll befindet sich in dessen `~/.cache/chloros/logs`, und es verwendet dessen `chloros-cli login` (`~/.chloros`); melden Sie sich daher zunächst einmal als dieser Benutzer an. PTP benötigt keine Root-Rechte. Wie jedes installierte Backend lauscht es ausschließlich auf `127.0.0.1:5000`.

`chloros-backend.service` führt das Backend als **root** aus. Es ist weiterhin so installiert, dass ein Rechner, auf dem es aktiviert ist, weiterläuft, aber alles, was ein Root-Backend schreibt, gehört zu Root, sodass ein Benutzer seine eigenen Aufzeichnungen nicht löschen kann. Um zu Ihrem Benutzer zu wechseln:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Das detaillierte Protokoll des Root-Dienstes befindet sich in `/var/log/chloros` und ist für Mitglieder der Gruppe `adm` lesbar (der Dienst setzt `CHLOROS_LOG_DIR`; siehe [Umgebungsvariablen](#environment-variables)). Bei der Installation des Pakets wird der Befehl `enable` mit Ihrem Benutzernamen ausgegeben, ebenso der Schalter, wenn der Root-Dienst aktiviert ist.

**Aktivieren Sie eine Backend-Einheit.** Es läuft jeweils nur eine Backend-Einheit. Eine zweite aktivierte Einheit (`chloros-backend.service` neben `chloros-backend@<user>.service` oder zwei Benutzer-Einheiten `@`) lässt sich nicht starten und bleibt bei `failed` mit dem Exit-Status 98 stehen; systemd startet sie nicht neu. Die Ausgabedatei „`systemctl status`“ gibt den Grund an: „`Another instance is already running. Exiting.`“, gefolgt von dem, was das Backend blockiert (Port 5000 oder die Sperrdatei und der zugehörige Benutzer), dann `systemd will not restart <unit> (exit status 98)`. Nach einem Neustart kann jede der beiden Einheiten diejenige sein, die läuft. Um zu wechseln, deaktivieren Sie diejenige, die Sie nicht verwenden, wie oben beschrieben.**Die CLI stoppt niemals das Backend eines Dienstes.** `chloros-cli` nutzt es so, wie es ist, ebenso wie die GUI und das SDK: `process` läuft darauf und lädt sein eigenes Projekt anstelle des letzten. Wo die CLI andernfalls das Backend neu starten würde (`--restart` oder ein `process`, dessen Import nichts gefunden hat), benennt sie die Einheit, gibt „`sudo systemctl restart <unit>`“ aus und beendet „`1`“, da ein auf diese Weise gestopptes Backend inaktiv bleiben würde. `GET /api/status` benennt die Einheit als `service` (`null` für ein Backend, auf dem kein Dienst läuft).**Upgrade und Entfernung.** Bei der Aktualisierung des Pakets werden alle Backend-Dienste angehalten und anschließend alle aktivierten Dienste neu gestartet, wobei die `@<user>`-Einheiten zuerst gestartet werden (eine manuell gestartete, aber nie aktivierte Einheit bleibt angehalten, wie nach einem Neustart). Sind mehrere aktiviert, läuft nur einer, und die anderen bleiben wie oben beschrieben im Fehlerzustand; das Upgrade gibt „`Chloros: N backend units are enabled: …`“ und „`Keep one enabled: sudo systemctl disable --now <unit>`“ aus. Durch das Entfernen werden sie gestoppt und deaktiviert. Durch das Löschen wird auch „`/var/log/chloros`“ entfernt.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Gleicher Aufbau wie das amd64-.deb, einschließlich Empfangspuffer und [Backend-Dienste](#running-the-backend-as-a-service-headless) für Jetson Orin / Orin NX / Orin Nano auf **JetPack 6** (L4T R36). Das enthaltene PyTorch ist der CUDA-Build von JetPack 6 für Orin, sodass die GPU-Verarbeitung – einschließlich des texturbasierten Debayers – auf der GPU des Jetson läuft. Das Paket enthält CUDA selbst nicht: Es nutzt die von JetPack 6 installierten CUDA- und cuDNN-Bibliotheken (`sudo apt install nvidia-jetpack` auf einem Image, auf dem diese fehlen). Wenn keine CUDA-GPU verfügbar ist, läuft die Verarbeitung auf der CPU, und „texture-aware“ verwendet den Standard-Debayer (siehe [`process`](#chloros-cli-process)).

### Einmalige Authentifizierung pro Rechner

Jede Plattform erfordert eine einmalige Chloros+-Anmeldung, bevor SDK- und CLI-Aufrufe funktionieren:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Die Anmeldedaten werden in `~/.chloros/user_session.json` zwischengespeichert.

### Backend starten

Die Befehle `lattice`, `project` und `daq` kommunizieren mit einem bereits laufenden Backend; sie starten niemals eines (siehe [Konventionen](#conventions)). Lassen Sie eines davon laufen, während Sie diese Befehle verwenden:

| Plattform | Vorgehensweise |
| --- | --- |
| Windows | Öffnen Sie die Chloros-Desktop-App (ihr Backend bedient auch die CLI) oder führen Sie `"C:\Program Files\Chloros\backend\chloros-backend.exe"` in einem eigenen Terminal aus (im Ordner `backend` Ihres Installationsordners) und lassen Sie es geöffnet. |
| Linux/Jetson | Führen Sie „`chloros-backend`“ in einem eigenen Terminal aus oder lassen Sie den [Backend-Dienst](#running-the-backend-as-a-service-headless) ständig laufen. |

Es läuft jeweils nur ein Backend; alle Clients (die GUI, die CLI, SDK-Skripte) nutzen dieses gemeinsam. Auch die Kernbefehle (`process`, `login`, `status`, …) nutzen es und starten eines, wenn noch keines läuft.

**Ein Backend läuft zwischen den Befehlen weiter**, sodass der nächste Befehl keine erneute Startphase durchläuft. Es wird beendet, wenn Sie sich abmelden (`chloros-cli logout`, es sei denn, die Chloros-App ist geöffnet oder ein Lauf bzw. ein Offline-Build ist im Gange), wenn Sie `chloros-cli backend stop` ausführen oder von selbst nach 30 Minuten, in denen es nicht genutzt wird: kein Befehl, Skript oder keine App, kein Lauf oder Offline-Build, keine verbundene Kamera oder kein Lichtsensor. `backend stop` lehnt den Befehl ab, solange ein Lauf oder ein Offline-Build läuft, und nennt diesen; `--force` beendet ihn sofort, und dieser Lauf oder Build geht verloren (eine Bildserie bleibt erhalten, und `array-build-video` erstellt sie erneut). Setzen Sie `CHLOROS_BACKEND_IDLE_MIN`, bevor das Backend startet, um die 30 Minuten zu ändern (`0`: niemals); der [Backend-Dienst](#running-the-backend-as-a-service-headless) stoppt niemals von selbst. `chloros-cli backend status` zeigt an, ob einer läuft, was ihn nutzt und ob ein Offline-Build läuft. `process` öffnet sein Projekt im laufenden Backend, es sei denn, die Chloros-App ist mit einem Projekt geöffnet, das sie geöffnet wurde: In diesem Fall wird der Vorgang abgelehnt, und Sie müssen den Vorgang über die App ausführen oder dieses Projekt zuerst schließen. Ein Projekt, das von einem früheren `process` oder einem SDK-Skript offen gelassen wurde, wird einfach ersetzt.

### Installation überprüfen

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` beendet `0` erst, wenn alle sieben Prüfungen bestanden sind. Die letzte Prüfung erfordert eine CUDA-GPU (eine NVIDIA-GPU, oder eines Jetson Orin) sowie die Denoiser-Modelle; auf einem Computer ohne GPU wird daher `Failed: CUDA + Denoiser` angezeigt und das Programm beendet `1`; dort funktioniert weiterhin alles außer dem ausschließlich GPU-basierten, texturbasierten Debayer.

> **Chloros+-Abonnement erforderlich.**Die CLI erfordert einen aktiven Chloros+-Tarif.**Copper**ist die Einstiegsstufe von Chloros+ – jede kostenpflichtige Chloros+-Stufe bietet Zugriff auf CLI und SDK; nur die kostenlose Stufe**Iron** bietet diesen Zugriff nicht. (Zuordnung der Tarif-IDs: `0`=Iron/kostenlos, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Upgrade unter [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Diese Untergrenze wird vom Backend durchgesetzt, nicht nur von der CLI: Eine mit SDK/CLI gekennzeichnete Anfrage ohne bezahlten Tarif wird mit dem Fehlercode `403 PLAN_UPGRADE_REQUIRED` abgelehnt, unabhängig davon, ob sie von `chloros-cli`, dem Python-SDK oder einem selbst entwickelten HTTP-Client stammt. Ein abgemeldeter Aufrufer erhält stattdessen den Fehlercode `401 AUTH_REQUIRED`. Der Zugriff funktioniert offline während der Nachfrist des Tarifs (30 Tage bei monatlicher Abrechnung, bis zum Ablauf bei jährlicher Abrechnung) und wird nach deren Ablauf unterbrochen; `chloros-cli status` funktioniert weiterhin, sodass der Grund sichtbar bleibt (es handelt sich um die einzige SDK-/CLI-Route, die von der Tier-Beschränkung ausgenommen ist – `GET /api/license-status`).

---

## Fünf-Minuten-Schnellstart

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (Schritt 2) startet ein Backend für den eigenen Durchlauf, wenn noch keines läuft, und beendet es nach Abschluss; starten Sie daher ab Schritt 4 eines. Siehe [Backend starten](#start-the-backend).

---

## `chloros-cli process`

Verarbeiten Sie einen Ordner mit Bildern durch die vollständige Chloros-Pipeline (Zielerkennung → Kalibrierung → Vignette → Reflexionsgrad → Index-Export).

### Übersicht

```
chloros-cli process INPUT [OPTIONS]
```

### Positionsargumente

| Argument | Beschreibung |
| --- | --- |
| `INPUT` | Pfad zum Eingabeordner, der Dateien vom Typ `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) oder `.dng` enthält. |

### Allgemeine Optionen

| Option | Standardwert | Beschreibung |
| --- | --- | --- |
| `-o, --output PATH` | Ein neuer Ordner mit dem Namen `-n` im Standard-Projektordner | Der Projektordner. Das Projekt wird danach benannt, und ein Ordner, der bereits eine Datei mit dem Namen `project.json` enthält, erhält eine Datei mit dem Namen `_1` / `_2`-Bruderordner, anstatt überschrieben zu werden. Der Standard-Projektordner ist derjenige, den `set-project-folder` (oder die GUI) ausgewählt hat, andernfalls `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | auto (Zeitstempel) | Projektname, wird nur **ohne** `-o` verwendet (mit `-o` übernimmt das Projekt den Namen des Ordners). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` verwendet einen „Chloros+“-Neural-Debayer für Survey3-Bilder; langsamer, aber höhere Qualität. Dies gilt nur für Survey3: LATTICE-Aufnahmen verwenden immer das Standard-Demosaicing; daher lehnt `process` bei einem Ordner mit LATTICE-Aufnahmen `texture-aware` nach dem Import ab, gibt dabei die Anzahl der betroffenen Dateien als Fehlermeldung an und beendet `1`, bevor überhaupt etwas verarbeitet wird (die Chloros-App bietet für diese Aufnahmen ebenfalls keine „Texture Aware“-Funktion an). Es wird eine CUDA-GPU benötigt (eine NVIDIA-GPU unter Windows/Linux x64, die Orin-GPU auf einem Jetson): Ohne eine solche verwendet der Lauf `standard`, und das Backend protokolliert eine Zeile `[DEBAYER] … needs a CUDA GPU …`, in der der Grund dafür angegeben wird (dargestellt mit `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Vignettierungskorrektur. |
| `--reflectance / --no-reflectance` | `--reflectance` | Reflektanzkalibrierung (verwendet Panel-Target, falls vorhanden, NIST-Kalibrierung pro Seriennummer für LATTICE). Bei LATTICE-Multispektralbildern dient dies gleichzeitig als Schalter für das Reflektanz-**Produkt** – siehe [Export-Schalter pro Produkt](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | aus | PPK-GNSS-Korrekturen aus Sidecar-Dateien anwenden. |
| `--exposure-pin-1 CAMERA_MODEL` | keine | Das an den Belichtungs-Pin 1 angeschlossene Kameramodell, für die PPK-Zeitsynchronisation. |
| `--exposure-pin-2 CAMERA_MODEL` | keine | Das an den Belichtungs-Pin 2 angeschlossene Kameramodell für die PPK-Zeitsynchronisation. |
| `--recal-interval SECONDS` | `0` | Minimales Neukalibrierungsintervall in Sekunden (das *Minimale Neukalibrierungsintervall* des Projekts). |
| `--timezone-offset HOURS` | `0` | Zeitzonenversatz des Lichtsensors gegenüber UTC in Stunden (der *Zeitzonenversatz des Lichtsensors* des Projekts). |
| `--format FORMAT` | `TIFF (16-bit)` | Einer der folgenden Werte: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros erhöht niemals die Bittiefe eines Bildes: Bei reinen 8-Bit-Bildern wird ein Format mit höherer Bittiefe als `TIFF (8-bit)` geschrieben, und dies wird im Programmablauf einmalig erwähnt. In einem Projekt, das 8-Bit- und 12/16-Bit-Aufnahmen mischt, wird jede Aufnahme mit ihrer eigenen Farbtiefe oder einer niedrigeren exportiert. Radiance ist immer 32-Bit-Float. |
| `--indices NAME [NAME ...]` | keine | Vegetationsindizes nach Voreinstellungsname (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), Groß- und Kleinschreibung spielt keine Rolle. Jeder Index wird dem Projekt für die Bänder der jeweiligen Kamera hinzugefügt, wobei ein neues Projekt einbezogen wird. Ein Name, der keine Voreinstellung ist oder für den die Bänder der Kamera nicht ausreichen (`NDRE` benötigt RE und NIR, was keine einzelne Kamera bereitstellt), wird in einer Warnung genannt und erhält kein Indexbild. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Erzwingt den Pipeline-Einstiegspunkt für LATTICE-TIFFs (Survey3.raw ist davon nicht betroffen). Außerdem die Notlösung, mit der eine Aufnahme ohne **Raw-Datei** überhaupt verarbeitet werden kann – siehe [So sieht ein Aufnahmordner aus](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | ein | Das linear debayerte Produkt ausgeben (`Debayered_Images`). Siehe [Export-Schalter pro Produkt](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | ein | Gibt die Anzeigevorschau aus (`Preview_Images`): RGB = Weißabgleich (DAQ-Lichtart, falls verfügbar, andernfalls „Gray-World“) + Gamma; Multispektral = Falschfarben-Streckung. |
| `--radiance / --no-radiance` | ein | Ausgabe der Strahlungsintensität als float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referenz für das LATTICE-Reflexionsprodukt: `auto` = Das QA-konforme Ziel innerhalb des Bildausschnitts ist die absolute Referenz (ein Ziel mit QA-Flag konkurriert mit dem DAQ hinsichtlich des gemessenen Werts), DAQ-Downwelling (ρ = π·L/E) als Fallback; `target` = streng (keine DAQ-Ersetzung); `daq` = DAQ-maßgeblich. Siehe [Export-Schalter pro Produkt](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | keine | Verzeichnis der **gemessenen** Zielreflexions-Scans pro Einheit (`<serial>.csv`); greift bei Fehlen auf die nominalen T3/T4P-Spektren zurück. |
| `--array-alignment / --no-array-alignment` | aktiviert | LATTICE-Arrays: Wendet die in der `Chloros:Alignment*`-XMP-Datei jeder Aufnahme vermerkte Modul-zu-Modul-Ausrichtung auf jedes verarbeitete Produkt an (Debayering / Vorschau / Strahlungsintensität / Reflektanz / Index). Keine Aktion für Bilder ohne diese Tags. |
| `--array-alignment-crop / --no-array-alignment-crop` | Zuschneiden | Schneidet die ausgerichteten Exporte auf den gemeinsamen Überlappungsbereich des Arrays zu, sodass alle Module denselben Fußabdruck haben; `--no-…` behält die volle Sensorfläche bei (schwarze Füllung außerhalb der Quelle). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Neuberechnung für die Ausrichtungsverzerrung. `nearest` bewahrt die exakten Quell-DNs bei (keine Vermischung radiometrischer Werte zwischen den Pixeln). |

### Optionen zur Zielerkennung

| Flag | Beschreibung |
| --- | --- |
| `--min-target-size PIXELS` | Minimale Größe des Kalibrierungsziels in Pixeln für den Detektor. |
| `--target-clustering 0-100` | Prozentsatz der Zielbündelung (Standard: `60`). |
| `--target / --targets` | Suche nach Kalibrierungszielen nur in einem `target`- oder `targets`-Unterordner der Eingabedatei (schneller). |

### Beispiele

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Export-Schalter pro Produkt (LATTICE Multispektral)

Die LATTICE-Verarbeitung verzweigt sich **in einem Durchgang auf jedes anwendbare Produkt**. Die vier Schalter pro Typ – `--debayered`, `--preview`, `--radiance`, `--reflectance` – sind**standardmäßig aktiviert**; verwenden Sie das Formular `--no-<type>`, um eine Option zu deaktivieren. RGB-Masterkameras geben ausschließlich debayerte Daten + Vorschau aus (keine bandweisen Strahlungs- oder Reflexionswerte), daher sind `--radiance`/`--reflectance` für sie ohne Auswirkung. Die Schalter werden für Survey3-`.raw` (das dem Standardpfad für Reflektanz/Ziel folgt) ignoriert. *(Das alte `--radiometric-output {reflectance,radiance,sensor-response}`-Flag wurde **entfernt** und durch diese Schalter ersetzt; es gibt keine `sensor-response`-Ebene mehr.)*

| Produkt | Ausgabe | DAQ-Downwelling erforderlich? |
| --- | --- | --- |
| `--debayered` | Lineares Demosaicing (`Debayered_Images`): dieselbe Datei, die eine Aufnahme auf ihrer `debayered`-Stufe speichert — der Dunkeloffset (DSNU) des Kalibrierungspakets wird vom Rohdatenwert abgezogen, anschließend erfolgt das kantensensitive Demosaicing mit OpenCV; keine Linearitäts-LUT, kein Flat-Field und kein Unmix. | Nein. |
| `--preview` | Vorschau anzeigen (`Preview_Images`): RGB = Weißabgleich + Gamma; Multispektral = Falschfarben-Streckung. | Nein. |
| `--radiance` | float32 W/m²/sr/nm aus der vollständigen radiometrischen Kette (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 Reflexionsgrad ρ (`32768` = 1,0), Pix4D-kompatibel. | **Ja**, es sei denn, es wird durch ein QA-konformes Zielobjekt innerhalb des Bildausschnitts verankert (siehe unten). |

`--reflectance-source` wählt die Reflektanzreferenz aus:**`auto`**(Standard) macht ein QA-konformes Zielobjekt im Bild zur**absoluten Referenz**– die zielverankerten empirischen Linienketten werden auf zurückbehaltenen Feldern kreuzweise ausgewertet und der gemessene Gewinner Band für Band angewendet: Ein Band, das der DAQ nicht kalibrieren kann oder dessen DAQ-Steigungsanpassung fehlschlägt, übernimmt die eigene Linie des Ziels (`ReflectanceElBandMethods` gibt an, welches) — ein Ziel, das die Qualitätssicherung mit einem Flag besteht (ein Panel wurde verworfen, ein Band ist außer Betrieb), konkurriert ebenfalls mit der DAQ-Trenngrenze um diesen gemessenen Wert, und ein Frame, gegen den es verliert, wird mit `ReflectanceChain=daq` gekennzeichnet — Rückgriff auf die DAQ-Downwelling-Grenze (ρ = π·L/E), wenn kein Ziel vorhanden ist oder die Qualitätssicherung fehlschlägt;**`target`**ist streng (keine DAQ-Ersetzung);**`daq`**weicht auf das DAQ-maßgebliche Verhalten aus. Die Zielgeometrie (ArUco / fester ROI / Streifen) stammt aus der Projektzielkonfiguration; `--target-reflectance-dir DIR` enthält pro Einheit**gemessene** Scans (`<serial>.csv`) pro Einheit, die anhand der Seriennummer/des QR-Codes der Ziel-Einheit nachgeschlagen werden, wobei die nominalen T3/T4P-Spektren als Fallback dienen.

Der DAQ-Reflexionspfad ermittelt automatisch die **zeitstempelabgeglichene Downwelling-Daten**aus einem aufgezeichneten**`.daq`**(DAQ-U/M/E)**oder einer nativen DAQ-M-Datei `.csv`**, die zusammen mit den Bilddaten vorliegt. Wenn ein kameraspezifisches oder DAQ-Kalibrierungspaket nicht lokal zwischengespeichert ist,**ruft die Pipeline es bei der ersten Verwendung automatisch von AWS ab** (erfordert einmalig eine Internetverbindung; wird unter `~/.chloros/` zwischengespeichert).**Wessen Messwert ist das?**Ein `.daq`, das eine Aufnahme speichert, ist der Messwert der Kamera(s), die es beschreibt – benannt in einem `<file>.daq.cameras.json`-Sidecar-Datensatz daneben (`lattice capture`, `multi-capture`, `array-capture` sowie in der GUI; eine Rohdatenserie nennt den Messwert jedes Mitglieds in ihrem Manifest) – und `process` ordnet ihn**nur Bildern dieser Kameras zu**. Eine eigenständige Kamera ohne eigene Messwerte nutzt daher nicht mehr die Datenerfassung (DAQ) einer anderen Kamera; weisen Sie ihr eine DAQ zu oder erstellen Sie eine eigene Aufzeichnung. Die eigene Aufzeichnung einer DAQ (`daq pool-record` oder eine in der GUI gestartete Aufzeichnung) oder jede `.daq` ohne diesen Sidecar (einschließlich älterer Aufnahmen) dient weiterhin allen Kameras.

#### Auslesen von Reflexionspixelwerten (Pix4D / Metashape / eigene Skripte)

Die Reflexion wird als ganzzahliger DN-Wert gespeichert, und **der DN-Wert, der ρ = 1,0 bedeutet, hängt von der Quellkamera ab**:

| Quelle | ρ = 1,0 ist | Wie zu erkennen |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (Spielraum bis ρ 2,0) | XMP-Tag „`Chloros:PixelScale=32768`“ ist in der Datei eingetragen. |
| Survey3 | `65535` (abgeschnitten bei ρ 1,0) | Keine `Chloros:*`-XMP-Tags – dieses Fehlen *ist* das Signal. |

**Lies `Chloros:PixelScale` und dividiere durch diesen Wert**, anstatt von einer Konstante auszugehen. Das Tag ist im uint16-Bereich definiert, daher bleibt es auch bei Ausgabeformaten, die eine Neuskalierung vornehmen, unverändert – `32768`, `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` und `TIFF (32-bit, Percent)` sind alle selbsterklärend (normalisieren Sie den gespeicherten Datentyp zunächst zurück auf uint16: ×257 von 8-Bit, ×65535 von Float).

> **Dateien aus älteren Builds dürfen keinen Skalierungsfaktor enthalten.** Ein 8-Bit-Reflexions-TIFF-Bild wird neu skaliert und wie die anderen mit „`Chloros:PixelScale`“ gekennzeichnet. Builds vor Version 1.3.0 haben es stattdessen auf 0..255 *beschnitten*, sodass jeder Wert über ρ≈0,008 auf 255 abgeflacht wurde, und sowohl das Tupel `Chloros:PixelScale` als auch das Tupel `MicaSense:RadiometricCalibration` wurden weggelassen. **Fehlt das Tag in einer LATTICE-Reflexionsdatei, gehe nicht von einem Maßstab aus – verarbeite sie erneut**, anstatt Pixel zu dividieren, die niemals teilbar waren.

#### Lesen von Indexpixeln (NDVI und die anderen Vegetationsindizes)

**Ein ganzzahliges Indexraster ist eine bildspezifische Kontraststreckung, keine −1…1-Zuordnung.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` und `JPG (8-bit)` – einschließlich des 8-Bit-TIFFs, als das eine 8-Bit-Aufnahme gespeichert wird – speichern

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

wobei `IndexMin` / `IndexMax` **die niedrigsten und höchsten endlichen Indexwerte dieses Frames** sind. DN 0 ist nicht −1, und der Vollbereich ist nicht +1.

Chloros versieht beide Endpunkte mit einem Stempel, damit die Abbildung invertierbar bleibt:

| Tag | Bedeutung |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` – die unten beschriebene Konvention |
| `Chloros:IndexMin` | der als DN 0 gespeicherte Indexwert |
| `Chloros:IndexMax` | der als Vollskala gespeicherte Indexwert |

Lies sie ein und kehre sie um, wobei du den gespeicherten DN zunächst auf den Maximalwert des gespeicherten Datentyps normierst, sodass ein Ausdruck jedes Ganzzahlformat abdeckt:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Zum Lesen wird kein `-config pix4d.config` benötigt – exiftool liest diese XMP-Eigenschaften selbstständig. Die Konfiguration ist nur zum *Schreiben* erforderlich.

> **`IndexMin == IndexMax`** bedeutet, dass jedes Pixel dieses Bildes denselben Indexwert hatte. Das Raster ist auf mittlerem Skalenbereich ausgefüllt und es gibt nichts, durch das man teilen müsste – man nimmt den Endpunkt.

> **Das Fehlen von `Chloros:Index*`-Tags bedeutet, dass keine Zuordnung angegeben ist.** Ein LUT-Export ist ein koloriertes Rendering, dessen Pixel Farbverläufe sind, die auf Ihr LUT-Schwellenwertfenster abgestimmt sind, und ein Bild ohne Pixel mit endlichem Index hat keinen anzugebenden Bereich. Gehen Sie in keinem der beiden Fälle von einer Zuordnung aus – es gilt dieselbe Regel wie bei einem fehlenden `PixelScale`.**Da die Streckung bildspezifisch ist, sind zwei Bilder eines Fluges hinsichtlich des DN nicht vergleichbar**– jedes Bild hat seine eigenen Endpunkte, sodass derselbe NDVI-Wert in verschiedenen Bildern auf unterschiedliche DN-Werte fällt. Für quantitative Auswertungen über einen Flug hinweg (zonale Statistiken, feste Schwellenwerte, Zeitreihen) exportieren Sie**`TIFF (32-bit, Percent)`**: Indexraster werden dort als float32-**Indexwerte selbst** geschrieben, ohne dass eine Neuskalierung oder Tags erforderlich sind. Trotz des Namens des Formats sind Indexprodukte nicht ×100 – diese Skalierung gilt nur für Reflektanzwerte vom Typ „Integer“.

#### Unmix-Clamp-Maske (auf welche Pixel ein Band geklemmt wurde)

Die 3×3-Spektralentmischung kann ein Band in den negativen Bereich treiben – dort, wo das Licht dieses Band kaum enthält oder wo die Szene außerhalb dessen liegt, was die Entmischungsmatrix der Kamera beschreibt. Da es sich dabei nicht um einen echten Wert handelt, setzt der Export ihn auf 0 – und die Live-Ansicht zeigt ebenfalls 0 an. Ein Band kann bei einigen wenigen Pixeln oder bei fast allen geklemmt sein; die Maske erfasst beides. `Chloros:Band{R,G,B}UnmixClampFraction` gibt an, wie viel Prozent eines Bildes davon betroffen waren; die Clamp-Maske gibt an, **welche Pixel**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— in einem `clamp_masks/`-Ordner**neben** dem Produktordner, niemals darin (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; ein Export von `process` führt zu `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) – für jedes Produkt aus Strahlungsintensität, Reflexionsgrad- und Index-Produkt, bei dem mindestens ein Pixel geklemmt wurde: sowohl Aufzeichnungen als auch `process`-Exporte (sowie die TIFF-Dateien pro Bild von `array-build-video --save-tiffs`). Einband-`uint8`, auf dem produkt-eigenen Pixelraster (einschließlich Ausrichtung, ROI, Binning und Array-Ausrichtung). Jede Maske benennt die Kamera ihres Produkts (`Make` / `Model` / Seriennummer), den Aufnahmezeitpunkt und die Objektiv-Tags und listet die Bänder auf, für die ihre Bits stehen (`XMP-Camera:BandName`, Bit 0 zuerst); die Bits `ImageDescription` und `Software` kennzeichnen sie als Clamp-Maske, und sie enthält keine Kalibrierung.
- **Bits**: Bit 0 = das Band im R-Kanal des Produkts (`Chloros:BandRName`), Bit 1 = G (`BandGName`), Bit 2 = B (`BandBName`). Ein gesetztes Bit bedeutet, dass der Wert des Pixels in diesem Band auf 0 begrenzt wurde – behandeln Sie dieses Bit und alle daraus abgeleiteten Indizes als wenig zuverlässig.
- **`Chloros:UnmixClampMask`** wird auf jedes Produkt gestempelt, bei dem die Entmischung durchgeführt wurde: der Pfad der Maske relativ zum eigenen Ordner des Produkts (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, immer durch `/` getrennt – fügen Sie es an den Ordner des Bildes an, um die Maske zu öffnen) oder `none`, wenn kein Pixel begrenzt wurde. Fehlt die Kennzeichnung, wurde der Unmix-Vorgang für das Produkt nie durchgeführt (bei RGB- oder Monochrom-Kameras, Raw-/Debayering-Ebenen).
- `lattice capture` / `array-capture` geben den Pfad der Maske unter der gespeicherten Datei an (`clamp mask: …`), und in der Liste der Aufnahmeresultate wird sie als `clamp_mask` des Frames aufgeführt.
- `lattice index --live --save-multiband` schreibt eine Maske pro Quellkamera für den Multiband-Stack – siehe [`lattice index` – Multiband-Stack-Masken](#lattice-index--multiband-stack-masks). Ein Video hat keine Maske: `array-build-video` meldet stattdessen die Clamp-Anteile jedes Einzelbildes (siehe [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Da kein Produktordner auf irgendeiner Ebene eine Maske enthält, kann ein Produktordner (`radiance/`, `reflectance/`, einem `process`-Export der Ordner `*_Images/`) Pix4D, Metashape oder einer anderen Photogrammetrie-Software unverändert übergeben werden – einschließlich aller Unterordner – und akzeptiert ausschließlich Bilder. Übergeben Sie den Produktordner, nicht den darüber liegenden Ordner (der enthält außerdem `clamp_masks/` und alle anderen Exporttypen). Der eigene Import von Chloros überspringt die Masken. Lesen einer Maske: `(mask & 1) != 0` bedeutet „das R-Kanal-Band wurde hier begrenzt“, `& 2` G, `& 4` B.

#### EXIF-Daten werden in den Export übernommen

`process` kopiert den **GPS-Block und dessen ExifIFD** der Quellaufnahme in jedes Produkt, sodass ein
Export `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` und
`CameraSerialNumber` neben den Georeferenzierungsdaten.

**`FocalLength` ist für die Photogrammetrie nicht optional.** Pix4D berechnet den Ground Sample Distance (GSD) aus
Brennweite und Höhe; fehlt das Tag, greift das Programm auf einen völlig falschen Maßstab zurück. Bei einem
Flug über einen Orangenhain mit 49 Aufnahmen verwandelte das fehlende Tag ein 411 m × 160 m großes Gelände in ein rekonstruiertes
47,8 km × 13 km – ein 455-MP-Orthobild mit überwiegend fehlenden Daten, was zunächst als Kachelungsproblem und
als BigTIFF-Problem interpretiert wurde, bevor jemand die GSD überprüfte. Falls Ihr Orthobild in einem unrealistischen
Maßstab ausgegeben wird, führen Sie zunächst `exiftool -FocalLength` auf das exportierte Produkt an.

Die Kopie ist bewusst **nicht** `-all:all`: Die Struktur-Tags von IFD0&#x27;dessen Struktur-Tags die LATTICE-Ausgabe beim
Kopieren stören, und `ExifImageWidth` / `ExifImageHeight` sind ausgeschlossen, da sie die
*Quell*-Aufnahme beschreiben – ein Export, dessen Größe jemals angepasst wurde, würde andernfalls Abmessungen enthalten
enthalten, die im Widerspruch zu seinem eigenen Raster stehen. XMP wird direkt geschrieben statt kopiert, da ExifTool
XMP-Tags desselben Aufrufs verworfen, wenn der XMP-Block kopiert wird (wodurch die MAPIR-
Kalibrierungstags verloren gingen).

### Wo die Ausgabedateien gespeichert werden

Die Ergebnisse werden **im Projektordner abgelegt, gruppiert nach Kamera und anschließend nach Dateiformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Der Kameraordner für LATTICE lautet `LATT-<sensor>-<lens>-F<filter>` (entsprechend dem EXIF-Tag der Aufnahme
`Model`) und `<model>_<filter>` für Survey3 – zwei Kameras, die sich einen Sensor und einen Filter teilen, sich jedoch
im Objektiv unterscheiden, bilden separate Verzeichnisstrukturen, da Vignettierung, Sichtfeld und Verzeichnung variieren. Der Formatordner
Ordnerstruktur lautet wie folgt: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` oder `tiff32` für
`TIFF (32-bit, Percent)`.

> **Jedes exportierte Produkt behält den Namen der QUELLDATEI bei.** Ein Radiance-Export von
> `capture_…_raw.tif` heißt weiterhin `capture_…_raw.tif` – er befindet sich lediglich in
> `tiff32/Radiance_Images/`. **Der Ordner identifiziert das Produkt, nicht der Dateiname**, daher liefert eine Suche
> nach „`*radiance*.tif`“ keine Ergebnisse; suchen Sie stattdessen nach dem Verzeichnis.

### Lichtsensoraufzeichnungen – kalibriert `.daq` + `.csv`

`process` verarbeitet auch die `.daq`-Aufzeichnungen in Ihrem Eingabeordner und benötigt **keine**
dafür Bildmaterial benötigt: Ein eigenständig geflogener DAQ-U / DAQ-M / DAQ-E stellt eine vollständige
Aufzeichnung dar, und ein Ordner, der ausschließlich `.daq`-Dateien enthält, ist eine gültige Eingabe.

Ein DAQ kann **ohne** seine Kalibrierung aufgezeichnet werden – genau das tun die öffentlichen
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) Recorder
(`record_daq.py`) standardmäßig tun: Sie schreiben die rohen Sensorwerte und versehen die Datei mit einem Zeitstempel, sodass
*Chloros* die Werkskalibrierung dieses Sensors **über die serielle Schnittstelle** (zuerst aus dem lokalen Cache,
dann aus der MAPIR-Cloud) und wendet sie an. `process` schreibt das Ergebnis zurück:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

Das `.csv` enthält eine Zeile pro Messwert: UTC-Zeitstempel, Integrationszeit, Gesamtleistung,
photopische/skotopische Lux, PPFD (und deren Aufteilung in Blau/Grün/Rot), Spitzenwellenlänge, dann das
Vollspektrum auf dem sensoreigenen Wellenlängengitter. Die Datei `.daq` wird erneut importiert, ohne
ein zweites Mal kalibriert zu werden.

`<name>` ist der Dateiname der Aufzeichnung selbst. Zwei Aufzeichnungen in unterschiedlichen Ordnern können sich
einen gemeinsamen Wert teilen – die pro Aufnahme erfassten Messwerte (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), die von zwei Kameras
im selben Millisekundenzeitraum gespeichert wurden –, und jede erhält dennoch ihre eigenen Ergebnisse: Die erste behält
`<name>`, die nächste fügt die Ordner, die sie unterscheiden, **nach** dem Namen hinzu
(`<name>__<capture>_daq_calibrated.daq`), sodass Datum und Uhrzeit der Aufzeichnung weiterhin an erster Stelle stehen.

Bei Erfolg meldet der Durchlauf `Light-sensor products written: N (calibrated .daq + .csv)`.
Die Angabe in Klammern beschreibt, was tatsächlich geschrieben wurde, es lautet also
`(RAW COUNTS — this sensor has no calibration bundle)` für einen Sensor ohne Bundle und
`(N calibrated, M raw counts)` für einen Ordner, der beides enthält. Die backendeigenen
Überschriften „`[DAQ-EXPORT]`“ und „`[RUN-SUMMARY]`“ leiten ihren Wortlaut auf dieselbe Weise ab – keine der
drei kann einen Roh-Export als kalibriert bezeichnen.

Eine DAQ-U-/DAQ-M-/DAQ-E-Aufzeichnung, deren Kalibrierungsbündel nicht abgerufen werden kann – Sie sind
offline oder für diesen Sensor liegt keine Kalibrierung vor –, wird **mit einer Begründung übersprungen** in einer
`[DAQ-EXPORT]`-Zeile und wird niemals als „kalibrierte“ Datei mit Rohwerten ausgegeben.
Stellen Sie eine Verbindung zum Internet her und führen Sie den Vorgang erneut aus. Der Grund ist derjenige, den das Lesegerät tatsächlich
für diese Datei ermittelt hat (unlesbares Schema, kein Paket, ein Schreibfehler), und die Laufzusammenfassung
listet **eindeutige** Gründe auf – zwanzig übersprungene Dateien aufgrund einer Ursache werden als eine
Ursache ausgewiesen, nicht als zwanzig Wiederholungen derselben. Die CLI nennt am Ende des
Laufs jede übersprungene Aufzeichnung mit dem jeweiligen Grund:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Ein Ordner mit Aufzeichnungen, von denen **keine** exportiert wird, schlägt fehl: siehe
[Ein Lauf, der keine Bilder schreibt, schlägt fehl](#a-run-that-writes-no-images-fails).

Eine Aufzeichnung, deren Name auf `_calibrated` oder `_raw` (`field_raw.daq`) endet, wird **nicht
importiert**: Dies sind die Namen, die `process` seinen eigenen Produkten zuweist, und ein Produkt, das
als Aufzeichnung zurückgelesen wird, würde doppelt gezählt werden. Die CLI gibt dies an und nennt die Datei –
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
– anstatt einen leeren Ordner zu melden. Benennen Sie eine solche Aufzeichnung vor der Verarbeitung um.

#### DAQ-A-Aufzeichnungen werden als Rohzählwerte exportiert

Die **DAQ-A** -Familie stammt aus der Zeit vor dem seriennummernbezogenen Bundle-System und verfügt über kein Kalibrierungs-Bundle,
das abgerufen werden könnte – sie wird stattdessen vor Ort anhand eines Reflexionsziels kalibriert, weshalb
sie nie eines benötigte. Würden diese Aufzeichnungen abgelehnt, gäbe es keine Möglichkeit, ihre
Werte überhaupt zu extrahieren; daher werden sie unter einem **anderen Namen** exportiert:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Ein anderer Dateiname anstelle eines Flags innerhalb der Datei, da die Angabe auch dann erhalten bleiben muss,
wenn sie als reiner Name per E-Mail weitergeleitet wird. Der Header „`.csv`“ gibt
„`raw spectral sensor counts (NOT irradiance)`“ an und warnt, dass die Werte
**innerhalb** der Datei vergleichbar sind – was genau dem Zweck der zielbasierten Kalibrierung entspricht – und
nicht sensorübergreifend. Die leistungsabhängigen photometrischen Spalten (Gesamtleistung, photopische und
skotopische Lux, PPFD) werden als **NULL** geschrieben, anstatt aus Zählwerten integriert zu werden, und die
Laufzusammenfassung lautet `RAW COUNTS`, sodass „exportierte“ in einem Protokoll kann nicht als Bestrahlungsstärke interpretiert werden.

Ältere **v1.01 / v1.02**-Aufzeichnungen (die von einem DAQ-A-SD geschrieben werden) enthalten keine Epochenangabe pro Messwert,
sondern nur den Schreibzeitpunkt der Datei. Der Bild↔Downwelling-Matcher lehnt sie nach wie vor ab – ein Abgleich eines
Bildes mit einem Schreibzeitstempel wäre unsichtbar falsch –, aber der Exporter liest sie ein, und
die CSV-Datei gibt „`clock=daq_created_on`“ aus, sodass das Produkt angibt, auf welcher Uhr es basiert.

### Hinweise

- `process` erkennt automatisch, ob es sich bei Ihrem Ordner um Survey3, „LATTICE“ oder eine gemischte Sammlung handelt.
- Der Fortschritt wird über Server-Sent Events übertragen; die CLI zeigt den Live-Fortschritt pro Thread an (Erkennen, Analysieren, Verarbeiten, Exportieren).
- Unter Linux/Jetson überprüft die CLI den Swap-Speicher und gibt möglicherweise eine Warnung aus, bevor große Ordner verarbeitet werden. Der texturbasierte Debayer wendet bei Jetson-Modellen mit geringer Leistung (Nano, Orin Nano) außerdem automatisch eine Begrenzung der GPU-Frequenz an.
- Bei erfolgreichem Abschluss meldet der Lauf, wie viele Bildprodukte geschrieben wurden (`Image products written: N`, bzw. `(plus M clamp masks)`, wenn beim Unmixing Pixel gekürzt wurden).
- **`process` nutzt ein laufendes Backend sicher gemeinsam.** Es startet ein bereits laufendes Backend automatisch nur dann neu, wenn dieses Backend im Leerlauf ist, ein vorheriges Projekt geladen ist und es keine Kameras, Arrays oder DAQs enthält. Ein Backend, das eines dieser Elemente enthält (die GUI, die ein Array streamt, ein SDK-Skript, eine DAQ-Aufzeichnung), wird unverändert verwendet und niemals neu gestartet, sodass `process` neben einem aktiven Array laufen kann. Wenn das Backend einen laufenden Lauf meldet, wartet `process` bis zu 15 Sekunden, bis dieser abgeschlossen ist (ein gerade beendeter Lauf benötigt etwa eine Sekunde, um freigegeben zu werden), und lehnt den Vorgang dann mit dem Fehler `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).` ab. Das globale Flag `--restart` erzwingt weiterhin ein neues Backend und beendet alle darauf laufenden Kamera-, Array- oder DAQ-Sitzungen. Ein Backend, das als Linux-systemd-Dienst läuft, wird niemals neu gestartet: `process` nutzt es, und `--restart` gibt stattdessen `sudo systemctl restart <unit>` aus (siehe [Ausführen des Backends als Dienst](#running-the-backend-as-a-service-headless)). Das gilt auch für das eigene Backend der Chloros-App: Bei geöffneter App nutzt `process` dieses Backend und lädt sein Projekt anstelle des App-Projekts (ein an seiner Stelle gestartetes Backend würde die geöffnete App aussperren).
- **Ein laufender Durchlauf behält sein Projekt bei.** Während ein Verarbeitungslauf läuft – gestartet durch die App, `process`, ein SDK-Skript oder `chloros-mcp` – verweigert das Backend das Erstellen, Öffnen, Importieren in, Hinzufügen oder Entfernen von Dateien, das Verschieben, zu löschen oder die Verarbeitungseinstellungen eines Projekts zu ändern – und gibt dabei die HTTP-Fehlercodes 409 `PROCESSING_IN_PROGRESS` und `A processing run is in progress on project '<name>', so ... now is refused` zurück; ein `process`, das dies erhält, beendet `1` beendet wird. Die Anzeigeeinstellungen (Miniaturbildgröße) werden weiterhin gespeichert. Der Verarbeitungsmodus und die Einstellungen für die Zielreflexion des Projekts werden auf dieselbe Weise abgelehnt (ein Lauf behält den Modus bei, in dem er gestartet wurde). Eine während des Laufs im Projekt gespeicherte Aufnahme (aus der App, der CLI, dem SDK oder `chloros-mcp`) wird wie gewohnt gespeichert, wird aber erst nach Beendigung des Laufs in die Dateiliste des Projekts aufgenommen; dies geht aus der entsprechenden Rückmeldung hervor (`project_add`), ebenso wie bei einer im Projekt angehaltenen Lichtsensoraufzeichnung. Zuvor konnte ein zweiter Client das geöffnete Projekt während des Laufs wechseln, woraufhin der laufende Durchlauf in das andere Projekt exportiert wurde.
- **Strg+C stoppt den Lauf im Backend, nicht nur in der CLI.** Die CLI fordert das Backend auf, die Verarbeitung zu beenden, gibt `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` aus und wartet bis zu 120 s auf `Processing stopped.`, damit `chloros-cli process … && next` nicht `next` auf einem Backend, das noch mit dem Lauf beschäftigt ist. Ein zweites „Strg+C“ beendet das Warten. `process` beendet dann `130`, wie es jeder unterbrochene Befehl tut.

#### Ein Lauf, der keine Bilder schreibt, schlägt fehl

Wenn Sie nach Produkten gefragt haben und der Lauf **keine** geschrieben hat – nur `project.json` und
`calibration_data.json` –, behandelt `process` dies als Fehler: Es gibt
`Processing finished but wrote no image products.` aus und **beendet den Lauf mit einem Wert ungleich Null**, sodass ein Skript
dies erkennen kann. Die Meldung nennt den Projektordner und die üblichen Ursachen:

- Der Eingabeordner wurde nicht als Aufzeichnung erkannt (überprüfen Sie das Layout und `--input-level`) oder
- jedes angeforderte Produkt wurde als für diese Kameras nicht anwendbar übersprungen (z. B. bei der Abfrage von
  Strahlungsintensität/Reflexionsgrad von reinen RGB-Kameras).

Führen Sie den Vorgang mit `--verbose` erneut aus und überprüfen Sie das Backend-Protokoll auf Zeilen mit den Nummern `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
die die kameraspezifischen Übersprünge erklären, die sonst nicht in der Ausgabe der CLI erscheinen.

Ein absichtlich nur auf Metadaten ausgerichteter Lauf – alle Produkte deaktiviert und kein `--indices` – ist dennoch ein
**Erfolg**, da eine leere Bildausgabe in diesem Fall das korrekte Ergebnis ist.

#### Ein Lauf, dessen Exporte nicht verifiziert werden konnten, schlägt fehl

Nach einem Durchlauf überprüft `process` die Tags der exportierten Dateien mit ExifTool. Wenn ExifTool nicht ausgeführt werden konnte,
wird Folgendes ausgegeben: `Exportüberprüfung FEHLGESCHLAGEN: ExifTool konnte nicht ausgeführt werden, daher wurden die Tags von N von M exportierten Dateien
nicht überprüft (siehe die [EXIFTOOL]-Zeilen im Backend-Protokoll)` and `Die Ergebnisse werden geschrieben und bleiben
an Ort und Stelle; führen Sie den Lauf erneut aus, um sie zu überprüfen.`, does not print `Verarbeitung abgeschlossen!` aus und **beendet sich mit einem Wert ungleich Null**. Die
Ergebnisse bleiben erhalten.

#### Welche Aufnahmen ein Lauf berücksichtigt

Aufnahmen, die vor der Ausführung von `process` fertiggestellt wurden, werden einbezogen. Während des Laufs gespeicherte Aufnahmen werden nicht berücksichtigt; `process`
gibt eine Warnung aus, wie viele ausgelassen wurden, und bei erneuter Ausführung werden diese einbezogen.

#### Ein Lauf, bei dem ein Bild verloren ging, schlägt fehl

Wenn ein Bild während eines Laufs fehlgeschlagen ist, gibt `process` die Meldung `N image(s) failed and were not exported` mit der
häufigsten Ursache aus, gibt die Meldung `Processing complete!` nicht aus und **beendet den Prozess mit einem Wert ungleich Null**. Die anderen Produkte werden
geschrieben und verbleiben an ihrem Platz, und das Backend-Protokoll enthält jeden Fehler.

Das Gleiche gilt für einen **Lauf nur mit Lichtsensor**: Ein Ordner mit `.daq`-Aufzeichnungen enthält per Definition keine Bilder zum Exportieren,
und der Durchlauf wird stattdessen anhand der kalibrierten `.daq`- und `.csv`-Dateien bewertet, die er geschrieben hat.
Wenn **keine** geschrieben wurden – also jede Aufzeichnung übersprungen wurde –, gibt er
`Processing finished but wrote no light-sensor products.` aus, benennt jede Aufzeichnung mit dem
Grund für das Überspringen und **beendet den Lauf mit einem Wert ungleich Null**.

---

## `chloros-cli login`

Authentifizieren Sie diesen Rechner mit einem Chloros+-Cloud-Konto. Die Anmeldedaten werden sicher in `~/.chloros/user_session.json` zwischengespeichert.

```
chloros-cli login EMAIL PASSWORD
```

### Beispiele

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (Entfernen von Teilen des Passworts oder Duplizieren von Passwortteilen). Bei einem 401-Fehler versucht die CLI automatisch einen erneuten Versuch mit `$$`,, anschließend mit einer deduplizierten Hälfte des Passworts; wenn ein Wiederholungsversuch erfolgreich ist, werden Sie angemeldet und die korrekte Syntax mit einfachen Anführungszeichen für das nächste Mal angezeigt.

> **Headless-/Skript-Verwendung: Keine zwischengespeicherte Sitzung bedeutet eine Anmeldeaufforderung.** Ein Core-Befehl (`process`, `status`, `export-status`, `time-sync`, …) wird ohne zwischengespeicherte Sitzung ausgeführt, fordert vor der Fortsetzung die Eingabe von `Email:` / `Password:` über stdin an. Ist stdin geschlossen (cron, eine systemd-Einheit), schlägt der Vorgang sofort mit dem Fehler `login cancelled` fehl; ist stdin ein offenes Terminal oder eine Pipe, wartet das Programm auf Eingaben. Führen Sie `chloros-cli login EMAIL PASSWORD` einmal pro Rechner aus, bevor Sie headless-Aufgaben einplanen.

---

## `chloros-cli logout`

Löscht die zwischengespeicherte Sitzung und erzwingt beim nächsten Aufruf eine neue Anmeldung.

Das Abmelden während eines laufenden Verarbeitungslaufs stoppt diesen nicht: Der Lauf wird in dem Verarbeitungsmodus beendet, in dem er gestartet wurde, und der nächste Lauf verwendet die Standardverarbeitung. `logout` weist darauf hin (`The processing run in progress finishes in <mode> mode; the next run uses standard`), ebenso wie die App, wenn Sie sich sich dort ab.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Zeigt die aktuelle Lizenzstufe (Iron/Copper/Bronze/Silver/Gold), den authentifizierten Benutzer und die Anzahl der Gerätebindungen an.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Den aktuellen Fortschritt des Thread-4-Exports abfragen. Kann **während** der Ausführung von `process` von einer anderen Shell aus sicher aufgerufen werden.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Legt die Anzeigesprache der CLI fest (38 Sprachen werden unterstützt, darunter CJK, RTL und indische Sprachen). Auf älteren Konsolen, die das Skript nicht darstellen können, wird nahtlos auf Englisch zurückgegriffen.

```
chloros-cli language [LANG_CODE] [--list]
```

### Beispiele

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Befehle für den Projektordner

Diese Befehle verwalten den Speicherort des Standard-Projektordners (gemeinsam mit der GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` gibt den Ordner aus, in dem neue Projekte abgelegt werden. Wenn der konfigurierte Ordner nicht mehr existiert (er wurde gelöscht, umbenannt oder befindet sich auf einem nicht angeschlossenen Laufwerk), greift die CLI, genau wie die GUI, auf den Standardordner zurück: Der Standardordner wird auf stdout ausgegeben, eine Warnung mit dem Namen des fehlenden Ordners wird an stderr gesendet und der Exit-Code lautet `0`.

---

## `chloros-cli update`

Nur Linux/Jetson. Überprüft `version_url` anhand von `/etc/chloros/update.conf` und lädt, falls eine neuere Version vorhanden ist, wird die entsprechende Version `.deb` heruntergeladen und sofort mit `sudo dpkg -i` installiert (keine Bestätigungsabfrage; bei `sudo` werden Sie möglicherweise nach Ihrem Passwort gefragt). Auf einem Jetson wird das Update nur für ein JetPack-6-System angeboten. Unter Windows führt der Befehl keine Aktion aus und meldet „`up to date`: Update mit einem neuen Installationsprogramm“.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Unter Linux/Jetson führt die CLI außerdem **bei jedem Start eine automatische Update-Prüfung** durch (nicht blockierend, verzögert den Befehl niemals): Sie liest „`/etc/chloros/update.conf`“, speichert das Ergebnis für 1 Stunde in „`~/.chloros/update_cache.json`“ im Cache und gibt „`Update available: vX.Y.Z / Run: chloros-cli update`“ aus, wenn eine neuere Version vorhanden ist. Bei Fehlern und unter Windows wird der Vorgang stillschweigend übersprungen; mit `CHLOROS_DISABLE_UPDATE_CHECK=1` wird er deaktiviert. Aufgrund des 1-stündigen Caches kann eine innerhalb dieser Stunde veröffentlichte Version weiterhin als aktuell angezeigt werden.

---

## `chloros-cli selftest`

Führt einen 7-stufigen Smoke-Test durch: Version, Portverfügbarkeit, Backend-Start, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), Vorhandensein des Denoiser-Modells, Bereitschaft von CUDA+Denoiser. Dabei wird ein bereits laufendes Backend verwendet oder für den Test gestartet und anschließend beendet. Es gibt `N/7 checks passed` und die fehlgeschlagenen Prüfungen aus und beendet `0` erst, wenn alle sieben Prüfungen bestanden sind; die CUDA- und Denoiser-Prüfung schlägt auf einem Computer ohne CUDA-GPU fehl (siehe [Installation überprüfen](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-Grandmaster-Status und -Steuerung. Der Chloros-Host betreibt den PTP-Grandmaster; LATTICE-Kameras und DAQ-E-Einheiten sind diesem als Slaves untergeordnet, um geräteübergreifende Zeitstempel zu gewährleisten (ca. 1 ms geräteübergreifend). PTP gleicht **Zeitstempel** ab: Die Kameras eines Arrays belichten gleichzeitig, weil der M8-Hardware-Trigger sie auslöst (weniger als 100 µs zwischen den Modulen), nicht aufgrund von PTP. `chloros-cli time-sync` ohne Unterbefehl führt `status` aus.

| Unterbefehl | Beschreibung |
| --- | --- |
| `status` | Zeigt den Grandmaster-Status, die BMCA-Prioritäten, die Taktidentität und an, ob seine Pakete das Netzwerk erreichen: `health` ist `OK` (`STOPPED`, wenn der Grandmaster nicht läuft) oder `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` zusammen mit einem `health_detail`, und anschließend wird eine Zeile „`WARNING:`“ auf stderr ausgegeben (stdout bleibt JSON). `sync_sent` / `announce_sent` zählen nur die Nachrichten, die gesendet wurden; `*_send_errors` und `last_send_error` zählen diejenigen, die nicht gesendet wurden. Nachdem eine Kameraverbindung unterbrochen wurde und wiederhergestellt ist, baut der Grandmaster seine Sockets selbstständig wieder auf (`rebinds`, `last_rebind`). |
| `peers` | Liste der über Delay_Req erkannten Slaves (Kameras + DAQ-E-Sensoren). |
| `cameras` | PTP-Zustand pro Kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Neustart des Grandmaster-Prozesses. |
| `set-priority --priority1 N --priority2 N` | BMCA-Prioritäten überschreiben. |

Der Grandmaster sendet Ankündigungen über die Netzwerkkarten der Kameras (jeder Adapter mit einer `169.254.x.x`-Adresse ist eingeschlossen). Ist keiner verfügbar, greift er nur auf den Adapter der Standardroute zurück, sofern dieser kabelgebunden ist – niemals auf WLAN oder einen virtuellen Adapter (Tailscale, WSL), da sich keine Kamera oder DAQ-E sich hinter einem solchen befindet. Da nichts anzukündigen ist, läuft er ohne Sockets (`health` `NO_SOCKETS`), und die nächste Kamera oder das nächste DAQ-Gerät baut ihn bei der Verbindung auf dem zu diesem Zeitpunkt aktiven Netzwerkadapter wieder auf; `time-sync restart` erledigt dies sofort.

Der Grandmaster läuft in einem eigenen Prozess neben dem Backend (`own_process` / `process_pid` in `time-sync status`), unter Windows mit hoher Priorität (`process_priority`), sodass ein ausgelastetes Backend (ein Verarbeitungslauf, ein Export, mehrere Befehle gleichzeitig) die von ihm gestempelten Zeitangaben nicht verzögern; zuvor konnte eine solche Verzögerung die Uhr jeder Kamera gleichzeitig verschieben. Dieser Prozess endet mit dem Backend; wenn er nicht gestartet werden kann, läuft der Grandmaster wie zuvor innerhalb des Backends (`own_process` false, und das Backend-Protokoll vermerkt dies).

### Beispiele

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE-Kamerasteuerung. Jeder Unterbefehl wird über das Chloros-Backend geleitet; das Backend verwaltet den Kamerapool, sodass nachfolgende CLI-Aufrufe denselben offenen Handle wiederverwenden.

### Allgemeine Optionen (gemeinsam für die meisten Unterbefehle)

| Flag | Beschreibung |
| --- | --- |
| `-d, --device N` | Kameraindex (Standard: 0). |
| `-s, --serial SN` | Spezifische Seriennummer; überschreibt `--device`. |
| `--serials SN1,SN2,…` | Durch Kommas getrennte Seriennummern für den Multi-Kamera-Betrieb. Die [transienten Multi-Kamera-Befehle](#multi-camera-transient-sessions) öffnen nur diese Kameras, wobei die erste den Auslöser betätigt. |
| `--all` | Auf jede erkannte Kamera anwenden. |
| `--exposure US` | Belichtungszeit in Mikrosekunden. |
| `--gain DB` | Verstärkung in dB. |
| `--pixel-format FMT` | z. B. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` auf einem M3M. Der Name legt die Bittiefe innerhalb der kameraspezifischen Familie fest: auf einem M3M legt ein Bayer-Name das Mono-Format seiner Tiefe fest (`BayerRG8` legt `Mono8` fest), auf einem M3C legt ein Mono-Name den Bayer-Namen fest (`Mono12` legt `BayerRG12` fest). |
| `--width N` / `--height N` | Bildabmessungen. |
| `--preset {default,high_quality,high_speed,triggered}` | Eine Voreinstellung anwenden: Pixelformat, Belichtung, Verstärkung und Bildrate (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` listet sie auf). Auf einem M3M ist das Format das Mono-Format mit derselben Tiefe: `high_speed` erfasst `Mono8`, `high_quality` und `triggered` sowie `Mono12`. Ein dabei angegebenes Flag (`--exposure`, `--gain`, `--pixel-format`) hat Vorrang vor der Voreinstellung, und eine Einstellung, die in der Voreinstellung nicht genannt wird, behält die kameraspezifische Einstellung bei. Alle freienLauf, mit Ausnahme von `triggered`, das die Kamera für eine Hardware-Flankensignaländerung auf Leitung 2 schaltet – wenn diese Leitung nicht angesteuert wird, wartet die Kamera unendlich lange, anstatt eine Aufnahme zu machen. |
| `-o, --output DIR` | Ausgabeverzeichnis (Standard: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-Paketgröße. `auto` führt ICMP+GVSP-Probes durch; `jumbo` = 9000; `standard` = 1500. |

### LATTICE-Kamera: Workflow für die Erstverbindung

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referenz zu Unterbefehlen

#### Erkennung und Informationen

| Unterbefehl | Zweck |
| --- | --- |
| `lattice info` | Liste der verbundenen Kameras (Hersteller, Modell, Seriennummer, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Host-System für optimale Kamerakonfiguration analysieren. `--no-discover` überspringt die Kameraerkennung (schneller, Analyse nur über Netzwerkkarte). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Netzwerkkarteneinstellungen prüfen/korrigieren (unter Linux auch die Socket-Empfangspuffer des Kernels – siehe [Linux-Hosts](#linux-hosts--socket-receive-buffers)); Bandbreite/FPS schätzen. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Backend-Netzwerk mit stabilem Schema-Fähigkeit + Array-Empfehlung (gibt `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` sind standardmäßig auf den vollständigen Sensor eingestellt, `2048` × `1536`. Es wird `0` für `ok`, `auto_capped_fps` und `auto_shrunk` sowie von `1` zu `needs_force_slip` und `error`, mit oder ohne `--json`. Es erreicht das Backend bei `--backend-url` (Standard: `http://127.0.0.1:5000`), nicht bei `CHLOROS_BACKEND_URL`. `auto_capped_fps` behält die angeforderte Auflösung bei, begrenzt jedoch die Ziel-fps – lesen Sie `recommended.recommended_target_fps` und übergeben Sie diesen Wert als Verbindungsziel; behandeln Sie dies als Erfolg, nicht als Fehler. `auto_shrunk` tritt nur auf, wenn der angeforderte Frame bei keiner Bildrate geliefert werden kann; „connect“ verkleinert Frames niemals von sich aus. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Was-wäre-wenn-Analyse ohne Kameras zu öffnen (`--width` / `--height` Standard `2048` × `1536`). Sowohl dies als auch `network-analysis` projizieren, was „connect“ streamen wird: ohne `--pixel-format` gelten die von „connect“ festgelegten Formate pro Kamera (Mono12 / BayerRG12), und bei `--packed auto` (Standard) die Entscheidung, die „connect“ auf diesem Computer bezüglich [gepackter 12-Bit](#packed-12-bit-transport---packed) – also die Bildrate, mit der das Array läuft. Beide geben die Packing-Entscheidung aus. **`--n-active` ist die Gesamtzahl der Kameras im Netzwerk, nicht nur die dieses Arrays**– erhöhen Sie diesen Wert, wenn eigenständige Kameras gleichzeitig streamen oder das Leitungsbudget anhand eines Bedarfs berechnet wird, der diese nicht vollständig berücksichtigt (Standard: `len(--models)`). Gibt immer die aggregierten Zeilen `Wire budget:` (geforderte MB/s vs. kollisionssichere Obergrenze) und `Max cameras:` aus und setzt das Flag `** OVER-SUBSCRIBED**`, wenn das Array die Leitung überbelegt — siehe [Array-FPS- und Burst-Modell](#array-fps--burst-model). |
| `lattice gpu` | GPU-Status anzeigen. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Kamera-Firmware prüfen oder aktualisieren. Die lokale `.fwa`-Auswahl ist festgesetzt: Die Datei in `firmware/<MODEL_PREFIX>/`, die mit dem `MIN_FIRMWARE_VERSION` des Builds übereinstimmt, wird, sofern vorhanden, geflasht (nur die höchste Version als Fallback), sodass ein neueres, auf der Festplatte bereitgestelltes Hersteller-Image inaktiv bleibt, bis diese Fixierung aufgehoben wird – bewusst neuere Versionen gelangen über das signierte AWS-Manifest auf die Geräte, was bevorzugt wird, wenn es neuer ist. |
| `lattice presets [--apply NAME]` | Kameravoreinstellungen auflisten oder anwenden. `--apply` legt fest, wie die Voreinstellungen benannt werden (Pixelformat, Belichtung, Verstärkung, Bildrate, Auslöser), und gibt diese Einstellungen so aus, wie die Kamera sie anschließend ausliest: auf einem M3M das Mono-Format der Tiefeninformation der Voreinstellung; die Bildrate, mit der sie läuft, die niedriger sein kann als von der Voreinstellung gefordert (`high_speed` fordert 60 fps; eine Vollformatkamera an einer 1-Gbit/s-Verbindung läuft mit etwa 35). Alle anderen Einstellungen, die „open“ geschrieben hat, werden unverändert zurückgesetzt. `--apply default` setzt die Kamera auf die Werkseinstellungen zurück: die Werkseinstellungen mit frei laufender automatischer Belichtung auf Line2, deaktivierter Verbindungsdurchsatzbegrenzung und deaktivierter Fehlerkorrektur, wobei der kalibrierte Weißabgleich einer RGB-Kamera beibehalten wird (bei allen anderen auf „unity“). Es wird nichts im Speicher der Kamera gespeichert, sodass sie beim nächsten Einschalten wie zuvor aus ihrem eigenen Speicher startet. Als `--preset` bei einer Aufnahme oder Verbindung ist `default` keine Voreinstellung. |
| `lattice status` | Live-Kamerastatus. |

#### Aufnahme

| Unterbefehl | Zweck |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Einzelbild. **Speichert standardmäßig jeden Exporttyp**(`--processing all`); siehe [Aufnahme-Exportstufen](#capture-export-levels-the-all-default). `--processing` akzeptiert `all`, `raw`, `debayered`, `radiance`, `reflectance` oder `display` (`preview` wird für `display` akzeptiert); jeder andere Name wird abgelehnt, bevor die Kamera berührt wird (Beenden von `1`). `--levels` speichert eine explizite Teilmenge (überschreibt `--processing`). Eine Kamera, die Chloros nicht verwaltet, wird für die Aufnahme geöffnet, und ihre Belichtung wird zunächst anhand der Szene eingestellt (höchstens einige Sekunden), wie es auch bei den [transienten Befehlen](#multi-camera-transient-sessions) der Fall ist; Wenn die Aufnahme endet – ganz gleich, wie sie endet –, wird diese Kamera wieder in den Zustand versetzt, in dem sie vorgefunden wurde (Format, Belichtung, Farbpipeline, jede Einstellung, die durch die Aufnahme verändert wurde – siehe die Liste unter [Multi-Kamera (Transient Sessions)](#multi-camera-transient-sessions)). Der DAQ-Messwert der verwendeten Aufnahme wird als `.daq`-Sidecar gespeichert, einschließlich eines reinen Rohdatenschnitts; `--no-daq` / `--daq-tolerance-ms` passen diesen an ([Das DAQ, das eine Aufzeichnung ausliest, speichert](#the-daq-reading-a-capture-saves)). `--force-daq` wird weiterhin akzeptiert. `--jpeg-quality` = JPEG-Qualität 1–100 (Standard 95).**Dateien, die in das geöffnete Projekt gespeichert werden, werden diesem hinzugefügt.** Wie in der App werden bei einer Aufnahme, deren Dateien im Ordner des in Chloros geöffneten Projekts landen, diese zur Dateiliste dieses Projekts hinzugefügt; an anderer Stelle gespeicherte Dateien bleiben unberührt. Der Befehl gibt eine Zeile aus, die dies bestätigt (`Project: ...`) oder – während ein Verarbeitungslauf läuft – dass sie zurückgehalten und nach Abschluss des Laufs hinzugefügt werden. Dasselbe gilt für `multi-capture`, `array-capture` (sowie `--smart`) und `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Auf die Festplatte streamen, bis Strg+C gedrückt wird. Ein Lauf von `tiff`s Frames bilden die Datei `raw` – ein Array, das die Aufzeichnungen dieser Kamera enthält: das Mosaik des Sensors, eine Ebene mit ihren eigenen Werten, unkomprimiert, zusammen mit dem Kalibrierungspaket (ein 2048 × 1536 12-Bit-Bild ist 6,0 MiB plus das Paket, unabhängig von der Szene), daher geht `process` von diesen aus. `png` und `jpg` enthalten 8 Bit; es handelt sich also um demosaizierte Bilder zur Anzeige, von denen `process` nicht ausgeht. Bei einer DAQ-gebundenen (`daq pool-bind`) oder im Pool gespeichert, erhält jedes Bild seine eigenen `.daq` und `.match.json` im `daq/`-Ordner des Laufs, da `lattice capture` einen speichert; `--no-daq` deaktiviert dies, und beim Beenden wird angezeigt, wie viele es sind. Jedes Bild wird einmal mit seinen vollständigen Metadaten geschrieben; bei Strg+C werden die bereits erfassten Bilder geschrieben, bevor die Kamera freigegeben wird. Beendet sich selbst, wenn das Laufwerk voll ist, gibt „`Failed:` /“ aus `Disk full:` aus und beendet den Vorgang mit `1`; jedes Bild, das nicht geschrieben werden konnte, führt ebenfalls zum Beenden mit `1`. Eine Kamera, die für einen Hardware-Trigger (`TriggerMode` `On`) wird für den Durchlauf auf Freilauf eingestellt. Nach Beendigung des Durchlaufs wird die Kamera in den ursprünglichen Zustand zurückversetzt – ihre Auslöseeinstellungen und alle während des Durchlaufs geänderten Einstellungen (Belichtung, Verstärkung, Format, …) – es sei denn, es handelt sich um `--keep-settings`, das die Einstellungen des Durchlaufs im kamereneigenen Speicher (Freilauf) speichert, oder um `--clear-settings`, das diesen Speicher auf die Werkseinstellungen zurücksetzt. Ein Durchlauf, bei dem kein Bild aufgezeichnet wird, gilt als Fehlschlag: Wenn 10 Sekunden lang kein Bild vorliegt, stoppt er sich selbst, gibt `No frames:` mit dem Grund aus und beendet sich mit `1`. Bilder, mit denen die Writer nicht Schritt halten konnten, werden als verworfen gezählt, und beim Abbruch wird „`Dropped:`“ mit der Ursache (Bildrate der Kamera, der Writer und der Warteschlange) ausgegeben; bei einer niedrigeren Bildrate oder einem 8-Bit-Pixelformat werden alle Bilder beibehalten. Der Befehl meldet sich alle 2 s beim Backend; wenn er abstürzt, ohne die Aufzeichnung zu beenden (`kill -9`, Speicher knapp), stoppt das Backend die Aufzeichnung 30 s nach der letzten Meldung und versetzt die Kamera zurück (`stop_reason` `client_gone` sowie eine Zeile im Backend-Protokoll). `multi-stream` und `align-stream` verhalten sich ebenso. Ein Startvorgang, bei dem das Öffnen der Kamera länger dauert als die Wartezeit des Befehls (60 s; 180 s bei `multi-stream` / `align-stream`) – die Kamera bootet noch –, wird vom Backend rückgängig gemacht: Die Kamera wird zurückgesetzt und geschlossen, und es läuft keine Aufzeichnung mehr (504 HTTP-Fehler `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Browserbasierte Live-MJPEG-Vorschau: stellt eine Verbindung zur Kamera her, gibt die Stream-URL aus und öffnet sie in Ihrem Browser. `--ae-damping` legt die Dämpfung der automatischen Belichtung fest (0,4–100). |

##### Live-Streams (MJPEG; kein RTSP)

Chloros liefert Live-Videos als **MJPEG über HTTP**(`multipart/x-mixed-replace`, Grenze `frame`) aus dem Backend bereit.**RTSP wird nicht unterstützt.** Öffnen Sie eine URL-Adresse in einem Browser oder lesen Sie sie mit einem beliebigen MJPEG-Client ein (OpenCVs `cv2.VideoCapture(url)`, ffmpeg, VLC). Die Kamera muss zuvor im Backend verbunden werden (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` oder über die GUI).

| URL (`http://127.0.0.1:5000` + …) | Stream |
| --- | --- |
| `/api/camera/stream/<serial>` | Die Vorschau, bis zu ca. 30 fps. Für ein Array-Element legen `?max_h=` (Standard: `720`) und `?jpeg_q=` (Standard: `80`) die Höhe und die JPEG-Qualität fest. Dies ist die URL, die `lattice viewer` öffnet. |
| `/api/camera/<serial>/stream-annotated` | Dieselbe Vorschau mit den Overlays der Benutzeroberfläche: `zebra=1`, `crosshair=1`, `grid=3x3` (oder `9x9`), `histogram=1`, `peaking=1`, `spot=1`; ohne Abfrage die im Projekt gespeicherten Überlagerungseinstellungen für diese Kamera. |
| `/api/camera/<serial>/stream-reflectance` | 8-Bit-Reflexionsgrad pro Band: ρ = Pixel ÷ der Teil`X-Chloros-Pixel-Scale`-Header (255, sofern `refl_max` nicht ausgelöst wird). `fps` (Standard: `10`), `jpeg_q` (Standard: `95`), `max_h` (Standard: `0` = nativ), `band=<name>`, `align=1`. JPEG, also verlustbehaftet. Wird mit HTTP-Code 409 abgelehnt, wenn die Kamera keine Reflexionswerte liefert (eine RGB-Kamera, kein Kalibrierungspaket, kein DAQ-Messwert). |

- **Abgleich von Bildern zwischen Kameras.**Jedes Teil, das ein**Array-Element** sendet, enthält in allen drei Datenströmen die Header `X-Chloros-Group-Id` (den Synchronisationsgruppenzähler des Arrays), die Header `X-Chloros-Frame-Id` und `X-Chloros-Timestamp-Ns`. Teile von verschiedenen Kameras mit demselben `X-Chloros-Group-Id` stammen aus demselben Trigger. Eine eigenständige Kamera hat keine Gruppe und sendet keine Gruppen-ID. Browser ignorieren die Header der einzelnen Teile; ein Client, der den Multipart-Stream selbst analysiert, liest diese.
- **Angehalten, solange die Array-Ansicht geöffnet ist.** `/api/camera/stream/<serial>` eines Array-Mitglieds sendet einen statischen „Paused“-Frame, solange der kombinierte Live-Stream des Arrays geöffnet ist (z. B. die Array-Ansicht der GUI), und setzt die Übertragung fort, sobald dieser geschlossen wird.

#### Sensor-Einstellung

| Unterbefehl | Zweck |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Lesen/Schreiben eines beliebigen GenICam-Knotens. `--set ExposureTime=…` (`Gain=…`) ohne die Schalter `ExposureAuto` (`GainAuto`), die den Automatikmodus zunächst deaktivieren, wie es `lattice exposure --set` tut; ein Wert, den die Kamera ablehnt, gibt `Failed to set <node>: …` auf stderr aus und beendet `1`. `--get` und die standardmäßige gruppierte Ansicht geben unter `ExposureTime` / `Gain` einen Wert `note:` aus, solange dieser Automatikmodus läuft – siehe [Auslesen von Belichtung und Verstärkung im Automatikmodus](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Belichtung &amp; AE. Eine Belichtung (`--set`) oder ein Zielwert (`--brightness`) wird als Ihr Wert beibehalten (die automatische Belichtungssteuerung von Chloros lässt es unberührt); die Verstärkung bleibt unverändert, sodass im Automatikmodus die Anpassung an das Ziel fortgesetzt wird, bis Sie `lattice gain` festlegen. `--auto` gibt die Belichtung zurück, `--brightness-auto` gibt den Zielwert an die automatische Zielwertsteuerung von Chloros zurück. Ohne Flags liest es die Belichtungsknoten aus, mit denselben Werten wie `note:` und `configure --get`. Lehnt die Kamera einen `--set`-Wert ab, gibt sie `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` auf stderr aus — im Automatikmodus wird der ausgeschaltete Schreibmodus wiederhergestellt, anstatt auf „Aus“ zu bleiben — gibt keine Zeile „`Exposure: … us (auto OFF)`“ aus und beendet das Programm mit „`1`“. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Verstärkung und automatische Verstärkung. Bei einem `--set`-Wert, den die Kamera ablehnt, gibt sie `Failed to set Gain: …` auf stderr aus, keine `Gain: … dB (auto OFF)`-Zeile, und beendet das Programm mit `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Sensor-ROI und Binning. Das Binning erfolgt immer als **Durchschnitt** – siehe unten. |
| `lattice format [--set FMT] [--list]` | Pixelformat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardware-/Software-Trigger. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (keine Flags = One-Shot-Weißabgleich) | Weißabgleich-Vorgänge. Nur RGB-/Bayer-Kameras; bei Mono-M3M-Kameras wird dieser Schritt übersprungen. Der One-Shot-Modus behält den ermittelten Weißabgleich (`BalanceWhiteAuto` und die Verstärkungen) bei; alle anderen Einstellungen, auf die er zugreift, werden unverändert zurückgesetzt, und zwar in vollem Umfang, wenn der One-Shot-Modus nicht ausgeführt werden kann. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | RGB-Farbpipeline für die Anzeige. `natural` (Standard) ist die kostengünstige Live-Nachbearbeitung; `enhanced` fügt Defringing + Lebendigkeit + lokalen CLAHE-Kontrast für den vollständigen Hub-Parity-Look hinzu, bei etwa doppelt so hohen Verarbeitungskosten pro Bild, was zu einer niedrigeren **Live**-Bildrate führt – gespeicherte Aufnahmen erhalten in jedem Fall die vollständige Verarbeitung. Nur für RGB-/Bayer-Kameras; wird bei Mono-M3M übersprungen. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Sättigung/Kontrast in der Live-Vorschau (Kameras mit RGB-Filter); `100` ist neutral, `--reset` setzt beide Werte auf 100. Gespeicherte Aufnahmen sind davon nicht betroffen. Wird bei Mono-M3M übersprungen. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (keine Flags = lesen) | **Highlights** anzeigen (Kameras mit RGB-Filter): Die hellsten, weißesten Pixel werden gedämpft – z. B. eine strahlend weiße Oberfläche, die als flacher weißer Fleck dargestellt wird. `0` = aus, `-100` = am stärksten. Die Live-Vorschau und die Aufnahmen auf Anzeigeebene richten sich danach; Strahlungs- und Reflexionswerte tun dies niemals, und es können keine Sensor-Daten wiederhergestellt werden, die übersteuert wurden (verringern Sie hierfür die Belichtung). Wird bei Mono-M3M übersprungen. |
| `lattice filter [--list]` | Listet die Filtermodelle auf, die Chloros kennt (`RGN-IMX265`, `OCN`, `NGB`, …). Der Filter einer Kamera wird aus ihrer Modellzeichenfolge ausgelesen und kann nicht eingestellt werden: `--set NAME` sucht lediglich nach einem Modell und beendet dann `1` mit `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Strom- und Temperaturknoten prüfen; Energiesparmodus umschalten. `--sleep` versetzt die Kamera in den Ruhezustand (die Begrenzung des Link-Durchsatzes ist aktiviert (`DeviceLinkThroughputLimitMode` `On`) auf ihrem Mindestwert, der Energiesparmodus ist aktiviert, sofern die Kamera über einen solchen verfügt) wobei alle anderen Einstellungen so wiederhergestellt werden, wie sie ursprünglich vorgefunden wurden; ein fehlgeschlagener Ruhezustand wird vollständig zurückgesetzt. Das Aufwachen versetzt die Verbindung wieder in den Zustand, in dem sie beim Eintritt in den Ruhezustand vorfand (die Begrenzung, ihren Modus und die Paketverzögerung `GevSCPD`, die die Kamera auf 0 hält, solange die Begrenzung aktiviert ist; der Datensatz wird auf diesem Computer gespeichert, sodass ein Aufwachen nach dem Neustart des Backends diese ebenfalls vorfindet), oder bei einer Kamera, die dieser Computer nicht in den Ruhezustand versetzt hat, die aber im Ruhezustand zu sein scheint (ihr Limit ist auf dem Mindestwert aktiviert), wird das Limit deaktiviert und ein `GevSCPD` bei 80 auf 0 belassen (die Werkseinstellungen); ein über dem Mindestwert festgelegtes Limit bleibt unverändert. Die Sonde und das Aufwachen setzen die Werte zurück, die bei ihrer Öffnung geschrieben wurden. |

> **Das Binning erfolgt immer als Durchschnitt.** `lattice resolution --binning N` legt das durchschnittliche Binning fest und bietet
> keine Modusumschaltung an. Beim Sum-Binning wird jeder DN mit N² multipliziert, was pro Binning-Schritt etwa zwei Blendenstufen an
> Headroom in den Lichtern kostet; Chloros dividiert diesen Faktor vor der
> radiometrischen Kette wieder heraus, aber Pixel, die durch „Sum“ bis an die Obergrenze des Sensors getrieben wurden, werden abgeschnitten und können nicht
> wiederhergestellt werden. Es bringt nichts, was „Average“ nicht auch bietet – ein Index ist skaleninvariant und die Strahlungsintensitäts-
> Skalare sind auf die „Average“-Skala kalibriert. Wenn eine Kamera bereits auf „Sum“ eingestellt ist (entweder
> durch ein anderes Tool oder durch einen Werksreset), erkennt Chloros dies anhand eines Knoten-Readbacks und
> gleicht dies aus; durch Ausführen von `lattice resolution --binning N` wird die Einstellung wieder auf „Average“ zurückgesetzt. Das 1.2.0
> `--binning-mode`-Flag wird weiterhin akzeptiert, sodass ältere Skripte laufen, es wird jedoch nur eine Warnung ausgegeben:
> Das Binning bleibt auf „Durchschnitt“ eingestellt.

> **Änderungen an Binning, Pixelformat und Trigger bei einer Kamera werden nicht beibehalten.** Bei einer Kamera, bei der das
> Backend offen bleibt und Daten streamt (`cam-connect`, die GUI), wird `lattice resolution --binning`
> wird nicht wirksam (es wird das weiterhin gültige Binning ausgegeben, `Binning: 1x1`), und
> `lattice format --set` sowie `lattice trigger --mode` werden von der Kamera abgelehnt
> (`SC_ERR_ERROR -1001`). Geben Sie diese zunächst frei (`lattice cam-disconnect -s <SN>`) oder legen Sie
> sie bei der Verbindung fest (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Auslesen von Belichtung und Verstärkung im Automatikmodus

Während der `ExposureAuto` (`GainAuto`) einer Kamera läuft, behält der Knoten `ExposureTime` (`Gain`) den **zuletzt geschriebenen Wert** und nicht den aktuellen Betriebszustand der Kamera. Für eine Kamera, deren Backend geöffnet ist (`cam-connect`, `array-connect`, die GUI), geben `lattice configure --get`, die Standardansicht `lattice configure` und `lattice exposure` (ohne Flags) daher den Knotenwert aus und darunter einen Hinweis mit dem neuesten Bild der Kamera:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Lesen Sie den Bildwert aus dem Hinweis aus; der Knotenwert entspricht nur dem letzten Schreibvorgang. Über HTTP bleibt bei derselben Antwort „`POST /api/camera/<serial>/read-nodes`“ der Wert „`nodes`“ unverändert, und es werden „`frame_values`“ (`{"ExposureTime": <us>, "Gain": <dB>}`, das des neuesten Bildes) und `notes` (der obige Satz) hinzu. Beide sind nur vorhanden, solange der Automatikmodus nicht `Off` ist und ein Bild bekannt ist; eine nur zum Lesen geöffnete Kamera hat kein Bild und erhält keines von beiden. `lattice gain` (keine Flags) gibt nur den Knoten aus – verwende `lattice configure --get Gain` für die Notiz.

##### Belichtung eines Array-Elements

Während ein Array streamt, läuft jedes Element mit **`AcquisitionFrameRateEnable`**mit einer Bildrate, die etwas über der Triggerrate des Arrays liegt: Seine Periode entspricht der Triggerperiode minus 15 % (mindestens 3 ms) oder der maximalen Bildrate der Kamera selbst, wenn der Trigger bereits nahe daran liegt. `configure --get` auf einem Element liest daher bei dieser Einstellung `AcquisitionFrameRateEnable` als `True` und `AcquisitionFrameRate`; die Freigabe wird wieder deaktiviert, wenn das Array steht.**Schreiben Sie diese beiden Knoten nicht auf ein Array-Element** – das Array hält sie im Takt mit seinem Trigger und schreibt sie neu.

Diese Einstellung ist auch die Belichtungsobergrenze des Elements: etwa 85 ms bei einem Trigger von 10 fps (Periode von 100 ms), bei **beliebigem ROI oder Binning**. Vor Version 1.3.0 begrenzte ein kleiner ROI die Belichtungszeit eines Elements stattdessen auf die Freilaufperiode des Sensors (4,9 ms bei 768 × 304), unabhängig davon, wie langsam das Array triggert.**Ein manuelles `ExposureTime`, das länger als die Obergrenze ist, verlangsamt den Trigger, um sich daran anzupassen**, wenn es mit `lattice exposure --set` oder `configure --set` (oder über die GUI) geschrieben wird. Das Array öffnet zunächst die Obergrenze dieses Mitglieds für den Wert und lässt die langsamere Rate einige Triggerperioden lang laufen, sodass der Schreibvorgang einige Sekunden länger dauern kann; der Trigger feuert dann nicht schneller, als es die Belichtung zulässt. Bei einem Array mit automatischer Rate kehrt die zuvor verwendete Rate zurück, sobald jedes Mitglied, dessen manuelle Belichtung das Array verlangsamt hat, wieder auf automatische Belichtung umgeschaltet ist. Ein Wert, den die Kamera selbst nicht aufnehmen kann, wird weiterhin abgelehnt, wobei der akzeptierte Bereich in der Fehlermeldung angegeben wird, und die automatische Belichtung des Mitglieds wird wiederhergestellt, falls sie nur für diesen Schreibvorgang ausgeschaltet war. Eine Belichtung nach dem Rezept `apply` verlangsamt den Auslöser nicht: Sie wird auf die Obergrenze gehalten. Chloros verkürzt niemals eine manuelle Belichtung, um sie an einen schnelleren Trigger anzupassen; eine festgelegte Rate (`array-connect --target-fps F`), die schneller ist, als es eine Belichtung zulässt, belässt diesem Mitglied eine Obergrenze, die lang genug dafür ist, und dieses Mitglied nimmt Trigger nicht schneller auf, als es seine Belichtung zulässt.

#### Kalibrierung &amp; Sensoren

| Unterbefehl | Zweck |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrierung anhand eines Reflexionsziels. Es wird in bis zu `--attempts` Frames (Standard: `30`) danach gesucht; ein unvollständiger Frame (GVSP-Paketverlust) kostet einen Versuch, nicht den gesamten Durchlauf. Der Vorgang schlägt bei `no target detected in N attempts` fehl, oder bei `no complete frame in N attempts: <last error>`, wenn kein vollständiger Frame angekommen ist. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Das Spektrum und die Bandamplituden eines DAQ-M, die über Bluetooth (BLE) vom Backend ausgelesen werden. `--mac` ist die sensoreigene BLE-MAC-Adresse und wird bei jeder der drei Aktionen benötigt: Es gibt keinen Standardwert, da jedes Gerät eine eigene hat (`daq pool-discover --only ble` listet diese auf); ohne diese wird der Befehl mit `1` beendet. Ohne Angabe einer Aktion wird die Hilfe ausgegeben. `--camera SERIAL` integriert die ausgegebenen Bandamplituden anhand der kameraeigenen gemessenen Spektralempfindlichkeit anstelle der Flotte-Kurve des Filtermodells – bei RGB-Kameras, sofern das Kalibrierungspaket der Kamera geladen (verbunden) oder auf diesem Computer zwischengespeichert ist. In der Überschrift wird angegeben, welche Kurve verwendet wurde: `Band magnitudes (camera <SERIAL>):` oder `Band magnitudes (model <filter>):`, wenn die Kamera keine hat (keine RGB-Kamera, oder kein Paket geladen oder zwischengespeichert ist) und stattdessen die Standardkurve verwendet wurde. Über HTTP nimmt `POST /api/dls/cli-read` `camera_serial` entgegen und gibt `band_magnitudes_source` zurück. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Wendet eine Vignettierungskorrektur auf vorhandene Bilder an, wobei jedes Bild das Flat-Field-Bild seines eigenen Objektivs verwendet (aus dem EXIF-Modell). `--lens-model` gilt für Bilder, deren Modell keine Kamera angibt; ein Bild, dessen Modell dem widerspricht, wird übersprungen. |

#### Mehrkamera (vorübergehende Sitzungen)

Ein temporärer Befehl öffnet die Kameras für einen Durchlauf und schließt sie nach dessen Ende: `multi-capture` ohne verbundenes persistentes Array, `multi-stream` und `multi-test` hier, sowie `align-calibrate`, `align-apply`, `align-stream` und `index --live` weiter unten. Für alle gelten folgende Regeln:

- **`--serials SN1,SN2,…` wählt die Kameras aus.** Nur diese werden geöffnet, und die erste löst den Auslöser aus (für `align-calibrate`, `--reference`, sofern angegeben). Eine serielle Kamera, die nicht im Netzwerk ist, führt zum Fehlschlagen des Durchlaufs, bevor eine Kamera geöffnet wird. Ohne `--serials` wird jede Kamera im Netzwerk geöffnet. Eine Kamera, die Chloros bereits hält (die GUI, `cam-connect`, `array-connect`), lehnt den Lauf ab, unabhängig davon, ob `--serials` sie nennt oder nicht: Sie nutzt dasselbe Netzwerk, für das der Lauf ausgelegt ist. (`index --live` ohne `--serials` öffnet stattdessen die in seinem Profil aktivierten Kameras: siehe [Index / Vegetationsmathematik](#index--vegetation-maths).)
- **Die Kameras teilen sich das Netzwerk so, wie `array-connect` es teilt.** Jede Kamera wird auf ihren Anteil an der Übertragungskapazität der Host-Verbindung abgestimmt (eine Verzögerung zwischen den Paketen, nicht die Durchsatzgrenze der Kamera, deren Untergrenze von 31,25 MB/s vier Kameras über 1 GbE bereits überschreiten), sodass vier Kameras über eine 1-Gbit/s-Verbindung laufen. Wenn die Kameras nicht alle gleichzeitig mit der langsamsten von ihnen erreichten Geschwindigkeit senden können, werden sie nacheinander ausgelöst, wobei jede eine eigene Verbindung erhält, und der Befehl gibt „`Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`“ aus. Wenn auch nur eine Kamera auf einmal nicht passt, wird der Durchlauf abgelehnt, bevor eine Kamera streamt: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP-Fehler 409, Beenden `1`.
- **Jede Kamera muss Daten liefern, sonst schlägt der Durchlauf fehl.** Eine Kamera, die kein Bild liefert, stellt einen Fehler dar: Der Befehl gibt „`Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`“ aus, wobei „`<why>`“ den Abruf-Fehler dieser Kamera angibt, gefolgt von den möglichen Ursachen, und beendet den Vorgang mit „`1`“. Ein Frame, der mit GVSP-Paketverlust (`IncompleteBufferError: GVSP packet loss …`) angekommen ist, bedeutet, dass das Netzwerk die kombinierte Bandbreite der Kameras (`Every missing frame arrived with GVSP packet loss: …`) nicht bewältigen kann: Überprüfen Sie die Verbindungsgeschwindigkeit des Hosts und des Uplinks des Switches (`lattice network`), verwenden Sie Jumbo-Frames, oder öffnen Sie weniger Kameras mit `--serials`. Eine Kamera, die nichts geliefert hat, erhält in der Regel keinen Trigger vom Master: Überprüfen Sie ihr M8-Synchronisationskabel und führen Sie anschließend `lattice multi-detect` aus. Was die einzelnen Befehle bei einer Frame-Gruppe tun, in der eine Kamera fehlt, ist in der jeweiligen Zeile angegeben.
- **Jede Kamera wird so zurückgesetzt, wie sie vorgefunden wurde.** Wenn der Durchlauf endet, erhält jede Kamera wieder den Zustand, den sie hatte, als der Durchlauf sie geöffnet hat: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, sowie `LineMode` / `LineSource` auf `Line2` und `Line3` (und die Entstörung des Triggereingangs durch ein Array liefert die Leitung eines Slaves, `LineFilterSelector` / `LineFilterWidth`); sein `PtpEnable` (und die damit festgelegten `PtpSlaveOnly` / `PtpDomainNumber`); `ExposureAuto` / `GainAuto` (sowie `ExposureTime` / `Gain`, bei denen ein manueller Wert eingestellt war); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (sowie alle `BalanceRatio`, bei denen der Weißabgleich manuell eingestellt war); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, Binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (sowie `Gamma`, bei denen Gamma aktiviert war), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; die Paketgröße (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); die Pacing-Einstellung, `DeviceLinkThroughputLimitMode` (und `DeviceLinkThroughputLimit`, wenn der Modus `On` war), diePaketverzögerung `GevSCPD` und die Startversetzung `GevSCFTD`, die ein Array für jede Kamera liefert; die Chunk-Daten, die jeder Stream aktiviert (`ChunkModeActive`, jeweils `ChunkEnable` unter seinem `ChunkSelector`); `AcquisitionFrameRateEnable` (und `AcquisitionFrameRate`, wo die Freigabe aktiviert war – ist sie deaktiviert, ist die Rate schreibgeschützt und gibt den maximalen Wert an, den das Timing zulässt; es handelt sich also nicht um eine Einstellung). Es wird nur das geschrieben, was sich unterscheidet; die Kamera wurde zunächst angehalten, die Taktung und Bildrate vor einer Belichtung, die ihre Bildperiode benötigt (eine Kamera begrenzt `ExposureTime` auf eine Bildperiode). Bei `--keep-settings` werden die Trigger- und Sync-Leitungen zurückgeschaltet, die Kamera läuft im Freilauf, und der Rest wird gemäß den Lauf-Einstellungen im kamerainternen Speicher gespeichert; `--clear-settings` setzt dort die Werkseinstellungen.
- **Die Belichtung wird anhand der Szene vor der Aufnahme des ersten Bildes festgelegt.** Der Durchlauf nimmt synchronisierte Bildgruppen auf und verwirft sie, während er die Belichtung und Verstärkung jeder Kamera anhand dieser Bilder einstellt. Das Ziel ist eine mittlere Helligkeit auf dem Ziel der Kamera, wobei nicht mehr als 1 % des Bildes überbelichtet sein darf; eine Szene mit hellen Lichtern bleibt hinter dem Ziel zurück. Dies dauert in der Regel 2–6 Bildgruppen und höchstens 5 s.
  - Jede Kamera richtet sich auf das Ziel, bei dem ein angeschlossener Array den Vorgang startet, entsprechend ihrem Filter: TargetBrightness **40**bei einer Multispektralkamera (RGN, OCN, NGB) und**60** bei einer RGB-Kamera. Das entspricht der Hälfte des von 80 vorgegebenen Belichtungsziels (drei Viertel bei RGB), sodass ein helles Reflexionspanel weniger wahrscheinlich übersteuert wird. `--brightness N` (0–255) legt stattdessen den Zielwert jeder Kamera fest (`multi-test` hat kein `--brightness`: Die automatische Belichtung ist deaktiviert). Die Protokolle des Backends benennen den Belichtungswert jeder Kamera: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  – Die Belichtung steht an erster Stelle, bis zur Grenze der kamerainternen automatischen Belichtung. Die Verstärkung deckt nur den verbleibenden Bereich ab.
  - Die ermittelte Belichtung und Verstärkung werden für den gesamten Durchlauf beibehalten, sodass jedes Einzelbild einer `multi-stream` mit derselben Belichtung aufgenommen wird. Eine Aufnahme, die sich ändernden Lichtverhältnissen folgen muss, gehört in ein verbundenes Array (`array-connect`), dessen automatische Belichtung sich ständig anpasst.
  - `--exposure` und `--gain` werden genau wie angegeben beibehalten. Sind beide angegeben, wird kein Einzelbild verworfen.
  – Nach Beendigung des Durchlaufs wird die automatische Belichtung jeder Kamera wieder aktiviert.
  - Das Protokoll des Backends meldet das Ergebnis: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` oder `exposure NOT solved after …` mit der Begründung sowie `(not settled)` für jede Kamera, bei der dies nicht der Fall war. Die Bilder werden in diesem Fall weiterhin aufgenommen.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` ändert das 5-Sekunden-Limit; `0` deaktiviert diese Funktion (siehe [Umgebungsvariablen](#environment-variables)).

**Solange Chloros eine beliebige Kamera hält** (die GUI, `cam-connect` oder ein angeschlossenes Array), werden `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` und `index --live` die Ausführung ab, da sie jede Kamera im Netzwerk öffnen und das zweite Öffnen einer bereits gehaltenen Kamera das Backend zum Absturz bringen kann. (`index --live` bricht ab, wenn eine Kamera, die es öffnen würde, bereits gehalten wird: entweder `--serials` selbst oder die Kameras seines Profils sowie jede Kamera eines Profils, das keine Kameras angibt.) Das Backend antwortet mit dem HTTP-Statuscode 409 für `CAMERA_HELD` und nennt dabei jede belegte Kamera sowie die Befehle zum Freigeben (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`) auf, woraufhin der Befehl beendet wird (`1`). Wenn ein Array angeschlossen ist, speichert `array-capture --continuous` dieses anstelle von `multi-stream`, und `array-capture` (standardmäßig ausgerichtet) speichert einen ausgerichteten Satz anstelle von `align-apply`; `multi-capture` und `align-calibrate` verwenden ein verbundenes Array, das ihre Kameras enthält, anstatt diese zu öffnen.

| Unterbefehl | Zweck |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Listet alle Kameras mit Synchronisationsrollen auf; mit `--serials` nur diese, die erste als `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Ein synchronisierter Frame von jeder Kamera. Speichert **standardmäßig alle Exporttypen**, wenn ein persistentes Array angeschlossen ist, sowie den verwendeten DAQ-Messwert ([Der DAQ-Messwert, den eine Aufnahme speichert](#the-daq-reading-a-capture-saves)); der vorübergehende Fallback ohne Array wird**nur debayert** (führen Sie für den Rest zuerst `array-connect` aus). `--brightness` gilt nur für den Fallback: Ein angeschlossenes Array behält seine eigene automatische Belichtung bei. Der Fallback speichert keine `.daq` (`--no-daq` / `--daq-tolerance-ms` haben dort keine Wirkung), und `multi-capture` übernimmt `--force-daq` nicht. Im Fallback scheitert die Aufnahme bei einer Kamera, die kein Bild liefert (`Error: multi-capture failed: no frame from …`, Beenden von `1`), und es wird keine Datei dieser Bildgruppe gespeichert. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Synchronisierte Frames streamen (vorübergehend), `--fps` Standard `3`, bis Strg+C oder `--count`-Gruppen. Eine Bildgruppe, für die eine Kamera nichts geliefert hat, wird nicht geschrieben, und drei aufeinanderfolgende beenden den Stream. Wenn eine Gruppe übersprungen wurde, endet der Lauf durch die Ausgabe von `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)` sowie `Error: the stream stopped: …`, wobei die Kamera genannt wird, bei der der Stream beendet wurde, und das Programm wird mit `1`X. Beendet sich selbst, wenn das Laufwerk voll ist, gibt „`Failed:`“ / „`Disk full:`“ aus und beendet das Programm mit „`1`“; jede Datei, die nicht geschrieben werden konnte, beendet ebenfalls `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO-Synchronisations-Timing-Test (transient), `--count` Standard-`10`-Trigger. Gibt aus, wie das Netzwerk gemeinsam genutzt wird (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)` sowie die Zeile `Serial emit:`, wenn die Kameras nacheinander ausgelöst werden, wobei die Zeitstempel dann diese zeitliche Versetzung widerspiegeln), eine Zeile `Sync group: master SN:<serial> -> SN:<serial>, …` pro Synchronisationsgruppe (`(no slaves: not synced)` für eine Kamera ohne Synchronisationskabel, die eigenständig ausgelöst wird), dann eine Zeile pro Auslösung: `[  0] SN:<serial>=<timestamp> \| …`, wobei `SN:<serial>=(no frame)` für eine Kamera steht, die bei dieser Auslösung nichts geliefert hat. Dann eine Zeile „`Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):`“ und eine Zeile „`SN:<serial>: mean <±x> us, jitter <y> us`“ pro Kamera, wobei jede Kamera innerhalb eines Takts mit dem Master synchronisiert ist: Ein Trigger, bei dem die Zeitstempel die Gerätetakte der Kameras und PTP (PTP ausgeschaltet oder noch beim Synchronisieren) wird ausgelassen, wobei eine Zeile „`Note:`“ diese zählt. Jeder verpasste Frame führt zum Fehlschlagen des Tests (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, Beenden von `1`); drei unvollständige Trigger in Folge beenden ihn vorzeitig. Der Test schlägt ebenfalls fehl (Beenden von `1`), wenn bei weniger als zwei Triggern nicht alle Kameras auf einem Takt liegen (`… the clocks cannot be compared`) oder wenn die Takte niemals übereinstimmen: PTP-Taktfrequenzen, die mehr als 1 ms (zuzüglich etwaiger Versätze auf der seriellen Ebene) vom Master abweichen (`… PTP is not locked`), oder Gerätetaktfrequenzen, deren Versatz sich zwischen den Triggern um mehr als diesen Wert verschiebt. Es startet den PTP-Grandmaster, falls dieser nicht läuft, aktiviert das PTP jeder Kamera (wie es `array-connect` tut, unabhängig von der Netzwerkkonfiguration) und wartet vor dem ersten Trigger ab, bis sich die PTP-Taktfrequenz jeder Kamera auf den Grandmaster synchronisiert und stabilisiert hat: `PtpStatus` Slave auf einem Grandmaster, innerhalb von 100 µs davon und 3 s lang stabil, und jede Kamera 120 s lang – etwa 2 Minuten ab dem Einschalten. Ein Takt, der sich gerade synchronisiert hat, weicht nach dem Lesen von „Slave“ etwa 2 Minuten lang um bis zu ~200 µs ab, und der kamereneigene `PtpOffsetFromMaster` zeigt dies nicht an; nach 120 s liegen die Kameras innerhalb von ~10 µs eines stabilen Takts. Eine Kamera ist seit einem früheren Test, bei dem PTP eingeschaltet blieb – so wie es dieses Backend sah – zählt ab diesem Synchronisationszeitpunkt weiter, sodass bei zwei direkt aufeinanderfolgenden Tests keine erneute Wartezeit entsteht; eine Kamera, die erst durch den Test eingeschaltet wird, oder eine mit aktiviertem PTP, deren Synchronisation dieses Backend noch nicht erkannt hat (die an anderer Stelle oder vor einem Neustart des Backends eingeschaltet wurde), wartet ihre 120 s ab, und in einem Array mit beiden Kameras warten die gerade synchronisierten Kameras weiterhin ihre Zeit ab. Eine Kamera, die den Slave-Modus verlässt, die 100 µs überschreitet oder während der Wartezeit drei Messwerte in Folge (1,5 s) einem anderen Grandmaster folgt, beginnt erneut mit dem Zählen; ein oder zwei solcher Messwerte werden protokolliert (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) und der Zählvorgang läuft weiter; eine Grandmaster-ID, die nicht gelesen werden kann, wird als unbekannt gewertet, nicht als Änderung. Wenn der Zählvorgang aller Kameras gleichzeitig (innerhalb einer Sekunde) neu startet, liegt dies an der Zeitverschiebung des Grandmasters, nicht an den Kameras: Das Backend-Protokoll zeigt „`[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`“ an, `ptp_lock` listet sie in `grandmaster_steps` auf, und eine Wartezeit, die dann abläuft, beginnt ihren Fehler mit `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Die Grenze wird dafür nicht erweitert. `--ptp-settle SECONDS` legt die Abwicklungszeit für den Lauf fest (0–600; `0` für keine Abwicklungswartezeit). Die Wartezeit ist auf die Einstellzeit plus 90 s begrenzt (standardmäßig 210 s, mindestens 90 s), und der Befehl wartet 240 s plus die Einstellzeit auf das Backend (standardmäßig 360 s). Es gibt „`PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`“ aus, und das Protokoll des Backends enthält alle 5 s eine Zeile mit dem Status und dem Offset jeder Kamera, wie lange sie bereits gesperrt ist und wie lange die verbleibende Zeit beträgt: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Jede Kamera wird mit deaktiviertem PTP eingeschaltet; daraufhin synchronisieren sich die Uhren in etwa 10 s. Danach wird jede Kamera wieder in den Zustand versetzt, in dem der Test sie vorgefunden hat – ihr PTP, die automatische Belichtung und die Verstärkung, die der Test für den Durchlauf auf „Aus“ gesetzt hat, sowie der Rest (sofern nicht `--keep-settings`) – bleibt also eine Kamera, bei der PTP aktiviert ist (`lattice configure -s <serial> --set PtpEnable=True` oder ein Durchlauf mit `--keep-settings`), synchronisiert, und sobald dieses Backend festgestellt hat, dass sie für die Stabilisierungszeit synchronisiert war, wartet ein späterer Test nicht mehr auf sie. Wenn sich die Takte während der Wartezeit nicht stabilisieren, schlägt der Test vor der Auslösung fehl (`PTP: NOT locked …`, Beenden `1`) und gibt an, was zu tun ist: Eine Kamera, die nie verriegelt wurde (`PtpStatus Listening`), bedeutet, dass der Grandmaster sie nicht erreicht (`time-sync status`); eine, die immer wieder ausfiel, wird mit der Dauer ihrer Verriegelung benannt (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); Uhren konvergieren noch: Überprüfen Sie `time-sync status`, und um sie vor dem Test zu synchronisieren, schalte PTP ein und lasse es eingeschaltet (`lattice configure -s <serial> --set PtpEnable=True` für jede Kamera oder führe den Test mit `--keep-settings` durch; der Test stellt die gefundenen Einstellungen wieder her, sodass PTP eingeschaltet bleibt), dann führe den Test durch. |
| `lattice multi-detect [--line LINE] [--json]` | Die Kameras auf der M8-Synchronisationsleitung und einen Standard-Master ermitteln: die RGB-Kamera (Modellbezeichnung endend auf `-FRGB`), sofern sie verkabelt ist, andernfalls die erste verkabelte Kamera in der Reihenfolge der Seriennummern. Die M8-Kabel laufen an einem Knotenpunkt zusammen, daher **kann jede verkabelte Kamera als Master fungieren** – die aufgedruckte Bezeichnung „`MASTER`“ ist ein Standardwert und keine Angabe zur Verkabelung. Jede Kamera treibt der Reihe nach die Leitung an, und die anderen werden auf ihren Impuls überprüft; eine Kamera, die bereits in der verkabelten Gruppe gefunden wurde, wird nicht erneut abgefragt, sodass eine vollständig verkabelte Anordnung einen Durchlauf benötigt. Fehlt „`--line`“, wird „`Line2`“ abgefragt, dann `Line3`. Es werden `MASTER: SN:…`, jeweils ein `SLAVE: SN:…` pro weiterer verkabelter Kamera und `(probed on <line>)` ausgegeben; `--json` fügt das vollständige Ergebnis hinzu (`master`, `slaves`, `no_cable` – Kameras, die der Impuls nicht erreicht hat – `line` sowie pro Kamera `results`). Nach Abschluss des Vorgangs erhält jede Kamera ihre ursprünglichen Trigger- und Sync-Line-Einstellungen zurück (die gleichen Knoten wie oben). |

#### Ausrichtung

| Unterbefehl | Zweck |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plus Detektor-/Matcher-Regler `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-Parameter `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, Mehrbildkombination `[--averaging mean\|median\|inlier_weighted]`, geometrische Einschränkungen `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, räumliche Einschränkung `[--roi X0,Y0,X1,Y1] [--mask PATH]` sowieSlave-Überschreibungen `[--per-cam-override SN:KEY=VALUE]` (wiederholbar) | Berechne das Ausrichtungsprofil anhand von Live-Kameras. Standardwerte: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (in dem Ordner, aus dem Sie das Programm ausführen), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` hat eigene Standardwerte; siehe [seine Tabelle](#project-align-calibrate-options).) `--prefilter` verwendet standardmäßig `gradient` (Kantenkarte; entspricht dem GUI-/Array-Aligner – Kanten bleiben über Spektralbänder hinweg erhalten). `--matcher flann` zahlt sich ab ca. 5000 Merkmalen aus; `--averaging median` ist robust gegenüber einer fehlerhaften Aufnahme, `inlier_weighted` gewichtet nach der Anzahl der Übereinstimmungen; `--lock-scale` projiziert auf die nächstgelegene Rotation (keine Skalierung), `--lock-axis` setzt eine Translationskomponente auf Null; `--mask` gilt für jede Kamera (verwenden Sie `--per-cam-override` für kameraspezifische Einstellungen, z. B. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` verweigert das Speichern einer Kalibrierung, deren RMS-Wert bei der Reprojektion den Schwellenwert überschreitet. `--method manual` bezieht seine Punkte aus `--points PATH`: ein JSON-Objekt `{"<serial>": [[x, y], ...], ...}`, das die Pixelkoordinaten derselben Merkmale für jede Kamera angibt, einschließlich der Referenzkamera (mindestens 2 pro Kamera); jedes der `--frames` wird aus diesen Punkten berechnet. Wenn ein verbundenes Array die Kameras enthält, verwendet die Kalibrierung die Live-Bilder dieses Arrays, und das Profil wird gleichzeitig zur Ausrichtung des Arrays für dessen Aufnahmen. Andernfalls wird ein [temporäres Array](#multi-camera-transient-sessions) geöffnet: Eine Kamera, die in keinem der `--frames`-Frames ein Bild liefert, führt zum Fehlschlagen der Kalibrierung (Beenden von `1` mit Angabe des Namens), und es wird kein Profil gespeichert. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Ein ausgerichtetes Multiband-Bild aufnehmen. `--bit-depth` passt sich standardmäßig an die Kamera an; `--no-crop` behält das gesamte Bild bei (mit Schwarz aufgefüllt); `--interpolation` (Standard: `linear`) und `--border-mode`/`--border-value` (Standard: `constant`/0) steuern die CPU-Verzerrung – der GPU-Pfad ist in jedem Fall bilinear. Öffnet ein [transientes Array](#multi-camera-transient-sessions): Eine Kamera, die kein Bild liefert, führt zum Abbruch der Aufnahme (Beenden von `1` unter Angabe des Namens), und es wird nichts geschrieben. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Stream-ausgerichtete Multiband-Frames (gleiche Warp-Regler wie `align-apply`), `--fps` Standard, `3`. Öffnet ein [Transient-Array](#multi-camera-transient-sessions): Eine Bildgruppe, für die eine Kamera nichts geliefert hat, wird nicht gespeichert, und drei aufeinanderfolgende Bilder beenden den Stream. Wurde eine Gruppe übersprungen, endet der Lauf durch die Ausgabe von `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` und `Error: …` und wird mit „`1`“ beendet. Ein Stream, der aufgrund eines anderen Fehlers endet, gibt ebenfalls „`Error: …`“ aus und wird mit „`1`“ beendet. |
| `lattice align-info --profile PATH [--json]` | Profildetails anzeigen. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Reihenfolge der Ebenen ändern. |

#### Index / Vegetationsmathematik

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` ordnet die Symbole einer Voreinstellung den Bändern zu** (NDVI: `red`, `nir`), jeweils ein `--channel SYMBOL=BAND`.

- **`--live`: nur für ein Symbol, das der Stack nicht zuordnen kann.** Wie bei `--input` erhält ein Symbol einer Voreinstellung, das Sie weglassen, das eine Band des Stacks (die Bänder der aktivierten Kameras des Profils), dessen Name damit beginnt. Dies wird vor dem Öffnen einer Kamera überprüft: Ein Symbol, auf das kein Band oder mehrere Bänder reagieren, wird dann abgelehnt, wobei diese namentlich genannt werden, und `1` wird beendet. Ein benutzerdefiniertes `--formula` benötigt für jedes Symbol ein `--channel`.
- **`--live`-Produkte geben an, wem sie gehören.** `index.tif` und `index_LUT.png` enthalten die Daten der Kamera im Stapel (Hersteller, Modell, Seriennummer), deren Aufnahmezeit (Belichtungszeit der Referenzkamera, UTC) sowie das Objektiv; beide benennen den Index als ihr einziges Band (`XMP-Camera:BandName` `NDVI`; die Farb-PNG-Datei benennt den Index, nicht ihre Farbkanäle), genau wie ein `--input`-Produkt. Ein Index aus den Bändern mehrerer Kameras benennt stattdessen jedes einzelne davon, und sein `Model` lautet `LATTICE` – siehe [Dateien aus mehreren Kameras](#files-made-from-several-cameras).
- **`--input`: nur für ein Symbol, das die Datei nicht zuordnen kann.** Ein Symbol, das Sie weglassen, bezieht sich auf das eine Band der Datei, dessen Name damit beginnt: `red` nimmt `Red_660` oder `Red_660_RGN` an, `nir` nimmt `NIR_850` an. Die Namen stammen aus der Datei selbst: aus der stack-eigenen Bandliste oder, bei einer Chloros-Strahlungs-/Reflexionsaufnahme oder einem `process`-Produkt deren Tags `Chloros:Band{R,G,B}Name` bzw. `XMP-Camera:BandName`. Daher benötigt `lattice index --input <reflectance capture> --preset NDVI` kein `--channel`.
- **Zwei Bänder, die auf ein Symbol verweisen, werden nicht voneinander unterschieden** (ein 9-Kamera-Stack verfügt über zwei NIR-Bänder): Benennen Sie dieses Band. Ein Symbol, dem kein Band zugeordnet ist, wird mit „`channel_map missing entries`“ abgelehnt; beenden Sie „`1`“.**Bandnamen in einem Stapel** tragen den Filter der Kamera: Die Bänder einer M3C-Kamera lauten `<band>_<filter>` – `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` bei einem RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` bei einem OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` bei einem RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` auf einem NGB – und dasEinzelband ist `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` listet die Bänder eines Profils auf.

Vollständiger Flagsatz: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (wiederholbar), `--capture-level raw|debayered|radiance|reflectance|unknown` (überschreibt die im Quell-TIFF-Datei aufgezeichnete Erfassungsstufe; Standard: aus den TIFF-Metadaten gelesen, andernfalls wird auf den Wert einer Chloros-Datei zurückgegriffen: `Chloros:ProcessingLevel`, sodass eine Reflexionserfassung den Wert `reflectance` angibt), `--output PATH` / `-o` (offline: ein Pfad; bei einer Bilddateiendung werden die Dateien anhand ihres Stamms benannt – `-o ndvi_FRGN.png` schreibt `ndvi_FRGN.tif` und `ndvi_FRGN_LUT.png` – und ohne diese wird ein Präfix verwendet, `-o ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`; Standardwert neben der Eingabe, wie `<input name>_<preset>`; mit `--live` und ohne `-o`, einem Ordner „`output/`“ unterhalb des Ordners, aus dem Sie den Befehl ausführen), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Bei `--live` gelten die Regler für die Ausrichtungsverzerrung ebenfalls: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (das Ziel der automatischen Belichtung der Kameras, siehe die Regeln für vorübergehende Sitzungen), `--serials SN1,SN2,…`. `--live` öffnet ein [transientes Array](#multi-camera-transient-sessions): Eine Kamera, die kein Bild liefert, führt zum Abbruch des Durchlaufs (Beenden von `1` unter Angabe des Namens), und es wird weder ein Stack noch ein Index geschrieben. Es öffnet `--serials` (die erste löst den Trigger aus), andernfalls die Kameras des Profils, deren `layer_order`-Eintrag aktiviert ist, wobei die Referenzkamera an erster Stelle steht – die Referenzkamera auch dann, wenn ihre Bänder deaktiviert sind, da die anderen an und löst den Trigger aus. Eine Kamera, deren Bänder deaktiviert sind, wird niemals kalibriert, sodass eine im Profil deaktivierte RGB-Kamera den Reflexionsindex (`no-absolute-radiance-scale`) nicht mehr verfehlt. Ein Profil, das keine Kameras benennt, öffnet wie bisher jede Kamera im Netzwerk.

Jedes Formular wird auf dem Backend ausgeführt, daher muss dieses laufen: Starten Sie es über die Chloros-GUI oder mit dem Befehl, den die CLI ausgibt, wenn sie es nicht erreichen kann. Im Offline-Modus liest das Backend `--input` und schreibt das Float-TIFF- und das kolorierte PNG-Datei selbst, sodass ein relativer `--input` oder `-o` wird relativ zu dem Ordner aufgelöst, aus dem Sie den Befehl ausführen (siehe [Konventionen](#conventions)). Beide Offline-Produkte enthalten die Kameradaten der Eingabe, genau wie ein `process`-Indexexport: Marke / Modell / Seriennummer, GPS, die Objektiv-Tags, den Aufnahmezeitpunkt (den der Eingabe selbst; wenn die EXIF-Daten keine Angaben enthalten, den aufgezeichneten Belichtungszeitpunkt oder den Zeitstempel im Dateinamen, in UTC) sowie die MAPIR-Kalibrierungs-ID – niemals die Raw-DN-Kalibrierung, die für einen Index nicht relevant ist – sowie `Chloros:ProcessingLevel=index` und den Indexausdruck; der Float-TIFF-Wert bezeichnet ein Band, die Voreinstellung. `--list-presets` und `--list-gradients` geben die Voreinstellungs- und Gradiententabellen des Backends aus. Eine vom Backend abgelehnte Formel oder ein `--channel`-Band, über das das TIFF-Dokument nicht verfügt, führt zum Beenden von `1` mit Angabe des Grundes durch das Backend.

> **Bei den `--channel`-Symbolen wird zwischen Groß- und Kleinschreibung unterschieden.** Die Symbolseite muss exakt mit den Kanalnamen der Voreinstellung übereinstimmen (Voreinstellungen verwenden Kleinbuchstaben, z. B. NDVI = `red`,`nir` – siehe `--list-presets`), und die Bandseite muss mit einem Bandnamen im ausgerichteten Stapel übereinstimmen (oder im Offline-Modus ein 0-basierter Bandindex sein). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` funktioniert; `--channel RED=660` schlägt mit einem `channel_map missing entries` Fehler.

#### `lattice index` – Masken für Mehrband-Stacks

Bei `--capture-level radiance` oder `reflectance` kann der 3×3-Unmix jeder Kamera ein Band auf 0 begrenzen begrenzen (siehe [Unmix-Clamp-Maske](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Ein Multiband-Stack enthält Bänder mehrerer Kameras, und jede Kamera begrenzt ihr eigenes Band; daher schreibt `--live --save-multiband` **eine Maske pro Quellkamera** neben den Stack:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

– Jede Maske ist ein Einzelband-`uint8` auf dem Pixelraster des **Stacks** (mit derselben Ausrichtungsverzerrung und demselben Zuschnitt), mit den kameraspezifischen Bits: Bit 0 = das Band des R-Kanals dieser Kamera, Bit 1 = G, Bit 2 = B. Ein aus Kanal *c* einer Kamera entnommenes Stapelband wird dort begrenzt, wo das Bit *c* dieser Kamera (R = 0, G = 1, B = 2) gesetzt ist.
– Die in jedem Band eingebettete Herkunftsangabe benennt die Maske der jeweiligen Kamera (`unmix_clamp_mask`, relativ zum Ordner des Stacks, oder `none`) sowie das zu lesende Bit (`unmix_clamp_mask_bit`) an.
- Eine Kamera, bei der der Unmix-Vorgang nichts geklemmt hat, erhält keine Datei; eine Mono- oder RGB-Kamera führt keinen Unmix-Vorgang durch und erhält ebenfalls keine. Die CLI gibt jede Maske unter dem Stapel aus (`clamp mask SN…: …`); die Antwort listet sie als `clamp_masks` auf.
- Die Index-TIFF-/PNG-Datei desselben Durchlaufs wird aus demselben Stapel auf demselben Raster berechnet, sodass die Masken auch diese beschreiben. `--capture-level raw` führt keine Entmischung durch und schreibt keine Maske.
- Der Stapel und der Index befinden sich direkt in `--output`, daher werden die Masken in `--output`s eigenem `clamp_masks/` abgelegt – niemals oberhalb des von Ihnen benannten Ordners. Dieser Ordner enthält verschiedene Produkttypen (Stack, Index, koloriertes Rendering): Wählen Sie die entsprechenden Dateien aus und übergeben Sie nicht den gesamten Ordner an die Photogrammetrie-Software.

#### Persistente Verbindungen (Smart-Prep, GUI-Äquivalent-Ablauf)

Diese Befehle halten Kameras im Backend-Pool über mehrere CLI-Aufrufe hinweg offen.

| Unterbefehl | Zweck |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Fügt dem Pool eine Kamera hinzu (Einzelkamera, kein Array). Die [allgemeinen Optionen](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` und `--packet-size` (`standard` = 1500, `jumbo` = 9000 oder Bytes; `auto`-Prüfungen) werden beim Anschließen (ein `--exposure`- oder `--gain`-Schalter, der den Automatikmodus deaktiviert). Der Befehl kehrt zurück, sobald der Stream der Kamera gestartet ist (darauf wird etwa 15 s gewartet, anschließend 10 s auf den ersten vollständigen Puffer), und gibt `stream: streaming` oder `stream: no_frames -- connected, no frames yet: …` aus, wenn der Stream nichts geliefert hat – bei einer Kamera mit Jumbo-Paketen, mit dem Hinweis, die Verbindung mit „`--packet-size standard`“ erneut herzustellen. Während eine Kamera eine Verbindung herstellt (bei der ersten Verbindung wird die Kalibrierung heruntergeladen, was etwa 70 s dauert), wartet ein anderer Befehl, der eine Kamera benötigt, höchstens 5 s darauf, dann schlägt er mit dem HTTP-Fehler 503 „`CAMERA_POOL_BUSY`“ fehl, wobei die Kamera, die gerade eine Verbindung herstellt, namentlich genannt wird; führen Sie den Befehl nach einer Weile erneut aus. „`lattice status`“ und die Kameraliste warten niemals: Eine Kamera, die gerade eine Verbindung herstellt, wird als „verbindet“ angezeigt, zusammen mit ihrem Status. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Freigabe. Die Kamera wird in den Zustand zurückversetzt, in dem sie bei der Verbindung vorgefunden wurde (Trigger-Verkabelung, PTP, Einstellungen); `--keep-settings` speichert ihre Einstellungen stattdessen in ihrem eigenen Speicher, `--clear-settings` setzt diesen Speicher auf die Werkseinstellungen zurück (siehe [Kameraspeicher](#camera-memory----keep-settings----clear-settings)). Eine Kamera, die Mitglied eines Arrays ist, gibt das gesamte Array frei. |
| `lattice cam-list` | Kameras im Pool auflisten. |
| **`lattice array-connect`**|**Ein dauerhaftes, synchronisiertes Array verbinden (DER empfohlene Einstiegspunkt).** Führt den vollständigen „Smart-Prep“-Ablauf über die GUI durch. Bei 9 Kameras dauert dies etwa eine Minute; unmittelbar nach dem Einschalten der Kameras kann es mehrere Minuten dauern. Während der Wartezeit wird jeder Schritt ausgegeben, den das Backend erreicht, als `    <s> s: <step>` (zum Beispiel `    41 s: open SN:214000535 (4 of 9)`), und das Protokoll des Backends enthält pro Schritt eine Zeitangabe (`[array-connect <id>] <step>: <s> s …`) sowie eine letzte Zeile mit den langsamsten Schritten. Es wartet bis zu 600 s. Eine bis dahin nicht hergestellte Verbindung wird vom Backend rückgängig gemacht: Jede von ihm geöffnete Kamera wird in den Zustand zurückversetzt, in dem sie vorgefunden wurde, und geschlossen; der Befehl gibt „`Error:`“ mit dem erreichten Schritt aus und beendet sich mit „`1`“, und es ist kein Array verbunden (HTTP-Fehler 504 „`CLIENT_DEADLINE_PASSED`“). Strg+C bricht die Verbindung auf dieselbe Weise ab (`Interrupted: …`); wenn das Array bereits verbunden war, wird dies in der Zeile angegeben und der Befehl `array-disconnect` genannt, der die Verbindung aufhebt. Ein Befehl, der während des Wartens abgebrochen wird (`kill -9`) führt dazu, dass die Verbindung rückgängig gemacht wird, wenn das Backend 30 Sekunden lang nichts von ihm gehört hat. Ein Array, das aktiv ist, aber noch keine synchronisierte Frame-Gruppe hat, wird weiterhin als verbunden gemeldet, wobei sein Stream als `no_frames` angegeben wird. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Ein Array freigeben. Jede Kamera bleibt im Freilaufmodus (`TriggerMode` `Off`), wobei alle anderen Einstellungen so belassen werden, wie sie bei der Verbindung vorgefunden wurden – ihre Trigger- und Synchronisationsleitungseinstellungen (die des Masters`TriggerSource` wieder auf den ursprünglichen Stand, die Synchronisationsleitung wieder auf einen Eingang), ihr PTP sowie die von der Sitzung geänderten Kameraeinstellungen – sodass ein Einzelkamera-Befehl für ein Mitglied sofort wirkt. `--keep-settings` speichert die Einstellungen jeder KameraEinstellungen so, wie sie bei Beendigung der Sitzung waren, im eigenen Speicher der Kamera; `--clear-settings` setzt diesen Speicher auf die Werkseinstellungen zurück. Siehe [Kameraspeicher](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Verbundene Arrays auflisten. |
| `lattice array-status [--array-id ID]` | Live-fps, PTP, letzter Fehler. `ptp_enabled` ist nur dann gleich `True`, wenn bei jedem Mitglied PTP aktiviert ist; `ptp_off:` nennt ein Mitglied, bei dem dies nicht der Fall ist. Die Zeile `state:`-Zeile liest `restarting (applying settings)` während eines Neustarts der Einstellungen, `recovering (…)` während eines Mitglieds, das die Stromversorgung verloren hat, wieder geöffnet wird (siehe Hinweis unten), `stalled (…)` während die Grab-Schleife läuft, aber keine Frame-Gruppe veröffentlicht, und `stopped (…)` sobald die Grab-Schleife aufgegeben hat (das Array erneut verbinden). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Eine synchronisierte Aufnahme aus dem Live-Array – Einzel / Kontinuierlich / Intervall / Schnellste. **Standardmäßig `all`**(eine Datei pro anwendbarem Exporttyp pro Kamera),**ausgerichtet** wie die Aufnahme der App (`--no-aligned` für native Dateien). Übersprungene Kameras (z. B. RGB, ausgeschlossen von Radiance/Reflexion) werden mit `Skipped: SN:<serial> (<reason>)` gemeldet; der DAQ-Wert, der die verwendete Aufnahme ausgelesen hat, wird ebenfalls gespeichert, eine reine Rohdatenaufnahme wird beigefügt und mit `DAQ: <path>` gemeldet ([Der DAQ-Wert, der eine Aufnahme ausliest, speichert](#the-daq-reading-a-capture-saves)). Siehe [Aufnahmemodi, Recorder und Offline-Nachbearbeitung](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Die kombinierte Live-Indexansicht als Video/GIF aufzeichnen (in Überwachungsqualität; wenn keine GUI-Ansicht geöffnet ist, rendert das Backend den NDVI-Wert des Arrays, oder `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Raw-Bayer-Serienaufnahme mit hoher Bildrate (Analysequalität; Offline-Nachbearbeitung). Wie `array-record` und `array-capture --continuous` meldet es sich bei jedem Abruf beim Backend: wird ohne Unterbrechung beendet (`kill -9`), das Backend stoppt es 30 s nach der letzten Rückmeldung (`stop_reason` `client_gone`; das Manifest der Bildserie wird finalisiert und die Erstellung des kombinierten Indexes gestartet, wie es bei einem Stopp der Fall ist) und das Array bleibt verbunden. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Einen gespeicherten Rohdaten-Burst zu kalibrierten Videos verarbeiten. |

> **Eine Kamera, die die Stromversorgung oder ihre Verbindung verliert, kehrt von selbst zurück.** Wenn ein Mitglied nicht mehr antwortet (ein Switch-Neustart startet jede von ihm versorgte Kamera neu), wartet das Array auf sie, öffnet die Verbindung wieder, sobald sie erneut antwortet, und übergibt ihr die von `array-connect` festgelegten Einstellungen zurück: PTP, Trigger- und Synchronisationsleitung, Netzwerk-Pacing, Pixelformat und Bildgröße sowie automatische Belichtung. Anschließend wird der Stream ohne neue `array-connect`-Einstellungen fortgesetzt. In der Zwischenzeit zeigt `array-status` `state: recovering (...)` an, und `last_error` gibt an, auf welche Kamera gewartet wird; anschließend wird `re-opened:` ausgegeben. Eine Kamera, die neu gestartet wurde, kehrt mit deaktiviertem PTP zurück; daher lautet `ptp_enabled` bis zu ihrer erneuten Öffnung `False`, und `ptp_off:` benennt sie.

> **Eine gehaltene Kamera wird niemals ein zweites Mal geöffnet.** Ein zweites Öffnen stoppt den Stream des Halters, überschreibt dessen Einstellungen und kann zum Absturz des Backends führen. Bei einer Kamera, die vom Pool gehalten wird (die GUI oder `cam-connect`), einem Array-Element oder einer Kamera, die von einem Hintergrundjob gehalten wird (ein laufendes `lattice continuous`, `multi-stream` oder `align-stream`, sei es sein eigener oder der eines anderen Clients), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, ein-Schuss-`white-balance` und `presets --apply` lehnen mit HTTP-Code 409 ab, `CAMERA_HELD` nennt den Inhaber und den Freigabebefehl (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>` oder bei einem Auftrag dessen ID und den Starter) sowie Beenden `1`. Das Gleiche gilt für eine Kamera, die gerade von einem anderen Befehl verwendet wird (ein `multi-test` oder `multi-capture`, der seine Kameras öffnet, ein weiterer One-Shot), für jeden Befehl hier: Die Fehlermeldung besagt, dass gewartet werden soll, bis dieser Befehl abgeschlossen ist. `lattice power` ohne `--sleep` liest über den eigenen Handle des Inhabers, sofern vorhanden (der Pool, ein Array, die geöffnete Kamera eines Jobs), und wird abgelehnt, wenn eine Kamera gerade von einem anderen Befehl verwendet wird. Jede `CAMERA_HELD`-Antwort enthält den Code sowohl in `code` als auch in `error_code`. Informationen zu den Befehlen, die jede Kamera öffnen, finden Sie unter [Multi-Kamera (transiente Sitzungen)](#multi-camera-transient-sessions).

##### `array-connect`-Optionen

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Alle LATTICE-Kameras automatisch erkennen (mindestens 2 erforderlich) | Die erste serielle Schnittstelle ist der MASTER. Bei Weglassen werden nur LATTICE-Modelle (M3C / M3M) erkannt und alle miteinander verbunden. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-Synchronisationsleitung. |
| `--target-fps F` | auto | Auslösefrequenz des Masters. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Tier-Auswahl überschreiben. |
| `--wire-ceiling-mbps MB_PER_S` | automatisch erkannt | **Das nachhaltige Bandbreitenbudget des Hosts in MB/s – der Wert, von dem die gesamte Array-Zuweisung abhängt.** Verringern Sie diesen Wert, wenn das Array GVSP-beschädigte Frames meldet: Der automatische Wert wird aus der vom Netzwerkadapter gemeldeten Verbindungsrate abgeleitet, die USB-Adapter, schmale PCIe-Lanes und stark ausgelastete gemeinsam genutzte Fabric-Strukturen überbewertet. Wird im Array-Erfassungsblock des Projekts gespeichert, sodass eine Wiedereröffnung bzw. eine erneute Verbindung über die CLI oder das SDK den Wert wiederherstellt. Siehe [Array-Zustand](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | keine (vollständiger Sensor) | Hardware-Binning (immer „Average“). |
| `--width N` / `--height N` / `--pixel-format FMT` | Vollsensor; kameraspezifisches Format | Eine explizite Bildgröße oder ein Pixelformat für jede Kamera, anstatt des Vollsensors und der Auswahl des kameraspezifischen Formats (Mono12 / BayerRG12). |
| `--no-recommend` | aus | Den Schritt der Netzwerkanalyse überspringen (jede Kamera behält dann ihre aktuelle Größe bei, anstatt auf den vollen Sensor eingestellt zu werden). |
| `--no-ptp` | aus | PTP deaktivieren (Zeitstempel verschiedener Kameras sind dann **nicht** vergleichbar). |
| `--packed {auto,on,off}` | `auto` | 12-Bit-Frames gepackt streamen und auf diesem Computer entpacken – mehr Frames pro Sekunde bei voller Sensorgröße, identische Daten. `auto` packt nur, wenn es sinnvoll ist und dieser Computer die Leistung dafür bereitstellen kann; `off` packt niemals. Siehe [Gepackter 12-Bit-Transport](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | GVSP-Paketgröße für jede Kamera im Array. `auto` lässt jede Kamera das Netzwerk abtasten; `jumbo` = 9000, `standard` = 1500 oder eine Größe in Byte legt den Wert fest. Vor Version 1.3.0 akzeptierte `array-connect` dieses Flag und ignorierte es. |

> **Standardmäßig voller Sensor.** Ohne `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` stellt jede Kamera auf den vollen Sensor ein – kein
> Binning, kein ROI-Ausschnitt – und lässt die Bildrate auf das Niveau sinken, das dieser Computer und
> das Netzwerk liefern, anstatt die Bilder zu verkleinern, um die Rate aufrechtzuerhalten. Die
> Zeile zur Netzwerkanalyse zeigt die prognostizierte Rate an, und die nächste Zeile listet
> schnellere Optionen (Binning oder einen kleineren ROI) mit ihren prognostizierten Raten auf; geben Sie
> `--binning` oder `--width`/`--height`, um eine davon auszuwählen. „Connect“ führt niemals von sich aus Binning oder Ausschnitte
> durch.

##### Gepackter 12-Bit-Transport (`--packed`)

Bei voller Sensorauslastung wird die Bildrate eines Arrays davon bestimmt, wie schnell jede Kamera
ihre Bilder senden kann. 12-Bit-Formate (`BayerRG12`, `Mono12`) werden normalerweise als 16-Bit-
Wörter übertragen; der **gepackte** Transport sendet dieselben 12 Bit in 1,5 Byte pro Pixel und
entpackt sie auf diesem Computer, sodass die Übertragung jedes Einzelbilds 25 % weniger Zeit in Anspruch nimmt –
etwa **33 % mehr Einzelbilder pro Sekunde bei voller Sensorauslastung** (gemessen: 10,7 → 14,0 fps
bei einem Array mit 9 Kameras). **Die gespeicherten Daten sind identisch**: gleiche Werte, gleiche
16-Bit-Dateien, gleiche Metadaten.

- `auto` (Standard) packt nur, wenn es sinnvoll ist **und** dieser Computer die
  Entpackung bewältigen kann. Bei der Verbindung misst das Backend seine eigene Entpackgeschwindigkeit, die Anzahl der CPU-Kerne,
  die aktuelle CPU-Auslastung und die verfügbare Netzwerkbandbreite, sodass ein Desktop-Computer mit freien Kernen
  bei voller Sensorauslastung komprimiert, während ein ausgelasteter oder leistungsschwacher Computer die Daten unkomprimiert streamt. Kleine ROIs und gebinnte Bilder bringen
  kaum.
- `off` komprimiert niemals – verwenden Sie diese Option, um die Komprimierung während der Diagnose auszuschalten, oder auf einem
  Computer, der seine gesamte CPU-Leistung für andere Aufgaben reservieren muss.
- `on` komprimiert immer 12-Bit-Formate.

Die „connect“-Ausgabe zeigt die Entscheidung und die Begründung an (`komprimiertes 12-Bit: bei (auto:
prognostiziert 11,0 -&gt; 14,4 fps, …)`); `/api/camera/array/status` meldet
`packed_transport` und `packed_decision`. Um eine maschinenweite Standardeinstellung festzulegen,
anstatt das Flag jedes Mal zu übergeben, setzen Sie `CHLOROS_ARRAY_PACKED12=off` (oder
`on`) in der Umgebung des Backends – ein explizites `--packed on|off` hat jedoch weiterhin Vorrang.
8-Bit- und 16-Bit-Formate werden niemals gepackt.

### Ein Projektdatensatz, alle Modi

Ein Projektordner enthält die Einstellungen aller Geräte: `cameras.json` – die Einstellungen jeder einzelnen Kamera (`cameras[].settings`, für eine einzelne Kamera und für jedes Array-Mitglied) sowie die arrayweiten Einstellungen jedes Arrays (`arrays[].capture`: Format, ROI, Binning, Triggerrate, Wire-Budget) – und `sensors.json`, die Einstellungen für jeden Lichtsensor (Integrationszeit, Bildmittelung, Obergrenze). Die App, die CLI, das SDK und der MCP-Server lesen und schreiben alle dieselben Dateien auf dieselbe Weise:

- **Ein unter einem Projekt verbundenes Gerät wird darin gespeichert und zeichnet jede Änderung darin auf.** „Unter einem Projekt“ bedeutet, dass im Backend (der App) ein Projekt geöffnet ist oder – vom SDK aus – über eine `ChlorosProject`-Datei, die einen eigenen Ordner angibt. Eine Kamera, ein Array oder ein Lichtsensor, die bzw. der noch nicht im Projekt enthalten ist, wird den Dateien des Projekts hinzugefügt, sobald eine Verbindung hergestellt wird (`lattice cam-connect`, `array-connect`, `daq pool-connect`, die App, das SDK, `chloros-mcp`) mit seinen Einstellungen. Eine Änderung aus einem beliebigen Modus (`lattice exposure`, `gain`, `configure --set`, `resolution`, die Einstellungen eines ArraysEinstellungen und Auslösefrequenz, die Obergrenze eines Lichtsensors, die Steuerelemente der App) wird kurz darauf in den Dateien dieses Projekts gespeichert. Eine an einer Kamera eingestellte Belichtung, Verstärkung oder ein AE-Ziel wird als „gespeichert“ markiert, sodass es beim erneuten Öffnen wieder übernommen wird.
- **Beim Öffnen eines Projekts werden die Geräte entsprechend dem Projekt konfiguriert**, und zwar in jedem Modus: Eine Kamera, ein Array oder ein Lichtsensor, die/der angeschlossen ist und keine eigenen Einstellungen besitzt, erhält die gespeicherten Einstellungen des Projekts (`lattice cam-connect` / `array-connect` bei geöffnetem Projekt, `chloros-cli project connect`, `open_project().connect_all()`, erneutes Öffnen der App). Ein Array, das unter einer neuen ID verbunden wird, aktualisiert das vom Projekt gespeicherte Array mit denselben Kameras; ein Array mit anderen Kameras wird daneben hinzugefügt.
- **Der speicherinterne Speichervorgang der App überschreibt niemals die Live-Einstellungen eines Geräts** mit einer älteren Kopie davon und behält die pro Element vorgenommenen Überschreibungen, die von einem CLI- oder SDK-Aufrufer gespeichert wurden, bei.
- **Das von Ihnen festgelegte Pixelformat ist dasjenige, das das Projekt beibehält.** Ein in den Array-Einstellungen, über `--pixel-format` oder über `connect_array(pixel_format=)` festgelegtes Format – einschließlich 8-Bit – wird gespeichert und bei der erneuten Verbindung exakt übernommen. Ist nichts festgelegt, verbindet sich jede Kamera mit ihrer 12-Bit-Standardeinstellung (`BayerRG12`, `Mono12` bei einem Mono-Sensor). Ein Formatname gibt die Bit-Tiefe an; die Familie entspricht der des Sensors, sodass eine Farbkamera niemals auf ein Mono-Format und eine Mono-Kamera niemals auf ein Bayer-Format eingestellt wird. Die Zeile `[PixelFormat <serial>] ... written` weist darauf hin, wenn ein Format zugeordnet wurde. Die Exporte einer 8-Bit-Aufnahme werden als 8-Bit-Daten exportiert, einschließlich der Reflexionswerte: neu skaliert und mit der Kennung `PixelScale` versehen.
- Ein Gerät, das nicht zu einem Projekt zugeordnet ist, zeichnet nichts auf.
- **Eine Aufzeichnung folgt einem Projekt, das nach ihr erstellt wurde; beim Schließen bleibt sie mit ihrem Projekt verbunden.**Wenn ein Projekt erstellt wird – ein SDK-`create_project()`, ein `chloros-mcp`-`create_project` oder das**Neue Projekt**der App – oder dupliziert wird (die Funktion**Duplizieren**der App, die die Kopie öffnet), während eine Aufzeichnung noch in das zuvor geöffnete Projekt schreibt, läuft die Aufzeichnung weiter, und das, was sie schreibt, landet im neuen Projekt an derselben Stelle darin (unten). Das Öffnen eines bestehenden Projekts — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, über die Funktion**Projekt öffnen** der App — führt dazu, dass eine laufende Aufnahme in dem Projekt fortgesetzt wird, in das sie aufzeichnet: Sie wird dort beendet, und ihre Dateien werden in die Dateiliste dieses Projekts aufgenommen (die App stoppt jede Aufnahme, bevor sie ein Projekt öffnet).
  - `array-record` und `array-burst`: Das Video bzw. der Burst-Ordner mit allen Einzelbildern wird nach Beendigung der Aufnahme vollständig dorthin verschoben (es handelt sich um `--duration`, Strg+C). Das ausgegebene `Burst dir` ist die neue Datei, und `--build` sowie `array-build-video` arbeiten davon aus. Wenn das Laufwerk eine Verschiebung verweigert, bleibt die Aufnahme dort, wo sie geschrieben wurde; dasProtokoll am Ende gibt den Grund an, ebenso wie die HTTP-Antwort beim Stopp (`relocation_error`). Die Rohbilder eines gestoppten Bursts sowie `.daq` werden in die Dateiliste des Projekts aufgenommen, in dem sie landen; ein Video (ein `array-record`, das aus einer Burst-Aufnahme erstellte Video) wird nicht zur Dateiliste eines Projekts hinzugefügt.
  - `array-capture --continuous`: Jede Aufnahme nach der Änderung wird im neuen Projekt gespeichert; die zuvor gespeicherten bleiben an ihrem Platz.
  - `daq pool-record` und `project sensor log` (sowie die app-eigene Lichtsensor-Aufnahme): Die Datei `.daq` wird nach Beendigung der Aufnahme in den neuen ProjektordnerOrdner „`light_sensor`“ des neuen Projekts verschoben und in die Dateiliste dieses Projekts aufgenommen.

  Eine Aufnahme in einen Ordner außerhalb des geöffneten Projekts (ein „`-o`“ an einem anderen Ort) bleibt dort, wo sie ist, und `lattice continuous` sowie `multi-stream` behalten den Ordner, der ihnen zugewiesen wurde.
- **In der App werden die Aufnahmen nach dem Anlegen eines neuen Projekts in dieses Projekt übernommen.** Beim Erstellen oder Duplizieren eines Projekts laufen alle laufenden Aufnahmemagazine und Aufnahmen weiter, und jede danach durchgeführte Aufnahme („Alle aufnehmen“, „Aufnahme starten“) wird im neuen Projekt gespeichert und in dessen Dateiliste aufgenommen. Das Öffnen eines bestehenden Projekts in der App stoppt zunächst alle laufenden Vorgänge.
- **Ein Projekt wird während der Aufzeichnung nicht verschoben.**Das Verschieben des geöffneten Projekts in ein anderes Arbeitsverzeichnis (in den**Einstellungen → Arbeitsverzeichnis** der App; kein CLI-Befehl verschiebt ein Projekt, ein HTTP-Client, der an `/api/move-project` sendet, tut dies) wird mit dem HTTP-Status 409 `PROJECT_RECORDING` abgelehnt, solange eine Array-Aufzeichnung oder eine Burst-Aufzeichnung, eine kontinuierliche Array-Erfassung, ein `lattice continuous`-/`multi-stream`-Auftrag oder eine Lichtsensor-Aufzeichnung in dieses Verzeichnis schreibt: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Die App lehnt dies auf dieselbe Weise ab, bevor sie nach einem Ordner fragt, während ihre eigene „Capture All“-Funktion, die Aufnahme-Start-Funktion oder eine Lichtsensor-Aufnahme läuft.

Dies ist unabhängig davon, wie die Kameras beim Trennen der Verbindung eingestellt sind (siehe [Kameraspeicher](#camera-memory----keep-settings----clear-settings)): Ein Projekt legt die Einstellungen seiner Kameras beim Öffnen selbst fest.

### Kameraspeicher — `--keep-settings` / `--clear-settings`

Wenn ein Array angeschlossen ist, übernehmen dessen Kameras die Einstellungen des Arrays, und Sie können diese anschließend ändern. Wenn Chloros die Kameras freigibt – `array-disconnect`, das Ende eines vorübergehenden Befehls (`multi-capture` ohne Array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), das Beenden von `lattice continuous`, das Beenden eines `lattice capture` einer Kamera, die nicht verbunden ist – es versetzt jede einzelne wieder in den Zustand, in dem sie vor der Verbindung vorgefunden wurde: die Verkabelung von Trigger und Synchronisationsleitung sowie die PTP- und Kameraeinstellungen (die vollständige Liste finden Sie unter [Multi-Kamera (vorübergehende Sitzungen)](#multi-camera-transient-sessions)).

Für die Verwendung der Kameras außerhalb eines Projekts schreiben zwei Flags bei jedem dieser Befehle in den **eigenen Speicher** der Kamera – also in die Einstellungen, mit denen sie in jeder Software hochfährt:

- **`--keep-settings`** speichert die Einstellungen jeder Kamera so, wie sie bei Beendigung der Sitzung in ihrem Speicher vorlagen (UserSet2, das zum Start-Set wird). Die Kameras laufen weiterhin im Freilaufmodus (`TriggerMode` `Off`), die Verkabelung der Synchronisationsleitung bleibt unverändert. Eine Kamera ohne UserSet2 speichert in UserSet1; dieser Computer merkt sich dies, sodass Chloros’ Array-Selbstheilung — was ein „UserSet1“ zurücksetzt, das von Chloros’ Basiswert abgewichen ist — lässt es unverändert.
- **`--clear-settings`** setzt den Speicher jeder Kamera auf die Werkseinstellungen zurück: Chloros’ Basiswert, den das Array-Selbstheilungsverfahren — Werkseinstellungen mit `TriggerMode`, `Off`, `TriggerSource` und `Line2`, automatische Belichtung, der kalibrierte Weißabgleich einer RGB-Kamera bleibt erhalten – in UserSet1, das zum Start-Set gemacht wurde.
- Beides lässt sich nicht kombinieren. Jede Kamera erhält eine Zeile: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` oder `SN:…: camera memory reset to the factory settings.`; die Antwort `camera_memory` enthält dasselbe pro Kamera.

Die App bietet diese Flags nicht an.

**Jeder Ausstieg versetzt die Kameras in den Ausgangszustand zurück.**Die gleiche Rückversetzung wie beim Auffinden wird bei jedem Ausstiegspfad ausgeführt, nicht nur bei den oben genannten Befehlen: `cam-disconnect` (und die Kamera**X**der App); das**X** der App auf einem Array oder in der Zeile eines beliebigen Array-Elements (ein Array kann ohne ein Element nicht weiterlaufen, daher wird das gesamte Array freigegeben); das Schließen der App (sie gibt dem Backend bis zu 20 s Zeit, jede Kamera, Array und das laufende `lattice continuous` zurückzusetzen, bevor es diese stoppt; ein zu diesem Zeitpunkt noch laufender Befehl – ein vorübergehender Befehl, der seine Kameras öffnet oder auf deren PTP-Uhren wartet, ein einmaliger `lattice capture`, `power`, `white-balance` oder `presets --apply` – wird an seinem nächsten sicheren Punkt angehalten, stellt zunächst seine Kameras wieder her (bis zu 8 Sekunden) und antwortet mit HTTP-Fehler 503 `SHUTTING_DOWN`; ein noch laufender Array-Verbindungsaufbau – `lattice array-connect`, die Verbindung der App – stoppt vor der nächsten Kamera und schließt die bereits geöffneten Kameras wieder, für bis zu 10 s, und antwortet mit HTTP-Fehler 503 `SHUTTING_DOWN`; und eine Verbindung, die während des Herunterfahrens hergestellt wird, öffnet nichts: HTTP-Fehler 503 `SHUTTING_DOWN`); und ein CLI-Befehl, der abgebrochen wird oder dessen Terminal geschlossen wird (SIGTERM / SIGHUP beenden den Vorgang – und seine Kameras wieder zurücksetzt –, bevor er beendet wird (`143` / `129`, wie bei Strg+C). Ein `lattice continuous`, `multi-stream` oder `align-stream`, der sofort beendet wird (SIGKILL) – was nichts stoppen kann –, wird vom Backend 30 s nach dem letzten Check-in zurückgesetzt. Ein Backend, das abstürzt oder zwangsweise beendet wird, kann nichts zurücksetzen; ein Neustart versetzt jede Kamera in ihren Speicherzustand zurück, und die nächste Chloros-Verbindung stellt eine Kamera, die sie als aktiviert vorfindet, neu ein.**Ein Projekt richtet seine Kameras selbst ein.** Wenn sich ein Array mit einem geöffneten Projekt verbindet, erhält jedes Mitglied die Einstellungen, die das Projekt für es gespeichert hat – einen Belichtungswert, eine Verstärkung oder einen AE-Zielwert, der für die Kamera festgelegt wurde (als Ihr eigener Wert gespeichert, sodass die automatische Belichtung diese Einstellung beibehält) sowie den Weißabgleichmodus (Korrektur durch Chloros; der kamerainterne Weißabgleich bleibt deaktiviert) – je nachdem, was der Speicher der Kamera enthält.

### Smart-AE / Smart-Capture

LATTICE-Arrays führen im Hintergrund eine kontinuierliche Belichtungsautomatik durch, sobald sie verbunden sind, aber eine neu ausgerichtete Szene benötigt einen Moment, um sich einzustellen. `array-capture --smart` ist die **praktische Komplettlösung**: Es wartet, bis sich die Belichtungsautomatik bei allen Kameras im Array eingependelt hat, und löst dann die Aufnahme aus. Verwenden Sie es, wenn Sie mitten in einer Sitzung die Szene wechseln.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Die Stabilisierungsrichtlinie ist standardmäßig konservativ eingestellt: 5 s Timeout, 1,5 s Stabilitätsfenster, Toleranz für Belichtungsstreuung von ±5 %. Passen Sie die Einstellungen über das SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) an, wenn Sie ein anderes Verhalten der Automatisierung benötigen.

#### So verhält sich die automatische Belichtung

- **Belichtung zuerst, Verstärkung zuletzt.** Bei „Auto“ für die Verstärkung bleibt diese bei 0 dB, bis die Belichtung den maximal zulässigen Wert erreicht hat.
- **Bei einer manuellen Belichtung bleibt die Verstärkung auf „Auto“**, bis Sie die Verstärkung ändern oder auf „Manuell“ stellen. Zurück auf „Auto“ gilt für die Verstärkung wieder die vorherige Begrenzung.
- **Kleine Lichter verdunkeln die Szene nicht.** Eine Spiegelung von bis zu 1 % des Bildausschnitts wird ignoriert, bei den Feldern eines bekannten Kalibrierungsziels bis zu 0,5 % eines Feldes.
- **Die DAQ-Belichtungsobergrenze gilt nur, solange ein Kalibrierungsziel im Blickfeld ist.** Sie verhindert das Ausklippen eines weißen Feldes; ist kein Feld zu sehen, belichtet die Kamera entsprechend der Szene.
- **Konstantes Licht bleibt stabil, und sich änderndes Licht pendelt sich innerhalb von Sekunden ein.** Bei konstantem Licht bleiben die Belichtung und ihre Grenze unverändert, und nach einer starken Lichtänderung pendelt sich das Bild innerhalb weniger Sekunden ein. Ein Übersteuerungsfehler begrenzt die Belichtung nicht mehr dauerhaft, und bei einer Änderung des ROI wird die Belichtung wiederhergestellt.
- **Das Helligkeitsziel bleibt in Reichweite.** Es steigt nicht an, solange sich die Kamera an ihrer Belichtungsgrenze befindet und ihre Verstärkung nicht erhöht werden kann, sodass ein wiederkehrendes Licht den Bildausschnitt nicht überbelichtet.
- **Eine Kamera, deren automatische Belichtung nicht mehr reagiert, wird auf dem Niveau gehalten, das der aktuellen Lichtstärke entspricht.** Bei strahlendem Sonnenschein könnte eine Kamera auf ihre kürzeste Belichtungszeit fallen und dort mit einem dunklen Bild verharren, wobei alle Einstellungen normal angezeigt werden. Chloros überwacht das Bild, hält die Kamera auf dem für diese Lichtstärke gewählten Niveau und lässt sie los, sobald sich die Lichtverhältnisse ändern. Ein verdunkelter oder abgedeckter Lichtsensor stört diesen Vorgang nicht.

### Aufnahme-Exportstufen (die Standardeinstellung von `all`)

Ab dieser Version, `lattice capture`, `lattice multi-capture` und `lattice array-capture` **standardmäßig auf `--processing all`** – eine gespeicherte Datei pro Exporttyp, die für jede Kamera gilt und dem Verhalten der GUI-Funktion „Alle aufnehmen“ entspricht. Die Stufen sind:

| Stufe | Ausgabe | Gilt für |
| --- | --- | --- |
| `raw` | Einkanal-Bayer (Mono-Kameras: das einzelne Band) direkt vom Sensor. | Alle Kameras. |
| `debayered` | 3-Kanal-Farbe, gespeichert in **RGB**-Kanalreihenfolge (ein TIFF mit RGB-photometrischer Interpretation; PNG/JPEG werden ebenfalls als RGB geöffnet): Der Dunkeloffset (DSNU) des Kalibrierungspakets wird vom Rohdatenwert abgezogen, anschließend erfolgt ein OpenCV-kantensensitives— ohne Linearitäts-LUT, Flat-Field-Korrektur oder Unmix (Mono-Kameras: 1-Kanal-Graustufen). Die im Speicher entbayerteten Bilder, die das Python-SDK zurückgibt (`grab()`, `frame_stream()`), liegen im**BGR**-Format vor, der Reihenfolge von OpenCV; nur die gespeicherte Datei ist im RGB-Format. | Alle Kameras. |
| `radiance` | float32 W/m²/sr/nm über die vollständige radiometrische Kette. | Nur multispektral (M3C/M3M) – **wird bei RGB-Filter-Kameras**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-kompatibel. | Nur multispektral und **nur, wenn ein DAQ gebunden ist und die Kamera kalibriert ist**; andernfalls übersprungen. |
| `display` | Vollständige GUI-Vorschau-Kette (CCM + WB + Gamma gemäß dem Profil der Kamera). Gespeichert in einem Ordner „`preview/`“. `lattice capture` akzeptiert hierfür auch den Namen `preview`; `array-capture` / `multi-capture` akzeptieren nur `display`. | Alle Kameras. |

Geben Sie eine einzelne Stufe an, um nur diese zu speichern (`--processing debayered`). Wenn Sie nach `all` fragen, werden Stufen, die für eine bestimmte Nockenwelle nicht zutreffen, übersprungen (und gemeldet), es wird jedoch kein Fehler ausgegeben – eine nicht angeschlossene oder nicht kalibrierte Kamera erhält weiterhin `raw` / `debayered` / `preview`.

#### Der von einem DAQ bei einer Aufzeichnung gelesene Wert wird gespeichert

Immer wenn ein DAQ an die Kamera oder das Array gebunden oder in einen Pool aufgenommen wird, wird der bei einer Aufzeichnung verwendete Downwelling-Messwert in eine **`.daq`**-Datei im Unterordner `daq/` des Ordners „captures“ gespeichert, zusammen mit dem `.match.json` (wie weit der Messwert von der Belichtung entfernt war) und dem `.cameras.json` (zu wem der Messwert gehört) versehen und in einer `DAQ:`-Zeile gemeldet. Dies gilt für jede Ebene, einschließlich einer Roh- oder einer debayierten Aufnahme, sodass jede Aufnahme später zu Radianz, Reflektanz oder einem Index nachbearbeitet werden kann. Es handelt sich um dieselbe Datei mit demselben Namen und denselben Sidecars, unabhängig davon, ob die Aufnahme über die GUI, die CLI oder das SDK erfolgte. Fehlt der DAQ-Messwert gänzlich, gibt es kein `.daq`, und ein `multi-capture` ohne zugehöriges Array (der vorübergehende Fallback) speichert nichts.

| Flag | Wirkung |
| --- | --- |
| `--no-daq` | Speichere den Messwert nicht bei einer Erfassung, deren Ebenen ihn nicht verwenden (`raw`, `debayered`). Eine Stufe `radiance`, `reflectance` oder `display` schreibt ihn dennoch, da der erneute Import den Messwert benötigt, mit dem das Produkt erstellt wurde. |
| `--daq-tolerance-ms MS` | Wie weit darf der Messwert von der Belichtung abweichen und dennoch als innerhalb der Toleranz liegend gelten (Standardwert 500)? Es ändert sich lediglich das Ergebnis (bei `.match.json`, `in_tolerance` und `tolerance_ms` sowie bei der Reflexion `DlsInTolerance`), niemals der verwendete Messwert. Siehe [Ein DAQ-Messwert weit entfernt von der Belichtung](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Die ältere Methode, um den Wert `.daq` bei einer reinen Rohdatenaufnahme abzufragen. Dies ist nun die Standardeinstellung, und `lattice capture` sowie `array-capture` akzeptieren das Flag weiterhin (`multi-capture` hingegen nicht). `--no-daq` hat Vorrang vor diesen. |

Welches DAQ: das an die Kamera oder das Array gebundene (die Auswahl des Lichtsensors in der GUI, eine gespeicherte Zuordnung eines Projekts oder `chloros-cli daq pool-bind`), andernfalls das aktuellste, über das das Backend verfügt. Eine Rohdaten-Serienaufnahme (`array-burst`) behält wie bisher für jeden einzelnen Messwert im Burst-Ordner seine eigene `.daq`-Datei.

**PNG- und JPEG-Dateien enthalten 8 Bit pro Kanal, daher verwenden sie nur die `display`-/`preview`- und Index-Produkte.** `--format png` oder `jpg` mit einem Wert, den sie nicht speichern können, werden vor der Erfassung abgelehnt, wobei der Wert und das TIFF-Format, das ihn enthält, angegeben werden:

- `radiance` (float32) und `reflectance` (uint16, `32768` = 1,0) — immer, daher wird auch `--processing all` (der Standardwert) zusammen mit `--format png`/`jpg` abgelehnt: Verwenden Sie `--format tiff` oder `--levels display`.
- `raw` und `debayered` von einer Kamera, die mehr als 8 Bit (die 12-Bit-M3C-/Mono12-M3M-Frames) – die CLI kann die Bittiefe der Kamera nicht erkennen, daher lehnt das Backend diese ab (HTTP-Fehler 400). Der einzelneEinzelbild-`lattice capture --processing debayered` erfasst ein 8-Bit-Bild für PNG/JPEG und wird nicht abgelehnt, ebenso wenig wie ein `multi-capture` ohne persistentes Array (8-Bit-Debayering).

Jede Aufnahmeroute antwortet auf dieselbe Weise (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst` sowie die dahinterliegenden HTTP-Routen.

### So sieht ein Aufzeichnungsordner aus

Jeder Exporttyp wird in einem **eigenen Unterordner**unter „`-o`“ abgelegt, sodass bei einer mehrstufigen Aufzeichnung niemals verschiedene Typen vermischt werden. Die Skizze zeigt die**Standard-Aufzeichnung `all` einer Kamera durch `lattice capture`** – mehrere Ebenen pro Kamera, sodass jede Datei die Endung `_<level>` trägt:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

So variieren die Dateinamen:

- **Kamerafeld – Seriennummer, dann Filter:** Jede Aufnahmedatei benennt die Kamera, die sie aufgenommen hat, als `SN<serial>-<filter>`, wobei `<filter>` der Filterteil des Kameramodells ist: `SN261902142-F450` für eine `M3M-L87-F450`, `SN213800234-FRGN` für eine `M3C-L87-FRGN`, `SN214701292-FRGB` für eine RGB-Kamera. Der Filterteil wird an die Seriennummer angehängt, z. B. `-`, nicht mit `_`, sodass ein Name dieselben durch `_` getrennten Felder wie zuvor aufweist (Präfix, Datum, Uhrzeit, Millisekunden, Kamera, Pegel). Eine Kamera, deren Modell nicht gelesen werden kann (eine gelöschte DeviceUserID), erhält den Namen `SN<serial>` allein. Aufnahmen aus früheren Versionen haben keinen Filter im Namen; Chloros liest beide.
- **Präfix:** `lattice capture` benennt Dateien als `capture_…`; `array-capture`, `multi-capture` bei einem verbundenen Array sowie `project capture` / `burst` / `stream` eines Arrays benennt sie als `sync_<ts>_SN<serial>-<filter>…`.
- **Ebenen-Suffix nur bei mehreren Ebenen.**Eine Kamera, die**mehrere**Ebenen in einer Aufnahme speichert (`all` oder `--levels a,b`), fügt `_<level>` hinzu. Eine Kamera, die**eine** Ebene speichert, behält den einfachen Namen ohne Suffix bei – `--processing reflectance` schreibt `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` –, sodass Skripte, die für Aufnahmen mit einer einzigen Ebene geschrieben wurden, weiterhin funktionieren. Die Ebene wird weiterhin durch den Ordner angegeben.
- **Clamp-Masken folgen dem Produktnamen:** `<product name>_clampmask.tif`. Eine Reflexionsaufnahme auf einer Ebene schreibt `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; eine `all`-Aufnahme schreibt `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Die Erweiterung** folgt auf `--format`: Der Standardwert `tiff` ergibt `.tiff`, `--format tif` ergibt `.tif` (Clamp-Masken lauten immer `.tif`).
- **Eine Ausnahme:** `lattice capture --processing debayered` (oder `--levels debayered`) nimmt einen schnellen einzelnen-Frame-Pfad und schreibt `capture_<ts>_SN<serial>-<filter>.tiff` direkt in `-o` – ohne Unterordner.
- **Streams und Bursts** enthalten dasselbe Feld: `lattice continuous` schreibt `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` und eine Rohdaten-Burst-Datei `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` ist der Aufzeichnungszeitstempel und `<serial>` die Seriennummer der Kamera, sodass eine synchronisierte Gruppe einen
gemeinsamen Zeitstempel über alle Kameras hinweg teilt. **Beachten Sie die eine Asymmetrie:** Die Ebene `display` wird in einem Ordner
mit dem Namen `preview/` gespeichert, während die mehrstufigen Dateien `_display` im Namen – Ordner und Suffix unterscheiden sich
nur für diese Ebene. Unbekannte Ebenen werden in einen Ordner mit ihrem eigenen Namen verschoben, und wenn der Unterordner
nicht erstellt werden kann, wird die Datei in das Ausgabeverzeichnis geschrieben, anstatt verloren zu gehen.

**Neuverarbeitung eines Aufnahmordners:**Richten Sie `chloros-cli process` auf das**Stammverzeichnis „captures“**
(`output/`). `process` importiert normalerweise nur den von Ihnen angegebenen Ordner; enthält dieser jedoch keine
Bilder enthält, aber Unterordner hat, durchsucht es automatisch die Unterordner – sodass die Unterordner der jeweiligen Ebene und der
Unterordner „`daq/`“ alle auf einmal erfasst werden. Jede Ebene einer Aufnahme wird als einzelnes Bild importiert, wobei
die anderen Ebenen als Modi verfügbar sind, anstatt als ein Bild pro Ebene.

Die direkte Benennung eines **Ebenen-Unterordners** (z. B. `output/raw/`) funktioniert ebenfalls. Dabei bleibt `daq/`
zurück; kopieren Sie daher den DAQ-Messwert mit oder verweisen Sie darauf, wenn Sieein radiometrisches
Produkt aus `raw/` ableiten – andernfalls kann der Zeitstempelabgleich nicht zugeordnet werden.

**Die Verarbeitung beginnt immer bei `raw`.** Innerhalb jeder Aufnahme ist das Rohbild die Pipeline-Quelle;
`debayered`, `radiance`, `reflectance` und `preview` dienen als Anzeigemodi, werden jedoch niemals
zurück in die Pipeline geleitet. Eine erneute Verarbeitung eines abgeleiteten Produkts würde die Vignettierung, CCM und
die Strahlungsberechnungen, die bereits in seine Pixel eingebrannt sind, erneut anwenden, weshalb Chloros dies ablehnt, anstatt
eine doppelte Verarbeitung durchzuführen. Zwei wichtige Konsequenzen:

- Die Renderings „`index/`“ und „`composite/`“ werden **niemals** verarbeitet. Es handelt sich um Ausgabedateien, nicht um Aufnahmen –
  ein NDVI-LUT-Rendering lässt keine aussagekräftige Strahlungsinterpretation zu.
- Ein **ohne** `raw` exportierter Aufnahmen-Ordner (z. B. `array-capture --processing reflectance`) enthält
  keine gültige Pipeline-Quelle. Diese Aufzeichnungen lassen sich normal importieren und anzeigen, aber `process` überspringt
  sie und gibt folgende Meldung aus:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Wenn Sie ein abgeleitetes Produkt wirklich durchlaufen lassen müssen – beispielsweise eine Hub-Sitzung, die mit
  `demosaic` aktiv ist, oder einen Legacy-Ordner –, erzwingt `--input-level {raw,debayered,processed}` den Einstiegspunkt
  und setzt die Überspringung außer Kraft. Dieses Flag ist die absichtlich vorgesehene Ausweichmöglichkeit; `auto` (die Standardeinstellung)
  verarbeitet niemals eine Aufzeichnung, die keine Rohdaten enthält.

**Kalibrierungstags: nur bei `raw`.** Die `raw`-Datei einer Erfassung enthält das Kalibrierungspaket der Kamera: die `CalibrationJSON`-Nutzdaten und die MicaSense-kompatiblen Tags (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), die die Rohwerte des Sensors beschreiben. Alle anderen Dateien auf dieser Ebene (`debayered`, `display`, `radiance`, `reflectance`, der `index/`-Render) sowie jeder `lattice continuous`- / `multi-stream`-Frame die Identität der Kamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, …) sowie den bildspezifischen Datensatz `XMP-mapir:FrameJSON` (Belichtung, Verstärkung, Temperatur, Pixelformat, Aufnahmezeit, DAQ-Übereinstimmung), da jedes `process`-Produkt diese Kennung trägt. `process` ermittelt die Kalibrierung einer solchen Datei anhand ihrer Seriennummer und `CalBundleSha256` im Kalibrierungs-Cache von Chloros, wobei das Paket einmal heruntergeladen wird, falls es auf diesem auf diesem Computer fehlt.

#### Dateien aus mehreren Kameras

Eine Datei, die aus den Bändern mehrerer Kameras erstellt wurde – ein Multiband-Stack (`align-apply`, `align-stream`, `index --live --save-multiband`), ein aus mehreren Kameras erstellter Live-Index (`index.tif`, `index_LUT.png`), das Bild einer Array-Aufnahme (`composite/`) – nennt jede Kamera, die darin enthalten ist:

- **Listen pro Kamera, in Bandreihenfolge** (die Zusammensetzung: Hintergrundebene, dann Vordergrund; eine Indexebene mit den Kameras, aus deren Bändern sie stammt): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (das Kalibrierungspaket, durch das die Bänder der jeweiligen Kamera geleitet wurden, andernfalls – ein Stapel aus `align-apply` und `align-stream`, dessen Bänder keinem unterzogen wurden – das Paket Chloros, das die Kamera zwischengespeichert hat und nach dem ihre Aufnahmedateien benannt sind), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` – parallele Listen, wobei Element *n* jeder Liste dieselbe Kamera beschreibt; `-`, wenn eine Kamera keinen Wert hat (kein Kalibrierungspaket, ein unbekanntes Objektiv). Jede Kamera ist **ein** Eintrag: Wenn ihr Paket mehrere Werte für ein Feld enthält – beispielsweise die Halogenlampenzertifikate eines Pakets –, werden diese mit `;` (`F-1479-1;F-1480-1;F-1481-1`) verbunden, niemals mit einem Komma, sodass eine Liste unabhängig von der Darstellungsweise einen Eintrag pro Kamera enthält.
- **EXIF `SerialNumber`** enthält dieselben Seriennummern, durch Kommas getrennt (`214000535,214000551`).
- Es enthält **keine** Einzelkamera-Identität (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): diese bezeichnen eine Kamera und deren Paket, und `process` würde daraus die Kalibrierung der Datei ermitteln. Ein aus den Bändern mehrerer Kameras berechneter Index (`index.tif`, `index_LUT.png`) enthält EXIF-Daten für das Rig (`Model`, `LATTICE`) (das Rig) und nicht das Modell einer einzelnen Kamera (`LATT-…`).

Eine Datei mit den Bändern **einer** Kamera – ein bandweiser TIFF-Datei (`--per-band`), ein `align-apply --per-camera`-Frame, ein Stack oder Live-Index einer einzelnen Kamera – enthält die eigenen Identitäts-Tags dieser Kamera, wie jede andere Aufnahmedatei auch. Der Stapel von `align-apply`, die band- und kameraspezifischen Dateien sowie jeder Stapel von `align-stream` und seine band-spezifischen Dateien enthalten die Aufnahmezeit ihrer Gruppe (die Belichtungszeit der Referenzkamera, UTC), und ein `--per-camera`-Frame einer Mono-Kamera besteht aus einer Ebene, genau wie der Stapel bei `--bit-depth`.**Kameraausrichtung (Spiegeln / Drehen).** Die Ausrichtungseinstellung einer Kamera (die Optionen „Mirror H“ /
„Mirror V“ / „Rotation“) in der Benutzeroberfläche bestimmt, was bei einer Aufnahme gespeichert wird:

- **`raw` und `debayered` werden genau so gespeichert, wie der Sensor sie ausgelesen hat**, unabhängig von der Einstellung. Ein
  gedrehtes oder gespiegeltes Bayer-Mosaik würde ein anderes Farbfiltermuster ergeben, und die Kalibrierungskarten
  (Dark-Offset, Flat-Field) beziehen sich auf das Raster des Sensors.
- **`preview`/`display`, `radiance`, `reflectance` und die `index/`-Renderdatei sind ausgerichtet** –
  einmal gedreht, nachdem alle Kalibrierungsberechnungen abgeschlossen sind, sodass sie sich so öffnen, wie die Kamera ausgerichtet war. Ihre
  `clamp_masks/…_clampmask.tif`-Dateien liegen auf demselben Raster.
- **Jede Aufnahme-Datei speichert die Einstellung** im XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, im Uhrzeigersinn, nach
  den Spiegelungen) sowie `ChlorosOrientationApplied` – `false` für Rohdaten und debayerte (native Pixel),
  `true` bei den ausgerichteten Produkten. `ChlorosProcessing` listet die auf die Pixel dieser Datei angewendeten
  Operationen auf (`none` bei nativen Pixeln). Dazu gehören jedes Bild, das `lattice continuous` überträgt (die
  kameraeigene Einstellung, nicht angewendet), sowie jedes Bild von `lattice multi-stream` (die Identität: Seine
  Zugangsbilder sind nativ, wie bei einer Array-Aufnahme). `lattice align-stream` / `align-apply`
  Stacks zeichnen die Einstellung der Master-Kamera auf, ohne dass diese angewendet wird: Ihre Bänder werden in das Bild der Master-Kamera
  verzerrt, aber nicht gedreht.
- **`process` wendet die aufgezeichnete Ausrichtung an**, wenn es ein Rohdatenbild verarbeitet: zuerst Dunkeloffset, Flatfield-Korrektur,
  Demosaicing und Unmix in Sensorkoordinaten, anschließend die Verzerrungskorrektur für die Array-Ausrichtung und
  Ausrichtung zuletzt – sodass die Vorschau-, Radiance-, Reflektanz- und Index-Produkte genau wie
  die der Live-Aufnahme ausfallen und `ChlorosOrientationApplied=true` aufzeichnen. Die unkalibrierten Fallback-
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) werden nach dem
  dem Flat-Field. Das debayerte Produkt bleibt im Originalformat.
- **Der Chloros-Bildbetrachter zeigt eine Roh- oder debayerte Datei, die entsprechend ihrer Aufzeichnungsausrichtung gedreht wurde** (nur zur Anzeige –
  die Datei wird nicht verändert), sodass sie wie die entsprechenden Produkte aussieht; die Miniaturansicht, die Ansicht in voller Größe, die
  Cursor-Anzeige und die Index-/LUT-Sandbox verwenden alle dieselbe Drehung. Eine Datei ohne Datensatz und
  ein Produkt (bereits ausgerichtet) werden so angezeigt, wie sie gespeichert wurden. Ein Sandbox-Export speichert die Ausrichtung, die seine
  Pixel enthalten (`ChlorosOrientationApplied=true`; die Identität für eine Ebene ohne Aufzeichnung).
- **Aufnahmen ohne Datensatz werden als nativ verarbeitet** (unverändertes Verhalten). Vor Version 1.3.0 hat eine
  Einzelkamera-Aufnahme mit eingestellter Spiegelung oder Drehung ihre Roh- und Debayering-Dateien selbst gedreht;
  Chloros kann eine solche bereits-gedrehten Rohdatei von einer nativen unterscheiden und macht keine Vermutungen, daher
  verarbeitet es sie so, wie sie gespeichert wurde. Die Kalibrierungskarten basieren auf dem sensoreigenen Raster; verwenden Sie daher für eine solche
  Aufnahme eine mit 1.3.0, wenn deren Kalibrierung von Bedeutung ist.
- **Arrays drehen standardmäßig keine Einzeldatei** (die Ausrichtungsverschiebung und die `Alignment*`-XMP-Datei liegen in
  den kameraspezifischen, nicht gedrehten Bildern) und zeichnen die Identität auf. `array-capture --orient` dreht die
  Produkte jedes Mitglieds nach der Ausrichtungstransformation: entweder um die Orientierung des **Masters**, als sie
  in dessen Bildrahmen transformiert wurden (damit die Mitglieder ko-registriert bleiben), oder um die ihrer eigenen Kamera. Jede
  Datei speichert die Ausrichtung des Bildausschnitts, in dem sich ihre eigenen Pixel befinden – ein Rohdatensatz, der die Ausrichtung der eigenen Kamera enthält –
  und ein Rohdatensatz, der die Ausrichtungstransformation enthält, speichert zudem `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: die des Masters, für ein Produkt, das
  in den Rahmen des Masters verzerrt wurde. `process` dreht jedes Produkt um den Wert, der zu seinen Pixeln gehört:
  um den ausgerichteten Wert, wenn es das Raw-Bild verzerrt (`--array-alignment`, der Standardwert – die Produkte der live ausgerichteten
  Aufnahme“), das kameraspezifische, wenn dies nicht der Fall ist (`--no-array-alignment` – die der
  nicht ausgerichteten Live-Aufnahme). Die gleiche Option gibt es für `array-burst`, `project capture` / `burst` /
  `stream`, Rezepte und das SDK.

### Übersprungene Aufnahmen in gemischten Filter-Arrays

Wenn Sie RGB- und Multispektralkameras in einem Array mischen, speichert `array-capture --processing radiance` (oder `reflectance`) die Multispektral-Bilder und **überspringt** die RGB-Kameras – die Strahlungsintensität pro Bayer-Pixel ist für einen Breitbandsensor nicht aussagekräftig. Die CLI gibt jede gespeicherte Datei (mit ihrer Exportstufe), jeden geschriebenen `.daq` und jedes Überspringen explizit aus, sodass die Dateianzahl nicht überraschend ist:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Token für Überspringungsgründe folgen dem Muster `<level>-not-applicable-to-rgb-cam`. Eine Kamera, deren Kalibrierungspaket nicht geladen ist, überspringt `radiance` / `reflectance` mit `<level>-needs-calibration (connect the camera so its cal pack loads)` überspringt; und bei fehlendem brauchbarem DAQ-Reflexionswert wird mit `reflectance-skipped-no-dls` übersprungen. Die Reflexion kann auch mit `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` übersprungen werden, sowie bei `dls-uncalibrated-band-<nm>`, wenn das Band größtenteils außerhalb des radiometrisch kalibrierten Bereichs des DAQ-Lichtsensors (~374–974 nm) liegt – unter den lieferbaren Artikelnummern gilt dies nur für F988, dessen unterstützter Arbeitsablauf auf dem Reflektanz-Panel basiert.

Verwenden Sie `--processing debayered` (oder `display`), um jede Kamera unabhängig vom Filtertyp einzubeziehen, oder den Standardwert `all`, um alle anwendbaren Werte pro Kamera in einem Durchgang zu erhalten.

### Ein DAQ-Messwert weit entfernt von der Belichtung

Jeder Wert „`.daq`“, den eine Aufnahme speichert, ist der zu dieser Belichtung gehörende DAQ-Messwert, und der daneben stehende Wert „`.match.json`“ gibt an, wie weit die beiden voneinander entfernt waren (`gap_ms`) und ob dies innerhalb der Toleranz liegt (`in_tolerance`, 500 ms, sofern nicht in der Aufnahmeeinstellung anders festgelegt `--daq-tolerance-ms`). Ein Wert außerhalb dieser Toleranz wird dennoch verwendet, sodass kein Bild verloren geht, und er wird gekennzeichnet: Die XMP-Datei der Reflektanzdatei enthält `DlsInTolerance=False` und `DlsMatchGapS`. `lattice capture`, `multi-capture` und `array-capture` geben dies ebenfalls unter dem betreffenden `.daq` aus, wobei der DAQ&#x27;Stream-Fehler, wenn man es erklärt:

**Wie ein Messwert einer Belichtung zugeordnet wird.** Eine Erfassung wartet auf den ersten Messwert des DAQ nach der Belichtung, damit sie die Messwerte auf beiden Seiten davon zusammenführen kann (`method` `interpolated`). Ein DAQ-E sendet bei Tageslicht alle 0,4 s und in Innenräumen alle 1–2 s einen Messwert, sodass sich die Wartezeit auf etwa eineinhalb dieser Intervalle zum Befehl summiert, nicht zur Belichtungszeit. Auf ein DAQ, das das Senden eingestellt hat, wird nicht gewartet. Ein-E-Messwert ist der über das Intervall seit der letzten Messung gemittelte Lichtwert; er wird daher in die Mitte dieses Intervalls gesetzt: `gap_ms` wird von der Belichtung bis zu dieser Mitte gemessen, und `bracket_stamps_ns` gibt die Zeitstempel der Messwerte selbst an. `array-capture --continuous` wartet, ohne die Aufnahme zu verlangsamen: Seine Rohdateien werden sofort geschrieben, und ihre `.daq`-Werte werden hinzugefügt, sobald der Messwert eintrifft. Bei `--interval` wartet jeder Durchlauf wie bei einer einzelnen Erfassung.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Wenn aus diesem Messwert keine Reflexionsdatei geschrieben wurde, endet die erste Zeile stattdessen mit `saved anyway; its .match.json says in_tolerance: false`.

Die Erfassung endet dennoch mit `0`: Jede Datei wurde geschrieben und gibt an, was sie enthält. Ein DAQ, dessen Datenstrom ins Stocken geraten ist, liefert weiterhin seinen letzten Messwert; überprüfen Sie diesen daher mit `chloros-cli daq pool-list`, bevor Sie sich auf die Reflexionswerte dieser Erfassung verlassen. Das Backend stellt die Verbindung zu einem gepoolten DAQ-E wieder her, das die Datenübertragung eingestellt hat (siehe [Ein DAQ-E, das keine Messwerte mehr liefert](#a-daq-e-that-stops-delivering)). Ein Messwert außerhalb der Toleranz gibt nur seine `DAQ:`-Zeile aus, es sei denn, es handelt sich um einen beibehaltenen Messwert (siehe unten).

**Ein Messwert, der nach dem Ausfall seines DAQ beibehalten wurde.** Wenn kein DAQ einen Messwert für eine Belichtung liefern kann (das gebundene DAQ ist nicht verfügbar oder kein gepooltes DAQ verfügt über einen, z. B. nach `daq pool-disconnect`), verwendet die Erfassung den letzten Messwert, den Chloros hatte, und seine Zeile „`.match.json`“ beschreibt diesen Messwert in Bezug auf diese Belichtung: „`method`“, „`retained`“, „`gap_ms` von dieser Belichtung zu den Proben, aus denen der Messwert stammt, `in_tolerance` aus dieser Lücke und `retained_from_epoch_ns`, die Belichtung, der der Messwert ursprünglich zugeordnet wurde. Eine Aufnahme, die lange nach dem Ausfall des DAQ gemacht wurde, druckt daher die oben genannten Zeilen aus und fügt diesen noch eine weitere, die bereits gedruckt wird, obwohl der Messwert noch innerhalb der Toleranz liegt:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Sein Wert „`.daq`“ gibt weiterhin das DAQ an, das den Messwert erfasst hat, sowie die Kalibrierung und die Kappe, unter denen dieser Messwert erfasst wurde, sodass bei einem erneuten Import bekannt ist, welches DAQ, welche Kalibrierung und welche Kappe ihn erzeugt haben.

### Dateien, die nicht geschrieben werden können

Eine Kamera, deren Datei nicht geschrieben werden kann – ein volles Laufwerk, ein Berechtigungsfehler –, verhindert nicht, dass die Dateien der anderen Kameras gespeichert werden. Jeder Aufnahmebefehl (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; sowie die Befehle `capture`, `burst` und `stream` eines Rezepts) gibt jede fehlgeschlagene Datei an „stderr“ aus, und **beendet dann `1`**, obwohl der Rest gespeichert wurde:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Jede Datei, die zu einer Aufnahme gehört, ist abgedeckt, einschließlich des einzelnenEinzelbild-Debayering-Datei `lattice capture --levels debayered` eingeschlossen. Ein `.daq`-Lesevorgang, dessen `.match.json`-Herkunfts-Sidecar-Datei nicht geschrieben werden konnte, wird als `[match]` aufgeführt (die `.daq`-Datei selbst wurde gespeichert).

Bei einem vollen Laufwerk werden wiederholte Aufnahmen (`array-capture --continuous` / `--interval`, `project burst` / `stream`, ein Rezept `burst` / `stream`) **endet bei diesem Durchlauf**, anstatt für den Rest des Laufs nichts mehr auf dasselbe Laufwerk zu schreiben. Jeder andere Fehler wird bei jedem Durchlauf gemeldet, bei dem er auftritt, und der Lauf wird bis zum Ende fortgesetzt, bevor er mit `1` beendet wird. `array-capture --continuous` (die kontinuierliche Erfassung des Backends) listet keine Dateien auf, auch keine fehlgeschlagenen: Es zählt sie in seinen Fortschritts- und Zusammenfassungszeilen (`N failed`, `N file(s) failed to write.`) und beendet `1` – siehe [Kontinuierlicher Modus](#continuous-mode---continuous).

Die Recorder – `array-burst` und `array-record` – melden ihre eigenen Fehler. `array-burst` stoppt sich selbst, wenn das Laufwerk voll ist oder kurz davor steht, voll zu sein. Beide laufen trotz aller anderen fehlgeschlagenen Schreibvorgänge weiter. Wenn einer von beiden einen Fehler meldet, wird dieser nach den Zählern ausgegeben, und der Befehl **beendet `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` anstelle von `Disk full:` bedeutet, dass der Burst vor dem Füllen des Laufwerk nicht vollständig gefüllt wurde. Die vor dem Abbruch aufgezeichneten Frames bleiben erhalten. `array-burst --build` bricht den Build ab, wenn der Speicherplatz knapp wird, da die Videos auf dasselbe Laufwerk geschrieben würden. Schaffen Sie zunächst freien Speicherplatz und führen Sie dann `array-build-video --burst-dir …` aus. Nach jedem anderen Fehler läuft der Build weiterhin.

Ein `array-record`, bei dem überhaupt kein Frame geschrieben wurde, ist ebenfalls eine fehlgeschlagene Aufzeichnung (`Recorder error: no-frames: the combined view published no frame to record …`, `1` beenden) – siehe [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Aufnahmemodi, Recorder und Offline-Nachbearbeitung

Diese arbeiten alle mit einem **persistenten Array** (führen Sie zuerst `array-connect` aus). Sie spiegeln das GUI-Aufnahmefenster wider.

### `array-capture`-Modi

`array-capture` ist ein einzelner Befehl mit vier Auslösemodi sowie einer Reihe von Exportoptionen:

| Modus | Flag | Verhalten |
| --- | --- | --- |
| **Einzelaufnahme** *(Standard)* | (keine) | Eine synchronisierte Aufnahmegruppe, dann Beenden. |
| **Kontinuierlich** | `--continuous` | Aufeinanderfolgende Erfassungen, bis `Ctrl+C`, `--count N` erfasst, `--duration S` oder ein volles Laufwerk. Führt die eigene kontinuierliche Erfassung des Backends aus (der „Kontinuierliche Modus“ der GUI) – siehe [Kontinuierlicher Modus](#continuous-mode---continuous) weiter unten. |
| **Intervall** | `--interval S` | Ein Durchlauf alle `S` Sekunden (gemessen ab dem Beginn jedes Durchlaufs), gleiche Grenzen. |
| **Schnellste** | `--fastest` | Nur Rohdaten-Daten + der zugewiesene DAQ-Messwert + der zusammengesetzte Index; überspringt die Berechnungen für Strahlungsintensität/Reflexionsgrad/Anzeige, sodass das Bild schnell angezeigt wird. Impliziert `--processing raw --force-daq`. Die gespeicherten `.daq`-Daten werden später zu kalibrierten Produkten nachbearbeitet. |

Export-Schaltflächen (kombinierbar mit jedem Modus; alle nutzen denselben GUI-/SDK-Endpunkt):

| Flag | Wirkung |
| --- | --- |
| `--array-id ID` | Das Array, aus dem erfasst werden soll (weglassen, wenn nur eines angeschlossen ist). |
| `-o, --output DIR` | Ausgabeordner (Standard: `output`, unterhalb des Ordners, aus dem der Befehl ausgeführt wird). |
| `--format {tiff,tif,png,jpg,jpeg}` | Dateiformat (Standard: `tiff`). PNG/JPEG akzeptieren nur `display` und den Index-Render; siehe [Aufnahme-Exportstufen](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Einzelne Exportstufe (`raw`, `debayered`, `radiance`, `reflectance`, `display`) oder `all` (Standard). |
| `--levels L1,L2,…` | Explizite Teilmenge von Exporttypen (z. B. `raw,radiance,reflectance`); **überschreibt `--processing`**. |
| `--aligned` / `--no-aligned` | **Standard: ausgerichtet, wie die Erfassung desselben Arrays durch die Chloros-App**(die Einstellung**Ausgerichtet** für die Erfassung pro Array ist aktiviert, sofern Sie sie nicht deaktivieren). Der Nicht-Raw-Export jedes Mitglieds (`debayered`, `display`, `radiance`, `reflectance`, das Index-Rendering) wird in den Bildausschnitt der Master-Kamera verschoben und auf die gemeinsame Überlappung der Mitglieder zugeschnitten, sodass die Dateien pixelgenau registriert sind und die gleichen Größen-, Zuschneide- und `Alignment*`-Tags aufweisen wie die der App – kleiner als der Sensor (oder des ROI) um die Offsets der Mitglieder; bei einem ROI wird die Transformation auf das Fenster neu bezogen. `XMP-mapir:AlignmentJSON` gibt `"applied": true` an und liefert das `crop_roi`. Raw-Dateien bleiben in ihrer nativen Größe unverzerrt, enthalten die Transformation jedoch in den Metadaten (`"applied": false`). `--no-aligned` speichert jede Datei in ihrer nativen Größe, unverzerrt. `--aligned` ist die Standardausführung. Unabhängig davon, ob die Verzerrungskorrektur angewendet wird oder nicht, zeichnet jede Dateikomponente die `Alignment*`-Transformation ihrer Kamera auf, sobald das Array über ein Ausrichtungsprofil verfügt – mit Ausnahme einer Kamera, die das Profil nicht ausrichtet (ausgelassen oder durch eine fehlgeschlagene Lösung auf die Identitätsmatrix gesetzt), sowie der Referenzkamera, wenn keine andere Kamera ausgerichtet ist: Diese enthalten keine `Alignment*`-Tags. Liegt für das Array kein Profil vor, wird die Aufnahme unausgerichtet gespeichert, und die CLI gibt den Grund in einer `Alignment:`-Zeile aus, wie dies auch in jedem anderen Fall geschieht, in dem die Verzerrungskorrektur nicht angewendet werden konnte (Elemente mit unterschiedlichen Auflösungen, ein Array ohne Kabelsynchronisation, ein Bild, auf das das Profil nicht abgebildet werden kann). Die automatische Ausrichtung des Arrays erstellt das Profil, sobald sich die Belichtung stabilisiert hat; `align-calibrate` erstellt auf Anfrage eines. |
| `--orient` | Die **Produkte**jedes Mitglieds – `display`, `radiance`, `reflectance`, das `index/`-Rendering und das `composite/` — nach der Ausrichtungsverzerrung an die Kameraausrichtung anpassen: ein Produkt, das durch die**Master**-Ausrichtung, sodass die Elemente weiterhin registriert bleiben; ein nicht ausgerichtetes Produkt (`--no-aligned` oder eine Aufnahme, auf die die Verzerrungskorrektur nicht angewendet werden konnte) entsprechend der Ausrichtung der eigenen Kamera.**Standard: deaktiviert – jede Elementdatei bleibt im nativen Format**, wie es bei Array-Aufnahmen immer der Fall ist. `raw` und `debayered` bleiben in jedem Fall im nativen Format. Jede Datei speichert die Orientierung, die ihre Produkte tragen (siehe [Kameraorientierung](#what-a-captures-folder-looks-like)); die `Alignment*`-XMP-Datei enthält weiterhin die Transformation im nativen Bildausschnitt, die vor der Ausrichtung angewendet wurde, sodass `process` (Verzerrung, dann Ausrichtung) das Live-Ergebnis reproduziert. |
| `--index` / `--no-index` | Das kameraspezifische Vegetationsindex-Overlay speichern bzw. überspringen, sofern eines konfiguriert ist. Standard: rendern. |
| `--no-daq` | Überspringe den `.daq`-Sidecar, wenn keine ausgewählte Ebene die DAQ verwendet (eine reine Rohdatenaufnahme). Standardmäßig wird der Messwert gespeichert, sodass die Bilder offline zu Reflektanz-/Indexwerten nachbearbeitet werden können. `--force-daq` wird weiterhin akzeptiert (Standard). |
| `--daq-tolerance-ms MS` | Die DAQ-Übereinstimmungstoleranz für diese Aufnahmen (Standard 500 ms). |
| `--smart` | Warten, bis sich der AE bei allen Kameras eingependelt hat, bevor ausgelöst wird (siehe [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-Pixelkomprimierung. `none` (Standard seit 1.3.0; 1.2.0 schrieb `deflate`) = unkomprimiert — die Sensorpixel genau so, wie sie ausgelesen werden, ~6,3 MB pro Bild in voller Auflösung und die schnellste Dauerübertragungsrate; `deflate` = verlustfreie zlib L1 + horizontaler Prädiktor, ~4,1 MB pro Bild, langsamer. Beide sind verlustfrei und werden beim Import identisch gelesen. |

> **Einmal geschriebenes TIFF-Format + das Modell mit konstanter Bildrate.**Jede Datei wird**einmal**geschrieben: Pixel, das XMP-Paket, IFD0 Make/Model/CameraSerialNumber sowie die EXIF-Tags `ExposureTime` / `SerialNumber` in einem Durchgang, ohne dass danach etwas überschrieben wird. `--continuous` erfasst aufeinanderfolgende, synchronisierte Gruppen – jede Erfassung übernimmt die Gruppe nach der vorherigen, einschließlich einer, die veröffentlicht wurde, während diese Erfassung noch gespeichert wurde –, sodass es mit der Rate des Arrays läuft. DEFLATE hält den Python-GIL, sodass komprimierte Schreibvorgänge**nicht**über die Schreib-Threads pro Kamera parallelisiert werden; eine dauerhafte Aufnahme in voller Auflösung mit Sensor-Rate erfordert unkomprimierte Schreibvorgänge (Standardeinstellung)**sowie** eine Festplatte der NVMe-Klasse (~600 MB/s Dauerleistung für ein Array mit 9 Kameras bei ~10 fps). Der gleiche Regler wird als `compression` auf `POST /api/camera/array/capture` bereitgestellt.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Serienbildmodus (`--continuous`)

`array-capture --continuous` (ohne `--interval` oder `--smart`) übergibt den Ablauf an die **eigene kontinuierliche Aufzeichnung des Backends** – die Sitzung, in der der kontinuierliche Modus der GUI läuft. Das Backend speichert jede synchronisierte Gruppe selbst, wobei zwei Aufnahmen gleichzeitig laufen und zwischen den Gruppen keine Anfrage von der CLI erfolgt, sodass es mit einem schnellen Array Schritt halten kann, bei dem eine Anfrage pro Aufnahme in Verzug gerät (gemessen wurden 73–77 Aufnahmen/s bei einem Array mit 9 Kameras und einem ROI von 512 Pixeln). Jede der oben genannten Exportoptionen gilt für jede einzelne Aufnahme.

- **Grenzen:** `--count N` stoppt nach N Aufnahmen (eine Aufnahme = eine synchronisierte Gruppe: die Dateien aller Mitglieder); `--duration S` nach S Sekunden. Ohne eine dieser Einstellungen läuft das Programm so lange, bis `Ctrl+C` erreicht ist, das Laufwerk voll ist, ein Fehler auftritt oder das Array nicht mehr verfügbar ist. `Ctrl+C` beendet die Sitzung des Backends und und gibt anschließend die Zusammenfassung aus.
- **Fortschritt** alle 2 s, wobei `, N failed` und `, N groups skipped` angehängt werden, wenn der Wert ungleich Null ist (eine übersprungene Gruppe ist eine vom Array veröffentlichte Gruppe, für die keine Erfassung stattgefunden hat):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Keine Auflistung pro Datei.** Der Durchlauf endet mit einer zusammenfassenden Zeile – Erfassungen, Rate, Dateien und der Ausgabeordner –, gefolgt von `N file(s) failed to write.` / `N group(s) skipped.`, wenn der Wert ungleich Null ist:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Bricht ab** bei vollem Laufwerk mit `Continuous capture stopped: the drive is full.` sowie bei einem Fehler oder dem Ausfall des Arrays mit `Continuous capture stopped: <reason>.` (beides auf stderr).
- **Beendigungsstatus:** `0`, wenn die Erfassungen gespeichert wurden und alle Dateien geschrieben wurden, einschließlich `Ctrl+C`; `1` bei jeder Datei, deren Schreiben fehlgeschlagen ist, bei einem vollen Laufwerk, einem Fehler, dem Verschwinden des Arrays oder gar keiner Aufzeichnung.
- **Microsoft Defender:** Wenn Defender einen CPU-Kern oder mehr für das Scannen der von der Aufzeichnung geschriebenen Dateien beansprucht, gibt der Lauf einmalig auf stderr Folgendes aus: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Siehe die [Systemeinstellungen](#windows-exe) des Installationsprogramms.
- Es ist jeweils nur eine fortlaufende Erfassung pro Array zulässig; eine zweite wird mit der Fehlermeldung `a continuous capture is already running for this array` abgelehnt. Über HTTP lautet die Sitzung `POST /api/camera/array/continuous/start` (der `/api/camera/array/capture`-Body plus `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` und `POST /api/camera/array/continuous/stop`.

**Intervall**(`--interval S`, mit oder ohne `--continuous`) und**`--continuous --smart`** behalten die Schleife pro Anfrage bei: eine Erfassungsanfrage pro Durchlauf – zu jedem geplanten Zeitpunkt für „Intervall“, nachdem sich AE für `--smart` eingependelt hat – wobei die Dateien jedes Durchlaufs wie im Einzelmodus aufgelistet sind, am Ende eine Zeile „`Done — N capture pass(es).`“ steht und „`--count N`“ die Anzahl der Durchläufe angibt.

### `array-record` – Video/GIF mit kombiniertem Index (in Überwachungsqualität)

Zeichnet die **Live-Kombinationsindexansicht** des Arrays in eine `.avi`-Datei (und optional in eine `.gif`-Datei) auf. Bei geöffneter Kombinationsansicht der Benutzeroberfläche wird diese Ansicht so aufgezeichnet, wie sie angezeigt wird. Ist keine Ansicht geöffnet, rendert das Backend die Ansicht selbst: den Index, den Sie mit `--expr` übergeben, oder standardmäßig den NDVI des Arrays, sobald das Array ausgerichtet ist. Der Standard-NDVI verwendet die Rot- und NIR-Bänder, die 660 nm und 850 nm am nächsten liegen, genau wie die Live-Ansicht. Eine während der Aufzeichnung geöffnete GUI-Ansicht wird aufgezeichnet, solange sie geöffnet bleibt. Der Fortschritt wird alle 2 s abgefragt, und die Aufzeichnung wird bei `--duration`, `Ctrl+C` oder wenn der Recorder den Vorgang selbst beendet.

Eine Aufzeichnung, bei der kein Bild geschrieben wird, schlägt fehl: Der Recorder meldet `no-frames: …` (das Array ist möglicherweise noch dabei, sich auszurichten oder die Belichtung einzustellen) und der Befehl **beendet `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--array-id ID` | nur Array | Ziel-Array (weglassen, wenn nur eines angeschlossen ist). |
| `-o, --output DIR` | `output` | Ausgabeverzeichnis. Ein relativer Pfad bezieht sich auf den Ordner, von dem aus den Befehl ausführen. |
| `--fps F` | `10` | Bildrate der Aufzeichnung. |
| `--duration S` | bis Strg+C | Automatischer Stopp nach `S` Sekunden. |
| `--gif` | aus | Auch ein animiertes GIF schreiben. |
| `--gif-only` | aus | Nur ein GIF erstellen (kein `.avi`). |
| `--expr EXPR` | der NDVI-Wert des Arrays | Indexausdruck, der aufgezeichnet werden soll, wenn keine GUI-Ansicht geöffnet ist, in den Bandnamen des Arrays (z. B. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; siehe [Bandnamen](#index--vegetation-maths)). Ein kurzer Farbname (`NIR`, `Red`) funktioniert nur, wenn ein Band des Arrays diesen Namen trägt. Eine geöffnete GUI-Ansicht wird stattdessen so aufgezeichnet, wie sie angezeigt wird. |

### `array-burst` – Raw-Bayer-Serienaufnahme mit hoher Bildrate (für Analysezwecke)

Liest direkt aus dem synchronisierten Gruppenpuffer der Aufnahmeschleife – **keine Kalibrierungskette, kein exiftool, keine Live-Ansicht erforderlich** – und läuft somit mit der vollen Bildaufnahmerate der Kamera. Schreibt RAW-Bilder + ein Manifest pro Bild + einen `.daq`-Eintrag pro eindeutigem DLS-Messwert unter `<output>/bursts/<base>/`. Offline nachbearbeiten (nächster Befehl) oder `--build` übergeben, um dies sofort nach Beendigung durchzuführen.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--array-id ID` | nur Array | Ziel-Array. |
| `-o, --output DIR` | `output` | Ausgabeverzeichnis (Burst landet in `<DIR>/bursts/<base>/`). |
| `--duration S` | bis Strg+C | Automatischer Stopp nach `S` Sekunden. |
| `--max-frames N` | unbegrenzt | AutomatischerStopp nach `N` Rohbildern. |
| `--orient` | aus | Die aus diesem Burst erstellten Produkte werden um die Kameraausrichtung gedreht – kameraspezifische Produkte um die ihrer eigenen Kamera, der kombinierte Index nach der des Masters – wie bei `array-capture --orient`. Die Rohbilder bleiben im Originalformat; `manifest.json` der Bildserie zeichnet die Ausrichtung jedes einzelnen Bildes sowie die Option auf. |
| `--build` | aus | Nach dem Stoppen die Bildserie sofort erneut verarbeiten (wie bei `array-build-video`). |
| `--products …` | `combined:index` | Bei `--build`: Welche(s) Video(s) soll(en) erstellt werden (siehe unten). |
| `--fps F` | `10` | Bei `--build`: Bildfrequenz (fps) des Ausgabevideos. |
| `--save-tiffs` | aus | Zusammen mit `--build`: Speichert zusätzlich kalibrierte TIFFs pro Bild (jeweils mit der entsprechenden [Unmix-Clamp-Maske](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on), wenn ein Pixel geklemmt wurde; das Build-Ergebnis listet sie als `clamp_masks` auf). |
| `--gif` | aus | Mit `--build`: zusätzlich animierte GIFs schreiben. |

### `array-build-video` — Offline-Nachbearbeitung einer gespeicherten Bildserie

Jeder Rohbildrahmen wird zeitlich mit dem nächstgelegenen gespeicherten `.daq`-Messwert abgeglichen und durch die **gleiche Radianz-/Reflexionsgrad-/Index-Kette wie die Import-Pipeline** geleitet, wodurch ein oder mehrere Videos gerendert werden.

`--products` ist eine durch Kommas getrennte Liste von `kind:level`-Elementen, wobei `kind` ∈ `per_cam` | `combined` und `level` ∈ `radiance` | `reflectance` | `index`. Ein einzelnes `level` (ohne `kind:`) wird standardmäßig auf `per_cam` gesetzt. Der Standardwert ist `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flag | Standardwert | Beschreibung |
| --- | --- | --- |
| `--burst-dir DIR` | (erforderlich) | Pfad zum Burst-Ordner (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-Liste, wie oben. |
| `--fps F` | `10` | Bildfrequenz des Ausgabevideos. |
| `--save-tiffs` | aus | Speichert zusätzlich zu den Videos auch kalibrierte TIFF-Dateien für jedes Einzelbild. |
| `--gif` | aus | Außerdem animierte GIFs schreiben. |

**Ein Video enthält keine Clamp-Maske** (ein Bildstrom, 8 Bit). Stattdessen enthält das Build-Ergebnis `clamp_fractions`: pro Videoprodukt den Anteil jedes Einzelbildes an den Pixeln, die der Unmix in dem Band geklemmt hat, das sein R-/G-/B-Kanal enthält – das `Chloros:Band{R,G,B}UnmixClampFraction`, mit dem das Strahlungs-/Reflexionsprodukt dieses Einzelbildes versehen würde (für den kombinierten Index, der aus den Daten aller beteiligten Kameras besteht). Der Lauf gibt das schlechteste Bild pro Produkt in einer Zeile mit dem Format `Clamp:` aus. `--save-tiffs` fügt die Pixel-Masken für die TIFFs hinzu, in „`clamp_masks/<level>/`“ neben dem Ordner „`<level>/`“ der TIFF-Dateien.**Ein vorzeitig abgebrochener Build hinterlässt kein beschädigtes Video.** Jedes Video wird als `<name>.partial.avi` geschrieben und erhält seinen endgültigen Namen erst, wenn es fertiggestellt ist; daher bleibt ein älteres Video mit diesem Namen bestehen, bis sein Ersatz vollständig ist. Führen Sie den Build erneut aus, um ein zurückgelassenes `.partial.avi` zu ersetzen.

Der Durchlauf gibt jedes geschriebene Video aus, gefolgt von allen `Skipped:`-Ergebnissen (eine angeforderte Kamera/Ebene – oder der kombinierte Index –, die kein Video erzeugt hat, mit Angabe des Grundes – z. B. kein Kalibrierungsbündel, keine DLS-Messung für die Reflexion oder kein Rot-/NIR-Bandpaar für den kombinierten Index) sowie alle `Note:`-Einträge. Eine RGB-Kamera, für die Radiance oder Reflexion angefordert wurde, erhält stattdessen ihr Anzeigevideo, das weißabgeglichen ist, sofern das Kalibrierungspaket der Kamera verfügbar ist; eine Kamera, die ohne dieses Kalibrierungsbündel rendert, wird in einem `Note:` benannt. Es **gibt einen Wert ungleich Null zurück**, wenn ein angefordertes Produkt übersprungen oder nicht wie angefordert erstellt wurde (ein konfigurierter Indexausdruck wird offline nicht ausgewertet); die erstellten Videos werden dennoch geschrieben und aufgelistet.

> **Wählen Sie den richtigen Recorder aus.** `array-record` ist für die *Überwachung geeignet* – er zeichnet das gerenderte Live-Composite auf: die kombinierte Ansicht der GUI oder, wenn keine GUI geöffnet ist, die NDVI-Ansicht (oder `--expr`) des Backends. `array-burst` → `array-build-video` ist *analysetauglich* – es speichert Rohsensordaten mit voller Rate und rekonstruiert anschließend kalibrierte Strahlungs-/Reflexions-/Index-Videos, wobei keine Live-Ansicht erforderlich ist.

### Mono (M3M) Einbandkameras

Die **M3M**-Reihe ist das Mono-Pendant zur Bayer-**M3C**: ein Schmalband-Interferenzfilter pro Kamera (`M3M-<lens>-F<wavelength>`, z. B. `M3M-L87-F685`), sodass der Sensor ein**einziges Graustufenband** ohne Bayer-Mosaik liefert. Es gibt nichts zu demosaiken, kein kanalübergreifendes Übersprechen, das entmischt werden müsste, und keinen Weißabgleich, der eingestellt werden müsste – die gesamte Farbpipeline für die RGB-Anzeige kommt hier schlichtweg nicht zum Tragen.

Was das in der CLI bedeutet:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**erkennen eine Mono-Kamera und**überspringen den Vorgang mit einer einzeiligen Meldung**, anstatt sinnlose Einstellungen zu übertragen. Sie laufen weiterhin normal, wenn in derselben Sitzung eine RGB- oder Bayer-M3C-Kamera verwendet wird.
- **Ein Bayer-Pixelformat bezeichnet dessen Farbtiefe.** Ein M3M bietet nur Mono-Formate an, daher legt ein Bayer-Format aus `--pixel-format` oder `--preset` das Mono-Format derselben Tiefe fest: `BayerRG8` und `--preset high_speed` ergeben `Mono8`; `BayerRG12`, `high_quality` und `triggered` ergeben `Mono12`; ein gepackter Name bleibt gepackt (`BayerRG12p` ergibt `Mono12p`). Die Dateien und der Name `lattice format` bezeichnen das Mono-Format. Ein Array, das M3C- und M3M-Kameras mischt, nimmt ein Format auf die : Bayer auf jeder M3C, deren Mono-Farbtiefe auf jeder M3M. Ein Name entspricht seiner Bittiefe: Das Gleiche gilt auch umgekehrt, sodass eine M3C niemals in ein Mono-Format umgewandelt wird (`--pixel-format Mono12` ist `BayerRG12` auf einer M3C, unabhängig davon, welche Kamera der Master des Arrays ist), und `lattice format --set` wird auf dieselbe Weise zugeordnet und gibt das festgelegte Format aus.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** funktionieren weiterhin – Radiance und Reflektanz sind *bandweise* radiometrische Karten und für ein einzelnes Band vollkommen eindeutig definiert. Mono-Bilder enthalten eine **Identitäts**-Sensorantwortmatrix (keine 3×3-Entmischung), sodass die Ebene die Kalibrierungsberechnungen unverändert durchläuft.
- **Eine einzelne Mono-Kamera kann keinen Vegetationsindex erzeugen.**NDVI, NDRE usw. benötigen mindestens zwei Bänder (z. B. Red + NIR). Um einen Index aus Mono-Hardware zu erhalten, richten Sie**mehrere** M3M-Kameras auf unterschiedliche Wellenlängen aus, fügen Sie sie zu einem Multiband-Stack zusammen und berechnen Sie *diesen* als Index:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` öffnet die Kameras selbst (eine [temporäre Sitzung](#multi-camera-transient-sessions)), schließen Sie also zunächst nichts an: `--serials`, andernfalls die im Profil aktivierten Kameras (siehe [Index / Vegetationsberechnungen](#index--vegetation-maths)). Solange Chloros eine Kamera hält (die GUI, `cam-connect`, `array-connect`) wird der Zugriff mit dem HTTP-Statuscode 409 `CAMERA_HELD` verweigert und das Programm beendet `1`; geben Sie diese mit `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all` frei.

`--channel`-Symbole müssen **genau** mit den Kanalnamen der Voreinstellung übereinstimmen (Groß-/Kleinschreibung beachten; NDVI-Werte sind in Kleinbuchstaben `red`, `nir` — siehe `--list-presets`), und der Band-Seitenname bezeichnet ein Band im ausgerichteten Stapel (im Offline-Modus werden auch 0-basierte Bandindizes akzeptiert, z. B. `--channel red=0 --channel nir=1`).

Das Unterscheidungsmerkmal im gesamten Stapel ist das Token `M3M` in der Modellzeichenfolge (es erscheint niemals in einer `M3C`-Zeichenfolge) und wird in der GUI/im SDK als `is_mono` angezeigt.

---

## Einrichtung und Optimierung der Host-Netzwerkkarte (LATTICE-Arrays)

LATTICE-Kameras streamen GVSP über den Ethernet-Adapter des Hosts. Bei Arrays mit mehreren Kameras sind daher der **Treiber**und die**Größe des Empfangsrings** des Adapters ebenso wichtig wie die Verbindungsrate. Falsche Einstellungen werden im Bereich „Array-Einstellungen“ als `FRAMES WILL DROP` / `Reduce ROI to enable`-Gate im Bereich „Array-Einstellungen“ (sowie in `lattice network-analysis` / dem SDK unter `analyze_array_network()`), selbst wenn die Kameras selbst einwandfrei funktionieren.

### USB-10-GbE-Adapter – Realtek RTL8157 („Realtek USB 10GbE Family Controller“)

Die folgenden Einstellungen sind Eigenschaften des Windows-Treibers. Unter Linux unterliegt ein USB-Adapter ebenfalls einer Obergrenze (400 MB/s auf x86_64, 200 MB/s auf NVIDIA Jetson) (siehe [Array-fps &amp; Burst-Modell](#array-fps--burst-model)), und die Socket-Empfangspuffer des Kernels spielen für jeden Adapter eine Rolle – siehe [Linux-Hosts](#linux-hosts--socket-receive-buffers).

| Element | Erforderlicher Wert | Warum es wichtig ist |
| --- | --- | --- |
| **Treiberversion**|**≥ v10.67 (Jan. 2026)**, INF `rtump64x64sta.inf` | Der veraltete**2016**-Treiber (v10.65, `rtump64x64.inf`) behandelt das Herunterfahren und Fehlerprüfungen (Bugchecks) mit**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**beim Herunterfahren, Neustart oder in den Ruhezustand falsch. Der Übergang hängt sich auf (~5 Minuten Timeout), der Benutzer schaltet das Gerät zwangsweise aus, und wiederholte unsaubere Herunterfahren**beschädigen das WMI-Repository**(PowerShell/Tools zeigen Fehler mit `Invalid class` an) und**blockieren den USB-Stack** beim nächsten Systemstart (Netzwerkkarte lässt sich nicht aktivieren; USB-Laufwerke werden nicht mehr erkannt). Führen Sie ein Update von realtek.com (oder vom Dongle-Hersteller) durch, bevor Sie sich auf saubere Neustarts verlassen. |
| **Empfangspuffer**— Schlüsselwort `ReceiveBufferLen` |**256**(Treibermaximum) | Der RX-Ring der Netzwerkkarte. Der Treiber-Standardwert von**32**lässt nur ~0,26 MB nutzbaren Ring übrig – viel zu wenig für einen Multi-Cam-Burst –, sodass das Array-Panel `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` meldet und Verbindungen blockiert. Bei**256**ist der Ring groß (**~13,5 MB, gemessen auf dem 10-GbE-Host im Labor**), was der RX-Pipeline echten Spielraum für Multi-Kamera-GVSP-Bursts bietet. (Ob eine bestimmte Konfiguration tatsächlich *verbunden* wird, wird durch zwei Prüfungen entschieden – die **drain-aware**Zulassungsprüfung und die**Aggregat-Überbelegungsprüfung** – nicht durch einen reinen Vergleich von Burst und Ring; siehe [Array-FPS- und Burst-Modell](#array-fps--burst-model).) |
| **Empfangs-URBs**— Schlüsselwort `PendingReceives` |**64** (max.) | USB-Anforderungsblöcke in Bearbeitung; sollte zusammen mit den Empfangspuffern erhöht werden, um Bursts aufzunehmen. |
| **Jumbo-Frame** — Schlüsselwort `*JumboPacket` | **9014** | Erforderlich für 9000-Byte-GVSP-Pakete (6× weniger Pakete/Frame als bei 1500). |

> ⚠️ **Eine Aktualisierung des Netzwerkkartentreibers SETZT diese erweiterten Eigenschaften auf die Standardwerte ZURÜCK.**Nach der Aktualisierung oder dem Austausch des Adaptertreibers müssen Sie**die Einstellungen erneut anwenden** `ReceiveBufferLen=256` und `PendingReceives=64` erneut an, da sonst das Array-Panel erneut sperrt, obwohl sich „an der Hardware nichts geändert hat“. Dies ist der häufigste Grund dafür, dass ein zuvor funktionierendes System plötzlich keine Verbindung mehr herstellt.

Wenden Sie die Befehle in einer **mit erhöhten Rechten** ausgeführten PowerShell an (ersetzen Sie den Namen Ihres Adapters, z. B. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` gilt für USB-10-GbE-Adapter.** Es erkennt nun den Adaptertyp und passt das richtige „receive-ring“-Schlüsselwort an: `*ReceiveBuffers`→2048 für PCIe-NICs (Intel I219, usw.) oder `ReceiveBufferLen`→256 + `PendingReceives`→64 für den Realtek **USB**-10-GbE-Controller (der `*ReceiveBuffers` nicht bereitstellt). Die Zielwerte werden auf den vom jeweiligen Treiber gemeldeten Maximalwert (`NumericParameterMaxValue`) begrenzt, sodass niemals ein Wert außerhalb des zulässigen Bereichs geschrieben wird. Führen Sie den Befehl in einem **Terminal mit erhöhten Rechten** aus; wie bei jeder registrierungsbasierten Optimierung wird die Änderung erst nach einem Neustart des Adapters oder des Systems wirksam. Die oben genannten manuellen Befehle `Set-NetAdapterAdvancedProperty` bleiben eine gute Alternative – sie werden live (durch erneutes Binden des Adapters) ohne Neustart angewendet.

### Linux
-Hosts – Socket-Empfangspuffer

Unter Linux
, einschließlich NVIDIA Jetson, sind die Socket-Empfangspuffer des Kernels ebenso wichtig wie der Empfangsring der Netzwerkkarte. Bei der Kernel-Standardeinstellung (`net.core.rmem_max` / `net.core.rmem_default` = 212.992 Byte) führt eine Flut von GVSP-Paketen einer Kamera zu einem Überlauf des Stream-Sockets, noch bevor Chloros
diese auslesen kann. Die Pakete werden im Kernel verworfen, während die Netzwerkkarte selbst keinen Verlust meldet, und ein Array verliert Frame-Gruppen und drosselt seine Bildrate. Gemessen an einem Array mit 4 Kameras hinter einem USB-10-GbE-Adapter führte die Erhöhung beider Werte auf 32 MB dazu, dass das Array nach dem Verbindungsaufbau von 4,96 auf 7,52 fps stieg und die kontinuierliche Rohdatenerfassung von 4,7 auf 7,6 Aufnahmen/s, ohne dass Pakete im Kernel verworfen wurden.

- **Das .deb-Paket legt diese Werte fest.** Bei der Installation des Pakets wird `/etc/sysctl.d/60-chloros-gige.conf` (beide auf `33554432`) geschrieben und sofort angewendet. Ein bereits höher eingestellter Wert bleibt unverändert.
- **`lattice network` überprüft diese Werte.** Wenn einer der beiden Werte niedriger ist, wird `socket receive buffers too small for GigE Vision (…)` unter *„Issues found“* aufgeführt, und das Protokoll des Backends enthält dieselbe Warnung beim ersten Verbindungsaufbau einer Kamera oder eines Arrays.
- **Um sie manuell zu erhöhen** (bis zum nächsten Neustart; die Datei des Pakets wendet sie bei jedem Systemstart an):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` erhöht diese Werte ebenfalls und behält sie bei, jedoch nur, wenn das Backend als Root ausgeführt wird. Ein Backend, das unter Ihrem Benutzerkonto läuft (eines, das mit `chloros-backend` oder [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless)) gestartet wurde), kann die Kernel-Einstellungen nicht ändern und gibt stattdessen die Zeile `sudo sysctl` aus.

**Die automatische Adapteroptimierung betrifft nur verbundene Adapter.** Beim ersten Anschließen einer Kamera optimiert ein als root ausgeführtes Backend die kabelgebundenen Adapter, deren Empfangsring klein ist. Es optimiert nun nur noch Adapter mit einer Verbindung, sodass ein nicht angeschlossener Onboard-Port seine MTU beibehält und die Empfangspuffer unverändert bleiben. `lattice network` listet die MTU eines nicht angeschlossenen AdaptersMTU eines nicht angeschlossenen Adapters nicht mehr als Problem auf. `lattice network`, `--fix`, `lattice probe` und die Adapteroptimierung berücksichtigen nur kabelgebundene Ethernet-Ports, nicht den CAN-Bus eines Jetson (`can0`) oder die Verbindungen im USB-Gerätemodus (`usb0`, `usb1`), die er einem Host-PC bereitstellt; ein USB-Ethernet-Adapter für Kameras zählt jedoch weiterhin.

### Netzwerk-Grundlagen (alle LATTICE-Verbindungen)

- **Adressierung:** Link-Local `169.254.0.0/16` (GigE Vision LLA). Der Host erhält eine statische Adresse `169.254.x.x/16`; Kameras und DAQ-E weisen sich selbst Adressen im selben Bereich zu. Kein DHCP/Gateway erforderlich.
- **Paketgröße:**Bevorzugen Sie Jumbo (9000), lassen Sie die automatische Erkennung jedoch die Größe ermitteln – diese umgeht die 1500-Byte-ICMP-Obergrenze der Kamera über eine GVSP-Abfrage und wählt daher Jumbo, sofern die Leitung dies tatsächlich unterstützt.**Jede Kamera wird einmal pro Backend-Sitzung gemessen und erneut, wenn sich ihre Verbindung ändert (anderer Adapter, andere Verbindungsgeschwindigkeit oder eine unterbrochene Verbindung, wie es beim Austausch eines Switches der Fall ist); bei erneuter Verbindung wird das Ergebnis wiederverwendet. Nach einer Änderung hinter einem Switch, der in Betrieb geblieben ist, starten Sie das Backend neu, um erneut zu messen.**Eine erste Verbindung kann bei dieser Überprüfung gelegentlich einige Minuten dauern, was im Protokoll vermerkt wird, wenn dies der Fall ist. Setzen Sie „pin“ mit `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` nur, wenn Sie es besser wissen als die Probe, und bevorzugen Sie eine befehlsbezogene Einstellung gegenüber einer permanenten: Ein „pin“ umgeht die Probe, wenn der Pfad also tatsächlich keine 9000 übertragen kann, läuft**jede** Erfassung mit `SC_ERR_TIMEOUT -1011` ab (siehe [Umgebungsvariablen](#environment-variables)).
- **Der RX-Ring skaliert mit `ReceiveBufferLen`:**Bei der Standardeinstellung `32` beträgt der nutzbare Ring ~0,26 MB (zu klein für jede Multi-Cam-Burst-Aufnahme); bei der maximalen Einstellung `256` ist er groß (~13,5 MB, gemessen auf dem 10-GbE-Host im Labor) und bietet somit echten Spielraum. Ob eine Konfiguration verbunden wird, entscheidet sich dann durch die drain-bewusste Zulassungsprüfung**und** die unten beschriebene Prüfung der aggregierten Überbelegung – nicht durch einen reinen Vergleich von Burst und Ring.

### Array-FPS- und Burst-Modell

So lesen Sie das Fenster „Array-Einstellungen“ (sowie `lattice analyze-array` / `analyze_array_network` im SDK
):

- **Der Burst wird pro Kamera im tatsächlichen Pixelformat der jeweiligen Kamera summiert.**Mono-**M3M**-Kameras streamen**Mono12 (2 B/px)**oder, bei Verwendung von `--pixel-format`, das Mono-Format ihrer Tiefe (`BayerRG8` entspricht `Mono8`, 1 Bit/Pixel);**M3C**-Bayer-Kameras streamen 8- oder 12-Bit (die M3C-Firmware gibt stillschweigend BayerRG12 aus, selbst wenn BayerRG8 angefordert wird). Ein Vollauflösungs-Frame von 4 Kameras ist also**~12,6 MB, wenn alle 8-Bit-Daten liefern, aber ~25 MB bei drei 12-Bit-Mono-Kameras**. Die Projektion ermittelt das Format jeder Kamera anhand ihres Modells (Identitäts-Cache), sodass der Datenburst genau dem entspricht, was tatsächlich über die Leitung übertragen wird – und nicht einer pauschalen Annahme von BayerRG8.
- **Ein USB-Ethernet-Adapter ist unabhängig von seinen technischen Angaben auf 200 MB/s begrenzt.** Die Effizienztabelle, die eine Verbindungsrate in einen Dauerwert umrechnet, basiert auf PCIe; eine USB-Netzwerkkarte gibt zwar ihre *Ethernet*-Verbindungsrate an, ist jedoch durch den USB-Bus und ihren Treiber begrenzt. Ein USB-10-GbE-Dongle erreichte früher ~1063 MB/s „Dauerleistung“ – ein Wert, der nie überprüft wurde –, und die daraus resultierende Ratenbegrenzung führte zu einer Beschädigung von 6–18 % der Frames, während weiterhin eine einwandfreie Ziel-FPS-Rate gemeldet wurde. USB-angeschlossene Netzwerkkarten sind nun absolut begrenzt: **200 MB/s** unter Windows
und auf NVIDIA Jetson, **400 MB/s** unter Linux
x86_64 (unter Linux
werden für einen Adapter an einem USB-2.0-Anschluss **30 MB/s** zugewiesen, und in der Leistungsbeschreibung wird empfohlen, ihn an einen USB-3-Anschluss anzuschließen) (die Begrenzung liegt beim USB-Bus und Treiber des Hosts, sodass sie nicht mit der Nennleistung skaliert; ein USB-1-GbE-Adapter erreicht ~80 MB/s und ist davon nicht betroffen). Unter Linux
wird der Adapter über sysfs identifiziert (mit Fallback auf `ethtool -i`’s `bus-info`). Jeder Wert wurde gemessen: Unter Windows
kam es bei demselben 4-Kamera-Array bei Datenraten über 200 MB/s zu 6–10 % fehlerhaften Bildern; unter Linux
x86_64 lief derselbe Adapter bei jeder Obergrenze bis zu 480 MB/s fehlerfrei (273 MB/s wurden geliefert, was der eigenen Grenze der Kameras entspricht). Auf dem NVIDIA Jetson lief derselbe Adapter nur bei 200 MB/s fehlerfrei und verlor oberhalb dieser Grenze zeitweise Bilder. `wire_ceiling_source` im Leistungsdatensatz beschreibt dies in Worten, und `nic_is_usb` kennzeichnet es. Beides kann mit `--wire-ceiling-mbps`.
- **Die Zulassung erfolgt drain-bewusst, nicht nach „ganzer Burst vs. Ring“.** Ein gleichzeitiger Burst muss nur in den *transienten Rückstau* passen = `max(0, Σ per-cam arrival − host drain) × emit_window`, nicht der gesamte Burst. In einer Fabric mit schnellem Host und langsamen Kameras (ein **PCIe**10G-Host + 4× 1-GbE-CAMs: Ankunft ≈ 320 MB/s, Drain ≈ 1063 MB/s) entleert sich der Host schneller, als die Kameras sich füllen; der Rückstau beträgt ≈ 0, sodass „full-res sim-emit“**zulässig**ist, obwohl der 25-MB-Burst den 13,5-MB-Ring überschreitet. Schließt man dieselben vier Kameras an einen**USB**-10-GbE-Adapter unter Windows
an, beträgt die Abflussrate 200 MB/s statt 1063 – die Ankunftsrate übertrifft sie, und der Verlust zeigt sich als beschädigte Frames statt als geringere Bildrate. Auf einem 1-GbE-Host führt der DLThr-Mindestwert der Kameras von 31,25 MB/s dazu, dass die Ankunftsrate die Abflussrate übersteigt → es wird korrekt **blockiert** (für *diese* Klasse von Blockierungen: ROI reduzieren oder Binning ≥ 2 verwenden). Die Zulassung erfolgt über eines von **zwei** Verbindungsgates – das andere ist die unten beschriebene Überprüfung der aggregierten Überbelegung.
- **Die prognostizierte Bildrate (fps) entspricht dem Zyklus der langsamsten Kamera.**Bei der simultanen Ausgabe sendet jede Kamera ihr Bild zur gleichen Zeit, sodass das Array im Zyklus der längeren der beiden Zeiten läuft: entweder der Sensorauslesezeit oder der Ausgabedauer des langsamsten Mitglieds (Bildbytes ÷ Ausgaberate: die**Zugriffsverbindung** der Kamera, 1 GbE ≈ 80 MB/s, oder ihr Anteil an der maximalen Übertragungsrate des Hosts, falls diese geringer ist). Jede Belichtung überlappt das Auslesen des vorherigen Frames, sodass die Belichtung nicht in den Zyklus einfließt: Die automatische Belichtung füllt die durch die Rate vorgegebene Periode aus, erhöht dann die Verstärkung, und eine dunkle Szene senkt die Rate nicht mehr (in 1.2.0 war dies noch der Fall). Nur eine manuelle Belichtung, die länger als die Periode dauert, verlangsamt den Auslöser (siehe [Belichtung bei einem Array-Element](#exposure-on-an-array-member)). Das Hinzufügen von Kameras verlangsamt das Array erst, wenn sich deren Wire-Anteile verringern. Bei der seriellen Emission (`slip-emit-and-capture`) addieren sich stattdessen die Emissionszeiten der Kameras, und die Belichtungszeit jedes Elements wird in die Staffelung einkalkuliert. Gemessen bei vollem Sensor, 12 Bit, auf einem Array mit 9 Kameras und einem PCIe-10-GbE-Host: 10,7 fps bzw. 14,0 fps mit [gepacktem Transport](#packed-12-bit-transport---packed).
- **Nach einer kontinuierlichen Aufnahme gibt die Projektion an, welchen Wert sie beibehalten hat.** Eine Aufnahme von 30 s oder mehr bei einer automatischen Bildrate zeichnet die Bildrate auf, auf die sich das System bei dieser Array-Größe auf diesem Computer eingependelt hat; die kurzen Versuche des Bildratensteuerers mit einer höheren Bildrate werden dabei außer Acht gelassen. Von da an geben die Zeile von `array-connect` und das Fenster „Array-Einstellungen“ diese Rate an, wenn sie unter der des Modells liegt, zusammen mit der Ursache für die Begrenzung: `clock-bound` (das Array lief mit der Rate, auf die es sich eingependelt hatte, während die Grab-Schleife auf ihren Trigger-Takt wartete), `loop-bound` (der Grab-Schleife blieb keine Zeit mehr), `capture-bound` (die Erfassung geriet in Rückstand und übersprang Gruppen) oder `measured` (es gibt keinen Hinweis darauf, um welchen Fall es sich handelt). Der Wert des Modells ist in `wire_fps_max` enthalten. Das Array steigt weiterhin auf den Wert an, den die Leitung überträgt.
- **Ein Array funktioniert in jedem Netzwerk – die Bildrate ist entscheidend.**Jede Kamera erhält einen gleichmäßigen Anteil an der**kollisionssicheren Leitungsgrenze**(`sustained × sim_emit_factor`), synchronisiert mit GevSCPD, und alle lösen weiterhin auf denselben Trigger aus. Bei einer langsamen Verbindung ist der Anteil gering und die Bildrate sinkt entsprechend: 8 12-Bit-Kameras mit voller Auflösung über 1 GbE ohne Jumbo-Frames erhalten jeweils 52 / 8 = 6,5 MB/s, was etwa**1 fps**(etwa 1,3 fps komprimiert), während ein 10G-Host sie mit 11–14 fps betreibt. Wenn die Kameras nicht so langsam getaktet werden können – lesen Sie den GevSCPD-Bereich jeder Kamera aus, um dies festzustellen –, wechselt das Array zum**seriellen**Emitieren (`slip-emit-and-capture`): Die Kameras nutzen die Leitung abwechselnd mit voller Verbindungsrate und werden nacheinander getriggert, sodass zwar jeder Frame ankommt, die Mitglieder jedoch**nicht gleichzeitig exponiert**werden. `analyze-array` und die „connect“-Ausgabe bezeichnen den Modus; setze `--force-tier sim-capture-sim-emit` auf „pin“, um den Fallback zu verweigern. Die Frame-Größe spielt bei dieser Aufteilung keine Rolle, daher**erhöhen Binning und ein kleinerer ROI zwar die Bildrate, ändern aber nichts daran, ob ein Array passt**. Nur eine Leitung, die das Array nicht einmal seriell übertragen kann, ist**überbelegt**– z. B. bei weitaus mehr Kameras, als eine Leitung bewältigen kann, oder wenn viele andere Kameras darauf streamen – und dort ist das Symptom ein GVSP-Paketverlust, nicht eine niedrigere Rate: `analyze-array` setzt die Werte für die erreichbaren fps auf Null und gibt `** OVER-SUBSCRIBED**` aus; die Verbindung wird mit einer deutlichen Warnung hergestellt, und eine HTTP
-Verbindung mit `pin_resolution: true` (Strict-Modus) **wird abgelehnt**. Abhilfemaßnahmen sind weniger Kameras, Jumbo-Frames durchgehend oder eine schnellere Netzwerkkarte. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` stuft die Ablehnung bei Benchmark-Tests auf eine deutliche Warnung herab – siehe [Umgebungsvariablen](#environment-variables).

### Array-Zustand – welches Subsystem verliert Frames

Das `GET /api/camera/array/<array_id>/capability` eines verbundenen Arrays enthält einen Live-
`health`-Block, der in einem rollierenden **10-Sekunden**-Fenster neu ausgewertet wird. Er unterteilt den Frame-Verlust
in die beiden Ursachen, die gegensätzliche Abhilfemaßnahmen erfordern, anstatt eine einzige „unvollständige“
Rate zu melden, die keine der beiden nennt:

| Feld | Bedeutung | Betroffenes Subsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pro serieller Schnittstelle) | Der Frame **kam an und wies strukturelle Fehler auf**— GVSP-Paketverlust. |**Netzwerk**: Leitungsbandbreite, Pacing, NIC-RX-Ring, MTU |
| `never_arrived_rate_pct` (pro serieller Schnittstelle) | Der Frame **ist überhaupt nicht angekommen**— die Kamera hat nicht ausgelöst oder es wurde nichts gesendet. |**Auslöser / Synchronisation**: M8-Kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Die schlechteste Rate der jeweiligen Kamera. | — |
| `per_cam_rate_pct` | Kombinierte Unvollständigkeitsrate pro Kamera (beide Ursachen zusammen). | — |
| `stable_for_seconds` | Wie lange jede Kamera unter 0,01 % geblieben ist. | — |

Oberhalb von 5 % protokolliert das Backend eine Zeile „`[array-health <id>] WARN`“, in der die Aufteilung benannt wird – beim
ersten Verstoß, bei einem Wechsel des Schweregrads, einmal pro Minute, solange der Zustand anhält, und einmal, wenn
er behoben ist. Die fehlerhafte Hälfte schreibt „`[gvsp-corrupt <SN>]`“ in die Backend-Protokolldatei beim
ersten Treffer pro Kamera und Grund, danach alle 60 Sekunden eine Zusammenfassung; das Protokollfenster der App zeigt stattdessen die
stattdessen die Zeile „`[array-health <id>]`“ an. Jede Auswertung landet weiterhin in der Backend-Protokolldatei;
die Zähler werden bei jedem Puffer aktualisiert, unabhängig davon, was ausgegeben wird.

Derselbe Datensatz gibt die Zahl an, an der die gesamte Zuweisung hängt:

| Feld | Bedeutung |
| --- | --- |
| `wire_ceiling_mbps` | Das aktuell gültige, nachhaltige Bandbreitenbudget des Hosts, MB/s. |
| `wire_ceiling_source` | Woher dieser Wert stammt, in Worten – z. B. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (unter Linux
x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) oder `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, wenn `--wire-ceiling-mbps` (oder das Feld **Wire Budget** der GUI) diesen Wert festlegt. |
| `nic_is_usb` | `true` für einen USB-Ethernet-Adapter – siehe die obige USB-Obergrenze. |

**Deutung:** Ein Wert ungleich Null für `gvsp_corrupt_rate_pct` bei `never_arrived_rate_pct` = 0
bedeutet, dass Triggerung und Kabelsynchronisation einwandfrei sind und 100 % des Verlusts auf dem Netzwerkpfad liegen
– senken Sie `--wire-ceiling-mbps` und stellen Sie die Verbindung erneut her. Das umgekehrte Muster deutet stattdessen auf das
Synchronisationskabel oder die Triggerleitung hin.

> **`--target-fps` ist nicht der Hebel für beschädigte Frames.** Das GevSCPD-Pacing wird
> einmalig bei der Verbindung festgelegt, daher verändert eine Verringerung der Triggerrate den Tastgrad und nicht die
> Burstrate bei simultaner Übertragung. Eine gemessene 5-fache Reduzierung der Anforderung brachte keine Verbesserung;
> die Senkung der Leitungsobergrenze von 240 auf 200 MB/s reduzierte bei demselben Rig den Anteil fehlerhafter Frames von 10,4 %
> auf 0,00 %.

> **Ein laufendes Array kann sich bei der LATTICE-Firmware (M3C / M3M) nicht selbst neu planen.** Es
> kann dieses Problem nicht selbst beheben; trennen Sie die Verbindung und stellen Sie sie erneut her, damit der „Connect-Time-Picker“
> mit der neuen Obergrenze neu planen kann.

### Symptom → Behebung

| Symptom (Array-Einstellungen / Verbindung / `analyze_array_network`) | Ursache | Behebung |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` wird auf 32 zurückgesetzt (typischerweise nach einem Treiber-Update) | Setze `ReceiveBufferLen`→256, `PendingReceives`→64; das Panel erneut öffnen (das Backend neu starten, falls es die alte Ringgröße zwischengespeichert hat) |
| Neustart/Herunterfahren hängen sich auf; später `Invalid class` WMI-Fehler, Netzwerkkarte lässt sich nicht aktivieren, USB-Laufwerke fehlen | Alter Realtek USB-10-GbE-Treiber aus dem Jahr 2016 → BSOD `0x9F` → erzwungene Ausschaltvorgänge | Aktualisieren Sie den Adaptertreiber auf ≥ v10.67 (2026), anschließend die oben genannten Receive-Ring-Einstellungen erneut anwenden |
| Das Array verbindet sich bei voller Sensorauslastung, liefert aber eine niedrige Bildrate | „Auto“ behält die volle Sensorauslastung bei und lässt die Bildrate auf das Niveau sinken, das die Leitung zulässt | Wählen Sie eine der schnelleren Optionen aus der Liste der Verbindungsausgänge (`--binning 2` / `4` oder eine kleinere `--width`/`--height`), aktivieren Sie Jumbo-Frames oder rüsten Sie die Verbindung auf |
| Das Array meldet eine korrekte Ziel-FPS, liefert aber nur einen Bruchteil davon; `health.gvsp_corrupt_rate_pct` ungleich Null, `never_arrived_rate_pct` 0 | Das abgeleitete Bandbreitenbudget des Hosts überschätzt die tatsächlich verfügbare Bandbreite (typisch bei einem USB-Ethernet-Adapter, einer schmalen PCIe-Lane oder einer gemeinsam genutzten Fabric) | Stellen Sie die Verbindung mit einem niedrigeren `--wire-ceiling-mbps`-Wert wieder her und überprüfen Sie den Integritätsblock erneut. **Nicht** `--target-fps` — Die GevSCPD-Ratenregelung wird bei der Verbindung festgelegt |
| Linux
: Ein Array verliert Frame-Gruppen und drosselt seine Bildrate, während die Netzwerkkarte keine Verluste meldet; `lattice network` listet `socket receive buffers too small` auf | Die Socket-Empfangspuffer des Kernels sind auf ihre Standardwerte eingestellt, sodass Kamerapakete im Kernel verworfen werden | Erhöhen Sie `net.core.rmem_max` / `rmem_default` auf 32 MB erhöhen – siehe [Linux
-Hosts](#linux-hosts--socket-receive-buffers). Das .deb-Paket führt dies bei der Installation durch |
| Kameras fehlen in den veröffentlichten Gruppen; `health.never_arrived_rate_pct` ungleich Null, `gvsp_corrupt_rate_pct` 0 | Auslöse-/Synchronisationspfad – die Kameras lösen nicht aus, kein Netzwerkproblem | Überprüfen Sie das M8-Synchronisationskabel und `--line`; stellen Sie sicher, dass alle Mitglieder scharfgeschaltet sind (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` in `analyze-array` überschritten oder Verbindungsablehnung im strengen Modus (`pin_resolution`; `array over-subscribes the wire`) | Die Leitung kann das Array bei keiner der von den Kameras erreichten Übertragungsraten bewältigen, weder gleichzeitig noch nacheinander – es sind weit mehr Kameras, als die Leitung bewältigen kann, oder es werden andere Kameras über dieselbe Leitung übertragen | Weniger Kameras, Jumbo-Frames durchgehend oder eine schnellere Netzwerkkarte. **ROI/Binning hilft NICHT** (die gemeinsame Nutzung ist unabhängig von der Frame-Größe). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` wird auf dem Teststand außer Kraft gesetzt (akzeptiert den Paketverlust) |
| Das Array wird in `slip-emit-and-capture` (seriell) über ein langsames Netzwerk verbunden | Die Kameras können nicht langsam genug getaktet werden, um die Leitung gleichzeitig gemeinsam zu nutzen, daher wechseln sie sich ab; die Frames werden nicht im selben Moment erfasst | Jumbo-Frames oder eine schnellere Netzwerkkarte stellen die gleichzeitige Erfassung wieder her; `--force-tier sim-capture-sim-emit` besteht darauf (und verliert stattdessen Pakete) |

---

## `chloros-cli daq`

Befehle für den Spektralsensor. Die Befehle `pool-*` sind „Thin“-HTTP
-Clients, die den Sensor über den persistenten Pool des Backends steuern; daher benötigen sie ein laufendes Backend (siehe [Backend starten](#start-the-backend)). Das Backend verwaltet den Transport, sodass die CLI
- und SDK
-Skripte sich ein aktives Handle teilen, anstatt um den seriellen Port zu konkurrieren. `pool-discover` ist die einzige Möglichkeit, die BLE-MAC-Adresse eines DAQ-M zu ermitteln.

> **`chloros-cli daq --help`** (oder `-h` oder `daq help`) listet die `pool-*`-Unterbefehle auf (unter Linux
zusammen mit den Befehlen für den direkten Hardwarezugriff, die im Build enthalten sind), und `chloros-cli daq <subcommand> --help` (oder `daq help <subcommand>`) gibt die jeweiligen Optionen an. Diese umfassen das Herstellen einer Verbindung, „stream“, das Aufzeichnen kalibrierter `.daq`-Dateien, das Austauschen von Cap-Profilen und das Binden eines DAQ an eine Kamera sowie für ein DAQ-E dessen Netzwerkeinstellungen, PTP, einen Neustart und Firmware-Updates. Das DAQ kann auch über Python
mit `chloros_sdk.connect_daq_sensor()` gesteuert werden, das denselben gepoolten Pfad verwendet.
>
> **Verwenden Sie die `pool-*`-Befehle auf jeder Plattform.** Sie verhalten sich unter Windows
, Linux
und Jetson gleich. Unter Windows
sind sind `daq discover` und `daq list` Aliase für `pool-discover`, und jeder andere `daq`-Befehl, der nicht `pool-*` ist, wird `1` mit `DAQ direct-hardware commands require the full daq package` beendet. Unter Linux
und auf Jetson sind `daq discover`, `daq list` und die anderen Befehle, die nicht `pool-*` oder `daq` lauten, die direktenHardware-Befehle des DAQ-Tools: Sie öffnen den Sensor in einem eigenen Prozess der CLI
, außerhalb des Pools des Backends (also nicht auf einem Sensor, den das Backend, die GUI oder ein Skript verwaltet), und dort werden `daq discover`, `--only` und `--timeout` akzeptiert, jedoch nicht `--json`. Diese Referenz dokumentiert ausschließlich die Befehle `pool-*`.

### Arbeitsablauf für die erstmalige Verbindung mit einem DAQ-Sensor

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`-Referenz

| Unterbefehl | Zweck |
| --- | --- |
| `daq pool-connect` (smart-detect) | Öffnet einen Sensor im Backend-Pool. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U an einem bestimmten seriellen Port. |
| `daq pool-connect --ble` | DAQ-M über BLE, MAC automatisch gescannt. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M über BLE mit bekannter MAC-Adresse (setzt `--ble` voraus). |
| `daq pool-connect --eth-host HOST` | DAQ-E über Ethernet mit bekanntem Hostnamen oder bekannter IP-Adresse (setzt `--eth` voraus). |
| `daq pool-connect --eth` | DAQ-E über Ethernet, Host automatisch erkannt (mDNS + ARP-Fallback; funktioniert auch bei leerem ARP-Cache unter Windows
und Linux
). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Integrationsfenster / AE-Zustand anpassen. |
| `daq pool-connect --no-stream` | Verbinden, aber noch nicht mit dem Streaming beginnen (Fortsetzung mit `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-Korrekturprofil. Standard im Backend ist `sunshine_cosine`; `none` ist ein Sensor ohne Abdeckung. Ein [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ignoriert diesen (fester Diffusor). **Die Sensoren werden ohne Kappe geliefert, benennen Sie daher die von Ihnen montierte Kappe:** Der Standardwert wird als (`cap_id_source=auto_default`) gespeichert, und die Verarbeitung gibt eine Warnung aus, da ein unbestückter DAQ-U unter Sonneneinstrahlung etwa 30-mal zu hohe Werte anzeigt. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Durchsuche jeden Transport nach Sensoren, mit denen du dich verbinden könntest, ohne eine Verbindung herzustellen. **So finden Sie die BLE-MAC-Adresse eines DAQ-M.** `--timeout` gilt pro Transport (Standard 5 s, 1–20). Sensoren, die bereits im Pool geöffnet sind, werden nicht aufgelistet – ein verbundenes DAQ-M sendet keine Advertisements mehr –, verwenden Sie daher für diese `pool-list`. `usb` listet nur serielle USB-Geräte auf: Die eigenen seriellen Anschlüsse eines Computers (`COM1`, bei Linux
`/dev/ttyS*`) können keine Sensoren sein und werden daher nicht berücksichtigt. |
| `daq pool-list` | Zeigt jeden Sensor im Backend-Pool an. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Bindet eine Kamera oder ein Array an ein DAQ, wie es die Auswahl des Lichtsensors in der GUI tut: Die Aufnahmen verwenden, gleichen ab und speichern die Messwerte dieses DAQ. `SENSOR` ist eine Sensor-ID aus `pool-list` oder die Adresse, unter der das DAQ erreichbar ist. `--clear` (kein `SENSOR`) löst die Verknüpfung auf, und die Erfassungen greifen auf das aktuellste DAQ zurück. Die Bindung besteht so lange, bis das Backend neu gestartet wird oder bis diese Kamera, dieses Array oder dieses DAQ getrennt wird: Eine Trennung (hier vom SDK
oder in der App) löst jede Bindung der getrennten Hardware auf, unabhängig davon, wer sie eingerichtet hat; die Elemente eines Arrays werden mit dem Array selbst getrennt. Die Kamera muss verbunden sein, sich im geöffneten Projekt befinden oder zuvor auf diesem Computer geöffnet worden sein, und das Array muss verbunden sein oder sich im geöffneten Projekt befinden; jede andere Seriennummer oder ID wird aufgrund des Namens abgelehnt und es wird keine Bindung hergestellt. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Freigabe – unabhängig davon, wer den Sensor geöffnet hat; somit wird auch ein Sensor freigegeben, den ein abgestürztes Skript im Pool zurückgelassen hat (etwaige Aufzeichnungen darauf werden zuvor beendet). Ein Sensor, der sich nicht im Pool befindet, hat nichts freizugeben: es gibt „`ID: already released (not in the pool)`“ aus und beendet sich mit „`0`“; eine Freigabe, die das Backend abgelehnt oder nicht bestätigt hat, beendet sich mit „`1`“. SDK
: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Die letzten N Spektrum-Frames (Standard: 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Streaming fortsetzen / anhalten. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Eine .daq-Aufzeichnung starten / stoppen. `-o` ist standardmäßig auf `~/Documents/DAQ Live View` auf dem Backend-Computer eingestellt (ein relativer Pfad befindet sich unter dem Ordner, von dem aus Sie den Befehl ausführen); `--duration` (`-d`, Standardwert `0` = bis `--stop`). `--confirm-after S` gibt eine Warnung aus, wenn S Sekunden nach dem Start noch kein Messwert eingegangen ist (Standardwert 20; 0 deaktiviert diese Funktion); bei `--duration` `--heartbeat S` gibt alle S Sekunden die Messwertanzahl aus und warnt, wenn diese nicht mehr ansteigt (Standard 30; 0 deaktiviert diese Funktion). Eine Aufzeichnung auf einem DAQ-E, das keine Daten mehr liefert, wird in derselben Datei fortgesetzt, sobald das Backend die Verbindung wiederhergestellt hat, und `--stop` listet die Lücken auf ([Ein DAQ-E, das keine Daten mehr liefert](#a-daq-e-that-stops-delivering)). Strg+C beendet eine `--duration`-Aufzeichnung vorzeitig und meldet dies; ohne `--duration` läuft die Aufzeichnung nach Strg+C (Beenden von `130`) weiter, bis `--stop` ausgeführt wird. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Das Cap-Korrekturprofil zur Laufzeit wechseln. `--cap-id` ist erforderlich; `--cap-id none` entfernt die Cap-Korrektur (ein Sensor ohne Kappe). Ein DAQ-E-S behält die Einstellung von `as_recorded` bei, unabhängig davon, was eingestellt ist. |
| `daq pool-enable-ptp --sensor-id ID` | Schaltet PTP für einen DAQ-E wieder ein, dessen PTP ausgeschaltet ist. Die Einstellung wird auf dem Gerät gespeichert, sodass ein ausgeschaltetes Gerät ausgeschaltet bleibt – seine Zeitstempel sind nicht PTP-synchronisiert –, bis dieser Befehl ausgeführt wird. `pool-list` kennzeichnet ein solches Gerät (`!! PTP is switched OFF`) und gibt diesen Befehl dafür aus. Chloros
schaltet ein Gerät niemals von selbst wieder ein, da das Ausschalten beabsichtigt sein kann. |
| `daq pool-info --sensor-id ID [--json]` | Die eigenen Einstellungen eines DAQ-E: seine Firmware (und ob diese Chloros
-Version eine neuere enthält), seine Adresse und wie er sie erhalten hat (DHCP / Link-Local / statisch), die gespeicherte Netzwerkeinstellung, bis der Sensor sie ausführt (eine mit `--no-reboot` gespeicherte), den Stream beim Einschalten und PTP. Bei einem Sensor, dessen Stream diesen Computer nicht erreicht, liest es den Sensor bei einer neuen Verbindung aus und meldet dies. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Einen DAQ-E auf DHCP (`--auto`) oder eine statische Adresse (`--mask`, Standard: `255.255.255.0`). Der Sensor wird neu gestartet, um die Einstellung zu übernehmen, und wird mit den bisherigen Einstellungen wieder verbunden, wobei er bis zu `--timeout` Sekunden (Standard 45) darauf wartet; `--no-reboot` speichert die Einstellung lediglich. Siehe [DAQ-E-Geräteeinstellungen](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Gibt an, ob ein DAQ-E beim Einschalten ohne angeschlossene Software mit dem Streaming beginnt (wird auf dem Sensor gespeichert). Chloros
startet den Stream in jedem Fall selbst. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Ein DAQ-E neu starten und mit den vorherigen Einstellungen wieder verbinden (Wartezeit bis zu `--timeout`, Standard 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Die in dieser Chloros
-Version enthaltene DAQ-E-Firmware auf einen Sensor flashen, auf dem eine ältere Version läuft, und diesen anschließend wieder anschließen (etwa eine Minute; nicht ausschalten; `--timeout` Standardwert 45 s, bis er wieder verfügbar ist). Ein Sensor, der bereits auf dem aktuellen Stand ist, wird nicht berührt. |

### Ein DAQ-E, der keine Daten mehr liefert

Ein DAQ-E im Pool, der 20 Sekunden lang kein Spektrum liefert, wird vom Backend neu verbunden. Dies geschieht durch einen Neustart des Switches, der den Sensor mit Strom versorgt: Der Sensor startet damit neu und überträgt erst wieder Daten, wenn er dazu aufgefordert wird (es sei denn, `pool-set-auto-stream --on` ist gesetzt).

- **Zunächst wird überprüft, ob es sich um denselben Sensor handelt.** Unter der bisherigen Adresse des Sensors werden – ohne Änderungen vorzunehmen – der Name, die MAC-Adresse und die Seriennummer des Sensors abgefragt. Er fragt auch unter dem Namen des Sensors „`daq-e-<id>.local`“ ab, vertraut dieser Antwort jedoch nur, wenn sie vom eigenen Netzwerkadapter (MAC) des Sensors stammt. Alles andere, was antwortet, wird nicht wieder verbunden oder beschrieben.
- **Es stellt die Verbindung mit den Einstellungen der Sitzung wieder her:** Belichtungszeit, Bildmittelung, automatische Belichtung und Belichtungsprofil bleiben unverändert und entsprechen nicht den Standardwerten des Sensors beim Einschalten. Ein Sensor, bei dem PTP aktiviert war und der nun mit deaktiviertem PTP zurückkommt, wird wieder eingeschaltet. Anschließend wird der Stream neu gestartet, ebenso wie der entsprechende Empfänger auf dem Computer.
- **Er versucht es in zunehmend längeren Intervallen erneut:** sofort, dann nach 5, 10, 20 und 40 Sekunden, anschließend alle 60 Sekunden, solange der Sensor nicht erreichbar ist. Bei jedem Versuch wird der Sensor lediglich gefragt, um welchen Sensor es sich handelt. Ein Sensor, der antwortet, aber nach drei Wiederverbindungsversuchen immer noch nichts sendet, wird in Ruhe gelassen und gemeldet (`auto-reconnect GAVE UP`). `pool-stream --start` versucht es erneut.
- **`pool-list` zeigt den aktuellen Stand an.** Solange der Sensor nicht antwortet, gibt die Zeile `STREAM FAULT` Auskunft darüber, was bei der Wiederherstellung der Verbindung geschieht. Die Zeile wird mit dem ersten Spektrum gelöscht, und der Sensor zeigt dann `reconnected automatically N time(s)` an.
- **Die Aufzeichnung wird im selben `.daq` fortgesetzt.** Die Datei markiert jede Lücke mit zwei Zeilen: `event_type` 5 beim letzten Spektrum vor der Lücke und 6 beim ersten danach, jeweils nur mit einem Zeitstempel. `pool-record --duration` gibt „`!! STREAM LOST …`“ aus, während der Sensor nicht vorhanden ist, und „`stream resumed after …`“, wenn er wieder da ist. `pool-record --stop` listet jede Lücke auf und gibt eine Warnung aus, wie es auch bei bei einer leeren Datei (eine Aufzeichnung, die keinen Messwert enthält, wird mit „`1`“ beendet).
- **Eine Aufzeichnung behält eine Kalibrierung bei.** Wenn der Sensor während einer laufenden Aufzeichnung mit einer anderen Kalibrierung zurückkehrt, behält der Rest der Aufzeichnung die ursprüngliche Kalibrierung bei, und die Stoppwarnung weist darauf hin. `pool-disconnect` + `pool-connect` übernimmt die neue Kalibrierung.
- **Ihre Befehle haben Vorrang.** Ein Sensor, den Sie mit `pool-disconnect` aktivieren oder mit `pool-stream --stop` pausieren, wird niemals erneut verbunden, selbst wenn bereits eine erneute Verbindung im Gange ist.
- **`pool-info` bei einem nicht antwortenden Sensor** liest den Sensor bei einer neuen Verbindung aus und fügt `!! stream NOT reaching this computer` hinzu. Ein Sensor, der nicht antwortet, wird sofort gemeldet. `pool-list` zeigt für einen Sensor, der nicht mehr antwortet, keinen PTP-Status mehr an.
- DAQ-U und DAQ-M werden nicht berücksichtigt. Ein DAQ-M stellt die Verbindung über Bluetooth selbstständig wieder her, und ein vom USB-Anschluss getrennter DAQ-U benötigt erneut `pool-connect`.

### DAQ-E-S – fester Diffusor und Neigungssensor

Der **DAQ-E-S** ist ein DAQ-E mit zwei Unterschieden, die für die Software von Bedeutung sind. Alles andere – Erkennung, Verbindung, PTP, Aufzeichnung sowie die unten aufgeführten Geräteeinstellungen – funktioniert genau wie bei einem DAQ-E.**Sein Diffusor ist fest eingebaut, sodass keine Kappenkorrektur erforderlich ist.** Der Diffusor im Sunshine-Stil ist fest integriert und befand sich bereits am Gerät, als dessen Werkskalibrierung gemessen wurde, sodass diese Kalibrierung bereits die tatsächliche nach unten gerichtete Bestrahlungsstärke angibt. Chloros
setzt seine Kappe unabhängig von der Anfrage auf `as_recorded` (kein Profil): Ein speziell dafür benannter Cap (`sunshine_cosine`, `fov_*`, `none`) wird mit einer Meldung ignoriert, und seine Aufzeichnungen speichern `cap_id_source=model`, sodass die Warnung wegen einer angenommenen Obergrenze hierfür nie ausgelöst wird. Wenden Sie auch nicht manuell eine Obergrenze auf die Daten an: „`sunshine_cosine`“ als Obergrenze würde zu etwa elfmal zu hohen Werten führen.

**Es verfügt über einen 3-Achsen-Beschleunigungssensor.** Die Kosinus-Charakteristik des Sonnenstrahlungsdiffusors gilt nur, solange er waagerecht ausgerichtet ist; daher meldet ein geneigter Sensor eine falsche Bestrahlungsstärke – und die Reflektanz übernimmt diesen Fehler –, ohne dass weitere Symptome auftreten. Jeder DAQ-E-S-Messwert enthält daher seine Ausrichtung:

| Feld | Bedeutung |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Winkel in Grad. `None`, wenn die Firmware den Winkel als undefiniert markiert (freier Fall, eine tote Achse). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Beschleunigung in Milli-g. Ein Wert von `mag_mg`, der weit von ~1000 abweicht, bedeutet, dass sich das Gerät bewegte und die Winkel für diese Messung keine Aussagekraft haben. |
| `cal_applied` | „Set“: Die Achsen und Winkel liegen im Koordinatensystem des Diffusors (die Kalibrierung der Gerätebefestigung wird angewendet). „Clear“: Rohdaten im Platinenkoordinatensystem, die eine Neigung von bis zu ~2,3° anzeigen können, die nicht real ist. |
| `sample_age_ms` | Alter der Beschleunigungsmesser-Probe zum Zeitpunkt der Spektrumübertragung. |
| `temp_raw` | Unkalibrierter **Trend** der Chip-Temperatur – nur Differenzen, niemals ein Thermometerwert. |

`daq pool-latest --json` gibt diese unter dem Schlüssel `imu` jedes Frames zurück (`null` bei einem DAQ-U, DAQ-M oder einfachem DAQ-E). Ein von einem DAQ-E-S aufgezeichnetes `.daq` speichert dieselben Werte in seinen `imu_*`-Spalten (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) neben jedem Spektrum, sodass der Leser eine Wolke von einem umgestoßenen Stativ unterscheiden kann. Bei allen anderen Modellen sind diese Spalten leer.

Der Lage-Datenstrom wird separat gemeldet: `daq pool-list` gibt `!! ATTITUDE STREAM FAULT (spectra unaffected)` aus, während es ausgefallen ist, sowie die Anzahl der Fehler, von denen es sich anschließend erholt hat. Ein Fehler an dieser Stelle führt niemals zu einer Lücke in der Spektralaufzeichnung. `daq pool-discover` listet das Gerät als `DAQ-E-S` auf (als `DAQ-E`, wenn es nicht innerhalb von etwa 2 s auf die Identitätsprüfung antwortet).

### DAQ-E-Geräteeinstellungen

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` ohne `--gateway` speichert kein Gateway** (alle zuvor eingestellten Gateways werden verworfen), was für ein Kameranetzwerk ohne Router richtig ist.
- **Eine statische Adresse muss im Netzwerk dieses Computers vorhanden sein.** Chloros
lehnt eine Adresse ab, deren Subnetz die Adresse dieses Computers im Netzwerk des Sensors nicht enthält: Nach dem Neustart könnte der Sensor nicht mehr erreicht werden, um die Einstellung zurückzusetzen. Übergeben Sie `--force` nur, wenn dieser Computer auf das neue Subnetz umadressiert oder dorthin weitergeleitet wird.
- **Ein Neustart wird abgelehnt, solange der Sensor aufzeichnet.** Beenden Sie zuerst die Aufzeichnung (`pool-record --stop`). Eine Aufzeichnung, die unmittelbar vor einem Neustart gestartet wurde, wird durch diesen beendet, und `pool-set-network`, `pool-reboot` sowie `pool-update-firmware` enden mit einer Zeile `Warning:`, in der der Dateiname der Aufzeichnung angegeben ist (auch wenn der Sensor danach nicht zurückkommt).
- **Nach einem Neustart verbindet Chloros
nur den Sensor selbst wieder.** Es wartet dort, wo sich der Sensor befand – und nach „`--auto`“ auch unter dessen Namen „`daq-e-<id>.local`“ – auf eine Antwort mit dem Namen des Sensors, Hostnamen und Modell, von dem Netzwerkadapter (MAC), den er hatte, und stellt die Verbindung an der Adresse wieder her, die geantwortet hat. Eine Antwort unter diesem Namen wird nur mit dieser MAC-Adresse akzeptiert; daher wird von einem Computer, der den Sensor über einen Router erreicht, oder von WSL2 in dessen Standard-NAT-Netzwerk ein Sensor, den DHCP auf eine neue Adresse verschoben hat, nicht wieder verbunden: Der Befehl gibt dies an und gibt den Befehl „`pool-connect --eth-host <its address> …`“ aus, der den Sensor mit den bisherigen Belichtungs- und Blendenwerten wiederherstellt.
- **`pool-update-firmware` lehnt einen Sensor ab, dessen Firmware-Version nicht gelesen werden kann.** Ein Sensor, dessen Rückmeldung nach dem Blitz länger dauert als die Wartezeit, ist nicht ausgefallen: Sobald er antwortet, wird die Verbindung mit den Einstellungen der neuen Firmware wiederhergestellt, und der Befehl warnt, dass die Antwort verzögert war.
- **Für ein Firmware-Update ist eine eingehende TCP-Verbindung erforderlich.** Der Sensor stellt erneut eine Verbindung zu diesem Computer her, um das Bild abzurufen. Chloros
überwacht den Adapter, der zum Sensor weiterleitet (auf jedem Adapter, sofern nicht erkennbar ist, welcher es ist), und bedient ausschließlich die eigene Adresse des Sensors; jede andere Verbindung wird ohne Bearbeitung geschlossen. Unter Windows
lässt die Firewall-Regel „`Chloros Backend (TCP-In)`“ des Installationsprogramms den Datenverkehr zu (siehe [Windows
(.exe)](#windows-exe)); eine Firewall, die den Datenverkehr blockiert, führt zu einem Fehlschlag des Updates mit dem Fehlercode `… device never connected back on TCP <port> …`.
- **Verwenden Sie unter WSL2 „Mirrored Networking“** (`networkingMode=mirrored` in `%UserProfile%\.wslconfig`). Im Standard-NAT-Modus ist die Adresse dieses Computers im Netzwerk des Sensors die von WSL selbst, sodass jede statische Adresse abgelehnt wird (abgesehen von `--force`), der Live-Stream WSL nicht erreicht und ein Firmware-Update nicht abgeschlossen werden kann, da der Sensor eine Verbindung zum Computer herstellt, um das Image zu empfangen.
- **Die Sensor-ID und der Hostname ändern sich nicht.** Unabhängig von seiner Adresse ist ein DAQ-E weiterhin `daq-e-<id>` und `daq-e-<id>.local`, sodass `--eth-host daq-e-<id>.local` ihn weiterhin findet.
- **Die GUI verfügt über dieselben Einstellungen**: Registerkarte „Lichtsensoren“, Sensoreinstellungen (Zahnrad), Netzwerk. Die GUI unterhält ihre eigene Verbindung zu einem Sensor; wenn Sie also einen Sensor ändern, zu dem die GUI eine offene Verbindung hat, über die GUI: Ein Neustart über die CLI
beendet die Sitzung der GUI mit diesem Sensor.
- Die entsprechenden SDK
-Bezeichnungen lauten `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` und `update_firmware()`.

---

## `chloros-cli project`

Öffnen, verbinden und steuern Sie ein gespeichertes Chloros
-Projekt (ein Ordner mit `cameras.json` + `sensors.json` + `project.json`) öffnen, eine Verbindung herstellen und ausführen. Alles läuft über das Backend, sodass die GUI und die CLI
denselben Hardware-Zustand erzeugen.

### Referenz der Unterbefehle

| Unterbefehl | Zweck |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Das Gerätemanifest des Projekts (Kameras, Arrays, Sensoren) ausgeben, aus den zugehörigen Dateien lesen und die Quelldateien überprüfen (siehe [Ein verschobenes Projekt](#a-moved-project---relink)). Das Projekt wird dabei nicht im Backend geöffnet: Das dort geöffnete Projekt sowie dessen Geräte und Aufzeichnungen bleiben unverändert. Beendet `1`, wenn keine der Quelldateien des Projekts gefunden werden kann. |
| `project devices PATH [--reconnect]` | Erkennung auflisten oder erneut ausführen. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Alle gespeicherten Kameras / Arrays / Sensoren verbinden. Jedes Array wird wie bei `lattice array-connect` verbunden: bis zu 600 s, wobei die einzelnen Schritte auf stderr ausgegeben werden (`[array <id>]    <s> s: <step>`; stdout ist der JSON
-Bericht), und ein Array, das innerhalb dieser Zeit nicht verbunden wurde, wird rückgängig gemacht – seine Kameras werden in den ursprünglichen Zustand zurückversetzt – und unter `errors` gemeldet; es bleibt niemals verbunden. `--packed` legt den gepackten 12-Bit-Transport für jedes Array fest (Standard: `auto`; ein Projekt speichert keine array-spezifischen Einstellungen; siehe [Gepackter 12-Bit-Transport](#packed-12-bit-transport---packed)). Jeder Lichtsensor wird im DAQ-Pool des Backends an seiner gespeicherten Adresse mit seiner gespeicherten Integrationszeit, der Bildmittelung, der automatischen Belichtung und der Begrenzung geöffnet: Ein Sensor, für den kein Cap ausgewählt wurde, erhält `sunshine_cosine`, ein als „bare“ gespeicherter Sensor (`none`) bleibt im „bare“-Modus, und ein DAQ-E-S nimmt niemals einen Cap an. Das DAQ-Fenster der GUI ist nicht Teil des Pools: Schließen Sie einen Sensor dort zuerst. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Einzelaufnahme von einer benannten Kamera oder einem Array (`--prefix` Standard, `capture`). `--orient` (Arrays): Das Array speichert Rohdaten, die im nativen Format bleiben – dies bedeutet, dass die daraus von `process` erstellten Produkte wie `array-capture --orient` ausgerichtet werden müssen. Standard: nativ. `project burst` / `stream` sowie die `capture` / `burst` / `stream` (`orient: true`) verwenden dieselbe Option. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | N-Frame-Burst von einer benannten Kamera oder einem Array (`-n/--count` Standardwert 5; `-i/--interval` Sekunden zwischen den Frames, Standardwert 0). Array-Serien entfernen Duplikate wiederholter synchronisierter Gruppen (Verfalls-Watchdog), sodass ein Array mit teilweiser Wiederholung nicht N Kopien eines Einzelbilds zurückgeben kann; gibt die Ergebnisse pro Iteration aus. Eine Erfassung, die vorzeitig fehlschlägt, stoppt die Burst-Aufnahme und beendet `1` mit der Angabe, wie viele Frames zuvor gespeichert wurden. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Erfassung auf Festplatte in einer Schleife: `--fps F` erfasst eine Sekunde (Standard 3), ein Bild pro Erfassung von einer Kamera, eine synchronisierte Bildgruppe aus einem Array (mit demselben Verfalls-Watchdog wie `project burst`), bis `-n N` (Standard: `0` = bis zum Drücken von Strg+C, woraufhin die Anzahl der gespeicherten Bilder ausgegeben wird und `0` beendet wird, oder `1`, wenn kein Bild gespeichert wurde oder ein Dateifehler auftrat; ein „Strg+C“ vor der Erfassung durch `-n N` beendet `130`). Eine Erfassung, die vorzeitig fehlschlägt, stoppt den Stream und beendet `1`, wobei angegeben wird, wie viele Frames zuvor gespeichert wurden. `--poll-interval` ist veraltet und wird ignoriert: Es wird weiterhin akzeptiert, damit ältere Skripte ausgeführt werden können, wobei ein Hinweis auf stderr ausgegeben wird. |
| `project sensor read PATH NAME [--json]` | Aktueller Spektrum-Frame. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Aufzeichnung einer .daq-Datei für `--seconds` (Standardwert 10). Eine Aufzeichnung ohne Messwerte wird auf stderr gemeldet und beendet `1`. |
| `project run PATH RECIPE.yaml` | Ein YAML-/JSON
-Erfassungsrezept ausführen. `--dry-run` führt eine Validierung ohne Ausführung durch. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Berechnet die Ausrichtung für ein Array – siehe [die untenstehende Flag-Tabelle](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Aktuelles Alignment-Profil ausgeben. |
| `project align clear PATH NAME` | Das zwischengespeicherte Profil löschen. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Die Transformation eines Slaves verschieben (Standardwerte: `0`, `0`, `0`, `1.0`: keine Änderung). |
| `project align export PATH NAME --to FILE` | Das Profil als JSON
speichern. |
| `project align import PATH NAME --from FILE [--no-validate]` | Ein gespeichertes Profil laden. |

#### `project align calibrate`-Optionen

| Flag | Standard | Beschreibung |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Ausrichtungsmethode. **Diese Schreibweisen unterscheiden sich von `lattice align-calibrate`**, das die Kurzformen `orb` / `akaze` / `phase` verwendet; die beiden Befehle sind bei diesem Flag nicht austauschbar. |
| `--model {translation, rigid, affine, homography}` | `affine` | Modell so transformieren, dass es passt. |
| `--frames N` | `1` | Synchronisierte Bildausschnitte auf Mittelwert. |
| `--reference SN` | dem Master | Seriennummer der Referenzkamera; jedes andere Mitglied wird darauf abgebildet. |
| `--max-features N` | `5000` | Obergrenze für die Anzahl der ORB-Merkmale. |
| `--ratio-threshold F` | `0.75` | Lowe&#x27;Verhältnis-Test. |
| `--ransac-threshold-px F` | `3.0` | RANSAC-Schwellenwert für Inlier. |
| `--min-matches N` | `15` | **Qualitätsschwelle** – Lösung wird abgelehnt, wenn die Anzahl der Inlier-Übereinstimmungen unter diesem Wert liegt. |
| `--max-reproj-err-px F` | `4.0` | **Qualitätsschwelle** – die Lösung wird verworfen, wenn der RMS-Fehler bei der Reprojektion diesen Wert überschreitet. |
| `--checkerboard RxC` | — | Platinengeometrie für `--method checkerboard`, z. B. `9x6`. |
| `--name PROFILE` | leer | Im gespeicherten JSON
eingebetteter Profilname. **Nicht der Array-Name** — das ist das Positions-`NAME`. |

Die beiden Qualitätsprüfungen sind der Grund, warum eine Kalibrierung zwar erfolgreich durchgeführt werden kann, das Speichern jedoch
verweigert wird: Ein Profil, das eine der beiden Prüfungen nicht besteht, würde jede
spätere Aufnahme stillschweigend falsch registrieren, daher wird es abgelehnt, anstatt gespeichert zu werden.

### Ein verschobenes Projekt (`--relink`)

Ein Projekt mit der Bezeichnung `project.json` benennt jede Quelldatei – jedes Bild und jede Lichtsensor-
Aufzeichnung – anhand ihres vollständigen Pfads. Wenn Sie den Projektordner oder den gesamten Projektordner
auf ein anderes Laufwerk verschieben, verweisen diese Pfade weiterhin auf den alten Speicherort.

Beim Öffnen eines Projekts wird selbstständig korrigiert, was möglich ist: Eine Datei, die sich zuvor im Projektordner
befand, wird in dem Ordner gesucht, in dem sich das Projekt nun befindet, und jeder dort gefundene Pfad
wird neu verweist und gespeichert (`project.json` wird zunächst als
`project.json.before-relink-<date>-<time>` gesichert). Dies geschieht bei jedem Öffnen – sei es der App,
`chloros-cli project open`, `chloros-cli process` in ein bestehendes Projekt, die des SDK
s
und die von `chloros-mcp`. Ein Pfad wird immer nur auf eine Datei umgeleitet, die dort vorhanden ist.

Dateien, die sich außerhalb des Projektordners befanden – beispielsweise bei einer Hub-Sitzung oder einem Import über „Ordner hinzufügen“ –
können auf diese Weise nicht gefunden werden. Geben Sie den Namen des Ordners an, in dem sie sich jetzt befinden:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` nimmt den Ordner, in den die Dateien verschoben wurden, oder einen beliebigen darüber liegenden Ordner: Der Ordner jeder fehlenden
Datei wird darunter anhand der an den Dateinamen angehängten Ordnernamen gesucht
(`Hub Projects/<session>/raw/<serial>`, dann `<session>/raw/<serial>`, …), und die
Ordner werden die Dateien von dem Ort übernommen, an dem die meisten davon gefunden werden. `project open` gibt aus,
was neu zugeordnet wurde, wie viele Quelldateien gefunden wurden und in welchen Ordnern sich die restlichen befanden
. Einige fehlen noch: eine Warnung, Beenden von `0` – das Projekt wird mit den übrigen Dateien geöffnet. Keine
gefunden: eine Fehlermeldung, in der diese Ordner und `--relink` genannt werden, Beenden von `1`; das Backend weigert sich ebenfalls
weigert sich, ein solches Projekt für `process` oder `project devices --reconnect` zu öffnen (HTTP
-Fehler 409 `PROJECT_SOURCES_MISSING`).

### Beispiele

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Rezept-DSL

`project run RECIPE.yaml` akzeptiert eine YAML- oder JSON-Datei, die eine Abfolge von Aktionen beschreibt:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Unterstützte Aktionen: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` übernimmt die Schlüssel `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (die älteren Namen `exposure_time`, `exposure_auto`, `gain`, `gain_auto` werden als Aliase akzeptiert). Eine auf diese Weise festgelegte Belichtung, Verstärkung oder Zielwert wird als Ihre Einstellung beibehalten: Die automatische Belichtungssteuerung von Chloros (einschließlich der durch das DAQ gemessenen Belichtung) lässt diese Einstellung unverändert. Bei einer manuellen Belichtung bleibt die Verstärkung unverändert: Im Automatikmodus passt die Kamera die Verstärkung (über den gesamten Bereich) so lange an die Zielhelligkeit an, bis Sie `gain_db` oder `auto_gain: "Off"` einstellen. Ein späteres `auto_exposure: "Continuous"` übergibt die Belichtung wieder an die Automatik, und `target_brightness_auto: true` übergibt einen festgehaltenen Zielwert wieder an die automatische Zielwertsteuerung vChloros. Die Aktion `burst` benötigt `name` (erforderlich), `count` (Standardwert 5), `interval` (Sekunden, Standardwert 0), `output`, `format`, sowie `settings` (gleiche Form der kameraspezifischen Einstellungen wie bei `apply`); Array-Bursts verwenden denselben Watchdog für frisch synchronisierte Gruppen wie `project burst`.

Ein `capture`, `burst` oder `stream` wird der Standardwert von `output` auf `output` gesetzt. Ein relativer `output` – der Standardwert ist enthalten – wird anhand des Ordners aufgelöst, aus dem Sie `chloros-cli project run` ausführen, nicht anhand des Ordners der Rezeptdatei oder des Projektordners (in 1.2.0 landete er im Installationsordner des Backends).

Führen Sie es aus:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Jeder Lauf wird zunächst anhand des Projekts überprüft.**Jede Aktion muss genau eines der oben genannten Verben als Schlüssel haben – `- capture: {name: FrontLeft}`, nicht `- action: capture` oder `- type: capture` — bei einem `name` verfügt das Projekt über `cameras.json` / `sensors.json`, eine Kamera oder ein Array für `apply` / `capture` / `burst` / `stream` sowie einen Lichtsensor für `sensor`; ein `stream` benötigt `count` &gt; 0 und ein `wait` eine Dauer. Ein Rezept mit einem solchen Fehler führt**keine** Aktion aus (`[recipe] error: …`, `[recipe] not run: …`, Beenden von `1`), selbst bei Vorhandensein von `stop_on_error: false`. Schlüssel, die ein Schritt nicht verwendet, sind ein `[recipe] warning:`, ebenso wie jedes Array oder jeder Lichtsensor, den das Rezept ansteuert und der derzeit nicht angeschlossen ist. Ein Rezept verbindet niemals Geräte: Führen Sie zuerst `chloros-cli project connect PATH` aus. `--dry-run` gibt das Rezept aus, dann die gleichen Fehler und Warnungen, und `[recipe] valid: …` / `NOT valid: …`, führt nichts aus und beendet `1`, wenn das Rezept einen Fehler aufweist.

---

## Umgebungsvariablen

| Variable | Wirkung |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Die Backend-URL überschreiben (Standard: `http://127.0.0.1:5000`) — **wird nur von den Befehlsfamilien `lattice`, `project` und `daq pool-*` berücksichtigt** (`lattice network-analysis` verwendet stattdessen `--backend-url`). Die Kernbefehle (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) legen `http://127.0.0.1:<port>` fest und ignorieren diese Variable (das IPv4-Literal umgeht die bei Windows-Anfragen auftretende Verzögerung von etwa 2 Sekunden pro Anfrage für `localhost`→`::1`), sodass sie immer auf den lokalen Rechner abzielen. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Wird in der Umgebung des Backends festgelegt. `1` stuft die Ablehnung bei Überbelegung herab – eine Array-Verbindung mit `pin_resolution` oder ein [Befehl für vorübergehende Mehrkamera-Sitzungen](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) – auf eine deutliche Warnung herab, wobei GVSP-Paketverluste akzeptiert werden. Nur zu Testzwecken – siehe [Array-FPS- und Burst-Modell](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Wird in der Umgebung des Backends festgelegt: `on` oder `off` legen den Standardwert [gepackter 12-Bit-Transport](#packed-12-bit-transport---packed) für jedes Array fest (Standard: `auto`). Eine explizite Angabe von `--packed` hat weiterhin Vorrang. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (oder `true` / `yes`) deaktiviert die [automatische Update-Prüfung](#chloros-cli-update) der Linux-CLI beim Start. |
| `CHLOROS_CLI_MODE` | Wird von der CLI selbst gesetzt; weist das Backend an, die parallele Verarbeitung zu aktivieren. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` überspringt die GVSP-Fallback-Prüfung (nur ICMP-Ergebnisse). **Dadurch wird Jumbo deaktiviert, es werden nicht nur die Log-Einträge unterdrückt** – die Kamera beantwortet DF-Pings auf jedem Pfad nur bis zu 1500, daher ist dieser Test der einzige Weg, Jumbo zu erkennen. Spart ca. 1 s pro Kamera und Verbindung; kostet ca. das 1,45-Fache der Leitungskapazität, wenn das Netzwerk Jumbo-Pakete hätte übertragen können. Das SDK gibt eine Warnung aus, wenn Sie diese Option aktivieren. |
| `CHLOROS_GVSP_SESSION_REUSE` | In der Umgebung des Backends festlegen: `0` misst die Paketgröße bei jeder Verbindung statt einmal pro Kamera und Backend-Sitzung. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixiert die GVSP-Paketgröße auf N Bytes; überspringt die Erkundung vollständig. Bevorzugen Sie die Einstellung pro Befehl (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) gegenüber einer dauerhaften Festlegung: Eine festgelegte Größe passt sich nicht mehr an das vorgelagerte Netzwerk an, und die Festlegung von 9000 auf einem Pfad, der keine Jumbo-Pakete übertragen kann, führt dazu, dass **jede** Erfassung mit `SC_ERR_TIMEOUT -1011` in eine Zeitüberschreitung mündet. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Wird in der Umgebung des Backends festgelegt: die maximale Anzahl an Sekunden, die ein [vorübergehender Mehrkamera-Befehl](#multi-camera-transient-sessions) oder ein `lattice capture` einer Kamera, die nichts hält, für die Einstellung der Belichtung jeder Kamera aus der Szene benötigt, bevor sie ein Bild speichert (Standardwert `5`). `0` deaktiviert diese Funktion: Die automatische Belichtung jeder Kamera läuft dann ab dem ersten Bild, beginnend bei ihrem Belichtungsgrenzwert. |
| `CHLOROS_LOG_DIR` | Wird in der Umgebung des Backends festgelegt: der Ordner, in den das detaillierte Protokoll geschrieben wird. Standardmäßig `%LOCALAPPDATA%\Chloros\logs` unter Windows, `~/.cache/chloros/logs` unter Linux (`$XDG_CACHE_HOME/chloros/logs`, wenn dies festgelegt ist). Der Root-Dienst des .deb-Pakets legt `/var/log/chloros` fest, auf das die Gruppe `adm` Lesezugriff hat – siehe [Ausführen des Backends als Dienst](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Speicherort für temporäre Dateien. Die CLI verwendet automatisch `/mnt/ssd/tmp`, falls vorhanden. |

---

## Exit-Codes

| Code | Bedeutung |
| --- | --- |
| `0` | Erfolg. |
| `1` | Allgemeiner Fehler (die meisten Unterbefehlsfehler). Ein Aufzeichnungsbefehl beendet sich außerdem mit dem Code `1`, wenn das Schreiben einer der Dateien, die geschrieben werden sollten, fehlgeschlagen ist, auch wenn die anderen gespeichert wurden, sowie mit den Codes `array-burst` / `array-record` und `1` beendet werden, wenn der Recorder einen Fehler meldet – siehe [Dateien, die nicht geschrieben werden können](#files-that-fail-to-write). `process` beendet den Vorgang ebenfalls, wenn die Exporte eines Laufs nicht überprüft werden konnten oder wenn ein Bild verloren ging, obwohl die anderen Produkte geschrieben wurden. |
| `2` | Argumentfehler (ein unbekanntes oder fehlendes Flag; ebenso `time-sync set-priority` ohne Angabe der Priorität). |
| `130` | Durch Strg+C unterbrochen: jeder Befehl, der dadurch abgebrochen wird – `lattice`, `project`, `daq` und `process` gleichermaßen (`process`, nachdem das Backend die Ausführung beendet hat, siehe [`process`-Hinweise](#notes)). Ein Befehl, dessen normale Beendigung mit Strg+C erfolgt – `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` und `project stream` mit `-n 0` – beendet den Vorgang stattdessen wie folgt: `0`, wenn alles geschrieben wurde, `1`, wenn etwas fehlgeschlagen ist. |
| `143` / `129` | Durch SIGTERM / SIGHUP (128 + das Signal) beendet, nachdem die Aufräumphase des Befehls abgeschlossen war: Ein laufender Job wurde gestoppt und seine Kameras zurückgesetzt, ein Verarbeitungslauf im Backend wurde beendet. Eine `lattice continuous` oder eine andere Aufzeichnung, deren normales Beenden mit Strg+C erfolgt, beendet ebenfalls `143`, nicht `0`. |

`lattice network-analysis` beendet `1` bei den Urteilen `needs_force_slip` und `error`, und `selftest` beendet `1`, sofern nicht alle sieben Prüfungen bestanden werden.

---

## Hinweise zur Fehlerbehebung

- **„Anmeldung erforderlich“** → Führen Sie `chloros-cli login EMAIL PASSWORD` einmal auf diesem Rechner aus.
- **„Der Projektordner ist für diesen Computer zu tief“** → Lange Windows-Pfade sind auf diesem Rechner deaktiviert (Windows-Standard), daher darf kein Pfad länger als 260 Zeichen sein, und Produkte befinden sich bis zu etwa 130 Zeichen unterhalb des Projektordners. Es wurde nichts verarbeitet. Die Meldung gibt den längsten zulässigen Projektordner an: Verwenden Sie einen kürzeren (`-o` oder `chloros-cli set-project-folder`) oder aktivieren Sie lange Windows-Pfade und starten Sie Chloros neu. Der Windows-Installer bietet dies als Option „Windows-Langpfade aktivieren“ an, die standardmäßig aktiviert ist; andernfalls verwenden Sie die Gruppenrichtlinie: *Computerkonfiguration &gt; Administrative Vorlagen &gt; System &gt; Dateisystem &gt; Win32-Langpfade aktivieren*.
- **„Backend nicht erreichbar“** → Starten Sie die Chloros-Desktop-App (Windows) oder starten Sie das Backend selbst: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` unter Windows (im Ordner `backend` Ihres Installationsordners; es befindet sich nicht in `PATH`), `chloros-backend` unter Linux. Oder überprüfen Sie `CHLOROS_BACKEND_URL`, falls es sich um einen Remote-Zugriff handelt.
- **`lattice`-Befehle schlagen mit der Meldung „LATTICE-Kameratreiber nicht gefunden“ fehl** → Das Backend konnte die LATTICE-Kamera-Laufzeitumgebung, mit der es installiert wurde, nicht laden. Jeder Befehl „`lattice`“ wird auf dem Backend ausgeführt; die CLI-Anwendung selbst enthält keine Kameratreiber. Installieren Sie Chloros neu: Der Windows-Installer und die .deb-Datei installieren die Laufzeitumgebung neben dem Backend.
- **„Array connect“ / „Array Settings“ zeigt „FRAMES WILL DROP“ oder „Reduce ROI to enable“ an** → Der Empfangsring der Host-Netzwerkkarte ist zu klein (wird nach einem Update des Netzwerkkartentreibers üblicherweise auf 32 zurückgesetzt). Siehe [Einrichtung und Optimierung der Host-Netzwerkkarte](#host-nic-setup--tuning-lattice-arrays) – `ReceiveBufferLen=256` und `PendingReceives=64` einstellen.
- **Der Rechner hängt sich beim Neustart/Herunterfahren auf, anschließend WMI `Invalid class` / Netzwerkkarte lässt sich nicht aktivieren / USB-Laufwerke fehlen** → Veralteter Treiber für den USB-10-GbE-Adapter verursacht `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aktualisieren Sie den Adaptertreiber – siehe [Einrichtung und Optimierung der Host-Netzwerkkarte](#host-nic-setup--tuning-lattice-arrays).
- **Linux: Die Aufzeichnungen gehören dem Benutzer „root“, daher können Sie sie nicht löschen, und das Protokoll des Backends befindet sich nicht in Ihrem Verzeichnis `~/.cache`** → Das Backend läuft als „root“ über `chloros-backend.service`. Wechseln Sie zu `chloros-backend@<user>.service`, wo es als Sie ausgeführt wird – siehe [Ausführen des Backends als Dienst](#running-the-backend-as-a-service-headless). Bis dahin befindet sich das Protokoll des Root-Dienstes in `/var/log/chloros` (lesbar für die Gruppe `adm`).
- **Linux: Eine Backend-Einheit ist `failed` mit der Meldung „Eine andere Instanz wird bereits ausgeführt“ (Exit-Status 98)** → Ein anderes Backend belegt den Port bereits: Es kann jeweils nur eines gleichzeitig laufen. Wenn zwei Backend-Einheiten aktiviert sind, behalte eine bei: `sudo systemctl disable --now <unit>` für diejenige, die du nicht verwendest – siehe [Ausführen des Backends als Dienst](#running-the-backend-as-a-service-headless). Wenn nur diese Einheit aktiviert ist, wird der Port von einem außerhalb von systemd gestarteten Backend (durch `chloros-backend`, `chloros-cli` oder ein SDK-Skript) belegt: Beenden Sie dieses Backend und führen Sie anschließend `sudo systemctl start <unit>` aus.
- **Jetson-Swap-Warnung** → Fügen Sie einen dateibasierten Swap hinzu; die CLI gibt die genauen Befehle `fallocate` / `swapon` aus.
- **„Backend nicht erreichbar“ bei einem Befehl `lattice`, `project` oder `daq`** → Es läuft kein Backend: Diese Befehle starten niemals eines. Führen Sie den Befehl `chloros-cli backend status` aus und lesen Sie den Abschnitt [Backend starten](#start-the-backend).
- **„`daq net ...` ist in Chloros nicht verfügbar“** (Linux und Jetson) → Einige der eigenen Einstellungen eines DAQ-E sind für den MAPIR-Dienst reserviert. Netzwerkeinstellungen, Neustarts und Firmware-Updates erfolgen über `pool-set-network`, `pool-reboot` und `pool-update-firmware`; wenden Sie sich für alles Weitere an den MAPIR-Support.
- **„Direkte Hardwarebefehle für DAQ erfordern das vollständige daq-Paket“** (Windows) → Verwenden Sie `pool-*`, das denselben Sensor über das Backend ansteuert, oder `chloros_sdk.connect_daq_sensor()` aus Python.

---

## Siehe auch

- [Python-SDK-Referenz](sdk-reference.md) – programmatisches Äquivalent zu jedem CLI-Befehl.
- [DAQ-Lichtsensoren](https://mapir.gitbook.io/chloros/daq-light-sensors) im Chloros-Handbuch – die Sensoren, ihre Nennwerte und kalibrierten Messbereiche, Aufzeichnung, DAQ-E-Vernetzung.
- [LATTICE-Kameras](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) im Chloros-Handbuch – Anschluss, Kameraeinstellungen, Aufnahmemodi, Arrays.
- Online-Dokumentation: `https://mapir.gitbook.io/chloros/cli`
