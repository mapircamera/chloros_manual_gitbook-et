# Pildi avamine täisekraanil

Chloros pildivaataja pakub spetsiaalset täisekraaniliidest multispektraalsete piltide vaatamiseks, analüüsimiseks ja töötlemiseks. Olgu tegemist originaalpiltide või töödeldud väljunditega, pakub pildivaataja võimsaid vahendeid kontrollimiseks ja analüüsimiseks.

## Pildivaatajasse sisenemine

### Failibrauserist

Kõige tavalisem viis pildi avamiseks pildivaatajas:

1. Veenduge, et olete **Failibrauseri** vahekaardil <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klõpsake pildiribal mis tahes **pildi pisipildil**

3. Pilt avatakse**peamises eelvaatealas** (ekraani keskel)
4. Pilt on nüüd laaditud ja valmis täisekraanil vaatamiseks

### Pildivaataja vahekaardi avamine

Kui pilt on eelvaatealasse laaditud:

1. Klõpsake vasakul külgribal **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> ikoonile vasakul külgribal
2. Avaneb pildivaataja vahekaart, mis kuvab valitud pildi täisekraanil
3. Vasakul külgribal muutuvad kättesaadavaks täiustatud vaatamis- ja analüüsitööriistad

***

## Pildivaataja liidese ülevaade

### Peamine kuvamisala

Ekraani suurim osa näitab teie pilti:

* **Täisresolutsioon**: pildid kuvatakse algupärases resolutsioonis
* **Suurendatav**: kasutage suurendamiseks juhtnuppe või hiirerattat
* **Liigutatav**: suurendatud pildi puhul klõpsake ja lohistage, et liikuda
* **Kuvasuhe säilitatud**: pildid skaaluvad proportsionaalselt***

## Vaatevalikud

### Pildi põhiline navigeerimine

#### Piltide sirvimine

Navigeerige pildikogus klahvikombinatsioonide või nuppude abil:

* **Järgmine pilt**: klõpsake nuppu → või vajutage klahvi**→** (parem nool)
* **Eelmine pilt**: klõpsake nuppu ← või vajutage klahvi**←** (vasak nool)
* **Hüppake konkreetsele pildile**: naaske failibrauserisse ja klõpsake soovitud pisipilti

#### Suumimise juhtnupud

Reguleerige suurendust, et vaadata pildi detaile:

**Suurendamine:*** Klõpsake nuppu **+** (pluss)
* Vajutage klahvi **+**või**=*** Kerige hiirerattaga **üles**

**Vähendamine:*** Klõpsake nuppu **−** (miinus)
* Vajutage klahvi **−** (miinus)
* Kerige hiirerattaga **alla**

#### Suumimisel pildil liikumine

Kui suum on suurem kui ekraani suurus:

1. Viige hiirekursor pildi kohale
2. Klõpsake ja **hoidke vasakut hiirenuppu**

3.**Lohistage**, et pilti liigutada
4. Vabastage, et pildil liikumine peatada

**Alternatiiv**: Kasutage nooleklahve, et liikuda väikeste sammude kaupa***

## Pikselväärtuste vaatamine

### Pikselväärtuste vaatamine kursori juures

Kui liigutad hiirekursorit pildi kohal, kuvatakse pikselväärtused reaalajas:**Väärtuste kuvamise asukoht:*** **Ujuv number ja punane joon parempoolse indeks-LUT-gradiendi legendis*** **Kui suumitakse veelgi sisse, ujuv väärtus kursori ja esiletõstetud pikseli juures*** Näitab väärtusi **kursori all või esiletõstetud** pikseli jaoks
* Uuendub hiire liikumisel

***

## Vaadeldavad pilditüübid

### JPG

**Kaamerast pärit JPG-pildid:**

* JPG-andmete kuvamine eelvaatena
* Originaalsete, korrigeerimata väärtuste kuvamine
* Kasulik pildi kvaliteedi kontrollimiseks enne töötlemist

### RAW (originaal)

### RAW (peegeldus)

**Pärast töötlemist:**

* Vignett korrigeeritud
* Peegeldus kalibreeritud
* Mitme sagedusribaga TIFF (Red, Green, NIR jne)
* Analüüsiks valmis teadusandmed

### RAW (indeks)

**NDVI, NDRE, GNDVI jne. (\_NDVI.tif-failid):**

* Ühekanalilised halltoonilised pildid
* Pikselväärtused esindavad indeksite arvutustulemusi
* Normaliseeritud indeksite vahemik on tavaliselt -1 kuni +1
* Visualiseerimiseks saab rakendada värvilisi LUT-e

***

## Indeksi ja LUT-i rakendamine

Rakenda multispektraalseid indekseid ja värvilisi Look-Up Tabelid:

1. Leia **Index/LUT Sandbox** **Image Viewer**-is <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> külgribal
2. Valige taimestiku indeks (NDVI, NDRE jne)
3. Valige multispektraalne valem või looge oma kohandatud valem (ainult Chloros+)
4. Rakendage visualiseerimiseks värvilist LUT-gradienti
5. Reguleerige väärtuste vahemikke ja künniseid

Vaata [Indeks/LUT Sandbox](index-lut-sandbox.md) üksikasjalike juhiste saamiseks.

***

## Klahvikombinatsioonid

### Navigeerimine

* **→** (parem nool): Järgmine pilt
* **←** (vasak nool): Eelmine pilt
* **Home**: Nimekirja esimene pilt
* **End**: nimekirja viimane pilt

### Suumimine

* **+**või**=**: suumimine sisse
* **−**: suumimine välja
* **Hiireratt**: suumimine sisse/välja***

### Indeksite arvutuste kontrollimine

Kontrollige, kas indeksid on arvutatud õigesti:

1. Avage NDVI või muu indeksipilt
2. Kontrollige taimestikuga kaetud alasid:
   * **NDVI**: Tervete taimede puhul peaks näitama väärtust 0,4–0,9
   * **NDRE**: Kiire kasvu puhul on väärtused kõrgemad
   * **GNDVI**: sarnane NDVI-iga, kuid klorofüllitundlik
3. Kontrollige mitte-taimestikku:
   * **Muld**: ligikaudu 0 või veidi negatiivne
   * **Vesi**: negatiivsed väärtused (-0,5 kuni 0)***

## Vaateprobleemide lahendamine

### Pilt ei avane

**Võimalikud põhjused:**

* Fail on töötlemise käigus rikutud
* Toetamata failivorming
* Suure pildi jaoks pole piisavalt mälu

**Lahendused:**

1. Proovige avada fail välises vaatajas, et kontrollida faili terviklikkust
2. Kontrollige, kas failivorming vastab oodatavale tüübile
3. Sulgege teised rakendused, et vabastada mälu
4. Proovige väiksemat/teist pilti

### Pildi kuvamine mustana või valgena

**Võimalikud põhjused:**

* Väärtuste vahemik väljaspool kuvamisvõimalusi
* 32-bitine ujukomapilt ebatavaliste väärtustega
* Indeksi arvutamise viga

**Lahendused:**

1. Kontrollige pikselväärtusi – kui need on kõik väga madalad või väga kõrged, reguleerige kuvamisvahemikku
2. Proovige avada QGIS-is või sarnases programmis, kus on automaatne vahemiku reguleerimine
3. Kontrollige töötlemise vealokist, kas seal on vigu

### Pikselväärtused tunduvad valed

**Võimalikud põhjused:**

* Vaadeldakse valet pilti (originaal vs töödeldud)
* Kalibreerimine ei toiminud korrektselt
* Valgusanduri andmed ei olnud sisendisse kaasatud
* Protsendirežiim oli valesti sisse lülitatud

**Lahendused:**

1. Veenduge, et vaatate töödeldud väljundit (kontrollige failinime laiendit)
2. Kontrollige protsendirežiimi nupu seisundit
3. Võrrelge sama andmekogumist pärit teadaolevalt korrektsete piltidega

***

## Järgmised sammud

Nüüd, kui saate pilte täisekraanil vaadata:

* [**Pildikihtid**](image-layers.md) – Tutvuge mitme lainepikkuse visualiseerimisega
* [**Indeks/LUT-liivakast**](index-lut-sandbox.md) – Rakendage kohandatud indekseid ja värvikaardistust
* [**Multispektraalsed indeksivalemid**](../project-settings/multispectral-index-formulas.md) – Tutvuge saadaval olevate indeksitega

Töötlemise töövoo kohta vaadake:

* [**Piltide töötlemine (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Täielik töötlemisjuhend
