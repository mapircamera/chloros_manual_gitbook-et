# Piltide raster

Pärast piltide projekti importimist näed neid peavaldkonnas rasterina paigutatuna. Rasteris saad valida, **millist versiooni igast pildist soovid vaadata** — selle kohal asuvad nupud lülitavad kõik pisipildid korraga allikafailide ja töödeldud tulemuste vahel.

## Pisipiltide suurus

Kasutage paremas ülemises nurgas asuvat suumiliugurit, et reguleerida piltide pisipiltide suurust. Liugur ulatub **64 pikselist 1200 pikselini**.

* **Ctrl + hiirerattaga** saab pisipilte samuti suurendada või vähendada.
* **Ctrl + `+`**/**Ctrl + `=`**ja**Ctrl + `−`** suurendavad või vähendavad suurust iga vajutuse kohta 4 px võrra. Klaviatuurikombinatsioonide abil saab suurust vähendada kuni 64 px-ni ja suurendada kuni suuruseni, mis mahutab praegusesse aknasse täpselt kaks pisipilti rea kohta.
* Valitud suurus salvestatakse koos projektiga (`UI → Grid thumbnail size` kuni `project.json`, vaikimisi `160`), seega taastub see projekti uuesti avamisel.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>Pisipildi *lahutusvõime* on pisipildi *suurusest* eraldi seade: vaata **Ekraan → Pisipildi lahutusvõime** [Projekti seaded](../project-settings/project-settings.md) (vaikimisi 512 pikselit pikemal serval). Suurus näitab, kui suureks ruut joonistatakse; eraldusvõime näitab, kui palju detaile selle täitmiseks laaditakse.

***

## Võrgustiku tööriistariba

Võrgustiku kohal asuv nupurida koosneb kuni kolmest rühmast, vasakult paremale:

1. **Iga käivitaja / Iga kaamera** — rühmitamisrežiim. Ilmub ainult projektides, mis sisaldavad LATTICE’i salvestusi.
2. **Kaamerafiltri nupud** — üks iga LATTICE’i kaamera kohta. Ilmuvad ainult režiimis „Iga kaamera kohta”.
3. **Ekspordi-/vaatamisrežiimi nupud** — millist toodet iga pisipilt näitab.

Kui aken on nende kõigi jaoks liiga kitsas, kokkuvarisevad rühmad paremalt vasakule hiirega üle libistamisel avanevate rippmenüüdeks: esmalt varisevad kokku eksportimise/vaatamise nupud, seejärel kaameranupud. Kokkuvarisenud rühmast jääb alles üksainus nupp, millel on märgitud hetkel aktiivne valik, ja selle peale hiirega libistamisel liigub kogu komplekt alla. **Nupud „Per Trigger” / „Per Camera” ei kokku klapi kunagi.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

***

## Ekspordi- ja vaatamisnupud

Need nupud lülitavad pisipiltide ristikku ühelt pilditüübilt teisele. **Nupp ilmub niipea, kui selle nimega toode on olemas** — mis allikafailide puhul tähendab kohe importimisel, mitte pärast töötlemist. Chloros skaneerib projekti tooted uuesti, kui töötlemine on käimas, seega ilmuvad nupud töötlemise ajal, kui iga toode hakkab kettale salvestuma.

### Põhinupp

Kõige vasakpoolsem ekspordinupp on märgistatud **sellega, mida tegelikult importisite**:

| Mida importisite | Nupu silt |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| LATTICE-kaadrid, mille juures on ekraanil eelvaade koos RAW-kaadriga | `PNG` või `TIFF`, sõltuvalt sellest, millised eelvaated on |
| LATTICE-pildid, kus põhiline fail **on** RAW-kaader | *nuppu pole* — `RAW (Original)` näitab seda faili juba |

Segaprojektis järgib silt seda laiendit, mida enamik pilte kasutab.

### Toote nupud

| Nupp | Näitab | Millal ilmub |
| --- | --- | --- |
| **Sihtmärgid** | Pildid, millel on tuvastatud kalibreerimissihmärk | Pärast töötlust, mille käigus sihtmärgid tuvastati |
| **Peegeldusvõime** | Kalibreeritud peegeldusvõime pildid | Ainult Survey3 projektides — LATTICE projektides kasutatakse selle asemel `RAW (Reflectance)`, seega ei kuvata võrgustikus kunagi kahte peegeldusvõime nuppu |
| **Valge tasakaalustatud** | Valge tasakaalustatud tulemus (RGB kaamerad) | Pärast töötlemist |
| **Vignettkorrektsioon** | Kalibreerimata, vignettkorrektsiooniga varuvariant | Pärast töötlust, kus peegelduskalibreerimist ei olnud võimalik rakendada ja *Vignettkorrektsioon* oli sisse lülitatud |
| **Anduri tundlikkus** | Kalibreerimata anduri tundlikkuse varuvariant | Sama, kuid *Vignettkorrektsioon* on välja lülitatud |
| **`RAW (<INDEX> Index)`** | Üks nupp iga arvutatud indeksi kohta | Pärast töötsüklit, kus indeksid olid konfigureeritud |
| **`<INDEX> LUT`** | Üks nupp iga värvikaardistatud indeksi kohta | Pärast LUT-iga konfigureeritud jooksu |
| **`<Index> <Index\|LUT> <NNN>`** | Üks nupp iga [indeksi/LUT-i liivakasti](index-lut-sandbox.md) ekspordikäigu kohta | Hetkel, mil liivakasti eksport lõpeb |

### LATTICE’i taseme nupud

LATTICE’i kaadreid sisaldavad projektid lisavad need, mis on märgistatud taseme nimega, mitte toote nimega:

| Nupp | Tase |
| --- | --- |
| **RAW (originaal)** | Imporditud algne RAW-kaader |
| **RAW (kiirgus)** | Float32 spektraalne kiirgus, W/m²/sr/nm |
| **RAW (peegeldus)** | uint16 peegeldus, 32768 = ρ 1,0 |

`RAW (Original)` on kättesaadav juba importimise hetkest alates — seda ei ole vaja töödelda. Kui LATTICE-impordil puudub baasnupp täielikult (iga jäädvustuse baasfail on selle toorkaader), liigub võrk ise esimese kättesaadava taseme nupule, nii et tööriistariba esiletõstmine vastab sellele, mida te näete.

Kahe tasemega Chloros ekspordid ei saa **oma võrgustikunuppu**:

* **Debayered** — `RAW (Original)`-vaade renderdub juba debayered-režiimis, seega oleks teine nupp visuaalselt identse pildi peal liigne. `RAW (Debayered)` toode salvestatakse endiselt kettale ja on endiselt valitav täisekraani kihtide rippmenüüst.
* **Eelvaade** — RGB-kaameratel on eelvaade registreeritud `White Balanced`-kihina, millel on nupp olemas. Multispektraalsetel kaameratel on see registreeritud kui `RAW (Preview)` ja sellele pääseb juurde täisekraani kihtide rippmenüüst.

{% hint style="info" %}
Need taseme nupud kuvatakse ainult projektide puhul, mis tegelikult sisaldavad LATTICE-kaadreid. Survey3-projektid registreerivad mõned samad sisemised kihinimed ning nupud filtreeritakse neist välja, nii et Survey3-võrgustik säilitab oma tuttava `JPG / Targets / Reflectance`-komplekti.
{% endhint %}

Võrgustiku pisipildi klõpsamine avab täisekraanil [Pildivaataja](opening-an-image-full-screen.md) **samas tootes, mida võrgustik kuvab** — kui võrgustik on seatud väärtusele `Targets`, avab pisipilt eksporditud sihtpildi.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## LATTICE-projekti rühmitamine: käivitaja kaupa vs kaamera kaupa

Massiivipildistamine toodab sama hetke kohta mitu pilti erinevatest kaameramoodulitest. Rühmitamine määrab, kuidas võrgustik neid järjestab. Mõlemas režiimis kuvatakse täislaiusega kokkupandavad päisribad; **iga rühm on alguses lahti** ja Chloros mäletab need, mida sulged. Kokkupandmise seisundit jälgitakse eraldi iga režiimi puhul, seega rühma sulgemine režiimis „Iga kaamera kohta” ei sulge midagi režiimis „Iga päästiku kohta”.

### Iga kaamera kaupa (vaikimisi)

Üks rühm iga kaameramooduli kohta. Pealkirjas kuvatakse kaamera mudel ja seerianumber (`LATT-M3M — <serial>`) ning fotode arv. Rühma sees olevad pildid on järjestatud kronoloogiliselt pildistamisaja järgi.

Selles režiimis ilmub tööriistaribale ka üks **kaamerafiltri nupp iga kaamera kohta**, millel on märge `MODEL (SERIAL)`. Kõik kaamerad on algselt valitud; nupule klõpsates tühistatakse selle kaamera valik ja eemaldatakse selle rühm võrgustikust. See on kiire viis ühe laineala läbivaatamiseks kogu lennu ulatuses.

### Ühe päästiku kohta

Üks rühm ühe pildistamisjuhtumi kohta — kõik kaadrid, mille kõik moodulid pildistasid sama päästiku ajel. Pealkirjas on näidatud pildistamise aeg, pildistamisel osalenud kaamerate arv ning märge iga rühmas oleva kaameramudeli kohta. Rühma sees olevad pildikastid on järjestatud kaamera seerianumbri järgi, nii et sama lainepikkus asub iga käivituse puhul samas veerus.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Segaprojektis olevad mitte-LATTICE-pildid ei ole rühmitatud – need kuvatakse tavaliste pildikastidena rühmade järel.

***

## Võrgustiku pisipildid järgivad GSD-ploki suurust

Kui olete pildi vahekaardi külgribal määranud **GSD (px)** plokisuuruse, kuvatakse ruudustiku pisipildid samas maapinna eraldusvõimes – mitte ainult täisekraanivaates. Plokisuurus 8 tähendab, et iga kuvatav piksel on allikapikslite 8 × 8 ploki keskmine väärtus kõikjal rakenduses, kus pilti kuvatakse.

Kuna ruut on alguses vaid paar sada pikslit lai, ei tekita jämedad plokisuurused ruudustikus enam nähtavat erinevust juba ammu enne, kui see täisekraanivaates ilmneb: 160 pikslise ruutu sisse joonistatud 4000 pikslise raami puhul on juba umbes 25 allikapikslit ühe kuvatava pikseli kohta. Vaata [Pildi avamine täisekraanil](opening-an-image-full-screen.md#gsd-block-size) kontrollelemendi kohta.

***

## Seotud leheküljed

* [**Pildi avamine täisekraanil**](opening-an-image-full-screen.md) — täisekraanivaataja, kursori väärtused ja histogramm
* [**Pildikihtid**](image-layers.md) — kihtide rippmenüü täisekraanivaatajas
* [**Indeksi/LUT-liivakast**](index-lut-sandbox.md) — indeksvisualisatsioonide loomine ja eksportimine
* [**Projekti seaded**](../project-settings/project-settings.md) — ekspordivalikud, mis määravad, millised tooted üldse olemas on
