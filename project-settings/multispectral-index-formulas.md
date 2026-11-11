---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektraalsete indeksite valemid

Allpool toodud indeksite valemites kasutatakse Survey3 filtri keskmise läbilaskvuse vahemike kombinatsiooni:

<table><thead><tr><th align="center">Survey3-filtri värv</th><th width="196.199951171875" align="center">Survey3-filtri nimi</th><th width="159.800048828125" align="center">Läbilaskvusvahemik (FWHM)</th><th align="center">Keskmine läbilaskvus</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Kui kasutatakse neid valemeid, võib nimetus lõppeda „\_1” või „\_2”ga, mis vastab sellele, millist NIR-filtrit – kas NIR1 või NIR2 – kasutati.

LATTICE M3C (Bayer kolmekordne ribapääs) kaamerate puhul kasutab sama indeksimootor M3C filtri ribasid:

| M3C filter | Rib 1 (keskpunkt/FWHM) | Rib 2 (keskpunkt/FWHM) | Rib 3 (keskpunkt/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M-kaamerad on ühe lainepikkusega (üks kitsasriba filter kaamera kohta), seega ei arvutata mitme lainepikkusega indekseid ühe M3M-pildi jaoks. Indeksite arvutamiseks M3M-iga kombineerige kaks või enam kaamerat joondatud mitme lainepikkusega pildistackiks ja kasutage LATTICE indeksimootorit (`chloros-cli lattice index` või graafilise kasutajaliidese reaalajas töötavat indeksikalkulaatorit).

***

## Kus iga indeksinimi toimib

Chlorosil on **kolm** indeksipinda ja nende eelseadistuste loendid ei ole identsed. Kasutage seda jaotist, et kontrollida, kas nimi toimib seal, kus te seda kasutada kavatsete.

| Kus te olete | Milline loend kehtib | Arv |
| --- | --- | --- |
| Projekti seaded → Indeks → Lisa indeks (GUI) | Pind 1 | 27 |
| Pildivaataja [Indeks/LUT-liivakast](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Pind 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Surface 2 | 22 |
| SDK `process_folder(indices=[...])` | Surface 2 | 22 |
| `chloros-cli lattice index --preset` | Pind 3 | 22 (teine 22) |
| Kaamerad-vahekaardi reaalajas indeksikalkulaator | Pind 3 | 22 (teine 22) |

Surface 1 ja 2 töötavad **ühe kaamera ühe pildiga korraga**, kasutades sümbolipesi `x`/`y`/`z`(/`a`), mis on seotud selle kaamera filterkanalitega. Pind 3 töötab**joondatud mitme sagedusriba pildistiku** põhjal — mitu LATTICE-kaamerat on ühe kuubina ühildatud — ja viitab kanalitele väiketähtedega nimedega.

### 1. GUI projekti seaded / pildivaataja sandboxi rippmenüü — 27 valemit

Rippmenüüs on need loetletud järgmises järjekorras (see on lisamisjärjekord, mitte tähestikuline järjestus):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

GUI-s lohistate oma kaamera filterkanalid valemi ribade pesadesse, nii et iga valemit saab kasutada koos mis tahes ribade määramisega, mida teie kaamera toetab. Teie salvestatud kohandatud valemid lisatakse selle loendi alla.

**Viis ainult GUI-s kasutatavat** valemit — need, mida CLI/SDK-i nimekiri `--indices` ei aktsepteeri — on rakendatud järgmiselt:

| Ainult GUI-s kasutatav eelseade | Valem (rakendatud kujul) | Pesad |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (neli pesa) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Iga valemi ettenähtud kaardistus on esitatud selle lehekülje allpool asuvas eraldi jaotises (näiteks GARI eeldab, et x=Green, y=NIR, z=Blue, a=Red). GARI on ainus valem Chloros-is, mis kasutab neljandat pesa.

### 2. CLI / SDK `--indices` nime laiendamine — 22 eelseadistust

Valik `chloros-cli process --indices` (ja SDK parameeter `indices`) aktsepteerib järgmisi eelseadistuste nimesid:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Tundmatud indeksinimed jäetakse märkamatult vahele.** Sellest nimekirjast väljaspool olev nimi (sealhulgas viis ainult graafilises kasutajaliideses kasutatavat valemit `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` ning kõik kasutajaliideses salvestatud kohandatud valemid) jäetakse kõrvale, andes sellest vaid teate logisse – käivitamine jätkub ilma selle indeksita ja tulemuseks on ikkagi edukas käivitamine. Teade kuvatakse järgmiselt:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Nimede võrdlemisel ei arvestata suurtähti pärast tühikute eemaldamist, seega on `ndvi`, `NDVI` ja ` NDVI ` sama eelseadistus. Eelseadistus jäetakse vahele ka juhul, kui see vajab sagedusriba, mida teie kaamera filter ei paku.
{% endhint %}

Täpsed rakendatud valemid (sümbolid `x`/`y`/`z` on lainepikkuste pesad; vaikimisi vastavus on näidatud iga eelseadistuse kohta):

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
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Kuidas eelseadistuse nimest saavad lainepikkuste positsioonid

Kui sisestate lihtsalt nime, näiteks `NDVI`, peab Chloros otsustama, millise faili millist kanalit iga sümbol loeb. Selleks kasutatakse järgmist tabelit, mis seostab filtri koodi iga kanali massiivi positsiooniga:

| Filtri kood | Kanal → massiivi indeks |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` aktsepteeritakse Orange'i aliasena, samuti 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Eelseadistuse **vaikimisi filtrit** (ülal olev veerg „Vaikimisi filter”) kasutatakse siis, kui projekt sisaldab selle filtriga pilte. Kui selliseid pilte ei ole, skannib Chloros projektis tegelikult olemasolevaid filtreid järjekorras `RGN, OCN, NGB, RGB, RE, NIR` ja valib esimese, mis suudab pakkuda kõiki eelseadistuse jaoks vajalikke kanaleid. Kui ükski neist seda ei suuda, jäetakse eelseadistus selle käivituse puhul kõrvale. Seetõttu annab `NDVI`, mida taotletakse ainult OCN-andmekogumil, ikkagi mõistliku tulemuse – see seostub OCN’i Orange- ja NIR-positsioonidega.

LATTICE M3C mudeli stringid kannavad filtrit, millel on eesliide `F` (`LATT-M3C-L41-FRGN`), kuid eesliide jäetakse ära, kui filtri kood loetakse pildilt välja, seega lahendab FRGN-kaamera selle eespool oleva rea `RGN` kaudu ja ei vaja erikohtlemist.

### 3. LATTICE indeksimootor (`lattice index --preset`, reaalajas indeksikalkulaator) — 22 eelseadistust

LATTICE-mootor töötab joondatud mitme sagedusribaga pildistakidega (reaalajas massiivid või eksporditud mitme sagedusribaga TIFF-failid) ja kasutab väiketähelisi kanalite nimesid (`red`, `green`, `blue`, `red_edge`, `nir`). Selle eelseadete nimekiri erineb kahest eelnevast:

| Eelseadistus | Valem | Kanalid |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | punane, lähi-infrapuna |
| GNDVI | `(nir - green) / (nir + green)` | roheline, lähi-infrapuna |
| BNDVI | `(nir - blue) / (nir + blue)` | sinine, lähi-infrapuna |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | punane_serv, lähi-infrapuna |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | sinine, roheline, lähi-infrapuna |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | punane, lähi-infrapuna |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | punane, infrapuna |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | punane, infrapuna |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | sinine, punane, infrapuna |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | punane, lähi-infrapuna |
| CVI | `nir * red / green**2` | punane, roheline, lähi-infrapuna |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | punane, lähi-infrapuna |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | punane, infrapuna |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | sinine, punane, infrapuna |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | punane, roheline, sinine |
| NGRDI | `(green - red) / (green + red)` | punane, roheline |
| VARI | `(green - red) / (green + red - blue)` | punane, roheline, sinine |
| TGI | `green - 0.39*red - 0.61*blue` | punane, roheline, sinine |
| EXG | `2*green - red - blue` | punane, roheline, sinine |
| CIRE | `(nir / red_edge) - 1` | punane\_serv, nir |
| CIGREEN | `(nir / green) - 1` | roheline, nir |
| NDWI | `(green - nir) / (green + nir)` | roheline, nir |

Käivitage käsk `chloros-cli lattice index --list-presets`, et printida see tabel teie installitud versioonist, ja käivitage `--list-gradients`, et kuvada saadavalolevad värvigradiendid. Kanalite sümbolid on suurtähelised ja peavad vastama eelseadistuse väiketähelistele nimedele (nt `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI – klorofülli taimestiku indeks

See indeks on tundlik lehtede klorofüllisisalduse suhtes laias lehtkatte tiheduse vahemikus. See kasutab NIR-, punast ja rohelist laineala.

$$
CVI = NIR * {Red \over Green^{2}}
$$

GUI-s ning CLI- ja SDK-eelseadete loendis on sümboli valem `(z/y)*(x/y)`, kus x=Red, y=Green, z=NIR (RGN-kaamera lainepikkused); LATTICE indeksimootori eelseadistus on `nir * red / green**2`. Iga pind arvutab sama indeksit.

_Viide: Vincini, M., E. Frazzi ja P. D'Alessio. „A Broad-Band Leaf Chlorophyll Vegetation Index at the Canopy Scale.” Precision Agriculture 9 (2008): 303–319._

***

## ENDVI – täiustatud normaliseeritud erinevuse taimestikuindeks

See indeks kasutab lisaks NIR- ja rohelisele kanalile ka sinist kanalit ning on populaarne NGB-filtriga kaamerate puhul, kus sinine kanal asendab punast.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Rakendamiseks kasutatakse sümbolvalemit `((x+y)-(2*z))/((x+y)+(2*z))` – määrake oma kaamera NIR- ja Green-kanalid x/y-pesadesse ning Blue z-pesasse (NGB-kaamera puhul: x=NIR, y=Green, z=Blue).

***

## EVI – täiustatud taimestiku indeks

See indeks töötati algselt välja kasutamiseks MODIS-andmetega kui NDVI täiustus, optimeerides taimestiku signaali piirkondades, kus lehepinna indeks (LAI) on kõrge. See on kõige kasulikum piirkondades, kus LAI on kõrge ja NDVI võib küllastuda. See kasutab sinise peegelduspiirkonda, et korrigeerida pinnase taustsignaale ja vähendada atmosfääri mõjusid, sealhulgas aerosoolide hajumist.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI väärtused peaksid taimestiku pikslite puhul jääma vahemikku 0–1. Heledad objektid, nagu pilved ja valged hooned, ning tumedad objektid, nagu vesi, võivad põhjustada EVI-pildil ebanormaalsed pikseliväärtused. Enne EVI-pildi loomist tuleks peegelduspildilt eemaldada pilved ja heledad objektid ning soovi korral määrata pikselväärtuste künnis vahemikku 0–1.

_Viide: Huete, A. jt. „MODIS-i taimestikuindeksite radiomeetrilise ja biofüüsikalise toimivuse ülevaade.“ Remote Sensing of Environment 83 (2002): 195–213._

***

## FCI1 – metsakatvuse indeks 1

_Ainult GUI – ei ole saadaval CLI- või SDK-eelseadistusena `--indices`._

See indeks eristab metsakatet muudest taimestikutüüpidest, kasutades multispektraalset peegelduspilti, mis sisaldab punase serva laineala.

$$
FCI1 = Red * RedEdge
$$

Metsastatud aladel on madalamad FCI1-väärtused, kuna puude peegeldusvõime on madalam ja puuvõrade sees esinevad varjud.

_Viide: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. „Robust forest cover indices for multispectral images.” Photogrammetric Engineering & Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – metsakatvuse indeks 2

_Ainult GUI – ei ole saadaval CLI- või SDK-eelseadistusena `--indices`._

See indeks eristab metsa lehtkatteid muudest taimestikutüüpidest, kasutades multispektraalseid peegelduspilte, mis ei sisalda punase serva laineala.

$$
FCI2 = Red * NIR
$$

Metsastatud aladel on madalamad FCI2-väärtused, kuna puude peegeldusvõime on madalam ja puuvõrade sees esinevad varjud.

_Viide: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. „Robust forest cover indices for multispectral images.” Photogrammetric Engineering & Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – globaalne keskkonnaseireindeks

_Ainult GUI – ei ole saadaval CLI- või SDK-eelseadistusena `--indices`._

Seda mittelineaarset taimestikuindeksit kasutatakse satelliitpiltide põhjal globaalse keskkonna seireks ning sellega püütakse korrigeerida atmosfääri mõjusid. See sarnaneb NDVI-ga, kuid on atmosfääri mõjude suhtes vähem tundlik. Seda mõjutab paljas pinnas; seetõttu ei soovitata seda kasutada hõreda või mõõdukalt tiheda taimestikuga aladel.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kus:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Viide: Pinty, B. ja M. Verstraete. „GEMI: mittelineaarne indeks globaalse taimestiku seireks satelliitide abil“. Vegetation 101 (1992): 15–20._

***

## GARI – atmosfäärikindlusega rohelise indeks (Green)

_Ainult GUI – ei ole saadaval CLI- või SDK-eelseadistusena `--indices`._

See indeks on NDVI-st tundlikum laiale klorofülli kontsentratsioonide vahemikule ja vähem tundlik atmosfääri mõjude suhtes.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammakonstant on kaalufunktsioon, mis sõltub atmosfääri aerosoolitingimustest. ENVI kasutab väärtust 1,7, mis on Gitelsoni, Kaufmani ja Merzylaki soovitatud väärtus (1996, lk 296).

_Viide: Gitelson, A., Y. Kaufman ja M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.” Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – roheline klorofülli indeks (Green Chlorophyll Index)

Seda indeksit kasutatakse lehtede klorofüllisisalduse hindamiseks paljude erinevate taimeliikide puhul.

$$
GCI = {NIR \over Green} - 1
$$

Laiad NIR- ja rohelised lainepikkused võimaldavad klorofüllisisaldust paremini ennustada, pakkudes samal ajal suuremat tundlikkust ja paremat signaali-müra suhet.

_Viide: Gitelson, A., Y. Gritz ja M. Merzlyak. „Seosed lehe klorofüllisisalduse ja spektraalse peegeldusvõime vahel ning algoritmid kõrgemate taimede lehtede klorofülli mittetõrjuvaks hindamiseks.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – roheliste lehtede indeks (Green Leaf Index)

See indeks loodi algselt kasutamiseks digitaalse RGB-kaameraga nisu katvuse mõõtmiseks, kus punased, rohelised ja sinised digitaalsed numbrid (DN-id) jäävad vahemikku 0–255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-väärtused jäävad vahemikku -1 kuni +1. Negatiivsed väärtused tähistavad mulda ja elutuid objekte, positiivsed väärtused aga rohelisi lehti ja varre.

_Viide: Louhaichi, M., M. Borman ja D. Johnson. „Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.“ Geocarto International 16, nr 1 (2001): 65–70._

***

## GNDVI – normaliseeritud rohelise spektri taimestiku indeks (Green)

See indeks sarnaneb NDVI-ga, kuid erinevus seisneb selles, et see mõõdab rohelist spektrit vahemikus 540–570 nm punase spektri asemel. See indeks on klorofülli kontsentratsiooni suhtes tundlikum kui NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Viide: Gitelson, A. ja M. Merzlyak. „Klorofülli kontsentratsiooni kaugseire kõrgemate taimede lehtedes.“ Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – rohelisele optimeeritud, mulla suhtes korrigeeritud taimestiku indeks (Green Optimized Soil Adjusted Vegetation Index)

See indeks töötati algselt välja värvi-infrapunafotograafia abil, et ennustada maisi lämmastikuvajadust. See sarnaneb OSAVI-ga, kuid asendab punase laine rohelisega.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Viide: Sripada, R. jt. „Maisi hooajalise lämmastikuvajaduse määramine õhust tehtud värvi-infrapunafotode abil.“ Doktoritöö, Põhja-Carolina Riiklik Ülikool, 2005._

***

## GRVI – rohelise suhte taimestiku indeks (Green Ratio Vegetation Index)

See indeks on tundlik metsade lehtkatte fotosünteesikiiruse suhtes, kuna rohelise ja punase peegeldusvõimet mõjutavad tugevalt lehepigmentide muutused.

$$
GRVI = {NIR \over Green }
$$

_Viide: Sripada, R. jt. „Õhust tehtud värvi-infrapunafotograafia maisi hooaja alguses tekkivate lämmastikuvajaduste kindlaksmääramiseks.“ Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – rohelise pinnase suhtes korrigeeritud taimestiku indeks (Green Soil Adjusted Vegetation Index)

See indeks töötati algselt välja värvi-infrapunafotograafia abil maisi lämmastikuvajaduse ennustamiseks. See sarnaneb SAVI-ga, kuid asendab punase laine rohelisega.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Viide: Sripada, R. jt. „Maisi hooaja jooksul tekkiva lämmastikuvajaduse määramine õhust tehtud värvi-infrapunafotograafia abil.“ Doktoritöö, Põhja-Carolina Riiklik Ülikool, 2005._

***

## LAI – lehepinna indeks

Seda indeksit kasutatakse lehekatte hindamiseks ning põllukultuuri kasvu ja saagi prognoosimiseks. ENVI arvutab rohelise LAI väärtuse, kasutades järgmist empiirilist valemit, mis pärineb Boegh jt (2002) tööst:

$$
LAI = 3.618 * EVI - 0.118
$$

Kus EVI on:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Kõrged LAI-väärtused jäävad tavaliselt vahemikku umbes 0–3,5. Kui pildil on aga pilved ja muud heledad objektid, mis tekitavad küllastunud piksleid, võivad LAI-väärtused ületada 3,5. Ideaaljuhul tuleks enne LAI-pildi loomist pildilt pilved ja heledad objektid välja maskeerida.

_Viide: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde ja A. Thomsen. „Õhust kogutud multispektraalsed andmed lehepinnaindeksi, lämmastikukontsentratsiooni ja fotosünteetilise efektiivsuse kvantifitseerimiseks põllumajanduses.” Remote Sensing of Environment 81, nr 2–3 (2002): 179–193._

***

## LCI – lehe klorofülli indeks

_Ainult GUI – ei ole saadaval CLI- või SDK-eelseadistusena `--indices`._

Seda indeksit kasutatakse kõrgtaimede klorofüllisisalduse hindamiseks; see on tundlik klorofülli neeldumisest tingitud peegeldusvõime muutuste suhtes.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Viide: Datt, B. „Eukalüptilehtede veesisalduse kaugseire.“ Journal of Plant Physiology 154, nr 1 (1999): 30–36._

***

## MNLI – modifitseeritud mittelineaarne indeks

See indeks on mittelineaarse indeksi (NLI) täiustus, mis hõlmab mulla taustaga arvestamiseks mulla suhtes korrigeeritud taimestiku indeksit (SAVI). ENVI kasutab lehtkatte taustakorrigeerimisteguri (_L_) väärtust 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Viide: Yang, Z., P. Willis ja R. Mueller. „Band-Ratio Enhanced AWIFS-pildi mõju põllukultuuride klassifitseerimise täpsusele.“ Pecora 17 kaugseire sümpoosioni (2008) materjalid, Denver, CO._

***

## MSAVI2 – modifitseeritud pinnasega korrigeeritud taimestiku indeks 2

See indeks on Qi jt poolt välja pakutud MSAVI-indeksi lihtsam versioon (1994) poolt välja pakutud MSAVI indeksist, mis täiustab pinnasega korrigeeritud taimestiku indeksit (SAVI). See vähendab pinnasest tulenevat müra ja suurendab taimestiku signaali dünaamilist ulatust. MSAVI2 põhineb induktiivsel meetodil, mis ei kasuta konstantset _L_ väärtust (nagu SAVI puhul), et esile tuua tervet taimestikku.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Viide: Qi, J., A. Chehbouni, A. Huete, Y. Kerr ja S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – modifitseeritud lihtsuhe

See indeks on lihtsa NIR/Red-suhte modifikatsioon, mille eesmärk on lineariseerida selle seost biofüüsikaliste parameetritega, ning see on suurema taimestiku tiheduse korral tundlikum kui NDVI.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Viide: Chen, J. „Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.“ Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – normaliseeritud erinevus RedEdge

See indeks sarnaneb NDVI-ga, kuid võrdleb NIR ja RedEdge vahelist kontrasti Red asemel, mis tuvastab taimestiku stressi sageli varem.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – normaliseeritud erinevus taimestiku indeks

See indeks mõõdab tervet, rohelist taimestikku. Normaliseeritud erinevuse valemi ja klorofülli suurima neeldumis- ja peegelduspiirkondade kasutamise kombinatsioon muudab selle indeksi usaldusväärseks väga erinevates tingimustes. Siiski võib see tiheda taimestiku tingimustes küllastuda, kui LAI-väärtus muutub kõrgeks.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Selle indeksi väärtus jääb vahemikku -1 kuni 1. Rohelise taimestiku tavaline vahemik on 0,2 kuni 0,8.

_Viide: Rouse, J., R. Haas, J. Schell ja D. Deering. Taimestikusüsteemide seire Suurtel tasandikel ERTS-i abil. Kolmas ERTS-i sümpoosion, NASA (1973): 309–317._

***

## NLI – mittelineaarne indeks

See indeks eeldab, et paljude taimestikuindeksite ja pinnase biofüüsikaliste parameetrite vaheline seos on mittelineaarne. See lineariseerib seoseid pinnase parameetritega, mis on tavaliselt mittelineaarsed.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Viide: Goel, N. ja W. Qin. „Lehtkatte arhitektuuri mõju erinevate taimestiku indeksite ning LAI ja Fpar vahelistele seostele: arvutisimulatsioon.” Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – optimeeritud pinnasega korrigeeritud taimestiku indeks

See indeks põhineb pinnasega korrigeeritud taimestiku indeksil (SAVI). Lehtkatte taustakorrigeerimistegurina kasutatakse standardväärtust 0,16. Rondeaux (1996) leidis, et see väärtus võimaldab madala taimkatte puhul suuremat pinnasevariatsiooni kui SAVI, näidates samas suuremat tundlikkust taimkatte suhtes, mis ületab 50%. Seda indeksit on kõige parem kasutada piirkondades, kus taimkate on suhteliselt hõre ja pinnas on lehtkatte kaudu nähtav.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Viide: Rondeaux, G., M. Steven ja F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – renormaliseeritud erinevusvegetatsiooniindeks

See indeks kasutab lähi-infrapuna- ja punaste lainepikkuste vahet koos NDVI-ga, et esile tuua tervet taimestikku. See ei ole tundlik mulla ega päikese vaatlusgeomeetria mõjude suhtes.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Viide: Roujean, J. ja F. Breon. „Taimestiku poolt neeldunud PAR-i hindamine kahesuunaliste peegeldusmõõtmiste abil.“ Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – pinnasega korrigeeritud taimestiku indeks

See indeks sarnaneb NDVI-ga, kuid see vähendab pinnasepikslite mõju. Selles kasutatakse lehtkatte taustakorrigeerimistegurit _L_, mis sõltub taimestiku tihedusest ja nõuab sageli eelnevat teavet taimestiku hulgast. Huete (1988) soovitab optimaalseks väärtuseks _L_=0,5, et arvesse võtta esimese järgu pinnase taustamuutusi. Seda indeksit on kõige parem kasutada piirkondades, kus taimestik on suhteliselt hõre ja pinnas on lehtkatte kaudu nähtav.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Viide: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).” Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – transformeeritud erinevuste taimestikuindeks

See indeks on kasulik taimkatte seireks linnakeskkonnas. See ei küllastu nagu NDVI ja SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Viide: Bannari, A., H. Asalhi ja P. Teillet. „Transformeeritud erinevuse taimestikuindeks (TDVI) taimkatte kaardistamiseks“ kogumikus „Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS '02, IEEE International“, 5. köide (2002)._

***

## VARI – nähtava valguse atmosfäärikindel indeks

See indeks põhineb ARVI-l ja seda kasutatakse taimestiku osakaalu hindamiseks pildil, kus on madal tundlikkus atmosfääriliste mõjude suhtes.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Viide: Gitelson, A., jt. „Taimestik ja pinnaseliinid nähtavas spektriruumis: kontseptsioon ja meetod taimestiku osakaalu kaughinnanguks“. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – laia dünaamilise ulatusega taimestiku indeks

See indeks sarnaneb NDVI-ga, kuid kasutab kaalukoefitsienti (_a_), et vähendada lähi-infrapuna- ja punaste signaalide panuse vahelist erinevust NDVI-s. WDRVI on eriti efektiivne stseenides, kus taimestiku tihedus on mõõdukas kuni kõrge ja NDVI ületab 0,6. NDVI kipub stabiliseeruma, kui taimestiku osakaal ja lehepinna indeks (LAI) suurenevad, samas kui WDRVI on tundlikum laiemale taimestiku osakaalude vahemikule ja LAI muutustele.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Kaalukoefitsient (_a_) võib olla vahemikus 0,1–0,2. Henebry, Viña ja Gitelson (2004) soovitavad väärtust 0,2.

_Viited_

_Gitelson, A. „Laiadünaamilise ulatusega taimestiku indeks taimestiku biofüüsikaliste omaduste kaugmõõtmiseks.“ Journal of Plant Physiology 161, nr 2 (2004): 165–173._

_Henebry, G., A. Viña ja A. Gitelson. „Laiadünaamilise ulatusega taimestiku indeks ja selle potentsiaalne kasulikkus lünkade analüüsis.“ Gap Analysis Bulletin 12: 50–56._
