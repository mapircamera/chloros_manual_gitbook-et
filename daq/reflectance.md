# Peegeldusvõime töövood

DAQ-valgusandur muudab radiomeetrilised pildid peegeldusvõimeks. On kaks erinevat töövoogu:

1. **Ühe anduriga** — üks DAQ mõõdab allapoole suunatud kiirgust, samal ajal kui kaamera pildistab, ning Chloros jagab kaamera kiirgust selle võrdlusväärtusega.
2. **Kahe anduriga** — kaks DAQ-andurit, millest üks jälgib taevast ja teine objekti, loovad reaalajas spektraalse peegeldusvõime kõvera ilma kaamerata.

## Üks andur + kaamera (allapoole suunatud võrdlusväärtus)

DAQ toimib allapoole suunatud valguse andurina (DLS): kaamera mõõdab ülespoole suunatud kiirgust **L**(W/m²/sr/nm), DAQ mõõdab allapoole suunatud kiirgust**E** (W/m²/nm) ning Chloros arvutab peegeldusvõime iga lainepikkuse kohta järgmiselt:

> ρ = π · L / E

DAQ-i näidud on alati **ajaliselt sünkroniseeritud ekspositsiooniga** — seetõttu kasutavad DAQ ja kaamerad ühist PTP-sünkroniseeritud kella (vt [DAQ-E võrgustik ja ajasünkroniseerimine](ethernet-ptp.md)). Paigalda välitöödeks Sunshine'i kosinusmüts ja deklareeri see õigesti; mütsi deklareerimine skaalab otseselt E-d (vt [Mütsiprofiilid ja kalibreeritud vahemik](caps-and-range.md)). Kvantitatiivse töö puhul pidage meeles seadme omadusi: kvantitatiivne kiirgustihedus arvutatakse vähemalt 15 sekundi jooksul tehtud mõõtmiste keskmisena.

### Reaalajas salvestamine

Seostage DAQ kaameraga vahekaardil „Kaamerad”: iga kaamera seadete paneelis on rippmenüü **Valgusandur**, milles on loetletud kõik ühendatud DAQ-seadmed (DAQ-U/M/E) vahekaardilt „Valgusandurid”; sünkroniseeritud massiivi puhul levib kogu massiivi hõlmav valgusanduri valik igale liikmele (üksikud kaamerad võivad selle siiski ümber kirjutada). Kui seos on loodud, suunatakse anduri spektrid kaamera DLS-pesasse ja eksporditavad peegeldusväärtused jagatakse vastava näiduga. Massiivis kasutab oma anduriga liige oma näitu – nii reaalajas komposiitpildis olevate ribade, eksporditud andmete kui ka automaatse särituse puhul – ning teised liikmed kasutavad massiivi näitu.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Kaks teadmisväärset käitumist:

* **Kui DAQ-seadet pole seotud → peegeldusväärtust ei aktsepteerita, seda ei võltsita.** Chloros lükkab peegeldusväärtuse tulemuse tagasi ja salvestab vahelejätmise põhjuse, selle asemel et vaikimisi tagastada madalamat tulemust.
* **Kasutatud näit säilitatakse.** Iga peegeldusraami puhul kirjutatakse tegelikult rakendatud DAQ-näit `.daq`-failina salvestuste kausta `daq/`, et salvestust saaks hiljem uuesti töödelda ([Salvestamine ja .daq-vorming](recording.md)). Selle kõrval asuv `.daq.cameras.json` nimetab kaameraid, mida see kirjeldab, ning uuesti töötlemisel sobitatakse see ainult nende kaamerate piltidega.

### Salvestatud piltide töötlemine

Lennujärgseks töötlemiseks salvestage seansi ajal `.daq` ja hoidke seda piltide kõrval — töötlemisahel lahendab ajamärgistusega sobiva allapoole suunatud kiirguse automaatselt, hankides esmakordsel kasutamisel puuduvad tehase kalibreerimisandmed MAPIR i pilvest. GUI-salvestused lisatakse avatud projektile automaatselt, kui need peatuvad.

Peegeldusvõime viidet saab valida töötlemise ajal – `--reflectance-source` või `chloros-cli process`, või GUI projekti seadetest peegeldusvõime allika seade:

| Väärtus | Käitumine |
| --- | --- |
| `auto` (vaikimisi) | Kvaliteedikontrolli läbinud raamisisene kalibreerimise sihtmärk on absoluutne viide; DAQ-andmete põhjal arvutatud allapoole suunatud kiirgus (ρ = π·L/E) on varuvariant. Sihtmärki, mis läbib kvaliteedikontrolli lipuga (paneel on välja jäänud või sagedusriba ei tööta), ei eelistata automaatselt: proovitakse järjest sihtmärki, sihtmärki koos DAQ-ga ja ainult DAQ-d, lähtudes nende mõõdetud skoorist väljajäetud paneelidel. |
| `daq` | DAQ-põhine |
| `target` | Rangelt raami sees olev sihtmärk; DAQ-asendust ei kasutata |

Vaata [kalibreerimissihte](../calibration-targets.md) sihtmärgi töövoogude kohta ning [LATTICE-peatükk](../lattice/README.md) ning [CLI-viide](../reference/cli-reference.md) kogu töötlemisprotsessi kohta. Eksporditud peegeldusvõime pikslite lugemisel kasutage märgitud skaalat (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) – vt [Väljundpildi formaadid](../output-image-formats.md).

### DAQ-i kalibreeritud vahemikust väljaspool asuvad lainepikkused

DAQ-i radiomeetriliselt kalibreeritud vahemik on ~374–974 nm. Chloros lükkab tagasi DAQ-põhise peegeldusandmed kõikide kaamerasageduste puhul, mille spektraalne kaal selles vahemikus on väiksem kui pool, teatades vahelejätmise põhjuseks `dls-uncalibrated-band-<nm>`. Müügil olevatest toodetest mõjutab see ainult mudelit F988: F988 peegeldusvõime kalibreeritakse stseeni sees asuva peegeldusvõime paneeli abil; kuna sagedusala jääb väljapoole DAQ-i valgusanduri kalibreeritud vahemikku, kasutab Chloros teie viimast paneeli mõõtmistulemust ja säilitab selle paneeli mõõtmiste vahelisel ajal. Kui F988-kaamerat kasutatakse ainult DAQ-režiimis, lükkab Chloros selle lainepikkuse puhul DAQ-põhise peegeldusvõime tagasi, näidates vahelejätmise põhjusena `dls-uncalibrated-band-988` – toetatud meetodiks on paneeli kasutamine.

## Kahe anduri kasutamine (ümbrusvalgus + objekt)

Kaks DAQ-andurit – mis tahes paar, mis tahes andmeedastusviisil – annavad reaalajas peegeldusspektri ilma kaamerata: üks andur on suunatud taeva poole (**ümbrusvalgusallikas**), teine objekti poole (**objekti skanner**), jChloros
arvutab lainepikkuse kaupa:

> R(λ) = objekt(λ) / ümbritsev valgus(λ)

(null, kui ümbritsev valgus ≤ 0).

### Graafilises kasutajaliideses

Kui mõlemad andurid on ühendatud vahekaardil „Valgusandurid”, avage anduri lisamise aknake (nupp „+” graafiku ruudul võrgustikuvaates) ja valige **Kombineeri ümbritsev valgus + objekt**. Valige rippmenüüdest „Ambient Light Source“ ja „Object Scanner“ kaks andurit ning klõpsake nuppu „Create“. Rühm kuvatakse eraldi graafikuna ja külgriba real, millel on roheline**REF**-märgis.



<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->
Peegeldusdiagrammi (loendivaade) all arvutab reaalajas **taimestiku indeksite tabel** indeksid kõverast, kasutades lainepikkuste keskpunkte: sinine 450 / roheline 550 / punane 670 / NIR 800 nm. Suhtarvupõhised indeksid, mis tühistavad absoluutse skaala (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR), kuvatakse alati; indeksid, mis vajavad absoluutset peegeldusvõimet (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI), kuvatakse ainult siis, kui mõlemad andurid on võimsuskalibreeritud mudelid.

### Näiv vs. suhteline — märgistamise reegel

Chloros märgistab kahe anduri väljundit vastavalt sellele, mida anduripaar tegelikult suudab pakkuda:

| Anduripaar | Märge |
| --- | --- |
| Mõlemad andurid kalibreeritud — tehase kalibreerimispakett laaditud | **Näiv peegeldusvõime** |
| Üks andur kalibreerimata | **Suhteline peegeldusvõime** |

Kõik kolm mudelit on radiomeetrilised: kui anduri tehase kalibreerimispakett on laaditud, on selle spektrid absoluutsed (W/m²/nm), seega annab kalibreeritud andurite paar suhtarvu absoluutsele näivale peegeldusvõimele — transport seda ei määra. Andur, mis edastab endiselt töötlemata loendustulemusi (paketti pole kättesaadav), alandab tulemuse suhteliseks kõveraks (spektri kuju jääb kehtivaks). Mõlemal anduril peaksid olema korrektselt deklareeritud piirangud ([Piiranguprofiilid ja kalibreeritud vahemik](caps-and-range.md)).

### Python
ist

Ühendatud SDK-liideses puudub spetsiaalne kahe anduri kutsufunktsioon: avage kaks seanssi `chloros_sdk.connect_daq_sensor()`-ga ja arvutage nende `latest()` spektrite suhe ise, kasutades sama märgistamiskonventsiooni. (Kahe anduri salvestusvahend on olemas ka MAPIR i sisemises otsese riistvaraliideses, mis on täielikkuse huvides loetletud [CLI-viites](../reference/cli-reference.md) – see ei kuulu kaasasoleva CLI-i koosseisu; ülaltoodud graafilise kasutajaliidese töövoog on toetatud reaalajas kasutatav viis.)
