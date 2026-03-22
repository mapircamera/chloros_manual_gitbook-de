# Linux – Überblick

Chloros 1.1.0 bietet native Linux-Unterstützung für **CLI**und**Python SDK** und ermöglicht so die headless Multispektralbildverarbeitung auf Linux-Workstations, Servern und NVIDIA Jetson-Edge-Geräten.

{% hint style="info" %}
**Keine GUI auf Linux.** Die Chloros-Desktop-GUI ist nur auf Windows verfügbar. Linux-Benutzer interagieren mit Chloros über [CLI](../CLI.md) und [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Plattform-Unterstützungsmatrix

| Funktion | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktop-GUI** | Ja | N/A | Nein | Nein |
| **CLI** | Ja | Ja | Ja | Ja |
| **Python SDK** | Ja | Ja | Ja | Ja |
| **GPU-Beschleunigung (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Textur-bewusster Debayer** | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamische Rechenanpassung** | Ja | Ja | Ja | Ja |***

## Unterstützte Architekturen

| Architektur | Beschreibung | Installationsmethode |
| --- | --- | --- |
| **amd64 (x86_64)** | Standard-Desktop-/Server-Prozessoren (Intel, AMD) | `.deb`-Paket |
| **arm64 (aarch64)** | ARM-basierte Prozessoren, vor allem NVIDIA Jetson | `.deb`-Paket (JetPack 6) |

## Unterstützte Linux-Distributionen

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson-Plattformen)***

## Was Linux-Benutzer erhalten

* **Chloros CLI** — Vollständige Befehlszeilenschnittstelle für Stapelverarbeitung, Automatisierung und Skripting
* **Chloros Python SDK** — Programmatische Python-Schnittstelle (`pip install chloros-sdk`) zur Integration in Forschungspipelines und benutzerdefinierte Tools
* **GPU-Beschleunigung** — CUDA-beschleunigte Verarbeitung auf NVIDIA-GPUs (Desktop und Jetson)
* **Dynamische Rechenanpassung** — Automatische Hardwareerkennung und Optimierung der Verarbeitungsstrategie
* **Alle Verarbeitungsfunktionen** — Gleiche multispektrale Verarbeitungspipeline wie Windows (Kalibrierung, Vignettierungskorrektur, Vegetationsindizes, alle Exportformate)
* **Chloros+-Funktionen** — Multithread-Verarbeitung, texturbasierter Debayer, benutzerdefinierte Indizes (mit Chloros+-Lizenz)

## Was Linux-Benutzer nicht erhalten

* **Desktop-GUI** — Keine grafische Benutzeroberfläche; die gesamte Interaktion erfolgt über CLI oder Python SDK
* **Bildbetrachter** — Kein interaktiver Bildbetrachter, keine Rasteransicht und keine Kartenmarkierungen
* **Visuelles Projektmanagement** — Projekte werden über CLI-Befehle und SDK-Aufrufe verwaltet***

## Erste Schritte mit Linux

1. **Installieren Sie Chloros** — Siehe [Linux Installation](linux-installation.md) für die Installation des `.deb`-Pakets
2. **Installieren Sie Python und SDK** (optional) — `pip install chloros-sdk`
3. **Aktivieren Sie Ihre Lizenz** — `chloros-cli login your@email.com 'password'`
4. **Verarbeiten Sie Ihren ersten Datensatz** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson-Benutzer finden im speziellen [NVIDIA Jetson-Leitfaden](nvidia-jetson-guide.md) Informationen zur plattformspezifischen Einrichtung und Optimierung.

***

## Nächste Schritte

* [Linux Installation](linux-installation.md) — Detaillierte Installationsanweisungen für amd64 und arm64
* [NVIDIA Jetson-Handbuch](nvidia-jetson-guide.md) – Jetson-spezifische Einrichtung, Wärmemanagement und Einsatz vor Ort
* [CLI : Befehlszeile](../CLI.md) – Vollständige CLI-Referenz
* [API : Python SDK](../api-python-sdk.md) – Vollständige SDK-Referenz
* [Dynamische Rechenanpassung](../processing-architecture/dynamic-compute-adaptation.md) — Wie sich Chloros an Ihre Hardware anpasst
