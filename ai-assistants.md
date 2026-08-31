# Verwendung von Chloros mit KI-Assistenten

Dieses Handbuch richtet sich an zwei Zielgruppen: Menschen und die KI-Assistenten, mit denen Menschen zunehmend arbeiten. Auf jeder Seite werden genaue Werte, Standardwerte und per Kopieren-Einfügen verwendbare Befehle angegeben, damit ein Assistent (Claude, ChatGPT, Copilot, ein Programmierassistent usw.) bereits beim ersten Versuch funktionierende Chloros-Automatisierungen erstellen kann.

Chloros-Version: **

1.2.0**. CLI-/SDK-Plattformen: Windows 10/11 x64 und Linux (x86_64 / Jetson aarch64).

## Was Sie Ihrem Assistenten übergeben sollten

| Ressource | URL | Verwendungszweck |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Maschinenlesbarer Index aller Seiten in diesem Handbuch. |
| **CLI-Referenz** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Die vollständige Befehlsschnittstelle von `chloros-cli`: alle Befehle, Flags, Standardwerte, Exit-Codes und Regeln für den Ausgabeordner. Verfasst für die Verwendung durch LLM. |
| **SDK-Referenz** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Das vollständige `chloros_sdk` Python API: Klassen, Signaturen, Ausnahmen und ausgearbeitete Beispiele. Verfasst für LLM-Nutzung. |
| **Jede Seite als reines Markdown** | Füge `.md` an die Seite URL an | z. B. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` gibt die Seite als reines Markdown zurück – ideal zum Einfügen in ein Kontextfenster oder zum Abrufen über einen Agenten. |

Links im Handbuch: [CLI Referenz](reference/cli-reference.md) · [SDK-Referenz](reference/sdk-reference.md).

{% hint style="info" %}
Die beiden Referenzseiten sind in sich geschlossen: Ein Assistent, der eine davon gelesen hat, benötigt den Rest des Handbuchs nicht, um ein korrektes Skript zu schreiben.
{% endhint %}

## Fertige Skripte

Kopieren Sie das Skript `<placeholders>`, füllen Sie es aus und fügen Sie es in Ihren Assistenten ein.

### 1. Einen Flugordner in NDVI verarbeiten

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Ein „captures“-Verzeichnis im Batch-Modus überwachen

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Ein LATTICE-Array anschließen und Daten erfassen

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. DAQ-Lichtsensorspektren aufzeichnen

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
DAQ-Skripte über die Befehlszeile laufen immer über die `daq pool-*`-Familie (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Andere `daq`-Unterbefehle, die Ihr Assistent möglicherweise erfindet, sind in den ausgelieferten Builds nicht verfügbar und führen zu einem Fehler.
{% endhint %}

## Warum von KI geschriebene Skripte gut mit Chloros funktionieren

Jedes dieser Beispiele ist ein echtes, verifiziertes Verhalten von Chloros 1.2.0 – sie beseitigen die klassischen Fehlerquellen maschinell erstellter Automatisierungen:

* **Kein umständliches Einrichtungsgefummel.**Die „Smart-Connect“-Helfer von SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) und die Verarbeitungs-Einstiegspunkte (`ChlorosLocal`, `process_folder`)**starten das lokale Backend automatisch**. Ein generiertes Skript benötigt weder eine geöffnete GUI noch einen manuell gestarteten Server – es muss lediglich das Paket „desktop/CLI“ installiert sein.
* **Die gesamte Pipeline erfolgt in einem einzigen Aufruf.** `chloros_sdk.process_folder("path", indices=["NDVI"])` führt Import → Kalibrierung → Reflexionsmessung → Indexexport durchgängig durch. Weniger Prozessschritte, weniger Stellen, an denen ein generiertes Skript fehlschlagen kann.
* **Läufe ohne Ausgabe führen eine Selbstdiagnose durch.** Nach `process()` wird die Zusammenfassung des Laufs dem Ergebnis angehängt, und jeder Verarbeitungshinweis (z. B. *warum* ein Lauf keine Ausgabe erzeugt hat) ebenfalls als Python `UserWarning` ausgegeben – sodass selbst ein Skript, das das Ergebnis-Dict nie überprüft, die Diagnose anzeigt.
* **Der Lauf CLI schlägt mit einer deutlichen Fehlermeldung fehl.**Ein `chloros-cli process`-Lauf, der Ergebnisse angefordert, aber keine geschrieben hat, gibt `Processing finished but wrote no image products.` aus und**beendet den Lauf mit einem Wert ungleich Null**, sodass Shell-Skripte und CI dies durch eine einfache Überprüfung des Exit-Codes erkennen können. Erfolgreiche Läufe melden „`Image products written: N`“.

Eine Besonderheit, die ein Assistent kennen sollte: Das `process()` des SDK löst bei einem Lauf ohne Produkte bewusst **keinen** Fehler aus – stattdessen erfolgt die Meldung über die Zusammenfassung/Hinweise. Wenn eine Python-Pipeline bei einem leeren Lauf gestoppt werden muss, überprüfen Sie die Zusammenfassung (Rezept 2 tut dies).

## Hinweise

* **Chloros+ Anmeldung erforderlich.**Die CLI und SDK erfordern eine**kostenpflichtige** Chloros+-Tarif, der serverseitig durchgesetzt wird: Anfragen schlagen mit dem Fehler `401 AUTH_REQUIRED` fehl, wenn keine Anmeldung vorliegt, und mit dem Fehler `403 PLAN_UPGRADE_REQUIRED` im kostenlosen Tarif. Führen Sie `chloros-cli login` einmal pro Rechner aus, bevor Sie die generierten Skripte ausführen. Siehe [Chloros+ Anmeldung](chloros+-login.md).
* **Die Capture-Befehle steuern echte Hardware.** Die Befehle „`lattice`“, „`daq`“ und „`project`“ sowie die Sitzungsobjekte „SDK“ stellen eine Verbindung zu physischen Kameras und Sensoren her, übertragen Daten von diesen und lösen Aktionen aus. Überprüfen Sie ein generiertes Skript vor der ersten Ausführung und führen Sie es unter Aufsicht der Hardware aus.
* **Überprüfen Sie die Ausgaben stichprobenartig.** Überprüfen Sie die Produktordner und einige Pixelwerte, bevor Sie die Ergebnisse veröffentlichen. Insbesondere werden Reflektanz-TIFFs pro Quelle skaliert – lesen Sie das `Chloros:PixelScale`-XMP-Tag (LATTICE: 32768 = Reflektanz 1,0; Survey3: 65535), anstatt einen Divisor anzunehmen. Beide Referenzseiten dokumentieren dies unter „Reflektanzpixel lesen“.
* **Kleine Fallstricke, die den generierten Code zum Straucheln bringen:**`pool-record` schreibt in das Dateisystem des**Backend-Hosts** (Standard: `~/Documents/DAQ Live View/`); auf Rechnern mit mehreren Netzwerkschnittstellen sollte `daq pool-connect --eth-host <ip-or-hostname>` der automatischen Erkennung vorgezogen werden; und verwenden Sie `http://127.0.0.1:5000` (niemals `localhost`) überall dort, wo ein Backend-URL auftaucht.
