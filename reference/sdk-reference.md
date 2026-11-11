# Chloros-Python-SDK-Referenz

**Version:**

1.3.0**Erstellt:**

29.07.2026, 19:19 Uhr ·**Überarbeitet:**

06.11.2026**Paket:** `chloros-sdk` (PyPI)**Zielgruppe:** Optimiert für die Nutzung durch große Sprachmodelle (LLM); für Menschen lesbar.**Umfang:** Alle öffentlichen Klassen, Funktionen und Hilfsfunktionen, die von `import chloros_sdk` bereitgestellt werden, mit Beispielen zum Kopieren und Einfügen, die Bildverarbeitung, die Steuerung einzelner Kameras, synchronisierte Arrays, DAQ-Sensoren und Projektautomatisierung sowie den `chloros-mcp`-Server für KI-Assistenten.

Wenn Sie nur die wichtigsten Informationen benötigen, springen Sie zu:
- [Installation & Schnellstart](#installation)
- [Smart-Connect für LATTICE-Arrays](#smart-connect-for-lattice-cameras)
- [DAQ-Sensor-Sitzungen](#daq-sensor-sessions)
- [Kamera-Dienstprogramme](#camera-utilities)
- [Transient-Arrays, Ausrichtungsprofile & Indizes](#transient-arrays-alignment-profiles--indices)
- [Projektordner & Rezepte](#project-folder--recipes)
- [Projektautomatisierung](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Live-Bilder in Ihrer eigenen Pipeline](#live-frames-in-your-own-pipeline)
- [MCP-Server (KI-Assistenten)](#mcp-server-ai-assistants)

> **Änderungen in 1.3.0** (was ein bestehendes 1.2.0-Skript bemerkt).
>
> **Änderungen, die ein 1.2.0-Skript möglicherweise berücksichtigen muss:**
>
> - **Ein Release, das vom Backend nicht bestätigt wird, löst nun einen Fehler aus.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` bei einem Array, das die Sitzung geöffnet hat, `DAQSensorSession.disconnect()` / `force_disconnect()` bei einem von ihr geöffneten Sensor sowie `ChlorosProject.disconnect_all()` lösen `ChlorosConnectError` aus, wenn das Backend die Freigabe ablehnt oder nicht rechtzeitig bestätigt, da das Gerät möglicherweise noch belegt ist. In 1.2.0 wurden sie stillschweigend zurückgegeben. Der Austritt aus einem `with`-Block löst nach wie vor niemals einen Fehler aus.
> - **Ein ausgeliehenes Gerät wird niemals freigegeben.** `with attach_array(...)` und ein expliziter Aufruf von `disconnect()` darauf lassen das Array weiterlaufen (in 1.2.0 führte das Verlassen des Blocks zur Freigabe, selbst wenn es sich im Besitz der GUI befand); rufen Sie `force_disconnect()` auf, wenn Sie den Vorgang beenden möchten. Ebenso lässt ein `connect_daq_sensor()`, der den Sensor bereits im Pool vorfand – einschließlich der Smart-Detect-Funktion mit Null-Flag –, diesen auf `disconnect()` und beim Verlassen seines `with`-Blocks offen.
> - **PNG und JPEG akzeptieren nur die Produkte `display` und Index-Produkte.** `capture()` mit `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) sowie auf der Ebene von `radiance` oder `reflectance`, oder `processing="all"`, löst vor jeder Anfrage `ValueError` aus; ein 12-Bit-`raw` / `debayered` wird vom Backend abgelehnt. Siehe [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` öffnet diesen Sensor oder löst** `ChlorosConnectError` aus, wobei die gefundenen Ergebnisse aufgelistet werden; es wird niemals auf einen anderen Sensor zurückgegriffen. Siehe [Pinned Transport](#pinned-transport).
> - **`frame_stream()` löst einen Fehler aus, anstatt still zu bleiben:** `FrameFetchError` würde bei einer Ablehnung im nächsten Tick (einen Reflexionsmesswert, der überhaupt keine DAQ-Daten enthält), nach 60 Sekunden ohne Frame und nach 60 Sekunden ohne neuen Trigger. Siehe [Ein Trigger pro Messung](#one-trigger-per-read--group_id).
> - **Ein Lese-Timeout oder eine unterbrochene Verbindung löst `ChlorosConnectError`** bei jedem Smart-Connect-Aufruf aus sowie `analyze_array_network()`, wobei in Version 1.2.0 der zugrunde liegende Timeout- oder Verbindungsfehler ausgelöst wurde.
> - **`connect_array()` wartet standardmäßig 600 s (`timeout`, 180 s in 1.2.0), und ein Verbindungsversuch, den es aufgibt, wird rückgängig gemacht.** Der Aufruf sendet seinen `timeout` an das Backend, das jede Kamera in den Zustand zurückversetzt, in dem sie gefunden wurde, und keine Verbindung herstellt, wenn die Verbindung nicht rechtzeitig hergestellt wurde; der Aufruf löst `ChlorosConnectError` aus (HTTP-Fehler 504 `CLIENT_DEADLINE_PASSED`) aus. In Version 1.2.0 konnte eine Verbindung, die unmittelbar nach dem Einschalten der Kameras hergestellt wurde, den Aufruf überdauern und das Array verbinden, nachdem dieser bereits ausgelöst worden war. `connect_all()` wartet auf jedes gespeicherte Array auf dieselbe Weise (120 s in 1.2.0). Siehe [`connect_array()`-Signatur](#connect_array-signature).
> - **`connect_array()` behält den vollständigen Sensor bei.** Ohne `width` / `height` / `binning` ist jede Kamera auf den vollen Sensor eingestellt, und die Bildrate sinkt auf das Niveau, das der Host und das Netzwerk liefern; in Version 1.2.0 hat die Bilder verkleinert oder gebinnt, um die Bildrate aufrechtzuerhalten. Siehe [Synchronisiertes Array](#synchronized-array--arraysession-smart-prep).
> - **Ein Bayer-Pixelformat bei einer M3M-Kamera (Mono) entspricht deren Farbtiefe.** Eine M3M bietet nur Mono-Formate an, und die Voreinstellungen bezeichnen diese als Bayer-Formate: In 1.2.0 lehnte die Kamera das Format ab und behielt ihr eigenes bei, während `settings.pixel_format` das Bayer-Format benannte. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, ein `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` sowie ein Arraygemeinsames Format legen nun das Mono-Format derselben Tiefe auf einem M3M fest: `"BayerRG8"` und `high_speed` ergeben `Mono8`; `"BayerRG12"`, `high_quality` und `triggered` ergeben `Mono12`; `cam.settings.pixel_format` benennt es. Das Gleiche gilt auch umgekehrt: Ein M3C wird niemals in ein Mono-Format umgewandelt, daher ergibt `"Mono12"` – unabhängig davon, ob es sich um eines dieser Elemente, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` oder das gespeicherte Format eines Arrays – ergibt auf einem M3C ein `BayerRG12`, unabhängig davon, welche Kamera der Master des Arrays ist. Ein von Ihnen übergebenes `CameraSettings` behält seinen Namen bei, sodass dasselbe Objekt weiterhin jede Kamera nach dieser Tiefe in ihrer eigenen Familie abfragt. `connect_camera(preset=...)` behält das Format der Voreinstellung bei (1.2.0 stellt die Kamera nach Anwendung der Voreinstellung auf 12 Bit ein), und eine über eine Voreinstellung geöffnete Kamera schreibt die aufgenommenen Daten nicht mehr in „`PRESETS`“. Jedes andere Format, das eine Kamera ablehnt (das in ihrer `PixelFormat`-Liste fehlt, wie z. B. `RGB8` auf einem M3M oder `BayerRG16` auf einer Firmware, die es nicht unterstützt), wird auch nicht als eigenes Format benannt: Nach einer Verbindung mit `LatticeCamera`-Verbindung geben `apply_settings()` oder `set_format()` `cam.settings.pixel_format` und `packed_transport` das Format, das die Kamera streamt, und eine gedruckte Zeile das Format, das sie abgelehnt hat.
> - **`analyze_array_network()` gibt an, welche Verbindung gilt:** `pixel_format=None` (früher `"BayerRG8"`) bezeichnet die von der Verbindung festgelegten Formate (Mono12 / BayerRG12), und `packed="auto"` die Entscheidung bezüglich des gepackten Transports, sodass die angegebene Bildrate von 1.2.0'. Ein `pixel_format`, das Sie übergeben, projiziert ein M3M in dieser Tiefe. Siehe [Netzwerkanalyse-Helfer](#network-analysis-helper).
> - **`CameraSession.capture()` gibt ein `CaptureResult` zurück** (immer noch eine Liste von Frame-Dicts) mit `.skipped`, `.failed`, `.daq_files` und `.daq_matches`. Ein `ArraySession.capture()` enthält zudem `.alignment` (ob die Ausrichtungsverzerrung angewendet wurde und warum nicht) sowie `.oriented`.
> - **Array-Erfassungen werden standardmäßig ausgerichtet, ebenso wie die der App.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` und `ArrayHandle.capture()` / `capture_smart()` verzerren die `debayered`-, `display`, `radiance`, `reflectance` sowie die Indexdateien in das Bild der Master-Kamera ein und beschneiden sie auf die Überlappung der Mitglieder (kleiner als der Sensor um die Offsets der Kameras, `XMP-mapir:AlignmentJSON` `"applied": true`), sodass es sich um dieselben Dateien handelt wie bei der Aufnahme derselben Szene durch die Chloros-App; `raw` bleibt in der nativen Größe erhalten. Zuvor wurden bei einer Aufnahme ohne `aligned=True` alle Dateien in ihrer nativen Größe und ohne Verzerrungskorrektur gespeichert. `aligned=False` behält dies bei. `chloros-mcp`’`capture_array` und `start_capture_run` verwenden dieselbe Standardeinstellung. Siehe [`ArraySession`-Methoden](#arraysession-methods).
> - **`connect_all()` benennt ein Backend, das das SDK ablehnt.** Ein laufendes Backend, das abgemeldet ist oder einen Tarif unterhalb von „Copper“ nutzt, wird unter `errors` mit einem eigenen Grund (HTTP-401/403), nicht als „kein Chloros-Backend läuft“, und jeder `connect_all()`-Aufruf überprüft das Backend erneut, sodass ein Aufruf nach der Anmeldung für dasselbe Projekt-Objekt funktioniert.
> - **Ein relativer `output_dir` funktioniert mit einem lokalen Backend.** Relative Pfade werden vor dem Senden in absolute Pfade relativ zum Arbeitsverzeichnis Ihres Skripts umgewandelt. In Version 1.2.0 hat das Backend sie in seinem eigenen Installationsordner aufgelöst, was mit der Fehlermeldung „Zugriff verweigert“ fehlschlug.
> - **`numpy` ist nun eine Voraussetzung** des SDKs (die Live-Frame-Aufrufe geben NumPy-Arrays zurück).
> - **Ein Name auf Export-Ebene, der nicht „one“ lautet, wird abgelehnt.** `processing=` und `levels=` akzeptieren `raw`, `debayered`, `radiance`, `reflectance` oder `display` (oder einen Alias wie `preview`; `processing=` sowie `"all"`). Jeder andere Name wird abgelehnt, bevor die Kamera berührt wird (HTTP-Fehler 400 `UNKNOWN_LEVEL`: `capture()` löst `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); 1.2.0 speicherte oder gab dafür `raw` zurück.
> - **„Texture Aware“ wird für LATTICE-Aufnahmen abgelehnt.** Bei einem Projekt, das LATTICE-Aufnahmen enthält, löst `configure(debayer="Texture Aware (Slow, Highest Quality)")` den Fehler `ChlorosConfigurationError` aus, ebenso wie `process()` nach einem `configure()`, der vor dem Import danach gefragt hatte. Ein zuvor vom Projekt gespeicherter „Texture Aware“-Wert wird mit einem `UserWarning` auf „Standard“ gesetzt. LATTICE-Aufnahmen erhalten immer das Standard-Demosaicing, daher hat Version 1.2.0 die Einstellung akzeptiert und ignoriert; auch die Chloros-App bietet diese Option für sie nicht an. Siehe [Unterstützte Werte](#supported-values).
> - **Kameras werden so zurückgesetzt, wie sie vorgefunden wurden – es sei denn, es liegt ein `keep_settings=True` vor.** `ArraySession.disconnect()` / `force_disconnect()`, das Ende jeder vorübergehenden Funktion (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) sowie das Beenden von `start_camera_continuous()` versetzen jede Kamera wieder in den ursprünglichen Zustand: die Verkabelung von Auslöser und Synchronisationsleitung, die PTP-Einstellung sowie alle Kameraeinstellungen, die während der Sitzung geändert wurden (Belichtung und Verstärkung, das AE-Ziel, Grenzwerte und Dämpfung, Weißabgleich, Schwarzpegel, Pixelformat, Binning und ROI, Spiegelungen, Bildfrequenz). `keep_settings=True` speichert stattdessen die Einstellungen jeder Kamera im kameraseitigen Speicher, sodass sie beim Einschalten mit diesen Einstellungen startet; `clear_settings=True` setzt diesen Speicher auf die Werkseinstellungen zurück. In beiden Fällen laufen die Kameras im Freilaufmodus weiter. Ein Projekt ruft beim Verbinden der Kameras deren eigene gespeicherte Einstellungen ab. Siehe [Kameraspeicher](#camera-memory--keep_settings--clear_settings).
> - **Jeder Modus speichert ein Projekt auf dieselbe Weise.** Jede Änderung an einer Kamera, einem Array oder einem Lichtsensor, die im Rahmen eines Projekts verbunden sind – sei es über das SDK, die App, die CLI oder den MCP-Server – wird in der Datei `cameras.json` / `sensors.json` gespeichert, und beim Öffnen des Projekts in einem beliebigen Modus werden die Geräte entsprechend konfiguriert; ein `ChlorosProject` benennt seinen eigenen Ordner im Backend. Zuvor speicherte nur die App das Projekt. Siehe [Ein Projekteintrag, jeder Modus](#one-project-record-every-mode).
> - **Dateien, die aus mehreren Kameras erstellt wurden, benennen jede einzelne.** Ein Multiband-Stack, ein aus den Bändern mehrerer Kameras erstellter Live-Index und das Bild einer Array-Aufnahme `composite/` enthalten die Seriennummer, das Modell und das Kalibrierungspaket jeder Quellkamera als Listen (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), und die EXIF-Daten `SerialNumber` listen die Seriennummern durch Kommas getrennt auf. Früher enthielten sie keine Kamerainformationen oder nur die Seriennummer der ersten Kamera. Siehe [Dateien, die von mehreren Kameras erstellt wurden](#files-made-from-several-cameras).
> - **Nur eine `raw`-Datei enthält die Kalibrierungs-Tags der Kamera.** Das MicaSense-kompatible Set (`BlackLevel`, `RadiometricCalibration`, die`Band{R,G,B}RadiometricCalibration`, die `MicaSenseVignetting*`-Sätze) sowie die `CalibrationJSON`-Nutzdaten beschreiben die Rohwerte des Sensors und werden daher ausschließlich in der `raw`-Erfassungsdatei. Alle anderen Dateien – eine Erfassungsdatei`debayered`, `display`, `radiance`, `reflectance` sowie Indexdateien, Frames aus kontinuierlichen Aufzeichnungen und Mehrkamera-– sowie jedes `process()`-Produkt – enthält die Identifikation der Kamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, …) und bei einer Aufnahme die Aufzeichnungsdaten des jeweiligen Bildes (`XMP-mapir:FrameJSON`: Belichtung, Verstärkung, Temperatur, Pixelformat, Aufnahmezeit). In Version 1.2.0 enthielten sie auch den Rohdatensatz, den ein Tool (Pix4D, Metashape) auf Pixel anwandte, bei denen es sich nicht um Rohwerte handelte. Die Verarbeitung ermittelt die Kalibrierung einer solchen Datei anhand ihrer Seriennummer und des Eintrags `CalBundleSha256`.
>
> **Weitere Änderungen:**
>
> - **Die automatische Bildrate eines Arrays richtet sich nach dem, was das Array liefert.** Eine Bildrate, die mehr als 1 % ihrer Auslöseimpulse verliert, wird heruntergestuft – wobei dies über mehrere längere Beobachtungszeiträume statt nur über einen einzigen beurteilt wird –, und eine Bildrate, von der sie heruntergestuft wurde, wird von Zeit zu Zeit erneut versucht. Eine von Ihnen selbst festgelegte Bildrate bleibt unverändert.
> - **Die automatische Belichtung ist stabiler.** Sie bleibt bei konstanter Lichtstärke konstant, pendelt sich innerhalb von Sekunden nach einer Änderung ein, ignoriert kleine Überbelichtungen und bringt eine Kamera wieder zum Laufen, die bei starker Sonneneinstrahlung nicht mehr reagiert. Siehe [So verhält sich die automatische Belichtung](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` zählt nur Ihre Bilder und Aufnahmen.** In einem Ordner, der die `camera_cal/`-Dateien eines Projekts enthält, werden diese Kalibrierungsdateien nicht mehr als Bilder und Aufnahmen gezählt, und ein Projekt, das sie bereits aufgenommen hat, verwirft sie.
> - **`configure(radiometric_output=...)` gilt.** Es legt nun die Projektprodukte `Export radiance` / `Export reflectance` fest; zuvor hatten die eigenen Produkteinstellungen jedes Projekts Vorrang, sodass `radiance`, `sensor-response` und `all` alle Strahlungs- und Reflexionswerte exportierten.
> - **`test_array_sync()` wird ab dem Einschaltzustand der Kameras ausgeführt.** Es schaltet das PTP der Kameras für den Durchlauf ein, wartet darauf, dass sich ihre Takte vor dem ersten Trigger (`ptp_lock`) synchronisieren und stabilisieren – jede Kamera benötigt 120 s, also etwa 2 Minuten ab dem Einschalten (`ptp_settle_s=` setzt es), da eine gerade synchronisierte Uhr etwa 2 Minuten lang eine Abweichung von bis zu ~200 µs aufweist — und stellt das PTP jeder Kamera wieder so ein, wie es vorgefunden wurde. Auf eine Kamera, die noch aus einem früheren Test synchronisiert ist, wird nicht erneut gewartet. Früher wurde das PTP in einem Netzwerk mit nur einem Switch, in dem kein DAQ-E antwortete, ausgeschaltet gelassen, und die frei laufende Uhr jeder Kamera wurde als PTP-Uhr gelesen, die sich nicht synchronisieren würde („PTP ist nicht synchronisiert“, bei jedem Durchlauf).
> - **Eine `.daq`-Aufzeichnung gibt an, ob ihre Kappe ausgewählt oder angenommen wurde.** DAQ-U, DAQ-M und DAQ-E werden ohne Kappe ausgeliefert, und Chloros kann nicht erkennen, ob eine angebracht ist; daher erhält ein Sensor, der ohne benannte Kappe angeschlossen ist, standardmäßig die Bezeichnung `sunshine_cosine` – eine *Annahme*. Jede Chloros-Aufzeichnung (Schema 1.30) speichert nun neben `cap_id` auch `cap_id_source`: `operator`, wenn Sie eine Kappe benannt haben (einschließlich `none`), `auto_default`, wenn es der Standardwert war. Bei der Verarbeitung eines Datensatzes, bei dem der Kondensator angenommen wurde, während der DAQ-Kondensator des Projekts auf „Auto“ belassen wurde, wird ein `[daq-dls] WARNING` ausgegeben, der den Dateinamen enthält: Ein reines DAQ-U unter der Sonneneinstrahlungskorrektur liefert einen etwa 30-fach zu hohen Wert. **Benennen Sie den von Ihnen eingestellten Cap** beim Anschließen (`connect_daq_sensor(cap_id=...)` oder `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap` oder die Kondensator-Auswahl in der GUI), oder legen Sie den DAQ-Kondensator des Projekts fest, dann verschwindet die Warnung.
> - **Keine PIP-Konsolenfenster mehr unter Windows.** Auf einem PC mit installiertem Python führten das Backend und jeder seiner Image-Worker beim Start einmal `pip` aus, woraufhin jeweils ein Konsolenfenster geöffnet wurde. Dies ist nun nicht mehr der Fall.
> - **Live-`.daq`-Aufzeichnungen überstehen einen Stromausfall.**Eine Aufzeichnung wird bei jedem Commit hinter dem Sensor mit einem Journal und einer vollständigen Festplattensynchronisation geschrieben, sodass eine langsame Festplatte zwar die Datei verzögert, niemals aber die Messwerte. Eine Aufzeichnung, die mitten in einem Commit gestoppt wurde (Stromausfall, Abbruch), kann eine `<name>.daq-journal`-Datei daneben hinterlassen:**Kopieren Sie beide zusammen.** Chloros rollt das Journal beim ersten Lesen der Datei zurück, sodass die Aufzeichnung mit allen vor dem Abbruch festgeschriebenen Daten geöffnet wird (etwa die letzten 3 Sekunden gehen verloren); in Version 1.2.0 konnte ein Stromausfall zum falschen Zeitpunkt dazu führen, dass die gesamte Datei unlesbar wurde („Datenbank-Festplattenabbild ist fehlerhaft“).
> - **`DAQSensorSession.record_start()` auf einer ausgelasteten Festplatte blockiert den Sensor nicht mehr**: Die Messwerte werden in eine Warteschlange gestellt und in einem eigenen Thread geschrieben; ein Wert, der nicht geschrieben werden konnte, wird in `status()`’s `record_errors` (zusammen mit `record_last_error`) gezählt.
> - **Direktes `LatticeCamera`:**Ein Frame ohne Chunk-Belichtung/Verstärkung meldet nun die**tatsächliche** Belichtung und Verstärkung der Kamera (die Fallback-Lösung las einen zwischengespeicherten Wert, der während eines gesamten automatischen Belichtungsdurchlaufs unverändert bleiben konnte), und `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` wenden denselben 5-µs-Eingangs-Deglitch an, den Chloros’ Kabelslaves erhalten (ungefiltert: Ein Slave löste Frames aus, ohne dass ein Trigger gesendet wurde).
> - **Softwaregesteuerte Kameras lassen nicht mehr jeden zweiten Trigger fallen.** Eine Kamera im Software-Trigger-Modus stellt nun einen Trigger, der während ihrer Auslesung eintrifft, in die Warteschlange (oder Chloros wartet, bis sie bereit ist), wie es bei kabelgesynchronisierten Kameras bereits der Fall war.
> - **`discover_daq_sensors()` bezeichnet ein DAQ-E-S als `DAQ-E-S`.** Sein `model` lautete für jeden Ethernet-Sensor `DAQ-E`; ein Sensor, der nicht innerhalb von etwa 2 s auf die Identitätsprüfung antwortet, wird weiterhin als `DAQ-E` aufgeführt.
> - **Neu: `chloros-mcp`, ein MCP-Server für KI-Assistenten.** Claude Desktop, Claude Code, Cursor und andere MCP-Clients können Bildmaterial verarbeiten (ein Projekt erstellen, Aufnahmen importieren, Einstellungen auswählen, verarbeiten, die Ergebnisse des Durchlaufs ausgeben) und LATTICE-Kameras, Arrays und DAQ-Lichtsensoren steuern: verbinden, Vorschau anzeigen, Belichtung und Farbe einstellen, aufnehmen, aufzeichnen. Die darin enthaltenen Werkzeuge decken die Funktionen dieses SDKs ab. Es wird zusammen mit dem SDK installiert, und das Linux-.deb-Paket legt es unter `/usr/bin` ab. Siehe [MCP-Server (KI-Assistenten)](#mcp-server-ai-assistants).
> - **Neu: Kamera-Dienstprogramme**, die Befehle in `chloros-cli lattice` als SDK-Aufrufe: Farbprofil, Sättigung/Kontrast, Lichter, Weißabgleich (Einmal, Auto, feste Verhältnisse), Voreinstellungen, Filtermodelle, Firmware-Status und -Update, Stromversorgung (Abfrage, Ruhemodus, Aufwachen), Zielkalibrierung, Netzwerkadapter (Prüfen, Schätzen, Korrigieren), Vignettierungskorrektur bei Dateien, eine DLS-Messung und die Host-Abfrage. Siehe [Kamera-Dienstprogramme](#camera-utilities).
> - **Neu: Der Rest der CLI als SDK-Aufrufe**, sodass die CLI, das SDK und `chloros-mcp` dieselben Funktionen bieten (die CLI behält aus Sicherheitsgründen nur die Anmeldung, die Bereitstellung eines DAQ-E und die direkten DAQ-Server bei): Kameras, die für einen Befehl geöffnet werden (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), Ausrichtungsprofil-Dateien (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), Indizes (`index_catalog()`, `compute_index()`, `capture_live_index()`), den PTP-Grandmaster (`time_sync_status()` und Co.), den Projektordner (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) sowie Rezepte (`run_project_recipe()`, der CLI-eigene Runner). Für Lichtsensoren: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` und `light_sensor_reflectance()` (Umgebungs- und Objektreflexion sowie Indizes von zwei Sensoren); für eine Kamera: `CameraSession.status()` und `capture_repeated()`; für ein Array: `ArraySession.capability()` und `set_member_settings()` (Belichtung, Verstärkung usw. eines Elements). Siehe [Transiente Arrays, Ausrichtungsprofile und Indizes](#transient-arrays-alignment-profiles--indices), [Projektordner und Rezepte](#project-folder--recipes) und [DAQ-Sensorsitzungen](#daq-sensor-sessions).
> - **Verbesserte LATTICE-Indexprodukte** (NDVI und andere Indizes) aus `process()` in jedem Verarbeitungsmodus.
> - **LATTICE-OCN-Kameras: verbesserte Radiance-, Reflektanz- und Indexprodukte bei Sonnenlicht.** Nach der Bandentmischung wird eine Flottenkorrektur angewendet – live, bei Aufnahmen und in `process()`; RGN-Kameras bleiben unverändert. Siehe [Radiometrische Ausgabe](#radiometric-output-lattice-multispectral-pipeline).
> - **Jede Aufnahme speichert den verwendeten DAQ-Messwert, genau wie die GUI.** `CameraSession.capture()` und `ArraySession.capture()` / `continuous()` schreiben die Datei `.daq` und deren `.match.json`, wann immer ein DAQ gebunden oder in den Pool aufgenommen wird, einschließlich einer reinen Rohdatenaufnahme; in 1.2.0 wurde eine Roh- oder debayerte Aufnahme nur mit `force_daq=True` gespeichert. `daq=False` deaktiviert dies und `daq_tolerance_ms=` legt die Übereinstimmungstoleranz fest; beide Ergebnisse geben an, was in `.daq_files` und `.daq_matches` gespeichert wurde. Eine Aufnahme wartet nun auf den nächsten Messwert des DAQ, damit sie die Messwerte auf beiden Seiten der Belichtung zusammenführen kann. Siehe [Der vom DAQ gelesene Messwert, den eine Aufnahme speichert](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` verfügt über Live-Frames und eine serverseitige kontinuierliche Aufnahme.** `grab()` und `frame_stream()` geben Numpy-Frames von einem Trigger zurück (derselbe Lesevorgang wie bei den Projekt-API-Funktionen `ArrayHandle.grab()` / `frame_stream()`), und `continuous()` speichert jede synchronisierte Gruppe im Backend und gibt eine [`ContinuousHandle`](#continuoushandle). Siehe [Live-Frames in Ihrer eigenen Pipeline](#live-frames-in-your-own-pipeline).
> - **Ein Array `grab()` ist an einen Trigger gebunden** (`ArrayHandle.grab()` und das neue `ArraySession.grab()`). In 1.2.0 liest `ArrayHandle.grab()` 12 von 20 vierKameras-`ArrayHandle.grab()`-Lesevorgänge erstreckten sich über zwei Trigger. Jedes Bild enthält nun `group_id`, und ein Array `frame_stream()` liefert jeden Trigger einmal. Siehe [Ein Trigger pro Lesevorgang](#one-trigger-per-read--group_id).
> - **Bandweise und ausgerichtete Frames:** `grab()` / `frame_stream()` übernehmen `band=`, `align=`, `bit_depth=` und `refl_max=` als Kamera- und Array-Handles. Siehe [Ein einzelnes Band exportieren](#exporting-a-single-band--band).
> - **Neue `ArraySession`-Methoden und -Argumente:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (zusammen mit `CaptureResult.group_seq` und `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` verkettet aufeinanderfolgende Gruppen und stoppt bei einem vollständigen Durchlauf.
> - **`ArraySession.record()` zeichnet auf, ohne dass eine GUI geöffnet ist.** Wenn die kombinierte Ansicht der GUI geschlossen ist, rendert das Backend die Ansicht selbst: denNDVI oder den Index, den Sie als `expr=` übergeben. `RecorderHandle.stop()` löst bei einer fehlgeschlagenen Aufzeichnung, bei der kein Frame geschrieben wurde, einen Fehler aus und speichert das Stopp-Ergebnis in `.result` (das Belassen eines `with`-Blocks löst keine Ausnahme aus: Überprüfen Sie `rec.result`). Siehe [`RecorderHandle`](#recorderhandle).
> - **Neue `DAQSensorSession`-Methoden:** `enable_ptp()`, `force_disconnect()` und für die eigenen Einstellungen eines DAQ-E `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` und `update_firmware()`. `record_stop()` gibt `warning` und `gaps` zurück. Ein Neustart, der eine Aufzeichnung beendet hat, nennt diese in seiner Antwort oder, falls der Neustart fehlschlägt, im `ChlorosConnectError` (`recording_closed`; siehe [DAQ-E-Geräteeinstellungen](#daq-e-device-settings)). Siehe [`DAQSensorSession`-Methoden](#daqsensorsession-methods).
> - **Ein gepooltes DAQ-E-Gerät, das keine Daten mehr liefert, wird vom Backend** mit seinen bisherigen Einstellungen wieder verbunden, und eine darauf laufende Aufzeichnung wird im selben `.daq` fortgesetzt, wobei die Lücke in der Datei markiert wird; `record_stop()` gibt die Lücken als `gaps` zurück. In Version 1.2.0 blieb der Sensor stumm, bis er getrennt und erneut verbunden wurde, wodurch die Aufzeichnung beendet wurde. Siehe in der CLI-Referenz den Abschnitt [Ein DAQ-E, das keine Daten mehr liefert](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` funktioniert über ein installiertes SDK.** Das Rad (das des Installersund PyPI) enthält nun die benötigten Projektlader; zuvor löste `open_project()` den Fehler `ImportError` aus, der `project_cameras` / `project_sensors` in jedem per pip installierten SDK aus. `connect_all()` verbindet die gespeicherten Kameras, Arrays und Lichtsensoren über ein laufendes Backend (Lichtsensoren über dessen DAQ-Pool), und ein gespeichertes Gerät, nach dem es nicht suchen kann, wird unter `errors` mit einer Begründung aufgeführt, anstatt wortlos übersprungen zu werden. Ein gespeichertes Array, das bereits im Backend (dem der GUI, oder in `chloros-cli project connect`), wird nun eingebunden und läuft weiter, während dies zuvor als Fehler gemeldet wurde („Kamera  befindet sich<sn> bereits im Array<id>“). Siehe [`ChlorosProject`-Methoden](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` und `process_lattice_capture()` finden Chloros unter Windows.** Sie suchen dort, wo der Installer das Backend ablegt (standardmäßig `<install folder>\backend\`, `C:\Program Files\Chloros\backend\`, ein benutzerdefinierter Ordner aus den eigenen Aufzeichnungen des Installationsprogramms) und stellen eine Verbindung zu einem bereits laufenden Backend her, ohne überhaupt nach einem zu suchen. In Version 1.2.0 wurde auf einem installierten Rechner die Fehlermeldung „Chloros-Backend nicht gefunden“ ausgegeben, sofern Sie `backend_exe=` nicht übergeben hatten.
> - **Ein Backend, das beim Start abstürzt, wird sofort gemeldet.** Wenn `ChlorosLocal()` oder ein Smart-Connect-Aufruf ein Backend startet, das beendet wird, bevor es antwortet, löst der Aufruf innerhalb von Sekunden den Fehler `ChlorosBackendError` (von `ChlorosLocal`) und `ChlorosConnectError` (von den Smart--Connect-Aufrufe ausgelöst, zusammen mit dem Beendigungscode, dem ausgeführten Befehl und den letzten Zeilen, die das Backend ausgegeben hat. Version 1.2.0 wartete das 60-Sekunden-Start-Timeout ab und gab keinen Grund an. Siehe [Backend-Autostart](#backend-auto-start).
> - **Das Backend läuft zwischen den Skripten weiter.** Das Verlassen eines `with ChlorosLocal(...)`-Blocks, `process_folder()` / `process_lattice_capture()`, die zurückkehren, und `chloros-mcp`, das beendet wird, halten ein vom SDK-gestartetes Backend nicht mehr an; das nächste Skript nutzt es weiter. Es wird bei `logout()` beendet (es sei denn, die Chloros-App ist geöffnet oder ein Lauf ist im Gange), bei `shutdown_backend()` oder von selbst nach 30 Minuten, wenn es nicht genutzt wird (`CHLOROS_BACKEND_IDLE_MIN` in der Umgebung des Backends ändert dies; `0`: niemals).
> - **Ein vom SDK-gestartetes Backend wird zum Herunterfahren aufgefordert, nicht beendet.** `shutdown_backend()` lässt das Backend seine Kameras und Lichtsensoren freigeben und etwaige Aufzeichnungen beenden; es wird nur beendet, wenn es 10 Sekunden später noch läuft. In Version 1.2.0 wurde es sofort beendet (unter Windows ein „Hard Kill“), was dazu führen konnte, dass ein mit diesem Backend verbundener DAQ-Lichtsensor nicht mehr reagierte. Ein Backend, das das SDK als laufend erkannte, wird weiterhin in Ruhe gelassen.
> - **Unter Linux führt der Autostart niemals ein `backend_server.py` aus dem aktuellen Ordner aus.** War kein Backend installiert, führte ein in einem Ordner mit einem Backend gestartetes Skript dieses aus; nun wird nur noch das installierte Backend gestartet (oder, im Fall von `ChlorosLocal`, das von `backend_exe=` angegebene).
> - **`configure(indices=...)`**fügt jede Voreinstellung für die Bänder der jeweiligen Kamera zum Projekt hinzu, einschließlich eines neuen Projekts, und gibt eine Warnung (`UserWarning`) aus, wenn ein Name nicht hinzugefügt werden kann.**`import_images()`** meldet `.daq`-Aufnahmen als `scan_count` und gibt keine Warnung mehr bezüglich eines LATTICE-Aufnahmeordners aus.
> - **`read_image_audit_tags()` funktioniert nach einer pip-Installation.** Es liest die Metadaten des Bildes selbst aus; in Version 1.2.0 gab es `{}` zurück, ohne dass die internen Komponenten der Desktop-Installation im Python-Pfad vorhanden waren.
> - **`capture()` direkt nach `connect_array()`** wartet bis zu 20 Sekunden auf die erste Bildgruppe des Arrays, anstatt mit dem HTTP-Fehler 503 „Array still warming up“ abzubrechen.
> - **`status()` meldet einen Stillstand.** Das neue Feld `stalled` ist „True“, während die Erfassungsschleife läuft, veröffentlicht jedoch nichts, und `actual_fps` ist dann 0. In Version 1.2.0 behielt `actual_fps` während eines einen Stillstand bei, sodass ein blockiertes Array „`actual_fps > 0`“ anzeigte, während die Aufnahmen mit dem Fehler 503 „No fresh synced frame group“ fehlschlugen.
> - **Weniger unvollständige Frame-Sätze und 503-Fehler bei der Erfassung:** Der Triggereingang jeder Slave-Kamera wird nun entglitcht.
> - **Gepackter 12-Bit-Transport** (`connect_array(packed="auto")`) erhöht die Bildraten des Vollsensor-Arrays um etwa ein Drittel. Siehe [Gepackter 12-Bit-Transport](#packed-12-bit-transport-packed).
> - **Belichtung des Arrays innerhalb eines ROI:** Ein kleiner ROI begrenzt die Belichtung eines Array-Mitglieds nicht mehr auf die Freilaufzeit des Sensors (4,9 ms bei 768 × 304). Siehe [Belichtung von Array-Mitgliedern](#exposure-on-array-members).
> - **Die automatische Belichtung senkt die automatische Bildrate eines Arrays nicht mehr.** Ein kabelgesynchronisiertes Array läuft mit der Rate, die das Netzwerk und die Auslesegeschwindigkeit der Kameras zulassen; die automatische Belichtung füllt die durch diese Rate vorgegebene Periode aus und erhöht dann die Verstärkung. Eine manuelle Belichtung, die länger als die Periode dauert, verlangsamt den Auslöser, um sie anzupassen, anstatt abgelehnt zu werden. Siehe [Belichtung auf Array-Elementen](#exposure-on-array-members).
> - **`read_nodes()`**fügt `frame_values` und `notes` hinzu, während die automatische Belichtung läuft (siehe [`read_nodes()` bei automatischer Belichtung](#read_nodes-under-auto-exposure--frame_values-and-notes)). Wenn die Kamera eine Belichtung oder Verstärkung ablehnt, gibt**`set_settings()`** den akzeptierten Bereich an und schaltet den für diesen Wert ausgeschalteten Automatikmodus wieder ein; `set_settings(target_brightness_auto=True)` ist neu.
> - **`orient=`** bei `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` und `ArrayHandle.capture()` / `capture_smart()` dreht die Produkte entsprechend der Kameraausrichtung. Siehe [`ArraySession`-Methoden](#arraysession-methods).
> - **`DLS()` benötigt die sensoreigene `mac_address`.** Es gibt keine Standard-MAC mehr (es handelte sich um die Adresse einer Einheit); `DLS()` ohne eine solche löst `DLSError` aus. Siehe [DAQ-M-Band-Magnituden](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Der Windows-Installer** bietet die Optionen „Windows-Langpfade aktivieren“ und „Das Chloros-Backend vom Microsoft Defender-Scan ausschließen“ an, die beide standardmäßig aktiviert sind. Siehe [Schritt 1](#step-1--install-the-chloros-platform-package).
> - **Das Linux-.deb-Paket** installiert das Python-SDK für das System `python3` (bisher wurde lediglich das Wheel auf der Festplatte abgelegt), vergrößert die vom Kernel benötigten Socket-Empfangspuffer für Kamerastreams und fügt einen Backend-Dienst hinzu, der unter Ihrem Benutzerkonto läuft. Das Jetson-Paket lautet `chloros_1.3.0_arm64_jp6.deb` für JetPack 6, mit JetPacks CUDA-PyTorch. Siehe [Schritt 1](#step-1--install-the-chloros-platform-package) und [Schritt 2](#linux-deb).

---

## Architektur in 60 Sekunden

Das SDK ist eine schlanke Python-Schicht über dem Chloros-Backend (derselbe Flask-Server, den auch die Desktop-GUI und die CLI verwenden). Zur Automatisierung importieren Sie `chloros_sdk` und rufen Methoden auf hoher Ebene auf; im Hintergrund wird jeder Aufruf zu einer HTTP-Anfrage an das lokale Backend auf Port 5000 – `http://127.0.0.1:5000/api/...` (bewusst nicht `localhost`, das unter Windows zunächst zu `::1` aufgelöst wird und bei einem reinen IPv4-Backend etwa 2 Sekunden pro Anfrage kostet). Das Backend verwaltet den Hardware-Pool – Kameras, DAQ-Sensoren, Ausrichtungsprofile, Bildpuffer –, sodass SDK-Skripte neben der GUI existieren können, ohne um serielle Schnittstellen oder Netzwerkbandbreite konkurrieren zu müssen.

Es gibt drei Schnittstellen, die Sie verwenden werden:

1. **`ChlorosLocal` + freie Funktionen** (`process_folder`, `process_lattice_capture`) – Bildverarbeitungspipeline. Führen Sie mit einem einzigen Python-Aufruf die Kalibrierung, den Debayering-Vorgang und den Index-Export für einen gesamten Ordner durch.
2. **Smart-Connect-Handles** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Öffnen Sie eine dauerhafte Backend-Sitzung für Live-Hardware. Gleicher „Smart-Prep“-Ablauf wie in der GUI: Netzwerkprüfung, automatische Tier-Auswahl, PTP, AE-Seeding, GPIO-Trigger-Konfiguration.
3. **`ChlorosProject` / `open_project`** — Lädt ein gespeichertes Projekt (Ordner mit `cameras.json` + `sensors.json` + `project.json`), verbindet dessen Kameras, Arrays und Lichtsensoren auf einmal und steuert die Aufnahmen über benannte Handles.

Die Oberflächen 1 und 2 **starten automatisch ein lokales Backend**, falls noch keines im Listen-Modus ist (dasselbe mitgelieferte Binärprogramm, das auch die GUI und die CLI starten) – so funktioniert ein einfaches Skript in einer neuen Shell, ohne dass Sie zuvor ein Backend starten müssen: `ChlorosLocal` (sowie `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` und `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` und `analyze_array_network()` starten niemals selbstständig: Sie benötigen ein laufendes Backend. Übergeben Sie `auto_start_backend=False`, um dies zu deaktivieren (z. B. wenn auf ein Remote-Backend verwiesen wird, das niemals gestartet wird). Siehe [Automatischer Start des Backends](#backend-auto-start). Surface 3 verhält sich anders: `open_project()` akzeptiert keinen `auto_start_backend`-Parameter, und `connect_all()` startet niemals ein Backend – es fragt einmal nach `http://127.0.0.1:5000` nach und greift, falls keine Antwort erfolgt, auf die direkte (backendfreie) Gerätesteuerung über `lattice_sdk` zurück. Dazu sind die Arena-SDK-Laufzeitumgebung und OpenCV (`cv2`) in Ihrem Python importierbar sein, was das SDK-Wheel nicht installiert; ohne diese werden jede gespeicherte Kamera und jedes Array als Fehler gemeldet, wobei der Grund angegeben wird. Die Lichtsensoren eines Projekts verfügen über keinen backend-freien Pfad: Wenn kein Backend läuft, wird jeder gespeicherte Sensor als Fehler gemeldet, wobei der Grund angegeben wird (siehe [`SensorHandle`](#sensorhandle)). Nur `proj.process()` und `stream(..., overlays=True)` erstellen verzögert ein `ChlorosLocal()` (das automatisch startet).

Alle drei sind auth: Führen Sie `chloros-cli login` einmal auf dem Rechner aus oder melden Sie sich über die Desktop-GUI an. Was ein Aufruf ohne gültige Sitzung auslöst, hängt von der Oberfläche ab (siehe [Ausnahmen](#exceptions)): `ChlorosAuthenticationError` von `ChlorosLocal` (sowie `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` von den smart-Connect-Aufrufe und deren Sitzungen, `status: "error"` mit einem `error_code` aus `analyze_array_network()` sowie ein Eintrag unter `errors` im `connect_all()`-Bericht.

Voraussetzungen:
- Python 3.7+ (wie vom Paket angegeben; entwickelt/getestet unter 3.10)
- Lokal installiertes Chloros Desktop (die Backend-Binärdatei ist im Installationsprogramm enthalten)
- Aktives Chloros+-Konto in **einer beliebigen kostenpflichtigen Chloros+-Stufe (Copper oder höher)**; die kostenlose**Iron**-Stufe bietet keinen SDK- bzw. CLI-Zugriff. Dies wird**auf der Server**:** Jede mit SDK oder CLI gekennzeichnete Anfrage muss sowohl eine aktive Sitzung als auch einen kostenpflichtigen Tarif enthalten. Ein Tarif unterhalb von „Copper“ erhält den Fehlercode `403` zusammen mit `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` aus `ChlorosLocal`, `ChlorosConnectError` bei Smart-Connect-Aufrufen). Ein abgemeldeter Aufrufer erhält stattdessen `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` aus `ChlorosLocal`, `ChlorosConnectError` aus den Smart-Connect-Anrufen) – die beiden unterscheiden sich, da die erneute Ausführung von `chloros-cli login` behebt den Fehler `401`, kann jedoch den Fehler `403` nicht beheben, für dessen Behebung ein Plan-Upgrade erforderlich ist.
- Die Offline-Nutzung wird innerhalb der Karenzzeit des Tarifs unterstützt: Die Tarifstufe wird aus dem Server-Validierungs-Cache (5 Minuten) oder dem signierten, maschinengebundenen Lizenz-Cache (30 Tage bei monatlichen Tarifen, bis zum Ablauf des Abonnements bei jährlichen Tarifen). Nach Ablauf dieser Karenzzeit wechselt der Tarif in die kostenlose Stufe, und der SDK-/CLI-Zugriff wird unterbrochen, bis das Gerät den Server einmal erreichen kann. Der Fehler `chloros-cli status` (`GET /api/license-status`) bleibt in der kostenlosen Stufe erreichbar, sodass der Grund sichtbar bleibt.
- Windows 10/11 64-Bit, **Ubuntu 22.04 LTS oder neuer**oder Jetson (JetPack 6). Ubuntu 20.04 wird**nicht** unterstützt: Die Abhängigkeiten von `.deb` leiten sich aus den Bibliotheken ab, gegen die das Backend verknüpft ist, einschließlich `libc6 (>= 2.35)`, und Focal liefert glibc 2.31 aus.

---

## Installation

Das Python-SDK ist eine dünne Python-Schicht über dem Chloros-Backend. Sofern Sie kein Backend auf einem anderen Rechner betreiben (siehe [Remote-Backend-Modus](#remote-backend-mode-pip-only-host-via-tunnel)), benötigen Sie das **Chloros-Desktop-Paket lokal installiert** (Windows-Installationsprogramm oder Linux `.deb`), das auch DAQ-Lichtsensoren enthält. Das Paket enthält die Backend-Binärdatei, die Arena-SDK-Laufzeitumgebung für LATTICE-Kameras sowie die Kalibrierungs-Bundles.

Aktuelle Downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Schritt 1 – Installieren Sie das Chloros-Plattformpaket

#### Windows (.exe)

1. Laden Sie `Chloros Installer 1.3.0.exe` von der Download-Seite herunter.
2. Führen Sie das Installationsprogramm aus und folgen Sie den Anweisungen des Assistenten. Der Standardinstallationspfad lautet `C:\Program Files\Chloros\`.
3. Starten Sie Chloros mindestens einmal und melden Sie sich mit Ihrem Chloros+-Konto an.

Das Installationsprogramm lässt das Backend (`chloros-backend.exe`, sonst nichts) in jedem Netzwerkprofil durch die Windows-Firewall: UDP-Eingangsverkehr für Kamerastreams, PTP, mDNS und den DAQ-E-Stream sowie TCP-Eingangsverkehr ausschließlich aus dem lokalen Subnetz für die Verbindung, die ein DAQ-E während eines Firmware-Updates zurück zum Computer herstellt ([`update_firmware()`](#daq-e-device-settings)). Das Deinstallationsprogramm entfernt beide Regeln. Die HTTP-API des Backends bleibt in jedem Fall auf `127.0.0.1`.

Die Gruppe **Systemeinstellungen** des Assistenten enthält zwei Optionen, die standardmäßig beide aktiviert sind:

| Option | Funktion | Deaktivierung der stillen Installation |
| --- | --- | --- |
| **Lange Windows-Pfade aktivieren** | Ermöglicht es, Projektordner tiefer als das Windows-Limit von 260 Zeichen anzulegen. Wenn lange Pfade deaktiviert sind (Windows-Standard), wird ein Projektordner, der für seine Produkte zu tief liegt, abgelehnt, bevor irgendetwas verarbeitet wird – siehe [Fehlerbehebung](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Das Chloros-Backend vom Microsoft Defender-Scan ausschließen** | Verhindert den Echtzeit-ScanEchtzeit-Prüfung jeder Datei, die eine Kameraaufnahme schreibt. Bei hohen Aufnahmeraten kostet die Prüfung etwa 30 % der Rate: Gemessen bei ROI 512 auf einem Array mit 9 Kameras lief die kontinuierliche RAW-Aufnahme mit 52,7 Aufnahmen/s mit Prüfung und mit 77,7 Aufnahmen/s ohne das Backend. | `/MERGETASKS="!defenderexclusion"` |

Eine kontinuierliche Aufnahme meldet die Auslastung von Defender als `defender_cores` (siehe [`ContinuousHandle`](#continuoushandle)).

#### Linux-amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Installieren Sie mit `apt install ./…`, nicht mit `dpkg -i`: apt installiert auch die Voraussetzungen des Python-SDKs, die das Paket empfiehlt, aber nicht zwingend erfordert (siehe [Schritt 2](#linux-deb)).

#### Linux arm64 – Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Für Jetson Orin / Orin NX / Orin Nano auf JetPack 6 (L4T R36). Das verwendete PyTorch ist die CUDA-Version von JetPack 6, sodass die GPU-Verarbeitung auf der GPU des Orin läuft; es nutzt die CUDA- und cuDNN-Bibliotheken, die JetPack 6 installiert (`sudo apt install nvidia-jetpack` auf einem Image, auf dem diese fehlen).

#### Linux: Empfangspuffer und der Backend-Dienst

Beide .deb-Pakete erhöhen die vom LATTICE-Kamerastream benötigten Socket-Empfangspuffer des Kernels auf 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). Bei der Kernel-Standardeinstellung werden Kamerapakete im Kernel verworfen, und ein Array verliert Frames und Bildrate; `chloros-cli lattice network` meldet, wenn die Puffer fast voll sind. Siehe den Abschnitt [Linux-Hosts – Socket-Empfangspuffer](cli-reference.md#linux-hosts--socket-receive-buffers) in der CLI-Referenz.

Führen Sie auf einem Rechner ohne grafische Oberfläche das Backend als Dienst **unter Ihrem Benutzer** aus, damit ein Skript ein laufendes Backend findet (`connect_all()` startet niemals ein Backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Das Backend läuft dann als Sie: Die von ihm geschriebenen Aufzeichnungen gehören Ihnen, und es verwendet Ihr `chloros-cli login`. Das Root-`chloros-backend.service` ist weiterhin auf Rechnern installiert, auf denen es aktiviert wurde, aber jede Datei, die ein Root-Backend schreibt, gehört root. Siehe den Abschnitt [Ausführen des Backends als Dienst](cli-reference.md#running-the-backend-as-a-service-headless) in der CLI-Referenz.

Ein vom Dienst ausgeführtes Backend wird gemeinsam genutzt und niemals von einem Client gestoppt: SDK-Skripte und `chloros-cli` nutzen es so, wie sie es vorfinden (`ChlorosLocal.shutdown_backend()` stoppt nur ein Backend, das `ChlorosLocal` selbst gestartet hat), und `GET /api/status` benennt die Einheit als `service`. Starten Sie es mit `sudo systemctl restart chloros-backend@$USER.service` neu. Bei der Aktualisierung des Pakets werden die Backend-Dienste während der Installation angehalten, sodass ein laufendes Skript den Kontakt zu seinen Kameras und Sensoren verliert und anschließend alle aktivierten Geräte erneut startet; das Entfernen des Pakets stoppt sie und deaktiviert die aktivierten Geräte. Es läuft jeweils nur ein Backend, aktivieren Sie daher nur eine Einheit: Eine zweite Einheit weigert sich zu starten und bleibt bei `failed` (Exit-Status 98, kein Neustart), und die Meldung `systemctl status` erklärt den Grund dafür. Um zwischen den Einheiten zu wechseln, deaktivieren Sie diejenige, die Sie nicht verwenden.</id></sn>

### Schritt 2 – Installieren Sie das Python-SDK

**Das Chloros-Installationsprogramm enthält ein passendes SDK-Wheel.** Jedes Windows-Installationsprogramm und jede Linux-.deb-Datei legt eine Datei namens „`chloros_sdk-X.Y.Z-py3-none-any.whl`“ auf der Festplatte ab, die genau mit der GUI-/CLI-/Backend-Version übereinstimmt. Sie müssen nicht ständig PyPI im Auge behalten, um auf dem neuesten Stand zu bleiben.

#### Windows

Das Installationsprogramm führt „`pip install`“ automatisch mit dem mitgelieferten Wheel unter Verwendung Ihres System-Pythons aus (vorzugsweise der Launcher „`py.exe`“, andernfalls „`python -m pip`“). Es sind keine weiteren Schritte erforderlich – `import chloros_sdk` funktioniert nach erfolgreicher Installation in Ihrer Python-Umgebung. Falls kein Python auf dem Rechner installiert ist, überspringt das Installationsprogramm diesen Schritt im Hintergrund, und die GUI sowie die CLI funktionieren weiterhin.

#### Linux (.deb)

Beide .deb-Pakete (x86_64 und Jetson) installieren das SDK für das System `python3`, sodass `import chloros_sdk` nach der Installation ohne pip und ohne Netzwerkverbindung funktioniert. Die Pakete (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` und die Projektlader) werden in `/usr/lib/python3/dist-packages` integriert, und der Befehl „`chloros-mcp`“ des SDKs (der [MCP-Server](#mcp-server-ai-assistants)) in „`/usr/bin`“; alle gehören zum Paket: Ein Upgrade ersetzt sie, und das Entfernen des Pakets entfernt sie.

Die eigenen Anforderungen des SDKs, `python3-numpy` und `python3-requests`, sind **empfohlen, aber nicht erforderlich** (die GUI, die CLI und das Backend sind in sich geschlossen): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` auf einem Jetson) installiert sie, `dpkg -i` hingegen nicht. Die Installationsausgabe gibt die von `python3` importierte SDK-Version an, sowie alle fehlenden Abhängigkeiten sowie ein `chloros_sdk` an einer anderen Stelle im Pfad von `python3`, das die Kopie des Pakets überschreibt (z. B. eine zuvor mit `pip install --user` installierte Version).

In einer virtualenv-Umgebung, die keinen Zugriff auf die Systempakete hat, installieren Sie das mitgelieferte Wheel darin:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Bei Jetson-Bereitstellungen mit Air-Gap erfolgt dies vollständig offline – das SDK und das Wheel befinden sich bereits auf der Festplatte.

#### Öffentliches PyPI

Für reine pip-Hosts (kein Chloros-Desktop-Paket installiert; das SDK steuert dann ein Backend auf einem anderen Rechner an, siehe [Remote-Backend-Modus](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI enthält die neueste stabile Version. Achten Sie darauf, dass das SDK-SDK dieselbe Version wie das von ihm angesteuerte Chloros-Backend hat (siehe [Versionierung & Kompatibilität](#versioning--compatibility)). Der DAQ-M-Helper `DLS` benötigt außerdem `bleak`: `pip install "chloros-sdk[camera]"` (oder `pip install bleak`).

#### Überprüfen

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+-Abonnement erforderlich.** Alle SDK-Aufrufe erfordern eine aktive Chloros+-Anmeldung. Führen Sie `chloros-cli login user@example.com 'YourPassword'` einmal pro Rechner aus; die Anmeldedaten werden in `~/.chloros/` zwischengespeichert.

### Benötige ich das Desktop-Paket?

Das pip-Paket allein reicht für die meisten Arbeitsabläufe **nicht** aus. Hier sehen Sie, was die einzelnen SDK-Oberflächen benötigen:

| SDK-Oberfläche | Benötigt das Desktop-Paket? | Warum |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ja**(um eines zu starten)**/ Nein** (ein Backend ist bereits erreichbar) | Verwende ein laufendes Backend (das der Desktop-App oder eines durch ein anderes Skript gestarteten); andernfalls starte das installierte automatisch: `/usr/lib/chloros/chloros-backend` (Linux) oder `<install folder>\backend\chloros-backend.exe` (unter Windows; standardmäßig `C:\Program Files\Chloros\backend\`, ein benutzerdefinierter Installationsordner wird aus den Daten des Installationsprogramms gelesen). Übergeben Sie `backend_exe=`, um es selbst zu benennen. |
| `chloros-mcp` ([MCP-Server](#mcp-server-ai-assistants)) | **Ja**(um einen zu starten)**/ Nein** (ein Backend ist bereits erreichbar) | Basiert auf `ChlorosLocal`, entspricht also der Zeile oben. |
| `read_image_audit_tags` | **Nein** | Liest die Metadaten des verarbeiteten Bildes aus (nur Python-Standardbibliothek). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ja**(lokal)**/ Nein**(remote) | Reine HTTP-Clients über das Backend. Lokales Backend → Desktop-Paket erforderlich. Remote-Backend → `backend_url=`**über einen Tunnel** (siehe Remote-Backend-Modus – mitgelieferte Backends binden nur an Loopback). |
| `ChlorosProject` / `open_project` | **Ja** | Ruft gespeicherte Projekte über das Backend ab. |
| Direkte LATTICE-Klassen (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ja** | Benötigt die Arena-SDK-Laufzeitumgebung, die im Desktop-Paket enthalten ist, sowie OpenCV (`cv2`) in Ihrer Python-Umgebung, was weder vom Desktop-Paket noch vom SDK installiert wird (`pip install opencv-python`). Andernfalls entspricht `CAMERA_AVAILABLE` beim Import `False`. `DLS` benötigt außerdem `bleak`. |

### Remote-Backend-Modus (nur pip-Host, über Tunnel)

> **Das installierte Backend ist über das LAN nicht erreichbar.** Es lauscht
> ausschließlich auf dem Loopback-Port (`127.0.0.1` / `::1`), daher kann `backend_url="http://<lan-ip>:5000"`
> **ein installiertes Chloros nicht erreichen**. Um ein Backend auf einem anderen
> Rechner zu betreiben, leiten Sie dessen Loopback-Port selbst weiter und richten Sie das SDK auf den
> Tunnel aus:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Headless-/CI-/Robotik-Hosts können einen Rechner mit der vollständigen Desktop-Installation als „Chloros-Server“ und `pip install chloros-sdk` überall sonst betreiben – der Transport zwischen ihnen erfolgt jedoch über den oben beschriebenen, vom Benutzer eingerichteten Tunnel und nicht über eine direkte LAN-URL.

> **`ChlorosLocal` auf einem reinen Pip-Host.** `ChlorosLocal(backend_url=BACKEND)` verbindet sich mit einem Backend, das es ohne Desktop-Installation erreichen kann: Es sucht nur dann nach einer lokalen Backend-Binärdatei, wenn es eines STARTEN muss (keine Antwort und `auto_start_backend=True`). Bis Version 1.2.0 suchte es zuerst nach der Binärdatei und löste auf einem reinen pip-Host den Fehler `ChlorosBackendError` („Chloros-Backend nicht gefunden…“) aus, selbst wenn ein Backend erreichbar war. Die von Ihnen übergebenen Ordner (`process_folder`, `import_images`) angeben, werden auf diesem Rechner überprüft und vom Backend gelesen; bei einem Remote-Backend müssen sie daher auf beiden Seiten denselben Ordner bezeichnen, beispielsweise ein freigegebenes Laufwerk.

### DAQ-only-Workflow (reiner Pip-Host)

`pip install chloros-sdk` allein kann keinen DAQ-Sensor ansteuern: Ein DAQ-Sensor wird immer über ein Chloros-Backend mit `connect_daq_sensor()` angesteuert:

- **Sensor auf diesem Rechner:** Installieren Sie hier das Chloros-Desktop-Paket. `connect_daq_sensor()` startet sein Backend, falls noch keines läuft.
- **Sensor auf einem anderen Rechner:** Dieser Rechner benötigt das Desktop-Paket; greifen Sie über den Tunnel im [Remote-Backend-Modus](#remote-backend-mode-pip-only-host-via-tunnel) auf dessen Backend zu. Das Backend öffnet den Sensor auf seinem eigenen Rechner, sodass ein DAQ-U an diesen Rechner angeschlossen ist, sich ein DAQ-M in dessen Bluetooth-Reichweite befindet und ein DAQ-E in einem Netzwerk ist, das er erreichen kann.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

In beiden Fällen erzwingt das Backend den „Chloros+“-Plan: Melden Sie sich auf dem Rechner an, auf dem das Backend läuft (`chloros-cli login` oder die Desktop-App), mit einem beliebigen kostenpflichtigen Tarif (Copper oder höher). Ein abgemeldetes Backend und ein Tarif unterhalb von „Copper“ lösen beide den Fehler „`ChlorosConnectError`“ aus, der den Grund des Backends enthält (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Schnellstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## API-Index der obersten Ebene

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Bildverarbeitung — `ChlorosLocal`

Die übergeordnete Pipeline-Klasse. Startet das Backend bei der Instanziierung (sofern keines antwortet und `auto_start_backend=True`), erstellt und konfiguriert Projekte, überwacht den Fortschritt und gibt Zusammenfassungen nach Abschluss der Ausführung zurück.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Methoden

| Methode | Beschreibung |
| --- | --- |
| `create_project(project_name, camera=None)` | Erstellt ein neues Projekt (optional mit einer Kameravorlage wie `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Ein gespeichertes Projekt öffnen (dessen Ordner oder dessen Namen im Projektordner) und es zum geöffneten Projekt machen. Die Pfade eines verschobenen Projekts werden auf den Ordner umgeleitet, in dem es sich nun befindet; `relink` benennt den Ordner, in dem sich die anderen fehlenden Dateien nun befinden. Gibt die Antwort des Backends mit `source_files` zurück; löst `ChlorosProjectFilesMissingError` aus, wenn keine gefunden werden kann. Siehe **Ein verschobenes Projekt** weiter unten. |
| `import_images(folder_path, recursive=False)` | Importiert RAW-/TIF-/JPG-/DNG-Bilder **sowie `.daq` Lichtsensoraufzeichnungen**. Gibt `count` (Bilder) und `scan_count` (Aufzeichnungen) zurück. Es wird nur eine Warnung ausgegeben, wenn der Ordner beides nicht enthält. |
| `export_light_sensor(daq=True, csv=True)` | Schreibt kalibrierte `.daq`- und `.csv`-Dateien für jede Lichtsensoraufzeichnung des Projekts in `<project>/Light Sensor/`. Siehe [Lichtsensoraufzeichnungen](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Verarbeitungsregler einstellen. `indices` sind Voreinstellungsnamen (`NDVI`, `GNDVI`, `OSAVI`, …), die jeweils für die Bänder ihrer jeweiligen Kamera zum Projekt hinzugefügt wurden; ein Name, der keine Voreinstellung ist oder für den die Bänder nicht ausreichen (`NDRE` benötigt RE und NIR, was keine einzelne Kamera bereitstellt), erhält die Bezeichnung „`UserWarning`“ und kein Indexbild. „`daq_log_path`“ bezeichnet eine Aufzeichnung des Lichtsensors (eine `.daq`- oder `.csv`-Datei oder einen Ordner mit solchen Dateien), mit der jeder Einzelbild mit abgeglichen wird – sowohl bei LATTICE als auch bei Survey3 –, anstatt der mit den Bildern importierten Aufnahmen; Ein Pfad, der fehlt oder keine Aufzeichnung enthält, wird angegeben, und es werden die importierten Aufzeichnungen verwendet. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Führt die Pipeline aus. Gibt `{"status": "complete", "async": False}` zurück, sowie einen `summary`-Schlüssel, sofern das Backend einen bereitstellt – siehe [Zusammenfassung und Hinweise nach dem Lauf](#post-run-summary--hints) — `export_check_failure`, wenn die Exporte des Laufs nicht verifiziert werden konnten, und `image_failure`, wenn ein Image verloren gegangen ist. `wait=False` gibt `{"status": "started", "async": True}` auf einmal zurück. |
| `get_config()` / `get_status()` / `status()` | Backend-Status überprüfen. |
| `logout()` | Abmelden: Löscht die zwischengespeicherten Anmeldedaten und gibt den Geräteslot dieses Rechners frei (`device_deregistration` im Ergebnis). Ein laufender Verarbeitungsdurchlauf wird nicht gestoppt: Er wird in dem Modus beendet, in dem er gestartet wurde (`run_in_progress: true` und `run_mode` im Ergebnis), und der nächste Durchlauf erfolgt im Standardmodus. |
| `shutdown_backend()` | Das Backend wird gestoppt, wenn diese Instanz es gestartet hat; ein Backend, das bereits läuft, wird nicht angefasst. Das Verlassen eines `with`-Blocks oder die Garbage Collection der Instanz führen nicht zum Stopp des Backends: Das Backend läuft für das nächste Skript weiter und beendet sich selbst nach 30 Minuten, wenn es nicht mehr genutzt wird. Das Backend wird zum Herunterfahren aufgefordert, gibt daher zunächst seine Kameras und Lichtsensoren frei und beendet alle laufenden Aufzeichnungen; es wird erst beendet, wenn es 10 Sekunden später noch läuft. |
| `discover_cameras()` | LATTICE-Kameras **über das Backend dieser Instanz** (`/api/camera/discover`). Gibt eine Liste von Dictionaries zurück (`serial`, `model`, `ip`, …) – in derselben Struktur, wie sie die GUI und die CLI sehen. Leere Liste, wenn keine gefunden wurden oder das Backend nicht erreichbar ist. |
| `camera_capture(output_dir, format="tiff", **settings)` | Nimmt ein einzelnes Bild**über das Backend**auf (wird automatisch über dieses Handle gestartet), sodass es die gleiche Vorbereitung erhält wie die GUI/CLI (Standard: 12 Bit, Pool-Wiederverwendung, eingebettete Kalibrierungsmetadaten). Das Ziel mit `serial=` oder `device_index=` auflösen; übergeben Sie `exposure`/`gain`/`pixel_format`/`preset` als `**settings`. Gibt das alte Metadaten-Dict zurück (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Liefert überlagerte Vorschau-Frames aus einer gepoolten Kamera – schlanker MJPEG-Client über die `/api/camera/<serial>/stream-annotated`-Route des Backends (Zebra / Raster / Fadenkreuz / Histogramm / Peaking / serverseitig gezeichnete Markierung). `decode=True` liefert BGR-Arrays und benötigt OpenCV (`cv2`), das vom SDK nicht installiert wird; `False` liefert rohe JPEG-Bytes. Auch projektbezogen als `ChlorosProject.stream(overlays=True)` erreichbar. |

**Ein laufender Prozess behält sein Projekt bei.** Während ein Verarbeitungslauf läuft – sei es dieses Skripts, der App, von `chloros-cli process` oder von `chloros-mcp` – weigert sich das Backend, ein Projekt zu erstellen, zu öffnen, zu importieren oder dessen Verarbeitungseinstellungen zu ändern (HTTP-Fehler 409 `PROCESSING_IN_PROGRESS`): `create_project()` und `import_images()` lösen `ChlorosProcessingError` aus, und `configure()` löst `ChlorosConfigurationError` aus, und `process()` mit einem anderen Modus als dem aktuellen löst `ChlorosProcessingError` aus (ein Lauf behält den Modus bei, in dem er gestartet wurde), jeweils mit der Meldung des Backends. Eine im Projekt gespeicherte Erfassung wird dann nach Abschluss des Laufs hinzugefügt (die Antwort `project_add` besagt dies). Warten Sie, bis der Lauf beendet ist (`get_status()`), und fahren Sie dann fort.**Ein verschobenes Projekt.** Das `project.json` eines Projekts benennt jede Quelldatei (jedes Bild und jede Lichtsensoraufzeichnung) mit ihrem vollständigen Pfad, sodass beim Verschieben des Projektordners – oder des gesamten Projektordners auf ein anderes Laufwerk – die Dateinamen weiterhin auf den alten Speicherort verweisen. Bei jedem Öffnen wird selbstständig korrigiert, was möglich ist: Eine Datei, die sich zuvor im Projektordner befand, wird im aktuellen Ordner des Projekts gesucht, und jeder dort gefundene Pfad wird neu verweist und gespeichert (`project.json` wird zunächst als `project.json.before-relink-<date>-<time>` gesichert). Ein Pfad wird immer nur auf eine Datei umgeleitet, die dort vorhanden ist. Dateien, die sich außerhalb des Projektordners befanden – eine Hub-Sitzung, ein „Add Folder“-Import – werden nur mit `relink=<the folder they are in now>` (oder einem darüber liegenden Ordner) gefunden. `open_project()` gibt `source_files` zurück: `total`, `found`, `missing`, `missing_roots` (die Ordner, in denen sich die fehlenden Dateien befanden), `missing_outside_project` und `relink` (was neu verweist, von wo nach wo). Ein Projekt, in dem einige Dateien fehlen, wird mit den übrigen Dateien geöffnet; eines, in dem keine Dateien gefunden wurden, wird nicht geöffnet, und `open_project()` löst `ChlorosProjectFilesMissingError` aus (`.missing_roots`, `.audit`; HTTP-Fehler 409 `PROJECT_SOURCES_MISSING`). Die CLI-Fehler `project open --relink` und `chloros-mcp``open_project` (`relink_folder`) der CLI tun dasselbe.

Verwendung als Kontextmanager für garantierte Bereinigung:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Lichtsensoraufzeichnungen – kalibriert `.daq` + `.csv`

Ein DAQ-U / DAQ-M / DAQ-E kann **ohne** sein Kalibrierungs-Bundle aufgezeichnet werden. Das ist
das, was die öffentlichen [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) standardmäßig tun: Sie schreiben die rohen Sensorwerte und versehen die
Datei mit einem Zeitstempel, sodass Chloros zunächst die Werkskalibrierung des Sensors **über die Seriennummer** – zunächst aus dem lokalen Cache, dann aus der MAPIR-Cloud – und wendet sie beim Import an.

Chloros schreibt das Ergebnis als zwei Produkte pro Aufzeichnung unter
`<project>/Light Sensor/` zurück:

| Produkt | Was es ist |
| --- | --- |
| `<name>_calibrated.daq` | Das wiederverarbeitbare Archiv – dasselbe Schema wie bei einer Live-Aufzeichnung, nun mit Angabe des Bundles, das es erzeugt hat. Ein erneuter Import führt **nicht** zu einer zweiten Kalibrierung. |
| `<name>_calibrated.csv` | Spektrale Bestrahlungsstärke in W/m²/nm auf dem sensoreigenen Wellenlängengitter, eine Zeile pro Messwert, plus photometrische Spalten (Gesamtleistung, photopische/skotopische Lux, PPFD und deren Aufteilung in Blau/Grün/Rot, Spitzenwellenlänge). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Nur sensoren ohne Bundle (DAQ-A).** Rohdaten der spektralen Sensorwerte – *keine* Bestrahlungsstärke. Siehe unten. |

`<name>` ist der Dateiname der jeweiligen Aufzeichnung. Zwei Aufzeichnungen in unterschiedlichen Ordnern können
einen gemeinsamen – pro Aufnahme erfasste Messwerte (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), die in derselben
Millisekunde von zwei Kameras gespeichert wurden – und jede erhält dennoch ihre eigenen Ergebnisse: Die erste behält
`<name>`, die nächste fügt die Ordner, die sie unterscheiden, **nach** dem Namen hinzu
(`<name>__<capture>_daq_calibrated.daq`), sodass Datum und Uhrzeit der Aufzeichnung weiterhin an erster Stelle stehen.

`process()` führt diesen Export als einen seiner Schritte durch. Es benötigt **keine** Bilddaten:
Ein eigenständig geflogener Lichtsensor ist ein vollwertiger Arbeitsablauf, und ein solches Projekt enthält konstruktionsbedingt
keine Bilder.

**DAQ-A-Aufzeichnungen werden als Rohzählwerte exportiert.** Die DAQ-A-Familie stammt aus der Zeit vor dem seriennummernbasierten
Bundle-System und verfügt über kein abzurufendes Bundle – sie wird stattdessen vor Ort anhand eines
Reflexionsziels kalibriert, weshalb sie nie eines benötigte. Diese Aufzeichnungen werden
unter einem `_raw`-Stamm statt unter `_calibrated` exportiert: ein anderer Dateiname statt eines Flags
innerhalb der Datei, da die Angabe den Versand per E-Mail als reiner Name überstehen muss. Der
`.csv`-Header gibt `raw spectral sensor counts (NOT irradiance)` an und warnt, dass die
Werte **innerhalb** der Datei vergleichbar sind – genau das, wofür die zielbasierte Kalibrierung
– und nicht sensorübergreifend. Die leistungsabhängigen photometrischen Spalten (Gesamtleistung,
photopische/skotopische Lux, PPFD) werden als **NULL** zurückgegeben, anstatt aus Zählwerten integriert zu werden.

Ein DAQ-U / DAQ-M / DAQ-E, dessen Bundle einfach nicht abgerufen werden konnte, wird weiterhin **übersprungen**,
nicht im Rohformat geschrieben: In diesem Fall existiert das Bundle, und „erneut verbinden und neu verarbeiten“ ist ein echter Ratschlag.

Ältere **v1.01 / v1.02**-Aufzeichnungen (ein DAQ-A-SD schreibt diese) enthalten keine Epochenangabe pro Messwert,
sondern nur den Schreibzeitpunkt der Datei. Der Bild↔Downwelling-Matcher lehnt sie weiterhin ab – der Abgleich eines
Frame mit einer Schreibzeit abzugleichen, würde zu unsichtbaren Fehlern führen – aber der Exporter liest sie ein, und die
Datei CSV gibt „`clock=daq_created_on`“ aus, sodass das Produkt angibt, auf welcher Uhr es basiert.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Eine Aufzeichnung, deren Kalibrierungsbündel nicht abgerufen werden kann (Offline-Modus oder ein Sensor ohne
Kalibrierung in der Datei), wird unter `skipped` **mit Angabe des Grundes** gemeldet. Sie wird niemals
als „kalibrierte“ Datei mit Rohzählwerten ausgegeben – stellen Sie eine Verbindung zum Internet her und
führen Sie den Vorgang erneut aus, dann wird der Export abgeschlossen.

### Fortschritts-Callbacks

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Zusammenfassung und Hinweise nach Abschluss

Nach Abschluss ruft `process()` die Datei `GET /api/processing-summary` ab und fügt den Hauptteil als `result["summary"]` an. Das Abrufen erfolgt nach bestem Bemühen und blockiert niemals eine erfolgreiche Rückgabe – sollte die Zusammenfassung nicht verfügbar sein, greift `process()` auf die einfache `{"status": "complete", "async": False}`-Form zurück. Jeder Eintrag in `summary["hints"]` – vollständige Sätze mit der vorgeschlagenen Abhilfe, z. B. warum ein Lauf keine Ausgabe erzeugt hat — wird ebenfalls als Python-`UserWarning` ausgegeben, sodass Läufe ohne Ausgabe selbstdiagnostisch sind, selbst wenn Sie das Wörterbuch nie überprüfen:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` ist der maschinenlesbare Teil:

| Schlüssel | Was gezählt wird |
| --- | --- |
| `models` | Kameragruppen im Lauf. |
| `images_in_groups` | Quellbilder über diese Gruppen hinweg. |
| `targets_found` | Erkannte Reflexionsziele. |
| `images_calibrated` | Bilder, für die der Durchlauf ein Reflektanzprodukt erstellt hat. |
| `images_failed` | Bilder, die fehlgeschlagen sind und nicht exportiert wurden. |
| `exported_files` | **Bildproduktdateien , die der Durchlauf erstellt hat**: jedes Produkt (Debayering, Vorschau, Strahlungsintensität, Reflektanz, Index, Ziel-Renderings) in jedem Format. |
| `reflectance_files` | Die Reflektanzprodukte unter ihnen (`Reflectance_Calibrated_Images`), unabhängig davon, ob sie durch ein Ziel oder die kalibrierte Bestrahlungsstärke des Lichtsensors (DAQ) ermittelt wurden. Ein Lauf ohne Ziel, bei dem dennoch Reflektanzdaten geschrieben wurden, weist in einem Hinweis darauf hin; nur bei einem Lauf, bei dem keine Reflektanzdaten geschrieben wurden, wird gemeldet, dass die Reflektanzausgaben fehlen. |
| `clamp_masks_written` | Die [Unmix-Clamp-Masken](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) – sind Begleitdaten, keine Ergebnisse, daher nicht in `exported_files` enthalten (die CLI-Zeile zählt sie ebenfalls separat). |
| `captures_left_out` | Rohbilder im Ordner `captures` des Projekts, die bei diesem Durchlauf nicht aufgenommen wurden. Ein Durchlauf wartet beim Start auf das, was zuvor erfasst wurde; daher ist dieser Wert 0, es sei denn, während des Durchlaufs wurde eine Erfassung gespeichert oder die Wartezeit ist abgelaufen. `summary["captures_left_out_note"]` gibt an, wie mit ihnen verfahren werden soll. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Aufzeichnungen des Lichtsensors, die absichtlich separat gezählt werden — sie stammen aus einer anderen Phase und existieren auch bei Durchläufen, bei denen überhaupt keine Bilder vorhanden sind; würden sie mitgezählt, würde ein reiner DAQ-Durchlauf so aussehen, als hätte er Bilder exportiert. |

Daneben: `summary["output_dirs"]` (jedes Verzeichnis, in das geschrieben wurde),
`summary["light_sensor_export"]`, `summary["stopped"]` (wahr, wenn der Benutzer den
Lauf unterbrochen hat, sodass Teilzählungen nicht als abgeschlossener Lauf gewertet werden, bei dem zu wenig produziert wurde) sowie
`summary["groups"]` (die Aufschlüsselung nach Gruppen).

`exported_files` wird von der Pipeline **beim Schreiben** erfasst und nicht nachträglich aus den
Image-Objekten des Projekts ausgelesen. Die Parallel- und GPU-Strategien erstellen ihre eigenen Image-Objekte
(in Worker-Unterprozessen für die GPU-Pfade), sodass der alte Scan
`0 file(s) written` für jeden solchen Durchlauf und gab dann den Hinweis „zero-exports“ aus – bei Durchläufen,
bei denen alles funktioniert hatte. Wenn Sie Ihr Skript auf diese Nummer ausrichten, meldet ein fehlerfreier paralleler Durchlauf nun
eine Zählung ungleich Null.

Überspringmeldungen des „Light-Sensors“ geben den Grund an, den der Reader tatsächlich für jede Datei festgestellt hat – ein
unlesbares Schema, ein fehlendes Bundle, ein Schreibfehler – **dedupliziert**, sodass zwanzig Dateien,
die aus einem Grund übersprungen wurden, als ein Grund ausgewiesen werden und nicht als zwanzig Wiederholungen desselben.

> **`process()` wird nicht ausgelöst, wenn ein Lauf keine Bilder erzeugt.** Dies ist eine der beiden Stellen, an denen sich das SDK und
> die CLI bewusst unterscheiden: `chloros-cli process` behandelt „Produkte wurden angefordert, es wurden keine
> geschrieben“ als Fehler und beendet den Lauf mit einem Wert ungleich Null, während das SDK normal zurückkehrt und den
> Zustand über `summary` / hints. Sollte Ihre Pipeline bei einem leeren Durchlauf anhalten, überprüfen Sie dies
> selbst – untersuchen Sie `summary` (oder zählen Sie die Dateien im Projektordner), anstatt sich darauf zu verlassen, dass
> keine Ausnahme auftritt. Die üblichen Ursachen sind ein Eingabeordner, der nicht als
> Aufzeichnung erkannt wurde, sowie Produkte, die als für die vorhandenen Kameras nicht anwendbar übersprungen wurden (z. B. Radiance von reinen RGB-
> Kameras).
>
> `process()` wird auch nicht ausgelöst, wenn die Exporte eines Durchlaufs nicht verifiziert werden konnten: Es wird eine Warnung ausgegeben und
> `result["export_check_failure"]` gesetzt. Die Produkte werden geschrieben. Um den Durchlauf in diesem Fall zu stoppen, überprüfen Sie diesen Schlüssel. Ein Lauf, bei dem
> ein Bild verloren gegangen ist, verhält sich ebenso: Es wird eine Warnung ausgegeben und `result["image_failure"]` gesetzt, bei dem `chloros-cli process`
> mit einem Wert ungleich Null beendet wird.

### Hilfsfunktionen

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Jeder Lauf läuft innerhalb seines eigenen `with ChlorosLocal(...)`; ein Backend, das er starten musste, läuft weiter, wenn er zurückkehrt, für den nächsten Aufruf, und beendet sich selbst nach 30 Minuten, wenn es nicht genutzt wird. Keine der beiden Funktionen akzeptiert `auto_start_backend`, `radiometric_output` oder `daq_log_path`; verwenden Sie stattdessen `ChlorosLocal` und `configure()` für diese Fälle. `process_lattice_capture()` akzeptiert weder `camera`, `debayer` noch `ppk`: Es überspringt die Panel-Erkennung und verwendet den Standard-Debayer.

### Unterstützte Werte

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrische Ausgabe (LATTICE-Multispektral-Pipeline)

Die LATTICE-Multispektral-Exportebene (M3C/M3M) der `process`-Pipeline – `reflectance` (Standard), `radiance`, `sensor-response` oder `all` (jeder anwendbare Modus pro Bild) – legt die **„Export-Strahlungsintensität“**und**„Export-Reflexionsgrad“**** fest: `radiance` exportiert nur Radiance, `reflectance` nur Reflectance, `sensor-response` keines von beiden (nur die debayered- und Vorschau-Produkte), `all` beide. Die Debayering- und Vorschau-Produkte können auf dieser Ebene nicht geändert werden. `configure()` verfügt über ein eigenes Schlüsselwort dafür:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Der erweiterte Ausweg – das Schreiben der Produkte über `custom_settings` – funktioniert weiterhin, aber denken Sie daran, dass dadurch der gesamte Einstellungsblock ersetzt wird (siehe die Warnung unten). Schreiben Sie die `"Export <product>"`-Schlüssel: Die projekt-eigenen `"Export radiance"`- / `"Export reflectance"`-Einstellungen eines Projekts haben Vorrang vor dem älteren Schlüssel „`"Radiometric output"`“, sodass dieser Schlüssel allein nichts ändert.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (Standard) dividiert die Kamerastrahlung durch die **zeitstempelabgeglichene DAQ-Downwelling-Strahlung**, die automatisch aus einem aufgezeichneten `.daq` (DAQ-U/M/E)**oder einem nativen DAQ-M-`.csv`**, der zusammen mit den Bilddaten vorliegt; fehlende kameraspezifische oder DAQ-Kalibrierungspakete werden bei der ersten Verwendung**automatisch von AWS abgerufen**. Die CLI stellt dies gemäß-Typ-Produktschalter auf `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Wessen Messwert ist das?**Ein `.daq`, den eine Aufnahme speichert, ist der Messwert der Kamera(s), die er beschreibt – benannt in einem `<file>.daq.cameras.json`-Sidecar daneben (Einzel-, synchronisierte und Array-Aufnahmen; bei einer Raw-Serienaufnahme werden die Messwerte jedes Elements in ihrem Manifest benannt) – und wird**nur mit Bildern dieser Kameras** abgeglichen. Eine eigenständige Kamera ohne eigenen Messwert nutzt daher nicht mehr die DAQ einer anderen Kamera; binden Sie ihr eine DAQ zu oder zeichnen Sie eine auf. Die eigene Aufnahme einer DAQ (`chloros-cli daq pool-record` oder eine in der GUI gestartete Aufzeichnung) sowie jede `.daq`-Datei ohne diese Sidecar-Datei (einschließlich älterer Aufnahmen) dient weiterhin allen Kameras.**OCN-Kameras im Sonnenlicht.** Jedes OCN-Kalibrierungspaket wird unter schmalbandigem Licht erstellt – eine LED pro Band –, während sich die Bänder im Sonnenlicht vermischen. Chloros 1.3.0 wendet nach der Bandentmischung eine flottenweite Sonnenkorrektur an, wodurch die OCN-Produkte für Radianz, Reflektanz und Index bei Sonnenlicht verbessert werden. Sie läuft überall dort, wo die Entmischung stattfindet – `process()`, Aufnahmen, `grab()` / `frame_stream()` und die Live-Ansicht – für OCN-Pakete, die dies unterstützen; RGN-Kameras bleiben unverändert. Auch mit einer früheren Version aufgenommene OCN-Bilder erhalten diese Korrektur, wenn sie auf einem Computer verarbeitet werden, auf dem die Kalibrierung gespeichert ist, mit der sie aufgenommen wurden (Chloros behält die Kalibrierung einer Kamera bei, sobald diese angeschlossen wurde); andernfalls werden sie wie bisher verarbeitet. Die OCN-Werte für Strahlungsintensität, Reflektanz und Index weichen daher von denen der Version 1.2.0 ab.

> `custom_settings` **ersetzt** den gesamten Block der berechneten Einstellungen (es umgeht bewusst die anderen Schlüsselwörter und die Validierung von `configure()`). Wenn Sie es verwenden, fügen Sie jeden für Sie relevanten `Project Settings`-Schlüssel ein, wie im obigen Beispiel.

#### Auslesen von Indexpixeln (NDVI und die anderen Vegetationsindizes)

**Ein ganzzahliges Indexraster ist eine bildspezifische Kontraststreckung, keine −1…1-Zuordnung.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` und `JPG (8-bit)` – einschließlich des 8-Bit-TIFFs, als das eine 8-Bit-Aufnahme gespeichert wird – speichern

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

wobei `IndexMin` / `IndexMax` **die niedrigsten und höchsten endlichen Indexwerte dieses Frames** sind. DN 0 ist nicht −1, und der Vollbereich ist nicht +1.

Chloros versieht beide Endpunkte mit einem Stempel, damit die Abbildung invertierbar bleibt:

| Tag | Bedeutung |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — die unten stehende Konvention |
| `Chloros:IndexMin` | der als DN 0 gespeicherte Indexwert |
| `Chloros:IndexMax` | der als Vollskala gespeicherte Indexwert |

Lies sie ein und kehre sie um, wobei du den gespeicherten DN zunächst auf den Maximalwert des gespeicherten Datentyps normierst (0…1), sodass ein Ausdruck jedes Ganzzahlformat abdeckt:

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

Zum Lesen wird kein `-config pix4d.config` benötigt – exiftool liest diese XMP-Eigenschaften von selbst. Die Konfiguration ist nur zum *Schreiben* erforderlich.

> **`IndexMin == IndexMax`** bedeutet, dass jedes Pixel dieses Bildes denselben Indexwert hatte. Das Raster ist in der Mitte der Skala ausgefüllt und es gibt nichts, durch das man teilen müsste – nimm den Endpunkt.

> **Keine `Chloros:Index*`-Tags bedeuten, dass keine Zuordnung angegeben ist.** Ein LUT-Export ist ein koloriertes Rendering, dessen Pixel Farbverläufe sind, die auf Ihr LUT-Schwellenwertfenster abgestimmt sind, und ein Bild ohne Pixel mit endlichem Index hat keinen anzugebenden Bereich. Gehen Sie in keinem der beiden Fälle von einer Zuordnung aus – es gilt dieselbe Regel wie bei einem fehlenden `PixelScale`.**Da die Streckung bildspezifisch ist, sind zwei Bilder eines Fluges hinsichtlich des DN nicht vergleichbar**– jedes hat seine eigenen Endpunkte, sodass derselbe NDVI-Wert in verschiedenen Bildern auf unterschiedliche DN-Werte fällt. Für quantitative Auswertungen über einen Flug hinweg (zonale Statistiken, feste Schwellenwerte, Zeitreihen) exportieren Sie**`TIFF (32-bit, Percent)`**: Indexraster werden dort als float32-**Indexwerte selbst** geschrieben, ohne dass eine Neuskalierung oder Tags erforderlich sind. Trotz des Namens des Formats sind Indexprodukte nicht ×100 – diese Skalierung gilt nur für Reflektanzwerte vom Typ „Integer“.

#### Unmix-Clamp-Maske – auf welche Pixel ein Band geklemmt wurde

Die 3×3-Spektralentmischung kann ein Band in den negativen Bereich treiben – dort, wo das Licht dieses Band kaum enthält oder wo die Szene außerhalb dessen liegt, was die Entmischungsmatrix der Kamera beschreibt. Da dies kein realer Wert ist, setzt der Export ihn auf 0 — und die Live-Ansicht zeigt ebenfalls 0 an. Ein Band kann bei einigen wenigen Pixeln oder bei fast allen geklemmt werden; die Maske erfasst beides. `Chloros:Band{R,G,B}UnmixClampFraction` gibt an, bei wie viel Prozent eines Bildes dies der Fall war; die Clamp-Maske gibt an, **welche Pixel** betroffen waren:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**– in einem `clamp_masks/`-Ordner**neben** dem Produktordner, niemals darin (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) – für jedes Radiance-, Reflektanz- und Index-Produkt, bei dem mindestens ein Pixel geklemmt wurde. Einzelband-`uint8`, auf dem produkt-eigenen Pixelraster (Ausrichtung, ROI, Binning und die Array-Ausrichtungsverzerrung eingeschlossen). Jede Maske benennt die Kamera ihres Produkts (`Make` / `Model` / fortlaufend), die Aufnahmezeit und die Objektiv-Tags und listet die Bänder auf, für die ihre Bits stehen (`XMP-Camera:BandName`, Bit 0 zuerst); die Bits `ImageDescription` und `Software` kennzeichnen sie als Clamp-Maske, und sie enthält keine Kalibrierung.
- **Bits**: Bit 0 = das Band im R-Kanal des Produkts (`Chloros:BandRName`), Bit 1 = G (`BandGName`), Bit 2 = B (`BandBName`). Ein gesetztes Bit bedeutet, dass der Wert des Pixels in diesem Band auf 0 begrenzt wurde – behandeln Sie dieses Bit und alle daraus abgeleiteten Indizes als wenig zuverlässig.
- **`Chloros:UnmixClampMask`** ist auf jedem Produkt vermerkt, bei dem die Entmischung durchgeführt wurde: der Pfad der Maske relativ zum eigenen Ordner des Produkts (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, immer durch `/`-getrennt – fügen Sie ihn an den Bildordner an, um die Maske zu öffnen) oder `none`, wenn kein Pixel begrenzt wurde. Fehlt die Kennzeichnung, wurde der Unmix für das Produkt nie durchgeführt (RGB- oder Mono-Kameras, Rohdaten/Debayering-Ebenen).
- Die Aufnahmeresultate enthalten es als **`clamp_mask`** des Frames (den Pfad der Maske oder `None`, wenn nichts geklemmt wurde) — in den Ergebnissen `CameraSession.capture()`, `ArraySession.capture()`'s `CaptureResult`-Frames und den HTTP-Aufnahmeantworten. Der Offline-Build listet seine Masken als `clamp_masks` auf.
- Der Multiband-Stack eines Live-Index (`/api/camera/cli-array-index` mit `save_multiband`) erhält eine Maske pro Quellkamera im eigenen `clamp_masks/` des Ausgabeordners – siehe die CLI-Referenz'`lattice index`. Ein Video hat keine Maske: Das Ergebnis des Offline-Builds gibt stattdessen den Clamp-Anteil jedes Einzelbildes an.

Da kein Produktordner, egal in welcher Ebene, eine Maske enthält, kann ein Produktordner unverändert – einschließlich aller Unterordner – an die Photogrammetrie-Software übergeben werden, die dann nur die Bilder verarbeitet. Übergeben Sie ihr den Produktordner, nicht den darüber liegenden Ordner. Der eigene Import von „Chloros“ überspringt die Masken.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect für LATTICE-Kameras

Persistente Backend-Sitzungen für Live-Hardware. Es werden dieselben Endpunkte verwendet wie von der GUI, sodass das Verhalten über SDK, CLI und GUI hinweg identisch ist.

### Einzelne Kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Die hier festgelegten Belichtung, Verstärkung und das Ziel werden als Ihre Einstellungen beibehalten: Die automatische Belichtungssteuerung von Chloros (einschließlich der vom DAQ gemessenen Belichtung) lässt sie unverändert. `cam.set_settings(exposure_auto="Continuous")` (oder `gain_auto=`) übergibt die Belichtung (Verstärkung) zurück, und `cam.set_settings(target_brightness_auto=True)` übergibt den Zielwert wieder an die automatische Zielwertsteuerung vChloros.

#### `connect_camera()` Signatur

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Die Funktion kehrt zurück, sobald der Stream gestartet wurde.** Das Backend wartet, begrenzt (etwa 15 s, bis der Stream startet, dann 10 s auf den ersten vollständigen Puffer – nicht auf das erste Vorschaubild, das etwa 1,5 s später folgt), sodass ein `status()` unmittelbar darauf folgt, woraufhin `streaming` angezeigt wird. Das `connect_stream` der Sitzung ist der `stream`-Block der Antwort: `state` `streaming`, `no_frames` (der Stream wurde gestartet und lieferte nichts – das besagt der `note`, und bei einer Kamera mit Jumbo-Paketen soll die Verbindung mit einer Paketgröße von 1500 Byte wiederhergestellt werden), `starting` oder `start_failed`, wobei `waited_s`. `None` für eine Kamera, die bereits verbunden war. Eine Vorschau (`get_camera_preview`, `/api/camera/jpeg/<serial>`), die unmittelbar nach dem Verbindungsaufbau angefordert wird, wartet begrenzt (4 s) auf das erste Vorschaubild der Kamera, anstatt mit „Noch kein Bild verfügbar“; eine Kamera, deren Stream noch nichts geliefert hat (`no_frames` oder für einen Hardware-Trigger aktiviert), antwortet weiterhin sofort.
- **Die Paketgröße kannst du selbst festlegen.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` öffnet die Kamera mit 1500-Byte-Paketen, ohne Abfrage (`chloros-cli lattice cam-connect --packet-size standard` sendet dasselbe).
- **Es darf jeweils nur eine Verbindung zum Kamerapool bestehen**: das Öffnen der Verbindung, die Abfrage der Paketgröße, ein erster Kalibrierungs-Download (ca. 70 s für eine Kamera, die auf diesem Computer neu ist), der Start des Streams. Eine Anfrage, die in der Zwischenzeit eine Kamera benötigt (`read_nodes`, `set_settings`, eine Aufzeichnung, eine zweite Verbindung) wartet höchstens 5 s darauf, schlägt dann mit HTTP-Fehler 503 (`CAMERA_POOL_BUSY`) fehl und nennt die sich verbindende Kamera (`ChlorosConnectError`); frage es in einem Moment erneut. Die Einträge warten nie: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` und `CameraSession.status()` antworten sofort. Eine Kamera, die gerade verbunden wird, wird unter der Pool-Antwort „`connecting`“ (`serial`, `phase` – `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` – sowie `elapsed_s`), und sein `status()` gibt `connecting: True` mit seinem `phase` an; die anderen Kameras werden live ausgelesen, wenn der Pool frei ist, andernfalls wird der Wert aus dem letzten Lesevorgang zurückgegeben (`live: False`).

#### `CameraSession`-Methoden

| Methode | Beschreibung |
| --- | --- |
| `status(timeout=15.0)` | Der Live-Status der Kamera, wie er von `chloros-cli lattice status` ausgelesen wird: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` oder `stopped`), `device` (Modell, Seriennummer, Firmware, Temperatur, `is_mono`) sowie dessen aktuelle Einstellungen. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | GenICam-Knoten lesen; gibt `{nodes, errors, enums, device, was_in_pool, in_array_id}`, sowie `frame_values` und `notes`, während die automatische Belichtung läuft (siehe unten). |
| `set_settings(**kwargs)` | Knoten unter ihrem benutzerfreundlichen Namen schreiben (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Ein `exposure_time` (`gain`) ist eine manuelle Belichtung (Verstärkung): Wird ohne `exposure_auto` (`gain_auto`) geschrieben wird, muss zuvor der Automatikmodus ausgeschaltet werden, damit der Wert nicht von einem laufenden Automatikmodus abgelehnt wird. Das Ergebnis `applied` enthält den von der Kamera aufgenommenen Wert; ein von der Kamera abgelehnter Schreibvorgang befindet sich in `errors` und wird nicht gespeichert. Der Fehler gibt den Bereich an, den die Kamera derzeit akzeptiert (ein Maximum von `ExposureTime` richtet sich nach der Bildrate der Kamera), und ein Automatikmodus, der nur für den abgelehnten Wert ausgeschaltet wurde, wird wieder aktiviert; `restored` listet diesen auf (z. B. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Aufnahme eines **einzelnen** Bild auf. Gibt ein `CaptureResult` zurück: eine Liste von Metadaten-Dictionaries für die Bilder (eines pro gespeicherter Ebene), mit `.skipped`, `.failed`, `.daq_files` und `.daq_matches`. (Die Serien-/Mehrfachbildaufnahme wurde entfernt – rufen Sie `capture()` in einer Schleife auf, wenn Sie eine Serie benötigen.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Aufeinanderfolgende (`interval_s=0`) oder Intervallaufnahmen dieser Kamera, für `count`-Aufnahmen und/oder `duration_s` (mindestens eine, damit der Vorgang beendet wird), so wie es `ArraySession.capture_repeated()` für ein Array tut. Gibt die Liste der `CaptureResult`s zurück; der Vorgang wird nach einer Aufnahme beendet, die einen vollen Speicher meldet. `on_capture(i, result)` wird nach jeder Aufnahme ausgeführt; andere Schlüsselwörter werden an `capture()` weitergeleitet. |
| `disconnect(keep_settings=False, clear_settings=False)` | Freigabe aus dem Pool; die Kamera wird in den Zustand zurückversetzt, in dem sie bei der Verbindung vorgefunden wurde (Trigger-Verkabelung, PTP, Einstellungen); `keep_settings=True` speichert seine Einstellungen stattdessen im eigenen Speicher, `clear_settings=True` setzt diesen Speicher zurück (siehe [Kameraspeicher](#camera-memory--keep_settings--clear_settings)); gibt `camera_memory` zurück, wenn eine der beiden Funktionen angefordert wird. Eine Kamera, die Teil eines Arrays ist, gibt das gesamte Array frei. Keine Aktion, wenn wir an eine bereits geöffnete Sitzung angehängt wurden. Gibt zurück, sobald das Backend die Freigabe bestätigt hat oder wenn nichts mehr freigegeben werden kann (die Kamera ist bereits nicht mehr vorhanden oder es läuft kein Backend); eine Freigabe, bei der eine Zeitüberschreitung aufgetreten ist oder die vom Backend abgelehnt wurde, löst `ChlorosConnectError` aus, da die Kamera möglicherweise noch belegt ist und die nächste Verbindung damit kollidieren würde. Der Ausstieg aus dem Block `with` löst niemals einen Fehler aus. |

Attribute: `serial`, `device` (der Erkennungsdatensatz der Kamera) und `backend_url`.

`capture()`-Exportsteuerung (gleiches Modell wie das Array + GUI):

- `processing` / `levels` — `processing="all"` speichert jeden anwendbaren Exporttyp; `levels=["raw","radiance"]` speichert nur diese (überschreibt `processing`). Lassen Sie beide weg, um die Backend-Standardeinstellung zu verwenden.
- `daq` — Der DAQ-Messwert der verwendeten Erfassung wird standardmäßig als `.daq`-Sidecar gespeichert, einschließlich einer reinen Rohdatenerfassung, sodass der Frame später in Reflektanz/Index umgewandelt werden kann. `daq=False` überspringt ihn, wenn keine Stufe ihn verwendet. `force_daq=True` ist die ältere Schreibweise der Standardeinstellung. Kein DAQ, kein `.daq`.
- `daq_tolerance_ms` – wie weit der Messwert von der Belichtung entfernt sein darf und dennoch innerhalb der Toleranz liegt (Standardwert 500). Es ändert sich lediglich die Bewertung.

#### `read_nodes()` bei automatischer Belichtung – `frame_values` und `notes`

Während `ExposureAuto` (`GainAuto`) läuft, liest der Knoten `ExposureTime` (`Gain`) den **zuletzt geschriebenen Wert** zurück, nicht den aktuellen Status der Kamera. `nodes` behält diesen Knotenwert weiterhin bei. Wenn Sie `ExposureTime` oder `Gain` bei einer angeschlossenen Kamera abfragen, deren Automatikmodus nicht `Off` ist, enthält die Antwort zusätzlich:

- `frame_values` – den Wert, den das neueste Bild der Kamera enthielt, unter dem Namen des Knotens (`ExposureTime` in µs, `Gain` in dB);
- `notes` – ein Satz pro Knoten, in dem beide Werte angegeben sind.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Beide Schlüssel fehlen, wenn der Automatikmodus `Off` ist (der Knoten entspricht dann demtatsächlicher Wert) und bei einer Kamera, die nicht mit dem Backend verbunden ist. Bei einem Array liefert `ArraySession.status()["member_ae"]` die Bildwerte aller Elemente auf einmal.

#### DAQ-gemessene Belichtungsobergrenze (HTTP)

Bei angeschlossenem DAQ-Lichtsensor begrenzt Chloros die Belichtung × Verstärkung einer Kamera anhand des gemessenen Tageslichts, damit ein weißes Referenzfeld nicht übersteuert wird. Die Begrenzung gilt nur, solange sich ein Kalibrierungsziel im Sichtfeld dieser Kamera befindet (innerhalb der letzten 10 Sekunden im Bildausschnitt erfasst); ist kein Ziel im Sichtfeld, greift stattdessen die normale automatische Belichtung der Kamera mit ihrem eigenen Schutz vor Überbelichtung, sodass der Rest der Szene nicht dunkler gehalten wird, als nötig. Es gibt keinen SDK-Wrapper; das Ein- und Ausschalten erfolgt pro Kamera über `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (der POST-Body `{"enabled": false}` deaktiviert die Funktion für diese Kamera). Die Antwort enthält `enabled`, `cap_us` (den Live-Höchstwert, `None` im inaktiven Zustand, auch wenn kein Kalibrierungsziel im Blickfeld ist), `k_exp_g` (die Konstante, mit der die Obergrenze berechnet wird) sowie:

| Feld | Bedeutung |
| --- | --- |
| `k_exp_g_source` | Welche Konstante verwendet wird: `camera` – die dieser Kamera eigene, berechnet aus dem Lab-Punkt in ihrem Kalibrierungspaket (bei RGB-Kameras); `fleet` – die Flottenkonstante, die verwendet wird, wenn das Paket keinen Lab-Punkt enthält; `override` – eine im Backend festgelegte Feldüberschreibung. |
| `k_exp_g_fleet` | Die Flottenkonstante, zum Vergleich. |

### Synchronisiertes Array — `ArraySession` (Smart-Prep)

`connect_array` ist **der empfohlene Einstiegspunkt** für Konfigurationen mit mehreren Kameras. Im Hintergrund wird der gesamte Smart-Prep-Ablauf über die grafische Benutzeroberfläche ausgeführt:

1. **Netzwerkanalyse**(`/api/camera/array/recommend`) – legt den**vollständigen Sensor** (kein Binning, kein ROI-Ausschnitt) und lässt die Bildrate auf das Niveau sinken, das dieser Host und das Netzwerk liefern, projiziert auf die von „Packing Connect“ verwendeten Pixelformate und Packungsarten. Bilder werden niemals verkleinert, um die Bildrate aufrechtzuerhalten: Geben Sie einen kleineren Wert für `width` / `height` oder `binning=2` / `4` übergeben, um Auflösung gegen Bildrate einzutauschen (die`alternatives` listet diese Optionen mit ihren projizierten fps auf).
2. **Automatische Auswahl der Stufe** — `sim-capture-sim-emit` (jede Kamera belichtet beim gleichen Auslöser, die Bildausgabe wird so getaktet, dass die Leitung gemeinsam genutzt wird; die Bildrate sinkt, wenn die gemeinsame Nutzung abnimmt) in jedem Netzwerk, in dem die Kameras so getaktet werden können, dass sie die Leitung gemeinsam nutzen; `slip-emit-and-capture` (Kameras nutzen die Leitung abwechselnd, die Belichtung erfolgt nicht gleichzeitig) nur, wenn dies nicht möglich ist.
3. **PTP standardmäßig aktiviert**– kamerübergreifende Zeitstempel liegen auf einer gemeinsamen Uhr mit einer Abweichung von**~1 ms**, der [PTP-Grandmaster](#ptp-grandmaster--time_sync_status) des Backends. Die gleichzeitige Belichtung erfolgt über den M8-Hardware-Trigger (**< 100 µs** zwischen den Modulen), nicht über PTP: PTP gleicht *Zeitstempel* ab, nicht Belichtungen.
4. **Automatische Auswahl des Pixelformats pro Kamera** – Mono → `Mono12`, Farbe → `BayerRG12`.
5. **AE-Seeding** — Erfasst den aktuellen AE-Zustand jeder Kamera, damit die Verbindung die Belichtung während des Betriebs nicht zurücksetzt.
6. **GPIO-Trigger-Konfiguration** — `connect_array` schaltet jede Kamera frei (`TriggerMode=On`, `TriggerSource=Line2`), sodass der Impuls des MastersImpuls die Slaves über das M8-Kabel ansteuert. Dies ist ein Schritt, der nur bei einem Array gilt: Eine einzelne Kamera, die mit `LatticeCamera` geöffnet wurde, läuft stattdessen im Freilaufmodus. Durch das Freigeben des Arrays (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) versetzt jede Kamera wieder in den Freilaufmodus (`TriggerMode=Off`), wobei alle anderen Einstellungen so beibehalten werden, wie sie bei der Verbindung vorgefunden wurden: die Einstellungen für Trigger und Synchronisationsleitung (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; die Synchronisationsleitung der Master-Kamera wieder auf einen Eingang), PTP sowie die Kameraeinstellungen, die während der Sitzung geändert wurden – es sei denn, `keep_settings=True` / `clear_settings=True` (siehe [Kameraspeicher](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Attribute: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` und `packed_reason` (die letzten vier sind `None` auf einem Handle von `attach_array`), `connect_stream` und `backend_url`. `connect_stream` ist der `stream`-Block der Verbindungsantwort, die Form der Einzelverbindung plus `members` (`{serial: {state, why?}}`): `streaming`, sobald die erste synchronisierte Gruppe des Arrays eingetroffen ist (das Backend wartet darauf, begrenzt auf etwa 10 s), oder `no_frames`, wobei der letzte Abruffehler jedes Mitglieds als `why` gilt und bei Jumbo-Paketen den 1500-Byte-Fallback im `note`. `None` auf einem `attach_array`-Handle.

#### `connect_array()`-Signatur

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` gibt an, wie lange der Aufruf auf die Verbindung wartet, und wird als
Frist an das Backend gesendet. Eine bis dahin nicht hergestellte Verbindung wird rückgängig gemacht: Jede Kamera, die
geöffnet wurde, wird in den Zustand zurückversetzt, in dem sie vor dem Öffnen war, und geschlossen; kein Array wird verbunden, und der
Aufruf löst den Fehler `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, wobei der
Schritt benannt wird, den die Verbindung erreicht hatte). Ein Aufruf, der einen Fehler ausgelöst hat, hinterlässt also niemals ein Array
hinterlässt. Ein Array mit 9 Kameras stellt die Verbindung in etwa einer Minute her; unmittelbar nach dem
Einschalten der Kameras kann es mehrere Minuten dauern. Ein Array, das zwar bereit ist, aber noch keine
synchronisierte Bildgruppe aufweist, wird mit dem Fehler `connect_stream["state"]`
`no_frames`. Das Protokoll des Backends enthält eine Zeitangabe pro Verbindungsschritt
(`[array-connect <id>] <step>: <s> s …`) und eine letzte Zeile, die den langsamsten Schritt benennt.

`force_tier`-Werte:
- `"sim-capture-sim-emit"` – echte Synchronität (alle Kameras lösen an derselben Taktflanke aus).
- `"sim-capture-ftd-stagger"` – flexible zeitliche Staffelung (die Kameras senden zu leicht versetzten Zeitpunkten, sodass die Pakete auf der Leitung serialisiert werden).
- `"slip-emit-and-capture"` – sequenzielle Erfassung pro Kamera (keine zeitliche Synchronisation). Wird automatisch nur dann ausgewählt, wenn die Kameras nicht langsam genug getaktet werden können, um die Leitung gleichzeitig zu nutzen.

`wire_ceiling_mbps` überschreibt das **dauerhafte Leitungsbudget des Hosts** in MB/s – die einzige
Zahl, von der die gesamte Array-Zuweisung abhängt. Belassen Sie den Wert auf `None`, um den automatisch ermittelten
Wert zu verwenden. Verringern Sie ihn, wenn das Array GVSP-beschädigte Frames meldet: Der automatisch ermittelte Wert wird
aus der vom Netzwerkadapter gemeldeten Verbindungsrate abgeleitet, die USB-Adapter, schmale PCIe-Lanes und
ausgelastete gemeinsam genutzte Fabric-Strukturen überbewertet – und diese Überschätzung äußert sich in beschädigten Frames statt in einer
sichtbar langsamen Verbindung. Der Wert wird im Array-Erfassungsblock des Projekts gespeichert, sodass ein
erneutes Öffnen oder ein späteres `connect_array` ihn wie jede andere Array-Einstellung wiederherstellt.
Siehe [Array-Zustand](#array-health--which-subsystem-is-losing-frames).

#### Gepackter 12-Bit-Transport (`packed=`)

Bei voller Sensorauslastung wird die Bildrate eines Arrays dadurch bestimmt, wie schnell jede Kamera
ihre Bilder senden kann. 12-Bit-Formate (`BayerRG12`, `Mono12`) werden normalerweise als 16-Bit-
Wörter übertragen; beim „Packed Transport“ werden dieselben 12 Bit in 1,5 Byte pro Pixel gesendet und
auf dem Host entpackt, sodass die Übertragung jedes Frames 25 % weniger Zeit in Anspruch nimmt – etwa
**33 % mehr Bilder pro Sekunde bei voller Sensorauslastung** (gemessen: 10,7 → 14,0 fps bei einem
Array aus 9 Kameras). **Die gespeicherten Daten sind identisch**: gleiche Werte, gleiche 16-Bit-
Dateien, gleiche Metadaten.

- `packed="auto"` (Standard) komprimiert nur, wenn es sinnvoll ist **und** der Host das
  Dekomprimieren bewältigen kann: Bei der Verbindung misst das Backend seine eigene Dekomprimierungsgeschwindigkeit,
  die Anzahl der CPU-Kerne, die aktuelle Auslastung und das Netzwerkbudget. Ein Desktop-Rechner mit freien Kernen
  komprimiert bei voller Sensorauslastung; ein ausgelasteter oder leistungsschwacher Host (Einplatinencomputer,
  eingebettete Steuerungen) streamt unkomprimiert. Kleine ROIs und gebinnte Frames profitieren kaum davon.
- `packed="off"` (oder `False`) komprimiert niemals – um dies bei der Diagnose auszuschließen
  oder auf einem Host, der seine CPU für andere Aufgaben freihalten muss.
- `packed="on"` (oder `True`) packt 12-Bit-Formate immer.
- Jeder andere Wert löst `ValueError` aus. 8-Bit- und 16-Bit-Formate werden niemals
  gepackt.

`analyze_array_network(..., packed="auto")` trifft dieselbe Entscheidung,
sodass seine fps die Rate sind, mit der das Array ausgeführt wird (die Antwort enthält
`packed_transport` und `packed_reason`); sein `pixel_format` ist standardmäßig auf das eingestellt, was
„connect“ anwendet (Mono12 / BayerRG12), anstatt auf 8-Bit.

Die Entscheidung und ihre Begründung befinden sich in der Sitzung:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` wendet dieselbe Überschreibung auf jedes
Array in einem Projekt an (Standardwert `None`, was bedeutet, dass `"auto"`: ein Projekt keine
gepackten Einstellungen pro Array speichert).
Für eine maschinenweite Standardeinstellung setzen Sie `CHLOROS_ARRAY_PACKED12=off` (oder `on`) in der
Umgebung des Backends; eine explizite Einstellung von `packed="on"` / `"off"` hat jedoch weiterhin Vorrang.
`/api/camera/array/status` meldet `packed_transport` und `packed_decision`.

#### Erwartete Bildraten

Gemessene Array-Bildraten (hardware-synchronisiert, 12 Bit; voller Sensor = 2048 × 1536). Verwenden Sie diese zur Plausibilitätsprüfung von `status()["actual_fps"]`; `analyze_array_network()` projiziert die Bildrate auf Ihren eigenen Host und Ihr Netzwerk.

| Konfiguration | Array-Bildrate |
| --- | --- |
| Vollsensor, PCIe-10-GbE-Host-Netzwerkkarte, gepackter Transport (`packed="auto"` auf einem Desktop-PC) | ~14 fps |
| Vollsensor, PCIe-10-GbE-Host-Netzwerkkarte, ungepackt (`packed="off"`) | ~10,5 fps |
| Vollsensor, 8 Kameras an einem USB-10-GbE-Adapter | ~3,8 fps |
| Vollsensor, 1-GbE-Host-Verbindung | 1,2–1,7 fps |
| 2 × 2-Binning (`binning=2`), 10-GbE-Host-Netzwerkkarte | ~30–33 fps |

- Bei einer PCIe-10-GbE-Verbindung bilden die **Kameras** bei voller Sensorauslastung die Grenze, nicht das Netzwerk: 4 und 8 Kameras laufen mit derselben Bildrate.
- **`target_fps` ist die Triggerrate, und das Array läuft niemals schneller.** `target_fps=1` lässt das Array mit 1 fps laufen; `None` (die Standardeinstellung) lässt es mit der höchsten Rate laufen, die Host und Netzwerk unterstützen. Eine Rate, die über diesem Wert liegt, wird auf diesen Wert gesenkt (siehe [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` von `4294` in `status()` ist keine Verbindungsrate.** Das bedeutet, dass Windows die Geschwindigkeit des Adapters nicht auslesen konnte (der Treiber meldete den größten 32-Bit-Wert), was typisch für USB-Ethernet-Adapter ist. Die nutzbare Rate eines USB-Adapters unter Windows ist ohnehin auf 200 MB/s begrenzt (siehe [Array Health](#array-health--which-subsystem-is-losing-frames)); Informationen zur Optimierung finden Sie im Abschnitt „USB-10-GbE-Adapter“ der CLI-Referenz unter [Einrichtung und Optimierung der Host-Netzwerkkarte](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Auch unter Linux gilt eine USB-Obergrenze: 400 MB/s auf x86_64, 200 MB/s auf NVIDIA Jetson.** Der Adapter wird über sysfs identifiziert (mit Fallback auf `ethtool -i`), sodass ein USB-5-GbE- oder 10-GbE-Adapter an der Obergrenze begrenzt wird und nicht gemäß seiner Nennleistung von 500 bzw. ~1060 MB/s. Der Wert für x86_64 wurde unter Linux gemessen: Derselbe Adapter, der unter Windows bei Geschwindigkeiten über 200 MB/s Frames beschädigt, ließ ein Array aus vier Kameras bei jeder Obergrenze bis zu 480 MB/s fehlerfrei laufen. Auf dem NVIDIA Jetson arbeitete derselbe Adapter nur bei 200 MB/s fehlerfrei und verlor oberhalb dieser Grenze zeitweise Frames. Wenn Ihr Host ohne `gvsp_corrupt` mehr Frames verarbeiten kann, erhöhen Sie diesen Wert mit `wire_ceiling_mbps`. Unter Linux sind für einen Adapter an einem USB-2.0-Anschluss 30 MB/s vorgesehen; schließen Sie ihn an einen USB-3-Anschluss an.

#### Überbelegung (die Untergrenze pro Kamera)

Ein Array funktioniert in jedem Netzwerk; die Bildrate ist entscheidend. Bei der „Sim-Emit“-Steuerung wird jeder Kamera ein gleicher Anteil des kollisionssicheren Leitungsbudgets zugewiesen, und alle liefern weiterhin Bilder beim gleichen Trigger aus. Auf einer langsamen Leitung ist der Anteil – und damit die Bildrate – daher gering: 8 voll-Auflösung, 12-Bit-Kameras auf 1 GbE ohne Jumbo-Frames erhalten jeweils 6,5 MB/s, was etwa **1 fps**entspricht (etwa 1,3 fps gepackt). Der Anteil ist auf die langsamste Rate begrenzt, mit der die Kameras getaktet werden können (`per_cam_floor_bps`: wird beim Verbindungsaufbau aus dem GevSCPD-Bereich jeder Kamera ausgelesen und liegt niemals unter 1 MB/s). Wenn die Leitung eine langsamere Übertragung erfordert, greift die Verbindung auf**serielle**Übertragung zurück (`slip-emit-and-capture`): Die Kameras wechseln sich bei voller Verbindungsrate ab, sodass zwar jeder Frame ankommt, die Daten jedoch**nicht gleichzeitig verfügbar sind**– die Stufe der Antwort und die Blocker (`SERIAL: ...`) weisen darauf hin; übergebe `force_tier="sim-capture-sim-emit"`, um den Fallback abzulehnen. Die Frame-Größe spielt bei der Aufteilung keine Rolle, daher**erhöhen Binning und ROI zwar die Bildrate, ändern aber nichts daran, ob ein Array passt**. Nur eine Verbindung, die das Array bei keinem Übertragungsrhythmus – weder gleichzeitig noch abwechselnd – übertragen kann,**ist überbelegt** – der Fehlermodus ist GVSP-Paketverlust, nicht eine niedrigere Bildrate. Abhilfemaßnahmen: weniger Kameras, Jumbo-Frames durchgehend oder eine schnellere Netzwerkkarte.

- Die Antworten `analyze_array_network()` und `/api/camera/array/connect` enthalten `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` und `per_cam_floor_bps`. Wenn `oversubscribed` wahr ist, setzt die Projektion **die fps-Felder auf Null** (`achievable_fps_max` / `fps_bright` / `fps_dark`), anstatt eine irreführende, zwar langsame, aber funktionierende Rate zu melden.
- `POST /api/camera/array/connect` akzeptiert einen `pin_resolution`-Body-Parameter (**nur HTTP – kein SDK-kwarg**; `connect_array` stellt ihn nicht zur Verfügung). Es handelt sich um den strengen Modus: Eine überbelegte Verbindung mit gesetztem `pin_resolution` wird**kategorisch abgelehnt**, wobei der Fehler jede Abhilfemaßnahme namentlich nennt. Ohne diesen Parameter wird die Verbindung mit einer Warnung fortgesetzt, dass das Array Pakete verlieren wird. In beiden Fällen behält die Verbindung die angeforderte Frame-Größe bei – sie führt niemals eigenständig eine Binnung oder Beschneidung durch.
- Nur zu Testzwecken: Setzen Sie `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` in der Umgebung des Backends, um die Ablehnung auf eine deutliche Warnung herabzustufen – Sie stellen die Verbindung trotzdem her und akzeptieren den Paketverlust.

#### Array-Zustand – welches Subsystem verliert Frames

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) enthält einen aktiven `health`-Block auf einem
verbundenen Array, der in einem rollierenden **10-Sekunden**-Fenster neu ausgewertet wird Fenster neu bewertet. Es unterteilt den Frame-Verlust
in die beiden Ursachen, die gegensätzliche Abhilfemaßnahmen erfordern, anstatt einer einzigen „Unvollständigkeitsrate“, die
keine der beiden nennt:

| Feld | Bedeutung | Betroffenes Subsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pro serieller Schnittstelle) | Das Bild **kam an und war strukturell fehlerhaft**– GVSP-Paketverlust. |**Netzwerk**: Leitungsbandbreite, Pacing, NIC-RX-Ring, MTU |
| `never_arrived_rate_pct` (pro serieller Schnittstelle) | Der Frame **ist überhaupt nicht angekommen**– die Kamera hat nicht ausgelöst oder es wurde nichts gesendet. |**Trigger / Sync**: M8-Kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Schlechteste Bildrate der Kameras für jeweils. | — |
| `per_cam_rate_pct` | Kombinierte Unvollständigkeitsrate pro Kamera (beide Ursachen zusammen). | — |
| `stable_for_seconds` | Wie lange jede Kamera unter 0,01 % geblieben ist. | — |

Neben `health` gibt derselbe Datensatz die Anzahl der gesamten ausstehenden Zuweisungen an:

| Feld | Bedeutung |
| --- | --- |
| `wire_ceiling_mbps` | Das derzeit gültige, nachhaltig verfügbare Bandbreitenbudget des Hosts, MB/s. |
| `wire_ceiling_source` | Woher dieser Wert stammt, in Worten – z. B. `USB-capped 200 MB/s (was theoretical 1062; …)` oder `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, wenn `wire_ceiling_mbps=` diesen Wert festlegt. |
| `nic_is_usb` | `true` für einen USB-Ethernet-Adapter. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Auswertung:** Ein Wert ungleich Null für `gvsp_corrupt_rate_pct` bei einem Wert von 0 für `never_arrived_rate_pct` bedeutet,
dass Triggerung und Kabelsynchronisation einwandfrei sind und 100 % des Verlusts auf dem Netzwerkpfad liegen – senken Sie
`wire_ceiling_mbps` und stellen Sie die Verbindung erneut her. Das umgekehrte Muster deutet stattdessen auf das Synchronisationskabel oder die
Triggerleitung hin.

> **`target_fps` ist nicht der Hebel für beschädigte Frames.** Das GevSCPD-Pacing wird einmalig bei der
> Verbindung festgelegt, sodass eine Verringerung der Triggerrate den Tastgrad verändert, nicht jedoch die
> Burstrate bei simultaner Übertragung. Eine gemessene 5-fache Reduzierung der Anforderung brachte keine Verbesserung, während
> die Senkung der Leitungsobergrenze von 240 auf 200 MB/s bei demselben System die Fehlerrate von 10,4 % auf
> 0,00 % senkte.

> **Ein laufendes Array kann sich bei der LATTICE-Firmware (M3C / M3M) nicht selbst neu planen.** Es kann
> dieses Problem nicht selbst beheben; trennen Sie die Verbindung und stellen Sie sie erneut her, damit der „Connect-Time-Picker“ die Planung anhand
> der neuen Obergrenze neu durchführt.

Ein **USB-Ethernet-Adapter wird von der Sonde auf 200 MB/s begrenzt**, unabhängig von seinem
Typenschild begrenzt: Die Effizienztabelle, die eine Verbindungsrate in einen Dauerwert umrechnet,
basiert auf PCIe, und eine USB-Netzwerkkarte gibt zwar ihre Ethernet-Verbindungsrate an, ist dabei jedoch durch den
USB-Bus und dessen Treiber begrenzt. Die Begrenzung ist absolut, kein Bruchteil – ein USB-1-GbE-Adapter
erzielt ~80 MB/s und ist davon nicht betroffen. Dies gilt für Windows und Linux (einschließlich Jetson);
siehe [Erwartete Bildraten](#expected-frame-rates) für die Messorte.

#### `ArraySession`-Methoden

| Methode | Beschreibung |
| --- | --- |
| `status(timeout=10.0)` | Live-Status des laufenden Arrays: `target_fps` / `target_fps_auto` (die Triggerrate und ob der Ratecontroller sie verwaltet), `target_fps_clamped_reason` (warum eine festgelegte Rate niedriger gehalten wird, andernfalls `None`), `actual_fps` (gemessen; 0, sobald die Grab-Schleife aufgegeben hat oder ins Stocken geraten ist), `recommended_fps`, `streaming` (Falsch, sobald die Grab-Schleife aufgegeben hat – dies prüfen, nicht nur `last_error`; bleibt auch nach einem Neustart der Einstellungen „True“, was durch `restarting` signalisiert wird), `stalled` (True, solange die Grab-Schleife läuft, aber seit etwa zehn Frame-Perioden, 2–30 s, keine Frame-Gruppe veröffentlicht hat; `last_error` meldet dies erst nach 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` nur, wenn das Array mit PTP verbunden war **und** jedes Mitglied zuletzt `PtpEnable` gelesen hat; nicht dasselbe wie „gesperrt“, was `ptp_epoch` misst), `ptp_configured` (was die Verbindung angefordert hat), `ptp_off_members` / `ptp_unread_members` (Mitglieder, die zuletzt `PtpEnable` „off“ gelesen haben / nicht geantwortet haben), `member_recovery` (`None` oder ein Mitglied, das die Stromversorgung oder seine Verbindung verloren hat: `state`, `waiting`, obwohl es nicht antwortet, `reopening`, dann `recovered`, zusammen mit `members` und `why` – das Array nimmt es wieder auf, sobald es erneut antwortet, und weist ihm seine PTP-Zeit zum Zeitpunkt der Verbindung zu, Trigger- und Sync-Leitungsverkabelung, Pacing- und Kameraeinstellungen, und überträgt dann erneut ohne neue `connect_array`) und `ptp_epoch` (gemessen), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` bedeutet, dass Windows die Geschwindigkeit des Adapters nicht lesen konnte – siehe [Erwartete Bildraten](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (ob die Strahlungskette pro Band verwendbar ist und welches Element sie unterbrochen hat) sowie `band_clamp` — während eine Live-Ansicht das Array darstellt: den Anteil der beleuchteten Pixel, den die Entmischung auf 0 begrenzt hat, pro Element (`members`: `{serial: [band0, band1, band2]}` in der Reihenfolge der R-, G- und B-Bänder der Kamera) und pro Band, das der Composite anzeigt (`composite`: `{band name: share}`), wobei `warn` den Anteil angibt, den die Benutzeroberfläche als „Banddaten mit geringer Zuverlässigkeit“ kennzeichnet (etwa einmal pro Sekunde gemessen; `None`, wenn keine Live-Ansicht läuft) und `member_ae` – pro Element die Belichtung (`exposure_us`) und die Verstärkung (`gain_db`) des zuletzt veröffentlichten Bildes, entnommen aus dem Bild selbst (`read_nodes` von `ExposureTime` / `Gain` gibt den letzten Wert zurück, der während der automatischen Belichtung geschrieben wurde, nicht den aktuellen Wert). |
| `set_member_settings(serial, **kwargs)` | Schreibt die Einstellungen einer Mitgliedskamera, wie `CameraSession.set_settings()` sie (Belichtung, Verstärkung, Zielhelligkeit, …); das Element verbleibt im Array. Löst `ChlorosConnectError` aus, wenn es sich um eine Seriennummer handelt, die kein Element ist. Siehe [Belichtung bei Array-Elementen](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Die von Smart-Prep festgelegten Werte (Aufnahmeebene, Bildgröße, `pixel_format`), der aktive `health`-Block und das Übertragungsbudget des Hosts. Siehe [Array-Zustand](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Ändert die Triggerrate während des Array-Betriebs, ohne die Verbindung neu herzustellen. Ein Zahl-Pin fixiert sie (der Ratencontroller lässt sie unverändert); `None` setzt sie wieder auf „auto“. Gibt `{requested, target_fps, target_fps_auto, clamped}` zurück – `clamped` ist „True“, wenn die Rate auf das Maß gesenkt wurde, das der Host und das Netzwerk bewältigen können. Siehe [Intervallaufnahmen für Drohnenvermessungen](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Ein Bild von jeder Kamera, **alle durch einen einzigen Trigger**, als NumPy-Arrays (es werden keine Dateien geschrieben). Gibt einen `GrabResult` (einen `dict`): `{serial: array}` oder `{serial: (array, metadata)}` mit `with_metadata=True`. `.group_id` ist der gemeinsame Trigger für die Einzelbilder; `.errors` ordnet jede nicht berücksichtigte Kamera dem Grund zu (eine Kamera, die bei diesem Trigger nicht bedient werden kann, wird ausgelassen und niemals durch eine andere ersetzt). `processing` ist `raw` / `debayered` / `radiance` / `reflectance` / `display`. Dies entspricht der Projekt-API [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) – eine gemeinsame Implementierung –, sodass jedes Schlüsselwort dort (`band`, `align`, `bit_depth`, `refl_max`) funktioniert hier. Benötigt `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterator, der pro Tick ein `grab()`-Ergebnis liefert (nur Schlüsselwortargumente), mit einer Rate von `fps`, begrenzt durch `duration` (Sekunden) und/oder `count` (`None` = unbegrenzt). **Jede zurückgegebene Menge ist ein neuer Trigger:** Ein Tick, dessen neuester Trigger die bereits ausgegebene Menge ist, wartet auf den nächsten Trigger, anstatt ihn zu wiederholen, sodass der Stream mit der langsameren der beiden Geschwindigkeiten `fps` und der eigenen Bildrate des Arrays läuft. Löst `FrameFetchError` aus, anstatt stillzustehen: sofort, wenn jede Kamera aus einem bestimmten Grund den nächsten Tick wiederholen würde, nach 60 Sekunden Ticks ohne Frame und nach 60 Sekunden ohne neuen Trigger („kein neuer Trigger seit 60 Sekunden“). Gleiche Implementierung wie `ArrayHandle.frame_stream()`. Siehe [Live-Frames in Ihrer eigenen Pipeline](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Starten der backend-eigenen kontinuierlichen Aufnahme (der „Continuous“-Modus der GUI) → [`ContinuousHandle`](#continuoushandle). Speichert **jede** synchronisierte Gruppe, die das Array veröffentlicht, so wie `capture()` sie speichern würde (gleiche Exportkontrollen), bis der Vorgang gestoppt wird oder bis `max_captures` / `duration_s` (`None` = bis zum Stoppen). Löst `ChlorosConnectError` aus, wenn für dieses Array bereits ein Prozess läuft. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Eine synchronisierte Erfassungsgruppe. Gibt ein `CaptureResult` zurück (Liste von Frame-Dictionaries + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Übergeben Sie ein vorheriges Ergebnis`group_seq` als `after_seq` übergeben, um die nächste synchronisierte Gruppe danach zu erhalten, anstatt auf eine neue zu warten. Exportsteuerungen siehe unten. |
| `capture(..., smart=True)` | **Intelligente Aufnahme** – wartet, bis sich der AE bei allen Kameras eingependelt hat, und löst dann aus. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Schnellste Aufnahme: nur Rohdaten + der zugewiesene DAQ-Messwert (+ den freien kombinierten Index). Entspricht der Schaltfläche „Schnellste Erfassung“ in der Benutzeroberfläche. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Einzel-/Serien-/Intervallaufnahme in einer begrenzten Schleife. Gibt `list[CaptureResult]` zurück.**Erfordert `count` und/oder `duration_s`**, damit es beendet wird (das SDK verfügt über keine Strg+C-Funktion). Wird außerdem vorzeitig beendet, nach dem Durchlauf, wenn ein Durchlauf ein volles Laufwerk meldet (ein `.failed`-Eintrag mit `disk_full`); dieser Durchlauf ist das letzte Ergebnis. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Starte die Aufzeichnung der kombinierten Live-Indexansicht als Video/GIF → `RecorderHandle`. Das Video ist ein Motion-JPEG-Dateiformat `.avi` (kein ffmpeg erforderlich). Wenn keine GUI-Ansicht geöffnet ist, rendert das Backend Folgendes: `expr` (ein Indexausdruck in den Bandnamen des Arrays, z. B. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`) oder standardmäßig den NDVI-Wert des Arrays. Ein Composite-Recorder pro Array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Starten Sie eine Raw-Bayer-Serienaufnahme mit hoher Bildrate → `RecorderHandle`. Führen Sie die Nachbearbeitung offline mit `build_video()` durch. `orient=True`: Die von `build_video()` erstellten Produkte werden entsprechend der Kameraausrichtung gedreht (Produkte pro Kamera entsprechend der jeweiligen Kameraausrichtung, der kombinierte Index entsprechend der Ausrichtung der Master-Kamera); die Rohbilder bleiben unverändert, und das Burst-Manifest zeichnet die Ausrichtung jedes einzelnen Bildes auf. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Offline-Neuverarbeitung einer gespeicherten RAW-Bildserie zu kalibrierten Videos. Wartet, bis der Vorgang abgeschlossen ist (`wait=True`), und gibt das Build-Ergebnis zurück: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` – siehe [das Build-Ergebnis](#the-build_video-result) unten. |
| `build_video_status(job_id, timeout=15.0)` | Einen Offline-Build-Job abfragen: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Das gesamte Array freigeben — **falls diese Sitzung es geöffnet hat** (`connect_array`): Dann ist es `force_disconnect()`, und es wird ein Ausnahmetritt `ChlorosConnectError` ausgelöst, wenn die Freigabe nicht bestätigt wird (der Ausstieg aus dem Block `with` löst niemals einen Fehler aus). Bei einem Handle aus `attach_array` lässt es das Array weiterlaufen und gibt eine entsprechende Zeile an stderr aus: Das Array kann zur GUI oder zu einem anderen Skript gehören. Jede Kamera bleibt im freien Betrieb und ansonsten so, wie sie bei der Verbindung vorgefunden wurde; `keep_settings=True` speichert ihre Einstellungen im eigenen Speicher der Kamera, `clear_settings=True` setzt diesen Speicher auf die Werkseinstellungen zurück (siehe [Kameraspeicher](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Gibt das gesamte Array frei, unabhängig davon, wer es geöffnet hat. Verwenden Sie diese Funktion für einen angehängten Handle, wenn Sie das Array beenden möchten (z. B., um es mit anderen Einstellungen erneut zu verbinden). Kehrt zurück, sobald das Backend die Freigabe bestätigt hat oder wenn nichts mehr freigegeben werden kann (das Array ist bereits nicht mehr vorhanden oder es läuft kein Backend); ein Teardown, bei dem eine Zeitüberschreitung aufgetreten ist oder das vom Backend abgelehnt wurde, löst `ChlorosConnectError` aus. Mit beiden Flags wird zurückgegeben, was der Speicher jeder Kamera erhalten hat (`camera_memory`), andernfalls `None`. |

`capture()`-Exportsteuerung (derselbe Endpunkt, den die GUI und die CLI verwenden):

- `processing` / `levels` — `processing="all"` (oder `levels=["raw","radiance",…]`) speichert jeden anwendbaren Exporttyp pro Kamera; ein einzelner Wert „`processing`“ speichert nur diese Ebene.
- `aligned` — **standardmäßig aktiviert (`None`), da die Aufnahme desselben Arrays durch die Chloros-App**(ihrer Array-spezifischen**ausgerichteten** Erfassungseinstellung). Der Nicht-Rohdaten-Export jedes Mitglieds wird anhand des [Ausrichtungsprofils](#array-alignment) in den Bildausschnitt der Hauptkamera verschoben und auf die gemeinsame Überlappung der Elemente zugeschnitten, sodass die Dateien pixelgenau registriert sind und die gleiche Größe, den gleichen Ausschnitt sowie die gleichen `Alignment*`-Tags wie die der App aufweisen (innerhalb eines ROI wird die Transformation auf das Fenster neu bezogen); `XMP-mapir:AlignmentJSON` gibt `"applied": true` mit dessen `crop_roi` an. Die Rohdatei bleibt in ihrer nativen Größe unverzerrt, enthält die Transformation jedoch in den Metadaten. `aligned=False` speichert jede Datei in der nativen Größe, unverzerrt; `aligned=True` ist die explizit festgelegte Standardeinstellung. `continuous()`, `capture(smart=True)`, `capture_repeated()` sowie die auf Projektebene liegenden `ArrayHandle.capture()` / `capture_smart()` verwenden dieselbe Standardeinstellung. Unabhängig davon, ob die Verzerrungskorrektur angewendet wird oder nicht, zeichnet jede Mitgliedsdatei die `Alignment*`-Transformation ihrer Kamera auf, sobald das Array über ein Ausrichtungsprofil verfügt – mit Ausnahme einer Kamera, die das Profil nicht ausrichtet (entfällt oder wird durch eine fehlgeschlagene Lösung auf die Identitätsmatrix gesetzt) sowie die Referenzkamera, wenn keine andere Kamera ausgerichtet ist: Diese enthalten keine `Alignment*`-Tags. Liegt für das Array kein Profil vor, wird die Aufnahme unausgerichtet gespeichert, und das Ergebnis gibt dies an: `result.alignment` ist `{"requested": True, "applied": False, "warnings": [...]}`, wobei pro Grund, aus dem die Verzerrungskorrektur nicht angewendet werden konnte (kein Profil, Mitglieder mit unterschiedlichen Auflösungen, ein Array, das nicht, ein Bild, auf das das Profil nicht abgebildet werden kann). `applied` ist nur dann `True`, wenn jeder Export, der die Verzerrungskorrektur angefordert hat, diese auch erhalten hat. Die automatische Ausrichtung des Arrays erstellt das Profil, sobald sich die Belichtung stabilisiert hat; `ArrayHandle.calibrate_alignment()` / das CLI-Befehl `align-calibrate` erstellen eines auf Anfrage. `result.oriented` gibt an, ob `orient=True` die Produkte gedreht hat.
- `render_index=False` – Überspringen der kameraspezifischen Überlagerung des Vegetationsindex; standardmäßig wird diese dort gerendert, wo sie konfiguriert ist.
- `daq` / `daq_tolerance_ms` – Der DAQ-Messwert wird standardmäßig als `.daq`-Sidecar gespeichert, auch wenn keine ausgewählte Ebene dies benötigt; `daq=False` überspringt dies dann, und `daq_tolerance_ms` legt die Übereinstimmungstoleranz fest (Standardwert 500). `force_daq=True` ist die ältere Schreibweise des Standardwerts. Siehe [Der DAQ-Messwert, den eine Aufzeichnung speichert](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — die **Produkte**jedes Mitglieds (`display`, `radiance`, `reflectance`, den kameraspezifischen Index und das Komposit) nach der Ausrichtungsverzerrung mit der Kameraausrichtung: mit der Ausrichtung der**Master-Kamera**, als sie in deren Koordinatensystem verzerrt wurden (die Standardeinstellung `aligned` – die Mitglieder bleiben ko-registriert), andernfalls mit der ihrer eigenen Kamera.**Standard: jede Elementdatei nativ**, wie es bei Array-Aufnahmen immer der Fall ist; `raw` und `debayered` bleiben in jedem Fall nativ. Jede Datei speichert die Ausrichtung des Bildausschnitts, in dem sich ihre Pixel befinden (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` – bei einer RAW-Datei die des eigenen Kamerasystems), und die XMP-Datei `Alignment*` enthält die native Bildrahmentransformation, die vor der Ausrichtung angewendet wurde; eine RAW-Datei, die diese Transformation enthält, speichert zudem `ChlorosAligned*` (die des Master-Bildes), also die Ausrichtung eines Produkts, das in den Bildrahmen des Master-Bildes verzerrt wurde. Die Verarbeitung dreht jedes Produkt um den Wert, der zu seinen Pixeln gehört: um den Wert der Ausrichtung, wenn die RAW-Datei verzerrt wird (Array-Ausrichtung aktiviert), um den Wert der Kamera selbst, wenn nicht. `capture_repeated()` und `capture_fastest()` leiten sie weiter; die `ArrayHandle.capture()` / `capture_smart()` auf Projektebene übernehmen sie ebenfalls.

**TIFF-Komprimierung (nur HTTP-Einstellung):** `ArraySession.capture()` sendet keinen `compression`-Schlüssel, sodass die Backend-Standardeinstellung gilt – `POST /api/camera/array/capture` liest einen `compression`-Body-Parameter, `"none"` standardmäßig: unkomprimiert, die Pixel des Sensors genau so, wie sie gelesen wurden (~6,3 MB pro Bild in voller Auflösung). `"deflate"` (verlustfreie zlib L1 + horizontaler Prädiktor, ~4,1 MB/Bild) schreibt kleinere Dateien langsamer – beide sind verlustfrei und werden beim Import identisch gelesen. Das SDK stellt hierfür keine kwarg bereit; die Ausweichmöglichkeit ist `chloros-cli lattice array-capture --compression deflate` oder rohes HTTP. DEFLATE belegt das Python-GIL, sodass komprimierte Schreibvorgänge nicht über die Schreib-Threads der einzelnen Kameras parallelisiert werden können – für eine kontinuierliche Aufnahme in voller Auflösung mit Sensor-Rate ist die unkomprimierte Standardeinstellung erforderlich. Details: [CLI-Referenz → array-capture](cli-reference.md).**Aufeinanderfolgende Gruppen (HTTP):** Der Endpunkt gibt `group_seq` zurück; eine Schleife, die diesen Wert bei der nächsten Anfrage als `after_seq` zurücksendet, erhält die nächste synchronisierte Gruppe – einschließlich einer, die veröffentlicht wurde, während die vorherige Aufnahme gespeichert wurde — anstatt auf eine nach der Anfrage veröffentlichte Gruppe zu warten, und `skipped_groups` meldet alle von der Schleife übersehenen Gruppen. `capture_repeated`, das `--continuous` der CLI und der „Continuous“-Modus der GUI tun dies alle; Intervall-Durchläufe bleiben zu jedem geplanten Zeitpunkt aktuell.**Export-Überschreibungen pro Mitglied (nur HTTP):**derselbe Endpunkt akzeptiert auch `exclude_serials` (Liste – Mitglieder aus dem gespeicherten Satz entfernen; das Array wird weiterhin als eine synchronisierte Gruppe ausgelöst, und ausgeschlossene Mitglieder werden in `excluded` zurückgegeben), `serial_levels` (`{serial: [level tokens]}` – Überschreibungen auf Kameraebene) sowie `serial_index` (`{serial: bool}` – Überschreibungen für Index-Overlays pro Kamera). Es handelt sich hierbei um GUI-paritätsbezogene Körperparameter und**noch nicht um SDK-kwargs**; in den Zuordnungen fehlende Elemente greifen auf die array-weiten Werte `levels` / `render_index` zurück.

##### Überprüfung übersprungener Cams — `CaptureResult.skipped`

`ArraySession.capture()` gibt ein `CaptureResult` zurück, das eine Unterklasse von `list` ist: Durchlaufen Sie es, indizieren Sie es, `len()` – alle bestehenden Muster funktionieren weiterhin. Neuer Code kann das Attribut `.skipped` überprüfen, um festzustellen, welche Kameras ausgeschlossen wurden und warum. Der häufigste Fall sind RGB-Kameras in einem Array mit gemischten Filtern, wenn man nach „`processing="radiance"`“ oder „`"reflectance"`“ fragt – die Strahlungsintensität pro Bayer-Pixel ist für einen Breitbandsensor bedeutungslos, daher überspringt das Backend diese Kameras, anstatt unsinnige Ergebnisse zu liefern.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Die Grund-Tokens folgen dem Muster `<level>-not-applicable-to-rgb-cam` (ein Eintrag pro übersprungener Ebene, jeder enthält `level`). Eine Kamera, deren Kalibrierungspaket nicht geladen ist, überspringt `radiance` / `reflectance` mit `<level>-needs-calibration (connect the camera so its cal pack loads)`, und ohne verwertbaren DAQ-Reflexionsmesswert überspringt sie mit `reflectance-skipped-no-dls`. Die anderen reflexionsspezifischen Übersprünge sind `reflectance-skipped-no-fresh-dls` (kein aktueller Downwelling-Messwert verfügbar), `reflectance-skipped-bound-daq-unavailable (…)` (der gebundene DAQ konnte nicht erreicht werden) sowie `dls-uncalibrated-band-<nm>` – das Band liegt größtenteils außerhalb des radiometrisch kalibrierten Bereichs des DAQ-Lichtsensors (~374–974 nm), sodass die absolute, auf dem DAQ basierende Reflektanztrennung abgelehnt wird und der Frame lautstark auf die Sensorantwort zurückgestuft wird. Unter den im Handel erhältlichen Artikeln löst nur F988 dies aus; der unterstützte Pfad dieser Kamera ist der Reflektanzpanel-Workflow.

##### Dateien, die nicht geschrieben werden konnten — `CaptureResult.failed`

Eine Aufnahme, bei der nur einige der Dateien geschrieben werden konnten, gibt dennoch Folgendes zurück: Die Liste enthält jede Datei, die **geschrieben wurde**, und `.failed` listet den Rest auf, ein Dict pro Datei mit `serial` (`None` für das kombinierte Composite und das Sidecar-Element eines Arrays `.daq`), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, oder `match` – die Herkunft von `.daq` und `.match.json`, als `.daq` geschrieben wurde, sein Sidecar jedoch nicht) sowie `error`. Auf einem vollen Laufwerk enthält der Eintrag auch `disk_full: True`; jede spätere Erfassung auf diesem Laufwerk würde auf dieselbe Weise fehlschlagen, weshalb `capture_repeated()` an dieser Stelle endet. Bei einer teilweisen Erfassung wird keine Ausnahme ausgelöst, und eine Datei in `.failed` wurde nicht halb geschrieben auf der Festplatte belassen. `.failed` ist bei einer fehlerfreien Erfassung leer.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Die DAQ, die eine Erfassung ausliest, speichert — `.daq_files` und `.daq_matches`

Immer wenn eine DAQ an die Kamera oder das Array gebunden oder in einen Pool eingebunden ist, speichert jede Erfassung den verwendeten Downwelling-Messwert: eine `.daq`-Datei im Unterordner `daq/` des Aufnahmeordners, zusammen mit der `.match.json`- und der `.cameras.json`-Datei (zu dem der Messwert gehört). Dies gilt auch für Roh- oder debayerte Aufnahmen, und es handelt sich um dieselbe Datei, unabhängig davon, ob die Aufnahme über die GUI, die CLI oder das SDK erfolgte. `daq=False` überspringt dies bei einer Aufnahme, deren Pegel nicht die DAQ-Ebene (ein `radiance`-, `reflectance`- oder `display`-Pegel schreibt es weiterhin); `daq_tolerance_ms` verschiebt die Toleranzgrenze für diese Aufnahme und niemals den verwendeten Messwert. Welches DAQ: das an die Kamera oder das Array gebundene (die Lichtsensorauswahl der GUI, eine Projektbindung oder `chloros-cli daq pool-bind`), andernfalls das aktuellste, über das das Backend verfügt. Die CLI bezeichnet die Steuerelemente als `--no-daq` und `--daq-tolerance-ms` ([CLI-Referenz](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Eine Erfassung speichert den verwendeten Lichtsensorwert als `.daq` zusammen mit einem `<file>.daq.match.json`, der angibt, wie der Wert an die Belichtung gekoppelt war: `method`, `gap_ms` (von der Belichtung zum nächstgelegenen DAQ-Messwert) und `in_tolerance` (innerhalb von 500 ms oder dem `daq_tolerance_ms` der Aufnahme). Das XMP der Reflexionsdatei enthält dieselbe Bewertung wie `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` und `Chloros:DlsInTolerance`.

Eine Aufnahme wartet auf den ersten Messwert des DAQ nach der Belichtung, damit sie die Messwerte auf beiden Seiten davon zusammenführen kann (`method` `interpolated`). Ein DAQ-E sendet bei Tageslicht alle 0,4 s und in Innenräumen alle 1–2 s einen Messwert, sodass `capture()` erst etwa eineinhalb dieser Intervalle später zurückkehrt; die Belichtung selbst wird nicht verzögert. Auf ein DAQ, das keine Messwerte mehr sendet, wird nicht gewartet. Ein DAQ-E-Messwert ist der über das Intervall seit dem vorherigen Messwert gemittelte Lichtwert, daher wird er in die Mitte dieses Intervalls gesetzt: `gap_ms` wird auf diese Mitte hin gemessen, und der Wert von `.match.json`’`bracket_stamps_ns` liefert die eigenen Zeitstempel der Messwerte. `ArraySession.continuous()` wartet, ohne die Erfassung zu verlangsamen. Die Rückläufe von `capture_repeated()`-zu-zurück-Durchläufe warten nicht, behalten also ihre Rate bei und werden dem nächstgelegenen Messwert vor der Belichtung zugeordnet.

Ein Messwert verfällt nie, daher geht kein Bild durch einen verspäteten Messwert verloren – stattdessen wird er gekennzeichnet:

- **Der Stream des DAQ ist ins Stocken geraten** – der neueste Messwert wird verwendet, und `gap_ms` gibt an, wie alt er ist (`in_tolerance: false`: älter als 500 ms).
- **Kein DAQ zum Auslesen**– Verbindung unterbrochen (`disconnect()`, `chloros-cli daq pool-disconnect`) oder das gebundene DAQ ist nicht erreichbar – der letzte Messwert der Kamera wird wiederverwendet und**dieser** Belichtung: `method`, `retained`, `gap_ms` aus dieser Belichtung, `in_tolerance` neu berechnet und `retained_from_epoch_ns` mit dem Namen der Belichtung, der es ursprünglich zugeordnet war. Eine Kamera, für die noch nie ein Messwert vorlag, erhält weiterhin keinen Reflexionsgrad, und das Ergebnis gibt an, warum (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` oder die Ablehnung der Aufnahme).

Dateien, die eine Aufnahme im Ordner des in Chloros geöffneten Projekts speichert, werden in die Dateiliste dieses Projekts aufgenommen, genau wie eine Aufnahme in der App (Dateien, die an anderer Stelle gespeichert werden, bleiben unberührt): `CameraSession.capture()`, `ArraySession.capture()` (sowie `smart=True`) und `capture_cameras_transient()`. Das Ergebnis ist in `.project_add` zu finden (`capture_cameras_transient()`: das `project_add` des Dicts) -- `held`, `files` und das Backend-Element `message`; `held` ist `True`, solange ein Verarbeitungslauf läuft, und die Dateien werden nach dessen Beendigung hinzugefügt. `None`, wenn keine Datei im Projekt gelandet ist.

`CameraSession.capture()` und `ArraySession.capture()` geben sie als Ergebnisse `.daq_files` und `.daq_matches` zurück, während die HTTP-Erfassungsantworten (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) die Werte `daq_files` und `daq_matches` enthalten, jeweils einen pro `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` und `stream_fault` — der DAQ-eigene Stream-Fehler, der nur bei einem Messwert außerhalb der Toleranz abgefragt wird. Bevor Sie sich auf die Reflexanz aus einer Erfassung verlassen, überprüfen Sie `in_tolerance` in dessen `.match.json` oder XMP oder die [`status()`](#daqsensorsession-methods).

`processing`-Pegel:

| Pegel | Ausgabe |
| --- | --- |
| `"raw"` | Einkanal-Bayer (Mono-Kameras: das Einzelband) direkt vom Sensor. |
| `"debayered"` *(Standard im SDK)* | 3-Kanal-Farbe: Der Dunkeloffset (DSNU) aus dem Kalibrierungspaket wird aus den Rohdaten entfernt, anschließend erfolgt die kantensensitive Demosaikierung mit OpenCV – ohne Linearitäts-LUT, Flat-Field-Korrektur oder Farbentmischung (Monochromkameras: 1-Kanal-Graustufen). Die gespeicherte TIFF-Datei hat die **RGB**-Kanalanordnung (photometrisches RGB, sodass jeder Bildbetrachter die echten Farben anzeigt); die von `grab()` / `frame_stream()` zurückgegebenen Speicher-Arrays haben die**BGR**-Reihenfolge von OpenCV Reihenfolge – kehren Sie die Kanäle um (`frame[..., ::-1]`), um die Daten mit einer gespeicherten Datei zu vergleichen. `chloros-cli process` schreibt dasselbe Ergebnis wie `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm über die vollständige radiometrische Kette. Nur multispektral – RGB-Kameras werden übersprungen. |
| `"reflectance"` | uint16 0..32768 (Pix4D-kompatibel); erfordert eine Live-DAQ-Kopplung für die absolute Referenz. Nur multispektral. |
| `"display"` | Vollständige Kette entsprechend der GUI-Vorschau (CCM + WB + Gamma gemäß dem Profil der Kamera). |
| `"all"` | **Eine Datei pro anwendbarer Stufe** für jede Kamera (entspricht der GUI-Einstellung „Capture All“ / CLI-Standard). Die zurückgegebene Datei „`CaptureResult`“ enthält dann pro „`(cam, level)`“ ein Frame-Dict, wobei die Stufe in jedem Dict angegeben ist; nicht zutreffende Stufen erscheinen in `.skipped`. Der DAQ, der die verwendete Aufzeichnung gelesen hat, wird als `.daq`-Sidecar gespeichert. |

> **Hinweis – Die Standardeinstellung weicht von der in der CLI ab.** `ArraySession.capture()` ist standardmäßig auf `processing="debayered"` gesetzt; der Befehl `chloros-cli lattice array-capture` ist standardmäßig auf `processing="all"` gesetzt. Übergeben Sie „`processing="all"`“ explizit aus dem SDK, um die mehrstufige Speicherung der CLI/GUI widerzuspiegeln.

##### Dateinamen der Array-Erfassungsdateien

Jede Ebene wird in einem eigenen Unterordner von `output_dir` gespeichert (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` für `display`, `index/`, `composite/`). Jede Datei benennt ihre Kamera wie folgt: `SN<serial>-<filter>`: die Seriennummer, gefolgt vom Filterteil des Kameramodells – `SN214000533-FOCN` für eine `M3C-L87-FOCN`, `SN261902142-F450` für eine `M3M-L87-F450` – verbunden mit `-`, sodass ein Name dieselben durch „`_`“ getrennten Felder wie zuvor aufweist. Eine Kamera, deren Modell nicht gelesen werden kann, lautet lediglich „`SN<serial>`“, und Aufnahmen aus früheren Versionen enthalten keinen Filter im Namen. Der Dateiname hängt davon ab, wie viele Ebenen von einem Mitglied angefordert wurden:

| Angefragtes Mitglied | Dateiname | Beispiel (`reflectance`, Standard `format="tiff"`) |
| --- | --- | --- |
| **Eine Ebene** (`processing="reflectance"` oder ein einelementiges `levels`) | `sync_<timestamp>_SN<serial>-<filter>.<format>` – kein Ebenensuffix, wie in früheren Versionen, sodass bestehende Skripte weiterhin funktionieren | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Mehrere Ebenen** (`processing="all"` oder `levels` mit mehr als einem Element) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

Die [Clamp-Maske](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) folgt auf den Produktnamen, `clamp_masks/<product folder>/<product name>_clampmask.tif`: EineEbenen-Reflexionserfassung schreibt `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, während `processing="all"` `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif` schreibt. Die Regel für das Suffix wird für jedes Element anhand der angeforderten Stufen festgelegt, sodass eine übersprungene Stufe (z. B. Reflektanz bei einer RGB-Kamera) keinen Einfluss darauf hat. Lesen Sie die Pfade aus den zurückgegebenen Frames aus, anstatt sie selbst zu erstellen. Eine Aufnahme mit einer einzelnen Kamera (`CameraSession.capture()`) fügt immer die Ebene `capture_<ts>_SN<serial>-<filter>_<level>.tiff` hinzu.

##### PNG/JPEG: Nur Anzeige- und Indexprodukte

Ein PNG- oder JPEG-Dateiformat enthält 8 Bit pro Kanal. Die Anforderung eines Formats mit einer Farbstufe, die es nicht unterstützen kann, wird vor der Aufnahme abgelehnt, wobei eine Meldung angezeigt wird, die die Farbstufe und TIFF als das Format angibt, das diese unterstützt:

- **`radiance`**(float32) und**`reflectance`**(uint16, `32768` = 1,0) – und somit auch `processing="all"`, das diese umfasst. Das SDK überprüft dies anhand der Argumente: `capture()` löst**`ValueError`** ohne entsprechende Anforderung aus (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` von einer Kamera, die mehr als 8 Bit streamt** (12-Bit-M3C, Mono12-M3M) – das Backend lehnt diese ab, da es die Bittiefe jeder Kamera kennt: HTTP-Fehler 400 mit `error_code` und `FORMAT_CANNOT_HOLD_LEVEL`, ausgelöst als `ChlorosConnectError` (`CameraSession` / `ArraySession`) oder `RuntimeError` (`CameraHandle` / `ArrayHandle`). Die einstufige `debayered`-Aufnahme einer Kamera erfasst ein 8-Bit-Bild für PNG/JPEG und wird nicht abgelehnt.

Speichern Sie radiometrische Daten als `format="tiff"` (`ext=".tiff"`); verwenden Sie PNG/JPEG für `display` und Index-Renderings.

#### So verhält sich die automatische Belichtung

- **Zuerst Belichtung, dann Verstärkung.** Bei automatischer Verstärkung bleibt die Verstärkung bei 0 dB, bis die Belichtung den maximal zulässigen Wert erreicht hat.
- **Bei einer manuellen Belichtung bleibt die Verstärkung auf „Auto“**, bis Sie die Verstärkung ändern oder auf „Manuell“ stellen. Zurück auf „Auto“ gilt wieder die vorherige Verstärkungsgrenze.
- **Kleine Lichter verdunkeln die Szene nicht.** Ein Spiegelungsbereich von bis zu 1 % des Bildausschnitts wird ignoriert, bei den Feldern eines bekannten Kalibrierungsziels bis zu 0,5 % eines Feldes.
- **Die DAQ-Belichtungsobergrenze gilt nur, solange ein Kalibrierungsziel im Blickfeld ist.** Sie verhindert das Ausklippen eines weißen Feldes; ist kein Feld zu sehen, belichtet die Kamera entsprechend der Szene.
- **Konstantes Licht bleibt stabil, und sich änderndes Licht pendelt sich innerhalb von Sekunden ein.** Bei konstantem Licht bleiben die Belichtung und ihre Grenze unverändert, und nach einer starken Lichtänderung pendelt sich die Ansicht innerhalb weniger Sekunden ein. Ein Übersteuerungsfehler begrenzt die Belichtung nicht mehr dauerhaft, und bei einer Änderung des ROI wird die Belichtung wiederhergestellt.
- **Das Helligkeitsziel bleibt erreichbar.** Es steigt nicht an, solange sich die Kamera an ihrer Belichtungsgrenze befindet und ihre Verstärkung nicht erhöht werden kann, sodass ein wiederkehrendes Licht den Bildausschnitt nicht überbelichtet.
- **Eine Kamera, deren automatische Belichtung nicht mehr reagiert, wird auf dem Niveau gehalten, das für die aktuelle Lichtstärke vorgesehen ist.** Bei strahlendem Sonnenschein könnte eine Kamera auf ihre kürzeste Belichtungszeit fallen und dort mit einem dunklen Bild verharren, wobei alle Einstellungen normale Werte anzeigen. Chloros überwacht das Bild, hält die Kamera auf dem Niveau, das sie für diese Lichtstärke beibehalten hat, und lässt sie los, wenn sich die Lichtverhältnisse ändern. Ein verdunkelter oder abgedeckter Lichtsensor stört diesen Vorgang nicht.

#### Belichtung bei Array-Mitgliedern

Eine LATTICE-Kamera begrenzt `ExposureTime` auf eine Periode ihrer Bildrate. Während ein Array streamt, lässt Chloros jedes Element mit aktiviertem `AcquisitionFrameRateEnable` laufen, mit einer Rate knapp über der Triggerrate, sodass die Belichtungsobergrenze eines Elements **in jedem ROI** etwas unter der Triggerperiode liegt:

- Die Bildperiode eines Elements entspricht der Triggerperiode abzüglich einer Toleranz von 15 % der Periode (mindestens 3 ms, höchstens die Hälfte der Periode), und seine Belichtungsobergrenze entspricht dieser Bildperiode abzüglich etwa 0,1 ms. Beispiel: Ein Trigger mit 10 fps (100 ms) ergibt eine Elementrate von etwa 11,8 fps und eine Belichtungsobergrenze von etwa 85 ms.
- **Eine manuelle `ExposureTime`, die länger als diese Obergrenze ist, verlangsamt den Trigger, um sie anzupassen.** Chloros öffnet zunächst die Obergrenze dieses Mitglieds für den Wert und lässt die langsamere Rate einige Triggerperioden lang laufen (das Schreiben kann einige Sekunden länger dauern), schreibt den Wert dann, und der Trigger feuert nicht schneller, als es die Belichtungszeit zulässt. Bei einem Array mit automatischer Bildrate kehrt die zuvor verwendete Bildrate zurück, sobald jedes Element, dessen manuelle Belichtung den Trigger verlangsamt hatte, wieder auf automatische Belichtung umgeschaltet ist. Ein Wert, den die Kamera selbst nicht aufnehmen kann, wird weiterhin abgelehnt, wobei der nun akzeptierte Bereich in der Fehlermeldung angegeben wird, und die automatische Belichtung des Elements wird wiederhergestellt, falls sie nur für diesen Schreibvorgang deaktiviert war. Vor Version 1.3.0 begrenzte ein kleiner ROI jedes Element auf die Freilaufperiode des Sensors, unabhängig von der Triggerrate: 4,9 ms bei 768 × 304, sodass kein Element aufgehellt werden konnte.
- Chloros verkürzt niemals eine manuelle Belichtung, um sie an einen schnelleren Trigger anzupassen. Eine automatische Rate bleibt innerhalb der Grenzen, die die manuellen Belichtungen zulassen; eine festgelegte Rate, die schneller ist, als es eine Belichtung zulässt, belässt diesem Element eine Obergrenze, die lang genug dafür ist, und es akzeptiert keine Trigger, die schneller sind, als es seine Belichtung zulässt. Verringern Sie die Belichtung oder die festgelegte Rate, wenn jedes Element jeden Trigger verarbeiten muss.
- **Rücklesung:** Beim Auslesen der Knoten eines Elements werden „`AcquisitionFrameRateEnable` True“ und „`AcquisitionFrameRate`“ mit der Frequenz des Elements angezeigt; die Freigabe wird wieder deaktiviert, wenn das Array stoppt.
- **Das Array besitzt diese beiden Knoten. Schreiben Sie `AcquisitionFrameRateEnable` oder `AcquisitionFrameRate` nicht auf ein Array-Element** – stellen Sie stattdessen dieRate stattdessen mit `target_fps` / `set_target_fps()` fest.

Schreiben Sie die Belichtung eines Mitglieds mit `arr.set_member_settings(serial, exposure_time=20000)` (die Schlüsselwörter von `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` hat dieselbe Funktion). Das Element verbleibt im Array. Öffnen Sie ein Array-Element nicht mit `connect_camera()`: „connect“ lehnt eine Kamera ab, die sich bereits in einem Array befindet, und die von „`CameraSession`“ zurückgegebene Funktion entfernt die Kamera beim Trennen der Verbindung aus ihrem Array. „`ArraySession.status()["member_ae"]`“ meldet die Belichtung jedes ElementsBelichtung und Verstärkung jedes Elements aus dem neuesten Bild.

### Ein Projekteintrag, jeder Modus

Ein Projektordner enthält die Einstellungen jedes Geräts: `cameras.json` – die jeder Kamera (`cameras[].settings`, für eine einzelne Kamera und für jedes Array-Element) sowie die arrayweiten Einstellungen jedes Arrays (`arrays[].capture`: Format, ROI, Binning, Triggerrate, Wire-Budget) – sowie `sensors.json`, die Einstellungen jedes Lichtsensors (Integrationszeit, Bilddurchschnitt, Cap). Die App, die CLI, das SDK und der MCP-Server lesen und schreiben alle dieselben Dateien auf dieselbe Weise:

- **Ein unter einem Projekt verbundenes Gerät wird darin gespeichert und jede Änderung darin protokolliert.** „Unter einem Projekt“ bedeutet, dass im Backend (der App) ein Projekt geöffnet ist, oder – über ein `ChlorosProject` (`open_project()`), das dem Backend seinen eigenen Ordner zuweist (`project_path`). Eine Kamera, ein Array oder ein Lichtsensor, die bzw. der noch nicht im Projekt enthalten ist, wird bei der Verbindung zu dessen Dateien hinzugefügt (`connect_camera()`, `connect_array()`, die Verbindung des Lichtsensors, die CLI, die App, `chloros-mcp`) mit den entsprechenden Einstellungen. Eine Änderung aus einem beliebigen Modus (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, die Array-Einstellungen, `set_cap()`, die CLI, die Steuerelemente der App) wird einen Moment später in den Dateien dieses Projekts gespeichert. Eine an einer Kamera eingestellte Belichtung, Verstärkung oder ein AE-Ziel wird als „gespeichert“ markiert, sodass es beim erneuten Öffnen wieder übernommen wird.
- **Beim Öffnen eines Projekts werden dessen Geräte entsprechend konfiguriert**, und zwar in jedem Modus: Eine Kamera, ein Array oder ein Lichtsensor, die angeschlossen sind, aber keine eigenen Einstellungen haben, erhalten diegespeicherte Einstellungen (`lattice cam-connect` / `array-connect` bei geöffnetem Projekt, `chloros-cli project connect`, `open_project().connect_all()`, beim erneuten Öffnen der App). Ein Array, das unter einer neuen ID verbunden wird, aktualisiert das vom Projekt gespeicherte Array mit denselben Kameras; ein Array mit anderen Kameras wird daneben hinzugefügt.
- **Die speicherbezogene Funktion der App überschreibt niemals die Live-Einstellungen eines Geräts** mit einer älteren Kopie davon und behält die pro Element vorgenommenen Überschreibungen, die von einem CLI- oder SDK-Aufrufer gespeichert wurden, bei.
- **Das von Ihnen festgelegte Pixelformat ist dasjenige, das ein Projekt beibehält.** Ein in den Array-Einstellungen, über `--pixel-format` oder über `connect_array(pixel_format=)` festgelegtes Format, einschließlich 8-Bit, wird gespeichert und bei der erneuten Verbindung exakt wiederhergestellt. Ist kein Format festgelegt, verbindet sich jede Kamera mit ihrer 12-Bit-Standardeinstellung (`BayerRG12`, `Mono12` bei einem Mono-Sensor). Ein Formatname bezeichnet dessen Bittiefe; die Familie entspricht der des Sensors, sodass eine Farbkamera niemals auf ein Mono-Format und eine Mono-Kamera niemals auf ein Bayer-Format eingestellt wird, und die Zeile `[PixelFormat <serial>] ... written` gibt an, wann ein Format zugeordnet wurde. Die Exporte einer 8-Bit-Aufnahme sind 8-Bit, einschließlich Reflexionswerte: neu skaliert und mit dem Wert `PixelScale` gekennzeichnet.
- Ein Gerät, das ohne Projekt angeschlossen ist, zeichnet nichts.
- **Eine Aufzeichnung folgt einem Projekt, das nach ihr erstellt wurde; beim Schließen bleibt sie bei ihrem Projekt.**Wenn ein Projekt erstellt wird — `create_project()`, `chloros-mcp`s `create_project` oder die Funktion**Neues Projekt**der App — oder dupliziert wird (die Funktion**Duplizieren**der App, die die Kopie öffnet), während eine Aufzeichnung noch in das zuvor geöffnete Projekt schreibt, läuft die Aufzeichnung weiter, und was sie schreibt, landet im neuen Projekt an derselben Stelle darin (unten). Das Öffnen eines bestehenden Projekts – `ChlorosProject.process()` (wodurch das Projekt im Hintergrund geöffnet wird), `chloros-cli process`, die Funktion**Projekt öffnen** der App — führt dazu, dass eine laufende Aufnahme in dem Projekt fortgesetzt wird, in das sie aufzeichnet: Sie wird dort beendet, und ihre Dateien werden in die Dateiliste dieses Projekts aufgenommen (die App stoppt jede Aufnahme, bevor sie ein Projekt öffnet).
  - `ArraySession.record()` und `ArraySession.burst()`: Das Video oder der Burst-Ordner mit allen Einzelbildern wird nach Beendigung der Aufnahme vollständig dorthin verschoben. Das Ergebnis des Beendigungsvorgangs meldet den neuen Pfad (`out_dir`, `video_path`), und eine Zusammenstellung der Serienbilder wird daraus erstellt. Verweigert das Laufwerk den Verschiebungsvorgang, verbleibt die Aufnahme an ihrem ursprünglichen Speicherort, was im Ergebnis vermerkt wird (`relocation_error`). Die Rohbilder einer gestoppten Bildserie sowie `.daq` werden in die Dateiliste des Projekts aufgenommen, in dem sie landen; ein Video wird nicht zur .
  - `ArraySession.continuous()`: Jede Aufnahme nach der Änderung wird im neuen Projekt gespeichert; die zuvor gespeicherten bleiben an ihrem Platz.
  - `DAQSensorSession.record_start()` (und die Aufzeichnung des App-eigenen Lichtsensors): Die `.daq` wird nach Beendigung der Aufnahme in den `light_sensor`-Ordner des neuen Projekts verschoben; `record_stop()` gibt den neuen Pfad zurück, und die Datei wird in die Dateiliste dieses Projekts aufgenommen.

  Eine Aufzeichnung in einem Ordner außerhalb des geöffneten Projekts bleibt an ihrem Platz, und `start_camera_continuous()` / Multi-Stream-Aufträge behalten den ihnen zugewiesenen Ordner bei.
- **In der App werden die Aufnahmen nach dem Erstellen eines neuen Projekts in dieses Projekt verschoben.** Das Erstellen oder Duplizieren eines Projekts hält alle laufenden Erfassungsvorgänge und Aufnahmen aufrecht, und jede danach durchgeführte Erfassung („Alle erfassen“, „Erfassung starten“) wird im neuen Projekt gespeichert und in dessen Dateiliste aufgenommen. Das Öffnen eines bestehenden Projekts in der App stoppt zunächst alle laufenden Vorgänge.
- **Ein Projekt wird während der Aufzeichnung nicht verschoben.**Das Verschieben des geöffneten Projekts in ein anderes Arbeitsverzeichnis (in den**Einstellungen → Arbeitsverzeichnis** der App; kein SDK-Aufruf verschiebt ein Projekt, ein HTTP-Client, der an `/api/move-project` sendet, tut dies) wird mit HTTP 409 `PROJECT_RECORDING` abgelehnt, solange eine Array-Aufzeichnung oder eine Serienaufnahme, eine kontinuierliche Array-Erfassung, ein kontinuierlicher Kamera-/Multi-Stream-Auftrag oder eine Lichtsensor-Aufzeichnung in dieses Verzeichnis schreibt: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Dies ist unabhängig davon, in welchem Zustand sich die Kameras beim Trennen der Verbindung befinden (siehe [Kameraspeicher](#camera-memory--keep_settings--clear_settings)): Ein Projekt legt die Einstellungen seiner Kameras beim Öffnen selbst fest.

### Kameraspeicher — `keep_settings` / `clear_settings`

Wenn ein Array verbunden ist, übernehmen seine Kameras die Einstellungen des Arrays, und Sie können diese anschließend ändern. Wenn Chloros die Kameras freigibt — `ArraySession.disconnect()` / `force_disconnect()`, das Ende einer transienten Funktion, das Beenden von `start_camera_continuous()` — werden die einzelnen Kameras wieder in den Zustand versetzt, in dem sie vor dem Anschließen vorgefunden wurden: ihre Trigger- und Sync-Leitungsverkabelung, PTP- und Kameraeinstellungen — `ExposureAuto` / `GainAuto` (sowie eine manuelle Einstellung von `ExposureTime` / `Gain`), `TargetBrightness` sowie die AE-Grenzwerte und -Dämpfung, den Weißabgleich, den Schwarzwert, `PixelFormat`, Binning sowie den ROI, die Farbpipeline (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTPs `PtpSlaveOnly` / `PtpDomainNumber`, die Paketgröße (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), die Pacing-Rate (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), die Chunk-Daten (`ChunkModeActive`, jeweils `ChunkEnable` unter seinem `ChunkSelector`), die Entstörung des Triggereingangs – ein Array liefert die Synchronisationsleitung eines Slaves (`LineFilterSelector` / `LineFilterWidth`) sowie die Erfassungsbildrate (`AcquisitionFrameRate` nur, wenn die Freigabe aktiviert war: Ist sie deaktiviert, wird die Rate mit dem höchsten Wert gelesen, den das Pacing zulässt – es handelt sich nicht um eine Einstellung). Es wird nur das geschrieben, was sich unterscheidet: der Zeitpunkt, zu dem die Kamera zuerst gestoppt wurde, das Pacing und die Bildrate vor einer Belichtung, die deren Bildperiode benötigt.

Für den Einsatz der Kameras außerhalb eines Projekts schreiben zwei Argumente den **eigenen Speicher** der Kamera – das, womit sie in jeder Software hochfährt (bei `disconnect()` / `force_disconnect()`, jede transiente Funktion, `start_camera_continuous()` und `ChlorosProject.disconnect_all()`; MCP: dieselben Argumente wie bei `disconnect_device` und den transienten Tools; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** speichert die Einstellungen jeder Kamera so, wie sie bei Beendigung der Sitzung im Speicher vorlagen (UserSet2, zum Start-Set gemacht). Die Kameras laufen weiterhin im Freilaufmodus (`TriggerMode` `Off`), die Verkabelung der Synchronisationsleitung bleibt unverändert. Eine Kamera ohne „UserSet2“ speichert in „UserSet1“; dieser Computer merkt sich dies, sodass die Selbstheilung des Chloros-Arrays – die ein „UserSet1“ zurücksetzt, das von Chloros’ Basiswert abgewichen ist – diese Kamera unberührt lässt.
- **`clear_settings=True`** setzt den Speicher jeder Kamera auf die Werkseinstellungen zurück: Chloros’ Basiswert, den die Selbstheilung des Arrays – Werkseinstellungen mit `TriggerMode`, `Off`, `TriggerSource`, `Line2`, automatischer Belichtung und dem kalibrierten Weißabgleich einer RGB-Kamera beibehalten – in UserSet1, das zum Boot-Set gemacht wurde.
- Nicht beides (`ValueError`; das Backend antwortet mit 400). Die Antwort enthält `camera_memory`, je nach Kamera `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` oder `{"cleared": bool, "errors": [...]}` — `force_disconnect()` gibt es zurück; für einen Auftrag wird es mit `stop()` geliefert.

Die App bietet diese nicht an.

**Jeder Ausstieg versetzt die Kameras zurück.**Die Rückversetzung in den ursprünglichen Zustand erfolgt auf jedem Ausstiegspfad: `CameraSession.disconnect()` (die Kamera**X**der App), das**X** der App auf einem Array oder einem beliebigen Array-Element (das gesamte Array wird freigegeben), das Schließen der App (das Backend hat bis zu 20 s Zeit, um alle Kameras, Arrays und laufende Daueraufzeichnungen zurückzusetzen; eine noch laufende transiente Funktion oder ein einmaliger Aufruf stoppt an ihrem nächsten sicheren Punkt, setzt zunächst ihre Kameras zurück, für bis zu 8 s, und antwortet mit HTTP-Fehler 503 `SHUTTING_DOWN`; ein noch laufender `connect_array()` stoppt vor seiner nächsten Kamera und stellt die von ihm geöffneten Kameras wieder her, für bis zu 10 s, und löst `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); und eine, die während des Herunterfahrens eintrifft, öffnet nichts: HTTP 503 `SHUTTING_DOWN`)sowie ein Befehl `chloros-cli`, der beendet wird oder dessen Terminal geschlossen wird (sein Job wird gestoppt und seine Kameras werden zuerst zurückgesetzt). Ein Backend, das abstürzt oder zwangsweise beendet wird, kann nichts zurücksetzen; ein Neustart versetzt jede Kamera in ihren Speicherzustand zurück.**Ein Projekt legt seine Kameras selbst fest.** Wenn sich ein Array über ein Projekt verbindet – das geöffnete Projekt der App oder ein `ChlorosProject`, das sich dem Backend (`project_path`) selbst benennt – erhält jedes Mitglied die Einstellungen, die das Projekt für es gespeichert hat: einen Belichtungs-, Verstärkungs- oder AE-Zielwert, der für es beibehalten wird (als Ihr eigener Wert behandelt, sodass die automatische Belichtung ihn unberührt lässt), sowie seinen Weißabgleichmodus (Korrektur nach Chloros; der kamereneigene Weißabgleich bleibt deaktiviert), je nachdem, was der Speicher der Kamera enthält.

### Aufnahmemodi & Aufzeichnungsgeräte

Die Oberfläche des Arrays spiegelt das Aufnahmefenster der Benutzeroberfläche wider: Einzelbild-/Serienbild-/Intervall-/Schnellauslöser-Modi sowie zwei Aufzeichnungsgeräte (Live-Composite-Video und RAW-Serienaufnahme → Offline-Nachbearbeitung).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**ist die Serien-/Intervallschleife des SDKs. Da es kein `Ctrl+C` gibt, um diese Schleife per Skript zu unterbrechen,**müssen** Sie `count` und/oder `duration_s` übergeben (die Schleife stoppt, sobald einer der beiden Werte erreicht ist). `interval_s` wird ab dem Beginn jedes Durchlaufs gemessen (entsprechend der GUI). Kontinuierliche Durchläufe (`interval_s=0`) verketten `group_seq` → `after_seq`, es handelt sich also um aufeinanderfolgende, synchronisierte Gruppen mit der Geschwindigkeit des Arrays. Die übrigen kwargs werden direkt an `capture()` weitergeleitet.
- **`continuous`**ist die backendeigene kontinuierliche Erfassung – der „Continuous“-Modus der GUI.**Verwenden Sie diese Option anstelle von `capture_repeated()`, wenn Sie möchten, dass jeder Trigger mit hoher Bildrate auf die Festplatte geschrieben wird:** `capture_repeated()` sendet eine HTTP-Anfrage pro Gruppe aus Ihrem Skript, während `continuous()` zwei Aufzeichnungen gleichzeitig im Backend verarbeitet, ohne dass ein Hin- und Rückweg zwischen den Gruppen erforderlich ist. Gemessen bei ROI 512 auf einem Array mit 9 Kameras: 73–77 Aufzeichnungen/s, im Vergleich zu einem Array, das etwa 78 veröffentlicht. Behalten Sie `capture_repeated()` für die Intervallaufzeichnung (`interval_s`) bei sowie für Durchläufe, die Sie einzeln überprüfen. Schließen Sie unter Windows das Backend bei hohen Bildraten von Microsoft Defender aus (siehe [Schritt 1](#step-1--install-the-chloros-platform-package)).
- **`record`** ist *für die Überwachung geeignet*: Es zeichnet den gerenderten Live-Composite des kombinierten Index auf. Bei geöffneter kombinierter Ansicht der GUI zeichnet es diese Ansicht so auf, wie sie angezeigt wird. Ist keine Ansicht geöffnet, rendert das Backend die Ansicht selbst: `expr`, wenn Sie eine übergeben, andernfalls den NDVI-Wert des Arrays (die roten und NIR-Bänder, die 660 nm und 850 nm am nächsten liegen, wie sie von der Live-Ansicht erfasst werden), sobald das Array ausgerichtet ist. Benennen Sie die Bänder in `expr` so, wie das Array sie benennt: `<band>_<filter>` für eine M3C-Kamera (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` für eine M3M (`Mono_850`); ein kurzer Farbname (`NIR`, `Red`) funktioniert nur, wenn ein Band des Arrays diesen Namen trägt. Eine während der Aufzeichnung geöffnete GUI-Ansicht wird aufgezeichnet, solange sie geöffnet bleibt. Eine Aufzeichnung, die keinen Frame schreibt, schlägt mit den Fehlern `error` und `"no-frames: …"` fehl (das Array befindet sich möglicherweise noch in der Ausrichtungsphase oder passt seine Belichtung an), und es wird ein Fehler mit `stop()` ausgelöst (siehe [`RecorderHandle`](#recorderhandle)). Ein Composite-Recorder pro Array (löst einen Fehler aus, wenn bereits einer läuft).
- **`burst` → `build_video`** ist *analysetauglich*: `burst` schreibt Rohbilder + ein Manifest pro Bild + einen `.daq` pro eindeutigem DLS-Lesevorgang unter `<output>/bursts/<base>/` mit der vollen Rate der Grab-Schleife (keine Verkettung, kein exiftool, keine Live-Ansicht). `build_video` gleicht jedes Bild zeitlich mit dem nächstgelegenen `.daq` ab und führt die Radiance-/Reflexionsgrad-/Index-Kette der Import-Pipeline erneut aus. `products` ist eine Liste von `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (Standard: der kombinierte Index). `burst().stop()` löst außerdem automatisch die Erstellung eines nach bestem Bemühen kombiniertenIndex-Build automatisch aus, der im Stop-Ergebnis als `build_job` zurückgegeben wird.

##### Das `build_video`-Ergebnis

| Schlüssel | Bedeutung |
| --- | --- |
| `success` | `True` nur, wenn jedes angeforderte Produkt wie gewünscht erstellt wurde: mindestens eine Ausgabe, nichts in `skipped`, und kein Eintrag in `degraded`. |
| `outputs` | Pfade der geschriebenen Videos (und GIFs). |
| `errors` | Probleme als Text. |
| `skipped` | Je ein `{"serial", "level", "reason"}` pro angefordertem Produkt, für das **kein** Video erzeugt wurde – entweder aufgrund eines Problems mit der Kamera (kein Kalibrierungspaket für diese Kamera, kein DLS-Messwert für die Reflexion, Fehler in der Verarbeitungskette) oder des kombinierten Index, der `"kind": "combined"` und `serial` `None` enthält (kein Rot-/NIR-Bandpaar unter den Kameras, keine Gruppe mit Strahlungsintensität für beide Bänder). Jeder Eintrag führt zu `success` und `False`; die anderen Videos werden weiterhin geschrieben und aufgelistet. |
| `substitutions` | Kameras, die auf einer anderen Ebene als angefordert gerendert wurden, und der Grund dafür – eine RGB-Kamera, für die Radiance/Reflexion angefordert wurde, erhält ihr Anzeigevideo, das weißabgeglichen ist, sofern das Kalibrierungsbundle der Kamera verfügbar ist. Kein Fehler. |
| `notes` | Informationstext, kein Fehler – z. B. hat eine Kamera ohne kameraspezifischen Indexausdruck kein Indexvideo, oder das Anzeigevideo einer RGB-Kamera ist nicht weißabgeglichen (kein Kalibrierungspaket oder eines ohne werkseitigen Weißabgleich), wobei die Kamera namentlich genannt wird. Kein Fehler. |
| `degraded` | `True`, wenn ein Produkt geschrieben wurde, das jedoch nicht dem angeforderten entspricht: Der offline kombinierte Index ist die integrierte normalisierte Differenz aus Rot und NIR, daher wird ein konfigurierter Indexausdruck nicht ausgewertet. Erzeugt `success` `False`. |
| `combined` | Zusammenfassung des kombinierten Index: die verwendeten Bänder, der LUT-Bereich, die erstellten Bilder. |
| `clamp_masks` | Pfade der [Unmix-Clamp-Masken](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) derEinzelbild-kalibrierten TIFFs (`save_tiffs=True`), in `clamp_masks/<level>/` neben dem `<level>/`-Ordner der TIFFs, jeweils eine pro TIFF, in dem ein Pixel geklemmt wurde. Ein Video enthält keine. |
| `clamp_fractions` | Was ein Video anstelle einer Maske angibt: ein `{"kind", "serial", "level", "frames"}` pro Videoprodukt (`kind` `"combined"` für die zum kombinierten Index gehörenden Kameras), `frames` eine Liste von `[frame, R, G, B]` – der Anteil der Pixel dieses Bildes, die bei der Entmischung in dem Band begrenzt wurden, das sein R-/G-/B-Kanal enthält (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Wird von `ArraySession.record()` und `ArraySession.burst()` zurückgegeben. Verwenden Sie es als Kontextmanager, um beim Beenden des Scopes automatisch zu stoppen, oder steuern Sie es manuell.

| Element | Beschreibung |
| --- | --- |
| `job_id` | Backend-Job-ID (str). |
| `kind` | `"composite"` (aus `record`) oder `"raw"` (aus `burst`). |
| `start_stats` | Das vom Aufruf `start` zurückgegebene Wörterbuch. |
| `result` | `None` während der Ausführung; das endgültige Stop-Ergebnis-Dict nach dem Stopp. |
| `stats(timeout=10.0)` | Live-Auftragsstatistiken (geschriebene Frames, erzielte fps, verstrichene). |
| `stop(timeout=60.0)` | Den Recorder stoppen; gibt das Endergebnis zurück und speichert es im Cache (`.result` bleibt `None`, wenn die Stoppanforderung selbst fehlgeschlagen ist). Bei einer fehlgeschlagenen Aufzeichnung (`record()`) – bei der kein Frame geschrieben wurde (`error` `"no-frames: …"`) — wird das Ergebnis in `.result` zwischengespeichert und anschließend wird ein Ausnahmestatus in `ChlorosConnectError` ausgelöst. Idempotent (ein zweiter Aufruf gibt das zwischengespeicherte Ergebnis zurück, ohne einen Ausnahmestatus auszulösen). Das Verlassen eines `with`-Blocks stoppt den Recorder, ohne einen Fehler auszulösen; überprüfen Sie daher anschließend `rec.result.get("error")`. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Wird von `ArraySession.continuous()` zurückgegeben. Verwenden Sie es als Kontextmanager — das Verlassen des Blocks `with` stoppt die Aufzeichnung, falls diese noch läuft — oder steuern Sie ihn mit `status()` / `wait()` / `stop()`. Pro Array läuft jeweils nur eine kontinuierliche Erfassung.

| Element | Beschreibung |
| --- | --- |
| `start_status` | Das vom Start-Aufruf zurückgegebene Status-Dict. |
| `result` | `None` während der Ausführung; das endgültige Status-Dict nach Beendigung der Erfassung (gesetzt durch `stop()`, oder durch einen `status()` / `wait()`, der das Ende festgestellt hat). |
| `status(timeout=10.0)` | Aktuelles Status-Dikt (Felder siehe unten). |
| `wait(timeout=None, poll_s=0.5)` | Blockieren, bis die Erfassung von selbst endet – `max_captures` oder `duration_s` erreicht, das Array leer, ein volles Laufwerk, ein Fehler – und den endgültigen Status zurückgeben. Mit `timeout` den aktuellen Status nach dieser Anzahl von Sekunden zurückgeben, auch wenn der Vorgang noch läuft: siehe `running`. |
| `stop(timeout=30.0)` | Beendet die Erfassung und gibt den Endstatus zurück. Idempotent (ein zweiter Aufruf gibt das zwischengespeicherte Ergebnis zurück). |

Statusfelder:

| Feld | Bedeutung |
| --- | --- |
| `running` | „True“, solange die Erfassung läuft. |
| `captures` | Synchronisierte Gruppen gespeichert. |
| `files` | Dateien geschrieben. |
| `failed_files` | Dateien, die nicht geschrieben werden konnten (wie in [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Insgesamt fehlgeschlagene Erfassungsversuche, einschließlich derer, bei denen die Erfassung erneut versucht wurde (ein HTTP-Fehler 503, während das Array nicht rechtzeitig eine neue Gruppe bereitgestellt hat). |
| `skipped_groups` | Vom Array veröffentlichte Gruppen, die nicht gespeichert wurden (die Erfassung geriet in Verzug). |
| `captures_per_s` | Durchschnittliche Speicherrate seit Beginn. |
| `elapsed_s` / `started_at` | Bisherige Laufzeit in Sekunden; Startzeit (Unix-Epochensekunden). |
| `last_group_seq` | Die neueste synchronisierte Gruppe, die gespeichert wurde. |
| `last_error` | Der letzte Fehler oder `None`. |
| `stop_reason` | Einmal beendet: `count` (`max_captures` erreicht), `duration` (`duration_s` erreicht), `stopped` (`stop()` aufgerufen), `array_gone` (das Array wurde getrennt), `error` (20 Fehler in Folge oder eine vom Backend abgelehnte Anfrage), `disk_full` (eine Datei, für die auf dem Laufwerk kein Platz war). `None` während der Ausführung. |
| `max_captures` / `duration_s` | Die zu Beginn festgelegten Grenzen (`0` = keine). |
| `defender_cores` | Von Microsoft Defender in den letzten 10 Sekunden genutzte CPU-Kerne; `None`, bis ein Wert gemessen wird, und immer `None` auf einem Host ohne Defender. |
| `defender_warned` | „True“, sobald die Erfassung protokolliert hat, dass Defender die Leistung beeinträchtigt (bei 1 Kern oder mehr). Schließen Sie das Backend vom Defender-Scan aus – siehe [Schritt 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Live-Frames in Ihrer eigenen Pipeline

Um synchronisierte Frames in Ihren eigenen Code einzuspeisen – maschinelles Lernen, einen benutzerdefinierten Viewer, Ihren eigenen Recorder – verwenden Sie `ArraySession.frame_stream()`. Jeder Tick liefert ein `GrabResult`: `{serial: numpy array}` für jede Kamera, **alles aus einem einzigen Trigger**, wobei die ID dieses Triggers in `.group_id` enthalten ist. Es handelt sich um dieselbe Implementierung wie bei `ArrayHandle.frame_stream()` der Projekt-API, daher gilt alles unter [Ein Trigger pro Lesevorgang](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) und [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) gilt für beide.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Jede ausgegebene Menge ist ein neuer Trigger.** `frames.group_id` wiederholt sich niemals innerhalb eines Streams; wenn Sie mehr `fps` anfordern, als das Array verarbeitet, wartet der Stream auf den nächsten Trigger, sodass er mit der langsameren der beiden Geschwindigkeiten läuft.
- **Fehlende Kameras werden namentlich aufgeführt, nicht nachgetragen.** Eine Kamera, die bei diesem Trigger nicht bereitgestellt werden konnte, fehlt im Dict und wird in `frames.errors` aufgeführt.
- **Es endet abrupt.** Bei einer Ablehnung würde sich `FrameFetchError` beim nächsten Tick wiederholen, nach 60 Sekunden ohne Frame oder nach 60 Sekunden ohne neuen Trigger.**Was es leistet.**`frame_stream()` überträgt ganze Numpy-Frames über die lokale HTTP-Verbindung und erreicht Spitzenwerte von fast**170 MB/s**. Das reicht für die Live-Verarbeitung von Roh-Frames oder eines gebinnten oder ROI-Arrays aus, jedoch nicht für jeden Trigger bei voller Auflösung – um**jeden** Trigger beizubehalten, speichern Sie ihn auf die Festplatte mit [`continuous()`](#continuoushandle). Gemessen an einem Array aus vier Kameras (FOCN, FRGN, FRGB, F450) mit „Packed Transport“; jeder Tick war ein vollständiger Ein-Trigger-Satz:

| | Voller Sensor (Array 14,2–14,4 fps) | 2 × 2-Binning (Array ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 Sätze/s | 22,2 Sätze/s |
| `frame_stream(processing="debayered")` | 2,3 Sätze/s | 9,3 Sätze/s |
| `frame_stream(processing="display")` | 0,8 Sätze/s | 3,0 Sätze/s |
| MJPEG pro Kamera, 4 Streams bei 720p | jeweils 13,5 fps; jedes Bild wird einem Trigger zugeordnet | jeweils ~25 fps; ~45 % der Trigger sind auf allen 4 vorhanden |

Verwenden Sie `processing="raw"` für den Durchsatz und die Demosaikierung auf Ihrer Seite; die Stufe `debayered` entfernt zudem den Dunkeloffset des Kalibrierungspakets, was bei einem Demosaicing mit `raw` nicht der Fall ist.

#### MJPEG-Streams pro Kamera (kein RTSP)

**RTSP wird nicht unterstützt.** Die Streaming-Schnittstelle ist MJPEG über HTTP (`multipart/x-mixed-replace`, ein JPEG pro Teil, jeder Teil mit einem `Content-Length`), eine URL pro Kamera:

| URL | Was wird bereitgestellt? |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-Bit-Vorschau, bis zu ~30 fps. Bei einem Array-Element legen `max_h` (Standard 720, 64–2160) und `jpeg_q` (Standard 80) legen Größe und Qualität fest. |
| `GET /api/camera/<serial>/stream-annotated` | Dieselbe Vorschau mit darauf gezeichneten GUI-Overlays (Zebra, Raster, Fadenkreuz, Histogramm, Peaking, Spotmeter). `ChlorosLocal.camera_stream()` liest diese Werte für Sie aus. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-Bit-Reflexionswerte pro Band – siehe [Live-Reflexionsstrom](#live-reflectance-stream-mjpeg). |

- **Trigger-Header.** Bei einem Array-Element enthält jeder Teil aller drei Streams `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` und `X-Chloros-Timestamp-Ns`. Teile von verschiedenen Kameras mit demselben `X-Chloros-Group-Id` gehören zum selben Trigger: Sammeln Sie die Teile und gruppieren Sie sie anhand dieses Headers, um einen Satz wieder zusammenzusetzen. Eine eigenständige Kamera hat keine Gruppe: Ihre Vorschau-Teile enthalten keinen dieser Header (ihre Reflexionskomponenten tragen einen leeren Header „`X-Chloros-Group-Id`“).
- **Der Vorschau-Stream lässt unter Last Frames ausfallen.** Er liefert das neueste Frame und überspringt alle, die er nicht rechtzeitig senden konnte; daher erreicht bei hohen Array-Raten nicht jeder Trigger den Stream jeder Kamera (siehe Tabelle oben). Verwenden Sie für jeden Trigger `frame_stream()` oder `continuous()`.
- **`/api/camera/stream/<serial>` wird angehalten, solange der kombinierte Live-Stream des Arrays geöffnet ist** (`/api/camera/array/stream/<array_id>` – die Array-Ansicht der GUI öffnet ihn). Der Stream jedes Mitglieds sendet dann einen Platzhalter-Frame „Paused (Array-Stream aktiv)“ ohne `X-Chloros-*`-Header und nichts weiter, bis der kombinierte Stream geschlossen wird; danach wird die Übertragung von selbst fortgesetzt. `stream-annotated` und `stream-reflectance` werden nicht angehalten. Schließen Sie die Array-Ansicht der GUI, wenn ein Skript die Vorschau-Streams der einzelnen Kameras ausliest.
- **Verwenden Sie für Zahlen statt Bilder `frame_stream()`.**JPEG ist verlustbehaftet und 8-Bit; `frame_stream()` liefert dieselben Ergebnisse, wenn kein Codec im Pfad vorhanden ist.

### Intervallaufnahme für Drohnenvermessungen

Ein Vermessungsflug benötigt alle paar Meter Flugstrecke eine synchronisierte Bildgruppe, nicht jedes Einzelbild, das das Array erzeugen kann. Zwei Einstellungen steuern dies:**`target_fps`**legt fest, wie oft die Kameras ein Bild aufnehmen, und**`capture_repeated(interval_s=…)`** legt fest, wie oft ein Bild gespeichert wird.**Die Kameras streamen zwischen den Speichervorgängen weiter.** Bei jedem Intervall wird die neueste synchronisierte Gruppe gespeichert; die dazwischenliegenden Einzelbilder werden nicht geschrieben. Die automatische Belichtung passt sich bei jedem Einzelbild an, sodass die Belichtung zum Zeitpunkt der Speicherung bereits stabil ist. Es gibt keinen Modus, in dem die Kameras zwischen den Aufnahmen anhalten und für jede einzelne wieder aktiviert werden: Jede Aufnahme müsste dann warten, bis der Datenstrom startet und sich die Belichtung stabilisiert hat.**Passen Sie die Bildrate an die Anforderungen des Intervalls an.**Bei der Einstellung „Auto“ läuft ein Array mit der höchsten Rate, die der Host und das Netzwerk unterstützen. Eine niedrigere Einstellung reduziert die Wärmeentwicklung der Kameras, die Netzwerkauslastung und die Belastung der Host-CPU, ohne dass sich die gespeicherten Daten ändern. Wählen Sie eine Rate, die**2–4 Bilder pro Intervall** liefert:

- Bei jedem Speichervorgang wird auf das nächste Bild gewartet, nachdem es angefordert wurde; daher wird eine gespeicherte Gruppe erst bis zu einer Bildperiode nach dem geplanten Zeitpunkt gespeichert. Eine höhere Bildrate hält diese Verzögerung gering.
- Die automatische Belichtung passt sich einmal pro Bild an; daher verlangsamt eine sehr niedrige Bildrate die Reaktion der Belichtung auf vorbeiziehende Wolken.

**Berechnen Sie das Intervall anhand des Flugplans:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Beispielsweise entspricht eine Abdeckungsfläche von 60 m bei 80 % Überlappung nach vorne einem Abstand von 12 m; bei 10 m/s ergibt sich `interval_s=1.2`, und ein festgelegter Wert von `target_fps=3` ergibt zwischen 3 und 4 Bilder pro Intervall. Die Aufnahme erfolgt zeitbasiert: Das SDK verfügt über keinen entfernungs- oder GPS-gesteuerten Modus, halten Sie daher auf Vermessungslinien eine konstante Geschwindigkeit über Grund ein.

**Speichern Sie die Rohdaten im Fluggerät und kalibrieren Sie sie später.** `interval_s` wird ab dem Beginn jedes Speichervorgangs gemessen; daher führt ein Speichervorgang, der länger als das Intervall dauert, dazu, dass der nächste verspätet beginnt und der Abstand driftet. Das Speichern der Rohdaten zusammen mit dem Wert des Lichtsensors (`processing="raw"`; der Wert wird standardmäßig gespeichert) sorgt dafür, dass jeder Speichervorgang kurz bleibt und jedes Bild anschließend in Reflexionswerte umgewandelt werden kann.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**legt die Rate von Beginn an fest;**`arr.set_target_fps(fps)`**ändert sie während des Fluges ohne erneute Verbindung, und**`arr.set_target_fps(None)`** stellt sie wieder auf „Auto“ zurück. Eine Bildrate, die über der vom Host und Netzwerk unterstützten Obergrenze liegt, wird auf diese Obergrenze gesenkt – siehe `clamped` im Ergebnis – und die niedrigste Bildrate beträgt 0,1 fps.
- **Verwenden Sie**`burst()`**nicht zum Ausdünnen von Frames.** Es schreibt jeden Frame, den das Array erzeugt; es folgt zwar einem festgelegten**`target_fps`**, ist also nur dann das richtige Werkzeug, wenn jeder Frame gewünscht ist.
- **`target_fps` behebt keine beschädigten Frames.** Eine Verringerung dieses Werts reduziert die Anzahl der erzeugten Frames, nicht die Geschwindigkeit, mit der jeder einzelne gesendet wird; bei durch GVSP beschädigten Frames sollten Sie stattdessen `wire_ceiling_mbps` verringern (siehe [Array-Zustand](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### An ein bereits verbundenes Array anbinden — `attach_array`

Wenn das Array bereits aktiv ist (die GUI hat es geöffnet oder eine vorherige SDK-Sitzung hat `connect_array` aufgerufen), verwende stattdessen `attach_array`, um ein Handle darauf zu erhalten, anstatt die Verbindung neu herzustellen. `connect_array` führt in dieser Situation immer zu der Fehlermeldung „Kamera  <sn>ist bereits im Array<id>“ aus, da ein POST-Aufruf von `/array/connect` für ein Mitglied im Pool nicht idempotent ist; `attach_array` liest `/api/camera/array/list` und gleicht die Daten entweder anhand der Array-ID oder der Seriennummern ab.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Muster: SDK-Skripte, die gemeinsam mit der Desktop-GUI ausgeführt werden, sollten zunächst `attach_array` versuchen und auf `connect_array` zurückgreifen, falls sich noch kein Array im Pool befindet.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Ein angehängtes Array wird ausgeliehen, es gehört einem nicht.** Ein Handle von `attach_array` baut das Array niemals ab: Weder das Verlassen des `with`-Blocks noch ein expliziter `disconnect()`-Aufruf gibt es frei (der explizite Aufruf gibt eine Zeile an stderr aus, die besagt, dass das Array weiterhin läuft). Es gilt dieselbe Regel wie für `CameraSession` / `DAQSensorSession`. Daher ist die gemeinsame Nutzung mit der GUI sowohl mit als auch ohne `with` sicher:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Wenn Sie die Verbindung tatsächlich beenden möchten – um beispielsweise eine neue Verbindung mit einem anderen `wire_ceiling_mbps` herzustellen –, rufen Sie `arr.force_disconnect()` auf. Der Versuch, nach einem einfachen `disconnect()` an einem verbundenen Handle eine neue Verbindung herzustellen, schlägt mit der Meldung „Kamera  befindet sich<sn> bereits im Array <id>“, da nichts freigegeben wurde.

### Netzwerk-Analyse-Hilfsprogramm

Nützlich vor dem Öffnen des Arrays – zeigt an, ob Ihre vorgeschlagenen Einstellungen passen:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Es wird niemals ein Backend gestartet: Ein Backend, das nicht antwortet, löst `ChlorosConnectError` aus, während eine abgelehnte Anfrage als Ergebnis zurückgegeben wird. Ein fehlerhaftes `packed` löst `ValueError` aus.

`status` ist einer der folgenden Fehler: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (ansonsten `error` – einschließlich einer abgelehnten Anfrage: nicht angemeldet, `error_code: AUTH_REQUIRED` oder ein Tarif unterhalb der SDK-Mindestgrenze, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` bedeutet, dass die angeforderte Auflösung nur bei einer begrenzten Triggerrate in den RX-Ring passt – behalte die Auflösung bei und wechsle von `target_fps=result["recommended"]["recommended_target_fps"]` zu `connect_array` (siehe [Beispiel 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` erscheint nur, wenn der angeforderte Frame bei keiner Bildrate geliefert werden kann; „connect“ verkleinert Frames niemals von sich aus, wenden Sie daher `recommended` selbst an.

**So interpretieren Sie die Projektion** (gleiches Modell wie im GUI-Fenster „Array-Einstellungen“):

- **Burst (`frame_bytes_total`) wird pro Kamera im tatsächlichen Pixelformat der jeweiligen Kamera summiert.**Mono**M3M**-Kameras streamen Mono12 (2 B/px) oder, bei Verwendung eines `pixel_format`, das Mono-Format ihrer Tiefe (`"BayerRG8"` entspricht `Mono8`, 1 Bit/Pixel), sodass ein Vollauflösungs-Frame mit vier Kameras bei drei Mono-Kameras**~25 MB** groß ist und nicht die ~12,6 MB, die eine Annahme von durchgehend 8 Bit ergeben würde. Das Backend ermittelt das Format jeder Kamera anhand ihres Modells.
- **Admittance (`burst_fits_nic_ring`) berücksichtigt die Auslastungssituation**, nicht „Whole-Burst-vs-Ring“: „Sim-Emit“ passt, wenn der Host den RX-Ring schneller entleert, als die Kameras ihn füllen. Ein 10G-Host + 1-GbE-Kameras**lassen** volle Auflösung zu, selbst wenn der Burst den Ring übersteigt; ein 1-GbE-Host blockiert (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` ist eine konservative Obergrenze für den seriellen Abruf** — `max(readout+emit, N×emit)` mit pro Kamera auf den 1-GbE-Camera-Link begrenztem Ausgabewert, belichtungsunabhängig. Z. B. ~2,8 fps für ein Array mit 4 Kameras bei voller Auflösung und 12 Bit (entspricht den in der Laufzeit gemessenen ~2,7–3,0). Vollständiges Modell: [CLI-Referenz → Array-fps- und Burst-Modell](cli-reference.md#array-fps--burst-model).
- **Nach einer kontinuierlichen Aufnahme gibt die Projektion den Wert an, den sie beibehalten hat.** Eine Aufnahme von 30 s oder mehr bei einer automatischen Bildrate zeichnet die Bildrate auf, auf die sie sich bei dieser Array-Größe auf diesem Computer eingependelt hat; die kurzen Versuche des Bildratensteuerers mit einer höheren Bildrate werden dabei nicht berücksichtigt. Von da an geben die Zeile von `array-connect`Zeile, die Analyse von `connect_array()` und das Fenster „Array-Einstellungen“ geben diese Rate an, wenn sie unter der des Modells liegt, zusammen mit der Angabe, was sie in `cycle_limiter` begrenzt hat: `"clock"` (das Array lief mit der Rate, auf die es sich eingependelt hatte, wobei die Grab-Schleife auf ihren Trigger-Takt wartete), `"loop"` (der Grab-Schleife blieb keine Zeit mehr), `"capture"` (die Erfassung geriet in Verzug und übersprang Gruppen) oder `"measured"` (es gibt keinen Hinweis darauf, welcher Fall vorliegt). Der Wert des Modells ist in `wire_fps_max` angegeben. Das Array steigt weiterhin auf den Wert an, den die Leitung überträgt.
- **Eine Überbelegung (`oversubscribed: true`) bedeutet, dass die Leitung das Array bei keiner der von den Kameras erreichten Bildraten übertragen kann** – die fps-Felder (`achievable_fps_max` / `fps_bright` / `fps_dark`) den Wert 0 an, und Binning kann das Problem nicht beheben (es verringert die Bytes pro Bild, nicht die getakteten Bytes pro Sekunde). Eine langsame Verbindung allein führt nicht zu diesem Problem: Stattdessen sinkt die Bildrate, und wenn die Kameras nicht langsam genug getaktet werden können, wechselt die Stufe zu `slip-emit-and-capture` (seriell). Abhilfemaßnahmen sind weniger Kameras, Jumbo-Frames oder eine schnellere Netzwerkkarte; `max_cams_collision_safe` gibt die Obergrenze für Ihre Leitung und Kameras an. Die Antwort enthält außerdem `aggregate_demand_bps`, `collision_safe_ceiling_bps` und `per_cam_floor_bps`. Siehe [Überbelegung](#over-subscription-the-per-cam-floor).

### Erkennung und Auflistung

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Jede dieser Funktionen akzeptiert nur Schlüsselwörter (`backend_url=` und `timeout=30.0`) und gibt eine Liste von Dictionaries zurück. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) akzeptiert auch `auto_start_backend=True`; die Aufrufe von `list_*` starten niemals ein Backend.

---

## Smart-AE / Smart-Capture

LATTICE-Arrays führen im Hintergrund kontinuierlich AE aus, sobald sie verbunden sind, aber eine neu ausgerichtete Szene benötigt einen Moment, um zu konvergieren. **Smart-Capture** ist die komfortable Lösung: Es fragt die Belichtung jeder Kamera ab, wartet, bis das Array über ein Fenster hinweg stabil ist, und löst dann die Aufnahme aus. Es entspricht der GUI: Die Schaltfläche „Smart Capture“ der Desktop-App ruft denselben Backend-Endpunkt auf.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Bei der Steuerung über `ChlorosProject` (nächster Abschnitt) stehen Ihnen weitere Einstellmöglichkeiten zur Verfügung:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Die Smart-AE-Richtlinie ist standardmäßig konservativ eingestellt. Verengen Sie `exposure_tolerance_pct` für anspruchsvolle radiometrische Arbeiten; erweitern Sie sie für sich schnell ändernde Szenen, bei denen „nahe genug“ ausreicht.

---

## DAQ-Sensorsitzungen

Persistenter Backend-Pool für Spektralsensoren (DAQ-U über USB, DAQ-M über BLE, DAQ-E über Ethernet). Spiegelt die Kameraoberfläche wider: Smart-Detect, Wiederverwendung des Pools, idempotente Anbindung.

### Smart-Detect (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Priorität: Ethernet → BLE → USB. Übergeben Sie einen beliebigen expliziten Hinweis, um den Transport festzulegen.

### Festgelegter Transport

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` benennt einen Sensor, und nur dieser Sensor wird jemals geöffnet. Er wird mit den bereits geöffneten Sensoren abgeglichen; anschließend wird der Hostname oder die IP-Adresse eines DAQ-E (ein DAQ-E, den der Netzwerkscan nur über die IP-Adresse erkennt, wird schreibgeschützt nach seinem Namen, seiner MAC-Adresse und seiner Seriennummer abgefragt, sodass auch jeder dieser Sensoren ihn findet), die BLE-MAC-Adresse eines DAQ-M sowie die ID, die ein DAQ-U oder DAQ-M nach dem Öffnen meldet (jeder andere zur Überprüfung geöffnete Sensor wird wieder geschlossen). Wird er nicht gefunden, löst `connect_daq_sensor` ein `ChlorosConnectError` aus, das auflistet, was gefunden wurde; einschließlich einer Adresse (`port=` / `mac=` / `eth_host=`), die einen anderen Sensor enthält, löst ebenfalls einen Fehler aus. Es greift niemals auf den gerade sichtbaren Sensor zurück – dafür ist der Aufruf mit dem Zero-Flag vorgesehen.</id></sn></id></sn>

### `connect_daq_sensor()`-Signatur

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Ein Sensor, der sich bereits im Pool befindet, wird unverändert zurückgegeben (ausgeliehen): `integration_time`, `frame_avg` und `enable_ae` werden nicht auf ihn angewendet. Trennen Sie ihn zunächst, um ihn mit Ihren Einstellungen zu öffnen. `cap_id` bildet hier eine Ausnahme: Bei einem ausgeliehenen Sensor wird er mit `set_cap()` gesetzt. Wird er weggelassen, gilt die Standardeinstellung des Backends (siehe Hinweis unter der Methodentabelle).

### `DAQSensorSession`-Methoden

| Methode | Beschreibung |
| --- | --- |
| `status(timeout=10.0)` | Zusammenfassung des Pool-Eintrags (Streaming-/Aufzeichnungsstatus, Wellenlängenbereich, Kalibrierungs-SHA, Integrationszeit, frame_avg, AE-Status). Für einen DAQ-E, dessen Stream verstummt ist: `reconnect` (sofern die automatische Wiederverbindung des Backends funktioniert: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None`, solange es streamt), `stream_lost_s`, `reconnects` und `last_reconnect` sowie `record_gaps` für die laufende Aufzeichnung; `rx_stream_fault` gibt den Status der Wiederverbindung an, bis das erste Spektrum eintrifft. |
| `latest(*, n=1, timeout=10.0)` | Gibt die bis zu N neuesten Spektrum-Frames zurück (jeweils `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Streaming fortsetzen / anhalten (Handle bleibt offen). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Aufnahme einer .daq-Datei starten. Gibt den Dateipfad zurück. Wird bei DAQ-U/M ohne AWS-Kalibrierungspaket abgelehnt (DAQ-E ist davon ausgenommen). |
| `record_stop()` | Beendet die Aufzeichnung. Gibt `{path, rows, warning, gaps}` zurück; `warning` entspricht `None` bei einer fehlerfreien Aufzeichnung, ansonsten die Diagnose des Backends (z. B. eine leere Datei oder eine Lücke im Stream), die ebenfalls auf stderr ausgegeben wird. `gaps` listet jeden Abschnitt auf, in dem der Stream nichts geliefert hat: `start_utc`, `end_utc` (`None`: beim Stoppen noch verloren), `duration_s`, `reconnected`, `marked_in_file`. Bei einer fortlaufenden Aufzeichnung ist das Feld leer. |
| `set_cap(cap_id, timeout=15.0)` | Legt das Cap-Korrekturprofil fest, das ab dem nächsten Messwert angewendet wird: `"sunshine_cosine"`, `"fov_15"` … `"fov_90"` oder `"none"` (bzw. `None`) bei einem Sensor ohne Abdeckung. Gibt dieZusammenfassung des Sensors zurück, die von `cap_id` ausgelesen wird. Ein DAQ-E-S behält `as_recorded` bei, unabhängig davon, was abgefragt wird (sein Diffusor ist integriert). Löst `ChlorosConnectError` für ein Profil aus, das nicht geladen werden kann. CLI-Befehl: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Nur DAQ-E: Schaltet PTP wieder ein für ein Gerät, dessen PTP ausgeschaltet ist (`status()["rx_ptp_enabled"]` entspricht `False`; die Einstellung wird auf dem Gerät gespeichert, sodass PTP deaktiviert bleibt, bis es wieder aktiviert wird). Gibt den aktualisierten Status zurück. Löst bei einem DAQ-U / DAQ-M den Fehler `ChlorosConnectError` aus oder wenn das Gerät keine Bestätigung liefert. Chloros schaltet ein Gerät niemals von selbst wieder ein, da der ausgeschaltete Zustand beabsichtigt sein kann. CLIE: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Nur DAQ-E: die geräteeigenen Einstellungen. `firmware` (`running`, `bundled`, `update_available`), `ip` und `ip_mode` (`dhcp` / `linklocal` / `static`: was derzeit läuft), `network` (die gespeicherte Einstellung: `mode` sowie `ip` / `mask` / `gateway` im statischen Zustand), `network_change_pending` (gilt, bis das Gerät die gespeicherte Einstellung ausführt, eine, die `set_network(..., reboot=False)` enthält), `auto_stream`, `ptp`. `firmware["update_available"]` ist gleich `None`, wenn dieVersion nicht ausgelesen werden konnte. Solange der Datenstrom des Geräts diesen Computer nicht erreicht, wird er bei einer neuen Verbindung ausgelesen, und das Wörterbuch fügt `stream_silent` (`True`), `stream_fault` und `reconnect`; ein Gerät, das nicht antwortet, löst einen Fehler aus. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ– Nur E: DHCP (`"auto"`) oder eine statische Adresse. Das Gerät wird neu gestartet, um die Einstellung zu übernehmen, und erneut verbunden; diese Sitzung läuft weiter und `address` folgt dem Gerät. `reboot=False` speichert die Einstellung lediglich. Siehe [DAQ-E-Geräteeinstellungen](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Nur DAQ-E: Beim Einschalten ohne angeschlossene Software streamen (auf dem Gerät gespeichert). Gibt die bestätigte Einstellung zurück. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Nur DAQ-E: Gerät neu starten und mit den zuvor gespeicherten Einstellungen wieder anschließen. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Nur DAQ-E: Flashen Sie die in dieser Chloros-Version enthaltene Firmware auf ein Gerät, auf dem eine ältere Version läuft, und schließen Sie es anschließend wieder an (Dauer ca. eine Minute; schalten Sie das Gerät nicht aus). Gibt `updated`, `fw_before`, `fw_after`, `bundled` und ein `warning`, wenn das Gerät nur langsam wieder online ging oder mit einer anderen Version online ging; `updated` entspricht `False` für ein Gerät, das bereits auf dem aktuellen Stand ist. Löst `ChlorosConnectError` für ein Gerät aus, dessen Version nicht gelesen werden kann. CLI-Befehl: `pool-update-firmware`. |
| `disconnect()` | Freigabe aus dem Pool. Kein-op für angehängte, aber nicht im Besitz befindliche Handles. Bei einem Sensor, den diese Sitzung geöffnet hat, lautet der Befehl `force_disconnect()`, und es wird `ChlorosConnectError` ausgelöst, wenn die Freigabe nicht bestätigt wird (der Ausstieg aus dem Block `with` löst niemals eine Ausnahme aus). |
| `force_disconnect(timeout=30.0)` | Den Sensor freigeben, unabhängig davon, wer ihn geöffnet hat – zum Beispiel einen Sensor, den ein abgestürztes Skript im Pool zurückgelassen hat und auf den ein neues Skript nur als angehängtes Handle zugreift, dessen `disconnect()` nichts bewirkt. Das Beenden des Sensors eines anderen Clients beendet alle laufenden Aufzeichnungen. Kehrt zurück, sobald das Backend die Freigabe bestätigt oder wenn nichts freigegeben werden kann (der Sensor befindet sich nicht im Pool oder es läuft kein Backend); eine Freigabe, bei der eine Zeitüberschreitung auftrat oder die vom Backend abgelehnt wurde, löst `ChlorosConnectError` aus. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Attribute: `sensor_id`, `model`, `transport`, `address` (folgt auf ein DAQ-E, das von `set_network()` neu adressiert wurde), `info` und `backend_url`.

> **Bezeichnen Sie die Kappe.** DAQ-U, DAQ-M und DAQ-E werden ohne Kondensator ausgeliefert, und Chloros kann nicht erkennen, ob einer angebracht ist: Ein Sensor, der ohne `cap_id` angeschlossen wird, erhält den Wert `sunshine_cosine` – eine Annahme, die laut seinen Aufzeichnungen getroffen wurde (`cap_id_source=auto_default`). Ein ungeschützter DAQ-U liefert unter dieser Korrektur etwa 30-mal zu hohe Messwerte; übergebe daher `cap_id="none"` für einen ungeschützten Sensor (oder `set_cap("none")`) und das Profil der Kappe, die Sie ansonsten angebracht haben.

### DAQ-E-S – fester Diffusor und Neigungssensor

Der **DAQ-E-S** ist ein DAQ-E mit zwei Unterschieden, die für die Software von Bedeutung sind. Alles andere – Erkennung, Verbindung, PTP, Aufzeichnung, die unten aufgeführten Geräteeinstellungen – funktioniert genau wie bei einem DAQ-E.**Sein Diffusor ist fest eingebaut, daher ist keine Kappenkorrektur erforderlich.** Der Diffusor im „Sunshine“-Stil ist fest eingebaut und befand sich bereits am Gerät, als dessen Werkskalibrierung gemessen wurde, sodass diese Kalibrierung bereits die tatsächliche nach unten gerichtete Strahlungsintensität angibt. Chloros stellt seine Kappe unabhängig von der Abfrage auf `as_recorded` (kein Profil) ein: Eine nach ihm benannte Kappe (`sunshine_cosine`, `fov_*`, `none`) wird mit einer Meldung ignoriert, und in seinen Aufzeichnungen wird `cap_id_source=model` gespeichert, sodass die Warnung wegen eines angenommenen Caps bei diesem Gerät nie ausgelöst wird. Wenden Sie auch nicht manuell eine Obergrenze auf seine Daten an: `sunshine_cosine` würde in diesem Fall etwa das 11-Fache des Wertes anzeigen.

**Es verfügt über einen 3-Achsen-Beschleunigungsmesser.** Die Kosinus-Charakteristik des Sonnenstrahlungsdiffusors gilt nur, solange er waagerecht ausgerichtet ist; ein geneigter Sensor meldet daher eine falsche Bestrahlungsstärke – und die Reflektanz übernimmt diesen Fehler –, ohne dass weitere Symptome auftreten. Jeder DAQ-E-S-Messwert enthält daher seine Ausrichtung:

| Feld | Bedeutung |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Winkel in Grad. `None`, wenn die Firmware den Winkel als undefiniert markiert (freier Fall, eine tote Achse). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Beschleunigung in Milli-g. `mag_mg` – ein Wert weit entfernt von ~1000 bedeutet, dass sich das Gerät bewegte und die Winkel für diese Messung keine Aussagekraft haben. |
| `cal_applied` | „Set“: Die Achsen und Winkel liegen im Koordinatensystem des Diffusors (die Kalibrierung der Gerätebefestigung wird angewendet). „Clear“: Rohdaten im Koordinatensystem der Platine, die eine Neigung von bis zu ~2,3° anzeigen können, die nicht real ist. |
| `sample_age_ms` | Alter der Beschleunigungssensor-Probe zum Zeitpunkt der Spektrumübertragung. |
| `temp_raw` | Unkalibrierter **Trend** der Chip-Temperatur – nur Differenzen, niemals ein Thermometerwert. |

`DAQSensorSession.latest()` gibt diese unter dem Schlüssel `imu` jedes Frames zurück (`None` bei einem DAQ-U, DAQ-M oder einfachen DAQ-E). Ein von einem DAQ-E-S aufgezeichneter `.daq` speichert dieselben Werte in seinen Spalten `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) neben jedem Spektrum, sodass der Leser eine Wolke von einem umgestoßenen Stativ unterscheiden kann. Bei allen anderen Modellen sind diese Spalten leer.

Der Lage-Datenstrom meldet seine eigenen Felder in `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`) und sind von den Spektraldaten getrennt, sodass ein Ausfall des Beschleunigungssensors niemals als Lücke in der Aufzeichnung interpretiert wird. `discover_daq_sensors()` und das `model` der Sitzung geben `DAQ-E-S` an.

### DAQ-E-Geräteeinstellungen

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Es muss eine statische Adresse im Netzwerk dieses Computers vorhanden sein.** `set_network` löst `ChlorosConnectError` für einen Computer aus, dessen Subnetz die Adresse dieses Computers nicht enthältim Netzwerk des Sensors enthält: Nach dem Neustart kann der Sensor nicht mehr erreicht werden, um die Einstellung zurückzusetzen. Geben Sie `force=True` nur dann weiter, wenn dieser Computer auf das neue Subnetz umadressiert oder weitergeleitet wird.
- **`set_network`, `reboot` und `update_firmware` starten den Sensor neu** und lösen `ChlorosConnectError` aus, während er aufzeichnet, und wenn er nicht innerhalb von `comeback_timeout` Sekunden zurückkehrt. Die Sensor-ID und der Hostname `daq-e-<id>.local` ändern sich nie.
- **Eine kurz vor dem Neustart gestartete Aufzeichnung wird durch diesen beendet und benannt.** Wenn der Sensor wieder erreichbar ist, enthält das zurückgegebene Dictionary `recording_closed` (den Dateipfad) und einen Wert von `warning`, der dies angibt. Wenn er nicht zurückkehrt oder die Firmware-Übertragung fehlschlägt, gibt stattdessen der Wert `ChlorosConnectError` dies an: Die Meldung endet mit der Warnung, und `e.recording_closed` / `e.warning` gesetzt (beide `None`, wenn keine Aufzeichnung geschlossen wurde). Starten Sie die Aufzeichnung erneut, sobald der Sensor wieder verfügbar ist.
- **Es wird nur der Sensor selbst wieder verbunden.** Nach dem Neustart muss das Gerät dort antworten, wo es sich befand – und nach `set_network("auto")` auch unter seinem `.local`-Namen – mit seinem Namen, Hostnamen und Modell sowie der MAC-Adresse des Netzwerkadapters, über den es verbunden war; `address` wird zur IP-Adresse, die geantwortet hat. Eine Antwort unter dem Namen gilt nur mit dieser MAC-Adresse; daher wird ein Sensor, der per DHCP verschoben wurde, von einem Computer, der den Sensor über einen Router erreicht, oder über das Standard-NAT-Netzwerk von WSL2 nicht erneut verbunden (`ChlorosConnectError`; stellen Sie die Verbindung an seiner neuen Adresse erneut her).
- **Für ein Firmware-Update muss der Sensor die Verbindung wiederherstellen.** `update_firmware()` fordert das Gerät an, und das Gerät eröffnet eine TCP-Verbindung zurück zu diesem Computer, um das Image zu empfangen, und zwar auf einem vom Betriebssystem ausgewählten Port. Das Backend lauscht auf dem Adapter, der das Gerät erreicht (auf jedem Adapter, sofern es nicht feststellen kann, welcher es ist), und bedient nur dieses Gerät: Jede andere Verbindung wird abgelehnt (`[DAQ-FW] … refused a firmware connection from <ip>` in der Ausgabe des Backends), während es weiter wartet. Eine Firewall, die die Verbindung blockiert, führt zum Fehlschlagen des Updates mit der Meldung `device never connected back on TCP <port>`. Der Windows-Installer fügt die entsprechende Regel hinzu (siehe [Schritt 1](#windows-exe)); bei einer anderen Firewall müssen eingehende TCP-Verbindungen zum Backend aus dem Subnetz des Sensors zugelassen werden.
- **Verwenden Sie unter WSL2 „Mirrored Networking“** (`networkingMode=mirrored` in `.wslconfig`): Im Standard-NAT-Modus wird jede statische Adresse abgelehnt (die WSL-eigene Adresse befindet sich nicht im Netzwerk des Sensors), der Datenstrom erreicht WSL nicht und ein Firmware-Update kann nicht abgeschlossen werden.
- Die GUI unterhält eine eigene Verbindung zu einem Sensor: Ein Neustart über ein Skript beendet die Sitzung der GUI mit dem Sensor. Die GUI verfügt über dieselben Einstellungen (Registerkarte „Lichtsensoren“, Sensoreinstellungen, „Netzwerk“).

### Erkennung – eine Adresse für die Verbindung finden

`discover_daq_sensors()` durchsucht USB / BLE / ETH nach Sensoren, die Sie *öffnen könnten*. Es ist das DAQ-Pendant zu `discover_lattice_cameras()` und die einzige Möglichkeit, die **BLE-MAC-Adresse eines DAQ-M** zu ermitteln – ein DAQ-E hat einen Hostnamen und ein DAQ-U einen COM-Port, aber eine MAC-Adresse ist weder auf dem Gerät aufgedruckt noch wird sie vom Betriebssystem aufgelistet.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Feld | Beschreibung |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-Port / BLE-MAC / Hostname – an `connect_daq_sensor` als `port=` übergeben / `mac=` / `eth_host=`. |
| `display` | Für Menschen lesbare Bezeichnung. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, oder `None`. Ein Ethernet-Sensor ist `DAQ-E-S` oder `DAQ-E`, da er sich bei Abfrage selbst meldet, schreibgeschützt, während des Scans (`DAQ-E`, wenn er nicht rechtzeitig antwortet). Eine serielle USB-Schnittstelle wird als `DAQ-U` erkannt, wenn ihr Bridge-Chip von einem DAQ-U verwendet wird (FTDI, CH340, CP210x, Prolific) – ein Hinweis aus der Beschreibung der Schnittstelle, keine Sonde — und `None` für jedes andere USB-Gerät, das weiterhin aufgeführt ist, sodass man es als `port=` übergeben kann. Ein serieller Port ohne USB-Hersteller-ID und ohne einen solchen Bridge-Chip (ein UART auf der Hauptplatine wie Linuxs `/dev/ttyS*`, eine serielle Bluetooth-Verbindung, Intel AMT) kann kein DAQ-U sein und wird nicht aufgeführt. |
| `extra` | Details pro Transport (BLE-Advertised-Name; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E IP/FW/…). Leere Werte werden weggelassen. |

| Parameter | Standardwert | Beschreibung |
| --- | --- | --- |
| `transports` | alle drei | Sequenz (oder CSV-Zeichenkette), die den Scan begrenzt. Sollte übergeben werden, wenn Sie wissen, was Sie wollen – BLE ist dabei der flashen Teil. |
| `scan_timeout` | 5 | Scan-Fenster pro Transport in Sekunden; das Backend begrenzt den Wert auf 1–20. |
| `timeout` | 60,0 | HTTP-Obergrenze für den gesamten Aufruf (wie auch an anderer Stelle im SDK). |
| `auto_start_backend` | `True` | Startet ein lokales Backend, falls noch keines läuft. Wird niemals für ein entferntes `backend_url` gestartet. |

> **Sensoren, die bereits im Pool geöffnet sind, werden nicht angezeigt.** Ein verbundenes BLE-Peripheriegerät stellt die Werbung ein, und ein offener COM-Port kann nicht abgefragt werden; daher listet die Erkennung nur das auf, was *für eine Verbindung verfügbar* ist. Ein leeres Ergebnis unmittelbar nach dem Anschließen eines Geräts ist zu erwarten – verwende `list_daq_sensors()` für bereits vorhandene Geräte. Transporte, deren Scan nicht ausgeführt werden kann (kein bleak / zeroconf installiert), werden übersprungen, anstatt einen Fehler auszulösen, sodass ein Rechner ohne Bluetooth dennoch seine USB- und ETH-Antworten erhält.

### Auflistung

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Welches DAQ versorgt welche Kamera – `daq_bindings()`

Die Reflexionswerte einer Aufnahme und die dabei gespeicherte Datei `.daq` verwenden das an diese Kamera oder dieses Array gebundene DAQ, andernfalls das aktuellste DAQ, über das das Backend verfügt. Die Auswahl des Lichtsensors in der GUI und die Datei `chloros-cli daq pool-bind` legen dies fest; ebenso wie:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` gibt `{"cameras": {serial: b}, "arrays": {array_id: b}}` zurück: Jedes `b` enthält `binding` (den festgelegten Wert), `source` (`project`, aus dem `sensors.json` des Projekts oder `runtime`), `known` / `known_as` (ob der Schlüssel „Kamera“ oder „Array“ einen Namen bezeichnet, den Chloros kennt, und wie: `connected`, `project` – der `cameras.json` des geöffneten Projekts — oder, bei einer Kamera, `seen`: wurde zuvor auf diesem Computer geöffnet; `known: False` für einen Schlüssel, der nichts benennt) und `resolved` (der Pool-Sensor, dem es jetzt zugeordnet ist: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Eine Laufzeitbindung hat Vorrang vor den Projekt-für die Sitzung; die Projektdatei wird nicht verändert. Das Trennen einer Kamera, eines Arrays oder eines DAQ-Geräts hebt alle Bindungen dieser Hardware auf, unabhängig davon, wer sie eingerichtet hat (die Elemente eines Arrays werden mit dem Array selbst verknüpft).

`set_daq_bindings()` bindet nur eine Kamera, die angeschlossen ist, sich im geöffneten Projekt befindet oder zuvor auf diesem Computer geöffnet wurde, sowie ein Array, das angeschlossen ist oder sich im geöffneten Projekt befindet (oder eine bereits gebundene Taste). Jede andere Taste löst `ChlorosConnectError` aus, wobei sie (`400`, `code`, `UNKNOWN_BINDING_TARGET` sowie die Schlüssel `unknown`), und **nichts** in diesem Aufruf wird gebunden. `None` löscht alle Schlüssel.

### Umgebungs- + Objektreflexion — `light_sensor_reflectance()`

Zwei zusammengeführte DAQs, von denen eine zum Himmel (Umgebungslicht) und eine zum Zielobjekt (Objekt) gerichtet ist, liefern die Reflektanz und die Vegetationsindizes des Zielobjekts anhand ihrer letzten Messwerte: das Paar „Umgebungslicht + Objekt“ in der GUI sowie `chloros-cli daq reflectance`, ohne die Sensoren ein zweites Mal zu öffnen. Beide müssen verbunden sein und Daten streamen.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Bänder.** Die Reflektanz jedes Bandes ist die bandintegrierte Strahlungsintensität des Objektsüber der Umgebungsstrahlung. `daq_band_presets(filter_code=None)` listet die Band-Sätze auf: einen pro LATTICE M3C-Filter (`RGN`, `OCN`, `NGB`, das kameraspezifische Bandmodell; daher sind ein DAQ-NDVI und ein Kamera-NDVI identisch) sowie `GENERIC` (Breitband). Das `suggested` ist die Voreinstellung, die mit `filter_code` übereinstimmt, andernfalls der Filter der ersten angeschlossenen Kamera (`suggested_from` gibt an, welcher).
- **Gibt zurück** `bands` (pro Band `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (die integrierten-Ins), `custom` (jeweils mit `value` oder `valid: False` und dem `error`), `invalid_bands`, `reflectance_type`, `preset` sowie die beiden Messwerte `timestamps` (`ts_ns`). Ein Messbereich, den der Umgebungssensor nicht erfassen konnte (außerhalb seines Messbereichs oder an seiner Rauschgrenze), wird als ungültig markiert, wobei der Grund angegeben wird; alle Indizes, die diesen Wert benötigen, werden ausgelassen und niemals auf Null berechnet.
- **Scheinbar oder relativ.** Die skalierungsabhängigen Indizes (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) benötigen ein absolutes Verhältnis; daher werden sie nur dann bereitgestellt, wenn beide Sensoren leistungskalibriert sind (jeder verfügt über ein `calibration_bundle_sha`): `"Apparent Reflectance"`. Andernfalls lautet das Ergebnis `"Relative Reflectance"`, wobei nur die skalierungsinvariante Indizes enthalten sind.
- **Zwei verschiedene Wellenlängengitter** werden hinsichtlich ihrer Überlappung verglichen: die Punkte des Umgebungssensors innerhalb der Reichweite des Objekts, das darauf interpolierte Spektrum des Objekts.
- **Zwei Sensoren, niemals nur einer.** Die Benennung eines Sensors als beide (gleiche ID, in jeder vom Pool akzeptierten Schreibweise) führt zu `ChlorosConnectError`: Der Messwert eines Sensors über sich selbst beträgt in jedem Band 1,0, was nichts misst. Auch das Backend lehnt dies ab (`POST /api/daq/indices` antwortet mit `400`, `code` auf `SAME_SENSOR`, während die vom SDK gesendeten `ambient_sensor_id` und `object_sensor_id` einen gepoolten Sensor darstellen, z. B. dessen ID und dessen Port).

### Gemeinsame Nutzung mit der CLI und anderen Skripten

Der DAQ-Sensorpool des Backends wird von jedem CLI- und SDK-Client auf dem Rechner gemeinsam genutzt. Befindet sich ein Sensor bereits im Pool – geöffnet durch `chloros-cli daq pool-connect` oder durch ein anderes Skript –, gibt der Aufruf von `connect_daq_sensor(port="COM3")` aus Python ein ausgeliehenes Handle zurück: Der Aufruf von `disconnect()` ist eine No-Op, sodass Ihr Skript den Sensor beim Verlassen des Scopes nicht dem Client entreißt, der ihn geöffnet hat. Das Gleiche gilt für `sensor_id=` (eingegeben mit `:` oder `-`, beides passt), für einen DAQ-E, das beim Öffnen über den Hostnamen anhand seiner IP-Adresse benannt wurde (oder umgekehrt), sowie für die Smart-Detect-Funktion mit Null-Flag, wenn das Backend mit einem Sensor antwortet, den es bereits hielt – ein `with`-Block gibt immer nur einen Sensor frei, den er selbst geöffnet hat.

Die Chloros-GUI ist **nicht** Teil dieses Pools: Ihr DAQ-Panel unterhält eine eigene Verbindung zum Sensor, sodass `connect_daq_sensor` niemals einen Sensor ausleiht, den die GUI geöffnet hat. Verwenden Sie denselben Sensor nicht gleichzeitig über die GUI und über ein Skript. Ein DAQ-U oder DAQ-M, den die GUI hält, lässt sich normalerweise nicht öffnen, ein DAQ-E akzeptiert jedoch eine zweite Sitzung: Der Verbindungsaufbau des Skripts ändert die Erfassungseinstellungen des Geräts unter der GUI, und sein `disconnect()` stoppt den Datenstrom des Geräts, wodurch die Live-Ansicht der GUI und alle laufenden Aufzeichnungen beendet werden. Schließen Sie den Sensor zuerst in der GUI.

### PTP-Grandmaster — `time_sync_status()`

Das Backend betreibt den PTP-Grandmaster, von dem LATTICE-Kameras und DAQ-E-Geräte ihre Zeitstempel beziehen. Es wird bei Bedarf gestartet: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` und `connect_all()` starten ihn, falls er nicht läuft – einschließlich einer Verbindung, die anschließend fehlschlägt, wie beispielsweise bei `connect_all()`, das versucht, eine nicht vorhandene DAQ-E-Einheit anzusprechen – ebenso wie `test_array_sync()` und jeder Aufruf, der Kameras für einen Befehl öffnet (`capture_cameras_transient()`, `start_multi_camera_stream()`, die Ausrichtungsaufrufe, `capture_live_index()`).

- **Wo es sich meldet.** Auf den kabelgebundenen Netzwerkadaptern, die mit den Kameras verbunden sind (einschließlich einer `169.254.x.x`-Link-Local-Adresse), andernfalls auf dem Adapter der Standardroute, sofern dieser kabelgebunden ist. Niemals über WLAN oder einen virtuellen Adapter (Tailscale, WSL): Dahinter befindet sich keine Kamera und kein DAQ-E. Ist kein solcher Adapter aktiv, läuft der Dienst ohne Ankündigung (`health` `NO_SOCKETS`) und bindet sich an den Adapter, der beim nächsten Verbindungsaufbau einer Kamera oder eines DAQ-E aktiviert wird.
- **Nach einem Verbindungsabbruch.** Wenn der Kameraadapter seine Verbindung verliert und wiederherstellt (Kabelzug, Austausch eines Switches, erneute Erkennung eines USB-Adapters), baut der Grandmaster seine Sockets selbstständig wieder auf, ohne Neustart, und überprüft jedes angeschlossene DAQ-E erneut, wobei er diejenigen, die während der Unterbrechung hängen geblieben sind, wieder aktiviert. Er baut die Verbindungen auch neu auf, wenn alle von ihm bedienten Kameras und DAQ-E 15 Sekunden lang inaktiv sind.
- **Erreicht er die Leitung?** Lies `time_sync_status()` (`GET /api/time-sync/status`). `state` gibt nur an, dass der Grandmaster-Thread läuft; `health` gibt an, ob seine Pakete gesendet werden: `OK` (kein Fehler festgestellt), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (einige Adapter fallen aus) oder `PEERS_SILENT` (die von ihm bedienten Kameras und DAQ-Geräte antworten nicht mehr), mit einem Satz in `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` zählen nur noch die Nachrichten, die gesendet wurden; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` und `last_delay_req_age_s` beschreiben diejenigen, bei denen dies nicht der Fall war, und `rebinds` / `last_rebind` die Neuaufbauten. `sync_sends_slow` zählt die Synchronisierungen, deren Sendevorgang länger als 200 µs dauerte, und `sync_send_max_us` ist der längste; die Ursprungszeit jedes Follow_Up zählt höchstens 200 µs seines Sendevorgangs. Der Grandmaster läuft in einem eigenen Prozess (`own_process`, `process_pid`) mit hoher Priorität unter Windows (`process_priority`), sodass ein ausgelastetes Backend die Zeitstempel nicht verzögern und die Uhren aller Kameras gleichzeitig nicht verschieben kann; wenn dieser Prozess nicht gestartet werden kann, läuft er wie bisher innerhalb des Backends (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funktion | Was sie tut |
| --- | --- |
| `time_sync_status()` | Der Zustand, der Integritätsstatus, die Uhr-ID, die Prioritäten und die Nachrichtenzähler des Grandmasters (siehe oben). `chloros-cli time-sync status`. |
| `time_sync_peers()` | Die PTP-Slaves, die er empfängt (Kameras, DAQ-Es): `peers`, jeweils `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Die -Nummern jeder angeschlossenen Array-Kamera: `PtpStatus`, `PtpOffsetFromMaster` und `PtpMeanPathDelay` (eine Kamera, deren Firmware keine PTP-Knoten enthält, verfügt nur über die Seriennummer). `time-sync cameras`. |
| `restart_time_sync()` | Den Grandmaster anhalten und neu starten; das PTP jedes im Pool befindlichen DAQ-E wird danach erneut überprüft (`daq_ptp_rechecked`). Löst `ChlorosConnectError` aus, wenn es nicht zurückkehrt. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Seine PTP-Prioritäten (0–255, der niedrigere Wert gewinnt die Wahl zum Best-Master) für ein Netzwerk mit einem anderen PTP-Master. Dieser muss laufen. `time-sync set-priority`. |

Jedes nimmt `backend_url=` und `timeout=` und startet ein lokales Backend, das noch nicht läuft.

---

## Kamera-Dienstprogramme

Was `chloros-cli lattice` über das Herstellen einer Verbindung und die Erfassung hinaus leistet, sind einfache Funktionen in `chloros_sdk` (Modul `chloros_sdk.camera_utils`). Jede ruft die Backend-Route auf, die der CLI-Befehl verwendet, und zwar mit den Regeln der CLI, sodass sich beide gleich verhalten. Sie nehmen eine Kamera-**Seriennummer** entgegen, keine Sitzung, sowie dieselben Schlüsselwörter wie die „smart-connect“-Aufrufe (`backend_url=` / `timeout=`); ein nicht laufendes Backend wird gestartet, wie es auch bei `connect_camera()` der Fall ist. Bei einer Ablehnung wird `ChlorosConnectError` mit dem Grund des Backends ausgelöst.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funktion | Was sie tut |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Das Farbprofil des Displays: `raw`, `linear`, `natural`, `enhanced` oder `custom_temp` sowie dessen Farbtemperatur (begrenzt auf 2000–10000 K; wird ausgelesen, wenn das Profil `custom_temp` ist). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Bildschirmsättigung und Kontrast, jeweils 0–200 % (100 = unverändert); `reset=True` setzt beide Werte wieder auf 100 zurück. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Die hellsten, weißesten Pixel der Bildschirmanzeige nach unten korrigieren: -100..0 (in Richtung Null abgeschnitten, dann begrenzt; 0 = aus). |
| `camera_white_balance_once(serial, settle_s=None)` | Einmaliger Weißabgleich: Die Kamera überträgt kurz, konvergiert und gibt `red_ratio` / `blue_ratio` (`ratios_read` „False“, wenn sie diese nicht melden kann). Sie behält den ermittelten Weißabgleich bei; alle anderen Einstellungen, die ihr „open“ geschrieben hat, werden unverändert zurückgegeben. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (kontinuierlich) oder `"off"`; oder feste `red` / `blue`-Ausgleichsverhältnisse, bei denen die Automatik zuerst ausgeschaltet wird. Ein Verhältnis, das die Kamera ablehnt, wird unter `errors` benannt. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Die Voreinstellungen der Kamera; bei Anwendung einer davon behält die Kamera die Einstellungen bei, die die Voreinstellungen benennen (die Antwort lautet `settings`, da die Kamera diese anschließend ausliest, z. B. die Bildrate, mit der sie läuft; `None` für ein Feld, das in der Voreinstellung nicht benannt ist), und alle anderen Einstellungen, die in der Datei „open“ geschrieben wurden, werden so zurückgesetzt, wie sie vorgefunden wurden. `"default"` setzt die Kamera auf die Werkseinstellungen zurück (Werkseinstellungen mit frei laufender automatischer Belichtung, deaktivierter Link-Durchsatzbegrenzung und deaktivierter Fehlerkorrektur, wobei der kalibrierte Weißabgleich einer RGB-Kamera beibehalten wird), die nicht im Speicher der Kamera gespeichert werden: Die Antwort enthält `factory_defaults: True` und jedes Feld von `settings`. In `camera_presets()` ist `default` gleich `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Die Filtermodelle und die Details eines Modells (Bänder, Konditionszahl der spektralen Empfindlichkeit). Das Filtermodell einer Kamera wird aus ihrer Modellzeichenfolge ausgelesen; es wird nicht manuell festgelegt. |
| `camera_firmware_status(serial=None)` | Die Firmware der jeweiligen KameraFirmware: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` und `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Die Firmware dieses Chloros-Builds auf eine Kamera flashen: Werksreset, Flashen, Neustart, die Chloros-Grundeinstellungen werden neu geschrieben; die Kalibrierung der Kamera bleibt erhalten. Dauer: einige Minuten; die Kamera muss mit Strom versorgt bleiben. `force=True` führt ein erneutes Flashen einer bereits installierten Kamera durch. |
| `camera_power_probe(serial)` | Die Strom- und Temperaturknoten der Kamera sowie deren Temperatur. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Der Energiesparmodus der Kamera im Leerlauf (sie bleibt im Ruhezustand, bis sie geweckt wird: die Begrenzung des Link-Durchsatzes ist aktiviert (`DeviceLinkThroughputLimitMode` `On`) auf ihrem Mindestwert, der Energiesparmodus ist aktiviert, sofern die Kamera über einen solchen verfügt), mit der Temperatur vor und nach dem Aufwachen; beim Aufwachen wird die Verbindung wieder so hergestellt, wie sie im Ruhezustand vorgefunden wurde (die Obergrenze, der Modus und die Paketverzögerung `GevSCPD`, die die Kamera auf 0 hält, solange die Obergrenze aktiv ist; die Aufzeichnung wird auf diesem Computer gespeichert, sodass auch ein späteres Backend beim Aufwachen darauf zugreifen kann), oder bei einer Kamera, die dieser Computer nicht in den Ruhezustand versetzt hat, die aber im Ruhezustand zu sein scheint (ihr Limit ist auf dem Mindestwert aktiviert), wird das Limit deaktiviert und ein `GevSCPD` bei 80 auf 0 belassen (die Werkseinstellungen); Ein über dem Mindestwert festgelegter Grenzwert bleibt unverändert. Nur der Energiezustand wird beibehalten; alle anderen Einstellungen, auf die der „open“-Befehl zugreift, werden auf den ursprünglichen Wert zurückgesetzt. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Bilder aufnehmen, bis ein MAPIR-Kalibrierungsziel im Blickfeld ist, und die kanalbezogenen Koeffizienten aus dessen Feldern anpassen: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Die Ethernet-Adapter dieses Computers für GigE Vision (MTU, Verbindungsgeschwindigkeit, Jumbo-Frames) sowie alle `issues`; `estimate=True` fügt die Bildrate und Bandbreite hinzu, die `cameras`-Kameras erhalten würden. |
| `fix_network_settings()` | Wendet die GigE-Vision-Adaptereinstellungen auf denKameraadapter anwenden. Dies ändert die Einstellungen des Adapters und funktioniert nur, wenn Chloros als Administrator ausgeführt wird; andernfalls wird ein Fehler mit einem Protokoll der ermittelten Ergebnisse ausgegeben. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Führt eine Vignettierungskorrektur für die Bilder in einem Ordner durch, wobei für jedes Bild die Flat-Field-Karte des jeweiligen Objektivs verwendet wird: `processed`, `lens_models` (das Objektiv, für das jede Datei korrigiert wurde), `skipped` (mit Begründungen). Ohne `lens_model` wird das Objektiv jeder Datei aus dem EXIF-Modell entnommen; ein `lens_model` gilt für Dateien, deren Modell keine Kamera angibt, und eine Datei, deren Modell dem widerspricht, wird übersprungen. Nur Dateien. |
| `read_dls(mac, filter=None, camera_serial=None)` | Ein Messwert von einem DLS (Bluetooth-Sensor für einfallendes Licht): Spektrum, Bestrahlungsstärke pro Band und Bandamplituden anhand der kameraspezifischen Empfindlichkeitskurve, wenn `camera_serial` angegeben wird. `discover_daq_sensors(transports=["ble"])` ermittelt die MAC-Adresse. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Die Hardware des Hosts für Kameras und die von ihm empfohlene Konfiguration (`report_text`) sowie die CPU, GPU und PyTorch-Laufzeitumgebung des Systems. |

- **Einige benötigen die Kamera exklusiv für sich.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` und `update_camera_firmware()` öffnen die Kamera. Solange Chloros sie hält (in der App verbunden, mit `connect_camera()` als Array-Element oder gerade durch einen anderen Befehl belegt, wie z. B. bei einem Array-Test oder einer Aufnahme, bei der die Kameras geöffnet werden), lehnt das Backend dies mit dem HTTP-Statuscode 409 `CAMERA_HELD` ab (sowohl in `code` als auch in `error_code`), wobei angegeben wird, wie die Freigabe erfolgen soll. Die Farbeinstellungen, die Highlights, `camera_power_probe()` und die Auslesungen funktionieren bei einer angeschlossenen Kamera.
- **Farbe, Highlights und Weißabgleich gelten für RGB-Kameras.** Bei einer Single-Band-Mono-Kamera (M3M) ändern die Setter nichts und geben `{"skipped": True, "reason": ...}` zurück, genau wie die CLI. Es handelt sich um Anzeigeeinstellungen: die Live-Ansicht und Aufnahmen auf Anzeigeebene, niemals Strahlungsintensität oder Reflexionsgrad.

---

## Transiente Arrays, Ausrichtungsprofile und Indizes

Was `chloros-cli lattice` ohne dauerhafte Verbindung tut, wie Funktionen in `chloros_sdk` (Modul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` und `index`. Jeder sendet die Anfrage, die der CLI-Befehl sendet, mit den Standardwerten der CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

– **Sie öffnen die Kameras selbst.** Jeder Befehl öffnet seine Kameras, führt seine Aufgabe aus und schließt sie wieder; daher benötigt er Kameras, die Chloros nicht hält (in der App verbunden, mit `connect_camera()` oder als Array-Element verbunden sind); das Backend lehnt dies mit einem HTTP-Fehler 409 (`CAMERA_HELD`) ab und gibt an, wie eine Kamera freigegeben werden kann. Ein vorübergehendes Öffnen dauert eine Weile (etwa 40 Sekunden für einige Kameras). Für wiederholte Aufnahmen ist ist `connect_array()` einmalig der schnellste Weg und spart bei jedem Exporttyp Zeit, während eine vorübergehende Aufnahme ein debayertes Bild speichert. `calibrate_alignment_profile()` ist die Ausnahme: Bei einem verbundenen Array dieser Kameras wird die Lösung aus den eigenen Bildern des Arrays ermittelt (`used_live_array`).
- **`serials`** sind die zu öffnenden Kameras, wobei die erste den Auslöser betätigt; `None` öffnet jede Kamera im Netzwerk.
- **`settings`** ist ein Dict mit Überschreibungen der Kameraeinstellungen, deren Felder durch die Flags der CLI festgelegt werden: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` und `preset` (eine Kamera-Voreinstellung mit diesem Namen). Eine angegebene Belichtung oder Verstärkung deaktiviert diese automatische Steuerung, ebenso wie die Flags.
- **`gpu`**: `True` erzwingt den GPU-Warp, `False` den CPU-Warp (`--gpu` / `--no-gpu`); `None` überlässt die Wahl dem Backend.
- **`keep_settings`**/**`clear_settings`** (jede Funktion, die Kameras öffnet, Standard: `False`): Am Ende wird jede Kamera wieder in den Zustand versetzt, in dem sie vorgefunden wurde; `keep_settings=True` speichert die Einstellungen des Durchlaufs stattdessen im eigenen Speicher der Kamera (Freilauf), `clear_settings=True` setzt diesen Speicher auf die Werkseinstellungen zurück – siehe [Kameraspeicher](#camera-memory--keep_settings--clear_settings). Die Antwort (für einen Auftrag, d. h. die von `stop()`) enthält `camera_memory`.
- **Pfade sind die des Backends**: Relative Pfade werden relativ zu Ihrem Arbeitsverzeichnis in absolute Pfade umgewandelt, wenn sich das Backend auf diesem Rechner befindet. Die Funktionen für die Alignment-Profil-Datei (`alignment_profile_info()`, `reorder_alignment_profile()`) lesen die Datei auf diesem Rechner, genau wie die CLI.

| Funktion | Was sie tut |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Nimmt kontinuierlich von einer Kamera auf, bis der Vorgang gestoppt wird. Gibt einen `CliJob` zurück. Die Frames eines `tiff`-Laufs sind die `raw`-Datei, die ein Array-Capture dieser Kamera schreibt (eine Mosaikebene mit deneigenen Werten, unkomprimiert); `png` und `jpg` enthalten jeweils 8 Bit, daher handelt es sich bei diesen Durchläufen um demosaizierte Bilder zur Anzeige. Ein gebundener oder gepoolter DAQ schreibt pro Bild des Durchlaufs ein `.daq` in den`daq/`-Ordner des Laufs (`daq_files` in den Statistiken). Jedes Bild wird einmal mit seinen vollständigen Metadaten geschrieben (Modell, Aufnahmezeit, Objektiv, Bänder, Kalibrierung auf einem Rohbild); `stop()` wird zurückgegeben, sobald jedes bereits erfasste Bild geschrieben wurde (bis zu 25 s), und erst dann wird die Kamera geschlossen. Eine Kamera, die für einen Hardware-Trigger bereitgestellt wurde (`TriggerMode` `On`) wird für den Durchlauf auf „Free-Run“ gesetzt (die Antwort „`trigger`“ bei Start gibt Auskunft darüber, was vorgefunden wurde). Beim Stopp wird die Kamera wieder in den Zustand versetzt, in dem sie vorgefunden wurde – einschließlich ihrer Auslöseeinstellungen und aller während des Durchlaufs geänderten Einstellungen –, es sei denn, `keep_settings=True` wird verwendet, wodurch die Einstellungen des Durchlaufs im kamereneigenen Speicher (Freilauf) gespeichert werden, oder `clear_settings=True`, das diesen Speicher zurücksetzt. Ein Lauf, bei dem kein Bild aufgezeichnet wird, beendet sich nach 10 s mit `stop_reason` und `no_frames`, wobei der Grund in `last_error`, und sein `stop()` gibt diesen Grund als `error` zurück. Frames, die verloren gingen, weil die Schreiber nicht mithalten konnten, werden gezählt, wobei `drop_reason` den Grund angibt. Für eine von Chloros gehaltene Kamera verwenden Sie `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Synchronisierte Frame-Gruppen im Stream bei `fps`, für `count` Gruppen oder bis zum Stopp. Gibt ein `CliJob` zurück (dessen `start_reply` `n_cams` und den Datenpfad `pacing` enthält); seine abschließenden Statistiken zählen die erfassten, geschriebenen und verworfenen Gruppen, die Dateien, die der Writer nicht schreiben konnte, sowie die Gruppen, für die eine Kamera nichts geliefert hat. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Eine synchronisierte Aufnahme, ein debayertes Bild pro Kamera: `results` (`filepath`, `serial`, `frame_id`) und `pacing`. Eine Gruppe, in der eine Kamera fehlt, wird angezeigt, wobei zunächst angegeben wird, wie viele Dateien gespeichert wurden. `lattice multi-capture` ohne angeschlossenes Array. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` – synchronisierte Aufnahmen bei fester Belichtung: `sync_groups`, `master`, eine `group_timestamps`-Zeile pro Auslösung (eine Kamera, die eine Aufnahme verpasst hat, fehlt in ihrer Zeile) und `sync`: Versatz jeder zweiten Kamera gegenüber der Master-Kamera, Auslösevorgang für Auslösevorgang (`offsets_us`: pro Kamera `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), auf EINEM Takt (`clock_domain`: `ptp` oder `device` – PTP ausgeschaltet, jede Kamera hat ihre eigene Betriebszeit-Uhr, daher entsteht der Versatz beim Hochfahren, und nur der Jitter misst die Synchronisation). Ein Trigger, dessen Zeitstempel die Geräτουhr und PTP mischen (PTP deaktiviert oder noch in der Synchronisationsphase), wird ausgelassen und gezählt (`groups_mixed_domain`), ebenso wie einer, bei dem eine PTP-Uhr noch mehr als `agree_within_us` vom Master abweicht (`groups_clocks_disagree`); `notes` geben dies an. `clock_offset_mean_us` / `clock_offset_jitter_us` sind der größte Mittelwert und der größte Jitter pro Kamera. `success` ist falsch, ebenso wie `error`, wenn ein Trigger verpasst wurde, wenn weniger als zwei Trigger bei jeder Kamera auf einem Takt liegen oder wenn die Takte nie übereinstimmen (`sync.problems`); die Zeilen werden in jedem Fall zurückgegeben. Es startet den PTP-Grandmaster, falls dieser nicht läuft (`time_sync` gibt `grandmaster_running` und `grandmaster_started_by_this_test` an), schaltet das PTP jeder Kamera ein (wie es `connect_array()` tut) und wartet vor dem ersten Trigger darauf, dass sich der PTP-Takt jeder Kamera auf den Grandmaster einpendelt und stabilisiert: `PtpStatus` „Slave“ auf einem Grandmaster, innerhalb von 100 µs davon und 3 s lang stabil, und jede Kamera ist auf diese Weise synchronisiert, ohne Unterbrechung, für 120 s – etwa 2 Minuten nach dem Einschalten. Ein Takt, der sich gerade synchronisiert hat, weicht nach dem Lesen von „Slave“ etwa 2 Minuten lang um bis zu ~200 us ab, und das eigene `PtpOffsetFromMaster` der Kamera zeigt dies nicht an; Nach 120 s liegen die Kameras innerhalb von ~10 µs eines stabilen Taktes. Eine Kamera, die seit einem früheren Test, bei dem PTP eingeschaltet blieb – so wie es dieses Backend sah –, noch immer synchronisiert ist, zählt ab dieser Synchronisation weiter, sodass bei zwei direkt aufeinanderfolgenden Tests keine erneute Wartezeit entsteht; eine Kamera, die durch den Test eingeschaltet wird, oder eine Kamera, bei der PTP aktiviert ist, deren Synchronisierung dieses Backend jedoch noch nicht erkannt hat (die an anderer Stelle oder vor einem Neustart des Backends eingeschaltet wurde), wartet ihre 120 s ab, und in einem Array mit beiden Kameras warten die gerade synchronisierten Kameras weiterhin ihre Zeit ab. Eine Kamera, die den Slave-Modus verlässt, 100 µs überschreitet oder 3 Messwerte in Folge einem anderen Grandmaster folgt (1,5 s) während der Wartezeit folgt, beginnt erneut mit dem Zählen; ein oder zwei solcher Messwerte werden protokolliert (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) und der Zählvorgang läuft weiter, und eine Grandmaster-ID, die nicht gelesen werden kann, wird als unbekannt gewertet, nicht als Änderung. Wenn der Zählvorgang aller Kameras gleichzeitig (innerhalb einer Sekunde) neu startet, liegt dies an der Zeitverschiebung des Grandmasters, nicht an der der Kameras: Das Protokoll des BackendsProtokoll lautet `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` listet es in `grandmaster_steps` auf, und eine Wartezeit, die dann abläuft, beginnt ihren Fehler mit `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Die Grenze wird dafür nicht erweitert. `ptp_settle_s` (0–600; `0` ohne Abwartungszeit) legt die Abwartungszeit für den Lauf fest; ein anderer Wert wird abgelehnt (400 HTTP). Die Wartezeit ist auf die Abwicklungszeit plus 90 s begrenzt (standardmäßig 210 s, mindestens 90 s), und der Aufruf wartet `timeout` s auf das Backend: standardmäßig 240 plus die Abwicklungszeit (360 s), genau wie bei `chloros-mcp` und `test_array_sync`. `ptp_lock` verfügt über `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (wie lange jede Kamera bereits stabil ausgerichtet ist), `found_settled` (die Kameras, die zum Zeitpunkt der Erfassung bereits für die Stabilisierungszeit ausgerichtet waren) sowie pro Kamera `status`, `offsets_ns`, `slave_s` (wie lange sie bereits synchronisiert ist; „None“: nicht synchronisiert) sowie `remaining_s` („None“, wenn das PTP keiner Kamera aktiviert ist). Das Protokoll des Backends enthält alle 5 Sekunden eine Zeile mit dem Status jeder Kamera, der Dauer der Synchronisation und der verbleibenden Zeit. Jede Kamera wird mit deaktiviertem PTP eingeschaltet; von da an synchronisieren sich die Uhren in etwa 10 s. Danach wird jede Kamera wieder in den Zustand versetzt, in dem der Test sie vorgefunden hat – ihr PTP, die automatische Belichtung und die Verstärkung, die der Test für die Dauer des Durchlaufs auf „Aus“ gesetzt hat (`ExposureAuto` / `GainAuto` sowie ein manueller Lauf `ExposureTime` / `Gain`), und die übrigen, sofern nicht `keep_settings=True` – also eine Kamera, bei der PTP aktiviert bleibt (`PtpEnable=True`, oder ein Durchlauf mit `keep_settings=True`) bleibt synchronisiert, und sobald dieses Backend festgestellt hat, dass sie für die Stabilisierungszeit synchronisiert war, wartet ein späterer Test nicht mehr auf sie. Wenn sich die Uhren nicht stabilisieren, ist `success` vor jedem Trigger falsch, und `error` gibt vor, was zu tun ist: Eine Kamera, die nie fixiert wurde (`not_slave`), bedeutet, dass der Grandmaster sie nicht erreicht (`time_sync_status()`); eine, die immer wieder ausfiel, wird mit der Dauer ihrer Fixierung benannt (`remaining_s`); Uhren, die noch konvergieren (`far`): Überprüfe `time_sync_status()`, und um sicherzustellen, dass sie vor dem Test synchronisiert sind, schalte PTP ein und lasse es (`PtpEnable=True` an jeder Kamera oder `keep_settings=True`; der Test stellt die gefundenen Einstellungen wieder her, sodass es eingeschaltet bleibt), dann führen Sie ihn aus. `lattice multi-test`. |
| `detect_array_master(line=None)` | Die Kameras an der Synchronisationsleitung und ein Standard-Master: `master_serial`, `slave_serials`, die `line` wurden geprüft. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Was für eine Vielzahl dieser Kameramodelle (Produktnamen, `"LATT-M3C-L87-FOCN"`) im Netzwerk dieses Computers auftauchen würde: `tier`, `achievable_fps_max`, das Kabelbudget (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` und `fix_suggestions`. Standardmäßig werden die Modelle der erkannten Kameras angezeigt; ist keine Kamera geöffnet. Informationen zu den Kameras, die Sie besitzen, sowie zu den Einstellungen, die eine Verbindung verwenden würde, finden Sie unter [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Erstellen Sie ein Ausrichtungsprofil aus Live-Bildern und speichern Sie es. `method`: `orb`, `akaze`, `phase`, `checkerboard` (zusammen mit `checkerboard="9x6"`, die inneren Ecken) oder `manual` (zusammen mit `control_points={serial: [[x, y], ...]}`, auch die der Referenzkamera). `model`: `translation`, `rigid`, `affine` oder `homography`. Die detaillierteren Optionen sind die Flags der CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` ist `{serial: {option: value}}`. `rms_threshold_px` weigert sich, eine schlechtere Lösung zu speichern. Gibt `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (pro Kamera `reprojection_error`, `num_matches`) sowie `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Ein ausgerichtetes Multiband-Bild über das Profil hinweg: `files` (das Multiband-TIFF-Bild mit seiner Größe `bands`, zuzüglich eines pro Band mit `per_band` und eines pro Kamera mit `per_camera`) sowie `vignette_corrected`. `bands` wählt die Bänder aus und `order` ordnet sie nach Namen (eine Liste oder eine durch Kommas getrennte Zeichenfolge). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Stream-ausgerichtete Multiband-Bilder (gleiche Warp-Optionen wie bei `capture_aligned_multiband()`), für `count` oder bis zum Abbruch. Gibt ein `CliJob` zurück; dessen abschließende Statistik umfasst `frames_written` und alle `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Eine als `align-info` zusammengefasste Profildatei gibt Folgendes aus: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (pro Kamera `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (pro Kamera `band_names`, `wavelengths_nm`, `enabled`); `profile` ist die gesamte Datei. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Ändert die Profildatei an Ort und Stelle: `order` sind Bandnamen (jeder verschiebt die Ebene seiner Kamera in dieser Reihenfolge nach oben), `enable` / `disable` sind Seriennummern. Gibt das neue `layer_order` zurück. `lattice align-reorder`. |
| `index_catalog()` | Die Index-Voreinstellungen (`presets`: jeweils `name`, `channels`, `formula`) sowie die Farbe `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Ein Index über ein ausgerichtetes Mehrband-TIFF: ein `preset` (`NDVI`, `NDRE`, ...) oder ein `formula` über die Symbole `channel_map` ordnet Bänder zu (`{"NIR": "NIR_850", "Red": 0}`, nach Name oder Index). Ein voreingestelltesSymbol `channel_map` lässt das eine Band der Datei weg, dessen Name damit beginnt (`red` -> `Red_660`, `nir` → `NIR_850_RGN`), benannt nach der Datei selbst (ihrer Bandliste oder dem `Chloros:Band{R,G,B}Name` einer Chloros-Aufnahme bzw. eines Chloros-Produkts), sodass NDVI bei einer Chloros-Reflexionsaufnahme kein `channel_map` benötigt; zwei Bänder, die einem Symbol entsprechen, werden abgelehnt, nicht erraten. `capture_level` gibt an, was die Bänder enthalten: `reflectance` oder `radiance` machen die Werte quantitativ; ein unkalibrierter Stapel (`align-apply`) liefert qualitative Werte; `"unknown"` liest diese aus der Datei aus, wobei `Chloros:ProcessingLevel` einer Chloros-Datei enthalten ist. Gibt `stats` zurück (Form, den 1–99-Perzentil-Bereich, den LUT-Bereich, den verwendeten `capture_level`) sowie den neben der Eingabe notierten `files` (`<input stem>_<preset>.tif`, `..._LUT.png`) oder an `output`, ein Pfad: Bei einer Bilddateiendung werden die Dateien anhand ihres Stamms benannt (`output="D:/out/ndvi_FRGN.png"` schreibt `ndvi_FRGN.tif` und `ndvi_FRGN_LUT.png`), ohne diese ist es ein Präfix (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); lies die Pfade ab `files`. Sie enthalten die Kameraaufzeichnung der Eingabe (Hersteller / Modell / Seriennummer, GPS, Objektiv, Aufnahmezeit in UTC, die Kalibrierungs-ID, jedoch niemals die Raw-DN-Kalibrierung), `Chloros:ProcessingLevel=index` sowie den Index als ihr einziges Band (`XMP-Camera:BandName`, das TIFF- und das LUT-PNG-Datei gleichermaßen). Nur Dateien; keine Kamera. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Aufnehmen, kalibrieren (`reflectance` oder `radiance`; `raw` überspringt diesen Schritt), ausrichten und einen Index in einem Schritt berechnen; `save_multiband=True` behält den kalibrierten Stapel bei, den `compute_index()` später übernehmen kann. Gibt `stats` (zusammen mit dem `serials`, dessen Bänder verwendet wurden) und das `files` zurück. Es öffnet `serials` (das erste löst den Auslöser aus), andernfalls die Kameras des Profils, deren `layer_order`-Eintrag aktiviert ist, wobei die Referenzkamera zuerst (wird auch bei deaktivierten Bändern geöffnet); eine Kamera, deren Bänder deaktiviert sind, wird niemals kalibriert. `channel_map` wird nur für ein Symbol benötigt, auf das der Stack nicht reagieren kann, wie in `compute_index()`: Das Symbol einer Voreinstellung übernimmt das eine aktivierte Band, dessen Name damit beginnt; dies wird geprüft, bevor eine Kamera geöffnet wird (wenn keine oder mehrere darauf antworten, wird dies abgelehnt, wobei sie namentlich genannt werden); ein benutzerdefiniertes `formula` ist für jedes Symbol erforderlich. `index.tif` und die LUT enthalten die Kamera des Stacks, die Aufnahmezeit (die Belichtungszeit der Referenzkamera, UTC) und das Objektiv, und beide benennen den Index als ihr einziges Band (`XMP-Camera:BandName`, z. B. `NDVI`: Die Farb-PNG-Datei gibt den Index an, nicht ihre Farbkanäle). `lattice index --live`. |

### `CliJob`

Das Handle, das die drei Streams zurückgeben. Der Job läuft im Backend, das die Kameras so lange verwaltet, bis er gestoppt wird.

| Element | Beschreibung |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (gespeicherte / geschrieben, verworfen, `fps`, `elapsed`, Schreibfehler, `stop_reason`; bei einem fortlaufenden Job außerdem `frames_grabbed` und `frames_queued` – jeder erfasste Frame wird geschrieben, in die Warteschlange gestellt, verworfen oder scheitert – sowie `drop_reason`, `writers` und `write_ms`, die durchschnittliche Zeit, die das Schreiben eines Frames in Anspruch nahm) sowie `output_dir`. `state` ist `running`, `ended` (er hat sich selbst gestoppt und wurde noch nicht gestoppt), `stopping` (ein Stopp schreibt gerade seine Warteschlange aus: Live-Statistiken) sowie `stopped_by`) oder `finished` (seine Abschlussstatistiken, die des gestoppten Prozesses `error` und `stopped_by`). Der Datensatz eines angehaltenen Jobs wird für jeden Client 10 Minuten lang aufbewahrt (maximal die letzten 32): Ein anderes Skript oder `chloros-mcp` liest denselben Status und dieselben Abschlussstatistiken aus. Ein Job, der mit `chloros-cli` gestartet wurde (`lattice continuous`, `multi-stream`, `align-stream`) wird vom Backend gestoppt, wenn die CLI 30 Sekunden lang keine Meldung mehr sendet – entweder weil sie beendet wurde oder ihr Rechner im Ruhezustand ist –, wobei `stop_reason`, `client_gone` und `stopped_by` das Backend informieren; das Auslesen seiner Statistiken von hier hält es nicht am Laufen. Ein vom SDK gestarteter Job läuft, bis er gestoppt wird. Jeder Start sendet sein `timeout` als Frist des Backends (`client_deadline_s`): Eine Kamera, die danach geöffnet wird (während eine andere noch hochfährt), wird zurückgesetzt und geschlossen, und es wird kein Job gestartet; der Aufruf löst `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`) aus. Eine Kamera, die von einem Job belegt ist, wird bei jedem weiteren Öffnungsversuch (`connect_camera()`, einem weiteren Start, einer vorübergehenden Aufnahme) mit dem HTTP-Code 409 (`CAMERA_HELD`) abgelehnt, wobei der Auftrag und derjenige, der ihn gestartet hat, genannt werden. |
| `stop(timeout=30.0)` | Beenden und die zugehörigen Kameras schließen: `final_stats`, `output_dir` und `error`, wenn der Stopp nicht ordnungsgemäß verlief. Ein zweiter Aufruf liefert dieselbe Antwort, ebenso wie ein Beendigungsbefehl von einem anderen Client: Ein Befehl, der gesendet wird, während der erste noch seine Warteschlange ausschreibt, wartet darauf (bis zu 25 s; danach löst er einen Fehler aus, und die abschließenden Statistiken folgen in `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Warten, bis der Prozess von selbst endet (sein `count` wurde erreicht, das Laufwerk ist voll, ein Fehler liegt vor), dann `stop()` ausführen, um die endgültigen Zählwerte zu erfassen. `timeout` löst `TimeoutError` aus, während der Job noch läuft; ein Job ohne `count` endet erst, wenn er gestoppt wird. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Seine ID und Art (`continuous`, `multi-stream`, `align-stream`), wohin es schreibt, die Antwort der Startroute und ob `stop()` ausgeführt wurde. |

Verwenden Sie es als Kontextmanager, um es beim Verlassen des Gültigkeitsbereichs zu beenden.

---

## Projektordner & Rezepte

**Der Projektordner** ist der Ordner, in dem neue Projekte erstellt werden und der von der Chloros-App, der CLI und dem SDK (Modul `chloros_sdk.system_utils`; `chloros-cli project-folder`) gemeinsam genutzt wird:

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Ein konfigurierter Ordner, der nicht mehr existiert, wird – ebenso wie bei der App – auf den Standardordner (`Documents/MAPIR/Chloros Projects`) zurückgesetzt. Ein relativer Pfad, der für `set_project_folder()` angegeben wird, wird aus Ihrem Arbeitsverzeichnis übernommen, wenn sich das Backend auf diesem Rechner befindet.

**Ein Rezept** ist die YAML-/JSON-Aktionsliste, die `chloros-cli project run PATH RECIPE` für ein gespeichertes Projekt ausführt. `run_project_recipe()` führt es mit dem CLI-eigeneneigenen Runner (der mit dem SDK installiert wird), sodass sich ein Rezept in beiden Fällen gleich verhält:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** ist eine `.yaml` / `.yml` / `.json`-Datei oder das Rezept selbst: ein Dict mit `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) sowie optional `overrides` und `stop_on_error` (Standardwert „true“: der Lauf wird bei der ersten fehlgeschlagenen Aktion beendet). Die Aktionen entsprechen der [Recipe DSL](cli-reference.md#recipe-dsl) aus der CLI-Referenz; ein relativer `output`-Pfad wird relativ zum Arbeitsverzeichnis Ihres Skripts aufgelöst.
- **Gibt Folgendes zurück** `ok` (alle Aktionen waren erfolgreich), `actions` (Anzahl), `log` (die Ausgabe des Runners; `echo=True` gibt sie während der Ausführung ebenfalls aus) und `validation`. Eine fehlende oder nicht parsebare Rezeptdatei löst `ChlorosConnectError` aus.
- **Wird vor der Ausführung überprüft.**`validation` ist `{"ok", "errors", "warnings", "steps"}`: Jede Aktion muss genau ein bekanntes Verb als Schlüssel haben (`{"capture": {"name": ...}}`, nicht `{"action": "capture"}` oder `{"type": "capture"}`), das eine im Projekt vorhandene Kamera, ein Array oder einen Sensor des richtigen Typs benennt. Ein Rezept mit einem Fehler führt**keine** Aktion aus (`ok` false, die Fehler in `log`), unabhängig davon, was `stop_on_error` angibt. Ungenutzte Schlüssel, sowie jedes Array oder jeder Lichtsensor, den das Rezept steuert, der derzeit nicht angeschlossen ist, sind `warnings`. Ein Rezept stellt keine Verbindungen zu Geräten her: Stellen Sie die Verbindung zum ersten Gerät des Projekts her (`ChlorosProject.connect_all()`, `chloros-cli project connect`, oder das MCP-Tool `connect_project_devices`). `dry_run=True` führt nichts aus und gibt `{"recipe": ..., "validation": ...}` zurück.
- In einem Prozess wird jeweils nur ein Rezept ausgeführt. Ein YAML-Rezept benötigt PyYAML.

---

## Projekt-Automatisierung — `ChlorosProject`

Ein gespeichertes Chloros-Projekt ist ein Ordner, der `cameras.json` + `sensors.json` + `project.json` enthält. `open_project(path, relink=None)` lädt das Manifest (`relink`: Der Ordner, in dem sich die fehlenden Quelldateien eines verschobenen Projekts nun befinden; wird verwendet, wenn das Backend es lädt – `process()` – siehe **Ein verschobenes Projekt** unter `ChlorosLocal`), und `connect_all` schaltet jede gespeicherte Kamera, jedes Array und jeden Lichtsensor mit den gespeicherten Einstellungen online – derselbe Hardwarezustand, den die GUI erzeugen würde. Lichtsensoren verbinden sich über den [DAQ-Sensorpool](#daq-sensor-sessions), daher benötigen sie ein laufendes Backend (siehe [`SensorHandle`](#sensorhandle)).

### Minimalbeispiel

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Oder als Kontextmanager:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-Methoden

| Methode | Beschreibung |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Alle gespeicherten Kameras, Arrays und Lichtsensoren erkennen und verbinden. Gibt einen Verbindungsbericht pro Klasse zurück (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, jeweils mit `connected`, `missing` und `errors`; `arrays` enthält zudem `alignment`). Wenn ein Backend auf `127.0.0.1:5000` lauscht, werden die gespeicherten Kameras und Arrays über dieses Backend gefunden und verbunden. Ist kein Backend vorhanden, findet und steuert dieser Prozess sie selbst über `lattice_sdk`, das die Arena-SDK-Laufzeitumgebung und OpenCV (`cv2`) benötigt – das SDK-Wheel installiert keines von beiden; wenn dies nicht möglich ist, wird jede gespeicherte Kamera und jedes Array unter `errors` mit dem Grund aufgeführt, und `missing` bleibt leer. Ein Backend, das läuft, aber das SDK ablehnt (abgemeldet: HTTP-401; ein Plan unterhalb von Copper: 403), gilt nicht als „kein Backend“: Jede gespeicherte Kamera, jedes Array und jeder Lichtsensor wird unter `errors` mit dem Grund des Backends aufgeführt, und in diesem Prozess wird nichts geöffnet. Bei jedem Aufruf wird das Backend erneut überprüft, sodass „`connect_all()`“ nach der Anmeldung oder nach dem Start von Chloros eine Verbindung zum selben Projekt-Objekt herstellt. Ein gespeichertes Array, das bereits im Backend mit denselben Kameras (denjenigen der GUI oder von `chloros-cli project connect`) vorhanden ist, wird unverändert angehängt, ohne dass seine gespeicherten Einstellungen erneut angewendet werden, und `disconnect_all()` lässt es weiterlaufen. Lichtsensoren laufen immer über das Backend (siehe [`SensorHandle`](#sensorhandle)). Es wird niemals ein Backend gestartet. |
| `disconnect_all()` | Alles abbauen. Zunächst wird jedes Gerät freigegeben; wenn das Backend die Freigabe einer Kamera, eines Arrays oder eines Lichtsensors nicht bestätigt hat (das Abbauen ist abgelaufen oder wurde abgelehnt), wird jeweils ein `ChlorosConnectError` benannt. Ein bereits nicht mehr vorhandenes Gerät oder bei dem kein Backend läuft, bleibt stumm. Ein Array oder Lichtsensor, den ein anderer Client geöffnet hatte, bleibt aktiv. Der Beendigungsbefehl des Projekts `with` wird nie ausgelöst. |
| `capture_all(output_dir=".")` | Ein Bild von jeder angeschlossenen Kamera + jedem Array, und ein Spektrum (`SensorHandle.read()`) von jedem verbundenen Lichtsensor, nacheinander erfasst (zuerst Kameras, dann Arrays, dann Sensoren), nicht gleichzeitig. |
| `stream(camera, overlays=False, fps=10.0)` | Generator, der BGR-`numpy`-Frames von einer benannten Kamera (oder einem Array) liefert. `overlays=False` liest Frames mit dem Handle`grab()` auf der Ebene `debayered`: über das Backend, wenn das Projekt über eines verbunden ist, andernfalls über eine direkte `lattice_sdk`-Erfassungsschleife (Arrays liefern `{serial: frame}`-Dicts). `overlays=True` leitet über `ChlorosLocal.camera_stream()` → den MJPEG-Feed des Backends (`/api/camera/<serial>/stream-annotated`), wobei der gespeicherte Block der Kamera als Abfrageparameter weitergeleitet wird. Erfordert den Backend-Modus, OpenCV (`cv2`) und eine **eigenständige Kamera**: Eine Kamera im Direktmodus löst `RuntimeError` aus (das Backend kann keine Kamera erfassen, die diesem Prozess gehört), und ein Array löst `NotImplementedError` aus (überlagert Composite pro Kamera – streamt ein Element anhand seines Namens). Einmalige Entsprechung: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Führt eine Ausrichtung für jedes derzeit verbundene Array durch. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Die Kalibrierungs-/Index-Pipeline auf die Bilder des Projekts anwenden (umschließt `ChlorosLocal.process`; diese vier sind die **einzigen** akzeptierten Schlüsselwörter – `indices=` usw. löst `TypeError` aus; Indizes über `ChlorosLocal.configure()` festlegen). Erstellt verzögert ein `ChlorosLocal()`, das ein Backend automatisch startet. |

Attribute:
- `proj.cameras` — `Dict[str, CameraHandle]`, indiziert nach Name UND Seriennummer.
- `proj.arrays` — `Dict[str, ArrayHandle]`, indiziert nach Name UND array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, indiziert nach Name UND slot_id.
- `proj.config` — `project.json["config"]`-Wörterbuch.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Verarbeitungsstufen.** `capture()`, `grab()` und `frame_stream()` verwenden alle dasselbe `processing`
Token, und die Kette ist kumulativ – jede Ebene führt alle darüber liegenden aus:

| Ebene | Ausgabe | Anmerkungen |
| --- | --- | --- |
| `raw` | 1-Kanal-Bayer, sensornativ | Kein Demosaicing. Überlagerungen sind auf dieser Ebene nicht verfügbar. |
| `debayered` | 3-Kanal-Farbe (**Standard** für `CameraHandle` und alle `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` und `stream()` sind standardmäßig auf `raw` eingestellt): BGR im Speicher (`grab()` / `frame_stream()`), von `capture()` als RGB gespeichert | Im Backend-Modus als Aufnahme: der Dunkeloffset (DSNU) des Kalibrierungspakets vom Rohdatenwert abgezogen, anschließend OpenCVs kantensensitives Demosaicing – ohne Linearitäts-LUT, Flat-Field-Korrektur oder Unmix. Die einzige Stufe, die ohne Backend-Modus funktioniert, ist das einfache bilineare Demosaicing der Kamerabibliothek (kein Dunkeloffset). Die gespeicherte TIFF-Datei ist photometrisches RGB; daher die Kanäle eines Frames im Arbeitsspeicher umkehren (`frame[..., ::-1]`), um es mit einer Datei zu vergleichen. |
| `radiance` | float32, W/m²/sr/nm, **eine Ebene pro Band** | Vollständige radiometrische Kette: Demosaicing + 3×3-Entmischung (Multispektral) + DSNU + Flat-Field-Korrektur + NIST-Skalierung, wobei Belichtung × Verstärkung herausgerechnet wurden, sodass die Werte absolut sind. |
| `reflectance` | uint16, 32768 = 1,0 (oder uint8, 255 = 1,0 – siehe `bit_depth`), **eine Ebene pro Band** | Strahlungsdichte geteilt durch die nach unten gerichtete Bestrahlungsstärke (ρ = π·L/E). Erfordert einen DLS/DAQ-Messwert – siehe Hinweis unten. |
| `display` | 8-Bit, sRGB-ähnlich | GUI-äquivalente Darstellung: CCM + Weißabgleich + Gamma über das aktive Farbprofil der Kamera. |

Im direkten (Backend-freien) Modus ist nur `debayered` verfügbar: `grab()` auf einer anderen Ebene hebt
`NotImplementedError`, `capture()` ignoriert `processing`, und `frame_stream()` löst
`RuntimeError` auf jeder Ebene aus. `reflectance` benötigt einen verwertbaren Downwelling-Messwert – der Frame-Endpunkt zieht
das gepoolte DAQ automatisch in den DLS-Slot der Kamera, doch ohne gebundenes DAQ lehnt die Kette den
Reflexionsausgang ab und vermerkt die Herabstufung ehrlich in den zurückgegebenen Metadaten, anstatt stillschweigend
ein minderwertiges Produkt zurückzugeben.

> **Reflexions-DN-Skala – nicht fest einprogrammieren.** Die LATTICE-Reflexion verwendet `32768` = ρ 1,0 und vermerkt
> XMP `Chloros:PixelScale=32768`; die Survey3-Reflexion verwendet `65535` = ρ 1,0 und enthält keine
> `Chloros:*`-Tags. Lies das Tag aus und dividiere durch diesen Wert. Er ist im uint16-Bereich definiert, bleibt also
> `32768` für jedes Format, das eine Neuskalierung vornimmt (16-Bit-TIFF, 8-Bit-TIFF, 8-Bit-PNG/JPG, 32-Bit-Prozent)
> — normalisiere den gespeicherten Datentyp zunächst zurück auf uint16 (×257 von 8-Bit, ×65535 von Float). Die einzige
> Ausnahme ist eine Datei aus einem Build vor 1.3.0: Ein 8-Bit-TIFF wurde damals *beschnitten* und nicht neu skaliert, sodass
> keine Skalierung darauf zutrifft und sie kein `PixelScale`- oder MicaSense-Tupel enthält. Behandeln Sie ein fehlendes Tag in einer
> LATTICE-Reflexionsdatei als „keine gültige Skalierung“, nicht als Standardwert.

**Kalibrierungstags: nur bei `raw`.** Die `raw`-Datei einer Aufnahme enthält das Kalibrierungspaket der Kamera: die `CalibrationJSON`-Nutzlast und die MicaSense-kompatiblen Tags (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), die die Rohwerte des Sensors beschreiben. Jede Datei der anderen Ebenen (`debayered`, `display`, `radiance`, `reflectance`, der Render `index/`) sowie jedes Einzelbild aus der kontinuierlichen Aufzeichnung bzw. dem Multi-Kamera-Stream enthält die Identifikation der Kamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, …) sowie die aufnahmeeigenen Daten des Einzelbilds, `XMP-mapir:FrameJSON` (Belichtung, Verstärkung, Temperatur, Pixelformat, Aufnahmezeit, DAQ-Übereinstimmung), da jedes `process()`-Produkt diese Kennung trägt. Die Verarbeitung ermittelt die Kalibrierung einer solchen Datei anhand ihrer Seriennummer und des `CalBundleSha256`-Eintrags im Kalibrierungs-Cache vChloros und lädt das Paket einmalig herunter, falls es auf diesem Computer noch nicht vorhanden ist.

#### Dateien aus mehreren Kameras

Eine Datei, die aus den Bändern mehrerer Kameras erstellt wurde – ein Multiband-Stack (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), ein aus mehreren Kameras erstellter Live-Index (`index.tif`, `index_LUT.png`) oder das Bild „`composite/`“ einer Array-Aufnahme – nennt jede Kamera, die darin enthalten ist:

- **Pro-Kamera-Listen, in Bandreihenfolge** (die Zusammensetzung: Hintergrundebene, dann Vordergrund; eine Indexebene, die Kameras, aus denen ihre Bänder stammen): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (das Kalibrierungspaket, das die Bänder jeder Kamera durchlaufen haben; andernfalls – ein Stapel aus `capture_aligned_multiband()` / `start_aligned_stream()`, dessen Bänder keines durchlaufen haben – das von der Kamera Chloros zwischengespeicherte Paket, dasjenige, nach dem ihre Aufnahmedateien benannt sind), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` – parallele Listen, wobei Element *n* jeder Liste dieselbe Kamera beschreibt; `-`, wenn eine Kamera keinen Wert hat (kein Kalibrierungspaket, ein unbekanntes Objektiv). Jede Kamera ist **ein** Element: Wenn ihr Paket mehrere Werte für ein Feld enthält – beispielsweise die Halogenlampenzertifikate eines Pakets — werden mit „`;`“ (`F-1479-1;F-1480-1;F-1481-1`) verbunden, niemals mit einem Komma; daher enthält eine Liste unabhängig von der Darstellungsweise einen Eintrag pro Kamera.
- **EXIF `SerialNumber`** enthält dieselben Seriennummern, durch Kommas getrennt (`214000535,214000551`).
- Es enthält **keine** Einzelkamera-Identität (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): Diese benennen eine Kamera und deren Paket, und die Verarbeitung würde daraus die Kalibrierung der Datei ermitteln. Ein aus den Bändern mehrerer Kameras berechneter Index (`index.tif`, `index_LUT.png`) enthält EXIF-Daten: `Model`, `LATTICE` (das Rig), nicht das `LATT-…`-Modell einer einzelnen Kamera.

Eine Datei mit den Bändern **einer** Kamera – ein TIFF pro Band, ein `capture_aligned_multiband(per_camera=True)`-Frame, ein Stack oder ein Live-Index einer einzelnen Kamera – enthält die eigenen Identitäts-Tags dieser Kamera, genau wie jede andere Aufnahmedatei. Der Stapel von `capture_aligned_multiband()`, die bandweisen und kameraspezifischen Dateien sowie jeder Stapel von `start_aligned_stream()` und seine bandweisen Dateien enthalten die Aufnahmezeit ihrer Gruppe (Belichtungszeit der Referenzkamera, UTC), und ein `per_camera`-Frame einer Mono-Kamera besteht aus einer Ebene, bei `bit_depth` genau wie der Stapel.**Kameraausrichtung (Spiegeln / Drehen).** Einzelbilder aus `grab()` / `frame_stream()` liegen immer in der
nativen Ausrichtung der Kamera vor, auf jeder Ebene. Gespeicherte Aufnahmen folgen der Ausrichtungs-
Einstellung der Kamera (Spiegeln H / Spiegeln V / Drehung in der Benutzeroberfläche):

- **`raw`- und `debayered`-Dateien entsprechen genau dem, was der Sensor erfasst hat**, unabhängig von der Einstellung – ein gedrehtes oder gespiegeltes Bayer-
  Mosaik wäre ein anderes Farbfiltermuster, und die Kalibrierungskarten beziehen sich
  auf dem sensoreigenen Raster.
- **`display`, `radiance`, `reflectance` und Indexdateien sind ausgerichtet** – einmal gedreht, nach Abschluss der gesamten
  Kalibrierungsberechnungen. Ihre Clamp-Masken liegen auf demselben Raster.
- **Jede Aufnahmedatei speichert die Einstellung** im XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, im Uhrzeigersinn, nach
  den Spiegelungen) sowie `ChlorosOrientationApplied` – `false` auf Rohdaten und debayerte Daten, `true` auf die ausgerichteten
  Produkte. `ChlorosProcessing` listet die auf die Pixel dieser Datei angewendeten Operationen auf (`none`, wenn
  nativ). Dazu gehören alle Einzelbilder einer fortlaufenden Aufnahme (die kameraspezifische Einstellung,
  nicht angewendet) sowie eines Mehrkamera-Streams (die Identität: Seine Einzelbilder sind nativ, wie bei einer
  Array-Aufnahme).
- **Bei der Verarbeitung eines Projekts wird die aufgezeichnete Ausrichtung** auf die daraus erstellten Produkte angewendet
  (zuerst die Kalibrierung in Sensorkoordinaten, dann die Array-Ausrichtungsverzerrung, zuletzt die Ausrichtung),
  sodass sie mit den Produkten der Live-Aufnahme übereinstimmen – einschließlich der unkalibrierten, sensorabhängigen und vignettierungskorrigierten
  Fallback-Produkte; das debayerte Produkt bleibt nativ.
- **Der Chloros-Bildbetrachter zeigt eine Roh- oder debayerte Datei so an, wie sie bei der Aufzeichnung gedreht wurde** – dies dient nur der Anzeige,
  die Datei wird nicht verändert; eine Datei ohne Aufzeichnungsinformation wird so angezeigt, wie sie gespeichert wurde.
- **Aufnahmen ohne Aufzeichnungsinformation werden als native Daten verarbeitet.** Vor Version 1.3.0 drehte eine Einzelkamera-Aufnahme
  ihre RAW- und debayered-Dateien selbst; Chloros kann ein solches RAW nicht von einer nativen Datei unterscheiden und
  macht keine Vermutungen.
- **Array-Aufnahmen behalten standardmäßig jede Einzeldatei im nativen Format** und zeichnen die Identität auf;
  `ArraySession.capture(orient=True)` wandelt deren Produkte um (siehe die Steuerelemente für Array-Aufnahmen).

#### bandweise Ausgabe und gemischte Arrays

Auf den Ebenen `radiance` und `reflectance` gibt eine Kamera **eine Ebene pro Band dieser
Kamera** zurück – drei für eine M3C (ihre drei ungemischten Bänder, BGR-Kanalreihenfolge), eine für eine M3M (ihren
einzigen Schmalbandfilter). Die replizierten Ebenen der Mono-Kamera werden zusammengefasst, sodass ein **gemischtes
M3C/M3M-Array liefert pro Serie unterschiedliche Array-Formen**. Lesen Sie den Schlüssel `bands` aus den Frame-Metadaten
aus, um zu erfahren, um welche Ebene es sich jeweils handelt; leiten Sie dies nicht aus der Kanalanzahl ab:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Überprüfen Sie immer `hdr["processing_level"] == "reflectance"`, bevor Sie die Werte verwenden. Eine Herabstufung
(kein DAQ-Messwert, kein Kalibrierungspaket, unbekanntes Bandlayout) wird in `processing_level` und
`radiometric_refused` gemeldet und nicht als Fehler angezeigt – das zurückerhaltene Array ist weiterhin ein gültiger Frame, nur
nicht das von Ihnen angeforderte Produkt. Eine gespeicherte Erfassung, die herabgestuft wurde, enthält in ihrer XMP-Datei denselben Code
wie `Chloros:RadiometricRefused` (z. B. `no-band-layout`, `no-dls-snapshot`).

#### Ein Trigger pro Lesevorgang – `group_id`

**Array-Mitglieder werden aus der synchronisierten Bildgruppe des Arrays bereitgestellt, nicht aus dem jeweiligen
Grab-Puffer der einzelnen Kameras.** Die Kameras eines hardwaresynchronisierten Arrays belichten gemeinsam, aber N Lesevorgänge pro Kamera
erfolgen nicht gemeinsam – das Auslesen des zuletzt gepufferten Bildes jeder Kamera erstreckt sich über die Triggergrenzen, und ein einziges ausgelassenes Bild desynchronisiert das Array, ohne dass dies bemerkt wird.

Die Bereitstellung aus der Gruppe ist notwendig, reicht aber allein nicht aus: Jede Anfrage erstellt weiterhin
*die aktuellste* Gruppe zu ihrem eigenen Zeitpunkt ab und die Grab-Schleife kann dazwischen neue Frames veröffentlichen.
Bei Messungen an einem Live-Array mit 4 Kameras **erstreckten sich 12 von 20 parallelen Lesevorgängen über zwei Gruppen** – immer
genau benachbart. Daher wird der Lesevorgang **festgepinnt**:

1. Jedes Bild enthält `group_id`, den monotonen Veröffentlichungszähler des Arrays.
2. `grab()` liest den Master (oder, falls der Master nicht bedient werden kann, die nächste Kamera, die dazu in der Lage ist)
   und bindet die verbleibenden Kameras an die Gruppen-ID, bei der es gelandet ist (`?group_id=N` am HTTP-
   Endpunkt).
3. Das Backend behält aktuelle Gruppen **mindestens eine halbe Sekunde** lang bei, sodass ein Element, das bereits
   von späteren Veröffentlichungen überholt wurde, bei diesem Trigger weiterhin bedient wird – lang genug für den Roundtrip eines
   Reflexionswert-Export pro Kamera, selbst bei einem Vollsensor-Array mit 9 Kameras. Eine Zuordnung, die das Backend nicht
   mehr hält, wird mit `409` und `group_aged_out: true` abgelehnt, anstatt stillschweigend
   mit einem anderen Trigger beantwortet wird – `grab()` liest den Satz erneut ein.

Bei erneuter Messung mit Pinning: **20 von 20 Lesevorgängen lieferten einen Trigger.** `grab()` erledigt dies für Sie;
Sie benötigen `group_id=` nur, wenn Sie den HTTP-Endpunkt direkt ansteuern.

- Eine Kamera, die in der festgelegten Gruppe nicht bedient werden kann – weil sie darin fehlt oder die Anfrage ablehnt
  (ein `band`, über das sie nicht verfügt, ein Pegel, den sie nicht erzeugen kann) – wird **weggelassen, anstatt durch einen anderen Trigger ersetzt zu werden**
  durch einen anderen Trigger, und die anderen Kameras werden weiterhin zurückgegeben.
- Das von `grab()` zurückgegebene Wörterbuch ist ein `GrabResult` (eine einfache `dict`-Unterklasse;
  `from chloros_sdk import GrabResult`): `.errors` ordnet jede
  ausgelassene Seriennummer der Ausnahme zu, die beim Lesen ausgelöst wurde, und `.group_id` ist die gemeinsame Gruppe. Ein leeres
  `.errors` bedeutet, dass jede Kamera diese Gruppe gebildet hat.
- Eine eigenständige Kamera hat keine Gruppe, meldet `group_id: None` und **lehnt** einen `group_id=`-
  Pin ab, anstatt ihn zu ignorieren.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Der gleiche Zähler wird in **jedem kameraspezifischen MJPEG-Stream** eines Array-Mitglieds veröffentlicht – der
Vorschau (`/api/camera/stream/<serial>`), der kommentierten Vorschau (`/stream-annotated`) und dem
Reflexionsstrom (`/stream-reflectance`) – jeweils als `X-Chloros-Group-Id`, neben
`X-Chloros-Frame-Id` und `X-Chloros-Timestamp-Ns`. Die Streams sind fortlaufend und keine
Zeitpunktaufnahmen; daher werden sie durch das Sammeln von Teilen und das Gruppieren anhand dieses Headers abgeglichen – es gibt keinen
Pin, der gesetzt werden muss. Eine eigenständige Kamera hat keine Gruppe. Siehe
[MJPEG-Streams pro Kamera](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` wird niemals still.** Ein Tick, der aus einem Grund fehlschlägt, der sich von selbst beheben lässt,
wird übersprungen – ein Kamerastream liefert nichts dafür, ein Array-Stream liefert die Kameras, die
geantwortet haben (möglicherweise keine). Diese Gründe sind: noch kein Bild; der Grab-Thread der Kamera wurde für ein
oder zwei Sekunden angehalten (`404 not streaming`, worauf das Backend immer dann antwortet, wenn eine andere Kamera eine Verbindung herstellt
oder trennt oder eine Einstellungsänderung den Pool neu ausbalanciert); ein Backend-Neustart oder `5xx`; die Ausrichtung
ist noch nicht stabil (`409 … retry shortly`); und eine Reflektanz-Ablehnung, weil das DAQ noch keinen Messwert
hat (`no-dls-snapshot`, ein veralteter oder leerer Messwert). Eine Ablehnung beim nächsten Tick würde sich nur wiederholen – ein
unbekannter Fehler `band`, `bit_depth=8` bei der Strahlungsintensität, ein Kalibrierungspaket ohne Strahlungsintensitätsskala, eine Reflektanz-
Ablehnung mit **überhaupt keinem DAQ** (keines an die Kamera oder deren Array gebunden, keines mit dem
Backend verbunden: das `dls_source` der Ablehnung ist `"none"`), jedes andere `4xx` –
löst `chloros_sdk.FrameFetchError` aus (ein `RuntimeError`, der `.status` und `.body` enthält) auf
einmal aus. Dasselbe gilt für **60 Sekunden** übersprungene Ticks ohne dazwischenliegenden Frame, unabhängig davon, was das `fps` ist – lang
genug, um die Ausrichtung zu stabilisieren und einen Backend-Neustart abzuwarten, kurz genug, dass eine Kamera, die
wirklich ausgefallen ist, den Stream beendet. Dieser Abbruchfehler ist ebenfalls ein `FrameFetchError`, sowohl für eine einzelne Kamera als auch für
ein Array, wobei die Ausnahme des letzten Lesevorgangs unter `.errors` (und bei einer Kamera als `__cause__`). Bei
einem Array bedeutet „refused“, dass jede Kamera abgelehnt wurde; die Ausnahme `.errors` enthält den jeweiligen
Grund für jede Kamera.

**Ein Array `frame_stream()` liefert jeden Trigger einmal** (`ArrayHandle` und `ArraySession` gleichermaßen). Ein
Tick, dessen neuester Satz der Auslöser ist, den er bereits ausgegeben hat, wartet auf den nächsten Auslöser, anstatt
denselben Satz erneut auszugeben, sodass der Stream mit der langsameren von `fps` und der eigenen Bildrate des Arrays läuft, und aufeinanderfolgende Sätze teilen sich niemals ein `group_id`. Ein Array, das die Veröffentlichung einstellt, löst
`FrameFetchError` („kein neuer Trigger seit 60 s“) aus, anstatt seinen letzten Satz zu wiederholen. Ohne diese Funktion würde ein
Stream, der mehr anfordert, als das Array veröffentlicht – oder der direkt nach dem Verbindungsaufbau startet –,
Wiederholungen ausgeben: In einem gemessenen Durchlauf enthielten 20 Sätze 11 verschiedene Gruppen-IDs.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Exportieren eines einzelnen Bandes – `band`

`grab()` / `frame_stream()` verwenden `band="<name>"`, um nur eines der Bänder einer Kamera als
einkanaliges `(H, W, 1)`-Array — in derselben Form, wie sie eine Monokamera erzeugt; daher wird die Ein-Band-Ausgabe
identisch behandelt, unabhängig davon, ob sie von einer M3M stammt oder durch Aufteilung einer M3C gewonnen wurde. Das Rotband einer FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Zulässige Schreibweisen sind der vollständige Layout-Name (`Red_660`, `NIR_850`, `Green_550`) oder dessen kurze
Kopfbezeichnung (`red`, `nir`, `green`), wobei die Groß-/Kleinschreibung keine Rolle spielt. Ein unbekannter Name wird abgelehnt und die verfügbaren
Namen werden aufgelistet.

> **Die Auswahl erfolgt nach dem Namen, nicht nach der Kanalnummer – und zwar bewusst.** „Kanal 1“ ist `Green_550` in
> der BGR-Kanalreihenfolge des Arrays, aber *rot* in derBuchstabenreihenfolge (`RGN`), und die beiden
> stimmen nicht überein. Wählt man den falschen aus, erhält man einen vollkommen plausiblen Frame des falschen Bandes, den
> nichts nachgeschaltetes erkennen kann. Aus diesem Grund gibt es keinen numerischen Index.

#### Ausrichten der Streams — `align`

`align=True` transformiert das Bild in den gemeinsamen Referenzrahmen seines Arrays. Jede Kamera wird
durch die **gleiche** Profilauflösung und denselben Ausschnitt-ROI transformiert, sodass unabhängig voneinander geöffnete kameraspezifische Streams**gegenseitig registriert** sind – man muss sie nicht gemeinsam anfordern:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Anforderungen und Verhalten:

- Die Kamera muss sich in einem **hardware-synchronisierten** Array (M8-Synchronisationskabel) befinden, das über ein Ausrichtungsprofil verfügt
  — die Co-Registrierung setzt voraus, dass die Mitglieder gleichzeitig belichtet werden, was ein software- oder
  Free-Run-Array nicht gewährleistet. Andernfalls wird der Vorgang mit dem Fehlercode `409` abgelehnt.
- **Wird abgelehnt, niemals stillschweigend nicht ausgerichtet.** Der Aufzeichnungspfad speichert möglicherweise nicht ausgerichtete Daten mit einer Warnung,
  da eine Datei ihre Herkunfts-Tags beibehält und später neu registriert werden kann; ein Live-Stream hat keine
  solche zweite Chance, daher ist ein fehlendes Profil ein Fehler und kein Fallback.
- **Die Anforderung der Ausrichtung startet diese.** Die Anforderung von `align` bei einem Array ohne Profil löst die
  Kalibrierung aus und gibt `409` mit dem zurück, worauf gewartet wird (z. B. *„Warten darauf, dass sich die Belichtung
  auf 214000551 (1/3) einpendelt“*) – bei einem erneuten Versuch gelingt es in der Regel innerhalb weniger Dutzend Sekunden.
  Dies ist im Headless-Modus von Bedeutung: Die automatische Ausrichtung bei der Verbindungsherstellung gibt nach 30 s auf, und jeder Versuch
  innerhalb dieses Zeitfensters wird übersprungen, solange sich die Smart-AE noch einpendelt. Daher hatte ein Array, dessen Belichtung
  sich nur langsam einpendelte, bisher kein Profil für die gesamte Sitzung und es gab keine Möglichkeit,
  eines zu erhalten, ohne die kombinierte Desktop-Ansicht zu öffnen. `alignment_status` meldet außerdem
  `ae_align_holdouts` und nennt dabei die Kameras, die sich noch einpendeln, sowie deren Fortschritt.
- Weisen Sie jedem Stream eines Arrays denselben **`max_h`** zu, da ihre Ausschnitte sonst auf unterschiedlichen Pixelrastern
  landen.
- Die Ausrichtung wird *vor* der Bit-Tiefen-Kodierung angewendet, sodass die Verzerrungskorrektur Werte mit voller Präzision
  interpoliert.

#### `bit_depth` – Präzision gegen Bandbreite eintauschen

`grab()` und `frame_stream()` akzeptieren `bit_depth=16` (Standard) oder `bit_depth=8`:

| `bit_depth` | Reflexionsgrad-Kodierung | Kosten |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, Spielraum bis ρ 2,0 | Volle Genauigkeit. |
| `8` | uint8, erstreckt sich über `0..refl_max` (Standardwert ρ 1,0) | Die Hälfte der Bytes; ~0,004 ρ-Quantisierung bei der Standard-Obergrenze. Werte oberhalb der Obergrenze werden abgeschnitten – siehe unten. |

##### Die 8-Bit-Obergrenze — `refl_max`

`refl_max` legt den ρ-Bereich fest, den der 8-Bit-Bereich abdeckt, sodass `pixel_scale = 255 / refl_max` und ρ weiterhin
`pixel / pixel_scale` sind. `grab()` und `frame_stream()` verwenden es als Schlüsselwort sowohl für die Kamera als auch für
Array-Handles (`?refl_max=` am HTTP-Endpunkt). Erhöhen Sie diesen Wert, um Spielraum für spiegelnde
Lichter zu schaffen, und zwar in proportional gröberen Schritten:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | Schritt |
| --- | --- | --- |
| `1.0` (Standard) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Die automatische Belichtung kann einen 8-Bit-Clip nicht beheben, und der Versuch, dies damit zu erreichen, verschlechtert die Datenqualität.**
> ρ = π·L/E, wobei Belichtung und Verstärkung herausgerechnet sind; daher ist ρ eine Eigenschaft der Szene und nicht der
> Kameraeinstellung – gemessen an einem Live-Array führte eine Halbierung der Belichtung zu einer Verschiebung des Mittelwerts von ρ um 1,3 %. Eine Erhöhung
> Belichtung, um den Bereich „anzupassen“, treibt die Pixel stattdessen in die **Sensor**-Sättigung, was
> L unterschätzt und ρ **nach unten** verzerrt, ohne dass ein Fehler gemeldet wird (gemessen: max. ρ 1,203 → 0,985 bei
> doppelter Belichtung, Mittelwert −23 %). Wählen Sie den Bereich mit `refl_max`; den Sensor vor
> Sättigung zu bewahren, ist bereits die Aufgabe der automatischen Belichtung (AE), und das schützt eher die *Gültigkeit* von ρ als dessen Bereich.

**Lesen Sie `reflectance_clipped_fraction`, nicht das Flag.** `reflectance_clipped_at_1_0` löst bereits bei einem
einzigen Pixel aus – bei einer realen Szene meldete es `True`, während der Anteil `0.000001` betrug, also ein Pixel
von einer Million. Der Bruchteil gibt an, ob die Obergrenze angehoben werden soll.

**`bit_depth=8` bezieht sich ausschließlich auf die Reflektanz.** Eine Abfrage für `radiance` wird abgelehnt statt
ausgeführt: Die Reflexion hat einen festen Bereich von 0 bis 1, der eine 8-Bit-Kodierung übersteht, während die Strahlungsintensität keinen
festen Bereich hat; um sie in 8 Bit unterzubringen, ist daher eine pro Bild erfolgende Perzentil-Streckung erforderlich, deren Zuordnung
sich mit jedem Bild ändert – ein Bild, keine Messung.

#### Live-Reflexionsgrad-Stream (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` liefert 8-Bit-Reflexionswerte pro Band als
`multipart/x-mixed-replace` – ein M3C als 3-Kanal-JPEG, ein M3M als Graustufenbild. Abfrageparameter:
`fps` (Standardwert 10), `jpeg_q` (Standardwert 95), `max_h` (0 = nativ), `band` (ein benanntes Band, wie
oben), `align` (`1` zur gemeinsamen Registrierung – gleiche Regeln wie beim SDK-Parameter `align`), `refl_max` (die Obergrenze der 8-Bit-
Kodierung, wie oben). Jedes Teil enthält
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` und `X-Chloros-Reflectance-Clipped`.
**Teile verschiedener Kameras, die denselben Wert `X-Chloros-Group-Id` tragen, gehören zum selben Trigger** —
Auf diese Weise werden mehrere Streams pro Kamera eines Arrays wieder zusammengesetzt, und dies wird
serverseitig garantiert, da die Array-Elemente aus der synchronisierten Frame-Gruppe bereitgestellt werden (siehe
[Ein Trigger pro Lesevorgang](#one-trigger-per-read--group_id)). Es gibt `409` zurück, wenn die Reflexanz
zum Zeitpunkt der Verbindung nicht verfügbar ist, und beendet den Stream, wenn sie während der Ausführung nicht mehr verfügbar ist; ein ins Stocken geratener
Stream bedeutet also „keine Reflexanz mehr“ und nicht „noch in Ordnung, nur statisch“.

> **Das JPEG-Format ist verlustbehaftet – diese Pixel stellen eine Messung dar.** Komprimierungsfehler wirken sich direkt auf die
> Reflexionswerte aus und sind an Kanten am stärksten, wo sich Materialgrenzen befinden. Aus diesem Grund ist die Qualität
> standardmäßig auf 95 eingestellt. Wenn Sie die Zahlen unverändert benötigen, verwenden Sie
> `grab(processing="reflectance")` / `frame_stream(...)`, die dasselbe Ergebnis liefern, ohne dass ein
> Codec im Pfad enthalten ist (und `bit_depth=8`, falls die Leitungs-Kosten und nicht der Codec die Einschränkung darstellen).

> **EXIF-Daten werden in den Export übernommen.** `process()` kopiert den GPS-Block der Quellaufnahme
> **und deren ExifIFD** in jedes Produkt, sodass Exporte die Bezeichnungen `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` und `CameraSerialNumber` sowie die
> Georeferenzierung. Anhand von `FocalLength` ermittelt Pix4D den Ground-Sample-Abstand – ohne diese Angabe
> fällt die Rekonstruktion auf einen völlig falschen Maßstab zurück (in einem gemessenen Fall wurde aus einem 411 m großen Gelände
> ein 47,8 km großes). Die Kopie ist bewusst nicht `-all:all`: Die Struktur-Tags von IFD0 zerstören
> die LATTICE-Ausgabe, und `ExifImageWidth`/`Height` werden ausgeschlossen, da sie die
> Quellaufnahme und nicht das exportierte Raster beschreiben.

Unterflags der Erfassungsphase (gelten für die radiometrischen Ebenen – `radiance`, `reflectance`, `display`):

| Flag | Standardwert | Bedeutung |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + Flat-Field + 3x3-Unmix + radiometrische NIST-Skala. |
| `apply_white_balance` | `True` | WB-LUT. DLS-fähig, wenn ein DAQ mit der Kamera verbunden ist. |
| `apply_index` | `False` | Auswertung des Vegetationsindex. |
| `index_expression` | `None` | Formel überschreiben. Nichtleer → aktiviert den Index automatisch. |
| `annotated` | `False` | GUI-Dekorationen überlagern (Zebra/Raster/Peaking). Nicht verfügbar für `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Der Rückgabewert ist ein `chloros_sdk.CapturePaths`, vom Typ `CapturePathMap`, nicht vom Typ `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` ist `Dict[str, Union[str, List[str]]]`: ein einstufiges
> `processing` weist jeder Seriennummer einen Pfad zu, während `processing="all"` ihr die
> **geordnete Liste** aller für diese Kamera gespeicherten Produkte zuweist. `ArrayHandle.capture()` akzeptiert
> kein `levels=` und ignoriert ihm unbekannte Schlüsselwortargumente (`levels`, `daq`,
> `render_index`, `force_daq`), übergebe daher nur die hier gezeigten (`aligned=False`
> speichert native Dateien, wie bei `ArraySession.capture()`); verwenden Sie
> `ArraySession.capture()` für diese Steuerelemente. Ein live kombiniertes Composite, falls eines gestreamt wurde, wird unter dem zusätzlichen
> `"combined"`-Schlüssel statt unter einer Seriennummer ein. Code, der von `str` ausgeht, bricht in der
> Listenform ab, ohne dass ein Typprüfer Einwände erhebt – die Anmerkung besagte, dass `Dict[str, str]`
> eine Zeit lang nach der Veröffentlichung der Listenform angegeben, weshalb der Alias existiert. Normalisieren Sie
> die Form, wenn Sie die flache Form wünschen:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Was nicht geschrieben werden konnte: `.failed`.** Die von `capture()` zurückgegebene Zuordnung ist ein
> `chloros_sdk.CapturePaths` — eine einfache `dict`-Unterklasse, sodass alles oben Genannte
> unverändert funktioniert — die zudem `.failed` enthält: die Dateien, die das Backend nicht schreiben konnte,
> dieselben Einträge wie [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error` und `disk_full: True` bei einem vollen Laufwerk). Eine teilweise
> Erfassung gibt weiterhin jeden Pfad zurück, der geschrieben wurde; bei einem vollen Laufwerk ist die Zuordnung
> leer und `.failed` erklärt, warum. `.skipped` listet die Übersprünge pro Kamera auf und
> `.group_seq` die synchronisierte Gruppe, und `.alignment` / `.oriented` sind die
> des Backends, wie bei `CaptureResult`. `capture_smart()` gibt sie als
> `result["frames"].failed` und `result["failed"]` zurück.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Array-Ausrichtung

`ArrayHandle` stellt die gesamte Ausrichtungsfläche bereit. Profile sind standardmäßig nur für die jeweilige Sitzung gültig – rufen Sie `export_alignment()` explizit auf, um sie dauerhaft zu speichern.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Ausrichtung bei Verbindungsaufbau

`connect_all(align=...)` kann jedes Array beim Verbindungsaufbau automatisch ausrichten:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Wenn nicht anders angegeben, wird auf `project.json["config"]["auto_align_on_connect"]` zurückgegriffen.

### `SensorHandle`

Ein gespeicherter Lichtsensor (DAQ-U, DAQ-M oder DAQ-E), der von `connect_all()` im [DAQ-Sensorpools](#daq-sensor-sessions) – demselben Pool, den auch `connect_daq_sensor()` und `chloros-cli daq pool-connect` verwenden.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` wartet auf das **nächste** Spektrum des Sensors – eines, das zum Zeitpunkt des Aufrufs bereits gepuffert ist, zählt nicht — und gibt `(spectrum, is_saturated, integration_time, x, y, z)` zurück, oder `None`, falls innerhalb von 5 s kein Spektrum eintrifft. Ein angehaltener Datenstrom wird fortgesetzt. `spectrum` wird in W/m²/nm angegeben, wenn der Sensor kalibriert ist.
- `.sensor` ist der [`DAQSensorSession`](#daq-sensor-sessions) des Sensors; in allen anderen Fällen: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` und `.settings` sind der gespeicherte Eintrag aus `sensors.json`.

**Was `connect_all` wiederherstellt.** Jeder Sensor wird an seiner gespeicherten Adresse geöffnet – seinem COM-Port, seiner Bluetooth-Adresse oder seinem DAQ-E-Host – und, sofern das Projekt die ID des Sensors aufgezeichnet hat, muss es sich um diesen Sensor handeln: Ein anderes Gerät an dieser Adresse wird abgelehnt und nicht verbunden. Seine gespeicherte Integrationszeit, die Bildmittelung und die automatische Belichtung werden angewendet, ebenso wie seine Kappenkorrektur:

| Gespeicherte Belichtungsbegrenzung | Verbunden mit |
| --- | --- |
| Ein Belichtungsbegrenzungsprofil (z. B. `fov_30`) | Dieses Profil. |
| Keine ausgewählt | `sunshine_cosine`, die Standardverbindung, die wie angenommen (`cap_id_source=auto_default`) erfasst wurde; daher gibt die Verarbeitung eine Warnung aus. Die Sensoren werden ohne Abdeckung ausgeliefert: Wählen Sie die montierte Abdeckung aus. |
| `none` – in der Benutzeroberfläche als **Keine (unmontierter Sensor)** ausgewählt | Keine Kappenkorrektur. |

Ein DAQ-E-S akzeptiert kein Kappenprofil, unabhängig davon, was gespeichert wurde: Sein Diffusor ist Teil seiner Kalibrierung (siehe [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Ein von einem früheren Chloros gespeichertes Projekt hat einen ungeschützten Sensor als keine Auswahl erfasst, daher stellt es die Verbindung erneut mit `sunshine_cosine` her; wählen Sie in der GUI erneut **Keine (ungeschützter Sensor)** aus und speichern Sie das Projekt.

Ein Sensor, den ein anderer Client bereits im Pool geöffnet hat (CLI, ein anderes Skript), wird unverändert verbunden, ohne dass Einstellungen erneut angewendet werden, und `disconnect_all()` lässt ihn weiterlaufen; ein von `connect_all()` geöffneter Sensor wird freigegeben. Wie bei `connect_daq_sensor()` ist die Chloros-GUI **nicht** Teil des Pools: Schließen Sie einen Sensor in der GUI, bevor Sie das Projekt über ein Skript verbinden.**Voraussetzungen und der Bericht.** Lichtsensoren benötigen ein laufendes Chloros-Backend, das bei einem kostenpflichtigen Chloros+-Tarif (Copper oder höher) angemeldet ist, und `connect_all()` startet niemals eines. Wenn keines läuft, wird jeder gespeicherte Sensor unter `report["sensors"]["errors"]` mit einem entsprechenden Hinweis aufgelistet. Ein Sensor, den das Backend nicht öffnen kann, wird dort ebenfalls aufgeführt, zusammen mit der Begründung des Backends; `report["sensors"]["missing"]` bleibt für Lichtsensoren leer.

---

## Direkte Hardware (ohne Backend)

Wenn Sie bei LATTICE-Kameras (CI, headless Roboter, Embedded) keinerlei Abhängigkeit vom Backend wünschen, importieren Sie `lattice_sdk` direkt – es wird von `chloros_sdk` erneut exportiert. Hinweis zu `CAMERA_AVAILABLE`: `lattice_sdk` ist im SDK-Wheel enthalten, benötigt jedoch die Arena-SDK-Laufzeitumgebung und OpenCV (`cv2`), die das Wheel jedoch nicht installiert. Lichtsensoren werden über das Backend abgewickelt – [`connect_daq_sensor()`](#daq-sensor-sessions) oder das `proj.sensors` eines Projekts — mit einer Ausnahme: dem DAQ-M-Band-Magnituden-Helper [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), der zudem `bleak` benötigt.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Voreinstellungen und der Trigger

Drei der vier Voreinstellungen laufen im **Free-Run**-Modus: Die Kamera belichtet kontinuierlich und ein
`capture()` liefert das nächste Bild. `triggered` bildet die Ausnahme — es versetzt die
Kamera in Bereitschaft für eine Hardware-Flanke auf Leitung 2, sodass sie nichts aufnimmt, bis eine eintrifft.

| Voreinstellung | Auslöser | Verwenden, wenn |
| --- | --- | --- |
| `default` | Free-Run | Allgemeiner Gebrauch |
| `high_speed` | Free-Run | 8 Bit, Begrenzung auf 60 fps, kurze Belichtungszeit |
| `high_quality` | Free-Run | 12 Bit, keine fps-Begrenzung – die übliche Wahl für Standbilder |
| `triggered` | **aktiviert, Leitung 2** | Die Kamera ist an ein M8-Synchronisationskabel angeschlossen und wird durch ein anderes Gerät ausgelöst |

Wenn Sie `triggered` wählen (oder `trigger_mode="On"` selbst einstellen), ohne dass
etwas an Leitung 2 anliegt, läuft bei jedem `capture()` ein Timeout ab – was korrekt ist, da Sie
die Kamera angewiesen haben, zu warten. Das SDK erklärt dies, wenn es auftritt; siehe
[SC_ERR_TIMEOUT während der Aufnahme](#direct-hardware-backend-free).

> **Hinweis – „GVSP-Probe“-Meldungen / `SC_ERR_TIMEOUT -1011`-Meldungen beim Verbindungsaufbau sind keine Fehler.**
> Beim Verbindungsaufbau versucht das SDK,**Jumbo-Frames** (9000-Byte-GVSP-Pakete) für einen höheren Durchsatz auszuhandeln. Bei einer direkten Punkt-zu-Punkt-NIC-Verbindung (z. B. einer link-lokalen `169.254.x.x`-Adresse) kann das Netzwerk in der Regel keine Jumbo-Frames übertragen, sodass diese Abfrage zeitlich ausläuft und Zeilen wie die folgenden protokolliert werden:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Dies ist der **vorgesehene Fallback**: Das SDK wechselt automatisch zu Standardpaketen mit 1.500 Byte zurück, und die Kamera stellt weiterhin normal eine Verbindung her (die folgenden Zeilen „`[chunk-enable …]`“ sind Teil der normalen Verbindungssequenz). Die Aufzeichnung funktioniert weiterhin.
>
> Sie können diese Probe überspringen, aber **sie dient nicht nur dazu, die Protokollierung zu unterdrücken – sie deaktiviert Jumbo-Frames.** Die Kamera beantwortet „Don’t-Fragment“-Pings nur bis zu einer Größe von 1500 Byte, egal wie gut Ihr Netzwerk ist; daher kann der Ping-Test allein niemals Jumbo-Frames erkennen – nur diese Probe ist dazu in der Lage. Deaktivieren Sie sie, und die Kamera verwendet für immer Standardpakete mit 1500 Byte, und zwar in jedem Netzwerk:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Das lohnt sich nur in einem Netzwerk, von dem Sie *wissen*, dass es keine Jumbo-Frames übertragen kann; dort spart es etwa eine Sekunde Verbindungszeit pro Kamera. Da es sich um einen echten Kompromiss und nicht um eine rein kosmetische Änderung handelt, weist das SDK nun bei Verwendung darauf hin:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Lassen Sie die Einstellung unverändert, es sei denn, Sie haben einen Grund dafür.**Bleibt die Option aktiviert, wird jede Kamera einmal pro Backend-Sitzung gemessen und erneut, wenn sich ihre Verbindung ändert (anderer Adapter, andere Verbindungsgeschwindigkeit oder eine unterbrochene Verbindung, wie es beim Wechseln eines Switches der Fall ist); bei erneuter Verbindung wird das Ergebnis wiederverwendet.**Nach einer Änderung hinter einem Switch, der in Betrieb geblieben ist, starten Sie das Backend neu, um erneut zu messen.** Eine erste Verbindung kann bei dieser Überprüfung gelegentlich einige Minuten dauern; dies wird im Protokoll vermerkt, wenn dies der Fall ist.
>
> Wenn Sie den Jumbo-Durchsatz *wünschen*, aktivieren Sie Jumbo End-to-End (NIC-MTU 9000 + ein Switch, der diese durchlässt) oder fixieren Sie die Größe mit `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, wenn Sie wissen, dass die Verbindung dies unterstützt – bevorzugen Sie jedoch eine kommandoabhängige Einstellung mit `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` gegenüber einer dauerhaften Festlegung, da eine festgelegte Größe die Probe überspringt und die Anpassung an das vorgelagerte Netzwerk unterbindet. **Jedes** Gerät auf dem Pfad muss Jumbo-Pakete weiterleiten – einschließlich aller PoE-Splitter oder -Injektoren, was der übliche Grund dafür ist, dass eine ansonsten Jumbo-fähige Konfiguration diese nicht übertragen kann.

> **`SC_ERR_TIMEOUT -1011` während `capture()` / `grab*()` ist ein anderes Problem – dabei handelt es sich um einen echten Fehler.**
> Der obige Hinweis bezieht sich ausschließlich auf `-1011`, das von der**Connect-Time-Probe**protokolliert wurde. Der gleiche Fehler, der bei einer**Erfassung** auftritt, bedeutet, dass die Kamera zwar erfolgreich verbunden wurde, aber keine Bilder sendet:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Ein eindeutiges Anzeichen ist eine Kamera, deren *Steuer*kanal einwandfrei funktioniert – die Erkennung klappt, Einstellungen und `[chunk-enable …]`-Schreibvorgänge sind erfolgreich –, während *jedes* Bild eine Zeitüberschreitung verursacht.
>
> **Die übliche Ursache ist, dass die Kamera für einen Hardware-Trigger aktiviert ist.** Bei den Fehlern `trigger_mode="On"` und `trigger_source="Line2"` sendet die Kamera überhaupt nichts, bis eine elektrische Flanke am M8-Synchronisationskabel anliegt. Wenn kein Kabel diese Leitung ansteuert, wartet jeder Abruf ewig. Die Kamera ist nicht defekt und das Netzwerk funktioniert einwandfrei – sie tut genau das, was ihr befohlen wurde.
>
> `CameraSettings()` sowie die Modelle `default` / `high_speed` / `high_quality`-Voreinstellungen laufen im Freilauf, und ein Grab, das im aktivierten Zustand eine Zeitüberschreitung verursacht, liefert eine Erklärung, anstatt lediglich „`-1011`“ auszugeben. `PRESETS["triggered"]` aktiviert Line2 – dies ist beabsichtigt.
>
> So zwingen Sie eine beliebige Kamera in den Freilaufmodus:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Wenn es mit `trigger_mode="Off"` immer noch zu einem Timeout kommt, liefert die Kamera tatsächlich keine Daten – senden Sie uns das Protokoll und `ip link show`.

#### Farbprofile (RGB-Live-Vorschau) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` wählt das Farbprofil des Bildschirms für die **Live-Vorschau** bei RGB-Kameras aus (Multispektral-Kameras ignorieren diese Einstellung):

| Profil | Bedeutung |
| --- | --- |
| `raw` | Die radiometrische Kette vollständig umgehen. |
| `linear` | DSNU + Flat + WB, kein CCM, kein Gamma. |
| `natural` | Linear + gemessenes CCM + sRGB-Gamma, nur mit der „Cheap“-Nachbearbeitung (Chroma-Glättung + Entsättigung der Lichter) — die realistische Standardeinstellung. |
| `enhanced` | `natural` plus die vollständige „Hub-Parity“-Nachbearbeitung (Defringe, Vibrance, CLAHE-Lokalkontrast). Satteres Erscheinungsbild bei etwa **doppelten Bearbeitungskosten pro Bild**, daher eine niedrigere LIVE-Bildrate. |
| `custom_temp` | `natural`, jedoch mit auf `custom_cct_k` Kelvin festgelegter Weißabgleich (DLS wird ignoriert; backendseitig auf 2000–10000 K begrenzt). |

Das Profil ist ein **nur für die Live-Vorschau geltender** Regler für Geschwindigkeit und Bildqualität: Gespeicherte Aufnahmen erhalten unabhängig vom ausgewählten Profil immer die volle, reichhaltige Bildqualität, sodass die Wahl von `natural`, um Frame-Zeit zurückzugewinnen, die Qualität der auf der Festplatte gespeicherten Daten nicht mindert. Ein unbekanntes Profil erhöht den Wert von `ValueError`; wenn ein Chloros-Backend erreichbar ist, wird die Änderung auch an dieses gesendet (POST), sodass der nächste Vorschaubild dies widerspiegelt (Direkt-SDK-Benutzer ohne Backend erhalten die Einstellungsänderung dennoch).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Highlights (RGB-Anzeige) — `set_highlights`

`LatticeCamera.set_highlights(value)` reduziert die hellsten, weißesten Pixel der **Anzeige** bei RGB-Kameras herunter – die Korrektur für eine helle weiße Oberfläche, die als flacher weißer Fleck dargestellt wird. `value` entspricht `-100`..`0` (begrenzt); `0` ist deaktiviert, und die Darstellung entspricht dann genau dem Zustand ohne diese Einstellung. Stärkere Einstellungen senken das Weiß des Displays (auf etwa 60 % bei `-100`) und verbreitern die Tonwertkurve, sodass eine Fläche, die durch die automatische Helligkeitsanpassung der Vorschau in reines Weiß verschoben wurde, ihre Details zurückerhält.

Es handelt sich um eine Bildschirmsteuerung, aber im Gegensatz zu Sättigung und Kontrast (die nur die Live-Vorschau verändern) wirkt sie sich auch auf das aus, was Sie speichern: Die Live-Vorschau und die Aufnahmen auf **Bildschirmebene** richten sich danach, sodass diese Dateien ebenfalls dunkler ausfallen (sie enthalten den Wert `Chloros:DisplayHighlights`); Exporte von Strahlungs- und Reflexionswerten tun dies niemals, und es können keine Sensor-Daten wiederhergestellt werden, die übersteuert wurden – verringern Sie hierfür die Belichtung. Multispektral- und Monochrom-Kameras ignorieren diese Einstellung. Der Wert wird unter `settings.highlights` gespeichert und an ein erreichbares Chloros-Backend gesendet, genau wie bei `set_color_profile`; das CLI-Äquivalent ist `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Mono- (M3M-) Kameras und `Calibration`

Eine Mono-**M3M**-Kamera (`M3M-<lens>-F<wavelength>`) ist einbandig: eine Graustufenebene, kein Bayer-Mosaik, keine 3×3-Spektral-Crosstalk-Matrix. `Calibration` erkennt dies und setzt ein `is_mono`-Flag. Die Reflexion gilt weiterhin als radiometrische Karte pro Band (die Entmischung ist die Identitätsmatrix), aber Multiband-Berechnungen mit einer einzigen Kamera liefern sinnvolle Ergebnisse, anstatt Unsinn zurückzugeben:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Um einen Vegetationsindex mit Monochrom-Hardware zu erstellen, kombinieren Sie mehrere M3M-Kameras mit unterschiedlichen Wellenlängen zu einem ausgerichteten Multiband-Stack (siehe [Array-Ausrichtung](#array-alignment)) und berechnen Sie den Index über diesen Stack hinweg statt auf einer einzelnen Kamera.

#### DAQ-M-Bandintensitäten — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` verbindet sich über Bluetooth mit einem DAQ-M mithilfe seiner eigenen BLE-MAC-Adresse. Es handelt sich um die einzige Lichtsensor-Klasse, die direkt mit dem Sensor kommuniziert und nicht über das Backend; daher benötigt sie die gleichen Voraussetzungen wie die direkten Kameraklassen (`CAMERA_AVAILABLE`: die Arena-SDK-Laufzeitumgebung und OpenCV) sowie `bleak` für Bluetooth (`pip install "chloros-sdk[camera]"` oder `pip install bleak`; ohne diese Komponente löst `DLS` den Fehler `DLSError` aus). Ein an anderer Stelle verbundener DAQ-M (im Pool des Backends, in einem Projekt, in der GUI) sendet keine Adressen mehr; geben Sie es daher dort zuerst frei. **`mac_address` ist erforderlich**: Es gibt keinen Standardwert, da jede Einheit ihren eigenen hat, und `DLS()` löst ohne diesen Wert `DLSError` aus, bevor eine Verbindung versucht wird. Finden Sie die MAC-Adresse mit `discover_daq_sensors(transports=["ble"])` (siehe [Erkennung](#discovery--finding-an-address-to-connect-with)) oder `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integriert ein DAQ-M-Spektrum (den letzten Messwert, wenn `spectrum` gleich `None` ist; `DLSError`, falls noch kein Wert vorliegt) über die Reaktionen der drei Kanäle einer Kamera und gibt `{"red": …, "green": …, "nir": …}` zurück — die Schlüssel benennen die Kanäle in dieser Reihenfolge; bei den Kurven einer RGB-Kamera ist der dritte (`nir`) der blaue Kanal.

- Ohne `spectral_response` wird die Flottenkurve des Filtermodells verwendet (die, mit der das `filter_model` und das `DLS` gebaut wurden), die an einem Gerät gemessen wurde.
- `spectral_response` verwendet stattdessen die **eigenen** gemessenen Kurven einer Kamera: einem Dict `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, wie es im Kalibrierungspaket dieser Kamera gespeichert ist (`color_correction.spectral_response_smoothed` – RGB-Kameras; Multispektralpakete enthalten keine solche Kurve). Kann ein Dict nicht verwendet werden, greift das System auf die Flottenkurve zurück.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

Das `chloros-cli lattice dls --camera SERIAL` der CLI führt denselben Vorgang über das Backend durch, indem es die Kurve der Kamera in ihrem geladenen oder zwischengespeicherten Kalibrierungspaket ermittelt und angibt, welche Kurve verwendet wurde. Über HTTP ist das `POST /api/dls/cli-read` mit `mac` (erforderlich: ohne diese Angabe lautet die Antwort HTTP-Fehler 400, `code` `MAC_REQUIRED`) und `camera_serial` im Hauptteil; das `band_magnitudes_source` der Antwort lautet `camera <serial>`, wenn die kameraspezifische Kurve verwendet wurde, oder `model <filter>` für die Flottenkurve (eine Kamera ohne Kurve: kein RGB oder kein geladenes Paket oder zwischengespeichert).

---

## Ausnahmen

Fangen Sie die Basisklasse ab, um „alles, was bei Chloros schiefgelaufen ist“ zu behandeln:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` und `ChlorosConfigurationError` sind Attribute von `chloros_sdk` (und von `chloros_sdk.exceptions`), aber nicht in `__all__` enthalten, daher werden sie von `from chloros_sdk import *` nicht eingebunden: Importiere sie namentlich. Die `ChlorosError`-Familie wird von `ChlorosLocal` und den darauf aufbauenden Funktionen (`process_folder()`, `process_lattice_capture()`) abgeleitet; die Smart-Connect-Oberfläche leitet `ChlorosConnectError` ab.

Hierarchie:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## End-to-End-Beispiele

### 1. Einen Ordner mit einem benutzerdefinierten Fortschrittsbalken verarbeiten

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Live-LATTICE-Array → Reflexionsgrad + DAQ-Referenz

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektgesteuerte Erfassungskampagne

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Multi-Kamera-Frame-Stream → NumPy-Pipeline

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Headless-Erfassungsskript direkt auf der Hardware (ohne Backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Funktionsprüfung vor dem Anschließen eines 4-Kamera-Arrays

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Äquivalentes Aufnahmerezept (reines Python)

`run_project_recipe()` führt ein CLI-Rezept unverändert aus (siehe [Projektordner & Rezepte](#project-folder--recipes)). In Python geschrieben lauten die gleichen Schritte wie folgt:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP-Server (KI-Assistenten)

`chloros-mcp` ermöglicht es einem KI-Assistenten, der das [Model Context Protocol](https://modelcontextprotocol.io) (MCP) spricht – wie beispielsweise Claude Desktop, Claude Code oder Cursor –, Chloros zu steuern: Bilder verarbeiten (ein Projekt erstellen, einen Ordner mit Aufnahmen importieren, Einstellungen auswählen, verarbeiten, über die Ergebnisse des Durchlaufs berichten) sowie LATTICE-Kameras, Arrays und DAQ-Lichtsensoren zu steuern (verbinden, Vorschau anzeigen, Belichtung und Farbe einstellen, Aufnahmen machen, aufzeichnen, Firmware aktualisieren). Seine Werkzeuge decken die Funktionen dieses SDKs ab. Sie geben Ihre Anweisungen in natürlicher Sprache ein, und der Assistent ruft die Werkzeuge des Servers auf:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Der Server basiert auf diesem SDK: Die Verarbeitungswerkzeuge rufen [`ChlorosLocal`](#image-processing--chloroslocal) auf, und die Hardware-Tools die [smart-connect-Aufrufe](#smart-connect-for-lattice-cameras), [DAQ-Sitzungen](#daq-sensor-sessions), `ChlorosProject` und die [Kamera-Dienstprogramme](#camera-utilities), die alle mit demselben Backend kommunizieren, das auch die Chloros-App nutzt. Ihre Anfragen enthalten `X-Chloros-Client: sdk`, sodass das Backend die Anmelde- und Planprüfungen des SDKs anwendet und die Regeln des SDKs damit einhergehen (Texture Aware wird beispielsweise für LATTICE-Aufnahmen abgelehnt).

### Installation

Der Server ist im `chloros-sdk`-Paket enthalten; es muss nichts weiter installiert werden. Das SDK installiert einen Befehl `chloros-mcp`, der mit `python -m chloros_sdk.mcp_server` identisch ist:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** Das Chloros-Installationsprogramm installiert das SDK per pip in Ihr Python, wodurch `chloros-mcp.exe` in den Ordner `Scripts` dieses Pythons kopiert wird.
- **Linux (.deb):** Das Paket installiert das SDK für das System `python3` und legt den Befehl `chloros-mcp` in `/usr/bin` ab, sodass er auf dem `PATH`. In einer virtualenv fügt `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` `chloros-mcp` zur virtualenv hinzu.

Der Server verwendet ausschließlich die Standardbibliothek auf Basis des SDKs (Python 3.7+). Ein MCP-Client startet ihn und kommuniziert über stdin/stdout mit ihm; Sie führen ihn nicht selbst aus, außer mit `--help`.

### Client-Einrichtung

**Claude Desktop:**Fügen Sie den Server zu „`claude_desktop_config.json`“ hinzu (unter Windows: „`%APPDATA%\Claude\claude_desktop_config.json`“; über**Einstellungen > Entwickler > Konfiguration bearbeiten** können Sie dies aufrufen), und starten Sie anschließend Claude Desktop neu:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude-Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** derselbe Eintrag wie bei Claude Desktop (`mcpServers`), und zwar in `~/.cursor/mcp.json` (in jedem Projekt; unter Windows in `%USERPROFILE%\.cursor\mcp.json`) oder in `.cursor/mcp.json` in einem Projekt.

Optionen werden in `args` oder nach dem Befehl für Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`) eingegeben. Um anstelle des `chloros-mcp` auf dem `PATH` ein bestimmtes Python-SDK zu verwenden, führen Sie das Modul mit diesem Python-SDK aus (dessen vollständiger Pfad als `command` anzugeben ist, sofern es nicht das erste `python3` auf dem `PATH` ist):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Ein Client startet den Server ohne Ihre Shell. Wenn also gemeldet wird, dass der Server nicht gestartet werden konnte, befindet sich `chloros-mcp` wahrscheinlich nicht auf dem `PATH`, den der Client sieht. Geben Sie stattdessen den vollständigen Pfad an (`where chloros-mcp` unter Windows, `which chloros-mcp` unter Linux, wobei die .deb-Datei `/usr/bin/chloros-mcp` ist) oder führen Sie das Modul mit der Python-Version aus, die das SDK enthält.

### Optionen

| Option | Standard | Funktion |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Das zu verwendende Chloros-Backend. |
| `--no-auto-start` | off | Kein Backend starten: Verwende ein bereits laufendes, z. B. das der Chloros-App, den Linux-Backend-Dienst oder einen Tunnel zu einem Ordner auf einem anderen Rechner (siehe [Remote-Backend-Modus](#remote-backend-mode-pip-only-host-via-tunnel); die Ordnerpfade müssen dann auf beiden Rechnern denselben Ordner benennen). |
| `--backend-exe PATH` | dort, wo Chloros es installiert | Das Backend, das gestartet werden soll, wenn keines läuft, für eine Chloros-Installation, nach der das SDK-verzeichnis nicht sucht. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` oder `ERROR`. Protokolle werden an `stderr` ausgegeben, was die meisten Clients in ihrem MCP-Protokoll anzeigen; `stdout` überträgt nur Protokollmeldungen. |

### Tools

Die Argumente jedes Tools entsprechen denen des SDK-Aufrufs, den es umschließt; der Client listet jedes einzelne mit seinem vollständigen Schema auf. Pfade sind absolut.

**Verarbeitung**

| Tool | SDK-Aufruf | Funktion |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Erstellt ein Projekt und macht es zum geöffneten Projekt, auf das die Verarbeitungswerkzeuge einwirken. Ein einfacher Name erstellt den Ordner im Standard-Projektordner; ein absoluter Pfad erstellt ihn dort. |
| `open_project` | `ChlorosLocal.open_project` | Öffnet ein gespeichertes Projekt und macht es zum geöffneten Projekt. Die Pfade eines verschobenen Projekts werden auf den Ordner umgeleitet, in dem es sich nun befindet; `relink_folder` (absolut) benennt den Ordner, in dem sich die anderen fehlenden Dateien nun befinden. Gibt `source_files` und `image_count` zurück. Keine gefunden: eine Fehlermeldung, die angibt, wo sie sich befanden, und `relink_folder` benennt; es ist nichts geöffnet. |
| `import_images` | `import_images` | Importiert Survey3-Paare `.RAW`+`.JPG` sowie LATTICE-Aufnahmen `.tif`-Erfassungen und DAQ `.daq`-Aufzeichnungen; gibt `count` (Bilder) und `scan_count` (Aufzeichnungen) zurück. |
| `configure_processing` | `configure` | Ändert nur die übergebenen Einstellungen und keine anderen (Werte: [Unterstützte Werte](#supported-values)); `custom_settings` sendet die gesamte Konfiguration wie angegeben. |
| `get_project_settings` | `get_config` | Liest die gespeicherten Verarbeitungs-, Export- und Indexeinstellungen des geöffneten Projekts. |
| `start_processing` | `process(wait=False)` | Beginnt mit der Verarbeitung des geöffneten Projekts und kehrt sofort zurück. |
| `get_processing_status` | `get_status` | Meldet `state` (`idle`, `starting`, `processing` oder `finished`) sowie den Fortschritt; nach Abschluss werden die Werte `summary`, `export_check_failure`, wenn die Exporte nicht verifiziert werden konnten, und `image_failure`, wenn Bilder verloren gingen, geschrieben; die anderen Produkte werden geschrieben; der Lauf wird als fehlgeschlagen gemeldet. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Erstellt ein Projekt für einen Ordner, importiert es, konfiguriert den Lauf und startet ihn in einem einzigen Aufruf (`lattice: true` für LATTICE-Aufnahmen). Während ein Lauf läuft, lehnt das System dies ab, bevor es irgendetwas erstellt, und das Backend lehnt zu diesem Zeitpunkt auch eine Projektänderung durch ein beliebiges Tool ab. |
| `export_light_sensor` | `export_light_sensor` | Schreibt kalibrierte `.daq`- und `.csv`-Dateien für die Lichtsensor-Aufzeichnungen des Projekts aufzeichnet, ohne Bilder zu verarbeiten, und ruft dabei die Werkskalibrierung aus der MAPIR-Cloud ab, wenn diese nicht im Cache vorhanden ist (siehe [Lichtsensoraufzeichnungen](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Liest die Chloros-Audit-Metadaten eines verarbeiteten Bildes (Quelle, Kalibrierung, angewendete Stufen). |

**Kameras und Arrays**

| Tool | SDK-Aufruf | Funktion |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Was derzeit in jedem Client verbunden ist, jeweils mit `connected_here`; sowie die laufenden Aufträge auf dem Server. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kameras und Lichtsensoren, die angeschlossen werden könnten. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Der Live-Status einer Kamera, eines Arrays (mit dessen Leistungsfähigkeit und Zustand) oder eines Lichtsensors. |
| `analyze_array_network` | `analyze_array_network` | Was dieser Host und dieses Netzwerk für ein vorgeschlagenes Array übertragen können. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Eine Kamera oder ein synchronisiertes Array (Master zuerst) verbinden. |
| `connect_project_devices` | `open_project(...).connect_all` | Alle Geräte verbinden, die in einem gespeicherten Projekt erfasst sind, einschließlich ihrer Einstellungen. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Eine Kamera, ein Array oder einen Lichtsensor freigeben (siehe „Verhalten“). `keep_settings: true` auf einem Array speichert die Einstellungen jeder Kamera in einem eigenen Speicher, `clear_settings: true` setzt diesen Speicher zurück (das Ergebnis enthält `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Die Geräte des Projekts, zusammen. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Kameraeinstellungen lesen oder schreiben, anhand des benutzerfreundlichen Namens oder des GenIC-am-Knotennamen. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Die Bildrate eines Arrays festlegen oder auf die automatische Bildrate zurücksetzen. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Eine Aufnahme (`fastest` und `wait_for_exposure` sind `capture_fastest` und die Aufnahme mit festgelegter Belichtung). Fehlt `output_dir`, wird die Aufnahme im Ordner des geöffneten Projekts`captures`-Ordner des geöffneten Projekts gespeichert, und die Dateien werden dem Projekt hinzugefügt (wobei dies zurückgestellt wird, bis ein Verarbeitungslauf beendet ist); dies geht aus der Antwort `project_add` hervor. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Kontinuierliche oder Intervallaufnahmen als Auftrag. |
| `start_array_recording` | `ArraySession.record` / `burst` | Ein Video mit kombiniertem Index oder eine Rohbildserie als Auftrag. |
| `build_burst_video` | `ArraySession.build_video` | Eine Bildserie als kalibriertes Video neu verarbeiten, als Auftrag. |
| `get_job_status` / `stop_job` | | Einen Auftrag verfolgen oder stoppen. |
| `get_camera_preview` | | Live-Vorschau jeder Kamera, als Bild zurückgegeben. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Ein Einzelbild (oder ein synchronisierter Satz) auf einer Verarbeitungsebene, ohne Speicherung: Statistiken pro Ebene und jeweils ein PNG-Bild. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Ausrichtung eines Arrays. |

**Lichtsensoren**

| Tool | SDK-Aufruf | Funktion |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Ein DAQ-U, DAQ-M oder DAQ-E anschließen (`cap_id` legt die Kappenkorrektur fest). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Die neuesten Messwerte: Zeitstempel, Sättigung, Spektralbereich und Spitzenwert sowie das in Wellenlängenbins gemittelte Spektrum (`full_spectrum` für jeden Punkt). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Den Datenstrom starten oder stoppen. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Eine `.daq`-Datei als Job aufzeichnen; `duration_s` stoppt ihn automatisch. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Die Kappenkorrektur (`none` für einen ungeschützten Sensor). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Welcher Sensor liefert die Reflexionswerte für die jeweilige Kamera und das jeweilige Array? |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Die Band-Sets sowie die Umgebungs- und Objektreflexionswerte und Indizes der beiden Streaming-Sensoren. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Eigene Einstellungen eines DAQ-E, die im Gerät gespeichert sind. |

**Kamera-Dienstprogramme** ([Kamera-Dienstprogramme](#camera-utilities))

| Tool | SDK-Aufruf | Funktion |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` und deren Setter | Die Anzeigefarben einer RGB-Kamera: Profil und Farbtemperatur, Sättigung / Kontrast, Lichter. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Einmaliger Weißabgleich oder Auto / Aus / feste Verhältnisse. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Kamera-Voreinstellungen. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Die Filtermodelle oder ein einzelnes Modell nach Name. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Firmware-Versionen und ein Flash (standardmäßig ein Auftrag). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Energie- und thermischer Zustand; Ruhezustand oder aktiv. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Koeffizienten eines im Blickfeld befindlichen Kalibrierungsziels. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Die Netzwerkadapter der Kamera, eine Schätzung und die Korrektur (nur für Administratoren). |
| `apply_vignette_correction` | `apply_vignette_correction` | Vignettierungskorrektur für einen Ordner mit Bildern. |
| `read_dls` | `read_dls` | Ein DLS-Messwert. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Hardware und Laufzeitumgebung des Hosts. |

**Transiente Kameras, Ausrichtungsprofile und Indizes** ([Transiente Arrays, Ausrichtungsprofile und Indizes](#transient-arrays-alignment-profiles--indices)). Jedes Tool hier, das Kameras öffnet, verwendet `keep_settings` / `clear_settings` (Standardwert: false): siehe [Kameraspeicher](#camera-memory--keep_settings--clear_settings).

| Tool | SDK-Aufruf | Funktion |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Eine Kamera aufzeichnen oder mehrere synchron streamen, als Job (`duration_s` beendet den Vorgang automatisch). |
| `capture_cameras_transient` | `capture_cameras_transient` | Eine synchronisierte Aufnahme ohne Verbindung zu einem Array. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Die Überprüfung der Triggersynchronisation und welche Kameras sich auf der Synchronisationsleitung befinden. |
| `analyze_array_models` | `analyze_array_models` | Was ein Array dieser Modelle in diesem Netzwerk leisten würde. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Eine Ausrichtungsprofil-Datei berechnen und speichern (`options` enthält die detaillierteren Solver-Optionen). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Ein ausgerichtetes Multiband-Bild oder eine Folge solcher Bilder als Auftrag. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Der Inhalt einer Profildatei; deren Bandreihenfolge und aktivierte Kameras. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Index-Voreinstellungen und Farbverläufe; ein Index für ein Mehrband-TIFF; Aufnahme und Indexierung in einem Schritt. |

**Zeitsynchronisation, Projektordner und Rezepte** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Projektordner & Rezepte](#project-folder--recipes))

| Tool | SDK-Aufruf | Funktion |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | Der Zustand des PTP-Grandmasters, seiner Peers und der PTP-Status jeder Array-Kamera. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Neustart; Prioritäten festlegen. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Der Ordner, in dem neue Projekte angelegt werden. |
| `run_project_recipe` | `run_project_recipe` | Das Rezept eines gespeicherten Projekts als Auftrag mit seinem Protokoll ausführen. Zunächst wird eine Überprüfung durchgeführt: Ein Rezept mit einem Fehler wird abgelehnt und es wird nichts ausgeführt; bei nicht verbundenen Arrays/Sensoren wird eine Warnung ausgegeben (sie werden nicht verbunden: zuerst `connect_project_devices`). `dry_run` bricht die Überprüfung ab und führt nichts aus. |

Es gibt kein Tool zum Öffnen eines gespeicherten Projekts zur Verarbeitung: Jeder Verarbeitungsjob beginnt mit `create_project` (oder `process_folder`).

### Voraussetzungen

- **Chloros muss auf demselben Computer installiert sein.** Das Chloros-Backend führt die Verarbeitung durch. Der Server kann stattdessen ein Backend verwenden, das über einen Tunnel erreichbar ist (`--backend-url` mit `--no-auto-start`, siehe [Optionen](#options)).
- **Auf diesem Computer angemeldet sein**, entweder in der Chloros-App oder über `chloros-cli login`. Der Server kann sich nicht für Sie anmelden: Ein Tool, das im abgemeldeten Zustand aufgerufen wird, gibt die Meldung „Nicht bei Chloros angemeldet“ zusammen mit dem Grund des Backends aus, und der Assistent informiert Sie darüber.
- **Ein beliebiger kostenpflichtiger Chloros+-Tarif (Copper oder höher).** Der Server ist über das SDK zugänglich, und der kostenlose Iron-Tarif bietet keinen solchen Zugriff: Ein Tool, das im kostenlosen Tarif aufgerufen wird, antwortet mit „In diesem Chloros-Tarif nicht verfügbar“ und gibt den Grund des Backends an.
- Python 3.7 oder neuer mit dem SDK sowie ein MCP-Client, der lokale (stdio-)Server startet.

### Verhalten

- **Das Backend.** Das Starten des Servers oder das Auflisten seiner Tools löst nichts aus; erst der erste Tool-Aufruf stellt eine Verbindung her. Wenn die Chloros-App (oder der Linux-Backend-Dienst) läuft, nutzt der Server dessen Backend und lässt es weiterlaufen. Läuft kein Backend, startet der Server das installierte Backend im Hintergrund. Wenn der Server beendet wird (Sie beenden den Client oder dieser startet den Server neu), stoppt er die von ihm gestarteten Erfassungsaufträge und Lichtsensoraufzeichnungen und gibt die von ihm verbundenen Geräte frei; das Backend läuft weiter, sodass ein noch nicht abgeschlossener Verarbeitungslauf bis zum Ende fortgesetzt wird; er beendet sich selbst nach 30 Minuten, wenn er nicht genutzt wird. Bei `--no-auto-start` startet der Server niemals ein Backend: Ein Toolaufruf, bei dem kein Backend läuft, fordert Sie auf, die Chloros-App zu öffnen, und der nächste Aufruf nach dem Öffnen stellt die Verbindung her.
- **Ein geöffnetes Projekt.** Das Backend hat ein geöffnetes Projekt, das mit der Chloros-App geteilt wird, solange diese geöffnet ist; arbeiten Sie daher nicht an einem anderen Projekt in der App, während der Assistent läuft.
- **Nur die übergebenen Einstellungen ändern sich.** `configure_processing` belässt jede Einstellung, die nicht übergeben wird, auf ihrem gespeicherten Wert (das SDKs `configure()` setzt Debayer, Vignettenkorrektur, Reflexionskalibrierung, PPK und Exportformat auf ihre Standardwerte zurück, wenn sie nicht übergeben werden; der Server füllt sie aus dem Projekt ein). Kann er die gespeicherten Einstellungen des Projekts nicht lesen, nimmt er keine Änderungen vor und gibt dies an, es sei denn, alle fünf Parameter werden übergeben. `indices` ersetzt die Indexliste des Projekts. Das Ergebnis listet auf, was als `settings_sent` gesendet wurde, und benennt unter `warnings` alle Indizes, die die Kamerabänder nicht bilden können.
- **Die Verarbeitung erfolgt nach dem „Start-then-Poll“-Prinzip.** `start_processing` kehrt zurück, sobald der Lauf beginnt, der dann Minuten bis Stunden dauert, und der Assistent schließt sich mit `get_processing_status` an. `state` ist entweder `idle` (keine Verarbeitung), `starting` (das Backend hat den Lauf noch nicht gemeldet, bis zu 20 s), `processing` oder `finished`, mit der [Zusammenfassung](#post-run-summary--hints): was exportiert wurde, sowie Hinweise, wenn weniger als erwartet exportiert wurde (keine Bilder, kein Kalibrierungsziel gefunden). Die Ergebnisse werden wie in der App unter dem Projektordner gespeichert. `start_processing` wird während eines laufenden Laufs abgelehnt, und ein Backend, das nicht mehr antwortet, wird als Fehler gemeldet, nicht als abgeschlossener Lauf.
- **Was der Assistent sieht.** Die Warnungen des SDK werden im Tool-Ergebnis unter `warnings` angezeigt, und Fehler (abgemeldet, Plan, ungültiger Pfad) werden als Fehlertext ausgegeben, den der Assistent lesen und an Sie weiterleiten kann. Eine Liste mit mehr als 20 Einträgen wird auf die ersten 20 gekürzt, wobei die vollständige Länge unter `<key>_total` angegeben wird. Pfade müssen absolut sein.
- **Geräte werden gemeinsam genutzt.** Kameras, Arrays und Lichtsensoren befinden sich im Backend, sodass der Assistent ein mit der Chloros-App verbundenes Gerät steuern kann und die vorgenommenen Änderungen in der App angezeigt werden. `disconnect_device` gibt aus, was der Server verbunden hat; ein Gerät, das ein anderer Client verbunden hat (die App, `chloros-cli`, ein Skript, ein anderes `chloros-mcp`) wird nur mit `force: true` freigegeben, was dem `force_disconnect` des SDK entspricht, und die Ablehnung nennt den Namen desjenigen, der die Verbindung hergestellt hat, sofern das Backend dies weiß; Ein Gerät, das das Backend überhaupt nicht verwaltet, wird als nicht verbunden gemeldet (`released: false`, `connected: false`, nichts freizugeben). Eine vom Backend ausgeführte Aufzeichnung oder ein Stream führt zu einer Ansicht mit den Werten `get_job_status` sowie `counts` – `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (Frame-Gruppen für einen Multi-Kamera-Stream) — sowie eine einzeilige `summary`-Meldung („220 Frames geschrieben, 55 verworfen, 0 fehlgeschlagen: …“) während des Laufs und nach Abschluss die endgültigen Zahlen; die Auflistung eines anderen Clients dazu (`other_clients_jobs`) enthält dieselben (`list_cameras()`, `list_arrays()` und `list_daq_sensors()` enthalten jeweils die`connected_by`: `client`, `label`, `name`, `at`). Jedes `chloros-mcp` meldet sich beim Backend als `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, gesendet als `X-Chloros-Client-Label`; jedes SDK-Skript kann dies festlegen). Das Beenden der Lichtsensoraufzeichnung eines anderen Clients erfordert ebenfalls `force`.
- **Lang andauernde Arbeit ist ein Job.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, und jede angeforderte Erfassung, Lösung oder Indizierung für `background` gibt sofort ein `job_id` zurück; darauf folgt `get_job_status` und `stop_job` beendet . Ein Lauf wird fortgesetzt, bis er gestoppt wird, es sei denn, Sie geben `max_captures`, `duration_s` (oder, bei einem Burst, `max_frames`) ein; `min_free_gb` weigert sich zu starten, wenn auf dem Laufwerk nicht genügend freier Speicherplatz vorhanden ist. Es gilt keine Begrenzung, es sei denn, Sie legen eine fest. Eine Aufzeichnung oder ein Stream, den das Backend ausführt (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) behält die Job-ID des Backends bei, sodass ein anderer `chloros-mcp` – der später gestartet wurde – ihn anhand dieser ID findet: `get_job_status` folgt darauf (und listet solche Jobs wie `other_clients_jobs`, ab `GET /api/camera/cli-jobs`, jeweils mit `started_by` auf), und `stop_job` beendet ihn erst mit `force: true`. Jeder andere Job (ein Erfassungslauf, eine Array-Aufzeichnung oder ein Burst, eine Hintergrunderfassung) läuft in dem `chloros-mcp`, der ihn gestartet hat, und nur dieser Server weiß davon: ein anderer, der danach fragt, gibt dies an. Ein angehaltener Job liest `stopping`, während er seine Warteschlange ausgibt, dann `finished` mit seinen endgültigen Zählwerten, 10 Minuten lang und von jedem `chloros-mcp`: derjenige, der den Vorgang gestartet hat, nimmt den erzwungenen Stopp eines anderen auf dieselbe Weise wahr. `get_job_status` wartet niemals auf einen Stopp: Während eines Stopps (seines eigenen `stop_job`, eines anderen Clients’oder den Stopp, der die Kameras eines von selbst beendeten Auftrags freigibt) antwortet er sofort mit `stopping` und den Live-Zählwerten, und `stop_job` eines bereits stoppenden Auftrags gibt dies an. Ein beendeter Auftrag gibt den Grund in `stop_reason` an: seinen eigenen Grund (`no_frames`, `disk_full`, `member_missing`, oder bei einem Array-Dauerbetrieb `count`, `duration`, `array_gone`), `limit reached` nur, wenn seine Zählwerte seinen `count` (oder das `max_frames` eines Bursts, das`duration_s`) erreicht wurde, `error`, `stopped` (mit `stopped_by`), wenn ein Client sie beendet hat, andernfalls `finished`.
- **Wohin die Aufzeichnungen gelangen.** In den Ordner „`captures`“ des geöffneten Projekts, wo die App sie speichert, sofern nicht „`output_dir`“ angegeben wird.
- **Bilder.** `get_camera_preview` und `grab_frames` liefern neben ihren JSON-Daten auch Bilder, die der Assistent anzeigen kann. `grab_frames`-Bilder werden zur Anzeige kontrastverstärkt; ihre statistischen Werte entsprechen den tatsächlichen Werten.
- **Wenn der Server beendet wird**, werden die von ihm gestarteten Aufträge gestoppt und eine von ihm gestartete Lichtsensoraufzeichnung geschlossen. Wenn der Server das Backend gestartet hat, gibt er vor dem Beenden des Backends auch die von ihm verbundenen Geräte frei; im Backend der App bleiben sie verbunden.
- **Protokoll.** Nur stdio (kein HTTP-Endpunkt), nur Tools (keine Ressourcen oder Eingabeaufforderungen), Text- und Bildeergebnisse, MCP-Protokollversionen vom 05.11.2024 bis 25.11.2025.**Was nicht enthalten ist.** Ein- oder Ausloggen (das Konto gehört Ihnen; der Server kann sich nicht anmelden) sowie das Beenden eines Backends, das der Server nicht gestartet hat. Für alle anderen Funktionen, die dieses SDK in Bezug auf Verarbeitung und Hardware bietet, gibt es ein Tool. (Die Bereitstellung eines DAQ-E – dessen Name, Authentifizierungstoken, OTA-Passwort, Signaturanforderung, Werksreset – sowie die direkten DAQ-Server sind ausschließlich `chloros-cli` vorbehalten, auch im SDK: Ein falscher Name oder ein falsches Token sperrt jede Chloros-Installation aus dem Gerät aus.)

---

## Automatischer Start des Backends

Die Smart-Connect-Einstiegspunkte – `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` und `discover_daq_sensors` – sind „Thin“-HTTP-Clients, die davon ausgehen, dass ein Backend auf `127.0.0.1:5000` (der Standard-URL der Smart-Connect-Oberfläche) auf Verbindungen wartet. Wenn die GUI oder die CLI bereits läuft, ist dies der Fall. Bei einem reinen Skript ist dies möglicherweise nicht der Fall – daher starten diese Funktionen **die mitgelieferte Backend-Binärdatei automatisch** (fensterlos, genau wie `ChlorosLocal`) vor ihrem ersten Aufruf und warten anschließend bis zu 60 Sekunden, bis sie hochgefahren ist. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` und `analyze_array_network()` starten niemals eines.

Regeln:

- **Es wird immer nur eine lokale URL-Adresse generiert.** Ein `backend_url`, der auf `localhost` / `127.0.0.1` / `[::1]` ist zulässig; jeder andere Host wird als Rechner eines anderen Benutzers angesehen und niemals gestartet.
- **Ein Backend, das beendet wird, bevor es antwortet, beendet das Warten.** Der Aufruf bricht sofort mit dem Exit-Code, dem ausgeführten Befehl und den letzten Zeilen, die das Backend ausgegeben hat, ab, anstatt die 60 Sekunden abzuwarten. Der Exit-Code 98 bildet eine Ausnahme: Ein anderes Backend hält bereits den Port oder die Sperrdatei (die der Chloros-App, des Backend-Dienstes oder eines anderen Skripts, das noch startet), sodass der Aufruf weiter darauf wartet, dass dieses antwortet. `ChlorosLocal` verhält sich ebenso.
- **Das Backend bleibt zur Wiederverwendung laufen** (genau wie bei der CLI) – es erfolgt kein implizites Herunterfahren, wenn Ihr Skript beendet wird. Bei erneuter Ausführung des Skripts wird das aktive Backend wiederverwendet.
- **Deaktivieren Sie dies mit `auto_start_backend=False`** bei jedem dieser Aufrufe (z. B. wenn Sie auf ein entferntes Backend verweisen oder den Lebenszyklus des Backends selbst verwalten).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Wenn die mitgelieferte Binärdatei nicht gefunden oder gestartet werden kann, löst der nachfolgende HTTP-Aufruf einen verarbeitbaren, **plattformabhängigen** `ChlorosConnectError`-Fehler aus, anstatt nur einen einfachen Trace wegen einer abgelehnten Verbindung — unter Windows verweist er auf die Desktop-App oder einen `chloros-cli`-Befehl; unter Linux (ohne GUI) verweist er auf einen `chloros-cli`-Befehl oder den `.deb`.

---

## Umgebung & Header

Das SDK kennzeichnet jeden HTTP-Aufruf im Backend mit `X-Chloros-Client: sdk`. Das Backend wendet die Lizenzierungsregeln für SDK und CLI an (Anmeldung **und** ein kostenpflichtiger Chloros+-Tarif erforderlich) anstelle des kostenlosen GUI-Zugangs. Dies wird beim Import automatisch festgelegt – Sie müssen nichts unternehmen.

Der Header wird jedem `requests`- bzw. `urllib`-Aufruf, dessen URL `localhost` oder `127.0.0.1` enthält, unabhängig vom Port – einschließlich Ihrer eigenen lokalen Dienste – und ersetzt einen Wert, den Sie selbst festgelegt haben. Aufrufe an andere Hosts bleiben davon unberührt.

Überschreiben Sie die Backend-URL, indem Sie `backend_url=` übergeben (`ChlorosLocal` akzeptiert `backend_url=` oder `api_url=`; `process_folder()` und `process_lattice_capture()` akzeptieren nur `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Ein installiertes Backend lauscht ausschließlich auf dem Loopback, sodass ein LAN-`backend_url` es nicht erreichen kann; siehe [Remote-Backend-Modus](#remote-backend-mode-pip-only-host-via-tunnel) für das Tunnel-Muster.)

---

## Versionierung & Kompatibilität

- Die SDK-Version wird als `chloros_sdk.__version__` bereitgestellt.
- Das SDK ist an das Verhalten der mitgelieferten Backend-Version gebunden. Die Kombination eines älteren SDKs mit einem neueren Backend funktioniert in der Regel (vorwärtskompatible Endpunkte), aber die Kombination eines neueren SDKs mit einem älteren Backend kann bei neuen Endpunkten zu `404`-Fehlern führen – aktualisieren Sie die Desktop-App entsprechend.
- Die Smart-Connect-Oberfläche (`connect_camera` / `connect_array` / `connect_daq_sensor`) und der Endpunkt zur Netzwerkanalyse geben stabile JSON-Schemas zurück; neue Felder werden zusätzlich hinzugefügt.

---

## Hinweise zur Fehlerbehebung

- **`ChlorosAuthenticationError: Login required`** (aus `ChlorosLocal`; die Smart-Connect-Aufrufe lösen aus demselben Grund den Fehler `ChlorosConnectError: Login required. …` aus) → Führen Sie `chloros-cli login EMAIL PASSWORD` einmal auf diesem Rechner aus oder melden Sie sich über die Chloros-Desktop-App an.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (oder `Failed to create project: …` aus `create_project()`) → Lange Pfade unter Windows sind auf diesem Rechner deaktiviert (Windows-Standard), daher darf kein Pfad länger als 260 Zeichen sein, und Produkte befinden sich bis zu etwa 130 Zeichen unterhalb des Projektordners. Es wurde nichts verarbeitet. Die Meldung gibt den längsten Projektordner an, der noch passt: Verwenden Sie einen kürzeren oder aktivieren Sie lange Windows-Pfade und starten Sie Chloros neu. Der Windows-Installer bietet dies als Option „Windows-Langpfade aktivieren“ an, die standardmäßig aktiviert ist; andernfalls verwenden Sie die Gruppenrichtlinie: *Computerkonfiguration > Administrative Vorlagen > System > Dateisystem > Win32-Langpfade aktivieren*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Der Smart-Connect-Aufruf startet automatisch ein lokales Backend, daher tritt dieser Fehler nur auf, wenn die mitgelieferte Binärdatei nicht gefunden oder gestartet werden kann (z. B. bei einem reinen pip-only-Host ohne Desktop-Paket). Die Meldung ist plattformabhängig: Öffnen Sie unter Windows die Desktop-App oder führen Sie einen beliebigen `chloros-cli`-Befehl aus; unter Linux führen Sie einen `chloros-cli`-Befehl aus (es gibt keine GUI) oder installieren Sie `.deb`. Für ein Remote-Backend übergeben Sie `backend_url=` (und `auto_start_backend=False`) übergeben.
- **`CAMERA_AVAILABLE == False`** beim Import → `lattice_sdk` konnte nicht geladen werden: Die Arena-SDK-Laufzeitumgebung oder OpenCV (`cv2`) kann in dieser Python-Version nicht importiert werden. Die backendgesteuerte Oberfläche (`connect_*`, wird über ein Backend projiziert, `ChlorosLocal`) funktioniert weiterhin.
- **Array verbindet sich bei vollem Sensor, aber niedriger Bildrate**→ So vorgesehen: Der Modus „auto“ behält den vollen Sensor bei und lässt die Bildrate auf das sinken, was die Leitung überträgt. Verwenden Sie `binning=2` / `4` oder ein kleineres `width` / `height` für mehr Bilder pro Sekunde, oder aktivieren Sie Jumbo-Frames / rüsten Sie die Verbindung auf. Binning und ROI decken eine aggregierte Überbelegung**nicht** ab (`oversubscribed: true`, fps-Felder 0): Zu viele Kameras für die Verbindung lassen sich nicht durch Binning/ROI beheben – reduzieren Sie die Anzahl der Kameras, aktivieren Sie Jumbo-Frames oder wechseln Sie zu einer schnelleren Netzwerkkarte (siehe [Überbelegung](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` meldet den Empfangsring der Netzwerkkarte als sehr klein (~0,26 MB) / Verbindungsgates mit „FRAMES WILL DROP“** → Der Empfangsring der Host-Netzwerkkarte befindet sich auf seinem Standardwert (wird nach einem Netzwerkkartentreiber-Update oft auf 32 zurückgesetzt). Bei einem Realtek-USB-10-GbE-Adapter setzen Sie `ReceiveBufferLen=256` und `PendingReceives=64` (mit erhöhten Rechten) und starten Sie anschließend das Backend neu, damit der Ring neu gelesen wird. Vollständige Vorgehensweise: [CLI-Referenz → Einrichtung und Optimierung der Host-Netzwerkkarte](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Der Host hängt sich beim Neustart/Herunterfahren auf, später treten WMI-Fehler `Invalid class` auf / Netzwerkkarte lässt sich nicht aktivieren** → Veralteter USB-10-GbE-Treiber verursacht `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aktualisieren Sie den Adaptertreiber auf eine aktuelle Version (≥ 2026) und wenden Sie die Einstellungen für den Empfangsring erneut an. Siehe [CLI-Referenz → Einrichtung und Optimierung der Host-Netzwerkkarte](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Array-Erfassung schlägt mit HTTP-Fehler 503 „No fresh synced frame group“ fehl** → Das Array hat innerhalb der Wartezeit keine neue Frame-Gruppe veröffentlicht. Überprüfen Sie `arr.status()`: `stalled: True` (oder `streaming: False`) zusammen mit `actual_fps`. Wenn `actual_fps` den Wert 0 hat, bedeutet dies, dass die Erfassungsschleife die Veröffentlichung eingestellt hat – trennen Sie die Verbindung zum Array und stellen Sie sie erneut her. Wenn `stalled` „False“ ist und `actual_fps` größer als 0 ist, versuchen Sie es erneut. Unmittelbar nach `connect_array()` wartet eine Erfassung bereits bis zu 20 s auf die erste Gruppe.
- **`status()` zeigt `link_speed_mbps: 4294` an** → Windows konnte die Geschwindigkeit des Adapters nicht auslesen; dies ist typisch für USB-Ethernet-Adapter und stellt keinen Fehler dar. Siehe [Erwartete Frameraten](#expected-frame-rates).
- **`update_firmware()` schlägt mit `device never connected back on TCP <port>` fehl** → Eine Firewall hat die Verbindung des DAQ-E zurück zu diesem Computer blockiert. Der Windows-Installer fügt die Regel hinzu (siehe [DAQ-E-Geräteeinstellungen](#daq-e-device-settings)); bei einer anderen Firewall müssen eingehende TCP-Verbindungen vom Subnetz des Sensors zum Backend zugelassen werden.
- **Reflexionsgrad abgelehnt** → Für die Reflektanzmessung im absoluten Maßstab muss ein aktives DAQ-Gerät an die Kamera (oder das Array) gebunden sein. Führen Sie die Bindung entweder über die Benutzeroberfläche durch oder verwenden Sie `processing="radiance"` (W/m²/sr/nm), wofür kein gekoppelter Sensor erforderlich ist.
- **Die Erfassung mit `smart=True` dauert länger als erwartet** → Die AE-Konvergenz hängt von der Dynamik der Szene ab. Verwenden Sie bei `ArrayHandle.capture_smart()` ein breiteres `exposure_tolerance_pct`, verkürzen Sie `stability_window_s` oder senken Sie `settle_timeout_s`, um einen schnelleren, weniger stabilen Auslöser zu erhalten.

---

## Siehe auch

- [CLI-Referenz](cli-reference.md) – Jeder CLI-Unterbefehl entspricht einem SDK-Aufruf.
- [DAQ-Lichtsensoren](https://mapir.gitbook.io/chloros/daq-light-sensors) im Chloros-Handbuch – die Sensoren, ihre Nennwerte und kalibrierten Messbereiche, Aufzeichnung, DAQ-E-Vernetzung.
- [LATTICE-Kameras](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) im Chloros-Handbuch – Anschluss, Kameraeinstellungen, Aufnahmemodi, Arrays.
- Online-Dokumentation: `https://mapir.gitbook.io/chloros/api-python-sdk`
