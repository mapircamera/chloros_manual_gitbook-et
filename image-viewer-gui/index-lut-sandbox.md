# Indeks/LUT-liivakast

Indeks/LUT-liivakast on interaktiivne töökeskkond Chloros pildivaatajas, mis võimaldab teil katsetada multispektraalsete indeksite arvutamist ja värvide visualiseerimist reaalajas. See võimas tööriist aitab teil testida erinevaid indekseid, täpsustada väärtuste vahemikke ja luua avaldamiseks valmis visualiseeringuid ilma kogu andmekogumit uuesti töötlemata.

## Mis on indeks/LUT-liivakast?

### Eesmärk

Liivakast pakub:

* **Reaalajas indeksarvutusi** – rakendage mis tahes taimestiku indeksit koheselt
* **Interaktiivset LUT-reguleerimist** – täpsustage värvide gradiente ja vahemikke
* **Töövoo optimeerimist** – määrake parimad seaded enne partii töötlemist

### Sandbox vs. projekti töötlemine

**Index/LUT Sandbox (interaktiivne):**

* Üks pilt korraga
* Kiire tagasiside
* Eksperimentaalne ja iteratiivne
* Failidesse ei tehta püsivaid muudatusi
* Ideaalne uurimiseks ja testimiseks

**Projekti töötlemine (partii):**

* Kogu andmekogum korraga
* Eelkonfigureeritud seaded
* Püsivad väljundfailid
* Aeganõudev
* Parim, kui seaded on lõplikud

{% vihje style=&quot;success&quot; %}
**Parim töövoog**: kasutage Sandboxi eksperimenteerimiseks ja optimaalse indeksi ja LUT-seadete leidmiseks, seejärel rakendage neid seadeid projekti töötlemise ajal kogu andmekogumile.
{% endhint %}

***

## Töö indeksiga/LUT Sandboxiga

### Eelnevalt arvutatud indeksite mõistmine

Chloros-is saab indekseid rakendada projekti töötlemise ajal. Ekspordile rakendatavate indeksite ja LUT-seadete kindlaksmääramiseks on kõige lihtsam kasutada pildivaataja sandboxi.

Sandbox võimaldab teil:

* **Rakendada uusi indekseid ja värvigradiente (LUT-e)** andmete visualiseerimiseks.
* **Kohandada visualiseerimise seadeid** interaktiivselt.
* **Vaadata** juba arvutatud indekspilte.
* **Kontrollida** pikselväärtusi kõikidel suumitasemetel.

### Sandboxi avamine

Indeksi/LUT-i sandboxile pääseb **pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> külgriba kaardilt:

1. Klõpsake failibrauseri pildiristil pilti, see avatakse **pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> vahekaardil
2. Klõpsake **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , et avada vasakpoolne külgriba, kui see pole veel avatud

### Indeksi/LUT-i rakendamiseks pildi valimine

Indeksi kasutamiseks pildivaatajas <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandboxis:

1. **Avage pilt** peamisest pildivõrgust, klõpsates sellel
2. Seejärel avatakse **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> vahekaart
3. Klõpsake **kihtide rippmenüüd** (vaataja ülemises paremas nurgas)
4. Valige rippmenüüst kiht:
   * RAW (peegeldus)

### Indeksi rakendamine pildile

Kui pilt on täisekraanil ja **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> vaate riba on avatud:

1. Märkige riba ülaosas asuv indeksikast
2. Valige vasakpoolsest rippmenüüst oma kaamera filter
3. Valige parempoolsest rippmenüüst soovitud indeksivalem
4. Lohistage filtri kanali värviringid allpool olevasse indeksivalemisse
5. Kui valem on kehtiv, uuendatakse pilt ja kuvatakse indeksiväärtused
6. Liigutage hiirekursorit, et näha väärtusi kursori asukohas
7. Suurendage pilti, et näha üksikuid piksleid ja nendega seotud väärtusi.

Igal indeksil on kindel väärtuste vahemik ja tähendus:

#### NDVI Näide

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Täieliku indeksivalemite dokumentatsiooni leiate [Multispektraalsete indeksivalemite](../project-settings/multispectral-index-formulas.md) alt.

***

## Töö LUT-idega (otsingutabelid)

### Mis on LUT?

**Otsingutabel (LUT)** seostab numbrilised indeksiväärtused värvidega visualiseerimiseks:

* **Sisend**: indeksi pikseliväärtus (nt NDVI 0,65)
* **Väljund**: RGB värv (nt ere roheline)
* **Eesmärk**: muuta mustrid kergemini nähtavaks ja tõlgendatavaks

**Halltoonide ja värvide LUT:**

* Halltoonid: teaduslikud ja neutraalsed, näitavad toorandmeid
* Värvide LUT: intuitiivsed ja mõjukad, rõhutavad mustreid ja erinevusi

{% hint style=&quot;success&quot; %}
**Visualiseerimise võimsus**: värvilise LUT-i rakendamine halltoonide indekspildile muudab mustrite, kõrvalekallete ja huvipakkuvate alade tuvastamise ühe pilguheiduga oluliselt lihtsamaks.
{% endhint %}

### LUT-i rakendamine indekspildile

Kui teil on indekspilt, mis näitab

1. Klõpsake <img src="../.gitbook/assets/image.png" alt="" data-size="line"> „+Lisa LUT” nuppu
2. Valige värvi gradient
3. Reguleerige lõikamise min/max lõpp-punkte
4. Reguleerige lõikamisrežiimi
5. Märkige **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> kõrvalribal, et rakendada LUT-i.

### Värvi gradiendi valimine

**Gradiendi valimine:**

1. Leidke LUT-paneelis **värviline gradiendi riba**.
2. Viige hiir selle kohale, et vaadata saadaval olevaid gradiendi eelseadistusi.
3. Valige soovitud gradient.
4. Kui indeksikast on märgitud, **uuendatakse pilt kohe** uute värvidega.

{% vihje style=&quot;success&quot; %}
**Parim tava**: Taimestiku indeksite puhul, nagu NDVI, on Red-Yellow-Green gradient kõige intuitiivsem, kuna see vastab looduslikele värviassotsiatsioonidele (roheline = terve, kollane = mõõdukas, punane = stressis).
{% endhint %}

### Värviklasside kohandamine

**Klasside kontroll** määrab, kui palju eraldiseisvaid värvitoone teie gradiendis ilmub:

**Klasside arvu valikud:**

* **2–5 klassi**: väga laiad kategooriad, eristatavad tsoonid
* **6–10 klassi**: tasakaalustatud, sobivad klassifitseerimiseks
* **11–20 klassi**: sujuvad gradientid, järjepidev välimus
* **20+ klassi**: peaaegu järjepidevad, maksimaalne sujuvus

**Kuidas reguleerida:**

1. LUT-paneelis otsige **värvivalikuväljad gradientribast allpool**.
2. Reguleerige klasside arvu, lisades neid + nupuga.
3. Eemaldage klasside arv, klõpsates värvivalikuväljal topeltklõpsuga.
4. Gradient uueneb pildil **reaalajas**.

**Mõju visualiseerimisele:**

* **Vähem klasse** (3–5): loob eristatavad tsoonid, lihtsustab klassifitseerimist, lihtsustab kategooriate eristamist.
* **Keskmine arv klasse** (6–10): tasakaalustatud lähenemine, sobib enamikule rakendustele.
* **Rohkem klasse** (15–20): sujuvad üleminekud, detailne variatsioon, fotorealistlik välimus.

**Millal kasutada:**

* **Vähe klasse (3–5)**: esitluse slaidid, klassifitseerimiskaardid, lihtsad aruanded
* **Keskmised klassid (6–10)**: üldine analüüs, tasakaalustatud detailid, standardaruanded
* **Palju klasse (15–20)**: teaduslik analüüs, detailne kontroll, publikatsioonikvaliteediga väljundid

### Väärtuste vahemike täpsustamine

**Väärtuste vahemiku kontrollid** määravad, millised indeksväärtused vastavad teie gradiendi värvidele:

**Vahemiku kontrollid LUT-paneelis:**

* **Minimaalne väärtus**: värviskaala alumine piir
* **Maksimaalne väärtus**: värviskaala ülemine piir
* **Vaheväärtused**: jaotatakse automaatselt minimaalse ja maksimaalse väärtuse vahel (klasside arvu alusel)

#### Minimaalse/maksimaalse väärtuse reguleerimine

**Väärtuste vahemike reguleerimiseks:**

1. Leidke LUT-paneelis sisestusväljad **Minimaalne väärtus** ja **Maksimaalne väärtus**.
2. Klõpsake väljal **Minimaalne väärtus**.
3. Sisestage soovitud minimaalne väärtus (nt `0.2`).
4. Vajutage **Enter** või klõpsake väljaspool välja.
5. Korrake sama **Maksimaalne väärtus** väljaga (nt `0.9`).
6. Visualiseerimine **uuendatakse kohe**.

{% vihje style=&quot;info&quot; %}
**Automaatne skaalamine**: kui rakendate LUT-i esimest korda, seab Chloros automaatselt miinimum-/maksimumväärtuse pildi tegeliku andmevahemiku järgi. Seejärel saate seda vahemikku kitsendada, et keskenduda konkreetsetele huvipakkuvatele väärtuste vahemikele.
{% endhint %}

**Näide NDVI vahemiku kohandamisest:**

* **Täisvahemik**: `-1.0` kuni `1.0` (näita kõiki võimalikke väärtusi)
* **Taimestikule keskendunud**: `0.2` kuni `0.9` (välja arvatud paljas pinnas ja vesi)
* **Ainult terve taimestik**: `0.5` kuni `0.9` (esile tõstetakse ainult jõulised taimed)
* **Stressi tuvastamine**: `0.2` kuni `0.5` (rõhutatakse probleemsed alad)
* **Kohandatud vahemik**: kohandage vastavalt oma täheldatud pikselväärtustele

**Miks vahemikke kohandada?**

* **Suurendage kontrasti** oma huvipakkuvas piirkonnas
* **Jätke välja ebaolulised väärtused** (nt veekogud, paljas pinnas)
* **Ühtlustage visualiseerimine** mitme pildi või kuupäeva puhul
* **Rõhutage väikeseid erinevusi** kitsas väärtuste vahemikus

### Vahemikust väljuvate väärtuste kärpimine

Kui pikseliväärtused jäävad väljapoole määratud min/max vahemikku, saate **kärpimisrežiimide** abil kontrollida, kuidas need kuvatakse.

#### **Saadaval olevad kärpimisrežiimi valikud:**

#### 1. Minimaalne ja maksimaalne

* Pikselid **alla minimaalse väärtuse** → kuvatakse gradientide **esimese värviga** (nt punane)
* Pikslid **üle maksimumi** → kuvatakse gradientide **viimase värviga** (nt roheline)
* **Kasutusjuhtum**: rõhutage äärmuslikke väärtusi, näidake kogu andmevahemikku küllastunud värvidega piiridel
* **Näide**: NDVI väärtused alla 0,2 kuvatakse punasena, väärtused üle 0,9 kuvatakse rohelisena

#### 2. Läbipaistev taust

* **Vahemikust väljaspool** olevad pikslid muutuvad **täielikult läbipaistvaks**
* Ainult **vahemikus** olevad pikslid näitavad värvi gradienti
* **Kasutusjuhtum**: GIS-kiht, eraldades konkreetsed väärtuste vahemikud, esile tõstes ainult huvipakkuvad alad
* **Näide**: Näita ainult NDVI 0,4–0,7 värviga, kõik muu läbipaistev

{% hint style=&quot;warning&quot; %}
**Läbipaistvuse piirang**: Läbipaistvad pikslid kuvatakse vaatajas taustavärvina. Töötlemise käigus ekspordimisel säilib läbipaistvus PNG formaadis, kuid mitte JPG-formaadis.
{% endhint %}

#### 3. Indeksi taust

* **Vahemikust väljaspool** olevad pikslid kuvatakse **halltoonides** (näidates toorindeksi väärtusi)
* **Vahemikus** olevad pikslid kuvatakse **värvi gradiendina**
* **Kasutusjuhtum**: õrn esiletõstmine, konteksti säilitamine ja huvipakkuvate alade esiletõstmine
* **Näide**: värviga esiletõstetud stressis taimestik (NDVI 0,3–0,5) ja terved alad halli värviga

#### 4. Algne taust

* **Vahemikust väljaspool** olevad pikslid kuvatakse **algse multispektraalse pildina**
* **Vahemikus olevad** pikslid näitavad **värvi gradienti**
* **Kasutusjuhtum**: kõige intuitiivsem – ühendab loomuliku pildi konteksti analüütilise värvi ülekattumisega
* **Näide**: vaadake tegelikku põllu/saagi välimust värvikoodiga stressipiirkondade ülekattumisega

### Õige lõikamisrežiimi valimine

| Lõikamisrežiim              | Sobib kõige paremini                                   | Visualiseerimisstiil          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimaalne ja maksimaalne**    | Täielik andmete kuvamine, teaduslik analüüs     | Kõik pikslid värvitud           |
| **Läbipaistev taust** | GIS-kihid, konkreetsete vahemike eraldamine    | Värv vahemikus, tühjad väljaspool |
| **Indeksi taust**       | Õrn rõhutamine, andmete konteksti säilitamine  | Värv vahemikus, hall väljaspool  |
| **Algne taust**    | Aruanded, esitlused, intuitiivne analüüs | Värv vahemikus, foto väljaspool |

### Kohandatud LUT-värvide loomine

Visualiseerimise täieliku kontrolli saavutamiseks saate luua **kohandatud värviüleminekuid**, muutes üksikuid värvipunkte.

**Kohandatud gradiendi loomiseks:**

1. Leidke LUT-paneelis **gradiendi eelvaate riba**.
2. Otsige gradiendi all **värviproovide ruudud**.
3. **Klõpsake värvipunkti**, et see valida.
4. Avaneb **värvivalija**.
5. Valige uus värv, kasutades:
   * **Värviratast**: visuaalne värvivalik
   * **RGB/HSV liugureid**: täpne värvihaldus
   * **Hex-koodi sisestamist**: täpne värvispetsifikatsioon (nt `#FF0000` punase jaoks)
6. Klõpsake värvivalijalt ära, **et rakendada uus värv**
7. Gradient **uuendatakse kohe** pildil

**Värvipeatuste lisamine või eemaldamine:**

* **Peatuse lisamine**: klõpsake ikoonil +, et lisada uus värviproov lõppu
* **Peatuse eemaldamine**: klõpsake värviruudul topeltklõpsuga, et eemaldada värviproov

**Kohandamisstrateegiad:**

* **Kalle pööramine**: pöörake värvide järjekord, et muuta tähendus vastupidiseks (nt roheline = madal, punane = kõrge)
* **Brändi värvid**: sobitage oma organisatsiooni värvipalett aruannetega
* **Värvipimedatele sobiv**: kasutage oranži-siniseid või lilla-kollaseid kombinatsioone
* **Trükkimise optimeerimine**: valige värvid, mis sobivad nii värvilisele kui ka halltoonilisele trükkimisele
* **Mitme lävega**: kasutage klassifitseerimiseks erinevaid värve kindlate väärtuste lävedel

{% vihje style=&quot;info&quot; %}
**Kohandatud gradientide salvestamine**: kohandatud gradientide saab salvestada ja uuesti kasutada. Klõpsake LUT-paneelis salvestamise ikooni, et säilitada kohandatud värviskeemid tulevikus kasutamiseks.
{% endhint %}

***

## Interaktiivne töövoog

### Reaalajas uuendused

Kõik LUT-i kohandused liivakastis uuendavad pilti **koheselt ja interaktiivselt**:

* **Vaheta kiht** → Pilt muutub kohe
* **Vali gradient** → Värvid uuendatakse kohe
* **Kohanda väärtuste vahemikku** → Kontrast muutub reaalajas
* **Muuda klasse** → Gradienti sujuvus uuendatakse kohe
* **Muuda lõikamist** → Taust kuvatakse kohe
* **Muuda värve** → Kohandatud gradient rakendub kohe

**Ei ole vaja nuppu „Rakenda”** – kõik muudatused on reaalajas ja interaktiivsed!

{% vihje stiil=&quot;edu&quot; %}
**Reaalajas tagasiside**: kohene visuaalne tagasiside võimaldab teil kiiresti katsetada erinevaid seadeid, kuni leiate oma analüüsi vajadustele optimaalse visualiseerimise.
{% endhint %}

### Iteratiivne täiustamise töövoog

**Tüüpiline LUT optimeerimise töövoog:**

1. **Valige indeksikiht** (nt RAW (peegeldus))
2. **Rakenda indeks** – vali kaamera filter ja indeksivalem, lohista värvilised ringid indeksivalemis sobivasse asukohta
3. **Rakenda LUT-gradient** – alusta eelseadistusega Red-Yellow-Green
4. **Kontrollige pikseliväärtusi** – liigutage kursorit, märkige väärtuste vahemikud
5. **Reguleerige min/max** – kitsendage, et keskenduda taimestikule (nt 0,2 kuni 0,9)
6. **Valige lõikamine** – proovige „Original Background” konteksti jaoks
7. **Värvide täpsustamine** – kohandage gradienti vajaduse korral konkreetse rõhuasetuse jaoks
8. **Seadete lõplik kinnitamine** – dokumenteerige seadistused ja kopeerige need ekspordi töötlemiseks projekti seadistustesse

### Pikselväärtuste kontrollimine

Tegelikute pikselväärtuste mõistmine on oluline tõhusate LUT-vahemike seadistamiseks:

**Kuidas väärtusi kontrollida:**

1. Pikselväärtused kuvatakse, kui pildil on märgitud kas indeks või nii indeks kui ka LUT **ruudud**.
2. **Liigutage kursorit** pildi erinevate alade kohale
3. **Vaadake pikselväärtusi**, mis kuvatakse legendis, kui kursor on nende kohal
4. Suurendage pilti, et näha üksikuid piksleid, mis on esile tõstetud ujuva väärtusega
5. **Tee märkmeid** erinevate omaduste väärtuste vahemike kohta:
   * **Terve taimestik**: nt NDVI 0,55–0,85
   * **Stressis taimestik**: nt NDVI 0,30–0,50
   * **Paljas pinnas**: nt NDVI 0,05–0,25
   * **Vesi** (kui olemas): nt NDVI -0,05 kuni 0,10

**Pikselväärtuste kasutamine LUT-vahemike seadistamiseks:**

Pärast pikselväärtuste kontrollimist kohandage vastavalt oma LUT min/max:

**Näide:**

* **Vaatlus**: Mulla väärtused = 0,05–0,25, stressis = 0,25–0,50, terved = 0,50–0,85
* **Eesmärk**: visualiseerida ainult taimede tervis (välja arvatud pinnas)
* **LUT-seaded**: Min = `0.25`, Max = `0.85`
* **Kärpimine**: „Algne taust”, et näha pinnast loomulikus värvis
* **Tulemus**: värvi gradient kehtib ainult taimestiku kohta, muld kuvatakse originaalpildina

{% hint style=&quot;info&quot; %}
**Dünaamiline ulatus**: erinevatel põllukultuuridel, aastaaegadel ja kasvufaasidel on erinevad väärtuste vahemikud. Enne LUT-vahemike seadistamist kontrollige alati oma konkreetses andmekogus pikseliväärtusi.
{% endhint %}

***

## Kohandatud indeksid (Chloros+)

### Kohandatud indeksivalemite loomine

{% hint style=&quot;info&quot; %}
**Kus luua**: Kohandatud indeksid saab konfigureerida **projekti seadetest** enne töötlemist, samuti pildivaataja sandboxi külgribal.
{% endhint %}

**Kohandatud indeksi loomiseks:**

1. **Ava projekti seaded** (enne töötlemist) või pildivaataja liivakasti külgriba.
2. Mine **indeksi valemi rippmenüüsse**.
3. Otsi **„Kohandatud”** valik (pead olema sisse logitud Chloros+ litsentsiga).
4. **Määrake valem**, kasutades ribamuutujaid:
   * Ribade nimed: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` jne.
   * Operaatorid: `+`, `-`, `*`, `/`, `^` (eksponent)
   * Funktsioonid: `sqrt()`, `abs()` jne (kui toetatakse)
   * Sulgudes: `()` operatsioonide järjekorra jaoks
5. **Nimetage oma indeks** (nt „MyIndex” või „CustomNDVI”)
6. **Salvestage konfiguratsioon**

**Näited kohandatud valemitest:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;warning&quot; %}
**Valemi valideerimine**: Veenduge, et valemis kasutatakse teie kaameras saadaval olevaid sagedusribasid. Näiteks RedEdge on saadaval ainult kaamerates, millel on RedEdge filter.
{% endhint %}

***

## Järgmised sammud

Nüüd, kui saate aru indeksist/LUT-i liivakastist:

* **Rakenda töötlemisele**: kasuta avastatud seadeid [projekti seadetest](../project-settings/project-settings.md)
* **Partii töötlemine**: rakendage optimeeritud indeksid kogu andmekogumile
* **Lisateave**: lugege [Multispektraalsed indeksivalemid](../project-settings/multispectral-index-formulas.md)

Seotud dokumentatsioon:

* [**Pildikihtid**](image-layers.md) – kihtide haldamine ja visualiseerimine
* [**Pildi avamine täisekraanil**](opening-an-image-full-screen.md) – pildivaataja põhitõed
* [**Piltide töötlemine (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – täielik töötlemise töövoog
