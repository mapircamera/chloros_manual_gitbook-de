# Chloros+ Anmeldung

## Chloros und Chloros (Browser) Anmeldung

Über das <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> Seitenleiste ermöglicht es Ihnen, sich bei Ihrem Chloros+-Konto anzumelden und zusätzliche Funktionen freizuschalten.

Nach der Anmeldung werden Ihre Kontodaten angezeigt:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI-Anmeldung

Melden Sie sich mit Ihren Chloros+-Anmeldedaten an, um die CLI-Verarbeitung zu aktivieren. Bei Linux (ohne GUI) ist dies die einzige Möglichkeit, Ihre Lizenz zu aktivieren.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK-Benutzer**: Das Python SDK bietet auch eine programmatische `logout()`-Methode zum Löschen zwischengespeicherter Anmeldedaten. Weitere Informationen finden Sie in der [Python SDK Dokumentation](api-python-sdk.md#logout).
{% endhint %}

**Beispiel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Sonderzeichen**: Setzen Sie Passwörter, die Zeichen wie `$`, `!` oder Leerzeichen enthalten, in einfache Anführungszeichen.
{% endhint %}

**Ausgabe:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Speicherung von Anmeldedaten

Zwischengespeicherte Anmeldedaten werden an einem plattformspezifischen Speicherort abgelegt:

| Plattform | Pfad zum Anmeldedaten-Cache |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Ablauf des Tarifs

Der im GUI angezeigte Ablauf des Tarifs gibt an, wann Ihre Lizenz ungültig wird. Bei monatlichen Abonnements erfolgt der Ablauf am Monatsende. Bei Jahresabonnements ist dies ein Jahr nach Beginn des Abonnements. Die Lizenzprüfung erfordert eine monatliche Internetverbindung zur Verifizierung, mit einer 30-tägigen Nachfrist.

### Gerätebegrenzung

Jeder Chloros+-Tarif bietet eine unterschiedliche Anzahl registrierter Geräte. Jedes Gerät, auf dem Sie sich mit einem Chloros+-Konto anmelden, wird auf Ihre Anzahl registrierter Geräte angerechnet. Sie können ein Gerät auf Ihrer MAPIR Cloud-Kontoseite umbenennen und entfernen.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+-Tarif</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILBER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Unterstützte Geräte</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
