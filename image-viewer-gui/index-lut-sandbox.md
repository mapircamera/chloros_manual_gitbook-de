# Index/LUT-Sandbox

Die Index/LUT-Sandbox ist ein interaktiver Arbeitsbereich innerhalb des Chloros Image Viewers, in dem Sie in Echtzeit mit multispektralen Indexberechnungen und Farbvisualisierungen experimentieren können. Mit diesem leistungsstarken Tool können Sie verschiedene Indizes testen, Wertebereiche verfeinern und publikationsreife Visualisierungen erstellen, ohne Ihren gesamten Datensatz erneut verarbeiten zu müssen.

## Was ist die Index/LUT-Sandbox?

### Zweck

Die Sandbox bietet:

* **Echtzeit-Indexberechnung** – Wenden Sie jeden Vegetationsindex sofort an.
* **Interaktive LUT-Anpassung** – Optimieren Sie Farbverläufe und -bereiche.
* **Workflow-Optimierung** – Ermitteln Sie die besten Einstellungen vor der Stapelverarbeitung.

### Sandbox vs. Projektverarbeitung

**Index/LUT-Sandbox (interaktiv):**

* Ein einzelnes Bild nach dem anderen
* Sofortiges Feedback
* Experimentell und iterativ
* Keine dauerhaften Änderungen an Dateien
* Perfekt zum Erkunden und Testen

**Projektverarbeitung (Batch):**

* Gesamter Datensatz auf einmal
* Vorkonfigurierte Einstellungen
* Permanente Ausgabedateien
* Zeitaufwändig
* Am besten geeignet, wenn die Einstellungen endgültig festgelegt sind

{% hint style=&quot;success&quot; %}
**Optimaler Arbeitsablauf**: Verwenden Sie die Sandbox, um zu experimentieren und die optimalen Index- und LUT-Einstellungen zu finden, und wenden Sie diese Einstellungen dann während der Projektverarbeitung auf Ihren gesamten Datensatz an.
{% endhint %}

***

## Arbeiten mit der Index-/LUT-Sandbox

### Vorberechnete Indizes verstehen

In Chloros können Indizes während der Projektverarbeitung angewendet werden. Um zu bestimmen, welche Index- und LUT-Einstellungen Sie auf Exporte anwenden möchten, verwenden Sie am besten die Bildbetrachtungs-Sandbox.

Mit der Sandbox können Sie:

* **Neue Indizes und Farbverläufe (LUTs)** anwenden, um die Daten zu visualisieren
* **Visualisierungseinstellungen** interaktiv anpassen
* Bereits berechnete Indexbilder **anzeigen**
* Pixelwerte auf allen Zoomstufen **überprüfen**

### Öffnen der Sandbox

Die Index-/LUT-Sandbox finden Sie in der **Bildbetrachtung** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Klicken Sie auf ein Bild im Bildraster des Dateibrowsers, um es im **Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Klicken Sie auf die Registerkarte **„Bildbetrachter“** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , um die linke Popout-Seitenleiste zu öffnen, falls sie noch nicht geöffnet ist.

### Auswählen eines Bildes, auf das ein Index/LUT angewendet werden soll

Um mit einem Index in der Sandbox des Bildbetrachters zu arbeiten <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> Sandbox zu arbeiten:

1. **Öffnen Sie ein Bild** aus dem Hauptbildraster, indem Sie darauf klicken.
2. Die Registerkarte **Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
3. Klicken Sie auf das **Dropdown-Menü „Ebene“** (oben rechts im Viewer).
4. Wählen Sie die Ebene aus dem Dropdown-Menü aus:
   * RAW (Reflexion)

### Anwenden eines Index auf ein Bild

Sobald das Bild im Vollbildmodus angezeigt wird und die Seitenleiste „**Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> geöffnet ist:

1. Aktivieren Sie das Kontrollkästchen „Index“ oben in der Seitenleiste.
2. Wählen Sie den Filter Ihrer Kamera aus dem Dropdown-Menü auf der linken Seite aus.
3. Wählen Sie die gewünschte Indexformel aus dem Dropdown-Menü auf der rechten Seite aus.
4. Ziehen Sie die Farbcirkel des Filterkanals an die entsprechenden Stellen in der Indexformel unten.
5. Sobald die Formel gültig ist, wird das Bild aktualisiert und zeigt die Indexwerte an.
6. Bewegen Sie den Mauszeiger, um die Werte an der Position des Mauszeigers anzuzeigen.
7. Vergrößern Sie das Bild, um einzelne Pixel und die zugehörigen Werte anzuzeigen.

Jeder Index hat einen bestimmten Wertebereich und eine bestimmte Bedeutung:

#### NDVI Beispiel

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Eine vollständige Dokumentation der Indexformeln finden Sie unter [Multispektrale Indexformeln](../project-settings/multispectral-index-formulas.md).

***

## Arbeiten mit LUTs (Look-Up-Tabellen)

### Was ist eine LUT?

Eine **Look-Up-Tabelle (LUT)** ordnet numerische Indexwerte Farben zur Visualisierung zu:

* **Eingabe**: Index-Pixelwert (z. B. NDVI 0,65)
* **Ausgabe**: RGB-Farbe (z. B. hellgrün)
* **Zweck**: Muster leichter erkennbar und interpretierbar machen

**Graustufen- vs. Farblut:**

* Graustufen: Wissenschaftlich und neutral, zeigt Rohdaten
* Farblut: Intuitiv und wirkungsvoll, hebt Muster und Unterschiede hervor

{% hint style=&quot;success&quot; %}
**Visualisierungsleistung**: Durch die Anwendung einer Farblut auf ein Graustufen-Indexbild lassen sich Muster, Anomalien und interessante Bereiche auf einen Blick deutlich leichter erkennen.
{% endhint %}

### Anwenden einer LUT auf ein Indexbild

Sobald Sie ein Indexbild haben, das Folgendes anzeigt

1. Klicken Sie auf die Schaltfläche <img src="../.gitbook/assets/image.png" alt="" data-size="line"> Schaltfläche „+LUT hinzufügen“
2. Wählen Sie den Farbverlauf aus
3. Passen Sie die minimalen/maximalen Endpunkte der Begrenzung an
4. Passen Sie den Begrenzungsmodus an
5. Aktivieren Sie das Kontrollkästchen „Index“ in der **Bildanzeige** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , um die LUT anzuwenden.

### Auswahl eines Farbverlaufs

**Auswahl eines Farbverlaufs:**

1. Suchen Sie im LUT-Bedienfeld die **farbige Verlaufsleiste**.
2. Bewegen Sie den Mauszeiger darüber, um die verfügbaren Verlaufsvoreinstellungen anzuzeigen.
3. Wählen Sie den gewünschten Verlauf aus.
4. Das Bild wird **sofort** mit neuen Farben aktualisiert, wenn das Kontrollkästchen „Index“ aktiviert ist.

{% Hinweis style=&quot;success&quot; %}
**Bewährte Vorgehensweise**: Für Vegetationsindizes wie NDVI ist der Farbverlauf Red-Gelb-Green am intuitivsten, da er den natürlichen Farbassoziationen entspricht (grün = gesund, gelb = mäßig, rot = gestresst).
{% endhint %}

### Anpassen der Farbklassen

Mit der **Steuerung „Klassen”** legen Sie fest, wie viele diskrete Farbstufen in Ihrem Farbverlauf angezeigt werden:

**Optionen für die Klassenanzahl:**

* **2–5 Klassen**: Sehr breite Kategorien, deutliche Zonen
* **6–10 Klassen**: Ausgewogen, gut für die Klassifizierung geeignet
* **11–20 Klassen**: Sanfte Farbverläufe, kontinuierliches Erscheinungsbild
* **20+ Klassen**: Nahezu kontinuierlich, maximale Sanftheit

**So nehmen Sie die Anpassung vor:**

1. Suchen Sie im LUT-Bedienfeld die **Farbquadrate unterhalb der Farbverlaufsleiste**.
2. Passen Sie die Anzahl der Klassen an, indem Sie mit der Schaltfläche „+“ weitere hinzufügen.
3. Entfernen Sie die Anzahl der Klassen durch Doppelklicken auf ein Farbfeld.
4. Der Farbverlauf wird **in Echtzeit** auf dem Bild aktualisiert.

**Auswirkung auf die Visualisierung:**

* **Weniger Klassen** (3–5): Erzeugt eindeutige Zonen, vereinfachte Klassifizierung, leichter zu unterscheidende Kategorien
* **Mittlere Klassen** (6–10): Ausgewogener Ansatz, gut für die meisten Anwendungen geeignet
* **Mehr Klassen** (15–20): Sanfte Übergänge, detaillierte Variationen, fotografisches Erscheinungsbild

**Verwendungszweck:**

* **Wenige Klassen (3–5)**: Präsentationsfolien, Klassifizierungskarten, einfache Berichte
* **Mittlere Klassen (6–10)**: Allgemeine Analyse, ausgewogene Details, Standardberichte
* **Viele Klassen (15–20)**: Wissenschaftliche Analysen, detaillierte Untersuchungen, Ergebnisse in Publikationsqualität

### Feinabstimmung der Wertebereiche

Mit den **Wertebereichssteuerungen** legen Sie fest, welche Indexwerte welchen Farben in Ihrem Farbverlauf zugeordnet werden:

**Bereichssteuerungen im LUT-Bedienfeld:**

* **Mindestwert**: Untergrenze der Farbskala
* **Höchstwert**: Obergrenze der Farbskala
* **Zwischenwerte**: Werden automatisch zwischen Minimum und Maximum verteilt (basierend auf der Klassenanzahl)

#### Anpassen der Min-/Max-Werte

**So passen Sie die Wertebereiche an:**

1. Suchen Sie im LUT-Bedienfeld die Eingabefelder **Minimalwert** und **Maximalwert**.
2. Klicken Sie auf das Feld **Minimalwert**.
3. Geben Sie den gewünschten Minimalwert ein (z. B. `0.2`).
4. Drücken Sie die **Eingabetaste** oder klicken Sie außerhalb des Feldes.
5. Wiederholen Sie den Vorgang für das Feld **Maximalwert** (z. B. `0.9`).
6. Die Visualisierung **wird sofort aktualisiert**.

{% hint style=&quot;info&quot; %}
**Automatische Skalierung**: Wenn Sie eine LUT zum ersten Mal anwenden, stellt Chloros automatisch den Minimal-/Maximalwert auf den tatsächlichen Datenbereich im Bild ein. Sie können diesen Bereich dann eingrenzen, um sich auf bestimmte Wertebereiche zu konzentrieren, die für Sie von Interesse sind.
{% endhint %}

**Beispiel für NDVI-Bereichsanpassungen:**

* **Gesamter Bereich**: `-1.0` bis `1.0` (alle möglichen Werte anzeigen)
* **Auf Vegetation fokussiert**: `0.2` bis `0.9` (nackten Boden und Wasser ausschließen)
* **Nur gesunde Vegetation**: `0.5` bis `0.9` (nur kräftige Pflanzen hervorheben)
* **Stresserkennung**: `0.2` bis `0.5` (Problembereiche hervorheben)
* **Benutzerdefinierter Bereich**: Anpassen basierend auf den beobachteten Pixelwerten

**Warum Bereiche anpassen?**

* **Kontrast erhöhen** in Ihrem Interessengebiet
* **Ausschluss irrelevanter Werte** (z. B. Gewässer, kahler Boden)
* **Standardisierung der Visualisierung** über mehrere Bilder oder Daten hinweg
* **Hervorhebung subtiler Unterschiede** innerhalb eines engen Wertebereichs

### Ausschneiden von Werten außerhalb des Bereichs

Wenn Pixelwerte außerhalb Ihres definierten Min-/Max-Bereichs liegen, können Sie mithilfe von **Ausschnittmodi** steuern, wie sie angezeigt werden.

#### **Verfügbare Clipping-Modus-Optionen:**

#### 1. Minimum und Maximum

* Pixel **unter dem Minimum** → Anzeige mit der **ersten Farbe** im Farbverlauf (z. B. Rot)
* Pixel **über dem Maximum** → Anzeige mit der **letzten Farbe** im Farbverlauf (z. B. Grün)
* **Anwendungsfall**: Hervorhebung von Extremen, Anzeige des gesamten Datenbereichs mit gesättigten Farben an den Grenzen
* **Beispiel**: NDVI-Werte unter 0,2 werden alle rot angezeigt, Werte über 0,9 werden alle grün angezeigt

#### 2. Transparenter Hintergrund

* Pixel **außerhalb des Bereichs** werden **vollständig transparent**
* Nur Pixel **innerhalb des Bereichs** zeigen einen Farbverlauf
* **Anwendungsfall**: GIS-Überlagerung, Isolieren bestimmter Wertebereiche, Hervorheben nur der Bereiche von Interesse
* **Beispiel**: Nur NDVI 0,4–0,7 in Farbe anzeigen, alles andere transparent

{% hint style=&quot;warning&quot; %}
**Transparenzbeschränkung**: Transparente Pixel werden im Viewer als Hintergrundfarbe angezeigt. Beim Export während der Verarbeitung bleibt die Transparenz im PNG-Format erhalten, nicht jedoch im JPG-Format.
{% endhint %}

#### 3. Indexhintergrund

* Pixel **außerhalb des Bereichs** werden in **Graustufen** angezeigt (mit den Rohindexwerten).
* Pixel **innerhalb des Bereichs** zeigen einen **Farbverlauf**
* **Anwendungsfall**: Dezente Hervorhebung, Beibehaltung des Kontexts bei gleichzeitiger Hervorhebung von Bereichen von Interesse
* **Beispiel**: Hervorhebung gestresster Vegetation durch Farbe (NDVI 0,3–0,5) bei gleichzeitiger Darstellung gesunder Bereiche in Grau

#### 4. Originalhintergrund

* Pixel **außerhalb des Bereichs** zeigen das **ursprüngliche multispektrale Bild** an
* Pixel **innerhalb des Bereichs** zeigen einen **Farbverlauf**
* **Anwendungsfall**: Am intuitivsten – kombiniert den natürlichen Bildkontext mit einer analytischen Farbüberlagerung
* **Beispiel**: Sehen Sie das tatsächliche Aussehen des Feldes/der Kulturpflanzen mit farbcodierten Stressbereichen als Überlagerung

### Auswahl des richtigen Clipping-Modus

| Clipping-Modus              | Am besten geeignet für                                   | Visualisierungsstil          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum und Maximum**    | Vollständige Datenanzeige, wissenschaftliche Analyse     | Alle Pixel farbig           |
| **Transparenter Hintergrund** | GIS-Überlagerungen, Isolierung bestimmter Bereiche    | Farbe im Bereich, außerhalb davon leer |
| **Indexhintergrund**       | Dezente Hervorhebung, Beibehaltung des Datenkontexts  | Farbe im Bereich, außerhalb davon grau  |
| **Originalhintergrund**    | Berichte, Präsentationen, intuitive Analyse | Farbe im Bereich, Foto außerhalb |

### Erstellen benutzerdefinierter LUT-Farben

Um die vollständige Kontrolle über Ihre Visualisierung zu haben, können Sie **benutzerdefinierte Farbverläufe** erstellen, indem Sie einzelne Farbstopps bearbeiten.

**So erstellen Sie einen benutzerdefinierten Farbverlauf:**

1. Suchen Sie im LUT-Bedienfeld die **Farbverlauf-Vorschauleiste**
2. Suchen Sie unterhalb des Farbverlaufs nach den **Farbfeldern**.
3. **Klicken Sie auf einen Farbstopp**, um ihn auszuwählen.
4. Ein **Farbwähler** wird geöffnet.
5. Wählen Sie eine neue Farbe aus:
   * **Farbrad**: Visuelle Farbauswahl
   * **RGB/HSV-Schieberegler**: Präzise Farbsteuerung
   * **Hex-Code-Eingabe**: Genaue Farbspezifikation (z. B. `#FF0000` für Rot)
6. Klicken Sie außerhalb des Farbwählers, **um die neue Farbe anzuwenden**.
7. Der Farbverlauf **wird sofort** im Bild aktualisiert.

**Hinzufügen oder Entfernen von Farbstopps:**

* **Einen Stopp hinzufügen**: Klicken Sie auf das Pluszeichen (+), um am Ende ein neues Farbfeld hinzuzufügen.
* **Farbstopp entfernen**: Doppelklicken Sie auf das Farbfeld, um das Farbfeld zu entfernen.

**Anpassungsstrategien:**

* **Farbverlauf umkehren**: Kehren Sie die Farbreihenfolge um, um die Bedeutung umzukehren (z. B. grün = niedrig, rot = hoch).
* **Markenfarben**: Passen Sie die Farbpalette Ihrer Organisation für Berichte an.
* **Farbenblindheitsfreundlich**: Verwenden Sie Orange-Blau- oder Violett-Gelb-Kombinationen.
* **Druckoptimierung**: Wählen Sie Farben, die sowohl für den Farb- als auch für den Graustufendruck geeignet sind.
* **Mehrere Schwellenwerte**: Verwenden Sie unterschiedliche Farben für bestimmte Werteschwellen zur Klassifizierung.

{% hint style=&quot;info&quot; %}
**Speichern benutzerdefinierter Farbverläufe**: Benutzerdefinierte Farbverläufe können gespeichert und wiederverwendet werden. Klicken Sie auf das Speichersymbol im LUT-Bedienfeld, um Ihre benutzerdefinierten Farbschemata für die zukünftige Verwendung zu speichern.
{% endhint %}

***

## Interaktiver Arbeitsablauf

### Echtzeit-Aktualisierungen

Alle LUT-Anpassungen in der Sandbox aktualisieren das Bild **sofort und interaktiv**:

* **Ebene wechseln** → Das Bild ändert sich sofort
* **Farbverlauf auswählen** → Die Farben werden sofort aktualisiert
* **Wertebereich anpassen** → Der Kontrast ändert sich in Echtzeit
* **Klassen ändern** → Die Glätte des Farbverlaufs wird sofort aktualisiert
* **Clipping ändern** → Hintergrundanzeige ändert sich sofort
* **Farben bearbeiten** → Benutzerdefinierter Farbverlauf wird sofort angewendet

**Keine „Anwenden”-Schaltfläche erforderlich** – alle Änderungen sind live und interaktiv!

{% Hinweis style=&quot;success&quot; %}
**Live-Feedback**: Dank des sofortigen visuellen Feedbacks können Sie schnell mit verschiedenen Einstellungen experimentieren, bis Sie die optimale Visualisierung für Ihre Analyseanforderungen gefunden haben.
{% endhint %}

### Iterativer Verfeinerungs-Workflow

**Typischer LUT-Optimierungs-Workflow:**

1. **Indexebene auswählen** (z. B. RAW (Reflexionsgrad))
2. **Index anwenden** – Kamerafilter und Indexformel auswählen, farbige Kreise an die entsprechende Stelle in der Indexformel ziehen
3. **LUT-Gradienten anwenden** – Mit der Voreinstellung Red-Yellow-Green beginnen
4. **Pixelwerte überprüfen** – Bewegen Sie den Cursor und notieren Sie sich die Wertebereiche.
5. **Min/Max anpassen** – Eingrenzen, um sich auf die Vegetation zu konzentrieren (z. B. 0,2 bis 0,9).
6. **Clipping auswählen** – Probieren Sie „Original Background“ für den Kontext aus.
7. **Farben verfeinern** – Passen Sie den Gradienten bei Bedarf an, um bestimmte Bereiche hervorzuheben.
8. **Einstellungen finalisieren** – Dokumentieren Sie die Einstellungen und kopieren Sie sie in die Projekteinstellungen für die Exportverarbeitung.

### Überprüfung der Pixelwerte

Das Verständnis der tatsächlichen Pixelwerte ist entscheidend für die Festlegung effektiver LUT-Bereiche:

**So überprüfen Sie die Werte:**

1. Die Pixelwerte werden angezeigt, wenn entweder das Kontrollkästchen „Index“ oder sowohl das Kontrollkästchen „Index“ als auch das Kontrollkästchen „LUT“ aktiviert sind.
2. **Bewegen Sie den Cursor** über verschiedene Bereiche des Bildes.
3. **Beobachten Sie die Pixelwerte**, die in der Legende angezeigt werden, wenn Sie mit dem Mauszeiger darüber fahren.
4. Vergrößern Sie das Bild, um einzelne Pixel zu sehen, die mit einem schwebenden Wert hervorgehoben sind.
5. **Notieren Sie** sich die Wertebereiche für verschiedene Merkmale:
   * **Gesunde Vegetation**: z. B. NDVI 0,55–0,85
   * **Gestresste Vegetation**: z. B. NDVI 0,30–0,50
   * **Bare Erde**: z. B. NDVI 0,05–0,25
   * **Wasser** (falls vorhanden): z. B. NDVI -0,05 bis 0,10

**Festlegen der LUT-Bereiche anhand von Pixelwerten:**

Passen Sie nach Überprüfung der Pixelwerte die LUT-Min-/Max-Werte entsprechend an:

**Beispielszenario:**

* **Beobachtung**: Bodenwerte = 0,05–0,25, gestresst = 0,25–0,50, gesund = 0,50–0,85
* **Ziel**: Nur die Pflanzengesundheit visualisieren (Boden ausschließen)
* **LUT-Einstellungen**: Min = `0.25`, Max = `0.85`
* **Ausschnitt**: „Original Hintergrund“, um den Boden in seiner natürlichen Farbe zu sehen
* **Ergebnis**: Farbverlauf gilt nur für Vegetation, Boden wird als Originalbild angezeigt

{% hint style=&quot;info&quot; %}
**Dynamikbereich**: Verschiedene Kulturen, Jahreszeiten und Wachstumsstadien haben unterschiedliche Wertebereiche. Überprüfen Sie immer die Pixelwerte in Ihrem spezifischen Datensatz, bevor Sie LUT-Bereiche festlegen.
{% endhint %}

***

## Benutzerdefinierte Indizes (Chloros+)

### Erstellen benutzerdefinierter Indexformeln

{% hint style=&quot;info&quot; %}
**Wo erstellen**: Benutzerdefinierte Indizes können vor der Verarbeitung in den **Projekteinstellungen** sowie in der Seitenleiste der Bildbetrachtungs-Sandbox konfiguriert werden.
{% endhint %}

**So erstellen Sie einen benutzerdefinierten Index:**

1. **Öffnen Sie die Projekteinstellungen** (vor der Verarbeitung) oder die Seitenleiste der Sandbox des Bildbetrachters.
2. Navigieren Sie zum **Dropdown-Menü „Indexformel“**.
3. Suchen Sie nach der Option **„Benutzerdefiniert“** (Sie müssen mit einer Chloros+-Lizenz angemeldet sein).
4. **Definieren Sie Ihre Formel** mithilfe von Bandvariablen:
   * Bandnamen: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` usw.
   * Operatoren: `+`, `-`, `*`, `/`, `^` (Exponent)
   * Funktionen: `sqrt()`, `abs()` usw. (sofern unterstützt)
   * Klammern: `()` für die Reihenfolge der Operationen
5. **Benennen Sie Ihren Index** (z. B. „MyIndex” oder „CustomNDVI”)
6. **Speichern Sie die Konfiguration**

**Beispiele für benutzerdefinierte Formeln:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% Hinweis style=&quot;warning&quot; %}
**Formelvalidierung**: Stellen Sie sicher, dass Ihre Formel die in Ihrer Kamera verfügbaren Bänder verwendet. Beispielsweise ist RedEdge nur auf Kameras mit einem RedEdge-Filter verfügbar.
{% endhint %}

***

## Nächste Schritte

Nachdem Sie nun die Index-/LUT-Sandbox verstanden haben:

* **Auf die Verarbeitung anwenden**: Verwenden Sie die ermittelten Einstellungen in [Projekteinstellungen](../project-settings/project-settings.md)
* **Stapelverarbeitung**: Wenden Sie optimierte Indizes auf vollständige Datensätze an
* **Weitere Informationen**: Lesen Sie [Multispektrale Indexformeln](../project-settings/multispectral-index-formulas.md)

Verwandte Dokumentation:

* [**Bildlayer**](image-layers.md) – Layer-Verwaltung und Visualisierung
* [**Bild im Vollbildmodus öffnen**](opening-an-image-full-screen.md) – Grundlagen des Bildbetrachters
* [**Bilder verarbeiten (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Vollständiger Verarbeitungsworkflow
