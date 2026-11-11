# GUI: Projekte Mit

Chloros können Sie Projekte erstellen, die später wieder geöffnet werden können. Ein Projekt ist ein einfacher Ordner (innerhalb Ihres Projektordners), der Folgendes enthält:

* `project.json` – Projekteinstellungen, Dateiliste und Anzeigeeinstellungen
* `cameras.json` – Kameras und Arrays, die während der Öffnung des Projekts angeschlossen waren, einschließlich ihrer Einstellungen
* `sensors.json` – während des Projektlaufs angeschlossene DAQ-Lichtsensoren sowie Kamera-Sensor-Zuordnungen
* Ihre Aufnahmen, `.daq`-Aufzeichnungen und Ordner mit verarbeiteten Ergebnissen

Es gibt kein proprietäres Projektdateiformat – der Ordner und seine JSON-Dateien bilden das Projekt, wodurch sich Projekte zudem leicht kopieren, archivieren und über die [CLI](CLI.md) oder das [Python-SDK](api-python-sdk.md) steuern lassen.

Sofern Sie dies nicht in den [Projekteinstellungen](project-settings/project-settings.md) ändern, befindet sich der Projektordner unter `Documents\MAPIR\Chloros Projects` in Ihrem Ordner „Dokumente“ (`~/Documents/MAPIR/Chloros Projects` unter Linux).

{% hint style="warning" %}
**Windows: Halten Sie Projektordner kurz, es sei denn, „Lange Pfade“ ist aktiviert.**Unter Windows ist die Option „Lange Pfade“ standardmäßig deaktiviert, sodass kein Dateipfad länger als 260 Zeichen sein darf. Verarbeitete Produkte befinden sich bis zu etwa 130 Zeichen unterhalb des Projektordners. Daher lehnt Chloros ab Version 1.3.0 einen zu tief verschachtelten Projektordner ab – beim Erstellen, Öffnen oder beim Start der Verarbeitung –, anstatt zu laufen und nichts zu schreiben. Die Meldung gibt den längsten Projektordner an, der noch zulässig ist. Verwenden Sie einen kürzeren Projektordner oder aktivieren Sie lange Pfade unter Windows (die Option**„Windows-Pfade aktivieren“** im Installationsprogramm, die standardmäßig aktiviert ist) und starten Sie Chloros neu. Siehe [Starten der Verarbeitung](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Neues Projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Wählen Sie im Hauptmenü „Neues Projekt“ aus und geben Sie einen eindeutigen Namen für Ihr Projekt ein.

Wenn Sie Projektvorlagen gespeichert haben, erscheint unterhalb des Namensfelds ein Dropdown-Menü **Vorlage auswählen** – wenn Sie eine Vorlage auswählen, wird das neue Projekt mit den Einstellungen dieser Vorlage gestartet. Vorlagen werden über [Projekteinstellungen](project-settings/project-settings.md) gespeichert: Geben Sie einen Namen in das Feld „Name der Projektvorlage speichern“ ein und klicken Sie auf das Speichersymbol.

## Projekt öffnen

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>Unter „Projekt öffnen“ werden alle Projekte in Ihrem Projektordner aufgelistet, unten befindet sich <strong>die</strong> Option<strong> Projektordner öffnen</strong></p></figcaption></figure>

Wählen Sie „Projekt öffnen“, um eine Liste der vorhandenen Projekte im Projektordner anzuzeigen. Wenn keine Projekte vorhanden sind, wird das sekundäre Seitenmenü nicht geöffnet. Auf dem obigen Foto sind einige über die Benutzeroberfläche erstellte Projekte (t1, t2, t3) aufgeführt. Die Projekte „DATE\_TIME“ wurden von der CLI unter Verwendung des Standard-Projektbenennungsschemas erstellt. Durch Klicken auf einen beliebigen Projektnamen wird das Projekt geöffnet.

Durch Klicken auf die Schaltfläche „Projektordner öffnen“ wird der Datei-Explorer Ihres Computers am Projektpfad geöffnet. Sie können den Projektpfad in den [Projekteinstellungen](project-settings/project-settings.md) anpassen.

Falls seit dem letzten Öffnen des Projekts Quellbilddateien verschoben oder gelöscht wurden, zeigt Chloros einen Dialog an, in dem genau aufgelistet wird, welche Dateien fehlen, anstatt ein leeres Raster zu öffnen.

## Projekt duplizieren

Diese Option ist verfügbar, sobald ein Projekt geöffnet ist. Wählen Sie „Projekt duplizieren“, um das aktuelle Projekt unter einem neuen Namen zu kopieren – Chloros schlägt den nächsten freien Namen vor (z. B. „MeinProjekt (2)“) – und das Duplikat wird sofort geöffnet.

## Dateien hinzufügen

Nachdem ein Projekt geöffnet wurde, wählen Sie „Dateien hinzufügen“ aus dem Hauptmenü, um einzelne Bilddateien zum aktuellen Projekt hinzuzufügen. Diese Funktion entspricht der Hinzufügen-Funktion des Dateibrowsers, ist jedoch der Einfachheit halber direkt über das Hauptmenü zugänglich.

## Ordner hinzufügen

Nachdem ein Projekt geöffnet wurde, wählen Sie im Hauptmenü „Ordner hinzufügen“, um Ordner mit Bildern zum aktuellen Projekt hinzuzufügen. Sie können mehrere Ordner in einem Schritt auswählen. Doppelte Dateien werden ignoriert.

## Verarbeitung starten / stoppen

Nachdem Dateien zu einem Projekt hinzugefügt wurden, wird im Hauptmenü die Option „Verarbeitung starten“ verfügbar. Dies entspricht dem Klicken auf die Schaltfläche „Wiedergabe/Start“ in der oberen Kopfzeile. Während der Verarbeitung ändert sich der Menüpunkt zu „Verarbeitung stoppen“, damit Sie die Pipeline anhalten können.

## Mit Kamera verbinden / Mit Lichtsensor verbinden

Am unteren Rand des Hauptmenüs befinden sich zwei Hardware-Schnellzugriffe, die unabhängig davon verfügbar sind, ob ein Projekt geöffnet ist oder nicht:

* **Mit Kamera verbinden** – öffnet die [Registerkarte „Kameras“](lattice/), um eine LATTICE-Kamera oder ein Array anzuschließen.
* **Mit Lichtsensor verbinden** – öffnet die [Registerkarte „Lichtsensoren“](daq/), um einen DAQ-Lichtsensor anzuschließen.

Wird die Hardware bei geöffnetem Projekt angeschlossen, wird sie im Projekt gespeichert (siehe unten). Ohne Projekt sind die Verbindungen nur für die aktuelle Sitzung gültig.

{% hint style="info" %}
Die Menüeinträge „Dateien hinzufügen“, „Ordner hinzufügen“ und „Verarbeitung starten/stoppen“ sind nur sichtbar oder aktiviert, wenn ein Projekt geöffnet ist und Dateien hinzugefügt wurden. Sie bieten schnellen Zugriff auf Aktionen, die auch über die Seitenleiste des Dateibrowsers und die Schaltflächen in der Kopfzeile verfügbar sind.
{% endhint %}

## Projekte merken sich Ihre Hardware

Neu in 1.2.0: Ein Projekt behält die angeschlossene Hardware bei, solange es geöffnet ist. Kameras und Arrays (mit ihren kameraspezifischen Einstellungen, Namen, Farben und Rasterlayouts) werden in `cameras.json` gespeichert, Lichtsensoren (mit Namen, Farben und Kameratoewordungen) in `sensors.json` – automatisch, während Sie arbeiten.

Wenn Sie ein Projekt **erneut öffnen**, greift Chloros zunächst nicht auf die Hardware zu. Die einzelnen Bereiche stellen die Verbindung erst wieder her, wenn Sie die entsprechende Registerkarte zum ersten Mal aufrufen:

* Beim Öffnen der Registerkarte **Kameras** werden die gespeicherten Kameras und Arrays erneut verbunden und ihre gespeicherten Einstellungen wieder angewendet.
* Beim Öffnen der Registerkarte **Lichtsensoren**werden die gespeicherten DAQ-Sensoren erneut verbunden und ihre gespeicherten Einstellungen – einschließlich der Kappe – wieder angewendet. Ein Sensor, der in seinem „Cap“-Menü auf**„None“ (unverhüllter Sensor)** eingestellt ist, bleibt unverhüllt. Ein mit einer früheren Chloros-Version gespeichertes Projekt hat einen unverhüllten Sensor als keine Auswahl erfasst, sodass er mit der Sonnenschein-Kosinus-Korrektur wiederhergestellt wird; stellen Sie ihn erneut auf „None“ ein und speichern Sie das Projekt.

Auf diese Weise werden Kameras beim Öffnen eines Projekts zum bloßen Durchsuchen oder Exportieren von Bildern niemals in den Streaming-Modus versetzt. Wenn ein gespeichertes Gerät beim Öffnen der entsprechenden Registerkarte nicht gefunden werden kann, informiert Sie ein Dialogfeld darüber, welche Geräte nicht verfügbar sind, sodass Sie diese erneut verbinden oder entfernen können.

## DAQ-Aufzeichnungen und .daq-Dateien in einem Projekt

* `.daq`-Aufzeichnungen, die bei geöffnetem Projekt erstellt wurden (über die Registerkarte „Lichtsensoren“ oder während der Erfassung), werden **automatisch dem Projekt hinzugefügt**.
* Importierte `.daq`-Dateien sowie alle Projektaufzeichnungen werden im Abschnitt **DAQ-Lichtsensor** der [Projekteinstellungen](project-settings/project-settings.md) aufgelistet, jeweils mit ihrem eigenen Cap-Korrekturprofil.
* Während der Verarbeitung liefern die `.daq`-Dateien des Projekts die von oben einfallende Beleuchtung für Reflektanzprodukte – siehe [Ausgabebildformate](output-image-formats.md).

## Ausführen eines gespeicherten Projekts ohne grafische Benutzeroberfläche

Ein gespeichertes Projekt kann ohne die grafische Benutzeroberfläche ausgeführt werden:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` arbeitet mit einem Projektordnerpfad – siehe [CLI-Referenz](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` gibt ein Projekt-Handle zurück; `connect_all()` schaltet alle gespeicherten Kameras, Arrays und Lichtsensoren mit ihren gespeicherten Einstellungen online (Lichtsensoren über ein laufendes Chloros-Backend) – siehe [SDK-Referenz](reference/sdk-reference.md).
