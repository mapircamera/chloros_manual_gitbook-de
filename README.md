---
metaLinks: {}
---

# Erste Schritte

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros ist eine Softwareanwendung von [MAPIR](https://www.mapir.camera) zur Verarbeitung von Bildern und anderen Sensordaten.

***{% hint style="success" %}**Neuerungen in Chloros 1.1.0**: Native Linux-Unterstützung (amd64 und arm64), NVIDIA Jetson Edge-Computing, Dynamic Compute Adaptation, 4-Thread-Verarbeitungspipeline, neue CLI-Befehle und -Optionen. Die vollständige Änderungsliste finden Sie unter [Download](download.md).
{% endhint %}

Chloros ist in 3 Anwendungsmodi verfügbar:

## Chloros: Desktop-GUI-Anwendung

Eigenständiges separates Fenster mit allen Funktionen. _Nur Windows._

## [Chloros CLI: Befehlszeilenschnittstelle](CLI.md)

Batch-Verarbeitung über die Befehlszeile. Ideal für Automatisierung, Skripting und den Headless-Betrieb. Verfügbar auf **Windows, Linux amd64 und Linux arm64 (NVIDIA Jetson)**. _Für den Zugriff auf die CLI ist eine Chloros+-Lizenz erforderlich._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmatische Python-Schnittstelle für Automatisierung und benutzerdefinierte Workflows. Ideal für Forschungspipelines, die Integration in bestehende Python-Anwendungen und die Entwicklung benutzerdefinierter Tools. Verfügbar auf **allen Plattformen** über `pip install chloros-sdk`. _Für den Zugriff auf die API ist eine Chloros+-Lizenz erforderlich._***

## Unterstützte Plattformen

| Plattform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nein | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nein | Ja | Ja |

Anweisungen zur Installation von Linux finden Sie im Abschnitt [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Zwar ist Chloros für die meisten Aufgaben kostenlos nutzbar, doch vielleicht möchten Sie mehr. In diesem Fall kann eine kostenpflichtige Lizenz für Chloros+ für Sie von Vorteil sein. Mit einer Chloros+-Lizenz können Sie neue Funktionen freischalten, wie zum Beispiel:

* **Multithread-Verarbeitung**: Beschleunigen Sie die Bildverarbeitung bei größeren Projekten erheblich, indem Bilder gleichzeitig durch die Pipeline verarbeitet werden.
* **GPU-Beschleunigung (CUDA)**: Nutzen Sie die Vorteile der heutigen GPU-Speicheroptionen mit höherer Kapazität, um die Bildverarbeitungs-Pipeline weiter zu beschleunigen. Für optimale Ergebnisse empfehlen wir 4 GB oder mehr VRAM.
* **Chloros+**[**CLI**](CLI.md)**Zugriff**: Führen Sie Chloros+ über die Befehlszeile aus, um die Verarbeitung zu automatisieren und in Ihre eigene Software zu integrieren.
* **Chloros+**[**API**](api-python-sdk.md)**Zugriff:** Führen Sie Chloros+ über Python aus, um eine programmatische Steuerung zu ermöglichen und eine nahtlose Integration in Ihre Forschungspipelines, Datenanalyse-Workflows und benutzerdefinierten Anwendungen zu gewährleisten.
* **Nutzung auf mehreren Geräten**: Jede Chloros+-Lizenz ermöglicht die Registrierung von 2 oder mehr Geräten. Verwenden Sie Ihr MAPIR-Cloud-Konto, um registrierte Geräte zu verwalten. Erweitern Sie die Unterstützung für weitere Geräte durch ein Upgrade Ihrer Chloros+-Lizenz.
* **Erweiterte texturbasierte Debayering-Methode:** Ein hochwertiges, kantenbewusstes Debayering-Verfahren in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt.
* **Benutzerdefinierte multispektrale Indexformeln:** Geben Sie benutzerdefinierte multispektrale Indizes in die Chloros-Rasterrechner ein, sowohl für die Verarbeitung als auch für die Bildbetrachtungs-Sandbox.
* **Linux &amp; Edge Computing:** Führen Sie Chloros auf Linux x86_64- und ARM64-Plattformen einschließlich NVIDIA Jetson für die Feld- und Edge-Verarbeitung aus. Siehe [Linux-Übersicht](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Preise &amp; Anmeldung</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
