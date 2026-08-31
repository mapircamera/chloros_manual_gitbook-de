---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<details>

<summary>Kann ich mit Chloros Bilder von Kameras verarbeiten, die nicht von der Marke MAPIR stammen?</summary>

Nein, Chloros unterstützt ausschließlich die Verarbeitung von Kamerabildern der MAPIR-Familie – also der Survey3- und LATTICE-Serien. Weitere Informationen finden Sie in der Liste der [unterstützten Kameramodelle](supported-cameras.md). Wir bieten die Verarbeitung anderer Kameras in der MAPIR Cloud an; die vollständige Liste finden Sie [hier](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Unterstützt Chloros LATTICE-Kameras?</summary>

Ja. Chloros 1.2.0 unterstützt die LATTICE-Kameramodule M3C und M3M durchgängig: **Live-Steuerung**– Erkennen, Verbinden, Vorschau und Aufzeichnung über die Registerkarte „Kameras“ der Benutzeroberfläche, `chloros-cli lattice` oder über das Python SDK, einschließlich synchronisierter Mehrkamera-Arrays mit PTP-Zeitsynchronisation – sowie**vollständige radiometrische Verarbeitung** der Aufnahmen (Rohdaten → Debayering → Strahlungsintensität → Reflexionsgrad → Index). Siehe [Unterstützte Kameras](supported-cameras.md) und das [LATTICE-Handbuch](lattice/README.md).

</details>

<details>

<summary>Kann ich meine Bilder ohne Kalibrierungsziel auf Reflektanz kalibrieren?</summary>

**Survey3:** Nein. Ohne ein Bild des Kalibrierungsziels, das etwa zur gleichen Zeit wie die Nicht-Ziel-Bilder aufgenommen wurde, können Sie die Pixelwerte des Bildes nicht einem bekannten Reflexionsgrad in Prozent zuordnen. Wenn Sie zudem kein Protokoll eines MAPIR-Lichtsensors beifügen, wird das Umgebungslichtspektrum nicht gemessen, und die Reflexionsergebnisse sind nicht genau.**LATTICE:** Ja. Die Reflexion kann auf die von einem DAQ-Lichtsensor gemessene nach unten gerichtete Bestrahlungsstärke bezogen werden, anstatt auf ein Panel (ρ = π·L/E). Wenn ein QA-konformes In-Frame-Ziel *vorhanden* ist, wird es standardmäßig zur absoluten Referenz (`--reflectance-source auto`). Eine Ausnahme: „F988: Die Reflexion wird mithilfe eines Reflexionspanels in der Szene kalibriert: Da das Band außerhalb des kalibrierten Bereichs des DAQ-Lichtsensors liegt, wendet Chloros Ihre letzte Panel-Erfassung an und behält diese zwischen den Panel-Messungen bei.“ Siehe [Kalibrierungsziele](calibration-targets.md).

</details>

<details>

<summary>Benötige ich einen DAQ-Lichtsensor?</summary>

Nicht für die Strahlungsintensität: LATTICE-Strahlungsintensitäts-Exporte basieren auf der werkseitigen radiometrischen Kalibrierung jeder Kamera und benötigen weder einen DAQ-Lichtsensor noch ein Ziel. Für die **Reflexion**benötigen Sie eine Referenz für das Umgebungslicht – entweder die Messung der nach unten gerichteten Strahlung durch einen DAQ-Lichtsensor oder ein Kalibrierungsziel innerhalb des Bildausschnitts. Mit einem DAQ-Sensor können Sie kalibrierte Reflexionswerte erstellen,**ohne Panels in der Szene platzieren zu müssen**. Aufgezeichnete `.daq`-Dateien werden automatisch anhand des Zeitstempels Ihren Bilddaten zugeordnet. Siehe [Kalibrierungsziele](calibration-targets.md) und die [CLI-Referenz](reference/cli-reference.md).

</details>

<details>

<summary>Kann ich Chloros mit einem KI-Assistenten (Claude, ChatGPT usw.) verwenden?</summary>

Ja – dieses Handbuch und die Dateien CLI/SDK sind dafür ausgelegt:

* Der vollständige Inhaltsverzeichnis des Handbuchs wird unter `https://mapir.gitbook.io/chloros/llms.txt` bereitgestellt, damit KI-Assistenten jede Seite finden können.
* Der Markdown-Rohtext jeder Seite ist unter der entsprechenden Seite in Kleinbuchstaben (URL) mit dem angehängten `.md` verfügbar (zum Beispiel `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* Die [CLI-Referenz](reference/cli-reference.md) und [SDK-Referenz](reference/sdk-reference.md) sind für die Verwendung durch LLMs verfasst: genaue Flags, Standardwerte, Beendigungssemantik und Befehle zum Kopieren und Einfügen.

Unter [KI-Assistenten](ai-assistants.md) erfahren Sie, wie Sie Ihren Assistenten auf Chloros einrichten können.

</details>

<details>

<summary>Wo werden meine verarbeiteten Ausgabedateien gespeichert?</summary>

Die Ergebnisse werden im Projektordner gespeichert, gruppiert nach Kamera und anschließend nach Dateiformat:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **Kamera-Ordner** — `LATT-<sensor>-<lens>-F<filter>` für LATTICE, `<model>_<filter>` (z. B. `Survey3N_RGN`) für Survey3
* **Format-Ordner** — `tiff16`, `tiff8`, `png8`, `jpg8` oder `tiff32`
* **Produktordner** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (immer unter `tiff32`), `<INDEX>_Index_Images/`**Exportierte Dateien behalten den Namen der Quelldatei bei – der Ordner identifiziert das Produkt, nicht eine Dateinamenerweiterung.**Bei der Verwendung von CLI wird der Projektordner neben dem Eingabeordner erstellt, sofern Sie nicht `-o` übergeben. Beachten Sie, dass ein `chloros-cli process`-Lauf, bei dem Produkte angefordert, aber keine geschrieben wurden, „`Processing finished but wrote no image products.`“ ausgibt und**mit einem Ergebnis ungleich Null** beendet wird, sodass Skripte dies erkennen können. Siehe [Ausgabebildformate](output-image-formats.md) und die [CLI-Referenz](reference/cli-reference.md).

</details>

<details>

<summary>Kann ich meine Bilder vor der Verarbeitung in Chloros bearbeiten?</summary>

Nein. Chloros geht davon aus, dass die Eingabedaten nicht verändert wurden. Ändern Sie die Dateinamen nicht.

</details>

<details>

<summary>Kann ich meine Kameras (MAPIR und Survey3) auf automatische Belichtung einstellen und die Bilder in Chloros verarbeiten?</summary>

Nein. Bilddatensätze von Survey3 müssen eine feste/gesperrte Belichtung aufweisen, d. h. keine automatische Verschlusszeit oder automatische ISO-Einstellung. Alle Bilder desselben Kameramodells müssen identische Verschlusszeiten und ISO-Werte (Belichtung) aufweisen.

LATTICE-Kameras unterliegen dieser Einschränkung nicht: Chloros steuert ihre Belichtung in Echtzeit (Smart AE), und bei jeder Aufnahme werden die tatsächlich verwendete Belichtung und Verstärkung aufgezeichnet, was die radiometrische Pipeline berücksichtigt.

</details>

<details>

<summary>Kann Chloros Orthomosaikbilder verarbeiten oder analysieren?</summary>

Nein. Es werden nur einzelne MAPIR-Kamerabilder unterstützt, keine zusammengesetzten Bilder wie eine Orthomosaik-Karte.

</details>

<details>

<summary>Wie kann ich den Schritt der Zielerkennung in Chloros beschleunigen?</summary>

Wenn Sie in der Tabelle des Dateibrowsers die Zielbilder in der rechten Spalte vorab auswählen, weist dies Chloros an, nur in diesen Bildern nach Kalibrierungszielen zu suchen, was die Verarbeitung erheblich beschleunigt.

</details>

<details>

<summary>Wenn ich meine Bilder in <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">die MAPIR Cloud</a> hochladen möchte, sollte ich sie dann vor dem Hochladen in Chloros verarbeiten?</summary>

Wenn Sie vorhaben, die Bilder auf unsere Online-Verarbeitungsplattform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) hochzuladen, bearbeiten Sie die Bilder vor dem Hochladen bitte nicht. Die Cloud führt dieselben Verarbeitungsschritte und noch weitere durch.

</details>

<details>

<summary>Wird MAPIR jemals die Funktion X unterstützen? Ich würde mir wirklich wünschen, dass MAPIR X anbieten würde.</summary>

Wir freuen uns immer über Feedback zu unseren Produkten. Wenn Sie ein Problem mit unseren Produkten feststellen oder einen Vorschlag zur Verbesserung unserer Produkte haben, kontaktieren Sie uns bitte [HIER](https://www.mapir.camera/community/contact), um uns Ihre Meinung mitzuteilen. Ein Großteil unserer Forschungs- und Entwicklungsarbeit orientiert sich daran, auf die wichtigsten Bedürfnisse unserer Kunden einzugehen.

</details>

<details>

<summary>Ist Chloros für Linux verfügbar?</summary>

Ja! Chloros 1.2.0 unterstützt Linux amd64 (x86_64) und arm64 (NVIDIA Jetson JetPack 6) über `.deb`-Pakete. CLI und Python SDK werden auf Linux vollständig unterstützt, einschließlich der Live-Steuerung von LATTICE-Kameras und DAQ-Sensoren. Für Linux gibt es keine grafische Benutzeroberfläche – die gesamte Interaktion erfolgt über [CLI](CLI.md) oder [Python SDK](api-python-sdk.md). Weitere Informationen finden Sie in der [Linux-Übersicht](linux/linux-overview.md).

</details>

<details>

<summary>Kann ich Chloros auf NVIDIA Jetson ausführen?</summary>

Ja! Chloros unterstützt NVIDIA Jetson-Plattformen, darunter Jetson Nano, Orin Nano, Orin NX und AGX Orin mit JetPack 6. Chloros erkennt Ihr Jetson-Modell automatisch und optimiert die Verarbeitungsstrategie entsprechend. Anweisungen zur Einrichtung und Bereitstellung finden Sie im [NVIDIA Jetson-Leitfaden](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Optimiert sich Chloros automatisch für meine Hardware?</summary>

Ja! Chloros verfügt über die [dynamische Rechenanpassung](processing-architecture/dynamic-compute-adaptation.md), die Ihre CPU, GPU, Ihren Arbeitsspeicher und (auf Jetson) die Temperatursensoren automatisch erkennt. Anschließend wählt es die optimale Verarbeitungsstrategie aus – von `GPU_PARALLEL` auf Systemen mit großem Arbeitsspeicher über `GPU_SINGLE` auf Geräten mit begrenzten Ressourcen bis hin zu `CPU_PARALLEL` auf Systemen ohne NVIDIA-GPU. Eine manuelle Konfiguration ist nicht erforderlich.

</details>

<details>

<summary>Was ist die 4-Thread-Verarbeitungspipeline?</summary>

Chloros nutzt eine 4-Thread-Pipeline-Architektur für Nutzer von Chloros+: Thread 1 (Erkennung) lädt Bilder und erkennt Kalibrierungsziele, Thread 2 (Kalibrierung) berechnet die Reflexionskalibrierung, Thread 3 (Verarbeitung) führt GPU-beschleunigtes Debayering und die Indexberechnung durch und Thread 4 (Export) schreibt Ausgabedateien. Für maximalen Durchsatz können mehrere Bilder gleichzeitig in verschiedenen Threads verarbeitet werden. Weitere Informationen finden Sie unter [Verarbeitungspipeline](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Wie führe ich Diagnosetests für meine Chloros-Installation durch?</summary>

Verwenden Sie den Befehl `selftest`, um einen 7-stufigen Smoke-Test durchzuführen: Version, Portverfügbarkeit, Backend-Start, API-Konnektivität (`/api/test`), Systeminformationen (`/api/system-info` – GPU/CUDA/PyTorch), Vorhandensein des Denoiser-Modells und Bereitschaft von CUDA + Denoiser:

```bash
chloros-cli selftest
```

Dies ist insbesondere auf Linux-/Jetson-Systemen nützlich, um die GPU- und CUDA-Konfiguration zu überprüfen.

</details>
