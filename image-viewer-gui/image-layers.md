# Pildikihtid

Pildivaataja paremas ülanurgas asuv **kihtide rippmenüü** võimaldab vaadata kõiki vaatamisel oleva pildi versioone – alates algpildist kuni iga töödeldud tooteni ja arvutatud indeksipiltideni – ilma vaatajast lahkumata.

## Mis on pildikihtid?

„Kiht“ Chloros

es on üks **tootet fail**, mis on registreeritud ühe algpildi juurde. Importimisel saad algfailid; töötlemise käigus lisatakse kiht iga toote jaoks, mille töötlemistsükkel on loonud. Eksporditud failid säilitavad allikafaili nime – toodet identifitseerib**kaust** ning kihi nimi on Chloros

e poolt sellele kaustale antud märge.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Kihiloend

### Alati olemas

| Kiht | Mis see on |
| --- | --- |
| **JPG**(või**PNG

**/**TIFF

**) | Pildistamisel saadud algfail. Survey3

impordib iga `.RAW` kõrvale faili `.JPG`; LATTICE-pildistamised toovad kaasa PNG

- või TIFF

-kuvaeelvaate. Märgistatud vastavalt sellele, mis tegelikult imporditi |
| **RAW (originaal)** | Allikaseks olev töötlemata kaader, kuvamiseks debayereeritud, ilma korrigeerimisteta. Saadaval alates importimise hetkest — ei vaja töötlemist |

LATTICE-salvestusel, mille alusfailiks **on** selle toorkaader, puudub eraldi aluskanne: `RAW (Original)` hõlmab seda juba.

### Survey3

töötlemistulemused

| Kiht | Salvestatud | Olemas alates |
| --- | --- | --- |
| **RAW (sihtmärk)** | — | Kaadri puhul tuvastati, et see sisaldab kalibreerimisihtmärki |
| **RAW (peegeldusvõime)** | `Reflectance_Calibrated_Images/` | Selle kaadri peegeldusvõime kalibreerimine õnnestus |
| **Vignettkorrektsioon**| `Vignette_Corrected_Images/` | Kaadrit ei olnud võimalik peegelduskalibreerida**ja** *vignettkorrektsioon* oli sisse lülitatud |
| **Anduri vastus**| `Sensor_Response_Images/` | Kaadrit ei olnud võimalik peegelduskalibreerida**ja** *vignettkorrektsioon* oli välja lülitatud |
| **Valge tasakaal** | `White_Balanced_Images/` | Valge tasakaaluga toode salvestati |

{% hint style="info" %}
**Vignettkorrektsioon ja anduri tundlikkus on alternatiivid, mitte kunagi mõlemad korraga.** Iga kaameramudeli kohta on ühe töötlemistsükli jooksul täpselt üks kalibreerimata varutoode ning *Vignettkorrektsiooni* lüliti valib, milline neist see on. Vaata [Projekti seaded](../project-settings/project-settings.md).
{% endhint %}

### LATTICE tasemed

LATTICE salvestab need ühe töötlemisvooru jooksul. Millised neist on olemas, sõltub projektiseadete tootekohastest ekspordilülititest ja sellest, mis kehtib konkreetse kaamera puhul.

| Kiht | Salvestatakse | Kehtib |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB

ja multispektraalne |
| **RAW (eelvaade)** | `Preview_Images/` | Multispektraalne (värvide võltsvärviga venitamine) |
| **Valgebalanss** | `Preview_Images/` | RGB

-põhikaamerad — RGB

-eelvaade on registreeritud selle nime all, et see oleks kooskõlas sama nimega Survey3

-kihiga |
| **RAW (kiirgus)** | `Radiance_Images/` | Ainult multispektraalne |
| **RAW (peegeldus)** | `Reflectance_Calibrated_Images/` | Ainult multispektraalne ja ainult juhul, kui kaadrit katab vastav `.daq` allapoole suunatud andmekirje või kvaliteedikontrolli läbinud kaadrisisesed sihtmärgid |

RGB

-põhikaameratel puudub ribapõhine radiomeetria, seega jäetakse nende puhul kiirgus ja peegeldusvahe **mittekohaldatavaks** — logifailis märgitakse seda, mitte ei jäeta seda märkimata.

{% hint style="info" %}
**Toored ja debayereeritud kihid joonduvad toodetega (uus versioonis 1.3.0).**LATTICE toored ja debayereeritud failid salvestatakse anduri algses orientatsioonis ning salvestavad XMP-s neile kuuluva orientatsiooni; teised tooted salvestatakse juba selle orientatsiooniga. Vaataja ja pildivõrk lülitavad kuvamiseks**RAW (originaal)**ja**RAW (debayered)** vastavalt sellele andmele, nii et kaader enam ei pöördu ümber, kui vahetate tootekihi ja toorandmekihi vahel. See puudutab ainult kuvamist — faile ei kirjutata kunagi ümber. Suunaandmeteta failid (mis on salvestatud enne versiooni 1.3.0 või loodud mõne teise tarkvaraga) kuvatakse sellisena, nagu need on salvestatud. Vaata [Väljundpildiformaadid](../output-image-formats.md#orientation).
{% endhint %}

Klambermaskid (`*_clampmask.tif` kaustas `clamp_masks/`) ei ole kihid: Chloros

jätab need importimisel vahele. Vaata [Väljundpildiformaadid](../output-image-formats.md#clamp-masks).

### Indeks-, LUT- ja liivakasti kihid

| Kihi tüüp | Näide | Päritolu |
| --- | --- | --- |
| **RAW (`<INDEX>` indeks)** | `RAW (NDVI Index)` | Üks iga projekti seadetes konfigureeritud indeksi kohta, arvutatakse töötlemise käigus |
| **`<INDEX>` LUT** | `NDVI LUT` | Indeksi värvikaardistatud versioon |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Üks iga [indeks/LUT Sandbox](index-lut-sandbox.md) eksporditsükkel |

Kui sama indeksinimi on konfigureeritud rohkem kui üks kord erinevate seadistustega, lisatakse teisele ja järgnevatele nimedesse number (`RAW (NDVI2 Index)`), et kihid oleksid eristatavad.

***

## Kihtide valija kasutamine

1. Avage pilt täisekraanil, klõpsates ruudustikus pisipildile
2. Klõpsake vaataja paremas ülemises nurgas asuvat **kihtide rippmenüüd**

3. Valige kiht – pilt uueneb kohe

Rippmenüüs on esimesena **JPG, RAW (originaal), RAW (siht), RAW (peegeldus)** selles järjekorras ning kõik ülejäänud on loetletud nende järel toodete registreerimise järjekorras.

### Kihtide eelistus navigeerimisel

Nuppude **←**/**→** vajutamisel liigutakse järgmisele pildile ja püütakse jääda samale kihile:

1. **Esmalt täpne vaste** — kui järgmisel pildil on sama nimega kiht, kuvatakse see. Just see hoiab teid kihil `RAW (NDVI Index)`, kui läbite kogu komplekti
2. **Seejärel vastavus tüübi järgi** — indeksikiht otsib mis tahes indeksikihti, LUT mis tahes LUT-i, peegeldusvõime kihti peegeldusvõime kihti, sihtkihti sihtkihti, originaali originaali, baasi baasi
3. **Seejärel, ainult ekspordikihtide puhul** — nimi säilitatakse isegi siis, kui kihiloend pole veel ajakohastunud, sest fail on kettal juba olemas. Just see võimaldab teil tooteid üle vaadata, kui tööprotsess neid veel kirjutab
4. **Muul juhul** — esimene kättesaadav kiht, mis on tavaliselt baaspilt

Projekti sidecar-faile `.daq` ja `.csv` jäetakse nooleklahvide abil navigeerimisel vahele, mistõttu piltide vahel liikudes ei jõuta kunagi valgusanduri salvestuseni.

Suum ja nihutamine toimivad ka piltide vahel, mis muudab sama väljapositsiooni enne-pärast võrdluse lihtsaks.

***

## Pikselväärtuste mõistmine kihtide kaupa

[Kursori väärtuste paneel](opening-an-image-full-screen.md#cursor-values) näitab kursori all oleva kanali tegelikku väärtust selles ühikus, milles kiht on salvestatud. Selle veerud muutuvad vastavalt kihile:

| Kiht | Näidatud ühik | Märkused |
| --- | --- | --- |
| Baas (JPG / PNG

/ TIFF

-eelvaade) | DN, 0–255 | Ekraaniväärtused, RGB

-le gamma-korrigeeritud. Ainult visuaalseks kontrollimiseks |
| RAW (originaal) | DN | Anduri töötlemata digitaalarvud. Histogrammi telg näitab sügavust: 255 (8-bitine), 4095 (12-bitine) või 65535 (16-bitine) |
| RAW (debayereeritud) | DN | Lineaarne, kuvamisel venitamata |
| RAW (eelvaade) / valgebalansiga | DN | Kuvamisväärtus – venitatud või gamma-korrigeeritud. Mitte mõõtmiseks |
| RAW (kiirgus) | **W/m²/sr/nm** | Float32 füüsiline kiirgus. DN-veergu pole |
| RAW (peegeldus) | DN **ja %** | Protsent arvutatud selle faili enda skaala alusel — vt allpool |
| Indeks / LUT / sandboxi ekspordid | Indeksväärtus või RGB

-komponendid | Ühekanaliline indeksifail näitab indeksväärtust; värvikaardistatud LUT-fail näitab Red

/Green

/Blue

-komponente |

### Peegeldusvõime: skaala on failipõhine

{% hint style="warning" %}
**„Jaga 65 535-ga” kehtib ainult Survey3

puhul.** LATTICE-i peegeldusvõime salvestatakse teistsugusel skaalal ning kahe jagaja segamine on kõige levinum viis saada peegeldusvõime väärtused, mis on täpselt poole väiksemad sellest, mis nad peaksid olema.
{% endhint %}

| Allikas | DN, mis vastab peegeldusvõimele 1,0 | Tuvastatud |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP-märge `Chloros:PixelScale=32768`, mis on lisatud igasse LATTICE-peegeldusvõime ekspordifaili. 2× varu tähendab, et ρ väärtust üle 1,0 on võimalik esitada, mitte kärpida |
| **Survey3**| **65535** | Puudub Chloros

i XMP-skaalamärge — Survey3

i kalibreerimine kirjutab ρ × dtype-max ja kärbib väärtuse 1,0 juures |

GIS-i ja skriptide jaoks: loe failist välja `Chloros:PixelScale` ja jaga sellega. Kui märget pole, on faili skaala Survey3

(65535). Vaatur, indeks/LUT-liivakast ja indeksi eksport määravad skaala kõik samamoodi kindlaks, seega on kursori juures kuvatav number sama, mida indeksarvutused kasutasid.

Selle skaala peale lisanduv formaadispetsiifiline salvestamine:

* **TIFF

(32-bitine, protsent)** salvestab DN / 65535 ujukomana
* **PNG

(8-bitine)**ja**JPG (8-bitine)** salvestavad DN × 255 / 65535
* **8-bitise pildistuse 8-bitine DN-väärtus** või enne versiooni 1.3.0 salvestatud peegeldusfail on piiratud vahemikuga 0–255 ja sellel puudub skaalamärge. Paneel kuvab nende failide puhul ainult DN-väärtust, protsendiveergu ei kuvata

### Indeksi väärtuste vahemikud

| Indeksi perekond | Tüüpiline vahemik | Näit |
| --- | --- | --- |
| Normaliseeritud erinevus (NDVI

, GNDVI

, NDRE

, ENDVI…) | −1 kuni +1 | Tervislik taimestik tavaliselt 0,4–0,9; paljas pinnas ligikaudu 0; vesi negatiivne |
| Pinnasega korrigeeritud (SAVI

, OSAVI

, MSAVI2…) | ligikaudu −1 kuni +1,5 | Sarnane näit nagu NDVI

, kuid pinnase taust on summutatud |
| Suhe (GRVI

, GCI

, MSR, CIRE…) | piiranguteta ülespoole | Suhted kasvavad piiranguteta, kui nimetaja lainepikkus läheneb nullile |
| EVI

/ LAI

| 0 kuni ~1, 0 kuni ~3,5 | Pilved ja muud küllastunud pikslid viivad mõlemad väärtused vahemikust välja — maskeerige need esmalt |

Vaadake [multispektraalsete indeksite valemeid](../project-settings/multispectral-index-formulas.md), et tutvuda iga eelseadistuse taga oleva täpse valemiga.

***

## Tavalised töövood

### Enne ja pärast võrdlus

1. Valige **RAW (originaal)** ja märkige ära vinjetatsioon ning kalibreerimata väärtused
2. Vaheta **RAW (peegeldus)**-režiimile
3. Võrdle — vinjetatsioon on eemaldatud, väärtused kalibreeritud. Suum ja nihutamine jäävad samaks, nii et vaatad sama maapinda

### Ühe indeksi läbivaatamine kogu pildiseeria ulatuses

1. Avage esimene töödeldud pilt ja valige indeksikiht
2. Vajutage korduvalt **→** — indeksikiht järgib teid pildilt pildile
3. Jälgige samal ajal külgribal olevat histogrammi: kaadrit, mille jaotus hüppab, tasub lähemalt uurida

### Kalibreerimise sihtmärkide kontrollimine

1. Valige sihtmärgi kaadril **RAW (Target)**

2. Veenduge, et sihtmärk on selgelt nähtav ja tuvastatud
3. Liiguge järgmise sihtmärgi kaadrile — sihtmärgi kiht järgneb

### Kontrollige peegeldusväärtuste täpsust

1. Valige **RAW (Reflectance)**

2. Loe paneelis „Cursor Values“ veergu**%** – see on selle faili jaoks juba õigesti skaalitud
3. Kontrolli, kas väärtused vastavad kaadris olevatele teadaolevatele materjalidele: tervel taimestikul on kõrge NIR

-väärtus ja madal punane väärtus; kalibreerimise sihtmärgi väärtus peaks olema lähedane selle avaldatud peegeldusväärtusele

***

## Veaotsing

### Oodatud kiht ei ole rippmenüüs

**Võimalikud põhjused**

* Pilti ei ole kunagi töödeldud — olemas on ainult baaskiht ja `RAW (Original)`
* Toote ekspordilüliti on projekti seadetest märkimata
* Toode ei kehti selle kaamera puhul (kiirgus ja peegeldusvõime RGB

-põhikaameral; mis tahes indeks ühekanalilisel M3M-mono-kaameral)
* Peegelduskalibreerimisel polnud millegi põhjal töötada — puudus `.daq` allapoole suunatud katvus ja puudus kvaliteedikontrolli läbinud sihtmärk kaadris — seega kasutati kaadri puhul tagavaravarianti „Vignette Corrected“ või „Sensor Response“

**Mida teha**

1. Kontrollige töökäigu logi: Chloros

märgib, millal taotletud eksporditoote loomine oli võimatu ja miks
2. Kontrollige tootekohaseid ekspordivalikuid [Projekti seadetes](../project-settings/project-settings.md)
3. Veendu, et tootekaust on olemas projekti väljundpuus
4. Töötle uuesti, kui toode on aktiveeritud

### Kihtide loend tundub aegunud

Chloros

skaneerib projekti tootekatalooge uuesti käivituse käigus ja parandab puuduvad kihtide registreeringud vastavalt sellele, mis tegelikult kettal asub, seega ilmub normaalse ekspordi lõpetanud kiht iseenesest küsitluse tulemustes. Pildilt ära- ja tagasi vahetamine sunnib süsteemi uuesti otsustama.

### Peegeldusväärtused tunduvad olevat poole väiksemad, kui peaksid olema

Peaaegu kindlasti jagate LATTICE-faili arvuga 65535. Kasutage `Chloros:PixelScale` (32768) või vaadake veergu **%**, kus see on juba rakendatud.

### Indeksikiht on olemas, kuid pilt on tühi

Indeks vajab lainepikkusi, mida teie kihil pole – näiteks indeks, mis loeb kolmandat kanalit, on rakendatud ühe- või kahekanalilisele failile. Vahetage mitme lainepikkusega kihile (peegeldusvõime või debayered) või valige indeks, mis sobib kaamera filtri jaoks.

***

## Järgmised sammud

* [**Pildi avamine täisekraanil**](opening-an-image-full-screen.md) — kursori näidud, histogramm ja GSD-regulaator
* [**Indeksi/LUT-i katsetuskeskkond**](index-lut-sandbox.md) — indeksi interaktiivne visualiseerimine ja eksport
* [**Multispektraalsed indeksivalemid**](../project-settings/multispectral-index-formulas.md) — indeksite viited
* [**Töötlemise lõpetamine**](../processing-images-gui/finishing-the-processing.md) — väljundkaustade puu, millele need kihid viitavad
