---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Herunterladen

Laden Sie die neueste Version von Chloros herunter, um mit der Verarbeitung multispektraler Bilder zu beginnen.

### Systemanforderungen

#### Windows


| Anforderung          | Mindestanforderungen                                              | Empfohlene Anforderungen                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Betriebssystem** | Windows 10 (64-Bit)                                  | Windows 11 (64-Bit)                                  |
| **Prozessor**        | Intel Core i5 oder gleichwertig                          | Intel Core i7 oder besser                              |
| **Arbeitsspeicher (RAM)**     | 8 GB                                                  | 16 GB oder mehr                                         |
| **Grafikkarte**    | DirectX 11-kompatibel                                | NVIDIA-GPU mit mindestens 4 GB VRAM                            |
| **Speicherplatz**          | 6 GB freier Speicherplatz                                       | SSD mit mindestens 10 GB freiem Speicherplatz                            |
| **Bildschirm**          | 1920×1080                                            | 2560×1440 oder höher                                  |
| **Internet**         | Erforderlich für die [optionale] Aktivierung der Chloros+-Lizenz | Erforderlich für die [optionale] Aktivierung der Chloros+-Lizenz |

#### Linux amd64 (x86_64)

| Anforderung       | Minimum                    | Empfohlen               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Prozessor**     | x86_64 (Intel/AMD)        | Intel Core i7 oder besser   |
| **Arbeitsspeicher (RAM)**  | 8 GB                        | 16 GB oder mehr              |
| **Grafikkarte** | Keine (CPU-Verarbeitung)      | NVIDIA-GPU mit 4 GB+ VRAM |
| **Speicherplatz**       | 2 GB freier Speicherplatz             | SSD mit mindestens 10 GB freiem Speicherplatz       |
| **Python**        | Python 3.7+ (für SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Anforderung      | Minimum                      | Empfohlen                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plattform**     | NVIDIA Jetson mit JetPack 6 | Jetson Orin NX 16 GB oder AGX Orin |
| **Arbeitsspeicher (RAM)** | 8 GB (gemeinsam genutzt von GPU und CPU)         | 16 GB+ gemeinsam genutzt                    |
| **Speicherplatz**      | 2 GB freier Speicherplatz               | NVMe-SSD mit mindestens 10 GB freiem Speicherplatz        |
| **Python**       | Python 3.7+ (für SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-Beschleunigung**: Chloros+-Benutzer mit NVIDIA-GPUs können die CUDA-Beschleunigung nutzen, um die Verarbeitung deutlich zu beschleunigen. Dies funktioniert mit NVIDIA-Desktop-GPUs unter Windows und Linux amd64 sowie auf NVIDIA Jetson mit JetPack 6. Chloros+-Benutzer profitieren zudem von einer Multithread-Verarbeitung für maximale Geschwindigkeit.
{% endhint %}

***

## Chloros
herunterladen

### Aktuelle stabile Version: Version 1.3.0 (11. November 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Chloros für Windows herunterladen (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Chloros für Linux amd64 (.deb) herunterladen</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Chloros für Linux arm64 / Jetson herunterladen (.deb)</a>


#### Windows-Installationsprogramm (GUI + CLI + Backend)

* **Dateityp**: .exe (Windows-Installationsprogramm)

**Installationsschritte:**

1. Laden Sie die oben genannte Datei „.exe“ herunter
2. Doppelklicken Sie auf das Installationsprogramm, um die Installation zu starten
3. Befolgen Sie die Anweisungen des Installationsassistenten
4. Wählen Sie das Installationsverzeichnis aus (Standard: `C:\Program Files\Chloros\`)
5. Lassen Sie auf der Seite **„Zusätzliche Aufgaben auswählen“**beide Optionen unter**„Systemeinstellungen“** aktiviert (empfohlen – siehe unten)
6. Schließen Sie die Installation ab und starten Sie Chloros oder Chloros CLI
7. Melden Sie sich mit Ihrem [MAPIR Cloud Chloros+-Konto](https://cloud.mapir.camera/pricing) an (oder fahren Sie mit der kostenlosen Version fort)

**Optionen für die Systemeinstellungen** (beide standardmäßig aktiviert):

| Option, wie im Installationsprogramm angegeben | Funktion | Deaktivierung der automatischen Installation |
| --- | --- | --- |
| **Lange Pfade unter Windows aktivieren** (ermöglicht Projektordner mit mehr als 260 Zeichen; empfohlen) | Setzt die systemweite Windows-Einstellung `LongPathsEnabled` auf 1. Windows wird standardmäßig mit deaktivierter Einstellung ausgeliefert, sodass kein Dateipfad länger als 260 Zeichen sein darf; Chloros-Produkte befinden sich bis zu etwa 130 Zeichen unterhalb des Projektordners, daher wird bei deaktivierten langen Pfaden ein tiefer liegender Projektordner abgelehnt. Gilt für Programme, die nach der Installation gestartet werden. Die Einstellung bleibt auch nach einer Deinstallation bestehen, da andere Programme möglicherweise darauf angewiesen sind. | `/MERGETASKS="!longpaths"` |
| **Das Chloros-Backend von der Überprüfung durch Microsoft Defender ausschließen** (deutlich schnellere Kameraaufzeichnung; empfohlen) | Fügt im Installationsordner eine Prozessausnahme für `chloros-backend.exe` in Microsoft Defender hinzu, sodass der Echtzeitschutz nicht jede Datei überprüft, die das Backend schreibt, unabhängig davon, wo diese gespeichert wird. Bei hohen Aufnahmeraten machen die Scanvorgänge etwa 30 % der Aufnahmerate aus (77,7 gegenüber 52,7 Aufnahmen pro Sekunde bei einer kontinuierlichen Aufnahme mit 9 Kameras). Es wird kein anderes Programm ausgeschlossen, und die Ausnahme wird bei der Deinstallation wieder aufgehoben. Wenn Defender nicht vorhanden ist oder durch Richtlinien gesperrt ist, schlägt dieser Schritt stillschweigend fehl und die Installation wird fortgesetzt. | `/MERGETASKS="!defenderexclusion"` |

Bei einer stillen Installation (`/SILENT` oder `/VERYSILENT`) werden beide Optionen angewendet, es sei denn, Sie deaktivieren dies mit dem angezeigten Wert `/MERGETASKS`.

{% hint style="success" %}
Das Installationsprogramm fügt den Ordner CLI (`C:\Program Files\Chloros\cli\`) einmalig zu Ihrem System hinzu (`PATH`) – bei einer Neuinstallation oder einem Upgrade wird kein doppelter Eintrag mehr hinzugefügt, und Duplikate, die von früheren Installationsprogrammen hinzugefügt wurden, werden entfernt. Öffnen Sie nach der Installation ein neues Terminal, um `chloros-cli` zu verwenden. Das Installationsprogramm fügt außerdem in jedem Netzwerkprofil zwei Windows-Firewall-Regeln für das Backend hinzu: **Chloros-Backend (UDP-In)**für Kamerastreams, PTP, mDNS und den DAQ-E-Datenstrom sowie**Chloros-Backend (TCP-In)**, beschränkt auf das lokale Subnetz, für ein DAQ-E-Firmware-Update, bei dem der Sensor eine Verbindung zu diesem Computer herstellt, um das Bild zu erfassen. Beide werden bei der Deinstallation entfernt.
{% endhint %}

#### Linux amd64 (.deb-Paket – CLI + Backend)

* **Dateityp**: .deb (Debian/Ubuntu-Paket)
* **Architektur**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 – NVIDIA Jetson (.deb-Paket – CLI + Backend)

* **Dateityp**: .deb (JetPack 6)
* **Architektur**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Verwenden Sie den tatsächlich heruntergeladenen Dateinamen und behalten Sie „`./`“ bei, damit „`apt`“ diese Datei installiert, anstatt nach einem Paket anhand des Namens zu suchen. Führen Sie die Installation mit „`apt install`“ durch, nicht mit `dpkg -i`: apt installiert auch die Abhängigkeiten des Python-SDKs, die das Paket empfiehlt, aber nicht erfordert, und `dpkg -i` überspringt diese. Seit Version 1.3.0 werden das Backend und die CLI unter `/usr/lib/chloros/` entpackt installiert, sodass sie starten, ohne sich zuvor selbst entpacken zu müssen.

Das Jetson-Paket verwendet die CUDA-Version von PyTorch aus JetPack 6, sodass die Verarbeitung – einschließlich des enthaltenen Texture-Aware-Debayers – auf der GPU des Jetson läuft. Es nutzt die CUDA- und cuDNN-Bibliotheken, die JetPack 6 installiert (`sudo apt install nvidia-jetpack` auf einem Image, auf dem diese fehlen).

Ausführliche Einrichtungsanweisungen finden Sie unter [Linux-Installation](linux/linux-installation.md) und jetsonspezifische Hinweise im [NVIDIA Jetson-Leitfaden](linux/nvidia-jetson-guide.md).

#### Python-SDK (alle Plattformen)

Jeder Installer enthält ein passendes `chloros_sdk`-Wheel, sodass die SDK-Version immer mit der installierten GUI/CLI/dem Backend übereinstimmt. Unter Windows installiert der Installer es automatisch in Ihrem System-Python. Unter Linux installiert das `.deb` das SDK für das System-`python3` selbst (in `/usr/lib/python3/dist-packages`), sodass `import chloros_sdk` direkt nach der Installation funktioniert, ohne pip und ohne Netzwerk. Nur eine virtualenv, die die Systempakete nicht sieht, benötigt das gebündelte Wheel, das darin installiert werden muss:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Für Hosts, auf denen nur pip verfügbar ist (kein Chloros-Paket installiert; das SDK steuert dann ein Backend auf einem anderen Rechner an), ist das SDK ebenfalls auf PyPI verfügbar:

```bash
pip install chloros-sdk
```

Weitere Informationen finden Sie in der Dokumentation unter [API: Python-SDK](api-python-sdk.md) und in der [SDK-Referenz](reference/sdk-reference.md).

{% hint style="info" %}
**Linux-Benutzer**: Das Paket `.deb` installiert die CLI, das Backend und das Python-SDK. Für Linux gibt es keine grafische Benutzeroberfläche – die gesamte Interaktion erfolgt über die CLI oder das SDK.
{% endhint %}

***

## Weitere Ressourcen

### Python-SDK


Für Entwickler und Automatisierungsworkflows ist das Chloros-Python-SDK in jedem Installationsprogramm enthalten: Das Windows-Installationsprogramm installiert es in Ihrem Python, und das Linux-Paket `.deb` installiert es für das System `python3`. Auf einem Rechner ohne Chloros-Paket (der ein Backend auf einem anderen Rechner steuert) installieren Sie es über PyPI:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python-SDK](api-python-sdk.md)

**Voraussetzungen**: ein Chloros-Backend (das Windows-Installationsprogramm oder das Linux-Paket `.deb` installiert eines) sowie ein Chloros+-Login in einer kostenpflichtigen Stufe

***

## Was ist enthalten?

### Windows-Installationsprogramm

* ✅ **Chloros-GUI** – Grafische Benutzeroberfläche mit vollem Funktionsumfang
* ✅ **Chloros-CLI** – Befehlszeilenschnittstelle (erfordert eine Chloros+-Lizenz)
* ✅ **Chloros-Backend** – Verarbeitungs-Engine
* ✅ **Python SDK** – wird in Ihr Python installiert, sofern vorhanden (erfordert eine Chloros+-Lizenz)
* ✅ **Kameraprofile** – Vorkonfigurierte MAPIR-Kameravorlagen

### Linux-.deb-Paket

* ✅ **Chloros CLI** – Befehlszeilenschnittstelle (erfordert eine Chloros+-Lizenz)
* ✅ **Chloros-Backend** – Verarbeitungsengine
* ✅ **Python-SDK** – wird für das System `python3` installiert (erfordert eine Chloros+-Lizenz)
* ✅ **Kameraprofile** – Vorkonfigurierte MAPIR-Kameravorlagen
* ❌ Keine grafische Benutzeroberfläche – Linux läuft headless, nur CLI/SDK


### Python-SDK (PyPI, für Rechner ohne Chloros-Paket)

* ✅ **Chloros-SDK** – Python-API (erfordert eine Chloros+-Lizenz)

***

## Upgrade auf Chloros+

Schalten Sie mit einem Chloros+-Abonnement erweiterte Funktionen frei:

* 🚀 **Multithread-Verarbeitung** – Bilder parallel verarbeiten
* ⚡ **GPU-Beschleunigung (CUDA)** – Nutzen Sie die Leistung von NVIDIA-GPUs
* 💻 **Zugriff über die CLI** – Automatisieren Sie Prozesse mit Befehlszeilentools
* 🐍 **Python-SDK** – Programmatischer API-Zugriff
* 📱 **Mehrere Geräte** – Nutzung auf 2–10+ Geräten (abhängig vom Tarif)
* **🐻 Fortschrittliche texturbasierte Debayering-Methode** – ein hochwertiges, kantensensitives Debayering-Verfahren in Kombination mit einem KI-/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt.
* 🧮 **Benutzerdefinierte Formeln** – Erstellen Sie benutzerdefinierte Multispektralindizes

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ Tarife und Preise anzeigen</a></p>***

## Hilfe zur Installation

### Fehlerbehebung

**Die Installation schlägt mit folgender Fehlermeldung fehl:**

* Stellen Sie sicher, dass Sie über Administratorrechte verfügen
* Deaktivieren Sie vorübergehend Ihre Antivirensoftware
* Überprüfen Sie, ob die Mindestsystemanforderungen erfüllt sind

**Die Anwendung lässt sich nicht starten (Windows):**

* Vergewissern Sie sich, dass Windows 10/11 (64-Bit) installiert ist
* Aktualisieren Sie die Grafiktreiber
* Überprüfen Sie die Windows-Ereignisanzeige auf Fehlerdetails
* Wenden Sie sich mit den Fehlerprotokollen an den Support

**Die CLI-Schnittstelle lässt sich nicht starten (Linux):**

* Überprüfen Sie, ob das Paket „`.deb`“ korrekt installiert ist: `dpkg -l | grep chloros`
* Überprüfen Sie die Berechtigungen: `sudo chmod +x /usr/bin/chloros-cli`
* Führen Sie eine Diagnose durch: `chloros-cli selftest`
* Auf fehlende Bibliotheken prüfen: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (ab Version 1.3.0 ist `/usr/lib/chloros/chloros-backend` ein kleines Startskript; das Programm selbst befindet sich in `/usr/lib/chloros/backend/`)

**Probleme bei der Lizenzaktivierung:**

* Stellen Sie sicher, dass eine Internetverbindung besteht
* Überprüfen Sie die Anmeldedaten unter [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Stellen Sie sicher, dass die Firewall Chloros nicht blockiert* Detaillierte Anweisungen finden Sie unter [Chloros+ Login](chloros+-login.md)

### Support

Benötigen Sie Hilfe bei der Installation oder Einrichtung?

* 📧 **E-Mail**: info@mapir.camera
* 🌐 **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Erste Schritte](./)
* ❓ **FAQ**: [Häufig gestellte Fragen](faq.md)***

## Software-Updates

Chloros sucht nach Updates, meldet, wenn eine neue Version verfügbar ist, und verlinkt auf diese Download-Seite – Sie führen das Update durch, indem Sie das neue signierte Installationsprogramm ausführen. Ihre Einstellungen, Projekte und Anmeldedaten bleiben bei Updates erhalten. Unter Linux und auf Jetson prüft `chloros-cli update`, ob eine neuere Version verfügbar ist, und bietet an, die passende Version `.deb` herunterzuladen und zu installieren (dieser Befehl ist nur unter Linux verfügbar).

***

## Änderungsprotokoll**Version 1.3.0 (Aktuellste Version, 11. November 2026)**– die vollständige Liste finden Sie unter**Neuerungen in Chloros

1.3.0** auf der Seite [Erste Schritte](./).**Version 1.2.0**— siehe**Neuerungen in Chloros 1.2.0** auf der Seite [Erste Schritte](./).

<details>

<summary>Version 1.0.5</summary>

**Veröffentlichungsdatum: 10. Februar 2026**

**Neue Funktionen*** **Textur-bewusste Debayer-Methode \[Nur Chloros+] –** „Texture Aware“ verwendet einen hochwertigen, kantensensitiven Debayer in Kombination mit einem KI/ML-Rauschunterdrückungsmodell, das nahezu das gesamte Debayering-Rauschen entfernt.
* **Unterstützung für T4P-Kalibrierungsziele*** **Schnellere Chloros+ GPU-Verarbeitung, besseres Speichermanagement**

**Fehlerbehebungen*** Völlig neues Frontend (GUI), sollte nun auf allen Windows-Computern funktionieren.

</details>

<details>

<summary>Version 1.0.4</summary>

**Veröffentlichungsdatum: 5. Januar 2026**

**Neue Funktionen*** **Umschaltfunktion für Bilder/Metadaten**: Im Dateibrowser wurde eine Umschaltfunktion hinzugefügt, um die Metadaten des ausgewählten Bildes in einer Tabelle anstelle des Bildrasters anzuzeigen
* **Zoom-Schieberegler für das Bildraster**: Neuer UI-Schieberegler zum Anpassen der Miniaturbildgröße (unterstützt auch STRG + Mausrad)
* **Exportschaltflächen für das Bildraster**: Schaltflächen in der obersten Zeile zum Umschalten der Miniaturansichten von JPG auf verarbeitete Exporte (Targets, Reflectance, Index, LUT)
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

</details>

***

## Lizenzvereinbarung**Proprietäre Software** – Copyright (c) 2026 MAPIR Inc.

Die unbefugte Nutzung, Verbreitung oder Änderung ist untersagt.

**Kostenlose Version**: Für den privaten und gewerblichen Gebrauch mit Funktionseinschränkungen verfügbar**Chloros+**: Abonnementbasierte Lizenz für erweiterte Funktionen und den gewerblichen Einsatz
