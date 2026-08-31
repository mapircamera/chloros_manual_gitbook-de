# Aufnahmeeinstellungen und -modi

Die Aufnahme auf der Registerkarte „Kameras“ erfolgt über eine rote Schaltfläche **„Alles aufnehmen“**und einen Bereich**„Aufnahmeeinstellungen“**, in dem festgelegt wird, was diese Schaltfläche bewirkt: welche Kameras beteiligt sind, welche Exportformate die einzelnen Kameras speichern und ob der Auslöser einmalig, kontinuierlich oder in bestimmten Intervallen betätigt wird. Diese Seite dokumentiert den gesamten Ablauf – die Konfiguration, die Aufnahme selbst, wo die Dateien auf der Festplatte gespeichert werden und wie sie später zu kalibrierten Produkten weiterverarbeitet werden können. Die Steuerelemente für die Kameras und das Array selbst befinden sich unter [Kameraeinstellungen](camera-settings.md).

{% hint style="info" %}
**Für Aufnahmen muss ein Projekt geöffnet sein.** Die Schaltfläche „Alle aufnehmen“ und das Zahnrad-Symbol für die Aufnahmeeinstellungen sind deaktiviert, bis ein Projekt geöffnet ist („Erstellen oder öffnen Sie ein Projekt, um Aufnahmen zu speichern“). Jede Aufnahme wird im Projektordner unter `captures/` gespeichert.
{% endhint %}

## Der Bereich „Aufnahmeeinstellungen“

Öffnen Sie ihn über das **Zahnrad neben „Alle aufnehmen“**in der Kameraliste der Seitenleiste oder über die Schaltfläche**„Aufnahmeeinstellungen öffnen…“** am unteren Rand eines beliebigen kameraspezifischen Einstellungsbereichs. Die Überschrift lautet „Aufnahmeeinstellungen“ und enthält eine ←-Zurück-Schaltfläche.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Ihre hier getroffenen Auswahlen – eingeschlossene Kameras, Kontrollkästchen pro Typ und der Aufnahmemodus – werden **pro Projekt** gespeichert und beim erneuten Öffnen des Projekts wiederhergestellt.

### Aufnahmemodi

Drei Modus-Schaltflächen am oberen Rand des Fensters:

| Modus | Funktion | Untereinstellungen (Standardwerte) |
| --- | --- | --- |
| **Einzelaufnahme** *(Standard)* | Eine Aufnahme über alle ausgewählten Kameras hinweg. | — |
| **Serienaufnahme**| Aufeinanderfolgende Aufnahmen bis zum Erreichen einer Stoppbedingung. | Stopp nach**Anzahl der Aufnahmen** (Standard: 1) *oder* **Aufnahmedauer** (Standard: 10 s; Einheiten: Sekunden / Minuten / Stunden / Tage). |
| **Intervall**(Zeitraffer) | Serienaufnahmen nach Zeitplan. |**Aufnahmen / Intervall**(Standard 1) ·**Alle**N Einheiten (Standard 5 s) ·**Für** N Einheiten (Standard 1 m). |

Im Serien- oder Intervallmodus wird die Schaltfläche „Alle aufnehmen“ während der Ausführung zur Schaltfläche **„Stopp (N)“**, wobei die Aufnahmen gezählt werden, sobald sie eintreffen.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Auswahl von Kameras und Exporttypen

Der Hilfetext des Fensters fasst es zusammen: Wählen Sie aus, welche Kameras und Exporttypen „Capture All“ erzeugt – standardmäßig ist alles aktiviert, und die Auswahl wird mit diesem Projekt gespeichert.

* Die Schaltflächen **„Alle auswählen“ / „Keine auswählen“** schalten die Kontrollkästchen aller Kameras auf einmal um.
* **Umschaltflächen für Massenexporttypen**(zwei Reihen mit Schaltflächen):**Alle Raw / Alle debayert / Alle Vorschau / Alle Radiance / Alle Reflektanz / Alle Index**. Jede Schaltfläche verfügt über drei Zustände: grün ✓ = aktiviert für jede Kamera, die diesen Typ unterstützt; gelb – = für einige aktiviert; grau = für keine. Eine Schaltfläche ist deaktiviert, wenn keine angeschlossene Kamera diesen Typ unterstützt. Alle Schaltflächen sind ausgegraut, solange „Fastest Capture“ aktiviert ist.
* **Zeilen pro Kamera**: ein Kontrollkästchen zum Einbeziehen sowie eine erweiterbare (▸/▾) Liste der für diese Kamera verfügbaren Exporttypen mit einzelnen Kontrollkästchen. Die Zeile zeigt einen Zählstand wie „4/6“ an.

### Exporttypen und welche Kameras sie unterstützen

Es gibt sechs Exporttypen: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. In der Zeile der jeweiligen Kamera werden nur die zutreffenden angezeigt:

| Exporttyp | Inhalt | RGB (FRGB) | Bayer-Multispektral (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-Mosaik (mono: das einzelne Band) direkt vom Sensor | ✓ | ✓ | ✓ |
| **Debayered** | Lineares Demosaicing (mono: 1-Kanal-Graustufen) | ✓ | ✓ | ✓ |
| **Vorschau** | Vollständige Bildkette (Weißabgleich + Gamma gemäß Kameraprofil; multispektral: Falschfarben-Streckung) | ✓ | ✓ | ✓ |
| **Strahlungsintensität** | float32 W/m²/sr/nm über die vollständige radiometrische Kette | — (nicht verfügbar) | ✓ | ✓ |
| **Reflexionsgrad** | uint16 ρ (32768 = 1,0) | — (nicht verfügbar) | ✓ — wird nur angezeigt, wenn die Kamera über einen DAQ-Lichtsensor verfügt (eigenen oder vom Array geerbten) | wie bei Multispektral |
| **Index** | Vegetationsindex (LUT) | — | ✓ — erfordert einen aktivierten, nicht leeren Indexausdruck auf der Kamera und wird für Mitglieder eines kombinierten Arrays nicht angeboten (das Array verfügt über einen gemeinsamen Index) | — (ein Index benötigt ≥2 Bänder; siehe [Mono-Kameras &amp; Vegetationsindizes](mono-indices.md)) |

Strahlungsintensität und Reflexionsgrad werden für RGB-Kameras niemals angeboten – die Strahlungsintensität pro Bayer-Pixel ist für einen breitbandigen photometrischen Sensor nicht aussagekräftig.

### Schnellste Aufnahme

Der Schalter **⚡ Schnellste Aufnahme – nur RAW**(orange, wenn aktiviert) überschreibt alle Exportauswahlen auf**nur RAW** – plus eine kostenlose Combined-Index-Komposition für Arrays –, sodass das Bild so schnell wie möglich gespeichert wird: Die Berechnungen für Radiance, Reflektanz und Anzeige werden zum Zeitpunkt der Aufnahme vollständig übersprungen.

{% hint style="info" %}
**Ein `.daq` wird weiterhin gespeichert.** Wenn ein Lichtsensor zugewiesen ist, schreibt „Schnellste Erfassung“ weiterhin den DAQ-Downwelling-Messwert neben die Rohbilder – so können Strahlungsintensität, Reflektanz und Indexprodukte später durch Nachbearbeitung erstellt werden (siehe [Nachbearbeitung von Aufnahmen](#re-processing-captures-into-calibrated-products)). „Fastest Capture“ wirkt sich zudem nicht destruktiv auf Ihre Auswahl in den Kontrollkästchen aus: Schalten Sie die Funktion aus, und die Auswahl wird wiederhergestellt.
{% endhint %}

### Steuerelemente pro Array

Jedes angeschlossene Array erhält im Fenster eine eigene Gruppenkarte:

* **Kontrollkästchen „Einbeziehen“** (drei Zustände für alle Elemente) und den Namen des Arrays mit seinem Anzeigemodus: „(kombiniert | separat)“.
* Kontrollkästchen **Ausgerichtet**(standardmäßig**aktiviert**): Passt die Exporte der Elemente an das Ausrichtungsprofil des Arrays an, sodass die Exporte über alle Kameras hinweg pixelgenau registriert sind. Die Rohdaten bleiben unverzerrt, enthalten die Transformation jedoch in ihren Metadaten. (Das Profil selbst wird im [Fenster „Array-Einstellungen“](camera-settings.md#alignment-co-registration-combined-only) berechnet.)
* Die Kamerareihen der Elemente sind innerhalb der Karte verschachtelt.

Die Array-Karte beherbergt außerdem zwei Recorder. Man kann sie als **Überwachung vs. Analyse** betrachten:

| Recorder | Grade | Was wird aufgezeichnet? |
| --- | --- | --- |
| **● Index-Video aufzeichnen / ■ Aufzeichnung beenden** *(nur bei kombinierten Arrays)* | **Überwachung** | Das Live-Composite des kombinierten Index als Video mit 10 fps – 8 Bit, Vorschauauflösung, LUT integriert. Erfordert ein geöffnetes Projekt und eine Live-Streaming-Ansicht. Zeigt während der Aufnahme die Anzahl der Bilder und die verstrichene Zeit an. |
| **⦿ Rohdaten-Burst aufzeichnen / ■ Rohdaten-Burst stoppen** *(beliebiges Array)* | **Analyse**| Rohdaten-Bayer-Bilder mit der Live-Aufnahmerate (ohne Verarbeitung) sowie ein Manifest pro Bild und `.daq`-Messwerte, gespeichert als `captures/bursts/`. Nach einer Serienaufnahme erscheint eine Schaltfläche**Video erstellen**: Sie verarbeitet die Serienaufnahme offline zu kalibriertem Video – kombinierter Index und/oder Radianz/Reflexionsgrad/Index pro Kamera – sowie optionale TIFFs. Die Erstellung des kombinierten Indexes startet automatisch, sobald Sie die Serienaufnahme stoppen.

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## Der „Capture All“-Ablauf

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

Klicken Sie in der Kameraliste der Seitenleiste auf **„Capture All“**:

1. Jede enthaltene, sichtbare und nicht angehaltene Kamera nimmt mit den jeweils ausgewählten Exporttypen auf. **Arrays werden als ein synchronisierter Auslöser ausgelöst** (eine einzige synchronisierte Gruppe über alle Mitglieder hinweg – siehe [Multi-Kamera-Arrays](arrays.md)); Einzelkameras nehmen individuell auf.
2. Ausgeblendete (mit dem Augensymbol markierte) oder angehaltene Kameras werden übersprungen. Ein Array wird nur dann vollständig blockiert, wenn *alle* seine Mitglieder ausgeblendet oder angehalten sind.
3. Immer wenn ein Lichtsensor zugewiesen ist, wird der entsprechende DAQ-Messwert für die nach unten gerichtete Strahlung als `.daq`-Datei zusammen mit den Bilddaten gespeichert – selbst bei reinen Rohdatenaufnahmen –, sodass radiometrische Produkte später jederzeit abgeleitet werden können.
4. Die Schaltfläche zeigt den aktuellen Fortschritt an – „Aufnahme… fertig/insgesamt“ – und wird im Dauer-/Intervallmodus zu **Stopp (N)**. Für jeden Aufnahmevorgang gilt ein Timeout von 300 s.
5. Nach Abschluss des Durchgangs meldet eine Ergebnisanzeige **„N Dateien gespeichert“**oder**„N gespeichert, F fehlgeschlagen“** sowie „(S ausgeblendet/angepauste übersprungen)“, wenn Kameras übersprungen wurden.

## Speicherort der Aufnahmen

Die Aufnahmen werden im geöffneten Projekt unter `<project>/captures/` gespeichert. Jeder Exporttyp wird in einem **eigenen Unterordner** abgelegt, sodass sich bei einer mehrstufigen Aufnahme die Typen nie vermischen:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` ist der Zeitstempel der Aufnahme und `<serial>` die Seriennummer der Kamera. Einzelaufnahmen erhalten den Namen `capture_<ts>_SN<serial>_<level>`; Array-Aufnahmen von einem synchronisierten Auslöser werden als `sync_<ts>_SN<serial>_<level>` bezeichnet und **teilen sich einen gemeinsamen Zeitstempel für alle Kameras in der Gruppe** (das Ebenen-Suffix entfällt, wenn eine Kamera nur eine einzige Ebene speichert).
* **Eine Besonderheit, die man beachten sollte:** Die Anzeigeebene wird in einem Ordner mit dem Namen `preview/` gespeichert, während die Dateien den Namen `_display` beibehalten – Ordner und Suffix unterscheiden sich nur bei dieser Ebene.
* Unbekannte Ebenen werden in einem Ordner mit ihrem eigenen Namen abgelegt; kann kein Unterordner erstellt werden, wird die Datei in das Stammverzeichnis der Aufnahmen geschrieben, anstatt verloren zu gehen.
* Capture-TIFFs werden standardmäßig verlustfrei (DEFLATE) komprimiert und enthalten ihre vollständigen Kalibrierungs- und Verarbeitungsmetadaten **innerhalb der XMP-Datei** – die Aufnahmen sind selbsterklärend und benötigen keine zusätzlichen Sidecar-Dateien außer der `.daq`-Datei.

Dies entspricht demselben Layout, mit dem `chloros-cli lattice capture` / `array-capture` in ihr `-o`-Verzeichnis schreiben – dokumentiert im [CLI-Referenz § Wie ein Aufnahmordner aussieht](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Nachbearbeitung der Aufnahmen zu kalibrierten Produkten

Die erfassten Rohbilder sowie die gespeicherte Datei „`.daq`“ sind alles, was die Verarbeitungspipeline benötigt – deshalb ist „Fastest Capture“ für den produktiven Einsatz geeignet.

* **GUI**: Fügen Sie den Capture-Ordner zu einem Projekt hinzu ([Dateien zu einem Projekt hinzufügen](../processing-images-gui/adding-files-to-a-project.md)) und verarbeiten Sie die Daten wie gewohnt.
* **CLI**: Verweisen Sie `process` auf das**Stammverzeichnis der Aufnahmen**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importiert normalerweise nur den von Ihnen angegebenen Ordner, doch wenn dieser Ordner keine Bilder enthält und Unterordner aufweist, durchläuft das Programm diese automatisch – sodass die Unterordner der einzelnen Ebenen und die Dateien im Stammverzeichnis `.daq` in einem Durchgang erfasst werden. Jede Erfassung wird als **einzelnes Bild** importiert, wobei die anderen Ebenen als anzeigbare Modi angehängt sind, nicht als ein Bild pro Ebene.

Die direkte Benennung eines Unterordners einer Ebene (z. B. `…/captures/raw/`) funktioniert ebenfalls, lässt jedoch die Stammdateien `.daq` zurück – kopieren Sie diese zusätzlich, wenn Sie-ein radiometrisches Produkt aus „`raw/`“ ableiten; andernfalls kann der Zeitstempelabgleich nicht zugeordnet werden.

{% hint style="warning" %}
**Die Verarbeitung beginnt immer mit `raw`.**Innerhalb jeder Erfassung ist das Rohbild die Quelle der Pipeline; `debayered`, `radiance`, `reflectance` und `preview` dienen als Anzeigemodi, werden jedoch nie wieder in die Pipeline zurückgeführt – eine erneute Verarbeitung eines abgeleiteten Produkts würde die bereits in seine Pixel eingebackenen Vignettierungs-, Farb- und Strahlungsberechnungen erneut anwenden, weshalb Chloros abgelehnt wird, anstatt eine doppelte Verarbeitung durchzuführen. Die Renderings `index/` und `composite/` werden überhaupt nicht verarbeitet (es handelt sich um Ausgabedateien, nicht um Aufnahmen). Ein „captures“-Ordner, der**ohne** RAW-Importe gespeichert wurde, wird normal angezeigt, aber `process` überspringt ihn und gibt dies an; `--input-level {raw,debayered,processed}` ist die absichtliche Ausweichmöglichkeit, die einen Einstiegspunkt erzwingt. Die genauen Überspringungsmeldungen finden Sie in der [CLI-Referenz](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Zwei weitere Verhaltensweisen, die Sie bei der Skripterstellung für die Nachbearbeitung kennen sollten:

* Ein `chloros-cli process`-Lauf, der Produkte angefordert hat, aber **keine Bildprodukte**geschrieben hat, schlägt mit einer Fehlermeldung fehl und wird mit einem Ergebnis ungleich Null beendet** – es kommt niemals zu einem stillen, leeren Lauf. Erfolgreiche Läufe melden ihre Produktanzahl. (Ein absichtlich nur auf Metadaten beschränkter Lauf gilt ebenfalls als Erfolg.)
* Neu importierte, bereits verarbeitete Exporte belegen niemals den Rohdatenslot einer Erfassung – die ursprünglichen Rohdaten bleiben immer die Quelle der Pipeline.

## CLI-Äquivalente

Alles auf dieser Seite kann im Headless-Modus ausgeführt werden. Die GUI-Erfassungsmodi entsprechen direkt den `chloros-cli lattice array-capture`-Modi:

| GUI | CLI |
| --- | --- |
| Einzel | `chloros-cli lattice array-capture` |
| Kontinuierlich | `array-capture --continuous [--count N] [--duration S]` |
| Intervall | `array-capture --interval S [--duration S]` |
| Schnellste Aufzeichnung | `array-capture --fastest` |
| Kontrollkästchen „Ausgerichtet“ | `--aligned / --no-aligned` |
| Kontrollkästchen für Exporttyp | `--processing LEVEL` oder `--levels L1,L2,…` (Standard: `all`) |
| Video mit Index aufzeichnen | `chloros-cli lattice array-record` |
| Rohdaten-Serienaufnahme aufzeichnen / Video erstellen | `chloros-cli lattice array-burst` / `array-build-video` |

Die vollständigen Flag-Tabellen, die Option „Smart-AE Settled-Capture“ (`--smart`) und das Modell mit konstanter Aufnahmerate sind im Abschnitt [CLI Referenz § Aufnahmemodi, Rekorder und Offline-Nachbearbeitung](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
