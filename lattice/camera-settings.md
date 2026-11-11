# Kaamera seaded

Vahekaart **Kaamerad** on Chloros

i LATTICE-kaamerate reaalajas juhtpaneel: peamine vooala, kus iga ühendatud kaamera kuvatakse reaalajas ruuduna, ning külgriba, mida saab liigutada kolme lehe vahel – **kaamerate nimekiri**,**seadete paneel**(kaamera-, massiivi- või salvestusseaded – üks korraga) ning**indeksikalkulaator**. Sellel lehel on kirjeldatud kõiki kaamerate nimekirjas, kaamera- ja massiivi seadete paneelides olevaid juhtnuppe. Salvestusrežiimid, eksporttüübi valik ja funktsioon „Capture All“ on kirjeldatud kaaslehel [Salvestusseaded ja režiimid](capture.md).

Vahekaart „Kaamerad“ ilmub külgribale, kui Chloros

i taustsüsteem on valmis. Kõik allpool nimetatud juhtelemendid suhtlevad kohaliku taustsüsteemiga kaudu `127.0.0.1:5000` kaudu; muudatused rakenduvad kaamera reaalajas pildile kohe, kui pole märgitud teisiti.

## Sellel lehel kasutatavad kaameratüübid

Juhtelemendid kuvatakse või peidetakse sõltuvalt sellest, milline kaamera on valitud. Käsiraamatus kasutatakse järgmisi termineid:

| Termin | Tähendus | Filtrikanalid |
| --- | --- | --- |
| **RGB

-kaamera** | LATTICE M3C koos FRGB-filtriga (mudel sisaldab `-FRGB`) | Red

/ Green

/ Blue

|
| **Bayer multispektraalne** | LATTICE M3C koos FRGN-, FOCN- või FNGB-filtriga | FRGN: Red

/ Green

/ NIR

· FOCN: Orange

/ Cyan

/ NIR

· FNGB: NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M — üks kitsasriba filter, üks kalibreeritud riba | Üks riba |
| **Massiivi liige** | Kaamera, mis on ühendatud sünkroniseeritud massiivi osana (ühendatud või eraldi kuvamine) | Vastavalt oma filtrile |

 RGB

-kaamerad läbivad fotomeetrilise töötluse (valgebalanss, värviprofiilid, gamma); multispektraalsed ja mono-kaamerad läbivad radiomeetrilise ahela ning vahevad fotomeetrilised seadistused vahele. Massiivi liikmed edastavad voo tasandi seaded (pikseliformaat, resolutsioon, binning, päästik, kaadrisagedus) massiivile — need read muutuvad kaamera-paneelis ainult lugemiseks ja liiguvad massiivi seadete paneeli.

## Peamine vooala

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Kui kaameraid pole ühendatud, kuvab vooala tervitusekraani **„Ühenda kaamera, et alustada“**koos kahe nupuga:**Ühenda kaamera**(roheline, avab ühe kaamera ühendamise dialoogi) ja**Ühenda kaamerarühm** (sinine, avab kaamerarühma ühendamise dialoogi). Ühendamise dialoogid on kirjeldatud jaotises [Kaamerate ühendamine](connecting.md); kaamerarühma põhimõtted (sünkroniseerimine, tasemed, ribalaius) on kirjeldatud jaotises [Mitme kaamera massiivid](arrays.md). Kui avate salvestatud projekti, milles on kaamerad, kuvatakse avalehel selle asemel pöörlev ikoon koos tekstiga „N salvestatud kaamerat avatakse uuesti…“, samal ajal kui Chloros

taastab viimase seansi vooge.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Ülemine riba

| Juhtnupp | Mida see teeb |
| --- | --- |
| **Vaate režiimi lüliti**| Lülitab**võrgustikuvaate**(kõik ruudud on lahtrid) ja**nimekirjavaate**(täislaiused read: iga kombineeritud massiiv ühe komposiitreana, iga**eraldi** massiivi liige oma reana koos oma voo ja kalibreerimise sihtmärgi tabeliga, seejärel ÜKS eraldiseisev kaamera – aktiivne – allpool) vahel. Vihjed: „Lülita üle ruudustikku” / „Lülita üle nimekirja”. |
| **Ruudustiku lukustus**(riiv) | Vaikimisi**lukustatud** — ruudud on paigal külmutatud. Lukustuse vabastamisel saab ruute lohistada ja ümber järjestada mis tahes kohale (vahed säilivad). Võrk lukustub automaatselt uuesti iga kord, kui ühendub uus kaamera. Vihjed: „Lukusta võrk lahti (võimaldab ruutude lohistamist)“ / „Lukusta võrk (kinnita ruudud paigale)“. |
| **Voogu suurendamise** liugur | Ruudu suurus, alates 60 pikselist kuni kogu konteineri laiuseni. Ruudud säilitavad 4:3 kuvasuhte. Kui ruudu laius on alla 200 px, peidetakse nimi ja kaadrisageduse (fps) ülekatted, et ruut jääks puhtaks. |

### Vooru ruudud

Iga kaamera renderdab komposiitse reaalajas ruudu; kaamera võib lisaks näidata kolme halltoonides **kanalite kaupa jagatud** ruutu (vt [Kanalite jagamine](#display-overlays-drawn-over-the-live-feed)), ning massiivid kuvavad kombineeritud ruutu. Aktiivsel ruudul on valikuring kaamera (või massiivi) värvis.

Ruudu peale hiirega liikudes ilmub **X** sulgemisnupp:

* Kui sulgeda **komposiit**-ruut, kui selle kanalite jagunemised jäävad alles, peidetakse vaid komposiit.
* Kui sulgeda **eraldi kaamera viimane nähtav ruut**, katkestatakse selle kaameraga ühendus.
* **Kombineeritud kaamerarühma liikmete jagatud ruudud ei katkesta kunagi** kaameraga ühendust — need peidetakse vaid.

Kui võrgustik on lukustamata, lohista mis tahes ruutu mis tahes pesasse; paigutus salvestatakse koos projektiga.

## Külgriba — kaamerate nimekiri

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Esimesel külgriba lehel on loetletud kõik ühendatud kaamerad ja massiivid:

* **Ühenda kaamera**(roheline) /**Ühenda massiiv** (sinine, kuvab skannimise ajal sõnumit „Otsin...“). Mõlemad on keelatud, kui ühendamise dialoog on avatud.
* **Salvesta kõik** (punane) — salvestab kõik loetletud kaamerad koos salvestusseadetes valitud eksporditüüpidega. Nõuab avatud projekti. Täielik kirjeldus on toodud jaotises [Salvestusseaded ja režiimid](capture.md).
* **Salvestusseadete hammasratas** (nupu „Salvesta kõik” kõrval) — avab [salvestusseadete paneeli](capture.md#the-capture-settings-pane). See on keelatud, kui projekti pole või salvestamine on käimas.

### Kaameraridad

Igal kaameraridal on värvikoodiga piirjoon (kaamera kohandatud värv), silt „CAM“ — koos sinise **M**(master) või rohelise**S** (slave) rollitähega massiivi liikmetele — ning kuvanimetuse. Vaikimisi nimi on `LATTICE-MODEL (serial)`; seda saab ümber nimetada kaamera-spetsiifiliste seadete paneelis. Rida nupud:

| Nupp | Toime |
| --- | --- |
| **Silm**| Nähtavuse lülitamine. Peidetud kaamerad eemaldatakse võrgustikust ja**jäetakse välja funktsioonist „Capture All“**. |
| **Hammasratas** | Avab kaamera-spetsiifilise seadete paneeli (järgmine osa). |
| **Paus / Esita**| Külmutab reaalaja eelvaate**ainult ekraanil** — tagapõhja salvestamine jätkub. Pausile pandud kaamerad ei saa salvestada. |
| **X** | Ühendus katkestamine. Kasutajaliides uueneb kohe (optimaalsel juhul); tagapõhja ühenduse katkestamine võib võtta aega 10–30 sekundit. |

### Massiivi read

Massiivi real kuvatakse massiivi värviga märge „ARRAY”, massiivi nimi (mida saab massiivi seadetes ümber nimetada) ja **DAQ · sisse/välja**nupp —**sisse**, kui massiivi tasandil on valgustundlik sensor sisse lülitatud *või* mõnel liikmel on kaamera-põhine sensor; selle abiteates on täpselt loetletud, milline andur millist pilti edastab. Liikmekaamerad on loetletud allpool sisselõigatud ridadena. Massiivi rea nupud: **silm**(peidab/näitab KÕIKI liikmeid korraga),**hammasratas**(massiivi seadete paneel),**X**(kogu massiivi ühenduse katkestamine).

Massiivi ridades ja massiivi seadete paneelis kasutataval valgussensoril (DLS) on neli seisundit:**väljas**,**ootab**(spektrit pole veel),**aktiivne**(spekter saabus viimase 3 sekundi jooksul) ja**vananenud** — 3 sekundi jooksul pole uut spektrit, kuid viimast näitu *kasutatakse endiselt* (DAQ-näidud ei aegu kunagi salvestusteel).

Kuni te seda ümber ei järjestage, kuvatakse nimekirjas esmalt massiivid, seejärel eraldiseisvad kaamerad, kusjuures kaamerad (ja iga massiivi liikmed) on **kaamerate järjekorras**: M3C (Bayer) enne M3M (mono); M3C-de seas on järjekorras FRGB, FRGN, FOCN, seejärel FNGB; M3M-id filtri lainepikkuse järgi, madalaimast alates. Võre vaikimisi paigutus järgib sama järjekorda. Saad loendi järjestust muuta, lohistades eraldiseisvaid kaameraid ja terveid massiivirühmi külgribal üksteise kõrvale; massiivi liikmeid ei saa eraldi lohistada.

## Kaamera-põhised seaded

Avage need kaamera rea **hammasrattaga**. Paneel libiseb kaamerate nimekirja peale.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Pealkiri**: kaamera**värvivalik**(klõpsake, et avada sisseehitatud värvivalija — määrab külgriba ääre ja ruudu valimisrõnga värvi),**nimi**koos pliiatsikujulise**Ümbernimetamine**-nupuga (tühja nime salvestamisel taastub vaikimisi `MODEL (serial)`) ja**×** sulgemiseks.

### Reaalajas histogramm

Paneeli ülaosas asub reaalajas luma-histogramm, mis arvutatakse ~8 Hz sagedusega JPEG

-eelvaate põhjal. Keskmine on Bayer-kaalutud — (R+2G+B)/4 — et see vastaks kaamera enda AE-mõõtmisele.

* **Oranž Orange

joon**= AE sihtmärk.**Lohistage seda horisontaalselt, et sihtmärki muuta** — vabastamisel saadetakse üks käsk ning lohistamine lülitab AE sihtmärgi režiimi käsitsi režiimile.
* **Roheline Green

joon** = tegelik keskmine heledus (mida AE hetkel annab).
* **RGB

-nupp** (ülemine parem nurk): lülitab sisse kaamera filtri järgi värvitud ribapõhised pealekantavad histogrammid (nt FRGN-režiimis: hall NIR

, roheline, punane). Mono (M3M) kaameratel on nupul kiri „MONO“ ja see on keelatud — mono režiimis kuvatakse alati üheribalist heleduse histogrammi.
* X-telje märgistused järgivad praeguse pikseliformaadi anduri bittisügavust: 0..255, 0..1023, 0..4095 või 0..65535.

### Kaamera teabe read

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->



| Rida | Käitumine |
| --- | --- |
| **Mudel** | Ainult lugemiseks (nt `LATT-M3C-L87-FRGN`). |
| **Radiomeetriline kalibreerimine** | Green**„Active“**märgis, mille allkirjas on näidatud kalibreerimistasand, hash, kalibreerimise kuupäev ja sagedusribade loend, mis on laaditud kaamera kalibreerimispaketist (vt [Tehase radiomeetriline kalibreerimine](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**RGB

-kaamerate puhul peidetud** — neil on fotomeetriline valgebalansi kalibreerimine, mitte sagedusribapõhine kiirgus. |
| **Kalibreerimisaruanne**|**Lae alla** nupp — avab kaamera seerianumbri järgi koostatud NIST-kalibreerimissertifikaadi PDF-faili teie operatsioonisüsteemi vaatajas. Kui sertifikaati pole veel vahemällu salvestatud, näitab Chloros

selle asemel vihjet. |
| **Seerianumber** | Ainult lugemiseks. |
| **Püsivara**| Näitab praegust versiooni, seejärel otsib selle mudeli jaoks saadaval oleva versiooni (puhverdatud mudeli kaupa — N-kaamerast koosnev massiiv kontrollib serverit üks kord). Seisundid: „Kontrollin…“ →**„Uuenda versioonile X“**nupp → „Uuendamine…“ → „Uuendatud A → B“ / „Ebaõnnestus: …“ / „Vahelet jäetud: …“ / roheline**„Ajakohane“**. Uuendamisnupu vihje: „Tehase seadete taastamine + tarkvara uuendamine + UserSet1 ümberprogrammeerimine. ~2–3 minutit; ühendust ei tohi katkestada.“ |
| **IP** | Ainult lugemiseks. |
| **Temperatuur** | Ainult lugemiseks, värskendatakse iga 3 sekundi järel. Muutub oranžiks temperatuuril ≥65 °C ja punaseks koos märkega ⚠ temperatuuril ≥75 °C. |
| **Kalibreerimise sihtmärk** valikukast | Lülitab sisse ArUco peegeldusvõime sihtmärgi tuvastamise koos paneelipõhise NDVI

-valideerimistabeliga reaalaja pildi all (loendivaade) — vt [Kalibreerimise sihtmärgi tabelit](#the-calibration-target-table). Ainult seansiks — avatakse alati välja lülitatuna. |
| **Valgusandur** rippmenüü | Seob DAQ-valgusanduri (DAQ-E/M/U, valikust „Valgusandurid”) selle kaameraga, et teha allapoole suunatud valguse (DLS) valgustuse korrigeerimist ja ennustavat automaatset säritust. „„Puudub“ tühistab seose. Kui andureid pole ühendatud, kuvab rippmenüü „(andureid pole ühendatud — ava DAQ-vahekaart)“. Seos salvestatakse koos projektiga. |

#### Kalibreerimismärgi tabel

Tabel on **ainult valideerimiseks** („ainult valideerimiseks — ei muuda kalibreerimist”). Selle pealkirjas on märgitud kaamera seerianumber, filter, märgi number ja märgi suurus pikslites ning igale paneelile vastab üks rida:

* **Paneel**, seejärel üks veerg iga laineala kohta, milles on paneeli väärtus — Bayeri kaamera näitab oma kolme laineala nime ja lainepikkuse järgi (nt FRGN: Red

660 nm / Green

550 nm / NIR

850 nm); **mono (M3M) kaamera näitab oma üht sagedusriba** (nt `F750`) ja sellel puuduvad NDVI

-veerud.
* **Sat** — paneeli küllastunud pikslite osakaal (merevaik alates 1 %, punane alates 5 %).
* **NDVI

Meas**,**NDVI

Theo** (paneeli võrdluspeegeldusvõime NDVI

) ja **Δ**— ainult Bayeri multispektraalkaamerad. Väike märge „NDVI

Meas“ järel näitab, mille alusel mõõdetud NDVI

arvutati:**ρ**(peegeldusvõime — valgusanduri näidud on kättesaadavad),**L**(kiirgus – valgusanduri näitu pole) või**DN**(kaamera toorväärtused). Ainult**ρ**-NDVI

on võrreldav NDVI

Theo-ga: L ja DN puhul on veerg Δ halliks muudetud ja märkus selgitab seda („Valgusanduri näitu pole: NDVI

on arvutatud kiirgusest, seega Δ ei ole kalibreerimiskontroll“ / „NDVI

on arvutatud kaamera toorväärtustest, seega Δ ei ole kalibreerimiskontroll“). Väärtuse ρ puhul on Δ roheline, kui väärtus on alla 0,02, kollane, kui alla 0,05, muul juhul punane.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Ekspositsioon ja võimendus

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Kõik siinsed numbrilised sisestusväljad kasutavad kiirendamiseks hoidmisega reguleeritavaid valikurattaid: puudutus = ±1, hoidmine &gt;1,5 s = ±10, hoidmine &gt;3 s = ±100. Väärtus saadetakse kaamerale, kui sõrme lahti lasete.

| Seade | Vahemik / valikud | Vaikimisi | Kehtib | Mida teeb |
| --- | --- | --- | --- | --- |
| **Valgustusaeg (us)**| Kaamera reaalajas min/max | Auto | Kõik | Valgustusaeg mikrosekundites, koos**Auto/Käsitsi**lüliti. Auto = kaamera poolt pidevalt reguleeritav automaatne säritus. Käsitsi särituse seadistamisel jääb**võimendus** automaatrežiimile (kui see oli varem nii), võimaldades kasutada kaamera kogu võimendusvahemikku, kuni võimendust muudate või seadistate käsitsi režiimile. |
| **Võimendus (dB)**| Kaamera reaalajas min/max (nt kuni 48 dB) | Käsitsi (väljas) | Kõik | Analoog-/digitaalne võimendus koos oma**Auto/Käsitsi** lülitiga. |
| **AE sihtheledus**| 0–255 | 80, režiim**Auto**| Kõik (muudetav, kui AE või automaatne võimendus on sisse lülitatud) | Heledus, millele AE püüab saavutada.**Auto**(vaikimisi) režiimis valib histogrammipõhine taustkontroller sihtmärgi ise, hoides säritust 60–75 %-l anduri maksimumist. Väärtuse sisestamine või histogrammi oranži joone lohistamine lülitab selle üle**Käsitsi** režiimile. |
| **AE silumine** | 0,5–40, samm 0,1 | 8,0 | Kõik | AE summutamine. Vihje: &quot;Madalam väärtus = AE reageerib kiiremini (võib kõrgel kaadrisagedusel pulseerida). Kõrgem väärtus = sujuvam / aeglasem.” Vaikimisi seadistusest oluliselt madalamad väärtused võivad põhjustada AE pulseerimist ja destabiliseerida voogedastust kõrgel kaadrisagedusel; 8,0 on stabiilne vaikimisi seadistus. |
| **AE huvipiirkond**| Lülita sisse valikukast +**Sihtimine**nupp | Väljas | Kõik | Kui see on sisse lülitatud, mõõdab AE ainult rohelise katkendjoonega piiratud piirkonda, mitte kogu kaadrit.**Sihtimine** aktiveerib reaalajas pildil klõpsuga paigutamise: klõps keskendab piirkonna kaadri 30 %-le; klõps ja lohistamine joonistab kohandatud ristküliku (minimaalselt 5 % × 5 %). „Aim“ lülitub pärast ühte paigutamist ise välja. Piirkond kaardistatakse tagasi kaamera algsetele koordinaatidele vastavalt teie poolt seatud pööramisele/peegeldamisele ning salvestatakse koos projektiga. |
| **AE Tune Speed** | 0,1–5, samm 0,1 | 1,0 | Ainult massiivi liikmetele | Kui kiiresti automaatne AE-sihtmärk jälgib stseeni heleduse muutusi; 1,0× kontrollib uuesti iga 2,5 sekundi järel. |
| **Heledate alade kaitse** | Range (1 %) / Normaalne (5 %) / Leebe (15 %) | Range | Kaamerad, mis toetavad seda seadet | Kui suur osa kaadrist võib muutuda valgeks, enne kui AE pilti tumedamaks muudab. |

{% hint style="info" %}
**Valgustuse nõuded Bayeri multispektraalsetele kaameratele (RGN

/ OCN

/ NGB

):** stseenis peab olema piisavalt valgust kõigis kolmes kanalit, vastasel juhul kalibreerimine ei tööta korrektselt — ühe anduri ekspositsioon hõlmab kõiki kolme spektrit. Kasutage valguse mõõtmiseks DAQ-valgusandurit või valige täielikult monokroomne režiim (M3M), et iga sagedusriba saaks oma ekspositsiooni. Kui pildistamine seda nõuet rikub, tuvastab Chloros

selle ja hoiatab teid (unmix-clamp-teade). Reaalajas kuvab kanalite jagatud pilt **„Madala usaldusväärsusega sagedusriba andmed · N%”**, kui unmix on selle sagedusriba 25 % või enamal selle pikslitest klambriga piiranud.
{% endhint %}

{% hint style="info" %}
**Kuidas automaatne ekspositsioon ja võimendus toimivad (1.3.0).** Kui võimendus on automaatrežiimil, hoiab Chloros

võimendust 0 dB tasemel, kuni ekspositsioon on saavutanud lubatud maksimaalse väärtuse (automaatse ekspositsiooni ülempiir, massiivi triggiperiood või DAQ ekspositsiooni ülempiir); alles siis tõuseb võimendus. Massiivis on triggisagedus esikohal: automaatne ekspositsioon täidab sageduse poolt määratud perioodi, seejärel tõstab võimendust, mistõttu pimenev stseen ei vähenda enam automaatse triggereerimissagedust (versioonis 1.2.0 see veel mõjutas). **Käsitsi**valgustusaeg massiivi liikmel, mis on pikem kui triggereerimisperiood, aeglustab massiivi triggereerimist, et see sobiks; automaatne sagedus taastub siis, kui kõik seda aeglustanud elemendid on tagasi automaatse ekspositsiooni režiimis. Kui**eksponatsioon**ja**AE sihtheledus** on mõlemad automaatrežiimis ning DAQ valgussensor annab näitu, hoitakse iga kaamera ekspositsiooni muutumatuna pärast automaatse ekspositsiooni stabiliseerumist, kuni mõõdetud valgus jääb vahemikku ±25 %; suurem ja püsiv valguse muutus, seadete taaskäivitamine (näiteks huvipiirkonna (ROI) muutus) või teie enda poolt tehtud ekspositsiooni muutus käivitab selle uuesti.
{% endhint %}

{% hint style="info" %}
**Automaatse ekspositsiooni toimimine (1.3.0).**

* Käsitsi ekspositsiooni korral jääb võimendus automaatrežiimile, kuni te võimendust muudate.
* Väikesed heledad alad ei muuda stseeni tumedamaks: eiratakse kuni 1 % kaadrist moodustavat peegeldust ning teadaoleva kalibreerimismärgi paneelidel kuni 0,5 % paneelist.
* DAQ-eksponatsiooni ülempiir kehtib ainult siis, kui kalibreerimismärk on vaateväljas; kui paneeli ei ole näha, eksponeerib kaamera stseeni järgi.
* Valgustamine püsib stabiilsena ühtlases valguses ja stabiliseerub mõne sekundi jooksul pärast suurt muutust. Ülevalgustamine ei piira enam valgustamist püsivalt ning huvipiirkonna (ROI) muutumisel leitakse uuesti õige valgustamine.
* Heleduse sihtmärk ei ületa kaameraulatusest välja, seega tagasitulev valgus ei tekita kaadris helklemist.
* Kaamera, mille automaatne valgustustundlikkus lakkab reageerimast eredas päikesevalguses, hoitakse valgustustasemel, mis vastab valgustuse anduri poolt loetud valgustingimustele või, kui andurit pole ühendatud, kaamera enda kaadrite põhjal, ja vabastatakse, kui valgus muutub. Pime või varjatud valgusandur seda ei häiri.
{% endhint %}

### Pikseliformaat ja eraldusvõime

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Massiivi liikmed** näitavad ainult lugemiseks mõeldud ridu „Current“ (formaat + WxH) ja „Binning“ koos märkusega „Set in array settings“ – ühe liikme voo taaskäivitamine katkestaks sünkroniseerimise, seega hallatakse neid [massiivi seadete paneelis](#array-settings-pane).**Eraldi kaamerad** saavad:

| Juhtimine | Valikud | Mida see teeb |
| --- | --- | --- |
| **Piksli formaat** | 8-bit / 10-bit / 12-bit / 16-bit | Anduri bittisügavus; formaadiperekond on anduri oma. Vaikimisi on 12-bit, projekt salvestab sinu seatud väärtuse, sealhulgas 8-bit, ja ühendub uuesti selle järgi. |
| **Eraldusvõime** | Täis / Pool / Veerand | Suhteline praeguse binninguga: Täis = 2048/N × 1536/N N×N-binningu puhul. |
| **Pikslite ühendamine** | 1x1 (puudub) / 2x2 / 4x4 | Riistvaraline N×N pikslite ühendamine — suuremad väärtused vähendavad eraldusvõimet, kuid suurendavad signaali-müra suhet (SNR) ja kaadrisagedust. Selle muutmine taaskäivitab voo ja seadistab kõik huvipiirkonnad (ROI) uuele täisvaateväljale. |
| **Praegune** | ainult lugemiseks | Kehtiv tegelik laius x kõrgus ja (x, y) nihke. |

### Reaalajas eelvaade

Kõik selles jaotises on **ainult kuvapoolne**— see muudab seda, mida näed reaalajas voos, samas kui salvestatud kaadrid jäävad lineaarseteks ja muutmata — ühe erandiga:**Vignette** on radiomeetriline ja mõjutab ka eksportimist (nimetatud allpool).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Seade | Vahemik / valikud | Vaikimisi | Kehtib | Mida teeb |
| --- | --- | --- | --- | --- |
| **Renderdamise resolutsioon** | 360p (kiireim) / 480p / 720p / 1080p / Anduri algne resolutsioon (aeglaseim) | 720p | Kõik | Kõrgus, millel tagapõhi käitab radiomeetrilist eelvaateketi. Madalam väärtus tagab suurema kaadrisageduse, muutmata vaatevälja. |
| **Indeks**| Valikukast „Luba“ + hammasratas | Väljas | Ainult Bayeri multispektraalne,**mitte** kombineeritud maatriksi liikmed | Reaalajas taimestikuindeksi eelvaade. Hammasratas avab jagatud [indeksi kalkulaatori](#index-calculator-pane), millesse on eelnevalt laaditud kaamera filtri loomulikud lainepikkused (nt `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Iga eelvaatekaadri puhul arvutatakse välja kohandatud avaldis koos LUT-iga (sisse/välja, vaikimisi tase 7-stop, vaikimisi miinimum 0,2, vaikimisi maksimum 1) arvutatakse iga eelvaatekaadri puhul. Kombineeritud massiivi liikmed peidavad selle rea — massiivil on üks jagatud indeks. |
| **Valgebalanss** | Väljas / Üks kord / Pidev + uuesti salvestamise nupp | Pidev | Ainult RGB

| Reaalajas valgebalanss. Värskendamisnupp salvestab valgebalansi uuesti praegusest DLS-spektrist (on välja lülitatud, kui režiim on „Väljas”). |
| **Gamma** | Sisse / Väljas | Sisse | Ainult RGB

| Näita gamma (γ = 2,2 LUT) reaalajas eelvaates. Salvestatud pildid jäävad lineaarseteks. |
| **Müra vähendamine** | Valikukast + tugevus 0–100 | Väljas / 50 | Kõik (iga kaamera kohta, isegi massiivides) | Kahepoolne filter reaalajas eelvaates. Kõrgem väärtus = siledam, kuid pehmemad detailid. |
| **Teravus** | Valikukast + tugevus 0–100 | Väljas / 30 | Kõik | Teravustamismaski kasutamine reaalajas eelvaates, rakendatakse viimasena. Võib müra võimendada. Ainult eelvaates. |
| **Vignett**| Valikukast + tugevus 0–100 | Väljas / 0 | Kõik | Käsitsi jääkvigneti eemaldamine (heledamaks nurgad), mis lisatakse massiivi Smart Vignette&#x27;i hinnangu peale.**Radiomeetriline — mõjutab reaalaja vaadet JA eksporti**, erinevalt müra vähendamisest ja teravusest. |
| **Värviprofiil** | Raw / Lineaarne / Looduslik / Täiustatud / Kohandatud temperatuur | Looduslik | Ainult RGB

| Vaata allpool. |
| **Värvitemperatuur** | 2000–10000 K, sammuga 100 | 5500 K | Ainult RGB

, kohandatud temperatuuriprofiil | Seab valgebalansi kindlale korrelatsioonilisele värvitemperatuurile (DLS-sisendit ignoreeritakse). Viimati valitud Kelvini väärtus jääb meelde ka profiilide vahetamisel. |
| **Heledamad alad** | −100 kuni 0, sammuga 1 | 0 (väljas) | Ainult RGB

| Uus versioonis 1.3.0. Tumendab pildi kõige heledamaid ja valgeimaid alasid (0 = muutmata, −100 = tugevaim). Kehtib reaalajas eelvaatele **ja ekraanitasandi (eelvaate) piltide salvestamisele**, mis salvestavad selle seadistuse (`Chloros:DisplayHighlights`); see ei mõjuta kunagi kiirguse ja peegeldusvõime eksportimist. See ei suuda taastada üleklippunud andureid – selleks tuleb vähendada säritust. Zebra-kiht järgib tumendatud pilti. Keelatud Raw-värviprofiili puhul („Raw-värviprofiili korral ei rakendata.”). |
| **Küllastus** | 0–200 (100 = neutraalne) | 100 | Ainult RGB

| HSV-küllastus reaalajas eelvaates. |
| **Kontrast** | 0–200 (100 = neutraalne) | 100 | Ainult RGB

| Lineaarne kontrast keskmise halli ümbruses reaalajas eelvaates. |
| **Peegelda horisontaalselt / Peegelda vertikaalselt** | Valikukastid | Väljas | Kõik | Pöörab eelvaadet horisontaalselt / vertikaalselt. |
| **Pööre**| 0° / 90° / 180° / 270° | 0° | Kõik | Pöörab eelvaadet. Suund rakendatakse eelvaate ahela lõpus.**Salvestatud pildid (1.3.0):**eraldiseisva kaamera eelvaate, kiirguse, peegelduse ja indeksi tooted pööratakse peegelduse/pööramise abil, samas kui**Raw ja Debayered jäävad kaamera algse orientatsiooni juurde**; iga pildifail salvestab seadistuse oma XMP-sse, nii et hilisem RAW-faili töötlemine rakendab seda samamoodi.**Massiivi liikmete failid jäävad algseks** (seadistus salvestatakse, kuid ei rakendata) ning massiivi komposiitvaated ignoreerivad seda. |**Värviprofiili semantik** (RGB

-kaamerad):

* **Raw** — töötlemisahel jäetakse täielikult vahele.
* **Lineaarne** — tume signaal + tasandus + valgebalanss; ei värvimatriitsi ega gammat.
* **Natural** *(vaikimisi)* — lineaarne pluss mõõdetud värvikorrektsioonimatriits ja stseeni järgi kohanduv toonikõver.
* **Enhanced**— Natural pluss värviküllus ja CLAHE kohalik kontrast. Lisakulu kehtib**ainult reaalajas eelvaate puhul** — salvestatud pildid saavad alati täieliku viimistluse, olenemata profiilist.
* **Kohandatud temperatuur** — „Natural”, kus valgebalanss on fikseeritud teie valitud Kelvini väärtusele.

{% hint style="warning" %}
Profiilide „Natural“, „Enhanced“ ja „Custom Temperature“ puhul kuvatakse paneelis toonimärkus: kaadrid heledamaks muudetakse vastavalt nende enda stseenile, seega ei ole salvestatud *ekraanil* kuvatavaid pilte kaadrihaaval võrreldavad. **Ekspordi kiirgusvõimsus või peegeldusvõime mõõtmiste jaoks.**
{% endhint %}

### Ekraanil kuvatavad kihtid (joonistatud reaalajas voo peale)

Need on ainult kasutajaliidese funktsioonid — joonistatakse video peale, puudutamata kunagi voogu ega salvestusi.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Kiht | Juhtelemendid | Vaikimisi | Mida see teeb |
| --- | --- | --- | --- |
| **Zebra** | Valikukast + künnis 200–255 | Väljas / 250 | Magenta diagonaalsed triibud kärbitud pikslitel. |
| **Rist** | Valikukast | Väljas | Kaadri keskpunkti märk. |
| **Võrgustik** | Väljas / 3 × 3 / 9 × 9 | Väljas | Kompositsioonivõrgustik. |
| **Histogramm** | Valikukast + laius 0,10–0,90 kaadri laiusest | Väljas / 0,25 | Otseülekandes kuvatav histogrammiriba. |
| **Fookuse tipp** | Valikukast + künnis 20–200 + värviproov | Väljas / 80 / `#ff5722` | Sobeli servade esiletõstmine fookustamiseks. |
| **Kanali jagamised** | Nupp „Näita jagamisi (Red

/ Green

/ NIR

)” / nupp „Peida jagamised“ | Peidetud | Lisab komposiitpildi kõrvale kolm sõltumatut halltoonilist kanalipõhist ruutu (nupu silt vastab kaamera filtrikanalitele). Iga jagatud ruutu saab lohistada ja see jagab kaamera raamivärvi. Ei ole saadaval monokroomkaameratel. Salvestatakse koos projektiga. |

### Punktmõõtur

* **Klõpsa proovi võtmiseks**valikukast: klõpsa reaalajas pilti, et võtta proov ühest pikslist (seda märgib ristkülik), või klõpsa ja lohista piirkonda, et saada pikslite keskmine.**Tühjenda**kustutab proovi ja sihtimismärgi. Vastastikku välistav AE-ROI**Sihtimise** režiimiga.
* **Näita**rippmenüü:**Raw (bitisügavus)**— anduri bitisügavusega algupärased digitaalarvud (nt 12-bitine → 0..4095) — või**Näita (8-bitine)** (vaikimisi). Kui reaalajas indeks on aktiivne, näitab „Display“ selle asemel arvutatud indeksväärtust (nt NDVI

).
* Näidupaneelil on loetletud pikselikoordinaadid, kaadri suurus, pikseliformaat, bittisügavus ja kanalitabel (Chan / Value / %) koos lainepikkuste ja kanalite nimetustega; Bayeri rohelised paarid on keskmistatud; piirkonna proovid näitavad „N px avg“.

Punktmõõturi olek kehtib ainult käesoleva seansi ajal.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Ennustav automaatne säritus (DLS-põhine)

See osa kuvatakse ainult siis, kui **on ühendatud vähemalt üks DAQ-valgusandur** — lahendaja vajab selle juhtimiseks reaalajas allapoole suunatud spektrit.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Juhtimine | Vahemik | Vaikimisi | Mida see teeb |
| --- | --- | --- | --- |
| **Luba** | Valikukast | Sisse (eraldiseisvad kaamerad) | Suletud vormi lahendaja kasutab DLS-spektrit ja kaamera kalibreerimispaketi skalaare, et viia kõige heledam riba küllastuse lähedale, hoides samal ajal kõige tumedama riba SNR-alampiiri kohal — üks eksponeerimine lahenduse kohta, ilma stabiliseerimistsüklita. Mõeldud päikeseenergiaga töötavate aeg-luupide jaoks, kus iga kaader peab olema õigesti eksponeeritud. Tagapõhi lülitub märkamatult tagasi reaktiivsele automaatsekspositsioonile (AE), kui DLS-näidud on aegunud/puuduvad või kalibreerimispaketti pole laaditud. |
| **Sileerimine (α)** | 0,05–1,0, sammuga 0,05 | 0,3 | Järgnevate ennustavate lahenduste sileerimine (madalam väärtus = sujuvam). |
| **Stseeni peegeldusvõime**|**Kalibreeri ρ uuesti** nupp | — | Hinnab uuesti stseeni peegeldusvõime tegurit, mida lahendaja kasutab. |

{% hint style="info" %}
**Massiivi ühendamine lülitab ennustava AE vaikimisi välja** — massiivide puhul reguleerivad valgustust (koos küllastumiskaitsega) Chloros

i nutikas AE ja kaamera poolne automaatne valgustuse reguleerimine, ning ennustava AE ühe stseenipeegeldusvõime hinnang ei ole segastseenide puhul usaldusväärne. Saate selle siin kaamera kaupa uuesti sisse lülitada, kui soovite konkreetselt DLS-põhist radiomeetrilist valgustust.
{% endhint %}

**DAQ-põhine valgustuse ülempiir ja langeva valguse järgi fikseeritud AE.** Sõltumata eespool olevast valikukastist, kui DAQ-valgusandur on määratud RGB

-kaamerale, arvutab Chloros

— mõõdetud absoluutse allapoole suunatud kiirguse alusel — maksimaalse ekspositsiooni×võimenduse, mille juures 100 %-peegeldusega pind jääb allapoole kärpimist, ja rakendab seda **ülempiirina**automaatse ekspositsiooni puhul. Alates versioonist 1.3.0 kehtib ülempiir ainult siis, kui kalibreerimismärk on selle kaamera vaateväljas (nähtav kaadri piires viimase 10 sekundi jooksul); kui sihtmärki vaateväljas pole, töötab selle asemel kaamera tavaline automaatne säritus koos oma heledate alade kaitsega, nii et ülejäänud stseeni ei hoita vajalikust tumedamana. Kui ülempiir on aktiivne, töötab kaamera**langeva valguse režiimis**: see töötab avatud ahelas langeva valguse mõõdetud ekspositsiooniga, võimendusega 0 dB — ekspositsioon järgib mõõdetud valgust, mitte stseeni sisu. Kuna ülempiir võib ekspositsiooni ainult lühendada, ei saa see ise tekitada ülevalgustust. Alates versioonist 1.3.0 arvutatakse ülepiir kaamera enda ekspositsioonikonstandi alusel, mis pärineb kalibreerimispakist (kui pakis seda väärtust pole, kasutatakse sõidukipargi keskmist väärtust). Ülepiir lülitub automaatselt välja – ja taastub tavaline stseeni automaatne ekspositsioon – kui kalibreerimissihik ei ole vaateväljas, DAQ-näidud puuduvad, on aegunud (&gt;30 s), tume või kui ≥15 % kaadrist lõikub fikseeritud säriajal (mis tähendab, et andur ja kaamera näevad erinevat valgustust). Graafilises kasutajaliideses puudub lüliti; see on standardne käitumine, kui RGB

-kaameral on DAQ seotud.

### Andmete kogumine ja käivitamine

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Massiivi liikmetel kuvatakse lisaks ainult lugemiseks mõeldud **Role**(Master sinisel / Slave rohelisel),**Sync Line**ja**Peers** read.

| Kontroll | Valikud | Vaikimisi | Märkused |
| --- | --- | --- | --- |
| **Triggerirežiim** | Väljas / Sisse | Sisse | Massiivi liikmetele keelatud (massiiv haldab triggereid). |
| **Trigger Source** | Software / Line0 (M8) / Line1 / Line2 | Line0 | Peidetud, kui Trigger Mode on Off; massiivi liikmetele keelatud. Line0 on M8 optiliselt isoleeritud väline triggerväljund. |
| **Frame Rate**| Auto / Manual + väärtus | Auto |**Auto**: kaamera kaadrisageduse piirang on välja lülitatud — kaadrisagedust määrab säriaeg ning kastis kuvatakse tegelik reaalajas kaadrisagedus.**Käsitsi**: saad piirata kaadrisagedust liuguriga (1 kuni ribalaiusega piiratud maksimumini), lähtudes praegusest tegelikust sagedusest. Massiivi liikmetele kuvatakse ainult lugemiseks mõeldud „N kaadrit sekundis (reaalajas)“ koos märkega „Määratud massiivi seadetes“. |

### Võrk / Andmeedastus

| Rida | Käitumine |
| --- | --- |
| **Paketi suurus**| 1500 (standard) / 9000 (jumbo) — vaikimisi**jumbo**. |
| **Läbilaskevõime** | Ainult lugemiseks mõeldud ühenduse läbilaskevõime piirang MB/s. Tagapõhi jaotab selle ümber kõigi ühendatud kaamerate vahel iga ühenduse loomise või katkestamise korral. |
| **Puhvri haldamine** | Ainult lugemiseks mõeldud puhvri haldamise režiim. |

### Salvestamine

Paneeli lõpus asub nupp **„Ava salvestusseaded…“**, mis viib [salvestusseadete paneelile](capture.md#the-capture-settings-pane) (on keelatud, kuni projekt on avatud — „Loo või ava projekt, et salvestada salvestusi”). Kui kaamera on peidetud või peatatud, tuletab vihje meelde, et enne salvestamist tuleb kaamera nähtavaks teha või salvestamine jätkata.

## Massiivi seaded

Ava seda **hammasratta**ikooniga ARRAY-real. Pealkiri: massiivi nimi koos ümbernimetamise pliiatsiga ja**×** sulgemiseks. Allpool *ainult kombineeritud* märgitud osad kuvatakse ainult kombineeritud kuvamisrežiimis ühendatud massiivide puhul.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Sünkroniseerimine

Ainult lugemiseks mõeldud **Master**,**Slaves**ja**Sync Line** read.

### Ümbritseva valguse andur

Näidatakse nii kombineeritud kui ka eraldi massiivide puhul:

* **Valguse andur** rippmenüü — seob ühe DAQ-i kogu massiiviga. Valik jõustub kohe, levib iga liikmekamera enda valgussensori rippmenüüsse (saad ikkagi iga kaamera puhul eraldi seadeid muuta) ning hakkab spektreid massiivi edastama.
* Reaalajas **Staatus**-rida: Väljas · „Oodatakse esimest spektrit…“ · „Aktiivne — kõik massiivi kaamerad on valgustuse suhtes korrigeeritud“ · „Viimase 3 sekundi jooksul pole uut spektrit — kasutatakse endiselt viimast näitu (aegumist ei ole)…“.
* Märkus paneelis: „Kogu massiivi hõlmav radiomeetriline korrigeerimine. Iga kaamera eraldi seaded tühistavad selle.”

### Kalibreerimissihik

Omaette rühm vahetult pärast ümbritseva valguse andurit, nii kombineeritud kui ka eraldi massiivide puhul:

* **Luba** valikukast — „MAPIR

i ArUco-sihiku tuvastamine ja NDVI

-i valideerimine paneeli peegeldusastme LUT-i suhtes”; määrab kombineeritud plaadi sihtmärgi ülekatte ja valideerimistabelid. **Eraldi** massiivi puhul lülitab see sisse iga liikme oma [kalibreerimissihmärgi tabeli](#the-calibration-target-table) (üks iga kaamera rea kohta loendivaates).

### Pildistamine — ühtsed anduriseaded

Näidatakse nii kombineeritud kui ka eraldi massiivide puhul (rühma pealkiri on „Pildistamine (ühtne kõigi kaamerate jaoks)”). Need seaded kehtivad ühtlaselt igale liikmele (liikmekohased muudatused rikuksid sünkroniseeritust). Muudatused salvestatakse vahepealsesse mällu ja rakendatakse koos nupuga **Rakenda**— välja arvatud lüliti**ROI sisse / ROI välja**, mis rakendub kohe.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Juhtelement | Valikud / vahemik | Mida see teeb |
| --- | --- | --- |
| **Piksli formaat** | 8-bitine / 10-bitine / 12-bitine / 16-bitine | Ühtne bittisügavus kõigile liikmetele; iga kaamera võtab selle sügavuse formaadi kasutusele oma anduri jaoks. |
| **Binning** | 1x1 / 2x2 / 4x4 | Riistvaraline binning — säilitab täieliku vaatevälja, suurendades samal ajal signaali-müra suhet (SNR) ja kaadrisagedust. Selle muutmine nullib ROI-väljad uue täisvaatevälja järgi. |
| **Eraldusvõime** eelseadistus | Täis / Pool / Veerand | Sõltub binningust; täidab ROI-väljad tsentreeritud kärpimisega. |
| **ROI-lõik (px)**|**ROI sisse / ROI välja**lüliti,**Valige**nupp, W / H / X / Y väljad, täissensori kaart | Sensori lõik — vaata [ROI-redaktorit](#the-roi-editor). Laius/kõrgus joonduvad 16-kordsetele (minimaalselt 64); nihked joonduvad 4 kordsetele. „max WxH“ vihje näitab ülempiiri ja**Reset** taastab väljad täisvaateväljale. Redigeerimise ajal jälgib oranž kärpimisruut iga väljade muutust kaardil ja maatriksi plaadil. |
| **Trigger Rate**| Lüliti „Auto“ / „Manual“ + fps 0,5–10, samm 0,5 |**Auto**(vaikimisi): tagapõhi arvutab käivitussageduse resolutsiooni ja ribalaiuse põhjal — sisestus on keelatud ja kuvab arvutatud väärtust.**Käsitsi**: kinnitab sinu väärtuse nupule „Kohalda” vajutamisel. Kinnitatud sagedus, mida uus kaadri suurus ei suuda edastada, hoitakse selle suuruse poolt lubatud kiireimal tasemel ning taastub, kui kaadri suurus seda uuesti võimaldab. |

Märkus paneelis: „Formaadi/lahutusvõime muutmine taaskäivitab kõik kaamerad lühiajaliselt. Triggerimissagedus rakendub reaalajas.“ Nupud **Kohalda / Tühista** asuvad paneeli allosas.

#### ROI-redaktor

Uus versioonis 1.3.0. ROI-lõikamisrida ja selle all olev kaart töötavad koos:

* **ROI sisse / ROI välja**— lülitab anduri lõikamise**kohe**sisse või välja (kaamerad taaskäivituvad lühiajaliselt; maatriksi ruudul kuvatakse „Seadete rakendamine…“).**Väljas**loeb kogu andurit ja**jätab su kärpimise meelde**— väljad näitavad seda jätkuvalt;**Sisse** rakendab seda uuesti. ROI sisselülitamine ilma kärpimise seadistamiseta rakendab keskele paigutatud poole suurusega kärpimist („Kärpimist ei seadistatud, seega rakendati keskele paigutatud poole suurusega kärpimist. Reguleeri seda väljade abil.“).
* **Valige**— joonistab kärpimise kaardile: lohistage üle selle või klõpsake kaht vastandlikku nurka (Esc tühistab). Ristkülik salvestatakse väljadesse W/H/X/Y, joondub eespool kirjeldatud sammude järgi; midagi ei rakendata enne, kui vajutad**Rakenda**, seega saad seda esmalt väljade nooltega täpsustada. Iga**Valige** vajutuse kohta üks ristkülik.
* **Kaart**näitab kärpimist kogu anduri ulatuses, seega saad seda tagasi laiendada: „Kogu andur, reaalajas“, kui kaamerad loevad kogu andurit, või „Täiskaadri hetktõmmis kell {aeg}. Kaamerad loevad ainult kärbitud osa“, kui need on kärbitud. Kui andur on alati edastanud ainult kärbitud pilti, kuvatakse kaardil „Täiskaadri vaadet pole veel: see andur on edastanud ainult kärbitud pilti.” koos nupuga**Näita täisraam**, mis lülitab huvipiirkonna välja. Lõigatud ala silt `W×H @ X,Y` asub kaardil ja reaalajas ruudul kasti väljaspool (selle kohal või all, kui kast puudutab ülaosa).
* **Kohalda**kinnitab väljad: kärbitud pilt lülitab ROI sisse; täisraam (pärast**Taasta**) jätab selle välja lülitatuks, kusjuures eelmine kärbitud pilt jääb meelde; välja lülitatud ROI, mille väljad näitavad endiselt meelde jäänud kärbitud pilti, jääb välja lülitatuks, kui kohaldad mõnda muud muudatust (formaat, binning, sagedus).
* **Joondatud kaamerarivis paigutatakse iga kaamera kärpimisaken joondamise abil**, nii et iga kaamera raamib sama stseeni osa. Enne joondamist kärbitud kaamerarivi joondatakse esmalt täisanduri järgi (kärpimine tühistatakse, joondamine lahendatakse ja kärpimine taastatakse kaamerapõhiste akendega).
* Pärast kärpimise muutmist kohandub automaatne säritus uue kadreeringuga. Kärpimise ajal võib liikme säritus ikkagi kesta kuni käivitamisperioodi lõpuni.
* ROI sisse-/väljalülitatud olek ja salvestatud kärpimine salvestatakse koos projektiga.

### Joondamine (koosregistreerimine)

Näidatakse **nii kombineeritud kui ka eraldi massiivide** puhul (versioonis 1.2.0 näidati seda ainult kombineeritud massiivide puhul); rühma pealkiri on „Alignment”.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Jääk**-pill: „RMS x.xx px“ — roheline alla 1 px, kollane alla 3 px, punane muudel juhtudel või kui mõni kaamera ebaõnnestus; „no profile“ enne esimest lahendamist.
* Kokkuvõteliin: „✓ Kõik kaamerad joondatud (N)“ / „⚠ p/N kaamerat joondatud —  <serial (filter)="">ebaõnnestus“ / „Kärpimine aktiivne — kalibreeri uuesti joondamiseks (kasutab kogu andurit)“ / „Oodatakse, kuni valgustatus stabiliseerub…” / „Seadete rakendamine…” (kaamerad taaskäivituvad) / „Kaamerarivi lõpetas kaadrite edastamise — ühendage kaamerarivi uuesti”.
* Kaamerapõhine tabel: kaamera (seerianumbri viimased 4 numbrit + filter), ümberprojitseerimisviga pikslites koos kokkulangevuste arvuga („ref“ peakaamera puhul) ning kattuvuse normaliseeritud ristkorrelatsiooni skoor võrreldes 0,35 lävendiga.
* **Kalibreeri joondus uuesti** nupp (näitab enne esimest profiili „Kalibreeri joondamine”) — käivitab ko-registreerimise uute kaadritega uuesti.
* **„Kaamerate automaatne eksponeerimine joondamiseks”** valikukast (vaikimisi märgitud) — heledab ajutiselt tumedaid või tasaseid kaameraid (esmalt ekspositsioon, seejärel võimendus), et neil oleks tekstuur, mida sobitada, ning taastab seejärel automaatse ekspositsiooni.

Chloros joondab iga massiivi automaatselt pärast ühendamist mõlemas kuvamisrežiimis, kui liikmete automaatne säritus on stabiliseerunud (proovib seda kuni 5 minutit); ka kombineeritud eelvaade joondub avamisel. Eraldi massiivil puudub komposiitpilt, seega kontrollib see osa joondamise seisundit ja käivitab ise funktsiooni „Recalibrate”. Kalibreeri uuesti, kui fookus või stseeni sügavus on muutunud. Joondamine on **disainilt ainult seansipõhine** — seda ei salvestata kunagi profiili, kuna see sõltub hetke stseeni kaugusest. Pildistatud materjali saab siiski eksportida pikseljoondatuna (vt [Joondatud ekspordid](capture.md#per-array-controls)).

### Nutikas vinjet

* **Luba korrigeerimine**valikukast – rakendab kaamerapõhist vinjeti hinnangut radiomeetrilisele ahelale (reaalajas**ja** eksportides).
* **Kalibreeri praeguse vaate järgi**— suuna maatriks esmalt ühtlasele sihtmärgile (lame ekraan, sein või taevas); iga kaamera tasandatakse eraldi ja staatus näitab tasanduse kasvu „n/N kaamerat · −x,x %“.**Tühista** eemaldab hinnangu.
* Staatus: „Kalibreeritud”, „Kalibreerimata” või „Osaliselt kalibreeritud — korda” juhul, kui vaid mõnede kaamerate hinnangud kehtivad endiselt nende praeguses kadreeringus (näiteks pärast seda, kui funktsioon „Kalibreeri uuesti” nihutas mõne kaamera kadreerimisakent) — käivita kalibreerimine uuesti.
* Täpsustage iga kaamera puhul eraldi kaamera-põhise **Vignette** liuguriga [Live Preview](#live-preview) aknas.

### Reaalajas eelvaade

* **Histogrammi**valikukast + laiuse liugur (0,10–0,90 kaadrist) — reaalajas voo histogrammiriba kõigi liikmete ruutudel korraga (sh kanalite jagatud ruudud), kogu massiivi jaoks määratud kaamera-põhine [histogrammi ülekatte](#display-overlays-drawn-over-the-live-feed) kogu massiivi jaoks. See kuvatakse sisse lülitatuna, kui igal liikmel on see juba olemas.**Eraldi** massiivis on see rühma ainus rida.
* *Ainult kombineeritud:* **Indeks**: märkeruut „Luba“ + hammasratas — avab jagatud [indeksi kalkulaatori](#index-calculator-pane), kus on joonistatud**kõigi** liikmete kaamerate vahemikud. Allpool asuv väljendi eelvaate rida näitab praegust väljendit („Väljendit pole määratud — ava kalkulaator, et seda luua“), mida värskendatakse iga sekundi järel.
* *Ainult kombineeritud režiimis:* **Renderdamise resolutsioon**rippmenüü (samad eelseadistused nagu kaamera kaupa, vaikimisi 720p): reaalajavaate voo kõrgus**ja** salvestatud komposiitpildi ekspordisuurus. Märkus paneelis: „Eelvaade + salvestatud komposiitpildi suurus. Kaamera kaupa pildid eksporditakse alati täisresolutsioonis.“

### Kuvamiskihid *(ainult kombineeritud režiimis)** **Luba** valikukast (vaikimisi välja lülitatud — peakaamera kuvatakse otse; sisse lülitatuna = kihiline komposiit).
* **Esiplaan**/**Tagaplaan**rippmenüüd: iga liikmekamera (nime järgi) või**Indeks**. Kui esiplaaniks on valitud „Indeks“, kuvatakse LUT-i miinimum- ja maksimumpiiridest väljaspool asuvad pikslid tagaplaani kihina.

### Jagatud vaade *(ainult kombineeritud)*

**„Näita liikmekameraid“**— nupp**Liikmekaamerate jagamine / peitmine**, mis lisab iga liikme oma reaalajas pildi eraldi ruudustikuna komposiidi kõrvale. Ruudud loevad massiivi olemasolevat kaadripuhvrit (ei vaja täiendavat kaameraühendust). Ainult ruudustikvaade; salvestatakse projekti raames iga massiivi kohta eraldi.

### Võimalused

Iga 5 sekundi järel uuendatav ainult lugemiseks mõeldud paneel:

* **Taseme märge**: „Samaaegne salvestamine“ (roheline) · „Samaaegne salvestamine (FTD-järkjärguline väljastamine)“ (roheline) · „Järkjärguline salvestamine (100 ms nihke)“ (kollane) · „Konfiguratsioon liiga suur“ (punane).
* **Kaadri tervis**: „x,xx % puudulik“ — roheline alla 1 %, kollane alla 5 %, punane 5 % või rohkem.
* ****Ühendusliin**: „NIC {mbps} Mbps – püsiv {MB/s} MB/s”.

See on massiivi reaalajas ribalaiuse eelarve. Aluseks oleva kaadrisageduse (fps) ja võrgumudeli kohta — ning selle kohta, mida muuta, kui tase muutub kollaseks või punaseks — vaata [Mitme kaameraga massiivid](arrays.md) ja [CLI-viite](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Indeksi kalkulaatori paneel

Kolmas külgriba lehekülg, mida jagavad nii kaamera-põhine indeksihammasratas kui ka kombineeritud massiivi indeksihammasratas (korraga üks – pealkiri on „Indeksi kalkulaator — <camera name="">“ või „Indeksi kalkulaator — <array name="">“). See võtab vastu ribade loendi (kaamera looduslikud filtriribad-loomulikud sagedusalad või kõik sagedusalad kogu massiivi liikmete ulatuses), kehtiva väljendi ja LUT-i konfiguratsiooni (sisse/välja, tase – vaikimisi 7-stop, min – vaikimisi 0,2, max – vaikimisi 1), lisaks reaalajas indeksihistogrammi. Nupp **Rakenda** kinnitab väljendi; LUT-i muudatused rakenduvad eelvaates reaalajas.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Kaamerapõhised vs massiivi poolt hallatavad seaded

Kiirviide selle kohta, mis asub kus, kui kaamera on massiivi liige:

| Massiivi poolt hallatav (kaamera paneelis ainult lugemiseks) | Ikka kaamerapõhine massiivi sees |
| --- | --- |
| Pikseliformaat, resolutsioon, binning | Automaatne säritus (säritus, võimendus, sihtmärk, silumine, ROI) |
| Päästikurežiim/allikas, kaadrisagedus | Müra vähendamine, teravus, vignett |
| | Suund (peegeldus/pööre), ekraanikatted (massiivi seaded → reaalaja eelvaade seadistab histogrammi kõigil liikmetel korraga), punktmõõtur |
| | Indeks (eraldi kuvatavad massiivid), valgusanduri sidumine |

Muud üldised toimingud:

* **Kombineeritud vs eraldi kuvamine** valitakse massiivi ühendamisel: kombineeritud = üks joondatud komposiitpaneel (liikmed edastavad pilti ainult jagatud vaate kaudu); eraldi = iga liige renderdab oma sünkroniseeritud paneeli. Kaamera ei näita kunagi korraga nii eraldiseisvat pilti kui ka massiivi paneeli.
* ****Automaatne ühenduse taastamine**: salvestatud projekti avamisel taastatakse selle kaamerad ja massiivid ning kõik salvestatud seaded rakendatakse uuesti tagapõhjas enne voogude taastamist.
* **Salvestamise piiramine**: peidetud või peatatud kaamerad jäetakse funktsioonist „Capture All” välja; massiiv blokeeritakse täielikult ainult siis, kui KÕIK liikmed on peidetud/peatatud. Vaata [Salvestusseaded ja režiimid](capture.md).

## Kuidas seaded säilivad

Kaamera vahekaardi seisund salvestatakse **koos projektiga**, mitte brauseris:

* Iga reageeriv muudatus salvestab kaamerate ja massiivide hetkeseisu projekti `cameras.json` (500 ms viivitusega). See hõlmab kaamerate nimesid ja värve, säri/võimenduse/automaatse säri (AE) seadeid, pikseliformaati/lahutusvõimet/binningut, käivitussagedust, eelvaate seadeid (renderdamise lahutusvõime, müra eemaldamine, teravus, vinjet, värviprofiil, heledad alad, küllastus/kontrast), orientatsiooni, pealekatteid, kanalite jagamist, indeksikonfiguratsiooni, ennustava AE seadeid, AE ROI-d, massiivi nimed, kuvamisrežiim, massiivi salvestusseaded (sealhulgas ROI-kärpimine, kas ROI on sisse lülitatud ja salvestatud kärpimine, kui see on välja lülitatud), massiivi juhtmeeelarve ja võrgustikublokk (voo suum, vaatamisrežiim, võrgustiku lukustus, käsitsi määratud plaadijärjekord, peidetud kaamerad, suletud plaadid, aktiivne kaamera).
* Valgusandurite seosed salvestatakse projekti faili `sensors.json`.
* Projekti uuesti avamisel ühendatakse riistvara uuesti ja kõik seaded rakendatakse uuesti.
* **Projekt pole avatud = ainult seanss**: kui projekti pole, ei säili midagi pärast Chloros sulgemist.
* Ainult seansipõhine, sõltumata projektist: pausi olek, punktmõõturi proovid, kaamera-põhine kalibreerimise sihtmärgi valikukast (on alati vaikimisi välja lülitatud) ja maatriksi joondamisprofiil (arvutatakse disaini kohaselt iga seansi jaoks uuesti).
* Üks erand: **Pildistamisseaded** ekspordivalikud ja pildistamisrežiim säilivad projekti kaupa kohalikus rakenduse mälus, mitte `cameras.json` — vt [Pildistamisseaded ja režiimid](capture.md).</array></camera></serial>
