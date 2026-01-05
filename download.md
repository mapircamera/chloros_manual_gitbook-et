---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Allalaadimine

Laadige alla Chloros uusim versioon, et alustada multispektraalsete piltide töötlemist.

### Süsteeminõuded

| Nõue          | Minimaalne                         | Soovitatav                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Operatsioonisüsteem** | Windows 10 (64-bitine)             | Windows 11 (64-bitine)             |
| **Protsessor**        | Intel Core i5 või samaväärne     | Intel Core i7 või parem         |
| **Mälu (RAM)**     | 8 GB                             | 16 GB või rohkem                    |
| **Graafikakaart**    | DirectX 11 ühilduv           | NVIDIA GPU 4 GB+ VRAM-iga       |
| **Salvestusruum**          | 6 GB vaba ruumi                  | SSD 10 GB+ vaba ruumiga       |
| **Ekraan**          | 1920x1080                       | 2560x1440 või suurem             |
| **Internet**         | Vajalik litsentsi aktiveerimiseks | Vajalik litsentsi aktiveerimiseks |

{% vihje style=&quot;info&quot; %}
**GPU kiirendus**: Chloros+ kasutajad, kellel on NVIDIA GPU (4 GB+ VRAM), saavad kasutada CUDA kiirendust, et saavutada oluliselt kiirem töötlemine. Chloros+ kasutajad saavad ka mitmeprotsessilise töötlemise, et saavutada maksimaalne kiirus.
{% endhint %}

***

## Lae alla Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Lae alla Chloros siit</a>

### Viimane stabiilne versioon

**Chloros paigaldaja Windows jaoks*** **Versioon**: 1.0.4
* **Väljalaske kuupäev**: 5. jaanuar 2026
* **Faili suurus (allalaadimine)**: 1,8 GB
* **Faili suurus (paigaldatud)**: 5,7 GB
* **Failitüüp**: .exe (Windows paigaldaja)

#### **Paigaldamise sammud:**

1. Lae alla fail `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Paigaldamise alustamiseks topeltklõpsa paigaldusprogrammil
3. Järgi paigaldusviisardi juhiseid
4. Vali paigalduskataloog (vaikimisi: `C:\Program Files\[USER]\Chloros\`)
5. Viige installimine lõpule ja käivitage Chloros, Chloros (brauser) või Chloros CLI
6. Logige sisse oma [MAPIR Cloud Chloros+ kontoga](https://cloud.mapir.camera/pricing) (või jätkake tasuta versiooniga)

{% vihje style=&quot;success&quot; %}
Paigaldaja lisab automaatselt `chloros-cli` teie süsteemi PATH-i, et võimaldada juurdepääsu käsureale.
{% endhint %}

***

## Lisaressursid

### Python SDK

Arendajate ja automatiseeritud töövoogude jaoks installige Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentatsioon**: [API: Python SDK](api-python-sdk.md)**Nõuded**: Chloros Desktop peab olema installitud, Chloros+ litsentsi sisselogimine on vajalik.***

## Komplektis sisaldub

Chloros installatsioon sisaldab:

* ✅ **Chloros** – täisfunktsionaalne graafiline liides
* ✅ **Chloros (brauser)** – veebipõhine liides madalama spetsifikatsiooniga süsteemidele
* ✅ **Chloros CLI** – käsurealiides (nõuab Chloros+ litsentsi)
* ✅ **Chloros SDK** - Python API (nõuab Chloros+ litsentsi)
* ✅ **Kaameraprofiilid** - Eelkonfigureeritud MAPIR kaameramallid***

## Uuendage Chloros+ versioonile

Avage täiustatud funktsioonid Chloros+ tellimusega:

* 🚀 **Mitmeprotsessiline töötlemine** – töötlege pilte paralleelselt
* ⚡ **GPU (CUDA) kiirendus** – kasutage NVIDIA GPU võimsust
* 💻 **CLI juurdepääs** – automatiseerimine käsurea tööriistadega
* 🐍 **Python SDK** – programmiline API juurdepääs
* 📱 **Mitmed seadmed** – kasutamine 2–10+ seadmel (sõltuvalt paketist)
* 🧮 **Kohandatud valemid** – looge kohandatud multispektraalsed indeksid

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Vaata Chloros+ plaane ja hindu</a></p>***

## Installimise abi

### Veaotsing

**Installimine ebaõnnestub veateatega:**

* Veenduge, et teil on administraatori õigused
* Lülitage ajutiselt välja viirusetõrje tarkvara
* Kontrollige, et teie süsteem vastab miinimumnõuetele

**Rakendus ei käivitu:**

* Proovige Chloros (brauser) versiooni
* Veenduge, et Windows 10/11 (64-bitine) on installitud
* Uuendage graafikadraiverid
* Kontrollige Windows sündmustevaatajat vea üksikasjade leidmiseks
* Võtke ühendust tugiteenistusega ja saatke vea logid

**Litsentsi aktiveerimise probleemid:**

* Veenduge, et internetiühendus on aktiivne
* Kontrollige oma kasutajatunnuseid [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontrollige, et tulemüür ei blokeeri Chloros
* Täpsed juhised leiate [Chloros+ Login](chloros+-login.md)

### Abi saamine

Vajate abi installimise või seadistamisega?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Veebisait**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentatsioon**: [Alustamine](./)
* ❓ **KKK**: [Korduma kippuvad küsimused](faq.md)***

## Muudatuste logi

<details>

<summary>Versioon 1.0.4</summary>

#### **Väljalaske kuupäev**: 5. jaanuar 2026**Uued funktsioonid*** **Pildi/metadata lüliti**: Lisatud lüliti failibrauserisse, et vaadata valitud pildi metadata tabelis, mitte pildirastris
* **Pildirastri suumimisliugur**: Uus kasutajaliidese liugur pisipiltide suuruse reguleerimiseks (toetab ka CTRL + hiirerattat)
* **Pildivõrgu ekspordiklahvid**: ülemises reas asuvad klahvid pisipiltide vahetamiseks JPG-vormingust töödeldud ekspordivormingusse (sihtmärgid, peegeldus, indeks, LUT)
* **Kaartide vahekaart**: uus interaktiivne 2D-kaart, mis näitab pildi GPS-asukoha märgiseid.
  * Toetab Google Mapsi ja ESRI kaarditahvleid (valib automaatselt parima tahvli teenuse vastavalt suumitaseme kättesaadavusele).
  * Hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega hiirega

</details>

<details>

<summary>Versioon 1.0.3</summary>

#### **Väljalaske kuupäev**: 20. detsember 2025**Uued funktsioonid*** Esmane käivitamine

**Parandused*** Esmane käivitamine

**Veaparandused*** Esmane käivitamine

**Teadaolevad probleemid*** Esmane käivitamine

</details>***

## Litsentsileping**Omanditarkvara** – Autoriõigus (c) 2025 MAPIR Inc.

Ebaõiguslik kasutamine, levitamine või muutmine on keelatud.

**Tasuta versioon**: saadaval isiklikuks ja äriliseks kasutamiseks funktsioonide piirangutega.**Chloros+**: tellimuspõhine litsents täiustatud funktsioonide ja ärilise kasutusega.
