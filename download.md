---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Allalaadimine

Laadige alla Chloros uusim versioon, et alustada multispektraalsete piltide töötlemist.

### Süsteeminõuded

#### Windows

| Nõue          | Minimaalne                                              | Soovitatav                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operatsioonisüsteem** | Windows 10 (64-bitine)                                  | Windows 11 (64-bitine)                                  |
| **Protsessor**        | Intel Core i5 või samaväärne                          | Intel Core i7 või parem                              |
| **Mälu (RAM)**     | 8 GB                                                  | 16 GB või rohkem                                         |
| **Graafikakaart**    | DirectX 11-ga ühilduv                                | NVIDIA GPU 4 GB+ VRAM-iga                            |
| **Salvestusruum**          | 6 GB vaba ruumi                                       | SSD 10 GB+ vaba ruumiga                            |
| **Ekraan**          | 1920x1080                                            | 2560x1440 või kõrgem                                  |
| **Internet**         | Vajalik \[valikulise] Chloros+ litsentsi aktiveerimiseks | Vajalik \[valikulise] Chloros+ litsentsi aktiveerimiseks |

#### Linux amd64 (x86\_64)

| Nõuded       | Minimaalsed                    | Soovitatavad               |
| ----------------- | -------------------------- | ------------------------- |
| **Distributsioon**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Protsessor**     | x86\_64 (Intel/AMD)        | Intel Core i7 või parem   |
| **Mälu (RAM)**  | 8 GB                        | 16 GB või rohkem              |
| **Graafikakaart** | Puudub (töötleb CPU)      | NVIDIA GPU 4 GB+ VRAM-iga |
| **Salvestusruum**       | 2 GB vaba ruumi             | SSD 10 GB+ vaba ruumiga       |
| **Python**        | Python 3.7+ (SDK jaoks)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Nõue      | Minimaalne                      | Soovitatav                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platvorm**     | NVIDIA Jetson koos JetPack 6-ga | Jetson Orin NX 16 GB või AGX Orin |
| **Mälu (RAM)** | 8 GB (jagatud GPU/CPU)         | 16 GB+ jagatud                    |
| **Salvestusruum**      | 2 GB vaba ruumi               | NVMe SSD, millel on 10 GB+ vaba ruumi        |
| **Python**       | Python 3.7+ (SDK jaoks)        | Python 3.10+                    |

{% hint style="info" %}
**GPU kiirendus**: Chloros+ kasutajad, kellel on NVIDIA GPU-d, saavad kasutada CUDA kiirendust, mis tagab oluliselt kiirema töötlemise. See toimib nii Windows (lauaarvuti GPU-d) kui ka Linux (lauaarvuti GPU-d ja NVIDIA Jetson) puhul. Chloros+ kasutajad saavad ka mitmeprotsessilise töötlemise maksimaalse kiiruse saavutamiseks.
{% endhint %}

***

## Lae alla Chloros

### Viimane stabiilne versioon (23. märts 2026): versioon 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Lae alla Chloros versioonile Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Lae alla Chloros versioonile Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Lae alla Chloros versioonile Linux arm64 / Jetson (.deb)</a>

#### Windows paigaldaja (GUI + CLI + Backend)

* **Failitüüp**: .exe (Windows paigaldaja)**Paigaldamise sammud:**

1. Lae alla eespool nimetatud .exe fail
2. Alusta paigaldamist, klõpsates paigaldajal kahekordselt
3. Järgi paigaldusviisardi juhiseid
4. Valige paigalduskataloog (vaikimisi: `C:\Program Files\[USER]\Chloros\`)
5. Viige paigaldamine lõpule ja käivitage Chloros või Chloros CLI
6. Logige sisse oma [MAPIR Cloud Chloros+ kontoga](https://cloud.mapir.camera/pricing) (või jätkake tasuta versiooniga)

{% hint style="success" %}
Paigaldaja lisab automaatselt `chloros-cli` teie süsteemi PATH-i, et võimaldada juurdepääsu käsurealt.
{% endhint %}

#### Linux amd64 (.deb pakett — CLI + Backend)

* **Failitüüp**: .deb (Debian/Ubuntu pakett)
* **Arhitektuur**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb pakett — CLI + Backend)

* **Failitüüp**: .deb (JetPack 6)
* **Arhitektuur**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Vaata [Linux paigaldamist](linux/linux-installation.md) üksikasjalike seadistusjuhiste saamiseks ja [NVIDIA Jetsoni juhendit](linux/nvidia-jetson-guide.md) Jetsonile spetsiifiliste juhiste saamiseks.

#### Python SDK (kõik platvormid)

```bash
pip install chloros-sdk
```

Vaadake dokumentatsiooni [API : Python SDK](api-python-sdk.md).

{% hint style="info" %}
**Linux kasutajad**: `.deb` pakett installib CLI ja backend&#x27;i. Python SDK installitakse eraldi pip&#x27;i kaudu. Linux-ile pole graafilist kasutajaliidest – kogu suhtlus toimub CLI või SDK kaudu.
{% endhint %}

***

## Lisaressursid

### Python SDK

Arendajate ja automatiseeritud töövoogude jaoks installige Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentatsioon**: [API: Python SDK](api-python-sdk.md)**Nõuded**: Chloros peab olema installitud (Windows installija või Linux `.deb` pakett), Chloros+ litsentsi sisselogimine nõutav***

## Komplektis sisaldub

### Windows paigaldaja

* ✅ **Chloros GUI** – täisfunktsionaalne graafiline kasutajaliides
* ✅ **Chloros CLI** – käsurealiides (nõuab Chloros+ litsentsi)
* ✅ **Chloros Backend** – töötlemismootor
* ✅ **Kaameraprofiilid** – Eelkonfigureeritud MAPIR kaameramallid

### Linux .deb-pakett

* ✅ **Chloros CLI** – Käsurealiides (nõuab Chloros+ litsentsi)
* ✅ **Chloros Backend** - Töötlemismootor
* ✅ **Kaameraprofiilid** - Eelkonfigureeritud MAPIR kaameramallid
* ❌ Ei ole graafilist kasutajaliidest — Linux on ainult peata CLI/SDK

### Python SDK (pip, kõik platvormid)

* ✅ **Chloros SDK** - Python API (nõuab Chloros+ litsentsi)***

## Uuendage Chloros+ versioonile

Avage täiustatud funktsioonid Chloros+ tellimusega:

* 🚀 **Mitmeprotsessiline töötlemine** – töötlege pilte paralleelselt
* ⚡ **GPU (CUDA) kiirendus** – kasutage ära NVIDIA GPU võimsust
* 💻 **CLI juurdepääs** – automatiseerige käsurea tööriistadega
* 🐍 **Python SDK** – programmiline API juurdepääs
* 📱 **Mitmed seadmed** – Kasutage 2–10+ seadmel (sõltuvalt paketist)
* **🐻 Täiustatud tekstuuritundlik debayer-meetod** – Kvaliteetne servatundlik debayer, mis on kombineeritud AI/ML müra vähendamise mudeliga, mis eemaldab peaaegu kogu debayeringu müra.
* 🧮 **Kohandatud valemid** – Looge kohandatud multispektraalsed indeksid

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Vaata Chloros+ pakette ja hindu</a></p>***

## Paigaldamisabi

### Veaotsing

**Paigaldamine ebaõnnestub järgmise veateatega:**

* Veenduge, et teil on administraatori õigused
* Lülitage ajutiselt välja viirusetõrjeprogramm
* Kontrollige, kas teie süsteem vastab miinimumnõuetele

**Rakendus ei käivitu (Windows):**

* Veenduge, et Windows 10/11 (64-bit) on installitud
* Uuendage graafikadraiverid
* Kontrollige Windows sündmustevaatajat vea üksikasjade leidmiseks
* Võtke ühendust tugiteenistusega koos vealogidega

**CLI ei käivitu (Linux):**

* Kontrollige, kas `.deb` pakett on õigesti installitud: `dpkg -l | grep chloros`
* Kontrollige õigusi: `sudo chmod +x /usr/bin/chloros-cli`
* Käivita diagnostika: `chloros-cli selftest`
* Kontrolli puuduvaid raamatukogusid: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Litsentsi aktiveerimise probleemid:**

* Veendu, et internetiühendus on aktiivne
* Kontrolli kasutajatunnuseid aadressil [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontrollige, et tulemüür ei blokeeri Chloros
* Täpsemad juhised leiate [Chloros+ Sisselogimine](chloros+-login.md)

### Abi saamine

Vajate abi installimisel või seadistamisel?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Veebisait**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentatsioon**: [Alustamine](./)
* ❓ **KKK**: [Korduma kippuvad küsimused](faq.md)***

## Muudatuste logi

<details>

<summary>Versioon 1.1.0 (viimane)</summary>

**Väljalaske kuupäev: märts 2026**

**Uued funktsioonid*** **Linux tugi** — Põhineb CLI ja SDK-il Linux amd64 (x86\_64) ja arm64 (NVIDIA Jetson JetPack 6) jaoks. Paigaldage `.deb` pakettide kaudu.
* **NVIDIA Jetson tugi** — Optimeeritud töötlemine Jetson Nano, Orin Nano, Orin NX ja AGX Orin servaseadmetele.
* **Dünaamiline arvutusvõimsuse kohandamine** — automaatne riistvara tuvastamine ja töötlemisstrateegia optimeerimine. Chloros kohandub teie riistvaraga alates Jetson Nanost kuni mitme GPU-ga tööjaamani.
* **4-niitline töötlemispipeline** — samaaegsed tuvastamise, kalibreerimise, töötlemise ja eksportimise niidid dünaamilise GPU-mälu eraldamisega.
* **Uued CLI-käskud** — `selftest` (süsteemidiagnostika) ja `update` (Linux-i värskenduste haldus).
* **Uued CLI protsessimärgid** — `--debayer` (standard/tekstuuritundlik), `--indices` (määra indeksid), `--target` (otsi esmalt sihtalamkaustast kiirema tuvastamise jaoks).
* **Uued GUI-menüüpunktid** — Failide lisamine, kausta lisamine ja töötlemise alustamine/lõpetamine on nüüd kättesaadavad peamenüü rippmenüüst.**Parandused**

* Platvormiülese tagapõhja automaatne tuvastamine (Windows ja Linux teed)
* Täiustatud SDK `get_status()` koos iga niidi edusammude jälgimisega
* Uued SDK erandid: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Soojuse juhtimine ja adaptiivne piiramine NVIDIA Jetsonile
* Automaatne mäluhaldus koos OOM-varuvariandiga ruudustatud GPU-töötlusele

</details>

<details>

<summary>Versioon 1.0.5</summary>

**Väljalaske kuupäev: 10. veebruar 2026**

**Uued funktsioonid*** **Tekstuuritundlik debayer-meetod \[Ainult Chloros+] -** Tekstuuritundlik kasutab kõrgekvaliteedilist servatundlikku debayerit koos AI/ML müra eemaldamise mudeliga, mis eemaldab peaaegu kogu debayerimüra.
* **T4P kalibreerimise sihtmärkide tugi*** **Kiirem Chloros+ GPU-töötlus, parem mäluhaldus**

**Veaparandused*** Täiesti uus kasutajaliides (GUI), peaks nüüd töötama kõigil Windows arvutitel.

</details>

<details>

<summary>Versioon 1.0.4</summary>

**Väljalaske kuupäev: 5. jaanuar 2026**

**Uued funktsioonid*** **Pildi/metadata lüliti**: Failibrauserisse lisati lüliti, et vaadata valitud pildi metadaate tabelis pildiresti asemel
* **Pildiresti suumiliugur**: Uus kasutajaliidese liugur pisipiltide suuruse reguleerimiseks (toetab ka CTRL + hiirerattat)
* **Pildiresti ekspordiklahvid**: Ülemises reas asuvad klahvid pisipiltide vahetamiseks JPG-vormingust töödeldud eksportidesse (Targets, Reflectance, Index, LUT)
* **Kaartide vahekaart**: uus interaktiivne 2D-kaart, mis näitab piltide GPS-asukoha märgiseid
  * Toetab Google Mapsi ja ESRI kaarditükke (valib automaatselt parima kaarditeenuse vastavalt suumitaseme kättesaadavusele)
  * Pildi eelvaade kaardimärgiste peal hiirega üle libistamisel

**Veaparandused*** Parandatud tugi Chloros installimiseks mitte-ingliskeelsetele arvutitele

</details>

<details>

<summary>Versioon 1.0.3</summary>

**Väljalaske kuupäev: 20. detsember 2025**

**Uued funktsioonid*** Esmane käivitamine

**Parandused*** Esmane käivitamine

**Veaparandused*** Esmane käivitamine

**Teadaolevad probleemid*** Esmane käivitamine

</details>***

## Litsentsileping**Autoriõigusega kaitstud tarkvara** – Copyright (c) 2026 MAPIR Inc.

Lubamatu kasutamine, levitamine või muutmine on keelatud.

**Tasuta versioon**: Saadaval isiklikuks ja äriliseks kasutamiseks, funktsioonide piirangutega**Chloros+**: Tellimuspõhine litsents täiustatud funktsioonide ja ärilise kasutuselevõtu jaoks
