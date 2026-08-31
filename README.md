---
metaLinks: {}
---

# Alustamine

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros on tarkvararakendus, mille on välja töötanud [MAPIR](https://www.mapir.camera), mis on mõeldud multispektraalsete piltide töötlemiseks, MAPIR riistvara reaalajas juhtimiseks ja andurite andmete salvestamiseks. Chloros 1.2.0 toetab kogu MAPIR tooteperekonda:

* **Survey3 kaamerad** — töötlevad RAW+JPG-pildid kalibreeritud peegeldus- ja taimestikuindeksikaartideks. Vaata [Toetatud kaamerad](supported-cameras.md).
* **LATTICE-kaamerad** — ühendage GigE multispektraalsed kaameramoodulid reaalajas, kas üksikult või sünkroniseeritud mitme kaamera massiividena: eelvaade, pildistamine ja töötlemine kalibreeritud kiirguse ja peegeldusvõime toodeteks. Vaadake [LATTICE-osa](lattice/README.md).
* **DAQ-valgusandurid** — DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet) spektraalandurid: kalibreeritud reaalajas spektrid, `.daq` salvestused ja allapoole suunatud valgustatus peegeldusvõime töötlemiseks. Vaata [DAQ-osa](daq/README.md).

{% hint style="success" %}
**Uued funktsioonid versioonis Chloros 1.2.0**: LATTICE-kaamera ja andurite reaalajas juhtimine, DAQ-valgusandurite integreerimine, salvestusrežiimid ja salvestusseadmed, täielik LATTICE-radiomeetriline töötlemisprotsess, projektide automatiseerimine CLI/SDK-i abil ja palju muud. Vaata allpool olevat uuenduste loendit ja [Lae alla](download.md) muudatuste loend.
{% endhint %}

{% hint style="info" %}
**Kasutad Chloros-i koos AI-assistendiga?** See juhend on just selleks loodud. Suuna oma assistent järgmistele lehtedele:

* `https://mapir.gitbook.io/chloros/llms.txt` — masinloetav indeks kõigi lehtede kohta.
* Mis tahes lehekülg toores Markdown-vormingus — lisage `.md` selle URL-ile (nt `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI viide](reference/cli-reference.md) ja [SDK viide](reference/sdk-reference.md) — täielikud, täpsete väärtustega viitelehed, mis on kirjutatud LLM-i jaoks.

Näide käsust: *„Loe https://mapir.gitbook.io/chloros/reference/cli-reference.md, ja kirjuta seejärel skript, mis logib sisse ja töötleb kausta ~/flights/flight_001, muutes selle peegeldusvõimeliseks + NDVI GeoTIFF-failideks.”*

Täielik juhend: [Chloros kasutamine AI-abistajatega](ai-assistants.md).
{% endhint %}

***

## Uued funktsioonid versioonis Chloros 1.2.0

* **Kaamera reaalajas juhtimine — uus vahekaart „Kaamerad“.** Ühendage LATTICE-kaamerad ükshaaval või sünkroniseeritud mitme kaamera massiivina (PTP-ajasünkroniseerimine, riistvarapõhine pildistamine), koos reaalajas eelvaate ülekattete, sagedusribade histogrammide, nutika automaatse särituse, reaalajas indeksarvutaja ja rakendusesisesete kaamera tarkvarauuendustega.
* **Valgusandurid — uus vahekaart „Valgusandurid”.** Ühendage DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet) andurid; vaadake kalibreeritud spektreid reaalajas (W/m²/nm), salvestage `.daq`-failid oma projekti, valige kap-korrektsiooniprofiilid ja uuendage DAQ-E püsivara võrgu kaudu.
* **Salvestusrežiimid ja salvestid.** Ühekordne / pidev / intervall-salvestus ning ainult toorandmetele mõeldud kiireim salvestusrežiim; projektipõhine valik, milliseid kaameraid ja eksporditüüpe „Capture All” toodab; massiivsalmestid seiretasemel indeksvideote ja analüüsitasemel toorandmete seeriate jaoks koos offline-videokoostamisega.
* **LATTICE-töötlusprotsess.** Importige LATTICE-salvestuskaustad ja jagage iga toores kaader debayeringu, eelvaate, float32-kiirguse (W/m²/sr/nm) ja peegeldusvõime tulemusteks, kus iga tulemuse jaoks on eraldi lülitid. Peegeldusvõime võib pärineda kaadri sees olevast kalibreerimismärgist või DAQ-i allapoole suunatud valgusest; eksportidele rakendatakse massiivi joondamist; puuduv tehase kalibreerimine laaditakse automaatselt alla kaamera seerianumbri järgi.
* **Projektid mäletavad riistvara.** Ühendatud kaamerad ja valgusandurid salvestatakse koos projektiga (`cameras.json` / `sensors.json`) ning ühenduvad uuesti oma salvestatud seadetega, kui projekt uuesti avatakse. Vaata [GUI: Projektid](projects.md).
* **Pildivaataja uuendused.** Kursori pikseli/indeksi väljund koos failipõhise õige peegeldusastme skaalimisega, kihtide histogrammid, GSD-binningu liugur, võrgustiku režiimid „Per Trigger“ ja „Per Camera“, LATTICE-tootevaated ning indeksi/LUT-i sandbox-eksport kettale.
* **CLI ja SDK, oluliselt laiendatud.** Uued käskude perekonnad `lattice`, `daq pool-*`, `project` ja `time-sync`; uued `process` valikud (`--input-level`, tootekohased lülitid, `--reflectance-source`, massiivi joondamise lipud); SDK nutikad ühenduskäepidemed (`connect_camera` / `connect_array` / `connect_daq_sensor`), mis käivitavad tagapõhja automaatselt; `open_project()` automatiseerimine; SDK-ratas on lisatud paigaldajatele ja avaldatud PyPI-s nimega `chloros-sdk`.
* **Aus veasemantika.** `chloros-cli process` käivitus, mis taotles tooteid, kuid ühtegi ei salvestanud, annab nüüd selge veateate ja lõpetab töötamise nullist erineva tulemusega; edukad käivitused teatavad, kui palju pilditooteid nad salvestasid.
* **Uus väljundipaigutus.** Tooted salvestatakse `<project>/<camera>/<format>/<Product>_Images/`-kaustadesse ja säilitavad allika failinime — toodet identifitseerib kaust, mitte failinime laiend. Vaata [Väljundpildi formaadid](output-image-formats.md).
* **Rohkem sisendvõimalusi, plaane ja keeli.** `.dng` sisendite tugi; kõik 38 liidese keelt on täielikult täidetud; plaani kohta kehtivad riistvarapiirangud, mille raames on tasuta (ilma sisselogimiseta) kasutamiseks lubatud kuni 4 kaamerat ja 2 valgussensorit.
* **Usaldusväärsus.** Töötlemise peatamine lõpeb korrektselt koos ausa käivituskokkuvõttega, mitme kaameraga projektid ekspordivad iga kaamera andmed ning paigaldaja uuendused ei logi teid enam välja.***

Chloros on saadaval kolmes kasutajaliidese vormis:

## Chloros: töölaua graafiline kasutajaliides

Eraldi aken kõigi funktsioonidega, sealhulgas vahekaardid „Kaamerad“ ja „Valgusandurid“ reaalajas. _Ainult Windows._

## [Chloros CLI: käsurealiides](CLI.md)

Käsurealt käivitatav paketttöötlus koos reaalajas käskudega `lattice`, `daq pool-*`, `project` ja `time-sync`. Ideaalne automatiseerimiseks, skriptide koostamiseks ja peata tööks. Saadaval **Windows, Linux amd64 ja Linux arm64 (NVIDIA Jetson)** versioonides. _CLI-le juurdepääsuks on vaja tasulist Chloros+ taset._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmeeritav Python-liides automatiseerimiseks ja kohandatud töövoogudeks: täielik töötlemisprotsess, reaalajas kaamera-/maatriksi-seansid, DAQ-andurite seansid ja salvestatud projektide automatiseerimine. Paigaldatakse koos desktop/CLI paketiga ja avaldatakse ka nimetusega `pip install chloros-sdk`. _API-le juurdepääsuks on vaja tasulist Chloros+ taset._

***

## Toetatud platvormid

| Platvorm | Graafiline kasutajaliides | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Jah | Jah | Jah |
| **Linux amd64 (x86_64)** | Ei | Jah | Jah |
| **Linux arm64 (NVIDIA Jetson)** | Ei | Jah | Jah |

Linux paigaldusjuhiseid leiate jaotisest [Linux ja servaarvutus](linux/linux-overview.md).

***

## Alustamine kolmes etapis

1. **Paigaldamine** — laadige alla ja käivitage oma platvormile mõeldud paigaldusprogramm. Vaadake [Allalaadimine](download.md).
2. **Logi sisse (valikuline GUI puhul)** — GUI töötleb pilte tasuta ka ilma kontota. [Chloros+ sisselogimine](chloros+-login.md) avab juurdepääsu paralleelsele töötlemisele, GPU-kiirenduse, suuremad seadmepiirangud ning juurdepääsu CLI/SDK-le.
3. **Loo oma esimene projekt** — ava Chloros, loo [uus projekt](projects.md), [lisage oma pildid](processing-images-gui/adding-files-to-a-project.md) ja [alustage töötlemist](processing-images-gui/starting-the-processing.md). Kui soovid selle asemel juhtida reaalajas töötavat riistvara, ava vahekaart „Kaamerad“ või „Valgusandurid“ — vaata [GUI: navigeerimine](navigation.md).***

## Chloros+

Kuigi Chloros on enamiku ülesannete jaoks tasuta kasutatav, võite avastada, et soovite rohkem. Siis võib Chloros+ tasuline litsents teile kasuks tulla. Chloros+ litsentsiga saate avada uusi funktsioone, nagu:

* **Mitmeprotsessiline töötlemine**: kiirendab oluliselt suuremate projektide pilditöötlust, töödeldes pilte töötlusketis samaaegselt.
* **GPU (CUDA) kiirendus**: kasutage ära tänapäeva suurema GPU-mälu mahtu, et pilditöötluse töötlemisahelat veelgi kiirendada. Parimate tulemuste saavutamiseks soovitame 4 GB või rohkem VRAM-i.
* **Chloros+**[**CLI**](CLI.md)**Juurdepääs**: käivita Chloros+ käsurealt, et automatiseerida ja integreerida see oma tarkvarasse. Saadaval kõikidel tasulistel tasemetel; rakendatakse serveri poolel.
* **Chloros+**[**API**](api-python-sdk.md)**Juurdepääs:** käivitage Chloros+ käskudest Python, et saavutada programmiline juhtimine, mis võimaldab sujuvat integreerimist teie uurimistöö protsessidega, andmete analüüsi töövoogudega ja kohandatud rakendustega. Saadaval kõigil tasulistel tasemetel; rakendatakse serveri poolel.
* **Kõrgemad riistvarapiirangud**: ühendage korraga rohkem kaameraid ja valgussensoreid. Ilma sisselogimiseta ühendab graafiline kasutajaliides kuni 4 kaamerat ja 2 DAQ-valgussensorit; tasulised paketid suurendavad mõlemat piirmäära:

| Pakett | Kaamerad | DAQ-valgussensorid |
| --- | --- | --- |
| Iron (tasuta, sisselogimata) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Mitme seadme kasutamine**: iga Chloros+ litsents võimaldab registreerida 2 või enam seadet. Kasutage oma MAPIR pilvekontot registreeritud seadmete haldamiseks. Lisage tugi rohkematele seadmetele, uuendades oma Chloros+ litsentsi.
* **Täiustatud tekstuuritundlik debayer-meetod:** kõrgekvaliteediline servatundlik debayer, mis on kombineeritud AI/ML müra vähendamise mudeliga, mis eemaldab peaaegu kogu debayerimisel tekkiva müra.
* **Kohandatud multispektraalsed indeksivalemid:** sisestage kohandatud multispektraalsed indeksid Chloros rasterarvutitesse nii töötlemiseks kui ka piltide vaatamise katsetuskeskkonna jaoks.
* **Linux ja servarvutus:** käivitage Chloros platvormidel Linux x86_64 ja ARM64, sealhulgas NVIDIA Jetsonil, välitööde ja servatöötluse jaoks. Vaata [Linux ülevaadet](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ hinnad ja registreerimine</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
