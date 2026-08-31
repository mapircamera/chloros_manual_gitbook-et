# Failide lisamine projekti

Kui olete Chloros-is projekti loonud või avanud, on järgmine samm multispektraalsete piltide lisamine, et alustada nende töötlemist. Vahekaardi „File Browser“ (<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">) abil on lihtne pilte importida ja andmekogumit hallata.

## Failibrauseri avamine

1. Avage või looge projekt Chloros-is
2. Klõpsake vasakul külgribal ikooni **Failibrauser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
3. Failibrauseri paneelis kuvatakse teie projekti failide loend

{% hint style="info" %}
**Toetatud failitüübid**:

* **Survey3W / Survey3N**: RAW+JPG-paarid ja JPG-pildid (soovitatav on RAW+JPG)
* **LATTICE**: `.tif` / `.tiff` salvestised — salvestatud Chloros kaamera juhtimise või LATTICE-keskuse abil
* **Valgusanduri andmed**: `.daq` salvestised (DAQ-U/M/E) ja DAQ-M `.csv` allapoole suunatud logid — imporditud koos piltidega peegeldusvõime kalibreerimiseks
{% endhint %}

***

## Piltide lisamine projektile

Projektile saab pilte lisada kahel peamisel viisil:

### Meetod 1: failide lisamine

Kasutage seda valikut üksikute pildifailide või väikese failivaliku importimiseks.

1. Klõpsake failibrauseri paneeli ülaosas asuvat nuppu **„Lisa faile”** (<img src="../.gitbook/assets/image (3).png" alt="" data-size="line">)
2. Navigeerige kaustani, kus asuvad teie pildid
3. Valige üks või mitu pildifaili (hoidke all klahvi **Ctrl**, et valida mitu faili)
4. Klõpsake nuppu **„Ava”**, et importida valitud failid

### Meetod 2: Kausta lisamine

Kasutage seda valikut, et importida korraga kõik kaustas olevad pildid. Ühes dialoogis saate valida **mitu kausta**.

1. Klõpsake failibrauseri paneeli ülaosas asuvat nuppu **„Lisa kaust”** (<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">)
2. Leidke ja valige kaust(ad), mis sisaldab (sisaldavad) teie pildistamissessiooni pilte
3. Klõpsake nuppu **„Vali kaust”**, et importida kõik toetatud pildid

{% hint style="info" %}
**Teavitatakse failidest, mida ei õnnestunud laadida.** Kui kaust sisaldab faile, mida Chloros tunneb ära, kuid ei suuda laadida, annab hoiatus sellest teada – pildid ei kao võrgustikust märkamatult.
{% endhint %}

***

## LATTICE-pildistamiskausta importimine

LATTICE-pildistamised salvestatakse **ühe alamkaustaga iga eksporditaseme kohta** – näiteks `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` – koos vastava `.daq` allalaadimisfailiga juurkataloogis:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Valige „Lisa kaust” ja osutage piltide juurkataloogile** (ülalolev `output/`). Kui valitud kaustas pole ise pilte, kuid seal on alamkaustad, liigub Chloros nendesse automaatselt — nii taseme alamkaustad kui ka juurkaust `.daq` võetakse korraga kaasa.**Kuidas pildid imporditakse:*** Iga pilt imporditakse **ühe pildina**, rühmitatuna pildi järgi (mitte üks kirje iga taseme kohta). Sama pildi teised tasemed kuvatakse selle ühe pildi vaatamisrežiimidena.
* **Töötlemine algab alati töötlemata kaadrist.** Teisi tasemeid on võimalik vaadata, kuid töötlemisprotsessi läbib ainult `raw` – juba töödeldud toote uuesti töötlemine rakendaks parandusi kahekordselt, mistõttu Chloros lükatakse tagasi. Uuesti imporditud eksport ei saa kunagi hõivamise toorkaadri pesa hõivata.
* Hõivamiskausta, mis on salvestatud **ilma** toorkaadrite impordita, kuvatakse normaalselt, kuid töötlemine jätab selle vahele ja märgib seda logisse. (Lipik CLI `--input-level` võib sellisel juhul sissepääsupunkti sundida – vt [CLI-viidet](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**LATTICE-keskuse seansid** imporditakse samamoodi: suunake „Add Folder” (Lisa kaust) hubist kopeeritud seansikaustale (see sisaldab `raw/` ja `previews/`), koos mis tahes DAQ-M `.csv` allavoolu logiga. Kui kaamera või DAQ-seadme kalibreerimisandmed pole teie arvutisse veel salvestatud, laadib Chloros need importimisel automaatselt seerianumbri järgi alla (nõuab ühekordset internetiühendust).***

## Failibrauseri tabeli mõistmine

Kui pildid on imporditud, kuvatakse need tabelis, millel on järgmised veerud:

### Failinimi

* Kaamera originaalne failinimi
* Säilitab kaamera nimetamiskonventsiooni (nt IMG\_0001.RAW või capture\_20260816\_101500\_SN213800234\_raw.tif)

### Ajamärge

* Pildi salvestamise kuupäev ja kellaaeg
* Saadud pildi EXIF-metandmetest
* Kasutatakse valgusanduri sobitamiseks, PPK-sünkroniseerimiseks ja kalibreerimise sihtmärkide ajastamiseks

### Kaamera mudel

* Automaatselt tuvastatud kaamera ja filtri konfiguratsioon
* Survey3 näited: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* LATTICE näited: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Kasutatakse õigete töötlusprofiilide rakendamiseks

### Sihtmärkide veerg (valikukast)

* Märkige see valikukast piltide puhul, mis sisaldavad kalibreerimise sihtmärke
* Kui vähemalt üks pilt on märgitud, **otsitakse sihtmärke ainult märgitud piltidest*** Täpsema teabe saamiseks vaadake [Sihtmärkide valimine](choosing-target-images.md)

### Pildi metaandmete vaatamine

Tabelist ülemises paremas nurgas asuva lülitusnupu klõpsamine kuvab valitud pildi metaandmed pildivõrgustikus.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Teie projekti valgusandurifailid

* Failid `.daq` ja `.csv` kuvatakse failibrauseri loendis, kuid neid ei saa klõpsata — need sisaldavad peegelduskalibreerimiseks vajalikku allapoole suunatud kiirgustihedust.
* Iga imporditud `.daq`/`.csv` on loetletud menüüs **Projekti seaded → DAQ valgusandur**, kus saate vaadata iga faili puhul kehtivat hajutikorki korrigeerimist. Vaata [Projekti seadete kohandamine](adjusting-project-settings.md).
* **Valgusandurid**-vahekaardil tehtud salvestused lisatakse avatud projektile automaatselt – käsitsi importimist ei ole vaja.***

## Failide haldamine projektis

### Failide eemaldamine

Soovimatute piltide eemaldamiseks projektist:

1. Valige failibrauseri tabelis üks või mitu pilti
2. Klõpsake nuppu **„Eemalda valitud”** (<img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">)
3. Kinnitage eemaldamine (faile ei kustutata kettalt, vaid need eemaldatakse ainult projektist)

### Sorteerimine ja filtreerimine

* **Sorteerimine veeru järgi**: klõpsake mis tahes veeru pealkirjal, et pilte sorteerida
* **Sorteerimine ajamärgise järgi**: kasulik kronoloogiliste pildiseeriate korraldamiseks
* **Kaameramudeli filter**: rühmitage pildid kaameratüübi järgi, kui kasutate mitut kaamerat***

## Pildi eelvaade

### Täispildi vaatamine

Klõpsake failibrauseris mis tahes pildi pisipildil, et kuvada see peamises eelvaatealas:

1. Pilt ilmub keskmisele eelvaatepaneelile
2. Kasutage suumikontrolle pildi detailide uurimiseks
3. Liiguge piltide vahel nooleklahvide abil

### Kiirnavigatsioon

* **Eelmine pilt**: klõpsake vasakule nooleklahvile või vajutage klahvi ←
* **Järgmine pilt**: klõpsake paremale nooleklahvile või vajutage klahvi →
* **Suurendamine/vähendamine**: kasutage hiirerattat või suumimisnuppe
* **Panoraam**: suurendatud pildil klõpsake ja lohistage***

## Duplikaatfailide käitlemine

Chloros tuvastab ja ignoreerib automaatselt duplikaatfailid:

* Failid, mille nimed on identsed, jäetakse vahele
* Takistab juhuslikku kahekordset töötlemist
* Duplikaatide tuvastamisel kuvatakse hoiatussõnum

{% hint style="warning" %}
**Oluline**: Ära nimeta ümber ega muuda originaalpildifaile enne importimist. Chloros tugineb õigeks töötlemiseks originaalsetele failinimedele ja metaandmetele.
{% endhint %}

***

## Erinevate kaamerate andmekogumid

Kui teie projekt sisaldab pilte mitmest MAPIR kaamerast:

1. Chloros tuvastab automaatselt iga kaameramudeli — Survey3, LATTICE või nende segu
2. Iga kaameratüüpi töödeldakse vastava kalibreerimisprofiiliga
3. Failibrauser kuvab kaameramudeli veerus „Camera Model“
4. Iga kaamera saab töötlemisel oma väljundkaustade puu

**Näiteolukorrad**: Survey3W RGN + Survey3N OCN kahe kaameraga seadistus, või LATTICE-massiiv, millel on RGB-peakaamera ja mitu kitsasribalist moodulit***

## Parimad tavad

### Korraldage andmed enne importimist

* Hoidke kalibreerimise sihtmärkide pilte samas kaustas kui uuringupilte
* Hoidke iga pildistamissessiooni `.daq` / `.csv` valgusanduri faile koos selle sessiooni piltidega
* Säilitage kaamera/SD-kaardi/jaoturi algne kaustastruktuur
* Ära sega ühes projektis eri seansside andmekogumeid

### Failide nimetamine

* Säilita kaamera algsed failinimed (IMG\_0001.RAW, capture\_... jne)
* Ära nimeta faile ümber enne importimist
* Algne nimi sisaldab olulisi metaandmeid

### Kalibreerimise sihtmärkide pildid

* Lisage alati 1–2 kalibreerimise sihtmärgi pilti iga seansi kohta (Survey3; LATTICE puhul võib selle asendada DAQ-salvestus – vt [Sihtmärkide piltide valimine](choosing-target-images.md))
* Pildistage kalibreerimise sihtmärke enne ja pärast pildistamissessiooni
* Paigutage sihtmärgid samadesse valgustingimustesse kui pildistamisala
* Märgistage sihtmärgipildid, kasutades valikukasti „Target“

***

## Tüüpilised probleemid ja lahendused

### Pildid ei ilmu pärast importimist

**Võimalikud põhjused:**

* Failivormingut ei toetata (vaata toetatud vormingute nimekirja käesoleva lehe ülaosas)
* Pildid on pärit muudest kui MAPIR kaameratest (vaata [Toetatud kaamerad](../supported-cameras.md))
* Fail on rikutud või SD-kaardilt ei ole seda täielikult üle kantud

**Lahendus**: Kontrollige failivormingu ja kaameramudeli ühilduvust ning vaadake failide laadimise hoiatust, et teada saada, millised täpselt failid ei õnnestunud laadida

### Kaameramudelit ei tuvastatud

**Võimalikud põhjused:**

* Muudetud EXIF-metadata
* Pildid on redigeeritud välises tarkvaras
* Ebatäielik failide ülekandmine

**Lahendus**: Impordi uuesti originaalsed, muutmata failid kaamerast/SD-kaardilt

### Puuduvad ajamärgid

**Võimalikud põhjused:**

* Kaamera kell pole õigesti seadistatud
* EXIF-andmed on välise tarkvara poolt eemaldatud

**Lahendus**: Kontrollige, kas kaamera aja seaded olid pildistamise ajal õiged

### Uuesti avatud projekt näitab puuduvaid faile

Kui allikafaile on pärast projekti viimast avamist teisaldatud või kustutatud, näitab Chloros teile, **millised** failid on kadunud, selle asemel et avada tühi võrgustik. Taasta failid nende algsetesse asukohtadesse või eemalda puuduvad kanded ja impordi need uuesti.***

## Järgmised sammud

Kui failid on imporditud:

1. **Vaata failide nimekirja üle** – veendu, et kõik pildid on õigesti laaditud
2. **Kontrolli kaameramudeleid** – veendu, et kaamerad on õigesti tuvastatud
3. **Märkige sihtpildid** – vt [Sihtpiltide valimine](choosing-target-images.md)
4. **Kohandage seadeid** – konfigureerige töötlemisvalikud [Projekti seaded](adjusting-project-settings.md)
5. **Alusta töötlemist** – vaata [Töötlemise alustamine](starting-the-processing.md)

Projekti seadistamise kohta leiate üksikasjalikku teavet jaotisest [Projekti seadete kohandamine](adjusting-project-settings.md).
