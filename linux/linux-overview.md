# Linux Ülevaade

Chloros 1.2.0 pakub Linux-i natiivset tuge **CLI**-ile ja**Python SDK** — peata multispektraalsete piltide töötlemise ning LATTICE-kaamera ja DAQ-valgusanduri reaalajas juhtimise — jaoks Linux-tööjaamades, serverites ja NVIDIA Jetson servaseadmetes.

{% hint style="info" %}
**Linux-il puudub töölaua graafiline kasutajaliides.**Chloros töölaua graafiline kasutajaliides on saadaval ainult Windows-il. Linux-kasutajad suhtlevad Chloros-iga [CLI](../CLI.md) ja [Python SDK](../api-python-sdk.md) kaudu. `.deb` lisab küll**Chloros CLI** kande teie rakenduste menüüsse — see lihtsalt avab terminaliemulaatori, milles töötab `chloros-cli`.
{% endhint %}

***

## Platvormide toetuse tabel

| Funktsioon | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Töölaua graafiline kasutajaliides** | Jah | Ei kohaldata | Ei | Ei |
| **CLI** (`chloros-cli`) | Jah | Jah | Jah | Jah |
| **Python SDK** (`chloros-sdk`) | Jah | Jah | Jah | Jah |
| **Pilditöötlusprotsess** | Jah | Jah | Jah | Jah |
| **LATTICE kaamerajuhtimine (reaalajas)** | Jah (vahekaart „Kaamerad”) | Jah (`chloros-cli lattice`, SDK) | Jah | Jah |
| **DAQ valgusandurid (reaalajas)** | Jah (vahekaart „Valgusandurid”) | Jah (`chloros-cli daq pool-*`, SDK) | Jah | Jah |
| **PTP-ajasünkroniseerimine (host on grandmaster)** | Jah | Jah (`chloros-cli time-sync`) | Jah | Jah |
| **GPU-kiirendus (CUDA)** | Jah | Jah | Jah | Jah (JetPack 6) |
| **Tekstuuritundlik debayer** | Jah (Chloros+) | Jah (Chloros+) | Jah (Chloros+) | Jah (Chloros+) |
| **Dünaamiline arvutusvõimsuse kohandamine** | Jah | Jah | Jah | Jah |
| **Tagapõhi süsteemiteenusena** (`chloros-backend.service`) | Ei | Ei | Jah (valikuline) | Jah (valikuline) |
| **Kohapealne uuendaja** (`chloros-cli update`) | Ei (käivita paigaldaja) | Ei (käivita paigaldaja) | Jah | Jah |***

## Toetatud arhitektuurid

| Arhitektuur | Kirjeldus | Pakett |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardseadmed: lauaarvutite ja serverite protsessorid (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-protsessorid — NVIDIA Jetson Orin-seeria | `chloros_<version>_arm64_jp6.deb` (JetPack 6 versioon) |

## Toetatud Linux distributsioonid

* **Ubuntu 22.04 LTS või uuem** (amd64)
* **Debian 12 või uuem** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin platvormid)***

## Mida saavad Linux kasutajad

* **Chloros CLI** — täielik käsurealiides partiitöötluseks, automatiseerimiseks ja skriptide kirjutamiseks
* **Chloros Python SDK** — programmeeritav Python liides teadustöö protsesside ja kohandatud tööriistade jaoks (paigaldatav PyPI-st ning lisatud ka `.deb`-i versiooniga sobiva wheel-failina)
* **LATTICE-kaamerate juhtimine** — LATTICE-kaamerate ja sünkroniseeritud mitmekaamerasüsteemide avastamine, ühendamine, konfigureerimine ja pildistamine `chloros-cli lattice` ja SDK kaudu; `.deb` sisaldab kaamerate jaoks vajalikku Arena SDK-i jooksuaega
* **DAQ-valgusandurite juhtimine** — ühenda DAQ-U/M/E andureid, edasta kalibreeritud spektreid ja salvestada `.daq`-faile `chloros-cli daq pool-*` ja SDK abil
* **PTP-ajasünkroniseerimine** — Chloros-tagapõhi käitab PTP-grandmasteri, millele LATTICE-kaamerad ja DAQ-E-andurid alluvad; kontrollige seda `chloros-cli time-sync` abil, ja hoidke seda töötamas ilma graafilise kasutajaliideseta süsteemi `chloros-backend.service` abil (vt [Linux paigaldamine](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projekti automatiseerimine** — käivita salvestatud projekte ilma kasutajaliideseta, kasutades `chloros-cli project` ja SDK-i `open_project`-i
* **GPU-kiirendus** — CUDA-kiirendatud töötlemine NVIDIA GPU-del (lauaarvutid ja Jetson)
* **Dynamic Compute Adaptation** — automaatne riistvara tuvastamine ja töötlemisstrateegia valik, kus `CHLOROS_STRATEGY` ülekirjutamine toimib eksperdi pääseteena
* **Kõik töötlemisfunktsioonid** — sama töötlemisprotsess nagu Windows-is: kalibreerimine, vinjeti korrigeerimine, taimestiku indeksid ja kõik ekspordivormingud
* **Chloros+ funktsioonid** — mitmeprotsessiline (torustatud) töötlemine, tekstuuritundlik debayer ja kohandatud indeksid, koos tasulise Chloros+ paketiga

## Mida Linux kasutajad ei saa

* **Töölaua graafiline kasutajaliides** — graafilist kasutajaliidest pole; kogu suhtlus toimub CLI või Python SDK kaudu
* **Pildivaataja** — puudub interaktiivne pildivaataja, ruudustikuvaade või kaardimärgid
* **Visuaalne projektijuhtimine** — projekte luuakse ja juhitakse CLI-käskude ja SDK-kutsete abil (riistvara ise — kaamerad, andurid, salvestamine — jääb täielikult terminali kaudu juhitavaks)***

## Litsentsinõuded

CLI ja SDK juurdepääs eeldab **tasulist Chloros+ taset — Copper või kõrgem**(Copper, Bronze, Silver, Gold). Tasuta**Iron**-tasemel puudub juurdepääs CLI/SDK-le. Seda piirangut rakendab tagapõhi, mitte ainult CLI:

| Olukord | Tagapõhja vastus |
| --- | --- |
| Sisselogimata | `401` koos `error_code: AUTH_REQUIRED` |
| Sisse logitud tasuta Iron-tasemel | `403` koos `error_code: PLAN_UPGRADE_REQUIRED`-iga |

`chloros-cli status` töötab igal tasemel — see on ainus marsruut, mis on väravast vabastatud — seega on keeldumise põhjus alati nähtav.

***

## Alustamine Linux-iga

1. **Paigalda Chloros** — vaata [Linux paigaldamine](linux-installation.md) `.deb` paigaldamiseks
2. **Kontrolli** — `chloros-cli --version` trükib välja `Chloros CLI 1.2.0`; `chloros-cli selftest` käivitab 7-astmelise diagnostika
3. **Paigalda Python ja SDK** (valikuline) — `pip install chloros-sdk`
4. **Logige sisse** — `chloros-cli login your@email.com 'your-password'` (üks kord iga seadme kohta ja uuesti pärast iga tarkvarapaketi uuendamist)
5. **Töötlege oma esimest andmekogumit** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetsoni puhul vaadake spetsiaalset [NVIDIA Jetsoni juhendit](nvidia-jetson-guide.md), milles on kirjeldatud platvormispetsiifilist seadistamist, soojusjuhtivust ja kasutuselevõttu.

***

## Järgmised sammud

* [Linux paigaldamine](linux-installation.md) — üksikasjalik paigaldusjuhend, failide asukohad ja veaotsing amd64 ja arm64 jaoks
* [NVIDIA Jetsoni juhend](nvidia-jetson-guide.md) — Jetsonile spetsiifiline seadistamine, mälu ja soojusjuhtivus, kasutuselevõtt
* [CLI : käsurea](../CLI.md) — CLI juhend
* [API : Python SDK](../api-python-sdk.md) — juhend SDK
* [CLI viide](../reference/cli-reference.md) ja [SDK viide](../reference/sdk-reference.md) — täielikud käskude/API loendid versioonile 1.2.0
* [Dünaamiline arvutusvõimsuse kohandamine](../processing-architecture/dynamic-compute-adaptation.md) — kuidas Chloros kohandub teie riistvaraga

{% hint style="info" %}
**Käesoleva kasutusjuhendi programmiline lugemine.** Iga lehekülg on kättesaadav ka töötlemata Markdown-vormingus oma URL ja `.md` (näiteks `https://mapir.gitbook.io/chloros/linux/linux-installation.md`) ning kogu kasutusjuhendi sisukord on avaldatud aadressil [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
