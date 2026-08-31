---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Laden Sie die neueste Version von Chloros herunter, um mit der multispektralen Bildverarbeitung zu beginnen.

### Systemanforderungen

#### Windows

| Anforderung          | Mindestanforderungen                                              | Empfohlene Anforderungen                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Betriebssystem** | Windows 10 (64-Bit)                                  | Windows 11 (64-Bit)                                  |
| **Prozessor**        | Intel Core i5 oder gleichwertig                          | Intel Core i7 oder besser                              |
| **Arbeitsspeicher (RAM)**     | 8 GB                                                  | 16 GB oder mehr                                         |
| **Grafikkarte**    | DirectX 11-kompatibel                                | NVIDIA-GPU mit mindestens 4 GB VRAM                            |
| **Speicherplatz**          | 6 GB freier Speicherplatz                                       | SSD mit mindestens 10 GB freiem Speicherplatz                            |
| **Bildschirm**          | 1920 × 1080                                            | 2560 × 1440 oder höher                                  |
| **Internet**         | Erforderlich für die [optionale] Chloros+-Lizenzaktivierung | Erforderlich für die [optionale] Chloros+-Lizenzaktivierung |

#### Linux amd64 (x86_64)

| Anforderung       | Mindestanforderungen                    | Empfohlene Anforderungen               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Prozessor**     | x86\_64 (Intel/AMD)        | Intel Core i7 oder besser   |
| **Arbeitsspeicher (RAM)** | 8 GB                        | 16 GB oder mehr              |
| **Grafikkarte** | Keine (CPU-Verarbeitung)      | NVIDIA-GPU mit 4 GB+ VRAM |
| **Speicherplatz**       | 2 GB freier Speicherplatz             | SSD mit mindestens 10 GB freiem Speicherplatz       |
| **Python**        | Python 3.7+ (für SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Anforderung      | Mindestanforderung                      | Empfehlung                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plattform**     | NVIDIA Jetson mit JetPack 6 | Jetson Orin NX 16 GB oder AGX Orin |
| **Arbeitsspeicher (RAM)** | 8 GB (gemeinsam genutzt von GPU und CPU)         | 16 GB+ gemeinsam genutzt                    |
| **Speicherplatz**      | 2 GB freier Speicherplatz               | NVMe-SSD mit mindestens 10 GB freiem Speicherplatz        |
| **Python**       | Python 3.7+ (für SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-Beschleunigung**: Nutzer von Chloros+ mit NVIDIA-GPUs können die CUDA-Beschleunigung nutzen, um die Verarbeitung deutlich zu beschleunigen. Dies funktioniert sowohl auf Windows (Desktop-GPUs) als auch auf Linux (Desktop-GPUs und NVIDIA Jetson). Nutzer von Chloros+ profitieren zudem von einer Multithread-Verarbeitung für maximale Geschwindigkeit.
{% endhint %}

***

## Chloros herunterladen

### Aktuelle stabile Version: Version 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Chloros für Windows (.exe) herunterladen</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Chloros für Linux amd64 (.deb) herunterladen</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Chloros für Linux arm64 / Jetson (.deb) herunterladen</a>

#### Windows-Installationsprogramm (GUI + CLI + Backend)

* **Dateityp**: .exe (Windows-Installationsprogramm)**Installationsschritte:**

1. Laden Sie die oben genannte .exe-Datei herunter
2. Doppelklicken Sie auf das Installationsprogramm, um die Installation zu starten
3. Folgen Sie den Anweisungen des Installationsassistenten
4. Wählen Sie das Installationsverzeichnis aus (Standard: `C:\Program Files\MAPIR\Chloros\`)
5. Schließen Sie die Installation ab und starten Sie Chloros oder Chloros CLI
6. Melden Sie sich mit Ihrem [MAPIR Cloud Chloros+-Konto](https://cloud.mapir.camera/pricing) an (oder fahren Sie mit der kostenlosen Version fort)

{% hint style="success" %}
Das Installationsprogramm fügt `chloros-cli` automatisch zum PATH Ihres Systems hinzu, um den Zugriff über die Befehlszeile zu ermöglichen.
{% endhint %}

#### Linux amd64 (.deb-Paket – CLI + Backend)

* **Dateityp**: .deb (Debian/Ubuntu-Paket)
* **Architektur**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 – NVIDIA Jetson (.deb-Paket – CLI + Backend)

* **Dateityp**: .deb (JetPack 6)
* **Architektur**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Ausführliche Anweisungen zur Einrichtung finden Sie unter [Linux-Installation](linux/linux-installation.md) und jetsonspezifische Hinweise im [NVIDIA Jetson-Handbuch](linux/nvidia-jetson-guide.md).

#### Python SDK (alle Plattformen)

Jedes Installationsprogramm enthält ein passendes `chloros_sdk`-Wheel, sodass die SDK-Version stets mit der installierten GUI/CLI/dem Backend übereinstimmt. Unter Windows installiert das Installationsprogramm es automatisch in Ihrem System Python; bei Linux legt das `.deb` das Wheel unter `/usr/lib/chloros/sdk/` ab und gibt den Installationsbefehl aus:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Für reine Pip-Hosts (auf denen kein Chloros-Paket installiert ist) ist das SDK ebenfalls auf PyPI verfügbar:

```bash
pip install chloros-sdk
```

Siehe [API : Python SDK](api-python-sdk.md) sowie die [SDK-Referenz](reference/sdk-reference.md) zur Dokumentation.

{% hint style="info" %}
**Linux-Benutzer**: Das Paket `.deb` installiert das CLI und das Backend. Für Linux gibt es keine grafische Benutzeroberfläche – die gesamte Interaktion erfolgt über CLI oder SDK.
{% endhint %}

***

## Weitere Ressourcen

### Python SDK

Für Entwickler und Automatisierungsworkflows installieren Sie Chloros, Python und SDK:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python SDK](api-python-sdk.md)**Voraussetzungen**: Chloros muss installiert sein (Windows-Installationsprogramm oder Linux `.deb`-Paket), Chloros+ Lizenz-Anmeldung erforderlich***

## Lieferumfang

### Windows-Installationsprogramm

* ✅ **Chloros-GUI** – Voll funktionsfähige grafische Benutzeroberfläche
* ✅ **Chloros CLI** – Befehlszeilenschnittstelle (erfordert eine Chloros+-Lizenz)
* ✅ **Chloros-Backend** – Verarbeitungsengine
* ✅ **Kameraprofile** – Vorkonfigurierte MAPIR-Kameravorlagen

### Linux .deb-Paket

* ✅ **Chloros CLI** – Befehlszeilenschnittstelle (erfordert eine Chloros+-Lizenz)
* ✅ **Chloros-Backend** – Verarbeitungs-Engine
* ✅ **Kameraprofile** – Vorkonfigurierte MAPIR-Kameravorlagen
* ❌ Keine grafische Benutzeroberfläche – Linux ist nur als Headless-Version (CLI/SDK) verfügbar

### Python SDK (PIP, alle Plattformen)

* ✅ **Chloros SDK** – Python API (erfordert eine Chloros+-Lizenz)***

## Upgrade auf Chloros+

Schalten Sie mit einem Chloros+-Abonnement erweiterte Funktionen frei:

* 🚀 **Multithread-Verarbeitung** – Bilder parallel verarbeiten
* ⚡ **GPU-Beschleunigung (CUDA)** – Die Leistung von NVIDIA-GPUs nutzen
* 💻 **Zugriff auf CLI** – Automatisierung mit Befehlszeilentools
* 🐍 **Python SDK** – Programmatischer Zugriff auf API
* 📱 **Mehrere Geräte** – Nutzung auf 2 bis 10+ Geräten (abhängig vom Tarif)
* **🐻 Erweiterte texturbasierte Debayering-Methode** – ein hochwertiges, kantensensitives Debayering-Verfahren in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt.
* 🧮 **Benutzerdefinierte Formeln** – Erstellen Sie benutzerdefinierte Multispektralindizes

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+-Tarife und Preise anzeigen</a></p>***

## Hilfe zur Installation

### Fehlerbehebung

**Die Installation schlägt mit folgender Fehlermeldung fehl:**

* Stellen Sie sicher, dass Sie über Administratorrechte verfügen
* Deaktivieren Sie vorübergehend Ihre Antivirensoftware
* Überprüfen Sie, ob die Mindestsystemanforderungen erfüllt sind

**Die Anwendung lässt sich nicht starten (Windows):**

* Vergewissern Sie sich, dass Windows 10/11 (64-Bit) installiert ist
* Aktualisieren Sie die Grafiktreiber
* Überprüfen Sie die Ereignisanzeige von Windows auf Fehlerdetails
* Wenden Sie sich mit den Fehlerprotokollen an den Support

**CLI lässt sich nicht starten (Linux):**

* Überprüfen Sie, ob das `.deb`-Paket korrekt installiert ist: `dpkg -l | grep chloros`
* Überprüfen Sie die Berechtigungen: `sudo chmod +x /usr/bin/chloros-cli`
* Führen Sie eine Diagnose durch: `chloros-cli selftest`
* Auf fehlende Bibliotheken prüfen: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Probleme bei der Lizenzaktivierung:**

* Stellen Sie sicher, dass eine Internetverbindung besteht
* Überprüfen Sie die Anmeldedaten unter [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Stellen Sie sicher, dass die Firewall Chloros nicht blockiert
* Detaillierte Anweisungen finden Sie unter [Chloros+ Anmeldung](chloros+-login.md)

### Support erhalten

Benötigen Sie Hilfe bei der Installation oder Einrichtung?

* 📧 **E-Mail**: info@mapir.camera
* 🌐 **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Erste Schritte](./)
* ❓ **FAQ**: [Häufig gestellte Fragen](faq.md)***

## Software-Updates

Chloros sucht nach Updates, meldet, wenn eine neue Version verfügbar ist, und verlinkt auf diese Download-Seite – Sie führen das Update durch, indem Sie das neue signierte Installationsprogramm ausführen. Ihre Einstellungen und Projekte bleiben bei Updates erhalten. Unter Linux und Jetson prüft `chloros-cli update`, ob eine neuere Version verfügbar ist, und bietet an, das passende `.deb` (dieser Befehl ist nur in Linux verfügbar).

***

## Änderungsprotokoll**Version 1.2.0 (aktuellste)**– die vollständige Liste der Funktionen finden Sie unter**Neuerungen in Chloros 1.2.0** auf der Seite [Erste Schritte](./).

<details>

<summary>Version 1.0.5</summary>

**Veröffentlichungsdatum: 10. Februar 2026**

**Neue Funktionen*** **Texture Aware-Debayer-Methode \[Nur Chloros+] –** Texture Aware verwendet einen hochwertigen, kantensensitiven Debayer in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt.
* **Unterstützung für T4P-Kalibrierungsziele*** **Schnellere GPU-Verarbeitung bei Chloros+, besseres Speichermanagement**

**Fehlerbehebungen*** Völlig neues Frontend (GUI), sollte nun auf allen Windows-Computern funktionieren.

</details>

<details>

<summary>Version 1.0.4</summary>

**Veröffentlichungsdatum: 5. Januar 2026**

**Neue Funktionen*** **Umschaltfunktion für Bilder/Metadaten**: Im Dateibrowser wurde eine Umschaltfunktion hinzugefügt, um die Metadaten des ausgewählten Bildes in einer Tabelle statt im Bildraster anzuzeigen
* **Zoom-Schieberegler für das Bildraster**: Neuer UI-Schieberegler zum Anpassen der Miniaturbildgröße (unterstützt auch STRG + Mausrad)
* **Exportschaltflächen für das Bildraster**: Schaltflächen in der obersten Zeile zum Umschalten der Miniaturansichten von JPG auf verarbeitete Exportformate (Targets, Reflectance, Index, LUT)
* **Registerkarte „Karte“**: Neue interaktive 2D-Karte mit GPS-Standortmarkierungen für Bilder
  * Unterstützt Google Maps und ESRI-Kartenkacheln (wählt automatisch den besten Kacheldienst basierend auf der Verfügbarkeit der Zoomstufen aus)
  * Vorschau der Miniaturansichten bei Mauszeiger-Überfahrt über Kartenmarkierungen

**Fehlerbehebungen*** Verbesserte Unterstützung für die Installation von Chloros auf Computern mit nicht-englischer Sprache

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

**Kostenlose Version**: Verfügbar für den privaten und gewerblichen Gebrauch mit Funktionseinschränkungen**Chloros+**: Abonnementbasierte Lizenz für erweiterte Funktionen und gewerbliche Einsätze
