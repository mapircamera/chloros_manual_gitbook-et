# Töötlemise alustamine

Kui olete pildid importinud, kalibreerimismärgid märkinud ja projekti seaded konfigureerinud, olete valmis töötlemist alustama. Käesolev lehekülg juhendab teid Chlorosi töötlemisprotsessi käivitamisel.

## Eeltöötlemise kontrollnimekiri

Enne nupule „Start“ klõpsamist veenduge, et kõik on valmis:

* [ ] **Failid imporditud** – kõik pildid on näha failibrauseris
* [ ] **Sihtmärgipildid märgitud** – kalibreerimispiltide puhul on märgitud veerg „Target“ (või LATTICE jaoks on imporditud `.daq`-salvestus)
* [ ] **Kaameramudelid tuvastatud** – veerus „Kaameramudel” kuvatakse õiged kaamerad
* [ ] **Seaded konfigureeritud** – projekti seaded on üle vaadatud ja kohandatud
* [ ] **Indeksid valitud** – soovitud multispektraalsed indeksid on lisatud (vajaduse korral)
* [ ] **Ekspordivorming valitud** – Teie töövoole sobiv väljundvorming

{% hint style="info" %}
**Nõuanne**: Klõpsake failibrauseris mõnel pildil, et veenduda enne töötlemist, et need on õigesti laaditud.
{% endhint %}

***

## Töötlemise alustamine

### Leia käivitusnupp

Käivitus-/esitusnupp asub Chlorosi ülemises päisribal:

* Asukoht: akna ülaosas keskel
* Ikoon: **Esitus-/käivitusnupp** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Seisund: nupp on aktiveeritud (heledalt valgustatud), kui töötlemiseks on valmis

### Klõpsake käivitamiseks

1. Klõpsake ülemises päises **Esita/Käivita-nuppu**

2. Töötlemine algab kohe
3. Töötlemise ajal muutub nupp **Peata**-nupuks
4. Edusammude riba uueneb, näidates töötlemise seisundit

{% hint style="success" %}
**Töötlemine alustatud**: Klõpsamisel teostab Chloros automaatselt kõik töötlemisetapid – sihtmärgi tuvastamine, debayering, kalibreerimine, indeksi arvutamine ja eksport. Programm tuvastab automaatselt, kas teie projekt on Survey3, LATTICE või nende kombinatsioon, ning rakendab iga kaamera jaoks õiget töötlemisprotsessi.
{% endhint %}

***

## Töötlemisrežiimide mõistmine

Chloros töötab sõltuvalt teie litsentsist kahes erinevas töötlemisrežiimis:

### Tasuta režiim (järkjärguline töötlemine)

**Kättesaadav kõigile kasutajatele**

**Kuidas see toimib:**

* Töötleb pilte ükshaaval, järjestikku
* Üheprotsessiline töö
* Väiksem mälukasutus

**Edusammude riba näitab 2 etappi:**

1.**Sihtmärkide tuvastamine** – kalibreerimisihtmärkide otsimine
2. **Töötlemine** – kalibreerimise rakendamine ja piltide eksportimine**Töötlemisaeg:**

* Palju aeglasem kui Chloros+ paralleelrežiim
* Sobib väikestele ja keskmise suurusega andmekogumitele (< 200 pilti)

### Chloros+ režiim (paralleelne töötlemine)

**Nõuab Chloros+ litsentsi**

**Kuidas see toimib:**

* Töötleb mitut pilti samaaegselt, kasutades [4-niitlist töötlemispipeliini](../processing-architecture/processing-pipeline.md)
* [Dünaamiline arvutusvõimsuse kohandamine](../processing-architecture/dynamic-compute-adaptation.md) valib käivitamisel automaatselt teie riistvarale optimaalse strateegia
* GPU (CUDA) kiirendus NVIDIA graafikakaartidega
* **Töötajate arv kohandub riistvaraga**: GPU-strateegiad käitavad**2–4 samaaegset töötajat** (sõltuvalt VRAM-ist — 3 2–12 GB GPU-l, 2–4 12 GB+ GPU-l); Ainult CPU-põhised süsteemid käitavad ühte tööprotsessi iga füüsilise tuuma kohta, miinus üks**Edusammude riba näitab 4 etappi** (vastavalt 4 töötlemisjärjekorra niidile):

1. **Tuvastamine** (niit 1) – kalibreerimise sihtmärkide leidmine
2. **Analüüs** (niit 2) – pildi metaandmete uurimine ja kalibreerimise arvutamine
3. **Kalibreerimine** (niit 3) – debayering, vinjeti korrigeerimine, kalibreerimine, indeksi arvutamine
4. **Eksportimine** (niit 4) – töödeldud piltide ja indeksite salvestamine**Edusammude riba kasutamine:*** **Viige hiir** riba kohale, et näha üksikasjalikku 4-etapilist rippmenüüd
* **Klõpsake** edusammude ribal, et rippmenüü paigale fikseerida
* **Klõpsake uuesti**, et fikseeringut tühistada ja rippmenüü peita**Töötlemisaeg:**

* Märkimisväärselt kiirem kui tasuta režiimis
* GPU kiirendus parandab kiirust veelgi

{% hint style="info" %}
**Chloros+ kiirus**: Suurte andmekogumite puhul võib paralleeltöötlus olla 5–10 korda kiirem kui järjestikune režiim. 500 pildiga projekt, mis tasuta režiimis võtab aega 2 tundi, võib Chloros+ abil valmida 15–20 minutiga.
{% endhint %}

***

## Mis toimub töötlemise ajal

### 1. etapp: sihtmärkide tuvastamine

**Mida Chloros teeb:**

* Skaneerib pilte, mille olete märkinud veerus „Sihtmärk“ (kui ükski pole märgitud, siis kõiki pilte)
* Tuvastab kalibreerimispaneelid igas sihtmärgis
* Eraldab sihtmärkide paneelidelt peegeldusväärtused
* Salvestab sihtmärkide ajamärgised kalibreerimise ajakava koostamiseks

**Kestus:** 1–30 sekundit (märgitud sihtmärkidega), 5–30+ minutit (märkimata)

### 2. etapp: Debayering (RAW-muundamine)

**Mida Chloros teeb:**

* Konverteerib RAW-vormingus olevad Bayeri mustri andmed täielikeks 3-kanalilisteks piltideks (LATTICE mono-moodulid jäävad ühekanalilisteks – nende puhul jäetakse debayering vahele ja logisse lisatakse vastav märkus)
* Rakendab valitud demosaicing-algoritmi
* Säilitab maksimaalse pildikvaliteedi ja detailid

**Kestus:** Sõltub piltide arvust ja CPU/GPU kiirusest

### 3. etapp: Kalibreerimine

**Mida Chloros teeb:*** **Vignette-korrektsioon**: Eemaldab objektiivi poolt põhjustatud tumenemise servades
* **Peegeldusvõime kalibreerimine**: normaliseerib sihtmärgi peegeldusväärtuste ja/või DAQ allapoole suunatud andmete abil
* Rakendab parandusi kõigile lainepikkustele/kanalitele
* Kasutab iga pildi jaoks ajamärgise põhjal sobivat kalibreerimisviidet

**Kestus:** Suurem osa töötlemisajast

### 4. etapp: Indeksi arvutamine

**Mida Chloros teeb:**

* Arvutab konfigureeritud multispektraalsed indeksid (NDVI, NDRE jne)
* Rakendab lainepikkuste matemaatilisi operatsioone kalibreeritud piltidele
* Genereerib indeksipildid iga valitud indeksi jaoks

**Kestus:** Mõni sekund pildi kohta

### 5. etapp: eksport

**Mida Chloros teeb:**

* Salvestab töödeldud pildid valitud formaadis
* **LATTICE fan-out**: iga töötlemata LATTICE-kaader eksporditakse ühe tööga kõigi aktiveeritud toodetena — debayered, eelvaade, radiance (alati float32), reflectance
* Kirjutab failid projekti väljundkataloogi: `<project>/<camera>/<format>/<Product>_Images/`
* **Säilitab allika failinime** — toodet identifitseerib kaust, sufiksi ei lisata**Kestus:** sõltub ekspordiformaadist ja faili suurusest***

## Töötlemise käitumine

### Automaatne töötlemisprotsess

Käivitamisel töötab kogu protsess automaatselt:

* Kasutaja sekkumist ei ole vaja
* Kõik konfigureeritud etapid täidetakse järjest
* Töö edenemist näidatakse reaalajas
* Eksporditud failid salvestatakse kettale kohe, kui need valmivad — saate avada valmis väljundeid juba enne protsessi lõppu

### Arvuti koormus töötlemise ajal

**Vaba režiim:**

* Suhteliselt madal CPU koormus (üheprotsessiline)
* Arvuti reageerib endiselt teistele ülesannetele
* Chloros'i võib ohutult minimeerida ja töötada teistes rakendustes

**Chloros+ paralleelrežiim:**

* Kõrge protsessori koormus kogu strateegia töötajate rühmas
* GPU kiirenduse korral: kõrge GPU koormus
* Arvuti võib töötlemise ajal reageerida aeglasemalt
* Vältige teiste protsessorimahukate ülesannete käivitamist

{% hint style="warning" %}
**Jõudluse näpunäide**: Parima Chloros+ jõudluse saavutamiseks sulgege teised rakendused ja laske Chlorosil kasutada kõiki süsteemi ressursse.
{% endhint %}

### Töötlemist ei saa peatada (kuid lõpetamine toimub puhtalt)

* Kord käivitatud töötlemist ei saa peatada ega hiljem jätkata
* Nupule **Stop** klõpsamine peatab töötlemise täielikult juba esimesel klõpsul
* Enne peatamist juba eksporditud tulemused jäävad kettale alles
* Peatatud töötlemine annab ausa ülevaate sellest, mis on lõpule viidud (vaata logis ridu `[RUN-SUMMARY]`)
* Uus käivitamine alustab töötlemisprotsessi algusest

**Planeerimisnõuanne:** väga suurte projektide puhul kaaluge töötlemist partiidena või kasutage CLI-d parema kontrolli saavutamiseks.***

## Töötlemise jälgimine

Töötlemise käigus saate:

* **Jälgida edenemisriba** – vaadata üldist valmimisprotsenti
* **Vaadata praegust etappi** – tuvastamine, analüüsimine, kalibreerimine või eksportimine
* **Kontrollida logi vahekaarti** – vaadata üksikasjalikke töötlemissõnumeid ja hoiatusi
* **Eelvaadata valmis pilte** – eksporditud failid ilmuvad kettale juba töötlemise ajal

Täpsemat teavet jälgimise kohta leiate jaotisest [Töötlemise jälgimine](monitoring-the-processing.md).

***

## Töötlemise peatamine

Kui peate töötlemise peatama:

### Kuidas peatada

1. Leidke **Peata-nupp** (asendab töötlemise ajal nuppu „Start”)
2. Klõpsake seda üks kord – ribal kuvatakse **„Peatamine...”**, kuni töötlemisel olev pilt lõpetatakse
3. Töötlemine lõpeb lõplikult peatatud olekus ja logisse kirjutatakse aus ülevaade `[RUN-SUMMARY]` sellest, mis on lõpetatud

### Millal peatada

**Põhjendatud põhjused peatamiseks:**

* Selgus, et kasutati valesid seadeid
* Unustati märgistada sihtpildid
* Imporditi valed pildid
* Süsteem töötab liiga aeglaselt või ei reageeri

**Pärast peatamist:**

* Enne peatamist eksporditud tulemused jäävad kettale
* Vaata üle ja paranda võimalikud vead, kohanda seadeid vastavalt vajadusele
* Käivita töötlemine uuesti — töö algab algusest

***

## Töötlemisaja hinnangud

Tegelik töötlemisaeg varieerub oluliselt sõltuvalt järgmistest teguritest:

* Piltide arv
* Pildi resolutsioon
* Sisendvorming (RAW või JPG)
* Töötlemisrežiim (Free vs Chloros+)
* Protsessori kiirus ja tuumade arv
* Graafikakaardi olemasolu (ainult Chloros+)
* Arvutatavate indeksite arv
* Aktiveeritud eksporditoodete arv (LATTICE)

### Ligikaudsed hinnangud (Chloros+, 12 MP pildid, kaasaegne protsessor)

| Piltide arv | Tasuta režiim | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 pilti   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 pilti  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 pilti  | 1–1,5 tundi | 20–30 min      | 10–15 min      |
| 500 pilti  | 2–3 tundi   | 45–60 min      | 20–30 min      |
| 1000 pilti | 4–6 tundi   | 1,5–2 tundi      | 40–60 minutit      |

{% hint style="info" %}
**Esimene käivitamine**: Esmane töötlemine võib võtta kauem aega, kuna Chloros loob vahemälusid ja profiile. Sarnaste andmekogumite edasine töötlemine toimub kiiremini.
{% endhint %}

***

## Tüüpilised probleemid käivitamisel

### Käivitusnupp on blokeeritud (halliks muutunud)

**Võimalikud põhjused:**

* Pilte pole imporditud
* Tagapõhi pole täielikult käivitatud
* Eelmine töötlemine on veel käimas
* Projekt pole täielikult laaditud

**Lahendused:**

1. Oota, kuni taustsüsteem on täielikult algseadistatud (kontrolli peamenüü ikooni)
2. Veendu, et pildid on failibrauserisse imporditud
3. Käivita Chloros uuesti, kui nupp jääb blokeerituks
4. Kontrolli veateateid silumislogist

### Töötlemine algab, kuid ebaõnnestub kohe

**Võimalikud põhjused:**

* Projektis pole kehtivaid pilte
* Rikutud pildifailid
* Ebapiisav kettaruum
* Ebapiisav mälu (RAM)

**Lahendused:**

1. Kontrollige veateateid silumislogist <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Kontrollige, kas kettaruum on piisav
3. Proovige töödelda väiksemat piltide alarühma
4. Veenduge, et pildid ei ole rikutud

### Töötlemine lõpeb, kuid pilte ei salvestata

Töötlemine, mille käigus taotleti pilditooteid, kuid ühtegi neist ei salvestatud, loetakse **ebaõnnestumiseks, mitte õnnestumiseks** — Chloros annab sellest selgelt teada:

* Graafilise kasutajaliidese logis kuvatakse veakoodid `[RUN-SUMMARY]`, mis viitavad tõenäolisele põhjusele — pilte ei imporditud, sihtmärki ei tuvastatud või kõik taotletud tooted jäeti kõrvale kui kohaldamatud (nt radiaantsuse/peegeldusvõime taotlemine ainult RGB-kaameratelt)
* Vastav CLI-kood (`chloros-cli process`) kuvab veateate `Processing finished but wrote no image products.` ja **lõpetab töötamise nullist erineva tulemusega**, mistõttu skriptid suudavad seda tuvastada
* Tahtlikult ainult metaandmetele suunatud käivitus (kõik eksporditooted on keelatud, indeksid puuduvad) loetakse ikkagi edukaks

Täieliku semantikaga tutvumiseks vaadake [CLI-viidet](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Projektikaust on selle arvuti jaoks liiga sügav

**Ainult Windows, uus alates versioonist 1.3.0.** Windowsis on pikad teed vaikimisi välja lülitatud ning seetõttu ei tohi ükski failitee olla pikem kui 260 tähemärki. Chlorosi tooted asuvad kuni umbes 130 tähemärki projektikausta allpool, seega ei saa projektikausta, mis asub sügavamal, oma tooteid vastu võtta. Selle asemel, et käivituda ja midagi kirjutamata jätta, kontrollib Chloros nüüd enne töötlemise algust:

* Nupule „Start“ klõpsamine ei käivita töötlemist: edenemisribal kuvatakse põhjus ja vealokis kuvatakse `[PROCESS] refused: The project folder is too deep for this computer…`. Sõnumis näidatakse projektikausta pikkust ja pikimat sobivat projektikausta.
* Projekti loomine või avamine kaustas, mis on liiga sügav isegi projekti enda failide jaoks, lükatakse samuti tagasi. Teateaknas selgitatakse põhjust, näidatakse sobivaim pikkusega projektikausta nime ja antakse juhised pikkade teede lubamiseks; midagi ei looda ega avata; veakiri salvestab selle koodina `[PROJECT] refused: …`. Projekti kopeerimine pikema nimega võib samuti piirmäära ületada: koopia luuakse, kuid seda ei avata, ning sama teade selgitab põhjust.

**Lahendused:**

1. Hoidke projekte lühemas kaustas — muutke**töökataloogi** [Projekti seaded](../project-settings/project-settings.md) all ja looge projekt uuesti, või
2. Lülita sisse Windowsi pikad teed ja käivita Chloros uuesti: käivita Chlorosi installija uuesti, valides valiku **Enable Windows long paths** (see on vaikimisi märgitud), või kasuta grupipoliitikat: *Arvuti konfiguratsioon > Haldusmallid > Süsteem > Failisüsteem > Enable Win32 long paths*.

### Hoiatus „Sihtmärke ei leitud”

**Võimalikud põhjused:**

* Unustasite sihtmärgipilte märgistada
* Sihtmärgipildid ei sisalda nähtavaid sihtmärke
* Sihtmärkide tuvastamise seaded on liiga ranged

**Lahendused:**

1. Vaadake läbi [Sihtmärgipiltide valimine](choosing-target-images.md)
2. Märgista sobivad pildid veerus „Sihtmärk“
3. Veendu, et märgistatud piltidel on sihtmärgid nähtavad
4. Vajaduse korral kohanda sihtmärkide tuvastamise seadeid

***

## Näpunäited edukaks töötlemiseks

### Enne alustamist

1. **Testige esmalt väikese alarühmaga** – töötlege 10–20 pilti, et seadeid kontrollida
2. **Kontrollige vaba kettaruumi** – tagage, et vaba ruumi oleks 2–3 korda andmekogumi suurusest rohkem (rohkem, kui kõik LATTICE-tooted on aktiveeritud)
3. **Sulge mittevajalikud rakendused** – vabasta süsteemiressursse
4. **Kontrolli sihtmärkide pilte** – vaata märgitud sihtmärke eelvaates, et veenduda nende kvaliteedis
5. **Salvesta projekt** – projekt salvestatakse automaatselt, kuid hea tava on salvestada ka käsitsi

### Töötlemise ajal

1. **Väldi süsteemi puhkeolekut** – lülita energiasäästurežiimid välja
2. **Hoia Chloros esiplaanil** – või vähemalt nähtavana tegumiribal
3. **Jälgi aeg-ajalt töökäiku** – kontrolli, kas on hoiatusi või vigu
4. **Ära lae teisi ressursimahukaid rakendusi** – eriti Chloros+ paralleelrežiimis

### Chloros+ GPU-kiirendus

Kui kasutate NVIDIA GPU-kiirendust:

1. Uuendage NVIDIA draiverid viimasele versioonile
2. Veenduge, et GPU-l on vähemalt 4 GB VRAM-i (7 GB või rohkem samaaegseks tekstuuriteadlikuks debayeringuks)
3. Sulgege GPU-mahukad rakendused (mängud, videotöötlus)
4. Jälgige GPU temperatuuri (tagage piisav jahutus)

***

## Järgmised sammud

Kui töötlemine on alanud:

1. **Jälgige töötlemise kulgu** – vt [Töötlemise jälgimine](monitoring-the-processing.md)
2. **Oodake töötlemise lõppu** – töötlemine kulgeb automaatselt
3. **Vaadake tulemusi** – vt [Töötlemise lõpetamine](finishing-the-processing.md)

Teavet selle kohta, mida töötlemise ajal teha, leiate jaotisest [Töötlemise jälgimine](monitoring-the-processing.md).
