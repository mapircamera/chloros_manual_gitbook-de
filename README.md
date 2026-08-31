---
metaLinks: {}
---

# Erste Schritte

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros ist eine Softwareanwendung von [MAPIR](https://www.mapir.camera) zur Verarbeitung multispektraler Bilder, zur Live-Steuerung von MAPIR-Hardware und zur Aufzeichnung von Sensordaten. Chloros 1.2.0 unterstützt die gesamte MAPIR-Produktfamilie:

* **Survey3-Kameras** – Verarbeitung von RAW+JPG-Aufnahmen zu kalibrierten Reflexions- und Vegetationsindexkarten. Siehe [Unterstützte Kameras](supported-cameras.md).
* **LATTICE-Kameras** – Schließen Sie die GigE-Multispektralkameramodule live an, einzeln oder als synchronisierte Mehrkamera-Arrays: Vorschau, Aufnahme und Verarbeitung zu kalibrierten Strahlungs- und Reflexionsprodukten. Siehe den [Abschnitt zu LATTICE](lattice/README.md).
* **DAQ-Lichtsensoren** — DAQ-U (USB), DAQ-M (Bluetooth) und DAQ-E (Ethernet) Spektralsensoren: kalibrierte Live-Spektren, `.daq`-Aufzeichnungen und nach unten gerichtete Beleuchtung für die Reflektanzverarbeitung. Siehe den [DAQ-Abschnitt](daq/README.md).

{% hint style="success" %}
**Neuerungen in Chloros 1.2.0**: Live-Steuerung von LATTICE-Kameras und -Arrays, Integration von DAQ-Lichtsensoren, Aufnahmemodi und Rekorder, eine vollständige radiometrische Verarbeitungspipeline für LATTICE, Projektautomatisierung über CLI/SDK und vieles mehr. Siehe die Liste der Neuerungen unten und [Download](download.md) für das Änderungsprotokoll.
{% endhint %}

{% hint style="info" %}
**Verwenden Sie Chloros mit einem KI-Assistenten?** Dieses Handbuch ist genau dafür gedacht. Weisen Sie Ihren Assistenten auf Folgendes hin:

* `https://mapir.gitbook.io/chloros/llms.txt` – maschinenlesbarer Index aller Seiten.
* Jede Seite als reines Markdown – fügen Sie „`.md`“ an deren „URL“ an (z. B. „`https://mapir.gitbook.io/chloros/reference/cli-reference.md`“).
* Die [CLI-Referenz](reference/cli-reference.md) und [SDK-Referenz](reference/sdk-reference.md) – vollständige Referenzseiten mit exakten Werten, die für die Verwendung durch LLMs verfasst wurden.

Beispiel-Prompt: *„Lies https://mapir.gitbook.io/chloros/reference/cli-reference.md, und schreibe dann ein Skript, das sich anmeldet und den Ordner ~/flights/flight_001 in GeoTIFFs im Format „Reflectance + NDVI“ verarbeitet.“*

Vollständige Anleitung: [Verwendung von Chloros mit KI-Assistenten](ai-assistants.md).
{% endhint %}

***

## Neuheiten in Chloros 1.2.0

* **Live-Kamerasteuerung – neuer Reiter „Kameras“.** Schließen Sie LATTICE-Kameras einzeln oder als synchronisierte Mehrkamera-Arrays an (PTP-Zeitsynchronisation, hardwaregesteuerte Aufnahme), mit Live-Vorschau-Overlays, bandweisen Histogrammen, intelligenter automatischer Belichtung, einem Live-Index-Rechner und In-App-Firmware-Updates für die Kameras.
* **Lichtsensoren – neuer Reiter „Lichtsensoren“.** Schließen Sie DAQ-U- (USB), DAQ-M- (Bluetooth) und DAQ-E- (Ethernet) Sensoren an; kalibrierte Live-Spektren (W/m²/nm) anzeigen, `.daq`-Dateien in Ihrem Projekt aufzeichnen, Cap-Korrekturprofile auswählen und die DAQ-E-Firmware über das Netzwerk aktualisieren.
* **Aufnahmemodi und Rekorder.** Einzel-/Daueraufnahme/Intervallaufnahme sowie ein reiner Rohdaten-Modus („Fastest Capture“); projektbezogene Auswahl, welche Kameras und Exporttypen „Capture All“ erzeugt; Array-Rekorder für Index-Videos in Überwachungsqualität und Rohdaten-Bursts in Analysequalität mit Offline-Video-Builds.
* **LATTICE-Verarbeitungspipeline.** Importieren Sie LATTICE-Aufnahmeordner und wandeln Sie jedes Rohbild in debayerte Bilder, Vorschaubilder, „Float32“-Radiance-Werte (W/m²/sr/nm) und Reflektanzwerte um – mit Umschaltmöglichkeiten pro Produkt. Die Reflektanz kann von einem Kalibrierungsziel innerhalb des Bildes oder von DAQ-Downwelling stammen; bei Exporten wird eine Array-Ausrichtung angewendet; fehlende Werkskalibrierungen werden automatisch anhand der Kameraseriennummer heruntergeladen.
* **Projekte merken sich die Hardware.** Angeschlossene Kameras und Lichtsensoren werden mit dem Projekt gespeichert (`cameras.json` / `sensors.json`) und stellen beim erneuten Öffnen des Projekts wieder eine Verbindung mit ihren gespeicherten Einstellungen her. Siehe [GUI: Projekte](projects.md).
* **Verbesserungen im Bildbetrachter.** Anzeige von Cursor-Pixeln/Indizes mit korrekter Reflektanzskalierung pro Datei, Ebenenhistogramme, ein GSD-Binning-Schieberegler, Rastermodi „Pro Trigger“ und „Pro Kamera“, LATTICE-Produktansichten sowie Exporte von Indizes/LUTs in die Sandbox auf die Festplatte.
* **CLI &amp; SDK, erheblich erweitert.** Neue Befehlsfamilien `lattice`, `daq pool-*`, `project` und `time-sync`; neue `process`-Optionen (`--input-level`, produktbezogene Umschaltoptionen, `--reflectance-source`, Flags zur Array-Ausrichtung); SDK-Smart-Connect-Handles (`connect_camera` / `connect_array` / `connect_daq_sensor`), die das Backend automatisch starten; `open_project()`-Automatisierung; das SDK-Wheel ist in den Installationsprogrammen enthalten und wird auf PyPI als `chloros-sdk` veröffentlicht.
* **Ehrliche Fehlersemantik.** Ein `chloros-cli process`-Lauf, der Produkte angefordert, aber keine geschrieben hat, schlägt nun deutlich fehl und beendet sich mit einem Ergebnis ungleich Null; erfolgreiche Läufe melden, wie viele Bildprodukte sie geschrieben haben.
* **Neues Ausgabe-Layout.** Die Produkte werden in `<project>/<camera>/<format>/<Product>_Images/`-Ordnern abgelegt und behalten den Quelldateinamen bei – der Ordner, nicht ein Dateinamenssuffix, identifiziert das Produkt. Siehe [Ausgabe-Bildformate](output-image-formats.md).
* **Mehr Eingaben, Pläne und Sprachen.** Unterstützung für `.dng`-Eingaben; alle 38 Schnittstellensprachen vollständig implementiert; planbezogene Hardware-Beschränkungen bei kostenloser (ohne Anmeldung) Nutzung von bis zu 4 Kameras und 2 Lichtsensoren.
* **Zuverlässigkeit.** „Verarbeitung beenden“ wird sauber mit einer aussagekräftigen Ausführungszusammenfassung beendet, bei Projekten mit mehreren Kameras werden alle Kameras exportiert, und bei Upgrades des Installationsprogramms werden Sie nicht mehr abgemeldet.***

Chloros ist in 3 Anwendungsumgebungen verfügbar:

## Chloros: Desktop-GUI-Anwendung

Eigenständiges separates Fenster mit allen Funktionen, einschließlich der Registerkarten „Live-Kameras“ und „Lichtsensoren“. _Nur für Windows._

## [Chloros CLI: Befehlszeilenschnittstelle](CLI.md)

Batch-Verarbeitung über die Befehlszeile sowie Live-Befehle `lattice`, `daq pool-*`, `project` und `time-sync`. Ideal für Automatisierung, Skripterstellung und den Headless-Betrieb. Verfügbar auf **Windows, Linux amd64 und Linux arm64 (NVIDIA Jetson)**. _Für den Zugriff auf die CLI ist eine kostenpflichtige Chloros+-Stufe erforderlich._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmatische Python-Schnittstelle für Automatisierung und benutzerdefinierte Workflows: vollständige Pipeline-Verarbeitung, Live-Kamera-/Array-Sitzungen, DAQ-Sensorsitzungen und Automatisierung gespeicherter Projekte. Wird mit dem Desktop-/CLI-Paket installiert und zusätzlich als `pip install chloros-sdk` veröffentlicht. _Für den Zugriff auf die API ist ein kostenpflichtiger Chloros+-Tarif erforderlich._

***

## Unterstützte Plattformen

| Plattform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nein | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nein | Ja | Ja |

Anweisungen zur Installation von Linux finden Sie im Abschnitt [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Erste Schritte in drei Schritten

1. **Installieren** – Laden Sie das Installationsprogramm für Ihre Plattform herunter und führen Sie es aus. Siehe [Download](download.md).
2. **Anmelden (optional für die GUI)** – Die GUI verarbeitet Bilder kostenlos auch ohne Konto. Eine [Chloros+-Anmeldung](chloros+-login.md) schaltet parallele Verarbeitung, GPU-Beschleunigung, höhere Gerätebeschränkungen sowie den Zugriff auf CLI/SDK.
3. **Erstellen Sie Ihr erstes Projekt** – öffnen Sie Chloros, erstellen Sie ein [Neues Projekt](projects.md), [fügen Sie Ihre Bilder hinzu](processing-images-gui/adding-files-to-a-project.md) und [starten Sie die Verarbeitung](processing-images-gui/starting-the-processing.md). Um stattdessen Live-Hardware anzusteuern, öffnen Sie die Registerkarte „Kameras“ oder „Lichtsensoren“ – siehe [GUI: Navigation](navigation.md).***

## Chloros+

Zwar ist Chloros für die meisten Aufgaben kostenlos nutzbar, doch möglicherweise wünschen Sie sich mehr Funktionen. In diesem Fall kann eine kostenpflichtige Lizenz für Chloros+ von Vorteil sein. Mit einer Chloros+-Lizenz können Sie neue Funktionen freischalten, wie zum Beispiel:

* **Multithread-Verarbeitung**: Beschleunigen Sie die Bildverarbeitung bei größeren Projekten erheblich, indem Bilder gleichzeitig durch die Pipeline verarbeitet werden.
* **GPU-Beschleunigung (CUDA)**: Nutzen Sie die Vorteile der heutigen GPUs mit höherer Speicherkapazität, um die Bildverarbeitungspipeline weiter zu beschleunigen. Für optimale Ergebnisse empfehlen wir 4 GB oder mehr VRAM.
* **Chloros+**[**CLI**](CLI.md)**Zugriff**: Führen Sie Chloros+ über die Befehlszeile aus, um die Verarbeitung zu automatisieren und in Ihre eigene Software zu integrieren. Verfügbar in allen kostenpflichtigen Tarifen; serverseitig erzwungen.
* **Chloros+**[**API**](api-python-sdk.md)**Zugriff:** Führen Sie „Chloros+“ über „Python“ aus, um eine programmatische Steuerung zu ermöglichen und eine nahtlose Integration in Ihre Forschungspipelines, Datenanalyse-Workflows und benutzerdefinierten Anwendungen zu gewährleisten. Verfügbar in allen kostenpflichtigen Tarifen; serverseitig durchgesetzt.
* **Höhere Hardware-Grenzen**: Schließen Sie mehr Kameras und Lichtsensoren gleichzeitig an. Ohne Anmeldung verbindet die GUI bis zu 4 Kameras und 2 DAQ-Lichtsensoren; kostenpflichtige Tarife erhöhen beide Obergrenzen:

| Tarif | Kameras | DAQ-Lichtsensoren |
| --- | --- | --- |
| Iron (kostenlos, ohne Anmeldung) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Nutzung auf mehreren Geräten**: Mit jeder Chloros+-Lizenz können 2 oder mehr Geräte registriert werden. Verwenden Sie Ihr MAPIR-Cloud-Konto, um registrierte Geräte zu verwalten. Erweitern Sie die Unterstützung auf weitere Geräte, indem Sie Ihre Chloros+-Lizenz upgraden.
* **Erweiterte texturbasierte Debayering-Methode:** Ein hochwertiges, kantenbewusstes Debayering in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt.
* **Benutzerdefinierte multispektrale Indexformeln:** Geben Sie benutzerdefinierte multispektrale Indizes in die Chloros-Rasterrechner ein, sowohl für die Verarbeitung als auch für die Bildbetrachtungs-Sandbox.
* **Linux &amp; Edge-Computing:** Führen Sie Chloros auf Linux x86_64- und ARM64-Plattformen aus, einschließlich NVIDIA Jetson, für die Feld- und Edge-Verarbeitung. Siehe [Linux-Übersicht](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ – Preise und Anmeldung</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
