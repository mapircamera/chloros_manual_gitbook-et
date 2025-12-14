# Projekti seadeid kohandamine

Enne piltide töötlemist on oluline konfigureerida projekti seaded vastavalt oma töövoo nõuetele. Projekti seaded <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> paneel võimaldab kalibreerimise, töötlemisvalikute, multispektraalsete indeksite ja ekspordivormingute põhjalikku juhtimist.

## Projektiseadete avamine

1. Avage oma projekt Chloros-is
2. Klõpsake **Projektiseaded** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ikoonile vasakul külgribal
3. Projekti seaded paneel kuvab kõik konfiguratsiooni valikud

{% vihje style=&quot;info&quot; %}
**Seaded salvestatakse automaatselt** koos projektiga. Kui projekt uuesti avada, taastatakse kõik seaded.
{% endhint %}

***

## Kiirhäälestus tavaliste töövoogude jaoks

### Vaikesätted (soovitatav enamikule kasutajatele)

Tüüpiliste MAPIR Survey3 kaameratöövoogude puhul sobivad hästi vaikimisi seaded:

* ✅ **Vignette correction**: Enabled
* ✅ **Reflectance calibration**: Enabled (nõuab MAPIR sihtmärkide pilte)
* ✅ **Debayer-meetod**: kõrge kvaliteet (kiirem)
* ✅ **Ekspordivorming**: TIFF (16-bitine)

Importige lihtsalt oma pildid ja alustage töötlemist nende vaikimisi seadetega.

***

## Projekti seaded ülevaade

Projekti seaded paneel on jagatud mitmeks kategooriaks. Allpool on iga sektsiooni kokkuvõte. Täieliku dokumentatsiooni leiate [Projekti seaded](../project-settings/project-settings.md).

### Sihtmärgi tuvastamine

Kontrollib, kuidas Chloros tuvastab kalibreerimise sihtmärke teie piltidel.

**Peamised seaded:**

* **Minimaalne kalibreerimisproovi ala**: sihtmärgi tuvastamise suuruslävi (vaikimisi: 25 pikslit)
* **Minimaalne sihtmärgi klastrite arv**: sihtmärgi piirkondade rühmitamise sarnasuse lävi (vaikimisi: 60)

**Millal kohandada:**

* Suurendage proovi ala, kui tekivad valed tuvastused.
* Vähendage, kui sihtmärke ei tuvastata.
* Kohandage klastrite arvu, kui sihtmärgid jagunevad mitmeks tuvastuseks.

### Töötlemine

Peamised pilditöötlus- ja kalibreerimisvalikud.

**Peamised seaded:**

* **Vignette&#x27;i korrigeerimine**: kompenseerib objektiivi tumenemise servades ✅ Soovitatav
* **Peegelduskalibreerimine**: normaliseerib väärtused kalibreerimise sihtmärkide abil ✅ Soovitatav
* **Debayer-meetod**: algoritm RAW-vormingu teisendamiseks 3-kanaliliseks multispektraalseks
* **Minimaalne kalibreerimise intervall**: aeg kalibreerimise sihtmärkide kasutamise vahel (0 = kasuta kõiki)

**Täpsemad seaded:**

* **Valgusanduri ajavööndi nihke**: PPK aja sünkroniseerimiseks (vaikimisi: 0)
* **Rakenda PPK parandusi**: kasutab .daq failide GPS/eksponeerimise pin andmeid
* **Exposure Pin 1/2**: Määrab kaamerad ekspositsioonipinnidele kahe kaameraga seadistuste jaoks

### Indeks (multispektraalsed indeksid)

Konfigureerige, milliseid taimestiku indekseid arvutada ja eksportida.

**Indeksite lisamine:**

1. Klõpsake nuppu **„Lisa indeks”**
2. Valige indeks rippmenüüst (NDVI, NDRE, GNDVI jne)
3. Konfigureerige visualiseerimise seaded (LUT värvid, väärtuste vahemikud)
4. Lisage vajadusel mitu indeksit

**Populaarsed indeksid:**

* **NDVI**: üldine taimestiku tervis (kõige levinum)
* **NDRE**: varajane stressi tuvastamine koos RedEdge-ga
* **GNDVI**: klorofülli kontsentratsioonile tundlik
* **OSAVI**: toimib hästi nähtava pinnasega
* **EVI**: kõrge lehepinna indeksiga (LAI) piirkonnad

**Kohandatud valemid (ainult Chloros+):**

* Looge kohandatud multispektraalsed indeksivalemid
* Kasutage riba matemaatikat kõigi pildikanalitega
* Salvesta kohandatud valemid taaskasutamiseks

Kõikide kättesaadavate indeksite ja valemite kohta vaadake [Multispektraalsed indeksivalemid](../project-settings/multispectral-index-formulas.md).

### Eksport

Kontrollib väljundfaili formaati ja kvaliteeti.

**Kättesaadavad formaadid:**

* **TIFF (16-bitine)**: soovitatav GIS-i ja teaduslikuks analüüsiks (vahemik 0–65 535)
* **TIFF (32-bitine, protsent)**: ujukomaga peegeldusväärtused (vahemik 0,0–1,0)
* **PNG (8-bitine)**: kaotuseta pakkimine visualiseerimiseks (vahemik 0–255)
* **JPG (8-bitine)**: väikseimad failid, kaotusega pakkimine (vahemik 0–255)

***

## Seadete salvestamine ja laadimine

### Projekti malli salvestamine

Looge korduvkasutatavad mallid ühtsete töövoogude jaoks:

1. Konfigureerige kõik soovitud seaded projekti seadeid paneelis.
2. Kerige alla **„Salvesta projekti mall”** sektsioonini.
3. Sisestage kirjeldav malli nimi (nt „Survey3N\_RGN\_Agriculture”).
4. Klõpsake salvestamise ikooni.

**Eelised:**

* Rakenda identseid seadeid mitmes projektis.
* Jaga konfiguratsioone meeskonnaliikmetega.
* Säilita järjepidevus korduvate uuringute puhul.

### Lae mall uude projekti

Uue projekti loomisel:

1. Vali peamenüüst **„Uus projekt”**.
2. Vali valik **„Lae mallist”**.
3. Vali salvestatud mall.
4. Kõik seaded rakenduvad automaatselt.

### Töökataloog

Seade **„Salvesta projekti kaust”** määrab, kuhu uued projektid vaikimisi loodakse:

* **Vaikimisi asukoht**: `C:\Users\[Username]\Chloros Projects`
* **Asukoha muutmine**: klõpsake redigeerimisikooni ja valige uus kaust
* **Millal muuta**:
  * Võrguketas meeskonnatööks
  * Teine ketas, millel on rohkem salvestusruumi
  * Aastate/klientide järgi organiseeritud kausta struktuur

***

## PPK (järelkäsitletud kinemaatika) seadistamine

Kui kasutate MAPIR DAQ salvestusseadmeid koos GPS-iga täpse geolokatsiooni jaoks:

### Eeltingimused

* MAPIR DAQ koos GPS (GNSS) mooduliga
* .daq logifail koos ekspositsiooni pin-koodidega
* Kaamera ühendatud DAQ eksponeerimispinnidega salvestamise ajal

### Konfiguratsiooni sammud

1. Paigutage .daq logifail oma projekti kausta.
2. Projektiseadetes aktiveerige **&quot;Kohalda PPK parandusi&quot;** valikukast.
3. Vajadusel seadistage **&quot;Valgusanduri ajavööndi nihke&quot;** (vaikimisi: 0 UTC jaoks).
4. Määrake kaamerad eksponeerimispinnidele:
   * **Üks kaamera**: Määratakse automaatselt pistikule 1.
   * **Kaks kaamerat**: Määrake iga kaamera käsitsi õigele pistikule.

**Pistikute määramine:**

* **Pistik 1**: Valige kaameramudel rippmenüüst.
* **Pistik 2**: Valige teine kaamera või „Ära kasuta”.
* Sama kaamerat ei saa määrata mõlemale pistikule.

{% vihje style=&quot;warning&quot; %}
**Oluline**: Ekspositsioonipinnid peavad olema õigesti määratud vastavatele kaameratele. Vale määramine põhjustab valed geolokatsiooniandmed.
{% endhint %}

***

## Täpsemad stsenaariumid

### Mitme kaameraga projektid

Kui töödeldakse ühes projektis mitme MAPIR kaamera pilte:

1. Chloros tuvastab automaatselt iga kaamera mudeli
2. Iga kaamera saab sobiva töötlusprofiili
3. PPK: määrake igale kaamerale käsitsi õige eksponeerimise pin
4. Kõik kaamerad kasutavad sama ekspordivormingut ja indekseid

**Näide**: Survey3W RGN + Survey3N OCN kahe kaameraga seadeldis

### Aeg-lapse või mitme kuupäeva uuringud

Sama ala korduvate uuringute jaoks aja jooksul:

1. Looge mall oma standardseadete abil.
2. Kasutage igas sessioonis ühtset kalibreerimise sihtmärgi seadistust.
3. Töötlege iga kuupäeva eraldi projektina.
4. Kasutage võrreldavate tulemuste saamiseks identseid seadeid.
5. Eksportige ajalise analüüsi jaoks samas formaadis.

### Suured andmekogumid

Projektide puhul, kus on palju pilte (500+):

* Kaaluge jagamist väiksemateks projektideks kuupäeva või ala järgi.
* Kasutage Chloros+ paralleelset töötlemist kiiremate tulemuste saamiseks.
* Kaaluge CLI või API kasutamist partiide automatiseerimiseks.
* Reguleerige minimaalne kalibreerimise intervall, et vähendada sihtmärgi tuvastamise aega.

***

## Seadete kontrollimine

Enne töötlemise alustamist kontrollige järgmisi olulisi seadeid:

* [ ] Kaamera mudel on failibrauseris õigesti tuvastatud
* [ ] Vignette&#x27;i korrigeerimine on lubatud
* [ ] Peegelduskalibreerimine on lubatud
* [ ] Vähemalt üks kalibreerimise sihtmärgi pilt on imporditud
* [ ] Soovitud multispektraalsed indeksid on lisatud
* [ ] Ekspordivorming sobib teie töövooguga
* [ ] PPK-seaded on konfigureeritud (kui kasutate .daq koos eksponeerimissündmustega)

***

## Järgmised sammud

Kui seaded on konfigureeritud:

1. **Märkige kalibreerimise sihtpildid** – vt [Sihtpiltide valimine](choosing-target-images.md)
2. **Alustage töötlemist** – vt [Töötlemise alustamine](starting-the-processing.md)
3. **Jälgige protsessi kulgu** – vt [Töötlemise jälgimine](monitoring-the-processing.md)

Kõigi saadaval olevate seadete täielikud üksikasjad leiate [Projekti seaded](../project-settings/project-settings.md) viitedokumentatsioonist.
