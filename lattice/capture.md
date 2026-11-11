# Aufnahmeeinstellungen und -modi

Die Aufnahme auf der Registerkarte „Kameras“ erfolgt über eine rote Schaltfläche **„Alle aufnehmen“**und einen Bereich**„Aufnahmeeinstellungen“**, in dem festgelegt wird, was diese Schaltfläche bewirkt: welche Kameras beteiligt sind, welche Exportformate die einzelnen Kameras speichern und ob der Auslöser einmalig, kontinuierlich oder in bestimmten Intervallen betätigt wird. Diese Seite dokumentiert den gesamten Ablauf – die Konfiguration, die Aufnahme selbst, wo die Dateien auf der Festplatte gespeichert werden und wie sie später zu kalibrierten Produkten weiterverarbeitet werden können. Die Steuerelemente für Kameras und Arrays selbst befinden sich unter [Kameraeinstellungen](camera-settings.md).

{% hint style="info" %}
**Für Aufnahmen muss ein Projekt geöffnet sein.** „Alle aufnehmen“ und das Zahnrad-Symbol für die Aufnahmeeinstellungen sind deaktiviert, bis ein Projekt geöffnet ist („Erstellen oder öffnen Sie ein Projekt, um Aufnahmen zu speichern“). Jede Aufnahme wird im Projektordner unter `captures/` gespeichert.
{% endhint %}

## Der Bereich „Aufnahmeeinstellungen“

Öffnen Sie ihn über das **Zahnrad neben „Alles aufnehmen“**in der Kameraliste der Seitenleiste oder über die Schaltfläche**„Aufnahmeeinstellungen öffnen…“** am unteren Rand eines beliebigen kameraspezifischen Einstellungsbereichs. Die Überschrift lautet „Aufnahmeeinstellungen“ und enthält eine ←-Zurück-Schaltfläche.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Ihre hier getroffenen Auswahlen – eingeschlossene Kameras, Kontrollkästchen pro Typ und der Aufnahmemodus – werden **pro Projekt** gespeichert und beim erneuten Öffnen des Projekts wiederhergestellt.

### Aufnahmemodi

Drei Modus-Schaltflächen am oberen Rand des Fensters:

| Modus | Funktion | Untereinstellungen (Standardwerte) |
| --- | --- | --- |
| **Einzelaufnahme** *(Standard)* | Eine Aufnahme über alle ausgewählten Kameras hinweg. | — |
| **Serienaufnahme**| Aufeinanderfolgende Aufnahmen bis zum Eintreten einer Stoppbedingung. | Stopp nach**Anzahl der Aufnahmen** (Standard 1) *oder* **Aufnahmedauer** (Standard 10 s; Einheiten: Sekunden / Minuten / Stunden / Tage). |
| **Intervall**(Zeitraffer) | Serienaufnahmen nach Zeitplan. |**Aufnahmen / Intervall**(Standard 1) ·**Alle**N Einheiten (Standard 5 s) ·**Für** N Einheiten (Standard 1 m). |

Im Modus „Kontinuierlich“ oder „Intervall“ wird die Schaltfläche „Alle aufnehmen“ während der Ausführung zur Schaltfläche **„Stopp (N)“**, wobei die Aufnahmen bei ihrer Erfassung gezählt werden. Ein volles Laufwerk stoppt den Lauf (siehe [Der „Capture All“-Ablauf](#the-capture-all-flow)).**Der Serienmodus bei Arrays läuft im Backend (neu in 1.3.0).** Wenn jede Kamera in der Liste zu einem Array gehört (Einzelkameras sind ausgeblendet oder ausgeschlossen), sendet der „Continuous“-Modus nicht mehr eine Anfrage pro Erfassung: Das Chloros

-Backend führt die Erfassungsschleife selbst aus, schreibt eine synchronisierte Gruppe, während es die nächste auswählt, und die Schaltfläche fragt deren Zählstand ab. Befinden sich Einzelkameras in der Liste, erfasst Chloros

wie bisher Pass für Pass.

{% hint style="warning" %}
**Schnelle Erfassung unter Windows

: Schließen Sie das Chloros

-Backend von Microsoft Defender aus.** Das Echtzeit-Scannen jeder Datei, die bei einer Erfassung mit hoher Bildrate geschrieben wird, kostet in MAPIR

-Tests etwa 30 % der Erfassungsrate (Hunderte von Dateien pro Sekunde bei einem kleinen ROI). Der Windows

-Installer bietet die Option **„Chloros

-Backend vom Microsoft Defender-Scan ausschließen (deutlich schnellere Kameraerfassung; empfohlen)“** an, die standardmäßig aktiviert ist und bei der Deinstallation wieder aufgehoben wird. Falls Sie diese Option deaktiviert haben, führen Sie das Installationsprogramm erneut aus, wobei die Option aktiviert ist, oder fügen Sie einen Defender-Prozessausschluss für „`chloros-backend.exe`“ hinzu (im Ordner „`backend`“ des Chloros

-Installationsordners). Wenn Defender während der kontinuierlichen Erfassung eines Arrays einen oder mehrere CPU-Kerne beansprucht, vermerkt das Chloros

-Backend dies einmalig in seinem Protokoll und schlägt dieselben beiden Abhilfemaßnahmen vor.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Auswahl von Kameras und Exporttypen

Der Hilfetext des Fensters fasst es zusammen: Wählen Sie aus, welche Kameras und Exporttypen „Capture All“ (Alles erfassen) generiert – standardmäßig ist alles aktiviert, und die Auswahl wird mit diesem Projekt gespeichert.

* Die Schaltflächen **„Alle auswählen“ / „Keine auswählen“** schalten die Kontrollkästchen zum Einbeziehen aller Kameras auf einmal um.
* **Schaltflächen für den Massenexport**(zwei Reihen mit Schaltflächen):**Alle Raw / Alle debayert / Alle Vorschau / Alle Radiance / Alle Reflektanz / Alle Index**. Jede Option wird in drei Farbstufen angezeigt: Grün ✓ = aktiviert für jede Kamera, die diesen Typ unterstützt; Gelb – = für einige aktiviert; Grau = keine. Ein Umschalter ist deaktiviert, wenn keine angeschlossene Kamera diesen Typ unterstützt. Alle sind ausgegraut, solange „Fastest Capture“ aktiviert ist.
* **Zeilen pro Kamera**: ein Kontrollkästchen zum Einbeziehen sowie eine erweiterbare (▸/▾) Liste der für diese Kamera verfügbaren Exporttypen mit einzelnen Kontrollkästchen. Die Zeile zeigt eine Anzahl aktivierter Optionen an, z. B. „4/6“.

### Exporttypen und welche Kameras sie unterstützen

Es gibt sechs Exporttypen: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. In der Zeile der jeweiligen Kamera werden nur die zutreffenden angezeigt:

| Exporttyp | Inhalt | RGB

(FRGB) | Bayer-Multispektral (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-Mosaik (Mono: das einzelne Band) direkt vom Sensor | ✓ | ✓ | ✓ |
| **Debayered** | Lineares Demosaicing: Der Dunkeloffset (DSNU) aus dem Kalibrierungspaket wird aus dem Rohdatenbild entfernt, anschließend erfolgt ein kantenorientiertes Demosaicing – ohne Linearitätskorrektur, Flatfield oder Unmix. Gespeichert als 3-Kanal-RGB

-TIFF

(Mono: 1-Kanal-Graustufen) | ✓ | ✓ | ✓ |
| **Vorschau** | Vollständige Anzeigekette (Weißabgleich + Gamma gemäß Kameraprofil sowie die „Highlights“-Einstellung der RGB

-Kamera; multispektral: Falschfarben-Streckung) | ✓ | ✓ | ✓ |
| **Strahlungsintensität** | float32 W/m²/sr/nm über die vollständige radiometrische Kette | — (nicht verfügbar) | ✓ | ✓ |
| **Reflexionsgrad** | uint16 ρ (32768 = 1,0) | — (nicht verfügbar) | ✓ — wird nur angezeigt, wenn die Kamera über einen DAQ-Lichtsensor verfügt (eigenen oder vom Array geerbten) | wie „Multispektral“ |
| **Index** | Vegetationsindex (LUT)-Darstellung | — | ✓ — erfordert einen aktivierten, nicht leeren Indexausdruck auf der Kamera und wird für Mitglieder eines kombinierten Arrays nicht angeboten (das Array verfügt über einen gemeinsamen Index) | — (ein Index benötigt ≥2 Bänder; siehe [Mono-Kameras &amp; Vegetationsindizes](mono-indices.md)) |

Strahlungsintensität und Reflexionsgrad werden für RGB

-Kameras niemals angeboten – die Strahlungsintensität pro Bayer-Pixel ist für einen breitbandigen photometrischen Sensor nicht aussagekräftig.

### Schnellste Aufnahme

Der Schalter **⚡ Schnellste Erfassung – nur RAW**(orange, wenn aktiviert) setzt alle Exportauswahlen auf**nur RAW** zurück – plus eine kostenlose Kombinationskomposition für Arrays –, damit das Bild so schnell wie möglich gespeichert wird: Die Berechnungen für Radianz, Reflektanz und Anzeige werden zum Zeitpunkt der Erfassung vollständig übersprungen.

{% hint style="info" %}
**Eine `.daq`-Datei wird weiterhin gespeichert.** Wenn ein Lichtsensor zugewiesen ist, schreibt „Schnellste Erfassung“ weiterhin den DAQ-Downwelling-Messwert (in den Ordner „`daq/`“ der Erfassungen) – so können Strahlungsintensität, Reflektanz und Indexprodukte später durch erneute Verarbeitung erstellt werden (siehe [Neuverarbeitung von Aufnahmen](#re-processing-captures-into-calibrated-products)). „Fastest Capture“ wirkt sich zudem nicht destruktiv auf Ihre Auswahl in den Kontrollkästchen aus: Schalten Sie die Funktion aus, werden die Einstellungen wiederhergestellt.
{% endhint %}

### Steuerelemente pro Array

Jedes angeschlossene Array erhält im Fenster eine eigene Gruppenkarte:

* **Kontrollkästchen „Einbeziehen“** (drei Zustände für alle Elemente) und den Namen des Arrays mit seinem Anzeigemodus: „(kombiniert | separat)“.
* Kontrollkästchen **Ausgerichtet**(standardmäßig**aktiviert**): Passt die Exporte der Elemente an das Ausrichtungsprofil des Arrays an, sodass die Exporte über alle Kameras hinweg pixelgenau registriert sind. Die Rohdaten bleiben unverzerrt, enthalten jedoch die Transformation in ihren Metadaten. (Das Profil selbst wird im [Fenster „Array-Einstellungen“](camera-settings.md#alignment-co-registration) berechnet.) Die Dateien der Array-Mitglieder werden in der nativen Ausrichtung der jeweiligen Kamera gespeichert, wobei etwaige Spiegelungs- oder Rotationseinstellungen in der Datei vermerkt, aber nicht angewendet werden.
* Die Kamerareihen der einzelnen Elemente sind in der Karte verschachtelt.

Die Array-Karte beherbergt außerdem zwei Recorder. Stellen Sie sich diese als **Überwachung vs. Analyse** vor:

| Recorder | Stufe | Was er aufzeichnet |
| --- | --- | --- |
| **● Index-Video aufzeichnen / ■ Aufzeichnung beenden** *(nur kombinierte Arrays)* | **Überwachung** | Das Live-Composite des kombinierten Index als Video mit 10 fps – 8 Bit, Vorschauauflösung, LUT eingebrannt. Erfordert ein geöffnetes Projekt und eine Live-Streaming-Ansicht. Zeigt während der Aufnahme Bildzahlen und verstrichene Zeit an. |
| **⦿ Rohdaten-Burst aufzeichnen / ■ Rohdaten-Burst beenden** *(beliebiges Array)* | **Analyse**| Rohdaten-Bayer-Bilder mit der Live-Aufnahmerate (ohne Verarbeitung) sowie ein Manifest pro Bild und `.daq`-Messwerte, gespeichert in `captures/bursts/`. Nach einer Serienaufnahme erscheint die Schaltfläche**Video erstellen**: Sie verarbeitet die Serienaufnahme offline zu kalibriertem Video – kombinierter Index und/oder Radianz/Reflexanz/Index pro Kamera – sowie optionalen TIFFs. Die Erstellung des kombinierten Indexes startet automatisch, sobald Sie die Serienaufnahme beenden. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Der „Capture All“-Ablauf

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->



Klicken Sie in der Kameraliste der Seitenleiste auf **„Capture All“**:

1. Jede enthaltene, sichtbare und nicht angehaltene Kamera nimmt mit den ausgewählten Exporttypen auf. **Arrays werden als ein synchronisierter Auslöser ausgelöst** (eine einzige synchronisierte Gruppe über alle Mitglieder hinweg – siehe [Multi-Kamera-Arrays](arrays.md)); Einzelkameras nehmen individuell auf.
2. Ausgeblendete (Auge) oder angehaltene Kameras werden übersprungen. Ein Array wird nur dann vollständig blockiert, wenn *alle* seine Mitglieder ausgeblendet oder angehalten sind.
3. Immer wenn ein Lichtsensor zugewiesen ist, wird der entsprechende DAQ-Downwelling-Messwert als `.daq`-Datei im `daq/`-Ordner der Aufnahmen gespeichert – auch bei reinen Rohdatenaufnahmen –, sodass radiometrische Produkte später jederzeit abgeleitet werden können.
4. Die Schaltfläche zeigt den aktuellen Fortschritt an – „Aufnahme… fertig/insgesamt“ – und wird im Dauer-/Intervallmodus zu **Stopp (N)**. Für jeden Aufnahmeeintrag gilt ein Timeout von 300 s.
5. Nach Abschluss des Durchlaufs meldet eine Ergebnisanzeige **„N Dateien gespeichert“**oder**„N gespeichert, F fehlgeschlagen“**sowie „(S ausgeblendet/angepausiert/übersprungen)“, wenn Kameras übersprungen wurden. Wenn das Laufwerk voll ist, wird der Durchlauf gestoppt und es wird**„Gestoppt: Laufwerk voll. N gespeichert, F fehlgeschlagen“** gemeldet – jede Datei, die nicht geschrieben werden konnte, wird als fehlgeschlagen gezählt.

## Speicherort der Aufzeichnungen

Aufzeichnungen werden im geöffneten Projekt unter `<project>/captures/` gespeichert. Jeder Exporttyp wird in einem **eigenen Unterordner** abgelegt, sodass bei einer mehrstufigen Aufzeichnung niemals verschiedene Typen vermischt werden:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` ist der Zeitstempel der Aufnahme und `<serial>` die Seriennummer der Kamera. Einzelaufnahmen erhalten den Namen `capture_<ts>_SN<serial>_<level>`; Array-Aufnahmen, die durch einen synchronisierten Auslöser ausgelöst wurden, erhalten den Namen `sync_<ts>_SN<serial>_<level>` und **teilen sich einen gemeinsamen Zeitstempel für alle Kameras in der Gruppe** (das Level-Suffix entfällt, wenn eine Kamera nur ein einziges Level speichert).
* **Eine Besonderheit, die man beachten sollte:** Die Anzeigeebene wird in einem Ordner namens `preview/` gespeichert, während die Dateien `_display` im Namen behalten – Ordner und Suffix unterscheiden sich nur für diese Ebene.
* Unbekannte Ebenen werden in einem Ordner mit ihrem eigenen Namen abgelegt; kann kein Unterordner erstellt werden, wird die Datei in das Stammverzeichnis der Aufnahmen geschrieben, anstatt verloren zu gehen.
* Aufnahme-TIFFs werden standardmäßig verlustfrei (DEFLATE) komprimiert und enthalten ihre vollständigen Kalibrierungs- und Verarbeitungs-Metadaten **innerhalb der XMP-Datei** — Die Aufnahmen sind selbsterklärend und enthalten keine weiteren Sidecar-Dateien außer der `.daq`-Datei (und deren `.daq.cameras.json`).
* **Clamp-Masken (neu in 1.3.0).**Wenn bei der 3×3-Spektralentmischung ein Band bei mindestens einem Pixel auf 0 geklemmt werden musste, erhalten die Radianz-, Reflektanz- und Index-Produkte eine pixelweise Maske, `clamp_masks/<product folder>/<product name>_clampmask.tif` (Bit 0 = das Band im R-Kanal des Produkts, Bit 1 = G, Bit 2 = B); das XMP-Tag `Chloros:UnmixClampMask` des Produkts verweist darauf. Die Masken befinden sich**neben** den Produktordnern, niemals in einem davon, sodass ein Produktordner wie `reflectance/` unverändert an eine Photogrammetrie-Software übergeben werden kann. Der eigene Import von „Chloros

“ überspringt die Masken.
* **Ausrichtung (1.3.0).** Die Produkte „Preview“, „Radiance“, „Reflectance“ und „Index“ einer eigenständigen Kamera werden durch die Einstellung „Spiegelung/Drehung“ gedreht; „Raw“ und „Debayered“ bleiben so, wie der Sensor sie ausgelesen hat. Alle Dateien im Array bleiben im Originalformat. Jede Datei speichert die Einstellung in ihrem XMP, sodass sie bei einer erneuten Verarbeitung auf dieselbe Weise angewendet wird.
* **Zu welcher Kamera gehört eine `.daq`-Datei (1.3.0)?** Der `.daq` einer Aufnahme wird zum Zeitpunkt der Verarbeitung nur mit Bildern der Kameras abgeglichen, die in ihrem `.daq.cameras.json` genannt sind. Eine eigenständige Kamera ohne eigenen Lichtsensor übernimmt daher nicht mehr die Messwerte einer anderen Kamera – ordnen Sie ihr einen Sensor zu oder tragen Sie einen im Reiter „Lichtsensoren“ ein. Eine Lichtsensor-Aufzeichnung (sowie alle `.daq` ohne diese Datei, einschließlich älterer Aufzeichnungen) dient weiterhin allen Kameras.

Dies ist dasselbe Layout, das `chloros-cli lattice capture` / `array-capture` in ihr Verzeichnis `-o` schreiben – dokumentiert in der [CLI

-Referenz § Wie ein Aufnahmordner aussieht](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Nachbearbeitung von Aufnahmen zu kalibrierten Produkten

Die aufgenommenen Rohbilder sowie die gespeicherte Datei `.daq` sind alles, was die Verarbeitungspipeline benötigt – deshalb ist „Fastest Capture“ für den produktiven Einsatz geeignet.

* **GUI**: Fügen Sie den Aufnahmordner zu einem Projekt hinzu ([Dateien zu einem Projekt hinzufügen](../processing-images-gui/adding-files-to-a-project.md)) und verarbeiten Sie ihn wie gewohnt.
* **CLI**: Verweisen Sie `process` auf das **Stammverzeichnis der Aufnahmen**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importiert normalerweise nur den von Ihnen angegebenen Ordner, aber wenn dieser Ordner keine Bilder enthält und Unterordner hat, wird er automatisch durchsuchte – sodass die Unterordner der jeweiligen Ebenen und der Ordner `daq/` in einem Durchgang erfasst werden. Jede Erfassung wird als **einzelnes Bild** importiert, wobei die anderen Ebenen als anzeigbare Modi angehängt werden, nicht als ein Bild pro Ebene.

Die direkte Benennung eines Unterordners einer Ebene (z. B. `…/captures/raw/`) funktioniert ebenfalls, lässt jedoch den Ordner „`daq/`“ zurück – kopieren Sie diesen zusätzlich, wenn Sie-ein radiometrisches Produkt aus „`raw/`“ ableiten; andernfalls kann der Zeitstempelabgleich nicht zugeordnet werden.

{% hint style="warning" %}
**Die Verarbeitung beginnt immer bei „`raw`“.** Innerhalb jeder Aufnahme ist das Rohbild die Quelle der Pipeline; `debayered`, `radiance`, `reflectance`, und `preview` dienen als Anzeigemodi, werden jedoch niemals durch die Pipeline zurückgeführt – eine erneute Verarbeitung eines abgeleiteten Produkts würde die bereits in seine Pixel eingebackenen Vignettierungs-, Farb- und Strahlungsberechnungen erneut anwenden, weshalb Chloros

dies ablehnt, anstatt eine doppelte Verarbeitung durchzuführen. Die Renderings `index/` und `composite/` werden überhaupt nicht verarbeitet (es handelt sich um Ausgabedateien, nicht um Aufnahmen). Ein „captures“-Ordner, der **ohne** Rohdatenimporte gespeichert wurde, wird normal angezeigt, aber `process` überspringt ihn und gibt dies auch an; `--input-level {raw,debayered,processed}` ist die absichtliche Ausweichmöglichkeit, die einen Einstiegspunkt erzwingt. Die genauen Überspringmeldungen finden Sie in der [CLI

-Referenz](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Zwei weitere Verhaltensweisen, die Sie bei der Skripterstellung für die Nachbearbeitung kennen sollten:

* Ein `chloros-cli process`-Lauf, der Produkte angefordert hat, aber **keine Bildprodukte**geschrieben hat, schlägt laut fehl und beendet sich mit einem Wert ungleich Null** – Sie werden niemals einen stillen, leeren Lauf erhalten. Erfolgreiche Läufe melden ihre Produktanzahl. (Ein absichtlich nur auf Metadaten beschränkter Lauf gilt dennoch als Erfolg.)
* Neu importierte, bereits verarbeitete Exporte belegen niemals den Raw-Slot einer Erfassung – das ursprüngliche Raw-Datenmaterial bleibt immer die Quelle der Pipeline.

## CLI

-Entsprechungen

Alles auf dieser Seite kann im Headless-Modus ausgeführt werden. Die Erfassungsmodi der GUI lassen sich direkt auf `chloros-cli lattice array-capture` abbilden:

| GUI | CLI

|
| --- | --- |
| Einzel | `chloros-cli lattice array-capture` |
| Kontinuierlich | `array-capture --continuous [--count N] [--duration S]` |
| Intervall | `array-capture --interval S [--duration S]` |
| Schnellste Erfassung | `array-capture --fastest` |
| Kontrollkästchen „Ausgerichtet“ | `--aligned / --no-aligned` |
| Kontrollkästchen für Exporttyp | `--processing LEVEL` oder `--levels L1,L2,…` (Standard: `all`) |
| Video-Index aufzeichnen | `chloros-cli lattice array-record` |
| Rohdaten-Burst aufzeichnen / Video erstellen | `chloros-cli lattice array-burst` / `array-build-video` |

Die vollständigen Flag-Tabellen, die Option „Smart-AE Settled-Capture“ (`--smart`) und das Modell mit konstanter Aufnahmerate sind im Abschnitt [CLI

-Referenz § Aufnahmemodi, Recorder und Offline-Nachbearbeitung](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess) beschrieben.
