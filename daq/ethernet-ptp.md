# DAQ-E: Netzwerk und Zeitsynchronisation

> Die physische Netzwerkeinrichtung für den Sensor – Verkabelung, PoE, IP-Zuweisung und die gerätespezifischen Netzwerkeinstellungen – finden Sie im **[DAQ-Benutzerhandbuch](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Diese Seite befasst sich mit dem Chloros: Anschluss, Zeitsynchronisation und was zu tun ist, wenn die Erkennung keine Ergebnisse liefert.

Das DAQ-E ist das Ethernet-Modell der DAQ-Familie: Es wird über PoE mit Strom versorgt, über mDNS (Dienst `_daq-e._tcp`) erkannt und ist über einen Hostnamen adressierbar, der von seiner Sensor-ID abgeleitet ist — `daq-e-<6 hex>.local`, z. B. `daq-e-def330.local`. Diese Seite behandelt, wie das Gerät Daten im Netzwerk überträgt und wie es an der PTP-Zeitsynchronisation teilnimmt.

## Übertragungsmodi

| Modus | Endpunkt | Empfänger | Anmerkungen |
| --- | --- | --- | --- |
| **Multicast** (Standard) | UDP `239.10.10.10:5002` | Beliebig viele Empfänger im selben LAN empfangen denselben Datenstrom | Jedes Datagramm wird per CRC-16/CCITT validiert |
| **Raw** | TCP-Port `5000` | Genau ein Client (exklusiv) | Byte-kompatibel mit DAQ-U |

Chloros verwendet standardmäßig Multicast, wodurch die GUI, CLI und SDK alle gleichzeitig einen Sensor überwachen können.

## Netzwerkanforderungen

* **Gleiche Broadcast-Domäne.** Der Rechner, auf dem Chloros läuft, muss sich im selben L2-Netzwerksegment wie der Sensor befinden – die mDNS-Erkennung durchquert keine Router.
* **Firewall-Abfrage zu Windows: Akzeptieren Sie diese.** Wenn Chloros zum ersten Mal die Multicast-Sockets bindet, fragt der Defender (Windows) einmal nach. Wenn Sie dies zulassen, werden DAQ-E-Daten (UDP 5002), mDNS (UDP 5353) und PTP (UDP 319/320) abgedeckt. Bei Linux erfolgt dies im Hintergrund.
* **PoE-Stromversorgung, keine Status-LED.** Das DAQ-E verfügt über keine eigene LED – überprüfen Sie die Stromversorgung über die Link-/PoE-Anzeige am Switch oder am Injektor-Port und warten Sie nach dem Einschalten einige Sekunden, bis das Gerät hochgefahren ist und sich mit dem Netzwerk verbunden hat.

## Verbinden

**GUI:** Registerkarte „Lichtsensoren“ → „Sensor verbinden“ → Gerätetyp „DAQ-E (Ethernet)“. Die Erkennung läuft nur, solange der Verbindungsdialog auf dem Bildschirm angezeigt wird (mDNS-Suche plus ARP-Scan auf Windows), und wiederholt sich alle 15 Sekunden; die Schaltfläche „Aktualisieren“ führt sofort einen erneuten Scan durch. Erkannte Sensoren erscheinen in der Dropdown-Liste; der erste erkannte Sensor wird automatisch ausgewählt.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (Backend läuft):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Hosts mit mehreren Netzwerkkarten und die erste Verbindung nach dem Systemstart

Auf Hosts mit mehr als einer aktiven Netzwerkschnittstelle kann die **erste** `pool-connect --eth` nach dem Booten leer sein, selbst wenn der Sensor funktionsfähig ist – die Erkennungsabfrage kann die Schnittstelle, auf der sich der Sensor befindet, übersehen, solange der ARP-Cache noch nicht aufgefüllt ist. Die zuverlässige Abhilfe besteht darin, die Erkennung zu überspringen und die Adresse explizit anzugeben:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` akzeptiert den mDNS-Hostnamen oder die IP-Adresse, zielt immer auf den richtigen Sensor ab und ist die empfohlene Form für Skripte und headless-Installationen. Verwenden Sie in der GUI die Schaltfläche „Aktualisieren“ im Verbindungsdialog und lassen Sie einen erneuten Suchdurchlauf zu.

## Geräteeinstellungen und Firmware

Der Sensor selbst enthält Netzwerkeinstellungen – statische IP-Adresse vs. DHCP + Link-Local-Adressierung, Gerätename, automatischer Stream beim Booten, OTA-Passwort. Diese geräteseitigen Einstellungen sind im ausgelieferten CLI nicht als Befehle verfügbar; sie werden über die Chloros-GUI verwaltet, sofern dort angezeigt, oder mit Hilfe des MAPIR-Supports.

**Firmware-Updates sind in die GUI integriert.**Wenn ein angeschlossenes DAQ-E eine Firmware verwendet, die älter ist als das mit Ihrem Chloros-Build mitgelieferte Image, wird in der Sensorzeile ein gelbes**Update verfügbar**-Symbol angezeigt, und im Einstellungsmodal mit dem Zahnrad-Symbol wird die<version>

Schaltfläche</version> „Auf aktualisieren<version>

“</version> angeboten<version>

. Das Update wird in etwa 30 Sekunden über das Netzwerk übertragen; der Sensor startet neu und stellt automatisch die Verbindung wieder her, und bei einer unterbrochenen Übertragung bleibt die aktuelle Firmware erhalten.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-Zeitsynchronisation

Die DAQ-E-Firmware ab Version 1.2.0 nimmt am IEEE 1588 PTPv2 als gewöhnliche (reine Slave-)Uhr teil. **Das Backend des Chloros-Hosts ist der PTP-Grandmaster** – jedes DAQ-E-Gerät und jede LATTICE-Kamera im LAN ist in Domäne 0 diesem untergeordnet, wodurch alle Zeitstempel der Geräte innerhalb einer Toleranz von ca. 1 ms gehalten werden. Diese gemeinsame Uhr sorgt dafür, dass die Zeitstempel der DAQ-Messwerte mit den Belichtungszeiten der Kamera abgeglichen werden können (siehe [Aufzeichnung und das .daq-Format](recording.md)).

Überprüfen Sie die Synchronisation anhand von CLI:

| Befehl | Zeigt an |
| --- | --- |
| `chloros-cli time-sync status` | Status des Host-Grandmasters, BMCA-Prioritäten, Taktidentität |
| `chloros-cli time-sync peers` | Alle erkannten Slaves (DAQ-E-Sensoren + LATTICE-Kameras) |
| `chloros-cli time-sync cameras` | PTP-Zustand pro Kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Neustart des Grandmaster-Prozesses |

In der GUI zeigt das Einstellungsfenster „DAQ-E“ eine Live-Zeile „**PTP-Synchronisation**“ mit dem aktuellen PTP-Status des Sensors an.

Details für Verbraucher mit strikter Synchronisation:

* Jedes gestreamte Datagramm enthält ein Flags-Feld; **Bit 2 ist bei Frames gesetzt, deren Zeitstempel PTP-synchronisiert ist**. Pipelines, die eine strikte Synchronisation von Kamera und DAQ erfordern, sollten anhand dieses Bits gesteuert werden.
* Vergewissern Sie sich vor einer synchronisierten Erfassung, dass der Sensor in `chloros-cli time-sync peers` aufgeführt ist. (Die interne, direkt auf die Hardware abgestimmte Tooling-Umgebung von MAPIR kann die Aufzeichnung ebenfalls auf den PTP-Lock ausrichten – mithilfe eines `--wait-ptp`-Flags, das bis zu 15 Sekunden darauf wartet, dass der Sensor den SLAVE-Zustand erreicht; diese Tooling-Funktion ist nicht Bestandteil des ausgelieferten CLI.)
* Während PTP aktiv als Slave arbeitet, lehnt der Sensor manuelle Taktbefehle ab („PTP liefert den Takt“). Dies entspricht dem Design – vertrauen Sie auf PTP.

## Hinweise zu Linux

* **PTP benötigt `libcap2-bin` bei der Installation.** Das Skript „`.deb` postinst“ erteilt „`cap_net_bind_service=+ep`“ auf „`/usr/lib/chloros/chloros-backend`“, damit PTP-Ports 319/320 ohne Root-Rechte gebunden werden können. Fehlt „`libcap2-bin`“, wird dieser Schritt übersprungen und PTP kann nicht gestartet werden. Behebung:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Headless Jetson / Raspberry Pi:** Bei der Erstinstallation wird die systemd-Einheit „`chloros-backend.service`“ generiert, aber nicht aktiviert. Für einen ständig aktiven PTP-Betrieb (und die Verfügbarkeit von DAQ) ohne GUI:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Ohne diese Einheit läuft PTP nur, solange die Chloros-GUI geöffnet ist.

## Fehlerbehebung: „Keine DAQ-E-Geräte gefunden“

| Überprüfung | Details |
| --- | --- |
| Stromversorgung | Keine LED am Sensor – überprüfen Sie die PoE- und Link-Anzeigen des Switches/Injektor-Ports; warten Sie nach dem Einschalten einige Sekunden |
| Broadcast-Domäne | Host und Sensor befinden sich im selben L2-Segment; mDNS wird nicht weitergeleitet |
| Windows-Firewall | Akzeptieren Sie die Defender-Eingabeaufforderung beim ersten Start (UDP 5002, 5353, 319/320) |
| Host mit mehreren Netzwerkkarten | Bei der ersten Erkennung nach dem Systemstart wird der Sensor möglicherweise nicht gefunden – stellen Sie eine Verbindung mit `--eth-host <ip-or-hostname>` her |
| Erneuter Suchlauf über die GUI | Die Erkennung läuft nur, solange der Verbindungsdialog geöffnet ist; verwenden Sie „Aktualisieren“ |</version>
