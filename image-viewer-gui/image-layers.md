# Pildikihtid

Pildivaataja paremas ülemises nurgas asuv **kihtide rippmenüü** võimaldab vahetada vaadeldava pildi kõigi versioonide vahel – alates algpildist kuni iga töödeldud tooteni ja arvutatud indeksipiltideni – ilma vaatajast lahkumata.

## Mis on pildikihtid?

„Kiht” Chloros-is on üks **tulemusfail**, mis on registreeritud ühe algpildi juurde. Importimisel saad algfailid; töötlemine lisab kihi iga tulemuse jaoks, mille töö käigus loodi. Eksporditud failid säilitavad allika failinime – toodet identifitseerib**kaust** ning kihi nimi on Chlorosi märge selle kausta jaoks.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Kihiloend

### Alati olemas

| Kiht | Mis see on |
| --- | --- |
| **JPG**(või**PNG**/**TIFF**) | Pildistamisega kaasas olnud algfail. Survey3 impordib iga `.RAW` kõrvale `.JPG`; LATTICE-salvestused toovad kaasa PNG või TIFF kuvamiselvaate. Märgistatud vastavalt sellele, mis tegelikult imporditi |
| **RAW (originaal)** | Allikas-RAW-kaader, kuvamiseks debayereeritud, ilma korrektsioonideta. Saadaval alates importimise hetkest — ei vaja töötlemist |

LATTICE-pildil, mille alusfailiks **on** selle toorkaader, puudub eraldi aluskanne: `RAW (Original)` hõlmab seda juba.

### Survey3 töötlemistulemused

| Kiht | Salvestatud | Olemas, kui |
| --- | --- | --- |
| **RAW (sihtmärk)** | — | Kaader tuvastati kalibreerimissihtmärki sisaldavana |
| **RAW (peegeldus)** | `Reflectance_Calibrated_Images/` | Selle kaadri peegelduskalibreerimine õnnestus |
| **Vignette Corrected**| `Vignette_Corrected_Images/` | Kaadrit ei olnud võimalik reflektsioonikalibreerida**ja** *vignette-korrektsioon* oli sisse lülitatud |
| **Anduri reaktsioon**| `Sensor_Response_Images/` | Kaadrit ei olnud võimalik peegelduskalibreerida**ja** *vignettkorrektsioon* oli välja lülitatud |
| **Valge tasakaal** | `White_Balanced_Images/` | Valge tasakaaluga toode salvestati |

{% hint style="info" %}
**Vignettkorrektsioon ja anduri tundlikkus on alternatiivid, mitte kunagi mõlemad korraga.** Iga kaameramudeli kohta on ühe töötlusvoo kohta täpselt üks kalibreerimata varutoode ning *Vignettkorrektsiooni* lüliti valib, milline neist see on. Vaata [Projekti seaded](../project-settings/project-settings.md).
{% endhint %}

### LATTICE tasemed

LATTICE salvestab need ühe töötlemisvooru jooksul. Millised neist on olemas, sõltub projektiseadete tootekohastest ekspordilülititest ja sellest, mis kehtib antud kaamera puhul.

| Kiht | Salvestatakse | Kehtib |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB ja multispektraalne |
| **RAW (eelvaade)** | `Preview_Images/` | Multispektraalne (värvide võltsvärviga venitamine) |
| **Valgebalanss** | `Preview_Images/` | RGB põhikaamerad — RGB eelvaade on registreeritud selle nime all, et see oleks kooskõlas sama nimega Survey3 kihiga |
| **RAW (kiirgus)** | `Radiance_Images/` | Ainult multispektraalne |
| **RAW (peegeldus)** | `Reflectance_Calibrated_Images/` | Ainult multispektraalne ja ainult juhul, kui vastav `.daq` allapoole suunatud andmekirje või kvaliteedikontrolli läbinud raamisisene sihtmärk katab kaadri |

RGB peakaameratel puudub ribapõhine radiomeetria, seega jäetakse nende puhul kiirgus ja peegeldusvahe **mittekohaldatavaks** — logis on see märgitud, mitte vaikimisi ebaõnnestumisena.

### Indeks, LUT ja sandbox-kihid

| Kihi muster | Näide | Kust pärineb |
| --- | --- | --- |
| **RAW (`<INDEX>` indeks)** | `RAW (NDVI Index)` | Üks iga projektiseadetes konfigureeritud indeksi kohta, arvutatakse töötlemise käigus |
| **`<INDEX>` LUT** | `NDVI LUT` | Indeksi värvikaardistatud versioon |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Üks iga [indeks/LUT Sandbox](index-lut-sandbox.md) eksporditsükli kohta |

Kui sama indeksinimi on konfigureeritud rohkem kui üks kord erinevate seadistustega, lisatakse teise ja järgnevate nimedele number (`RAW (NDVI2 Index)`), et kihid oleksid eristatavad.

***

## Kihtide valiku kasutamine

1. Avage pilt täisekraanil, klõpsates ruudustikus pisipilti
2. Klõpsake vaataja paremas ülanurgas asuvat **kihtide rippmenüüd**

3. Valige kiht – pilt uueneb kohe

Rippmenüüs kuvatakse esimesena **JPG, RAW (originaal), RAW (siht), RAW (peegeldus)** selles järjekorras ning kõik ülejäänud on loetletud toodete registreerimise järjekorras.

### Kihtide eelistus navigeerimisel

Nuppude **←**/**→** vajutamisel liigutakse järgmisele pildile ja püütakse jääda samale kihile:

1. **Esmalt täpne vaste** – kui järgmisel pildil on sama nimega kiht, kuvatakse see. Just see hoiab sind kihil `RAW (NDVI Index)`, kui läbid kogu komplekti
2. **Seejärel sobib tüübi järgi** — indeksikiht otsib mis tahes indeksikihti, LUT mis tahes LUT-i, peegeldusvõime kihti peegeldusvõime kihti, sihtmärki sihtmärki, originaali originaali, baasi baasi
3. **Seejärel, ainult ekspordikihtide puhul** — nimi säilitatakse isegi siis, kui kihiloend pole veel ajakohastunud, kuna fail on kettal juba olemas. See võimaldab sul tooteid üle vaadata, kui tööprotsess neid veel kirjutab
4. **Muudel juhtudel** — esimene kättesaadav kiht, mis on tavaliselt baaspilt

Projekti sidecar-faile `.daq` ja `.csv` jäetakse nooleklahvide abil navigeerimisel vahele, nii et piltide vahel liikudes ei jõuta kunagi valgussensori salvestuseni.

Suum ja nihutamine toimivad ka piltide vahel, mis muudab sama väljapositsiooni enne-pärast võrdluse lihtsaks.

***

## Pikselväärtuste mõistmine kihtide kaupa

[Kursori väärtuste paneel](opening-an-image-full-screen.md#cursor-values) näitab kursori all oleva kanali tegelikku väärtust selles ühikus, milles kiht on salvestatud. Selle veerud muutuvad vastavalt kihile:

| Kiht | Näidatud ühik | Märkused |
| --- | --- | --- |
| Base (JPG / PNG / TIFF eelvaade) | DN, 0–255 | Ekraaniväärtused, RGB-is gammakorrigeeritud. Ainult visuaalseks vaatamiseks |
| RAW (originaal) | DN | Anduri töötlemata digitaalsed väärtused. Histogrammi telg näitab sügavust: 255 (8-bitine), 4095 (12-bitine) või 65535 (16-bitine) |
| RAW (debayereeritud) | DN | Lineaarne, kuvamisel venitamata |
| RAW (eelvaade) / valgebalansiga | DN | Kuvamisväärtus – venitatud või gamma-korrigeeritud. Mõõtmiseks ei sobi |
| RAW (kiirgus) | **W/m²/sr/nm** | Float32 füüsiline kiirgus. DN-veerg puudub |
| RAW (peegeldus) | DN **ja %** | Protsent arvutatakse selle faili enda skaala alusel — vt allpool |
| Indeks / LUT / sandboxi ekspordid | Indeksväärtus või RGB komponendid | Ühekanaliline indeksifail näitab indeksväärtust; värvikaardistatud LUT-fail näitab komponente Red/Green/Blue |

### Peegeldusvõime: skaala on määratud faili kohta

{% hint style="warning" %}
**„Jaga 65 535-ga” on õige ainult Survey3 puhul.** LATTICE-peegeldusvõime on salvestatud teistsuguses skaalas ning kahe jagaja segamine on kõige levinum viis saada peegeldusvõime väärtused, mis on täpselt poole väiksemad sellest, mis nad peaksid olema.
{% endhint %}

| Allikas | DN, mis vastab peegeldusvõimele 1,0 | Tuvastatud |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP-märge `Chloros:PixelScale=32768`, mis on lisatud igasse LATTICE-peegeldusvõime ekspordifaili. 2× varu tähendab, et ρ väärtusi üle 1,0 on võimalik esitada, mitte kärpida |
| **Survey3**|**65535** | Kui XMP-skaalamärgist Chloros puudub, kirjutab kalibreerimine ρ × dtype-max ja kärbib väärtuse 1,0 juures |

GIS-i ja skriptimise jaoks: loe failist `Chloros:PixelScale` ja jaga sellega. Kui märget pole, on fail Survey3-skaalaga (65535). Vaatur, indeks/LUT-liivakast ja indeksi eksport määravad skaala kõik samal viisil, seega on kursori juures kuvatav number sama, mida indeksarvutused kasutasid.

Selle skaala peal olev formaadispetsiifiline salvestus:

* **TIFF (32-bitine, protsent)** salvestab DN / 65535 ujukomaarvuna
* **PNG (8-bitine)**ja**JPG (8-bitine)** salvestavad DN × 255 / 65535
* **8-bitise allika salvestuse 8-bitine TIFF-eksport** ei ole ümberskaalatud, vaid piiratud vahemikuga 0–255, ning sellel puudub tahtlikult skaalamärge. Paneel kuvab nende failide puhul ainult DN-väärtust, protsendiveergu ei näidata

### Indeksiväärtuste vahemikud

| Indeksiperekond | Tüüpiline vahemik | Lugem |
| --- | --- | --- |
| Normaliseeritud erinevus (NDVI, GNDVI, NDRE, ENDVI…) | −1 kuni +1 | Tervisel taimestikul tavaliselt 0,4–0,9; paljas pinnas ligikaudu 0; vee puhul negatiivne |
| Pinnasega korrigeeritud (SAVI, OSAVI, MSAVI2…) | ligikaudu −1 kuni +1,5 | Sarnane näit nagu NDVI, kuid mulla taust on summutatud |
| Suhe (GRVI, GCI, MSR, CIRE…) | piiranguteta ülespoole | Suhted kasvavad piiramatult, kui nimetaja väärtus läheneb nullile |
| EVI / LAI | 0 kuni ~1, 0 kuni ~3,5 | Pilved ja muud küllastunud pikslid viivad mõlemad väärtused vahemikust välja — maskeerige need esmalt |

Vaata [multispektraalsete indeksite valemeid](../project-settings/multispectral-index-formulas.md), et näha iga eelseadistuse täpset valemit.

***

## Tavalised töövood

### Enne ja pärast võrdlus

1. Valige **RAW (originaal)** ja märkige ära vinjetatsioon ning kalibreerimata väärtused
2. Vahetage **RAW (peegeldus)**-režiimile
3. Võrrelge — vinjetatsioon on eemaldatud, väärtused kalibreeritud. Suurendamine ja nihutamine jäävad samaks, nii et vaatate sama maapinda

### Ühe indeksi läbivaatamine kogu komplekti ulatuses

1. Avage esimene töödeldud pilt ja valige indeksikiht
2. Vajutage korduvalt nuppu **→** — indeksikiht järgib teid pildilt pildile
3. Jälgige käigus külgribal asuvat histogrammi: kaader, mille jaotus hüppab, väärib lähemat vaatamist

### Kalibreerimise sihtmärkide kontrollimine

1. Valige sihtmärgi kaadril **RAW (Target)**

2. Veenduge, et sihtmärk on selgelt nähtav ja tuvastatud
3. Liiguge järgmise sihtmärgi kaadrile — sihtmärgi kiht järgneb

### Kontrollige peegeldusväärtuste täpsust

1. Valige **RAW (Reflectance)**

2. Loe paneelis „Cursor Values“ veergu**%** — see on selle faili jaoks juba õigesti skaalatud
3. Kontrolli tulemuste õigsust raamis olevate teadaolevate materjalide põhjal: tervel taimestikul on kõrge NIR-väärtus ja madal punane väärtus; kalibreerimismärgi väärtus peaks olema lähedane selle avaldatud peegeldusväärtusele

***

## Veaotsing

### Oodatud kiht puudub rippmenüüst

**Võimalikud põhjused**

* Pilti pole kunagi töödeldud — olemas on ainult baaskiht ja `RAW (Original)`
* Projekti seadetes ei ole toote ekspordikastike märgitud
* Toode ei kehti selle kaamera puhul (kiirgus ja peegeldusvõime RGB-masteril; mis tahes indeks ühekanalilisel M3M-mono-kaameral)
* Peegeldusvõime kalibreerimiseks puudusid andmed — puudus `.daq` allapoole suunatud katvus ja puudus kvaliteedikontrolli läbinud sihtmärk kaadris — seega kasutati kaadri puhul tagavaravõimalust „Vignette Corrected” või „Sensor Response”

**Mida teha**

1. Kontrollige töölogi: Chloros näitab, millal taotletud eksporditoode oli võimatu ja miks
2. Kontrollige tootekohaseid ekspordilüliteid [Projekti seaded](../project-settings/project-settings.md)
3. Veendu, et tootekausta on olemas projekti väljundpuus
4. Töötle uuesti, kui toode on aktiveeritud

### Kihtide nimekiri tundub aegunud

Chloros skannib projekti tootekatalooge uuesti käivituse käigus ja parandab puuduvad kihtide registreeringud vastavalt sellele, mis tegelikult kettal on, seega ilmub normaalse ekspordi lõpetanud kiht iseenesest küsitluse tulemustes. Pildi vahepealne sulgemine ja tagasi avamine sunnib süsteemi uuesti otsustama.

### Peegeldusväärtused tunduvad olevat poole väiksemad, kui peaksid olema

Peaaegu kindlasti jagate LATTICE-faili arvuga 65535. Kasutage `Chloros:PixelScale` (32768) või vaadake veergu **%**, kus see on juba rakendatud.

### Indeksikiht on olemas, kuid pilt on tühi

Indeks vajab lainealasid, mida teie kihil pole – näiteks indeks, mis loeb kolmandat kanalit, mida on rakendatud ühe- või kahekanalilisele failile. Vahetage mitme laineala kihi vastu (peegeldusvõime või debayered) või valige indeks, mis sobib kaamera filtri jaoks.

***

## Järgmised sammud

* [**Pildi avamine täisekraanil**](opening-an-image-full-screen.md) – kursori näidud, histogramm ja GSD-juhtimine
* [**Indeksi/LUT-i katsetuskeskkond**](index-lut-sandbox.md) — indeksite interaktiivne visualiseerimine ja eksport
* [**Multispektraalsete indeksite valemid**](../project-settings/multispectral-index-formulas.md) — indeksite viited
* [**Töötlemise lõpetamine**](../processing-images-gui/finishing-the-processing.md) — väljundkaustade puu, millele need kihid viitavad
