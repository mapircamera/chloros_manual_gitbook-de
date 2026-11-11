# GUI: Navigation

Wenn Sie Chloros zum ersten Mal starten, wird das Verarbeitungs-Backend gestartet. Sobald das Backend bereit ist, wird das Symbol für das Hauptmenü oben links angezeigt <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">, und die Registerkarten „Kameras“ und „Lichtsensoren“ in der linken Seitenleiste werden freigeschaltet (bis dahin sind sie ausgegraut).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Von links nach rechts enthält die obere Kopfzeile:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Hauptmenü

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Über das Hauptmenü können Sie:

* **Neues Projekt**— ein neues Projekt erstellen. Wenn Sie Projektvorlagen gespeichert haben, erscheint ein Dropdown-Menü**Vorlage auswählen**, sodass das neue Projekt auf den Einstellungen einer Vorlage basiert.
* **Projekt öffnen**— ein bestehendes Projekt öffnen. Die Liste enthält eine Schaltfläche**Projektordner öffnen**, die den Projektordner in Ihrem Datei-Explorer öffnet.
* **Projekt duplizieren** – das aktuell geöffnete Projekt unter einem neuen Namen kopieren (ein freier Name wie „MeinProjekt (2)“ wird vorgeschlagen) und die Kopie öffnen. _(sichtbar, sobald ein Projekt geöffnet ist)_
* **Dateien hinzufügen** — Fügen Sie einzelne Bilddateien zum aktuellen Projekt hinzu _(sichtbar, sobald ein Projekt geöffnet ist)_
* **Ordner hinzufügen** — Fügen Sie einen oder mehrere Ordner mit Bildern zum aktuellen Projekt hinzu _(sichtbar, sobald ein Projekt geöffnet ist)_
* **Verarbeitung starten / Verarbeitung stoppen** — die Bildverarbeitungs-Pipeline starten oder stoppen _(aktiviert, nachdem Dateien hinzugefügt wurden)_
* **Mit Kamera verbinden** — zum [Reiter „Kameras“](lattice/) springen, um eine LATTICE-Kamera oder ein Array anzuschließen. Funktioniert auch ohne geöffnetes Projekt.
* **Mit Lichtsensor verbinden** — Wechselt zur [Registerkarte „Lichtsensoren“](daq/), um einen DAQ-Lichtsensor anzuschließen. Funktioniert auch ohne geöffnetes Projekt.

{% hint style="info" %}
**Nur Windows**: Die Chloros-Desktop-GUI ist unter Windows verfügbar. Linux-Benutzer sollten die Dokumentation zur [CLI](CLI.md) und zum [Python-SDK](api-python-sdk.md) für die Headless-Verarbeitung konsultieren.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Schaltfläche „Wiedergabe/Start“

Wenn diese Option aktiviert ist, startet die Schaltfläche „Verarbeitung starten“ die Bildverarbeitungs-Pipeline.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Fortschrittsbalken <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



Im kostenlosen Chloros-Modus, in dem alle Dateien nacheinander verarbeitet werden, zeigt der Fortschrittsbalken zwei Phasen an: Zielerkennung und Verarbeitung.

Im kostenpflichtigen Chloros+-Modus, in dem alle Dateien gleichzeitig verarbeitet werden, zeigt der Fortschrittsbalken vier Phasen an: Erkennen, Analysieren, Kalibrieren, Exportieren. Wenn Sie den Mauszeiger über den Chloros+-Fortschrittsbalken bewegen, wird das erweiterte 4-teilige Fortschrittsbalken-Panel angezeigt, sodass Sie den Fortschritt verfolgen können. Durch Klicken auf den oberen Fortschrittsbalken wird das Dropdown-Panel fixiert, durch erneutes Klicken wird die Fixierung aufgehoben.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Seitenmenü

Das Menü in der linken Seitenleiste enthält verschiedene Symbole zur Interaktion, in dieser Reihenfolge von oben nach unten:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projekteinstellungen](project-settings/project-settings.md)

Auf der Registerkarte „Projekteinstellungen“ können Sie globale Projekteinstellungen und Einstellungen für die Projektverarbeitung anpassen. Passen Sie diese an, bevor Sie mit der Verarbeitung Ihrer Dateien beginnen.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Dateibrowser

Fügen Sie Dateien/Ordner hinzu und entfernen Sie Dateien aus dem Projekt. Doppelte Dateien werden ignoriert. Aktivieren Sie das Kontrollkästchen in der Zielspalte für ein beliebiges Zielbild; die Verarbeitung berücksichtigt dann nur die markierten Bilder als Ziele, was die Verarbeitungszeit erheblich verkürzt. Verwenden Sie die Umschaltfläche „Bild/Metadaten“, um zwischen der Anzeige des Miniaturbildrasters des ausgewählten Bildes und einer detaillierten Metadatentabelle zu wechseln.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Bildbetrachter](image-viewer-gui/opening-an-image-full-screen.md)

Wenn Sie im Hauptbildbetrachter auf ein Bild klicken, wird es im Reiter „Bildbetrachter“ im Vollbildmodus geöffnet.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Kartenbetrachter](image-viewer-gui/map-markers.md)

Zeigen Sie Ihre Bilder auf einer interaktiven 2D-Karte basierend auf ihren GPS-Koordinaten an. Unterstützt Google Maps und ESRI-Kachelanbieter und wählt automatisch den besten Dienst für Ihren Standort aus. Bewegen Sie den Mauszeiger über Markierungen, um Vorschaubilder der Bilder anzuzeigen.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kameras](lattice/)

Verbinden und steuern Sie LATTICE-Kameras live – einzeln oder als synchronisierte Mehrkamera-Arrays. Die Registerkarte zeigt Live-Vorschau-Kacheln mit Überlagerungen und Histogrammen, Einstellungen pro Kamera und pro Array sowie Aufnahmemöglichkeiten, mit denen festgelegt wird, welche Kameras und Exporttypen bei „Capture All“ verwendet werden. Verfügbar, sobald das Backend bereit ist; eine vollständige Anleitung finden Sie im [LATTICE-Abschnitt](lattice/).

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Lichtsensoren](daq/)

Schließen Sie DAQ-Lichtsensoren an – DAQ-U (USB), DAQ-M (Bluetooth) und DAQ-E (Ethernet) – und zeigen Sie deren kalibrierte Live-Spektrumdiagramme in W/m²/nm an. Von hier aus können Sie `.daq`-Dateien in das geöffnete Projekt aufzeichnen, Sensoren umbenennen, Profile zur Kappenkorrektur auswählen und die DAQ-E-Firmware aktualisieren. Verfügbar, sobald das Backend bereit ist; eine vollständige Anleitung finden Sie im [DAQ-Abschnitt](daq/).

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">-Debug-Protokoll

Überprüfen Sie das Protokoll auf Debug-Ausgaben, wenn Probleme auftreten. Kopieren/laden Sie das Protokoll herunter und senden Sie es an den [MAPIR-Support](https://www.mapir.camera/community/contact), um Unterstützung zu erhalten.

Neu in 1.3.0: Über das Fehler-Symbol oben auf der Registerkarte „Protokoll“ (**Fehler melden**) wird ein Bericht mit Ihrer Beschreibung, bis zu 10 Bildern, den MAPIR-Protokollen und Systemdetails an Chloros gesendet. Sie müssen angemeldet sein, um einen Bericht zu senden. Siehe [Einen Fehler über die Registerkarte „Protokoll“ melden](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Benutzeranmeldung](chloros+-login.md)

Über die Seitenleiste „Benutzeranmeldung“ können Sie sich bei Ihrem Chloros+-Konto anmelden, um erweiterte Funktionen freizuschalten. Außerdem können Sie die aktuelle Anwendungsversion einsehen sowie die Sprache der angezeigten Texte in der Chloros-GUI und der CLI anpassen.
