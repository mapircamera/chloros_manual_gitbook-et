# Töötlemise lõpetamine

Kui Chloros on töötlemise lõpetanud, on aeg tulemusi üle vaadata, väljundi kvaliteeti kontrollida ja töödeldud pilte töövoos kasutamiseks ette valmistada. Käesolev lehekülg juhendab teid viimaste sammude ja järgmiste tegevuste läbiviimisel.

## Töötlemise lõpetamise märgid

Kui töötlemine lõpeb edukalt, näed mitmeid märgid:

* ✅ **Edusammude riba**: jõuab 100%ni
* ✅ **Vigade logi**: näitab viimaseid `[RUN-SUMMARY]` ridu koos arvudega (pildid, kaamerarühmad, sihtmärgid, kalibreeritud pildid, salvestatud failid)
* ✅ **Käivitusnupp**: muutub uuesti aktiveerituks (valmis järgmiseks töötlemisvooruks)
* ✅ **Väljundfailid**: kõik töödeldud pildid on salvestatud projekti väljundkataloogi (allpool)

{% hint style="warning" %}
**Käivitus, mille käigus ei salvestata ühtegi pilti, loetakse ebaõnnestunuks.** Kui te taotlesite pilditooteid ja käivituse käigus neid ei salvestatud, teatab Chloros sellest kui ebaõnnestumisest — `[RUN-SUMMARY]` vihjab logi nimes tõenäolisele põhjusele (midagi ei imporditud, sihtmärki ei tuvastatud või kõik taotletud tooted jäeti kõrvale kui mittekohaldatavad). CLI-i ekvivalent lõpetab töötamise nullist erineva tulemusega. Tahtlikult ainult metaandmetele suunatud käivitus (kõik eksporditooted välja lülitatud, indeksid puuduvad) loetakse ikkagi edukaks. Vaata [CLI-i viidet](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Töödeldud piltide leidmine

### Väljundkausta avamine

1. Klõpsake **peamenüü** ikooni „<img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">“ (ülemine vasak nurk)
2. Valige **„Open Project Folder“**

3. Failibrauser avab projekti kataloogi
4. Leidke oma projekt nime järgi

### Väljundpuu

Tulemused salvestatakse **projekti kausta alla, rühmitatuna kaamera ja seejärel failivormingu järgi**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Kaamera kaust**: `LATT-<sensor>-<lens>-F<filter>` kaamera LATTICE jaoks (vastab pildi EXIF-andmetele `Model`), `<model>_<filter>` kaamera Survey3 jaoks (nt `Survey3N_RGN`). Kaks kaamerat, millel on ühine sensor ja filter, kuid erinev objektiiv, hoitakse eraldi puudes – vinjetatsioon, vaateväli ja moonutus erinevad.
* **Formaatikausta**: järgib teie ekspordiformaadi seadet – `tiff16`, `tiff8`, `png8`, `jpg8` või `tiff32`, kui TIFF (32-bitine, protsent). Kiirgus on alati float32 ja salvestatakse alati kausta `tiff32` alla.
* **Tootekaustad**:
  * `Reflectance_Calibrated_Images/` — kalibreeritud peegeldusvõime
  * `Debayered_Images/` — lineaarselt debayereeritud (LATTICE)
  * `Preview_Images/` — ekraanil eelvaade (LATTICE)
  * `Radiance_Images/` — float32 spektraalne kiirgusvõimsus, W/m²/sr/nm (LATTICE multispektraalne)
  * `Vignette_Corrected_Images/` **või** `Sensor_Response_Images/` — kalibreerimata varuväärtus kaadritele, millel puudub peegeldusvõime viide; iga käivituse kohta on olemas täpselt üks neist kahest, mis valitakse Vignette&#x27;i korrigeerimise seadistuse alusel
  * `<INDEX>_Index_Images/` — üks kaust valitud indeksi kohta (nt `NDVI_Index_Images`)

{% hint style="info" %}
**Iga eksporditud toode säilitab ALGFAILI nime.**`capture_..._raw.tif` kiirguse eksport kannab endiselt nime `capture_..._raw.tif` — see asub lihtsalt kataloogis `tiff32/Radiance_Images/`.**Toote identifitseerib kaust, mitte failinimi**, seega `*radiance*.tif` otsing ei anna tulemusi; otsige selle asemel kataloogi järgi.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Kui palju faile peaks olema?

Ärge arvutage valemi järgi – väljundite arv sõltub sellest, millised tooted olid aktiveeritud ja millised kehtivad iga kaamera puhul (nt RGB-kaamerad ei saa kiirgust/peegeldust). Ametlik arv on logis: viimane rida `[RUN-SUMMARY]` näitab täpselt, kui palju faile kirjutati, ja selgitavad read kirjeldavad kõiki vahele jäetud faile.

***

## Töödeldud piltide vaatamine

### Kiire eelvaade failibrauseris

**Windows sisseehitatud eelvaade:**

1. Navigeerige tootekausta (nt `tiff16/Reflectance_Calibrated_Images/`)
2. Valige pildifail
3. Eelvaade ilmub Windows Explorer&#x27;i eelvaateaknas
4. Kasutage nooleklahve piltide sirvimiseks

### Eelvaade välistes pildivaatajates

**Soovitatavad vaatajad:*** **QGIS** – tasuta GIS-tarkvara (sobib kõige paremini georefereeritud multispektraalanalüüsiks)
* **IrfanView** – kiire ja kerge pildivaataja (toetab TIFF-i)
* **Adobe Photoshop** – professionaalne pilditöötlus (toetab TIFF-i)
* **GIMP** – tasuta alternatiiv Photoshopile
* **Windows Photos** – põhiline vaatamine (võib mitte toetada 16-bitist TIFF-i)

### Eelvaade Chloros pildivaatajas

Kasutage Chloros sisseehitatud pildivaatajat täpsemaks vaatamiseks:

1. Klõpsake failibrauseris pildi pisipildile
2. Pilt avatakse peamises eelvaatealas
3. Klõpsake vasakul külgribal asuval vahekaardil **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
4. Interaktiivseks analüüsiks kasutage [Indeks/LUT-liivakasti](../image-viewer-gui/index-lut-sandbox.md)

Täpsemad juhised leiate [Pildivaatajast](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Peegeldusvõime pikselväärtuste lugemine (GIS / Pix4D / skriptid)

Peegeldusvõime salvestatakse täisarvuna DN ning **DN, mis tähendab ρ = 1,0, sõltub allikakaamerast**:

| Allikas          | ρ = 1,0 on | Kuidas kindlaks teha                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (varu kuni ρ 2,0) | Failile on lisatud XMP-märge `Chloros:PixelScale=32768` |
| Survey3         | **65535** (piiratud väärtusega ρ 1,0)     | XMP-märgiseid `Chloros:*` ei ole — see puudumine on märgiks |

**Loe `Chloros:PixelScale`-märget ja jaga sellega**, selle asemel et eeldada üldist väärtust 65535 — LATTICE-peegeldusvõime jagamine 65535-ga vähendab vaikimisi iga väärtuse poole võrra. Üks äärejuhtum ei sisalda disainilt mingit skaalat: 8-bitise allikaga salvestus, mis on kirjutatud 8-bitise väljundina, kärbitakse, mitte ei skaalata ümber, ning sellele ei lisata tahtlikult skaalamärgist — ekspordi see uuesti 16-bitisena või 32-bitisena, selle asemel et seda jagada. Vaata [Väljundpildi formaadid](../output-image-formats.md) täieliku ülevaate saamiseks.***

## Eksportimisel kaasa kantavad metaandmed

Iga toode säilitab allika pildi **GPS-ploki**ja selle**EXIF-alam-IFD**, seega
sisaldab eksport `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` ja
`CameraSerialNumber` ning georeferentsi.

{% hint style="warning" %}
**Kui ortomoosaik tuleb välja absurdses mõõtkavas, kontrollige esmalt `FocalLength`-i.**
Pix4D arvutab maapinna proovivahe kauguse fokaalkauguse ja kõrguse põhjal. Ilma selle märgistuseta
kasutatakse tagavara-mõõtkava, mis on täiesti vale – ühel mõõdetud 49-kaadri lennul rekonstrueeriti 411 m × 160 m
suur apelsinipuuaed 47,8 km × 13 km suuruseks, mille tulemusena tekkis 455-megapiksline ortomosaik, mis koosnes peamiselt
tühjast ruumist. Aeglane piltide kokkupanek ja ootamatult suur fail on selle sümptomid, mitte eraldi
probleemid.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Mitte *kõiki* sildid ei kopeerita. IFD0 struktuurilised sildid jäetakse tahtlikult välja (nende kopeerimine
rikub LATTICE väljundit), ning `ExifImageWidth` / `ExifImageHeight` jäetakse välja,
kuna need kirjeldavad algset pildistust – muidu väidaks suurusega muudetud eksport
mõõtmeid, mis on vastuolus selle enda rasteriga.

***

## Vealogi läbivaatamine

### Hoiatuste ja vigade kontrollimine

1. Avage **Debug Log** (vealogi) vaheka<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Sirvige sõnumeid
3. Otsige kollaseid hoiatusi või punaseid vigu
4. Lugege `[RUN-SUMMARY]` ridu ja kõiki vihjeid
5. Pöörduge abi saamiseks MAPIR-i tugiteenistuse poole

### Logi salvestamine

Töötlemise dokumenteerimiseks või MAPIR-i tugiteenistusele saatmiseks:

1. Klõpsake nuppu **„Kopeeri”**või**„Laadi alla”**

2. Salvesta tekstifailina projekti kausta
3. Lisage see projekti dokumentatsiooni
4. Saatke MAPIR tugiteenistusele, kui tekib probleeme

***

## Tüüpilised väljundiga seotud probleemid ja lahendused

### Probleem: puuduvad väljundfailid

**Võimalikud põhjused:**

* Toode ei sobi selle kaameraga (nt kiirgus/peegeldus RGB kaamerate puhul – logis on see märgitud)
* Puudus nõutav viide (nt peegeldusvõime ilma sihtmärgi ja `.daq` allapoole suunatud kiirguseta)
* Toote ekspordi valikukast oli projekti seadetest keelatud
* Ekspordi ajal sai kettaruum otsa

**Lahendused:**

1. Kontrollige veakirjas ridu `[RUN-SUMMARY]` ja `[EXPORT-CHECK]` – need selgitavad kaamerapõhiseid vahelejätmisi
2. Kontrollige eksporditoote valikukaste [Projekti seaded](adjusting-project-settings.md)
3. Veenduge, et kettaruumi oli piisavalt
4. Pärast põhjuse kõrvaldamist töötlege andmed uuesti

### Probleem: tumedad või heledad servad (vignettimine on endiselt nähtav)

**Võimalikud põhjused:**

* Vignettkorrektsioon on välja lülitatud
* Kaamera/objektiiv puudub Chloros profiilide andmebaasist
* Äärmuslik vignett, mida ei ole võimalik korrigeerida

**Lahendused:**

1. Veenduge, et vinjeti korrigeerimine on projektiseadetes sisse lülitatud
2. Kontrollige, kas kaamera mudel on õigesti tuvastatud
3. Kui vinjetatsioon püsib, võtke ühendust MAPIR tugiteenistusega

### Probleem: Ebaõiged värvid või väärtused

**Võimalikud põhjused:**

* Kalibreerimismärke ei tuvastatud
* Valiti vale kalibreerimismärgi mudel
* Peegelduskalibreerimine on välja lülitatud
* Kalibreerimismärkide pildid on halva kvaliteediga

**Lahendused:**

1. Veenduge, et peegelduskalibreerimine on sisse lülitatud
2. Kontrollige veateadete logist sõnumeid „Märk leitud“
3. Kontrollige sihtmärgi pildi kvaliteeti
4. Töötlege uuesti, märkides ära õiged sihtmärgid

### Probleem: NDVI väärtused tunduvad valed

**Oodatavad NDVI vahemikud:*** **Vesi, kivid, pinnas**: -0,1 kuni 0,2
* **Hõre/ebaterve taimestik**: 0,2 kuni 0,4
* **Mõõdukas taimestik**: 0,4 kuni 0,6
* **Terve, tihe taimestik**: 0,6 kuni 0,9**Kui väärtused jäävad väljapoole neid vahemikke:**

1. Veenduge, et peegelduskalibreerimine on rakendatud
2. Veenduge, et valgusanduri logi on lisatud
3. Kontrollige, kas kalibreerimismärgid on tuvastatud
4. Veenduge, et on tuvastatud õige kaameramudel
5. Vaadake üle sihtmärkide pildistamise ajastus ja tingimused
6. Kui arvutate indekseid ise peegeldusfailide põhjal, veenduge, et jagasite tulemuse faili `Chloros:PixelScale`-iga (vt eespool)

***

## Töödeldud piltide kasutamine

### Fotogramm-meetria / ortomozaigi loomiseks

**Soovitatav töövoog:**

1.**Importige kalibreeritud peegelduspilte** fotogramm-meetria tarkvarasse:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Säilitage EXIF-metadata**: veenduge, et GPS-andmed jäävad alles geotagimiseks
3. **Kalibreeritud töövood**: Kasutage teadusliku täpsuse tagamiseks peegelduspilte — LATTICE peegelduspildid sisaldavad XMP-kalibreerimismärke, mida Pix4D loeb
4. **Töötlege indeksmosaiike**: Looge üksikutest indeksipiltidest ortomosaiikid nimega NDVI
5. **Ekspordi georefereeritud GeoTIFF**: kasutamiseks GIS-rakendustes

### GIS-analüüsiks

**Soovitatav töövoog:**

1.**Laadi QGIS-i, ArcGIS-i või sarnasse programmi**

2.**Kasuta 16-bitiseid TIFF** peegelduspilte mitme lainepikkuse analüüsiks (jaga faili `Chloros:PixelScale`-ga)
3. **Kasutage indeksipilte** (NDVI, NDRE) valmis kasutuseks mõeldud taimestikukihtidena
4. **Rasterarvuti**: kombineeri laineid kohandatud analüüsi jaoks
5. **Eksport**: loo klassifitseerimiskaardid, muutuste tuvastamine, taimestiku tervisekaardid

### Otsese analüüsi / aruandluse jaoks

**Soovitatav töövoog:**

1.**Kasuta indeksipilte LUT-värvidega** visuaalsete aruannete jaoks
2. **Statistika väljavõtmine**: NDVI keskmine väärtus põllu/katseala kohta
3. **Aegrida**: indeksite võrdlemine mitme seansi vahel
4. **Aruannete koostamine**: lisage kaardid, statistika ja visualiseeringud***

## Arhiveerimine ja varundamine

### Soovitatav varundamisstrateegia

**Mida salvestada:*** ✅ **Algupärased RAW/JPG-pildid või LATTICE-i toorandmed** – arhiveerige eraldi kettale/pilve; toorandmed on töötlemisprotsessi allikas ja kõike muud saab nende põhjal uuesti genereerida
* ✅ **`.daq` / `.csv` valgusanduri failid** – vajalikud peegeldusvõime hilisemaks uuesti arvutamiseks
* ✅ **Töödeldud väljundid** – säilita kalibreeritud pildid ja indeksid
* ✅ **Projekti kaust** (`project.json` ja sellega seotud failid) – sisaldab kõiki seadeid, mis on vajalikud töötlemise kordamiseks, kui see osutub vajalikuks
* ✅ **Veakiri** – dokumenteerib töötlemise üksikasju
* ✅ **Kalibreerimise sihtpildid** – kontrollimiseks ja uuesti töötlemiseks**Säilitamise soovitused:*** **Viivitamatu varukoopia**: väline kõvaketas
* **Pikaajaline arhiiv**: pilvesalvestus (Google Drive, Dropbox jne.)
* **Kriitilised andmed**: hoidke 2–3 koopiat erinevates asukohtades***

## Järgmised töötlemistsüklid

### Projektiseadete taaskasutamine

Kui töötlete tulevikus sarnaseid andmekogumeid:

1. **Salvestage projektimall** (kui seda pole veel tehtud)
2. **Loo uus projekt** salvestatud malli abil
3. **Impordi uued pildid**

4.**Töötle**ühtsuse tagamiseks samade seadistustega

### Mitme seansi kogumitöötlus

Mitme seansi/andmekogumi puhul:**Võimalus 1: graafiline kasutajaliides (GUI) – mitu projekti**

* Loo iga seansi jaoks eraldi projekt
* Kasutage ühtseid malli seadeid
* Töötlege ükshaaval

**Võimalus 2: Chloros CLI (ainult Chloros+)**

* Automatiseeri kogumitöötlus
* Töötle mitut kausta skriptide abil
* Vaata [CLI dokumentatsiooni](../CLI.md) ja [CLI viiteid](../reference/cli-reference.md)

**

3. võimalus: Python SDK (ainult Chloros+)**

* Programmeeritav juhtimine
* Integreerimine analüüsipipeliinidega
* Vaata [API dokumentatsiooni](../api-python-sdk.md) ja [SDK viite](../reference/sdk-reference.md)

***

## Järel töötlemise veaotsing

### Uuesti töötlemine erinevate seadistustega

Kui tulemused ei ole rahuldavad:

1. Säilitage originaalpildid (ärge kunagi kustutage)
2. Avage sama projekt Chloros-is
3. Reguleerige seadeid paneelis „Project Settings“
4. Töötlege uuesti — väljundid salvestatakse samadesse tootekaustadesse, seega asendatakse eelmise töötlemiskorra samanimelised failid

### Piltide alarühma töötlemine

Ainult teatud piltide uuesti töötlemiseks:

1. Loo uus projekt
2. Impordi ainult need pildid, mida on vaja uuesti töödelda
3. Kasuta sama seadete mall
4. Töötle väiksemat andmekogumit

### Abi saamine

Kui tekib probleeme:

* 📧 **E-post**: info@mapir.camera (lisage vealogi)
* 🌐 **Tugi**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **KKK**: [Korduma kippuvad küsimused](../faq.md)
* 📖 **Dokumentatsioon**: [Chloros kasutusjuhend](../)***

## Kokkuvõte: Täielik töövoog

Olete nüüd läbinud kogu Chloros töötlemise töövoo:

1. ✅ **Projekt loodud** – vt [Projektid](../projects.md)
2. ✅ **Lisatud failid** – vt [Failide lisamine](adding-files-to-a-project.md)
3. ✅ **Seadistused kohandatud** – vt [Projekti seadistuste kohandamine](adjusting-project-settings.md)
4. ✅ **Sihtmärgid märgitud** – vaata [Sihtmärkide valimine](choosing-target-images.md)
5. ✅ **Töötlemine alustatud** – vaata [Töötlemise alustamine](starting-the-processing.md)
6. ✅ **Jälgitud töötlemise kulgu** – vt [Töötlemise jälgimine](monitoring-the-processing.md)
7. ✅ **Tulemused läbi vaadatud** – see lehekülg**Teie kalibreeritud ja peegeldusega korrigeeritud multispektraalsed pildid on valmis analüüsimiseks!**

***

## Lisamaterjalid

### Täiustatud funktsioonid

* [**Pildivaataja**](../image-viewer-gui/opening-an-image-full-screen.md) – interaktiivne visualiseerimine ja analüüs
* [**Indeksi/LUT-katsetuskeskkond**](../image-viewer-gui/index-lut-sandbox.md) – kohandatud indeksite testimine
* [**Multispektraalsed indeksivalemid**](../project-settings/multispectral-index-formulas.md) – täielik indeksite viitekogu

### Automatiseerimine ja integreerimine

* [**CLI dokumentatsioon**](../CLI.md) – käsurealt käivitatav paketttöötlus
* [**Python SDK**](../api-python-sdk.md) – Programmeeritud automatiseerimine
* [**Chloros+ Funktsioonid**](../#chloros) – Täiustatud töötlemisvõimalused

### Tugi ja õppematerjalid

* [**KKK**](../faq.md) – Vastused levinud küsimustele
* [**Kalibreerimismärgid**](../calibration-targets.md) – Peegeldusvõime kalibreerimise põhimõtted
* [**Toetatud kaamerad**](../supported-cameras.md) – Ühilduv riistvara
