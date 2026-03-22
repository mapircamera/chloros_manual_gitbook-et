# Projekti seaded

Projekti seaded <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> Chloros-is võimaldab teil konfigureerida kõiki pilditöötluse, kalibreerimise sihtmärgi tuvastamise, multispektraalsete indeksite arvutamise ja projekti ekspordivõimaluste aspekte. Need seaded salvestatakse koos teie projektiga ja neid saab salvestada mallidena, et neid mitmes projektis uuesti kasutada.

## Projektiseadetele juurdepääs

Projektiseadetele juurdepääsuks:

1. Avage projekt Chloros-is
2. Klõpsake vasakul külgribal **Projektiseaded**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> vasakul külgribal
3. Seadete paneelis kuvatakse kõik saadaval olevad konfiguratsiooni valikud kategooriate kaupa

***

## Sihtmärkide tuvastamine

Need seaded määravad, kuidas Chloros tuvastab ja töötleb kalibreerimissihte teie piltidel.

### Minimaalne kalibreerimisproovi pindala (px)

* **Tüüp**: Number
* **Vahemik**: 0 kuni 10 000 pikslit
* **Vaikimisi**: 25 pikslit
* **Kirjeldus**: Määrab minimaalse ala (pikslites), mis on vajalik, et tuvastatud piirkonda loetaks kehtivaks kalibreerimise sihtmärgi prooviks. Väiksemad väärtused tuvastavad väiksemaid sihtmärke, kuid võivad suurendada valetuvastuste arvu. Suuremad väärtused nõuavad tuvastamiseks suuremaid ja selgemaid sihtmärgi piirkondi.
* **Millal reguleerida**:
  * Suurendage, kui saate valehäireid väikeste pildiartefaktide puhul
  * Vähendage, kui teie kalibreerimise sihtmärgid paistavad piltidel väikesed ja neid ei tuvastata

### Minimaalne sihtmärkide klastrite arv (0–100)

* **Tüüp**: Number
* **Vahemik**: 0 kuni 100
* **Vaikimisi**: 60
* **Kirjeldus**: Reguleerib klastrite künnist sarnaste värviga piirkondade rühmitamiseks kalibreerimise sihtmärkide tuvastamisel. Suuremad väärtused nõuavad rohkem sarnaste värvide rühmitamist, mille tulemuseks on konservatiivsem sihtmärkide tuvastamine. Väiksemad väärtused lubavad sihtmärkide rühmas rohkem värvivaheldusi.
* **Millal reguleerida**:
  * Suurendage, kui kalibreerimise sihtmärgid jagunevad mitmeks tuvastuseks
  * Vähendage, kui värvivaheldusega kalibreerimise sihtmärke ei tuvastata täielikult

***

## Töötlemine

Need seaded reguleerivad, kuidas Chloros teie pilte töötleb ja kalibreerib.

### Vignettkorrektsioon

* **Tüüp**: Valikukast
* **Vaikimisi**: Lubatud (märgitud)
* **Kirjeldus**: Rakendab vignettkorrektsiooni, et kompenseerida objektiivi tumenemist piltide servades. Vignett on tavaline optiline nähtus, kus pildi nurgad ja servad tunduvad objektiivi omaduste tõttu tumedamad kui keskkosa.
* **Millal keelata**: Keelake see ainult juhul, kui teie kaamera/objektiivi kombinatsioon on vinjeti korrigeerimist juba rakendanud või kui soovite vinjetit järelkäsitluses käsitsi korrigeerida.

### Peegeldusvõime kalibreerimine / valgebalanss

* **Tüüp**: Valikukast
* **Vaikimisi**: Lubatud (märgitud)
* **Kirjeldus**: Lubab automaatset peegeldusvõime kalibreerimist, kasutades piltidel tuvastatud kalibreerimise sihtmärke. See normaliseerib peegeldusväärtused kogu andmekogus ja tagab ühtsed mõõtmistulemused sõltumata valgustingimustest.
* **Millal välja lülitada**: Lülitage välja ainult siis, kui soovite töödelda töötlemata, kalibreerimata pilte või kui kasutate teistsugust kalibreerimise töövoogu.

### Debayeri meetod

* **Tüüp**: Rippmenüü
* **Valikud**:
  * Standard (kiire, keskmine kvaliteet)
  * Tekstuuritundlik (aeglane, kõrgeim kvaliteet) \[Chloros+]
* **Vaikimisi**: Standard (kiire, keskmine kvaliteet)
* **Kirjeldus**: Valib demosaicing-algoritmi, mida kasutatakse töötlemata Bayeri mustri andurite andmete teisendamiseks täisvärvilisteks piltideks. Meetod „Standard (kiire, keskmine kvaliteet)” pakub optimaalset tasakaalu töötlemiskiiruse ja pildi kvaliteedi vahel. „Tekstuuritundlik (aeglane, kõrgeim kvaliteet)” \[Chloros+] kasutab kõrgekvaliteedilist servatundlikku demosaicingut koos AI/ML müra eemaldamise mudeliga, mis eemaldab peaaegu kogu demosaicingu müra. Tekstuuritundliku mudeli töötamiseks on vaja GPU mälu (VRAM). Soovitame seda kasutada, kui teil on kiirema töötlemise jaoks saadaval &gt;4 GB VRAM-i.
* **Märkus**: Chloros tulevastes versioonides võidakse lisada täiendavaid debayeri meetodeid.

### Minimaalne kalibreerimise intervall

* **Tüüp**: Number
* **Vahemik**: 0 kuni 3600 sekundit
* **Vaikimisi**: 0 sekundit
* **Kirjeldus**: Määrab minimaalse ajavahemiku (sekundites) kalibreerimise sihtmärkide kasutamise vahel. Kui väärtuseks on seatud 0, kasutab Chloros iga tuvastatud kalibreerimise sihtmärki. Kui väärtuseks on seatud suurem number, kasutab Chloros ainult neid kalibreerimise sihtmärke, mis on üksteisest eraldatud vähemalt selle arvu sekunditega, vähendades töötlemisaega andmekogumite puhul, kus kalibreerimise sihtmärke salvestatakse sagedasti.
* **Millal kohandada**:
  * Seadke väärtuseks 0, et saavutada maksimaalne kalibreerimistäpsus, kui valgustingimused muutuvad
  * Suurendage väärtust (nt 60–300 sekundini), et töötlemine oleks kiirem, kui valgustingimused on stabiilsed ja teil on sagedased kalibreerimismärgi pildid

### Valgusanduri ajavööndi nihke

* **Tüüp**: Number
* **Vahemik**: -12 kuni +12 tundi
* **Vaikimisi**: 0 tundi
* **Kirjeldus**: Määrab valgussensori andmete ajamärkide ajavööndi nihke (tundides UTC-st). Seda kasutatakse PPK (Post-Processed Kinematic) andmefailide töötlemisel, et tagada piltide salvestamise ja GPS-andmete vaheline õige ajasünkroniseerimine.
* **Millal kohandada**: Määrake see oma kohaliku ajavööndi nihkeks, kui teie PPK-andmed kasutavad UTC asemel kohalikku aega. Näiteks:
  * Vaikse ookeani aeg: -8 või -7 (sõltuvalt suveajast)
  * Ida-Ameerika aeg: -5 või -4 (sõltuvalt suveajast)
  * Kesk-Euroopa aeg: +1 või +2 (sõltuvalt suveajast)

### Rakenda PPK-korrektsioone

* **Tüüp**: Valikukast
* **Vaikimisi**: Keelatud (valikukast märkimata)
* **Kirjeldus**: Lubab kasutada GPS-i (GNSS) sisaldavate MAPIR DAQ-salvestite järelkinnitatud kinemaatilisi (PPK) korrektsioone. Kui see on lubatud, kasutab Chloros kõiki teie projektikataloogis olevaid .daq-logifaile, mis sisaldavad eksponeerimispin-andmeid, ja rakendab teie piltidele täpseid geolokatsiooni parandusi.
* **Nõue**: teie projektikataloogis peab olema .daq-logifail, mis sisaldab eksponeerimispin-kandeid
* **Millal lubada**: soovitatav on alati lubada PPK-parandusi, kui teie .daq-logifailis on eksponeerimise tagasiside kanded.

### Valgustuse kontakt 1

* **Tüüp**: Rippmenüü
* **Nähtavus**: Nähtav ainult siis, kui „Kohalda PPK-korrektsioone” on aktiveeritud JA valgustuse andmed on saadaval kontaktile 1
* **Valikud**:
  * Projektis tuvastatud kaameramudelite nimed
  * „Ära kasuta” – ignoreeri seda valgustuse kontakti
* **Vaikimisi**: Valitakse automaatselt projekti konfiguratsiooni alusel
* **Kirjeldus**: Määrab ekspositsiooni pistikule 1 konkreetse kaamera PPK-aja sünkroniseerimiseks. Ekspositsiooni pistik salvestab täpse ajahetke, mil kaamera katik vallandub, mis on täpse PPK-geolokatsiooni jaoks kriitilise tähtsusega.
* **Automaatvaliku käitumine**:
  * Üks kaamera + üks pistik: Valib kaamera automaatselt
  * Üks kaamera + kaks viiku: viik 1 määratakse kaamerale automaatselt
  * Mitmed kaamerad: vaja on käsitsi valikut

### Ekspositsiooniviik 2

* **Tüüp**: rippmenüü
* **Nähtavus**: nähtav ainult siis, kui „Kohalda PPK-parandusi” on sisse lülitatud JA ekspositsiooniandmed on saadaval viigule 2
* **Valikud**:
  * Projektis tuvastatud kaameramudelite nimed
  * „Ära kasuta” – ignoreeri seda ekspositsiooni viiku
* **Vaikimisi**: valitakse automaatselt projekti konfiguratsiooni alusel
* **Kirjeldus**: määrab ekspositsiooni viigule 2 konkreetse kaamera PPK-aja sünkroniseerimiseks kahe kaamera seadistuse kasutamisel.
* **Automaatvaliku käitumine**:
  * Üks kaamera + üks viik: viik 2 seatakse automaatselt „Ära kasuta”
  * Üks kaamera + kaks viiku: viik 2 seatakse automaatselt „Ära kasuta”
  * Mitme kaameraga: vajalik käsitsi valik
* **Märkus**: Sama kaamerat ei saa määrata korraga nii viigule 1 kui ka viigule 2.***

## Indeks

Need seaded võimaldavad teil konfigureerida multispektraalseid indekseid analüüsimiseks ja visualiseerimiseks.

### Lisa indeks

* **Tüüp**: spetsiaalne indeksite konfigureerimise paneel
* **Kirjeldus**: Avab interaktiivse paneeli, kus saate valida ja konfigureerida multispektraalseid taimestiku indekseid (NDVI, NDRE, EVI jne), mida arvutada pilditöötluse käigus. Saate lisada mitu indeksit, millest igaühel on oma visualiseerimisseaded.
* **Saadaval olevad indeksid**: Süsteem sisaldab üle 30 eelmääratud multispektraalset indeksit, sealhulgas:
  * NDVI (normaliseeritud erinevuse taimestiku indeks)
  * NDRE (normaliseeritud erinevus RedEdge)
  * EVI (täiustatud taimestiku indeks)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Ja palju muud (täieliku nimekirja leiate [Multispektraalsete indeksite valemid](multispectral-index-formulas.md) alt)
* **Funktsioonid**:
  * Valige eeldefineeritud indeksite valemite hulgast
  * Konfigureerige visualiseerimise värvigradiente (LUT – Look-Up Tables)
  * Määrake analüüsi künnisväärtused
  * Looge kohandatud indeksite valemeid

### Kohandatud valemid (Chloros+ funktsioon)

* **Tüüp**: Kohandatud valemite määratluste massiiv
* **Kirjeldus**: Võimaldab teil luua ja salvestada kohandatud multispektraalseid indeksivalemeid, kasutades ribade matemaatikat. Kohandatud valemid salvestatakse koos teie projekti seadetega ja neid saab kasutada täpselt nagu sisseehitatud indekseid.
* **Kuidas luua**:
  1. Otsige indeksite konfiguratsioonipaneelis kohandatud valemite valik
  2. Määrake oma valem, kasutades lainepikkuste identifikaatoreid (nt NIR, Red, Green, Blue)
  3. Salvesta valem kirjeldava nimega
* **Valemi süntaks**: Toetatakse standardseid matemaatilisi operatsioone, sealhulgas:
  * Aritmeetika: `+`, `-`, `*`, `/`
  * Sulgudes tegevuste järjekord
  * Vahemikuvihjed: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Eksport

Need seaded määravad eksporditavate töödeldud piltide formaadi ja kvaliteedi.

### Kalibreeritud pildi formaat

* **Tüüp**: Rippmenüü
* **Valikud**:
  * **TIFF (16-bit)** - Tihendamata 16-bitine TIFF formaat
  * **TIFF (32-bitine, protsent)** – 32-bitine ujukomaga TIFF, kus peegeldusväärtused on protsentides
  * **PNG (8-bitine)** - Tihendatud 8-bitine PNG-vorming
  * **JPG (8-bitine)** - Tihendatud 8-bitine JPEG-vorming
* **Vaikimisi**: TIFF (16-bitine)
* **Kirjeldus**: Valib töödeldud ja kalibreeritud piltide salvestamise failivormingu.
* **Vormingu soovitused**:
  * **TIFF (16-bitine)**: Soovitatav teaduslikuks analüüsiks ja professionaalseteks töövoogudeks. Säilitab maksimaalse andmekvaliteedi ilma tihendamisest tulenevate artefaktideta. Sobib kõige paremini multispektraalsele analüüsile ja edasisele töötlemisele GIS-tarkvaras.
  * **TIFF (32-bitine, protsent)**: Sobib kõige paremini töövoogudele, mis nõuavad peegeldusväärtusi protsentides (0–100%). Pakub maksimaalset täpsust radiomeetriliste mõõtmiste jaoks.
  * **PNG (8-bitine)**: Sobib veebivaatamiseks ja üldiseks visualiseerimiseks. Väiksemad failisuurused kaotuseta pakkimisega, kuid vähenenud dünaamiline ulatus.
  * **JPG (8-bitine)**: Väikseimad failisuurused, sobib ainult eelvaatamiseks ja veebis kuvamiseks. Kasutab kaotusega pakkimist, mis ei sobi teaduslikuks analüüsiks.***

## Projekti malli salvestamine

See funktsioon võimaldab teil salvestada oma praegused projekti seaded korduvkasutatava mallina.

* **Tüüp**: Teksti sisestus + Salvestamise nupp
* **Kirjeldus**: Sisestage oma seademallile kirjeldav nimi ja klõpsake salvestamise ikoonile. Mall salvestab kõik teie praegused projekti seaded (sihtmärgi tuvastamine, töötlemisvalikud, indeksid ja ekspordivorming) tulevaste projektide jaoks hõlpsaks korduvkasutamiseks.
* **Kasutusjuhtumid**:
  * Loo mallid erinevate kaamerasüsteemide jaoks (RGB, multispektraalne, NIR)
  * Salvesta standardkonfiguratsioonid konkreetsete põllukultuuride või analüüsitöövoogude jaoks
  * Jaga ühtseid seadeid kogu meeskonnaga
* **Kasutamine**:
  1. Konfigureerige kõik soovitud projekti seaded
  2. Sisestage malli nimi (nt „RedEdge Survey3 NDVI Standard”)
  3. Klõpsake salvestamise ikooni
  4. Mall on nüüd uute projektide loomisel laaditav

***

## Projekti kausta salvestamine

See seade määrab, kuhu uued projektid vaikimisi salvestatakse.

* **Tüüp**: Kataloogi tee kuvamine + Muuda nupp
* **Vaikimisi (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Vaikimisi (Linux)**: `~/.local/share/chloros/projects`
* **Kirjeldus**: Näitab praegust vaikimisi kataloogi, kuhu luuakse uued Chloros projektid. Klõpsake redigeerimisikoonil, et valida teine kataloog.
* **Millal muuta**:
  * Määrake võrgukettale meeskonnatöö jaoks
  * Vahetage suuremate andmekogumite jaoks ketta vastu, millel on rohkem salvestusruumi
  * Korraldage projektid aasta, kliendi või projekti tüübi järgi erinevatesse kaustadesse
* **Märkus**: Selle seadistuse muutmine mõjutab ainult UUSI projekte. Olemasolevad projektid jäävad oma algsetesse asukohtadesse.***

## Seadistuste püsivus

Kõik projekti seadistused salvestatakse automaatselt koos teie projektifailiga (`.mapir` projektivorming). Kui avate projekti uuesti, taastatakse kõik seaded täpselt sellisena, nagu need olid.

### Seadete hierarhia

Seaded rakendatakse järgmises järjekorras:

1. **Süsteemi vaikimisi seaded** – Chloros poolt määratud sisseehitatud vaikimisi seaded
2. **Malli seaded** – kui laadite projekti loomisel malli
3. **Salvestatud projekti seaded** – projektifailiga salvestatud seaded
4. **Käsitsi tehtud muudatused** – kõik muudatused, mida teete käesoleva seansi jooksul

### Seaded ja pilditöötlus

Enamik seadete muudatusi (eriti kategooriates „Töötlus” ja „Eksport”) käivitab piltide uuesti töötlemise, et kajastada uusi seadeid. Mõned seaded on aga „ainult ekspordiks” ja ei vaja kohest uuesti töötlemist:

* Projekti malli salvestamine
* Töökataloog
* Kalibreeritud pildiformaat (kehtib eksportimisel)

***

## Parimad tavad

1. **Alusta vaikimisi seadetega**: Vaikimisi seaded sobivad hästi enamiku MAPIR kaamerasüsteemide ja tüüpiliste töövoogude jaoks.
2. **Loo mallid**: Kui oled seaded konkreetse töövoo või kaamera jaoks optimeerinud, salvesta need mallina, et tagada projektidevaheline järjepidevus.
3. **Testige enne täielikku töötlemist**: Kui katsetate uusi seadeid, testige neid väikesel pildivalimil enne kogu andmekogumi töötlemist.
4. **Dokumenteerige oma seaded**: Kasutage kirjeldavaid mallinimesid, mis näitavad kaamerasüsteemi, töötlemise tüüpi ja kavandatud kasutust (nt „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Ekspordivormingu valik**: Valige ekspordivorming vastavalt lõppkasutusele:
   * Teaduslik analüüs → TIFF (16-bitine või 32-bitine)
   * GIS-töötlus → TIFF (16-bitine)
   * Kiire visualiseerimine → PNG (8-bitine)
   * Veebis jagamine → JPG (8-bitine)

***

Lisateavet Chloros multispektraalsete indeksite kohta leiate lehelt [Multispektraalsete indeksite valemid](multispectral-index-formulas.md).
