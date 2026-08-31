# CLI Schnellstart (pool-*)

Die ausgelieferten `chloros-cli`-Laufwerke steuern DAQ-Sensoren über die **`daq pool-*`** Befehlsfamilie – schlanke HTTP-Clients, die den Sensor über den persistenten Sensor-Pool des Chloros-Backends betreiben. Da das Backend für den Transport zuständig ist, teilen sich die GUI, das Skript „CLI“ und das Skript „SDK“ ein gemeinsames Live-Handle, anstatt um den Port zu konkurrieren. Alles, was ein Kunde benötigt, ist über `pool-*` erreichbar: Verbinden, Streamen, Aufzeichnen kalibrierter `.daq`-Dateien und Austauschen von Kap-Profilen.

`pool-*` ist zudem die **einzige** DAQ-Oberfläche in veröffentlichten Builds. `chloros-cli daq --help` listet die `pool-*`-Unterbefehle auf, und der Aufruf eines Direkt-Hardware-DAQ-Unterbefehls in einer veröffentlichten Version führt zu einem expliziten Fehler, der das fehlende Paket nennt und Sie auf `pool-*` verweist – nichts schlägt stillschweigend fehl. (Die Befehle für die direkte Hardware-Ansteuerung laufen nur aus einem MAPIR-Quellcode-Checkout; `pip install chloros-sdk` stellt sie ebenfalls nicht bereit.)

***

## Voraussetzungen

* **Das Chloros-Backend muss laufen** – die `pool-*`-Befehle sind HTTP-Clients, keine Hardwaretreiber. Starten Sie auf Windows die Desktop-App Chloros (diese startet das Backend). Auf einem headless Linux/Jetson den Dienst aktivieren: `sudo systemctl enable --now chloros-backend.service`.
* **Anmeldung bei Chloros+ (kostenpflichtige Stufe)**: Führen Sie zuerst `chloros-cli login` aus. Die Durchsetzung erfolgt serverseitig – ohne Anmeldung schlagen Befehle bei `401 AUTH_REQUIRED` fehl; in der kostenlosen (Iron-)Stufe schlagen sie bei `403 PLAN_UPGRADE_REQUIRED` fehl.
* Die Befehle zielen standardmäßig auf `http://127.0.0.1:5000` ab; die `daq pool-*`-Familie berücksichtigt die Umgebungsvariable `CHLOROS_BACKEND_URL`, falls Ihr Backend an einem anderen Ort ausgeführt wird.

***

## Eine fünfminütige Sitzung

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` – einen Sensor im Pool öffnen

| Variante | Bedeutung |
| --- | --- |
| `daq pool-connect` | Smart-Detect: Alle DAQs auf diesem Rechner suchen. |
| `daq pool-connect --port PORT` | DAQ-U an einem bestimmten seriellen Port (z. B. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M über BLE, MAC automatisch gescannt. |
| `daq pool-connect --mac MAC` | DAQ-M an einer bekannten BLE-MAC-Adresse (impliziert `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E mit bekanntem Hostnamen oder bekannter IP-Adresse – **der zuverlässige Weg**. |
| `daq pool-connect --eth` | DAQ-E mit automatischer Erkennung (mDNS, mit ARP-Fallback). Siehe den Hinweis unten. |

Optimierungsflags, alle optional:

| Flag | Bedeutung |
| --- | --- |
| `--integration-time MS` / `-t MS` | Manuelle Integrationszeit in Millisekunden. |
| `--frame-avg N` / `-f N` | Durchschnittliche Anzahl der Frames pro gemeldetem Spektrum. |
| `--no-ae` | Automatische Belichtung deaktivieren (AE ist standardmäßig aktiviert). |
| `--no-stream` | Verbinden, ohne den Stream zu starten (später mit `pool-stream --start` fortsetzen). |
| `--cap-id CAP` | Cap-Korrekturprofil; Backend-Standard ist `sunshine_cosine`. Siehe [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**`--eth` – Hinweis zur automatischen Erkennung.** Auf einem Host mit mehreren Netzwerkanschlüssen (mehr als eine aktive Netzwerkschnittstelle) kann der *erste* Aufruf von `pool-connect --eth` nach dem Systemstart leer ausgehen, selbst wenn der Sensor funktionsfähig ist – der Erkennungsdurchlauf kann die Schnittstelle des Sensors übersehen, solange der ARP-Cache noch nicht gefüllt ist. Wenn `--eth` nichts findet, wiederholen Sie den Vorgang oder überspringen Sie die Erkennung komplett mit `--eth-host <ip-or-hostname>`, was auf Hosts mit mehreren Netzwerkanschlüssen der zuverlässigere Weg ist. Der Hostname des DAQ-E lautet `daq-e-<id>.local` (z. B. `daq-e-def330.local`); die reine IP-Adresse funktioniert ebenfalls.
{% endhint %}

## `pool-list` – anzeigen, was verbunden ist

Zeigt jeden Sensor im Backend-Pool an, einschließlich des Befehls `sensor_id`, den alle anderen Befehle benötigen:

| Modell | `sensor_id`-Format | Beispiel |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 Oktette mit Bindestrichen | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` – Spektrum-Frames lesen

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Gibt den aktuellsten Frame oder die aktuellsten `--recent N`-Frames zurück; `--json` gibt maschinenlesbare Ausgabe für Skripte aus. Die Frames sind radiometrisch kalibrierte Spektralstrahlungsintensitäten (W/m²/nm) auf einem 135-Punkte-Raster von 340–1010 nm, wobei das Sensor-Cap-Profil bereits angewendet wurde. Für quantitative Bestrahlungsstärkewerte sollten Frames über mindestens 15 Sekunden gemittelt werden – dies ist eine Geräteeigenschaft und kein Fehler.

## `pool-stream` – Streaming anhalten oder fortsetzen

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` – `.daq`-Datei aufzeichnen

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Flag | Standard | Bedeutung |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Aufnahmedauer in Sekunden; `0` bedeutet: so lange ausführen, bis Sie `--stop` eingeben. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Ausgabeverzeichnis, das **auf dem Rechner, auf dem das Backend läuft**, aufgelöst wird. |
| `--device-name NAME` | — | Mit der Aufzeichnung gespeicherte Bezeichnung. |
| `--stop` | — | Eine laufende Aufzeichnung stoppen. |

{% hint style="info" %}
Die Aufzeichnung erfolgt im Backend, sodass die Datei `.daq` im Dateisystem des **Backend-Rechners** abgelegt wird – standardmäßig dort in `~/Documents/DAQ Live View/`, nicht unbedingt dort, wo Sie das CLI ausgeführt haben. Die Dateinamen enthalten die Sensor-ID und einen Zeitstempel.
{% endhint %}

## `pool-set-cap` – Angabe der montierten Schutzkappe

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Die Kappen-ID wählt das werkseitig gemessene Korrekturprofil aus, das auf jedes Spektrum angewendet wird, und sie **muss mit der physisch am Sensor angebrachten Kappe übereinstimmen** – weder der Sensor noch die Software können die Kappe selbstständig erkennen, und die Auswahl wird in jede `.daq`-Datei eingetragen. Die Standard-Kappe ist überall `sunshine_cosine` (jedes DAQ wird mit der installierten „Sunshine“-Kosinus-Korrekturkappe ausgeliefert, die konstruktionsbedingt eine Dämpfung von ca. 12× aufweist – ein nicht deklarierter Kappewechsel führt zu einer um ungefähr diesen Faktor verzerrten Spektralkorrektur).

| `--cap-id` | Verfügbar für |
| --- | --- |
| `sunshine_cosine` (Standard) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Nur DAQ-U |
| `none` | Nur DAQ-E – siehe Hinweis |

Eine Kappe mit einer ID außerhalb des für den Sensor festgelegten Bereichs wird beim Anschließen mit einem eindeutigen Fehler abgelehnt. `none` (DAQ-E) bedeutet, dass die Kappe physisch entfernt wurde – es wird weiterhin ein werkseitiges Geometrieprofil für den versenkten Glasdiffusor des DAQ-E angewendet, es handelt sich also nicht um eine No-Op-Funktion, und ein „bloßer“ DAQ-E ist eine Laborkonfiguration, keine unterstützte Feldkonfiguration. (Ein ungeschützter DAQ-U ist wirklich ungeschützt und benötigt überhaupt kein Korrekturprofil; der DAQ-M wird mit seiner Sunshine-Kappe verwendet.)

## `pool-disconnect` – Sensoren freigeben

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Befehlsübersicht

| Befehl | Zweck |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Einen Sensor im Backend-Pool öffnen. |
| `daq pool-list` | Alle Sensoren im Pool mit ihrer `sensor_id`-Nummer anzeigen. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Die letzten N kalibrierten Spektralrahmen. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Streaming fortsetzen / anhalten. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Eine `.daq`-Aufzeichnung starten / stoppen (Backend-seitig). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Das Cap-Korrekturprofil zur Laufzeit wechseln. |
| `daq pool-disconnect --sensor-id ID [--all]` | Einen Sensor oder alle Sensoren freigeben. |

***

## Fehlerbehebung bei der ersten DAQ-E-Verbindung

1. Das DAQ-E verfügt über keine Status-LED – überprüfen Sie die Stromversorgung über die PoE-/Link-Anzeige am Switch oder am Injektor-Port und warten Sie nach dem Einschalten einige Sekunden, bis das Gerät hochgefahren ist und sich mit dem Netzwerk verbunden hat.
2. Der Backend-Rechner muss sich in derselben **Broadcast-Domäne** wie der Sensor befinden – mDNS durchquert keine Router.
3. Akzeptieren Sie bei Windows beim ersten Start die Abfrage der Defender-Firewall (mDNS UDP 5353, DAQ-E-Daten UDP 5002, PTP UDP 319/320).
4. Immer noch keine Reaktion von `--eth`? Verwenden Sie `--eth-host` mit dem Hostnamen des Geräts (`daq-e-<id>.local`) oder dessen IP-Adresse – dies ist der zuverlässigste Weg, insbesondere bei Hosts mit mehreren Netzwerkanschlüssen.

***{% hint style="info" %}**Tipp für KI-Assistenten.** Jede Seite dieses Handbuchs wird als reines Markdown bereitgestellt – fügen Sie `.md` an den kleingeschriebenen Slug einer Seite (URL) an (diese Seite: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); der maschinenlesbare Index lautet dann `https://mapir.gitbook.io/chloros/llms.txt`. Für eine vollständige Dokumentation auf Flag-Ebene zu `chloros-cli daq` und jeder anderen Befehlsfamilie rufe die [CLI-Referenz](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); der Pfad zu Python lautet `chloros_sdk.connect_daq_sensor()` in der [SDK-Referenz](../reference/sdk-reference.md).
{% endhint %}
