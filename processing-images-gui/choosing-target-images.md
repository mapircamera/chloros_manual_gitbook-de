# Auswahl der Zielbilder

Indem Sie markieren, welche Bilder Kalibrierungsziele enthalten, teilen Sie Chloros genau mit, wo danach gesucht werden soll. Wenn in der Spalte „Ziel“ mindestens ein Bild markiert ist, scannt Chloros **nur die markierten Bilder** – das Markieren von Zielen dient also sowohl der Beschleunigung der Verarbeitung als auch dazu, zu verhindern, dass Vermessungsbilder fälschlicherweise als Ziel erkannt werden.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Warum Zielbilder markieren?

### Die Markierung steuert den Scanvorgang

Wenn Sie in der Spalte „Ziel“ bestimmte Bilder markieren:

* Chloros scannt nur die markierten Bilder nach Zielen
* Die Zielerkennung ist deutlich schneller abgeschlossen
* Vermessungsbilder können keine falschen Zielerkennungen verursachen

Wenn **keine** Bilder markiert sind, greift „Chloros“ auf den Standardmodus zurück und scannt jedes Bild im Projekt:

* Die Algorithmen zur Zielerkennung werden auf jedes Bild angewendet
* Hunderte oder Tausende von Bildern werden unnötigerweise überprüft
* Die Verarbeitung dauert deutlich länger, insbesondere bei großen Datensätzen

{% hint style="success" %}
**Geschwindigkeitssteigerung**: Das Markieren von 2–3 Zielbildern in einem Datensatz mit 500 Bildern kann die Zeit für die Zielerkennung von über 30 Minuten auf unter 1 Minute reduzieren.
{% endhint %}

***

## So markieren Sie Zielbilder

### Schritt 1: Identifizieren Sie Ihre Zielbilder

Sehen Sie sich Ihre importierten Bilder im Dateibrowser an und stellen Sie fest, welche Bilder Kalibrierungsziele enthalten.

**Häufige Szenarien:*** **Ziel vor der Aufnahme**: Vor Beginn der Sitzung aufgenommen
* **Ziel nach der Aufnahme**: Nach Abschluss der Sitzung aufgenommen
* **Ziele im Aufnahmebereich**: Ziele, die innerhalb des Aufnahmebereichs platziert wurden
* **Mehrere Ziele**: 2–3 Zielbilder pro Sitzung (empfohlen)

### Schritt 2: Überprüfen Sie die Spalte „Ziel“ (<img src="../.gitbook/assets/image (33).png" alt="" data-size="original">)

Für jedes Bild, das ein Kalibrierungsziel enthält:

1. Suchen Sie das Bild in der Tabelle des Dateibrowsers.
2. Suchen Sie die Spalte **Ziel** (äußerste rechte Spalte).
3. Aktivieren Sie das Kontrollkästchen in der Spalte „Ziel“ für dieses Bild.
4. Wiederholen Sie diesen Vorgang für alle Bilder, die Ziele enthalten.

### Schritt 3: Überprüfen Sie Ihre Auswahl

Überprüfen Sie vor der Verarbeitung noch einmal:

* [ ] Alle Bilder mit Kalibrierungszielen sind markiert
* [ ] Es sind keine Bilder ohne Kalibrierungsziel versehentlich markiert
* [ ] Die Kalibrierungsziele sind in den markierten Bildern deutlich sichtbar

***

## LATTICE: Targets sind optional, wenn eine DAQ aufzeichnet

Bei multispektralen LATTICE-Kameras ist ein im Bildfeld befindliches Kalibrierungstarget **eine von zwei** möglichen Reflexionsreferenzquellen:

* **Zielobjekt im Bildausschnitt**: Wenn ein markiertes Zielbild die Qualitätsprüfungs-Gates (QA) vChloros passiert, wird das Zielobjekt zur**absoluten Reflexionsreferenz** für die umgebenden Bildbereiche.
* **DAQ-Downwelling**: Wenn kein Ziel vorhanden ist (oder die Qualitätsprüfung fehlschlägt), berechnet „Chloros“ die Reflexion stattdessen anhand der nach unten gerichteten Bestrahlungsstärke des DAQ-Lichtsensors (ρ = π·L/E). Wenn eine `.daq`- oder DAQ-M-`.csv`-Aufzeichnung Ihre Aufnahmen abdeckt, erhalten Sie kalibrierte Reflektanzwerte**ganz ohne Zielbilder**.

Dieses automatische Verhalten ist die Standardeinstellung. In der „CLI“ / „SDK“ entspricht dies `--reflectance-source auto`; Sie können auch `target` (strikt – keine DAQ-Ersetzung) oder `daq` (DAQ-maßgeblich) erzwingen. Siehe die [CLI-Referenz](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**LATTICE-Zielgeometrien**: Neben der klassischen Panel-Erkennung, die für „Survey3“ verwendet wird, unterstützt die LATTICE-Verarbeitung**ArUco-markierte Ziele**,**Ziele mit festem ROI**und**Streifenziele**, die projektbezogen konfiguriert werden. Pro Einheit können**gemessene** Reflexionsgrad-Scans der Zielobjekte anhand der Seriennummer bereitgestellt werden (CLI: `--target-reflectance-dir`, jeweils ein `<serial>.csv` pro Zieleinheit), wobei die nominalen T3/T4P-Spektren als Ausweichwert dienen.

{% hint style="info" %}
**F988-Modul**: Die Reflektanz des F988 wird mithilfe einer Reflektanzplatte vor Ort kalibriert: Da das Band außerhalb des kalibrierten Bereichs des DAQ-Lichtsensors liegt, wendet „Chloros“ Ihre zuletzt erfasste Plattenaufnahme an und behält diese zwischen den Plattenmessungen bei. Wird ein F988-Modul ausschließlich über DAQ verarbeitet, lehnt „Chloros“ die DAQ-basierte Reflexion für dieses Band ab (Überspringgrund `dls-uncalibrated-band-988`) – der Panel-Workflow ist der unterstützte Weg.
{% endhint %}

***

## Bewährte Verfahren für Zielbilder

### Richtlinien zur Aufnahme von Zielbildern

**Zeitpunkt:**

* Nehmen Sie Zielbilder unmittelbar vor und während Ihrer Aufnahmesitzung auf
* Unter denselben Lichtbedingungen wie Ihr DAQ-Lichtsensor
* Nehmen Sie Zielbilder idealerweise so oft wie möglich auf, um optimale Ergebnisse zu erzielen. Andernfalls werden die Lichtsensordaten verwendet, um die Kalibrierung im Laufe der Zeit anzupassen.

**Kameraposition:**

* Halten Sie die Kamera so über das Ziel, dass dieses zentriert ist und etwa 40–60 % der Bildmitte ausfüllt.
* Halten Sie die Kamera parallel zur Zieloberfläche bzw. senkrecht darüber (Nadir).

**Beleuchtung:**

* Gleiche Umgebungsbeleuchtung wie bei Ihrem DAQ-Lichtsensor
* Vermeiden Sie Schatten auf den Zieloberflächen
* Versperren Sie die Lichtquelle nicht mit Ihrem Körper, Ihrem Fahrzeug oder Vegetation
* Bewölkte Bedingungen liefern die konsistentesten Ergebnisse

**Zustand des Ziels:**

* Halten Sie die Zielplatten sauber und trocken.
* Alle Platten Ihres Ziels (z. B. alle 4 bei einem T4) sollten deutlich sichtbar und frei von Hindernissen sein.
* Richten Sie die Ziele nach Möglichkeit senkrecht bzw. nadir zur Lichtquelle aus.

### Wie viele Zielbilder?

**Minimum:**1 Zielbild pro Sitzung.**Empfohlen:** 3–5 Zielbilder pro Sitzung.**Empfohlener Ablauf:**

* 3–5 Bilder, die kurz nach Beginn der Aufzeichnung durch den Lichtsensor aufgenommen werden
* Drehen Sie die Kamera zwischen den Aufnahmen, um optimale Ergebnisse zu erzielen
* Optional: in regelmäßigen Abständen während der Sitzung, wenn sich die Lichtverhältnisse ständig ändern

***

## Arbeiten mit mehreren Kameras

### Konfigurationen mit zwei Kameras

Bei gleichzeitiger Verwendung von zwei „MAPIR“-Kameras (z. B. Survey3W RGN + Survey3N OCN):

1. Nehmen Sie Zielbilder mit **beiden Kameras** gleichzeitig auf
2. Verwenden Sie für beide Kameras das **gleiche physische Zielobjekt**

3. Markieren Sie die Zielbilder für**beide Kameratypen** im Dateibrowser
4. „Chloros“ verwendet die entsprechenden Zielobjekte für die Kalibrierung der jeweiligen Kamera

### Spalte „Kameramodell“

Die Spalte **„Kameramodell“** hilft dabei, zu erkennen, welche Bilder von welcher Kamera stammen:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* usw.

Verwenden Sie diese Spalte, um zu überprüfen, ob Sie für jeden Kameratyp in Ihrem Projekt Ziele markiert haben.

***

## Einstellungen zur Zielerkennung

### Anpassen der Erkennungsempfindlichkeit

Wenn „Chloros“ Ihre Ziele nicht korrekt erkennt, passen Sie diese Einstellungen in den [Projekteinstellungen](adjusting-project-settings.md) an:**Mindestfläche für Kalibrierungsbeispiele (px):*** **Standard**: 25 Pixel
* **Erhöhen**, wenn bei kleinen Artefakten Fehlerkennungen auftreten
* **Verringern**, wenn Ziele nicht erkannt werden**Minimale Zielbündelung (0–100):*** **Standard**: 60
* **Erhöhen** Sie diesen Wert, wenn Ziele in mehrere Erkennungen aufgeteilt werden
* **Verringern** Sie diesen Wert, wenn Ziele mit Farbabweichungen nicht vollständig erkannt werden

{% hint style="info" %}
**Tipp zu CLI**: `chloros-cli process` akzeptiert dieselben Regler (`--min-target-size`, `--target-clustering`), und sein Flag `--target`/`--targets` kennzeichnet einen gesamten Eingabeordner als „nur Ziel-Panel“. Siehe die [CLI-Referenz](../reference/cli-reference.md).
{% endhint %}

***

## Häufige Probleme mit Zielbildern

### Problem: Keine Ziele erkannt

**Mögliche Ursachen:**

* Zielbilder sind im Dateibrowser nicht markiert
* Ziel im Bildausschnitt zu klein (< 30 % des Bildes)
* Schlechte Lichtverhältnisse (Schatten, Blendeffekte)
* Zu strenge Einstellungen für die Zielerkennung

**Lösungen:**

1. Überprüfen Sie, ob die Spalte „Ziel“ für die richtigen Bilder angekreuzt ist
2. Überprüfen Sie die Qualität der Zielbilder in der Vorschau
3. Nehmen Sie die Ziele erneut auf, wenn die Qualität schlecht ist
4. Passen Sie die Einstellungen für die Zielerkennung bei Bedarf an

### Problem: Falsche Zielerkennungen

**Mögliche Ursachen:**

* Weiße Gebäude, Fahrzeuge oder Bodenbedeckung werden fälschlicherweise als Ziele erkannt
* Helle Stellen in der Vegetation
* Zu geringe Erkennungsempfindlichkeit

**Lösungen:**

1. Nur tatsächliche Zielbilder markieren – nur markierte Bilder werden gescannt
2. Die minimale Kalibrierungsprobenfläche vergrößern
3. Den minimalen Zielclusterwert erhöhen
4. Sicherstellen, dass Zielbilder nur das Ziel zeigen (minimale Hintergrundstörungen)

***

## Checkliste zur Überprüfung

Überprüfen Sie vor Beginn der Verarbeitung Ihre Zielbildauswahl:

* [ ] Mindestens 1 markiertes Zielbild pro Sitzung (oder bei LATTICE eine `.daq`/`.csv`-Aufzeichnung, die die Sitzung abdeckt)
* [ ] Die Kontrollkästchen in der Zielspalte sind für alle Zielbilder aktiviert
* [ ] Die Zielbilder wurden im gleichen Zeitrahmen wie die Untersuchung aufgenommen
* [ ] Die Ziele sind in der Vorschau deutlich sichtbar, wenn man darauf klickt
* [ ] Alle Kalibrierungsfelder sind in jedem Zielbild sichtbar
* [ ] Keine Schatten oder Hindernisse auf den Zielen
* [ ] Bei Doppelkamera: Ziele für beide Kameratypen markiert

***

## Zielfreie Verarbeitung

### LATTICE: Mit einer DAQ-Aufzeichnung

Wenn ein DAQ-Lichtsensor während Ihrer LATTICE-Aufnahmen die nach unten gerichtete Bestrahlungsstärke aufgezeichnet hat, ist kein Ziel erforderlich:

1. Importieren Sie die Datei „`.daq`“ (oder „DAQ-M `.csv`“) mit den Bilddaten
2. Lassen Sie das Kontrollkästchen in der Spalte „Ziel“ deaktiviert
3. Die Reflexion wird automatisch anhand der von der DAQ gemessenen nach unten gerichteten Referenzstrahlung berechnet
4. Für die Strahlungsintensität ist weder ein Ziel noch ein DAQ erforderlich – sie ergibt sich allein aus der werkseitigen radiometrischen Kalibrierung der Kamera

### Verarbeitung ohne jegliche Referenz

Sie können die Verarbeitung auch ohne Ziele und ohne DAQ durchführen:

1. Lassen Sie alle Kontrollkästchen in der Spalte „Ziel“ deaktiviert
2. **Deaktivieren** Sie „Reflexionskalibrierung / Weißabgleich“ in den Projekteinstellungen – die Zielerkennung wird dann vollständig übersprungen
3. Die Vignettierungskorrektur wird weiterhin angewendet
4. Die Ausgabe wird nicht auf absolute Reflexion kalibriert (LATTICE Multispektral exportiert weiterhin debayered, Vorschau- und Radiance-Produkte)

{% hint style="warning" %}
**Nicht empfohlen für wissenschaftliche Arbeiten mit „Survey3“**: Ohne Reflektanzkalibrierung stellen die Pixelwerte von „Survey3“ lediglich relative Helligkeitswerte dar, keine wissenschaftlichen Reflektanzmessungen. Verwenden Sie Kalibrierungsziele (oder bei LATTICE einen DAQ-Lichtsensor), um genaue, wiederholbare Ergebnisse zu erzielen.
{% endhint %}

***

## Nächste Schritte

Sobald Sie Ihre Zielbilder markiert haben:

1. **Überprüfen Sie Ihre Einstellungen** – Siehe [Anpassen der Projekteinstellungen](adjusting-project-settings.md)
2. **Starten Sie die Verarbeitung** – siehe [Starten der Verarbeitung](starting-the-processing.md)
3. **Überwachen Sie den Fortschritt** – siehe [Überwachen der Verarbeitung](monitoring-the-processing.md)

Weitere Informationen zu den Kalibrierungszielen selbst finden Sie unter [Kalibrierungsziele](../calibration-targets.md).
