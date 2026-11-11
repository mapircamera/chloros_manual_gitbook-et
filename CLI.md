# CLI: käsurea

> **Täielik juhend:**[CLI-juhend](reference/cli-reference.md) dokumenteerib**iga alamkäsu iga lipu** ja on optimeeritud AI-abistajate jaoks — kleepige selle URL oma abistajasse ja küsige töötavat käsku: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Nõuanne AI-tööriistadele:** käesoleva juhendi iga lehekülg on kättesaadav toores Markdown-vormingus, kui lisada selle URL-ile `.md` (nt `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), ning `https://mapir.gitbook.io/chloros/llms.txt` indekseerib kogu juhendi LLM-i kasutamiseks.

> **Kas soovid, et su assistent käivitaks Chloros'i ise?** `chloros-mcp`, MCP-server, mis on installitud koos Python'i SDK-ga, töötab koos Claude Desktopi, Claude Code'i, Cursori ja teiste MCP-klientidega: vaata [Chloros'i kasutamine AI-assistentidega](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Mis on CLI


`chloros-cli` on käsurealiides, mis kasutab sama töötlemismootorit, mida kasutab ka Chloros i töölauarakendus. Tegemist on kerge HTTP-kliendiga, mis suhtleb Chloros i tagapõhjaga (kohalik server `127.0.0.1:5000`-is) — enamik käske käivitab tagapõhja automaatselt, seega piisab skriptile ühestainsast `chloros-cli process …`-kutsest. Tagapõhi jääb seejärel järgmise käsu jaoks töötama: see peatub käskluse `chloros-cli logout` korral (välja arvatud juhul, kui rakendus on avatud või käimas on käivitamine või koostamine), käskluse `chloros-cli backend stop` korral või iseenesest 30 minuti möödudes, kui puuduvad kliendid, käivitused või ühendatud seadmed. `chloros-cli backend status` näitab seda.

See töötab **Windows 10/11 (x64)**ja**Linux (x86_64 ning NVIDIA Jetson arm64 JetPack 6-l)** operatsioonisüsteemides, mis tahes terminalis, ilma graafilise kasutajaliidese vajaduseta. Kontrollige installi järgmise käskuga:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Käskude rühmad lühidalt:

* **Töötlemine ja konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 keelt — vt [Toetatud keeled](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (ainult Linux/Jetson)
* **Töötav riistvara** — `lattice` (LATTICE kaamera juhtimine, üle 45 alamkäsu), `daq pool-*` (DAQ valgussensorid), `time-sync` (PTP)
* **Automatiseerimine** — `project` (salvestatud Chloros-projekti juhtimine ilma kasutajaliideseta, sealhulgas YAML-salvestusretseptid)

Teadmiseks väärt üldised valikud: `--port N` (tagapõhja port, vaikimisi `5000`), `-v/--verbose`, `--restart` (tagapõhja sundtaaskäivitamine), `--backend-exe PATH`. Täieliku loetelu leiate [CLI-viitest](reference/cli-reference.md).

***

## Paigaldamine

CLI **on kaasas Chloros i paigaldajaga** kõikidel platvormidel – eraldi CLI-i allalaadimist ei ole. Lae paigaldaja alla [Allalaadimise](download.md) lehelt.

### Windows


Paigaldaja paigutab CLI järgmisesse asukohta:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

ja lisab selle kausta teie süsteemi `PATH` ühe korra (uuendamine eemaldab ka varasemate paigaldajate poolt jäetud dubleeritud kanded) — **avage pärast paigaldamist uus terminal**, et uuendatud `PATH` saaks tuvastatud. Paigaldaja salvestab paigalduskataloogi ka käivitusskriptid (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) ning**Chloros i CLI** Start-menüü kiirklahvi, millest igaüks avab terminali, kus `chloros-cli` on kasutusvalmis.

### Linux


Paigalda oma arhitektuurile sobiv `.deb` koos `apt`-iga (mitte `dpkg -i`, mis jätab täitmata Python i SDK soovitatud nõuded):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

See installib `chloros-cli` kuni `/usr/bin/chloros-cli` (juba versioonis `PATH`), tagapõhi ja CLI-programmid lahti pakitud versioonidesse `/usr/lib/chloros/backend/` ja `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` on käivitaja, mis käivitab backend'i), Arena SDK-i käitamiskeskkond, mida backend vajab LATTICE-kaamerate jaoks, ning süsteemi Python-SDK `python3`. Täpsemad üksikasjad leiate [Linux i paigaldusjuhendist](linux/linux-installation.md).

### Kontrollimine

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Sisselogimine ja litsentsimine

CLI-le (ja Python SDK-le) juurdepääs eeldab **tasulist Chloros+ paketti**— see on olemas igas tasulises paketis, tasuta paketis aga mitte. Piirangut rakendab**serveri poolel** backend, mitte CLI-i binaar: sisselogimata kutsele vastatakse veakoodiga `401 AUTH_REQUIRED`, ja sisselogitud kasutaja päring tasuta paketil veakoodiga `403 PLAN_UPGRADE_REQUIRED`, olenemata sellest, kas see pärineb `chloros-cli`-ist, SDK-st või käsitsi loodud HTTP-kliendist. Uuendage aadressil [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Logige sisse **üks kord iga seadme kohta**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Erimärke sisaldavad paroolid**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` moonutatakse käsurea poolt (CLI tuvastab selle 401-vea korral ja proovib automaatselt uuesti, kuid ühekordsed jutumärgid aitavad seda probleemi täielikult vältida).
{% endhint %}

Seanss salvestatakse vahemällu `~/.chloros/user_session.json` ja toimib offline-režiimis kogu paketi armuaja jooksul (kuupakettide puhul 30 päeva, aastapakettide puhul kuni kehtivusaja lõpuni). `chloros-cli status` töötab isegi ilma tasulise plaanita, seega on keeldumise põhjus alati nähtav.

{% hint style="danger" %}
**Kavandate peata tööd? Logige esmalt sisse.**Tagapõhja käivitav käsk (`process`, `status`, `export-status`, …) ei lõpe kiiresti, kui seda käivitatakse**ilma vahemällu salvestatud sessioonita**— see läheb üle interaktiivsele `Email:` / `Password:`-käsureale stdin-i kaudu. Seetõttu**jääb järelevalveta cron-ülesanne või CI-etapp sisendi ootamisel kinni**. Käivita masinal enne mis tahes ülesande planeerimist üks kord käsk `chloros-cli login EMAIL 'PASSWORD'`.
{% endhint %}

***

## Teie esimene töötlemiskäivitus

Suunake `process` salvestuste kausta poole — see tuvastab automaatselt Survey3 (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` või nende kombinatsiooni:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Edusammude vood on reaalajas nähtavad iga toru niidi kohta (avastamine, analüüsimine, töötlemine, eksportimine) ning edukas käivitus lõpeb sellega, et teatatakse, kui palju pilditooteid kirjutati: `Image products written: N`, millele järgneb `(plus M clamp masks)`, kui käivituse käigus salvestati ka [kinnitusmaskid](output-image-formats.md#clamp-masks) (neid ei loeta toodeteks).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Kui Chloros-tagapõhi juba töötab (näiteks töölaua rakenduse oma), kasutab `process` seda ja avab selles oma projekti. See keeldub, kui rakendus on avatud projektis, mille rakendus avas: töötle rakendusest või sulge see projekt. Kui eelmine käivitus on veel lõpetamisel, ootab ta seda kuni 15 sekundit enne keeldumist. `--restart` sunnib kasutama uut backendit. Linuxis ei taaskäivitatud kunagi backendit, mida käitab systemd-teenus: `process` kasutab seda sellisena, nagu see on, ja kui ta muidu seda taaskäivitaks (`--restart` või import, mis midagi ei leidnud), kuid väljastab veateate `sudo systemctl restart <unit>` ja lõpetab töö `1`.

### Kuhu väljundid salvestatakse

`process` kirjutab tulemused **projekti kausta**, mitte sinu sisendkausta:

* Kui `-o` puudub: projekt luuakse teie vaikimisi projektikausta alla (mida jagatakse graafilise kasutajaliidesega; haldage seda `get-project-folder` / `set-project-folder` abil; kui seda pole määratud, siis `Documents/MAPIR/Chloros Projects` teie kasutajakausta), mille nimeks on `-n/--project-name` või ajamärge (`YYYYMMDD_HHMMSS`), kui seda pole märgitud.
* `-o PATH` puhul: see kaust **on** projekti kaust. Kui see sisaldab juba `project.json`-i, luuakse selle asemel `_1`/`_2`… lõpuga kaust selle asemel, et seda üle kirjutada. Suhteline `PATH` võetakse suhteliselt kausta suhtes, millest te `chloros-cli` käivitate.

{% hint style="info" %}
**Suhtelised teed (muudetud versioonis 1.3.0).** Iga tee, mille tagapõhi avab — `-o`/`--output` mis tahes käsu puhul (sealhulgas salvestuskäskude vaikimisi `output`), `--input`, `--burst-dir`, `--profile`, `--save` ja `--mask` — lahendatakse selle kausta suhtes, kust käivitate `chloros-cli`, seega kirjutab `-o captures/` faili `<current folder>/captures/`. Versioonis 1.2.0 lahendas tagapõhi need omaenda installikausta suhtes, mis lõppes veaga „Juurdepääs keelatud”. Kaugtagapõhi (`CHLOROS_BACKEND_URL`) puhul saadetakse teed täpselt nii, nagu need on sisestatud, ja need lahendatakse sellel masinal.
{% endhint %}

{% hint style="warning" %}
**Windows: hoidke projektikausta lühike, kui pikad teed pole sisse lülitatud.** Kui Windowsis on pikad teed välja lülitatud (Windows i vaikimisi seade), ei tohi ükski failitee olla pikem kui 260 tähemärki ning tooted asuvad kuni umbes 130 tähemärki allpool projektikausta. `process` kontrollib seda enne käivitumist: liiga sügav projektikaust lükatakse tagasi veateatega **„Projektikaust on selle arvuti jaoks liiga sügav“**, kuvatakse pikim sobiv projektikaust ning juhised pikkade teede sisselülitamiseks – muidu ei toimu mingit töötlemist. Kasutage lühemat `-o` (või `set-project-folder`) või lülitage sisse Windows i pikad teed (paigaldaja valik **Luba Windows i pikad teed**, mis on vaikimisi märgitud) ja taaskäivitage Chloros.
{% endhint %}

Projekti sees on tooted rühmitatud **kaamera järgi, seejärel failivormingu järgi**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kaamera kaust on LATTICE puhul `LATT-<sensor>-<lens>-F<filter>` (vastab pildi EXIF-andmetele `Model`) ja Survey3 puhul `<model>_<filter>` (nt `Survey3N_RGN`) puhul, mis on tehtud kaameraga Survey3. Failivormingu kaust järgib `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` või `tiff32` kaustale `TIFF (32-bit, Percent)`.

Multispektraalse M3C-kaamera puhul võib formaadikausta ilmuda ka kaust `clamp_masks/`, tootekastude kõrval (näiteks `tiff16/clamp_masks/Reflectance_Calibrated_Images/` ja `tiff32/clamp_masks/Radiance_Images/`). See sisaldab kiirguse ja peegeldusvõime toodete pikselipõhiseid maske, milles lainepikkuse segunemise eemaldamine on fikseerinud negatiivse väärtuse nullini — vt [Fikseerimismaskid](output-image-formats.md#clamp-masks). Ükski tootekaust ei sisalda kunagi maski.

{% hint style="info" %}
**Iga eksporditud toode säilitab SOURCE-faili nime.**`capture_..._raw.tif` kiirguse eksport kannab endiselt nime `capture_..._raw.tif` — see asub lihtsalt kataloogis `tiff32/Radiance_Images/`.**Toote identifitseerimiseks kasutatakse kausta, mitte failinime**, seega otsige kataloogi, mitte `*radiance*`-lõppu.
{% endhint %}

### Valikud, mida tegelikult kasutate

| Lipuk | Vaikimisi | Mida see teeb |
| --- | --- | --- |
| `-o, --output PATH` | projekti vaikimisi kaust | Projekti kausta asukoht (vt eespool). |
| `-n, --project-name NAME` | ajamärge | Projekti nimi. |
| `--format FMT` | `TIFF (16-bit)` | Üks järgmistest: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros ei suurenda resolutsiooni: kõigil 8-bitistel piltidel kirjutatakse sügavama formaadiga pilt `TIFF (8-bit)`-na ja seda märgitakse üks kord. |
| `--indices NAME [NAME ...]` | puudub | Eksporditavad taimestiku indeksid (vt [Taimestiku indeksid](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neurovõrgu abil debayering, aeglasem, kõrgeim kvaliteet (Chloros+, vajab CUDA GPU-d: selle puudumisel kasutatakse käivitamisel `standard`; keelatud LATTICE-pildistuste puhul, mis saavad alati `standard`). |
| `--vignette / --no-vignette` | sisse lülitatud | Vignette-korrektsioon. |
| `--reflectance / --no-reflectance` | sisse lülitatud | Peegelduskalibreerimine; LATTICE puhul on see ka peegeldusvõimsuse lüliti. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Sundi LATTICE TIFF-failide töötlemisahela alguspunkti. |

Kõige muu kohta — sihtmärgi tuvastamise häälestamine, PPK, eksponeerimispinnid, massiivi joondamise lipukesed — vaata [CLI-viite jaotist `process`](reference/cli-reference.md).

***

## Eksporditava valimine (LATTICE tooted)

LATTICE töötlemine hargneb **ühe läbimisega kõikidele asjakohastele toodetele**. Kõik neli tootekohast lülitit on**vaikimisi sisse lülitatud**; kasuta vormi `--no-`, et ühe välja lülitada:

| Lüliti | Toode |
| --- | --- |
| `--debayered` | Lineaarne demosaik → `Debayered_Images/`: kalibreerimispaketi tume nihke eemaldamine, seejärel servadest lähtuv demosaik, ilma lineaarsuse korrigeerimise, tasase välja või segamise eemaldamiseta — sama fail, mille salvestamine toimub `debayered` tasemel. Säilitatakse anduri algses orientatsioonis (salvestatud XMP-s). |
| `--preview` | Eelvaate kuvamine (valgebalanss + gamma; multispektraalsete piltide puhul valevärvide venitamine) → `Preview_Images/` |
| `--radiance` | float32 kiirgus, W/m²/sr/nm → `Radiance_Images/` (alati `tiff32/`) |
| `--reflectance` | uint16 peegeldusvõime, Pix4D-valmis → `Reflectance_Calibrated_Images/` |

RGB-peakaamerad edastavad alati ainult debayereeritud + eelvaate — lairibaandilise anduri puhul ei ole sagedusribapõhine kiirgusvõimsus/peegeldusvõime oluline, seega on need lülitid nende puhul mõttetud. Survey3 `.raw` eirab neid lüliteid ja järgib standardset peegeldusvõime/sihtmärgi teekonda.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (vaikimisi `auto`) valib peegeldusvõime võrdlusaluse: `auto` teeb kvaliteedikontrolli läbinud kaadris oleva [kalibreerimissihiku](calibration-targets.md) absoluutseks võrdluspunktiks ja kasutab tagavarana DAQ-i valgusanduri allapoole suunatud valguse jaotust (ρ = π·L/E), kui sihtmärki pole (lipuga kvaliteedikontrolli läbinud sihtmärk konkureerib DAQ-iga mõõdetud tulemuse osas); `target` on range (DAQ-i asendamine puudub); `daq` on DAQ-põhine. Ühiku kohta mõõdetud sihtmärgi skaneeringuid saab esitada koos `--target-reflectance-dir`-iga.

{% hint style="info" %}
**Peegeldusvõime pikslite lugemine:**DN, mille tähendus on ρ = 1,0, on**allika kohta** — LATTICE-failid lisavad XMP-sse märgise `Chloros:PixelScale=32768`; Survey3-failid kasutavad väärtust 65535 ja neil puudub märge `Chloros:PixelScale` (kuid neil on muid töötlemismärke `Chloros:*`). Lugege märget ja jagage sellega, selle asemel et eeldada konstanti. Üksikasjad ja üks tahtlik skaalata äärejuhtum on toodud [CLI-viites](reference/cli-reference.md).
{% endhint %}

**Töötlemine algab alati `raw`-ist.** Tuletatud tooted (debayeringu tulemused / kiirguse / peegeldusvõime ekspordid) ei suunata kunagi tagasi töötlemisahelasse — nende uuesti importimine ja töötlemine rakendaks kalibreerimisarvutusi kahekordselt, seetõttu jätab Chloros need vahele ja teavitab sellest. `--input-level` on tahtlik pääsetee, kui on tõesti vaja sisenemispunkti jõuga määrata.

***

## Kui töötlemine ebaõnnestub

Alates versioonist 1.2.0 annab `process` selge veateate, selle asemel et „õnnestuda“ ilma tulemust näitamata:

* Käivitus, mis **nõudis tooteid, kuid ei kirjutanud ühtegi**— ainult `project.json` ja `calibration_data.json` — väljastab veakoodi `Processing finished but wrote no image products.` ja**lõpetab töötamise nullist erineva tulemusega**, seega suudavad skriptid seda tuvastada. Tavapärased põhjused: sisendkausta ei tunnistatud pildistamisena (kontrollige paigutust ja `--input-level`) või kõik taotletud tooted olid nende kaamerate jaoks kohaldamatud (nt radiaantsuse/peegeldusvõime taotlemine ainult RGB-kaameratelt).
* **Tahtlikult ainult metandmetega käivitus** (kõik tooted välja lülitatud, ilma `--indices`-ita) on ikkagi edukas — tühi pildiväljund on selles juhul õige tulemus.
* Käivita uuesti koos `--verbose`-iga ja kontrolli tagapõhja logist ridu `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, mis selgitavad kaamerapõhiseid vahelejätmisi.
* Windowsis, kus pikad teed on välja lülitatud, lükatakse 260-tähemärgilise teepiirangu ületav liiga sügav projektikaust tagasi **enne** töötlemise algust, kasutades selle asemel pikimat sobivat projektikaustu (vt [Kuhu väljundid salvestatakse](#where-the-outputs-land)).
* Salvestuskäsud (`lattice capture`, `lattice array-capture` ja muud salvestuspered) nimetavad iga faili, mida nad kirjutada ei suutnud, ja lõpetavad töö koodiga `1`; korduvad salvestused peatuvad, kui kett on täis.

Lõpetamiskoodid: `0` õnnestus · `1` üldine viga · `2` argumentide viga · `130` katkestati klahvikombinatsiooniga Ctrl+C.

***

## Taimestiku indeksid

Käivita `--indices` ühe või mitme eelseadistatud nimega; iga indeks salvestatakse omaette `<INDEX>_Index_Images/` kausta:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

22 eelseadistatud nime, mida `process --indices` aktsepteerib:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Projektile lisatakse nimi, mida selles veel ei ole. Nimi, mis ei ole eelseadistatud või mida kaamerate sagedusalad ei suuda luua, jäetakse vahele ja selle kohta kuvatakse hoiatus. Kui CLI-liides ei suuda projekti praegust indeksiloendit lugeda, jätab ta loendi muutmata ja annab hoiatuse, selle asemel et seda asendada.

{% hint style="warning" %}
**Indeksiloendeid on kolm – ärge neid segage.**GUI-liidese „Project Settings“ rippmenüüs on 27 valemit (lisab `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – need viis on mõeldud ainult kasutajaliidesele ja**ei** kehti `--indices` puhul). Reaalajas/offline-režiimis kasutatav käsk `lattice index --preset` kasutab oma eraldi 22-eelseadistusega nimekirja. Valemid ja lainepikkuste arvutused on dokumenteeritud dokumendis [Multispektraalsete indeksite valemid](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ valgusandurid: lühike ülevaade

`daq pool-*`-seeria juhib MAPIR i DAQ-spektraalsensoreid (DAQ-U USB-ühenduse kaudu, DAQ-M BLE-ühenduse kaudu, DAQ-E Etherneti kaudu) tagapõhja püsiva reservi kaudu — GUI, CLI ja SDK jagavad ühte reaalajas käepidet. **`pool-*` on tarnitud CLI-s toetatud DAQ-tee**; muud `daq` alamkäskud, millele võite viiteid näha, on MAPIR i sisemised, ainult allikana kasutatavad liidesed, mis lõpetavad töö selge veateatega, mis suunab teid `pool-*`-i juurde.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` ilma `--duration`-ita töötab kuni `pool-record --stop`-ini; vaikimisi väljundkataloog on `~/Documents/DAQ Live View/` **tagapõhja masinal**. Katteparandusprofiil valitakse ühenduse loomise ajal (`--cap-id`, tagapõhja vaikimisi `sunshine_cosine`) ja seda saab reaalajas vahetada `pool-set-cap` — piirväärtuste profiile ja anduri kalibreeritud mõõtevahemikku käsitletakse käesoleva juhendi DAQ-peatükkides.

{% hint style="warning" %}
**DAQ-E mitme võrgukaardiga hostis:** esimene `pool-connect --eth` automaatne avastamine pärast käivitamist võib ebaõnnestuda isegi töökorras anduri puhul. `--eth-host <ip-or-hostname>` on usaldusväärne variant – kasutage seda alati, kui avastamine ei anna tulemusi.
{% endhint %}

***

## LATTICE-kaamerad, PTP ja projekti automatiseerimine

`lattice`-perekond (üle 45 alamkäsku) hõlmab LATTICE-kaamerate tööd algusest lõpuni: avastamine, üksikud pildistamised, püsivad sünkroniseeritud massiivid koos GUI nutika ettevalmistus-ühendamisvooga, reaalajas brauseri eelvaade, joondamine, indeksimatemaatika ja host-NIC-i diagnostika. Näide:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Lisaks sellele: `chloros-cli time-sync` annab aru PTP-grandmasterist, mida Chloros-host käitab (LATTICE-kaamerad ja DAQ-E-andurid töötavad selle alluvuses seadmetevaheliste ajamärkide saamiseks), ning `chloros-cli project` avab salvestatud Chloros-projekti ja juhib selle kaameraid, andureid ja andurimassiive ilma graafilise kasutajaliideseta — sealhulgas skriptipõhiseid YAML-pildistamisretsepte.

Need kolm perekonda (`lattice`, `project`, `daq pool-*`) on ka ainsad, mis toetavad `CHLOROS_BACKEND_URL`-i **kaugjuhtimiseks**; põhilised käsud on alati suunatud kohalikule masinale.

Täielikud juhendid on käesoleva käsiraamatu LATTICE-peatükkides; kõik lipud on toodud [CLI-viites](reference/cli-reference.md).

***

## Veaotsing: 5 levinuimat

| Sümptom | Lahendus |
| --- | --- |
| `Login required` või ajastatud ülesanne jääb kinni `Email:`-käsureale | Käivita `chloros-cli login EMAIL 'PASSWORD'` üks kord sellel arvutil — käsud, millel puudub vahemällu salvestatud seanss, töötavad interaktiivselt, mitte ei lõpe kohe ebaõnnestumisega. |
| `backend unreachable` | Käivita Chloros i töölaua rakendus või käivita tagapõhi ise: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` Windowsis (sinu installikausta `backend` kaust; seda pole `PATH`-is), `chloros-backend` Linuxis. `chloros-cli backend status` näitab, kas mõni neist töötab; `lattice`/`project`/`daq pool-*` ei käivita seda kunagi. Kui suunate need kaug-backendile, kontrollige `CHLOROS_BACKEND_URL`. |
| Massiivi ühendus blokeeritud: `FRAMES WILL DROP` / `Reduce ROI to enable` | Hosti võrgukaardi vastuvõturõngas on taastatud vaikimisi seadistustele — see on peamine põhjus, miks varem töötanud seadmestik keeldub ühendusest, tavaliselt pärast võrgukaardi draiveri uuendamist. Käivita `chloros-cli lattice network --fix` **õigustega** terminalist (või määra `ReceiveBufferLen=256`, `PendingReceives=64`); vaata viite jaotist *Host NIC Setup & Tuning*. |
| `daq` alamkäsk lõpetab töö: „nõuab täielikku DAQ-paketti…“ | Kasutage käske `daq pool-*`, mis juhivad sama andurit tagapõhja kaudu: ühendamine, voogedastus, salvestamine, andmekogumite valik ning DAQ-E puhul selle võrguseaded, taaskäivitamine ja püsivara uuendused (või `chloros_sdk.connect_daq_sensor()` Python ist). |
| Jetson kuvab enne suurte kaustade avamist vahemälumälude hoiatuse | Lisage failipõhine vahemälumälu — CLI kuvab täpsed käskud `fallocate`/`swapon`, mida tuleb käivitada. |

***

## Abi saamine

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Iga lipuk, iga alamkäsk:** [CLI-viide](reference/cli-reference.md)
* **Python-ekvivalent:** [Python SDK](api-python-sdk.md) ja [SDK Reference](reference/sdk-reference.md)
* **Tugi:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
