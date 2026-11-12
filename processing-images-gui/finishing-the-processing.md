# Töötlemise lõpetamine

Kui Chloros on töötlemise lõpetanud, on aeg tulemusi üle vaadata, väljundi kvaliteeti kontrollida ja töödeldud pilte töövoos kasutamiseks ette valmistada. Käesolev lehekülg juhendab teid viimaste sammude ja järgmiste tegevuste läbiviimisel.

## Töötlemise lõpetamise märgid

Kui töötlemine on edukalt lõppenud, näete mitmeid märgid:

* ✅ **Edusammude riba**: jõuab 100% täitumiseni
* ✅ **Veakontrolli logi**: näitab viimaseid `[RUN-SUMMARY]` ridu koos arvudega (pildid, kaamerarühmad, sihtmärgid, kalibreeritud pildid, salvestatud failid)
* ✅ **Käivitusnupp**: muutub uuesti aktiveerituks (valmis järgmiseks töötlemisvooruks)
* ✅ **Väljundfailid**: kõik töödeldud pildid on salvestatud projekti väljundkataloogi (allpool)

{% hint style="warning" %}
**Töötlusvoor, mille käigus pilte ei salvestata, loetakse ebaõnnestunuks.** Kui taotlesite pilditooteid ja töötlusvoog ei salvestanud ühtegi, märgib Chloros seda ebaõnnestumisena — logi nimes sisalduv viide `[RUN-SUMMARY]` osutab tõenäolisele põhjusele (midagi ei imporditud, sihtmärki ei tuvastatud või kõik taotletud tooted jäeti kõrvale kui mittekohaldatavad). Vastav CLI-käsk lõpetab töötamise nullist erineva tulemusega. Tahtlikult ainult metandmetele suunatud käivitus (kõik eksporditooted välja lülitatud, indeksid puuduvad) loetakse siiski edukaks. Vaata [CLI-viidet](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Töödeldud piltide leidmine

### Väljundkausta avamine

1. Klõpsake **peamenüü** ikooni <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(ülemine vasak nurk)
2. Valige **„Ava projekti kaust”**

3. Failibrauser avab projekti kataloogi
4. Leidke oma projekt nime järgi

### Väljundpuu

Tooted salvestatakse **projekti kausta alla, rühmitatuna kaamera ja seejärel failivormingu järgi**:

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

* **Kaamera kaust**: `LATT-<sensor>-<lens>-F<filter>` kaamera LATTICE jaoks (vastab pildi EXIF-andmetele `Model`), `<model>_<filter>` kaamera Survey3 puhul (nt `Survey3N_RGN`). Kaks kaamerat, millel on ühine sensor ja filter, kuid erinev objektiiv, hoitakse eraldi puudes — vinjet, vaateväli ja moonutus erinevad.
* **Formaatikausta**: järgib teie ekspordiformaadi seadet — `tiff16`, `tiff8`, `png8`, `jpg8`, või `tiff32` TIFF-i puhul (32-bitine, protsent). Kiirgus on alati float32 ja asub alati kaustas `tiff32`.
* **Tootekaustad**:
  * `Reflectance_Calibrated_Images/` — kalibreeritud peegeldusvõime
  * `Debayered_Images/` — lineaarselt debayereeritud (LATTICE): tumedusnihe eemaldatud ja demosaicitud, ilma tasandusvälja või segunemise eemaldamiseta, anduri algses orientatsioonis
  * `Preview_Images/` — ekraanil eelvaade (LATTICE)
  * `Radiance_Images/` — float32 spektraalne kiirgusvõimsus, W/m²/sr/nm (LATTICE multispektraalne)
  * `Vignette_Corrected_Images/` **või** `Sensor_Response_Images/` — kalibreerimata varuvariant kaadritele, millel puudub peegeldusvõime viide; iga käivituse kohta on olemas täpselt üks neist kahest, mille valib Vignette’i korrigeerimise seade
  * `<INDEX>_Index_Images/` — üks kaust iga valitud indeksi kohta (nt `NDVI_Index_Images`)
* **`clamp_masks/`** (uus versioonis 1.3.0, ainult LATTICE multispektraalsed M3C-kaamerad): ilmub formaadikausta tootekastide kõrvale, kui lainepikkuse segunemise eemaldamine kärbib negatiivsed väärtused nullini kiirguse või peegeldusvõime tootes. See sisaldab ühte pikselipõhist maski iga mõjutatud toote kohta, nt `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Maske ei ole kunagi tootekausis, seega saab tootekausi edastada fotogramm-meetria tarkvarale sellisena, nagu see on. Vaata [Kinnitusmaskid](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Iga eksporditud toode säilitab ALGFAILI nime.**`capture_..._raw.tif` kiirguse eksport kannab endiselt nime `capture_..._raw.tif` — see asub lihtsalt kataloogis `tiff32/Radiance_Images/`.**Toodet identifitseerib kaust, mitte failinimi**, seega `*radiance*.tif` otsimine ei anna tulemusi; otsige selle asemel kausta järgi.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Kui palju faile peaks olema?

Ärge arvutage valemi järgi — väljundite arv sõltub sellest, millised tooted olid aktiveeritud ja millised kehtivad iga kaamera puhul (nt RGB-kaamerad ei saa kiirgust/peegeldust). Õige arv on kirjas logis: viimane rida `[RUN-SUMMARY]` näitab täpselt, kui palju faile kirjutati, ja selgitavad read kirjeldavad kõike, mis vahele jäi.

***

## Töödeldud piltide vaatamine

### Kiire eelvaade failiotsijas

**Sisseehitatud eelvaade Windowsis:**

1. Navigeerige tootekausta (nt `tiff16/Reflectance_Calibrated_Images/`)
2. Valige pildifail
3. Eelvaade ilmub Windows i Explorer'i eelvaatepaneelis
4. Kasutage nooleklahve piltide sirvimiseks

### Eelvaade välistes pildivaatajates

**Soovitatavad vaatajad:*** **QGIS** – tasuta GIS-tarkvara (parim georefereeritud multispektraalanalüüsiks)
* **IrfanView** – kiire ja kerge pildivaataja (toetab TIFF-vormingut)
* **Adobe Photoshop** – professionaalne pilditöötlus (TIFF-tugi)
* **GIMP** – tasuta alternatiiv Photoshopile
* **Windows Photos** – põhiline vaatamine (võib-olla ei toeta 16-bitist TIFF-vormingut)

### Eelvaade Chloros i pildivaatajas

Kasutage Chloros i sisseehitatud pildivaatajat täiustatud visualiseerimiseks:

1. Klõpsake failibrauseris pildi pisipildile
2. Pilt avaneb peamises eelvaatealas
3. Klõpsake vasakul külgribal vahekaardile **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">

4. Kasutage interaktiivseks analüüsiks [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md)

Üksikasjalikud juhised leiate [Image Viewerist](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Peegeldusvõime pikselväärtuste lugemine (GIS / Pix4D / skriptid)

Peegeldusvõime salvestatakse täisarvuna DN ning **DN, mis tähendab ρ = 1,0, sõltub allikakaamerast**:

| Allikas          | ρ = 1,0 on | Kuidas kindlaks teha                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (varu kuni ρ 2,0) | Failile on lisatud XMP-märge `Chloros:PixelScale=32768` |
| Survey3          | **65535** (piiratud väärtusega ρ 1,0)     | `Chloros:PixelScale`-märget ei ole — selle puudumine ongi märk (Survey3 i tooted kannavad küll teisi `Chloros:*`-töötlemismärke) |

**Loe `Chloros:PixelScale`-märget ja jaga sellega**, selle asemel et eeldada ühtset väärtust 65535 — LATTICE-peegeldusvõime jagamine 65535-ga vähendab vaikimisi iga väärtust poole võrra. Enne versiooni 1.3.0 loodud 8-bitisena salvestatud peegeldusfail on kärbitud, mitte ümberskaalatud, ja sellel puudub skaalamärge: ekspordige see uuesti 16-bitisena või 32-bitisena, selle asemel et seda jagada. Täieliku ülevaate leiate jaotisest [Väljundpildi formaadid](../output-image-formats.md).

***

## Eksportimisel kaasa kantavad metaandmed

Iga toode säilitab allika pildistamise **GPS-ploki**ja selle**EXIF-alam-IFD**, seega
eksporditakse kaasa `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` ja
`CameraSerialNumber` ning georeferentsi.

{% hint style="warning" %}
**Kui ortomosaiik kuvatakse absurdses mõõtkavas, kontrollige esmalt `FocalLength`-i.**
Pix4D arvutab maapinna proovivahe kauguse fokaalkauguse ja kõrguse põhjal. Ilma selle märgistuseta
kasutatakse äärmiselt valet mõõtkava – ühel 49 pildiga mõõdetud lennul rekonstrueeriti 411 m × 160 m
apelsinipuuaed rekonstrueeriti 47,8 km × 13 km suuruseks, mille tulemusena tekkis 455-megapiksline ortomosaik, mis koosnes peamiselt
tühjast ruumist. Aeglane piltide kokkupanek ja ootamatult suur fail on selle sümptomid, mitte eraldi
probleemid.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Mitte *kõiki* sildid ei kopeerita. IFD0 struktuurilised sildid jäetakse tahtlikult välja (nende kopeerimine
rikub LATTICE väljundit) ning `ExifImageWidth` / `ExifImageHeight` jäetakse välja,
kuna need kirjeldavad algset salvestust – muidu nõuaks suurust muudetud eksport
mõõtmeid, mis on vastuolus selle enda rasteriga.

Chloros kirjutab ka oma `Chloros:*` XMP-märgised, mis kirjeldavad, mida iga tootega tehti
(näiteks `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). Alates versioonist 1.3.0 sisaldavad Survey3 tooted neid igas töötlemisrežiimis
režiimis; indeksitooted kirjeldavad indeksit ennast (`Chloros:ProcessingLevel=index`,
valemit ja seda, kas see arvutati peegeldusvõime või kiirguse põhjal), selle asemel et korrata
allika toorandmete sildid, ning ka kaustades, mille nimi on pikem kui 260 tähemärki, säilitatakse metaandmed
samuti. Vaata [Väljundpildi formaadid](../output-image-formats.md#index-products).

***

## Veakiri läbivaatamine

### Hoiatuste ja vigade kontrollimine

1. Avage vahekaart **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Sirvige sõnumeid
3. Otsige kollaseid hoiatusi või punaseid vigu
4. Lugege ridu `[RUN-SUMMARY]` ja kõiki vihjeid
5. Abisaamiseks võtke ühendust MAPIR i tugiteenistusega

### Logi salvestamine

Töötlemise dokumenteerimiseks või MAPIR i tugiteenistusele saatmiseks:

1. Klõpsa nuppu **„Kopeeri”**või**„Laadi alla”**

2. Salvesta tekstifailina projekti kausta
3. Lisa projekti dokumentatsiooni
4. Saada MAPIR i tugiteenistusele, kui tekivad probleemid

Probleemi otse MAPIRile saatmiseks kasuta veakuvandit (**Teata veast**) vahekaardi „Debug Log“ ülaosas — vaata [Veast teatamine vahekaardilt „Log“](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Tüüpilised väljundiga seotud probleemid ja lahendused

### Probleem: puuduvad väljundfailid

**Võimalikud põhjused:**

* Toode ei sobi selle kaameraga (nt kiirgus/peegeldusvõime RGB-kaamerate puhul — seda näitab logi)
* Puudus vajalik viide (nt peegeldusvõime ilma sihtmärgi ja `.daq` allapoole suunatud kiirguseta)
* Toote ekspordi valikukast oli projekti seadetes keelatud
* Ekspordi ajal sai kettaruum otsa

**Lahendused:**

1. Kontrollige vealogis ridu `[RUN-SUMMARY]` ja `[EXPORT-CHECK]` – need selgitavad kaamerapõhiseid vahelejätmisi
2. Kontrollige eksporditoote valikukastide olemasolu [Projekti seadetes](adjusting-project-settings.md)
3. Kontrollige, kas kettaruum oli piisav
4. Korrake töötlemist pärast põhjuse kõrvaldamist

### Probleem: tumedad või heledad servad (vignettimine on endiselt nähtav)

**Võimalikud põhjused:**

* Vignettimise korrigeerimine on välja lülitatud
* Kaamera/objektiiv puudub Chloros i profiilide andmebaasist
* Äärmuslik vignettimine, mida ei ole võimalik korrigeerida

**Lahendused:**

1. Veenduge, et projektiseadetes on vignettimise korrigeerimine sisse lülitatud
2. Kontrollige, kas kaameramudel on õigesti tuvastatud
3. Kui vignettimine püsib, võtke ühendust MAPIR i tugiteenistusega

### Probleem: valed värvid või väärtused

**Võimalikud põhjused:**

* Kalibreerimismärke ei tuvastatud
* Valitud on vale kalibreerimismärgi mudel
* Peegelduskalibreerimine on välja lülitatud
* Kalibreerimismärkide pildid on halva kvaliteediga

**Lahendused:**

1. Veendu, et peegelduskalibreerimine on sisse lülitatud
2. Kontrollige vealokis sõnumeid „Sihtmärk leitud“
3. Kontrollige sihtmärgi pildi kvaliteeti
4. Töötlege uuesti, märgistades õiged sihtmärgid

### Probleem: NDVI-väärtused tunduvad valed

**Oodatavad NDVI-vahemikud:*** **Vesi, kivid, muld**: -0,1 kuni 0,2
* **Hõre/ebaterve taimestik**: 0,2 kuni 0,4
* **Mõõdukas taimestik**: 0,4 kuni 0,6
* **Terve, tihe taimestik**: 0,6 kuni 0,9**Kui väärtused jäävad väljapoole neid vahemikke:**

1. Veendu, et peegelduskalibreerimine on rakendatud
2. Veendu, et valgusanduri logi on lisatud
3. Kontrolli, kas kalibreerimise sihtmärgid on tuvastatud
4. Veendu, et on tuvastatud õige kaameramudel
5. Vaadake üle sihtmärkide pildistamise ajastus ja tingimused
6. Kui arvutate indekseid ise peegeldusfailide põhjal, veenduge, et jagasite tulemuse faili `Chloros:PixelScale`-iga (vt eespool)

***

## Töödeldud piltide kasutamine

### Fotogramm-meetria / ortomoosaikide loomiseks

**Soovitatav töövoog:**

1.**Importige kalibreeritud peegelduspilte** fotogramm-meetria tarkvarasse:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Säilita EXIF-metadata**: veendu, et geotagimiseks säiliksid GPS-andmed
3. **Kalibreeritud töövood**: kasuta teadusliku täpsuse tagamiseks peegelduspilte — LATTICE peegelduspildid sisaldavad XMP-kalibreerimismärke, mida Pix4D loeb
4. **Töötle indeksmosaiike**: loo NDVI-ortomosaiike üksikutest indeksipiltidest
5. **Ekspordi georefereeritud GeoTIFF**: kasutamiseks GIS-rakendustes

### GIS-analüüsi jaoks

**Soovitatav töövoog:**

1.**Laadi QGIS-i, ArcGIS-i või sarnasse programmi**

2.**Kasutage 16-bitiseid TIFF** peegelduspilte mitme lainepikkuse analüüsiks (jagage faili `Chloros:PixelScale`-iga)
3. **Kasutage indeksipilte** (NDVI, NDRE) valmis taimestikukihtidena
4. **Rasterarvuti**: kombineeri laineid kohandatud analüüsi jaoks
5. **Eksport**: loo klassifitseerimiskaardid, muutuste tuvastamine, taimestiku tervisekaardid

### Otseseks analüüsiks / aruandluseks

**Soovitatav töövoog:**

1.**Kasuta indeksipilte LUT-värvidega** visuaalsete aruannete jaoks
2. **Statistika väljavõtmine**: keskmine NDVI väli/katseala kohta
3. **Aegrida**: indeksite võrdlemine mitme seansi vahel
4. **Aruannete koostamine**: lisage kaardid, statistika ja visualiseeringud***

## Arhiveerimine ja varundamine

### Soovitatav varundamisstrateegia

**Mida salvestada:*** ✅ **Algupärased RAW/JPG-pildid või LATTICE-i toorpildid** – arhiveerige eraldi kettale/pilve; toorpildid on töötlemisprotsessi allikas ja kõik muu saab neist uuesti genereerida
* ✅ **`.daq` / `.csv` valgusanduri failid** – vajalikud peegeldusvõime hilisemaks uuesti arvutamiseks
* ✅ **Töödeldud väljundid** – säilita kalibreeritud pildid ja indeksid
* ✅ **Projekti kaust** (`project.json` ja sellega seotud failid) – Sisaldab kõiki seadeid vajadusel uuesti töötlemiseks
* ✅ **Veakõrvalduslogi** – Dokumenteerib töötlemise üksikasju
* ✅ **Kalibreerimise sihtmärgipildid** – Kontrollimiseks ja uuesti töötlemiseks**Salvestamise soovitused:*** **Viivitamatu varukoopia**: väline kõvaketas
* **Pikaajaline arhiiv**: pilvesalvestus (Google Drive, Dropbox jne)
* **Kriitilised andmed**: hoidke 2–3 koopiat erinevates asukohtades***

## Järgmised töötlemistsüklid

### Projekti seadete taaskasutamine

Kui töötlete tulevikus sarnaseid andmekogumeid:

1. **Salvestage projekti mall** (kui seda pole veel tehtud)
2. **Looge uus projekt** salvestatud malli abil
3. **Importige uued pildid**

4.**Töötlege**ühtsuse tagamiseks samade seadete abil

### Mitme seansi kogumitöötlus

Mitme seansi/andmekogumi puhul:**Võimalus 1: graafiline kasutajaliides (GUI) – mitu projekti**

* Loo iga seansi jaoks eraldi projekt
* Kasuta ühtseid malli seadeid
* Töötle ükshaaval

**Võimalus 2: Chloros CLI (ainult Chloros+)**

* Automatiseeri hulgitöötlus
* Töötle mitut kausta skriptide abil
* Vaata [CLI dokumentatsiooni](../CLI.md) ja [CLI viiteid](../reference/cli-reference.md)

**Võimalus 3: Python SDK (ainult Chloros+)**

* Programmeeritud juhtimine
* Integreerimine analüüsipipeliinidega
* Vaata [API-dokumentatsiooni](../api-python-sdk.md) ja [SDK-viiteid](../reference/sdk-reference.md)

***

## Järelkäsitluse veaotsing

### Uuesti töötlemine erinevate seadete abil

Kui tulemused ei ole rahuldavad:

1. Säilita originaalpildid (ära kunagi kustuta)
2. Ava sama projekt Chloros es 3. Muuda seadeid projekti seadete paneelis
4. Töötle uuesti — väljundid salvestatakse samadesse tootekausta, seega asendatakse eelmise töötlemiskorra samanimelised failid

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
* 📖 **Dokumentatsioon**: [Chloros i kasutusjuhend](../)

***

## Kokkuvõte: täielik töövoog

Olete nüüd läbinud kogu Chloros i töötlemise töövoo:

1. ✅ **Projekt loodud** – vaadake [Projekte](../projects.md)
2. ✅ **Lisasid faile** – vaata [Failide lisamine](adding-files-to-a-project.md)
3. ✅ **Seadistused kohandatud** – vt [Projekti seadistuste kohandamine](adjusting-project-settings.md)
4. ✅ **Sihtmärgid märgitud** – vt [Sihtmärgipiltide valimine](choosing-target-images.md)
5. ✅ **Töötlemine alustatud** – vt [Töötlemise alustamine](starting-the-processing.md)
6. ✅ **Töötlemise kulgemise jälgimine** – vt [Töötlemise jälgimine](monitoring-the-processing.md)
7. ✅ **Tulemuste läbivaatamine** – See lehekülg**Teie kalibreeritud ja peegeldusega korrigeeritud multispektraalsed pildid on valmis analüüsimiseks!**

***

## Lisamaterjalid

### Täiustatud funktsioonid

* [**Pildivaataja**](../image-viewer-gui/opening-an-image-full-screen.md) – interaktiivne visualiseerimine ja analüüs
* [**Indeksi/LUT-katsetuskeskkond**](../image-viewer-gui/index-lut-sandbox.md) – kohandatud indeksite testimine
* [**Multispektraalsed indeksivalemid**](../project-settings/multispectral-index-formulas.md) – täielik indeksite viitekogu

### Automatiseerimine ja integreerimine

* [**Käsurea (CLI) dokumentatsioon**](../CLI.md) – käsurea kaudu toimuv paketttöötlus
* [**Python SDK**](../api-python-sdk.md) – programmeeritud automatiseerimine
* [**„Chloros+“ funktsioonid**](../#chloros) – Täiustatud töötlemisvõimalused

### Tugi ja õppimine

* [**KKK**](../faq.md) – Vastused levinumatele küsimustele
* [**Kalibreerimise sihtmärgid**](../calibration-targets.md) – peegeldusvõime kalibreerimise põhimõtted
* [**Toetatud kaamerad**](../supported-cameras.md) – ühilduv riistvara
