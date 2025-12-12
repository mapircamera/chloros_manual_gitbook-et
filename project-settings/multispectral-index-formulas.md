---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektraalsed indeksivalemid

Allpool esitatud indeksivalemites kasutatakse Survey3 filtri keskmise läbilaskvuse vahemike kombinatsiooni:

<table><thead><tr><th align="center">Survey3 Filtri värv</th><th width="196.199951171875" align="center">Survey3 Filtri nimi</th><th width="159.800048828125" align="center">Läbilaskvusvahemik (FWHM)</th><th align="center">Keskmine läbilaskvus</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Kui kasutatakse neid valemeid, võib nimi lõppeda „\_1” või „\_2”ga, mis vastab sellele, millist NIR filtrit, kas NIR1 või NIR2, kasutati.

***

## EVI – täiustatud taimestiku indeks

See indeks töötati algselt välja kasutamiseks koos MODIS andmetega, et täiustada NDVI indeksit, optimeerides taimestiku signaali piirkondades, kus on kõrge lehepinna indeks (LAI). See on kõige kasulikum kõrge LAI piirkondades, kus NDVI võib küllastuda. See kasutab sinise peegelduspiirkonda, et korrigeerida pinnase taustsignaale ja vähendada atmosfääri mõjusid, sealhulgas aerosoolide hajumist.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI väärtused peaksid taimestiku pikslite puhul olema vahemikus 0 kuni 1. Heledad objektid, nagu pilved ja valged hooned, ning tumedad objektid, nagu vesi, võivad põhjustada EVI pildil ebanormaalsed piksliväärtused. Enne EVI pildi loomist tuleks peegelduselt eemaldada pilved ja heledad objektid ning valikuliselt määrata pikselite väärtuste künniseks 0 kuni 1.

_Viide: Huete, A. jt. „Ülevaade MODIS taimestiku indeksite radiomeetrilisest ja biofüüsilisest toimivusest”. Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Metsakatte indeks 1

See indeks eristab metsakatet muudest taimestikutüüpidest, kasutades multispektraalset peegelduspilti, mis sisaldab punast serva.

$$
FCI1 = Red * RedEdge
$$

Metsastatud aladel on FCI1 väärtused madalamad, kuna puud peegeldavad vähem ja lehtede vahel on varjud.

_Viide: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. „Robust forest cover indices for multispectral images.” Fotogramm-meetria ja kaugseire 84.8 (2018): 505-512._

***

## FCI2 – Metsakatte indeks 2

See indeks eristab metsakatteid muudest taimestikutüüpidest, kasutades multispektraalset peegelduspilti, mis ei sisalda punast serva.

$$
FCI2 = Red * NIR
$$

Metsastatud aladel on FCI2 väärtused madalamad, kuna puude peegeldusvõime on madalam ja puude latvades on varjud.

_Viide: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. „Robust forest cover indices for multispectral images” (Tugevad metsade katvuse indeksid multispektraalsete piltide jaoks). Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – globaalne keskkonnaseire indeks

Seda mittelineaarset taimestiku indeksit kasutatakse globaalse keskkonnaseireks satelliitpiltide abil ja sellega püütakse korrigeerida atmosfääri mõjusid. See on sarnane NDVI-ga, kuid on atmosfääri mõjudele vähem tundlik. Seda mõjutab paljas pinnas, seetõttu ei soovitata seda kasutada hõreda või mõõdukalt tiheda taimestikuga aladel.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kus:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Viide: Pinty, B. ja M. Verstraete. GEMI: mittelineaarne indeks globaalse taimestiku seireks satelliitide abil. Vegetation 101 (1992): 15-20._

***

## GARI - Green Atmosfäärikindel indeks

See indeks on tundlikum laia klorofülli kontsentratsioonide vahemiku suhtes ja vähem tundlik atmosfääri mõjude suhtes kui NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammakonstant on kaalufunktsioon, mis sõltub atmosfääri aerosoolide tingimustest. ENVI kasutab väärtust 1,7, mis on Gitelsoni, Kaufmani ja Merzylaki (1996, lk 296) soovitatud väärtus.

_Viide: Gitelson, A., Y. Kaufman ja M. Merzylak. „Green kanali kasutamine globaalse taimestiku kaugseires EOS-MODIS abil.” Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green klorofülli indeks

Seda indeksit kasutatakse lehtede klorofüllisisalduse hindamiseks paljude taimeliikide puhul.

$$
GCI = {NIR \over Green} - 1
$$

Lai NIR ja roheliste lainepikkustega on võimalik klorofüllisisaldust paremini ennustada, samal ajal tagades suurema tundlikkuse ja parema signaali-müra suhte.

_Viide: Gitelson, A., Y. Gritz ja M. Merzlyak. „Lehtede klorofüllisisalduse ja spektraalse peegeldusvõime vahelised seosed ning algoritmid kõrgemate taimede lehtede klorofüllisisalduse mittetõrjuva hindamise jaoks.” Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Leheindeks

See indeks oli algselt mõeldud kasutamiseks digitaalse RGB kaameraga nisu katvuse mõõtmiseks, kus punased, rohelised ja sinised digitaalarvud (DN) on vahemikus 0–255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI väärtused ulatuvad -1-st +1-ni. Negatiivsed väärtused tähistavad mulda ja elutuid objekte, positiivsed väärtused aga rohelisi lehti ja varre.

_Viide: Louhaichi, M., M. Borman ja D. Johnson. „Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.” Geocarto International 16, nr 1 (2001): 65–70._

***

## GNDVI – Green normaliseeritud taimestiku erinevusindeks

See indeks on sarnane NDVI-ga, välja arvatud see, et see mõõdab rohelist spektrit 540–570 nm asemel punast spektrit. See indeks on klorofülli kontsentratsiooni suhtes tundlikum kui NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Viide: Gitelson, A. ja M. Merzlyak. „Klorofülli kontsentratsiooni kaugseire kõrgemate taimede lehtedes.” Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimeeritud pinnasekorrigeeritud taimestiku indeks

See indeks loodi algselt värvilise infrapunafotograafia abil, et ennustada maisi lämmastikuvajadust. See on sarnane OSAVI-ga, kuid asendab rohelise sagedusriba punasega.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Viide: Sripada, R., et al. „Maisi hooajalise lämmastikuvajaduse määramine õhust tehtud värvilise infrapunafotograafia abil.” Doktori väitekiri, Põhja-Carolina Riiklik Ülikool, 2005._

***

## GRVI – Green suhtarvu taimestiku indeks

See indeks on tundlik metsade latvade fotosünteesi kiiruse suhtes, kuna rohelise ja punase peegeldusvõime mõjutavad tugevalt lehtede pigmentide muutused.

$$
GRVI = {NIR \over Green }
$$

_Viide: Sripada, R. jt. „Õhust tehtud värviline infrapunafotograafia maisi varase hooaja lämmastikuvajaduse kindlaksmääramiseks.” Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Pinnasega korrigeeritud taimestiku indeks

See indeks loodi algselt värvilise infrapunafotograafia abil maisi lämmastiku vajaduse ennustamiseks. See on sarnane SAVI-ga, kuid asendab rohelise laine punasega.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Viide: Sripada, R., et al. „Maisi hooajalise lämmastikuvajaduse määramine õhust tehtud värvilise infrapunafotograafia abil.” Doktoriõpe, Põhja-Carolina Riiklik Ülikool, 2005._

***

## LAI – lehepinna indeks

Seda indeksit kasutatakse lehtede katvuse hindamiseks ning põllukultuuri kasvu ja saagikuse prognoosimiseks. ENVI arvutab rohelise LAI järgmise Boegh et al (2002) empiirilise valemi abil:

$$
LAI = 3.618 * EVI - 0.118
$$

Kus EVI on:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Kõrged LAI väärtused on tavaliselt vahemikus 0 kuni 3,5. Kui aga pildil on pilved ja muud heledad objektid, mis tekitavad küllastunud piksleid, võivad LAI väärtused ületada 3,5. Ideaaljuhul peaksite enne LAI-pildi loomist maskeerima stseenist pilved ja heledad objektid.

_Viide: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde ja A. Thomsen. „Õhust kogutud multispektraalsed andmed lehepinna indeksi, lämmastiku kontsentratsiooni ja fotosünteesi efektiivsuse kvantifitseerimiseks põllumajanduses.” Remote Sensing of Environment 81, nr 2-3 (2002): 179-193._

***

## LCI – lehe klorofülli indeks

Seda indeksit kasutatakse klorofülli sisalduse hindamiseks kõrgemates taimedes, mis on tundlikud klorofülli neeldumise põhjustatud peegeldusvõime muutuste suhtes.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Viide: Datt, B. „Eukalüptuse lehtede veesisalduse kaugseire.” Journal of Plant Physiology 154, nr 1 (1999): 30–36._

***

## MNLI – modifitseeritud mittelineaarne indeks

See indeks on mittelineaarse indeksi (NLI) täiustus, mis hõlmab pinnase kohandatud taimestiku indeksit (SAVI), et arvesse võtta pinnase tausta. ENVI kasutab lehtede taustakohandusteguri (_L_) väärtust 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Viide: Yang, Z., P. Willis ja R. Mueller. „Bändisuhtega täiustatud AWIFS-pildi mõju põllukultuuride klassifitseerimise täpsusele.” Pecora 17 kaugseire sümpoosioni (2008) protokoll, Denver, CO._

***

## MSAVI2 – modifitseeritud pinnase kohandatud taimestiku indeks 2

See indeks on Qi jt (1994) poolt välja pakutud indeksi MSAVI lihtsam versioon, mis täiustab pinnase kohandatud taimestiku indeksit (SAVI). See vähendab pinnase müra ja suurendab taimestiku signaali dünaamilist ulatust. MSAVI2 põhineb induktiivsel meetodil, mis ei kasuta konstantset _L_ väärtust (nagu SAVI), et esile tuua tervet taimestikku.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Viide: Qi, J., A. Chehbouni, A. Huete, Y. Kerr ja S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119–126._

***

## NDRE – normaliseeritud erinevus RedEdge

See indeks on sarnane NDVI-ga, kuid võrdleb kontrasti NIR ja RedEdge vahel, mitte Red-ga, mis tihti tuvastab taimestiku stressi varem.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – normaliseeritud erinevus taimestiku indeks

See indeks on tervisliku, rohelise taimestiku mõõdik. Normaliseeritud erinevuse valemi ja klorofülli kõrgeima neeldumis- ja peegeldumispiirkonna kasutamise kombinatsioon muudab selle indeksi mitmesugustes tingimustes töökindlaks. Siiski võib see tiheda taimestiku tingimustes küllastuda, kui LAI muutub kõrgeks.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Selle indeksi väärtus on vahemikus -1 kuni 1. Rohelise taimestiku tavaline vahemik on 0,2 kuni 0,8.

_Viide: Rouse, J., R. Haas, J. Schell ja D. Deering. Taimestiku süsteemide seire Suurel tasandikul ERTS-i abil. Kolmas ERTS-i sümpoosion, NASA (1973): 309–317._

***

## NLI – mittelineaarne indeks

See indeks eeldab, et paljude taimestiku indeksite ja pinnaja biofüüsikaliste parameetrite vaheline suhe on mittelineaarne. See lineariseerib suhted pinnaparametritega, mis on tavaliselt mittelineaarsed.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Viide: Goel, N. ja W. Qin. „Lehtede struktuuri mõju erinevate taimestiku indeksite ja LAI ning Fpar vahelistele suhetele: arvutisimulatsioon.” Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – optimeeritud pinnasekorrigeeritud taimestiku indeks

See indeks põhineb pinnasekorrigeeritud taimestiku indeksil (SAVI). See kasutab lehtede taustakorrigeerimisteguri standardväärtust 0,16. Rondeaux (1996) leidis, et see väärtus annab madala taimkatte puhul suurema pinnase variatsiooni kui SAVI, näidates samal ajal suuremat tundlikkust taimkatte suhtes, mis on suurem kui 50%. Seda indeksit on kõige parem kasutada piirkondades, kus taimkate on suhteliselt hõre ja pinnas on lehtede kaudu nähtav.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Viide: Rondeaux, G., M. Steven ja F. Baret. „Optimization of Soil-Adjusted Vegetation Indices” (Pinnase kohandatud taimestiku indeksite optimeerimine). Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – renormaliseeritud taimestiku erinevusindeks

See indeks kasutab lähi-infrapuna ja punase lainepikkuse vahet koos NDVI-ga, et esile tuua terve taimestik. See on tundetu pinnase ja päikese vaatamisgeomeetria mõjude suhtes.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Viide: Roujean, J. ja F. Breon. „Hinnang PAR-i neeldumisele taimestikus kahepoolsete peegeldusmõõtmiste põhjal.” Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – pinnasega korrigeeritud taimestiku indeks

See indeks on sarnane NDVI-ga, kuid see summutab pinnase pikslite mõju. See kasutab lehtede taustakorrigeerimistegurit _L_, mis on taimestiku tiheduse funktsioon ja nõuab sageli eelnevat teadmist taimestiku kogusest. Huete (1988) soovitab optimaalseks väärtuseks _L_=0,5, et arvesse võtta esimese järgu pinnase taustamuutusi. Seda indeksit on kõige parem kasutada piirkondades, kus taimestik on suhteliselt hõre ja pinnas on lehtede kaudu nähtav.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Viide: Huete, A. „Pinnasega korrigeeritud taimestiku indeks (SAVI).” Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – transformeeritud erinevusvegetatsiooni indeks

Seda indeksit on kasulik kasutada linnakeskkonna taimkatte seireks. See ei küllasta nagu NDVI ja SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Viide: Bannari, A., H. Asalhi ja P. Teillet. „Transformeeritud erinevuste taimestiku indeks (TDVI) taimestiku katte kaardistamiseks” Geoteaduste ja kaugseire sümpoosioni, IGARSS &#x27;02, IEEE International, 5. köide (2002) materjalides._

***

## VARI – nähtav atmosfäärikindel indeks

See indeks põhineb ARVI-il ja seda kasutatakse atmosfääri mõjule vähe tundliku stseeni taimestiku osakaalu hindamiseks.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Viide: Gitelson, A., et al. „Vegetatsioon ja pinnaseliinid nähtavas spektraalses ruumis: kontseptsioon ja tehnika vegetatsiooni osakaalu kaughindamiseks. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – laia dünaamilise ulatusega taimestiku indeks

See indeks on sarnane NDVI-ga, kuid kasutab kaalukoefitsienti (_a_), et vähendada lähi-infrapuna- ja punaste signaalide osakaalu erinevust NDVI-s. WDRVI on eriti efektiivne stseenides, kus taimestiku tihedus on keskmine kuni kõrge, kui NDVI ületab 0,6. NDVI kipub tasanduma, kui taimestiku osakaal ja lehepinna indeks (LAI) suurenevad, samas kui WDRVI on tundlikum laiemale taimestiku osakaalu vahemikule ja muutustele LAI-is.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Kaalukoefitsient (_a_) võib olla vahemikus 0,1–0,2. Henebry, Viña ja Gitelson (2004) soovitavad väärtust 0,2.

_Viited_

_Gitelson, A. „Lai dünaamiline vahemik taimestiku indeks taimestiku biofüüsikaliste omaduste kaugkvantifitseerimiseks.&quot; Journal of Plant Physiology 161, nr 2 (2004): 165-173._

_Henebry, G., A. Viña ja A. Gitelson. &quot;Lai dünaamilise ulatusega taimestiku indeks ja selle potentsiaalne kasulikkus lünkade analüüsis.&quot; Gap Analysis Bulletin 12: 50-56._
