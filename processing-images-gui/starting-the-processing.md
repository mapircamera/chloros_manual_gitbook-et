# Töötlemise alustamine

Kui olete pildid importinud, kalibreerimismärgid märkinud ja projekti seaded konfigureerinud, olete valmis töötlemist alustama. Käesolev lehekülg juhendab teid Chloros töötlemisprotsessi käivitamisel.

## Eeltöötlemise kontrollnimekiri

Enne nupule „Start“ klõpsamist veenduge, et kõik on valmis:

* [ ] **Failid imporditud** – kõik pildid on näha failibrauseris
* [ ] **Sihtmärgipildid märgitud** – kalibreerimispiltide puhul on veerus „Target“ märge (või LATTICE jaoks on imporditud `.daq` salvestus)
* [ ] **Kaameramudelid tuvastatud** – veerus „Kaameramudel“ kuvatakse õiged kaamerad
* [ ] **Seaded konfigureeritud** – projekti seaded on üle vaadatud ja kohandatud
* [ ] **Indeksid valitud** – soovitud multispektraalsed indeksid on lisatud (vajaduse korral)
* [ ] **Ekspordivorming valitud** – Teie töövoole sobiv väljundvorming

{% hint style="info" %}
**Nõuanne**: Klõpsake failibrauseris mõnel pildil, et veenduda, et need on enne töötlemist õigesti laaditud.
{% endhint %}

***

## Töötlemise alustamine

### Leia käivitusnupp

Käivitus-/esitusnupp asub Chloros ülemises päisribas:

* Asukoht: akna ülemine keskkohta
* Ikoon: **Esitus-/käivitusnupp** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Seisund: Nupp on aktiveeritud (heledalt valgustatud), kui töötlemiseks on valmis

### Klõpsake käivitamiseks

1. Klõpsake ülemises päises **Esita/Käivita-nuppu**

2. Töötlemine algab kohe
3. Töötlemise ajal muutub nupp **Peata**-nupuks
4. Edusammude riba uueneb, näidates töötlemise seisundit

{% hint style="success" %}
**Töötlemine alustatud**: Klõpsamisel teostab Chloros automaatselt kõik töötlemisetapid – sihtmärgi tuvastamine, debayering, kalibreerimine, indeksi arvutamine ja eksport. See tuvastab automaatselt, kas teie projekt on Survey3, LATTICE või nende kombinatsioon, ning rakendab igale kaamerale sobivat töötlusprotsessi.
{% endhint %}

***

## Töötlemisrežiimide selgitus

Chloros töötab sõltuvalt teie litsentsist kahes erinevas töötlemisrežiimis:

### Tasuta režiim (järjestikune töötlemine)

**Kättesaadav kõigile kasutajatele**

**Kuidas see toimib:**

* Töötleb pilte ükshaaval, järjestikku
* Üheprotsessiline töö
* Väiksem mälukasutus

**Edusammude riba näitab 2 etappi:**

1.**Sihtmärgi tuvastamine** – kalibreerimisihtmärkide otsimine
2. **Töötlemine** – kalibreerimise rakendamine ja piltide eksportimine**Töötlemisaeg:**

* Palju aeglasem kui Chloros+ paralleelrežiim
* Sobib väikestele ja keskmise suurusega andmekogumitele (&lt; 200 pilti)

### Chloros+ režiim (paralleelne töötlemine)

**Nõuab Chloros+ litsentsi**

**Kuidas see töötab:**

* Töötleb mitut pilti korraga, kasutades [4-niitlist töötlemispipeliini](../processing-architecture/processing-pipeline.md)
* [Dünaamiline arvutusvõimsuse kohandamine](../processing-architecture/dynamic-compute-adaptation.md) valib käivitamisel automaatselt teie riistvarale optimaalse strateegia
* GPU (CUDA) kiirendus NVIDIA graafikakaartidega (lauaarvuti ja Jetson)
* **Töötajate arv kohandub riistvaraga**: GPU-strateegiad käitavad**1–4 samaaegset töötajat** (sõltuvalt VRAM-ist — väikese mälumahuga Jetsonil töötab 1, 12 GB+ lauaarvuti GPU-l kuni 4); ainult CPU-ga süsteemid käitavad ühte töötajat füüsilise tuuma kohta, miinus üks**Edusammude riba näitab 4 etappi** (mis vastavad 4 töötlusjärjekorra niidile):

1. **Tuvastamine** (niit 1) – kalibreerimise sihtmärkide leidmine
2. **Analüüs** (niit 2) – pildi metaandmete uurimine ja kalibreerimise arvutamine
3. **Kalibreerimine** (niit 3) – debayering, vinjeti korrigeerimine, kalibreerimine, indeksite arvutamine
4. **Eksportimine** (niit 4) – töödeldud piltide ja indeksite salvestamine**Edusammude riba kasutamine:*** **Viige hiir** riba kohale, et näha üksikasjalikku 4-etapilist rippmenüüd
* **Klõpsake** edusammude ribal, et rippmenüü paigale kinnitada
* **Klõpsake uuesti**, et kinnitus tühistada ja menüü peita**Töötlemisaeg:**

* Märkimisväärselt kiirem kui tasuta režiimis
* GPU kiirendus suurendab kiirust veelgi

{% hint style="info" %}
**Chloros+ kiirus**: Suurte andmekogumite puhul võib paralleelne töötlemine olla 5–10 korda kiirem kui järjestikune režiim. 500 pildist koosnev projekt, mille töötlemine tasuta režiimis võtab aega 2 tundi, võib Chloros+ abil valmida 15–20 minutiga.
{% endhint %}

***

## Mis toimub töötlemise ajal

### 1. etapp: sihtmärkide tuvastamine

**Mida Chloros teeb:**

* Skaneerib pilte, mille olete märkinud veerus „Sihtmärk” (kui ükski pole märgitud, siis kõiki pilte)
* Tuvastab kalibreerimispaneelid igas sihtmärgis
* Eraldab sihtmärkide paneelidelt peegeldusväärtused
* Salvestab sihtmärkide ajamärgised kalibreerimise ajakava koostamiseks

**Kestus:** 1–30 sekundit (märgitud sihtmärkidega), 5–30+ minutit (märkimata)

### 2. etapp: Debayering (RAW-failide teisendamine)

**Chloros tegevused:**

* Teisendab RAW-vormingus olevad Bayeri mustri andmed täielikeks 3-kanalilisteks piltideks (LATTICE mono-moodulid jäävad ühekanalilisteks – nende puhul jäetakse debayering vahele ja logisse lisatakse vastav märkus)
* Rakendab valitud demosaicing-algoritmi
* Säilitab maksimaalse pildikvaliteedi ja detailid

**Kestus:** Sõltub piltide arvust ja CPU/GPU kiirusest

### 3. etapp: Kalibreerimine

**Chloros tegevus:*** **Vignette-korrektsioon**: eemaldab objektiivi poolt põhjustatud tumeduse servades
* **Peegeldusvõime kalibreerimine**: normaliseerib, kasutades sihtmärgi peegeldusväärtusi ja/või DAQ allapoole suunatud andmeid
* Rakendab korrektsioone kõigile sagedusaladele/kanalitele
* Kasutab iga pildi jaoks sobivat kalibreerimisviidet, lähtudes ajamärgist

**Kestus:** Suurem osa töötlemisajast

### 4. etapp: Indeksite arvutamine

**Chloros tegevused:**

* Arvutab konfigureeritud multispektraalsed indeksid (NDVI, NDRE jne)
* Rakendab kalibreeritud piltidele lainepikkuste matemaatilisi operatsioone
* Genereerib indeksipildid iga valitud indeksi jaoks

**Kestus:** Mõni sekund pildi kohta

### 5. etapp: Eksport

**Mida teeb Chloros:**

* Salvab töödeldud pildid valitud formaadis
* **LATTICE fan-out**: iga töötlemata LATTICE-kaader eksporditakse ühe tööetapiga kõigi aktiveeritud toodetena — debayering, eelvaade, kiirgus (alati float32), peegeldus
* Kirjutab failid projekti väljundkataloogi: `<project>/<camera>/<format>/<Product>_Images/`
* **Säilitab allika failinime** — toodet identifitseerib kaust, sufiksi ei lisata**Kestus:** sõltub ekspordivormingust ja faili suurusest***

## Töötlemise käitumine

### Automaatne töötlemisprotsess

Kord käivitatuna töötab kogu töötlemisprotsess automaatselt:

* Kasutaja sekkumist ei ole vaja
* Kõik konfigureeritud etapid täidetakse järjest
* Töö edenemist näidatakse reaalajas
* Eksporditud failid salvestatakse kettale kohe, kui need valmivad — valmis väljundeid saab avada juba töötlemise käigus

### Arvuti koormus töötlemise ajal

**Vaba režiim:**

* Suhteliselt madal CPU-koormus (ühekiuline)
* Arvuti reageerib endiselt teistele ülesannetele
* Chloros-i võib ohutult minimeerida ja töötada teistes rakendustes

**Chloros+ paralleelrežiim:**

* Kõrge protsessori koormus kogu strateegia töötajate rühmas
* GPU kiirenduse korral: kõrge GPU koormus
* Arvuti võib töötlemise ajal reageerida aeglasemalt
* Vältige teiste protsessorimahukate ülesannete käivitamist

{% hint style="warning" %}
**Jõudluse näpunäide**: Parima Chloros+ jõudluse saavutamiseks sulgege teised rakendused ja laske Chloros-il kasutada kogu süsteemi ressursse.
{% endhint %}

### Töötlemist ei saa peatada (kuid lõpetamine toimub puhtalt)

* Kord alustatud töötlemist ei saa peatada ega hiljem jätkata
* Nupule **Stop** klõpsamine lõpetab käivituse puhtalt esimesel klõpsul
* Enne peatamist juba eksporditud tooted jäävad kettale alles
* Peatatud käivitus annab ausalt teada, mida on lõpetatud (vaata logis ridu `[RUN-SUMMARY]`)
* Uus käivitamine alustab töötlemisahelat algusest

**Planeerimisnõuanne:** Väga suurte projektide puhul kaaluge töötlemist partiidena või kasutage CLI-i parema kontrolli saavutamiseks.***

## Töötlemise jälgimine

Töötlemise käigus saate:

* **Jälgida edenemisriba** – vaadata üldist valmimisprotsenti
* **Vaadata praegust etappi** – tuvastamine, analüüsimine, kalibreerimine või eksportimine
* **Vaadata logi vahekaarti** – näha üksikasjalikke töötlemissõnumeid ja hoiatusi
* **Eelvaadata valmis pilte** – eksporditud failid ilmuvad töötlemise ajal kettale

Täpsemat teavet jälgimise kohta leiate jaotisest [Töötlemise jälgimine](monitoring-the-processing.md).

***

## Töötlemise peatamine

Kui soovite töötlemise peatada:

### Kuidas peatada

1. Leidke **Peata-nupp** (asendab töötlemise ajal Start-nuppu)
2. Klõpsake seda üks kord – ribal kuvatakse **„Peatamine...”**, kuni pooleliolev pilt lõpetatakse
3. Töötlus lõpeb lõplikult peatatud olekus ja logisse kirjutatakse aus ülevaade `[RUN-SUMMARY]` sellest, mis on lõpule viidud

### Millal peatada

**Põhjendatud põhjused peatamiseks:**

* Selgus, et kasutati valesid seadeid
* Unustasid märgistada sihtpildid
* Imporditi valed pildid
* Süsteem töötab liiga aeglaselt või ei reageeri

**Pärast peatamist:**

* Enne peatamist eksporditud tulemused jäävad kettale alles
* Vaata üle ja paranda võimalikud vead, kohanda seadeid vastavalt vajadusele
* Alusta töötlemist uuesti — töötlemine algab algusest

***

## Töötlemisaja hinnangud

Tegelik töötlemisaeg sõltub suuresti järgmistest teguritest:

* Piltide arv
* Pildi resolutsioon
* Sisendvorming (RAW või JPG)
* Töötlemisrežiim (Free või Chloros+)
* Protsessori kiirus ja tuumade arv
* Graafikakaardi olemasolu (ainult Chloros+)
* Arvutatavate indeksite arv
* Aktiveeritud eksporditoodete arv (LATTICE)

### Ligikaudsed hinnangud (Chloros+, 12 MP pildid, kaasaegne protsessor)

| Piltide arv | Tasuta režiim | Chloros+ (protsessor) | Chloros+ (graafikakaart) |
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

## Tavapärased probleemid käivitamisel

### Käivitusnupp on blokeeritud (halliks muutunud)

**Võimalikud põhjused:**

* Pilte ei ole imporditud
* Tagapõhi ei ole täielikult käivitatud
* Eelmine töötlemine on veel käimas
* Projekt ei ole täielikult laaditud

**Lahendused:**

1. Oodake, kuni tagapõhi on täielikult algseadistatud (kontrollige peamenüü ikooni)
2. Veenduge, et pildid on failibrauserisse imporditud
3. Kui nupp jääb blokeerituks, käivita Chloros uuesti
4. Kontrolli veateateid silumislogist

### Töötlemine algab, kuid ebaõnnestub kohe

**Võimalikud põhjused:**

* Projektis pole kehtivaid pilte
* Rikutud pildifailid
* Ebapiisav kettaruum
* Ebapiisav mälu (RAM)

**Lahendused:**

1. Kontrolli veateateid veakirjelduse logist (<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">)
2. Veendu, et kettaruum on piisav
3. Proovi töödelda väiksemat piltide alarühma
4. Veendu, et pildid ei ole rikutud

### Töötlemine lõpeb, kuid pilte ei salvestata

Käivitust, mille käigus taotleti pilditooteid, kuid ühtegi ei salvestatud, käsitletakse **ebaõnnestumisena, mitte õnnestumisena** — Chloros annab sellest selgelt teada:

* Graafilise kasutajaliidese logis kuvatakse `[RUN-SUMMARY]` vihjed, mis nimetavad tõenäolise põhjuse — pilte ei imporditud, sihtmärki ei tuvastatud või kõik taotletud tooted jäeti kõrvale kui sobimatud (nt radiaantsuse/peegeldusvõime taotlemine ainult RGB-kaameratelt)
* CLI-i ekvivalent (`chloros-cli process`) väljastab `Processing finished but wrote no image products.` ja **lõpetab töötamise nullist erineva väärtusega**, mistõttu skriptid suudavad seda tuvastada
* Tahtlikult ainult metandmetega käivitatud töö (kõik eksporditooted on keelatud, indeksid puuduvad) loetakse ikkagi edukaks

Täieliku semantikaga tutvumiseks vaata [CLI-i viidet](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Hoiatus „Sihtmärke ei tuvastatud”

**Võimalikud põhjused:**

* Unustasid sihtmärgipilte märgistada
* Sihtmärgipildid ei sisalda nähtavaid sihtmärke
* Sihtmärkide tuvastamise seaded on liiga ranged

**Lahendused:**

1. Vaadake läbi [Sihtmärkide piltide valimine](choosing-target-images.md)
2. Märgistage sobivad pildid veerus „Sihtmärk“
3. Veenduge, et märgistatud piltidel on sihtmärgid nähtavad
4. Vajadusel kohandage sihtmärkide tuvastamise seadeid

***

## Näpunäited edukaks töötlemiseks

### Enne alustamist

1. **Testi esmalt väikese alarühmaga** – töötle 10–20 pilti, et seadeid kontrollida
2. **Kontrolli vaba kettaruumi** – veendu, et vaba ruumi on 2–3 korda rohkem kui andmekogumi suurus (rohkem, kui kõik LATTICE-tooted on aktiveeritud)
3. **Sulge mittevajalikud rakendused** – vabasta süsteemiressursse
4. **Kontrolli sihtmärkide pilte** – vaata märgitud sihtmärke eelvaates, et veenduda nende kvaliteedis
5. **Salvesta projekt** – projekt salvestatakse automaatselt, kuid hea tava on salvestada ka käsitsi

### Töötlemise ajal

1. **Vältige süsteemi puhkeolekut** – lülitage energiasäästurežiimid välja
2. **Hoidke Chloros esiplaanil** – või vähemalt nähtavana tegumiribal
3. **Jälgige aeg-ajalt töökäiku** – kontrollige, kas on hoiatusi või vigu
4. **Ära lae teisi ressursimahukaid rakendusi** – Eriti Chloros+ paralleelrežiimis

### Chloros+ GPU-kiirendus

Kui kasutad NVIDIA GPU-kiirendust:

1. Uuenda NVIDIA draiverid viimase versioonini
2. Veendu, et GPU-l on vähemalt 4 GB VRAM-i (7 GB või rohkem samaaegse Texture Aware debayeringu jaoks)
3. Sulge GPU-mahukad rakendused (mängud, videotöötlus)
4. Jälgi GPU temperatuuri (tagada piisav jahutus)

***

## Järgmised sammud

Kui töötlemine on alanud:

1. **Jälgi töötlemise edenemist** – vt [Töötlemise jälgimine](monitoring-the-processing.md)
2. **Oota töötlemise lõppu** – töötlemine kulgeb automaatselt
3. **Vaadake tulemusi läbi** – vt [Töötlemise lõpetamine](finishing-the-processing.md)

Teavet selle kohta, mida töötlemise ajal teha, leiate jaotisest [Töötlemise jälgimine](monitoring-the-processing.md).
