# Salvestamine ja .daq-vorming

`.daq`-fail on Chloros-i valgusanduri salvestusvorming: ühe DAQ-anduri kalibreeritud spektraalkaadrite **SQLite-andmebaas**. Salvestage üks selline fail salvestussessiooni ajal ja peegeldusvõimsuse töötlemisahel saab hiljem jagada iga pildi just sel hetkel mõõdetud allapoole suunatud kiirguse tugevusega.

## Mida .daq-fail sisaldab

| Omadus | Väärtus |
| --- | --- |
| Konteiner | SQLite-andmebaas, üks fail iga anduri kohta iga salvestuse kohta |
| Failinimi | Sisaldab **anduri ID-d**ja**ajamärget**, nt `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spekter kaadri kohta | 135 punkti, 340–1010 nm 5 nm sammuga, pluss CIE XYZ kolmestimuluse koordinaadid |
| Ühikud | Kalibreeritud spektraalne kiirgusintensiivsus, **W/m²/nm** (rakendatud tehase kalibreerimispakett + kateprofiil) |
| Märgitud metaandmed | Anduri ID (võti selle seadme tehase kalibreerimise andmete hankimiseks) ja kehtiv kateprofiil — vt [Kateprofiilid ja kalibreeritud vahemik](caps-and-range.md) |

Vorming on DAQ-U, DAQ-M ja DAQ-E puhul identne, seega ei ole järgneval töötlemisel oluline, milline andmeedastusseade selle salvestas.

Kalibreeritud salvestamiseks on vaja anduri tehase kalibreerimispaketti. DAQ-U ja DAQ-M puhul laadib tagapõhi komplekti MAPIR pilvest anduri ID järgi (kui see ei õnnestu, lükatakse salvestamine tagasi); DAQ-E seadmed on erandiks, kuna nende kalibreerimisandmed asuvad seadmes endas.

## Salvestamine graafilise kasutajaliidese kaudu

Salvestamine graafilises kasutajaliideses eeldab **avatud projekti** (muidu on salvestusnupud blokeeritud):

* **Salvesta kõik / Peata kõik** — valgusandurite külgriba ülaosas; käivitab või peatab `.daq` salvestuse kõigil ühendatud anduritel korraga.
* **Salvestamine / Salvestamise peatamine** — iga anduri jaoks eraldi, hammasratta kujulises seadete aknas. Salvestamise ajal kuvatakse anduri reaalajas teabe ridades punane „REC” märk.

Failid salvestatakse `<project>/light_sensor/`-i ja kui salvestamine peatatakse — kas nupu „Peata”, „Peata kõik” abil või salvestavat andurit lahti ühendades — lisatakse valmis `.daq` **automaatselt avatud projekti**. See ilmub projekti faililoendisse ilma käsitsi lisamiseta, olles juba valmis peegeldusvõime töötlemiseks.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Salvestamine CLI-ist

CLI salvestab tagapõhja andurite kaudu (tagapõhi peab töötama – need käsud on kerged HTTP-kliendid):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Hangi `--sensor-id` väärtus `chloros-cli daq pool-list`-ist. Kaks teadmisväärset vaikimisi seadet:

| Valik | Vaikimisi |
| --- | --- |
| `--duration` | `0` — salvestada kuni `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` **tagapõhja** failisüsteemis, mitte CLI-i omas |

Väljundkataloogi eristamine on oluline, kui CLI on suunatud teisel masinal asuvale backendile: fail salvestatakse sinna, kus backend töötab.

## Salvestamine Python-ist

`DAQSensorSession` (mille tagastab `chloros_sdk.connect_daq_sensor()`) näitab sama koondatud salvestust: `record_start(output_dir=None, device_name=None)` tagastab faili asukoha, `record_stop()` tagastab `{path, rows}`. Vaata [SDK-i viidet](../reference/sdk-reference.md), et tutvuda täieliku seansiga API. SDK-i otsesed riistvaraklassid (ainult töölauainstallatsioonid) kirjutavad salvestused vaikimisi `~/Documents/DAQ/`-i; väljalastud versioonide puhul on toetatud marsruudiks eespool nimetatud koondatud tee.

## .daq-faili kasutamine töötlemise ajal

Piltidest peegeldusvõimsuse arvutamiseks vajab Chloros iga ekspositsiooniga sobivat allapoole suunatud kiirgustihedust:

* **Hoidke `.daq` koos piltidega.**Töötlemise ajal määrab töötlemisahel automaatselt**ajamärgiga sobiva allapoole suunatud kiirguse** salvestatud `.daq`-failist (mis tahes DAQ-mudel) või piltide kõrval leiduvast DAQ-M-i omast `.csv`-failist — mis asub piltide kõrval. GUI-salvestused vastavad sellele nõudele automaatselt, kuna need lisatakse projektile kohe, kui salvestamine lõpeb.
* **Kalibreerimisandmed laaditakse alla nõudmise korral.** Kui kaamera- või DAQ-põhist tehase kalibreerimispaketti pole veel kohalikku vahemällu salvestatud, laadib Chloros selle esmakordsel kasutamisel automaatselt alla MAPIR pilvest (vajalik ühekordne internetiühendus; salvestatakse vahemällu `~/.chloros/` all).
* **Reaalajas salvestused loovad oma sidecar-faili.** Iga reaalajas salvestatud peegelduskaadri puhul salvestatakse tegelikult kasutatud DAQ-näit `.daq`-sidecar-failina pildi kõrvale, nii et salvestust saab hiljem uuesti töödelda ilma originaalsalvestuseta.

## Kiirguse andmete taastamine

Projekti töötlemisel eksporditakse ka kõik selle sisalduvad valgusanduri salvestised
`Light Sensor/`-kaustasse pilditoodete kõrvale. Selleks **ei** ole vaja pilte:
eraldi lennutatud valgussensor on täielik salvestus ning kaust, mis sisaldab ainult `.daq`
faile, on kehtiv sisend. Töötlemisprotsess annab aru, kui palju valgussensori tooteid see salvestas.

| Toode | Mis see on |
| --- | --- |
| `<name>_calibrated.daq` | Ümbertöödeldav arhiiv, mis järgib sama skeemi kui reaalajas salvestus, kuid sisaldab nüüd selle loomisel kasutatud kalibreerimispaketti. Selle uuesti importimine **ei** kalibreeri seda teist korda. |
| `<name>_calibrated.csv` | Spektraalne kiirgustihedus ühikutes W/m²/nm anduri enda lainepikkuste võrgustikus, üks rida iga mõõtmistulemuse kohta, pluss fotomeetrilised veerud: koguvõimsus, fotopiline ja skotopiline luks, PPFD koos sinise/rohelise/punase jaotusega ning tipp-lainepikkus. |

DAQ-U või DAQ-M, mille kalibreerimispaketti ei ole võimalik alla laadida — olete võrguühenduseta või
selle anduri kalibreerimisandmed puuduvad failist — **jätatakse põhjendusega vahele**, seda ei salvestata kunagi
„kalibreeritud” failina, mis sisaldab töötlemata loendusi. Ühenduge internetiga ja käivitage uuesti. DAQ-E
kannab oma kalibreeringut endas, seega vajab seda ainult siis, kui seade ei ole ühendatud ja
kohalikult pole midagi vahemällu salvestatud.

### DAQ-A: töötlemata loendid ja miks see on õige lahendus

**DAQ-A** on vanem kui seerianumbri-põhine kalibreerimispaketi süsteem ja sellel pole paketti, mida
alla laadida. See ei ole tähelepanematus: DAQ-A kalibreeritakse kohapeal
peegeldusmäära sihtmärgi abil ning sihtmärgipõhine kalibreerimine vajab ainult anduri *suhtelist*
vastust — mis on täpselt see, mida toorloendid esindavad. Chloros kalibreerib nendega tänapäeval.

Seega eksporditakse DAQ-A salvestus, kuid teise nime all:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, mitte `_calibrated` – tegemist on erineva failinimega, mitte failisiseselt märgistusega,
sest väide peab säilima ka siis, kui faili saadetakse e-postiga edasi pelgalt nimena. Pealkiri `.csv`
näitab `raw spectral sensor counts (NOT irradiance)` ja hoiatab, et väärtused on
võrreldavad **selle** faili piires, mitte erinevate andurite vahel. Veerud, mis on olulised
ainult tegeliku kiirguse puhul – koguvõimsus, luks, PPFD – jäetakse tühjaks, mitte
arvutatakse loendustest.

Vanemad DAQ-A-SD salvestused (skeem v1.01 / v1.02) salvestavad ainult faili kirjutamise aja, mitte
iga lugemise ajamärget. Chloros ei sobi neid pilte nendega kokku – kaadri sidumine
kirjutamisajaga oleks vale, kuigi see ei paistaks üldse valena –, kuid eksport loeb neid hästi ja
CSV näitab, millisel kellal see põhineb.

Täieliku teabe saamiseks peegeldusvõime kohta – ühe anduriga koos kaameraga ja kahe anduriga (ümbrus/objekt) – vaata [Peegeldusvõime töövood](reflectance.md).
