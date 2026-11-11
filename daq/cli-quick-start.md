# CLI-Schnellstart (pool-*)

Die mitgelieferten `chloros-cli`-Treiber steuern DAQ-Sensoren über die Befehlsfamilie **`daq pool-*`** – einfache HTTP-Clients, die den Sensor über den persistenten Sensorpool des Chloros-Backends bedienen. Da das Backend für den Transport zuständig ist, nutzen die GUI, die CLI und die SDK-Skripte alle denselben Live-Handle, anstatt sich um den Port zu streiten. Alles, was ein Kunde benötigt, ist über `pool-*` erreichbar: Verbinden, Streamen, Aufzeichnen kalibrierter `.daq`-Dateien und Austauschen von Cap-Profilen.

`pool-*` ist die in veröffentlichten Builds unterstützte DAQ-Oberfläche. `chloros-cli daq --help` listet die `pool-*`-Unterbefehle auf, und `daq discover` / `daq list` führen denselben Scan aus wie `pool-discover`. Ist ein DAQ-Unterbefehl für den direkten Hardwarezugriff nicht verfügbar, wird der Vorgang mit einer expliziten Fehlermeldung beendet, die das fehlende Paket nennt und auf `pool-*` verweist – es gibt keine stillen Fehler. (Auch `pip install chloros-sdk` stellt keine Befehle für die direkte Hardware-Ansteuerung bereit.)

***

## Voraussetzungen

* **Das Chloros-Backend muss laufen** – die Befehle `pool-*` sind HTTP-Clients, keine Hardwaretreiber. Starten Sie unter Windows die Chloros-Desktop-App (sie startet das Backend). Auf einem headless Linux/Jetson-System aktivieren Sie den Dienst als Ihr Benutzer: `sudo systemctl enable --now chloros-backend@$USER.service` (siehe [Linux-Installation](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Anmeldung bei Chloros+ (kostenpflichtige Stufe)**: Führen Sie zunächst `chloros-cli login` aus. Die Durchsetzung erfolgt serverseitig – ohne Anmeldung schlagen Befehle bei `401 AUTH_REQUIRED` fehl; in der kostenlosen (Iron-)Stufe schlagen sie bei `403 PLAN_UPGRADE_REQUIRED` fehl.
* Die Befehle zielen standardmäßig auf `http://127.0.0.1:5000` ab; die `daq pool-*`-Familie berücksichtigt die Umgebungsvariable `CHLOROS_BACKEND_URL`, falls Ihr Backend an einem anderen Ort ausgeführt wird.

***

## Eine fünfminütige Sitzung

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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

## `pool-connect` – Einen Sensor im Pool öffnen

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

Das Verbinden mit einem DAQ-E, das sich bereits unter einer anderen seiner Adressen – seinem Hostnamen `daq-e-<id>.local` oder seiner IP-Adresse – im Pool befindet, gibt den bestehenden Pool-Eintrag zurück, anstatt das Gerät ein zweites Mal zu öffnen.

{% hint style="warning" %}
**`--eth` – Hinweis zur automatischen Erkennung.** Auf einem Host mit mehreren Netzwerkanschlüssen (mehr als eine aktive Netzwerkschnittstelle) kann der *erste* `pool-connect --eth`-Aufruf nach dem Systemstart leer ausgehen, selbst wenn der Sensor funktionsfähig ist – die Erkennungsabfrage kann die Schnittstelle des Sensors übersehen, solange der ARP-Cache noch nicht gefüllt ist. Wenn `--eth` nichts findet, wiederholen Sie den Vorgang oder überspringen Sie die Erkennung vollständig mit `--eth-host <ip-or-hostname>` – dies ist der zuverlässige Weg auf Hosts mit mehreren Netzwerkanschlüssen. Der Hostname des DAQ-E lautet `daq-e-<id>.local` (z. B. `daq-e-def330.local`); seine reine IP-Adresse funktioniert ebenfalls.
{% endhint %}

## `pool-discover` – Scannen ohne Verbindung

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Sucht nach Sensoren, mit denen eine Verbindung hergestellt werden könnte, und zeigt die Adresse für die Verbindung an, ohne eine Verbindung herzustellen. `--only` akzeptiert eine durch Kommas getrennte Teilmenge von `usb,ble,eth`; `--timeout SEC` legt die Scan-Zeit pro Transport fest (Standardwert 5 s, begrenzt auf 1–20 s). Sensoren, die bereits im Pool geöffnet sind, werden nicht aufgelistet (ein verbundenes DAQ-M sendet keine Signale mehr) – verwenden Sie hierfür `pool-list`. In einer ausgelieferten Version führen `chloros-cli daq discover` und `daq list` denselben Scan durch.

## `pool-list` – Anzeigen der verbundenen Geräte

Zeigt jeden Sensor im Backend-Pool an, einschließlich des Befehls `sensor_id`, den alle anderen Befehle benötigen:

| Modell | `sensor_id`-Form | Beispiel |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 Oktette mit Bindestrichen | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Ein DAQ-E, dessen PTP deaktiviert wurde (die Einstellung wird auf dem Gerät gespeichert), wird in `pool-list` mit `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` gekennzeichnet, zusammen mit dem Befehl, der es wieder aktiviert – siehe [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` – Spektrum-Frames lesen

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Gibt den aktuellsten Frame oder die aktuellsten `--recent N`-Frames zurück; `--json` gibt maschinenlesbare Ausgabe für Skripte aus. Die Frames sind radiometrisch kalibrierte spektrale Bestrahlungsstärken (W/m²/nm) auf einem 135-Punkte-Raster von 340–1010 nm, wobei das Kap-Profil des Sensors bereits angewendet wurde. Für quantitative Bestrahlungsstärkewerte sollten Frames über mindestens 15 Sekunden gemittelt werden – dies ist eine Eigenschaft des Geräts und kein Fehler.

## `pool-stream` – Streaming anhalten oder fortsetzen

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` – eine `.daq`-Datei aufzeichnen

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Flag | Standard | Bedeutung |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Aufnahmedauer in Sekunden; `0` bedeutet: so lange ausführen, bis Sie `--stop` eingeben. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Ausgabeverzeichnis. Bei einem Backend auf diesem Computer wird ein relativer Pfad relativ zu dem Ordner aufgelöst, in dem Sie den Befehl ausgeführt haben; bei einem Remote-Backend wird der Pfad **auf dem Rechner, auf dem das Backend läuft** aufgelöst. |
| `--device-name NAME` | — | Mit der Aufzeichnung gespeicherte Bezeichnung. |
| `--stop` | — | Eine laufende Aufzeichnung beenden. |
| `--confirm-after SEC` | `20` | Nach dem Start wird bis zu dieser Zeit darauf gewartet, dass die erste Zeile geschrieben wird, und es wird laut gewarnt (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`), wenn keine eintrifft. `0` deaktiviert die Überprüfung. |
| `--heartbeat SEC` | `30` | Mit `--duration` wird die Zeilenanzahl alle SEC Sekunden ausgegeben und eine Warnung ausgegeben, wenn sie nicht weiter ansteigt. `0` deaktiviert diese Funktion. |

{% hint style="info" %}
Die Aufzeichnung erfolgt im Backend, daher landet die Datei „`.daq`“ im Dateisystem des **Backend-Rechners** – standardmäßig in „`~/Documents/DAQ Live View/`“ dort, nicht unbedingt dort, wo Sie die CLI ausgeführt haben. Die Dateinamen enthalten die Sensor-ID und einen Zeitstempel.
{% endhint %}

## `pool-set-cap` – Angabe der montierten Schutzkappe

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Die Kappen-ID wählt das werkseitig gemessene Korrekturprofil aus, das auf jedes Spektrum angewendet wird, und sie **muss mit der physisch am Sensor angebrachten Kappe übereinstimmen** – weder der Sensor noch die Software können die Kappe selbstständig erkennen, und die Auswahl wird in jede `.daq`-Datei eingetragen. Die Standard-ID ist überall `sunshine_cosine` (jedes DAQ-Gerät wird mit der installierten „Sunshine“-Kosinus-Korrekturkappe ausgeliefert, die konstruktionsbedingt eine Dämpfung von ca. 12× aufweist – ein nicht deklarierter Kappenwechsel führt zu einer um etwa diesen Faktor verzerrten Korrektur der Spektren).

| `--cap-id` | Verfügbar für |
| --- | --- |
| `sunshine_cosine` (Standard) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Nur DAQ-U |
| `none` | Nur DAQ-E – siehe Hinweis |

Eine Kappe mit einer ID außerhalb des für den Sensor festgelegten Bereichs wird beim Anschließen mit einem eindeutigen Fehler verworfen. `none` (DAQ-E) bedeutet, dass die Kappe physisch entfernt wurde – es wird weiterhin ein werkseitiges Geometrieprofil für den versenkten Glasdiffusor des DAQ-E angewendet, es handelt sich also nicht um eine No-Op-Funktion, und ein „nackter“ DAQ-E ist eine Laborkonfiguration, keine unterstützte Feldkonfiguration. (Ein ungeschützter DAQ-U ist vollständig ungeschützt und benötigt überhaupt kein Korrekturprofil; der DAQ-M wird mit seiner Sunshine-Kappe verwendet.)

## `pool-disconnect` – Sensoren freigeben

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` gibt einen Sensor frei, unabhängig davon, wer ihn geöffnet hat; somit wird auch ein Sensor freigegeben, den ein abgestürztes Skript im Pool zurückgelassen hat (eine darauf laufende Aufzeichnung wird zuvor beendet). Ein Sensor, der sich nicht im Pool befindet, kann nicht freigegeben werden: Der Befehl gibt „`<sensor-id>: already released (not in the pool)`“ aus und beendet sich mit „`0`“. Eine Freigabe, die vom Backend abgelehnt oder nicht bestätigt wurde, beendet sich mit „`1`“.

## `pool-enable-ptp` – PTP eines DAQ-E wieder einschalten

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Ein DAQ-E speichert seine PTP-Einstellung (Ein/Aus) im eigenen Speicher, sodass ein Gerät, dessen PTP ausgeschaltet wurde, ausgeschaltet bleibt – und dessen Zeitstempel nicht PTP-synchronisiert sind –, bis es wieder eingeschaltet wird. `pool-list` markiert ein solches Gerät und gibt diesen Befehl für es aus. Chloros schaltet ein Gerät niemals von selbst wieder ein, da das Ausschalten beabsichtigt sein kann. Neu in 1.3.0.

## DAQ-E-Geräteeinstellungen

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` zeigt die Firmware des Sensors an (sowie, ob diese Chloros-Version eine neuere enthält), seine Adresse und wie er diese erhalten hat, die gespeicherte Netzwerkeinstellung, sofern sie von der aktuellen abweicht, sowie Power-On-Streaming und PTP. `pool-set-network` startet den Sensor neu, um die Änderung zu übernehmen, und verbindet ihn erneut mit den zuvor geltenden Einstellungen; `--no-reboot` speichert die Einstellungen lediglich. Eine statische Adresse muss im Netzwerk dieses Computers vorhanden sein, andernfalls lehnt Chloros sie ab; `--force` setzt dies außer Kraft, jedoch nur für einen Computer, der neu adressiert oder dorthin weitergeleitet wird. Neustarts werden abgelehnt<version\>,</version\> solange der Sensor Daten aufzeichnet. `pool-update-firmware` entspricht <version\>der Befehlszeilenversion von</version\> **„Update to \<version\>“** </version\>in der GUI <version\>und dient zur Aktualisierung eines DAQ-E unter Linux, wo keine GUI verfügbar ist. Neu in 1.3.0.***

## Befehlsübersicht

| Befehl | Zweck |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Einen Sensor im Backend-Pool öffnen. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Nach Sensoren suchen, ohne eine Verbindung herzustellen (ermittelt die BLE-MAC-Adresse eines DAQ-M). |
| `daq pool-list` | Zeigt jeden Sensor im Pool mit seiner `sensor_id`-ID an. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Die letzten N kalibrierten Spektrum-Frames. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Streaming fortsetzen / anhalten. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Eine `.daq`-Aufzeichnung starten / stoppen (backendseitig). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Das Cap-Korrekturprofil zur Laufzeit wechseln. |
| `daq pool-disconnect --sensor-id ID [--all]` | Einen Sensor oder alle Sensoren freigeben. |
| `daq pool-enable-ptp --sensor-id ID` | PTP für ein DAQ-E wieder aktivieren, dessen PTP deaktiviert ist. |
| `daq pool-info --sensor-id ID [--json]` | Firmware, Adresse, gespeicherte Netzwerkeinstellungen, Power-On-Streaming und PTP eines DAQ-E. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP oder eine statische Adresse; startet den Sensor neu und stellt die Verbindung wieder her. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Streaming beim Einschalten. |
| `daq pool-reboot --sensor-id ID` | Einen DAQ-E neu starten und erneut verbinden. |
| `daq pool-update-firmware --sensor-id ID` | Die DAQ-E-Firmware dieser Version auf einen älteren Sensor flashen. |

***

## Fehlerbehebung bei der ersten DAQ-E-Verbindung

1. Der DAQ-E verfügt über keine Status-LED – überprüfen Sie die Stromversorgung über die PoE-/Link-Anzeige am Switch oder am Injektor-Port und warten Sie nach dem Einschalten einige Sekunden, bis das Gerät hochgefahren ist und sich mit dem Netzwerk verbunden hat.
2. Der Backend-Rechner muss sich in derselben **Broadcast-Domäne** wie der Sensor befinden – mDNS durchquert keine Router.
3. Eine Firewall muss mDNS (UDP 5353), DAQ-E-Daten (UDP 5002) und PTP (UDP 319/320) durchlassen. Das Windows-Installationsprogramm fügt hierfür eine Regel hinzu, und das Linux-Programm `.deb` öffnet diese in einer `ufw`- bzw. `firewalld`-Firewall, die zum Zeitpunkt der Installation aktiv war; eine Firewall eines Drittanbieters oder eine später aktivierte Firewall muss diese zulassen.
4. Immer noch keine Reaktion von `--eth`? Verwenden Sie `--eth-host` mit dem Hostnamen (`daq-e-<id>.local`) oder der IP-Adresse des Geräts, wodurch die Erkennung übersprungen wird.

***
{% hint style="info" %}
**Tipp für KI-Assistenten.** Jede Seite dieses Handbuchs wird als reines Markdown bereitgestellt – fügen Sie `.md` an den kleingeschriebenen URL-Slug einer Seite an (diese Seite: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); der maschinenlesbare Index lautet `https://mapir.gitbook.io/chloros/llms.txt`. Für eine vollständige Dokumentation auf Flag-Ebene zu `chloros-cli daq` und allen anderen Befehlsfamilien rufe die [CLI-Referenz](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`) ab; Der Python-Pfad lautet `chloros_sdk.connect_daq_sensor()` in der [SDK-Referenz](../reference/sdk-reference.md).
{% endhint %}
</version\>
