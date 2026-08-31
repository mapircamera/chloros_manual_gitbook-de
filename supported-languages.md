# Unterstützte Sprachen

Chloros bietet vollständige Benutzeroberflächenunterstützung in **38 Sprachen weltweit** und ist somit für Nutzer auf der ganzen Welt zugänglich. Sie können sowohl in der Desktop-Benutzeroberfläche als auch im CLI sofort zwischen den Sprachen wechseln.

Chloros unterstützt die folgenden Sprachen:

| # | Sprache | Name in der Originalsprache | CLI-Code |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Englisch | English | `en` |
| 2 | 🇪🇸 Spanisch | Español | `es` |
| 3 | 🇵🇹 Portugiesisch | Português | `pt` |
| 4 | 🇫🇷 Französisch | Français | `fr` |
| 5 | 🇩🇪 Deutsch | Deutsch | `de` |
| 6 | 🇮🇹 Italienisch | Italiano | `it` |
| 7 | 🇯🇵 Japanisch | 日本語 | `ja` |
| 8 | 🇰🇷 Koreanisch | 한국어 | `ko` |
| 9 | 🇨🇳 Chinesisch (vereinfacht) | 简体中文 | `zh` |
| 10 | 🇹🇼 Chinesisch (traditionell) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Russisch | Русский | `ru` |
| 12 | 🇳🇱 Niederländisch | Nederlands | `nl` |
| 13 | 🇸🇦 Arabisch | العربية | `ar` |
| 14 | 🇵🇱 Polnisch | Polski | `pl` |
| 15 | 🇹🇷 Türkisch | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonesisch | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamesisch | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thailändisch | ไทย | `th` |
| 20 | 🇸🇪 Schwedisch | Svenska | `sv` |
| 21 | 🇩🇰 Dänisch | Dansk | `da` |
| 22 | 🇳🇴 Norwegisch | Norsk | `no` |
| 23 | 🇫🇮 Finnisch | Suomi | `fi` |
| 24 | 🇬🇷 Griechisch | Ελληνικά | `el` |
| 25 | 🇨🇿 Tschechisch | Čeština | `cs` |
| 26 | 🇭🇺 Ungarisch | Magyar | `hu` |
| 27 | 🇷🇴 Rumänisch | Română | `ro` |
| 28 | 🇺🇦 Ukrainisch | Українська | `uk` |
| 29 | 🇧🇷 Brasilianisches Portugiesisch | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Kantonesisch | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malaiisch | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slowakisch | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgarisch | Български | `bg` |
| 34 | 🇭🇷 Kroatisch | Hrvatski | `hr` |
| 35 | 🇱🇹 Litauisch | Lietuvių | `lt` |
| 36 | 🇱🇻 Lettisch | Latviešu | `lv` |
| 37 | 🇪🇪 Estnisch | Eesti | `et` |
| 38 | 🇸🇮 Slowenisch | Slovenščina | `sl` |

## So ändern Sie die Sprache

### In Chloros Desktop

1. Öffnen Sie die Anwendungseinstellungen.
2. Navigieren Sie zum Sprachauswahlmenü.
3. Wählen Sie Ihre bevorzugte Sprache aus der Liste aus.
4. Die Benutzeroberfläche wird sofort aktualisiert.

### In Chloros CLI

Verwenden Sie den Befehl `language`, um die Sprache der Benutzeroberfläche von CLI anzuzeigen oder zu ändern:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Weitere Informationen finden Sie in der [CLI-Dokumentation](CLI.md).

## Abdeckung

Alle 38 Sprachen werden vollständig unterstützt in:

* **Chloros Desktop** – Vollständige Übersetzung der grafischen Benutzeroberfläche
* **Chloros CLI** – Befehlszeilenschnittstelle und Ausgabemeldungen

Das Python SDK API und die dazugehörige [Referenzdokumentation](reference/sdk-reference.md) werden in englischer Sprache bereitgestellt.

Die Sprachunterstützung gewährleistet, dass Nutzer weltweit barrierefrei und effizient in ihrer Muttersprache arbeiten können.
