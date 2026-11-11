# Chloros+ Anmeldung

## Anmeldung über die grafische Benutzeroberfläche

Über das Seitenleistenmenü des Benutzers <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> können Sie sich bei Ihrem Chloros+-Konto anmelden und zusätzliche Funktionen freischalten.

**Sie müssen sich nur einmal pro Rechner anmelden.** Die GUI, die CLI und das Python-SDK nutzen dieselbe zwischengespeicherte Sitzung – die Anmeldung über die Desktop-GUI aktiviert auch die CLI und das SDK auf diesem Rechner (und umgekehrt über `chloros-cli login`).

Nach der Anmeldung werden Ihre Kontodaten angezeigt:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Tarifstufen

| Tarif | `plan_id` | Typ |
| --- | --- | --- |
| Iron | `0` | Kostenlos |
| Copper | `1` | Kostenpflichtig (Chloros

+) |
| Bronze | `2` | Kostenpflichtig (Chloros

+) |
| Silver | `3` | Kostenpflichtig (Chloros

+) |
| Gold | `4` | Kostenpflichtig (Chloros

+) |

Unter [Tarife und Preise](https://cloud.mapir.camera/pricing) erfahren Sie, was die einzelnen kostenpflichtigen Tarife beinhalten.

### Für den Zugriff auf CLI

und SDK

ist ein kostenpflichtiger Tarif erforderlich

Der Zugriff auf CLI

und das Python

-SDK

erfordert **eine beliebige kostenpflichtige Chloros

+-Stufe (Copper oder höher)**. Dies wird**serverseitig** durchgesetzt – jede CLI

-/SDK

-Anfrage muss sowohl eine aktive Sitzung als auch einen kostenpflichtigen Tarif enthalten:

| HTTP

-Status | `error_code` | Bedeutung | Behebung |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Auf diesem Rechner nicht angemeldet | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Angemeldet, aber die Tarifstufe ist zu niedrig (kostenlose „Iron“-Stufe) | Auf einen beliebigen kostenpflichtigen „Chloros

+“-Tarif upgraden |

`chloros-cli status` bleibt im kostenlosen Tarif erreichbar, sodass Sie jederzeit Ihren aktuellen Tarif einsehen und erfahren können, warum der Zugriff verweigert wird.

### Beschränkungen für angeschlossene Hardware pro Tarif

Jeder Tarif legt fest, wie viele LATTICE-Kameras und DAQ-Lichtsensoren gleichzeitig live angeschlossen werden können:

| Tarif | LATTICE-Kameras | DAQ-Lichtsensoren |
| --- | --- | --- |
| Iron (kostenlos / nicht angemeldet) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI

-Anmeldung

Melden Sie sich mit Ihren Chloros

+-Anmeldedaten an, um die CLI

-Verarbeitung zu aktivieren. Unter Linux

(ohne GUI) ist dies die einzige Möglichkeit, Ihre Lizenz zu aktivieren.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK

-Benutzer**: Das Python

-SDK

bietet außerdem eine programmatische Methode `logout()` zum Löschen zwischengespeicherter Anmeldedaten. Weitere Informationen finden Sie in der [SDK

-Referenz](reference/sdk-reference.md).
{% endhint %}

**Beispiel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Sonderzeichen**: Setzen Sie Passwörter, die Zeichen wie `$`, `!` oder Leerzeichen enthalten, in einfache Anführungszeichen.
{% endhint %}

**Ausgabe:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Speicherung von Anmeldedaten

Zwischengespeicherte Anmeldedaten und Konfigurationen werden auf **allen Plattformen** im Ordner „`.chloros`“ Ihres Benutzer-Home-Verzeichnisses gespeichert:

| Plattform | Pfad zum Anmelde-Cache |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Wenn Sie eine neuere Version über eine bestehende installieren, bleiben Ihre Anmeldedaten unter Windows

(im Installationsprogramm) und unter Linux

(im Ordner „`.deb`“, einschließlich „`chloros-cli update`“) erhalten. Durch die Deinstallation von Chloros

– oder das Entfernen des Linux

-Pakets – wird die zwischengespeicherte Anmeldung gelöscht, sodass Sie sich nach einer Neuinstallation erneut anmelden müssen.

### Ablauf des Abonnements und Offline-Gnadenfrist

Der in der Benutzeroberfläche angezeigte Ablauf des Abonnements gibt an, wann Ihre Lizenz ungültig wird. Bei monatlichen Abonnements erfolgt der Ablauf am Monatsende; bei Jahresabonnements ein Jahr nach Beginn des Abonnements.

Chloros

validiert Ihre Lizenz online, unterstützt jedoch die Offline-Nutzung innerhalb einer Übergangsfrist:

* Erfolgreiche Servervalidierungen werden **5 Minuten** lang zwischengespeichert, sodass bei normaler Nutzung nur sehr wenige Lizenzabfragen erforderlich sind.
* Ein signierter, maschinengebundener Lizenz-Cache deckt längere Offline-Zeiträume ab: **30 Tage bei monatlichen Tarifen**und**bis zum Ablaufdatum Ihres Abonnements (maximal 365 Tage) bei jährlichen Tarifen**.
* Nach Ablauf der Karenzzeit wird der Tarif auf die kostenlose „Iron“-Stufe zurückgestuft, bis der Rechner den Lizenzserver einmal erreichen kann; der Zugriff wird bei der nächsten erfolgreichen Überprüfung wiederhergestellt.

### Gerätebegrenzung

Jeder Chloros

+-Tarif bietet eine unterschiedliche Anzahl registrierter Geräte. Jedes Gerät, auf dem Sie sich mit einem Chloros

+-Konto anmelden, wird auf Ihre Anzahl registrierter Geräte angerechnet. Sie können ein Gerät auf Ihrer MAPIR

-Cloud-Kontoseite umbenennen und entfernen.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros-+-Tarif</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILBER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Unterstützte Geräte</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Die genaue Anzahl der für Ihr Konto zulässigen Geräte wird auf Ihrer MAPIR-Cloud-Kontoseite angezeigt. Durch das Abmelden von einem Gerät wird dessen Platz zuverlässig freigegeben, und ein bereits registriertes Gerät kann sich jederzeit wieder anmelden, selbst wenn das Konto sein Geräte-Limit erreicht hat.
