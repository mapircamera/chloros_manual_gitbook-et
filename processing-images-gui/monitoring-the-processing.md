# Töötlemise jälgimine

Kui töötlemine on alanud, pakub Chloros mitmeid võimalusi protsessi jälgimiseks, probleemide kontrollimiseks ja andmekogumi olukorra mõistmiseks. Sellel lehel selgitatakse, kuidas jälgida töötlemist ja tõlgendada Chloros poolt pakutavat teavet.

## Ülevaade edenemisribast

Ülemises päises asuv edenemisriba näitab reaalajas töötlemise staatust ja valmimisprotsenti.

### Vaba režiimi edenemisriba

Kasutajatele, kellel pole Chloros+ litsentsi:

**2-etapiline edenemise kuvamine:**

1. **Sihtmärgi tuvastamine** – kalibreerimise sihtmärkide leidmine piltidest
2. **Töötlemine** – paranduste rakendamine ja eksportimine

**Progressiriba näitab:**

* Üldine valmidusprotsent (0–100%)
* Praeguse etapi nimi
* Lihtne horisontaalne riba

### Chloros+ progressiriba

Kasutajatele, kellel on Chloros+ litsents:

**4-etapiline progressinäit:**

1. **Avastamine** – kalibreerimise sihtmärkide leidmine
2. **Analüüsimine** – piltide uurimine ja töötlemisprotsessi ettevalmistamine
3. **Kalibreerimine** – vinjeti ja peegelduskorrektsioonide rakendamine
4. **Eksportimine** – töödeldud failide salvestamine

**Interaktiivsed funktsioonid:**

* **Viige hiir** edusammude riba kohale, et näha laiendatud 4-etapilist paneeli
* **Klõpsake** edenemisribale, et laiendatud paneel külmutada/kinnitada
* **Klõpsake uuesti**, et külmutamine tühistada ja hiirega eemaldamisel automaatselt peita
* Iga etapp näitab individuaalset edenemist (0–100%)

***

## Iga töötlemisetapi mõistmine

### Etapp 1: tuvastamine (sihtmärgi tuvastamine)

**Mis toimub:**

* Chloros skaneerib sihtmärgi valikukastiga märgitud pilte
* Arvutinägemise algoritmid tuvastavad 4 kalibreerimispaneeli
* Iga paneelist ekstraheeritakse peegeldusväärtused
* Sihtmärgi ajamärgid salvestatakse õige kalibreerimise ajastamise jaoks

**Kestus:**

* Märgitud sihtmärkidega: 10–60 sekundit
* Märgitud sihtmärkideta: 5–30+ minutit (skaneerib kõik pildid)

**Edusammude indikaator:**

* Tuvastamine: 0% → 100%
* Skaneeritud piltide arv
* Leitud sihtmärkide arv

**Mida jälgida:**

* Kui sihtmärgid on õigesti märgitud, peaks protsess kiiresti lõppema.
* Kui protsess võtab liiga kaua aega, võib olla, et sihtmärgid ei ole märgitud.
* Kontrollige veaotsingulogi, kas seal on sõnum „Sihtmärk leitud”.

### 2. etapp: analüüsimine

**Mis toimub:**

* Pildi EXIF-metadata lugemine (ajamärgid, säri seaded)
* Kalibreerimisstrateegia määramine sihtmärkide ajamärkide põhjal
* Pilditöötluse järjekorra korraldamine
* Paralleelse töötlemise töötajate ettevalmistamine (ainult Chloros+)

**Kestus:** 5–30 sekundit

**Edusammude indikaator:**

* Analüüsimine: 0% → 100%
* Kiire etapp, tavaliselt lõpeb kiiresti

**Mida jälgida:**

* Peab kulgema stabiilselt ilma pausideta
* Hoiatused puuduvate metaandmete kohta ilmuvad veaotsingulogis

### 3. etapp: kalibreerimine

**Mis toimub:**

* **Debayering**: RAW Bayer-mustri teisendamine 3 kanaliks
* **Vignette correction**: objektiivi servade tumenemise eemaldamine
* **Reflectance calibration**: normaliseerimine sihtväärtustega
* **Index calculation**: multispektraalsete indeksite arvutamine
* Iga pildi töötlemine kogu protsessi vältel

**Kestus:** enamik kogu töötlemisajast (60–80%)

**Edusammude indikaator:**

* Kalibreerimine: 0% → 100%
* Praegu töödeldav pilt
* Valmis pildid / Kõik pildid

**Töötlemise käitumine:**

* **Vaba režiim**: töötleb ühe pildi korraga järjest
* **Chloros+ režiim**: töötleb kuni 16 pilti korraga
* **GPU kiirendus**: kiirendab seda etappi oluliselt

**Mida jälgida:**

* Pidev edasiminek piltide arvu järgi
* Kontrollige veaparanduslogist iga pildi valmimise teateid
* Hoiatused pildi kvaliteedi või kalibreerimise probleemide kohta

### 4. etapp: eksportimine

**Mis toimub:**

* Kalibreeritud piltide kirjutamine valitud formaadis kettale
* Multispektraalsete indekspiltide eksportimine LUT-värvidega
* Kaameramudeli alamkataloogide loomine
* Originaalfailide nimede säilitamine sobivate laienditega

**Kestus:** 10–20% kogu töötlemisajast

**Edusammude indikaator:**

* Eksportimine: 0% → 100%
* Failide kirjutamine
* Ekspordi formaat ja sihtkoht

**Mida jälgida:**

* Kettaruumi hoiatused
* Failide kirjutamise vead
* Kõigi konfigureeritud väljundite valmimine

***

## Debug Log tab

Debug Log annab üksikasjalikku teavet töötlemise edenemise ja tekkinud probleemide kohta.

### Debug Logi avamine

1. Klõpsake **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> ikoonile vasakul külgribal
2. Avaneb logi paneel, mis näitab reaalajas töötlemise sõnumeid
3. Automaatne kerimine, et näidata viimaseid sõnumeid

### Logi sõnumite mõistmine

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

**Tegevus:** Vaadake hoiatused pärast töötlemist läbi, kuid ärge katkestage töötlemist.

#### Veateated (Red)

Kriitilised probleemid, mis võivad põhjustada töötlemise ebaõnnestumise:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Tegevus:** Peatage töötlemine, lahendage viga, alustage uuesti.

### Üldised logisõnumid

| Sõnum                          | Tähendus                                | Vajalik tegevus                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Sihtmärk avastatud failis \[failinimi]” | Kalibreerimise sihtmärk leitud  | Puudub – normaalne                                         |
| „Töödeldakse pilti X Y-st”        | Praegune edusammude uuendus                | Puudub – normaalne                                         |
| „Sihtmärke ei leitud”               | Kalibreerimise sihtmärke ei avastatud        | Märgi sihtmärgi pildid või keela peegelduskalibreerimine |
| „Kettaruumi ei ole piisavalt”        | Väljundiks ei ole piisavalt salvestusruumi          | Vabasta kettaruumi                                    |
| „Rikutud faili vahelejätmine”        | Pildifail on rikutud                  | Kopeeri fail uuesti SD-kaardilt                             |
| „PPK andmed rakendatud”               | .daq failist GPS-korrektsioonid rakendatud | Puudub – normaalne                                         |

### Logiandmete kopeerimine

Logi kopeerimine veaotsinguks või toe jaoks:

1. Avage veaotsingu logi paneel.
2. Klõpsake nuppu **„Kopeeri logi”** (või paremklõpsake → Valige kõik).
3. Kleepige tekstifaili või e-kirja.
4. Saatke vajadusel MAPIR-i toele.

***

## Süsteemi ressursside jälgimine

### CPU kasutamine

**Vaba režiim:**

* 1 CPU tuum ~100%
* Muud tuumad on jõudeolekus või saadaval
* Süsteem jääb reageerivaks

**Chloros+ paralleelrežiim:**

* Mitmed tuumad 80–100% (kuni 16 tuuma)
* Kõrge üldine CPU kasutamine
* Süsteem võib tunduda vähem reageeriv

**Jälgimiseks:**

* Windows Task Manager (Ctrl+Shift+Esc)
* Vahekaart Performance → CPU sektsioon
* Otsige protsesse „Chloros” või „chloros-backend”

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
* Kalibreerimisetapp on oluliselt kiirem

**Jälgimiseks:**

* NVIDIA süsteemikuvandi ikoon
* Ülesannete haldur → Jõudlus → GPU
* GPU-Z või sarnane jälgimistööriist

### Ketta I/O

**Mida oodata:**

* Kõrge ketta lugemine analüüsimise etapis
* Kõrge ketta kirjutamine eksportimise etapis
* SSD on oluliselt kiirem kui HDD

**Jõudluse näpunäide:**

* Kasutage projekti kausta jaoks võimaluse korral SSD-d
* Vältige võrgukettaid suurte andmekogumite puhul
* Veenduge, et ketta maht ei ole peaaegu täis (mõjutab kirjutuskiirust)

***

## Probleemide avastamine töötlemise ajal

### Hoiatavad märgid

**Töö seisab (ei muutu 5+ minutit):**

* Kontrollige veaotsingulogi vigu
* Kontrollige ketta vaba ruumi
* Kontrollige ülesannete haldurist, et Chloros töötab

**Veateated ilmuvad sageli:**

* Peatage töötlemine ja vaadake vead üle
* Tavapärased põhjused: ketta ruum, rikutud failid, mäluprobleemid
* Vaadake allpool olevat veaotsingu osa

**Süsteem ei vasta:**

* Chloros+ paralleelrežiim kasutab liiga palju ressursse
* Kaaluge samaaegsete ülesannete vähendamist või riistvara uuendamist
* Vaba režiim on ressursse vähem nõudev

### Millal töötlemine peatada

Peatage töötlemine, kui näete:

* ❌ Vead „Ketas täis” või „Ei saa faili kirjutada”
* ❌ Korduvad pildifailide rikkumisvead
* ❌ Süsteem on täielikult külmunud (ei reageeri)
* ❌ Avastati, et konfigureeriti valed seaded
* ❌ Imporditi valed pildid

**Kuidas peatada:**

1. Klõpsake **Stop/Cancel nuppu** (asendab Start nuppu)
2. Töötlemine peatub, edusammud kaovad
3. Parandage probleemid ja alustage uuesti algusest

***

## Töötlemise ajal tekkinud probleemide lahendamine

### Töötlemine on väga aeglane

**Võimalikud põhjused:**

* Märkimata sihtpildid (kõikide piltide skaneerimine)
* HDD asemel SSD-mälu
* Ebapiisavad süsteemiressursid
* Palju konfigureeritud indekseid
* Võrguketta juurdepääs

**Lahendused:**

1. Kui töötlemine on just alanud ja asub tuvastamisjärgus: tühista, märgi sihtpildid, alusta uuesti
2. Edaspidiseks: kasuta SSD-d, vähenda indeksite arvu, uuenda riistvara
3. Kaaluge CLI kasutamist suurte andmekogumite kogumitöötlemiseks

### Hoiatused „Kettaruum”

**Lahendused:**

1. Vabasta kohe kettaruumi
2. Viige projekt kettale, kus on rohkem ruumi
3. Vähendage eksporditavate indeksite arvu.
4. Kasutage TIFF asemel JPG-vormingut (väiksemad failid).

### Sagedased „Corrupted File” (Rikutud fail) teated

**Lahendused:**

1. Kopeerige pildid SD-kaardilt uuesti, et tagada terviklikkus.
2. Kontrollige SD-kaardi vead.
3. Eemaldage rikutud failid projektist.
4. Jätkake ülejäänud piltide töötlemist.

### Süsteemi ülekuumenemine / aeglustumine

**Lahendused:**

1. Tagage piisav ventilatsioon.
2. Puhastage arvuti ventilatsiooniavad tolmust.
3. Vähendage töötlemiskoormust (kasutage vaba režiimi Chloros+ asemel).
4. Töötlege jahedamal ajal päevast.

***

## Töötlemise lõpetamise teade

Kui töötlemine on lõppenud:

* Jõudlusribal jõuab 100%
* **Debug Logis** ilmub sõnum **„Töötlemine lõpetatud”**
* Start-nupp muutub taas aktiveerituks
* Kõik väljundfailid asuvad kaameramudeli alamkataloogis

***

## Järgmised sammud

Kui töötlemine on lõppenud:

1. **Vaadake tulemusi** – vt [Töötlemise lõpetamine](finishing-the-processing.md)
2. **Kontrollige väljundkausta** – veenduge, et kõik failid on eksporditud õigesti
3. **Vaadake üle veaparanduslogi** – kontrollige, kas seal on hoiatusi või vigu
4. **Vaadake eelvaadet töödeldud piltidest** – kasutage pildivaatajat või välist tarkvara

Töödeldud tulemuste ülevaatamise ja kasutamise kohta leiate teavet [Töötlemise lõpetamine](finishing-the-processing.md).
