# Pildikihtid

Chloros pildivaataja rippmenüü „Pildikihtid” võimaldab teil kiiresti vahetada sama pildi erinevate versioonide vahel – alates originaalsetest piltidest kuni töödeldud peegeldusvõimsuse väljunditeni ja arvutatud indeksipiltideni.

## Mis on pildikihtid?

Chloros-is tähistavad **kihid** ühe algpildi jaoks kättesaadavaid erinevaid pildiväljundeid. Kui töötlete pilte, loob Chloros mitu versiooni:

* **Algpildid** (kaamera JPG- ja RAW-failid)
* **Peegelduskalibreeritud** väljundid (kui peegelduskalibreerimine oli lubatud)
* **Sihtmärgipildid** (kui pilt sisaldab kalibreerimisihtmärke)
* **Indeksipildid** (NDVI, NDRE, GNDVI jne, kui indeksid olid konfigureeritud)

Pildivaataja ülemises paremas nurgas asuv **kihtide valiku rippmenüü** võimaldab teil vahetada nende versioonide vahel ilma vaatajast lahkumata.***

## Saadaval olevad kihitüübid

### JPG

* Teie kaamera originaalne JPG-eelvaatepilt
* Alati saadaval kõigi piltide jaoks
* Töötlemata, nagu kaamera on pildistanud
* Laadub ja kuvatakse kõige kiiremini

**Millal vaadata:**

* Originaalse pildi kiire eelvaade
* Pildi kompositsiooni ja kadreeringu kontrollimine
* Pildi kvaliteedi kontrollimine enne töötlemist

### RAW (originaal)

* Teie kaamera originaalsed RAW-sensoriandmed
* Debayered, ilma järel töötlemiseta
* Suurem bittisügavus kui JPG-l (tavaliselt 12-bitised või 14-bitised sensoriandmed)

**Millal vaadata:**

* Originaalsete andurite andmete kvaliteedi kontrollimine
* Andurite probleemide või artefaktide kontrollimine
* Töötlemiseelsete ja -järgsete tulemuste võrdlemine

### RAW (sihtmärk)

* Ilmub ainult piltidel, milles on tuvastatud kalibreerimissihte
* Näitab originaalset RAW-pilti koos tuvastatud sihtmärgiga
* Kasutatakse sihtmärgi tuvastamise õnnestumise kontrollimiseks

**Millal vaadata:**

* Kalibreerimise sihtmärkide õige tuvastamise kinnitamiseks
* Sihtmärgi pildi kvaliteedi kontrollimiseks
* Kalibreerimisprobleemide lahendamiseks

{% hint style="info" %}
**Sihtmärgi kiht**: See kiht kuvatakse rippmenüüs ainult piltide puhul, mis sisaldavad kalibreerimise sihtmärke. Tavapärastel piltidel seda valikut ei ole.
{% endhint %}

### RAW (peegeldus)

* Kalibreeritud peegeldusväljundpilt
* Vignettkorrektsiooniga (kui töötlemisel on see aktiveeritud)
* Peegeldus kalibreeritud sihtmärgiandmete abil (kui see on aktiveeritud)
* Mitme lainepikkusega TIFF kõigi kaamera kanalitega
* Pikselväärtused esindavad peegelduse protsenti (kui kasutatakse protsendirežiimi)
* Valmis töötlemiseks [Index/LUT Sandbox](index-lut-sandbox.md)

**Millal vaadata:**

* Kalibreeritud tulemuste kontrollimine
* Kalibreerimise kvaliteedi kontrollimine
* Pikselväärtuste teadusliku täpsuse kontrollimine
* Võrdlemine originaaliga, et näha kalibreerimise mõju

{% hint style="success" %}
**Soovitatav**: Kasutage RAW (peegeldus) kihti, kui kontrollite pikselväärtuste teaduslikku täpsust mõõtmiste ja analüüside jaoks.
{% endhint %}

### RAW (NDVI indeks)... ja sarnased

* Arvutatud taimestikuindeksi pilt (selles näites NDVI)
* Indeksi nimi muutub sõltuvalt sellest, milline indeks töötlemise käigus konfigureeriti
* Näited: RAW (NDVI indeks), RAW (NDRE indeks), RAW (GNDVI indeks) jne.
* Ühe lainepikkusega halltooniline pilt, mis näitab indeksi arvutustulemusi
* Iga projekti seadetes konfigureeritud indeksi jaoks kuvatakse üks kiht

**Võimalikud indeksinimed:**

* RAW (NDVI indeks)
* RAW (NDRE indeks)
* RAW (GNDVI indeks)
* RAW (OSAVI indeks)
* RAW (EVI indeks)
* RAW (SAVI indeks)
* Ja palju muud... (vaata [Multispektraalsete indeksite valemid](../project-settings/multispectral-index-formulas.md))

**Millal vaadata:**

* Indeksi arvutustulemuste uurimine
* Indeksi väärtuste vahemike kontrollimine
* Huvi pakkuvate alade kindlaksmääramine
* Indeksipiltide kontrollimine enne kasutamist GIS-is või analüüsis

***

## Kihtide valija kasutamine

### Rippmenüü avamine

1. Avage pilt täisekraanirežiimis (klõpsake pildivaatajas mis tahes pisipildil)
2. Leidke vaataja paremas ülemises nurgas asuv **kihtide rippmenüü**

3. Rippmenüüs kuvatakse hetkel valitud kiht (nt „JPG”)
4. Klõpsake rippmenüüd, et näha kõiki saadaval olevaid kihte

### Kihtide vahetamine

1. Klõpsake kihi rippmenüüd, et avada nimekiri
2. Kuvatakse kõik hetke pildi jaoks saadaval olevad kihid
3. Klõpsake mis tahes kihi nimel, et vahetada selle versiooni juurde
4. Pilt uueneb kohe, et näidata valitud kihti

**Kiire vahetamine:**

* Rippmenüü mäletab teie viimast valikut
* Järgmisele pildile liikudes üritab Chloros näidata sama kihitüüpi
* Kui seda kihti järgmisel pildil ei ole, on vaikimisi JPG

### Kihtide kättesaadavus

Kõik kihid ei ole iga pildi jaoks kättesaadavad:

**Alati kättesaadavad:*** ✅ JPG (igal pildil on JPG-eelvaade)

**Tingimuslikult kättesaadavad:**

* ⚠️ RAW (originaal) – ainult juhul, kui pilt on pildistatud RAW- või RAW+JPG-režiimis
* ⚠️ RAW (sihtmärk) – ainult juhul, kui pilt sisaldab tuvastatud kalibreerimissihte
* ⚠️ RAW (peegeldus) – ainult pärast töötlemist, kui peegelduskalibreerimine on sisse lülitatud
* ⚠️ RAW (\[Index] Index) – ainult pärast töötlemist, kui indeksid on konfigureeritud

***

## Kihtide püsivus

### Piltide vahel navigeerimine

Kui navigeerite teise pildi juurde (nooleklahvide abil või pisipilte klõpsates):**Kihtide eelistused säilivad:**

* Kui vaatate „RAW (peegeldus)”, näitab järgmine pilt „RAW (peegeldus)” (kui see on saadaval)
* Kui vaadate &quot;RAW (NDVI indeks)&quot;, kuvatakse järgmisel pildil &quot;RAW (NDVI indeks)&quot; (kui see on saadaval)
* Kui sama kihti ei ole, on vaikimisi JPG

**Näide töövoost:**

1. Avage pilt 1, vahetage RAW-ile (NDVI Index)
2. Vajutage →, et vaadata pilti 2
3. Pilt 2 kuvab automaatselt RAW (NDVI Index) kihi
4. Jätkake navigeerimist – kõik pildid kuvavad NDVI kihi
5. Väga tõhus indeksitulemuste läbivaatamiseks paljude piltide puhul

***

## Tavalised töövood

### Töövoog 1: Enne/pärast võrdlus

**Eesmärk**: Võrrelge originaali ja kalibreeritud pilti

1. Avage töödeldud pilt pildivaatajas
2. Valige rippmenüüst **RAW (originaal)**

3. Märkige ära vinjetatsioon ja kalibreerimata väärtused
4. Vahetage rippmenüüst **RAW (peegeldus)**

5. Võrdle – vinjetatsioon on eemaldatud, väärtused kalibreeritud

### Töövoog 2: Indeksi läbivaatamine

**Eesmärk**: NDVI tulemuste kiire läbivaatamine kogu andmekogus

1. Ava esimene töödeldud pilt
2. Vali rippmenüüst **RAW (NDVI Index)**

3. Kasutage → nooleklahvi, et liikuda järgmise pildi juurde
4. NDVI kiht jääb automaatselt püsima
5. Jätkake kõigi piltide läbivaatamist, kontrollides NDVI mustreid
6. Valige võrdlemiseks **RAW (NDRE Index)**### Töövoog 3: Sihtmärkide kontrollimine**Eesmärk**: Kontrollida, et kõik sihtmärgipildid on õigesti tuvastatud

1. Liigu sihtmärgipildile
2. Vali rippmenüüst **RAW (Target)**

3. Kontrolli, et kalibreerimissihmärgid on selgelt nähtavad ja tuvastatud
4. Liigu järgmisele sihtmärgipildile
5. Korda kontrollimist kõigi sihtmärkide puhul

### Töövoog 4: Pikselväärtuste kontroll

**Eesmärk**: Kontrollida peegeldusväärtuste teaduslikku täpsust

1. Avage töödeldud pilt
2. Valige kiht **RAW (Reflectance)**

3. Lülitage sisse režiim**Pixel Percent** (nupp parempoolses ülemises tööriistaribal)
4. Liigutage kursorit taimestikualade kohale
5. Veenduge, et pikselväärtused jäävad oodatavatesse vahemikesse (30–70% NIR puhul, 5–15% Red puhul)
6. Kontrollige, et mulla- ja veealadel oleksid väärtused sobivad

***

## Pikselväärtuste mõistmine kihtide kaupa

Erinevad kihid näitavad erinevaid pikselväärtuste vahemikke:

### JPG-kiht

* **Vahemik**: 0–255 (8-bitine)
* **Tähendus**: kuvamisväärtused, gammakorrigeeritud
* **Kasutamine**: ainult visuaalne kontroll, mitte teaduslikuks mõõtmiseks

### RAW (originaal)

* **Vahemik**: 0–65535 (16-bitine)
* **Tähendus**: sensori töötlemata digitaalsed numbrid
* **Kasutamine**: sensori töökindluse kontrollimiseks, kalibreerimata

### RAW (peegeldus)

* **Vahemik**: 0–65 535 (16-bitine TIFF) või 0,0–1,0 (32-bitine protsent)
* **Tähendus**: Kalibreeritud peegeldusprotsent
* **Kasutamine**: Teaduslikud mõõtmised ja analüüsid**16-bitise TIFF puhul:**Jagage 65 535-ga, et saada peegeldusprotsent**32-bitise protsendi puhul:** Väärtused esindavad otseselt protsenti (0,5 = 50% peegeldus)

### RAW (indeksipildid)

* **Vahemik**: Sõltub indeksist (tavaliselt -1,0 kuni +1,0 normaliseeritud indeksite puhul)
* **Tähendus**: Indeksi arvutamise tulemus
* **Näited**:
  * NDVI: -1 kuni +1 (taimestik tavaliselt 0,4 kuni 0,9)
  * NDRE: -1 kuni +1 (stressi tuvastamine)
  * EVI: 0 kuni 1 (täiustatud taimestik)

***

## Näpunäited ja parimad tavad

### Tõhus kihtide vahetamine

* **Klaviatuurikombinatsioonide tundmine**: Kuigi kihtidele pole klaviatuurikombinatsioone, töötavad navigeerimisnooled (←/→) kõigi kihtide puhul
* **Järjepidevad töövood**: Valige üks kiht (nt NDVI) ja vaadake läbi kogu andmekogum, enne kui vahetate teisele
* **Kiired võrdlused**: Lülitage vaheldumisi Original ja Reflectance vahel, et kontrollida töötlemise kvaliteeti

### Jõudlusega seotud kaalutlused

* **JPG laadib kõige kiiremini**: Kasutage kiireks navigeerimiseks paljude piltide vahel
* **RAW-kihid laadivad aeglasemalt**: Kõrgem resolutsioon ja bittide sügavus
* **Indeksikihid**: Sarnane kiirus peegelduskihtidega
* **Esimene laadimine on aeglaseim**: Sama kihi järgmised vaatamised salvestatakse vahemällu ja on kiiremad

### Kvaliteedi kontroll

* **Kontrollige alati RAW-i (originaali)**: kontrollige allikaandmete kvaliteeti enne töödeldud väljundite usaldamist
* **Võrrelge kihte**: kasutage kihi vahetamist, et veenduda, et töötlemine toimis korrektselt
* **Kontrollige indeksivahemikke**: kasutage indeksikihtide puhul režiimi „Pixel Percent”, et veenduda, et väärtused on mõistlikud***

## Veaotsing

### Kiht pole saadaval

**Probleem**: oodatud kiht ei ilmu rippmenüüs**Võimalikud põhjused:**

* Pilti ei töödeldud (saadaval on ainult JPG ja RAW (originaal))
* Töötlemise ajal oli peegelduskalibreerimine keelatud
* Projektiseadetes ei olnud konkreetset indeksit konfigureeritud
* Pilt on ainult sihtmärgipilt (sihtmärkide jaoks indekseid ei genereerita)

**Lahendused:**

1. Veenduge, et pilt on töödeldud (kontrollige väljundkausta töödeldud failide olemasolu)
2. Kontrollige projektiseadeid, et veenduda, et indeksid on konfigureeritud
3. Töötle uuesti, lülitades soovitud indeksid sisse

### Näidatakse vale kiht

**Probleem**: Pilt avatakse ootamatus kihis**Põhjus**: Eelmise pildi kihi eelistus on üle kantud, kuid seda kihti pole praegusel pildil olemas**Lahendus**: Chloros lülitub automaatselt tagasi JPG-le, kui eelistatud kiht pole saadaval – see on normaalne käitumine

### Kalibreerimismärke ei ole näha

**Probleem**: RAW (märk) kiht ei näita märkide tuvastamist**Võimalikud põhjused:**

* Märke ei tuvastatud töötlemise käigus
* Pilt ei sisalda tegelikult märke
* Märkide tuvastamise seaded on liiga ranged

**Lahendused:**

1. Kontrollige vealokist sõnumeid „Märk leitud”
2. Veenduge, et pilt sisaldab tegelikult nähtavaid kalibreerimismärke
3. Reguleerige märkide tuvastamise seadeid projekti seadetest
4. Vaadake [Märkide piltide valimine](../processing-images-gui/choosing-target-images.md)

***

## Seotud funktsioonid

### Pildivaataja tööriistad

Mis tahes kihi vaatamisel saate kasutada:

* **Suumikontrollid**: Suurendage, et vaadata detaile
* **Panoraam**: Klõpsake ja lohistage, et liikuda suurendatud pildil
* **Piksli väärtuse vaatamine**: Vaadake väärtusi kursori asukohas
* **Navigatsiooninooled**: Liiguge piltide vahel, säilitades kihi
* **Piksli protsendi režiim**: Lülitage ümber DN- ja protsendinäidu vahel

Vaadake [Pildi avamine täisekraanil](opening-an-image-full-screen.md), et tutvuda pildivaataja täieliku dokumentatsiooniga.

### Indeks/LUT-liivakast

Interaktiivseks indeksite testimiseks ja visualiseerimiseks:

* **Reaalajas indeksite arvutamine**: Testige erinevaid indeksivalemeid
* **LUT-värvikaardistamine**: Rakendage värvigradiente halltoonides indeksitele
* **Visualisatsioonide eksportimine**: Salvesta värvilised indeksipildid

Vaadake [Indeks/LUT-liivakast](index-lut-sandbox.md) täpsema info saamiseks.

***

## Järgmised sammud

Nüüd, kui oled pildikihtidest aru saanud:

* [**Pildi avamine täisekraanil**](opening-an-image-full-screen.md) – Image Vieweri täielik juhend
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktiivne indeksite visualiseerimine
* [**Multispektraalsed indeksivalemid**](../project-settings/multispectral-index-formulas.md) – kättesaadavate indeksite viited
* [**Töötlemise lõpetamine**](../processing-images-gui/finishing-the-processing.md) – töödeldud väljundite mõistmine
