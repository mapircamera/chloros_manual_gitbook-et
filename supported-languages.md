# Toetatud keeled

Chloros pakub täielikku kasutajaliidese tuge **38 keeles üle maailma**, muutes selle kättesaadavaks kasutajatele kogu maailmas. Keelt saab vahetada koheselt kõigis kasutajaliidestes: töölaual, brauseris, CLI ja Python SDK.

Chloros toetab järgmisi keeli:

| # | Keel | Keele nimi | CLI Kood |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Inglise keel | English | `en` |
| 2 | 🇪🇸 Hispaania | Español | `es` |
| 3 | 🇵🇹 Portugali | Português | `pt` |
| 4 | 🇫🇷 Prantsuse | Français | `fr` |
| 5 | 🇩🇪 Saksa keel | Deutsch | `de` |
| 6 | 🇮🇹 Itaalia keel | Italiano | `it` |
| 7 | 🇯🇵 Jaapani keel | 日本語 | `ja` |
| 8 | 🇰🇷 Korea | 한국어 | `ko` |
| 9 | 🇨🇳 Hiina (lihtsustatud) | 简体中文 | `zh` |
| 10 | 🇹🇼 Hiina (traditsiooniline) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Vene | Русский | `ru` |
| 12 | 🇳🇱 Hollandi keel | Nederlands | `nl` |
| 13 | 🇸🇦 Araabia keel | العربية | `ar` |
| 14 | 🇵🇱 Poola keel | Polski | `pl` |
| 15 | 🇹🇷 Türgi | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indoneesia | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnami | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Tai | ไทย | `th` |
| 20 | 🇸🇪 Rootsi | Svenska | `sv` |
| 21 | 🇩🇰 Taani | Dansk | `da` |
| 22 | 🇳🇴 Norra | Norsk | `no` |
| 23 | 🇫🇮 Soome | Suomi | `fi` |
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
| 34 | 🇭🇷 Horvaatia keel | Hrvatski | `hr` |
| 35 | 🇱🇹 Leedu keel | Lietuvių | `lt` |
| 36 | 🇱🇻 Läti keel | Latviešu | `lv` |
| 37 | 🇪🇪 Eesti | Eesti | `et` |
| 38 | 🇸🇮 Sloveeni | Slovenščina | `sl` |

## Kuidas keelt muuta

### Chloros töölaual/brauseris

1. Avage rakenduse seaded
2. Minge keelevaliku menüüsse
3. Valige loendist soovitud keel
4. Kasutajaliides uueneb kohe

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

Lisateabe saamiseks vaadake [CLI dokumentatsiooni](CLI.md).

### Chloros Python SDK

Määrake keeleparameeter SDK algseadistamisel, et saada teateid ja väljundeid oma eelistatud keeles.

## Toetatavad keeled

Kõiki 38 keelt toetatakse täielikult järgmistes rakendustes:

* **Chloros Desktop** – täielik graafilise kasutajaliidese tõlge
* **Chloros Browser** – veebiliides kõigis keeltes
* **Chloros CLI** – käsurealiides ja väljundsõnumid
* **Chloros Python SDK** – API-sõnumid ja dokumentatsioon

Keelte tugi tagab, et kasutajad üle kogu maailma saavad takistusteta ja tõhusalt töötada oma emakeeles.
