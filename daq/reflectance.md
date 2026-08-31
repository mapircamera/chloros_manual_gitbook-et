# Peegeldusvõime töövood

DAQ-valgusandur teisendab radiomeetrilised pildid peegeldusvõimeks. On kaks erinevat töövoogu:

1. **Ühe anduriga** — üks DAQ mõõdab allapoole suunatud kiirgust, samal ajal kui kaamera pildistab, ning Chloros jagab kaamera kiirgust selle võrdlusväärtusega.
2. **Kahe anduriga** — kaks DAQ-andurit, millest üks jälgib taevast ja teine objekti, genereerivad reaalajas spektraalset peegeldusvõime kõverat ilma kaamerata.

## Üks andur + kaamera (alla suunatud võrdlusväärtus)

DAQ toimib alla suunatud valguse andurina (DLS): kaamera mõõdab ülespoole suunatud kiirgust **L**(W/m²/sr/nm), DAQ mõõdab allapoole suunatud kiirgust**E** (W/m²/nm) ja Chloros arvutab peegeldusvõime iga lainepikkuse kohta järgmiselt:

> ρ = π · L / E

DAQ-i näidud on alati **ajastatud vastavalt eksponeerimisajale** — seetõttu kasutavad DAQ ja kaamerad ühist PTP-sünkroniseeritud kella (vt [DAQ-E võrgustik ja ajasünkroniseerimine](ethernet-ptp.md)). Paigaldage välitöödeks Sunshine&#x27;i koosinuskate ja deklareerige see õigesti; katte deklaratsioon mõjutab otseselt E väärtust (vt [Katteprofiilid ja kalibreeritud vahemik](caps-and-range.md)). Kvantitatiivseks tööks pidage meeles seadme omadusi: kvantitatiivne kiirgustihedus arvutatakse vähemalt 15 sekundi pikkuste mõõtmistulemuste keskmisena.

### Reaalajas salvestamine

Seostage DAQ kaameraga vahekaardil „Kaamerad”: iga kaamera seadete paneelis on rippmenüü **Valgusandur**, milles on loetletud kõik ühendatud DAQ-seadmed (DAQ-U/M/E) vahekaardilt „Valgusandurid”; sünkroniseeritud massiivi puhul levib kogu massiivi hõlmav valgusanduri valik igale liikmele (üksikud kaamerad võivad selle valiku siiski tühistada). Kui seos on loodud, edastatakse anduri spektrid kaamera DLS-pesasse ja eksporditavad peegeldusandmed jagatakse vastava näiduga.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Kaks olulist käitumist, mida tasub teada:

* **Kui DAQ-seadet pole seotud → peegeldusvõimsus lükatakse tagasi, seda ei võltsita.** Chloros lükkab peegeldusvõimsuse tulemuse tagasi ja salvestab vahelejätmise põhjuse, selle asemel et vaikimisi tagastada madalamat tulemust.
* **Kasutatud näit säilitatakse.** Iga peegeldusraami puhul kirjutatakse tegelikult kasutatud DAQ-näit pildi kõrvale `.daq`-sidecarina, et salvestist saaks hiljem uuesti töödelda ([Salvestamine ja .daq-vorming](recording.md)).

### Salvestatud piltide töötlemine

Lennujärgseks töötlemiseks salvestage seansi ajal `.daq` ja hoidke seda piltide kõrval – töötlemisprotsess lahendab ajamärgistusega sobiva allapoole suunatud kiirguse automaatselt, laadides esmakordsel kasutamisel puuduvad tehase kalibreerimisandmed MAPIR pilvest. GUI-salvestused lisatakse avatud projektile automaatselt, kui need lõpevad.

Peegeldusvõime viidet saab valida töötlemise ajal – `--reflectance-source` failis `chloros-cli process` või GUI projekti seadete peegeldusvõime allika seadistus:

| Väärtus | Käitumine |
| --- | --- |
| `auto` (vaikimisi) | Kvaliteedikontrolli läbinud raamisisene kalibreerimise sihtmärk on absoluutne etalon; DAQ-i allapoole suunatud kiirgus (ρ = π·L/E) on varuvariant |
| `daq` | DAQ-põhine |
| `target` | Rangelt raami sees olev sihtmärk; DAQ-asendust ei kasutata |

Vaata [kalibreerimissihte](../calibration-targets.md) sihtmärkide töövoogude kohta ja [LATTICE peatükki](../lattice/README.md) ning [CLI viite](../reference/cli-reference.md) kohta, et tutvuda täieliku töötlemisprotsessiga. Eksporditud peegeldusvõime pikslite lugemisel kasutage märgitud skaalat (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — vt [Väljundpildi formaadid](../output-image-formats.md).

### DAQ-i kalibreeritud vahemikust väljaspool asuvad lainepikkused

DAQ-i radiomeetriliselt kalibreeritud vahemik on ~374–974 nm. Chloros keeldub DAQ-põhisest peegeldusvõimest mis tahes kaamera lainepikkuse puhul, mille spektraalne kaal jääb alla poole sellest vahemikust, teatades vahelejätmise põhjuseks `dls-uncalibrated-band-<nm>`. Müügil olevatest tootenumbritest mõjutab see ainult F988-t: F988 peegeldusvõimet kalibreeritakse stseeni sees oleva peegeldusvõime paneeli abil: lainepikkus jääb väljapoole DAQ-i valgussensori kalibreeritud vahemikku, mistõttu Chloros kasutab teie viimast paneeli salvestust ja säilitab seda paneeli vaatluste vahel. Kui F988-kaamerat kasutatakse ainult DAQ-režiimis, lükkab Chloros selle lainepikkuse puhul DAQ-põhise peegeldusvõime tagasi, kasutades vahelejätmise põhjust `dls-uncalibrated-band-988` – paneeli kasutamine on toetatud meetod.

## Kahe anduri süsteem (ümbrusvalgus + objekt)

Kaks DAQ-andurit – mis tahes paar, mis tahes andmeedastusviisidel – annavad reaalajas peegeldusspektri ilma kaamerata: üks andur on suunatud taeva poole (**ümbrusvalgusallikas**), teine objekti poole (**objekti skanner**), ning Chloros arvutab iga lainepikkuse kohta:

> R(λ) = objekt(λ) / ümbritsev valgus(λ)

(null, kui ümbritsev valgus ≤ 0).

### Graafilises kasutajaliideses

Kui mõlemad andurid on vahekaardil „Valgusandurid“ ühendatud, avage anduri lisamise hüpikaken (nupp „+“ graafikuplaadil rastervaates) ja valige **Kombineeri ümbritsev valgus + objekt**. Valige rippmenüüdest „Ambient Light Source“ ja „Object Scanner“ kaks andurit ning klõpsake nuppu „Create“. Rühm kuvatakse eraldi graafikuna ja külgriba reana, millel on roheline**REF**-märgis.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Peegeldusdiagrammi (loendivaade) all arvutab reaalajas **taimestikuindeksite tabel** indekseid kõverast, kasutades lainepikkuste keskpunkte sinine 450 / roheline 550 / punane 670 / NIR 800 nm. Suhtarvupõhised indeksid, mis tühistavad absoluutse skaala (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) kuvatakse alati; indeksid, mis vajavad absoluutset peegeldusvõimet (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) kuvatakse ainult siis, kui mõlemad andurid on võimsuskalibreeritud mudelid.

### Näiv vs. suhteline — märgistamise reegel

Chloros märgistab kahe anduri väljundi selle järgi, mida anduripaar tegelikult suudab pakkuda:

| Anduripaar | Märge |
| --- | --- |
| Mõlemad andurid kalibreeritud — tehase kalibreerimispakett laaditud | **Näiv peegeldusvõime** |
| Üks andur kalibreerimata | **Suhteline peegeldusvõime** |

Kõik kolm mudelit on radiomeetrilised: kui anduri tehase kalibreerimispakett on laaditud, on selle spektrid absoluutsed (W/m²/nm), seega annab kalibreeritud andurite paar suhtarvu absoluutsele näivale peegeldusvõimele — transport seda ei määra. Andur, mis edastab endiselt töötlemata loendustulemusi (paketti pole kättesaadav), alandab tulemuse suhteliseks kõveraks (spektri kuju jääb kehtima). Mõlemal anduril peaksid olema korrektselt deklareeritud piirangud ([Piiranguprofiilid ja kalibreeritud vahemik](caps-and-range.md)).

### Alates Python

Ühendatud SDK pinnal puudub spetsiaalne kahe anduri kutsung: avage kaks seanssi `chloros_sdk.connect_daq_sensor()`-ga ja arvutage nende `latest()` spektrite suhe ise, kasutades sama märgistamiskonventsiooni. (Kahe anduri salvestustööriist on olemas ka MAPIR-i sisemisel otsese riistvarapinnal, mis on täielikkuse huvides loetletud [CLI-i viites](../reference/cli-reference.md) – see ei kuulu tarnitud CLI komplekti; ülaltoodud graafilise kasutajaliidese töövoog on toetatud reaalajas kasutatav viis.)
