# Mitme kaamera süsteemid

LATTICE **süsteem**koosneb kahest või enamast LATTICE-kaamerast, mis on ühendatud üheks sünkroniseeritud tervikuks. Üks kaamera on**peakaamera**: see saadab riistvaralise GPIO-triggerimpulsi ühisele sünkroniseerimisliinile (vaikimisi**Line2**), nii et kõik kaamerad pildistavad sama hetke. Chloros lisab PTP-ajasünkroniseerimise, reaalajas eelvaate (kaamerate kaupa jaotatud ruudud või üks ühtlustatud mitme lainepikkusega komposiit) ning sünkroniseeritud pildistamise – iga pildistamistsükkel toodab ühe**kaadrirühma**, milles kõik kaamerad jagavad sama ajamärget ja kaadri ID-d (näidatakse pildistamisväljundis kui `fid:N`).

Massiivid on viis, kuidas monokamerad (M3M) loovad taimestikuindekseid – üks kaamera annab ühe laineala ja massiiv joondab need mitme laineala pildistesse. Vaata [Monokamerad ja taimestikuindeksid](mono-indices.md).

Massiivi ühendamiseks on kolm võrdväärset viisi ja neil kõigil käivitatakse sama „smart-prep“ töövoog:

| Pind | Sisenemispunkt |
| --- | --- |
| Graafiline kasutajaliides | Vahekaart „Kaamerad“ → **Ühenda massiiv** (sinine nupp) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (esimene seerianumber = peakaamera) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (esimene seerianumber = peaseade) |

Smart-prep teostab järgmises järjekorras: võrguvõimekuse kontroll (ICMP DF ping + GVSP-kontroll), sünkroniseerimistaseme valik, raami suuruse automaatne vähendamine ühendusliini sobivaks, PTP aktiveerimise, kaamera-põhise pikseliformaadi automaatse valiku, automaatse säri algväärtuse määramise iga kaamera salvestatud seisundi põhjal ning GPIO-trigeri konfigureerimise Line2-l.

{% hint style="info" %}
Enne kui miski sellest toimib, peavad kaamerad olema ühenduses kättesaadavad – vaata [Kaamerate ühendamine](connecting.md) seadmete avastamise, aadressimise ja esmakordse ühenduse kalibreerimise allalaadimise kohta. Mitme kaameraga seadmetel on võrguühenduse kiirusega sama olulised ka host-võrgukaardi vastuvõturingi seaded; täielik sümptomite ja lahenduste tabel asub [CLI viite § Host-võrgukaardi seadistamine ja häälestamine](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialoogaken „Array Connect“

Vahekaardil „Cameras“ → **Connect Array**avaneb kolmeastmeline viisard:**Valik → Kuvamisrežiim → Seaded**.

### 1. samm — Valige juhtkaamera ja allkaamerad

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

Dialoogikast skannib võrku kohe pärast avamist („Võrgu skannimine...“), seejärel kontrollib GPIO-käivituse juhtmestikku („GPIO-juhtmestiku kontrollimine...“). Massiivi loomiseks on vaja vähemalt **2 kaamerat**.

Juhtmestiku kontroll täidab rollide valiku eeltäitmise, kui see on võimalik, ja kuvab ühe kolmest teadaandest:

| Teadaanne | Tähendus |
| --- | --- |
| „GPIO peakaamera tuvastatud — valikud eeltäidetud” (roheline) | Test leidis päästiku topoloogia; peakaamera ja allkaamerate valikukastid on juba märgitud. |
| „Peakaamerat ei tuvastatud – kontrolli GPIO-kaablit” (oranž) | Ükski kaamera ei tuvastanud päästiku impulssi; kontrolli sünkroniseerimiskaableid. Rolle saab siiski ka käsitsi valida. |
| „Sünkroniseerimiskaabel puudub: {seerianumbrid}“ (oranž) | Loetletud kaameratel pole sünkroniseerimiskaablit ühendatud. |

Kaamerate tabelis on veerud **Kaamera / Seerianumber / IP / Master (raadio) / Slave (valikukast)**:

* Valige täpselt **üks master**ja**üks või mitu slave’i**. Praeguse masteri raadionupule uuesti klõpsates tühistatakse valik.
* Kaamerat, mille juures on märge **„Sünkroniseerimiskaabel puudub“**, ei saa kunagi valida slave’iks — slave, millel puudub triggerahel, ootaks sünkroniseerimisliinil lõputult ja edastaks tühja pilti. Ühendage see kaamera selle asemel iseseisva kaamerana.
* Juba iseseisvalt ühendatud kaamerad *ei* lülitu välja: massiivi ühendamine vabastab iseseisva seansi ja avab kaamera uuesti massiivi sees.

**Järgmine: kuvarežiim →**aktiveerub, kui on valitud üks peakaamera ja vähemalt üks orjakaamera.**Uuesti skaneerima** käivitab uuesti seadmete avastamise ja ühenduste kontrolli.

{% hint style="warning" %}
**Tühista** on keelatud, kui skaneerimine või juhtme kontroll on käimas — tühistamine kontrolli keskel võib põhjustada kaamera SDK kokkujooksmise LATTICE kaamera püsivara puhul. Oodake, kuni pöörlev ikoon lõpetab töötamise.
{% endhint %}

### 2. samm — Kuvamisrežiim

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Režiim | Mida saate |
| --- | --- |
| **Eraldi kaamerad** | Üks reaalajas ruut iga kaamera kohta, kõik käivitatakse üheaegselt, et kaadrid püsiksid sünkroonis. Iga kaamera säilitab oma värvi ja seaded. |
| **Kombineeritud kaamerad** *(vaikimisi)* | Üks ruut, mis kuvab joondatud mitme sagedusriba NDVI/index komposiiti. Kaamerad jagavad massiivi värvi. |

Kuvarežiim muudab ainult reaalajas eelvaate esitust — salvestamise käitumine on mõlemal juhul sama.

### 3. samm — Maatriksi seaded ja prognoositav tulemus

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Sellesse stseeni sisenemisel küsib Chloros tagapõhjalt **soovitust**ja rakendab automaatselt ROI + binningu kombinatsiooni, mis sobib teie NIC-i vastuvõturingiga (see eelistab binningut ROI kärpimisele, kuna binning säilitab kogu vaatevälja). Iga teie tehtud muudatus käivitab analüüsi uuesti reaalajas ja uuendab parempoolset**Prognoositav tulemus** paneeli.

Vasak veerg — seaded:

| Kontroll | Valikud | Vaikimisi | Märkused |
| --- | --- | --- | --- |
| **ROI (vaateväli)** | Täis (2048×1536) / Pool (1024×768) / Veerand (512×384) | Täis | Anduri kärpimine: poole või veerandi kärpimine väiksemaks piirkonnaks algse pikselisuurusega. |
| **Piksli ühendamine** | 1× / 2× (summa 2×2) / 4× (summa 4×4) | 1× | Riistvaraline piksli ühendamine: 2×2 = täisvaateväli veerandiga juhtme kulust; 4×4 = täisvaateväli 1/16 kuluga. Peidetud, kui kaamerad ei toeta binningut. |
| **Andmesidepoolsed pildid** (väljalugemine) | — | — | Pärast binningut tegelikult kaabli kaudu saadetud laius × kõrgus, ümardatud 16-kordseteks (minimaalselt 64). |
| **Pin-lahutusvõime**| valikukast | välja lülitatud | Chloros suurendab tavaliselt binningut automaatselt ühendamisel, kui prognoositav kiirus langeb alla**1,5 kaadrit sekundis**. Pinning säilitab valitud kaadri suuruse ja aktsepteerib madalamat kiirust — ning muudab ülekoormatud konfiguratsiooni automaatse kiiruse alandamise asemel ühenduse kindlaks keeldumiseks. |
| **Trigger Rate** | 0,5–60 fps, samm 0,1 | tühi = auto | Masteri triggeri tulistamissagedus. Jäta tühjaks, et Chloros saaks selle ise tuletada. |
| **Andmeside eelarve**| 20–2000 MB/s, samm 0,1 | tühi = auto | Kui palju host tegelikult suudab vastu võtta, MB/s –**see on number, millest sõltub kogu massiivi eraldamine.** Tuvastatakse automaatselt võrgukaardi abil. Vähendage seda, kui massiiv teatab vigastest raamidest: tuvastatud väärtus ülehinnab USB-adaptereid ja jagatud lüliteid. Selle muutmine käivitab prognoosi uuesti reaalajas. |

Parempoolne veerg — **Prognoositav tulemus**:

* **Sünkroniseerimistasand** — „Samaaegne salvestamine“ (roheline), „Samaaegne salvestamine (FTD-astmelise väljastamisega)“ (roheline), „Astmeline salvestamine (100 ms nihke)“ (kollane) või „Konfiguratsioon liiga suur“ (punane).
* **kaadrisageduse prognoos** — kuvatakse vahemikuna („hämar → ere”), kuna sünkroniseeritud massiivi kiirus sõltub aeglaseima kaamera säriajast.
* **NIC-liin** — ühenduse kiirus ja püsiv läbilaskevõime („NIC {mbps} Mbps · püsiv {N} MB/s”).
* **Sim-emit-purske kontroll** — kas hosti NIC-i vastuvõturõngas suudab vastu võtta ühe samaaegse purske kõikidelt kaameratelt („Sim-emit-purske: X MB · NIC-rõnga kasutatav maht: Y MB ✓/✗“).
* **Kaabli läbilaskevõime kontroll** — püsiseisundi koondnõudlus võrreldes kokkupõrkevaba kaabli ülempiiriga („Kaabli läbilaskevõime: {nõudlus} MB/s, mida nõuavad {n} kaamerat · ülempiir {ceiling} MB/s ✓/✗ ületatud”).
* **„Maksimaalne kaamerate arv sellel liinil: {n} — määratud kaamera kohta kehtiva ribalaiuse alampiiriga, seega kaamerate koondamine seda ei suurenda.”** — kuvatakse, kui olete lähedal (või ületanud) kaamerate arvu ülempiiri.
* **„Nende seadete juures KAOB KAADREID.“**— punane hoiatus koos tagapõhja põhjendusega, lisaks takistuste nimekiri ja sinised**parandusettepanekud** („Selle massiivi võrku mahutamiseks“ / „Samaaegse salvestamise avamiseks“).**Rakenda ja ühenda** on blokeeritud, kuni prognoos on olemas, ning selle silt näitab, miks see keeldub:

| Nupu silt | Tähendus | Mis tegelikult aitab |
| --- | --- | --- |
| „Analüüsitakse...” | Analüüs on veel käimas. | Oota. |
| **„Selle võrgu jaoks on liiga palju kaameraid“**| Kaamerate mass ületab võrgu läbilaskevõime (koondkontroll ebaõnnestus). | Vähem kaameraid, läbivalt kasutatavad jumbo-raamid või kiirem võrgukaart.**Väiksem ROI EI AITA** — vt allpool. |
| **„Vähenda ROI-d, et võimaldada&quot;** | Nende seadete juures langeksid raamid ära (burst/ring-kontroll ebaõnnestus). | Vähenda ROI-d, suurenda binningut või paranda võrgukaardi vastuvõturingi. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Ühendamise ajal võib ilmuda roheline **kalibreerimise allalaadimise paneel** koos seerianumbri kaupa edastatava edenemisribaga: kui kaamera ühendatakse seadmega esimest korda, laadib Chloros GigE kaudu kaamerast alla umbes 3,8 MB suuruse tehase kalibreerimispaketi (umbes 70 sekundit kaamera kohta). Vahemällu salvestatud kaamerad ei näita seda paneeli kunagi. Vaata [Kaamerate ühendamine](connecting.md).

## Ribalaius: kui palju kaameraid mahub

See, mida massiiv suudab edastada, sõltub ühendusliini omadustest, mitte Chloros-ist, seega leiab planeerimisandmed riistvara kasutusjuhendist: **[Massiivi ribalaiuse planeerimine](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Mida Chloros nendega teeb: ühendusdialoog käivitab võrguproovi, prognoosib saavutatavat kaadrisagedust ja valib sobiva taseme. Kui massiiv koormab kaablit liiga palju, keeldub see ühendusest, selle asemel et pakette vaikselt maha visata — vaata eespool kirjeldatud prognoositavate tulemuste paneeli.

## Kui kaadrid kaovad

Kaamera võib avaldatud rühmast puududa kahel täiesti erineval põhjusel,
mille lahendused on vastandlikud. Chloros loendab neid eraldi, selle asemel et näidata üht
„ebatäielikku” arvu, mis ei täpsusta kumbagi põhjust:

| Mis juhtus | Mida see tähendab | Kus otsida |
| --- | --- | --- |
| **Rikutud**— kaadrid jõudsid kohale, kuid olid struktuuriliselt vigased | GVSP-pakettide kadu võrguteel |**Ühendusvõrgu läbilaskevõime**, võrgukaardi vastuvõturing, jumbo-kaadrid, lüliti |
| **Ei jõudnud kunagi kohale**— raami ei tulnud üldse | Kaamera ei käivitatud või sellest ei väljunud midagi |**M8 sünkroniseerimiskaabel**, sünkroniseerimisliin, kas kõik seadmed on sisse lülitatud |

Jaotust hinnatakse uuesti iga 10 sekundi järel, kui kaamerarida edastab andmeid. Kui see ületab 5 %,
logitakse see mõlema numbri nimetamisega ning iga vigane puhver registreeritakse esmakordsel
esinemisel kaamera kohta, seejärel koondatakse andmed kord minutis, et pikk seanss jääks loetavaks.

**Rikutud kaadrid, mille puhul „ei jõudnud kohale“ väärtus on null, tähendavad, et päästik ja kaabli sünkroniseerimine toimivad ideaalselt**ning iga kadunud kaader on võrguteel. Lahenduseks on**Wire Budget** väärtuse alandamine ja
uuesti ühendamine.

{% hint style="warning" %}
**Triggerimäära alandamine ei aita vigaste kaadrite puhul.** Kaamera pakettide
edastamissagedus määratakse kindlaks ühenduse loomisel. Triggerimäära alandamine muudab seda, kui tihti pursk
toimub, mitte seda, kui kiiresti pursk ise kaablisse jõuab. Mõõdetud 4-kaameralisel seadmel
triggerimäära vähendamine 5 korda ei muutnud midagi, samas kui andmeedastusmahtu vähendamine 240-lt
200 MB/s-ni viis sama süsteemi vigaste raamide määra 10,4 %-lt nullini.
{% endhint %}

Juba töötav massiiv ei saa ennast ümber planeerida – katkesta ühendus ja ühenda uuesti, et ühendusaja
valija saaks töötada uue andmeedastusmahtu arvestades.

### USB-võrgukaartide kiirus on piiratud 200 MB/s-ga

USB-Ethernet-adapter näitab oma *Etherneti* ühenduskiirust, kuid seda, mida ta tegelikult
suudab säilitada, piirab USB-siin ja selle draiver. USB 10GbE-donglile omistati varem
läbilaskevõimet umbes 1000 MB/s – number, mida keegi kunagi mõõtnud polnud – ja nelja kaamera
nelja kaamera töötamine selle fantoomse varu piires rikkus 6–18 % kaadritest, samal ajal kui süsteem
näitas endiselt normaalset sihtkaadrisagedust. USB-ühendusega adapterite maksimaalne kiirus on nüüd piiratud
**200 MB/s**. Piirang on absoluutne, mitte protsentuaalne, sest piiriks on
siin buss: USB 1 GbE-adapter saavutab umbes 80 MB/s ja seda piirangut see ei mõjuta.

Kui teie host on tegelikult kiirem kui piirang, suurendage **Wire Budget**-i vastavalt.

## PTP-aja sünkroniseerimine

Kaadrite *sünkroniseerimine* toimub riistvaralise trigeri abil; **PTP** (IEEE 1588 PTPv2) tagab kõigil seadmetel võrreldavad *ajamärgised*. See on vaikimisi sisse lülitatud massiivi ühendamisel:

* **Chloros hosti backend töötab PTP grandmasterina**. LATTICE kaamerad ja DAQ-E valgusandurid on selle alluvuses domeenis 0, mistõttu pildi ajamärgised ja DAQ spektrid langevad ühele kellale (~1 ms).
* `--no-ptp` (CLI) lülitab selle laboritööde jaoks välja — sel juhul **ei** ole kaameratevahelised ajamärgid võrreldavad.
* Sünkroniseerimise seisundi kontrollimiseks kasutage CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Vahekaardil „Kaamerad” endal puudub PTP-indikaator; seal kuvatakse kaamerate kaupa sünkroniseerimise andmed, milleks on ainult lugemiseks mõeldud **roll**(Master/Slave),**sünkroniseerimisliin** ja massiivi võimekuse tase. DAQ-E PTP-olek kuvatakse vahekaardi „Valgusandurid” andurite üksikasjades.

## Massiivi reaalaja vaade

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

Peamine vooala pakub kahte paigutust (lülitamine ülemises ribas): **võrgustikuvaade**(iga ruut on üks lahter; lohistades saab neid ümber järjestada, kui võrgustiku lukk on avatud) ja**nimekirjavaade**(massiivid täislaiuses ülaosas, üks aktiivne kaamera allpool).**Feed Zoom** liuguriga saab ruutude suurust muuta; kui lahtri laius on alla 200 px, peituvad nime ja kaadrisageduse (fps) ülekatted automaatselt.**Eraldi režiim** näitab ühte ruutu kaamera kohta. Igal ruudul on järgmised ülekatted:

* kaamera nimi (ülemises vasakus nurgas),
* **kaadrisageduse näidik** (ülemises paremas nurgas) — see on tagapõhja poolt edastatud kaamera *tegelik kaadrisagedus*, mitte eelvaate küsitlussagedus (reaalajas eelvaade on piiratud 30 kaadriga sekundis, sõltumata kaadrisagedusest),
* staatuse punkt — roheline (ülekanne käimas) / kollane (laadimine) / punane (viga),
* **vananenud kaadri pöörlev ikoon**, kui 2 sekundi jooksul pole saabunud ühtegi uut kaadrit — see on normaalne umbes 5 sekundi jooksul pärast ühenduse loomist või katkestamist, kui tagapõhi tasakaalustab kaamerate vahelist andmeedastusmahtu.**Kombineeritud režiim**näitab ühte koondpildi: tagapõhi teostab debayeringut, skaalab, joondab, eemaldab müra, muudab sagedusribapõhiseks kiirguseks (pluss DLS-peegeldusvõime, kui valgussensor on seotud), hindab massiivi indeksaväljendit, rakendab LUT-i ja edastab tulemuse MJPEG-vormingus. Kuni esimese joondatud kaadri renderdamiseni näitab ruut oma olekut: „Massiivi ettevalmistamine…“, „Kalibreerib joondamist…“, „Ootab esimest kaadrit…“ või – kui automaatse joondamise korduskatse aeg (~30 s) on otsa saanud – „Joondamine vajalik“ koos nupuga**Kalibreeri joondamine**.

Kasulikud faktid kombineeritud režiimi kohta:

* Komposiit on registreeritud **peakaamera**kaadri järgi. AE-ROI suunamine ja punktmõõtmine komposiidil on peakaamera puhul täpsed ning allkaamerate puhul ligikaudsed; kasuta**Split View** (massiivi seaded → „Näita liikmekaameraid”), et saada pikselitäpsed kaamerapõhised plaadid ilma täiendavaid kaameraühendusi avamata.
* **Kihid kuvada**(massiivi seaded; vaikimisi välja lülitatud) võimaldab valida esi- ja taustakihti — mis tahes liikmekamera või**Indeks**. Kui esikiht = Indeks, kuvatakse LUTi miinimum-/maksimumväärtusest väljaspool olevad pikslid taustakihina.
* **Renderdamise resolutsioon** (vaikimisi 720p) määrab otseülekande kõrguse *ja* salvestatud komposiidi ekspordimõõtmed. Iga kaamera pildid eksporditakse alati täisresolutsioonis.
* Joondamine arvutatakse iga seansi kohta eraldi ja seda ei salvestata kunagi – vaadake massiivi seadete paneeli joondamise jaotist, et näha RMS-jääke ja nuppu „Recalibrate”.

## Salvestamine: jälgimine vs analüüs

Massiivi salvestuspinnad jagunevad selgelt **jälgimiskvaliteediks**(salvestatakse see, mida näed) ja**analüüsikvaliteediks** (salvestatakse toorandmed, kalibreeritakse hiljem):

| Töövoog | Tase | Mida salvestatakse | Kasutajaliides | CLI |
| --- | --- | --- | --- | --- |
| **Salvestamine**(pildid) | Analüüs | Üks sünkroniseeritud kaadrirühm iga läbimise kohta; iga kaamera failid igal valitud eksporditasemel (toored andmed/debayered/kiirgus/peegeldus/eelvaade/indeks) + `.daq` sidecar |**Nupp „Salvesta kõik”** + salvestusseaded | `lattice array-capture` |
| **Salvesta indeksvideo** | Jälgimine | Näidatud reaalajas kombineeritud indeks-komposiit — 8-bitine, eelvaate resolutsioon, sisseehitatud LUT; vajab avatud reaalajas voogu | ● Salvesta indeksvideo (kombineeritud massiivid) | `lattice array-record` |
| **Toores pildiseeria → video koostamine**| Analüüs | Toored andurikaadrid täieliku salvestuskiirusega + manifest + `.daq`, seejärel offline-rekonstrueerimine kalibreeritud kiirguse / peegelduse / indeksvideoks, ajaliselt sobitatud DAQ-näitudega | ⦿ Salvestada toores pildiseeria →**Koosta video** | `lattice array-burst` → `lattice array-build-video` |

Praktiline reegel: kui pikslid annavad *mõõtmistulemusi*, kasuta salvestamist või seeriat (analüüsitasand); kui soovid lihtsalt *vaadata või demonstreerida*, mida kaamera nägi, salvestage indeksvideo (seiretasand).

### Salvestusseaded (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

**Capture All** kõrval asuv hammasratas avab salvestusseadete paneeli (nõuab avatud projekti – salvestised salvestatakse sinna):

* **Salvestusrežiim**:**Ühekordne**(üks läbimine) /**Järjepidev**(järjepidi; piiratud salvestuste arvuga, vaikimisi 1, või kestusega, vaikimisi 10 s) /**Intervall** (ajavahemik: N salvestust iga X intervalli järel kokku Y korda, vaikimisi 1 iga 5 s järel 1 minuti jooksul).
* **Eksporditüübid kaamera kohta**: Raw, Debayered, Radiance, Reflectance, Preview, Index — kõik sobivad on vaikimisi sisse lülitatud. Radiance/Reflectance on peidetud RGB-filter kaamerate puhul;**Reflectance kuvatakse ainult siis, kui kaameral on DAQ-valgusandur** (oma või massiivilt pärandatud); Index nõuab konfigureeritud indeksaväljendit.
* **Joondatud**(massiivi kohta, vaikimisi**sisse lülitatud**): joondab liikmete ekspordid massiivi joondamisprofiiliga, nii et ekspordid on pikselite kaupa registreeritud. Raw jääb alati joondamata, kuid kannab teisendust metaandmetes.
* **Kiireim salvestamine** (lüliti): ainult toorandmed + määratud DAQ-näidud + tasuta kombineeritud indeksi komposiit, jättes salvestamise ajal kalibreerimisarvutused vahele maksimaalse kiiruse saavutamiseks — taastage kiirgus/peegeldus/indeks hiljem salvestatud `.daq`-ist.
* Valikud säilivad koos projektiga. Peidetud või peatatud kaamerad jäetakse vahele.

Vastav CLI (sama tagapõhja lõpppunkt, sama semantik):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

TIFF salvestuste pakkimine on `deflate` (kaotuseta, vaikimisi) või `none` — täielikud liputabelid, salvestuskausta struktuur ja ümbertöötlemise reeglid on esitatud [CLI viites](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## DAQ-valgusanduri sidumine

Peegeldusastme ja valgustusega korrigeeritud eelvaated vajavad DAQ-andurilt (mis on ühendatud vahekaardil **Valgusandurid**) allapoole suunatud valguse andmeid:

* Külgribal **massiivi real**kuvatakse**„DAQ · sisse/välja” nupp** — *sisse* lülitatud, kui on seadistatud massiivi tasandi valgussensor **või** kui mõnel kaameral on oma; selle tööriista vihjes on täpselt loetletud, milline sensor millist kaamerat toidab.
* Määrake massiiviülene seade massiivi seadetest → **Ümbritseva valguse andur**→**Valgusandur** rippmenüüst. Valik jääb projekti juurde, levib igale liikmekamerale ning üksikud kaamerad saavad seda endiselt oma anduriga ümber kirjutada.
* Allpool asuv staatusrida näitab hetkeolukorda: **Väljas**→ „Oodatakse esimest spektrit…“ →**„Aktiivne — kõik massiivi kaamerad on valgustuse suhtes korrigeeritud”** → või, kui viimase 3 sekundi jooksul pole saabunud uut spektrit, aegunud teade — kasutatakse jätkuvalt viimast mõõtmistulemust (mõõtmistulemused ei aegu kunagi salvestusteel).

Kui andur on määratud: muutub kättesaadavaks eksporditüüp „Reflectance“, reaalajas eelvaated on valgustuse suhtes korrigeeritud, ennustav automaatne säritus saab spektrit kasutada ning iga peegeldusvõte kirjutab tegelikult kasutatud DAQ-näidu **`.daq` sidecar** pildi kõrvale, et salvestust saaks hiljem uuesti töödelda.

## `array-connect` CLI valikud

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--serials SN1,SN2,…` | avasta automaatselt kõik LATTICE-kaamerad (vajalik ≥2) | **Esimene seerianumber on MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO sünkroniseerimisliin. |
| `--target-fps F` | automaatne | Masteri päästiku sagedus. |
| `--binning {1,2,4}` | automaatne | Riistvaraline binning. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automaatne | Sünkroniseerimistaseme valija ekspert-üleskirjutamine. |
| `--wire-ceiling-mbps MB_PER_S` | automaatselt tuvastatud | Host-liini eelarve MB/s — vorm CLI väljal **Wire Budget**. Vähendage seda, kui massiiv teatab vigastest raamidest. Salvestatakse koos projektiga, nii et hilisem ühenduse taastamine taastab selle. |
| `--no-recommend` | välja lülitatud | Jäta võrguanalüüsi etapp vahele. |
| `--no-ptp` | välja lülitatud | Lülita PTP välja (siis ei ole kaameratevahelised ajamärgid võrreldavad). |

`lattice array-list`, `array-status` ja `array-disconnect` haldavad püsivat seanssi. Täielik alamkäskude viide, sealhulgas joondamine (`align-calibrate` / `align-apply`) ja võrgutööriistad, asub [CLI Viide § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); SDK-i ekvivalendid (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) asuvad [SDK viites](../reference/sdk-reference.md). Alates mudelist Python on juhtme eelarve `connect_array(..., wire_ceiling_mbps=120)` ning kunagi kohale jõudmataei jõudnud kunagi kohale] asub [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
