---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Laden Sie die neueste Version von Chloros herunter, um mit der multispektralen Bildverarbeitung zu beginnen.

### Systemanforderungen

| Anforderung          | Minimum                                              | Empfohlen                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Betriebssystem** | Windows 10 (64-Bit)                                  | Windows 11 (64-Bit)                                  |
| **Prozessor**        | Intel Core i5 oder gleichwertig                          | Intel Core i7 oder besser                              |
| **Arbeitsspeicher (RAM)**     | 8 GB                                                  | 16 GB oder mehr                                         |
| **Grafikkarte**    | DirectX 11-kompatibel                                | NVIDIA-GPU mit 4 GB+ VRAM                            |
| **Speicherplatz**          | 6 GB freier Speicherplatz                                       | SSD mit 10 GB+ freiem Speicherplatz                            |
| **Bildschirm**          | 1920 x 1080                                            | 2560 x 1440 oder höher                                  |
| **Internet**         | Erforderlich für die [optionale] Aktivierung der Chloros+-Lizenz | Erforderlich für die [optionale] Aktivierung der Chloros+-Lizenz |

{% hint style="info" %}
**GPU-Beschleunigung**: Chloros+-Benutzer mit NVIDIA-GPUs können die CUDA-Beschleunigung für eine deutlich schnellere Verarbeitung nutzen. Chloros+-Benutzer profitieren außerdem von Multithreading für maximale Geschwindigkeit.
{% endhint %}

***

## Chloros herunterladen

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Chloros hier herunterladen</a>

### Neueste stabile Version

**Chloros-Installationsprogramm für Windows*** **Version**: 1.0.5
* **Veröffentlichungsdatum**: 10. Februar 2026
* **Dateigröße (Download)**: 1,6 GB
* **Dateigröße (installiert)**: 5,7 GB
* **Dateityp**: .exe (Windows-Installationsprogramm)

#### **Installationsschritte:**

1. Laden Sie die Datei `CHLOROS INSTALLER - CURRENT VERSION.exe` herunter.
2. Doppelklicken Sie auf das Installationsprogramm, um die Installation zu starten.
3. Befolgen Sie die Anweisungen des Installationsassistenten.
4. Wählen Sie das Installationsverzeichnis (Standard: `C:\Program Files\[USER]\Chloros\`).
5. Schließen Sie die Installation ab und starten Sie Chloros oder Chloros CLI
6. Melden Sie sich mit Ihrem [MAPIR Cloud Chloros+ Konto](https://cloud.mapir.camera/pricing) an (oder fahren Sie mit der kostenlosen Version fort).

{% hint style="success" %}
Das Installationsprogramm fügt `chloros-cli` automatisch zu Ihrem System-PATH hinzu, um den Zugriff über die Befehlszeile zu ermöglichen.
{% endhint %}

***

## Weitere Ressourcen

### Python SDK

Für Entwickler und Automatisierungs-Workflows installieren Sie Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python SDK](api-python-sdk.md)**Anforderungen**: Chloros Desktop muss installiert sein, Chloros+ Lizenzanmeldung erforderlich.***

## Lieferumfang

Die Installation von Chloros umfasst:

* ✅ **Chloros** – Grafische Benutzeroberfläche (GUI) mit vollem Funktionsumfang
* ✅ **Chloros CLI** – Befehlszeilenschnittstelle (erfordert Chloros+-Lizenz)
* ✅ **Chloros SDK** – Python API (erfordert Chloros+ Lizenz)
* ✅ **Kameraprofile** – Vorkonfigurierte MAPIR-Kameravorlagen***

## Upgrade auf Chloros+

Schalten Sie mit einem Chloros+-Abonnement erweiterte Funktionen frei:

* 🚀 **Multithread-Verarbeitung** – Bilder parallel verarbeiten
* ⚡ **GPU-Beschleunigung (CUDA)** – Nutzen Sie die Leistung der NVIDIA-GPU
* 💻 **CLI-Zugriff** – Automatisieren Sie mit Befehlszeilentools
* 🐍 **Python SDK** – Programmatischer API-Zugriff
* 📱 **Mehrere Geräte** – Verwendung auf 2 bis 10+ Geräten (abhängig vom Tarif)
* **🐻 Erweiterte texturabhängige Debayer-Methode** – ein hochwertiges, kantenabhängiges Debayering in Kombination mit einem AI/ML-Entrauschungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt. 
* 🧮 **Benutzerdefinierte Formeln** – Erstellen Sie benutzerdefinierte multispektrale Indizes

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ Pläne und Preise anzeigen</a></p>***

## Hilfe zur Installation

### Fehlerbehebung

**Die Installation schlägt mit folgender Fehlermeldung fehl:**

* Stellen Sie sicher, dass Sie über Administratorrechte verfügen.
* Deaktivieren Sie vorübergehend Ihre Antivirensoftware.
* Überprüfen Sie, ob Sie die Mindestsystemanforderungen erfüllen.

**Die Anwendung lässt sich nicht starten:**

* Überprüfen Sie, ob Windows 10/11 (64-Bit) installiert ist
* Aktualisieren Sie die Grafiktreiber
* Überprüfen Sie die Ereignisanzeige von Windows auf Fehlerdetails
* Wenden Sie sich mit den Fehlerprotokollen an den Support

**Probleme bei der Lizenzaktivierung:**

* Stellen Sie sicher, dass die Internetverbindung aktiv ist.
* Überprüfen Sie die Anmeldedaten unter [https://cloud.mapir.camera](https://cloud.mapir.camera).
* Stellen Sie sicher, dass die Firewall Chloros nicht blockiert.
* Ausführliche Anweisungen finden Sie unter [Chloros+ Anmeldung](chloros+-login.md)

### Support erhalten

Benötigen Sie Hilfe bei der Installation oder Einrichtung?

* 📧 **E-Mail**: info@mapir.camera
* 🌐 **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Erste Schritte](./)
* ❓ **FAQ**: [Häufig gestellte Fragen](faq.md)***

## Änderungsprotokoll

<details>

<summary>Version 1.0.5</summary>

#### **Veröffentlichungsdatum**: 10. Februar 2026**Neue Funktionen*** **Texture Aware Debayer-Verfahren \[Chloros+ Only] –** Texture Aware verwendet ein hochwertiges, kantenbewusstes Debayer-Verfahren in Kombination mit einem AI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt.
* **Unterstützung für T4P-Kalibrierungsziele*** **Schnellere Chloros+ GPU-Verarbeitung, bessere Speicherverwaltung**

**Fehlerbehebungen*** Komplett neues Frontend (GUI), sollte jetzt auf allen Windows-Computern funktionieren.

</details>

<details>

<summary>Version 1.0.4</summary>

#### **Veröffentlichungsdatum**: 5. Januar 2026**Neue Funktionen*** **Bild-/Metadaten-Umschaltung**: Im Dateibrowser wurde eine Umschaltfunktion hinzugefügt, um die Metadaten des ausgewählten Bildes in einer Tabelle anstelle des Bildrasters anzuzeigen.
* **Bildraster-Zoom-Schieberegler**: Neuer UI-Schieberegler zum Anpassen der Miniaturansichtsgröße (unterstützt auch STRG + Mausrad).
* **Export-Schaltflächen für Bildraster**: Schaltflächen in der oberen Zeile zum Umschalten der Miniaturansichten von JPG zu verarbeiteten Exporten (Ziele, Reflexion, Index, LUT).
* **Registerkarte „Karte“**: Neue interaktive 2D-Karte mit GPS-Standortmarkierungen für Bilder.
  * Unterstützt Google Maps und ESRI-Kartenkacheln (wählt automatisch den besten Kachel-Dienst basierend auf der Verfügbarkeit der Zoomstufe aus).
  * Miniaturansicht bei Mausberührung auf Kartenmarkierungen.

**Fehlerbehebungen*** Verbesserte Unterstützung für die Installation von Chloros auf nicht englischsprachigen Computern.

</details>

<details>

<summary>Version 1.0.3</summary>

#### **Veröffentlichungsdatum**: 20. Dezember 2025**Neue Funktionen*** Erstmalige Veröffentlichung

**Verbesserungen*** Erstmalige Veröffentlichung

**Fehlerbehebungen*** Erstmalige Veröffentlichung

**Bekannte Probleme*** Erstmalige Veröffentlichung

</details>***

## Lizenzvereinbarung**Proprietäre Software** – Copyright (c) 2026 MAPIR Inc.

Die unbefugte Nutzung, Verbreitung oder Änderung ist untersagt.

**Kostenlose Version**: Verfügbar für den privaten und gewerblichen Gebrauch mit eingeschränkten Funktionen.**Chloros+**: Abonnementbasierte Lizenz für erweiterte Funktionen und gewerbliche Nutzung.
