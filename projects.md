# GUI: Projekte

Mit Chloros können Sie Projekte erstellen, die später wieder geöffnet werden können. Ein Projekt ist ein einfacher Ordner (innerhalb Ihres Projektordners), der Folgendes enthält:

* `project.json` – Projekteinstellungen, Dateiliste und Anzeigeeinstellungen
* `cameras.json` – Kameras und Arrays, die während der Öffnung des Projekts angeschlossen waren, samt deren Einstellungen
* `sensors.json` – während des Projektlaufs angeschlossene DAQ-Lichtsensoren sowie Kamera-Sensor-Zuordnungen
* Ihre Aufnahmen, `.daq`-Aufzeichnungen und Ordner mit verarbeiteten Ausgabedaten

Es gibt kein proprietäres Projektdateiformat – der Ordner und seine JSON-Dateien bilden das Projekt, wodurch sich Projekte zudem leicht kopieren, archivieren und von der [CLI](CLI.md) oder [Python SDK](api-python-sdk.md) auszuführen.

## Neues Projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Wählen Sie im Hauptmenü „Neues Projekt“ aus und geben Sie einen eindeutigen Namen für Ihr Projekt ein.

Wenn Sie Projektvorlagen gespeichert haben, erscheint unterhalb des Namensfelds ein Dropdown-Menü „**Vorlage auswählen**“ – wenn Sie eine Vorlage auswählen, wird das neue Projekt mit den Einstellungen dieser Vorlage gestartet. Vorlagen werden über die [Projekteinstellungen](project-settings/project-settings.md) gespeichert: Geben Sie einen Namen in das Feld „Name der Projektvorlage speichern“ ein und klicken Sie auf das Speichersymbol.

## Projekt öffnen

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>Unter „Projekt öffnen“ werden alle Projekte in Ihrem Projektordner aufgelistet, unten befindet sich <strong>die</strong> Option<strong> „Projektordner öffnen“</strong></p></figcaption></figure>Wählen Sie „Projekt öffnen“, um eine Liste der vorhandenen Projekte im Projektordner anzuzeigen. Wenn keine Projekte vorhanden sind, wird das sekundäre Seitenmenü nicht geöffnet. Auf dem obigen Foto sind einige über die Benutzeroberfläche erstellte Projekte (t1, t2, t3) aufgeführt. Die Projekte mit dem Namen „DATE\_TIME“ wurden von CLI unter Verwendung des Standard-Projektbenennungsschemas erstellt. Durch Klicken auf einen Projektnamen wird das entsprechende Projekt geöffnet.

Ein Klick auf die Schaltfläche „Projektordner öffnen“ öffnet den Datei-Explorer Ihres Computers am Projektpfad. Sie können den Projektpfad in den [Projekteinstellungen](project-settings/project-settings.md) anpassen.

Falls seit dem letzten Öffnen des Projekts Quellbilddateien verschoben oder gelöscht wurden, zeigt Chloros einen Dialog an, in dem genau aufgelistet wird, welche Dateien fehlen, anstatt ein leeres Raster zu öffnen.

## Projekt duplizieren

Verfügbar, sobald ein Projekt geöffnet ist. Wählen Sie „Projekt duplizieren“, um das aktuelle Projekt unter einem neuen Namen zu kopieren – Chloros schlägt den nächsten freien Namen vor (z. B. „MeinProjekt (2)“) – und das Duplikat wird sofort geöffnet.

## Dateien hinzufügen

Nachdem ein Projekt geöffnet wurde, wählen Sie „Dateien hinzufügen“ aus dem Hauptmenü, um einzelne Bilddateien zum aktuellen Projekt hinzuzufügen. Diese Funktion entspricht der Hinzufügen-Funktion des Dateibrowsers, ist jedoch der Einfachheit halber direkt über das Hauptmenü zugänglich.

## Ordner hinzufügen

Nachdem ein Projekt geöffnet wurde, wählen Sie im Hauptmenü „Ordner hinzufügen“, um Ordner mit Bildern zum aktuellen Projekt hinzuzufügen. Sie können mehrere Ordner in einem Durchgang auswählen. Doppelte Dateien werden ignoriert.

## Verarbeitung starten / stoppen

Nachdem Dateien zu einem Projekt hinzugefügt wurden, wird im Hauptmenü die Option „Verarbeitung starten“ verfügbar. Dies entspricht dem Klicken auf die Schaltfläche „Wiedergabe/Start“ in der oberen Kopfzeile. Während der Verarbeitung ändert sich der Menüpunkt zu „Verarbeitung stoppen“, damit Sie die Pipeline anhalten können.

## Mit Kamera verbinden / Mit Lichtsensor verbinden

Im unteren Bereich des Hauptmenüs befinden sich zwei Hardware-Schnellzugriffe, die unabhängig davon verfügbar sind, ob ein Projekt geöffnet ist oder nicht:

* **Mit Kamera verbinden** – öffnet die [Registerkarte „Kameras“](lattice/), um eine LATTICE-Kamera oder ein Array anzuschließen.
* **Mit Lichtsensor verbinden** – öffnet die [Registerkarte „Lichtsensoren“](daq/), um einen DAQ-Lichtsensor anzuschließen.

Wird die Hardware bei geöffnetem Projekt angeschlossen, wird sie im Projekt gespeichert (siehe unten). Ohne Projekt gelten die Verbindungen nur für die aktuelle Sitzung.

{% hint style="info" %}
Die Menüeinträge „Dateien hinzufügen“, „Ordner hinzufügen“ und „Verarbeitung starten/stoppen“ sind nur sichtbar oder aktiviert, wenn ein Projekt geöffnet ist und Dateien hinzugefügt wurden. Sie bieten schnellen Zugriff auf Aktionen, die auch über die Seitenleiste „Dateibrowser“ und die Schaltflächen in der Kopfzeile verfügbar sind.
{% endhint %}

## Projekte merken sich Ihre Hardware

Neu in 1.2.0: Ein Projekt behält die angeschlossene Hardware bei, solange es geöffnet ist. Kameras und Arrays (mit ihren kameraspezifischen Einstellungen, Namen, Farben und Rasterlayouts) werden in `cameras.json` gespeichert, Lichtsensoren (mit Namen, Farben und Kameratoewordungen) in `sensors.json` – automatisch, während Sie arbeiten.

Wenn Sie ein Projekt **erneut öffnen**, greift Chloros nicht sofort auf die Hardware zu. Die einzelnen Hälften stellen die Verbindung erst wieder her, wenn Sie die entsprechende Registerkarte zum ersten Mal aufrufen:

* Beim Öffnen der Registerkarte **Kameras** werden die gespeicherten Kameras und Arrays erneut verbunden und ihre gespeicherten Einstellungen wieder angewendet.
* Beim Öffnen der Registerkarte **Lichtsensoren** werden die gespeicherten DAQ-Sensoren erneut verbunden.

Auf diese Weise werden Kameras beim Öffnen eines Projekts zum reinen Durchsuchen oder Exportieren von Bildern niemals in den Streaming-Modus versetzt. Wenn ein gespeichertes Gerät beim Öffnen der entsprechenden Registerkarte nicht gefunden werden kann, informiert Sie ein Dialogfeld darüber, welche Geräte nicht verfügbar sind, sodass Sie diese erneut verbinden oder entfernen können.

## DAQ-Aufzeichnungen und .daq-Dateien in einem Projekt

* `.daq`-Aufzeichnungen, die bei geöffnetem Projekt erstellt wurden (über die Registerkarte „Lichtsensoren“ oder während der Erfassung), werden **automatisch dem Projekt hinzugefügt**.
* Importierte `.daq`-Dateien sowie alle Projektaufzeichnungen werden im Abschnitt **DAQ-Lichtsensor** der [Projekteinstellungen](project-settings/project-settings.md) aufgelistet, jeweils mit ihrem eigenen Cap-Korrekturprofil.
* Während der Verarbeitung liefern die `.daq`-Dateien des Projekts die von oben einfallende Beleuchtung für Reflexionsprodukte – siehe [Ausgabebildformate](output-image-formats.md).

## Ausführen eines gespeicherten Projekts ohne GUI

Ein gespeichertes Projekt kann ohne die grafische Benutzeroberfläche ausgeführt werden:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` arbeitet mit einem Projektordnerpfad – siehe [CLI-Referenz](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` gibt ein Projekt-Handle zurück; `connect_all()` schaltet alle gespeicherten Kameras und Sensoren mit ihren gespeicherten Einstellungen online – siehe [SDK-Referenz](reference/sdk-reference.md).
