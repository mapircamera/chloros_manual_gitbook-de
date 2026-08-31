# Kameras anschließen

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Die Registerkarte „Kameras“, bevor etwas angeschlossen ist</p></figcaption></figure>Chloros erkennt LATTICE-Kameras auf der Verbindung automatisch – über die Registerkarte „Kameras“ der Benutzeroberfläche, über `chloros-cli lattice` oder über die Python- und SDK-Dateien. Die Modellbezeichnung der Kamera bestimmt alle nachfolgenden Schritte: Chloros ermittelt das Sensorprofil, die Bandanordnung und die Werkskalibrierung anhand der `DeviceUserID`- und `DeviceSerialNumber`-Daten der Kamera, sodass **pro Kamera keine Konfiguration erforderlich ist**.

Stellen Sie vor dem Anschluss sicher, dass das Host-Netzwerk eingerichtet ist – Link-Local-Adressierung, Jumbo-Frames und bei Arrays die Einstellungen für den Empfangspuffer der Netzwerkkarte. Dies ist die hardwareseitige Einrichtung und wird im LATTICE-Handbuch beschrieben: [**Netzwerkeinrichtung**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Verbindung über die GUI

Öffnen Sie die Registerkarte **Kameras**in der Seitenleiste von Chloros (die Hardware-Registerkarten werden angezeigt, sobald der Start des Backends abgeschlossen ist), oder verwenden Sie das Hauptmenü →**Mit Kamera verbinden**. Beide Optionen öffnen das Dialogfeld**Kamera(s) verbinden**.

### Der Dialog „Kamera(s) verbinden“

Der Dialog scannt das Netzwerk unmittelbar nach dem Öffnen („Netzwerk wird gescannt...“) und listet alle gefundenen Kameras auf. Jede Zeile zeigt das **Modell**der Kamera (z. B. `LATT-M3M-L41-F550`), die**Seriennummer**und die**IP-Adresse** an.

* **Klicken Sie auf eine Zeile, um sie auszuwählen**(grüne Markierung). Sie können**mehrere Kameras** auswählen und diese auf einmal verbinden – Chloros verbindet sie nacheinander.
* Zeilen mit dem Hinweis **„Verbunden“** sind bereits verbunden und können nicht erneut ausgewählt werden.
* Zeilen mit dem Hinweis **„Im Array“** gehören zu einem derzeit verbundenen Kamera-Array. Trennen Sie das Array zunächst, um diese Kamera einzeln zu verwenden.
* **Verbinden** – verbindet die ausgewählten Kameras; die Schaltfläche zeigt eine Anzahl an, z. B. „Verbinden (3)“, wenn mehr als eine Kamera ausgewählt ist.
* **Erneut scannen** – führt die Erkennung erneut durch.
* **Schließen** – schließt das Dialogfeld.
* Wenn der Suchvorgang ohne Ergebnisse endet, zeigt das Dialogfeld **„Keine Kameras im Netzwerk gefunden“** an – siehe [Fehlerbehebung](connecting.md#troubleshooting) weiter unten.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Der Dialog „Kamera(s) verbinden“ – hier dargestellt, wenn keine Kameras im Netzwerk vorhanden sind</p></figcaption></figure>### Erste Verbindung: Herunterladen des Kalibrierungspakets

Wenn eine bestimmte Kamera zum **ersten Mal**an einen Rechner angeschlossen wird, ruft Chloros das werkseitige Kalibrierungspaket der Kamera (~3,8 MB) über GigE direkt von der Kamera ab. Während dieses Vorgangs zeigt der Dialog ein grünes Feld mit der Aufschrift**„Kalibrierungsdaten von der Kamera herunterladen“**und einem seriennummernspezifischen Fortschrittsbalken an – rechnen Sie mit etwa**70 Sekunden** pro Kamera. Das Paket wird auf dem Host zwischengespeichert, sodass bei späteren Verbindungen derselben Kamera der Download vollständig übersprungen wird (und das Feld nie angezeigt wird).

### System analysieren

Die Schaltfläche **System analysieren** im Dialogfeld überprüft den Host und das Netzwerk (während des Vorgangs wird „Analyse läuft...“ angezeigt) und erstellt einen Diagnosebericht:

* **Host** – CPU-Kerne und RAM; GPU-Name und Speicher oder „GPU: Keine erkannt“.
* **Netzwerkschnittstellen** – Name jeder Netzwerkkarte, Verbindungsgeschwindigkeit, MTU (mit dem Vermerk „Jumbo“, sofern aktiv), Up-/Down-Status und Angabe, ob sie an einem USB-Bus angeschlossen ist.
* **Kameras**– Seriennummer, Modell, IP-Adresse und**an welcher Netzwerkkarte die jeweilige Kamera angeschlossen ist**.
* **Leistung** – aktuelle vs. ideale Bildrate (fps) pro Kamera für das Pixelformat, mit einer grünen Zeile „Potenzial: N-fache Verbesserung möglich“, wenn der Idealwert den aktuellen Wert übersteigt.
* **Warnungen und nummerierte Empfehlungen** — oder „Das System scheint für die aktuelle Kameraanzahl in Ordnung zu sein“, wenn nichts zu beheben ist.

Führen Sie das Tool aus, wann immer die Erkennung oder das Streaming unerwartet verläuft — es identifiziert die meisten Probleme auf Netzwerkkartenebene (falsche MTU, Kamera an der falschen Schnittstelle, Einschränkungen des USB-Adapters), ohne den Dialog zu verlassen.

### Ein Array verbinden

Um zwei oder mehr Kameras als **synchronisiertes Array**zu verbinden, verwenden Sie stattdessen den Array-Verbindungsassistenten (**Kamera-Array verbinden**): Er führt Sie Schritt für Schritt durch die Auswahl von Master/Slave (vorausgefüllt durch eine GPIO-Verdrahtungsprüfung), die Wahl des Anzeigemodus (separate vs. kombinierte Kacheln) und eine Szene mit Array-Einstellungen, in der vor der Bestätigung eine Live-Prognose der erreichbaren fps und der Leitungsbandbreite angezeigt wird. Der Assistent und die Array-Workflows werden im Abschnitt „Multi-Kamera-Arrays“ dieses Handbuchs behandelt; das CLI-Äquivalent ist der „LATTICE Camera First-Connect Workflow“ in der [CLI-Referenz](../reference/cli-reference.md).

## Zugriff über die CLI- und SDK-

Für den Zugriff auf CLI und SDK ist ein kostenpflichtiges Chloros+-Abo erforderlich und Sie müssen angemeldet sein; dies wird serverseitig durchgesetzt (`401 AUTH_REQUIRED`, wenn nicht angemeldet, `403 PLAN_UPGRADE_REQUIRED` in der kostenlosen Stufe).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Vollständige Signaturen, Optionen und Erfassungsworkflows: [CLI-Referenz](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK-Referenz](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Wie die Kalibrierung bei der Verbindung erfolgt

Jede LATTICE-Kamera verfügt **auf der Kamera** über ihr werkseitiges Kalibrierungspaket, und Chloros überprüft außerdem die Cloud von MAPIR, wenn die Kamera eine Verbindung herstellt:

| Situation   | Was Chloros verwendet                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| Die**neueste für diese Seriennummer veröffentlichte Kalibrierung** – die Kopie in der Cloud hat Vorrang vor der Kopie auf der Kamera. Eine Kamera, die von MAPIR neu kalibriert oder aktualisiert wurde, aktualisiert sich daher automatisch; es ist kein Eingreifen des Benutzers erforderlich. |
| **Offline**| Das**Kamera-Paket**, wie es ist. Vollständig offline ablaufende Arbeitsabläufe funktionieren weiterhin; sie übernehmen einfach keine neueren Kalibrierungen, bis die Kamera einmal online ist (oder auf die Werkseinstellungen zurückgesetzt wurde).                                                  |

Zum Zeitpunkt der Aufnahme werden die tatsächlich angewendeten Koeffizienten **in den XMP-Metadaten jedes Bildes festgeschrieben**. Ein späteres Kalibrierungsupdate verändert niemals unbemerkt bereits aufgenommene Bilder – bei der Nachbearbeitung einer alten Aufnahme werden die in deren XMP festgehaltenen Koeffizienten verwendet, nicht die aktuellsten.

## Fehlerbehebung

* **„Keine Kameras im Netzwerk gefunden“**– Überprüfen Sie die Link-Local-Konfiguration unter [Netzwerkeinrichtung](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statische Host-Netzwerkkarte `169.254.x.x/16`, Kameras auf demselben Link, kein DHCP/Gateway erwartet. Verwenden Sie anschließend im Verbindungsdialog die Option**System analysieren**, um zu überprüfen, auf welcher Netzwerkkarte jede Kamera sichtbar ist (oder nicht). Führen Sie nach jeder Änderung der Verkabelung oder der Netzwerkkarte einen**Neuscan** durch.
* **Ein zuvor funktionierendes System lässt sich nicht mehr verbinden** (Array-Panel-Gates mit `FRAMES WILL DROP` / `Reduce ROI to enable`) — ein Update des Netzwerkkartentreibers hat die „Receive-Ring“-Einstellungen stillschweigend zurückgesetzt. Wenden Sie diese erneut an oder führen Sie „`chloros-cli lattice network --fix`“ in einem Terminal mit erhöhten Rechten aus; siehe [Netzwerkeinrichtung](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Eine Kamera zeigt „In Array“ an** — sie gehört zu einer verbundenen Array-Sitzung. Trennen Sie das Array, um die Kamera eigenständig zu verwenden.
