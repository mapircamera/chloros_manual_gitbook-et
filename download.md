---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Allalaadimine

Laadige alla Chloros uusim versioon Windows jaoks, et alustada multispektraalsete piltide töötlemist.

### Süsteeminõuded

| Nõue          | Minimaalne                         | Soovitatav                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Operatsioonisüsteem** | Windows 10 (64-bitine)             | Windows 11 (64-bitine)             |
| **Protsessor**        | Intel Core i5 või samaväärne     | Intel Core i7 või parem         |
| **Mälu (RAM)**     | 8 GB                             | 16 GB või rohkem                    |
| **Graafikakaart**    | DirectX 11 ühilduv           | NVIDIA GPU 4 GB+ VRAM-iga       |
| **Salvestusruum**          | 2 GB vaba ruumi                  | SSD 10 GB+ vaba ruumiga       |
| **Ekraan**          | 1920x1080                       | 2560x1440 või suurem             |
| **Internet**         | Vajalik litsentsi aktiveerimiseks | Vajalik litsentsi aktiveerimiseks |

{% vihje style=&quot;info&quot; %}
**GPU kiirendus**: Chloros+ kasutajad, kellel on NVIDIA GPU (4 GB+ VRAM), saavad kasutada CUDA kiirendust, et töötlemine oleks oluliselt kiirem.
{% endhint %}

***

## Lae alla Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Lae alla Chloros siit</a>

### Viimane stabiilne versioon

**Chloros installija Windows jaoks**

* **Versioon**: 1.0.3
* **Väljalaske kuupäev**: detsember ?, 2025
* **Faili suurus**: 1,6 GB
* **Failitüüp**: .exe (Windows installija)

#### **Paigaldamise sammud:**

1. Lae alla fail `CHLOROS INSTALLER - CURRENT VERSION.exe`.
2. Paigaldamise alustamiseks topeltklõpsa paigaldusprogrammi.
3. Järgi paigaldusviisardi juhiseid.
4. Vali paigalduskataloog (vaikimisi: `C:\Program Files\Chloros\`).
5. Viima paigaldamine lõpule ja käivita Chloros.
6. Logige sisse oma MAPIR Cloud Chloros+ kontoga (või jätkake tasuta versiooniga)

{% hint style=&quot;success&quot; %}
Installija lisab automaatselt `chloros-cli` teie süsteemi PATH-i, et võimaldada juurdepääsu käsureale.
{% endhint %}

***

## Lisaressursid

### Python SDK

Arendajate ja automatiseeritud töövoogude jaoks installige Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentatsioon**: [API: Python SDK](api-python-sdk.md)

**Nõuded**: Chloros Desktop peab olema installitud, Chloros+ litsents on vajalik.

***

## Komplektis sisalduv

Chloros installatsioon sisaldab:

* ✅ **Chloros Desktop GUI** – täisfunktsionaalne graafiline liides
* ✅ **Chloros (brauser)** – veebipõhine liides madalama spetsifikatsiooniga süsteemidele
* ✅ **Chloros CLI** – käsurealiides (nõuab Chloros+ litsentsi)
* ✅ **Backend Engine** – pilditöötlusprotsess
* ✅ **Kaameraprofiilid** - Eelkonfigureeritud MAPIR kaameramallid

***

## Uuendage Chloros+ versioonile

Avage täiustatud funktsioonid Chloros+ tellimusega:

* 🚀 **Mitmeprotsessiline töötlemine** – töötlege pilte paralleelselt
* ⚡ **GPU (CUDA) kiirendus** – NVIDIA GPU võimsuse ärakasutamine
* 💻 **CLI juurdepääs** – automatiseerimine käsurea tööriistadega
* 🐍 **Python SDK** - Programmiline API juurdepääs
* 📱 **Mitmed seadmed** - Kasutamine 2–10+ seadmel (sõltuvalt paketist)
* 🧮 **Kohandatud valemid** - Kohandatud multispektraalsete indeksite loomine

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Vaata Chloros+ plaane ja hindu</a></p>***

## Installimise abi

### Veaotsing

**Installimine ebaõnnestub veateatega:**

* Veenduge, et teil on administraatori õigused
* Lülitage ajutiselt välja viirusetõrjeprogramm
* Kontrollige, kas teie süsteem vastab miinimumnõuetele

**Rakendus ei käivitu:**

* Proovige Chloros (brauseri) versiooni
* Veenduge, et Windows 10/11 (64-bitine) on installitud
* Uuendage graafikadraiverid
* Kontrollige Windows sündmustevaatajat vea üksikasjade leidmiseks
* Võtke ühendust tugiteenistusega ja saatke vea logid

**Litsentsi aktiveerimise probleemid:**

* Veenduge, et internetiühendus on aktiivne
* Kontrollige oma kasutajatunnust [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontrollige, et tulemüür ei blokeeri Chloros
* Vaadake [Chloros+ Login](chloros+-login.md) üksikasjalikke juhiseid

### Abi saamine

Vajate abi installimise või seadistamisega?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Veebisait**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentatsioon**: [Alustamine](./)
* ❓ **KKK**: [Korduma kippuvad küsimused](faq.md)

***

## Muudatuste logi

<details>

<summary>Versioon 1.0.3</summary>

### **Väljalaske kuupäev**: detsember ?, 2025

#### Uued funktsioonid

* Esmane käivitamine

#### Parandused

* Esmane käivitamine

#### Veaparandused

* Esmane käivitamine

#### Teadaolevad probleemid

* Esmane käivitamine

</details>***

## Litsentsileping

**Omanditarkvara** – Autoriõigus (c) 2025 MAPIR Inc.

Ebaõiguslik kasutamine, levitamine või muutmine on keelatud.

**Tasuta versioon**: saadaval isiklikuks ja äriliseks kasutamiseks funktsioonide piirangutega.

**Chloros+**: tellimuspõhine litsents täiustatud funktsioonide ja ärilise kasutusega.
