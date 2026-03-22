# Index-/LUT-Sandbox

Die Index-/LUT-Sandbox ist ein interaktiver Arbeitsbereich innerhalb des Chloros Image Viewers, in dem Sie in Echtzeit mit multispektralen Indexberechnungen und Farbvisualisierungen experimentieren können. Dieses leistungsstarke Tool hilft Ihnen dabei, verschiedene Indizes zu testen, Wertebereiche zu verfeinern und publikationsreife Visualisierungen zu erstellen, ohne Ihren gesamten Datensatz neu verarbeiten zu müssen.

## Was ist die Index/LUT-Sandbox?

### Zweck

Die Sandbox bietet:

* **Indexberechnung in Echtzeit** – Wenden Sie jeden Vegetationsindex sofort an
* **Interaktive LUT-Anpassung** – Feinabstimmung von Farbverläufen und -bereichen
* **Workflow-Optimierung** – Ermitteln Sie die besten Einstellungen vor der Stapelverarbeitung

### Sandbox vs. Projektverarbeitung

**Index/LUT-Sandbox (interaktiv):**

* Jeweils ein Bild
* Sofortiges Feedback
* Experimentell und iterativ
* Keine dauerhaften Änderungen an Dateien
* Ideal zum Erkunden und Testen

**Projektverarbeitung (Stapelverarbeitung):**

* Gesamter Datensatz auf einmal
* Vorkonfigurierte Einstellungen
* Permanente Ausgabedateien
* Zeitaufwändig
* Am besten geeignet, wenn die Einstellungen endgültig festgelegt sind

{% hint style="success" %}
**Bester Workflow**: Verwenden Sie die Sandbox zum Experimentieren und Finden der optimalen Index- und LUT-Einstellungen und wenden Sie diese Einstellungen dann während der Projektverarbeitung auf Ihren gesamten Datensatz an.
{% endhint %}

***

## Arbeiten mit der Index-/LUT-Sandbox

### Vorberechnete Indizes verstehen

In Chloros können Indizes während der Projektverarbeitung angewendet werden. Um zu bestimmen, welche Index- und LUT-Einstellungen Sie auf Exporte anwenden möchten, nutzen Sie am besten die Bildbetrachter-Sandbox.

Die Sandbox ermöglicht Ihnen:

* **neue Indizes und Farbverläufe (LUTs) anzuwenden**, um die Daten zu visualisieren
* **Visualisierungseinstellungen** interaktiv anzupassen
* **bereits berechnete Indexbilder anzuzeigen*** **Pixelwerte auf allen Zoomstufen zu überprüfen**

### Öffnen der Sandbox

Der Zugriff auf die Index-/LUT-Sandbox erfolgt über die Registerkarte <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Klicken Sie auf ein Bild im Bildraster des Dateibrowsers; es wird im **Bildbetrachter**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> 2. Klicken Sie auf die Registerkarte**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , um die linke Pop-out-Seitenleiste zu öffnen, falls diese noch nicht geöffnet ist

### Ein Bild auswählen, auf das ein Index/eine LUT angewendet werden soll

So arbeiten Sie mit einem Index in der <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> Sandbox zu arbeiten:

1. **Öffnen Sie ein Bild** aus dem Hauptbildraster, indem Sie darauf klicken
2. Die Registerkarte **Bildbetrachter** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
3. Klicken Sie auf das **Ebenen-Dropdown-Menü** (oben rechts im Viewer).
4. Wählen Sie die Ebene aus dem Dropdown-Menü aus:
   * RAW (Reflexion)

### Anwenden eines Index auf ein Bild

Sobald das Bild im Vollbildmodus angezeigt wird und die Seitenleiste des **Bildbetrachters** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -Seitenleiste geöffnet ist:

1. Aktivieren Sie das Kontrollkästchen „Index“ oben in der Seitenleiste
2. Wählen Sie den Filter Ihrer Kamera aus dem linken Dropdown-Menü
3. Wählen Sie die gewünschte Indexformel aus dem rechten Dropdown-Menü
4. Ziehen Sie die Farbkreise des Filterkanals an die entsprechenden Stellen in der Indexformel darunter
5. Sobald die Formel gültig ist, wird das Bild aktualisiert und zeigt die Indexwerte an
6. Bewegen Sie den Mauszeiger, um die Werte an der Position des Zeigers anzuzeigen
7. Zoomen Sie heran, um einzelne Pixel und die zugehörigen Werte zu sehen

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
* **Ausgabe**: Farbe (z. B. hellgrün)
* **Zweck**: Muster leichter erkennbar und interpretierbar machen**Graustufen- vs. Farb-LUT:**

* Graustufen: Wissenschaftlich und neutral, zeigt Rohdaten
* Farb-LUT: Intuitiv und wirkungsvoll, hebt Muster und Unterschiede hervor

{% hint style="success" %}
**Visualisierungsleistung**: Durch Anwenden einer Farb-LUT auf ein Graustufen-Indexbild lassen sich Muster, Anomalien und Bereiche von Interesse auf einen Blick deutlich leichter erkennen.
{% endhint %}

### Anwenden einer LUT auf ein Indexbild

Sobald Sie ein Indexbild haben, das Folgendes anzeigt

1. Klicken Sie auf die <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> Schaltfläche „+LUT hinzufügen“
2. Wählen Sie den Farbverlauf aus
3. Passen Sie die minimalen und maximalen Endpunkte der Begrenzung an
4. Passen Sie den Begrenzungsmodus an
5. Aktivieren Sie das Kontrollkästchen „Index“ in der Seitenleiste des **Bildbetrachters** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , um die LUT anzuwenden

### Einen Farbverlauf auswählen

**Auswahl eines Farbverlaufs:**

1. Suchen Sie im LUT-Bedienfeld den**farbigen Verlaufsbalken**

2. Bewegen Sie den Mauszeiger darüber, um die verfügbaren Farbverlaufsvoreinstellungen anzuzeigen
3. Wählen Sie den gewünschten Farbverlauf aus
4. Das Bild **wird sofort** mit den neuen Farben aktualisiert, sobald das Kontrollkästchen „Index“ aktiviert ist

{% hint style="success" %}
**Bewährte Vorgehensweise**: Für Vegetationsindizes wie NDVI ist der Farbverlauf Red-Gelb-Green am intuitivsten, da er den natürlichen Farbassoziationen entspricht (grün = gesund, gelb = mäßig, rot = gestresst).
{% endhint %}

### Farbklassen anpassen

Der **Regler „Klassen“**bestimmt, wie viele diskrete Farbstufen in Ihrem Farbverlauf erscheinen:**Optionen für die Anzahl der Klassen:*** **2–5 Klassen**: Sehr breite Kategorien, deutlich voneinander abgegrenzte Zonen
* **6–10 Klassen**: Ausgewogen, gut für die Klassifizierung
* **11–20 Klassen**: Sanfte Farbverläufe, kontinuierliches Erscheinungsbild
* **20+ Klassen**: Nahezu kontinuierlich, maximale Sanftheit**So passen Sie die Einstellung an:**

1. Suchen Sie im LUT-Bedienfeld die**Farbfeldquadrate unterhalb der Farbverlaufsleiste**

2. Passen Sie die Anzahl der Klassen an, indem Sie mit der Schaltfläche „+“ weitere hinzufügen
3. Entfernen Sie Klassen, indem Sie auf ein Farbfeld doppelklicken
4. Der Farbverlauf wird **in Echtzeit** im Bild aktualisiert**Auswirkung auf die Visualisierung:*** **Weniger Klassen** (3–5): Erzeugt klar abgegrenzte Zonen, vereinfachte Klassifizierung, leichter zu unterscheidende Kategorien
* **Mittlere Anzahl an Klassen** (6–10): Ausgewogener Ansatz, gut für die meisten Anwendungen
* **Mehr Klassen** (15–20): Sanfte Übergänge, detaillierte Variationen, fotorealistisches Erscheinungsbild**Anwendungsbereiche:*** **Wenige Klassen (3–5)**: Präsentationsfolien, Klassifizierungskarten, einfache Berichte
* **Mittlere Klassen (6–10)**: Allgemeine Analysen, ausgewogene Detailgenauigkeit, Standardberichte
* **Viele Klassen (15–20)**: Wissenschaftliche Analysen, detaillierte Untersuchungen, Ausgabedateien in Publikationsqualität

### Feinabstimmung der Wertebereiche

Die **Wertebereichsregler**legen fest, welche Indexwerte in Ihrem Farbverlauf welchen Farben zugeordnet werden:**Regler im LUT-Bedienfeld:*** **Mindestwert**: Untergrenze der Farbskala
* **Höchstwert**: Obergrenze der Farbskala
* **Zwischenwerte**: Werden automatisch zwischen Min und Max verteilt (basierend auf der Anzahl der Klassen)

#### Min-/Max-Werte anpassen

**So passen Sie die Wertebereiche an:**

1. Suchen Sie im LUT-Fenster die Eingabefelder**Min-Wert**und**Max-Wert**

2. Klicken Sie auf das Feld**Min-Wert**

3. Geben Sie den gewünschten Mindestwert ein (z. B. `0.2`)
4. Drücken Sie die **Eingabetaste** oder klicken Sie außerhalb des Feldes
5. Wiederholen Sie den Vorgang für das Feld **Maximalwert** (z. B. `0.9`)
6. Die Visualisierung **wird sofort aktualisiert**{% hint style="info" %}**Automatische Skalierung**: Wenn Sie eine LUT zum ersten Mal anwenden, setzt Chloros automatisch den Min-/Max-Wert auf den tatsächlichen Datenbereich im Bild. Sie können diesen Bereich dann eingrenzen, um sich auf bestimmte Wertebereiche von Interesse zu konzentrieren.
{% endhint %}

**Beispiel für Bereichsanpassungen in NDVI:*** **Vollständiger Bereich**: `-1.0` bis `1.0` (alle möglichen Werte anzeigen)
* **Auf Vegetation fokussiert**: `0.2` bis `0.9` (ohne nackten Boden und Wasser)
* **Nur gesunde Vegetation**: `0.5` bis `0.9` (nur kräftige Pflanzen hervorheben)
* **Stresserkennung**: `0.2` bis `0.5` (Problembereiche hervorheben)
* **Benutzerdefinierter Bereich**: Passen Sie den Bereich anhand Ihrer beobachteten Pixelwerte an**Warum Bereiche anpassen?*** **Kontrast erhöhen** in Ihrem Untersuchungsgebiet
* **Irrelevante Werte ausschließen** (z. B. Gewässer, nackter Boden)
* **Visualisierung vereinheitlichen** über mehrere Bilder oder Zeitpunkte hinweg
* **Feine Unterschiede hervorheben** innerhalb eines engen Wertebereichs

### Werte außerhalb des Bereichs abschneiden

Wenn Pixelwerte außerhalb des von Ihnen definierten Min/Max-Bereichs liegen, können Sie mithilfe von **Clipping-Modi** steuern, wie diese angezeigt werden.

#### **Verfügbare Optionen für Clipping-Modi:**

#### 1. Minimum und Maximum

* Pixel **unterhalb des Minimums**→ Anzeige mit der**ersten Farbe** im Farbverlauf (z. B. Rot)
* Pixel **über dem Maximum**→ Darstellung mit der**letzten Farbe** im Farbverlauf (z. B. Grün)
* **Anwendungsfall**: Hervorhebung von Extremwerten, Darstellung des gesamten Datenbereichs mit gesättigten Farben an den Grenzen
* **Beispiel**: NDVI-Werte unter 0,2 erscheinen alle rot, Werte über 0,9 erscheinen alle grün

#### 2. Transparenter Hintergrund

* Pixel **außerhalb des Bereichs**werden**vollständig transparent*** Nur Pixel **innerhalb des Bereichs** zeigen den Farbverlauf
* **Anwendungsfall**: GIS-Überlagerung, Isolieren bestimmter Wertebereiche, Hervorheben nur der relevanten Bereiche
* **Beispiel**: Nur NDVI-Werte von 0,4–0,7 in Farbe anzeigen, alles andere transparent

{% hint style="warning" %}
**Einschränkung der Transparenz**: Transparente Pixel werden im Viewer in der Hintergrundfarbe angezeigt. Beim Export während der Verarbeitung bleibt die Transparenz im PNG-Format erhalten, nicht jedoch im JPG-Format.
{% endhint %}

#### 3. Index-Hintergrund

* Pixel **außerhalb des Bereichs**werden in**Graustufen** angezeigt (mit den rohen Indexwerten)
* Pixel **innerhalb des Bereichs**zeigen einen**Farbverlauf*** **Anwendungsfall**: Dezente Hervorhebung, Beibehaltung des Kontexts bei gleichzeitiger Betonung von Bereichen von Interesse
* **Beispiel**: Farblich hervorheben von gestresster Vegetation (NDVI 0,3–0,5), während gesunde Bereiche in Grau dargestellt werden

#### 4. Originalhintergrund

* Pixel **außerhalb des Bereichs**zeigen das**ursprüngliche Multispektralbild*** Pixel **innerhalb des Bereichs**zeigen einen**Farbverlauf*** **Anwendungsfall**: Am intuitivsten – kombiniert den natürlichen Bildkontext mit einer analytischen Farbüberlagerung
* **Beispiel**: Betrachten Sie das tatsächliche Erscheinungsbild des Feldes/der Kulturpflanze mit farbcodierten, überlagerten Stressbereichen

### Auswahl des richtigen Clipping-Modus

| Clipping-Modus              | Am besten geeignet für                                   | Visualisierungsstil          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum und Maximum**    | Vollständige Datenanzeige, wissenschaftliche Analyse     | Alle Pixel farbig           |
| **Transparenter Hintergrund** | GIS-Überlagerungen, Isolierung bestimmter Bereiche    | Farbe im Bereich, leer außerhalb |
| **Index-Hintergrund**       | Dezente Hervorhebung, Beibehaltung des Datenkontexts  | Farbe im Bereich, grau außerhalb  |
| **Original-Hintergrund**    | Berichte, Präsentationen, intuitive Analyse | Farbe im Bereich, Foto außerhalb |

### Benutzerdefinierte LUT-Farben erstellen

Um die volle Kontrolle über Ihre Visualisierung zu haben, können Sie **benutzerdefinierte Farbverläufe** erstellen, indem Sie einzelne Farbstufen bearbeiten.**So erstellen Sie einen benutzerdefinierten Farbverlauf:**

1. Suchen Sie im LUT-Bedienfeld die**Vorschauleiste für den Farbverlauf**

2. Suchen Sie unterhalb des Farbverlaufs nach den**Farbfeldern**

3.**Klicken Sie auf eine Farbstufe**, um sie auszuwählen
4. Ein **Farbwähler** öffnet sich
5. Wählen Sie eine neue Farbe aus mit:
   * **Farbrad**: Visuelle Farbauswahl
   * **RGB/HSV-Schieberegler**: Präzise Farbsteuerung
   * **Hex-Code-Eingabe**: Genaue Farbspezifikation (z. B. `#FF0000` für Rot)
6. Klicken Sie außerhalb der Farbauswahl, **um die neue Farbe anzuwenden**

7. Der Farbverlauf**wird sofort** auf dem Bild aktualisiert**Farbstopps hinzufügen oder entfernen:*** **Farbstopp hinzufügen**: Klicken Sie auf das +-Symbol, um am Ende ein neues Farbfeld hinzuzufügen
* **Farbstopp entfernen**: Doppelklicken Sie auf das Farbfeld, um es zu entfernen**Anpassungsstrategien:*** **Farbverlauf umkehren**: Kehren Sie die Farbreihenfolge um, um die Bedeutung umzukehren (z. B. grün = niedrig, rot = hoch)
* **Markenfarben**: Passen Sie die Farbpalette Ihrer Organisation für Berichte an
* **Farbenblind-freundlich**: Verwenden Sie Orange-Blau- oder Violett-Gelb-Kombinationen
* **Druckoptimierung**: Wählen Sie Farben, die sowohl im Farb- als auch im Graustufendruck gut zur Geltung kommen
* **Mehrere Schwellenwerte**: Verwenden Sie unterschiedliche Farben bei bestimmten Werteschwellen für die Klassifizierung

{% hint style="info" %}
**Benutzerdefinierte Farbverläufe speichern**: Benutzerdefinierte Farbverläufe können gespeichert und wiederverwendet werden. Klicken Sie auf das Speichersymbol im LUT-Bedienfeld, um Ihre benutzerdefinierten Farbschemata für die spätere Verwendung zu speichern.
{% endhint %}

***

## Interaktiver Arbeitsablauf

### Echtzeit-Aktualisierungen

Alle LUT-Anpassungen in der Sandbox aktualisieren das Bild **sofort und interaktiv**:

* **Ebene wechseln** → Das Bild ändert sich sofort
* **Farbverlauf auswählen** → Farben werden sofort aktualisiert
* **Wertebereich anpassen** → Kontrast ändert sich in Echtzeit
* **Klassen ändern** → Glättung des Farbverlaufs wird sofort aktualisiert
* **Clipping ändern** → Hintergrundanzeige ändert sich sofort
* **Farben bearbeiten** → Benutzerdefinierter Farbverlauf wird sofort angewendet**Keine „Anwenden“-Schaltfläche erforderlich** – alle Änderungen erfolgen live und interaktiv!

{% hint style="success" %}
**Live-Feedback**: Dank des sofortigen visuellen Feedbacks können Sie schnell mit verschiedenen Einstellungen experimentieren, bis Sie die optimale Visualisierung für Ihre Analyseanforderungen gefunden haben.
{% endhint %}

### Iterativer Verfeinerungs-Workflow

**Typischer LUT-Optimierungs-Workflow:**

1.**Indexebene auswählen** (z. B. RAW (Reflexion))
2. **Index anwenden** – Wählen Sie einen Kamerafilter und eine Indexformel aus, ziehen Sie die farbigen Kreise an die entsprechende Stelle in der Indexformel
3. **LUT-Verlauf anwenden** – Beginnen Sie mit der Voreinstellung Red-Yellow-Green
4. **Pixelwerte prüfen** – Bewegen Sie den Cursor und notieren Sie sich die Wertebereiche
5. **Min/Max anpassen** – Bereich eingrenzen, um den Fokus auf Vegetation zu legen (z. B. 0,2 bis 0,9)
6. **Clipping auswählen** – Probieren Sie „Original Background“ für den Kontext aus
7. **Farben verfeinern** – Passen Sie den Farbverlauf bei Bedarf an, um bestimmte Bereiche hervorzuheben
8. **Einstellungen abschließen**– Dokumentieren Sie die Einstellungen und kopieren Sie sie in die Projekteinstellungen für die Exportverarbeitung

### Überprüfung der Pixelwerte

Das Verständnis der tatsächlichen Pixelwerte ist entscheidend für die Festlegung effektiver LUT-Bereiche:**So überprüfen Sie die Werte:**

1. Die Pixelwerte werden angezeigt, wenn im Bild entweder das Kästchen „Index“ oder sowohl das Kästchen „Index“ als auch das Kästchen „LUT“**aktiviert** ist.
2. **Bewegen Sie den Cursor** über verschiedene Bereiche des Bildes
3. **Beobachten Sie die Pixelwerte**, die in der Legende angezeigt werden, während Sie den Mauszeiger darüber bewegen
4. Zoomen Sie heran, um einzelne Pixel zu sehen, die mit einem schwebenden Wert hervorgehoben sind
5. **Notieren Sie sich** die Wertebereiche für verschiedene Merkmale:
   * **Gesunde Vegetation**: z. B. NDVI 0,55–0,85
   * **Gestresste Vegetation**: z. B. NDVI 0,30–0,50
   * **Boden**: z. B. NDVI 0,05–0,25
   * **Wasser** (falls vorhanden): z. B. NDVI -0,05 bis 0,10**Verwendung von Pixelwerten zur Festlegung der LUT-Bereiche:**Passen Sie nach der Überprüfung der Pixelwerte die LUT-Min-/Max-Werte entsprechend an:**Beispielszenario:*** **Beobachtung**: Bodenwerte = 0,05–0,25, gestresst = 0,25–0,50, gesund = 0,50–0,85
* **Ziel**: Nur den Gesundheitszustand der Pflanzen visualisieren (Boden ausschließen)
* **LUT-Einstellungen**: Min = `0.25`, Max = `0.85`
* **Clipping**: „Original Background“, um den Boden in seiner natürlichen Farbe zu sehen
* **Ergebnis**: Der Farbverlauf gilt nur für die Vegetation, der Boden wird wie im Originalbild dargestellt

{% hint style="info" %}
**Dynamikbereich**: Verschiedene Kulturen, Jahreszeiten und Wachstumsstadien weisen unterschiedliche Wertebereiche auf. Überprüfen Sie immer die Pixelwerte in Ihrem spezifischen Datensatz, bevor Sie LUT-Bereiche festlegen.
{% endhint %}

***

## Benutzerdefinierte Indizes (Chloros+)

### Erstellen benutzerdefinierter Indexformeln

{% hint style="info" %}
**Wo erstellen**: Benutzerdefinierte Indizes können vor der Verarbeitung in den**Projekteinstellungen** sowie in der Seitenleiste der Image Viewer-Sandbox konfiguriert werden.
{% endhint %}

**So erstellen Sie einen benutzerdefinierten Index:**

1.**Öffnen Sie die Projekteinstellungen** (vor der Verarbeitung) oder die Seitenleiste der Image Viewer-Sandbox
2. Navigieren Sie zum **Dropdown-Menü „Indexformel“**

3. Suchen Sie nach der Option**„Benutzerdefiniert“** (Sie müssen mit einer Chloros+-Lizenz angemeldet sein)
4. **Definieren Sie Ihre Formel** unter Verwendung von Bandvariablen:
   * Bandnamen: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` usw.
   * Operatoren: `+`, `-`, `*`, `/`, `^` (Exponent)
   * Funktionen: `sqrt()`, `abs()` usw. (sofern unterstützt)
   * Klammern: `()` für die Reihenfolge der Operationen
5. **Benennen Sie Ihren Index** (z. B. „MyIndex“ oder „CustomNDVI“)
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

{% hint style="warning" %}
**Formelvalidierung**: Stellen Sie sicher, dass Ihre Formel Bänder verwendet, die in Ihrer Kamera verfügbar sind. Beispielsweise ist RedEdge nur auf Kameras mit einem RedEdge-Filter verfügbar.
{% endhint %}

***

## Nächste Schritte

Nachdem Sie nun die Index-/LUT-Sandbox verstanden haben:

* **Auf die Verarbeitung anwenden**: Verwenden Sie die ermittelten Einstellungen in den [Projekteinstellungen](../project-settings/project-settings.md)
* **Stapelverarbeitung**: Wenden Sie optimierte Indizes auf vollständige Datensätze an
* **Weitere Informationen**: Lesen Sie [Multispektrale Indexformeln](../project-settings/multispectral-index-formulas.md)

Verwandte Dokumentation:

* [**Bildebenen**](image-layers.md) – Verwaltung und Visualisierung von Ebenen
* [**Ein Bild im Vollbildmodus öffnen**](opening-an-image-full-screen.md) – Grundlagen des Bildbetrachters
* [**Bildbearbeitung (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Vollständiger Bearbeitungsworkflow
