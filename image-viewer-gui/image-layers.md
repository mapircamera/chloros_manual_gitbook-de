# Bild-Ebenen

Über das Dropdown-Menü „Bild-Ebenen“ im Chloros-Bildbetrachter können Sie schnell zwischen verschiedenen Versionen desselben Bildes wechseln – von den Originalaufnahmen über bearbeitete Reflexionsausgaben bis hin zu berechneten Indexbildern.

## Was sind Bild-Ebenen?

In Chloros beziehen sich **Ebenen** auf die verschiedenen Bildausgaben, die für ein einzelnes Quellbild verfügbar sind. Wenn Sie Bilder verarbeiten, erstellt Chloros mehrere Versionen:

* **Originalbilder** (JPG- und RAW-Dateien von Ihrer Kamera)
* **Reflexionskalibrierte** Ausgaben (sofern die Reflexionskalibrierung aktiviert war)
* **Zielbilder** (sofern das Bild Kalibrierungsziele enthält)
* **Indexbilder** (NDVI, NDRE, GNDVI usw., sofern Indizes konfiguriert wurden)

Über das **Dropdown-Menü „Layer Selector“** oben rechts im Bildbetrachter können Sie sofort zwischen diesen Versionen wechseln, ohne den Betrachter zu verlassen.***

## Verfügbare Ebenentypen

### JPG

* Das ursprüngliche JPG-Vorschaubild von Ihrer Kamera
* Immer für alle Bilder verfügbar
* Unbearbeitet, wie von der Kamera aufgenommen
* Am schnellsten zu laden und anzuzeigen

**Wann anzeigen:**

* Schnelle Vorschau der Originalaufnahme
* Überprüfen der Bildkomposition und des Bildausschnitts
* Überprüfen der Aufnahmequalität vor der Bearbeitung

### RAW (Original)

* Die ursprünglichen RAW-Sensordaten Ihrer Kamera
* Debayert, ohne Nachbearbeitung
* Höhere Bittiefe als JPG (typischerweise 12-Bit- oder 14-Bit-Sensordaten)

**Wann anzeigen:**

* Überprüfung der Qualität der ursprünglichen Sensordaten
* Überprüfung auf Sensorprobleme oder Artefakte
* Vergleich der Ergebnisse vor und nach der Bearbeitung

### RAW (Ziel)

* Erscheint nur bei Bildern, die Kalibrierungsziele enthalten
* Zeigt das ursprüngliche RAW-Bild mit erkanntem Ziel an
* Dient zur Überprüfung, ob die Zielerkennung erfolgreich war

**Wann anzeigen:**

* Bestätigung, dass Kalibrierungsziele korrekt erkannt wurden
* Überprüfung der Bildqualität der Targets
* Fehlerbehebung bei Kalibrierungsproblemen

{% hint style="info" %}
**Target-Ebene**: Diese Ebene erscheint nur in der Dropdown-Liste für Bilder, die Kalibrierungstargets enthalten. Normale Aufnahmen verfügen nicht über diese Option.
{% endhint %}

### RAW (Reflexion)

* Das kalibrierte Reflexionsausgabebild
* Vignettierungskorrektur (sofern bei der Verarbeitung aktiviert)
* Reflektanz kalibriert anhand von Zielwerten (sofern aktiviert)
* Multiband-TIFF mit allen Kamerakanälen
* Pixelwerte stellen die Reflektanz in Prozent dar (bei Verwendung des Prozentmodus)
* Bereit zur Bearbeitung mit der [Index/LUT-Sandbox](index-lut-sandbox.md)

**Anwendungsfälle:**

* Überprüfung der kalibrierten Ergebnisse
* Überprüfung der Kalibrierungsqualität
* Überprüfung der Pixelwerte auf wissenschaftliche Genauigkeit
* Vergleich mit dem Original, um Kalibrierungseffekte zu erkennen

{% hint style="success" %}
**Empfohlen**: Verwenden Sie die RAW-Ebene (Reflexion), wenn Sie Pixelwerte für wissenschaftliche Messungen und Analysen überprüfen.
{% endhint %}

### RAW (NDVI-Index)... und ähnliche

* Berechnetes Vegetationsindexbild (in diesem Beispiel NDVI)
* Der Indexname ändert sich je nachdem, welcher Index während der Verarbeitung konfiguriert wurde
* Beispiele: RAW (NDVI-Index), RAW (NDRE-Index), RAW (GNDVI-Index) usw.
* Einband-Graustufenbild, das die Ergebnisse der Indexberechnung zeigt
* Für jeden in den Projekteinstellungen konfigurierten Index erscheint eine Ebene

**Mögliche Indexnamen:**

* RAW (NDVI-Index)
* RAW (NDRE-Index)
* RAW (GNDVI-Index)
* RAW (OSAVI-Index)
* RAW (EVI-Index)
* RAW (SAVI-Index)
* Und viele mehr... (siehe [Multispektrale Indexformeln](../project-settings/multispectral-index-formulas.md))

**Anwendungsfälle:**

* Überprüfung der Ergebnisse der Indexberechnung
* Überprüfen der Indexwertbereiche
* Identifizieren von Bereichen von Interesse
* Überprüfen von Indexbildern vor der Verwendung in GIS oder Analysen

***

## Verwenden der Ebenenauswahl

### Öffnen des Dropdown-Menüs

1. Öffnen Sie ein Bild im Vollbildmodus (klicken Sie auf eine beliebige Miniaturansicht im Bildbetrachter)
2. Suchen Sie das **Ebenen-Dropdown-Menü** in der oberen rechten Ecke des Bildbetrachters
3. Das Dropdown-Menü zeigt die aktuell ausgewählte Ebene an (z. B. „JPG“)
4. Klicken Sie auf das Dropdown-Menü, um alle verfügbaren Ebenen anzuzeigen

### Ebenen wechseln

1. Klicken Sie auf das Ebenen-Dropdown-Menü, um die Liste zu öffnen
2. Alle für das aktuelle Bild verfügbaren Ebenen werden angezeigt
3. Klicken Sie auf einen beliebigen Ebenennamen, um zu dieser Version zu wechseln
4. Das Bild wird sofort aktualisiert und zeigt die ausgewählte Ebene an

**Schnellwechsel:**

* Das Dropdown-Menü merkt sich Ihre letzte Auswahl
* Beim Wechsel zum nächsten Bild versucht Chloros, denselben Ebenentyp anzuzeigen
* Wenn diese Ebene im nächsten Bild nicht vorhanden ist, wird standardmäßig JPG angezeigt

### Verfügbarkeit von Ebenen

Nicht alle Ebenen sind für jedes Bild verfügbar:

**Immer verfügbar:*** ✅ JPG (jedes Bild verfügt über eine JPG-Vorschau)

**Bedingt verfügbar:**

* ⚠️ RAW (Original) – Nur wenn das Bild im RAW- oder RAW+JPG-Modus aufgenommen wurde
* ⚠️ RAW (Ziel) – Nur wenn das Bild erkannte Kalibrierungsziele enthält
* ⚠️ RAW (Reflexion) – Nur nach der Verarbeitung mit aktivierter Reflexionskalibrierung
* ⚠️ RAW (\[Index] Index) – Nur nach der Verarbeitung mit konfigurierten Indizes

***

## Persistenz der Ebenen

### Zwischen Bildern navigieren

Wenn Sie zu einem anderen Bild navigieren (mithilfe der Pfeiltasten oder durch Klicken auf Miniaturansichten):**Die Ebeneneinstellung bleibt erhalten:**

* Wenn Sie „RAW (Reflexion)“ anzeigen, wird im nächsten Bild „RAW (Reflexion)“ angezeigt (sofern verfügbar)
* Wenn Sie „RAW (NDVI Index)“ anzeigen, wird im nächsten Bild „RAW (NDVI Index)“ (sofern verfügbar)
* Wenn dieselbe Ebene nicht vorhanden ist, wird standardmäßig JPG angezeigt

**Beispiel-Workflow:**

1. Bild 1 öffnen, zu RAW (NDVI Index) wechseln
2. → drücken, um Bild 2 anzuzeigen
3. Bild 2 zeigt automatisch die Ebene RAW (NDVI Index) an
4. Weiterblättern – alle Bilder zeigen die NDVI-Ebene
5. Sehr effizient zum Überprüfen der Indexergebnisse über viele Bilder hinweg

***

## Gängige Arbeitsabläufe

### Arbeitsablauf 1: Vorher-Nachher-Vergleich

**Ziel**: Vergleichen des Originalbildes mit dem kalibrierten Bild

1. Bearbeitetes Bild im Bildbetrachter öffnen
2. Wählen Sie **RAW (Original)** aus der Dropdown-Liste
3. Beachten Sie die Vignettierung und die unkalibrierten Werte
4. Wechseln Sie in der Dropdown-Liste zu **RAW (Reflexion)**

5. Vergleichen – Vignettierung entfernt, Werte kalibriert

### Arbeitsablauf 2: Indexüberprüfung

**Ziel**: Schnelle Überprüfung der NDVI-Ergebnisse im gesamten Datensatz

1. Erstes bearbeitetes Bild öffnen
2. Wählen Sie **RAW (NDVI Index)** aus der Dropdown-Liste
3. Verwenden Sie die Pfeiltaste →, um zum nächsten Bild zu navigieren
4. Die Ebene NDVI bleibt automatisch bestehen
5. Fahren Sie mit allen Bildern fort und überprüfen Sie die Muster von NDVI
6. Wechseln Sie zu **RAW (NDRE Index)**, um zu vergleichen

### Arbeitsablauf 3: Zielüberprüfung

**Ziel**: Überprüfen, ob alle Zielbilder korrekt erkannt wurden

1. Navigieren Sie zu einem Zielbild
2. Wählen Sie **RAW (Target)** aus der Dropdown-Liste
3. Überprüfen Sie, ob die Kalibrierungsziele deutlich sichtbar sind und erkannt wurden
4. Navigieren Sie zum nächsten Zielbild
5. Wiederholen Sie die Überprüfung für alle Ziele

### Arbeitsablauf 4: Überprüfung der Pixelwerte

**Ziel**: Überprüfen Sie die Reflexionswerte auf wissenschaftliche Genauigkeit

1. Öffnen Sie das verarbeitete Bild
2. Wählen Sie die Ebene **RAW (Reflexion)**

3. Aktivieren Sie den Modus**Pixelprozent** (Schaltfläche in der Symbolleiste oben rechts)
4. Bewegen Sie den Cursor über Vegetationsbereiche
5. Überprüfen Sie, ob die Pixelwerte im erwarteten Bereich liegen (30–70 % für NIR, 5–15 % für Red)
6. Überprüfen Sie Boden- und Wasserflächen auf angemessene Werte

***

## Pixelwerte nach Ebene verstehen

Verschiedene Ebenen weisen unterschiedliche Pixelwertbereiche auf:

### JPG-Ebene

* **Bereich**: 0–255 (8 Bit)
* **Bedeutung**: Anzeigewerte, gammakorrigiert
* **Verwendung**: Nur zur visuellen Überprüfung, nicht für wissenschaftliche Messungen

### RAW (Original)

* **Bereich**: 0–65535 (16-Bit)
* **Bedeutung**: Digitale Rohdaten des Sensors
* **Verwendung**: Überprüfung der Sensorleistung, nicht kalibriert

### RAW (Reflexion)

* **Bereich**: 0–65.535 (16-Bit TIFF) oder 0,0–1,0 (32-Bit Prozent)
* **Bedeutung**: Kalibrierte Reflexionsprozentsätze
* **Verwendung**: Wissenschaftliche Messungen und Analysen**Für 16-Bit TIFF:**Durch 65.535 dividieren, um den Reflexionsprozentsatz zu erhalten**Für 32-Bit Prozent:** Die Werte stellen direkt den Prozentsatz dar (0,5 = 50 % Reflexion)

### RAW (Indexbilder)

* **Bereich**: Variiert je nach Index (typischerweise -1,0 bis +1,0 für normalisierte Indizes)
* **Bedeutung**: Ergebnis der Indexberechnung
* **Beispiele**:
  * NDVI: -1 bis +1 (Vegetation typischerweise 0,4 bis 0,9)
  * NDRE: -1 bis +1 (Stresserkennung)
  * EVI: 0 bis 1 (verbesserte Vegetation)

***

## Tipps und bewährte Vorgehensweisen

### Effizientes Wechseln zwischen Ebenen

* **Tastaturkürzel**: Es gibt zwar keine Tastaturkürzel für Ebenen, aber die Navigationstasten (←/→) funktionieren über alle Ebenen hinweg
* **Konsistente Arbeitsabläufe**: Wählen Sie eine Ebene aus (z. B. NDVI) und überprüfen Sie den gesamten Datensatz, bevor Sie zu einer anderen wechseln
* **Schnelle Vergleiche**: Wechseln Sie zwischen „Original“ und „Reflexion“, um die Verarbeitungsqualität zu überprüfen

### Überlegungen zur Leistung

* **JPG-Dateien werden am schnellsten geladen**: Verwenden Sie diese für die schnelle Navigation durch viele Bilder
* **RAW-Ebenen werden langsamer geladen**: Höhere Auflösung und Bittiefe
* **Index-Ebenen**: Ähnliche Geschwindigkeit wie Reflexions-Ebenen
* **Das erste Laden dauert am längsten**: Nachfolgende Ansichten derselben Ebene werden zwischengespeichert und sind schneller

### Qualitätsprüfung

* **Prüfen Sie immer RAW (Original)**: Überprüfen Sie die Qualität der Quelldaten, bevor Sie den verarbeiteten Ergebnissen vertrauen
* **Vergleichen Sie Ebenen**: Nutzen Sie den Ebenenwechsel, um zu überprüfen, ob die Verarbeitung korrekt funktioniert hat
* **Prüfen Sie Indexbereiche**: Verwenden Sie den Modus „Pixelprozent“ mit Indexebenen, um zu überprüfen, ob die Werte plausibel sind***

## Fehlerbehebung

### Ebene nicht verfügbar

**Problem**: Die erwartete Ebene erscheint nicht im Dropdown-Menü**Mögliche Ursachen:**

* Das Bild wurde nicht verarbeitet (nur JPG und RAW (Original) verfügbar)
* Die Reflektanzkalibrierung war während der Verarbeitung deaktiviert
* Der spezifische Index wurde in den Projekteinstellungen nicht konfiguriert
* Das Bild ist ein reines Zielbild (für Ziele werden keine Indizes generiert)

**Lösungen:**

1. Überprüfen Sie, ob das Bild verarbeitet wurde (suchen Sie im Ausgabeordner nach verarbeiteten Dateien)
2. Überprüfen Sie die Projekteinstellungen, um sicherzustellen, dass Indizes konfiguriert wurden
3. Führen Sie die Verarbeitung erneut durch, wobei die gewünschten Indizes aktiviert sind

### Falsche Ebene angezeigt

**Problem**: Das Bild wird in einer unerwarteten Ebene geöffnet**Ursache**: Die Ebeneneinstellung aus dem vorherigen Bild wurde übernommen, aber diese Ebene existiert im aktuellen Bild nicht**Lösung**: Chloros wechselt automatisch zu JPG, wenn die bevorzugte Ebene nicht verfügbar ist – dies ist normales Verhalten

### Kalibrierungsziele sind nicht sichtbar

**Problem**: Die RAW-Ebene (Zielebene) zeigt keine Zielerkennung an**Mögliche Ursachen:**

* Ziele wurden während der Verarbeitung nicht erkannt
* Das Bild enthält tatsächlich keine Ziele
* Einstellungen für die Zielerkennung sind zu streng

**Lösungen:**

1. Überprüfen Sie das Debug-Protokoll auf Meldungen wie „Ziel gefunden“
2. Vergewissern Sie sich, dass das Bild tatsächlich sichtbare Kalibrierungsziele enthält
3. Passen Sie die Einstellungen für die Zielerkennung in den Projekteinstellungen an
4. Siehe [Auswahl von Zielbildern](../processing-images-gui/choosing-target-images.md)

***

## Verwandte Funktionen

### Bildbetrachtungswerkzeuge

Beim Anzeigen einer beliebigen Ebene können Sie Folgendes verwenden:

* **Zoomsteuerung**: Vergrößern, um Details zu untersuchen
* **Verschieben**: Klicken und ziehen, um sich im vergrößerten Bild zu bewegen
* **Pixelwertprüfung**: Werte an der Cursorposition anzeigen
* **Navigationspfeile**: Zwischen Bildern wechseln, während die Ebene beibehalten wird
* **Pixelprozent-Modus**: Zwischen DN- und Prozentanzeige umschalten

Die vollständige Dokumentation zum Bildbetrachter finden Sie unter [Bild im Vollbildmodus öffnen](opening-an-image-full-screen.md).

### Index/LUT-Sandbox

Für interaktive Index-Tests und Visualisierung:

* **Echtzeit-Indexberechnung**: Testen Sie verschiedene Indexformeln
* **LUT-Farbzuordnung**: Wenden Sie Farbverläufe auf Graustufen-Indizes an
* **Visualisierungen exportieren**: Speichern Sie farbige Indexbilder

Weitere Informationen finden Sie unter [Index/LUT-Sandbox](index-lut-sandbox.md).

***

## Nächste Schritte

Nachdem Sie nun die Bildebenen verstanden haben:

* [**Ein Bild im Vollbildmodus öffnen**](opening-an-image-full-screen.md) – Vollständige Anleitung zum Image Viewer
* [**Index/LUT-Sandbox**](index-lut-sandbox.md) – Interaktive Indexvisualisierung
* [**Multispektrale Indexformeln**](../project-settings/multispectral-index-formulas.md) – Referenz der verfügbaren Indizes
* [**Abschluss der Verarbeitung**](../processing-images-gui/finishing-the-processing.md) – Verständnis der verarbeiteten Ergebnisse
