# Auswahl der Zielbilder

Das Markieren der Bilder, die Kalibrierungsziele enthalten, ist ein entscheidender Schritt, der die Verarbeitungsabläufe von Chloros erheblich beschleunigt. Durch die Vorauswahl der Zielbilder entfällt die Notwendigkeit, dass Chloros jedes einzelne Bild in Ihrem Datensatz nach Kalibrierungszielen durchsucht.

## Warum Zielbilder markieren?

### Verarbeitungsgeschwindigkeit

Ohne die Markierung von Zielbildern muss Chloros:

* jedes einzelne Bild in Ihrem Projekt scannen
* Zielerkennungsalgorithmen auf jedes Bild anwenden
* Hunderte oder Tausende von Bildern unnötigerweise überprüfen

**Ergebnis**: Die Verarbeitung kann erheblich länger dauern, insbesondere bei großen Datensätzen.

### Mit markierten Zielbildern

Wenn Sie in der Spalte „Ziel“ bestimmte Bilder markieren:

* scannt Chloros nur die markierten Bilder nach Zielen
* ist die Zielerkennung deutlich schneller abgeschlossen
* wird die Gesamtverarbeitungszeit erheblich reduziert

{% hint style="success" %}
**Geschwindigkeitssteigerung**: Das Markieren von 2–3 Zielbildern in einem Datensatz mit 500 Bildern kann die Zielerkennungszeit von über 30 Minuten auf unter 1 Minute reduzieren.
{% endhint %}

***

## So markieren Sie Zielbilder

### Schritt 1: Identifizieren Sie Ihre Zielbilder

Sehen Sie sich Ihre importierten Bilder im Dateibrowser an und identifizieren Sie, welche Bilder Kalibrierungsziele enthalten.

**Häufige Szenarien:*** **Ziel vor der Aufnahme**: Vor Beginn der Sitzung aufgenommen
* **Ziel nach der Aufnahme**: Nach Abschluss der Sitzung aufgenommen
* **Ziele im Aufnahmebereich**: Ziele, die innerhalb des Aufnahmebereichs platziert wurden
* **Mehrere Ziele**: 2–3 Zielbilder pro Sitzung (empfohlen)

### Schritt 2: Überprüfen Sie die Spalte „Ziel“

Für jedes Bild, das ein Kalibrierungsziel enthält:

1. Suchen Sie das Bild in der Tabelle des Dateibrowsers.
2. Suchen Sie die Spalte **Ziel** (Spalte ganz rechts).
3. Aktivieren Sie das Kontrollkästchen in der Spalte „Ziel“ für dieses Bild.
4. Wiederholen Sie diesen Vorgang für alle Bilder, die Ziele enthalten.

### Schritt 3: Überprüfen Sie Ihre Auswahl

Überprüfen Sie vor der Verarbeitung noch einmal:

* [ ] Alle Bilder mit Kalibrierungszielen sind markiert
* [ ] Es sind keine Bilder ohne Ziel versehentlich markiert
* [ ] Die Ziele sind in den markierten Bildern deutlich sichtbar

***

## Bewährte Verfahren für Zielbilder

### Richtlinien zur Aufnahme von Zielbildern

**Zeitpunkt:**

* Nehmen Sie Zielbilder unmittelbar vor und während Ihrer Aufnahmesitzung auf
* Unter denselben Lichtbedingungen wie Ihr DAQ-Lichtsensor
* Nehmen Sie Zielbilder idealerweise so oft wie möglich auf, um die besten Ergebnisse zu erzielen. Andernfalls werden die Lichtsensordaten verwendet, um die Kalibrierung im Laufe der Zeit anzupassen.

**Kameraposition:**

* Halten Sie die Kamera so über das Ziel, dass es zentriert ist und etwa 40–60 % der Bildmitte ausfüllt.
* Halten Sie die Kamera parallel/senkrecht zur Zieloberfläche

**Beleuchtung:**

* Gleiche Umgebungsbeleuchtung wie bei Ihrem DAQ-Lichtsensor
* Vermeiden Sie Schatten auf den Zieloberflächen
* Blockieren Sie die Lichtquelle nicht mit Ihrem Körper, Ihrem Fahrzeug oder Vegetation
* Bewölkte Bedingungen liefern die konsistentesten Ergebnisse

**Zieleigenschaften:**

* Halten Sie die Zieltafeln sauber und trocken
* Alle 4 Tafeln sollten klar sichtbar und frei von Hindernissen sein
* Ziele nach Möglichkeit senkrecht/nadir zur Lichtquelle

### Wie viele Zielbilder?

**Minimum:**1 Zielbild pro Sitzung.**Empfohlen:** 3–5 Zielbilder pro Sitzung.**Empfohlener Ablauf:**

* 3–5 Bilder, die kurz nach Beginn der Aufzeichnung durch den Lichtsensor aufgenommen werden
* Drehen Sie die Kamera zwischen den Aufnahmen, um optimale Ergebnisse zu erzielen
* Optional: regelmäßig während der Sitzung, wenn sich die Lichtverhältnisse ständig ändern

***

## Arbeiten mit mehreren Kameras

### Konfigurationen mit zwei Kameras

Bei gleichzeitiger Verwendung von zwei MAPIR-Kameras (z. B. Survey3W RGN + Survey3N OCN):

1. Nehmen Sie Zielbilder mit **beiden Kameras** gleichzeitig auf
2. Verwenden Sie für beide Kameras das **gleiche physische Ziel**

3. Markieren Sie die Zielbilder für**beide Kameratypen** im Dateibrowser
4. Chloros verwendet die entsprechenden Ziele für die Kalibrierung der jeweiligen Kamera

### Spalte „Kameramodell“

Die Spalte **„Kameramodell“** hilft dabei, zu erkennen, welche Bilder von welcher Kamera stammen:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* usw.

Verwenden Sie diese Spalte, um zu überprüfen, ob Sie für jeden Kameratyp in Ihrem Projekt Ziele markiert haben.

***

## Einstellungen zur Zielerkennung

### Anpassen der Erkennungsempfindlichkeit

Wenn Chloros Ihre Ziele nicht korrekt erkennt, passen Sie diese Einstellungen in den [Projekteinstellungen](adjusting-project-settings.md) an:**Mindestbereich für Kalibrierungsbeispiele:*** **Standard**: 25 Pixel
* **Erhöhen**, wenn bei kleinen Artefakten falsche Erkennungen auftreten
* **Verringern**, wenn Ziele nicht erkannt werden**Mindestzielclusterung:*** **Standard**: 60
* **Erhöhen**, wenn Ziele in mehrere Erkennungen aufgeteilt werden
* **Verringern**, wenn Ziele mit Farbabweichungen nicht vollständig erkannt werden***

## Häufige Probleme mit Zielbildern

### Problem: Keine Ziele erkannt

**Mögliche Ursachen:**

* Zielbilder im Dateibrowser nicht markiert
* Ziel im Bildausschnitt zu klein (&lt; 30 % des Bildes)
* Schlechte Beleuchtung (Schatten, Blendeffekte)
* Zu strenge Einstellungen für die Zielerkennung

**Lösungen:**

1. Überprüfen Sie, ob die Spalte „Ziel“ für die richtigen Bilder aktiviert ist
2. Überprüfen Sie die Qualität der Zielbilder in der Vorschau
3. Erfassen Sie die Ziele erneut, wenn die Qualität schlecht ist
4. Passen Sie die Einstellungen für die Zielerkennung bei Bedarf an

### Problem: Falsche Zielerkennungen

**Mögliche Ursachen:**

* Weiße Gebäude, Fahrzeuge oder Bodenbedeckung werden fälschlicherweise als Ziele erkannt
* Helle Flecken in der Vegetation
* Zu geringe Erkennungsempfindlichkeit

**Lösungen:**

1. Markieren Sie nur tatsächliche Zielbilder, um den Erfassungsbereich einzuschränken
2. Erhöhen Sie die minimale Kalibrierungsprobenfläche
3. Erhöhen Sie den minimalen Zielclusterwert
4. Stellen Sie sicher, dass Zielbilder nur das Ziel zeigen (minimale Hintergrundstörungen)

***

## Überprüfungscheckliste

Überprüfen Sie vor Beginn der Verarbeitung Ihre Zielbildauswahl:

* [ ] Mindestens 1 Zielbild pro Sitzung markiert
* [ ] Die Kontrollkästchen in der Zielspalte sind für alle Zielbilder aktiviert
* [ ] Zielbilder wurden im gleichen Zeitrahmen wie die Erfassung aufgenommen
* [ ] Ziele sind in der Vorschau deutlich sichtbar, wenn sie angeklickt werden
* [ ] Alle 4 Kalibrierungsfelder sind in jedem Zielbild sichtbar
* [ ] Keine Schatten oder Hindernisse auf den Zielen
* [ ] Bei Dual-Kamera: Ziele für beide Kameratypen markiert

***

## Zielfreie Verarbeitung

### Verarbeitung ohne Kalibrierungsziele

Obwohl dies für wissenschaftliche Arbeiten nicht empfohlen wird, können Sie die Verarbeitung ohne Ziele durchführen:

1. Lassen Sie alle Kontrollkästchen in der Spalte „Ziel“ deaktiviert
2. **Deaktivieren** Sie „Reflexionskalibrierung“ in den Projekteinstellungen
3. Die Vignettierungskorrektur wird weiterhin angewendet
4. Die Ausgabe wird nicht auf absolute Reflexion kalibriert

{% hint style="warning" %}
**Nicht empfohlen**: Ohne Reflektanzkalibrierung stellen die Pixelwerte nur die relative Helligkeit dar, keine wissenschaftlichen Reflektanzmessungen. Verwenden Sie Kalibrierungsziele für genaue, wiederholbare Ergebnisse.
{% endhint %}

***

## Nächste Schritte

Sobald Sie Ihre Zielbilder markiert haben:

1. **Überprüfen Sie Ihre Einstellungen** – Siehe [Anpassen der Projekteinstellungen](adjusting-project-settings.md)
2. **Starten Sie die Verarbeitung** – Siehe [Starten der Verarbeitung](starting-the-processing.md)
3. **Überwachen Sie den Fortschritt** – siehe [Überwachen der Verarbeitung](monitoring-the-processing.md)

Weitere Informationen zu den Kalibrierungszielen selbst finden Sie unter [Kalibrierungsziele](../calibration-targets.md).
