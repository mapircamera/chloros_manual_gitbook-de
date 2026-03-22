# GUI: Navigation

Wenn Sie Chloros und Chloros (Browser) zum ersten Mal starten, wird das Backend gestartet. Sobald es bereit ist, wird das Symbol für das Hauptmenü oben links angezeigt <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Von links nach rechts enthält die obere Kopfzeile:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Hauptmenü

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Über das Hauptmenü können Sie:

* **Neues Projekt** — ein neues Projekt erstellen
* **Projekt öffnen** — ein bestehendes Projekt öffnen
* **Projektordner öffnen** — den Projektordner in Ihrem Datei-Explorer öffnen
* **Dateien hinzufügen** — einzelne Bilddateien zum aktuellen Projekt hinzufügen _(sichtbar, nachdem ein Projekt geöffnet wurde)_
* **Ordner hinzufügen** — einen Ordner mit Bildern zum aktuellen Projekt hinzufügen _(sichtbar, nachdem ein Projekt geöffnet wurde)_
* **Verarbeitung starten / Verarbeitung stoppen** — die Bildverarbeitungs-Pipeline starten oder stoppen _(aktiviert, nachdem Dateien hinzugefügt wurden)_

{% hint style="info" %}
**Nur Windows**: Die Chloros-Desktop-GUI ist auf Windows verfügbar. Benutzer von Linux sollten die [CLI](CLI.md) und [Python SDK](api-python-sdk.md) Dokumentation zur Headless-Verarbeitung.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Wiedergabe-/Start-Schaltfläche

Wenn aktiviert, startet die Schaltfläche „Verarbeitung starten“ die Bildverarbeitungs-Pipeline.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Fortschrittsbalken <img src=".gitbook/assets/image (5).png" alt="" data-size="line">Im kostenlosen Modus Chloros, der alle Dateien nacheinander verarbeitet, zeigt der Fortschrittsbalken zwei Phasen an: Zielerkennung und Verarbeitung.

Im kostenpflichtigen Modus Chloros+, der alle Dateien gleichzeitig verarbeitet, zeigt der Fortschrittsbalken vier Phasen an: Erkennen, Analysieren, Kalibrieren, Exportieren. Wenn Sie den Mauszeiger über den Chloros+-Fortschrittsbalken bewegen, wird das erweiterte 4-stufige Fortschrittsbalken-Panel angezeigt, sodass Sie den Fortschritt verfolgen können. Durch Klicken auf den oberen Fortschrittsbalken wird das Dropdown-Panel fixiert, durch erneutes Klicken wird es wieder freigegeben.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Seitenmenü

Das Menü in der linken Seitenleiste enthält verschiedene Symbole, mit denen Sie interagieren können:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projekteinstellungen](project-settings/project-settings.md)

Auf der Registerkarte „Projekteinstellungen“ können Sie globale Projekteinstellungen und Einstellungen für die Projektverarbeitung anpassen. Nehmen Sie diese Anpassungen vor, bevor Sie mit der Verarbeitung Ihrer Dateien beginnen.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Dateibrowser

Fügen Sie Dateien/Ordner hinzu und entfernen Sie Dateien aus dem Projekt. Doppelte Dateien werden ignoriert. Aktivieren Sie das Kontrollkästchen in der Zielspalte für ein beliebiges Zielbild, und die Verarbeitung berücksichtigt nur die markierten Bilder als Ziele, was die Verarbeitungszeit erheblich verkürzt. Verwenden Sie die Umschaltfläche „Bild/Metadaten“, um zwischen der Anzeige des Miniaturbildrasters des ausgewählten Bildes und einer detaillierten Metadatentabelle zu wechseln.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Bildbetrachter](image-viewer-gui/opening-an-image-full-screen.md)

Wenn Sie im Hauptbildbetrachter auf ein Bild klicken, wird es im Vollbildmodus auf der Registerkarte „Bildbetrachter“ geöffnet.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Karte](image-viewer-gui/map-markers.md)

Zeigen Sie Ihre Bilder auf einer interaktiven 2D-Karte basierend auf ihren GPS-Koordinaten an. Unterstützt Google Maps und ESRI-Kachelanbieter und wählt automatisch den besten Dienst für Ihren Standort aus. Bewegen Sie den Mauszeiger über Markierungen, um Vorschaubilder der Bilder anzuzeigen.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Debug-Protokoll

Überprüfen Sie das Protokoll auf Debug-Ausgaben, wenn Probleme auftreten. Kopieren/laden Sie das Protokoll herunter und senden Sie es an den [MAPIR-Support](https://www.mapir.camera/community/contact), um Unterstützung zu erhalten.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Benutzeranmeldung](chloros+-login.md)

Über die Seitenleiste „Benutzeranmeldung“ können Sie sich bei Ihrem Chloros+-Konto anmelden, um erweiterte Funktionen freizuschalten. Sie können dort auch die aktuelle Anwendungsversion einsehen sowie die Sprache der angezeigten Texte in der Chloros-Benutzeroberfläche und in CLI anpassen.
