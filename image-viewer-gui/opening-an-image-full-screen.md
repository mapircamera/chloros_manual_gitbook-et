# Pildi avamine täisekraanil

Chloros pildivaataja pakub spetsiaalset täisekraani liidest multispektraalsete piltide vaatamiseks, analüüsimiseks ja töötlemiseks. Olgu tegemist originaalpiltide või töödeldud väljunditega, pakub pildivaataja võimsaid tööriistu kontrollimiseks ja analüüsimiseks.

## Pildivaataja avamine

### Failibrauserist

Kõige tavalisem viis pildi avamiseks pildivaatajas:

1. Veenduge, et olete **Failibrauseri** vahekaardil <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klõpsake mis tahes **pildi pisipilti** pildiristis
3. Pilt avatakse **peamises eelvaatealas** (ekraani keskel)
4. Pilt on nüüd laaditud ja valmis täisekraanil vaatamiseks

### Pildivaataja vahekaardi avamine

Kui pilt on eelvaatealasse laaditud:

1. Klõpsake **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> ikooni vasakul külgribal.
2. Pildivaataja vahekaart avatakse ja kuvab valitud pildi täisekraanil.
3. Vasakul külgribal muutuvad kättesaadavaks täiustatud vaatamis- ja analüüsitööriistad.

***

## Pildivaataja liidese ülevaade

### Peamine kuvamisala

Suurim osa ekraanist näitab teie pilti:

* **Täisresolutsioon**: pildid kuvatakse algresolutsioonis.
* **Suumitav**: kasutage suumimiseks juhtnuppe või hiirerattat
* **Pööratav**: suumimisel klõpsake ja lohistage, et pilti liigutada
* **Kuvasuhe säilitatakse**: pildid skaleeritakse proportsionaalselt

***

## Vaatevalikud

### Pildi põhiline navigeerimine

#### Piltide sirvimine

Navigeerige pildikogus klaviatuurilühendite või nuppude abil:

* **Järgmine pilt**: klõpsake nuppu → või vajutage klahvi **→** (parem nooleklahv)
* **Eelmine pilt**: klõpsake nuppu ← või vajutage klahvi **←** (vasak nooleklahv)
* **Hüppake konkreetsele pildile**: naaske failibrauserisse ja klõpsake soovitud pisipildil

#### Suumikontrollid

Reguleerige suurendust, et vaadata pildi detaile:

**Suurendamine:**

* Klõpsake nuppu **+** (pluss)
* Vajutage klahvi **+** või **=**
* Kerige hiirerattaga **üles**

**Vähendamine:**

* Klõpsake nuppu **−** (miinus)
* Vajutage klahvi **−** (miinus)
* Kerige hiirerattaga **alla**

**Ekraanile sobivaks:**

* Klõpsake nuppu **↔** (Sobivaks)
* Vajutage klahvi **0** (null)
* Klõpsake pildil topeltklõpsuga

#### Suumimisel pildi nihutamine

Kui suum on suurem kui ekraani suurus:

1. Liigutage hiirekursor pildi kohale
2. Klõpsake ja **hoidke hiire vasakut nuppu all**
3. **Lohistage**, et pilti liigutada
4. Vabastage, et pildil liigutamine lõpetada

**Alternatiiv**: kasutage nooleklahve, et pilti väikeste sammude kaupa liigutada

***

## Pikselväärtuste kontrollimine

### Pikselväärtuste vaatamine kursori juures

Kui liigutate hiirekursorit pildi kohale, kuvatakse pikselväärtused reaalajas:

**Väärtuste kuvamise asukoht:**

* **Ujuv number ja punane joon parempoolse indeksi LUT gradiendi legendis**
* **Suurendamisel ujuv väärtus kursori lähedal ja esiletõstetud piksel**
* Näitab väärtusi pikseli **kursori all või esiletõstetud**
* Uuendatakse hiire liigutamisel

***

## Vaadeldavad pilditüübid

### Originaalpildid (eeltöötlus)

**RAW + JPG pildid kaamerast:**

* Näitab RAW-andmeid eelvaatena
* Näitab originaalseid, korrigeerimata väärtusi
* Kasulik pildi kvaliteedi kontrollimiseks enne töötlemist

### Kalibreeritud peegeldusvõime pildid

**Pärast töötlemist:**

* Vignette korrigeeritud
* Peegeldus kalibreeritud
* Mitme sagedusega TIFF (Red, Green, NIR jne)
* Analüüsiks valmis teadusandmed

### Indeksipildid

**NDVI, NDRE, GNDVI jne (\_NDVI.tif failid):**

* Ühe sagedusriba halltoonilised pildid
* Pikselväärtused esindavad indeksite arvutamise tulemusi
* Normaliseeritud indeksite vahemik on tavaliselt -1 kuni +1
* Visualiseerimiseks saab rakendada värvi LUT-e

***

## Indeksi ja LUT-i rakendamine

Rakenda multispektraalsed indeksid ja värvi Look-Up tabelid:

1. Leia **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> külgribal
2. Valige taimestiku indeks (NDVI, NDRE jne)
3. Valige multispektraalne valem või looge oma kohandatud valem (ainult Chloros+)
4. Rakendage visualiseerimiseks värvi LUT gradiendi
5. Reguleerige väärtuste vahemikke ja künniseid

Täpsemad juhised leiate [Indeks/LUT Sandboxist](index-lut-sandbox.md).

***

## Klaviatuurilühendid

### Navigeerimine

* **→** (parem nooleklahv): järgmine pilt
* **←** (vasak nooleklahv): eelmine pilt
* **Home**: esimene pilt nimekirjas
* **End**: nimekirja viimane pilt

### Suum

* **+** või **=**: suumimine
* **−**: suumimine
* **0** (null): ekraanile sobivaks kohandamine
* **Hiirerattaga**: suumimine/suumimine

### Vaate juhtimise nuppud

* **P**: Pikselprotsendi režiimi lülitamine
* **L**: Kihtide paneeli lülitamine
* **Esc**: Täisekraani sulgemine või tagasipöördumine failibrauserisse

### Muu

* **Ctrl+S**: Praeguse pildi salvestamine
* **F**: Täisekraani režiim (kui saadaval)

***

### Indeksi arvutuste kontrollimine

Kontrollige, et indeksid on arvutatud õigesti:

1. Avage NDVI või muu indeksipilt.
2. Kontrollige taimestikualasid:
   * **NDVI**: Tervete taimede puhul peaks näitama 0,4–0,9.
   * **NDRE**: jõulise kasvu korral peaks näitama kõrgemaid väärtusi
   * **GNDVI**: sarnane NDVI-ga, kuid klorofüllitundlik
3. Kontrollige taimestikuväliseid alasid:
   * **Muld**: ligi 0 või veidi negatiivne
   * **Vesi**: negatiivsed väärtused (-0,5 kuni 0)

***

## Vaadeldavuse probleemide lahendamine

### Pilt ei avane

**Võimalikud põhjused:**

* Fail on töötlemise käigus rikutud
* Failivormingut ei toetata
* Suure pildi jaoks ei ole piisavalt mälu

**Lahendused:**

1. Proovige avada välises vaatajas, et kontrollida faili terviklikkust.
2. Kontrollige, kas failivorming vastab oodatavale tüübile.
3. Sulgege teised rakendused, et vabastada mälu.
4. Proovige väiksemat/teistsugust pilti.

### Must või valge pildi kuvamine

**Võimalikud põhjused:**

* Väärtuste vahemik väljaspool kuvamisvõimalusi.
* 32-bitine ujukomapilt ebatavaliste väärtustega.
* Indeksi arvutamise viga.

**Lahendused:**

1. Kontrollige pikseliväärtusi – kui need on kõik väga madalad või väga kõrged, reguleerige kuvamisvahemikku.
2. Proovige avada QGIS-is või sarnases rakenduses, mis reguleerib vahemikku automaatselt.
3. Kontrollige töötlemise veaotsingulogi vead.

### Pikseliväärtused tunduvad valed

**Võimalikud põhjused:**

* Vale pildi vaatamine (originaal vs töödeldud)
* Kalibreerimine ei rakendunud õigesti
* Valgusanduri andmed ei olnud sisestatud
* Protsendimoodus lülitati valesti sisse

**Lahendused:**

1. Veenduge, et vaatate töödeldud väljundit (kontrollige failinime laiendit)
2. Kontrollige protsendimooduse nupu seisundit
3. Võrrelge sama andmekogumi teadaolevalt heade piltidega

***

## Järgmised sammud

Nüüd, kui saate pilte täisekraanil vaadata:

* [**Pildikihtide**](image-layers.md) – Tutvuge mitme sagedusriba visualiseerimisega
* [**Indeksi/LUT-liivakast**](index-lut-sandbox.md) – Rakendage kohandatud indekseid ja värvide kaardistamist
* [**Multispektraalsed indeksivalemid**](../project-settings/multispectral-index-formulas.md) – tutvuge kättesaadavate indeksitega

Töötlemise töövoo kohta vaadake:

* [**Piltide töötlemine (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – täielik töötlemise juhend
