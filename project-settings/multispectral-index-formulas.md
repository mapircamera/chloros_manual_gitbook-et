---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektraalsete indeksite valemid

Allpool toodud indeksite valemites kasutatakse Survey3 filtri keskmise läbilaskvuse vahemike kombinatsiooni:

<table><thead><tr><th align="center">Survey3 filtri värv</th><th width="196.199951171875" align="center">Survey3 Filtri nimi</th><th width="159.800048828125" align="center">Läbilaskvusvahemik (FWHM)</th><th align="center">Keskmine läbilaskvus</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN– Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Kui kasutatakse neid valemeid, võib nimi lõppeda „\_1” või „\_2”ga, mis vastab sellele, millist NIR filtrit, kas NIR1 või NIR2, kasutati.

***

## EVI – täiustatud taimestiku indeks

See indeks töötati algselt välja kasutamiseks MODIS andmetega kui täiustus NDVI-ile, optimeerides taimestiku signaali piirkondades, kus on kõrge lehepinna indeks (LAI). See on kõige kasulikum piirkondades, kus LAI on kõrge ja kus NDVI võib olla küllastunud. See kasutab sinise peegelduse piirkonda, et korrigeerida pinnase taustsignaale ja vähendada atmosfääri mõjusid, sealhulgas aerosoolide hajumist.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI väärtused peaksid taimestiku pikslite puhul jääma vahemikku 0–1. Heledad objektid, nagu pilved ja valged hooned, ning tumedad objektid, nagu vesi, võivad põhjustada ebanormaalsed pikseliväärtused EVI pildil. Enne EVI-pildi loomist tuleks peegelduspildilt eemaldada pilved ja heledad objektid ning soovi korral määrata pikselväärtuste künnis vahemikku 0–1.

_Viide: Huete, A. jt. „Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Metsakatte indeks 1

See indeks eristab metsakrooni muudest taimestikutüüpidest, kasutades multispektraalset peegelduspilti, mis sisaldab punase serva laineala.

$$
FCI1 = Red * RedEdge
$$

Metsaga kaetud aladel on madalamad FCI1 väärtused, kuna puude peegeldusvõime on madalam ja puuvõrade sees on varju.

_Viide: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. „Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Metsakatte indeks 2

See indeks eristab metsa latvust muudest taimestikutüüpidest, kasutades multispektraalset peegelduspilti, mis ei sisalda punast serva sagedusriba.

$$
FCI2 = Red * NIR
$$

Metsapiirkondadel on madalamad FCI2 väärtused, kuna puude peegeldusvõime on madalam ja latvuses esinevad varjud.

_Viide: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. „Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – globaalne keskkonnaseire indeks

Seda mittelineaarset taimestiku indeksit kasutatakse satelliidipiltide põhjal globaalse keskkonna seireks ning sellega püütakse korrigeerida atmosfääri mõjusid. See on sarnane indeksiga NDVI, kuid on atmosfääri mõjude suhtes vähem tundlik. Seda mõjutab paljas pinnas; seetõttu ei soovitata seda kasutada hõreda või mõõdukalt tiheda taimestikuga aladel.

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

See indeks on tundlikum laiale klorofülli kontsentratsioonide vahemikule ja vähem tundlik atmosfääri mõjude suhtes kui NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammakonstant on kaalufunktsioon, mis sõltub atmosfääri aerosoolitingimustest. ENVI kasutab väärtust 1,7, mis on Gitelsoni, Kaufmani ja Merzylaki (1996, lk 296) soovitatud väärtus.

_Viide: Gitelson, A., Y. Kaufman ja M. Merzylak. „Green kanali kasutamine globaalse taimestiku kaugseires EOS-MODIS abil.” Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green klorofülli indeks

Seda indeksit kasutatakse lehtede klorofüllisisalduse hindamiseks paljude taimeliikide puhul.

$$
GCI = {NIR \over Green} - 1
$$

Laiad NIR ja rohelised lainepikkused võimaldavad klorofüllisisaldust paremini ennustada, pakkudes samal ajal suuremat tundlikkust ja paremat signaali-müra suhet.

_Viide: Gitelson, A., Y. Gritz ja M. Merzlyak. „Lehe klorofüllisisalduse ja spektraalse peegeldusvõime vahelised seosed ning algoritmid kõrgemate taimede lehtede klorofülli mittetõkestavaks hindamiseks.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green leheindeks

See indeks oli algselt mõeldud kasutamiseks digitaalkaameraga RGB nisu katvuse mõõtmiseks, kus punased, rohelised ja sinised digitaalsed numbrid (DN) ulatuvad vahemikust 0 kuni 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI väärtused jäävad vahemikku -1 kuni +1. Negatiivsed väärtused tähistavad mulda ja elutut, positiivsed väärtused aga rohelisi lehti ja varre.

_Viide: Louhaichi, M., M. Borman ja D. Johnson. „Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.” Geocarto International 16, nr 1 (2001): 65–70._

***

## GNDVI – Green normaliseeritud taimestiku erinevusindeks

See indeks on sarnane indeksiga NDVI, välja arvatud see, et see mõõdab rohelist spektrit vahemikus 540–570 nm punase spektri asemel. See indeks on klorofülli kontsentratsiooni suhtes tundlikum kui indeks NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Viide: Gitelson, A. ja M. Merzlyak. „Klorofülli kontsentratsiooni kaugseire kõrgemate taimede lehtedes.” Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimeeritud pinnasega korrigeeritud taimestiku indeks

See indeks loodi algselt värvi-infrapunafotograafia abil maisi lämmastikuvajaduse prognoosimiseks. See sarnaneb indeksiga OSAVI, kuid asendab rohelise lainepikkuse punasega.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Viide: Sripada, R. jt. „Maisi hooajalise lämmastikuvajaduse määramine õhust tehtud värvi-infrapunafotode abil.” Doktoritöö, Põhja-Carolina Riiklik Ülikool, 2005._

***

## GRVI – Green suhtarvuline taimestiku indeks

See indeks on tundlik metsade latvade fotosünteesikiiruse suhtes, kuna rohelise ja punase peegeldusvõimet mõjutavad tugevalt lehepigmentide muutused.

$$
GRVI = {NIR \over Green }
$$

_Viide: Sripada, R. jt. „Aerial Color Infrared Photography for Determining Early In-season Nitrogen Requirements in Corn.” Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green mulla suhtes korrigeeritud taimestiku indeks

See indeks loodi algselt värvi-infrapunafotograafia abil maisi lämmastikuvajaduse prognoosimiseks. See sarnaneb indeksiga SAVI, kuid asendab rohelise lainepikkuse punasega.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Viide: Sripada, R. jt. „Maisi hooajalise lämmastikuvajaduse määramine õhust tehtud värvi-infrapunafotode abil.” Doktoritöö, Põhja-Carolina Riiklik Ülikool, 2005._

***

## LAI – Lehepinna indeks

Seda indeksit kasutatakse lehekatte hindamiseks ning põllukultuuride kasvu ja saagi prognoosimiseks. ENVI arvutab rohelise LAI väärtuse, kasutades järgmist Boegh jt (2002) empiirilist valemit:

$$
LAI = 3.618 * EVI - 0.118
$$

Kus EVI on:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Kõrged LAI väärtused jäävad tavaliselt vahemikku umbes 0 kuni 3,5. Kui pildil on aga pilved ja muud heledad objektid, mis tekitavad küllastunud piksleid, võivad LAI väärtused ületada 3,5. Ideaaljuhul peaksite enne LAI pildi loomist pildilt pilved ja heledad objektid välja maskeerima.

_Viide: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde ja A. Thomsen. „Õhust kogutud multispektraalsed andmed lehepinna indeksi, lämmastikukontsentratsiooni ja fotosünteetilise efektiivsuse kvantifitseerimiseks põllumajanduses.” Remote Sensing of Environment 81, nr 2–3 (2002): 179–193._

***

## LCI – Leheklorofülli indeks

Seda indeksit kasutatakse kõrgemate taimede klorofüllisisalduse hindamiseks, kuna see on tundlik klorofülli neeldumisest tingitud peegeldusvõime muutuste suhtes.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Viide: Datt, B. „Eukalüptilehtede veesisalduse kaugseire.” Journal of Plant Physiology 154, nr 1 (1999): 30–36._

***

## MNLI – modifitseeritud mittelineaarne indeks

See indeks on mittelineaarse indeksi (NLI) täiustus, mis hõlmab pinnase suhtes korrigeeritud taimestiku indeksit (SAVI), et arvesse võtta pinnase taustat. ENVI kasutab lehtkatte taustakorrigeerimisteguri (_L_) väärtust 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Viide: Yang, Z., P. Willis ja R. Mueller. „Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.” Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – modifitseeritud pinnase korrigeeritud taimestiku indeks 2

See indeks on Qi jt (1994) poolt välja pakutud indeksist MSAVI lihtsam versioon, mis täiustab pinnase korrigeeritud taimestiku indeksit (SAVI). See vähendab pinnase müra ja suurendab taimestiku signaali dünaamilist ulatust. MSAVI2 põhineb induktiivsel meetodil, mis ei kasuta konstantset _L_ väärtust (nagu SAVI puhul), et esile tuua tervet taimestikku.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Viide: Qi, J., A. Chehbouni, A. Huete, Y. Kerr ja S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119–126._

***

## NDRE – normaliseeritud erinevus RedEdge

See indeks on sarnane indeksiga NDVI, kuid võrdleb kontrasti indeksite NIR ja RedEdge vahel, mitte indeksiga Red, mis tuvastab taimestiku stressi sageli varem.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – normaliseeritud taimestiku erinevuse indeks

See indeks mõõdab tervet, rohelist taimestikku. Normaliseeritud erinevuse valemi ja klorofülli kõrgeima neeldumis- ja peegelduspiirkonna kasutamise kombinatsioon muudab selle indeksit stabiilseks laias tingimuste vahemikus. Siiski võib see tiheda taimestiku tingimustes küllastuda, kui LAI muutub kõrgeks.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Selle indeksi väärtus jääb vahemikku -1 kuni 1. Rohelise taimestiku tavaline vahemik on 0,2 kuni 0,8.

_Viide: Rouse, J., R. Haas, J. Schell ja D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309–317._

***

## NLI – mittelineaarne indeks

See indeks eeldab, et paljude taimestikuindeksite ja pinnase biofüüsikaliste parameetrite vaheline suhe on mittelineaarne. See lineariseerib suhted pinnase parameetritega, mis on tavaliselt mittelineaarsed.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Viide: Goel, N. ja W. Qin. „Lehtkatte arhitektuuri mõju erinevate taimestiku indeksite ning LAI ja Fpar vahelistele suhetele: arvutisimulatsioon.” Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimeeritud pinnasega korrigeeritud taimestiku indeks

See indeks põhineb pinnasega korrigeeritud taimestiku indeksil (SAVI). See kasutab lehtkatte taustakorrigeerimistegurina standardväärtust 0,16. Rondeaux (1996) leidis, et see väärtus tagab madala taimkatte puhul suurema pinnase variatsiooni kui SAVI, näidates samas suuremat tundlikkust taimkatte suhtes, mis on suurem kui 50%. Seda indeksit on kõige parem kasutada piirkondades, kus taimkate on suhteliselt hõre ja kus pinnas on lehtkatte kaudu nähtav.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Viide: Rondeaux, G., M. Steven ja F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – renormaliseeritud erinevuse taimestiku indeks

See indeks kasutab lähi-infrapuna- ja punase lainepikkuste vahet koos NDVI-iga, et esile tuua tervet taimestikku. See ei ole tundlik mulla ja päikese vaatamisgeomeetria mõjude suhtes.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Viide: Roujean, J. ja F. Breon. „Taimestiku poolt neeldunud PAR-i hindamine kahepoolse peegeldusmõõtmiste abil.” Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Pinnase suhtes korrigeeritud taimestiku indeks

See indeks on sarnane indeksiga NDVI, kuid see summutab pinnase pikslite mõju. Selles kasutatakse lehtkatte taustakorrigeerimistegurit _L_, mis on taimestiku tiheduse funktsioon ja nõuab sageli eelnevat teadmist taimestiku hulgast. Huete (1988) soovitab optimaalseks väärtuseks _L_=0,5, et arvesse võtta esimese järgu pinnase taustamuutusi. Seda indeksit on kõige parem kasutada piirkondades, kus taimestik on suhteliselt hõre ja pinnas on lehtkatte kaudu nähtav.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Viide: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – Transformeeritud erinevuste taimestiku indeks

See indeks on kasulik taimkatte seireks linnakeskkonnas. See ei küllastu nagu NDVI ja SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Viide: Bannari, A., H. Asalhi ja P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” (Transformeeritud taimestiku erinevuse indeks (TDVI) taimkatte kaardistamiseks) Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, 5. köide (2002)._

***

## VARI – nähtav atmosfäärikindel indeks

See indeks põhineb indeksil ARVI ja seda kasutatakse taimestiku osakaalu hindamiseks pildil, kus tundlikkus atmosfääri mõjude suhtes on madal.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Viide: Gitelson, A. jt. „Taimestiku ja pinnase piirid nähtavas spektriruumis: kontseptsioon ja meetod taimestiku osakaalu kaughindamiseks. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – laia dünaamilise ulatusega taimestiku indeks

See indeks on sarnane indeksiga NDVI, kuid kasutab kaalukoefitsienti (_a_), et vähendada lähi-infrapuna- ja punaste signaalide panuse erinevust indeksis NDVI. WDRVI on eriti efektiivne stseenides, kus taimestiku tihedus on mõõdukas kuni kõrge, kui NDVI ületab 0,6. NDVI kipub tasapisi vähenema, kui taimestiku osakaal ja lehepinna indeks (LAI) suurenevad, samas kui WDRVI on tundlikum laiemale taimestiku osakaalude vahemikule ja muutustele LAI-is.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Kaalukoefitsient (_a_) võib olla vahemikus 0,1–0,2. Henebry, Viña ja Gitelson (2004) soovitavad väärtust 0,2.

_Viited_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.” Journal of Plant Physiology 161, nr 2 (2004): 165–173._

_Henebry, G., A. Viña ja A. Gitelson. „Laiadünaamilise ulatusega taimestiku indeks ja selle potentsiaalne kasulikkus lünkade analüüsis.” Gap Analysis Bulletin 12: 50–56._
