# Dünaamiline arvutusvõimsuse kohandamine

Chloros versioonis 1.1.0 on kasutusele võetud nutikas riistvara tuvastamine ja töötlemisstrateegia automaatne valik. Töötlemismootor kohandub teie riistvaraga – alates Jetson Nanost kuni mitme GPU-ga tööjaamani – ilma käsitsi seadistamiseta.

***

## Kuidas see toimib

Kui Chloros käivitub, koostab see automaatselt teie süsteemi profiili:

1. **Tuvastab operatsioonisüsteemi** — Windows või Linux
2. **Määrab kindlaks CPU tuumad ja RAM-i kogumahu**

3.**Tuvastab GPU olemasolu** — NVIDIA CUDA võimekus, VRAM, mudel
4. **Määrab kindlaks Jetsoni mudeli** (kui see on olemas) — `/proc/device-tree/model` kaudu
5. **Kontrollib temperatuuriandureid** (Jetson) — temperatuuritundlikuks töötlemiseks
6. **Valib optimaalse arvutusstrateegia** — kõigi tuvastatud riistvarade põhjal
7. **Konfigureerib töötajate arvu, töötlemisjärjekorra tüübi ja mälu eraldamise** automaatselt

Tulemus salvestatakse vahemällu, et järgmised käivitused algaksid kiiremini. Kui riistvara muutub (nt lisatakse GPU), teeb Chloros järgmisel käivitamisel uue profiili.

***

## Arvutusstrateegiad

Chloros valib teie riistvara põhjal ühe kolmest arvutusstrateegiast:

| Strateegia | Vajalik GPU | Töötajad | Töötlemisahel | Sobib kõige paremini |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Jah (12 GB+ VRAM või 16 GB+ jagatud) | 3–4 | `fused_gpu` | Lauaarvuti GPU-d 12 GB+ mäluga, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Jah (&lt; 12 GB VRAM) | 1–3 | `tiled_gpu` | algtaseme GPU-d, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Ei | tuumad – 1 | `cpu_fallback` | Süsteemid ilma NVIDIA GPU-ta |

### Töötlemisliinide tüübid

* **`fused_gpu`** — Täielik GPU töötlemisliin. Kõik debayeri, korrektsiooni ja indeksimise operatsioonid töötavad GPU-l ühe ühendatud läbimisega. Kõrgeim läbilaskevõime, kuid nõuab rohkem VRAM-i.
* **`tiled_gpu`** — Mälu-efektiivne GPU-tee. Töötleb pilte ruutudena, et need mahuksid piiratud GPU-mälu sisse. Madalam läbilaskevõime, kuid töötab mälupiiranguga seadmetel.
* **`cpu_fallback`** — Ainult CPU-põhine töötlemine, kasutades mitmeprotsessilist paralleelsust. Kasutatakse juhul, kui NVIDIA GPU pole saadaval.***

## Platvormispetsiifiline käitumine

| Platvorm | Strateegia | Töötajad | Töötlemisahel | Märkused |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serialiseeritud) | Mälu säästev režiim, töötleb ühe pildi korraga |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (paralleelne) | Soovitatav servaseade — tõeline paralleelne GPU-töötlus |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (paralleelne) | Maksimaalne servaseadme jõudlus |
| **Lauaarvuti 8 GB GPU-ga** | `GPU_SINGLE` | 3 | `tiled_gpu` | Hea lauaarvuti jõudlus mälusäästlike plaadidega |
| **Töölauaarvuti 12 GB+ GPU-ga** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimaalne töölauaarvuti jõudlus |
| **Ainult CPU-ga süsteem** | `CPU_PARALLEL` | tuumad – 1 | `cpu_fallback` | GPU-d ei vaja, kasutab ThreadPooli |

{% hint style="info" %}
**Jetsoni ühtne mälu**: Jetsoni seadmed jagavad GPU ja CPU mälu. Jetson Orin NX 16 GB näitab ~15,3 GB VRAM-i, kuid see on sama füüsiline RAM, mida kasutavad operatsioonisüsteem ja CPU protsessid. Chloros arvestab seda mälu eraldamise künniste seadmisel.
{% endhint %}

***

## Dünaamiline GPU mälu eraldamine

Chloros kasutab [4-niitlist töötlemispipeliini](processing-pipeline.md):

* **Niit 1** (Tuvastamine) — Pildi laadimine, EXIF-i analüüs, sihtmärgi tuvastamine
* **Niit 2** (Kalibreerimine) — Peegeldusvõime kalibreerimise arvutamine
* **Lõng 3** (töötlemine) — GPU debayer, vinjeti korrigeerimine, indeksi arvutamine
* **Lõng 4** (eksport) — failide kirjutamine, metaandmete lisamine

Kui varasemad töötlemisjärjekorra lõngad on oma töö lõpetanud (nt kõik pildid on tuvastatud), vabastatakse nende GPU mälu eraldus ja **jaotatakse ümber ülejäänud aktiivsetele lõngadele**. See tähendab, et niit 3 (GPU-mahukas etapp) saab töötlemisahela edenedes järjest rohkem mälu, parandades läbilaskevõimet kõige arvutusmahukamate ülesannete puhul.

### Eraldamise etapid

| Etapp | Aktiivsed niidid | GPU mälu jaotus |
| --- | --- | --- |
| **Alguses** | 1, 2, 3, 4 | Jagatud kõigi niitide vahel |
| **Varajane-keskmine** | 2, 3, 4 | Niidi 1 mälu ümberjaotatud |
| **Keskmine-hiline** | 3, 4 | Niidid 1+2 mälu läheb 3+4-le |
| **Hiline** | 3 või 4 | Maksimaalne mälu ülejäänud niidile |***

## Tekstuuritundlik töötlemine

Tekstuuritundlik debayer-meetod (ainult Chloros+) kasutab AI/ML-müra vähendamise mudeli tõttu märkimisväärselt rohkem GPU mälu kui standardmeetod:

* Süsteemid, millel on **&lt; 7 GB VRAM**, on tekstuuritundliku režiimi puhul sunnitud kasutama sünkroonset töötlemistsüklit (üks pilt korraga)
* Süsteemid, millel on **7 GB+ VRAM**, saavad tekstuuritundlikku töötlemist teha samaaegselt, kuigi töötajate arv on standardmeetodiga võrreldes väiksem***

## Soojusjuhtimine (Jetson)

Jetson-seadmetel on soojuspiirangud, eriti suletud või õhus kasutamisel. Chloros jälgib GPU ja CPU temperatuure ning kohandab töötlemist automaatselt:

| Temperatuur | Reaktsioon |
| --- | --- |
| **&lt; 70 °C** | Normaalne töö — täiskiirus |
| **70 °C** (hoiatus) | Vähenda partii suurust |
| **80 °C** (kriitiline) | Agressiivne piiramine — vähenda samaaegsust ja töötajate arvu |
| **90 °C** (Seiskamine) | Peata GPU töötlemine täielikult |

Temperatuuri jälgimiseks kasutatakse Jetsoni platvormidel `tegrastats`-i. Piisava jahutusega lauaarvutites käivitub soojuspiirang harva.

***

## Mälu koormuse käsitlemine

Chloros jälgib töötlemise ajal süsteemi mälukoormust:

* **Mälukünnis**: 85% kasutamine käivitab konservatiivse käitumise
* **OOM-vähendus**: Kui tekib mälupuudus, vähendatakse eraldust 25% võrra (kordaja 0,75x)
* **Pipeline&#x27;i tagasipöördumine**: Tõsise mälukoormuse korral lülitub torujuhtme töö automaatselt `fused_gpu`-ilt `tiled_gpu`-ile
* **Vahetusmälusoovitused**: Jetsonil hoiatab Chloros, kui vahetusmälu ei ole andmekogumi suurusele piisav***

## Arvutusvõimsuse kohandamise jälgimine

### CLI staatuse väljund

Töötlemise alguses kuvab CLI tuvastatud riistvaraprofiili:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Süsteemidiagnostika

Käivitage `chloros-cli selftest`, et näha täielikku riistvaraprofiili ja kontrollida arvutusvõimsust:

```bash
chloros-cli selftest
```

See kontrollib CUDA kättesaadavust, GPU mälu, müra vähendamise mudeleid ja tagapõhja ühenduvust.

***

## Järgmised sammud

* [Töötlemispipeline](processing-pipeline.md) — 4-niitilise pipeline&#x27;i arhitektuuri mõistmine
* [NVIDIA Jetsoni juhend](../linux/nvidia-jetson-guide.md) — Jetsonile spetsiifiline kasutuselevõtt ja optimeerimine
* [CLI : käsurea](../CLI.md) — Täielik CLI viide
