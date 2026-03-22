# CLI: Befehlszeile

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>Das **Chloros CLI** bietet leistungsstarken Befehlszeilenzugriff auf die Bildverarbeitungs-Engine Chloros und ermöglicht so Automatisierung, Skripterstellung und den Headless-Betrieb für Ihre Bildverarbeitungs-Workflows.

### Hauptmerkmale

* 🚀 **Automatisierung** – Skripten Sie die Stapelverarbeitung mehrerer Datensätze
* 🔗 **Integration** – Einbindung in bestehende Workflows und Pipelines
* 💻 **Headless-Betrieb** – Ausführung ohne GUI
* 🌍 **Mehrsprachigkeit** – Unterstützung für 38 Sprachen
* ⚡ **Parallelverarbeitung** – [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) optimiert automatisch für Ihre Hardware

### Anforderungen

| Anforderung          | Details                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Betriebssystem** | Windows 10/11 (64-Bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Lizenz**          | Chloros+ ([kostenpflichtiger Tarif erforderlich](https://cloud.mapir.camera/pricing)) |
| **Arbeitsspeicher**           | Mindestens 8 GB RAM (16 GB empfohlen)                                  |
| **Internet**         | Für die Lizenzaktivierung erforderlich                                     |
| **Speicherplatz**       | Variiert je nach Projektgröße                                              |

{% hint style="warning" %}
**Lizenzanforderung**: Für CLI ist ein kostenpflichtiges Chloros+-Abonnement erforderlich. Standard-Tarife (kostenlos) bieten keinen Zugriff auf CLI. Besuchen Sie [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), um ein Upgrade durchzuführen.
{% endhint %}

## Schnellstart

### Installation

#### Windows

Das CLI ist automatisch im Chloros-Installationsprogramm enthalten:

1. Laden Sie den **Chloros Installer.exe** herunter und führen Sie ihn aus.
2. Führen Sie den Installationsassistenten durch.
3. CLI wird installiert unter: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Das Installationsprogramm fügt `chloros-cli` automatisch zum System-PATH hinzu. Starten Sie Ihr Terminal nach der Installation neu.
{% endhint %}

#### Linux

Installieren Sie das `.deb`-Paket für Ihre Architektur:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Ausführliche Informationen zur Einrichtung von Linux finden Sie unter [Linux Installation](linux/linux-installation.md).

### Ersteinrichtung

Bevor Sie CLI verwenden, aktivieren Sie Ihre Chloros+-Lizenz:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Grundlegende Verwendung

Verarbeiten Sie einen Ordner mit den Standardeinstellungen:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Befehlsreferenz

### Allgemeine Syntax

```
chloros-cli [global-options] <command> [command-options]
```

***

## Befehle

### `process` – Bilder verarbeiten

Bilder in einem Ordner mit Kalibrierung verarbeiten.

**Syntax:**

```bash
chloros-cli process <input-folder> [options]
```

**Beispiele:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Optionen des Verarbeitungsbefehls

| Option                | Typ    | Standard        | Beschreibung                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Pfad    | _Erforderlich_     | Ordner mit RAW-/JPG-Multispektralbildern                                         |
| `-o, --output`        | Pfad    | Wie Eingabe  | Ausgabeordner für verarbeitete Bilder                                                     |
| `-n, --project-name`  | Zeichenfolge  | Automatisch generiert | Benutzerdefinierter Projektname                                                                    |
| `--vignette`          | Flag    | Aktiviert        | Vignettierungskorrektur aktivieren                                                             |
| `--no-vignette`       | Flag    | -              | Vignettierungskorrektur deaktivieren                                                            |
| `--reflectance`       | Flag    | Aktiviert        | Reflektanzkalibrierung aktivieren                                                         |
| `--no-reflectance`    | Flag    | -              | Reflektanzkalibrierung deaktivieren                                                        |
| `--ppk`               | Flag    | Deaktiviert       | PPK-Korrekturen aus .daq-Lichtsensordaten anwenden                                      |
| `--format`            | Auswahl  | TIFF (16-Bit)  | Ausgabeformat: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Ganzzahl | Auto           | Minimale Zielgröße in Pixeln für die Erkennung des Kalibrierungsfeldes                          |
| `--target-clustering` | Ganzzahl | Auto           | Schwellenwert für Zielclusterung (0–100)                                                    |
| `--debayer`           | Auswahl  | `standard`     | Debayer-Methode: `standard` oder `texture-aware` (nur Chloros+)                          |
| `--target`, `--targets` | Flag  | Deaktiviert       | Suche nur in einem Unterordner „target“ oder „targets“ nach Kalibrierungszielen (beschleunigt die Verarbeitung) |
| `--indices`           | Liste    | Keine           | Zu berechnende Vegetationsindizes (z. B. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Zeichenfolge  | Keine           | Belichtung für Kameramodell sperren (Pin 1)                                                 |
| `--exposure-pin-2`    | Zeichenkette  | Keine           | Belichtung für Kameramodell sperren (Pin 2)                                                 |
| `--recal-interval`    | Ganzzahl | Auto           | Neukalibrierungsintervall in Sekunden                                                      |
| `--timezone-offset`   | Ganzzahl | 0              | Zeitzonenversatz in Stunden                                                               |

***

### `login` – Konto authentifizieren

Melden Sie sich mit Ihren Chloros+-Anmeldedaten an, um die CLI-Verarbeitung zu aktivieren.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

**Beispiel:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Sonderzeichen**: Setzen Sie Passwörter, die Zeichen wie `$`, `!` oder Leerzeichen enthalten, in einfache Anführungszeichen.
{% endhint %}

**Ausgabe:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` – Anmeldedaten löschen

Löscht gespeicherte Anmeldedaten und meldet Sie von Ihrem Konto ab.

**Syntax:**

```bash
chloros-cli logout
```

**Beispiel:**

```bash
chloros-cli logout
```

**Ausgabe:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK-Benutzer**: Das Python SDK bietet auch eine programmatische `logout()`-Methode zum Löschen von Anmeldedaten innerhalb von Python-Skripten. Weitere Informationen finden Sie in der [Python SDK-Dokumentation](api-python-sdk.md#logout).
{% endhint %}

***

### `status` – Lizenzstatus prüfen

Zeigt den aktuellen Lizenz- und Authentifizierungsstatus an.

**Syntax:**

```bash
chloros-cli status
```

**Beispiel:**

```bash
chloros-cli status
```

**Ausgabe:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` – Exportfortschritt prüfen

Überwachen Sie den Exportfortschritt von Thread 4 während oder nach der Verarbeitung.

**Syntax:**

```bash
chloros-cli export-status
```

**Beispiel:**

```bash
chloros-cli export-status
```

**Anwendungsfall:** Rufen Sie diesen Befehl während der laufenden Verarbeitung auf, um den Exportfortschritt zu überprüfen.***

### `language` – Sprache der Benutzeroberfläche verwalten

Zeigen Sie die Sprache der Benutzeroberfläche von CLI an oder ändern Sie sie.

**Syntax:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Beispiele:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Unterstützte Sprachen (insgesamt 38)

| Code    | Sprache              | Name in der Originalsprache      |
| ------- | --------------------- | ---------------- |
| `en`    | Englisch               | English          |
| `es`    | Spanisch               | Español          |
| `pt`    | Portugiesisch            | Português        |
| `fr`    | Französisch                | Français         |
| `de`    | Deutsch                | Deutsch          |
| `it`    | Italienisch               | Italiano         |
| `ja`    | Japanisch              | 日本語              |
| `ko`    | Koreanisch                | 한국어              |
| `zh`    | Chinesisch (vereinfacht)  | 简体中文             |
| `zh-TW` | Chinesisch (traditionell) | 繁體中文             |
| `ru`    | Russisch               | Русский          |
| `nl`    | Niederländisch                | Nederlands       |
| `ar`    | Arabisch                | العربية          |
| `pl`    | Polnisch                | Polski           |
| `tr`    | Türkisch               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesisch            | Bahasa Indonesia |
| `vi`    | Vietnamesisch            | Tiếng Việt       |
| `th`    | Thailändisch                  | ไทย              |
| `sv`    | Schwedisch               | Svenska          |
| `da`    | Dänisch                | Dansk            |
| `no`    | Norwegisch             | Norsk            |
| `fi`    | Finnisch               | Suomi            |
| `el`    | Griechisch                 | Ελληνικά         |
| `cs`    | Tschechisch                | Čeština          |
| `hu`    | Ungarisch             | Magyar           |
| `ro`    | Rumänisch              | Română           |
| `uk`    | Ukrainisch             | Українська       |
| `pt-BR` | Brasilianisches Portugiesisch  | Português Brasileiro |
| `zh-HK` | Kantonesisch             | 粵語             |
| `ms`    | Malaiisch                 | Bahasa Melayu    |
| `sk`    | Slowakisch                | Slovenčina       |
| `bg`    | Bulgarisch             | Български        |
| `hr`    | Kroatisch              | Hrvatski         |
| `lt`    | Litauisch            | Lietuvių         |
| `lv`    | Lettisch               | Latviešu         |
| `et`    | Estnisch              | Eesti            |
| `sl`    | Slowenisch             | Slovenščina      |

{% hint style="success" %}
**Automatische Speicherung**: Ihre Spracheinstellung wird in `~/.chloros/cli_language.json` gespeichert und bleibt über alle Sitzungen hinweg erhalten.
{% endhint %}

***

### `set-project-folder` – Standard-Projektordner festlegen

Ändern Sie den Speicherort des Standard-Projektordners (gemeinsam mit der GUI unter Windows).

**Syntax:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Beispiele:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` – Projektordner anzeigen

Zeigt den aktuellen Speicherort des Standard-Projektordners an.

**Syntax:**

```bash
chloros-cli get-project-folder
```

**Beispiel:**

```bash
chloros-cli get-project-folder
```

**Ausgabe:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` – Auf Standard zurücksetzen

Setzt den Projektordner auf den Standardpfad zurück.

**Syntax:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` – Systemdiagnose ausführen

Führt 7 Diagnosetests durch, um Ihre Systemkonfiguration zu überprüfen.

**Syntax:**

```bash
chloros-cli selftest
```

**Durchgeführte Diagnosen:**

1. Versionsprüfung
2. Portverfügbarkeit (5000)
3. Backend-Start
4. API-Konnektivitätstest
5. Systeminformationen und GPU-Erkennung
6. Überprüfung der Denoiser-Modelle
7. CUDA-Verfügbarkeitsprüfung

{% hint style="info" %}
**Nützlich für die Fehlerbehebung**: Führen Sie nach der Installation `selftest` aus, um zu überprüfen, ob Ihr System korrekt konfiguriert ist, insbesondere auf Linux/Jetson, wo die GPU- und CUDA-Einstellungen möglicherweise überprüft werden müssen.
{% endhint %}

***

### `update` – Nach Updates suchen (nur Linux)

Suchen Sie nach Updates für CLI auf Linux-Systemen und installieren Sie diese.

**Syntax:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Option    | Beschreibung                        |
| --------- | ---------------------------------- |
| `--check` | Nur nach Updates suchen, nicht installieren |

{% hint style="info" %}
Dieser Befehl ist nur auf Linux verfügbar. Auf Windows werden Updates über das Installationsprogramm bereitgestellt.
{% endhint %}

***

## Globale Optionen

Diese Optionen gelten für alle Befehle:

| Option            | Typ    | Standard       | Beschreibung                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Pfad    | Automatisch erkannt | Pfad zur ausführbaren Backend-Datei                       |
| `--port`          | Ganzzahl | 5000          | Portnummer des Backends API                          |
| `--restart`       | Flag    | -             | Neustart des Backends erzwingen (beendet laufende Prozesse) |
| `--version`       | Flag    | -             | Versionsinformationen anzeigen und beenden                |
| `--help`          | Flag    | -             | Hilfeinformationen anzeigen und beenden                   |

{% hint style="info" %}
**Automatische Erkennung des Backends**: Der Pfad `--backend-exe` wird je nach Plattform automatisch erkannt:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuell)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Beispiel mit globalen Optionen:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Leitfaden zu den Verarbeitungseinstellungen

### Parallele Verarbeitung &amp; Dynamische Rechenanpassung

Chloros 1.1.0 enthält [Dynamische Rechenanpassung](processing-architecture/dynamic-compute-adaptation.md) – die Verarbeitungs-Engine **erkennt automatisch Ihre Hardware** und wählt die optimale Strategie aus:

| Plattform | Strategie | Worker | Pipeline | Anmerkungen |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Speichereffizient, serialisiert |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Parallele GPU-Verarbeitung |
| **Desktop mit 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Gute Desktop-Leistung |
| **Desktop mit 12 GB+ GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimale Desktop-Leistung |
| **Reines CPU-System** | `CPU_PARALLEL` | Kerne – 1 | `cpu_fallback` | Keine GPU erforderlich |

{% hint style="success" %}
**Keine manuelle Konfiguration erforderlich!** Chloros erkennt Ihre CPU, GPU, RAM und (auf Jetson) die Temperatursensoren automatisch und konfiguriert dann automatisch die optimale Verarbeitungs-Pipeline.
{% endhint %}

### Debayer-Methoden

| Methode | CLI-Flag | Qualität | Geschwindigkeit | Lizenz |
| --- | --- | --- | --- | --- |
| **Standard (schnell, mittlere Qualität)** | `--debayer standard` | Gut | Schnell | Kostenlos / Chloros+ |
| **Texture Aware (Langsam, höchste Qualität)** | `--debayer texture-aware` | Höchste | Langsam | Nur Chloros+ |

Die Standard-Debayer-Methode ist **Standard**. Die Methode**Texturbewusst** verwendet ein KI/ML-Rauschunterdrückungsmodell für die höchste Ausgabequalität, erfordert jedoch eine Chloros+-Lizenz und eine NVIDIA-GPU.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Vignettenkorrektur

**Funktionsweise:** Korrigiert den Lichtabfall an den Bildrändern (dunkle Ecken, wie sie bei Kameraaufnahmen häufig vorkommen).

* **Standardmäßig aktiviert** – Die meisten Benutzer sollten diese Option aktiviert lassen
* Verwenden Sie `--no-vignette`, um sie zu deaktivieren

{% hint style="success" %}
**Empfehlung**: Aktivieren Sie die Vignettenkorrektur immer, um eine gleichmäßige Helligkeit im gesamten Bild zu gewährleisten.
{% endhint %}

### Reflektanzkalibrierung

Konvertiert rohe Sensorwerte mithilfe von Kalibrierungsfeldern in standardisierte Reflektanzprozentsätze.

* **Standardmäßig aktiviert** – Unverzichtbar für die Vegetationsanalyse
* Erfordert Kalibrierungszielfelder in den Bildern
* Verwenden Sie `--no-reflectance`, um die Funktion zu deaktivieren

{% hint style="info" %}
**Voraussetzungen**: Stellen Sie sicher, dass die Kalibrierungsfelder in Ihren Bildern richtig belichtet und sichtbar sind, um eine genaue Reflektanzumwandlung zu gewährleisten.
{% endhint %}

### PPK-Korrekturen

**Funktionsweise:** Wendet kinematische Korrekturen nach der Verarbeitung unter Verwendung von DAQ-A-SD-Protokolldaten an, um die GPS-Genauigkeit zu verbessern.

* **Standardmäßig deaktiviert**
* Verwenden Sie `--ppk` zum Aktivieren
* Erfordert .daq-Dateien im Projektordner vom MAPIR DAQ-A-SD-Lichtsensor.

### Ausgabeformate

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bittiefe</th><th width="116.5999755859375">Dateigröße</th><th>Am besten geeignet für</th></tr></thead><tbody><tr><td><strong>TIFF (16-Bit)</strong> ⭐</td><td>16-Bit-Ganzzahl</td><td>Groß</td><td>GIS-Analyse, Photogrammetrie (empfohlen)</td></tr><tr><td><strong>TIFF (32-Bit, Prozent)</strong></td><td>32-Bit-Gleitkomma</td><td>Sehr groß</td><td>Wissenschaftliche Analyse, Forschung</td></tr><tr><td><strong>PNG (8-Bit)</strong></td><td>8-Bit-Ganzzahl</td><td>Mittel</td><td>Sichtprüfung, Web-Freigabe</td></tr><tr><td><strong>JPG (8-Bit)</strong></td><td>8-Bit-Ganzzahl</td><td>Klein</td><td>Schnellvorschau, komprimierte Ausgabe</td></tr></tbody></table>***

## Automatisierung &amp; Skripterstellung

### PowerShell-Stapelverarbeitung (Windows)

Automatische Verarbeitung mehrerer Datensatzordner auf Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows-Batch-Skript (Windows)

Einfache Schleife für die Stapelverarbeitung auf Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash-Batch-Verarbeitung (Linux)

Verarbeitung mehrerer Datensatzordner auf Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python-Automatisierungsskript (plattformübergreifend)

Erweiterte Automatisierung mit Fehlerbehandlung (funktioniert auf Windows und Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Verarbeitungsablauf

### Standard-Workflow

1. **Eingabe**: Ordner mit RAW/JPG-Bildpaaren
2. **Erkennung**: CLI sucht automatisch nach unterstützten Bilddateien
3. **Verarbeitung**: Der Parallelmodus skaliert entsprechend der Anzahl Ihrer CPU-Kerne (Chloros+)
4. **Ausgabe**: Erstellt Unterordner nach Kameramodell mit den verarbeiteten Bildern

### Beispiel für die Ausgabestruktur

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Geschätzte Verarbeitungszeiten

Typische Verarbeitungszeiten für 100 Bilder (jeweils 12 MP):

| Plattform | Modus | Geschätzte Zeit | Anmerkungen |
| --- | --- | --- | --- |
| **Desktop mit 12 GB+ GPU** | `GPU_PARALLEL` | 5–10 Min. | Schnellste Option |
| **Desktop mit 8 GB GPU** | `GPU_SINGLE` | 10–15 Min. | Gute Leistung |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15–25 Min. | Edge-Computing |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30–60 Min. | Speicherbeschränkt |
| **Nur CPU** | `CPU_PARALLEL` | 20–40 Min. | Keine GPU erforderlich |

{% hint style="info" %}
**Leistungstipp**: Die Verarbeitungszeit hängt von der Anzahl der Bilder, der Auflösung, der Debayer-Methode und der Hardware ab. „Texture Aware Debayer“ dauert deutlich länger als „Standard“. Weitere Informationen finden Sie unter [Dynamische Rechenanpassung](processing-architecture/dynamic-compute-adaptation.md).
{% endhint %}

***

## Fehlerbehebung

### CLI nicht gefunden

**Windows Fehler:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Lösungen:**

1. Überprüfen Sie den Installationsort:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Verwenden Sie den vollständigen Pfad, wenn er nicht in PATH enthalten ist:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Fügen Sie ihn manuell zu PATH hinzu:
   * Öffnen Sie „Systemeigenschaften“ → „Umgebungsvariablen“
   * Bearbeiten Sie die Variable PATH
   * Fügen Sie hinzu: `C:\Program Files\Chloros\resources\cli`
   * Terminal neu starten

**Linux Fehler:**

```
chloros-cli: command not found
```

**Linux Lösungen:**

1. Installation überprüfen:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Laden Sie Ihre Shell neu:

```bash
source ~/.bashrc
```

3. Überprüfen Sie die Berechtigungen:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Backend konnte nicht gestartet werden**Fehler:**

```

Backend failed to start within 30 seconds
```

**Lösungen:**

1. Prüfen Sie, ob das Backend bereits läuft (schließen Sie es zuerst)
2. Prüfen Sie, ob die Firewall den Datenverkehr blockiert (Windows) oder prüfen Sie die Portverfügbarkeit (Linux: `lsof -i :5000`)
3. Versuchen Sie es mit einem anderen Port:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Backend neu starten:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Bei Linux prüfen, ob die Backend-Ausführungsdatei vorhanden ist:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Probleme mit der Lizenz / Authentifizierung**Fehler:**

```

Chloros+ license required for CLI access
```

**Lösungen:**

1. Überprüfen Sie, ob Sie über ein aktives Chloros+-Abonnement verfügen
2. Melden Sie sich mit Ihren Anmeldedaten an:

```bash
chloros-cli login user@example.com 'password'
```

3. Überprüfen Sie den Lizenzstatus:

```bash
chloros-cli status
```

4. Wenden Sie sich an den Support: info@mapir.camera

***

### Keine Bilder gefunden**Fehler:**

```

No images found in the specified folder
```

**Lösungen:**

1. Überprüfen Sie, ob der Ordner unterstützte Formate enthält (.RAW, .TIF, .JPG)
2. Überprüfen Sie, ob der Ordnerpfad korrekt ist (verwenden Sie Anführungszeichen für Pfade mit Leerzeichen)
3. Stellen Sie sicher, dass Sie Leserechte für den Ordner haben
4. Überprüfen Sie, ob die Dateiendungen korrekt sind

***

### Verarbeitung stockt oder hängt sich auf**Lösungen:**

1. Überprüfen Sie den verfügbaren Speicherplatz (stellen Sie sicher, dass genügend Platz für die Ausgabe vorhanden ist)
2. Schließen Sie andere Anwendungen, um Speicherplatz freizugeben
3. Reduzieren Sie die Anzahl der Bilder (verarbeiten Sie in Stapeln)

***

### Port bereits in Gebrauch**Fehler:**

```

Port 5000 is already in use
```

**Lösungen:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## FAQ

### F: Benötige ich eine Lizenz für CLI?

**A:**Ja! Für CLI ist eine kostenpflichtige**Chloros+-Lizenz** erforderlich.

* ❌ Standard-Tarif (kostenlos): CLI deaktiviert
* ✅ Chloros+ (kostenpflichtiger) Tarif: CLI vollständig aktiviert

Abonnieren unter: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### F: Kann ich CLI auf einem Server ohne GUI verwenden?**A:** Ja! CLI läuft vollständig headless. Dies ist der primäre Anwendungsfall bei Linux.**Windows-Server:**
* Windows Server 2016 oder höher
* Visual C++ Redistributable installiert

**Linux-Server:**
* Ubuntu 20.04+ / Debian 11+ (amd64) oder JetPack 6 (arm64)
* Installation über das `.deb`-Paket

**Beide Plattformen:**
* Mindestens 8 GB RAM (16 GB empfohlen)
* Einmalige Lizenzaktivierung: `chloros-cli login user@example.com 'password'`

***

### F: Wo werden die verarbeiteten Bilder gespeichert?**A:**Standardmäßig werden die verarbeiteten Bilder im**selben Ordner wie die Eingabedateien** in Unterordnern nach Kameramodell gespeichert (z. B. `Survey3N_RGN/`).

Verwenden Sie die Option `-o`, um einen anderen Ausgabeordner anzugeben:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### F: Kann ich mehrere Ordner gleichzeitig bearbeiten?**A:** Nicht direkt mit einem einzigen Befehl, aber Sie können Skripte verwenden, um Ordner nacheinander zu verarbeiten. Siehe Abschnitt [Automatisierung &amp; Skripting](CLI.md#automation--scripting).***

### F: Wie speichere ich die CLI-Ausgabe in einer Protokolldatei?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### F: Was passiert, wenn ich während der Verarbeitung Strg+C drücke?**A:** Das CLI wird:

1. Die Verarbeitung ordnungsgemäß beenden
2. Das Backend herunterfahren
3. Mit Code 130 beenden

Teilweise verarbeitete Bilder können im Ausgabeordner verbleiben.

***

### F: Kann ich die Verarbeitung mit CLI automatisieren?**A:** Auf jeden Fall! CLI ist für die Automatisierung ausgelegt. Siehe [Automatisierung &amp; Skripting](CLI.md#automation--scripting) für PowerShell (Windows), Batch (Windows), Bash (Linux) und Python (plattformübergreifend).***

### F: Wie überprüfe ich die Version von CLI?**A:**

```bash
chloros-cli --version
```

**Ausgabe:**

```

Chloros CLI 1.1.0
```

***

## Hilfe

### Hilfe über die Befehlszeile

Rufen Sie die Hilfeinformationen direkt in CLI auf:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Supportkanäle

* **E-Mail**: info@mapir.camera
* **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Preise**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Vollständige Beispiele

### Beispiel 1: Grundlegende Bearbeitung

Bearbeitung mit Standardeinstellungen (Vignette, Reflexion):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Beispiel 2: Hochwertige wissenschaftliche Ausgabe

32-Bit-Float TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Beispiel 3: Schnelle Vorschauverarbeitung

8-Bit-PNG ohne Kalibrierung für die schnelle Überprüfung:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Beispiel 4: PPK-korrigierte Verarbeitung

PPK-Korrekturen mit Reflexionsgrad anwenden:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Beispiel 5: Benutzerdefinierter Ausgabeort

Verarbeitung an einem anderen Ort mit spezifischem Format:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Beispiel 6: Authentifizierungsablauf

Vollständiger Authentifizierungsablauf (auf allen Plattformen identisch):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Beispiel 7: Mehrsprachige Nutzung

Ändern der Sprache der Benutzeroberfläche (auf allen Plattformen identisch):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
