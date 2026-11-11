# Vahekaart „DAQ” programmis Chloros

Vahekaart „DAQ” — mis Chlorosi külgribal kannab nimetust **Valgusandurid** — on [DAQ-U, DAQ-M ja DAQ-E valgusandurite](README.md) reaalajas juhtpaneel: ühendage andureid mis tahes andmeedastusprotokolli kaudu, jälgige kalibreeritud spektreid reaalajas, arvutage anduripaari abil reaalajas peegeldusvõimet ning salvestage `.daq`-failid otse oma projekti.

Vahekaart muutub kättesaadavaks, kui Chlorosi taustsüsteem on käivitumise lõpetanud. Vahekaardi graafikuid varustab Chlorosi DAQ-teenus reaalajas ühenduse kaudu, mis katkestuse korral ühendub automaatselt uuesti (2–10 sekundi viivitusega); kui teenus on kättesaamatu, kuvatakse anduri staatuse real **No Server**.

Paigutus koosneb **andurite külgribast**(üks rida iga ühendatud anduri kohta) ja**diagrammialast** (üks diagrammiplaat iga anduri või rühma kohta).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Anduri ühendamine

Klõpsake külgriba ülaosas nuppu **Ühenda andur**. Ühendamise dialoogaken avaneb põhivaates (või teise anduri lisamisel ülekatteaknana – sel juhul ilmub nupp „Tühista”).

| Kontroll | Käitumine |
| --- | --- |
| **Seadme tüüp** | `DAQ-U (USB)` (vaikimisi), `DAQ-M (Bluetooth)` või `DAQ-E (Ethernet)`. Valiku muutmine käivitab uuesti skannimise äsja valitud ülekandeviisi jaoks. |
| **Port / BLE-seade / Hostinimi / IP** | Näitab avastatud seadmeid kujul `device - description`; esimene andurina tuvastatud kirje valitakse automaatselt. Skaneerimise ajal kuvatakse `Scanning...` (USB), `Scanning (N)...` koos 8-sekundilise tagasilugemisega (BLE) või `Discovering ethernet sensors (N)...` koos 5-sekundilise tagasilugemisega (Ethernet). Tühjade tulemuste korral kuvatakse `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Värskenda** | Skaneerib valitud ühendusviisi kohe uuesti (BLE/Etherneti skaneerimise ajal on see funktsioon keelatud). |
| **Ühenda andur** | Saab aktiveerida pärast seadme valimist; ühenduse loomise ajal muutub märge `Connecting...`-iks. |

Otsing töötab ainult **siis, kui ühendusdialoog on ekraanil**, ja kordub iga 15 sekundi järel ainult valitud ühendusviisi puhul — lihtsalt vahekaardi avamine ei käivita skaneerimist. Ebaõnnestumise korral kuvatakse dialoogis: *„Ühendamine ebaõnnestus. Proovi andurit lahti ühendada ja uuesti ühendada, seejärel klõpsa uuesti nupule „Ühenda”.”*

Külgriba avaneb automaatselt, kui esimene andur ühendub.

{% hint style="info" %}
**DAQ-E ei kuvata?**DAQ-E-l puudub oleku-LED — kontrollige PoE/ühenduse indikaatorit lülitil või injektori pordil, kuhu see on ühendatud, ning oodake pärast sisselülitamist mõni sekund, kuni seade käivitub. Chloros-seade peab asuma samas levidomeenis (mDNS ei läbi ruutereid). Paigaldaja**„Chloros Backend (UDP-In)“** tulemüüri reegel lubab läbi mDNS-i (UDP 5353), DAQ-E andmeid (UDP 5002) ja PTP-d (UDP 319/320); kolmanda osapoole tulemüüris peavad samad pordid olema avatud. Kaks DAQ-E seadet ühes LAN-võrgus tuvastatakse eraldi, kumbki oma `daq-e-<id>.local` hostinimega.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Seadmetüüp pakub valikuid DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet)</figcaption></figure>

***

## Andurite külgriba

Iga ühendatud andur saab oma rea (pluss üks rida iga „Ambient+Object“ rühma kohta). Ridu saab lohistades ümber järjestada ning nende järjekord määrab ka graafikupaneelide järjestuse. Klõpsake reale, et muuta see andur/rühm loendivaates aktiivseks graafikuks.

| Element | Tähendus |
| --- | --- |
| Värviline vasak serv | Anduri graafiku värv. |
| Transpordimärk | `DAQ-U` / `DAQ-M` / `DAQ-E` või roheline märk `REF` „Ambient+Object“ peegeldusgrupi puhul. |
| Seadme nimi | Vaikimisi on see anduri seerianumber (selle püsiv identiteet kalibreerimiseks, `.daq` failinimede jaoks ning impordi sobitamiseks); kohandatud nimed säilivad projekti kaupa. |
| **Kalibreeritud** märge (roheline) | Kuvatakse, kui anduri tehase kalibreerimispakett on laaditud, st spektrid on tõelised W/m²/nm. |
| **Uuendus saadaval** (merevaik, ainult DAQ-E) | Kasutusel olev püsivara on vanem kui selle Chloros

-koostisega kaasas olev pilt. Uuendamise ajal kuvatakse reaalajas edenemist (`Flashing… N%`, `Restarting sensor…`, seejärel `Updated X → Y` või `Failed`). |
| Silm | Lülitab selle anduri nähtavuse graafikul sisse või välja. |
| Hammasratas | Avab anduri seadete akna (allpool). |
| ✕ (punane) | Lülitab anduri lahti või eemaldab „Ambient+Object“ rühma. |

Ridade kohal asuvad kaks nuppu:

* **Ühenda andur** — avab ühendusdialoogi (muutub `Connecting...`-iks, kui on hõivatud).
* **Salvesta kõik / Peata kõik**— alustab või peatab `.daq` salvestamist**kõigil**ühendatud anduritel. Nõuab vähemalt ühte andurit**ja avatud projekti** (tööriista vihje: „Ava projekt salvestamiseks“); muutub punaseks, kui mõni salvestus on käimas.

Tühjas olekus kuvatakse tekst „Ühtegi andurit pole ühendatud“.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Anduripõhised seaded (hammasratta-aknas)

Avage hammasratta ikooniga anduri real. Sisu järjekorras:

* **Info read** — Seadme tüüp (DAQ-U/M/E), Ühendus (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM-port, BLE-aadress või host) ja seerianumber.
* **Kalibreerimisaruanne: alla laadida** — laadib alla selle seadme NIST-jälgitava kalibreerimissertifikaadi (PDF) ja avab selle teie PDF-vaatajas. Saadaval, kui seerianumber on teada; sertifikaat salvestatakse vahemällu esimesel ühendamisel.
* **Seadme nimi** — klõpsa pliiatsi ikoonil, et nime muuta; säilib projekti kaupa.
* **Graafiku joone värv** — värvivalik; säilib projekti kaupa.
* **Integreerimisaeg (ms)**— liugur + number,**1–500 ms**, vaikimisi**32 ms**. Keelatud, kui AE on sisse lülitatud.
* **Kaadri keskmine**— liugur + number,**1–50 kaadrit**, vaikimisi**20**.
* **AE: sisse/välja**— automaatse särituse lüliti;**ühendamisel vaikimisi sisse**. Lülita see välja, et seadistada integratsiooniaega käsitsi.
* **Peata voog / Alusta voog** — peatab või jätkab otseülekannet.
* **Salvestamine / Salvestamise lõpetamine** — salvestamine anduri `.daq` kaupa (vajab avatud projekti).
* **Cap** — cap-korrektsiooni profiil (järgmine jaotis).
* **Reaalajas teabe read** — integratsiooniaeg (ms), kaadrisagedus (FPS), proovid, salvestamine (punane `REC` või `Off`) ja staatus (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Ainult DAQ-E: võrgu-, püsivara- ja PTP-read

* **Hostinimi / IP** — seadme praegune aadress.
* **Püsivara** — praegune püsivara versioon, millele lisandub tegevuslahter:<version\>

nupp</version\>

**Uuenda versioonile \<version\>** ilmub, kui see Chloros

i versioon sisaldab uuemat DAQ-E püsivara kujutist. Uuendus laaditakse võrgu kaudu umbes 30 sekundiga; andur taaskäivitub ja ühendub automaatselt uuesti, ning katkestatud ülekande korral jääb praegune püsivara muutumatuks. Progressiandmed edastatakse reaalajas (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`) ning lahtris kuvatakse `Up to date`, kui see on hetkeolukord.
* **PTP-sünkroniseerimine** — reaalajas PTP-olek (langeb tagasi väärtusele `unknown`). DAQ-E püsivara versioon 1.2.0+ osaleb IEEE 1588 PTPv2-s ainult orja-kellana; Chloros

i hosti backend on PTP grandmaster ning iga DAQ-E ja LATTICE kaamera LAN-võrgus on selle orjaks domeenis 0, hoides ajamärke ligikaudu 1 ms piires.
* **Võrk**— seadme aadress ja selle saamise viis (DHCP, link-local või staatiline) ning salvestatud seade, mis kehtib pärast taaskäivitamist.**Muuda…**avab vormi:**Automaatne (DHCP)**või**Staatiline IP**koos IP-aadressi, alamvõrgu maskiga ja valikulise väravaga.**Kohalda ja taaskäivita** salvestab seaded, taaskäivitab anduri ja ühendab selle uuesti. Selle arvuti võrgust väljaspool asuv staatiline aadress lükatakse tagasi, kuna Chloros

ei suutnud pärast taaskäivitamist anduriga ühendust saada; seejärel ilmub kasti, mis lubab seda siiski teha, kui arvuti aadress muudetakse või suunatakse sellesse võrku. **Ülekande alustamine sisselülitamisel**määrab, kas andur alustab ülekannet kohe pärast sisselülitamist, enne kui ükski tarkvara ühendust loob.**Restart sensor** taaskäivitab anduri ja ühendab selle uuesti praeguste seadistustega. Taaskäivitamine ei ole võimalik, kui andur salvestab andmeid. Uus versioonis 1.3.0.

Ambient+Object-grupi puhul kuvab hammasratta-aknas ainult grupi allikandurid, seadme nime ja graafiku joone värvi.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Katte valik

Rippmenüü **Kate** näitab Chloros

ele, milline füüsiline kate on paigaldatud anduri hajuti peale, ning rakendab selle katte tehases mõõdetud korrigeerimisprofiili igale spektrile. Valikud sõltuvad mudelist:

| Mudel | Katevalikud |
| --- | --- |
| DAQ-U | Puudub (paljas andur), vaateväli 15°, vaateväli 30°, vaateväli 45°, vaateväli 60°, vaateväli 90°, Sunshine (koosinuskorrektor) |
| DAQ-M | Puudub (paljas andur), Sunshine (kosinuskorrektor) |
| DAQ-E | Puudub (paljas andur), vaateväli 15°, vaateväli 45°, vaateväli 90°, Sunshine (kosinuskorrektor) |

**Kõigi mudelite vaikimisi seadistus on „Sunshine“ (kosinuskorrektor)** — MAPIR

tarnib iga DAQ-seadme koos paigaldatud „Sunshine“-kapsliga ning see on standardne välistingimustes kasutatav konfiguratsioon: 180° poolkera vaateväli koos kosinusveaga ≤ ±4 % kuni 60° ja ≤ ±4,5 % kuni 70° (ei soovitata päikese kõrguse all ~15°), konstruktsioonist tulenev summutamine (~12×). Teie valik jääb projektis püsima.

{% hint style="warning" %}
**Katte valik peab vastama füüsilisele katele.**Ei andur ega tarkvara suuda tuvastada, milline kate on paigaldatud. Valik mõjutab nii reaalajas korrigeerimist kui ka igasse `.daq`-faili kirjutatud märget — Sunshine-katte ~12× nõrgenduse tõttu muudab deklareerimata kate vahetus spektrit umbes sama teguriga valesti. (Sama katte eemaldamine ja uuesti paigaldamine kordab seda umbes 1,5 % ulatuses.) Valige**„None“ (paljas andur)** ainult siis, kui kate on füüsiliselt eemaldatud; DAQ-E puhul rakendab valik „None“ ikkagi tehase geomeetriaprofiili selle süvendatud klaasdiffuusori jaoks – see ei ole tühine toiming – ning paljas DAQ-E on laborikonfiguratsioon, mitte toetatud välitingimuste konfiguratsioon.
{% endhint %}

{% hint style="info" %}
Uuendamine varasemast kasutusjuhendist: brauseri poolne lüliti „Sunshine Diffuser Installed” versioonist 1.1.0 on kadunud. Kaanega tegelemine toimub nüüd anduri kaupa kaaneprofiili abil, mida rakendatakse serveri poolel.
{% endhint %}

***

## Graafikuala

Kinnitatud ülemine riba sisaldab **nimekirja ⇄ ruudustiku vaate lülitit**ja**diagrammi suumimise** liugurit (ruudu suurus 200–2000 px). Vaade lülitub automaatselt ruudustikule, kui on olemas rohkem kui üks diagrammirühm, ja tagasi nimekirjale, kui neid on üks või vähem. Vaaterežiim ja graafiku suurus jäävad projekti kaupa püsima.

Iga anduri **spektrigraafik** näitab:

* **X-telg** — lainepikkus (nm). Andurite võrgustik on 340–1010 nm 5 nm sammuga (135 punkti), mis kuvamiseks interpoleeritakse 1 nm-ni.
* **Y-telg**— kalibreeritud anduri puhul (roheline**Kalibreeritud**märge),**võimsus (W/m²/nm)**— radiomeetriliselt kalibreeritud spektraalne kiirgustihedus kõigil kolmel kandjal — koos automaatse SI-eesliitega (m/µ/n), mis valitakse tippväärtuse põhjal. Andur, mille kalibreerimispakett pole laaditud, kuvab oma töötlemata näidud**intensiivsusena (loendid)**, skaalimata. Kui graafik kuvab mõlemat tüüpi andmeid, on võimsus vasakul teljel ja loendid paremal teljel.
* Ühe joone all on vikerkaarevärviline spektraalne täitmine; mitu andurit ühel graafikul kuvatakse värviliste joontena, mille täitmine on tuhmendatud.
* **Hiirega üle libistamine**— vertikaalne kursor, millel on näidatud lainepikkus ja anduri väärtus (kalibreeritud anduri puhul W/m²/nm, muudel juhtudel koguarvud);**lohistamine** suurendamiseks (suurendatud vaates ilmub väiksendamise nupp).
* **+**-nupp (ainult võrgustikuvaates), et lisada andur sellele graafikule või luua rühm (allpool).
* Seadme nimi on ülaosas keskel ning esimesele kaadrile jõudmiseni kuvatakse pöörlev ikoon.

**Küllastus** ei ole graafikul endal märgitud: küllastunud andur kuvab punast `SATURATED` staatuseteksti ja punast `Saturated: Yes` rida reaalajas andmete tabelis. Selle tühistamiseks vähendage integratsiooniaega või lülitage AE uuesti sisse.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Reaalaja andmetabel (loendivaade)

Diagrammi all asub loendivaade, mida värskendatakse iga 500 ms järel:

* **Kõik mudelid**: valguse värviproov (sRGB CIE XYZ-st), küllastunud (Jah/Ei), CIE 1931 X/Y/Z, kromaatilisus x/y, CIE u′/v′, CCT (K), CRI (Ra), domineeriv lainepikkus (nm), tipp-lainepikkus (nm), ergastuse puhtus, Duv, CIE L\*/a\*/b\* ja Munsell H/V/C.
* **Ainult kalibreeritud andurid**(mis tahes DAQ-U / DAQ-M / DAQ-E, kui selle tehase kalibreerimispakett on laaditud – selle tunnuseks on roheline**Kalibreeritud** märgis anduri real): koguvõimsus (W/m²), fotopiline luks (lx), skotopiline luks (lx), S/P suhe, PPFD ning PPFD Red

/Green

/Blue

(µmol/m²/s) ja opilised kiirgustihedused — S-koonus, melanopiline, rodopiline, M-koonus, L-koonus (kõik W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Peegeldusgrupid (Ümbritsev valgus + objekt)

Kaks ühendatud andurit saab kombineerida reaalajas peegeldusnäidikuks — ilma kaamerata:

1. Võrgustikuvaates klõpsake diagrammiruudul **+**ja valige**Kombineeri ümbritsev valgus + objekt**.
2. Valige **Ambient Light Source**andur ja**Object Scanner**andur (kaks erinevat andurit), seejärel**Create**.

Chloros

arvutab kahe reaalajas voo põhjal iga lainepikkuse jaoks R(λ) = object(λ) / ambient(λ) (0, kui ambient ≤ 0). Rühma nimetus sõltub andurite kalibreerimisklassist:

* Mõlemad andurid on kalibreeritud (komplekt laaditud) → **„Näiv peegeldusvõime“**.
* Kui mõni andur on kalibreerimata → **„Suhteline peegeldusvõime”**.

Rühm kuvatakse külgribal rohelise reana `REF` ja omaette diagrammina (vikerkaarevärviga täidetud, väärtused hiirega üle libistades 4 kümnendkohani, lohistades saab suumida).

Menüü **+**pakub ka valikut**Lisa uus andur** kolme paigutusvõimalusega: *Kombineeri uus andur* (lisa sellele graafikule), *Tõsta olemasolev andur siia* või *Vaata uut andurit* (omaette graafik).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Taimestikuindeksi tabel

Loendivaates asub taimestikuindeksi tabel peegeldusgrupi graafiku all; see arvutatakse reaalajas peegeldusväärtuste põhjal lainepikkuste keskpunktides **sinine 450 / roheline 550 / punane 670 / NIR

800 nm** (väärtused 4 kümnendkohani, `---`, kui arvutamine ei ole võimalik; viige hiir indeksi nime kohale, et näha selle täisnime):

* **Alati kuvatud** (skaalast sõltumatu, mis tahes andurite kombinatsioon): NDVI

, GNDVI

, ENDVI, WDRVI

, GRVI

, CVI

, GCI

, MSR.
* **Ainult siis, kui mõlemad andurid on võimsuskalibreeritud** (mõlemad andurikomplektid laaditud): EVI

, SAVI

, OSAVI

, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI

, NLI, MNLI, FCI, GEMI

.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## `.daq`-failide salvestamine

* Salvestamiseks on vaja **avatud projekti** – vastasel juhul on nii nupp „Salvesta kõik“ (külgribal) kui ka iga anduri juures olev salvestusnupp blokeeritud.
* Failid salvestatakse kausta **`<project folder>/light_sensor/`**; failinimed sisaldavad anduri ID-d ja ajamärget ning seadme nimi salvestatakse koos salvestusega.
* Kui salvestamine peatub (nupp „Peata”, „Peata kõik” või ühenduse katkemine salvestamise käigus), **lisatakse valmis `.daq` automaatselt avatud projekti** — see ilmub projekti faililoendisse ilma käsitsi lisamiseta, valmis kasutamiseks allapoole suunatud kiirguse andmetena [peegeldusvõime töötlemisel](README.md).
* Salvestamise ajal kuvatakse seadete akna reaalajas reas punane `REC`-indikaator.

Kvantitatiivsete kiirgustiheduse väärtuste saamiseks arvutage keskmine vähemalt 15 sekundi pikkusest andmestikust – see on seadme omadus, mitte viga.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Mitme anduri paigutused ja projekti püsivus

* Kombineerige mitu andurit ühele graafikule (ühised teljed), hoidke graafikuid eraldi (automaatne võrgustiku paigutus), liigutage andureid graafikute vahel, lohistage ridu/ruute ümber ja peidake üksikuid andureid silma ikooniga.
* Chloros

säilitab iga projekti puhul järgmised andmed: seadmete nimed, graafiku värvid, graafiku suurus, vaatamisrežiim ja iga anduri seaded (integreerimisaeg, kaadrite keskmistamine, AE-olek, kate valik).
* **Projekti uuesti avamisel ühendatakse selle andurid automaatselt** aadressi järgi – DAQ-U puhul COM-port, DAQ-M puhul BLE-seade, mDNS-hostinimi DAQ-E puhul (lahendatakse isegi siis, kui seadme IP-aadress on muutunud) — ning rakendab uuesti iga anduri salvestatud kateprofiili, kaadrite keskmistamist, AE-olekut ja käsitsi määratud integratsiooniaega.***

## Kaamera sidumine (DLS)

Paarida pole midagi. Erinevalt droonide DLS-töövoogudest, mis seovad valgussensori kaameraga juba eelnevalt, sobitab Chloros

DAQ-andmed piltidega hiljem: importimise/töötlemise ajal interpoleeritakse `.daq` näidud iga pildi säriaja ajamärgiga. Salvestage mis tahes ühendatud anduriga (`.daq` lisatakse projekti automaatselt) ja peegeldusvõimsuse töötlemine leiab õiged näidud aja järgi — vaadake [DAQ-valgusandureid](README.md), et teada saada, kuidas allapoole suunatud andmeid kasutatakse. **Reaalajas**LATTICE-pildistamiste puhul valige andur hoopis vahekaardi „Kaamerad“ rippmenüüst**Valgusandur**; iga salvestus salvestab seejärel kasutatud näidu, mis on seotud ainult selle salvestuse kaameratega (vaata [Peegeldusvoolud](reflectance.md)).</version\>
