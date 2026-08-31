# Linux – Übersicht

Chloros 1.2.0 bietet native Linux-Unterstützung für **CLI**und**Python SDK** — die Verarbeitung von multispektralen Bildern ohne Bildschirm sowie die Live-Steuerung von LATTICE-Kameras und DAQ-Lichtsensoren — auf Linux-Workstations, Servern und NVIDIA Jetson-Edge-Geräten.

{% hint style="info" %}
**Keine Desktop-GUI auf Linux.**Die Chloros-Desktop-GUI ist ausschließlich für Windows verfügbar. Linux-Benutzer interagieren mit Chloros über [CLI](../CLI.md) und [Python SDK](../api-python-sdk.md). Das `.deb` fügt zwar einen Eintrag**Chloros CLI**zum Menü Ihrer Anwendung hinzu – es öffnet lediglich einen Terminalemulator, auf dem**`chloros-cli`** läuft.
{% endhint %}

***

## Plattform-Unterstützungsmatrix

| Funktion | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktop-GUI** | Ja | k. A. | Nein | Nein |
| **CLI** (`chloros-cli`) | Ja | Ja | Ja | Ja |
| **Python SDK** (`chloros-sdk`) | Ja | Ja | Ja | Ja |
| **Bildverarbeitungs-Pipeline** | Ja | Ja | Ja | Ja |
| **LATTICE-Kamerasteuerung (Live)** | Ja (Registerkarte „Kameras“) | Ja (`chloros-cli lattice`, SDK) | Ja | Ja |
| **DAQ-Lichtsensoren (Live)** | Ja (Registerkarte „Lichtsensoren“) | Ja (`chloros-cli daq pool-*`, SDK) | Ja | Ja |
| **PTP-Zeitsynchronisation (Host ist Grandmaster)** | Ja | Ja (`chloros-cli time-sync`) | Ja | Ja |
| **GPU-Beschleunigung (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Textur-bewusster Debayer** | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamische Rechenanpassung** | Ja | Ja | Ja | Ja |
| **Backend als Systemdienst** (`chloros-backend.service`) | Nein | Nein | Ja (Opt-in) | Ja (Opt-in) |
| **In-Place-Updater** (`chloros-cli update`) | Nein (Installationsprogramm ausführen) | Nein (Installationsprogramm ausführen) | Ja | Ja |***

## Unterstützte Architekturen

| Architektur | Beschreibung | Paket |
| --- | --- | --- |
| **amd64 (x86_64)** | Standard-Desktop-/Server-Prozessoren (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-Prozessoren – NVIDIA Jetson Orin-Familie | `chloros_<version>_arm64_jp6.deb` (JetPack 6-Build) |

## Unterstützte Linux-Distributionen

* **Ubuntu 22.04 LTS oder neuer** (amd64)
* **Debian 12 oder neuer** (amd64)
* **NVIDIA JetPack 6** (arm64 – Jetson Orin-Plattformen)***

## Was Linux-Benutzer erhalten

* **Chloros CLI** – die vollständige Befehlszeilenschnittstelle für Stapelverarbeitung, Automatisierung und Skripterstellung
* **Chloros Python SDK** — programmatische Python-Schnittstelle für Forschungspipelines und benutzerdefinierte Tools (installierbar über PyPI und zudem als versionsangepasstes Wheel im `.deb` enthalten)
* **LATTICE-Kamerasteuerung** — Erkennen, Verbinden, Konfigurieren und Aufnehmen mit LATTICE-Kameras und synchronisierten Mehrkamera-Arrays über `chloros-cli lattice` und SDK; das `.deb` enthält die von den Kameras benötigte Arena-Laufzeitumgebung SDK
* **DAQ-Lichtsensorsteuerung** — Schließen Sie DAQ-U/M/E-Sensoren an, streamen Sie kalibrierte Spektren und zeichnen Sie `.daq`-Dateien über `chloros-cli daq pool-*` und das SDK auf
* **PTP-Zeitsynchronisation** — Das Chloros-Backend betreibt den PTP-Grandmaster, an den sich LATTICE-Kameras und DAQ-E-Sensoren als Slaves anschließen; überprüfen Sie dies mit `chloros-cli time-sync`, und lassen Sie es mit der systemd-Einheit „`chloros-backend.service`“ im Headless-Modus laufen (siehe [Linux-Installation](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projektautomatisierung** — Führen Sie gespeicherte Projekte im Headless-Modus mit `chloros-cli project` und der `open_project`-Funktion des SDK aus
* **GPU-Beschleunigung** — CUDA-beschleunigte Verarbeitung auf NVIDIA-GPUs (Desktop und Jetson)
* **Dynamische Rechenanpassung** — automatische Hardwareerkennung und Auswahl der Verarbeitungsstrategie, mit der `CHLOROS_STRATEGY`-Überschreibung als Experten-Ausweichmöglichkeit
* **Alle Verarbeitungsfunktionen** — dieselbe Pipeline wie bei Windows: Kalibrierung, Vignettierungskorrektur, Vegetationsindizes und alle Exportformate
* **Chloros+-Funktionen** – Multithread-Verarbeitung (Pipeline), texturbasierter Debayer und benutzerdefinierte Indizes, mit einem kostenpflichtigen Chloros+-Abo

## Was Nutzer von Linux nicht erhalten

* **Desktop-GUI** — keine grafische Benutzeroberfläche; die gesamte Interaktion erfolgt über CLI oder Python SDK
* **Bildbetrachter** – kein interaktiver Bildbetrachter, keine Rasteransicht und keine Kartenmarkierungen
* **Visuelles Projektmanagement** – Projekte werden über CLI-Befehle und SDK-Aufrufe erstellt und gesteuert (die Hardware selbst – Kameras, Sensoren, Erfassung – bleibt vom Terminal aus vollständig steuerbar)***

## Lizenzanforderungen

Der Zugriff auf CLI und SDK erfordert eine **kostenpflichtige Chloros+-Stufe – Copper oder höher**(Copper, Bronze, Silver, Gold). Die kostenlose**Iron**-Stufe bietet keinen Zugriff auf CLI/SDK. Diese Mindestanforderung wird vom Backend durchgesetzt, nicht nur durch den CLI:

| Situation | Antwort des Backends |
| --- | --- |
| Nicht angemeldet | `401` mit `error_code: AUTH_REQUIRED` |
| Auf der kostenlosen Iron-Stufe angemeldet | `403` mit `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funktioniert auf jeder Stufe – es ist die einzige Route, die von der Sperre ausgenommen ist –, sodass der Grund für eine Ablehnung immer sichtbar ist.

***

## Erste Schritte mit Linux

1. **Installieren Sie Chloros** – siehe [Installation von Linux](linux-installation.md) für die Installation von `.deb`
2. **Überprüfen** — `chloros-cli --version` druckt `Chloros CLI 1.2.0` aus; `chloros-cli selftest` führt die 7-stufige Diagnose durch
3. **Installieren Sie Python und SDK** (optional) — `pip install chloros-sdk`
4. **Anmelden** – `chloros-cli login your@email.com 'your-password'` (einmal pro Gerät und erneut nach jedem Paket-Upgrade)
5. **Ersten Datensatz verarbeiten** – `chloros-cli process ~/datasets/flight001`

Für NVIDIA Jetson finden Sie im speziellen [NVIDIA Jetson-Handbuch](nvidia-jetson-guide.md) Informationen zur plattformspezifischen Einrichtung, zum thermischen Verhalten und zum Einsatz vor Ort.

***

## Nächste Schritte

* [Linux Installation](linux-installation.md) – detaillierte Installationsanleitung, Dateipfade und Fehlerbehebung für amd64 und arm64
* [NVIDIA Jetson-Handbuch](nvidia-jetson-guide.md) – Jetson-spezifische Einrichtung, Speicher- und thermisches Verhalten, Einsatz vor Ort
* [CLI: Befehlszeile](../CLI.md) – das CLI-Handbuch
* [API: Python SDK](../api-python-sdk.md) – das SDK-Handbuch
* [CLI-Referenz](../reference/cli-reference.md) und [SDK-Referenz](../reference/sdk-reference.md) – vollständige Befehls-/API-Listen für 1.2.0
* [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md) – wie sich Chloros an Ihre Hardware anpasst

{% hint style="info" %}
**Programmgesteuertes Auslesen dieses Handbuchs.** Jede Seite wird außerdem als reines Markdown unter ihrer eigenen URL sowie `.md` (zum Beispiel `https://mapir.gitbook.io/chloros/linux/linux-installation.md`) bereitgestellt, und ein Index des gesamten Handbuchs ist unter [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt) veröffentlicht.
{% endhint %}
