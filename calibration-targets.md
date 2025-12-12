---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreerimise sihtmärgid

MAPIR pakub erinevaid kalibreerimise sihtmärke, mis katavad mitmesuguseid rakendusi. Allpool näha olev kompaktne T4-R50 sisaldab 4 paneeli, mille valguse peegeldusvõime on mõõdetud vahemikus 250–2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4 hajusvalgustuse võrdlusmärklauad on järgmiste peegeldusvõime kõveratega, [andmed alla laadida siit](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 peegeldus :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 peegeldus :: 400–1000 nm</p></figcaption></figure>Peegeldusgraafikut vaadates näete, et väärtused on lainepikkus (x-telg) ja peegeldusprotsent (y-telg). Kui me kalibreerimise sihtmärgi pildi jäädvustame, loome seose pikselväärtuse ja peegeldusprotsendi vahel spektris, mille suhtes iga kaamera anduririba on tundlik.

See tähendab, et iga pildi puhul, mille te meie kaameratega pildistate, saate kasutada meie peegeldusmäära sihtmärkide fotot, näiteks [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) või [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), et kalibreerida pildid peegeldusmäära järgi. Pärast kalibreerimist on iga pildi piksel võrdne peegeldusprotsendiga.

Kui kalibreeritud pildid väljastatakse Chloros-is tüüpilise JPG- või TIFF-ina, arvutatakse peegeldusprotsent, jagades pikselväärtuse pildi formaadi bittisügavusega. Seega JPG puhul jagatakse 255-ga ja TIFF puhul jagatakse 65 535-ga. Võite valida ka PERCENT-vormingu väljundi Chloros-is, mille puhul iga piksel on vahemikus 0,0 kuni 1,0 (0% kuni 100% peegeldusvõime). Pange lihtsalt tähele, et mõned pildirakendused ei toeta protsentuaalseid (ujuvkomaga) pilte ja need on salvestusruumi poolest suured.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
