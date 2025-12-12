# Pildikihtid

Chloros pildivaataja pildikihtide rippmenüü võimaldab teil kiiresti vahetada sama pildi erinevate versioonide vahel – alates originaalpiltidest kuni töödeldud peegeldusväljundite ja arvutatud indekspiltideni.

## Mis on pildikihtid?

Chloros-is viitavad **kihid** ühe allika pildi jaoks kättesaadavatele erinevatele pildiväljunditele. Kui töötlete pilte, loob Chloros mitu versiooni:

* **Algupärased pildid** (JPG- ja RAW-failid teie kaamerast)
* **Peegelduskalibreeritud** väljundid (kui peegelduskalibreerimine oli lubatud)
* **Sihtpildid** (kui pilt sisaldab kalibreerimise sihtmärke)
* **Indeksipildid** (NDVI, NDRE, GNDVI jne, kui indeksid olid konfigureeritud)

Pildi vaataja paremas ülemises nurgas asuv **kihtide valiku rippmenüü** võimaldab teil vaadata neid versioone vahetult, ilma vaatajast lahkumata.

***

## Saadaval olevad kihitüübid

### JPG

* Teie kaamera originaalne JPG-eelvaatepilt
* Alati saadaval kõikide piltide jaoks
* Töötlemata, nagu kaamera on pildistanud
* Kõige kiirem laadima ja kuvama

**Millal vaadata:**

* Originaalkaadri kiire eelvaade
* Pildi kompositsiooni ja kadreeringu kontrollimine
* Kaadri kvaliteedi kontrollimine enne töötlemist

### RAW (originaal)

* Teie kaamera originaalsed RAW-andurid
* Debayering ilma järel töötlemiseta
* Suurem bittisügavus kui JPG-l (tavaliselt 12-bitine või 14-bitine andurite andmed)

**Millal vaadata:**

* Originaalse andurite andmete kvaliteedi kontrollimine
* Andurite probleemide või artefaktide kontrollimine
* Töötlemise eel- ja järel tulemuste võrdlemine

### RAW (sihtmärk)

* Ilmub ainult piltidel, mis on identifitseeritud kalibreerimise sihtmärkidega
* Näitab originaalset RAW-pilti koos tuvastatud sihtmärgiga
* Kasutatakse sihtmärgi tuvastamise edukuse kontrollimiseks

**Millal vaadata:**

* Kalibreerimise sihtmärkide õige tuvastamise kinnitamiseks
* Sihtmärgi pildi kvaliteedi kontrollimiseks
* Kalibreerimisprobleemide lahendamiseks

{% hint style=&quot;info&quot; %}
**Sihtmärgi kiht**: See kiht kuvatakse rippmenüüs ainult kalibreerimise sihtmärke sisaldavate piltide puhul. Tavapärastel piltidel seda valikut ei ole.
{% endhint %}

### RAW (peegeldus)

* Kalibreeritud peegeldusega väljundpilt
* Vignette parandatud (kui see on töötlemisel lubatud)
* Peegeldus kalibreeritud sihtmärgi andmete abil (kui see on aktiveeritud)
* Mitme sagedusega TIFF kõigi kaamera kanalitega
* Pikseliväärtused esindavad peegelduse protsenti (protsendimoodi kasutamisel)
* Valmis töötlemiseks [Index/LUT Sandboxiga](index-lut-sandbox.md)

**Millal vaadata:**

* Kalibreeritud tulemuste kontrollimine
* Kalibreerimise kvaliteedi kontrollimine
* Pikselväärtuste kontrollimine teadusliku täpsuse seisukohast
* Võrdlemine originaaliga, et näha kalibreerimise mõju

{% vihje style=&quot;success&quot; %}
**Soovitus**: Kasutage RAW (peegeldus) kihti, kui kontrollite pikselväärtusi teaduslike mõõtmiste ja analüüside jaoks.
{% endhint %}

### RAW (NDVI indeks)... ja sarnased

* Arvutatud taimestiku indeksipilt (NDVI selles näites)
* Indeksi nimi muutub sõltuvalt sellest, milline indeks töötlemise ajal konfigureeriti.
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

1. Avage pilt täisekraanil (klõpsake pildivaatajas mis tahes pisipildil)
2. Leidke vaataja paremas ülemises nurgas **kihtide rippmenüü**.
3. Rippmenüüs on näha praegu valitud kiht (nt „JPG”).
4. Klõpsake rippmenüüd, et näha kõiki saadaval olevaid kihte.

### Kihte vahetamine

1. Klõpsake kihtide rippmenüüd, et avada nimekiri.
2. Näidatakse kõik praeguse pildi jaoks saadaval olevad kihid.
3. Klõpsake mis tahes kihi nimel, et vahetada selle versioonile.
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
* ⚠️ RAW (\[Index] indeks) – ainult pärast töötlemist indeksitega

***

## Kihtide püsivus

### Piltide vahel navigeerimine

Kui navigeerite teise pildile (nooleklahvide või pisipiltide klõpsamisega):

**Kihtide eelistused säilivad:**

* Kui vaatate „RAW (peegeldus)”, näitab järgmine pilt „RAW (peegeldus)” (kui see on saadaval)
* Kui vaatate „RAW (NDVI indeks)”, näitab järgmine pilt „RAW (NDVI indeks)” (kui see on saadaval)
* Kui sama kihti ei ole, on vaikimisi JPG.

**Näide töövoost:**

1. Avage pilt 1, vahetage RAW (NDVI Index)
2. Vajutage →, et vaadata pilti 2.
3. Pilt 2 kuvab automaatselt RAW (NDVI Index) kihi.
4. Jätkake navigeerimist – kõik pildid kuvavad NDVI kihti.
5. Väga efektiivne paljude piltide indeksitulemuste vaatamiseks.

***

## Tavapärased töövood

### Töövoog 1: Enne/pärast võrdlus

**Eesmärk**: Võrrelda originaali ja kalibreeritud pilti.

1. Avage töödeldud pilt pildivaatajas.
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
4. Navigeerige järgmise sihtmärgi pildini.
5. Korrake kontrollimist kõigi sihtmärkide puhul.

### Töövoog 4: Pikselväärtuste kontrollimine

**Eesmärk**: Kontrollida peegeldusväärtuste teaduslikku täpsust.

1. Avage töödeldud pilt.
2. Valige kiht **RAW (peegeldus)**.
3. Lülitage sisse režiim **Pikselprotsent** (nupp tööriistariba ülemises paremas nurgas).
4. Liiguta kursor taimestiku alade kohale
5. Kontrolli, et pikseliväärtused on oodatud vahemikus (30–70% NIR jaoks, 5–15% Red jaoks)
6. Kontrolli, et mulla- ja veealade väärtused on sobivad

***

## Pikseliväärtuste mõistmine kihiti

Erinevad kihid näitavad erinevaid pikselväärtuste vahemikke:

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

### Efektiivne kihtide vahetamine

* **Klaviatuuri kiirklahvide tundmine**: kuigi kihtide jaoks ei ole klaviatuuri kiirklahve, töötavad navigeerimisnooled (←/→) kõikidel kihtidel
* **Järjepidevad töövood**: valige üks kiht (nt NDVI) ja vaadake kogu andmekogum läbi enne teisele kihile üleminekut
* **Kiired võrdlused**: vahetage originaali ja peegelduse vahel, et kontrollida töötlemise kvaliteeti

### Jõudlusega seotud kaalutlused

* **JPG laadib kõige kiiremini**: kasutage kiireks navigeerimiseks paljude piltide vahel.
* **RAW-kihid laadivad aeglasemalt**: kõrgem resolutsioon ja bittisügavus.
* **Indeksi kihid**: sarnane kiirus peegelduskihtidega.
* **Esimene laadimine on aeglaseim**: sama kihi järgmised vaated salvestatakse vahemällu ja on kiiremad.

### Kvaliteedi kontroll

* **Kontrollige alati RAW (originaal)**: kontrollige allika andmete kvaliteeti enne töödeldud väljundite usaldamist
* **Võrrelge kihte**: kasutage kihi vahetamist, et kontrollida töötlemise õigsust
* **Kontrollige indeksivahemikke**: kasutage indeksikihtidega pikseliprotsendi režiimi, et kontrollida väärtuste mõistlikkust

***

## Probleemide lahendamine

### Kiht pole saadaval

**Probleem**: Oodatud kiht ei ilmu rippmenüüs

**Võimalikud põhjused:**

* Pilti ei ole töödeldud (saadaval on ainult JPG ja RAW (originaal))
* Peegelduskalibreerimine oli töötlemise ajal keelatud
* Projekti seadetest ei olnud konfigureeritud konkreetne indeks
* Pilt on ainult sihtmärgi pilt (sihtmärkide jaoks ei ole indekseid loodud)

**Lahendused:**

1. Kontrollige, kas pilt on töödeldud (kontrollige töödeldud faile väljundkausta).
2. Kontrollige projekti seadeid, et veenduda, kas indeksid on konfigureeritud.
3. Töötlege uuesti, aktiveerides soovitud indeksid.

### Näidatakse vale kiht

**Probleem**: pilt avatakse ootamatus kihis.

**Põhjus**: eelmise pildi kihi eelistus on üle kantud, kuid see kiht ei ole praegusel pildil olemas.

**Lahendus**: Chloros lülitub automaatselt tagasi JPG-vormingule, kui eelistatud kiht pole kättesaadav – see on normaalne käitumine.

### Kalibreerimise sihtmärke pole näha

**Probleem**: RAW (sihtmärgi) kiht ei näita sihtmärgi tuvastamist.

**Võimalikud põhjused:**

* Sihtmärke ei tuvastatud töötlemise ajal.
* Pilt ei sisalda tegelikult sihtmärke.
* Sihtmärgi tuvastamise seaded on liiga ranged

**Lahendused:**

1. Kontrollige veaparanduslogist, kas seal on sõnum „Sihtmärk leitud”
2. Veenduge, et pilt sisaldab tegelikult nähtavaid kalibreerimise sihtmärke
3. Reguleerige sihtmärgi tuvastamise seadeid projekti seadetest
4. Vaadake [Sihtmärgi piltide valimine](../processing-images-gui/choosing-target-images.md)

***

## Seotud funktsioonid

### Pildivaataja tööriistad

Mis tahes kihi vaatamisel saate kasutada järgmisi vahendeid:

* **Suumikontrollid**: suurendage, et vaadata detaile
* **Panoraam**: klõpsake ja lohistage, et liikuda suumitud pildil
* **Pikselväärtuse kontroll**: vaadake väärtusi kursori asukohas
* **Navigatsiooninooled**: liikuge piltide vahel, säilitades kihi
* **Pikseliprotsendi režiim**: lülitage DN- ja protsendimäära kuvamise vahel

Vaadake [Pildi avamine täisekraanil](opening-an-image-full-screen.md), et saada täielikku teavet pildivaataja kohta.

### Indeksi/LUT-liivakast

Interaktiivseks indeksitestiks ja visualiseerimiseks:

* **Reaalajas indeksiarvutus**: testige erinevaid indeksivalemeid
* **LUT värvide kaardistamine**: värvide gradiendi rakendamine halltoonide indeksitele
* **Visualiseeringute eksportimine**: värviliste indeksite piltide salvestamine

Vaata [Indeks/LUT Sandbox](index-lut-sandbox.md) täpsema info saamiseks.

***

## Järgmised sammud

Nüüd, kui sa mõistad pildikihtide olemust:

* [**Pildi avamine täisekraanil**](opening-an-image-full-screen.md) – täielik pildivaataja juhend
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – interaktiivne indeksite visualiseerimine
* [**Multispektraalsed indeksivalemid**](../project-settings/multispectral-index-formulas.md) – kättesaadavad indeksid
* [**Töötlemise lõpetamine**](../processing-images-gui/finishing-the-processing.md) – töödeldud väljundite mõistmine
