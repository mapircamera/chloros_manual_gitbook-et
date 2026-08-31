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
| **Graafikakaart**    | DirectX 11-ga ühilduv                                | NVIDIA GPU, millel on 4 GB või rohkem VRAM-i                            |
| **Salvestusruum**          | 6 GB vaba ruumi                                       | SSD, millel on vähemalt 10 GB vaba ruumi                            |
| **Ekraan**          | 1920x1080                                            | 2560x1440 või suurem                                  |
| **Internet**         | Vajalik \[valikulise] Chloros+ litsentsi aktiveerimiseks | Vajalik \[valikulise] Chloros+ litsentsi aktiveerimiseks |

#### Linux amd64 (x86\_64)

| Nõue       | Miinimum                    | Soovituslik               |
| ----------------- | -------------------------- | ------------------------- |
| **Distributsioon**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Protsessor**     | x86_64 (Intel/AMD)        | Intel Core i7 või parem   |
| **Mälu (RAM)**  | 8 GB                        | 16 GB või rohkem              |
| **Graafikakaart** | Puudub (töötleb protsessor)      | NVIDIA GPU 4 GB+ VRAM-iga |
| **Salvestusruum**       | 2 GB vaba ruumi             | SSD, millel on vähemalt 10 GB vaba ruumi       |
| **Python**        | Python 3.7+ (SDK jaoks)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Nõuded      | Minimaalsed                      | Soovitatavad                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platvorm**     | NVIDIA Jetson koos JetPack 6-ga | Jetson Orin NX 16 GB või AGX Orin |
| **Mälu (RAM)** | 8 GB (jagatud GPU/CPU)         | 16 GB+ jagatud                    |
| **Salvestusruum**      | 2 GB vaba ruumi               | NVMe SSD, millel on 10 GB+ vaba ruumi        |
| **Python**       | Python 3.7+ (mudelile SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-kiirendus**: Chloros+ kasutajad, kellel on NVIDIA GPU-d, saavad kasutada CUDA kiirendust, mis tagab märkimisväärselt kiirema töötlemise. See toimib nii Windows (lauaarvuti GPU-d) kui ka Linux (lauaarvuti GPU-d ja NVIDIA Jetson) puhul. Chloros+ kasutajad saavad lisaks mitmeprotsessilise töötlemise, mis tagab maksimaalse kiiruse.
{% endhint %}

***

## Lae alla Chloros

### Viimane stabiilne versioon: versioon 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Lae alla Chloros mudelile Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Lae alla Chloros versioonile Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Lae alla Chloros versioonile Linux arm64 / Jetson (.deb)</a>

#### Windows paigaldaja (GUI + CLI + tagapõhi)

* **Failitüüp**: .exe (Windows paigaldaja)**Paigaldamise juhised:**

1. Laadige alla eespool nimetatud .exe-fail
2. Alustage paigaldamist, klõpsates paigaldusprogrammi ikoonil topeltklõpsuga
3. Järgige paigaldusviisardi juhiseid
4. Valige paigalduskataloog (vaikimisi: `C:\Program Files\MAPIR\Chloros\`)
5. Viige paigaldamine lõpule ja käivitage Chloros või Chloros CLI
6. Logige sisse oma [MAPIR Cloud Chloros+ kontoga](https://cloud.mapir.camera/pricing) (või jätkake tasuta versiooniga)

{% hint style="success" %}
Paigaldaja lisab automaatselt `chloros-cli` teie süsteemi PATH-i, et võimaldada juurdepääsu käsurealt.
{% endhint %}

#### Linux amd64 (.deb-pakett — CLI + tagapõhi)

* **Failitüüp**: .deb (Debian/Ubuntu pakett)
* **Arhitektuur**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb-pakett — CLI + Backend)

* **Failitüüp**: .deb (JetPack 6)
* **Arhitektuur**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Vaata [Linux paigaldusjuhendit](linux/linux-installation.md) üksikasjalike seadistamisjuhiste saamiseks ja [NVIDIA Jetsoni juhendit](linux/nvidia-jetson-guide.md) Jetsonile spetsiifiliste juhiste saamiseks.

#### Python SDK (kõik platvormid)

Iga paigaldaja sisaldab sobivat `chloros_sdk`-ratast, seega vastab SDK-versioon alati paigaldatud GUI-le/CLI-ile/tagapõhjale. Windows-i puhul installib paigaldaja selle automaatselt teie süsteemi Python-i; versioonil Linux paigutab `.deb` ratta kataloogi `/usr/lib/chloros/sdk/` ja kuvab paigalduskäsu:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Ainult pip-i kasutavate hostide puhul (kus pole paigaldatud Chloros paketti) on SDK saadaval ka PyPI-s:

```bash
pip install chloros-sdk
```

Vaata [API : Python SDK](api-python-sdk.md) ja [SDK viite](reference/sdk-reference.md) dokumentatsiooni jaoks.

{% hint style="info" %}
**Linux kasutajad**: Pakett `.deb` installib CLI ja tagapõhja. Linux-ile ei ole graafilist kasutajaliidest – kogu suhtlus toimub CLI või SDK kaudu.
{% endhint %}

***

## Lisamaterjalid

### Python SDK

Arendajate ja automatiseeritud töövoogude jaoks installige Chloros, Python ja SDK:

```bash
pip install chloros-sdk
```

**Dokumentatsioon**: [API: Python SDK](api-python-sdk.md)**Nõuded**: Chloros peab olema installitud (Windows installija või Linux `.deb` pakett), nõutav on Chloros+ litsentsi sisselogimine***

## Komplektis sisaldub

### Windows paigaldusprogramm

* ✅ **Chloros GUI** – täisfunktsionaalne graafiline kasutajaliides
* ✅ **Chloros CLI** – käsurealiides (nõuab Chloros+ litsentsi)
* ✅ **Chloros Backend** – töötlemismootor
* ✅ **Kaameraprofiilid** – Eelkonfigureeritud MAPIR kaameramallid

### Linux .deb-pakett

* ✅ **Chloros CLI** – käsurealiides (nõuab Chloros+ litsentsi)
* ✅ **Chloros Backend** – töötlemismootor
* ✅ **Kaameraprofiilid** – eelkonfigureeritud MAPIR kaameramallid
* ❌ Graafilist kasutajaliidest pole — Linux on ainult peata CLI/SDK

### Python SDK (PIP, kõik platvormid)

* ✅ **Chloros SDK** – Python API (nõuab Chloros+ litsentsi)***

## Uuendage versioonile Chloros+

Avage täiustatud funktsioonid Chloros+ tellimusega:

* 🚀 **Mitmeprotsessiline töötlemine** – töötle pilte paralleelselt
* ⚡ **GPU (CUDA) kiirendus** – kasuta ära NVIDIA GPU võimsust
* 💻 **CLI-juurdepääs** – automatiseerige käsurea tööriistadega
* 🐍 **Python SDK** - Programmeeritud juurdepääs API-ile
* 📱 **Mitmed seadmed** – Kasutamine 2–10+ seadmel (sõltub paketist)
* **🐻 Täiustatud tekstuuritundlik debayer-meetod** – kõrgekvaliteediline servatundlik debayer, mis on kombineeritud AI/ML müra eemaldamise mudeliga, mis eemaldab peaaegu kogu debayerimisel tekkiva müra.
* 🧮 **Kohandatud valemid** – loo kohandatud multispektraalsed indeksid

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Vaata Chloros+ pakette ja hindu</a></p>***

## Paigaldamisabi

### Veaotsing

**Paigaldamine ebaõnnestub järgmise veateatega:**

* Veendu, et sul on administraatoriõigused
* Lülita ajutiselt välja viirusetõrjeprogramm
* Kontrolli, kas su arvuti vastab miinimumsüsteeminõuetele

**Rakendus ei käivitu (Windows):**

* Veenduge, et Windows 10/11 (64-bit) on installitud
* Uuendage graafikakaartide draivereid
* Kontrollige veateateid Windows sündmustevaatajas
* Võtke ühendust tugiteenistusega ja edastage vealogid

**CLI ei käivitu (Linux):**

* Veenduge, et `.deb` pakett on õigesti paigaldatud: `dpkg -l | grep chloros`
* Kontrollige õigusi: `sudo chmod +x /usr/bin/chloros-cli`
* Käivitage diagnostika: `chloros-cli selftest`
* Kontrollige, kas puuduvad mõned teegid: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Litsentsi aktiveerimisega seotud probleemid:**

* Veenduge, et internetiühendus on aktiivne
* Kontrollige kasutajatunnuseid aadressil [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontrollige, et tulemüür ei blokeeri Chloros
* Vaata üksikasjalikke juhiseid leheküljelt [Chloros+ Sisselogimine](chloros+-login.md)

### Abi saamine

Vajad abi paigaldamise või seadistamisega?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Veebisait**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentatsioon**: [Esimesed sammud](./)
* ❓ **KKK**: [Korduma kippuvad küsimused](faq.md)***

## Tarkvara uuendused

Chloros kontrollib uuenduste olemasolu, teavitab uue versiooni kättesaadavusest ja suunab selle allalaadimislehele — uuendamiseks tuleb käivitada uus allkirjastatud installija. Teie seaded ja projektid säilivad uuenduste järel. Linux-is ja Jetsonil kontrollib `chloros-cli update` uuema versiooni olemasolu ning pakub alla laadida ja installida vastava `.deb` (see käsk on saadaval ainult versioonis Linux).

***

## Muudatuste ajalugu**Versioon 1.2.0 (uusim)**— täieliku funktsioonide loendi leiate lehelt [Alustamine](./) jaotises**Mis on uut Chloros 1.2.0-s**.

<details>

<summary>Versioon 1.0.5</summary>

**Väljalaskekuupäev: 10. veebruar 2026**

**Uued funktsioonid*** **Tekstuuritundlik debayer-meetod \[Ainult Chloros+] –** Tekstuuritundlik meetod kasutab kõrgekvaliteedilist servatundlikku debayerit koos AI/ML-müra vähendamise mudeliga, mis eemaldab peaaegu kogu debayerimisel tekkiva müra.
* **T4P-kalibreerimismärkide tugi*** **Kiirem Chloros+ GPU-töötlus, parem mäluhaldus**

**Veaparandused*** Täiesti uus kasutajaliides (GUI), peaks nüüd töötama kõigil Windows arvutitel.

</details>

<details>

<summary>Versioon 1.0.4</summary>

**Väljalasku kuupäev: 5. jaanuar 2026**

**Uued funktsioonid*** **Pildi/metadata lüliti**: Failibrauserisse on lisatud lüliti, millega saab vaadata valitud pildi metadaate tabelis pildiresti asemel
* **Pildiresti suumiliugur**: Uus kasutajaliidese liugur pisipiltide suuruse reguleerimiseks (toetab ka klahvikombinatsiooni CTRL + hiirerattaga)
* **Pildiresti ekspordiklahvid**: Ülemises reas asuvad nupud, millega saab pisipilte vahetada JPG-vormingust töödeldud eksportvorminguteks (Targets, Reflectance, Index, LUT)
* **Kaart-vahekaart**: Uus interaktiivne 2D-kaart, mis kuvab piltide GPS-asukohamärke
  * Toetab Google Mapsi ja ESRI kaarditahvleid (valib automaatselt parima kaarditeenuse vastavalt suumitaseme kättesaadavusele)
  * Pildi pisipildi eelvaade kaardimärkide kohal hiirega üle libistamisel

**Veaparandused*** Parandatud toetus Chloros installimiseks mitte-ingliskeelsetele arvutitele

</details>

<details>

<summary>Versioon 1.0.3</summary>

**Väljalaskekuupäev: 20. detsember 2025**

**Uued funktsioonid*** Esmane käivitamine

**Parandused*** Esmane käivitamine

**Veaparandused*** Esmane käivitamine

**Teadaolevad probleemid*** Esmane käivitamine

</details>***

## Litsentsileping**Autoriõigusega kaitstud tarkvara** – Autoriõigus (c) 2026 MAPIR Inc.

Lubamatu kasutamine, levitamine või muutmine on keelatud.

**Tasuta versioon**: Saadaval isiklikuks ja äriliseks kasutamiseks, funktsioonide piirangutega**Chloros+**: Tellimuspõhine litsents täiustatud funktsioonide ja ärilise kasutuselevõtu jaoks
