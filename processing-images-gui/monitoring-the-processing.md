# Töötlemise jälgimine

Kui töötlemine on alanud, pakub Chloros mitmeid võimalusi protsessi kulgemise jälgimiseks, probleemide tuvastamiseks ja andmekogumi seisundi mõistmiseks. Sellel lehel selgitatakse, kuidas jälgida töötlemist ja tõlgendada Chloros i poolt pakutavat teavet.

## Edasijõudmise riba ülevaade

Ülemises päises asuv edusammude riba näitab töötlemise seisundit ja valmimisprotsenti reaalajas. Edusammude andmed edastatakse serverist reaalajas Server-Sent Events (SSE) kaudu, seega peegeldab riba seda, mida töötlemisahel tegelikult teeb.

### Vaba režiimi edusammude riba

Chloros+ litsentsita kasutajatele:

**2-etapiline edusammude kuvamine:**

1.**Sihtmärkide tuvastamine** – kalibreerimisihtmärkide leidmine piltidelt
2. **Töötlemine** – paranduste rakendamine ja eksportimine**Edusammude riba näitab:**

* Üldine valmimisprotsent (0–100%)
* Praeguse etapi nimi
* Lihtne horisontaalne riba

### „Chloros+“ edenemisriba

Kasutajatele, kellel on „Chloros+“ litsents:

**4-etapiline edenemise kuvamine:**

1.**Tuvastamine** – kalibreerimise sihtmärkide leidmine
2. **Analüüsimine** – piltide uurimine ja töötlemisprotsessi ettevalmistamine
3. **Kalibreerimine** – vinjeti- ja peegeldusparanduste rakendamine
4. **Eksportimine** – töödeldud failide salvestamine**Interaktiivsed funktsioonid:*** **Viige hiir** edastusriba kohale, et näha laiendatud 4-etapilist paneeli
* **Klõpsake** edastusribal, et laiendatud paneel fikseerida
* **Klõpsake uuesti**, et lukustus tühistada ja paneel hiire eemaldamisel automaatselt peita
* Iga etapp näitab individuaalset edenemist (0–100%)

{% hint style="info" %}
**Pariteet CLI-ga**: `chloros-cli process` käivituse ajal kuvatakse samad neli protsessiniiti kui „Detecting“, „Analyzing“, „Processing“ ja „Exporting“, ning `chloros-cli export-status` näitab teise terminali kaudu reaalajas protsessiniidi 4 ekspordi edenemist. Vaata [CLI-viidet](../reference/cli-reference.md).
{% endhint %}

***

## Iga töötlemisetapi mõistmine

{% hint style="info" %}
**Töötlemisprotsessi arhitektuur**: Need 4 GUI-etappi vastavad [4-niitilisele töötlemisprotsessile](../processing-architecture/processing-pipeline.md). GPU-kiirendusega süsteemides kasutab 3. niit (kalibreerimine) [dünaamilist arvutusadapteerimist](../processing-architecture/dynamic-compute-adaptation.md), mis optimeerib töötlemist teie konkreetse riistvara jaoks.
{% endhint %}

### 1. etapp: tuvastamine (sihtmärgi tuvastamine)

**Mis toimub:**

* Chloros skaneerib pilte, mille juures on märgitud valikukast „Sihtmärk“ (kõik pildid, kui ükski pole märgitud)
* Arvutinägemise algoritmid tuvastavad kalibreerimispaneelid
* Iga paneeli peegeldusväärtused eraldatakse
* Sihtmärkide ajamärgid salvestatakse õige kalibreerimise ajastamiseks

**Kestus:**

* Märgitud sihtmärkidega: 10–60 sekundit
* Märgimata sihtmärkidega: 5–30+ minutit (skaneeritakse kõiki pilte)

**Edusammude näitaja:**

* Tuvastamine: 0% → 100%
* Skaneeritud piltide arv (loendab ainult tegelikult skaneeritud pilte)
* Leitud sihtmärkide arv

**Mida tuleks jälgida:**

* Kui sihtmärgid on korralikult märgistatud, peaks protsess kiiresti lõppema
* Kui protsess võtab liiga kaua aega, ei pruugi sihtmärgid olla märgistatud
* Kontrollige vealokist sõnumeid „Target found“

### 2. etapp: Analüüsimine

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

**Mis toimub:*** **Debayering**: RAW-Bayer-mustri teisendamine 3 kanaliks (LATTICE mono-moodulite puhul jäetakse vahele, koos märkusega)
* **Vignette'i korrigeerimine**: objektiivi servade tumenemise eemaldamine
* **Peegeldusvõime kalibreerimine**: normaliseerimine sihtväärtuste ja/või DAQ-i allapoole suunatud signaaliga
* **Indeksi arvutamine**: multispektraalsete indeksite arvutamine
* Iga pildi töötlemine kogu töötlusprotsessi vältel

**Kestus:** Suurem osa kogu töötlemisajast (60–80%)**Edusammude näitaja:**

* Kalibreerimine: 0% → 100%
* Hetkel töödeldav pilt
* Töödeldud pildid / Piltide koguarv

**Töötlemise režiimid:*** **Vaba režiim**: töötleb järjest ühte pilti korraga
* **Režiim „Chloros+“**: kasutab riistvarale kohanduvat töötajate rühma – 1–4 samaaegset töötajat GPU-süsteemidel (sõltuvalt VRAM-ist), üks töötaja füüsilise tuuma kohta (miinus üks) ainult CPU-ga süsteemidel. Vaata [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-kiirendus**: kiirendab seda etappi märkimisväärselt**Mida jälgida:**

* Piltide arvu järkjärguline vähenemine
* Kontrolli vealokist iga pildi kohta kuvatavaid lõpetamisteateid
* Hoiatused pildi kvaliteedi või kalibreerimisprobleemide kohta

### 4. etapp: Eksportimine

**Mis toimub:**

* Töödeldud pildid kirjutatakse valitud formaadis kettale kohe, kui need valmivad
* **LATTICE**: iga kaader jaguneb kõikidesse aktiveeritud toodetesse (debayered / eelvaade / radiance / reflectance)
* Multispektraalsete indeksipiltide eksportimine LUT-värvidega
* Väljundpuud `<project>/<camera>/<format>/<Product>_Images/` loomine — eksporditud failid säilitavad allika failinime; kaust määrab toote

**Kestus:** 10–20% kogu töötlemisajast**Edusammude näitaja:**

* Eksportimine: 0% → 100%
* Failide kirjutamine
* Ekspordivorming ja sihtkoht

**Mida jälgida:**

* Kõvakettaruumi hoiatused
* Failide kirjutamisvead
* Kõigi konfigureeritud väljundite valmimine

***

## Vahekaart „Debug Log“

Debug Log pakub üksikasjalikku teavet töötlemise edenemise ja tekkinud probleemide kohta. Ka tagapõhja käivitumissõnumid kuvatakse logikonsolis, seega annab logi täieliku ülevaate isegi siis, kui avate selle hiljem.

### Debug Logi avamine

1. Klõpsake vasakul külgribal ikooni **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">


2. Avaneb logipaneel, mis kuvab reaalajas töötlemissõnumeid
3. Logi keritakse automaatselt, et näidata viimaseid sõnumeid

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Logisõnumite mõistmine

Chloros i logiridadele on eesliitena lisatud sulgudes märksõnad, mis nimetavad alamsüsteemi — näiteks `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Kõige olulisem on teada **käivituse kokkuvõtet**, mis kuvatakse iga käivituse lõpus (sh peatatud käivitused):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Lisaks järgnevad `[RUN-SUMMARY]` vihjed alati, kui midagi vajab selgitust — näiteks käivitus, mis ei andnud tulemust, või kaamera, mille taotletud tulemus jäeti kõrvale, kuna see ei olnud kohaldatav. `[EXPORT-CHECK]`-read selgitavad kaamerapõhiseid vahelejätmisi (nt miks RGB-kaamera ei saanud kiirguse tulemust).

Üldised sõnumite tõsiduse tasemed (allpool toodud näited on illustratiivsed, mitte sõna-sõnalt):

#### Teabesõnumid (valge/hall)

Tavalised töötlemise uuendused: töötlemine alustatud, sihtmärgid tuvastatud (koos paneelide arvuga), pildi kaupa kalibreerimise edenemine, failid eksporditud, töötlemine lõpetatud.

#### Hoiatussõnumid (kollane)

Mittekriitilised probleemid, mis ei peata töötlemist – nt puuduvad GPS-andmed kaadris, suur ajamärgiste vahe sihtmärkide piltide vahel või madal kontrast kalibreerimispaneelis.

**Tegevus:** Vaadake hoiatused pärast töötlemist läbi, kuid ärge katkestage protsessi

#### Veateated (värv Red)

Kriitilised probleemid, mis võivad põhjustada töötlemise ebaõnnestumise – nt kõvaketas täis, rikutud pildifail või sihtmärkide tuvastamata jäämine, kui nõuti peegelduskalibreerimist.

**Tegevus:** Peata töötlemine, kõrvalda viga, alusta uuesti

### Tüüpilised logiolukorrad

| Olukord                             | Tähendus                                       | Vajalik tegevus                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Sihtmärk tuvastatud failis \[failinimi]        | Kalibreerimisihtmärk leitud edukalt         | Puudub – normaalne                                         |
| Pildi kaupa edusammude jooned              | Praeguse edusammude uuendus                       | Puudub – normaalne                                         |
| Sihtmärke ei leitud                      | Kalibreerimissihtmärke ei tuvastatud               | Märgi sihtmärgipildid või lülita peegelduskalibreerimine välja |
| Ebapiisav kettaruum               | Väljundandmete salvestamiseks pole piisavalt ruumi                 | Vabasta kettaruumi                                    |
| Rikutud faili vahelejätmine               | Pildifail on rikutud                         | Kopeerige fail uuesti SD-kaardilt                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Pildistust, millel puudub toorkaader, ei saa töödelda | Pildistage uuesti koos toorkaadriga, või kasutage CLI-d `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Töötlemine ei andnud pilditooteid — teatati veana koos vihjetega | Lugege vihjeridu; kontrollige, mis jäeti vahele ja miks |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Windows i pikad teed on välja lülitatud ja projektikausta asukoht on 260-tähemärgilise teepiirangu jaoks liiga sügav; midagi ei töödeldud | Kasutage lühemat projektikausta (teates on näidatud pikim sobiv tee) või lülita sisse Windows i pikad teed ja käivita Chloros uuesti — vaata [Töötlemise alustamine](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | LATTICE-toote klambermaski ei õnnestunud kirjutada; toode ise on korras ja näitab `Chloros:UnmixClampMask=unavailable` | Kontrollige kettaruumi ja kausta õigusi, seejärel töötlege uuesti, kui maski vajate |

### Logiandmete kopeerimine

Logi kopeerimiseks veaotsinguks või tugiabi saamiseks:

1. Avage paneel „Debug Log“
2. Klõpsake nuppu **Kopeeri**(kopeerib logi lõikelauale) või**Laadi alla**, et salvestada see tekstifailina
3. Kleepige tekstifaili või e-kirja
4. Saada vajadusel MAPIR i tugiteenistusele

### Vea teatamine logi vahekaardilt

Uus versioonis 1.3.0: veaikoon (**Teata veast**) „Debug Log“ vahekaardi ülaosas saadab aruande otse MAPIRile.

1. Klõpsa veaikoonil „Debug Log“ vahekaardi ülaosas
2. Kirjeldage probleemi: mida te tegite, mida ootasite ja mis tegelikult juhtus (kohustuslik)
3. Soovi korral märkige valik **Lisa mõned pildid (kuni 10)**ja valige pildid avatud projektist või kasutage nuppu**Sirvi...** muude pildifailide valimiseks
4. Klõpsake nuppu **Saada**

Aruanne sisaldab teie kirjeldust, kõiki lisatud pilte, Chloros i logifaile ja süsteemi andmeid (Chloros i versioon, operatsioonisüsteem, GPU ja projekti seaded). Aruande saatmiseks peate olema sisse logitud Chloros isse (vahekaart „User“). Saatmise ajal võite akna sulgeda; kui saatmine on lõppenud, kirjutatakse logisse viitenumber, näiteks `Bug report sent. Reference: BR-…` – märkige see ära, kui võtate ühendust MAPIR iga. Kui aruannet ei ole võimalik üles laadida (näiteks internetiühenduse puudumise tõttu), salvestab Chloros selle teie arvutisse ja näitab asukoha, et saaksite selle e-postiga saata aadressile support@mapir.camera.

***

## Süsteemiressursside seire

### CPU kasutamine

**Vaba režiim:**

* 1 protsessorituum töötab ~100% koormusega
* Ülejäänud tuumad on ooterežiimis või vabad
* Süsteem reageerib endiselt kiiresti

**„Chloros+“ paralleelrežiim:**

* Mitmed tuumad on suure koormusega – nende arv sõltub [dünaamilise arvutusvõimsuse kohandamise](../processing-architecture/dynamic-compute-adaptation.md) valitud strateegiast
* Süsteem võib tunduda vähem reageeriv

**Jälgimiseks:**

* Windows i ülesannete haldur (Ctrl+Shift+Esc)
* Vahekaart „Jõudlus“ → jaotis „Protsessor“
* Otsi protsesse „Chloros “ või „chloros-backend“

### Mälu (RAM) kasutamine

**Tüüpiline kasutamine:**

* Väikesed projektid (< 100 pilti): 2–4 GB
* Keskmise suurusega projektid (100–500 pilti): 4–8 GB
* Suured projektid (üle 500 pildi): 8–16 GB
* „Chloros+“ paralleelrežiim kasutab rohkem RAM-i

**Kui mälu on vähe:**

* Töötle väiksemaid partiisid
* Sulge teised rakendused
* Suurenda RAM-mälu mahtu, kui töötled regulaarselt suuri andmekogumeid

### GPU kasutamine (Chloros+ koos CUDA-ga)

Kui GPU kiirendus on sisse lülitatud:

* NVIDIA GPU kasutamine on kõrge (60–90%)
* VRAM-i kasutamine suureneb (nõuab vähemalt 4 GB VRAM-i; samaaegse tekstuuriteadliku debayeringu jaoks vähemalt 7 GB)
* Kalibreerimisetapp on märkimisväärselt kiirem

**Jälgimiseks:**

* NVIDIA süsteemiriba ikoon
* Ülesannete haldur → Jõudlus → GPU
* GPU-Z või sarnane seiretööriist

### Ketta sisend-väljund

**Mida oodata:**

* Analüüsietapis on ketta lugemiskoormus kõrge
* Ekspordietapis on ketta kirjutuskoormus kõrge
* SSD on märkimisväärselt kiirem kui HDD

**Jõudluse näpunäide:**

* Kasutage projektikausta jaoks võimaluse korral SSD-d
* Vältige võrgukettasid suurte andmekogumite puhul
* Veenduge, et kettaruum ei ole peaaegu täis (mõjutab kirjutuskiirust)

***

## Probleemide tuvastamine töötlemise ajal

### Hoiatavad märgid

**Töö seiskub (muutusi pole olnud üle 5 minuti):**

* Kontrollige veateateid silumislogist
* Kontrollige, kas kettal on piisavalt vaba ruumi
* Kontrollige ülesannete haldurist, kas Chloros töötab

**Veateated ilmuvad sageli:**

* Peatage töötlemine ja vaadake vead üle
* Tavapärased põhjused: kettaruum, rikutud failid, mäluprobleemid
* Vaadake allpool olevat jaotist „Vigade kõrvaldamine”

**Süsteem ei reageeri:**

* „Chloros+“ paralleelrežiim kasutab liiga palju ressursse
* Kaaluge samaaegsete ülesannete arvu vähendamist või riistvara uuendamist
* Vaba režiim on ressursisäästlikum

### Millal töötlemine peatada

Peatage töötlemine, kui näete:

* ❌ Vead „Kettaruum täis” või „Faili ei saa kirjutada”
* ❌ Korduvad pildifailide rikkumisvead
* ❌ Süsteem on täielikult hangunud (ei reageeri)
* ❌ Saite aru, et seadistused olid valed
* ❌ Imporditi valed pildid

**Kuidas peatada:**

1. Klõpsake**Peata-nuppu** (asendab Käivita-nuppu) – piisab ühest korrast
2. Ribal kuvatakse „Peatamine...“, kuni pooleliolev pilt lõpetatakse, seejärel lõpeb töötlemine peatatud olekus
3. Juba eksporditud tooted jäävad kettale alles; logisse kirjutatakse aus ülevaade `[RUN-SUMMARY]` selle kohta, mis lõpule viidi
4. Parandage probleemid ja käivitage uuesti — töötlemine algab algusest

***

## Tõrkeotsing töötlemise ajal

### Töötlemine on väga aeglane

**Võimalikud põhjused:**

* Märkimata sihtpildid (kõikide piltide skaneerimine)
* SSD-kõvaketta asemel HDD-kõvaketas
* Ebapiisavad süsteemiressursid
* Konfigureeritud on palju indekseid
* Võrguketta kasutamine

**Lahendused:**

1. Kui töötlemine on just alanud ja on tuvastamisetapis: peatage, märgistage sihtmärgid, alustage uuesti
2. Edaspidiseks: kasutage SSD-d, vähendage indeksite arvu, uuendage riistvara
3. Kaaluge CLI-i kasutamist suurte andmekogumite partiitöötlemiseks

### „Kettaruum” hoiatused

**Lahendused:**

1. Vabastage kohe kettaruumi
2. Viige projekt kettale, kus on rohkem ruumi
3. Vähendage eksporditavate indeksite arvu
4. Lülitage välja LATTICE eksporditooted, mida te ei vaja (Project Settings → Processing)
5. Kasutage JPG-vormingut TIFF i asemel (väiksemad failid)

### Sagedased „Corrupted File” teated

**Lahendused:**

1. Kopeerige pildid SD-kaardilt uuesti, et tagada nende terviklikkus
2. Kontrollige SD-kaarti vigade suhtes
3. Eemaldage rikutud failid projektist
4. Jätkake ülejäänud piltide töötlemist

### Süsteemi ülekuumenemine / jõudluse piiramine

**Lahendused:**

1. Tagage piisav ventilatsioon
2. Puhastage arvuti ventilatsiooniavad tolmust
3. Vähendage töötlemiskoormust (kasutage „Free” režiimi asemel „Chloros+” režiimi)
4. Töötlege pildid päeva jahedamatel aegadel

***

## Teade töötlemise lõpetamise kohta

Kui töötlemine lõpeb:

* Edusammude riba jõuab 100%ni
* Debug-logisse ilmuvad read `[RUN-SUMMARY]` koos lõplike arvudega
* Nupp „Start“ muutub uuesti aktiivseks
* Kõik väljundfailid asuvad projekti kaamerapõhises väljundkataloogis: `<project>/<camera>/<format>/<Product>_Images/`

***

## Järgmised sammud

Kui töötlemine on lõppenud:

1. **Vaadake tulemusi läbi** – vt [Töötlemise lõpetamine](finishing-the-processing.md)
2. **Kontrollige väljundkausta** – veenduge, et kõik failid on õigesti eksporditud
3. **Vaadake läbi vealogi** – kontrollige, kas seal on hoiatusi või vigu
4. **Vaadake töödeldud pilte eelvaates** – kasutage pildivaatajat või välist tarkvara

Töödeldud tulemuste läbivaatamise ja kasutamise kohta leiate teavet jaotisest [Töötlemise lõpetamine](finishing-the-processing.md).
