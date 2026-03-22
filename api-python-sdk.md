# API : Python SDK

Das **Chloros Python SDK** bietet programmatischen Zugriff auf die Bildverarbeitungs-Engine Chloros und ermöglicht so Automatisierung, benutzerdefinierte Workflows sowie eine nahtlose Integration in Ihre Python-Anwendungen und Forschungs-Pipelines.

### Hauptmerkmale

* 🐍 **Natives Python** – Sauberes, Pythones API für die Bildverarbeitung
* 🔧 **Voller Zugriff auf API** – Vollständige Kontrolle über die Chloros-Verarbeitung
* 🚀 **Automatisierung** – Erstellen Sie benutzerdefinierte Workflows für die Stapelverarbeitung
* 🔗 **Integration** – Binden Sie Chloros in bestehende Python-Anwendungen ein
* 📊 **Forschungsbereit** – Perfekt für wissenschaftliche Analyse-Pipelines
* ⚡ **Parallelverarbeitung** – Skaliert entsprechend Ihrer CPU-Kernanzahl (Chloros+)

### Anforderungen

| Anforderung          | Details                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros installiert** | Windows: Desktop-Installationsprogramm; Linux: `.deb`-Paket                  |
| **Lizenz**          | Chloros+ ([kostenpflichtiger Tarif erforderlich](https://cloud.mapir.camera/pricing)) |
| **Betriebssystem** | Windows 10/11 (64-Bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 oder höher                                                |
| **Arbeitsspeicher**           | Mindestens 8 GB RAM (16 GB empfohlen)                                  |
| **Internet**         | Erforderlich für die Lizenzaktivierung                                     |

{% hint style="warning" %}
**Lizenzanforderungen**: Für den Zugriff auf Python SDK ist ein kostenpflichtiges Chloros+-Abonnement erforderlich. Standard-Tarife (kostenlos) bieten keinen Zugriff auf API/SDK. Besuchen Sie [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), um ein Upgrade durchzuführen.
{% endhint %}

## Schnellstart

### Installation

Installation über pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Erstkonfiguration**: Bevor Sie SDK verwenden, aktivieren Sie Ihre Chloros+-Lizenz, indem Sie Chloros, Chloros (Browser) oder Chloros CLI und melden Sie sich mit Ihren Zugangsdaten an. Dies muss nur einmal durchgeführt werden. Auf Linux (ohne GUI) verwenden Sie: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Grundlegende Verwendung

Verarbeiten Sie einen Ordner mit nur wenigen Zeilen:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Plattformübergreifende Pfade**: Die Code-Beispiele auf dieser Seite verwenden Pfade im Stil von Windows (z. B. `C:\\DroneImages\\Flight001`). Verwenden Sie unter Linux stattdessen Pfade im Format Linux (z. B. `/home/user/drone_images/flight001` oder `~/drone_images/flight001`). Der Befehl SDK funktioniert auf beiden Plattformen identisch.
{% endhint %}

### Vollständige Kontrolle

Für fortgeschrittene Workflows:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Installationsanleitung

### Voraussetzungen

Stellen Sie vor der Installation von SDK sicher, dass Sie Folgendes haben:

1. **Chloros installiert** — Windows: Desktop-Installationsprogramm ([Download](download.md)); Linux: `.deb`-Paket ([Linux-Installation](linux/linux-installation.md))
2. **Python 3.7+** installiert ([python.org](https://www.python.org))
3. **Aktive Chloros+-Lizenz** ([Upgrade](https://cloud.mapir.camera/pricing))

### Installation über pip

**Standardinstallation:**

```bash
pip install chloros-sdk
```

**Mit Unterstützung für Fortschrittsanzeige:**

```bash
pip install chloros-sdk[progress]
```

**Entwicklungsinstallation:**

```bash
pip install chloros-sdk[dev]
```

### Installation überprüfen

Testen Sie, ob SDK korrekt installiert ist:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Ersteinrichtung

### Lizenzaktivierung

SDK verwendet dieselbe Lizenz wie Chloros, Chloros (Browser) und Chloros CLI. Aktivieren Sie die Lizenz einmalig über die GUI oder CLI:**Windows:**Öffnen Sie**Chloros oder Chloros (Browser)** und melden Sie sich auf der Registerkarte „Benutzer“ <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> oder verwenden Sie das CLI.**Linux:** Verwenden Sie das CLI (keine GUI verfügbar):

```bash
chloros-cli login user@example.com 'your_password'
```

Die Lizenz wird lokal zwischengespeichert und bleibt auch nach einem Neustart erhalten.

{% hint style="success" %}
**Einmalige Einrichtung**: Nach der Anmeldung über die GUI oder CLI verwendet SDK automatisch die zwischengespeicherte Lizenz. Keine zusätzliche Authentifizierung erforderlich!
{% endhint %}

{% hint style="info" %}

****Abmelden**: SDK-Benutzer können zwischengespeicherte Anmeldedaten programmgesteuert mit der Methode `logout()` löschen. Siehe [logout()-Methode](#logout) in der API-Referenz.
{% endhint %}

### Verbindung testen

Überprüfen Sie, ob SDK eine Verbindung zu Chloros herstellen kann:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API-Referenz

### Klasse ChlorosLocal

Hauptklasse für die lokale Bildverarbeitung mit Chloros.

#### Konstruktor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parameter:**

| Parameter                 | Typ | Standardwert                   | Beschreibung                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL des lokalen Chloros-Backends          |
| `auto_start_backend`      | bool | `True`                    | Backend bei Bedarf automatisch starten |
| `backend_exe`             | str  | `None` (auto-Erkennung)      | Pfad zur ausführbaren Backend-Datei            |
| `timeout`                 | int  | `30`                      | Zeitlimit für Anfragen in Sekunden            |
| `backend_startup_timeout` | int  | `60`                      | Zeitlimit für den Start des Backends (Sekunden) |

**Beispiele:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Plattformübergreifende automatische Erkennung**: SDK versucht automatisch, den richtigen Backend-Pfad für Ihre Plattform zu ermitteln:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuell)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Methoden

#### `create_project(project_name, camera=None)`

Erstellen Sie ein neues Chloros-Projekt.

**Parameter:**

| Parameter      | Typ | Erforderlich | Beschreibung                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Ja      | Name für das Projekt                                     |
| `camera`       | str  | Nein       | Kameravorlage (z. B. „Survey3N\_RGN“, „Survey3W\_OCN“) |

**Rückgabewerte:** `dict` – Antwort auf die Projekterstellung**Beispiel:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Bilder aus einem Ordner importieren.

**Parameter:**

| Parameter     | Typ     | Erforderlich | Beschreibung                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Pfad | Ja      | Pfad zum Ordner mit Bildern         |
| `recursive`   | bool     | Nein       | Unterordner durchsuchen (Standard: False) |

**Rückgabewert:** `dict` – Importergebnisse mit Dateianzahl**Beispiel:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Verarbeitungseinstellungen konfigurieren.

**Parameter:**

| Parameter                 | Typ | Standardwert                 | Beschreibung                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | „Standard (schnell, mittlere Qualität)“ | Debayer-Methode            |
| `vignette_correction`     | bool | `True`                  | Vignettenkorrektur aktivieren      |
| `reflectance_calibration` | bool | `True`                  | Reflektanzkalibrierung aktivieren  |
| `indices`                 | list | `None`                  | Zu berechnende Vegetationsindizes |
| `export_format`           | str  | „TIFF (16-Bit)“         | Ausgabeformat                   |
| `ppk`                     | bool | `False`                 | PPK-Korrekturen aktivieren          |
| `custom_settings`         | dict | `None`                  | Erweiterte benutzerdefinierte Einstellungen        |

**Exportformate:**

* `"TIFF (16-bit)"` – Empfohlen für GIS/Fotogrammetrie
* `"TIFF (32-bit, Percent)"` – Wissenschaftliche Analyse
* `"PNG (8-bit)"` – Visuelle Inspektion
* `"JPG (8-bit)"` – Komprimierte Ausgabe

**Verfügbare Indizes:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 und weitere.**Beispiel:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Verarbeiten Sie die Projektbilder.

**Parameter:**

| Parameter           | Typ     | Standardwert      | Beschreibung                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Verarbeitungsmodus: „parallel“ oder „seriell“   |
| `wait`              | bool     | `True`       | Auf Abschluss warten                       |
| `progress_callback` | callable | `None`       | Fortschritts-Callback-Funktion(progress, msg) |
| `poll_interval`     | float    | `2.0`        | Abfrageintervall für den Fortschritt (Sekunden)   |

**Gibt zurück:** `dict` – Verarbeitungsergebnisse

{% hint style="warning" %}
**Parallelmodus**: Erfordert die Lizenz Chloros+. Skaliert automatisch auf Ihre CPU-Kerne (bis zu 16 Worker).
{% endhint %}

**Beispiel:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Aktuelle Projektkonfiguration abrufen.

**Rückgabewert:** `dict` – Aktuelle Projektkonfiguration**Beispiel:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Backend-Statusinformationen abrufen, einschließlich des Verarbeitungsfortschritts pro Thread.

**Rückgabewert:** `dict` – Backend-Status mit folgender Struktur:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Beispiel:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Beendet das Backend (sofern es mit SDK gestartet wurde).

**Beispiel:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Löscht zwischengespeicherte Anmeldedaten aus dem lokalen System.

**Beschreibung:**

Führt eine programmgesteuerte Abmeldung durch, indem zwischengespeicherte Anmeldedaten entfernt werden. Dies ist nützlich für:
* Das Wechseln zwischen verschiedenen Chloros+-Konten
* Das Löschen von Anmeldedaten in automatisierten Umgebungen
* Sicherheitszwecke (z. B. das Entfernen von Anmeldedaten vor der Deinstallation)

**Rückgabewert:** `dict` – Ergebnis des Abmeldevorgangs**Beispiel:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Neuauthentifizierung erforderlich**: Nach dem Aufruf von `logout()` müssen Sie sich erneut über Chloros, Chloros (Browser) oder Chloros CLI anmelden, bevor Sie SDK nutzen können.
{% endhint %}

***

### Komfortfunktionen

#### `process_folder(folder_path, **options)`

Einzeilige Komfortfunktion zur Bearbeitung eines Ordners.

**Parameter:**

| Parameter                 | Typ     | Standardwert         | Beschreibung                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Erforderlich        | Pfad zum Ordner mit Bildern     |
| `project_name`            | str      | Automatisch generiert  | Projektname                   |
| `camera`                  | str      | `None`          | Kameravorlage                |
| `indices`                 | list     | `["NDVI"]`      | Zu berechnende Indizes           |
| `vignette_correction`     | bool     | `True`          | Vignettierungskorrektur aktivieren     |
| `reflectance_calibration` | bool     | `True`          | Reflexionskalibrierung aktivieren |
| `export_format`           | str      | &quot;TIFF (16-Bit)&quot; | Ausgabeformat                  |
| `mode`                    | str      | `"parallel"`    | Verarbeitungsmodus                |
| `progress_callback`       | aufrufbar | `None`          | Fortschritts-Callback              |

**Rückgabewerte:** `dict` – Verarbeitungsergebnisse**Beispiel:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Unterstützung von Kontextmanagern

SDK unterstützt Kontextmanager für die automatische Bereinigung:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Vollständige Beispiele

{% hint style="info" %}
**Linux-Benutzer**: Alle folgenden Beispiele verwenden Windows-Pfade. Ersetzen Sie die `C:\\...`-Pfade durch Ihre Linux-Pfade (z. B. `/home/user/...` oder `~/...`). Alle SDK-Funktionen sind plattformübergreifend identisch.
{% endhint %}

### Beispiel 1: Grundlegende Verarbeitung

Verarbeiten Sie einen Ordner mit den Standardeinstellungen:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Beispiel 2: Benutzerdefinierter Workflow

Volle Kontrolle über die Verarbeitungs-Pipeline:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Beispiel 3: Stapelverarbeitung mehrerer Ordner

Verarbeiten Sie mehrere Flugdatensätze:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Beispiel 4: Integration in die Forschungspipeline

Integrieren Sie Chloros in die Datenanalyse:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Beispiel 5: Benutzerdefinierte Fortschrittsüberwachung

Erweiterte Fortschrittsverfolgung mit Protokollierung:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Beispiel 6: Fehlerbehandlung

Robuste Fehlerbehandlung für den produktiven Einsatz:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Beispiel 7: Kontoverwaltung und Abmeldung

Anmeldedaten programmgesteuert verwalten:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Beispiel 8: Befehlszeilentool

Erstellen Sie ein benutzerdefiniertes CLI-Tool mit dem SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Verwendung:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Ausnahmebehandlung

Das SDK stellt spezifische Ausnahmeklassen für verschiedene Fehlertypen bereit:

### Ausnahmehierarchie

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Ausnahmebeispiele

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Fortgeschrittene Themen

### Benutzerdefinierte Backend-Konfiguration

Verwenden Sie einen benutzerdefinierten Backend-Speicherort oder eine benutzerdefinierte Konfiguration:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Nicht blockierende Verarbeitung

Verarbeitung starten und mit anderen Aufgaben fortfahren:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Speicherverwaltung

Bei großen Datensätzen in Stapeln verarbeiten:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Fehlerbehebung

### Backend startet nicht

**Problem:** SDK kann das Backend nicht starten**Lösungen:**

1. Überprüfen Sie, ob Chloros installiert ist:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Überprüfen Sie die Firewall (Windows) oder die Portverfügbarkeit (Linux: `lsof -i :5000`)
3. Versuchen Sie es mit einem manuellen Backend-Pfad:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Lizenz nicht erkannt**Problem:** SDK warnt vor fehlender Lizenz**Lösungen:**

1. Öffnen Sie Chloros, Chloros (Browser) oder Chloros CLI und melden Sie sich an.
2. Überprüfen Sie, ob die Lizenz zwischengespeichert ist:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Bei Problemen mit den Anmeldedaten löschen Sie die zwischengespeicherten Anmeldedaten und melden Sie sich erneut an:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Wenden Sie sich an den Support: info@mapir.camera

***

### Importfehler**Problem:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Lösungen:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Zeitüberschreitung bei der Verarbeitung**Problem:** Zeitüberschreitung bei der Verarbeitung**Lösungen:**

1. Erhöhen Sie die Zeitüberschreitung:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Verarbeiten Sie kleinere Stapel
3. Überprüfen Sie den verfügbaren Speicherplatz
4. Überwachen Sie die Systemressourcen

***

### Port bereits belegt**Problem:** Backend-Port 5000 belegt**Lösungen:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Oder suchen und schließen Sie den konfliktverursachenden Prozess:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Leistungstipps

### Verarbeitungsgeschwindigkeit optimieren

1. **Parallelmodus verwenden** (erfordert Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Ausgabeauflösung reduzieren** (sofern akzeptabel)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Nicht benötigte Indizes deaktivieren**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Auf SSD verarbeiten** (nicht auf HDD)***

### Speicheroptimierung

Für große Datensätze:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Hintergrundverarbeitung

Python für andere Aufgaben freigeben:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Integrationsbeispiele

### Django-Integration

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## FAQ

### F: Benötigt SDK eine Internetverbindung?

**A:** Nur für die erstmalige Lizenzaktivierung. Nach der Anmeldung über Chloros, Chloros (Browser) oder Chloros CLI wird die Lizenz lokal zwischengespeichert und funktioniert 30 Tage lang offline.***

### F: Kann ich SDK auf einem Server ohne GUI verwenden?**A:** Ja! Die SDK funktioniert headless sowohl auf Windows- als auch auf Linux-Servern.**Linux (empfohlen für Headless):**
* Installation über das `.deb`-Paket
* Lizenz aktivieren: `chloros-cli login user@example.com 'password'`

**Windows-Server:**
* Windows Server 2016 oder höher
* Chloros installiert (einmalig)
* Lizenz aktiviert über CLI oder auf einem beliebigen Rechner

***

### F: Was ist der Unterschied zwischen Desktop, CLI und SDK?

| Funktion         | Desktop-GUI | CLI-Befehlszeile | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Schnittstelle**   | Point-and-Click | Befehlszeile          | Python API  |
| **Am besten geeignet für**    | Visuelle Arbeit | Skripting        | Integration |
| **Automatisierung**  | Eingeschränkt     | Gut             | Hervorragend   |
| **Flexibilität** | Grundlegend       | Gut             | Maximal     |
| **Lizenz**     | Chloros+    | Chloros+         | Chloros+    |***

### F: Kann ich mit SDK erstellte Apps vertreiben?**A:** Der Code von SDK kann in Ihre Anwendungen integriert werden, jedoch:

* Endbenutzer benötigen die Installation von Chloros
* Endbenutzer benötigen aktive Chloros+-Lizenzen
* Die kommerzielle Verbreitung erfordert eine OEM-Lizenzierung

Wenden Sie sich bei Fragen zur OEM-Lizenzierung an info@mapir.camera.

***

### F: Wie aktualisiere ich den SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### F: Wo werden die verarbeiteten Bilder gespeichert?

Standardmäßig im Projektpfad:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### F: Kann ich Bilder aus Python-Skripten verarbeiten, die nach einem Zeitplan ausgeführt werden?**A:** Ja! Verwenden Sie den Scheduler Ihres Betriebssystems mit Python-Skripten:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Planen Sie über den Taskplaner eine tägliche Ausführung.**Linux:** Planen Sie über Cron:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### F: Unterstützt SDK async/await?**A:** Die aktuelle Version ist synchron. Für asynchrones Verhalten verwenden Sie `wait=False` oder führen Sie den Prozess in einem separaten Thread aus:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### F: Wie wechsle ich zwischen verschiedenen Chloros+-Konten?**A:** Verwenden Sie die Methode `logout()`, um zwischengespeicherte Anmeldedaten zu löschen, und melden Sie sich dann mit dem neuen Konto erneut an:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Melden Sie sich nach dem Abmelden über die GUI, den Browser oder CLI mit dem neuen Konto an, bevor Sie SDK erneut verwenden.

***

## Hilfe

### Dokumentation

* **API-Referenz**: Diese Seite

### Supportkanäle

* **E-Mail**: info@mapir.camera
* **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Preise**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Beispielcode

Alle hier aufgeführten Beispiele sind getestet und produktionsreif. Kopieren Sie sie und passen Sie sie an Ihren Anwendungsfall an.

***

## Lizenz**Proprietäre Software** – Copyright (c) 2025 MAPIR Inc.

SDK erfordert ein aktives Chloros+-Abonnement. Die unbefugte Nutzung, Verbreitung oder Änderung ist untersagt.
