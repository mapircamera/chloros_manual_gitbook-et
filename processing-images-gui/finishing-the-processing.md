# Töötlemise lõpetamine

Kui Chloros on töötlemise lõpetanud, on aeg tulemusi üle vaadata, väljundi kvaliteeti kontrollida ja töödeldud pilte töövoos kasutamiseks ette valmistada. Käesolev lehekülg juhendab teid viimaste sammude ja järgmiste toimingute läbiviimisel.

## Töötlemise lõpetamise märge

Kui töötlemine lõpeb edukalt, näete mitmeid märke:

* ✅ **Edusammude riba**: jõuab 100% valmiduseni
* ✅ **Veakontrolli logi**: kuvab sõnumi „Töötlemine lõpetatud”
* ✅ **Käivitusnupp**: muutub taas aktiveerituks (valmis järgmiseks töötlemisvooruks)
* ✅ **Väljundfailid**: kõik töödeldud pildid on salvestatud kaameramudeli alamkausta***

## Töödeldud piltide leidmine

### Väljundkausta avamine

1. Klõpsake **peamenüü** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> ikoonile (ülemine vasak nurk)
2. Valige **„Ava projekti kaust”**

3. Failibrauser avab projekti kataloogi
4. Leidke oma projekt nime järgi

***

## Töödeldud piltide vaatamine

### Kiire eelvaade failibrauseris

**Windows sisseehitatud eelvaade:**

1. Navigeerige kaamera mudeli alamkausta
2. Valige pildifail
3. Eelvaade ilmub Windows Explorer eelvaateaknas
4. Kasutage nooleklahve piltide sirvimiseks

### Eelvaade välistes pildivaatajates

**Soovitatavad vaatajad:*** **QGIS** – tasuta GIS-tarkvara (parim georefereeritud multispektraalanalüüsiks)
* **IrfanView** – kiire ja kerge pildivaataja (toetab TIFF)
* **Adobe Photoshop** – professionaalne redigeerimine (TIFF tugi)
* **GIMP** – tasuta alternatiiv Photoshopile
* **Windows Photos** – põhiline vaatamine (võib mitte toetada 16-bitist TIFF-i)

### Eelvaade Chloros pildivaatajas

Kasutage Chloros sisseehitatud pildivaatajat täiustatud visualiseerimiseks:

1. Klõpsake failibrauseris pildi pisipildile
2. Pilt avaneb peamises eelvaatealas
3. Klõpsake vasakul külgribal **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> vasakul külgribal
4. Kasutage interaktiivseks analüüsiks [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md)

Vaadake üksikasjalikke juhiseid [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) alt.

***

## Debug Logi läbivaatamine

### Kontrollige hoiatusi ja vigu

1. Avage **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> vahekaarti
2. Sirvige sõnumeid
3. Otsige kollaseid hoiatusi või punaseid vigu
4. Vaadake läbi kõik märgitud probleemid
5. Abiks võtke ühendust MAPIR tugiteenistusega

### Logi salvestamine

Töötlemise andmete säilitamiseks või MAPIR tugiteenistusele saatmiseks:

1. Klõpsake nuppu **„Kopeeri”**või**„Laadi alla”**

2. Salvesta tekstifailina projekti kausta
3. Lisa projekti dokumentatsiooni
4. Saada MAPIR tugiteenistusele, kui tekib probleeme

***

## Tüüpilised väljundiga seotud probleemid ja lahendused

### Probleem: puuduvad väljundfailid

**Võimalikud põhjused:**

* Failid ei vastanud töötlemiskriteeriumidele
* Ainult sihtpildid (ekspordist välja jäetud)
* Ekspordi ajal sai kettaruum otsa
* Faili rikkumine töötlemise ajal

**Lahendused:**

1. Kontrollige vealokist vahelejätmise/veateateid
2. Veenduge, et kettaruumi oli piisavalt
3. Loendage faile: tulemus peaks vastama (algne arv – sihtarv) × (indeksid + 1)
4. Importige puuduvad failid uuesti ja töötlege need uuesti

### Probleem: tumedad või heledad servad (vignettimine on endiselt nähtav)

**Võimalikud põhjused:**

* Vignettide korrigeerimine on keelatud
* Kaamera/objektiiv puudub Chloros profiili andmebaasist
* Äärmuslik vignettimine, mida ei ole võimalik korrigeerida

**Lahendused:**

1. Veenduge, et vinjeti korrigeerimine on projektiseadetes sisse lülitatud
2. Kontrollige, kas kaamera mudel on õigesti tuvastatud
3. Kui vinjetatsioon püsib, võtke ühendust MAPIR tugiteenistusega

### Probleem: Ebaõiged värvid või väärtused

**Võimalikud põhjused:**

* Kalibreerimise sihtmärke ei tuvastatud
* Valitud on vale kalibreerimise sihtmärgi mudel
* Peegelduskalibreerimine on välja lülitatud
* Sihtmärkide pildid on halva kvaliteediga

**Lahendused:**

1. Kontrollige, kas peegelduskalibreerimine on sisse lülitatud
2. Kontrollige vealogi sõnumeid „Sihtmärk leitud”
3. Kontrollige sihtmärgi pildi kvaliteeti
4. Töötlege uuesti, märkides ära õiged sihtmärgid

### Probleem: NDVI väärtused tunduvad valed

**Oodatavad NDVI vahemikud:*** **Vesi, kivid, pinnas**: -0,1 kuni 0,2
* **Hõre/ebaterve taimestik**: 0,2 kuni 0,4
* **Mõõdukas taimestik**: 0,4 kuni 0,6
* **Terve, tihe taimestik**: 0,6 kuni 0,9**Kui väärtused jäävad väljapoole neid vahemikke:**

1. Kontrollige, kas peegelduskalibreerimine on rakendatud
2. Kontrollige, kas valgussensori logi on lisatud
3. Kontrollige, kas kalibreerimise sihtmärgid on tuvastatud
4. Veenduge, et on tuvastatud õige kaameramudel
5. Vaadake üle sihtmärgi pildistamise aeg ja tingimused

***

## Töödeldud piltide kasutamine

### Fotogramm-meetria / ortomoosaikide loomiseks

**Soovitatav töövoog:**

1.**Kalibreeritud peegeldusvõime piltide importimine** fotogramm-meetria tarkvarasse:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **EXIF-metadata säilitamine**: veenduge, et geotagimiseks on GPS-andmed säilitatud
3. **Kalibreeritud töövood**: kasutage teadusliku täpsuse tagamiseks peegeldusvõime pilte
4. **Töötle indeksmosaiike**: Loo NDVI ortomosaiike üksikutest indeksipiltidest
5. **Ekspordi georefereeritud GeoTIFF**: Kasutamiseks GIS-rakendustes

### GIS-analüüsiks

**Soovitatav töövoog:**

1.**Laadige QGIS-i, ArcGIS-i või sarnasse programmi**

2.**Kasutage 16-bitiseid TIFF** peegelduspilte mitme lainepikkuse analüüsiks
3. **Kasutage indeksipilte** (NDVI, NDRE) valmis kasutuseks taimestikukihtidena
4. **Rasterarvuti**: kombineeri laineid kohandatud analüüsi jaoks
5. **Eksport**: loo klassifitseerimiskaardid, muutuste tuvastamine, taimestiku tervisekaardid

### Otsese analüüsi / aruandluse jaoks

**Soovitatav töövoog:**

1.**Kasuta indeksipilte LUT-värvidega** visuaalsete aruannete jaoks
2. **Statistika väljavõtmine**: keskmine NDVI põllu/katseala kohta
3. **Aegrida**: indeksite võrdlemine mitme seansi vahel
4. **Aruannete koostamine**: lisage kaardid, statistika ja visualiseeringud***

## Arhiveerimine ja varundamine

### Soovitatav varundamisstrateegia

**Mida salvestada:*** ✅ **Algne RAW/JPG-pildid** – arhiveerige eraldi kettale/pilve
* ✅ **Töödeldud väljundid** – säilitage kalibreeritud pildid ja indeksid
* ✅ **Projektifail** – sisaldab kõiki seadeid vajadusel uuesti töötlemiseks
* ✅ **Veakontrolli logi** – dokumenteerib töötlemise üksikasjad
* ✅ **Kalibreerimise sihtmärgipildid** - Kontrollimiseks ja uuesti töötlemiseks**Soovitused salvestamiseks:*** **Kohene varundamine**: Välisketas
* **Pikaajaline arhiiv**: Pilvesalvestus (Google Drive, Dropbox jne.)
* **Kriitilised andmed**: Säilita 2–3 koopiat erinevates asukohtades***

## Järgmised töötlemistsüklid

### Projekti seadete taaskasutamine

Kui töötlete tulevikus sarnaseid andmekogumeid:

1. **Salvestage projekti mall** (kui seda pole veel tehtud)
2. **Looge uus projekt** salvestatud malli abil
3. **Importige uued pildid**

4.**Töötlege**ühtsuse tagamiseks identseid seadeid kasutades

### Mitme seansi kogumitöötlus

Mitme seansi/andmekogumi puhul:**Võimalus 1: GUI – mitu projekti**

* Loo iga seansi jaoks eraldi projekt
* Kasuta ühtseid malliseadeid
* Töötle ükshaaval

**Võimalus 2: Chloros CLI (ainult Chloros+)**

* Automatiseeri kogumitöötlus
* Töötle mitut kausta skriptide abil
* Vaata [CLI dokumentatsiooni](../CLI.md)

**Võimalus 3: Python SDK (ainult Chloros+)**

* Programmeeritud juhtimine
* Integreerimine analüüsipipeliinidega
* Vaata [API dokumentatsiooni](../api-python-sdk.md)

***

## Järelkäsitluse veaotsing

### Uuesti töötlemine erinevate seadete abil

Kui tulemused ei ole rahuldavad:

1. Säilita originaalpildid (ära kunagi kustuta)
2. Avage sama projekt Chloros-is
3. Reguleerige seadeid projekti seadepaneelis
4. Töötlege uuesti – väljundid kirjutavad üle eelmised tulemused

### Piltide alarühma töötlemine

Konkreetsete piltide uuesti töötlemiseks:

1. Looge uus projekt
2. Importige ainult need pildid, mida on vaja uuesti töötleda
3. Kasutage sama seadete mall
4. Töötlege väiksemat andmekogumit

### Abi saamine

Kui tekib probleeme:

* 📧 **E-post**: info@mapir.camera (lisage vealogi)
* 🌐 **Tugi**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **KKK**: [Korduma kippuvad küsimused](../faq.md)
* 📖 **Dokumentatsioon**: [Chloros kasutusjuhend](../)***

## Kokkuvõte: Töövoo lõpetamine

Olete nüüd läbinud kogu Chloros töötlemise töövoo:

1. ✅ **Projekt loodud** – vaadake [Projekte](../projects.md)
2. ✅ **Lisatud failid** – vaata [Failide lisamine](adding-files-to-a-project.md)
3. ✅ **Seadistused kohandatud** – vaata [Projekti seadistuste kohandamine](adjusting-project-settings.md)
4. ✅ **Märgitud sihtmärgid** – vaata [Sihtmärkide valimine](choosing-target-images.md)
5. ✅ **Töötlemine alustatud** – vaata [Töötlemise alustamine](starting-the-processing.md)
6. ✅ **Jälgitud protsessi kulgu** – vaata [Töötlemise jälgimine](monitoring-the-processing.md)
7. ✅ **Tulemused läbi vaadatud** – see lehekülg**Sinu kalibreeritud, peegeldusega korrigeeritud multispektraalsed pildid on valmis analüüsimiseks!**

***

## Lisaressursid

### Täiustatud funktsioonid

* [**Pildivaataja**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktiivne visualiseerimine ja analüüs
* [**Indeksi/LUT-liivakast**](../image-viewer-gui/index-lut-sandbox.md) – Kohandatud indeksite testimine
* [**Multispektraalsed indeksivalemid**](../project-settings/multispectral-index-formulas.md) – Täielik indeksiviide

### Automatiseerimine ja integratsioon

* [**CLI dokumentatsioon**](../CLI.md) – Käsurea kaudu toimuv pakettide töötlemine
* [**Python SDK**](../api-python-sdk.md) – Programmeeritud automatiseerimine
* [**Chloros+ funktsioonid**](../#chloros) – Täiustatud töötlemisvõimalused

### Tugi ja õppimine

* [**KKK**](../faq.md) – vastused levinud küsimustele
* [**Kalibreerimismärgid**](../calibration-targets.md) – peegelduskalibreerimise põhimõtted
* [**Toetatud kaamerad**](../supported-cameras.md) – Ühilduv riistvara
