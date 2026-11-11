# Index/LUT-Sandbox

Die Index/LUT-Sandbox ist der interaktive Arbeitsbereich in der Seitenleiste des Chloros-Bildbetrachters. Sie wählen eine Formel aus, ordnen ihr die Kanäle Ihrer Kamera zu, fügen einen Farbverlauf hinzu und passen den Wertebereich an – und das Bild wird dabei in Echtzeit aktualisiert. Seit Version 1.2.0 können Sie **Ihre Einstellungen** auch speichern, entweder für ein einzelnes Bild oder für das gesamte Projekt, ohne die Bilder erneut verarbeiten zu müssen.

## Wozu dient die Sandbox?

| Index/LUT-Sandbox (interaktiv)        | Projektverarbeitung (Stapelverarbeitung)       |
| -------------------------------------- | -------------------------------- |
| Jeweils ein Bild, sofortiges Feedback  | Der gesamte Datensatz in einem Durchgang     |
| Experimentell und iterativ             | Vorkonfigurierte Einstellungen          |
| Rendert live; speichert nur auf Wunsch  | Schreibt immer Ausgabedateien      |
| Perfekt, um die richtigen Einstellungen zu finden | Am besten, sobald die Einstellungen endgültig sind |

{% hint style="success" %}
**Der übliche Arbeitsablauf**: Passen Sie die Einstellungen in der Sandbox so lange an, bis die Visualisierung Ihren Vorstellungen entspricht, und exportieren Sie dann entweder direkt aus der Sandbox oder kopieren Sie die gleichen Index- und LUT-Einstellungen in die [Projekteinstellungen](../project-settings/project-settings.md), damit sie beim nächsten Verarbeitungsdurchlauf in jedes Bild eingebacken werden.
{% endhint %}

***

## Die Sandbox öffnen

1. Klicken Sie auf ein Bild im Raster – es wird im Vollbildmodus im **Bildbetrachter** auf der Registerkarte „<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">“ geöffnet
2. Klicken Sie auf das Symbol **Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">, um die linke Seitenleiste auszuklappen, falls diese noch nicht geöffnet ist
3. Wählen Sie eine Multiband-Ebene aus dem Ebenen-Dropdown-Menü oben rechts aus – **RAW (Reflexionsgrad)** ist die übliche Wahl, da auf der Grundlage des kalibrierten Reflexionsgrads berechnete Indexwerte zwischen den Bildern vergleichbar sind

Die Seitenleiste zeigt von oben nach unten:

* den Bildnamen und das Kameramodell
* die Schaltfläche **Bild(er) exportieren/speichern** – erscheint, sobald „Index“ oder „LUT“ angekreuzt ist
* die Kontrollkästchen **Index**und**LUT**
* das Konfigurationsfenster für den Index
* das Fenster **Cursor-Werte** mit der Anzeige, dem Histogramm und der GSD-Steuerung

{% hint style="warning" %}
**Nicht verfügbar für Mono-Kameras.** Bei einem Single-Band-LATTICE-M3M-Bild sind beide Kontrollkästchen deaktiviert, mit dem Tooltip _„Nicht verfügbar für Mono- (M3M-) Sensoren“_ — ein Multiband-Index ist auf einem Band nicht definiert. Um Indizes aus M3M-Kameras zu berechnen, kombinieren Sie zwei oder mehr zu einem ausgerichteten Multiband-Stack und verwenden Sie die LATTICE-Index-Engine.
{% endhint %}

***

## Einen Index anwenden

1. Aktivieren Sie das Kontrollkästchen **Index** oben in der Seitenleiste.
2. Wählen Sie den Filter Ihrer Kamera aus dem Dropdown-Menü auf der linken Seite aus (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Wählen Sie eine Indexformel aus dem rechten Dropdown-Menü aus – 27 integrierte Formeln sowie alle benutzerdefinierten Formeln, die Sie gespeichert haben
4. Die Formel wird unten als mathematische Formel dargestellt, mit einem leeren Kreis an jedem Band-Slot. **Ziehen Sie einen farbigen Kanalkreis auf einen Platz**, um ihn zuzuweisen.
5. Sobald jeder von der Formel verwendete Platz zugewiesen ist, wird das Bild aktualisiert und zeigt Indexwerte an.
6. Bewegen Sie den Cursor über das Bild, um Werte abzulesen; das Fenster **Cursorwerte** fügt eine Indexzeile mit dem Wert unter dem Cursor hinzu.

Doppelklicken Sie auf einen verknüpften Platz, um ihn zu löschen. Eine unvollständige Formel ist ein normaler Zustand während des Ziehens und kein Fehler – das Bild wird einfach erst aktualisiert, wenn die Formel vollständig ist.

Die Kanalkreise sind farblich gekennzeichnet: rot = Red, grün = Green, blau = Blue, orange = Orange, Cyan = Cyan, Violett = NIR, Magenta = RE. Die gleichen Farben werden für die Kanalkreise und die Histogrammkurven im Bereich „Cursor-Werte“ verwendet.

### Beispiel NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Die vollständige Formelübersicht – alle drei voreingestellten Listen und welche Namen wo funktionieren – finden Sie unter [Multispektrale Indexformeln](../project-settings/multispectral-index-formulas.md).

### Mit aktivierter Option „Index“, aber ohne LUT

Das Bild wird in **Graustufen** dargestellt und zwischen den beiden Schwellenwerten gestreckt. Dies ist beabsichtigt: Das Indexbild besteht aus Skalardaten, und Graustufen sind die originalgetreue Darstellung dieser Daten. Fügen Sie eine LUT hinzu, wenn Sie Farbe wünschen.***

## Arbeiten mit LUTs (Look-Up-Tabellen)

Eine **Look-Up-Tabelle** ordnet Indexwerte Farben zu: Eingabe NDVI 0,65, Ausgabe ein bestimmtes Grün. Sie verändert nicht die Daten – sie verändert lediglich, wie Sie diese lesen.

### Eine LUT hinzufügen

1. Klicken Sie unterhalb der Formel auf die Schaltfläche **„+ LUT hinzufügen“** (<img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">)
2. Wählen Sie einen Farbverlauf aus
3. Legen Sie den minimalen und maximalen Clipping-Bereich fest
4. Wählen Sie einen Clipping-Modus
5. Aktivieren Sie das Kontrollkästchen **LUT** in der Seitenleiste, um die Darstellung zu aktivieren

Das Kontrollkästchen **LUT** bleibt deaktiviert, bis tatsächlich eine LUT im Index konfiguriert wurde.

### Einen Farbverlauf auswählen

Bewegen Sie den Mauszeiger über die **Verlaufsleiste**, um die Voreinstellungsliste zu öffnen – Chloros enthält**sieben** Verlaufsvoreinstellungen:

| # | Farbverlauf                            | Form                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Gelb → Green (**Standard**)  | Divergierend – entspricht der üblichen Vorstellung von Vegetation: Grün = gesund |
| 2 | Violett → Gelb → Green             | Divergierend, mit einem deutlich ausgeprägten unteren Bereich                                  |
| 3 | Braun → Weiß → Blue                | Abweichend um einen hellen Mittelwert herum                                   |
| 4 | Schwarz → Violett → Rosa → Hellgelb | Sequenziell, von dunkel nach hell                                           |
| 5 | Red → Gelb → Blue                 | Abweichend um einen hellen Mittelpunkt herum                                   |
| 6 | Violett → Blue → Green → Gelb      | Sequenziell, dunkel nach hell                                           |
| 7 | Orange → Weiß → Violett             | Divergierend um einen hellen Mittelpunkt                                   |

Ein **divergierender**Farbverlauf platziert eine neutrale Farbe in der Mitte Ihres Fensters, was gut lesbar ist, wenn der Mittelpunkt eine bestimmte Bedeutung hat (ein Schwellenwert, ein Stichtag). Ein**sequenzieller** Farbverlauf verläuft monoton von dunkel nach hell, was gut lesbar ist für eine Größe, bei der es nur „mehr“ und „weniger“ gibt.

Jede Voreinstellung verfügt über sieben Farbstufen. Klicken Sie auf eine Voreinstellung, und das Bild wird sofort aktualisiert (sofern das Kontrollkästchen „LUT“ aktiviert ist).

### Bearbeiten der Farbstufen

Unterhalb der Farbverlaufsleiste befindet sich eine Reihe von Farbfeldern, eines pro Farbstufe:

* **Farbe ändern**: Klicken Sie auf ein Farbfeld, um die Farbauswahl zu öffnen (Farbrad, RGB/HSV-Schieberegler oder einen Hex-Code wie `#FF0000`)
* **Eine Stufe hinzufügen**: Klicken Sie auf die Schaltfläche**+** am Ende der Reihe – eine weiße Stufe wird angehängt
* **Eine Stufe entfernen**:**Doppelklicken** Sie auf das Farbfeld
* **Einen bearbeiteten Farbverlauf beibehalten**: Klicken Sie auf das Speichersymbol neben der Farbverlaufsleiste, um Ihren bearbeiteten Farbverlauf zur Voreinstellungsliste hinzuzufügen, damit Sie ihn erneut auswählen können

Der Farbverlauf, den Sie für einen Index konfiguriert haben, wird zusammen mit diesem Index in den Projekteinstellungen gespeichert, sodass er auch beim Schließen und erneuten Öffnen des Projekts erhalten bleibt.

**Weniger Stufen**erzeugen deutlich voneinander abgegrenzte Bereiche, die als Klassifizierung lesbar sind;**mehr Stufen** erzeugen sanfte, nahezu fotografische Übergänge. Drei bis fünf Stufen eignen sich für Präsentationsfolien und Klassifizierungskarten; sechs bis zehn für allgemeine Analysen; fünfzehn oder mehr für detaillierte Untersuchungen und Publikationsabbildungen.

### Festlegen des Wertebereichs

Der Schwellenwertregler ist ein **Schieberegler mit zwei Griffen**, der von −1 bis +1 reicht und an jedem Ende ein bearbeitbares Textfeld für genaue Werte sowie eine**AUTO**-Schaltfläche aufweist.

* Ziehen Sie einen der Schieberegler oder geben Sie eine Zahl in das entsprechende Feld ein und drücken Sie die Eingabetaste
* **AUTO**legt den Bereich auf das**

2. und 98. Perzentil** der gültigen Indexwerte des Bildes – ein guter Ausgangspunkt, der Ausreißer ignoriert. Chloros rundet das Ergebnis adaptiv auf 4 Dezimalstellen bei einem sehr engen Bereich, auf 3 bei einem engen Bereich und ansonsten auf 2
* Jede manuelle Anpassung hat Vorrang vor „AUTO“, bis Sie erneut auf „AUTO“ klicken

Beispiel für NDVI-Fenster:

| Ziel                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Alles anzeigen                         | −1,0 | 1,0 |
| Nur Vegetation, Boden und Wasser ausschließen | 0,2  | 0,9 |
| Nur gesunde Vegetation                 | 0,5  | 0,9 |
| Stress hervorheben                        | 0,2  | 0,5 |

Durch die Verengung des Fensters wird der Kontrast innerhalb Ihres Untersuchungsbereichs erhöht und alles andere wird aus dem Bereich verdrängt – wobei der **Clipping-Modus** entscheidet, was damit geschieht.***

## Clipping-Modi

Wenn der Indexwert eines Pixels außerhalb des Min/Max-Fensters liegt, bestimmt der Clipping-Modus, wie es gezeichnet wird.

| Dropdown-Bezeichnung                  | Gespeicherter Wert      | Pixel außerhalb des Bereichs werden gezeichnet als                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum & Maximum** (Standard) | `clip`            | Die nächstgelegene Endfarbe des Farbverlaufs – Werte unterhalb des Minimums erhalten die erste Farbe, Werte oberhalb des Maximums die letzte |
| **Transparenter Hintergrund**      | `transparent`     | Vollständig transparent (echtes Alpha)                                                                                                  |
| **Index-Hintergrund**| `indexColor`      | Graustufen, gestreckt über den**gesamten** Indexbereich des Bildes, sodass Strukturen außerhalb des Bereichs weiterhin in Grau sichtbar sind                |
| **Originalhintergrund**         | `backgroundColor` | Das zugrunde liegende Bild selbst, sodass die Farbüberlagerung über der tatsächlichen Szene liegt                                                |

| Modus                       | Am besten geeignet für                               | Aussehen                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimum & Maximum**      | Vollständige Datenanzeige, wissenschaftliche Analyse | Jedes Pixel ist eingefärbt                      |
| **Transparenter Hintergrund** | GIS-Überlagerungen, Isolierung eines Wertebereichs   | Farbe innerhalb des Fensters, nichts außerhalb |
| **Index-Hintergrund**       | Hervorhebung unter Beibehaltung des Datenkontexts    | Farbe im Inneren, grau außerhalb               |
| **Original-Hintergrund**    | Berichte und Präsentationen              | Farbe im Inneren, Foto außerhalb         |

{% hint style="info" %}
**Pixel ohne Daten sind in jedem Modus stets transparent.** Ein Pixel, dessen Index nicht endlich ist (eine Division durch 0) oder genau −1,0 oder +1,0 beträgt (Sättigungsgrenzwerte, wenn ein Band den Wert Null aufweist, das andere jedoch nicht), wird als „keine Daten“ und nicht als Extremwert behandelt. Dadurch werden überbelichtete Bereiche und völlig abgedunkelte Schatten aus Ihrer Farbskala herausgehalten, anstatt sie als den extremsten Messwert im Bild darzustellen. Dieselbe Regel legt fest, welche Pixel in die AUTO-Schwellenwerte und das Indexhistogramm einfließen, sodass alle drei übereinstimmen.
{% endhint %}

Die Transparenz bleibt erhalten, wenn der Export im Format PNG erfolgt. Sie kann im JPG-Format nicht dargestellt werden.

***

## Werte ablesen während der Feinabstimmung

Das Bedienfeld **Cursorwerte** unterhalb des Konfigurationsfensters dient als Messinstrument für die Sandbox:

* Bewegen Sie den Cursor über das Bild und lesen Sie die Quellwerte pro Kanal sowie den Indexwert in der entsprechenden Zeile ab
* Aktivieren Sie die Schaltfläche **INDEX** oberhalb des Histogramms, um die Verteilung der Indexwerte im Bild zu sehen, wobei Ihre beiden Clip-Schwellenwerte als orangefarbene gestrichelte Linien und der Cursorwert als weiße Linie dargestellt werden – dies ist der schnellste Weg, um ein Fenster auszuwählen, das tatsächlich Ihre Daten enthält
* Aktivieren Sie **CURSOR**, um Markierungslinien an den Werten unter dem Zeiger anzuzeigen
* Zoomen Sie auf eine Vergrößerung von über 60× (weniger, wenn eine GSD-Blockgröße festgelegt ist), um einzelne angezeigte Pixel mit einem schwebenden Wert hervorzuheben

Eine praktische Vorgehensweise:

1. Notieren Sie die Werte über gesunder Vegetation, gestresster Vegetation, nacktem Boden und Wasser
2. Sehen Sie sich an, wo sich diese Cluster im Indexhistogramm befinden
3. Legen Sie Min./Max. fest, um den für Sie relevanten Cluster einzugrenzen
4. Wählen Sie einen Ausschnittmodus – _Original Hintergrund_ hält die Umgebung der Szene sichtbar

***

## Exportieren aus der Sandbox

Alles oben Genannte ist eine Live-Vorschau, bis Sie es speichern. Die Schaltfläche **„Bild(er) exportieren/speichern“** oben in der Seitenleiste öffnet einen Bereich, der über die Seitenleiste gleitet (anstatt das Bild zu verdecken, sodass Sie weiterhin sehen können, worüber Sie gerade entscheiden).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Optionen

| Option                          | Wirkung                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Auf aktuelles Bild anwenden**      | Speichert genau das angezeigte Bild mit diesen Einstellungen                                                                                                |
| **Auf alle Projektbilder anwenden** | Wendet die identische Konfiguration erneut auf jedes Bild im Projekt an. Bilder, denen die für diesen Index erforderlichen Bänder fehlen, werden übersprungen und nicht als Fehler behandelt |
| **Index-/LUT-Farbverlaufsbalken**      | Schreibt außerdem pro Export ein separates Legendenbild, auf dem der Wertebereich beschriftet ist                                                                     |
| **Index-Histogramm**             | Schreibt außerdem pro Export ein separates Histogrammbild, das die Daten-Min/Max-Werte und die Clip-Schwellenwerte anzeigt                                               |

Wenn die **GSD-Blockgröße** auf der Registerkarte „Bild“ größer als 1 ist, wird dies im Fenster vor dem Speichern angezeigt: Der Export speichert das, was Sie gerade sehen, einschließlich Blockmittelung. Setzen Sie den GSD-Regler zunächst wieder auf 1 zurück, wenn Sie die volle Auflösung wünschen.

### Speicherort der Dateien

Bei jedem Klick auf **Exportieren**wird ein**neuer, nie wiederverwendeter Ordner** angelegt:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Beispiele: `Sandbox_Exports/NDVI_LUT_001/`, dann `Sandbox_Exports/NDVI_LUT_002/` für den nächsten Durchlauf. Die Nummerierung wird durch das Scannen der bereits auf der Festplatte vorhandenen Dateien ermittelt, sodass sie auch bei Neustarts und manuell gelöschten Ordnern erhalten bleibt. Es wird niemals etwas überschrieben – der Sinn der Sandbox besteht ja gerade darin, einen Versuch mit dem letzten zu vergleichen.

Innerhalb des Ordners, pro Bild:

| Datei                                                   | Inhalt                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Das gerenderte Bild, Pixel für Pixel so, wie es der Viewer angezeigt hat |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Die Begleitdatei mit dem Farbverlauf, falls angefordert                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Die Begleitdatei mit dem Indexhistogramm, falls angefordert                  |

Die beiden Begleitdateien werden immer in **voller Auflösung** geschrieben, selbst wenn das Hauptbild blockweise gemittelt wird: Eine Blockgröße entspricht einer Bildschirmauflösung, und beide Begleitdateien enthalten die tatsächlichen Indexwerte pro Pixel. Außerdem enthalten sie mehr Informationen als die Bildschirmversionen – beide vermerken sowohl das Stretch-Fenster _als auch_ die tatsächlichen Minimal- und Maximalwerte der Daten, sodass eine gespeicherte Legende auch Monate später noch lesbar ist, ohne dass das Projekt geöffnet sein muss.

### Fortschritt und Ergebnisse

Ein Export des gesamten Projekts dauert nur wenige Minuten, daher meldet der Lauf den Fortschritt über einen Live-Fortschrittskanal zurück, anstatt den Systemablauf zu blockieren:

* Ein Fortschrittsbalken zeigt „`current / total`“ und die Datei an, die gerade geschrieben wird
* Nach Abschluss des Vorgangs zeigt der Bereich an, wie viele Bilder exportiert und wie viele übersprungen wurden, sowie den Pfad zum Ausgabeordner
* Übersprungene Bilder werden mit dem Grund aufgelistet (bis zu fünf werden angezeigt, danach folgt die Zeile „+N weitere“). Der übliche Grund ist eine Ebene, die nicht über die Kanäle verfügt, die dieser Index benötigt
* Wenn **kein** Bild im Projekt den Index verwenden konnte, meldet der Durchlauf einen Fehler, anstatt einen leeren Ordner zu hinterlassen

Es läuft jeweils nur ein Sandbox-Export gleichzeitig. Der Start eines zweiten Durchlaufs, während ein erster noch läuft, wird mit einer eindeutigen Meldung abgelehnt, anstatt zuzulassen, dass zwei Durchläufe um dieselbe Projektdatei konkurrieren.

### Das Raster nimmt den Lauf auf

Jeder abgeschlossene Lauf erscheint als eigene Schaltfläche in der [Bildraster](image-grid.md)-Symbolleiste mit der Bezeichnung `<IndexName> <Index|LUT> <NNN>`. So vergleichen Sie Durchläufe: Führen Sie zwei Exporte mit unterschiedlichen Gradienten oder Schwellenwerten durch und wechseln Sie dann zwischen den beiden Schaltflächen im Raster hin und her.

***

## Benutzerdefinierte Indexformeln (Chloros+)

{% hint style="info" %}
**Wo man sie erstellt**: in der Sandbox-Seitenleiste oder in den**Projekteinstellungen** vor der Verarbeitung. Beide schreiben in dieselbe Liste auf Projektebene.
{% endhint %}

1. Öffnen Sie den Rechner für benutzerdefinierte Formeln über das Dropdown-Menü „Indexformel“ (erfordert die Anmeldung mit einem berechtigten Chloros+-Abonnement)
2. Geben Sie die Formel unter Verwendung der **Band-Slot-Symbole** `x`, `y`, `z`, `a`, `b`, `c` – keine Bandnamen
3. Verfügbare Operatoren: `+`, `-`, `*`, `/`, `^` und `()` zum Gruppieren
4. Verfügbare Funktionen: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Benennen und speichern Sie sie – sie erscheint unten im Formeln-Dropdown-Menü, und Sie binden ihre Slots durch Ziehen von Kanalkreisen, genau wie bei einem integrierten Preset

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Benutzerdefinierte Formeln sind nur über die Benutzeroberfläche verfügbar.** Die Option „CLI/SDK `--indices`“ erweitert die 22 integrierten Voreinstellungsnamen und überspringt im Hintergrund alles andere, einschließlich Ihrer benutzerdefinierten Formeln. Um eine benutzerdefinierte Formel stapelweise anzuwenden, konfigurieren Sie sie in den Projekteinstellungen und führen Sie die Verarbeitung durch, oder nutzen Sie den Export „Auf alle Projektbilder anwenden“ der Sandbox.
{% endhint %}

***

## Fehlerbehebung

### „Diese Ebene verfügt nicht über die Kanäle, die dieser Index benötigt“

Die Formel liest eine Kanalposition aus, über die die aktuelle Ebene nicht verfügt – beispielsweise einen Index mit drei Slots bei einer Ein- oder Zweikanaldatei. Wechseln Sie zu einer Multiband-Ebene (Reflexionswert oder debayert) oder wählen Sie einen Index, der zum Filter Ihrer Kamera passt.

### „Das Backend für die Bildverarbeitung konnte nicht erreicht werden“

Das Backend antwortet nicht. Überprüfen Sie die Registerkarte „Protokoll“; wenn das Backend neu gestartet wird, stellt sich die Sandbox von selbst wieder her, sobald es wieder verfügbar ist.

### Das Bild hat sich nicht verändert, als ich einen Kreis gezogen habe

Die Formel ist noch nicht vollständig. Eine unvollständige Formel wird wie ein normaler Zustand während des Ziehens behandelt – es wird nichts gerendert und es wird kein Fehler gemeldet. Füllen Sie alle Felder aus, die die Formel verwendet.

### Das gesamte Bild hat nur eine Farbe

Ihr Clip-Fenster liegt wahrscheinlich weit außerhalb der Daten. Drücken Sie **AUTO**, um es an das 2. bzw. 98. Perzentil anzupassen, oder aktivieren Sie das**INDEX**-Histogramm, um zu sehen, wo die Daten tatsächlich liegen.

### Die exportierten Farben stimmen nicht mit dem überein, was ich gesehen habe

Das sollten sie aber – der Exportpfad spiegelt bewusst die Live-Vorschau wider, einschließlich des Alpha-Werts im Clipping-Modus, und die Blockmittelung wird _nach_ der Kolorierung angewendet, genau wie es der Viewer tut. Falls Abweichungen auftreten, überprüfen Sie, ob sich die GSD-Blockgröße zwischen der Anzeige und dem Export nicht geändert hat.

***

## Nächste Schritte

* [**Bildebenen**](image-layers.md) – auf welcher Ebene ein Index berechnet werden soll und was dessen Werte bedeuten
* [**Ein Bild im Vollbildmodus öffnen**](opening-an-image-full-screen.md) – die Cursoranzeige, das Histogramm und die GSD-Steuerung im Detail
* [**Formeln für multispektrale Indizes**](../project-settings/multispectral-index-formulas.md) – jede Voreinstellung auf jeder Oberfläche
* [**Projekteinstellungen**](../project-settings/project-settings.md) – Speichern der gewählten Einstellungen für einen Verarbeitungsdurchlauf
