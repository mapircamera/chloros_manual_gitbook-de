---
metaLinks: {}
---

# Erste Schritte

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros ist eine Softwareanwendung von [MAPIR](https://www.mapir.camera) zur Verarbeitung multispektraler Bilder, zur Live-Steuerung von MAPIR-Hardware und zur Aufzeichnung von Sensordaten. Chloros 1.3.0 unterstützt die gesamte MAPIR-Produktfamilie:

* **Survey3-Kameras** – Verarbeitung von RAW+JPG-Aufnahmen zu kalibrierten Reflektanz- und Vegetationsindexkarten. Siehe [Unterstützte Kameras](supported-cameras.md).
* **LATTICE-Kameras** – Verbinden Sie die GigE-Multispektralkameramodule live, einzeln oder als synchronisierte Mehrkamera-Arrays: Vorschau, Aufnahme und Verarbeitung zu kalibrierten Strahlungs- und Reflektanzprodukten. Siehe den [LATTICE-Abschnitt](lattice/README.md).
* **DAQ-Lichtsensoren** — DAQ-U (USB), DAQ-M (Bluetooth) sowie DAQ-E und DAQ-E-S (Ethernet) Spektralsensoren: kalibrierte Live-Spektren, `.daq`-Aufzeichnungen sowie von der nach unten gerichteten Beleuchtung für die Reflektanzverarbeitung. Siehe den [DAQ-Abschnitt](daq/README.md).

{% hint style="success" %}
**Neuerungen in Chloros 1.3.0**: KI-Assistenten können Chloros über `chloros-mcp` steuern, und die CLI, das SDK sowie `chloros-mcp` bieten dieselben Funktionen. Die LATTICE-Index-Produkte wurden verbessert, Vollsensor-Arrays sind schneller und funktionieren in jedem Netzwerk, und das Backend läuft zwischen den Befehlen weiter. Siehe die Liste der Neuerungen unten und [Download](download.md) für die Installationsprogramme.
{% endhint %}

{% hint style="info" %}
**Verwenden Sie Chloros mit einem KI-Assistenten?** Dieses Handbuch ist genau dafür konzipiert. Weisen Sie Ihren Assistenten auf Folgendes hin:

* `https://mapir.gitbook.io/chloros/llms.txt` – maschinenlesbarer Index aller Seiten.
* Beliebige Seite als reines Markdown – füge `.md` an die URL an (z. B. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* Die [CLI-Referenz](reference/cli-reference.md) und die [SDK-Referenz](reference/sdk-reference.md) – vollständige Referenzseiten mit exakten Werten, die für die Verwendung durch LLMs verfasst wurden.

Beispiel-Prompt: *„Lies https://mapir.gitbook.io/chloros/reference/cli-reference.md, und schreibe dann ein Skript, das sich anmeldet und den Ordner ~/flights/flight_001 in GeoTIFFs mit Reflektanz- und NDVI-Werten verarbeitet.“*

Oder lassen Sie Chloros direkt ausführen: `chloros-mcp`, ein mit dem SDK installierter MCP-Server, funktioniert mit Claude Desktop, Claude Code, Cursor und anderen MCP-Clients.

Vollständige Anleitung: [Verwendung von Chloros mit KI-Assistenten](ai-assistants.md).
{% endhint %}

***

## Neuerungen in Chloros 1.3.0

* **KI-Assistenten können Chloros steuern.** `chloros-mcp`, ein mit dem Python-SDK installierter MCP-Server, ermöglicht es Claude Desktop, Claude Code, Cursor und anderen MCP-Clients, Bilddaten zu verarbeiten und LATTICE-Kameras, Arrays sowie DAQ-Lichtsensoren zu betreiben. Siehe [Verwendung von Chloros mit KI-Assistenten](ai-assistants.md).
* **Die CLI, das SDK und `chloros-mcp` bieten dieselben Funktionen.** Aus Sicherheitsgründen verbleiben nur die Anmeldung, die DAQ-E-Bereitstellung und die direkten DAQ-Server bei der CLI. Einige Skripte der Version 1.2.0 müssen angepasst werden: siehe **Änderungen in 1.3.0** am Anfang der [CLI-Referenz](reference/cli-reference.md) und der [SDK-Referenz](reference/sdk-reference.md).
* **Verbesserte LATTICE-Indexprodukte.** Der NDVI-Index und die anderen aus LATTICE-Aufnahmen exportierten Indexprodukte wurden in jedem Verarbeitungsmodus verbessert.
* **Windows-Installationsoptionen.**Unter**Systemeinstellungen**bietet der Installer die Optionen**„Windows-Langpfade aktivieren“**und**„Chloros-Backend vom Microsoft Defender-Scan ausschließen“** an, die beide standardmäßig aktiviert sind. Die erste Option ermöglicht Projektordner mit Pfadlängen von mehr als 260 Zeichen; die zweite verhindert, dass Echtzeit-Scans die Kameraaufnahme verlangsamen (bei hohen Aufnahmeraten um etwa 30 %). Der Installer fügt nun außerdem den Ordner CLI nur einmal zu „`PATH`“ hinzu und entfernt die doppelten Einträge, die frühere Installationsprogramme hinterlassen haben. Siehe [Download](download.md).
* **Zu tiefe Projektordner werden von vornherein abgelehnt.** Wenn lange Pfade unter Windows deaktiviert sind (Windows-Standard), wird ein Projektordner, der die Pfadlänge von 260 Zeichen überschreitet, bereits vor der Ausführung abgelehnt; dabei wird der längste zulässige Projektordner angezeigt und erklärt, wie lange Pfade aktiviert werden können. Zuvor wurden bei einem solchen Lauf keine Produkte geschrieben. Bei aktivierten langen Pfaden behalten Produkte in sehr tiefen Ordnern nun ihre Metadaten (GPS- und XMP-Tags) bei.
* **Clamp-Masken.** Wenn die Bandentmischung eines LATTICE-Strahlungs- oder Reflektanzprodukts einen negativen Bandwert erzeugt, wird dieser Wert auf Null gekürzt, und eine pixelweise Maske in einem `clamp_masks/`-Ordner neben den Produktordnern protokolliert, welche Pixel und Bänder betroffen waren. Das Produkt benennt seine Maske im `Chloros:UnmixClampMask`-Tag. Live-Ansichten werden nun genau wie beim Export beschnitten. Siehe [Ausgabebildformate](output-image-formats.md).
* **Produkte beschreiben sich selbst.** Indexprodukte tragen ihre eigenen Verarbeitungs-Tags (`Chloros:ProcessingLevel=index`, den Indexausdruck und die Angabe, ob sie aus Radianz oder Reflektanz berechnet wurden) anstelle derjenigen der Rohdatenquelle. Survey3-Produkte tragen in jedem Verarbeitungsmodus Chloros-Verarbeitungs-Tags. LATTICE-Rohdaten und debayerte Dateien behalten die native Ausrichtung des Sensors bei und speichern diese in XMP; der Bildbetrachter dreht sie nur zur Anzeige.
* **Die Linux-Pakete werden entpackt ausgeliefert.** Das Backend und die CLI-Schnittstelle werden als Ordner unter `/usr/lib/chloros/` installiert, sodass sie sich nicht mehr bei jedem Start selbst entpacken. Auf x86_64 benötigt das Backend nicht mehr etwa 100 Sekunden zum Starten, und die Datei „`.deb`“ ist etwa halb so groß wie zuvor. Das mitgelieferte ExifTool läuft nun auf einer sauberen Ubuntu-Installation. Das `.deb` installiert nun auch das Python-SDK für das System `python3` (Installieren Sie es mit `sudo apt install ./chloros_1.3.0_<arch>.deb`), erhöht die vom Kernel benötigten Socket-Empfangspuffer für Kamerastreams und fügt einen Backend-Dienst hinzu, der unter Ihrem Benutzerkonto läuft. Das Jetson-Paket, `chloros_1.3.0_arm64_jp6.deb`, ist für JetPack 6 ausgelegt und verarbeitet Daten auf der GPU des Jetson mit der CUDA-Version von PyTorch aus dem JetPack. Unter Linux gelangt jedes exportierte Bild nun einmal auf die Festplatte (im Speicher geschrieben und mit Tags versehen, dann an den Bestimmungsort kopiert): Auf einem Jetson, wo die Festplatte das Tempo vorgibt, dauert ein LATTICE-Lauf mit 288 Bildern 14 Minuten statt 24. Siehe [Linux-Installation](linux/linux-installation.md).
* **LATTICE-OCN-Kameras lesen unter Sonneneinstrahlung genauer.** Strahlungsintensität, Reflexionsgrad und Indizes werden nach der Bandentmischung schnell korrigiert, sodass ein graues Zielobjekt bei Sonneneinstrahlung einen nahezu neutralen NDVI-Wert aufweist. RGN-Kameras bleiben unverändert.
* **Fehlerberichte innerhalb der App.** Über ein Fehler-Symbol oben auf der Registerkarte „Debug-Protokoll“ werden eine Beschreibung, bis zu 10 Bilder, die Protokolle und Systemdetails an MAPIR gesendet. Siehe [GUI: Navigation](navigation.md).
* **LATTICE-Arrays.**Arrays behalten standardmäßig den vollen Sensor bei, und der komprimierte 12-Bit-Transport (`--packed`, Standard: `auto`) erhöht die Bildraten bei vollem Sensor um etwa ein Drittel; ein Array funktioniert in jedem Netzwerk mit der dort zulässigen Bildrate (eine langsame Verbindung senkt die Bildrate, anstatt das Array abzulehnen oder dessen Bilder zu verkleinern); ein Array, dessen Kameras einen Neustart durchlaufen, erholt sich von selbst, wobei PTP wieder aktiviert wird; die automatische Belichtung senkt diese Rate bei schwachem Licht nicht mehr (sie füllt den Zeitraum auf und erhöht dann die Verstärkung); Die Serienaufnahme hält mit schnellen Arrays Schritt; eine**Highlights**-Steuerung für RGB-Kameras; und das SDK erhält die Funktionen `ArraySession`, `grab()`, `frame_stream()` und `continuous()`. Siehe den [LATTICE-Abschnitt](lattice/README.md).
* **Stabilere automatische Belichtung.** Die Belichtung bleibt bei konstantem Licht konstant und stabilisiert sich innerhalb weniger Sekunden nach einer Änderung; kleine helle Bereiche verdunkeln die Szene nicht mehr; bei manueller Belichtung bleibt die Verstärkung auf „Auto“ eingestellt; und eine Kamera, die bei starker Sonneneinstrahlung nicht mehr reagiert, wird auf dem für ihr Licht angemessenen Niveau gehalten, das vom Lichtsensor oder – falls kein Lichtsensor angeschlossen ist – anhand der eigenen Bilder der Kamera ermittelt wird. Siehe [Kameraeinstellungen](lattice/camera-settings.md).
* **Lichtsensoren.** **Unterstützung für DAQ-E-S:** Der feststehende Diffusor erfordert keine Korrektur der Blende, und jeder Messwert enthält die Neigungs-, Roll- und Pitch-Werte des Geräts. Eine Aufzeichnung gibt an, ob die Kappe ausgewählt oder angenommen wurde, und die Verarbeitung warnt bei einer angenommenen Kappe: DAQ-U, DAQ-M und DAQ-E werden ohne Kappe ausgeliefert, benennen Sie daher die montierte Kappe. Live-Aufzeichnungen von `.daq` bleiben auch bei einem Stromausfall erhalten (bewahren Sie eine `.daq-journal`-Datei zusammen mit ihrer `.daq` auf), und ein DAQ-E, das keine Daten mehr liefert, wird wieder angeschlossen, wobei die Lücke in der Aufzeichnung markiert ist. Ein Projekt, das ausschließlich Aufzeichnungen von Lichtsensoren enthält, listet diese auf und verarbeitet sie über die App und die CLI; eine mit dem `chloros_scripts` erstellte Aufzeichnung wird als das DAQ-Gerät aufgeführt, das sie erstellt hat, wobei Sie eine Obergrenze festlegen können. Bei jeder Erfassung wird der verwendete DAQ-Messwert gespeichert. Siehe den [DAQ-Abschnitt](daq/README.md).
* **Verarbeitung.**LATTICE-Frames beginnen mit der Verarbeitung während der Zielerkennung. Ziel- und DAQ-Referenzen werden pro Band festgelegt. Bei einem verschobenen Projekt werden die Dateien neu verknüpft. Die Exporte einer 8-Bit-Aufnahme erfolgen im 8-Bit-Format mit einem neuen**TIFF-Format (8-Bit)**. Ein Survey3-Lauf ohne Kalibrierungsziel wird als dessen Fallback-Produkt (`Vignette_Corrected_Images` oder `Sensor_Response_Images`) gespeichert, nicht als Reflektanz. Die bandweisen DLS-Tags eines Reflektanzbildes entsprechen immer denen des jeweiligen Frames. „Texture Aware“ wird für LATTICE-Aufnahmen abgelehnt, die immer den Standard-Debayer erhalten.
* **CLI & SDK.** Relative Ausgabepfade führen nun in den Ordner, aus dem Sie den Befehl oder das Skript ausführen (ein installiertes Backend löste sie früher in seinem eigenen Installationsordner auf und schlug dabei fehl). Das Backend läuft zwischen Befehlen und Skripten weiter und beendet sich selbst nach 30 Minuten Inaktivität; `chloros-cli backend status` und `backend stop` verwalten dies. Erfassungsbefehle listen alle Dateien auf, die sie nicht schreiben konnten, und beenden sich mit Code 1; wiederholte Erfassungen werden bei vollem Laufwerk beendet. Ein erfolgreicher `process`-Lauf meldet `Image products written: N (plus M clamp masks)`. Unter Windows finden die Befehle `ChlorosLocal` / `process_folder` des SDKs das installierte Backend (und verwenden ein bereits laufendes), und `read_image_audit_tags` funktioniert über ein einfaches `pip install`.

***

## Neuerungen in Chloros 1.2.0

* **Live-Kamerasteuerung – neue Registerkarte „Kameras“.** Schließen Sie LATTICE-Kameras einzeln oder als synchronisierte Mehrkamera-Arrays an (PTP-Zeitsynchronisation, hardwaregesteuerte Aufnahme), mit Live-Vorschau-Overlays, bandweisen Histogrammen, intelligenter automatischer Belichtung, einem Live-Index-Rechner und In-App-Firmware-Updates für die Kameras.
* **Lichtsensoren – neue Registerkarte „Lichtsensoren“.** Schließen Sie DAQ-U- (USB), DAQ-M- (Bluetooth) und DAQ-E- (Ethernet) Sensoren an; kalibrierte Live-Spektren (W/m²/nm) anzeigen, `.daq`-Dateien in Ihrem Projekt aufzeichnen, Cap-Korrekturprofile auswählen und die DAQ-E-Firmware über das Netzwerk aktualisieren.
* **Aufnahmemodi und Rekorder.** Einzel-/Daueraufnahme/Intervallaufnahme sowie ein reiner Rohdaten-Modus („Fastest Capture“); projektbezogene Auswahl, welche Kameras und Exporttypen „Capture All“ erzeugt; Array-Rekorder für Index-Videos in Überwachungsqualität und Rohdaten-Bursts in Analysequalität mit Offline-Video-Builds.
* **LATTICE-Verarbeitungspipeline.** Importieren Sie LATTICE-Aufnahmeordner und wandeln Sie jedes Rohbild in debayerte Bilder, Vorschaubilder, „Float32“-Strahlungsintensitätswerte (W/m²/sr/nm) und Reflexionswerte um – mit Umschaltmöglichkeiten pro Produkt. Die Reflektanz kann von einem Kalibrierungsziel innerhalb des Bildes oder von DAQ-Downwelling stammen; bei Exporten wird eine Array-Ausrichtung angewendet; fehlende Werkskalibrierungen werden automatisch anhand der Kameraseriennummer heruntergeladen.
* **Projekte merken sich die Hardware.** Angeschlossene Kameras und Lichtsensoren werden mit dem Projekt gespeichert (`cameras.json` / `sensors.json`) und stellen beim erneuten Öffnen des Projekts wieder eine Verbindung mit ihren gespeicherten Einstellungen her. Siehe [GUI: Projekte](projects.md).
* **Verbesserungen im Bildbetrachter.** Anzeige von Cursor-Pixeln/Indizes mit korrekter Reflektanzskalierung pro Datei, Ebenenhistogramme, ein GSD-Binning-Schieberegler, Rastermodi „Pro Trigger“ und „Pro Kamera“, LATTICE-Produktansichten sowie Exporte von Indizes/LUTs in die Sandbox auf die Festplatte.
* **CLI und SDK wurden erheblich erweitert.** Neue Befehlsfamilien `lattice`, `daq pool-*`, `project` und `time-sync`; neue `process`-Optionen (`--input-level`, produktbezogene Umschaltoptionen, `--reflectance-source`, Flags zur Array-Ausrichtung); SDK-Smart-Connect-Handles (`connect_camera` / `connect_array` / `connect_daq_sensor`), die das Backend automatisch starten; `open_project()`-Automatisierung; das SDK-Wheel ist in den Installationsprogrammen enthalten und wird auf PyPI als `chloros-sdk` veröffentlicht.
* **Ehrliche Fehlersemantik.** Ein `chloros-cli process`-Lauf, der Produkte angefordert, aber keine geschrieben hat, schlägt nun deutlich fehl und beendet sich mit einem Ergebnis ungleich Null; erfolgreiche Läufe melden, wie viele Image-Produkte sie geschrieben haben.
* **Neues Ausgabe-Layout.** Produkte werden in `<project>/<camera>/<format>/<Product>_Images/`-Ordnern abgelegt und behalten den Quelldateinamen bei – der Ordner, nicht ein Dateinamenssuffix, identifiziert das Produkt. Siehe [Ausgabe-Bildformate](output-image-formats.md).
* **Mehr Eingaben, Pläne und Sprachen.** Unterstützung für `.dng`-Eingaben; alle 38 Schnittstellensprachen vollständig verfügbar; planbezogene Hardware-Beschränkungen bei kostenloser (ohne Anmeldung) Nutzung von bis zu 4 Kameras und 2 Lichtsensoren.
* **Zuverlässigkeit.** „Stop Processing“ wird sauber mit einer aussagekräftigen Ausführungszusammenfassung beendet, bei Projekten mit mehreren Kameras werden alle Kameras exportiert, und bei Upgrades über das Installationsprogramm werden Sie nicht mehr abgemeldet.***

Chloros ist in drei Anwendungsumgebungen verfügbar:

## Chloros: Desktop-GUI-Anwendung

Eigenständiges separates Fenster mit allen Funktionen, einschließlich der Registerkarten „Live-Kameras“ und „Lichtsensoren“. _Nur Windows._

## [Chloros CLI: Befehlszeilenschnittstelle](CLI.md)

Batch-Verarbeitung über die Befehlszeile sowie Live-Befehle `lattice`, `daq pool-*`, `project` und `time-sync`. Ideal für Automatisierung, Skripterstellung und den Headless-Betrieb. Verfügbar unter **Windows, Linux amd64 und Linux arm64 (NVIDIA Jetson)**. _Für den Zugriff auf die CLI ist ein kostenpflichtiges Chloros+-Abo erforderlich._

## [Chloros-API: Python-SDK](api-python-sdk.md)

Programmatische Python-Schnittstelle für Automatisierung und benutzerdefinierte Workflows: vollständige Pipeline-Verarbeitung, Live-Kamera-/Array-Sitzungen, DAQ-Sensor-Sitzungen und Automatisierung gespeicherter Projekte. Wird mit dem Desktop-/CLI-Paket installiert und ist außerdem unter der Bezeichnung `pip install chloros-sdk` veröffentlicht. _Für den Zugriff auf die API ist ein kostenpflichtiges Chloros+-Abo erforderlich._

***

## Unterstützte Plattformen

| Plattform | GUI | CLI | Python-SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nein | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nein | Ja | Ja |

Anweisungen zur Installation unter Linux finden Sie im Abschnitt [Linux & Edge Computing](linux/linux-overview.md).

***

## Erste Schritte in drei Schritten

1. **Installieren** – Laden Sie das Installationsprogramm für Ihre Plattform herunter und führen Sie es aus. Siehe [Download](download.md).
2. **Anmelden (optional für die GUI)** – Die GUI verarbeitet Bilder kostenlos und ohne Konto. Eine [Chloros+-Anmeldung](chloros+-login.md) schaltet Parallelverarbeitung, GPU-Beschleunigung, höhere Gerätebeschränkungen sowie CLI- und SDK-Zugriff frei.
3. **Erstellen Sie Ihr erstes Projekt** — öffnen Sie Chloros, erstellen Sie ein [neues Projekt](projects.md), [fügen Sie Ihre Bilder hinzu](processing-images-gui/adding-files-to-a-project.md) und [die Verarbeitung starten](processing-images-gui/starting-the-processing.md). Um stattdessen Live-Hardware anzusteuern, öffnen Sie die Registerkarte „Kameras“ oder „Lichtsensoren“ – siehe [GUI: Navigation](navigation.md).

***

## Chloros+

Chloros ist für die meisten Aufgaben zwar kostenlos nutzbar, aber vielleicht möchten Sie mehr. In diesem Fall kann eine kostenpflichtige Lizenz für Chloros+ von Vorteil sein. Mit einer Chloros+-Lizenz können Sie neue Funktionen freischalten, wie zum Beispiel:

* **Multithread-Verarbeitung**: Beschleunigen Sie die Bildverarbeitung bei größeren Projekten erheblich, indem Bilder gleichzeitig durch die Pipeline verarbeitet werden.
* **GPU-Beschleunigung (CUDA)**: Nutzen Sie die Vorteile der heutigen GPUs mit größerem Speicher, um die Bildverarbeitungspipeline weiter zu beschleunigen. Für optimale Ergebnisse empfehlen wir 4 GB oder mehr VRAM.
* **Chloros+**[**CLI**](CLI.md) **Zugriff**: Führen Sie Chloros+ über die Befehlszeile aus, um die Verarbeitung zu automatisieren und in Ihre eigene Software zu integrieren. Verfügbar in allen kostenpflichtigen Tarifen; serverseitig erzwungen.
* **Chloros+**[**API**](api-python-sdk.md) **Zugriff:** Führen Sie Chloros+ über Python aus, um eine programmatische Steuerung zu ermöglichen und eine nahtlose Integration in Ihre Forschungspipelines, Datenanalyse-Workflows und benutzerdefinierten Anwendungen zu gewährleisten. Verfügbar in allen kostenpflichtigen Tarifen; serverseitig durchgesetzt.
* **Höhere Hardware-Limits**: Schließen Sie mehr Kameras und Lichtsensoren gleichzeitig an. Ohne Anmeldung verbindet die GUI bis zu 4 Kameras und 2 DAQ-Lichtsensoren; kostenpflichtige Tarife erhöhen beide Obergrenzen:

| Tarif | Kameras | DAQ-Lichtsensoren |
| --- | --- | --- |
| Iron (kostenlos, ohne Anmeldung) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Nutzung auf mehreren Geräten**: Mit jeder Chloros+-Lizenz können 2 oder mehr Geräte registriert werden. Verwenden Sie Ihr MAPIR-Cloud-Konto, um registrierte Geräte zu verwalten. Erweitern Sie die Unterstützung für weitere Geräte durch ein Upgrade Ihrer Chloros+-Lizenz.
* **Erweiterte texturbasierte Debayering-Methode:** Ein hochwertiges, kantenbewusstes Debayering in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt.
* **Benutzerdefinierte multispektrale Indexformeln:** Geben Sie benutzerdefinierte multispektrale Indizes in die Chloros-Rasterrechner ein, sowohl für die Verarbeitung als auch für die Bildbetrachtungs-Sandbox.
* **Linux & Edge-Computing:** Führen Sie Chloros auf Linux-x86_64- und ARM64-Plattformen aus, einschließlich NVIDIA Jetson, für die Feld- und Edge-Verarbeitung. Siehe [Linux-Übersicht](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Preise und Anmeldung für Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
