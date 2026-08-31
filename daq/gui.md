# Vahekaart „DAQ” rakenduses Chloros

DAQ-vahekaart — mis Chloros-i külgribal kannab nimetust **Valgusandurid** — on [DAQ-U, DAQ-M ja DAQ-E valgusandurite](README.md) reaalajas juhtpaneel: ühendage andureid mis tahes andmeedastusprotokolli kaudu, jälgige kalibreeritud spektreid reaalajas, arvutage anduripaari abil reaalajas peegeldusvõimet ning salvestage `.daq`-faile otse oma projekti.

Vahekaart muutub kättesaadavaks, kui Chloros-tagapõhi on käivitumise lõpetanud. Vahekaardi graafikuid toidab Chloros-i DAQ-teenus reaalajas ühenduse kaudu, mis katkestuse korral ühendub automaatselt uuesti (2–10 sekundi viivitusega); kui teenusele ei ole juurdepääsu, kuvatakse anduri staatuse real **No Server**.

Paigutus koosneb **anduri külgribast**(üks rida iga ühendatud anduri kohta) ja**diagrammialast** (üks diagrammiplaat iga anduri või rühma kohta).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Anduri ühendamine

Klõpsake külgriba ülaosas nuppu **Ühenda andur**. Ühendamise dialoogaken avaneb põhivaates (või kattekihina, kui lisatakse veel üks andur – sel juhul ilmub nupp „Tühista”).

| Kontroll | Käitumine |
| --- | --- |
| **Seadme tüüp** | `DAQ-U (USB)` (vaikimisi), `DAQ-M (Bluetooth)` või `DAQ-E (Ethernet)`. Valiku muutmisel alustatakse uuesti uue valitud ülekandeviisi otsimist. |
| **Port / BLE-seade / Hostinimi / IP** | Loetleb avastatud seadmed kujul `device - description`; esimene andurina tuvastatud kirje valitakse automaatselt. Skaneerimise ajal kuvatakse `Scanning...` (USB), `Scanning (N)...` koos 8-sekundilise tagasilugemisega (BLE) või `Discovering ethernet sensors (N)...` koos 5-sekundilise tagasilugemisega (Ethernet). Tühjade tulemuste korral kuvatakse `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Värskenda** | Skanneerib valitud ühendusviisi kohe uuesti (BLE/Etherneti skaneerimise ajal on see funktsioon keelatud). |
| **Ühenda** | Aktiveerub, kui seade on valitud; ühenduse loomise ajal muutub märge `Connecting...`-iks. |

Otsing töötab ainult **siis, kui ühendusdialoog on ekraanil**, ja kordub iga 15 sekundi järel ainult valitud andmeside puhul — lihtsalt vahekaardi avamine ei käivita skaneerimist. Ebaõnnestumise korral kuvab dialoog: *„Ühendamine ebaõnnestus. Proovi andurit lahti ühendada ja uuesti ühendada, seejärel klõpsa uuesti nupule „Ühenda“.“*

Külgriba avaneb automaatselt, kui esimene andur ühendub.

{% hint style="info" %}
**DAQ-E ei kuvata?** DAQ-E-l puudub oleku-LED – kontrollige PoE/ühenduse indikaatorit lülitil või süstiku pordil, kuhu see on ühendatud, ning oodake pärast sisselülitamist mõni sekund, et seade saaks käivituda. Chloros seade peab asuma samas levidomeenis (mDNS ei läbi ruutereid). Seadmel Windows tuleb esmakordsel seadme Chloros multisaatepistikute sidumisel (mDNS UDP 5353, DAQ-E andmed UDP 5002, PTP UDP 319/320). Ühes LAN-võrgus asuvaid kahte DAQ-E-seadet tuvastatakse eraldi, kummalgi oma `daq-e-<id>.local`-hostinimega.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Seadmetüüp pakub valikuid DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet)</figcaption></figure>***

## Andurite külgriba

Iga ühendatud andurile eraldatakse üks rida (pluss üks rida iga „Ambient+Object“ rühma kohta). Ridu saab lohistades ümber järjestada ning nende järjekord määrab ka graafikupaneelide järjestuse. Klõpsake rida, et muuta vastav andur või rühm loendivaates aktiivseks graafikuks.

| Element | Tähendus |
| --- | --- |
| Värviline vasak serv | Anduri graafiku värv. |
| Transpordimärk | `DAQ-U` / `DAQ-M` / `DAQ-E`, või roheline märk `REF` keskkond+objekti peegeldusgrupi puhul. |
| Seadme nimi | Vaikimisi on see anduri seerianumber (selle püsiv identiteet kalibreerimiseks, `.daq` failinimede jaoks ning impordi sobitamiseks); kohandatud nimed säilivad projekti kaupa. |
| **Kalibreeritud** märge (roheline) | Kuvatakse, kui anduri tehase kalibreerimispakett on laaditud, st spektrid on tõelised W/m²/nm. |
| **Uuendus saadaval** märge (merevaik, ainult DAQ-E) | Kasutusel olev püsivara on vanem kui selle Chloros versiooniga kaasas olev pilt. Uuendamise ajal kuvatakse reaalajas edenemist (`Flashing… N%`, `Restarting sensor…`, seejärel `Updated X → Y` või `Failed`). |
| Silm | Lülitab selle anduri nähtavuse graafikul sisse või välja. |
| Hammasratas | Avab anduripõhised seaded (allpool). |
| ✕ (punane) | Lülitab anduri lahti või eemaldab Ambient+Object-rühma. |

Ridade kohal asuvad kaks nuppu:

* **Ühenda andur** — avab ühendusdialoogi (muudab nimetuse `Connecting...`-iks, kui ühendamine on käimas).
* **Salvestada kõik / Peata kõik**— alustab või peatab `.daq` salvestamist**kõigil**ühendatud anduritel. Nõuab vähemalt ühte andurit**ja avatud projekti** (tööriista vihje: „Ava projekt salvestamiseks”); muutub punaseks, kui mõni salvestus on käimas.

Tühjas olekus kuvatakse tekst „Ühtegi andurit pole ühendatud“.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Anduripõhised seaded (hammasratta-aknas)

Avage see anduri rea hammasratta-ikooniga. Sisu järjekorras:

* **Inforidad** — seadme tüüp (DAQ-U/M/E), ühendus (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM-port, BLE-aadress või host) ja seerianumber.
* **Kalibreerimisaruanne: Lae alla** — laadib alla selle seadme NIST-jälgitava kalibreerimissertifikaadi (PDF) ja avab selle teie PDF-vaatajas. Saadaval, kui seerianumber on teada; sertifikaat salvestatakse vahemällu esimesel ühendamisel.
* **Seadme nimi** — nime muutmiseks klõpsa pliiatsi ikoonile; säilib projekti kaupa.
* **Graafiku joone värv** — värvivalik; säilib projekti kaupa.
* **Integreerimisaeg (ms)**— liugur + number,**1–500 ms**, vaikimisi**32 ms**. Keelatud, kui AE on sisse lülitatud.
* **Kaadri keskmine**— liugur + number,**1–50 kaadrit**, vaikimisi**20**.
* **AE: sisse/välja**— automaatse särituse lüliti; ühendamisel on**vaikimisi sisse lülitatud**. Lülita see välja, et seadistada integratsiooniaega käsitsi.
* **Peata voog / Alusta voog** — peatab või jätkab otseülekannet.
* **Salvestamine / Salvestamise peatamine** — `.daq` salvestamine iga anduri kohta (vajab avatud projekti).
* **Cap** — cap-korrektsiooni profiil (järgmine jaotis).
* **Reaalaja inforidad** — integratsiooniaeg (ms), kaadrisagedus (FPS), proovid, salvestamine (punane `REC` või `Off`) ja staatus (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Ainult DAQ-E: võrgu-, püsivara- ja PTP-read

* **Hostinimi / IP** — seadme praegune aadress.
* **Püsivara** — püsivara versioon reaalajas, lisaks tegevuslahter:<version\>

nupp</version\>

**Värskenda versioonile \<version\>** ilmub, kui see Chloros versioon sisaldab uuemat DAQ-E püsivara kujutist. Uuendus laaditakse võrgu kaudu umbes 30 sekundiga; andur taaskäivitub ja ühendub automaatselt uuesti ning katkestatud ülekande korral jääb praegune püsivara muutumatuks. Progress kuvatakse reaalajas (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`) ning kui versioon on ajakohane, kuvatakse lahtris `Up to date`.
* **PTP-sünkroniseerimine** — reaalajas PTP-olek (langeb tagasi väärtusele `unknown`). DAQ-E püsivara versioon 1.2.0+ osaleb IEEE 1588 PTPv2-s ainult orja-kellana; Chloros-host on PTP grandmaster, ning kõik LAN-võrgus olevad DAQ-E ja LATTICE kaamerad on selle orjad domeenis 0, hoides ajamärke ligikaudu 1 ms piires.

Ambient+Object rühma puhul näitab seadmete modaalvaade ainult rühma allikaandureid, seadme nime ja graafiku joone värvi.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Kaanevalik

Rippmenüü **Kate** teatab Chloros-ile, milline füüsiline kate on paigaldatud anduri hajuti peale, ning rakendab selle katte tehases mõõdetud korrigeerimisprofiili igale spektrile. Valikud sõltuvad mudelist:

| Mudel | Katevalikud |
| --- | --- |
| DAQ-U | Puudub (paljas andur), vaateväli 15°, vaateväli 30°, vaateväli 45°, vaateväli 60°, vaateväli 90°, „Sunshine“ (koosinuskorrektor) |
| DAQ-M | Puudub (paljas andur), Sunshine (kosinuskorrektor) |
| DAQ-E | Puudub (paljas andur), vaateväli 15°, vaateväli 45°, vaateväli 90°, Sunshine (kosinuskorrektor) |

**Kõigi mudelite vaikimisi seadistus on „Sunshine“ (kosinuskorrektor)** — MAPIR tarnib iga DAQ-seadme koos paigaldatud „Sunshine“-kaane, ja see on standardne välistingimustes kasutatav konfiguratsioon: 180° poolkera vaateväli koos kosinusveaga ≤ ±4 % kuni 60° ja ≤ ±4,5 % kuni 70° (ei soovitata kasutada päikese kõrguse allpool ~15°), konstruktsioonist tuleneva summutamisega (~12×). Teie valik jääb projektis püsima.

{% hint style="warning" %}
**Katte valik peab vastama füüsilisele katele.**Ei andur ega tarkvara suuda tuvastada, milline kate on paigaldatud. Valik mõjutab nii reaalajas korrigeerimist kui ka igasse `.daq`-faili kirjutatud märget – arvestades Sunshine-katte ~12× nõrgenemist, põhjustab deklareerimata kattevahetus spektrite ebaõiget korrigeerimist ligikaudu sama teguri võrra. (Sama kate eemaldamine ja uuesti paigaldamine kordub umbes 1,5 %-ni.) Valige**None (paljas andur)** ainult siis, kui kork on füüsiliselt eemaldatud; DAQ-E puhul rakendab valik „None“ ikkagi tehase geomeetrilist profiili selle süvendatud klaasdiffuusori jaoks – see ei ole tühitoiming – ning paljas DAQ-E on laborikonfiguratsioon, mitte toetatud välitingimustes kasutatav konfiguratsioon.
{% endhint %}

{% hint style="info" %}
Varasemast käsiraamatust uuendamine: brauseri poolne lüliti „Sunshine Diffuser Installed” versioonist 1.1.0 on kadunud. Kaitsekorkide haldamine toimub nüüd iga anduri jaoks eraldi kaitsekorki profiili kaudu, mida rakendatakse serveri poolel.
{% endhint %}

***

## Graafikuala

Kinnitatud ülemine riba sisaldab **loendi ⇄ rastervaate lülitit**ja**graafiku suumimise** liugurit (ruudu suurus 200–2000 px). Vaade lülitub automaatselt ruudustikule, kui on olemas rohkem kui üks diagrammirühm, ja tagasi nimekirjale, kui neid on üks või vähem. Vaate režiim ja diagrammi suurus jäävad projekti kaupa püsima.

Iga anduri **spektridiagramm** näitab:

* **X-telg** — lainepikkus (nm). Andurite võrgustik on 340–1010 nm 5 nm sammuga (135 punkti), mis kuvamiseks interpoleeritakse 1 nm-ni.
* **Y-telg** — võimsus (W/m²), millele valitakse tippväärtuse põhjal automaatselt SI-eesliide (m/µ/n). Spektrid on radiomeetriliselt kalibreeritud spektraalne kiirgusintensiivsus (W/m²/nm) kõigil kolmel andmekandjal.
* Ühe joone all on vikerkaarevärviline spektraalne täide; mitu andurit ühel graafikul kuvatakse üksteise peal värviliste joontena, mille täide on tuhmistatud.
* **Hiirega üle libistamine**— vertikaalne kursor, millel on näidatud lainepikkus ja anduri väärtus;**lohistamine** suurendamiseks (suurendatud vaates ilmub vähendamise nupp).
* **+** nupp (ainult võrgustikvaates) anduri lisamiseks sellele graafikule või rühma loomiseks (allpool).
* Seadme nimi on ülaosas keskel ning esimesele kaadrile jõudmiseni kuvatakse pöörlev ikoon.

**Küllastumine** ei ole graafikul endal märgitud: küllastunud andur näitab punast `SATURATED` staatusteksti ja punast `Saturated: Yes` rida reaalajas andmetabelis. Selle tühistamiseks vähendage integratsiooniaega või lülitage automaatne säri (AE) uuesti sisse.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Reaalajas andmete tabel (loendivaade)

Diagrammi all asub loendivaade, mis uueneb iga 500 ms järel:

* **Kõik mudelid**: valguse värviproov (sRGB CIE XYZ-st), küllastunud (Jah/Ei), CIE 1931 X/Y/Z, kromaatilisus x/y, CIE u′/v′, CCT (K), CRI (Ra), domineeriv lainepikkus (nm), tipp-lainepikkus (nm), ergastuspuhastus, Duv, CIE L\*/a\*/b\* ja Munsell H/V/C.
* **Ainult kalibreeritud andurid**(mis tahes DAQ-U / DAQ-M / DAQ-E, kui selle tehase kalibreerimispakett on laaditud — tunnuseks on roheline**Kalibreeritud** märgis anduri real): koguvõimsus (W/m²), fotopiline luks (lx), skotopiline luks (lx), S/P suhe, PPFD ja PPFD Red/Green/Blue (µmol/m²/s) ning opilised kiirgustihedused – S-koonus, melanopiline, rodopiline, M-koonus, L-koonus (kõik W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Peegeldusgrupid (Ümbritsev valgus + objekt)

Kaks ühendatud andurit saab kombineerida reaalajas peegeldusnäidikuks – kaamerat pole vaja:

1. Võrgustikuvaates klõpsake diagrammi ruudul **+**ja valige**Kombineeri ümbritsev valgus + objekt**.
2. Valige **Ambient Light Source**andur ja**Object Scanner**andur (kaks erinevat andurit), seejärel**Create**.

Chloros arvutab kahe reaalajas voo põhjal lainepikkuse kohta R(λ) = objekt(λ) / ümbritsev(λ) (0, kui ümbritsev ≤ 0). Rühma nimetus sõltub andurite kalibreerimisklassist:

* Mõlemad andurid on kalibreeritud (pakett laaditud) → **„Näiv peegeldusvõime”**.
* Kui mõni andur on kalibreerimata → **„Suhteline peegeldusvõime”**.

Rühm kuvatakse külgribal rohelise `REF`-reana ja omaette diagrammina (vikerkaarevärviga täidetud, väärtused kuvatakse hiirega üle libistamisel 4 kümnendkohani, suurendamine lohistamisega).

Menüü **+**pakub ka valikut**Lisa uus andur** kolme paigutusvõimalusega: *Ühenda uus andur* (liita selle diagrammiga), *Liiguta olemasolev andur siia* või *Vaata uut andurit* (omaette diagramm).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Taimestikuindeksi tabel

Loendivaates asub taimestikuindeksi tabel peegeldusgrupi graafiku all, mis on arvutatud reaalajas peegeldusväärtustest lainepikkuste keskpunktides **sinine 450 / roheline 550 / punane 670 / NIR 800 nm** (väärtused 4 kümnendkohani, `---`, kui arvutamine ei ole võimalik; viige hiir indeksinime kohale, et näha selle täisnime):

* **Alati kuvatud** (skaalast sõltumatu, mis tahes andurite kombinatsioon): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Ainult juhul, kui mõlemad andurid on kalibreeritud toitepõhiselt** (mõlemad komplektid laaditud): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## `.daq`-failide salvestamine

* Salvestamiseks on vaja **avatud projekti** — vastasel juhul on nii nupp „Salvesta kõik“ (külgribal) kui ka anduripõhine salvestusnupp blokeeritud.
* Failid salvestatakse **`<project folder>/light_sensor/`**-i; failinimed sisaldavad anduri ID-d ja ajamärget ning seadme nimi salvestatakse koos salvestusega.
* Kui salvestamine lõpeb (nupp „Peata”, „Peata kõik” või ühenduse katkemine salvestamise käigus), **lisatakse valmis `.daq` automaatselt avatud projekti** — see ilmub projekti faililoendisse ilma käsitsi lisamiseta, valmis kasutamiseks allapoole suunatud kiirguse andmetena [peegeldusvõime töötlemiseks](README.md).
* Salvestamise ajal kuvatakse seadete akna reaalajas reas punane `REC` indikaator.

Kvantitatiivsete kiirgustiheduse näitajate saamiseks arvutage keskmine vähemalt 15 sekundi pikkusest andmereast — see on seadme omadus, mitte viga.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Mitme anduri paigutused ja projekti püsivus

* Kombineerige mitu andurit ühele graafikule (ühised teljed), hoidke graafikuid eraldi (automaatne ruudustiku paigutus), liigutage andureid graafikute vahel, lohistage ridu/ruutu ümber ja peidake üksikuid andureid silmaikooniga.
* Iga projekti puhul säilivad Chloros järgmised andmed: seadmete nimed, graafiku värvid, graafiku suurus, vaaterežiim ja iga anduri seaded (integreerimisaeg, kaadrite keskmistamine, AE-olek, piirangu valik).
* **Projekti uuesti avamisel ühendatakse andurid automaatselt** aadressi järgi — DAQ-U puhul COM-port, DAQ-M puhul BLE-seade, DAQ-E puhul mDNS-hostinimi (lahendatakse isegi siis, kui seadme IP-aadress on muutunud) – ning rakendab uuesti iga anduri salvestatud kateprofiili, kaadrite keskmistamist, AE-olekut ja käsitsi määratud integratsiooniaega.***

## Kaamera sidumine (DLS)

Paarida pole midagi. Erinevalt droonide DLS-töövoogudest, mis seovad valgussensori kaameraga juba alguses, sobitab Chloros DAQ-andmed piltidega alles hiljem: importimise/töötlemise ajal interpoleeritakse `.daq` näidud iga pildi säriaja ajamärgistusega. Salvestage mis tahes ühendatud anduriga (`.daq` lisatakse projektile automaatselt) ning peegeldusvõimsuse töötlemine leiab õiged näidud aja järgi — vaadake [DAQ-valgusandurid](README.md), et teada saada, kuidas allapoole suunatud andmeid kasutatakse.</version\>
