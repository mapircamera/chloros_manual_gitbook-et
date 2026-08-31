# Pildi avamine täisekraanil

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Täisekraanil avatud pilt, mille paremas ülanurgas asub kihtide valija</p></figcaption></figure>

Chloros pildivaataja on täisekraaniline liides piltide vaatamiseks, uurimiseks ja mõõtmiseks. Siin saate lugeda **tegelikke pikselväärtusi** — kanali kohta DN, peegeldusprotsenti või kiirgust W/m²/sr/nm — mitte ekraanil kuvatavat venitatud eelvaadet.

## Pildivaataja avamine

### Failibrauserist

1. Avage vahekaart **Failibrauser** (<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">)
2. Klõpsake mis tahes **pisipilti** [pildirestis](image-grid.md)
3. Pilt avaneb täisekraanil vahekaardil **Pildivaataja**

Pilt avatakse selles tootes, mida raster parasjagu näitas. Kui raster on seatud `RAW (Reflectance)`-ile, siis on see kiht, millele jõuate.

### Pildivaataja külgriba avamine

Klõpsake vasakul külgribal ikooni **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">, et avada analüüsipaneel. See sisaldab ülevalt alla järgmist:

* pildi nimi ja kaamera mudel
* nupp **Ekspordi/salvesta pilt(eid)** (ainult siis, kui indeks või LUT on aktiivne)
* valikukastid **Indeks**ja**LUT** ning indeksi konfiguratsioonipaneel — vaata [Indeksi/LUT-i katsetuskeskus](index-lut-sandbox.md)
* paneel **Kursori väärtused**: kanalipõhine väljund, kihi histogramm ja GSD-regulaator***

## Navigeerimine ja suumimine

### Piltide sirvimine

* **Järgmine pilt**: nupp → või klahv**→** (parem nool)
* **Eelmine pilt**: nupp ← või klahv**←** (vasak nool)
* **Hüppa konkreetsele pildile**: naase võrgustikule ja klõpsa selle pisipildil

Suurendus ja nihutamine säilivad piltide vahel liikudes, nii et saad pildiseeria läbi vaadata, jäädes samal ajal raami samasse osasse.

### Suurendamine

Suurendamist juhitakse **hiirerattaga** 15% sammuga, kusjuures suurendus on seotud kursoriga – kursori all olev punkt jääb kursori alla. Suurendamisvahemik on piiratud pildi ja akna suurusega: sa ei saa vähendada pilti väiksemaks kui „aknasse mahutamine” ning ülemine piir on määratud pildi algse resolutsiooniga.

Täisekraanivaatajas puuduvad spetsiaalsed suumimisnupud. (Ristikuvaates muudavad **Ctrl + `+` / `−`** pisipiltide suurust – see on teistsugune juhtimine.)

### Paanimine suurendatud vaates

Klõpsake pildi peal vasaku hiirenupuga, hoidke seda all ja lohistage. Paanimine on piiratud, nii et pilti ei saa ekraanilt välja lohistada.

### Pikslipõhine vaatlus suurel suumil

Kui efektiivne suurendus ületab **60×**, joonistab Chloros kursori all oleva üksiku kuvatava pikseli ümber esiletõstetud raami ja selle kõrvale ujukomaarvulise väärtuse.

„Tegelik” suurendus arvestab GSD-ploki suurust: ploki suurusega 8 ilmub esiletõstmine juba 7,5× suurenduse juures, mitte 60× juures, sest üks kuvatav piksel on juba 8 × 8 allikapikslit. Kui suurendust vähendada allapoole künnist, kaob esiletõstmine.

### Klahvikombinatsioonid

| Klahv                             | Asukoht       | Tegevus                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Täisekraan | Järgmine pilt                          |
| **←**                           | Täisekraan | Eelmine pilt                      |
| **Ctrl + R**                    | Täisekraan | Indeksi/LUT-liivakasti tühjendamine         |
| **Ctrl + `+`**/**Ctrl + `=`** | Võrgustik        | Suuremad pisipildid (4 pikselit vajutuse kohta)  |
| **Ctrl + `−`**                  | Raster        | Väiksemad pisipildid (4 pikselit iga vajutuse kohta) |***

## Kursori väärtused

Liiguta kursorit pildi kohale ja paneelis **Kursori väärtused** kuvatakse selle all oleva iga kanali väärtus.

{% hint style="success" %}
**Need on faili tegelikud numbrid.** Ekraanil olev pilt on 8-bitine venitatud eelvaade ja ei suuda neid väärtusi kuvada, seega võtab Chloros lugemiseks proove tegelikust tootefailist. Seetõttu näitab 12-bitine toorkaader näitab väärtusi üle 255 ja miks float32-kiirguskiht näitab füüsikalisi ühikuid.
{% endhint %}

### Veergude tähendus

Paneel kohandub vastavalt vaadeldavale kihile:

| Vaadeldav kiht              | Näidatavad veerud    | Märkused                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Peegeldusvõime                        | **DN**ja**%** | Protsent arvutatakse selle faili enda skaala alusel – vt allpool                                      |
| Kiirgus                           | **W/m²/sr/nm**   | Ujukomaga füüsikalised väärtused; DN-veergu pole, kuna DN on siin tähendusetu                           |
| Toores / Debayered / eelvaade / JPG    | **DN**           | Täisarvulised digitaalsed numbrid                                                                         |
| 32-bitise protsendilise peegeldusvõime eksport | ainult **%**       | Salvestatud ujukomaarv ei ole DN, seega ümardamine täisarvuks tooks kaasa mõttetud väljundid nagu `0` või `1` |

Iga rida on märgistatud teie kaamera filtri kanalinimega — `Red / Green / NIR` vastab RGN-ile, `Orange / Cyan / NIR` vastab OCN-ile, `NIR / Green / Blue` NGB puhul, `Red / Green / Blue` kaamera RGB puhul ning ühe kanali nimi kaamerate RE, NIR ja mono-M3M-kaamerate puhul. Igal sildil on värviline punkt, mis vastab indeksivalemi redaktoris kasutatavatele kanaliringidele.

Salvestatud **indeks- ja LUT-**pildid on erijuhtum: need sisaldavad pigem värvikaardi komponente kui spektraalribasid, seega on nende read märgistatud `Red / Green / Blue` (või `Index` ühekanalilise indeksifaili puhul) asemel kaamera filternimede asemel.

Kui indeks on liivakastis aktiivne, ilmub kanalite alla lisanrida, mis näitab kursori asukohas olevat **indeksiväärtust** koos indeksi nimega ja valge punktiga, mis vastab selle märgile histogrammil.

### Peegeldusprotsent kasutab iga faili enda skaalat

{% hint style="warning" %}
**Ärge eeldage, et 65535 = 100%.** Chloros salvestab peegeldusvõime erinevates skaalades sõltuvalt sellest, milline kaamera selle genereeris, ning vaataja määrab iga faili puhul õige skaala kindlaks.
{% endhint %}

| Allikas                  | DN, mis vastab peegeldusvõimele 1,0 | Kuidas seda tuvastatakse                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP-märge `Chloros:PixelScale=32768` kirjutatakse igasse LATTICE peegeldusvõime eksporti. 2× varu võimaldab failil kanda ρ väärtust üle 1,0 ilma kärpimiseta |
| **Survey3**|**65535**                      | XMP-skaala sildit Chloros ei ole — kalibreerimine kirjutab ρ × dtype-max ja kärbib väärtuse 1,0 juures                                                               |

Vaatur, indeks/LUT-liivakast ja indeksi eksport määravad skaala kindlaks ühe ja sama rakenduse kaudu, seega on kursori juures loetav väärtus sama, mida indeksarvutused kasutasid.

Kaks olulist tagajärge:

* **32-bitine protsent**TIFF salvestab DN/65535 ujukomana ja**8-bitine** PNG/JPG-eksport salvestab DN × 255/65535 — vaataja teisendab mõlemad tagasi enne protsendi kuvamist.
* Üht juhtumit ei ole võimalik taastada: **8-bitise allika salvestuse 8-bitine TIFF-eksport** kärbitakse vahemikku 0–255, mitte ei skaalata ümber, ning sellel puudub tahtlikult skaalimärge. Nende failide puhul kuvab paneel ainult DN-väärtusi, protsendikolonn puudub. See on aus vastus, mitte viga.***

## Kihtide histogramm

Kursori ridade all asub vaadeldava kihi reaalajas histogramm, mis on jagatud **256 vahemikku**. Vaikimisi joonistatakse üks kombineeritud kõver, kaalutud `(R + 2G + B) / 4` — sama mõõtmisruum, mida kasutavad LATTICE-kaamera histogrammid. Kui lülitada sisse**RGB** sisse, asendatakse see kanalipõhiste kõveratega kanalite värvides, mis on additiivselt segatud, et kattuvad osad jääksid loetavaks. Monokroomkihtidel joonistatakse alati üksainus kõver.

Horisontaaltelg on kihti iseloomustavas ühikus:

| Kiht       | Telje ühik  | Telje maksimum                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Peegeldusvõime | protsent    | 125% — toote varu võimaldab ρ väärtust üle 1,0           |
| Kiirgusvõimsus | W/m²/sr/nm | Kaadri enda tippväärtus, ümardatud kahe olulise numbrini |
| 8-bitised andmed | DN         | 255                                                        |
| 12-bitised andmed | DN         | 4095                                                       |
| 16-bitised andmed | DN         | 65535                                                      |

Kui telg on DN-režiimis ja jääb ühe neist kolmest ülempiirist, teab Chloros ka selle pildi bittisügavust, mida vaatate.

Histogrammi kohal asuvad kolm nuppu:

| Nupp     | Vaikimisi | Mõju                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **KURSOR** | Sisse      | Joonistab histogrammile märgistusjooned täpselt ülaltoodud ridades näidatud väärtustele, nii et saate näha, kus kursori all olev piksel raami jaotuses asub. Režiimis RGB on igal kanalil üks oma värvi märgistusjoon; muul juhul on üks valge märgistusjoon kombineeritud väärtuse juures |
| **INDEX**| Sisse      | Ilmub ainult siis, kui indeks on aktiivne. Lülitab histogrammi allikakanalitelt üle**indeksväärtuste jaotusele**, kus kaks klippimiskünnist on joonistatud oranžide katkendjoontena ja kursori indeksväärtus valge joonena                                                          |
| **RGB**| Väljas     | Lülitab ümber kombineeritud kõveralt kanalipõhistele kõveratele. Monosensoril on selle nupu tekstiks**MONO** ja see on keelatud — kuvada on vaid üks kanal                                                                                                                                  |

Histogramm arvutatakse **nähtavate plokkide** põhjal, mitte nende taga olevate allikapikslite põhjal: GSD-ploki suuruse muutmisel arvutatakse jaotus uuesti, nii et histogramm, kursori märk ja kuvatav pilt on alati omavahel kooskõlas.***

## GSD-ploki suurus

Paneeli allosas asub **GSD (px)**-regulaator: numbrikast, liugur vahemikus**1 kuni 256**ja**RESET**-nupp.

See muudab _kuvatava_ pildi jämedamaks, arvutades N × N allikapikslite ploki keskmise üheks kuvatavaks piksliks. `1` on algne eraldusvõime.

* See mõjutab **täisekraanivaadet, võrgustiku pisipilte, kursori näitu ja mõlemat histogrammi** – kõik, mis kuvab pilti, kasutab sama põhilahutusvõimet.
* See mõjutab **ainult kuvamist**. Töötlemine ja eksport jäävad muutmata. Üks erand on tahtlik: [Index/LUT Sandbox](index-lut-sandbox.md) eksport salvestab selle, mida te parasjagu vaatate, seega säilitab see praeguse plokisuuruse, ja ekspordipaneel hoiatab teid, kui plokisuurus ületab 1.
* Väärtus salvestatakse **projekti kaupa** nimega `viewer_display.gsd_bin` failis `project.json`, seega säilib see ka programmi sulgemisel ja uuesti avamisel.
* Kui ploki suurus on suurem kui 1, näitab kursori näidik ploki väärtust, mitte allika pikslit – kuvatav väärtus on kursori all oleva ploki keskmine väärtus.

{% hint style="info" %}
**Miks „ploki suurus” ja mitte sentimeetrid pikseli kohta?** Cm/px näitaja eeldab kõrgust maapinnast. Ühe kaadri EXIF-andmed sisaldavad GPS-kõrgust keskmise merepinna suhtes, mitte selle maastiku suhtes, millele kaamera oli suunatud, seega Chloros ei prindi maapinnast kaugust, mida ta ausalt öeldes ei saa tuletada. Ploki suurus allikapikslites on sama varuvariant, mida MAPIR pilvetöötlusvahendid kasutavad, kui maapinna proovivahe on teadmata.
{% endhint %}

***

## Vaadeldavad pilditüübid

Vaateakna ülemises paremas nurgas asuvas kihtide rippmenüüs on loetletud kõik praeguse pildi versioonid. Millised kanded kuvatakse, sõltub kaamerast ja sellest, mida on töödeldud – vaata [Pildikihtidest](image-layers.md), et näha täielikku nimekirja ja rippmenüü toimimist.

### Survey3

* **JPG** — kaamera enda eelvaatefail
* **RAW (originaal)** — allikas `.RAW`, kuvamiseks debayereeritud, ilma parandusteta
* **RAW (sihtmärk)** — kaader, mis on identifitseeritud kalibreerimisihtmärki sisaldavana
* **RAW (peegeldusvõime)** — kalibreeritud peegeldusvõime tulemus (65535 = ρ 1,0)
* **Vignette Corrected**/**Sensor Response** — kalibreerimata varutulemus
* **Valgebalansiga** — valgebalansiga töödeldud pilt
* **RAW (`<INDEX>` indeks)**ja**`<INDEX>` LUT** — arvutatud indeksipildid

### LATTICE

LATTICE-pildistamisel kasutatakse sama rippmenüüd, milles on esitatud töötlemisahela tasandite nimed:

| Kiht                 | Sisu                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (originaal)**    | Salvestatud algne RAW-kaader                                     |
| **RAW (debayered)**   | Lineaarne debayered-pilt                                           |
| **RAW (eelvaade)**     | Ekraanil kuvatav eelvaade — multispektraalsete kaamerate jaoks värvide võltsvärviga venitus |
| **Valgebalanss**    | RGB-põhikaamerate eelvaade (valgebalanss + gamma)   |
| **RAW (kiirgus)**    | Float32 spektraalne kiirgus ühikutes W/m²/sr/nm                              |
| **RAW (peegeldusvõime)** | uint16 peegeldusvõime, 32768 = ρ 1,0                                    |

Kiirgusvõimsus ja peegeldusvõime on saadaval ainult multispektraalsete kaamerate puhul: RGB põhikaameral puudub ribapõhine radiomeetria, seega neid kihte selle jaoks ei genereerita.

***

## Indeksi ja LUT-i rakendamine

Rakendage multispektraalseid indekseid ja värvilisi Look-Up Tabeleid (LUT) külgribalt:

1. Avage külgribal **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Märkige **Indeks**

3. Valige oma kaamera filter ja indeksivalem, seejärel lohistage kanaliringid valemi lahtritesse
4. Lisage LUT ja valige gradient, künnisväärtused ja kärpimisrežiim
5. Lugege väärtused kursori juures ja salvestage tulemus valikuga **Ekspordi/Salvesta pilt(id)**Täieliku juhendi leiate [Indeksi/LUT-i katsetuskastist](index-lut-sandbox.md).***

## Veaotsing

### Pilt ei avane

**Võimalikud põhjused**: faili on pärast importimist teisaldatud või kustutatud; toodet ei salvestatud kunagi; väga suure pildi jaoks pole piisavalt mälu.**Mida teha**:

1. Kontrollige, kas kihi fail on endiselt olemas projekti väljundpuus
2. Avage fail välises vaatajas, et veenduda, et see on terviklik
3. Sulgege teised rakendused, et vabastada mälu

### Pilt on must, valge või äärmiselt värvikas

**Võimalikud põhjused**: kuvari venitusel pole millegagi töötada (peaaegu konstantne kaader); float32-kiht ebatavaliste väärtustega; indeks, mis ei andnud kehtivaid andmeid.**Mida teha**:

1. Loe kursori väärtused – kui iga kanal on nullis või selle lähedal, on probleem andmetes, mitte kuvamisel
2. Kontrolli histogrammi: üksik piik ühes otsas näitab, et kaader on kärbitud või tühi
3. Kontrolli töötlemislogi selle käivituse kohta, mis kihti genereeris

### Väärtused tunduvad valed

**Võimalikud põhjused**: olete teisel kihil, kui arvate; võrdlete protsenti toore DN-iga; võrdlete LATTICE-faili Survey3-failiga, kasutades sama jagajat.**Mida teha**:

1. Kontrollige rippmenüüst valitud kihti – paneeli ühikud järgivad kihti
2. Peegeldusvõime puhul kasutage **%**-veergu, selle asemel et jagada DN-väärtust ise; kui pead jagama, kasuta selle faili `Chloros:PixelScale`-väärtust (32768 LATTICE-faili puhul, puudumine tähendab 65535 Survey3-faili puhul)
3. Seadke GSD-ploki suurus tagasi väärtusele 1 — kui see on suurem kui 1, loete ploki keskmist, mitte pikslit
4. Kontrollige, kas selle kaadri jaoks tegelikult teostati peegeldusvõime kalibreerimine; kalibreerimata varutoode (Sensor Response / Vignette Corrected) ei ole peegeldusvõime

***

## Järgmised sammud

* [**Pildikihtid**](image-layers.md) — iga kihi nimi (kui see on olemas) ja selle väärtuste tähendus
* [**Indeksi/LUT-katsetuskeskkond**](index-lut-sandbox.md) — indeksite visualiseerimise loomine, häälestamine ja eksportimine
* [**Kaardimärgid**](map-markers.md) — sama pildikogum kaardil
* [**Multispektraalsete indeksite valemid**](../project-settings/multispectral-index-formulas.md) — indeksite viited

Töötlemise töövoo kohta vaata [Piltide töötlemine (GUI)](../processing-images-gui/adding-files-to-a-project.md).
