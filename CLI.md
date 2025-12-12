# CLI: Befehlszeile

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

Die **Chloros CLI** bietet leistungsstarken Befehlszeilenzugriff auf die Bildverarbeitungs-Engine Chloros und ermöglicht so die Automatisierung, Skripterstellung und den Headless-Betrieb für Ihre Bildverarbeitungs-Workflows.

### Hauptmerkmale

* 🚀 **Automatisierung** – Skriptbasierte Stapelverarbeitung mehrerer Datensätze
* 🔗 **Integration** – Einbettung in bestehende Workflows und Pipelines
* 💻 **Headless-Betrieb** – Ausführung ohne GUI
* 🌍 **Mehrsprachigkeit** – Unterstützung für 38 Sprachen
* ⚡ **Parallelverarbeitung** – Dynamische Skalierung an Ihre CPU (bis zu 16 parallele Worker)

### Anforderungen

| Anforderung          | Details                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Betriebssystem** | Windows 10/11 (64-Bit)                                              |
| **Lizenz**          | Chloros+ ([kostenpflichtiger Tarif erforderlich](https://cloud.mapir.camera/pricing)) |
| **Arbeitsspeicher**           | Mindestens 8 GB RAM (16 GB empfohlen)                                  |
| **Internet**         | Für die Lizenzaktivierung erforderlich                                     |
| **Festplattenspeicher**       | Variiert je nach Projektgröße                                              |

{% Hinweis style=&quot;warning&quot; %}
**Lizenzanforderungen**: Für CLI ist ein kostenpflichtiges Chloros+-Abonnement erforderlich. Standard-Tarife (kostenlos) haben keinen Zugriff auf CLI. Besuchen Sie [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), um ein Upgrade durchzuführen.
{% endhint %}

## Schnellstart

### Installation

Das CLI ist automatisch im Chloros-Installationsprogramm enthalten:

1. Laden Sie das **Chloros-Installationsprogramm.exe** herunter und führen Sie es aus.
2. Führen Sie den Installationsassistenten aus.
3. CLI installiert unter: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Das Installationsprogramm fügt `chloros-cli` automatisch zu Ihrem System-PATH hinzu. Starten Sie Ihr Terminal nach der Installation neu.
{% endhint %}

### Erstmalige Einrichtung

Bevor Sie CLI verwenden, aktivieren Sie Ihre Chloros+-Lizenz:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Grundlegende Verwendung

Verarbeiten Sie einen Ordner mit den Standardeinstellungen:

```powershell
chloros-cli process "C:\Images\Dataset001"
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

**Beispiel:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Optionen für Verarbeitungsbefehle

| Option                | Typ    | Standard        | Beschreibung                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Pfad    | _Erforderlich_     | Ordner mit RAW/JPG-Multispektralbildern                                         |
| `-o, --output`        | Pfad    | Wie Eingabe  | Ausgabeordner für verarbeitete Bilder                                                     |
| `-n, --project-name`  | Zeichenfolge  | Automatisch generiert | Benutzerdefinierter Projektname                                                                    |
| `--vignette`          | Flag    | Aktiviert        | Vignettenkorrektur aktivieren                                                             |
| `--no-vignette`       | Flag    | -              | Vignettenkorrektur deaktivieren                                                            |
| `--reflectance`       | Flag    | Aktiviert        | Reflektionskalibrierung aktivieren                                                         |
| `--no-reflectance`    | Flag    | -              | Reflektionskalibrierung deaktivieren                                                        |
| `--ppk`               | Flag    | Deaktiviert       | PPK-Korrekturen aus .daq-Lichtsensordaten anwenden                                      |
| `--format`            | Auswahl  | TIFF (16-Bit)  | Ausgabeformat: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Ganzzahl | Auto           | Mindestzielgröße in Pixeln für die Erkennung des Kalibrierungsfelds                          |
| `--target-clustering` | Ganzzahl | Auto           | Schwellenwert für die Zielclusterung (0–100)                                                    |
| `--exposure-pin-1`    | Zeichenfolge  | Keine           | Belichtung für Kameramodell sperren (Pin 1)                                                 |
| `--exposure-pin-2`    | Zeichenfolge  | Keine           | Belichtung für Kameramodell sperren (Pin 2)                                                 |
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

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Sonderzeichen**: Verwenden Sie einfache Anführungszeichen um Passwörter, die Zeichen wie `$`, `!` oder Leerzeichen enthalten.
{% endhint %}

**Ausgabe:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` – Anmeldedaten löschen

Löschen Sie gespeicherte Anmeldedaten und melden Sie sich von Ihrem Konto ab.

**Syntax:**

```bash
chloros-cli logout
```

**Beispiel:**

```powershell
chloros-cli logout
```

**Ausgabe:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` – Lizenzstatus überprüfen

Zeigt den aktuellen Lizenz- und Authentifizierungsstatus an.

**Syntax:**

```bash
chloros-cli status
```

**Beispiel:**

```powershell
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

### `export-status` – Exportfortschritt überprüfen

Überwachen Sie den Exportfortschritt von Thread 4 während oder nach der Verarbeitung.

**Syntax:**

```bash
chloros-cli export-status
```

**Beispiel:**

```powershell
chloros-cli export-status
```

**Anwendungsfall:** Rufen Sie diesen Befehl während der Verarbeitung auf, um den Exportfortschritt zu überprüfen.

***

### `language` – Verwaltung der Sprache der Benutzeroberfläche

Anzeigen oder Ändern der Sprache der Benutzeroberfläche CLI.

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

```powershell
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

| Code    | Sprache              | Name in der Landessprache      |
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
| `nl`    | Niederländisch                 | Nederlands       |
| `ar`    | Arabisch                | العربية          |
| `pl`    | Polnisch                | Polski           |
| `tr`    | Türkisch               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesisch            | Bahasa Indonesia |
| `vi`    | Vietnamesisch            | Tiếng Việt       |
| `th`    | Thai                  | ไทย              |
| `sv`    | Schwedisch               | Svenska          |
| `da`    | Dänisch                | Dansk            |
| `no`    | Norwegisch             | Norsk            |
| `fi`    | Finnisch               | Suomi            |
| `el`    | Griechisch                 | Ελληνικά         |
| `cs`    | Tschechisch                 | Čeština          |
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

{% hint style=&quot;success&quot; %}
**Automatische Speicherung**: Ihre Spracheinstellung wird unter `~/.chloros/cli_language.json` gespeichert und bleibt über alle Sitzungen hinweg erhalten.
{% endhint %}

***

### `set-project-folder` – Standardprojektordner festlegen

Ändern Sie den Speicherort des Standardprojektordners (gemeinsam mit GUI).

**Syntax:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Beispiel:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` – Projektordner anzeigen

Zeigt den aktuellen Standardprojektordner an.

**Syntax:**

```bash
chloros-cli get-project-folder
```

**Beispiel:**

```powershell
chloros-cli get-project-folder
```

**Ausgabe:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` – Auf Standard zurücksetzen

Setzt den Projektordner auf den Standardpfad zurück.

**Syntax:**

```bash
chloros-cli reset-project-folder
```

***

## Globale Optionen

Diese Optionen gelten für alle Befehle:

| Option          | Typ    | Standard       | Beschreibung                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Pfad    | Automatisch erkannt | Pfad zur ausführbaren Backend-Datei                       |
| `--port`        | Ganzzahl | 5000          | Backend-Portnummer API                          |
| `--restart`     | Flag    | -             | Neustart des Backends erzwingen (beendet vorhandene Prozesse) |
| `--version`     | Flag    | -             | Versionsinformationen anzeigen und beenden                |
| `--help`        | Flag    | -             | Hilfeinformationen anzeigen und beenden                   |

**Beispiel mit globalen Optionen:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Anleitung zu den Verarbeitungseinstellungen

### Parallele Verarbeitung

Chloros+ CLI **skaliert automatisch** die parallele Verarbeitung entsprechend den Fähigkeiten Ihres Computers:

**So funktioniert es:**

* Erkennt Ihre CPU-Kerne und Ihren Arbeitsspeicher
* Weist Worker zu: **2× CPU-Kerne** (nutzt Hyperthreading)
* **Maximal: 16 parallele Worker** (für Stabilität)

**Systemstufen:**

| Systemtyp   | CPU        | RAM      | Worker  | Leistung     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-End**  | 16+ Kerne  | 32+ GB   | Bis zu 16 | Maximale Geschwindigkeit   |
| **Mittelklasse** | 8–15 Kerne | 16–31 GB | 8–16     | Ausgezeichnete Geschwindigkeit |
| **Einstiegsklasse**   | 4–7 Kerne  | 8–15 GB  | 4–8      | Gute Geschwindigkeit      |

{% hint style=&quot;success&quot; %}
**Automatische Optimierung**: Der CLI erkennt automatisch Ihre Systemspezifikationen und konfiguriert die optimale Parallelverarbeitung. Keine manuelle Konfiguration erforderlich!
{% endhint %}

### Debayer-Methoden

Der CLI verwendet **High Quality (Faster)** als standardmäßigen und empfohlenen Debayer-Algorithmus:

| Methode                      | Qualität | Geschwindigkeit | Beschreibung                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Hohe Qualität (schneller)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Kantensensitiver Algorithmus (Standard, empfohlen) |

### Vignettierungskorrektur

**Funktion:** Korrigiert den Lichtabfall an den Bildrändern (dunklere Ecken, die häufig bei Kameraaufnahmen auftreten).

* **Standardmäßig aktiviert** – Die meisten Benutzer sollten diese Option aktiviert lassen.
* Verwenden Sie `--no-vignette`, um sie zu deaktivieren.

{% hint style=&quot;success&quot; %}
**Empfehlung**: Aktivieren Sie immer die Vignettenkorrektur, um eine gleichmäßige Helligkeit im gesamten Bild zu gewährleisten.
{% endhint %}

### Reflektionskalibrierung

Konvertiert rohe Sensorwerte mithilfe von Kalibrierungsfeldern in standardisierte Reflektionsprozentsätze.

* **Standardmäßig aktiviert** – Unverzichtbar für die Vegetationsanalyse.
* Erfordert Kalibrierungszieltafeln in Bildern.
* Verwenden Sie `--no-reflectance`, um diese Funktion zu deaktivieren.

{% hint style=&quot;info&quot; %}
**Anforderungen**: Stellen Sie sicher, dass die Kalibrierungsfelder in Ihren Bildern richtig belichtet und sichtbar sind, um eine genaue Reflektionsumwandlung zu gewährleisten.
{% endhint %}

### PPK-Korrekturen

**Funktion:** Wendet nachbearbeitete kinematische Korrekturen unter Verwendung von DAQ-A-SD-Protokolldaten an, um die GPS-Genauigkeit zu verbessern.

* **Standardmäßig deaktiviert**
* Verwenden Sie `--ppk`, um die Funktion zu aktivieren.
* Erfordert .daq-Dateien im Projektordner von MAPIR DAQ-A-SD-Lichtsensor.

### Ausgabeformate

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bittiefe</th><th width="116.5999755859375">Dateigröße</th><th>Am besten geeignet für</th></tr></thead><tbody><tr><td><strong>TIFF (16 Bit)</strong> ⭐</td><td>16-Bit-Ganzzahl</td><td>Groß</td><td>GIS-Analyse, Photogrammetrie (empfohlen)</td></tr><tr><td><strong>TIFF (32-Bit, Prozent)</strong></td><td>32-Bit-Gleitkomma</td><td>Sehr groß</td><td>Wissenschaftliche Analyse, Forschung</td></tr><tr><td><strong>PNG (8 Bit)</strong></td><td>8-Bit-Ganzzahl</td><td>Mittel</td><td>Visuelle Inspektion, Web-Freigabe</td></tr><tr><td><strong>JPG (8-Bit)</strong></td><td>8-Bit-Ganzzahl</td><td>Klein</td><td>Schnellvorschau, komprimierte Ausgabe</td></tr></tbody></table>***

## Automatisierung und Skripting

### PowerShell-Stapelverarbeitung

Automatische Verarbeitung mehrerer Datensatzordner:

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

### Windows-Stapelscript

Einfache Schleife für die Stapelverarbeitung:

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

### Python Automatisierungsskript

Erweiterte Automatisierung mit Fehlerbehandlung:

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

## Verarbeitungsworkflow

### Standard-Workflow

1. **Eingabe**: Ordner mit RAW/JPG-Bildpaaren
2. **Erkennung**: CLI sucht automatisch nach unterstützten Bilddateien
3. **Verarbeitung**: Der Parallelmodus skaliert entsprechend Ihrer CPU-Kerne (Chloros+)
4. **Ausgabe**: Erstellt Unterordner für Kameramodelle mit verarbeiteten Bildern

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

### Geschätzte Verarbeitungszeit

Typische Verarbeitungszeiten für 100 Bilder (jeweils 12 MP):

| Modus              | Zeit      | Hardware                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Parallelmodus** | 5–10 Min.  | i7/Ryzen 7, 16 GB RAM, SSD (bis zu 16 Worker) |
| **Parallelmodus** | 10–15 Min. | i5/Ryzen 5, 8 GB RAM, HDD (bis zu 8 Worker)   |

{% hint style=&quot;info&quot; %}
**Leistungstipp**: Die Bearbeitungszeit hängt von der Anzahl der Bilder, der Auflösung und den Computerspezifikationen ab.
{% endhint %}

***

## Fehlerbehebung

### CLI nicht gefunden

**Fehler:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Lösungen:**

1. Überprüfen Sie den Installationsort:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Verwenden Sie den vollständigen Pfad, wenn er nicht in PATH enthalten ist:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Fügen Sie ihn manuell zu PATH hinzu:
   * Öffnen Sie Systemeinstellungen → Umgebungsvariablen.
   * Bearbeiten Sie die Variable PATH.
   * Fügen Sie hinzu: `C:\Program Files\Chloros\resources\cli`
   * Terminal neu starten

***

### Backend konnte nicht gestartet werden

**Fehler:**

```
Backend failed to start within 30 seconds
```

**Lösungen:**

1. Überprüfen Sie, ob das Backend bereits ausgeführt wird (schließen Sie es zuerst)
2. Überprüfen Sie, ob die Firewall Windows nicht blockiert
3. Versuchen Sie es mit einem anderen Port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Erzwingen Sie einen Neustart des Backends:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Probleme mit der Lizenz/Authentifizierung

**Fehler:**

```
Chloros+ license required for CLI access
```

**Lösungen:**

1. Überprüfen Sie, ob Sie über ein aktives Chloros+-Abonnement verfügen.
2. Melden Sie sich mit Ihren Anmeldedaten an:

```powershell
chloros-cli login user@example.com 'password'
```

3. Überprüfen Sie den Lizenzstatus:

```powershell
chloros-cli status
```

4. Wenden Sie sich an den Support: info@mapir.camera

***

### Keine Bilder gefunden

**Fehler:**

```
No images found in the specified folder
```

**Lösungen:**

1. Überprüfen Sie, ob der Ordner unterstützte Formate enthält (.RAW, .TIF, .JPG).
2. Überprüfen Sie, ob der Ordnerpfad korrekt ist (verwenden Sie Anführungszeichen für Pfade mit Leerzeichen).
3. Stellen Sie sicher, dass Sie Leserechte für den Ordner haben.
4. Überprüfen Sie, ob die Dateierweiterungen korrekt sind.

***

### Verarbeitung stockt oder hängt

**Lösungen:**

1. Überprüfen Sie den verfügbaren Speicherplatz (stellen Sie sicher, dass genügend Speicherplatz für die Ausgabe vorhanden ist).
2. Schließen Sie andere Anwendungen, um Speicherplatz freizugeben.
3. Reduzieren Sie die Anzahl der Bilder (verarbeiten Sie sie in Stapeln).

***

### Port bereits in Verwendung

**Fehler:**

```
Port 5000 is already in use
```

**Lösung:**

Geben Sie einen anderen Port an:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## FAQ

### F: Benötige ich eine Lizenz für CLI?

**A:** Ja! Für CLI ist eine kostenpflichtige **Chloros+-Lizenz** erforderlich.

* ❌ Standard-Tarif (kostenlos): CLI deaktiviert
* ✅ Chloros+ (kostenpflichtige) Tarife: CLI vollständig aktiviert

Abonnieren Sie unter: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### F: Kann ich CLI auf einem Server ohne GUI verwenden?

**A:** Ja! CLI läuft vollständig headless. Anforderungen:

* Windows Server 2016 oder höher
* Visual C++ Redistributable installiert
* Ausreichend RAM (mindestens 8 GB, 16 GB empfohlen)
* Einmalige Aktivierung der GUI-Lizenz auf einem beliebigen Rechner

***

### F: Wo werden die verarbeiteten Bilder gespeichert?

**A:** Standardmäßig werden die verarbeiteten Bilder im **gleichen Ordner wie die Eingabe** in Unterordnern des Kameramodells (z. B. `Survey3N_RGN/`) gespeichert.

Verwenden Sie die Option `-o`, um einen anderen Ausgabeordner anzugeben:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### F: Kann ich mehrere Ordner gleichzeitig verarbeiten?

**A:** Nicht direkt mit einem Befehl, aber Sie können Skripte verwenden, um Ordner nacheinander zu verarbeiten. Siehe Abschnitt [Automatisierung und Skripterstellung](CLI.md#automation--scripting).

***

### F: Wie speichere ich die Ausgabe von CLI in einer Protokolldatei?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### F: Was passiert, wenn ich während der Verarbeitung Strg+C drücke?

**A:** CLI wird:

1. Die Verarbeitung ordnungsgemäß beenden
2. Das Backend herunterfahren
3. Mit dem Code 130 beenden

Teilweise verarbeitete Bilder können im Ausgabeordner verbleiben.

***

### F: Kann ich die Verarbeitung von CLI automatisieren?

**A:** Auf jeden Fall! CLI ist für die Automatisierung ausgelegt. Beispiele für PowerShell, Batch und Python finden Sie unter [Automatisierung und Skripting](CLI.md#automation--scripting).

***

### F: Wie kann ich die Version von CLI überprüfen?

**A:**

```powershell
chloros-cli --version
```

**Ausgabe:**

```
Chloros CLI 1.0.2
```

***

## Hilfe erhalten

### Befehlszeilenhilfe

Zeigen Sie die Hilfeinformationen direkt in CLI an:

```powershell
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Support-Kanäle

* **E-Mail**: info@mapir.camera
* **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Preise**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Vollständige Beispiele

### Beispiel 1: Grundlegende Verarbeitung

Verarbeitung mit Standardeinstellungen (Vignette, Reflexion):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Beispiel 2: Hochwertige wissenschaftliche Ausgabe

32-Bit-Float TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Beispiel 3: Schnelle Vorschauverarbeitung

8-Bit PNG ohne Kalibrierung für schnelle Überprüfung:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Beispiel 4: PPK-korrigierte Verarbeitung

Anwendung von PPK-Korrekturen mit Reflexion:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Beispiel 5: Benutzerdefinierter Ausgabeort

Verarbeitung auf einem anderen Laufwerk mit spezifischem Format:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Beispiel 6: Authentifizierungs-Workflow

Authentifizierungsablauf abschließen:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Beispiel 7: Mehrsprachige Verwendung

Sprache der Benutzeroberfläche ändern:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
