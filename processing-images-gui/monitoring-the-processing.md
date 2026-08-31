# Töötlemise jälgimine

Kui töötlemine on alanud, pakub Chloros mitmeid võimalusi protsessi kulgemise jälgimiseks, probleemide tuvastamiseks ja andmekogumi seisundi mõistmiseks. Käesoleval lehel selgitatakse, kuidas jälgida töötlemist ja tõlgendada Chloros poolt pakutavat teavet.

## Edusammude riba ülevaade

Ülemises päises asuv edusammude riba näitab töötlemise reaalajas seisundit ja valmimisprotsenti. Edusammude andmed edastatakse reaalajas tagapõhjast Server-Sent Events (SSE) kaudu, seega peegeldab riba seda, mida töötlemisprotsess tegelikult teeb.

### Vaba režiimi edusammude riba

Kasutajatele, kellel puudub Chloros+ litsents:

**2-etapiline edusammude kuvamine:**

1.**Sihtmärgi tuvastamine** – kalibreerimisihtmärkide leidmine piltidelt
2. **Töötlemine** – paranduste rakendamine ja eksportimine**Edusammude riba näitab:**

* Üldine valmimisprotsent (0–100%)
* Praeguse etapi nimi
* Lihtne horisontaalne riba

### Chloros+ edenemisriba

Kasutajatele, kellel on Chloros+ litsents:

**4-etapiline edenemisnäidik:**

1.**Tuvastamine** – kalibreerimismärkide leidmine
2. **Analüüsimine** – piltide uurimine ja töötlemisprotsessi ettevalmistamine
3. **Kalibreerimine** – vinjett- ja peegeldusparanduste kohaldamine
4. **Eksportimine** – töödeldud failide salvestamine**Interaktiivsed funktsioonid:*** **Viige hiir** edusammuriba kohale, et näha laienenud 4-etapilist paneeli
* **Klõpsake** edusammuriba peal, et laienenud paneel fikseerida
* **Klõpsake uuesti**, et fikseering tühistada ja paneel hiire eemaldamisel automaatselt peita
* Iga etapp näitab individuaalset edenemist (0–100%)

{% hint style="info" %}
**CLI pariteet**: `chloros-cli process` käivituse ajal näitavad samad neli niiti seisundit „Detecting“, „Analyzing“, „Processing“ ja „Exporting“, ning `chloros-cli export-status` näitab teise terminali kaudu reaalajas niidi 4 ekspordi edenemist. Vaata [CLI viidet](../reference/cli-reference.md).
{% endhint %}

***

## Iga töötlemisetapi mõistmine

{% hint style="info" %}
**Pipeline-arhitektuur**: Need 4 GUI-etappi vastavad [4-niitilisele töötlemispipeline&#x27;ile](../processing-architecture/processing-pipeline.md). GPU-kiirendusega süsteemides kasutab 3. niit (kalibreerimine) [dünaamilist arvutusadapteerimist](../processing-architecture/dynamic-compute-adaptation.md), mis optimeerib töötlemist teie konkreetse riistvara jaoks.
{% endhint %}

### Etapp 1: tuvastamine (sihtmärgi tuvastamine)

**Mis toimub:**

* Chloros skaneerib pilte, mille olete märkinud valikukastiga „Sihtmärk“ (kõik pildid, kui ükski pole märgitud)
* Arvutinägemise algoritmid tuvastavad kalibreerimispaneelid
* Iga paneeli peegeldusväärtused ekstraheeritakse
* Sihtmärkide ajamärgised salvestatakse õige kalibreerimise ajastamiseks

**Kestus:**

* Märgitud sihtmärkidega: 10–60 sekundit
* Märkimata sihtmärkidega: 5–30+ minutit (skannitakse kõiki pilte)

**Edusammude indikaator:**

* Tuvastamine: 0% → 100%
* Skaneeritud piltide arv (loendab ainult tegelikult skaneeritud pilte)
* Leitud sihtmärkide arv

**Mida tuleks jälgida:**

* Kui sihtmärgid on korralikult märgistatud, peaks protsess kiiresti lõppema
* Kui protsess võtab liiga kaua aega, ei pruugi sihtmärgid olla märgistatud
* Kontrolli vealokist sõnumeid „Target found“

### 2. etapp: analüüsimine

**Mis toimub:**

* Pildi EXIF-metadata lugemine (ajamärgid, säri seaded)
* Kalibreerimisstrateegia kindlaksmääramine sihtmärkide ajamärkide ja kättesaadavate DAQ allapoole suunatud andmete põhjal
* Pilditöötlusjärjekorra korraldamine
* Paralleelse töötlemise töötajate ettevalmistamine (ainult Chloros+)

**Kestus:** 5–30 sekundit**Edusammude näitaja:**

* Analüüsimine: 0% → 100%
* Kiire etapp, lõpeb tavaliselt kiiresti

**Mida jälgida:**

* Protsess peaks kulgema ühtlaselt ilma pausideta
* Hoiatused puuduvate metaandmete kohta ilmuvad vealogisse

### 3. etapp: Kalibreerimine

**Mis toimub:*** **Debayering**: RAW-Bayer-mustri teisendamine 3 kanaliks (LATTICE mono-moodulite puhul vaheletään, koos märkusega)
* **Vignette&#x27;i korrigeerimine**: objektiivi servade tumenemise eemaldamine
* **Peegeldusvõime kalibreerimine**: normaliseerimine sihtväärtuste ja/või DAQ allapoole suunatud kiirgusega
* **Indeksi arvutamine**: multispektraalsete indeksite arvutamine
* Iga pildi töötlemine kogu töötlusprotsessi vältel

**Kestus:** Suurem osa kogu töötlemisajast (60–80%)**Edusammude näitaja:**

* Kalibreerimine: 0% → 100%
* Hetkel töödeldav pilt
* Töödeldud pildid / Piltide koguarv

**Töötlemiskäitumine:*** **Vaba režiim**: Töötleb järjest ühte pilti korraga
* **Chloros+ režiim**: Käivitab riistvarale kohanduva töötajate rühma — 1–4 samaaegset töötajat GPU-süsteemidel (VRAM-i järgi), üks töötaja füüsilise tuuma kohta (miinus üks) ainult CPU-ga süsteemidel. Vaata [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-kiirendus**: kiirendab seda etappi märkimisväärselt**Mida jälgida:**

* Pidev edasiliikumine piltide arvu järgi
* Kontrolli silumislogist iga pildi lõpetamise teateid
* Hoiatused pildi kvaliteedi või kalibreerimisprobleemide kohta

### 4. etapp: eksportimine

**Mis toimub:**

* Töödeldud pildid kirjutatakse valitud formaadis kettale kohe, kui need valmivad
* **LATTICE**: iga kaader jaguneb kõikide aktiveeritud toodete vahel (debayered / eelvaade / kiirgus / peegeldus)
* Multispektraalsete indeksipiltide eksportimine LUT-värvidega
* Väljundpuude `<project>/<camera>/<format>/<Product>_Images/` loomine — eksporditud failid säilitavad allika failinime; kaust määrab toote

**Kestus:** 10–20% kogu töötlemisajast**Edusammude indikaator:**

* Eksportimine: 0% → 100%
* Failide kirjutamine
* Ekspordivorming ja sihtkoht

**Mida jälgida:**

* Kõvakettaruumi hoiatused
* Failide kirjutamisvead
* Kõigi konfigureeritud väljundite valmimine

***

## Vahekaart „Debug Log”

Veakontrolli logi pakub üksikasjalikku teavet töötlemise edenemise ja esinenud probleemide kohta. Ka tagapõhja käivitumissõnumid kuvatakse logikonsolis, seega annab logi tervikliku ülevaate isegi siis, kui avate selle hiljem.

### Veakontrolli logile juurdepääs

1. Klõpsake vasakul külgribal ikooni **Veakontrolli logi**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">


2. Avaneb logipaneel, mis kuvab reaalajas töötlemissõnumeid
3. Logi keritakse automaatselt, et näidata viimaseid sõnumeid

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Logisõnumite mõistmine

Chloros logiridade ees on sulgudes märksõnad, mis nimetavad alamsüsteemi — näiteks `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Kõige olulisem on teada **käivituse kokkuvõtet**, mis kuvatakse iga käivituse lõpus (sh peatatud käivitused):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Lisaks järgnevad `[RUN-SUMMARY]` vihjed, kui midagi vajab selgitust — näiteks jooks, mis ei andnud tulemust, või kaamera, mille taotletud toode jäeti kõrvale, kuna see ei olnud kohaldatav. `[EXPORT-CHECK]`-read selgitavad kaamerapõhiseid vahelejätmisi (nt miks RGB-kaamera ei saanud kiirguse tulemust).

Sõnumite üldised tõsidusastmed (allpool toodud näited on illustratiivsed, mitte sõna-sõnalt):

#### Teabesõnumid (valge/hall)

Tavalised töötlemise uuendused: töötlemine alustatud, sihtmärgid tuvastatud (koos paneelide arvuga), pildi kaupa kalibreerimise edenemine, failid eksporditud, töötlemine lõpetatud.

#### Hoiatussõnumid (kollased)

Mittekriitilised probleemid, mis ei peata töötlemist – nt GPS-andmete puudumine kaadris, suur ajamärkide vahe sihtmärkide piltide vahel või madal kontrast kalibreerimispaneelis.

**Tegevus:** Vaadake hoiatusi pärast töötlemist läbi, kuid ärge katkestage protsessi

#### Veateated (Red)

Kriitilised probleemid, mis võivad põhjustada töötlemise ebaõnnestumise – nt kett täis, rikutud pildifail või sihtmärkide tuvastamata jäämine, kui nõuti peegelduskalibreerimist.

**Tegevus:** Peata töötlemine, kõrvalda viga, alusta uuesti

### Tüüpilised logiolukorrad

| Olukord                             | Tähendus                                       | Vajalik tegevus                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Sihtmärk tuvastatud failis \[failinimi]        | Kalibreerimisihtmärk leitud edukalt         | Puudub – normaalne                                         |
| Pildi kaupa kuvatavad edenemisjooned              | Praeguse edenemise uuendus                       | Puudub – normaalne                                         |
| Sihtmärke ei leitud                      | Kalibreerimisihtmärke ei tuvastatud               | Märgi sihtmärgipildid või lülita peegelduskalibreerimine välja |
| Kõvakettal ei ole piisavalt ruumi               | Väljundandmete salvestamiseks pole piisavalt ruumi                 | Vabasta kõvakettal ruumi                                    |
| Rikutud faili vahelejätmine               | Pildifail on rikutud                         | Kopeerige fail uuesti SD-kaardilt                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Pildistust, millel puudub toorkaader, ei saa töödelda | Pildistage uuesti toorkaadriga, või kasutage CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Töötlemine ei andnud pilditooteid — teatati veana koos vihjetega | Lugege vihjeridu; kontrollige, mis jäeti vahele ja miks |

### Logiandmete kopeerimine

Logi kopeerimiseks veaotsinguks või tugiabi saamiseks:

1. Avage paneel „Debug Log“
2. Klõpsake nuppu **„Copy Log“** (või paremklõpsake → Valige kõik)
3. Kleepige tekstifaili või e-kirja
4. Saada vajadusel MAPIR tugiteenistusele

***

## Süsteemiressursside seire

### CPU kasutamine

**Vaba režiim:**

* 1 CPU tuum töötab ~100% koormusega
* Ülejäänud tuumad on ooterežiimis või vabad
* Süsteem reageerib endiselt

**Chloros+ paralleelrežiim:**

* Mitme tuuma kasutamine on kõrge — nende arv sõltub [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) poolt valitud strateegiast
* Süsteem võib tunduda vähem reageeriv

**Jälgimiseks:**

* Windows Ülesannete haldur (Ctrl+Shift+Esc)
* Vahekaart „Jõudlus“ → jaotis „CPU“
* Otsi protsesse „Chloros“ või „chloros-backend“

### Mälu (RAM) kasutamine

**Tüüpiline kasutamine:**

* Väikesed projektid (&lt; 100 pilti): 2–4 GB
* Keskmise suurusega projektid (100–500 pilti): 4–8 GB
* Suured projektid (üle 500 pildi): 8–16 GB
* Paralleelrežiim Chloros+ kasutab rohkem RAM-i

**Kui mälu on vähe:**

* Töötle väiksemaid partiisid
* Sulge teised rakendused
* Suurenda RAM-i mahtu, kui töötled regulaarselt suuri andmekogumeid

### GPU kasutamine (Chloros+ koos CUDA-ga)

Kui GPU-kiirendus on sisse lülitatud:

* NVIDIA GPU kasutustase on kõrge (60–90%)
* VRAM-i kasutamine suureneb (nõuab vähemalt 4 GB VRAM-i; samaaegse tekstuuriteadliku debayeringu puhul vähemalt 7 GB)
* Kalibreerimisetapp on märkimisväärselt kiirem

**Jälgimiseks:**

* NVIDIA süsteemiriba ikoon
* Ülesannete haldur → Jõudlus → GPU
* GPU-Z või sarnane seiretööriist

### Ketta sisend-väljund

**Mida oodata:**

* Suur ketta lugemiskoormus analüüsietapi ajal
* Eksportimise etapis on kõrge kettale kirjutamise koormus
* SSD on märkimisväärselt kiirem kui HDD

**Jõudluse näpunäide:**

* Kasutage projekti kausta jaoks võimaluse korral SSD-d
* Vältige võrgukettasid suurte andmekogumite puhul
* Veenduge, et kettal ei oleks ruumi peaaegu täis (mõjutab kirjutuskiirust)

***

## Probleemide tuvastamine töötlemise ajal

### Hoiatavad märgid

**Töö seiskub (5+ minutit muutusi pole):**

* Kontrollige veaotsingulogist, kas seal on vigu
* Kontrollige, kas kettal on piisavalt vaba ruumi
* Kontrollige ülesannete haldurist, kas protsess Chloros töötab

**Veateated ilmuvad sageli:**

* Peata töötlemine ja vaata vead üle
* Tavapärased põhjused: kettaruum, rikutud failid, mäluprobleemid
* Vaata allpool olevat jaotist „Vigade kõrvaldamine”

**Süsteem ei reageeri:**

* Chloros+ paralleelrežiim kasutab liiga palju ressursse
* Kaalu samaaegsete ülesannete arvu vähendamist või riistvara uuendamist
* Vaba režiim on ressursisäästlikum

### Millal töötlemine peatada

Peata töötlemine, kui näed:

* ❌ Veateateid „Kettaruum täis” või „Faili ei saa kirjutada”
* ❌ Korduvaid pildifailide rikkumisvigu
* ❌ Süsteem on täielikult hangunud (ei reageeri)
* ❌ Saite aru, et seadistused olid valed
* ❌ Imporditi valed pildid

**Kuidas peatada:**

1. Klõpsake**Peata-nuppu** (asendab Käivita-nuppu) – piisab ühest korrast
2. Ribal kuvatakse „Peatamine...“, kuni käimasolev pilt lõpeb, seejärel lõpeb töötlus peatatud olekus
3. Juba eksporditud tooted jäävad kettale; logisse kirjutatakse aus ülevaade `[RUN-SUMMARY]` sellest, mis lõpetati
4. Parandage vead ja alustage uuesti — töötlus algab algusest

***

## Töötlemise ajal esinevate probleemide lahendamine

### Töötlemine on väga aeglane

**Võimalikud põhjused:**

* Märkimata sihtpildid (kõik pildid skaneeritakse)
* SSD-mäluseadme asemel kasutatakse HDD-d
* Ebapiisavad süsteemiressursid
* Konfigureeritud on palju indekseid
* Võrguketta kasutamine

**Lahendused:**

1. Kui protsess on just alanud ja asub tuvastamisetapis: peatage, märgistage sihtmärgid, käivitage uuesti
2. Edaspidiseks: kasutage SSD-d, vähendage indeksite arvu, uuendage riistvara
3. Kaaluge CLI kasutamist suurte andmekogumite partiitöötlemiseks

### Hoiatused „Kettaruum”

**Lahendused:**

1. Vabasta kohe kettaruumi
2. Viia projekt kettale, kus on rohkem ruumi
3. Vähenda eksporditavate indeksite arvu
4. Lülita välja LATTICE eksporditooted, mida sa ei vaja (Projekti seaded → Töötlemine)
5. Kasutage TIFF asemel JPG-vormingut (väiksemad failid)

### Sagedased „Rikutud faili” teated

**Lahendused:**

1. Kopeerige pildid SD-kaardilt uuesti, et tagada nende terviklikkus
2. Kontrollige SD-kaarti vigade suhtes
3. Eemalda rikutud failid projektist
4. Jätka ülejäänud piltide töötlemist

### Süsteemi ülekuumenemine / jõudluse piiramine

**Lahendused:**

1. Tagada piisav ventilatsioon
2. Puhasta arvuti ventilatsiooniaukudest tolm
3. Vähenda töötlemiskoormust (kasutage Chloros+ asemel vaba režiimi)
4. Töötlege päeva jahedamatel aegadel

***

## Töötlemise lõpetamise teade

Kui töötlemine lõpeb:

* Edusammuriba jõuab 100%ni
* Debug-logisse ilmuvad read `[RUN-SUMMARY]` koos lõplike arvudega
* Nupp „Start“ muutub uuesti aktiivseks
* Kõik väljundfailid asuvad projekti kaamerapõhises väljundpuus: `<project>/<camera>/<format>/<Product>_Images/`

***

## Järgmised sammud

Kui töötlemine on lõppenud:

1. **Tulemuste läbivaatamine** – vaata [Töötlemise lõpetamine](finishing-the-processing.md)
2. **Väljundkausta kontrollimine** – veendu, et kõik failid on õigesti eksporditud
3. **Debug-logi läbivaatamine** – kontrolli, kas seal on hoiatusi või vigu
4. **Vaata töödeldud pilte eelvaates** – kasuta pildivaatajat või välist tarkvara

Töödeldud tulemuste läbivaatamise ja kasutamise kohta vaata [Töötlemise lõpetamine](finishing-the-processing.md).
