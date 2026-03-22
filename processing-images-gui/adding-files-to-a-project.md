# Failide lisamine projekti

Kui olete Chloros-is projekti loonud või avanud, on järgmine samm multispektraalsete piltide lisamine, et alustada töötlemist. Failibrauseri<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> võimaldab hõlpsasti pilte importida ja andmekogumit hallata.

## Failibrauseri avamine

1. Avage või looge projekt Chloros-is
2. Klõpsake vasakul külgribal **Failibrauser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> ikoonile vasakul külgribal
3. Failibrauseri paneelil kuvatakse teie projekti failide loend

{% hint style="info" %}
**Toetatud failitüübid**: Chloros toetab RAW+JPG ja JPG pildifaile kaameratest MAPIR, Survey3W ja Survey3N. Soovitatav on kasutada ainult RAW+JPG faile.
{% endhint %}

***

## Piltide lisamine projekti

Projektile pilte lisada saab peamiselt kahel viisil:

### Meetod 1: Failide lisamine

Kasutage seda valikut üksikute pildifailide või väikese failivaliku importimiseks.

1. Klõpsake nuppu **„Lisa faile”** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> nuppu failibrauseri paneeli ülaosas
2. Navigeerige kaustani, mis sisaldab teie pilte
3. Valige üks või mitu pildifaili (hoidke all **Ctrl**, et valida mitu faili)
4. Klõpsake **&quot;Ava&quot;**, et importida valitud failid

### Meetod 2: Kausta lisamine

Kasutage seda valikut, et importida kõik pildid kaustast korraga.

1. Klõpsake **&quot;Lisa kaust&quot;** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> nuppu failibrauseri paneeli ülaosas
2. Navigeerige kaustani, mis sisaldab teie pildistamissessiooni pilte, ja valige see
3. Klõpsake **&quot;Vali kaust&quot;**, et importida kõik toetatud pildid sellest kaustast***

## Failibrauseri tabeli mõistmine

Kui pildid on imporditud, kuvatakse need tabelis järgmiste veergudega:

### Failinimi

* Kaamera originaalne failinimi
* Säilitab kaamera nimetamiskonventsiooni (nt IMG\_0001.RAW)

### Aeg

* Pildi salvestamise kuupäev ja kellaaeg
* Saadud pildi EXIF-metadata
* Kasutatakse PPK sünkroniseerimiseks ja kalibreerimise sihtmärgi tuvastamiseks

### Kaamera mudel

* Automaatselt tuvastatud kaamera ja filtri konfiguratsioon
* Näited: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Kasutatakse õigete töötlusprofiilide rakendamiseks

### Sihtmärgi veerg (valikukast)

* Märkige see ruut piltide puhul, mis sisaldavad kalibreerimise sihtmärke
* Kiirendab oluliselt sihtmärkide tuvastamist töötlemise ajal
* Täpsemad üksikasjad leiate jaotisest [Sihtmärkide piltide valimine](choosing-target-images.md)

### Pildi metaandmete vaatamine

Tabelist ülemises paremas nurgas asuva lüliti nupule klõpsates kuvatakse valitud pildi metaandmed pildirestis.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Failide haldamine projektis

### Failide eemaldamine

Soovimatute piltide eemaldamiseks projektist:

1. Valige failibrauseri tabelis üks või mitu pilti
2. Klõpsake nuppu **&quot;Eemalda valitud&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> nupule
3. Kinnitage eemaldamine (faile ei kustutata kettalt, vaid eemaldatakse ainult projektist)

### Sorteerimine ja filtreerimine

* **Sorteeri veeru järgi**: klõpsake mis tahes veeru pealkirjal, et pilte sorteerida
* **Sorteeri ajamärgise järgi**: kasulik kronoloogiliste pildiseeriate organiseerimiseks
* **Kaameramudeli filter**: rühmitage pildid kaameratüübi järgi, kui kasutate mitut kaamerat***

## Pildi eelvaade

### Täispildi vaatamine

Klõpsake failibrauseris mis tahes pildi pisipildil, et kuvada see peamises eelvaatealas:

1. Pilt ilmub keskmisele eelvaatepaneelile
2. Kasutage suumikontrolle pildi detailide uurimiseks
3. Liiguge piltide vahel nooleklahvide abil

### Kiirnavigatsioon

* **Eelmine pilt**: Klõpsake vasakule noolele või vajutage klahvi ←
* **Järgmine pilt**: klõpsake paremat noolt või vajutage klahvi →
* **Suurendamine/vähendamine**: kasutage hiirerattat või suumimisnuppe
* **Panoraam**: klõpsake ja lohistage pilti, kui see on suurendatud***

## Duplikaatfailide käitlemine

Chloros tuvastab ja ignoreerib automaatselt duplikaatfailid:

* Failid, millel on identsed failinimed, jäetakse vahele
* Vältib juhuslikku topeltkäsitlemist
* Duplikaatide tuvastamisel kuvatakse hoiatussõnum

{% hint style="warning" %}
**Oluline**: Ärge nimetage ümber ega muudake originaalpildifaile enne importimist. Chloros tugineb nõuetekohaseks töötlemiseks originaalfailinimedele ja metaandmetele.
{% endhint %}

***

## Segatud kaamerate andmekogumid

Kui teie projekt sisaldab pilte mitmest MAPIR kaamerast:

1. Chloros tuvastab automaatselt iga kaameramudeli
2. Iga kaameratüüpi töödeldakse vastava kalibreerimisprofiiliga
3. Failibrauser kuvab kaameramudeli veerus „Camera Model“
4. Töötlemisel rakendatakse iga kaameratüübi jaoks õigeid seadeid

**Näide**: Survey3W RGN + Survey3N OCN kahe kaameraga seadistus***

## Parimad tavad

### Korraldage enne importimist

* Hoidke kalibreerimise sihtmärkide pilte samas kaustas kui uuringupilte
* Säilitage kaamera/SD-kaardi algne kaustastruktuur
* Ärge segage ühes projektis eri seansside andmekogumeid

### Failide nimetamine

* Säilitage kaamera algsed failinimed (IMG\_0001.RAW jne)
* Ärge nimetage faile ümber enne importimist
* Algne nimi sisaldab olulisi metaandmeid

### Kalibreerimise sihtmärgi pildid

* Lisage alati 1–2 kalibreerimise sihtmärgi pilti seansi kohta
* Pildistage sihtmärke enne ja pärast pildistamisseanssi
* Paigutage sihtmärgid samadesse valgustingimustesse kui pildistamisala
* Märgistage sihtmärgi pildid valikukastiga „Target”, et kiirendada töötlemist

***

## Tüüpilised probleemid ja lahendused

### Pildid ei ilmu pärast importimist

**Võimalikud põhjused:**

* Failivormingut ei toetata (ainult RAW+JPG ja JPG MAPIR kaameratelt)
* Pildid on pärit muudest kui MAPIR kaameratest (vaata [Toetatud kaamerad](../supported-cameras.md))
* Fail on rikutud või SD-kaardilt ei ole kogu sisu üle kantud

**Lahendus**: Kontrollige failivormingu ja kaameramudeli ühilduvust

### Kaameramudelit ei tuvastata

**Võimalikud põhjused:**

* Muudetud EXIF-metadata
* Pildid on redigeeritud välises tarkvaras
* Failide ülekanne on poolik

**Lahendus**: Importige kaamerast/SD-kaardilt uuesti originaalsed, muutmata failid

### Puuduvad ajamärgid

**Võimalikud põhjused:**

* Kaamera kell ei ole õigesti seadistatud
* EXIF-andmed on välise tarkvara poolt eemaldatud

**Lahendus**: Kontrollige, kas kaamera aja seaded olid pildistamise ajal õiged***

## Järgmised sammud

Kui failid on imporditud:

1. **Vaadake failide nimekirja** – veenduge, et kõik pildid on õigesti laaditud
2. **Kontrollige kaameramudeleid** – veenduge, et kaamera on õigesti tuvastatud
3. **Märkige sihtpildid** – vt [Sihtpiltide valimine](choosing-target-images.md)
4. **Kohandage seadeid** – konfigureerige töötlemisvalikud [Projekti seaded](adjusting-project-settings.md)
5. **Alustage töötlemist** – vt [Töötlemise alustamine](starting-the-processing.md)

Projekti konfigureerimise kohta leiate üksikasjalikku teavet jaotisest [Projekti seadete kohandamine](adjusting-project-settings.md).
