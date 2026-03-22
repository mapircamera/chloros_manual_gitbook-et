# Indeks/LUT-katsekeskkond

Indeks/LUT-katsekeskkond on interaktiivne töökeskkond Chloros pildivaatajas, mis võimaldab teil reaalajas katsetada multispektraalsete indeksite arvutamist ja värvilisi visualiseeringuid. See võimas tööriist aitab teil testida erinevaid indekseid, täpsustada väärtuste vahemikke ja luua avaldamiseks valmis visualiseeringuid ilma kogu andmekogumit uuesti töötlemata.

## Mis on indeks/LUT-liivakast?

### Eesmärk

Liivakast pakub:

* **Reaalajas indeksite arvutamine** – rakendage mis tahes taimestikuindeksit koheselt
* **Interaktiivne LUT-i reguleerimine** – täpsustage värvigradiente ja vahemikke
* **Töövoo optimeerimine** – määrake parimad seaded enne partiitöötlust

### Sandbox vs. projekti töötlemine

**Indeksi/LUT-i Sandbox (interaktiivne):**

* Üks pilt korraga
* Kohene tagasiside
* Eksperimentaalne ja iteratiivne
* Failidesse ei tehta püsivaid muudatusi
* Sobib ideaalselt uurimiseks ja katsetamiseks

**Projekti töötlemine (partii):**

* Kogu andmekogum korraga
* Eelkonfigureeritud seaded
* Püsivad väljundfailid
* Aeganõudev
* Sobib kõige paremini, kui seaded on lõplikult paigas

{% hint style="success" %}
**Parim töövoog**: Kasutage Sandboxi eksperimenteerimiseks ja optimaalse indeksi ning LUT-seadete leidmiseks, seejärel rakendage neid seadeid kogu andmekogumi puhul projektitöötluse käigus.
{% endhint %}

***

## Töö indeks/LUT-liivakastiga

### Eelnevalt arvutatud indeksite mõistmine

Chloros-is saab indekseid rakendada projekti töötlemise käigus. Selleks, et otsustada, milliseid indeks- ja LUT-seadeid soovite ekspordile rakendada, on kõige lihtsam kasutada pildivaataja liivakasti.

Liivakast võimaldab teil:

* **Rakendada uusi indekseid ja värvigradiente (LUT-e)** andmete visualiseerimiseks
* **Reguleerida visualiseerimise seadeid** interaktiivselt
* **Vaadata** juba arvutatud indeksipilte
* **Kontrollida** pikselväärtusi kõigil suumitasanditel

### Sandboxi avamine

Indeks/LUT Sandboxile pääseb juurde **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> külgriba vahekaardilt:

1. Klõpsake failibrauseri pildiribal pildil, see avatakse **pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> vahekaardil
2. Klõpsake **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , et avada vasakpoolne väljapopiv külgriba, kui see pole veel avatud

### Pildi valimine indeks/LUT-i rakendamiseks

Indeksiga töötamiseks pildivaataja <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> liivakastis töötamiseks:

1. **Avage pilt** peamisest pildiruudust, klõpsates sellel
2. Seejärel avaneb **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> vahekaart
3. Klõpsake **kihtide rippmenüüd** (vaataja ülemises paremas nurgas)
4. Valige rippmenüüst kiht:
   * RAW (peegeldus)

### Indeksi rakendamine pildile

Kui pilt on täisekraanil ja **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> vahekaardi külgriba on avatud:

1. Märkige külgriba ülaosas asuv indeksikast
2. Valige vasakult rippmenüüst oma kaamera filter
3. Valige paremalt rippmenüüst soovitud indeksivalem
4. Lohistage filtri kanali värviringid allpool asuvasse indeksivalemisse
5. Kui valem on kehtiv, uueneb pilt ja kuvab indeksiväärtused
6. Liigutage hiirekursorit, et näha väärtusi kursori asukohas
7. Suurendage pilti, et näha üksikuid piksleid ja nendega seotud väärtusi

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

Indeksivalemite täieliku dokumentatsiooni leiate [Multispektraalsete indeksivalemite](../project-settings/multispectral-index-formulas.md) alt.

***

## Töö LUT-idega (otsingutabelid)

### Mis on LUT?

**Otsingutabel (LUT)** seostab numbrilised indeksväärtused värvidega visualiseerimiseks:

* **Sisend**: indeks-piksli väärtus (nt NDVI 0,65)
* **Väljund**: RGB värv (nt heleroheline)
* **Eesmärk**: muuta mustrid kergemini nähtavaks ja tõlgendatavaks**Halltoonide ja värviliste LUT-ide võrdlus:**

* Halltoonid: teaduslikud ja neutraalsed, näitavad toorandmeid
* Värvilised LUT-id: intuitiivsed ja mõjuvad, toovad esile mustrid ja erinevused

{% hint style="success" %}
**Visualiseerimise võimsus**: värvilise LUT-i rakendamine halltoonilise indeksipildile muudab mustrite, kõrvalekallete ja huvipakkuvate alade tuvastamise ühe pilguheiduga märkimisväärselt lihtsamaks.
{% endhint %}

### LUT-i rakendamine indeksipildile

Kui teil on indeksipilt, mis näitab

1. Klõpsake nuppu <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> nupule „+Lisa LUT“
2. Valige värvigradient
3. Reguleerige lõikamise miinimum- ja maksimumlõpp-punkte
4. Reguleerige lõikamisrežiimi
5. Märkige **Pildivaataja** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , et rakendada LUT-i

### Värvigradiendi valimine

**Gradiendi valimine:**

1. Leidke LUT-paneelis**värviline gradiendiriba**

2. Viige hiir selle kohale, et vaadata saadaval olevaid gradiendi eelseadistusi
3. Valige soovitud gradient
4. Kui indeksikast on märgitud, **uuendatakse pilt kohe** uute värvidega

{% hint style="success" %}
**Parim tava**: Taimestiku indeksite puhul, nagu NDVI, on Red-kollane-Green gradient kõige intuitiivsem, kuna see vastab looduslikele värviassotsiatsioonidele (roheline = terve, kollane = mõõdukas, punane = stressis).
{% endhint %}

### Värviklasside reguleerimine

**Klasside reguleerimisega**määratakse, mitu eraldiseisvat värvitooni astet gradiendis kuvatakse:**Klasside arvu valikud:*** **2–5 klassi**: väga laiad kategooriad, selged tsoonid
* **6–10 klassi**: tasakaalustatud, sobib hästi klassifitseerimiseks
* **11–20 klassi**: sujuvad gradiendid, järjepidev välimus
* **20+ klassi**: peaaegu järjepidev, maksimaalne sujuvus**Kuidas reguleerida:**

1. Leidke LUT-paneelis**värviproovide ruudud gradiendiriba all**

2. Reguleerige klasside arvu, lisades neid nupuga +
3. Eemaldage klasside arvu, tehes topeltklõpsu värviproovile
4. Gradient uueneb pildil **reaalajas**

**Mõju visualiseerimisele:*** **Vähem klasse** (3–5): loob selged tsoonid, lihtsustab klassifitseerimist, kategooriate eristamine on lihtsam
* **Keskmine klasside arv** (6–10): tasakaalustatud lähenemine, sobib enamikule rakendustele
* **Rohkem klasse** (15–20): sujuvad üleminekud, detailne variatsioon, fotorealistlik välimus**Millal kasutada:*** **Vähesed klassid (3–5)**: esitluse slaidid, klassifitseerimiskaardid, lihtsad aruanded
* **Keskmised klassid (6–10)**: üldine analüüs, tasakaalustatud detailid, standardaruanded
* **Palju klasse (15–20)**: teaduslik analüüs, üksikasjalik uurimine, avaldamiskvaliteediga väljundid

### Väärtuste vahemike täpsustamine

**Väärtuste vahemiku reguleerijad**määravad, millised indeksväärtused vastavad teie gradiendi millistele värvidele:**Vahemiku reguleerijad LUT-paneelis:*** **Minimaalne väärtus**: värviskaala alumine piir
* **Maksimaalne väärtus**: värviskaala ülemine piir
* **Vaheväärtused**: jaotuvad automaatselt minimaalse ja maksimaalse väärtuse vahele (klasside arvu alusel)

#### Minimaalse ja maksimaalse väärtuse reguleerimine

**Väärtuste vahemike reguleerimiseks:**

1. Leidke LUT-paneelis sisestusväljad**Minimaalne väärtus**ja**Maksimaalne väärtus**

2. Klõpsake väljal**Minimaalne väärtus**

3. Sisestage soovitud minimaalne väärtus (nt `0.2`)
4. Vajutage **Enter** või klõpsake väljaspool väljakut
5. Korrake sama **Maksimaalse väärtuse** väljaga (nt `0.9`)
6. Visualiseerimine **uuendub kohe**{% hint style="info" %}**Automaatne skaalamine**: Kui rakendate LUT-i esimest korda, seab Chloros miinimum- ja maksimumväärtused automaatselt vastavalt pildi tegelikule andmevahemikule. Seejärel saate seda vahemikku kitsendada, et keskenduda teid huvitavatele konkreetsetele väärtuste vahemikele.
{% endhint %}

**Näide NDVI vahemiku kohandamisest:*** **Täisvahemik**: `-1.0` kuni `1.0` (näita kõiki võimalikke väärtusi)
* **Taimestikule keskendunud**: `0.2` kuni `0.9` (välistab palja mulla ja vee)
* **Ainult terve taimestik**: `0.5` kuni `0.9` (esile tõsta ainult jõulised taimed)
* **Stressi tuvastamine**: `0.2` kuni `0.5` (rõhutada probleemsed alad)
* **Kohandatud vahemik**: kohandada vastavalt teie täheldatud pikselväärtustele**Miks vahemikke kohandada?*** **Suurendada kontrasti** teie huvipakkuvas piirkonnas
* **Jätke välja ebaolulised väärtused** (nt veekogud, paljas pinnas)
* **Ühtlustage visualiseerimine** mitme pildi või kuupäeva vahel
* **Rõhutage peeneid erinevusi** kitsas väärtuste vahemikus

### Väljaspool vahemikku jäävate väärtuste kärpimine

Kui pikseliväärtused jäävad väljapoole määratud miinimum-/maksimumvahemikku, saate **kärpimisrežiimide** abil kontrollida, kuidas neid kuvatakse.

#### **Saadaval olevad kärpimisrežiimide valikud:**

#### 1. Minimaalne ja maksimaalne

* Pikslid **alla minimaalse**→ kuvatakse kasutades gradiendi**esimest värvi** (nt punane)
* Pikslid **üle maksimaalse**→ kuvatakse kasutades gradiendi**viimast värvi** (nt roheline)
* **Kasutusjuhtum**: Rõhutage äärmusväärtusi, näidake kogu andmevahemikku piiridel küllastunud värvidega
* **Näide**: NDVI väärtused alla 0,2 kuvatakse kõik punasena, väärtused üle 0,9 kuvatakse kõik rohelisena

#### 2. Läbipaistev taust

* **Vahemikust väljaspool olevad**pikslid muutuvad**täielikult läbipaistvaks*** Ainult **vahemikus olevad** pikslid kuvavad värvigradienti
* **Kasutusjuhtum**: GIS-kiht, teatud väärtuste vahemike eraldamine, ainult huvipakkuvate alade esiletõstmine
* **Näide**: Näita värvilisena ainult NDVI 0,4–0,7, kõik muu läbipaistev

{% hint style="warning" %}
**Läbipaistvuse piirang**: Läbipaistvad pikslid kuvatakse vaatajas taustavärvina. Töötlemise käigus eksportimisel säilib läbipaistvus PNG-vormingus, kuid mitte JPG-vormingus.
{% endhint %}

#### 3. Indeksi taust

* **Vahemikust väljaspool**olevad pikslid kuvatakse**halltoonides** (näidates tooreid indeksväärtusi)
* **Vahemikus**olevad pikslid kuvatakse**värvigradiendina*** **Kasutusjuhtum**: peenike esiletõstmine, konteksti säilitamine huvipakkuvate alade esiletõstmisel
* **Näide**: värviga esiletõstetud stressis taimestik (NDVI 0,3–0,5), terved alad kuvatakse hallina

#### 4. Algne taust

* **Vahemikust väljaspool**olevad pikslid kuvavad**algset multispektraalset pilti*** **Vahemikku jäävad**pikslid näitavad**värvigradienti*** **Kasutusjuhtum**: Kõige intuitiivsem – ühendab loomuliku pildi konteksti analüütilise värvikihtiga
* **Näide**: Vaata tegelikku põllu/põllukultuuri välimust, millele on peale kantud värvikoodiga stressialad

### Õige lõikamisrežiimi valimine

| Lõikamisrežiim              | Sobib kõige paremini                                   | Visualiseerimisstiil          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum ja maksimum**    | Täielik andmete kuvamine, teaduslik analüüs     | Kõik pikslid värvitud           |
| **Läbipaistev taust** | GIS-kihtide lisamine, konkreetsete vahemike eraldamine    | Värv vahemikus, tühjaks väljaspool |
| **Indeksitaust**       | Õrn rõhutamine, andmete konteksti säilitamine  | Värv vahemikus, hall väljaspool  |
| **Algne taust**    | Aruanded, esitlused, intuitiivne analüüs | Värv vahemikus, foto väljaspool |

### Kohandatud LUT-värvide loomine

Visualiseerimise täielikuks kontrollimiseks saate luua **kohandatud värvigradiente**, muutes üksikuid värvipeatusi.**Kohandatud gradiendi loomiseks:**

1. Leidke LUT-paneelist**gradiendi eelvaate riba**

2. Otsige gradiendi all**värviproovide ruudud**

3.**Klõpsake värvipeatusele**, et see valida
4. Avaneb **värvivalija**

5. Valige uus värv, kasutades:
   * **Värviring**: visuaalne värvivalik
   * **RGB/HSV liugurid**: täpne värvihaldus
   * **Hex-koodi sisestamine**: täpne värvispetsifikatsioon (nt `#FF0000` punase jaoks)
6. Klõpsake värvivalija väljaspool **uue värvi rakendamiseks**

7. Gradient**uuendub kohe** pildil**Värvipeatuste lisamine või eemaldamine:*** **Peatuse lisamine**: Klõpsake ikoonile +, et lisada uus värviproov lõppu
* **Peatuse eemaldamine**: Topeltklõpsake värviruudule, et värviproov eemaldada**Kohandamisstrateegiad:*** **Gradiendi pööramine**: Pöörake värvide järjekorda, et muuta tähendust (nt roheline = madal, punane = kõrge)
* **Brändivärvid**: Sobitage aruannete värvipalett oma organisatsiooni värvipalettiga
* **Värvipimedatele sobiv**: Kasutage oranži-sinise või lilla-kollase kombinatsioone
* **Trükkimise optimeerimine**: vali värvid, mis sobivad nii värvi- kui ka halltoonides trükkimiseks
* **Mitme lävega**: kasuta klassifitseerimiseks erinevaid värve kindlate väärtuste lävede juures

{% hint style="info" %}
**Kohandatud gradientide salvestamine**: kohandatud gradiente saab salvestada ja uuesti kasutada. Klõpsa LUT-paneelis salvestamise ikoonil, et säilitada oma kohandatud värviskeemid tulevaseks kasutamiseks.
{% endhint %}

***

## Interaktiivne töövoog

### Reaalajas uuendused

Kõik LUT-i kohandused liivakastis uuendavad pilti **koheselt ja interaktiivselt**:

* **Vaheta kihti** → Pilt muutub kohe
* **Vali gradient** → Värvid uuenevad koheselt
* **Väärtuste vahemiku reguleerimine** → Kontrast muutub reaalajas
* **Klasside muutmine** → Gradiendi sujuvus uueneb kohe
* **Kärpimise muutmine** → Taust kuvatakse kohe teisiti
* **Värvide muutmine** → Kohandatud gradient rakendub kohe**&quot;Rakenda&quot; nuppu pole vaja** – kõik muudatused on reaalajas ja interaktiivsed!

{% hint style="success" %}
**Reaalajas tagasiside**: kohene visuaalne tagasiside võimaldab teil kiiresti katsetada erinevaid seadeid, kuni leiate oma analüüsivajadustele optimaalse visualiseerimise.
{% endhint %}

### Iteratiivne täiustamise töövoog

**Tüüpiline LUT-optimeerimise töövoog:**

1.**Valige indeksikiht** (nt RAW (peegeldus))
2. **Rakendage indeks** – valige kaamerafilter ja indeksivalem, lohistage värvilised ringid indeksivalemis sobivasse asukohta
3. **Rakendage LUT-gradient** – alustage eelseadistusega Red-Yellow-Green
4. **Kontrollige pikselväärtusi** – liigutage kursorit ringi, märkige väärtuste vahemikud
5. **Reguleeri miinimum/maksimum** – kitsenda vahemikku, et keskenduda taimestikule (nt 0,2 kuni 0,9)
6. **Vali kärpimine** – proovi konteksti jaoks „Original Background”
7. **Täpsusta värve** – kohanda gradienti vajaduse korral konkreetse rõhuasetuse jaoks
8. **Kinnita seaded**– dokumenteeri seaded ja kopeeri need projekti seadetesse eksportimiseks

### Pikselväärtuste kontrollimine

Tõhusate LUT-vahemike seadistamiseks on oluline mõista tegelikke pikselväärtusi:**Kuidas väärtusi kontrollida:**

1. Pikselväärtused kuvatakse, kui pildil on märgitud kas „Index” või nii „Index” kui ka „LUT”**ruudud**.
2. **Liigutage kursorit** pildi erinevate alade kohale
3. **Vaadake pikselväärtusi**, mis kuvatakse legendis, kui kursoriga üle pildi liigute
4. Suurendage pilti, et näha üksikuid piksleid, mis on esile tõstetud ujuva väärtusega
5. **Tehke märkmeid** erinevate objektide väärtuste vahemike kohta:
   * **Terve taimestik**: nt NDVI 0,55–0,85
   * **Stressis taimestik**: nt NDVI 0,30–0,50
   * **Paljas pinnas**: nt NDVI 0,05–0,25
   * **Vesi** (kui olemas): nt NDVI -0,05 kuni 0,10**Piksli väärtuste kasutamine LUT-vahemike seadmiseks:**Pärast piksli väärtuste kontrollimist kohandage oma LUT-i miinimum- ja maksimumväärtusi vastavalt:**Näide:*** **Vaatlus**: Mulla väärtused = 0,05–0,25, stressis = 0,25–0,50, terved = 0,50–0,85
* **Eesmärk**: visualiseerida ainult taimede tervislikku seisundit (välja arvatud pinnas)
* **LUT-seaded**: Min = `0.25`, Max = `0.85`
* **Kärpimine**: „Algne taust”, et näha mulda loomulikus värvis
* **Tulemus**: Värvigradient kehtib ainult taimestikule, muld kuvatakse algse pildina

{% hint style="info" %}
**Dünaamiline ulatus**: Erinevatel põllukultuuridel, aastaaegadel ja kasvufaasidel on erinevad väärtuste vahemikud. Kontrollige alati oma konkreetse andmekogumi pikselväärtusi enne LUT-vahemike seadistamist.
{% endhint %}

***

## Kohandatud indeksid (Chloros+)

### Kohandatud indeksivalemite loomine

{% hint style="info" %}
**Kus luua**: Kohandatud indekseid saab konfigureerida**projekti seadetest** enne töötlemist, samuti pildivaataja sandboxi külgribal.
{% endhint %}

**Kohandatud indeksi loomiseks:**

1.**Avage projekti seaded** (enne töötlemist) või pildivaataja liivakasti külgriba
2. Navigeerige **indeksivalemi rippmenüüsse**

3. Otsige valikut**„Kohandatud”** (peate olema sisse logitud Chloros+ litsentsiga)
4. **Määrake oma valem**, kasutades vahemiku muutujaid:
   * Vahemiku nimed: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` jne.
   * Operatsioonid: `+`, `-`, `*`, `/`, `^` (eksponent)
   * Funktsioonid: `sqrt()`, `abs()` jne. (kui toetatakse)
   * Sulgudes: `()` tehete järjekorra jaoks
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

{% hint style="warning" %}
**Valemi valideerimine**: Veenduge, et teie valem kasutab teie kaameras saadaval olevaid lainepikkusi. Näiteks RedEdge on saadaval ainult kaameratel, millel on RedEdge filter.
{% endhint %}

***

## Järgmised sammud

Nüüd, kui te mõistate indeks/LUT-liivakasti:

* **Rakendage töötlemisel**: kasutage leitud seadeid [Projekti seaded](../project-settings/project-settings.md)
* **Patsitöötlus**: rakendage optimeeritud indekseid kogu andmekogumile
* **Lisateave**: lugege [Multispektraalsed indeksivalemid](../project-settings/multispectral-index-formulas.md)

Seotud dokumentatsioon:

* [**Pildikiht**](image-layers.md) – Kihtide haldamine ja visualiseerimine
* [**Pildi avamine täisekraanil**](opening-an-image-full-screen.md) – Pildivaataja põhitõed
* [**Piltide töötlemine (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Täielik töötlemisprotsess
