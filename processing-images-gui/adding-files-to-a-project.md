# Failide lisamine projektile

Kui olete Chloros-is projekti loonud või avanud, on järgmine samm multispektraalsete piltide lisamine töötlemise alustamiseks. Failibrauser<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> teeb piltide importimise ja andmekogumi haldamise lihtsaks.

## Failibrauseri avamine

1. Avage või looge projekt Chloros-is
2. Klõpsake **Failibrauser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> ikoonile vasakul külgribal
3. Failibrauseri paneel kuvab teie projekti failide loendi

{% vihje style=&quot;info&quot; %}
**Toetatud failitüübid**: Chloros toetab RAW+JPG ja JPG pildifaile kaameratest MAPIR Survey3W ja Survey3N. Soovitatav on kasutada ainult RAW+JPG faile.
{% endhint %}

***

## Piltide lisamine projekti

Piltide lisamiseks projekti on kaks peamist viisi:

### Meetod 1: failide lisamine

Kasutage seda valikut üksikute pildifailide või väikese failivaliku importimiseks.

1. Klõpsake **„Lisa faile”** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> nupule failibrauseri paneeli ülaosas
2. Navigeerige kaustani, mis sisaldab teie pilte
3. Valige üks või mitu pildifaili (hoidke all **Ctrl**, et valida mitu faili)
4. Klõpsake **&quot;Ava&quot;**, et importida valitud failid

### Meetod 2: kausta lisamine

Kasutage seda valikut, et importida kõik pildid kaustast korraga.

1. Klõpsake nuppu **„Lisa kaust”** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> failibrauseri paneeli ülaosas.
2. Leidke ja valige kaust, mis sisaldab teie pildistamissessiooni pilte.
3. Klõpsake **„Vali kaust”**, et importida kõik selle kausta toetatud pildid.***

## Failibrauseri tabeli mõistmine

Pärast piltide importimist kuvatakse need tabelis, millel on järgmised veerud:

### Failinimi

* Kaamera originaalne failinimi
* Säilitab kaamera nimetamiskonventsiooni (nt IMG\_0001.RAW)

### Aeg

* Pildi salvestamise kuupäev ja kellaaeg
* Väljavõte pildi EXIF-metadatast
* Kasutatakse PPK sünkroniseerimiseks ja kalibreerimise sihtmärgi tuvastamiseks

### Kaamera mudel

* Automaatselt tuvastatud kaamera ja filtri konfiguratsioon
* Näited: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Kasutatakse õigete töötlusprofiilide rakendamiseks

### Sihtmärgi veerg (valikukast)

* Märkige see kast piltide puhul, mis sisaldavad kalibreerimise sihtmärke
* Kiirendab oluliselt sihtmärgi tuvastamist töötlemise ajal
* Täpsemad andmed leiate jaotisest [Sihtmärgi piltide valimine](choosing-target-images.md)

***

## Projektis olevate failide haldamine

### Failide eemaldamine

Soovimatute piltide eemaldamiseks projektist:

1. Valige failibrauseri tabelis üks või mitu pilti
2. Klõpsake nuppu **„Eemalda valitud”** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Kinnitage eemaldamine (faile ei kustutata kettalt, vaid eemaldatakse ainult projektist).

### Sorteerimine ja filtreerimine

* **Sorteeri veeru järgi**: klõpsake mis tahes veeru pealkirjal, et pilte sorteerida.
* **Sorteeri ajamärgi järgi**: kasulik kronoloogiliste pildistamisseeriate korraldamiseks.
* **Kaameramudeli filter**: rühmitage pildid kaameratüübi järgi, kui kasutate mitut kaamerat.***

## Pildi eelvaade

### Täispildi vaatamine

Klõpsake failibrauseris mis tahes pildi pisipildil, et kuvada see peamises eelvaatealas:

1. Pilt kuvatakse keskmises eelvaatepaneelis.
2. Kasutage suumikontrolle, et vaadata pildi detaile.
3. Liiguge piltide vahel nooleklahvide abil.

### Kiire navigeerimine

* **Eelmine pilt**: klõpsake vasakule nooleklahvile või vajutage ← klahvile
* **Järgmine pilt**: klõpsake paremale nooleklahvile või vajutage → klahvile
* **Suurendamine/vähendamine**: kasutage hiirerattat või suumiklahve
* **Panorameerimine**: suumimisel klõpsake ja lohistage pilti***

## Duplikaatfailide käitlemine

Chloros tuvastab ja ignoreerib automaatselt dubleeritud faile:

* Failid, millel on identne failinimi, jäetakse vahele.
* Vältib juhuslikku topeltkäsitlemist.
* Dubleeritud failide tuvastamisel kuvatakse hoiatus.

{% hint style=&quot;warning&quot; %}
**Oluline**: Ärge nimetage ümber ega muudake originaalseid pildifaile enne importimist. Chloros tugineb õigeks töötlemiseks originaalsetele failinimedele ja metaandmetele.
{% endhint %}

***

## Segatud kaamerate andmekogud

Kui teie projekt sisaldab pilte mitmest MAPIR kaamerast:

1. Chloros tuvastab automaatselt iga kaameramudeli.
2. Iga kaameratüüp töödeldakse vastava kalibreerimisprofiiliga.
3. Failibrauser kuvab kaameramudeli veerus „Kaameramudel”.
4. Töötlemisel rakendatakse iga kaameratüübi jaoks õigeid seadeid.

**Näide**: Survey3W RGN + Survey3N OCN kahe kaameraga seadistus.***

## Parimad tavad

### Korraldage enne importimist

* Hoidke kalibreerimise sihtpildid samas kaustas kui uuringupildid
* Säilitage kaamera/SD-kaardi algne kausta struktuur
* Ärge segage ühes projektis eri sessioonide andmekogusid

### Failide nimetamine

* Säilitage kaamera algsed failinimed (IMG\_0001.RAW jne)
* Ärge nimetage faile enne importimist ümber
* Algne nimi sisaldab olulisi metaandmeid

### Kalibreerimise sihtpildid

* Lisage alati 1–2 kalibreerimise sihtpilti seansi kohta.
* Pildistage sihtmärgid enne ja pärast pildistamise seanssi.
* Paigutage sihtmärgid samadesse valgustingimustesse kui pildistamispiirkond.
* Märgistage sihtpildid sihtmärgi valikukastiga, et kiirendada töötlemist.

***

## Tavapärased probleemid ja lahendused

### Pildid ei kuvata pärast importimist

**Võimalikud põhjused:**

* Failivormingut ei toetata (ainult RAW+JPG ja JPG MAPIR kaameratest)
* Pildid on pärit muudest kui MAPIR kaameratest (vt [Toetatud kaamerad](../supported-cameras.md))
* Fail on rikutud või SD-kaardilt ülekandmine on poolik

**Lahendus**: Kontrollige failivormingu ja kaameramudeli ühilduvust.

### Kaameramudelit ei tuvastatud

**Võimalikud põhjused:**

* Muudetud EXIF-metadata
* Välises tarkvaras redigeeritud pildid
* Ebatäielik failide ülekandmine

**Lahendus**: Importige uuesti originaalsed, muutmata failid kaamerast/SD-kaardilt.

### Puuduvad ajamärgised

**Võimalikud põhjused:**

* Kaamera kell ei ole õigesti seadistatud
* EXIF-andmed on eemaldatud välise tarkvaraga

**Lahendus**: Kontrollige, et kaamera aja seaded olid pildistamise ajal õiged***

## Järgmised sammud

Kui failid on imporditud:

1. **Vaadake failide loendit** – veenduge, et kõik pildid on õigesti laaditud
2. **Kontrollige kaameramudeleid** – veenduge, et kaamera on õigesti tuvastatud
3. **Märkige sihtpildid** – vt [Sihtpiltide valimine](choosing-target-images.md)
4. **Kohandage seadeid** – konfigureerige töötlemisvalikud [Projekti seaded](adjusting-project-settings.md)
5. **Alustage töötlemist** – vt [Töötlemise alustamine](starting-the-processing.md)

Täpsemat teavet projekti konfigureerimise kohta leiate jaotisest [Projekti seaded](adjusting-project-settings.md).
