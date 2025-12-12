# Bildlayer

Über das Dropdown-Menü „Bildlayer“ im Chloros-Bildbetrachter können Sie schnell zwischen verschiedenen Versionen desselben Bildes wechseln – von den Originalaufnahmen über die verarbeiteten Reflektionsausgaben bis hin zu den berechneten Indexbildern.

## Was sind Bildlayer?

In Chloros beziehen sich **Layer** auf die verschiedenen Bildausgaben, die für ein einzelnes Quellbild verfügbar sind. Wenn Sie Bilder verarbeiten, erstellt Chloros mehrere Versionen:

* **Originalbilder** (JPG- und RAW-Dateien von Ihrer Kamera)
* **Reflexionskalibrierte** Ausgaben (wenn die Reflexionskalibrierung aktiviert war)
* **Zielbilder** (wenn das Bild Kalibrierungsziele enthält)
* **Indexbilder** (NDVI, NDRE, GNDVI usw., wenn Indizes konfiguriert wurden)

Über das **Dropdown-Menü „Layer Selector“** oben rechts im Bildbetrachter können Sie sofort zwischen diesen Versionen wechseln, ohne den Betrachter zu verlassen.

***

## Verfügbare Layer-Typen

### JPG

* Das originale JPG-Vorschaubild Ihrer Kamera
* Immer für alle Bilder verfügbar
* Unbearbeitet, wie von der Kamera aufgenommen
* Am schnellsten zu laden und anzuzeigen

**Wann anzeigen:**

* Schnelle Vorschau der Originalaufnahme
* Überprüfen der Bildkomposition und des Bildausschnitts
* Überprüfen der Aufnahmequalität vor der Bearbeitung

### RAW (Original)

* Die ursprünglichen RAW-Sensordaten Ihrer Kamera
* Debayering ohne Nachbearbeitung
* Höhere Bittiefe als JPG (in der Regel 12-Bit- oder 14-Bit-Sensordaten)

**Anzeigezweck:**

* Überprüfen der Qualität der ursprünglichen Sensordaten
* Überprüfen auf Sensorprobleme oder Artefakte
* Vergleichen der Ergebnisse vor und nach der Bearbeitung

### RAW (Ziel)

* Wird nur für Bilder angezeigt, die als kalibrierungszielhaltig identifiziert wurden
* Zeigt das Original-RAW-Bild mit erkanntem Ziel an
* Dient zur Überprüfung, ob die Zielerkennung erfolgreich war

**Anzeigezweck:**

* Bestätigung, dass die Kalibrierungsziele korrekt erkannt wurden
* Überprüfung der Bildqualität des Ziels
* Fehlerbehebung bei Kalibrierungsproblemen

{% hint style=&quot;info&quot; %}
**Zielebene**: Diese Ebene wird nur im Dropdown-Menü für Bilder angezeigt, die Kalibrierungsziele enthalten. Normale Aufnahmen verfügen nicht über diese Option.
{% endhint %}

### RAW (Reflexionsgrad)

* Das kalibrierte Reflexionsgrad-Ausgabebild
* Vignettierung korrigiert (sofern in der Verarbeitung aktiviert)
* Reflexion kalibriert anhand von Zieldaten (sofern aktiviert)
* Multiband TIFF mit allen Kamerakanälen
* Pixelwerte stellen die prozentuale Reflexion dar (bei Verwendung des Prozentmodus)
* Bereit zur Bearbeitung mit der [Index/LUT-Sandbox](index-lut-sandbox.md)

**Anzeigezwecke:**

* Überprüfen der kalibrierten Ergebnisse
* Überprüfen der Kalibrierungsqualität
* Überprüfen der Pixelwerte auf wissenschaftliche Genauigkeit
* Vergleichen mit dem Original, um die Kalibrierungseffekte zu sehen

{% hint style=&quot;success&quot; %}
**Empfohlen**: Verwenden Sie die RAW-Ebene (Reflexionsgrad), wenn Sie Pixelwerte für wissenschaftliche Messungen und Analysen überprüfen.
{% endhint %}

### RAW (NDVI Index)... und ähnliche

* Berechnetes Vegetationsindexbild (in diesem Beispiel NDVI)
* Der Name des Index ändert sich je nachdem, welcher Index während der Verarbeitung konfiguriert wurde
* Beispiele: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) usw.
* Einbandiges Graustufenbild, das die Ergebnisse der Indexberechnung zeigt
* Für jeden in den Projekteinstellungen konfigurierten Index wird eine Ebene angezeigt

**Mögliche Indexnamen:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI Index)
* RAW (EVI-Index)
* RAW (SAVI-Index)
* Und viele mehr... (siehe [Multispektrale Indexformeln](../project-settings/multispectral-index-formulas.md))

**Anzeigezwecke:**

* Überprüfen der Indexberechnungsergebnisse
* Überprüfen der Indexwertbereiche
* Identifizieren von Bereichen von Interesse
* Überprüfen der Indexbilder vor der Verwendung in GIS oder Analysen

***

## Verwenden des Ebenenauswahlfelds

### Öffnen des Dropdown-Menüs

1. Öffnen Sie ein Bild im Vollbildmodus (klicken Sie auf eine beliebige Miniaturansicht im Bildbetrachter).
2. Suchen Sie das **Ebenen-Dropdown-Menü** in der oberen rechten Ecke des Betrachters.
3. Das Dropdown-Menü zeigt die aktuell ausgewählte Ebene an (z. B. „JPG“).
4. Klicken Sie auf das Dropdown-Menü, um alle verfügbaren Ebenen anzuzeigen.

### Wechseln zwischen Ebenen

1. Klicken Sie auf das Ebenen-Dropdown-Menü, um die Liste zu öffnen.
2. Alle für das aktuelle Bild verfügbaren Ebenen werden angezeigt.
3. Klicken Sie auf einen beliebigen Ebenennamen, um zu dieser Version zu wechseln.
4. Das Bild wird sofort aktualisiert und zeigt die ausgewählte Ebene an.

**Schnelles Wechseln:**

* Das Dropdown-Menü merkt sich Ihre letzte Auswahl.
* Beim Navigieren zum nächsten Bild versucht Chloros, denselben Ebenentyp anzuzeigen.
* Wenn diese Ebene im nächsten Bild nicht vorhanden ist, wird standardmäßig JPG angezeigt.

### Verfügbarkeit von Ebenen

Nicht alle Ebenen sind für jedes Bild verfügbar:

**Immer verfügbar:**

* ✅ JPG (jedes Bild hat eine JPG-Vorschau)

**Bedingt verfügbar:**

* ⚠️ RAW (Original) – Nur wenn das Bild im RAW- oder RAW+JPG-Modus aufgenommen wurde
* ⚠️ RAW (Ziel) – Nur wenn das Bild erkannte Kalibrierungsziele enthält
* ⚠️ RAW (Reflexionsgrad) – Nur nach der Verarbeitung mit aktivierter Reflexionsgradkalibrierung
* ⚠️ RAW (\[Index] Index) – Nur nach der Verarbeitung mit konfigurierten Indizes

***

## Layer-Persistenz

### Zwischen Bildern navigieren

Wenn Sie zu einem anderen Bild navigieren (mit den Pfeiltasten oder durch Klicken auf Miniaturansichten):

**Die Layer-Einstellung bleibt erhalten:**

* Wenn Sie „RAW (Reflexionsgrad)” anzeigen, wird das nächste Bild als „RAW (Reflexionsgrad)” angezeigt (sofern verfügbar).
* Wenn Sie „RAW (NDVI Index)” anzeigen, wird als nächstes Bild „RAW (NDVI Index)” angezeigt (sofern verfügbar).
* Wenn dieselbe Ebene nicht vorhanden ist, wird standardmäßig JPG angezeigt.

**Beispiel-Workflow:**

1. Öffnen Sie Bild 1 und wechseln Sie zu RAW (NDVI Index)
2. Drücken Sie →, um Bild 2 anzuzeigen.
3. Bild 2 zeigt automatisch die Ebene RAW (NDVI Index) an.
4. Fahren Sie mit der Navigation fort – alle Bilder zeigen die Ebene NDVI an.
5. Sehr effizient für die Überprüfung von Indexergebnissen über viele Bilder hinweg.

***

## Häufige Arbeitsabläufe

### Arbeitsablauf 1: Vorher/Nachher-Vergleich

**Ziel**: Vergleichen Sie das Originalbild mit dem kalibrierten Bild.

1. Öffnen Sie das verarbeitete Bild im Bildbetrachter.
2. Wählen Sie **RAW (Original)** aus der Dropdown-Liste.
3. Beachten Sie die Vignettierung und die unkalibrierten Werte.
4. Wechseln Sie in der Dropdown-Liste zu **RAW (Reflexionsgrad)**.
5. Vergleichen – Vignettierung entfernt, Werte kalibriert.

### Arbeitsablauf 2: Indexüberprüfung

**Ziel**: Schnelle Überprüfung der NDVI-Ergebnisse im gesamten Datensatz.

1. Öffnen Sie das erste verarbeitete Bild.
2. Wählen Sie **RAW (NDVI Index)** aus der Dropdown-Liste.
3. Verwenden Sie die Pfeiltaste →, um zum nächsten Bild zu navigieren.
4. Die Ebene NDVI bleibt automatisch erhalten.
5. Fahren Sie mit allen Bildern fort und überprüfen Sie die Muster NDVI.
6. Wechseln Sie zum Vergleich zu **RAW (NDRE Index)**.

### Arbeitsablauf 3: Zielüberprüfung

**Ziel**: Überprüfen Sie, ob alle Zielbilder korrekt erkannt wurden.

1. Navigieren Sie zu einem Zielbild.
2. Wählen Sie **RAW (Ziel)** aus dem Dropdown-Menü.
3. Überprüfen Sie, ob die Kalibrierungsziele deutlich sichtbar sind und erkannt werden.
4. Navigieren Sie zum nächsten Zielbild.
5. Wiederholen Sie die Überprüfung für alle Ziele.

### Arbeitsablauf 4: Überprüfung der Pixelwerte

**Ziel**: Überprüfen Sie die Reflexionswerte auf wissenschaftliche Genauigkeit.

1. Öffnen Sie das verarbeitete Bild.
2. Wählen Sie die Ebene **RAW (Reflexion)** aus.
3. Aktivieren Sie den Modus **Pixelprozent** (Schaltfläche in der Symbolleiste oben rechts).
4. Bewegen Sie den Cursor über Vegetationsbereiche.
5. Überprüfen Sie, ob die Pixelwerte im erwarteten Bereich liegen (30–70 % für NIR, 5–15 % für Red).
6. Überprüfen Sie, ob die Werte für Boden- und Wasserflächen angemessen sind.

***

## Pixelwerte nach Ebenen verstehen

Verschiedene Ebenen zeigen unterschiedliche Pixelwertbereiche an:

### JPG-Ebene

* **Bereich**: 0–255 (8 Bit)
* **Bedeutung**: Anzeige der Werte, gammakorrigiert
* **Verwendung**: Nur zur visuellen Überprüfung, nicht für wissenschaftliche Messungen

### RAW (Original)

* **Bereich**: 0–65535 (16 Bit)
* **Bedeutung**: Digitale Rohdaten des Sensors
* **Verwendung**: Überprüfung der Sensorleistung, nicht kalibriert

### RAW (Reflexionsgrad)

* **Bereich**: 0–65.535 (16 Bit TIFF) oder 0,0–1,0 (32 Bit Prozent)
* **Bedeutung**: Kalibrierte prozentuale Reflexion
* **Verwendung**: Wissenschaftliche Messungen und Analysen

**Für 16-Bit TIFF:** Durch 65.535 teilen, um die prozentuale Reflexion zu erhalten **Für 32-Bit Prozent:** Die Werte geben direkt den Prozentsatz an (0,5 = 50 % Reflexion)

### RAW (Indexbilder)

* **Bereich**: Variiert je nach Index (typischerweise -1,0 bis +1,0 für normalisierte Indizes)
* **Bedeutung**: Ergebnis der Indexberechnung
* **Beispiele**:
  * NDVI: -1 bis +1 (Vegetation typischerweise 0,4 bis 0,9)
  * NDRE: -1 bis +1 (Stresserkennung)
  * EVI: 0 bis 1 (verbesserte Vegetation)

***

## Tipps und bewährte Verfahren

### Effizientes Wechseln zwischen Ebenen

* **Tastaturkürzel**: Es gibt zwar keine Tastaturkürzel für Ebenen, aber die Navigationspfeile (←/→) funktionieren für alle Ebenen.
* **Konsistente Arbeitsabläufe**: Wählen Sie eine Ebene aus (z. B. NDVI) und überprüfen Sie den gesamten Datensatz, bevor Sie zu einer anderen wechseln.
* **Schnelle Vergleiche**: Wechseln Sie zwischen „Original“ und „Reflexion“, um die Verarbeitungsqualität zu überprüfen.

### Leistungsaspekte

* **JPG wird am schnellsten geladen**: Verwenden Sie dieses Format für die schnelle Navigation durch viele Bilder.
* **RAW-Ebenen werden langsamer geladen**: Höhere Auflösung und Bittiefe.
* **Indexebenen**: Ähnliche Geschwindigkeit wie Reflektions-Ebenen.
* **Das erste Laden ist am langsamsten**: Nachfolgende Ansichten derselben Ebene werden zwischengespeichert und sind schneller.

### Qualitätsüberprüfung

* **Überprüfen Sie immer RAW (Original)**: Überprüfen Sie die Qualität der Quelldaten, bevor Sie den verarbeiteten Ergebnissen vertrauen.
* **Ebenen vergleichen**: Verwenden Sie die Ebenenumschaltung, um zu überprüfen, ob die Verarbeitung korrekt funktioniert hat
* **Indexbereiche überprüfen**: Verwenden Sie den Pixelprozentmodus mit Indexebenen, um zu überprüfen, ob die Werte angemessen sind

***

## Fehlerbehebung

### Ebene nicht verfügbar

**Problem**: Die erwartete Ebene wird nicht in der Dropdown-Liste angezeigt

**Mögliche Ursachen:**

* Das Bild wurde nicht verarbeitet (nur JPG und RAW (Original) verfügbar).
* Die Reflektionskalibrierung wurde während der Verarbeitung deaktiviert.
* Der spezifische Index wurde nicht in den Projekteinstellungen konfiguriert.
* Das Bild ist ein reines Zielbild (es wurden keine Indizes für Ziele generiert).

**Lösungen:**

1. Überprüfen Sie, ob das Bild verarbeitet wurde (überprüfen Sie den Ausgabeordner auf verarbeitete Dateien).
2. Überprüfen Sie die Projekteinstellungen, um sicherzustellen, dass die Indizes konfiguriert wurden.
3. Führen Sie die Verarbeitung erneut durch und aktivieren Sie die gewünschten Indizes.

### Falsche Ebene angezeigt

**Problem**: Das Bild wird in einer unerwarteten Ebene geöffnet.

**Ursache**: Die Ebeneneinstellung aus dem vorherigen Bild wurde übernommen, aber diese Ebene ist im aktuellen Bild nicht vorhanden.

**Lösung**: Chloros wechselt automatisch zu JPG, wenn die bevorzugte Ebene nicht verfügbar ist – dies ist ein normales Verhalten.

### Kalibrierungsziele werden nicht angezeigt

**Problem**: Die RAW-Ebene (Ziel) zeigt keine Zielerkennung an.

**Mögliche Ursachen:**

* Die Ziele wurden während der Verarbeitung nicht erkannt.
* Das Bild enthält tatsächlich keine Ziele.
* Die Einstellungen für die Zielerkennung sind zu streng.

**Lösungen:**

1. Überprüfen Sie das Debug-Protokoll auf Meldungen wie „Ziel gefunden“.
2. Vergewissern Sie sich, dass das Bild tatsächlich sichtbare Kalibrierungsziele enthält.
3. Passen Sie die Einstellungen für die Zielerkennung in den Projekteinstellungen an.
4. Siehe [Auswahl von Zielbildern](../processing-images-gui/choosing-target-images.md).

***

## Verwandte Funktionen

### Bildbetrachtungswerkzeuge

Beim Betrachten einer Ebene können Sie Folgendes verwenden:

* **Zoomsteuerung**: Vergrößern Sie das Bild, um Details zu untersuchen.
* **Verschieben**: Klicken und ziehen Sie, um sich im gezoomten Bild zu bewegen.
* **Pixelwertprüfung**: Zeigen Sie die Werte an der Cursorposition an.
* **Navigationspfeile**: Wechseln Sie zwischen Bildern, während die Ebene beibehalten wird.
* **Pixelprozentmodus**: Wechseln Sie zwischen DN- und Prozentanzeige.

Die vollständige Dokumentation zum Bildbetrachter finden Sie unter [Bild im Vollbildmodus öffnen](opening-an-image-full-screen.md).

### Index/LUT-Sandbox

Für interaktive Indextests und Visualisierungen:

* **Echtzeit-Indexberechnung**: Testen Sie verschiedene Indexformeln
* **LUT-Farbzuordnung**: Wenden Sie Farbverläufe auf Graustufenindizes an
* **Visualisierungen exportieren**: Speichern Sie farbige Indexbilder

Weitere Informationen finden Sie unter [Index/LUT-Sandbox](index-lut-sandbox.md).

***

## Nächste Schritte

Nachdem Sie nun die Bildlayer verstanden haben:

* [**Öffnen eines Bildes im Vollbildmodus**](opening-an-image-full-screen.md) – Vollständige Anleitung zum Bildbetrachter
* [**Index/LUT-Sandbox**](index-lut-sandbox.md) – Interaktive Indexvisualisierung
* [**Multispektrale Indexformeln**](../project-settings/multispectral-index-formulas.md) – Referenz zu verfügbaren Indizes
* [**Beenden der Verarbeitung**](../processing-images-gui/finishing-the-processing.md) – Verstehen der verarbeiteten Ergebnisse
