# Chloros+-Anmeldung

## Chloros und Chloros (Browser) Login

Über das Benutzermenü in der Seitenleiste <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> können Sie sich bei Ihrem Chloros+-Konto anmelden und zusätzliche Funktionen freischalten.

Wenn Sie angemeldet sind, werden Ihre Kontodaten angezeigt:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLI-Anmeldung

Melden Sie sich mit Ihren Chloros+-Anmeldeinformationen an, um die CLI-Verarbeitung zu aktivieren.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

**Beispiel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Sonderzeichen**: Verwenden Sie einfache Anführungszeichen um Passwörter, die Zeichen wie `$`, `!` oder Leerzeichen enthalten.
{% endhint %}

**Ausgabe:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### Planablauf

Der Planablauf in der GUI zeigt an, wann Ihre Lizenz ungültig wird. Bei wiederkehrenden Monatsabonnements endet der Ablauf am Monatsende. Bei Jahresabonnements beträgt der Zeitraum ein Jahr nach Beginn des Abonnements. Für die Lizenzprüfung ist eine monatliche Internetverbindung zur Überprüfung erforderlich, mit einer Nachfrist von 30 Tagen.

### Gerätelimit

Jeder Chloros+-Plan bietet eine unterschiedliche Anzahl registrierter Geräte. Jedes Gerät, bei dem Sie sich mit einem Chloros+-Konto anmelden, wird auf die Anzahl Ihrer registrierten Geräte angerechnet. Sie können ein Gerät auf Ihrer MAPIR Cloud-Kontoseite umbenennen und entfernen.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ Plan</th><th align="center">KUPFER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Geräte Unterstützt</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
