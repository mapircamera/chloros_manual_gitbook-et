# Pildikihtide valik

Chloros pildivaataja pildikihtide rippmenüü võimaldab teil kiiresti vahetada sama pildi erinevate versioonide vahel – alates originaalpiltidest kuni töödeldud peegeldusväljundite ja arvutatud indekspiltideni.

## Mis on pildikihtid?

Chloros-is tähendavad **kihid** ühe allika pildi jaoks saadaval olevaid erinevaid pildiväljundeid. Kui töötlete pilte, loob Chloros mitu versiooni:

* **Algupärased pildid** (kaamera JPG- ja RAW-failid)
* **Peegelduskalibreeritud** väljundid (kui peegelduskalibreerimine oli lubatud)
* **Sihtpildid** (kui pilt sisaldab kalibreerimise sihtmärke)
* **Indeksipildid** (NDVI, NDRE, GNDVI jne, kui indeksid olid konfigureeritud)

Pildi vaataja paremas ülemises nurgas asuv **kihtide valiku rippmenüü** võimaldab teil vaadata neid versioone vahetada, ilma vaatajat sulgemata.

***

## Saadaval olevad kihitüübid

### JPG

* Teie kaamera originaalne JPG-eelvaatepilt
* Alati saadaval kõikide piltide jaoks
* Töötlemata, nagu kaamera on pildistanud
* Kõige kiirem laadima ja kuvama

**Millal vaadata:**

* Originaalpildi kiire eelvaade
* Pildi kompositsiooni ja kadreeringu kontrollimine
* Pildi kvaliteedi kontrollimine enne töötlemist

### RAW (originaal)

* Teie kaamera originaalne RAW-andurite andmed
* Debayered, ilma järelkäsitluseta
* Suurem bittisügavus kui JPG-l (tavaliselt 12-bitine või 14-bitine andurite andmed)

**Millal vaadata:**

* Originaalsete andurite andmete kvaliteedi kontrollimine
* Andurite probleemide või artefaktide kontrollimine
* Töötlemise tulemuste võrdlemine enne ja pärast

### RAW (sihtmärk)

* Ilmub ainult piltidel, mis sisaldavad kalibreerimise sihtmärke
* Näitab originaalset RAW-pilti koos tuvastatud sihtmärgiga
* Kasutatakse sihtmärgi tuvastamise edukuse kontrollimiseks

**Millal vaadata:**

* Kalibreerimise sihtmärkide õige tuvastamise kinnitamine
* Sihtmärgi pildi kvaliteedi kontrollimine
* Kalibreerimisprobleemide lahendamine

{% hint style=&quot;info&quot; %}
**Sihtmärgi kiht**: See kiht kuvatakse rippmenüüs ainult piltide puhul, mis sisaldavad kalibreerimise sihtmärke. Tavapärastel piltidel seda valikut ei ole.
{% endhint %}

### RAW (peegeldus)

* Kalibreeritud peegelduse väljundpilt
* Vignette korrigeeritud (kui see on töötlemisel lubatud)
* Peegeldus kalibreeritud sihtmärgi andmete abil (kui see on aktiveeritud)
* Mitme sagedusega TIFF kõigi kaamera kanalitega
* Pikseliväärtused esindavad peegelduse protsenti (protsendimoodi kasutamisel)
* Valmis töötlemiseks [Index/LUT Sandboxiga](index-lut-sandbox.md)

**Millal vaadata:**

* Kalibreeritud tulemuste kontrollimine
* Kalibreerimise kvaliteedi kontrollimine
* Pikselväärtuste kontrollimine teadusliku täpsuse seisukohast
* Võrdlemine originaaliga, et näha kalibreerimise mõju

{% hint style=&quot;success&quot; %}
**Soovitus**: Kasutage RAW (peegeldus) kihti, kui kontrollite pikselväärtusi teaduslike mõõtmiste ja analüüside jaoks.
{% endhint %}

### RAW (NDVI indeks)... ja sarnased

* Arvutatud taimestiku indeks pilt (selles näites NDVI)
* Indeksi nimi muutub sõltuvalt sellest, milline indeks töötlemise ajal konfigureeriti
* Näited: RAW (NDVI indeks), RAW (NDRE indeks), RAW (GNDVI indeks) jne.
* Ühe sagedusega halltooniline pilt, mis näitab indeksi arvutamise tulemusi
* Iga projekti seadetest konfigureeritud indeksi jaoks kuvatakse üks kiht

**Võimalikud indeksi nimed:**

* RAW (NDVI indeks)
* RAW (NDRE indeks)
* RAW (GNDVI indeks)
* RAW (OSAVI indeks)
* RAW (EVI indeks)
* RAW (SAVI indeks)
* Ja palju muud... (vt [Multispektraalsed indeksivalemid](../project-settings/multispectral-index-formulas.md))

**Millal vaadata:**

* Indeksi arvutustulemuste uurimine
* Indeksi väärtuste vahemike kontrollimine
* Huvipakkuvate alade kindlaksmääramine
* Indeksi piltide kontrollimine enne kasutamist GIS-is või analüüsis

***

## Kihtide valija kasutamine

### Rippmenüü avamine

1. Avage pilt täisekraanil (klõpsake mis tahes pisipilti pildivaatajas)
2. Leidke vaataja paremas ülemises nurgas **kihtide rippmenüü**
3. Rippmenüüs on näha praegu valitud kiht (nt „JPG”)
4. Klõpsake rippmenüüd, et näha kõiki saadaval olevaid kihte

### Kihtide vahetamine

1. Klõpsake kihtide rippmenüüd, et avada nimekiri
2. Näidatakse kõik praeguse pildi jaoks saadaval olevad kihid
3. Klõpsake mis tahes kihi nime, et vahetada selle versioonile.
4. Pilt uueneb kohe, et näidata valitud kihti.

**Kiire vahetamine:**

* Rippmenüü mäletab teie viimast valikut.
* Järgmisele pildile liikudes üritab Chloros näidata sama kihitüüpi.
* Kui see kiht järgmisel pildil puudub, on vaikimisi valitud JPG.

### Kihtide kättesaadavus

Kõik kihid ei ole iga pildi jaoks kättesaadavad:

**Alati kättesaadavad:**

* ✅ JPG (iga pildil on JPG-eelvaade)

**Tingimuslikult kättesaadavad:**

* ⚠️ RAW (originaal) – ainult juhul, kui pilt on salvestatud RAW- või RAW+JPG-režiimis
* ⚠️ RAW (sihtmärk) – ainult juhul, kui pilt sisaldab tuvastatud kalibreerimise sihtmärke
* ⚠️ RAW (peegeldus) – ainult pärast töötlemist peegelduse kalibreerimise abil
* ⚠️ RAW (\[indeks] indeks) – ainult pärast töötlemist konfigureeritud indeksitega

***

## Kihtide püsivus

### Piltide vahel navigeerimine

Kui navigeerite teise pildile (nooleklahvide või pisipiltide klõpsamisega):

**Kihtide eelistus säilib:**

* Kui vaatate „RAW (peegeldus)”, näitab järgmine pilt „RAW (peegeldus)” (kui see on saadaval)
* Kui vaatate „RAW (NDVI indeks)”, kuvatakse järgmine pilt „RAW (NDVI indeks)” (kui see on saadaval)
* Kui sama kiht ei ole olemas, on vaikimisi JPG

**Näide töövoost:**

1. Avage pilt 1, vahetage RAW (NDVI Index)
2. Vajutage →, et vaadata pilti 2
3. Pilt 2 kuvab automaatselt kihi RAW (NDVI Index)
4. Jätkake navigeerimist – kõik pildid kuvavad NDVI kihti
5. Väga efektiivne paljude piltide indeksitulemuste läbivaatamiseks

***

## Tavapärased töövood

### Töövoog 1: Enne/pärast võrdlus

**Eesmärk**: Võrrelda originaali ja kalibreeritud pilti

1. Avage töödeldud pilt pildivaatajas
2. Valige rippmenüüst **RAW (originaal)**.
3. Märkige vinjetiseerumine ja kalibreerimata väärtused.
4. Vahetage rippmenüüst **RAW (peegeldus)**.
5. Võrdlege – vinjetiseerumine on eemaldatud, väärtused kalibreeritud.

### Töövoog 2: indeksite läbivaatamine

**Eesmärk**: NDVI tulemuste kiire läbivaatamine kogu andmekogus

1. Avage esimene töödeldud pilt.
2. Valige rippmenüüst **RAW (NDVI indeks)**.
3. Kasutage → nooleklahvi, et liikuda järgmise pildi juurde.
4. NDVI kiht jääb automaatselt alles.
5. Jätkake kõigi piltide läbivaatamist, kontrollides NDVI mustreid.
6. Vahetage **RAW (NDRE indeks)**, et võrrelda

### Töövoog 3: Sihtmärgi kontrollimine

**Eesmärk**: Kontrollida, et kõik sihtmärgi pildid on õigesti tuvastatud

1. Navigeerige sihtmärgi pildile
2. Valige rippmenüüst **RAW (sihtmärk)**
3. Kontrollige, et kalibreerimise sihtmärgid on selgelt nähtavad ja tuvastatud
4. Navigeerige järgmise sihtmärgi pildini
5. Korrake kontrollimist kõigi sihtmärkide puhul

### Töövoog 4: Pikselväärtuste kontrollimine

**Eesmärk**: Kontrollida peegeldusväärtuste teaduslikku täpsust

1. Avage töödeldud pilt
2. Valige **RAW (peegeldus)** kiht
3. Lülitage sisse **Pikselprotsent** režiim (nupp tööriistariba ülemises paremas nurgas)
4. Liigutage kursor taimestikualade kohale.
5. Veenduge, et pikseliväärtused on oodatud vahemikus (30–70% NIR puhul, 5–15% Red puhul).
6. Kontrollige, et mulla- ja veepindade väärtused on sobivad.

***

## Pikseliväärtuste mõistmine kihiti

Erinevad kihid näitavad erinevaid pikseliväärtuste vahemikke:

### JPG-kiht

* **Vahemik**: 0–255 (8-bitine)
* **Tähendus**: kuvatavad väärtused, gammakorrigeeritud
* **Kasutamine**: ainult visuaalne kontroll, mitte teaduslikuks mõõtmiseks

### RAW (originaal)

* **Vahemik**: 0–65535 (16-bitine)
* **Tähendus**: toores anduri digitaalne number
* **Kasutamine**: anduri töökindluse kontrollimine, kalibreerimata

### RAW (peegeldus)

* **Vahemik**: 0–65 535 (16-bitine TIFF) või 0,0–1,0 (32-bitine protsent)
* **Tähendus**: kalibreeritud protsentuaalne peegeldusvõime
* **Kasutamine**: teaduslikud mõõtmised ja analüüsid

**16-bitise TIFF puhul:** jagage 65 535-ga, et saada protsentuaalne peegeldusvõime **32-bitise protsendi puhul:** väärtused esindavad otseselt protsenti (0,5 = 50% peegeldusvõime)

### RAW (indekspildid)

* **Vahemik**: varieerub indeksi järgi (tavaliselt -1,0 kuni +1,0 normaliseeritud indeksite puhul)
* **Tähendus**: indeksi arvutamise tulemus
* **Näited**:
  * NDVI: -1 kuni +1 (taimestik tavaliselt 0,4 kuni 0,9)
  * NDRE: -1 kuni +1 (stressi tuvastamine)
  * EVI: 0 kuni 1 (täiustatud taimestik)

***

## Näpunäited ja parimad tavad

### Tõhus kihtide vahetamine

* **Klaviatuuri kiirklahvide tundmine**: kuigi kihtide jaoks ei ole klaviatuuri kiirklahve, töötavad navigeerimisnooled (←/→) kõikidel kihtidel
* **Järjepidevad töövood**: valige üks kiht (nt NDVI) ja vaadake kogu andmekogum läbi enne teise kihi valimist
* **Kiired võrdlused**: vahetage originaali ja peegelduse vahel, et kontrollida töötlemise kvaliteeti.

### Jõudlusega seotud kaalutlused

* **JPG laadib kõige kiiremini**: kasutage kiireks navigeerimiseks paljude piltide vahel.
* **RAW-kihid laadivad aeglasemalt**: kõrgem resolutsioon ja bittisügavus.
* **Indeksikiht**: sarnane kiirus peegelduskihtidega.
* **Esimene laadimine on aeglaseim**: sama kihi järgmised vaated salvestatakse vahemällu ja on kiiremad.

### Kvaliteedi kontroll

* **Kontrollige alati RAW (originaal)**: kontrollige allika andmete kvaliteeti enne töödeldud väljundite usaldamist
* **Võrdle kihte**: kasutage kihi vahetamist, et kontrollida töötlemise õigsust
* **Kontrollige indeksivahemikke**: kasutage indeksikihtidega pikseliprotsendi režiimi, et kontrollida väärtuste mõistlikkust

***

## Probleemide lahendamine

### Kiht pole saadaval

**Probleem**: Oodatud kiht ei ilmu rippmenüüs

**Võimalikud põhjused:**

* Pilti ei töödeldud (saadaval on ainult JPG ja RAW (originaal))
* Peegelduskalibreerimine oli töötlemise ajal keelatud
* Projektiseadetes ei olnud konfigureeritud konkreetne indeks
* Pilt on ainult sihtmärgi pilt (sihtmärkide jaoks ei ole indekseid loodud)

**Lahendused:**

1. Veenduge, et pilt on töödeldud (kontrollige töödeldud faile väljundkausta).
2. Kontrollige projekti seadeid, et veenduda, et indeksid on konfigureeritud.
3. Töötlege pilt uuesti, aktiveerides soovitud indeksid.

### Näidatakse vale kiht

**Probleem**: Pilt avatakse ootamatus kihis.

**Põhjus**: Eelmise pildi kihi eelistus on üle kantud, kuid see kiht ei ole praegusel pildil olemas.

**Lahendus**: Chloros lülitub automaatselt tagasi JPG-vormingule, kui eelistatud kiht ei ole saadaval – see on normaalne käitumine.

### Kalibreerimise sihtmärke ei ole näha

**Probleem**: RAW (sihtmärgi) kiht ei näita sihtmärgi tuvastamist.

**Võimalikud põhjused:**

* Sihtmärke ei tuvastatud töötlemise ajal.
* Pilt ei sisalda tegelikult sihtmärke.
* Sihtmärgi tuvastamise seaded on liiga ranged.

**Lahendused:**

1. Kontrollige veaotsingu logist, kas seal on sõnum „Sihtmärk leitud”.
2. Veenduge, et pilt sisaldab tegelikult nähtavaid kalibreerimise sihtmärke.
3. Reguleerige sihtmärgi tuvastamise seadeid projekti seadetest.
4. Vaadake [Sihtmärgi piltide valimine](../processing-images-gui/choosing-target-images.md).

***

## Seotud funktsioonid

### Pildivaataja tööriistad

Mis tahes kihi vaatamisel saate kasutada:

* **Suumikontrolle**: suurendage, et vaadata detaile.
* **Panoraam**: klõpsake ja lohistage, et liikuda suurendatud pildil
* **Pikselväärtuse kontrollimine**: vaadake väärtusi kursori asukohas
* **Navigatsiooninooled**: liikuge piltide vahel, säilitades kiht
* **Pikselprotsendi režiim**: lülitage DN- ja protsendimäära vahel

Vaadake [Pildi avamine täisekraanil](opening-an-image-full-screen.md), et saada täielikku teavet pildivaataja kohta.

### Indeks/LUT-liivakast

Interaktiivseks indeksite testimiseks ja visualiseerimiseks:

* **Reaalajas indeksite arvutamine**: testige erinevaid indeksite valemeid
* **LUT-värvide kaardistamine**: rakendage värvide gradiente halltoonides indeksitele
* **Visualiseeringute eksportimine**: salvestage värvilised indeksite pildid

Täpsema teabe saamiseks vaadake [Indeks/LUT-liivakast](index-lut-sandbox.md).

***

## Järgmised sammud

Nüüd, kui saate aru pildikihtidest:

* [**Pildi avamine täisekraanil**](opening-an-image-full-screen.md) – täielik Image Viewer juhend
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – interaktiivne indeksite visualiseerimine
* [**Multispektraalsed indeksivalemid**](../project-settings/multispectral-index-formulas.md) – kättesaadavad indeksid
* [**Töötlemise lõpetamine**](../processing-images-gui/finishing-the-processing.md) – töödeldud väljundite mõistmine
