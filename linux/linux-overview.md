# Linux-Übersicht

Chloros 1.3.0 bietet native Linux-Unterstützung für die **CLI**und das**Python-SDK** – headless Multispektralbildverarbeitung sowie Live-Steuerung von LATTICE-Kameras und DAQ-Lichtsensoren – auf Linux-Workstations, Servern und NVIDIA Jetson-Edge-Geräten.

{% hint style="info" %}
**Keine Desktop-GUI unter Linux.**Die Chloros-Desktop-GUI ist nur für Windows verfügbar. Linux-Benutzer interagieren mit Chloros über die [CLI](../CLI.md) und das [Python-SDK](../api-python-sdk.md). Das `.deb` fügt zwar einen Eintrag**„Chloros CLI“** zu Ihrem Anwendungsmenü hinzu – es öffnet jedoch lediglich einen Terminalemulator, auf dem `chloros-cli` ausgeführt wird.
{% endhint %}

***

## Plattform-Unterstützungsmatrix

| Funktion | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/ --- | --- | --- | --- |
| **Desktop-GUI** | Ja | k. A. | Nein | Nein |
| **CLI** (`chloros-cli`) | Ja | Ja | Ja | Ja |
| **Python-SDK** (`chloros-sdk`) | Ja | Ja | Ja | Ja |
| **Bildverarbeitungs-Pipeline** | Ja | Ja | Ja | Ja |
| **LATTICE-Kamerasteuerung (live)** | Ja (Registerkarte „Kameras“) | Ja (`chloros-cli lattice`, SDK) | Ja | Ja |
| **DAQ-Lichtsensoren (live)** | Ja (Registerkarte „Lichtsensoren“) | Ja (`chloros-cli daq pool-*`, SDK) | Ja | Ja |
| **PTP-Zeitsynchronisation (Host ist Grandmaster)** | Ja | Ja (`chloros-cli time-sync`) | Ja | Ja |
| **GPU-Beschleunigung (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Textur-bewusster Debayer** (erfordert eine CUDA-GPU) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamische Rechenanpassung** | Ja | Ja | Ja | Ja |
| **Backend als Systemdienst** (`chloros-backend@<user>.service`) | Nein | Nein | Ja (Opt-in) | Ja (Opt-in) |
| **In-Place-Updater** (`chloros-cli update`) | Nein (Installationsprogramm ausführen) | Nein (Installationsprogramm ausführen) | Ja | Ja |***

## Unterstützte Architekturen

| Architektur | Beschreibung | Paket |
| --- | --- | --- |
| **amd64 (x86_64)** | Standard-Desktop-/Server-Prozessoren (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-Prozessoren – NVIDIA Jetson Orin-Familie (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Unterstützte Linux-Distributionen

* **Ubuntu 22.04 LTS oder neuer** (amd64)
* **Debian 12 oder neuer** (amd64)
* **NVIDIA JetPack 6** (arm64 – Jetson Orin-Plattformen)***

## Was Linux-Benutzer erhalten

* **Chloros CLI** – die vollständige Befehlszeilenschnittstelle für die Stapelverarbeitung, Automatisierung und Skripterstellung
* **Chloros Python SDK** – programmatische Python-Schnittstelle für Forschungspipelines und benutzerdefinierte Tools, installiert durch das `.deb` für das System `python3` (ein versionskompatibles Wheel für virtualenvs ist in `/usr/lib/chloros/sdk/` enthalten; PyPI ist für Rechner ohne Chloros-Paket vorgesehen)
* **LATTICE-Kamerasteuerung** – Erkennen, Verbinden, Konfigurieren und Aufnehmen mit LATTICE-Kameras und synchronisierten Mehrkamera-Arrays über `chloros-cli lattice` und das SDK; `.deb` enthält die von den Kameras benötigte Arena-SDK-Laufzeitumgebung
* **Steuerung von DAQ-Lichtsensoren** – Schließen Sie DAQ-U/M/E-Sensoren an, streamen Sie kalibrierte Spektren und zeichnen Sie `.daq`-Dateien über `chloros-cli daq pool-*` und das SDK auf
* **PTP-Zeitsynchronisation** – Das Chloros-Backend betreibt den PTP-Grandmaster, an den sich LATTICE-Kameras und DAQ-E-Sensoren als Slaves anschließen; überprüfen Sie ihn mit `chloros-cli time-sync` und halten Sie ihn mit der systemd-Einheit `chloros-backend@<user>.service` im Headless-Modus am Laufen (siehe [Linux-Installation](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projektautomatisierung** — Führen Sie gespeicherte Projekte im Headless-Modus mit `chloros-cli project` und dem `open_project` des SDK aus
* **GPU-Beschleunigung** – CUDA-beschleunigte Verarbeitung auf NVIDIA-GPUs (Desktop-GPUs auf amd64 und die GPU des Jetson mit JetPack 6)
* **Dynamische Rechenanpassung** — automatische Hardwareerkennung und Auswahl der Verarbeitungsstrategie, mit der `CHLOROS_STRATEGY`-Überschreibung als Ausweichmöglichkeit für Experten
* **Alle Verarbeitungsfunktionen** — dieselbe Pipeline wie unter Windows: Kalibrierung, Vignettierungskorrektur, Vegetationsindizes und alle Exportformate
* **Chloros+-Funktionen** – Multithread-Verarbeitung (Pipeline), „Texture Aware“-Debayer (auf einer CUDA-GPU) und benutzerdefinierte Indizes mit einem kostenpflichtigen Chloros+-Abonnement

## Was Linux-Nutzer nicht erhalten

* **Desktop-GUI** — keine grafische Benutzeroberfläche; die gesamte Interaktion erfolgt über die CLI oder das Python-SDK
* **Bildbetrachter** — kein interaktiver Bildbetrachter, keine Rasteransicht und keine Kartenmarkierungen
* **Visuelles Projektmanagement** — Projekte werden über CLI-Befehle und SDK-Aufrufe erstellt und gesteuert (die Hardware selbst – Kameras, Sensoren, Erfassung – bleibt vom Terminal aus vollständig steuerbar)***

## Lizenzanforderungen

Der Zugriff auf CLI und SDK erfordert eine **kostenpflichtige Chloros+-Stufe – Copper oder höher**(Copper, Bronze, Silver, Gold). Die kostenlose**Iron**-Stufe bietet keinen CLI-/SDK-Zugriff. Diese Beschränkung wird vom Backend durchgesetzt, nicht nur von der CLI:

| Situation | Backend-Antwort |
| --- | --- |
| Nicht angemeldet | `401` mit `error_code: AUTH_REQUIRED` |
| Angemeldet auf der kostenlosen „Iron“-Stufe | `403` mit `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funktioniert auf jeder Stufe – es ist die einzige Route, die von der Sperre ausgenommen ist –, sodass der Grund für eine Ablehnung immer sichtbar ist.

***

## Erste Schritte unter Linux

1. **Chloros installieren** – siehe [Linux-Installation](linux-installation.md) für die Installation von `.deb`
2. **Überprüfen** – `chloros-cli --version` gibt `Chloros CLI 1.3.0` aus; `chloros-cli selftest` führt die 7-stufige Diagnose durch
3. **Python-SDK prüfen** — `.deb` installiert es: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (in einer virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Anmelden** — `chloros-cli login your@email.com 'your-password'` (einmal pro Rechner; bei Paket-Upgrades bleibt die Anmeldung erhalten, beim Entfernen des Pakets wird sie gelöscht)
5. **Verarbeiten Sie Ihren ersten Datensatz** — `chloros-cli process ~/datasets/flight001`

Für NVIDIA Jetson finden Sie im speziellen [NVIDIA Jetson-Leitfaden](nvidia-jetson-guide.md) Informationen zur plattformspezifischen Einrichtung, zum thermischen Verhalten und zum Einsatz vor Ort.

***

## Nächste Schritte

* [Linux-Installation](linux-installation.md) – detaillierte Installation, Dateipfade und Fehlerbehebung für amd64 und arm64
* [NVIDIA Jetson-Handbuch](nvidia-jetson-guide.md) – Jetson-spezifische Einrichtung, Speicher- und thermisches Verhalten, Einsatz vor Ort
* [CLI: Befehlszeile](../CLI.md) – das CLI-Handbuch
* [API: Python-SDK](../api-python-sdk.md) – das SDK-Handbuch
* [CLI-Referenz](../reference/cli-reference.md) und [SDK-Referenz](../reference/sdk-reference.md) – umfassende Befehls- und API-Listen für Version 1.3.0
* [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md) – Wie sich Chlorosan Ihre Hardware anpasst

{% hint style="info" %}
**Programmgesteuertes Auslesen dieses Handbuchs.** Jede Seite wird außerdem als reines Markdown unter einer eigenen URL plus `.md` (zum Beispiel `https://mapir.gitbook.io/chloros/linux/linux-installation.md`) bereitgestellt, und ein Index des gesamten Handbuchs ist unter [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
