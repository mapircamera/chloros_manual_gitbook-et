# Töötlemise alustamine

Kui olete pildid importinud, kalibreerimismärgid märkinud ja projekti seaded konfigureerinud, olete valmis töötlemist alustama. Käesolev lehekülg juhendab teid Chloros töötlemisprotsessi käivitamisel.

## Eeltöötlemise kontrollnimekiri

Enne nupule „Start” klõpsamist veenduge, et kõik on valmis:

* [ ] **Failid imporditud** – kõik pildid kuvatakse failibrauseris
* [ ] **Sihtmärgipildid märgitud** – kalibreerimispiltide sihtmärgi veerg on märgitud
* [ ] **Kaameramudelid tuvastatud** – veerus „Kaameramudel” kuvatakse õiged kaamerad
* [ ] **Seaded konfigureeritud** – projekti seaded on üle vaadatud ja kohandatud
* [ ] **Indeksid valitud** – soovitud multispektraalsed indeksid on lisatud (vajaduse korral)
* [ ] **Ekspordivorming valitud** – teie töövoole sobiv väljundvorming

{% hint style="info" %}
**Nõuanne**: Klõpsake failibrauseris mõnel pildil, et veenduda, et need on enne töötlemist õigesti laaditud.
{% endhint %}

***

## Töötlemise alustamine

### Leia nupp „Start”

Nupp „Start/Play” asub Chloros ülemises päisribal:

* Asukoht: akna ülemine keskkohta
* Ikoon: **nupp „Play/Start”** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Staatus: nupp on aktiveeritud (heleda värviga), kui töötlemiseks on valmis

### Klõpsake alustamiseks

1. Klõpsake ülemises päises **Esita/Alusta nuppu**

2. Töötlemine algab kohe
3. Töötlemise ajal muutub nupp deaktiveerituks (halliks)
4. Edusammude riba uueneb, näidates töötlemise staatust

{% hint style="success" %}
**Töötlemine alustatud**: Kui nupule klõpsatakse, teostab Chloros automaatselt kõik töötlemisetapid – sihtmärgi tuvastamine, debayering, kalibreerimine, indeksi arvutamine ja eksport.
{% endhint %}

***

## Töötlemisrežiimide mõistmine

Chloros töötab sõltuvalt teie litsentsist kahes erinevas töötlemisrežiimis:

### Tasuta režiim (järkjärguline töötlemine)

**Kättesaadav kõigile kasutajatele**

**Kuidas see töötab:**

* Töötleb pilte ükshaaval, järjestikku
* Üheprotsessiline töö
* Väiksem mälukasutus

**Edusammude riba näitab 2 etappi:**

1.**Sihtmärgi tuvastamine** – kalibreerimissihte otsimine
2. **Töötlemine** – kalibreerimise rakendamine ja piltide eksportimine**Töötlemisaeg:**

* Palju aeglasem kui Chloros+ paralleelrežiim
* Sobib väikestele ja keskmise suurusega andmekogumitele (&lt; 200 pilti)

### Chloros+ režiim (paralleelne töötlemine)

**Nõuab Chloros+ litsentsi**

**Kuidas see töötab:**

* Töötleb mitut pilti samaaegselt, kasutades [4-niitlist töötlemispipeliini](../processing-architecture/processing-pipeline.md)
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) valib automaatselt teie riistvarale optimaalse strateegia
* GPU (CUDA) kiirendus NVIDIA graafikakaartidega (lauaarvuti ja Jetson)
* Skaleeritav alates Jetson Nanost (1 töötaja) kuni lauaarvutini, millel on 12 GB+ GPU (3–4 töötajat)

**Edusammude riba näitab 4 etappi** (vastavalt 4 töötlemisjärjekorra niidile):

1. **Tuvastamine** (niit 1) – kalibreerimise sihtmärkide leidmine
2. **Analüüsimine** (niit 2) – pildi metaandmete uurimine ja kalibreerimise arvutamine
3. **Kalibreerimine** (niit 3) – GPU debayering, vinjeti korrigeerimine, indeksite arvutamine
4. **Eksportimine** (niit 4) – töödeldud piltide ja indeksite salvestamine**Progressiriba kasutamine:*** **Viige hiir** ribale, et näha üksikasjalikku 4-etapilist rippmenüüd
* **Klõpsake** progressiribal, et rippmenüü paigale külmutada
* **Klõpsake uuesti**, et paneel vabastada ja peita**Töötlemisaeg:**

* Märkimisväärselt kiirem kui tasuta režiim
* Skaalub vastavalt CPU tuumade arvule
* GPU kiirendus parandab kiirust veelgi

{% hint style="info" %}
**Chloros+ kiirus**: Paralleelne töötlemine võib suurte andmekogumite puhul olla 5–10 korda kiirem kui järjestikune režiim. 500 pildiga projekt, mis vabarežiimis võtab aega 2 tundi, võib Chloros+ abil valmida 15–20 minutiga.
{% endhint %}

***

## Mis toimub töötlemise ajal

### 1. etapp: sihtmärgi tuvastamine

**Mida Chloros teeb:**

* Skaneerib märgistatud sihtmärgi pilte (või kõiki pilte, kui ühtegi pole märgistatud)
* Tuvastab igas sihtmärgis 4 kalibreerimispaneeli
* Eraldab sihtmärgi paneelidelt peegeldusväärtused
* Salvestab sihtmärkide ajamärgid kalibreerimise ajakava jaoks

**Kestus:** 1–30 sekundit (märgitud sihtmärkidega), 5–30+ minutit (märkimata)

### 2. etapp: Debayering (RAW-muundamine)

**Mida Chloros teeb:**

* Konverteerib RAW-vormingus olevad Bayeri mustri andmed täielikeks RGB piltideks
* Rakendab kõrgekvaliteedilist demosaicing-algoritmi
* Säilitab maksimaalse pildikvaliteedi ja detailid

**Kestus:** Sõltub piltide arvust ja protsessori kiirusest

### 3. etapp: Kalibreerimine

**Mida Chloros teeb:*** **Vignette-korrektsioon**: Eemaldab objektiivi tumenemise servades
* **Peegeldusvõime kalibreerimine**: Normaliseerib sihtmärgi peegeldusvõime väärtuste abil
* Rakendab korrektsioone kõigile sagedusaladele/kanalitele
* Kasutab iga pildi jaoks sobivat kalibreerimissihte, lähtudes ajamärgist

**Kestus:** Suurem osa töötlemisajast

### 4. etapp: Indeksi arvutamine

**Mida Chloros teeb:**

* Arvutab konfigureeritud multispektraalsed indeksid (NDVI, NDRE jne)
* Rakendab kalibreeritud piltidele lainepikkuste matemaatikat
* Genereerib indeksipildid iga valitud indeksi jaoks

**Kestus:** Mõni sekund pildi kohta

### 5. etapp: Eksport

**Mida Chloros teeb:**

* Salvestab kalibreeritud pildid valitud formaadis
* Ekspordib indeksipildid konfigureeritud LUT-värvidega
* Kirjutab failid kaameramudeli alamkaustadesse
* Säilitab originaalfailinimed koos laienditega

**Kestus:** Sõltub ekspordiformaadist ja faili suurusest***

## Töötlemise käitumine

### Automaatne töötlemisprotsess

Kui protsess on käivitatud, töötab kogu protsess automaatselt:

* Kasutaja sekkumist ei ole vaja
* Kõik konfigureeritud sammud täidetakse järjest
* Töö edenemist näidatakse reaalajas

### Arvuti kasutamine töötlemise ajal

**Vaba režiim:**

* Suhteliselt madal CPU kasutamine (ühekiuline)
* Arvuti jääb teiste ülesannete jaoks reageerivaks
* On ohutu minimeerida Chloros ja töötada teistes rakendustes

**Chloros+ Paralleelrežiim:**

* Kõrge CPU kasutamine (mitmekiuline, kuni 16 tuuma)
* GPU kiirendusega: kõrge GPU koormus
* Arvuti võib töötlemise ajal reageerida aeglasemalt
* Vältige teiste CPU-mahukate ülesannete käivitamist

{% hint style="warning" %}
**Jõudluse näpunäide**: Parima Chloros+ jõudluse saavutamiseks sulgege teised rakendused ja laske Chloros-il kasutada kõiki süsteemi ressursse.
{% endhint %}

### Töötlemist ei saa peatada

**Olulised piirangud:**

* Kui töötlemine on käivitatud, ei saa seda peatada
* Töötlemise saab tühistada, kuid edusammud kaovad
* Osalisi tulemusi ei salvestata
* Tühistamise korral tuleb alustada algusest

**Soovitus planeerimiseks:** väga suurte projektide puhul kaaluge töötlemist partiidena või kasutage parema kontrolli saavutamiseks CLI-i.***

## Töötlemise jälgimine

Töötlemise käigus saate:

* **Vaadata edenemisribat** – näha üldist valmimisprotsenti
* **Vaadata praegust etappi** – tuvastamine, analüüs, kalibreerimine või eksport
* **Vaadata logi vahekaarti** – vaadata üksikasjalikke töötlemissõnumeid ja hoiatusi
* **Vaadata eelvaadet valmis piltidest** – mõned ekspordifailid võivad ilmuda töötlemise ajal

Täpsema teabe jälgimise kohta leiate jaotisest [Töötlemise jälgimine](monitoring-the-processing.md).

***

## Töötlemise tühistamine

Kui peate töötlemise peatama:

### Kuidas tühistada

1. Leidke **Peata/Tühista nupp** (asendab töötlemise ajal Start nuppu)
2. Klõpsake Peata nuppu
3. Töötlemine peatub kohe
4. Osalised tulemused kõrvaldatakse

### Millal tühistada

**Kehtivad põhjused tühistamiseks:**

* Saadi aru, et kasutati valesid seadeid
* Unustati märkida sihtpildid
* Imporditi valed pildid
* Süsteem töötab liiga aeglaselt või ei reageeri

**Pärast tühistamist:**

* Vaadake üle ja parandage kõik probleemid
* Kohandage seadeid vastavalt vajadusele
* Alustage töötlemist uuesti algusest
* Parima tulemuse saamiseks sulgege Chloros täielikult ja käivitage uuesti

{% hint style="warning" %}
**Osalisi tulemusi ei ole**: Tühistamine tühistab kogu edusammud. Chloros ei salvesta osaliselt töödeldud pilte.
{% endhint %}

***

## Töötlemise aja hinnangud

Tegelik töötlemisaeg varieerub suuresti sõltuvalt:

* Piltide arvust
* Pildi resolutsioonist
* RAW vs JPG sisendformaadist
* Töötlemisrežiimist (Free vs Chloros+)
* CPU kiirus ja tuumade arv
* GPU kättesaadavus (ainult Chloros+)
* Arvutatavate indeksite arv
* Ekspordiformaadi keerukus

### Ligikaudsed hinnangud (Chloros+, 12MP pildid, kaasaegne CPU)

| Piltide arv | Tasuta režiim | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 pilti   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 pilti  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 pilti  | 1–1,5 tundi | 20–30 min      | 10–15 min      |
| 500 pilti  | 2–3 tundi   | 45–60 min      | 20–30 min      |
| 1000 pilti | 4–6 tundi   | 1,5–2 tundi      | 40–60 min      |

{% hint style="info" %}
**Esimene käivitamine**: Esmane töötlemine võib võtta kauem aega, kuna Chloros loob vahemälusid ja profiile. Sarnaste andmekogumite edasine töötlemine toimub kiiremini.
{% endhint %}

***

## Tüüpilised probleemid käivitamisel

### Käivitusnupp on keelatud (halliks muutunud)

**Võimalikud põhjused:**

* Pilte pole imporditud
* Tagapõhi pole täielikult käivitatud
* Eelmine töötlemine on veel käimas
* Projekt pole täielikult laaditud

**Lahendused:**

1. Oodake, kuni tagapõhi on täielikult algseadistatud (kontrollige peamenüü ikooni)
2. Veenduge, et pildid on failibrauserisse imporditud
3. Käivitage Chloros uuesti, kui nupp jääb blokeerituks
4. Kontrollige veateateid silumislogist

### Töötlemine algab, kuid ebaõnnestub kohe

**Võimalikud põhjused:**

* Projektis pole kehtivaid pilte
* Rikutud pildifailid
* Ebapiisav kettaruum
* Ebapiisav mälu (RAM)

**Lahendused:**

1. Kontrollige silumislogist <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> veateadete leidmiseks
2. Kontrollige vaba kettaruumi olemasolu
3. Proovige töödelda väiksemat piltide alarühma
4. Kontrollige, et pildid ei ole rikutud

### Hoiatus „Sihtmärke ei leitud”

**Võimalikud põhjused:**

* Unustasite sihtmärgipilte märgistada
* Sihtmärgipildid ei sisalda nähtavaid sihtmärke
* Sihtmärkide tuvastamise seaded on liiga ranged

**Lahendused:**

1. Vaadake läbi [Sihtpiltide valimine](choosing-target-images.md)
2. Märgistage sobivad pildid veerus „Sihtmärk”
3. Veenduge, et märgistatud piltidel on sihtmärgid nähtavad
4. Vajadusel kohandage sihtmärkide tuvastamise seadeid

***

## Näpunäited edukaks töötlemiseks

### Enne alustamist

1. **Testige esmalt väikese alarühmaga** – töötlege 10–20 pilti, et seadeid kontrollida
2. **Kontrollige vaba kettaruumi** – tagage, et vaba ruumi oleks 2–3 korda rohkem kui andmekogumi suurus
3. **Sulgege mittevajalikud rakendused** – vabastage süsteemiressursse
4. **Kontrollige sihtpilte** – vaadake märgitud sihtmärke eelvaates, et veenduda kvaliteedis
5. **Salvestage projekt** – projekt salvestatakse automaatselt, kuid hea tava on salvestada ka käsitsi

### Töötlemise ajal

1. **Vältige süsteemi puhkeolekut** – lülitage energiasäästurežiimid välja
2. **Hoidke Chloros esiplaanil** – või vähemalt nähtaval tegumiribal
3. **Jälgige aeg-ajalt töötlemise kulgu** – kontrollige hoiatuste ja vigade olemasolu
4. **Ärge laadige teisi ressursimahukaid rakendusi** – Eriti Chloros+ paralleelrežiimis

### Chloros+ GPU kiirendus

Kui kasutate NVIDIA GPU kiirendust:

1. Uuendage NVIDIA draiverid viimase versioonini
2. Veenduge, et GPU-l on vähemalt 4 GB VRAM-i
3. Sulge GPU-mahukad rakendused (mängud, videotöötlus)
4. Jälgi GPU temperatuuri (tagada piisav jahutus)

***

## Järgmised sammud

Kui töötlemine on alanud:

1. **Jälgi protsessi kulgu** – vaata [Töötlemise jälgimine](monitoring-the-processing.md)
2. **Oodake töötlemise lõppu** – töötlemine toimub automaatselt
3. **Vaadake tulemusi** – vt [Töötlemise lõpetamine](finishing-the-processing.md)

Teavet selle kohta, mida töötlemise ajal teha, leiate jaotisest [Töötlemise jälgimine](monitoring-the-processing.md).
