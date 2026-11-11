---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<details>

<summary>Kann ich mit Chloros Bilder von Kameras verarbeiten, die nicht von der Marke MAPIR stammen?</summary>

Nein, Chloros unterstützt ausschließlich die Verarbeitung von Bildern von MAPIR-Kameras – den Produktfamilien Survey3 und LATTICE. Weitere Informationen finden Sie in der Liste der [unterstützten Kameramodelle](supported-cameras.md). Die Verarbeitung von Bildern anderer Kameras bieten wir in der MAPIR-Cloud an; die vollständige Liste finden Sie [hier](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Unterstützt Chloros LATTICE-Kameras?</summary>

Ja. Chloros 1.3.0 unterstützt die LATTICE-Kameramodule M3C und M3M durchgängig: **Live-Steuerung**– Erkennen, Verbinden, Vorschau und Aufzeichnung über die Registerkarte „Kameras“ der Benutzeroberfläche, `chloros-cli lattice`, oder das Python-SDK, einschließlich synchronisierter Mehrkamera-Arrays mit PTP-Zeitsynchronisation – sowie**vollständige radiometrische Verarbeitung** der Aufnahmen (Rohdaten → Debayering → Strahlungsintensität → Reflexionsgrad → Index). Siehe [Unterstützte Kameras](supported-cameras.md) und das [LATTICE-Handbuch](lattice/README.md).

</details>

<details>

<summary>Kann ich meine Bilder ohne Kalibrierungsziel auf Reflektanz kalibrieren?</summary>

**Survey3:** Nein. Ohne ein Bild des Kalibrierungsziels, das etwa zur gleichen Zeit wie die Nicht-Ziel-Bilder aufgenommen wurde, können Sie die Pixelwerte des Bildes nicht mit einem bekannten Reflexionsgrad in Prozent in Beziehung setzen. Wenn Sie zudem kein Protokoll eines MAPIR-Lichtsensors beifügen, wird das Umgebungslichtspektrum nicht gemessen, und die Reflexionsergebnisse sind nicht genau.

**LATTICE:** Ja. Die Reflexion kann auf die von einem DAQ-Lichtsensor gemessene nach unten gerichtete Bestrahlungsstärke anstelle eines Panels bezogen werden (ρ = π·L/E). Wenn ein QA-konformes Zielobjekt im Bild *vorhanden* ist, wird es standardmäßig zur absoluten Referenz (`--reflectance-source auto`), es sei denn, es wird mit einem Flag als konform markiert; in diesem Fall konkurrieren das Zielobjekt und die DAQ-Ketten um den gemessenen Wert. Eine Ausnahme: „Die Reflektanz von F988 wird mithilfe eines Reflektanzpanels in der Szene kalibriert: Da das Band außerhalb des kalibrierten Bereichs des DAQ-Lichtsensors liegt, wendet Chloros Ihre letzte Panel-Erfassung an und behält diese zwischen den Panel-Messungen bei.“ Siehe [Kalibrierungsziele](calibration-targets.md).

</details>

<details>

<summary>Benötige ich einen DAQ-Lichtsensor?</summary>

Nicht für die Strahlungsintensität: LATTICE-Strahlungsintensitäts-Exporte basieren auf der werkseitigen radiometrischen Kalibrierung jeder Kamera und erfordern weder einen DAQ-Sensor noch ein Ziel. Für die **Reflexion**benötigen Sie eine Referenz für das Umgebungslicht – entweder die Messung der nach unten gerichteten Strahlung eines DAQ-Lichtsensors oder ein Kalibrierungsziel innerhalb des Bildausschnitts. Mit einem DAQ-Sensor können Sie kalibrierte Reflexionswerte ermitteln,**ohne dass Sie Panels in der Szene platzieren müssen**. Aufgezeichnete `.daq`-Dateien werden automatisch anhand des Zeitstempels Ihren Bilddaten zugeordnet. Siehe [Kalibrierungsziele](calibration-targets.md) und die [CLI-Referenz](reference/cli-reference.md).

</details>

<details>

<summary>Kann ich Chloros mit einem KI-Assistenten (Claude, ChatGPT usw.) verwenden?</summary>

Ja – dieses Handbuch und die CLI/das SDK sind darauf ausgelegt:

* Der vollständige Handbuchindex wird unter `https://mapir.gitbook.io/chloros/llms.txt` bereitgestellt, damit KI-Assistenten jede Seite finden können.
* Der Rohtext im Markdown-Format jeder Seite ist unter der entsprechenden URL in Kleinbuchstaben mit dem Anhang `.md` verfügbar (zum Beispiel `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* Die [CLI-Referenz](reference/cli-reference.md) und die [SDK-Referenz](reference/sdk-reference.md) sind für die Nutzung durch LLMs verfasst: genaue Flags, Standardwerte, Beendigungssemantik und Befehle zum Kopieren und Einfügen.

Unter [KI-Assistenten](ai-assistants.md) erfahren Sie, wie Sie Ihren Assistenten auf Chloros ausrichten können.

</details>

<details>

<summary>Wo werden meine verarbeiteten Ausgabedateien gespeichert?</summary>

Die Ergebnisse werden im Projektordner gespeichert, gruppiert nach Kamera und anschließend nach Dateiformat:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **Kamera-Ordner** — `LATT-<sensor>-<lens>-F<filter>` für LATTICE, `<model>_<filter>` (z. B. `Survey3N_RGN`) für Survey3
* **Format-Ordner** — `tiff16`, `tiff8`, `png8`, `jpg8` oder `tiff32`
* **Produktordner** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (immer unter `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — nur LATTICE Multispektral: Pixelmasken neben den Produktordnern, siehe unten**Exportierte Dateien behalten den Namen der Quelldatei bei – der Ordner identifiziert das Produkt, nicht eine Dateinamenerweiterung.**Bei Verwendung der CLI wird der Projektordner unter Ihrem Standard-Projektordner (`Documents/MAPIR/Chloros Projects`, sofern Sie diesen nicht geändert haben) erstellt, es sei denn, Sie übergeben `-o`; ein relativer Pfad „`-o`“ wird relativ zu dem Ordner interpretiert, aus dem Sie den Befehl ausführen. Beachten Sie, dass ein `chloros-cli process`-Lauf, der Produkte angefordert, aber keine geschrieben hat, `Processing finished but wrote no image products.` ausgibt und**mit einem Ergebnis ungleich Null beendet wird**, sodass Skripte dies erkennen können. Siehe [Ausgabebildformate](output-image-formats.md) und die [CLI-Referenz](reference/cli-reference.md).

</details>

<details>

<summary>Was sind die „clamp_masks“-Ordner und die *_clampmask.tif*-Dateien?</summary>

Neu in Version 1.3.0. Bei den Radianz- und Reflektanz-Produkten einer multispektralen LATTICE M3C-Kamera wird eine 3×3-Band-Entmischung durchgeführt, und wenn dabei in einem Band ein negativer Wert entsteht, schneidet Chloros diesen Wert auf Null ab. Ein Produkt mit mindestens einem beschnittenen Pixel erhält eine einbandige `uint8`-Maske, die markiert, welche Pixel und Bänder beschnitten wurden (Bit 0 = R-Kanal des Produkts, Bit 1 = G, Bit 2 = B). Die Masken werden in einem `clamp_masks/`-Ordner **neben** den Produktordnern gespeichert – zum Beispiel `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — und das XMP-Tag des Produkts `Chloros:UnmixClampMask` benennt dessen Maske (oder lautet `none`).

Es handelt sich dabei nicht um Bilder der Szene: Der Chloros-eigene Import überspringt sie, und da sie sich nie in einem Produktordner befinden, kannst du einen Produktordner unverändert an Pix4D oder Metashape übergeben. Siehe [Clamp-Masken](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Warum meldet Chloros: „Der Projektordner ist für diesen Computer zu tief verschachtelt“?</summary>

Dies gilt nur für Windows. Bei Windows sind lange Pfade standardmäßig deaktiviert, sodass kein Dateipfad länger als 260 Zeichen sein darf. Chloros-Produkte befinden sich bis zu etwa 130 Zeichen unterhalb des Projektordners, daher lehnt Chloros einen tiefer liegenden Projektordner vor der Verarbeitung ab – anstatt das Programm auszuführen und nichts zu schreiben, wie es vor Version 1.3.0 der Fall sein konnte. Die Meldung gibt den längsten Projektordner an, der noch zulässig ist.

Beheben Sie das Problem auf eine der folgenden Weisen:

* Verwenden Sie einen kürzeren Projektordner (das **Arbeitsverzeichnis** in den Projekteinstellungen oder `-o` / `chloros-cli set-project-folder` für die CLI).
* Aktivieren Sie lange Pfade unter Windows und starten Sie Chloros neu. Die Option **„Enable Windows long paths“** des Chloros-Installationsprogramms (standardmäßig aktiviert) sorgt dafür; führen Sie das Installationsprogramm erneut aus, falls Sie diese Option deaktiviert haben.

Siehe [Starten der Verarbeitung](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Kann ich meine Bilder vor der Verarbeitung in Chloros bearbeiten?</summary>

Nein. Chloros geht davon aus, dass die Eingabedaten nicht verändert wurden. Ändern Sie die Dateinamen nicht.

</details>

<details>

<summary>Kann ich meine MAPIR-Survey3-Kameras auf automatische Belichtung einstellen und die Bilder in Chloros verarbeiten?</summary>

Nein. Survey3-Bilddatensätze müssen eine feste/gesperrte Belichtung aufweisen, d. h. keine automatische Verschlusszeit oder automatische ISO-Einstellung. Alle Bilder desselben Kameramodells müssen identische Verschlusszeiten und ISO-Werte (Belichtung) aufweisen.

LATTICE-Kameras unterliegen dieser Einschränkung nicht: Chloros steuert ihre Belichtung in Echtzeit (Smart AE), und bei jeder Aufnahme werden die tatsächlich verwendete Belichtung und Verstärkung aufgezeichnet, was von der radiometrischen Pipeline berücksichtigt wird.

</details>

<details>

<summary>Kann Chloros Orthomosaikbilder verarbeiten oder analysieren?</summary>

Nein. Es werden nur einzelne MAPIR-Kamerabilder unterstützt, keine zusammengefügten Bilder wie beispielsweise eine Orthomosaik-Karte.

</details>

<details>

<summary>Wie kann ich den Schritt der Zielerkennung in Chloros beschleunigen?</summary>

Wenn Sie in der Tabelle des Dateibrowsers die Zielbilder in der rechten Spalte vorab auswählen, weist dies Chloros an, nur in diesen Bildern nach Kalibrierungszielen zu suchen, was die Verarbeitung erheblich beschleunigt.

</details>

<details>

<summary>Wenn ich meine Bilder in <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">die MAPIR-Cloud</a> hochladen möchte, sollte ich sie dann vor dem Hochladen in Chloros bearbeiten?</summary>

Wenn Sie vorhaben, Ihre Bilder auf unsere Online-Verarbeitungsplattform [MAPIR-Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) hochzuladen, bearbeiten Sie die Bilder bitte nicht vor dem Hochladen. Die Cloud führt dieselben Verarbeitungsschritte und noch weitere durch.

</details>

<details>

<summary>Wird MAPIR jemals die Funktion X unterstützen? Ich würde mir wirklich wünschen, dass MAPIR X anbietet.</summary>

Wir sind stets an Feedback zu unseren Produkten interessiert. Wenn Sie ein Problem mit unseren Produkten feststellen oder einen Vorschlag zur Verbesserung unserer Produkte haben, kontaktieren Sie uns bitte [HIER](https://www.mapir.camera/community/contact), um uns Ihre Meinung mitzuteilen. Ein Großteil unserer Forschungs- und Entwicklungsarbeit orientiert sich an den wichtigsten Bedürfnissen unserer Kunden.

</details>

<details>

<summary>Ist Chloros für Linux verfügbar?</summary>

Ja! Chloros 1.3.0 unterstützt Linux-amd64 (x86_64) und arm64 (NVIDIA Jetson JetPack 6) über die `.deb`-Pakete. Die CLI und das Python-SDK werden unter Linux vollständig unterstützt, einschließlich der Live-Steuerung von LATTICE-Kameras und DAQ-Sensoren. Es gibt keine grafische Benutzeroberfläche für Linux – die gesamte Interaktion erfolgt über die [CLI](CLI.md) oder das [Python-SDK](api-python-sdk.md). Weitere Informationen finden Sie in der [Linux-Übersicht](linux/linux-overview.md).

</details>

<details>

<summary>Kann ich Chloros auf NVIDIA Jetson ausführen?</summary>

Ja! Chloros unterstützt die NVIDIA Jetson **Orin-Familie** – Orin Nano, Orin NX und AGX Orin – mit JetPack 6. Ältere Modelle (Jetson Nano, TX2, Xavier NX) können JetPack 6 nicht ausführen und werden vom aktuellen Paket nicht unterstützt. Chloros erkennt Ihr Jetson-Modell automatisch und optimiert die Verarbeitungsstrategie entsprechend. Das Jetson-Paket nutzt die CUDA-Version von PyTorch aus JetPack 6, sodass die Verarbeitung auf der GPU des Jetson erfolgt. Anweisungen zur Einrichtung und Bereitstellung finden Sie im [NVIDIA Jetson-Leitfaden](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Optimiert sich Chloros automatisch für meine Hardware?</summary>

Ja! Chloros verfügt über [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), die Ihre CPU, GPU, Ihren Arbeitsspeicher und (auf Jetson) die Temperatursensoren automatisch erkennt. Anschließend wählt es die optimale Verarbeitungsstrategie aus – von `GPU_PARALLEL` auf GPUs mit großem Arbeitsspeicher über `GPU_SINGLE` auf kleineren GPUs bis hin zu `CPU_PARALLEL` auf Systemen ohne CUDA-GPU, die PyTorch nutzen kann. Eine manuelle Konfiguration ist nicht erforderlich.

</details>

<details>

<summary>Was ist die 4-Thread-Verarbeitungspipeline?</summary>

Chloros nutzt für Chloros+-Nutzer eine 4-Thread-Pipeline-Architektur: Thread 1 (Erkennung) lädt Bilder und erkennt Kalibrierungsziele, Thread 2 (Kalibrierung) berechnet die Reflexionskalibrierung, Thread 3 (Verarbeitung) führt GPU-beschleunigtes Debayering und die Indexberechnung durch und Thread 4 (Export) schreibt Ausgabedateien. Für maximalen Durchsatz können mehrere Bilder gleichzeitig in verschiedenen Threads verarbeitet werden. Weitere Informationen finden Sie unter [Verarbeitungspipeline](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Wie führe ich eine Diagnose meiner Chloros-Installation durch?</summary>

Verwenden Sie den Befehl `selftest`, um einen 7-stufigen Smoke-Test durchzuführen: Version, Portverfügbarkeit, Backend-Start, API-Konnektivität (`/api/test`), Systeminformationen (`/api/system-info` – GPU/CUDA/PyTorch), Vorhandensein des Denoiser-Modells und Bereitschaft von CUDA + Denoiser:

```bash
chloros-cli selftest
```

Dies ist insbesondere auf Linux-Systemen nützlich, um die GPU- und CUDA-Konfiguration zu überprüfen. Auf einem Jetson bedeutet `CUDA: False` in der Regel, dass die CUDA- und cuDNN-Bibliotheken von JetPack 6 fehlen: Installieren Sie diese mit `sudo apt install nvidia-jetpack`.

</details>
