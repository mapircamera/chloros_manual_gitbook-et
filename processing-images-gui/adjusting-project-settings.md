# Projekti seadete kohandamine

Enne piltide töötlemist on oluline konfigureerida projekti seaded vastavalt oma töövoo nõuetele. Projekti seaded <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> paneel võimaldab põhjalikult juhtida kalibreerimist, töötlemisvõimalusi, multispektraalseid indekseid ja ekspordivorminguid.

## Projektiseadete avamine

1. Avage oma projekt Chloros-is
2. Klõpsake vasakul külgribal **Projektiseaded** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ikoonile vasakul külgribal
3. Projekti seaded paneel kuvab kõik konfiguratsiooni valikud

{% hint style="info" %}
**Seaded salvestatakse automaatselt** koos teie projektiga. Kui avate projekti uuesti, taastatakse kõik seaded.
{% endhint %}

***

## Kiirhäälestus tavaliste töövoogude jaoks

### Vaikesätted (soovitatav enamikule kasutajatele)

Tüüpiliste MAPIR Survey3 kaameratöövoogude puhul sobivad vaikesätted hästi:

* ✅ **Vignette-korrektsioon**: lubatud
* ✅ **Peegelduskalibreerimine**: lubatud (nõuab MAPIR sihtmärkide pilte)
* ✅ **Debayer-meetod**: Standard (kiire, keskmine kvaliteet)
* ✅ **Ekspordivorming**: TIFF (16-bitine)

Lihtsalt importige oma pildid ja alustage töötlemist nende vaikimisi seadetega.

***

## Projekti seaded – ülevaade

Projekti seaded on jagatud mitmesse kategooriasse. Allpool on iga sektsiooni kokkuvõte. Täieliku dokumentatsiooni leiate [Projekti seaded](../project-settings/project-settings.md).

### Sihtmärkide tuvastamine

Määrab, kuidas Chloros tuvastab kalibreerimissihtmärke teie piltidel.

**Peamised seaded:*** **Minimaalne kalibreerimisproovi pindala**: sihtmärkide tuvastamise suuruse künnis (vaikimisi: 25 pikslit)
* **Minimaalne sihtmärkide klastrite arv**: sihtmärkide piirkondade rühmitamise sarnasuse künnis (vaikimisi: 60)**Millal kohandada:**

* Suurendage proovi pindala, kui tekivad valed tuvastused
* Vähendage seda, kui sihtmärke ei tuvastata
* Reguleerige klastrit, kui sihtmärgid jagunevad mitmeks tuvastuseks

### Töötlemine

Peamised pilditöötlus- ja kalibreerimisvalikud.

**Peamised seaded:*** **Vignette-korrektsioon**: Kompenseerib objektiivi tumenemist servades ✅ Soovitatav
* **Peegelduskalibreerimine**: Normaliseerib väärtused kalibreerimissihte kasutades ✅ Soovitatav
* **Debayer-meetod**: Algoritm RAW-failide teisendamiseks 3-kanaliliseks multispektraalseks
* **Minimaalne kalibreerimisintervall**: Aeg kalibreerimissihikute kasutamise vahel (0 = kasuta kõiki)**Täpsemad seaded:*** **Valgusanduri ajavööndi nihke**: PPK-aja sünkroniseerimiseks (vaikimisi: 0)
* **Rakenda PPK-korrektsioone**: Kasutab .daq-failidest pärit GPS/eksponatsioonipin-andmeid
* **Eksponatsioonipin 1/2**: Määrab kaamerad eksponatsioonipinidele kahekaameraseadmete puhul

### Debayeri meetod

Praegu pakume Chloros-is 2 debayeri meetodit:

#### Standard (kiire, keskmine kvaliteet)

Standardne debayering töötleb kiiresti, kuid tekitab debayeringu värvimüra, mille tulemuseks on vähem täpsed ja mürarikkamad pildid.

#### Tekstuuritundlik (aeglane, kõrgeim kvaliteet) \[Ainult Chloros+]

Tekstuuritundlik kasutab kõrgekvaliteedilist servatundlikku debayeringut koos AI/ML müra eemaldamise mudeliga, mis eemaldab peaaegu kogu debayeringu müra. Tekstuuritundliku mudeli töötamiseks on vaja GPU mälu (VRAM). Soovitame seda kasutada, kui teil on kiirema töötlemise jaoks saadaval &gt;4 GB VRAM-i.

### Indeks (multispektraalsed indeksid)

Määrake, milliseid taimestikuindekseid arvutada ja eksportida.

**Kuidas indekseid lisada:**

1. Klõpsake nuppu**&quot;Lisa indeks&quot;**

2. Valige indeks rippmenüüst (NDVI, NDRE, GNDVI jne)
3. Määrake visualiseerimise seaded (LUT-värvid, väärtuste vahemikud)
4. Lisage vajaduse korral mitu indeksit

**Populaarsed indeksid:*** **NDVI**: üldine taimestiku tervis (kõige levinum)
* **NDRE**: varajane stressi tuvastamine koos RedEdge-iga
* **GNDVI**: tundlik klorofülli kontsentratsioonile
* **OSAVI**: Sobib hästi nähtava pinnasega
* **EVI**: Suure lehepinna indeksiga (LAI) piirkonnad**Kohandatud valemid (ainult Chloros+):**

* Loo kohandatud multispektraalsed indeksivalemid
* Kasuta ribade matemaatikat kõigi pildikanalitega
* Salvesta kohandatud valemid taaskasutamiseks

Kõigi saadaval olevate indeksite ja valemite kohta vaata [Multispektraalsed indeksivalemid](../project-settings/multispectral-index-formulas.md).

### Eksport

Reguleerib väljundfaili formaati ja kvaliteeti.

**Saadaval olevad formaadid:*** **TIFF (16-bitine)**: soovitatav GIS-i ja teadusliku analüüsi jaoks (vahemik 0–65 535)
* **TIFF (32-bitine, protsent)**: ujukomaga peegeldusväärtused (vahemik 0,0–1,0)
* **PNG (8-bit)**: Kaotuseta pakkimine visualiseerimiseks (vahemik 0–255)
* **JPG (8-bit)**: Väikseimad failid, kaotusega pakkimine (vahemik 0–255)***

## Seadete salvestamine ja laadimine

### Projekti malli salvestamine

Looge korduvkasutatavad mallid järjepidevate töövoogude jaoks:

1. Määrake kõik soovitud seaded projekti seadepaneelis
2. Kerige alla **„Salvesta projektimall”** sektsioonini
3. Sisestage kirjeldav malli nimi (nt „Survey3N\_RGN\_Agriculture”)
4. Klõpsake salvestamise ikooni

**Eelised:**

* Rakendage identseid seadeid mitmes projektis
* Jagage konfiguratsioone meeskonnaliikmetega
* Säilitage järjepidevus korduvate uuringute puhul

### Laadige mall uude projekti

Uue projekti loomisel:

1. Valige peamenüüst **„Uus projekt”**

2. Valige valik**„Laadi mallist”**

3. Valige oma salvestatud mall
4. Kõik seaded rakenduvad automaatselt

### Töökataloog

Seade **&quot;Projekti salvestamise kaust&quot;** määrab, kuhu uued projektid vaikimisi luuakse:

* **Vaikimisi asukoht**: `C:\Users\[Username]\Chloros Projects`
* **Asukoha muutmine**: klõpsake redigeerimisikoonil ja valige uus kaust
* **Millal muuta**:
  * Võrguketas meeskonnatööks
  * Teine ketas, millel on rohkem salvestusruumi
  * Korraldatud kaustastruktuur aasta/kliendi järgi

***

## PPK (Post-Processed Kinematic) seadistamine

Kui kasutate täpse geolokatsiooni jaoks MAPIR DAQ-salvestajaid koos GPS-iga:

### Eeltingimused

* MAPIR DAQ koos GPS (GNSS) mooduliga
* .daq logifail koos eksponeerimispinide sisestustega
* Kaamera, mis on salvestamise ajal ühendatud DAQ eksponeerimispinidega

### Konfigureerimise sammud

1. Paigutage .daq logifail oma projektikausta
2. Projektiseadetes aktiveerige **&quot;Kohalda PPK-korrektsioone&quot;** valik
3. Määrake vajadusel **&quot;Valgusanduri ajavööndi nihke&quot;** (vaikimisi: 0 UTC-le)
4. Määrake kaamerad ekspositsioonipinidele:
   * **Üks kaamera**: Määratakse automaatselt pinile 1
   * **Kaks kaamerat**: Määrake iga kaamera käsitsi õigele pinile**Ekspositsioonipinide määramine:*** **Valgustuse kontakt 1**: valige kaameramudel rippmenüüst
* **Valgustuse kontakt 2**: valige teine kaamera või „Ära kasuta”
* Sama kaamerat ei saa määrata mõlemale kontaktile

{% hint style="warning" %}
**Oluline**: Valgustuse kontaktid peavad olema õigesti määratud vastavatele kaameratele. Vale määramine põhjustab valesid geolokatsiooniandmeid.
{% endhint %}

***

## Täpsemad stsenaariumid

### Mitme kaameraga projektid

Kui töödeldakse ühes projektis mitme MAPIR kaamera pilte:

1. Chloros tuvastab automaatselt iga kaameramudeli
2. Iga kaamera saab sobiva töötlemisprofiili
3. PPK: Määrake iga kaamera käsitsi õigele eksponeerimispinnile
4. Kõik kaamerad kasutavad sama ekspordivormingut ja indekseid

**Näide**: Survey3W RGN + Survey3N OCN kahekaameraline seadmestik

### Aeg-luup või mitme kuupäeva uuringud

Sama ala korduvate uuringute jaoks aja jooksul:

1. Loo mall oma standardseadistustega
2. Kasuta igas seansis ühtset kalibreerimise sihtmärgi seadistust
3. Töötle iga kuupäeva eraldi projektina
4. Kasuta võrreldavate tulemuste saamiseks identseid seadeid
5. Ekspordi ajaliseks analüüsiks samas formaadis

### Suured andmekogumid

Projektide puhul, kus on palju pilte (500+):

* Kaaluge projektide jagamist väiksemateks projektideks kuupäeva või ala järgi
* Kasutage Chloros+ paralleelset töötlemist kiiremate tulemuste saamiseks
* Kaaluge CLI või API kasutamist partiide automatiseerimiseks
* Reguleerige minimaalse kalibreerimise intervalli, et vähendada sihtmärgi tuvastamise aega

***

## Seadete kontrollimine

Enne töötlemise alustamist kontrollige järgmisi olulisi seadeid:

* [ ] Kaameramudel on failibrauseris õigesti tuvastatud
* [ ] Vignettkorrektsioon on sisse lülitatud
* [ ] Peegelduskalibreerimine on sisse lülitatud
* [ ] Vähemalt üks kalibreerimise sihtmärgi pilt on imporditud
* [ ] Soovitud multispektraalsed indeksid on lisatud
* [ ] Ekspordivorming sobib teie töövooguga
* [ ] PPK-seaded on konfigureeritud (kui kasutate .daq-faili koos eksponeerimissündmustega)

***

## Järgmised sammud

Kui seaded on konfigureeritud:

1. **Märkige kalibreerimise sihtpildid** – vt [Sihtpiltide valimine](choosing-target-images.md)
2. **Alustage töötlemist** – vt [Töötlemise alustamine](starting-the-processing.md)
3. **Jälgige töötlemise kulgu** – vt [Töötlemise jälgimine](monitoring-the-processing.md)

Kõigi saadaval olevate seadete kohta leiate täpsemat teavet viitedokumendist [Projekti seaded](../project-settings/project-settings.md).
