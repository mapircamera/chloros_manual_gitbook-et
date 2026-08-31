# Kaaneprofiilid ja kalibreeritud vahemik

> Kaaned ise – milline kaan sobib millise anduriga, kuidas neid paigaldatakse ja millised on nende optilised omadused – on kirjeldatud **[DAQ-kasutusjuhendis](https://mapir.gitbook.io/daq)**. Käesoleval lehel käsitletakse paigaldatud kate *deklarimist* Chloros-ile, mis tagab korrektsiooni õigsuse.

Iga DAQ-valgusanduri tehases läbi viidud radiomeetriline kalibreerimine kirjeldab *paljast* andurit. Difuusori peale paigaldatud füüsiline kate muudab seda, millist valgust andur kogub, seega rakendab Chloros kalibreerimispaketi peale tehases mõõdetud **katte korrektsiooniprofiili**. Õige kate deklareerimine on osa kalibreeritud andmete saamisest – käesoleval lehel käsitletakse, millised katted on olemas iga mudeli puhul, kuidas neid deklareerida ja milline on anduri tegelik kalibreeritud spektriala.

## Katted mudelite kaupa

| Katte profiil (`cap_id`) | Füüsiline kate | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Päikesekiirguse koosinuskorrektsioonikate (**vaikimisi igal mudelil**) | Jah | Jah | Jah |
| `fov_15` / `fov_45` / `fov_90` | Vaatevälja piiravad koonused (15° / 45° / 90°) | Jah | — | Jah |
| `fov_30` / `fov_60` | Vaatevälja piiravad koonused (30° / 60°) | Jah | — | — |
| `none` | Kaaneta | — | — | Jah |

Mudelispetsiifilised märkused:

* **DAQ-M-il on üks kaaneprofiil: `sunshine_cosine`.** „Bare-plus-Sunshine-cap” on selle toote määratlus ning paljas DAQ-M ei vaja geomeetriaprofiili.
* **Paljas DAQ-U on tõeliselt paljas** — see ei vaja üldse geomeetriaprofiili, mistõttu selle jaoks profiili `none` ei ole olemas.
* **`none` DAQ-E-l EI OLE tühikäik.** DAQ-E süvendatud, klaasiga kaetud hajutajal on oma tegelik geomeetriline korrektsioon, seega on „kate puudub“ selle mudeli puhul iseenesest mõõdetud profiil.
* **Paljas DAQ-E ei suuda mõõta otsest päikesevalgust ühelgi kõrgusel** — Sunshine-kate on välitingimustes kasutatav konfiguratsioon. Ärge planeerige välitöid palja DAQ-E-ga.

GUI anduripõhistes seadetes (hammasratta ikoon vahekaardil „Light Sensors”) pakub rippmenüü **Cap** ka valikut „None (bare sensor)” mudelitel DAQ-U ja DAQ-M — nendel kahel mudelil tähendab „bare” lihtsalt seda, et kattekorrektsiooni ei rakendata, vastavalt eespool toodud märkustele. Valige see ainult siis, kui kate on füüsiliselt eemaldatud.

## Kate deklareerimine — ja miks see on oluline

**Deklareeritud `cap_id` peab vastama anduril füüsiliselt olevale kattedele.** Ei andur ega tarkvara suuda paigaldatud kaanet tuvastada. Deklaratsioon mõjutab kahte asja:

1. Iga spektrile rakendatavat **reaalajas korrigeerimist**.
2. **Iga `.daq` salvestusse kirjutatud kaanemärget**, millele järgnev peegeldusvõime töötlemine tugineb.

Sunshine’i kate nõrgendab signaali **disaini järgi umbes 12×**, seega vale katte deklareerimine moonutab spektrite skaalat umbes sama teguri võrra. Deklarage kattevahetus kohe.

### Katte seadistamine

GUI: vahekaart „Light Sensors“ → hammasrattaikoon anduri real → rippmenüü **Cap**. Iga mudeli vaikimisi seadistus on `sunshine_cosine` (kõik DAQ-andurid tarnitakse koos paigaldatud koosinuskorrektoriga) ning valik säilib projekti raames.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (tagapõhi peab töötama):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI aktsepteerib süntaktiliselt kogu `cap_id` nimekirja (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); iga profiili kehtivust kontrollitakse ühendamisel anduri mudeli alusel, seega annab kättesaamatu kapseli ID (näiteks ainult E-tüüpi ID seadmel DAQ-U) selge veateate, mitte ei korrigeeri valesti. Kui midagi ei edastata, on tagapõhja vaikimisi väärtus `sunshine_cosine`.

Python SDK märkus: `cap_id` **ei** ole SDK-nupp — `connect_daq_sensor()` / `DAQSensorSession` ei avalda ühtegi kap-parameetrit. Valige piirang eespool toodud CLI-käskude või graafilise kasutajaliidese rippmenüü kaudu; vt [SDK-viidet](../reference/sdk-reference.md).

Täpsem teave: profiilid on kaasas Chloros installis asukohas `daq/cap_profiles/<u|m|e>/<cap_id>.json` ning neid saab kasutaja kaupa ümber kirjutada asukohas `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Eraldi piirangutest saavad andurid, mida pole kunagi uuesti kalibreeritud, automaatselt väikese, sõidukipargist tuletatud tumeda nihke täpsustuse — kasutaja sekkumist ei ole vaja.

## Päikesekiirguse piirangu toimivus (välistingimuste konfiguratsioon)

Arvud, mille põhjal saate protseduure koostada:

| Omadus | Väärtus |
| --- | --- |
| Vaateväli | 180° poolkera |
| Kosinusreaktsiooni viga | ≤ ±4 % kuni 60° langemisnurgani; ≤ ±4,5 % kuni 70° |
| Madala päikese asendi piir | Ei soovitata alla ~15° päikese kõrguse |
| Sumbumine | ~12× (konstruktsioonist tulenevalt) |
| Kaitsekilbi uuesti paigaldamise korratavus | ≈ 1,5 % |
| Kvantitatiivne kiirgustihedus | Keskmine **≥ 15 s** mõõtmistulemuste keskmine (seadme omadus, mitte defekt) |

Kõigi kvantitatiivsete kiirgustiheduse väärtuste puhul – sealhulgas peegeldusvõime viited – kasutage ühe kaadri asemel vähemalt 15 sekundi mõõtmistulemuste keskmist.

## Kalibreeritud spektrivahemik

| Omadus | Väärtus |
| --- | --- |
| Spektraalne proovivõtt | 340–1010 nm 5 nm sammuga (135 punkti) |
| Radiomeetriliselt kalibreeritud vahemik | **~374–974 nm** (tarkvara poolt kehtestatud) |

Andur edastab kogu 340–1010 nm võrgustiku, kuid NIST-ile viitav radiomeetriline võimendus hõlmab vahemikku ~374–974 nm. Chloros **keeldub absoluutpeegeldusvõimsuse jagamisest** kõigi kaamerasagedusalade puhul, mille spektraalne kaal jääb alla poole nimetatud vahemikust, ning esitab vahelejätmise põhjuse `dls-uncalibrated-band-<nm>`, selle asemel et luua kalibreerimata tulemust. Müügil olevatest kaameramudelite hulgast jääb ainult F988-filter sellest vahemikust välja; selle puhul kasutatakse selle asemel peegelduspaneli töövoogu – vt [Peegelduspaneli töövood](reflectance.md).

Andurimudelite, andurite tüüpide ja andurite ID-de kohta vaata [DAQ-ülevaadet](README.md). Selle kohta, kuidas töötlemise käigus kasutatakse piirmäära, vaata [Salvestamine ja .daq-vorming](recording.md).
