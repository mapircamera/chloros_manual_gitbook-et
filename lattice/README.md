# LATTICE-kaamerad

LATTICE on MAPIRi modulaarne multispektraalne kaamerasüsteem põllumajanduslikuks ja teaduslikuks pildistamiseks. Iga LATTICE-kaamera põhineb Sony IMX265 globaal-katikuga sensoril (**3,1 MP, 3,45 µm pikslid**) ja ühendub Etherneti kaudu**GigE Vision**-seadmena.

Chloros 1.2.0 juhib LATTICE-kaameraid reaalajas – avastamine, reaalajas eelvaade, pildistamine ja sünkroniseeritud mitme kaamera massiivid – kolmest keskkonnast:

| Keskkond    | Kus                                                          | Platvormid                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Graafiline kasutajaliides        | Chloros külgribal olev vahekaart **Kaamerad**                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice` käskude perekond                           | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Otsite riistvara?**Kaameramoodulid, objektiivid, filtrid ja lainepikkused, raamid ja kinnitused, kaablid, PoE ja päästemehhanismi juhtmestik on dokumenteeritud [**LATTICE&#x27;i kasutusjuhendis**](https://mapir.gitbook.io/lattice-camera). Käesolevas peatükis käsitletakse kaamerate juhtimist Chloros-i abil.

LATTICE salvestised on standardseid `.tif`/`.tiff`-failid ning Chloros töötleb neid alati alates tooresalvestisest. Vaata [CLI-viidet](../reference/cli-reference.md) ja [SDK-viidet](../reference/sdk-reference.md), et tutvuda täielike käskude ja API-pinnaga.

## Kaks andurikonfiguratsiooni

| Konfiguratsioon | Andur       | Filter                                | Mida üks kaamera edastab                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayeri värvisüsteem | kolmekordne ribapääsufilter                |**Kolm kalibreeritud laineala ühe ekspositsiooni tulemusena**                 |
| **M3M**| Monokroomne   | üks kitsasribaline interferentsfilter |**Üks kalibreeritud laineala**; indeksite saamiseks kombineerige mitu M3M-kaamerat |

Kuna M3M-kaamera on ühe filtri taga monokroomne, saab iga sagedusala oma ekspositsiooni. M3C-kaamera katab kõik kolm sagedusala ühe anduri ekspositsiooniga.

## Mudelistringid ja nimetamine

Iga kaamera salvestab oma identiteedi GenICam-is `DeviceUserID` kujul mudelistringina:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros kuvab seda eesliitega `LATT-` (näiteks `LATT-M3M-L87-F450`). Sama string `LATT-…` kirjutatakse iga ekspordi EXIF-märgisse `Model` ja seda kasutatakse töödeldud projektides kaamera väljundkausta nimena.

| Komponent | Väärtused                                                   | Tähendus                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Bayeri värv / mustvalge                                                                          |
| Objektiiv | `L41` / `L87`                                            | Number tähistab **horisontaalset vaatevälja kraadides**: L41 = kitsas (41°), L87 = lai (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) või `F<nm>` (M3M) | Vaata [Filtrid ja spektraalribad](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Mudelistring juhib kõiki järgnevaid protsesse: Chloros määrab kindlaks anduri profiili, ribade paigutuse ja tehase kalibreeringu väärtused, lähtudes `DeviceUserID` + `DeviceSerialNumber` väärtustest. Kaamera kohta pole midagi konfigureerida — vaata [Kaamerate ühendamine](connecting.md).

## Filtrid ja spektriribad

Spektriribade keskpunktid, FWHM servad ja kogu 23-SKU M3M kataloog on toote spetsifikatsioonid, seega asuvad need riistvara kasutusjuhendis: [**Filtrid ja spektriribad**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Tarkvara poolelt on oluline see, et mudeli stringis olev filtri kood määrab, milliseid tooteid Chloros suudab luua. RGB-filtriga kaamerad (`FRGB`) väljastavad ainult debayered- ja eelvaate tooteid — lairibaandilise anduri puhul ei ole ribapõhine kiirgus ja peegeldusvõime olulised, seega Chloros jätab need vahele ja märgib seda. Iga muu filter annab täieliku kiirguse → peegeldusvõime → indeksi ahela.

## Radiomeetriline kalibreerimine lühidalt

Iga LATTICE-kaamera kalibreeritakse tehases individuaalselt NIST-i jälgitava ahela alusel ning kaameraga kaasas on iga kaamera kohta eraldi sertifikaat. Mida see hõlmab, kuidas seda mõõdetakse ja millist täpsust võib nimetada, on kirjas riistvara kasutusjuhendis: [**Tehase radiomeetriline kalibreerimine**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Tarkvara poolelt on oluline, et Chloros määrab kaamera ühendamisel õige kalibreeringu ja fikseerib rakendatud koefitsiendid igasse eksporti — vt [Kaamerate ühendamine](connecting.md).

## Selles peatükis

* [Kaamerate ühendamine](connecting.md) — automaatne tuvastamine, graafilise kasutajaliidese ühendusdialoog, CLI/SDK ekvivalendid ning kuidas kaamera ühendamisel määratakse tehase kalibreerimine (kaamera sisemine pakett vs. pilv).

Muud LATTICE-teemad — kaamera seaded ja reaalajas juhtimine, salvestusrežiimid, mitme kaameraga süsteemid ning mono (M3M) töötlemine ja indeksid — on käsitletud käesoleva juhendi eraldi jaotistes ning täielik käskude loend asub [CLI viites](../reference/cli-reference.md) ja [SDK viite](../reference/sdk-reference.md).
