# Graafiline kasutajaliides: Projektid

Chloros’is saate luua projekte, mida on võimalik tulevikus uuesti avada. Projekt on tavaline kaust (teie projektikaustas), mis sisaldab järgmist:

* `project.json` — projekti seaded, failide loend ja kuvamise eelistused
* `cameras.json` — projekti avamise ajal ühendatud kaamerad ja andurimassiivid koos nende seadistustega
* `sensors.json` — projekti avamise ajal ühendatud DAQ-valgusandurid koos kaamera↔anduri seostega
* teie pildistused, `.daq` salvestused ja töödeldud väljundkaustad

Projekti jaoks ei ole olemas spetsiaalset failivormingut — projekt koosneb kaustast ja selle JSON-failidest, mis muudab projektide kopeerimise, arhiveerimise ja käivitamise [CLI-i](CLI.md) või [Python SDK-i](api-python-sdk.md) kaudu lihtsaks.

Kui te seda [projekti seadetest](project-settings/project-settings.md) ei muuda, asub projektikausta `Documents\MAPIR\Chloros Projects` teie „Dokumendid” kausta (Linuxis `~/Documents/MAPIR/Chloros Projects`).

{% hint style="warning" %}
**Windows: hoidke projektikaustad lühikesed, kui pikad teed pole sisse lülitatud.**Windowsis on pikad teed vaikimisi välja lülitatud ning sel juhul ei tohi ükski failitee olla pikem kui 260 tähemärki. Töödeldud tooted asuvad kuni umbes 130 tähemärki allpool projektikausta, mistõttu alates versioonist 1.3.0 keeldub Chloros liiga sügava projektikausta kasutamisest – nii selle loomisel, avamisel kui ka töötlemise alustamisel – selle asemel, et programmi käivitada ja midagi kirjutamata jätta. Teates näidatakse pikim sobiv projektikaust. Kasutage lühemat projektikausta või lülitage sisse Windowsi pikad teed (paigaldaja valik**Luba Windowsi pikad teed**, mis on vaikimisi märgitud) ja taaskäivitage Chloros. Vaadake [Töötlemise alustamine](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Uus projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Valige peamenüüst „Uus projekt“ ja sisestage oma projektile unikaalne nimi.

Kui olete salvestanud mõne projektimalli, ilmub nimevälja alla rippmenüü **Vali mall** — selle valimine alustab uut projekti selle malli seadetest. Mallid salvestatakse menüüst [Project Settings](project-settings/project-settings.md): sisestage nimi väljale „Save Project Template Name“ ja klõpsake salvestusikoonile.

## Projekti avamine

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>„Ava projekt“ loetleb kõik teie projektikausta projektid, allosas on <strong>nupp Ava projektikausta</strong></p></figcaption></figure>

Valige „Ava projekt”, et näha projektikausta olemasolevate projektide nimekirja. Kui projekte pole, ei avane teine külgmenüü. Ülaloleval pildil on näha mõned GUI abil loodud projektid (t1, t2, t3). Projektid DATE\_TIME on loodud CLI-i abil, kasutades vaikimisi projektide nimetamisskeemi. Mis tahes projekti nime klõpsates avatakse see.

Nupule „Ava projektikausta” klõpsates avatakse teie arvuti failibrauser projekti asukohas. Projekti asukohta saate muuta [Projekti seaded](project-settings/project-settings.md) menüüs.

Kui mõni projekti algpildifail on viimasest avamisest alates teisaldatud või kustutatud, näitab Chloros dialoogaknas täpselt, millised failid puuduvad, selle asemel et avada tühi raster.

## Projekti kopeerimine

Saadaval pärast projekti avamist. Valige „Kopeeri projekt”, et kopeerida praegune projekt uue nimega – Chloros soovitab järgmist vaba nime (nt „MinuProjekt (2)”) – ning koopia avatakse kohe.

## Failide lisamine

Pärast projekti avamist valige peamenüüst „Failide lisamine”, et lisada praegusesse projekti üksikuid pildifaile. See toimib sarnaselt failibrauseri lisamisfunktsiooniga, kuid mugavuse huvides on see kättesaadav otse peamenüüst.

## Kausta lisamine

Kui projekt on avatud, valige peamenüüst „Kausta lisamine”, et lisada pildikaustaid praegusesse projekti. Ühe korraga saate valida mitu kausta. Duplikaatfaile ignoreeritakse.

## Töötlemise alustamine / peatamine

Pärast failide lisamist projekti muutub peamenüüs kättesaadavaks valik „Alusta töötlemist”. See on sama tegevus kui ülemises päises asuva nuppu „Esita/Alusta” klõpsamine. Töötlemise ajal muutub menüüvalik „Peata töötlemine”, mis võimaldab teil töötlemisprotsessi peatada.

## Ühenda kaameraga / Ühenda valgusanduriga

Peamenüü allosas on kaks riistvaralühendit, mis on kättesaadavad nii avatud kui ka suletud projekti korral:

* **Ühenda kaameraga** — avab [kaamerate vahekaardi](lattice/), et ühendada LATTICE-kaamera või -maatriks.
* **Ühenda valgusanduriga** — avab [Valgusandurid-vahekaardi](daq/), et ühendada DAQ-valgusandur.

Riistvara ühendamine avatud projekti korral salvestab selle projekti (vt allpool). Ilma projektita kehtivad ühendused ainult käesoleva seansi ajaks.

{% hint style="info" %}
Menüüvalikud „Lisa faile”, „Lisa kausta” ja „Alusta/peata töötlemine” on nähtavad või aktiveeritud ainult siis, kui projekt on avatud ja failid on lisatud. Need võimaldavad kiiret juurdepääsu toimingutele, mis on kättesaadavad ka failibrauseri külgribal ja päisenuppude kaudu.
{% endhint %}

## Projektid mäletavad teie riistvara

Uus versioonis 1.2.0: projekt säilitab selle avatuse ajal ühendatud riistvara. Kaamerad ja massiivid (koos kaamerapõhiste seadistuste, nimede, värvide ja rasterpaigutusega) salvestatakse hetktõmmisena faili `cameras.json` ning valgusandurid (koos nimede, värvide ja kaameraga seostamisega) faili `sensors.json` — automaatselt, kui te töötate.

Kui te projekti **uuesti avate**, ei puutu Chloros kohe ühegi riistvaraga kokku. Iga pool ühendub uuesti, kui te esimest korda avate selle jaoks mõeldud vahekaardi:

* Vahekaardi **Kaamerad** avamisel ühendatakse salvestatud kaamerad ja massiivid uuesti ning rakendatakse nende salvestatud seaded.
* **Valgusandurite**vahekaardi avamisel ühendatakse salvestatud DAQ-andurid uuesti ja rakendatakse nende salvestatud seaded, kaane seaded kaasa arvatud. Andur, mille**Cap**-menüüs on valitud**None (paljas andur)**, jääb paljaks. Varasema Chlorosi versiooniga salvestatud projektis registreeriti paljas andur kui valikuvõimaluseta, seega taastub see koos päikesepaiste koosinuskorrektsiooniga; seadke see uuesti**None**-iks ja salvestage projekt.

Sel viisil ei lülita projekti avamine pelgalt piltide sirvimiseks või eksportimiseks kaameraid kunagi voogedastusrežiimi. Kui salvestatud seadet ei leita selle vahekaardi avamisel, teatab dialoogaken, millised seadmed on kättesaamatud, et saaksite need uuesti ühendada või eemaldada.

## DAQ-salvestused ja .daq-failid projektis

* Projekt avatuna tehtud `.daq`-salvestused (vahekaardilt „Valgusandurid“ või pildistamise ajal) **lisatakse projektile automaatselt**.
* Imporditud `.daq`-failid ja kõik projekti salvestused on loetletud [Projekti seaded](project-settings/project-settings.md) jaotises **DAQ valgusandur**, igaühel koos oma valgustugevuse korrigeerimisprofiiliga.
* Töötlemise ajal pakuvad projekti `.daq`-failid allapoole suunatud valgustust peegeldusvõimsuse tulemuste jaoks — vt [Väljundpildi formaadid](output-image-formats.md).

## Salvestatud projekti käivitamine graafilise kasutajaliidese kasutamiseta

Salvestatud projekti saab käivitada ilma graafilise kasutajaliideseta:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` töötab projekti kausta teekonna alusel — vt [CLI-viide](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` tagastab projekti käepideme; `connect_all()` lülitab kõik salvestatud kaamerad, massiivid ja valgussensorid sisse koos nende salvestatud seadistustega (valgussensorid töötava Chloros-tagapõhja kaudu) — vt [SDK-viidet](reference/sdk-reference.md).
