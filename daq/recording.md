# Salvestamine ja .daq-vorming

`.daq`-fail on Chloros i valgusanduri salvestusvorming: ühe DAQ-anduri kalibreeritud spektraalkaadrite **SQLite-andmebaas**. Salvestage üks selline fail salvestussessiooni ajal ja peegeldusvõimsuse töötlemisahel saab hiljem jagada iga pildi just sel hetkel mõõdetud allapoole suunatud kiirguse tihedusega.

## Mida .daq-fail sisaldab

| Omadus | Väärtus |
| --- | --- |
| Konteiner | SQLite-andmebaas, üks fail iga anduri ja salvestuse kohta |
| Failinimi | Sisaldab **anduri ID-d**ja**ajamärget**, nt `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spekter kaadri kohta | 135 punkti, 340–1010 nm 5 nm sammuga, pluss CIE XYZ kolmestimuluse koordinaadid |
| Ühikud | Kalibreeritud spektraalne kiirgustihedus, **W/m²/nm** (rakendatud tehase kalibreerimispakett + kateprofiil) |
| Märgitud metaandmed | Anduri ID (võti selle seadme tehase kalibreerimise andmete hankimiseks) ja kehtiv kateprofiil — vt [Kateprofiilid ja kalibreeritud vahemik](caps-and-range.md) |

Formaat on identne mudelitel DAQ-U, DAQ-M, DAQ-E ja DAQ-E-S, seega ei ole järgneval töötlemisel oluline, milline andmeedastusseade salvestuse tegi. DAQ-E-S salvestus talletab iga spektri kõrvale ka seadme kallutuse, pöörde ja kaldenurga.

**Salvestised säilivad ka voolukatkestuse korral (1.3.0).**Reaalajas salvestamine toimub ajakirja abil ja iga salvestamise käigus toimub täielik kettasünkroniseerimine, mistõttu voolukatkestus või protsessi katkestamine kaotab umbes viimased 3 sekundit, mitte kogu faili. Kirjutamise käigus peatunud salvestus võib jätta faili `.daq` kõrvale faili `<name>.daq-journal`:**säilitage ja kopeerige need kaks koos.** Chloros kerib ajakirja tagasi, kui loeb faili esimest korda. Hõivatud ketas viivitab faili salvestamist, kuid mitte kunagi anduri näidud.

**Andmeid edastamast lakanud DAQ-E ühendatakse uuesti (1.3.0).** Kui DAQ-E andmevoog katkeb (lüliti toide lülitati välja, ühendus katkes), ühendab tagapõhi selle uuesti varasemate seadistustega ning salvestamine jätkub samas `.daq`-failis, kus lünk on failis märgitud.

Kalibreeritud salvestamine nõuab anduri tehase kalibreerimispaketti. DAQ-U ja DAQ-M puhul laadib backend paketi MAPIR i pilvest anduri ID järgi alla (kui see ei õnnestu, lükatakse salvestamine tagasi); DAQ-E seadmed on sellest vabastatud, kuna nende kalibreerimisandmed on salvestatud seadmes endas.

## Salvestamine graafilise kasutajaliidese kaudu

Salvestamine graafilises kasutajaliideses nõuab **avatud projekti** (muidu on salvestusnupud blokeeritud):

* **Salvesta kõik / Peata kõik** — valgusandurite külgriba ülaosas; alustab või peatab `.daq` salvestamist kõigil ühendatud anduritel korraga.
* **Salvestamine / Salvestamise peatamine** — iga anduri kohta eraldi, hammasratta ikooniga seadete aknas. Salvestamise ajal kuvatakse anduri reaalajas teabe ridades punane „REC“-märk.

Failid salvestatakse `<project>/light_sensor/`-i ja kui salvestamine peatatakse — kas nupu „Peata”, „Peata kõik” abil või salvestavat andurit lahti ühendades — lisatakse valmis `.daq` **automaatselt avatud projektile**. See ilmub projekti faililoendisse ilma käsitsi lisamiseta, olles juba valmis peegeldusvõimsuse töötlemiseks.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Salvestamine CLI

kaudu

CLI salvestab tagapõhja andurite kogumi kaudu (tagapõhi peab töötama – need käsud on lihtsad HTTP-kliendid):

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

Hangi väärtus `--sensor-id` andurilt `chloros-cli daq pool-list`. Kaks teadmisväärset vaikimisi seadet:

| Valik | Vaikimisi |
| --- | --- |
| `--duration` | `0` — salvestab kuni `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` **tagapõhja** failisüsteemis, mitte CLI-s |

Väljundkataloogi eristamine on oluline, kui CLI suunab tagapõhja teisele masinale: fail salvestatakse sinna, kus tagapõhi töötab. Kui backend asub samal arvutil, siis suhteline `-o` (nagu eespool toodud `./out`) määratakse kindlaks selle kausta suhtes, kus käsk käivitati.

## Salvestamine Python

ist `DAQSensorSession` (mille tagastab `chloros_sdk.connect_daq_sensor()`) näitab sama koondatud salvestust: `record_start(output_dir=None, device_name=None)` tagastab faili tee, `record_stop()` tagastab `{path, rows}`. Täieliku seansi API kohta vaadake [SDK-viidet](../reference/sdk-reference.md). SDK otsese riistvaraga seotud klassid (ainult töölaua installatsioonid) kirjutavad salvestused vaikimisi `~/Documents/DAQ/`-i; väljalastud versioonide puhul on toetatud eespool nimetatud koondatud tee.

## .daq-faili kasutamine töötlemise ajal

Piltidest peegeldusvõimsuse arvutamiseks vajab Chloros iga ekspositsiooniga sobivat allapoole suunatud kiirgust:

* **Hoidke `.daq` koos piltidega.**Töötlemise ajal määrab töötlemisahel automaatselt**ajamärgistusega sobiva allapoole suunatud kiirguse** salvestatud `.daq`-failist (mis tahes DAQ-mudel) — või DAQ-M-i omast `.csv`-failist — mis asub piltide kõrval. GUI-salvestused vastavad sellele nõudele automaatselt, kuna need lisatakse projektile kohe, kui salvestamine lõpeb.
* **Kalibreerimisandmed laaditakse vajaduse korral.** Kui kaamera- või DAQ-põhist tehase kalibreerimispaketti pole veel kohalikult vahemällu salvestatud, laadib Chloros selle esmakordsel kasutamisel automaatselt MAPIR i pilvest alla (vajalik ühekordne internetiühendus; salvestatakse vahemällu failina `~/.chloros/`).
* **Reaalajas salvestused kirjutavad oma sidecar-faili.** Iga reaalajas salvestatud peegelduskaadri puhul salvestatakse tegelikult kasutatud DAQ-näit `.daq`-failina salvestiste kausta `daq/`, nii et salvestist saab hiljem uuesti töödelda ilma originaalsalvestuseta.
* **Salvestused salvestitest `chloros_scripts` (1.3.0).**Avalike [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) salvestite poolt loodud `.daq` on loetletud selle loonud DAQ-mudelina ning selle piirangut saab määrata**Projekti seaded → DAQ valgusandur** nagu iga teise salvestuse puhul. Need salvestid kirjutavad salvestusse anduri toorandmeid; Chloros laadib anduri tehase kalibreerimisandmed seerianumbri järgi alla ja rakendab neid.
* **Kelle mõõtmistulemus see on (1.3.0).**Iga salvestuse `.daq` kõrval on `.daq.cameras.json`, mis nimetab kaamera(id), mille kaadreid see näid kirjeldab, ning töötlemine seostab selle**ainult nende kaamerate piltidega**. Seetõttu ei kasuta iseseisev kaamera, millel puudub oma valgussensor, enam teise kaamera näitu – seostage sellega andur või salvestage ise näit. DAQ-iga endaga tehtud salvestus (GUI Record, `daq pool-record`) ja kõik `.daq`-failid, milles seda faili pole (sh vanemad salvestused), teenivad endiselt kõiki kaameraid.

## Kiirguse andmete taastamine

Projekti töötlemisel eksporditakse ka kõik selle sisalduvad valgusanduri salvestused kausta
`Light Sensor/` kausta pilditoodete kõrvale. Selleks **ei** ole vaja pilte:
eraldi salvestatud valgusandur on täielik salvestus ning kaust, mis sisaldab ainult `.daq`
faile, on kehtiv sisend. Projekt, mis sisaldab ainult valgusanduri salvestusi, loetleb need ja
töötleb neid nii rakenduses kui ka CLI-s, ning need jäävad loetelusse ka pärast töötlemist. Töötlemine
näitab, kui palju valgusanduri tooteid kirjutati.

| Toode | Mis see on |
| --- | --- |
| `<name>_calibrated.daq` | Uuesti töödeldav arhiiv, mis järgib sama skeemi kui reaalajas salvestus, kuid sisaldab nüüd selle loonud kalibreerimispaketti. Selle uuesti importimine **ei** kalibreeri seda teist korda. |
| `<name>_calibrated.csv` | Spektraalne kiirgustihedus ühikutes W/m²/nm anduri enda lainepikkuste võrgustikus, üks rida iga mõõtmistulemuse kohta, lisaks fotomeetrilised veerud: koguvõimsus, fotopiline ja skotopiline luks, PPFD koos sinise/rohelise/punase jaotusega ning tipp-lainepikkus. |

DAQ-U või DAQ-M, mille kalibreerimispaketti ei ole võimalik alla laadida — olete võrguühenduseta või
selle anduri kalibreerimisandmed puuduvad failist — **jätatakse põhjendusega vahele**, seda ei salvestata kunagi
„kalibreeritud” failina, mis sisaldab töötlemata loendusi. Ühenduge internetiga ja käivitage uuesti. DAQ-E
kannab oma kalibreerimist endas, seega vajab ta seda ainult siis, kui seade ei ole ühendatud ja
kohalikult pole midagi vahemällu salvestatud.

### DAQ-A: töötlemata loendid ja miks see on õige lahendus

**DAQ-A** on vanem kui seerianumbri-põhine kalibreerimispaketi süsteem ja sellel pole paketti, mida
alla laadida. See ei ole tähelepanematus: DAQ-A kalibreeritakse kohapeal
peegeldusmäära sihtmärgi abil ning sihtmärgipõhine kalibreerimine vajab ainult anduri *suhtelist*
vastust — mis ongi täpselt see, mida toores loend näitab. Chloros kalibreerib neid tänapäevalgi.

Seega eksporditakse DAQ-A salvestus, kuid teise nime all:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, mitte `_calibrated` – tegemist on erineva failinimega, mitte failisisesega märgistusega,
sest väide peab säilima ka siis, kui faili saadetakse edasi lihtsalt nimena. `.csv`
pealkiri näitab `raw spectral sensor counts (NOT irradiance)` ja hoiatab, et väärtusi saab
võrrelda **selle** faili piires, mitte erinevate andurite vahel. Veerud, mis on olulised
ainult tegeliku kiirguse puhul – koguvõimsus, luks, PPFD – jäetakse tühjaks, mitte
arvutatakse loenduste põhjal.

Vanemad DAQ-A-SD salvestused (skeem v1.01 / v1.02) salvestavad ainult faili kirjutamise aja, mitte
iga lugemise ajamärget. Chloros ei sobi pilte nendega kokku – kaadri sidumine
kirjutamisajaga oleks vale, kuigi see ei paistaks esmapilgul valena –, kuid eksport loeb neid hästi ja
CSV-failis on märgitud, millise kella järgi see on.

Täieliku peegeldusvõime ülevaate saamiseks – ühe anduriga koos kaameraga ja kahe anduriga (ümbrus/objekt) – vaata [Peegeldusvõime töövood](reflectance.md).
