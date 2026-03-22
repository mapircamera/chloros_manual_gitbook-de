---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<details>

<summary>Kann ich mit Chloros Bilder von Kameras verarbeiten, die nicht von der Marke MAPIR stammen?</summary>

Nein, Chloros unterstützt nur die Verarbeitung von MAPIR-Kamerabildern. Weitere Informationen finden Sie in der Liste der [unterstützten Kameramodelle](supported-cameras.md). Wir bieten die Verarbeitung von Bildern anderer Kameras in der MAPIR Cloud an, die vollständige Liste finden Sie [hier](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Kann ich meine Bilder ohne Kalibrierungsziel auf Reflexion kalibrieren?</summary>

Nein. Ohne ein Bild des Kalibrierungsziels, das etwa zur gleichen Zeit wie die Nicht-Zielbilder aufgenommen wurde, können Sie die Pixelwerte des Bildes nicht mit einem bekannten Reflexionsgrad in Prozent in Beziehung setzen. Wenn Sie zudem nicht das Protokoll eines MAPIR-Lichtsensors beifügen, wird das Umgebungslichtspektrum nicht gemessen, und die Reflexionsergebnisse sind nicht genau.

</details>

<details>

<summary>Kann ich meine Bilder vor der Verarbeitung in Chloros bearbeiten?</summary>

Nein. Chloros geht davon aus, dass die Eingabedaten nicht verändert wurden. Ändern Sie die Dateinamen nicht.

</details>

<details>

<summary>Kann ich meine MAPIR- und Survey3-Kameras auf automatische Belichtung einstellen und die Bilder in Chloros verarbeiten?</summary>

Nein. Survey3-Bilddatensätze müssen eine feste/gesperrte Belichtung haben, also keine automatische Verschlusszeit oder automatische ISO. Alle Bilder desselben Kameramodells müssen identische Verschlusszeiten und ISO-Werte (Belichtung) aufweisen.

</details>

<details>

<summary>Kann Chloros Orthomosaikbilder verarbeiten oder analysieren?</summary>

Nein. Es werden nur einzelne MAPIR-Kamerabilder unterstützt, keine zusammengesetzten Bilder wie eine Orthomosaik-Karte.

</details>

<details>

<summary>Wie kann ich den Schritt der Zielerkennung in Chloros beschleunigen?</summary>

Wenn Sie in der Dateibrowser-Tabelle die Zielbilder in der rechten Spalte vorab auswählen, weist dies Chloros an, nur in diesen Bildern nach Kalibrierungszielen zu suchen, was die Verarbeitung erheblich beschleunigt.

</details>

<details>

<summary>Wenn ich meine Bilder in <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">die MAPIR Cloud</a> hochladen möchte, sollte ich sie dann vor dem Hochladen in Chloros verarbeiten?</summary>

Wenn Sie vorhaben, die Bilder auf unsere Online-Verarbeitungsplattform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) hochzuladen, bearbeiten Sie die Bilder bitte nicht vor dem Hochladen. Die Cloud führt dieselben Verarbeitungsschritte und noch mehr durch.

</details>

<details>

<summary>Wird MAPIR jemals die Funktion X unterstützen? Ich würde mir wirklich wünschen, dass MAPIR X anbietet.</summary>

Wir sind immer an Feedback zu unseren Produkten interessiert. Wenn Sie ein Problem mit unseren Produkten feststellen oder einen Vorschlag zur Verbesserung unserer Produkte haben, kontaktieren Sie uns bitte [HIER](https://www.mapir.camera/community/contact), um uns Ihre Meinung mitzuteilen. Ein Großteil unserer Forschung und Entwicklung orientiert sich an den wichtigsten Bedürfnissen unserer Kunden.

</details>

<details>

<summary>Ist Chloros für Linux verfügbar?</summary>

Ja! Chloros 1.1.0 unterstützt Linux amd64 (x86_64) und arm64 (NVIDIA Jetson JetPack 6) über `.deb`-Pakete. CLI und Python SDK werden auf Linux vollständig unterstützt. Für Linux gibt es keine GUI – die gesamte Interaktion erfolgt über [CLI](CLI.md) oder [Python SDK](api-python-sdk.md). Weitere Informationen finden Sie unter [Linux Übersicht](linux/linux-overview.md).

</details>

<details>

<summary>Kann ich Chloros auf NVIDIA Jetson ausführen?</summary>

Ja! Chloros 1.1.0 unterstützt NVIDIA Jetson-Plattformen, darunter Jetson Nano, Orin Nano, Orin NX und AGX Orin mit JetPack 6. Chloros erkennt Ihr Jetson-Modell automatisch und optimiert die Verarbeitungsstrategie. Anweisungen zur Einrichtung und Bereitstellung finden Sie im [NVIDIA Jetson-Handbuch](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Optimiert sich Chloros automatisch für meine Hardware?</summary>

Ja! Chloros 1.1.0 enthält [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), das Ihre CPU, GPU, RAM und (auf Jetson) die Temperatursensoren automatisch erkennt. Anschließend wählt es die optimale Verarbeitungsstrategie aus – von `GPU_PARALLEL` auf Systemen mit großem Arbeitsspeicher über `GPU_SINGLE` auf Geräten mit begrenzten Ressourcen bis hin zu `CPU_PARALLEL` auf Systemen ohne NVIDIA-GPU. Eine manuelle Konfiguration ist nicht erforderlich.

</details>

<details>

<summary>Was ist die 4-Thread-Verarbeitungspipeline?</summary>

Chloros 1.1.0 verwendet eine 4-Thread-Pipeline-Architektur für Benutzer von Chloros+: Thread 1 (Erkennung) lädt Bilder und erkennt Kalibrierungsziele, Thread 2 (Kalibrierung) berechnet die Reflexionskalibrierung, Thread 3 (Verarbeitung) führt GPU-beschleunigtes Debayering und Indexberechnung durch und Thread 4 (Export) schreibt Ausgabedateien. Für maximalen Durchsatz können mehrere Bilder gleichzeitig in verschiedenen Threads verarbeitet werden. Weitere Informationen finden Sie unter [Verarbeitungspipeline](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Wie führe ich eine Diagnose meiner Chloros-Installation durch?</summary>

Verwenden Sie den Befehl `selftest`, um 7 Systemdiagnosen auszuführen, darunter Versionsprüfung, Portverfügbarkeit, Backend-Start, API-Konnektivität, Systeminformationen, Denoiser-Modelle und CUDA-Verfügbarkeit:

```bash
chloros-cli selftest
```

Dies ist besonders auf Linux/Jetson-Systemen nützlich, um die GPU- und CUDA-Konfiguration zu überprüfen.

</details>
