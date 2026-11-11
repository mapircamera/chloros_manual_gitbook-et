---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Toetatud kaamerad

Chloros
töötleb **kõikidel platvormidel** (Windows
, Linux
amd64 ja Linux
arm64/Jetson) kahe MAPIR
-kaameraseeria pilte:

* **Survey3** — Survey3W
(laia vaateväljaga) ja Survey3N
(kitsa vaateväljaga) kaamerad. Sisend: `RAW+JPG`.
* **LATTICE**— M3C ja M3M multispektraalsed kaameramoodulid. Sisend: `.tif`/`.tiff` pildid. LATTICE-kaameraid saab**reaalajas juhtida** ka Chloros
i kaudu — kasutajaliidese kaamerate vahekaardi (Windows
) või `chloros-cli lattice` / Python
i SDK
i (Windows
ja Linux
) abil — sealhulgas sünkroniseeritud mitme kaameraga süsteeme. Vaata [LATTICE-juhendit](lattice/).

Töötlemisprotsess toetab ka `.dng` sisendfaile.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Tootja</th><th width="250">Kaamera mudel</th><th width="138">Filtri mudel</th><th width="187">Pildi tüüp</th></tr></thead><tbody><tr><td><strong>Tootja</strong></td><td><strong>Kaamera mudel</strong></td><td><strong>Filtri mudel</strong></td><td><strong>Pildi tüüp</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

LATTICE-seeria on modulaarne multispektraalne kaamerasüsteem, mis põhineb Sony IMX265 globaalse katikuga sensoril (3,1 MP, 3,45 µm pikslid). Iga kaamera salvestab oma identiteedi mudelistringina:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```.

Chloros

kuvab seda eesliitega `LATT-` (näiteks `LATT-M3M-L41-F550`), ning mudelistring juhib kõiki järgnevaid protsesse – anduri profiil, lainepikkuste paigutus ja kalibreerimine määratakse automaatselt; iga kaamera jaoks eraldi pole vaja midagi seadistada. Objektiivi number tähistab **horisontaalset vaatevälja kraadides**: `L41` = kitsas 41°, `L87` = lai 87°.

Andureid on kaks konfiguratsiooni:

| Konfiguratsioon | Andur      | Filtri tüüp                           | Sagedusribad kaamera kohta                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayeri värvifilter | Kolmekordne ribapääsufilter                       | 3 spektraalset riba ühe ekspositsiooni kohta                                 |
| **M3M**       | Monokroomne  | Ühe kitsasribaline interferentsfilter | 1 kalibreeritud riba — kombineerige mitu M3M-kaamerat taimestiku indeksite saamiseks |

### M3C (Bayer) filtri valikud

| Filter | Lained (nimi @ keskpunkt nm / FWHM

nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue

475/30 · Green

550/30 · Red

625/30  |
| `FRGN` | Red

660/21 · Green

550/30 · NIR

850/30   |
| `FOCN` | Orange

618,4/46 · Cyan

494,1/40 · NIR

821,6/54 (mõõdetud — vt allpool) |
| `FNGB` | Blue

475/30 · Green

550/30 · NIR

850/30  |

`FOCN` puhul on väärtused filtri **mõõdetud** poolmaksimaalne lainepikkus (Chloros

1.3.0 ja uuemad versioonid; varasemates versioonides kasutati tehniliste andmete väärtusi 615/21 · 490/38 · 808/14). Ribade nimed jäävad samaks: `Orange_615`, `Cyan_490` ja `NIR_808`, ning mõõdetud keskpunkt ja laius on need, mida Chloros

märgib igasse kalibreeritud ekspordifaili.

### M3M (mono) filtri kataloog — 23 SKU-d

F-number on SKU-märge; mõõdetud lainepikkus (mis on märgitud igale kalibreeritud ekspordile) on partii kohta tehtud filtri skaneering:

| SKU    | Keskpunkt (nm, mõõdetud) | FWHM

servad (nm) | Laius (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominaalne)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominaalne)           | 867–893         | 26         |
| F905   | — (nominaalne)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_„Ribade servad on mõõdetud poolmaksimumilise laiusena MAPIR

i partiipõhiste filtriskannide põhjal — need on samad väärtused, mida Chloros

märgib igasse kalibreeritud ekspordisse.“_ „— (nominaalne)“ = partii skaneeringut pole veel tehtud; nende SKU-de puhul on märgitud keskpunktiks SKU-number ja laiuseks tootja poolt esitatud väärtus.

† „F988 peegeldusvõimet kalibreeritakse stseeni sees oleva peegeldusvõime paneeli abil: sagedusala jääb väljapoole DAQ-valgusanduri kalibreeritud vahemikku, seega kasutab Chloros

teie viimast paneeli pildistust ja säilitab selle paneeli vaatluste vahel.“ Vaata [kalibreerimismärke](calibration-targets.md).

Kaamera reaalajas juhtimise, massiivide, võrguseadistuste ja radiomeetrilise töötlemisahela kohta vaata [LATTICE juhendit](lattice/).
