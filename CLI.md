# CLI : Käsurea

> **Täielik viide:**[CLI viide](reference/cli-reference.md) dokumenteerib**iga alamkäsu iga lipu** ja on optimeeritud tehisintellekti abiliste jaoks — kleebi selle URL oma abilisse ja küsi töötavat käsku: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Nõuanne AI-tööriistade jaoks:** käesoleva juhendi iga lehekülg on kättesaadav toores Markdown-vormingus, kui lisada selle URL-ile `.md` (nt `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), ning `https://mapir.gitbook.io/chloros/llms.txt` indekseerib kogu juhendi LLM-i kasutamiseks.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->
## Mis on CLI

`chloros-cli` on sama töötlemismootori käsurealiides, mida kasutab ka töölaua rakendus Chloros. Tegemist on kerge HTTP-kliendiga, mis töötab Chloros-tagapõhja (`127.0.0.1:5000`-i kohalik server) kaudu — enamik käske käivitab tagapõhja automaatselt, seega piisab skriptile ühestainsast `chloros-cli process …`-kutsest.

See töötab **Windows 10/11 (x64)**ja**Linux (x86_64 ning NVIDIA Jetson arm64 JetPack 6-l)** keskkondades, mis tahes terminalis, ilma et oleks vaja graafilist kasutajaliidest. Kontrollige oma installi järgmise käskuga:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Käskude rühmad lühidalt:

* **Töötlemine ja konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 keelt — vt [Toetatud keeled](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (Linux/ainult Jetson)
* **Töötav riistvara** — `lattice` (LATTICE kaamera juhtimine, üle 45 alamkäsu), `daq pool-*` (DAQ valgusandurid), `time-sync` (PTP)
* **Automatiseerimine** — `project` (salvestatud Chloros projekti juhtimine ilma kasutajaliideseta, sealhulgas YAML-pildistamisretseptid)

Teadmiseks väärt üldised valikud: `--port N` (tagapõhja port, vaikimisi `5000`), `-v/--verbose`, `--restart` (tagapõhja sundtaaskäivitamine), `--backend-exe PATH`. Täieliku loetelu leiate [CLI-i viitest](reference/cli-reference.md).

***

## Paigaldamine

CLI **on kaasas Chloros paigaldusprogrammis** kõikidel platvormidel — eraldi CLI-i allalaadimist ei ole. Lae paigaldusprogramm alla [Allalaadimise](download.md) lehelt.

### Windows

Paigaldusprogramm paigutab CLI järgmisesse asukohta:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

ja lisab selle kausta teie süsteemi `PATH` — **avage pärast installimist uus terminal**, et uuendatud `PATH` leitaks. Installija paigutab ka käivitusskriptid (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) paigalduskataloogi ning**Chloros CLI** Start-menüü kiirklahvi, millest igaüks avab terminali, kus `chloros-cli` on valmis kasutamiseks.

### Linux

Paigalda oma arhitektuurile sobiv `.deb`:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

See paigaldab `chloros-cli` kuni `/usr/bin/chloros-cli` (juba versioonil `PATH`) ja tagapõhja versioonile `/usr/lib/chloros/chloros-backend`, koos LATTICE-kaamerate jaoks vajaliku Arena SDK-i tööajaga. Täpsemad üksikasjad leiate [Linux paigaldamisjuhendist](linux/linux-installation.md).

### Kontrollige

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Sisselogimine ja litsentsimine

CLI (ja Python SDK) juurdepääs eeldab **tasulist Chloros+ paketti**— see on olemas igas tasulises paketis, tasuta paketis aga mitte. Piirang rakendatakse**serveripoolelt** backend&#x27;i poolt, mitte CLI-i binaarfaili poolt: välja logitud kasutaja kõne lükatakse tagasi veakoodiga `401 AUTH_REQUIRED` ja sisse logitud kasutaja kõne tasuta tasemel veakoodiga `403 PLAN_UPGRADE_REQUIRED`, olenemata sellest, kas see pärineb `chloros-cli`-ist, SDK või käsitsi loodud HTTP-kliendist. Uuendage aadressil [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Logige sisse **üks kord iga arvuti kohta**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->
{% hint style="warning" %}
**Erimärke sisaldavad paroolid**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` moonutab shell (CLI tuvastab selle 401-vea korral ja proovib automaatselt uuesti, kuid ühekordsed jutumärgid vältivad seda probleemi täielikult).
{% endhint %}

Seanss salvestatakse vahemällu `~/.chloros/user_session.json` ja töötab offline-režiimis plaani armuaja jooksul (kuuplaanide puhul 30 päeva, aastaplaanide puhul kuni kehtivusaja lõpuni). `chloros-cli status` töötab isegi ilma tasulise plaanita, seega on keeldumise põhjus alati nähtav.

{% hint style="danger" %}
**Kavandate peata tööd? Logige esmalt sisse.**Tagapõhja käivitav käsk (`process`, `status`, `export-status`, …) ei ebaõnnestu kiiresti, kui seda käivitatakse**ilma vahemällu salvestatud sessioonita**— see lülitub stdin-i kaudu interaktiivsesse `Email:` / `Password:`-käsureale. Seetõttu**jääb järelevalveta cron-ülesanne või CI-samm sisendit oodates kinni**. Käivita masinal enne mis tahes ülesande planeerimist üks kord käsk `chloros-cli login EMAIL 'PASSWORD'`.
{% endhint %}

***

## Sinu esimene töötlemiskäivitus

Suunake `process` salvestatud andmete kausta – see tuvastab automaatselt Survey3 (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` või nende kombinatsiooni:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Progressi vood edastatakse reaalajas iga toru-niidi kohta (avastamine, analüüsimine, töötlemine, eksportimine) ning edukas käivitamine lõpeb aruandega, kui palju pilditooteid salvestati (`Image products written: N`).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Kuhu väljundid salvestatakse

`process` salvestab andmed **projektikausta**, mitte teie sisendkausta:

* Kui `-o` puudub: projekt luuakse teie vaikimisi projektikausta alla (mida jagatakse graafilise kasutajaliidesega; haldage seda `get-project-folder` / `set-project-folder`, varuvariant `~/Chloros Projects`), mille nimeks on `-n/--project-name` või ajamärge (`YYYYMMDD_HHMMSS`), kui see on välja jäetud.
* Kui kasutatakse `-o PATH`-i, **on** see kaust projektikaust. Kui see sisaldab juba `project.json`-i, luuakse selle asemel, et seda üle kirjutada, `_1`/`_2`…-lõpuga kaust.

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

Kaamera kaust on `LATT-<sensor>-<lens>-F<filter>` kaamera LATTICE jaoks (vastab pildi EXIF-andmetele `Model`) ning `<model>_<filter>` (nt `Survey3N_RGN`) kaamera Survey3 puhul. Formaatikausta järgib `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` või `tiff32` asemel `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Iga eksporditud toode säilitab ALGFAILI nime.**`capture_..._raw.tif`-i radiance-eksport kannab endiselt nime `capture_..._raw.tif` — see asub lihtsalt kataloogis `tiff32/Radiance_Images/`.**Toote identifitseerimiseks kasutatakse kausta, mitte failinime**, seega otsige kataloogi, mitte `*radiance*`-lõppu.
{% endhint %}

### Valikud, mida tegelikult kasutate

| Lipuk | Vaikimisi | Mida see teeb |
| --- | --- | --- |
| `-o, --output PATH` | projekti vaikimisi kaust | Projekti kausta asukoht (vt eespool). |
| `-n, --project-name NAME` | ajamärge | Projekti nimi. |
| `--format FMT` | `TIFF (16-bit)` | Üks järgmistest: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | puudub | Eksporditavad taimestiku indeksid (vt [Taimestiku indeksid](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neurovõrgu abil debayerimine, aeglasem, kõrgeim kvaliteet (Chloros+, NVIDIA GPU). |
| `--vignette / --no-vignette` | sisse | Vignettkorrektsioon. |
| `--reflectance / --no-reflectance` | sisse | Peegelduskalibreerimine; LATTICE puhul on see ka peegeldustoote lüliti. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | LATTICE TIFF-failide töötlemisahela alguspunkti sundimine. |

Kõige muu kohta — sihtmärgi tuvastamise häälestamine, PPK, ekspositsioonitapid, massiivi joondamise lipukesed — vaata [`process` jaotist viites CLI](reference/cli-reference.md).

***

## Eksporditava valimine (LATTICE tooted)

LATTICE töötlemine hargneb **ühe läbimisega kõikidele asjakohastele toodetele**. Kõik neli tootekohast lülitit on**vaikimisi sisse lülitatud**; ühe väljalülitamiseks kasutage vormi `--no-`:

| Lüliti | Toode |
| --- | --- |
| `--debayered` | Lineaarne demosaik → `Debayered_Images/` |
| `--preview` | Eelvaate kuvamine (valgebalanss + gamma; valevärvide venitus multispektraalsete piltide puhul) → `Preview_Images/` |
| `--radiance` | float32 kiirgusintensiivsus, W/m²/sr/nm → `Radiance_Images/` (alati `tiff32/`) |
| `--reflectance` | uint16 peegeldusvõime, Pix4D-valmis → `Reflectance_Calibrated_Images/` |

RGB peakaamerad edastavad alati ainult debayered + eelvaadet — laiaribalise anduri puhul ei ole sagedusribapõhine kiirgusvõimsus/peegeldusvõime oluline, seega on need lülitid nende puhul toiminguteta. Survey3 `.raw` eirab lülitusi ja järgib standardset peegeldusvõime/sihtmärgi teekonda.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (vaikimisi `auto`) valib peegeldusvõime võrdluspunkti: `auto` loob kaadris kvaliteedikontrolli läbinud [kalibreerimissihiku](calibration-targets.md) absoluutseks etaloniks ja kasutab sihtmärgi puudumisel tagavarana DAQ-i valgusanduri allapoole suunatud valguse jagunemist (ρ = π·L/E); `target` on range (ei kasuta DAQ-i asendust); `daq` tugineb DAQ-ile. Ühiku kohta mõõdetud sihtmärgi skaneeringuid saab esitada koos `--target-reflectance-dir`-iga.

{% hint style="info" %}
**Peegeldusvõime pikslite lugemine:**DN-väärtus, mis tähendab ρ = 1,0, on**allika kohta** — LATTICE-failid lisavad XMP-sse märgise `Chloros:PixelScale=32768`; Survey3-failid kasutavad väärtust 65535 (ja neil puuduvad `Chloros:*`-märgised). Loe silt ja jaga sellega, selle asemel et eeldada konstanti. Üksikasjad ja üks tahtlik skaalata äärejuhtum on toodud [CLI viites](reference/cli-reference.md).
{% endhint %}

**Töötlemine algab alati failist `raw`.** Tuletatud tooted (debayered/kiirgus/peegeldusvõime ekspordid) ei suunata kunagi tagasi töötlemisahelasse – nende uuesti importimine ja töötlemine rakendaks kalibreerimisarvutusi topelt, seega Chloros jätab need vahele ja teatab sellest. `--input-level` on tahtlik pääsetee, kui on tõesti vaja sisenemispunkti jõuga määrata.***

## Kui käivitamine ebaõnnestub

Alates versioonist 1.2.0 annab `process` selge veateate, selle asemel et näidata „õnnestumist“ ilma tulemusteta:

* Käivitus, mis **taotles tooteid, kuid ei kirjutanud ühtegi**— ainult `project.json` ja `calibration_data.json` — väljastab veakoodi `Processing finished but wrote no image products.` ja**lõpetab töötamise nullist erineva väärtusega**, seega skriptid suudavad seda tuvastada. Tavapärased põhjused: sisendkausta ei tunnistatud pildistamisena (kontrollige paigutust ja `--input-level`) või iga taotletud toode oli nende kaamerate jaoks sobimatu (nt radiance/peegeldusvõimet ainult RGB-kaameratelt).
* **Tahtlikult ainult metandmetega käivitus** (kõik tooted välja lülitatud, `--indices` puudub) on ikkagi edukas – tühja pildi väljund on selles juhul õige tulemus.
* Käivita uuesti koos `--verbose`-iga ja kontrolli tagapõhja logist `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` ridu, mis selgitavad kaamerapõhiseid vahelejätmisi.

Väljumiskoodid: `0` edukas · `1` üldine viga · `2` argumendiviga · `130` katkestatud klahvikombinatsiooniga Ctrl+C.

***

## Taimestiku indeksid

Käivitage `--indices` ühe või mitme eelseadistuse nimega; iga indeks salvestatakse omaette `<INDEX>_Index_Images/` kausta:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22 eelseadistatud nime, mida `process --indices` aktsepteerib:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Indeksiloendeid on kolm – ärge neid segage.**GUI-liidese rippmenüüs „Project Settings“ on 27 valemit (lisatud on `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – need viis on mõeldud ainult graafilise kasutajaliidese jaoks ja**ei** kehti `--indices` jaoks). Reaalajas/offline-režiimis töötav käsk `lattice index --preset` kasutab oma eraldi 22-eelseadistuste nimekirja. Valemid ja lainepikkuste arvutused on dokumenteeritud [Multispektraalsete indeksite valemites](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ-valgusandurid: lühike ülevaade

`daq pool-*`-seeria juhib MAPIR DAQ-spektraalandureid (DAQ-U USB-ühenduse kaudu, DAQ-M BLE kaudu, DAQ-E Etherneti kaudu) tagapõhja püsiva reservi kaudu — GUI, CLI ja SDK jagavad ühte reaalajas käepidet. **`pool-*` on tarnitud CLI-is toetatud DAQ-tee**; muud `daq` alamkäskud, millele võite viiteid näha, on MAPIR-i sisemine ainult allikana kasutatav pind ning need lõpetavad töö selge veateatega, mis suunab teid `pool-*`-i poole.

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

`pool-record` ilma `--duration`-ita töötab kuni `pool-record --stop`-ini; vaikimisi väljundkataloog on `~/Documents/DAQ Live View/` **tagapõhja masinal**. Katteparandusprofiil valitakse ühenduse loomise ajal (`--cap-id`, tagapõhja vaikimisi `sunshine_cosine`) ja seda saab reaalajas vahetada `pool-set-cap` — piirväärtuste profiile ja anduri kalibreeritud mõõtevahemikku on käsitletud käesoleva juhendi DAQ-peatükkides.

{% hint style="warning" %}
**DAQ-E mitme võrgukaardiga hostis:** esimene `pool-connect --eth` automaatne avastamine pärast käivitamist võib ebaõnnestuda isegi töökorras anduri puhul. `--eth-host <ip-or-hostname>` on usaldusväärne variant – kasutage seda alati, kui avastamine ei anna tulemust.
{% endhint %}

***

## LATTICE-kaamerad, PTP ja projekti automatiseerimine

`lattice`-seeria (üle 45 alamkäsu) hõlmab LATTICE-kaamerate tööd algusest lõpuni: avastamine, üksikud pildistamised, püsivad sünkroniseeritud massiivid koos GUI nutika ettevalmistus- ja ühendamisvooga, reaalajas brauseri eelvaade, joondamine, indeksite arvutamine ja host-NIC-i diagnostika. Näide:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Lisaks sellele: `chloros-cli time-sync` annab aru PTP-grandmasterist, mida käitab Chloros-host (LATTICE-kaamerad ja DAQ-E-andurid töötavad selle alluvuses seadmetevaheliste ajamärkide jaoks), ning `chloros-cli project` avab salvestatud Chloros projekti ja juhib selle kaameraid, massiive ja andureid ilma kasutajaliideseta — sealhulgas skriptitud YAML-salvestusretsepte.

Need kolm perekonda (`lattice`, `project`, `daq pool-*`) on ka ainsad, mis toetavad `CHLOROS_BACKEND_URL`-i **kaugjuhtimiseks**; põhikäsud on alati suunatud kohalikule masinale.

Täielikud juhendid on käesoleva käsiraamatu LATTICE-peatükkides; kõik lipud on toodud [CLI-viites](reference/cli-reference.md).

***

## Veahaldus: 5 kõige levinumat probleemi

| Sümptom | Lahendus |
| --- | --- |
| `Login required` või ajastatud ülesanne jääb kinni `Email:`-käsureal | Käivita `chloros-cli login EMAIL 'PASSWORD'` üks kord sellel arvutil — käsud, millel puudub vahemällu salvestatud seanss, töötavad interaktiivselt, mitte ei ebaõnnestu kohe. |
| `backend unreachable` | Käivita töölaua rakendus Chloros või käivita tagapõhja binaarfail otse (`chloros-backend`). Kui suunate `lattice`/`project`/`daq pool-*` kaugele tagapõhja, kontrollige `CHLOROS_BACKEND_URL`. |
| Array-ühendus blokeeritud: `FRAMES WILL DROP` / `Reduce ROI to enable` | Hosti võrgukaardi vastuvõturõngas on taastatud vaikimisi seadistustele — see on peamine põhjus, miks varem töötanud seadmestik keeldub ühendusest, tavaliselt pärast võrgukaardi draiveri uuendamist. Käivita `chloros-cli lattice network --fix` **õigustega** terminalist (või määra `ReceiveBufferLen=256`, `PendingReceives=64`); vaata viite jaotist *Host NIC Setup &amp; Tuning*. |
| `daq` alamkäsk lõpetab töö: „nõuab täielikku daq-paketti…“ | Oodatav tarnitud versioonides — kompileeritud CLI sisaldab ainult `daq pool-*`-perekonda, mis hõlmab ühendamist, voogedastust, salvestamist ja kap-valikut. Kasutage `pool-*`-i (või `chloros_sdk.connect_daq_sensor()`-i versioonist Python). |
| Jetson kuvab enne suurte kaustade töötlemist vahetushoiatuse | Lisa failipõhine vahetus — CLI kuvab täpsed käivitamiseks vajalikud `fallocate`/`swapon`-käskud. |

***

## Abi saamine

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Iga lipuk, iga alamkäsk:** [CLI viide](reference/cli-reference.md)
* **Python ekvivalent:** [Python SDK](api-python-sdk.md) ja [SDK viide](reference/sdk-reference.md)
* **Tugi:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
