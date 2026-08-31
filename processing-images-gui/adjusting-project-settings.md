# Projekti seadete kohandamine

Enne piltide töötlemist on oluline konfigureerida projekti seaded vastavalt oma töövoo nõuetele. Projekti seaded <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> paneel võimaldab põhjalikku kontrolli kalibreerimise, töötlemisvalikute, multispektraalsete indeksite ja ekspordiformaatide üle.

## Projektiseadete avamine

1. Avage oma projekt Chloros-is
2. Klõpsake vasakul külgribal ikooni **Projektiseaded** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">
3. Projektiseadete paneelis kuvatakse kõik seadistusvõimalused

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Projekti seaded – kuvamine, sihtmärgi tuvastamine ja töötlemine</p></figcaption></figure>{% hint style="info" %}
**Seaded salvestatakse automaatselt** koos teie projektiga. Kui avate projekti uuesti, taastatakse kõik seaded.
{% endhint %}

***

## Kiirhäälestus tavaliste töövoogude jaoks

### Vaikesätted (soovitatav enamikule kasutajatele)

Vaikesätted sobivad hästi tüüpiliste Survey3 ja LATTICE töövoogude jaoks:

* ✅ **Vignette-korrektsioon**: sisse lülitatud
* ✅ **Peegelduskalibreerimine / valgebalanss**: sisse lülitatud (kasutab MAPIR sihtmärke ja/või DAQ valgusanduri andmeid)
* ✅ **Debayeri meetod**: Standardne (kiire, keskmine kvaliteet)
* ✅ **Ekspordivorming**: TIFF (16-bitine)
* ✅ **Kõik eksporditavad tulemused**: Lülitatud sisse (LATTICE salvestab automaatselt tulemused debayeri, eelvaate, kiirguse ja peegeldusvõime vormingutes)

Lihtsalt importige oma pildid ja alustage töötlemist nende vaikimisi seadetega.

***

## Projekti seaded – ülevaade

Projekti seaded on jaotatud allpool toodud osadeks. Kaks lisajaotust — **DAQ valgusandur**ja**maatriksi joondamine** — ilmuvad automaatselt, kui teie projekt sisaldab vastavaid faile. Täieliku dokumentatsiooni leiate [Projekti seaded](../project-settings/project-settings.md).

### Kuvatäitlus

* **Pildi pisipildi eraldusvõime**: Pildivõrgu pisipiltide eraldusvõime. Valikud:**Vaikimisi (512 px)**,**1024 px**,**2048 px**,**Täis eraldusvõime**. Ainult kuvamiseks — ei mõjuta kunagi töötlemist. Suuremad väärtused näevad suurendatuna teravamad välja, kuid laadivad aeglasemalt.

### Sihtmärgi tuvastamine

Määrab, kuidas Chloros tuvastab teie piltidel kalibreerimissihte.

**Peamised seaded:*** **Minimaalne kalibreerimisproovi pindala (px)**: Sihtmärgi tuvastamise suuruse künnis (vaikimisi:**25**, vahemik 0–10000)
* **Minimaalne sihtmärkide klastrite arv (0–100)**: Sihtmärkide piirkondade rühmitamise sarnasuse künnis (vaikimisi:**60**)**Millal reguleerida:**

* Suurendage proovipinda, kui tekivad valed tuvastused
* Vähendage seda, kui sihtmärke ei tuvastata
* Reguleerige rühmitamist, kui sihtmärgid jagunevad mitmeks tuvastuseks

{% hint style="info" %}
Need seaded on hallid, kui **peegelduskalibreerimine / valgebalanss** on välja lülitatud – selle väljalülitamisel ei tööta sihtmärgi tuvastamine üldse.
{% endhint %}

### Töötlemine

Peamised pilditöötlus- ja kalibreerimisvalikud.

**Olulised seaded:*** **Vignette-korrektsioon**: kompenseerib objektiivi servade tumenemist ✅ Soovitatav
* **Peegelduskalibreerimine / valgebalanss**: Kalibreerib pilte tuvastatud sihtmärkide (Survey3) ja/või DAQ-valgusanduri andmete (LATTICE) abil ✅ Soovitatav
* **Debayeri meetod**: algoritm RAW-failide teisendamiseks 3-kanaliliseks multispektraalseks kujutiseks
* **Minimaalne kalibreerimisintervall**: minimaalne aeg sekundites kalibreerimismärgi kasutamise vahel (vaikimisi:**0** = kasuta kõiki, vahemik 0–3600)**Kalibreerimata varutooted:**Kui kaadri peegeldusvõimet ei ole võimalik kalibreerida (sihtmärki pole saadaval või kalibreerimine on keelatud), eksporditakse see ühe kahest varutootest —**iga käivituse kohta on olemas täpselt üks neist kahest**, mille valib Vignette’i korrektsiooni lüliti:

* **Ekspordi anduri vastus**: salvestab `Sensor_Response_Images` — kasutatakse, kui vinjeti korrigeerimine on**väljalülitatud*** **Ekspordi vinjettkorrigeeritud**: salvestab `Vignette_Corrected_Images` — kasutatakse, kui vinjettkorrektsioon on**sisse lülitatud**Valikukast, mis ei ole kasutusel, on halliks muudetud. Aktiivse valiku tühistamine peatab selle faili salvestamise täielikult.**LATTICE eksporditooted** (näidatakse iga projekti puhul; need kehtivad LATTICE-pildistuste kohta):

* **Eksporti debayered**: lineaarne debayered-pilt (`Debayered_Images`). Kehtib RGB ja multispektraalsete moodulite puhul.
* **Ekspordi eelvaade**: ekraanil kuvatav eelvaade (`Preview_Images`). RGB = valgebalanss (DAQ-valgusallikas, kui see on saadaval, muidu hall maailm) + gamma; multispektraalne = valevärvide venitus.
* **Kiirguse eksport**: float32 spektraalne kiirgus (`Radiance_Images`, W/m²/sr/nm). Ainult multispektraalsed moodulid — ei kehti RGB-i masterite puhul.
* ****Ekspordi peegeldusvõime**: uint16 peegeldusvõime (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), kui `.daq` allapoole suunatud mõõtmistulemus või raami sees olev sihtmärk katab kogu raami. Ainult multispektraalsed moodulid.

Kõik neli on **vaikimisi sisse lülitatud**– üks imporditud LATTICE toorkaader jaguneb ühe töötlemisvooruga kõikidele aktiveeritud ja asjakohastele toodetele. Valikukast**Ekspordi peegeldusvõime** on halliks muudetud, kui peegeldusvõime kalibreerimine on välja lülitatud. Seaded, mille muutmist takistab ülemine lüliti, on alati hallid ja neil on abivihje, milles on nimetatud muudetav lüliti.**Täpsemad seaded:*** **Valgusanduri ajavööndi nihe**: Tunnid UTC-st valgusanduri aja ühtlustamiseks (vaikimisi: 0, vahemik −12 kuni +12)
* **Rakenda PPK-korrektsioone**: kasutab `.daq`-failidest pärit GPS-i/eksponatsioonipinide andmeid (vaikimisi: välja lülitatud)
* **Eksponatsioonipin 1/2**: määrab kaamerad eksponatsioonipinidele kahekaameraseadistuste puhul

{% hint style="info" %}
**LATTICE&#x27;i sisendtase on automaatne.** LATTICE&#x27;i salvestused kannavad oma töötlemistaset XMP-metandmetes ning töötlemine algab alati toorkaadri tasandil – graafilises kasutajaliideses pole midagi seadistada. (Lipuk CLI `--input-level` on mõeldud kogenud kasutajale metandmete kadumise korral piltide ülekirjutamiseks; vt [CLI viidet](../reference/cli-reference.md).)
{% endhint %}

### Debayeringu meetod

Praegu pakume Chloros-is kahte debayeringu meetodit:

#### Standard (kiire, keskmine kvaliteet)

Standardne debayering töötleb kiiresti, kuid tekitab debayeringu värvimüra, mille tulemuseks on vähem täpsed ja müraisemad pildid.

#### Tekstuuritundlik (aeglane, kõrgeim kvaliteet) \[Ainult Chloros+]

Tekstuuritundlik meetod kasutab kõrgekvaliteedilist servatundlikku debayeringut koos AI/ML müra vähendamise mudeliga, mis eemaldab peaaegu kogu debayeringust tuleneva müra. Mudeli töötamiseks on vaja GPU mälu (VRAM): **7 GB või rohkem VRAM-i** korral saab see töödelda mitut pilti korraga; alla 7 GB töötab see ühe pildi kaupa (märgatavalt aeglasemalt). Vaata [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-pildid kasutavad alati standarddemosaiki.** LATTICE-piltide jaoks ei ole olemas LATTICE-le õpetatud tekstuuritundlikku mudelit, seega seda valikut LATTICE-piltide puhul ei pakuta — sama projekti Survey3-pildid võivad seda siiski kasutada.
{% endhint %}

### Indeksid (multispektraalsed indeksid)

Määrake, milliseid taimestikuindekseid arvutada ja eksportida. Graafilise kasutajaliidese rippmenüüs on **27 eelmääratud indeksivalemit**.**Kuidas indekseid lisada:**

1. Klõpsake nuppu**„Lisa indeks””**

2. Valige indeks rippmenüüst (NDVI, NDRE, GNDVI jne)
3. Määrake visualiseerimise seaded (LUT-värvid, väärtuste vahemikud)
4. Lisage vajaduse korral mitu indeksit

**Populaarsed indeksid:*** **NDVI**: taimestiku üldine tervislik seisund (kõige levinum)
* **NDRE**: stressi varajane avastamine koos indeksiga RedEdge
* **GNDVI**: tundlik klorofülli kontsentratsioonile
* **OSAVI**: sobib hästi nähtava mullaga
* **EVI**: piirkonnad suure lehepinna indeksiga (LAI)**Kohandatud valemid:**

* Looge kohandatud multispektraalsed indeksivalemid, kasutades kõigi pildikanalite vahelist matemaatilist töötlust
* Salvesta kohandatud valemid hilisemaks kasutamiseks
* Kohandatud valemid on Chloros+ funktsioon; kättesaadavus sõltub teie paketi tasemest

Kõigi kättesaadavate indeksite ja valemite kohta – sealhulgas selle kohta, millised nimed on saadaval ainult graafilises kasutajaliideses ja millised töötavad ka CLI/SDK-is – vaadake [Multispektraalsed indeksivalemid](../project-settings/multispectral-index-formulas.md).

### Eksport

Määrab väljundfaili formaadi.

**Saadaval olevad formaadid**(seade:**Kalibreeritud pildi formaat**, vaikimisi**TIFF (16-bit)**):

* **TIFF (16-bit)**: soovitatav GIS-i ja teadusliku analüüsi jaoks
* **TIFF (32-bit, protsent)**: ujukomaväärtused
* **PNG (8-bit)**: kaotuseta pakkimine visualiseerimiseks
* **JPG (8-bit)**: väikseimad failid, kaotusega pakkimine

Väljundfailid salvestatakse projekti kausta alla, rühmitatuna kaamera ja formaadi järgi: `<project>/<camera>/<format>/<Product>_Images/`. Radiance salvestatakse **alati** float32-tüüpi andmetena kausta `tiff32`, sõltumata sellest seadistusest. Eksporditud failid säilitavad allika failinime — toodet identifitseerib kaust. Täieliku väljundpuustruktuuri kohta vaata [Töötlemise lõpetamine](finishing-the-processing.md).

{% hint style="warning" %}
**Peegeldusväärtuste lugemine**: DN, mis tähendab ρ = 1,0, sõltub allikakaamerast – LATTICE kasutab 32768 (märgitud XMP-failis `Chloros:PixelScale`), Survey3 kasutab 65535. Lugege sildilt väärtus ära, selle asemel et eeldada konstanti. Vaadake [Väljundpildi formaadid](../output-image-formats.md).
{% endhint %}

### DAQ valgusandur

Selles jaotises on loetletud kõik projekti DAQ allapoole suunatud andmed (`.daq` / `.csv`), üks rida faili kohta, näidates anduri mudelit, failinime ja selle faili jaoks kehtivat hajuti **katte** korrigeerimist.

* **Katte ülekirjutamine (kõik failid)**: üks projektipõhine rippmenüü.**Auto** (vaikimisi) kasutab iga faili salvestatud katet — kui midagi pole salvestatud, eeldatakse päikesepaistet, kuna kõik MAPIR DAQ-id on varustatud päikesepaistekorrektoriga. Piirväärtuse valimine asendab kõikide failide piirväärtused: töötlemata salvestusi korrigeeritakse sellega ning salvestusi, millel on juba piirväärtus, viidatakse uuesti (salvestatud korrigeerimine tühistatakse, valitud piirväärtus rakendatakse).
* Ridades kuvatakse hoiatus, kui salvestatud piirväärtus oli jaoturi eeldatud vaikimisi väärtus, mitte operaatori kinnitatud, ning kui valitud piirväärtusel puudub profiil selle seadmemudeli jaoks (ülekirjutamine lükatakse selle faili puhul tagasi).

Vahekaardil „Valgusandurid” tehtud DAQ-salvestused lisatakse avatud projektile automaatselt ning imporditud `.daq` / `.csv`-failid ilmuvad siia kohe, kui need on lisatud.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Projekti alumised seaded — indeks, ekspordivorming, DAQ-valgusandurite osa ning projekti malli/kausta juhtelemendid</p></figcaption></figure>### Massiivi joondamine

See osa kuvatakse **ainult** siis, kui vähemalt üks projekti pilt sisaldab moodulitevahelist joondamistransformatsiooni, mille LATTICE-massiivid salvestamisel lisavad (`Chloros:Alignment*` XMP). Siin näidatakse, mitmel pildil on märgised ja milline kaamera on viitekaamera, ning pakutakse järgmisi juhtnuppe:

* **Rakenda massiivi joondamist** (vaikimisi: sisse lülitatud): joondab iga töödeldud toote (debayering / eelvaade / kiirgus / peegeldus / indeks) massiivi ühise viitegeomeetria järgi. Väljalülitatuna eksporditakse andmed anduri alggeomeetrias.
* **Lõika ühisele kattumisalale** (vaikimisi: sisse lülitatud): lõika joondatud ekspordid piirkonnale, mida kõik moodulid jagavad, nii et igal lainealal on sama pindala. Väljalülitamisel säilitatakse kogu anduri pind (allika välispoolne osa täidetakse mustaga).
* **Ümberdiskrimineerimine**:**Bilineaarne (sile, vaikimisi)**,**Lähim (säilitab täpsed väärtused)**— pikslitevahelist segamist ei toimu, ranget radiomeetrilist analüüsi silmas pidades — või**Kubiline (kõige teravam)**.***

## Seadete salvestamine ja laadimine

### Projekti malli salvestamine

Loo korduvkasutatavaid malle ühtsete töövoogude jaoks:

1. Määra kõik soovitud seaded paneelis „Projekti seaded“
2. Kerige alla sektsioonini **„Salvesta projekti malli“**

3. Sisestage kirjeldav malli nimi (nt „Survey3N\_RGN\_Agriculture“)
4. Klõpsake salvestamise ikooni

**Eelised:**

* Rakendage identsed seaded mitmes projektis
* Jagage seadistusi meeskonnaliikmetega
* Tagage korduvate uuringute ühtsus

### Malli laadimine uude projekti

Uue projekti loomisel:

1. Valige peamenüüst **„Uus projekt”**

2. Valige soovi korral malli valikust sobiv projektimall
3. Kõik malli seaded rakenduvad automaatselt

### Töökataloog

Seade **&quot;Töökataloog&quot;** määrab, kuhu uued projektid vaikimisi luuakse:

* **Vaikimisi asukoht**: `C:\Users\[Username]\Chloros Projects`
* **Asukoha muutmine**: klõpsa redigeerimisikoonil ja vali uus kaust
* **Jagatud kaustaga CLI**: `chloros-cli` kasutab sama vaikimisi projektikausta seadet
* **Millal muuta**:
  * Võrguketas meeskonnatöö jaoks
  * Erinev ketas, millel on rohkem salvestusruumi
  * Korraldatud kaustastruktuur aasta/kliendi kaupa

***

## PPK (Post-Processed Kinematic) seadistamine

Kui kasutate täpse geopositsioneerimise jaoks MAPIR DAQ-salvestajaid koos GPS-iga:

### Eeltingimused

* MAPIR DAQ koos GPS (GNSS) mooduliga
* .daq-logifail, milles on sissekanded eksponeerimispinide kohta
* Kaamera, mis on salvestamise ajal ühendatud DAQ-seadme eksponeerimispinidega

### Seadistamise sammud

1. Paigutage .daq-logifail oma projektikausta
2. Projektiseadete all märkige ruut **„Kohalda PPK-korrektsioone”**

3. Määrake vajadusel**„Valgusanduri ajavööndi nihke”** (vaikimisi: 0 UTC-le)
4. Määra kaamerad ekspositsioonipinidele:
   * **Üks kaamera**: määratakse automaatselt pinile 1
   * **Kaks kaamerat**: määra iga kaamera käsitsi õigele pinile**Ekspositsioonipinide määramine:*** **Valgustuse kontakt 1**: vali kaameramudel rippmenüüst
* **Valgustuse kontakt 2**: vali teine kaamera või „Ära kasuta“
* Ühte ja sama kaamerat ei saa määrata mõlemale kontaktile

{% hint style="warning" %}
**Oluline**: Ekspositsioonipinnid tuleb õigesti määrata vastavatele kaameratele. Vale määramine põhjustab valesid geolokatsiooniandmeid.
{% endhint %}

***

## Täpsemad stsenaariumid

### Mitme kaameraga projektid

Kui töödeldakse ühes projektis mitme MAPIR kaamera pilte:

1. Chloros tuvastab automaatselt iga kaameramudeli (nii Survey3 kui ka LATTICE)
2. Iga kaamera saab sobivad töötlemisprofiilid ja iga kaamera saab oma väljundkaustade struktuuri
3. PPK: Määrake igale Survey3-kaamerale käsitsi õige ekspositsioonipin
4. Kõik kaamerad kasutavad sama ekspordivormingut ja indekseid

**Näited**: Survey3W, RGN + Survey3N, OCN kahekaameraline seadmestik, või LATTICE-maatriks, milles on segatud RGB-peakaamera ja kitsasriba moodulid

### Aeg-lõigu või mitme kuupäeva uuringud

Sama ala korduvateks uuringuteks aja jooksul:

1. Looge mall oma standardseadistustega
2. Kasutage igal seansil ühesugust kalibreerimismärgi seadistust
3. Töötlege iga kuupäeva eraldi projektina
4. Kasutage võrreldavate tulemuste saamiseks identsed seadistused
5. Eksportige samas formaadis ajalise analüüsi jaoks

### Suured andmekogumid

Projektide puhul, kus on palju pilte (500+):

* Kaaluge projektide jagamist väiksemateks projektideks kuupäeva või piirkonna järgi
* Kasutage kiiremate tulemuste saamiseks Chloros+ paralleelset töötlemist
* Kaaluge CLI või API kasutamist partiide automatiseerimiseks
* Reguleerige minimaalse kalibreerimise intervalli, et vähendada sihtmärgi tuvastamise aega

***

## Seadete kontrollimine

Enne töötlemise alustamist kontrollige järgmisi olulisi seadeid:

* [ ] Kaameramudel on failibrauseris õigesti tuvastatud
* [ ] Vignettide korrigeerimine on sisse lülitatud
* [ ] Peegelduskalibreerimine on sisse lülitatud
* [ ] Mudeli Survey3 puhul: vähemalt üks kalibreerimismärgi pilt on imporditud ja kontrollitud; mudeli LATTICE puhul: olemas on märk ja/või `.daq` allapoole suunatud salvestus
* [ ] Soovitud multispektraalsed indeksid on lisatud
* [ ] Teie töövoogule sobiv ekspordivorming
* [ ] PPK-seaded on konfigureeritud (kui kasutate .daq-faili koos eksponeerimissündmustega)

***

## Järgmised sammud

Kui seaded on konfigureeritud:

1. **Märkige kalibreerimise sihtpildid** – vt [Sihtpiltide valimine](choosing-target-images.md)
2. **Alustage töötlemist** – vt [Töötlemise alustamine](starting-the-processing.md)
3. **Jälgige töötlemise kulgu** – vt [Töötlemise jälgimine](monitoring-the-processing.md)

Kõigi saadaval olevate seadistuste kohta leiate täielikku teavet viitedokumendist [Projekti seadistused](../project-settings/project-settings.md).
