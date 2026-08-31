# Projekti seaded

Chloros-i külgribal asuv „Projekti seaded“ (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

) võimaldab teil konfigureerida kõiki pilditöötluse, kalibreerimismärgi tuvastamise, multispektraalsete indeksite arvutamise ja projekti ekspordivõimaluste aspekte. Need seaded salvestatakse koos projektiga ning neid saab salvestada mallidena, et neid mitmes projektis korduvalt kasutada.

## Projektiseadete avamine

Projektiseadete avamiseks:

1. Avage projekt Chloros-is
2. Klõpsake vasakul külgribal asuval vahekaardil „<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

” nuppu **Project Settings**

3. Seadete paneelis kuvatakse kõik kättesaadavad konfiguratsiooni valikud kategooriate kaupa

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Seaded, mis sõltuvad teistest seadetest, on halliks muudetud.** Kui ülemine valik muudab mõne seade valimise võimatuks (näiteks kui tühistada valik *Peegelduskalibreerimine / valgebalanss*, muutub *Peegelduse eksport* võimatuks), on sõltuv valik keelatud ja selle abiteates on nimetatud valik, mida tuleb muuta.
{% endhint %}

***

## Ekraan

### Pildi pisipildi resolutsioon

* **Tüüp**: rippmenüü
* **Valikud**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Vaikimisi**: Vaikimisi (512 px)
* **Kirjeldus**: Eraldusvõime (pikim serv, pikslites), millega pildiresti pisipildid renderdatakse. Suuremad väärtused näevad suurendamisel teravamad välja, kuid laadivad aeglasemalt ja kasutavad rohkem mälu. Täisresolutsioon vastab originaalpildi suurusele.
* **Märkus**: Ainult kuvamiseks — see ei mõjuta kunagi töötlemist ega eksporditud faile.***

## Sihtmärkide tuvastamine

Need seaded määravad, kuidas Chloros tuvastab ja töötleb kalibreerimissihte teie piltidel. Mõlemad on aktiivsed ainult siis, kui **Peegelduskalibreerimine / valgebalanss** on sisse lülitatud (muidu on need hallid, kuna sihtmärgi tuvastamine jäetakse täielikult vahele).

### Minimaalne kalibreerimisproovi pindala (px)

* **Tüüp**: Number
* **Vahemik**: 0 kuni 10 000 pikslit
* **Vaikimisi**: 25 pikslit
* **Kirjeldus**: Määrab minimaalse pindala (pikslites), mis on vajalik, et tuvastatud piirkonda loetaks kehtivaks kalibreerimismärgi prooviks. Väiksemad väärtused tuvastavad väiksemaid sihtmärke, kuid võivad suurendada valetuvastuste arvu. Suuremad väärtused nõuavad tuvastamiseks suuremaid ja selgemaid sihtmärgi piirkondi.
* **Millal kohandada**:
  * Suurendage, kui saate valetuvastusi väikeste pildiartefaktide puhul
  * Vähendage, kui teie kalibreerimissihmärgid paistavad piltidel väikesed ja neid ei tuvastata

### Sihtmärkide minimaalne klastrite moodustamine (0–100)

* **Tüüp**: Number
* **Vahemik**: 0 kuni 100
* **Vaikimisi**: 60
* **Kirjeldus**: Reguleerib klastrite moodustamise künnist sarnaste värviga piirkondade rühmitamiseks kalibreerimissihte tuvastamisel. Kõrgemad väärtused nõuavad, et rohkem sarnaseid värve rühmitataks kokku, mille tulemuseks on konservatiivsem sihtmärkide tuvastamine. Madalamad väärtused lubavad sihtmärkide rühmas rohkem värvivaheldusi.
* **Millal reguleerida**:
  * Suurendage, kui kalibreerimissihmärke jagatakse mitmeks tuvastuseks
  * Vähendage, kui värvivaheldustega kalibreerimissihmärke ei tuvastata täielikult

***

## Töötlemine

Need seaded määravad, kuidas Chloros teie pilte töötleb ja kalibreerib.

### Vignettkorrektsioon

* **Tüüp**: Valikukast
* **Vaikimisi**: Lülitatud sisse (märgitud)
* **Kirjeldus**: Rakendab vinjeti korrigeerimist, et kompenseerida objektiivi poolt põhjustatud pildi servade tumenemist. Vinjetatsioon on levinud optiline nähtus, mille puhul pildi nurgad ja servad tunduvad objektiivi omaduste tõttu keskosast tumedamad.
* **Kõrvalmõju**: See lüliti valib ka selle, millist *kalibreerimata varutoodet* jooks kirjutab (vt allpool).

### Peegelduskalibreerimine / valgebalanss

* **Tüüp**: Valikukast
* **Vaikimisi**: Lülitatud sisse (märgitud)
* **Kirjeldus**: Lülitab sisse peegelduskalibreerimise — vastavalt kaamerale ja kättesaadavusele kas raamis tuvastatud kalibreerimismärkide ja/või DAQ valgussensori allapoole suunatud andmete põhjal. See normaliseerib peegeldusväärtused kogu andmekogus ja tagab ühtlased mõõtmistulemused olenemata valgustingimustest.
* **Kui lülitatud välja**: Sihtmärkide tuvastamine jäetakse täielikult vahele ning**ükski kaamera ei saa luua ühtegi peegeldusvõime toodet** — nii Survey3 sihtmärgipõhised kui ka LATTICE DAQ-põhised. Sõltuvad seaded (*Ekspordi peegeldusvõime*, *Minimaalne taaskalibreerimise intervall* ja sihtmärgi tuvastamise künnised) on halliks muudetud.

### Kalibreerimata varutooted: Ekspordi anduri vastus / Ekspordi vinjeti korrigeeritud

* **Tüüp**: Kaks märkeruutu
* **Vaikimisi**: Mõlemad on aktiveeritud (märgitud)
* **Kirjeldus**: Kui kaadri peegeldusvõimet ei ole võimalik kalibreerida (kalibreerimissihte ei leitud või peegeldusvõime kalibreerimine on välja lülitatud), salvestatakse see selle asemel *kalibreerimata varutoodanguna*. **Iga kaameramudeli puhul on ühe töötluse kohta olemas täpselt üks kahest varutoodangust**, mille valib lüliti *Vignettkorrektsioon*:
  * Vignettkorrektsioon **sisse lülitatud**→ `Vignette_Corrected_Images/` (määrab**Ekspordi vinjetikorrektsiooniga**)
  * Vinjetikorrektsioon **väljas**→ `Sensor_Response_Images/` (määrab**Ekspordi anduri tundlikkusega**)
* Kasutusel mitte olev varuvariant on halliks muudetud. Kasutusel oleva valiku tühistamine takistab selle faili salvestamist üldse.

### LATTICE-eksporttooted

LATTICE-kaadritega projektide puhul jaguneb iga imporditud LATTICE-kaader ühe töötlemisvooru jooksul kõigile aktiveeritud **ja kohaldatavatele**toodetele. Jagunemist juhivad neli märkeruutu (kõik on vaikimisi**sisse lülitatud**):

| Seade | Väljundkaust | Mida eksporditakse |
| --- | --- | --- |
| **Ekspordi debayered** | `Debayered_Images/` | Lineaarne debayered-pilt. Kehtib kaamerate RGB ja multispektraalkaamerate puhul. |
| **Eelvaate eksport** | `Preview_Images/` | Ekraanil kuvatav eelvaade. RGB = valgebalanss (DAQ-valgusallikas, kui see on saadaval, muidu hallmaailm) + gamma; multispektraalne = valevärvide venitus. |
| **Ekspordi kiirgus** | `Radiance_Images/` | Float32 spektraalne kiirgus ühikutes W/m²/sr/nm. Ainult multispektraalne (M3C/M3M) — ei kehti RGB-i originaalidele. Kirjutatakse alati 32-bitise TIFF-ina, sõltumata *Kalibreeritud pildiformaadi* seadistusest. |
| **Ekspordi peegeldusvõime**| `Reflectance_Calibrated_Images/` | Uint16 peegeldusvõime, skaalitud nii, et**32768 = peegeldusvõime 1,0** (märgitud XMP-na `Chloros:PixelScale`). Ainult multispektraalne, kirjutatakse siis, kui vastav `.daq` allapoole suunatud kirje (või kvaliteedikontrolli läbinud kaadris olev sihtmärk) katab kaadri. |

* RGB põhikaamerad edastavad debayered + eelvaate; nende puhul jäetakse kiirgus/peegeldusvahe välja, kuna need ei ole kohaldatavad.
* Debayereeritud/eelvaate bittisügavus järgib seadet *Kalibreeritud pildiformaat*; kiirgus on alati float32.
* Nimetatud neli lülitit ei mõjuta Survey3 töötlemist.

Samad neli lülitit on olemas ka ilma pealkirjata kui `chloros-cli process --debayered / --preview / --radiance / --reflectance` ja SDK vastavate parameetritena. Need asendasid vana `--radiometric-output` lipu, mida enam ei eksisteeri.

{% hint style="warning" %}
**Kõigi asjakohaste toodete väljalülitamine põhjustab töötlemise ebaõnnestumise.** Alates versioonist 1.2.0 teatab töötlemisjooks, millelt küsiti tooteid, kuid mis ei kirjutanud ühtegi pilditoodet, veast ning CLI lõpetab töötamise nullist erineva tulemusega, selle asemel et teatada vaikimisi õnnestumisest. Logis nimetatakse toodet, mida ei suudetud kirjutada, ja põhjust. Tahtlikult ainult metandmetega-ainult käivitus (midagi ei taotletud) on endiselt edukas.
{% endhint %}

### Peegeldusallikas (projekti seade, määratakse CLI/SDK kaudu)

Projekt salvestab ka selle, millist **peegeldusviidet** LATTICE peegeldustoode kasutab. Seadete paneelis puudub selleks eraldi juhtelement; väärtus salvestatakse projekti konfiguratsioonis kui `Processing → "Target reflectance source"` ja see määratakse parameetriga `chloros-cli process --reflectance-source {auto,target,daq}` või parameetriga SDK&#x27;`reflectance_source`:

* **`auto`** (vaikimisi): kvaliteedikontrolli läbinud kaadrisisesest kalibreerimise sihtmärkist saab absoluutne etalon, mille puudumisel või kvaliteedikontrolli ebaõnnestumisel kasutatakse DAQ-i allapoole suunatud valguse jagunemist (ρ = πL/E).
* **`target`**: range sihtmärgipõhine peegeldusvõime — DAQ-i asendamist ei toimu.
* **`daq`**: DAQ-i poolt määratud peegeldusvõime; raamis olevaid sihtmärke ei kasutata võrdlusalusena.

Salvestatud väärtusega võrreldakse suurtähti arvestamata ning mõned kirjapildid aktsepteeritakse sünonüümidena: `target`, `target_image`, `empirical` ja `empirical_line` tähendavad kõik **sihtmärki**; `daq`, `dls`, `light_sensor` ja `sensor` tähendavad kõik**daq**. Kõik muu — sealhulgas puuduv võti — tõlgendatakse kui**auto**.

Ühiku kaupa **mõõdetud** sihtmärgi skaneeringuid otsitakse sihtmärgi seerianumbri/QR-koodi järgi, näiteks `<serial>.csv`, kolmest kohast: kataloogist, mis on määratud `--target-reflectance-dir` (salvestatud kui `Processing → "Target reflectance dir"`), projekti enda kaustas `target_reflectance/` ning keskkonnamuutuja `CHLOROS_TARGET_REFLECTANCE_DIR` teekonnas. Kui selle üksuse jaoks pole mõõdetud skaneeringut, kasutatakse selle asemel sihtmudeli avaldatud nominaalkõverat.

### Debayeri meetod

* **Tüüp**: rippmenüü
* **Valikud**:
  * Standard (kiire, keskmine kvaliteet)
  * Tekstuuritundlik (aeglane, kõrgeim kvaliteet) \[Chloros+]
* **Vaikimisi**: Standard (kiire, keskmine kvaliteet)
* **Kirjeldus**: Valib demosaicing-algoritmi, mida kasutatakse toores Bayer-mustriga andurite andmete teisendamiseks täisvärvilisteks piltideks. Meetod „Standard (kiire, keskmine kvaliteet)” tagab optimaalse tasakaalu töötlemiskiiruse ja pildikvaliteedi vahel. Meetod „Tekstuuritundlik (aeglane, kõrgeim kvaliteet)” \[Chloros+] kasutab kõrgekvaliteediga servatundlikku demosaicingut kombineerituna AI/ML müra eemaldamise mudeliga, mis eemaldab peaaegu kogu demosaicingust tuleneva müra. Tekstuuritundliku mudeli töötamiseks on vaja GPU mälu (VRAM). Soovitame seda kasutada, kui teil on kiirema töötlemise tagamiseks saadaval &gt;4 GB VRAM-i.
* **Kui rida on üldse rippmenüü**: kahe valikuvõimalusega rippmenüü ilmub ainult siis, kui**mõlemad**tingimused on täidetud — olete sisse logitud sobiva Chloros+ tellimusega,**ja** projekt ei sisalda ühtegi LATTICE-kaadrit. Muidu kuvatakse rida tavalise tekstina „`Standard (Fast, Medium Quality)`“, ilma et oleks midagi valida.
* **LATTICE-märkus**: LATTICE-le treenitud tekstuuritundlikku mudelit ei ole olemas ning töötlemisahel sunnib LATTICE-kaadrite puhul kasutama standardset demosaiki, sõltumata salvestatud väärtusest. Kui lisate LATTICE-kausta projekti, kus oli juba valitud „Texture Aware“, kirjutab Chloros seadistuse tagasi „Standard“-iks, selle asemel et jätta `project.json`-is aegunud väärtus.

### Minimaalne kalibreerimise intervall

* **Tüüp**: Number
* **Vahemik**: 0 kuni 3 600 sekundit
* **Vaikimisi**: 0 sekundit
* **Kirjeldus**: Määrab kalibreerimismärkide kasutamise vahelise minimaalse ajavahemiku (sekundites). Kui väärtuseks on seatud 0, kasutab Chloros iga tuvastatud kalibreerimissihikut. Kui väärtuseks on seatud suurem number, kasutab Chloros ainult neid kalibreerimissihikuid, mille vaheline aeg on vähemalt nii palju sekundeid, vähendades sellega andmekogumite töötlemisaega, kus kalibreerimissihikuid salvestatakse sageli.
* **Millal reguleerida**:
  * Seadke väärtuseks 0, et saavutada maksimaalne kalibreerimistäpsus muutuvates valgustingimustes
  * Suurendage väärtust (nt 60–300 sekundini), et kiirendada töötlemist, kui valgustingimused on stabiilsed ja kalibreerimismärkide pilte salvestatakse sageli

### Valgusanduri ajavööndi nihe

* **Tüüp**: Number
* **Vahemik**: -12 kuni +12 tundi
* **Vaikimisi**: 0 tundi
* **Kirjeldus**: Määrab valgussensori andmete ajamärkide ajavööndi nihke (tundides UTC-st), mida kasutatakse valgussensori logide sobitamisel pildistamise aegadega. Uuemad `.daq` salvestused sisaldavad oma ajavööndi päritolu, seega on see vajalik peamiselt vanemate, kohaliku aja järgi salvestatud logide puhul.

### PPK-korrektsioonide rakendamine

* **Tüüp**: Valikukast
* **Vaikimisi**: Väljalülitatud (valikukast märkimata)
* **Kirjeldus**: Võimaldab kasutada GPS-i (GNSS) sisaldavatest MAPIR DAQ-salvestitest saadud PPK-korrektsioone (Post-Processed Kinematic). Kui see on lubatud, kasutab Chloros teie projektikataloogis olevaid .daq-logifaile, mis sisaldavad särituspin-andmeid, ning rakendab teie piltidele täpseid geolokatsiooni parandusi.
* **Nõue**: Teie projektikataloogis peab olema .daq-logifail, mis sisaldab valgustuse pin-andmeid
* **Millal lubada**: Soovitatav on PPK-korrektsioon alati lubada, kui teie .daq-logifailis on valgustuse tagasiside andmed.

### Valgustuse pin 1

* **Tüüp**: Rippmenüü
* **Nähtavus**: Nähtav ainult siis, kui valik „Kohalda PPK-korrektsioone” on aktiveeritud JA ekspositsiooniandmed on saadaval viigu 1 jaoks
* **Valikud**:
  * Projektis tuvastatud kaameramudelite nimed
  * „Ära kasuta” – ignoreeri seda ekspositsiooniviiku
* **Vaikimisi**: Valitakse automaatselt projekti konfiguratsiooni alusel
* **Kirjeldus**: Määrab ekspositsioonipistikule 1 konkreetse kaamera PPK-aja sünkroniseerimiseks. Ekspositsioonipistik salvestab täpse ajahetke, mil kaamera katik vallandub, mis on täpse PPK-geolokatsiooni jaoks kriitilise tähtsusega.
* **Automaatvaliku toimimine**:
  * Üks kaamera + üks viik: valib kaamera automaatselt
  * Üks kaamera + kaks viiki: viik 1 määratakse kaamerale automaatselt
  * Mitu kaamerat: vajalik käsitsi valik

### Valgustuse viik 2

* **Tüüp**: Rippmenüü
* **Nähtavus**: Nähtav ainult siis, kui valik „Kohalda PPK-korrektsioone” on sisse lülitatud JA viigu 2 jaoks on saadaval eksponeerimisandmed
* **Valikud**:
  * Projektis tuvastatud kaameramudelite nimed
  * „Ära kasuta“ – ignoreeri seda ekspositsioonipinni
* **Vaikimisi**: valitakse automaatselt projekti konfiguratsiooni alusel
* **Kirjeldus**: määrab ekspositsioonipinnile 2 konkreetse kaamera PPK-ajasünkroniseerimiseks, kui kasutatakse kahe kaameraga seadistust.
* **Automaatvaliku toimimine**:
  * Üks kaamera + üks pin: pin 2 seatakse automaatselt seisundisse „Ära kasuta“
  * Üks kaamera + kaks viiku: viik 2 seatakse automaatselt seisundisse „Ära kasuta“
  * Mitu kaamerat: vajalik käsitsi valik
* **Märkus**: Sama kaamerat ei saa määrata korraga nii viigule 1 kui ka viigule 2.***

## DAQ valgusandur

See osa kuvatakse projekti seadetest ja loetleb kõik projekti DAQ allapoole suunatud failid — `.daq` salvestused ja DAQ-M `.csv` allapoole suunatud logid. Vahekaardil „Valgusandurid“ tehtud salvestused lisatakse avatud projektile automaatselt.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Igas reas on näidatud fail, anduri mudel ja selle faili puhul tegelikult kehtiv hajuti-korki korrektsioon. Ridade kohal asub üks projektipõhine juhtnupp:

### Korki ülekirjutamine (kõik failid)

* **Tüüp**: rippmenüü
* **Valikud**: `Auto` ning projektis olevate anduritüüpide jaoks kehtivad katekorrektsiooniprofiilid
* **Vaikimisi**: Auto
* **Salvestatud kui**: `Processing → "DAQ cap id"` (vaikimisi `auto`)
* **Kirjeldus**: `Auto` kasutab iga faili salvestatud kattekorrektsiooni (kui midagi pole salvestatud, eeldatakse päikesekatte kasutamist – kõik MAPIR andmekogumisüsteemid on varustatud päikesekorrektoriga). Konkreetse kattekorrektsiooni valimine tühistab**kõik** projekti allapoole suunatud faili: tooresid salvestusi korrigeeritakse sellega ja salvestusi, millel on juba piirang, viidatakse uuesti (salvestatud korrigeerimine tühistatakse ja rakendatakse valitud piirang).
* **Oluline**: Valitud piirang peab vastama piirangule, mis oli salvestamise ajal füüsiliselt paigaldatud. Ei andur ega tarkvara suuda füüsilist kate tuvastada – sobimatu kate ID korrigeerib spektreid valesti.

Siin on tahtlikult kasutatud **üht** kogu projekti hõlmavat seadistust, mitte failipõhiseid rippmenüüsid: seadistus mõjutab kõiki projekti allapoole suunatud allikaid.***

## Maatriksi joondamine

See osa kuvatakse **ainult** siis, kui vähemalt ühel projekti pildil on moodulitevaheline joondamistransformatsioon, mille LATTICE-massiivid salvestamise ajal lisavad (XMP-märgised `Chloros:Alignment*`). See näitab, mitmel pildil on joondamismärgised, milline kaamera on viitekaamera (`REF` märgis), ning kaamerate kaupa jaotatud piltide arvu tabeli.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Massiivi joondamise rakendamine

* **Tüüp**: Valikukast
* **Vaikimisi**: Lubatud (märgitud)
* **Salvestatakse kui**: `Processing → "Array alignment"`
* **Kirjeldus**: Muudab iga töödeldud toote (debayered / eelvaade / kiirgus / peegeldus / indeks) massiivi ühisesse viitegeomeetriasse, kasutades pildistamise ajal salvestatud teisendust. Väljalülitatud = eksportimine anduri alggeomeetrias.

### Lõika ühisele kattuvale alale

* **Tüüp**: Valikukast (aktiivne ainult siis, kui *Rakenda massiivi joondamine* on sisse lülitatud)
* **Vaikimisi**: Lubatud (märgitud)
* **Salvestatakse kui**: `Processing → "Array alignment crop"`
* **Kirjeldus**: Lõikab joondatud ekspordid piirkonnaks, mida kõik kaameramoodulid jagavad, nii et igal lainealal on sama pindala. Väljalülitamisel säilitatakse täielik anduripind (allika välispiirid täidetakse mustaga).

### Ümberdiskrimineerimine

* **Tüüp**: Rippmenüü (aktiivne ainult siis, kui valik *Kohalda maatriksi joondamine* on sisse lülitatud)
* **Valikud**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Vaikimisi**: Bilineaarne
* **Salvestatakse kui**: `Processing → "Array alignment interpolation"`
* **Kirjeldus**: Joondamise moonutuse poolt kasutatav interpolatsioon. „Lähim” säilitab täpsed allika väärtused (pikslitevahelist segamist ei toimu) rangeks radiomeetriliseks analüüsiks; „Bilineaarne“ sobib kõige paremini kaardistamiseks ja visuaalseks kasutamiseks.

Samad kolm valikut on olemas ka ilma eesliiteta kujul `chloros-cli process --array-alignment`, `--array-alignment-crop` ja `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Need seaded võimaldavad teil konfigureerida multispektraalseid indekseid analüüsi ja visualiseerimise jaoks.

### Lisa indeks

* **Tüüp**: Spetsiaalne indeksite konfigureerimispaneel
* **Kirjeldus**: Avab interaktiivse paneeli, kus saate valida ja konfigureerida multispektraalseid taimestiku indekseid (NDVI, NDRE, EVI jne), mida arvutada pilditöötluse käigus. Võite lisada mitu indeksit, millest igaühel on oma visualiseerimisseaded.
* **Saadaval olevad indeksid**: Graafilise kasutajaliidese rippmenüüs on**27** eelmääratud multispektraalset indeksivalemit (täieliku nimekirja leiate [Multispektraalsed indeksivalemid](multispectral-index-formulas.md), sealhulgas nimed, mida aktsepteerib ka valik CLI/SDK `--indices`).
* **Funktsioonid**:
  * Valige eeldefineeritud indeksivalemite hulgast
  * Lohistage oma kaamera filtri kanalid valemi lainepikkuste pesadesse
  * Konfigureerige visualiseerimise värvigradiente (LUT – Look-Up Tables)
  * Määrake künnisväärtused ja kärpimisrežiimid
  * Looge kohandatud indeksivalemid
* **Märkus**: Indekseid ei arvutata üheribal LATTICE M3M monokaamerate puhul – mitmeribalised indeksid on ühel ribal määratlemata. See ei mõjuta mudeleid Survey3 ja LATTICE M3C.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Iga lisatud indeks kuvab oma valemi matemaatilise valemina, kus iga ribapesa on tähistatud värvilise ringiga: punane = Red, roheline = Green, sinine = Blue, oranž = Orange, tsüaan = Cyan, lilla = NIR, magenta = RE. Lohistage ring valemi kohal olevast reast pesale, et see siduda; topeltklõpsake seotud pesal, et see tühistada. Indeks arvutatakse välja ainult siis, kui igas valemis kasutatud pesas on kanal.

### Kohandatud valemid (funktsioon Chloros+)

* **Tüüp**: Kohandatud valemite määratluste massiiv
* **Kättesaadavus**: Nõuab sisselogimist sobiva Chloros+ tellimusega.
* **Kirjeldus**: Võimaldab luua ja salvestada kohandatud multispektraalsete indeksite valemeid, kasutades ribade matemaatikat. Kohandatud valemid salvestatakse koos projekti seadetega ja neid saab kasutada täpselt samamoodi nagu sisseehitatud indekseid.
* **Kuidas luua**:
  1. Avage indeksite konfiguratsioonipaneelis kohandatud valemite kalkulaator
  2. Kirjutage valem, kasutades **spektririba-sümboleid**, mitte ribade nimesid
  3. Salvestage valem kirjeldava nimega – see ilmub seejärel valemite rippmenüü alaossa ja te saate oma kaamera kanaliringid selle pesadesse lohistada täpselt nagu sisseehitatud eelseadistuse puhul
* **Valemi süntaks**:
  * Sagedusribade pesad: `x`, `y`, `z`, `a`, `b`, `c` — kuus positsiooni, mille saad tõeliste kanalitega seostada lohistamise teel
  * Operaatorid: `+`, `-`, `*`, `/`, `^` ja `()` rühmitamiseks
  * Funktsioonid: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Miks sümbolid, mitte bändinimed**: valem, mis on kirjutatud kujul `(y-x)/(y+x)`, töötab igal kaameral, sest lohistamis- ja-lohistamise kaardistamine otsustab, kas `y` on RGN-filtri 850 nm NIR või OCN-filtri 808 nm NIR OCN filtri puhul. Sisseehitatud eelseadistused salvestatakse samamoodi – vaata [Multispektraalsed indeksivalemid](multispectral-index-formulas.md) , et näha kõigi 27 valemi täpset sümbolivormi.
* **Kus need toimivad**: kohandatud valemid salvestatakse koos projekti seadetega ja neid saab kasutada nii [indeksi/LUT-i katsetuskeskkonnas](../image-viewer-gui/index-lut-sandbox.md) kui ka töötlemisel. Neid**ei** CLI/SDK `--indices` nimekirjas, mis laiendab ainult 22 sisseehitatud eelseadistuse nime.***

## Eksport

Need seaded määravad eksporditavate töödeldud piltide formaadi ja kvaliteedi.

### Kalibreeritud pildi formaat

* **Tüüp**: rippmenüü
* **Valikud**:
  * **TIFF (16-bit)** – tihendamata 16-bitine TIFF-vorming
  * **TIFF (32-bitine, protsent)** – 32-bitine ujukomavormingus TIFF, kus peegeldusväärtused on protsentides
  * **PNG (8-bitine)** – pakitud 8-bitine PNG-vorming
  * **JPG (8-bitine)** – pakitud 8-bitine JPEG-vorming
* **Vaikimisi**: TIFF (16-bitine)
* **Kirjeldus**: Valib töödeldud ja kalibreeritud piltide salvestamise failivormingu. Eksporditud failid salvestatakse iga kaamera kausta sees asuvasse vormingupõhisesse alamkausta (`tiff16`, `tiff32`, `png8`, `jpg8`), kusjuures iga toote kohta on üks `<Product>_Images/`-kaust. Eksporditud failid säilitavad allika failinime – toodet identifitseerib kaust, mitte faililaiend.
* **Formaadisoovitused**:
  * **TIFF (16-bit)**: Soovitatav teaduslikuks analüüsiks ja professionaalseteks töövoogudeks. Säilitab maksimaalse andmekvaliteedi ilma tihendamisest tulenevate artefaktideta. Sobib kõige paremini multispektraalsele analüüsile ja edasisele töötlemisele GIS-tarkvaras.
  * **TIFF (32-bitine, protsent)**: Sobib kõige paremini töövoogudele, mis nõuavad peegeldusväärtusi protsentides (0–100%). Pakub maksimaalset täpsust radiomeetriliste mõõtmiste jaoks.
  * **PNG (8-bitine)**: Sobib veebis vaatamiseks ja üldiseks visualiseerimiseks. Väiksemad failisuurused kaotuseta tihendamisega, kuid vähenenud dünaamiline ulatus.
  * **JPG (8-bitine)**: Kõige väiksemad failisuurused, sobib ainult eelvaatamiseks ja veebis kuvamiseks. Kasutab kadudega pakkimist, mis ei sobi teaduslikuks analüüsiks.
* **Märkus**: LATTICE-kiirgusvõimsus eksporditakse alati 32-bitise ujukomana TIFF, sõltumata sellest seadistusest.***

## Projekti malli salvestamine

See funktsioon võimaldab teil salvestada oma praegused projekti seaded korduvkasutatava mallina.

* **Tüüp**: Teksti sisestusväli + Salvestamise nupp
* **Kirjeldus**: Sisestage oma seademallile kirjeldav nimi ja klõpsake salvestamise ikooni. Mall salvestab kõik teie praegused projekti seaded (sihtmärgi tuvastamine, töötlusvalikud, indeksid ja ekspordivorming), et neid tulevastes projektides hõlpsasti uuesti kasutada. Mallid salvestatakse teie projekti salvestuskausta sees asuvasse kausta `Project Templates/` ning neid saab valida või eksportida ka peamenüüst (*Vali mall* / *Salvesta mall* / *Ekspordi mall*).
* **Kasutusjuhtumid**:
  * Loo mallid erinevatele kaamerasüsteemidele (RGB, multispektraalne, NIR)
  * Salvesta standardseadistused konkreetsete põllukultuuride või analüüsitöövoogude jaoks
  * Jagage ühtseid seadeid kogu meeskonnaga
* **Kasutamine**:
  1. Määrake kõik soovitud projekti seaded
  2. Sisestage malli nimi (nt „RedEdge Survey3 NDVI Standard”)
  3. Klõpsake salvestusikoonile
  4. Mall on nüüd uute projektide loomisel laaditav

***

## Projekti kausta salvestamine

See seade määrab, kuhu uued projektid vaikimisi salvestatakse.

* **Tüüp**: Kataloogi tee kuvamine + nupp „Muuda“
* **Vaikimisi (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Vaikimisi (Linux)**: `~/Chloros Projects`
* **Kirjeldus**: Näitab praegust vaikimisi kataloogi, kuhu luuakse uued Chloros projektid. Klõpsake redigeerimisikoonil, et valida teine kataloog. Muudatus salvestatakse ühe tekstireana failis `~/.chloros/working_directory.txt` failis – Windows, st `C:\Users\<Username>\.chloros\working_directory.txt`. Kui see fail puudub või sisaldab teed, mida enam ei ole, kasutab Chloros eespool nimetatud vaikimisi kataloogi. CLI loeb ja kirjutab sama faili, seega on `chloros-cli` ja graafiline kasutajaliides alati ühel meelel selles, kus projektid asuvad.
* **Projektimallid** asuvad selle kataloogi alamkataloogis `Project Templates/`.
* **Millal muuta**:
  * Määrake võrgukettaks meeskonnatöö jaoks
  * Vahetage suurema salvestusruumiga kettaks suurte andmekogumite puhul
  * Korraldage projektid aasta, kliendi või projekti tüübi järgi erinevatesse kaustadesse
* **Märkus**: Selle seade muutmine mõjutab ainult UUSI projekte. Olemasolevad projektid jäävad oma algsetesse asukohtadesse.***

## Seadete säilitamine

Chloros projekt on **kaust**. Kõik projekti seaded salvestatakse selle sees asuvasse `project.json`-i; ühendatud riistvara salvestatakse koos sellega `cameras.json`-i ja `sensors.json`-i, mistõttu projekti uuesti avamisel ühendatakse ka selle kaamerad ja valgussensorid uuesti. Kui projekti uuesti avate, taastatakse kõik seaded täpselt sellisena, nagu te need jätsite. Salvestatud projekte saab juhtida ka ilma kasutajaliideseta, kasutades `chloros-cli project`-i või SDK-i `open_project`-i.

### Seadete hierarhia

Seaded rakenduvad järgmises järjekorras:

1. **Süsteemi vaikimisi seaded** – Chloros poolt määratud sisseehitatud vaikimisi seaded
2. **Malli seaded** – kui laadite projekti loomisel malli
3. **Salvestatud projekti seaded** – projektifailiga koos salvestatud seaded
4. **Käsitsi tehtud muudatused** – kõik muudatused, mida teete käesoleva seansi jooksul

### Seaded ja pilditöötlus

Töötlemisseaded loetakse sisse töötlemise käivitamisel. Seadete muutmine ei mõjuta tagasiulatuvalt juba kettale salvestatud tulemusi – uute seadete rakendamiseks tuleb töötlemine uuesti käivitada. Mõned seaded ei mõjuta töötlemist üldse:

* Pildi pisipildi resolutsioon (ainult kuvamiseks)
* Projekti malli salvestamine
* Projekti kausta salvestamine

***

## Konfiguratsioonivõtmete viide

Automatiseerimiseks (CLI `--config`, SDK `configure`, või `project.json` otse lugemise puhul) on need täpsed võtmed `Project Settings` all:

| Võtme asukoht | Tüüp | Vaikimisi |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | number 0-10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | number 0-100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | number 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | number -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | piiranguprofiili ID või `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | indeksite konfiguratsioonide loend | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Võtmed `Array alignment` kirjutatakse esmakordselt, kui sektsioon „Array Alignment“ renderitakse või kui automaatikakõne need seadistab. Kui need puuduvad, kasutab töötlemisahel samu eespool näidatud väärtusi (`true`, `true`, bilineaarne), seega käitub projekt, milles neid pole (.json), täpselt samamoodi kui projekt, milles need on olemas.

### Võtmed, mis on salvestatud `project.json`-is ja mille üle seadete paneelis puudub kontroll

Need asuvad samas `Project Settings`-puus ja töötlemise käigus loetakse neid, kuid külgribal ei leia nende jaoks vidinat:

| Võtme tee | Tüüp | Vaikimisi | Määrab |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Määrab ümber, kuidas LATTICE-sisend-TIFF-faile tõlgendatakse; `auto` järeldab iga faili`Chloros:ProcessingLevel` XMP-märgist ja kanalite arvu. Ignoreeritakse Survey3 `.raw` pildistamiste puhul. See ei ole tahtlikult GUI-seade — seadistus „auto“ on igas tavapärases olukorras õige. |
| `Processing → Target reflectance dir` | teepath | `""` | `chloros-cli process --target-reflectance-dir` või projekti sihtkoht API |
| `Processing → Target reflectance config` | kaamera seerianumbri järgi indekseeritud sõnastik | `{}` | Kaadris oleva sihtmärgi registreerimine (režiim `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | teepärastreng | `""` | SDK `process_folder(daq_log_path=…)`. Viitab `.daq` salvestusele või nende kausta |
| `Target Detection → Minimum calibration target squares` | number | `4` | Vanem vaikimisi seade; ei ole juhtimist ega CLI lipukest |
| `UI → Grid thumbnail size` | number | `160` | Pildiresti enda pisipiltide suumiliugur |

Kaks vaataja eelistust on salvestatud **`project.json`-i ülemisel tasandil**, täiesti väljaspool `Project Settings`, kuna need on pigem kuvamisseaded kui töötlemisseaded:

| Võtme tee | Tüüp | Vaikimisi | Määrab |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | täisarv 1–256 | `1` | Pildivahekaardi GSD (px) reguleerija — vt [Pildi avamine täisekraanil](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Parimad tavad

1. **Alustage vaikimisi seadetest**: Vaikimisi seaded sobivad hästi enamiku MAPIR kaamerasüsteemide ja tüüpiliste töövoogude jaoks.
2. **Loo mallid**: Kui oled seadistused konkreetse töövoo või kaamera jaoks optimeerinud, salvesta need mallina, et tagada ühtsus kõikides projektides.
3. **Testi enne täielikku töötlemist**: Uute seadistustega eksperimenteerimisel testi neid väikesel pildivalimil, enne kui töötled kogu andmekogumit.
4. **Dokumenteerige oma seaded**: Kasutage kirjeldavaid mallinimesid, mis näitavad kaamerasüsteemi, töötlemise tüüpi ja kasutusotstarvet (nt „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Ekspordivormingu valik**: Valige ekspordivorming vastavalt lõppkasutusele:
   * Teaduslik analüüs → TIFF (16-bitine või 32-bitine)
   * GIS-töötlus → TIFF (16-bitine)
   * Kiire visualiseerimine → PNG (8-bitine)
   * Veebis jagamine → JPG (8-bitine)

***

Lisateavet Chloros multispektraalsete indeksite kohta leiate lehelt [Multispektraalsete indeksite valemid](multispectral-index-formulas.md).
