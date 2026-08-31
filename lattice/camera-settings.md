# Kaamera seaded

Vahekaart **Kaamerad**on Chloros-i LATTICE-kaamerate reaalajas juhtpaneel: peamine pildivaateala, kus iga ühendatud kaamera kuvatakse reaalajas ruuduna, ning külgriba, mida saab liigutada kolme lehe vahel –**kaamerate nimekiri**,**seadete paneel**(kaamera-, massiivi- või salvestusseaded – üks korraga) ja**indeksikalkulaator**. Sellel lehel on kirjeldatud kõik kaamerate loendi, kaamerapõhiste seadete paneeli ja massiivi seadete paneeli juhtelemendid. Salvestusrežiimid, eksporditüübi valik ja „Salvesta kõik“ voog asuvad kaaslehel [Salvestusseaded ja režiimid](capture.md).

Vahekaart „Kaamerad“ ilmub külgribale, kui Chloros-tagapõhi on valmis. Kõik allpool loetletud juhtelemendid suhtlevad kohaliku tagapõhjaga `127.0.0.1:5000` kaudu; muudatused rakenduvad reaalajas kaamerale kohe, kui ei ole märgitud teisiti.

## Sellel lehel kasutatavad kaameratüübid

Juhtelemendid kuvatakse või peidetakse sõltuvalt sellest, milline kaameratüüp on valitud. Käsiraamatus kasutatakse järgmisi termineid:

| Termin | Tähendus | Filtrikanalid |
| --- | --- | --- |
| **RGB kaamera** | LATTICE M3C koos FRGB-filtriga (mudel sisaldab `-FRGB`) | Red / Green / Blue |
| **Bayer multispektraalne** | LATTICE M3C koos FRGN, FOCN või FNGB-ga | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — üks kitsasriba filter, üks kalibreeritud sagedusala | Üks sagedusala |
| **Massiivi liige** | Kaamera, mis on ühendatud sünkroniseeritud massiivi osana (ühendatud või eraldi ekraan) | Vastavalt oma filtrile |

RGB kaamerad läbivad fotomeetrilise töötluse (valgebalanss, värviprofiilid, gamma); multispektraalsed ja monokaamerad läbivad radiomeetrilise ahela ning fotomeetrilised seadistused jäetakse vahele. Massiivi liikmed edastavad voo tasandi seadistused (piksli formaat, resolutsioon, binning, triggereerimine, kaadrisagedus) massiivile — need read muutuvad kaamera-paneelis ainult lugemiseks ja liiguvad massiivi seadistuste paneeli.

## Peamine vooala

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

Kui ühtegi kaamerat pole ühendatud, kuvatakse vooalas teade **„Ühenda kaamera, et alustada“**koos kahe nupuga:**Ühenda kaamera**(roheline, avab ühe kaamera ühendamise dialoogakna) ja**Ühenda massiiv** (sinine, avab massiivi ühendamise dialoogakna). Ühendamise dialoogaknad on dokumenteeritud jaotises [Kaamerate ühendamine](connecting.md); kaamerarühmade põhimõtted (sünkroniseerimine, tasemed, ribalaius) on kirjeldatud [Mitme kaameraga rühmad](arrays.md). Kui avate salvestatud projekti, milles on kaamerad, kuvab tervitusekraan selle asemel pöörlevat ikooni koos tekstiga „Avatakse uuesti N salvestatud kaamerat…”, samal ajal kui Chloros taastab viimase seansi vooge.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Ülemine riba

| Juhtnupp | Mida see teeb |
| --- | --- |
| **Vaate režiimi lüliti**| Lülitab**võrgustikuvaate**(kõik ruudud on lahtritena) ja**nimekirjavaate** (ülemises osas täislaiusega kaamerarühmad, all üks aktiivne kaamera) vahel. Tööriistavihjed: „Lülita võrgustikuvaatesse” / „Lülita nimekirjavaatesse”. |
| **Ristiku lukustus**(riiv) | Vaikimisi**lukustatud** — ruudud on paigal külmutatud. Lukustuse vabastamisel saab ruute lohistada ja ümber paigutada mis tahes kohale (vahed säilivad). Ristik lukustub automaatselt uuesti iga kord, kui uus kaamera ühendub. Vihjed: „Vabasta ristik (võimaldab ruutude lohistamist)“ / „Lukusta ruudustik (kinnita plaadid paigale)”. |
| **Voogu suurendamise** liugur | Plaadi suurus, alates 60 pikselist kuni kogu konteineri laiuseni. Lahtrid säilitavad 4:3 kuvasuhte. Kui lahtri laius on alla 200 pikseli, peidetakse nimi ja kaadrisageduse (fps) ülekatted, et plaat jääks puhtaks. |

### Voogu ruudud

Iga kaamera renderdab komposiit-reaalajas ruudu; kaamera võib lisaks näidata kolme halltoonides **kanalipõhiseid jagatud** ruutu (vt [Kanalite jagamised](#display-overlays-drawn-over-the-live-feed)), ning massiivid renderdavad kombineeritud ruutu. Aktiivsel ruudul on valikuring kaamera (või massiivi) värvitoonis.

Ruudu peale hiirega liikudes ilmub **X** sulgemisnupp:

* Kui sulgeda **koond**-ruut, kui selle kanalite jagatud vaated on endiselt nähtavad, peidetakse vaid koondvaade.
* Kui sulgeda **eraldi kaamera viimane nähtav ruut**, katkestatakse selle kaameraga ühendus.
* **Kombineeritud kaameramassiivi liikmete jagatud ruudud ei katkesta kunagi** kaameraga ühendust — need peidavad ruudu vaid ära.

Kui võrgustik on lukustamata, lohista mis tahes ruutu mis tahes pesasse; paigutus salvestatakse koos projektiga.

## Külgriba — kaamerate nimekiri

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Külgriba esimesel lehel on loetletud kõik ühendatud kaamerad ja massiivid:

* **Ühenda kaamera**(roheline) /**Ühenda massiiv** (sinine, kuvab skannimise ajal „Tuvastatakse...“). Mõlemad on keelatud, kui ühendamise dialoog on avatud.
* **Kõik salvestada** (punane) — salvestab kõik loetletud kaamerad koos salvestusseadetes valitud eksporditüüpidega. Nõuab avatud projekti. Täielik kirjeldus on [Salvestusseaded ja režiimid](capture.md).
* **Salvestusseadete hammasratas** (kõrval nupule „Salvesta kõik”) — avab [salvestusseadete paneeli](capture.md#the-capture-settings-pane). On keelatud, kui projekti pole või salvestamise ajal.

### Kaameraridad

Igal kaameraribal on värvikoodiga piirjoon (kaamera kohandatud värv), märge „CAM“ — koos sinise **M**(master) või rohelise**S** (slave) rollitähte massiivi liikmetele — ning kuvatava nime. Vaikimisi nimi on `LATTICE-MODEL (serial)`; seda saab ümber nimetada kaamera seadete paneelis. Rea nupud:

| Nupp | Toime |
| --- | --- |
| **Silm**| Nähtavuse lülitamine. Peidetud kaamerad eemaldatakse ruudustikust ja**jäetakse välja funktsioonist „Capture All“**. |
| **Hammasratas** | Avab kaamera-spetsiifilise seadete paneeli (järgmine jaotis). |
| **Paus / Esita**| Jääta reaalajas eelvaade**ainult ekraanil** seisma — tagapõhja salvestamine jätkub. Peatatud kaamerad ei saa salvestada. |
| **X** | Ühendus katkestada. Kasutajaliides uueneb kohe (optimaalsel juhul); tagapõhja ühenduse katkestamine võib kesta 10–30 sekundit. |

### Massiivi read

Massiivi real kuvatakse massiivi värviga märge „ARRAY”, massiivi nimi (mida saab massiivi seadetes ümber nimetada) ja **DAQ · sisse/välja**nupp —**sisse**, kui massiivi tasandi valgussensor on seadistatud *või* mõnel liikmel on kaamera-põhine sensor; selle abiteates on täpselt loetletud, milline andur millist teavet edastab. Massiivi liikmekaamerad on loetletud allpool sisselõigatud ridadena omaette. Massiivi rea nupud: **silm**(peidab/näitab KÕIKI liikmeid korraga),**hammasratas**(massiivi seadete paneel),**X**(kogu massiivi ühenduse katkestamine).

Massiivi ridades ja massiivi seadete paneelis kasutataval valgussensoril (DLS) on neli olekut:**väljas**,**ootab**(spektrit pole veel),**aktiivne**(spekter saabus viimase 3 sekundi jooksul) ja**vananenud** — 3 sekundi jooksul pole uut spektrit, kuid viimast näitu *kasutatakse endiselt* (DAQ-näidud ei aegu kunagi salvestusteel).

Saate loendi järjestust muuta, lohistades külgribal eraldiseisvaid kaameraid ja terveid massiivi rühmi üksteise kõrvale; massiivi liikmeid ei saa eraldi lohistada.

## Kaamera-spetsiifiline seadete paneel

Avage see kaamera rea **hammasratta** abil. Paneel libiseb kaamerate loendi peale.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Pealkiri**: kaamera**värviproov**(klõpsake, et avada sisseehitatud värvivalija – määrab külgriba ääre ja ruudu valikuringi värvi),**nimi**koos pliiatsikujulise**Ümbernimetamine**(tühja nime salvestamisel taastub vaikimisi `MODEL (serial)`) ning**×** sulgemiseks.

### Reaalajas histogramm

Paneeli ülaosas asub reaalajas luma histogramm, mis arvutatakse JPEG eelvaate põhjal sagedusega ~8 Hz. Keskmine on Bayer-kaalutud — (R+2G+B)/4 — et vastata kaamera enda AE mõõtmisele.

* **Orange katkendjoon**= AE sihtmärk.**Lohistage seda horisontaalselt, et sihtmärki muuta** – vabastamisel saadetakse üks käsk ning lohistamine lülitab AE sihtmärgi režiimi käsitsi režiimile.
* **Green pidev joon** = tegelik keskmine heledus (mida AE hetkel annab).
* **RGB nupp** (ülemine parem nurk): lülitab sisse kaamera filtri järgi värvitud sagedusribade kaupa kuvatavad histogrammid (nt FRGN-režiimis: hall NIR, roheline, punane). Mono (M3M) kaameratel on nupul kiri „MONO“ ja see on keelatud — mono režiimis kuvatakse alati ühe sagedusriba heleduse histogrammi.
* X-telje märgistused järgivad praeguse pikseliformaadi anduri bittisügavust: 0..255, 0..1023, 0..4095 või 0..65535.

### Kaamera teabe read

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

| Rida | Toimimine |
| --- | --- |
| **Mudel** | Ainult lugemiseks (nt `LATT-M3C-L87-FRGN`). |
| **Radiomeetriline kalibreerimine**| Green**&quot;Aktiivne&quot;**märgis, mille allkirjas on näidatud kalibreerimistasand, hash, kalibreerimise kuupäev ja sagedusribade loend, mis on laaditud kaamera kalibreerimispaketist (vt [Tehase radiomeetriline kalibreerimine](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Peidetud kaamerate puhul RGB** — neil on fotomeetriline valgebalansi kalibreerimine, mitte sagedusribade kaupa kiirgustugevus. |
| **Kalibreerimisaruanne**|**Lae alla** nupp — avab kaamera seerianumbri järgi koostatud NIST-kalibreerimissertifikaadi PDF-faili teie operatsioonisüsteemi vaatajas. Kui sertifikaati pole veel vahemällu salvestatud, kuvab Chloros selle asemel vihje. |
| **Seerianumber** | Ainult lugemiseks. |
| **Püsivara**| Näitab praegust versiooni, seejärel otsib selle mudeli jaoks saadaval oleva versiooni (vahemällu salvestatud mudeli kaupa — N-kaamerast koosnev massiiv kontrollib serverit üks kord). Seisundid: „Kontrollitakse…“ → nupp**„Uuenda versioonile X“**→ „Värskendatakse…“ → „Uuendatud A → B“ / „Ebaõnnestus: …“ / „Vahelet jäetud: …“ / roheline**„Ajakohane“**. Uuendamisnupu vihje: „Tehase seaded + uuendamine + UserSet1 ümberprogrammeerimine. ~2–3 minutit; ärge katkestage ühendust.“ |
| **IP** | Ainult lugemiseks. |
| **Temperatuur** | Ainult lugemiseks, värskendatakse iga 3 sekundi järel. Muutub oranžiks temperatuuril ≥65 °C ja punaseks koos märgiga ⚠ temperatuuril ≥75 °C. |
| **Kalibreerimise sihtmärk** valikukast | Lülitab sisse ArUco peegeldusmäära sihtmärgi tuvastamise koos paneelipõhise NDVI valideerimistabeliga reaalaja pildi all (loendivaade). Ainult seansiks — avatakse alati välja lülitatuna. |
| **Valgusandur** rippmenüü | Seob DAQ-valgusanduri (DAQ-E/M/U, valikust „Valgusandurid“) selle kaameraga allapoole suunatud valguse (DLS) valgustuse korrigeerimiseks ja ennustava automaatse särituse jaoks. Valik „Puudub“ tühistab seose. Kui andureid pole ühendatud, kuvatakse rippmenüüs „(andureid pole ühendatud – ava DAQ-vahekaart)“. Seos salvestatakse koos projektiga. |

### Ekspositsioon ja võimendus

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Kõik siinsed numbrilised sisestusväljad kasutavad hoides kiirendatavaid pöörikuid: puudutamine = ±1, hoidmine &gt;1,5 s = ±10, hoidmine &gt;3 s = ±100. Väärtus saadetakse kaamerale, kui sõrme lahti lasete.

| Seade | Vahemik / valikud | Vaikimisi | Kehtib | Mida teeb |
| --- | --- | --- | --- | --- |
| **Säriaeg (us)**| Kaamera reaalajas min/max | Auto | Kõik | Säriaeg mikrosekundites, koos**Auto/Käsitsi** lülitiga. Auto = pidev kaamerapoolne automaatne säriaeg. |
| **Võimendus (dB)**| Kaamera reaalajas miinimum/maksimum (nt kuni 48 dB) | Käsitsi (väljas) | Kõik | Analoog-/digitaalne võimendus koos eraldi**Auto/Käsitsi** lülitiga. |
| **AE sihtheledus**| 0–255 | 80, režiim**Auto**| Kõik (muudetav, kui AE või automaatne võimendus on sisse lülitatud) | Heledus, millele AE püüab saavutada. Režiimis**Auto**(vaikimisi) valib histogrammipõhine taustkontroller sihtmärgi ise, hoides ekspositsiooni 60–75 %-l anduri maksimumist. Väärtuse sisestamine või histogrammi oranži joone lohistamine lülitab selle režiimile**Käsitsi**. |
| **AE silumine** | 0,5–40, samm 0,1 | 8,0 | Kõik | AE summutamine. Vihje: „Madalam = AE reageerib kiiremini (võib kõrge kaadrisageduse juures pulseerida). Kõrgem = sujuvam / aeglasem.” Vaikimisi väärtusest oluliselt madalamad väärtused võivad põhjustada AE pulseerimist ja voogedastuse ebastabiilsust kõrgel kaadrisagedusel; 8,0 on stabiilne vaikimisi väärtus. |
| **AE huvipiirkond**| Lülita sisse valikukast +**Sihtimine**nupp | Väljas | Kõik | Kui see on sisse lülitatud, mõõdab AE ainult rohelise katkendjoonega piiratud piirkonda, mitte kogu kaadrit.**Sihtimine** aktiveerib klõpsuga paigutamise reaalajas voos: klõps keskendab piirkonna kaadri 30 %-le; klõps-ja-tõmba joonistab kohandatud ristküliku (minimaalselt 5 % × 5 %). „Aim“ lülitub pärast ühte paigutust ise välja. Piirkond kaardistatakse tagasi kaamera algsetele koordinaatidele vastavalt sinu seatud pööramisele/peegeldamisele ning salvestatakse koos projektiga. |
| **AE Tune Speed** | 0,1–5, samm 0,1 | 1,0 | Ainult Array liikmetele | Kui kiiresti automaatne AE-sihtmärk jälgib stseeni heleduse muutusi; 1,0× kontrollib uuesti iga 2,5 sekundi järel. |
| **Heledate alade kaitse** | Range (1 %) / Normaalne (5 %) / Leebe (15 %) | Range | Kaamerad, mis toetavad seda seadet | Kui suur osa kaadrist võib muutuda valgeks, enne kui AE pilti tumedamaks muudab. |

{% hint style="info" %}
**Valgustuse nõuded Bayeri multispektraalsetele kaameratele (RGN / OCN / NGB):** stseenis peab olema piisavalt valgust kõigis kolmes kanalit, vastasel juhul kalibreerimine ei tööta korrektselt — üksainus anduri säritus hõlmab kõiki kolme spektrit. Kasutage DAQ valgussensorit valguse mõõtmiseks või valige täielikult monokroomne režiim (M3M), et iga kanal saaks oma eraldi särituse. Kui pildistamine seda nõuet rikub, tuvastab Chloros selle ja hoiatab teid (unmix-clamp-teade).
{% endhint %}

### Pikselivorming ja eraldusvõime**Massiivi

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

liikmed** kuvavad ainult lugemiseks mõeldud „Current“ (vorming + WxH) ja „Binning” rea koos märkusega „Määratud massiivi seadetes” — voo taaskäivitamine ühel elemendil katkestaks sünkroniseeringu, seega hallatakse neid [massiivi seadete paneelis](#array-settings-pane).**Eraldi kaamerad** saavad:

| Juhtelement | Valikud | Mida see teeb |
| --- | --- | --- |
| **Piksli formaat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Anduri pikseliformaat (bitisügavus). |
| **Eraldusvõime** | Täis / Pool / Veerand | Suhteline praeguse binninguga: Täis = 2048/N × 1536/N N×N binningu puhul. |
| **Pikslite ühendamine** | 1x1 (puudub) / 2x2 / 4x4 | Riistvaraline N×N pikslite ühendamine — suuremad väärtused vähendavad eraldusvõimet, kuid suurendavad signaali-müra suhet (SNR) ja kaadrisagedust. Selle muutmine taaskäivitab voo ja seadistab kõik huvipiirkonnad (ROI) uuele täisvaateväljale. |
| **Praegune** | ainult lugemiseks | Kehtiv tegelik laius x kõrgus ja (x, y) nihke. |

### Reaalajas eelvaade

Kõik selles jaotises on **ainult kuvapoolne**— see muudab seda, mida näed reaalajas voos, samas kui salvestatud kaadrid jäävad lineaarseteks ja muutmata — ühe erandiga:**Vignette** on radiomeetriline ja mõjutab ka eksportimist (selgitatud allpool).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Regulaator | Vahemik / valikud | Vaikimisi | Kehtib | Mida teeb |
| --- | --- | --- | --- | --- |
| **Renderdamise resolutsioon** | 360p (kiireim) / 480p / 720p / 1080p / Sensori algne resolutsioon (aeglaseim) | 720p | Kõik | Kõrgus, millel tagapõhi käitab radiomeetrilist eelvaate ahelat. Madalam väärtus tagab suurema kaadrisageduse, muutmata vaatevälja. |
| **Indeks**| Valikukast „Luba“ + hammasratas | Väljas | Ainult Bayeri multispektraalne,**mitte** kombineeritud maatriksi liikmed | Reaalajas taimestikuindeksi eelvaade. Hammasratas avab jagatud [Indeksiarvutaja](#index-calculator-pane), millesse on eelnevalt laaditud kaamera loomulikud filtriribad (nt `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Iga eelvaatekaadri puhul arvutatakse kohandatud avaldis koos LUT-iga (sisse/välja, tase vaikimisi 3, min vaikimisi 0,2, maks vaikimisi 1). Kombineeritud massiivi liikmed peidavad selle rea — massiivil on üks jagatud indeks. |
| **Valgebalanss** | Väljas / Üks kord / Pidev + uuesti salvestamise nupp | Pidev | Ainult RGB | Reaalajas valgebalanss. Värskendamisnupp salvestab valgebalansi uuesti praegusest DLS-spektrist (keelatud, kui režiim on Väljas). |
| **Gammakorrektsioon** | Sisse / Väljas | Sisse | Ainult RGB | Gammakorrektsiooni (γ = 2,2 LUT) kuvamine reaalajas eelvaates. Salvestatud pildid jäävad lineaarseteks. |
| **Müra vähendamine** | Valikukast + tugevus 0–100 | Väljas / 50 | Kõik (iga kaamera kohta, isegi massiivides) | Kahepoolne filter reaalajas eelvaates. Kõrgem väärtus = siledam, kuid pehmemad detailid. |
| **Teravus** | Valikukast + tugevus 0–100 | Väljas / 30 | Kõik | Teravustamismaski reaalajas eelvaates, rakendatakse viimasena. Võib müra võimendada. Ainult eelvaates. |
| **Vignett**| Valikukast + tugevus 0–100 | Väljas / 0 | Kõik | Käsitsi jääkvignett-vignett (heledab nurki), mis on paigutatud massiivi „Smart Vignette“ hinnangu peale.**Radiomeetriline — mõjutab reaalaja vaadet JA eksporti**, erinevalt müra vähendamisest ja teravusest. |
| **Värviprofiil** | Raw / Lineaarne / Looduslik / Täiustatud / Kohandatud temperatuur | Looduslik | Ainult RGB | Vaata allpool. |
| **Värvitemperatuur** | 2000–10000 K, sammuga 100 | 5500 K | Ainult RGB, kohandatud temperatuuri profiil | Kinnitab valgebalansi kindlale korrelatsioonilisele värvitemperatuurile (DLS-sisendit ei arvestata). Viimati valitud Kelvini väärtus jääb meelde ka profiili vahetamisel. |
| **Küllastus** | 0–200 (100 = neutraalne) | 100 | Ainult RGB | HSV-küllastus reaalajas eelvaates. |
| **Kontrast** | 0–200 (100 = neutraalne) | 100 | Ainult RGB | Lineaarne kontrast reaalajas eelvaates keskmise halli ümbruses. |
| **Peegelda horisontaalselt / Peegelda vertikaalselt** | Valikukastid | Väljas | Kõik | Pöörab eelvaadet horisontaalselt / vertikaalselt. |
| **Pööramine**| 0° / 90° / 180° / 270° | 0° | Kõik | Pöörab eelvaadet. Suund rakendatakse eelvaate töötlemisahela lõpus —**salvestatud pildid jäävad kaamera algse suuna järgi** ning massiivi komposiitvaated seda ei arvesta. |**Värviprofiili semantik** (RGB kaamerad):

* **Raw** — jätab töötlemisahela täielikult vahele.
* **Lineaarne** — tume signaal + tasandatud väli + valgebalanss; ilma värvimatriitsita, ilma gammata.
* **Looduslik** *(vaikimisi)* — lineaarne pluss mõõdetud värvikorrektsioonimatriits ja stseeni järgi kohanduv toonikõver.
* **Täiustatud**— „Looduslik” pluss värviküllus ja CLAHE kohalik kontrast. Lisatasu kehtib**ainult reaalajas eelvaate puhul** — salvestatud pildid saavad alati täieliku viimistluse, sõltumata profiilist.
* **Kohandatud temperatuur** — „Natural”, kus valgebalanss on fikseeritud teie valitud Kelvini väärtusele.

{% hint style="warning" %}
Režiimide „Looduslik”, „Täiustatud” ja „Kohandatud temperatuur” puhul kuvatakse paneelis toonimärkus: kaadrid heledamaks muudetakse vastavalt nende enda stseenile, mistõttu salvestatud *ekraanil kuvatavaid* pilte ei saa kaadrihaaval võrrelda. **Eksportige mõõtmiseks kiirgust või peegeldust.**
{% endhint %}

### Ekraanil kuvatavad ülekattepildid (joonistatakse reaalajas voo peale)

Need on ainult kasutajaliidese jaoks — joonistatakse video peale, puudutamata kunagi voogu ega salvestusi.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Ülekattepilt | Juhtnupud | Vaikimisi | Mida see teeb |
| --- | --- | --- | --- |
| **Zebra** | Valikukast + künnis 200–255 | Väljas / 250 | Magenta diagonaalsed triibud kärbitud pikslitel. |
| **Sihtjoon** | Valikukast | Väljas | Kaadri keskpunkti märk. |
| **Võrgustik** | Väljas / 3 × 3 / 9 × 9 | Väljas | Kompositsioonivõrgustik. |
| **Histogramm** | Valikukast + laius 0,10–0,90 kaadri laiusest | Väljas / 0,25 | Otseülekande histogrammiriba. |
| **Fookuse tipp** | Valikukast + künnis 20–200 + värvivalik | Väljas / 80 / `#ff5722` | Sobeli servade esiletõstmine fookustamiseks. |
| **Kanali jagamised** | &quot;Näita jagamisi (Red / Green / NIR)&quot; / Nupp „Peida jagamised“ | Peidetud | Lisab komposiidipildi kõrvale kolm sõltumatut halltoonilist kanalipõhist ruutu (nupu silt vastab kaamera filtri kanalitele). Iga jagatud ruutu saab lohistada ja see jagab kaamera raamivärvi. Ei ole saadaval monokroomkaameratel. Salvestatakse koos projektiga. |

### Punktmõõtur

* **Klõpsa proovi võtmiseks**valikukast: klõpsa reaalajas pildil, et võtta proov ühest pikslist (seda märgib ristjoon), või klõpsa ja lohista piirkonda, et saada pikslite keskmine.**Tühista**kustutab proovi ja ristjoone. See funktsioon ei ole ühildatav AE-ROI**Aim**-režiimiga.
* **Show**rippmenüü:**Raw (bit depth)**— andurite algupärased digitaalsed väärtused anduri bittisügavuses (nt 12-bit → 0..4095) — või**Display (8-bit)** (vaikimisi). Kui reaalajas indeks on aktiivne, näitab „Display“ selle asemel arvutatud indeksiväärtust (nt NDVI).
* Näidupaneelil on loetletud pikselikoordinaadid, kaadri suurus, pikseliformaat, bittisügavus ja kanalitabel (Chan / Value / %) koos ribamärgistuste ja lainepikkustega; Bayeri rohelised paarid on keskmistatud; piirkonna proovid näitavad „N px avg“.

Punktmõõturi olek kehtib ainult käesoleva seansi ajal.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Ennustav automaatne säritus (DLS-põhine)

See osa kuvatakse ainult siis, kui **on ühendatud vähemalt üks DAQ-valgusandur** — lahendaja vajab selle juhtimiseks reaalajas allapoole suunatud spektrit.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Juhtimine | Vahemik | Vaikimisi | Mida see teeb |
| --- | --- | --- | --- |
| **Luba** | Valikukast | Sisse (iseseisvad kaamerad) | Suletud vormiga lahendaja kasutab DLS-spektrit ja kaamera kalibreerimispaketi skalaare, et viia kõige heledam laine lähedale küllastumisele, hoides samal ajal kõige tumedama laine üle signaali-müra suhte (SNR) alampiiri — üks eksponeerimiskirje iga lahenduse kohta, ilma stabiliseerimistsüklita. Mõeldud päikeseenergiaga töötavate aeg-luup-kaamerate jaoks, kus iga kaader peab olema õigesti valgustatud. Tagapõhi lülitub märkamatult tagasi reaktiivsele automaatvalgustusele (AE), kui DLS-näidud on aegunud või puuduvad või kui kalibreerimispaketti pole laaditud. |
| **Sileerimine (α)** | 0,05–1,0, samm 0,05 | 0,3 | Järgnevate ennustavate lahenduste silumine (madalam väärtus = silumisem). |
| **Stseeni peegeldusvõime**|**Kalibreeri ρ uuesti** nupp | — | Hinnab uuesti stseeni peegeldusvõime tegurit, mida lahendaja kasutab. |

{% hint style="info" %}
**Massiivi ühendamine lülitab ennustava AE vaikimisi välja** — massiivide puhul haldab valgustust Chlorosi nutikas AE koos kaamerapoolse automaatse valgustuse funktsiooniga (koos küllastumiskaitsega) ning ennustava AE ühtne stseeni peegeldusvõime hinnang ei ole segastseenide puhul usaldusväärne. Saate selle siin kaamera kaupa uuesti sisse lülitada, kui soovite konkreetselt DLS-põhist radiomeetrilist ekspositsiooni.
{% endhint %}

**DAQ-põhine ekspositsiooni ülempiir ja langeva valguse järgi fikseeritud AE.**Sõltumata ülaltoodud valikukastist, kui RGB kaamerale on määratud DAQ-valgusandur, arvutab Chloros — mõõdetud absoluutse allapoole suunatud kiirguse alusel — maksimaalse ekspositsiooni × võimenduse, mille juures 100 % peegeldusega pind jääb allapoole klippimist, ning rakendab seda**ülempiirina**automaatse ekspositsiooni puhul. Kui ülempiir on aktiivne, töötab kaamera**langeva valguse järgi**: see töötab avatud ahelas langeva valguse järgi mõõdetud ekspositsiooniga, võimendusega 0 dB — ekspositsioon järgib mõõdetud valgust, mitte stseeni sisu. Kuna ülempiir saab ekspositsiooni ainult lühendada, ei saa see ise klippimist põhjustada. Ülempiir lülitub automaatselt välja — ja taastub tavaline stseeni automaatne ekspositsioon — kui DAQ-näit puudub, on aegunud (&gt;30 s), või tume, või kui ≥15 % kaadrist lõikub fikseeritud ekspositsiooni juures (mis tähendab, et andur ja kaamera näevad erinevat valgustust). Graafilist kasutajaliidese lülitit ei ole; see on standardne käitumine alati, kui RGB-kaameral on DAQ-side.

### Andmete kogumise ja triggereid

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

kasutavate massiivi liikmete puhul kuvatakse lisaks ainult lugemiseks mõeldud **Role**(Master sinisel / Slave rohelisel),**Sync Line**ja**Peers** read.

| Kontroll | Valikud | Vaikimisi | Märkused |
| --- | --- | --- | --- |
| **Trigger Mode** | Off / On | On | Massiivi liikmetele keelatud (massiiv haldab triggereid). |
| **Triggeriallikas** | Tarkvara / Line0 (M8) / Line1 / Line2 | Line0 | Peidetud, kui triggerirežiim on välja lülitatud; massiivi liikmetele keelatud. Line0 on M8 optiliselt isoleeritud väline triggerisisend. |
| **Kaadrisagedus**| Auto / Käsitsi + väärtus | Auto |**Auto**: kaamera kaadrisageduse piirang on välja lülitatud — kaadrisagedust määrab säriaeg ja kastis kuvatakse tegelik reaalajas kaadrisagedus.**Käsitsi**: saate kaadrisagedust piirata liuguriga (1 kuni ribalaiusega piiratud maksimumini), lähtudes praegusest tegelikust kaadrisagedusest. Massiivi liikmetele kuvatakse ainult lugemiseks mõeldud „N kaadrit sekundis (reaalajas)“ koos märkega „Määratud massiivi seadetes“. |

### Võrk / Andmeedastus

| Rida | Käitumine |
| --- | --- |
| **Paketi suurus**| 1500 (Standard) / 9000 (Jumbo) — vaikimisi**Jumbo**. |
| **Läbilaskevõime** | Ainult lugemiseks mõeldud ühenduse läbilaskevõime piirang MB/s. Tagapõhi jaotab selle ümber kõigi ühendatud kaamerate vahel iga ühenduse loomise või katkestamise korral. |
| **Puhvri haldamine** | Ainult lugemiseks mõeldud puhvri haldamise režiim. |

### Salvestamine

Paneel lõpeb nupuga **„Ava salvestusseaded…“**, mis viib [Salvestusseadete paneelile](capture.md#the-capture-settings-pane) (deaktiveeritud, kuni projekt on avatud — „Loo või ava projekt salvestuste salvestamiseks”). Kui kaamera on peidetud või peatatud, tuletab vihje meelde, et enne salvestamist tuleb kaamera uuesti nähtavaks teha või salvestamine jätkata.

## Massiivi seaded

Avage see ARRAY-rea **hammasrattaga**. Pealkiri: massiivi nimi koos ümbernimetamise pliiatsiga ja**×** sulgemiseks. Allpool *ainult kombineeritud* märgitud osad ilmuvad ainult kombineeritud kuvamisrežiimis ühendatud massiivide puhul.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Sünkroniseerimine

Ainult lugemiseks mõeldud **Master**,**Slaves**ja**Sync Line** read.

### Ümbritseva valguse andur

Näidatakse nii kombineeritud kui ka eraldi massiivide puhul:

* **Kalibreerimise sihtmärk** valikukast — „Tuvasta MAPIR ArUco sihtmärk ja valideeri NDVI võrreldes paneeli peegeldusastme LUT-iga”; juhib kombineeritud plaadi sihtmärgi pealiskujutist ja valideerimistabelit.
* **Valgusandur** rippmenüü — seob ühe DAQ-seadme kogu massiiviga. Valik jõustub kohe, levib iga massiivi kuuluva kaamera enda valgusanduri rippmenüüsse (saate ikkagi kaamera kaupa seda ümber kirjutada) ja hakkab spektreid massiivile edastama.
* Reaalajas **Staatus**-rida: Väljas · „Oodatakse esimest spektrit…&quot; · „Aktiivne — kõik massiivi kaamerad on valgustuse suhtes korrigeeritud“ · „Viimase 3 sekundi jooksul pole uut spektrit — kasutatakse endiselt viimast mõõtmistulemust (aegumist ei ole)…“.
* Märkus paneelis: „Massiiviülene radiomeetriline korrigeerimine. Kaamerapõhised seaded tühistavad selle.“

### Pildistamine — ühtsed anduri seaded *(ainult kombineeritud)*

Need seaded kehtivad ühtselt kõigi massiivi liikmete kohta (liikmekohased muudatused rikuksid sünkroniseerimist). Muudatused salvestatakse vahepealselt ja rakendatakse üheaegselt.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Kontroll | Valikud / vahemik | Mida see teeb |
| --- | --- | --- |
| **Piksli formaat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Ühtne anduri formaat kõigile liikmetele. |
| **Piksli ühendamine** | 1x1 / 2x2 / 4x4 | Riistvaraline piksli ühendamine — säilitab täieliku vaatevälja, suurendades samal ajal signaali-müra suhet (SNR) ja kaadrisagedust. Selle muutmine seadistab ROI-väljad uuele täielikule vaateväljale. |
| **Eraldusvõime** eelseadistus | Täis / Pool / veerand | Sõltub binningust; täidab ROI-väljad tsentreeritud kärpimisega. |
| **ROI kärpimine (px)**| W / H / X / Y numbrilised väljad | Anduri kärpimine. Laius/kõrgus joonduvad 16-kordsetele väärtustele (minimaalselt 64); nihked joonduvad 4-kordsetele väärtustele. „max WxH“ vihje näitab ülempiiri ja**Taasta** naaseb täisvaateväljale. Redigeerimise ajal joonistatakse maatriksi ruudule oranž reaalajas eelvaatekast (sealhulgas täissensori skeem, kui kärbitust väljapoole laiendatakse). |
| **Trigger Rate**| Automaatne / Käsitsi lüliti + fps 0,5–10, samm 0,5 |**Automaatne**(vaikimisi): tagapõhi arvutab käivitussageduse resolutsiooni ja ribalaiuse põhjal — sisestus on keelatud ja kuvab arvutatud väärtust.**Käsitsi**: kinnitab väärtuse nupule „Kohalda” vajutamisel. |

Märkus paneelis: „Formaadi/resolutsiooni muutmine taaskäivitab kõik kaamerad lühiajaliselt. Käivitussagedus rakendub reaalajas.” **Rakenda / Tühista** nupud asuvad paneeli allosas.

### Joondamine (koosregistreerimine) *(ainult kombineeritud)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Jääk**-paneel: „RMS x,xx px“ — roheline alla 1 px, kollane alla 3 px, muul juhul punane või kui mõni kaamera ebaõnnestus; „profiili pole“ enne esimest lahendamist.
* Kokkuvõteliin: „✓ Kõik kaamerad joondatud (N)“ / „⚠ p/N kaamerat joondatud —  <serial (filter)="">ebaõnnestus“ / „Kärpimine aktiivne — joondamiseks kalibreeri uuesti (kasutab kogu andurit)“ / „Oodatakse, kuni valgustatus stabiliseerub…“.
* Kaamerapõhine tabel: kaamera (seerianumbri viimased 4 numbrit + filter), reprojitseerimisviga pikslites koos sobituste arvuga („ref“ peakaamera puhul) ning kattuvuse normaliseeritud ristkorrelatsiooni skoor võrreldes 0,35 lävendiga.
* **Kalibreeri joondus uuesti** nupp (esimesel profiilil kuvatakse „Kalibreeri joondus”) — käivitab ko-registreerimise uuesti värskete kaadritega.
* **„Kaamerate automaatne valgustamine joondamiseks”** valikukast (vaikimisi märgitud) — heledab ajutiselt tumedaid või tasaseid kaameraid (esmalt valgustust, seejärel võimendust), et neil oleks tekstuur, mida sobitada, ning taastab seejärel automaatse valgustuse.

Kombineeritud eelvaade joondub avamisel automaatselt; kalibreerige uuesti, kui fookus või stseeni sügavus on muutunud. Joondamine on **disainilt ainult seansipõhine** — seda ei salvestata kunagi profiili, kuna see sõltub hetke stseeni kaugusest. Pildid saab siiski eksportida pikseliregistreeritud kujul (vt [Joondatud ekspordid](capture.md#per-array-controls)).

### Nutikas vinjet

* **Luba korrigeerimine**valikukast – rakendab iga kaamera vinjeti hinnangut radiomeetrilisele ahelale (reaalajas**ja** eksportides).
* **Kalibreeri praeguse vaate järgi**– suuna kaamerarida esmalt ühtlasele sihtmärgile (lame ekraan, sein või taevas); iga kaamera tasandatakse eraldi ja staatus näitab tasanduse kasvu „n/N kaamerat · −x,x %“.**Tühista** eemaldab hinnangu.
* Täpsustage iga kaamera puhul eraldi kaamera-põhise **Vignette** liuguriga [Reaalajas eelvaates](#live-preview).

### Reaalajas eelvaade *(ainult kombineeritud)** **Indeks**: märkige valikukast ja klõpsake hammasratast — avab jagatud [indeksi kalkulaatori](#index-calculator-pane), kus on joonistatud**kõigi** liikmekaamerate ribad. Allpool asuv väljendi eelvaate rida näitab praegust väljendit („Väljendit pole määratud — avage kalkulaator, et seda luua”), mida värskendatakse iga sekundi järel.
* **Renderdamise resolutsioon**rippmenüü (samad eelseadistused nagu kaamera kaupa, vaikimisi 720p): reaalaja vaatevoo kõrgus**ja** salvestatud komposiitpildi ekspordisuurus. Märkus paneelis: „Eelvaade + salvestatud komposiitpildi suurus. Kaamera kaupa pildid eksporditakse alati täisresolutsioonis.“

### Kuvamiskihid *(ainult kombineeritud)** **Luba** valikukast (vaikimisi välja lülitatud — peakaamera kuvatakse otse; sisse lülitatud = kihiline komposiit).
* **Esiplaan**/**Tagaplaan**rippmenüüd: iga liikmekamera (nime järgi) või**Indeks**. Kui esiplaaniks on valitud „Indeks”, kuvatakse LUT-i miinimum- ja maksimumpiiridest väljaspool asuvad pikslid tagaplaani kihina.

### Jagatud vaade *(ainult kombineeritud)*

**„Näita liikmekameraid“**— nupp**Jaga / Peida liikmekamerad**, mis lisab iga liikme oma reaalaja pildi eraldi ruudustikuna komposiidi kõrvale. Ruudud loevad massiivi olemasolevast kaadripuhvrist (ilma täiendava kaamera ühenduseta). Ainult ruudustikvaade; salvestatakse massiivi kaupa koos projektiga.

### Võimalused

Iga 5 sekundi järel uuenev ainult lugemiseks mõeldud paneel:

* **Tase**: „Samaaegne salvestamine“ (roheline) · „Samaaegne salvestamine (FTD-järkjärguline edastamine)“ (roheline) · „Järkjärguline salvestamine (100 ms viivitus)“ (kollane) · „Konfiguratsioon liiga suur“ (punane).
* **Kaadri tervis**: „x,xx % puudulik“ — roheline alla 1 %, kollane alla 5 %, punane 5 % või rohkem.
* **Ühendusliin**: „NIC {mbps} Mbps – püsikiirus {MB/s} MB/s“.

See on kaamerate massiivi reaalajas ribalaiuse eelarve. Aluseks oleva kaadrisageduse (fps) ja võrgumudeli kohta — ning selle kohta, mida muuta, kui tase muutub kollaseks või punaseks — vaata [Mitme kaameraga massiivid](arrays.md) ja [CLI viite](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Indeksi kalkulaatori paneel

Kolmas külgriba lehekülg, mida jagavad nii kaamerapõhine indeksihammasratas kui ka kombineeritud massiivi indeksihammasratas (korraga üks — pealkiri on „Indeksi kalkulaator — <camera name="">“ või „Indeksi kalkulaator —<array name="">

“). See võtab vastu sagedusribade nimekirja (kaamera filtri loomulikud sagedusribad või kõik sagedusribad kogu süsteemi liikmete ulatuses), kehtiva väljendi ja LUT-i konfiguratsiooni (sisse/välja, tase — vaikimisi 3, min — vaikimisi 0,2, max — vaikimisi 1) ning reaalajas indeksihistogrammi. Nupp **Rakenda** kinnitab väljendi; LUT-i muudatused rakenduvad reaalajas eelvaatesse.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Kaamerapõhised vs massiivi poolt hallatavad seaded

Kiirülevaade sellest, mis asub kus, kui kaamera on massiivi liige:

| Massiivi poolt hallatavad (kaamera paneelis ainult lugemisõigusega) | Jätkuvalt kaamerapõhised massiivi sees |
| --- | --- |
| Pikseliformaat, resolutsioon, binning | Automaatne säritus (säritus, võimendus, sihtmärk, silumine, ROI) |
| Käivitusrežiim/allikas, kaadrisagedus | Müra vähendamine, teravus, vinjetatsioon |
| | Suund (peegeldus/pööre), ekraanikatted, punktmõõtur |
| | Indeks (eraldi kuvatavad massiivid), valgusanduri sidumine |

Muud üldised toimingud:

* **Kombineeritud vs eraldi kuvamine** valitakse massiivi ühendamisel: kombineeritud = üks joondatud komposiitpaneel (liikmed edastavad pilti ainult Split View kaudu); eraldi = iga liige renderdab oma sünkroniseeritud paneeli. Kaamera ei näita kunagi korraga nii eraldiseisvat pilti kui ka massiivi paneeli.
* **Automaatne ühenduse taastamine**: salvestatud projekti avamisel taastatakse selle kaamerad ja massiivid ning kõik salvestatud seaded rakendatakse uuesti tagapõhjas enne voogude taastamist.
* **Salvestamise piiramine**: peidetud või peatatud kaamerad jäetakse funktsioonist „Salvesta kõik” välja; massiiv blokeeritakse täielikult ainult siis, kui KÕIK liikmed on peidetud või peatatud. Vaata [Salvestamise seaded ja režiimid](capture.md).

## Kuidas seaded säilivad

Kaamera vahekaardi seisund salvestatakse **koos projektiga**, mitte brauseris:

* Iga reageeriv muudatus salvestab kaamerate ja massiivide hetkeseisu projekti `cameras.json`-i (500 ms viivitusega). See hõlmab kaamerate nimesid ja värve, säri/võimenduse/automaatse säri (AE) seadeid, pikseliformaati/lahutusvõimet/binningut, käivitussagedust, eelvaate seadeid (renderdamise eraldusvõime, müra vähendamine, teravus, vinjet, värviprofiil, küllastus/kontrast), orientatsioon, pealekatted, kanalite jagamine, indeksite konfiguratsioon, ennustav AE-seaded, AE ROI, massiivide nimed, kuvamisrežiim, massiivide salvestusseaded (sealhulgas ROI kärpimise asukoht) ning võrgustikubloki (voo suum, vaatamisrežiim, võrgustiku lukustus, käsitsi määratud ruutude järjekord, peidetud kaamerad, suletud ruudud, aktiivne kaamera).
* Valgusandurite seosed salvestatakse projekti `sensors.json`-faili.
* Projekti uuesti avamisel ühendatakse riistvara uuesti ja kõik seaded rakendatakse uuesti.
* **Kui projekt pole avatud = ainult seanss**: projekti puudumisel ei säili midagi pärast Chloros sulgemist.
* Ainult seansipõhine, sõltumata projektist: pausi olek, punktmõõturi proovid, kaamera-põhine kalibreerimise sihtmärgi valikukast (on alati vaikimisi välja lülitatud) ja massiivi joondamisprofiil (arvutatakse disaini järgi iga seansi jaoks uuesti).
* Üks erand: **Pildistamisseaded** ekspordivalikud ja pildistamisrežiim säilivad projekti kaupa kohalikus rakenduse mälus, mitte `cameras.json`-is — vt [Pildistamisseaded ja -režiimid](capture.md).</array></camera></serial>
