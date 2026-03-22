# Töötlemise jälgimine

Kui töötlemine on alanud, pakub Chloros mitmeid võimalusi protsessi kulgemise jälgimiseks, probleemide tuvastamiseks ja andmekogumi seisundi mõistmiseks. Sellel lehel selgitatakse, kuidas töötlemist jälgida ja kuidas tõlgendada Chloros poolt pakutavat teavet.

## Edusammude riba ülevaade

Ülemises päises asuv edusammude riba näitab töötlemise staatust ja valmimisprotsenti reaalajas.

### Vaba režiimi edusammude riba

Kasutajatele, kellel puudub Chloros+ litsents:

**2-etapiline edusammude kuvamine:**

1.**Sihtmärgi tuvastamine** – kalibreerimissihte leidmine piltidelt
2. **Töötlemine** – paranduste rakendamine ja eksportimine**Edusammude riba näitab:**

* Üldine valmimisprotsent (0–100%)
* Praeguse etapi nimi
* Lihtne horisontaalne riba visualiseerimine

### Chloros+ edusammude riba

Kasutajatele, kellel on Chloros+ litsents:

**4-etapiline edusammude kuvamine:**

1.**Tuvastamine** – kalibreerimise sihtmärkide leidmine
2. **Analüüsimine** – piltide uurimine ja töötlemisprotsessi ettevalmistamine
3. **Kalibreerimine** – vinjeti ja peegeldusparanduste rakendamine
4. **Eksportimine** – töödeldud failide salvestamine**Interaktiivsed funktsioonid:*** **Viige hiir** edastusribale, et näha laienenud 4-etapilist paneeli
* **Klõpsake** edastusribale, et laienenud paneel fikseerida
* **Klõpsake uuesti**, et fikseeringut tühistada ja paneel hiire eemaldamisel automaatselt peita
* Iga etapp näitab individuaalset edastust (0–100%)

***

## Iga töötlemisetapi mõistmine

{% hint style="info" %}
**Töötlemisprotsessi arhitektuur**: Need 4 GUI etappi vastavad [4-niitilisele töötlemisprotsessile](../processing-architecture/processing-pipeline.md). GPU kiirendusega süsteemides kasutab 3. niit (kalibreerimine) [dünaamilist arvutusadapteerimist](../processing-architecture/dynamic-compute-adaptation.md), mis optimeerib töötlemist teie konkreetse riistvara jaoks.
{% endhint %}

### Etapp 1: Avastamine (sihtmärgi avastamine)

**Mis toimub:**

* Chloros skaneerib pilte, mis on märgitud sihtmärgi valikukastiga
* Arvutinägemise algoritmid tuvastavad 4 kalibreerimispaneeli
* Iga paneeli peegeldusväärtused ekstraheeritakse
* Sihtmärgi ajamärgid salvestatakse õige kalibreerimise ajastamiseks

**Kestus:**

* Märgitud sihtmärkidega: 10–60 sekundit
* Märgimata sihtmärkidega: 5–30+ minutit (skaneerib kõik pildid)

**Edusammude indikaator:**

* Avastamine: 0% → 100%
* Skaneeritud piltide arv
* Leitud sihtmärkide arv

**Mida jälgida:**

* Kui sihtmärgid on korralikult märgitud, peaks protsess kiiresti lõppema
* Kui protsess võtab liiga kaua aega, ei pruugi sihtmärgid olla märgitud
* Kontrollige vealogi, kas seal on sõnumeid „Sihtmärk leitud”

### 2. etapp: analüüsimine

**Mis toimub:**

* Pildi EXIF-metadata lugemine (ajamärgid, säri seaded)
* Kalibreerimisstrateegia kindlaksmääramine sihtmärkide ajamärkide põhjal
* Pilditöötlusjärjekorra korraldamine
* Paralleelse töötlemise töötajate ettevalmistamine (ainult Chloros+)

**Kestus:** 5–30 sekundit**Edusammude näitaja:**

* Analüüsimine: 0% → 100%
* Kiire etapp, lõpeb tavaliselt kiiresti

**Mida jälgida:**

* Protsess peaks kulgema sujuvalt ja pausideta
* Hoiatused puuduvate metaandmete kohta ilmuvad vealokisse

### 3. etapp: Kalibreerimine

**Mis toimub:*** **Debayering**: RAW-Bayer-mustri teisendamine 3 kanaliks
* **Vignette correction**: objektiivi servade tumenemise eemaldamine
* **Reflectance calibration**: normaliseerimine sihtväärtustega
* **Index calculation**: multispektraalsete indeksite arvutamine
* Iga pildi töötlemine kogu protsessi vältel

**Kestus:** Suurem osa kogu töötlemisajast (60–80%)**Edusammude indikaator:**

* Kalibreerimine: 0% → 100%
* Hetkel töödeldav pilt
* Töödeldud pildid / Piltide koguarv

**Töötlemise käitumine:*** **Vaba režiim**: Töötleb ühte pilti korraga järjest
* **Chloros+ režiim**: Töötleb kuni 16 pilti korraga
* **GPU kiirendus**: kiirendab seda etappi märkimisväärselt**Mida jälgida:**

* Piltide arvu järjepidev suurenemine
* Kontrollige vealokist iga pildi valmimise teateid
* Hoiatused pildi kvaliteedi või kalibreerimisprobleemide kohta

### 4. etapp: eksportimine

**Mis toimub:**

* Kalibreeritud piltide kirjutamine kettale valitud formaadis
* Multispektraalsete indeksipiltide eksportimine LUT-värvidega
* Kaameramudelite alamkaustade loomine
* Originaalfailinimede säilitamine sobivate laienditega

**Kestus:** 10–20% kogu töötlemisajast**Edusammude indikaator:**

* Eksportimine: 0% → 100%
* Kirjutatavad failid
* Ekspordivorming ja sihtkoht

**Mida jälgida:**

* Kõvakettaruumi hoiatused
* Failide kirjutamisvead
* Kõigi konfigureeritud väljundite valmimine

***

## Debug Log (veakontrolli logi) vahekaart

Debug Log (veakontrolli logi) pakub üksikasjalikku teavet töötlemise edenemise ja tekkinud probleemide kohta.

### Debug Logi avamine

1. Klõpsake vasakul külgribal **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> ikoonile vasakul külgribal
2. Avaneb logipaneel, mis kuvab reaalajas töötlemissõnumeid
3. Logi keritakse automaatselt, et kuvada viimaseid sõnumeid

### Logisõnumite mõistmine

#### Teabesõnumid (valge/hall)

Tavalised töötlemise uuendused:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Hoiatussõnumid (kollane)

Mittekriitilised probleemid, mis ei peata töötlemist:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Tegevus:** Vaadake hoiatusi pärast töötlemist läbi, kuid ärge katkestage töötlemist

#### Veateated (Red)

Kriitilised probleemid, mis võivad põhjustada töötlemise ebaõnnestumise:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Tegevus:** Peatage töötlemine, kõrvaldage viga, alustage uuesti

### Tavapärased logisõnumid

| Sõnum                          | Tähendus                                | Vajalik tegevus                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Sihtmärk tuvastatud failis \[failinimi]&quot; | Kalibreerimisihtmärk leitud  | Puudub – normaalne                                         |
| &quot;Töödeldakse pilti X / Y&quot;        | Praegune edusammude ülevaade                | Puudub – normaalne                                         |
| &quot;Sihtmärke ei leitud&quot;               | Kalibreerimisihtmärke ei tuvastatud        | Märgi sihtmärgipildid või lülita peegelduskalibreerimine välja |
| &quot;Kõvakettal pole piisavalt ruumi&quot;        | Väljundandmete salvestamiseks pole piisavalt ruumi          | Vabasta kõvakettal ruumi                                    |
| &quot;Vahelejäänud rikutud fail&quot;        | Pildifail on rikutud                  | Kopeeri fail uuesti SD-kaardilt                             |
| &quot;PPK-andmed rakendatud&quot;               | .daq-failist pärit GPS-korrektsioonid rakendatud | Puudub – normaalne                                         |

### Logiandmete kopeerimine

Logi kopeerimiseks veaotsinguks või tugi jaoks:

1. Avage paneel „Debug Log“
2. Klõpsake nuppu **„Copy Log“** (või paremklõpsake → Valige kõik)
3. Kleepige tekstifaili või e-kirja
4. Saatke vajadusel MAPIR tugiteenistusele

***

## Süsteemiressursside jälgimine

### CPU kasutamine

**Vaba režiim:**

* 1 CPU tuum ~100%
* Ülejäänud tuumad on ooterežiimis või vabad
* Süsteem reageerib endiselt

**Chloros+ Paralleelrežiim:**

* Mitmed tuumad 80–100% (kuni 16 tuuma)
* Kõrge üldine CPU kasutamine
* Süsteem võib tunduda vähem reageeriv

**Jälgimiseks:**

* Windows Task Manager (Ctrl+Shift+Esc)
* Performance tab → CPU section
* Otsi protsesse &quot;Chloros&quot; või &quot;chloros-backend&quot;

### Mälu (RAM) kasutamine

**Tüüpiline kasutamine:**

* Väikesed projektid (&lt; 100 pilti): 2–4 GB
* Keskmise suurusega projektid (100–500 pilti): 4–8 GB
* Suured projektid (üle 500 pildi): 8–16 GB
* Chloros+ paralleelrežiim kasutab rohkem RAM-i

**Kui mälu on vähe:**

* Töötle väiksemaid partiisid
* Sulge teised rakendused
* Uuenda RAM-i, kui töötled regulaarselt suuri andmekogusid

### GPU kasutamine (Chloros+ koos CUDA-ga)

Kui GPU kiirendus on sisse lülitatud:

* NVIDIA GPU näitab kõrget kasutust (60–90%)
* VRAM-i kasutamine suureneb (nõuab 4 GB+ VRAM-i)
* Kalibreerimisetapp on märkimisväärselt kiirem

**Jälgimiseks:**

* NVIDIA süsteemiriba ikoon
* Task Manager → Performance → GPU
* GPU-Z või sarnane jälgimistööriist

### Ketta I/O

**Mida oodata:**

* Kõrge kettalugemine analüüsietapis
* Kõrge kettakirjutamine ekspordietapis
* SSD on oluliselt kiirem kui HDD

**Jõudluse näpunäide:**

* Kasutage projektikausta jaoks võimaluse korral SSD-d
* Vältige võrgukettaid suurte andmekogumite puhul
* Veenduge, et kettal ei oleks palju vaba ruumi (mõjutab kirjutuskiirust)

***

## Probleemide tuvastamine töötlemise ajal

### Hoiatavad märgid

**Töö seisab (muutusi pole üle 5 minuti):**

* Kontrollige vead silumislogist
* Kontrollige kettaruumi olemasolu
* Kontrollige ülesannete haldurist, et Chloros töötab

**Veateated ilmuvad sageli:**

* Peatage töötlemine ja vaadake vead üle
* Tavapärased põhjused: kettaruum, rikutud failid, mäluprobleemid
* Vaadake allpool olevat jaotist „Vigade kõrvaldamine”

**Süsteem ei reageeri:**

* Chloros+ paralleelrežiim kasutab liiga palju ressursse
* Kaaluge samaaegsete ülesannete arvu vähendamist või riistvara uuendamist
* Vaba režiim on ressursisäästlikum

### Millal töötlemine peatada

Peatage töötlemine, kui näete:

* ❌ Vead „Kett on täis” või „Faili ei saa kirjutada”
* ❌ Korduvad pildifailide rikkumisvead
* ❌ Süsteem on täielikult hangunud (ei reageeri)
* ❌ Saite aru, et seadistused olid valed
* ❌ Imporditi valed pildid

**Kuidas peatada:**

1. Klõpsake**Peata/Tühista nuppu** (asendab Start nuppu)
2. Töötlemine peatub, edusammud kaovad
3. Parandage probleemid ja alustage uuesti algusest

***

## Tõrkeotsing töötlemise ajal

### Töötlemine on väga aeglane

**Võimalikud põhjused:**

* Märkimata sihtpildid (kõikide piltide skaneerimine)
* HDD asemel SSD-salvestusruum
* Ebapiisavad süsteemiressursid
* Konfigureeritud on palju indekseid
* Võrguketta juurdepääs

**Lahendused:**

1. Kui töötlemine on just alanud ja on tuvastamisetapis: tühista, märgi sihtmärgid, alusta uuesti
2. Edaspidiseks: kasuta SSD-d, vähenda indekseid, uuenda riistvara
3. Kaalu CLI kasutamist suurte andmekogumite partiitöötlemiseks

### Hoiatused „Kettaruum” kohta

**Lahendused:**

1. Vabasta kohe kettaruumi
2. Viia projekt kettale, kus on rohkem ruumi
3. Vähenda eksporditavate indeksite arvu
4. Kasuta JPG-vormingut TIFF asemel (väiksemad failid)

### Sagedased „Rikutud faili” teated

**Lahendused:**

1. Kopeeri pildid SD-kaardilt uuesti, et tagada terviklikkus
2. Kontrolli SD-kaarti vigade suhtes
3. Eemalda rikutud failid projektist
4. Jätka ülejäänud piltide töötlemist

### Süsteemi ülekuumenemine / aeglustumine

**Lahendused:**

1. Tagage piisav ventilatsioon
2. Puhastage arvuti ventilatsiooniavad tolmust
3. Vähendage töötlemiskoormust (kasutage Free režiimi asemel Chloros+)
4. Töötlege päeva jahedamatel aegadel

***

## Töötlemise lõpetamise teade

Kui töötlemine lõpeb:

* edenemisribal jõuab 100%ni
* **&quot;Töötlemine lõpetatud&quot;** sõnum ilmub veaparanduslogisse
* Käivitusnupp muutub taas aktiivseks
* Kõik väljundfailid asuvad kaameramudeli alamkaustas

***

## Järgmised sammud

Kui töötlemine on lõppenud:

1. **Vaadake tulemusi** – vt [Töötlemise lõpetamine](finishing-the-processing.md)
2. **Kontrollige väljundkausta** – veenduge, et kõik failid on õigesti eksporditud
3. **Vaadake läbi vealogi** – kontrollige, kas seal on mingeid hoiatusi või vigu
4. **Vaadake töödeldud pilte eelvaates** – kasutage pildivaatajat või välist tarkvara

Töödeldud tulemuste läbivaatamise ja kasutamise kohta leiate teavet jaotisest [Töötlemise lõpetamine](finishing-the-processing.md).
