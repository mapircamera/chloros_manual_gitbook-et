# Projekti seaded

Chloros-i külgribal asuvad projekti seaded <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

võimaldavad teil seadistada kõiki pilditöötluse, kalibreerimismärgi tuvastamise, multispektraalsete indeksite arvutamise ja projekti ekspordivõimaluste aspekte. Need seaded salvestatakse koos teie projektiga ning neid saab salvestada mallidena, et neid mitmes projektis korduvalt kasutada.

## Projektiseadetele juurdepääs

Projektiseadetele juurdepääsuks:

1. Avage projekt Chloros es
2. Klõpsake vasakul külgribal asuvat vahekaarti **Projektiseaded** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">


3. Seadete paneelis kuvatakse kõik saadaval olevad konfiguratsioonivõimalused kategooriate kaupa



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Teistest seadistustest sõltuvad seadistused on halliks muudetud.** Kui ülemine lüliti muudab mõne seadistuse võimatuks (näiteks *Peegelduskalibreerimine / valgebalanss* valiku tühistamine muudab *Peegelduse eksportimise* võimatuks), on sõltuv juhtelement keelatud ja selle abiteates on nimetatud lüliti, mida tuleb muuta.
{% endhint %}

***

## Ekraan

### Pildi pisipildi eraldusvõime

* **Tüüp**: rippmenüü
* **Valikud**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Vaikimisi**: Vaikimisi (512 px)
* **Kirjeldus**: Eraldusvõime (pikim serv, pikslites), millega pildivõrgu pisipildid renderdatakse. Suuremad väärtused näevad suurendamisel teravamad välja, kuid laadivad aeglasemalt ja kasutavad rohkem mälu. Täisresolutsioon vastab originaalpildi suurusele.
* **Märkus**: Ainult kuvamiseks — see ei mõjuta kunagi töötlemist ega eksporditud faile.***

## Sihtmärgi tuvastamine

Need seaded määravad, kuidas Chloros tuvastab ja töötleb teie piltidel kalibreerimisihtmärke. Mõlemad on aktiivsed ainult siis, kui **Peegelduskalibreerimine / valgebalanss** on sisse lülitatud (muul juhul on need hallid, kuna sihtmärgi tuvastamine jäetakse täielikult vahele).

### Kalibreerimisproovi minimaalne pindala (px)

* **Tüüp**: Number
* **Vahemik**: 0 kuni 10 000 pikslit
* **Vaikimisi**: 25 pikslit
* **Kirjeldus**: Määrab minimaalse pindala (pikslites), mis on vajalik, et tuvastatud piirkonda loetaks kehtivaks kalibreerimise sihtmärgi prooviks. Väiksemad väärtused tuvastavad väiksemaid sihtmärke, kuid võivad suurendada valetuvastuste arvu. Suuremad väärtused nõuavad tuvastamiseks suuremaid ja selgemaid sihtmärgi piirkondi.
* **Millal kohandada**:
  * Suurendage väärtust, kui väikeste pildiartefaktide puhul tekivad valed tuvastused
  * Vähendage väärtust, kui kalibreerimise sihtmärgid paistavad piltidel väikesed ja neid ei tuvastata

### Sihtmärkide klastrite miinimum (0–100)

* **Tüüp**: Arv
* **Vahemik**: 0 kuni 100
* **Vaikimisi**: 60
* **Kirjeldus**: Reguleerib klastrite moodustamise künnist sarnaste värvipiirkondade rühmitamiseks kalibreerimissihte tuvastades. Kõrgemad väärtused nõuavad, et rohkem sarnaseid värve rühmitataks kokku, mille tulemuseks on konservatiivsem sihtmärkide tuvastamine. Madalamad väärtused lubavad sihtmärkide rühmas rohkem värvivaheldusi.
* **Millal reguleerida**:
  * Suurendage, kui kalibreerimise sihtmärgid jagunevad mitmeks tuvastuseks
  * Vähendage, kui värvivaheldusega kalibreerimise sihtmärke ei tuvastata täielikult

***

## Töötlemine

Need seaded määravad, kuidas Chloros teie pilte töötleb ja kalibreerib.

### Vignettkorrektsioon

* **Tüüp**: Valikukast
* **Vaikimisi**: Lülitatud sisse (märgitud)
* **Kirjeldus**: Rakendab vinjeti korrigeerimist, et kompenseerida objektiivi poolt põhjustatud pildi servade tumenemist. Vinjetatsioon on levinud optiline nähtus, mille puhul pildi nurgad ja servad tunduvad objektiivi omaduste tõttu keskosast tumedamad.
* **Kõrvalmõju**: See lüliti valib ka selle, millist *kalibreerimata varutoodet* töötlus kirjutab (vt allpool).

### Peegelduskalibreerimine / valgebalanss

* **Tüüp**: Valikukast
* **Vaikimisi**: Lülitatud sisse (märgitud)
* **Kirjeldus**: Lülitab sisse peegelduskalibreerimise — kas raamis tuvastatud kalibreerimismärkide ja/või DAQ valgusanduri allapoole suunatud andmete põhjal, sõltuvalt kaamerast ja sellest, mis on saadaval. See normaliseerib peegeldusväärtused kogu andmekogus ja tagab ühtlased mõõtmistulemused sõltumata valgustingimustest.
* **Kui funktsioon on välja lülitatud**: Sihtmärkide tuvastamine jäetakse täielikult vahele ning**ükski kaamera ei saa luua peegeldusvõime toodet** — nii Survey3-i sihtmärkidel põhinev kui ka LATTICE DAQ-l põhinev süsteem. Sõltuvad seaded (*Ekspordi peegeldusvõime*, *Minimaalne kalibreerimisintervall* ja sihtmärkide tuvastamise künnised) on halliks muudetud.

### Kalibreerimata varuvõimalused: Ekspordi anduri vastus / Ekspordi vinjetikorrigeeritud

* **Tüüp**: Kaks märkeruutu
* **Vaikimisi**: Mõlemad on lubatud (märgitud)
* **Kirjeldus**: Kui kaadri peegeldusvõimet ei ole võimalik kalibreerida (kalibreerimissihte ei leitud või peegeldusvõime kalibreerimine on välja lülitatud), salvestatakse see selle asemel *kalibreerimata varutoodetena*. **Iga kaameramudeli kohta on ühe mõõtmistsükli jooksul olemas täpselt üks kahest varutootest**, mille valib lüliti *Vignette correction*:
  * Vignettkorrektsioon **sisse lülitatud**→ `Vignette_Corrected_Images/` (määrab**Ekspordi vignettkorrigeeritud**)
  * Vignettkorrektsioon **välja lülitatud**→ `Sensor_Response_Images/` (määrab**Ekspordi anduri vastus**)
* Kasutusel mitteolev varuvariant on halliks muudetud. Kasutusel oleva valiku märgistuse eemaldamine takistab selle faili salvestamist üldse.

### LATTICE-eksporttooted

LATTICE-kaadritega projektide puhul jaguneb iga imporditud LATTICE-kaader ühe töötlemisvooru jooksul kõikide aktiveeritud **ja kohaldatavate**toodete vahel. Jaotamist juhivad neli märkeruutu (kõik on vaikimisi**sisse lülitatud**):

| Seade | Väljundkaust | Mida eksporditakse |
| --- | --- | --- |
| **Ekspordi debayered** | `Debayered_Images/` | Lineaarne debayered-pilt: kalibreerimispaketi tume nihe on eemaldatud, seejärel on tehtud servadest lähtuv demosaik, ilma lineaarsuse korrigeerimise, tasase välja või segamise eemaldamiseta — sama fail, mille pilt salvestab debayered-tasandil. Säilitatakse anduri algses orientatsioonis (salvestatud XMP-s). Kehtib RGB- ja multispektraalsete kaamerate puhul. |
| **Eelvaate eksport** | `Preview_Images/` | Ekraanil kuvatav eelvaade. RGB = valgebalanss (DAQ-valgusallikas, kui see on saadaval, muidu hall maailm) + gamma; multispektraalne = valevärvide venitus. |
| **Ekspordi kiirgus** | `Radiance_Images/` | Float32 spektraalne kiirgus ühikutes W/m²/sr/nm. Ainult multispektraalne (M3C/M3M) — ei kehti RGB-põhifailide puhul. Kirjutatakse alati 32-bitise TIFF-failina, sõltumata seadistusest *Kalibreeritud pildiformaat*. |
| **Ekspordi peegeldusvõime**| `Reflectance_Calibrated_Images/` | Uint16 peegeldusvõime, skaalitud nii, et**32768 = peegeldusvõime 1,0** (märgitud XMP-na `Chloros:PixelScale`). Ainult multispektraalne, salvestatakse siis, kui vastav allapoole suunatud `.daq` kirje (või kvaliteedikontrolli läbinud kaadrisisesed sihtmärgid) katab kaadri. |

* RGB-põhikaamerad edastavad debayered-andmeid + eelvaadet; kiirgust ja peegeldusvõimet nende puhul ei salvestata, kuna need ei ole kohaldatavad.
* Debayered-andmete ja eelvaate bittisügavus järgib seadet *Kalibreeritud pildiformaat*; kiirgus on alati float32.
* Need neli lülitit ei mõjuta *Survey3

* töötlemist.
* Multispektraalsete M3C-kaamerate kiirguse ja peegeldusvõime andmed võivad kirjutada ka pikselipõhise **kinnitusmaski** `clamp_masks/` kausta tootekataloogide kõrvale, märkides, kus lainepikkuse segamise eemaldamine kärpis negatiivse väärtuse nullini. Vaata [Väljundpildiformaadid](../output-image-formats.md#clamp-masks).

Samad neli lülitit on olemas ka peata versioonina `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Need asendasid vana CLI-lipu `--radiometric-output`, mida enam ei eksisteeri.

{% hint style="warning" %}
**Kõigi asjakohaste toodete väljalülitamine põhjustab töötlemise ebaõnnestumise.** Alates versioonist 1.2.0 teatab töötlemisprotsess, millelt nõuti tooteid, kuid mis ei kirjutanud ühtegi pilditoodet, ebaõnnestumisest ning CLI lõpetab töö nullist erineva tulemusega, selle asemel et teatada vaikimisi õnnestumisest. Logis nimetatakse toodet, mida ei suudetud kirjutada, ja selle põhjust. Tahtlikult ainult metaandmetega töötlemine (midagi ei taotletud) loetakse endiselt edukaks.
{% endhint %}

### Peegeldusallikas (projekti seade, määratakse CLI

kaudu)

Projekt salvestab ka selle, millist **peegeldusviidet** LATTICE peegeldustoode kasutab. Seadete paneelis puudub selle jaoks eraldi juhtelement; väärtus salvestatakse projekti konfiguratsioonis kui `Processing → "Target reflectance source"` ja see määratakse `chloros-cli process --reflectance-source {auto,target,daq}` abil (SDK-s `configure()`-il puudub selle jaoks märksõna):

* **`auto`** (vaikimisi): kvaliteedikontrolli läbinud kaadrisisesest kalibreerimise sihtmärkist saab absoluutne viide; kui sihtmärki pole või kvaliteedikontroll ebaõnnestub, kasutatakse tagavarana DAQ-i allapoole suunatud jaotust (ρ = πL/E). Sihtmärki, mis läbib kvaliteedikontrolli lipuga (paneel on kukkunud või sagedusriba ei tööta), ei eelistata automaatselt: proovitakse järjest sihtmärki, sihtmärki+DAQ-i ja ainult DAQ-i ahelaid vastavalt nende mõõdetud skoorile väljajäetud paneelidel.
* **`target`**: range sihtmärgipõhine peegeldusvõime — DAQ-asendamist ei toimu.
* **`daq`**: DAQ-põhine peegeldusvõime; raamis olevaid sihtmärke ei kasutata võrdlusalusena.

Salvestatud väärtusega võrreldakse suurtähti arvestamata ning mõned kirjapildid aktsepteeritakse sünonüümidena: `target`, `target_image`, `empirical` ja `empirical_line` tähendavad kõik **sihtmärki**; `daq`, `dls`, `light_sensor` ja `sensor` tähendavad kõik**daq**. Kõik muu – sealhulgas puuduv võti – tõlgendatakse kui**auto**.

Ühiku kaupa **mõõdetud** sihtmärgi skaneeringuid otsitakse sihtmärgi seerianumbri/QR-koodi järgi, näiteks `<serial>.csv`, kolmest kohast: kataloogis, mis on määratud `--target-reflectance-dir`-iga (salvestatud kui `Processing → "Target reflectance dir"`), projekti enda `target_reflectance/`-kaustas, ning keskkonnamuutuja `CHLOROS_TARGET_REFLECTANCE_DIR` teekonnas. Kui selle seadme jaoks mõõdetud skaneeringut ei ole, kasutatakse selle asemel sihtmudeli avaldatud nominaalkõverat.

### Debayeri meetod

* **Tüüp**: rippmenüü
* **Valikud**:
  * Standard (kiire, keskmine kvaliteet)
  * Tekstuuritundlik (aeglane, kõrgeim kvaliteet) \[Chloros+]
* **Vaikimisi**: Standard (kiire, keskmine kvaliteet)
* **Kirjeldus**: Valib demosaicing-algoritmi, mida kasutatakse toores Bayer-mustriga andurite andmete teisendamiseks täisvärvilisteks piltideks. Meetod „Standard (kiire, keskmine kvaliteet)” pakub optimaalset tasakaalu töötlemiskiiruse ja pildikvaliteedi vahel. Meetod „Tekstuuritundlik (aeglane, kõrgeim kvaliteet)” \[Chloros+] kasutab kõrgekvaliteedilist servadest lähtuvat demosaicingut koos AI/ML müra eemaldamise mudeliga, mis eemaldab peaaegu kogu demosaicingust tuleneva müra. Tekstuuritundliku mudeli töötamiseks on vaja GPU mälu (VRAM). Soovitame seda kasutada, kui teil on kiirema töötlemise jaoks saadaval >4 GB VRAM-i.
* **Kui rida on üldse rippmenüü**: kahe valikuga rippmenüü ilmub ainult siis, kui**mõlemad** tingimused on täidetud — olete sisse logitud sobiva Chloros+ tellimusega **ja** projekt ei sisalda ühtegi LATTICE-kaadrit. Muidu kuvatakse rida tavalise tekstina `Standard (Fast, Medium Quality)`, ilma et oleks midagi valida.
* **LATTICE-märkus**: LATTICE-põhist tekstuuritundlikku mudelit ei ole olemas ning töötlemisahel sunnib LATTICE-kaadrite puhul kasutama standardset demosaiki, sõltumata salvestatud väärtusest. Kui lisate LATTICE-kausta projekti, kus oli juba valitud tekstuuritundlik režiim, kirjutab Chloros selle seadistuse ümber tagasi standardrežiimiks, selle asemel et jätta vananenud väärtus `project.json`.

### Minimaalne kalibreerimise intervall

* **Tüüp**: Arv
* **Vahemik**: 0 kuni 3600 sekundit
* **Vaikimisi**: 0 sekundit
* **Kirjeldus**: Määrab kalibreerimismärkide kasutamise vahelise minimaalse ajavahemiku (sekundites). Kui väärtuseks on seatud 0, kasutab Chloros kõiki tuvastatud kalibreerimismärke. Kui väärtuseks on seatud suurem number, kasutab Chloros ainult neid kalibreerimismärke, mille vaheline aeg on vähemalt nii palju sekundeid, vähendades sellega töötlemisaega andmekogumite puhul, kus kalibreerimismärke salvestatakse sageli.
* **Millal reguleerida**:
  * Seadke väärtuseks 0, et saavutada maksimaalne kalibreerimistäpsus muutuvates valgustingimustes
  * Suurendage väärtust (nt 60–300 sekundini), et kiirendada töötlemist, kui valgustingimused on stabiilsed ja kalibreerimismärkide pilte on palju

### Valgusanduri ajavööndi nihke

* **Tüüp**: Number
* **Vahemik**: -12 kuni +12 tundi
* **Vaikimisi**: 0 tundi
* **Kirjeldus**: Määrab valgusanduri andmete ajamärkide ajavööndi nihke (tundides UTC-st), mida kasutatakse valgusanduri logide sobitamisel pildistamise aegadega. Uuemad `.daq` salvestised sisaldavad oma ajavööndi päritolu, seega on seda vaja peamiselt vanemate, kohaliku aja järgi salvestatud logide puhul.

### Rakenda PPK-korrektsioone

* **Tüüp**: Valikukast
* **Vaikimisi**: Väljalülitatud (märkimata)
* **Kirjeldus**: Võimaldab kasutada GPS-i (GNSS) sisaldavatest MAPIR i DAQ-salvestitest pärit PPK-korrektsioone (Post-Processed Kinematic). Kui see on lubatud, kasutab Chloros kõik teie projektikataloogis olevaid .daq-logifaile, mis sisaldavad ekspositsiooni pin-andmeid, ning rakendab teie piltidele täpseid geolokatsiooni korrektsioone.
* **Nõue**: teie projektikataloogis peab olema .daq-logifail, mis sisaldab ekspositsioonipinnide andmeid
* **Millal aktiveerida**: Soovitatav on PPK-korrektsioon alati aktiveerida, kui teie .daq-logifailis on ekspositsiooni tagasiside andmed.

### Valgustuse kontakt 1

* **Tüüp**: rippmenüü
* **Nähtavus**: nähtav ainult siis, kui on aktiveeritud valik „Kohalda PPK-korrektsioone” JA eksponeerimise andmed on saadaval eksponeerimispiigile 1
* **Valikud**:
  * Projektis tuvastatud kaameramudelite nimed
  * „Ära kasuta” – ignoreeri seda eksponeerimispiiki
* **Vaikimisi**: Valitakse automaatselt projekti konfiguratsiooni alusel
* **Kirjeldus**: Määrab ekspositsiooni viigule 1 konkreetse kaamera PPK-aja sünkroniseerimiseks. Ekspositsiooni viik salvestab täpse ajahetke, mil kaamera katik vallandub, mis on täpse PPK-geolokatsiooni jaoks kriitilise tähtsusega.
* **Automaatvaliku toimimine**:
  * Üks kaamera + üks kontakt: valib kaamera automaatselt
  * Üks kaamera + kaks kontakti: kontakt 1 määratakse kaamerale automaatselt
  * Mitme kaamera korral: vaja on käsitsi valikut

### Ekspositsiooni kontakt 2

* **Tüüp**: Rippmenüü
* **Nähtavus**: Nähtav ainult siis, kui valik „Kohalda PPK-korrektsioone” on aktiveeritud JA kontaktile 2 on saadaval ekspositsiooniandmed
* **Valikud**:
  * Projektis tuvastatud kaameramudelite nimed
  * „Ära kasuta“ – ignoreeri seda ekspositsioonipinni
* **Vaikimisi**: Valitakse automaatselt projekti konfiguratsiooni alusel
* **Kirjeldus**: Määrab ekspositsioonipinnile 2 konkreetse kaamera PPK-ajasünkroniseerimiseks, kui kasutatakse kahe kaameraga seadistust.
* **Automaatvaliku toimimine**:
  * Üks kaamera + üks pin: pin 2 seatakse automaatselt asendisse „Ära kasuta“
  * Üks kaamera + kaks pini: pin 2 seatakse automaatselt asendisse „Ära kasuta“
  * Mitu kaamerat: vajalik käsitsi valik
* **Märkus**: Sama kaamerat ei saa määrata korraga nii kontaktile 1 kui ka kontaktile 2.***

## DAQ valgusandur

See osa kuvatakse projekti seadetest ja loetleb kõik projekti DAQ allapoole suunatud failid — `.daq` salvestused ja DAQ-M `.csv` allapoole suunatud logid. Vahekaardil „Valgusandurid“ tehtud salvestused lisatakse avatud projektile automaatselt.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Igas reas on näidatud fail, anduri mudel ja selle faili puhul tegelikult kehtiv hajutikorki korrigeerimine. Ridade kohal asub üks projektile kehtiv juhtnupp:

### Korki ülekirjutamine (kõik failid)

* **Tüüp**: rippmenüü
* **Valikud**: `Auto` ning projektis olevate anduritüüpide jaoks kehtivad katekorrektsiooniprofiilid
* **Vaikimisi**: Auto
* **Salvestatakse kui**: `Processing → "DAQ cap id"` (vaikimisi `auto`)
* **Kirjeldus**: `Auto` kasutab iga faili salvestatud katet (kui midagi pole salvestatud, eeldatakse Sunshine'i katet, kuid DAQ-U, DAQ-M ja DAQ-E tarnitakse ilma kateta ning Chloros ei näe, milline on paigaldatud, seega annab töötlemine hoiatuse eeldatava piirangu kohta: valige paigaldatud piirang). Konkreetse piirangu valimine tühistab **kõik** projektis olevad allapoole suunatud failid, välja arvatud DAQ-E-S-i omad, mille hajuti on osa kalibreerimisest; ainult DAQ-E-S-i sisaldavas projektis ei kuvata rippmenüüd: tooresalvestusi korrigeeritakse sellega, ja salvestusi, millel on juba kate, viidatakse uuesti (salvestatud korrigeerimine tühistatakse ja rakendatakse valitud korrigeerimine).
* **Oluline**: Valitud kate peab vastama salvestamise ajal füüsiliselt paigaldatud katele. Ei andur ega tarkvara suuda füüsilist katet tuvastada – vale kate-ID korrigeerib spektreid valesti.

Siin on teadlikult kasutatud **üht** kogu projekti hõlmavat juhtimist, mitte failipõhiseid rippmenüüsid: see seade mõjutab kõiki projekti allapoole suunatud allikaid.***

## Maatriksi joondamine

See jaotis kuvatakse **ainult** siis, kui vähemalt üks projekti pilt sisaldab moodulitevahelist joondamistransformatsiooni, mille LATTICE-massiivid salvestamise ajal lisavad (XMP-märgised `Chloros:Alignment*`). Siin näidatakse, mitmel pildil on joondamismärgised, milline kaamera on viitekaamera (`REF` märge) ning kaamerate kaupa tabel piltide arvuga.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Rakenda massiivi joondamist

* **Tüüp**: Valikukast
* **Vaikimisi**: Lubatud (märgitud)
* **Salvestatud kui**: `Processing → "Array alignment"`
* **Kirjeldus**: Muudab iga töödeldud tulemuse (debayered / eelvaade / kiirgus / peegeldus / indeks) massiivi ühisesse viitegeomeetriasse, kasutades pildistamisel salvestatud teisendust. Väljalülitatuna eksporditakse andmed iga anduri algses geomeetrias.

### Lõika ühisele kattuvale alale

* **Tüüp**: Valikukast (aktiivne ainult siis, kui *Rakenda massiivi joondamine* on sisse lülitatud)
* **Vaikimisi**: Sisse lülitatud (märgitud)
* **Salvestatud kui**: `Processing → "Array alignment crop"`
* **Kirjeldus**: Lõikab joondatud ekspordid piirkonnaks, mida kõik kaameramoodulid jagavad, nii et igal lainealal on sama pindala. Väljalülitamisel säilitatakse kogu anduri pind (allika välispiirid täidetakse mustaga).

### Ümberdiskrimineerimine

* **Tüüp**: Rippmenüü (aktiivne ainult siis, kui valik *Rakenda massiivi joondamine* on sisse lülitatud)
* **Valikud**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Vaikimisi**: Bilineaarne
* **Salvestatakse kui**: `Processing → "Array alignment interpolation"`
* **Kirjeldus**: Joondamise moonutuse poolt kasutatav interpolatsioon. Valik „Lähim“ säilitab täpsed allika väärtused (pikslitevahelist segamist ei toimu) ranget radiomeetrilist analüüsi jaoks; valik „Bilineaarne“ sobib kõige paremini kaardistamiseks ja visuaalseks kasutamiseks.

Samad kolm valikut on olemas ka ilma eesliiteta: `chloros-cli process --array-alignment`, `--array-alignment-crop` ja `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Need seaded võimaldavad teil konfigureerida multispektraalseid indekseid analüüsimiseks ja visualiseerimiseks.

### Lisa indeks

* **Tüüp**: Spetsiaalne indeksite seadistuspaneel
* **Kirjeldus**: Avab interaktiivse paneeli, kus saate valida ja seadistada multispektraalseid taimestikuindekseid (NDVI, NDRE, EVI jne), mida arvutada pilditöötluse käigus. Võite lisada mitu indeksit, millest igaühel on oma visualiseerimisseaded.
* **Saadaval olevad indeksid**: Graafilise kasutajaliidese rippmenüüs on**27** eelmääratud multispektraalset indeksivalemit (täieliku nimekirja, sealhulgas nimed, mida aktsepteerib ka CLI/SDK `--indices` valik).
* **Funktsioonid**:
  * Valige eeldefineeritud indeksivalemite hulgast
  * Lohistage kaamera filtri kanalid valemi lainepikkuste pesadesse
  * Konfigureerige visualiseerimise värvigradiente (LUT – Look-Up Tables)
  * Määrake künnisväärtused ja kärpimisrežiimid
  * Looge kohandatud indeksivalemid
* **Märkus**: Ühe sagedusribaga LATTICE M3M monokaamerate puhul indekseid ei arvutata — mitme sagedusribaga indeksid on ühe sagedusriba puhul määratlemata. See ei mõjuta mudeleid Survey3 ja LATTICE M3C.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Iga lisatud indeks kuvab oma valemi matemaatilise valemina, kus iga ribapesa juures on värviline ring: punane = Red, roheline = Green, sinine = Blue, oranž = Orange, tsüaan = Cyan, lilla = NIR, magenta = RE. Lohistage ring valemi kohal olevast reast pesale, et see seostada; topeltklõpsake seotud pesal, et see tühistada. Indeks arvutatakse ainult siis, kui igas valemis kasutatud pesas on kanal.

### Kohandatud valemid (Chloros+ funktsioon)

* **Tüüp**: Kohandatud valemite määratluste massiiv
* **Kättesaadavus**: Nõuab sisselogimist sobiva Chloros+ tellimusega.
* **Kirjeldus**: Võimaldab teil luua ja salvestada kohandatud multispektraalseid indeksivalemeid, kasutades lainepikkuste matemaatikat. Kohandatud valemid salvestatakse koos teie projekti seadistustega ja neid saab kasutada täpselt nagu sisseehitatud indekseid.
* **Kuidas luua**:
  1. Avage indeksite konfiguratsioonipaneelis kohandatud valemite kalkulaator
  2. Kirjutage valem, kasutades **ribalõigu sümboleid**, mitte ribade nimesid
  3. Salvesta valem kirjeldava nimega – see ilmub seejärel valemite rippmenüü alaossa ning saad oma kaamera kanaliringid selle pesadesse lohistada täpselt nagu sisseehitatud eelseadistuse puhul
* **Valemi süntaks**:
  * Spektriribade pesad: `x`, `y`, `z`, `a`, `b`, `c` — kuus positsiooni, mille saad tõmmates reaalsetele kanalitele kaardistada
  * Operaatorid: `+`, `-`, `*`, `/`, `^` ja `()` rühmitamiseks
  * Funktsioonid: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Miks sümbolid, mitte bändinimed**: valem, mis on kirjutatud kujul `(y-x)/(y+x)`, töötab igal kaameral, sest lohistamisel määratakse, kas `y` on RGN-filtri 850 nm NIR või OCN-filtri 808 nm NIR. Sisseehitatud eelseadistused salvestatakse samamoodi — vaata [Multispektraalsete indeksite valemeid](multispectral-index-formulas.md), et näha kõigi 27 valemi täpset sümbolivormi.
* **Kus need toimivad**: kohandatud valemid salvestatakse koos projekti seadetega ja neid saab kasutada nii [indeksi/LUT-liivakastis](../image-viewer-gui/index-lut-sandbox.md) kui ka töötlemisel. CLI- ja SDK-nimekiri `--indices` neid **ei** toeta, kuna see laiendab ainult 22 sisseehitatud eelseadistuse nime.***

## Eksport

Need seaded määravad eksporditavate töödeldud piltide formaadi ja kvaliteedi.

### Kalibreeritud pildi formaat

* **Tüüp**: rippmenüü
* **Valikud**:
  * **TIFF (16-bit)** – tihendamata 16-bitine TIFF-formaat
  * **TIFF (8-bit)** – tihendamata 8-bitine TIFF-vorming
  * **TIFF (32-bit, protsent)** – 32-bitine ujukomaga TIFF, kus peegeldusväärtused on väljendatud protsentides
  * **PNG (8-bit)** – tihendatud 8-bitine PNG-vorming
  * **JPG (8-bitine)** – pakitud 8-bitine JPEG-vorming
* **Vaikimisi**: TIFF (16-bitine); täielikult 8-bitine projekt kuvab TIFF (8-bitine) ja pakub ainult 8-bitiseid vorminguid
* **Kirjeldus**: Valib töödeldud ja kalibreeritud piltide salvestamise failivormingu. Eksporditud failid salvestatakse iga kaamera kausta sees asuvasse vormingupõhisesse alamkausta (`tiff16`, `tiff32`, `png8`, `jpg8`), kus iga toote kohta on üks `<Product>_Images/` kaust. Eksporditud failid säilitavad allika failinime – toodet identifitseerib kaust, mitte failinime laiend.
* **Soovitatavad formaadid**:
  * **TIFF (16-bitine)**: Soovitatav teaduslikuks analüüsiks ja professionaalseteks töövoogudeks. Säilitab maksimaalse andmekvaliteedi ilma tihendamisest tulenevate artefaktideta. Sobib kõige paremini multispektraalsele analüüsile ja edasisele töötlemisele GIS-tarkvaras.
  * **TIFF (32-bitine, protsent)**: Sobib kõige paremini töövoogudele, mis nõuavad peegeldusväärtusi protsentides (0–100%). Pakub maksimaalset täpsust radiomeetriliste mõõtmiste jaoks.
  * **PNG (8-bit)**: Sobib hästi veebis vaatamiseks ja üldiseks visualiseerimiseks. Väiksemad failisuurused kaotuseta pakkimisega, kuid vähenenud dünaamiline ulatus.
  * **JPG (8-bitine)**: Kõige väiksemad failisuurused, sobib ainult eelvaatamiseks ja veebis kuvamiseks. Kasutab kadudega pakkimist, mis ei sobi teaduslikuks analüüsiks.
* **Märkus**: LATTICE-kiirgus eksporditakse alati 32-bitise ujukomaga TIFF-failina, sõltumata sellest seadistusest.

***

## Projekti malli salvestamine

See funktsioon võimaldab teil salvestada oma praegused projekti seaded korduvkasutatava mallina.

* **Tüüp**: Teksti sisestusväli + Salvestamise nupp
* **Kirjeldus**: Sisestage oma seademallile kirjeldav nimi ja klõpsake salvestamise ikooni. Mall salvestab kõik teie praegused projekti seaded (sihtmärgi tuvastamine, töötlemisvalikud, indeksid, ja ekspordivorming), et neid oleks tulevastes projektides lihtne uuesti kasutada. Mallid salvestatakse teie projekti salvestuskausta sees asuvasse kausta `Project Templates/` ning neid saab valida või eksportida ka peamenüüst (*Vali mall* / *Salvesta mall* / *Ekspordi mall*).
* **Kasutusjuhtumid**:
  * Loo mallid erinevate kaamerasüsteemide jaoks (RGB, multispektraalne, NIR)
  * Salvesta standardseadistused konkreetsete põllukultuuride või analüüsitöövoogude jaoks
  * Jaga ühtseid seadeid kogu meeskonnaga
* **Kasutamine**:
  1. Määra kõik soovitud projekti seaded
  2. Sisestage malli nimi (nt „RedEdge Survey3 NDVI Standard”)
  3. Klõpsake salvestusikoonile
  4. Malli saab nüüd laadida uute projektide loomisel

***

## Projekti salvestuskaust

See seade määrab, kuhu uued projektid vaikimisi salvestatakse.

* **Tüüp**: kataloogitee kuvamine + nupp „Muuda“
* **Vaikimisi (Windows)**: `Documents\MAPIR\Chloros Projects` teie kaustas „Dokumendid“ — tavaliselt `C:\Users\[Username]\Documents\MAPIR\Chloros Projects` või seal, kuhu OneDrive või rändprofiil on kausta „Dokumendid“ viinud
* **Vaikimisi (Linux)**: `~/Documents/MAPIR/Chloros Projects`
* **Kirjeldus**: Näitab praegust vaikimisi kataloogi, kuhu luuakse uued Chloros-projektid. Klõpsake redigeerimisikoonile, et valida teine kataloog. Ülekirjutatud seade salvestatakse ühe tekstireana failis `~/.chloros/working_directory.txt` — Windowsis on see `C:\Users\<Username>\.chloros\working_directory.txt`. Kui see fail puudub või viitab teele, mida enam ei ole, kasutab Chloros eespool nimetatud vaikimisi kataloogi. CLI loeb ja kirjutab samasse faili, seega on `chloros-cli` ja graafiline kasutajaliides alati ühel meelel projektide asukoha osas.
* **Projektimallid** asuvad selle kataloogi alamkataloogis `Project Templates/`.
* **Millal muuta**:
  * Määrake võrgukettale meeskonnatöö jaoks
  * Vahetage suurema salvestusruumiga kettale suurte andmekogumite jaoks
  * Korraldage projektid aasta, kliendi või projekti tüübi järgi erinevatesse kaustadesse
* **Märkus**: Selle seadistuse muutmine mõjutab ainult UUSI projekte. Olemasolevad projektid jäävad oma algsetesse asukohtadesse.
* **Windows i teepikkus**: kui Windows i pikad teed ei ole sisse lülitatud (paigaldaja valik **Luba Windows i pikad teed**), lükatakse enne töötlemist tagasi projektikaust, mille nimi on pikem kui umbes 130 tähemärki, kuna selle tulemused ületaksid Windows i 260-tähemärgilise teepiirangu. Vaata [Töötlemise alustamine](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Seadete säilitamine

Chloros-projekt on **kaust**. Kõik projekti seaded salvestatakse selle sees asuvasse `project.json`-i; ühendatud riistvara salvestatakse koos nendega kaustadesse `cameras.json` ja `sensors.json`, mistõttu projekti uuesti avamisel ühendatakse ka selle kaamerad ja valgusandurid uuesti. Kui projekt uuesti avatakse, taastatakse kõik seaded täpselt sellisena, nagu need olid jäetud. Salvestatud projekte saab juhtida ka ilma kasutajaliideseta, kasutades faili `chloros-cli project` või SDK-i faili `open_project`.

### Seadete hierarhia

Seaded rakenduvad järgmises järjekorras:

1. **Süsteemi vaikimisi seaded** – Chloros i poolt määratud sisseehitatud vaikimisi seaded 2. **Malli seaded** – kui laadite projekti loomisel malli
3. **Salvestatud projekti seaded** – projektifailiga koos salvestatud seaded
4. **Käsitsi tehtud muudatused** – kõik muudatused, mida teete käesoleva seansi jooksul

### Seaded ja pilditöötlus

Töötlusseaded loetakse sisse töötluse käivitamisel. Seadete muutmine ei mõjuta tagasiulatuvalt juba kettale salvestatud tulemusi – uute seadete rakendamiseks tuleb töötlus uuesti käivitada. Mõned seaded ei mõjuta töötlust üldse:

* Pildi pisipildi eraldusvõime (ainult kuvamiseks)
* Salvesta projekti mall
* Salvesta projekti kaust

***

## Konfiguratsioonivõtmete viited

Automatiseerimiseks (CLI-i `--config`, SDK-i `configure` või `project.json`-i otselugemiseks) on need täpsed võtmed `Project Settings`-i all:

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
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Võtmed `Array alignment` kirjutatakse esmakordselt, kui sektsioon „Array Alignment“ renderitakse või kui automaatikakõne need seadistab. Kui need puuduvad, kasutab töötlusketi samu väärtusi, mis on näidatud eespool (`true`, `true`, bilineaarne), seega käitub projekt.json ilma nendeta täpselt samamoodi kui projekt, milles need on olemas.

### Võtmed, mis on salvestatud `project.json`-is ja mille üle seadete paneelis puudub kontroll

Need asuvad samas `Project Settings` puus ja töötlemise käigus loetakse neid, kuid külgribal ei leia te nende jaoks vidinat:

| Võtme tee | Tüüp | Vaikimisi | Määrab |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Ületab LATTICE-sisend-TIFF-failide tõlgendamise; `auto` järeldab iga faili `Chloros:ProcessingLevel` XMP-märgist ja kanalite arvu põhjal. Ignoreeritakse Survey3-i `.raw` salvestuste puhul. See ei ole tahtlikult GUI-seade — automaatne valik on igas tavapärases olukorras õige. |
| `Processing → Target reflectance dir` | teepath | `""` | `chloros-cli process --target-reflectance-dir` või projekti siht-API |
| `Processing → Target reflectance config` | kaamera seerianumbri järgi indekseeritud sõnastik | `{}` | Kaadris oleva sihtmärgi registreerimine (režiim `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | teekonna string | `""` | SDK `process_folder(daq_log_path=…)`. Viitab `.daq` salvestusele või nende kaustale |
| `Target Detection → Minimum calibration target squares` | number | `4` | Vana vaikimisi seade; ei ole juhtimist ega CLI-lipukest |
| `UI → Grid thumbnail size` | number | `160` | Pildivõrgu enda pisipiltide suumiliugur |

Kaks vaataja eelistust on salvestatud **`project.json`-i ülemisel tasandil**, täielikult väljaspool `Project Settings`-i, kuna need on pigem kuvamisolekud kui töötlemisseaded:

| Võtme tee | Tüüp | Vaikimisi | Määrab |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | täisarv 1–256 | `1` | Pildivahekaardi GSD (px) reguleerimisnupp — vt [Pildi avamine täisekraanil](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Parimad tavad

1. **Alustage vaikimisi seadetest**: Vaikimisi seaded sobivad hästi enamiku MAPIR i kaamerasüsteemide ja tüüpiliste töövoogude jaoks.
2. **Loo mallid**: Kui oled seadistused konkreetse töövoo või kaamera jaoks optimeerinud, salvesta need mallina, et tagada ühtsus projektide vahel.
3. **Testi enne täielikku töötlemist**: Uute seadistustega eksperimenteerimisel testi neid väikesel pildivalimil, enne kui töötled kogu andmekogumit.
4. **Dokumenteerige oma seaded**: Kasutage kirjeldavaid mallinimesid, mis näitavad kaamerasüsteemi, töötlemise tüüpi ja kavandatud kasutusotstarvet (nt „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Ekspordivormingu valik**: Valige ekspordivorming vastavalt lõppkasutusele:
   * Teaduslik analüüs → TIFF (16-bitine või 32-bitine)
   * GIS-töötlus → TIFF (16-bitine)
   * Kiire visualiseerimine → PNG (8-bitine)
   * Veebis jagamine → JPG (8-bitine)

***

Lisateavet multispektraalsete indeksite kohta programmis Chloros leiate lehelt [Multispektraalsete indeksite valemid](multispectral-index-formulas.md).
