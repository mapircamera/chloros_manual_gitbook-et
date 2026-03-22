# Linux ülevaade

Chloros versioon 1.1.0 toetab nüüd natiivselt **CLI**ja**Python SDK**, võimaldades peata multispektraalset pilditöötlust Linux tööjaamades, serverites ja NVIDIA Jetson servaseadmetes.

{% hint style="info" %}
**Linux-il puudub graafiline kasutajaliides.** Chloros töölaua graafiline kasutajaliides on saadaval ainult Windows-il. Linux kasutajad suhtlevad Chloros-iga läbi [CLI](../CLI.md) ja [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Platvormide toetuse maatriks

| Funktsioon | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Töölaua graafiline kasutajaliides** | Jah | Ei kohaldata | Ei | Ei |
| **CLI** | Jah | Jah | Jah | Jah |
| **Python SDK** | Jah | Jah | Jah | Jah |
| **GPU kiirendus (CUDA)** | Jah | Jah | Jah | Jah (JetPack 6) |
| **Tekstuuritundlik debayer** | Jah (Chloros+) | Jah (Chloros+) | Jah (Chloros+) | Jah (Chloros+) |
| **Dünaamiline arvutusadapteerimine** | Jah | Jah | Jah | Jah |***

## Toetatud arhitektuurid

| Arhitektuur | Kirjeldus | Paigaldusmeetod |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardne lauaarvuti/serveri protsessor (Intel, AMD) | `.deb` pakett |
| **arm64 (aarch64)** | ARM-põhised protsessorid, peamiselt NVIDIA Jetson | `.deb` pakett (JetPack 6) |

## Toetatud Linux distributsioonid

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson-platvormid)***

## Mida Linux kasutajad saavad

* **Chloros CLI** — Täielik käsurealiides partiitöötluseks, automatiseerimiseks ja skriptimiseks
* **Chloros Python SDK** — Programmeeritav Python liides (`pip install chloros-sdk`) integreerimiseks uurimistöö voogudesse ja kohandatud tööriistadesse
* **GPU kiirendus** — CUDA-kiirendatud töötlemine NVIDIA GPU-del (lauaarvutid ja Jetson)
* **Dünaamiline arvutusadapteerimine** — Automaatne riistvara tuvastamine ja töötlemisstrateegia optimeerimine
* **Kõik töötlemisfunktsioonid** — Sama multispektraalne töötlemisprotsess nagu Windows (kalibreerimine, vinjettide korrigeerimine, taimestiku indeksid, kõik ekspordivormingud)
* **Chloros+ funktsioonid** — mitmeprotsessiline töötlemine, tekstuuritundlik debayer, kohandatud indeksid (koos Chloros+ litsentsiga)

## Mida Linux kasutajad ei saa

* **Töölaua graafiline kasutajaliides** — Graafilist kasutajaliidest pole; kogu suhtlus toimub CLI või Python SDK kaudu
* **Pildivaataja** — puudub interaktiivne pildivaataja, ruudustikvaade või kaardimärgid
* **Visuaalne projektijuhtimine** — projekte hallatakse CLI-käskude ja SDK-kutsete kaudu***

## Alustamine Linux-iga

1. **Chlorosi installimine** — `.deb`i paketi installimiseks vaadake [Linuxi installimist](linux-installation.md)
2. **Paigalda Python SDK** (valikuline) — `pip install chloros-sdk`
3. **Aktiveeri oma litsents** — `chloros-cli login your@email.com 'password'`
4. **Töötle oma esimest andmekogumit** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetsoni kasutajad leiavad platvormispetsiifilise seadistamise ja optimeerimise juhised spetsiaalsest [NVIDIA Jetsoni juhendist](nvidia-jetson-guide.md).

***

## Järgmised sammud

* [Linux paigaldamine](linux-installation.md) — üksikasjalikud paigaldusjuhised amd64 ja arm64 jaoks
* [NVIDIA Jetsoni juhend](nvidia-jetson-guide.md) — Jetsonile spetsiifiline seadistamine, soojusjuhtimine ja kasutuselevõtt
* [CLI : käsurea](../CLI.md) — Täielik CLI viide
* [API : Python SDK](../api-python-sdk.md) — Täielik SDK viide
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — Kuidas Chloros kohandub teie riistvaraga
