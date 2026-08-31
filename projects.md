# Graafiline kasutajaliides: Projektid

Chloros võimaldab teil luua projekte, mida saab tulevikus uuesti avada. Projekt on lihtne kaust (teie projektikaustas), mis sisaldab:

* `project.json` — projekti seaded, failide loend ja kuvamise eelistused
* `cameras.json` — projekti avamise ajal ühendatud kaamerad ja andurimassiivid koos nende seadistustega
* `sensors.json` — projekti avamise ajal ühendatud DAQ-valgusandurid ning kaamera↔anduri seosed
* teie pildistused, `.daq` salvestised ja töödeldud väljundkaustad

Projekti jaoks ei ole olemas spetsiaalset failivormingut – projektiks on kogu kaust koos selle JSON-failidega, mis muudab projektide kopeerimise, arhiveerimise ja [CLI](CLI.md) või [Python SDK](api-python-sdk.md) kaudu.

## Uus projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Valige peamenüüst „Uus projekt” ja sisestage oma projektile unikaalne nimi.

Kui olete salvestanud mõne projektimalli, ilmub nimevälja alla rippmenüü **Vali mall** – selle valimine alustab uut projekti selle malli seadetest. Mallid salvestatakse [Projekti seaded](project-settings/project-settings.md) kaudu: sisesta nimi väljale „Salvesta projektimalli nimi“ ja klõpsa salvestusikoonile.

## Ava projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>„Ava projekt“ loetleb kõik teie projektikausta projektid, allosas on <strong>nupp „Ava projektikausta“</strong></p></figcaption></figure>Valige „Ava projekt”, et näha projektikausta olemasolevate projektide nimekirja. Kui projekte pole, ei avane teine külgmenüü. Ülaltoodud pildil on näha mõned GUI abil loodud projektid (t1, t2, t3). Projektid DATE\_TIME loodi CLI abil, kasutades vaikimisi projektide nimetamisskeemi. Mis tahes projekti nime klõpsamine avab selle.

Nupule „Ava projektikaust” klõpsates avatakse teie arvuti failihaldur projekti asukohas. Projektikaustu asukohta saate muuta menüüs [Projekti seaded](project-settings/project-settings.md).

Kui mõni projekti allikafailidest on viimasest avamisest alates teisaldatud või kustutatud, kuvab Chloros dialoogakna, milles on täpselt loetletud, millised failid puuduvad, selle asemel, et avada tühi võrgustik.

## Projekti kopeerimine

Saadaval pärast projekti avamist. Valige „Projekti kopeerimine”, et kopeerida praegune projekt uue nimega — Chloros soovitab järgmist vaba nime (nt „MyProject (2)”) — ning koopia avatakse kohe.

## Failide lisamine

Pärast projekti avamist valige peamenüüst „Failide lisamine”, et lisada praegusesse projekti üksikuid pildifaile. See funktsioon peegeldab failibrauseri lisamisfunktsiooni, kuid mugavuse huvides on see kättesaadav otse peamenüüst.

## Kausta lisamine

Pärast projekti avamist valige peamenüüst „Kausta lisamine”, et lisada pildikaustaid praegusesse projekti. Ühe korraga saate valida mitu kausta. Duplikaatfaile ignoreeritakse.

## Töötlemise alustamine / peatamine

Pärast failide lisamist projektile muutub peamenüüs kättesaadavaks valik „Alusta töötlemist”. See on sama tegevus kui ülemises päises asuva nuppu „Esita/Alusta” klõpsamine. Töötlemise ajal muutub menüüvalik „Peata töötlemine”, mis võimaldab teil töötlemisprotsessi peatada.

## Ühenda kaameraga / Ühenda valgusanduriga

Peamenüü allosas on kaks riistvaralühendit, mis on kättesaadavad nii avatud kui ka suletud projekti korral:

* **Ühenda kaameraga** — avab [kaamerate vahekaardi](lattice/), et ühendada LATTICE-kaamera või -maatriks.
* **Ühendus valgusanduriga** — avab [Valgusandurid-vahekaardi](daq/), et ühendada DAQ-valgusandur.

Riistvara ühendamine avatud projekti korral salvestab selle projekti (vt allpool). Ilma projektita kehtivad ühendused ainult käesoleva seansi ajaks.

{% hint style="info" %}
Menüüvalikud „Lisa faile”, „Lisa kausta” ja „Alusta/peata töötlemine” on nähtavad või aktiveeritud ainult siis, kui projekt on avatud ja failid on lisatud. Need võimaldavad kiiret juurdepääsu toimingutele, mis on kättesaadavad ka failibrauseri külgribal ja päisnupudest.
{% endhint %}

## Projektid mäletavad teie riistvara

Uus versioonis 1.2.0: projekt säilitab selle avatuse ajal ühendatud riistvara. Kaamerad ja massiivid (koos iga kaamera seadistuste, nimede, värvide ja rasterpaigutusega) salvestatakse hetktõmmisena faili `cameras.json` ning valgusandurid (koos nimede, värvide ja kaameraga seostamisega) faili `sensors.json` — automaatselt, kui töötad.

Kui **avada** projekt uuesti, ei puutu Chloros kohe ühegi riistvaraga kokku. Iga pool ühendub uuesti esmakordsel sisenemisel selle vahelehele, millele see kuulub:

* **Kaamerad**-vahekaardi avamisel ühendatakse salvestatud kaamerad ja massiivid uuesti ning rakendatakse nende salvestatud seaded.
* **Valgusandurid**-vahekaardi avamisel ühendatakse uuesti salvestatud DAQ-andurid.

Sel viisil ei lülita projektide avamine pelgalt piltide sirvimiseks või eksportimiseks kaameraid kunagi voogedastusrežiimi. Kui salvestatud seadet ei leita vahekaardi avamisel, teatab dialoogaken, millised seadmed on kättesaamatud, et saaksite need uuesti ühendada või eemaldada.

## DAQ-salvestused ja .daq-failid projektis

* Projekti avatuna tehtud `.daq`-salvestused (vahekaardilt „Valgusandurid“ või salvestamise ajal) **lisatakse projektile automaatselt**.
* Imporditud `.daq`-failid ja kõik projekti salvestused on loetletud [Projekti seaded](project-settings/project-settings.md) jaotises **DAQ valgusandur**, igaühel koos oma valgustugevuse korrigeerimisprofiiliga.
* Töötlemise ajal pakuvad projekti `.daq`-failid allapoole suunatud valgustust peegeldusvõime tulemuste jaoks — vt [Väljundpildi formaadid](output-image-formats.md).

## Salvestatud projekti käivitamine ilma graafilise kasutajaliideseta

Salvestatud projekti saab käivitada ilma graafilise kasutajaliideseta:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` töötab projekti kausta teekonnaga — vt [CLI viide](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` tagastab projekti käepideme; `connect_all()` lülitab kõik salvestatud kaamerad ja andurid koos nende salvestatud seadistustega võrku — vt [SDK viide](reference/sdk-reference.md).
