---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreerimismärklauad

MAPIR pakub erinevaid kalibreerimismärklaudu, mis sobivad mitmesugusteks rakendusteks. Allpool näidatud kompaktne T4-R50 sisaldab 4 paneeli, mille valguse peegeldusvõime on mõõdetud vahemikus 250–2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4 hajuspeegeldusega etalonitel on järgmised peegeldusastme kõverad, [andmed saab alla laadida siit](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 peegeldusvõime :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 peegeldusvõime :: 400–1000 nm</p></figcaption></figure>T4P hajuspeegeldusega võrdlusmärkidel on järgmised peegelduskurvid, [andmed alla laadida siit](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P peegeldusvõime :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P peegeldusvõime :: 400–1000 nm</p></figcaption></figure>Peegeldusgraafikut vaadates näete, et väärtused on esitatud lainepikkuse (x-telg) ja peegeldusprotsendi (y-telg) suhtena. Kui pildistame kalibreerimismärki, loome seose pikselväärtuse ja peegeldusprotsendi vahel spektris, mille suhtes iga kaamera anduririba on tundlik.

See tähendab, et iga meie kaameratega jäädvustatud pildi puhul saate kasutada meie peegeldusmärkide fotot, näiteks [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) või [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), et kalibreerida pildid peegeldusvõime järgi. Pärast kalibreerimist vastab iga pildi piksel peegeldusvõime protsendile.

**Survey3** väljundite puhul, kui kalibreeritud pildid salvestatakse formaadis Chloros tavalise JPG-failina või formaadis TIFF, arvutatakse peegeldusvõime protsent piksli väärtuse jagamisel pildiformaadi bittisügavusega. Seega JPG puhul jagatakse 255-ga ja TIFF puhul 65 535-ga. Võite valida ka väljundvormingu PERCENT failis Chloros, mille puhul iga pikseli väärtus jääb vahemikku 0,0–1,0 (peegeldusvõime 0%–100%). Pidage lihtsalt meeles, et mõned pildirakendused ei toeta protsendilisi (ujuvkomaarvulisi) pilte ning need võtavad salvestusruumis palju ruumi.

{% hint style="info" %}
**LATTICE peegeldusvõime kasutab teistsugust pikseliskaalat.** LATTICE peegeldusvõime salvestatakse nii, et DN 32768 = 100% peegeldusvõime (mitte 65535), ning iga fail sisaldab XMP-märget `Chloros:PixelScale`, mis näitab selle skaalat. Lugege silt läbi ja jagage sellega, selle asemel et eeldada konstanti – vt [Väljundpildiformaadid](output-image-formats.md).
{% endhint %}

## Kalibreerimismärgid LATTICE-kaameratega

LATTICE-kaamerate puhul on kalibreerimismärklaud peegeldusvõime jaoks **valikuline**: Chloros võib peegeldusvõime asemel viidata DAQ-valgusanduriga mõõdetud allapoole suunatud kiirgusintensiivsusele (ρ = π·L/E). Viide valitakse peegeldusallika seadistusega (GUI-s „Projektiseaded“; `--reflectance-source` failis CLI; `reflectance_source` failis SDK):

| Väärtus | Käitumine |
| --- | --- |
| `auto` *(vaikimisi)* | Kvaliteedikontrolli läbinud raamis olev sihtmärk on **absoluutne etalon**; kui sihtmärki pole või kvaliteedikontroll ebaõnnestub, kasutab Chloros tagavarana DAQ allapoole suunatud jagamist. |
| `target` | Ainult sihtmärk – DAQ-asendust ei kasutata. |
| `daq` | DAQ-põhine – allapoole suunatud mõõtmistulemus on alati võrdlusväärtus. |

LATTICE jaoks täiendav sihtmärgi käitumine:

* **Sihtmärkide geomeetriad** — toetatakse nii ArUco-märgistatud paneele, fikseeritud ROI-paneele kui ka ribasihtmärke; geomeetria pärineb projekti sihtmärgi konfiguratsioonist.
* **Ühiku kaupa mõõdetud sihtmärgi andmed** — `--target-reflectance-dir DIR` osutab kataloogile, kus asuvad ühiku kaupa mõõdetud sihtmärgi peegeldusvõime skaneeringud (`<serial>.csv`, mida otsitakse sihtmärgi ühiku seerianumbri/QR-koodi järgi). Kui sihtmärki ei leita, kasutab Chloros tagavarana nominaalseid T3/T4P spektreid.
* **Ajaline kinnitamine** — tuvastatud sihtmärk kalibreerib seda ümbritsevaid kaadreid ja hoitakse sihtmärgi vaatlemiste vahel.

Täielikud lipusemantika ja näited on toodud [CLI viites](reference/cli-reference.md) (vt „Toote kaupa ekspordi lülitid”).

### F988

„F988 peegeldusvõimet kalibreeritakse stseeni sees oleva peegeldusvõime paneeli abil: sagedusala jääb väljapoole DAQ-valgusanduri kalibreeritud vahemikku, seega rakendab Chloros teie viimast paneeli salvestust ja hoiab seda paneeli vaatamiste vahel.“

Kui F988 käivitatakse ainult DAQ-kalibreerimisega, lükkab Chloros selle lainepikkuse puhul DAQ-põhise peegeldusvõime tagasi ja selgitab põhjust (vahelejätmise põhjus `dls-uncalibrated-band-988`); paneeli töökäik on toetatud viis.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
