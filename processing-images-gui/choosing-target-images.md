# Auswahl der Zielbilder

Das Markieren der Bilder, die Kalibrierungsziele enthalten, ist ein wichtiger Schritt, der die Verarbeitungspipeline von Chloros erheblich beschleunigt. Durch die Vorauswahl der Zielbilder muss Chloros nicht jedes Bild in Ihrem Datensatz nach Kalibrierungszielen durchsuchen.

## Warum Zielbilder markieren?

### Verarbeitungsgeschwindigkeit

Ohne die Markierung von Zielbildern muss Chloros:

* jedes einzelne Bild in Ihrem Projekt scannen
* Zielerkennungsalgorithmen für jedes Bild ausführen
* Hunderte oder Tausende von Bildern unnötigerweise überprüfen

**Ergebnis**: Die Verarbeitung kann erheblich länger dauern, insbesondere bei großen Datensätzen.

### Mit markierten Zielbildern

Wenn Sie die Spalte „Ziel“ für bestimmte Bilder aktivieren:

* Chloros scannt nur die aktivierten Bilder nach Zielen.
* Die Zielerkennung ist wesentlich schneller abgeschlossen.
* Die Gesamtverarbeitungszeit wird erheblich reduziert.

{% hint style=&quot;success&quot; %}
**Geschwindigkeitsverbesserung**: Durch Markieren von 2–3 Zielbildern in einem Datensatz mit 500 Bildern kann die Zielerkennungszeit von über 30 Minuten auf unter 1 Minute reduziert werden.
{% endhint %}

***

## So markieren Sie Zielbilder

### Schritt 1: Identifizieren Sie Ihre Zielbilder

Sehen Sie sich Ihre importierten Bilder im Dateibrowser an und identifizieren Sie, welche Bilder Kalibrierungsziele enthalten.

**Häufige Szenarien:**

* **Ziel vor der Aufnahme**: Vor Beginn der Sitzung aufgenommen
* **Ziel nach der Aufnahme**: Nach Abschluss der Sitzung aufgenommen
* **Ziele im Feld**: Ziele, die innerhalb des Aufnahmebereichs platziert wurden
* **Mehrere Ziele**: 2–3 Zielbilder pro Sitzung (empfohlen)

### Schritt 2: Überprüfen Sie die Zielspalte

Für jedes Bild, das ein Kalibrierungsziel enthält:

1. Suchen Sie das Bild in der Dateibrowser-Tabelle.
2. Suchen Sie die Spalte **Ziel** (Spalte ganz rechts).
3. Klicken Sie auf das Kontrollkästchen in der Spalte „Ziel” für dieses Bild.
4. Wiederholen Sie diesen Vorgang für alle Bilder, die Ziele enthalten.

### Schritt 3: Überprüfen Sie Ihre Auswahl

Überprüfen Sie vor der Verarbeitung noch einmal Folgendes:

* [ ] Alle Bilder mit Kalibrierungszielen sind markiert.
* [ ] Es sind keine Bilder ohne Ziele versehentlich markiert.
* [ ] Die Ziele sind in den markierten Bildern deutlich sichtbar.

***

## Bewährte Verfahren für Zielbilder

### Richtlinien für die Zielerfassung

**Zeitpunkt:**

* Erfassen Sie Zielbilder unmittelbar vor und während Ihrer Erfassungssitzung.
* Unter denselben Lichtbedingungen wie Ihr DAQ-Lichtsensor.
* Nehmen Sie Zielbilder idealerweise so oft wie möglich auf, um die besten Ergebnisse zu erzielen. Andernfalls werden die Daten des Lichtsensors verwendet, um die Kalibrierung im Laufe der Zeit anzupassen.

**Kameraposition:**

* Halten Sie die Kamera so über das Ziel, dass es zentriert ist und etwa 40-60 % der Bildmitte ausfüllt.
* Halten Sie die Kamera parallel/nadir zur Zieloberfläche.

**Beleuchtung:**

* Gleiche Umgebungsbeleuchtung wie Ihr DAQ-Lichtsensor
* Vermeiden Sie Schatten auf den Zieloberflächen
* Blockieren Sie die Lichtquelle nicht mit Ihrem Körper, Ihrem Fahrzeug oder Vegetation
* Bewölkte Bedingungen liefern die konsistentesten Ergebnisse

**Zielbedingungen:**

* Halten Sie die Zieltafeln sauber und trocken
* Alle 4 Tafeln sollten gut sichtbar und frei von Hindernissen sein
* Ziele sollten möglichst senkrecht/nadir zur Lichtquelle stehen

### Wie viele Zielbilder?

**Minimum:** 1 Zielbild pro Sitzung. **Empfohlen:** 3–5 Zielbilder pro Sitzung.

**Empfohlener Zeitplan:**

* 3–5 Bilder kurz nach Beginn der Aufzeichnung durch den Lichtsensor aufnehmen.
* Die Kamera zwischen den Aufnahmen drehen, um optimale Ergebnisse zu erzielen.
* Optional: bei sich ständig ändernden Lichtverhältnissen regelmäßig während der Sitzung.

***

## Arbeiten mit mehreren Kameras

### Dual-Kamera-Setups

Bei gleichzeitiger Verwendung von zwei MAPIR-Kameras (z. B. Survey3W RGN + Survey3N OCN):

1. Nehmen Sie die Zielbilder mit **beiden Kameras** gleichzeitig auf.
2. Verwenden Sie für beide Kameras dasselbe physische Ziel.
3. Markieren Sie die Zielbilder für **beide Kameratypen** im Dateibrowser.
4. Chloros verwendet für die Kalibrierung jeder Kamera die entsprechenden Ziele.

### Spalte „Kameramodell”

Die Spalte **Kameramodell** hilft dabei, zu erkennen, welche Bilder von welcher Kamera stammen:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* usw.

Verwenden Sie diese Spalte, um zu überprüfen, ob Sie Ziele für jeden Kameratyp in Ihrem Projekt markiert haben.

***

## Einstellungen für die Zielerkennung

### Anpassen der Erkennungsempfindlichkeit

Wenn Chloros Ihre Ziele nicht richtig erkennt, passen Sie diese Einstellungen in den [Projekteinstellungen](adjusting-project-settings.md) an:

**Mindestkalibrierungsbereich:**

* **Standard**: 25 Pixel
* **Erhöhen** Sie diesen Wert, wenn bei kleinen Artefakten Fehlalarme auftreten.
* **Verringern** Sie diesen Wert, wenn Ziele nicht erkannt werden.

**Mindestzielclusterung:**

* **Standard**: 60
* **Erhöhen** Sie den Wert, wenn Ziele in mehrere Erkennungen aufgeteilt werden.
* **Verringern** Sie den Wert, wenn Ziele mit Farbabweichungen nicht vollständig erkannt werden.

***

## Häufige Probleme mit Zielbildern

### Problem: Keine Ziele erkannt

**Mögliche Ursachen:**

* Zielbilder sind im Dateibrowser nicht markiert.
* Das Ziel ist im Rahmen zu klein (&lt; 30 % des Bildes).
* Schlechte Beleuchtung (Schatten, Blendung)
* Zu strenge Einstellungen für die Zielerkennung

**Lösungen:**

1. Überprüfen Sie, ob die Spalte „Ziel“ für die richtigen Bilder markiert ist.
2. Überprüfen Sie die Qualität des Zielbildes in der Vorschau.
3. Nehmen Sie die Ziele erneut auf, wenn die Qualität schlecht ist.
4. Passen Sie bei Bedarf die Einstellungen für die Zielerkennung an.

### Problem: Falsche Zielerkennungen

**Mögliche Ursachen:**

* Weiße Gebäude, Fahrzeuge oder Bodenbedeckung werden fälschlicherweise als Ziele erkannt.
* Helle Flecken in der Vegetation.
* Erkennungsempfindlichkeit zu niedrig.

**Lösungen:**

1. Markieren Sie nur tatsächliche Zielbilder, um den Erkennungsbereich zu begrenzen.
2. Erhöhen Sie den minimalen Kalibrierungsbereich.
3. Erhöhen Sie den minimalen Zielclusterwert.
4. Stellen Sie sicher, dass die Zielbilder nur das Ziel zeigen (minimale Hintergrundstörungen).

***

## Überprüfungscheckliste

Überprüfen Sie vor Beginn der Verarbeitung Ihre Zielbildauswahl:

* [ ] Mindestens 1 Zielbild pro Sitzung markiert
* [ ] Die Kontrollkästchen der Zielspalte sind für alle Zielbilder aktiviert
* [ ] Zielbilder wurden im gleichen Zeitraum wie die Vermessung aufgenommen
* [ ] Ziele sind in der Vorschau beim Anklicken deutlich sichtbar
* [ ] Alle 4 Kalibrierungsfelder sind in jedem Zielbild sichtbar
* [ ] Keine Schatten oder Hindernisse auf den Zielen
* [ ] Bei zwei Kameras: Ziele für beide Kameratypen markiert

***

## Zielfreie Verarbeitung

### Verarbeitung ohne Kalibrierungsziele

Obwohl dies für wissenschaftliche Arbeiten nicht empfohlen wird, können Sie die Verarbeitung auch ohne Ziele durchführen:

1. Lassen Sie alle Kontrollkästchen in der Spalte „Ziel“ deaktiviert.
2. **Deaktivieren** Sie „Reflexionskalibrierung“ in den Projekteinstellungen.
3. Die Vignettenkorrektur wird weiterhin angewendet.
4. Die Ausgabe wird nicht für die absolute Reflexion kalibriert.

{% Hinweis style=&quot;warning&quot; %}
**Nicht empfohlen**: Ohne Reflektionskalibrierung geben die Pixelwerte nur die relative Helligkeit wieder, nicht die wissenschaftlichen Reflektionsmessungen. Verwenden Sie Kalibrierungsziele für genaue, wiederholbare Ergebnisse.
{% endhint %}

***

## Nächste Schritte

Nachdem Sie Ihre Zielbilder markiert haben:

1. **Überprüfen Sie Ihre Einstellungen** – Siehe [Anpassen der Projekteinstellungen](adjusting-project-settings.md)
2. **Starten Sie die Verarbeitung** – Siehe [Starten der Verarbeitung](starting-the-processing.md)
3. **Überwachen Sie den Fortschritt** – Siehe [Überwachen der Verarbeitung](monitoring-the-processing.md)

Weitere Informationen zu Kalibrierungszielen finden Sie unter [Kalibrierungsziele](../calibration-targets.md).
