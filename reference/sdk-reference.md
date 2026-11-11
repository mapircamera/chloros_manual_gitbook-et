# Chlorosi Pythoni SDK-viide

**Versioon:**

1.3.0**Koostatud:**

29.07.2026 kell 19:19 ·**Muudetud:**

06.11.2026**Pakett:** `chloros-sdk` (PyPI)**Sihtrühm:** Optimeeritud LLM-i kasutamiseks; inimesele loetav.**Ulatus:** Kõik `import chloros_sdk` poolt avalikustatud klassid, funktsioonid ja abifunktsioonid, koos kopeeritavate näidetega, mis hõlmavad pilditöötlust, ühe kaamera juhtimist, sünkroniseeritud massiive, DAQ-andureid ja projekti automatiseerimist, lisaks `chloros-mcp`-serverit tehisintellekti abilistele.

Kui soovite tutvuda vaid olulisema infoga, minge otse järgmistesse osadesse:
- [Paigaldamine ja kiirstart](#installatsioon)
- [Smart-Connect LATTICE-kaamerate jaoks](#smart-connect-for-lattice-cameras)
- [DAQ-andurite seansid](#daq-sensor-sessions)
- [Kaamera abiprogrammid](#camera-utilities)
- [Transient-massiivid, joondamisprofiilid ja indeksid](#transient-arrays-alignment-profiles--indices)
- [Projektikausta ja retseptid](#project-folder--recipes)
- [Projekti automatiseerimine](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Reaalajas kaadrid oma töötlusprotsessis](#live-frames-in-your-own-pipeline)
- [MCP-server (AI-abistajad)](#mcp-server-ai-assistants)

> **Muudatused versioonis 1.3.0** (mida olemasolev 1.2.0 skript märkab).
>
> **Muudatused, millega 1.2.0 skript võib vajada tegelemist:**
>
> - **Väljaanne, mida tagapõhi ei kinnita, tekitab nüüd vea.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` seanssi poolt avatud massiivi puhul, `DAQSensorSession.disconnect()` / `force_disconnect()` selle avatud anduri puhul ning `ChlorosProject.disconnect_all()` tekitab veateate `ChlorosConnectError`, kui tagapõhi vabastamist keeldub või seda õigeaegselt ei kinnita, sest seade võib endiselt hõivatud olla. Versioonis 1.2.0 naasid need märkamatult. Bloki `with` väljumine ei tekita endiselt kunagi viga.
> - **Laenatud seadet ei lammutata kunagi.** `with attach_array(...)` ja selle peal olev selgesõnaline `disconnect()` jätavad massiivi töötama (versioonis 1.2.0 lammutas ploki lahkumine selle, isegi kui GUI seda omas); kutsu välja `force_disconnect()`, kui soovid seda lõpetada. Samamoodi jätab `connect_daq_sensor()`, mis leidis anduri juba reservist (sealhulgas null-lipuga nutikas tuvastamine), selle avatuks `disconnect()`-i ja selle `with`-ploki lahkumisel.
> - **PNG ja JPEG võtavad vastu ainult `display` ja indeksitooteid.** `capture()` koos `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) ning `radiance` või `reflectance` tasemel, või `processing="all"`, tekitab enne iga päringut `ValueError`; 12-bitine `raw` / `debayered` lükatakse tagapõhja poolt tagasi. Vaata [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` avab selle anduri või tekitab** `ChlorosConnectError`, loetledes leitud andmed; see ei lülitu kunagi tagasi teisele andurile. Vaata [Kinnitatud transport](#pinned-transport).
> - **`frame_stream()` tekitab vea asemel, et vaikida:** `FrameFetchError` kordab keeldumise korral järgmisel taktil (peegeldusnäidu lugemine, mis ei sisalda üldse DAQ-i), pärast 60 sekundit ilma raamita ja pärast 60 sekundit ilma uue triggita. Vaata [Üks trigger lugemise kohta](#one-trigger-per-read--group_id).
> - **Lugemise ooteaja lõppemine või ühenduse katkemine tekitab veateate `ChlorosConnectError`** igast smart-connect-kutsest ja `analyze_array_network()`, kus versioonis 1.2.0 tekitas aluseks oleva ooteaja lõppemise või ühendusvea.
> - **`connect_array()` ootab vaikimisi 600 sekundit (`timeout`, versioonis 1.2.0 180 sekundit) ning ühendus, mille loomist ta loobub, tühistatakse.** Kõne saadab oma `timeout` tagapõhja, mis taastab iga kaamera sellisena, nagu see leiti, ja ei ühenda midagi, kui ühendust ei õnnestu õigeaegselt luua; kõne tekitab vea `ChlorosConnectError` (HTTP-kood 504 `CLIENT_DEADLINE_PASSED`). Versioonis 1.2.0 võis kohe pärast kaamerate sisselülitamist algav ühendus kestada kauem kui kõne ja ühendada massiivi pärast seda, kui kõne oli juba lõpetatud. `connect_all()` ootab iga salvestatud massiivi puhul samamoodi (120 s versioonis 1.2.0). Vaata [`connect_array()` allkirja](#connect_array-signature).
> - **`connect_array()` säilitab täieliku andurite massiivi.** Ilma `width` / `height` / `binning`-ita on iga kaamera seadistatud täisandurile ja kaadrisagedus langeb sellele tasemele, mida host ja võrk suudavad pakkuda; versioonis 1.2.0 vähendas või koondas kaadreid, et kaadrisagedust säilitada. Vaata [Sünkroniseeritud massiiv](#synchronized-array--arraysession-smart-prep).
> - **Bayer-pikseliformaat M3M (mono) kaameral on selle sügavus.** M3M pakub ainult monoformaate ning eelseadistused nimetavad neid Bayer-formaatideks: versioonis 1.2.0 keeldus kaamera seda formaati kasutama ja säilitas omaenda formaadi, samas kui `settings.pixel_format` nimetas seda Bayeri formaadiks. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` ja massiivijagatud formaat määravad nüüd M3M-is sama sügavusega Mono-formaadi: `"BayerRG8"` ja `high_speed` annavad tulemuseks `Mono8`; `"BayerRG12"`, `high_quality` ja `triggered` annavad tulemuseks `Mono12`; `cam.settings.pixel_format` nimetab selle. Sama kehtib ka vastupidi: M3C-d ei salvestata kunagi Mono-formaadis, seega `"Mono12"` – mis tahes neist, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` või massiivi salvestatud formaadist – annab M3C-l tulemuseks `BayerRG12`, sõltumata sellest, milline kaamera on massiivi juhtkaamera. Edastatud `CameraSettings` säilitab oma nime, seega küsib sama objekt endiselt igalt kaameralt seda sügavust oma perekonnas. `connect_camera(preset=...)` säilitab eelseadistuse formaadi (versioonis 1.2.0 seadistatakse kaamera pärast eelseadistuse rakendamist 12-bitiseks) ning eelseadistuse alusel avatud kaamera ei kirjuta enam oma salvestatud andmeid `PRESETS`-i. Mis tahes muu formaat, mida kaamera keeldub (mis puudub selle `PixelFormat` nimekirjast, näiteks `RGB8` M3M-il või `BayerRG16` selleta püsivara puhul), ei nimetata ka eraldi: pärast `LatticeCamera` ühenduse loomisel nimetavad `apply_settings()` või `set_format()`, `cam.settings.pixel_format` ja `packed_transport` nimetavad formaati, milles kaamera edastab voogu, ning üks trükitud rida nimetab formaati, mida kaamera keeldus vastu võtmast.
> - **`analyze_array_network()` näitab, milline ühendus kehtib:** `pixel_format=None` (varem `"BayerRG8"`) tähistab formaate, mille ühendus määrab (Mono12 / BayerRG12), ning `packed="auto"` näitab pakitud edastuse valikut, mistõttu võib selle poolt näidatud kaadrisagedus erineda versioonist 1.2.0&#x27;. Kui edastad `pixel_format`, siis projitseerib see M3M-i sellel sügavusel. Vaata [Võrguanalüüsi abivahend](#network-analysis-helper).
> - **`CameraSession.capture()` tagastab `CaptureResult`** (endiselt raamidiktide nimekiri) koos `.skipped`, `.failed`, `.daq_files` ja `.daq_matches`-ga. `ArraySession.capture()` tulemus sisaldab ka `.alignment` (kas joondamise moonutust rakendati ja miks mitte) ning `.oriented`.
> - **Massiivide hõivamised joondatakse vaikimisi, nagu ka rakenduse omad.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` ja `ArrayHandle.capture()` / `capture_smart()` paigutavad iga liikme `debayered`, `display`, `radiance`, `reflectance` ja indeksifailid peakaamera kaadrisse ning kärbib neid liikmete kattumisala (mis on kaamerate nihkete võrra väiksem kui sensor, `XMP-mapir:AlignmentJSON` `"applied": true`), nii et need on samad failid, mida rakendus Chloros samast stseenist salvestas; `raw` jääb algupäraseks. Varem salvestati jäädvustus ilma `aligned=True`-ita iga faili algses suuruses, moonutamata. `aligned=False` säilitab selle. `chloros-mcp`&#x27;`capture_array` ja `start_capture_run` kasutavad sama vaikimisi seadet. Vaata [`ArraySession` meetodeid](#arraysession-methods).
> - **`connect_all()` nimetab tagapõhja, mis keeldub SDK-st.** Töötav tagapõhi, mis on välja logitud või kasutab Copperist madalamat paketti, kuvatakse `errors` all koos oma põhjendusega (HTTP 401 / 403), mitte kui „ükski Chloros-tagapõhi ei tööta”, ning iga `connect_all()`-kõne kontrollib tagapõhja uuesti, seega selle kutsumine pärast sisselogimist toimib sama projekti objekti puhul.
> - **Suhteline `output_dir` töötab kohaliku backendiga.** Suhtelised teed muudetakse enne saatmist absoluutseteks teie skripti töökataloogi suhtes. Versioonis 1.2.0 lahendas backend need oma installikataloogis, mis lõppes veaga „Juurdepääs keelatud”.
> - **`numpy` on nüüd SDK-i nõue** (live-frame&#x27;i kutsed tagastavad numpy-massiive).
> - **Eksporditaseme nimi, mis ei ole „one”, lükatakse tagasi.** `processing=` ja `levels=` võtavad vastu `raw`, `debayered`, `radiance`, `reflectance` või `display` (või selliseid aliase nagu `preview`; `processing=` ja ka `"all"`). Kõik muud nimed lükatakse tagasi enne kaamera puudutamist (HTTP 400 `UNKNOWN_LEVEL`: `capture()` tekitab `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); versioonis 1.2.0 salvestati või tagastati selle jaoks `raw`.
> - **LATTICE-pildistuste puhul ei lubata kasutada funktsiooni „Texture Aware”.** Projektis, mis sisaldab LATTICE-pildistusi, tekitab `configure(debayer="Texture Aware (Slow, Highest Quality)")` vea `ChlorosConfigurationError`, samuti tekitab seda `process()` pärast `configure()`-i, mis küsis seda enne importimist. Projekti varem salvestatud Texture Aware väärtus seatakse Standardiks koos veaga `UserWarning`. LATTICE-kaadrid saavad alati standardse demosaici, seega versioon 1.2.0 aktsepteeris seadet ja ignoreeris seda; ka rakendus Chloros ei paku seda nende jaoks. Vaata [Toetatud väärtused](#supported-values).
> - **Kaamerad taastatakse selliseks, nagu nad leiti — välja arvatud juhul, kui `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, iga ajutise funktsiooni lõpp (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) ning funktsiooni `start_camera_continuous()` lõpetamine taastavad iga kaamera algse seisundi: selle päästiku ja sünkroniseerimisliini ühendused, PTP ning kõik seansi käigus muudetud kaamera seaded (valgustusaeg ja võimendus, AE-sihtmärk, piirid ja summutamine, valgebalanss, mustatase, pikseliformaat, binning ja ROI, peegeldused, kaadrisagedus). `keep_settings=True` salvestab aga iga kaamera seaded kaamera enda mällu, nii et kaamera käivitub nendega; `clear_settings=True` taastab selle mälu tehase seadetele. Mõlemal juhul jäetakse kaamerad vabalt töötama. Projekt seadistab oma kaamerad nende enda salvestatud seadistuste järgi, kui need ühendatakse. Vaata [Kaamera mälu](#camera-memory--keep_settings--clear_settings).
> - **Iga režiim salvestab projekti samal viisil.** Iga muudatus, mis tehakse projekti alla ühendatud kaamerale, massiivile või valgussensorile – olgu see tehtud SDK, rakenduse, CLI või MCP-serveri kaudu –, salvestatakse selle projekti `cameras.json` / `sensors.json`, ning projekti avamine mis tahes režiimis seadistab seadmed selle järgi; `ChlorosProject` määrab oma kausta nime tagapõhjasüsteemis. Varem salvestas projekti ainult rakendus. Vaata [Üks projektikirje, iga režiim](#one-project-record-every-mode).
> - **Mitmest kaamerast loodud failid nimetavad igaühe eraldi.** Mitme kaamera sagedusribadest koostatud multiband-piltide kogum, mitme kaamera sagedusribadest koostatud reaalajas indeks ja masspildistuse `composite/` pilt sisaldavad iga allikakaamera seerianumbrit, mudelit ja kalibreerimispaketti loenditena (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), ning EXIF-andmed `SerialNumber` loetlevad seerianumbrid komadega eraldatuna. Varem ei sisaldanud need ühtegi kaamerat või ainult esimese kaamera seerianumbrit. Vaata [Mitmest kaamerast loodud failid](#files-made-from-several-cameras).
> - **Ainult fail `raw` sisaldab kaamera kalibreerimismärke.** MicaSense’iga ühilduv komplekt (`BlackLevel`, `RadiometricCalibration`, igakanaliga `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*` komplektid) ning `CalibrationJSON` andmekogum kirjeldavad anduri toorväärtusi, seega asuvad need ainult `raw` salvestusfailis. Kõik ülejäänud failid — salvestusfailid`debayered`, `display`, `radiance`, `reflectance` ja indeksifailid, pideva salvestamise ja mitme kaameravoo kaadrid ning iga `process()` toode – sisaldab kaamera identiteeti (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) ning pildistamisel ka kaadri enda salvestusandmed (`XMP-mapir:FrameJSON`: säriaeg, võimendus, temperatuur, pikseliformaat, pildistamise aeg). Versioonis 1.2.0 sisaldasid need ka toorandmete komplekti, mida seda rakendav tööriist (Pix4D, Metashape) kohaldas pikslitele, mis ei olnud toorväärtused. Töötlemine leiab sellise faili kalibreerimise selle seerianumbri ja `CalBundleSha256` järgi.
>
> **Muud muudatused:**
>
> - **Massiivi automaatne kaadrisagedus kohandub vastavalt sellele, mida massiiv edastab.** Sagedus, mis kaotab rohkem kui 1 % oma triggereist, langetatakse – seda hinnatakse mitme pika vaatluse põhjal, mitte ühe põhjal – ning sagedust, millelt see langetati, proovitakse aeg-ajalt uuesti. Kasutaja poolt ise seatud sagedust ei muudeta.
> - **Automaatne säritus on stabiilsem.** See püsib konstantsel valgustasemel, stabiliseerub muutuse järel mõne sekundi jooksul, ignoreerib väikeseid ülevalgustatud alasid ja taastab kaamera töötamise, kui see on päikesepaistes reageerimise lõpetanud. Vaata [Kuidas automaatne säritus toimib](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` loendab ainult teie pilte ja salvestusi.** Kausta, mis sisaldab projekti `camera_cal/`-faili, kalibreerimisfailid ei loeta enam piltide ja salvestustena, ning projekt, mis on need juba lisanud, eemaldab need.
> - **`configure(radiometric_output=...)` kehtib.** Nüüd määratakse projekti `Export radiance` / `Export reflectance` tooted; varem olid iga projekti enda tooteseaded ülimuslikud, seega eksportisid `radiance`, `sensor-response` ja `all` kõik kiirgust ja peegeldust.
> - **`test_array_sync()` käivitub kaamerate sisselülitatud olekust.** See lülitab kaamerate PTP-režiimi sisse käivitamiseks, ootab, kuni nende kellad lukustuvad ja stabiliseeruvad enne esimest triggert (`ptp_lock`) — iga kaamera lukustub 120 sekundiks, umbes 2 minutit pärast sisselülitamist (`ptp_settle_s=` seadistab selle), kuna äsja sünkroniseeritud kell võib umbes 2 minuti jooksul erineda kuni ~200 us — ning taastab iga kaamera PTP-seadistuse sellisena, nagu see oli. Kaamerat, mis on eelmisest testist alates endiselt sünkroniseeritud, ei oodata uuesti. Varem jäeti PTP välja lülitatuks ühe lülitiga võrgus, kus ükski DAQ-E ei vastanud, ning loeti iga kaamera vabalt töötavat kella PTP-kellana, mis ei sünkroniseeru („PTP ei ole lukustatud“, iga käivituse korral).
> - **`.daq` salvestus näitab, kas selle kate valiti või eeldati.** DAQ-U, DAQ-M ja DAQ-E tarnitakse ilma korgita ning Chloros ei näe, kas see on paigaldatud, seega saab nimetatud korgita ühendatud andur vaikimisi nimetuse `sunshine_cosine`, mis on *oletus*. Iga Chlorosi salvestus (skeem 1.30) salvestab nüüd `cap_id_source`-i kõrval ka `cap_id`-i: `operator`, kui andsite korgile nime (sh `none`), `auto_default`, kui see oli vaikimisi. Kui töödeldakse salvestust, mille kondensaator oli eeldatud, ja projekti DAQ-kondensaator on jäetud seadistusele „Auto“, kuvatakse faili nimeks `[daq-dls] WARNING`: päikesekorrektsioonita DAQ-U näitab umbes 30 korda liiga kõrget väärtust. **Nimetage paigaldatud kondensaator** ühendamisel (`connect_daq_sensor(cap_id=...)` või `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap` või GUI kondensaatori rippmenüü), või seadistage projekti DAQ-kondensaator, ja hoiatus kaob.
> - **Windowsis ei kuvata enam pip-konsooliaknaid.** Arvutis, kuhu on installitud Python, käivitasid backend ja iga selle pilditöötleja käivitamisel ühe korra `pip` ning igaüks avas konsooliakna. Nüüd seda enam ei juhtu.
> - **Reaalajas `.daq` salvestused säilivad ka voolukatkestuse korral.**Salvestus kirjutatakse iga kinnituse korral koos päevikuga ja täieliku kettasünkroniseerimisega anduri taga, seega aeglane ketas viivitab faili salvestamist, kuid mitte kunagi mõõtmistulemusi. Salvestus, mis peatati keset kinnitamist (voolukatkestus, sundlõpetamine), võib jätta kõrvale faili `<name>.daq-journal`:**kopeerige need kaks kokku.** Chloros taastab ajakirja esimesel faili lugemisel, seega avatakse salvestus koos kõigi enne peatamist kinnitatud andmetega (umbes viimased 3 sekundit lähevad kaduma); versioonis 1.2.0 võib valel hetkel toimunud voolukatkestus jätta kogu faili loetamatuks („andmebaasi kettapilt on vigane”).
> - **`DAQSensorSession.record_start()` hõivatud kettal ei põhjusta enam anduri takerdumist**: mõõtmistulemused pannakse järjekorda ja kirjutatakse omaette protsessis, ning see, mida ei õnnestunud kirjutada, loetakse `status()`-i `record_errors`-is (koos `record_last_error`-iga).
> - **Otsene `LatticeCamera`:**kaader, millel puudub tükkide ekspositsioon/võimendus, edastab nüüd kaamera**tegeliku** ekspositsiooni ja võimenduse (varuvariandina loeti vahemällu salvestatud väärtust, mis võis jääda muutumatuks kogu automaatse ekspositsiooni jooksul), ning `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` rakendavad sama 5 µs sisendi glitch-eemaldust, mida saavad Chlorosi kaabelorjad (filtreerimata, orja poolt vallandatud kaadrid, millele ei saadetud triggert).
> - **Tarkvaratriggeriga kaamerad ei jäta enam iga teist triggerit vahele.** Tarkvaratriggeri režiimis olev kaamera paneb nüüd järjekorda triggeri, mis saabub selle väljundperioodi jooksul (või Chloros ootab, kuni see on valmis), nagu kaabelsünkroniseeritud kaamerad seda juba tegid.
> - **`discover_daq_sensors()` nimetab DAQ-E-S-i `DAQ-E-S`-iks.** Iga Ethernet-anduri puhul oli selle `model` nimeks `DAQ-E`; andur, mis ei vasta identiteedikontrollile umbes 2 sekundi jooksul, on endiselt nimekirjas nimega `DAQ-E`.
> - **Uus: `chloros-mcp`, MCP-server tehisintellekti abilistele.** Claude Desktop, Claude Code, Cursor ja muud MCP-kliendid suudavad töödelda pilte (luua projekti, importida pildistusi, valida seadeid, töödelda, esitada aruandeid eksporditud tulemuste kohta) ning juhtida LATTICE-kaameraid, maatrikseid ja DAQ-valgusandureid: ühendada, eelvaadata, seadistada säriaega ja värve, pildistada, salvestada. Selle tööriistad hõlmavad kõiki selle SDK funktsioone. See installitakse koos SDK-ga ning Linuxi .deb-fail paigutab selle kataloogi `/usr/bin`. Vaata [MCP-server (AI-abistajad)](#mcp-server-ai-assistants).
> - **Uus: kaamera abiprogrammid**, `chloros-cli lattice`-käskudena, mida SDK kutsub välja: värviprofiil, küllastus / kontrast, heledad alad, valgebalanss (ühekordne, automaatne, fikseeritud suhted), eelseadistused, filtrimudelid, püsivara seisund ja uuendamine, toide (kontroll, puhkeolek, äratamine), sihtmärgi kalibreerimine, võrgukaardid (kontroll, hinnang, parandus), failide vinjeti korrigeerimine, DLS-näidu lugemine ja hosti kontroll. Vaata [Kaamera utiliidid](#camera-utilities).
> - **Uus: ülejäänud CLI on SDK-kutsed**, seega pakuvad CLI, SDK ja `chloros-mcp` samu funktsioone (CLI säilitab turvalisuse tagamiseks ainult sisselogimise, DAQ-E seadistamise ja otsesed DAQ-serverid): ühe käsuga avatavad kaamerad (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), joondusprofiilifailid (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indeksid (`index_catalog()`, `compute_index()`, `capture_live_index()`), PTP-grandmaster (`time_sync_status()` ja sellega seotud failid), projektikausta (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) ja retseptid (`run_project_recipe()`, CLI-i enda käivitaja). Valgusandurite jaoks: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` ja `light_sensor_reflectance()` (kahe anduri ümbritseva keskkonna ja objekti peegeldusvõime ning indeksid); kaamera jaoks: `CameraSession.status()` ja `capture_repeated()`; maatriksi puhul: `ArraySession.capability()` ja `set_member_settings()` (elemendi valgustatus, võimendus jne). Vaata [Ajutised massiivid, joondamisprofiilid ja indeksid](#transient-arrays-alignment-profiles--indices), [Projekti kaust ja retseptid](#project-folder--recipes) ning [DAQ-andurite seansid](#daq-sensor-sessions).
> - **Täiustatud LATTICE indeksitooted** (NDVI ja muud indeksid) alates `process()`-st, kõigis töötlemisrežiimides.
> - **LATTICE OCN-kaamerad: täiustatud kiirguse, peegelduse ja indeksite tooted päikesevalguses.** Pärast lainepikkuste eraldamist rakendatakse pargi korrigeerimist – reaalajas, salvestustes ja `process()`-is; RGN-kaamerad jäävad muutumatuks. Vaata [Radiomeetriline väljund](#radiometric-output-lattice-multispectral-pipeline).
> - **Iga salvestus salvestab kasutatud DAQ-näidu, nagu seda teeb ka graafiline kasutajaliides.** `CameraSession.capture()` ja `ArraySession.capture()` / `continuous()` kirjutavad `.daq` ja selle `.match.json`, kui DAQ on seotud või koondatud, sealhulgas ainult toores salvestus; versioonis 1.2.0 salvestati toores või debayered pilt ainult koos `force_daq=True`-ga. `daq=False` lülitab selle välja ja `daq_tolerance_ms=` määrab sobivuse tolerantsi; mõlemad tulemused näitavad, mis salvestati `.daq_files` ja `.daq_matches`. Salvestus ootab nüüd DAQ-i järgmist lugemist, et saaks ühendada ekspositsiooni mõlemal poolel tehtud lugemised. Vaata [DAQ-i lugemine, mille salvestus salvestab](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` sisaldab reaalajas kaadreid ja serveripoolset pidevat salvestust.** `grab()` ja `frame_stream()` tagastavad numpy-kaadrid ühest triggerst (sama lugemine kui projekti API-s `ArrayHandle.grab()` / `frame_stream()`) ning `continuous()` salvestab iga sünkroniseeritud rühma tagapõhisesse süsteemi ja tagastab [`ContinuousHandle`](#continuoushandle). Vaata [Reaalajas kaadrid oma töötlusvoos](#live-frames-in-your-own-pipeline).
> - **Massiiv `grab()` on seotud ühe triggiga** (`ArrayHandle.grab()` ja uus `ArraySession.grab()`). Versioonis 1.2.0 loeb 12 neljastkaameraga `ArrayHandle.grab()` lugemist hõlmasid kaks triggert. Nüüd sisaldab iga kaader `group_id`-i ning massiiv `frame_stream()` annab iga triggert ühe korra. Vaata [Üks triggerväärtus iga lugemise kohta](#one-trigger-per-read--group_id).
> - **Riba- ja joondatud kaadrid:** `grab()` / `frame_stream()` võtavad kaamerate ja massiivide käepidemetelt vastu `band=`, `align=`, `bit_depth=` ja `refl_max=` kaamera ja massiivi käepidemeid. Vaata [Ühe sagedusriba eksportimine](#exporting-a-single-band--band).
> - **Uued `ArraySession`-meetodid ja argumendid:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (koos `CaptureResult.group_seq` ja `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` ühendab järjestikused rühmad ja peatub, kui ketas on täis.
> - **`ArraySession.record()` salvestab, kui graafiline kasutajaliides ei ole avatud.** Kui graafilise kasutajaliidese kombineeritud vaade on suletud, renderdab tagapõhi vaate ise: massiiviNDVI või indeks, mille edastate kui `expr=`. `RecorderHandle.stop()` tekitab `ChlorosConnectError` ebaõnnestunud salvestuse puhul, mille puhul ei kirjutatud ühtegi kaadrit, ning säilitab peatamise tulemuse `.result` (`with`-bloki jätmine ei tekita viga: kontrollige `rec.result`). Vaadake [`RecorderHandle`](#recorderhandle).
> - **Uued `DAQSensorSession`-meetodid:** `enable_ptp()`, `force_disconnect()` ning DAQ-E enda seadete jaoks `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` ja `update_firmware()`. `record_stop()` tagastab `warning` ja `gaps`. Taaskäivitus, mis sulges salvestuse, nimetab selle oma vastuses või, kui taaskäivitus ebaõnnestub, `ChlorosConnectError`-is (`recording_closed`; vt [DAQ-E seadme seaded](#daq-e-device-settings)). Vaata [`DAQSensorSession`-meetodid](#daqsensorsession-methods).
> - **Ühendatud DAQ-E, mis lõpetab andmete edastamise, ühendatakse tagapõhja poolt uuesti**, säilitades selle varasemad seaded, ning sellel olev salvestus jätkub samas `.daq`-is, kusjuures vahe on failis märgitud; `record_stop()` tagastab lüngad kui `gaps`. Versioonis 1.2.0 jäi andur vaikseks, kuni see lahti ühendati ja uuesti ühendati, mis lõpetas salvestamise. Vaata CLI-viite [DAQ-E, mis lõpetab andmete edastamise](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` töötab paigaldatud SDK abil.** Wheel (paigaldajaja PyPI-d) sisaldab nüüd vajalikke projektilaadijaid; varem tekitas `open_project()` vea `ImportError`, nimetades `project_cameras` / `project_sensors`-i. `connect_all()` ühendab salvestatud kaamerad, massiivid ja valgusandurid töötava tagapõhja (valgusandurid selle DAQ-pooli kaudu) ning salvestatud seade, mida tal pole võimalik leida, on loetletud `errors` all koos põhjendusega, selle asemel et seda sõnagi ütlemata vahele jätta. Salvestatud massiiv, mis on tagapõhisüsteemis juba käivitatud (kas kasutajaliidese, või `chloros-cli project connect`-i kaudu) on ühendatud ja jäetud töötama, kus varem teatati sellest veana („Kaamera  <sn>on juba massiivis<id>”). Vaata [`ChlorosProject`-i meetodid](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` ja `process_lattice_capture()` leiavad Chloros&#x27;i Windowsis.** Need otsivad kohta, kuhu paigaldaja backend&#x27;i paigutab (vaikimisi `<install folder>\backend\`, `C:\Program Files\Chloros\backend\`, installija enda andmetest tulenevasse kohandatud kausta) ning ühenduvad juba töötava backendiga, ilma seda üldse otsimata. Versioonis 1.2.0 kuvasid need installitud arvutis veateate „Chlorosi backendit ei leitud”, kui ei antud edasi `backend_exe=`-i.
> - **Backendist, mis lõpetab töö käivitamisel, teatatakse kohe.** Kui `ChlorosLocal()` või smart-connect-kõne käivitab backendi, mis lõpetab töö enne vastamist, tekitab see mõne sekundi jooksul veateate `ChlorosBackendError` (kui tegemist on `ChlorosLocal`-ga) ja `ChlorosConnectError` (kui tegemist on smart--connect-kutsetest, koos lõpetamiskoodiga, käivitatud käsuga ja tagapõhja viimaste väljatrükitud ridadega. Versioon 1.2.0 ootas ära 60-sekundilise käivitumise ooteaja ega selgitanud põhjust. Vaata [Tagapõhja automaatne käivitamine](#backend-auto-start).
> - **Tagapõhi töötab skriptide vahel edasi.** `with ChlorosLocal(...)`-plokist väljumisel `process_folder()` / `process_lattice_capture()` naasmine ja `chloros-mcp` lõpetamine ei peata enam SDK-i poolt käivitatud tagapõhja; järgmine skript kasutab seda uuesti. See peatub `logout()`-is (välja arvatud juhul, kui Chloros-rakendus on avatud või käimas on käivitus), `shutdown_backend()`-is või iseenesest 30 minuti möödudes, kui seda ei kasuta keegi (`CHLOROS_BACKEND_IDLE_MIN` tagapõhja keskkonnas muudab seda; `0`: mitte kunagi).
> - **SDK-i poolt käivitatud tagapõhjal palutakse sulguda, seda ei lõpetata jõuga.** `shutdown_backend()` lubab backendil vabastada oma kaamerad ja valgusandurid ning lõpetada kõik salvestused; see lõpetatakse ainult juhul, kui see töötab veel 10 sekundit hiljem. Versioon 1.2.0 lõpetas selle kohe (Windowsis jõuline lõpetamine), mis võis põhjustada selle backendiga ühendatud DAQ-valgusanduri reageerimatuse. Backend, mille SDK töötamas leidis, jäetakse endiselt rahule.
> - **Linuxis ei käivita automaatne käivitamine kunagi `backend_server.py`-i praegusest kaustast.** Kui backendit ei olnud installitud, käivitas seda skript, mis käivitati kaustas, kus see asus; nüüd käivitatakse ainult installitud backend (või `ChlorosLocal` puhul see, mille `backend_exe=` nimetab).
> - **`configure(indices=...)`**lisab projekti iga eelseadistuse vastavalt kaamera sagedusaladele, sealhulgas uue projekti, ning annab hoiatuse (`UserWarning`), kui nime ei ole võimalik lisada.**`import_images()`** näitab `.daq` salvestusi kui `scan_count` ja ei hoiata enam LATTICE salvestuskausta kohta.
> - **`read_image_audit_tags()` töötab pip-installist.** See loeb pildi metaandmeid ise; versioonis 1.2.0 tagastas see `{}` ilma töölaua installi sisemiste komponentideta Pythoni teekonnas.
> - **`capture()` vahetult pärast `connect_array()`** ootab kuni 20 sekundit massiivi esimest kaadrirühma, selle asemel et anda veateadet HTTP-koodiga 503 „Massiiv soojeneb veel“.
> - **`status()` teatab takerdumisest.** Uus väli `stalled` on väärtusega „True“, kui kaadrite kogumise tsükkel töötab, kuid midagi ei avaldata, ning `actual_fps` on sel juhul 0. Versioonis 1.2.0 säilitas `actual_fps` oma viimase väärtuse seisaku ajal, mistõttu seisakus olev massiiv näitas väärtust `actual_fps > 0`, samal ajal kui pildistamine ebaõnnestus veaga 503 „No fresh synced frame group“.
> - **Vähem puudulikke kaadrikogumeid ja salvestamisvigu 503:** iga orjakaamera päästiku sisend on nüüd glitšidest vabastatud.
> - **Pakitud 12-bitine andmeedastus** (`connect_array(packed="auto")`) suurendab koguanduri massiivi kaadrisagedust umbes kolmandiku võrra. Vaata [Pakitud 12-bitine andmeedastus](#packed-12-bit-transport-packed).
> - **Massiivi ekspositsioon ROI all:** väike ROI ei piira enam liikme ekspositsiooni anduri vabajooksuperioodi pikkusega (4,9 ms resolutsioonil 768 × 304). Vaata [Ekspositsioon massiivi liikmetel](#exposure-on-array-members).
> - **Automaatne valgustamine ei vähenda enam maatriksi automaatset kaadrisagedust.** Kaabli kaudu sünkroniseeritud massiiv töötab kiirusel, mida võimaldavad võrk ja kaamerate väljalugemine; automaatne valgustusaeg täidab selle kiiruse poolt antud perioodi ja suurendab seejärel võimendust. Käsitsi valgustusaeg, mis on pikem kui antud periood, aeglustab päästikut, et see sobiks, selle asemel et seda keeldutaks. Vaata [Valgustusaeg massiivi elementidel](#exposure-on-array-members).
> - **`read_nodes()`**lisab `frame_values` ja `notes` automaatse särituse töötamise ajal (vt [`read_nodes()` automaatse särituse korral](#read_nodes-under-auto-exposure--frame_values-and-notes)). Kui kaamera lükkab ekspositsiooni või võimenduse tagasi, näitab**`set_settings()`** vahemiku, mida kaamera aktsepteerib, ja taastab automaatrežiimi, mille ta selle väärtuse puhul välja lülitas; `set_settings(target_brightness_auto=True)` on uus.
> - **`orient=`** koos `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` ja `ArrayHandle.capture()` / `capture_smart()` pöörab tooteid vastavalt kaamera suunale. Vaata [`ArraySession` meetodid](#arraysession-methods).
> - **`DLS()` vajab anduri enda `mac_address`-i.** Vaikimisi MAC-aadressi enam ei ole (see oli ühe seadme aadress); `DLS()` ilma selleta tekitab `DLSError`. Vaata [DAQ-M sagedusribade amplituudid](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Windowsi paigaldaja** pakub valikuid „Luba Windowsi pikad teed” ja „Jäta Chlorosi tagapõhi Microsoft Defenderi skaneerimisest välja”, mõlemad on vaikimisi märgitud. Vaata [1. samm](#step-1--install-the-chloros-platform-package).
> - **Linuxi .deb-pakett** installib süsteemi jaoks Pythoni SDK-i `python3` (varem paigutati kettale vaid wheel-fail), suurendab tuuma pistikupesa vastuvõtupuhvreid, mida kaameravoogudele vaja on, ning lisab tagapõhiteenuse, mis töötab teie kasutajana. Jetsoni pakett on `chloros_1.3.0_arm64_jp6.deb`, mis on mõeldud JetPack 6-le, koos JetPacki CUDA PyTorchiga. Vaata [1. samm](#step-1--install-the-chloros-platform-package) ja [2. samm](#linux-deb).

---

## Arhitektuur 60 sekundiga

SDK on õhuke Python-kiht Chloros-tagapõhja peal (sama Flask-server, mida kasutavad ka töölaua graafiline kasutajaliides ja CLI). Automatiseerimiseks impordite `chloros_sdk` ja kutsute välja kõrgetasemelisi meetodeid; sisuliselt muutub iga kutsumine HTTP-päringuks kohalikule backendile pordil 5000 — `http://127.0.0.1:5000/api/...` (tahtlikult mitte `localhost`, mis Windowsis suunatakse esmalt `::1`-ile ja maksab ~2 s päringu kohta, kui backend toetab ainult IPv4-i). Tagapõhja haldab riistvaraparki – kaamerad, DAQ-andurid, joondamisprofiilid, kaadripuhvrid –, mistõttu saavad SDK-skriptid eksisteerida koos graafilise kasutajaliidesega, ilma et nad peaksid võitlema jadaportide või võrgukaardi ribalaiuse pärast.

Kasutada on kolm liidest:

1. **`ChlorosLocal` + vabad funktsioonid** (`process_folder`, `process_lattice_capture`) – pilditöötluse töötlusketi. Kogu kausta kalibreerimine, debayering ja indeksite eksportimine ühe Python-kõne abil.
2. **Smart-connect-käepidemed** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Avage püsiv tagapõhja-seanss reaalajas töötava riistvara jaoks. Sama „smart-prep” voog nagu graafilises kasutajaliideses: võrgu proov, taseme automaatne valik, PTP, AE algväärtustamine, GPIO-trigeri konfiguratsioon.
3. **`ChlorosProject` / `open_project`** — Laadige salvestatud projekt (kaust, mis sisaldab `cameras.json` + `sensors.json` + `project.json`), ühendage selle kaamerad, massiivid ja valgusandurid korraga ning juhtige salvestamist nimeliste käepidemete abil.

Pinnad 1 ja 2 **käivitavad automaatselt kohaliku taustaprogrammi**, kui see veel ei ole kuulamisrežiimis (sama komplektis olev binaar, mida GUI ja CLI käivitavad) — seega töötab lihtne skript värskest shellist, ilma et peaksite taustaprogrammi esmalt käivitama: `ChlorosLocal` (ja `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` ja `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` ja `analyze_array_network()` ei käivitu kunagi: need vajavad töötavat tagapõhja. Edasta `auto_start_backend=False`, et sellest loobuda (nt kui viidatakse kaug-backendile, mida kunagi ei käivitatagi). Vaata [Backendi automaatne käivitamine](#backend-auto-start). Surface 3 käitub teisiti: `open_project()` ei võta vastu `auto_start_backend` parameetrit ning `connect_all()` ei käivita kunagi backendit — ta proovib `http://127.0.0.1:5000`-i üks kord ja kui keegi ei vasta, lülitub ta tagasi otsesele (tagapõhja-vabale) `lattice_sdk` seadme juhtimisele. Selleks on vaja Arena SDK-i käitamiskeskkonda ja OpenCV (`cv2`) importimist teie Pythonisse, mida SDK-i wheel ei installi; ilma nendeta kuvatakse iga salvestatud kaamera ja massiiv veana koos põhjendusega. Projekti valgussensoritel puudub backendita lahendus: kui backend ei tööta, kuvatakse iga salvestatud sensor veana koos põhjendusega (vt [`SensorHandle`](#sensorhandle)). Ainult `proj.process()` ja `stream(..., overlays=True)` loovad viivitusega `ChlorosLocal()`-i (mis käivitub automaatselt).

Kõik kolm on auth-piiranguga: käivita `chloros-cli login` masinal üks kord või logi sisse töölaua graafilise kasutajaliidese kaudu. See, millise vea tekitab kehtiva sessiooni puudumine, sõltub kasutajaliidesest (vt [Erandid](#exceptions)): `ChlorosAuthenticationError` tekib `ChlorosLocal`-ist (ja `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` smart-connect-kutsetest ja nende sessioonidest, `status: "error"` koos `error_code`-iga, mis pärineb `analyze_array_network()`-ist, ning kirje `errors` all `connect_all()` aruandest.

Nõuded:
- Python 3.7+ (vastavalt paketi kirjeldusele; arendatud/testitud versioonil 3.10)
- Kohalikult installitud Chloros Desktop (tagapõhja binaar on paigaldaja sees)
- Aktiivne Chloros+ sisselogimine **mis tahes tasulisel Chloros+ tasemel (Copper või kõrgem)**; tasuta**Iron**tasemel puudub juurdepääs SDK-le/CLI-le. Seda rakendatakse**serveripoolel**: iga SDK- või CLI-märgistusega päring peab sisaldama nii aktiivset sessiooni kui ka tasulist paketti. Copperist madalamal tasemel pakett saab veakoodi `403` koos veakoodiga `error_code: PLAN_UPGRADE_REQUIRED` (veakood `ChlorosLicenseError` pärineb `ChlorosLocal`, `ChlorosConnectError` smart-connect-kõnedest). Välja logitud helistajale antakse selle asemel `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` koodidest `ChlorosLocal` ja `ChlorosConnectError`, mis pärinevad smart-connect-kõnedest) — need kaks on erinevad, sest koodi `chloros-cli login` uuesti käivitamine parandab vea `401`, kuid ei suuda parandada viga `403`, mille lahendamiseks on vaja plaani uuendamist.
- Offline-kasutamine on toetatud paketi armuaja jooksul: tasand loetakse serveri valideerimise vahemälust (5 minutit) või allkirjastatud, seadmega seotud litsentsi vahemälust (kuupakettide puhul 30 päeva, aastapakettide puhul kuni tellimuse lõppemiseni). Kui see ajapikendus lõpeb, muutub pakett tasuta versiooniks ja SDK/CLI-juurdepääs peatatakse, kuni seade suudab serveriga ühendust luua. Viga `chloros-cli status` (`GET /api/license-status`) jääb tasuta tasemel kättesaadavaks, et viga oleks nähtav.
- Windows 10/11 64-bitine, **Ubuntu 22.04 LTS või uuem**või Jetson (JetPack 6). Ubuntu 20.04**ei** ole toetatud: `.deb` sõltuvused tulenevad sellest, millega tagapõhi on seotud, sealhulgas `libc6 (>= 2.35)`, ning Focal tarnib glibc 2.31.

---

## Paigaldamine

Pythoni SDK on õhuke Pythoni kiht Chlorosi backend&#x27;i peal. Kui te ei kasuta backend&#x27;i teisel masinal (vt [Kaug-backend&#x27;i režiim](#remote-backend-mode-pip-only-host-via-tunnel)), peab teil olema **Chlorosi töölaua pakett kohalikult installitud** (Windowsi paigaldaja või Linuxi `.deb`), mis sisaldab DAQ valgussensoreid. Pakett sisaldab backend-binaarfaili, Arena SDK-i käivituskeskkonda LATTICE-kaamerate jaoks ning kalibreerimispakette.

Viimased allalaadimised: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### 1. samm — Chlorosi platvormipaketi installimine

#### Windows (.exe)

1. Laadige alla `Chloros Installer 1.3.0.exe` allalaadimislehelt.
2. Käivita paigaldaja ja järgi paigaldusviisardi juhiseid. Vaikimisi paigalduskataloog on `C:\Program Files\Chloros\`.
3. Käivita Chloros vähemalt üks kord ja logi sisse oma Chloros+ kontoga.

Paigaldaja lubab tagapõhja (`chloros-backend.exe`, mitte midagi muud) läbi Windowsi tulemüüri igas võrguprofiilis: UDP sissetulev liiklus kaameravoogude, PTP, mDNS ja DAQ-E voo jaoks ning TCP sissetulev liiklus ainult kohalikust alamvõrgust, et DAQ-E saaks püsivara uuendamise ajal arvutiga ühendust luua ([`update_firmware()`](#daq-e-device-settings)). Eemaldaja eemaldab mõlemad reeglid. Backendi HTTP-API jääb igal juhul aadressile `127.0.0.1`.

Viisardi **Süsteemi seaded** rühmas on kaks valikut, mõlemad on vaikimisi märgitud:

| Valik | Mida see teeb | Vaikimisi paigaldamise tühistamine |
| --- | --- | --- |
| **Luba Windowsi pikad teed** | Võimaldab projektikausta asuda sügavamal kui Windowsi 260-tähemärgiline teepiirang. Kui pikad teed on välja lülitatud (Windowsi vaikimisi seade), lükatakse toodete jaoks liiga sügav projektikausta tagasi enne, kui midagi töödeldakse — vt [Vigade kõrvaldamine](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Jäta Chloros-tagapõhi Microsoft Defenderi skaneerimisest välja** | Peatab Defenderi reaalajas-aegset skannimist iga faili puhul, mille kaamera salvestab. Suure salvestussageduse korral võtab skannimine aega umbes 30 % salvestussagedusest: mõõdetuna ROI 512 juures 9-kaameralisel massiivil töötas pidev RAW-salvestus skannimisega kiirusel 52,7 kaadrit sekundis ja backendist väljajätmisel kiirusel 77,7 kaadrit sekundis. | `/MERGETASKS="!defenderexclusion"` |

Pidev salvestamine näitab Defenderi koormust kui `defender_cores` (vt [`ContinuousHandle`](#continuoushandle)).

#### Linuxi amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Paigaldage `apt install ./…`, mitte `dpkg -i`: apt paigaldab ka Pythoni SDK nõuded, mida pakett soovitab, kuid ei nõua (vt [2. samm](#linux-deb)).

#### Linuxi arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Jetson Orin / Orin NX / Orin Nano jaoks JetPack 6-l (L4T R36). Selle PyTorch on JetPack 6 CUDA-koostis, seega toimub GPU-töötlus Orini GPU-l; see kasutab CUDA ja cuDNN raamatukogusid, mille JetPack 6 paigaldab (`sudo apt install nvidia-jetpack` pildil, millel need puuduvad).

#### Linux: vastuvõtupuhvrid ja tagapõhiteenus

Mõlemad .deb-failid suurendavad tuuma sokli vastuvõtupuhvreid, mida LATTICE-kaamerate voogudele on vaja, 32 MB-ni (`/etc/sysctl.d/60-chloros-gige.conf`). Tuuma vaikimisi seadistuse korral kaamera paketid kukuvad tuumas ära ja massiiv kaotab kaadreid ning kaadrisagedust; `chloros-cli lattice network` annab teada, kui puhvrid on peaaegu täis. Vaata CLI-viite jaotist [Linux-hostid — pistikupesa vastuvõtupuhvrid](cli-reference.md#linux-hosts--socket-receive-buffers).

Peata masinal käivita backend teenusena **oma kasutajana**, et skript leiaks töötava backendi (`connect_all()` ei käivita kunagi backendi):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Seejärel töötab backend teie nimel: selle poolt salvestatud andmed kuuluvad teile ja see kasutab teie `chloros-cli login`-i. Root-kasutaja `chloros-backend.service` on endiselt paigaldatud masinatele, kus see on lubatud, kuid iga fail, mille root-backend kirjutab, kuulub root-kasutajale. Vaata CLI-viite jaotist [Tagapõhja käitamine teenusena](cli-reference.md#running-the-backend-as-a-service-headless).

Teenuse poolt käitatav tagapõhi on jagatud ja klient ei peata seda kunagi: SDK-skriptid ja `chloros-cli` kasutavad seda sellisena, nagu nad selle leiavad (`ChlorosLocal.shutdown_backend()` peatab ainult backend&#x27;i, mille `ChlorosLocal` ise käivitas), ning `GET /api/status` nimetab üksuse `service`-iks. Käivita see uuesti `sudo systemctl restart chloros-backend@$USER.service`-iga. Paketi uuendamine peatab tagapõhiteenused installimise ajaks, seega kaotab töötav skript oma kaamerad ja andurid ning käivitab seejärel kõik aktiveeritud seadmed uuesti; paketi eemaldamine peatab need ja lülitab aktiveeritud seadmed välja. Korraga töötab ainult üks backend, seega aktiveerige üks seade: teine keeldub käivitumast ja jääb seisundisse `failed` (lõpetamisstaatus 98, ei taaskäivitu) ning selle `systemctl status` näitab põhjust. Üksuste vahetamiseks lülitage välja see, mida te ei kasuta.</id></sn>

### 2. samm — Pythoni SDK paigaldamine

**Chlorosi paigaldaja sisaldab sobivat SDK-wheel-faili.** Iga Windowsi paigaldaja ja Linuxi .deb-pakett salvestab kettale faili `chloros_sdk-X.Y.Z-py3-none-any.whl`, mis vastab täpselt GUI / CLI / backend-versioonile. Teil ei ole vaja PyPI-st versioone otsida, et versioonid oleksid sünkroonis.

#### Windows

Paigaldaja käivitab automaatselt faili `pip install` koos kaasasoleva wheel-failiga, kasutades teie süsteemi Pythonit (eelistatakse käivitajat `py.exe`, tagavaraks on `python -m pip`). Tegevust ei ole vaja – `import chloros_sdk` töötab teie Python-keskkonnas pärast edukat paigaldamist. Kui arvutis pole Pythonit, jätab paigaldaja selle sammu märkamatult vahele ning GUI ja CLI töötavad edasi.

#### Linux (.deb)

Mõlemad .deb-failid (x86_64 ja Jetson) installivad SDK süsteemile `python3`, mistõttu `import chloros_sdk` töötab pärast installimist ilma pip-ita ja võrguta. Paketid (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` ja projektilaadijad) paigaldatakse `/usr/lib/python3/dist-packages`-i, ja SDK-i käsk `chloros-mcp` ([MCP-server](#mcp-server-ai-assistants)) paigutatakse kataloogi `/usr/bin`; need kõik kuuluvad paketti: uuendamine asendab need ja paketi eemaldamine eemaldab need.

SDK enda nõuded, `python3-numpy` ja `python3-requests`, on **soovitatavad, mitte kohustuslikud** (graafiline kasutajaliides, CLI ja tagapõhi on iseseisvad): `sudo apt install ./chloros_<version>_amd64.deb` (Jetsonil `…_arm64_jp6.deb`) installib need, `dpkg -i` aga mitte. Paigaldamise väljund näitab, millise SDK-versiooni `python3` impordib, millised nõuded puuduvad, ning `chloros_sdk`-i, mis asub `python3`-i teekonnas mujal ja varjutab paketi koopia (näiteks varem `pip install --user`-iga paigaldatud versiooni).

Virtualenv-keskkonnas, mis ei näe süsteemi pakette, installige komplektis olev wheel sinna:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Air-gapped Jetsoni kasutuselevõtude puhul toimub see täielikult offline-režiimis — SDK ja wheel on juba kettal.

#### Avalik PyPI

Ainult pip-i kasutavate hostide puhul (Chlorosi töölaua pakett pole installitud; SDK juhib sel juhul teisel masinal asuvat backendit, vt [Kaug-backend-režiim](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI-s on saadaval uusim stabiilne versioon. Hoidke SDK-i versioon sama, mis selle poolt juhitava Chloros-i tagapõhja versioon (vt [Versioonid ja ühilduvus](#versioning--compatibility)). DAQ-M-abiprogramm `DLS` vajab samuti `bleak`: `pip install "chloros-sdk[camera]"` (või `pip install bleak`).

#### Kontrolli

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Vajalik on Chloros+ tellimus.** Kõik SDK-kutsed nõuavad aktiivset Chloros+ sisselogimist. Käivita `chloros-cli login user@example.com 'YourPassword'` üks kord iga masina kohta; autentimisandmed salvestatakse vahemällu `~/.chloros/`.

### Kas mul on vaja töölaua paketti?

Enamiku töövoogude jaoks **ei** piisa ainult pip-paketist. Siin on loetletud, mida iga SDK-liides vajab:

| SDK-liides | Kas vajab töölaua paketti? | Miks |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Jah**(selle käivitamiseks)**/ Ei** (tagapõhi on juba kättesaadav) | Kasutada töötavat tagapõhja (töölaua rakenduse oma või mõne teise käivitatud skripti oma); muidu käivitada automaatselt installitud tagapõhi: `/usr/lib/chloros/chloros-backend` (Linux) või `<install folder>\backend\chloros-backend.exe` (Windows; vaikimisi `C:\Program Files\Chloros\backend\`, kohandatud installikausta loetakse installija andmetest). Edasta `backend_exe=`, et anda sellele ise nimi. |
| `chloros-mcp` ([MCP-server](#mcp-server-ai-assistants)) | **Jah**(käivitamiseks)**/ Ei** (tagapõhi on juba kättesaadav) | Põhineb `ChlorosLocal`-il, seega sama mis eelmises reas. |
| `read_image_audit_tags` | **Ei** | Loeb töödeldud pildi enda metaandmeid (ainult Pythoni standardteek). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Jah**(kohalik)**/ Ei**(kaugjuurdepääs) | Puhtad HTTP-kliendid tagapõhja kaudu. Kohalik tagapõhi → vajalik töölaua pakett. Kaugjuurdepääsuga tagapõhi → `backend_url=`**tunneli kaudu** (vt kaugjuurdepääsu režiim — kaasasolevad tagapõhjad seovad ainult loopbacki). |
| `ChlorosProject` / `open_project` | **Jah** | Käitab salvestatud projekte tagapõhja kaudu. |
| Otsesed LATTICE-klassid (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Jah** | Vajab Arena SDK-i käivituskeskkonda, mis on kaasas töölaua paketis, ning OpenCV-d (`cv2`) teie Python-is, mida ei installeeri ei töölaua pakett ega SDK (`pip install opencv-python`). Muidu on `CAMERA_AVAILABLE` importimisel sama mis `False`. `DLS` vajab ka `bleak`-i. |

### Kaug-tagapõhja režiim (ainult pip-host, tunneli kaudu)

> **Paigaldatud tagapõhja ei ole võimalik LAN-võrgu kaudu kätte saada.** See kuulab
> ainult loopback-porti (`127.0.0.1` / `::1`), seega `backend_url="http://<lan-ip>:5000"`
> **ei saa ühendust paigaldatud Chlorosiga**. Et juhtida backendit teisel
> masinal, suuna selle loopback-port ise edasi ja suuna SDK
> tunneli poole:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Headless / CI / robotika-hostid võivad hoida ühte masinat täieliku töölaua installatsiooniga „Chloros-serverina” ja `pip install chloros-sdk` kõikjal mujal — kuid nende vaheline ühendus on eespool nimetatud kasutaja poolt korraldatud tunnel, mitte otsene LAN-URL-aadress.

> **`ChlorosLocal` ainult pip-i kasutaval hostil.** `ChlorosLocal(backend_url=BACKEND)` ühendub tagapõhja, milleni ta pääseb ilma töölaua installita: ta otsib kohalikku tagapõhja binaarfaili ainult siis, kui peab selle KÄIVITAMA (kui keegi ei vasta ja `auto_start_backend=True`). Versioonini 1.2.0 otsis ta esmalt binaarfaili ja tekitas veateate `ChlorosBackendError` („Chlorosi tagapõhja ei leitud…”) pip-only-hostil isegi siis, kui tagapõhi oli kättesaadav. Kaustad, mille sa edastad (`process_folder`, `import_images`) kontrollitakse selles masinas ja backend loeb neid, seega peavad need kaugbackendi korral olema mõlemal masinal sama nimega, näiteks jagatud kettal.

### Ainult DAQ-töövoog (ainult pip-host)

`pip install chloros-sdk` üksi ei suuda DAQ-andurit juhtida: DAQ-andurit juhitakse alati Chlorosi backend&#x27;i kaudu, kasutades `connect_daq_sensor()`-i:

- **Andur sellel masinal:** paigaldage siia Chlorosi töölaua pakett. `connect_daq_sensor()` käivitab oma backend&#x27;i, kui see veel ei tööta.
- **Andur teisel masinal:** sellele masinale on vaja töölaua paketti; pääsege selle backendini tunneli kaudu [kaug-backend-režiimis](#remote-backend-mode-pip-only-host-via-tunnel). Backend avab anduri oma masinal, seega peab DAQ-U olema ühendatud selle arvutiga, DAQ-M asuma selle Bluetoothi ulatuses ja DAQ-E olema võrgus, millele backend pääseb ligi.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Mõlemal juhul rakendab backend „Chloros+“ plaani: logi sisse backendit käitaval masinal (`chloros-cli login` või töölaua rakendus) mis tahes tasulise tasemega (Copper või kõrgem). Välja logitud backend ja „Copper“-ist madalam pakett tekitavad mõlemad veateate `ChlorosConnectError`, mis sisaldab backendi põhjust (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Kiirjuhend

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Ülemise taseme API-indeks

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Pilditöötlus — `ChlorosLocal`

Peamine töötlusklass. Käivitab tagapõhja selle loomisel (kui ükski ei vasta ja `auto_start_backend=True`), loob / konfigureerib projekte, jälgib edenemist, tagastab töötlemisejärgsed kokkuvõtted.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `create_project(project_name, camera=None)` | Uue projekti loomine (valikuliselt koos kaameramalliga, nagu `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Avab salvestatud projekti (selle kausta või projekti nime projektide kaustas) ja teeb sellest avatud projekti. Ümberpaigutatud projekti teed suunatakse ümber kausta, kus see praegu asub; `relink` nimetab kausta, kus asuvad teised puuduvad failid. Tagastab tagapõhja vastuse koos `source_files`-iga; tekitab veateate `ChlorosProjectFilesMissingError`, kui ühtegi ei leita. Vaata allpool **Ümberpaigutatud projekt**. |
| `import_images(folder_path, recursive=False)` | Impordib RAW/TIF/JPG/DNG-pilte **ja `.daq` valgusanduri salvestusi**. Tagastab `count` (pildid) ja `scan_count` (salvestised). Annab hoiatuse ainult juhul, kui kaustas pole kumbagi. |
| `export_light_sensor(daq=True, csv=True)` | Kirjutab kalibreeritud `.daq` + `.csv` iga projekti valgusanduri salvestuse kohta kausta `<project>/Light Sensor/`. Vaata [Valgusanduri salvestused](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Seadista töötlemisnupud. `indices` on eelseadistuste nimed (`NDVI`, `GNDVI`, `OSAVI`, …), millest igaüks on projektile lisatud vastava kaamera lainealade jaoks; nimi, mis ei ole eelseadistus või mille lainealasid kaamera ei suuda moodustada (`NDRE` vajab RE ja NIR lainealasid, mida ükski kaamera eraldi ei paku), saab nimeks `UserWarning` ja sellele ei lisata indeksipilti. `daq_log_path` nimetab valgusanduri salvestust (fail `.daq` või `.csv` või nende kaust), millega iga kaadrit võrreldakse nii LATTICE’is kui ka Survey3-s, mitte piltidega koos imporditud salvestustega; kui tee puudub või ei sisalda salvestust, kasutatakse imporditud salvestusi. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Käivita töötlusketi. Tagastab `{"status": "complete", "async": False}`, lisaks võtme `summary`, kui tagapõhi seda pakub — vt [Käivitusejärgne kokkuvõte ja näpunäited](#post-run-summary--hints) — `export_check_failure`, kui käivituse eksporditulemusi ei olnud võimalik kontrollida, ja `image_failure`, kui kadus pilt. `wait=False` tagastab korraga `{"status": "started", "async": True}` korraga. |
| `get_config()` / `get_status()` / `status()` | Kontrolli tagapõhja seisundit. |
| `logout()` | Väljumine: tühistab vahemällu salvestatud autentimisandmed ja vabastab selle masina seadme pesa (tulemuseks on `device_deregistration`). Käimasolevat töötlemistsüklit ei peatatata: see lõpeb selles režiimis, milles see algas (tulemuses `run_in_progress: true` ja `run_mode`), ning järgmine tsükkel kasutab standardtöötlust. |
| `shutdown_backend()` | Peata tagapõhi, kui see instants selle käivitas; juba töötavat tagapõhja jäetakse puutumata. `with`-ploki jätmine või instantsi prügikogumise käivitamine ei peata seda: tagapõhi jätkab töötamist järgmise skripti jaoks ja peatab end ise 30 minuti pärast, kui seda keegi ei kasuta. Tagapõhjal palutakse sulguda, seega vabastab ta esmalt oma kaamerad ja valgusandurid ning lõpetab kõik salvestused; see lõpetatakse ainult siis, kui see töötab veel 10 sekundit hiljem. |
| `discover_cameras()` | Leia LATTICE-kaamerad **selle instantsi backend&#x27;i kaudu** (`/api/camera/discover`). Tagastab sõnastikute loendi (`serial`, `model`, `ip`, …) – sama kujuga, nagu seda näevad GUI ja CLI. Tühi nimekiri, kui ühtegi ei leitud või backend ei ole kättesaadav. |
| `camera_capture(output_dir, format="tiff", **settings)` | Salvestab ühe kaadri**tagapõhja kaudu**(käivitatakse automaatselt selle käepideme poolt), nii et see saab sama ettevalmistuse kui GUI/CLI (vaikimisi 12-bitine, mälupooli taaskasutus, sisseehitatud kalibreerimismetadandid). Määrake sihtmärk kindlaks käsuga `serial=` või `device_index=`; edasta `exposure`/`gain`/`pixel_format`/`preset` kui `**settings`. Tagastab vana metaandmete sõnastiku (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genereerib ühendatud kaamerast ülekattega koostatud eelvaatekaadreid — kerge MJPEG-klient tagapõhja `/api/camera/<serial>/stream-annotated` marsruudi kaudu (sebra / võrgustik / sihtjoon / histogramm / tippude esiletõstmine / serveripoolne märgi joonistamine). `decode=True` genereerib BGR-massiive ja vajab OpenCV-d (`cv2`), mida SDK ei installeeri; `False` genereerib töötlemata JPEG-baitid. Samuti kättesaadav projektipõhiselt kui `ChlorosProject.stream(overlays=True)`. |

**Käimasolev töö säilitab oma projekti.** Kui töötlemisprotsess on käimas – olgu see siis käesoleva skripti, rakenduse, `chloros-cli process` või `chloros-mcp` oma –, keeldub tagapõhi projekti loomast, avamast, sinna importimast või selle töötlemisseadeid muutmast (HTTP 409 `PROCESSING_IN_PROGRESS`): `create_project()` ja `import_images()` tekitavad vea `ChlorosProcessingError`, ning `configure()` tekitab veakoodi `ChlorosConfigurationError`, ning `process()`, mille režiim erineb käimasolevast, tekitab veakoodi `ChlorosProcessingError` (käivitus säilitab režiimi, milles ta algas), igaüks koos tagapõhja sõnumiga. Projektisse salvestatud salvestus liidetakse sellega siis, kui käivitus lõpeb (seda näitab selle vastuse `project_add`). Oota, kuni käivitus lõpeb (`get_status()`), ja jätka seejärel.**Ümberpaigutatud projekt.** Projekti `project.json` nimetab iga allikafaili (iga pildi ja valgusanduri salvestuse) selle täieliku tee järgi, seega projekti kausta – või kogu projektide kausta – teisele kettale teisaldamine jätab need endiselt vana asukohta viitama. Iga avamine parandab ise, mida suudab: projekti kausta sees olnud faili otsitakse kaustast, kus projekt praegu asub, ja iga seal leitud tee suunatakse ümber ning salvestatakse (`project.json` salvestatakse esmalt varukoopiana nimega `project.json.before-relink-<date>-<time>`). Teekond suunatakse ümber ainult sellele failile, mis seal tegelikult asub. Failid, mis asusid väljaspool projekti kausta – näiteks hub-seanss või „Lisa kaust” impordi tulemus – leitakse ainult `relink=<the folder they are in now>`-i (või selle kohal asuva mis tahes kausta) abil. `open_project()` tagastab `source_files`: `total`, `found`, `missing`, `missing_roots` (kaustad, kus puuduvad failid asusid), `missing_outside_project` ja `relink` (mida suunati ümber, kust kuhu). Projekt, millest mõned failid puuduvad, avatakse koos ülejäänutega; projekti, millest ühtegi faili ei leitud, ei avata, ning `open_project()` tekitab vea `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). CLI-i vead `project open --relink` ja `chloros-mcp``open_project` (`relink_folder`) teevad sama.

Kasutage kontekstihaldurina, et tagada puhastamine:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Valgusanduri salvestused — kalibreeritud `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E andmeid saab salvestada **ilma** kalibreerimispaketita. Just seda
teevad avalikud [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
salvestid (`record_daq.py`) vaikimisi teevad: nad kirjutavad salvestusse andurite toorandmed ja märgistavad
faili nii, et Chloros laadib selle anduri tehase kalibreerimise **seerianumbri järgi** — kõigepealt kohalikust vahemälust
, seejärel MAPIRi pilvest — ja rakendab seda importimisel.

Chloros salvestab tulemuse tagasi kahe tootena iga salvestuse kohta, nimetusega
`<project>/Light Sensor/`:

| Toode | Mis see on |
| --- | --- |
| `<name>_calibrated.daq` | Ümbertöödeldav arhiiv — sama skeem nagu reaalajasalvestusel, kuid nüüd on märgitud selle loonud pakett. Selle uuesti importimine **ei** kalibreeri seda teist korda. |
| `<name>_calibrated.csv` | Spektraalne kiirgustihedus ühikutes W/m²/nm anduri enda lainepikkuste võrgustikus, üks rida iga mõõtmistulemuse kohta, lisaks fotomeetrilised veerud (koguvõimsus, fotopiline/skotopiline luks, PPFD ja selle sinine/roheline/punane jaotus, tipp-lainepikkus). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Ainult pakettideta andurid (DAQ-A).** Anduri toores spektraalne loend – *mitte* kiirgustihedus. Vaata allpool. |

`<name>` on salvestuse enda failinimi. Kaks erinevates kaustades asuvat salvestust võivad jagada
üht — iga kaadri kohta mõõdetud väärtust (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), mis on salvestatud
samal millisekundil kahe kaamera poolt — ning igaüks saab ikkagi oma tulemused: esimene säilitab
`<name>`, järgmine lisab nime **järgi** kaustad, mis seda eristavad
(`<name>__<capture>_daq_calibrated.daq`), nii et salvestuse kuupäev ja kellaaeg on endiselt esikohal.

`process()` teostab seda eksporti ühe oma etapina. See **ei** vaja pilte:
üksinda lennutatud valgussensor on esmaklassiline töövoog ning sellisel projektil pole
konstruktsiooni tõttu ühtegi pilti.

**DAQ-A salvestised eksporditakse toorloenditena.** DAQ-A-seeria on vanem kui seerianumbri-põhine
pakettide süsteem ja sellel pole paketti, mida alla laadida — seda kalibreeritakse hoopis välitingimustes
peegeldusmäära sihtmärgi abil, mistõttu pole seda kunagi vaja olnud. Need salvestised eksporditakse
tüübiga `_raw`, mitte `_calibrated`: tegemist on erineva failinimega, mitte lipuga
faili sees, sest nimetus peab säilima ka siis, kui seda edastatakse e-postiga pelgalt nimena.
`.csv` pealkiri näitab `raw spectral sensor counts (NOT irradiance)` ja hoiatab, et
väärtused on võrreldavad **faili sees** – just seda kasutabki sihtmärgipõhine kalibreerimine
neid kasutab — mitte aga andurite vahel. Võimsusest sõltuvad fotomeetrilised veerud (koguvõimsus,
fotopiline/skotopiline luks, PPFD) annavad tulemuseks **NULL**, mitte loendustest integreeritud väärtused.

DAQ-U / DAQ-M / DAQ-E, mille paketti lihtsalt ei õnnestunud alla laadida, **jäetakse ikkagi vahele**,
mitte kirjutatakse toorandmetena: sel juhul pakett on olemas ja „ühenda uuesti ja töötle uuesti” on tõeline nõuanne.

Vana versiooni **v1.01 / v1.02** salvestused (neid kirjutab DAQ-A-SD) ei sisalda lugemipõhist ajamärki,
vaid ainult faili kirjutamise aja. Pildi↔allapoole suunatud valguse sobitaja lükkab need endiselt tagasi —
kaadri sobitamine kirjutamisajaga oleks märkamatult vale — kuid eksportija loeb neid ning
failis CSV kuvatakse `clock=daq_created_on`, nii et toode näitab, millisel kellal see põhineb.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Salvestus, mille kalibreerimispaketti ei ole võimalik alla laadida (offline-režiimis või andur, mille
kalibreerimisandmed puuduvad failist), kuvatakse `skipped` all **koos põhjendusega**. Seda ei
kirjutata kunagi välja „kalibreeritud” failina, mis sisaldab töötlemata loendusi – ühenduge internetiga ja
käivitage uuesti, ning eksport lõpeb.

### Edusammude tagasiside

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Järeldokument ja näpunäited

Lõpetamisel laadib `process()` alla `GET /api/processing-summary` ja lisab selle sisu failina `result["summary"]`. Allalaadimine toimub parima püüdluse põhimõttel ega takista kunagi edukat tagastamist — kui kokkuvõte pole kättesaadav, kasutab `process()` tagavarana lihtsat `{"status": "complete", "async": False}`-vormi. Iga kirje `summary["hints"]`-is — täielikud laused soovitatud parandusmeetmetega, nt miks käivitus andis nullväljundi — edastatakse samuti uuesti Pythoni `UserWarning`-na, nii et nullväljundiga käivitused on isediagnostiseerivad isegi siis, kui sa sõnastikku kunagi ei vaata:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` on masinloetav pool:

| Võti | Mida see loendab |
| --- | --- |
| `models` | Käivituse kaamerarühmad. |
| `images_in_groups` | Nende rühmade allikapildid. |
| `targets_found` | Avastatud peegeldusmäärade sihtmärgid. |
| `images_calibrated` | Pildid, mille jaoks töötsükkel kirjutas peegeldusvõime toote. |
| `images_failed` | Ebaõnnestunud pildid, mida ei eksporditud. |
| `exported_files` | **Töötsükli poolt kirjutatud pilditootefailid**, mille jooks salvestas**: iga toode (debayered, eelvaade, kiirgus, peegeldusvõime, indeks, sihtmärgi renderdused) igas formaadis. |
| `reflectance_files` | Nende hulgas olevad peegeldusvõime tooted (`Reflectance_Calibrated_Images`), olenemata sellest, kas need kalibreeriti sihtmärgi või valgusanduri (DAQ) kiirgustiheduse abil. Kui käivituses puudus sihtmärk, kuid peegeldusvõime andmed siiski salvestati, märgitakse seda vihjes; ainult käivituse puhul, kus neid andmeid ei salvestatud, teatatakse, et peegeldusvõime väljundid puuduvad. |
| `clamp_masks_written` | [Unmix-klambri maskid](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) – need on kaasandmed, mitte tulemused, seega ei ole need kirjas `exported_files`-is (CLI loendab neid eraldi samamoodi). |
| `captures_left_out` | Projekti `captures`-kaustas olevad töötlemata kaadrid, mida käesolev käivitus ei salvestanud. Käivitus ootab alguses eelmise salvestuse valmimist, seega on see väärtus 0, kui käivituse ajal ei salvestatud ühtegi kaadrit või kui ooteaeg lõppes. `summary["captures_left_out_note"]` näitab, mida nendega teha. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Valgusanduri salvestused, mida loetakse tahtlikult eraldi — need pärinevad teisest etapist ja esinevad ka jooksudes, kus pilte üldse pole, seega nende kaasamine muudaks ainult andmekogumisega jooksu selliseks, nagu oleks see pilte eksportinud. |

Nende kõrval: `summary["output_dirs"]` (iga kataloog, kuhu on kirjutatud),
`summary["light_sensor_export"]`, `summary["stopped"]` (tõene, kui kasutaja katkestas
jooksu, nii et osalised loendid ei näita seda kui lõpetatud jooksu, mille toodang jäi alla oodatu), ja
`summary["groups"]` (jaotus rühmade kaupa).

`exported_files` salvestatakse torujuhtme poolt **kirjutamise ajal**, mitte skannitakse hiljem
projekti pildiobjektidest välja. Paralleel- ja GPU-strateegiad loovad oma pildiobjektid
(GPU-teede puhul töötajate alamprotsessides), seega vana skannimine teatas
`0 file(s) written` iga sellise käivituse kohta ja väljastas seejärel null-eksportide vihje — käivituste puhul,
kus kõik oli töötanud. Kui te loote skripti selle numbri põhjal, näitab nüüd korralik paralleelkäivitus
nullist erinevat arvu.

Vahelejätmiste aruanded näitavad iga faili puhul lugeja poolt tegelikult kindlaks tehtud põhjust –
loetamatu skeem, puuduv pakett, kirjutusviga – **duplikaatidest puhastatuna**, nii et kakskümmend faili,
mis ühe põhjuse tõttu vahele jäid, kuvatakse ühe põhjusena, mitte selle kahekümne kordusena.

> **`process()` ei tekita veateadet, kui käivitamine ei tooda pilte.** See on üks kahest kohast, kus SDK ja
> CLI erinevad tahtlikult: `chloros-cli process` käsitleb olukorda „tooteid taotleti, kuid ühtegi ei
> kirjutatud” kui ebaõnnestumist ja lõpetab töötamise nullist erineva tulemusega, samas kui SDK naaseb normaalselt ja teatab
> olukorra kaudu `summary` / vihjed. Kui teie töötlusketi peaks tühja käivituse korral peatuma, kontrollige seda
> ise – uurige `summary` (või loe üle projektikausta all olevad failid), selle asemel et tugineda
> erandite puudumisele. Tavapärased põhjused on sisendkaust, mida ei tunnistatud
> salvestuskaustana, ning tooted, mis jäeti vahele, kuna need ei olnud olemasolevate kaamerate jaoks kohaldatavad (nt ainult RGB-kaamerate
> kiirgus).
>
> `process()` ei teki ka siis, kui käivituse eksporditulemusi ei olnud võimalik kontrollida: see annab hoiatuse ja seab
> `result["export_check_failure"]`. Tooted salvestatakse. Selle peatamiseks kontrollige seda võtit. Sama kehtib ka jooksu kohta, millest
> on kadunud pilt: see annab hoiatuse ja seab `result["image_failure"]`, mille korral `chloros-cli process`
> lõpetab töötamise nullist erineva väärtusega.

### Abifunktsioonid

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Iga käivitus toimub oma `with ChlorosLocal(...)`-i sees; tagapõhi, mille ta pidi käivitama, jätkab töötamist, kui ta tagasi pöördub, järgmise kutsumise jaoks, ning peatub ise 30 minuti pärast, kui seda keegi ei kasuta. Kumbki neist ei võta vastu `auto_start_backend`, `radiometric_output` ega `daq_log_path`; kasutage nende jaoks `ChlorosLocal` ja `configure()`. `process_lattice_capture()` ei võta vastu `camera`, `debayer` ega `ppk`: see jätab paneeli tuvastamise vahele ja kasutab standardseid debayeri.

### Toetatud väärtused

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiomeetriline väljund (LATTICE multispektraalne töötlusketi)

`process` töötlusketi LATTICE multispektraalne (M3C/M3M) eksporditase — `reflectance` (vaikimisi), `radiance`, `sensor-response` või `all` (iga pildi puhul vastav režiim) — määrab projekti **„Ekspordi kiirgus”**ja**„Ekspordi peegeldus”**** tooted: `radiance` ekspordib ainult kiirgust, `reflectance` ainult peegeldust, `sensor-response` ei ekspordi kumbagi (ainult debayered- ja eelvaatetooted), `all` ekspordib mõlemat. Debayered- ja eelvaate tooteid ei saa sellel tasandil muuta. `configure()`-il on selleks spetsiaalne märksõna:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Täpsem pääsetee — toodete kirjutamine läbi `custom_settings` — toimib endiselt, kuid pidage meeles, et see asendab kogu seadete ploki (vaata allpool olevat hoiatust). Kirjuta `"Export <product>"` võtmed: projekti enda `"Export radiance"` / `"Export reflectance"` seaded võidavad vanema võtme `"Radiometric output"` üle, nii et see võti üksi midagi ei muuda.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (vaikimisi) jagab kaamera kiirgust **ajamärgiga sobiva DAQ-i allapoole suunatud kiirgusega**, mis määratakse automaatselt kindlaks salvestatud `.daq` (DAQ-U/M/E)**või DAQ-M-i omast `.csv`**, mis on leitud piltide juurest; kõik kohapeal puuduvad kaamera- või DAQ-kalibreerimispaketid**laetakse esmakordsel kasutamisel automaatselt AWS-ist**. CLI kuvab seda vastavalt-tüüpi tootevalikuid `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Kelle mõõtmistulemus see on?**`.daq`-is salvestatud pilt on selle kaamera(te) mõõtmistulemus, mida see kirjeldab — nimetatud selle kõrval asuvas `<file>.daq.cameras.json`-i sidecar-failis (ühekaameralised, sünkroniseeritud ja massiivsalvestused; toores pildiseeria nimetab iga liikme salvestuse oma manifestis) — ning see sobib kokku**ainult nende kaamerate piltidega**. Seetõttu ei laena iseseisev kaamera, millel puudub oma salvestus, enam teise kaamera andmekogumist (DAQ); seostage sellega andmekogumine või salvestage üks. Andmekogumise enda salvestus (`chloros-cli daq pool-record` või GUI-s alustatud salvestus) või mis tahes `.daq` ilma selle sidecar-failita (sh vanemad salvestused) teenib endiselt kõiki kaameraid.**Päikesevalguses olevad OCN-kaamerad.** Iga OCN-kalibreerimispakett on koostatud kitsasribalise valguse tingimustes, üks LED iga riba kohta, samas kui päikesevalgus segab ribasid. Chloros 1.3.0 rakendab pärast ribade eraldamist päikesevalguse põhjal korrigeerimist, mis parandab OCN-kiirgust, peegeldust ja indeksitooteid päikesevalguses. See töötab kõikjal, kus toimub lainepikkuste eraldamine — `process()`, pildistamised, `grab()` / `frame_stream()` ja reaalaja vaade — OCN-pakettide puhul, mis seda toetavad; RGN-kaamerad jäävad muutumatuks. Varasema versiooniga salvestatud OCN-pildid saavad selle samuti, kui neid töödeldakse arvutis, milles on salvestatud nende pildistamise ajal kasutatud kalibreerimisandmed (Chloros säilitab kaamera kalibreerimisandmed pärast kaamera ühendamist); muul juhul töödeldakse neid nagu varem. Seetõttu erinevad OCN-kiirguse, peegeldusvõime ja indeksi väärtused versioonist 1.2.0.

> `custom_settings` **asendab** kogu arvutatud seadete ploki (see jätab disaini tõttu `configure()` muud märksõnad ja valideerimise vahele). Kui seda kasutate, lisage kõik teile olulised `Project Settings` märksõnad, nagu eespool toodud näites.

#### Indeksipikslite lugemine (NDVI ja muud taimestiku indeksid)

**Täisarvuline indeksraster on pildi-põhine kontrasti venitus, mitte −1…1 kaart.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` ja `JPG (8-bit)` indeksitooted — sealhulgas 8-bitine TIFF, millena 8-bitine pilt salvestatakse — salvestavad

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

kus `IndexMin` / `IndexMax` on **selle kaadri enda madalaim ja kõrgeim piiratud indeksväärtus**. DN 0 ei ole −1 ja täisskaala ei ole +1.

Chloros märgib mõlemad äärepunktid, et kaardistus jääks pööratavaks:

| Silt | Tähendus |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — allpool toodud konventsioon |
| `Chloros:IndexMin` | indeksväärtus, mis on salvestatud kui DN 0 |
| `Chloros:IndexMax` | täisskaalana salvestatud indeksväärtus |

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

> **`IndexMin == IndexMax`** tähendab, et selle kaadri iga piksel sisaldas sama indeksväärtust. Raster on täidetud keskmise skaalaga ja jagada pole millegagi — võta lõpppunkt.

> **`Chloros:Index*`-märgiste puudumine tähendab, et kaardistust pole määratud.** LUT-eksport on värvitud renderdus, mille pikslid on gradientvärvid, mis on seotud teie LUT-i läviväärtuse aknaga, ning kaadril, millel puuduvad piiratud indeksiga pikslid, pole vahemikku, mida määrata. Ärge eeldage kummagi puhul kaardistust — kehtib sama reegel kui puuduva `PixelScale` puhul.**Kuna venitus on pildipõhine, ei ole ühe lennu kahte kaadrit võimalik võrrelda DN-na**— igal kaadril on oma lõpppunktid, mistõttu sama NDVI-väärtus langeb erinevates piltides erinevatele DN-väärtustele. Kogu lennu hõlmava kvantitatiivse töö jaoks (tsoonistatistika, fikseeritud künnised, aegridad) ekspordige**`TIFF (32-bit, Percent)`**: indeksrasterid salvestatakse sinna float32-tüüpi**indeksväärtustena**, ilma ümberskaalimiseta ja märgistusteta. Vaatamata formaadi nimele ei ole indeksitooted ×100 – see skaalimine kehtib ainult täisarvulise tüüpi peegeldusvõime puhul.

#### Segunemise piiramismask — millistel pikslitel riba piiratud on

3×3 spektraalne segunemine võib viia riba negatiivseks — kui valgus sisaldab seda riba vaevu või kui stseen jääb väljapoole seda, mida kaamera segunemismatriits kirjeldab. See ei ole tegelik väärtus, seega fikseerib eksport selle väärtuseks 0 — ja reaalajas vaade näitab samuti 0. Ribale võib kehtestada piirangu mõne pikseli või peaaegu kõigi pikselite puhul; mask salvestab mõlema variandi. `Chloros:Band{R,G,B}UnmixClampFraction` näitab, kui suurele osale kaadrist see juhtus; piirangumask näitab **millised pikslid**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— `clamp_masks/`-kaustas**toote kausta kõrval**, mitte kunagi selle sees (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — iga kiirguse, peegelduse ja indeksi toote jaoks, milles vähemalt üks piksel on piiratud. Ühe lainepikkusega `uint8`, toote enda pikselvõrgustikus (suund, ROI, binnimine ja massiivi joondamise moonutus kaasa arvatud). Iga mask nimetab oma toote kaamera (`Make` / `Model` / seerianumber), salvestusaja ja objektiivi märksõnad ning loetleb lainepihud, mida selle bittid tähistavad (`XMP-Camera:BandName`, bit 0 esimesena); selle `ImageDescription` ja `Software` märgivad seda kui fikseeritud maski ning see ei sisalda kalibreerimist.
- **Bitid**: bit 0 = toote R-kanali sagedusala (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Seadistatud bitt tähendab, et selle sagedusriba pikseli väärtus on piiratud 0-ni — käsitle seda ja sellest koostatud indeksit madala usaldusväärsusega andmetena.
- **`Chloros:UnmixClampMask`** on märgitud igale tootele, mille puhul on läbi viidud segunemise eemaldamine: maski asukoht suhtes toote enda kausta (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, alati `/`-eraldatud – ühendage see pildi kaustaga, et maski avada) või `none`, kui ühtegi pikslit ei ole piiratud. Märgise puudumine tähendab, et tootel ei ole kunagi unmix-töötlust läbi viidud (RGB- või monokaamerad, toored/debayered tasemed).
- Pildistamise tulemused kannavad seda kaadri **`clamp_mask`**-na (maski tee või `None`, kui midagi ei ole piiratud) — `CameraSession.capture()` tulemustes on `ArraySession.capture()`&#x27;kaadrite ja HTTP-salvestusvastuste puhul. Offline-koostis loetleb maske kui `clamp_masks`.
- Reaalajas indeksi mitme sagedusriba pinu (`/api/camera/cli-array-index` koos `save_multiband`-ga) saab ühe maski iga allikakaamera kohta, väljundkausta enda `clamp_masks/`-is — vt CLI-viidet`lattice index`. Videol maski ei ole: offline-koostamise tulemus näitab selle asemel iga kaadri piiramisosa.

Kuna üheski tootekausta tasandil ei ole maski, võib tootekausta edastada fotogramm-meetria tarkvarale sellisena, nagu see on – koos alamkaustadega –, ning tarkvara võtab vastu ainult pilte. Edastage talle tootekausta, mitte selle kohal asuvat kausta. Chlorose enda importimisfunktsioon jätab maskid vahele.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect LATTICE-kaameratele

Püsivad tagapõhja-seansid reaalajas töötava riistvara jaoks. Kasutatakse samu lõpppunkte, mida kasutab ka graafiline kasutajaliides (GUI), seega on käitumine identne nii SDK-s, CLI-s kui ka GUI-s.

### Üks kaamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Siin määratud säri, võimendus ja sihtmärk jäävad kehtima: Chlorosi automaatne särijuhtimine (sealhulgas DAQ-mõõdetud säri) ei muuda neid. `cam.set_settings(exposure_auto="Continuous")` (või `gain_auto=`) annab säri (võimenduse) tagasi, ja `cam.set_settings(target_brightness_auto=True)` annab sihtmärgi tagasi Chlorose automaatsele sihtmärgile.

#### `connect_camera()` Allkiri

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **See naaseb, kui voog on käivitatud.** Tagapõhi ootab piiratud aja jooksul (umbes 15 sekundit, kuni voog algab, seejärel 10 sekundit, kuni esimene täielik puhver on valmis – mitte esimese eelvaatekaadri jaoks, mis järgneb umbes 1,5 sekundit hiljem), seega `status()` kohe pärast seda, kui on öeldud `streaming`. Sessiooni `connect_stream` on vastuse `stream` plokk: `state` `streaming`, `no_frames` (voog algas, kuid ei edastanud midagi – seda näitab `note` ning jumbo-pakette kasutava kaamera puhul tuleb ühendus taastada 1500-baidise paketi suurusega), `starting` või `start_failed`, kus `waited_s`. `None` kaamera jaoks, mis oli juba ühendatud. Eelvaade (`get_camera_preview`, `/api/camera/jpeg/<serial>`) päringule järgneb piiratud ooteaeg (4 s) kaamera esimese eelvaatekaadri saamiseks, selle asemel et vastata „No frame available yet“; kaamera, mille voog pole midagi edastanud (`no_frames` või riistvaratriggeri jaoks aktiveeritud), vastab ikkagi kohe.
- **Paketi suuruse saad ise määrata.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` avab kaamera 1500-baidiste pakettidega, ilma et seda eelnevalt testitaks (`chloros-cli lattice cam-connect --packet-size standard` saadab sama).
- **Kaamerapooli hoiab kinni üks ühendus korraga**: avamine, paketi suuruse proovimine, esimene kalibreerimisandmete allalaadimine (umbes 70 s selle arvuti jaoks uue kaamera puhul), voo alustamine. Päring, mis vajab vahepeal kaamerat (`read_nodes`, `set_settings`, salvestamine, teine ühendus) ootab seda maksimaalselt 5 sekundit, seejärel ebaõnnestub HTTP-veaga 503 `CAMERA_POOL_BUSY`, nimetades ühenduvat kaamerat (`ChlorosConnectError`); küsi hetke pärast uuesti. Nimekirjad ei oota kunagi: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` ja `CameraSession.status()` vastavad kohe. Ühendamisel olev kaamera on loetelus pool-vastuse all `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — ja `elapsed_s`), ning selle `status()` näitab `connecting: True` koos selle `phase`-iga; muid kaameraid loetakse reaalajas, kui ressurss on vaba, muul juhul vastatakse nende viimase lugemise põhjal (`live: False`).

#### `CameraSession`-meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `status(timeout=15.0)` | Kaamera reaalajas olek, nagu seda loeb `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` või `stopped`), `device` (mudel, seerianumber, püsivara, temperatuur, `is_mono`) ja selle praegused seaded. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Loeb GenICam-sõlmi; tagastab `{nodes, errors, enums, device, was_in_pool, in_array_id}`, lisaks `frame_values` ja `notes`, kui töötab automaatne säritus (vt allpool). |
| `set_settings(**kwargs)` | Kirjutab sõlmedele sõbraliku nime järgi (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). `exposure_time` (`gain`) on käsitsi valgustatus (võimendus): kirjutamisel ilma `exposure_auto` (`gain_auto`) kirjutamisel lülitatakse esmalt automaatrežiim välja, et töötav automaatrežiim väärtust tagasi ei lükkaks. Tulemuseks olev `applied` sisaldab seda, mida kaamera salvestas; kaamera poolt tagasi lükatud kirjutamine on salvestatud `errors`-is ja seda ei säilitata. Viga määrab kaamera praeguse aktsepteeritava vahemiku (`ExposureTime` maksimum sõltub kaamera kaadrisagedusest), ning automaatrežiim, mis lülitati välja ainult keeldutud väärtuse jaoks, lülitatakse uuesti sisse; `restored` loetleb need (nt `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Salvestab **ühe** kaadri. Tagastab `CaptureResult`: kaadrite metaandmete sõnastikute nimekirja (üks iga salvestatud taseme kohta), koos `.skipped`, `.failed`, `.daq_files` ja `.daq_matches`. (Seeria-/mitme kaadri salvestamine on eemaldatud — kui vajate seeriat, kutsuge `capture()`-i tsüklis.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Järjestikune (`interval_s=0`) või intervalliga pildistamine selle kaameraga, `count` pildistamiste ja/või `duration_s` puhul (vähemalt üks, et see lõppeks), nagu `ArraySession.capture_repeated()` teeb massiivi puhul. Tagastab `CaptureResult`-ide nimekirja; see peatub pärast salvestust, mis teatab ketta täitumisest. `on_capture(i, result)` käivitatakse pärast iga salvestust; muud märksõnad suunatakse `capture()`-i. |
| `disconnect(keep_settings=False, clear_settings=False)` | Vabastamine reservist, kaamera taastatakse selliseks, nagu see ühendamisel leiti (päästiku juhtmestik, PTP, seaded); `keep_settings=True` salvestab seadistused hoopis oma mällu, `clear_settings=True` tühistab selle mälu (vt [Kaamera mälu](#camera-memory--keep_settings--clear_settings)); tagastab `camera_memory`, kui küsitakse kumbagi neist. Massiivi liikmeks olev kaamera vabastab kogu massiivi. Ei tee midagi, kui ühendati juba avatud sessiooniga. Tagastab tulemuse, kui tagapõhi kinnitab vabastamise või kui pole midagi vabastada (kaamera on juba kadunud või ükski tagapõhi ei tööta); vabastamine, mille puhul aegus ooteaeg või mille tagapõhi keeldus, tekitab veakoodi `ChlorosConnectError`, sest kaamera võib endiselt kinni olla ja järgmine ühendus põrkuks sellega kokku. Bloki `with` lõpetamine ei tekita kunagi veakoodi. |

Atribuudid: `serial`, `device` (kaamera avastamisandmed) ja `backend_url`.

`capture()` ekspordikontrollid (sama mudel kui massiiv + graafiline kasutajaliides):

- `processing` / `levels` — `processing="all"` salvestab kõik kohaldatavad eksporditüübid; `levels=["raw","radiance"]` salvestab ainult need (asendab `processing`). Jäta mõlemad välja, et kasutada tagapõhja vaikimisi seadeid.
- `daq` — kasutatud salvestust lugenud DAQ salvestatakse vaikimisi `.daq`-i lisafailina, mis sisaldab ainult töötlemata andmeid, nii et kaadrit saab hiljem ümber töödelda peegeldusvõime/indeksiks. `daq=False` jätab selle vahele, kui ükski tase seda ei kasuta. `force_daq=True` on vaikimisi seadistuse vanem kirjapilt. Kui DAQ-i pole, ei ole ka `.daq`.
- `daq_tolerance_ms` — kui kaugel võib lugemine ekspositsioonist olla ja ikkagi jääda lubatud piiridesse (vaikimisi 500). Muutub ainult tulemus.

#### `read_nodes()` automaatse eksponeerimise korral — `frame_values` ja `notes`

Kui `ExposureAuto` (`GainAuto`) töötab, loeb `ExposureTime` (`Gain`) sõlm loeb tagasi **viimase kirjutatud väärtuse**, mitte seda, mida kaamera parasjagu teeb. `nodes` hoiab endiselt seda sõlme väärtust. Kui küsida `ExposureTime` või `Gain` ühendatud kaameralt, mille automaatrežiim ei ole `Off`, lisatakse vastusele:

- `frame_values` — kaamera viimases kaadris sisaldunud väärtus, sõlme nime all (`ExposureTime` mikrosekundites, `Gain` detsibellides);
- `notes` — üks lause iga sellise sõlme kohta, milles on märgitud mõlemad väärtused.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Mõlemad võtmed puuduvad, kui automaatrežiim on `Off` (sel juhul on sõlmetegelik väärtus) ja kaamera puhul, mis ei ole ühendatud tagapõhjaga. Massiivi puhul annab `ArraySession.status()["member_ae"]` korraga iga elemendi kaadri väärtused.

#### DAQ-mõõdetud säri ülempiir (HTTP)

Kui DAQ-valgusandur on ühendatud, piirab Chloros kaamera säri × võimendust mõõdetud päevavalguse järgi, et valge võrdluspaneel ei läheks üle. Piirang kehtib ainult siis, kui kalibreerimissihik on selle kaamera vaateväljas (nähtav kaadri piires viimase 10 sekundi jooksul); kui sihtmärki vaateväljas pole, töötab selle asemel kaamera tavaline automaatne säritus koos oma heledate alade kaitsega, nii et ülejäänud stseen ei jää vajalikust tumedamaks. SDK-moodulit ei ole; seda saab iga kaamera puhul lugeda või muuta käskudega `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (POST-keha `{"enabled": false}` lülitab selle selle kaamera jaoks välja). Vastuses on `enabled`, `cap_us` (reaalajas ülempiir, `None`, kui funktsioon on mitteaktiivne, sealhulgas juhul, kui kalibreerimise sihtmärk ei ole vaateväljas), `k_exp_g` (konstant, mille abil ülempiir arvutatakse) ja:

| Väli | Tähendus |
| --- | --- |
| `k_exp_g_source` | Millise konstanti kasutatakse: `camera` — selle kaamera oma, arvutatud kalibreerimispaketi laboripunktist (RGB-kaamerad); `fleet` — seeria konstant, mida kasutatakse juhul, kui paketis puudub laboripunkt; `override` — backendis määratud välja ülekirjutamine. |
| `k_exp_g_fleet` | Seadmepargi konstant, võrdluseks. |

### Sünkroniseeritud massiiv — `ArraySession` (Smart-Prep)

`connect_array` on **soovitatav lähtepunkt** mitme kaameraga seadistuste jaoks. See käitab taustal täielikku GUI-põhist Smart-Prep-protsessi:

1. **Võrguanalüüs**(`/api/camera/array/recommend`) — seadistab**kogu anduri** (ilma binninguta, ilma ROI kärpimiseta) ja laseb kaadrisagedusel langeda sellele tasemele, mida see host ja võrk suudavad pakkuda, arvestades pikseliformaate ja pakkimist, mida Connect kasutab. Kaadreid ei vähendata kunagi kaadrisageduse säilitamiseks: edasta väiksem `width` / `height` või `binning=2` / `4`, et vahetada resolutsiooni kaadrisageduse vastu (lõpppunkti`alternatives` loetleb need valikud koos nende prognoositavate kaadrisagedustega).
2. **Taseme automaatvalik** — `sim-capture-sim-emit` (iga kaamera teeb eksponeeringu sama trigeri peale, väljastus on ajastatud, et jagada ühendusliini, kaadrisagedus langeb, kui jagatud ühendusliini maht väheneb) mis tahes võrgus, kus kaameraid saab ajastada ühendusliini jagamiseks; `slip-emit-and-capture` (kaamerad kasutavad ühendust kordamööda, mitte samaaegselt) ainult juhul, kui see pole võimalik.
3. **PTP on vaikimisi sisse lülitatud**— kaameratevahelised ajamärgised langevad ühele ühisele kellale**~1 ms**täpsusega, tagapõhja [PTP grandmaster](#ptp-grandmaster--time_sync_status). Samaaegne eksponeerimine tuleneb M8 riistvaratriggerist (**&lt; 100 µs** moodulite vahel), mitte PTP-st: PTP joondab *ajamärke*, mitte eksponeerimisi.
4. **Iga kaamera pikseliformaadi automaatne valik** — mustvalge → `Mono12`, värviline → `BayerRG12`.
5. **AE algväärtustamine** — salvestab iga kaamera hetke AE-seisundi, et ühendus ei nulliks säriaega töötamise käigus.
6. **GPIO-trigeri konfiguratsioon** — `connect_array` aktiveerib kõik kaamerad (`TriggerMode=On`, `TriggerSource=Line2`), nii et peakaameraimpulss juhib orikaameraid M8-kaabli kaudu. See on ainult massiivi jaoks mõeldud samm: üksik kaamera, mis on avatud käskuga `LatticeCamera`, töötab selle asemel vabalt. Massiivi vabastamisel (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) viib kõik kaamerad tagasi vabajooksule (`TriggerMode=Off`), kusjuures kõik muu jääb selliseks, nagu ühendus selle leidis: päästiku ja sünkroniseerimisliini seaded (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; peakaamera sünkroniseerimisliin tagasi sisendiks), PTP ja seansi käigus muudetud kaamera seaded – välja arvatud juhul, kui `keep_settings=True` / `clear_settings=True` (vt [Kaamera mälu](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atribuudid: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` ja `packed_reason` (viimased neli on `None` käepideme peal, mis pärineb `attach_array`-ist), `connect_stream` ja `backend_url`. `connect_stream` on ühendusvastuse plokk `stream`, ühe ühenduse kuju pluss `members` (`{serial: {state, why?}}`): `streaming`, kui massiivi esimene sünkroniseeritud rühm on kohale jõudnud (tagapõhi ootab seda piiratud aja jooksul, umbes 10 sekundit), või `no_frames`, mille `why` on iga liikme viimane andmete hankimise ebaõnnestumine ning jumbo-pakettide puhul 1500-baidise varuvariandi korral `note`. `None` `attach_array` käepideme puhul.

#### `connect_array()` allkiri

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` näitab, kui kaua kutsumine ühendust ootab, ning see saadetakse
tagapõhja selle tähtajana. Kui ühendust selleks ajaks ei ole loodud, tühistatakse see: iga kaamera, mille
see avas, viiakse tagasi sellisesse seisundisse, nagu see leiti, ja suletakse, ühtegi massiivi ei ühendata ning
kõne tekitab vea `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, nimetades
etapi, milleni ühendus oli jõudnud). Seega ei jäta kutsung, mis tekitas vea, kunagi massiivi
ühendatud massiivi. 9-kaameraline massiiv ühendub umbes minuti jooksul; vahetult pärast
kaamerate sisselülitamist võib see võtta mitu minutit. Massiiv, mis on üles seatud, kuid millel pole veel
sünkroniseeritud kaadrirühma, tagastatakse koos veakoodiga `connect_stream["state"]`
`no_frames`. Tagapõhja logis on üks ajastusrida iga ühendusetapi kohta
(`[array-connect <id>] <step>: <s> s …`) ja viimane rida, mis nimetab aeglaseimat.

`force_tier` väärtused:
- `"sim-capture-sim-emit"` — tõeline sünkroonsus (kõik kaamerad vallanduvad sama taktsignaali serval).
- `"sim-capture-ftd-stagger"` — paindlik ajadomeeni nihutus (kaamerad saadavad signaali veidi nihutatud aegadel, nii et paketid järjestuvad juhtmes).
- `"slip-emit-and-capture"` — järjestikune salvestamine kaamera kaupa (ajaliselt sünkroniseerimata). Valitakse automaatselt ainult siis, kui kaameraid ei ole võimalik piisavalt aeglaselt tööle panna, et nad saaksid ühendust samaaegselt jagada.

`wire_ceiling_mbps` tühistab **host&#x27;i püsiva ühenduse eelarve** MB/s-is — see on ainus
arv, millest sõltub kogu massiivi eraldus. Jätke see `None`-iks, et kasutada automaatselt tuvastatud
väärtust. Vähendage seda, kui massiiv teatab GVSP-riknenud raamidest: automaatne väärtus tuletatakse
võrgukaardi teatatud ühenduskiirusest, mis ülehinnab USB-adaptereid, kitsaid PCIe-kanaleid ja
hõivatud jagatud struktuure — ning see ülehinnang avaldub pigem vigaste raamidena kui
nähtavalt aeglase ühendusena. Väärtus salvestatakse projekti massiivi salvestusplokki, seega
taastub see massiivi uuesti avamisel või hiljem `connect_array` abil nagu iga teine massiivi seade.
Vaata [Massiivi tervis](#array-health--which-subsystem-is-losing-frames).

#### Pakitud 12-bitine edastus (`packed=`)

Täisanduriga massiivi kaadrisagedus sõltub sellest, kui kiiresti iga kaamera suudab
oma kaadreid saata. 12-bitised formaadid (`BayerRG12`, `Mono12`) edastatakse tavaliselt 16-bitiste
sõnadena; pakitud edastus saadab samad 12 bitti 1,5 baiti pikseli kohta ja
pakib need lahti hostis, mistõttu iga kaadri saatmine võtab 25% vähem aega — umbes
**33% rohkem kaadreid sekundis täisanduriga** (mõõdetud: 10,7 → 14,0 kaadrit sekundis
9-kaameralisel massiivil). **Salvestatud andmed on identsed**: samad väärtused, samad 16-bitised
failid, samad metaandmed.

- `packed="auto"` (vaikimisi) pakib andmeid ainult siis, kui see on kasulik **ja** kui host suudab
  pakkimist lahti pakkida: ühenduse loomisel mõõdab backend oma lahtipakkimiskiirust,
  protsessorite arvu, hetke koormust ja võrgu läbilaskevõimet. Töölauaarvuti, millel on vabu tuumaid,
  pakib andmeid täieliku andurikoormuse juures; hõivatud või madala võimsusega host (üheplaadilised arvutid,
  sisseehitatud kontrollerid) edastab andmeid lahtipakitud kujul. Väikesed huvipiirkonnad (ROI) ja binnitud kaadrid ei anna suurt kasu.
- `packed="off"` (või `False`) ei pakenda kunagi — selle välistamiseks diagnoosimise ajal
  või hostil, mis peab hoidma oma CPU-d muude ülesannete jaoks vabana.
- `packed="on"` (või `True`) pakib alati 12-bitiseid formaate.
- Mis tahes muu väärtus tekitab veakoodi `ValueError`. 8-bitiseid ja 16-bitiseid formaate ei pakita kunagi.
  `analyze_array_network(..., packed="auto")` projektid tehakse sama otsusega,

`analyze_array_network(..., packed="auto")` teeb sama otsuse,
seega on selle fps kiirus, millega massiiv töötab (vastus sisaldab
`packed_transport` ja `packed_reason`); selle `pixel_format` vaikimisi väärtuseks on see, mida
connect rakendab (Mono12 / BayerRG12), mitte 8-bitine.

Otsus ja selle põhjendus on seansis:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` kohaldab sama ümberkirjutust igale
projekti massiivile (vaikimisi `None`, mis tähendab `"auto"`: projekt ei salvesta
pakitud seadeid massiivi kohta).
Kogu masina hõlmava vaikimisi seadistuse jaoks määrake `CHLOROS_ARRAY_PACKED12=off` (või `on`)
tagapõhja keskkonnas; kuid selgesõnaline `packed="on"` / `"off"` on ikkagi ülimuslik.
`/api/camera/array/status` teatab `packed_transport` ja `packed_decision`-i.

#### Eeldatavad kaadrisagedused

Mõõdetud massiivi kaadrisagedused (riistvarasünkroniseeritud, 12-bitised; täisandur = 2048 × 1536). Kasutage neid `status()["actual_fps"]` kontrollimiseks; `analyze_array_network()` prognoosib kaadrisageduse teie enda hosti ja võrgu jaoks.

| Seadistus | Massiivi kaadrisagedus |
| --- | --- |
| Täissensor, PCIe 10 GbE host-võrgukaart, pakitud edastus (`packed="auto"` lauaarvutil) | ~14 kaadrit sekundis |
| Täisandur, PCIe 10 GbE host-võrgukaart, pakkimata (`packed="off"`) | ~10,5 kaadrit sekundis |
| Täisandur, 8 kaamerat USB 10 GbE-adapteril | ~3,8 kaadrit sekundis |
| Täissensor, 1 GbE host-link | 1,2–1,7 kaadrit sekundis |
| 2 × 2 binning (`binning=2`), 10 GbE host-võrgukaart | ~30–33 kaadrit sekundis |

- PCIe 10 GbE-ühenduse korral on täissensoriga töötamisel piiranguks **kaamerad**, mitte võrk: 4 ja 8 kaamerat töötavad sama kiirusega.
- **`target_fps` on käivitussagedus ja massiiv ei tööta kunagi kiiremini.** `target_fps=1` juhib kaamerate rühma kiirusega 1 kaadrit sekundis; `None` (vaikimisi) juhib seda kiirusega, mida host ja võrk suudavad toetada. Seda kiirust ületav väärtus alandatakse sellele tasemele (vt [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` väärtus `4294`-is `status()`-is ei ole ühenduse kiirus.** See tähendab, et Windows ei suutnud adapteri kiirust lugeda (draiver teatas suurimast 32-bitisest väärtusest), mis on tüüpiline USB-Ethernet-adapterite puhul. USB-adapteri kasutatav kiirus Windowsis on igal juhul piiratud 200 MB/s-ga (vt [Array Health](#array-health--which-subsystem-is-losing-frames)); selle häälestamiseks vaata CLI-viite jaotist „USB 10GbE adapterid” rubriigis [Host NIC Setup &amp; Tuning](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **USB-piirang kehtib ka Linuxis: 400 MB/s x86_64-platvormil, 200 MB/s NVIDIA Jetsonil.** Adapter tuvastatakse sysfs-i kaudu (tagavaraks `ethtool -i`), seega arvestatakse USB 5 GbE või 10 GbE adapteri puhul piiranguga, mitte selle nimivõimsusega 500 / ~1060 MB/s. x86_64 näitaja mõõdeti Linuxis: sama adapter, mis Windowsis rikub kaadreid kiirustel üle 200 MB/s, töötas 4-kaameraga süsteemis veatult igal piirkiirusel kuni 480 MB/s. NVIDIA Jetsonil töötas sama adapter veatult vaid kiirusel 200 MB/s ja kaotas kaadreid vahelduvalt sellest kiirusest suuremal kiirusel. Kui teie host suudab ilma `gvsp_corrupt`-kaadriteta rohkem andmeid vastu võtta, suurendage seda parameetrit `wire_ceiling_mbps` abil. Linuxis on USB 2.0-pordis olevale adapterile eraldatud 30 MB/s; ühendage see USB 3-pordiga.

#### Ülekoormus (kaamera kohta kehtiv miinimum)

Massiiv töötab igas võrgus; kaadrisagedus sõltub võrgu võimsusest. Sim-emit-tempo jaotab igale kaamerale võrdse osa kokkupõrkevabast andmeside eelarvest ning kõik kaamerad eksponeerivad end ikkagi sama trigeri järgi, seega aeglasel ühendusel on osa – ja kaadrisagedus – väike: 8 täisres 12-bitist kaamerat 1 GbE-võrgus ilma jumbo-raamideta saavad igaüks 6,5 MB/s, umbes **1 fps**(pakendatuna umbes 1,3 fps). Osakaalu alampiiriks on kaamerate aeglaseim võimalik edastuskiirus (`per_cam_floor_bps`: loetakse ühendamisel iga kaamera GevSCPD vahemikust ja kunagi alla 1 MB/s). Kui võrk nõuab neilt aeglasemat kiirust, lülitub ühendus tagasi**jadaliidese**edastamisele (`slip-emit-and-capture`): kaamerad töötavad kordamööda täisühenduskiirusel, nii et iga raam jõuab kohale, kuid liikmed**ei ole üheaegselt kättesaadavad**— vastuse tase ja blokeerijad (`SERIAL: ...`) seda kinnitavad; edasta `force_tier="sim-capture-sim-emit"`, et tagasipöördumist keelata. Kaadri suurus ei mõjuta jagamist, seega**kaadrite ühendamine ja huvipiirkond (ROI) suurendavad kaadrisagedust, kuid ei muuda seda, kas massiiv mahub**. Ainult sideühendus, mis ei suuda massiivi edastada ühelgi sagedusel, ei samaaegselt ega vaheldumisi,**ületab võimsust** — rikkeviisiks on GVSP-pakettide kadu, mitte madalam kaadrisagedus. Lahendused: vähem kaameraid, jumbo-kaadrid otsast otsani või kiirem võrgukaart.

- Vastused `analyze_array_network()` ja `/api/camera/array/connect` sisaldavad `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` ja `per_cam_floor_bps`. Kui `oversubscribed` on tõene, siis projektsioon **nullib fps-väljad** (`achievable_fps_max` / `fps_bright` / `fps_dark`), selle asemel et näidata eksitavat aeglast, kuid toimivat kiirust.
- `POST /api/camera/array/connect` aktsepteerib `pin_resolution` keha parameetrit (**ainult HTTP — mitte SDK-i kwarg**; `connect_array` seda ei avalda). See on range režiim: ülemäära koormatud ühendus, kus `pin_resolution` on määratud,**lükatakse kindlalt tagasi** koos veateatega, milles nimetatakse kõik võimalikud lahendused. Ilma selleta jätkub ühenduse loomine hoiatusega, et massiiv kaotab pakette. Mõlemal juhul säilitab ühendus taotletud raami suuruse — see ei jaota ega kärbi kunagi ise.
- Ainult testimiseks: määrake tagapõhja keskkonnas `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`, et alandada keeldumist valjuks hoiatuseks – te loote ühenduse niikuinii ja aktsepteerite pakettide kadu.

#### Massiivi tervis – milline alamsüsteem kaotab raame

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) kannab reaalajas `health`-plokki
ühendatud massiivis, mida hinnatakse uuesti jooksvalt **10-sekundilise** aknas. See jagab raamikao
kaheks põhjuseks, mis vajavad vastupidiseid parandusi, selle asemel, et näidata üht „ebatäieliku“ määra, mis
mõlemat ei nimeta:

| Väli | Mida see tähendab | Milline alamsüsteem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (seerianumbri kaupa) | Kaader **saabus, kuid oli struktuuriliselt vigane**— GVSP-pakettide kadu. |**Võrk**: ühenduse läbilaskevõime, tempo, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (seerianumbri kaupa) | Raam **ei jõudnud üldse kohale**— kaamera ei vallandunud või midagi ei väljunud sellest. |**Trigger / sünkroniseerimine**: M8-kaabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Kõige halvem kaamera edastuskiirus iga kaamera kohta. | — |
| `per_cam_rate_pct` | Kaamera kohta kombineeritud puudulikkuse määr (mõlemad põhjused kokku). | — |
| `stable_for_seconds` | Kui kaua on iga kaamera püsinud alla 0,01 %. | — |

Koos `health`-iga näitab sama kirje ka kogu eraldise mahu:

| Väli | Mida see tähendab |
| --- | --- |
| `wire_ceiling_mbps` | Hosti kehtiv püsiv andmeside maht, MB/s. |
| `wire_ceiling_source` | Selle arvu päritolu sõnadega — nt `USB-capped 200 MB/s (was theoretical 1062; …)` või `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, kui `wire_ceiling_mbps=` selle seadistas. |
| `nic_is_usb` | `true` USB-Ethernet-adapteri puhul. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Lugemine:** nullist erinev `gvsp_corrupt_rate_pct` koos `never_arrived_rate_pct`-iga väärtusega 0 tähendab, et
trigger ja kaabli sünkroniseerimine on täiuslikud ning 100 % kadustest on võrguteel — vähendage
`wire_ceiling_mbps` ja ühendage uuesti. Vastupidine muster viitab pigem sünkroniseerimiskaablile või
triggeriliinile.

> **`target_fps` ei ole rikutud raamide reguleerimisvahend.** GevSCPD-i taktsagedus kirjutatakse ühe korra
> ühenduse loomisel, seega triggereerimissageduse alandamine muudab töötsüklit, mitte
> samaaegse saatmise pursesagedust. Mõõdetud 5-kordne nõudluse vähendamine ei toonud parandust, samas kui
> kaabli ülempiiri langetamine 240-lt 200 MB/s-le viis sama seadme vigaste raamide määra 10,4 %-lt
> 0,00 %-ni.

> **Töötav massiiv ei saa LATTICE (M3C / M3M) püsivara puhul ise uut plaani koostada.** See ei suuda
> seda ise parandada; ühendage seade lahti ja ühendage uuesti, et ühendusaja valija planeeriks ümber vastavalt
> uuele ülempiirile.

**USB-Etherneti adapteri kiirus on proovi poolt piiratud 200 MB/s-ni**, hoolimata selle
nimetusest: tõhusustabel, mis teisendab ühenduse kiiruse püsivaks näitajaks, on
PCIe-põhine, ning USB-võrgukaart edastab oma Etherneti ühenduse kiiruse, olles piiratud
USB-bussi ja selle draiveriga. Piirang on absoluutne, mitte osaline — USB 1 GbE-adapter
saavutab ~80 MB/s ja see ei mõjuta seda. See kehtib Windowsis ja Linuxis (sh Jetson);
vaata [Eeldatavad kaadrisagedused](#expected-frame-rates), kus seda mõõdeti.

#### `ArraySession` meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `status(timeout=10.0)` | Käimasoleva massiivi reaalajas seisund: `target_fps` / `target_fps_auto` (triggeri sagedus ja kas sagedusregulaator kontrollib seda), `target_fps_clamped_reason` (miks fikseeritud sagedus hoitakse madalamal, muidu `None`), `actual_fps` (mõõdetud; 0, kui haaramistsükkel on loobunud või takerdunud), `recommended_fps`, `streaming` (väär, kui haaramistsükkel on loobunud — kontrollige seda, mitte ainult `last_error`; jääb tõeks ka seadete taaskäivitamise järel, mida märgib `restarting`), `stalled` (tõene, kui haaramistsükkel töötab, kuid ei ole avaldanud ühtegi kaadrirühma umbes kümne kaadriperioodi jooksul, 2–30 s; `last_error` märgib seda alles 60 sekundi möödudes), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` ainult siis, kui massiiv oli ühendatud PTP-ga **ja** iga liige luges viimati `PtpEnable` sisse; ei ole sama, mis lukustatud, mida mõõdab `ptp_epoch`), `ptp_configured` (mida ühendus nõudis), `ptp_off_members` / `ptp_unread_members` (liikmed, kes lugesid viimati `PtpEnable` välja / ei vastanud), `member_recovery` (`None` või liige, kes kaotas toite või ühenduse: `state`, `waiting`, kuigi ta ei vasta, `reopening`, seejärel `recovered`, koos `members` ja `why` — massiiv avab selle uuesti, kui see vastab, ja annab talle tagasi ühendusaja PTP, triggeri ja sünkroniseerimisliini ühendused, tempo ja kaamera seaded, seejärel edastab uuesti ilma uute `connect_array`) ja `ptp_epoch` (mõõdetud), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` tähendab, et Windowsil ei õnnestunud adapteri kiirust lugeda — vt [Oodatavad kaadrisagedused](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (kas sagedusriba-põhine kiirgusahel on kasutatav ja milline liige selle katki tegi) ning `band_clamp` — samal ajal, kui reaalajas vaade renderdab maatriksit: valgustatud pikslite osakaal, mille segamisvaba töötlus on piiranud 0-ni, iga elemendi kohta (`members`: `{serial: [band0, band1, band2]}` kaamera R-, G-, B-sagedusribade järjekorras) ning sagedusriba kohta näitab komposiit (`composite`: `{band name: share}`), kusjuures `warn` näitab osakaalu, mida kasutajaliides märgistab kui „madala usaldusväärsusega laineandmed“ (mõõdetakse umbes kord sekundis; `None`, kui reaalaja vaadet ei ole käivitatud) ning `member_ae` — iga liikme kohta ekspositsioon (`exposure_us`) ja võimendus (`gain_db`) selle viimases avaldatud kaadris, mis pärinevad kaadrist endast (`read_nodes` või `ExposureTime` / `Gain` tagastab viimase väärtuse, mis kirjutati automaatse särituse töötamise ajal, mitte selle, mida see parasjagu teeb). |
| `set_member_settings(serial, **kwargs)` | Kirjutab ühe liikmekaamera seaded, nagu `CameraSession.set_settings()` need vastu võtab (eksponatsioon, võimendus, sihtheledus jne); liige jääb massiivi. Tekitab veateate `ChlorosConnectError`, kui tegemist on massiivi mittekuuluva seeriaga. Vaata [Massiivi liikmete eksponeerimine](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Mida smart-prep valis (pildistamistasand, kaadri suurus, `pixel_format`), reaalajas töötav `health` plokk ja hosti andmeside eelarve. Vaata [Massiivi tervislik seisund](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Muuda käivitussagedust massiivi töötamise ajal ilma ühendust uuesti loomata. Numbriline väärtus fikseerib selle (sagedusregulaator ei muuda seda); `None` taastab automaatrežiimi. Tagastab `{requested, target_fps, target_fps_auto, clamped}` — `clamped` on True, kui sagedus on alandatud tasemele, mida host ja võrk suudavad edastada. Vaata [Intervallpildistamine drooniga tehtavate uuringute puhul](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Üks kaader igast kaamerast, **kõik ühe käivituse tulemusel**, numpy-massiividena (faile ei salvestata). Tagastab `GrabResult` (`dict`): `{serial: array}` või `{serial: (array, metadata)}` koos `with_metadata=True`-iga. `.group_id` on kaadrite ühine käiviti; `.errors` seostab iga välja jäetud kaamera põhjuse (kaamera, mida selle käivitiga teenindada ei saa, jäetakse välja ega täideta kunagi teise kaameraga). `processing` on `raw` / `debayered` / `radiance` / `reflectance` / `display`. Sama lugemine nagu projekti API-s [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — üks ühine rakendus — nii et iga sealne märksõna (`band`, `align`, `bit_depth`, `refl_max`) töötab siin. Vajab `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iteraator, mis annab ühe `grab()` tulemuse iga takti kohta (ainult märksõnaargumendid), sünkroniseeritud `fps`-ga, piiratud väärtusega `duration` (sekundites) ja/või `count` (`None` = piiramatu). **Iga tagastatud hulk on uus käivitaja:** tikk, mille uusim triggert on juba väljastatud hulk, ootab järgmist triggert selle kordamise asemel, mistõttu voog töötab aeglasemal kiirusel, mis on väiksem järgmistest: `fps` ja massiivi enda kaadrisagedus. Tekitab `FrameFetchError` asemel vaikimist: kohe, kui iga kaamera keeldub mingil põhjusel järgmise tiki kordamisest, pärast 60 sekundit kestnud tike ilma kaadrita ning pärast 60 sekundit ilma uue triggita („60 sekundit pole uut triggit”). Sama rakendus nagu `ArrayHandle.frame_stream()`. Vaata [Reaalajas kaadrid oma töötlusketis](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Käivita tagapõhja enda pidev salvestamine (GUI pidev režiim) → [`ContinuousHandle`](#continuoushandle). Salvestab **iga** sünkroniseeritud rühma, mida massiiv avaldab, nii nagu `capture()` seda salvestaks (samad ekspordikontrollid), kuni peatamiseni või kuni `max_captures` / `duration_s` (`None` = kuni peatamiseni). Tekitab veateate `ChlorosConnectError`, kui selle massiivi jaoks on juba üks käimas. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Üks sünkroniseeritud salvestusgrupp. Tagastab `CaptureResult` (kaadrite sõnastikute nimekiri + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Edasta eelmine tulemus`group_seq` edasi kui `after_seq`, et saada järgmine sünkroniseeritud rühm selle järel, selle asemel et oodata uut. Allpool on ekspordikontrollid. |
| `capture(..., smart=True)` | **Nutikas salvestamine** — ootab, kuni AE stabiliseerub kõigil kaameratel, ja käivitub seejärel. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Kiireim salvestamine: ainult toorandmed + määratud DAQ-näit (+ vaba kombineeritud indeks). Peegeldab kasutajaliidese nuppu „Kiireim salvestamine”. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Ühekordne / Pidev / Intervall ühes piiratud tsüklis. Tagastab `list[CaptureResult]`.**Nõuab `count` ja/või `duration_s`**, et see lõpetataks (SDK-s puudub Ctrl+C). Lõpetab ka ennetähtaegselt, pärast läbimist, kui läbiviimine teatab ketta täitumisest (`.failed`-kanne koos `disk_full`-ga); see läbiviimine on viimane tulemus. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Alusta kombineeritud indeksivaate salvestamist videoks/GIF-iks → `RecorderHandle`. Video on Motion-JPEG-vormingus `.avi` (ffmpeg-i pole vaja). Kui graafilist kasutajaliidest pole avatud, renderdab tagapõhi seda: `expr` (indeksaväljend massiivi ribade nimedes, nt `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`) või vaikimisi massiivi NDVI. Üks komposiitsalvesti iga massiivi kohta. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Käivita kõrge kaadrisagedusega toores Bayer-pildiseeria → `RecorderHandle`. Töötle uuesti offline-režiimis `build_video()` abil. `orient=True`: `build_video()` poolt loodud tooted pööratakse vastavalt kaamera orientatsioonile (iga kaamera tooted vastavalt oma kaamera orientatsioonile, kombineeritud indeks vastavalt peakaamera orientatsioonile); toorkaadrid jäävad algupäraseks ja pildiseeria manifest salvestab iga liikme orientatsiooni. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Töötle salvestatud toorkaadrite seeria offline-režiimis ümber kalibreeritud videoks (videodeks). Blokeerib kuni lõpetamiseni (`wait=True`) ja tagastab koostamistulemuse: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — vaata [koostamistulemust](#the-build_video-result) allpool. |
| `build_video_status(job_id, timeout=15.0)` | Küsitle offline-koostamistööd: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Vabasta kogu massiiv — **kui see sessioon selle avas** (`connect_array`): siis on see `force_disconnect()` ja tekitab veateate `ChlorosConnectError`, kui vabastamist ei kinnitata (`with`-ploki väljumisel ei tekita kunagi viga). Kui käepide pärineb `attach_array`-ist, jäetakse massiiv töötama ja stderr-i väljastatakse üks rida, mis seda teatab: massiiv võib kuuluda graafilisele kasutajaliidesele või mõnele teisele skriptile. Iga kaamera jäetakse vabalt töötama ja muus osas sellisena, nagu ühendus selle leidis; `keep_settings=True` salvestab selle seaded kaamera enda mällu, `clear_settings=True` taastab selle mälu tehase seadetele (vt [Kaamera mälu](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Vabastab kogu massiivi, sõltumata sellest, kes selle avas. Kasutage seda seotud käepideme puhul, kui soovite massiivi lõpetada (nt selleks, et ühendada see uuesti teiste seadistustega). Tagastab tulemuse, kui tagapõhi kinnitab vabastamise või kui pole midagi vabastada (massiiv on juba kadunud või ükski tagapõhi ei tööta); ajalimiidiga lõppenud või tagapõhi poolt keeldutud vabastamine tekitab vea `ChlorosConnectError`. Mõlema lipuga tagastatakse see, mida iga kaamera mälu sai (`camera_memory`), muidu `None`. |

`capture()` ekspordikontrollid (sama lõpppunkt, mida kasutavad GUI ja CLI):

- `processing` / `levels` — `processing="all"` (või `levels=["raw","radiance",…]`) salvestab iga kaamera kohta kõik kohaldatavad eksporditüübid; üks `processing` väärtus salvestab vaid selle taseme.
- `aligned` — **vaikimisi sisse lülitatud (`None`), kuna Chloros-rakenduse sama massiivi salvestus on**(selle massiivi kohta kehtiv**joondatud** salvestusseade). Iga liikme mitte-toores eksport joondatakse massiivi [joondamisprofiili](#array-alignment) ja kärbitakse liikmete ühisele kattuvale alale, nii et failid on pikselite kaupa registreeritud ning nende suurus, kärpimine ja `Alignment*`-märgised vastavad rakenduse omadele (ROI piires viiakse teisendus uuesti akna suhtes baasjoonele); `XMP-mapir:AlignmentJSON` ütleb `"applied": true` koos oma `crop_roi`-iga. Raw-failid jäävad moonutamata algses suuruses, kuid kannavad teisendust metaandmetes. `aligned=False` salvestab iga faili algses suuruses, moonutamata; `aligned=True` on vaikimisi määratud. `continuous()`, `capture(smart=True)`, `capture_repeated()` ja projektitasandi `ArrayHandle.capture()` / `capture_smart()` kasutavad sama vaikimisi seadet. Sõltumata sellest, kas moonutust rakendatakse või mitte, salvestab iga liikmefail oma kaamera `Alignment*`-transformeerimise, kui massiivil on joondamisprofiil — välja arvatud kaamera, mida profiil ei joondata (on välja jäetud või sunnitud identiteediks ebaõnnestunud lahenduse tõttu) ning viitekaamera, kui ühtegi teist kaamerat ei ole joondatud: need ei kanna mingeid `Alignment*`-märgiseid. Kui massiivil puudub profiil, salvestatakse salvestus joondamata ning tulemus näitab seda: `result.alignment` on `{"requested": True, "applied": False, "warnings": [...]}`, üks hoiatus iga põhjuse kohta, miks moonutust ei saanud rakendada (profiili puudumine, erineva resolutsiooniga liikmed, massiiv, mida ei saa-sünkroniseeritud massiiv, kaader, millele profiili ei saa kaardistada). `applied` on `True` ainult siis, kui iga eksport, mis moonutust nõudis, selle ka sai. Massiivi automaatne joondamine loob profiili, kui selle ekspositsioon on stabiliseerunud; `ArrayHandle.calibrate_alignment()` / CLI-i `align-calibrate` loovad profiili nõudmise korral. `result.oriented` näitab, kas `orient=True` pööras tooteid.
- `render_index=False` — jäta kaamera-põhine taimestikuindeksi kiht vahele; vaikimisi renderdatakse see seal, kus see on konfigureeritud.
- `daq` / `daq_tolerance_ms` — DAQ-näit salvestatakse vaikimisi `.daq`-kõrvalfailina, isegi kui ükski valitud tase seda ei vaja; `daq=False` jätab selle sel juhul vahele, ja `daq_tolerance_ms` määrab kokkulangevuse tolerantsi (vaikimisi 500). `force_daq=True` on vaikimisi seadistuse vanem kirjapilt. Vaata [DAQ-näit, mida salvestus salvestab](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — lülita iga liikme **tooted**sisse (`display`, `radiance`, `reflectance`, kaamera-põhine indeks ja komposiit) kaamera orientatsiooni järgi pärast joondamisdeformatsiooni:**peakaamera**orientatsiooni järgi, kui need paigutati selle raami sisse (`aligned` vaikimisi — liikmed jäävad ühildatuks), muidu oma kaamera orientatsiooni järgi.**Vaikimisi: iga liikmefaili algne**, nagu massiivpildistustel alati on; `raw` ja `debayered` jäävad igal juhul algseks. Iga fail salvestab selle kaadri orientatsiooni, milles selle pikslid asuvad (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — toorfail, oma kaamera oma), ning `Alignment*` XMP-fail säilitab algse raami teisenduse, mis on rakendatud enne orientatsiooni; seda teisendust sisaldav RAW-fail salvestab ka `ChlorosAligned*` (põhiraami), mis on põhiraami järgi moonutatud toote orientatsioon. Töötlemine pöörab iga toodet vastavalt selle pikslitele kuuluvale orientatsioonile: joondatud orientatsioonile, kui RAW-faili moonutatakse (Array alignment on), või kaamera enda orientatsioonile, kui ei joondata. `capture_repeated()` ja `capture_fastest()` lasevad selle läbi; projekti tasandi `ArrayHandle.capture()` / `capture_smart()` võtavad selle samuti vastu.

**TIFF-pakkimine (ainult HTTP-i nupp):** `ArraySession.capture()` ei saada `compression` võtit, seega kehtib tagapõhja vaikimisi seade — `POST /api/camera/array/capture` loeb `compression` keha parameetrit, `"none"` vaikimisi: pakkimata, anduri pikslid täpselt nii, nagu need loetakse (~6,3 MB täisresolutsiooniga kaadri kohta). `"deflate"` (kaotuseta zlib L1 + horisontaalne ennustaja, ~4,1 MB/kaader) kirjutab väiksemaid faile aeglasemalt — mõlemad on kaotuseta ja loetakse importimisel identselt. SDK ei paku selle jaoks ühtegi kwarg-i; pääseteeks on `chloros-cli lattice array-capture --compression deflate` või toores HTTP. DEFLATE hoiab kinni Pythoni GIL-i, seega kompresseeritud kirjutamised ei toimu paralleelselt kaamera-põhiste kirjutajate niitide vahel — püsiv täisresolutsiooniga salvestamine anduri kiirusel nõuab kompresseerimata vaikimisi seadet. Täpsemalt: [CLI-viide → array-capture](cli-reference.md).**Järgnevad rühmad (HTTP):** lõpppunkt tagastab `group_seq`; tsükkel, mis saadab selle järgmisel päringul tagasi kui `after_seq`, saab järgmise sünkroniseeritud rühma — sealhulgas selle, mis avaldati eelmise salvestamise ajal — selle asemel, et oodata päringu järel avaldatud rühma, ning `skipped_groups` teatab kõikidest rühmadest, mida tsükkel vahele jättis. `capture_repeated`, CLI-i `--continuous` ja GUI-i pidev režiim teevad seda kõik; intervallipõhised andmed jäävad iga planeeritud hetkel ajakohaseks.**Liikmekohased ekspordi ümberkirjutused (ainult HTTP):**sama lõpppunkt aktsepteerib ka `exclude_serials` (loend – liikmete eemaldamine salvestatud hulgast; massiiv käivitub endiselt ühe sünkroniseeritud rühmana ja väljajäetud liikmed tagastatakse `excluded`-is), `serial_levels` (`{serial: [level tokens]}` kaamera-tasandi ümberkirjutused) ja `serial_index` (`{serial: bool}` kaamera-indeksi-ülekatte ümberkirjutused). Need on GUI-pariteediga keha parameetrid ja**SDK-i kwargid veel mitte**; kaartidest puuduvad liikmed kasutavad tagavarana massiiviüleseid `levels` / `render_index`.

##### Vahelejäänud kaamerate kontrollimine — `CaptureResult.skipped`

`ArraySession.capture()` tagastab `CaptureResult`, mis on `list` alamklass: iteratiivse läbivaatamise, indekseerimise ja `len()` — kõik olemasolevad mustrid toimivad endiselt. Uus kood saab kontrollida `.skipped` atribuuti, et näha, millised kaamerad jäeti välja ja miks. Kõige tavalisem juhtum on RGB-kaamerad segafiltreid kasutavas massiivis, kui küsitakse `processing="radiance"` või `"reflectance"` — lairibaanduri puhul on Bayeri-põhine kiirgusvõimsus mõttetu, seega jätab tagapõhi need kaamerad vahele, selle asemel et toota mõttetut tulemust.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Põhjenduste märgid järgivad mustrit `<level>-not-applicable-to-rgb-cam` (üks kanne iga vahelejäetud taseme kohta, millest igaüks sisaldab `level`). Kaamera, mille kalibreerimispakett pole laaditud, jätab `radiance` / `reflectance` vahele, kasutades selle asemel `<level>-needs-calibration (connect the camera so its cal pack loads)`, ning kui kasutatavat DAQ-peegeldusnäitu pole, vaheletakse `reflectance-skipped-no-dls`-ga. Muud peegeldusnäitustest tulenevad vahelejätmised on `reflectance-skipped-no-fresh-dls` (puudub värske allapoole suunatud kiirguse näit), `reflectance-skipped-bound-daq-unavailable (…)` (seotud DAQ-seadmele ei olnud võimalik ligi pääseda) ning `dls-uncalibrated-band-<nm>` — lainepikkus jääb peamiselt väljapoole DAQ-valgusanduri radiomeetriliselt kalibreeritud vahemikku (~374–974 nm), mistõttu lükatakse DAQ-põhine absoluutne peegeldusastme jagamine tagasi ja kaader langetatakse selgelt anduri vastuse tasemele. Müügil olevatest toodetest põhjustab seda ainult F988; selle kaamera toetatud töövoog on peegeldusastme-paneeli töövoog.

##### Failid, mida ei suudetud kirjutada — `CaptureResult.failed`

Salvestus, mis suutis kirjutada vaid osa oma failidest, annab ikkagi järgmise tulemuse: nimekiri sisaldab kõiki faile, mis **kirjutati**, ning `.failed` loetleb ülejäänud, üks sõnastik iga faili kohta koos `serial`-iga (`None` kombineeritud komposiidi ja massiivi `.daq` sidecar&#x27;i jaoks), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, või `match` — `.daq`-i päritolu `.match.json`, kui `.daq` kirjutati, kuid selle sidecar&#x27;i ei kirjutatud) ja `error`. Täis kettal sisaldab kirje ka `disk_full: True`-i; iga hilisem salvestamine sellele kettale ebaõnnestuks samamoodi, mistõttu `capture_repeated()` seal peatub. Osalise salvestamise puhul ei teki erandit ning faili `.failed` ei jäetud kettale poolikult kirjutatuks. `.failed` on tühja salvestuse korral tühi.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Salvestamist lugenud DAQ salvestab — `.daq_files` ja `.daq_matches`

Iga kord, kui DAQ on seotud kaamera või massiiviga või koondatud, salvestab iga salvestus kasutatud allapoole suunatud lugemise: faili `.daq` salvestatakse pildistamiskausta alamkaustas `daq/` koos failidega `.match.json` ja `.cameras.json` (kelle mõõtmistulemus see on). See kehtib ka töötlemata või debayereeritud salvestuse puhul ning fail on sama, olenemata sellest, kas salvestus tehti graafilise kasutajaliidese (GUI), käsurealiidese (CLI) või tarkvararakenduse arenduskomplekti (SDK) kaudu. `daq=False` jätab selle vahele salvestuse puhul, mille tasemed ei kasuta DAQ-i (`radiance`, `reflectance` või `display` tase kirjutab selle siiski); `daq_tolerance_ms` nihutab selle salvestuse puhul, mitte kunagi kasutatud näidu puhul. Milline DAQ: kas kaameraga või maatriksiga seotud (GUI valgusanduri valik, projekti seos või `chloros-cli daq pool-bind`), muidu kasutatakse tagapõhja värskeimat. CLI kirjeldab juhtnuppe `--no-daq` ja `--daq-tolerance-ms` ([CLI viide](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Salvestus talletab kasutatud valgusanduri näidu väärtusena `.daq`, mille kõrval on `<file>.daq.match.json`, mis näitab, kuidas näit seostati säriajaga: `method`, `gap_ms` (valgustuse ja lähima DAQ-näidu vahel) ning `in_tolerance` (500 ms jooksul või salvestuse `daq_tolerance_ms`). Peegeldusfaili XMP-andmed sisaldavad sama tulemust kui `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` ja `Chloros:DlsInTolerance`.

Salvestus ootab DAQ-seadme esimest mõõtmistulemust pärast valgustusaega, et saaks ühendada selle mõlemal pool asuvaid mõõtmistulemusi (`method` `interpolated`). DAQ-E saadab näidu päevavalguses iga 0,4 sekundi järel ja siseruumis iga 1–2 sekundi järel, seega tagastab `capture()` tulemuse umbes poolteist sellist intervalli hiljem; valgustuse enda puhul viivitust ei teki. DAQ-seadet, mis on saatmise lõpetanud, ei oodata. DAQ-E mõõtmistulemus on valgustuse keskmine väärtus alates eelmisest mõõtmisest, seega paigutatakse see selle intervalli keskele: `gap_ms` mõõdetakse selle keskpunkti suhtes ning `.match.json`&#x27;`bracket_stamps_ns` annab lugemite enda ajamärgid. `ArraySession.continuous()` ootab ilma salvestamist aeglustamata. `capture_repeated()` tagasiei oota, seega säilitavad nad oma sageduse ja sobitatakse lähima lugemisega enne ekspositsiooni.

Lugemine ei aegu kunagi, seega ei lähe ükski kaader hilinenud lugemise tõttu kaduma — selle asemel märgitakse:

- **DAQ-i voog on seiskunud** — kasutatakse selle uusimat mõõtmistulemust ning `gap_ms` näitab, kui vana see on (`in_tolerance: false` üle 500 ms).
- **DAQ-i pole, millelt lugeda**— ühendus katkenud (`disconnect()`, `chloros-cli daq pool-disconnect`) või seotud DAQ-ile ei ole juurdepääsu — kaamera viimast näitu kasutatakse uuesti ja kirjeldatakse**selle** eksponatsiooniga: `method`, `retained`, `gap_ms` sellest eksponatsioonist, `in_tolerance` arvutatakse uuesti ning `retained_from_epoch_ns` nimetab ekspositsiooni, millega see esmakordselt sobitati. Kaamera, millel pole kunagi olnud mõõtmistulemust, ei saa ikkagi peegeldusvõimet, ja tulemus näitab, miks (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` või pildi salvestamise ebaõnnestumine).

Chlorosis avatud projekti kausta salvestatud pildid lisatakse selle projekti faililoendisse, nagu seda teeb ka rakendus (mujal salvestatud faile ei muudeta): `CameraSession.capture()`, `ArraySession.capture()` (samuti `smart=True`) ja `capture_cameras_transient()`. Tulemus on kirjas failis `.project_add` (`capture_cameras_transient()`: sõnastiku fail `project_add`) -- `held`, `files` ja tagapõhja `message`; `held` on `True`, kui töötlemisvoor on käimas, ning failid liidetakse kokku, kui see lõpeb. `None`, kui ühtegi faili ei ole projekti lisatud.

`CameraSession.capture()` ja `ArraySession.capture()` tagastavad need tulemusena `.daq_files` ja `.daq_matches`, HTTP-püüdmise vastused (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) sisaldavad `daq_files` ja `daq_matches`, üks iga `.daq` kohta: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` ja `stream_fault` — DAQ-i enda voo viga, mida kontrollitakse ainult lubatud piiridest väljapoole jääva näidu korral. Enne salvestusest saadud peegeldusväärtusele tuginemist kontrollige `in_tolerance`-i selle `.match.json`-is või XMP-s või DAQ-i [`status()`](#daqsensorsession-methods).

`processing` tasemed:

| Tase | Väljund |
| --- | --- |
| `"raw"` | Ühekanaliline Bayer (mono-kaamerad: üks lainepikkus) otse andurist. |
| `"debayered"` *(SDK-i vaikimisi)* | 3-kanaliline värv: kalibreerimispaketi tumeda nihke (DSNU) eemaldamine toorandmetest, seejärel OpenCV servatundlik demosaik – ilma lineaarsuse LUT-ita, tasase väljaga või segamata (mustvalged kaamerad: 1-kanaliline halltoon). Salvestatud TIFF-fail on **RGB**kanalijärjekorras (fotomeetriline RGB, seega kuvab iga pildivaataja tõelisi värve); mälus olevad massiivid, mida `grab()` / `frame_stream()` tagastavad, on OpenCV-i**BGR** järjekorras — pöörake kanalid ümber (`frame[..., ::-1]`), et võrrelda tulemust salvestatud failiga. `chloros-cli process` kirjutab sama tulemuse kui `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm täieliku radiomeetrilise ahela kaudu. Ainult multispektraalne — RGB-kaamerad jäetakse vahele. |
| `"reflectance"` | uint16 0..32768 (Pix4D-valmis); nõuab reaalajas andmekogumise (DAQ) sidumist absoluutse viite saamiseks. Ainult multispektraalne. |
| `"display"` | Täielik ahel, mis vastab GUI-eelvaatele (CCM + WB + gamma vastavalt kaamera profiilile). |
| `"all"` | **Üks fail iga kohaldatava taseme kohta** iga kaamera jaoks (vastab GUI-s valikule „Capture All“ / CLI-i vaikimisi seadistusele). Tagastatud `CaptureResult` sisaldab seejärel ühte kaadri sõnastikku iga `(cam, level)` kohta, kusjuures iga sõnastikus on taseme väärtus; mittekohaldatavad tasemed kuvatakse `.skipped`-is. Kasutatud salvestust lugenud DAQ salvestatakse `.daq`-i lisafailina. |

> **Märkus — vaikimisi seaded erinevad CLI-st.** `ArraySession.capture()`-i vaikimisi väärtus on `processing="debayered"`; käsu `chloros-cli lattice array-capture` vaikimisi väärtus on `processing="all"`. Edastage `processing="all"` selgesõnaliselt SDK-st, et peegeldada CLI- ja GUI-liideste mitmetasandilist salvestamist.

##### Massiivi salvestusfailide nimed

Iga tase salvestatakse oma alamkaustasse kataloogis `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/`, kui `display`, `index/`, `composite/`). Iga faili nimi koosneb kaamera seerianumbrist `SN<serial>-<filter>`: seerianumber, seejärel kaamera mudeli filtri osa — `SN214000533-FOCN` vastab kaamerale `M3C-L87-FOCN`, `SN261902142-F450` mudeli `M3M-L87-F450` puhul — ühendatud mudeliga `-`, seega on nimel samad `_`-ga eraldatud väljad nagu varemgi. Kaamera, mille mudelit ei ole võimalik lugeda, on lihtsalt `SN<serial>`, ning varasemate versioonide salvestustel puudub nimest filter. Failinimi sõltub sellest, mitut taset liikmelt küsiti:

| Liikmelt küsiti | Failinimi | Näide (`reflectance`, vaikimisi `format="tiff"`) |
| --- | --- | --- |
| **Üks tase** (`processing="reflectance"` või ühe elemendiga `levels`) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — taseme järelliidet ei ole, nagu varasemates versioonides, seega toimivad olemasolevad skriptid edasi | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Mitu taset** (`processing="all"` või `levels`, millel on rohkem kui üks kirje) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[Klambermask](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) järgneb toote nimele, `clamp_masks/<product folder>/<product name>_clampmask.tif`: ühetaseme peegeldusandmete salvestamisel kirjutatakse `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, samas kui `processing="all"` kirjutab `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Lõpuliite reegel määratakse iga elemendi puhul eraldi vastavalt taotletud tasemetele, seega vahelejäänud tase (nt peegeldusvõime RGB-kaameral) seda ei muuda. Lugege teed tagastatud kaadritest, mitte ärge neid ise koostage. Ühe kaameraga salvestus (`CameraSession.capture()`) lisab alati taseme: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG / JPEG: võetakse vastu ainult kuvamis- ja indeksitooted

PNG- või JPEG-fail mahutab 8 bitti kanali kohta. Kui küsitakse taset, mida fail ei suuda mahutada, lükatakse taotlus tagasi enne pildistamist, näidates teate, milles nimetatakse taset ja TIFF-vormingut, mis seda toetab:

- **`radiance`**(float32) ja**`reflectance`**(uint16, `32768` = 1,0) — ning seega ka `processing="all"`, mis hõlmab neid mõlemaid. SDK kontrollib seda argumentide puhul: `capture()` tekitab**`ValueError`** ilma päringuta (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` kaameralt, mis edastab rohkem kui 8 bitti** (12-bitine M3C, Mono12 M3M) — tagapõhi lükkab need tagasi, teades iga kaamera bittisügavust: HTTP-vastus 400 koos `error_code` ja `FORMAT_CANNOT_HOLD_LEVEL`, mis tekitatakse kui `ChlorosConnectError` (`CameraSession` / `ArraySession`) või `RuntimeError` (`CameraHandle` / `ArrayHandle`). Ühe kaamera ühetasandiline salvestus `debayered` salvestab 8-bitise kaadri PNG- või JPEG-vormingus ja seda ei lükata tagasi.

Salvestage radiomeetrilised andmed formaadis `format="tiff"` (`ext=".tiff"`); kasutage PNG- või JPEG-formaati `display` ja indeksrenderduste jaoks.

#### Kuidas toimib automaatne säritus

- **Esmalt säritus, viimasena võimendus.** Kui võimendus on automaatrežiimil, jääb see 0 dB juures, kuni säritus on saavutanud lubatud maksimaalse väärtuse.
- **Käsitsi valgustuse korral jääb võimendus automaatrežiimile**, kuni te võimendust muudate või seadistate käsitsi režiimile. Tagasi automaatrežiimile minnes on võimenduse piir sama, mis varem.
- **Väikesed heledad alad ei tumenda stseeni.** Ignoreeritakse peegeldust, mis moodustab kuni 1 % kaadrist, ning teadaoleva kalibreerimismärgi paneelidel kuni 0,5 % paneelist.
- **DAQ-i ekspositsiooni ülempiir kehtib ainult siis, kui kalibreerimismärk on vaateväljas.** See hoiab ära valge paneeli ülevalgustumise; kui paneeli pole näha, valgustab kaamera vastavalt stseenile.
- **Püsiv valgus jääb stabiilseks ja muutunud valgus stabiliseerub sekundite jooksul.** Püsiva valguse korral jäävad valgustus ja selle piir muutumatuks ning pärast suurt valguse muutust stabiliseerub pilt mõne sekundi jooksul. Ülevalgustamine ei piira enam pildistamisparameetreid püsivalt ning huvipiirkonna (ROI) muutumisel leitakse uuesti õige valgustatus.
- **Heleduse sihtmärk jääb saavutatavaks.** See ei tõuse, kui kaamera on valgustuse piirväärtusel ja selle võimendust ei saa suurendada, seega tagasi tulev valgus ei tekita pildis helklemist.
- **Kaamera, mille automaatne valgustatus ei reageeri enam, hoitakse selle hetke valgustuse tasemel.** Eredas päikesevalguses võib kaamera langeda lühimale valgustusaegale ja jääda sinna tumeda kaadri juurde, kuigi kõik selle seaded näitavad normaalset väärtust. Chloros jälgib kaadrit, hoiab kaamerat selle valgustuse jaoks sobival tasemel ning vabastab selle, kui valgustus muutub. Tume või katetud valgussensor seda ei häiri.

#### Valgustamine massiivi liikmetel

LATTICE-kaamera piirab `ExposureTime`-i ühe kaadrisageduse perioodiga. Kui massiiv edastab andmeid, käivitab Chloros iga liiget, millel on `AcquisitionFrameRateEnable` sisse lülitatud, kiirusega, mis on veidi kõrgem kui käivitussagedus, nii et elemendi valgustuse ülempiir jääb veidi alla käivitamisperioodi, **mis tahes huvipiirkonnas**:

- Liikme kaadri periood on triggiperiood, millest on maha arvatud 15 % perioodist (vähemalt 3 ms, maksimaalselt pool perioodist), ja selle valgustuse ülempiir on see kaadri periood, millest on maha arvatud umbes 0,1 ms. Näide: 10 fps käivitussagedus (100 ms) annab liikme sageduseks umbes 11,8 fps ja eksponeerimise ülempiiriks umbes 85 ms.
- **Kui käsitsi seatud `ExposureTime` ületab seda piirmäära, aeglustab süsteem triggert, et see piirmääraga sobiks.** Chloros avab esmalt selle liikme piirmäära vastavale väärtusele ja laseb aeglasemal sagedusel töötada mõne triggerperioodi jooksul (kirjutamine võib võtta mõne sekundi kauem aega), seejärel kirjutab selle sisse, ning päästik ei käivitu kiiremini, kui valgustusaeg lubab. Automaatse sagedusega massiivis taastub varasem sagedus siis, kui kõik liikmed, kelle käsitsi valgustusaeg seda aeglustas, on tagasi automaatsel valgustusel. Väärtus, mida kaamera ise vastu võtta ei saa, lükatakse endiselt tagasi, kusjuures kaamera aktsepteeritav vahemik kuvatakse veateates, ning liikme automaatne valgustusaeg taastatakse, kui see oli välja lülitatud ainult selle kirjutamise ajaks. Enne versiooni 1.3.0 piiras väike huvipiirkond (ROI) iga liikme sagedust anduri enda vabajooksuperioodiga, sõltumata triggereerimiskiirusest: 4,9 ms resolutsioonil 768 × 304, mistõttu ühtegi liiget ei saanud heledamaks muuta.
- Chloros ei lühenda kunagi käsitsi valgustusaega, et see sobiks kiirema triggiga. Automaatne sagedus jääb käsitsi valgustusaegade lubatud piiridesse; kui fikseeritud sagedus on kiirem kui valgustusaeg lubab, jäetakse sellele liikmele piisavalt pikk ülempiir, ning see võtab vastu triggereid mitte kiiremini, kui valgustusaeg lubab. Vähendage valgustusaega või fikseeritud sagedust, kui iga liige peab reageerima igale triggerile.
- **Tagasilugemine:** elemendi sõlmede lugemisel kuvatakse `AcquisitionFrameRateEnable` True ja `AcquisitionFrameRate` elemendi sagedusel; lubamine lülitub välja, kui massiiv peatub.
- **Massiiv omab neid kahte sõlme. Ära kirjuta `AcquisitionFrameRateEnable` ega `AcquisitionFrameRate` massiivi liikmele** — määra massiivikiirus `target_fps` / `set_target_fps()` abil.

Kirjutage liikme ekspositsioon `arr.set_member_settings(serial, exposure_time=20000)`-iga (märksõnad `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` teeb sama). Liige jääb massiivi. Ärge avage massiivi liiget käsuga `connect_camera()`: connect keeldub juba massiivis olevat kaamerat ja `CameraSession`, mida see tagastaks, eemaldab kaamera massiivist, kui ühendus katkestatakse. `ArraySession.status()["member_ae"]` teatab iga liikmeeksponatsiooni ja võimenduse oma viimasest kaadrist.

### Üks projektikirje, iga režiim

Projektikausta sisaldab iga seadme seadeid: `cameras.json` — iga kaamera omad (`cameras[].settings`, ühe kaamera jaoks ja iga massiivi liikme jaoks) ning iga massiivi kogu massiivi hõlmavad seaded (`arrays[].capture`: formaat, ROI, binning, käivitussagedus, andmeedastusmaht) — ning `sensors.json`, iga valgussensori seaded (integratsiooniaeg, kaadri keskmine, piirang). Rakendus, CLI, SDK ja MCP-server loevad ja kirjutavad samu faile samal viisil:

- **Projektiga ühendatud seade salvestatakse sellesse ja salvestab sinna iga muudatuse.** „Projekti alla” tähendab, et projekt on avatud tagapõhis (rakenduses) või — läbi `ChlorosProject` (`open_project()`), mis määrab tagapõhisele oma kausta nime (`project_path`). Kaamera, andurite massiiv või valgussensor, mida projekt veel ei sisalda, lisatakse selle failidesse ühendamisel (`connect_camera()`, `connect_array()`, valgussensori ühendamine, CLI, rakendus, `chloros-mcp`) koos selle seadistustega. Üleminek mis tahes režiimist (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, massiivi seaded, `set_cap()`, CLI, rakenduse juhtnuppude kaudu) salvestatakse hetke pärast selle projekti failidesse. Kaamerale seatud säriaeg, võimendus või AE-sihtmärk salvestatakse kui hoitud, nii et projekti uuesti avamisel säilitatakse need seaded.
- **Projekti avamisel määratakse selle seadmed projekti järgi** igas režiimis: kaamera, massiiv või valgussensor, mis on ühendatud ilma oma seadistusteta, saab projektisalvestatud seaded (`lattice cam-connect` / `array-connect` avatud projekti korral, `chloros-cli project connect`, `open_project().connect_all()`, rakenduse uuesti avamisel). Uue ID all ühendatud massiiv uuendab projekti salvestatud massiivi samade kaameratega; teiste kaameratega massiiv lisatakse selle kõrvale.
- **Rakenduse enda salvestamine ei kirjuta kunagi üle seadme hetkeseadeid** nende vanema koopiaga ning säilitab iga liikme puhul CLI- või SDK-kutsuja poolt salvestatud ümberkirjutused.
- **Projekt säilitab teie poolt määratud pikseliformaadi.** „Array Settings“ (Massiivi seaded) kaudu, `--pixel-format` või `connect_array(pixel_format=)` abil määratud formaat, sealhulgas 8-bitine, salvestatakse ja taastatakse täpselt samasugusena. Kui formaati ei ole määratud, ühendub iga kaamera oma 12-bitise vaikimisi seadistusega (`BayerRG12`, `Mono12` monosensoril). Formaadi nimi tähistab selle bittisügavust; perekond on sensori oma, seega värvikaamerat ei seadistata kunagi monovormingule ega monokaamerat Bayeri vormingule, ja rida `[PixelFormat <serial>] ... written` näitab seda, kui formaat on kaardistatud. 8-bitise salvestuse ekspordid on 8-bitised, sealhulgas peegeldus: ümberskaalitud ja märgistatud selle `PixelScale`-ga.
- Ühegi projekti alla ühendatud seade ei salvestada ei salvestata midagi.
- **Salvestus järgib pärast seda loodud projekti; avamine jätab selle oma projekti juurde.**Kui luuakse projekt — `create_project()`, `chloros-mcp` või rakenduse**Uus projekt**— või duplikeeritakse (rakenduse**Duplikaat**, mis avab koopia), samal ajal kui salvestus kirjutab endiselt varem avatud projekti, jätkub salvestamine ja salvestatud andmed lähevad uude projekti, samasse kohta selle sees (allpool). Olemasoleva projekti avamine — `ChlorosProject.process()` (mis avab projekti taustal), `chloros-cli process`, rakenduse**Ava projekt** — jätab salvestuse jätkuma koos projektiga, kuhu see salvestatakse: salvestus lõpeb seal ja selle failid lisanduvad selle projekti faililoendisse (rakendus peatab iga salvestuse enne projekti avamist).
  - `ArraySession.record()` ja `ArraySession.burst()`: video või iga kaadri sisaldav sarivõtte kaust liigub sinna tervikuna, kui salvestamine peatub. Peatamise tulemusena kuvatakse uus asukoht (`out_dir`, `video_path`) ning sealt koostatakse sarivõtte koondversioon. Kui kõvaketas keeldub faili teisaldamisest, jääb salvestus sinna, kuhu see kirjutati, ja tulemus näitab seda (`relocation_error`). Peatatud sarivõtte töötlemata kaadrid ja `.daq` lisatakse selle projekti faililoendisse, kuhu need lõpuks satuvad; videot ei lisata projekti faililoendisse.
  - `ArraySession.continuous()`: iga pärast muudatust tehtud salvestus salvestatakse uude projekti; enne seda salvestatud salvestused jäävad sinna, kus nad on.
  - `DAQSensorSession.record_start()` (ja rakenduse enda valgusanduri salvestus): `.daq` liigub uue projekti `light_sensor` kausta, kui salvestamine lõpeb; `record_stop()` tagastab uue asukoha ning fail lisatakse selle projekti faililoendisse.

  Salvestus, mis on tehtud avatud projektist väljaspool asuvasse kausta, jääb sinna, kus see on, ning `start_camera_continuous()` / mitme voo ülesanded säilitavad neile määratud kausta.
- **Rakenduses salvestatakse uue projekti loomise järel tehtud salvestused sinna.** Projekti loomine või kopeerimine jätkab kõigi käimasolevate salvestuste ja salvestusprotsesside tööd ning iga pärast seda tehtud salvestus („Salvesta kõik”, „Alusta salvestamist”) salvestatakse uude projekti ja lisatakse selle faililoendisse. Olemasoleva projekti avamine rakenduses peatab esmalt kõik need.
- **Projekti ei liigutata salvestamise ajal.**Avatud projekti teisaldamine teise töökataloogi (rakenduse**Seaded → Töökataloog**; ükski SDK-kõne ei liiguta projekti, HTTP-klient, kes saadab andmeid aadressile `/api/move-project`, teeb seda), lükatakse tagasi HTTP-veaga 409 `PROJECT_RECORDING`, kui massiivi salvestamine või sarivõte, massiivi pidev pildistamine, kaamera pidev / mitme voo töö või valgusanduri salvestamine kirjutab sinna: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

See on eraldiseisev sellest, millised on kaamerate seaded ühenduse katkestamisel (vt [Kaamera mälu](#camera-memory--keep_settings--clear_settings)): projekt seadistab oma kaamerad ise avamisel.

### Kaamera mälu — `keep_settings` / `clear_settings`

Kui massiiv on ühendatud, võtavad selle kaamerad üle massiivi seaded, mida saate seejärel muuta. Kui Chloros vabastab kaamerad — `ArraySession.disconnect()` / `force_disconnect()`, ajutise funktsiooni lõpp, `start_camera_continuous()` peatamine — taastab iga kaamera selliseks, nagu see oli enne ühendamist: selle päästiku ja sünkroniseerimisliini juhtmestik, PTP ja kaamera seaded — `ExposureAuto` / `GainAuto` (ja käsitsi `ExposureTime` / `Gain`), `TargetBrightness` ning AE piirid ja summutamine, valgebalanss, must tase, `PixelFormat`, piksliühendamine ja huvipiirkond (ROI), värviprotsess (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP-d `PtpSlaveOnly` / `PtpDomainNumber`, paketi suurus (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), stimuleerimissagedus (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), tükkandmed (`ChunkModeActive`, iga `ChunkEnable` oma `ChunkSelector` all), triggerväljundi glitšide eemaldamine massiivi abil annab orja sünkroniseerimisliini (`LineFilterSelector` / `LineFilterWidth`) ning andmete kogumise kaadrisageduse (`AcquisitionFrameRate` ainult juhul, kui selle lubamine oli sisse lülitatud: kui see on välja lülitatud, loeb sagedus maksimaalset väärtust, mida taktsagedus võimaldab, mitte seadistust). Kirjutatakse ainult see, mis erineb: kaamera peatus esimesena, taktsagedus ja kaadrisagedus enne säritust, mis vajab oma kaadri perioodi.

Kaamerate kasutamiseks väljaspool projekti kirjutavad kaks argumenti kaamera **enda mällu** – sellega, millega see käivitub, mis tahes tarkvaras (`disconnect()` / `force_disconnect()`, kõik ajutised funktsioonid, `start_camera_continuous()` ja `ChlorosProject.disconnect_all()`; MCP: samad argumendid nagu `disconnect_device` ja ajutised tööriistad; CLI-s: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** salvestab iga kaamera seaded sellisena, nagu need seanssi lõppedes mällu jäid (UserSet2, millest sai käivitusseadistus). Kaamerad jäetakse endiselt vabalt töötama (`TriggerMode` `Off`), sünkroniseerimisliini juhtmestik jääb selliseks, nagu see leiti. Kaamera, millel puudub UserSet2, salvestab seaded UserSet1-i; arvuti mäletab seda, seega Chlorosi massiivi eneseparandusfunktsioon — mis taastab Chlorosi baasjoonest kõrvale kaldunud UserSet1-i — jätab selle puutumata.
- **`clear_settings=True`** taastab iga kaamera mälu tehase seadetele: Chlorosi baasjoonele, millele selle massiivi eneseparandus tugineb — tehase vaikimisi seaded koos `TriggerMode`, `Off`, `TriggerSource`, `Line2`, automaatse särituse, RGB-kaamera kalibreeritud valgebalansiga säilitatakse — UserSet1-is, mis moodustab käivituskomplekti.
- Mitte mõlemad (`ValueError`; tagapõhi vastab 400). Vastuses on `camera_memory`, iga kaamera kohta `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` või `{"cleared": bool, "errors": [...]}` — `force_disconnect()` tagastab selle; ülesande puhul kaasneb sellega `stop()`.

Rakendus neid ei paku.

**Iga väljumistee taastab kaamerad.**Leitud kujul taastamine toimub igal väljumisteel: `CameraSession.disconnect()` (rakenduse kaamera**X**), rakenduse**X** massiivis või mis tahes massiivi elemendil (kogu massiiv vabastatakse), rakenduse sulgemisel (tagapõhjal on kuni 20 sekundit aega, et taastada kõik kaamerad, massiivid ja käimasolevad pidevad salvestused; veel töötav ajutine funktsioon või ühekordne kutsumine peatub järgmises ohutuspunktis, taastab esmalt oma kaamerad kuni 8 sekundit, ning vastab HTTP-koodiga 503 `SHUTTING_DOWN`; endiselt töötav `connect_array()` peatub enne oma järgmist kaamerat ja taastab avatud kaamerad, kuni 10 sekundit, ning tekitab veakoodi `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); ning see, mis saabub sulgemise ajal, ei ava midagi: HTTP 503 `SHUTTING_DOWN`), ning `chloros-cli`-käsk, mis lõpetatakse või mille terminal suletakse (selle töö peatatakse ja kaamerad pannakse esmalt tagasi). Tagapõhi, mis kukub kokku või lõpetatakse sundkorras, ei saa midagi tagasi panna; toite väljalülitamine ja sisselülitamine taastab iga kaamera mällu.**Projekt seadistab oma kaamerad ise.** Kui massiiv ühendub projekti kaudu – rakenduse avatud projekti või `ChlorosProject` kaudu, mis nimetab end tagapõhja (`project_path`) — saab iga liige seaded, mille projekt tema jaoks salvestas: sellele määratud säriaeg, võimendus või AE-sihtmärk (määratud sinu omaks, nii et automaatne säri seda ei muuda) ja selle valgebalansi režiim (Chlorosi korrektsioon; kaamera enda valgebalanss jääb välja lülitatuks), olenemata sellest, mida kaamera mälus hoiab.

### Pildistamisrežiimid ja salvestajad

Massiivi pind peegeldab graafilise kasutajaliidese pildistamispaneeli: ühekordne / järjestikune / intervall / kiireim katikurežiim, pluss kaks salvestajat (reaalajas komposiitvideo ja toores pildiseeria → offline-töötlus).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**on SDK-i pidev/intervall-tsükkel. Kuna puudub `Ctrl+C`, millega seda skriptist katkestada,**peate** edastama `count` ja/või `duration_s` (tsükkel peatub, kui jõutakse kummagi väärtuseni). `interval_s` mõõdetakse iga läbimise algusest (vastavalt graafilisele kasutajaliidesele). Pidevad läbimised (`interval_s=0`) moodustavad ahela `group_seq` → `after_seq`, seega on need massiivisagedusega. Ülejäänud kwargid edastatakse otse `capture()`-ile.
- **`continuous`**on tagapõhja enda pidev salvestus – graafilise kasutajaliidese pidev režiim.**Kasutage seda `capture_repeated()` asemel, kui soovite, et iga käivitus salvestataks kettale kõrge kaadrisagedusega:** `capture_repeated()` teeb teie skriptist ühe HTTP-päringu grupi kohta, samas kui `continuous()` hoiab tagapõhisüsteemis käimas kahte salvestust, ilma et gruppide vahel toimuks edasi-tagasi liikumist. Mõõdetud ROI 512 juures 9-kaameralisel massiivil: 73–77 salvestust sekundis, võrreldes massiiviga, mis avaldab umbes 78. Jätke `capture_repeated()` intervallsalvestuseks (`interval_s`) ja kaadrite jaoks, mida kontrollite ükshaaval. Windowsis tuleb tagapõhja kõrge kaadrisageduse saavutamiseks Microsoft Defenderist välja jätta (vt [1. samm](#step-1--install-the-chloros-platform-package)).
- **`record`** on *jälgimiskvaliteediga*: see salvestab renderdatud reaalajas kombineeritud indeksiga komposiiti. Kui graafilise kasutajaliidese kombineeritud vaade on avatud, salvestab ta selle vaate sellisena, nagu see kuvatakse. Kui ükski vaade pole avatud, renderdab tagapõhi vaate ise: `expr`, kui edastate ühe, muidu massiivi NDVI (punane ja NIR-ribad, mis asuvad kõige lähemal 660 nm ja 850 nm, nagu reaalajas vaade neid valib), kui massiiv on joondatud. Nimetage lainepikkused `expr`-is nii, nagu massiiv neid nimetab: `<band>_<filter>` M3C-kaamera puhul (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` M3M-i jaoks (`Mono_850`); lühike värvinimi (`NIR`, `Red`) toimib ainult siis, kui see on olemas ühel massiivi ribal. Salvestamise ajal avatud GUI-vaade salvestatakse seni, kuni see on avatud. Salvestus, mis ei kirjuta ühtegi kaadrit, ebaõnnestub veakoodidega `error`, `"no-frames: …"` (massiiv võib veel joonduda või ekspositsiooni stabiliseerida), ning tekitab veateate `stop()` (vt [`RecorderHandle`](#recorderhandle)). Üks komposiitsalvesti iga massiivi kohta (tekitab veateate, kui üks on juba töös).
- **`burst` → `build_video`** on *analüüsiklassi*: `burst` kirjutab toorkaadrid + kaadri-põhise manifesti + ühe `.daq` iga eraldi DLS-lugemise kohta `<output>/bursts/<base>/` all pildivõtutsükli täiskiirusel (ilma ahelata, ei exiftool, ei reaalaja vaadet). `build_video` sobitab iga kaadri aja järgi lähima `.daq`-iga ja käivitab uuesti importimispipeliini kiirguse/peegeldusvõime/indeksi ahelat uuesti läbi. `products` on nimekiri `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`-ist (vaikimisi: kombineeritud indeks). `burst().stop()` käivitab automaatselt ka parima võimaliku kombineeritudindeksi koostamise, mis tagastatakse lõpetustulemusena `build_job`-na.

##### Tulemus `build_video`

| Võti | Tähendus |
| --- | --- |
| `success` | `True` ainult juhul, kui kõik taotletud tooted koostati vastavalt taotlusele: vähemalt üks väljund, `skipped`-is midagi puudub, ja puudub `degraded`. |
| `outputs` | Salvestatud videote (ja GIF-ide) asukohad. |
| `errors` | Probleemid tekstina. |
| `skipped` | Üks `{"serial", "level", "reason"}` iga taotletud toote kohta, mille puhul **ei** tekkinud videot — kaamera (selle kaamera jaoks puudub kalibreerimispakett, puudub DLS-näid peegeldusvõime kohta, töötlemisahel ebaõnnestus) või kombineeritud indeksid, mis sisaldavad `"kind": "combined"` ja `serial` `None` (kaamerate seas puudub punase ja NIR-sagedusriba paar, puudub rühm, millel oleks kiirgusvõimsus mõlemal sagedusalal). Iga kirje tekitab `success` ja `False`; ülejäänud videod kirjutatakse ikkagi välja ja loetletakse. |
| `substitutions` | Kaamerad renderdatakse taotletust erineval tasemel ja miks — RGB-kaamera, millelt nõutakse kiirgust/peegeldust, saab oma kuvamisvideo, mille valgebalanss on korrigeeritud, kui kaamera kalibreerimispakett on saadaval. See ei ole viga. |
| `notes` | Teavet sisaldav tekst, mitte vead — nt kaameral, millel puudub kaamerapõhine indeksaväljend, puudub indeksvideo, või RGB-kaamera kuvavideo ei ole valgebalansseeritud (kalibreerimispakett puudub või selles puudub tehase valgebalanss), nimetades kaamera. Ei ole viga. |
| `degraded` | `True`, kui toode on kirjutatud, kuid ei ole see, mida taotleti: offline-kombineeritud indeks on sisseehitatud punase/NIR-normaliseeritud erinevus, seega konfigureeritud indeksavaldist ei hinnata. Tekitab vead `success` ja `False`. |
| `combined` | Kombineeritud indeksi kokkuvõte: kasutatud lainealad, LUT-vahemik, loodud kaadrid. |
| `clamp_masks` | Iga kaadri [segamisvastaste klambrimaskide](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) asukohadkaadrihaaval kalibreeritud TIFF-failide [unmix-clamp-maskide](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) asukohad (`save_tiffs=True`), asuvad `clamp_masks/<level>/` kaustas TIFF-failide `<level>/` kõrval, üks iga TIFF-faili kohta, milles pikseli väärtust piiratud. Videol neid ei ole. |
| `clamp_fractions` | Mida video edastab maski asemel: üks `{"kind", "serial", "level", "frames"}` iga videotoote kohta (`kind` `"combined"` ühendatud indeksi liikmekaamerate jaoks), `frames` nimekiri `[frame, R, G, B]`-dest — selle kaadri pikslite osakaal, mille unmix piiras sagedusribasse, mida selle R / G / B kanal hoiab (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Tagastatakse funktsioonide `ArraySession.record()` ja `ArraySession.burst()` poolt. Kasutage seda kontekstihaldurina, et peatada töö automaatselt, kui scope lõpetab töö, või juhtige seda käsitsi.

| Liige | Kirjeldus |
| --- | --- |
| `job_id` | Tagapõhja töö ID (str). |
| `kind` | `"composite"` (alates `record`) või `"raw"` (alates `burst`). |
| `start_stats` | `start`-kõne poolt tagastatud sõnastik. |
| `result` | `None` käivitamise ajal; lõplik peatamistulemuse sõnastik pärast peatamist. |
| `stats(timeout=10.0)` | Töö reaalajas statistika (kirjutatud kaadrid, tegelik kaadrisagedus, kulunud). |
| `stop(timeout=60.0)` | Salvestaja peatamine; tagastab ja salvestab vahemällu lõpptulemuse (`.result` jääb `None`-iks, kui peatamisnõue ise ebaõnnestus). Ebaõnnestunud salvestuse puhul — mille puhul ei kirjutatud ühtegi kaadrit (`error` `"no-frames: …"`) — salvestatakse tulemus vahemällu `.result`, seejärel tekitatakse viga `ChlorosConnectError`. Idempotentne (teine kutsumine tagastab vahemällu salvestatud tulemuse, ilma viga tekitamata). `with`-ploki vahele jätmine peatab salvestaja ilma veateadet tekitamata, seega kontrollige pärast seda `rec.result.get("error")`-i. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Tagastab `ArraySession.continuous()`. Kasutage seda kontekstihaldurina — `with`-ploki hülgamine peatab salvestamise, kui see veel töötab — või juhige seda `status()` / `wait()` / `stop()` abil. Korraga töötab iga massiivi kohta üks pidev salvestamine.

| Liige | Kirjeldus |
| --- | --- |
| `start_status` | Käivituskõne poolt tagastatud staatuse sõnastik. |
| `result` | `None` käivitamise ajal; lõplik staatusdikt, kui salvestamine on lõppenud (määratud `stop()` poolt või `status()` / `wait()` poolt, mis nägi selle lõppemist). |
| `status(timeout=10.0)` | Reaalajas oleku sõnastik (väljad allpool). |
| `wait(timeout=None, poll_s=0.5)` | Blokeeri, kuni salvestamine lõpeb iseenesest — `max_captures` või `duration_s` on saavutatud, massiiv on tühi, kõvaketas on täis, on tekkinud viga — ja tagasta lõplik staatus. Funktsiooniga `timeout` tagastatakse viimane staatus pärast seda arvu sekundeid, isegi kui protsess veel töötab: vaata `running`. |
| `stop(timeout=30.0)` | Peata salvestamine ja tagasta selle lõplik staatus. Idempotentne (teine kutsumine tagastab vahemällu salvestatud tulemuse). |

Staatusväljad:

| Välja | Tähendus |
| --- | --- |
| `running` | Tõene, kuni salvestamine kestab. |
| `captures` | Sünkroniseeritud rühmad salvestatud. |
| `files` | Failid kirjutatud. |
| `failed_files` | Failid, mida ei õnnestunud kirjutada (nagu kirjeldatud [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Täielikult ebaõnnestunud salvestamiskatsed, sealhulgas need, mida prooviti uuesti (HTTP-viga 503, kuna massiivil ei olnud õigel ajal uut rühma). |
| `skipped_groups` | Massiivi avaldatud rühmad, mida ei salvestatud (salvestamine jäi ajaliselt maha). |
| `captures_per_s` | Keskmine salvestamismäär alates algusest. |
| `elapsed_s` / `started_at` | Seni töötatud sekundid; algusaeg (Unixi epohhi sekundid). |
| `last_group_seq` | Viimane salvestatud sünkroniseeritud rühm. |
| `last_error` | Viimane viga või `None`. |
| `stop_reason` | Kui lõpetatud: `count` (`max_captures` saavutatud), `duration` (jõuti `duration_s`-ni), `stopped` (kutsuti `stop()`), `array_gone` (massiiv lahti ühendatud), `error` (20 järjestikust ebaõnnestumist või tagapõhja poolt tagasi lükatud päring), `disk_full` (fail, mille jaoks kettal ei olnud ruumi). `None` töötamise ajal. |
| `max_captures` / `duration_s` | Algseadistused (`0` = puudub). |
| `defender_cores` | Microsoft Defenderi poolt viimase 10 sekundi jooksul kasutatud CPU-tuumad; `None` kuni tuuma mõõtmiseni ning alati `None` Defenderita arvutis. |
| `defender_warned` | Tõene, kui salvestus on registreerinud, et Defender aeglustab seda (1 tuuma või rohkem). Jäta tagapõhi Defenderi skaneerimisest välja — vaata [1. samm](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Reaalajas kaadrid oma töötlusvoos

Sünkroniseeritud kaadrite edastamiseks oma koodi — masinõppesse, kohandatud vaatajasse, oma salvestusseadmesse — kasuta `ArraySession.frame_stream()`. Iga tik toodab iga kaamera jaoks `GrabResult`: `{serial: numpy array}` iga kaamera kohta, **kõik ühest triggist**, kusjuures selle triggri ID on `.group_id`. See on sama rakendus kui projekti API-s olev `ArrayHandle.frame_stream()`, seega kehtib kõik, mis on kirjas jaotistes [Üks triggereid iga lugemise kohta](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#voogude-joondamine--joondamine) ja [`bit_depth`](#bitisügavus--täpsuse-vahetamine-ribalaiuse-vastu) kehtib mõlema puhul.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Iga väljastatud kogum on uus käivitaja.** `frames.group_id` ei kordu kunagi ühe voo piires; kui nõuate rohkem `fps`-i, kui massiiv töötab, ootab voog järgmist käivitajat, seega töötab see kahe kiiruse hulgast aeglasemal.
- **Puuduvad kaamerad nimetatakse, neid ei täiendata.** Kaamera, mida selle trigeri ajal ei suudetud teenindada, puudub sõnastikust ja on loetletud `frames.errors`-is.
- **See lõpeb järsult.** `FrameFetchError` korral kordub järgmine tsükkel keeldumise korral, kui 60 sekundi jooksul ei ole ühtegi kaadrit või kui 60 sekundi jooksul ei ole uut triggert.**Mida see pakub.**`frame_stream()` edastab terved numpy-kaadrid kohaliku HTTP-ühenduse kaudu ja saavutab maksimaalse kiiruse ligikaudu**170 MB/s**. Sellest piisab toorkaadrite, binnitud või ROI-massiivi reaalajas töötlemiseks, kuid mitte iga trigeri töötlemiseks täisresolutsioonis — et säilitada**iga** trigeri, salvestage kettale käsuga [`continuous()`](#continuoushandle). Mõõdetud nelja kaamera massiivil (FOCN, FRGN, FRGB, F450) pakitud edastusega; iga tik oli täielik ühe trigeri komplekt:

| | Täissensor (massiiv 14,2–14,4 kaadrit sekundis) | 2 × 2 binnimine (massiiv ~33 kaadrit sekundis) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 komplekti/s | 22,2 komplekti/s |
| `frame_stream(processing="debayered")` | 2,3 komplekti/s | 9,3 komplekti/s |
| `frame_stream(processing="display")` | 0,8 komplekti/s | 3,0 komplekti/s |
| Kaamera kohta MJPEG, 4 voogu resolutsiooniga 720p | igaühe puhul 13,5 kaadrit sekundis; iga kaader on seotud käivitajaga | igaühe puhul ~25 kaadrit sekundis; ~45 % käivitajatest on olemas kõigil neljal |

Kasutage `processing="raw"`-i läbilaskevõime ja demosaikimise jaoks oma poolel; `debayered`-i tase eemaldab ka kalibreerimispaketi tumeda nihke, mida `raw` demosaik ei tee.

#### MJPEG-voogud kaamera kohta (ei RTSP)

**RTSP-d ei toetata.** Voogedastusliides on MJPEG üle HTTP-i (`multipart/x-mixed-replace`, üks JPEG osa kohta, igal osal on `Content-Length`), üks URL kaamera kohta:

| URL | Mida see edastab |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-bitine eelvaade, kuni ~30 kaadrit sekundis. Massiivi liikme puhul määravad `max_h` (vaikimisi 720, 64–2160) ja `jpeg_q` (vaikimisi 80) määravad suuruse ja kvaliteedi. |
| `GET /api/camera/<serial>/stream-annotated` | Sama eelvaade, millele on joonistatud kasutajaliidese kihid (zebra, võrgustik, ristjoon, histogramm, tippväärtuste näitamine, punktmõõtur). `ChlorosLocal.camera_stream()` loeb selle teie eest välja. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-bitine peegeldusvõime sagedusriba kohta — vaata [Reaalajas peegeldusvõime voog](#live-reflectance-stream-mjpeg). |

- **Triggeri päised.** Massiivi elemendi puhul sisaldab iga osa kõigist kolmest voost `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` ja `X-Chloros-Timestamp-Ns`. Erinevatest kaameratest pärit osad, millel on sama `X-Chloros-Group-Id`, moodustavad sama käivitaja: koguge osad kokku ja rühmitage need selle päise järgi, et komplekt uuesti kokku panna. Iseseisval kaameral puudub rühm: selle eelvaateosad ei sisalda ühtegi neist päistest (selle peegeldusosadel on tühi `X-Chloros-Group-Id`).
- **Eelvaatevoog jätab koormuse all kaadreid vahele.** See edastab uusima kaadri ja jätab vahele kõik need, mida ei suutnud õigeaegselt saata, seega kõrgetel massiivi kiirustel ei jõua iga triggert iga kaamera vooguni (vt ülaltoodud tabelit). Iga trigeri puhul kasutage `frame_stream()` või `continuous()`.
- **`/api/camera/stream/<serial>` peatub, kui massiivi kombineeritud reaalajas voog on avatud** (`/api/camera/array/stream/<array_id>` — seda avab kasutajaliidese massiivi vaade). Seejärel saadab iga massiivi liikme voog ühe asenduskaadri „Paused (array stream active)”, millel puuduvad `X-Chloros-*`-pealkirjad, ja midagi muud enne, kui ühendatud voog sulgub; seejärel jätkub voog iseenesest. `stream-annotated` ja `stream-reflectance` ei peatu. Sulge graafilise kasutajaliidese massiivi vaade, kui skript loeb kaamerapõhiseid eelvaatevooge.
- **Numbrite, mitte piltide jaoks kasutage `frame_stream()`.**JPEG-vorming on kadudega ja 8-bitine; `frame_stream()` tagastab samad tulemused, kui teekonnas pole koodekit.

### Intervallpildistamine drooniga tehtavate uuringute jaoks

Uuringulennuks on vaja ühte sünkroniseeritud pildirühma iga paari meetri järel, mitte iga kaadrit, mida massiiv suudab toota. Seda juhivad kaks seadet:**`target_fps`**määrab, kui tihti kaamerad kaadri jäädvustavad, ja**`capture_repeated(interval_s=…)`** määrab, kui tihti kaader salvestatakse.**Kaamerad jätkavad pildivoogu salvestamiste vahel.** Iga intervalli järel salvestatakse uusim sünkroniseeritud rühm; vahepealsed kaadrid ei salvestata. Automaatne säritus kohandub iga kaadri puhul, seega on säritus salvestamise hetkeks juba stabiliseerunud. Puudub režiim, kus kaamerad peatuksid kaadrite vahel ja äratataks iga kaadri jaoks uuesti üles: sel juhul peaks iga kaader ootama, kuni voog algab ja säritus stabiliseerub.**Määrake kaadrisagedus vastavalt intervalli vajadustele.**Kui jätate selle automaatrežiimile, töötab kaamerasüsteem kõrgeima sagedusega, mida server ja võrk toetavad. Madalama sageduse valimine vähendab kaamera soojenemist, võrgu koormust ja serveri protsessori koormust, ilma et see mõjutaks salvestatavat materjali. Valige sagedus, mis tagab**2–4 kaadrit intervalli kohta**:

- Iga salvestamine ootab pärast taotlust järgmist kaadrit, seega jõuab salvestatud rühm kohale kuni ühe kaadriperioodi võrra pärast planeeritud aega. Kõrgem kaadrisagedus hoiab selle vahe lühikesena.
- Automaatne säritus reguleerub üks kord kaadri kohta, seega väga madal kaadrisagedus aeglustab seda, kui kiiresti säritus suudab mööduvat pilve jälgida.

**Arvutage intervall lennuplaani alusel välja:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Näiteks 60 m pindala ja 80 %-line edasisuunaline kattuvus tähendab 12 m vahekaugust; kiirusel 10 m/s on see `interval_s=1.2`, ja fikseeritud `target_fps=3` annab intervalli kohta 3–4 kaadrit. Pildistamine toimub ajapõhiselt: SDK-l puudub kaugus- või GPS-põhine käivitamisrežiim, seega hoidke mõõdistusliinidel stabiilset maapinnakiirust.

**Salvestage toorandmed õhusõidukis ja kalibreerige hiljem.** `interval_s` mõõdetakse iga salvestamise algusest, seega kui salvestamine võtab aega rohkem kui intervall, algab järgmine salvestamine hilinemisega ja vahekaugus nihkub. Toorandmete salvestamine koos valgussensori näiduga (`processing="raw"`; näit salvestatakse vaikimisi) hoiab iga salvestamise lühikese ja tagab, et iga kaadrit on hiljem võimalik teisendada peegeldusvõimeks.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**fikseerib sageduse algusest peale;**`arr.set_target_fps(fps)`**muudab seda lennu ajal ilma uuesti ühendamata ning**`arr.set_target_fps(None)`** taastab automaatrežiimi. Kiirus, mis ületab hosti ja võrgu kandevõimet, alandatakse selle ülempiirini — kontrollige tulemuses `clamped` — ning madalaim kiirus on 0,1 kaadrit sekundis.
- **Ärge kasutage `burst()`-i kaadrite hõrendamiseks.** See kirjutab iga kaadri, mida massiiv toodab; see järgib fikseeritud `target_fps`-i, seega on see õige vahend ainult siis, kui soovitakse iga kaadrit.
- **`target_fps` ei paranda vigaseid kaadreid.** Selle väärtuse alandamine vähendab toodetavate kaadrite arvu, mitte iga kaadri saatmise kiirust; GVSP-vigaste kaadrite puhul alandage selle asemel `wire_ceiling_mbps` väärtust (vt [Massiivi tervis](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Ühendamine juba ühendatud massiiviga — `attach_array`

Kui massiiv on juba üles seatud (kas GUI avas selle või eelmine SDK-seanss kutsus välja `connect_array`), kasuta selle asemel uuesti ühendamise asemel `attach_array`, et saada selle käepide. `connect_array` annab selles olukorras alati vea „Camera  <sn>on juba massiivis<id>”, sest `/array/connect`-i POST-kõne massiivi liikme jaoks ei ole idempotentne; `attach_array` loeb `/api/camera/array/list`-i ja teeb vasteid kas array_id või seerianumbrite alusel.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Soovitus: SDK-skriptid, mis töötavad koos töölaua graafilise kasutajaliidesega, peaksid esmalt proovima `attach_array`-i ja kui massiivis pole veel ühtegi massiivi, siis kasutama `connect_array`-i.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Kinnitatud massiiv on laenatud, mitte omandatud.** `attach_array`-ist saadud käepide ei lõpeta massiivi kunagi: seda ei vabasta ei `with`-ploki lõpetamine ega ka selgesõnaline `disconnect()` (selgesõnaline kutsumine trükib stderr-i ühe rea, milles öeldakse, et massiiv jäi töötama). Sama reegel kehtib ka `CameraSession` / `DAQSensorSession` puhul. Seega on GUI-ga ühine kasutamine ohutu nii `with`-iga kui ka ilma:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Kui soovid protsessi tõepoolest lõpetada — näiteks selleks, et ühenduda uue `wire_ceiling_mbps`-iga —, kutsu välja `arr.force_disconnect()`. Ühenduse taastamine pärast tavalist `disconnect()`-i käskluse kasutamist ühendatud käepideme puhul ebaõnnestub veateatega „Kaamera on<sn> juba massiivis<id>“, sest midagi ei vabastatud.

### Võrguanalüüsi abivahend

Kasulik enne massiivi avamist – prognoosib, kas sinu kavandatud seaded sobivad:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

See ei käivita kunagi tagapõhja: kui tagapõhi ei vasta, tekib viga `ChlorosConnectError`, samas kui tagasilükatud päring tuleb tulemusena tagasi. Vale `packed` tekitab vea `ValueError`.

`status` on üks järgmistest: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (muul juhul `error` — sealhulgas tagasilükatud taotlus: pole sisse logitud, `error_code: AUTH_REQUIRED` või SDK-miinimumist madalam pakett, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` tähendab, et taotletud resolutsioon sobib RX-ringiga ainult piiratud käivitussagedusel — säilita resolutsioon ja edasta `target_fps=result["recommended"]["recommended_target_fps"]` kuni `connect_array` (vt [Näide 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` ilmub ainult siis, kui taotletud kaadrit ei ole võimalik edastada ühegi kaadrisagedusega; connect ei vähenda kaadreid kunagi ise, seega rakendage `recommended` ise.

**Kuidas projektsiooni lugeda** (sama mudel kui GUI-paneelis „Array Settings”):

- **Burst (`frame_bytes_total`) summeeritakse iga kaamera kohta vastavalt iga kaamera tegelikule pikseliformaadile.**Mono**M3M**kaamerad edastavad Mono12 (2 B/px) või `pixel_format`-i abil selle sügavuse Mono-formaati (`"BayerRG8"` on `Mono8`, 1 B/px), seega ilma selleta on nelja kaameraga täisresolutsiooniga kaader**~25 MB** kolme mono-kaameraga, mitte ~12,6 MB, nagu eeldab täielikult 8-bitine eeldus. Tagapõhi määrab iga kaamera formaadi kindlaks selle mudeli järgi.
- **Admittance (`burst_fits_nic_ring`) arvestab andmevoo koormusega**, mitte kogu-burst-vs-ring: sim-emit sobib, kui host tühjendab RX-ringi kiiremini, kui kaamerad seda täidavad. 10G host + 1 GbE kaamerad**võimaldavad** täisresolutsiooni isegi siis, kui burst ületab ringi mahtu; 1 GbE host blokeerib (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` on konservatiivne järjestikuse andmete väljavõtmise ülempiir** — `max(readout+emit, N×emit)`, kus iga kaamera väljastus on piiratud 1 GbE kaameraliiniga, sõltumata säriajast. Näiteks ~2,8 kaadrit sekundis 4-kaameralise täisresolutsioonilise 12-bitise massiivi puhul (vastab käitamisaja mõõdetud ~2,7–3,0-le). Täielik mudel: [CLI-viide → Kaamerate kaadrisageduse ja sarivõtte mudel](cli-reference.md#array-fps--burst-model).
- **Pärast pidevat salvestamist näitab projektsioon, milline oli kaadrisagedus.** 30 sekundit või kauem kestnud salvestamine automaatse kaadrisagedusega registreerib selle kaadrisageduse, millele süsteem selle kaameramassiivi suuruse juures sellel arvutil stabiilselt püsis; kaadrisageduse regulaatori lühiajalised katsed kiirema kaadrisagedusega jäetakse välja. Sellest alates `array-connect`&#x27;, `connect_array()` analüüs ja massiivi seaded-paneel näitavad seda kiirust, kui see on mudeli omast madalam, koos sellega, mis seda `cycle_limiter` piiras: `"clock"` (massiiv töötas kiirusel, millele see stabiliseerus, andmete kogumise tsükkel ootas oma käivituskella), `"loop"` (andmete kogumise tsüklil ei jäänud aega), `"capture"` (salvestamine jäi maha ja vaheles jättis rühmi) või `"measured"` (ei ole teada, milline neist). Mudeli näitaja on esitatud `wire_fps_max`-is. Massiiv tõuseb ikkagi selleni, mida andmesidekanal suudab edastada.
- **Ülekoormus (`oversubscribed: true`) tähendab, et andmesidekanal ei suuda massiivi edastada ühegi kaamerate saavutatava sageduse juures** — kaadrisageduse väljad (`achievable_fps_max` / `fps_bright` / `fps_dark`) väärtuseks 0 ning pildide ühendamine ei suuda seda parandada (see vähendab kaadri kohta baite, mitte sünkroniseeritud baite sekundis). Aeglane kaabel üksi seda ei põhjusta: selle asemel langeb kaadrisagedus, ja kui kaameraid ei ole võimalik piisavalt aeglaselt sünkroniseerida, muutub tase `slip-emit-and-capture` (seriaalne). Lahendused on vähem kaameraid, jumbo-kaadrid või kiirem võrgukaart; `max_cams_collision_safe` näitab teie ühendusliini ja kaamerate maksimaalset võimsust. Vastus sisaldab ka koode `aggregate_demand_bps`, `collision_safe_ceiling_bps` ja `per_cam_floor_bps`. Vaata [Ülemäärane tellimus](#over-subscription-the-per-cam-floor).

### Avastamine ja loetlemine

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Mõlemad võtavad vastu ainult märksõnu `backend_url=` ja `timeout=30.0` ning tagastavad sõnastikute loendi. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) võtab vastu ka `auto_start_backend=True`; `list_*`-kutsed ei käivita kunagi tagapõhja.

---

## Smart-AE / Smart-Capture

LATTICE-massiivid käivitavad taustal pideva automaatse eksponeerimise (AE) niipea, kui need on ühendatud, kuid äsja suunatud stseeni konvergeerumine võtab hetke aega. **Smart-capture** on mugavuslahendus: see küsib iga kaamera säriaega, ootab, kuni massiiv on aknas stabiilne, ja käivitab seejärel pildistamise. See on GUI-ekvivalent: töölaua rakenduse „smart” pildistamisnupp kutsub välja sama tagapõhja lõpppunkti.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Kui kasutate `ChlorosProject`-i (järgmine jaotis), saate rohkem reguleerimisvõimalusi:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Smart-AE-režiim on vaikimisi konservatiivne. Karmistage `exposure_tolerance_pct`-i nõudlike radiomeetriliste tööde jaoks; leevendage seda kiiresti muutuvate stseenide puhul, kus soovite lihtsalt „piisavalt lähedast“ tulemust.

---

## DAQ-andurite seansid

Püsiv tagapõhja-pool spektraalsensoritele (DAQ-U USB-ühenduse kaudu, DAQ-M BLE-ühenduse kaudu, DAQ-E Etherneti kaudu). Peegeldab kaamera pinda: nutikas tuvastamine, pooli taaskasutamine, idempotentne ühendamine.

### Nutikas tuvastamine (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Eelistusjärjekord: Ethernet → BLE → USB. Edasta ükskõik milline selgesõnaline vihje, et fikseerida ülekandeviis.

### Fikseeritud ülekandeviis

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` nimetab ühe anduri ja avatakse ainult see andur. Seda võrreldakse juba avatud anduritega, seejärel küsitakse DAQ-E hostinime või IP-aadressi (DAQ-E-lt, mida võrguskaneerimine näeb ainult IP-aadressi järgi, küsitakse lugemisõigustega selle nime, MAC-aadressi ja seerianumbrit, nii et ka need leiavad selle üles), DAQ-M-i BLE-MAC-aadressi ning DAQ-U või DAQ-M-i poolt avamisel edastatavat ID-d (kõik teised kontrollimiseks avatud seadmed suletakse uuesti). Kui seda ei leita, tekitab `connect_daq_sensor` sündmuse `ChlorosConnectError`, loetledes leitud andmed; koos aadressiga (`port=` / `mac=` / `eth_host=`), mis sisaldab teist andurit, tekitab see samuti veateate. See ei pöördu kunagi tagasi selle anduri juurde, mis on nähtav – selleks ongi null-lipu kasutamine.</id></sn></id></sn>

### `connect_daq_sensor()` allkiri

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Juba basseinis olev andur tagastatakse sellisena, nagu see on (laenatud): `integration_time`, `frame_avg` ja `enable_ae` ei kehti selle puhul. Ühendage see esmalt lahti, et avada see oma seadistustega. Erandiks on `cap_id`: laenatud anduril seatakse see `set_cap()`-iga. Kui seda välja jäetakse, kehtib tagapõhja vaikimisi seadistus (vt märkust meetodite tabeli all).

### `DAQSensorSession` meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `status(timeout=10.0)` | Andmepooli sissekande kokkuvõte (voogedastus-/salvestusseisund, lainepikkuse vahemik, kalibreerimise sha, integratsiooniaeg, frame_avg, AE-seisund). DAQ-E puhul, mille voog on vaikseks jäänud: `reconnect` (kui tagapõhja automaatne ühenduse taastamine toimib: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None`, kui voog edastab andmeid), `stream_lost_s`, `reconnects` ja `last_reconnect` ning `record_gaps` käimasoleva salvestuse puhul; `rx_stream_fault` näitab ühenduse taastamise seisundit, kuni saabub esimene spektriraam. |
| `latest(*, n=1, timeout=10.0)` | Tagastab kuni N viimast spektriraami (igaüks `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Jätka / peata voogedastus (käepide jääb avatuks). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Alustab .daq-faili salvestamist. Tagastab faili asukoha. Keeldub DAQ-U/M-i puhul, millel puudub AWS-kalibreerimispakett (DAQ-E on erand). |
| `record_stop()` | Peatab salvestamise. Tagastab `{path, rows, warning, gaps}`; `warning` on `None` puhta salvestuse korral, muul juhul tagastab tagapõhja diagnoosi (näiteks tühi fail või lünk voos), mis väljastatakse ka stderr-i. `gaps` loetleb kõik ajavahemikud, mil voog midagi ei edastanud: `start_utc`, `end_utc` (`None`: endiselt kadunud peatamise ajal), `duration_s`, `reconnected`, `marked_in_file`. Pideva salvestamise korral on see tühi. |
| `set_cap(cap_id, timeout=15.0)` | Määra järgmistest näitudest alates rakendatav piirangukorrektsiooni profiil: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"` või `"none"` (või `None`) palja anduri puhul. Tagastab andurikokkuvõtte, mille `cap_id` loeb tagasi. DAQ-E-S hoiab `as_recorded` väärtust olenemata päringust (selle hajuti on sisseehitatud). Tõstab `ChlorosConnectError` väärtust profiili puhul, mis ei lae. CLI-käsk: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Ainult DAQ-E: lülitab PTP uuesti sisse seadme puhul, mille PTP on välja lülitatud (`status()["rx_ptp_enabled"]` on `False`; seade salvestatakse seadmesse, mistõttu jääb see välja lülitatuks, kuni see uuesti sisse lülitatakse). Tagastab värskendatud oleku. Tekitab veakoodi `ChlorosConnectError` seadmete DAQ-U / DAQ-M puhul või juhul, kui seade ei kinnita. Chloros ei lülita seadet kunagi ise tagasi sisse, kuna väljalülitatud olek võib olla tahtlik. CLI-käsk: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Ainult DAQ-E: seadme enda seaded. `firmware` (`running`, `bundled`, `update_available`), `ip` ja `ip_mode` (`dhcp` / `linklocal` / `static`: mis hetkel töötab), `network` (salvestatud seade: `mode`, pluss `ip` / `mask` / `gateway` staatilises olekus), `network_change_pending` (kehtib seni, kuni seade käivitab salvestatud seadistuse, mille hulka kuulub ka `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` on `None`, kui seadmeversiooni ei õnnestunud lugeda. Kui seadme voog ei jõua sellesse arvutisse, loetakse see uue ühenduse kaudu ja sõnastik lisab `stream_silent` (`True`), `stream_fault` ja `reconnect`; seade, mis ei vasta, tekitab veateate. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E ainult: DHCP (`"auto"`) või staatiline aadress. Seade taaskäivitatakse seadistuse rakendamiseks ja ühendatakse uuesti; käesolev seanss jätkub ja `address` järgib seadet. `reboot=False` salvestab selle ainult. Vaata [DAQ-E seadme seaded](#daq-e-device-settings). CLI-käsk: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Ainult DAQ-E: edastab andmevoogu sisselülitamisel, kui tarkvara pole ühendatud (salvestatud seadmesse). Näitab kinnitatud seadet. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Ainult DAQ-E: seadme taaskäivitamine ja ühendamine varasemate seadistustega. CLI-käsk: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Ainult DAQ-E: laadige selle Chloros-versiooniga kaasas olev püsivara vanemat versiooni kasutavale seadmele ja ühendage see seejärel uuesti (umbes minut; ärge lülitage seadet välja). Tagastab `updated`, `fw_before`, `fw_after`, `bundled` ja `warning`, kui seade taastub aeglaselt või taastub teise versiooniga; `updated` on `False` seadme puhul, mis on juba ajakohane. Tekitab veakoodi `ChlorosConnectError` seadme puhul, mille versiooni ei ole võimalik lugeda. CLI-käsk: `pool-update-firmware`. |
| `disconnect()` | Vabastamine reservist. Ei-op ei toimu seotud, kuid mitte omandatud käepidemete puhul. Selle sessiooni poolt avatud anduri puhul on see `force_disconnect()` ning tekitab veateate `ChlorosConnectError`, kui vabastamist ei kinnitata (`with`-ploki väljumisel ei tekita kunagi viga). |
| `force_disconnect(timeout=30.0)` | Vabasta andur sõltumata sellest, kes selle avas — näiteks andur, mille kukkunud skript jättis reservi ja milleni uus skript pääseb ainult lisatud käepidemena, mille `disconnect()` ei tee midagi. Teise kliendi anduri lõpetamine lõpetab kõik selle poolt tehtavad salvestused. Tagastab tulemuse, kui tagapõhi vabastamist kinnitab või kui pole midagi vabastada (andur ei ole andurite reservis või tagapõhi ei tööta); vabastamine, mille ajalimiit lõppes või mille tagapõhi keeldus, tekitab vea `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atribuudid: `sensor_id`, `model`, `transport`, `address` (järgneb DAQ-E-le, mille `set_network()` ümber adresseeris), `info` ja `backend_url`.

> **Nimetage kate.** DAQ-U, DAQ-M ja DAQ-E tarnitakse ilma kondensaatorita ning Chloros ei suuda tuvastada, kas see on paigaldatud: ilma `cap_id`-ita ühendatud andur saab väärtuseks `sunshine_cosine`, eeldades, et selle salvestused näitavad, et see on tehtud (`cap_id_source=auto_default`). Kõnealuse korrigeerimise korral näitab kattega varustamata DAQ-U umbes 30 korda suuremat väärtust, seega edastage `cap_id="none"` kattega varustamata anduri puhul (või `set_cap("none")`) ja muul juhul paigaldatud korgi profiili.

### DAQ-E-S — fikseeritud hajuti ja kaldenurga andur

**DAQ-E-S** on DAQ-E, millel on kaks tarkvara seisukohast olulist erinevust. Kõik muu — seadme tuvastamine, ühendamine, PTP, salvestamine ja allpool toodud seadme seaded — toimib täpselt samamoodi kui DAQ-E puhul.**Selle hajuti on püsiv, seega ei vaja see katekorrektsiooni.** Sunshine-tüüpi hajuti on sisse ehitatud ja oli seadmel juba siis, kui mõõdeti selle tehase kalibreerimist, mistõttu kalibreerimine näitab juba õiget allapoole suunatud kiirgustihedust. Chloros seab oma piirangu väärtuseks `as_recorded` (ilma profiilita) olenemata sellest, mida küsitakse: selle nimeline piirang (`sunshine_cosine`, `fov_*`, `none`) ignoreeritakse koos veateatega ning selle salvestustesse talletatakse `cap_id_source=model`, mistõttu eeldatava piirangu hoiatus selle puhul kunagi ei käivitu. Ärge rakendage selle andmetele ka käsitsi piirangut: ülemine `sunshine_cosine` näitaks umbes 11 korda liiga kõrget väärtust.

**Sellel on 3-teljeline kiirendusmõõtur.** Päikesekiirguse hajutaja koosinuskarakteristik kehtib ainult siis, kui see on horisontaalselt, seega näitab kallutatud andur valet kiirgustihedust – ja peegeldusvõime võtab vea üle –, ilma et muid sümptomeid esineks. Seetõttu sisaldab iga DAQ-E-S-i näid oma asendiandmeid:

| Väli | Tähendus |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Nurgad kraadides. `None`, kui püsivara märgib nurga määratlemata olevaks (vaba langemine, rikkis telg). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Kiirendus millig-des. `mag_mg`, mis erineb oluliselt väärtusest ~1000, tähendab, et seade oli liikumas ja nurgad ei ole selle proovi puhul olulised. |
| `cal_applied` | Seadistatud: teljed ja nurgad on difuusori koordinaatsüsteemis (rakendatakse seadme paigalduskalibreeringut). Tühjendatud: toores plaadi koordinaatsüsteem, mis võib näidata kuni ~2,3° kallakut, mis ei ole tegelik. |
| `sample_age_ms` | Kiirendusmõõturi proovi vanus spektri saatmise hetkel. |
| `temp_raw` | Kalibreerimata kiibi temperatuuri **trend** — ainult erinevused, mitte kunagi termomeetri näit. |

`DAQSensorSession.latest()` tagastab need iga raami `imu` võtme all (`None` seadmetel DAQ-U, DAQ-M või tavalisel DAQ-E-l). DAQ-E-S-seadmest salvestatud `.daq` salvestab samad väärtused oma `imu_*` veergudesse (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) iga spektri kõrvale, nii et lugeja saab eristada pilve kolksunud statiivist. Kõigil teistel mudelitel on need veerud tühjad.

Asendiandmevoog edastab andmeid oma väljadele `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), mis on spektriandmetest eraldatud, nii et kiirendusmõõturi riket ei loeta kunagi salvestuse lüngaks. `discover_daq_sensors()` ja seansi `model` näitavad `DAQ-E-S`.

### DAQ-E seadme seaded

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Sellel arvutil peab olema staatiline aadress.** `set_network` tekitab veateate `ChlorosConnectError`, kui seadme alamvõrk ei sisalda selle arvutiaadressi anduri võrgus: pärast taaskäivitamist ei pääse keegi andurini, et seda tagasi muuta. Laske `force=True` läbi ainult siis, kui sellele arvutile määratakse uus aadress või suunatakse see uude alamvõrku.
- **`set_network`, `reboot` ja `update_firmware` taaskäivitavad anduri** ning tekitavad veateate `ChlorosConnectError`, kui andur salvestab, ja kui see ei taastu `comeback_timeout` sekundi jooksul. Anduri ID ja `daq-e-<id>.local` hostinimi ei muutu kunagi.
- **Vahetult enne taaskäivitamist alustatud salvestus suletakse taaskäivitamise käigus ja sellele antakse nimi.** Kui andur taastub, sisaldab tagastatud sõnastik `recording_closed` (faili asukoht) ja `warning`, mis seda kinnitab. Kui andur ei tule tagasi või püsivara edastamine ebaõnnestub, näitab seda hoopis `ChlorosConnectError`: selle sõnum lõpeb hoiatusega ning `e.recording_closed` / `e.warning` on määratud (mõlemad `None`, kui ühtegi salvestust ei suletud). Alustage salvestamist uuesti, kui andur on tagasi.
- **Ühendatakse ainult andur ise.** Pärast taaskäivitamist peab seade vastama, kus ta asus -- ja pärast `set_network("auto")` ka oma `.local` nime all -- oma nime, hostinime ja mudeliga, kasutades selle võrgukaardi (MAC) aadressi, mis tal oli; `address` muutub vastanud IP-aadressiks. Nime all antud vastus loetakse kehtivaks ainult selle MAC-aadressiga, seega arvutist, mis jõuab andurini ruuteri kaudu, või WSL2 vaikimisi NAT-võrgustikust ei ühendata DHCP poolt ümber paigutatud andurit uuesti (`ChlorosConnectError`; ühendage see uuesti selle uue aadressiga).
- **Püsivara uuendamiseks peab andur uuesti ühenduma.** `update_firmware()` kutsub seadet üles ja seade avab pildi allalaadimiseks TCP-ühenduse tagasi sellele arvutile operatsioonisüsteemi valitud pordil. Tagapõhi kuulab seadmega ühendust võimaldavat adapterit (kõikidel adapteritel, kui ei ole võimalik kindlaks teha, milline see on) ja teenindab ainult seda seadet: kõik muud ühendused lükatakse tagasi (`[DAQ-FW] … refused a firmware connection from <ip>` tagapõhja väljundis), kuni seade ootab. Tulemüür, mis blokeerib ühendust, põhjustab värskenduse ebaõnnestumise veaga `device never connected back on TCP <port>`. Windowsi installija lisab selle jaoks reegli (vt [1. samm](#windows-exe)); teises tulemüüris tuleb lubada sissetulevat TCP-liiklust backendile anduri alamvõrgust.
- **WSL2-s kasutage peegeldatud võrgustikku** (`networkingMode=mirrored` ja `.wslconfig`): vaikimisi NAT-režiimis lükatakse tagasi kõik staatilised aadressid (WSL-i enda aadress ei asu anduri võrgus), voog ei jõua WSL-i ja püsivara uuendamine ei saa lõpule viia.
- Graafiline kasutajaliides (GUI) säilitab oma ühenduse anduriga: skriptist käivitatud taaskäivitamine lõpetab GUI seansi anduriga. GUI-l on samad seaded (vahekaart „Valgusandurid”, anduri seaded, „Võrk”).

### Avastamine — ühendamiseks vajaliku aadressi leidmine

`discover_daq_sensors()` otsib USB / BLE / ETH-võrgust andureid, mida *võiks* avada. See on `discover_lattice_cameras()`-i DAQ-vastand ning ainus viis saada **DAQ-M-i BLE MAC-aadress** — DAQ-E-l on hostinimi ja DAQ-U-l COM-port, kuid MAC-aadressi ei ole seadmele trükitud ega ole see ka operatsioonisüsteemis loetletud.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Väli | Kirjeldus |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-port / BLE MAC / hostinimi — edasta `connect_daq_sensor`-ile kui `port=` / `mac=` / `eth_host=`. |
| `display` | Inimloetav silt. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, või `None`. Ethernet-andur on `DAQ-E-S` või `DAQ-E`, nagu ta end skannimise ajal päringu korral teatab (ainult lugemiseks) (`DAQ-E`, kui ta ei vasta õigeaegselt). USB-jadaporti väärtus on `DAQ-U`, kui selle sillakiip on selline, mida DAQ-U kasutab (FTDI, CH340, CP210x, Prolific) — vihje porti kirjeldusest, mitte sond – ning `None` mis tahes muu USB-seadme puhul, mis on ikkagi loetelus, nii et saad seda edastada kui `port=`. Jadaport, millel puudub USB-tootja ID ja selline sillakiip (emaplaadi UART, nagu näiteks Linuxi `/dev/ttyS*`, Bluetoothi jadaliides, Intel AMT), ei saa olla DAQ-U ja seda ei ole loetelus. |
| `extra` | Transpordiviisi andmed (BLE reklaamitud nimi; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Tühjad väärtused on välja jäetud. |

| Parameeter | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `transports` | kõik kolm | Skaneerimist piirav jada (või CSV-string). Tasub edastada, kui teate, mida soovite — BLE on aeglane lüli. |
| `scan_timeout` | 5 | Skaneerimisaken transpordi kohta sekundites; tagapõhi piirab seda vahemikku 1–20. |
| `timeout` | 60,0 | Kogu kõne HTTP-piirang (nagu mujalgi SDK-s). |
| `auto_start_backend` | `True` | Käivitab kohaliku tagapõhja, kui ükski ei ole käimas. Kaugjuhtimisega `backend_url` puhul ei käivita kunagi. |

> **Poolis juba avatud andurid ei kuvata.** Ühendatud BLE-perifeeria lõpetab reklaamimise ja avatud COM-porti ei saa uurida, seega loetleb avastamine seda, mis on *ühendamiseks saadaval*. Vahetult pärast ühenduse loomist on oodata tühja tulemust — kasuta `list_daq_sensors()`-i selle jaoks, mis sul juba olemas on. Transpordid, mille skaneerimine ei saa töötada (bleak / zeroconf pole installitud), jäetakse vahele, mitte ei tekitata viga, seega saab Bluetoothita arvuti ikkagi oma USB- ja ETH-vastused.

### Nimekiri

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Milline DAQ toidab millist kaamerat — `daq_bindings()`

Pildi peegeldusvõime ja selle salvestamiseks kasutatav `.daq` kasutavad selle kaamera või massiiviga seotud DAQ-i, muidu kasutatakse tagapõhja uusimat DAQ-i. Seda määravad GUI valgusanduri valik ja `chloros-cli daq pool-bind`; samuti:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` tagastab `{"cameras": {serial: b}, "arrays": {array_id: b}}`: iga `b` sisaldab `binding` (määratud väärtus), `source` (`project`, projekti `sensors.json` või `runtime`), `known` / `known_as` (kas kaamera või massiivi võtme nimi on Chlorosile teada ja kuidas: `connected`, `project` — avatud projekti `cameras.json` — või kaamera puhul `seen`: varem sellel arvutil avatud; `known: False` võtme puhul, mis midagi ei nimeta) ja `resolved` (pooliandur, millega see praegu seotud on: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Käitamisaja seos on seansile tähtsam kui projektisessiooni ajaks; projektifaili ei muudeta. Kaamera, massiivi või DAQ-seadme lahtiühendamine tühistab kõik selle riistvaraga seotud seosed, olenemata sellest, kes need seadistas (massiivi liikmed seotakse massiiviga).

`set_daq_bindings()` seob ainult ühendatud kaamera, mis on avatud projektis või varem sellel arvutil avatud, ning ühendatud massiivi või avatud projektis oleva massiivi (või juba seotud võtme). Iga muu võti tekitab `ChlorosConnectError`, nimetades seda (`400`, `code`, `UNKNOWN_BINDING_TARGET` koos võtmetega `unknown`), ning selles kutses **ei seotud midagi**. `None` tühistab kõik võtmed.

### Ümbritseva keskkonna + objekti peegeldusvõime — `light_sensor_reflectance()`

Kaks ühendatud andmekogumisandurit (DAQ), millest üks vaatab üles taeva poole (ümbrus) ja teine alla sihtmärgi poole (objekt), annavad sihtmärgi peegeldusvõime ja taimestiku indeksid oma viimaste näitude põhjal: GUI-s asuv paar „Ambient + Object“ ning `chloros-cli daq reflectance` ilma andureid teist korda avamata. Mõlemad peavad olema ühendatud ja andmeid edastama.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Lained.** Iga laine peegeldusvõime on objekti laineüle ümbritseva valguse. `daq_band_presets(filter_code=None)` loetleb lainepikkuste komplektid: üks iga LATTICE M3C filtri kohta (`RGN`, `OCN`, `NGB`, kaamera enda lainepikkusmudel, seega on DAQ-NDVI ja kaamera NDVI sama suurus) ning `GENERIC` lairiba. Selle `suggested` on eelseadistus, mis vastab `filter_code`-ile, muidu esimese ühendatud kaamera filtri (`suggested_from` näitab, milline).
- **Tagastab** `bands` (iga sagedusriba kohta `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (sisseehitatudlisad), `custom` (igaüks koos `value` või `valid: False` ja `error`), `invalid_bands`, `reflectance_type`, `preset` ja kahe näidu `timestamps` (`ts_ns`). Vahemik, mida ümbritseva keskkonna andur ei suutnud mõõta (väljaspool mõõtevahemikku, müra tasemel), loetakse kehtetuks koos põhjendusega ning kõik indeksid, mis seda vajavad, jäetakse välja ega arvutata kunagi nulliga.
- **Näiv või suhteline.** Skaalast sõltuvad indeksid (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) vajavad absoluutset suhet, seega kuvatakse need ainult siis, kui mõlemad andurid on võimsuskalibreeritud (kummagi puhul on `calibration_bundle_sha`): `"Apparent Reflectance"`. Muidu on tulemuseks `"Relative Reflectance"`, mis sisaldab ainult skaalast sõltumatuid indekseid.
- **Kaks erinevat lainepikkuste võret** võrreldakse nende kattuvuse alusel: ümbritseva keskkonna anduri punktid asuvad objekti ulatuses, objekti spekter interpoleeritakse neile.
- **Kaks andurit, mitte kunagi üks.** Ühe anduri nimetamine mõlemaks (sama ID, mis tahes kirjapildis, mida andmebaas aktsepteerib) tekitab vea `ChlorosConnectError`: anduri näid on enda suhtes igal sagedusalal 1,0, mis ei mõõda midagi. Ka tagapõhi lükkab selle tagasi (`POST /api/daq/indices` vastab `400`, `code` vastab `SAME_SENSOR`-ile, kui `ambient_sensor_id` ja `object_sensor_id`, mida SDK saadab, on üks ühendatud andur, nt selle ID ja selle port).

### Ühine kasutamine CLI-ga ja teiste skriptidega

Tagapõhja DAQ-andurite koond on jagatud kõigi masinal olevate CLI- ja SDK-klientide vahel. Kui andur on juba koondis – avatud `chloros-cli daq pool-connect` või mõne teise skripti poolt –, siis `connect_daq_sensor(port="COM3")` kutsumine Pythonist tagastab laenatud käepideme: selle `disconnect()` on tühitoiming, seega teie skript ei võta andurit ära kliendilt, kes selle avas, kui skript lõpetab töö. Sama kehtib ka funktsiooni `sensor_id=` puhul (tüübitud `:` või `-`, mõlemad sobivad), kui DAQ-E puhul, mida nimetatakse selle IP-aadressi järgi, kui see avati hostnime järgi (või vastupidi), ning null-lipuga nutika tuvastamise puhul, kui tagapõhi vastab anduriga, mida ta juba hoidis — `with`-plokk vabastab alati ainult anduri, mille ta ise avas.

Chlorosi graafiline kasutajaliides **ei** kuulu sellesse reservi: selle DAQ-paneel haldab omaenda ühendust anduriga, seega `connect_daq_sensor` ei laena kunagi andurit, mille graafiline kasutajaliides on avanud. Ärge kasutage sama andurit samaaegselt nii kasutajaliidese kaudu kui ka skripti kaudu. DAQ-U või DAQ-M, mida kasutajaliides hoiab, ei avane tavaliselt, kuid DAQ-E võimaldab teist seanssi: skripti ühendus muudab seadme andmekogumise seadeid GUI all ning selle `disconnect()` peatab seadme voo, mis lõpetab GUI reaalaja vaate ja kõik selle poolt tehtavad salvestused. Sulge andur esmalt GUI-s.

### PTP-grandmaster — `time_sync_status()`

Tagapõhisüsteem käitab PTP-grandmasterit, millest LATTICE-kaamerad ja DAQ-E-seadmed saavad oma ajamärgised. See käivitub nõudmisel: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` ja `connect_all()` käivitavad selle, kui see ei tööta — sealhulgas ühenduse loomise katse, mis seejärel ebaõnnestub, näiteks kui `connect_all()` üritab ühenduda DAQ-E-seadmega, mida pole olemas — ning sama teevad ka `test_array_sync()` ja iga käsk, mis avab kaamerad ühe käsu jaoks (`capture_cameras_transient()`, `start_multi_camera_stream()`, joondamiskutsed, `capture_live_index()`).

- **Kus see teavitatakse.** Kaameratega ühendatud traadiga võrguadapteritel (sealhulgas `169.254.x.x` link-local-aadress), muul juhul vaikimisi marsruudi adapteril, kui see on traadiga ühendatud. Mitte kunagi Wi-Fi-võrgus ega virtuaalsel adapteril (Tailscale, WSL): nende taga ei ole ühtegi kaamerat ega DAQ-E-d. Kui sellist adapterit pole ühendatud, teeb ta kuulutusi tühjale (`health` `NO_SOCKETS`) ja seob end järgmise kaamera või DAQ-i ühendamisel selle adapteriga.
- **Pärast ühenduse katkemist.** Kui kaameraadapter kaotab ühenduse ja saab selle tagasi (kaabli tõmbamine, lüliti vahetamine, USB-adapteri uuesti loendamine), ehitab grandmaster oma pistikupesad ise üles, ilma taaskäivitamiseta, ja kontrollib uuesti kõiki ühendatud DAQ-E-seadmeid, taaskäivitades need, mille ühendus oli katkenud. Samuti ehitab ta ühendused uuesti üles, kui kõik kaamerad ja DAQ-E-seadmed, mida ta teenindas, jäävad 15 sekundiks vaikseks.
- **Kas ühendus on katkenud?** Loe `time_sync_status()` (`GET /api/time-sync/status`). `state` näitab ainult, et grandmasteri protsess töötab; `health` näitab, kas selle paketid väljuvad: `OK` (viga ei tuvastatud), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (mõned adapterid ei tööta) või `PEERS_SILENT` (kaamerad ja DAQ-E-seadmed, mida see teenindas, lõpetasid vastamise), koos lausega `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` loevad ainult väljasaadetud sõnumeid; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` ja `last_delay_req_age_s` kirjeldavad neid, mis ei lahkunud, ning `rebinds` / `last_rebind` taastamisi. `sync_sends_slow` loendab sünkroniseerimisi, mille saatmine kestis üle 200 us, ja `sync_send_max_us` on neist kõige pikem; iga Follow_Up’i algusaeg arvestab maksimaalselt 200 us selle saatmisest. Grandmaster töötab omaette protsessis (`own_process`, `process_pid`), Windowsis kõrge prioriteediga (`process_priority`), nii et hõivatud backend ei saa viivitada ajamärgistustega ega liigutada kõigi kaamerate kella korraga; kui seda protsessi ei ole võimalik käivitada, töötab see nagu varemgi backendi sees (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funktsioon | Mida see teeb |
| --- | --- |
| `time_sync_status()` | Grandmasteri seisund, tervis, kella identiteet, prioriteedid ja sõnumite loendurid (ülal). `chloros-cli time-sync status`. |
| `time_sync_peers()` | PTP-orjad, mida ta kuuleb (kaamerad, DAQ-E-d): `peers`, iga `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Iga ühendatud massiivkaamera `PtpStatus`, `PtpOffsetFromMaster` ja `PtpMeanPathDelay` (kaameral, mille püsivara ei sisalda PTP-sõlmi, on ainult seerianumber). `time-sync cameras`. |
| `restart_time_sync()` | Peatab ja taaskäivitab grandmasteri; pärast seda kontrollitakse uuesti iga ühendatud DAQ-E PTP-d (`daq_ptp_rechecked`). Tekitab `ChlorosConnectError`, kui see ei taastu. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Selle PTP prioriteedid (0–255, madalam võidab parima meistri valimised) võrgus, kus on veel üks PTP-meister. Vajab, et see töötaks. `time-sync set-priority`. |

Igaüks võtab `backend_url=` ja `timeout=` ning käivitab kohaliku tagapõhja, mis ei ole käivitatud.

---

## Kaamera utiliidid

Mida `chloros-cli lattice` teeb lisaks ühendamisele ja pildistamisele, mis on lihtsad funktsioonid `chloros_sdk`-is (moodul `chloros_sdk.camera_utils`). Kumbki kutsub välja tagapõhja marsruuti, mida CLI-käsk kasutab, koos CLI reeglitega, seega käituvad need kaks sarnaselt. Need võtavad vastu kaamera **seerianumbri**, mitte seansi, ning samu `backend_url=` / `timeout=` märksõnu nagu smart-connect-kutsed; kui tagapõhi ei tööta, käivitatakse see, nagu teeb ka `connect_camera()`. Keeldumise korral tekitatakse `ChlorosConnectError` koos tagapõhi põhjendusega.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funktsioon | Mida see teeb |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Ekraani värviprofiil: `raw`, `linear`, `natural`, `enhanced` või `custom_temp` ning selle värvitemperatuur (piiratud vahemikuga 2000–10000 K; loetakse, kui profiiliks on `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Ekraani küllastus ja kontrast, mõlemad vahemikus 0–200 % (100 = muutmata); `reset=True` seab mõlemad tagasi väärtusele 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Vähendab ekraani renderdamisel kõige heledamaid ja valgeimaid piksleid: -100..0 (lõigatakse nullile lähemale, seejärel piiratakse; 0 = välja lülitatud). |
| `camera_white_balance_once(serial, settle_s=None)` | Ühekordne valgebalanss: kaamera edastab lühikest aega pilti, viib pildi kokku ja tagastab `red_ratio` / `blue_ratio` (`ratios_read` Vale, kui neid ei saa edastada). See säilitab leitud valgebalansi; kõik muud seadistused, mida avatud fail kirjutas, jäävad muutmata. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (pidev) või `"off"`; või fikseeritud `red` / `blue` tasakaalu suhtarvud, mis lülitavad automaatrežiimi esimesena välja. Suhe, mida kaamera ei aktsepteeri, on nimetatud `errors` all. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Kaamera eelseadistused ja ühe rakendamine: kaamera säilitab seadistused, mida eelseadistuse nimed tähistavad (vastuses on `settings`, kuna kaamera loeb need pärast, nt kaadri sagedus, millega see töötab; `None` väljale, mida eelseadistus ei nimeta), ning kõik muud seaded, mida selle avatud fail kirjutas, taastuvad leitud kujul. `"default"` taastab kaamera tehasevaikimisi seaded (tehase seaded vabalt töötava automaatse säritusega, andmeedastuskiiruse piirang ja defektide korrigeerimine välja lülitatud, RGB-kaamera kalibreeritud valgebalanss säilitatud), mida ei salvestata kaamera mällu: vastuses on `factory_defaults: True` ja kõik `settings` väljad. `camera_presets()`-is on `default` võrdne `{factory_defaults: True, description}`-iga. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Filtrimudelid ja ühe mudeli üksikasjad (sagedusribad, spektraalvastuse tingimusarv). Kaamera filtrimudel loetakse selle mudelistringist; seda ei seadistata käsitsi. |
| `camera_firmware_status(serial=None)` | Iga kaamerapõhinev püsivara: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` ja `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Laadige selle Chloros-koostise püsivara ühele kaamerale: tehase algseadistuste taastamine, püsivara laadimine, taaskäivitamine, Chlorosi baasseadistused kirjutatakse ümber; kaamera kalibreering säilib. Kestab mõne minuti; kaamera peab jääma voolu alla. `force=True` laadib praegusele kaamerale uue püsivara. |
| `camera_power_probe(serial)` | Kaamera toite- ja soojusandurid ning selle temperatuur. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Kaamera madala energiatarbimisega ooterežiim (kaamera jääb magama, kuni ta üles äratatakse: selle ühenduse läbilaskevõime piirang on sisse lülitatud (`DeviceLinkThroughputLimitMode` `On`) miinimumtasemel, energiasäästurežiim on sisse lülitatud, kui kaameral see olemas on), koos temperatuuriga enne ja pärast; äratamine taastab ühenduse sellisena, nagu see oli uinaku alguses (piirang, režiim ja paketi viivitus `GevSCPD`, mida kaamera hoiab väärtusel 0, kui piirang on sisse lülitatud; andmeid säilitatakse selles arvutis, nii et ka hilisem tagapõhja äratamine leiab need üles), või kaamera puhul, mida see arvuti ei pannud magama, kuid mis näib olevat magamisrežiimis (piirang on põrandatasemel sisse lülitatud), kus piirang lülitatakse välja ja `GevSCPD` jäetakse väärtusele 0 tasemel 80 (tehase seaded); piirang, mis on seatud miinimumist kõrgemale, jäetakse muutmata. Säilitatakse ainult toiteolek; kõik muud seaded, mille avatud kirjutamine on leitud, taastatakse leitud kujul. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Salvestada kaadreid, kuni MAPIR-kalibreerimissihik on vaateväljas, ja kohandada selle paneelidelt saadud kanalipõhiseid koefitsiente: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Selle arvuti GigE Visioni Ethernet-adapterid (MTU, ühenduse kiirus, jumbo-kaadrid) ja kõik `issues`; `estimate=True` lisab kaadrisageduse ja ribalaiuse, mida `cameras` kaamerad saaksid. |
| `fix_network_settings()` | Rakenda GigE Visioni adapteri seaded selle arvutikaameraadapterile. See muudab adapteri seadeid ja töötab ainult siis, kui Chloros töötab administraatori õigustega; muidu kuvab see leitud andmete logi. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Korrigeeri kaustas olevate piltide vinjetti, kasutades iga pildi puhul selle objektiivi tasapinnakaarti: `processed`, `lens_models` (objektiiv, mille jaoks iga faili korrigeeriti), `skipped` (koos põhjendustega). Ilma `lens_model`-ita pärineb iga faili objektiivi andmed selle EXIF-mudelist; `lens_model` kehtib failide puhul, mille mudelinimes ei ole kaamerat nimetatud, ning fail, mille mudel on sellega vastuolus, jäetakse vahele. Ainult failid. |
| `read_dls(mac, filter=None, camera_serial=None)` | Üks DLS-i (Bluetoothi allapoole suunatud valgusandur) mõõtmistulemus: spektri, sagedusribade kaupa kiirgustiheduse ja sagedusribade tugevused kaamera enda vastuse kaudu, kui on antud `camera_serial`. `discover_daq_sensors(transports=["ble"])` leiab MAC-aadressi. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Host-seadme kaamerate riistvara ja soovitatav konfiguratsioon (`report_text`) ning süsteemi CPU / GPU / PyTorch-käivituskeskkond. |

- **Mõned vajavad kaamerat ainult enda kasutuseks.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` ja `update_camera_firmware()` avavad kaamera. Kui Chloros hoiab seda (ühendatud rakenduses, koos `connect_camera()`-iga, massiivi liikmena või kui seda kasutab hetkel mõni teine käsk, nagu massiivi test või pildistamine, mis avab oma kaamerad), keeldub tagapõhi HTTP-veaga 409 `CAMERA_HELD` (nii `code` kui ka `error_code` puhul), milles on märgitud, kuidas seda vabastada. Värvisätted, esiletõstetud alad, `camera_power_probe()` ja lugemised toimivad ühendatud kaameral.
- **Värv, esiletõstetud alad ja valgebalanss on mõeldud RGB-kaameratele.** Ühe sagedusribaga monokroomkaameral (M3M) ei muuda seadistused midagi ja tagastavad `{"skipped": True, "reason": ...}`, nagu teeb ka CLI. Need on kuvaseadistused: reaalaja vaade ja kuvatasandi pildistamised, mitte kunagi kiirgusvõimsus ega peegeldusvõime.

---

## Ajutised massiivid, joondamisprofiilid ja indeksid

Mida teeb `chloros-cli lattice` püsiva ühenduse puudumisel, nagu funktsioonid `chloros_sdk`-is (moodul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` ja `index`. Igaüks neist saadab CLI-käsu poolt saadetud päringu, kasutades CLI vaikimisi seadeid.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Need avavad kaamerad ise.** Iga käsk avab oma kaamerad, teeb oma töö ära ja sulgeb need, seega vajab see kaameraid, mida Chloros ei hoia (mis on rakenduses ühendatud, `connect_camera()`-iga või massiivi elemendina); tagapõhi keeldub HTTP-veaga 409 `CAMERA_HELD` ja annab juhiseid, kuidas ühte vabastada. Ajutine avamine võtab aega (mõne kaamera puhul umbes 40 sekundit). Korduvate pildistamiste puhul on `connect_array()` üks kord kiireim viis ning see salvestab iga eksporditüübi, kus ajutine pildistamine salvestab debayereeritud pildi. `calibrate_alignment_profile()` on erand: nende kaamerate ühendatud massiivis lahendab see massiivi enda kaadrite põhjal (`used_live_array`).
- **`serials`** on avatavad kaamerad, millest esimene käivitab päästiku; `None` avab kõik võrgus olevad kaamerad.
- **`settings`** on kaameraseadete ümberkirjutuste sõnastik, mille väljad määravad CLI lipud: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` ja `preset` (kaamera eelseadistus nime järgi). Antud säri või võimendus lülitab automaatjuhtimise välja, nagu seda teevad ka lipud.
- **`gpu`**: `True` sunnib kasutama GPU warp&#x27;i, `False` aga CPU (`--gpu` / `--no-gpu`); `None` laseb tagapõhjal valida.
- **`keep_settings`**/**`clear_settings`** (iga funktsioon, mis avab kaamerad, vaikimisi `False`): lõpus taastatakse iga kaamera selliseks, nagu see leiti; `keep_settings=True` salvestab käivituse seaded hoopis kaamera enda mällu (vabalt töötav režiim), `clear_settings=True` taastab selle mälu tehase seadistustele — vt [Kaamera mälu](#camera-memory--keep_settings--clear_settings). Vastus (töö puhul, `stop()`) sisaldab `camera_memory`.
- **Tee on tagapõhja oma**: suhtelised teed muudetakse absoluutseteks teie töökataloogi suhtes, kui tagapõhi asub sellel arvutil. Joondamisprofiili faili funktsioonid (`alignment_profile_info()`, `reorder_alignment_profile()`) loevad faili sellel arvutil, nagu seda teeb ka CLI.

| Funktsioon | Mida see teeb |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Salvestab ühe kaamera pilti pidevalt, kuni see peatatakse. Tagastab `CliJob`. `tiff` käivituse kaadrid moodustavad `raw` faili, mille massiiv salvestab selle kaamera andmetest (üks mosaiikpind andurienda väärtustega, tihendamata); `png` ja `jpg` mahutavad 8 bitti, seega on need jooksud vaatamiseks demosaikitud kaadrid. Seotud või koondatud DAQ kirjutab ühe `.daq` faili iga kaadri kohta jooksu`daq/` kausta (`daq_files` selle statistikas). Iga kaader kirjutatakse üks kord koos täielike metaandmetega (mudel, salvestusaeg, objektiiv, lainepikkused, kalibreerimine toorkaadril); `stop()` tagastab vastuse iga kord, kui kõik juba salvestatud kaadrid on kirjutatud (kuni 25 s), ja alles siis suletakse kaamera. Kaamera, mis on jäetud riistvaratriggeri jaoks valmisolekusse (`TriggerMode` `On`) seatakse käivituse ajaks vabajooksule (käivituse vastuse `trigger` näitab, mida leiti). Peatamisel taastatakse kaamera selliseks, nagu see leiti – selle käivituse seaded ja kõik jooksu käigus muudetud seaded –, välja arvatud juhul, kui kasutatakse `keep_settings=True`, mis salvestab jooksu seaded kaamera enda mällu (vabajooks), või `clear_settings=True`, mis selle mälu tühistab. Jooks, mis ei salvestanud ühtegi kaadrit, lõpetab end ise 10 sekundi pärast käskudega `stop_reason` ja `no_frames` ning põhjus on esitatud käskus `last_error`, ning selle käsk `stop()` tagastab selle põhjuse käskuna `error`. Kaadrid, mis jäid välja, kuna kirjutajad ei suutnud sammu pidada, loetakse kokku, kusjuures `drop_reason` selgitab põhjust. Chlorosi poolt hoitava kaamera puhul kasutage `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Sünkroniseeritud kaadrirühmad voos `fps`, `count` rühmade kaupa või kuni peatamiseni. Tagastab `CliJob` (selle `start_reply`-il on `n_cams` ja andmevoog `pacing`); selle lõplikud statistilised andmed hõlmavad haaratud, kirjutatud ja kõrvale jäetud gruppe, faile, mida kirjutaja ei suutnud kirjutada, ning gruppe, mille puhul kaamera midagi ei edastanud. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Üks sünkroniseeritud salvestus, üks debayereeritud pilt kaamera kohta: `results` (`filepath`, `serial`, `frame_id`) ja `pacing`. Ilmub teade puuduva kaamera kohta, milles märgitakse, kui palju faile esmalt salvestati. `lattice multi-capture`, millele pole ühtegi kaamerat ühendatud. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` sünkroniseeritud pildistamised kindla säriajaga: `sync_groups`, `master`, üks `group_timestamps` rida iga päästiku kohta (kaamera, mis ühe pildi vahele jättis, puudub oma reast) ja `sync`: iga teise kaamera nihke võrreldes peakaameraga, päästiku kaupa (`offsets_us`: kaamera kohta `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), ühe takti jooksul (`clock_domain`: `ptp` või `device` — PTP välja lülitatud, iga kaamera kasutab oma tööaja kella, seega on selle nihke hetk, mil kaamera käivitati, ja sünkroniseerimist mõõdab ainult jitter). Trigger, mille ajamärgised segavad seadme kella ja PTP-d (PTP välja lülitatud või veel lukustumas), jäetakse välja ja loetakse (`groups_mixed_domain`), samuti see, kus PTP-kell on ikka veel rohkem kui `agree_within_us` kaugusel peamassist (`groups_clocks_disagree`); seda näitavad `notes`. `clock_offset_mean_us` / `clock_offset_jitter_us` on suurimad kaamera kohta arvutatud keskmised väärtused ja kõikumised. `success` on vale, koos `error`-ga, kui triggert ei tabatud, kui vähem kui kahel triggertil on kõik kaamerad ühel taktil või kui taktid ei ole kunagi ühtlased (`sync.problems`); read tagastatakse igal juhul. See käivitab PTP-grandmasteri, kui see ei tööta (`time_sync` ütleb `grandmaster_running` ja `grandmaster_started_by_this_test`), lülitab sisse iga kaamera PTP (nagu teeb `connect_array()`) ning ootab enne esimest triggert, kuni iga kaamera PTP-kell on lukustunud grandmasteriga ja stabiliseerunud: `PtpStatus` on ühe grandmasteri orjaks, jäädes sellest 100 us piiresse ja püsides stabiilsena 3 s, ning iga kaamera on selliselt sünkroniseeritud, katkematult 120 s jooksul — umbes 2 minutit pärast sisselülitamist. äsja sünkroniseeritud kell on pärast „Slave” näidu ilmumist umbes 2 minuti jooksul kuni ~200 us võrra kõrvalekalduv, ning kaamera enda `PtpOffsetFromMaster` seda ei näita; 120 s viib kaamerad stabiilse kella täpsuseni ~10 us. Kaamera, mis on endiselt lukustatud varasema testi ajast, mille järel PTP jäi sisse lülitatuks (nagu seda nägi antud backend), loeb aega alates sellest lukustumisest, seega üksteisele järgnev test ei pea uuesti ootama; kaamera, mille test sisse lülitab, või kaamera, mis leiti PTP-ga sisse lülitatuna, kuid mille sünkroniseerumist see backend pole näinud (lülitati sisse mujal või enne backendi taaskäivitamist), ootab oma 120 s ära, ning mõlema kaamera massiivis ootavad just sünkroniseerunud kaamerad endiselt oma aega. Kaamera, mis lahkub orja režiimist, ületab 100 us piiri või järgib kolme järjestikuse lugemise jooksul teist grandmasterit (1,5 s) ooteaja jooksul, alustab loendamist uuesti; üks või kaks sellist lugemist logitakse (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) ja loendamine jätkub, ning grandmasteri ID-d, mida ei saa lugeda, käsitletakse tundmatuna, mitte muutusena. Kui kõikide kaamerate loendamine algab uuesti korraga (sekundi jooksul), on tegemist grandmasteri aja liikumisega, mitte kaamerate omaga: backend’ilogis on kirjas `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` loetleb seda `grandmaster_steps`-is ning seejärel lõppev ooteaeg algab veaga `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Selle jaoks piiri ei pikendata. `ptp_settle_s` (0–600; `0`, kui ooteaega ei ole) määrab käivituse stabiliseerumisaja; muud väärtused lükatakse tagasi (400 HTTP-viga). Ooteaeg on piiratud stabiliseerumisajaga pluss 90 s (vaikimisi 210 s, vähemalt 90 s) ning kutsumine ootab tagapõhja `timeout` s: vaikimisi 240 pluss stabiliseerumisaeg (360 s), nagu seda teeb ka `chloros-mcp`-i `test_array_sync`. `ptp_lock`-il on `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (kui kaua iga kaamera on olnud lukustatud ja stabiilselt koos), `found_settled` (kaamerad, mis olid juba lukustatud stabiliseerumisaja jooksul, kui need leiti) ning iga kaamera kohta `status`, `offsets_ns`, `slave_s` (kui kaua see on olnud lukustatud; Puudub: pole lukustatud) ja `remaining_s` (Puudub, kui ühegi kaamera PTP ei lülitu sisse). Tagapõhja logis on iga 5 sekundi järel rida, mis sisaldab iga kaamera seisundit, kui kaua see on olnud lukustatud ja kui palju aega on veel jäänud. Iga kaamera lülitub sisse PTP-funktsiooniga välja lülitatuna; sealt alates sünkroniseeruvad kellad umbes 10 sekundi jooksul. Seejärel seatakse iga kaamera tagasi selliseks, nagu test seda leidis – selle PTP, automaatne säri ja võimendus, mille test hoiab käivituse ajaks välja lülitatuna (`ExposureAuto` / `GainAuto` ning käsitsi seadistatud `ExposureTime` / `Gain`), ja ülejäänud, välja arvatud juhul, kui `keep_settings=True` – seega kaamera, mille PTP on sisse jäetud (`PtpEnable=True` või käivitus, kus `keep_settings=True`) jääb lukustatuks ning kui see tagapõhi on näinud, et see on olnud lukustatud stabiliseerumisaja jooksul, ei oota hilisem test seda. Kui kellad ei stabiliseeru, on `success` enne mis tahes triggereid vale ning `error` ütleb, mida teha: kaamera, mis pole kunagi lukustunud (`not_slave`), tähendab, et grandmaster ei jõua selle juurde (`time_sync_status()`); kaamera, mis pidevalt ühendusest välja langes, nimetatakse selle järgi, kui kaua see oli lukustunud (`remaining_s`); kellad on endiselt ühtlustumas (`far`): kontrollige `time_sync_status()` ning et need oleksid enne testi lukustatud, lülitage PTP sisse ja jätke see sisse (`PtpEnable=True` igal kaameral või `keep_settings=True`; test taastab leitud seaded, seega jääb see sisse lülitatuks), seejärel käivita test. `lattice multi-test`. |
| `detect_array_master(line=None)` | Sünkroniseerimisliinil olevad kaamerad ja vaikimisi määratud peakaamera: `master_serial`, `slave_serials`, mida `line` kontrollis. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Milline hulk neid kaameramudeleid (tootenimed, `"LATT-M3C-L87-FOCN"`) võiks selle arvuti võrgus olla: `tier`, `achievable_fps_max`, kaabli eelarve (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` ja `fix_suggestions`. Vaikimisi avastatud kaameramudelid; ei ole ühtegi kaamerat avatud. Teie olemasolevate kaamerate kohta, mille seadistusi ühendus kasutaks, vaadake [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Lahenda joondamisprofiil reaalajas kaadritest ja salvesta see. `method`: `orb`, `akaze`, `phase`, `checkerboard` (koos `checkerboard="9x6"`-ga, sisemised nurgad) või `manual` (koos `control_points={serial: [[x, y], ...]}`-ga, ka võrdluskaamera omad). `model`: `translation`, `rigid`, `affine` või `homography`. Täpsemad valikud on CLI lipud: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` on `{serial: {option: value}}`. `rms_threshold_px` keeldub halvemast lahendusest salvestamast. Tagastab `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (iga kaamera kohta `reprojection_error`, `num_matches`) ja `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Üks profiili kaudu joondatud mitme sagedusribaga pilt: `files` (mitme sagedusribaga TIFF-fail koos selle `bands` ja suurusega, pluss üks iga sagedusriba kohta koos `per_band` ja üks iga kaamera kohta koos `per_camera`) ning `vignette_corrected`. `bands` valib ja `order` järjestab lainealad nime järgi (nimekiri või komadega eraldatud string). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Jada järgi joondatud mitme riba pildid (samad joondamisvalikud kui `capture_aligned_multiband()`), kuni `count` või kuni peatamiseni. Tagastab `CliJob`; selle lõplikud statistilised andmed `frames_written` ja kõik `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Profiilifail, mille kokkuvõte on `align-info`, väljastab järgmised andmed: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (iga kaamera kohta `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (iga kaamera kohta `band_names`, `wavelengths_nm`, `enabled`); `profile` on kogu fail. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Muuda profiilifaili kohapeal: `order` on ribade nimed (igaüks nihutab oma kaamera kihi ülespoole, selles järjekorras), `enable` / `disable` on seerianumbrid. Tagastab uue `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Indeksi eelseadistused (`presets`: iga `name`, `channels`, `formula`) ja värv `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Indeks joondatud mitme ribaga TIFF-faili kohta: `preset` (`NDVI`, `NDRE`, ...) või `formula` sümbolite üle, mis `channel_map` kaardistab ribadele (`{"NIR": "NIR_850", "Red": 0}`, nime või indeksi järgi). Eelseadistusesümbol `channel_map` jätab välja faili selle ühe sagedusriba, mille nimi sellega algab (`red` -&gt; `Red_660`, `nir` → `NIR_850_RGN`), mille nime määrab fail ise (selle ribade loend või Chloros-pildi / toote `Chloros:Band{R,G,B}Name`), seega ei vaja NDVI-väärtus Chlorosi peegeldusandmetel `channel_map`-i; kaks ühele sümbolile vastavat laineala lükatakse tagasi, neid ei oletata. `capture_level` näitab, mida lainealad sisaldavad: `reflectance` või `radiance` muudavad väärtused kvantitatiivseks; kalibreerimata andmekogum (`align-apply`) annab kvalitatiivseid väärtusi; `"unknown"` loeb need failist, sealhulgas Chloros-faili `Chloros:ProcessingLevel`. Tagastab `stats` (kuju, 1.–99. protsentiili vahemik, LUT-vahemik, kasutatud `capture_level`) ning `files`, mis on kirjutatud sisendi kõrvale (`<input stem>_<preset>.tif`, `..._LUT.png`) või asukohas `output`, tee: pildilaiendiga failid nimetatakse selle tüve järgi (`output="D:/out/ndvi_FRGN.png"` kirjutab `ndvi_FRGN.tif` ja `ndvi_FRGN_LUT.png`), ilma laiendita on see eesliide (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); loe teed alates `files`. Need sisaldavad sisendi kaameraandmeid (tootja / mudel / seerianumber, GPS, objektiiv, pildistamise aeg UTC-s, kalibreerimistunnus, kuid mitte kunagi selle raw-DN-kalibreerimist), `Chloros:ProcessingLevel=index` ning indeks on nende ainsaks ribaks (`XMP-Camera:BandName`, nii TIFF- kui ka LUT-PNG-failid). Ainult failid; kaamerat pole. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Pildistamine, kalibreerimine (`reflectance` või `radiance`; `raw` jätab selle vahele), joondamine ja indeksi arvutamine ühes etapis; `save_multiband=True` säilitab kalibreeritud pildistiku, mille `compute_index()` saab hiljem võtta. Tagastab `stats` (koos `serials`-iga, mille ribasid ta kasutas) ja `files`. See avab `serials` (esimene käivitab päästiku), muidu profiili kaamerad, mille `layer_order`-kanne on lubatud, esmalt võrdluskaamera (avatakse isegi siis, kui selle sagedusalad on keelatud); kaamerat, mille sagedusalad on keelatud, ei kalibreerita kunagi. `channel_map` on vajalik ainult sümboli puhul, millele pinu ei suuda vastata, nagu näites `compute_index()`: eelseadistuse sümbol võtab selle ühe aktiveeritud sagedusriba, mille nimi algab sellega, mida kontrollitakse enne mis tahes kaamera avamist (kui ükski või mitu vastab, lükatakse need tagasi, nimetades neid); iga sümbol vajab kohandatud `formula`-i. `index.tif` ja LUT sisaldavad pinu kaamerat, pildistamise aega (võrdluskaamera säriaega, UTC) ja objektiivi, ning mõlemad nimetavad indeksit oma ainsaks ribaks (`XMP-Camera:BandName`, nt `NDVI`: värviline PNG-fail nimetab indeksit, mitte selle värvikanaleid). `lattice index --live`. |

### `CliJob`

Kolme voo poolt tagastatud käepide. Töö käib tagapõhiselt, kusjuures süsteem haldab oma kaameraid kuni töö peatamiseni.

| Liige | Kirjeldus |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (salvestatud kaadrid / kirjutatud, kõrvale jäetud, `fps`, `elapsed`, kirjutamisvead, `stop_reason`; pidev töö ka `frames_grabbed` ja `frames_queued` – iga salvestatud kaader kirjutatakse, pannakse järjekorda, jäetakse kõrvale või ebaõnnestub – ning `drop_reason`, `writers` ja `write_ms`, ühe kaadri kirjutamiseks kulunud keskmine aeg) ning `output_dir`. `state` on `running`, `ended` (see peatas end ise ja pole veel peatatud), `stopping` (peatus kirjutab oma järjekorda välja: reaalajas statistikat) ning `stopped_by`) või `finished` (selle lõplikud statistilised andmed, peatamise `error` ja `stopped_by`). Peatatud töö andmeid säilitatakse 10 minutit (kõige rohkem viimased 32), iga kliendi jaoks: teine skript või `chloros-mcp` loeb sama seisundit ja lõplikke statistilisi andmeid. Töö `chloros-cli` alustati (`lattice continuous`, `multi-stream`, `align-stream`) peatatakse tagapõhja poolt, kui CLI ei ole 30 sekundit sisse loginud — töö lõpetati, masin on puhkeolekus — koos `stop_reason`, `client_gone` ja `stopped_by` tagapõhjas; selle statistika siit lugemine ei hoia seda töös. SDK-st käivitatud töö jookseb kuni selle peatamiseni. Iga käivitamine saadab oma `timeout` tagapõhja tähtajana (`client_deadline_s`): pärast seda avatud kaamera (mis on veel käivitumas) pannakse tagasi ja suletakse ning ühtegi tööd ei käivitatud, ning kõne tekitab `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Kaamera, mida töö hoiab, lükatakse tagasi igale teisele avamisele (`connect_camera()`, uue käivitamise või ajutise salvestuse korral) HTTP-veaga 409 (`CAMERA_HELD`), nimetades ülesande ja selle algataja. |
| `stop(timeout=30.0)` | Peata see ja sulge selle kaamerad: `final_stats`, `output_dir` ja `error`, kui peatamine ei läinud sujuvalt. Teine päring annab sama vastuse, samuti ka peatamine teiselt kliendilt: kui esimene klient kirjutab veel oma järjekorda, ootab teine seda (kuni 25 sekundit; pärast seda tekib viga ja lõplikud statistilised andmed järgnevad `stats()`-is). |
| `wait(timeout=None, poll_s=0.5)` | Oota, kuni see ise lõpeb (kuni jõutakse `count`-ini, kõvaketas täitub või tekib viga), seejärel `stop()`, et koguda lõplikud loendid. `timeout` tekitab `TimeoutError`-i, kui see veel töötab; töö, millel puudub `count`, lõpeb alles siis, kui see peatatakse. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | selle ID ja tüüp (`continuous`, `multi-stream`, `align-stream`), kuhu see kirjutab, algse marsruudi vastus ning kas `stop()` on käivitatud. |

Kasuta seda kontekstihaldurina, et peatada see ulatuse lõppemisel.

---

## Projektikaust ja retseptid

**Projektikausta** on kaust, kuhu luuakse uued projektid, mida jagavad Chloros-rakendus, CLI ja SDK (moodul `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Kui konfigureeritud kaust enam ei eksisteeri, kasutatakse vaikimisi kaustu (`Documents/MAPIR/Chloros Projects`), nagu teeb seda ka rakendus. `set_project_folder()`-ile antud suhteline tee võetakse teie töökataloogist, kui tagapõhi asub sellel masinal.

**Retsept** on YAML- või JSON-tegevuste loend, mida `chloros-cli project run PATH RECIPE` käivitab salvestatud projekti suhtes. `run_project_recipe()` käivitab selle CLI(mis on paigaldatud koos SDK-ga), seega käitub üks retsept mõlemal juhul ühtmoodi:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** on `.yaml` / `.yml` / `.json` fail või retsept ise: sõnastik, mis sisaldab `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) ning valikuliselt `overrides` ja `stop_on_error` (vaikimisi true: käivitus peatub esimese ebaõnnestunud toimingu juures). Toimingud on CLI-viite [Recipe DSL](cli-reference.md#recipe-dsl); suhteline `output` lahendatakse teie skripti töökataloogi suhtes.
- **Tagastab** `ok` (kõik toimingud õnnestusid), `actions` (kui palju), `log` (käivitaja väljund; `echo=True` kuvab seda ka käivitamise ajal) ja `validation`. Puuduv või analüüsimatu retseptifail tekitab veakoodi `ChlorosConnectError`.
- **Kontrollitakse enne käivitamist.**`validation` on `{"ok", "errors", "warnings", "steps"}`: iga tegevuse võtmesõnaks peab olema täpselt üks teadaolev verb (`{"capture": {"name": ...}}`, mitte `{"action": "capture"}` või `{"type": "capture"}`), mis nimetab projektis olevat õiget tüüpi kaamerat, massiivi või andurit. Veaga retsept ei käivita**ühtegi** tegevust (`ok` false, vead on `log`), olenemata sellest, mida `stop_on_error` ütleb. Kasutamata võtmed, ning kõik retsepti poolt juhitavad massiivid või valgusandurid, mis pole hetkel ühendatud, on `warnings`. Retsept ei ühenda seadmeid: ühendage projekti esimesed (`ChlorosProject.connect_all()`, `chloros-cli project connect`, või MCP-tööriistaga `connect_project_devices`). `dry_run=True` ei käivita midagi ja tagastab `{"recipe": ..., "validation": ...}`.
- Protsessis töötab korraga üks retsept. YAML-retsept vajab PyYAML-i.

---

## Projekti automatiseerimine — `ChlorosProject`

Salvestatud Chloros-projekt on kaust, mis sisaldab `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` laadib manifesti (`relink`: kaust, kus asuvad nüüd ümberpaigutatud projekti puuduvad allikafailid, mida kasutatakse, kui tagapõhi seda laadib — `process()` — vt **Ümberpaigutatud projekt** jaotises `ChlorosLocal`) ning `connect_all` viib kõik salvestatud kaamerad, maatriksid ja valgussensorid koos nende salvestatud seadistustega võrku – sama riistvaraseisund, mille GUI tekitaks. Valgussensorid ühenduvad tagapõhja [DAQ-andurite reservi](#daq-sensor-sessions) kaudu, seega vajavad nad töötavat backendit (vt [`SensorHandle`](#sensorhandle)).

### Minimaalne näide

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Või kontekstihaldurina:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject` meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Leiab ja ühendab kõik salvestatud kaamerad, massiivid ja valgussensorid. Tagastab klassipõhise ühendamisaruande (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, millest igaühe puhul on `connected`, `missing` ja `errors`; `arrays` sisaldab ka `alignment`). Kui tagapõhi kuulab kanalil `127.0.0.1:5000`, leitakse salvestatud kaamerad ja massiivid ning ühendatakse nende kaudu. Kui seda pole, leiab ja juhib protsess neid ise kanali `lattice_sdk` kaudu, mis vajab Arena SDK-i jooksuaega ja OpenCV-d (`cv2`) — SDK-i paigalduspakett ei installeeri kumbagi; kui see ei õnnestu, loetletakse kõik salvestatud kaamerad ja massiivid `errors` all koos põhjendusega ning `missing` jääb tühjaks. Töötav tagapõhi, mis aga keeldub SDK-st (välja logitud: HTTP 401; Copperist madalam tase: 403), ei tähenda „tagapõhi puudub”: iga salvestatud kaamera, massiiv ja valgussensor on loetletud `errors` all koos tagapõhja põhjendusega, ja selles protsessis ei avata midagi. Iga kutsumine kontrollib tagapõhja uuesti, seega ühendub `connect_all()` pärast sisselogimist või Chlorosi käivitamist sama projekti objektiga. Backendis juba käivitatud salvestatud massiiv, millel on samad kaamerad (kas kasutajaliidese omad või `chloros-cli project connect` omad), lisatakse sellisena, nagu see on, ilma salvestatud seadeid uuesti rakendamata, ja `disconnect_all()` jätab selle töötama. Valgusandurid läbivad alati tagapõhja (vt [`SensorHandle`](#sensorhandle)). See ei käivita kunagi tagapõhja. |
| `disconnect_all()` | Kõik maha võtta. Kõigepealt vabastatakse kõik seadmed; seejärel, kui backend ei kinnitanud kaamera, massiivi või valgussensori vabastamist (mahavõtmine aegus või lükati tagasi), nimetab igaühe eraldi `ChlorosConnectError`-iga. Juba kadunud seade, või kui tagapõhi ei tööta, jääb vaikseks. Massiiv või valgussensor, mille mõni teine klient oli avanud, jääb töötama. Projekti `with` lõpetamine ei käivitu kunagi. |
| `capture_all(output_dir=".")` | Üks kaader igast ühendatud kaamerast + massiivist, ja üks spekter (`SensorHandle.read()`) igalt ühendatud valgussensorilt, võetud järjekorras (kaamerad, seejärel massiivid, seejärel sensorid), mitte ühel hetkel. |
| `stream(camera, overlays=False, fps=10.0)` | Generaator, mis toodab BGR-formaadis `numpy` kaadreid nimetatud kaamerast (või maatriksist). `overlays=False` loeb kaadreid käepideme`grab()` tasandil: tagapõhja kaudu, kui projekt on sellega ühendatud, muidu otsese `lattice_sdk` kaadrite salvestamise tsükli kaudu (massiivid annavad `{serial: frame}` sõnastikke). `overlays=True` suunab läbi `ChlorosLocal.camera_stream()` → tagapõhja `/api/camera/<serial>/stream-annotated` MJPEG-voo, kusjuures kaamera salvestatud `ui.overlay` plokk edastatakse päringuparameetritena. Nõuab tagapõhja režiimi, OpenCV-d (`cv2`) ja **iseseisvat kaamerat**: otserežiimis kaamera tekitab vea `RuntimeError` (tagapõhi ei saa haarata kaamerat, mis kuulub sellele protsessile) ja massiiv tekitab vea `NotImplementedError` (koondab komposiidid kaamera kaupa — edastab liikme nime järgi). Ühekordse käivituse ekvivalent: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Käivita joondamine igal hetkel ühendatud massiivil. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Käivita kalibreerimis-/indekseerimispipeline projekti piltidel (ümbritseb `ChlorosLocal.process`; need neli on **ainukesed** aktsepteeritavad kwargid — `indices=` jne. tekitab veateate `TypeError`; seab indeksid `ChlorosLocal.configure()` kaudu). Loob viivitusega `ChlorosLocal()`, mis käivitab automaatselt tagapõhja. |

Atribuudid:
- `proj.cameras` — `Dict[str, CameraHandle]`, mille võtmeks on nimi JA seerianumber.
- `proj.arrays` — `Dict[str, ArrayHandle]`, mille võtmeteks on nimi JA array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, mille võtmeteks on nimi JA slot_id.
- `proj.config` — `project.json["config"]` sõnastik.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Töötlemistasemed.** `capture()`, `grab()` ja `frame_stream()` võtavad kõik vastu sama `processing`
tokenit ning ahel on kumulatiivne — iga tase käivitab kõik sellest kõrgemal asuvad tasemed:

| Tase | Väljund | Märkused |
| --- | --- | --- |
| `raw` | 1-kanaliline Bayer, anduri omapärane | Demosaiki ei toimu. Selles tasemes pole ülekatteid saadaval. |
| `debayered` | 3-kanaliline värv (**vaikimisi** `CameraHandle` ja kõikide `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` ja `stream()` on vaikimisi seadistatud `raw`-ile): BGR mälus (`grab()` / `frame_stream()`), salvestatud RGB-na `capture()` abil | Tagapõhja režiimis, pildistamisel: kalibreerimispaketi tumeda nihke (DSNU) kohaldamine toorandmetele, seejärel OpenCV servatundlik demosaik – ilma lineaarsuse LUT-ita, tasandatud väljata või segamise eemaldamiseta. Ainus tase, mis töötab ilma tagapõhimooduseta, on kaameraraamatukogu tavaline bilineaarne demosaik (ilma tumeda nihketa). Salvestatud TIFF-fail on fotomeetriline RGB, seega pöörake mälus oleva kaadri kanalid ümber (`frame[..., ::-1]`), et seda failiga võrrelda. |
| `radiance` | float32, W/m²/sr/nm, **üks tasand iga laineala kohta** | Täielik radiomeetriline ahel: demosaik + 3×3 segamise eemaldamine (multispektraalne) + DSNU + tasandus + NIST-skaala, kus ekspositsioon × võimendus on välja jagatud, nii et väärtused on absoluutsed. |
| `reflectance` | uint16, 32768 = 1,0 (või uint8, 255 = 1,0 — vt `bit_depth`), **üks tasand iga sagedusriba kohta** | Kiirgusjõud jagatud allapoole suunatud kiirgusintensiivsusega (ρ = π·L/E). Vajab DLS/DAQ-näitu — vt allpool olevat märkust. |
| `display` | 8-bitine sRGB-laadne | GUI-ekvivalentne renderdamine: CCM + valgebalanss + gamma kaamera aktiivse värviprofiili kaudu. |

Ainult otserežiimis (ilma tagapõhja) on saadaval ainult `debayered`: `grab()` teisel tasandil tõstab
`NotImplementedError`, `capture()` ignoreerib `processing` ja `frame_stream()` tekitab
`RuntimeError`-i igal tasandil. `reflectance` vajab kasutatavat allapoole suunatud mõõtmistulemust — raami lõpp-punkt tõmbab
koondatud DAQ-i automaatselt kaamera DLS-pesasse, kuid kui DAQ-i pole seotud, keeldub ahel
peegeldusvõime väljundist ja märgib alandamise ausalt tagastatud metaandmetesse, selle asemel et vaikselt
tagastades madalama kvaliteediga toodet.

> **Peegeldusvõime DN-skaala – ära koodi seda kõvaks.** LATTICE’i peegeldusvõime kasutab `32768` = ρ 1,0 ja märgib
> XMP `Chloros:PixelScale=32768`; Survey3’i peegeldusvõime kasutab `65535` = ρ 1,0 ning ei sisalda
> `Chloros:*`-märgistusi. Loe märgistus välja ja jaga sellega. See on määratletud uint16-domeenis, seega jääb see
> `32768`-iks igas formaadis, mis skaalab ümber (16-bitine TIFF, 8-bitine TIFF, 8-bitine PNG/JPG, 32-bitine protsent)
> — normaliseeri salvestatud andmetüüp esmalt tagasi uint16-ks (8-bitisest ×257, ujukomast ×65535). Ainus
> erand on fail, mis pärineb versioonist enne 1.3.0: 8-bitist TIFF-faili *kärbiti* tol ajal, mitte skaalati ümber, seega
> seda ei kirjeldata ühegi skaalaga ning see ei sisalda `PixelScale` ega MicaSense&#x27;i tuplit. Käsitle puuduvat märgistust
> LATTICE peegeldusfailis kui „kehtivat skaalat pole”, mitte kui vaikimisi väärtust.

**Kalibreerimismärgised: ainult `raw`-is.** Salvestuse `raw`-fail sisaldab kaamera kalibreerimispaketti: `CalibrationJSON`-andmekogum ja MicaSense-ga ühilduvad märgised (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), mis kirjeldavad anduri toorväärtusi. Iga teise taseme fail (`debayered`, `display`, `radiance`, `reflectance`, `index/` render) ning iga pideva salvestuse / mitme kaamera voo kaader sisaldab kaamera identiteeti (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, …) ning kaadri enda salvestusandmed, `XMP-mapir:FrameJSON` (säriaeg, võimendus, temperatuur, pikseliformaat, salvestusaeg, DAQ-vastavus), kuna iga `process()` toode kannab seda identiteeti. Töötlemine leiab sellise faili kalibreerimise selle seerianumbri ja `CalBundleSha256` järgi Chlorosi kalibreerimismälust, laadides paketi alla üks kord, kui sellel arvutil seda pole.

#### Mitmest kaamerast loodud failid

Fail, mis on loodud rohkem kui ühe kaamera lainealadest – mitme laineala pildistack (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), mitmest kaamerast koostatud reaalajas indeks (`index.tif`, `index_LUT.png`), masspildistuse `composite/` pilt — nimetab iga kaamera, mis sinna sisse läks:

- **Iga-kaamera nimekirjad, sagedusribade järjekorras** (komposiit: taustakiht, seejärel esiplaan; indeksikiht, kaamerad, millest selle lained pärinevad): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibreerimispakett, mille kaamera iga sagedusriba läbis; muul juhul — `capture_aligned_multiband()` / `start_aligned_stream()` kogu, mille sagedusribad ei läbinud ühtegi — kaamera pakett, mille Chloros on vahemällu salvestanud, see, mille järgi on nimetatud selle pildifailid), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — paralleelsed loendid, millest igaühe *n*-nda kirje kirjeldab sama kaamerat; `-`, kus kaameral puudub väärtus (kalibreerimispakett puudub, objektiiv on teadmata). Iga kaamera on **üks** kirje: kui selle pakettil on ühe välja jaoks mitu väärtust — paketi halogeenlambi sertifikaadid — need on ühendatud `;` (`F-1479-1;F-1480-1;F-1481-1`) abil, mitte kunagi komaga, seega loetakse nimekirjast üks kirje kaamera kohta, olenemata sellest, kuidas see trükitakse.
- **EXIF `SerialNumber`** sisaldab samu seerianumbreid, mis on ühendatud komadega (`214000535,214000551`).
- See **ei** sisalda ühegi kaamera identiteeti (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): need nimetavad ühte kaamerat ja selle paketti ning töötlemisel määratakse nende põhjal kindlaks faili kalibreerimine. Mitme kaamera lainepikkustest arvutatud indeks (`index.tif`, `index_LUT.png`) on EXIF-andmetes `Model` `LATTICE` (seade), mitte ühe kaamera mudel `LATT-…`.**Ühe** kaamera lainealade fail – lainealapõhine TIFF, `capture_aligned_multiband(per_camera=True)` kaader, ühe kaamera pildistamiskogum või reaalajas indeks – sisaldab selle kaamera enda identiteedimärke, nagu iga teine salvestusfailgi. `capture_aligned_multiband()`-i pildiseeria, sagedusriba- ja kaamerapõhised failid ning iga `start_aligned_stream()`-i pildiseeria ja selle sagedusriba-failid sisaldavad oma rühma salvestusaega (viitekaamera säriaeg, UTC), ning monokaamera `per_camera` kaader on ühe tasandi pilt, `bit_depth` nagu ka pildiseeria.**Kaamera orientatsioon (peegeldamine / pööramine).** Kaadrid failidest `grab()` / `frame_stream()` on alati
kaamera algses orientatsioonis, igal tasandil. Salvestatud pildid järgivad kaamera orientatsiooni
seadet (kasutajaliidese peegeldus H / peegeldus V / pööre):

- **`raw` ja `debayered` failid on täpselt see, mida andur luges**, olenemata seadistusest — pööratud või peegelpildis olev Bayer-
  mosaiik oleks teistsugune värvifiltri muster ning kalibreerimiskaardid on
  sensori enda võrgustikul.
- **`display`, `radiance`, `reflectance` ja indeksifailid on orienteeritud** – pööratud ühe korra pärast kõiki
  kalibreerimisarvutused. Nende klambrimaskid asuvad samal võrgustikul.
- **Iga pildifail salvestab seadistuse** XMP-vormingus: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, päripäeva, pärast
  pööramiste järel) ning `ChlorosOrientationApplied` — `false` töötlemata ja debayereeritud piltidel, `true` orienteeritud
  piltidel. `ChlorosProcessing` loetleb selle faili pikslitele rakendatud operatsioone (`none`, kui
  on tegemist algupärase failiga). See hõlmab iga kaadrit pidevast salvestusest (kaamera enda seade,
  rakendamata) ja mitme kaameraga voost (identiteet: selle liikmekaadrid on algupärased, nagu
  massiivi salvestuse puhul).
- **Projekti töötlemisel rakendatakse salvestatud orientatsiooni** toodetele, mis luuakse toorest materjalist
  (esmalt kalibreerimine anduri koordinaatides, seejärel massiivi joondamise moonutus, viimasena orientatsioon),
  nii et need vastavad reaalajas salvestuse tulemustele — sealhulgas kalibreerimata anduri vastus / vinjetikorrigeeritud
  varutulemused; selle debayereeritud tulemus jääb algupäraseks.
- **Chlorosi pildivaataja näitab toor- või debayered-faili, mida on pööratud vastavalt salvestusandmetele** — ainult kuvamiseks,
  faili sisu ei muudeta; faili, millel salvestusandmed puuduvad, näidatakse sellisena, nagu see on salvestatud.
- **Salvestusi, millel salvestusandmed puuduvad, töödeldakse originaalvormis.** Enne versiooni 1.3.0 pööras ühe kaameraga pildistamine
  ise oma toor- ja debayered-faile; Chloros ei suuda sellist toorfaili eristada algupärasest ja
  ei tee selle kohta oletusi.
- **Massiivpildistamised hoiavad vaikimisi iga liikmefaili algupärasena** ja salvestavad identiteedi;
  `ArraySession.capture(orient=True)` teisendab nende tulemused (vaata massiivi salvestamise juhtnuppe).

#### Väljund ribade kaupa ja segamassiivid

Tasemetel `radiance` ja `reflectance` tagastab kaamera **ühe tasandi iga selle
kaamera riba kohta** — kolm M3C puhul (selle kolm segamata sagedusriba, kanalite järjekord BGR), üks M3M-i puhul (selle
ainus kitsasriba filter). Monokaamera dubleeritud tasandid on kokku koondatud, seega annab **segatud
M3C/M3M-massiiv annab iga seeria kohta erinevaid massiivi kujusid**. Lugege kaadri
metandmetest välja `bands`-võti, et teada saada, milline on iga tasand; ärge järeldage seda kanalite arvu põhjal:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Kontrollige alati `hdr["processing_level"] == "reflectance"`-i enne väärtuste kasutamist. Alandamine
(puudub DAQ-näid, puudub kalibreerimispakett, teadmata sagedusriba paigutus) kajastub `processing_level` ja
`radiometric_refused` väärtustes, mitte tõstetuna — tagasi saadud massiiv on endiselt kehtiv raam, lihtsalt
mitte see toode, mida te soovisite. Alandatud salvestatud kaader kannab oma XMP-s sama koodi
nagu `Chloros:RadiometricRefused` (nt `no-band-layout`, `no-dls-snapshot`).

#### Üks triggereid lugemise kohta — `group_id`

**Massiivi liikmed teenindatakse massiivi sünkroniseeritud kaadrirühmast, mitte iga kaamera enda
pildipuhvrist.** Riistvaraliselt sünkroniseeritud massiivi kaamerad eksponeerivad üheaegselt, kuid N lugemist kaamera kohta
ei *võta proove* üheaegselt — iga kaamera viimase puhvrisse salvestatud kaadri lugemine ulatub üle triggereid
piiridest üle, ning üksainus kadunud kaader desünkroniseerib kogu komplekti, ilma et keegi seda märkaks.

Rühmast edastamine on vajalik, kuid sellest üksi ei piisa: iga päring teeb ikkagi
võtab hetkel *viimase* grupi pildi ja salvestusring võib vahepeal uusi kaadreid avaldada.
Reaalajas 4-kaameraga massiivi mõõtmistulemuste põhjal **ülendasid 20 paralleelsest lugemisest 12 kaht gruppi** — alati
täpselt kõrvuti. Seega on lugemine **fikseeritud**:

1. Iga kaader kannab `group_id`-i, mis on kaamerate rühma monotoonne avaldamislugem.
2. `grab()` loeb master-kaamerat (või, kui master-kaamerat ei ole võimalik teenindada, järgmist kaamerat, mis suudab seda teha),
   seejärel kinnitab ülejäänud kaamerad rühma ID-le, millele ta maandus (`?group_id=N` HTTP
   lõpppunktis).
3. Tagapõhi säilitab viimaseid gruppe **vähemalt poole sekundi** jooksul, seega teenindatakse ka sellist kaamerat, mis on juba
   hilisemate avaldamiste poolt ületatud – piisavalt kaua, et kaamera-põhine
   peegeldusandmete ekspordi edasi-tagasi liikumiseks isegi 9-kaameralise täissensoriga massiivi puhul. Seos, mida tagapõhi enam
   ei hoia, lükatakse tagasi koodidega `409` ja `group_aged_out: true`, mitte vastatakse vaikimisi
   teise triggiga – `grab()` loeb komplekti uuesti sisse.

Uuesti mõõdetud kinnitamisega: **20-st 20-st lugemisest tagastati üks triggert.** `grab()` teeb seda teie eest;
HTTP-lõpppunkti otse juhtides vajate ainult `group_id=`-i.

- Kaamera, mida kinnitatud rühmas ei saa teenindada — see puudub rühmast või keeldub taotlusest
  (`band`, mida tal pole, tase, mida ta ei suuda toota) — **jäetakse välja, mitte ei täideta tagantjärele**
  teise käivitaja kaudu ning teised kaamerad kuvatakse endiselt.
- Sõnastik, mida `grab()` tagastab, on `GrabResult` (tavaline `dict` alamklass;
  `from chloros_sdk import GrabResult`): `.errors` seostab iga
  välja jäetud seerianumbri erandiga, mille lugemine selle põhjustas, ning `.group_id` on jagatud rühm. Tühi
  `.errors` tähendab, et kõik kaamerad moodustasid selle rühma.
- Iseseisval kaameral pole rühma, see teatab `group_id: None` ja **lükkab tagasi** `group_id=`
  pin-i, selle asemel et seda ignoreerida.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Sama loendur avaldatakse massiivi liikme **iga kaamera-põhise MJPEG-voo** puhul —
eelvaade (`/api/camera/stream/<serial>`), märgistatud eelvaade (`/stream-annotated`) ja
peegeldusvoog (`/stream-reflectance`) – kui `X-Chloros-Group-Id` igas osas, lisaks
`X-Chloros-Frame-Id`-ile ja `X-Chloros-Timestamp-Ns`-ile. Voogud on pigem pidevad kui
hetkelised, seega sobitatakse neid osade kogumise ja selle päise järgi rühmitamise teel — seadistatavat
pini pole. Iseseisval kaameral pole rühma. Vaata
[Kaamerapõhised MJPEG-vood](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` ei jää kunagi vaikseks.** Tikk, mis ebaõnnestub põhjusel, mis võib iseenesest lahenduda,
jätatakse vahele – kaamera voog ei anna selle kohta midagi, massiivi voog annab kaamerad, mis
vastasid (võimalik, et ühtegi). Need põhjused on: veel pole kaadrit; kaamera pildivõtmise niit on peatunud
sekundiks või kaheks (`404 not streaming`, millele tagapõhi vastab iga kord, kui teine kaamera ühendub
või lahti ühendub või kui seadete muutmine tasakaalustab uuesti kaamerapooli); tagapõhja taaskäivitamine või `5xx`; joondamine
on veel pooleli (`409 … retry shortly`); ning peegeldusvõime keeldumine, kuna DAQ-seadmel pole veel näitu
(`no-dls-snapshot`, aegunud või tühi näit). Keeldumine järgmisel taktil korduks lihtsalt —
tundmatu `band`, `bit_depth=8` kiirguse puhul, kalibreerimispakett ilma kiirgusskaalata, peegeldusvõime
keeldumine, kui **DAQ-i pole üldse** (ükski pole seotud kaamera või selle maatriksiga, ükski pole ühendatud
tagapõhjaga: keeldumise `dls_source` on `"none"`), mis tahes muu `4xx` —
tekitab `chloros_sdk.FrameFetchError` (`RuntimeError`, mis kannab `.status` ja `.body`)
korraga. Sama kehtib ka **60 sekundi** pikkuse vahelejäänud tikide puhul, mille vahel pole ühtegi kaadrit, olenemata sellest, mis on `fps` — piisavalt
pikk, et üle elada joondumise stabiliseerumist ja tagapõhja taaskäivitamist, kuid piisavalt lühike, et kaamera, mis on
on tõesti kadunud, lõpetaks voo. See loobumisviga on samuti `FrameFetchError`, nii kaamera kui ka
massiivi puhul, kus viimase lugemise enda erand on `.errors` (ja kaamera puhul `__cause__`).
Massiivi puhul tähendab „refused” seda, et kõik kaamerad keeldusid; erand `.errors` sisaldab iga kaamera
põhjust.

**Massiiv `frame_stream()` annab iga trigeri välja üks kord** (nii `ArrayHandle` kui ka `ArraySession`).
Tik, mille uusim komplekt on juba väljastatud käivitaja, ootab järgmist käivitajat, selle asemel et
väljastada sama komplekti uuesti, mistõttu voog töötab aeglasemal kiirusel, mis on `fps` ja massiivi enda kaadrisageduse
, ning järjestikused komplektid ei jaga kunagi ühtegi `group_id`-i. Massiiv, mis lõpetab avaldamise, tekitab
`FrameFetchError` („60 sekundi jooksul pole uusi triggereid”) selle asemel, et oma viimast komplekti korrata. Ilma selle funktsioonita jagaks
voog, millele nõuti rohkem, kui massiiv avaldas – või mis alustas kohe pärast ühenduse loomist –,
kordusi: ühe mõõdetud katsetuse jooksul sisaldasid 20 komplekti 11 erinevat rühma ID-d.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Ühe sagedusriba eksportimine — `band`

`grab()` / `frame_stream()` võtavad `band="<name>"`, et eksportida vaid üks kaamera laineala
ühekanalilise `(H, W, 1)` massiivina — sama kujuga, mida toodab monokaamera, seega käsitletakse ühe laineala väljundit
täpselt samamoodi, olenemata sellest, kas see pärineb M3M-ist või M3C jagamisest. FRGN-i punane laineala:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Aktsepteeritavad kirjapildid on täielik paigutuse nimi (`Red_660`, `NIR_850`, `Green_550`) või selle lühike
pealkiri (`red`, `nir`, `green`), suurtähti ei arvestata. Tundmatu nimi lükatakse tagasi koos loetletud
kättesaadavate nimedega.

> **Valik tehakse nime, mitte kanalinumbri järgi – see on teadlik valik.** „Kanal 1“ on `Green_550`
> massiivi BGR-kanalite järjekorras, kuid *punane* filtri kooditähtede järjekorras (`RGN`), ning need kaks
> ei ühti. Vale valik annab täiesti usutava kaadri valest sagedusalast, mida
> allavoolus asuvad seadmed ei suuda tuvastada. Sel põhjusel puudub numbriline indeks.

#### Voogude joondamine — `align`

`align=True` teisendab kaadri massiivi ühisesse võrdluskaadrisse. Iga kaamera teisendatakse
**sama** profiili resolutsiooni ja kärbitud huvipiirkonna kaudu, nii et eraldi avatud kaamerapõhised vood
tulevad välja **vastastikku registreeritud** — neid ei pea koos taotlema:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Nõuded ja toimimine:

- Kaamera peab asuma **riistvarasünkroniseeritud** massiivis (M8 sünkroniseerimiskaabel), millel on joondamisprofiil
  — ühiskirjutus eeldab, et liikmed on eksponeeritud samaaegselt, mida tarkvara- või
  vabalt töötav massiiv seda ei taga. Muul juhul lükatakse taotlus tagasi veakoodiga `409`.
- **Keeldutakse, kunagi ei jäeta vaikimisi joondamata.** Pildistamistee võib salvestada joondamata koos hoiatusega,
  kuna fail säilitab oma päritolusildid ja seda saab hiljem uuesti registreerida; otseülekandel pole
  sellist teist võimalust, seega on profiili puudumine pigem viga kui tagavaravõimalus.
- **Joondamise taotlemine käivitab selle.** `align` taotlemine profiilita massiivi puhul käivitab
  kalibreerimise ja tagastab `409` koos sellega, mida oodatakse (nt *&quot;ootab, kuni ekspositsioon
  stabiliseerub väärtusel 214000551 (1/3)&quot;*) — proovi uuesti ja see õnnestub, tavaliselt mõne kümne sekundi jooksul.
  See on oluline peata režiimis: ühenduse loomise ajal toimuv automaatne joondamine katkestatakse 30 sekundi pärast ja iga katse
  selle ajavahemiku jooksul jäetakse vahele, kui smart-AE veel stabiliseerub, mistõttu massiiv, mille ekspositsioon
  läheneb aeglaselt, jäi varem kogu seansi jooksul ilma profiilita ja seda polnud võimalik saada
  ilma töölaua kombineeritud vaadet avamata. `alignment_status` teatab ka
  `ae_align_holdouts`, nimetades kaamerad, mis on veel stabiliseerumas, ja nende edusamme.
- Anna igale ühe massiivi voole **sama `max_h`**, muidu jäävad nende väljalõiked erinevatele pikselvõrkudele
 .
- Joondamine rakendatakse *enne* bittisügavuse kodeerimist, seega interpolatsioon arvutab täis-täpsusega
  väärtusi.

#### `bit_depth` — täpsuse vahetamine ribalaiuse vastu

`grab()` ja `frame_stream()` aktsepteerivad `bit_depth=16` (vaikimisi) või `bit_depth=8`:

| `bit_depth` | Peegeldusvõime kodeerimine | Kulu |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, varu kuni ρ 2,0 | Täis täpsus. |
| `8` | uint8, hõlmab `0..refl_max` (vaikimisi ρ 1,0) | Pool baitidest; ~0,004 ρ kvantiseerimine vaikimisi ülempiiril. Ülempiirist suuremad väärtused kärbitakse — vt allpool. |

##### 8-bitine ülempiir — `refl_max`

`refl_max` määrab ρ väärtuse, mida 8-bitine vahemik hõlmab, seega `pixel_scale = 255 / refl_max` ja ρ on endiselt
`pixel / pixel_scale`. `grab()` ja `frame_stream()` võtavad seda märksõnana nii kaamera kui ka
massiivi käepidemeid (`?refl_max=` HTTP-lõpppunktis). Suurendage seda, et saada rohkem ruumi peegeldavatele
valguskohadele, proportsionaalselt jämedamate sammudega:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | samm |
| --- | --- | --- |
| `1.0` (vaikimisi) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Automaatne säritus ei suuda 8-bitist klippimist parandada ja selle kasutamine halvendab andmeid veelgi.**
> ρ = π·L/E, kus valgustus ja võimendus on välja jagatud, seega on ρ stseeni omadus, mitte
> kaamera seade — mõõdetuna reaalajas maatriksil vähendas valgustuse poole võrra vähendamine keskmist ρ väärtust 1,3% võrra. Valgustuse suurendamine
> säriaega, et see „sobiks“ vahemikuga, viib pigem pikslid **anduri** küllastumiseni, mis
> alahindab L-i ja nihutab ρ-d **madalamaks** ilma veateate ilma (mõõdetud: maksimaalne ρ 1,203 → 0,985
> 2× säriajal, keskmine −23%). Valige vahemik `refl_max` abil; anduri hoidmine
> küllastumise väljaspool ongi juba automaatse ekspositsiooni (AE) ülesanne, mis kaitseb pigem ρ *kehtivust* kui selle vahemikku.

**Lugege `reflectance_clipped_fraction`, mitte lipukest.** `reflectance_clipped_at_1_0` käivitub ühe
piksli puhul — reaalses stseenis näitas see `True`, kuigi osakaal oli `0.000001`, üks piksel
miljonist. Just murdosa näitab, kas ülempiiri tuleb tõsta.

**`bit_depth=8` on ainult peegeldusvõime.** Selle taotlemine `radiance` jaoks lükatakse tagasi, mitte
ei rahuldata: peegeldusvõimel on kindel vahemik 0–1, mis säilib 8-bitises kodeeringus, samas kui kiirgusel pole
kindlat vahemikku, mistõttu selle mahutamiseks 8 biti sisse on vaja iga kaadri kohta protsentiilide venitamist, mille kaardistus
muutub iga kaadri järel — see on pilt, mitte mõõtmistulemus.

#### Reaalajas peegeldusvõime voog (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` edastab 8-bitist ribapõhist peegeldusvõimet kui
`multipart/x-mixed-replace` — M3C-vormingus 3-kanalilise JPEG-ina, M3M-vormingus halltoonidena. Päringuparameetrid:
`fps` (vaikimisi 10), `jpeg_q` (vaikimisi 95), `max_h` (0 = algupärane), `band` (üks nimetatud sagedusala, nagu
eespool), `align` (`1` kaasregistreerimiseks — samad reeglid kui SDK-s `align`), `refl_max` (8-bitine
kodeerimise ülempiir, nagu eespool). Iga osa kannab
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` ja `X-Chloros-Reflectance-Clipped`.
**Erinevatest kaameratest pärit osad, millel on sama `X-Chloros-Group-Id`, moodustavad sama käivitaja** —
niimoodi koondatakse ühe massiivi mitu kaamerapõhist voogu uuesti kokku ning see on tagatud
serveri poolel, kuna massiivi liikmeid edastatakse sünkroniseeritud kaadrirühmast (vt
[Üks triggert lugemise kohta](#one-trigger-per-read--group_id)). See tagastab `409`, kui peegeldusvõime on
ühenduse loomise ajal kättesaamatu, ja lõpetab voo, kui see muutub kättesaamatuks töötamise käigus, seega tähendab seiskunud
voog pigem „peegeldusvõimet enam pole“ kui „veel korras, lihtsalt staatiline“.

> **JPEG on kadudega – need pikslid on mõõtmistulemused.** Tihendamisviga kajastub otse
> peegeldusväärtustes ja on kõige tõsisem servades, kus asuvad materjalide piirid. Kvaliteedi
> vaikimisi väärtus on sel põhjusel 95. Kui vajate numbreid muutmata kujul, kasutage
> `grab(processing="reflectance")` / `frame_stream(...)`, mis annavad sama tulemuse ilma
> koodekita (ja `bit_depth=8`, kui piiranguks on andmeside maksumus, mitte koodek).

> **EXIF-andmed kantakse ekspordisse üle.** `process()` kopeerib allika salvestuse GPS-ploki
> **ja selle ExifIFD** igale tootele, seega sisaldavad ekspordid `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` ja `CameraSerialNumber` ning
> georeferentsi. `FocalLength` on see, mille alusel Pix4D arvutab maapinna proovivahekauguse – ilma selleta
> langeb rekonstruktsioon äärmiselt valele mõõtkavale (ühel mõõdetud juhul muutus 411 m suurune ala
> 47,8 km suuruseks). Koopia ei ole tahtlikult `-all:all`: IFD0 struktuurimärgised rikuvad
> LATTICE väljundit, ning `ExifImageWidth`/`Height` on välja jäetud, kuna need kirjeldavad allika
> pildistamist, mitte eksporditud rasterit.

Salvestusetapi alamlipud (kehtivad radiomeetriliste tasemete kohta — `radiance`, `reflectance`, `display`):

| Lipuke | Vaikimisi | Tähendus |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + tasandusvälja + 3x3 segunemise eemaldamine + NIST-i radiomeetriline skaala. |
| `apply_white_balance` | `True` | WB LUT. DLS-tundlik, kui DAQ on kaameraga seotud. |
| `apply_index` | `False` | Taimestikuindeksi hindamine. |
| `index_expression` | `None` | Valemi ümberkirjutamine. Tühitühi → lülitab indeksi automaatselt sisse. |
| `annotated` | `False` | GUI-kaunistuste (zebra/võrk/piik) pealekandmine. Ei ole saadaval `raw` jaoks. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Tagastatav väärtus on `chloros_sdk.CapturePaths`, tüübiga `CapturePathMap`, mitte `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` on `Dict[str, Union[str, List[str]]]`: ühetasandiline
> `processing` annab igale seerianumbrile ühe tee, samas kui `processing="all"` annab sellele
> **järjestatud nimekirja** kõigist selle kaamera jaoks salvestatud toodetest. `ArrayHandle.capture()` ei võta
> vastu `levels=`-i ning ignoreerib talle tundmatuid märksõnaargumente (`levels`, `daq`,
> `render_index`, `force_daq`), seega edasta ainult siin näidatud argumendid (`aligned=False`
> salvestab algupäraseid faile, nagu ka `ArraySession.capture()`); kasutage
> `ArraySession.capture()` nende juhtelementide jaoks. Reaalajas kombineeritud komposiit, kui seda voogedastati, saabub täiendava
> `"combined"` võtme all, mitte seerianumbri all. Kood, mis eeldab `str`-i, annab vea
> loendivormis, ilma et tüübikontrollija sellele vastu oleks – märkus ütles, et `Dict[str, str]`
> veel mõnda aega pärast loendivormi väljalaskmist, mistõttu see alias üldse olemas on. Normaliseeri
> siis, kui soovid lamedat vormi:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Mida ei saanud kirjutada: `.failed`.** Kaart, mida `capture()` tagastab, on
> `chloros_sdk.CapturePaths` — lihtne alamklass `dict`, seega kõik eespool toodud toimib
> muutmata kujul — mis sisaldab ka `.failed`: faile, mida tagapõhi ei suutnud kirjutada,
> samad kanded kui [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error` ja `disk_full: True` täis kettal). Osaline
> salvestamine näitab ikkagi kõiki kirjutatud teekondi; täis kõvakettal on kaart
> tühi ja `.failed` selgitab, miks. `.skipped` loetleb kaamerate kaupa vahelejäetud kaadrid ja
> `.group_seq` sünkroniseeritud rühma, ning `.alignment` / `.oriented` on
> tagapõhja omad, nagu ka `CaptureResult`. `capture_smart()` tagastab need kui
> `result["frames"].failed` ja `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Massiivi joondamine

`ArrayHandle` avab kogu joondamispinna. Profiilid on vaikimisi seansipõhised — püsivaks salvestamiseks tuleb selgesõnaliselt kutsuda `export_alignment()`.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Ühendamisel toimuv joondamine

`connect_all(align=...)` suudab iga massiivi ühendamisel automaatselt joondada:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Kui seda ei ole määratud, kasutatakse tagavarana `project.json["config"]["auto_align_on_connect"]`-i.

### `SensorHandle`

Salvestatud valgusandur (DAQ-U, DAQ-M või DAQ-E), mille avab `connect_all()` tagapõhja [DAQ-andurite reservis](#daq-sensor-sessions) – samas reservis, mida kasutavad ka `connect_daq_sensor()` ja `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` ootab anduri **järgmist** spektrit — see, mis on juba puhverdatud käivitamise hetkel, ei loe — ja tagastab `(spectrum, is_saturated, integration_time, x, y, z)` või `None`, kui 5 sekundi jooksul ei saabu ühtegi. Peatatud voog taastatakse. `spectrum` on väljendatud W/m²/nm, kui andur on kalibreeritud.
- `.sensor` on anduri [`DAQSensorSession`](#andmekogumise-andur-seansid), kõigi muude juhtude puhul: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` ja `.settings` on salvestatud kirjed failist `sensors.json`.

**Mida `connect_all` taastab.** Iga andur avatakse selle salvestatud aadressil — selle COM-port, Bluetoothi aadress või DAQ-E host — ning kui projekt on salvestanud anduri ID, peab tegemist olema just selle anduriga: teist seadet sellel aadressil ei aktsepteerita, see jääb ühendamata. Rakendatakse selle salvestatud integratsiooniaega, kaadri keskmistamist ja automaatset säritust ning selle kateparandust:

| Salvestatud kaamera | Ühendatud |
| --- | --- |
| Kaamera profiil (nt `fov_30`) | See profiil. |
| Valimata | `sunshine_cosine`, ühenduse vaikimisi valik, salvestatud eeldatuna (`cap_id_source=auto_default`), mistõttu töötlemine annab hoiatuse. Andurid tarnitakse ilma kaaneta: valige paigaldatud kaan. |
| `none` — valitud kasutajaliideses **Puudub (paljas andur)** | Kaanekorrektsiooni ei rakendata. |

DAQ-E-S ei võta arvesse ühtegi kateprofiili, olenemata sellest, mis oli salvestatud: selle hajuti on osa kalibreerimisest (vt [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Varasema Chlorosi poolt salvestatud projektis oli palja anduri valik puudus, seega ühendub see uuesti mudeliga `sunshine_cosine`; valige kasutajaliideses uuesti **Puudub (paljas andur)** ja salvestage projekt.

Andur, mille mõni teine klient on juba avanud andurite kogumis (CLI-s, mõnes teises skriptis), ühendatakse sellisena, nagu see on, ilma seadeid uuesti rakendamata, ja `disconnect_all()` jätab selle töötama; `connect_all()` poolt avatud andur vabastatakse. Nagu `connect_daq_sensor()` puhul, ei kuulu Chlorosi kasutajaliides **mitte** andurite kogumisse: sulge andur GUI-s enne projekti skriptist ühendamist.**Nõuded ja aruanne.** Valgusandurid vajavad töötavat Chlorosi tagapõhja, mis on sisse logitud tasulisele Chloros+ tasemele (Copper või kõrgem), ning `connect_all()` ei käivita seda kunagi. Kui ükski ei tööta, on iga salvestatud andur loetletud `report["sensors"]["errors"]` all, kus seda ka märgitakse. Ka andur, mida backend ei suuda avada, on seal loetletud koos backendi põhjendusega; `report["sensors"]["missing"]` jääb valgusandurite puhul tühjaks.

---

## Otsene riistvara (backendita)

Kui soovid, et LATTICE-kaamerad (CI, peata robotid, sisseehitatud) ei sõltuks üldse backendist, impordi otse `lattice_sdk` — seda eksporditakse uuesti `chloros_sdk` kaudu. Hoiatus seoses `CAMERA_AVAILABLE`-iga: `lattice_sdk` on SDK-i wheelis, kuid vajab Arena SDK-i käivituskeskkonda ja OpenCV-d (`cv2`), mida wheel ei installi. Valgusandurid töötavad backendi kaudu — [`connect_daq_sensor()`](#daq-sensor-sessions) või projekti `proj.sensors` kaudu — ühe erandiga, milleks on DAQ-M sagedusriba-suurusjärgu abiprogramm [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), mis vajab samuti `bleak`-i.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Eelseadistused ja päästik

Neljast eelseadistusest kolm on **vabajooks**: kaamera pildistab pidevalt ja
`capture()` tagastab järgmise kaadri. `triggered` on erand — see seadistab
kaamera ootama riistvaralist serva 2. liinil, seega ei jäädvusta see midagi enne, kui see saabub.

| Eelseadistus | Trigger | Kasutada, kui |
| --- | --- | --- |
| `default` | vabajooks | üldine kasutus |
| `high_speed` | vabajooks | 8-bitine, maksimaalselt 60 kaadrit sekundis, lühike säriaeg |
| `high_quality` | vabajooks | 12-bitine, kaadrisageduse piirang puudub — tavaline valik fotode jaoks |
| `triggered` | **valmis, Line 2** | kaamera on ühendatud M8 sünkroniseerimiskaabliga ja midagi muud käivitab selle |

Kui valite `triggered` (või seadistate ise `trigger_mode="On"`) nii, et
liinil 2 pole midagi, siis iga `capture()` aegub — õigesti, kuna sa palusid
kaameral oodata. SDK selgitab seda, kui see juhtub; vaata
[SC_ERR_TIMEOUT salvestamise ajal](#direct-hardware-backend-free).

> **Märkus — „GVSP probe“ / `SC_ERR_TIMEOUT -1011`-sõnumid ühenduse loomisel ei ole vead.**
> Ühenduse loomisel püüab SDK läbirääkimisi pidada**jumbo-raamide** (9000-baidised GVSP-paketid) üle, et saavutada suuremat läbilaskevõimet. Otsese punkt-punkt-võrgukaardi ühenduse korral (nt link-local `169.254.x.x`-aadress) ei suuda võrk tavaliselt jumbo-raame edastada, mistõttu see proov aegub ja logib selliseid ridu nagu:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> See on **ette nähtud tagavarameede**: SDK naaseb automaatselt standardse 1500-baidise paketi juurde ja kaamera jätkab ühenduse loomist tavapäraselt (järgnevad `[chunk-enable …]`-read on osa tavalisest ühendusprotsessist). Pildistamine toimib endiselt.
>
> Sa võid selle proovi vahele jätta, kuid **see ei ole lihtsalt logide summutaja – see lülitab jumbo-raamid välja.** Kaamera vastab „Don&#x27;t-Fragment”-pingidele ainult kuni 1500 baiti, olenemata sellest, kui hea on teie võrk, seega ping-test üksi ei suuda kunagi jumbo-raame tuvastada; ainult see proov suudab seda teha. Lülitage see välja ja kaamera kasutab igavesti standardseid 1500-baitiseid pakette, mis tahes võrgus:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> See tasub end ära ainult võrgus, mille kohta *teate*, et see ei toeta jumbo-raame, kus see säästab umbes ühe sekundi ühendusaega iga kaamera kohta. Kuna tegemist on pigem tegeliku kui kosmeetilise kompromissiga, märgib SDK seda nüüd, kui seda kasutad:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Jäta see rahule, kui sul pole selleks põhjust.**Kui see on sisse lülitatud, mõõdetakse iga kaamerat üks kord iga backend-seansi jooksul ja uuesti, kui selle ühendus muutub (teine adapter, teine ühenduskiirus või ühenduse katkemine, nagu lüliti vahetamisel juhtub); uuesti ühendamisel kasutatakse tulemust uuesti.**Pärast muutust lüliti taga, mis jäi tööle, taaskäivita backend, et uuesti mõõta.** Esimene ühendus võib selle kontrolli puhul aeg-ajalt võtta paar minutit aega ning logis on sellisel juhul vastav märge.
>
> Kui *soovite* jumbo-läbilaskevõimet, aktiveerige jumbo läbivus (NIC MTU 9000 + lüliti, mis neid läbi laseb) või fikseerige see `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`-iga, kui teate, et ühendus seda toetab — kuigi eelistage käsukohast `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …`-i selle püsivale seadistamisele, kuna fikseeritud suurus jätab proovi vahele ja lõpetab kohanemise eesoleva võrguga. **Iga** seadme teel peab jumbo-pakette edastama — sealhulgas kõik PoE-jagajad või -injektorid, mis on tavaline põhjus, miks muidu jumbo-pakette toetav seadistus neid edastada ei suuda.

> **`SC_ERR_TIMEOUT -1011` `capture()` / `grab*()` ajal on teine probleem — see on tõeline viga.**
> Eespool toodud märkus puudutab ainult**ühendusaegse proovi**poolt registreeritud `-1011`-i. Sama viga, mis tekkis**salvestuse** käigus, tähendab, et kaamera ühendus õnnestus, kuid see ei saada ühtegi pilti:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Selle tunnuseks on kaamera, mille *juhtimiskanal* on korras — avastamine toimib, seadistused ja `[chunk-enable …]`-kirjutused õnnestuvad kõik — samas kui *iga* kaadri puhul tekib ajapiirang.
>
> **Tavaliselt on põhjuseks see, et kaamera on seadistatud riistvaralisele päästikule.** Koodide `trigger_mode="On"` ja `trigger_source="Line2"` korral ei saada kaamera midagi, kuni M8 sünkroniseerimiskaablile jõuab elektriline signaal. Kui sellel liinil pole ühtegi kaablit, ootab iga kaadri salvestamine igavesti. Kaamera ei ole katki ja võrk töötab korralikult — see teeb täpselt seda, mida talle on kästud.
>
> `CameraSettings()` ning `default` / `high_speed` / `high_quality` eelseadistused töötavad vabajooksurežiimis ning ajastatud pildivõtt, mis aegub aktiveeritud olekus, selgitab seda ise, selle asemel et kuvada lihtsalt `-1011`. `PRESETS["triggered"]` aktiveerib Line2, nagu ette nähtud.
>
> Mis tahes kaamera vabajooksu sundimiseks:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Kui ajapiirang ikkagi lõpeb `trigger_mode="Off"` korral, siis kaamera tõepoolest andmeid ei edasta — saatke meile logi ja `ip link show`.

#### Värviprofiilid (RGB-reaalaja eelvaade) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` valib RGB-kaamerate **reaalaja eelvaate** jaoks ekraani värviprofiili (multispektraalsed kaamerad ignoreerivad seda seadet):

| Profiil | Tähendus |
| --- | --- |
| `raw` | Jäta radiomeetriline ahel täielikult vahele. |
| `linear` | DSNU + flat + WB, ilma CCM-ita, ilma gammata. |
| `natural` | Lineaarne + mõõdetud CCM + sRGB gamma, ainult odava viimistlusega (kroma silumine + heledate alade desaturatsioon) — realistlik vaikimisi seade. |
| `enhanced` | `natural` pluss täielik hub-parity viimistlus (värvirebendite eemaldamine, värviküllus, CLAHE kohalik kontrast). Rikkalikum välimus, mille raami kohta kuluv viimistluskulu on ligikaudu **kaks korda suurem**, seega madalam LIVE kaadrisagedus. |
| `custom_temp` | `natural`, kuid valgebalanss fikseeritud `custom_cct_k` Kelvinile (DLS-i ignoreeritakse; piiratud vahemikuga 2000–10000 K tagapoolel). |

Profiil on **ainult reaalajas eelvaate jaoks** mõeldud kiiruse/välimuse reguleerimisnupp: salvestatud kaadrid saavad alati täieliku ja rikkaliku viimistluse, sõltumata valitud profiilist, seega `natural` valimine kaadri aja kokkuhoiuks ei vähenda kettale salvestatava materjali kvaliteeti. Tundmatu profiil tõstab väärtust `ValueError`; kui chloros-tagapõhi on kättesaadav, saadetakse muudatus ka sinna POST-päringuga, nii et järgmine eelvaatekaader seda kajastab (otsese SDK-kasutajad, kellel tagapõhja pole, saavad seadete muutuse ikkagi kätte).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Esiletõstetud alad (RGB-ekraan) — `set_highlights`

`LatticeCamera.set_highlights(value)` vähendab **ekraani** kõige heledamaid ja valgeimaid piksleid RGB-kaameratel — parandus heledale valgele pinnale, mis kuvatakse ühtlase valge laiguna. `value` on `-100`..`0` (piiratud); `0` on välja lülitatud ja renderdamine on sel juhul täpselt selline, nagu see oleks ilma selle seadistuseta. Tugevamad seadistused vähendavad ekraani valget (umbes 60%ni tasemel `-100`) ja laiendavad toonikõverat, nii et pind, mille eelvaate automaatne heledus oli muutnud puhtaks valgeks, saab oma detailid tagasi.

Tegemist on ekraani reguleerimisega, kuid erinevalt küllastusest ja kontrastist (mis muudavad ainult reaalajas eelvaadet) muudab see ka seda, mida salvestate: reaalajas eelvaade ja **ekraanitasandi** pildid järgivad seda, seega tulevad ka need failid tumedamaks (neil on seadistus `Chloros:DisplayHighlights`); kiirguse ja peegelduse ekspordid seda kunagi ei tee ning see ei suuda taastada klippunud andureid – selleks tuleb vähendada säritust. Multispektraalsed ja monokroomkaamerad ignoreerivad seda. Väärtus salvestatakse `settings.highlights`-i ja POST-itakse kättesaadavasse chloros-tagapõhja, täpselt nagu `set_color_profile`; selle CLI-ekvivalent on `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Mono (M3M) kaamerad ja `Calibration`

Mono **M3M** kaamera (`M3M-<lens>-F<wavelength>`) on ühe lainepikkusega: üks halltoonide tasand, puudub Bayeri mosaiik, puudub 3×3 spektraalne ristkõne maatriks. `Calibration` tuvastab selle ja näitab lipukest `is_mono`. Peegeldus kehtib endiselt ribapõhise radiomeetrilise kaardina (segamise tulemuseks on identiteetmaatriks), kuid ühe kaamera puhul mitmeribaline matemaatika annab pigem tulemuse kui mõttetut väljundit:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Et luua taimestiku indeks mono-riistvaraga, kombineerige mitu erinevatel lainepikkustel töötavat M3M-kaamerat joondatud mitme lainepikkusega pildistamiskogumiks (vt [Pildistamiskogumi joondamine](#array-alignment)) ja arvutage indeks kogu selle kogumi põhjal, mitte ühe kaamera põhjal.

#### DAQ-M-i lainepikkuste tugevused — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` ühendub DAQ-M-iga Bluetoothi kaudu oma BLE MAC-i abil. See on ainus valgusanduriklass, mis suhtleb anduriga otse, mitte tagapõhja kaudu, seega vajab see seda, mida vajavad otsesed kaameraklassid (`CAMERA_AVAILABLE`: Arena SDK-i käitamiskeskkond ja OpenCV), pluss `bleak` Bluetoothi jaoks (`pip install "chloros-sdk[camera]"` või `pip install bleak`; ilma selleta tekitab `DLS` vea `DLSError`). Kui mujal ühendatud DAQ-M (tagapõhja pool, projekt, graafiline kasutajaliides) lõpetab reklaamimise, seega vabasta see esmalt sealt. **`mac_address` on kohustuslik**: vaikimisi väärtust ei ole, kuna igal seadmel on oma, ja `DLS()` ilma selleta käivitab `DLSError` enne, kui ühendust proovitakse luua. Leia MAC-aadress XPR-igaOTX003351 abil (vt [Avastamine](#discovery--finding-an-address-to-connect-with)) või `chloros-cli daq pool-discover --only ble` abil.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integreerib DAQ-M spektri (viimane näid, kui `spectrum` on `None`; `DLSError`, kui seda veel pole) kaamera kolme kanali vastuste kaudu ja tagastab `{"red": …, "green": …, "nir": …}` — võtmed nimetavad kanaleid selles järjekorras; RGB-kaamera kõverate puhul on kolmas (`nir`) selle sinine kanal.

- Ilma `spectral_response`-ita kasutatakse filtri mudeli tüüpkõverat (mille alusel `filter_model` ja `DLS` ehitati), mis mõõdeti ühe seadme puhul.
- `spectral_response` kasutab selle asemel kaamera **enda** mõõdetud kõveraid: sõnastik `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, mis on salvestatud selle kaamera kalibreerimispaketti (`color_correction.spectral_response_smoothed` — RGB-kaamerad; multispektraalsed paketid ei sisalda sellist kõverat). Kui sõnastikku ei saa kasutada, kasutatakse tagavarana seeria kõverat.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLI-i `chloros-cli lattice dls --camera SERIAL` teeb sama tagapõhja kaudu, leides kaamera kõvera selle laaditud või vahemällu salvestatud kalibreerimispaketist ning näitab, kelle kõverat kasutati. HTTP-i kaudu on see `POST /api/dls/cli-read` koos `mac` (nõutav: ilma selleta on vastuseks HTTP-viga 400, `code` `MAC_REQUIRED`) ja `camera_serial` vastuse sisus; vastuse `band_magnitudes_source` on `camera <serial>`, kui kasutati selle kaamera enda kõverat, või `model <filter>` seadmepargi kõvera puhul (kaamera, millel pole kõverat: pole RGB-d või pole paketti laaditud või vahemällu salvestatud).

---

## Erandid

Kasutage baasklassi, et käsitleda „kõiki Chlorosi vigu”:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` ja `ChlorosConfigurationError` on `chloros_sdk` (ja `chloros_sdk.exceptions`) atribuudid, kuid mitte `__all__`-is, seega `from chloros_sdk import *` neid sisse ei too: impordi need nime järgi. `ChlorosError`-perekond tuleneb `ChlorosLocal`-ist ja sellele rajatud funktsioonidest (`process_folder()`, `process_lattice_capture()`); smart-connect-pind tekitab `ChlorosConnectError`.

Hierarhia:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Läbivad näited

### 1. Kausta töötlemine kohandatud edenemisribaga

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Reaalajas LATTICE-massiiv → peegeldusvõime + DAQ-viide

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektipõhine salvestuskampaania

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Mitme kaamera kaadrivoog → NumPy-töötlusketi

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Peata otse riistvarale (ilma tagapõhja) suunatud salvestamisskript

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Võimekuse kontroll enne 4-kaameraga massiivi ühendamist

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Salvestusretsepti ekvivalent (puhas Python)

`run_project_recipe()` käivitab CLI-retsepti sellisena, nagu see on (vt [Projektikausta ja retseptid](#project-folder--recipes)). Pythonis kirjutatuna on samad sammud järgmised:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP-server (tehisintellekti abilised)

`chloros-mcp` võimaldab [Model Context Protocol](https://modelcontextprotocol.io) (MCP) keelt kõneleval AI-assistendil, nagu Claude Desktop, Claude Code või Cursor, juhtidChloros&#x27;i: töödelda pilte (luua projekt, importida pildikausta, valida seaded, töödelda, anda aru, mida käivitamine eksportis) ning juhtida LATTICE-kaameraid, maatrikseid ja DAQ-valgusandureid (ühendada, eelvaadata, seadistada säritust ja värvi, pildistada, salvestada, uuendada püsivara). Selle tööriistad hõlmavad kõiki selle SDK funktsioone. Te esitate küsimuse lihtsas keeles ja assistent kutsub välja serveri tööriistad:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Server põhineb sellel SDK-l: töötlemistööriistad kutsuvad esile [`ChlorosLocal`](#image-processing--chloroslocal) ning riistvaratööriistad [smart-connecti kutsed](#smart-connect-for-lattice-cameras), [DAQ-seansse](#daq-sensor-sessions), `ChlorosProject` ja [kaamera abiprogramme](#camera-utilities), mis kõik suhtlevad sama tagapõhjaga, mida kasutab Chloros-rakendus. Nende päringud kannavad koodi `X-Chloros-Client: sdk`, seega rakendab tagapõhi SDK sisselogimise ja paketi kontrolli ning sellega kaasnevad SDK reeglid (näiteks keelatakse LATTICE-pildistuste puhul Texture Aware).

### Paigaldamine

Server on kaasas `chloros-sdk` paketis; muud pole vaja paigaldada. SDK paigaldab käsu `chloros-mcp`, mis on sama kui `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** Chlorosi paigaldaja paigaldab SDK&#x27;i pip-i abil teie Pythonisse, mis paigutab `chloros-mcp.exe` selle Pythoni `Scripts` kausta.
- **Linux (.deb):** pakett installib SDK-i süsteemi `python3` ja paigutab käsu `chloros-mcp` kataloogi `/usr/bin`, nii et see asub iga kasutaja `PATH`-is. Virtualenvis lisab `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` `chloros-mcp` virtualenvi `bin`-i.

Server kasutab ainult SDK-i (Python 3.7+) peal olevat standardteeki. MCP-klient käivitab selle ja suhtleb sellega stdin/stdout kaudu; te ei käivita seda ise, välja arvatud `--help` puhul.

### Kliendi seadistamine

**Claude Desktop:**lisage server `claude_desktop_config.json`-i (Windowsis `%APPDATA%\Claude\claude_desktop_config.json`; avamiseks valige**Settings &gt; Developer &gt; Edit Config**), seejärel taaskäivitage Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude&#x27;i kood:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Kursor:** sama `mcpServers`-kanne nagu Claude Desktopis, asub `~/.cursor/mcp.json`-is (kõikides projektides; Windowsis `%USERPROFILE%\.cursor\mcp.json`) või `.cursor/mcp.json` ühes projektis.

Valikud sisestatakse `args`-i või Claude&#x27;i koodi käsu järele (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Et kasutada konkreetse Pythoni SDK-i asemel `chloros-mcp`-i `PATH`-is, käivita moodul selle Pythoniga (selle täielik tee on `command`, kui see ei ole esimene `python3` `PATH`-is):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Klient käivitab serveri ilma teie shellita, seega kui ta teatab, et serveri käivitamine ebaõnnestus, ei ole `chloros-mcp` tõenäoliselt kliendi poolt nähtaval `PATH`-is. Märkige selle asemel täielik tee (`where chloros-mcp` Windowsis, `which chloros-mcp` Linuxis, kus .deb-failid asuvad `/usr/bin/chloros-mcp`), või käivitage moodul Pythoniga, millel on SDK.

### Valikud

| Valik | Vaikimisi | Mida see teeb |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Kasutatav Chlorosi tagapõhi. |
| `--no-auto-start` | off | Tagapõhja ei käivitatagi: kasutatakse juba töötavat tagapõhja, näiteks Chloros-rakenduse oma, Linuxi tagapõhja teenust või tunnelit teise masina (vt [Kaug-backend-režiim](#remote-backend-mode-pip-only-host-via-tunnel); sel juhul peavad kaustateed viitama mõlemal masinal samale kaustale). |
| `--backend-exe PATH` | kuhu Chloros selle installib | Tagapõhi, mida käivitada, kui ükski ei tööta, juhul kui Chloros on installitud kohta, mida SDK ei otsi. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` või `ERROR`. Logid suunatakse stderr-i, mida enamik kliente kuvab oma MCP-logis; stdout edastab ainult protokollisõnumeid. |

### Tööriistad

Iga tööriista argumendid on samad, mis selle poolt ümbritsetud SDK-kõnel; klient loetleb igaühe koos täieliku skeemiga. Teed on absoluutsed.

**Töötlemine**

| Tööriist | SDK-kõne | Mida see teeb |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Loob projekti ja teeb sellest avatud projekti, mille suhtes töötlemistööriistad tegutsevad. Lihtne nimi loob kausta vaikimisi projektikausta; absoluutne tee loob selle sinna. |
| `open_project` | `ChlorosLocal.open_project` | Avab salvestatud projekti ja teeb sellest avatud projekti. Ümberpaigutatud projekti teed suunatakse ümber kaustale, kus see praegu asub; `relink_folder` (absoluutne) nimetab kausta, kus asuvad teised puuduvad failid. Tagastab `source_files` ja `image_count`. Ei leitud: veateade, milles on loetletud nende asukohad ja nimetatud `relink_folder`; midagi ei ole avatud. |
| `import_images` | `import_images` | Impordib Survey3 `.RAW`+`.JPG` paare, „LATTICE“ `.tif` salvestused ja DAQ `.daq` salvestused; tagastab `count` (pildid) ja `scan_count` (salvestused). |
| `configure_processing` | `configure` | Muudab ainult edastatud seadeid ja mitte muid (väärtused: [Toetatud väärtused](#supported-values)); `custom_settings` saadab kogu konfiguratsiooni sellisena, nagu see on antud. |
| `get_project_settings` | `get_config` | Loeb avatud projekti salvestatud töötlemis-, ekspordi- ja indeksiseaded. |
| `start_processing` | `process(wait=False)` | Alustab avatud projekti töötlemist ja naaseb kohe. |
| `get_processing_status` | `get_status` | Annab teada `state` (`idle`, `starting`, `processing` või `finished`) ja edusammud; kui töö on lõpetatud, kirjutatakse käivituse `summary`, `export_check_failure`, kui eksporti ei olnud võimalik kontrollida, ja `image_failure`, kui pildid kadusid: ülejäänud tooted kirjutatakse; märgi käivitus ebaõnnestunuks. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Loob kausta jaoks projekti, impordib selle, konfigureerib ja käivitab käivituse ühe käsklusega (`lattice: true` LATTICE-pildistamiste puhul). Kui käivitus on käimas, lükkab süsteem selle tagasi enne, kui midagi loodud saab, ning ka tagapõhi lükkab sel ajal tagasi projekti muutmise mis tahes tööriistast. |
| `export_light_sensor` | `export_light_sensor` | Kirjutab projekti valgusanduri salvestuste jaoks kalibreeritud `.daq` ja `.csv` failid salvestuste jaoks kalibreeritud `.daq` ja `.csv` faile, ilma pilte töötlemata, ning laadib MAPIRi pilvest tehase kalibreerimise andmed, kui need pole vahemällu salvestatud (vt [Valgusanduri salvestused](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Loeb töödeldud pildi Chloros-auditi metaandmeid (allikas, kalibreerimine, rakendatud etapid). |

**Kaamerad ja maatriksid**

| Tööriist | SDK-kõne | Mida see teeb |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Mis on hetkel ühendatud, mis tahes kliendis, igaühel koos `connected_here`-iga; lisaks serveri käimasolevad tööd. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kaamerad ja valgussensorid, mida võiks ühendada. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Ühe kaamera, massiivi (koos selle võimekuse ja seisundiga) või valgussensori reaalajas seisund. |
| `analyze_array_network` | `analyze_array_network` | Mida see host ja võrk suudavad kavandatava massiivi jaoks edastada. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Ühenda kaamera või sünkroniseeritud kaamerarivi (esmalt peakaamera). |
| `connect_project_devices` | `open_project(...).connect_all` | Ühendage kõik seadmed, mida salvestatud projekt mäletab, koos nende seadistustega. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Vabasta kaamera, massiiv või valgussensor (vt „Käitumine”). `keep_settings: true` massiivil salvestab iga kaamera seaded oma mällu, `clear_settings: true` tühistab selle mälu (selle tulemuseks on `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Projekti seadmed kokku. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Kaamera seadete lugemine või kirjutamine, kasutades sõbralikku nime või GenIC-iam-sõlme nime järgi. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Määra massiivi kiirus kindlaks või taasta automaatne kiirus. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Üks pildistamine (`fastest` ja `wait_for_exposure` on `capture_fastest` ja fikseeritud säriajaga pilt). Kui `output_dir` puudub, salvestatakse need avatud projekti`captures` kausta ning failid lisatakse projekti (hoitakse kinni kuni töötlemistsükli lõpuni); vastuses `project_add` on see märgitud. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Pidev või intervallpildistamine, tööna. |
| `start_array_recording` | `ArraySession.record` / `burst` | Kombineeritud indeksiga video või töötlemata pildiseeria, tööna. |
| `build_burst_video` | `ArraySession.build_video` | Pildiseeria ümbertöötlemine kalibreeritud videoks, tööna. |
| `get_job_status` / `stop_job` | | Töö jälgimine või peatamine. |
| `get_camera_preview` | | Iga kaamera reaalajas eelvaade, mis tagastatakse pildina. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Üks kaader (või üks sünkroniseeritud komplekt) töötlemistasandil, salvestamata: statistika iga tasandi kohta ja igaühe kohta PNG-fail. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Massiivi joondus. |

**Valgusandurid**

| Tööriist | SDK-kõne | Mida see teeb |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Ühendab DAQ-U, DAQ-M või DAQ-E (`cap_id` seadistab selle kattekorrektsiooni). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Viimased mõõtmistulemused: ajamärge, küllastus, spektri ulatus ja tipp ning lainepikkuse vahemikesse keskmistatud spekter (`full_spectrum` iga punkti kohta). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Andmevoo käivitamine või peatamine. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Salvestab `.daq`-faili tööna; `duration_s` peatab selle ise. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Kattkorrektsioon (`none` palja anduri puhul). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Milline andur edastab iga kaamera ja maatriksi peegeldusvõime. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Lainealade komplektid ning kahe vooganduri ümbritseva keskkonna ja objekti peegeldusvõime ning indeksid. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | DAQ-E seadme enda seaded, mis on salvestatud seadmesse. |

**Kaamera utiliidid** ([Kaamera utiliidid](#camera-utilities))

| Tööriist | SDK-kõne | Mida see teeb |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` ja nende seadistajad | RGB-kaamera ekraani värv: profiil ja värvitemperatuur, küllastus / kontrast, heledad alad. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Ühekordne valgebalanss või automaatne / välja lülitatud / fikseeritud suhted. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Kaamera eelseadistused. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Filtrimudelid või üks nimepõhiselt. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Püsivara versioonid ja välklamp (vaikimisi töö). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Toite- ja soojusolek; puhke- või ärkamisrežiim. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Vaates oleva kalibreerimise sihtmärgi koefitsiendid. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Kaamera võrgukaardid, hinnang ja parandus (ainult administraatorile). |
| `apply_vignette_correction` | `apply_vignette_correction` | Piltide kausta vinjeti korrigeerimine. |
| `read_dls` | `read_dls` | Üks DLS-lugemine. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Hosti riistvara ja töökeskkond. |

**Transient-kaamerad, joondamisprofiilid ja indeksid** ([Transient-massiivid, joondamisprofiilid ja indeksid](#transient-arrays-alignment-profiles--indices)). Iga siin olev tööriist, mis avab kaameraid, võtab `keep_settings` / `clear_settings` (vaikimisi false): vt [Kaamera mälu](#camera-memory--keep_settings--clear_settings).

| Tööriist | SDK-kõne | Mida see teeb |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Salvestab ühe kaamera või edastab sünkroonselt mitme kaamera voogu ühe ülesandena (`duration_s` peatab selle ise). |
| `capture_cameras_transient` | `capture_cameras_transient` | Üks sünkroniseeritud salvestus ilma massiivi ühendamiseta. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Triggeri sünkroniseerimise kontroll ja millised kaamerad on sünkroniseerimisliinil. |
| `analyze_array_models` | `analyze_array_models` | Mida selliste mudelite kaamerate massiiv selles võrgus saavutaks. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Lahenda ja salvesta joondamisprofiili fail (`options` sisaldab täpsemaid lahendamisvalikuid). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Joondatud mitme sagedusribaga pilt või nende voog ühe tööna. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Profiilifaili sisu; selle ribade järjekord ja aktiveeritud kaamerad. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Eelseadistuste ja gradiendi indeks; mitme sagedusribaga TIFF-faili indeks; pildistamine ja indeksimine ühes etapis. |

**Aja sünkroniseerimine, projektikausta ja retseptid** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Projektikausta ja retseptid](#project-folder--recipes))

| Tööriist | SDK-kõne | Mida see teeb |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | PTP-grandmasteri seisund, selle võrdväärsed seadmed ja iga massiivkaamera PTP-seisund. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Käivita see uuesti; määra selle prioriteedid. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Kaust, kuhu luuakse uued projektid. |
| `run_project_recipe` | `run_project_recipe` | Käivita salvestatud projekti retsept tööna koos selle logiga. Esmalt kontrollitakse: veaga retsept lükatakse tagasi ja midagi ei käivitatata; ühendamata massiividest / anduritest antakse hoiatus (neid ei ühendata: kõigepealt `connect_project_devices`). `dry_run` lõpetab kontrolli ja midagi ei käivitatata. |

Salvestatud projekti töötlemiseks avamiseks pole vahendit: iga töötlemisülesanne algab `create_project`-iga (või `process_folder`-iga).

### Nõuded

- **Chloros peab olema installitud samale arvutile.** Töötlemise teostab Chlorosi tagapõhi. Server võib selle asemel kasutada tagapõhja, millele pääseb ligi tunneli kaudu (`--backend-url` koos `--no-auto-start`-ga, vt [Valikud](#options)).
- **Sisselogitud sellel arvutil**, kas Chlorosi rakenduses või `chloros-cli login` kaudu. Server ei saa teie eest sisse logida: kui tööriist käivitatakse väljalogitud olekus, vastab see „Chlorosisse pole sisse logitud“ koos tagapõhja põhjendusega, ja assistent teavitab teid sellest.
- **Mis tahes tasuline Chloros+ tasand (Copper või kõrgem).** Serveril on SDK-juurdepääs, aga tasuta Iron-tasandil seda pole: tasuta tasandil käivitatud tööriist vastab „Sellel Chloros-paketil pole saadaval” koos backendi põhjendusega.
- Python 3.7 või uuem koos SDK-ga ning MCP-klient, mis käivitab kohalikke (stdio) servereid.

### Tööpõhimõte

- **Backend.** Serveri käivitamine või selle tööriistade loetlemine ei käivita midagi; esimene tööriista käsk loob ühenduse. Kui Chloros-rakendus (või Linuxi backend-teenus) töötab, kasutab server selle backendit ja jätab selle töötama. Kui ükski backend ei tööta, käivitab server installitud backendi aknata. Kui server sulgub (kui sulged kliendi või kui server taaskäivitatakse), peatab ta enda poolt alustatud pildistamisülesanded ja valgussensori salvestused ning vabastab ühendatud seadmed; backend jätkab töötamist, nii et pooleliolev töötlemisprotsess jätkub lõpuni ning ta sulgeb end ise 30 minuti pärast, kui keegi seda ei kasuta. `--no-auto-start` puhul ei käivita server kunagi taustteenust: kui taustteenus ei tööta ja tööriista kutsutakse, palutakse sul avada Chloros-rakendus ning järgmine kutse pärast seda loob ühenduse.
- **Üks avatud projekt.** Tagapõhisel süsteemil on üks avatud projekt, mida jagatakse rakendusega Chloros, kui rakendus on avatud, seega ärge töötage rakenduses teise projektiga, kui abiline töötab.
- **Muutuvad ainult edastatud seaded.** `configure_processing` jätab kõik seaded, mida talle ei ole edastatud, nende salvestatud väärtustele (SDK `configure()` taastab debayeri, vinjeti korrigeerimise, peegelduskalibreerimise, PPK ja ekspordivormingu vaikimisi väärtustele, kui neid ei edastata; server täidab need projekti andmetega). Kui ta ei suuda projekti salvestatud seadeid lugeda, ei muuda ta midagi ja teatab sellest, välja arvatud juhul, kui kõik viis on edastatud. `indices` asendab projekti indeksiloendi. Tulemus loetleb, mis saadeti `settings_sent` all saadetud andmed ning nimetab `warnings` all kõik indeksid, mida kaamera lainepikkused ei suuda moodustada.
- **Töötlemine toimub põhimõttel „alusta ja küsitle”.** `start_processing` naaseb niipea, kui töötlemine algab, mis võtab aega mõnest minutist kuni mitme tunnini, ja assistent jätkab seejärel käskuga `get_processing_status`. `state` on `idle` (töötlemist ei toimu), `starting` (tagapõhi ei ole käivitust veel teatanud, kuni 20 sekundit), `processing` või `finished`, koos käivituse [kokkuvõte](#post-run-summary--hints): mida eksporditi, ning vihjed, kui eksporditi oodatust vähem (puuduvad pildid, kalibreerimise sihtmärki ei leitud). Tulemused salvestatakse projekti kausta, nagu rakenduseski. `start_processing` lükatakse tagasi, kui käivitus on pooleli, ning tagapõhi, mis lõpetab vastamise, kajastatakse veana, mitte lõpetatud käivituse.
- **Mida assistent näeb.** SDK hoiatused kuvatakse tööriista tulemuses koodi `warnings` all ning vead (välja logitud, plaan, vale tee) kuvatakse veatekstina, mida assistent suudab lugeda ja teile edastada. Kui nimekiri on pikem kui 20 kirjet, näidatakse sellest vaid esimesed 20, täielik pikkus kuvatakse koodina `<key>_total`. Teed peavad olema absoluutsed.
- **Seadmed on jagatud.** Kaamerad, maatriksid ja valgusandurid asuvad tagapõhis, mistõttu saab assistent juhtida ühte neist, mis on ühendatud rakendusega Chloros, ning tehtud muudatused kuvatakse rakenduses. `disconnect_device` kuvab, millega server on ühendatud; seade, millega on ühendatud teine klient (rakendus, `chloros-cli`, skript, teine `chloros-mcp`) vabastatakse ainult `force: true` abil, mis on SDK-i `force_disconnect`, ning tagasilükkamise korral nimetatakse ühendaja, kui backend seda teab; seade, mida backend üldse ei halda, kuvatakse ühendamata olevana (`released: false`, `connected: false`, vabastada pole midagi). Backendi poolt käivitatud salvestus või voog edastab oma `get_job_status`-vaate koos `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (kaadrirühmad mitme kaameraga voo jaoks) — ning üherealine `summary` („220 kaadrit salvestatud, 55 kaotatud, 0 ebaõnnestunud: ...”) selle töötamise ajal ja lõpetamisel selle lõplikud numbrid; teise kliendi loetelu sellest (`other_clients_jobs`) sisaldab sama teavet (`list_cameras()`, `list_arrays()` ja `list_daq_sensors()` sisaldavad iga seadme`connected_by`: `client`, `label`, `name`, `at`). Iga `chloros-mcp` nimetab end tagapõhjas `chloros-mcp (pid N)`-iks (`chloros_sdk.connect.CLIENT_LABEL`, saadetakse kui `X-Chloros-Client-Label`; iga SDK-skript võib selle määrata). Teise kliendi valgussensori salvestamise peatamine võtab samuti `force` aega.
- **Pikk töö on ülesanne.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, ning iga `background`-i jaoks taotletud hõivamine, lahendamine või indekseerimine tagastab kohe `job_id`-i; sellele järgneb `get_job_status` ja `stop_job` lõpetab seda. Töötamine jätkub kuni peatamiseni, kui te ei anna talle `max_captures`, `duration_s` (või, puhul, `max_frames`); `min_free_gb` keeldub käivitumast, kui kõvakettal on vähe vaba ruumi. Piiranguid ei kohaldata, kui te ise neid ei määra. Tagapõhja poolt käivitatud salvestus või voog (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) säilitab backend&#x27;i töö ID, nii et teine `chloros-mcp` — hiljem käivitatud — leiab selle selle ID järgi: `get_job_status` järgneb sellele (ja loetleb selliseid tööid nagu `other_clients_jobs`, alates `GET /api/camera/cli-jobs`, millest igaühel on `started_by`), ja `stop_job` peatab selle alles `force: true` abil. Iga muu töö (salvestusvoor, massiivi salvestus või purske, taustasalvestus) töötab selles `chloros-mcp`-is, mis selle käivitas, ja ainult see server teab seda: kui teine server seda küsib, siis nii öeldakse. Peatatud töö loeb `stopping`-i, kui ta kirjutab oma järjekorda välja, seejärel `finished`-i koos lõplike loenditega, 10 minutit ja mis tahes `chloros-mcp`-ist: see, mis selle käivitas, näeb teise sunnitud peatamist samamoodi. `get_job_status` ei oota kunagi peatamist: peatamise ajal (omaenda `stop_job`, teise kliendivõi peatumist, mis vabastab ise lõppenud töö kaamerad), vastab ta kohe `stopping`-iga ja reaalajas loenditega, ning juba peatuva töö `stop_job` teatab sellest. Lõppenud töö selgitab põhjust `stop_reason`: oma põhjus (`no_frames`, `disk_full`, `member_missing`, või massiivi pideva käivituse puhul `count`, `duration`, `array_gone`), `limit reached` ainult siis, kui selle loendid näitavad selle `count` (või seeria `max_frames`, salvestuse`duration_s`) on saavutatud, `error`, `stopped` (koos `stopped_by`-iga), kui klient selle peatas, muidu `finished`.
- **Kuhu salvestused lähevad.** Avatud projekti kausta `captures`, kuhu rakendus need salvestab, kui ei ole määratud `output_dir`.
- **Pildid.** `get_camera_preview` ja `grab_frames` tagastavad pildid, mida abiline saab vaadata, lisaks nende JSON-andmetele. `grab_frames` piltide kontrasti on vaatamiseks suurendatud; nende statistilised andmed on tegelikud väärtused.
- **Kui server sulgub**, peatatakse selle poolt käivitatud ülesanded ja suletakse selle poolt käivitatud valgussensori salvestus. Kui server käivitas tagapõhja, vabastab ta enne tagapõhja peatamist ka seadmed, millega ta oli ühenduse loonud; rakenduse tagapõhjas jäävad need seadmed ühendatuks.
- **Protokoll.** Ainult stdio (HTTP-lõpppunkti pole), ainult tööriistad (ressursse ega küsitlusi pole), teksti- ja pilditulemused, MCP-protokolli versioonid 2024-11-05 kuni 2025-11-25.**Mida pole kaasatud.** Sisse- või väljalogimine (konto kuulub teile; server ei saa sisse logida) ning sellise backend&#x27;i peatamine, mida server ei käivitanud. Kõik muu, mida see SDK teeb seoses töötlemise ja riistvaraga, on hõlmatud tööriistadega. (DAQ-E seadistamine – selle nimi, autentimistoken, OTA-parool, allkirjanõue, tehase seadete taastamine – ning otsesed DAQ-serverid on ainult `chloros-cli`-i omad, ka SDK-s: vale nimi või token lukustab iga Chlorosi installi seadmest välja.)

---

## Tagapõhja automaatne käivitamine

Smart-Connecti sisenemispunktid – `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` ja `discover_daq_sensors` — on kerged HTTP-kliendid, mis eeldavad, et tagapõhi kuulab aadressil `127.0.0.1:5000` (smart-connect-liidese vaikimisi URL). Kui graafiline kasutajaliides või CLI juba töötab, siis on see juba käivitatud. Lihtsa skripti puhul võib see puududa — seega **käivitavad need funktsioonid enne esimest kutsumist automaatselt kaasasoleva tagapõhja binaarfaili** (aknata, samamoodi nagu teeb seda `ChlorosLocal`) ja ootavad seejärel kuni 60 sekundit, kuni see käivitub. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` ja `analyze_array_network()` ei käivita seda kunagi.

Reeglid:

- **Käivitatakse ainult kohalik URL.** `backend_url`, mis viitab `localhost`-ile / `127.0.0.1` / `[::1]` suunatud, on lubatud; mis tahes muud hosti käsitletakse kellegi teise masinana ja seda ei käivitatagi kunagi.
- **Tagapõhi, mis lõpetab töö enne vastamist, katkestab ootamise.** Kõne lõpetatakse kohe väljumiskoodi, käivitatud käsu ja tagapõhi poolt väljastatud viimaste ridade edastamisega, selle asemel et oodata 60 sekundit. Väljumiskood 98 on erand: teine tagapõhi hoiab juba porti või lukufaili (Chloros-rakenduse, tagapõhiteenuse või mõne teise skripti oma, mis on veel käivitumas), mistõttu kutsumine jätkab selle vastuse ootamist. `ChlorosLocal` toimib samamoodi.
- **Tagapõhi jäetakse töötama taaskasutamiseks** (nagu ka CLI-s) — skripti lõpetamisel ei toimu vaikimisi sulgemist. Skripti uuesti käivitamisel kasutatakse uuesti sama tagapõhja.
- **Võite sellest loobuda, kasutades `auto_start_backend=False`** mis tahes nimetatud kutsete puhul (nt kui olete suunanud kaugbackendile või haldate backendi elutsüklit ise).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Kui kaasasolevat binaarfaili ei õnnestu leida ega käivitada, tekitab järgnev HTTP-kõne tegevusele kutsuva, **platvormist sõltuva** `ChlorosConnectError` vea, mitte lihtsalt ühenduse keeldumise jälje — Windowsis suunab see teid töölaua rakenduse või `chloros-cli`-käsu juurde; Linuxis (ilma graafilise kasutajaliideseta) suunab see teid `chloros-cli`-käsu või `.deb` juurde.

---

## Keskkond ja päised

SDK märgistab iga tagapõhja HTTP-kõne koodiga `X-Chloros-Client: sdk`. Tagapõhi rakendab SDK/CLI litsentsieeskirju (vajalik on sisselogimine **ja** tasuline Chloros+ pakett), mitte graafilise kasutajaliidese tasuta versiooni reegleid. See seadistatakse automaatselt importimise ajal – te ei pea midagi tegema.

Pealkiri lisatakse igale `requests` / `urllib`-kutsele, mille URL-aadress sisaldab `localhost` või `127.0.0.1`, mis tahes pordis – sealhulgas teie enda kohalikes teenustes — ning asendab väärtuse, mille olete ise määranud. Teistele hostidele suunatud kutsed jäävad muutmata.

Tagapõhja URL-i saab ümber kirjutada, edastades `backend_url=` (`ChlorosLocal` võtab vastu `backend_url=` või `api_url=`; `process_folder()` ja `process_lattice_capture()` võtavad vastu ainult `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Paigaldatud tagapõhi kuulab ainult loopbacki kaudu, seega LAN-võrgus asuv `backend_url` ei pääse sellele ligi; vaata [Kaugtagapõhi režiim](#remote-backend-mode-pip-only-host-via-tunnel))

---

## Versioonid ja ühilduvus

- SDK-versioon on esitatud kui `chloros_sdk.__version__`.
- SDK seob käitumise kaasasoleva backend-versiooniga. Vanema SDK ja uuema backend-i kombineerimine toimib tavaliselt (edasiühilduvad lõpppunktid), kuid uuema SDK ja vanema backend-i kombineerimine võib uutes lõpppunktides põhjustada `404` vigu — uuendage töölaua rakendus vastavalt.
- Smart-Connecti liides (`connect_camera` / `connect_array` / `connect_daq_sensor`) ja võrguanalüüsi lõpppunkt tagastavad stabiilseid JSON-skeeme; uued väljad on täiendavad.

---

## Veaotsingunõuanded

- **`ChlorosAuthenticationError: Login required`** (pärineb veast `ChlorosLocal`; smart-connect-kutsed tekitavad samal põhjusel vea `ChlorosConnectError: Login required. …`) → Käivita sellel arvutil üks kord `chloros-cli login EMAIL PASSWORD` või logi sisse Chlorosi töölaua rakenduse kaudu.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (või `Failed to create project: …`, mis tuleneb veast `create_project()`) → Windowsi pikad teed on sellel arvutil välja lülitatud (Windowsi vaikimisi seade), seega ei tohi ükski tee ületada 260 tähemärki ning tooted asuvad kuni umbes 130 tähemärki allpool projektikausta. Midagi ei töödeldud. Teade näitab pikimat sobivat projektikausta: kasuta lühemat või lülita sisse Windowsi pikad teed ja taaskäivita Chloros. Windowsi paigaldaja pakub seda valikuna „Luba Windowsi pikad teed”, mis on vaikimisi märgitud; muidu kasuta grupipoliitikat: *Arvuti konfiguratsioon &gt; Haldusmallid &gt; Süsteem &gt; Failisüsteem &gt; Luba Win32 pikad teed*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-Connect käivitab automaatselt kohaliku tagapõhja, seega ilmub see veateade ainult siis, kui kaasasolevat binaarfaili ei leita või seda ei saa käivitada (nt pip-ainult host, millel puudub töölaua pakett). Teade sõltub platvormist: Windowsis avage töölaua rakendus või käivitage mis tahes `chloros-cli`-käsk; Linuxis käivita käsk `chloros-cli` (graafilist kasutajaliidest ei ole) või paigalda `.deb`. Kaugbackendi puhul edasta `backend_url=` (ja `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** importimisel → `lattice_sdk` laadimine ebaõnnestus: Arena SDK-i käitamiskeskkond või OpenCV (`cv2`) ei ole selles Pythonis importitav. Tagapõhjal põhinev pind (`connect_*`, projitseerib tagapõhja kaudu, `ChlorosLocal`) töötab endiselt.
- **Array ühendub täisanduriga, kuid madala kaadrisagedusega**→ Nii on see kavandatud: automaatrežiim säilitab täisanduri ja laseb kaadrisagedusel langeda võrgu kandevõime piires. Kasutage `binning=2` / `4` või väiksema `width` / `height`, et saada rohkem kaadreid sekundis, või lülita sisse jumbo-kaadrid / uuenda ühendust. Binning ja ROI**ei** kata koondatud ülemärkimist (`oversubscribed: true`, fps-väljad 0): liiga palju kaameraid ühe ühenduse jaoks ei ole võimalik lahendada binningu/ROI abil — vähendage kaamerate arvu, lubage jumbo-kaadrid või vahetage kiirema võrgukaardi vastu (vt [Ülekoormus](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` teatab, et võrgukaardi vastuvõturing on väga väike (~0,26 MB) / ühendusväravad näitavad „FRAMES WILL DROP”** → Host-võrgukaardi vastuvõturing on vaikimisi seadistuses (sageli taastatakse väärtuseks 32 pärast võrgukaardi draiveri uuendamist). Realteki USB 10GbE-adapteri puhul seadistage `ReceiveBufferLen=256` ja `PendingReceives=64` (kõrgendatud), seejärel taaskäivitage backend, et see loeks ringi uuesti sisse. Täielik juhend: [CLI-viide → Host-võrgukaardi seadistamine ja häälestamine](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host hangub taaskäivitamisel/väljalülitamisel, hiljem ilmnevad WMI-vead `Invalid class` / võrgukaart ei aktiveeru** → Vananenud USB 10GbE draiver põhjustab veat koodi `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Uuendage adapteri draiver viimase versioonini (≥ 2026) ja rakendage vastuvõturingi seaded uuesti. Vaata [CLI-viide → Host-võrgukaardi seadistamine ja häälestamine](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Massiivi salvestamine ebaõnnestub HTTP-veaga 503 „No fresh synced frame group”** → Massiiv ei avaldanud ooteaja jooksul ühtegi uut raamigruppi. Kontrollige `arr.status()`: `stalled: True` (või `streaming: False`) koos `actual_fps`-iga, kus 0 tähendab, et salvestamistsükkel on lõpetanud avaldamise — katkesta ühendus massiiviga ja ühenda uuesti. Kui `stalled` on False ja `actual_fps` on suurem kui 0, proovi uuesti. Vahetult pärast `connect_array()` on salvestus juba oodanud esimest rühma kuni 20 sekundit.
- **`status()` näitab `link_speed_mbps: 4294`** → Windowsil ei õnnestunud lugeda adapteri kiirust; see on tüüpiline USB-Ethernet-adapterite puhul ega ole viga. Vaata [Oodatavad kaadrisagedused](#expected-frame-rates).
- **`update_firmware()` ebaõnnestub koos `device never connected back on TCP <port>`-iga** → Tulemüür blokeeris DAQ-E ühenduse tagasi selle arvutiga. Windowsi paigaldaja lisab reegli (vt [DAQ-E seadme seaded](#daq-e-device-settings)); teiste tulemüüride puhul lubage sissetulevat TCP-liiklust tagapõhja andurite alamvõrgust.
- **Peegeldusvõime keelatud** → Absoluutse skaalaga peegeldusvõime saamiseks peab kaamera (või kaamerate massiiv) olema seotud reaalajas töötava DAQ-iga. Seostage kas graafilise kasutajaliidese kaudu või kasutage `processing="radiance"`-i (W/m²/sr/nm), mis ei vaja paaris olevat andurit.
- **`smart=True` andmete kogumine võtab oodatust kauem aega** → AE konvergents sõltub stseeni dünaamikast. Kasutage `ArrayHandle.capture_smart()`, laiendage `exposure_tolerance_pct`, lühendage `stability_window_s` või vähendage `settle_timeout_s` väärtust, et saavutada kiirem ja vähem stabiilne käivitamine.

---

## Vaata ka

- [CLI-viide](cli-reference.md) — iga CLI-alakäsk vastab ühele SDK-kutsele.
- [DAQ-valgusandurid](https://mapir.gitbook.io/chloros/daq-light-sensors) Chloros-kasutusjuhendis — andurid, nende piirangud ja kalibreeritud mõõtevahemik, salvestamine, DAQ-E võrgustik.
- [LATTICE-kaamerad](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) Chloros-kasutusjuhendis — ühendamine, kaamera seaded, pildistamisrežiimid, massiivid.
- Veebidokumendid: `https://mapir.gitbook.io/chloros/api-python-sdk`
