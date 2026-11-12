---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Allalaadimine

Laadige alla Chloros i uusim versioon, et alustada multispektraalsete piltide töötlemist.

### Süsteeminõuded

#### Windows


| Nõue          | Miinimum                                              | Soovitus                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operatsioonisüsteem** | Windows 10 (64-bitine)                                  | Windows 11 (64-bitine)                                  |
| **Protsessor**        | Intel Core i5 või samaväärne                          | Intel Core i7 või parem                              |
| **Mälu (RAM)**     | 8 GB                                                  | 16 GB või rohkem                                         |
| **Graafikakaart**    | DirectX 11-ga ühilduv                                | NVIDIA GPU, millel on vähemalt 4 GB VRAM-i                            |
| **Salvestusruum**          | 6 GB vaba ruumi                                       | SSD, millel on vähemalt 10 GB vaba ruumi                            |
| **Ekraan**          | 1920x1080                                            | 2560x1440 või suurem                                  |
| **Internet**         | Vajalik \[valikulise] Chloros+ litsentsi aktiveerimiseks | Vajalik \[valikulise] Chloros+ litsentsi aktiveerimiseks |

#### Linux amd64 (x86\_64)

| Nõuded       | Miinimum                    | Soovituslik               |
| ----------------- | -------------------------- | ------------------------- |
| **Distributsioon**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Protsessor**     | x86\_64 (Intel/AMD)        | Intel Core i7 või parem   |
| **Mälu (RAM)**  | 8 GB                        | 16 GB või rohkem              |
| **Graafikakaart** | Puudub (töötleb CPU)      | NVIDIA GPU, millel on vähemalt 4 GB VRAM |
| **Salvestusruum**       | 2 GB vaba ruumi             | SSD, millel on vähemalt 10 GB vaba ruumi       |
| **Python**        | Python 3.7+ (SDK jaoks)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Nõuded      | Miinimum                      | Soovituslik                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platvorm**     | NVIDIA Jetson koos JetPack 6-ga | Jetson Orin NX 16 GB või AGX Orin |
| **Mälu (RAM)** | 8 GB (jagatud GPU/CPU)         | 16 GB+ jagatud                    |
| **Salvestusruum**      | 2 GB vaba ruumi               | NVMe SSD, millel on vähemalt 10 GB vaba ruumi        |
| **Python**       | Python 3.7+ (SDK jaoks)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-kiirendus**: NVIDIA graafikakaartidega Chloros+ kasutajad saavad kasutada CUDA-kiirendust, mis tagab oluliselt kiirema töötlemise. See toimib NVIDIA lauaarvuti graafikakaartidega Windowsis ja Linuxis (amd64) ning NVIDIA Jetsonil koos JetPack 6-ga. Chloros+ kasutajad saavad ka mitmeprotsessilise töötlemise, mis tagab maksimaalse kiiruse.
{% endhint %}

***

## Lae alla Chloros


### Viimane stabiilne versioon: versioon 1.3.0 (11. november 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Lae alla Chloros Windowsile (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Lae alla Chloros Linuxile amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Lae alla Chloros Linuxi arm64 / Jetson jaoks (.deb)</a>


#### Windows i installija (GUI + CLI + Backend)

* **Failitüüp**: .exe (Windows i installija)

**Paigaldamise juhised:**

1. Lae alla eespool nimetatud fail .exe
2. Alusta paigaldamist, klõpsates paigaldusprogrammi ikoonil topeltklõpsuga
3. Järgi paigaldusviisardi juhiseid
4. Vali paigalduskataloog (vaikimisi: `C:\Program Files\Chloros\`)
5. Lehel **Valige lisategevused**jäta mõlemad**Süsteemi seaded** valikud märgistatuks (soovitatav – vaata allpool)
6. Viia paigaldamine lõpule ja käivita Chloros või Chloros CLI
7. Logi sisse oma [MAPIR Cloud Chloros+ kontoga](https://cloud.mapir.camera/pricing) (või jätka tasuta versiooniga)

**Süsteemiseadete valikud** (mõlemad on vaikimisi märgitud):

| Valik, nagu see on sõnastatud paigaldajas | Mida see teeb | Vaikimisi paigaldamise tühistamine |
| --- | --- | --- |
| **Luba Windows i pikad teed** (võimaldab projektikausta asuda sügavamal kui 260 tähemärki; soovitatav) | Seab Windows i kogu arvutit hõlmava `LongPathsEnabled`-seadistuse väärtuseks 1. Windowsis on see vaikimisi välja lülitatud, mistõttu ükski failitee ei tohi olla pikem kui 260 tähemärki; Chloros i tooted asuvad kuni umbes 130 tähemärki projektikausta allpool, seega pikkade teede väljalülitamisel ei lubata sügavamat projektikausta. Jõustub pärast installimist käivitatud programmide puhul. See jääb sisse ka pärast desinstallimist, kuna teised programmid võivad sellele tugineda. | `/MERGETASKS="!longpaths"` |
| **Jäta Chloros i taustaprogramm Microsoft Defenderi skaneerimisest välja** (palju kiirem kaamerapildistamine; soovitatav) | Lisab installikausta asuvale `chloros-backend.exe`-ile Microsoft Defenderi protsessi välistuse, nii et reaalajas kaitse ei skaneeri iga faili, mida taustaprogramm kirjutab, olenemata sellest, kuhu ta selle kirjutab. Kõrgel pildistamissagedusel moodustas skannimine umbes 30% pildistamissagedusest (77,7 vs 52,7 pildistust sekundis 9 kaameraga pideva pildistamise korral). Ühtegi teist programmi ei jäeta välja ning välistamine eemaldatakse desinstallimisel. Kui Defender puudub või on poliitika tõttu lukustatud, ebaõnnestub see samm märkamatult ja installimine jätkub. | `/MERGETASKS="!defenderexclusion"` |

Vaikne paigaldamine (`/SILENT` või `/VERYSILENT`) rakendab mõlemat valikut, kui te ei loobu sellest näidatud väärtusega `/MERGETASKS`.

{% hint style="success" %}
Paigaldaja lisab teie süsteemi kord CLI-kausta (`C:\Program Files\Chloros\cli\`) – uuesti paigaldamisel või uuendamisel ei lisata enam dubleeritud kirjet ning eemaldatakse varasemate paigaldajate poolt lisatud dubleeritud kirjed. Avage pärast installimist uus terminal, et kasutada `chloros-cli`-i. Installija lisab ka kaks Windows i tulemüüri reeglit tagapõhja jaoks igasse võrguprofiili: **Chloros Backend (UDP-In)**, kaameravoogude, PTP, mDNS ja DAQ-E andmevoo jaoks, ning**Chloros Backend (TCP-In)**, mis on piiratud kohaliku alamvõrguga ja mõeldud DAQ-E püsivara uuendamiseks, mille käigus andur ühendub tagasi selle arvutiga pildi kogumiseks. Mõlemad eemaldatakse desinstallimisel.
{% endhint %}

#### Linux amd64 (.deb-pakett — CLI + backend)

* **Failitüüp**: .deb (Debian/Ubuntu pakett)
* **Arhitektuur**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (.deb-pakett — CLI + backend)

* **Failitüüp**: .deb (JetPack 6)
* **Arhitektuur**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Kasutage tegelikult alla laetud failinime ja säilitage `./`, et `apt` installiks just selle faili, selle asemel et otsida paketti nime järgi. Installige `apt install`-iga, mitte `dpkg -i`: apt installib ka Python i SDK nõuded, mida pakett soovitab, mitte ei nõua, ning `dpkg -i` jätab need vahele. Alates versioonist 1.3.0 paigaldatakse backend ja CLI lahtipakkimata `/usr/lib/chloros/` alla, nii et need käivituvad ilma end esmalt lahtipakkimata.

Jetsoni pakett kasutab JetPack 6 CUDA-versiooni PyTorchist, mistõttu töötlemine, sealhulgas kaasasolev Texture Aware debayer, toimub Jetsoni GPU-l. See kasutab CUDA ja cuDNN raamatukogusid, mille JetPack 6 paigaldab (`sudo apt install nvidia-jetpack` pildil, millel need puuduvad).

Vaata [Linux i paigaldusjuhendit](linux/linux-installation.md) üksikasjalike seadistamisjuhiste saamiseks ja [NVIDIA Jetsoni juhendit](linux/nvidia-jetson-guide.md) Jetsonile spetsiifiliste juhiste saamiseks.

#### Python i SDK (kõik platvormid)

Iga paigaldaja sisaldab sobivat `chloros_sdk`-ratast, seega vastab SDK-versioon alati paigaldatud GUI-le/CLI-le/tagapõhjale. Windowsis paigaldab paigaldaja selle automaatselt teie süsteemi Python isse. Linuxis installib `.deb` SDK süsteemi `python3` jaoks (kataloogi `/usr/lib/python3/dist-packages`), seega töötab `import chloros_sdk` kohe pärast paigaldamist, ilma pip-ita ja võrguta. Ainult virtualenv, mis ei näe süsteemi pakette, vajab komplektis oleva wheeli paigaldamist:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Ainult pip-i kasutavate hostide puhul (Chloros-pakett pole paigaldatud; SDK juhib sel juhul teisel masinal asuvat backendit) on SDK saadaval ka PyPI-s:

```bash
pip install chloros-sdk
```

Vaata dokumentatsiooni [API: Python SDK](api-python-sdk.md) ja [SDK viite](reference/sdk-reference.md).

{% hint style="info" %}
**Linux i kasutajad**: Pakett `.deb` installib CLI, backend-mooduli ja Python i SDK. Linux i jaoks ei ole graafilist kasutajaliidest – kogu suhtlus toimub CLI või SDK kaudu.
{% endhint %}

***

## Lisaressursid

### Python i SDK


Arendajate ja automatiseeritud töövoogude jaoks on Chloros i Python i SDK kaasas igas paigaldajas: Windows i paigaldaja installib selle teie Python isse ning Linux i `.deb` installib selle süsteemi `python3` jaoks. Kui arvutis puudub Chloros-pakett (st kui see juhib teise arvuti tagapõhja), paigalda see PyPI-st:

```bash
pip install chloros-sdk
```

**Dokumentatsioon**: [API: Python SDK](api-python-sdk.md)

**Nõuded**: juhitav Chloros i backend (Windows i installija või Linux i `.deb` pakett installib selle) ning Chloros+ sisselogimine tasulises paketis

***

## Mis on kaasas

### Windows i installija

* ✅ **Chloros GUI** – täisfunktsionaalne graafiline kasutajaliides
* ✅ **Chloros CLI** – käsurealiides (nõuab Chloros+ litsentsi)
* ✅ **Chloros backend** – töötlemismootor
* ✅ **Python SDK** – paigaldatakse teie Python isse, kui see on olemas (nõuab Chloros+ litsentsi)
* ✅ **Kaameraprofiilid** – eelkonfigureeritud MAPIR-kaameramallid

### Linux i .deb-pakett

* ✅ **Chloros CLI** – käsurealiides (nõuab Chloros+ litsentsi)
* ✅ **Chloros Backend** – töötlemismootor
* ✅ **Python SDK** – paigaldatakse süsteemile `python3` (nõuab Chloros+ litsentsi)
* ✅ **Kaameraprofiilid** – eelkonfigureeritud MAPIR-kaameramallid
* ❌ Graafilist kasutajaliidest pole – Linux on ainult CLI/SDK-põhine

### Python-SDK (PyPI, masinatele, millel puudub Chloros-pakett)

* ✅ **Chloros-SDK** – Python i API (nõuab Chloros+ litsentsi)

***

## Uuendage Chloros+ versioonile

Avage täiustatud funktsioonid Chloros+ tellimusega:

* 🚀 **Mitmeprotsessiline töötlemine** – töötlege pilte paralleelselt
* ⚡ **GPU (CUDA) kiirendus** – Kasuta ära NVIDIA GPU võimsust
* 💻 **Juurdepääs CLI-le** – Automatiseeri käsurea tööriistadega
* 🐍 **Python SDK** – Programmeeritav API-juurdepääs
* 📱 **Mitmed seadmed** – Kasuta 2–10+ seadmel (sõltuvalt paketist)
* **🐻 Täiustatud tekstuuritundlik debayering-meetod** – kõrgekvaliteediline servatundlik debayering, mis on kombineeritud AI/ML-müra eemaldamise mudeliga, mis kõrvaldab peaaegu kogu debayeringust tuleneva müra.
* 🧮 **Kohandatud valemid** – loo kohandatud multispektraalsed indeksid

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Vaata Chloros+ pakette ja hindu</a></p>

***

## Paigaldamisabi

### Veaotsing

**Paigaldamine ebaõnnestub järgmise veateatega:**

* Veendu, et sul on administraatoriõigused
* Lülita ajutiselt välja viirusetõrjeprogramm
* Kontrolli, kas su arvuti vastab miinimumsüsteeminõuetele

**Rakendus ei käivitu (Windows):**

* Veendu, et Windows 10/11 (64-bitine) on installitud
* Uuenda graafikakaartide draivereid
* Kontrolli Windows i sündmustevaatajast vea üksikasju
* Võta ühendust tugiteenistusega ja edasta vealogid

**CLI ei käivitu (Linux):**

* Veenduge, et pakett `.deb` on õigesti paigaldatud: `dpkg -l | grep chloros`
* Kontrollige kasutusõigusi: `sudo chmod +x /usr/bin/chloros-cli`
* Käivitage diagnostika: `chloros-cli selftest`
* Kontrollige, kas puuduvad mõned teegid: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (alates versioonist 1.3.0 on `/usr/lib/chloros/chloros-backend` väike käivitusskript; programm ise asub failis `/usr/lib/chloros/backend/`)

**Litsentsi aktiveerimisega seotud probleemid:**

* Veendu, et internetiühendus on aktiivne
* Kontrolli kasutajatunnuseid aadressil [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontrollige, et tulemüür ei blokeeri Chloros'i* Vaadake üksikasjalikke juhiseid lehelt [Chloros+ sisselogimine](chloros+-login.md)

### Abi saamine

Vajate abi paigaldamise või seadistamisega?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Veebisait**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentatsioon**: [Alustamine](./)
* ❓ **KKK**: [Korduma kippuvad küsimused](faq.md)

***

## Tarkvarauuendused

Chloros kontrollib uuenduste olemasolu, teavitab uue versiooni kättesaadavusest ja suunab selle allalaadimislehele — uuendamiseks käivita uus allkirjastatud paigaldusprogramm. Sinu seaded, projektid ja sisselogimise andmed säilivad uuenduste järel. Linuxis ja Jetsonis kontrollib `chloros-cli update` uuema versiooni olemasolu ning pakub vastava `.deb` allalaadimist ja installimist (see käsk on saadaval ainult Linuxis).

***

## Muudatuste loend**Versioon 1.3.0 (viimane, 11. november 2026)**— täieliku loendi leiate jaotisest**„Chloros

i 1.3.0 uuendused”** lehel [Alustamine](./).**Versioon 1.2.0**— vaata**Chloros i 1.2.0 uuendused** leheküljel [Alustamine](./).

<details>

<summary>Versioon 1.0.5</summary>

**Väljalaskekuupäev: 10. veebruar 2026**

**Uued funktsioonid*** **Tekstuuritundlik debayer-meetod \[Ainult Chloros+] –** Tekstuuritundlik meetod kasutab kõrgekvaliteedilist servatundlikku debayerit koos AI/ML-müra vähendamise mudeliga, mis eemaldab peaaegu kogu debayerimisel tekkiva müra.
* **T4P-kalibreerimismärkide tugi*** **Kiirem „Chloros+“ GPU-töötlus, parem mäluhaldus**

**Veaparandused*** Täiesti uus kasutajaliides (GUI), peaks nüüd töötama kõigil Windows-arvutitel.

</details>

<details>

<summary>Versioon 1.0.4</summary>

**Väljalaskekuupäev: 5. jaanuar 2026**

**Uued funktsioonid*** **Pildi/metadata lüliti**: failibrauserisse on lisatud lüliti, millega saab valitud pildi metadaate vaadata tabelis, mitte pildirestis
* **Pildiresti suumiliugur**: uus kasutajaliidese liugur pisipiltide suuruse reguleerimiseks (toetab ka klahvikombinatsiooni CTRL + hiirerattaga)
* **Pildiresti ekspordiknopid**: Ülemises reas asuvad nupud, millega saab pisipilte vahetada JPG-vormingust töödeldud ekspordivorminguteks (Targets, Reflectance, Index, LUT)
* **Kaart-vahekaart**: Uus interaktiivne 2D-kaart, mis kuvab piltide GPS-asukohamärke
  * Toetab Google Mapsi ja ESRI kaarditükke (valib automaatselt parima kaarditeenuse vastavalt suumitaseme kättesaadavusele)
  * Pildi pisipildi eelvaade kaardimärkide kohal hiirega üle libistamisel

**Veaparandused*** Parandatud tugi Chloros installimiseks mitte-ingliskeelsetele arvutitele

</details>

<details>

<summary>Versioon 1.0.3</summary>

**Väljalaskekuupäev: 20. detsember 2025**

**Uued funktsioonid*** Esmane käivitamine

**Parandused*** Esmane käivitamine

**Veaparandused*** Esmane käivitamine

**Teadaolevad probleemid*** Esmane käivitamine

</details>

***

## Litsentsileping**Autoriõigusega kaitstud tarkvara** – Copyright (c) 2026 MAPIR Inc.

Lubamatu kasutamine, levitamine või muutmine on keelatud.

**Tasuta versioon**: Saadaval isiklikuks ja äriliseks kasutamiseks, funktsioonide piirangutega**Chloros+**: Tellimuspõhine litsents täiustatud funktsioonide ja ärilise kasutuselevõtu jaoks
