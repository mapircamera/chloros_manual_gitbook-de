# Einsatz von Chloros mit KI-Assistenten

Dieses Handbuch richtet sich an zwei Zielgruppen: Menschen und die KI-Assistenten, mit denen Menschen zunehmend arbeiten. Auf jeder Seite werden genaue Werte, Standardwerte und kopier- und einfügbare Befehle angegeben, damit ein Assistent (Claude, ChatGPT, Copilot, ein Programmierassistent usw.) auf Anhieb funktionierende Chloros-Automatisierungen erstellen kann.

Chloros-Version: **

1.3.0**. CLI-/SDK-Plattformen: Windows 10/11 x64 und Linux (x86\_64 / Jetson aarch64).

## Was Sie Ihrem Assistenten zur Verfügung stellen sollten

| Ressource | URL | Wozu dient sie? |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Maschinenlesbarer Index aller Seiten in diesem Handbuch. |
| **CLI-Referenz** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Die vollständige `chloros-cli`-Befehlsschnittstelle: alle Befehle, Flags, Standardwerte, Exit-Codes und Regeln für den Ausgabeordner. Für die Verwendung durch LLMs verfasst. |
| **SDK-Referenz** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Die vollständige `chloros_sdk` Python-API: Klassen, Signaturen, Ausnahmen und Anwendungsbeispiele. Geschrieben für die Verwendung durch LLM. |
| **Jede Seite als reines Markdown** | Fügen Sie „`.md`“ an die Seiten-URL an | z. B. gibt „`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`“ die Seite als reines Markdown zurück – ideal zum Einfügen in ein Kontextfenster oder zum Abrufen über einen Agenten. |

Links im Handbuch: [CLI-Referenz](reference/cli-reference.md) · [SDK-Referenz](reference/sdk-reference.md).

{% hint style="info" %}
Die beiden Referenzseiten sind in sich geschlossen: Ein Assistent, der eine davon gelesen hat, benötigt den Rest des Handbuchs nicht, um ein korrektes Skript zu schreiben.
{% endhint %}

## Lassen Sie Ihren Assistenten Chloros direkt ausführen: `chloros-mcp`

Bei allen oben genannten Beispielen *schreibt* Ihr Assistent ein Skript oder einen Befehl. Mit `chloros-mcp` steuert ein Assistent, der das [Model Context Protocol](https://modelcontextprotocol.io) (MCP) beherrscht – wie beispielsweise Claude Desktop, Claude Code oder Cursor – Chloros selbst. Sie geben eine Anfrage in natürlicher Sprache ein, und der Assistent ruft die Tools des Servers auf:

* **Bilder verarbeiten:** Erstellen Sie ein Projekt, importieren Sie einen Ordner mit Aufnahmen, wählen Sie Einstellungen aus, verarbeiten Sie die Bilder und lassen Sie sich die Ergebnisse des Durchlaufs anzeigen.
* **Hardware betreiben:** LATTICE-Kameras, Arrays und DAQ-Lichtsensoren anschließen, eine Vorschau anzeigen, Belichtung und Farbe einstellen, Aufnahmen machen, aufzeichnen und die Firmware aktualisieren.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Die Tools des Servers decken die Funktionen des [Python-SDK](api-python-sdk.md) ab. Sie werden zusammen mit dem SDK installiert, sodass in der Regel keine weiteren Installationen erforderlich sind: Unter Windows legt das Chloros-Installationsprogramm `chloros-mcp.exe` im `Scripts`-Ordner Ihres Python ab, und das Linux-`.deb` legt `chloros-mcp` in `/usr/bin` ab.

**Claude-Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**Fügen Sie dies zu „`claude_desktop_config.json`“ hinzu (über**Einstellungen > Entwickler > Konfiguration bearbeiten**können Sie die Datei öffnen) und starten Sie anschließend Claude Desktop neu.**Cursor** benötigt denselben Eintrag in „`~/.cursor/mcp.json`“.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Wenn der Client meldet, dass der Server nicht gestartet werden konnte, ist `chloros-mcp` wahrscheinlich nicht in der Liste `PATH` enthalten, die er sieht: Geben Sie stattdessen den vollständigen Pfad an (`where chloros-mcp` unter Windows, `which chloros-mcp` unter Linux).

Chloros muss auf demselben Computer installiert sein und Sie müssen dort angemeldet sein – entweder in der App oder über `chloros-cli login` – und zwar in einem kostenpflichtigen Chloros+-Tarif (Copper oder höher). Der Server kann sich nicht für Sie anmelden.

{% hint style="info" %}
Client-Einrichtung für jede App, Optionen, jedes Tool und das Verhalten eines Laufs: [MCP-Server (KI-Assistenten)](reference/sdk-reference.md#mcp-server-ai-assistants) in der SDK-Referenz.
{% endhint %}

## Prompt-Rezepte

Kopieren Sie den Text, füllen Sie die Lücke `<placeholders>` aus und fügen Sie ihn in Ihren Assistenten ein.

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
Die Skripterstellung für die Datenerfassung über die Befehlszeile erfolgt immer über die `daq pool-*`-Familie (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Andere `daq`-Unterbefehle, die Ihr Assistent möglicherweise erfindet, sind in den ausgelieferten Builds nicht verfügbar und führen zu einem Fehler.
{% endhint %}

## Warum von KI geschriebene Skripte gut mit Chloros funktionieren

Jedes dieser Beispiele ist ein echtes, verifiziertes Verhalten von Chloros 1.3.0 – sie beseitigen die klassischen Fehlerquellen maschinell geschriebener Automatisierungen:

* **Keine umständlichen Einrichtungsschritte.**Die Smart-Connect-Helfer des SDKs (`connect_camera`, `connect_array`, `connect_daq_sensor`) und die Verarbeitungs-Einstiegspunkte (`ChlorosLocal`, `process_folder`)**starten das lokale Backend automatisch**. Ein generiertes Skript benötigt weder eine geöffnete GUI noch einen manuell gestarteten Server – es muss lediglich das Desktop-/CLI-Paket installiert sein.
* **Die gesamte Pipeline erfolgt in einem einzigen Aufruf.** `chloros_sdk.process_folder("path", indices=["NDVI"])` führt Import → Kalibrierung → Reflexionsmessung → Index-Export durchgängig durch. Geringere Komplexität, weniger Stellen, an denen ein generiertes Skript fehlschlagen kann.
* **Läufe ohne Ausgabe führen eine Selbstdiagnose durch.** Nach `process()` wird die Zusammenfassung des Laufs an das Ergebnis angehängt, und jeder Verarbeitungshinweis (z. B. *warum* ein Lauf keine Ausgabe erzeugt hat) wird ebenfalls als Python `UserWarning` ausgegeben – sodass selbst ein Skript, das das Ergebnis-Dict nie überprüft, die Diagnose anzeigt.
* **Die CLI-Anwendung meldet den Fehler deutlich.**Ein `chloros-cli process`-Lauf, der Produkte angefordert, aber keine geschrieben hat, gibt `Processing finished but wrote no image products.` aus und**beendet den Lauf mit einem Wert ungleich Null**, sodass Shell-Skripte und CI dies durch eine einfache Überprüfung des Exit-Codes erkennen können. Erfolgreiche Ausführungen melden „`Image products written: N`“ sowie „`(plus M clamp masks)`“, wenn Masken geschrieben wurden. Capture-Befehle nennen ebenfalls jede Datei, die sie nicht schreiben konnten, und beenden die Ausführung mit „`1`“.
* **Relative Pfade entsprechen den Erwartungen des Skripts.** Bei einem lokalen Backend wird ein relativer Ausgabepfad (`-o out/` in der CLI, `output_dir="out/"` im SDK) relativ zu dem Ordner aufgelöst, aus dem der Befehl oder das Skript ausgeführt wird.

Eine Besonderheit, die ein Assistent kennen sollte: Das `process()` des SDK löst bei einem Lauf ohne Ergebnis bewusst **keinen** Fehler aus – stattdessen erfolgt die Meldung über die Zusammenfassung/Hinweise. Wenn eine Python-Pipeline bei einem leeren Lauf angehalten werden muss, überprüfen Sie die Zusammenfassung (Rezept 2 tut dies).

## Hinweise

* **Chloros+-Anmeldung erforderlich.**Die CLI und das SDK erfordern eine**kostenpflichtige** Chloros+-Stufe, die serverseitig durchgesetzt wird: Anfragen schlagen mit dem Fehler `401 AUTH_REQUIRED` fehl, wenn keine Anmeldung vorliegt, und mit dem Fehler `403 PLAN_UPGRADE_REQUIRED` bei der kostenlosen Stufe. Führen Sie `chloros-cli login` einmal pro Rechner aus, bevor Sie die generierten Skripte ausführen. Siehe [Chloros+-Anmeldung](chloros+-login.md).
* **Erfassungsbefehle steuern reale Hardware.** `lattice` / `daq` / `project`-Befehle, die SDK-Sitzungsobjekte und die Tools von `chloros-mcp` stellen eine Verbindung zu physischen Kameras und Sensoren her, übertragen Daten von diesen und lösen Aktionen aus. Überprüfen Sie ein generiertes Skript vor der ersten Ausführung und führen Sie es unter Aufsicht der Hardware aus.
* **Überprüfen Sie die Ausgaben stichprobenartig.** Überprüfen Sie die Produktordner und einige Pixelwerte, bevor Sie die Ergebnisse veröffentlichen. Insbesondere werden Reflektanz-TIFFs pro Quelle skaliert – lesen Sie das `Chloros:PixelScale`-XMP-Tag (LATTICE-Dateien enthalten es: 32768 = 1,0 Reflektanz; Survey3-Dateien enthalten ihn nicht und verwenden 65535 = 1,0) anstelle der Annahme eines Divisors. Beide Referenzseiten dokumentieren dies unter „Lesen von Reflexionspixeln“.
* **Kleine Fallstricke, die den generierten Code zum Straucheln bringen:**`pool-record` schreibt in das Dateisystem des**Backend-Hosts** (Standard: `~/Documents/DAQ Live View/`); auf Rechnern mit mehreren Netzwerkschnittstellen sollte `daq pool-connect --eth-host <ip-or-hostname>` der automatischen Erkennung vorgezogen werden; verwenden Sie `http://127.0.0.1:5000` (niemals `localhost`) überall dort, wo eine Backend-URL erscheint; Ein Skript, das den Ausgabebaum eines Projekts durchläuft, sollte `clamp_masks/`-Ordner überspringen (die darin enthaltenen `*_clampmask.tif`-Dateien sind Masken, keine Bilder – siehe [Ausgabe-Bildformate](output-image-formats.md#clamp-masks)); und unter Windows, bei dem lange Pfade deaktiviert sind, sollten Projektordner kurz gehalten werden – ein Projektordner, der tiefer als etwa 130 Zeichen liegt, wird vor Beginn der Verarbeitung abgelehnt.
