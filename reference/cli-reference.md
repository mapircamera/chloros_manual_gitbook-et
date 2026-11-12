# Chlorosi CLI-juhend

**Versioon:**

1.3.0**Koostatud:**

29.07.2026 kell 19:19 ·**Muudetud:**

06.11.2026**Sihtrühm:** Optimeeritud masinõppe (LLM) kasutamiseks; inimesele loetav.**Käsitlusala:** Kõik `chloros-cli` kasutajale suunatud alamkäskud koos valikuvõimaluste ja kopeeritavate näidetega.

Käesolev dokument on täielik viide `chloros-cli`-i käsurea tööriistale, mis kuulub MAPIR Chlorosi komplekti. See on tahtlikult põhjalik, et LLM (või inimene) saaks allpool esitatud loetelude põhjal koostada mis tahes toetatud töövoogu ilma lähtekoodi uurimata.

Kui vajate vaid olulisimat teavet, minge otse järgmistesse osadesse:
- [Viie minuti kiirstart](#five-minute-quickstart)
- [LATTICE-kaamera esmakordse ühendamise töövoog](#lattice-camera-first-connect-workflow)
- [DAQ-anduri esmakordse ühendamise töövoog](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Salvestusrežiimid, salvestid ja offline-taas töötlemine](#capture-modes-recorders--offline-reprocess)

> **Chlorosi juhtimine AI-assistendi abil?** Claude Desktop, Claude Code, Cursor ja muud MCP-kliendid saavad käivitada töötlemist, LATTICE-kaameraid ja -maatrikseid ning DAQ-valgusandureid `chloros-mcp` kaudu, MCP-serveri kaudu, mis installitakse koos Python'i SDK-ga (ja Linuxi .deb-failiga, vt `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Seadistused, valikud ja tööriistad on SDK-viite jaotises [MCP-server (AI-assistendid)](sdk-reference.md#mcp-server-ai-assistants). Shelliga assistent saab ka käivitada iga käsku sellel lehel otse.

> **Muudetud versioonis 1.3.0** (mida märkab olemasolev 1.2.0 skript või töövoog).
>
> **Muudatused, millega 1.2.0 skript võib vajada tegelemist:**
>
> - **Kehtib `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` ja `align-stream` jäeti versioonis 1.2 välja.0 ja töötas kaamera enda seadetega. Nüüd määrab see eelseadistuse formaadi, säri, võimenduse ja kaadrisageduse: `--preset high_speed` salvestab 8-bitist pilti 5 ms, 6 dB, 60 kaadrit sekundis. `lattice cam-connect --preset` säilitab samuti eelseadistuse formaadi (versioonis 1.2.0 seadistati kaamera pärast eelseadistuse rakendamist 12-bitiseks). Vaata [Üldised valikud](#common-options-shared-by-most-subcommands).
> - **M3M (mono) kaamera puhul on Bayeri pikseliformaat selle sügavus.** Eelseadistused nimetavad Bayeri formaate, mida monosensor ei suuda salvestada: versioonis 1.2.0 keeldus kaamera sellest formaadist ja säilitas oma senise formaadi, seega `--preset high_speed` ei olnud seal 8-bitine. Nüüd on see sama sügavusega Mono-formaat: `high_speed` ja `--pixel-format BayerRG8` annavad `Mono8`, `high_quality`, `triggered` ja `BayerRG12` annavad tulemuseks `Mono12`. Segatud M3C + M3M massiivis võtab iga kaamera kasutusele oma perekonna ühise formaadi ning projektid `network-analysis` / `analyze-array --pixel-format` projitseerivad iga kaamera sellele sügavusele. Sama kehtib ka vastupidi: Mono-formaati nõudev M3C võtab kasutusele selle sügavuse Bayer-formaadi, seega massiiv, mille peakaamera on M3M, ei ava enam oma M3C-kaameraid Mono-režiimis. Vaata [Mono (M3M) ühe sagedusribaga kaamerad](#mono-m3m-single-band-cameras).
> - **`lattice array-capture`-eksportfailid on vaikimisi joondatud, nagu ka rakenduse omad.** Iga liikme `debayered`, `display`, `radiance`, `reflectance` ja indeksifailid paigutatakse peakaamera kaadrisse ja kärbitakse liikmete kattuvuse piires (mis on kaamerate nihkete tõttu väiksem kui sensor, `XMP-mapir:AlignmentJSON` `"applied": true`), seega on CLI-salvestus samad failid, mis Chloros-rakenduse salvestus samast stseenist; `raw` jääb algupäraseks. Varem salvestati salvestus ilma `--aligned`-ita iga faili algupärases suuruses, moonutamata. `--no-aligned` säilitab selle. Sama vaikimisi seade kehtib ka `--smart`, `--continuous` ja `--interval` ning SDK-le ja `chloros-mcp`-ile. Vaata [`array-capture` režiime](#array-capture-modes).
> - **`lattice array-capture --compression` on vaikimisi `none`.** Failid salvestatakse tihendamata kujul (umbes 6,3 MB täisresolutsiooniga kaadri kohta), et saavutada kiireim püsiv salvestuskiirus; versioon 1.2.0 salvestas `deflate`-vormingus (umbes 4,1 MB). Väiksemate failide saamiseks kasuta `--compression deflate`. Mõlemad on kadudeta.
> - **PNG- ja JPEG-vormingud toetavad ainult `display` / `preview` ja indeksitud tooteid.** `--format png` või `jpg` salvestus, mille tase ületab 8-bitise faili mahutavuse, lükatakse tagasi enne salvestamise alustamist (HTTP-kood 400, väljumine `1`), samuti lükatakse tagasi vaikimisi `--processing all` koos `--format png`/`jpg`-ga. Vaata [Salvestuse eksporditasemed](#capture-export-levels-the-all-default).
> - **`lattice dls` vajab `--mac`-i.**Vaikimisi MAC-aadressi enam ei ole (see oli ühe seadme aadress); `daq pool-discover --only ble` leiab anduri aadressi.**`lattice dls --camera SERIAL`** on uus.
> - **Vead põhjustavad `1` lõpetamist.** `process`, kui käivituse eksporti ei suudetud kontrollida või kui kaotati pilt, kuigi teised tooted on salvestatud; salvestamiskäsk koos mis tahes failiga, mida ei suudetud salvestada (ülejäänud failid salvestatakse ikkagi); `lattice continuous` ja `multi-stream`, kui kõvaketas on täis; `array-burst` / `array-record`, kui salvestusseade teatab veast; `array-build-video`, kui taotletud toode jäeti vahele või ei koostatud vastavalt taotlusele; `project burst` / `stream`, kui salvestamine ebaõnnestub pooleli; `project sensor log`, kui salvestus ei sisalda ühtegi lugemist; `lattice exposure --set`, `gain --set` ja `configure --set`, kui kaamera keeldub väärtust vastu võtmast. `lattice network-analysis --json` lõpetab töö koos kokkuvõttega (`1` asemel `needs_force_slip`). Vastupidi: andur, mis ei kuulu enam andurite kogumisse, lõpetab töö koodiga `daq pool-disconnect`, mis lõpetab `0`. Vaata [Lõpetamiskoodid](#exit-codes).
> - **Aeglane `lattice array-connect` ei ebaõnnestu kunagi, kui massiiv on ühendatud.** See ootab kuni 600 s (1.2.0 versioonis 180 s; `project connect` 90 s), väljastades samal ajal tagapõhja toiminguid, ning kui ühendus selleks ajaks pole loodud, tühistab tagapõhi — iga kaamera asetatakse tagasi sellesse asendisse, milles see leiti — seega tähendab `Error:` (väljumine `1`), et massiiv ei ole ühendatud. Versioonis 1.2.0 võis kohe pärast kaamerate sisselülitamist algav ühendamine ooteaja üle kesta ja ühendada massiivi pärast seda, kui käsk oli ebaõnnestunud. Nüüd tühistab Ctrl+C ühendamise. Vaata [Püsivad ühendused](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` säilitab täieliku anduri.** Kui `--width` / `--height` / `--binning` puuduvad, on iga kaamera seadistatud täissensorile ja kaadrisagedus langeb selle arvuti ja võrgu võimekuse tasemele; ühenduse väljund loetleb kiiremaid võimalusi (pildipunktide ühendamine või väiksem huvipiirkond) koos nende prognoositavate kiirustega. Versioon 1.2.0 vähendas või ühendas kaadreid, et kiirust säilitada. Vaata [`array-connect` valikuid](#array-connect-options).
> - **Aeglane võrk ei takista enam massiivi töötamist.** Iga kaamera töötab vastavalt oma võrguühenduse kiirusele ja kaadrisagedus langeb sellega koos. Kui kaameraid ei ole võimalik nii aeglaselt tööle panna, töötab massiiv järjestikusrežiimis (`slip-emit-and-capture`): iga kaader jõuab kohale, kuid kaameraid ei eksponeerita üheaegselt. `--force-tier sim-capture-sim-emit` nõuab üheaegset eksponeerimist. Vaata [Massiivi kaadrisagedus ja sarivõtte mudel](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` määravad kindlaks, millist ühendust rakendatakse**: ilma `--pixel-format`-ita määrab „connect” formaadid (Mono12 / BayerRG12, varem oli BayerRG8) ja [pakitud 12-bitine](#packed-12-bit-transport---packed) valiku. Seetõttu võib sama käsk näidata teistsugust kaadrisagedust kui versioonis 1.2.0.
> - **Suhtelised teed viitavad kohale, kus käsk käivitatakse.** `-o` ja muud teed, mida tagapõhi avab (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), retsepti sammu `output`, ning `lattice index --live` vaikimisi `output/` lahendatakse selle kausta suhtes, millest te `chloros-cli` käivitate. Versioonis 1.2.0 lahendas tagapõhi need omaenda installikausta järgi, mis installitud versiooni puhul lõppes veaga „Juurdepääs keelatud”. Vaata [Konventsioone](#conventions).
> - **`lattice index` töötab backendis igas vormis**, nagu ka teised `lattice`-käskud, seega käivita esmalt backend. Offline-indeks (`--input`) arvutab välja backend, mis loeb TIFF-faili ja kirjutab tulemused selle kõrvale või kataloogi `-o`, ning `--list-presets` / `--list-gradients` trükivad välja backend'i tabelid. Windowsis toimivad need nüüd: varem vastas Windowsi CLI igale `lattice index` vormile, sealhulgas `--live`, vastusega `Error: lattice_sdk not available`. Linuxis töötas offline-indeks varem CLI-s ilma backendita ja sisselogimiseta; nüüd vajab see mõlemat, nagu iga teine CLI-käsk. Vaata [Indeks / Taimestiku matemaatika](#index--vegetation-maths).
> - **Uued failid toodete kõrval.** Iga kiirguse, peegelduse ja indeksi toode, milles lainepikkuse segunemise eemaldamine piksli väärtust piiras, saab maski `clamp_masks/`-kausta toote kausta kõrvale ning print-väljundisse lisatakse rida `clamp mask:`. Vaata [Lahutamise piiramismask](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Kaamera orientatsioon.** Ühe kaameraga salvestamisel salvestatakse failid `raw` ja `debayered` sellisena, nagu andur need luges, ning kaamera pööramine / pöörlemine salvestatakse XMP-faili; `process` rakendab seda loodud toodetele. Massiivi liikmete failid jäävad algkujul, kui te ei edasta uut `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Vaata [Kaamera suund](#kuidas-näeb-välja-pildikausta-sisu).
> - **Eksporditaseme nimi, mis ei ole üks, lükatakse tagasi.** `lattice capture --processing` ja kõik `--levels` nimekirjad võtavad vastu `raw`, `debayered`, `radiance`, `reflectance` või `display` (või aliase, nagu `preview`; `--processing` ja ka `all`). Mis tahes muu nimi lükatakse tagasi enne kaamera puudutamist (HTTP 400 `UNKNOWN_LEVEL`, väljumine `1`); 1.2.0 salvestas selle jaoks `raw` ja lõpetas töö `0`.
> - **`daq pool-set-cap` vajab `--cap-id`-i.** Ilma selleta lõpetab käsk töötamise `2`; 1.2.0 eemaldas kattekorrektsiooni, jättes alles palja anduri. `--cap-id none` küsib paljast andurit.
> - **`lattice resolution --binning-mode` on kadunud.** Binning on alati „Average“; lipukest aktsepteeritakse endiselt, kuvatakse hoiatus ja midagi ei tehta. Vaata [Anduri häälestamine](#sensor-tuning).
> - **`process --debayer texture-aware`-i kasutamine LATTICE-pildistuste puhul keelatakse.** LATTICE-pildistusi sisaldava kausta puhul väljub `process` pärast importimist `1`-ist, nimetades piltide arvu, enne kui midagi töödeldakse. LATTICE-pildid läbivad alati standardse demosaikimise, seega versioon 1.2.0 aktsepteeris selle valiku, kuid ignoreeris seda; ka rakendus Chloros ei paku seda nende jaoks. Vaata [`chloros-cli process`](#chloros-cli-process).
> - **Kaamerad lähevad tagasi sellisesse seisundisse, millises nad leiti — välja arvatud juhul, kui kasutatakse `--keep-settings`.** `lattice array-disconnect`, iga ajutine käsk (`multi-capture` ilma massiivita, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) ning `lattice continuous` taastavad iga kaamera selliseks, nagu see leiti: selle päästiku ja sünkroniseerimisjuhtme ühendused, PTP ning kõik seansi käigus muudetud kaamera seaded (valgustatus ja võimendus, automaatse eksponeerimise sihtmärk, piirid ja summutamine, valgebalanss, mustatase, pikseliformaat, pikselite ühendamine ja huvipiirkond, pildi peegeldamine, kaadrisagedus). `--keep-settings` salvestab aga iga kaameraseaded kaamera enda mällu, nii et see käivitub nendega; `--clear-settings` taastab selle mälu tehase seadetele. Mõlemal juhul jäetakse kaamerad vabalt töötama. Projekt seadistab oma kaamerad oma salvestatud seadetest, kui need ühendatakse. Vaata [Kaamera mälu](#camera-memory----keep-settings----clear-settings).
> - **Iga režiim salvestab projekti samal viisil.** Iga muudatus projektiga ühendatud kaameras, massiivis või valgussensoris – olgu see tehtud rakenduse, CLI, SDK või MCP-serveri kaudu – salvestatakse selle projekti `cameras.json` / `sensors.json`, ning projekti avamine mis tahes režiimis seadistab seadmed selle järgi. Varem salvestas projekti ainult rakendus ning selle järgmine salvestamine kirjutas üle kõik muu. Vaata [Üks projektikirje, iga režiim](#one-project-record-every-mode).
> - **Mitmest kaamerast loodud failid nimetavad igaühe eraldi.** Mitme sagedusriba pildistack (`align-apply`, `align-stream`, `index --live --save-multiband`), mitme kaamera lainealadest koostatud reaalajas indeks ning masspildistuse `composite/` pilt sisaldavad iga allikakaamera seerianumbrit, mudelit ja kalibreerimispaketti loenditena (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), ning EXIF-andmed `SerialNumber` loetlevad seerianumbrid komadega eraldatuna. Varem ei sisaldanud need üldse kaamerate andmeid või ainult esimese kaamera seerianumbrit. Vaata [Mitmest kaamerast loodud failid](#files-made-from-several-cameras).
> - **Ainult fail `raw` sisaldab kaamera kalibreerimismärke.** MicaSense-ga ühilduv komplekt (`BlackLevel`, `RadiometricCalibration`, kanalipõhine `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*` komplektid) ning `CalibrationJSON` andmekogum kirjeldavad anduri toorväärtusi, seega asuvad need ainult `raw` salvestusfailis. Kõik ülejäänud failid – salvestuse `debayered`, `display`, `radiance`, `reflectance` ja indeksifailid, `lattice continuous` ja `multi-stream` kaadrid, ning iga `process`-toode – sisaldab kaamera identiteeti (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) ning pildistamisel ka kaadri enda andmed (`XMP-mapir:FrameJSON`: säriaeg, võimendus, temperatuur, pikseliformaat, pildistamise aeg). Versioonis 1.2.0 sisaldasid need ka toorandmete komplekti, mida töötlev tööriist (Pix4D, Metashape) rakendas piksleidele, mis ei ole toorväärtused. `process` leiab sellise faili kalibreerimise selle seerianumbri ja `CalBundleSha256` abil.
>
> **Muud muudatused:**
>
> - **Massiivi automaatne kaadrisagedus kohandub vastavalt sellele, mida massiiv edastab.** Sagedus, mis kaotab rohkem kui 1 % oma triggereist, langetatakse – seda hinnatakse mitme pika vaatluse põhjal, mitte ühe põhjal –, ning sagedust, millelt see langetati, proovitakse aeg-ajalt uuesti. Kasutaja poolt ise seatud sagedust jäetakse muutmata.
> - **Automaatne säritus on stabiilsem.** See püsib konstantsel valgustasemel, stabiliseerub muutuse järel mõne sekundi jooksul, ignoreerib väikeseid ülevalgustatud alasid ja taastab kaamera töötamise, kui see on päikesepaistes reageerimise lõpetanud. Vaata [kuidas automaatne säri toimib](#how-auto-exposure-behaves).
> - **`process` loendab ainult sinu pilte ja salvestusi.** Kui käivitad selle uuesti kaustas, kus asub projekti `camera_cal/`, ei loeta neid kalibreerimisfailide enam („Leiti 96 pilti” 90-pildilises kogumis), ja projekt, mis on need juba sisse lugenud, jätab need välja.
> - **`lattice multi-test` lähtub kaamerate sisselülitatud seisundist.** See lülitab käivitamise ajaks kaamerate PTP sisse, ootab, kuni nende kellad lukustuvad ja stabiliseeruvad enne esimest päästmist — iga kaamera lukustub 120 sekundiks, umbes 2 minutit pärast sisselülitamist (`--ptp-settle SECONDS` seadistab selle), kuna äsja sünkroniseeritud kell võib umbes 2 minuti jooksul kalduda kuni ~200 us – ning taastab iga kaamera PTP-seadistuse sellisena, nagu see leiti. Kaamerat, mis on eelmisest testist veel sünkroniseeritud, ei oodata uuesti. Varem jäeti PTP välja ühe lülitiga võrgus, kus ükski DAQ-E ei vastanud, ning loeti iga kaamera vabalt töötavat kella PTP-kellana, mis ei lukustunud (`PTP is not locked`, iga käivituse puhul). Vaata [Mitme kaameraga (ajutised seansid)](#multi-camera-transient-sessions).
> - **`.daq` salvestus näitab, kas kaamera kate valiti või eeldati selle olemasolu.** DAQ-U, DAQ-M ja DAQ-E tarnitakse ilma korgita ning Chloros ei näe, kas see on paigaldatud, seega saab nimetamata korgita ühendatud andur vaikimisi väärtuse `sunshine_cosine`, mis on *oletus*. Iga Chlorosi salvestus (skeem 1.30) salvestab nüüd `cap_id_source`-i kõrval ka `cap_id`-i: `operator`, kui andsite korgile nime (sh `none`), `auto_default`, kui see oli vaikimisi. Kui töödeldakse salvestust, mille korgi olemasolu eeldati, ning projekti DAQ-korgi seadistuseks on jäetud „Auto“, kuvab veateate `[daq-dls] WARNING`, milles nimetatakse faili: paljas DAQ-U päikesekorrektsiooni all näitab umbes 30 korda suuremat väärtust. **Nimetage ühendamisel paigaldatud piirang** või seadistage projekti DAQ-piirang, ja hoiatus kaob.
> - **Windowsis ei ilmu enam pip-konsooliaknaid.** Pythoniga arvutis käivitasid backend ja iga selle pilditöötleja käivitamisel korra veateate `pip`, ja igaüks avas konsooliakna. Nüüd seda enam ei juhtu.
> - **`.daq` reaalajasalvestused säilivad ka voolukatkestuse korral.**Salvestus kirjutatakse iga kinnituse korral koos päevikuga ja täieliku kettasünkroniseerimisega anduri taga, seega aeglane ketas viivitab faili salvestamist, kuid mitte kunagi mõõtmistulemusi. Salvestus, mis peatati keset salvestamist (voolukatkestus, protsessi lõpetamine), võib jätta selle kõrvale `<name>.daq-journal`:**kopeerige need kaks kokku.** Chloros taastab ajakirja esimesel faili lugemisel, seega avaneb salvestus koos kõigi enne peatamist kinnitatud andmetega (kaotatud on umbes viimased 3 sekundit); versioonis 1.2.0 võis valel hetkel toimunud voolukatkestus jätta kogu faili loetamatuks („andmebaasi kettapilt on vigane“).
> - **Salvestamine hõivatud kettale ei põhjusta enam anduri seiskumist.** `daq pool-record` paneb lugemised järjekorda ja kirjutab need eraldi protsessis; lugemine, mida ei suudetud kirjutada, loetakse (`daq pool-list` näitab `N write error(s)` viimase veaga) selle asemel, et andurit külmutada (üks kirjutamine võttis aega 8,4 sekundit, kui neli kaamerat salvestasid TIFF-faile).
> - **`lattice multi-capture` ilma massiivita ei kaota enam salvestatud kaadreid, kui päästiku aegumisaeg lõpeb.** Kaamera trigeri aegumine (`SC_ERR_TIMEOUT -1011`, täheldatud nelja kaameraga ühel 1 GbE lülitil) lõpetab töötamise selle kaadrirühma juures: juba salvestatud rühmad on loetletud koos `Saved:`-iga, seejärel ilmub viga, väljumine `1`. Versioonis 1.2.0 oli tegemist HTTP-veaga 500 ilma nimekirjata, kuigi failid olid kettal olemas.
> - **Tarkvarapõhiselt käivitatavad kaamerad ei jäta enam iga teist käivitust vahele.** Tarkvarapõhises käivitamisrežiimis olev kaamera paneb nüüd järjekorda käivituse, mis saabub selle väljundperioodi jooksul (või Chloros ootab, kuni see on valmis), nagu seda tegid juba kaabli kaudu sünkroniseeritud kaamerad.
> - **`daq pool-discover` nimetab DAQ-E-S-i `DAQ-E-S`** veergus `MODEL` (ja `--json` veerus `model`). Iga Etherneti anduri puhul oli kirjas `DAQ-E`; kuid see, mis ei vasta identiteedikontrollile umbes 2 sekundi jooksul, on endiselt loetelus nimetusega `DAQ-E`.
> - **Täiustatud LATTICE indeksitooted** (NDVI ja muud indeksid) alates `process`-st, kõigis töötlemisrežiimides.
> - **OCN-kaamerad: täiustatud kiirguse, peegelduse ja indeksitooted päikesevalguses.** Pärast lainepikkuste eraldamist rakendatakse seadmepargi korrigeerimist; RGN-kaamerad jäävad muutumatuks ning OCN-väärtused erinevad versioonist 1.2.0. See kehtib ka varasema versiooniga tehtud OCN-pildistuse puhul, kui seda töödeldakse arvutis, milles on salvestatud pildistamise ajal kasutatud kalibreerimine (Chloros säilitab kaamera kalibreerimise pärast kaamera ühendamist).
> - **`lattice continuous` salvestab töötlemata kaadri.** `tiff` käivitamisel kirjutati demosaicitud 3-tasandilist kaadrit, millest `process` ei saa alustada. Selle kaadrid on nüüd `raw`-fail, mille massiivpildistamine kirjutab: üks mosaiikitasand anduri enda väärtustega, tihendamata, koos kalibreerimispaketiga. Skript, mis loeb neist failidest kolme tasandit, loeb vaid ühte.
>
> - **Iga salvestus salvestab kasutatud DAQ-näidu, nagu seda teeb ka graafiline kasutajaliides.** `lattice capture`, `lattice continuous` (üks `.daq` iga kaadri kohta), `array-capture` ja `multi-capture` koos ühendatud massiiviga (ja SDK-ga) kirjutavad failid `.daq` ja selle `.match.json` iga kord, kui DAQ seotakse või koondatakse, sealhulgas ka ainult toores pildistamine; versioonis 1.2.0 salvestati toores või debayered pildistamine ainult `--force-daq`-iga. `--no-daq` lülitab selle välja, `--daq-tolerance-ms MS` määrab selle pildistamise sobitustolerantsi ja uus `daq pool-bind` seob kaamera või massiivi DAQ-iga CLI kaudu. `.daq` salvestatud pildistamine sobitatakse `process`-is ainult nimetatud kaameratega. Salvestus ootab nüüd DAQ-i järgmist lugemist, et saaks ühendada ekspositsiooni mõlemal poolel tehtud lugemised. Vaata [DAQ-i lugemine, mille salvestab salvestus](#the-daq-reading-a-capture-saves) ja [DAQ-i lugemine kaugel ekspositsioonist](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** lisab projekti indeksid iga kaamera lainealade jaoks, sealhulgas uue projekti. Versioonis 1.2.0 ignoreeriti `Unknown index name(s) ignored` abil projekti nimekirjas veel puuduvat nime.
> - **`process` reaalajas töötava massiivi kõrval:**see ei taaskäivita enam kaameraid või DAQ-seadmeid haldavat tagapõhja ning ootab kuni 15 sekundit lõppeva käivituse lõppemist.**Ctrl+C** peatab käivituse tagapõhjas, mitte ainult CLI-s. Vaata [`process` märkusi](#notes).
> - **Windowsi 260-tähemärgilise teepikkuse piirangust liiga sügav projektikaust** lükatakse kohe tagasi, kasutades selle asemel pikimat sobivat kausta, selle asemel et käivitada käivitust, mis midagi ei kirjuta. Windowsi installija pakub nüüd valikut „Luba Windowsi pikad teed” ja Microsoft Defenderi erandit backendile, mõlemad on vaikimisi märgitud. Vaata [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** käivitab tagapõhja enda pideva salvestamise: iga 2 sekundi järel kuvatakse edenemisriba, failide loetelu asemel kuvatakse kokkuvõte ning programm lõpetatakse veakoodiga, kui mõni fail jääb kirjutamata või kui kõvaketas on täis. Vaata [`array-capture` režiime](#array-capture-modes).
> - **Esimene `array-capture` pärast `array-connect`** ootab kuni 20 sekundit massiivi esimest kaadrirühma, selle asemel et anda veateadet HTTP-koodiga 503 „Massiiv soojeneb veel”.
> - **Massiivi ekspositsioon ROI all:** väike ROI ei piira enam massiivi liikme ekspositsiooni anduri vabajooksuperioodi pikkusega (4,9 ms resolutsioonil 768 × 304). Vaata [Ekspositsioon massiivi liikmel](#exposure-on-an-array-member).
> - **Automaatne valgustusaeg ei vähenda enam maatriksi automaatset kaadrisagedust.** Kaabli kaudu sünkroniseeritud maatriks töötab sagedusega, mida võimaldavad võrk ja kaamerate andmete väljalugemine; automaatne valgustusaeg täidab selle sagedusega antud perioodi ja suurendab seejärel võimendust. Käsitsi valgustusaeg, mis on pikem kui periood (`lattice exposure --set`, `configure --set`), aeglustab päästikut, et see sobiks, selle asemel et seda keeldutaks, ning kaadrisagedus taastub, kui element naaseb automaatrežiimile. Vaata [Kaadri sünkroniseerimine massiivi liikmes](#exposure-on-an-array-member).
> - **Massiivid annavad vähem puudulikke kaadrirühmi** ja vähem 503-vigu („Puudub uus sünkroniseeritud kaadrirühm”): iga orjakaamera päästiku sisend on nüüd glitchidest vabastatud.
> - **Pakitud 12-bitine andmeedastus** (`--packed`, vaikimisi `auto`) suurendab täissensoriga massiivi kaadrisagedust umbes kolmandiku võrra. Nüüd on kasutusele võetud `array-connect --packet-size` (versioon 1.2.0 aktsepteeris seda, kuid ignoreeris). Vaata [Pakitud 12-bitine andmeedastus](#packed-12-bit-transport---packed).
> - **Massiiv, mille kaamerad taastuvad ise pärast toite väljalülitamist ja sisselülitamist.** Pärast lüliti toite väljalülitamist ja sisselülitamist avab `array-connect`-i massiiv-avab kaamerad uuesti, kui need vastavad, ja alustab uuesti voogedastust, kui PTP on taas sisse lülitatud; versioonis 1.2.0 peatus see lõplikult ja vajas uut `array-connect`-i. `array-status` teatab `ptp_enabled: False` (ja `ptp_off:`), kui kaamera loeb PTP-d välja, kus varem teatas see seadistust, millega massiiv oli ühendatud. Vaata [Püsivad ühendused](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` salvestab, kui graafiline kasutajaliides ei ole avatud.** Kui graafilise kasutajaliidese kombineeritud vaade on suletud, renderdab tagapõhi vaate ise: massiiviNDVI või indeks, mille edastate uue `--expr`-iga. Salvestus, mis ei kirjuta ühtegi kaadrit, ebaõnnestub nüüd (`no-frames: …`, väljumine `1`). Varem salvestati ainult siis, kui graafilise kasutajaliidese kombineeritud vaade oli avatud, ja salvestus, mis midagi ei kirjutanud, lõpetas ikkagi `0`. Vaata [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Kaamerat, mida Chloros juba hoiab, ei avata teist korda.** Käsk, mis seda ise avaks, vastab selle asemel HTTP-koodiga 409 (`CAMERA_HELD`), nimetades selle, kes seda hoiab, ja käsu, mis selle vabastab. See kehtib ka kaamera kohta, mida teine käsk parasjagu kasutab (nt `multi-test` või `multi-capture`, mis avab oma kaameraid, või `lattice capture`): teine käsk lükatakse tagasi, kuni esimene on lõpetanud. Selle uuesti avamine võib peatada hoidja voo või põhjustada tagapõhja kokkujooksmise. Vaata [Püsivad ühendused](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Ajutised mitme kaamera käsud edastavad kõik kaamerad või ebaõnnestuvad.** `lattice multi-capture`, millele pole ühendatud ühtegi kaamerat, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` ja `index --live` lõpetavad töötamise, `1` nimetab kaamerat, mis ei edastanud ühtegi kaadrit. Vood jätavad vahele kaamerat puuduva kaadrirühma ja peatuvad pärast kolme järjestikust; `multi-test` väljastab ühe rea iga trigeri kohta, `(no frame)` näitab, kus kaamera jäi vahele; `align-stream` lõpetab samuti töö `1` töövea korral. Iga käivitamine seab seejärel iga kaamera tagasi selliseks, nagu see leiti: selle triggereid ja sünkroniseerimisliini ühendused, selle PTP (`PtpEnable`) ning kõik käivituse käigus muudetud kaamera seaded – säri ja võimendus, AE sihtmärk, piirid ja summutamine, valgebalanss, must tase, pikseliformaat, binning ja ROI, pööramised, kaadrisagedus –, välja arvatud juhul, kui `--keep-settings` / `--clear-settings` (vt [Kaamera mälu](#camera-memory----keep-settings----clear-settings)). Varem säilitasid need käsud kõik saabunud kaadrid ja väljusid `0`-ist (`align-stream` kuvas töövea kui `Note:`), ja jätsid kõik kaamerad `TriggerMode` `On`-i. Samuti seadistasid need iga kaamera säriaja eelmise kaadri järgi (kõige rohkem paar sekundit), kusjuures ühendatud massiiv käivitab iga kaamera järgmiste seadistustega: 40 multispektraalset, 60 RGB-kaadrit (`--brightness N` seadistab selle) ning hoiab seda kogu töö jooksul; sama teeb ka üksik `lattice capture` kaamera puhul, mida Chloros ei toeta. Varem võeti iga kaamera esimene kaader kaamera enda säriaja piiranguga, umbes 100 ms, olenemata valgustingimustest, ning see oli sageli kärbitud. Vaata [Mitme kaamera (ajutised seansid)](#multi-camera-transient-sessions).
> - **Ajutised mitme kaamera käsud jagavad võrku samamoodi nagu `array-connect` ja avavad ainult `--serials`.** Iga kaamera töötab vastavalt oma osale võrguühendusest, seega töötavad neli kaamerat 1 Gbps ühendusel; varem nõudsid need neli kaamerat kokku 125 MB/s, kõik 1 GbE kaudu, ning igas kaadris kadusid paketid. Võrk lükkab tagasi sellise massiivi, mida see ei suuda edastada, enne kui ükski kaamera hakkab voogedastama. `--serials` valib nüüd kaamerad, millest esimene käivitab trigeri; varem avas iga käsk kõik võrgus olevad kaamerad, hoolimata sellest, mida `--serials` ütles. GVSP-pakettide kadu näitab nüüd seda, selle asemel et osutada M8 sünkroniseerimiskaablile. Vaata [Mitme kaamera (ajutised seansid)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` jätab kaamerad vabalt töötama, nagu need leiti.** `TriggerMode`, `Off` ja kõik muu nii, nagu ühendus selle leidis — päästiku ja sünkroniseerimisliini seaded (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP ning seansi käigus muudetud kaamera seaded – nii et üksainuskaamerakäsk endise liikme suhtes toob kaadrid korraga sisse ning juhtkaamera ei jää enam `TriggerSource` `Software`-le. `--keep-settings` salvestab seansi seaded iga kaamera mällu, `--clear-settings` lähtestab selle mälu (vt [Kaamera mälu](#camera-memory----keep-settings----clear-settings)). `lattice continuous` seab ka kaamera, mille ta leiab olevat aktiveeritud, jooksu ajaks vabajooksule (ja lülitab selle pärast tagasi), ning jooks, millel puudub raam, väljub `1`-ist.
> - **`lattice configure --get` / `lattice exposure`** väljastavad `note:`-i koos kaamerategeliku säriaja või võimendusega, kui kaamera töötab automaatrežiimis. Keeldutud `lattice exposure --set` / `configure --set` näitab kaamera aktsepteeritavat vahemikku ja lülitab automaatrežiimi tagasi, selle asemel et jätta automaatne säri välja lülitatuks. Vaata [Anduri häälestamine](#sensor-tuning).
> - **Uued kaamera juhtimiskäsud:**`lattice highlights` (vähendab heledate valgete pindade heledust RGB-kuva renderdamisel) ja `lattice exposure --brightness-auto`.**`lattice multi-detect`** leiab ühe proovimisvooru jooksul kaabliga ühendatud kaamerad ja määrab peakaameraks vaikimisi RGB-kaamera.
> - **Uued DAQ-käskud:** `daq pool-bind`; `pool-enable-ptp`; ning DAQ-E enda seadistuste jaoks `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` ja `pool-update-firmware`. Vaata [DAQ-E seadme seaded](#daq-e-device-settings).
> - **Backend ühendab uuesti DAQ-E-seadme, mis on andmete edastamise lõpetanud.** Kui ühendatud DAQ-E-seadme voog vaikib (seda toitva lüliti taaskäivitamine, kaameraühenduse katkemine), ühendab backend selle uuesti varasemate seadistustega, proovides ühendust üha pikemate intervallidega, kuni seade on kadunud. Salvestamine jätkub samas `.daq`-is, kusjuures vahe on failis märgitud, ning `pool-record --stop` loetleb iga katkestuse. Versioonis 1.2.0 jäi andur vaikseks kuni `pool-disconnect` + `pool-connect`, mis lõpetas ka salvestamise. Vaata [DAQ-E, mis lõpetab andmete edastamise](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** teatab, kas PTP-grandmasteri paketid jõuavad võrku (`health`), ning kui need ei jõua, kuvatakse stderr-is `WARNING:`. Vaata [`time-sync`](#chloros-cli-time-sync).
> - **Linuxi .deb:** see suurendab tuuma sokli vastuvõtupuhvreid, mida kaameravoogudele vaja on (tuuma vaikimisi seadistuse korral kaotatakse kaadreid ja kaadrisagedus), lisab tagapõhiteenuse, mis töötab teie kasutajana, ning `lattice network` kontrollib puhvreid. CLI-keskkond ei peata kunagi teenusena töötavat tagapõhja ning uuendamine käivitab aktiveeritud tagapõhja teenused uuesti. Vaata [Tagapõhja käitamine teenusena](#running-the-backend-as-a-service-headless) ja [Linux-hostid — pistikupesa vastuvõtupuhvrid](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` teatab, kui joondamine ei õnnestunud:**iga põhjus, miks salvestus jäi joondamata (puudub joondamisprofiil, liikmed erineva resolutsiooniga, massiiv ei ole kaabli kaudu sünkroniseeritud, raam, millele profiili ei saa kaardistada), kuvatakse `Alignment:`-real.**`chloros-cli daq help`** on `daq --help` (see lõpetas `2`) ning Linuxis loetletakse abis otsesed riistvarakäsud `daq` , mis on selles koostes sisalduvad.
> - **Jetson:** pakett on `chloros_1.3.0_arm64_jp6.deb`, mõeldud JetPack 6 jaoks, ning selle GPU-töötlus toimub Orinil JetPacki CUDA PyTorchi abil. Vaata [Linuxi arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konventsioonid

- Kõik käskudele on eesliiteks `chloros-cli`. Windowsis on binaarfail `chloros-cli.exe`; Linuxis/Jetsonil on see `chloros-cli`.
- Valikulised argumendid on esitatud kujul `--flag`. Nõutavad positsioonilised argumendid on esitatud ilma sulgudeta.
- Kui on antud vaikimisi väärtus, kasutatakse selle väärtust, kui lipukest ei märgita.
- CLI on HTTP-kliendi kerge versioon, mis töötab Chloros-tagapõhja kaudu (server aadressil `127.0.0.1:5000`, mida kasutab ka töölaua rakendus). **Millised käsud seda käivitavad:**
  - Põhikäsud — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` ja `selftest` — kasutavad juba töötavat backendit või käivitavad selle, kui ükski ei tööta, ning peatavad selle uuesti, kui töö on lõppenud.
  - **Käsud `lattice`, `project` ja `daq` ei käivita seda kunagi.** Need vajavad juba töötavat backendit: Windowsi töölaua rakenduse oma, Linuxi [backend-teenust](#running-the-backend-as-a-service-headless) või seda, mille käivitad ise (vt [Backendi käivitamine](#start-the-backend)). Ilma backendita kuvavad need veateate `Error: Chloros backend not reachable at http://127.0.0.1:5000` koos selle käivitamise käsuga ja lõpetavad töö `1`.
- `CHLOROS_BACKEND_URL=<url>` suunab **`lattice`**,**`project`**ja**`daq pool-*`** teise tagapõhja (`lattice network-analysis` suunab selle asemel `--backend-url`-i). Põhikäsud fikseerivad tahtlikult `http://127.0.0.1:<port>`-i ja ignoreerivad seda (IPv4-literaal vältib Windowsi `localhost`→`::1` ~2 s-per-request karistust). Vaata [Keskkonnamuutujad](#environment-variables).
- **Suhtelised teed lahendatakse selle kausta suhtes, kust `chloros-cli` käivitatakse.** Tagapõhi on protsess, mis avab väljundi- ja sisendkaustad, seega enne tee saatmist muudab CLI selle absoluutseks: `-o` / `--output` (sealhulgas selle vaikimisi väärtus, `output`), `--input`, `--burst-dir`, `--profile`, `--save` ja `--mask`, [retsepti](#recipe-dsl) sammu `output` (vaikimisi `output`)ning `lattice index --live` vaikimisi kaust `output`. Seetõttu kirjutab `-o captures/` kausta `<current folder>/captures/` ning väljastatud teed on absoluutsed. `~` laiendatakse. Kaug-backendi puhul (`CHLOROS_BACKEND_URL`, mis viitab teisele masinale) saadetakse teed täpselt nii, nagu need on sisestatud, ja need lahendatakse selles masinas.
- Kõikide SDK/CLI kõikide kutsete jaoks (käivita `chloros-cli login` üks kord iga masina kohta; salvestatakse vahemällu `~/.chloros/`).
- Näidetes kasutatakse Linuxi teekondi; Windowsis asenda `/home/user/...` `C:/Users/.../...`-iga. DAQ-U'sariport on Windowsis `COM3`-tüüpi ja Linuxis `/dev/ttyUSB0` / `/dev/ttyACM0`.
- Ülevaadetes kirjutatakse üksteist välistavad lipud `{--a | --b}` ja valikulised lipud `[...]`-na.

---

## Ülemine sünopsis

```
chloros-cli [global options] COMMAND [command options]
```

### Üldised valikud

| Lipu nimi | Kirjeldus |
| --- | --- |
| `--backend-exe PATH` | Asenda automaatselt tuvastatud tagapõhja käivitatav fail. |
| `--port N` | Tagapõhja HTTP-port (vaikimisi: `5000`). Kasutatakse ainult põhikäsudega (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` ja `daq` kasutavad `CHLOROS_BACKEND_URL` (vaikimisi `http://127.0.0.1:5000`). |
| `-v, --verbose` | Lülita sisse üksikasjalik väljund. |
| `--restart` | Sundida tagapõhja taaskäivitamist (lõpetab kõik töötavad `backend_server.py`-id). Süsteemiteenuse poolt käitatavat tagapõhja (Linux .deb) ei peata kunagi: käsk väljastab selle asemel `sudo systemctl restart <unit>` ja lõpetab töö `1`. |
| `--version` | Kuvab versiooni (`Chloros CLI 1.3.0`). |
| `--help` | Kuvab ülemise taseme abi. |

### Käskude indeks

| Käsk | Otstarve |
| --- | --- |
| [`process`](#chloros-cli-process) | Töötleb kaustat, mis sisaldab Survey3 või „LATTICE” salvestusi algusest lõpuni. |
| [`login`](#chloros-cli-login) | Autentida see arvuti Chloros+ kontoga. |
| [`logout`](#chloros-cli-logout) | Tühjenda vahemällu salvestatud autentimisandmed. |
| [`status`](#chloros-cli-status) | Näita praegust litsentsi / autentimise staatust. |
| [`export-status`](#chloros-cli-export-status) | Näita Thread-4 ekspordi käiku reaalajas `process` käivituse ajal. |
| [`language`](#chloros-cli-language) | Määra või näita CLI-i kuvakeelt (toetatakse 38 keelt). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Vaikimisi projektikausta (jagatud GUI-ga). |
| [`update`](#chloros-cli-update) | Chlorosi värskenduste otsimine ja installimine (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Süsteemi diagnostika + kiirtestid. |
| [`time-sync`](#chloros-cli-time-sync) | PTP grandmasteri staatus / juhtimine. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-kaamera juhtimine ja pildistamine (üle 45 alamkäsu). |
| [`daq`](#chloros-cli-daq) | DAQ spektraalsensorite juhtimine (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Salvestatud Chloros-projekti (kaamerad + DAQ-seadmed) avamine ja käivitamine. |

---

## Paigaldamine

`chloros-cli` on kaasas Chlorosi töölaua paigaldusprogrammis kõigil toetatud platvormidel — eraldi CLI-versiooni ei ole võimalik alla laadida. Platvormipaketi paigaldamisel lisatakse `chloros-cli` teie `PATH`-i koos töölaua rakenduse ja selle poolt juhitava tagapõhja binaarfailiga.

Viimased allalaadimised: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Paigaldaja sisaldab ka mugavaid käivitusskripte (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), mis avavad kohe kasutusvalmis CLI-keskkonna.

### Windows (.exe)

1. Laadige Windowsi paigaldaja alla allalaadimislehelt.
2. Käivitage `Chloros Installer 1.3.0.exe` ja järgige paigaldusviisardi juhiseid. Vaikimisi paigalduskataloog on `C:\Program Files\Chloros\` (CLI-keskkond asub kataloogis `C:\Program Files\Chloros\cli\`, mille paigaldaja lisab muutujasse PATH).
3. Avage uus terminal (`cmd.exe`, PowerShell või Windowsi terminal), et uuendatud `PATH` saaks tuvastatud.

```powershell
chloros-cli --version
```

Paigaldaja lisab automaatselt `chloros-cli.exe` teie süsteemi `PATH` ja lisab LATTICE kaameradraiverid. Tagapõhi on `C:\Program Files\Chloros\backend\chloros-backend.exe` (teie installikausta kaust `backend`), mida `PATH`-is ei ole.

**Süsteemi seaded.** Paigaldaja lehel *Valige lisategevused* on **Süsteemi seaded**all kaks valikut, mõlemad**on vaikimisi märgitud**:

| Valik (nagu paigaldajas sõnastatud) | Mida see teeb | Vaikimisi paigaldamise tühistamine |
| --- | --- | --- |
| **Luba Windowsi pikad teed** (võimaldab projektikausta asuda sügavamal kui 260 tähemärki; soovitatav) | Seab Windowsi kogu arvutit hõlmava parameetri `LongPathsEnabled` väärtuseks 1. Windowsis on see vaikimisi välja lülitatud, ning sel juhul ei tohi ükski tee ületada 260 tähemärki; Chlorosi tooted asuvad kuni umbes 130 tähemärki projektikausta all, seega keeldutakse sügavamatest projektikaustadest (vt [Vigade kõrvaldamine](#troubleshooting-pointers)). Jõustub pärast installimist käivitatud programmide puhul. Jätke see desinstallimisel sisse lülitatuks, kuna teised programmid võivad sellele tugineda. | `/MERGETASKS="!longpaths"` |
| **Jäta Chlorosi taustaprogramm Microsoft Defenderi skaneerimisest välja**(palju kiirem kaamerapildistamine; soovitatav) | Lisab installikausta `chloros-backend.exe` jaoks Defenderi**protsessi** välistuse, nii et reaalajas kaitse ei skaneeri iga faili, mida taustaprogramm kirjutab, olenemata sellest, kuhu ta selle kirjutab. Muud programmid ei jää välja; välistamine eemaldatakse desinstallimisel. Kui Defender puudub või on poliitika tõttu lukustatud, ebaõnnestub see samm märkamatult ja installimine jätkub. | `/MERGETASKS="!defenderexclusion"` |

**Miks Defenderi valik on oluline:** kõrgel salvestussagedusel kulub salvestusfailide reaalajas skannimisele umbes 30 % salvestussagedusest (mõõdetud 9 kaameraga pideva salvestamise korral: 77,7 salvestust/s jäeti välja, 52,7 skanniti). `lattice array-capture --continuous` kuvab teate, kui mõõdab, et Defender kasutab salvestamisel ühte või enamat CPU tuuma; käivitage paigaldaja uuesti, valides selle valiku.

Vaikimisi installimine (`/SILENT` või `/VERYSILENT`) rakendab mõlemat, kui seda ei ole keelatud, nt `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Windowsi tulemüür.** Paigaldaja lisab alati igasse võrguprofiili kaks sissetulevat reeglit `chloros-backend.exe` jaoks ning eemaldaja eemaldab need: `Chloros Backend (UDP-In)` kaameravoogude, PTP, mDNS ja DAQ-E andmevoo jaoks ning `Chloros Backend (TCP-In)`, mis on piiratud kohaliku alamvõrguga, DAQ-E püsivara uuendamiseks, mille käigus andur ühendub tagasi selle arvutiga pildi kogumiseks (vt [DAQ-E seadme seaded](#daq-e-device-settings)). Tagapõhja HTTP-API jääb seotuks `127.0.0.1`-iga.

### Linux amd64 (.deb)

Ubuntu 22.04 LTS või uuemate versioonide / Debianil põhinevate x86_64 tööjaamade jaoks.

> **Ubuntu 20.04 ei ole toetatud.** Paketi sõltuvuste nimekiri tuleneb
> sellest, millega tagapõhi tegelikult seotud on, ja see hõlmab `libc6 (>= 2.35)`-i;
> Focal tarnib glibc 2.31. `apt` keeldub installimisest, selle asemel et lasta sellel
> käivitamisel ebaõnnestuda. Nimekirjas on nimetatud ka `libgl1`, mida komplektis olev OpenCV laadib ja mida minimaalses
> serveri kujutises võib puududa; `apt install` lisab selle.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

.deb-pakett installib:
- `chloros-cli` kuni `/usr/bin/chloros-cli` ja `chloros-backend` (käivitab tagapõhja) kuni `/usr/bin/chloros-backend`
- Kompileeritud backend ja CLI, lahtipakitud, asuvad `/usr/lib/chloros/backend/` ja `/usr/lib/chloros/cli/`
- LATTICE’i kaameradraiverid
- Müra vähendamise mudelid, kalibreerimispaketid ja uuenduskanali konfiguratsioon
- Süsteemi Pythoni SDK `python3` (`import chloros_sdk`). `apt install` toob kaasa ka oma nõuded: `python3-numpy` ja `python3-requests` (soovitatav, mitte kohustuslik: eespool nimetatud programmid on iseseisvad; `dpkg -i` jätab need vahele ning installimise väljund näitab, mis puudub). Virtualenvis: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
– `/etc/sysctl.d/60-chloros-gige.conf`, mis suurendab kaamera voogude jaoks 32 MB-ni ja rakendab need kohe (juba suuremaks seatud väärtust ei muudeta). Paketi eemaldamine kustutab selle faili. Vaata [Linux-hostid — pistikupesa vastuvõtupuhvrid](#linux-hosts--socket-receive-buffers).
- Kaks valikulist systemd-teenust tagapõhja jaoks, kumbki pole aktiveeritud (järgmine jaotis)

#### Tagapõhja käitamine teenusena (ilma graafilise kasutajaliideseta)

Ilma teenuseta töötab tagapõhi (ja koos sellega PTP-grandmaster) ainult siis, kui keegi on selle käivitanud: `chloros-backend` terminalis, SDK-skriptis või core-käsk `chloros-cli` selle töötamise ajaks. Alati töötava tagapõhja jaoks ekraanita masinas aktiveerige teenus **oma kasutajana**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` käivitab backend'i selle kasutajana. Teenuse poolt salvestatud andmed kuuluvad sellele kasutajale, teenuse üksikasjalik logi asub tema `~/.cache/chloros/logs`-is, ning ta kasutab selle kasutaja `chloros-cli login` (`~/.chloros`) kontot, seega logi esmalt korra sisse selle kasutajana. PTP ei vaja root-õigusi. Nagu iga installitud backend, kuulab ta ainult `127.0.0.1:5000`-i.

`chloros-backend.service` käitab tagapõhja **root**-õigustes. See on endiselt paigaldatud nii, et masin, kus see on aktiveeritud, jätkab töötamist, kuid kõik, mida root-tagapõhi kirjutab, kuulub rootile, seega ei saa kasutaja oma salvestusi kustutada. Oma kasutajaks vahetamiseks:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Root-teenuse üksikasjalik logi asub failis `/var/log/chloros`, mida saavad lugeda rühma `adm` liikmed (teenus seab `CHLOROS_LOG_DIR`; vt [Keskkonnamuutujad](#environment-variables)). Paketi installimisel kuvatakse käsk `enable`, millesse on sisestatud teie kasutajanimi, ning samuti lüliti, kui root-teenus on aktiveeritud.

**Ühe tagapõhjaüksuse aktiveerimine.** Korraga töötab ainult üks tagapõhiüksus. Teine aktiveeritud üksus (`chloros-backend.service` lisaks `chloros-backend@<user>.service`-ile või kahe kasutaja `@`-üksused) keeldub käivitumast ja jääb seisundisse `failed` väljumisstaatusega 98; systemd ei taaskäivita seda. Selle `systemctl status` näitab põhjust: `Another instance is already running. Exiting.`, seejärel seda, mis tagab backendit (port 5000 või lukufail ja selle omanik), siis `systemd will not restart <unit> (exit status 98)`. Pärast taaskäivitamist võib töötada kumbki üksus. Vahetamiseks lülita välja see, mida sa ei kasuta, nagu eespool kirjeldatud.**CLI ei peata kunagi teenuse backendit.** `chloros-cli` kasutab seda sellisena, nagu seda teevad ka GUI ja SDK: `process` töötab selle peal, laadides viimase projekti asemel omaenda projekti. Kui CLIil oleks muidu vaja tagapõhja uuesti käivitada (`--restart` või `process`, mille importimisel midagi ei leitud), siis nimetab ta mooduli, väljastab `sudo systemctl restart <unit>` ja lõpetab `1`, sest sel viisil peatatud tagapõhi jääks seisma. `GET /api/status` nimetab üksuse `service`-iks (`null` juhul, kui tagapõhis ei jookse ühtegi teenust).**Uuendamine ja eemaldamine.** Paketi uuendamine peatab kõik tagapõhja teenused, seejärel käivitab uuesti kõik aktiveeritud teenused, alustades `@<user>`-üksustest (üks, mis on käsitsi käivitatud, kuid kunagi aktiveerimata, jääb peatatuks, nagu pärast taaskäivitamist). Kui aktiveeritud on rohkem kui üks, töötab ainult üks ja teised jäävad ebaõnnestunuks nagu eespool kirjeldatud; uuendamine kuvab veakoodid `Chloros: N backend units are enabled: …` ja `Keep one enabled: sudo systemctl disable --now <unit>`. Paketi eemaldamine peatab ja deaktiveerib need. Selle täielik kustutamine eemaldab ka `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Sama ülesehitus kui amd64 .deb-failil, sisaldab vastuvõtupuhvreid ja [tagapõhiteenuseid](#running-the-backend-as-a-service-headless), mõeldud Jetson Orinile / Orin NX-ile / Orin Nanole **JetPack 6**-l (L4T R36). Selle PyTorch on JetPack 6 CUDA-koostis Orini jaoks, seega töötab GPU-töötlus, sealhulgas tekstuuritundlik debayer, Jetsoni GPU-l. Pakett ei sisalda CUDA-t ennast: see kasutab CUDA- ja cuDNN-teeke, mille JetPack 6 installib (`sudo apt install nvidia-jetpack` pildil, millel need puuduvad). Kui CUDA-GPU-d ei ole võimalik kasutada, toimub töötlemine CPU-l ja tekstuuritundlik funktsioon kasutab standardset debayeri (vt [`process`](#chloros-cli-process)).

### Autentimine üks kord iga masina kohta

Igal platvormil on vaja teha ühekordne Chloros+ sisselogimine, enne kui SDK- ja CLI-kutsed hakkavad toimima:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Autentimisandmed salvestatakse vahemällu failis `~/.chloros/user_session.json`.

### Tagapõhja käivitamine

Käsud `lattice`, `project` ja `daq` suhtlevad juba töötava tagapõhjaga; nad ei käivita seda kunagi (vt [Konventsioonid](#conventions)). Hoia üks neist töös, kui neid kasutad:

| Platvorm | Kuidas |
| --- | --- |
| Windows | Avage Chlorosi töölaua rakendus (selle backend teenindab ka CLI-d) või käivitage `"C:\Program Files\Chloros\backend\chloros-backend.exe"` eraldi terminalis (teie installikausta kaustas `backend`) ja jätke see avatuks. |
| Linux / Jetson | Käivita `chloros-backend` eraldi terminalis või hoia üks alati töös [tagapõhiteenuse] abil (#running-the-backend-as-a-service-headless). |

Korraga töötab ainult üks backend; kõik kliendid (GUI, CLI, SDK-skriptid) jagavad seda. Ka põhikäsud (`process`, `login`, `status`, …) kasutavad seda ja käivitavad ta, kui ükski ei tööta.

**Backend töötab käskude vahel edasi**, seega järgmine käsk ei pea seda uuesti käivitama. See peatub, kui logite välja (`chloros-cli logout`, välja arvatud juhul, kui Chloros-rakendus on avatud või käimas on käivitus või offline-koostamine), kui käivitate `chloros-cli backend stop` või kui see peatub ise 30 minuti pärast, kui seda ei kasuta: ei käsku, skripti ega rakendust, ei käivitust ega offline-koostamist, ei ühendatud kaamerat ega valgussensorit. `backend stop` keeldub, kui käivitamine või offline-koostamine on käimas, nimetades selle; `--force` peatab selle kohe ja see käivitamine või koostamine läheb kaduma (pildiseeria säilitatakse ja `array-build-video` koostab selle uuesti). Määrake `CHLOROS_BACKEND_IDLE_MIN` enne, kui tagapõhi hakkab 30-minutilist intervalli muutma (`0`: kunagi); [tagapõhi teenus](#running-the-backend-as-a-service-headless) ei peatu kunagi iseenesest. `chloros-cli backend status` näitab, kas mõni on käimas, mis seda kasutab ja kas mõni offline-koostamine on käimas. `process` avab oma projekti käimasolevas backendis, välja arvatud juhul, kui Chloros-rakendus on avatud projektiga, mille : sel juhul keeldub ta seda tegemast ja peate esmalt töötlema rakendusest või sulgema selle projekti. Projekt, mille varasem `process` või SDK-skript avatuks jättis, asendatakse lihtsalt.

### Kontrolli installi õigsust

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` sulgeb `0` alles siis, kui kõik seitse kontrolli on edukalt läbitud. Viimane kontroll nõuab CUDA-graafikakaarti (NVIDIA graafikakaarti, või Jetson Orini oma) ja müra eemaldamise mudeleid, seega arvutis, kus seda pole, kuvatakse `Failed: CUDA + Denoiser` ja programm lõpetab töö `1`; seal töötab endiselt kõik peale GPU-põhise tekstuuritundliku debayeri.

> **Vajalik on Chloros+ tellimus.**CLI vajab aktiivset Chloros+ paketti.**Copper**on Chloros+ algtaseme pakett – igal tasulisel Chloros+ paketil on juurdepääs CLI-le ja SDK-le; ainult tasuta**Iron**-paketil seda pole. (Paketi-ID vastavus: `0`=Iron/tasuta, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Uuendage aadressil [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Seda piirangut rakendab backend, mitte ainult CLI: SDK- või CLI-märgistusega päring, millel puudub tasuline pakett, lükatakse tagasi veakoodiga `403 PLAN_UPGRADE_REQUIRED`, olenemata sellest, kas see pärineb `chloros-cli`-st, Python-i SDK-st või käsitsi loodud HTTP-kliendist. Välja logitud kasutaja saab selle asemel veakoodi `401 AUTH_REQUIRED`. Juurdepääs toimib offline-režiimis paketi armuaegade jooksul (kuupaketi puhul 30 päeva, aastapaketi puhul kuni kehtivusaja lõpuni) ja lõpeb selle möödumisel; `chloros-cli status` jätkab töötamist, et põhjus oleks nähtav (see on ainus SDK-i/CLI-i marsruut, mis on vabastatud taseme piirangust — `GET /api/license-status`).

---

## Viieminutiline kiirjuhend

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (2. samm) käivitab oma tööks tagapõhja, kui ükski ei ole käimas, ja peatab selle, kui töö on lõppenud, seega käivita üks alates 4. sammust. Vaata [Backendi käivitamine](#start-the-backend).

---

## `chloros-cli process`

Töötle pildikausta läbi kogu Chlorosi töötlusketi (sihtmärgi tuvastamine → kalibreerimine → vinjett → peegeldus → indeksi eksport).

### Ülevaade

```
chloros-cli process INPUT [OPTIONS]
```

### Positsioonilised argumendid

| Argument | Kirjeldus |
| --- | --- |
| `INPUT` | Sisendkausta tee, mis sisaldab faile `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) või `.dng`. |

### Üldised valikud

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `-o, --output PATH` | uus kaust nimega `-n` vaikimisi projektikaustas | Projektikaust. Projekt nimetatakse selle järgi ning kaust, milles juba asub `project.json`, saab nimeks `_1` / `_2` nimelise kaustaga, selle asemel et seda üle kirjutada. Vaikimisi projektikaust on see, mille `set-project-folder` (või kasutajaliides) valis, muidu `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | automaatne (ajamärge) | Projekti nimi, kasutatakse ainult **ilma** `-o`-ita (koos `-o`-iga võtab projekt kausta nime). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` kasutab Chloros+ neurovõrgu abil toimivat debayeri Survey3-piltide jaoks; aeglasem, kuid parema kvaliteediga. See on mõeldud ainult Survey3-le: LATTICE'i pildid kasutavad alati standardset demosaic-töötlust, seega kausta puhul, mis sisaldab LATTICE'i pilte, lükkab `process` pärast importimist tagasi `texture-aware`, näidates veateates, kui palju neid on, ja lõpetab töötamise `1` enne mis tahes töötlemist (Chlorosi rakendus ei paku nende jaoks ka Texture Aware'i). Selleks on vaja CUDA GPU-d (NVIDIA GPU Windowsis / Linuxis x64, Orini GPU Jetsonil): ilma selleta kasutab käivitus `standard`-i ning tagapõhi logib ühe `[DEBAYER] … needs a CUDA GPU …`-rea, milles selgitatakse põhjust (näidatud koos `--verbose`-iga). |
| `--vignette / --no-vignette` | `--vignette` | Vignette-korrektsioon. |
| `--reflectance / --no-reflectance` | `--reflectance` | Peegeldusvõime kalibreerimine (kasutab paneelisihtmärki, kui see leitakse, LATTICE puhul NIST-i seerianumbri järgi kalibreerimist). LATTICE multispektraalsete andmete puhul toimib see ka peegeldusvõime **korrutisena** lüliti — vt [Toote-põhised ekspordilülitid](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | välja lülitatud | Rakenda PPK GNSS-korrektsioone sidecar-failidest. |
| `--exposure-pin-1 CAMERA_MODEL` | puudub | PPK-aja sünkroniseerimiseks ekspositsioonipinnile 1 ühendatud kaameramudel. |
| `--exposure-pin-2 CAMERA_MODEL` | puudub | PPK-aja sünkroniseerimiseks ekspositsiooni viigule 2 ühendatud kaameramudel. |
| `--recal-interval SECONDS` | `0` | Minimaalne kalibreerimisintervall sekundites (projekti *Minimaalne kalibreerimisintervall*). |
| `--timezone-offset HOURS` | `0` | Valgusanduri ajavööndi nihke UTC-st, tundides (projekti *Valgusanduri ajavööndi nihke*). |
| `--format FORMAT` | `TIFF (16-bit)` | Üks järgmistest: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros ei suurenda kunagi pildi bittisügavust: kõikidel 8-bitistel piltidel kirjutatakse sügavama formaadiga pilt `TIFF (8-bit)`-na ja seda mainitakse käskluses üks kord. Projektis, kus on segatud 8-bitised ja 12/16-bitised pildid, eksporditakse iga pilt oma bitisügavusega või madalama bitisügavusega. Radiance on alati 32-bitine ujukomaarv. |
| `--indices NAME [NAME ...]` | puudub | Taimestiku indeksid eelseadistuse nime järgi (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), suurtähti ei arvestata. Iga indeks lisatakse projekti vastavalt kaamera lainepikkustele, sealhulgas uus projekt. Nimi, mis ei ole eelseadistus või mida kaamera lainepikkused ei suuda luua (`NDRE` vajab RE ja NIR sagedusalasid, mida ükski kaamera eraldi ei paku), märgitakse hoiatusena ja sellele ei luua indeksipilti. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Sundida LATTICE TIFF-failide töötlemisahela alguspunkti (Survey3.raw-faili see ei mõjuta). Samuti on see pääsetee, mis võimaldab töötleda ka **raw-failita** salvestusi — vt [Kuidas salvestuste kaust välja näeb](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | sisse | Saada lineaarne debayeringu tulemus (`Debayered_Images`). Vaata [Toote-põhised ekspordilülitid](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | sisse | Väljastab ekraanilise eelvaate (`Preview_Images`): RGB = valgebalanss (DAQ-valgusallikas, kui see on saadaval, muidu hall maailm) + gamma; multispektraalne = valevärvide venitus. |
| `--radiance / --no-radiance` | sisse | Edastab float32-tüüpi kiirgustiheduse (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Viide LATTICE peegeldusvõime tootele: `auto` = kvaliteedikontrolli läbinud raamisisene sihtmärk on absoluutne viide (sihtmärk, millel on kvaliteedikontrolli märge, konkureerib mõõdetud tulemuse osas DAQ-ga), DAQ-allapoole suunatud (ρ = π·L/E) tagavaravariant; `target` = range (DAQ-i asendamine puudub); `daq` = DAQ-i autoriteetne. Vaata [Toote-põhised ekspordilülitid](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | puudub | Ühiku kaupa **mõõdetud** sihtmärgi peegeldusvõime skaneeringute kataloog (`<serial>.csv`); kasutab puudumise korral nominaalseid T3/T4P spektreid. |
| `--array-alignment / --no-array-alignment` | sisse | LATTICE-massiivid: rakendatakse iga pildi `Chloros:Alignment*` XMP-failis märgitud moodulitevahelist joondamist igale töödeldud tootele (debayering / eelvaade / kiirgus / peegeldusvõime / indeks). Ilma märgeteta piltide puhul ei toimu midagi. |
| `--array-alignment-crop / --no-array-alignment-crop` | kärpimine | Kärbi joondatud ekspordid massiivi ühisele kattuvale alale, nii et kõik moodulid jagavad ühte pindala; `--no-…` säilitab täieliku anduri pildi (allika väljaspool must täitmine). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Ühtlustamise moonutuse jaoks ümberproovivõtt. `nearest` säilitab täpsed allika digitaalsed numbrid (radiomeetriliste väärtuste pikselitevaheline segunemine puudub). |

### Sihtmärgi tuvastamise valikud

| Lipuk | Kirjeldus |
| --- | --- |
| `--min-target-size PIXELS` | Detektori kalibreerimissihte miinimumsuurus pikslites. |
| `--target-clustering 0-100` | Sihtmärkide klastrite protsent (vaikimisi `60`). |
| `--target / --targets` | Otsi kalibreerimissihteid ainult sisendi alamkaustast `target` või `targets` (kiirem). |

### Näited

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Tootekohased ekspordilülitid (LATTICE multispektraalne)

LATTICE töötlemine jaguneb **ühe läbimisega kõikidesse asjakohastesse toodetesse**. Neli tüübipõhist lülitit — `--debayered`, `--preview`, `--radiance`, `--reflectance` — on kõik**vaikimisi sisse lülitatud**; kasutage vormi `--no-<type>`, et ühte neist välja lülitada. RGB-peakaamerad edastavad alati ainult debayereeritud + eelvaate (mitte ribapõhist kiirgust/peegeldust), seega on `--radiance`/`--reflectance` nende puhul mõjuta. Lülitajaid ignoreeritakse Survey3-i puhul `.raw` (mis järgib standardset peegeldusvõime/sihtmärgi teekonda). *(Vana `--radiometric-output {reflectance,radiance,sensor-response}` lipukest **eemaldati** ja asendati nende lülititega; `sensor-response` taset enam ei ole.)*

| Toode | Väljund | Kas on vaja DAQ allapoole suunatud andmeid? |
| --- | --- | --- |
| `--debayered` | Lineaarne demosaik (`Debayered_Images`): sama fail, mille salvestab kaamera oma `debayered` tasemel — kalibreerimispaketi tume nihe (DSNU) toorandmetest, seejärel OpenCV servatundlik demosaik; puudub lineaarsus-LUT, tasandusvälja või segunemise eemaldamine. | Ei. |
| `--preview` | Eelvaade ekraanil (`Preview_Images`): RGB = valgebalanss + gamma; multispektraalne = valevärvide venitus. | Ei. |
| `--radiance` | float32 W/m²/sr/nm täielikust radiomeetrilisest ahelast (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 peegeldusvõime ρ (`32768` = 1,0), Pix4D-valmis. | **Jah**, välja arvatud juhul, kui seda kinnitab kvaliteedikontrolli läbinud raamisisene sihtmärk (vt allpool). |

`--reflectance-source` valib peegeldusvõime võrdluspunkti:**`auto`**(vaikimisi) muudab kvaliteedikontrolli läbinud kaadris oleva sihtmärgi**absoluutseks võrdluspunktiks**— sihtmärgiga kinnitatud empiirilised joonekettid võrreldakse kõrvale jäetud paneelidel ja mõõdetud võitja rakendatakse ribahaaval: riba, mida DAQ ei suuda kalibreerida või mille DAQ-kalle sobitamine ebaõnnestub, võtab sihtmärgi enda joone (`ReflectanceElBandMethods` näitab, milline) — sihtmärk, mis läbib kvaliteedikontrolli lipuga (üks paneel välja jäetud, üks sagedusala rikkis), konkureerib samuti DAQ-i jagamisega selle mõõdetud tulemuse alusel ning raam, millele ta kaotab, märgistatakse `ReflectanceChain=daq` — kui sihtmärki pole või kvaliteedikontroll ebaõnnestub, kasutatakse DAQ-i allapoole suunatud jagamisreeglit (ρ = π·L/E);**`target`**on range (DAQ-i asendamist ei toimu);**`daq`**loobub DAQ-i autoriteetsest käitumisest. Sihtmärgi geomeetria (ArUco / fikseeritud ROI / riba) pärineb projekti sihtmärgi konfiguratsioonist; `--target-reflectance-dir DIR` säilitab iga üksuse kohta**mõõdetud** skaneeringuid (`<serial>.csv`), mida otsitakse sihtmärgi seadme seerianumbri/QR-koodi järgi, kasutades varuvariandina nominaalseid T3/T4P spektreid.

DAQ-i peegeldusrada määrab **ajamärgiga sobiva allapoole suunatud kiirguse**automaatselt kindlaks salvestatud**`.daq`**(DAQ-U/M/E)**või piltide juures leitud DAQ-M-i omast `.csv`**. Kui kaamera- või DAQ-kalibreerimispaketti pole kohalikult vahemällu salvestatud, laadib töötlemisahel selle esmakordsel kasutamisel**automaatselt AWS-ist alla** (nõuab ühekordset internetiühendust; salvestatakse `~/.chloros/`-i alla).**Kelle näidud need on.**`.daq`, mille salvestus salvestab, on selle kirjeldatud kaamera(te) näidud — nimetatud selle kõrval asuvas `<file>.daq.cameras.json`-sidecaris (`lattice capture`, `multi-capture`, `array-capture` ja ka kasutajaliides; toores pildiseeria nimetab iga liikme näitu oma manifestis) — ning `process` sobitab seda**ainult nende kaamerate piltidega**. Seetõttu ei kasuta iseseisev kaamera, millel puuduvad oma lugemid, enam teise kaamera andmekogumist (DAQ); seostage sellega andmekogumise seade või salvestage ise. Andmekogumise seadme enda salvestus (`daq pool-record` või graafilises kasutajaliideses alustatud salvestus) või mis tahes `.daq` ilma selle lisafailita (sh vanemad salvestused) teenib endiselt kõiki kaameraid.

#### Peegeldusvõime pikslite lugemine (Pix4D / Metashape / teie enda skriptid)

Peegeldusvõime salvestatakse täisarvuna DN, ja **DN, mis tähendab ρ = 1,0, sõltub allikakaamerast**:

| Allikas | ρ = 1,0 on | Kuidas teada saada |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (varu kuni ρ 2,0) | Failile on märgitud XMP-märge `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (piiratud väärtusega ρ 1,0) | Puuduvad `Chloros:*` XMP-märgised — see puudumine *onki* märk. |

**Loe `Chloros:PixelScale` ja jaga sellega**, selle asemel et eeldada konstanti. Märge on defineeritud uint16-domeenis, seega jääb see `32768`-iks ka väljundvormingutes, mis skaalavad ümber — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` ja `TIFF (32-bit, Percent)` on kõik ise kirjeldavad (normaliseerige salvestatud andmetüüp esmalt tagasi uint16-ks: 8-bitisest ×257, float-tüübist ×65535).

> **Vanematest versioonidest pärit failid ei pruugi sisaldada skaalat.** 8-bitine peegeldusvõime TIFF-fail skaalatakse ümber ja märgistatakse `Chloros:PixelScale`-iks nagu teisedki. Versioonides enne 1.3.0 *kärpisid* selle asemel vahemikku 0..255, nii et iga väärtus, mis ületas ρ≈0,008, tasandati väärtuseks 255, ning nad jätsid välja nii `Chloros:PixelScale` kui ka `MicaSense:RadiometricCalibration`-tuple'i. **Kui LATTICE-peegeldusfailil puudub see märge, ärge eeldage skaalat – töötlege fail uuesti**, selle asemel et jagada piksleid, mida ei olnud kunagi võimalik jagada.

#### Indeksipikslite lugemine (NDVI ja muud taimestiku indeksid)

**Täisarvuline indeksraster on pildi-põhine kontrasti venitus, mitte −1…1 kaart.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` ja `JPG (8-bit)` indeksitooted – sealhulgas 8-bitine TIFF, millena 8-bitine pilt salvestatakse – salvestavad

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

kus `IndexMin` / `IndexMax` on **selle kaadri enda madalaim ja kõrgeim piiratud indeksväärtus**. DN 0 ei ole −1 ja täisskaala ei ole +1.

Chloros märgistab mõlemad äärepunktid, et kaardistus jääks pööratavaks:

| Märge | Tähendus |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — allpool toodud konventsioon |
| `Chloros:IndexMin` | indeksväärtus, mis on salvestatud kui DN 0 |
| `Chloros:IndexMax` | indeksväärtus, mis on salvestatud kui täisskaala |

Loe need sisse ja pööra ümber, normaliseerides salvestatud DN esmalt salvestatud andmetüübi maksimumväärtuse järgi vahemikku 0…1, nii et üks avaldis hõlmaks kõiki täisarvuvorminguid:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Lugemiseks ei ole vaja `-config pix4d.config` — exiftool loeb neid XMP-omadusi ise. Konfiguratsioon on vajalik ainult nende *kirjutamiseks*.

> **`IndexMin == IndexMax`** tähendab, et selle kaadri iga piksel sisaldas sama indeksväärtust. Raster on täidetud skaala keskel ja jagada pole millegagi — võta lõpppunkt.

> **`Chloros:Index*`-märgiste puudumine tähendab, et kaardistust ei ole määratud.** LUT-eksport on värvitud renderdus, mille pikslid on gradientvärvid, mis on seotud teie LUT-i lävendi aknaga, ning kaadril, millel puuduvad piiratud indeksiga pikslid, ei ole vahemikku, mida määrata. Ära eelda kaardistust kummagi juhtumi puhul – kehtib sama reegel kui puuduva `PixelScale` puhul.**Kuna venitus on pildipõhine, ei ole ühe lennu kahte kaadrit võimalik võrrelda DN-na**— igal kaadril on oma lõpppunktid, mistõttu sama NDVI-väärtus langeb erinevates piltides erinevatele DN-väärtustele. Kogu lennu hõlmava kvantitatiivse töö jaoks (tsoonistatistika, fikseeritud künnised, aegridad) ekspordige**`TIFF (32-bit, Percent)`**: indeksrastrid salvestatakse sinna float32-tüüpi**indeksväärtustena**, ilma ümberskaalimiseta ja märgistusteta. Vaatamata formaadi nimele ei ole indeksitooted ×100 – see skaalimine kehtib ainult täisarvutüüpi peegeldusvõime puhul.

#### Segunemise piiramismask (millistel pikslitel riba piiratud oli)

3×3 spektraalne segunemine võib viia riba negatiivseks – kui valgus sisaldab seda riba vaevu või kui stseen jääb väljapoole seda, mida kaamera segunemismatriits kirjeldab. See ei ole tegelik väärtus, seega fikseerib eksport selle väärtuseks 0 – ja reaalajas vaade näitab samuti 0. Ribasid võib piirata mõne pikseli ulatuses või peaaegu kõigi pikslite ulatuses; mask salvestab mõlemad variandid. `Chloros:Band{R,G,B}UnmixClampFraction` näitab, kui suure osa kaadrist see hõlmas; piiramismask näitab **millised pikslid**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— `clamp_masks/`-kaustas**toote kausta kõrval**, mitte kunagi selle sees (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; `process` ekspordi puhul `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — iga kiirguse, peegeldusvõime ja indeksiga toote puhul, milles vähemalt üks piksel on piiratud: nii salvestused kui ka ekspordid (ning ka kaadrihaaval TIFF-failid). Ühe lainepikkusega, toote enda pikselvõrgustikus (sh orientatsioon, uurimisala, binnimine ja massiivi joondamine). Iga mask nimetab oma toote kaamera (`Make` / `Model` / seerianumber), salvestusaja ja objektiivi märksõnad ning loetleb lainealad, mida selle bittid tähistavad (`XMP-Camera:BandName`, bit 0 esimesena); selle `ImageDescription` ja `Software` märgivad seda klambrimaskina ning see ei sisalda kalibreerimist.
- **Bitid**: bit 0 = toote R-kanali sagedusala (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Seadistatud bitt tähendab, et selle sagedusriba pikseli väärtus on piiratud 0-ni — käsitle seda ja sellest koostatud indeksit madala usaldusväärsusega andmetena.
- **`Chloros:UnmixClampMask`** on märgitud igale tootele, mille puhul on läbi viidud segunemise eemaldamine: maski asukoht toote enda kausta suhtes (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, alati `/`-ga eraldatud – ühendage see pildi kaustaga, et maski avada) või `none`, kui ühtegi pikslit ei ole piiratud. Märgistuse puudumine tähendab, et tootel ei ole kunagi unmix-protsessi läbi viidud (RGB- või monokaamerad, toorandmed/debayered-tasemed).
- `lattice capture` / `array-capture` trükivad maski asukoha salvestatud faili alla (`clamp mask: …`) ning salvestustulemuste nimekirjas on see loetletud kaadri `clamp_mask`-na.
- `lattice index --live --save-multiband` kirjutab mitme sagedusriba pildistamise puhul ühe maski iga allikakaamera kohta — vt [`lattice index` — mitme sagedusriba pildistamise maskid](#lattice-index--multiband-stack-masks). Videol maski ei ole: `array-build-video` näitab selle asemel iga kaadri klambri fraktsioone (vt [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Kuna üheski tootekausta ei ole maski, võib mis tahes tasandil tootekausta (`radiance/`, `reflectance/`, `process` ekspordi kaust `*_Images/`) edastada Pix4D-le, Metashape’ile või muule fotogramm-meetria tarkvarale just sellisena, nagu see on — ka alamkaustad kaasa arvatud — ning see võtab vastu ainult pilte. Edastage sellele tootekataloog, mitte selle kohal asuv kataloog (selles on ka `clamp_masks/` ja kõik muud eksporditüübid). Chlorose enda importimisfunktsioon jätab maskid vahele. Maski lugemine: `(mask & 1) != 0` tähendab „R-kanali sagedusala piiratud siin”, `& 2` tähendab G, `& 4` tähendab B.

#### EXIF-andmed kantakse ekspordisse üle

`process` kopeerib allika pildi **GPS-ploki ja selle ExifIFD** igale tootele, seega
eksporditakse koos georeferentsiga ka `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` ja
`CameraSerialNumber` koos georeferentsiga.

**`FocalLength` ei ole fotogrammmeetrias valikuline.** Pix4D arvutab maapinna proovivahe kauguse
fokaalkauguse ja kõrguse põhjal; kui märge puudub, kasutab programm tagavaraks äärmiselt vale mõõtkava. Ühel
49 kaadri pikkusel apelsinipuude istanduse lennul muutis puuduv märge 411 m × 160 m suuruse ala rekonstrueeritud
47,8 km × 13 km suuruse alaks – 455 MP ortofoto, mis koosnes peamiselt andmeteta aladest, mida peeti esialgu mosaiigiprobleemiks ja
BigTIFF-probleemiks, enne kui keegi GSD-d kontrollis. Kui teie ortofoto tulemus on ebausutavas
mõõtkavas, käivita eksporditud toote peal esmalt `exiftool -FocalLength`.

Koopia ei ole tahtlikult **`-all:all`**: IFD0'struktuurimärgised rikuvad LATTICE väljundit, kui
neid kopeeritakse, ning `ExifImageWidth` / `ExifImageHeight` on välja jäetud, kuna need kirjeldavad
*allika* pildistamist – eksport, mille suurust on kunagi muudetud, kannaks muidu mõõtmeid,, mis on vastuolus selle enda rasteriga. XMP kirjutatakse otse, mitte kopeeritakse, sest ExifTool
kõrvaldab sama käivituse XMP-märgised, kui XMP-plokk kopeeritakse (mis jätaks välja MAPIRi
kalibreerimismärgised).

### Kuhu väljundid salvestatakse

Tulemused salvestatakse **projekti kausta alla, rühmitatuna kaamera ja seejärel failivormingu järgi**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE kaamera kaust on `LATT-<sensor>-<lens>-F<filter>` (vastab pildi EXIF-i
`Model`) ja `<model>_<filter>` (Survey3) – kaks kaamerat, millel on ühine sensor ja filter, kuid erinevad
objektiivid, moodustavad eraldi puud, kuna vinjetatsioon, vaateväli ja moonutus erinevad. Järgmine formaat
kausta struktuur on järgmine: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` või `tiff32`, kui tegemist on
`TIFF (32-bit, Percent)`-ga.

> **Iga eksporditud toode säilitab ALGFAILI nime.** Kiirguse eksport
> `capture_…_raw.tif` kannab endiselt nime `capture_…_raw.tif` — see asub lihtsalt kataloogis
> `tiff32/Radiance_Images/`. **Toote identifitseerib kataloog, mitte failinimi**, seega `*radiance*.tif`
> järgi otsides ei leita midagi; otsige pigem kataloogi järgi.

### Valgusanduri salvestused — kalibreeritud `.daq` + `.csv`

`process` töötleb ka sisendkaustas olevaid `.daq` salvestusi ning selleks **ei**
selleks mingeid pilte: eraldi lennutatud DAQ-U / DAQ-M / DAQ-E on täielik
salvestus ning kaust, mis sisaldab ainult `.daq` faile, on kehtiv sisend.

DAQ-seadet saab salvestada **ilma** selle kalibreerimiseta – just seda teevad avalikud
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) salvestid
(`record_daq.py`) vaikimisi teevad: nad kirjutavad salvestusse andurite toorandmed ja märgistavad faili nii, et
Chloros laadib selle anduri tehase kalibreerimise **seerianumbri järgi** (esmalt kohalikust vahemälust,
seejärel MAPIRi pilvest) ja rakendab seda. `process` kirjutab tulemuse tagasi:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` sisaldab ühte rida iga mõõtmistulemuse kohta: UTC-ajamärge, integratsiooniaeg, koguvõimsus,
fotopiline/skotopiline luks, PPFD (ja selle sinine/roheline/punane jaotus), tipp-lainepikkus, seejärel
täisspektri andmed anduri enda lainepikkuste võrgustiku järgi. `.daq` imporditakse uuesti ilma
teistkordse kalibreerimiseta.

`<name>` on salvestuse enda failinimi. Kaks erinevates kaustades asuvat salvestust võivad jagada
ühte – salvestuskohta näitajaid (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), mis on salvestatud
sama millisekundi jooksul kahe kaamera poolt – ning igaüks saab ikkagi oma tooted: esimene säilitab
`<name>`, järgmine lisab nime **järgi** kaustad, mis seda eristavad
(`<name>__<capture>_daq_calibrated.daq`), nii et salvestuse kuupäev ja kellaaeg on endiselt esikohal.

Eduka käivituse korral näitab käivitusaruanne `Light-sensor products written: N (calibrated .daq + .csv)`.
Sulgudes kirjeldatakse, mida tegelikult kirjutati, seega on tulemuseks
`(RAW COUNTS — this sensor has no calibration bundle)`, kui andur on ilma kaustata, ja
`(N calibrated, M raw counts)`, kui kaust sisaldab mõlemat. Tagapõhja enda
pealkirjad `[DAQ-EXPORT]` ja `[RUN-SUMMARY]` on sõnastatud samamoodi – ühtegi
neist kolmest ei saa nimetada kalibreeritud toorekspordiks.

DAQ-U / DAQ-M / DAQ-E salvestus, mille kalibreerimispaketti ei ole võimalik hankida – olete
offline-režiimis või selle anduri kalibreerimisandmed puuduvad failist – **jätatakse vahele koos põhjendusega** real
`[DAQ-EXPORT]`, seda ei salvestata kunagi „kalibreeritud” failina, mis sisaldab töötlemata loendusi.
Ühenduge internetiga ja käivitage uuesti. Põhjus on see, mille lugeja tegelikult
selle faili jaoks kindlaks tegi (loetamatu skeem, puuduv pakett, kirjutusviga), ning käivituse
kokkuvõttes loetletakse **erinevad** põhjused – kakskümmend ühe põhjuse tõttu vahele jäetud faili loetakse ühe
põhjusena, mitte selle kahekümne kordusena. CLI nimetab iga vahele jäetud salvestuse käivituse lõpus
koos selle põhjusega:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Salvestuste kaust, millest **ühtegi** ei ekspordita, ebaõnnestub: vt
[Käivitus, mis ei kirjuta ühtegi pilti, ebaõnnestub](#a-run-that-writes-no-images-fails).

Salvestus, mille nimi lõpeb `_calibrated` või `_raw` (`field_raw.daq`), **ei****: need on nimed, mille `process` annab oma väljunditele, ja väljundit, mida loetakse tagasi
salvestisena, loetaks kahekordselt. CLI näitab seda ja nimetab faili —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— selle asemel, et teatada tühjast kaustast. Nimetage selline salvestis ümber enne selle töötlemist.

#### DAQ-A salvestised eksporditakse toorloenditena

**DAQ-A** perekond on vanem kui seerianumbri-põhine pakettide süsteem ja sellel puudub kalibreerimispakett,
mida alla laadida – see kalibreeritakse hoopis kohapeal peegeldusmäära sihtmärgi abil, mistõttu
pole seda kunagi vaja olnud. Nende salvestuste tagasilükkamine jättis neile võimaluse
oma andmeid üldse välja saada, seega eksporditakse need **teise nime** all:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Teine failinimi, mitte lipukese faili sees, sest väide peab säilima
ka siis, kui seda edastatakse e-kirjaga pelgalt nimena. `.csv` pealkiri näitab
`raw spectral sensor counts (NOT irradiance)` ja hoiatab, et väärtused on võrreldavad
**faili sees** — mis ongi just see, milleks sihtmärgipõhine kalibreerimine neid kasutab — ja
mitte erinevate andurite vahel. Võimsusest sõltuvad fotomeetrilised veerud (koguvõimsus, fotopiline ja
skotopiline luks, PPFD) kirjutatakse **NULL**-ina, mitte loenduste põhjal integreerituna, ning käivituse
kokkuvõttes on kirjas `RAW COUNTS`, seega ei saa logis „eksportitud“ ei saa lugeda kiirgustihedusena.

Vana versiooni **v1.01 / v1.02** salvestused (neid kirjutab DAQ-A-SD) ei sisalda iga lugemise ajamärget,
vaid ainult faili kirjutamise aja. Pildi↔allapoole suunatud kiirguse sobitaja ikka veel neid ei aktsepteeri –
kaadri sobitamine kirjutamisajaga oleks märkamatult vale –, kuid eksportija loeb neid ning
CSV-failis kuvatakse `clock=daq_created_on`, seega näitab toode, millisel kellal see põhineb.

### Märkused

- `process` tuvastab automaatselt, kas teie kaust on Survey3, „LATTICE” või segatud.
- Töö edenemine edastatakse Server-Sent Eventsi kaudu; CLI näitab reaalajas iga protsessiniidi edenemist (tuvastamine, analüüsimine, töötlemine, eksportimine).
- Linuxi/Jetsoni puhul kontrollib CLI vahemälumahtu ja võib enne suurte kaustade töötlemist hoiatada. Tekstuuritundlik debayer rakendab madala energiatarbimisega Jetsonitel (Nano, Orin Nano) automaatselt ka GPU sageduse piirangut.
- Eduka töötlemise korral näitab käivitusaruanne, kui palju pilditooteid salvestati (`Image products written: N`; kui segamise eemaldamine piiras mõnda pikslit, siis `(plus M clamp masks)`).
- **`process` jagab töötavat tagapõhja ohutult.** See taaskäivitab juba töötava tagapõhja automaatselt ainult siis, kui see tagapõhi on eelmise projekti laadimise järel ooterežiimis ega hoia endas ühtegi kaamerat, massiivi ega andmekogumisseadet. Backend, mis hoiab mõnda neist (GUI, mis edastab massiivi, SDK-skripti, DAQ-salvestust), kasutatakse sellisena, nagu see on, ja seda ei taaskäivitatud kunagi, seega saab `process` töötada reaalajas töötava massiivi kõrval. Kui backend teatab käimasolevast tööst, ootab `process` ootab kuni 15 sekundit, kuni see lõpeb (äsja lõppenud käivituse tühistamine võtab umbes sekundi), seejärel annab veateate `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Globaalne lipuk `--restart` sunnib ikkagi kasutama uut backendit ja lõpetab kõik selle peal töötavad kaamera-, massiivi- või DAQ-seansid. Backendit, mida käitab Linuxi systemd-teenus, ei taaskäivitatud kunagi: `process` kasutab seda ning `--restart` väljastab selle asemel `sudo systemctl restart <unit>` (vt [Backendi käitamine teenusena](#running-the-backend-as-a-service-headless)). Samuti ei ole seda Chloros-rakenduse enda backend: kui rakendus on avatud, kasutab seda `process`, laadides oma projekti rakenduse projekti asemel (selle asemel käivitatud backend lukustaks avatud rakenduse sellest välja).
- **Käimasolev töötlus säilitab oma projekti.** Kui töötlemisprotsess on käimas — käivitatud rakenduse, `process`, SDK-skripti või `chloros-mcp` poolt — keeldub backend faile looma, avamast, importimast, neisse faile lisamast või neist eemaldamast, faile teisaldamast, kustutada ega muuta projekti töötlemisseadeid, andes HTTP-veakoodi 409 `PROCESSING_IN_PROGRESS` ja `A processing run is in progress on project '<name>', so ... now is refused`; `process`, mis selle kätte saab, väljub `1`. Kuvaseaded (pisipildi suurus) salvestatakse endiselt. Töötlemisrežiimi ja projekti sihtpeegeldusastme seadeid ei lubata samal viisil muuta (töötlus säilitab režiimi, milles see algas). Käivituse käigus projekti salvestatud pilt (rakendusest, CLI-st, SDK-st või `chloros-mcp`-st) salvestatakse tavapäraselt, kuid lisatakse projekti faililoendisse alles käivituse lõppedes; seda kinnitab ka vastus (`project_add`), samuti projekti peatatud valgusanduri salvestus. Varem võis teine klient käivituse käigus avatud projekti vahetada ja käimasolev käivitus eksporditi teise projekti.
- **Ctrl+C peatab käivituse tagapõhiselt, mitte ainult CLI-s.** CLI-liides palub tagapõhjal töötlemine peatada, kuvab `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` ja ootab kuni 120 sekundit `Processing stopped.`-i, et `chloros-cli process … && next` ei käivitaks `next` käivitamist tagapõhis, mis on veel käivitamisega hõivatud. Teine Ctrl+C lõpetab ootamise. Seejärel lõpetab `process` käivituse `130`, nagu iga katkestatud käsk.

#### Käivitus, mis ei kirjuta pilte, ebaõnnestub

Kui te küsisite tooteid ja käivitus kirjutas **ühtegi** — ainult `project.json` ja
`calibration_data.json` — käsitleb `process` seda ebaõnnestumisena: see väljastab
`Processing finished but wrote no image products.` ja **lõpetab töötamise nullist erineva tulemusega**, nii et skript saab
seda tuvastada. Teates on nimetatud projekti kaust ja tavapärased põhjused:

- sisendkaustu ei tunnistatud salvestusena (kontrollige paigutust ja `--input-level`) või
- kõik taotletud tooted jäeti vahele, kuna need ei olnud nende kaamerate puhul rakendatavad (nt
  RGB-kaameratelt kiirgust/peegeldust).

Käivitage uuesti koos `--verbose`-ga ja kontrollige tagapõhja logist ridu `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
mis selgitavad kaamerapõhiseid vahelejätmisi, mis muidu CLI väljundisse ei jõua.

Tahtlikult ainult metandmetega käivitus — kõik tooted välja lülitatud ja ilma `--indices`-ita — on ikkagi
**edu**, sest tühi pildiväljund on selles juhul õige tulemus.

#### Käivitus, mille eksporditud faile ei olnud võimalik kontrollida, ebaõnnestub

Käivituse järel kontrollib `process` eksporditud failide sildid ExifTooliga. Kui ExifTooli ei olnud võimalik käivitada,
kuvatakse `Ekspordi kontroll EBAÕNNESTUS: ExifTooli ei olnud võimalik käivitada, seega M eksporditud failist N sildid
ei kontrollitud (vaata tagapõhja logis ridu [EXIFTOOL])` and `Tulemused on salvestatud ja jäävad
paigale; käivita protsess uuesti, et neid kontrollida.`, does not print `Töötlemine lõpetatud!` ning **lõpetab töötamise nullist erineva väärtusega**.
Tulemused säilitatakse.

#### Millised pildid käivituse käigus salvestatakse

Enne `process` käivitamist lõpetatud pildistamised on kaasatud. Käivituse ajal salvestatud pildid ei ole kaasatud; `process`
hoiatab, kui palju jäi välja, ja uuesti käivitamisel lisatakse need.

#### Jooks, mille käigus kaotati pilt, ebaõnnestub

Kui pilt ebaõnnestus käivituse ajal, väljastab `process` `N image(s) failed and were not exported` koos
kõige tavalisema põhjusega, ei väljasta `Processing complete!`-i ning **lõpetab töötamise nullist erineva tulemusega**. Muud tulemused
kirjutatakse välja ja jäävad alles ning tagapõhja logifailis on kirjas iga ebaõnnestumine.

Sama kehtib ka **ainult valgussensoriga käivituse** puhul: `.daq` salvestuste kaustas pole määratluse järgi eksportimiseks pilte
ning käivitust hinnatakse selle asemel kalibreeritud `.daq` / `.csv` põhjal, mille see kirjutas.
Kui see ei kirjutanud **midagi** — kõik salvestused jäeti vahele —, kuvab see
`Processing finished but wrote no light-sensor products.`, nimetab iga salvestuse selle
vahele jätmise põhjuse järgi ja **lõpetab nullist erineva tulemusega**.

---

## `chloros-cli login`

Autentige see seade Chloros+ pilvekontoga. Autentimisandmed salvestatakse turvaliselt vahemällu `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Näited

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (parooli osade eemaldamine või dubleerimine). Veakoodi 401 korral proovib CLI automaatselt uuesti, lisades `$$`-lisatud, seejärel parooli dubleerimata poolega; kui uuesti proovimine õnnestub, logib süsteem teid sisse ja kuvab õige ühekõnekeele süntaksi, mida järgmisel korral kasutada.

> **Peata/skriptipõhine kasutamine: puuduv vahemällu salvestatud seanss tähendab sisselogimiskutset.** Core-käsk (`process`, `status`, `export-status`, `time-sync`, …) käivitamine ilma vahemällu salvestatud sessioonita nõuab enne jätkamist stdin-ist sisendit `Email:` / `Password:`. Kui stdin on suletud (cron, systemd-üksus), ebaõnnestub käivitamine kohe veaga `login cancelled`; kui stdin on avatud terminal või toru, ootab programm sisendit. Käivita `chloros-cli login EMAIL PASSWORD` üks kord iga masina kohta enne peata tööde planeerimist.

---

## `chloros-cli logout`

Tühistab vahemällu salvestatud seansi ja sunnib järgmisel kutsumisel uuesti sisse logima.

Välja logimine töötlemise käigus ei peata seda: töötlemine lõpeb selles režiimis, milles see algas, ja järgmine töötlemine toimub standardrežiimis. `logout` kinnitab seda (`The processing run in progress finishes in <mode> mode; the next run uses standard`), samuti ka rakendus, kui te sealt välja logite.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Näitab praegust litsentsitaset (Iron/Copper/Bronze/Silver/Gold), autentimitud kasutajat ja seadmega seotud kontode arvu.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Küsib reaalajas Thread-4 ekspordi edenemist. Seda on ohutu kutsuda **`process` käivitamise ajal** teisest shellist.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

CLI kuvakeele seadistamine (toetatakse 38 keelt, sealhulgas CJK, RTL ja india keeled). Vanematel konsoolidel, mis skripti kuvada ei suuda, lülitub süsteem sujuvalt inglise keelele.

```
chloros-cli language [LANG_CODE] [--list]
```

### Näited

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Projektikausta käsud

Need haldavad vaikimisi projektikausta asukohta (jagatud GUI-ga).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` kuvab kausta, kuhu uued projektid salvestatakse. Kui konfigureeritud kausta enam ei ole (see on kustutatud, ümber nimetatud või asub ühendamata kettal), siis kasutab CLI, nagu ka graafiline kasutajaliides, vaikimisi kausta: vaikimisi kausta nimi kuvatakse stdout-i, puuduva kausta nime sisaldav hoiatus saadetakse stderr-i ja väljumiskoodiks on `0`.

---

## `chloros-cli update`

Ainult Linuxis / Jetsonil. Kontrollib `version_url` versiooni `/etc/chloros/update.conf` suhtes ja kui uuem versioon on olemas, laadib alla sobiva versiooni `.deb` ja installib selle kohe koos `sudo dpkg -i`-ga (kinnitust ei küsita; `sudo` võib küsida teie parooli). Jetsonil pakutakse värskendust ainult JetPack 6 süsteemile. „Windowsis“ ei tee käsk midagi ja teatab `up to date`: värskenda uue paigaldajaga.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Linuxis/Jetsonil teostab CLI **iga käivitumise ajal automaatse uuenduste kontrolli** (blokeerimata, ei viivita kunagi käsku): see loeb `/etc/chloros/update.conf`, salvestab tulemuse 1 tunniks vahemällu `~/.chloros/update_cache.json` ja kuvab `Update available: vX.Y.Z / Run: chloros-cli update`, kui uuem versioon on olemas. Vead ja Windowsi keskkondades vahetatakse see vaikimisi üle; `CHLOROS_DISABLE_UPDATE_CHECK=1` lülitab selle välja. 1-tunnise vahemälude tõttu võib tunni jooksul avaldatud versioon ikkagi kuvada end ajakohasena.

---

## `chloros-cli selftest`

Käivitab 7-astmelise kiirtesti: versioon, pordi kättesaadavus, tagapõhja käivitamine, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), müra vähendaja mudeli olemasolu, CUDA+müra vähendaja valmisolek. See kasutab töötavat tagapõhja või käivitab selle testimiseks ja peatab selle pärast. See väljastab `N/7 checks passed` ja ebaõnnestunud kontrollid ning lõpetab `0` töötamise alles siis, kui kõik seitse on edukad; CUDA+müra vähendaja kontroll ebaõnnestub arvutis, millel puudub CUDA GPU (vt [Paigalduse kontrollimine](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP grandmasteri staatus ja juhtimine. Chloros-host käitab PTP grandmasteri; LATTICE-kaamerad ja DAQ-E-seadmed töötavad selle alluvuses seadmetevaheliste ajamärkide saamiseks (seadmetevahelise viivitusega umbes 1 ms). PTP viib **ajamärgised** ühtlust: kaamerate massiiv teeb üheaegseid ülesvõtteid, kuna M8 riistvaratrigger käivitab need (moodulite vahel vähem kui 100 µs), mitte PTP tõttu. `chloros-cli time-sync` ilma alamkäskudeta käivitab `status`.

| Alamkäsk | Kirjeldus |
| --- | --- |
| `status` | Näitab grandmasteri seisundit, BMCA prioriteete, kella identiteeti ja seda, kas selle paketid jõuavad võrku: `health` on `OK` (`STOPPED`, kui grandmaster ei tööta) või `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` koos `health_detail`-iga, ja seejärel trükitakse stderr-i rida `WARNING:` (stdout jääb JSON-vormingusse). `sync_sent` / `announce_sent` loendavad ainult neid sõnumeid, mis läksid läbi; `*_send_errors` ja `last_send_error` loendavad neid, mis ei läinud läbi. Pärast kaameraühenduse katkemist ja taastumist ehitab grandmaster oma pistikupesad ise uuesti üles (`rebinds`, `last_rebind`). |
| `peers` | Loetleb „Delay_Req“ kaudu tuvastatud orjad (kaamerad + DAQ-E andurid). |
| `cameras` | PTP-töökorrasolek kaamera kaupa (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Grandmasteri protsessi taaskäivitamine. |
| `set-priority --priority1 N --priority2 N` | BMCA prioriteetide ümberkirjutamine. |

Grandmaster teeb teadaande kaamera võrgukaartide kaudu (mis tahes adapter, mille aadress sisaldab `169.254.x.x`). Kui ühtegi pole saadaval, kasutab ta tagavarana vaikimisi marsruudi adapterit ainult siis, kui see on ühendatud kaabliga, mitte kunagi Wi-Fi-d ega virtuaalset adapterit (Tailscale, WSL), kuna ükski kaamera ega DAQ-E ei asu ühegi taga. Kui pole midagi teada anda, töötab ta ilma pistikupesadeta (`health` `NO_SOCKETS`) ning järgmise kaamera või DAQ-seadme ühendamisel ehitatakse see uuesti üles selleks ajaks töötavale võrgukaardile; `time-sync restart` teeb seda kohe.

Grandmaster töötab omaette protsessis, backendist eraldi (`own_process` / `process_pid` failis `time-sync status`), kõrge prioriteediga Windowsis (`process_priority`), nii et hõivatud backend (töötlemistsükkel, eksport, mitu käsku korraga) ei saa viivitada selle poolt märgitud aegu; varem võis üks selline viivitus nihutada korraga kõigi kaamerate kellaaega. See protsess lõpeb backendiga; kui seda ei saa käivitada, töötab grandmaster endiselt backendi sees (`own_process` false, ja backend-logis on see märgitud).

### Näited

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE kaamerajuhtimine. Iga alamkäsk suunatakse läbi Chloros-tagapõhja; tagapõhi haldab kaamerapooli, mistõttu järgnevad CLI-kutsed kasutavad uuesti sama avatud käepidet.

### Üldised valikud (enamikul alamkäskudel ühised)

| Lipuk | Kirjeldus |
| --- | --- |
| `-d, --device N` | Kaamera indeks (vaikimisi: 0). |
| `-s, --serial SN` | Konkreetne seerianumber; asendab `--device`. |
| `--serials SN1,SN2,…` | Komadega eraldatud seerianumbrid mitme kaamera kasutamiseks. [Ajutised mitme kaamera käsud](#multi-camera-transient-sessions) avavad ainult need kaamerad, millest esimene käivitab päästiku. |
| `--all` | Töötab kõigi avastatud kaameratega. |
| `--exposure US` | Säriaeg mikrosekundites. |
| `--gain DB` | Võimendus dB-des. |
| `--pixel-format FMT` | nt `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` M3M-il. Nimi määrab selle bittisügavuse kaamera enda perekonnas: M3M-il määrab Bayeri nimi selle sügavuse Mono-formaadi (`BayerRG8` määrab `Mono8`), M3C-l määrab Mono-nimi Bayeri nime (`Mono12` määrab `BayerRG12`). |
| `--width N` / `--height N` | Pildi mõõtmed. |
| `--preset {default,high_quality,high_speed,triggered}` | Eelseadistuse rakendamine: selle pikseliformaat, säriaeg, võimendus ja kaadrisagedus (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 kaadrit sekundis; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` loetleb need). M3M-vormingus on tegemist sama sügavusega mono-vorminguga: `high_speed` salvestab `Mono8`, `high_quality` ja `triggered` `Mono12`. Sellega kaasnev lipuk (`--exposure`, `--gain`, `--pixel-format`) on eelseadistusest ülimuslik, ning seadistus, mida eelseadistus ei nimeta, säilitab kaamera enda seadistuse. Kõik vabadtöötavad vabalt, välja arvatud `triggered`, mis aktiveerib kaamera riistvaralise serva jaoks liinil 2 — kui seda liini midagi ei juhita, ootab kaamera igavesti, selle asemel et salvestada. |
| `-o, --output DIR` | Väljundkataloog (vaikimisi: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-paketi suurus. `auto` käivitab ICMP+GVSP-proovid; `jumbo` = 9000; `standard` = 1500. |

### LATTICE kaamera esmakordse ühendamise töövoog

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Alakäskude viide

#### Avastamine ja teave

| Alakäsk | Otstarve |
| --- | --- |
| `lattice info` | Ühendatud kaamerate loetelu (tootja, mudel, seerianumber, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analüüsib host-süsteemi kaamera optimaalse konfiguratsiooni leidmiseks. `--no-discover` jätab kaamera avastamise vahele (kiirem, ainult võrgukaardi analüüs). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Kontrolli/paranda võrgukaardi seadeid (Linuxis ka tuuma pistikupesa vastuvõtupuhvreid — vt [Linuxi hostid](#linux-hosts--socket-receive-buffers)); hinda ribalaiust/kaadrisagedust. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Stable-schema tagapõhja võrgu-võime + massiivi soovitus (tagastab `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` on vaikimisi seadistatud täielikule andurile, `2048` × `1536`. See lõpetab `0`, et jätkata `ok`, `auto_capped_fps` ja `auto_shrunk` ning `1` asendab `needs_force_slip` ja `error`, kas koos või ilma `--json`-iga. See jõuab tagapõhja aadressil `--backend-url` (vaikimisi `http://127.0.0.1:5000`), mitte `CHLOROS_BACKEND_URL`. `auto_capped_fps` säilitab taotletud resolutsiooni, kuid piirab sihtkaadrisagedust — loe `recommended.recommended_target_fps` ja edasta see ühenduse sihtmärgina; käsitle seda eduna, mitte veana. `auto_shrunk` ilmub ainult siis, kui taotletud kaadrit ei ole võimalik edastada ühegi kaadrisagedusega; connect ei vähenda kunagi kaadreid omal algatusel. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | „Mis-kui” analüüs kaameraid avamata (`--width` / `--height` vaikimisi `2048` × `1536`). Nii see kui ka `network-analysis` prognoosivad, mida connect edastab: ilma `--pixel-format`-ita kehtivad Connecti kaamerapõhised formaadid (Mono12 / BayerRG12), ning koos `--packed auto`-iga (vaikimisi) [pakitud 12-bitine](#packed-12-bit-transport---packed) otsus, mille Connect teeks sellel arvutil — seega on nimetatud kaadrisagedus (fps) see kiirus, millega massiiv töötab. Mõlemad väljastavad pakkimisotsuse. **`--n-active` on võrgus olevate kaamerate koguarv, mitte ainult selle massiivi oma**— suurendage seda, kui eraldiseisvad kaamerad edastavad andmeid samaaegselt või kui võrgu läbilaskevõime arvutatakse nõudluse põhjal, mis neid alahindab (vaikimisi: `len(--models)`). Kuvab alati koondandmed `Wire budget:` (nõutav MB/s vs kokkupõrkeohutu ülempiir) ja `Max cameras:` ning märgistab `** OVER-SUBSCRIBED**`, kui massiiv ületab võrgu läbilaskevõime — vt [Massiivi kaadrisagedus ja pursemudel](#array-fps--burst-model). |
| `lattice gpu` | Näita GPU staatust. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Kontrolli või uuenda kaamera püsivara. Kohalik `.fwa` valik on fikseeritud: `firmware/<MODEL_PREFIX>/`-is olev fail, mis vastab koostusversiooni `MIN_FIRMWARE_VERSION`-ile, salvestatakse, kui see on olemas (kõrgeim versioon ainult varuvariandina), seega jääb kettale salvestatud uuem tootja pilt passiivseks, kuni seda kinnitust muudetakse — tahtlikult uuemad versioonid jõuavad seadmetesse allkirjastatud AWS-manifesti kaudu, mida eelistatakse, kui see on uuem. |
| `lattice presets [--apply NAME]` | Kaamera eelseadistuste loetlemine või rakendamine. `--apply` määrab kindlaks, mida eelseadistuse nimed tähistavad (pikseliformaat, säri, võimendus, kaadrisagedus, päästik), ning kuvab need seaded sellisena, nagu kaamera neid pärast järgmist loeb: M3M-il eelseadistuse sügavuse Mono-formaat; kaadri kiirus, millega see töötab, mis võib olla madalam kui eelseadistus nõuab (`high_speed` nõuab 60 kaadrit sekundis; täiskaadriline kaamera 1 Gbps ühendusel töötab umbes 35 kaadrit sekundis). Kõik muud seadistused, mida selle avatud fail kirjutas, taastatakse leitud kujul. `--apply default` taastab kaamera tehasevaikimisi seaded: tehase seaded vabalt töötava automaatse säritusega Line2-l, ühenduse läbilaskevõime piirang ja defektide korrigeerimine välja lülitatud, RGB-kaamera kalibreeritud valgebalanss säilitatud (muudel kaameratel ühtsus). Kaamera mällu ei salvestata midagi, seega järgmisel sisselülitamisel alustab kaamera oma mälust nagu varemgi. Kuna tegemist on salvestamise või ühenduse käskudega `--preset` ja `default`, ei ole siin eelseadistusi. |
| `lattice status` | Kaamera reaalajas seisund. |

#### Salvestamine

| Alakäsk | Otstarve |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Üks kaader. **Salvestab vaikimisi kõik eksporditüübid**(`--processing all`); vt [Salvestamise eksporditasemed](#capture-export-levels-the-all-default). `--processing` võtab vastu `all`, `raw`, `debayered`, `radiance`, `reflectance` või `display` (`preview` aktsepteeritakse `display` puhul aktsepteeritakse); mis tahes muu nimi lükatakse tagasi enne kaamera puudutamist (väljumine `1`). `--levels` salvestab selgesõnalise alamhulga (asendab `--processing`). Pildistamiseks avatakse kaamera, mida Chlorosil pole, ja selle säriaeg määratakse esmalt stseeni järgi (kõige rohkem paar sekundit), nagu seda teevad [ajutised käsud](#multi-camera-transient-sessions); kui salvestamine lõpeb, olenemata sellest, kuidas see lõpeb, taastatakse see kaamera selliseks, nagu see leiti (formaat, säriaeg, värvipipeline, kõik salvestamise käigus muudetud seaded — nimekiri jaotises [Mitme kaameraga (ajutised seansid)](#multi-camera-transient-sessions)). Kasutatud salvestust lugenud DAQ salvestatakse `.daq`-sidecarina, mis sisaldab ainult toorandmeid; `--no-daq` / `--daq-tolerance-ms` kohandavad seda ([Salvestust lugenud DAQ salvestab](#the-daq-reading-a-capture-saves)). `--force-daq` on endiselt lubatud. `--jpeg-quality` = JPEG-kvaliteet 1–100 (vaikimisi 95).**Avatud projekti salvestatud failid lisatakse sellesse.** Nagu rakenduseski, lisab Chlorosis avatud projekti kausta sattunud salvestuse failid selle projekti faililoendisse; mujale salvestatud faile ei muudeta. Käsk kuvab ühe rea, milles seda märgitakse (`Project: ...`) või, kui töötlemisprotsess on käimas, et need hoitakse tagasi ja lisatakse projekti, kui protsess lõpeb. Sama kehtib ka käskude `multi-capture`, `array-capture` (samuti `--smart`) ja `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Salvestab voogu kettale kuni Ctrl+C-ni. `tiff`-käitusekaadrid moodustavad faili `raw`, millesse salvestatakse selle kaamera pildid: anduri mosaiik, üks tasand oma väärtustega, tihendamata, koos kalibreerimispaketiga (2048 x 1536 12-bitine kaader on 6,0 MiB pluss pakett, olenemata stseenist), seega `process` alustab neist. `png` ja `jpg` sisaldavad 8 bitti, seega on need demosaikitud kaadrid vaatamiseks, millest `process` ei alusta. DAQ-iga seotuna (`daq pool-bind`) või koondatuna saab iga kaader oma `.daq` ja `.match.json` jooksu `daq/` kausta, kuna `lattice capture` salvestab ühe; `--no-daq` lülitab selle välja ja peatamisel kuvatakse, kui palju neid on. Iga kaader kirjutatakse ühe korra koos täielike metaandmetega; kui vajutada Ctrl+C, kirjutatakse juba salvestatud kaadrid enne kaamera vabastamist. Peatab end ise, kui kõvaketas täitub, väljastab `Failed:` / `Disk full:` ja lõpetab töö `1`; iga kaader, mida ei suudetud salvestada, lõpetab samuti töö `1`. Kaamera, mis on jäetud riistvaralise päästiku jaoks valmisolekusse (`TriggerMode` `On`) seatakse käivituse ajaks vabajooksule. Kui käivitus lõpeb, seatakse kaamera tagasi selliseks, nagu see leiti — selle käivituse seaded ja kõik käivituse käigus muudetud seaded (säriaeg, võimendus, formaat jne) – välja arvatud juhul, kui kasutatakse `--keep-settings`, mis salvestab seansi seaded kaamera enda mällu (vabajooks), või `--clear-settings`, mis taastab selle mälu tehase seadistustele. Jooks, mis ei salvestanud ühtegi kaadrit, loetakse ebaõnnestunuks: kui 10 sekundi jooksul ei salvestata ühtegi kaadrit, peatub see ise, trükib välja `No frames:` koos põhjendusega ja lõpetab töö `1`. Kaadrid, millega kirjutajad ei suutnud sammu pidada, loetakse kadunuks, ja peatamisel kuvatakse `Dropped:` koos põhjendusega (kaamera kaadrisagedus, kirjutajate kaadrisagedus ja järjekord); madalam kaadrisagedus või 8-bitine pikseliformaat säilitab need kõik. Käsk teeb tagasisidet backendiga iga 2 sekundi järel; kui see lõpetab töö ilma salvestamist peatamata (`kill -9`, mälu täis), peatab backend salvestamise 30 sekundit pärast viimast tagasisidet ja paneb kaamera tagasi (`stop_reason` `client_gone` ning rida tagapõhja logis). `multi-stream` ja `align-stream` toimivad samamoodi. Kui kaamera avamine võtab kauem aega, kui käsk ootab (60 s; 180 s `multi-stream` / `align-stream` puhul) — kaamera on veel käivitumas — tühistab backend käivituse: kaamera pannakse tagasi ja suletakse ning salvestamine ei jätku (504 HTTP-viga `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Brauseri-põhine MJPEG-reaalaja eelvaade: ühendab kaameraga, kuvab voo URL-i ja avab selle brauseris. `--ae-damping` seadistab automaatse säri summutuse (0,4–100). |

##### Otseülekanded (MJPEG; RTSP-d ei toeta)

Chloros edastab otseülekandeid **MJPEG-vormingus HTTP-i kaudu**(`multipart/x-mixed-replace`, piir `frame`) kaudu tagapõhja.**RTSP-d ei toetata.** Avage URL-aadress brauseris või lugege seda mõne MJPEG-kliendiga (OpenCV-i `cv2.VideoCapture(url)`, ffmpeg, VLC). Kaamera peab olema esmalt backendiga ühendatud (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` või graafiline kasutajaliides).

| URL (`http://127.0.0.1:5000` + …) | Voog |
| --- | --- |
| `/api/camera/stream/<serial>` | Eelvaade, kuni umbes 30 kaadrit sekundis. Massiivi elemendi puhul määravad `?max_h=` (vaikimisi `720`) ja `?jpeg_q=` (vaikimisi `80`) kõrguse ja JPEG-kvaliteedi. See on URL, mille `lattice viewer` avab. |
| `/api/camera/<serial>/stream-annotated` | Sama eelvaade, millele on joonistatud kasutajaliidese katted: `zebra=1`, `crosshair=1`, `grid=3x3` (või `9x9`), `histogram=1`, `peaking=1`, `spot=1`; kui päringut ei esitata, kasutatakse selle kaamera jaoks projekti salvestatud kihtide seadeid. |
| `/api/camera/<serial>/stream-reflectance` | 8-bitine peegeldusvõime sagedusriba kohta: ρ = piksel ÷`X-Chloros-Pixel-Scale` päises (255, kui `refl_max` ei ole tõstetud). `fps` (vaikimisi `10`), `jpeg_q` (vaikimisi `95`), `max_h` (vaikimisi `0` = algne), `band=<name>`, `align=1`. JPEG, seega kadudega. Keeldutakse HTTP-koodiga 409, kui kaameral puudub peegeldusvõime (RGB-kaamera, kalibreerimispakett puudub, DAQ-näidud puuduvad). |

- **Kaadrite sobitamine kaamerate vahel.**Iga osa, mille**massiivi liige** saadab kõigis kolmes voos, sisaldab `X-Chloros-Group-Id` (massiivi sünkroniseeritud rühma loendur), `X-Chloros-Frame-Id` ja `X-Chloros-Timestamp-Ns` päised. Erinevatest kaameratest pärit osad, millel on sama `X-Chloros-Group-Id`, pärinevad samast käivitajast. Iseseisval kaameral puudub grupp ja see ei saada grupi ID-d. Veebibrauserid ignoreerivad osade pealkirju; klient, kes analüüsib mitmeosalist voogu ise, loeb neid.
- **Peatatud, kui massiivi vaade on avatud.** Massiivi liikme `/api/camera/stream/<serial>` saadab staatilise „peatatud” kaadri, kui massiivi kombineeritud reaalajas voog on avatud (näiteks GUI massiivi vaade), ning jätkab tööd, kui see suletakse.

#### Anduri häälestamine

| Alakäsk | Otstarve |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | GenICam-sõlme lugemine/kirjutamine. `--set ExposureTime=…` (`Gain=…`) ilma selle `ExposureAuto` (`GainAuto`) lülititeta, mis lülitavad automaatrežiimi esmalt välja, nagu seda teeb `lattice exposure --set`; kui kaamera keeldub väärtust vastu võtmast, väljastab ta stderr-ile `Failed to set <node>: …` ja lõpetab töötamise `1`. `--get` ja vaikimisi rühmitatud vaade kuvavad `note:` all `ExposureTime` / `Gain`, kui automaatrežiim töötab — vt [Säriaja ja võimenduse lugemine automaatrežiimis](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Ekspositsioon ja AE. `--set` ekspositsiooni või `--brightness` sihtmärki peetakse sinu omaks (Chlorosi automaatne särijuhtimine ei sekku sellesse); võimendus jääb samaks, seega automaatrežiimis kohandub see sihtmärgi suunas, kuni sa seadistad `lattice gain`. `--auto` annab ekspositsiooni tagasi, `--brightness-auto` annab sihtmärgi tagasi Chlorose automaatsele sihtmärgile. Lipukeste puudumisel loeb see ekspositsioonisõlmi, kasutades sama `note:`-i kui `configure --get`. Kui kaamera keeldub `--set` väärtusest, trükib ta stderr-ile `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` — kui kirjutamine on välja lülitatud, taastatakse automaatrežiim, selle asemel et seda välja lülitatuna jätta — ei väljasta rida `Exposure: … us (auto OFF)` ning lõpetab töö `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Võimendus ja automaatne võimendus. Kui kaamera ei aktsepteeri `--set` väärtust, väljastab ta stderr-ile `Failed to set Gain: …`, ei väljasta `Gain: … dB (auto OFF)` rida ja lõpetab töö `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Anduri ROI ja binning. Binning on alati **keskmine** — vt allpool. |
| `lattice format [--set FMT] [--list]` | Pikseliformaat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Riistvara-/tarkvarapõhine käivitaja. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (lipud puuduvad = ühekordne valgebalanss) | Valgebalanssi operatsioonid. Ainult RGB- ja Bayer-kaamerad; mono-M3M-i puhul ei tehta midagi (vaheletõmmatud). Ühekordne valge tasakaalu määramine säilitab leitud valge tasakaalu (`BalanceWhiteAuto` ja võimendused); kõik muud avatud kirjutatud seaded taastatakse leitud kujul ning kõik need, kui ühekordset valge tasakaalu määramist ei saa käivitada. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | RGB-ekraani värviprotsess. `natural` (vaikimisi) on odav reaalajas töötlemine; `enhanced` lisab värvihäirete eemaldamise + elavuse + CLAHE kohaliku kontrasti, et saavutada täielik hub-parity välimus, mis maksab umbes kaks korda rohkem kaadri kohta, mistõttu on **reaalajas** kaadrisagedus madalam — salvestatud kaadrid saavad igal juhul täieliku töötluse. Ainult RGB/Bayer-kaamerad; jätatakse vahele mono M3M-i puhul. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Reaalaja eelvaate küllastus/kontrast (kaamerad RGB-filtriga); `100` on neutraalne, `--reset` seab mõlemad väärtuseks 100. See ei mõjuta salvestatud kaadreid. Jäetakse vahele mono-M3M-i puhul. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (lipukeste puudumine = loe) | Näita **kõrgeid valgustusi** (kaamerad RGB-filtriga): vähenda kõige heledamaid, valgeimaid piksleid — nt heledat valget pinda, mis kuvatakse ühtlase valge laiguna. `0` = välja lülitatud, `-100` = tugevaim. Reaalajas eelvaade ja kuvatase järgivad seda; kiirgus/peegeldus ei järgi seda kunagi ning see ei suuda taastada klippunud andureid (selleks tuleb vähendada säritust). Jäetakse vahele mono M3M-il. |
| `lattice filter [--list]` | Loetleb filtri mudeleid, mida Chloros tunneb (`RGN-IMX265`, `OCN`, `NGB`, …). Kaamera filter loetakse selle mudelistringist ja seda ei saa seadistada: `--set NAME` otsib ainult mudeli üles, seejärel väljub `1` koos `setting the filter model is not supported`-iga. |
| `lattice power [--sleep]` | Kontrollib toite- ja soojusandureid; lülitab sisse madala võimsusega ooterežiimi. `--sleep` jätab kaamera puhkeolekusse (selle ühenduse läbilaskevõime piirang on sisse lülitatud (`DeviceLinkThroughputLimitMode` `On`) miinimumtasemel, energiasäästurežiim on sisse lülitatud, kui kaameral see olemas on) kõik muud seaded taastatakse sellisena, nagu need algselt leiti; kui magamisrežiimi sisselülitamine ebaõnnestub, taastatakse see täielikult. Ärkamine taastab ühenduse sellisena, nagu see oli enne magamisrežiimi sisselülitamist (piirang, režiim ja paketi viivitus `GevSCPD`, mida kaamera hoiab väärtusel 0, kui piirang on sisse lülitatud; andmed salvestatakse sellele arvutile, nii et ka tagapõhja taaskäivitamise järel toimuv äratamine leiab selle samuti), või kui tegemist on kaameraga, mida see arvuti ei pannud puhkeolekusse, kuid mis näib olevat puhkeolekus (piirang on põrandatasemel sisse lülitatud), lülitatakse piirang välja ja `GevSCPD` jäetakse 0-le tasemel 80 (tehase seaded); piirang, mis on seatud miinimumist kõrgemale, jäetakse muutmata. Andur ja äratamine taastavad selle, mida nende avamine kirjutas. |

> **Binning on alati keskmine.** `lattice resolution --binning N` seab keskmise binningu ja
> ei paku režiimi vahetust. Sum-binnimine korrutab iga DN väärtuse N²-ga, mis maksab umbes kaks stoppi
> heledate alade varu iga binnimissammu kohta; Chloros jagab selle teguri tagasi enne
> radiomeetrilist ahelat, kuid pikslid, mille Sum-binnimine viis anduri ülepiirini, lõigatakse ära ja neid ei saa
> taastada. See ei anna midagi, mida „Average“ ei anna – indeks on skaalast sõltumatu ja kiirguse
> skalaarid on kalibreeritud „Average“ skaalal. Kui kaamera on juba seadistatud „Sum“-ile (jätnud
> sinna mõni teine tööriist või tehase algseadistuse taastamine), tuvastab Chloros selle sõlme tagasilugemisest ja
> kompenseerib; käsu `lattice resolution --binning N` käivitamine seab selle tagasi režiimile „Average“. Versioonis 1.2.0
> `--binning-mode` lipu aktsepteeritakse endiselt, nii et vanemad skriptid töötavad, kuid see kuvab vaid hoiatuse:
> binning jääb „Average“-režiimile.

> **Kaamera binningu, pikseliformaadi ja trigeri muutmine ei jää püsima.** Kui kaamera
> backend jääb avatuks ja edastab voogu (`cam-connect`, graafiline kasutajaliides), siis `lattice resolution --binning`
> ei hakka kehtima (kuvatakse, et binning on endiselt kehtiv, `Binning: 1x1`), ning
> `lattice format --set` ja `lattice trigger --mode` lükatakse kaamera poolt tagasi
> (`SC_ERR_ERROR -1001`). Vabastage see esmalt (`lattice cam-disconnect -s <SN>`) või seadistage
> need ühendamisel (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Ekspositsiooni ja võimenduse lugemine automaatrežiimis

Kui kaamera `ExposureAuto` (`GainAuto`) töötab, säilitab selle `ExposureTime` (`Gain`) säilitab **viimati kirjutatud väärtust**, mitte seda, mida kaamera hetkel teeb. Kaamera puhul, mille tagapõhi on avatud (`cam-connect`, `array-connect`, graafiline kasutajaliides), kuvavad `lattice configure --get`, vaikimisi vaade `lattice configure` ja `lattice exposure` (ilma lipukesteta) seetõttu sõlme väärtuse ning selle all märkuse kaamera uusima kaadri kohta:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Loe kaadri väärtus märkusest; sõlme väärtus on vaid viimane kirjutatud väärtus. HTTP-i kaudu jätab sama `POST /api/camera/<serial>/read-nodes` vastus `nodes` muutmata ja lisab `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, uusima kaadri väärtus) ja `notes` (eelmine lause). Mõlemad on olemas ainult siis, kui automaatrežiim ei ole `Off` ja kaader on teada; ainult lugemiseks avatud kaameral pole kaadrit ja see ei saa kumbagi. `lattice gain` (ilma lipukesteta) väljastab ainult sõlme — märkuse jaoks kasuta `lattice configure --get Gain`.

##### Maatriksi elemendi eksponeerimine

Kui maatriks edastab andmeid, töötab iga element **`AcquisitionFrameRateEnable`**, kaadrisagedusega, mis on veidi suurem kui massiivi triggereerimiskiirus: selle periood on triggereerimisperiood miinus 15 % (vähemalt 3 ms) või kaamera enda maksimaalne kiirus, kui triggereerimiskiirus on sellele juba lähedal. Seega loeb `configure --get` massiivi elemendil selle seadistuse juures `AcquisitionFrameRateEnable`-i kui `True`-i ja `AcquisitionFrameRate`-i; aktiveerimine lülitub välja, kui massiiv peatub.**Ärge kirjutage neid kahte sõlme massiivi liikmele** — massiiv hoiab neid sünkroonis oma triggiga ja kirjutab need ümber.

See seadistus on ka liikme eksponeerimise ülempiir: umbes 85 ms 10 fps triggiga (100 ms periood), **mis tahes ROI või binningu** korral. Enne versiooni 1.3.0 piiras väike ROI elemendi ekspositsiooni hoopis anduri vabajooksuperioodiga (4,9 ms resolutsioonil 768 × 304), olenemata sellest, kui aeglaselt massiiv triggereerus.**Kui käsitsi sisestatud `ExposureTime` on üle piirmäära, aeglustab süsteem triggereid, et see mahuks sisse**, kui seda kirjutatakse `lattice exposure --set` või `configure --set` abil (või graafilise kasutajaliidese kaudu). Maatriks avab kõigepealt selle elemendi ülempiiri vastavale väärtusele ja laseb aeglasemal sagedusel töötada mõne käivitamisperioodi jooksul, mistõttu kirjutamine võib võtta mõne sekundi kauem aega; seejärel käivitub andur mitte kiiremini, kui valgustusaeg seda lubab. Automaatsagedusega maatriksil taastub varasem sagedus siis, kui kõik elemendid, mille käsitsi valgustusaeg seda aeglustas, on tagasi automaatvalgustuse režiimis. Väärtus, mida kaamera ise vastu võtta ei saa, lükatakse endiselt tagasi, veateates näidatakse selle aktsepteeritav vahemik, ning liikme automaatne valgustusaeg taastatakse, kui see oli välja lülitatud ainult selle kirjutamise ajaks. Retsepti `apply` valgustusaeg ei aeglusta päästikut: see hoitakse ülempiiril. Chloros ei lühenda kunagi käsitsi valgustusaega, et see sobiks kiirema päästikuga; fikseeritud sagedus (`array-connect --target-fps F`), mis on kiirem kui valgustusaeg lubab, jätab sellele liikmele piisavalt pika ülempiiri, ning see liige võtab päästikuid mitte kiiremini, kui tema valgustusaeg lubab.

#### Kalibreerimine ja andurid

| Alakäsk | Otstarve |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibreerimine peegeldusmäära sihtmärgi järgi. Selle leidmiseks otsitakse kuni `--attempts` kaadrit (vaikimisi `30`); ebatäielik raam (GVSP-paketi kadu) maksab ühe katse, mitte kogu jooksu. See ebaõnnestub väärtusega `no target detected in N attempts`, või `no complete frame in N attempts: <last error>`, kui ükski raam ei jõudnud tervena kohale. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | DAQ-M spektri ja sagedusribade tugevused, mis loetakse Bluetoothi (BLE) kaudu tagapõhja kaudu. `--mac` on anduri enda BLE MAC-aadress ja on vajalik kõigi kolme toimingu puhul: vaikimisi väärtust ei ole, kuna igal seadmel on oma (`daq pool-discover --only ble` loetleb need); ilma selleta lõpetab käsk töötamise `1`. Kui toimingut ei valita, kuvatakse abiinfo. `--camera SERIAL` integreerib kuvatud ribade amplituudid selle kaamera enda mõõdetud spektraalvastuse kaudu, mitte filtri mudeli kalibreerimiskõvera asemel — RGB-kaamerad, mille kalibreerimispakett on laaditud (ühendatud) või salvestatud selle arvuti vahemällu. Pealkirjas on märgitud, kelle kõverat kasutati: `Band magnitudes (camera <SERIAL>):` või `Band magnitudes (model <filter>):`, kui kaameral seda pole (ei ole RGB-kaamera, või pole paketti laaditud ega vahemällu salvestatud) ning selle asemel kasutati standardkõverat. HTTP-i kaudu võtab `POST /api/dls/cli-read` vastu `camera_serial` ja tagastab `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Rakenda vinjeti korrigeerimist olemasolevatele piltidele, millest igaühel on oma objektiivi tasapinnaline väli (EXIF-mudeli järgi). `--lens-model` kohaldub piltidele, mille mudeli nimetus ei viita ühelegi kaamerale; pilt, mille mudeli nimetus on sellega vastuolus, jäetakse vahele. |

#### Mitme kaameraga (ajutised seansid)

Ajutine käsk avab kaamerad üheks töötsükliks ja sulgeb need selle lõppedes: `multi-capture` ilma püsiva massiiviga, `multi-stream` ja `multi-test` siin, ning allpool `align-calibrate`, `align-apply`, `align-stream` ja `index --live`. Kõigi nende puhul kehtivad järgmised reeglid:

- **`--serials SN1,SN2,…` valib kaamerad.** Avatakse ainult need, ja esimene neist käivitab päästiku (kui on määratud `align-calibrate` ja `--reference`). Seeria, mis ei ole võrgus, põhjustab käivituse ebaõnnestumise enne, kui ükski kaamera avatakse. Ilma `--serials`-ita avatakse kõik võrgus olevad kaamerad. Kaamera, mida Chloros juba hoiab (graafiline kasutajaliides, `cam-connect`, `array-connect`), keeldub käivitamisest, olenemata sellest, kas `--serials` seda nimetab või mitte: see jagab võrku, mille jaoks käivitamine on mõeldud. (`index --live` ilma `--serials`-ita avab selle asemel oma profiili aktiveeritud kaamerad: vt [Indeks / Taimestiku matemaatika](#index--vegetation-maths).)
- **Kaamerad jagavad võrku samamoodi, nagu `array-connect` seda jagab.** Iga kaamera on sünkroniseeritud vastavalt oma osale sellest, mida host-link edastab (pakettidevaheline viivitus, mitte kaamera läbilaskevõime piir, mille 31,25 MB/s alampiiri neli kaamerat 1 GbE-l juba ületavad), seega töötavad neli kaamerat 1 Gbps ühendusel. Kui kaamerad ei suuda kõik korraga saata oma aeglaseima saavutatava kiirusega, käivitatakse need ükshaaval, igaühel oma eraldi ühendus, ja käsk väljastab `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Kui isegi üks kaamera korraga ei mahu, lükatakse käivitus tagasi enne, kui ükski kaamera hakkab andmeid edastama: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP-kood 409, lõpetamine `1`.
- **Kõik kaamerad peavad andmeid edastama, vastasel juhul ebaõnnestub käivitus.** Kaamera, mis ei edasta ühtegi kaadrit, on viga: käsk kuvab `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, kus `<why>` on selle kaamera andmete hankimise viga, seejärel selle põhjused ning lõpetab töö `1`. GVSP-pakettide kadudega saabunud kaader (`IncompleteBufferError: GVSP packet loss …`) tähendab, et võrk ei suuda edastada kaamerate koondribalaiust (`Every missing frame arrived with GVSP packet loss: …`): kontrollige hosti ja lüliti üleslingi ühenduskiirust (`lattice network`), kasutage jumbo-kaadreid, või avage vähem kaameraid käsuga `--serials`. Kaamera, mis ei edastanud midagi, ei saa tavaliselt peakaamera käivitussignaali: kontrollige selle M8-sünkroniseerimiskaablit ja käivitage seejärel käsk `lattice multi-detect`. Iga käsuga seotud toiming kaamerat puuduva raamigrupi puhul on kirjeldatud vastavas reas.
- **Iga kaamera taastatakse sellisena, nagu see leiti.** Kui käivitamine lõpeb, saab iga kaamera tagasi selle, mis tal oli käivitamise alguses: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, ning `LineMode` / `LineSource` asuvad `Line2` ja `Line3` (ning massiivi triggerväljundi glitšide eemaldamine annab orja liini, `LineFilterSelector` / `LineFilterWidth`); selle `PtpEnable` (ja sellega seotud `PtpSlaveOnly` / `PtpDomainNumber`); `ExposureAuto` / `GainAuto` (ja `ExposureTime` / `Gain`, kus see oli käsitsi sisestatud väärtus); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (ja iga `BalanceRatio`, kus valgebalanss oli käsitsi seadistatud); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, piksli ühendamine, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (ja `Gamma`, kus gamma oli sisse lülitatud), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; paketi suurus (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); edastussagedus, `DeviceLinkThroughputLimitMode` (ja `DeviceLinkThroughputLimit`, kus režiim oli `On`),pakettidevaheline viivitus `GevSCPD` ja alguse nihutus `GevSCFTD`, mida massiiv annab iga kaamera jaoks; tükkandmed, mille iga voog aktiveerib (`ChunkModeActive`, iga `ChunkEnable` oma `ChunkSelector` all); `AcquisitionFrameRateEnable` (ja `AcquisitionFrameRate`, kus aktiveerimine oli sisse lülitatud — kui see on välja lülitatud, on kiirus ainult lugemiseks ja loeb maksimaalselt nii palju, kui sagedus lubab, seega ei ole see seadistus). Kirjutatakse ainult see, mis erineb: kaamera peatus esmalt, püsikiirus ja kaadrisagedus enne sellist ekspositsiooni, mis vajab oma kaadriperioodi (kaamera piirab `ExposureTime` ühe kaadriperioodiga). `--keep-settings` korral lülitatakse päästiku- ja sünkroniseerimisjuhtmed välja, kaamera töötab vabalt ning ülejäänud andmed salvestatakse kaamera enda mällu vastavalt käivitussetile; `--clear-settings` taastab seal tehase seaded.
- **Säriaeg määratakse kindlaks stseeni põhjal enne esimese kaadri salvestamist.** Jooks võtab sünkroniseeritud kaadrirühmi ja viskab need ära, samal ajal seadistades iga kaamera valgustust ja võimendust nende kaadrite põhjal. Eesmärgiks on kaamera sihtmärgi keskmine heledus, kus mitte rohkem kui 1 % kaadrist on ülevalgustatud; stseen, milles on eriti heledad alad, jääb sihtmärgist veidi maha. See võtab tavaliselt aega 2–6 kaadrirühma ja maksimaalselt 5 sekundit.
  - Iga kaamera suunab oma filtri abil sihtmärgi poole, millest ühendatud massiiv alustab: sihtmärgi heledus **40**multispektraalkaameral (RGN, OCN, NGB) ja**60** RGB-kaameral. See on pool 80-ga antud eksponeerimise sihtmärgist (RGB puhul kolm neljandikku), mistõttu on heleda peegeldusplaadi ülekärpimise tõenäosus väiksem. `--brightness N` (0–255) määrab selle asemel iga kaamera sihtmärgi (`multi-test`-il puudub `--brightness`: selle automaatne säritus on välja lülitatud). Backendi logifailid nimetavad iga kaamera sihtmäära: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Esmalt määratakse säri, kuni kaamera enda automaatse säri piirini. Võimendus hõlmab ainult seda, mis jääb alles.
  - Leitud valgustatus ja võimendus säilitatakse kogu jooksul, seega võetakse iga `multi-stream` kaadri pildistamisel sama valgustatusega. Pildistamine, mis peab järgima muutuvat valgustust, kuulub ühendatud massiivi (`array-connect`), mille automaatne valgustatus kohandub pidevalt.
  - `--exposure` ja `--gain` säilitatakse täpselt nii, nagu need on määratud. Kui mõlemad on esitatud, ei jäeta ühtegi kaadrit kõrvale.
  - Kui seeria lõpeb, lülitatakse iga kaamera automaatne säritus uuesti sisse.
  - Tagapõhja logi kajastab tulemust: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` või `exposure NOT solved after …` koos põhjendusega ning `(not settled)` iga kaamera puhul, kus seda ei tehtud. Sellisel juhul võetakse kaadrid ikkagi vastu.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` muudab 5-sekundilist piirangut; `0` lülitab selle välja (vt [Keskkonnamuutujad](#environment-variables)).

**Kui Chloros hoiab mõnda kaamerat** (graafiline kasutajaliides, `cam-connect` või ühendatud massiiv), siis `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` ja `index --live` keelduvad töötamast, kuna need avavad kõik võrgus olevad kaamerad ja hoitava kaamera teine avamine hoitud kaamera avamine võib põhjustada tagapõhja kokkujooksmise. (`index --live` keeldub, kui kaamera, mida ta avada tahaks, on hoitud: kas tegemist on `--serials`-iga või selle profiili kaameratega, või mis tahes kaameraga profiili jaoks, milles pole ühtegi kaamerat nimetatud.) Tagapõhi vastab HTTP-koodiga 409 `CAMERA_HELD`, nimetades iga blokeeritud kaamera ja vabastamiskäsud (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`) ning käsk lõpetab töö `1`. Kui massiiv on ühendatud, salvestab `array-capture --continuous` selle `multi-stream` asemel ning `array-capture` (vaikimisi joondatud) salvestab joondatud komplekti `align-apply` asemel; `multi-capture` ja `align-calibrate` kasutavad ühendatud massiivi, mis hoiab nende kaameraid, selle asemel et neid avada.

| Alakäsk | Otstarve |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Loetleb kõik kaamerad, millel on sünkroniseerimisülesanded; käsuga `--serials` loetletakse ainult need, esimene neist on `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Üks sünkroniseeritud kaader igast kaamerast. Salvestab **vaikimisi kõik eksporditüübid**, kui on ühendatud püsiv massiiv, ning kasutatud DAQ-lugemise ([Salvestatud kaadri DAQ-lugemine](#the-daq-reading-a-capture-saves)); ajutine massiivita varuvariant on**ainult debayereeritud** (ülejäänu jaoks käivita esmalt `array-connect`). `--brightness` kehtib ainult varuvariandi puhul: ühendatud massiiv säilitab oma automaatse särituse. Varuvariant ei salvesta `.daq` (`--no-daq` / `--daq-tolerance-ms` ei avalda seal mingit mõju) ning `multi-capture` ei võta arvesse `--force-daq`-i. Varuvariandis kaamera, mis ei edasta ühtegi kaadrit, ebaõnnestub salvestamisel (`Error: multi-capture failed: no frame from …`, väljumine `1`) ja selle kaadrirühma faile ei salvestata. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Sünkroniseeritud kaadrid (ajutine), `--fps` vaikimisi `3`, kuni Ctrl+C või `--count` rühmad. Kaadrirühma, mille jaoks kaamera midagi ei edastanud, ei kirjutata, ja kolm järjestikust kaadrit lõpetavad voo. Kui mõni rühm vahele jäeti, lõpeb töö, väljastades `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, lisaks `Error: the stream stopped: …`, milles nimetatakse kaamera, kui see voo lõpetas, ning lõpetab töö `1`X. Peatub ise, kui kõvaketas täitub, väljastab `Failed:` / `Disk full:` ja lõpetab töö `1`; kui mõnda faili ei õnnestunud kirjutada, lõpetab ka `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO sünkroniseerimise ajastuse test (ajutine), `--count` vaikimisi `10` käivitajad. Kuvab, kuidas võrk on jagatud (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)` ja rida `Serial emit:`, kui kaamerad käivituvad üksteise järel, mille ajalisest nihkest kajastuvad ajamärgid), üks `Sync group: master SN:<serial> -> SN:<serial>, …` rida sünkroniseerimisgrupi kohta (`(no slaves: not synced)` kaamera jaoks, millel pole sünkroniseerimiskaablit ja mis käivitub iseseisvalt), seejärel üks rida iga käivituse kohta: `[  0] SN:<serial>=<timestamp> \| …`, kusjuures `SN:<serial>=(no frame)` tähistab kaamerat, mis selle käivituse puhul midagi ei edastanud. Seejärel `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` ja üks rida `SN:<serial>: mean <±x> us, jitter <y> us` iga kaamera kohta, iga kaamera võrreldes peakaameraga ühe takti jooksul: väljalöök, mille ajamärgid segavad kaamerate seadmekelli ja PTP (PTP välja lülitatud või veel lukustumas) jäetakse välja, kusjuures `Note:` loendab neid. Iga vahele jäänud kaader põhjustab testi ebaõnnestumise (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, väljumine `1`); kolm järjestikust puudulikku triggert lõpetavad testi ennetähtaegselt. Test ebaõnnestub (exit `1`) ka siis, kui vähem kui kahel triggil on kõik kaamerad ühel taktil (`… the clocks cannot be compared`) või kui kellad ei ole kunagi ühtlased: PTP-kellad on masterist rohkem kui 1 ms (pluss mis tahes jadatasandi nihke) erinevad (`… PTP is not locked`) või seadmete kellade nihke suureneb triggereid vahel rohkem kui see. See käivitab PTP-grandmasteri, kui see ei tööta, lülitab iga kaamera PTP sisse (nagu teeb `array-connect`, olenemata võrgu konfiguratsioonist) ning ootab enne esimest triggert, kuni iga kaamera PTP-kell lukustub grandmasteriga ja stabiliseerub: `PtpStatus` orja kell on ühe grandmasteri juures, sellega 100 us piires ja stabiilne 3 s jooksul, ning iga kaamera on selliselt lukustunud, ilma katkestusteta, 120 s jooksul — umbes 2 minutit alates toite. äsja sünkroniseerunud kell on umbes 2 minutit pärast „Slave” näidu ilmumist kuni ~200 us hälbega, mida kaamera enda `PtpOffsetFromMaster` ei näita; 120 s jooksul jõuavad kaamerad stabiilse kella suhtes ~10 us täpsusega. Kaamera on endiselt lukustunud varasema testi ajast, mille järel PTP jäi sisse lülitatuks, nagu see backend seda nägi, loendab alates sellest sünkroniseerimisest, seega üksteisele järgnev test ei oota uuesti; kaamera, mille test sisse lülitab, või kaamera, mis leiti PTP-ga sisse lülitatuna, kuid mille sünkroniseerumist see backend pole näinud (lülitati sisse mujal või enne backendi taaskäivitamist), ootab oma 120 sekundit, ning mõlema kaamera massiivis ootavad just sünkroniseerunud kaamerad endiselt oma aega. Kaamera, mis väljub orjaolekust, ületab 100 us või järgib ootamise ajal kolm lugemist järjest (1,5 s) teist grandmasterit, alustab oma loendamist uuesti; üks või kaks sellist lugemist logitakse (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) ning loendamine jätkub; grandmasteri ID-d, mida ei ole võimalik lugeda, käsitletakse tundmatuna, mitte muutusena. Kui kõikide kaamerate loendamine algab uuesti korraga (sekundi jooksul), on tegemist grandmasteri aja liikumisega, mitte kaamerate omaga: tagapõhja logis on kirjas `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` loetleb selle `grandmaster_steps`-is ning seejärel lõppev ooteaeg algab veaga `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Selle jaoks piiri ei pikendata. `--ptp-settle SECONDS` määrab käivituse stabiliseerumisaja (0–600; `0` tähistab stabiliseerumisaja puudumist). Ooteaeg on piiratud stabiliseerumisajaga pluss 90 s (vaikimisi 210 s, vähemalt 90 s) ning käsk ootab tagapõhja 240 s pluss stabiliseerumisaja (vaikimisi 360 s). See väljastab `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)` ning tagapõhja logis on iga 5 s järel rida, mis sisaldab iga kaamera seisundit ja nihket, kui kaua see on olnud lukustatud ja kui palju aega on veel jäänud: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Iga kaamera lülitub sisse PTP-funktsiooniga välja lülitatuna; sealt lukustuvad kellad umbes 10 sekundi jooksul. Seejärel seatakse iga kaamera tagasi selliseks, nagu test selle leidis – selle PTP, automaatne säri ja võimendus, mille test hoiab käivituse ajaks välja lülitatuna, ning ülejäänud (kui just `--keep-settings`) — seega jääb kaamera, mille PTP on sisse lülitatud (`lattice configure -s <serial> --set PtpEnable=True` või käivitus koos `--keep-settings`-iga), lukustatuks, ja kui see tagapõhi on näinud, et see on olnud lukustatud stabiliseerumisaja jooksul, ei oota hilisem test seda enam. Kui kellad ei stabiliseeru ootamise ajal, ebaõnnestub see enne käivitamist (`PTP: NOT locked …`, väljumine `1`) ja annab juhised, mida teha: kaamera, mis ei lukustunud kunagi (`PtpStatus Listening`), tähendab, et grandmaster ei jõua selle juurde (`time-sync status`); kaamera, mis pidevalt ühendusest välja langes, nimetatakse selle järgi, kui kaua see oli lukustunud (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); kellad on endiselt ühtlustumas: kontrollige `time-sync status`, ja et need oleksid enne testi lukustatud, lülita PTP sisse ja jäta see sisse (`lattice configure -s <serial> --set PtpEnable=True` iga kaamera jaoks või käivita test koos `--keep-settings`-ga; test taastab leitud seaded, seega jääb see sisse), seejärel käivita test. |
| `lattice multi-detect [--line LINE] [--json]` | Leia M8 sünkroniseerimisliinil olevad kaamerad ja vaikimisi määratud peakaamera: RGB-kaamera (mudeli lõpp `-FRGB`), kui see on kaabeldatud, muidu esimene kaabeldatud kaamera seerianumbri järgi. M8-kaablid kohtuvad ühes ühenduspunktis, seega **võib peakaameraks olla mis tahes ühendatud kaamera** — trükitud `MASTER` on vaikimisi valik, mitte ühendusviisi fakt. Iga kaamera juhib omakorda liini ja teiste puhul kontrollitakse selle impulsi olemasolu; kaamerat, mis on ühendatud rühmas juba leitud, ei kontrollita uuesti, seega võtab täielikult kaabeldatud massiiv ühe vooru. Kui `--line` puudub, kontrollitakse `Line2`, seejärel `Line3`. Trükib välja `MASTER: SN:…`, ühe `SLAVE: SN:…` iga teise ühendatud kaamera kohta ja `(probed on <line>)`; `--json` lisab kogu tulemuse (`master`, `slaves`, `no_cable` — kaamerad, milleni impulss ei jõudnud — `line` ning iga kaamera kohta `results`). Kui protsess lõpeb, taastatakse igale kaamerale selle varasemad päästiku ja sünkroniseerimisliini seaded (samad sõlmed nagu eespool). |

#### Joondamine

| Alakäsk | Otstarve |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — pluss detektori/sobitaja nupud `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSACi parameetrid `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, mitme kaadri kombinatsioon `[--averaging mean\|median\|inlier_weighted]`, geomeetrilised piirangud `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, ruumiline piirang `[--roi X0,Y0,X1,Y1] [--mask PATH]` jaorja ülekirjutused `[--per-cam-override SN:KEY=VALUE]` (korratav) | Arvuta joondamisprofiil reaalajas kaamerate põhjal. Vaikimisi: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (selles kaustas, kust seda käivitatakse), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate`-il on oma vaikimisi seaded; vaata [selle tabelit](#project-align-calibrate-options).) `--prefilter`-i vaikimisi seadistus on `gradient` (servakaart; vastab graafilise kasutajaliidese/massiivi joondajale — servad säilivad spektriribade vahel). `--matcher flann` annab tulemusi alates ~5000 tunnusest; `--averaging median` on vastupidav ühele halvale pildile, `inlier_weighted` kaalub vastavuste arvu järgi; `--lock-scale` projitseerib lähimale pööramisele (ilma skaalata), `--lock-axis` nullib ühe transleerimiskomponendi; `--mask` kehtib kõigi kaamerate kohta (kasutage `--per-cam-override` kaamerapõhiste seadistuste jaoks, nt `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` keeldub salvestamast kalibreeringut, mille reprojitseerimise RMS ületab piirmäära. `--method manual` võtab oma punktid `--points PATH`-ist: JSON-objekt `{"<serial>": [[x, y], ...], ...}`, mis annab iga kaamera pikselikoordinaadid samade tunnuste kohta, sealhulgas viitekaamera omad (vähemalt 2 punkti kaamera kohta); iga `--frames` lahendatakse nende punktide põhjal. Kui kaamerad on ühendatud massiivina, kasutab kalibreerimine selle massiivi reaalajas kaadreid ning profiilist saab ka massiivi joondus selle salvestuste jaoks. Muul juhul avatakse [ajutine massiiv](#multi-camera-transient-sessions): kui mõni kaamera ei edasta ühtegi kaadrit üheski `--frames`-is, ebaõnnestub kalibreerimine (väljumine `1`, nimetades seda) ja profiili ei salvestata. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Salvestab ühe joondatud mitme sagedusriba kaadri. `--bit-depth` joondub vaikimisi kaameraga; `--no-crop` säilitab täiskaadri (täidab tühjad kohad mustaga); `--interpolation` (vaikimisi `linear`) ja `--border-mode`/`--border-value` (vaikimisi `constant`/0) juhivad CPU-deformeerimist — GPU-tee on igal juhul bilineaarne. Avab [ajutise massiivi](#multi-camera-transient-sessions): kaamera, mis ei edasta ühtegi kaadrit, ebaõnnestub salvestamisel (väljub `1`, nimetades seda) ja midagi ei kirjutata. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Vooguga joondatud mitme sagedusribaga kaadrid (samad warp-regulaatorid kui `align-apply`), `--fps` vaikimisi `3`. Avab [ajutise massiivi](#multi-camera-transient-sessions): kaadrirühma, mille jaoks kaamera midagi ei edastanud, ei salvestata, ja kolm järjestikust kaadrit lõpetavad voo. Kui mõni rühm vahele jäeti, lõpeb käivitus, kuvades `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` ja `Error: …`, ja lõpetab töö `1`. Ka mis tahes muu vea tõttu lõppev voog kuvab `Error: …` ja lõpetab töö `1`. |
| `lattice align-info --profile PATH [--json]` | Profiili üksikasjade kuvamine. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Kihtide järjestuse muutmine. |

#### Indeks / Taimestiku matemaatika

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` seostab eelseadistuse sümbolid lainepikkustega** (NDVI: `red`, `nir`), igaühele üks `--channel SYMBOL=BAND`.

- **`--live`: ainult sümboli jaoks, millele pildikogum ei suuda vastata.** Nagu ka `--input` puhul, võtab eelseadistuse sümbol, mille välja jätad, selle pinu ühe laine (profiili aktiveeritud kaamerate lained), mille nimi sellega algab. Seda kontrollitakse enne iga kaamera avamist: sümbol, millele ükski sagedusala või mitu sagedusala ei vasta, lükatakse sel juhul tagasi, nimetades need, ning väljutakse `1`. Kohandatud `--formula` vajab iga sümboli jaoks `--channel`-i.
- **`--live`-tooted näitavad, kellele need kuuluvad.** `index.tif` ja `index_LUT.png` sisaldavad pildistamiskomplekti kaamerat (tootja, mudel, seerianumber), selle pildistamise aega (võrdluskaamera säriaeg, UTC) ja objektiivi, ning mõlemad nimetavad indeksit oma üheks ribaks (`XMP-Camera:BandName` `NDVI`; värviline PNG-fail nimetab indeksit, mitte selle värvikanaleid), nagu teeb ka `--input`-toode. Mitme kaamera lainepikkustest koosnev indeks nimetab aga igaüht neist eraldi, ja selle `Model` on `LATTICE` — vaata [Mitmest kaamerast koostatud failid](#files-made-from-several-cameras).
- **`--input`: ainult sümboli jaoks, millele fail ei suuda vastata.** Välja jäetud sümbol võtab failist selle ühe laine, mille nimi sellega algab: `red` võtab `Red_660` või `Red_660_RGN`, `nir` võtab `NIR_850`. Nimed pärinevad failist endast: virna enda ribade loendist või, Chlorosi kiirguse / peegelduse salvestuse või `process` toote puhul selle `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName` sildid. Seega ei vaja `lattice index --input <reflectance capture> --preset NDVI` `--channel`.
- **Kaks ühele sümbolile vastavat laineala ei arvestata** (9-kaameralisel pildistamisel on kaks NIR-lainelaadi): nimetage see. Sümbol, millele midagi ei vasta, lükatakse tagasi `channel_map missing entries`-ga, väljuge `1`-ga.**Kogumi lainealade nimed** sisaldavad kaamera filtrit: M3C-kaamera lainealad on `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` (RGN puhul); `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` kaameral OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` kaameral RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` NGB-l — ning M3M-kaameraainus laine on `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` loetleb profiili laineid.

Täielik lipukomplekt: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (korratav), `--capture-level raw|debayered|radiance|reflectance|unknown` (asendab allikas-TIFF-failis salvestatud salvestustaseme; vaikimisi: loetakse TIFF-i metaandmetest, tagavaraks kasutatakse Chloros-faili `Chloros:ProcessingLevel`-i, seega kajastab peegeldusvõtt `reflectance`), `--output PATH` / `-o` (offline-režiimis: kataloog; pildilaiendiga failid nimetatakse selle tüve järgi — `-o ndvi_FRGN.png` kirjutab `ndvi_FRGN.tif` ja `ndvi_FRGN_LUT.png` — ning ilma laiendita on see eesliide, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; vaikimisi sisendi kõrval, nagu `<input name>_<preset>`; koos `--live` ja ilma `-o`, kaust `output/` selle kausta all, kust käsk käivitatakse), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Alates failist `--live` kehtivad ka joondamise moonutuse nupud: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (kaamerate automaatse särituse sihtmärk, vt ajutiste sündmuste reegleid), `--serials SN1,SN2,…`. `--live` avab [ajutiste sündmuste massiivi](#multi-camera-transient-sessions): kaamera, mis ei edasta ühtegi kaadrit, põhjustab käivituse ebaõnnestumise (väljumine `1`, nimetades selle), ning pinu ega indeksit ei kirjutata. See avab `--serials` (esimene käivitab päästiku), muidu profiili kaamerad, mille `layer_order`-kanne on aktiveeritud, alustades viitekaamerast — viitekaamera isegi siis, kui selle lainealad on keelatud, kuna teised on sellega joondatud sellega joondatud ja see käivitab trigeri. Kaamerat, mille sagedusalad on keelatud, ei kalibreerita kunagi, seega profiilis keelatud RGB-kaamera ei ebaõnnestu enam peegeldusindeksi (`no-absolute-radiance-scale`) puhul. Profiil, mis ei nimeta ühtegi kaamerat, avab kõik võrgus olevad kaamerad, nagu varemgi.

Iga vorm töötab tagapõhiselt, seega peab see töötama: käivitage see Chlorosi graafilisest kasutajaliidesest või käsuga, mille CLI väljastab, kui ta backendile ligi ei pääse. Offline-režiimis loeb backend `--input`-i ja kirjutab ise ujukomaarvulise TIFF-faili ning värvitud PNG-faili, seega lahendab suhteline `--input` või `-o` viitab kaustale, kust käsk käivitatakse (vt [Konventsioonid](#conventions)). Mõlemad offline-tooted sisaldavad sisendi kaamerateavet, nagu seda teeb ka `process`-indeksi eksport: tootja / mudel / seerianumber, GPS, objektiivi andmed, pildistamise aeg (sisendi enda; kui EXIF-andmetes seda ei ole, siis salvestatud säriaeg või failinime ajamärge, UTC-s) ning MAPIR-kalibreerimistunnus – mitte kunagi raw-DN-kalibreerimist, mis indeksile ei kehti – pluss `Chloros:ProcessingLevel=index` ja indeksiväljend; ujukomaarvuga TIFF määrab ühe laineala, eelseadistuse. `--list-presets` ja `--list-gradients` väljastavad tagapõhja eelseadistuste ja gradiendi tabelid. Valem, mida tagapõhi tagasi lükkab, või `--channel`-riba, mida TIFF-failis pole, väljub `1`-st koos tagapõhi põhjendusega.

> **`--channel` sümbolites arvestatakse suur- ja väiketähti.** Sümboli pool peab täpselt vastama eelseadistuse kanalite nimedele (eelseadistustes kasutatakse väiketähti, nt NDVI = `red`,`nir` — kontrolli `--list-presets`)ning ribade pool peab vastama joondatud pildistiku ribanimele (või olema offline-režiimis 0-põhine riba indeks). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` töötab; `--channel RED=660` ebaõnnestub ja tekitab vea `channel_map missing entries` veaga.

#### `lattice index` — mitme lainepikkusega pildistackide maskid

Käsu `--capture-level radiance` või `reflectance` korral võib iga kaamera 3×3 segunemise eemaldamine piirata lainepikkuse väärtuse 0-ni (vt [Segunemise piiramismask](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Mitme sagedusriba pildistakett sisaldab mitme kaamera sagedusribasid ja iga kaamera fikseerib oma sagedusriba, seega kirjutab `--live --save-multiband` pildistaketi kõrvale **ühe maski iga allikakaamera kohta**:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Iga mask on ühe ribaga `uint8` **pildistiku** pikselvõrgustikus (läbi sama joondamise, moonutamise ja kärpimise), sisaldades selle kaamera enda bitte: bit 0 = selle R-kanali riba, bit 1 = G, bit 2 = B. Kaamera kanalist *c* võetud pildistiku sagedusala piiratakse seal, kus on seatud selle kaamera bit *c* (R = 0, G = 1, B = 2).
- Iga sagedusala sisseehitatud päritolu nimetab selle kaamera maski (`unmix_clamp_mask`, suhteline pildistamiskogumi kausta suhtes, või `none`) ja loetava biti (`unmix_clamp_mask_bit`).
- Kaamera, mille segamise eemaldamine midagi ei piiranud, ei saa faili; mono- või RGB-kaamera ei tee segamise eemaldamist ja ei saa samuti faili. CLI väljastab iga maski pildistamissarja all (`clamp mask SN…: …`); vastus loetleb need kui `clamp_masks`.
- Sama käivituse indeks-TIFF- / PNG-fail arvutatakse samast virnast, samal võrgustikul, seega kirjeldavad maske ka seda. `--capture-level raw` ei käivita segamise eemaldamist ega kirjuta maski.
- Stiik ja indeks asuvad otse kataloogis `--output`, seega lähevad maskid kataloogi `--output` enda kataloogi `clamp_masks/` – mitte kunagi teie nimetatud kataloogi kohale. Selles kaustas on segamini erinevad tooteliigid (stak, indeks, värvitud renderdus): vali sellest vajalikud failid, ära anna kogu kausta fotogramm-meetria tarkvarale.

#### Püsivad ühendused (Smart-Prep, GUI-ekvivalentne voog)

Need käsud hoiavad kaamerad tagapõhja-poolis avatuna ka mitme CLI-käivituse vahel.

| Alakäsk | Otstarve |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Lisab ühe kaamera reservi (üks kaamera, mitte massiiv). [Ühised valikud](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` ja `--packet-size` (`standard` = 1500, `jumbo` = 9000 või baiti; `auto`-andurid) rakendatakse ühendamisel (`--exposure` või `--gain` lülitab automaatrežiimi välja). See naaseb, kui kaamera voog on alanud (ootab selleks umbes 15 s, seejärel 10 s esimese täieliku puhvri jaoks) ja väljastab `stream: streaming` või `stream: no_frames -- connected, no frames yet: …`, kui voog ei edastanud midagi — kui kaamera kasutab jumbo-pakette, koos soovitusega uuesti ühenduda käskuga `--packet-size standard`. Kui üks kaamera ühendub (esimene ühendus laadib alla selle kalibreerimise, umbes 70 s), ootab teine kaamerat vajav käsk seda maksimaalselt 5 s, seejärel ebaõnnestub HTTP-veaga 503 (`CAMERA_POOL_BUSY`), nimetades ühenduvat kaamerat; käivita see hetke pärast uuesti. `lattice status` ja kaamerate nimekiri ei oota kunagi: ühenduse keskel olev kaamera kuvatakse ühenduvana koos oma faasiga. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Vabastamine. Kaamera taastatakse sellisena, nagu see ühendamisel leiti (päästiku ühendused, PTP, seaded); `--keep-settings` salvestab selle seaded hoopis oma mällu, `--clear-settings` taastab selle mälu tehase seadetele (vt [Kaamera mälu](#camera-memory----keep-settings----clear-settings)). Kaamera, mis kuulub massiivi, vabastab kogu massiivi. |
| `lattice cam-list` | Loetleb massiivis olevaid kaameraid. |
| **`lattice array-connect`**|**Ühenda püsiv sünkroniseeritud massiiv (SOOVITATAV alguspunkt).** Käivitab täieliku GUI smart-prep-protsessi. 9 kaamera puhul võtab see aega umbes minuti; vahetult pärast kaamerate sisselülitamist võib see võtta mitu minutit. Ootamise ajal kuvatakse iga samm, milleni tagapõhi jõuab, näiteks `    <s> s: <step>` (nt `    41 s: open SN:214000535 (4 of 9)`), ning tagapõhja logis on iga etapi kohta üks ajastusrida (`[array-connect <id>] <step>: <s> s …`) ja viimane rida, mis näitab aeglaseimaid etappe. Ooteaeg on kuni 600 s. Kui ühendust ei ole selleks ajaks loodud, tühistab backend selle: iga avatud kaamera viiakse tagasi sellisesse seisundisse, nagu see leiti, ja suletakse, käsk väljastab `Error:` koos jõutud sammuga ja lõpetab töö `1`, ning ühtegi kaamerate massiivi ei ühendata (HTTP 504 `CLIENT_DEADLINE_PASSED`). Ctrl+C tühistab ühenduse samamoodi (`Interrupted: …`); kui massiiv oli juba ühendatud, näitab rida seda ja nimetab käsu `array-disconnect`, mis selle vabastab. Ootamise ajal katkestatud käsk (`kill -9`), jääb ühendus tühistamata, kui tagapõhi pole sellest 30 sekundi jooksul midagi kuulnud. Massiiv, mis on töös, kuid millel pole veel sünkroniseeritud raamigruppi, kuvatakse endiselt ühendatuna, selle voog on `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Vabasta massiiv. Iga kaamera jäetakse vabalt töötama (`TriggerMode` `Off`), kusjuures kõik muu jääb selliseks, nagu ühendus selle leidis – selle päästiku ja sünkroniseerimisliini seaded (peakaamera`TriggerSource` taastatakse endiseks, selle sünkroniseerimisliin taastatakse sisendiks), selle PTP ja seansi poolt muudetud kaamera seaded – nii toimib liikme suhtes antud ühe kaamera käsk kohe. `--keep-settings` salvestab iga kaameraseaded sellisena, nagu need olid seansi lõppedes, kaamera enda mällu; `--clear-settings` taastab selle mälu tehase seadetele. Vaata [Kaamera mälu](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Loetleb ühendatud massiivid. |
| `lattice array-status [--array-id ID]` | Reaalajas kaadrisagedus (fps), PTP, viimane viga. `ptp_enabled` on `True` ainult siis, kui kõik liikmed loevad PTP-d sisse; `ptp_off:` nimetab liikme, kes seda ei tee. `state:`-rida loeb `restarting (applying settings)` seadete taaskäivitamise ajal, `recovering (…)` siis, kui toite kaotanud liige avatakse uuesti (vt allpool olevat märkust), `stalled (…)`, kui haaramistsükkel töötab, kuid ei avalda ühtegi raamigruppi, ning `stopped (…)`, kui haaramistsükkel on loobunud (ühenda massiiv uuesti). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Üks sünkroniseeritud salvestus reaalajas töötavast massiivist — Ühekordne / Pidev / Intervall / Kiireim. **Vaikimisi on `all`**(üks fail iga asjakohase eksporditüübi kohta iga kaamera kohta),**joondatud** vastavalt rakenduse salvestusele (`--no-aligned` originaalfailide puhul). Vahelejäänud kaamerad (nt RGB, mis on välja jäetud kiirguse/peegelduse andmetest) kajastatakse koodiga `Skipped: SN:<serial> (<reason>)`; kasutatud salvestust lugenud DAQ-i näid salvestatakse koos sellega, lisatakse ainult tooresalvestus ning kajastatakse koodiga `DAQ: <path>` ([DAQ, mis salvestust loeb, salvestab](#the-daq-reading-a-capture-saves)). Vaata [Salvestusrežiimid, salvestusseadmed ja offline-taas töötlemine](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Salvestab reaalajas kombineeritud indeksivaate videona/GIF-ina (seirekvaliteediga; kui GUI-vaade pole avatud, renderdab tagapõhi massiivi NDVI-väärtuse või `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Kõrge kaadrisagedusega toores Bayer-pildiseeria (analüüsikvaliteediga; töötle offline-režiimis). Nagu `array-record` ja `array-capture --continuous`, teeb see iga küsitluse ajal tagasisidet backendiga: lõpetatakse ilma peatamiseta (`kill -9`), backend peatab selle 30 sekundit pärast viimast ühenduse kontrollimist (`stop_reason` `client_gone`; sarivõtte manifest viiakse lõpule ja selle kombineeritud indeksi koostamine algab, nagu peatamisel tavaliselt juhtub) ning massiiv jääb ühendatuks. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Töötle salvestatud töötlemata andmepakett ümber kalibreeritud videoks (videodeks). |

> **Kaamera, mis kaotab toite või ühenduse, taastub ise.** Kui mõni liige lakkab vastamast (lüliti toite tsükkel taaskäivitab kõik selle poolt toidetavad kaamerad), ootab massiiv seda, avab selle uuesti, kui see vastab, ja taastab selle seaded, mille `array-connect` määras: PTP, päästiku- ja sünkroniseerimisliini, võrgu sageduse, pikseliformaadi ja kaadri suuruse ning automaatse säri. Seejärel jätkab ta voogedastust ilma uue `array-connect`-i seadistamiseta. Samal ajal näitab `array-status` väärtust `state: recovering (...)` ja `last_error` näitab, millist kaamerat oodatakse; seejärel väljastab süsteem väärtuse `re-opened:`. Taaskäivitatud kaamera naaseb PTP-režiimiga välja lülitatuna, seega kuni selle uuesti avamiseni on `ptp_enabled` võrdne `False`-ga ja `ptp_off:` nimetab seda.

> **Hoidmisel olevat kaamerat ei avata kunagi teist korda.** Teine avamine peatab hoidja voo ja kirjutab selle seaded ümber ning võib põhjustada tagapõhja kokkujooksmise. Seega kaamera puhul, mida hoiab pool (GUI või `cam-connect`), massiivi liige või kaamera, mida hoiab taustatöö (käiv `lattice continuous`, `multi-stream` või `align-stream`, kas enda või mõne teise kliendi oma), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, ühe-shot `white-balance` ja `presets --apply` keelduvad HTTP-koodiga 409 `CAMERA_HELD`, nimetades omaniku ja vabastamiskäsu (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>` või töö puhul selle ID ja kes selle käivitas) ning lõpetab `1`. Sama kehtib ka kaamera kohta, mida mõni teine käsk hetkel kasutab (`multi-test` või `multi-capture`, mis avab selle kaamerad, veel üks ühekordne käsk), iga siin nimetatud käsu puhul: veateade ütleb, et tuleb oodata, kuni see käsk lõpeb. `lattice power` ilma `--sleep`-ita loeb läbi hoidja enda käepideme, kui see on olemas (pool, massiiv, töö avatud kaamera), ning lükatakse tagasi, kui kaamerat kasutab mõni teine käsk. Iga `CAMERA_HELD` vastus sisaldab koodi nii `code` kui ka `error_code`. Kõiki kaameraid avavate käskude kohta vaata [Mitme kaameraga (ajutised seansid)](#multi-camera-transient-sessions).

##### `array-connect` valikud

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--serials SN1,SN2,…` | avasta automaatselt kõik LATTICE-kaamerad (vajalik ≥2) | Esimene seerianumber on MASTER. Kui see jäetakse välja, filtreeritakse avastamine LATTICE (M3C / M3M) mudelite järgi ja ühendatakse need kõik. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO sünkroniseerimisliin. |
| `--target-fps F` | automaatne | Masteri päästiku tulistussagedus. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automaatne | Tasevaliku ümberkirjutamine. |
| `--wire-ceiling-mbps MB_PER_S` | automaatselt tuvastatud | **Host-seadme püsiv andmeside maht, mõõdetuna MB/s — number, millest sõltub kogu massiivi eraldus.** Vähenda seda, kui massiiv teatab GVSP-rikkis raamidest: automaatne väärtus tuletatakse võrgukaardi teatatud ühenduskiirusest, mis ülehinnab USB-adaptereid, kitsaid PCIe-kanaleid ja hõivatud jagatud võrgustikke. Säilitatakse projekti massiivi salvestusplokis, seega taastub see uuesti avamisel / CLI-s / SDK-s ühenduse taastamisel. Vaata [Massiivi tervis](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | puudub (täissensor) | Riistvaraline binning (alati keskmine). |
| `--width N` / `--height N` / `--pixel-format FMT` | täissensor; kaamerapõhine formaat | Iga kaamera jaoks selgesõnaline kaadri suurus või pikseliformaat, mitte täissensori ja kaamerapõhise formaadi valik (Mono12 / BayerRG12). |
| `--no-recommend` | välja lülitatud | Võrguanalüüsi etapp vaheletakse (iga kaamera säilitab seejärel oma praeguse suuruse, selle asemel et see seataks täissensorile). |
| `--no-ptp` | välja lülitatud | Lülita PTP välja (kaameratevahelised ajamärgised ei ole seejärel **võrreldavad**). |
| `--packed {auto,on,off}` | `auto` | Edasta 12-bitiseid kaadreid pakitud kujul ja lae need lahti sellel arvutil — rohkem kaadreid sekundis täissensori korral, samad andmed. `auto` pakib ainult siis, kui see on kasulik ja kui arvuti suudab seda teha; `off` ei pakenda kunagi. Vaata [Pakendatud 12-bitine edastus](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | GVSP-paketi suurus iga kaamera jaoks massiivis. `auto` lubab igal kaameral võrku uurida; `jumbo` = 9000, `standard` = 1500 või baidides väljendatud suurus piirab seda. Enne versiooni 1.3.0 võttis `array-connect` selle lipu vastu ja ignoreeris seda. |

> **Vaikimisi täissensor.** Kui `--width` / `--height` / `--binning` /
> `--pixel-format`, seadistab `array-connect` iga kaamera täisandurirežiimile — ilma
> binninguta, ei ole ROI-kärpimist — ning laseb kaadrisagedusel langeda sellele tasemele, mida see arvuti ja
> võrk suudavad pakkuda, selle asemel et kaadreid vähendada, et sagedust ülal hoida.
> Võrguanalüüsi rida näitab prognoositavat sagedust ning järgmine rida loetleb
> kiiremaid valikuid (piksliühendamine või väiksem ROI) koos nende prognoositavate sagedustega; vali
> `--binning` või `--width`/`--height`, et valida üks neist. Connect ei kasuta kunagi ise piltide ühendamist ega kärpimist
> .

##### Pakitud 12-bitine edastus (`--packed`)

Täisanduriga massiivi kaadrisagedus sõltub sellest, kui kiiresti iga kaamera suudab
oma kaadreid saata. 12-bitised formaadid (`BayerRG12`, `Mono12`) edastatakse tavaliselt 16-bitiste
sõnadena; **pakitud** edastus saadab samad 12 bitti 1,5 baiti piksli kohta ja
pakib need lahti siin arvutis, mistõttu iga kaadri saatmine võtab 25% vähem aega —
umbes **33% rohkem kaadreid sekundis täisandurirežiimis** (mõõdetud: 10,7 → 14,0 kaadrit sekundis
9-kaameralisel massiivil). **Salvestatud andmed on identsed**: samad väärtused, samad
16-bitised failid, samad metaandmed.

- `auto` (vaikimisi) pakib andmeid ainult siis, kui see on kasulik **ja** kui käesoleval arvutil on piisavalt ressursse
  lahtipakkimiseks. Ühenduse loomisel mõõdab tagapõhi oma lahtipakkimise kiirust, protsessorite arvu,
  hetkelist protsessorikoormust ja võrguvõimsust, nii et arvuti, millel on vabu tuumaid, pakib
  täisanduril, samas kui hõivatud või madala võimsusega arvuti edastab andmeid lahtipakitud kujul. Väikesed huvipiirkonnad ja binnitud kaadrid võidavad
  vähe.
- `off` ei pakenda kunagi — kasuta seda pakendamise välistamiseks diagnoosimise ajal või
  arvutis, mis peab kogu oma protsessorivõimsuse hoidma muude ülesannete jaoks.
- `on` pakib alati 12-bitiseid formaate.

Ühenduse väljund näitab otsust ja selle põhjendust (`pakitud 12-bitine: (auto:
prognoositud 11,0 -> 14,4 kaadrit sekundis, …)`); `/api/camera/array/status` teatab
`packed_transport` ja `packed_decision`. Et määrata kogu masina hõlmav vaikimisi seade
selle asemel, et iga kord lipukest edastada, seadista `CHLOROS_ARRAY_PACKED12=off` (või
`on`) tagapõhja keskkonnas – selgesõnaline `--packed on|off` on siiski ülimuslik.
8-bitiseid ja 16-bitiseid formaate ei pakita kunagi.

### Üks projektikirje, kõik režiimid

Projekti kaust sisaldab iga seadme seadeid: `cameras.json` – iga kaamera omad (`cameras[].settings`, ühe kaamera ja iga massiivi liikme jaoks) ning iga massiivi kogu massiivi hõlmavad seaded (`arrays[].capture`: formaat, ROI, binning, käivitussagedus, andmeedastusmaht) — ja `sensors.json`, iga valgusanduri omad (integratsiooniaeg, kaadri keskmine, piirang). Rakendus, CLI, SDK ja MCP-server loevad ja kirjutavad samu faile samal viisil:

- **Projektiga ühendatud seade salvestatakse sellesse ja salvestab sinna iga muudatuse.** „Projekti alla” tähendab, et tagapõhis (rakenduse) on projekt avatud, või — SDK-st — läbi `ChlorosProject`-i, mis määrab oma kausta nime. Kaamera, maatriks või valgusandur, mida projekt veel ei sisalda, lisatakse selle failidesse, kui see ühendub (`lattice cam-connect`, `array-connect`, `daq pool-connect`, rakendus, SDK, `chloros-mcp`) koos selle seadistustega. Muutus mis tahes režiimis (`lattice exposure`, `gain`, `configure --set`, `resolution`, massiiviseaded ja käivitussagedus, valgussensori piirang, rakenduse juhtnupud) salvestatakse hetke pärast selle projekti failidesse. Kaamerale seatud säriaeg, võimendus või AE-sihtmärk salvestatakse kui hoitud, seega avamisel säilitatakse need uuesti.
- **Projekti avamisel seadistatakse selle seadmed vastavalt projektile** igas režiimis: kaamera, maatriks või valgusandur, mis on ühendatud ilma oma seadistusteta, saab projekti salvestatud seadistused (`lattice cam-connect` / `array-connect` avatud projektiga, `chloros-cli project connect`, `open_project().connect_all()`, rakenduse uuesti avamine). Uue ID all ühendatud massiiv uuendab projekti salvestatud massiivi samade kaameratega; teiste kaameratega massiiv lisatakse selle kõrvale.
- **Rakenduse enda salvestamine ei kirjuta kunagi üle seadme hetkeseadeid** nende vanema koopiaga ning säilitab iga liikme puhul CLI- või SDK-kutsuja poolt salvestatud ümberkirjutused.
- **Teie määratud pikseliformaat on see, mida projekt säilitab.** „Array Settings“ (Massiivi seaded) kaudu, `--pixel-format` või `connect_array(pixel_format=)` abil määratud formaat, sealhulgas 8-bitine, salvestatakse ja taastatakse täpselt samasugusena. Kui midagi pole määratud, ühendub iga kaamera oma 12-bitise vaikimisi seadistusega (`BayerRG12`, `Mono12` monosensoril). Formaadi nimi tähistab selle bittisügavust; perekond on sensori oma, seega värvikaamerat ei seadistata kunagi monofoorseks formaadiks ega monokaamerat Bayeri formaadiks, ning rida `[PixelFormat <serial>] ... written` näitab seda, kui formaat on kaardistatud. 8-bitise salvestuse ekspordid on 8-bitised, sealhulgas peegeldus: ümberskaalatud ja märgistatud selle `PixelScale`-ga.
- Projektita ühendatud seade ei salvestada midagi.
- **Salvestus järgib pärast seda loodud projekti; avamine jätab selle oma projekti juurde.**Kui luuakse projekt – kas SDK-i `create_project()`, `chloros-mcp` või rakenduse**Uus projekt**– või duplikeeritakse (rakenduse**Duplicate**, mis avab koopia), samal ajal kui salvestus kirjutab endiselt varem avatud projekti, jätkub salvestamine ja salvestatud andmed jõuavad uude projekti, samasse kohta selle sees (allpool). Olemasoleva projekti avamine — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, rakenduse**Ava projekt** — jätab salvestuse jätkuma selles projektis, kuhu see salvestatakse: see lõpeb seal ja selle failid lisatakse selle projekti faililoendisse (rakendus peatab iga salvestuse enne projekti avamist).
  - `array-record` ja `array-burst`: video või iga kaadri sisaldav sarivõtte kaust liigub sinna tervikuna, kui salvestamine peatub (selle `--duration`, Ctrl+C). `Burst dir` on uus fail, millest lähtuvad `--build` ja `array-build-video`. Kui kõvaketas keeldub faili teisaldamisest, jääb salvestus sinna, kuhu see kirjutati;lõpu logis on kirjas põhjus, samuti ka peatamise HTTP-vastuses (`relocation_error`). Peatatud salvestusjärgu toorkaadrid ja `.daq` lisatakse selle projekti faililoendisse, kuhu see lõpuks satub; videot (`array-record`, seeria põhjal koostatud video) projekti faililoendisse ei lisata.
  - `array-capture --continuous`: iga pärast muudatust tehtud salvestus salvestatakse uude projekti; enne muudatust salvestatud jäävad endisele kohale.
  - `daq pool-record` ja `project sensor log` (ning rakenduse enda valgusanduri salvestus): `.daq` liigub uue projekti`light_sensor` kausta, kui salvestamine lõpeb, ning lisatakse selle projekti faililoendisse.

  Salvestus avatud projektist väljaspool asuvasse kausta (`-o` mujal) jääb sinna, kus see on, ning `lattice continuous` ja `multi-stream` säilitavad neile määratud kausta.
- **Rakenduses salvestatakse uue projekti loomise järel tehtud salvestused sellesse.** Projekti loomine või kopeerimine jätkab kõigi salvestusprotsesside ja salvestuste tööd ning iga pärast seda tehtud salvestus („Salvesta kõik”, „Alusta salvestamist”) salvestatakse uude projekti ja lisatakse selle faililoendisse. Olemasoleva projekti avamine rakenduses peatab kõigepealt kõik need protsessid.
- **Projekti ei saa salvestamise ajal teisaldada.**Avatud projekti teisaldamine teise töökataloogi (rakenduse**Seaded → Töökataloog**; CLI-käskudega projekti üle kanda ei saa, seda teeb HTTP-klient, kes saadab andmeid aadressile `/api/move-project`) lükatakse tagasi HTTP-veaga 409 `PROJECT_RECORDING`, kui massiivi salvestamine või seeria, massiivi pidev pildistamine, `lattice continuous` / `multi-stream`-ülesanne või valgusanduri salvestus sinna kirjutab: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Rakendus keeldub samamoodi, enne kui ta küsib kausta, samal ajal kui töötab tema enda „Capture All”, „Start Capturing” või valgusanduri salvestus.

See on eraldi sellest, millised on kaamerate seaded ühenduse katkestamisel (vt [Kaamera mälu](#camera-memory----keep-settings----clear-settings)): projekt seadistab oma kaamerad ise avamisel.

### Kaamera mälu — `--keep-settings` / `--clear-settings`

Kui massiiv on ühendatud, võtavad selle kaamerad üle massiivi seaded, mida saate seejärel muuta. Kui Chloros vabastab kaamerad — `array-disconnect`, ajutise käsu lõpp (`multi-capture` ilma massiivita, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), `lattice continuous` peatamine, kaamera `lattice capture` lõpetamine, mida see ei hoia — see paneb igaühe tagasi selliseks, nagu see oli enne ühendamist: selle päästiku ja sünkroniseerimisliini juhtmestik, PTP ja kaamera seaded (täielik nimekiri on jaotises [Mitme kaamera (ajutised seansid)](#multi-camera-transient-sessions)).

Kaamerate kasutamiseks väljaspool projekti kirjutavad kaks lipukest mis tahes neist käskudest kaamera **enda mällu** — sellega, millega see mis tahes tarkvaras sisse lülitub:

- **`--keep-settings`** salvestab iga kaamera seaded sellisena, nagu need olid seansi lõppedes kaamera mälus (UserSet2, millest moodustub käivituskomplekt). Kaamerad jäävad endiselt vabalt töötama (`TriggerMode` `Off`), sünkroniseerimisliini juhtmestik jääb selliseks, nagu see leiti. Kaamera, millel puudub UserSet2, salvestab seaded UserSet1-i; see arvuti mäletab seda, mistõttu Chlorose massiivi eneseparandus — mis taastab UserSet1-i, mis on kaldunud Chlorosi baasjoonest kõrvale — jätab selle puutumata.
- **`--clear-settings`** taastab iga kaamera mälu tehase seadetele: Chlorosi baasjoonele, mille massiivi iseparandamine kinnistab — tehase vaikimisi seaded koos `TriggerMode`, `Off`, `TriggerSource` ja `Line2`, automaatne säri, RGB-kaamera kalibreeritud valgebalanss säilitatakse — UserSet1-is, mis on määratud käivitusseadistuseks.
- Neid kahte ei saa kombineerida. Iga kaamera saab oma rea: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` või `SN:…: camera memory reset to the factory settings.`; vastuses `camera_memory` on sama iga kaamera kohta.

Rakendus neid lipukesi ei paku.

**Iga väljumistee taastab kaamerad algseisundisse.**Sama taastamine toimub igal väljumisteel, mitte ainult eespool nimetatud käskude korral: `cam-disconnect` (ja rakenduse kaamera**X**); rakenduse**X** massiivis või mis tahes massiivi elemendi real (massiiv ei saa töötada ilma elemendita, seega vabastatakse kogu massiiv); rakenduse sulgemine (see annab tagapõhjale kuni 20 sekundit, et paigutada kõik kaamerad, massiivi ja töötava `lattice continuous` tagasi enne, kui ta selle peatab; sel hetkel veel töötav käsk — ajutine käsk, mis avab oma kaameraid või ootab nende PTP-kelli, ühekordne `lattice capture`, `power`, `white-balance` või `presets --apply` – peatub järgmisel ohutul hetkel, taastab esmalt oma kaamerad kuni 8 sekundi jooksul, ja vastab HTTP-le 503 `SHUTTING_DOWN`; endiselt töötav ühendusmassiiv — `lattice array-connect`, rakenduse ühendus — peatub enne järgmist kaamerat ja paneb avatud kaamerad tagasi kuni 10 sekundiks, ja vastab HTTP-koodiga 503 `SHUTTING_DOWN`; ning üks, mis saabub sulgemise ajal, ei ava midagi: HTTP 503 `SHUTTING_DOWN`); ja CLI-käsk, mis lõpetatakse või mille terminal suletakse (SIGTERM / SIGHUP peatavad selle töö — ja paneb kaamerad tagasi — enne väljumist `143` / `129`, nagu Ctrl+C). `lattice continuous`, `multi-stream` või `align-stream`, mis lõpetatakse kohe (SIGKILL) ja mis ei saa midagi peatada, mille töö peatatakse ja kaamerad pannakse tagasi tagapõhja poolt 30 sekundit pärast viimast sisselogimist. Tagapõhi, mis kukub kokku või suletakse jõuga, ei saa midagi taastada; toite väljalülitamine ja sisselülitamine taastab iga kaamera mälu, ning järgmine Chloros-ühendus ühendab uuesti kaamera, mille ta leiab valmisolekus olevana.**Projekt seadistab oma kaamerad ise.** Kui massiiv ühendub avatud projektiga, saab iga liige projekti poolt selle jaoks salvestatud seaded – sellele määratud säriaja, võimenduse või AE-sihtmärgi (mida peetakse sinu omaks, seega automaatne säri seda ei muuda) ning valgebalansi režiimi (Chlorosi korrigeerimine; kaamera enda valgebalanss jääb välja lülitatuks) – olenemata sellest, mida kaamera mälus hoiab.

### Smart-AE / Smart-Capture

LATTICE-massiivid käivitavad taustal pideva automaatse särituse (AE) niipea, kui nad on ühendatud, kuid äsja suunatud stseeni puhul kulub hetk, enne kui pilt stabiliseerub. `array-capture --smart` on **valmis lahendus**: see ootab, kuni AE on stabiliseerunud kõigis massiivi kaamerates, ja käivitab seejärel pildistamise. Kasutage seda, kui vahetate stseene seansi keskel.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Vaikimisi on stabiliseerumispoliitika konservatiivne: 5-sekundiline ooteaeg, 1,5-sekundiline stabiilsusaken, ±5 % ekspositsiooni hajuvuse tolerants. Kui vajate automaatikalt teistsugust käitumist, saate seda SDK-i (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) kaudu häälestada.

#### Kuidas automaatne ekspositsioon toimib

- **Esmalt valgustus, viimasena võimendus.** Kui võimendus on režiimis „Auto”, jääb see 0 dB-le, kuni valgustus on saavutanud lubatud maksimaalse väärtuse.
- **Käsitsi valgustuse seadistamisel jääb võimendus automaatrežiimile**, kuni te võimendust muudate või seadistate käsitsi režiimile. Tagasi automaatrežiimile minnes on võimenduse piir sama, mis varem.
- **Väikesed heledad alad ei muuda stseeni tumedamaks.** Kuni 1 % kaadrist moodustavat peegeldust ignoreeritakse ning teadaoleva kalibreerimismärgi paneelidel kuni 0,5 % paneelist.
- **DAQ-i ekspositsiooni ülempiir kehtib ainult siis, kui kalibreerimismärk on vaateväljas.** See hoiab ära valge paneeli ülevalgustumise; kui paneeli ei ole näha, seadistab kaamera säriaja vastavalt stseenile.
- **Püsiv valgus jääb stabiilseks ja muutuv valgus stabiliseerub sekundite jooksul.** Püsiva valguse korral jäävad säriaeg ja selle piir muutumatuks ning pärast suurt valguse muutust stabiliseerub pilt mõne sekundi jooksul. Ülevalgustus ei piira enam säriaega püsivalt ning huvipiirkonna (ROI) muutumisel leitakse säriaeg uuesti.
- **Heleduse sihtmärk jääb saavutatavaks.** See ei tõuse, kui kaamera on oma valgustuse piirväärtusel ja selle võimendust ei saa suurendada, seega tagasi tulev valgus ei tekita pildis ülevalgustust.
- **Kaamera, mille automaatne säritus ei reageeri enam, hoitakse selle hetke valgustingimustele vastaval tasemel.** Eredas päikesevalguses võib kaamera langeda lühimale säritusajale ja jääda sinna tumeda kaadri juurde, kuigi kõik selle seaded on normaalsed. Chloros jälgib kaadrit, hoiab kaamerat selle valguse jaoks sobival tasemel ning vabastab selle, kui valgus muutub. Tume või varjatud valgusandur seda ei häiri.

### Pildistamise eksporditasemed (`all` vaikimisi)

Alates käesolevast versioonist, `lattice capture`, `lattice multi-capture` ja `lattice array-capture` **vaikimisi `--processing all`** — üks salvestatud fail eksporditüübi kohta, mis kehtib iga kaamera puhul, vastavalt kasutajaliidese funktsioonile „Capture All” (Kõik salvestada). Tasemed on järgmised:

| Tase | Väljund | Kehtib |
| --- | --- | --- |
| `raw` | Ühekanaliline Bayer (mustvalged kaamerad: üks lainepikkus) otse sensorist. | Kõik kaamerad. |
| `debayered` | 3-kanaliline värv, salvestatud **RGB**kanalijärjekorras (TIFF-fail RGB fotomeetrilise tõlgendusega; PNG/JPEG avatakse samuti RGB-na): kalibreerimispaketi tumeda nihke (DSNU) väärtus on toorandmetest eemaldatud, seejärel rakendatakse OpenCV servadearvestav demosaik – ilma lineaarsus-LUT-ita, tasandatud väljaga või segamiseta (mustvalged kaamerad: 1-kanaliline halltoon). Mälus demosaikitud kaadrid, mida Python'i SDK tagastab (`grab()`, `frame_stream()`), on**BGR**-järjekorras, nagu OpenCV-s; ainult salvestatud fail on RGB-formaadis. | Kõik kaamerad. |
| `radiance` | float32 W/m²/sr/nm täieliku radiomeetrilise ahela kaudu. | Ainult multispektraalsed (M3C/M3M) — **jätatakse vahele RGB-filtriga kaamerate puhul**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-valmis. | Ainult multispektraalne ja **ainult siis, kui DAQ on seotud + kaamera on kalibreeritud**; muidu jäetakse vahele. |
| `display` | Täielik GUI-eelvaate ahel (CCM + WB + gamma vastavalt kaamera profiilile). Salvestatakse kausta `preview/`. `lattice capture` aktsepteerib selle jaoks ka nime `preview`; `array-capture` / `multi-capture` aktsepteerivad ainult `display`. | Kõik kaamerad. |

Edasta üks tasand, et salvestada just see üks (`--processing debayered`). Kui küsid `all`, jäetakse tasandid, mis antud kaamerale ei kehti, vahele (ja neist teatatakse), kuid viga ei teki — ühendamata või kalibreerimata kaamera saab ikkagi `raw` / `debayered` / `preview`.

#### DAQ-i poolt salvestuse lugemisel salvestatud andmed

Iga kord, kui DAQ on seotud kaamera või kaameramassiiviga või koondatud, kirjutatakse salvestuse lugemisel kasutatud allapoole suunatud andmed faili **`.daq`** faili, mis asub pildistuste kausta alamkaustas `daq/`, koos selle `.match.json` (kui kaugel lugemine oli ekspositsioonist) ja `.cameras.json` (kelle lugemine see on), ning esitatakse real `DAQ:`. See kehtib iga taseme kohta, sealhulgas toores või debayered pilt, nii et iga pilti saab hiljem uuesti töödelda kiirgusvõimsuseks, peegeldusvõimsuseks või indeksiks. Tegemist on sama failiga, millel on sama nimi ja lisafailid, olenemata sellest, kas pilt pärineb graafilisest kasutajaliidesest (GUI), käsurealiidesest (CLI) või tarkvararakendusarenduskitist (SDK). Kui DAQ-näitu üldse pole, puudub ka `.daq`, ning `multi-capture`, millel puudub seotud massiiv (ajutine varuvariant) ei salvesta midagi.

| Lipuk | Mõju |
| --- | --- |
| `--no-daq` | Ära salvesta lugemit salvestuses, mille tasemed seda ei kasuta (`raw`, `debayered`). `radiance`, `reflectance` või `display` tase kirjutab selle siiski, sest uuesti importimiseks on vaja lugemit, mille alusel toode valmistati. |
| `--daq-tolerance-ms MS` | Kui kaugel võib lugemine eksponeerimisest olla ja ikkagi jääda tolerantsi piiresse (vaikimisi 500). Muutub ainult otsus (`.match.json`, `in_tolerance` ja `tolerance_ms`, peegeldusvõime puhul `DlsInTolerance`), mitte kunagi kasutatud mõõtmistulemust. Vaata [DAQ-näit, mis on kaugel ekspositsioonist](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Vanem viis `.daq`-i küsimiseks ainult toorandmetega salvestuse puhul. See on nüüd vaikimisi seadistus ning `lattice capture` ja `array-capture` aktsepteerivad seda lipukest endiselt (`multi-capture` ei aktsepteeri). `--no-daq` on sellest üle. |

Milline DAQ: kas kaameraga või maatriksiga seotud (GUI valgusanduri valik, projekti salvestatud seos või `chloros-cli daq pool-bind`), muidu tagapõhja uusim. Toores pildiseeria (`array-burst`) säilitab endiselt iga eraldi lugemise jaoks oma `.daq`-faili sarja kausta all, nagu varemgi.

**PNG- ja JPEG-failid sisaldavad 8 bitti kanali kohta, seega kasutavad nad ainult `display` / `preview` ja indeksitooteid.** `--format png` või `jpg`, mille tase ületab nende mahutavuse, lükatakse tagasi enne andmete salvestamist, nimetades taseme ja selle mahutava formaadi (TIFF):

- `radiance` (float32) ja `reflectance` (uint16, `32768` = 1,0) — alati, seega lükatakse tagasi ka `--processing all` (vaikimisi) koos `--format png`/`jpg`-ga: kasutage `--format tiff` või `--levels display`.
- `raw` ja `debayered` kaameralt, mis edastab rohkem kui 8 bitti (12-bitised M3C / Mono12 M3M-kaadrid) — CLI-liides ei näe kaamera bittisügavust, mistõttu tagapõhi lükkab need tagasi (HTTP 400). Ühekaadriline `lattice capture --processing debayered` salvestab 8-bitise kaadri PNG- ja JPEG-vormingus ning seda ei lükata tagasi, samuti mitte `multi-capture`-i, millel puudub püsiv massiiv (8-bitine debayering).

Iga salvestusmarsruut vastab samamoodi (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst` ning nende taga asuvad HTTP-marsruudid.

### Kuidas salvestuskaust välja näeb

Iga eksporditüüp salvestatakse oma **eraldi alamkaustasse**kausta `-o` all, seega mitmetasandilises salvestuses ei segune tüübid kunagi. Joonisel on näidatud**`all`-i poolt ühe kaamera jaoks loodud vaikimisi salvestus `lattice capture`** – iga kaamera kohta on mitu tasandit, mistõttu iga failil on laiend `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Kuidas failinimed varieeruvad:

- **Kaamera väli — seerianumber, seejärel filter:** iga salvestusfail nimetab selle teinud kaamera `SN<serial>-<filter>`-iks, kus `<filter>` on kaamera mudeli filtri osa: `SN261902142-F450` vastab `M3M-L87-F450`-ile, `SN213800234-FRGN` mudeli `M3C-L87-FRGN` puhul, `SN214701292-FRGB` mudeli RGB-kaamera puhul. Filtri abil ühendatakse seerianumber mudeliga `-`, mitte `_`, seega on nimel samad `_`-ga eraldatud väljad nagu varemgi (eesliide, kuupäev, kellaaeg, millisekundid, kaamera, tase). Kaamera, mille mudelit ei ole võimalik lugeda (kustutatud DeviceUserID), nimetatakse lihtsalt `SN<serial>`. Varasemate versioonide salvestustel puudub nimest filter; Chloros loeb mõlemat.
- **Eesliide:** `lattice capture` nimetab faile `capture_…`; `array-capture`, `multi-capture` ühendatud massiiviga ning `project capture` / `burst` / `stream`, millel on massiivi nimi, nimetab need `sync_<ts>_SN<serial>-<filter>…`.
- **Taseme sufiks ainult mitme taseme puhul.**Kaamera, mis salvestab**mitu**taset ühe salvestuse jooksul (`all` või `--levels a,b`), lisab `_<level>`. Kaamera, mis salvestab**ühe** taset, säilitab lihtsa nime ilma sufiksita — `--processing reflectance` kirjutab `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — nii et ühetasemeliste piltide jaoks kirjutatud skriptid jätkavad töötamist. Tase on endiselt määratud kausta järgi.
- **Klambermaskid järgivad toote nime:** `<product name>_clampmask.tif`. Ühetasandiline peegeldusvõte kirjutab `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; `all`-võte kirjutab `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Laiend** järgneb `--format`-ile: vaikimisi `tiff` annab tulemuseks `.tiff`, `--format tif` annab tulemuseks `.tif` (piirmaskid on alati `.tif`).
- **Üks erand:** `lattice capture --processing debayered` (või `--levels debayered`) võtab kiire ühe-kaadri pikkuse teekonna ja kirjutab `capture_<ts>_SN<serial>-<filter>.tiff` otse `-o`-i — alamkausta pole.
- **Vood ja pursed** kannavad sama välja: `lattice continuous` kirjutab `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` ja töötlemata pildiseeria `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` on salvestamise ajamärge ja `<serial>` kaamera seerianumber, seega jagab üks sünkroniseeritud rühm
ajamärget kõigi kaamerate vahel. **Pange tähele üht asümmeetriat:** `display`-tase salvestatakse kausta
nimega `preview/`, samas kui mitmetasandilised failid säilitavad `_display` nimes — kaust ja laiend erinevad
ainult selle taseme puhul. Tundmatud tasemed salvestatakse oma nime kandvasse kausta ning kui alamkaustu
ei ole võimalik luua, kirjutatakse fail väljundkataloogi, selle asemel et see kaoks.

**Salvestuste kausta uuesti töötlemine:**suuna `chloros-cli process`**captures-juurkataloogi**
(`output/`) poole. `process` impordib tavaliselt ainult sinu nimetatud kausta, kuid kui selles kausas pole
pilte, kuid sisaldab alamkaustu, liigub ta automaatselt allapoole — nii et kõik taseme alamkaustad ja
alamkaust „`daq/`“ võetakse korraga sisse. Iga „capture“ tase imporditakse ühe pildina, kusjuures
teised tasemed on kättesaadavad režiimidena, mitte ühe pildina taseme kohta.

**Taseme alamkausta** otsene nimetamine (nt `output/raw/`) toimib samuti. Sel juhul jääb `daq/`
maha, seega kopeerige või suunake DAQ-lugemine kõrvale, kui tetuletate radiomeetrilist
toodet failist `raw/` — vastasel juhul ei ole ajamärgise vastavusel millegi vastu lahendada.

**Töötlemine algab alati failist `raw`.** Iga salvestuse puhul on töötlemisjärjekorra allikaks töötlemata kaader;
`debayered`, `radiance`, `reflectance` ja `preview` on vaatamisrežiimid, kuid neid ei suunata kunagi
tagasi töötlemisahelasse. Tuletatud toote uuesti töötlemine rakendaks uuesti vinjeti, CCM-i ja
kiirguse matemaatikat, mis on juba selle pikslitesse sisse küpsetatud, seetõttu loobub Chloros sellest, selle asemel et
teostada topelt-töötlust. Kaks teadmisväärset tagajärge:

- Renderdusi `index/` ja `composite/` **ei töödeldagi kunagi** töödeldud. Need on väljundid, mitte salvestused —
  NDVI-LUT-renderil puudub mõtestatud kiirguse tõlgendus.
- Salvestuste kaust, mis on eksporditud **ilma** `raw`-ita (nt `array-capture --processing reflectance`), ei ole
  puudub õige töötlusvoo allikas. Need salvestised imporditakse ja kuvatakse normaalselt, kuid `process` jätab
  need vahele ja teatab sellest:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Kui teil on tõepoolest vaja tuletatud toodet läbi suruda — näiteks hub-seanss, mis on salvestatud
  `demosaic` sisse lülitatud, või vana kaust – sunnib `--input-level {raw,debayered,processed}` sisenemispunkti
  ja tühistab vahelejätmise. See lipuk on tahtlik pääsetee; `auto` (vaikimisi)
  ei töödelda kunagi salvestust, millel puudub toorandmed.

**Kalibreerimismärgised: ainult `raw`-is.** Salvestuse fail `raw` sisaldab kaamera kalibreerimispaketti: `CalibrationJSON`-andmeid ja MicaSense-ga ühilduvaid märgiseid (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), mis kirjeldavad anduri toorväärtusi. Iga teise taseme fail (`debayered`, `display`, `radiance`, `reflectance`, `index/` render) ning iga `lattice continuous` / `multi-stream` kaadri puhul sisaldab kaamera identiteeti (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, …) ning kaadri enda andmed, `XMP-mapir:FrameJSON` (valgustusaeg, võimendus, temperatuur, pikseliformaat, salvestusaeg, DAQ-vastavus), kuna iga `process` toode kannab seda identiteeti. `process` leiab sellise faili kalibreerimise selle seerianumbri järgi ja `CalBundleSha256` Chlorosi kalibreerimise vahemälust, laadides paketi alla üks kord, kui arvutis seda pole.

#### Mitmest kaamerast loodud failid

Fail, mis on loodud rohkem kui ühe kaamera lainealadest – mitme laineala pildistack (`align-apply`, `align-stream`, `index --live --save-multiband`), mitmest kaamerast koostatud reaalajas indeks (`index.tif`, `index_LUT.png`), masspildistuse `composite/` pilt — nimetab iga kaamera, millest see koosneb:

- **Kaamerate kaupa loetelud, ribade järjekorras** (komposiit: taustakiht, seejärel esiplaan; indeksikiht, kaamerad, millest selle ribad pärinevad): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibreerimispakett, mille kaudu iga kaamera lainealad läbisid, muidu — `align-apply` / `align-stream`-kogum, mille sagedusalad ei läbinud ühtegi — kaamera pakett, mille Chloros on vahemällu salvestanud, see, mille järgi on nimetatud selle salvestusfailid), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — paralleelsed loendid, millest igaühe *n*-nda kirje kirjeldab sama kaamerat; `-`, kui kaameral puudub väärtus (kalibreerimispakett puudub, objektiiv on teadmata). Iga kaamera on **üks** kirje: kui selle paketil on ühe välja jaoks mitu väärtust — näiteks paketi halogeenlambi sertifikaadid —, ühendatakse need `;` (`F-1479-1;F-1480-1;F-1481-1`) abil, mitte kunagi komaga, nii et nimekiri sisaldab ühte kirjet iga kaamera kohta, olenemata sellest, kuidas see trükitakse.
- **EXIF `SerialNumber`** sisaldab samu seerianumbreid, mis on ühendatud komadega (`214000535,214000551`).
- See **ei** sisalda ühegi kaamera identiteeti (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): need nimetavad ühte kaamerat ja selle komplekti ning `process` määraks nende põhjal kindlaks faili kalibreerimise. Mitme kaamera lainepikkustest arvutatud indeks (`index.tif`, `index_LUT.png`) sisaldab EXIF-andmetes `Model` `LATTICE` (seadmestik), mitte ühegi kaamera `LATT-…` mudel.**Ühe** kaamera lainealade fail – lainealapõhine TIFF-fail (`--per-band`), üks `align-apply --per-camera` kaadri, ühe kaamera pildistuste kogum või reaalajas indeks – sisaldab selle kaamera enda identiteedimärke, nagu iga teine pildistatud fail. `align-apply`-i pildiseeria, ribapõhised ja kaamerapõhised failid ning iga `align-stream`-i pildiseeria ja selle ribapõhised failid sisaldavad oma rühma salvestusaega (viitekaamera säriaeg, UTC), ning monokaamera `--per-camera` kaadri puhul on tegemist ühe tasandiga, nagu ka pildistamiskogumi puhul (`--bit-depth`).**Kaamera orientatsioon (peegeldamine / pööramine).** Kaamera orientatsiooni seade (kasutajaliidese peegeldus H /
Mirror V / pööramine) määrab, mida pildistamisel salvestatakse:

- **`raw` ja `debayered` salvestatakse täpselt nii, nagu andur need luges**, olenemata seadistusest.
  Pööratud või peegelpildis olev Bayer-mosaiik oleks teistsugune värvifiltri muster ning kalibreerimiskaardid
  (tumedusnihe, tasane väli) asuvad anduri enda võrgustikus.
- **`preview`/`display`, `radiance`, `reflectance` ja renderdatud fail `index/` on orienteeritud** — pööratud
  üks kord pärast kõiki kalibreerimisarvutusi, nii et need avanevad samamoodi, nagu kaamera oli kadreeritud. Nende
  `clamp_masks/…_clampmask.tif`-failid asuvad samal võrgustikul.
- **Iga pildifail salvestab seaded** XMP-vormingus: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, päripäeva, pärast
  pööramisi) ning `ChlorosOrientationApplied` — `false` toor- ja debayereeritud (natiivpikslid) piltidel,
  `true` orienteeritud tulemustel. `ChlorosProcessing` loetleb selle faili pikslitele rakendatud operatsioone
  (`none` algpikslite puhul). See hõlmab iga kaadrit, mida `lattice continuous` edastab (
  kaamera enda seaded, rakendamata) ja iga kaadrit `lattice multi-stream`-ist (identiteet: selle
  liikmekaadrid on algupärased, nagu massiivi pildistamisel). `lattice align-stream` / `align-apply`
  stakid salvestavad peakaamera seaded, mida ei ole rakendatud: nende lainepikad on moonutatud peakaamera
  kaadrisse, kuid neid ei ole pööratud.
- **`process` rakendab salvestatud orientatsiooni**, kui töötleb toorandmeid: esmalt tumeda nihke, tasanduse
  korrektsiooni, demosaikimise ja segamise eemaldamise andurikoordinaatides, seejärel massiivi joondamise moonutuse ning
  suund viimasena — nii et eelvaade, kiirgus, peegeldus ja indeksitulemused tulevad välja täpselt nagu
  reaalajas pildistamisel ning salvestatakse `ChlorosOrientationApplied=true`. Kalibreerimata varuvariandid
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) on pärast
  flat-välja kasutamist. Debayereeritud toode jääb algupäraseks.
- **Chloros-pildivaataja näitab RAW- või debayereeritud faili, mis on pööratud vastavalt selle salvestusele** (ainult kuvamiseks —
  faili sisu ei muutu), nii et see näeb välja nagu selle tooted; pisipilt, täissuuruses vaade,
  kursori näidud ja indeks/LUT-liivakast kasutavad kõik sama pööramist. Fail, millel puudub salvestus, ja
  toode (mis on juba orienteeritud) kuvatakse sellisena, nagu need on salvestatud. Sandboxi eksport salvestab orientatsiooni, mida selle
  piksleid kannavad (`ChlorosOrientationApplied=true`; identiteet kihile, millel puudub orientatsioon).
- **Salvestusi, millel puudub salvestusandmed, töödeldakse algkujul** (muutumatu käitumine). Enne versiooni 1.3.0
  pööras ühe kaameraga tehtud salvestus, millele oli määratud peegeldus või pööre, ise oma toor- ja debayered-faile;
  Chloros ei suuda sellist jubapööratud toorfaili eristada originaalfailist ega tee oletusi, seega
  töötleb seda salvestatud kujul. Kalibreerimiskaardid asuvad anduri enda koordinaatvõrgustikus, seega sellise
  pildistuse puhul kasutage 1.3.0 versiooniga tehtud pilti, kui kalibreerimine on oluline.
- **Massiivid ei pööra vaikimisi ühtegi liikmefaili** (joondamise moonutus ja `Alignment*` XMP asuvad
  kaamerate endi, pööramata kaadrites) ning salvestavad identiteedi. `array-capture --orient` pöörab iga
  liikme tulemusi pärast joondamise moonutust: vastavalt **peakaamera** orientatsioonile, kui need
  moonutati selle raami järgi (nii et liikmed jäävad ühildatuks), muul juhul vastavalt nende enda kaamera orientatsioonile. Iga
  fail salvestab selle kaadri orientatsiooni, milles asuvad selle enda pikslid – toorandmed, mis vastavad selle enda kaamera orientatsioonile –
  ning toorandmed, mis sisaldavad joondamismuutust, salvestavad ka `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: masteri orientatsioon, kui toode on paigutatud
  peakaadri järgi. `process` pöörab iga toodet vastavalt sellele, mis kuulub selle pikslitele:
  joondatud kaadri järgi, kui see moonutab toorkaadrit (`--array-alignment`, vaikimisi — reaalajas joondatud
  pildistamise tulemused), kaamera enda, kui seda ei tehta (`--no-array-alignment` – reaalajas
  joondamata pildistamise tooted). Sama valik on olemas ka `array-burst`, `project capture` / `burst` /
  `stream`, retseptide ja SDK puhul.

### Vahelejäänud pildistamised segafiltreid kasutavates massiivides

Kui ühendate ühte massiivi RGB- ja multispektraalkaamerad, salvestab `array-capture --processing radiance` (või `reflectance`) multispektraalkaadrid ja **jätab vahele** RGB-kaamerad — Bayeri-põhine kiirgus ei ole lairibaanduri puhul oluline. CLI kuvab iga salvestatud faili (koos eksporditasemega), iga kirjutatud `.daq`-i ja iga vahelejätmise selgesõnaliselt, seega ei ole failide arv üllatav:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Vahelejätmise põhjuse märgid järgivad mustrit `<level>-not-applicable-to-rgb-cam`. Kaamera, mille kalibreerimispakett ei ole laaditud, jätab vahele `radiance` / `reflectance` koos `<level>-needs-calibration (connect the camera so its cal pack loads)`, ning kui DAQ-i peegeldusnäit ei ole kasutatav, vaheletakse koodidega `reflectance-skipped-no-dls`. Peegeldusnäit võib vahele jääda ka koodidega `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, ning koos `dls-uncalibrated-band-<nm>`-iga, kui lainepikkus jääb peamiselt väljapoole DAQ-valgusanduri radiomeetriliselt kalibreeritud vahemikku (~374–974 nm) — saadaval olevatest tootenumbritest ainult F988, mille toetatud töövoog on peegeldusvõime-paneeli töövoog.

Kasutage `--processing debayered` (või `display`), et hõlmata kõiki kaameraid sõltumata filtri tüübist, või vaikimisi seadistust `all`, et saada ühe korraga kõik asjakohased tasemed iga kaamera kohta.

### DAQ-näidustus, mis erineb oluliselt ekspositsioonist

Iga salvestatud `.daq` on selle ekspositsiooniga sobitatud DAQ-näidustus ning selle kõrval olev `.match.json` näitab, kui suur oli nende kahe vahe (`gap_ms`) ja kas see jääb lubatud hälbe piiresse (`in_tolerance`, 500 ms, kui pildistamisseadetes pole teisiti määratud `--daq-tolerance-ms`). Ka lubatud piiridest väljaspool olevat näitu kasutatakse ikkagi, seega ei lähe ükski kaader kaduma, ning see märgistatakse: peegeldusfaili XMP-andmetes on kirjas `DlsInTolerance=False` ja `DlsMatchGapS`. `lattice capture`, `multi-capture` ja `array-capture` kuvavad seda ka asjaomase `.daq` all, koos DAQ'voo veaga, kui seda selgitada:

**Kuidas mõõtmistulemus sobitatakse ekspositsiooniga.** Salvestus ootab DAQ-i esimest mõõtmistulemust pärast ekspositsiooni, et saaks ühendada mõõtmistulemused mõlemal pool seda (`method` `interpolated`). DAQ-E saadab näidu päevavalguses iga 0,4 sekundi järel ja siseruumis iga 1–2 sekundi järel, seega lisandub ooteaeg umbes poolteist sellist intervalli käsklusele, mitte valgustamisele. DAQ-i, mis on saatmise lõpetanud, ei oodata. DAQ-E näidud on valgustuse keskmine väärtus alates eelmisest näidust, seega paigutatakse need selle intervalli keskele: `gap_ms` mõõdetakse säriajast kuni selle keskpunktini ning `bracket_stamps_ns` annab näidude enda ajamärgid. `array-capture --continuous` ootab ilma pildistamist aeglustamata: selle toorfailid kirjutatakse kohe ja nende `.daq` lisatakse, kui lugemine saabub. `--interval` puhul ootab iga läbiviimine nagu üksik pildistamine.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Kui sellest lugemisest ei kirjutatud ühtegi peegeldusfaili, lõpeb esimene rida selle asemel `saved anyway; its .match.json says in_tolerance: false`-ga.

Salvestus lõpeb ikkagi `0`-ga: kõik failid on kirjutatud ja neis on märgitud, mida need sisaldavad. Andmekogumisseade, mille voog on seiskunud, edastab jätkuvalt oma viimast lugemit, seega kontrollige seda `chloros-cli daq pool-list` abil, enne kui tuginete selle salvestuse peegeldusandmetele. Tagapõhi ühendab uuesti ühendatud DAQ-E-seadme, mis on andmete edastamise lõpetanud (vt [Andur, mis lõpetab andmete edastamise](#a-daq-e-that-stops-delivering)). Lubatud piiridest väljas olev näid trükib välja ainult oma `DAQ:` rea, välja arvatud juhul, kui tegemist on säilitatud näiduga (allpool).

**Mõõtmistulemus, mis säilitatakse pärast DAQ-i kadumist.** Kui ükski DAQ ei suuda ekspositsiooni jaoks mõõtmistulemust edastada (seotud DAQ ei ole kättesaadav või ühelgi koondatud DAQ-il pole seda, nt pärast `daq pool-disconnect`), kasutab salvestus uuesti viimast näitu, mis Chlorosil oli, ja selle `.match.json` kirjeldab seda näitu seoses selle ekspositsiooniga: `method`, `retained`, `gap_ms` sellest ekspositsioonist proovidele, millest näidud võeti, `in_tolerance` sellest vahemikust ning `retained_from_epoch_ns`, ekspositsioon, millega näidud esmakordselt sobitati. Seega trükib pika aja möödudes pärast DAQ-i kadumist tehtud salvestus eespool nimetatud read ning nende alla neist veel üks, mis trükitakse välja isegi siis, kui mõõtmistulemus on veel lubatud piirides:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Selle `.daq` nimetab endiselt seda DAQ-seadet, mis lugemise tegi, koos kalibreerimise ja kaanega, mille all lugemine tehti, nii et uuesti importimisel on teada, milline DAQ, kalibreerimine ja kaas selle tekitasid.

### Failid, mida ei õnnestu salvestada

Kaamera, mille faili ei ole võimalik salvestada – täis kõvaketas, õiguste viga –, ei takista teiste kaamerate failide salvestamist. Iga salvestuskäsk (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; ning retsepti `capture`, `burst` ja `stream`) väljastab iga ebaõnnestunud faili stderr-i, seejärel **lõpetab töö `1`**, kuigi ülejäänud salvestati:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Kõik salvestuse juurde kuuluvad failid on hõlmatud, sealhulgas ükskaadri debayeringuga töödeldud `lattice capture --levels debayered` kaasa arvatud. `.daq` lugemine, mille `.match.json` päritolu sidecar-faili ei olnud võimalik kirjutada, on loetletud kui `[match]` (`.daq` ise salvestati).

Täis kõvakettal kuvatakse korduvad salvestused (`array-capture --continuous` / `--interval`, `project burst` / `stream`, retsept `burst` / `stream`) **peatub selles läbiviimises**, selle asemel et jooksu ülejäänud osa jooksul samale kettale midagi kirjutamata jätta. Iga muu viga teatatakse igas läbiviimises, kus see esineb, ning jooks jätkub lõpuni, seejärel lõpetatakse `1`. `array-capture --continuous` (tagapõhja pidev salvestamine) ei loetle ühtegi faili, kaasa arvatud ebaõnnestunud failid: need loetakse üles käigu ja kokkuvõtte ridadel (`N failed`, `N file(s) failed to write.`) ning lõpetab töö `1` — vt [Pidev režiim](#continuous-mode---continuous).

Salvestajad — `array-burst` ja `array-record` — teatavad oma vigadest. `array-burst` peatub ise, kui draiv täitub või vahetult enne seda. Mõlemad jätkavad tööd ka muude ebaõnnestunud kirjutamiskatseid hoolimata. Kui kumbki neist teatab veast, kuvatakse see loendurite järel ning käsk **`1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` asemel `Disk full:` tähendab, et salvestamine peatati enne, kui kõvaketas täis ei saanud. Enne peatamist salvestatud kaadrid säilitatakse. `array-burst --build` jätab koostamise vahele, kui ruum otsa sai, sest videod salvestataks samale kõvakettale. Vabastage kõigepealt ruumi ja käivitage seejärel `array-build-video --burst-dir …`. Mis tahes muu vea korral jätkub koostamine ikkagi.

`array-record`, mis ei kirjutanud ühtegi kaadrit, on samuti ebaõnnestunud salvestus (`Recorder error: no-frames: the combined view published no frame to record …`, välju `1`) — vaata [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Salvestusrežiimid, salvestajad ja offline-taas töötlemine

Need kõik töötavad **püsiva massiiviga** (käivita esmalt `array-connect`). Need peegeldavad graafilise kasutajaliidese salvestuspaneeli.

### `array-capture` režiimid

`array-capture` on üks käsk, millel on neli pildistamisrežiimi ja rida ekspordivalikuid:

| Režiim | Lipuk | Käitumine |
| --- | --- | --- |
| **Ühekordne** *(vaikimisi)* | (puudub) | Üks sünkroniseeritud salvestusgrupp, seejärel lõpetamine. |
| **Järjepidev** | `--continuous` | Järjestikused salvestused kuni `Ctrl+C`, `--count N` salvestusteni, `--duration S` või kogu ketta täitumiseni. Käivitab tagapõhja enda pideva salvestamise (graafilise kasutajaliidese pidev režiim) — vaata allpool [Pidev režiim](#continuous-mode---continuous). |
| **Intervall** | `--interval S` | Üks läbimine iga `S` sekundi järel (mõõdetuna iga läbimise algusest), samad piirid. |
| **Kiireim** | `--fastest` | Ainult toores-ainult + määratud DAQ-näit + kombineeritud indeksist koosnev komposiit; jätab kiirguse/peegelduse/kuvamise arvutused vahele, et kaader jõuaks kiiresti ekraanile. Eeldab `--processing raw --force-daq`. Töötle salvestatud `.daq` hiljem ümber kalibreeritud toodeteks. |

Ekspordilülitid (kombineeritavad mis tahes režiimiga; kõik kasutavad sama GUI/SDK-lõpppunkti):

| Lipuk | Mõju |
| --- | --- |
| `--array-id ID` | Massiiv, millest andmeid koguda (jätke välja, kui ühendatud on ainult üks). |
| `-o, --output DIR` | Väljundkaust (vaikimisi `output`, selles kausis, kust käsk käivitatakse). |
| `--format {tiff,tif,png,jpg,jpeg}` | Failivorming (vaikimisi `tiff`). PNG / JPEG võtavad vastu ainult `display` ja indeksrenderduse; vaata [Salvestamise eksporditasemed](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Üks eksporditase (`raw`, `debayered`, `radiance`, `reflectance`, `display`) või `all` (vaikimisi). |
| `--levels L1,L2,…` | Eksporditüüpide selgesõnaline alamhulk (nt `raw,radiance,reflectance`); **asendab `--processing`**. |
| `--aligned` / `--no-aligned` | **Vaikimisi: joondatud, nagu Chloros-rakenduse sama massiivi salvestus**(selle massiivi kohta kehtiv**Joondatud** salvestusseade, mis on sisse lülitatud, kui seda välja ei lülita). Iga liikme mitte-toores eksport (`debayered`, `display`, `radiance`, `reflectance`, indeksrenderdus) moonutatakse põhikaamera kaadrisse ja kärbitakse liikmete ühisele kattuvale alale, nii et failid on pikselite kaupa registreeritud ning nende suurus, kärpimine ja `Alignment*`-märgised vastavad rakenduse omadele — väiksemad kui anduri (või huvipiirkonnast) liikmete nihkete võrra; huvipiirkonna puhul viiakse teisendus akna suhtes uuesti baasjoonele. `XMP-mapir:AlignmentJSON` viitab `"applied": true`-ile ja annab tulemuseks `crop_roi`. Raw-fail jääb moonutamata algses suuruses, kuid kannab teisendust metaandmetes (`"applied": false`). `--no-aligned` salvestab iga faili algses suuruses, moonutamata. `--aligned` on vaikimisi määratud. Sõltumata sellest, kas moonutust rakendatakse või mitte, salvestab iga liikmefail oma kaamera `Alignment*`-transformeerimise, kui massiivil on joondamisprofiil — välja arvatud kaamera, mida profiil ei joondata (jätetakse välja või sunnitakse ebaõnnestunud lahenduse tõttu identiteediks), ning viitekaamera, kui ühtegi teist kaamerat ei ole joondatud: need ei sisalda `Alignment*`-märgiseid. Kui massiivil puudub profiil, salvestatakse pildistamine joondamata ning CLI väljastab põhjuse `Alignment:`-real, nagu see tehakse ka kõigi teiste juhtumite puhul, kus moonutust ei saanud rakendada (liikmed erineva resolutsiooniga, massiiv, mis ei ole kaabli abil sünkroniseeritud, kaader, millele profiili ei saa kaardistada). Massiivi automaatne joondamine loob profiili, kui selle ekspositsioon on stabiliseerunud; `align-calibrate` loob selle nõudmise korral. |
| `--orient` | Pöörake iga elemendi **tooteid**— `display`, `radiance`, `reflectance`, render `index/` ja `composite/` — vastavalt kaamera orientatsioonile pärast joondamisdeformatsiooni: toode, mis on deformeeritud**põhiraami**orientatsiooni järgi, nii et liikmed jäävad ühildatuks; joondamata pilt (`--no-aligned` või pilt, millele moonutust ei saanud rakendada) oma kaamera orientatsiooni järgi.**Vaikimisi: välja lülitatud — iga liikmefail jääb algupäraseks**, nagu massiivi pildistamiste puhul alati. `raw` ja `debayered` jäävad mõlemal juhul algkujule. Iga fail salvestab oma toodete orientatsiooni (vt [Kaamera orientatsioon](#kuidas-salvestuskausta-välimus-on)); `Alignment*` XMP-fail jääb algse raami teisenduseks, mida rakendatakse enne orientatsiooni, seega `process` (moonutamine, seejärel orientatsioon) taasesitab reaalajas toodet. |
| `--index` / `--no-index` | Salvestada / vahele jätta kaamera-põhine taimestikuindeksi kiht, kui see on konfigureeritud. Vaikimisi: renderdada see. |
| `--no-daq` | Jäta vahele `.daq` sidecar-fail, kui ükski valitud tase ei kasuta DAQ-i (ainult toorandmete salvestus). Vaikimisi salvestatakse näidud, et kaadreid saaks offline-režiimis ümbertöötada peegeldusvõime/indeksiks. `--force-daq` on endiselt lubatud (vaikimisi). |
| `--daq-tolerance-ms MS` | DAQ-i sobitustolerants nendele salvestustele (vaikimisi 500 ms). |
| `--smart` | Oota, kuni AE on kõigil kaameratel stabiliseerunud, enne kui käivitad (vt [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-piksli tihendus. `none` (vaikimisi alates versioonist 1.3.0; versioonis 1.2.0 kirjutati `deflate`) = tihendamata — anduri pikslid täpselt sellisena, nagu need loetakse, ~6,3 MB täisresolutsiooniga kaadri kohta ja kiireim püsikiirus; `deflate` = kadudeta zlib L1 + horisontaalne ennustaja, ~4,1 MB kaadri kohta, aeglasem. Mõlemad on kadudeta ja loetakse importimisel identselt. |

> **Ühekordse kirjutamisega TIFF + püsikiiruse mudel.**Iga fail kirjutatakse**üks kord**: pikslid, XMP-pakett, IFD0 Make/Model/CameraSerialNumber ja EXIF-märgised `ExposureTime` / `SerialNumber` kirjutatakse ühe läbimisega, ilma et midagi seda hiljem ümber kirjutaks. `--continuous` salvestab järjestikuseid sünkroniseeritud gruppe – iga salvestus võtab eelmise grupi järel järgmise, sealhulgas ühe, mis avaldati veel selle salvestamise ajal –, seega töötab see massiivi kiirusel. DEFLATE hoiab kinni Pythoni GIL-i, seega kompresseeritud kirjutamised**ei**toimu paralleelselt kaamera-põhiste kirjutajate niitide vahel; pidev täisresolutsiooniga salvestamine anduri kiirusel nõuab tihendamata kirjutamist (vaikimisi)**ja** NVMe-klassi kõvaketast (~600 MB/s püsikiirus 9-kaameralise massiivi puhul kiirusel ~10 kaadrit sekundis). Sama seadistus on kättesaadav ka `compression`-na `POST /api/camera/array/capture`-is.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Pidev režiim (`--continuous`)

`array-capture --continuous` (ilma `--interval` või `--smart`-ita) annab juhtimise üle **tagapõhja enda pidevale salvestamisele** — sessioonile, milles töötab GUI pidev režiim. Backend salvestab iga sünkroniseeritud rühma ise, tehes seda kahe salvestuse jooksul ja ilma CLI-päringuteta rühmade vahel, mistõttu suudab ta pidada sammu kiire massiiviga, kus üks päring salvestuse kohta jääb maha (mõõdetud kiirus 73–77 salvestust sekundis 9-kaameralisel massiivil 512-pikselise huvipiirkonna puhul). Iga eespool nimetatud ekspordilüliti kehtib iga salvestuse kohta.

- **Piirangud:** `--count N` peatub pärast N salvestust (üks salvestus = üks sünkroniseeritud rühm: iga liikme failid); `--duration S` peatub pärast S sekundit. Kui kumbki neist ei ole määratud, töötab programm kuni `Ctrl+C`, kõvaketta täitumiseni, vea tekkimiseni või kaamerasüsteemi kadumiseni. `Ctrl+C` peatab tagapõhja seansi, seejärel kuvab kokkuvõtte.
- **Edusammud** iga 2 sekundi järel, millele lisatakse `, N failed` ja `, N groups skipped`, kui väärtus on nullist erinev (vahelejäänud rühm on selline, mille massiiv avaldas, kuid mille kohta ühtegi salvestust ei tehtud):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Failide kaupa loetelu puudub.** Töötamine lõpeb ühe kokkuvõtva reaga — salvestused, kiirus, failid ja väljundkaust — millele järgnevad `N file(s) failed to write.` / `N group(s) skipped.`, kui väärtus on nullist erinev:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Peatub** täis kettaga koos `Continuous capture stopped: the drive is full.` ning vea korral või massiivi kadumisel koos `Continuous capture stopped: <reason>.` (mõlemad stderr-is).
- **Lõpetamisstaatus:** `0`, kui salvestati pildistused ja kõik failid kirjutati, sealhulgas `Ctrl+C`; `1`, kui mõne faili kirjutamine ebaõnnestus, kui kõvaketas on täis, tekib viga, massiiv kaob või salvestust üldse ei ole.
- **Microsoft Defender:** kui Defender kasutab ühte või enamat CPU-tuuma salvestuse poolt kirjutatud failide skaneerimiseks, kuvatakse käivitamisel üks kord stderr-is: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Vaata installija [süsteemiseadete](#windows-exe) kohta.
- Ühe massiivi kohta on korraga lubatud üks pidev salvestus; teine lükatakse tagasi veaga `a continuous capture is already running for this array`. HTTP-i kaudu on sessioon `POST /api/camera/array/continuous/start` (`/api/camera/array/capture`-i sisu pluss `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` ja `POST /api/camera/array/continuous/stop`.

**Intervall**(`--interval S`, koos `--continuous`-iga või ilma) ja**`--continuous --smart`** säilitavad päringu-põhise tsükli: üks salvestuspäring iga läbiviimise kohta — Intervalli puhul igal planeeritud hetkel, pärast AE stabiliseerumist `--smart` puhul — iga läbiviimise failid on loetletud nagu ühekaupa režiimis, lõpus on rida `Done — N capture pass(es).` ja `--count N` loendab läbiviimisi.

### `array-record` — kombineeritud indeksiga video/GIF (jälgimiskvaliteediga)

Salvestab massiivi **reaalajas kombineeritud indeksivaate** faili `.avi` (ja valikuliselt faili `.gif`). Kui graafilise kasutajaliidese kombineeritud vaade on avatud, salvestab see vaate sellisena, nagu see kuvatakse. Kui ükski vaade pole avatud, renderdab tagapõhi vaate ise: indeks, mille edastate koos `--expr`-iga, või vaikimisi massiivi NDVI, kui massiiv on joondatud. Vaikimisi NDVI kasutab punast ja NIR-ribasid, mis asuvad kõige lähemal 660 nm ja 850 nm, nagu seda teeb ka reaalaja vaade. Salvestamise ajal avatud GUI-vaade salvestatakse seni, kuni see on avatud. See kontrollib edusamme iga 2 sekundi järel ja peatub `--duration`, `Ctrl+C` või kui salvestaja lõpetab töö ise.

Salvestus, mis ei kirjuta ühtegi kaadrit, ebaõnnestub: salvestaja teatab veakoodist `no-frames: …` (massiiv võib veel joonduda või valgustust reguleerida) ja käsk **lõpetab töö `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--array-id ID` | ainult maatriks | Sihtmaatriks (jätke välja, kui on ühendatud ainult üks). |
| `-o, --output DIR` | `output` | Väljundkataloog. Suhteline tee asub selle kausta all, kust käsk käivitatakse. |
| `--fps F` | `10` | Salvestamise kaadrisagedus. |
| `--duration S` | kuni Ctrl+C | Automaatne peatamine pärast `S` sekundit. |
| `--gif` | off | Kirjuta ka animeeritud GIF-fail. |
| `--gif-only` | välja lülitatud | Salvestada ainult GIF-fail (ilma `.avi`-ita). |
| `--expr EXPR` | massiivi NDVI | Indeksi väljend, mida salvestada, kui ühtegi GUI-vaadet pole avatud, massiivi ribade nimedes (nt `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; vt [ribalaiused](#index--vegetation-maths)). Lühike värvinimi (`NIR`, `Red`) toimib ainult siis, kui massiivi ühel ribal on see olemas. Avatud GUI-vaade salvestatakse selle asemel nii, nagu see kuvatakse. |

### `array-burst` — toores Bayer-formaadis kõrge kaadrisagedusega sarivõte (analüüsikvaliteediga)

Loeb otse kaadrivõtu tsükli sünkroniseeritud rühma puhvrit — **kalibreerimisahelat, exiftool'i ega reaalaja vaadet ei ole vaja** — seega töötab see kaamera täieliku pildivõtusagedusega. Kirjutab RAW-kaadrid + iga kaadri manifesti + ühe `.daq` iga eraldi DLS-lugemise kohta `<output>/bursts/<base>/` all. Töötle uuesti offline-režiimis (järgmine käsk) või edasta `--build`, et teha seda kohe peale peatamist.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--array-id ID` | ainult massiiv | Sihtmassiiv. |
| `-o, --output DIR` | `output` | Väljundkataloog (andmed salvestatakse kataloogi `<DIR>/bursts/<base>/`). |
| `--duration S` | kuni Ctrl+C | Automaatne peatamine pärast `S` sekundit. |
| `--max-frames N` | piiramatu | Automaatnepeatamine pärast `N` toorkaadrit. |
| `--orient` | välja lülitatud | Selle andmepaketi põhjal koostatud tulemused pööratakse vastavalt kaamera orientatsioonile — iga kaamera tulemused vastavalt oma kaamera orientatsioonile, koondindeks vastavalt peakaamera omale — nagu teeb `array-capture --orient`. Toorkaadrid jäävad algkujul; seeria `manifest.json` salvestab iga liikme orientatsiooni ja valiku. |
| `--build` | välja lülitatud | Peatamise järel töödeldakse seeria kohe uuesti (sama mis `array-build-video`). |
| `--products …` | `combined:index` | Koos `--build`-iga: millist(eid) videot(eid) luua (vt allpool). |
| `--fps F` | `10` | Koos `--build`-iga: väljundvideo kaadrisagedus (fps). |
| `--save-tiffs` | välja lülitatud | Koos `--build`-ga: salvestatakse ka iga kaadri kohta kalibreeritud TIFF-failid (igaühel oma [unmix-klambri mask](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on), kui piksel on piiratud; koostamistulemus loetleb neid kui `clamp_masks`). |
| `--gif` | välja lülitatud | Koos `--build`-iga: kirjutab ka animeeritud GIF-faile. |

### `array-build-video` — töötleb salvestatud pildiseeria offline-režiimis uuesti

Sünkroniseerib iga töötlemata kaadri ajaliselt lähima salvestatud `.daq`-näiduga ja suunab selle läbi **sama kiirguse / peegeldusvõime / indeksi ahela kui importimise töötlusprotsess**, renderdades ühe või mitu videot.

`--products` on komadega eraldatud nimekiri `kind:level` elementidest, kus `kind` ∈ `per_cam` | `combined` ja `level` ∈ `radiance` | `reflectance` | `index`. Kui `level` on esitatud eraldi (ilma `kind:`-ita), on selle vaikimisi väärtuseks `per_cam`. Vaikimisi väärtus on `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--burst-dir DIR` | (nõutav) | Pildiseeria kausta asukoht (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level` nimekiri, nagu eespool. |
| `--fps F` | `10` | Väljundvideo kaadrisagedus (fps). |
| `--save-tiffs` | off | Salvesta video(de) kõrvale ka iga kaadri jaoks kalibreeritud TIFF-failid. |
| `--gif` | off | Kirjuta ka animeeritud GIF-failid. |

**Videol puudub klambrimask** (üks pildivoog, 8-bitine). Selle asemel sisaldab koostamistulemus `clamp_fractions`: iga videotoodangu kohta iga kaadri pikslite osa, mille unmix on klambitud selle R / G / B kanali ribasse — `Chloros:Band{R,G,B}UnmixClampFraction`, millega selle kaadri kiirguse/peegelduse tulemusele pandaks märge (kombineeritud indeksi jaoks, iga osaleva kaamera puhul). Käivitus väljastab iga toote halvima kaadri reale `Clamp:`. `--save-tiffs` lisab TIFF-failide pikselipõhised maskid, `clamp_masks/<level>/`-is TIFF-failide `<level>/`-kausta kõrval.**Ennetähtaegselt katkestatud koostamine ei jäta maha katkenud videot.** Iga video salvestatakse nimega `<name>.partial.avi` ja saab lõpliku nime alles siis, kui see on valmis, seega jääb sama nimega vanem video alles, kuni selle asendaja on terviklik. Käivita koostamine uuesti, et asendada maha jäänud `.partial.avi`.

Käivitamine väljastab iga salvestatud video, seejärel kõik `Skipped:`-tüüpi tulemused (nõutud kaamera/tase – või kombineeritud indeks –, mille puhul videot ei salvestatud, koos põhjendusega – nt kalibreerimispaketi puudumine, puudus DLS-lugemine peegeldusvõime jaoks või puudus punase/NIR-sagedusriba paar kombineeritud indeksile) ning kõik `Note:`-failid. RGB-kaamera, millelt nõuti kiirgust või peegeldusvõimet, saab selle asemel ekraanivideo, mille valgebalanss on korrigeeritud, kui kaamera kalibreerimispakett on saadaval; kaamera, mis renderdati ilma selleta, nimetatakse `Note:`-is. See **lõpetab töötamise nullist erineva väärtusega**, kui taotletud toode vahelejäeti või ei koostatud vastavalt taotlusele (konfigureeritud indeksaväljendit ei hinnata offline-režiimis); koostatud videod kirjutatakse siiski välja ja loetletakse.

> **Vali õige salvestaja.** `array-record` on *jälgimiskvaliteediga* — see salvestab renderdatud reaalajas komposiiti: kasutajaliidese kombineeritud vaadet või, kui kasutajaliides pole avatud, tagapõhja enda NDVI-vaadet (või `--expr`). `array-burst` → `array-build-video` on *analüüsitasemel* — see salvestab töötlemata andurandmeid täiskiirusel ja rekonstrueerib hiljem kalibreeritud kiirguse/peegelduse/indeksi videod, ilma et oleks vaja reaalajas vaadet.

### Mono (M3M) ühekanalilised kaamerad

**M3M**-seeria on Bayeri**M3C**mono-versioon: üks kitsasribaline interferentsfilter kaamera kohta (`M3M-<lens>-F<wavelength>`, nt `M3M-L87-F685`), seega annab sensor**ühe halltoonide riba** ilma Bayeri mosaiigita. Ei ole vaja teha demosaiki, ei ole vaja eraldada kanalitevahelist ülekandumist ega seadistada valgebalanssi – kogu RGB-kuva värviprotsess lihtsalt ei kehti.

Mida see tähendab CLI-s:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**tuvastavad monokamera ja**jätavad selle üherealise sõnumiga vahele**, selle asemel et edastada mõttetuid seadeid. Samal seansil töötavad need endiselt normaalselt RGB- või Bayer-tüüpi M3C-kaameraga.
- **Bayer-pikseliformaat tähendab selle sügavust.** M3M pakub ainult monokroomformaate, seega seab Bayer-formaat käskudest `--pixel-format` või `--preset` sama sügavusega monokroomformaadi: `BayerRG8` ja `--preset high_speed` annavad tulemuseks `Mono8`; `BayerRG12`, `high_quality` ja `triggered` annavad tulemuseks `Mono12`; pakitud nimi jääb pakitud (`BayerRG12p` annab `Mono12p`). Failid ja `lattice format` nimetavad Mono-vormingut. M3C- ja M3M-kaameraid segav massiiv võtab ühe vormingu samamoodi: Bayer iga M3C puhul, Mono sügavus iga M3M puhul. Nimi on selle bittide sügavus: sama kehtib ka vastupidi, seega ei paigutata M3C-d kunagi Mono-formaati (`--pixel-format Mono12` on M3C-l `BayerRG12`, sõltumata sellest, kumb kaamera on massiivi juhtkaamera), ja `lattice format --set` kaardistab samamoodi ning väljastab selle formaadi, mille ta on seadistanud.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** töötavad endiselt — kiirgus ja peegeldus on *ribalised* radiomeetrilised kaardid ning on ühe riba puhul täiesti hästi määratletud. Monokroomkaadrid sisaldavad **identiteet**-sensori vastusmaatriksit (ilma 3×3 segamiseta), seega läbib pilt kalibreerimisarvutused muutmata kujul.
- **Üksainus monokroomkaamera ei suuda luua taimestikuindeksit.**NDVI, NDRE jms vajavad vähemalt kahte sagedusriba (nt Red + NIR). Et saada indeksit monokroomse riistvara abil, suunake**mitu** M3M-kaamerat erinevatele lainepikkustele, joondage need üheks mitmesageduslikuks pildistackiks ja arvutage indeks *sellest*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` avab kaamerad ise (a [ajutine seanss](#multi-camera-transient-sessions)), seega ärge esialgu midagi ühendage: `--serials`, muidu profiili aktiveeritud kaamerad (vt [Indeks / Taimestiku matemaatika](#index--vegetation-maths)). Kui Chloros hoiab mõnda kaamerat (GUI, `cam-connect`, `array-connect`) keeldutakse sellest HTTP-veaga 409 `CAMERA_HELD` ja programm lõpetatakse `1`; vabastage need `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all` abil.

`--channel` sümbolid peavad **täpselt** vastama eelseadistuse kanalite nimedele (suurtähti arvestatakse; NDVI-d on väiketähed `red`, `nir` — vt `--list-presets`), ning sagedusriba nimi peab vastama sagedusribale joondatud pildistikus (offline-režiimis aktsepteeritakse ka 0-põhiseid sagedusriba indekseid, nt `--channel red=0 --channel nir=1`).

Kogu pinu eristajaks on mudelistringis olev märge `M3M` (see ei esine kunagi stringis `M3C`), mis kuvatakse kasutajaliideses (GUI) ja SDK-s kui `is_mono`.

---

## Host-võrgukaardi seadistamine ja häälestamine (LATTICE-kaamerarühmad)

LATTICE-kaamerad edastavad GVSP-d hosti Ethernet-adapteri kaudu, seega on mitme kaameraga rühmade puhul adapteri **draiver**ja**vastuvõturingi suurus** on sama olulised kui ühenduse kiirus. Valed seaded ilmnevad `FRAMES WILL DROP` / `Reduce ROI to enable`-veana massiivi seadete paneelis (ning `lattice network-analysis` / SDK-s `analyze_array_network()`), isegi kui kaamerad ise on töökorras.

### USB 10GbE-adapterid — Realtek RTL8157 („Realtek USB 10GbE Family Controller”)

Allpool toodud seaded on Windowsi draiveri omadused. Linuxis kehtib USB-adapterile samuti piirang (400 MB/s x86_64-l, 200 MB/s NVIDIA Jetsonil) (vt [Array fps & burst model](#array-fps--burst-model)), ning iga adapteri puhul on olulised tuuma pistikupesa vastuvõtupuhvrid — vt [Linuxi hostid](#linux-hosts--socket-receive-buffers).

| Kirje | Nõutav väärtus | Miks see on oluline |
| --- | --- | --- |
| **Draiveri versioon**|**≥ v10.67 (jaanuar 2026)**, INF `rtump64x64sta.inf` | Vana **2016. aasta** draiver (v10.65, `rtump64x64.inf`) ei suuda korralikult käsitleda väljalülitamist ja tekitab**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**vigu süsteemi sulgemisel, taaskäivitamisel või puhkeolekusse minemisel. Üleminek jääb kinni (~5-minutiline ooteaeg), kasutaja lülitab seadme jõuga välja ning korduvad ebaõnnestunud väljalülitused**rikuvad WMI-andmebaasi**(PowerShell/tööriistad hakkavad `Invalid class`-veaga tõrkuma) ja**blokeerivad USB-pinu** järgmisel käivitamisel (võrgukaart ei aktiveeru; USB-seadmed ei loenda enam seadmeid). Lae alla värskendus realtek.com-ist (või dongli tootjalt), enne kui loodad puhtale taaskäivitamisele. |
| **Vastuvõtupuhvrid**— märksõna `ReceiveBufferLen` |**256**(draiveri maksimaalne väärtus) | Võrgukaardi vastuvõturõngas. Draiveri vaikimisi väärtus**32**jätab kasutatavaks vaid ~0,26 MB rõngast — mis on mitme kaamera puhul liiga väike — seega kuvab massiivi paneel veakoodi `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` ja blokeerib ühenduse. Väärtusel**256**on ring suur (**~13,5 MB, mõõdetuna laboris 10GbE-hostil**), andes RX-torustikule reaalset varu mitme kaameraga GVSP-pursete jaoks. (Kas antud konfiguratsioon tegelikult *ühendub*, otsustavad kaks kontrolli — **drain-aware**lubatavuse kontroll ja**koondatud ülemärkimise** kontroll – mitte toores võrdlus purske ja ringi vahel; vt [Massiivi kaadrisagedus ja purskemudel](#array-fps--burst-model).) |
| **Vastuvõtu-URB-d**— märksõna `PendingReceives` |**64** (maks) | Lendavad USB-päringublokid; suurendage koos vastuvõtupuhvritega pursete neeldamiseks. |
| **Jumbo-raam** — märksõna `*JumboPacket` | **9014** | Vajalik 9000-baidiste GVSP-pakettide jaoks (6× vähem pakette raami kohta kui 1500-baidiste puhul). |

> ⚠️ **Võrgukaardi draiveri uuendamine TAASTAB need täpsemad omadused vaikimisi seadetele.**Pärast adapteri draiveri uuendamist või asendamist**rakendage uuesti** `ReceiveBufferLen=256` ja `PendingReceives=64`, vastasel juhul lülitub massiivi paneel uuesti välja, kuigi „riistvaras pole midagi muutunud“. See on peamine põhjus, miks varem töötanud seadmestik äkki ühendust keeldub looma.

Kohaldage neid **kõrgendatud õigustega** PowerShellist (asendage oma adapteri nimi, nt `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` hõlmab USB 10GbE-adaptereid.** Nüüd tuvastatakse adapteri tüüp ja seadistatakse õige „receive-ring“ märksõna: `*ReceiveBuffers`→2048 PCIe-võrgukaartide puhul (Intel I219, jne) või `ReceiveBufferLen`→256 + `PendingReceives`→64 Realteki **USB** 10GbE-kontrollerile (mis ei avalda `*ReceiveBuffers`). Sihtväärtused piiratakse iga draiveri poolt teatatud maksimumiga (`NumericParameterMaxValue`), nii et see ei kirjuta kunagi vahemikust väljapoole jäävat väärtust. Käivita see **õigustega** terminalis; nagu iga registripõhise seadistuse puhul, jõustub muudatus alles pärast adapteri taaskäivitamist või süsteemi taaskäivitamist. Eespool nimetatud käsitsi sisestatavad `Set-NetAdapterAdvancedProperty`-käskud on endiselt hea alternatiiv — need rakenduvad kohe (seovad adapteri uuesti) ilma taaskäivitamiseta.

### Linux-põhised arvutid — pistikupesa vastuvõtupuhvrid

Linuxis, sealhulgas NVIDIA Jetsonil, on tuuma pistikupesa vastuvõtupuhvrid sama olulised kui võrgukaardi vastuvõturing. Tuuma vaikimisi seadistuse korral (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 baiti) kaamera GVSP-pakettide voog põhjustab voo-sokli üleujutuse enne, kui Chloros need ära loeb. Paketid kaovad tuumas, kuigi võrgukaart ise ei teata ühestki kaotusest, ning massiiv kaotab kaadrirühmi ja vähendab kaadrisagedust. Mõõdetuna 4-kaameralisel massiivil, mis on ühendatud USB 10 GbE-adapteriga, tõstis mõlema väärtuse suurendamine 32 MB-ni massiivi kaadrisageduse ühenduse loomise järel 4,96-lt 7,52 fps-ni ning pideva toorandmete salvestamise kiiruse 4,7-lt 7,6 salvestusele sekundis, ilma et tuumikus oleks pakette kaduma läinud.

- **.deb-pakett seadistab need.** Paketi installimisel kirjutatakse väärtus `/etc/sysctl.d/60-chloros-gige.conf` (mõlemad väärtused on `33554432`) ja see rakendatakse kohe. Juba kõrgemaks seatud väärtust ei muudeta.
- **`lattice network` kontrollib neid.** Kui mõni neist on madalam, kuvatakse `socket receive buffers too small for GigE Vision (…)` loendis *Leitud probleemid* ning tagapõhja logis kuvatakse sama hoiatus esimese kaamera või massiivi ühendamisel.
- **Nende käsitsi suurendamiseks** (kuni järgmise taaskäivitamiseni; paketi fail rakendab neid iga käivitamise ajal):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` suurendab ja säilitab need samuti, kuid ainult siis, kui backend töötab root-õigustes. Backend, mis töötab teie kasutajana (see, mis on käivitatud `chloros-backend` või [`chloros-backend@<user>`]-ga (#running-the-backend-as-a-service-headless)) käivitatud backend ei saa muuta tuuma seadeid ning kuvab selle asemel rea `sudo sysctl`.

**Automaatne adapterite häälestamine puudutab ainult ühendatud adaptereid.** Esimesel kaamera ühendamisel häälestab root-õigustes töötav backend neid traadiga adaptereid, mille vastuvõturing on väike. Nüüd häälestab see ainult ühendusega adaptereid, seega säilitab lahtiühendatud sisseehitatud port oma MTU ning vastuvõtupuhvreid ei muudeta. `lattice network` ei näita enam lahtiühendatud adapteriMTU-d enam probleemina. `lattice network`, `--fix`, `lattice probe` ja adapterite häälestamine arvestavad ainult traadiga Etherneti porte, mitte Jetsoni CAN-bussi (`can0`) ega USB-seadmerežiimi ühendusi (`usb0`, `usb1`), mida see pakub host-arvutile; kaamerate jaoks mõeldud USB-Ethernet-adapter loetakse siiski kaasa.

### Võrgu põhitõed (kõik LATTICE-ühendused)

- **Aadressimine:** link-local `169.254.0.0/16` (GigE Vision LLA). Host arvuti võtab staatilise aadressi `169.254.x.x/16`; kaamerad ja DAQ-E määravad endale ise aadressid samas vahemikus. DHCP-d ega väravat ei ole vaja.
- **Paketi suurus:**eelistatakse jumbo-paketti (9000), kuid laske automaatsel proovimisel see leida — see ületab kaamera 1500-baidise ICMP-piirangu GVSP-proovi abil, nii et see jõuab jumbo-pakettini seal, kus kaabel seda tegelikult edastab.**Iga kaamerat mõõdetakse üks kord iga backend-seansi jooksul ja uuesti, kui selle ühendus muutub (teine adapter, teine ühenduskiirus või ühenduse katkemine, nagu lüliti vahetamisel juhtub); uuesti ühendamisel kasutatakse tulemust uuesti. Pärast muutust lüliti taga, mis jäi tööle, taaskäivita backend, et uuesti mõõta.**Esimene ühendus võib selle kontrolli puhul aeg-ajalt võtta paar minutit aega ja logi märgib seda, kui see juhtub. Kasutage `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`-i ainult siis, kui teate paremini kui sond, ning eelistage käsupõhist lahendust püsivale: pin-seade jätab sondi vahele, nii et kui ühendustee ei suuda tegelikult edastada 9000, siis**iga** salvestamine lõpeb ajapiiranguga `SC_ERR_TIMEOUT -1011` (vt [Keskkonnamuutujad](#environment-variables)).
- **RX-ring skaleerub vastavalt `ReceiveBufferLen`-ile:**vaikimisi `32` on kasutatav ring ~0,26 MB (liiga väike mis tahes mitme kaamera pildiseeria jaoks); maksimaalsel `256` on see suur (~13,5 MB, mõõdetud laboris 10GbE-hostil), pakkudes reaalset varu. Kas konfiguratsioon ühendub, otsustab seejärel voolutarbimist arvestav lubatavuse kontroll**ja** allpool kirjeldatud koondatud ülemärkimise kontroll – mitte toores võrdlus purske ja ringi vahel.

### Array fps ja burst-mudel

Kuidas lugeda Array Settings paneeli (ja `lattice analyze-array` / SDK-i `analyze_array_network`):

- **Burst summeeritakse iga kaamera puhul vastavalt selle tegelikule pikseliformaadile.**Mono**M3M**kaamerad edastavad**Mono12 (2 B/px)**või `--pixel-format`-i puhul selle sügavuse Mono-formaati (`BayerRG8` on `Mono8`, 1 B/px);**M3C**Bayeri kaamerad edastavad 8- või 12-bitist voogu (M3C-põhivara edastab vaikimisi BayerRG12-formaati isegi siis, kui nõutakse BayerRG8-formaati). Seega on nelja kaameraga täisresolutsiooniga kaadri suurus**~12,6 MB, kui kõik on 8-bitised, kuid ~25 MB kolme 12-bitise mono-kaameraga**. Projektsioon määrab iga kaamera formaadi kindlaks selle mudeli järgi (identiteedipuhver), nii et andmevoog vastab sellele, mida kaabel tegelikult edastab – mitte ühtsele BayerRG8 eeldusele.
- **USB-Etherneti adapteri maksimaalne kiirus on 200 MB/s, olenemata selle nimikilbil märgitud andmetest.** Tõhusustabel, mis teisendab ühenduskiiruse püsivaks näitajaks, on tuletatud PCIe-st; USB-võrgukaart reklaamib oma *Etherneti* ühenduskiirust, kuid on piiratud USB-siini ja selle draiveriga. USB 10GbE-dongel, mida kasutati, saavutas ~1063 MB/s „püsivaks“ kiiruseks – näitaja, mida kunagi ei uuritud –, ning sellest tulenev andmeedastuse aeglustumine rikkus 6–18 % raamidest, kuigi näidati endiselt korrektset siht-kaadrisagedust (fps). USB-ühendusega võrgukaartide kiirus on nüüd absoluutselt piiratud: **200 MB/s** Windowsis ja NVIDIA Jetsonil, **400 MB/s** Linux x86_64-s (Linuxis on USB 2.0-pordis olevale adapterile eraldatud **30 MB/s** ning tootespetsifikatsioon soovitab ühendada selle USB 3-pordiga) (piirang tuleneb hosti USB-siinist ja draiverist, seega ei sõltu see seadme nimivõimsusest; USB 1 GbE-adapter saavutab ~80 MB/s ja see ei mõjuta kiirust). Linuxis tuvastatakse adapter sysfs-i kaudu (varuvariandina kasutatakse `ethtool -i`-i `bus-info`-i). Iga näitaja on mõõdetud: „Windowsis“ rikkus sama 4-kaameraline süsteem 6–10 % kaadritest kiirustel üle 200 MB/s; „Linux x86_64“ süsteemis töötas sama adapter veatult igal kiirusel kuni 480 MB/s (273 MB/s edastatud, mis on kaamerate enda piir). NVIDIA Jetsonil töötas sama adapter veatult ainult kiirusel 200 MB/s ja kaotas sellest suuremal kiirusel vahetevahel kaadreid. `wire_ceiling_source` võimekuse kirjes on see sõnadega kirjas ja `nic_is_usb` märgib seda. Mõlemal juhul saab seda ümber kirjutada XPR-igaOTX002475 abil.
- **Läbilaskvus arvestab voolutarbimist, mitte kogu purset võrreldes ringiga.** Samaaegne burst peab sobima ainult *ajutise tagantjärele jäänud andmemahtu* = `max(0, Σ per-cam arrival − host drain) × emit_window`, mitte kogu burstiga. Kiire hosti / aeglase kaamera struktuuris (**PCIe**10G host + 4× 1 GbE cam'id: saabumine ≈ 320 MB/s, tühjendamine ≈ 1063 MB/s) tühjeneb host kiiremini, kui kaamerad täituvad, backlog ≈ 0, seega**lubab**täisresolutsiooniga sim-emit andmeid edasi, kuigi 25 MB burst ületab 13,5 MB ringi. Kui paigutada samad neli kaamerat**USB** 10GbE-adapteri taha Windowsis, on väljundkiirus 200 MB/s, mitte 1063 — sissetulev andmevoog ületab väljundkiirust ning kadu ilmneb pigem vigaste raamidena kui madalama kaadrisagedusena. 1 GbE-hostil põhjustab kaamerate 31,25 MB/s DLThr-alampiir, et saabuv andmevoog ületab väljundkiirust → see **blokeerib**korrektselt (selle blokeerimisklassi puhul vähendage ROI-d või kasutage binningut ≥ 2). Lubamine on üks**kahest** ühendusväravast — teine on allpool kirjeldatud koondatud ülemärkimise kontroll.
- **Prognoositav kaadrisagedus (fps) on aeglaseima kaamera tsükkel.**Samaaegse väljastamise korral saadab iga kaamera oma kaadri samal ajal, seega töötab massiiv selle aeglaseima liikme anduri lugemise ja väljastamise aja pikema järgi (kaadri baitid ÷ selle väljastamissagedus: kaamera**juurdepääsuliin**, 1 GbE ≈ 80 MB/s, või selle osa hosti ühenduskiiruse ülempiirist, kui see on väiksem). Iga ekspositsioon kattub eelmise kaadri lugemisajaga, seega ekspositsioon ei kuulu tsüklisse: automaatne säriaeg täidab kiiruse poolt määratud perioodi, seejärel suurendab võimendust, ning hämar stseen ei vähenda enam kiirust (versioonis 1.2.0 see veel mõjutas). Ainult käsitsi valitud säriaeg, mis on pikem kui periood, aeglustab päästikut (vt [Säriaeg massiivi liikmel](#exposure-on-an-array-member)). Kaamerate lisamine ei aeglusta massiivi, kuni nende andmeside osakaalud ei kahaneks. Jadaemissioonis (`slip-emit-and-capture`) liidetakse kaamerate emissiooniajad kokku ning iga elemendi ekspositsioon arvestatakse vaheajasse. Mõõdetud täisanduriga, 12-bitisena, 9-kaameralisel massiivil koos PCIe 10 GbE hostiga: 10,7 kaadrit sekundis või 14,0 kaadrit sekundis [pakitud edastuse](#packed-12-bit-transport---packed) korral.
- **Pärast pidevat salvestamist näitab projektsioon, milline oli salvestuskiirus.** 30 sekundit või kauem kestnud salvestus automaatsel kiirusel salvestab kiiruse, millele süsteem selle kaamerate massiivi suuruse juures sellel arvutil stabiliseerus; kiiruse regulaatori lühikesed katsed kiirema kiirusega jäetakse välja. Sellest alates näitavad `array-connect` rida ja massiivi seaded seda kiirust, kui see on mudeli omast madalam, koos selle piirajaga: `clock-bound` (massiiv töötas kiirusel, millele see stabiliseerus, haaramissilmus ootas oma triggertakt), `loop-bound` (haaramissilmusel ei olnud enam aega), `capture-bound` (haaramine jäi maha ja vaatas rühmi üle) või `measured` (ei ole teada, milline neist). Mudeli näitaja on `wire_fps_max`. Massiiv tõuseb ikkagi võrgu kandevõime piirini.
- **Massiiv töötab igas võrgus – piiravaks teguriks on kaadrisagedus.**Iga kaamera saab võrdse osa**kokkupõrkeohutust ühenduskiiruse ülempiirist**(`sustained × sim_emit_factor`), mis on sünkroniseeritud GevSCPD-ga, ning kõik kaamerad eksponeerivad endiselt sama päästiku järgi. Aeglasel ühendusel on osa väike ja kaadrisagedus langeb sellega koos: 8 täisresolutsiooniga 12-bitist kaamerat 1 GbE-võrgus ilma jumbo-kaadriteta saavad igaüks 52 / 8 = 6,5 MB/s, umbes**1 kaader sekundis**(pakendatuna umbes 1,3 kaadrit sekundis), samas kui 10G-host käitab neid kiirusel 11–14 kaadrit sekundis. Kui kaameraid ei ole võimalik nii aeglaselt sünkroniseerida – selle väljaselgitamiseks lugege iga kaamera GevSCPD vahemikku –, lülitub massiiv üle**järjestikusele**edastamisele (`slip-emit-and-capture`): kaamerad kasutavad ühendust kordamööda täisühenduse kiirusel, käivitatuna üksteise järel, nii et iga raam jõuab ikkagi kohale, kuid liikmed**ei ole üheaegselt avatud**. `analyze-array` ja käsu „connect“ väljund näitavad režiimi; määrake `--force-tier sim-capture-sim-emit`, et keelata tagasipöördumine. Kaadri suurus ei mõjuta seda jagamist, seega**pildiosade ühendamine ja väiksem huvipiirkond suurendavad kaadrisagedust, kuid ei muuda seda, kas massiiv mahub**. Ainult selline liin, mis ei suuda massiivi isegi järjestikku edastada, on**ülekoormatud**— nt kui kaameraid on palju rohkem, kui üks liin suudab toime tulla, või kui sellel voogestab palju teisi kaameraid — ja sel juhul on sümptomiks GVSP-pakettide kadu, mitte madalam kiirus: `analyze-array` nullib saavutatava kaadrisageduse näitajad ja väljastab `** OVER-SUBSCRIBED**`, ühendus loodakse koos valju hoiatusega ning HTTP-ühendus koos `pin_resolution: true` (ranges režiimis) **keeldub**. Lahendused on vähem kaameraid, jumbo-raamid kogu ühenduse ulatuses või kiirem võrgukaart. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` alandab keeldumise valju hoiatuseni katsetööde puhul – vaata [Keskkonnamuutujad](#environment-variables).

### Massiivi tervis — milline alamsüsteem kaotab raame

Ühendatud massiivi `GET /api/camera/array/<array_id>/capability` kannab reaalajas
`health`-plokki, mida hinnatakse uuesti **10-sekundilise** jooksva akna jooksul. See jagab kaadrite kaotuse
kaheks põhjuseks, mis vajavad vastupidiseid parandusi, selle asemel, et teatada ühest „puudulikust”
määrast, mis ei nimeta kumbagi:

| Väli | Mida see tähendab | Milline alamsüsteem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (seeria kaupa) | Raam **saabus, kuid oli struktuuriliselt vigane**— GVSP-paketi kadu. |**Võrk**: ühenduse läbilaskevõime, tempo, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (iga seerianumbri kohta) | Raam **ei jõudnud üldse kohale**— kaamera ei käivitatud või sellest ei väljunud midagi. |**Päästik / sünkroniseerimine**: M8-kaabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Iga kaamera halvim näitaja. | — |
| `per_cam_rate_pct` | Kaamera kohta arvutatud kombineeritud puudulikkuse määr (mõlemad põhjused kokku). | — |
| `stable_for_seconds` | Kui kaua on iga kaamera püsinud alla 0,01 %. | — |

Üle 5 % registreerib backend `[array-health <id>] WARN`-rea, milles nimetatakse jaotus —
esimese rikkumise korral, tõsiduse astme muutumisel, kord minutis, kuni olukord püsib, ja üks kord, kui
olukord laheneb. Rikkis pool kirjutab backend-logifaili rea `[gvsp-corrupt <SN>]`
iga kaamera ja põhjuse esimese tabamise korral, seejärel koondandmed iga 60 sekundi järel; rakenduse logipaneel näitab selle asemel rida
selle asemel rida `[array-health <id>]`. Iga hindamine jõuab ikkagi tagapõhja logifaili;
loendurid liiguvad iga puhvri puhul edasi, hoolimata sellest, mis kuvatakse.

Sama kirje näitab numbrit, millest kogu eraldus sõltub:

| Väli | Mida see tähendab |
| --- | --- |
| `wire_ceiling_mbps` | Hosti kehtiv püsiv andmeedastusmaht, MB/s. |
| `wire_ceiling_source` | Kust see number pärineb, sõnadega — nt `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (Linux i x86_64-platvormil `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) või `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, kui `--wire-ceiling-mbps` (või graafilise kasutajaliidese väli **Wire Budget**) on selle seadistanud. |
| `nic_is_usb` | `true` USB-Ethernet-adapteri puhul — vt eespool toodud USB-piirangut. |

**Tõlgendus:** kui `gvsp_corrupt_rate_pct` on nullist erinev ja `never_arrived_rate_pct` on 0
tähendab, et triggereerimine ja kaabli sünkroniseerimine on täiuslikud ning 100 % kaost on võrguteel
— vähenda `--wire-ceiling-mbps` väärtust ja ühenda uuesti. Vastupidine muster viitab hoopis
sünkroniseerimiskaablile või triggereerimisliinile.

> **`--target-fps` ei ole rikutud raamide reguleerimisvahend.** GevSCPD-i taktsagedus kirjutatakse
> ühendamisel üks kord, seega triggereerimissageduse alandamine muudab töötsüklit, mitte
> samaaegse saatmise pursesagedust. Mõõdetud 5-kordne nõudluse vähendamine ei andnud tulemust;
> juhtme ülempiiri langetamine 240-lt 200 MB/s-le viis sama seadme vigaste raamide määra 10,4 %-lt
> 0,00 %-ni.

> **Töötav massiiv ei saa LATTICE (M3C / M3M) püsivara puhul ise uut plaani koostada.** See
> ei suuda seda ise parandada; katkestage ühendus ja ühendage uuesti, et ühendusaja valija saaks
> uue ülempiiriga uuesti planeerida.

### Sümptom → lahendus

| Sümptom (massiivi seaded / ühendus / `analyze_array_network`) | Põhjus | Lahendus |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` taastub väärtusele 32 (tavaliselt pärast draiveri uuendamist) | Määrake `ReceiveBufferLen`→256, `PendingReceives`→64; ava paneel uuesti (taaskäivita backend, kui see on vanad ringisuurused vahemällu salvestanud) |
| Taaskäivitamine/väljalülitamine hangub; hiljem `Invalid class` WMI-vead, võrgukaart ei lülitu sisse, USB-seadmed puuduvad | Vanad 2016. aasta Realtek USB 10GbE draiverid → BSOD `0x9F` → sundväljalülitused | Uuendage adapteri draiver versioonile ≥ v10.67 (2026), seejärel rakenda uuesti eespool nimetatud vastuvõturingi seaded |
| Massiiv ühendub täisanduriga, kuid kaadrisagedus on madal | Automaatrežiim säilitab täisanduri ja laseb kaadrisagedusel langeda võrgu kandevõime tasemele | Vali üks kiirematest valikutest, mis on loetletud ühendusväljundites (`--binning 2` / `4` või väiksem `--width`/`--height`), lubage jumbo-kaadrid või uuendage ühendust |
| Array näitab korrektset sihtkaadrisagedust, kuid edastab vaid murdosa sellest; `health.gvsp_corrupt_rate_pct` on nullist erinev, `never_arrived_rate_pct` 0 | Hosti järeldatud andmeedastusvõimsus ületab tegelikku võimsust (tüüpiline USB-Etherneti adapteri, kitsa PCIe-kanali või jagatud võrgustiku puhul) | Ühendage uuesti madalama `--wire-ceiling-mbps`-väärtusega ja kontrollige terviseblokki uuesti. **Mitte** `--target-fps` — GevSCPD-i andmeedastuskiirus on ühenduse loomisel fikseeritud |
| Linux: massiiv kaotab raamigruppe ja vähendab oma raamikiirust, kuigi võrgukaart ei teata ühtegi kadumist; `lattice network` loetleb `socket receive buffers too small` | Tuuma pistikupesa vastuvõtupuhvrid on vaikimisi seadistatud, mistõttu kaamera paketid kaovad tuumas | Suurendage `net.core.rmem_max` / `rmem_default` väärtust 32 MB-ni — vt [Linux-hostid](#linux-hosts--socket-receive-buffers). .deb-pakett teeb seda installimisel |
| Kaamerad puuduvad avaldatud rühmadest; `health.never_arrived_rate_pct` on nullist erinev, `gvsp_corrupt_rate_pct` 0 | Käivitus-/sünkroniseerimisrada — kaamerad ei käivitu, ei ole võrguprobleem | Kontrollige M8 sünkroniseerimiskaablit ja `--line`; veenduge, et kõik liikmed on sisse lülitatud (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` ületatud väärtus `analyze-array` või ühenduse keeldumine ranges režiimis (`pin_resolution`; `array over-subscribes the wire`) | Kaabel ei suuda edastada andmeid ühegi kaamerate poolt saavutatava edastuskiiruse juures, ei samaaegselt ega vaheldumisi — kaameraid on palju rohkem, kui kaabel suudab jagada, või sellel edastavad andmeid ka teised kaamerad | Vähem kaameraid, jumbo-raamid algusest lõpuni või kiirem võrgukaart. **ROI/binning EI aita** (jagamise kiirus ei sõltu raami suurusest). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` asendab katseseadmel (aktsepteerib pakettide kadu) |
| Kaamerate massiiv ühendub `slip-emit-and-capture`-is (jadaliides) aeglases võrgus | Kaameraid ei saa piisavalt aeglaselt sünkroniseerida, et kaablit samaaegselt jagada, seega töötavad nad kordamööda; kaadrid ei ole eksponeeritud samal hetkel | Jumbo-raamid või kiirem võrgukaart taastavad samaaegse salvestamise; `--force-tier sim-capture-sim-emit` nõuab seda (ja kaotab selle asemel pakette) |

---

## `chloros-cli daq`

Spektraalsensori käsud. `pool-*`-käskud on lihtsad HTTP-kliendid, mis juhivad andurit tagapõhja püsiva reservi kaudu, seega vajavad nad töötavat tagapõhja (vt [Tagapõhja käivitamine](#start-the-backend)). Tagapõhi haldab ülekannet, mistõttu jagavad CLI- ja SDK-skriptid ühte aktiivset käepidet, selle asemel et võidelda saripordi pärast. `pool-discover` on ainus viis DAQ-M-i BLE MAC-i teada saada.

> **`chloros-cli daq --help`** (või `-h` või `daq help`) loetleb `pool-*` alamkäskud (Linuxis koos kompileerimisel kaasatud otseste riistvarakäskudega), ning `chloros-cli daq <subcommand> --help` (või `daq help <subcommand>`) näitab igaühe valikuid. Need hõlmavad ühendamist, voog, kalibreeritud `.daq`-failide salvestamine, kap-profiilide vahetamine ja DAQ-seadme kaameraga sidumine ning DAQ-E puhul selle võrguseaded, PTP, taaskäivitamine ja püsivara uuendused. DAQ-i saab juhtida ka Python ist käsuga `chloros_sdk.connect_daq_sensor()`, mis kasutab sama koondatud teekonda.
>
> **Kasutage käske `pool-*` igal platvormil.** Need toimivad ühtmoodi Windowsis, Linuxis ja Jetsonis. Windowsis on on `daq discover` ja `daq list` `pool-discover`-i aliased ning iga muu `daq`-käsk, mis ei ole `pool-*`, lõpetab `1` käivitamisel `DAQ direct-hardware commands require the full daq package`. Linuxis ja Jetsonis on `daq discover`, `daq list` ja muud `pool-*`-st erinevad `daq`-käskud DAQ-tööriista otsesed-riistvarakäsud: need avavad anduri CLI-i enda protsessis, väljaspool tagapõhja ressursipooli (seega mitte anduril, mida hoiab tagapõhi, graafiline kasutajaliides või skript), ning seal võetakse vastu `daq discover`, `--only` ja `--timeout`, kuid mitte `--json`. Käesolev viide dokumenteerib ainult `pool-*`-käsklusi.

### DAQ-anduri esmakordse ühendamise töövoog

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` viide

| Alakäsk | Otstarve |
| --- | --- |
| `daq pool-connect` (smart-detect) | Avab anduri tagapõhja reservis. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U kindlal jadaportil. |
| `daq pool-connect --ble` | DAQ-M BLE kaudu, MAC-aadressi automaatne skannimine. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M BLE-võrgu kaudu teadaoleva MAC-aadressiga (eeldab `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E Etherneti kaudu teadaoleva hostinime või IP-aadressiga (eeldab `--eth`). |
| `daq pool-connect --eth` | DAQ-E Etherneti kaudu, hosti automaatne avastamine (mDNS + ARP varuvõimalus; töötab tühja ARP-vahemäluga Windowsis ja Linuxis). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Integreerimisakna / AE-oleku häälestamine. |
| `daq pool-connect --no-stream` | Ühendus loodud, kuid voogedastust veel ei alustata (jätka käskuga `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-korrektsiooni profiil. Tagapõhja vaikimisi väärtus on `sunshine_cosine`; `none` on paljas andur. [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ignoreerib seda (fikseeritud hajuti). **Andurid tarnitakse ilma kaaneta, seega nimetage paigaldatud kaan:** vaikimisi salvestatakse see eeldatava (`cap_id_source=auto_default`) ja töötlemine annab selle kohta hoiatuse, kuna paljas DAQ-U päikesepaiste korrigeerimise korral näitab ligikaudu 30 korda liiga kõrget väärtust. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skaneeri iga transporti, et leida andureid, millega saaksid ühendust luua, ilma neid ühendamata. **Nii leiad DAQ-M-i BLE MAC-aadressi.** `--timeout` kehtib iga ülekande kohta (vaikimisi 5 s, 1–20). Poolis juba avatud andureid ei kuvata – ühendatud DAQ-M lõpetab reklaamimise –, seega kasuta nende jaoks `pool-list`-i. `usb` loetleb ainult USB-jadaseadmeid: arvuti enda jadaportid (`COM1`, Linux i `/dev/ttyS*`) ei saa olla andurid ja need jäetakse välja. |
| `daq pool-list` | Näita kõiki tagapõhja-poolis olevaid andureid. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Seo kaamera või massiiv DAQ-iga, nagu seda teeb graafilise kasutajaliidese valgusanduri valik: selle salvestused kasutavad, sobitavad ja salvestavad selle DAQ-i näidud. `SENSOR` on anduri ID alates `pool-list` või aadress, millel DAQ-ile pääseb ligi. `--clear` (puudub `SENSOR`) katkestab seose ja salvestamine lülitub tagasi värskeimale DAQ-ile. Seos püsib kuni tagapõhja taaskäivitamiseni või kuni selle kaamera, massiivi või DAQ-i lahtiühendamiseni: lahtiühendamine (siin SDK-st või rakenduses) katkestab kõik seosed lahtiühendatava riistvaraga, olenemata sellest, kes need seadistas, ning massiivi liikmete seosed massiiviga. Kaamera peab olema ühendatud, avatud projektis või varem sellel arvutil avatud, ning massiiv peab olema ühendatud või avatud projektis; mis tahes muu seerianumber või ID lükatakse nimepõhiselt tagasi ja midagi ei seota. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Vabastamine — sõltumata sellest, kes anduri avas, vabastab see ka anduri, mille kukkunud skript on jätnud andurite reservi (kõik sellega seotud salvestused lõpetatakse esmalt). Anduril, mida reservis pole, pole midagi vabastada: see väljastab `ID: already released (not in the pool)` ja lõpetab töötamise `0`; vabastamine, mille tagapõhi keeldus või ei kinnitanud, lõpetab töötamise `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Viimased N spektriraami (vaikimisi 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Voogedastuse jätkamine / peatamine. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | .daq-salvestuse alustamine / lõpetamine. `-o` on tagapõhja arvutis vaikimisi `~/Documents/DAQ Live View` (suhteline tee asub kaustas, kust käsk käivitatakse); `--duration` (`-d`, vaikimisi `0` = kuni `--stop`). `--confirm-after S` annab hoiatuse, kui S sekundit pärast käivitamist pole ühtegi mõõtmistulemust saabunud (vaikimisi 20; 0 lülitab selle välja); `--duration` puhul `--heartbeat S` kuvab lugemite arvu iga S sekundi järel ja annab hoiatuse, kui see enam ei kasva (vaikimisi 30; 0 lülitab selle välja). DAQ-seadme-E-seadmel, mis jääb vaikseks, jätkub samas failis, kui tagapõhi on ühenduse taastanud, ning `--stop` loetleb lüngad ([DAQ-E, mis lõpetab andmete edastamise](#a-daq-e-that-stops-delivering)). Ctrl+C lõpetab `--duration` salvestuse ennetähtaegselt ja teatab sellest; ilma `--duration`-ita jätkub salvestamine pärast Ctrl+C-d (väljumine `130`) kuni `--stop`-ini. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Vaheta kap-korrektsiooni profiili käitamise ajal. `--cap-id` on vajalik; `--cap-id none` eemaldab kap-korrektsiooni (paljas andur). DAQ-E-S säilitab `as_recorded`-i sõltumata sellest, mis on seadistatud. |
| `daq pool-enable-ptp --sensor-id ID` | Lülita PTP uuesti sisse DAQ-E-l, mille PTP on välja lülitatud. Seadistus salvestatakse seadmesse, seega jääb välja lülitatud seade välja lülitatuks — selle ajamärgised ei ole PTP-ga sünkroniseeritud — kuni käsk käivitatakse. `pool-list` märgistab sellise seadme (`!! PTP is switched OFF`) ja väljastab selle jaoks käesoleva käsu. Chloros ei lülita seadet kunagi ise tagasi sisse, sest väljalülitamine võib olla tahtlik. |
| `daq pool-info --sensor-id ID [--json]` | DAQ-E enda seaded: selle püsivara (ja kas käesolev Chloros i versioon sisaldab uuemat), selle aadress ja kuidas see saadi (DHCP / link-local / staatiline), salvestatud võrguseaded kuni anduri käivitamiseni (üks salvestatud koos `--no-reboot`-iga), voog sisselülitamisel ja PTP. Anduri puhul, mille voog ei jõua sellesse arvutisse, loeb programm andurit uue ühenduse kaudu ja teatab sellest. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | DAQ-E seadistamine DHCP-le (`--auto`) või staatilise aadressi (`--mask`, vaikimisi `255.255.255.0`). Andur taaskäivitub seadistuse rakendamiseks ja ühendub uuesti varasemate seadistustega, oodates seda kuni `--timeout` sekundit (vaikimisi 45); `--no-reboot` salvestab selle ainult. Vaata [DAQ-E seadme seaded](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Kas DAQ-E alustab andmevoogu sisselülitamisel, kui tarkvara pole ühendatud (salvestatakse andurisse). Chloros alustab andmevoogu igal juhul ise. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Käivita DAQ-E uuesti ja ühenda see uuesti eelmiste seadistustega (ootab kuni `--timeout`, vaikimisi 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Laadige käesoleva Chloros-versiooniga kaasas olev DAQ-E-püsivara vanemat versiooni kasutavale andurile ja ühendage see seejärel uuesti (umbes minut; ärge lülitage seda välja; `--timeout` vaikimisi 45 s, kuni see taastub). Juba ajakohastatud andurit ei muudeta. |

### DAQ-E, mis lõpetab andmete edastamise

Backend ühendab uuesti DAQ-E-anduri, mis asub andurite rühmas ja ei edasta spektrit 20 sekundi jooksul. Selleks lülitatakse andurit toitva lüliti toide välja ja uuesti sisse: andur taaskäivitub koos sellega ja ei edasta andmeid enne, kui talle antakse selleks käsk (välja arvatud juhul, kui `pool-set-auto-stream --on` on aktiveeritud).

- **Esmalt kontrollitakse, kas tegemist on sama anduriga.** Anduri endiselt aadressilt küsitakse – midagi muutmata – anduri nime, MAC-aadressi ja seerianumbrit. Samuti saadab ta päringu anduri nime `daq-e-<id>.local` all, kuid usaldab seda vastust ainult siis, kui see pärineb anduri enda võrgukaardilt (MAC). Muudele vastajatele ei looda uut ühendust ega kirjutata andmeid.
- **See loob uue ühenduse sessiooni seadistustega:** integratsiooniaeg, kaadri keskmistamine, automaatne säri ja katteprofiil jäävad samaks, mitte anduri sisselülitamisel kehtivad vaikimisi seaded. Andur, millel oli PTP sisse lülitatud ja mis naasis sellega välja lülitatuna, lülitatakse uuesti sisse. Seejärel taaskäivitatakse voog ja selle jaoks mõeldud arvuti vastuvõtja.
- **Ta proovib uuesti üha pikemate vahedega:** kohe, seejärel 5, 10, 20 ja 40 sekundi pärast, seejärel iga 60 sekundi järel, kuni andur on kadunud. Iga katse küsib andurilt ainult, kes ta on. Andurit, mis vastab, kuid ei saada ikkagi midagi pärast kolme ühenduse taastamise katset, jäetakse rahule ja sellest teatatakse (`auto-reconnect GAVE UP`). `pool-stream --start` proovib uuesti.
- **`pool-list` näitab, millises seisus see on.** Kui andur vaikib, näitab selle `STREAM FAULT` rida, mida ühenduse taastamine teeb. Rida tühjeneb esimese spektriga ja andur näitab seejärel `reconnected automatically N time(s)`.
- **Salvestamine jätkub samas `.daq`-is.** Fail märgib iga lünga kahe reaga: `event_type` 5 viimases spektris enne lünka ja 6 esimeses pärast seda, mõlemal on ainult ajamärge. `pool-record --duration` väljastab `!! STREAM LOST …`, kui andur on eemal, ja `stream resumed after …`, kui see on tagasi. `pool-record --stop` loetleb iga lünga ja annab hoiatuse, nagu ta teeb ka tühja faili puhul (salvestus, mis ei sisalda ühtegi mõõtmistulemust, lõpetab töö `1`).
- **Salvestus säilitab ühe kalibreeringu.** Kui andur naaseb salvestuse käigus teistsuguse kalibreeringuga, säilitab salvestuse ülejäänud osa algse kalibreeringu ning peatamise hoiatus teavitab sellest. `pool-disconnect` + `pool-connect` võtab kasutusele uue kalibreeringu.
- **Teie käsud on ülimuslikud.** Andurit, mida te `pool-disconnect`-iga peatate või `pool-stream --stop`-iga pausile panete, ei ühendata kunagi uuesti, isegi kui ühendamine on juba käimas.
- **`pool-info` vaikival anduril** loeb andurit uue ühenduse kaudu ja lisab `!! stream NOT reaching this computer`. Andurist, mis ei vasta, teatatakse kohe. `pool-list` ei näita enam PTP-olekut anduri puhul, mis on vastamise lõpetanud.
- DAQ-U ja DAQ-M ei ole hõlmatud. DAQ-M ühendub Bluetoothi kaudu ise uuesti, ning USB-st lahti ühendatud DAQ-U vajab uuesti koodi `pool-connect`.

### DAQ-E-S — fikseeritud hajuti ja kaldenurga andur

**DAQ-E-S** on DAQ-E, millel on kaks tarkvara seisukohast olulist erinevust. Kõik muu – avastamine, ühendamine, PTP, salvestamine, allpool toodud seadme seaded – toimib täpselt samamoodi nagu DAQ-E puhul.**Selle hajuti on püsiv, seega ei vaja see katekorrektsiooni.** Sunshine-tüüpi hajuti on sisse ehitatud ja oli seadmel juba siis, kui mõõdeti selle tehase kalibreerimist, mistõttu kalibreerimine näitab juba õiget allapoole suunatud kiirgust. Chloros seab selle katekoodiks `as_recorded` (ilma profiilita) olenemata sellest, mida küsitakse: selle nimega piirang (`sunshine_cosine`, `fov_*`, `none`) ignoreeritakse koos veateatega ning selle salvestustes säilitatakse `cap_id_source=model`, mistõttu eeldatava piirangu hoiatus selle puhul kunagi ei käivitu. Ärge rakendage andmetele piirangut ka käsitsi: ülemine `sunshine_cosine` näitaks umbes 11 korda liiga kõrget väärtust.

**Seadmel on 3-teljeline kiirendusmõõtur.** Päikesekiirguse hajutaja koosinuskarakteristik kehtib ainult siis, kui see on horisontaalselt paigutatud, seega näitab kallutatud andur valet kiirgustihedust – ja peegeldusvõime võtab vea üle –, ilma et muid sümptomeid esineks. Seetõttu sisaldab iga DAQ-E-S-i näid oma asendiandmeid:

| Väli | Tähendus |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Nurgad kraadides. `None`, kui püsivara märgib nurga määratlemata olevaks (vaba langus, surnud telg). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Kiirendus millig-des. Kui `mag_mg` on kaugel väärtusest ~1000, tähendab see, et seade oli liikumas ja nurgad ei ole selle proovi puhul olulised. |
| `cal_applied` | Seadistatud: teljed ja nurgad on difuusori koordinaatides (rakendatakse seadme paigalduskalibreeringut). Tühjendatud: töötlemata plaadi koordinaatides, mis võivad näidata kuni ~2,3° kallakut, mis ei ole tegelik. |
| `sample_age_ms` | Kiirendusmõõturi proovi vanus spektri saatmise hetkel. |
| `temp_raw` | Kalibreerimata kiibi temperatuuri **trend** — ainult erinevused, mitte kunagi termomeetri näit. |

`daq pool-latest --json` tagastab need iga raami `imu` võtme all (`null` seadmetel DAQ-U, DAQ-M või tavalise DAQ-E puhul). DAQ-E-S-ist salvestatud `.daq` salvestab samad väärtused oma `imu_*` veergudesse (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) iga spektri kõrvale, nii et lugeja saab eristada pilve kolksatunud statiivist. Kõigil teistel mudelitel on need veerud tühjad.

Asendiandmevoog esitatakse eraldi: `daq pool-list` väljastab `!! ATTITUDE STREAM FAULT (spectra unaffected)`, kui seade on maas, ning näitab, mitmest veast see hiljem taastus. Seal esinev viga ei tähista kunagi lünka spektri salvestuses. `daq pool-discover` loetleb seadme kui `DAQ-E-S` (kui seade ei vasta identiteedikontrollile umbes 2 sekundi jooksul, siis kui `DAQ-E`).

### DAQ-E seadme seaded

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` ilma `--gateway`-ita ei salvesta ühtegi väravat** (kõik varem seadistatud väravad kustutatakse), mis sobib ruuterita kaameravõrgustikule.
- **Sellel arvutil peab olema staatiline aadress.** Chloros lükkab tagasi sellise aadressi, mille alamvõrk ei sisalda selle arvuti aadressi anduri võrgus: pärast taaskäivitamist ei pääse miski andurini, et seda tagasi muuta. Kasutage `--force`-i ainult siis, kui sellele arvutile määratakse uus aadress või suunatakse see uude alamvõrku.
- **Taaskäivitamine ei ole lubatud, kui andur salvestab.** Peatage esmalt salvestamine (`pool-record --stop`). Taaskäivitamise hetkel alustatud salvestus suletakse sellega, ning `pool-set-network`, `pool-reboot` ja `pool-update-firmware` lõpevad `Warning:`-reaga, milles nimetatakse salvestuse faili (isegi juhul, kui andur seejärel tagasi ei tule).
- **Taaskäivitamise järel ühendab Chloros uuesti ainult anduri enda.** Ta ootab anduri asukohas – ja pärast `--auto` ka selle nime `daq-e-<id>.local` all – vastust, mis sisaldab anduri nime, hostinime ja mudeliga, selle varasemast võrgukaardist (MAC) ning ühendab selle uuesti vastanud aadressiga. Nime all antud vastus aktsepteeritakse ainult selle MAC-aadressiga, seega arvutist, mis jõuab andurini ruuteri kaudu, või WSL2-st selle vaikimisi NAT-võrgustikus, ei ühendata uuesti andurit, mille DHCP on uuele aadressile suunanud: käsk teatab sellest ja kuvab käsu `pool-connect --eth-host <its address> …`, mis taastab ühenduse anduri varasemate valgustuse ja säri seadistustega.
- **`pool-update-firmware` lükkab tagasi anduri, mille püsivara versiooni ta ei suuda lugeda.** Andur, mille ühenduse taastamine pärast välgu vilkumist võtab kauem aega kui ooteaeg, ei ole rikkis: kui andur vastab, ühendatakse see uuesti, sisestades uue püsivara seaded, ning käsk hoiatab, et ühendus oli aeglane.
- **Püsivara uuendamiseks on vaja sissetulevat TCP-ühendust.** Andur ühendub uuesti selle arvutiga, et pilt alla laadida. Chloros kuulab adapterit, mis suunab anduri poole (kõigil adapteritel ainult siis, kui ta ei suuda kindlaks teha, milline neist), ja teenindab ainult anduri enda aadressi; kõik muud ühendused suletakse teenindamata. Windowsis lubab installija tulemüüri reegel `Chloros Backend (TCP-In)` selle sisse (vt [Windows (.exe)](#windows-exe)); tulemüür, mis seda blokeerib, põhjustab värskenduse ebaõnnestumise veaga `… device never connected back on TCP <port> …`.
- **WSL2-s kasutage peegeldatud võrgustikku** (`networkingMode=mirrored` ja `%UserProfile%\.wslconfig`). Vaikimisi NAT-režiimis on selle arvuti aadress anduri võrgus WSL-i enda aadress, mistõttu lükatakse tagasi kõik staatilised aadressid (välja arvatud `--force`), otseülekanne ei jõua WSL-i ja püsivara uuendamine ei saa lõpule viia, kuna andur ühendub tagasi arvutiga, et saada pilt.
- **Anduri ID ja hostinimi ei muutu.** Mis tahes aadressil DAQ-E ka ei asuks, on see endiselt `daq-e-<id>` ja `daq-e-<id>.local`, mistõttu `--eth-host daq-e-<id>.local` leiab selle ikka üles.
- **GUI-l on samad seaded**: vahekaart „Valgusandurid”, anduri seaded (hammasratas), „Võrk”. GUI säilitab oma ühenduse anduriga, seega kui muudad andurit, millega GUI on ühenduses, GUI kaudu: CLI-st taaskäivitamine lõpetab graafilise kasutajaliidese seansi sellega.
- Vastavad SDK-koodid on `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` ja `update_firmware()`.

---

## `chloros-cli project`

Salvestatud Chloros-projekti avamine, sellega ühenduse loomine ja juhtimine (kaust, mis sisaldab `cameras.json` + `sensors.json` + `project.json`). Kõik suunatakse läbi tagapõhja, nii et graafiline kasutajaliides (GUI) ja käsurealiides (CLI) tekitavad identse riistvara seisundi.

### Alakäskude viide

| Alakäsk | Otstarve |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Prindi projekti seadmete nimekiri (kaamerad, massiivid, andurid), loe selle failidest ja kontrolli selle allikafaile (vaata [Ümberpaigutatud projekt](#a-moved-project---relink)). See ei ava projekti backendis: seal avatud projekt ning selle seadmed ja salvestised jäävad muutumatuks. Lõpetab `1`, kui ühtegi projekti allikafaili ei leita. |
| `project devices PATH [--reconnect]` | Loetleb või käivitab avastamise uuesti. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Ühendab kõik salvestatud kaamerad / massiivid / andurid. Iga massiiv ühendub samamoodi nagu `lattice array-connect`: kuni 600 sekundit, selle sammud kuvatakse stderr-i (`[array <id>]    <s> s: <step>`; stdout on JSON-aruanne), ja massiiv, mida selle aja jooksul ei ühendata, tühistatakse — selle kaamerad taastatakse leitud seisundisse — ning sellest teatatakse `errors` all, jättes need kunagi ühendatuks. `--packed` määrab iga massiivi pakitud 12-bitise edastusviisi (vaikimisi `auto`; projekt ei salvesta massiivi-põhiseid seadeid; vt [Pakitud 12-bitine transport](#packed-12-bit-transport---packed)). Iga valgussensor avatakse tagapõhja DAQ-pools salvestatud aadressil koos salvestatud integratsiooniaja, kaadri keskmistamise, automaatse särituse ja piiranguga: andur, mille jaoks ülempiiri ei ole valitud, saab seadistuse `sunshine_cosine`, üks salvestatud „bare” (`none`) jääb „bare”-seadistusega ning DAQ-E-S ei võta kunagi ülempiiri. GUI DAQ-paneel ei kuulu pooli: sulgege andur esmalt seal. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Üksik pilt nimetatud kaamerast või massiivist (`--prefix` vaikimisi `capture`). `--orient` (massiivid): massiiv salvestab toorandmed, mis jäävad algkujul – see tähendab, et `process` poolt sellest loodud tooted tuleb orienteerida, nagu `array-capture --orient`. Vaikimisi: algkuju. `project burst` / `stream` ja retsepti `capture` / `burst` / `stream` (`orient: true`) kasutavad sama valikut. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | N-kaadriline pildiseeria nimetatud kaamerast või massiivist (`-n/--count` vaikimisi 5; `-i/--interval` kaadritevaheline aeg sekundites, vaikimisi 0). Massiivi pildiseeriad eemaldavad dubleeritud sünkroniseeritud rühmad (aegumise järelevalve), nii et osaliselt tsüklit läbiv massiiv ei saa ühe kaadri kohta N koopiat tagasi anda; trükib iga iteratsiooni tulemused. Pooleli jäänud salvestamine peatab pildiseeria ja lõpetab `1`, näidates, kui palju pilte enne seda salvestati. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Salvestamine kettale tsüklis: `--fps F` salvestab teise (vaikimisi 3), ühe kaadri iga salvestuse kohta kaamerast, ühe sünkroniseeritud kaadrirühma massiivist (sama aegumise järelevalvega kui `project burst`), kuni `-n N` salvestab (vaikimisi `0` = kuni Ctrl+C-ni, mis kuvab, kui palju salvestati, ja lõpetab `0`, või `1`, kui midagi ei salvestatud või faili salvestamine ebaõnnestus; kui Ctrl+C vajutatakse enne, kui `-n N` salvestab, lõpetab programm `130`). Pooleli jäänud salvestamine peatab voo ja lõpetab `1`, näidates, kui palju kaadreid enne seda salvestati. `--poll-interval` on vananenud ja seda ignoreeritakse: seda aktsepteeritakse siiski, et vanemad skriptid töötaksid, kuid stderr-i kuvatakse vastav märkus. |
| `project sensor read PATH NAME [--json]` | Viimane spektri kaader. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Salvestab .daq-faili `--seconds` (vaikimisi 10). Salvestus, mis ei sisalda ühtegi mõõtmistulemust, kuvatakse stderr-is ja programm lõpetatakse `1`. |
| `project run PATH RECIPE.yaml` | Käivita YAML- või JSON-salvestusretsept. `--dry-run` valideerib ilma käivitamiseta. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Arvutab massiivi joonduse — vt [allpool olevat liputabelit](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Prindi praegune joondamisprofiil. |
| `project align clear PATH NAME` | Kustuta vahemällu salvestatud profiil. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Muuda ühe orja teisendust (vaikimisi `0`, `0`, `0`, `1.0`: muutusteta). |
| `project align export PATH NAME --to FILE` | Salvesta profiil JSON-vormingusse. |
| `project align import PATH NAME --from FILE [--no-validate]` | Laadi salvestatud profiil. |

#### `project align calibrate` Valikud

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Joondamismeetod. **Need kirjapildid erinevad `lattice align-calibrate`**-st, mis võtab vastu lühendid `orb` / `akaze` / `phase`; need kaks käsku ei ole selle lipu puhul omavahel asendatavad. |
| `--model {translation, rigid, affine, homography}` | `affine` | Muuda mudelit sobivaks. |
| `--frames N` | `1` | Sünkroniseeritud kaadrite hetktõmmised keskmiseks. |
| `--reference SN` | peakaamera | Viitekaamera seerianumber; kõik ülejäänud liikmed moonutatakse selle järgi. |
| `--max-features N` | `5000` | ORB-tunnuste arvu ülempiir. |
| `--ratio-threshold F` | `0.75` | Lowe’s suhtarvu test. |
| `--ransac-threshold-px F` | `3.0` | RANSACi sisepunktide künnis. |
| `--min-matches N` | `15` | **Kvaliteedikontroll** — lükka lahendus tagasi, kui sissejäävate vastete arv on väiksem kui see väärtus. |
| `--max-reproj-err-px F` | `4.0` | **Kvaliteedikontroll** — lükka lahendus tagasi, kui RMS-ülekandeviga ületab selle väärtuse. |
| `--checkerboard RxC` | — | Plaadi geomeetria `--method checkerboard` jaoks, nt `9x6`. |
| `--name PROFILE` | tühi | Salvestatud JSON-failis sisalduv profiili nimi. **Mitte massiivi nimi** — see on positsiooniline `NAME`. |

Need kaks kvaliteedikontrolli on põhjus, miks kalibreerimine võib lahendamisel õnnestuda, kuid
salvestamine siiski ebaõnnestuda: profiil, mis ei läbi ühtegi neist, registreeriks vaikimisi valesti iga
järgneva pildistuse, seega lükatakse see tagasi, mitte ei salvestata.

### Ümberpaigutatud projekt (`--relink`)

Projekti `project.json` nimetab iga allikafaili — iga pildi ja valgusanduri
salvestuse — selle täieliku tee järgi. Kui projektikausta või kogu projektide kausta
viia teisele kettale, viitavad need teed endiselt vanale asukohale.

Projekti avamisel parandatakse see, mida on võimalik ise parandada: faili, mis asus projekti
kaustas, otsitakse kaustast, kus projekt praegu asub, ja iga seal leitud tee
suunatakse ümber ja salvestatakse (`project.json` salvestatakse esmalt varukoopiana nimega
`project.json.before-relink-<date>-<time>`). See toimub iga avamise korral – nii rakenduse,
`chloros-cli project open`, `chloros-cli process` olemasolevasse projekti, SDK-de
ja `chloros-mcp`-i puhul. Teekond suunatakse ümber ainult seal asuvale failile.

Faile, mis asusid väljaspool projektikausta – näiteks Hub-seanss või „Lisa kaust” import –
ei ole sel viisil võimalik leida. Nimetage kaust, milles need praegu asuvad:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` võtab kaustaks selle, kuhu failid kanti, või mis tahes selle kohal asuva kausta: iga puuduva
faili kausta otsitakse selle all vastavalt failinime lõpus olevale kaustanimele
(`Hub Projects/<session>/raw/<serial>`, seejärel `<session>/raw/<serial>`, ...), ning iga
kausta failid võetakse sellest kohast, kust leitakse neid kõige rohkem. `project open` kuvab
selle, millised viited muudeti, kui palju allikafaile leiti ja millistes kaustades ülejäänud asusid. Mõned puuduvad endiselt: hoiatus, lõpetatakse `0` – projekt avatakse ülejäänud osaga. Ühtegi
ei leitud: viga, milles nimetatakse neid kaustu ja `--relink`, lõpetatakse `1`; tagapõhi
keeldub sellist projekti avamast `process` või `project devices --reconnect` puhul (HTTP 409 `PROJECT_SOURCES_MISSING`).

### Näited

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Retsepti DSL

`project run RECIPE.yaml` võtab vastu YAML- või JSON-faili, milles kirjeldatakse tegevuste jada:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Toetatud tegevused: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` võtab võtmed `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (vanemad nimed `exposure_time`, `exposure_auto`, `gain`, `gain_auto` aktsepteeritakse aliasteina). Sel viisil seatud säritus, võimendus või sihtmärk jääb kehtima: Chlorosi automaatne särituse juhtimine (sealhulgas DAQ-mõõdetud säritus) ei muuda seda. Käsitsi valitud säriaeg jätab võimenduse muutmata: automaatrežiimis reguleerib kaamera võimendust (kogu ulatuses) sihtmärgi heleduse suunas, kuni seadistate `gain_db` või `auto_gain: "Off"`. Hilisem käsk `auto_exposure: "Continuous"` annab säriaja tagasi automaatrežiimile ja käsk `target_brightness_auto: true` annab fikseeritud sihtmärgi tagasi Chlorose automaatsele sihtmärgile. Tegevus `burst` võtab vastu `name` (nõutav), `count` (vaikimisi 5), `interval` (sekundites, vaikimisi 0), `output`, `format`, ja `settings` (sama kaamerapõhine seadistus kui `apply`); massiivpildistamised kasutavad sama värskelt sünkroniseeritud rühma järelevalvet kui `project burst`.

`capture`, `burst` või `stream` sammu puhul on `output` vaikimisi `output`. Suhteline `output` — mis on vaikimisi kaasas — määratakse kindlaks selle kausta järgi, kust te `chloros-cli project run` käivitate, mitte retseptifaili kausta või projekti kausta järgi (versioonis 1.2.0 asus see backendi installikausta).

Käivita see:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Iga käivitamine kontrollitakse esmalt projekti suhtes.**Iga tegevuse võtmesõnaks peab olema täpselt üks eespool nimetatud tegusõnadest — `- capture: {name: FrontLeft}`, mitte `- action: capture` või `- type: capture` — kui `name`-il on projekti `cameras.json` / `sensors.json`, siis kaamera või massiiv `apply` / `capture` / `burst` / `stream` jaoks ning valgusandur `sensor` jaoks; `stream` vajab, et `count` oleks suurem kui 0, ning `wait` vajab kestust. Retsept, milles esineb mõni selline viga,**ei** käivita ühtegi tegevust (`[recipe] error: …`, `[recipe] not run: …`, väljumine `1`), isegi kui `stop_on_error: false` on olemas. Võtmed, mida samm ei kasuta, on `[recipe] warning:`, nagu ka iga massiiv või valgussensor, mida retsept juhib, kuid mis pole hetkel ühendatud. Retsept ei ühenda kunagi seadmeid: käivita kõigepealt `chloros-cli project connect PATH`. `--dry-run` väljastab retsepti, seejärel samad vead ja hoiatused ning `[recipe] valid: …` / `NOT valid: …`, ei käivita midagi ja lõpetab töötamise `1`, kui retseptis on viga.

---

## Keskkonnamuutujad

| Muutuja | Mõju |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Asendab tagapõhja URL-i (vaikimisi `http://127.0.0.1:5000`) — **võetakse arvesse ainult käskude perekondades `lattice`, `project` ja `daq pool-*`** (`lattice network-analysis` kasutab selle asemel `--backend-url`-i). Põhikäsud (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) seovad `http://127.0.0.1:<port>` ja ignoreerivad seda muutujat (IPv4-literaal vältib Windowsi `localhost`→`::1` ~2 s-per-request karistust), seega suunatakse need alati kohalikule arvutile. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Määratakse tagapõhja keskkonnas. `1` vähendab ülemäärase tellimuse tagasilükkamise rangust — massiivi ühendus koos `pin_resolution`-ga või [ajutine mitme kaamera käsk](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — valjuks hoiatuseks, aktsepteerides GVSP-pakettide kadu. Ainult testimiseks — vt [Massiivi kaadrisageduse ja purske mudel](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Määratakse tagapõhja keskkonnas: `on` või `off` määrab iga massiivi jaoks vaikimisi [pakitud 12-bitise transpordi](#packed-12-bit-transport---packed) (vaikimisi `auto`). Selgesõnaline `--packed` on siiski ülimuslik. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (või `true` / `yes`) lülitab käivitamisel välja Linuxi CLI [automaatse uuenduste kontrolli](#chloros-cli-update). |
| `CHLOROS_CLI_MODE` | Määrab CLI ise; käsib tagapõhjal lubada paralleelset töötlemist. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` jätab vahele GVSP varuplaaniproovi (ainult ICMP-tulemused). **See lülitab jumbo välja, mitte lihtsalt vaigistab logi** — kaamera vastab DF-pingidele ainult kuni 1500-ni igal marsruudil, seega on see proov ainus, mis suudab jumbo tuvastada. Säästab ~1 s kaamera kohta ühenduse kohta; kulutab ~1,45× ühenduse maksimaalset läbilaskevõimet, kui võrk *oleks* suutnud jumbo-pakette edastada. SDK annab hoiatuse, kui selle seadistad. |
| `CHLOROS_GVSP_SESSION_REUSE` | Seadistatakse tagapõhja keskkonnas: `0` mõõdab paketi suurust iga ühenduse puhul, mitte üks kord kaamera kohta tagapõhja seansi jooksul. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Kinnitab GVSP-paketi suuruse N baiti; jätab proovimise täielikult vahele. Eelistage käsupõhist seadistust (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) püsiseadistuse asemel: fikseeritud suurus ei kohane enam eesoleva võrguga ning 9000-baidise suurusega fikseerimine teel, mis ei toeta jumbo-pakette, põhjustab **iga** salvestamise ajal `SC_ERR_TIMEOUT -1011`-i ajapiirangu ületamist. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Määratakse tagapõhja keskkonnas: maksimaalne sekundite arv, mille jooksul [ajutine mitme kaamera käsk](#multi-camera-transient-sessions) või kaamera `lattice capture`, mida keegi ei hoia, kulutab iga kaamera ekspositsiooni seadistamisele eelnevast stseenist enne kaadri salvestamist (vaikimisi `5`). `0` lülitab selle välja: iga kaamera oma automaatne säritus töötab siis alates esimesest kaadrist, alustades selle särituse piirist. |
| `CHLOROS_LOG_DIR` | Määratakse tagapõhja keskkonnas: kaust, kuhu salvestatakse selle üksikasjalik logi. Vaikimisi `%LOCALAPPDATA%\Chloros\logs` Windowsis, `~/.cache/chloros/logs` Linuxis (`$XDG_CACHE_HOME/chloros/logs`, kui see on määratud). .deb-faili juurteenus määrab `/var/log/chloros`, mida saab lugeda rühm `adm` — vt [Tagapõhja käitamine teenusena](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Ajutiste failide asukoht. CLI kasutab automaatselt `/mnt/ssd/tmp`-i, kui see on olemas. |

---

## Väljumiskoodid

| Kood | Tähendus |
| --- | --- |
| `0` | Edukas. |
| `1` | Üldine viga (enamik alamkäskude vigu). Salvestuskäsk lõpetab töö ka koodiga `1`, kui mõnda faili, mida paluti kirjutada, ei õnnestunud kirjutada, isegi kui ülejäänud salvestati, ning koodiga `array-burst` / `array-record` lõpetab töötamise, kui salvestaja teatab veast — vt [Kirjutamisel ebaõnnestunud failid](#files-that-fail-to-write). `process` lõpetab töötamise ka `1` juhul, kui käivituse eksporditulemusi ei õnnestunud kontrollida või kui kaotati pilt, kuigi teised tooted salvestati. |
| `2` | Argumendiviga (tundmatu või puuduv lipuk; samuti `time-sync set-priority`, kui kumbki prioriteet puudub). |
| `130` | Katkestatud klahvikombinatsiooniga Ctrl+C: iga käsk, mida see katkestab — `lattice`, `project`, `daq` ja `process` (`process` pärast seda, kui tagapõhi on käivituse peatunud, vt [`process` märkused](#notes)). Käsk, mille tavaline peatamine toimub klahvikombinatsiooniga Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` ja `project stream` koos `-n 0`-ga — lõpetab töö vastavalt sellele, mida ta tegelikult tegi: `0`, kui kõik oli kirjutatud, `1`, kui midagi ebaõnnestus. |
| `143` / `129` | Lõpetati SIGTERM-i / SIGHUP-i (128 + signaal) poolt pärast käsu puhastamise käivitamist: töötav ülesanne peatati ja selle kaamerad pandi tagasi, töötlemine peatati tagapõhiselt. `lattice continuous` või muu salvestus, mille tavaline peatamine toimub klahvikombinatsiooniga Ctrl+C, lõpetab ka `143`, mitte `0`. |

`lattice network-analysis` lõpetab `1`-i `needs_force_slip`-i ja `error`-i otsuste korral, ning `selftest` lõpetab `1`, kui kõik seitse kontrolli ei ole edukad.

---

## Vihjeid probleemide lahendamiseks

- **„Vajalik sisselogimine”** → Käivita `chloros-cli login EMAIL PASSWORD` sellel arvutil üks kord.
- **„Projektikausta asukoht on selle arvuti jaoks liiga sügav”** → Selles arvutis on Windowsi pikad teed välja lülitatud (Windowsi vaikimisi seade), seega ei tohi ükski tee ületada 260 tähemärki ning tooted asuvad kuni umbes 130 tähemärki allpool projektikausta. Mitte midagi ei töödeldud. Teade näitab pikimat sobivat projektikausta: kasuta lühemat (`-o` või `chloros-cli set-project-folder`) või lülita sisse Windowsi pikad teed ja taaskäivita Chloros. Windowsi paigaldaja pakub seda valikuna „Luba Windowsi pikad teed”, mis on vaikimisi märgitud; muidu kasutage rühmapoliitikat: *Arvuti konfiguratsioon > Haldusmallid > Süsteem > Failisüsteem > Luba Win32 pikad teed*.
- **„backend unreachable“** → Käivita Chlorosi töölauarakendus (Windows) või käivita backend ise: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` Windowsis (sinu installikausta kaust `backend`; seda pole `PATH`-is), `chloros-backend` Linuxis. Või kontrollige `CHLOROS_BACKEND_URL`-i, kui kasutate kaugjuurdepääsu.
- **`lattice`-käskude täitmine ebaõnnestub veateatega „LATTICE kaameradraivereid ei leitud”** → Tagapõhi ei suutnud laadida LATTICE kaamera käitamiskeskkonda, millega see on paigaldatud. Iga `lattice`-käsk töötab tagapõhis; CLI-liides ise ei sisalda kaameradraivereid. Paigaldage Chloros uuesti: Windowsi paigaldaja ja .deb-fail paigaldavad käitamiskeskkonda tagapõhi kõrvale.
- **Array connect / Array Settings näitab „FRAMES WILL DROP“ või „Reduce ROI to enable“** → Hosti võrgukaardi vastuvõturõngas on liiga väike (tavaliselt taastub see pärast võrgukaardi draiveri uuendamist väärtusele 32). Vaadake [Host NIC Setup & Tuning](#host-nic-setup--tuning-lattice-arrays) — seadistage `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Seade hangub taaskäivitamisel/väljalülitamisel, seejärel ilmub WMI-veateade `Invalid class` / võrgukaarti ei saa aktiveerida / USB-draivid puuduvad** → Vananenud USB 10GbE-adapteri draiver põhjustab veateate `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Uuendage adapteri draiverit — vt [Host-võrgukaardi seadistamine ja häälestamine](#host-nic-setup--tuning-lattice-arrays).
- **Linux: salvestised kuuluvad kasutajale root, seega ei saa neid kustutada, ja tagapõhja logi ei asu sinu `~/.cache`** → Tagapõhi töötab kasutajana root kaudu `chloros-backend.service`. Vahetage kataloogiks `chloros-backend@<user>.service`, kus backend töötab teie kasutajana — vt [Backendi käitamine teenusena](#running-the-backend-as-a-service-headless). Seni asub root-teenuse logifail `/var/log/chloros`-is (mida saab lugeda rühm `adm`).
- **Linux: tagapõhjaüksus on `failed` koos veateatega „Teine instants juba töötab“ (lõpetusstaatus 98)** → Teine tagapõhjaüksus hoiab porti juba hõivatuna: korraga töötab ainult üks. Kui kaks backend-üksust on aktiveeritud, säilita üks: `sudo systemctl disable --now <unit>` selle jaoks, mida sa ei kasuta — vaata [Backendi käivitamine teenusena](#running-the-backend-as-a-service-headless). Kui ainult see üksus on aktiveeritud, hoiab seda süsteemist väljaspool käivitatud backend (käivitatud käskudega `chloros-backend`, `chloros-cli` või SDK-skriptiga): peatage see backend ja seejärel käsk `sudo systemctl start <unit>`.
- **Jetsoni vahetusmälu hoiatus** → Lisage failipõhine vahetusmälu; CLI kuvab täpsed käskud `fallocate` / `swapon`.
- **„backend not reachable“ (tagapõhja ei ole võimalik saavutada) `lattice`, `project` või `daq` käsust** → ükski tagapõhi ei tööta: need käsud ei käivita kunagi ühtegi. Käivita käsk `chloros-cli backend status` ja vaata jaotist [Backendi käivitamine](#start-the-backend).
- **"`daq net ...` ei ole Chlorosis saadaval"** (Linux ja Jetson) → Mõned DAQ-E enda seaded on reserveeritud MAPIR-teenusele. Võrguseaded, taaskäivitamine ja püsivara uuendamine on `pool-set-network`, `pool-reboot` ja `pool-update-firmware`; ülejäänu puhul võtke ühendust MAPIRi tugiteenistusega.
- **„DAQ-i otsesed riistvarakäsud nõuavad täielikku daq-paketti“** (Windows) → Kasutage `pool-*`-i, mis juhib sama andurit tagapõhja kaudu, või `chloros_sdk.connect_daq_sensor()`-i Pythonist.

---

## Vaata ka

- [Python'i SDK-viide](sdk-reference.md) — iga CLI-käsu programmeerimislik vaste.
- [DAQ-valgusandurid](https://mapir.gitbook.io/chloros/daq-light-sensors) Chloros-kasutusjuhendis — andurid, nende piirangud ja kalibreeritud vahemik, salvestamine, DAQ-E võrgustik.
- [LATTICE-kaamerad](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) Chloros-kasutusjuhendis — ühendamine, kaamera seaded, pildistamisrežiimid, massiivid.
- Veebidokumendid: `https://mapir.gitbook.io/chloros/cli`
