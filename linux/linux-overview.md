# Linuxi ülevaade

Chloros 1.3.0 pakub **CLI**-le ja**Python SDK**-le natiivset Linuxi tuge – ekraanita multispektraalsete piltide töötlemine ning LATTICE-kaamera ja DAQ-valgusandurite reaalajas juhtimine – Linuxi tööjaamades, serverites ja NVIDIA Jetson servaseadmetes.

{% hint style="info" %}
**Linuxis puudub töölaua graafiline kasutajaliides.**Chlorosi töölaua graafiline kasutajaliides on saadaval ainult Windowsis. Linuxi kasutajad suhtlevad Chlorosiga [CLI](../CLI.md) ja [Python SDK](../api-python-sdk.md) kaudu. `.deb` lisab küll teie rakenduste menüüsse**Chlorosi CLI**-kande — see avab lihtsalt terminaliemulaatori, milles töötab `chloros-cli`.
{% endhint %}

***

## Platvormide toetuse tabel

| Funktsioon | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Töölaua graafiline kasutajaliides (GUI)** | Jah | Ei kohaldata | Ei | Ei |
| **Käsurealiides (CLI)** (`chloros-cli`) | Jah | Jah | Jah | Jah |
| **Pythoni SDK** (`chloros-sdk`) | Jah | Jah | Jah | Jah |
| **Pilditöötluse töötlusketi** | Jah | Jah | Jah | Jah |
| **LATTICE kaamera juhtimine (reaalajas)** | Jah (vahekaart „Kaamerad”) | Jah (`chloros-cli lattice`, SDK) | Jah | Jah |
| **DAQ valgusandurid (reaalajas)** | Jah (vahekaart „Valgusandurid”) | Jah (`chloros-cli daq pool-*`, SDK) | Jah | Jah |
| **PTP ajasünkroniseerimine (host on grandmaster)** | Jah | Jah (`chloros-cli time-sync`) | Jah | Jah |
| **GPU kiirendus (CUDA)** | Jah | Jah | Jah | Jah (JetPack 6) |
| **Tekstuuritundlik debayer** (vajab CUDA-graafikakaarti) | Jah (Chloros+) | Jah (Chloros+) | Jah (Chloros+) | Jah (Chloros+) |
| **Dünaamiline arvutusadapteerimine** | Jah | Jah | Jah | Jah |
| **Tagapõhi süsteemiteenusena** (`chloros-backend@<user>.service`) | Ei | Ei | Jah (valikuline) | Jah (valikuline) |
| **Kohapealne uuendaja** (`chloros-cli update`) | Ei (käivita paigaldaja) | Ei (käivita paigaldaja) | Jah | Jah |

***

## Toetatud arhitektuurid

| Arhitektuur | Kirjeldus | Pakett |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardseadmete töölaua- ja serveriprotsessorid (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-protsessorid — NVIDIA Jetson Orin-seeria (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Toetatud Linuxi distributsioonid

* **Ubuntu 22.04 LTS või uuem** (amd64)
* **Debian 12 või uuem** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin platvormid)

***

## Mida Linuxi kasutajad saavad

* **Chloros CLI** — täielik käsurealiides partiitöötluseks, automatiseerimiseks ja skriptide kirjutamiseks
* **Chloros Python SDK** — programmeeritav Python-liides teadustöö protsesside ja kohandatud tööriistade jaoks, mida installib `.deb` süsteemi `python3` jaoks (versiooniga sobiv wheel virtuaalkeskkondadele (virtualenvs) asub `/usr/lib/chloros/sdk/`-is; PyPI on mõeldud masinatele, millel puudub Chloros-pakett)
* **LATTICE kaamerate juhtimine** — LATTICE-kaamerate ja sünkroniseeritud mitme kaameraga massiivide avastamine, ühendamine, konfigureerimine ja pildistamine `chloros-cli lattice` ja SDK kaudu; `.deb` sisaldab kaamerate jaoks vajalikku Arena SDK-i käivituskeskkonda
* **DAQ valgusandurite juhtimine** — ühendage DAQ-U/M/E andurid, edastage kalibreeritud spektreid ja salvestage `.daq`-faile `chloros-cli daq pool-*` ja SDK kaudu
* **PTP-aja sünkroniseerimine** — Chloros tagapõhi käitab PTP-grandmasteri, mille orjadeks on LATTICE-kaamerad ja DAQ-E-andurid; kontrollige seda `chloros-cli time-sync` abil ja hoidke seda töötamas taustal `chloros-backend@<user>.service` systemd-üksuse abil (vt [Linuxi paigaldamine](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projekti automatiseerimine** — käivita salvestatud projekte taustal töötades `chloros-cli project` ja SDK-i `open_project` abil
* **GPU-kiirendus** — CUDA-kiirendatud töötlemine NVIDIA GPU-del (amd64-platvormi lauaarvuti GPU-del ja Jetsoni GPU-l koos JetPack 6-ga)
* **Dünaamiline arvutusadapteerimine** — automaatne riistvara tuvastamine ja töötlusstrateegia valik, kus `CHLOROS_STRATEGY` ülekirjutamine toimib eksperdi pääseteena
* **Kõik töötlusfunktsioonid** — sama töötlusprotsess nagu Windowsis: kalibreerimine, vinjeti korrigeerimine, taimestiku indeksid ja kõik ekspordivormingud
* **Chloros+ funktsioonid** — mitmeprotsessiline (pipelined) töötlemine, Texture Aware debayer (CUDA graafikaprotsessoril) ja kohandatud indeksid, tasulise Chloros+ paketi korral

## Mida Linuxi kasutajad ei saa

* **Töölaua graafiline kasutajaliides** — graafilist kasutajaliidest pole; kogu suhtlus toimub CLI- või Python-SDK kaudu
* **Pildivaataja** — puudub interaktiivne pildivaataja, võrgustikvaade või kaardimärgid
* **Visuaalne projektijuhtimine** — projekte luuakse ja juhitakse CLI-käskude ja SDK-kutsete kaudu (riistvara ise — kaamerad, andurid, pildistamine — jääb täielikult terminali kaudu juhitavaks)

***

## Litsentsinõuded

CLI-le ja SDK-le juurdepääsuks on vaja **tasulist Chloros+ paketti — Copper või kõrgem**(Copper, Bronze, Silver, Gold). Tasuta**Iron**-paketil puudub juurdepääs CLI-le ja SDK-le. Piirangut rakendab backend, mitte ainult CLI:

| Olukord | Backendi vastus |
| --- | --- |
| Sisse logimata | `401` koos `error_code: AUTH_REQUIRED` |
| Sisse logitud tasuta tasemel „Iron“ | `403` koos `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` toimib igal tasemel — see on ainus marsruut, mis on väravast vabastatud — seega on keeldumise põhjus alati nähtav.

***

## Alustamine Linuxis

1. **Paigalda Chloros** — vaata [Linuxi paigaldamine](linux-installation.md) `.deb` paigaldamiseks
2. **Kontrolli** — `chloros-cli --version` kuvab `Chloros CLI 1.3.0`; `chloros-cli selftest` käivitab 7-astmelise diagnostika
3. **Pythoni SDK kontrollimine** — `.deb` installib selle: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (virtualenv-keskkonnas, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Logi sisse** — `chloros-cli login your@email.com 'your-password'` (üks kord iga masina kohta; paketi uuendused säilitavad sisselogimise andmed, paketi eemaldamine kustutab need)
5. **Töötle oma esimest andmekogumit** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetsoni puhul vaadake spetsiaalset [NVIDIA Jetsoni juhendit](nvidia-jetson-guide.md), milles on kirjeldatud platvormispetsiifilist seadistamist, soojusjuhtivust ja kasutuselevõttu.

***

## Järgmised sammud

* [Linuxi paigaldamine](linux-installation.md) — üksikasjalik paigaldusjuhend, failide asukohad ja veaotsing amd64 ja arm64 jaoks
* [NVIDIA Jetsoni juhend](nvidia-jetson-guide.md) — Jetsonile spetsiifiline seadistamine, mälu ja soojusjuhtivus, kasutuselevõtt
* [CLI: käsurea](../CLI.md) — CLI juhend
* [API: Python SDK](../api-python-sdk.md) — SDK juhend
* [CLI-viide](../reference/cli-reference.md) ja [SDK-viide](../reference/sdk-reference.md) — põhjalikud käskude/API-loendid versioonile 1.3.0
* [Dünaamiline arvutusvõimsuse kohandamine](../processing-architecture/dynamic-compute-adaptation.md) — kuidas Chloros kohandub teie riistvaraga

{% hint style="info" %}
**Käesoleva juhendi programmiline lugemine.** Iga lehekülg on kättesaadav ka toores Markdown-vormingus oma URL-i aadressil, millele on lisatud `.md` (näiteks `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), ning kogu käsiraamatu sisukord on avaldatud aadressil [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
