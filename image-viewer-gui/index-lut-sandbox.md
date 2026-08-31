# Indeks/LUT-katsekeskkond

Indeks/LUT-katsekeskkond on interaktiivne töökeskkond Chloros pildivaataja külgribal. Valige valem, seostage sellega oma kaamera kanalid, värvige see gradiendiga ja reguleerige väärtuste vahemikku – pilt uueneb reaalajas, kui te seda teete. Alates versioonist 1.2.0 saate ka **oma loodud seadistuse salvestada**, kas ühe pildi või kogu projekti jaoks, ilma et peaksite seda uuesti töötlema.

## Milleks Sandboxi kasutatakse

| Index/LUT Sandbox (interaktiivne)        | Projekti töötlemine (partii)       |
| -------------------------------------- | -------------------------------- |
| Üks pilt korraga, kohene tagasiside  | Kogu andmekogum ühe käivituse jooksul     |
| Eksperimentaalne ja iteratiivne             | Eelkonfigureeritud seaded          |
| Renderdab reaalajas; salvestab ainult teie soovi korral  | Kirjutab alati tulemusfailid      |
| Ideaalne õigete seadistuste leidmiseks | Parim tulemus saavutatakse, kui seadistused on lõplikud |

{% hint style="success" %}
**Tavaline töövoog**: häälestage Sandboxis, kuni visualiseerimine vastab teie soovidele, seejärel eksportige otse Sandboxist või kopeerige samad indeks- ja LUT-seaded [Projekti seadistustesse](../project-settings/project-settings.md), et järgmine töötlemistsükkel rakendaks neid igale pildile.
{% endhint %}

***

## Sandboxi avamine

1. Klõpsa võrgustikus pildil — see avaneb täisekraanil **Pildivaataja** vahekaardil „<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">“
2. Klõpsa **Pildivaataja** ikoonil „<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">“, et avada vasak külgriba, kui see pole veel avatud
3. Valige paremas ülemises nurgas asuvast kihtide rippmenüüst mitme sagedusribaga kiht — tavaliselt valitakse **RAW (peegeldus)**, sest kalibreeritud peegelduse alusel arvutatud indeksväärtused on piltide vahel võrreldavad

Külgribal kuvatakse ülalt alla järgmised andmed:

* pildi nimi ja kaamera mudel
* nupp **Ekspordi/Salvesta pilt(id)** — ilmub, kui valik „Indeks“ või „LUT“ on märgitud
* valikukastid **Indeks**ja**LUT**
* indeksi konfiguratsioonipaneel
* paneel **Kursori väärtused**, millel on näidud, histogramm ja GSD-regulaator

{% hint style="warning" %}
**Ei ole saadaval monokromaatiliste kaamerate puhul.** Ühe lainepikkusega LATTICE M3M-pildil on mõlemad valikukastid keelatud ning neile kuvatakse abiteade _„Ei ole saadaval monokromaatiliste (M3M) andurite puhul“_ — ühe sagedusriba puhul on mitmesageduslik indeks määratlemata. M3M-kaamerate indeksite arvutamiseks kombineerige kaks või enam pilti joondatud mitmesageduslikuks pildistapliks ja kasutage LATTICE indeksimootorit.
{% endhint %}

***

## Indeksi rakendamine

1. Märkige külgriba ülaosas asuv ruut **Index**

2. Valige vasakpoolsest rippmenüüst oma kaamera filter (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Valige parempoolsest rippmenüüst indeksivalem – 27 sisseehitatud valemit ning lisaks kõik teie salvestatud kohandatud valemid
4. Valem kuvatakse allpool matemaatilise valemina, kus iga sagedusriba kohal on tühi ring. **Lohistage värviline kanaliring sagedusriba kohale**, et see seostada
5. Kui kõik valemi poolt kasutatavad pesad on seotud, uueneb pilt ja kuvab indeksväärtused
6. Viige kursor pildi kohale, et väärtusi lugeda; paneel **Kursori väärtused** lisab indeksirea, milles on kursori all olev väärtus

Tühjenda seotud pesa topeltklõpsuga. Poolik valem on normaalne seisund lohistamise ajal, mitte viga – pilt lihtsalt ei uuene enne, kui valem on täielik.

Kanalite ringid on värvikoodiga: punane = Red, roheline = Green, sinine = Blue, oranž = Orange, tsüaan = Cyan, lilla = NIR, magenta = RE. Samu värve kasutatakse ka paneelis „Kursori väärtused” kanalite punktide ja histogrammi kõverate puhul.

### NDVI näide

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Täieliku valemite viite – kõik kolm eelseadistatud nimekirja ja millised nimed kus töötavad – leiate [Multispektraalsete indeksite valemid](../project-settings/multispectral-index-formulas.md).

### Kui indeks on märgitud, kuid LUT puudub

Pilt joonistatakse **halltoonides**, venitatuna kahe künnisväärtuse vahele. See on taotluslik: indeksipilt on skalaarandmed ja halltoonid on selle tõeline esitusviis. Lisage LUT, kui soovite värvi.***

## Töö LUT-idega (otsingutabelid)**Look-Up Table** seostab indeksiväärtused värvidega: sisendiks on NDVI 0,65, väljundiks konkreetne roheline värv. See ei muuda andmeid – see muudab seda, kuidas neid loetakse.

### LUT-i lisamine

1. Klõpsa valemi all asuvat **„+ Lisa LUT“** nuppu <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">
2. Vali värvigradient
3. Määra lõikamise miinimum- ja maksimumväärtus
4. Valige kärpimisrežiim
5. Märkige külgribal **LUT**-ruut, et seda renderdada

LUT-ruut jääb keelatuks, kuni indeksile on tegelikult LUT konfigureeritud.

### Värvigradiendi valimine

Viige hiir **gradiendiriba**kohale, et avada eelseadete loend — Chloros sisaldab**seitset** gradiendi eelseadet:

| # | Gradient                            | Kuju                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Kollane → Green (**vaikimisi**)  | Hajuv — vastab tavapärasele taimestiku intuitsioonile: roheline = terve |
| 2 | Lilla → Kollane → Green             | Hajuv, selge madala otsaga                                  |
| 3 | Pruun → Valge → Blue                | Hajuv, heleda keskpunktiga                                   |
| 4 | Must → Lilla → Roosa → Hele kollane | Järjestikune, tumedast heledaks                                           |
| 5 | Red → Kollane → Blue                 | Erinev, heleda keskpunkti ümber                                   |
| 6 | Lilla → Blue → Green → Kollane      | Järjestikune, tumedast heledaks                                           |
| 7 | Orange → Valge → Lilla             | Hajuvad heleda keskpunkti ümber                                   |

**Hargnev**gradient paigutab neutraalse värvi akna keskele, mis on hästi loetav, kui keskpunktil on konkreetne tähendus (lävi, algkuupäev).**Järkjärguline** gradient kulgeb monotoonselt tumedast heledani, mis on hästi loetav suuruse puhul, millel on ainult „rohkem“ ja „vähem“.

Igal eelseadistusel on seitse värvipeatust. Klõpsake eelseadistust ja pilt uueneb kohe (kui LUT-ruut on märgitud).

### Värvipeatuste muutmine

Gradiendiriba all on rida värviproove, üks iga peatuse kohta:

* **Värvi muutmine**: klõpsake värviproovi, et avada värvivalija (värviratas, RGB/HSV liugurid või kuuekohaline värvikood, näiteks `#FF0000`)
* **Peatuse lisamine**: klõpsa rea lõpus olevale**+**-nupule – lisatakse valge peatuse
* **Peatuse eemaldamine**:**topeltklõpsa** värviproovile
* **Salvesta muudetud gradient**: klõpsa gradientiriba kõrval olevale salvestusikoonile, et lisada muudetud gradient eelseadete nimekirja, et saaksid seda uuesti valida

Indeksile seadistatud gradient salvestatakse koos selle indeksiga projekti seadistustesse, seega säilib see ka projekti sulgemisel ja uuesti avamisel.

**Vähem vahepunkte**tekitavad selgepiirilisi tsoone, mis on loetavad klassifikatsioonina;**rohkem vahepunkte** tekitavad sujuvad, peaaegu fotorealistlikud üleminekud. Kolm kuni viis vahepunkti sobivad esitlusdiapositiividele ja klassifikatsioonikaartidele; kuus kuni kümme sobivad üldiseks analüüsiks; viisteist või rohkem sobivad üksikasjalikuks uurimiseks ja avaldatavate jooniste jaoks.

### Väärtuste vahemiku seadistamine

Künnisregulaator on **kahe käepidemega liugur**vahemikus −1 kuni +1, mille mõlemas otsas on täpsete väärtuste sisestamiseks muudetav tekstikast ning**AUTO**-nupp.

* Lohistage kumbagi käepidet või sisestage number vastavasse lahtrisse ja vajutage Enter
* **AUTO**seab vahemiku pildi kehtivate indeksväärtuste**

2. ja 98. protsentiili** vahele — see on hea lähtepunkt, mis jätab kõrvalekalded arvesse võtmata. Chloros ümardab tulemuse adaptiivselt: väga kitsa vahemiku puhul 4 kümnendkohani, kitsa vahemiku puhul 3 kümnendkohani, muudel juhtudel 2 kümnendkohani
* Kõik käsitsi tehtud kohandused on AUTO-režiimist ülimuslikud, kuni vajutad uuesti nuppu AUTO

Näide NDVI akendest:

| Eesmärk                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Näita kõike                         | −1,0 | 1,0 |
| Ainult taimestik, välja arvatud pinnas ja vesi | 0,2  | 0,9 |
| Ainult terve taimestik                 | 0,5  | 0,9 |
| Rõhuta stressi                        | 0,2  | 0,5 |

Vahemiku kitsendamine suurendab kontrasti huvipakkuval alal ja lükkab kõik muu vahemikust välja — kusjuures **lõikamisrežiim** otsustab, mis sellega juhtub.***

## Lõikamisrežiimid

Kui pikseli indeksväärtus jääb miinimum-/maksimumvahemikust väljapoole, otsustab lõikamisrežiim, kuidas see joonistatakse.

| Rippmenüü silt                  | Salvestatud väärtus      | Vahemikust väljapoole jäävad pikslid joonistatakse järgmiselt                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum ja maksimum** (vaikimisi) | `clip`            | Gradiendi lähim lõppvärv — väärtused, mis jäävad alla miinimumi, võtavad esimese värvi; väärtused, mis jäävad üle maksimumi, võtavad viimase |
| **Läbipaistev taust**      | `transparent`     | Täielikult läbipaistev (reaalne alfa)                                                                                                  |
| **Indeksitaust**| `indexColor`      | Halltoonid, mis ulatuvad pildi**kogu** indeksivahemikku, nii et vahemikust väljas olev struktuur on endiselt hallina nähtav                |
| **Algne taust**         | `backgroundColor` | Aluseks olev pilt ise, nii et värvikiht asub tegeliku stseeni peal                                                |

| Režiim                       | Sobib kõige paremini                               | Välimus                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimaalne ja maksimaalne**      | Andmete täielik kuvamine, teaduslik analüüs | Iga piksel on värvitud                      |
| **Läbipaistev taust** | GIS-kihtide pealekandmine, väärtuste vahemiku eraldamine   | Värv akna sees, väljaspool midagi |
| **Indeksitaust**       | Rõhutamine, säilitades andmete konteksti    | Värv sees, hall väljaspool               |
| **Algne taust**    | Aruanded ja esitlused              | Värv sees, foto väljaspool         |

{% hint style="info" %}
**Andmeteta pikslid on alati läbipaistvad, igas režiimis.** Pikslit, mille indeks ei ole piiratud (jagamine 0/0) või on täpselt −1,0 või +1,0 (küllastuse piirväärtused, kui ühe riba väärtus on null, teise aga mitte) käsitletakse pigem andmeteta pikslina kui äärmusliku väärtusena. See hoiab ülepõlenud heledad alad ja tumedad varjud värviskaalast välja, selle asemel et kujutada neid kaadri kõige äärmuslikumate väärtustena. Sama reegel määrab, millised pikslid toidavad AUTO künniseid ja indeksihistogrammi, nii et need kolm on omavahel kooskõlas.
{% endhint %}

Läbipaistvus säilib, kui eksport kirjutatakse formaadis PNG. Seda ei saa JPG-vormingus esitada.

***

## Väärtuste lugemine seadistamise ajal

Konfiguratsioonipaneeli all asuv paneel **Kursori väärtused** on Sandboxi mõõtevahend:

* Liigutage kursorit pildi üle ja lugege kanali kaupa allika väärtusi ning indeksväärtust selle enda real
* Lülita sisse histogrammi kohal olev nupp **INDEX**, et näha indeksväärtuste jaotust kaadris, kus kaks klippimiskünnist on kujutatud oranžide katkendjoontena ja kursori väärtus valge joonena — see on kiireim viis valida aken, mis tegelikult sisaldab sinu andmeid
* Lülita sisse **CURSOR**, et näha märgistusjooni kursori all olevate väärtuste juures
* Suurenda üle 60× (vähem, kui on määratud GSD-ploki suurus), et esile tõsta üksikuid kuvatavaid piksleid ujukomaväärtusega

Praktiline juhend:

1. Märgi üles väärtused terve taimestiku, stressis taimestiku, palja pinnase ja vee kohal
2. Vaadake, kus need klastrid asuvad indeksihistogrammil
3. Määrake miinimum- ja maksimumväärtused, et piiritleda teile oluline klaster
4. Valige kärpimisrežiim – _Original Background_ hoiab ümbruskonna pildi nähtavana

***

## Eksportimine Sandboxist

Kõik eespool nimetatu on reaalajas eelvaade, kuni te seda salvestate. Külgriba ülaosas asuv nupp **Export/Save Image(s)** avab paneeli, mis libiseb külgriba peale (selle asemel, et pilti katta, nii et saate endiselt näha, mille üle otsustate).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Valikud

| Valik                          | Mõju                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Kohalda praegusele pildile**      | Salvestab täpselt kuvatud pildi koos nende seadistustega                                                                                                |
| **Kohalda kõigile projekti piltidele** | Korrata sama konfiguratsiooni kõigi projekti piltide puhul. Pildid, millel puuduvad selle indeksiga vajalikud lainepikkused, jäetakse vahele ega käsitleta vigadena |
| **Indeksi/LUT gradiendiriba**      | Salvestab iga ekspordi kohta ka eraldi legendipildi, millel on märgitud väärtuste vahemik                                                                     |
| **Indeksi histogramm**             | Salvestab iga ekspordi kohta ka eraldi histogrammipildi, millel on näidatud andmete miinimum- ja maksimumväärtused ning kärpimiskünnised                                               |

Kui pildi vahekaardi **GSD-ploki suurus** on suurem kui 1, näitab paneel seda enne kinnitamist: eksport salvestab selle, mida te hetkel vaatate, sealhulgas plokkide keskmistamist. Kui soovite täisresolutsiooni, seadke GSD-regulaator esmalt tagasi väärtusele 1.

### Kuhu failid salvestatakse

Iga **Eksport**-nupule vajutusega luuakse**uus, kunagi varem kasutamata kaust**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Näited: `Sandbox_Exports/NDVI_LUT_001/`, järgmisel käivitamisel `Sandbox_Exports/NDVI_LUT_002/`. Numbrite genereerimine põhineb kettal juba olevate failide skaneerimisel, seega säilib see ka süsteemi taaskäivitamisel ja käsitsi kustutatud kaustade puhul. Mitte midagi ei kirjutata kunagi üle – Sandboxi peamine eesmärk on võrrelda ühte katset eelmise katsega.

Kausta sees, iga pildi kohta:

| Fail                                                   | Sisu                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Renderdatud pilt, piksel pikseli haaval täpselt nii, nagu vaataja seda näitas |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Gradiendiriba kõrvalfail, kui seda on taotletud                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Indeksihistogrammi kõrvalfail, kui seda on taotletud                  |

Mõlemad lisad salvestatakse alati **täisresolutsioonis**, isegi kui põhikuva on plokkide kaupa keskmistatud: ploki suurus on ekraani resolutsioon ning mõlemad lisad loevad tõelisi pikselipõhiseid indeksväärtusi. Samuti kuvavad need rohkem teavet kui ekraanil kuvatavad versioonid – mõlemad märgivad ära nii venitusakna _kui ka_ tõelised andmete miinimum- ja maksimumväärtused, mistõttu on salvestatud legend loetav veel mitu kuud hiljemgi, ilma et projekt oleks avatud.

### Edusammud ja tulemused

Kogu projekti eksportimine võtab vaid mõne minuti, seega edastab programm tulemusi reaalajas edusammude kanali kaudu, mitte blokeerides süsteemi:

* Edusammude riba näitab `current / total` ja kirjutamisel olevat faili
* Kui eksport on lõppenud, näitab paneel, kui palju pilte eksporditi, kui palju jäeti vahele ja väljundkausta asukoha
* Vahele jäetud pildid on loetletud koos põhjendusega (kuni viis kuvatakse, seejärel rida „+N veel”). Tavaliselt on põhjuseks kiht, millel puuduvad selle indeksiga vajalikud kanalid
* Kui projektis pole **ühtegi** pilti, mis saaks indeksit kasutada, teatab käivitus ebaõnnestumisest, selle asemel et jätta teile tühi kaust

Korraga töötab ainult üks liivakasti eksport. Teise käivitamine, kui esimene on veel käimas, lükatakse selge sõnumiga tagasi, selle asemel et lasta kahel protsessil sama projektifaili pärast võidelda.

### Võrgustik võtab käivituse üle

Iga lõpetatud käivitus kuvatakse eraldi nupuna [pildivõrgustiku](image-grid.md) tööriistaribal, märgistusega `<IndexName> <Index|LUT> <NNN>`. Nii saadki jookse võrrelda: ekspordi kaks korda erinevate gradientide või läviväärtustega, seejärel vaheta ruudustiku kahe nupu vahel.

***

## Kohandatud indeksivalemid (Chloros+)

{% hint style="info" %}
**Kus neid luua**: Sandboxi külgribal või**Projekti seaded** menüüs enne töötlemist. Mõlemad salvestavad andmed samasse projektitasandi nimekirja.
{% endhint %}

1. Avage kohandatud valemi kalkulaator indeksivalemite rippmenüüst (nõuab sisselogimist sobiva Chloros+ tellimusega)
2. Kirjutage valem, kasutades **vahemiku- ja pesa sümboleid** `x`, `y`, `z`, `a`, `b`, `c` — mitte sagedusribade nimed
3. Saadaval olevad operaatorid: `+`, `-`, `*`, `/`, `^` ja `()` rühmitamiseks
4. Saadaval olevad funktsioonid: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Nimetage ja salvestage see – see ilmub valemi rippmenüü alla ja saate selle pesad siduda, lohistades kanalite ringe, täpselt nagu sisseehitatud eelseadistuse puhul

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Kohandatud valemid on saadaval ainult graafilises kasutajaliideses.** Valik CLI/SDK `--indices` laiendab 22 sisseehitatud eelseadistuse nime ja jätab vaikimisi kõik muu vahele, sealhulgas teie kohandatud valemid. Kohandatud valemi hulgitöötlemiseks konfigureerige see projekti seadetest ja käivitage töötlemine või kasutage Sandboxi ekspordifunktsiooni „Kohalda kõigile projekti piltidele”.
{% endhint %}

***

## Veaotsing

### „Sellel kihil puuduvad selle indeksi jaoks vajalikud kanalid”

Valem loeb kanali asukohta, mida praegusel kihil pole – näiteks kolme pesaga indeks ühe- või kahekanalilises failis. Vahetage mitme sagedusribaga kihile (peegeldusvõime või debayered) või valige indeks, mis sobib teie kaamera filtri jaoks.

### „Pilditöötluse tagapõhja ei õnnestunud saavutada“

Tagapõhi ei vasta. Kontrollige vahekaarti „Log“; kui tagapõhi taaskäivitub, taastub Sandbox ise, kui see on jälle saadaval.

### Pilt ei muutunud, kui ma ringi lohistasin

Valem pole veel täielik. Ebatäielikku valemit käsitletakse tavalise lohistamise vaheolukorrana – midagi ei renderdata ja midagi ei teatata veana. Täida kõik valemis kasutatavad lahtrid.

### Kogu pilt on ühevärviline

Su klippaken on tõenäoliselt andmetest kaugel väljaspool. Vajuta **AUTO**, et see joondada 2. või 98. protsentiili järgi, või lülita sisse**INDEX**-histogramm, et näha, kus andmed tegelikult asuvad.

### Eksporditud värvid ei vasta sellele, mida ma nägin

Need peaksid vastama – eksporditee on teadlikult reaalajas eelvaate peegelpilt, sealhulgas klippimisrežiimi alfa, ning plokkide keskmistamine rakendatakse _pärast_ värvimist täpselt nii, nagu vaataja seda teeb. Kui need erinevad, kontrollige, et GSD-ploki suurus ei oleks vaatamise ja eksportimise vahel muutunud.

***

## Järgmised sammud

* [**Pildikihtid**](image-layers.md) — millisel kihil indeksit käivitada ja mida selle väärtused tähendavad
* [**Pildi avamine täisekraanil**](opening-an-image-full-screen.md) — kursori näidud, histogramm ja GSD-juhtimine üksikasjalikult
* [**Multispektraalsete indeksite valemid**](../project-settings/multispectral-index-formulas.md) — kõik eelseadistused, igal pinnal
* [**Projekti seaded**](../project-settings/project-settings.md) — leitud seadete salvestamine töötlemisprotsessi jaoks
