# Projekti seadete kohandamine

Enne piltide töötlemist on oluline konfigureerida projekti seaded vastavalt oma töövoo nõuetele. Projekti seadete paneel <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> võimaldab põhjalikku kontrolli kalibreerimise, töötlemisvalikute, multispektraalsete indeksite ja ekspordiformaatide üle.

## Projektiseadete avamine

1. Avage oma projekt programmis Chloros
2. Klõpsake vasakul külgribal ikooni **Projektiseaded** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">
3. Paneelis „Projektiseaded“ kuvatakse kõik seadistusvõimalused

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Paneel „Projekti seaded“ — kuvamine, sihtmärkide tuvastamine ja töötlemine</p></figcaption></figure>

{% hint style="info" %}
**Seaded salvestatakse automaatselt** koos projektiga. Kui avate projekti uuesti, taastatakse kõik seaded.
{% endhint %}

***

## Kiirhäälestus tavaliste töövoogude jaoks

### Vaikesätted (soovitatav enamikule kasutajatele)

Vaikesätted sobivad hästi tüüpiliste Survey3
- ja LATTICE-töövoogude jaoks:

* ✅ **Vignette-korrektsioon**: sisse lülitatud
* ✅ **Peegelduskalibreerimine / valgebalanss**: sisse lülitatud (kasutab MAPIR
-sihtmärke ja/või DAQ-valgusanduri andmeid)
* ✅ **Debayeri meetod**: standardne (kiire, keskmine kvaliteet)
* ✅ **Ekspordivorming**: TIFF
(16-bitine) (TIFF
(8-bitine) täielikult 8-bitise projekti puhul)
* ✅ **Kõik eksporditavad tulemused**: sisse lülitatud (LATTICE salvestab automaatselt debayered-, eelvaate-, kiirguse- ja peegeldusandmed)

Lihtsalt importige oma pildid ja alustage töötlemist nende vaikimisi seadetega.

***

## Projekti seaded – ülevaade

Projekti seaded on jagatud allpool toodud osadeks. Kaks lisajaotist — **DAQ valgusandur**ja**maatriksi joondamine** — ilmuvad automaatselt, kui teie projekt sisaldab vastavaid faile. Täieliku dokumentatsiooni leiate [Projekti seaded](../project-settings/project-settings.md).

### Kuvamine

* **Pildi pisipildi eraldusvõime**: Pildivõrgu pisipiltide eraldusvõime. Valikud:**Vaikimisi (512 px)**,**1024 px**,**2048 px**,**Täisresolutsioon**. Ainult kuvamiseks – ei mõjuta kunagi töötlemist. Suuremad väärtused näevad suurendatuna teravamad välja, kuid laadivad aeglasemalt.

### Sihtmärgi tuvastamine

Määrab, kuidas Chloros
tuvastab teie piltidel kalibreerimisihtmärke.

**Peamised seaded:*** **Minimaalne kalibreerimisproovi pindala (px)**: Sihtmärgi tuvastamise suuruse künnis (vaikimisi:**25**, vahemik 0–10000)
* **Minimaalne sihtmärkide klastrite arv (0–100)**: sarnasuse künnis sihtmärkide piirkondade rühmitamiseks (vaikimisi:**60**)**Millal seadistada:**

* Suurendage proovipinda, kui tekivad valed tuvastused
* Vähendage seda, kui sihtmärke ei tuvastata
* Reguleerige klastrite suurust, kui sihtmärgid jagunevad mitmeks tuvastuseks

{% hint style="info" %}
Need seaded on hallid, kui **peegelduskalibreerimine / valgebalanss** on välja lülitatud — selle väljalülitamisel ei toimu sihtmärkide tuvastamist üldse.
{% endhint %}

### Töötlemine

Peamised pilditöötlemise ja kalibreerimise valikud.

**Olulisemad seaded:*** **Vignette correction**: Kompenseerib objektiivi servade tumenemist ✅ Soovitatav
* **Reflectance calibration / white balance**: Kalibreerib pilte tuvastatud sihtmärkide (Survey3
) ja/või DAQ valgusanduri andmete (LATTICE) abil ✅ Soovitatav
* **Debayeri meetod**: algoritm RAW-failide teisendamiseks 3-kanaliliseks multispektraalseks pildiks
* **Minimaalne kalibreerimisintervall**: minimaalne aeg sekundites kalibreerimismärgi kasutamise vahel (vaikimisi:**0** = kasuta kõiki, vahemik 0–3600)**Kalibreerimata varutooted:**Kui kaadri peegeldusvõimet ei ole võimalik kalibreerida (sihtmärki pole saadaval või kalibreerimine on keelatud), eksporditakse see ühe kahest varutootest —**iga jooksu kohta on olemas täpselt üks neist kahest**, mille valib Vignette’i korrigeerimise lüliti:

* **Ekspordi anduri vastus**: salvestab `Sensor_Response_Images` — kasutatakse, kui vignettkorrektsioon on**väljalülitatud*** **Ekspordi vignettkorrigeeritud**: salvestab `Vignette_Corrected_Images` — kasutatakse, kui vignettkorrektsioon on**sisse lülitatud**Valikukast, mis ei ole kasutusel, on halliks muudetud. Aktiivse valiku tühistamine peatab selle faili salvestamise täielikult.**LATTICE eksporditavad failid** (näidatakse iga projekti puhul; need kehtivad LATTICE-pildistuste kohta):

* **Ekspordi debayered**: lineaarne debayered-pilt (`Debayered_Images`): kalibreerimispaketi tume nihe on eemaldatud, seejärel teostatud demosaic, ilma tasapinnata või segamiseta — sama fail, mida salvestus salvestab debayered-tasemel — anduri algses orientatsioonis. Kehtib RGB
- ja multispektraalsete moodulite puhul.
* **Ekspordi eelvaade**: ekraanil kuvatav eelvaade (`Preview_Images`). RGB
= valgebalanss (DAQ-valgusallikas, kui see on saadaval, muidu gray-world) + gamma; multispektraalne = valevärvide venitus.
* **Kiirguse eksport**: float32 spektraalne kiirgus (`Radiance_Images`, W/m²/sr/nm). Ainult multispektraalsed moodulid – ei kehti RGB
-põhifailide puhul.
* **Ekspordi peegeldusvõime**: uint16 peegeldusvõime (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), kui `.daq` allapoole suunatud mõõtmistulemus või kaadri sees olev sihtmärk katab kogu kaadri. Ainult multispektraalsed moodulid.

Kõik neli on **vaikimisi sisse lülitatud**— üks imporditud LATTICE toorkaader jaguneb ühe töötlemisvooru jooksul kõikidele aktiveeritud ja kohaldatavatele toodetele. Valikukast**Ekspordi peegeldusvõime** on halliks muudetud, kui peegeldusvõime kalibreerimine on välja lülitatud. Seaded, mille muutmist takistab ülemine lüliti, on alati halliks muudetud ja neile on lisatud abiteade, milles on nimetatud muudetav lüliti.**Täpsemad seaded:*** **Valgusanduri ajavööndi nihke**: Tunnid UTC-st valgusanduri aja ühtlustamiseks (vaikimisi: 0, vahemik −12 kuni +12)
* **Rakenda PPK-korrektsioone**: kasutab `.daq`-failidest pärit GPS-/eksponatsioonipinnide andmeid (vaikimisi: välja lülitatud)
* **Eksponatsioonipinnid 1/2**: määrab kaamerad eksponatsioonipinnidele kahekaameraliste seadistuste puhul

{% hint style="info" %}
**LATTICE&#x27;i sisendtase on automaatne.** LATTICE&#x27;i salvestused sisaldavad oma töötlemistaset XMP-metandmetes ning töötlemine algab alati toorkaadri tasandil – graafilises kasutajaliideses pole midagi seadistada. (CLI
-lipu `--input-level` on mõeldud kogenud kasutajale metandmete kaotuse korral; vt [CLI
-viidet](../reference/cli-reference.md).)
{% endhint %}

### Debayeri meetod

Praegu pakume Chloros
es kahte debayeri meetodit:

#### Standard (kiire, keskmine kvaliteet)

Standardne debayer töötleb kiiresti, kuid tekitab debayeri värvimüra, mille tulemuseks on vähem täpsed ja mürarikkamad pildid.

#### Tekstuuritundlik (aeglane, kõrgeim kvaliteet) \[Ainult Chloros
+]

Tekstuuritundlik meetod kasutab kõrgekvaliteedilist servatundlikku debayeringut koos AI/ML müra eemaldamise mudeliga, mis eemaldab peaaegu kogu debayeringust tuleneva müra. Mudeli töötamiseks on vaja GPU mälu (VRAM): **7 GB või rohkem VRAM-i** korral saab see töödelda mitut pilti korraga; alla 7 GB töötab see ühe pildiga korraga (märgatavalt aeglasemalt). Vaata [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-pildid kasutavad alati standarddemosaiki.** LATTICE-piltide jaoks ei ole olemas LATTICE-le treenitud tekstuuritundlikku mudelit, seega seda valikut LATTICE-piltide puhul ei pakuta — sama projekti Survey3
-pildid võivad seda siiski kasutada.
{% endhint %}

### Indeks (multispektraalsed indeksid)

Määrake, milliseid taimestikuindekseid arvutada ja eksportida. Graafilise kasutajaliidese rippmenüüs on **27 eelmääratud indeksivalemit**.**Kuidas indekseid lisada:**

1. Klõpsake nuppu**„Lisa indeks”**

2. Valige rippmenüüst indeks (NDVI
, NDRE
, GNDVI
jne)
3. Määrake visualiseerimise seaded (LUT-värvid, väärtuste vahemikud)
4. Lisage vajaduse korral mitu indeksit

**Populaarsed indeksid:*** **NDVI**: taimestiku üldine tervislik seisund (kõige levinum)
* **NDRE**: RedEdge
abil varajane stressi tuvastamine* **GNDVI**: tundlik klorofülli kontsentratsiooni suhtes
* **OSAVI**: sobib hästi nähtava pinnasega
* **EVI**: piirkonnad, kus lehepinna indeks (LAI
) on kõrge

**Kohandatud valemid:**

* Looge kohandatud multispektraalsed indeksivalemid, kasutades kõigi pildikanalite vahelist matemaatikat
* Salvesta kohandatud valemid hilisemaks kasutamiseks
* Kohandatud valemid on Chloros
+ funktsioon; kättesaadavus sõltub teie paketi tasemest

Kõigi kättesaadavate indeksite ja valemite kohta – sealhulgas selle kohta, millised nimed on saadaval ainult graafilises kasutajaliideses (GUI) ja millised töötavad ka CLI
-s/SDK
-s – vaadake [Multispektraalsete indeksite valemeid](../project-settings/multispectral-index-formulas.md).

### Eksport

Määrab väljundfaili formaadi.

**Saadaval olevad formaadid**(seade:**Kalibreeritud pildi formaat**, vaikimisi**TIFF
(16-bit)**):

* **TIFF
(16-bit)**: soovitatav GIS-i ja teadusliku analüüsi jaoks
* **TIFF
(8-bit)**: 8-bitine TIFF
* **TIFF
(32-bit, protsent)**: ujukomaväärtused
* **PNG
(8-bit)**: kaotuseta pakkimine visualiseerimiseks
* **JPG (8-bit)**: väikseimad failid, kaotusega pakkimine

Loend näitab, milliseid pilte projekt võib sisaldada: Chloros
ei suurenda bittisügavust. Kõik 8-bitised pildid pakuvad ainult TIFF
-i (8-bit), PNG
-i (8-bit) ja JPG-d (8-bit), ning vaikimisi on valitud TIFF
(8-bit). Segaprojektis eksporditakse iga pilt oma sügavusega või madalama sügavusega. Radiance jääb 32-bitise ujukomaarvuna.

Väljundfailid salvestatakse projekti kausta alla, rühmitatuna kaamera ja formaadi järgi: `<project>/<camera>/<format>/<Product>_Images/`. Radiance salvestatakse **alati** float32-na kausta `tiff32`, sõltumata sellest seadistusest. Eksporditud failid säilitavad allika failinime – toodet identifitseerib kaust. LATTICE multispektraalsed ekspordid võivad kirjutada ka pikselipõhiseid piiramismaske kausta `clamp_masks/`, mis asub tootekastude kõrval. Täieliku väljundpuustruktuuri kohta vaadake [Töötlemise lõpetamine](finishing-the-processing.md).

{% hint style="warning" %}
**Peegeldusväärtuste lugemine**: DN, mis tähendab ρ = 1,0, sõltub allikakaamerast – LATTICE kasutab väärtust 32768 (märgitud XMP-failis `Chloros:PixelScale`), Survey3
kasutab väärtust 65535. Lugege märget, selle asemel et eeldada konstanti. Vaadake [Väljundpildi formaadid](../output-image-formats.md).
{% endhint %}

### DAQ valgusandur

Selles jaotises on loetletud kõik teie projekti DAQ allapoole suunatud failid (`.daq` / `.csv`), üks rida faili kohta, näidates anduri mudelit, failinime ja selle faili jaoks kehtivat hajuti **kate** korrektsiooni, mis selle faili puhul kehtib.

* **Katte ülekirjutamine (kõik failid)**: üks projektipõhine rippmenüü.**Auto** (vaikimisi) kasutab iga faili salvestatud piirmäära — kui midagi pole salvestatud, eeldatakse päikesepaistet, kuid DAQ-U, DAQ-M ja DAQ-E tarnitakse ilma piirmäärata ning Chloros
ei näe, milline on sobitatud, seega annab töötlemine hoiatuse eeldatava piirmäära kohta: valige sobitatud piirmäär. Piirväärtuse valimine tühistab kõik failid, välja arvatud DAQ-E-S-i failid, mille hajuti on osa kalibreerimisest: tooresid salvestusi korrigeeritakse sellega ning salvestusi, millel on juba piirväärtus, viidatakse uuesti (salvestatud korrigeerimine tühistatakse, valitud piirväärtus rakendatakse).
* Ridades kuvatakse hoiatus, kui salvestatud kate oli jaoturi eeldatud vaikimisi väärtus, mitte operaatori kinnitatud, ning kui valitud kattel puudub profiil selle seadmemudeli jaoks (selle faili puhul lükatakse ümberkirjutamine tagasi).

Vahekaardil „Valgusandurid“ tehtud DAQ-salvestused lisatakse avatud projektile automaatselt ning imporditud `.daq` / `.csv`-failid ilmuvad siia kohe, kui need on lisatud.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Projekti alumised seaded — indeks, ekspordivorming, DAQ-valgusandurite osa ning projekti malli/kausta juhtelemendid</p></figcaption></figure>### Massiivi joondamine

See osa kuvatakse **ainult** siis, kui vähemalt üks projekti pilt sisaldab moodulitevahelist joondamistransformatsiooni, mille LATTICE-massiivid salvestamisel lisavad (`Chloros:Alignment*` XMP). Siin näidatakse, mitmel pildil on märgised ja milline kaamera on viitekaamera, ning on järgmised juhtelemendid:

* **Rakenda massiivi joondamist** (vaikimisi: sisse lülitatud): joondab iga töödeldud toote (debayering / eelvaade / kiirgus / peegeldus / indeks) massiivi ühise viitegeomeetria järgi. Väljas = eksportimine anduri alggeomeetrias.
* **Lõika ühisele kattuvale alale** (vaikimisi: sisse lülitatud): lõika joondatud ekspordid piirkonnale, mida kõik moodulid jagavad, nii et igal lainealal on sama pindala. Väljalülitamisel säilitatakse kogu anduri pind (allika välispoolne ala täidetakse mustaga).
* **Ümberdiskrimineerimine**:**Bilineaarne (sile, vaikimisi)**,**Lähim (säilitab täpsed väärtused)**— pikslitevahelist segamist ei toimu, rangete radiomeetriliste analüüside jaoks — või**Kubiline (kõige teravam)**.***

## Seadete salvestamine ja laadimine

### Projekti malli salvestamine

Looge korduvkasutatavaid malle järjepidevate töövoogude jaoks:

1. Määrake kõik soovitud seaded projekti seadete paneelis
2. Kerige alla **„Projekti malli salvestamine”** jaotiseni
3. Sisestage kirjeldav malli nimi (nt „Survey3N

_RGN_Agriculture”)
4. Klõpsake salvestusikoonile

**Eelised:**

* Rakendage identseid seadeid mitmes projektis
* Jagage konfiguratsioone meeskonnaliikmetega
* Säilitage järjepidevus korduvate uuringute puhul

### Malli laadimine uude projekti

Uue projekti loomisel:

1. Valige peamenüüst **„Uus projekt“**

2. Valige soovi korral malli valikust sobiv projektimall
3. Kõik malli seaded rakenduvad automaatselt

### Töökataloog

Seade **„Töökataloog”** määrab, kuhu uued projektid vaikimisi luuakse:

* **Vaikimisi asukoht**: `Documents\MAPIR\Chloros Projects` teie Windows

i kausta „Dokumendid” (tavaliselt `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; see järgib kausta „Dokumendid”, kui OneDrive või rändprofiil seda teisaldab)
* **Asukoha muutmine**: klõpsake redigeerimisikoonil ja valige uus kaust
* **Jagatud CLI

-ga**: `chloros-cli` kasutab sama vaikimisi projektikausta seadet
* **Windows

is hoidke nimi lühike**: kui Windows

i pikad teed pole sisse lülitatud, lükatakse enne töötlemist tagasi projektikausta nimi, mis on pikem kui umbes 130 tähemärki — vt [Töötlemise alustamine](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Millal muuta**:
  * Võrguketas meeskonnatööks
  * Erinev ketas, millel on rohkem salvestusruumi
  * Korraldatud kaustastruktuur aasta/kliendi kaupa

***

## PPK (Post-Processed Kinematic) seadistamine

Kui kasutate täpse geolokatsiooni jaoks GPS-iga varustatud MAPIR

i DAQ-salvestajaid:

### Eeltingimused

* GPS-mooduliga (GNSS) MAPIR

-andmekogumisseade
* .daq-logifail, milles on säriaja viikude andmed
* Kaamera, mis on salvestamise ajal ühendatud andmekogumisseadme säriaja viikudega

### Seadistamise sammud

1. Paigutage .daq-logifail oma projektikausta
2. Projektiseadetes märkige valik **„Kohalda PPK-korrektsioone”**

3. Määrake vajadusel**„Valgusanduri ajavööndi nihke”** (vaikimisi: 0 UTC-le)
4. Määrake kaamerad eksponeerimisviikudele:
   * **Üks kaamera**: määratakse automaatselt viigule 1
   * **Kaks kaamerat**: määrake iga kaamera käsitsi õigele viigule**Eksponeerimisviikude määramine:*** **Eksponeerimisviik 1**: vali kaameramudel rippmenüüst
* **Eksponeerimisviik 2**: vali teine kaamera või „Ära kasuta“
* Ühte ja sama kaamerat ei saa määrata mõlemale viigule

{% hint style="warning" %}
**Oluline**: Eksponeerimispinnid peavad olema õigesti määratud vastavatele kaameratele. Vale määramine põhjustab valesid geolokatsiooniandmeid.
{% endhint %}

***

## Täpsemad stsenaariumid

### Mitme kaameraga projektid

Kui töödeldakse ühes projektis mitme MAPIR

-kaamera pilte:

1. Chloros

tuvastab automaatselt iga kaameramudeli (nii Survey3

kui ka LATTICE)
2. Iga kaamera saab sobivad töötlusprofiilid ja iga kaamera saab oma väljundkaustade struktuuri
3. PPK: Määrake iga Survey3

-kaamera käsitsi õigele ekspositsioonipinnile
4. Kõik kaamerad kasutavad sama ekspordivormingut ja indekseid

**Näited**: Survey3W

RGN

+ Survey3N

OCN

kahekaameraline seadmestik või LATTICE-seade, milles on kombineeritud RGB

-põhikaamera ja kitsasriba moodulid

### Aeg-luup-võtted või mitme kuupäeva uuringud

Sama ala korduvate uuringute puhul aja jooksul:

1. Looge mall oma standardseadistustega
2. Kasutage igal seansil ühtset kalibreerimismärgi seadistust
3. Töötlege iga kuupäeva eraldi projektina
4. Kasutage võrreldavate tulemuste saamiseks identsed seaded
5. Eksportige samas formaadis ajalise analüüsi jaoks

### Suured andmekogumid

Projektide puhul, kus on palju pilte (üle 500):

* Kaaluge projektide jagamist väiksemateks projektideks kuupäeva või piirkonna järgi
* Kasutage Chloros

+ paralleelset töötlemist kiiremate tulemuste saamiseks
* Kaaluge CLI

-i või API

-i kasutamist partiide automatiseerimiseks
* Reguleerige minimaalse kalibreerimise intervalli, et vähendada sihtmärkide tuvastamise aega

***

## Seadete kontrollimine

Enne töötlemise alustamist kontrollige järgmisi olulisi seadeid:

* [ ] Kaameramudel on failibrauseris õigesti tuvastatud
* [ ] Vignettkorrektsioon on sisse lülitatud
* [ ] Peegelduskalibreerimine on sisse lülitatud
* [ ] Survey3

puhul: vähemalt üks kalibreerimissihtepilt on imporditud ja kontrollitud; „LATTICE” puhul: sihtmärk ja/või `.daq` allapoole suunatud salvestus on olemas
* [ ] Soovitud multispektraalsed indeksid on lisatud
* [ ] Ekspordivorming sobib teie töövooguga
* [ ] PPK-seaded on konfigureeritud (kui kasutate .daq-faili koos eksponeerimissündmustega)

***

## Järgmised sammud

Kui seaded on konfigureeritud:

1. **Märgistage kalibreerimise sihtpildid** – vt [Sihtpiltide valimine](choosing-target-images.md)
2. **Alustage töötlemist** – vt [Töötlemise alustamine](starting-the-processing.md)
3. **Jälgige töötlemise kulgu** – vt [Töötlemise jälgimine](monitoring-the-processing.md)

Kõigi saadaval olevate seadistuste kohta leiate täpsemat teavet viitedokumendist [Projekti seadistused](../project-settings/project-settings.md).
