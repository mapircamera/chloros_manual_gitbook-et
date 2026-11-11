# Failide lisamine projekti

Kui olete Chloroses projekti loonud või avanud, on järgmine samm multispektraalsete piltide lisamine, et alustada töötlemist. Vahekaardi „Failibrauser”<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> abil on lihtne pilte importida ja andmekogumit hallata.

## Failibrauseri avamine

1. Avage või looge projekt Chloroses
2. Klõpsake vasakul külgribal ikooni **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
3. Paneelis „File Browser“ kuvatakse teie projekti failide loend

{% hint style="info" %}
**Toetatud failitüübid**:

* **Survey3W / Survey3N**: RAW+JPG-paarid ja JPG-pildid (soovitatav on RAW+JPG)
* **LATTICE**: `.tif` / `.tiff` salvestused — salvestatud Chlorosi kaamerajuhtimise või LATTICE-keskuse abil
* **Valgusanduri andmed**: `.daq` salvestused (DAQ-U/M/E) ja DAQ-M `.csv` allapoole suunatud logid — imporditud koos piltidega peegelduskalibreerimise läbiviimiseks
{% endhint %}

***

## Piltide lisamine projektile

Projektile saab pilte lisada kahel peamisel viisil:

### Meetod 1: Failide lisamine

Kasutage seda valikut üksikute pildifailide või väikese failivaliku importimiseks.

1. Klõpsake failibrauseri paneeli ülaosas asuvat nuppu **„Lisa faile”** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line">
2. Navigeerige kaustani, kus asuvad teie pildid
3. Valige üks või mitu pildifaili (hoidke all **Ctrl**-klahvi, et valida mitu faili)
4. Klõpsake **„Ava”**, et importida valitud failid

### Meetod 2: Kausta lisamine

Kasutage seda valikut, et importida korraga kõik kausta pildid. Ühes dialoogis saate valida **mitu kausta**.

1. Klõpsake failibrauseri paneeli ülaosas asuvat nuppu **„Lisa kaust”** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">
2. Navigeerige kaustani, mis sisaldab teie pildistamissessiooni pilte, ja valige see (või need)
3. Klõpsake nuppu **„Vali kaust”**, et importida kõik toetatud pildid

{% hint style="info" %}
**Teavitatakse failidest, mida ei õnnestunud laadida.** Kui kaust sisaldab faile, mida Chloros tunnistab, kuid mida ei suuda laadida, kuvatakse hoiatus – pildid ei kao võrgustikust märkamatult.
{% endhint %}

***

## LATTICE-salvestuskaustade importimine

LATTICE-salvestused salvestatakse **ühe alamkaustaga iga eksporditaseme kohta** — näiteks `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` – koos vastava `.daq` allalaadimisfailiga juurkataloogis:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Valige kaust „Add Folder” piltide juurkataloogis** (ülalpool olev `output/`). Kui valitud kaustas pole ise pilte, kuid seal on alamkaustad, liigub Chloros neisse automaatselt — nii alamkaustade tase kui ka juurkaust `.daq` võetakse ühekorraga sisse.**Kuidas pildid imporditakse:*** Iga pilt imporditakse **ühe pildina**, rühmitatuna pildi järgi (mitte üks kirje iga taseme kohta). Sama pildi teised tasemed kuvatakse selle ühe pildi vaatamisrežiimidena.
* **Töötlemine algab alati toorkaadrist.** Teised tasemed on vaadeldavad, kuid töötlemisprotsessi läbib ainult `raw` — juba töödeldud toote uuesti töötlemine rakendaks parandusi kahekordselt, mistõttu Chloros seda ei tee. Uuesti imporditud eksport ei saa kunagi hõivamise toorkaadri kohta.
* Hõivamiskausta, mis on salvestatud **ilma** toorkaadrite importimiseta, kuvatakse normaalselt, kuid töötlemine jätab selle vahele ja märgib seda logisse. (CLI-lipu `--input-level` abil saab sellisel juhul sisenemispunkti sundida — vt [CLI-viidet](../reference/cli-reference.md#what-a-captures-folder-looks-like).)
* Alates versioonist 1.3.0 võib pildistamiskausta sisaldada ka `clamp_masks/`-kausta (pikslihaaval maske kiirguse ja peegeldusastmete jaoks — vt [väljundpildi formaadid](../output-image-formats.md#clamp-masks)). Maskid ei ole stseeni pildid ning Chloros jätab need importimisel kõrvale, olenemata nende asukohast.

**LATTICE hubi seansid** imporditakse samamoodi: valige „Add Folder“ (Lisa kaust) ja osutage hubist kopeeritud seansikaustale (see sisaldab `raw/` ja `previews/`), koos mis tahes DAQ-M `.csv` allapoole suunatud logiga. Kui kaamera või DAQ-seadme kalibreerimisandmed pole teie arvutisse veel salvestatud, laadib Chloros need importimisel automaatselt seerianumbri järgi alla (nõuab ühekordset internetiühendust).***

## Failibrauseri tabeli mõistmine

Kui pildid on imporditud, kuvatakse need tabelis, millel on järgmised veerud:

### Failinimi

* Kaamera originaalne failinimi
* Säilitab kaamera nimetamiskonventsiooni (nt IMG\_0001.RAW või capture\_20260816\_101500\_SN213800234\_raw.tif)

### Ajamärge

* Pildi salvestamise kuupäev ja kellaaeg
* Saadud pildi EXIF-metadata põhjal
* Kasutatakse valgusandurite sobitamiseks, PPK-sünkroniseerimiseks ja kalibreerimise sihtmärkide ajastamiseks

### Kaamera mudel

* Automaatselt tuvastatud kaamera ja filtri konfiguratsioon
* Survey3i näited: Survey3W_RGN, Survey3N_OCN, Survey3W_RGB
* LATTICE'i näited: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Kasutatakse õigete töötlusprofiilide rakendamiseks

### Sihtmärgi veerg (valikukast)

* Märkige see kast piltide puhul, mis sisaldavad kalibreerimissihte
* Kui vähemalt üks pilt on märgitud, **otsitakse sihtmärke ainult märgitud piltidest*** Täpsemad üksikasjad leiate jaotisest [Sihtmärgipiltide valimine](choosing-target-images.md)

### Pildi metaandmete vaatamine

Tabelist ülemises paremas nurgas asuva lülitusnupu klõpsamine kuvab valitud pildi metaandmed pildirestis.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Teie projekti valgusandurifailid

* Failid `.daq` ja `.csv` kuvatakse failibrauseri nimekirjas, kuid neid ei saa klõpsata — need annavad peegelduskalibreerimiseks allapoole suunatud kiirgustiheduse.
* Iga imporditud `.daq`/`.csv` on loetletud menüüs **Projekti seaded → DAQ valgusandur**, kus saate vaadata iga faili puhul kehtivat hajutikorki korrigeerimist. Vaata [Projekti seadete kohandamine](adjusting-project-settings.md).
* **Valgusandurid**-vahekaardil tehtud salvestused lisatakse avatud projektile automaatselt – käsitsi importimist ei ole vaja.***

## Failide haldamine projektis

### Failide eemaldamine

Soovimatute piltide eemaldamiseks projektist:

1. Valige failibrauseri tabelis üks või mitu pilti
2. Klõpsake nuppu **„Eemalda valitud”** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Kinnitage eemaldamine (faile ei kustutata kettalt, vaid eemaldatakse ainult projektist)

### Sorteerimine ja filtreerimine

* **Sorteerimine veeru järgi**: klõpsake mis tahes veeru pealkirjal, et pilte sorteerida
* **Sorteerimine ajamärgise järgi**: kasulik kronoloogiliste pildiseeriate korrastamiseks
* **Kaameramudeli filter**: rühmitage pildid kaameratüübi järgi, kui kasutate mitut kaamerat***

## Pildi eelvaade

### Täispildi vaatamine

Klõpsake failibrauseris mis tahes pildi pisipildil, et kuvada see peamises eelvaatealas:

1. Pilt ilmub keskmisele eelvaatepaneelile
2. Kasutage suumikontrolle pildi detailide uurimiseks
3. Liiguge piltide vahel nooleklahvide abil

### Kiirnavigatsioon

* **Eelmine pilt**: klõpsa vasakule nooleklahvile või vajuta klahvi ←
* **Järgmine pilt**: klõpsa paremale nooleklahvile või vajuta klahvi →
* **Suurenda/vähenda**: kasuta hiirerattat või suumimisnuppe
* **Liikumine**: klõpsa ja lohista pilti, kui see on suurendatud***

## Duplikaatfailide käitlemine

Chloros tuvastab ja ignoreerib automaatselt duplikaatfaile:

* Failid, millel on identsed failinimed, jäetakse vahele
* Vältib juhuslikku topeltkäsitlemist
* Duplikaatide tuvastamisel kuvatakse hoiatussõnum

{% hint style="warning" %}
**Oluline**: Ära nimeta ümber ega muuda originaalpildifaile enne importimist. Chloros tugineb õigeks töötlemiseks originaalsetele failinimedele ja metaandmetele.
{% endhint %}

***

## Erinevate kaamerate andmekogumid

Kui teie projekt sisaldab pilte mitmest MAPIR-kaamerast:

1. Chloros tuvastab automaatselt iga kaameramudeli — Survey3, LATTICE või nende kombinatsiooni
2. Iga kaameratüüpi töödeldakse vastava kalibreerimisprofiiliga
3. Failibrauser kuvab kaameramudeli veerus „Camera Model“
4. Iga kaamera saab töötlemisel oma väljundkaustade puu

**Näiteid**: Survey3W RGN + Survey3N OCN kahekaameraline seadistus või LATTICE-massiiv, millel on RGB-põhikaamera ja mitu kitsasriba moodulit***

## Parimad tavad

### Korraldage andmed enne importimist

* Hoidke kalibreerimise sihtmärkpilte samas kaustas kui uuringupilte
* Hoidke iga pildistamissessiooni `.daq` / `.csv` valgusandurifailid koos selle sessiooni piltidega
* Säilitage kaamera/SD-kaardi/jaoturi algne kaustastruktuur
* Ära sega ühes projektis eri pildistamissessioonide andmekogumeid

### Failide nimetamine

* Säilita kaamera algsed failinimed (IMG\_0001.RAW, capture\_..., jne.)
* Ära nimeta faile ümber enne importimist
* Algne nimi sisaldab olulisi metaandmeid

### Kalibreerimise sihtmärkide pildid

* Lisage alati 1–2 kalibreerimise sihtmärgi pilti iga seansi kohta (Survey3; LATTICE puhul võib seda asendada DAQ-salvestus – vt [Sihtmärkide piltide valimine](choosing-target-images.md))
* Pildista sihtmärke enne ja pärast pildistamissessiooni
* Aseta sihtmärgid samadesse valgustingimustesse kui pildistamisala
* Märgi sihtmärgipildid valikukastiga „Target“

***

## Tüüpilised probleemid ja lahendused

### Pildid ei kuvata pärast importimist

**Võimalikud põhjused:**

* Failivormingut ei toetata (vaata toetatud failitüüpide loendit selle lehe ülaosas)
* Pildid on pärit muudest kui MAPIR-kaameratest (vaata [Toetatud kaamerad](../supported-cameras.md))
* Fail on rikutud või SD-kaardilt ei ole seda täielikult üle kantud

**Lahendus**: Kontrollige failivormingu ja kaameramudeli ühilduvust ning vaadake failide laadimise hoiatust, et teada saada, millised konkreetsed failid ei õnnestunud

### Kaameramudelit ei tuvastatud

**Võimalikud põhjused:**

* Muudetud EXIF-metadata
* Pildid on redigeeritud välises tarkvaras
* Ebatäielik failiedastus

**Lahendus**: Impordi uuesti originaalsed, muutmata failid kaamerast/SD-kaardilt

### Puuduvad ajamärgised

**Võimalikud põhjused:**

* Kaamera kell pole õigesti seadistatud
* EXIF-andmed on välise tarkvara poolt eemaldatud

**Lahendus**: Veenduge, et kaamera aja seaded olid pildistamise ajal õiged

### Uuesti avatud projekt näitab puuduvaid faile

Kui allikafaile on projekti viimase avamise järel teisaldatud või kustutatud, näitab Chloros teile, **millised**failid on kadunud, selle asemel et avada tühi raster. Kui projekti kaust ise on teisaldatud, suunab Chloros allikateed ümber kausta `project.json` uude asukohta ja säilitab varukoopia (`project.json.before-relink-<date>-<time>`). Kui faile ei leita, pakub aken**Allikafaile ei leitud**võimalust**Sirvida** uue asukoha leidmiseks, seejärel valida „Uus projekt” või „Kustuta projekt”. CLI-st kasutage `chloros-cli project open PATH --relink FOLDER`; SDK-st `open_project(path, relink=...)`.***

## Järgmised sammud

Kui failid on imporditud:

1. **Vaata failide nimekirja üle** – veendu, et kõik pildid on õigesti laaditud
2. **Kontrolli kaameramudeleid** – veendu, et kaamerad on õigesti tuvastatud
3. **Märgi sihtpildid** – vaata [Sihtpiltide valimine](choosing-target-images.md)
4. **Kohanda seadeid** – konfigureeri töötlemisvalikud [Projekti seaded](adjusting-project-settings.md)
5. **Alustage töötlemist** – vt [Töötlemise alustamine](starting-the-processing.md)

Üksikasjalikku teavet projekti konfigureerimise kohta leiate jaotisest [Projekti seadete kohandamine](adjusting-project-settings.md).
