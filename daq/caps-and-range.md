# Katted ja kalibreeritud vahemik

> Katted ise – milline kate kuulub millise anduri juurde, kuidas need paigaldatakse ja millised on nende optilised omadused – on dokumenteeritud **[DAQ-kasutusjuhendis](https://mapir.gitbook.io/daq)**. Käesolev lehekülg käsitleb paigaldatud katte *deklarimist* Chloros
ele, mis tagab korrigeerimise õigsuse.

Iga DAQ-valgusanduri tehases läbi viidud radiomeetriline kalibreerimine kirjeldab *paljast* andurit. Difuusori peale paigaldatud füüsiline kate muudab seda, millist valgust andur kogub, seega rakendab Chloros
kalibreerimispaketi peale tehases mõõdetud **katte korrigeerimisprofiili**. Õige kate deklareerimine on osa kalibreeritud andmete saamisest – käesoleval lehel on kirjeldatud, millised katted on olemas iga mudeli puhul, kuidas neid deklareerida ja milline on anduri tegelik kalibreeritud spektriala.

## Katted mudelite kaupa

| Katteprofiil (`cap_id`) | Füüsiline kate | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Päikesepaiste koosinuskorrektoriga kate (**vaikimisi igal mudelil**) | Jah | Jah | Jah |
| `fov_15` / `fov_45` / `fov_90` | Vaatevälja piiravad koonused (15° / 45° / 90°) | Jah | — | Jah |
| `fov_30` / `fov_60` | Vaatevälja piiravad koonused (30° / 60°) | Jah | — | — |
| `none` | Kaaneta | — | — | Jah |

Mudelispetsiifilised märkused:

* **DAQ-M-il on üks kaaneprofiil: `sunshine_cosine`.** „Bare-plus-Sunshine-cap“ on selle toote määratlus ning paljas DAQ-M ei vaja geomeetriaprofiili.
* **Paljas DAQ-U on tõeliselt paljas** — see ei vaja üldse geomeetriaprofiili, mistõttu selle jaoks profiili `none` ei ole olemas.
* **`none` DAQ-E-l EI OLE tühikäik.** DAQ-E süvendatud, klaasiga kaetud hajutajal on oma tegelik geomeetriline korrektsioon, seega on „kate puudub“ selle mudeli puhul iseenesest mõõdetud profiil.
* **Paljas DAQ-E ei suuda mõõta otsest päikesevalgust ühelgi kõrgusel** — Sunshine-kate on välitingimustes kasutatav konfiguratsioon. Ärge planeerige välitöid palja DAQ-E-ga.

GUI anduripõhistes seadetes (hammasratta ikoon vahekaardil „Light Sensors”) pakub rippmenüü **Cap** ka valikut „None (bare sensor)” mudelitel DAQ-U ja DAQ-M — nendel kahel mudelil tähendab „bare” lihtsalt seda, et kattekorrektsiooni ei rakendata, vastavalt eespool toodud märkustele. Valige see ainult siis, kui kate on füüsiliselt eemaldatud.

## Kate deklareerimine — ja miks see on oluline

**Deklareeritud `cap_id` peab vastama anduril füüsiliselt olevale kattedele.** Ei andur ega tarkvara suuda paigaldatud kate tuvastada. Deklaratsioon mõjutab kahte asja:

1. Iga spektrile rakendatavat **reaalajas korrigeerimist**.
2. **Iga `.daq` salvestusse kirjutatud kate märget**, millele järgnev peegeldusvõime töötlemine tugineb.

Sunshine’i kate nõrgendab valgust **konstruktsiooni järgi umbes 12 korda**, seega vale kate deklareerides salvestatakse spektrid umbes sama teguriga valesti skaalutatuna. Deklarage kate muutused viivitamatult.

### Kate seadistamine

GUI: vahekaart „Light Sensors“ → hammasratta ikoon anduri real → rippmenüü **Cap**. Iga mudeli vaikimisi väärtus on `sunshine_cosine` – see on eeldus (DAQ-U, DAQ-M ja DAQ-E tarnitakse ilma kaaneta ning salvestus näitab, et kaant eeldati, kuni te ise ühe valite) ning valik säilib projekti raames.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(tagapõhi peab töötama):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI
aktsepteerib süntaktiliselt kogu `cap_id` nimekirja (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); iga profiili kehtivust kontrollitakse ühendamisel anduri mudeli alusel, seega annab kättesaamatu piirangu ID (näiteks ainult E-mudelile mõeldud ID DAQ-U-l) selge veateate, mitte ei korrigeeri valesti. Kui midagi ei edastata, on tagapõhja vaikimisi väärtus `sunshine_cosine`.

Python
SDK
: nimetage kap ühendamisel `connect_daq_sensor(cap_id="sunshine_cosine")`-ga või vahetage seda käitamise ajal `DAQSensorSession.set_cap("fov_45")`-ga; vaadake [SDK
-viidet](../reference/sdk-reference.md#daq-sensor-sessions).

Täpsem teave: profiilid on Chloros
i installis kaasas nimetusega `daq/cap_profiles/<u|m|e>/<cap_id>.json` ja neid saab kasutaja kaupa ümber kirjutada nimetusega `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Lisaks piirangutele saavad andurid, mida pole kunagi uuesti kalibreeritud, automaatselt väikese, seadmepargist tuletatud tumeoffseti täpsustuse – kasutaja sekkumist ei ole vaja.

## DAQ-E-S: fikseeritud hajuti ja kaldenurk

**DAQ-E-S** on DAQ-E, millel on kaks Chloros
e jaoks olulist erinevust. Kõik muu (avastamine, ühendamine, PTP, salvestamine, seadme seaded) toimib täpselt samamoodi kui DAQ-E puhul.

* **Selle hajuti on püsiv, seega ei vaja see kate-korrektsiooni.** Hajuti oli seadmel juba siis, kui mõõdeti selle tehase kalibreerimist, seega näitab kalibreerimine juba tõelist allapoole suunatud kiirgustihedust. Chloros
ei rakenda sellele mingit piirmäära profiili, olenemata sellest, mida palutakse: selle jaoks nimetatud piirmäär ignoreeritakse koos teatega, selle salvestused näitavad, et piirmäär pärineb mudelist, ning eeldatava piirmäära hoiatus selle puhul kunagi ei käivitu. Projektiseadetes → DAQ valgusandur ei näita selle rida ühtegi piirmäära ning kogu projekti hõlmav piirmäära ümberkirjutamine seda ei mõjuta. Ärge rakendage andmetele käsitsi piirmäära: päikesepaiste korrigeerimine annaks tulemuseks umbes 11 korda liiga kõrge väärtuse.
* **Seadmel on 3-teljeline kiirendusmõõtur.** Hajuti koosinuskarakteristik kehtib ainult siis, kui see on horisontaalselt, seega näitab kallutatud andur valet kiirgustaset ilma muude sümptomiteta. Seetõttu sisaldab iga DAQ-E-S-i näidud selle kallutust, pööramist ja kaldenurka ning ühest seadmest salvestatud `.daq` salvestab need iga spektri kõrvale (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), nii et saate eristada pilve ja kõigutatud alusest.

Iga asendi väljendi ja asendiandmevoo vea teatamise viisi kohta vaadake [DAQ-E-S CLI
-viite](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Sunshine capi jõudlus (välistingimustes kasutamine)

Arvud, mille alusel saab protseduure koostada:

| Omadus | Väärtus |
| --- | --- |
| Vaateväli | 180° poolkera |
| Kosinusvastuse viga | ≤ ±4 % kuni 60° langemisnurgani; ≤ ±4,5 % kuni 70° |
| Madala päikese asendi piir | Ei soovitata alla ~15° päikese kõrguse |
| Nõrgenemine | ~12× (konstruktsioonist tulenevalt) |
| Katte uuesti paigaldamise korratavus | ≈ 1,5 % |
| Kvantitatiivne kiirgustihedus | Keskmine **≥ 15 s** mõõtmistulemuste keskmine (seadme omadus, mitte defekt) |

Kõigi kvantitatiivsete kiirgustiheduse väärtuste puhul – sealhulgas peegeldusvõime viited – kasutage ühe kaadri asemel vähemalt 15 sekundi mõõtmistulemuste keskmist.

## Kalibreeritud spektrivahemik

| Omadus | Väärtus |
| --- | --- |
| Spektraalne proovivõtt | 340–1010 nm 5 nm sammuga (135 punkti) |
| Radiomeetriliselt kalibreeritud vahemik | **~374–974 nm** (tarkvara poolt kehtestatud) |

Andur edastab kogu 340–1010 nm võrgustiku, kuid NIST-ile viitav radiomeetriline võimendus hõlmab vahemikku ~374–974 nm. Chloros
**keeldub absoluutse peegeldusvõime jagamisest** mis tahes kaamera lainepikkuse puhul, mille spektraalne kaal jääb alla poole nimetatud vahemikust, ning edastab vahelejätmise põhjuse `dls-uncalibrated-band-<nm>`, selle asemel et toota kalibreerimata tulemust. Müügil olevatest kaameramudelite hulgast jääb ainult F988 filter sellest vahemikust välja; selle puhul kasutatakse selle asemel peegelduspaneli töövoogu – vt [Peegeldusvoogud](reflectance.md).

Andurimudelite, andurite tüüpide ja andurite ID-de kohta vaata [DAQ-ülevaadet](README.md). Selle kohta, kuidas töötlemise käigus kasutatakse piirmäära, vaata [Salvestamine ja .daq-vorming](recording.md).
