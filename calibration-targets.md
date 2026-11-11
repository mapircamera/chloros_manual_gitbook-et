---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreerimismärklauad

MAPIR pakub mitmesuguseid kalibreerimismärklaudu, mis sobivad erinevateks rakendusteks. Allpool näidatud kompaktne T4-R50 sisaldab 4 paneeli, mille valguse peegeldusvõime on mõõdetud vahemikus 250–2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4 hajuspeegeldusega etalonitel on järgmised peegelduskurvid, [andmed allalaadimiseks siit](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIRi T4 peegeldusvõime :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIRi T4 peegeldusvõime :: 400–1000 nm</p></figcaption></figure>T4P hajusvalgusega võrdlusmärkidel on järgmised peegelduskurvid, [andmed saab alla laadida siit](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIRi T4P peegeldusvõime :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIRi T4P peegeldusvõime :: 400–1000 nm</p></figcaption></figure>Peegeldusgraafikut vaadates näete, et väärtused on esitatud lainepikkuse (x-telg) ja peegeldusprotsendi (y-telg) suhtena. Kui pildistame kalibreerimismärki, loome seose pikselväärtuse ja peegeldusprotsendi vahel spektris, mille suhtes iga kaamera anduririba on tundlik.

See tähendab, et iga meie kaameratega jäädvustatud pildi puhul saate kasutada meie peegeldusmärke, näiteks [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) või [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), et kalibreerida pildid peegeldusvõime järgi. Pärast kalibreerimist vastab iga pildi piksel peegeldusvõime protsendile.

**Survey3** väljundite puhul, kui ekspordite kalibreeritud pilte Chloros-is tavalise JPG- või TIFF-vorminguna, arvutatakse peegeldusprotsent piksli väärtuse jagamisel pildiformaadi bittisügavusega. Seega JPG-vormingu puhul jagatakse 255-ga ja TIFF-vormingu puhul 65 535-ga. Võite valida ka **Chloros**-is väljundvormingu PERCENT, mille puhul iga pikseli väärtus jääb vahemikku 0,0–1,0 (0%–100% peegeldusvõime). Pange lihtsalt tähele, et mõned pilditöötlusrakendused ei toeta protsentides (ujuvkomaarvudes) väljendatud pilte ning need võtavad salvestusruumis palju ruumi.

{% hint style="info" %}
**LATTICE-peegeldusvõime kasutab teistsugust pikseliskaalat.** LATTICE-peegeldusvõime salvestatakse nii, et DN 32768 = 100% peegeldusvõime (mitte 65535), ning iga fail sisaldab XMP-sildit `Chloros:PixelScale`, mis näitab selle skaalat. Lugege silt ja jagage sellega, selle asemel et eeldada konstanti — vt [Väljundpildi formaadid](output-image-formats.md).
{% endhint %}

## Kalibreerimismärgid LATTICE-kaameratega

LATTICE-kaamerate puhul on kalibreerimismärk peegeldusvõime jaoks **valikuline**: Chloros võib peegeldusvõime asemel võtta aluseks DAQ-valgusanduri poolt mõõdetud allapoole suunatud kiirgustiheduse (ρ = π·L/E). Viide valitakse projekti peegeldusallika seadistusega, mis määratakse `chloros-cli process --reflectance-source` abil (graafilises kasutajaliideses puudub selle jaoks juhtelement ning kasutatakse projekti salvestatud väärtust, vaikimisi `auto` — vt [Projekti seaded](project-settings/project-settings.md)):

| Väärtus | Käitumine |
| --- | --- |
| `auto` *(vaikimisi)* | Kvaliteedikontrolli läbinud raamisisene sihtmärk on **absoluutne võrdluspunkt**; kui sihtmärki pole või kvaliteedikontroll ebaõnnestub, kasutab Chloros tagavarana DAQ-i allapoole suunatud jaotust. Sihtmärk, mis läbib kvaliteedikontrolli lipuga (paneel on välja jäänud või sagedusala ei tööta), ei ole automaatselt eelistatud: proovitakse järjest sihtmärki, sihtmärki+DAQ-i ja ainult DAQ-i ahelaid vastavalt nende mõõdetud skoorile väljajäetud paneelidel. |
| `target` | Rangelt ainult sihtmärk — DAQ-asendust ei kasutata. |
| `daq` | DAQ-põhine — allapoole suunatud mõõtmine on alati võrdlusaluseks. |

LATTICE jaoks täiendav sihtmärgi käitumine:

* **Sihtmärgi geomeetriad** — toetatakse nii ArUco-märgistatud paneele, fikseeritud huvipiirkonnaga (ROI) paneele kui ka ribasihtmärke; geomeetria pärineb projekti sihtmärgi konfiguratsioonist.
* **Ühiku kaupa mõõdetud sihtmärgi andmed** — `--target-reflectance-dir DIR` osutab kataloogile, kus asuvad ühiku kaupa mõõdetud sihtmärgi peegeldusvõime skaneeringud (`<serial>.csv`, mida otsitakse sihtmärgi ühiku seerianumbri/QR-koodi järgi). Kui sihtmärki ei leita, kasutab Chloros tagavarana nominaalseid T3/T4P spektreid.
* **Ajaline kinnistamine** — tuvastatud sihtmärk kalibreerib seda ümbritsevaid kaadreid ja hoitakse sihtmärgi vaatluste vahel.

Täielikud lipusemantika ja näited on toodud [CLI-viites](reference/cli-reference.md) (vt „Toote-põhised ekspordilülitid”).

### F988

„F988 peegeldusvõimet kalibreeritakse stseeni sees oleva peegeldusvõime paneeli abil: sagedusala jääb väljapoole DAQ-valgusanduri kalibreeritud vahemikku, seega kasutab Chloros teie viimast paneeli salvestust ja hoiab seda paneeli vaatamiste vahel.“

Kui F988 käivitatakse ainult DAQ-kalibreerimisega, lükkab Chloros selle lainepikkuse puhul DAQ-põhise peegeldusvõime tagasi ja selgitab põhjust (vahelejätmise põhjus `dls-uncalibrated-band-988`); toetatud viis on paneeli kasutamine.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
