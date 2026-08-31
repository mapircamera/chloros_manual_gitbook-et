---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektraalsete indeksite valemid

Allpool toodud indeksite valemites kasutatakse Survey3 filtri keskmise läbilaskvuse vahemike kombinatsiooni:

<table><thead><tr><th align="center">Survey3 filtri värv</th><th width="196.199951171875" align="center">Survey3 filtri nimi</th><th width="159.800048828125" align="center">Läbilaskvusvahemik (FWHM)</th><th align="center">Keskmine läbilaskvus</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN – Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN – NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Kui kasutatakse neid valemeid, võib nimetus lõppeda „\_1“ või „\_2“ga, mis vastab sellele, millist NIR filtrit kasutati – kas NIR1 või NIR2.

LATTICE M3C (Bayer kolmekordne ribapääs) kaamerate puhul kasutab sama indeksimootor M3C filtri ribasid:

| M3C filter | Riba 1 (keskpunkt/FWHM) | Riba 2 (keskmine/FWHM) | Rib 3 (keskmine/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M-kaamerad on ühe lainepikkusega (üks kitsasribaline filter kaamera kohta), seega ei arvutata mitme lainepikkusega indekseid ühe M3M-pildi jaoks. Indeksite arvutamiseks M3M-iga kombineerige kaks või enam kaamerat joondatud mitme lainepikkusega pildistikuks ja kasutage LATTICE indeksimootorit (`chloros-cli lattice index` või graafilise kasutajaliidese reaalajas indeksikalkulaatorit).

***

## Kus iga indeksinimi toimib

Chloros-il on **kolm** indeksipinda ning nende eelseadistatud nimekirjad ei ole identsed. Kasutage seda jaotist, et kontrollida, kas nimi toimib seal, kus te seda kasutada kavatsete.

| Kus te olete | Milline nimekiri kehtib | Arv |
| --- | --- | --- |
| Projekti seaded → Indeks → Lisa indeks (GUI) | Pind 1 | 27 |
| Pildivaataja [Indeks/LUT-liivakast](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Pind 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Pind 2 | 22 |
| SDK `process_folder(indices=[...])` | Pind 2 | 22 |
| `chloros-cli lattice index --preset` | Pind 3 | 22 (teine 22) |
| Vahekaart „Kaamerad“ – reaalajas indeksikalkulaator | Pind 3 | 22 (teine 22) |

Pinnad 1 ja 2 töötavad **ühe kaamera ühe pildiga korraga**, kasutades sümbolikohti `x`/`y`/`z`(/`a`), mis on seotud selle kaamera filterkanalitega. Pind 3 töötab**joondatud mitme sagedusribaga pildistakiga** — mitu LATTICE-kaamerat on ühiselt registreeritud üheks kuubiks — ja viitab kanalitele väiketähtedega nimedega.

### 1. GUI projekti seaded / pildivaataja sandboxi rippmenüü — 27 valemit

Rippmenüüs on need loetletud järgmises järjekorras (see on lisamise järjekord, mitte tähestikuline järjekord):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

GUI-s lohistate oma kaamera filtri kanalid valemi ribade pesadesse, nii et iga valemit saab kasutada koos iga riba määramisega, mida teie kaamera toetab. Teie salvestatud kohandatud valemid lisatakse selle loendi alla.

**Viis ainult GUI-s kasutatavat** valemit — neid, mida nimekiri CLI/SDK `--indices` ei aktsepteeri — on rakendatud järgmiselt:

| Ainult kasutajaliidese eelseadistus | Valem (rakendatuna) | Pesad |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (neli pesad) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Iga elemendi jaoks ettenähtud vastavus on esitatud eraldi jaotises käesoleva lehe allpool (näiteks GARI eeldab, et x=Green, y=NIR, z=Blue, a=Red). GARI on ainus valem Chloros-is, mis kasutab neljandat pesa.

### 2. CLI / SDK `--indices` nime laiendamine — 22 eelseadistust

Valik `chloros-cli process --indices` (ja parameeter SDK `indices`) aktsepteerib järgmisi eelseadistuste nimesid:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Tundmatud indeksinimed jäetakse märkamatult vahele.** Selles nimekirjas mitte sisalduv nimi (sealhulgas viis ainult graafilises kasutajaliideses kasutatavat valemit `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` ja mis tahes kasutajapoolne valem, mille olete graafilises kasutajaliideses salvestanud) jäetakse kõrvale, andes sellest teada vaid logis – käivitamine jätkub ilma selle indeksita ja käivitamine ise kajastatakse ikkagi edukana. Teade kuvatakse järgmiselt:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Nimede võrdlemisel ei arvestata suurtähti ega väiketähti ning tühikud eemaldatakse, seega on `ndvi`, `NDVI` ja ` NDVI ` on sama eelseadistus. Eelseadistus jäetakse vahele ka juhul, kui see vajab lainepiirkonda, mida teie kaamera filter ei paku.
{% endhint %}

Täpsed rakendatud valemid (sümbolid `x`/`y`/`z` on sagedusribade pesad; vaikimisi kaardistus on näidatud iga eelseadistuse kohta):

| Eelseadistus | Valem (rakendatud kujul) | Vaikimisi filter | Pesad (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Kuidas eelseadistuse nimest saavad kanalipositsioonid

Kui edastatakse lihtsalt nimi, näiteks `NDVI`, peab Chloros otsustama, millise faili millisest kanalist iga sümbol loetakse. Selleks kasutatakse järgmist tabelit, mis seostab filtri koodi iga kanali massiivi positsiooniga:

| Filtrikood | Kanal → massiivi indeks |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` aktsepteeritakse Orange aliasena, samuti 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Eelseadistuse **vaikimisi filter** (ülal olev veerg „Vaikimisi filter”) kasutatakse siis, kui projekt sisaldab selle filtriga pilte. Kui seda ei ole, otsib Chloros projektis tegelikult olemasolevaid filtreid järjekorras `RGN, OCN, NGB, RGB, RE, NIR` ja valib esimese, mis suudab pakkuda kõiki eelseadistuse jaoks vajalikke kanaleid. Kui ükski neist ei suuda seda teha, jäetakse eelseadistus selle käivituse puhul kõrvale. Seetõttu annab `NDVI`, mida taotletakse ainult OCN-andmekogumil, ikkagi mõistliku tulemuse – see seostub OCN&#x27;positsioonidega Orange ja NIR.

LATTICE M3C mudeli stringid kannavad filtrit eesliitega `F` (`LATT-M3C-L41-FRGN`), kuid eesliide jäetakse ära, kui filtri kood pildilt loetakse, mistõttu FRGN-kaamera tuvastab selle üleval asuva `RGN` rea kaudu ega vaja erikohtlemist.

### 3. LATTICE indeksimootor (`lattice index --preset`, reaalajas indeksikalkulaator) — 22 eelseadistust

LATTICE-mootor töötab joondatud mitme sagedusriba pildistakidega (reaalajas massiivid või eksporditud mitme sagedusriba TIFF-failid) ja kasutab väiketähelisi kanalite nimesid (`red`, `green`, `blue`, `red_edge`, `nir`). Selle eelseadete nimekiri erineb kahest eelnevast:

| Eelseade | Valem | Kanalid |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | punane, NIR |
| GNDVI | `(nir - green) / (nir + green)` | roheline, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | sinine, lähi-infrapuna |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | punane\_serv, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | sinine, roheline, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | punane, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | punane, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | punane, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | sinine, punane, infrapuna |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | punane, infrapuna |
| CVI | `(nir / green) - (red / green)` | punane, roheline, infrapuna |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | punane, infrapuna |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | punane, infrapuna |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | punane, roheline, infrapuna |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | punane, roheline, sinine |
| NGRDI | `(green - red) / (green + red)` | punane, roheline |
| VARI | `(green - red) / (green + red - blue)` | punane, roheline, sinine |
| TGI | `green - 0.39*red - 0.61*blue` | punane, roheline, sinine |
| EXG | `2*green - red - blue` | punane, roheline, sinine |
| CIRE | `(nir / red_edge) - 1` | punane_serv, infrapuna |
| CIGREEN | `(nir / green) - 1` | roheline, nir |
| NDWI | `(green - nir) / (green + nir)` | roheline, nir |

Käivita käsk `chloros-cli lattice index --list-presets`, et printida see tabel sinu paigaldatud versioonist, ja käsk `--list-gradients`, et näha saadavalolevaid värvigradiente. Kanalisümbolid on suurtähelised ja peavad vastama eelseadistuse väiketähelistele nimedele (nt `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Nagu rakendatud graafilises kasutajaliideses ja eelseadete nimekirjas CLI/SDK, on CVI suhete suhete valem:

$$
CVI = {(z / y) \over (x / y)}
$$

kus vaikimisi RGB kanalite vastavus on x=Red, y=Green, z=Blue. Graafilises kasutajaliideses saate lohistada mis tahes oma kaamera kanali x/y/z pesadesse. Pange tähele, et LATTICE indeksimootori eelseadistus `CVI` kasutab teistsugust valemit, `(NIR / Green) - (Red / Green)` — kontrollige ülaltoodud tabelitest, millist pinda te kasutate.

***

## ENDVI – täiustatud normaliseeritud taimestiku erinevuse indeks

See indeks kasutab lisaks NIR-le ja rohelisele kanalile ka sinist kanalit ning on populaarne NGB-filtriga kaamerate puhul, kus sinine sagedusala asendab punast.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Rakendamiseks kasutatakse sümbolvalemit `((x+y)-(2*z))/((x+y)+(2*z))` — määrake oma kaamera kanalid NIR ja Green x/y-pesadesse ning Blue z-pesasse (NGB-kaamera puhul: x=NIR, y=Green, z=Blue).

***

## EVI – täiustatud taimestiku indeks

See indeks töötati algselt välja kasutamiseks koos MODIS andmetega kui täiustus indeksile NDVI, optimeerides taimestiku signaali piirkondades, kus lehepinna indeks on kõrge (LAI). See on kõige kasulikum piirkondades, kus LAI väärtus on kõrge ja kus NDVI võib küllastuda. Selles kasutatakse sinise peegelduspiirkonna andmeid, et korrigeerida pinnase taustsignaale ja vähendada atmosfääri mõjusid, sealhulgas aerosoolide hajumist.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI väärtused peaksid taimestikupikslite puhul jääma vahemikku 0–1. Heledad objektid, nagu pilved ja valged hooned, ning tumedad objektid, nagu vesi, võivad põhjustada ebanormaalsed pikseliväärtused EVI-pildil. Enne EVI-pildi loomist tuleks peegelduspildilt maskeerida välja pilved ja heledad objektid ning soovi korral määrata pikseliväärtuste künnis vahemikku 0–1.

_Viide: Huete, A. jt. „Ülevaade MODIS taimestikuindeksite radiomeetrilisest ja biofüüsikalisest toimivusest.“ Remote Sensing of Environment 83 (2002): 195–213._

***

## FCI1 – Metsakatte indeks 1

_Ainult GUI-s – ei ole saadaval eelseadistustena CLI/SDK või `--indices`._

See indeks eristab metsa latvust muudest taimestikutüüpidest, kasutades multispektraalset peegeldusvõime pildimaterjali, mis sisaldab punase serva laineala.

$$
FCI1 = Red * RedEdge
$$

Metsapiirkondadel on madalamad FCI1 väärtused, kuna puude peegeldusvõime on madalam ja puuvõrade sees esinevad varjud.

_Viide: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. „Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Metsakatte indeks 2

_Ainult graafilises kasutajaliideses (GUI) – ei ole saadaval eelseadistustena CLI/SDK ega `--indices`._

See indeks eristab metsa latvust muudest taimestikutüüpidest, kasutades multispektraalset peegeldusvõime pildimaterjali, mis ei sisalda punase serva sagedusriba.

$$
FCI2 = Red * NIR
$$

Metsapiirkondadel on madalamad FCI2 väärtused, kuna puude peegeldusvõime on madalam ja puuvõrade sees esinevad varjud.

_Viide: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. „Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – globaalne keskkonnaseireindeks

_Ainult GUI – ei ole saadaval eelseadistustena CLI/SDK `--indices`._

Seda mittelineaarset taimestikuindeksit kasutatakse satelliitpiltide põhjal toimuva ülemaailmse keskkonnaseire jaoks ning selle eesmärk on korrigeerida atmosfääri mõjusid. See on sarnane indeksiga NDVI, kuid on atmosfääri mõjude suhtes vähem tundlik. Seda mõjutab paljas pinnas; seetõttu ei soovitata seda kasutada hõreda või mõõdukalt tiheda taimestikuga aladel.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kus:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Viide: Pinty, B. ja M. Verstraete. GEMI: mittelineaarne indeks globaalse taimestiku seireks satelliitide abil. Vegetation 101 (1992): 15–20._

***

## GARI – Green atmosfäärikindlad indeksid

_Ainult GUI-s – ei ole saadaval eelseadistustena CLI/SDK `--indices`._

See indeks on tundlikum laiale klorofülli kontsentratsioonide vahemikule ja vähem tundlik atmosfääriliste mõjude suhtes kui NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammakonstant on kaalufunktsioon, mis sõltub atmosfääri aerosoolitingimustest. ENVI kasutab väärtust 1,7, mis on Gitelsoni, Kaufmani ja Merzylaki (1996, lk 296) soovitatud väärtus.

_Viide: Gitelson, A., Y. Kaufman ja M. Merzylak. „Green-kanali kasutamine globaalse taimestiku kaugseires EOS-MODIS abil.“ Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green klorofülli indeks

Seda indeksit kasutatakse lehtede klorofüllisisalduse hindamiseks paljude taimeliikide puhul.

$$
GCI = {NIR \over Green} - 1
$$

Laiade NIR ja roheliste lainepikkuste kasutamine võimaldab klorofüllisisaldust paremini ennustada, pakkudes samal ajal suuremat tundlikkust ja paremat signaali-müra suhet.

_Viide: Gitelson, A., Y. Gritz ja M. Merzlyak. „Seosed lehe klorofüllisisalduse ja spektraalse peegeldusvõime vahel ning algoritmid kõrgtaimede lehtede klorofülli mittetõkestavaks hindamiseks.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green leheindeks

See indeks oli algselt mõeldud kasutamiseks digitaalkaameraga RGB nisu katvuse mõõtmiseks, kus punase, rohelise ja sinise värvi digitaalsed numbrid (DN) jäävad vahemikku 0–255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI väärtused jäävad vahemikku -1 kuni +1. Negatiivsed väärtused tähistavad mulda ja eluta objekte, positiivsed väärtused aga rohelisi lehti ja varre.

_Viide: Louhaichi, M., M. Borman ja D. Johnson. „Ruumiandmetega platvorm ja õhufotograafia nisu karjatamise mõju dokumenteerimiseks.“ Geocarto International 16, nr 1 (2001): 65–70._

***

## GNDVI - Green Normaliseeritud taimestiku erinevusindeks

See indeks sarnaneb indeksiga NDVI, kuid erinevus seisneb selles, et see mõõdab rohelist spektrit vahemikus 540–570 nm punase spektri asemel. See indeks on klorofülli kontsentratsiooni suhtes tundlikum kui NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Viide: Gitelson, A. ja M. Merzlyak. „Klorofülli kontsentratsiooni kaugseire kõrgemate taimede lehtedes.“ Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimeeritud mullaga korrigeeritud taimestiku indeks

See indeks töötati algselt välja värvi-infrapunafotograafia abil, et ennustada maisi lämmastikuvajadust. See sarnaneb indeksiga OSAVI, kuid asendab rohelise lainepikkuse punasega.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Viide: Sripada, R., jt. „Maisi hooajalise lämmastikuvajaduse määramine õhust tehtud värvi-infrapunafotode abil.“ Doktoritöö, Põhja-Carolina Riiklik Ülikool, 2005._

***

## GRVI - Green suhtarvuline taimestikuindeks

See indeks on tundlik metsade lehtkatte fotosünteesikiiruse suhtes, kuna rohelise ja punase peegeldusvõimet mõjutavad tugevalt lehepigmentide muutused.

$$
GRVI = {NIR \over Green }
$$

_Viide: Sripada, R. jt. „Aerial Color Infrared Photography for Determining Early In-season Nitrogen Requirements in Corn.“ Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Pinnase järgi korrigeeritud taimestiku indeks

See indeks töötati algselt välja värvi-infrapunafotograafia abil maisi lämmastikuvajaduse prognoosimiseks. See sarnaneb indeksiga SAVI, kuid rohelise laineala asemel kasutatakse punast.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Viide: Sripada, R. jt. „Maisi hooajalise lämmastikuvajaduse määramine õhust tehtud värvi-infrapunafotode abil.“ Doktoritöö, Põhja-Carolina Riiklik Ülikool, 2005._

***

## LAI – lehepinna indeks

Seda indeksit kasutatakse lehtkatte hindamiseks ning põllukultuuri kasvu ja saagi prognoosimiseks. ENVI arvutab rohelise LAI väärtuse, kasutades järgmist empiirilist valemit Boegh jt (2002) järgi:

$$
LAI = 3.618 * EVI - 0.118
$$

Kus EVI on:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Kõrged LAI väärtused jäävad tavaliselt vahemikku umbes 0 kuni 3,5. Kui pildil on aga pilved ja muud heledad objektid, mis tekitavad küllastunud piksleid, võivad LAI väärtused ületada 3,5. Ideaaljuhul tuleks enne LAI-pildi loomist pildilt pilved ja heledad objektid välja maskeerida.

_Viide: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde ja A. Thomsen. „Õhust kogutud multispektraalsed andmed lehepinnaindeksi, lämmastikukontsentratsiooni ja fotosünteetilise efektiivsuse kvantifitseerimiseks põllumajanduses.“ Remote Sensing of Environment 81, nr 2–3 (2002): 179–193._

***

## LCI – Lehe klorofülli indeks

_Ainult GUI-s – ei ole saadaval eelseadistustena CLI/SDK `--indices`._

Seda indeksit kasutatakse kõrgtaimede klorofüllisisalduse hindamiseks, kuna see on tundlik klorofülli neeldumisest tingitud peegeldusvõime muutuste suhtes.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Viide: Datt, B. „Eukalüpti lehtede veesisalduse kaugseire.“ Journal of Plant Physiology 154, nr 1 (1999): 30–36._

***

## MNLI – modifitseeritud mittelineaarne indeks

See indeks on mittelineaarse indeksi (NLI) täiustus, mis hõlmab mulla taustaga arvestavat mulla korrigeeritud taimestiku indeksit (SAVI). ENVI kasutab lehtkatte taustakorrigeerimisteguri (_L_) väärtust 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Viide: Yang, Z., P. Willis ja R. Mueller. „Band-Ratio Enhanced AWIFS-pildi mõju põllukultuuride klassifitseerimise täpsusele.” Pecora 17 kaugseire sümpoosioni (2008) materjalid, Denver, CO._

***

## MSAVI2 – modifitseeritud pinnasega korrigeeritud taimestiku indeks 2

See indeks on Qi jt (1994) poolt välja pakutud indeksi MSAVI lihtsam versioon, mis täiustab pinnasega korrigeeritud taimestiku indeksit (SAVI). See vähendab pinnase müra ja suurendab taimestiku signaali dünaamilist ulatust. MSAVI2 põhineb induktiivsel meetodil, mis ei kasuta konstantset _L_ väärtust (nagu SAVI puhul), et esile tuua tervet taimestikku.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Viide: Qi, J., A. Chehbouni, A. Huete, Y. Kerr ja S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – modifitseeritud lihtsuhe

See indeks on lihtsa NIR/Red suhte modifikatsioon, mis on loodud selle seose lineariseerimiseks biofüüsikaliste parameetritega, ning on suurema taimestiku tiheduse korral tundlikum kui NDVI.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Viide: Chen, J. „Taimestiku indeksite ja modifitseeritud lihtsuhte hindamine boreaalse vööndi rakenduste jaoks.” Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – normaliseeritud erinevus RedEdge

See indeks sarnaneb indeksiga NDVI, kuid võrdleb kontrasti indeksite NIR ja RedEdge vahel, mitte indeksite Red vahel, mis tuvastab taimestiku stressi sageli varem.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – normaliseeritud erinevuse taimestiku indeks

See indeks mõõdab tervet, rohelist taimestikku. Normaliseeritud erinevuse valemi ja klorofülli kõrgeima neeldumis- ja peegelduspiirkondade kasutamise kombinatsioon muudab selle indeksit usaldusväärseks laias tingimuste vahemikus. Siiski võib see tiheda taimestiku tingimustes küllastuda, kui LAI väärtus muutub kõrgeks.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Selle indeksi väärtus jääb vahemikku -1 kuni 1. Rohelise taimestiku tavaline väärtusvahemik on 0,2–0,8.

_Viide: Rouse, J., R. Haas, J. Schell ja D. Deering. „Monitoring Vegetation Systems in the Great Plains with ERTS“. Kolmas ERTS-sümpoosion, NASA (1973): 309–317._

***

## NLI – mittelineaarne indeks

See indeks eeldab, et paljude taimestikuindeksite ja pinna biofüüsikaliste parameetrite vaheline seos on mittelineaarne. See lineariseerib seosed pinna parameetritega, mis on tavaliselt mittelineaarsed.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Viide: Goel, N. ja W. Qin. „Lehtkatte arhitektuuri mõju erinevate taimestikuindeksite ning LAI ja Fpar vahelistele seostele: arvutisimulatsioon.“ Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – optimeeritud pinnasega korrigeeritud taimestiku indeks

See indeks põhineb pinnasega korrigeeritud taimestiku indeksil (SAVI). Selles kasutatakse lehtkatte taustakorrigeerimistegurina standardväärtust 0,16. Rondeaux (1996) tuvastas, et see väärtus võimaldab madala taimkatte korral suuremat pinnase varieeruvust kui SAVI, näidates samas suuremat tundlikkust taimkatte suhtes, mis ületab 50%. Seda indeksit on kõige parem kasutada piirkondades, kus taimkate on suhteliselt hõre ja muld on lehtkatte kaudu nähtav.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Viide: Rondeaux, G., M. Steven ja F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – renormaliseeritud erinevusvegetatsiooniindeks

See indeks kasutab lähi-infrapuna- ja punaste lainepikkuste vahet koos NDVI-iga, et esile tuua tervet taimestikku. See on tundetu mulla ja päikese vaatamisgeomeetria mõjude suhtes.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Viide: Roujean, J. ja F. Breon. „Taimestiku poolt neeldunud PARi hindamine kahepoolsetest peegeldusmõõtmistest.” Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – pinnase mõju arvesse võttev taimestiku indeks

See indeks sarnaneb indeksiga NDVI, kuid see summutab pinnasepikslite mõju. Selles kasutatakse lehtkatte taustakorrektsioonitegurit _L_, mis sõltub taimestiku tihedusest ja nõuab sageli eelnevat teavet taimestiku hulgast. Huete (1988) soovitab optimaalseks väärtuseks _L_=0,5, et arvesse võtta esimese järgu pinnase taustamuutusi. Seda indeksit on kõige parem kasutada piirkondades, kus taimestik on suhteliselt hõre ja pinnas on lehtkatte kaudu nähtav.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Viide: Huete, A. „Pinnasega korrigeeritud taimestiku indeks (SAVI).“ Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – transformeeritud erinevuste taimestiku indeks

See indeks on kasulik taimkatte seireks linnakeskkonnas. See ei küllastu nagu NDVI ja SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Viide: Bannari, A., H. Asalhi ja P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” kogumikus „Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International”, 5. köide (2002)._

***

## VARI – nähtava valguse atmosfäärikindlusindeks

See indeks põhineb indeksil ARVI ja seda kasutatakse taimestiku osakaalu hindamiseks pildil, kus on madal tundlikkus atmosfääriliste mõjude suhtes.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Viide: Gitelson, A. jt. „Taimestik ja mullapiirid nähtavas spektriruumis: kontseptsioon ja meetod taimestiku osakaalu kaughinnanguks“. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – laia dünaamilise ulatusega taimestiku indeks

See indeks sarnaneb indeksiga NDVI, kuid selles kasutatakse kaalukoefitsienti (_a_), et vähendada lähi-infrapuna- ja punaste signaalide panuse erinevust indeksis NDVI. Indeks WDRVI on eriti tõhus piltidel, millel on mõõdukaskuni kõrge taimestiku tihedusega, kui NDVI ületab 0,6. NDVI kipub tasapisi vähenema, kui taimestiku osakaal ja lehepinna indeks (LAI) suurenemisel, samas kui WDRVI on tundlikum laiemale taimestiku osakaalu vahemikule ja muutustele näitajas LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Kaalukoefitsient (_a_) võib olla vahemikus 0,1–0,2. Henebry, Viña ja Gitelson (2004) soovitavad väärtust 0,2.

_Viited_

_Gitelson, A. „Laiadünaamilise ulatusega taimestikuindeks taimestiku biofüüsikaliste omaduste kaugmõõtmiseks.“ Journal of Plant Physiology 161, nr 2 (2004): 165–173._

_Henebry, G., A. Viña ja A. Gitelson. „Laiadünaamilise ulatusega taimestikuindeks ja selle potentsiaalne kasulikkus lünkade analüüsis.“ Gap Analysis Bulletin 12: 50–56._
