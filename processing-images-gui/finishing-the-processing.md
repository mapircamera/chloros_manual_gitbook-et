# Töötlemise lõpetamine

Kui Chloros on töötlemise lõpetanud, on aeg vaadata üle tulemused, kontrollida väljundi kvaliteeti ja valmistada töödeldud pildid ette kasutamiseks oma töövoos. See lehekülg juhendab teid viimaste sammude ja järgmiste tegevuste läbi.

## Töötlemise lõpetamise märge

Kui töötlemine on edukalt lõpetatud, näete mitmeid märke:

* ✅ **Progressiriba**: jõuab 100% täitmiseni
* ✅ **Debug Log**: näitab sõnumit „Processing Complete” (Töötlemine lõpetatud)
* ✅ **Start button**: muutub uuesti aktiveerituks (valmis järgmiseks töötlemiseks)
* ✅ **Väljundfailid**: kõik töödeldud pildid salvestatakse kaamera mudeli alamkausta

***

## Töödeldud piltide leidmine

### Väljundkausta avamine

1. Klõpsake **peamenüü** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (ülemine vasak nurk)
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

**Soovitatavad vaatajad:**

* **QGIS** – tasuta GIS-tarkvara (parim georeferentseeritud multispektraalanalüüsiks)
* **IrfanView** – kiire ja kerge pildivaataja (toetab TIFF)
* **Adobe Photoshop** – professionaalne redigeerimine (TIFF tugi)
* **GIMP** – tasuta alternatiiv Photoshopile
* **Windows Photos** – põhiline vaatamine (ei pruugi toetada 16-bitist TIFF)

### Eelvaade Chloros pildivaatajas

Kasutage Chloros sisseehitatud pildivaatajat täiustatud visualiseerimiseks:

1. Klõpsake failibrauseris pildi pisipilti.
2. Pilt avatakse peamises eelvaatealas.
3. Klõpsake vasakul külgribal **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> vasakul külgribal.
4. Kasutage interaktiivseks analüüsiks [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md).

Täpsemad juhised leiate [Image Viewerist](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Debug Logi läbivaatamine

### Kontrollige hoiatusi ja vigu

1. Avage **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> sakk
2. Sirvige sõnumeid
3. Otsige kollaseid hoiatusi või punaseid vigu
4. Vaadake läbi kõik märgitud probleemid
5. Abiks võtke ühendust MAPIR tugiteenistusega

### Logi salvestamine

Töötlemise salvestamiseks või MAPIR tugiteenistusele saatmiseks:

1. Klõpsake nuppu **„Kopeeri”** või **„Lae alla”**
2. Salvesta tekstifailina projekti kausta
3. Lisa projekti dokumentatsioonile
4. Saada MAPIR tugiteenistusele, kui tekib probleeme

***

## Tavapärased väljundiprobleemid ja lahendused

### Probleem: puuduvad väljundifailid

**Võimalikud põhjused:**

* Failid ei vastanud töötlemiskriteeriumidele.
* Ainult sihtpildid (ekspordist välja jäetud).
* Ekspordi ajal sai kettaruum otsa.
* Failide rikkumine töötlemise ajal.

**Lahendused:**

1. Kontrollige veaotsingu logist, kas seal on vahelejätmise/veateateid.
2. Veenduge, et kettaruumi oli piisavalt.
3. Loendage faile: peaks vastama (algne arv – sihtarv) × (indeksid + 1).
4. Importige puuduvad failid uuesti ja töötlege need uuesti.

### Probleem: tumedad või heledad servad (vinjetatsioon on endiselt nähtav)

**Võimalikud põhjused:**

* Vinjetatsiooni korrigeerimine on keelatud.
* Kaamera/objektiiv ei ole Chloros profiili andmebaasis.
* Äärmuslik vinjetatsioon, mida ei ole võimalik korrigeerida.

**Lahendused:**

1. Kontrollige, kas vinjeti korrigeerimine on projektiseadetes lubatud
2. Kontrollige, kas kaamera mudel on õigesti tuvastatud
3. Kui vinjetatsioon püsib, võtke ühendust MAPIR tugiteenistusega

### Probleem: vale värv või väärtus

**Võimalikud põhjused:**

* Kalibreerimise sihtmärke ei tuvastatud
* Valitud on vale kalibreerimise sihtmärgi mudel
* Peegelduskalibreerimine on keelatud
* Sihtmärgi pildid on halva kvaliteediga

**Lahendused:**

1. Kontrollige, kas peegelduskalibreerimine on lubatud.
2. Kontrollige veaotsingu logis sõnumit „Sihtmärk leitud”.
3. Kontrollige sihtmärgi pildi kvaliteeti.
4. Töötlege uuesti, märkides õiged sihtmärgid.

### Probleem: NDVI väärtused tunduvad valed.

**Oodatavad NDVI vahemikud:**

* **Vesi, kivid, pinnas**: -0,1 kuni 0,2
* **Hõre/ebatervislik taimestik**: 0,2 kuni 0,4
* **Mõõdukas taimestik**: 0,4 kuni 0,6
* **Tervislik, tihe taimestik**: 0,6 kuni 0,9

**Kui väärtused jäävad väljapoole neid vahemikke:**

1. Kontrollige, kas peegelduskalibreerimine on rakendatud.
2. Kontrollige, kas valgussensori logi on lisatud.
3. Kontrollige, kas kalibreerimise sihtmärgid on tuvastatud.
4. Veenduge, et on tuvastatud õige kaameramudel.
5. Kontrollige sihtmärgi pildi jäädvustamise aega ja tingimusi.

***

## Töödeldud piltide kasutamine

### Fotogramm-meetria / ortomosaigi loomiseks

**Soovitatav töövoog:**

1. **Importige kalibreeritud peegeldusvõime pildid** fotogramm-meetria tarkvarasse:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Säilitage EXIF-metadata**: veenduge, et GPS-andmed on geotagimiseks säilitatud
3. **Kalibreeritud töövood**: kasutage peegeldusvõime pilte teadusliku täpsuse tagamiseks
4. **Töötlege indeksmosaiike**: Looge NDVI ortomosaiigid üksikutest indeksipiltidest
5. **Eksportige georeferentseeritud GeoTIFF**: kasutamiseks GIS-rakendustes

### GIS-analüüsiks

**Soovitatav töövoog:**

1. **Laadige QGIS-i, ArcGIS-i või sarnasesse**
2. **Kasutage 16-bitiseid TIFF** peegelduspilte mitme sagedusriba analüüsiks
3. **Kasutage indekspilte** (NDVI, NDRE) valmis kasutuseks taimestiku kihtidena
4. **Rasterarvuti**: kombineerige sagedusribad kohandatud analüüsiks
5. **Eksport**: looge klassifitseerimiskaardid, muutuste tuvastamine, taimestiku tervisekaardid

### Otseseks analüüsiks / aruandluseks

**Soovitatav töövoog:**

1. **Kasutage indekspilte LUT-värvidega** visuaalsete aruannete jaoks
2. **Väljavõtted statistika**: keskmine NDVI põllu/krundi kohta
3. **Aegread**: võrdle indekseid mitme seansi vahel
4. **Aruannete koostamine**: lisa kaardid, statistika ja visualiseeringud

***

## Arhiveerimine ja varundamine

### Soovitatav varundamisstrateegia

**Mida salvestada:**

* ✅ **Originaalsed RAW/JPG pildid** – arhiveeri eraldi kettal/pilves
* ✅ **Töödeldud väljundid** – hoidke kalibreeritud pilte ja indekseid
* ✅ **Projektifail** – sisaldab kõiki vajaduse korral töötlemiseks vajalikke seadeid
* ✅ **Debug-logi** – dokumenteerib töötlemise üksikasjad
* ✅ **Kalibreerimise sihtpildid** – kontrollimiseks ja töötlemiseks

**Säilitamise soovitused:**

* **Vahetu varukoopia**: välisele kõvakettale
* **Pikaajaline arhiiv**: pilvesalvestus (Google Drive, Dropbox jne)
* **Kriitilised andmed**: hoidke 2–3 koopiat erinevates asukohtades

***

## Järgmised töötlemiskäigud

### Projekti seadete taaskasutamine

Kui töötlete tulevikus sarnaseid andmekogusid:

1. **Salvestage projektimall** (kui seda pole veel tehtud)
2. **Looge uus projekt** salvestatud malli abil
3. **Importige uued pildid**
4. **Töötlege** ühtsuse tagamiseks identseid seadeid kasutades

### Mitme seansi kogumitöötlus

Mitme seansi/andmekogumi puhul:

**Võimalus 1: GUI – mitu projekti**

* Looge iga seansi jaoks eraldi projekt.
* Kasutage ühtseid malliseadeid.
* Töötlege ükshaaval.

**Võimalus 2: Chloros CLI (ainult Chloros+)**

* Automatiseerige partii töötlemine.
* Töötlege mitut kausta skriptidega.
* Vaata [CLI dokumentatsiooni](../CLI.md)

**Variant 3: Python SDK (ainult Chloros+)**

* Programmiline juhtimine
* Integreerimine analüüsi torujuhtmete
* Vaata [API dokumentatsiooni](../api-python-sdk.md)

***

## Järelkäsitluse veaotsing

### Uuesti töötlemine erinevate seadete abil

Kui tulemused ei ole rahuldavad:

1. Säilita originaalpildid (ära kunagi kustuta)
2. Avage sama projekt Chloros-is
3. Kohandage seadeid projekti seadeid paneelis
4. Töötlege uuesti – väljundid kirjutavad üle eelmised tulemused

### Piltide alarühma töötlemine

Ainult teatud piltide uuesti töötlemiseks:

1. Looge uus projekt
2. Importige ainult uuesti töötlemist vajavad pildid
3. Kasutage sama seadete mall
4. Töötlege väiksemat andmekogumit

### Abi saamine

Kui tekib probleeme:

* 📧 **E-post**: info@mapir.camera (lisage veaotsingu logi)
* 🌐 **Tugi**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **KKK**: [Korduma kippuvad küsimused](../faq.md)
* 📖 **Dokumentatsioon**: [Chloros kasutusjuhend](../)

***

## Kokkuvõte: täielik töövoog

Nüüd olete lõpetanud kogu Chloros töötlemise töövoo:

1. ✅ **Loodud projekt** – vaadake [Projekte](../projects.md)
2. ✅ **Lisatud failid** – vaata [Failide lisamine](adding-files-to-a-project.md)
3. ✅ **Kohandatud seaded** – vaata [Projekti seadete kohandamine](adjusting-project-settings.md)
4. ✅ **Märgitud sihtmärgid** – vaata [Sihtmärgi piltide valimine](choosing-target-images.md)
5. ✅ **Alustatud töötlemine** – vaata [Töötlemise alustamine](starting-the-processing.md)
6. ✅ **Jälgitud edenemine** – vaata [Töötlemise jälgimine](monitoring-the-processing.md)
7. ✅ **Tulemuste läbivaatamine** – See lehekülg

**Teie kalibreeritud, peegelduskorrektsiooniga multispektraalsed pildid on valmis analüüsimiseks!**

***

## Lisaressursid

### Täiustatud funktsioonid

* [**Pildivaataja**](../image-viewer-gui/opening-an-image-full-screen.md) – interaktiivne visualiseerimine ja analüüs
* [**Indeks/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – kohandatud indeksi testimine
* [**Multispektraalsed indeksivalemid**](../project-settings/multispectral-index-formulas.md) – täielik indeksite viide

### Automatiseerimine ja integreerimine

* [**CLI dokumentatsioon**](../CLI.md) – käsurea partii töötlemine
* [**Python SDK**](../api-python-sdk.md) – Programmiline automatiseerimine
* [**Chloros+ funktsioonid**](../#chloros) – Täiustatud töötlemisvõimalused

### Tugi ja õppimine

* [**KKK**](../faq.md) – vastused tavapärastele küsimustele
* [**Kalibreerimise sihtmärgid**](../calibration-targets.md) – peegelduskalibreerimise mõistmine
* [**Toetatud kaamerad**](../supported-cameras.md) – ühilduv riistvara
