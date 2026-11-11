# DAQ-E-Netzwerk und Zeitsynchronisation

> Die physische Netzwerkeinrichtung für den Sensor – Verkabelung, PoE, IP-Zuweisung und die gerätespezifischen Netzwerkeinstellungen – ist im **[DAQ-Benutzerhandbuch](https://mapir.gitbook.io/daq/daq-e/network-setup)** beschrieben. Diese Seite befasst sich mit der Chloros

-Seite: Anschluss, Zeitsynchronisation und was zu tun ist, wenn die Erkennung keine Ergebnisse liefert.

Der DAQ-E ist das Ethernet-Modell der DAQ-Familie: Er wird über PoE mit Strom versorgt, über mDNS (Dienst `_daq-e._tcp`) erkannt und ist über einen Hostnamen adressierbar, der sich aus seiner Sensor-ID — `daq-e-<6 hex>.local`, z. B. `daq-e-def330.local`. Diese Seite behandelt, wie das Gerät Daten im Netzwerk überträgt und wie es an der PTP-Zeitsynchronisation teilnimmt.

## Übertragungsmodi

| Modus | Endpunkt | Empfänger | Anmerkungen |
| --- | --- | --- | --- |
| **Multicast** (Standard) | UDP `239.10.10.10:5002` | Beliebig viele Empfänger im selben LAN empfangen denselben Datenstrom | Jedes Datagramm wird per CRC-16/CCITT validiert |
| **Raw** | TCP-Port `5000` | Genau ein Client (exklusiv) | Drop-in-Byte-kompatibel mit DAQ-U |

Chloros

verwendet standardmäßig Multicast, wodurch die GUI, die CLI

und das SDK

gleichzeitig einen Sensor überwachen können.

## Netzwerkanforderungen

* **Gleiche Broadcast-Domäne.** Der Rechner, auf dem Chloros

läuft, muss sich im selben L2-Netzwerksegment wie der Sensor befinden – die mDNS-Erkennung durchquert keine Router.
* **Firewall.** Unter Windows

fügt das Installationsprogramm in jedem Netzwerkprofil die Firewall-Regel **„Chloros

-Backend (UDP-In)“**für das Backend hinzu, die DAQ-E-Daten (UDP 5002), mDNS (UDP 5353) und PTP (UDP 319/320) abdeckt, sowie eine auf das lokale Subnetz beschränkte Regel**„Chloros

-Backend (TCP-In)“** für die Verbindung, die ein DAQ-E während eines Firmware-Updates zum Computer herstellt. Unter Linux

öffnet `.deb` die UDP-Ports in `ufw` oder `firewalld`, wenn diese Firewall zum Zeitpunkt der Installation aktiv ist. Bei einer Firewall eines Drittanbieters oder einer später aktivierten Firewall müssen dieselben Ports freigegeben werden.
* **PoE-Stromversorgung, keine Status-LED.** Das DAQ-E verfügt über keine eigene LED – überprüfen Sie die Stromversorgung über die Link-/PoE-Anzeige am Switch oder am Injektor-Port und warten Sie nach dem Einschalten einige Sekunden, bis das Gerät hochgefahren ist und sich mit dem Netzwerk verbunden hat.

## Verbinden

**GUI:** Registerkarte „Lichtsensoren“ → „Sensor verbinden“ → Gerätetyp „DAQ-E (Ethernet)“. Die Erkennung läuft nur, solange der Verbindungsdialog auf dem Bildschirm angezeigt wird (mDNS-Suche plus ARP-Scan unter Windows

), und wird alle 15 Sekunden wiederholt; die Schaltfläche „Aktualisieren“ führt sofort einen erneuten Suchlauf durch. Erkannte Sensoren werden in der Dropdown-Liste angezeigt; der erste erkannte Sensor wird automatisch ausgewählt.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (Backend läuft):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Ein bereits im Pool vorhandener DAQ-E wird sowohl anhand seines Hostnamens als auch anhand seiner IP-Adresse erkannt: Wird er erneut unter der anderen Adresse verbunden, wird der bestehende Pool-Eintrag zurückgegeben, anstatt das Gerät ein zweites Mal zu öffnen.

### Hosts mit mehreren Netzwerkkarten und die erste Verbindung nach dem Systemstart

Auf Hosts mit mehr als einer aktiven Netzwerkschnittstelle kann der **erste** `pool-connect --eth` nach dem Systemstart leer sein, selbst wenn der Sensor funktionsfähig ist – die Erkennungsabfrage kann die Schnittstelle, auf der sich der Sensor befindet, übersehen, solange der ARP-Cache noch nicht geladen ist. Die zuverlässige Abhilfe besteht darin, die Erkennung zu überspringen und die Adresse explizit zu übergeben:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` akzeptiert den mDNS-Hostnamen oder die IP-Adresse, zielt immer auf den richtigen Sensor ab und ist die empfohlene Form für Skripte und Headless-Installationen. Verwenden Sie in der GUI die Schaltfläche „Aktualisieren“ im Verbindungsdialog und warten Sie einen erneuten Suchdurchlauf ab.

## Geräteeinstellungen und Firmware

Der Sensor selbst speichert seine Netzwerkeinstellungen: DHCP (die Werkseinstellung, die auf eine Link-Local-Adresse vom Typ 169.254.x.x zurückgreift, wenn kein DHCP-Server antwortet) oder eine statische Adresse. Außerdem wird gespeichert, ob er beim Einschalten sofort mit dem Streaming beginnt. Ändern Sie diese Einstellungen in der GUI (in den Zahnrad-Einstellungen des Sensors unter **Netzwerk**:**Ändern…**,**Beim Einschalten streamen**,**Sensor neu starten**; siehe [Registerkarte „DAQ“](gui.md)) oder über die CLI

und das SDK

mit `pool-set-network`, `pool-set-auto-stream` und `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; siehe [CLI

-Referenz](../reference/cli-reference.md#daq-e-device-settings)). Der Sensor wird neu gestartet, um eine Netzwerkänderung zu übernehmen, und Chloros

stellt die Verbindung wieder her. Im Netzwerk des Computers, auf dem Chloros

läuft, muss eine statische Adresse vorhanden sein: Chloros

lehnt eine Adresse ab, die es nach dem Neustart nicht erreichen konnte. Die ID des Sensors und sein Hostname `daq-e-<id>.local` ändern sich nie. Neu in 1.3.0.

**Firmware-Updates sind in die GUI integriert.** Wenn ein angeschlossener DAQ-E eine Firmware verwendet, die älter ist als das mit Ihrem Chloros

-Build mitgelieferte Image, wird in der Sensorzeile ein gelbes **Update verfügbar**-Symbol angezeigt, und im Einstellungsmodal mit dem Zahnrad wird die <version>Schaltfläche</version>

„Auf aktualisieren<version>“</version>

angeboten<version>. Das Update wird in etwa 30 Sekunden über das Netzwerk übertragen; der Sensor startet neu und stellt automatisch die Verbindung wieder her, und bei einer unterbrochenen Übertragung bleibt die aktuelle Firmware erhalten. Über die CLI oder das SDK sowie unter Linux, wo keine GUI vorhanden ist, führen Sie `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`) aus.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-Zeitsynchronisation

Die DAQ-E-Firmware v1.2.0+ nimmt am IEEE 1588 PTPv2 als gewöhnliche (reine Slave-)Uhr teil. **Das Backend des Chloros-Hosts ist der PTP-Grandmaster** – jedes DAQ-E-Gerät und jede LATTICE-Kamera im LAN ist in Domäne 0 diesem untergeordnet, wodurch alle Zeitstempel der Geräte innerhalb einer Toleranz von ca. 1 ms gehalten werden. Dank dieser gemeinsamen Uhr lassen sich die DAQ-Messwerte zeitlich mit den Belichtungszeiten der Kameras abgleichen (siehe [Aufzeichnung und das .daq-Format](recording.md)).

Überprüfen Sie die Synchronisation über die CLI:

| Befehl | Zeigt an |
| --- | --- |
| `chloros-cli time-sync status` | Status des Host-Grandmasters, BMCA-Prioritäten, Taktidentität sowie `health`: `OK` oder `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` mit einer Angabe von `health_detail`, warum seine Pakete das Netzwerk nicht erreichen |
| `chloros-cli time-sync peers` | Alle Slaves werden erkannt (DAQ-E-Sensoren + LATTICE-Kameras) |
| `chloros-cli time-sync cameras` | PTP-Zustand pro Kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Grandmaster-Prozess neu starten |

In der Benutzeroberfläche zeigt das Einstellungsfenster für DAQ-E eine Live-Zeile „**PTP-Synchronisation**“ mit dem aktuellen PTP-Status des Sensors an.**Ein DAQ-E mit deaktiviertem PTP bleibt deaktiviert.** Das Gerät speichert seine PTP-Einstellung (ein/aus) in seinem eigenen Speicher, sodass ein Gerät, dessen PTP deaktiviert war, auch nach einer erneuten Verbindung oder einem Neustart deaktiviert bleibt und seine Zeitstempel nicht PTP-synchronisiert sind. `chloros-cli daq pool-list` kennzeichnet ein solches Gerät (`!! PTP is switched OFF on this unit …`) und gibt den Befehl aus, der das Problem behebt:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros schaltet ein Gerät niemals von selbst wieder ein, da der ausgeschaltete Zustand beabsichtigt sein kann.

Details für Anwender, die eine strenge Synchronisation benötigen:

* Jedes gestreamte Datagramm enthält ein Flags-Feld; **Bit 2 ist bei Frames gesetzt, deren Zeitstempel PTP-synchronisiert ist**. Pipelines, die eine strenge Synchronisation von Kamera und Datenerfassung erfordern, sollten anhand dieses Bits gesteuert werden.
* Vergewissern Sie sich vor einer synchronisierten Erfassung, dass der Sensor in `chloros-cli time-sync peers` erscheint und dass `chloros-cli daq pool-list` dessen PTP nicht als ausgeschaltet kennzeichnet. (Die internen, direkt auf die Hardware zugreifenden Tools von MAPIR können die Aufzeichnung auch anhand der PTP-Synchronisation mit einem `--wait-ptp`-Flag steuern, das bis zu 15 s darauf wartet, dass der Sensor den SLAVE-Zustand erreicht; diese Tools sind nicht Teil der ausgelieferten CLI.)
* Solange PTP aktiv als Slave arbeitet, lehnt der Sensor manuelle Taktübergaben ab („PTP liefert den Takt“). Dies entspricht dem Design – vertrauen Sie auf PTP.

## Hinweise zu Linux

* **PTP-Ports ohne Root-Rechte.** PTP bindet an UDP 319/320. `.deb` schreibt `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), sodass das Backend diese Ports bindet und als Ihr Benutzer ausführt. Ab Version 1.3.0 setzt das Paket keine Dateiberechtigung (`setcap`) für das Backend und benötigt kein `libcap2-bin`; fügen Sie keine Berechtigung selbst hinzu, da das entpackte Backend sonst nicht startet. Falls PTP den Port 319 nicht binden kann, siehe [Linux-Installation](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Headless Jetson / Linux-Host:** Das Paket installiert die Backend-Dienste, aktiviert jedoch keinen davon. Für einen ständig aktiven PTP-Dienst (und DAQ-Verfügbarkeit), ohne dass jemand einen Befehl ausführt, aktivieren Sie den Dienst als Ihr Benutzer:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Ohne diese Maßnahme läuft PTP nur, solange das Backend von etwas gestartet wurde – einem Befehl „`chloros-cli`“ oder einem SDK-Skript. Siehe [Linux-Installation](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) für den Root-Benutzer `chloros-backend.service` und Informationen zum Wechseln aus diesem Benutzer.

## Fehlerbehebung: „Keine DAQ-E-Geräte gefunden“

| Überprüfung | Details |
| --- | --- |
| Stromversorgung | Keine LED am Sensor – überprüfen Sie die PoE- und Link-Anzeigen des Switches/Injektor-Ports; warten Sie nach dem Einschalten einige Sekunden |
| Broadcast-Domäne | Host und Sensor befinden sich im selben L2-Segment; mDNS wird nicht weitergeleitet |
| Firewall | Windows: Die Regel **„Chloros Backend (UDP-In)“** des Installationsprogramms lässt den DAQ-E-Stream (UDP 5002), mDNS (5353) und PTP (319/320) durch; eine Firewall eines Drittanbieters muss dies ebenfalls zulassen. Linux: `.deb` öffnet diese Ports in `ufw` / `firewalld` nur, wenn die Firewall zum Zeitpunkt der Installation aktiv war – lassen Sie sie selbst zu, wenn die Firewall später aktiviert wird |
| Host mit mehreren Netzwerkkarten | Die Erkennung durchsucht mDNS mit einem ARP-Fallback; `--eth-host <ip-or-hostname>` überspringt die Erkennung und stellt direkt eine Verbindung her |
| Erneuter Scan über die GUI | Die Erkennung läuft nur, solange der Verbindungsdialog geöffnet ist; verwenden Sie „Aktualisieren“ |</version>
