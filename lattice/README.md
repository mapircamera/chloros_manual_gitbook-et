# LATTICE-kaamerad

LATTICE on MAPIRi modulaarne multispektraalne kaamerasüsteem põllumajanduslikuks ja teaduslikuks pildistamiseks. Iga LATTICE-kaamera põhineb Sony IMX265 globaalse katikuga sensoril (**3,1 MP, 3,45 µm pikslid**) ja ühendub Etherneti kaudu**GigE Vision**-seadmena.

Chloros 1.3.0 juhib LATTICE-kaameraid reaalajas – avastamine, reaalajas eelvaade, pildistamine ja sünkroniseeritud mitme kaamera massiivid – kolmest kasutajaliidesest:

| Kasutajaliides    | Kus                                                          | Platvormid                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Graafiline kasutajaliides        | **Kaamerad**-vahekaart Chlorosi külgribal                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice`-käskude perekond                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Pythoni SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Otsite riistvara?**Kaameramoodulid, objektiivid, filtrid ja lainepikkused, raamid ja kinnitused, kaablid, PoE ja päästemehhanismide juhtmestik on dokumenteeritud [**LATTICE'i kasutusjuhendis**](https://mapir.gitbook.io/lattice-camera). Käesolevas peatükis käsitletakse kaamerate juhtimist Chloros'i kaudu.

LATTICE salvestused on standardse formaadiga `.tif`/`.tiff` failid ning Chloros töötleb neid alati algandmetest alates. Täieliku käskude ja API-liideste loetelu leiate [CLI-viitest](../reference/cli-reference.md) ja [SDK-viitest](../reference/sdk-reference.md).

## Kaks andurikonfiguratsiooni

| Konfiguratsioon | Andur       | Filter                                | Mida üks kaamera edastab                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayeri värv | kolmekordne ribapääsufilter                |**Kolm kalibreeritud laineala ühe ekspositsiooni kohta**                 |
| **M3M**| Monokroomne   | üks kitsasribaline interferentsfilter |**Üks kalibreeritud sagedusala**; indeksite saamiseks kombineerige mitut M3M-kaamerat |

Kuna M3M-kaamera on ühe filtri taga monokroomne, saab iga sagedusala oma eraldi ekspositsiooni. M3C-kaamera katab kõik kolm oma sagedusala ühe anduri ekspositsiooniga.

## Mudelistringid ja nimetamine

Iga kaamera salvestab oma identiteedi GenICam-is `DeviceUserID` mudelistringina:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros kuvab seda eesliitega `LATT-` (näiteks `LATT-M3M-L87-F450`). Sama string `LATT-…` kirjutatakse iga ekspordi EXIF-märgisse `Model` ja seda kasutatakse töödeldud projektides kaamera väljundkausta nimena.

| Komponent | Väärtused                                                   | Tähendus                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Bayeri värv / mustvalge                                                                          |
| Objektiiv | `L41` / `L87`                                            | Number tähistab **horisontaalset vaatevälja kraadides**: L41 = kitsas (41°), L87 = lai (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) või `F<nm>` (M3M) | Vaata [Filtrid ja spektraalribad](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Mudelistring juhib kõiki järgnevaid protsesse: Chloros määrab kindlaks anduri profiili, ribade paigutuse ja tehase kalibreeringu väärtused kombinatsioonist `DeviceUserID` + `DeviceSerialNumber`. Iga kaamera jaoks pole vaja midagi eraldi seadistada — vaata [Kaamerate ühendamine](connecting.md).

## Filtrid ja spektriribad

Spektriribade keskpunktid, FWHM-servad ja kogu 23 SKU-ga M3M-kataloog on toote spetsifikatsioonid, seega asuvad need riistvara kasutusjuhendis: [**Filtrid ja spektriribad**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Tarkvara poolelt on oluline järgmine: mudeli stringis olev filtri kood määrab, milliseid tooteid Chloros suudab luua. RGB-filtriga kaamerad (`FRGB`) väljastavad ainult debayereeritud ja eelvaate tooteid — lairibaandilise anduri puhul ei ole sagedusriba-põhine kiirgus ja peegeldusvõime olulised, seega Chloros jätab need vahele ja teavitab sellest. Iga muu filter annab täieliku kiirgus → peegeldusvõime → indeks ahela.

## Radiomeetriline kalibreerimine lühidalt

Iga LATTICE-kaamera kalibreeritakse tehases individuaalselt NIST-i jälgitava ahela alusel ning tarnitakse koos iga kaamera sertifikaadiga. Mida see hõlmab, kuidas seda mõõdetakse ja millist täpsust saate esitada, on kirjas riistvara kasutusjuhendis: [**Tehase radiomeetriline kalibreerimine**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Tarkvara poolelt on oluline, et Chloros määraks kaamera ühendamisel õige kalibreeringu ja fikseeriks rakendatud koefitsiendid igasse ekspordisse — vt [Kaamerate ühendamine](connecting.md).

## Selles peatükis

* [Kaamerate ühendamine](connecting.md) – automaatne tuvastamine, graafilise kasutajaliidese ühendusdialoog, CLI- ja SDK-vasted ning kuidas kaamera ühendamisel määratakse tehase kalibreerimine (kaamerasisene pakett vs. pilv).
* [Kaamera seaded](camera-settings.md) — kõik „Kaamerad” vahekaardi juhtelemendid: kaamerate nimekiri, kaamerapõhised seaded ja massiivi seaded (sh huvipiirkonna (ROI) redaktor ja joondamine).
* [Salvestusseaded ja -režiimid](capture.md) — „Salvesta kõik”, üksik-, pidev- ja intervall-salvestus, eksporditüübid ning salvestuste kausta struktuur.
* [Mitme kaamera massiivid](arrays.md) — massiivi ühendamise viisard, sünkroniseerimiskaabli sond, ribalaius, PTP ja massiivi reaalaja vaade.
* [Mono-kaamerad ja taimestiku indeksid](mono-indices.md) — M3M-kaamerad ning joondamine → kihistamine → indeksimine.

Täielik käskude loetelu on esitatud [CLI-viites](../reference/cli-reference.md) ja [SDK-viites](../reference/sdk-reference.md).
