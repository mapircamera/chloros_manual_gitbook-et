# Mitme kaamera massiivid

LATTICE **massiiv**on kaks või enam LATTICE-kaamerat, mis on ühendatud üheks sünkroniseeritud üksuseks. Üks kaamera on**juhtkaamera**: see saadab riistvaralise GPIO-triggerimpulsi ühisele sünkroniseerimisliinile (vaikimisi**Line2**), nii et iga liige pildistab samal hetkel. Chloros lisab PTP-ajasünkroniseerimise, reaalajas eelvaate (kaamerapõhised plaadid või üks joondatud mitme sagedusribaga komposiitpilt) ja sünkroniseeritud pildistamise – iga pildistamisvoor toodab ühe **kaadrirühma**, milles kõik kaamerad jagavad sama ajamärget ja kaadri ID-d (salvestusväljundis kuvatakse see kujul `fid:N`).

Massiivid on viis, kuidas monokromaatilised (M3M) kaamerad loovad taimestiku indekseid – üks kaamera annab ühe laineala ja massiiv joondab need mitme laineala pildistamiskogumiks. Vaata [Monokaamerad ja taimestiku indeksid](mono-indices.md).

Massiivi ühendamiseks on kolm võrdväärset viisi ja neil kõigil käivitatakse sama „smart-prep“ voog:

| Pind | Sisenemispunkt |
| --- | --- |
| GUI | Vahekaart „Kaamerad“ → **Ühenda massiiv** (sinine nupp) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (esimene seerianumber = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (esimene seerianumber = master) |

„Smart-prep“ teostab järgmist: võrguvõimekuse kontrolli (ICMP DF ping + GVSP-kontroll), sünkroniseerimistaseme valiku, kaadrisageduse prognoosi taotletud kaadri suuruse juures, pakitud 12-bitise edastusviisi valiku (vt [Pakitud 12-bitine transport](#packed-12-bit-transport)), PTP aktiveerimise, kaamera-põhise pikseliformaadi automaatse valiku, automaatse särituse algväärtuse määramise iga kaamera salvestatud seisundi põhjal ning GPIO-trigeri konfigureerimise Line2-l.

**Vaikimisi täisandur (uus versioonis 1.3.0).** Connect säilitab teie poolt soovitud kaadrisuuruse — täieliku 2048×1536 anduri, ilma piksliühendamiseta ja kärpimiseta, kui te teisiti ei vali — ning laseb kaadrisagedusel langeda sellisele tasemele, mida host ja võrk suudavad pakkuda. See ei tee kunagi ise piksleid kokku ega kärbi pilti (versioonis 1.2.0 hakati piksleid automaatselt kokku panema alla 1,5 kaadri sekundis). Pikslite ühendamine või väiksem huvipiirkond on teie valik: valige ühendusdialoogis üks neist ja „Prognoositav tulemus“ näitab sellega kaasnevat kaadrisagedust; `array-connect` väljastab kiiremad valikud koos nende prognoositavate kaadrisagedustega.

{% hint style="info" %}
Enne kui miski sellest toimib, peavad kaamerad olema ühenduse kaudu kättesaadavad — vaata [Kaamerate ühendamine](connecting.md), et saada teavet kaamerate avastamise, aadressimise ja esmakordse ühenduse kalibreerimise allalaadimise kohta. Mitme kaameraga seadmete puhul on võrguühenduse kiirusega sama olulised ka host-võrgukaardi vastuvõturingi seaded; täielik sümptomite ja lahenduste tabel asub jaotises [CLI-viide § Host-võrgukaardi seadistamine ja häälestamine](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialoog „Array Connect“

Vahekaardil „Kaamerad“ → **„Connect Array“**avaneb kolmeastmeline viisard:**Valik → Kuvarežiim → Seaded**.

### 1. samm — Valige juhtkaamera ja allkaamerad

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Dialoogaken skaneerib võrku kohe pärast avamist („Võrgu skaneerimine...”) ning seejärel kontrollib sünkroniseerimiskaableid („Sünkroniseerimiskaablite kontrollimine...”), et leida, millised kaamerad on ühendatud M8 sünkroniseerimisliiniga. Massiivi loomiseks on vaja vähemalt **2 kaamerat**.

M8 sünkroniseerimiskaablid ühinevad ühes liitmikus, seega **võib peakaameraks olla mis tahes kaabeldatud kaamera**. Test täidab rollid eelnevalt – RGB-kaamera (`-FRGB`) on peakaamera, kui see on ühendatud, muidu on peakaameraks esimene ühendatud kaamera ja teised ühendatud kaamerad on allkaamerad – ning kuvab ühe kolmest bännerist:

| Bänner | Tähendus |
| --- | --- |
| „Sünkroniseerimiskaabel leitud {n} kaameral. Igaüks neist võib olla juhtkaamera; RGB-kaamera (muidu esimene) on eelvalitud.“ (roheline) | Sond leidis kaamerad sünkroniseerimisliinilt; juhtkaamera ja allkaamera valikukastid on juba täidetud. |
| „Peakaamerat ei tuvastatud – kontrollige GPIO-kaablit“ (oranž) | Ükski kaamera ei tuvastanud käivitusimpulssi; kontrollige sünkroniseerimiskaablit. Rollid on endiselt eeltäidetud (RGB-kaamera, muidu nimekirja esimene kaamera, on peakaameraks) ja neid saab muuta. |
| „Sünkroniseerimiskaablit pole: {serials}“ (oranž) | Loetletud kaameratele pole ühendatud sünkroniseerimiskaablit. |

Kaameratabelis on veerud **Kaamera / Seerianumber / IP / Põhikaamera (raadio) / Allkaamera (valikukast)**. Read on loetletud**kaamerate järjekorras** — M3C (Bayer) kaamerad enne M3M (mono); M3C-de seas FRGB, FRGN, FOCN, seejärel FNGB; M3M-kaamerad on järjestatud filtri lainepikkuse järgi, madalaimast alustades (F450 enne F808). Külgriba kaamerate nimekiri ja tabeli vaikimisi paigutus kasutavad sama järjekorda.

* Valige täpselt **üks master**ja**üks või mitu slave'i**. Praeguse masteri raadionupule uuesti klõpsates tühistatakse valik. Uue masteri valimine toimub ühe klõpsuga: uus master lahkub slave'ide hulgast ja eelmine master muutub slave'iks (kui tal pole sünkroniseerimiskaablit).
* Kaamerat, mille juures on märge **„No Sync Cable”**, ei saa kunagi valida allkaameraks — allkaamera, millel puudub päästiku juhtmestik, ootaks sünkroniseerimisliinil igavesti ja edastaks tühja pilti. Ühendage see kaamera selle asemel iseseisva kaamerana.
* Juba iseseisvana ühendatud kaamerad *ei* lülitu välja: massiivi ühendamine vabastab iseseisva seansi ja avab kaamera uuesti massiivi sees.

**Järgmine: Kuvarežiim →**muutub kättesaadavaks, kui on valitud üks juhtkaamera ja vähemalt üks allkaamera.**Uuesti skaneerima** käivitab uuesti seadmete avastamise ja ühenduste kontrolli.

{% hint style="warning" %}
**Tühista** on keelatud, kui skaneerimine või ühenduse kontroll on käimas – kontrolli keskel tühistamine võib põhjustada kaamera SDK-i kokkujooksmise LATTICE-kaamera püsivara puhul. Oodake, kuni pöörlev ikoon lõpetab töö.
{% endhint %}

### 2. samm — Kuvarežiim

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->



| Režiim | Mida saad |
| --- | --- |
| **Eraldi kaamerad** | Üks reaalajas ruut iga kaamera kohta, kõik käivitatakse üheaegselt, et kaadrid püsiksid sünkroonis. Iga kaamera säilitab oma värvi ja seaded. |
| **Kombineeritud kaamerad** *(vaikimisi)* | Üks ruut, mis kuvab joondatud mitme sagedusriba NDVI/indeksi komposiiti. Kaamerad jagavad massiivi värvi. |

Kuvarežiim muudab ainult reaalaja eelvaate esitust — pildistamise käitumine on mõlemal juhul sama.

### 3. samm — Massiivi seaded ja prognoositav tulemus

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Sellesse stseeni sisenedes küsib Chloros tagapõhjalt **soovitust**ja rakendab seda:**täisandur – ilma binninguta, ilma ROI-lõikamiseta – alati, kui kaadrisagedus seda võimaldab**, olenemata sellest, millise kiirusega host ja ühendus andmeid edastavad. Ainult juhul, kui ükski kiirus ei suuda täisandurit edastada, vähendab soovitus kaadrit, ning sel juhul eelistatakse pikslite ühendamist ROI-lõikamisele, kuna pikslite ühendamine säilitab täieliku vaatevälja. Sealt alates vahetad resolutsiooni kaadrisageduse vastu. Iga tehtud muudatus käivitab analüüsi uuesti reaalajas ja uuendab parempoolset**prognoositava tulemuse** paneeli.

Vasak veerg – seaded:

| Kontroll | Valikud | Vaikimisi | Märkused |
| --- | --- | --- | --- |
| **ROI (vaateväli)** | Täis (2048×1536) / Pool (1024×768) / Veerand (512×384) | Täis | Anduri kärpimine: poole või veerandi kärpimine väiksemaks piirkonnaks algse pikselisammuga. |
| **Binning** | 1× / 2× (summa 2×2) / 4× (summa 4×4) | 1× | Riistvaraline binning: 2×2 = täisvaateväli veojuhikulu neljandiku juures; 4×4 = täisvaateväli 1/16 juures. Peidetud, kui kaamerad binningut ei toeta. |
| **Juhtmepoolne pilt** (väljalugemine) | — | — | Binningujärgne laius × kõrgus, mis tegelikult juhtmele saadetakse, ümardatud 16-kordsete väärtusteni (minimaalselt 64). |
| **Triggeri sagedus** | 0,5–60 kaadrit sekundis, samm 0,1 | tühi = automaatne | Põhiseadme triggeri sagedus. Jäta tühjaks, et lasta Chloros el seda arvutada. |
| **Andmeside eelarve**| 20–2000 MB/s, samm 10 | tühi = automaatne | Kui palju host tegelikult vastu võtta suudab, MB/s —**see on number, millest sõltub kogu massiivi eraldamine.** Määratakse automaatselt kindlaks võrgukaardi abil. Vähendage seda, kui massiiv teatab vigastest raamidest: tuvastatud väärtus ülehinnab USB-kaarte ja jagatud lüliteid. Selle muutmine käivitab prognoosi uuesti reaalajas. Väli avaneb viimase massiiviga seotud andmemahtuga (salvestatud koos projektiga), seega säilitatakse kord korrigeeritud andmemaht. |

Alates versioonist 1.2.0 on **Pin resolution** valikukast kadunud: ühendamisel säilitatakse alati valitud kaadri suurus ja selle asemel vähendatakse kaadrisagedust.

Parempoolne veerg — **Prognoositav tulemus**:

* **Sünkroniseerimistasand** — „Samaaegne salvestamine“ (roheline), „Samaaegne salvestamine (FTD-järkjärguline edastus)“ (roheline), „Järkjärguline salvestamine (100 ms nihke)“ (kollane) või „Konfiguratsioon liiga suur“ (punane). Chloros valib taseme: samaaegne salvestamine, kui kaablit on võimalik jagada kiirusel, millega kaamerad suudavad sammu pidada (kaadrisagedus langeb, kui iga kaamera osa väheneb), ja vahelduv salvestamine ainult siis, kui see pole võimalik — seega ühendub massiiv mis tahes võrguga, võrgu poolt lubatud kaadrisagedusega.
* **fps-prognoos** — kaadrisagedus, millega kaamerate massiiv eeldatavasti töötab. Sünkroonse salvestamise korral on see üks number: esmalt määratakse kaadrisagedus ja automaatne säritus täidab sellega antud ajavahemiku, seejärel suurendab võimendust, nii et hämar stseen seda ei vähenda (vt [Kuidas automaatne säritus kasutab säritusaega ja võimendust](camera-settings.md)). Järkjärgulise salvestamise korral kuvatakse see vahemikuna („hämar → hele”), sest iga kaamera ekspositsioon pikendab hämaras stseenis järkjärgulist salvestamist.
* **NIC-liin** — ühenduse kiirus ja püsiv läbilaskevõime („NIC {mbps} Mbps · püsiv {N} MB/s”).
* **Sim-emit-purske kontroll** — kas hosti võrgukaardi ringvõrk suudab vastu võtta ühe samaaegse purske kõigilt kaameratelt („Sim-emit-purske: X MB · võrgukaardi ringvõrgu kasutatav maht: Y MB ✓/✗”).
* **Võrgu läbilaskevõime kontroll** — püsiseisundi koondnõudlus võrreldes kokkupõrkeohutu võrgu ülempiiriga („Võrgu läbilaskevõime: {nõudlus} MB/s, mida nõuavad {n} kaamerat · ülempiir {ülempiir} MB/s ✓/✗ ületatud”).
* **„Maksimaalne kaamerate arv sellel võrguühendusel: {n} — määratud kaamera kohta kehtiva ribalaiuse alampiiriga, seega binning seda ei suurenda.“** — kuvatakse, kui olete lähedal (või ületanud) kaamerate arvu ülempiiri.
* **„Nende seadete juures KAOTATAKSE KAADREID."**— punane hoiatus koos tagapõhja põhjendusega, lisaks takistajate nimekiri ja sinised**lahendussoovitused** („Selle massiivi võrku mahutamiseks“ / „Samaaegse salvestamise vabastamiseks“).**Rakenda ja ühenda** on blokeeritud, kuni prognoos on olemas, ning selle silt näitab, miks see keeldub:

| Nupu silt | Tähendus | Mis tegelikult aitab |
| --- | --- | --- |
| „Analüüsitakse...” | Analüüs on veel käimas. | Oota. |
| **„Selle võrgu jaoks on liiga palju kaameraid”**| Võrk ei suuda massiivi edastada isegi ühe kaamera kaupa (koondkontroll ebaõnnestus): kaameraid on palju rohkem, kui võrk suudab tööle panna, või võrgus voogestavad ka teised kaamerad. | Vähem kaameraid, läbivalt jumbo-raamid või kiirem võrgukaart.**Väiksem ROI EI aita** — vaata allpool. |
| **„Vähenda ROI-d, et võimaldada“** | Nende seadete juures langeksid kaadrid välja (burst/ring-kontroll ebaõnnestus). | Vähenda ROI-d, suurenda binningut või paranda võrgukaardi vastuvõturingi. |



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Ühendamise ajal võib ilmuda roheline **kalibreerimise allalaadimise paneel** koos iga seerianumbri kohta eraldi edastusribaga: kui kaamera ühendatakse seadmega esimest korda, laadib Chloros GigE kaudu kaamerast alla selle ~3,8 MB suuruse tehase kalibreerimispaketi (umbes 70 sekundit kaamera kohta). Vahemällu salvestatud kaamerad ei näita seda paneeli kunagi; kui kaameraid on rohkem kui kuus, keritakse ridu. Vaata [Kaamerate ühendamine](connecting.md).

### Pakitud 12-bitine andmeedastus

12-bitised formaadid (`BayerRG12`, `Mono12`) edastatakse tavaliselt 16-bitiste sõnadena. **Pakitud**edastus saadab samad 12 bitti 1,5 baiti pikseli kohta ja pakib need lahti sihtseadmes, mistõttu iga kaadri saatmine võtab 25 % vähem aega — täisanduriga on see umbes kolmandiku võrra rohkem kaadreid sekundis.**Salvestatud andmed on identsed.** Array Connect otsustab selle automaatselt (`auto`): see pakib andmeid ainult siis, kui see on kasulik ja kui hostil on lahtipakkimiseks piisavalt vaba protsessorivõimsust. Graafilises kasutajaliideses (GUI) pole selle jaoks lülitit; käsurealt (CLI) kasutage `array-connect --packed {auto,on,off}` (vaikimisi `auto`). 8-bitiseid ja 16-bitiseid formaate ei pakita kunagi.

## Ribalaius: kui palju kaameraid mahub

See, mida array suudab edastada, sõltub ühendusliinist, mitte Chloros est, seega leiab planeerimisandmed riistvara kasutusjuhendist: **[Array Bandwidth Planning](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Chloros i tegevus nende andmete põhjal: ühendusdialoog käivitab võrgu proovi, prognoosib saavutatavat kaadrisagedust ja valib sobiva taseme. Massiiv ühendub igasuguse võrguga: aeglase ühenduse korral kohandatakse iga kaamera vastavalt oma osale ühendusest ja kaadrisagedus langeb, selle asemel et massiivi tagasi lükataks või selle kaadreid vähendataks. Keeldutakse ainult sellisest massiivist, mille puhul ühendus ei suuda korraga isegi ühtegi kaamerat edastada (**Selle võrgu jaoks on liiga palju kaameraid**), selle asemel et pakette vaikselt ära visata — vt eespool kirjeldatud prognoositavate tulemuste paneeli.

## Kui kaadrid kaovad

Kaamera võib avaldatud rühmast puududa kahel täiesti erineval põhjusel,
mis nõuavad vastupidiseid lahendusi. Chloros loendab neid eraldi, selle asemel et näidata üht
„ebatäielikku“ arvu, mis ei täpsusta kumbagi põhjust:

| Mis juhtus | Mida see tähendab | Kust otsida |
| --- | --- | --- |
| **Rikutud**— kaadrikogum jõudis kohale, kuid oli struktuuriliselt vigane | GVSP-pakettide kadu võrguteel |**Võrgu ribalaius**, võrgukaardi vastuvõturõngas, jumbo-kaadrikogumid, lüliti |
| **Ei jõudnud kunagi kohale**— raami ei tulnud üldse | Kaamera ei käivitatud või sellest ei läinud midagi välja |**M8 sünkroniseerimiskaabel**, sünkroniseerimisliin, kas kõik liikmed on aktiveeritud |

Jaotust hinnatakse uuesti iga 10 sekundi järel, kui kaamerarühm edastab andmeid. Kui see ületab 5 %,
logitakse see mõlema numbri nimetamisega ning iga vigane puhver registreeritakse esmakordsel
esinemisel kaamera kohta, seejärel koondatakse andmed kord minutis, et pikk seanss jääks loetavaks.

**Rikutud kaadrid, mille juures „ei jõudnud kohale“ väärtus on null, tähendavad, et päästik ja kaabli sünkroniseerimine toimivad täiuslikult**ning iga kadunud kaader on võrguteel. Lahenduseks on**Wire Budget**i vähendamine ja
uuesti ühendamine.

{% hint style="warning" %}
**Triggerimäära alandamine ei aita rikutud kaadrite puhul.** Kaamera pakettide
sagedus määratakse kindlaks ühekordselt ühenduse loomisel. Triggerimäära alandamine muudab seda, kui tihti pursk
toimub, mitte seda, kui kiiresti pursk ise võrku jõuab. Mõõdetud 4-kaameralisel seadmel ei
muutnud triggereerimissageduse 5-kordne vähendamine midagi, samas kui Wire Budgeti vähendamine 240-lt
200 MB/s viis sama süsteemi vigaste kaadrite määra 10,4 %-lt nullini.
{% endhint %}

Töötav massiiv ei saa ennast ümber planeerida – katkesta ühendus ja ühenda uuesti, et ühendusaja
valija saaks töötada uue andmeedastusmahu piires.

### USB-võrgukaartidel on piirangud

USB-Ethernet-adapter näitab oma *Etherneti* ühenduskiirust, kuid seda, mida ta tegelikult
suudab säilitada, piiravad USB-siin ja selle draiver. USB 10GbE-donglile omistati varem
läbilaskevõimsuseks umbes 1000 MB/s – arv, mida keegi polnud kunagi mõõtnud –, ning nelja kaamera
sünkroniseerimine selle fantoomse varuga rikkus 6–18 % kaadritest, samal ajal kui massiiv
näitas endiselt korralikku sihtkaadrisagedust. USB-ühendusega adapterite läbilaskevõime on nüüd piiratud
Windowsis ja NVIDIA Jetsonil **200 MB/s** ning Linux i x86_64-s **400 MB/s**. Piirang on absoluutne, mitte protsentuaalne, sest piiriks on
siin buss: USB 1 GbE-adapter saavutab umbes 80 MB/s ja seda piirangut ei mõjuta.

Kui teie host on piirangust tõepoolest kiirem, suurendage **Wire Budget** vastavalt.

## PTP-ajasünkroniseerimine

Kaadrite *sünkroniseerimine* toimub riistvaratriggeri abil; **PTP** (IEEE 1588 PTPv2) tagab võrreldavad *ajamärgised* kõigil seadmetel. See on vaikimisi sisse lülitatud massiivi ühendamisel:

* **PTP-grandmasterit käitab Chloros i hosti backend**. LATTICE-kaamerad ja DAQ-E valgusandurid töötavad selle alluvuses domeenis 0, mistõttu piltide ajamärgised ja DAQ-spektrid langevad ühele kellale (~1 ms).
* `--no-ptp` (CLI) lülitab selle laboritööde jaoks välja — sel juhul **ei** ole kaameratevahelised ajamärgised võrreldavad.
* Kontrollige sünkroniseerimise seisundit CLI kaudu:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Vahekaardil „Cameras“ (Kaamerad) puudub PTP-indikaator; seal kuvatakse kaamerapõhised sünkroniseerimisandmed, milleks on ainult lugemiseks mõeldud **Role**(Master/Slave),**Sync Line** ja massiivi võimekuse tase. DAQ-E PTP-olek kuvatakse vahekaardil „Light Sensors“ (Valgusandurid) andurite üksikasjade all.

## Massiivi reaalajas vaade

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

Peamine vooala pakub kahte paigutust (lülita ümber ülemises ribas): **võrgustikvaade**(iga ruut on lahter; kui ruudustiku lukk on avatud, saab paigutust lohistamisega muuta) ja**nimekirjavaade**(täislaiused read:**kombineeritud**massiiv on üks rida – selle koondruut –, samas kui**eraldi**massiiv annab**igale liikmele oma rea**, millel on oma reaalajas voog ja oma kalibreerimise sihtmärgi tabel; massiivide all üks eraldiseisev kaamera – aktiivne).**Feed Zoom** liuguriga saab ruutude suurust reguleerida; kui lahtri laius on alla 200 px, peituvad nime/kaadrisageduse ülekatted automaatselt. Liikmekaamerad kuvatakse külgribal ja ruudustiku vaikimisi paigutuses kaamerate järjekorras (M3C enne M3M, esmalt FRGB, monokroomkaamerad lainepikkuse järgi); võre lukustuse vabastamine ja lohistamine muudab siiski võre järjestust.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Eraldi režiim** näitab ühte ruutu kaamera kohta. Igal ruudul on järgmised ülekatted:

* kaamera nimi (ülemine vasak nurk),
* **kaadrisageduse näit** (ülemine parem nurk) — see on tagapõhja poolt edastatud kaamera *tegelik kaadrisagedus*, mitte eelvaate küsitlussagedus (reaalajas eelvaade on piiratud 30 kaadriga sekundis, sõltumata kaadrisagedusest),
* staatuse punkti — roheline (voogedastus) / kollane (laadimine) / punane (viga),
* **vananenud kaadri pöörlevat ikooni**, kui 2 sekundi jooksul pole saabunud ühtegi uut kaadrit — see on normaalne umbes 5 sekundi jooksul pärast ühenduse loomist või katkestamist, kui tagapõhi jaotab kaamerate vahel uuesti andmeedastusmahtu.**Kombineeritud režiim**kuvab üht komposiitruutu: tagapõhi teostab debayeringu, skaalab, joondab, eemaldab müra, teisendab ribapõhiseks kiirguseks (pluss DLS-peegeldusvõime, kui valgussensor on seotud), hindab massiivi indeksaväljendit, rakendab LUT-i ja edastab tulemuse MJPEG-vormingus. Kuni esimese joondatud kaadri renderdamiseni näitab ruut oma seisundit: „Massiivi ettevalmistamine…“, „Kalibreerib joondamist…”, „Ootab esimest kaadrit…”, või – kui plaadi automaatse joondamise korduskatseid (umbes minut) on ammendatud – „Vajalik joondamine” koos nupuga**Kalibreeri joondamine**.

Massiivi plaadi kaadrisageduse näidik näitab ka voo seisundit:**„Rakendatakse seadeid…“**, kui seadete muutmine (formaat, binning, ROI) taaskäivitab kaamerad,**„Seisak“**, kui massiiv on aktiivne, kuid on lõpetanud kaadrirühmade edastamise, ja**„Peatunud”**, kui voog on peatunud – sel juhul ühendage massiiv uuesti. Kui spektraalne eraldamine pidi komposiitpildil näidatud sagedusriba suurel osal selle pikslitest (25 % või rohkem) piirama, kuvab paneel**„Madala usaldusväärsusega sagedusriba andmed · N%"**: sellel lainealusel põhinevad indeksväärtused on kahtlased (tavaliselt sai üks laineala liiga vähe valgust).

Kasulikud faktid kombineeritud režiimi kohta:

* Komposiit on registreeritud **peakaamera**kaadri järgi. AE-ROI suunamine ja punktmõõtmine komposiidil on peakaamera puhul täpsed ja allkaamerate puhul ligikaudsed; kasutage**Split View** (massiivi seaded → „Show member cameras”), et saada pikselitäpsed kaamerapõhised plaadid ilma täiendavaid kaameraühendusi avamata.
* **Display Layers** (massiivi seaded; vaikimisi välja lülitatud) võimaldab valida esiplaani ja taustakihti — mis tahes liikmekamera või**Index**. Kui esiplaaniks on valitud**Index**, kuvatakse LUT-i miinimum- ja maksimumväärtustest väljaspool asuvad pikslid taustakihtina.
* **Renderdamise resolutsioon** (vaikimisi 720p) määrab nii otseülekande kõrguse *kui ka* salvestatud komposiitpildi ekspordimõõtmed. Iga kaamera pildid eksporditakse alati täisresolutsioonis.
* Joondamine arvutatakse iga seansi kohta eraldi ja seda ei salvestata kunagi. Chloros joondab **iga massiivi automaatselt pärast ühenduse loomist, mõlemas kuvamisrežiimis**, kui liikmete automaatne säritus on stabiliseerunud (seda proovitakse kuni 5 minutit). Massiivi seadete paneeli**Joondamine**-osas kuvatakse RMS-jäägid ja nupp**Joondamine uuesti kalibreerida** — ka eraldi massiivide puhul, millel pole komposiitpaneeli.

## Salvestamine: seire vs analüüs

Massiivi salvestuspinnad jagunevad selgelt **seiretasemeks**(salvestab seda, mida näed) ja**analüüsitasemeks** (salvestab toorandmeid, kalibreerib hiljem):

| Töövoog | Tase | Mida salvestatakse | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Salvestamine**(pildid) | Analüüs | Üks sünkroniseeritud kaadrirühm iga läbimise kohta; iga kaamera failid igal valitud eksporditasemel (toorandmed/debayered/kiirgus/peegeldus/eelvaade/indeks) + `.daq` sidecar |**Nupp „Capture All**“ + salvestusseaded | `lattice array-capture` |
| **Indeksvideo salvestamine** | Jälgimine | Näidatav reaalajas kombineeritud indeks-komposiit – 8-bitine, eelvaate resolutsioon, sisseehitatud LUT. Kui GUI-vaade pole avatud, renderdab tagapõhi CLI-st massiivi NDVI-d (või `--expr`-d) kohe, kui massiiv on joondatud; salvestamine, mis ei kirjuta ühtegi kaadrit, ebaõnnestub | ● Salvestada indeksvideo (kombineeritud massiivid) | `lattice array-record` |
| **Toores pildiseeria → video koostamine**| Analüüs | Toored andurikaadrid täieliku salvestuskiirusega + manifest + `.daq`, seejärel offline-rekonstrueerimine kalibreeritud kiirguse / peegelduse / indeksvideoks, ajaliselt sobitatud DAQ-näitudega | ⦿ Salvestage toores pildiseeria →**Koostage video** | `lattice array-burst` → `lattice array-build-video` |

Praktiline reegel: kui piksleid kasutatakse *mõõtmisteks*, kasuta salvestamist või pildiseeriat (analüüsitasand); kui soovid lihtsalt *vaadata või demonstreerida*, mida kaamera nägi, salvestada indeksvideo (seiretasand).

### Salvestusseaded (GUI)



<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

**Capture All** kõrval asuv hammasratas avab salvestusseadete paneeli (nõuab avatud projekti – salvestised salvestatakse sinna):

* **Salvestusrežiim**:**Single**(üks läbivõte) /**Continuous**(järjepidev; piiratud salvestuste arvuga, vaikimisi 1, või kestusega, vaikimisi 10 s) /**Interval** (ajavahemik: N salvestust iga X intervalli järel kokku Y korda, vaikimisi 1 iga 5 s järel 1 minuti jooksul).
* **Eksporditüübid kaamera kohta**: Raw, Debayered, Radiance, Reflectance, Preview, Index — kõik sobivad on vaikimisi sisse lülitatud. Radiance/Reflectance on peidetud RGB-filtriga kaamerate puhul; **Reflectance ilmub ainult siis, kui kaameral on DAQ-valgusandur** (oma või massiivilt pärandatud); Index nõuab konfigureeritud indeksaväljendit.
* **Joondatud**(massiivi kohta, vaikimisi**sisse lülitatud**): moonutab liikmete ekspordid massiivi joondamisprofiili järgi, nii et ekspordid on pikselite kaupa registreeritud. Raw-andmed jäävad alati moonutamata, kuid sisaldavad teisendust metaandmetes. Massiivi liikmete failid salvestatakse iga kaamera algses orientatsioonis (kõik peegeldus- ja pööramisseaded salvestatakse faili, kuid neid ei rakendata).
* **Kiireim salvestamine** (lüliti): ainult toores andmestik + määratud DAQ-näit + tasuta kombineeritud indeksiga komposiit, jättes salvestamise ajal kalibreerimisarvutused vahele maksimaalse kiiruse saavutamiseks — taasta kiirgus/peegeldus/indeks hiljem salvestatud `.daq`-failist.
* Valikud jäävad projekti juurde alles. Peidetud või peatatud kaamerad jäetakse vahele.
* **Pidev** režiim massiividel töötab *Chloros

* tagapõhis (uus versioonis 1.3.0): kui iga nimekirjas olev kaamera kuulub massiivi, käivitab tagapõhi ise salvestusringi, kirjutades ühe grupi salvestusele samal ajal, kui valib järgmise, ilma et iga salvestuse jaoks oleks vaja eraldi päringut. Täis kõvaketas peatab töötamise („Peatatud: kõvaketas täis. Salvestatud N, F ebaõnnestus”). Vaata [Salvestusseaded ja -režiimid](capture.md#capture-modes), et tutvuda Microsoft Defenderi märkusega kõrge salvestussageduse kohta.

Vastav CLI-käsk (sama backend-lõpppunkt, sama semantik):

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

Pildistamise TIFF-pakkimine on `deflate` (kaotuseta, vaikimisi) või `none` — täielikud liputabelid, pildistamiskausta struktuur ja ümbertöötlemise reeglid on toodud [CLI-viites](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## DAQ-valgusanduri sidumine

Peegeldus- ja valgustusega korrigeeritud eelvaated vajavad DAQ-andurilt (mis on ühendatud vahekaardil **Valgusandurid**) allapoole suunatud valguse andmeid:

* Külgribal **maatriksi reas**kuvatakse**„DAQ · sisse/välja” nupp** — *sisse* lülitatud, kui on seadistatud maatriksi tasandi valgussensor **või** kui mõnel kaameral on oma sensor; selle abiteates on täpselt loetletud, milline sensor millist kaamerat varustab.
* Määrake massiiviülene seade massiivi seadetest → **Ümbritseva valguse andur**→**Valgusandur** rippmenüüst. Valik säilib koos projektiga, levib igale massiivi kuuluvale kaamerale ning üksikud kaamerad saavad seda endiselt oma anduriga ümber kirjutada.
* Allpool asuv staatusrida näitab hetkeolukorda: **Väljas**→ „Oodatakse esimest spektrit…” →**„Aktiivne — kõik massiivi kaamerad on valgustuse suhtes korrigeeritud”** → või, kui viimase 3 sekundi jooksul pole saabunud uut spektrit, aegunud teade — kasutatakse jätkuvalt viimast mõõtmistulemust (mõõtmistulemused ei aegu kunagi salvestusrajal).

Kui andur on määratud: muutub kättesaadavaks eksporditüüp „Reflectance“, reaalajas eelvaated on valgustuse suhtes korrigeeritud, ennustav automaatne säritus saab spektrit kasutada ning iga peegeldusvõte salvestab tegelikult kasutatud DAQ-näidu **`.daq`-failina** võtete kausta `daq/` kausta, et salvestust saaks hiljem uuesti töödelda. Selle kõrval nimetab fail `.daq.cameras.json` kaamerad, mille kaadreid see näid kirjeldab; uuesti töötlemisel sobitab Chloros näidu ainult nende kaamerate piltidega.

## `array-connect` CLI-valikud

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--serials SN1,SN2,…` | avasta automaatselt kõik LATTICE-kaamerad (vajalik ≥2) | **Esimene seerianumber on MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO sünkroniseerimisliin. |
| `--target-fps F` | auto | Masteri päästmise sagedus. |
| `--binning {1,2,4}` | auto | Riistvaraline binning. „Auto“ tähendab täisandurit (ilma binninguta), kui kaadrisagedus seda toetab; „connect never“ ei kasuta binningut, et säilitada kaadrisagedus. |
| `--packed {auto,on,off}` | `auto` | [Pakitud 12-bitine edastus](#packed-12-bit-transport): `auto` pakib andmeid ainult siis, kui see on kasulik ja kui host suudab lahtipakkimist toetada. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Sünkroniseerimistaseme valija asjatundlik ümberkirjutamine. |
| `--wire-ceiling-mbps MB_PER_S` | automaatselt tuvastatud | Hosti andmeside eelarve MB/s-is — **Wire Budget**-välja CLI-vorm. Vähendage seda, kui massiiv teatab vigastest raamidest. Salvestatakse koos projektiga, seega taastub see hilisema ühenduse taastamisel. |
| `--no-recommend` | välja lülitatud | Jäta võrguanalüüsi etapp vahele. |
| `--no-ptp` | välja lülitatud | Lülita PTP välja (siis ei ole kaameratevahelised ajamärgised võrreldavad). |

`lattice array-list`, `array-status` ja `array-disconnect` haldavad püsivat seanssi. Täielik alamkäskude viide, sealhulgas joondamine (`align-calibrate` / `align-apply`) ja võrgutööriistad, asub [CLI-viite § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); vastavad SDK-funktsioonid (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) on toodud [SDK-viites](../reference/sdk-reference.md). Pythonis on andmeedastuse eelarve `connect_array(..., wire_ceiling_mbps=120)` ning andmete jagunemine „rikkis” ja „ei jõudnud kohale” vahel on esitatud [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
