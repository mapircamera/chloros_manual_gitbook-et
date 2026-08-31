# DAQ valgusandurid

> **Otsite riistvara?**Andurid ise – mudelid, paigaldus, katted, pordid, toide ja SCANNER-rakendus – on kirjeldatud**[DAQ-kasutusjuhendis](https://mapir.gitbook.io/daq)**. Käesolevas peatükis käsitletakse nende kasutamist alates mudelist Chloros.

MAPIRi **DAQ** valgussensorid mõõdavad ümbritsevat valgust radiomeetriliselt kalibreeritud spektritena. Chlorosis täidavad need kahte ülesannet:

* **Iseseisev spektraalmõõteriist** — reaalajas spektridiagrammid, kolorimeetrilised andmed ja `.daq` salvestused, kõik [Valgusandurite vahekaardilt](gui.md), [CLI](cli-quick-start.md) või Python SDK kaudu.
* **Allapoole suunatud kiirguse allikas peegeldusvõime arvutamiseks** — töötlemise käigus interpolab Chloros teie `.daq` mõõtmistulemused iga pildieksponeerimise ajamärgile ning kasutab mõõdetud allapoole suunatud valgust kaamera kiirguse teisendamiseks peegeldusvõimeks (`--reflectance-source daq`); kalibreeritud lainealade puhul ei ole vaja stseeni sees asuvat paneeli.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Kolm mudelit, üks andmevorming

| Mudel | Andmeedastus | Avastamine |
| --- | --- | --- |
| **DAQ-U** | USB (jadaport) | jadapordi skaneerimine |
| **DAQ-M** | Bluetooth Low Energy | BLE-skaneerimine nime järgi |
| **DAQ-E** | Ethernet (IPv4, PoE-toitega) | mDNS `_daq-e._tcp` (hostinimi `daq-e-<id>.local`) |

Kõik kolm kasutavad sama andmesideprotokolli ja edastavad identseid andmeid:

* **135-punktiline spekter vahemikus 340–1010 nm 5 nm sammuga**, lisaks CIE XYZ kolmestimuluse väärtused igas kaadris.
* **Radiomeetriliselt kalibreeritud spektraalne kiirgustihedus ühikutes W/m²/nm** — enne andmete jõudmist teieni rakendatakse iga seadme tehase kalibreerimispaketti (koos selle aktiivse kattekorrektsiooniprofiiliga).
* Sama **`.daq` salvestusformaat** (SQLite-fail). Järgnev töötlemine on identne, sõltumata sellest, milline andmeedastusviis faili genereeris.

Transpordipakid (USB-seriaal, BLE, mDNS/zeroconf) on koondatud Chloros-i tagapõhja – GUI-st või CLI-i `pool-*`-käskude kaudu suhtlemiseks ei ole vaja midagi installida.

***

## Kalibreeritud vahemik: aruandes 340–1010 nm, kalibreeritud ~374–974 nm

Andur edastab andmeid kogu 340–1010 nm vahemiku kohta, kuid NIST-i standarditele vastav radiomeetriline võimendus hõlmab ligikaudu **374–974 nm**. Chloros keeldub absoluutse peegeldusvõime jagamisest mis tahes kaamerasagedusriba puhul, mille spektraalne kaal jääb kalibreeritud vahemikust alla poole; vahelejäänud sagedusriba kuvatakse koos vahelejätmise põhjusega `dls-uncalibrated-band-<nm>`.

Müügil olevatest LATTICE-filtrite tootenumbritest puudutab see ainult **F988**:

F988 peegeldusvõime on kalibreeritud stseeni-sisese peegeldusvõime paneeli abil: sagedusala jääb väljapoole DAQ-valgusanduri kalibreeritud vahemikku, mistõttu Chloros kasutab teie viimast paneeli salvestust ja säilitab seda paneeli vaatluste vahel.

Kui F988 salvestust töödeldakse ainult DAQ-andmete alusel, lükkab Chloros selle lainepikkuse puhul DAQ-põhise peegeldusvõime tagasi vahelejätmise põhjusega `dls-uncalibrated-band-988` — [peegelduspaneeli töövoog](../calibration-targets.md) on F988 jaoks toetatud protsess.

***

## Andurite ID-d

Iga DAQ edastab stabiilse anduri ID. Selle vorm erineb mudelite lõikes:

| Mudel | ID vorm | Näide |
| --- | --- | --- |
| DAQ-U | 5-oktettiline, sidekriipsudega | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktettiline, sidekriipsudega | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Anduri ID on:

* sisse kirjutatud igasse `.daq` faili, mida see salvestab,
* võti, mida Chloros kasutab selle seadme tehase kalibreerimispaketi hankimiseks,
* väärtus, mille edastate `--sensor-id`-ile käskudes CLI ja `pool-*`, ning
* DAQ-E puhul ka selle mDNS-hostinimi (`daq-e-def330.local`) – väärtus, mida `--eth-host` aktsepteerib.

***

## Tehase kalibreerimine ja pilv

Iga DAQ-seade on tehases individuaalselt kalibreeritud NIST-i standarditele vastava radiomeetrilise ahelaga ning Chloros laadib iga seadme kalibreerimispaketi, mis on seotud selle anduri ID-ga. Seadme kalibreerimisaruande (PDF) saab alla laadida anduri seadetest [vahekaardil „Valgusandurid”](gui.md).

{% hint style="warning" %}
**DAQ-U ja DAQ-M vajavad kalibreerimiseks pilvejuurdepääsu.**Kumbki mudel ei salvesta midagi seadmesse: nende tehase kalibreerimispaketid asuvad MAPIR pilves ja need laaditakse alla anduri ID järgi (seejärel salvestatakse need kohalikku vahemällu). Chloros vajab internetiühendust, et edastada kalibreeritud W/m²/nm andmeid DAQ-U-st või DAQ-M-st.**DAQ-E on erand** — selle kalibreerimisandmed asuvad seadmes endas.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Kuhu salvestused salvestatakse

| Pind | `.daq` vaikimisi sihtkoht |
| --- | --- |
| Kasutajaliides — vahekaart „Valgusandurid“ | `<project folder>/light_sensor/` (valmis salvestused lisatakse projektile automaatselt) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` tagapõhja käitaval arvutil |

Iga `.daq` failinimi sisaldab anduri ID-d ja ajamärget.

***

## Käesolevas peatükis

* [**Chloros-i vahekaart „DAQ”**](gui.md) — täielik juhend graafilise kasutajaliidese kohta: iga mudeli ühendamine, anduripõhised seaded, spektridiagrammid, reaalajas kolorimeetrilised andmed, kahe anduri peegeldusvõime ja salvestamine.
* [**CLI kiirstart (pool-\*)**](cli-quick-start.md) — DAQ-andurite juhtimine `chloros-cli daq pool-*`-i kaudu, toetatud käsurea tee.
* [**Piirprofiilid ja kalibreeritud vahemik**](caps-and-range.md) — millised piirid on olemas iga mudeli puhul, kuidas neid deklareerida ning kalibreeritud spektrivahemik üksikasjalikult.
* [**Salvestamine ja .daq-vorming**](recording.md) — `.daq` SQLite-vorming ja salvestamise töövood.
* [**DAQ-E võrgustik ja ajasünkroniseerimine**](ethernet-ptp.md) — DAQ-E edastusrežiimid ja PTP ajasünkroniseerimine.
* [**Peegeldusvõimsuse töövood**](reflectance.md) — DAQ allapoole suunatud andmete kasutamine peegeldusvõimsuse arvutamiseks.
* Täieliku liputase dokumentatsiooni leiate [CLI viite](../reference/cli-reference.md) (jaotis `chloros-cli daq`) ja [SDK viite](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), mis on mõlemad koostatud nii, et neid saaksid otse kasutada AI-abilised.
