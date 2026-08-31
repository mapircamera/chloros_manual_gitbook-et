# Toetatud keeled

Chloros pakub täielikku kasutajaliidese tuge **38 keeles üle maailma**, muutes selle kättesaadavaks kasutajatele kogu maailmas. Keelt saab vahetada hetkega nii töölaua graafilises kasutajaliideses kui ka CLI-is.

Chloros toetab järgmisi keeli:

| # | Keel | Algne nimi | CLI-kood |
|---|----------|-------------|----------|
| 1 | 🇺🇸 inglise keel | English | `en` |
| 2 | 🇪🇸 hispaania keel | Español | `es` |
| 3 | 🇵🇹 Portugali keel | Português | `pt` |
| 4 | 🇫🇷 Prantsuse keel | Français | `fr` |
| 5 | 🇩🇪 Saksa keel | Deutsch | `de` |
| 6 | 🇮🇹 Itaalia keel | Italiano | `it` |
| 7 | 🇯🇵 jaapani keel | 日本語 | `ja` |
| 8 | 🇰🇷 korea keel | 한국어 | `ko` |
| 9 | 🇨🇳 Hiina (lihtsustatud) | 简体中文 | `zh` |
| 10 | 🇹🇼 Hiina (traditsiooniline) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Vene | Русский | `ru` |
| 12 | 🇳🇱 Hollandi keel | Nederlands | `nl` |
| 13 | 🇸🇦 Araabia keel | العربية | `ar` |
| 14 | 🇵🇱 Poola keel | Polski | `pl` |
| 15 | 🇹🇷 Türgi keel | Türkçe | `tr` |
| 16 | 🇮🇳 hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indoneesia keel | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnami keel | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Tai keel | ไทย | `th` |
| 20 | 🇸🇪 Rootsi keel | Svenska | `sv` |
| 21 | 🇩🇰 Taani keel | Dansk | `da` |
| 22 | 🇳🇴 Norra keel | Norsk | `no` |
| 23 | 🇫🇮 Soome | Suomi | `fi` |
| 24 | 🇬🇷 Kreeka | Ελληνικά | `el` |
| 25 | 🇨🇿 Tšehhi | Čeština | `cs` |
| 26 | 🇭🇺 Ungari | Magyar | `hu` |
| 27 | 🇷🇴 Rumeenia keel | Română | `ro` |
| 28 | 🇺🇦 Ukraina keel | Українська | `uk` |
| 29 | 🇧🇷 Brasiilia portugali keel | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Kantoni keel | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malai keel | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovaki keel | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgaaria keel | Български | `bg` |
| 34 | 🇭🇷 Horvaadi keel | Hrvatski | `hr` |
| 35 | 🇱🇹 Leedu | Lietuvių | `lt` |
| 36 | 🇱🇻 Läti | Latviešu | `lv` |
| 37 | 🇪🇪 Eesti | Eesti | `et` |
| 38 | 🇸🇮 Sloveeni | Slovenščina | `sl` |

## Kuidas keelt muuta

### Chloros-i töölaual

1. Avage rakenduse seaded
2. Minge keelevaliku menüüsse
3. Valige nimekirjast soovitud keel
4. Kasutajaliides uueneb kohe

### Chloros ja CLI rakendustes

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

Täpsema teabe saamiseks vaata [CLI dokumentatsiooni](CLI.md).

## Toetatavad keeled

Kõiki 38 keelt toetatakse täielikult järgmistes kohtades:

* **Chloros Desktop** – täielik graafilise kasutajaliidese tõlge
* **Chloros CLI** – käsurealiides ja väljundsõnumid

Python SDK API ja selle [viitedokumentatsioon](reference/sdk-reference.md) on kättesaadavad inglise keeles.

Keelte tugi tagab, et kasutajad üle kogu maailma saavad takistusteta ja tõhusalt töötada oma emakeeles.
