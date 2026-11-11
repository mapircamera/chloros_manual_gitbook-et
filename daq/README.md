# DAQ valgusandurid

> **Otsite riistvara?**Andurid ise – mudelid, paigaldus, katted, pordid, toide ja SCANNER-rakendus – on kirjeldatud**[DAQ-kasutusjuhendis](https://mapir.gitbook.io/daq)**. Käesolevas peatükis käsitletakse nende kasutamist Chloroses.

MAPIRi **DAQ** valgussensorid mõõdavad ümbritsevat valgust radiomeetriliselt kalibreeritud spektritena. Chloroses täidavad need kahte ülesannet:

* **Iseseisev spektrimeeter** – reaalajas spektridiagrammid, kolorimeetrilised andmed ja `.daq` salvestused, kõik [Valgusandurite vahekaardilt](gui.md), [CLI-st](cli-quick-start.md) või Pythoni SDK-st.
* **Allapoole suunatud kiirguse allikas peegeldusvõime mõõtmiseks** — töötlemise käigus interpoleerib Chlorosi teie `.daq`-näidud iga pildi eksponeerimise ajamärgise järgi ja kasutab mõõdetud allapoole suunatud valgust, et teisendada kaamera kiirgust peegeldusvõimeks (`--reflectance-source daq`); kalibreeritud lainepikkuste puhul ei ole vaja stseeni-paneeli.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Kolm mudelit, üks andmevorming

| Mudel | Andmeedastus | Avastamine |
| --- | --- | --- |
| **DAQ-U** | USB (jadaport) | jadaporti skaneerimine |
| **DAQ-M** | Bluetooth Low Energy | BLE-skaneerimine nime järgi |
| **DAQ-E** | Ethernet (IPv4, PoE-toitega) | mDNS `_daq-e._tcp` (hostinimi `daq-e-<id>.local`) |**DAQ-E-S (uus versioonis 1.3.0).** DAQ-E, millel on püsiv hajuti ja kaldenurgaandur. Chloros loetleb seda nimetusega `DAQ-E-S` ja käitab seda täpselt nagu DAQ-E-d, välja arvatud see, et see ei vaja kattekorrektsiooni ja iga näiduga kaasneb seadme kaldenurk, pöörde- ja kallutusnurk. Vaata [DAQ-E-S: fikseeritud hajuti ja kaldenurk](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Kõik kolm kasutavad sama andmesideprotokolli ja edastavad identset andmestikku:

* **135-punktiline spekter vahemikus 340–1010 nm 5 nm sammuga**, lisaks CIE XYZ kolmestimuluse väärtused igas kaadris.
* **Radiomeetriliselt kalibreeritud spektraalne kiirgustihedus ühikutes W/m²/nm** — enne andmete teieni jõudmist rakendatakse iga seadme tehase kalibreerimispaketti (koos selle aktiivse kate-korrektsiooniprofiiliga).
* Sama **`.daq` salvestusformaat** (SQLite-fail). Järgnev töötlemine on identne sõltumata sellest, milline andmeedastusviis faili genereeris.

Andmeedastusviisid (USB-seriaal, BLE, mDNS/zeroconf) on koondatud Chlorosi tagapõhja — GUI-st või CLI-i `pool-*`-käskude kaudu suhtlemiseks ühegi kolmest mudelist ei ole vaja midagi installida.

***

## Kalibreeritud vahemik: aruandes 340–1010 nm, kalibreeritud ~374–974 nm

Andur edastab andmeid kogu 340–1010 nm vahemiku kohta, kuid NIST-i standarditele vastav radiomeetriline võimendus hõlmab ligikaudu **374–974 nm**. Chloroseerib absoluutse peegeldusvõime jagamise kõikide kaamerasagedusalade puhul, mille spektraalne kaal jääb kalibreeritud vahemikust alla poole; vahelejäänud sagedusala kuvatakse koos vahelejätmise põhjusega `dls-uncalibrated-band-<nm>`.

LATTICE&#x27;i müügil olevatest filtri tootenumbritest puudutab see ainult **F988**:

F988 peegeldusvõime on kalibreeritud stseeni-sisese peegeldusvõime paneeli abil: sagedusala jääb väljapoole DAQ-valgusanduri kalibreeritud vahemikku, seega kasutab Chloros teie viimast paneeli mõõtmistulemust ja säilitab selle paneeli mõõtmiste vahelisel ajal.

Kui F988 salvestust töödeldakse ainult DAQ-andmete põhjal, lükkab Chloros selle lainepikkuse puhul DAQ-põhise peegeldusvõime tagasi, märkides vahelejätmise põhjuseks `dls-uncalibrated-band-988` — [peegeldusplaadi töövoog](../calibration-targets.md) on F988 jaoks toetatud viis.

***

## Andurite ID-d

Iga DAQ edastab stabiilse anduri ID. Selle vorm erineb mudeli järgi:

| Mudel | ID vorm | Näide |
| --- | --- | --- |
| DAQ-U | 5-oktettiline, sidekriipsudega | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktettiline, sidekriipsudega | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Anduri ID on:

* sisse kirjutatud igasse `.daq` faili, mida see salvestab,
* võti, mida Chloros kasutab selle seadme tehase kalibreerimispaketi hankimiseks,
* väärtus, mille edastate `--sensor-id`-ile CLI-i `pool-*`-käskudes, ning
* DAQ-E puhul ka selle mDNS-hostinimi (`daq-e-def330.local`) – väärtus, mida `--eth-host` aktsepteerib.

***

## Tehase kalibreerimine ja pilv

Iga DAQ-seade on tehases individuaalselt kalibreeritud NIST-i jälgitava radiomeetrilise ahela abil ning Chloros laadib iga seadme kalibreerimispaketi, mis on seotud selle anduri ID-ga. Seadme kalibreerimisaruande (PDF) saab alla laadida anduri seadetest [Valgusandurite vahekaardilt](gui.md).

{% hint style="warning" %}
**DAQ-U ja DAQ-M vajavad kalibreerimiseks pilvejuurdepääsu.**Kumbki mudel ei salvesta midagi seadmesse: nende tehase kalibreerimispaketid asuvad MAPIRi pilves ja need laaditakse alla anduri ID järgi (seejärel salvestatakse need kohalikku vahemällu). Chloros vajab internetiühendust, et edastada kalibreeritud W/m²/nm andmeid DAQ-U või DAQ-M-ist.**DAQ-E on erand** — selle kalibreerimisandmed asuvad seadmes endas.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Kuhu salvestised jõuavad

| Pind | `.daq` vaikimisi sihtkoht |
| --- | --- |
| Graafiline kasutajaliides — vahekaart „Valgusandurid“ | `<project folder>/light_sensor/` (valmis salvestised lisatakse projektile automaatselt) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` tagapõhja käitaval arvutil (suhteline `-o` viitab kaustale, kus käsk käivitati, kui backend asub samal arvutil) |

Iga `.daq` failinimi sisaldab anduri ID-d ja ajamärget.

***

## Selles peatükis

* [**Chlorosi DAQ-vahekaart**](gui.md) — täielik GUI-juhend: iga mudeli ühendamine, anduripõhised seaded, spektridiagrammid, reaalajas kolorimeetrilised andmed, kahe anduri peegeldusvõime ja salvestamine.
* [**Kiirjuhend CLI-le (pool-\*)**](cli-quick-start.md) — DAQ-andurite juhtimine `chloros-cli daq pool-*` kaudu, toetatud käsurea tee.
* [**Piirprofiilid ja kalibreeritud vahemik**](caps-and-range.md) — millised piirid on olemas iga mudeli puhul, kuidas neid deklareerida ning kalibreeritud spektrivahemik üksikasjalikult.
* [**Salvestamine ja .daq-vorming**](recording.md) — `.daq` SQLite-vorming ja salvestamise töövood.
* [**DAQ-E võrgustik ja ajasünkroniseerimine**](ethernet-ptp.md) — DAQ-E andmeedastusrežiimid ja PTP-ajasünkroniseerimine.
* [**Peegeldusvõimsuse töövood**](reflectance.md) — DAQ allapoole suunatud andmete kasutamine peegeldusvõimsuse arvutamiseks.
* Täieliku liputaseme dokumentatsiooni leiate [CLI-viitest](../reference/cli-reference.md) (jaotis `chloros-cli daq`) ja [SDK-viitest](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), mis on mõlemad koostatud nii, et neid saaksid otse kasutada AI-assistendid.
