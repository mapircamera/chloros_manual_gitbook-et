# Töötlemise alustamine

Kui olete importinud pildid, märkinud kalibreerimise sihtmärgid ja konfigureerinud projekti seaded, olete valmis töötlemise alustamiseks. See lehekülg juhendab teid Chloros töötlemisprotsessi käivitamisel.

## Eeltöötluse kontrollnimekiri

Enne nupule „Start” klõpsamist veenduge, et kõik on valmis:

* [ ] **Failid imporditud** – kõik pildid kuvatakse failibrauseris
* [ ] **Sihtmärgid märgitud** – kalibreerimispiltide sihtmärgi veerg on märgitud
* [ ] **Kaameramudelid tuvastatud** – kaameramudeli veerus on näidatud õiged kaamerad
* [ ] **Seaded konfigureeritud** – projekti seaded on üle vaadatud ja kohandatud
* [ ] **Indeksid valitud** – soovitud multispektraalsed indeksid on lisatud (vajaduse korral)
* [ ] **Ekspordivorming valitud** – teie töövoogule sobiv väljundvorming

{% vihje style=&quot;info&quot; %}
**Vihje**: Klõpsake failibrauseris mõnel pildil, et veenduda, et need on enne töötlemist õigesti laaditud.
{% endhint %}

***

## Töötlemise alustamine

### Leia alustamisnupp

Alustamis-/esitusnupp asub Chloros ülemises päisribas:

* Asukoht: akna ülemine keskosa
* Ikoon: **Esitus-/alustamisnupp** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Staatus: nupp on aktiveeritud (heleda värviga), kui on valmis töötlemiseks

### Alustamiseks klõpsake

1. Klõpsake ülemises ribas **Play/Start nuppu**
2. Töötlemine algab kohe
3. Nupp muutub töötlemise ajal mitteaktiivseks (halliks)
4. Progressiriba uueneb, näidates töötlemise staatust

{% vihje stiil=&quot;edu&quot; %}
**Töötlemine alustatud**: Pärast klõpsamist käsitleb Chloros automaatselt kõiki töötlemisetappe – sihtmärgi tuvastamine, debayering, kalibreerimine, indeksite arvutamine ja eksport.
{% endhint %}

***

## Töötlemisrežiimide mõistmine

Chloros töötab sõltuvalt teie litsentsist kahes erinevas töötlemisrežiimis:

### Tasuta režiim (järkjärguline töötlemine)

**Kättesaadav kõigile kasutajatele**

**Kuidas see töötab:**

* Töötleb pilte ükshaaval järjestikku
* Üheprotsessiline töö
* Väiksem mälu kasutamine

**Progressiriba näitab 2 etappi:**

1. **Sihtmärgi tuvastamine** – kalibreerimise sihtmärkide otsimine
2. **Töötlemine** – kalibreerimise rakendamine ja piltide eksportimine

**Töötlemisaeg:**

* Palju aeglasem kui Chloros+ paralleelrežiim
* Sobib väikeste ja keskmise suurusega andmekogumite jaoks (&lt; 200 pilti)

### Chloros+ režiim (paralleelne töötlemine)

**Nõuab Chloros+ litsentsi**

**Kuidas see töötab:**

* Töötleb mitut pilti korraga
* Mitmeprotsessiline töö (kuni 16 paralleelset töötajat)
* Kasutab mitut CPU tuuma
* Valikuline GPU (CUDA) kiirendus NVIDIA graafikakaartidega

**Edusammude riba näitab 4 etappi:**

1. **Avastamine** – kalibreerimise sihtmärkide leidmine
2. **Analüüsimine** – pildi metaandmete uurimine ja töötlemisprotsessi ettevalmistamine
3. **Kalibreerimine** – paranduste ja kalibreerimiste rakendamine
4. **Eksportimine** – töödeldud piltide ja indeksite salvestamine

**Progressiriba interaktsioon:**

* **Viige hiir** riba kohale, et näha üksikasjalikku 4-etapilist rippmenüüd
* **Klõpsake** progressiribal, et rippmenüü paigale külmutada
* **Klõpsake uuesti**, et paneel vabastada ja peita

**Töötlemisaeg:**

* Oluliselt kiirem kui vaba režiim
* Skaalub vastavalt CPU tuumade arvule
* GPU kiirendus parandab kiirust veelgi

{% vihje style=&quot;info&quot; %}
**Chloros+ kiirus**: Paralleelne töötlemine võib olla 5–10 korda kiirem kui järjestikune režiim suurte andmekogumite puhul. 500 pildiga projekt, mis vabarežiimis võtab aega 2 tundi, võib Chloros+ abil valmida 15–20 minutiga.
{% endhint %}

***

## Mis toimub töötlemise ajal

### 1. etapp: sihtmärgi tuvastamine

**Mida Chloros teeb:**

* Skaneerib märgistatud sihtmärgi pildid (või kõik pildid, kui märgitud sihtmärke pole)
* Identifitseerib iga sihtmärgi 4 kalibreerimispaneeli
* Ekstraktib sihtmärgi paneelidelt peegeldusväärtused
* Salvestab sihtmärgi ajamärgid kalibreerimise ajakava jaoks

**Kestus:** 1–30 sekundit (märgitud sihtmärkide puhul), 5–30+ minutit (märgimata sihtmärkide puhul)

### 2. etapp: Debayering (RAW-konverteerimine)

**Chloros teeb järgmist:**

* Konverteerib RAW-Bayer-mustri andmed täielikeks RGB-piltideks
* Rakendab kõrgekvaliteedilist demosaicing-algoritmi
* Säilitab maksimaalse pildikvaliteedi ja detailid

**Kestus:** sõltub piltide arvust ja CPU kiirusest

### 3. etapp: kalibreerimine

**Chloros teeb järgmist:**

* **Vignette&#x27;i korrigeerimine**: eemaldab objektiivi tumenemise servades
* **Peegelduskalibreerimine**: normaliseerib sihtmärgi peegeldusväärtuste abil
* Rakendab korrigeerimisi kõikides sagedusalades/kanalites
* Kasutab iga pildi jaoks sobivat kalibreerimise sihtmärki ajamärgise alusel

**Kestus:** Enamik töötlemisajast

### 4. etapp: Indeksi arvutamine

**Mida Chloros teeb:**

* Arvutab konfigureeritud multispektraalsed indeksid (NDVI, NDRE jne)
* Rakendab kalibreeritud piltidele riba matemaatikat
* Genereerib indeksipildid iga valitud indeksi jaoks

**Kestus:** Mõni sekund pildi kohta

### 5. etapp: Eksportimine

**Mida Chloros teeb:**

* Salvestab kalibreeritud pildid valitud formaadis
* Ekspordib indeksipildid konfigureeritud LUT-värvidega
* Kirjutab failid kaameramudeli alamkaustadesse
* Säilitab originaalfailide nimed koos laienditega

**Kestus:** Sõltub ekspordiformaadist ja faili suurusest

***

## Töötlemise käitumine

### Automaatne töötlemise protsess

Kui protsess on käivitatud, töötab kogu protsess automaatselt:

* Kasutaja sekkumist ei ole vaja
* Kõik konfigureeritud sammud täidetakse järjest
* Protsessi kulgemine kuvatakse reaalajas

### Arvuti kasutamine töötlemise ajal

**Vaba režiim:**

* Suhteliselt madal CPU kasutamine (üheprotsessiline)
* Arvuti jääb reageerivaks teiste ülesannete jaoks
* Ohutu minimeerida Chloros ja töötada teistes rakendustes

**Chloros+ Paralleelrežiim:**

* Kõrge CPU kasutamine (mitmeprotsessiline, kuni 16 tuuma)
* GPU kiirenduse korral: kõrge GPU kasutamine
* Arvuti võib töötlemise ajal olla vähem reageeriv
* Vältige teiste CPU-mahukate ülesannete käivitamist

{% vihje style=&quot;warning&quot; %}
**Jõudluse näpunäide**: Parima Chloros+ jõudluse saavutamiseks sulgege teised rakendused ja laske Chloros kasutada kõiki süsteemi ressursse.
{% endhint %}

### Töötlemist ei saa peatada

**Olulised piirangud:**

* Kord alustatud töötlemist ei saa peatada.
* Töötlemise saab tühistada, kuid edusammud kaovad.
* Osalisi tulemusi ei salvestata.
* Tühistamise korral tuleb alustada uuesti algusest.

**Planeerimise näpunäide:** väga suurte projektide puhul kaaluge töötlemist partiidena või kasutage CLI-i, et saavutada parem kontroll.

***

## Töötlemise jälgimine

Töötlemise käigus saate:

* **Vaadata edenemisribat** – näha üldist valmimisprotsenti
* **Vaadata praegust etappi** – tuvastamine, analüüsimine, kalibreerimine või eksportimine
* **Vaadata logi vahekaarti** – näha üksikasjalikke töötlemissõnumeid ja hoiatusi
* **Vaadata valmis pilte** – mõned ekspordifailid võivad ilmuda töötlemise käigus

Üksikasjalikku teavet jälgimise kohta leiate jaotisest [Töötlemise jälgimine](monitoring-the-processing.md).

***

## Töötlemise tühistamine

Kui soovite töötlemise peatada:

### Kuidas tühistada

1. Leidke **Peata/Tühista nupp** (asendab töötlemise ajal nuppu Alusta)
2. Klõpsake nuppu Peata
3. Töötlemine peatub kohe
4. Osalised tulemused kustutatakse

### Millal tühistada

**Kehtivad põhjused tühistamiseks:**

* Saite aru, et kasutasite valesid seadeid
* Unustasite märkida sihtpildid
* Importisite valed pildid
* Süsteem töötab liiga aeglaselt või ei reageeri

**Pärast tühistamist:**

* Vaadake üle ja parandage kõik probleemid
* Kohandage seadeid vastavalt vajadusele
* Alustage töötlemist uuesti algusest
* Parima tulemuse saamiseks sulgege Chloros täielikult ja alustage uuesti

{% vihje style=&quot;warning&quot; %}
**Osalisi tulemusi ei ole**: tühistamine kustutab kogu töötlemise käigu. Chloros ei salvesta osaliselt töödeldud pilte.
{% endhint %}

***

## Töötlemise aja hinnangud

Tegelik töötlemisaeg sõltub suuresti järgmistest teguritest:

* Piltide arv
* Pildi resolutsioon
* RAW vs JPG sisendformaat
* Töötlemisrežiim (tasuta vs Chloros+)
* CPU kiirus ja tuumade arv
* GPU kättesaadavus (ainult Chloros+)
* Arvutatavate indeksite arv
* Ekspordiformaadi keerukus

### Ligikaudsed hinnangud (Chloros+, 12 MP pildid, kaasaegne CPU)

| Piltide arv | Tasuta režiim | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 pilti   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 pilti  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 pilti  | 1–1,5 tundi | 20–30 min      | 10–15 min      |
| 500 pilti  | 2–3 tundi   | 45–60 min      | 20–30 min      |
| 1000 pilti | 4–6 tundi   | 1,5–2 tundi      | 40–60 min      |

{% vihje style=&quot;info&quot; %}
**Esimene käivitus**: Esmane töötlemine võib võtta kauem aega, kuna Chloros loob vahemälud ja profiilid. Järgnevate sarnaste andmekogumite töötlemine on kiirem.
{% endhint %}

***

## Tavapärased probleemid käivitamisel

### Käivitusnupp on keelatud (halliks muutunud)

**Võimalikud põhjused:**

* Pilte ei ole imporditud
* Tagapõhi ei ole täielikult käivitatud
* Eelmine töötlemine on veel käimas
* Projekt ei ole täielikult laaditud

**Lahendused:**

1. Oodake, kuni backend on täielikult algatatud (kontrollige peamenüü ikooni)
2. Kontrollige, kas pildid on failibrauserisse imporditud
3. Kui nupp jääb keelatuks, taaskäivitage Chloros
4. Kontrollige veateateid vealogist

### Töötlemine algab, kuid ebaõnnestub kohe

**Võimalikud põhjused:**

* Projektis pole kehtivaid pilte
* Pildifailid on rikutud
* Ebapiisav kettaruum
* Ebapiisav mälu (RAM)

**Lahendused:**

1. Kontrollige veaotsingulogi <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> veateadete olemasolu
2. Kontrollige kättesaadavat kettaruumi
3. Proovige töödelda väiksemat piltide alarühma
4. Kontrollige, et pildid ei ole rikutud

### Hoiatus „Sihtmärke ei leitud”

**Võimalikud põhjused:**

* Unustasite märkida sihtmärgid
* Sihtmärgid ei sisalda nähtavaid sihtmärke
* Sihtmärkide tuvastamise seaded on liiga ranged

**Lahendused:**

1. Vaadake läbi [Sihtpiltide valimine](choosing-target-images.md)
2. Märkige sobivad pildid sihtkolonnis
3. Kontrollige, et sihtmärgid on märgitud piltidel nähtavad
4. Vajadusel kohandage sihtmärgi tuvastamise seadeid

***

## Nõuanded edukaks töötlemiseks

### Enne alustamist

1. **Testige esmalt väikese alarühmaga** – töötlege 10–20 pilti, et kontrollida seadeid
2. **Kontrollige vaba kettaruumi** – veenduge, et vaba ruumi on 2–3 korda andmekogumi suurusest
3. **Sulge mittevajalikud rakendused** – vabastage süsteemi ressursse
4. **Kontrollige sihtpilte** – vaadake eelvaadet märgitud sihtmärkidest, et veenduda kvaliteedis
5. **Salvestage projekt** – projekt salvestatakse automaatselt, kuid hea tava on salvestada ka käsitsi.

### Töötlemise ajal

1. **Vältige süsteemi puhkeolekut** – lülitage välja energiasäästu režiimid.
2. **Hoidke Chloros esiplaanil** – või vähemalt nähtaval tegumiribal.
3. **Jälgige aeg-ajalt töötlemise kulgu** – kontrollige hoiatusi ja vigu.
4. **Ärge laadige teisi raskeid rakendusi** – eriti Chloros+ paralleelrežiimis

### Chloros+ GPU kiirendus

Kui kasutate NVIDIA GPU kiirendust:

1. Uuendage NVIDIA draiverid viimase versioonini
2. Veenduge, et GPU-l on 4 GB+ VRAM
3. Sulgege GPU-mahukad rakendused (mängud, videotöötlus)
4. Jälgige GPU temperatuuri (tagage piisav jahutus)

***

## Järgmised sammud

Kui töötlemine on alanud:

1. **Jälgige protsessi kulgu** – vaadake [Töötlemise jälgimine](monitoring-the-processing.md)
2. **Oodake töötlemise lõppu** – töötlemine toimub automaatselt.
3. **Vaadake tulemusi** – vt [Töötlemise lõpetamine](finishing-the-processing.md).

Teave selle kohta, mida töötlemise ajal teha, on esitatud jaotises [Töötlemise jälgimine](monitoring-the-processing.md).
