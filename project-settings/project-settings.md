# Projekti seaded

Projekti seaded <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> kõrvalribal Chloros võimaldab teil konfigureerida kõik pilditöötluse, kalibreerimise sihtmärgi tuvastamise, multispektraalsete indeksite arvutamise ja ekspordivõimaluste aspektid oma projektis. Need seaded salvestatakse koos projektiga ja neid saab salvestada mallidena, et neid mitmes projektis uuesti kasutada.

## Projektiseadete avamine

Projektiseadete avamiseks:

1. Avage projekt Chloros-is
2. Klõpsake vasakul külgribal **Projektiseaded**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> vasakul külgribal
3. Seadete paneel kuvab kõik kättesaadavad konfiguratsiooni valikud kategooriate kaupa

***

## Sihtmärgi tuvastamine

Need seaded kontrollivad, kuidas Chloros tuvastab ja töötleb kalibreerimise sihtmärke teie piltidel.

### Minimaalne kalibreerimise valimi pindala (px)

* **Tüüp**: number
* **Vahemik**: 0 kuni 10 000 pikslit
* **Vaikimisi**: 25 pikslit
* **Kirjeldus**: määrab minimaalse ala (pikslites), mis on vajalik, et tuvastatud piirkond loetaks kehtivaks kalibreerimise sihtmärgi prooviks. Väiksemad väärtused tuvastavad väiksemad sihtmärgid, kuid võivad suurendada valepositiivseid tulemusi. Suuremad väärtused nõuavad tuvastamiseks suuremaid ja selgemaid sihtmärgi piirkondi.
* **Millal kohandada**:
  * Suurendage, kui saate valepositiivseid tulemusi väikeste pildiartefaktide puhul.
  * Vähendage, kui kalibreerimise sihtmärgid on piltidel väikesed ja neid ei tuvastata.

### Minimaalne sihtmärkide klastrite arv (0–100)

* **Tüüp**: number
* **Vahemik**: 0–100
* **Vaikimisi**: 60
* **Kirjeldus**: Reguleerib klastrite künnist sarnaste värvialade rühmitamiseks kalibreerimise sihtmärkide tuvastamisel. Kõrgemad väärtused nõuavad sarnasemate värvide rühmitamist, mis toob kaasa konservatiivsema sihtmärkide tuvastamise. Madalamad väärtused lubavad sihtmärkide rühmas suuremat värvivalikut.
* **Millal reguleerida**:
  * Suurendage, kui kalibreerimise sihtmärgid jagunevad mitmeks tuvastamiseks.
  * Vähendage, kui värvivalikuga kalibreerimise sihtmärgid ei tuvastata täielikult.

***

## Töötlemine

Need seaded kontrollivad, kuidas Chloros töötleb ja kalibreerib teie pilte.

### Vignette&#x27;i korrigeerimine

* **Tüüp**: Valikukast
* **Vaikimisi**: Lubatud (valitud)
* **Kirjeldus**: Rakendab vignette&#x27;i korrigeerimist, et kompenseerida objektiivi tumenemist pildi servades. Vignett on tavaline optiline nähtus, mille puhul pildi nurgad ja servad on objektiivi omaduste tõttu tumedamad kui keskosa.
* **Millal keelata**: Keelake ainult juhul, kui teie kaamera/objektiivi kombinatsioon on juba rakendanud vignettide korrigeerimist või kui soovite vignette käsitsi korrigeerida järel töötlemisel.

### Peegelduskalibreerimine / valgebalanss

* **Tüüp**: Valikukast
* **Vaikimisi**: Lubatud (valitud)
* **Kirjeldus**: võimaldab automaatset peegelduskalibreerimist, kasutades piltidel tuvastatud kalibreerimise sihtmärke. See normaliseerib peegeldusväärtused kogu andmekogus ja tagab ühtlased mõõtmised olenemata valgustingimustest.
* **Millal keelata**: keelake ainult siis, kui soovite töödelda töötlemata, kalibreerimata pilte või kui kasutate teistsugust kalibreerimise töövoogu.

### Debayer-meetod

* **Tüüp**: rippmenüü
* **Valikud**:
  * Kõrge kvaliteet (kiirem) – praegu ainus saadaval olev valik
* **Vaikimisi**: Kõrge kvaliteet (kiirem)
* **Kirjeldus**: Valib demosaicing-algoritmi, mida kasutatakse töötlemata Bayer-mustri andurite andmete teisendamiseks täisvärvilisteks piltideks. Meetod „Kõrge kvaliteet (kiirem)” tagab optimaalse tasakaalu töötlemiskiiruse ja pildi kvaliteedi vahel.
* **Märkus**: Chloros tulevastes versioonides võidakse lisada täiendavaid debayer-meetodeid.

### Minimaalne kalibreerimise intervall

* **Tüüp**: number
* **Vahemik**: 0 kuni 3600 sekundit
* **Vaikimisi**: 0 sekundit
* **Kirjeldus**: Seab minimaalse ajavahemiku (sekundites) kalibreerimise sihtmärkide kasutamise vahel. Kui väärtuseks on seatud 0, kasutab Chloros kõiki tuvastatud kalibreerimise sihtmärke. Kui väärtuseks on seatud suurem number, kasutab Chloros ainult kalibreerimise sihtmärke, mis on üksteisest eraldatud vähemalt selle arvu sekunditega, vähendades töötlemisaega andmekogumite puhul, kus kalibreerimise sihtmärke on sageli.
* **Millal reguleerida**:
  * Seadke väärtuseks 0, et saavutada maksimaalne kalibreerimistäpsus, kui valgustingimused muutuvad.
  * Suurendage väärtust (nt 60–300 sekundini), et kiirendada töötlemist, kui valgustingimused on ühtlased ja teil on sagedased kalibreerimise sihtmärgi pildid.

### Valgusanduri ajavööndi nihke

* **Tüüp**: number
* **Vahemik**: -12 kuni +12 tundi
* **Vaikimisi**: 0 tundi
* **Kirjeldus**: Määrab valgussensori andmete ajamärgete ajavööndi nihke (tundides UTC-st). Seda kasutatakse PPK (Post-Processed Kinematic) andmefailide töötlemisel, et tagada piltide salvestamise ja GPS-andmete õige ajaline sünkroniseerimine.
* **Millal kohandada**: Määrake see oma kohalikule ajavööndi nihkele, kui teie PPK andmed kasutavad kohalikku aega UTC asemel. Näiteks:
  * Vaikse ookeani aeg: -8 või -7 (sõltuvalt suveajast)
  * Ida-Ameerika aeg: -5 või -4 (sõltuvalt suveajast)
  * Kesk-Euroopa aeg: +1 või +2 (sõltuvalt suveajast)

### PPK-paranduste rakendamine

* **Tüüp**: Valikukast
* **Vaikimisi**: Keelatud (valikukast ei ole märgitud)
* **Kirjeldus**: Võimaldab kasutada MAPIR DAQ salvestite järelkäsitletud kinemaatilisi (PPK) parandusi, mis sisaldavad GPS-i (GNSS). Kui see on lubatud, kasutab Chloros kõiki .daq logifailid, mis sisaldavad eksponeerimispinna andmeid teie projektikataloogis, ja rakendab teie piltidele täpseid geolokatsiooni parandusi.
* **Nõue**: .daq logifail eksponeerimispinna sissekannetega peab olema teie projektikataloogis olemas.
* **Millal lubada**: Soovitatav on alati lubada PPK parandusi, kui teie .daq logifailis on eksponeerimise tagasiside sissekanded.

### Ekspositsioonipin 1

* **Tüüp**: rippmenüü valik
* **Nähtavus**: nähtav ainult siis, kui „Kohalda PPK-parandusi” on aktiveeritud JA ekspositsiooniandmed on saadaval pin 1 jaoks
* **Valikud**:
  * Projektis tuvastatud kaameramudelite nimed
  * „Ära kasuta” – ignoreeri see ekspositsioonipin
* **Vaikimisi**: valitakse automaatselt projekti konfiguratsiooni alusel
* **Kirjeldus**: määrab PPK aja sünkroniseerimiseks ekspositsiooni pin 1-le konkreetse kaamera. Ekspositsiooni pin salvestab kaamera katiku täpse käivitamise aja, mis on täpse PPK geolokatsiooni jaoks väga oluline.
* **Automaatse valiku toimimine**:
  * Üks kaamera + üks pin: valib kaamera automaatselt
  * Üks kaamera + kaks pistikupesa: pistikupesa 1 määratakse automaatselt kaamerale
  * Mitmed kaamerad: vajalik käsitsi valik

### Ekspositsiooni pistikupesa 2

* **Tüüp**: rippmenüü valik
* **Nähtavus**: nähtav ainult siis, kui „Kohalda PPK parandusi” on aktiveeritud JA ekspositsiooni andmed on pistikupesa 2 jaoks kättesaadavad
* **Valikud**:
  * Projektis tuvastatud kaameramudelite nimed
  * „Ära kasuta” – ignoreeri see ekspositsiooni pin
* **Vaikimisi**: valitakse automaatselt projekti konfiguratsiooni alusel
* **Kirjeldus**: määrab kahe kaamera seadistuse kasutamisel PPK aja sünkroniseerimiseks ekspositsiooni pinile 2 kindla kaamera.
* **Automaatse valiku käitumine**:
  * Üks kaamera + üks pin: pin 2 määratakse automaatselt „Ära kasuta”
  * Üks kaamera + kaks pistikupesa: pistikupesa 2 seatakse automaatselt „Ära kasuta”
  * Mitmed kaamerad: vajalik käsitsi valik
* **Märkus**: sama kaamerat ei saa määrata samaaegselt nii pistikupesale 1 kui ka pistikupesale 2.

***

## Indeks

Need seaded võimaldavad teil konfigureerida multispektraalsed indeksid analüüsimiseks ja visualiseerimiseks.

### Indeksi lisamine

* **Tüüp**: spetsiaalne indeksi konfiguratsiooni paneel
* **Kirjeldus**: avab interaktiivse paneeli, kus saate valida ja konfigureerida multispektraalsed taimestiku indeksid (NDVI, NDRE, EVI jne), mida arvutada pilditöötluse käigus. Saate lisada mitu indeksit, millest igaühel on oma visualiseerimise seaded.
* **Saadaval olevad indeksid**: Süsteem sisaldab üle 30 eeldefineeritud multispektraalset indeksit, sealhulgas:
  * NDVI (normaliseeritud erinevusvegetatsiooni indeks)
  * NDRE (normaliseeritud erinevus RedEdge)
  * EVI (täiustatud taimestiku indeks)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Ja palju muud (täieliku loendi leiate [Multispektraalsete indeksite valemid](multispectral-index-formulas.md))
* **Funktsioonid**:
  * Valige eeldefineeritud indeksite valemite hulgast
  * Konfigureerige visualiseerimise värvigradientid (LUT – Look-Up Tables)
  * Määrake analüüsi künnisväärtused
  * Looge kohandatud indeksite valemid

### Kohandatud valemid (Chloros+ funktsioon)

* **Tüüp**: kohandatud valemite määratluste massiiv
* **Kirjeldus**: võimaldab luua ja salvestada kohandatud multispektraalsed indeksivalemid, kasutades riba matemaatikat. Kohandatud valemid salvestatakse koos projekti seadetega ja neid saab kasutada nagu sisseehitatud indekseid.
* **Kuidas luua**:
  1. Indeksi konfiguratsiooni paneelis otsige kohandatud valemi valik
  2. Määrake valem, kasutades riba identifikaatoreid (nt NIR, Red, Green, Blue)
  3. Salvesta valem kirjeldava nimega
* **Valemi süntaks**: Toetatakse standardseid matemaatilisi operatsioone, sealhulgas:
  * Aritmeetika: `+`, `-`, `*`, `/`
  * Sulgudes operatsioonide järjekord
  * Band viited: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Eksport

Need seaded määravad eksporditavate töödeldud piltide formaadi ja kvaliteedi.

### Kalibreeritud pildiformaat

* **Tüüp**: rippmenüü
* **Valikud**:
  * **TIFF (16-bitine)** – pakkimata 16-bitine TIFF formaat
  * **TIFF (32-bitine, protsent)** – 32-bitine ujukomaga TIFF, mille peegeldusväärtused on protsentides
  * **PNG (8-bitine)** - Pakitud 8-bitine PNG formaat
  * **JPG (8-bitine)** - Pakitud 8-bitine JPEG formaat
* **Vaikimisi**: TIFF (16-bitine)
* **Kirjeldus**: Valib failivormingu töödeldud ja kalibreeritud piltide salvestamiseks.
* **Vormingu soovitused**:
  * **TIFF (16-bitine)**: Soovitatav teaduslikuks analüüsiks ja professionaalseteks töövoogudeks. Säilitab maksimaalse andmekvaliteedi ilma kompressiooniartefaktideta. Parim multispektraalanalüüsiks ja edasiseks töötlemiseks GIS-tarkvaras.
  * **TIFF (32-bitine, protsent)**: Sobib kõige paremini töövoogudele, mis nõuavad peegeldusväärtusi protsentides (0–100%). Pakub maksimaalset täpsust radiomeetriliste mõõtmiste jaoks.
  * **PNG (8-bitine)**: Sobib hästi veebis vaatamiseks ja üldiseks visualiseerimiseks. Väiksemad failisuurused kaotuseta pakkimisega, kuid väiksem dünaamiline ulatus.
  * **JPG (8-bitine)**: Väikseimad failisuurused, sobib kõige paremini eelvaatamiseks ja veebis kuvamiseks. Kasutab kaotusega pakkimist, mis ei sobi teaduslikuks analüüsiks.

***

## Projekti malli salvestamine

See funktsioon võimaldab teil salvestada oma praegused projekti seaded korduvkasutatava mallina.

* **Tüüp**: Teksti sisestamine + Salvesta nupp
* **Kirjeldus**: Sisestage oma seadete mallile kirjeldav nimi ja klõpsake salvestamise ikooni. Mall salvestab kõik teie praegused projekti seaded (sihtmärgi tuvastamine, töötlemisvalikud, indeksid ja ekspordivorming) tulevaste projektide jaoks hõlpsaks taaskasutamiseks.
* **Kasutusjuhtumid**:
  * Loo mallid erinevatele kaamerasüsteemidele (RGB, multispektraalne, NIR)
  * Salvesta standardkonfiguratsioonid konkreetsetele põllukultuuridele või analüüsi töövoogudele
  * Jaga ühtsed seaded kogu meeskonnaga
* **Kasutamine**:
  1. Konfigureeri kõik soovitud projekti seaded
  2. Sisestage malli nimi (nt „RedEdge Survey3 NDVI Standard”)
  3. Klõpsake salvestamise ikooni
  4. Mall on nüüd uute projektide loomisel laaditav

***

## Projekti kausta salvestamine

See seade määrab, kuhu uued projektid vaikimisi salvestatakse.

* **Tüüp**: Kataloogi tee kuvamine + redigeerimisnupp
* **Vaikimisi**: `C:\Users\[Username]\Chloros Projects`
* **Kirjeldus**: Näitab praegust vaikimisi kataloogi, kuhu uued Chloros projektid loodud. Klõpsake redigeerimisikooni, et valida teine kataloog.
* **Millal muuta**:
  * Määrake võrgukettaks meeskonnatöö jaoks
  * Muutke suuremate andmekogumite jaoks kettaks, millel on rohkem salvestusruumi
  * Korraldage projektid aasta, kliendi või projekti tüübi järgi erinevatesse kaustadesse
* **Märkus**: Selle seade muutmine mõjutab ainult UUSI projekte. Olemasolevad projektid jäävad oma algsetesse asukohtadesse.

***

## Seadete püsivus

Kõik projekti seaded salvestatakse automaatselt koos projektifailiga (`.mapir` projekti formaat). Kui avate projekti uuesti, taastatakse kõik seaded täpselt nii, nagu need olid.

### Seadete hierarhia

Seaded rakenduvad järgmises järjekorras:

1. **Süsteemi vaikimisi seaded** – Chloros poolt määratud sisseehitatud vaikimisi seaded
2. **Malliseaded** – kui laadite projekti loomisel malli
3. **Salvestatud projekti seaded** – projektifailiga salvestatud seaded
4. **Käsitsi tehtud muudatused** – kõik muudatused, mis teete praeguse seansi jooksul

### Seaded ja pilditöötlus

Enamik seadeid (eriti kategooriates „Töötlus” ja „Eksport”) käivitab piltide uuesti töötlemise, et kajastada uusi seadeid. Mõned seaded on aga „ainult ekspordiks” ja ei vaja kohest töötlemist:

* Projekti malli salvestamine
* Töökataloog
* Kalibreeritud pildiformaat (kehtib eksportimisel)

***

## Parimad tavad

1. **Alustage vaikimisi seadetega**: vaikimisi seaded sobivad hästi enamiku MAPIR kaamerasüsteemide ja tüüpiliste töövoogude jaoks.
2. **Loo mallid**: kui oled optimeerinud seaded konkreetse töövoo või kaamera jaoks, salvesta need mallina, et tagada projektide ühtsus.
3. **Testige enne täielikku töötlemist**: kui katsetate uusi seadeid, testige neid väiksel pildivalimil enne kogu andmekogumi töötlemist.
4. **Dokumenteerige oma seaded**: kasutage kirjeldavaid mallinimesid, mis näitavad kaamerasüsteemi, töötlemise tüüpi ja kasutusotstarvet (nt „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Ekspordivormingu valik**: valige ekspordivorming vastavalt lõppkasutusele:
   * Teaduslik analüüs → TIFF (16-bitine või 32-bitine)
   * GIS-töötlemine → TIFF (16-bitine)
   * Kiire visualiseerimine → PNG (8-bitine)
   * Veebis jagamine → JPG (8-bitine)

***

Lisateavet Chloros multispektraalsete indeksite kohta leiate lehelt [Multispektraalsete indeksite valemid](multispectral-index-formulas.md).
