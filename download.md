---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Laden Sie die neueste Version von Chloros herunter, um mit der Verarbeitung multispektraler Bilder zu beginnen.

### Systemanforderungen

#### Windows

| Anforderung          | Mindestanforderungen                                              | Empfohlene Anforderungen                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Betriebssystem** | Windows 10 (64-Bit)                                  | Windows 11 (64-Bit)                                  |
| **Prozessor**        | Intel Core i5 oder gleichwertig                          | Intel Core i7 oder besser                              |
| **Arbeitsspeicher (RAM)**     | 8 GB                                                  | 16 GB oder mehr                                         |
| **Grafikkarte**    | DirectX 11-kompatibel                                | NVIDIA-GPU mit 4 GB+ VRAM                            |
| **Speicherplatz**          | 6 GB freier Speicherplatz                                       | SSD mit 10 GB+ freiem Speicherplatz                            |
| **Bildschirm**          | 1920x1080                                            | 2560x1440 oder höher                                  |
| **Internet**         | Erforderlich für die [optionale] Chloros+ Lizenzaktivierung | Erforderlich für die [optionale] Chloros+ Lizenzaktivierung |

#### Linux amd64 (x86\_64)

| Anforderung       | Minimum                    | Empfohlen               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Prozessor**     | x86\_64 (Intel/AMD)        | Intel Core i7 oder besser   |
| **Arbeitsspeicher (RAM)**  | 8 GB                        | 16 GB oder mehr              |
| **Grafikkarte** | Keine (CPU-Verarbeitung)      | NVIDIA-GPU mit 4 GB+ VRAM |
| **Speicherplatz**       | 2 GB freier Speicherplatz             | SSD mit 10 GB+ freiem Speicherplatz       |
| **Python**        | Python 3.7+ (für SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Anforderung      | Minimum                      | Empfohlen                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plattform**     | NVIDIA Jetson mit JetPack 6 | Jetson Orin NX 16 GB oder AGX Orin |
| **Arbeitsspeicher (RAM)** | 8 GB (gemeinsam genutzt von GPU/CPU)         | 16 GB+ gemeinsam genutzt                    |
| **Speicherplatz**      | 2 GB freier Speicherplatz               | NVMe-SSD mit 10 GB+ freiem Speicherplatz        |
| **Python**       | Python 3.7+ (für SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-Beschleunigung**: Nutzer von Chloros+ mit NVIDIA-GPUs können die CUDA-Beschleunigung für eine deutlich schnellere Verarbeitung nutzen. Dies funktioniert sowohl auf Windows (Desktop-GPUs) als auch auf Linux (Desktop-GPUs und NVIDIA Jetson). Chloros+-Benutzer profitieren zudem von Multithread-Verarbeitung für maximale Geschwindigkeit.
{% endhint %}

***

## Chloros herunterladen

### Aktuelle stabile Version (23. März 2026): Version 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Chloros für Windows (.exe) herunterladen</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Chloros für Linux amd64 (.deb) herunterladen</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Chloros für Linux arm64 / Jetson (.deb) herunterladen####</a>

Windows-Installationsprogramm (GUI + CLI + Backend)

* **Dateityp**: .exe (Windows-Installationsprogramm)**Installationsschritte:**

1. Laden Sie die oben genannte .exe-Datei herunter
2. Doppelklicken Sie auf das Installationsprogramm, um die Installation zu starten
3. Befolgen Sie die Anweisungen des Installationsassistenten
4. Wählen Sie das Installationsverzeichnis (Standard: `C:\Program Files\[USER]\Chloros\`)
5. Schließen Sie die Installation ab und starten Sie Chloros oder Chloros CLI
6. Melden Sie sich mit Ihrem [MAPIR Cloud Chloros+-Konto](https://cloud.mapir.camera/pricing) an (oder fahren Sie mit der kostenlosen Version fort)

{% hint style="success" %}
Das Installationsprogramm fügt `chloros-cli` automatisch zum System-PATH für den Zugriff über die Befehlszeile hinzu.
{% endhint %}

#### Linux amd64 (.deb-Paket – CLI + Backend)

* **Dateityp**: .deb (Debian/Ubuntu-Paket)
* **Architektur**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb-Paket — CLI + Backend)

* **Dateityp**: .deb (JetPack 6)
* **Architektur**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Ausführliche Installationsanweisungen finden Sie unter [Linux Installation](linux/linux-installation.md) und jetsonspezifische Hinweise im [NVIDIA Jetson-Handbuch](linux/nvidia-jetson-guide.md).

#### Python SDK (Alle Plattformen)

```bash
pip install chloros-sdk
```

Siehe [API : Python SDK](api-python-sdk.md) für die Dokumentation.

{% hint style="info" %}
**Linux-Benutzer**: Das Paket `.deb` installiert CLI und das Backend. Python und SDK werden separat über pip installiert. Es gibt keine GUI für Linux – die gesamte Interaktion erfolgt über CLI oder SDK.
{% endhint %}

***

## Weitere Ressourcen

### Python SDK

Für Entwickler und Automatisierungs-Workflows installieren Sie Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python SDK](api-python-sdk.md)**Voraussetzungen**: Chloros muss installiert sein (Windows-Installationsprogramm oder Linux `.deb`-Paket), Chloros+ Lizenz-Login erforderlich***

## Lieferumfang

### Windows-Installationsprogramm

* ✅ **Chloros GUI** – Grafische Benutzeroberfläche mit vollem Funktionsumfang
* ✅ **Chloros CLI** – Befehlszeilenschnittstelle (erfordert Chloros+-Lizenz)
* ✅ **Chloros Backend** – Verarbeitungsengine
* ✅ **Kameraprofile** – Vorkonfigurierte MAPIR-Kameravorlagen

### Linux .deb-Paket

* ✅ **Chloros CLI** – Befehlszeilenschnittstelle (erfordert Chloros+-Lizenz)
* ✅ **Chloros Backend** – Verarbeitungs-Engine
* ✅ **Kameraprofile** – Vorkonfigurierte MAPIR-Kameravorlagen
* ❌ Keine GUI – Linux ist nur als headless CLI/SDK verfügbar

### Python SDK (pip, alle Plattformen)

* ✅ **Chloros SDK** – Python API (erfordert eine Chloros+-Lizenz)***

## Upgrade auf Chloros+

Schalten Sie mit einem Chloros+-Abonnement erweiterte Funktionen frei:

* 🚀 **Multithread-Verarbeitung** – Verarbeiten Sie Bilder parallel
* ⚡ **GPU (CUDA)-Beschleunigung** – Nutzen Sie die Leistung von NVIDIA-GPUs
* 💻 **CLI-Zugriff** – Automatisieren Sie mit Befehlszeilentools
* 🐍 **Python SDK** – Programmatischer API-Zugriff
* 📱 **Mehrere Geräte** – Nutzung auf 2–10+ Geräten (abhängig vom Tarif)
* **🐻 Erweiterte texturbasierte Debayering-Methode** – ein hochwertiges, kantensensitives Debayering-Verfahren in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt.
* 🧮 **Benutzerdefinierte Formeln** – Erstellen Sie benutzerdefinierte Multispektralindizes

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ Tarife &amp; Preise anzeigen</a></p>***

## Hilfe zur Installation

### Fehlerbehebung

**Die Installation schlägt mit folgender Fehlermeldung fehl:**

* Stellen Sie sicher, dass Sie über Administratorrechte verfügen
* Deaktivieren Sie vorübergehend Ihre Antivirensoftware
* Überprüfen Sie, ob die Mindestsystemanforderungen erfüllt sind

**Die Anwendung startet nicht (Windows):**

* Überprüfen Sie, ob Windows 10/11 (64-Bit) installiert ist
* Aktualisieren Sie die Grafiktreiber
* Überprüfen Sie den Windows-Ereignisanzeiger auf Fehlerdetails
* Wenden Sie sich mit den Fehlerprotokollen an den Support

**CLI lässt sich nicht starten (Linux):**

* Überprüfen Sie, ob das `.deb`-Paket korrekt installiert ist: `dpkg -l | grep chloros`
* Überprüfen Sie die Berechtigungen: `sudo chmod +x /usr/bin/chloros-cli`
* Führen Sie eine Diagnose durch: `chloros-cli selftest`
* Überprüfen Sie, ob Bibliotheken fehlen: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Probleme bei der Lizenzaktivierung:**

* Stellen Sie sicher, dass eine Internetverbindung besteht
* Überprüfen Sie die Anmeldedaten unter [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Überprüfen Sie, ob die Firewall Chloros blockiert
* Detaillierte Anweisungen finden Sie unter [Chloros+ Login](chloros+-login.md)

### Support erhalten

Benötigen Sie Hilfe bei der Installation oder Einrichtung?

* 📧 **E-Mail**: info@mapir.camera
* 🌐 **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Erste Schritte](./)
* ❓ **FAQ**: [Häufig gestellte Fragen](faq.md)***

## Änderungsprotokoll

<details>

<summary>Version 1.1.0 (Aktuellste)</summary>

**Veröffentlichungsdatum: März 2026**

**Neue Funktionen*** **Linux-Unterstützung** — Native CLI- und SDK-Unterstützung für Linux amd64 (x86\_64) und arm64 (NVIDIA Jetson JetPack 6). Installation über `.deb`-Pakete.
* **NVIDIA Jetson-Unterstützung** — Optimierte Verarbeitung für Jetson Nano-, Orin Nano-, Orin NX- und AGX Orin-Edge-Geräte.
* **Dynamische Rechenanpassung** — Automatische Hardwareerkennung und Optimierung der Verarbeitungsstrategie. Chloros passt sich an Ihre Hardware an, vom Jetson Nano bis hin zu einer Workstation mit mehreren GPUs.
* **4-Thread-Verarbeitungspipeline** — Parallele Threads für Erkennung, Kalibrierung, Verarbeitung und Export mit dynamischer GPU-Speicherzuweisung.
* **Neue CLI-Befehle** — `selftest` (Systemdiagnose) und `update` (Linux-Update-Verwaltung).
* **Neue CLI-Prozessflags** — `--debayer` (Standard/texturorientiert), `--indices` (Indizes angeben), `--target` (zuerst Zielunterordner durchsuchen für schnellere Erkennung).
* **Neue GUI-Menüpunkte** — „Dateien hinzufügen“, „Ordner hinzufügen“ und „Verarbeitung starten/stoppen“ sind nun über das Dropdown-Menü des Hauptmenüs zugänglich.**Verbesserungen**

* Plattformübergreifende automatische Backend-Erkennung (Pfade Windows und Linux)
* Verbessertes SDK `get_status()` mit Fortschrittsverfolgung pro Thread
* Neue SDK-Ausnahmen: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Wärmemanagement und adaptive Drosselung für NVIDIA Jetson
* Automatische Speicherverwaltung mit OOM-Fallback auf gekachelte GPU-Verarbeitung

</details>

<details>

<summary>Version 1.0.5</summary>

**Veröffentlichungsdatum: 10. Februar 2026**

**Neue Funktionen*** **Texture Aware Debayer-Methode \[Nur Chloros+] –** Texture Aware verwendet einen hochwertigen, kantenbewussten Debayer in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt.
* **Unterstützung für T4P-Kalibrierungsziele*** **Schnellere Chloros+-GPU-Verarbeitung, besseres Speichermanagement**

**Fehlerbehebungen*** Komplett neues Frontend (GUI), sollte nun auf allen Windows-Computern funktionieren.

</details>

<details>

<summary>Version 1.0.4</summary>

**Veröffentlichungsdatum: 5. Januar 2026**

**Neue Funktionen*** **Umschalten zwischen Bild und Metadaten**: Im Dateibrowser wurde eine Umschaltfunktion hinzugefügt, um die Metadaten des ausgewählten Bildes in einer Tabelle statt im Bildraster anzuzeigen
* **Zoom-Schieberegler für Bildraster**: Neuer UI-Schieberegler zum Anpassen der Miniaturbildgröße (unterstützt auch STRG + Mausrad)
* **Exportschaltflächen für Bildraster**: Schaltflächen in der obersten Zeile zum Umschalten der Miniaturansichten von JPG auf verarbeitete Exporte (Targets, Reflectance, Index, LUT)
* **Registerkarte „Karte“**: Neue interaktive 2D-Karte mit GPS-Standortmarkierungen für Bilder
  * Unterstützt Google Maps und ESRI-Kartenkacheln (wählt automatisch den besten Kacheldienst basierend auf der Verfügbarkeit der Zoomstufen aus)
  * Vorschau der Miniaturansicht bei Mauszeiger über Kartenmarkierungen

**Fehlerbehebungen*** Verbesserte Unterstützung für die Installation von Chloros auf nicht englischsprachigen Computern

</details>

<details>

<summary>Version 1.0.3</summary>

**Veröffentlichungsdatum: 20. Dezember 2025**

**Neue Funktionen*** Erstveröffentlichung

**Verbesserungen*** Erstveröffentlichung

**Fehlerbehebungen*** Erstveröffentlichung

**Bekannte Probleme*** Erstveröffentlichung

</details>***

## Lizenzvereinbarung**Urheberrechtlich geschützte Software** – Copyright (c) 2026 MAPIR Inc.

Die unbefugte Nutzung, Verbreitung oder Änderung ist untersagt.

**Kostenlose Version**: Für den privaten und gewerblichen Gebrauch mit Funktionseinschränkungen verfügbar**Chloros+**: Abonnementbasierte Lizenz für erweiterte Funktionen und gewerbliche Einsätze
