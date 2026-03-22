---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreerimismärklauad

MAPIR pakub erinevaid kalibreerimismärklaudu mitmesugusteks rakendusteks. Allpool näidatud kompaktne mudel T4-R50 sisaldab 4 paneeli, mille valguse peegeldusvõime on mõõdetud vahemikus 250–2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4 hajusvalgusega etalonitel on järgmised peegelduskurvid, [andmed alla laadida siit](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 peegeldusvõime :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 peegeldusvõime :: 400–1000 nm</p></figcaption></figure>T4P hajusvalgusega võrdlusmärkidel on järgmised peegelduskurvid, [andmed alla laadida siit](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P peegeldusvõime :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P peegeldusvõime :: 400–1000 nm</p></figcaption></figure>Peegeldusgraafikut vaadates näete, et väärtused on esitatud lainepikkuse (x-telg) ja peegeldusprotsendi (y-telg) suhtena. Kui pildistame kalibreerimismärki, loome seose pikselväärtuse ja peegeldusprotsendi vahel spektris, millele iga kaamera anduririba on tundlik.

See tähendab, et iga meie kaameratega jäädvustatud pildi puhul saate kasutada meie peegeldusmärke, näiteks [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) või [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), et kalibreerida pilte peegelduse järgi. Pärast kalibreerimist vastab iga pildi piksel protsendilisele peegeldusele.

Kui väljastate kalibreeritud pildid Chloros-is tavalise JPG- või TIFF-vorminguna, arvutatakse peegeldusprotsent piksli väärtuse jagamisel pildiformaadi bittisügavusega. Seega JPG-vormingu puhul jagatakse 255-ga ja TIFF-vormingu puhul 65 535-ga. Võite valida ka PERCENT-vormingu väljundi Chloros-is, mille puhul iga pikseli väärtus jääb vahemikku 0,0–1,0 (0%–100% peegeldusvõime). Pange lihtsalt tähele, et mõned pilditöötlusrakendused ei toeta protsentuaalseid (ujuvkomaarvulisi) pilte ning need võtavad salvestusruumis palju ruumi.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
