---
metaLinks: {}
---

# Alustamine

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros on [MAPIR](https://www.mapir.camera) tarkvararakendus, mis on mõeldud multispektraalsete piltide töötlemiseks, MAPIR-riistvara reaalajas juhtimiseks ja andurite andmete salvestamiseks. Chloros 1.3.0 toetab kogu MAPIR-tooteperekonda:

* **Survey3-kaamerad** — töötlevad RAW+JPG-pildid kalibreeritud peegeldus- ja taimestikuindeksikaartideks. Vaata [Toetatud kaamerad](supported-cameras.md).
* **LATTICE-kaamerad** — ühendage GigE multispektraalsed kaameramoodulid reaalajas, üksikult või sünkroniseeritud mitme kaamera massiividena: eelvaade, pildistamine ja töötlemine kalibreeritud kiirguse ja peegeldusvõime toodeteks. Vaadake [LATTICE-osa](lattice/README.md).
* **DAQ valgussensorid** — DAQ-U (USB), DAQ-M (Bluetooth) ning DAQ-E ja DAQ-E-S (Ethernet) spektraalsensorid: kalibreeritud spektrid reaalajas, `.daq` salvestused ja allapoole suunatud valgustatus peegeldusvõime töötlemiseks. Vaata [DAQ-osa](daq/README.md).

{% hint style="success" %}
**Chloros 1.3.0 uuendused**: AI-abilised saavad juhtida Chloros'i läbi `chloros-mcp` ning CLI, SDK ja `chloros-mcp` pakuvad samu funktsioone. LATTICE indeksitooted on täiustatud, täissensorite massiivid on kiiremad ja töötavad igas võrgus ning tagapõhi jätkab töötamist käskude vahel. Vaadake allpool olevat uuenduste loendit ja [laadige alla](download.md) paigaldusprogrammid.
{% endhint %}

{% hint style="info" %}
**Kasutad Chloros'i koos AI-assistendiga?** See kasutusjuhend on just selleks mõeldud. Suuna oma assistent järgmistele linkidele:

* `https://mapir.gitbook.io/chloros/llms.txt` — masinloetav indeks kõigi lehtede kohta.
* Mis tahes leht toores Markdown-vormingus — lisage selle URL-ile `.md` (nt `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-viide](reference/cli-reference.md) ja [SDK-viide](reference/sdk-reference.md) — täielikud, täpsete väärtustega viitelehed, mis on kirjutatud LLM-i kasutamiseks.

Näide käsust: *„Loe https://mapir.gitbook.io/chloros/reference/cli-reference.md, ja kirjuta seejärel skript, mis logib sisse ja töötleb kausta ~/flights/flight_001, muutes selle peegeldusvõime ja NDVI väärtustega GeoTIFF-failideks.”*

Või lase tal käivitada otse Chloros: `chloros-mcp`, SDK-ga paigaldatud MCP-server, töötab koos Claude Desktopi, Claude Code’i, Cursori ja teiste MCP-klientidega.

Täielik juhend: [Chloros e kasutamine AI-abistajatega](ai-assistants.md).
{% endhint %}

***

## Uued funktsioonid Chloros 1.3.0 versioonis

* **Tehisintellekti abilised saavad juhtida Chloros'i.** `chloros-mcp`, Python i SDK-ga installitud MCP-server, võimaldab Claude Desktopil, Claude Code'il, Cursoril ja teistel MCP-klientidel töödelda pilte ning juhtida LATTICE-kaameraid, andurimassiive ja DAQ-valgusandureid. Vaata [Chloros'i kasutamine koos tehisintellekti abilistega](ai-assistants.md).
* **CLI, SDK ja `chloros-mcp` pakuvad samu funktsioone.** Turvalisuse tagamiseks jäävad CLI-le ainult sisselogimine, DAQ-E seadistamine ja otsesed DAQ-serverid. Mõned 1.2.0 skriptid vajavad muudatusi: vaata **Muudatused versioonis 1.3.0** [CLI-viite](reference/cli-reference.md) ja [SDK-viite](reference/sdk-reference.md) ülaosas.
* **Täiustatud LATTICE indeksitooted.** Kõikides töötlemisrežiimides on täiustatud NDVI-d ja teisi LATTICE-piltidest eksporditud indeksitooteid.
* **Windows i paigaldaja valikud.** **Süsteemi seaded**all pakub paigaldaja valikuid**Luba Windows i pikad teed**ja**Jäta Chloros i tagapõhi Microsoft Defenderi skaneerimisest välja**, mis on mõlemad vaikimisi märgitud. Esimene valik võimaldab projektikausta asukohta, mis on pikem kui 260 tähemärki; teine takistab reaalajas skannimisel kaamera salvestamise aeglustumist (umbes 30% salvestuskiirusest kõrgel kiirusel). Paigaldaja lisab nüüd ka CLI-kausta `PATH`-i ainult üks kord ning eemaldab varasemate paigaldajate poolt jäetud dubleeritud kanded. Vaata [Allalaadimine](download.md).
* **Liiga sügavad projektikaustad lükatakse kohe tagasi.** Kui Windows i pikad teed on välja lülitatud (Windows i vaikimisi seade), lükatakse 260-märgilise teepikkuse piirangust liiga sügav projektikaust tagasi enne mis tahes toimingute käivitamist, näidates samas pikimat sobivat projektikaustu ja selgitades, kuidas pikad teed sisse lülitada. Varem ei salvestanud selline käivitus ühtegi toodet. Kui pikad teed on sisse lülitatud, säilitavad väga sügavates kaustades asuvad tooted nüüd oma metaandmed (GPS- ja XMP-märgised).
* **Maskide piiramine.** Kui LATTICE-kiirguse või peegeldusvõime toote lainepikkuste eraldamine annab negatiivse lainepikkuse väärtuse, lõigatakse see väärtus nullini ning tootekataloogide kõrval asuvas `clamp_masks/`-kataloogis asuv pikselipõhine mask salvestab, millised pikslid ja lainepikkused olid mõjutatud. Toode nimetab oma maski `Chloros:UnmixClampMask`-märgises. Reaalajasvaated kärbitakse nüüd täpselt samamoodi nagu eksportimisel. Vaata [Väljundpildi formaadid](output-image-formats.md).
* **Tooted kirjeldavad end ise.** Indeksitooted kannavad oma töötlemismärke (`Chloros:ProcessingLevel=index`, indeksavaldis ning kas need arvutati kiirguse või peegelduse põhjal) allika toorfailide omade asemel. Survey3-tooted kannavad igas töötlemisrežiimis Chloros-töötlemismärke. LATTICE toorfailid ja debayered-failid säilitavad anduri algse orientatsiooni ja salvestavad selle XMP-sse; pildivaataja pöörab neid vaid kuvamise eesmärgil.
* **Linux i paketid tarnitakse lahtipakitud kujul.** Tagapõhi ja CLI paigaldatakse kaustadena kataloogi `/usr/lib/chloros/` alla, mistõttu need ei paku end enam iga käivitamise ajal lahti. x86_64-platvormil ei kuluta backend enam käivitumiseks umbes 100 sekundit ning `.deb` on umbes poole väiksem kui varem. Komplektis olev ExifTool töötab nüüd puhta Ubuntu installi peal. `.deb` installib nüüd ka Python i SDK süsteemi jaoks `python3` (paigaldage see `sudo apt install ./chloros_1.3.0_<arch>.deb` abil), suurendab tuuma sokli vastuvõtupuhvreid, mida kaamera voogudele vaja on, ning lisab tagapõhiteenuse, mis töötab teie kasutajana. Jetsoni pakett, `chloros_1.3.0_arm64_jp6.deb`, on koostatud JetPack 6 jaoks ning töötleb andmeid Jetsoni GPU-l, kasutades JetPacki CUDA-põhist PyTorchi versiooni. Linuxis jõuab iga eksporditud pilt nüüd kord kettale (kirjutatakse ja märgistatakse mälus, seejärel kopeeritakse õigele kohale): Jetsonil, kus ketta kiirus määrab töö tempo, võtab 288 pildiga LATTICE-käivitus aega 14 minutit 24 asemel. Vaata [Linux i paigaldamine](linux/linux-installation.md).
* **LATTICE OCN-kaamerad loevad päikesevalguses täpsemalt.** Radiance, peegeldusvõime ja indeksid saavad pärast lainepikkuste eraldamist kiire korrektsiooni, mistõttu loetakse halli sihtmärki päikese käes peaaegu neutraalse NDVI-na. RGN-kaamerad jäävad muutumatuks.
* **Rakendusesisesed veateated.** Veakuvand „Debug Log“ vahekaardi ülaosas saadab MAPIR-ile kirjelduse, kuni 10 pilti, logid ja süsteemiandmed. Vaata [GUI: navigeerimine](navigation.md).
* **LATTICE-massiivid.**Massiivid säilitavad vaikimisi täisandurirežiimi ning pakitud 12-bitine andmeedastus (`--packed`, vaikimisi `auto`) suurendab täisandurirežiimi kaadrisagedust umbes kolmandiku võrra; massiiv töötab igas võrgus selle võimaldatava kaadrisagedusega (aeglane ühendus alandab kaadrisagedust, selle asemel et massiivi tagasi lükata või selle kaadreid vähendada); massiiv, mille kaamerad on läbinud toite tsükli, taastub ise, kusjuures PTP lülitub uuesti sisse; automaatne säri ei alanda enam seda sagedust hämaras valguses (see täidab perioodi ja suurendab seejärel võimendust); Pidev salvestamine suudab kiirete massiividega sammu pidada;**Highlights**-regulaator RGB-kaameratele; ning SDK-i `ArraySession` saab juurde `grab()`, `frame_stream()` ja `continuous()`. Vaata [LATTICE-osa](lattice/README.md).
* **Stabiilsem automaatne säri.** Säriparameeter püsib konstantsena ja stabiliseerub mõne sekundi jooksul pärast valgustuse muutust; väikesed heledad alad ei tumenda enam stseeni; käsitsi säriparameetri seadistamisel jääb võimendus automaatrežiimile; ning kaamera, mis lakkab reageerimast eredas päikesevalguses, hoitakse selle valgustuse tasemel, mida loeb valgussensor või – kui seda pole ühendatud – kaamera enda kaadrid. Vaata [Kaamera seaded](lattice/camera-settings.md).
* **Valgusandurid.** **DAQ-E-S tugi:** selle fikseeritud hajuti ei vaja katekorrektsiooni ning iga mõõtmistulemus sisaldab seadme kallutust, pööramist ja kaldenurka. Salvestus näitab, kas kaamera kate valiti või eeldati, ning töötlemine annab hoiatuse eeldatud kaane puhul: DAQ-U, DAQ-M ja DAQ-E tarnitakse ilma kaaneta, seega nimetage paigaldatud kaane tüüp. Reaalajas `.daq` salvestused säilivad ka voolukatkestuse korral (hoidke `.daq-journal` faili koos selle `.daq`-iga), ning DAQ-E, mis lõpetab andmete edastamise, ühendatakse uuesti, kusjuures salvestuses märgitakse andmevahe. Projekt, mis sisaldab ainult valgusanduri salvestusi, loetleb neid ja töötleb neid nii rakenduse kui ka CLI kaudu, ning `chloros_scripts`-iga tehtud salvestus on loetletud selle DAQ-na, mis selle tegi, koos teie poolt seadistatava piiranguga. Iga salvestus salvestab kasutatud DAQ-i näidu. Vaadake [DAQ-i jaotist](daq/README.md).
* **Töötlemine.**LATTICE-kaadrid alustavad töötlemist sihtmärgi tuvastamise ajal. Sihtmärgi ja DAQ-i viited määratakse iga sagedusriba kohta eraldi. Ümberpaigutatud projekt loob oma failidele uued lingid. 8-bitise salvestuse ekspordid on 8-bitised, uues**TIFF (8-bit)** formaadis. Survey3-i käivitamine ilma kalibreerimissihita salvestatakse selle varutootena (`Vignette_Corrected_Images` või `Sensor_Response_Images`), mitte peegeldusvõime pildina. Peegeldusvõime pildi ribapõhised DLS-märgised on alati selle enda kaadri omad. LATTICE-pildistamiste puhul ei toeta funktsioon „Texture Aware”, mistõttu neile rakendatakse alati standardset debayeri.
* **CLI ja SDK.** Suhtelised väljunditeed asuvad nüüd kaustas, kust käsk või skript käivitatakse (varem lahendas installitud backend need oma installikaustas ja see lõppes ebaõnnestumisega). Tagapõhi töötab käskude ja skriptide vahel edasi ning peatub ise 30 minuti pärast, kui seda ei kasutata; seda haldavad `chloros-cli backend status` ja `backend stop`. Salvestuskäsud nimetavad iga faili, mida nad kirjutada ei suutnud, ja lõpetavad töötamise koodiga 1; korduvad salvestused peatuvad, kui kõvaketas on täis. Edukas `process` käivitamine annab tulemuseks `Image products written: N (plus M clamp masks)`. Windowsis leiab SDK `ChlorosLocal` / `process_folder` leiavad installitud tagapõhja (ja kasutavad juba töötavat) ning `read_image_audit_tags` töötab tavalisest `pip install`-ist.

***

## Uued funktsioonid versioonis Chloros 1.2.0

* **Kaamera reaalajas juhtimine — uus vahekaart „Kaamerad“.** Ühendage LATTICE-kaamerad ükshaaval või sünkroniseeritud mitme kaamera massiividena (PTP-ajasünkroniseerimine, riistvarapõhine pildistamine), koos reaalajas eelvaate ülekattete, sagedusribade kaupa histogrammide, nutika automaatse särituse, reaalajas indeksikalkulaatori ja rakendusesiseste kaamera tarkvarauuendustega.
* **Valgusandurid — uus vahekaart „Valgusandurid”.** Ühendage DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet) andurid; vaadake kalibreeritud spektreid reaalajas (W/m²/nm), salvestage `.daq`-failid oma projekti, valige kap-korrektsiooniprofiilid ja uuendage DAQ-E püsivara võrgu kaudu.
* **Salvestusrežiimid ja salvestid.** Ühekordne / pidev / intervall-salvestus ning ainult toorandmetele mõeldud kiireim salvestusrežiim; projektipõhine valik, milliseid kaameraid ja eksporditüüpe „Capture All” toodab; massiivsalmestid seiretasemel indeksvideote ja analüüsitasemel toorandmete seeriate jaoks koos offline-videokoostamisega.
* **LATTICE-töötlusprotsess.** Importige LATTICE-salvestuskaustad ja jagage iga toorkaader debayeringuga töödeldud, eelvaate, float32-kiirguse (W/m²/sr/nm) ja peegeldusvõime tulemusteks, millel on tootepõhised lülitid. Peegeldusvõime võib pärineda kaadri sees olevast kalibreerimismärgist või DAQ-i allapoole suunatud valgusest; eksportidele rakendatakse massiivi joondamist; puuduv tehase kalibreerimine laaditakse automaatselt alla kaamera seerianumbri järgi.
* **Projektid mäletavad riistvara.** Ühendatud kaamerad ja valgusandurid salvestatakse koos projektiga (`cameras.json` / `sensors.json`) ning ühenduvad uuesti oma salvestatud seadetega, kui projekt uuesti avatakse. Vaata [GUI: Projektid](projects.md).
* **Pildivaataja uuendused.** Kursori pikseli/indeksi väljund koos failipõhise õige peegeldusastme skaalimisega, kihtide histogrammid, GSD-binningu liugur, võrgustikurežiimid „Per Trigger“ ja „Per Camera“, LATTICE-tootevaated ning indeksi/LUT-i sandbox-eksport kettale.
* **CLI ja SDK on oluliselt laiendatud.** Uued käskude perekonnad `lattice`, `daq pool-*`, `project` ja `time-sync`; uued `process` valikud (`--input-level`, tootekohased lülitid, `--reflectance-source`, massiivi joondamise lipud); SDK-i smart-connect-käepidemed (`connect_camera` / `connect_array` / `connect_daq_sensor`), mis käivitavad tagapõhja automaatselt; `open_project()` automatiseerimine; SDK-ratas on lisatud paigaldajatele ja avaldatud PyPI-s nimega `chloros-sdk`.
* **Aus veasemantika.** `chloros-cli process` käivitus, mis taotles tooteid, kuid ühtegi ei salvestanud, annab nüüd selge veateate ja lõpetab töötamise nullist erineva tulemusega; edukad käivitused teatavad, kui palju pilditooteid nad salvestasid.
* **Uus väljundpaigutus.** Tooted salvestatakse `<project>/<camera>/<format>/<Product>_Images/`-kaustadesse ja säilitavad allika failinime — toodet identifitseerib kaust, mitte failinime laiend. Vaata [Väljundpildiformaadid](output-image-formats.md).
* **Rohkem sisendvõimalusi, plaane ja keeli.** `.dng` sisendite tugi; kõik 38 liidese keelt on täielikult täidetud; plaani kohta kehtivad riistvarapiirangud, mis võimaldavad tasuta (ilma sisselogimiseta) kasutada kuni 4 kaamerat ja 2 valgussensorit.
* **Usaldusväärsus.** Töötlemise peatamine lõpeb korrektselt koos ausa töökokkuvõttega, mitme kaameraga projektid ekspordivad iga kaamera andmed ning installija uuendused ei logi teid enam välja.***

Chloros on saadaval kolmes kasutajaliidese vormis:

## Chloros: töölaua graafiline kasutajaliides

Eraldi aken kõigi funktsioonidega, sealhulgas vahekaardid „Kaamerad“ ja „Valgusandurid“ reaalajas. _Ainult Windows._

## [Chloros CLI: käsurealiides](CLI.md)

Käsurea paketttöötlus koos reaalajas käskudega `lattice`, `daq pool-*`, `project` ja `time-sync`. Ideaalne automatiseerimiseks, skriptide koostamiseks ja peata tööks. Saadaval **Windowsis, Linux amd64 ja Linux arm64 (NVIDIA Jetson)**. _CLI
-le juurdepääsuks on vaja tasulist Chloros+ taset._

## [Chloros i API: Python SDK](api-python-sdk.md)

Programmeeritav Python-liides automatiseerimiseks ja kohandatud töövoogudeks: täielik töötlemisprotsess, reaalajas kaamera-/massiivi-seansid, DAQ-andurite seansid ja salvestatud projektide automatiseerimine. Paigaldatakse koos töölaua-/CLI-paketiga ning avaldatakse ka nimetusega `pip install chloros-sdk`. _API-le juurdepääsuks on vaja tasulist Chloros+ taset._

***

## Toetatud platvormid

| Platvorm | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Jah | Jah | Jah |
| **Linux amd64 (x86_64)** | Ei | Jah | Jah |
| **Linux arm64 (NVIDIA Jetson)** | Ei | Jah | Jah |

Linux i paigaldusjuhiseid leiate jaotisest [Linux ja servaarvutus](linux/linux-overview.md).

***

## Alustamine kolmes sammus

1. **Paigaldamine** — laadige alla ja käivitage oma platvormile mõeldud paigaldusprogramm. Vaadake [Allalaadimine](download.md).
2. **Logi sisse (valikuline GUI puhul)** — GUI töötleb pilte tasuta ilma kontota. [Chloros+ sisselogimine](chloros+-login.md) avab juurdepääsu paralleelsele töötlemisele, GPU kiirendusele, suurematele seadmepiirangutele ning CLI-le ja SDK-le.
3. **Loo oma esimene projekt** — ava Chloros, loo [uus projekt](projects.md), [lisa oma pildid](processing-images-gui/adding-files-to-a-project.md) ja [alustage töötlemist](processing-images-gui/starting-the-processing.md). Kui soovite selle asemel juhtida reaalajas riistvara, avage vahekaart „Kaamerad“ või „Valgusandurid“ — vt [GUI: navigeerimine](navigation.md).

***

## Chloros+

Kuigi Chloros on enamiku ülesannete jaoks tasuta, võite avastada, et soovite rohkem. Siis võib Chloros+ tasuline litsents teile kasuks tulla. Chloros+ litsentsiga saate avada uusi funktsioone, nagu:

* **Mitmeprotsessiline töötlemine**: kiirendage oluliselt pilditöötlust suuremate projektide puhul, töödeldes pilte töötlusketis samaaegselt.
* **GPU (CUDA) kiirendus**: kasutage ära tänapäeva suurema GPU-mälu mahtu, et pilditöötluse protsessi veelgi kiirendada. Parimate tulemuste saavutamiseks soovitame 4 GB või rohkem VRAM-i.
* **Chloros+**[**CLI**](CLI.md) **Juurdepääs**: käivitage Chloros+ käsurealt, et automatiseerida ja integreerida see oma tarkvarasse. Saadaval kõikidel tasulistel tasemetel; rakendatakse serveri poolel.
* **Chloros+**[**API**](api-python-sdk.md) **Juurdepääs:** käivita Chloros+ Pythonis programmiliseks juhtimiseks, mis võimaldab sujuvat integreerimist sinu uurimisprotsesside, andmeanalüüsi töövoogude ja kohandatud rakendustega. Saadaval kõigil tasulistel tasemetel; rakendatakse serveri poolel.
* **Kõrgemad riistvarapiirangud**: ühendage korraga rohkem kaameraid ja valgussensoreid. Ilma sisselogimiseta ühendab graafiline kasutajaliides kuni 4 kaamerat ja 2 DAQ-valgussensorit; tasulised paketid suurendavad mõlemat piirmäära:

| Pakett | Kaamerad | DAQ-valgussensorid |
| --- | --- | --- |
| Iron (tasuta, sisselogimata) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Mitme seadme kasutamine**: iga Chloros+ litsents võimaldab registreerida 2 või enam seadet. Kasutage oma MAPIR Cloudi kontot registreeritud seadmete haldamiseks. Lisage tugi rohkematele seadmetele, uuendades oma Chloros+ litsentsi.
* **Täiustatud tekstuuritundlik debayer-meetod:** kõrgekvaliteediline servatundlik debayer, mis on kombineeritud AI/ML-müra eemaldamise mudeliga, mis kõrvaldab peaaegu kogu debayerimisel tekkiva müra.
* **Kohandatud multispektraalsed indeksivalemid:** sisestage Chloros i rasterarvutitesse kohandatud multispektraalsed indeksid nii töötlemiseks kui ka piltide vaatamise katsetuskeskkonna jaoks.
* **Linux ja servarvutus:** käivitage Chloros Linux x86_64 ja ARM64 platvormidel, sealhulgas NVIDIA Jetsonil, välitöötlemiseks ja servarvutuseks. Vaadake [Linux i ülevaadet](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ hinnad ja registreerimine</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
