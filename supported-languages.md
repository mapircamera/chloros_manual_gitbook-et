# Toetatud keeled

Chloros pakub täielikku liidese tuge **38 keeles üle maailma**, muutes selle kättesaadavaks kasutajatele kogu maailmas. Keelt saab vahetada koheselt kõigis liidestes: töölaual, brauseris, CLI ja Python SDK.

Chloros toetab järgmisi keeli:

| # | Keel | Emakeel | CLI kood |
|---|----------|-------------|----------|
| 1 | 🇺🇸 inglise keel | inglise keel | `en` |
| 2 | 🇪🇸 Hispaania | Español | `es` |
| 3 | 🇵🇹 Portugali | Português | `pt` |
| 4 | 🇫🇷 Prantsuse | Français | `fr` |
| 5 | 🇩🇪 Saksa keel | Deutsch | `de` |
| 6 | 🇮🇹 Itaalia keel | Italiano | `it` |
| 7 | 🇯🇵 Jaapani keel | 日本語 | `ja` |
| 8 | 🇰🇷 Korea keel | 한국어 | `ko` |
| 9 | 🇨🇳 Hiina keel (lihtsustatud) | 简体中文 | `zh` |
| 10 | 🇹🇼 Hiina (traditsiooniline) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Vene | Русский | `ru` |
| 12 | 🇳🇱 Hollandi keel | Nederlands | `nl` |
| 13 | 🇸🇦 Araabia keel | العربية | `ar` |
| 14 | 🇵🇱 Poola keel | Polski | `pl` |
| 15 | 🇹🇷 türgi keel | Türkçe | `tr` |
| 16 | 🇮🇳 hindi keel | हिंदी | `hi` |
| 17 | 🇮🇩 Indoneesia keel | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnami keel | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Tai | ไทย | `th` |
| 20 | 🇸🇪 Rootsi | Svenska | `sv` |
| 21 | 🇩🇰 Taani | Dansk | `da` |
| 22 | 🇳🇴 Norra keel | Norsk | `no` |
| 23 | 🇫🇮 Soome keel | Suomi | `fi` |
| 24 | 🇬🇷 Kreeka | Ελληνικά | `el` |
| 25 | 🇨🇿 Tšehhi | Čeština | `cs` |
| 26 | 🇭🇺 Ungari | Magyar | `hu` |
| 27 | 🇷🇴 Rumeenia | Română | `ro` |
| 28 | 🇺🇦 Ukraina | Українська | `uk` |
| 29 | 🇧🇷 Brasiilia portugali keel | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Kantoni keel | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malai keel | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovaki keel | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgaaria keel | Български | `bg` |
| 34 | 🇭🇷 Horvaadi keel | Hrvatski | `hr` |
| 35 | 🇱🇹 Leedu keel | Lietuvių | `lt` |
| 36 | 🇱🇻 Läti keel | Latviešu | `lv` |
| 37 | 🇪🇪 Eesti | Eesti | `et` |
| 38 | 🇸🇮 Sloveenia | Slovenščina | `sl` |

## Keele muutmine

### Chloros töölaual/brauseris

1. Avage rakenduse seaded.
2. Navigeerige keelevaliku menüüsse.
3. Valige loendist soovitud keel.
4. Kasutajaliides uueneb kohe.

### Chloros CLI

Kasutage käsku `language`, et vaadata või muuta CLI kasutajaliidese keelt:

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

Täpsema teabe saamiseks vaadake [CLI dokumentatsiooni](CLI.md).

### Chloros Python SDK

Määrake keele parameeter SDK algseadistamisel, et saada sõnumeid ja väljundeid soovitud keeles.

## Katvus

Kõik 38 keelt on täielikult toetatud järgmistes rakendustes:

* **Chloros Desktop** - täielik GUI tõlge
* **Chloros Browser** - veebiliides kõigis keeltes
* **Chloros CLI** - käsurealiides ja väljundsõnumid
* **Chloros Python SDK** – API sõnumid ja dokumentatsioon

Keelte tugi tagab, et kasutajad üle kogu maailma saavad tõhusalt töötada oma emakeeles ilma takistusteta.
