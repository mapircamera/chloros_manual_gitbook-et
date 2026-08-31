# Chloros CLI Viide

**Versioon:**

1.2.0**Koostatud:**

29.07.2026 kell 19:19 ·**Muudetud:**

30.08.2026**Sihtgrupp:** Optimeeritud LLM-kasutuseks; inimesele loetav.**Kohaldamisala:** Kõik `chloros-cli` kasutajale suunatud alamkäskud koos valikuvõimaluste ja kopeeritavate näidetega.

Käesolev dokument on täielik viide MAPIRi Chloros koos tarnitava `chloros-cli`-i käsurea tööriista kohta. See on tahtlikult põhjalik, et LLM (või inimene) saaks allpool toodud loetelude põhjal koostada mis tahes toetatud töövoogu ilma lähtekoodi uurimata.

Kui vajate ainult olulisimat teavet, minge edasi:
- [Viieminutiline kiirjuhend](#five-minute-quickstart)
- [LATTICE-kaamera esmakordse ühendamise töövoog](#lattice-camera-first-connect-workflow)
- [DAQ-anduri esmakordse ühendamise töövoog](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Salvestusrežiimid, salvestusseadmed ja offline-taas töötlemine](#capture-modes-recorders--offline-reprocess)

---

## Konventsioonid

- Kõikidele käskudele on eesliiteks `chloros-cli`. Windows-l on binaarfaili nimi `chloros-cli.exe`; Linux /Jetsonil on see `chloros-cli`.
- Valikulised argumendid on esitatud kujul `--flag`. Nõutavad positsioonilised argumendid on esitatud ilma sulgudeta.
- Kui on antud vaikimisi väärtus, kasutatakse selle väärtust, kui lipukest ei märgita.
- CLI on kerge HTTP-klient, mis töötab Chloros-tagapõhja kaudu (Flask-server aadressil `127.0.0.1:5000`). Tagapõhi käivitatakse enamiku käskude poolt automaatselt. `CHLOROS_BACKEND_URL=<url>` suunab **`lattice`**,**`project`**ja**`daq pool-*`** kaugbackendile – põhikäsud (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fikseerivad tahtlikult `http://127.0.0.1:<port>` ja ignoreerivad seda (IPv4-literaal vältib Windows&#x27; `localhost`→`::1` ~2 s-per-request penalty). Vaata [Keskkonnamuutujad](#environment-variables).
- Kõikide SDK / CLI kutsete jaoks on vaja Chloros+ kontoga sisselogimist (käivita `chloros-cli login` üks kord iga masina kohta; salvestatud `~/.chloros/`-is).
- Näidetes kasutatakse Linux teid; Windows asendage `/home/user/...` `C:/Users/.../...`-iga.

---

## Ülevaade

```
chloros-cli [global options] COMMAND [command options]
```

### Üldised valikud

| Lipuk | Kirjeldus |
| --- | --- |
| `--backend-exe PATH` | Asenda automaatselt tuvastatud tagapõhja käivitatav fail. |
| `--port N` | Tagapõhja HTTP port (vaikimisi: `5000`). |
| `-v, --verbose` | Luba üksikasjalik väljund. |
| `--restart` | Sundida backendit uuesti käivituma (lõpetab kõik töötavad `backend_server.py`-protsessid). |
| `--version` | Kuvab versiooni (`Chloros CLI 1.2.0`). |
| `--help` | Kuvab ülemise taseme abi. |

### Käskude indeks

| Käsk | Otstarve |
| --- | --- |
| [`process`](#chloros-cli-process) | Töötleb kaustSurvey3- või LATTICE-salvestusi algusest lõpuni. |
| [`login`](#chloros-cli-login) | Autentida see arvuti Chloros+ kontoga. |
| [`logout`](#chloros-cli-logout) | Tühjendada vahemällu salvestatud autentimisandmed. |
| [`status`](#chloros-cli-status) | Näita praegust litsentsi / autentimise staatust. |
| [`export-status`](#chloros-cli-export-status) | Näita Thread-4 ekspordi käiku reaalajas `process` käivituse ajal. |
| [`language`](#chloros-cli-language) | Määra või näita „CLI“ kuvamiskeelt (toetatakse 38 keelt). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Vaikimisi projektikaust (ühine graafilise kasutajaliidesega). |
| [`update`](#chloros-cli-update) | „CLI“ värskenduste otsimine ja paigaldamine (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Süsteemi diagnostika + kiirtestid. |
| [`time-sync`](#chloros-cli-time-sync) | PTP grandmasteri staatus / juhtimine. |
| [`lattice`](#chloros-cli-lattice) | LATTICE kaamera juhtimine ja pildistamine (üle 45 alamkäsu). |
| [`daq`](#chloros-cli-daq) | DAQ spektraalsensori juhtimine (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Salvestatud „Chloros“ projekti (kaamerad + DAQ-seadmed) avamine ja käivitamine. |

---

## Paigaldamine

`chloros-cli` on kaasas Chloros töölaua paigaldusprogrammis kõigil toetatud platvormidel — eraldi CLI allalaadimist ei ole. Platvormipaketi paigaldamine lisab `chloros-cli` teie `PATH`-i koos töölaua rakendusega ja selle poolt juhitava taustaprogrammi kõrvale.

Viimased allalaadimised: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Paigaldaja sisaldab ka mugavuseks käivituskripte (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), mis avavad kohe kasutamiseks valmis CLI-keskkonna; neid käsitletakse [CLI kasutusjuhendis](../CLI.md) ja neid siin ei korrata.

### Windows (.exe)

1. Laadige alla „Windows“ paigaldusprogramm allalaadimislehelt.
2. Käivitage `Chloros-Setup-x.y.z.exe` ja järgige paigaldusviisardi juhiseid. Vaikimisi paigalduskataloog on `C:\Program Files\Chloros\` (CLI paigaldatakse kataloogi `C:\Program Files\Chloros\cli\`, mille paigaldusprogramm lisab PATH-i).
3. Avage uus terminal (`cmd.exe`, PowerShell või Windows Terminal), et uuendatud `PATH` saaks tuvastatud.

```powershell
chloros-cli --version
```

Paigaldaja lisab automaatselt `chloros-cli.exe` teie süsteemi `PATH` ja lisab LATTICE-kaamerate jaoks vajaliku Arena SDK käitamiskeskkonda.

### Linux amd64 (.deb)

Ubuntu 22.04 LTS või uuemate versioonide / Debianil põhinevate x86_64 tööjaamade jaoks.

> **Ubuntu 20.04 ei ole toetatud.** Paketi sõltuvuste loend tuleneb sellest,
> millega tagapõhi tegelikult seotud on, ja see hõlmab `libc6 (>= 2.34)`;
> Focal tarnib glibc 2.31. `apt` keeldub installimisest, selle asemel et lasta sellel
> käivitamisel ebaõnnestuda.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

.deb-pakett installib:
- `chloros-cli` kuni `/usr/bin/chloros-cli`
- Kompileeritud backend&#x27;i kuni `/usr/lib/chloros/chloros-backend`
- Arena SDK käitamisajami (LATTICE-kaamerate jaoks)
- Müra vähendamise mudelid, kalibreerimispaketid ja uuenduskanali konfiguratsioon

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Sama struktuur nagu amd64 .deb-failil, kuid CUDA-koostisega, mis on häälestatud Jetson Orin / Orin NX / Orin Nano jaoks.

### Autentimine üks kord iga seadme kohta

Igal platvormil on vaja teha ühekordne sisselogimine aadressil Chloros+, enne kui funktsioonid SDK / CLI hakkavad töötama:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Autentimisandmed salvestatakse vahemällu `~/.chloros/user_session.json`.

### Paigalduse kontrollimine

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Vajalik on Chloros+ tellimus.**CLI nõuab aktiivset Chloros+ paketti.**Copper**on algtaseme Chloros+ pakett — igal tasulisel Chloros+ paketil on juurdepääs CLI / SDK; ainult tasuta**Iron**-paketil seda pole. (Paketi-ID vastavus: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Uuendamine aadressil [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Seda alampiiri rakendab backend, mitte ainult CLI: SDK / CLI lipuga päring, millel puudub tasuline pakett, lükatakse tagasi veakoodiga `403 PLAN_UPGRADE_REQUIRED`, olenemata sellest, kas see pärineb `chloros-cli`, Python SDK, või käsitsi loodud HTTP kliendist. Välja logitud kasutaja saab selle asemel veakoodi `401 AUTH_REQUIRED`. Juurdepääs toimib offline-režiimis paketiarmuaega (kuupaketi puhul 30 päeva, aastapaketi puhul kuni kehtivusaja lõpuni) ja lõpeb selle möödumisel; `chloros-cli status` jätkab töötamist, et põhjus oleks nähtav (see on ainus marsruut SDK / CLI, mis on vabastatud taseme piirangust — `GET /api/license-status`).

---

## Viieminutiline kiirstart

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Töötle pildikausta läbi kogu Chloros töötlusprotsessi (sihtmärgi tuvastamine → kalibreerimine → vinjetatsioon → peegeldus → indeksi eksport).

### Ülevaade

```
chloros-cli process INPUT [OPTIONS]
```

### Asukohaparameetrid

| Parameeter | Kirjeldus |
| --- | --- |
| `INPUT` | Sisendkausta tee, mis sisaldab faile `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE), või `.dng` faile. |

### Üldised valikud

| Lipik | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `-o, --output PATH` | uus ajamärgistusega kaust teie vaikimisi projektikaustas (`~/Chloros Projects`, kui pole teisiti konfigureeritud) | Loositav või taaskasutatav projektikausta. Kui kausta sisaldab juba `project.json`-faili, luuakse selle asemel `_1`/`_2`-kaust selle asemel, et seda üle kirjutada. |
| `-n, --project-name NAME` | automaatne (ajamärge) | Projekti nimi. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` kasutab Chloros+ neurovõrgu abil toimivat debayeri; aeglasem, kuid kvaliteetsem. |
| `--vignette / --no-vignette` | `--vignette` | Vignettkorrektsioon. |
| `--reflectance / --no-reflectance` | `--reflectance` | Peegelduskalibreerimine (kasutab paneeli sihtmärki, kui see leitakse, LATTICE puhul NIST-i seerianumbri järgset kalibreerimist). LATTICE multispektraalsete andmete puhul toimib see ka peegelduse **korrutisena** lüliti — vt [Toote-põhised ekspordilülitid](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | välja | Rakenda PPK GNSS-korrektsioone sidecar-failidest. |
| `--exposure-pin-1 MODEL` | off | Kinnita „Survey3” kahekaameraseadme „pin-1” mudel. |
| `--exposure-pin-2 MODEL` | off | Kinnita „pin-2“ mudel. |
| `--recal-interval SECONDS` | 0 | Sundi kalibreerimisarvutusi uuesti käivitama iga N sekundi järel salvestusaja jooksul. |
| `--timezone-offset HOURS` | local | Ületa väljundmetadata sisse kirjutatud ajavööndi nihke. |
| `--format FORMAT` | `TIFF (16-bit)` | Üks järgmistest: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | puudub | Taimestiku indeksid (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Sundida LATTICE TIFF-failide töötlemisjärjekorra alguspunkti (failid Survey3 .raw jäävad muutumatuks). Samuti on olemas tagavara, mis võimaldab töödelda ka **raw-failita** salvestusi — vt [Kuidas salvestuste kaust välja näeb](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | sisse lülitatud | Väljastab lineaarselt debayereeritud tulemuse (`Debayered_Images`). Vaata [Toote-põhised ekspordilülitid](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | sisse | Väljastab ekraanilise eelvaate (`Preview_Images`): RGB = valgebalanss (DAQ-valgusallikas, kui see on saadaval, muidu hall maailm) + gamma; multispec = false-värvi venitus. |
| `--radiance / --no-radiance` | sisse | Edastab float32-tüüpi kiirgustiheduse (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | LATTICE peegeldusvõime toote viide: `auto` = kvaliteedikontrolli läbinud raamisisene sihtmärk on absoluutne viide, DAQ-allapoole suunatud kiirgus (ρ = π·L/E) on varuvariant; `target` = range (ilma DAQ asendamiseta); `daq` = DAQ-autoriteetne. Vaata [Tootekohased ekspordilülitid](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | puudub | Ühiku kaupa **mõõdetud** sihtmärgi peegeldusvõime skaneeringute kataloog (`<serial>.csv`); ebaõnnestumise korral kasutatakse tagavarana nominaalseid T3/T4P spektreid. |
| `--array-alignment / --no-array-alignment` | sisse | LATTICE-massiivid: rakenda iga pildi `Chloros:Alignment*` XMP-failis märgitud moodulitevahelist joondamist igale töödeldud tootele (debayering / eelvaade / kiirgus / peegeldus / indeks). Sildita piltide puhul ei tehta midagi. |
| `--array-alignment-crop / --no-array-alignment-crop` | kärpimine | Kärbi joondatud ekspordid massiivi ühisele kattuvale alale, nii et kõik moodulid jagavad ühte pindala; `--no-…` säilitab täieliku anduri pildi (must täitmine väljaspool allika väljaspool). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Ümberdiskrimineerimine joondamise moonutuse jaoks. `nearest` säilitab täpsed allika digitaalsed numbrid (radiomeetriliste väärtuste segunemist pikslite vahel ei toimu). |

### Sihtmärgi tuvastamise valikud

| Lipuk | Kirjeldus |
| --- | --- |
| `--min-target-size PIXELS` | Detektori minimaalne paneeli-sihtmärgi suurus (px). |
| `--target-clustering 0-100` | Klastrite tundlikkus. |
| `--target / --targets` | Käsitle sisendkausta ainult sihtmärkide paneelina (jätta vaatluse tuvastamine vahele). |

### Näited

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

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

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Tootepõhised ekspordilülitid (LATTICE multispektraalne)

LATTICE töötlemine hargneb **ühe läbimisega kõikidesse asjakohastesse toodetesse**. Neli tüübipõhist lülitit — `--debayered`, `--preview`, `--radiance`, `--reflectance` — on kõik**vaikimisi sisse lülitatud**; kasuta `--no-<type>`-vormi, et üks neist välja lülitada. RGB peamised kaamerad edastavad alati ainult debayered + eelvaate (mitte-ribalist kiirgust/peegeldust), seega on `--radiance`/`--reflectance` nende puhul tühikäigud. Lülitajaid ignoreeritakse Survey3 `.raw` puhul (mis järgib standardset peegelduse/sihtteed järgivad). *(Vana lipuk `--radiometric-output {reflectance,radiance,sensor-response}` **eemaldati** ja asendati nende lülititega; `sensor-response` taset enam ei ole.)*

| Toode | Väljund | Kas on vaja DAQ allapoole suunatud andmeid? |
| --- | --- | --- |
| `--debayered` | Lineaarne demosaik (`Debayered_Images`). | Ei. |
| `--preview` | Eelvaate kuvamine (`Preview_Images`): RGB = WB + gamma; multispec = false-värvide venitamine. | Nr. |
| `--radiance` | float32 W/m²/sr/nm täielikust radiomeetrilisest ahelast (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 peegeldusvõime ρ (`32768` = 1,0), Pix4D-valmis. | **Jah**, välja arvatud juhul, kui seda kinnitab kvaliteedikontrolli läbinud raamisisene sihtmärk (vt allpool). |

`--reflectance-source` valib peegeldusvõime võrdluspunkti:**`auto`**(vaikimisi) muudab kvaliteedikontrolli läbinud kaadris oleva sihtmärgi**absoluutseks võrdluspunktiks**— sihtmärgiga kinnitatud empiirilised joonekettid võrreldakse väljajäetud paneelidel ja rakendatakse mõõdetud võitjat — kui sihtmärki pole või kvaliteedikontroll ebaõnnestub, kasutatakse tagavarana andmekogumise allapoole suunatud jaotust (ρ = π·L/E);**`target`**on range (andmekogumise asendamist ei toimu);**`daq`**loobub DAQ-i autoriteetsele käitumisele. Sihtmärgi geomeetria (ArUco / fikseeritud ROI / riba) pärineb projekti sihtmärgi konfiguratsioonist; `--target-reflectance-dir DIR` säilitab ühiku kohta**mõõdetud** skaneeringuid (`<serial>.csv`), mida otsitakse sihtmärgi seerianumbri/QR järgi, kasutades varuvariandina nominaalseid T3/T4P spektreid.

DAQ-peegeldusrada määrab **ajamärgiga sobiva allapoole suunatud kiirguse**automaatselt salvestatud**`.daq`**(DAQ-U/M/E)**või piltide juures leitud DAQ-M-i omast `.csv`**. Kui kaamera- või DAQ-kalibreerimispaketti pole kohalikult vahemällu salvestatud,**laadib selle esmakordsel kasutamisel automaatselt AWS-ist** (nõuab ühekordset internetiühendust; salvestatakse `~/.chloros/`-i alla).

#### Peegeldusvõime pikslite lugemine (Pix4D / Metashape / teie enda skriptid)

Peegeldusvõime salvestatakse täisarvulise DN-väärtusena ning **DN-väärtus, mis tähendab ρ = 1,0, sõltub allikakaamerast**:

| Allikas | ρ = 1,0 on | Kuidas kindlaks teha |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (varu kuni ρ 2,0) | Failile on märgitud XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (lõigatud väärtusel ρ 1,0) | `Chloros:*` XMP-märgiseid pole — see puudumine *onki* signaal. |

**Loe `Chloros:PixelScale` ja jaga sellega**, selle asemel et eeldada konstanti. Silt on defineeritud uint16-domeenis, seega jääb see `32768` samaks ka väljundvormingutes, kus toimub ümberskaalamine — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` ja `TIFF (32-bit, Percent)` on kõik iseenesest kirjeldavad (normaliseeri salvestatud andmetüüp esmalt tagasi uint16-ks: ×257 8-bitisest, ×65535 float-tüübist).

> **Üks juhtum ei sisalda skaalat, see on nii kavandatud.** Kui 8-bitine allikas (BayerRG8) kirjutatakse 8-bitiseks TIFF-iks, siis töötlemisahel *kärbib* väärtused vahemikku 0..255 ümberskaalamise asemel, mistõttu iga väärtus, mis ületab ρ≈0,008, tasandatakse väärtuseks 255 ja faili ei kirjeldata ühegi skaalaga. Chloros jätab seal tahtlikult välja nii `Chloros:PixelScale` kui ka `MicaSense:RadiometricCalibration` tuplit ning logib selle põhjuse. **Kui LATTICE-peegeldusfailis puudub see silt, ärge eeldage skaalat — ekspordi uuesti 16-bitise või 32-bitisena**, selle asemel et jagada piksleid, mida ei olnud kunagi võimalik jagada.

#### EXIF-andmed kantakse ekspordisse üle

`process` kopeerib allika pildi **GPS-ploki ja selle ExifIFD** igale tootele, seega sisaldab
eksporditakse koos georeferentsiga ka `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` ja
`CameraSerialNumber` koos georeferentsiga.

**`FocalLength` ei ole fotogramm-meetria jaoks valikuline.** Pix4D arvutab maapinna proovivahe kauguse
fokaalkauguse ja kõrguse põhjal; märgise puudumisel kasutatakse äärmiselt vale mõõtkava. Ühel
49 kaadri oranžipuude istanduse lennu puhul muutis puuduv märge 411 m × 160 m suuruse ala rekonstrueeritud
47,8 km × 13 km suuruse alaks – 455 MP ortofoto, mis koosnes peamiselt „nodata“ andmetest, mida peeti esialgu mosaiigiprobleemiks ja
BigTIFF-probleemiks, enne kui keegi GSD-d kontrollis. Kui teie ortofoto tulemuseks on ebausutav
mõõtkava, käivitage eksporditud toote suhtes esmalt `exiftool -FocalLength`.

Koopia ei ole teadlikult **`-all:all`**: IFD0 struktuurimärgised rikuvad LATTICE väljundit, kui
kopeeritakse, ning `ExifImageWidth` / `ExifImageHeight` on välistatud, kuna need kirjeldavad
*allika* pildistamist — muidu kannaks suuruselt muudetud eksport mõõtmeid,
mis oleksid vastuolus selle enda rasteriga. XMP kirjutatakse otse, mitte kopeeritakse, sest ExifTool
kõrvaldab sama käivituse XMP-märgised, kui XMP-plokk kopeeritakse (mis jätaks välja MAPIR
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
`Model`) ja `<model>_<filter>` (Survey3 – kaks kaamerat, millel on ühine sensor ja filter, kuid erinev
objektiiv, hoiavad eraldi puid, kuna vinjetatsioon, vaateväli ja moonutus erinevad. Formaat
kausta järgib `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` või `tiff32`, kui
`TIFF (32-bit, Percent)`.

> **Iga eksporditud toode säilitab SOURCE-faili nime.** Radiance’i eksport
> `capture_…_raw.tif` kannab endiselt nime `capture_…_raw.tif` — see asub lihtsalt kataloogis
> `tiff32/Radiance_Images/`. **Toote identifitseerib kaust, mitte failinimi**, seega `*radiance*.tif`
> otsinguga ei leita midagi; otsige selle asemel kataloogi järgi.

### Valgusanduri salvestused — kalibreeritud `.daq` + `.csv`

`process` töötleb ka sisendkaustas olevaid `.daq` salvestusi ning selleks **ei**
vaja see mingeid pilte: eraldi lennutatud DAQ-U / DAQ-M / DAQ-E on täielik
salvestusseade ning kaust, mis sisaldab ainult `.daq`-faile, on kehtiv sisend.

DAQ-seadet saab salvestada **ilma** selle kalibreerimiseta – just seda näitab avalik
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) salvestid
(`record_daq.py`) vaikimisi teevad: nad kirjutavad salvestusse andurite toorandmed ja märgistavad faili nii, et
Chloros laadib selle anduri tehase kalibreerimise **seerianumbri järgi** (esmalt kohalikust vahemälust,
seejärel MAPIR pilvest) ja rakendab seda. `process` kirjutab tulemuse tagasi:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` sisaldab ühte rida iga mõõtmistulemuse kohta: UTC-ajamärge, integratsiooniaeg, koguvõimsus,
fotopiline/skotopiline luks, PPFD (ja selle sinine/roheline/punane jaotus), tipplainepikkus, seejärel kogu
spektri andmed anduri enda lainepikkuste võrgustikus. `.daq` imporditakse uuesti ilma
teistkordse kalibreerimiseta.

Eduka käivituse korral näitab käivitus aruannet `Light-sensor products written: N (calibrated .daq + .csv)`.
Sulgudes olev tekst kirjeldab seda, mis tegelikult kirjutati, seega see kõlab
`(RAW COUNTS — this sensor has no calibration bundle)` pakendita anduri puhul ja
`(N calibrated, M raw counts)` kahe asja sisaldava kausta puhul. Tagapõhja enda
pealkirjad `[DAQ-EXPORT]` ja `[RUN-SUMMARY]` on sõnastatud samamoodi – ükski
neist kolmest ei saa nimetada kalibreerimata eksporti kalibreerituks.

DAQ-U / DAQ-M / DAQ-E salvestus, mille kalibreerimispaketti ei ole võimalik alla laadida – olete
offline-režiimis või sellel anduril puudub kalibreerimisandmed failis –, **jätatakse põhjendusega vahele**
real `[DAQ-EXPORT]` ning seda ei kirjutata kunagi välja kui „kalibreeritud” failina, mis sisaldab töötlemata loendusi.
Ühenduge internetiga ja käivitage uuesti. Põhjus on see, mille lugeja tegelikult
selle faili jaoks kindlaks tegi (loetamatu skeem, paketi puudumine, kirjutusviga), ning käivituse
kokkuvõttes loetletakse **erinevad** põhjused – kakskümmend ühe põhjuse tõttu vahele jäetud faili kuvatakse ühe
põhjusena, mitte selle kahekümne kordusena.

#### DAQ-A salvestuste eksportimine toorandmetena

**DAQ-A** seeria on vanem kui seerianumbri-põhine pakettide süsteem ja sellel puudub kalibreerimispakett,
mida alla laadida — selle asemel kalibreeritakse seda kohapeal peegeldusmäära sihtmärgi abil, mis on
põhjus, miks seda kunagi vaja ei olnud. Nende salvestuste tagasilükkamine jättis neile võimaluse
oma numbreid üldse välja saada, seega eksporditakse need **teise nime** all:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Teine failinimi, mitte lipukese faili sees, sest väide peab säilima
ka siis, kui seda edastatakse e-kirjaga pelgalt nimena. Pealkiri `.csv` näitab
`raw spectral sensor counts (NOT irradiance)` ja hoiatab, et väärtused on võrreldavad
**faili sees** — mis ongi just see, milleks sihtpõhine kalibreerimine neid kasutab — ja
mitte andurite vahel. Võimsusest sõltuvad fotomeetrilised veerud (koguvõimsus, fotopiline ja
skotopiline luks, PPFD) kirjutatakse **NULL**, mitte integreeritakse loendustest, ning käivituse
kokkuvõttes on kirjas `RAW COUNTS`, seega ei saa logisse „eksporditud“ andmeid lugeda kiirgustihedusena.

Vanemad **v1.01 / v1.02** salvestused (neid kirjutab DAQ-A-SD) ei sisalda lugemise-põhist ajamärget,
vaid ainult faili kirjutamise aja. Pildi↔allapoole suunatud valguse sobitaja ikka veel lükkab need tagasi –
kaadri sobitamine kirjutamisajaga oleks märkamatult vale — kuid eksportija loeb neid ning
CSVi väljundis kuvatakse `clock=daq_created_on`, seega näitab toode, millisel kellal see põhineb.

### Märkused

- `process` tuvastab automaatselt, kas teie kaust on tüüpi „Survey3”, „LATTICE” või segatüüpi.
- Töö edenemine edastatakse Server-Sent Eventsi kaudu; CLI näitab reaalajas iga niidi edenemist (tuvastamine, analüüsimine, töötlemine, eksportimine).
- Juhul, kui kasutate „Linux” /Jetsoni puhul kontrollib „CLI“ vahemälumahtu ja võib enne suurte kaustade töötlemist hoiatada. Tekstuuritundlik debayer rakendab madala energiatarbimisega Jetsonitel (Nano, Orin Nano) automaatselt- rakendab madala energiatarbimisega Jetsonitel (Nano, Orin Nano) GPU sageduse piirangut.
- Eduka käivituse korral näitab käivitusaruanne, kui palju pilditooteid kirjutati (`Image products written: N`).

#### Käivitus, mis ei kirjuta ühtegi pilti, ebaõnnestub

Kui palusite tooteid ja käivitus kirjutas **ühtegi** — vaid `project.json` ja
`calibration_data.json` — käsitleb `process` seda ebaõnnestumisena: ta väljastab
`Processing finished but wrote no image products.` ja **lõpetab töötamise nullist erineva tulemusega**, nii et skript saab
seda tuvastada. Sõnumis on nimetatud projekti kaust ja tavapärased põhjused:

- sisendkaustu ei tunnistatud salvestusena (kontrolli paigutust ja `--input-level`) või
- kõik taotletud tooted jäeti vahele, kuna need ei olnud nende kaamerate puhul rakendatavad (nt kui küsiti
  kiirgust/peegeldusvõimet kaameratelt, mis toetavad ainult RGB).

Käivita uuesti koos `--verbose`-iga ja kontrolli tagapõhja logist ridu `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
mis selgitavad kaamerapõhiseid vahelejätmisi, mis muidu ei jõua CLI väljundisse.

Tahtlikult ainult metandmetega käivitus — kõik tooted välja lülitatud ja ilma `--indices`-ita — on ikkagi
**edu**, sest tühi pildiväljund on selles juhul õige tulemus.

Sama kehtib ka **ainult valgussensoriga käivituse**: `.daq` salvestuste kaustas pole määratluse järgi eksporditavaid pilte
ning käivitust hinnatakse selle asemel kalibreeritud `.daq` / `.csv` järgi, mille see kirjutas.

---

## `chloros-cli login`

Autentige see seade pilvekontoga Chloros+. Autentimisandmed salvestatakse turvaliselt `~/.chloros/user_session.json`-i.

```
chloros-cli login EMAIL PASSWORD
```

### Näited

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (paroolist osade eemaldamine või dubleerimine). Veakoodi 401 korral proovib CLI automaatselt uuesti, lisades esmalt `$$` ja seejärel parooli dubleerimata poole; kui uus katse õnnestub, logib see teid sisse ja kuvab õige ühekordsetsitaadi süntaksi, mida järgmisel korral kasutada.

> **Peata/skriptipõhine kasutamine: vahemällu salvestamata seanss tähendab interaktiivset käskuderida, mitte kiiret ebaõnnestumist.** Iga tagapõhja käivitav käsk (`process`, `status`, `export-status`, `time-sync`, …) käivitatakse ilma vahemällu salvestatud litsentsi/seansita, ilmub enne jätkamist stdin-i interaktiivne `Email:` / `Password:`-käsurea. Seetõttu jääb vahemällu salvestamata seansiga järelevalveta töö ootama sisendit – käivita `chloros-cli login EMAIL PASSWORD` üks kord iga masina kohta enne peata tööde planeerimist.

---

## `chloros-cli logout`

Tühistab vahemällu salvestatud seansi ja sunnib järgmisel kutsumisel uuesti sisse logima.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Näitab praegust litsentsitaset (Iron/Copper/Bronze/Silver/Gold), autentimitud kasutajat ja seadmetega seotud kontode arvu.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Küsib Thread-4 ekspordi hetke edenemist. Seda on ohutu kutsuda **`process` käivitamise ajal** teisest shellist.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

CLI&#x27;i kuvakeele seadistamine (toetatakse 38 keelt, sealhulgas CJK, RTL ja india keeled). Vanematel konsoolidel, mis ei suuda skripti kuvada, lülitub süsteem sujuvalt inglise keelele.

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

Need haldavad projekti vaikimisi kausta asukohta (jagatud graafilise kasutajaliidesega).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Ainult Jetsonile. Kontrollib `version_url`-i alates `/etc/chloros/update.conf`-ist ja pakub sobiva `.deb`-i allalaadimist ja installimist.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Linux-l / Jetsonil teostab CLI ka **iga käivitamisel automaatse uuenduste kontrolli** (blokeerimata, ei viivita kunagi käsu täitmist): see loeb `/etc/chloros/update.conf`, salvestab tulemuse 1 tunniks vahemällu `~/.chloros/update_cache.json` ja väljastab `Update available: vX.Y.Z / Run: chloros-cli update`, kui uuem versioon on olemas. Vea korral vaikitakse see üle ja Windows korral.

---

## `chloros-cli selftest`

Käivitab 7-etapilise kiirtesti: versioon, pordi kättesaadavus, tagapõhja käivitamine, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), müra eemaldaja mudeli olemasolu, CUDA+müra eemaldaja valmisolek.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP grandmasteri staatus ja juhtimine. PTP grandmasteri käitab „Chloros“ host; LATTICE-kaamerad ja DAQ-E-seadmed töötavad selle alluvuses seadmetevaheliste ajamärgiste jaoks.

| Alakäsk | Kirjeldus |
| --- | --- |
| `status` | Näita grandmasteri seisundit, BMCA prioriteete, kella identiteeti. |
| `peers` | Loetle Delay_Req kaudu tuvastatud orjad (kaamerad + DAQ-E andurid). |
| `cameras` | PTP-töökorrasolek kaamera kaupa (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Grandmaster-protsessi taaskäivitamine. |
| `set-priority --priority1 N --priority2 N` | BMCA prioriteetide ümberkirjutamine. |

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

LATTICE kaamera juhtimine. Iga alamkäsk suunatakse läbi tagapõhja „Chloros“; tagapõhi haldab kaamerate kogumit, mistõttu järgnevad „CLI“-kutsed kasutavad uuesti sama avatud käepidet.

### Üldised valikud (enamikul alamkäskudel ühised)

| Lipuk | Kirjeldus |
| --- | --- |
| `-d, --device N` | Kaamera indeks (vaikimisi: 0). |
| `-s, --serial SN` | Konkreetne seerianumber; asendab `--device`. |
| `--serials SN1,SN2,…` | Komadega eraldatud seerianumbrid mitme kaamera kasutamiseks. |
| `--all` | Töötab kõigi avastatud kaameratega. |
| `--exposure US` | Säriaeg mikrosekundites. |
| `--gain DB` | Võimendus dB-des. |
| `--pixel-format FMT` | nt `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Pildi mõõtmed. |
| `--preset {default,high_quality,high_speed,triggered}` | Rakenda eelseadistust. Kõik on vabajooks, välja arvatud `triggered`, mis seadistab kaamera riistvaralisele servale 2. liinil — kui seda liini midagi ei aktiveeri, ootab kaamera lõpmatult, selle asemel et pilti salvestada. |
| `-o, --output DIR` | Väljundkataloog (vaikimisi: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-paketi suurus. `auto` käivitab ICMP+GVSP-proovid; `jumbo` = 9000; `standard` = 1500. |

### LATTICE-kaamera esmakordse ühendamise töövoog

```bash
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
#      - Auto-shrink frame size to fit the wire
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
| `lattice network [--fix] [--estimate] [--cameras N]` | Võrgukaardi seadete kontrollimine/parandamine; ribalaiuse/kaadrisageduse (FPS) hindamine. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Stabiilse skeemiga tagapõhja võrguvõimekus + massiivi soovitus (tagastab `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` säilitab taotletud resolutsiooni, kuid piirab siht-fps-i — loe `recommended.recommended_target_fps` ja edasta see ühenduse sihtmärgiks; käsitle seda eduna, mitte veana. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | „Mis-kui“ analüüs ilma kaameraid avamata. **`--n-active` on võrgus olevate kaamerate koguarv, mitte ainult selle massiivi**– suurendage seda, kui eraldiseisvad kaamerad edastavad voogu samaaegselt või kui võrgu eelarve arvutatakse nõudluse alusel, mis neid alahindab (vaikimisi: `len(--models)`). Kuvab alati koondväärtuse `Wire budget:` (nõutav MB/s vs kokkupõrkeohutu ülempiir) ja `Max cameras:` rida ning märgib `** OVER-SUBSCRIBED**`, kui massiiv koormab võrku üle – vt [Massiivi kaadrisagedus ja pursemudel](#array-fps--burst-model). |
| `lattice gpu` | Näita GPU staatust. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Kontrolli või uuenda kaamera püsivara. Kohalik `.fwa` valik on fikseeritud: `firmware/<MODEL_PREFIX>/` asuv fail, mis vastab ehituse `MIN_FIRMWARE_VERSION`-ile, laaditakse sisse, kui see on olemas (kõrgeim versioon ainult varuvariandina), seega on uuem tarnija pildifail jääb kettal passiivseks, kuni seda kinnitust muudetakse – tahtlikult jõuavad uuemad versioonid seadmetesse allkirjastatud AWS-manifesti kaudu, mida eelistatakse, kui see on uuem. |
| `lattice presets [--apply NAME]` | Kaamera eelseadistuste loetlemine või rakendamine. |
| `lattice status` | Kaamera reaalajas seisund. |

#### Salvestamine

| Alakäsk | Otstarve |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Üks kaader. **Salvestab vaikimisi kõik eksporditüübid** (`--processing all`); vt [Salvestamise eksporditasemed](#capture-export-levels-the-all-default). `--levels` salvestab konkreetse alamhulga (asendab `--processing`); `--force-daq` kirjutab määratud DAQ-näidu `.daq`-sidecarina isegi ainult toorandmetega salvestuse korral. `--jpeg-quality` = JPEG kvaliteet 1–100 (vaikimisi 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Salvestab andmevoogu kettale kuni Ctrl+C-ni. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Brauseri-põhine reaalajas MJPEG-eelvaade. `--ae-damping` seadistab automaatsevalgustuse summutuse (0,4–100). |

#### Anduri häälestamine

| Alakäsk | Otstarve |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | GenICam-sõlme lugemine/kirjutamine. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Valgustusaeg ja automaatne valgustuse reguleerimine. |
| `lattice gain [--auto] [--off] [--set DB]` | Võimendus ja automaatne võimendus. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Anduri huvipiirkond ja pikslite ühendamine. |
| `lattice format [--set FMT] [--list]` | Pikseliformaat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Riistvara-/tarkvarapõhine triggereerimine. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (lipud puuduvad = ühekordne valge tasakaal) | Valge tasakaalu operatsioonid. RGB /Ainult Bayeri kaamerad; mono M3M-i puhul ei toimu midagi (vaheletõmmatud). |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB ekraani värviprotsess. `natural` (vaikimisi) on odav reaalajas viimistlus; `enhanced` lisab värvirebendi eemaldamise + elavuse + CLAHE kohaliku kontrasti täieliku hub-parity välimuse saavutamiseks, mis maksab umbes 2× rohkem viimistluskulu kaadri kohta, mistõttu on **reaalajas** kaadrisagedus madalam — salvestatud kaadrid saavad igal juhul täieliku viimistluse. Ainult RGB /Bayer-kaamerad; mono M3M-il vaheletõmmatud. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Ekraani küllastus/kontrast (RGB-filtriga kaamerad). Mono M3M-il vaheletõmmatud. |
| `lattice filter [--set NAME] [--list]` | Kaamera filtri mudeli seadistamine (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Andurite toite-/soojuspunktid; lülita sisse madala võimsusega ooterežiim. |

#### Kalibreerimine ja andurid

| Alakäsk | Otstarve |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibreerimine peegeldusmäära sihtmärgi abil. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Sisseehitatud allapoole suunatud valguse anduri käsud. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Vignettkorrektsiooni rakendamine olemasolevatele piltidele. |

#### Mitme kaamera kasutamine (ajutised seansid)

| Alakäsk | Otstarve |
| --- | --- |
| `lattice multi-info` | Loetleb kõik kaamerad, millel on sünkroniseerimisrollid. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Üks sünkroniseeritud kaader igast kaamerast. Salvestab **vaikimisi kõik eksporditüübid**, kui on ühendatud püsiv massiiv; ajutise massiivi puudumise korral on pilt**ainult debayereeritud** (ülejäänud jaoks käivita esmalt `array-connect`). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Sünkroniseeritud kaadrite voogedastus (ajutine). |
| `lattice multi-test [--count N]` | GPIO sünkroniseerimise ajastuse test. |
| `lattice multi-detect [--line LINE] [--json]` | GPIO-meistri/orja juhtmestiku automaatne tuvastamine. |

#### Joondamine

| Alakäsk | Otstarve |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — pluss detektor/sobitaja nupud `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-nupud `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, mitme kaadri kombinatsioon `[--averaging mean\|median\|inlier_weighted]`, geomeetrilised piirangud `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, ruumiline piirang `[--roi X0,Y0,X1,Y1] [--mask PATH]` ja iga orja jaoks eraldi ümberkirjutused `[--per-cam-override SN:KEY=VALUE]` (korratav) | Arvuta joondamisprofiil reaalajas kaamerate põhjal. `--prefilter` on vaikimisi `gradient` (servakaart; vastab graafilise kasutajaliidese/massiivi joondajale — servad säilivad kõigis spektriribades). `--matcher flann` annab tulemusi alates ~5000 tunnusest; `--averaging median` on vastupidav ühe halva pildi suhtes, `inlier_weighted` kaalub vastavuste arvu järgi; `--lock-scale` projitseerib lähimale pööramisele (ilma skaalata), `--lock-axis` nullib ühe nihkekomposendi; `--mask` kehtib iga kaamera kohta (kasutage `--per-cam-override` kaamerapõhiste seadistuste jaoks, nt `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` keeldub salvestada kalibreeringut, mille reprojitseerimise RMS ületab piirmäära. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Salvestab ühe joondatud mitme sagedusribaga kaadri. `--bit-depth` sobitab vaikimisi kaameraga; `--no-crop` säilitab täiskaadri (täidab tühjad kohad mustaga); `--interpolation` (vaikimisi `linear`) ja `--border-mode`/`--border-value` (vaikimisi `constant`/0) juhivad CPU-deformeerimist — GPU-tee on igal juhul bilineaarne. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Vooguga joondatud mitme sagedusriba raamid (samad warp-regulaatorid kui `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Profiili üksikasjade kuvamine. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Muuda kihtide järjekorda. |

#### Indeks / Taimestiku matemaatika

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Täielik lipukomplekt: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (korduv), `--capture-level raw|debayered|radiance|reflectance|unknown` (asendab allikas TIFF salvestatud salvestustaseme; vaikimisi: loetakse TIFF metaandmetest), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Koos mudeliga `--live` kehtivad ka joondamise nihutusnupud: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel` sümbolid on suurtähe-tundlikud.** Sümbolite pool peab täpselt vastama eelseadistuse kanalite nimedele (eelseadistustes kasutatakse väiketähti, nt NDVI = `red`,`nir` — vaata `--list-presets`), ning sagedusriba pool peab vastama sagedusriba nimele joondatud kuhjas (või olema 0-põhine sagedusriba indeks offline-režiimis). `--channel red=Red_660 --channel nir=NIR_850` töötab; `--channel RED=660` ebaõnnestub veaga `channel_map missing entries`.

#### Püsivad ühendused (Smart-Prep, GUI-ekvivalentne voog)

Need käsud hoiavad kaamerad tagapoolse reservi avatuna mitme „CLI“ käivituse jooksul.

| Alakäsk | Otstarve |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Lisab ühe kaamera reservi (üks kaamera, mitte massiiv). |
| `lattice cam-disconnect [--serial SN] [--all]` | Vabastab. |
| `lattice cam-list` | Loetleb puulis olevaid kaameraid. |
| **`lattice array-connect`**|**Ühendab püsiva sünkroniseeritud massiivi (SOOVITATAV alguspunkt).** Käivitab täieliku GUI smart-prep-voo. |
| `lattice array-disconnect [--array-id ID] [--all]` | Vabasta massiiv. |
| `lattice array-list` | Näita ühendatud massiivide loendit. |
| `lattice array-status [--array-id ID]` | Reaalajas kaadrisagedus (fps), PTP, viimane viga. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Üks sünkroniseeritud pilt reaalajas massiivist — Üksik / Pidev / Intervall / Kiireim. **Vaikimisi `all`** (üks fail iga kaamera kohta vastava eksporditüübi kohta). Vahelejäänud kaamerad (nt RGB, mis on välja jäetud kiirguse/peegelduse mõõtmisest) kajastatakse koodiga `Skipped: SN:<serial> (<reason>)`; peegelduse mõõtmiseks kasutatud DAQ-näit salvestatakse koos nendega ja kajastatakse koodiga `DAQ: <path>`. Vaata [Salvestusrežiimid, salvestusseadmed ja offline-töötlemine](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Salvestab reaalajas kombineeritud indeksivaate videona/GIF-ina (seirekvaliteediga; nõuab kombineeritud voo avamist). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Kõrge kaadrisagedusega toores Bayer-pildiseeria (analüüsikvaliteet; töötle offline-režiimis). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Töötle salvestatud toores pildiseeria kalibreeritud videoks (videodeks). |

##### `array-connect` Valikud

| Märge | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--serials SN1,SN2,…` | avasta automaatselt kõik LATTICE-kaamerad (vajalik ≥2) | Esimene seerianumber on MASTER. Kui see jäetakse välja, filtreeritakse avastamine LATTICE (`TRI032*`) mudelite peale ja ühendab need kõik. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO sünkroniseerimisliin. |
| `--target-fps F` | auto | Masteri päästiku tulistamissagedus. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Tasevalija ümberkirjutamine. |
| `--wire-ceiling-mbps MB_PER_S` | automaatselt tuvastatud | **Hosti püsiv andmeside eelarve, MB/s – arv, millest sõltub kogu massiivi eraldamine.** Vähendage seda, kui massiiv teatab GVSP-rikkis raamidest: automaatne väärtus tuletatakse võrgukaardi teatatud ühenduskiirusest, mis ülehinnab USB-adaptereid, kitsaid PCIe-kanaleid ja hõivatud jagatud võrgustikke. Säilitatakse projektimassiivi salvestusplokis, seega taastub see pärast uuesti avamist / CLI / SDK ühenduse taastamist. Vaata [Massiivi tervis](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | automaatne | Riistvaraline binning. |
| `--no-recommend` | off | Võrguanalüüsi etapi vahelejätmine. |
| `--no-ptp` | off | PTP-i keelamine (sel juhul ei ole kaameratevahelised ajamärgised **võrreldavad**). |

### Smart-AE / Smart-Capture

LATTICE-kaameramassiivid käivitavad taustal pideva automaatse ekspositsiooni (AE) niipea, kui need on ühendatud, kuid äsja suunatud stseeni puhul kulub hetk, kuni pilt stabiliseerub. `array-capture --smart` on **pakettlahendus**: see ootab, kuni AE on kõikides massiivi kaamerates stabiliseerunud, ja käivitab seejärel pildistamise. Kasutage seda, kui vahetate stseene seansi keskel.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Vaikimisi on stabiliseerumispoliitika konservatiivne: 5-sekundiline ooteaeg, 1,5-sekundiline stabiilsusaken, ±5 % ekspositsiooni hajuvuse tolerants. Kui vajate automaatikalt teistsugust käitumist, saate seda häälestada kaudu „SDK” (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`).

### Pildistamise eksporditasemed (vaikimisi `all`)

Alates käesolevast versioonist on `lattice capture`, `lattice multi-capture` ja `lattice array-capture` **vaikimisi `--processing all`** — üks salvestatud fail eksporditüübi kohta, mis kehtib iga kaamera puhul, vastavalt kasutajaliidese „Capture All” käitumisele. Tasemed on järgmised:

| Tase | Väljund | Kehtib |
| --- | --- | --- |
| `raw` | Ühekanaliline Bayer (mustvalged kaamerad: üks sagedusala) otse sensorist. | Kõik kaamerad. |
| `debayered` | 3-kanaliline BGR-demosaik (mustvalged kaamerad: 1-kanaliline halltoon). | Kõik kaamerad. |
| `radiance` | float32 W/m²/sr/nm täieliku radiomeetrilise ahela kaudu. | Ainult multispektraalsed (M3C/M3M) — **RGB-filtriga kaamerate puhul vahelet jäetud**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-valmis. | Ainult multispektraalne ja **ainult juhul, kui DAQ on seotud + kaamera on kalibreeritud**; muidu jäetakse vahele. |
| `preview` / `display` | Täielik GUI-eelvaate ahel (CCM + WB + gamma vastavalt kaamera profiilile). `lattice capture` nimetab seda `preview`; `array-capture`/`multi-capture` kasutavad `display`. | Kõik kaamerad. |

Edasta üksainus tase, et salvestada just see üks (`--processing debayered`). Kui küsid `all`-i, jäetakse tasemed, mis antud kaamerale ei kehti, vahele (ja neist teatatakse), ilma et tekiks viga — ühendamata või kalibreerimata kaamera saab ikkagi `raw` / `debayered` / `preview`.

Iga peegeldusraami puhul kirjutatakse tegelikult kasutatud DAQ allapoole suunatud mõõtmistulemus pildi kõrvale **`.daq`**-lisasse (et salvestust saaks hiljem uuesti töödelda) ja see kajastatakse `DAQ:`-real.

### Kuidas näeb välja salvestuste kaust

Iga eksporditüüp paigutatakse oma **alakausta** alla kataloogis `-o`, seega mitmetasandilises salvestuses ei segune tüübid kunagi:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` on salvestamise ajamärge ja `<serial>` kaamera seerianumber, seega jagab üks sünkroniseeritud rühm
ajamärget kõigi kaamerate vahel. **Pange tähele üht asümmeetriat:** `display`-tase salvestatakse kausta
nimega `preview/`, samas kui failide nimedes säilitatakse `_display` — kausta ja laiendi nimed erinevad
ainult selle taseme puhul. Tundmatud tasemed salvestatakse oma nime kandvasse kausta ning kui alamkausta
ei ole võimalik luua, kirjutatakse fail väljundi juurkaustasse, selle asemel et see kaotsi läheks.

**Salvestatud kaustade uuesti töötlemine:**suunake `chloros-cli process`**captures juurkaustale**
(`output/`). `process` impordib tavaliselt ainult nimetatud kausta, kuid kui selles kaustas pole
pilte ja seal on alamkaustad, liigub programm automaatselt sügavamale — nii et juurkaustas olevad taseme alamkaustad ja
juurkataloog `.daq` võetakse kõik korraga kaasa. Iga salvestuse tase imporditakse ühe pildina, kusjuures
teised tasemed on kättesaadavad režiimidena, mitte ühe pildina iga taseme kohta.

**Taseme alamkataloogi** otsene nimetamine (nt `output/raw/`) toimib samuti. Sel juhul jääb juurkataloog
`.daq` kõrvale, seega kopeerige või suunake DAQ-lugemine kõrvale, kui tuletate uuesti radiomeetrilist
toodet kataloogist `raw/` – vastasel juhul ei ole ajamärgise vastavusel millegi suhtes lahendada.

**Töötlemine algab alati kaustast `raw`.** Iga salvestuse puhul on töötlemisvoo allikaks töötlemata kaader;
`debayered`, `radiance`, `reflectance` ja `preview` on vaatamisrežiimid, kuid neid ei suunata kunagi
tagasi töötlemisahelasse. Tuletatud toote uuesti töötlemine rakendaks uuesti vinjetti, CCM-i ja
kiirguse matemaatikat, mis on juba selle pikslitesse sisse põletatud, seetõttu loobub „Chloros“ sellest, selle asemel et
teostada topelt-töötlust. Kaks teadmisväärset tagajärge:

- Renderdusi `index/` ja `composite/` **kunagi** ei töödeldata. Need on väljundid, mitte salvestused —
  NDVIi LUT-renderdusel puudub radiance&#x27;i mõtestatud tõlgendus.
- Salvestuste kaust, mis on eksporditud **ilma** `raw` (nt `array-capture --processing reflectance`) ei ole
  puudub õiguspärane töötlusvoo allikas. Need salvestised imporditakse ja kuvatakse normaalselt, kuid `process` jätab
  need vahele ja teatab sellest järgmiselt:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Kui teil on tõepoolest vaja läbi suruda tuletatud toodet – näiteks hub-seanss, mis on salvestatud
  `demosaic` sisselülitatuna, või vana kaust –, siis `--input-level {raw,debayered,processed}` sunnib sissekande
  ja tühistab vahelejätmise. See lipuk on tahtlik pääsetee; `auto` (vaikimisi)
  ei töödelda kunagi salvestust, millel puudub toorandmed.

### Vahelejäetud salvestused segafiltreid kasutavates massiivides

Kui ühes massiivis segatakse „RGB“ ja multispektraalkaameraid, salvestab `array-capture --processing radiance` (või `reflectance`) multispektraalkaadrid ja **vaatab üle** „RGB“ kaamerad — Bayeri-põhine kiirgus ei ole lairibaandurite puhul oluline. CLI-fail väljastab iga salvestatud faili (koos selle eksporditasemega), iga kirjutatud `.daq`-i ja iga vahelejätmise selgesõnaliselt, seega ei ole failide arv üllatav:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Vahelejätmise põhjuse märgid järgivad mustrit `<level>-not-applicable-to-rgb-cam`. Peegeldusvõime võib samuti vahele jääda märkidega `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` ning `dls-uncalibrated-band-<nm>`, kui lainepikkus jääb peamiselt väljapoole DAQ-valgusanduri radiomeetriliselt kalibreeritud vahemikku (~374–974 nm) – saadaval olevatest toodetest ainult F988, mille toetatud töövoog on peegeldusvõime-paneeli töövoog.

Kasutage `--processing debayered` (või `display`), et hõlmata kõiki kaameraid sõltumata filtri tüübist, või vaikimisi `all`, et saada ühe korraga kõik sobivad tasemed iga kaamera kohta.

---

## Salvestusrežiimid, salvestid ja offline-taas töötlemine

Need kõik töötavad **püsiva massiiviga** (käivita esmalt `array-connect`). Need peegeldavad graafilise kasutajaliidese salvestuspaneeli.

### `array-capture` režiimid

`array-capture` on üks käsk, millel on neli katikurežiimi ja ekspordivalikud:

| Režiim | Lipuk | Käitumine |
| --- | --- | --- |
| **Ühekordne** *(vaikimisi)* | (puudub) | Üks sünkroniseeritud salvestusgrupp, seejärel lõpetamine. |
| **Järjepidev** | `--continuous` | Järjestikused läbiviimised kuni `Ctrl+C`, `--count N` või `--duration S`. |
| **Intervall** | `--interval S` | Üks läbimine iga `S` sekundi järel (mõõdetuna iga läbimise algusest), samad piirid. |
| **Kiireim** | `--fastest` | Ainult toorandmed + määratud DAQ-näidud + kombineeritud indeksiga komposiit; jätab kiirguse/peegelduse/kuvamise arvutused vahele, et kaader laadiks kiiresti. Eeldab `--processing raw --force-daq`. Töötle salvestatud `.daq` hiljem ümber kalibreeritud toodeteks. |

Ekspordilülitid (kombineeritavad mis tahes režiimiga; kõik kasutavad ühist GUI/ SDK lõpppunkti):

| Lipuk | Mõju |
| --- | --- |
| `--processing LEVEL` | Üks eksporditase või `all` (vaikimisi). |
| `--levels L1,L2,…` | Eksporditüüpide selgesõnaline alarühm (nt `raw,radiance,reflectance`); **asendab `--processing`**. |
| `--aligned` / `--no-aligned` | Iga elemendi mitte-toorekspordi moonutamine massiivi [joondusprofiili](#joondus) (koosregistreeritud). Toores andmestik jääb joondamata, kuid kannab teisendust metandmetes. Kui massiivil puudub profiil, kasutatakse tagavarana joondamata versiooni (hoiatusega). |
| `--index` / `--no-index` | Salvesta / jäta vahele kaamera-põhine taimestikuindeksi ülekanne, kui see on konfigureeritud. Vaikimisi: renderda see. |
| `--force-daq` | Salvesta määratud DAQ/DLS-näit `.daq`-sidecarina isegi siis, kui ükski valitud tase seda ei vaja (nt ainult toorandmetega kaadri salvestamine), et kaadreid saaks offline-režiimis uuesti töödelda peegeldusvõime/indeksiks. |
| `--smart` | Oota, kuni AE on kõigil kaameratel stabiliseerunud, enne käivitamist (vt [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF pikselite pakkimine. `deflate` (vaikimisi) = kadudeta zlib L1 + horisontaalne ennustaja, ~4,1 MB täisresolutsiooniga kaadri kohta; `none` = pakkimata, ~5× kiirem kirjutamine, ~6,3 MB kaadri kohta — kasuta maksimaalse püsikiiruse saavutamiseks, kui kettaruum seda võimaldab. Mõlemad on kadudeta ja loetakse importimisel ühtmoodi. |

> **Ühekordne kirjutamine TIFF + püsikiiruse mudel.**Pildid kirjutatakse**ühe**TIFF-faili käiguga, mis sisaldab piksleid + XMP + IFD0 tootja/mudel (mõõdetud täisresolutsioonilise Mono12 puhul: 36 ms pakitud / 6,5 ms pakkimata, võrreldes ~148 ms-ga vana kirjutamise-ja-ExifTooli ümberkirjutamine); ainus järelejäänud ExifTooli töö (EXIF-i sub-IFD-i viimistlus) töötab asünkroonses taustaprotsessis ning kaader on valmis ja importimiseks sobiv isegi siis, kui see protsess kunagi ei käivitu. Pange tähele, et DEFLATE-pakkimine hoiab kinni Python GIL-ist, seega pakitud kirjutamised**ei**toimu paralleelselt kaamerapõhiste kirjutajate niitide vahel — püsiv 8-kaameraga täisresolutsiooniga salvestamine anduri kiirusel (~10,4 kaadrit sekundis) nõuab `--compression none`-i**ja** NVMe-klassi kõvaketast (~500 MB/s püsivat kirjutuskiirust). Sama seadistus on `POST /api/camera/array/capture`-is kättesaadav nimetusega `compression`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — kombineeritud indeksiga video/GIF (jälgimiskvaliteediga)

Salvestab kõik, mida **reaalajas kombineeritud indeksivaade** näitab, `.avi`-ile (ja valikuliselt `.gif`-ile). Kuna see kasutab reaalajas komposiitvoogu, peab kombineeritud voog olema avatud (nt kui massiivi eelvaadet kuvatakse graafilises kasutajaliideses) , et kaadrid jõuaksid salvestusse. See kontrollib edusamme iga 2 sekundi järel ja peatub seadmetel `--duration`, `Ctrl+C` või kui salvestaja lõpetab töö ise.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Lipuk | Vigaviga | Kirjeldus |
| --- | --- | --- |
| `--array-id ID` | ainult massiiv | Sihtmassiiv (jätta välja, kui on ühendatud vaid üks). |
| `-o, --output DIR` | `output` | Väljundkataloog (tagapõhja-kohalik). |
| `--fps F` | `10` | Salvestamise kaadrisagedus. |
| `--duration S` | kuni Ctrl+C | Automaatne peatamine pärast `S` sekundit. |
| `--gif` | välja lülitatud | Kirjuta ka animeeritud GIF. |
| `--gif-only` | välja lülitatud | Kirjuta ainult GIF (ilma `.avi`). |

### `array-burst` — toores Bayer-formaadis kõrge kaadrisagedusega sarivõte (analüüsikvaliteediga)

Loeb otse kaadrivõtu tsükli sünkroniseeritud rühma puhvrit — **ilma kalibreerimisahelata, ilma exiftoolita, ei vaja reaalaja vaadet** — seega töötab kaamera täieliku pildivõtusagedusega. Kirjutab RAW-kaadrid + kaadri-põhise manifesti + ühe `.daq` iga eraldi DLS-lugemise kohta `<output>/bursts/<base>/` all. Töötle uuesti offline-režiimis (järgmine käsk) või edasta `--build`, et teha seda kohe peale peatamist.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--array-id ID` | ainult massiiv | Sihtmassiiv. |
| `-o, --output DIR` | `output` | Väljundkataloog (burst salvestatakse `<DIR>/bursts/<base>/`). |
| `--duration S` | kuni Ctrl+C | Automaatne peatamine pärast `S` sekundit. |
| `--max-frames N` | piiramatu | Automaatne peatamine pärast `N` toorkaadrit. |
| `--build` | välja lülitatud | Peatamise järel töötle pildiseeria kohe uuesti (sama mis `array-build-video`). |
| `--products …` | `combined:index` | Koos `--build`-ga: millist(eid) videot(eid) luua (vt allpool). |
| `--fps F` | `10` | Koos `--build`: väljundvideo kaadrisagedus (fps). |
| `--save-tiffs` | välja lülitatud | Koos `--build`-iga: salvestatakse ka iga kaadri kohta kalibreeritud TIFF-failid. |
| `--gif` | välja lülitatud | Koos `--build`: salvestatakse ka animeeritud GIF-failid(id). |

### `array-build-video` — salvestatud pildiseeria offline-taas töötlemine

Sünkroniseerib iga töötlemata kaadri ajaliselt lähima salvestatud `.daq` näiduga ja suunab selle läbi **sama kiirguse / peegeldusvõime / indeksi ahela kaudu nagu importimise töötlemisahel**, renderdades ühe või mitu videot.

`--products` on komadega eraldatud nimekiri `kind:level` elementidest, kus `kind` ∈ `per_cam` | `combined` ning `level` ∈ `radiance` | `reflectance` | `index`. Tühja `level` (ilma `kind:`-ita) on vaikimisi `per_cam`. Vaikimisi on `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--burst-dir DIR` | (nõutav) | Teekond burst-kausta (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level` nimekiri, nagu eespool. |
| `--fps F` | `10` | Väljundvideo kaadrisagedus (fps). |
| `--save-tiffs` | välja lülitatud | Salvesta ka video(de) kõrvale iga kaadri kohta kalibreeritud TIFF-failid. |
| `--gif` | välja lülitatud | Salvesta ka animeeritud GIF-failid. |

> **Vali õige salvestaja.** `array-record` on *jälgimiskvaliteediga* — see salvestab ekraanil kuvatavat reaalajas komposiitpilti ja vajab avatud voogu. `array-burst` → `array-build-video` on *analüüsitasemel* — see salvestab tooresensorandmeid täiskiirusel ja rekonstrueerib kalibreeritud kiirgust/peegeldust/indeksit käsitlevad videod hiljem, ilma et oleks vaja reaalajas vaatamist.

### Mono (M3M) ühe sagedusribaga kaamerad

**M3M**-seeria on Bayeri**M3C**mono-versioon: üks kitsasribaline interferentsfilter kaamera kohta (`M3M-<lens>-F<wavelength>`, nt `M3M-L87-F685`), seega edastab andur**ühe halltoonide sagedusriba** ilma Bayeri mosaiigita. Ei ole vaja teha demosaiki, ei ole vaja eraldada kanalitevahelist ülekandumist ega seadistada valgebalanssi – kogu värviprotsess „RGB“ lihtsalt ei kehti.

Mida see tähendab mudelite puhul CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**tuvastavad monokamera ja**jätavad selle vahele üherealise teatega**, selle asemel et suruda peale mõttetuid seadeid. Samal seansil töötavad need endiselt normaalselt ka RGB /Bayer M3C kaameraga samas seansis.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** töötavad endiselt — kiirgus ja peegeldus on *ribalised* radiomeetrilised kaardid ning on ühe riba jaoks täiesti hästi määratletud. Monokroomkaadrid sisaldavad **identiteetset** anduri vastusmaatriksit (ilma 3×3 segamiseta), seega läbib tasand kalibreerimisarvutused muutmata kujul.
- **Üksik monokromaatiline kaamera ei suuda luua taimestiku indeksit.**NDVI / NDRE / jne vajavad vähemalt kahte laineala (nt Red + NIR). Et saada indeksit monokromaatilisest riistvarast, suuna**mitu** M3M-kaamerat erinevatele lainepikkustele, joondage need üheks mitme lainepikkusega pildistamiskogumiks ja indekseerige *seda*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` sümbolid peavad **täpselt** vastama eelseadistuse kanalite nimedele (suurtähti arvestatakse; NDVI on väiketähed `red`,`nir` — vt `--list-presets`), ning sagedusriba nimi peab vastama ühendatud kogu sagedusribale (offline-režiimis aktsepteeritakse ka 0-põhiseid sagedusriba indekseid, nt `--channel red=0 --channel nir=1`).

Kogu virna eristajaks on mudelistringis olev märk `M3M` (see ei esine kunagi stringis `M3C`), mis kuvatakse kasutajaliideses (GUI) / veebiliideses (SDK) kujul `is_mono`.

---

## Host-võrgukaardi seadistamine ja häälestamine (LATTICE-massiivid)

LATTICE-kaamerad edastavad GVSP-d hosti Ethernet-adapteri kaudu, seega mitme kaameraga massiivide puhul ei ole adapteri **draiver**ja**vastuvõturingi suurus** on sama olulised kui ühenduse kiirus. Valed seaded kuvatakse massiivi seadete paneelis veana `FRAMES WILL DROP` / `Reduce ROI to enable` (ja `lattice network-analysis` / SDKi `analyze_array_network()`), isegi kui kaamerad ise töötavad korralikult.

### USB 10GbE-adapterid — Realtek RTL8157 („Realtek USB 10GbE Family Controller”)

| Kirje | Nõutav väärtus | Miks see on oluline |
| --- | --- | --- |
| **Draiveri versioon**|**≥ v10.67 (jaanuar 2026)**, INF `rtump64x64sta.inf` | Vana**

2016. aasta**draiver (v10.65, `rtump64x64.inf`) ei suuda õigesti käsitleda väljalülitamist ja tekitab süsteemirikkeid**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**. Üleminek jääb kinni (~5-minutiline ooteaeg), kasutaja lülitab seadme sundkorras välja ning korduvad ebaõiged väljalülitused**rikuvad WMI-andmebaasi**(PowerShell ja tööriistad hakkavad `Invalid class` tõrkeid andma) ning**blokeerivad USB-pinu** järgmisel käivitamisel (võrgukaart ei aktiveeru; USB-seadmete loendamine peatub). Lae alla värskendus realtek.com-ist (või dongli tootjalt), enne kui loodad puhtale taaskäivitamisele. |
| **Vastuvõtupuhvrid**— märksõna `ReceiveBufferLen` |**256**(draiveri maksimaalne väärtus) | Võrgukaardi vastuvõturõngas. Draiveri vaikimisi väärtus**32**jätab kasutatavaks vaid ~0,26 MB — mis on mitme kaamera puhul liiga väike — seega kuvab massiivi paneel veakoodi `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` ja blokeerib ühenduse. Väärtusel**256**on rõngas piisavalt suur (**~13,5 MB, mõõdetud laboris 10GbE-hostil**), andes vastuvõtu torule piisava varu mitme kaameraga GVSP-pursete jaoks. (Kas antud konfiguratsioon tegelikult *ühendub*, otsustavad kaks kontrolli – **drain-aware**vastuvõtukontroll ja**agregaatne ülemärkimise** kontroll – mitte toores pursete ja ringi võrdlus; vt [Massiivi kaadrisagedus ja pursemudel](#array-fps--burst-model).) |
| **Vastuvõtu-URB-d**— märksõna `PendingReceives` |**64** (maks) | Lennus olevad USB-päringublokid; suurendatakse koos vastuvõtupuhvritega pursete neeldamiseks. |
| **Jumbo-raam** — märksõna `*JumboPacket` | **9014** | Vajalik 9000-baidiste GVSP-pakettide jaoks (6× vähem pakette raami kohta kui 1500-baidiste puhul). |

> ⚠️ **Võrgukaardi draiveri uuendamine TAASTAB need täpsemad omadused vaikimisi seadetele.**Pärast adapteri draiveri uuendamist või asendamist**rakendage uuesti** `ReceiveBufferLen=256` ja `PendingReceives=64`, vastasel juhul hakkab massiivipaneel uuesti blokeerima, kuigi „riistvaras pole midagi muutunud“. See on peamine põhjus, miks varem töötanud seadmestik äkki ühendust keeldub looma.

Kohalda **kõrgendatud õigustega** PowerShellist (asenda oma adapteri nimega, nt `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` hõlmab USB 10GbE-adaptereid.** Nüüd tuvastatakse adapteri tüüp ja seadistatakse õige vastuvõturingi märksõna: `*ReceiveBuffers`→2048 PCIe-võrgukaartide puhul (Intel I219 jne) või `ReceiveBufferLen`→256 + `PendingReceives`→64 Realteki **USB** 10GbE kontrollerile (mis ei avalda `*ReceiveBuffers`-i). Sihtmäärad piiratakse iga draiveri poolt teatatud maksimumväärtusega (`NumericParameterMaxValue`), nii et kunagi ei kirjutata vahemikust väljas olevat väärtust. Käivita see **õigustega** terminalis; nagu iga registripõhisepõhinev seadistus, jõustub muudatus pärast adapteri taaskäivitamist või süsteemi taaskäivitamist. Eespool toodud käsitsi sisestatavad `Set-NetAdapterAdvancedProperty`-käskud on endiselt hea alternatiiv – need rakenduvad kohe (seovad adapteri uuesti) ilma taaskäivitamiseta.

### Võrgu põhitõed (kõik LATTICE-lingid)

- **Aadressimine:** link-local `169.254.0.0/16` (GigE Vision LLA). Host võtab staatilise `169.254.x.x/16`; kaamerad + DAQ-E määravad endale ise aadressi samas vahemikus. DHCP-d ega väravat ei ole vaja.
- **Paketi suurus:**eelistage jumbo (9000), kuid laske automaatsel proovil see leida — see mõõdab uuesti iga ühenduse korral ja ületab juba kaamera 1500-baidise ICMP-piirangu GVSP-proovi kaudu, nii et see jõuab jumbo-suuruseni seal, kus kaabel seda tõepoolest toetab. Kasutage `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`-i ainult siis, kui tead paremini kui proov, ja eelista käskupõhist seadet püsiseadistuse asemel: seadistusega jäetakse proov vahele, seega kui ühendus ei suuda tegelikult 9000-baidiseid pakette edastada, aegub**iga** salvestus `SC_ERR_TIMEOUT -1011` (vt [Keskkonnamuutujad](#environment-variables)).
- **RX-ringi suurus sõltub `ReceiveBufferLen`-ist:**vaikimisi `32` on kasutatav ring ~0,26 MB (liiga väike mis tahes mitme kaameraga pildiseeria jaoks); maksimaalse `256` väärtuse juures on see suur (~13,5 MB, mõõdetuna laboris 10GbE-hostil), pakkudes reaalset varu. Kas konfiguratsioon ühendub, otsustab seejärel voolutarbimist arvestav lubatavuse kontroll**ja** allpool toodud koondatud ülemärkimise kontroll – mitte toores võrdlus purske ja ringi vahel.

### Massiivi kaadrisagedus ja purskemudel

Kuidas lugeda massiivi seadete paneeli (ja `lattice analyze-array` / „SDK” `analyze_array_network`):

- **Purske maht liidetakse iga kaamera puhul kokku vastavalt kaamera tegelikule pikseliformaadile.**Mono**M3M**kaamerad edastavad**Mono12 (2 bitti/piksel)**;**M3C**Bayeri kaamerad edastavad 8- või 12-bitist andmevoogu (TRI032S edastab vaikimisi BayerRG12-t isegi siis, kui nõutakse BayerRG8-t). Seega on 4-kaameraga täisresolutsiooniga kaadri maht**~12,6 MB, kui kõik on 8-bitised, kuid ~25 MB kolme 12-bitise mono kaameraga**. Projektsioon määrab iga kaamera formaadi kindlaks selle mudeli järgi (identiteedi vahemälu), seega vastab burst sellele, mida andmeside tegelikult edastab — mitte ühtsele BayerRG8 eeldusele.
- **USB-Etherneti adapteri maksimaalne kiirus on 200 MB/s, hoolimata selle nimikilbil märgitud andmetest.** Tõhusustabel, mis teisendab ühenduse kiiruse püsivaks näitajaks, on tuletatud PCIe-st; USB-võrgukaart reklaamib oma *Etherneti* ühenduskiirust, kuid on piiratud USB-siini ja selle draiveriga. USB 10GbE-dongel saavutas varem ~1063 MB/s „püsikiiruse“ — näitaja, mida kunagi ei uuritud – ning sellest tulenev andmeedastuse takistus rikkus 6–18 % raamidest, kuigi näidati endiselt korrektset siht-fps-i. USB-ga ühendatud võrgukaartidel on nüüd absoluutne piir **200 MB/s** (piiranguks on siin siin, seega see ei sõltu nimikilbil märgitud kiirusest; USB 1 GbE-adapter saavutab ~80 MB/s ja see piirang teda ei mõjuta). `wire_ceiling_source` võimekuse kirjes ütleb seda sõnadega ja `nic_is_usb` märgistab seda. Mõlemat saab ümber kirjutada `--wire-ceiling-mbps` abil.
- **Läbilaskvus arvestab andmete väljaviimist, mitte kogu purset võrreldes ringiga.** Samaaegne purske peab mahuma vaid *ajutisse järjekorda* = `max(0, Σ per-cam arrival − host drain) × emit_window`, mitte kogu purske. Kiirel hostil / aeglase CAM-iga struktuuris (**PCIe**10G host + 4× 1 GbE CAM-id: saabumine ≈ 320 MB/s, väljastus ≈ 1063 MB/s) tühjeneb host kiiremini, kui CAM-id täituvad, tagantjärele jääv maht ≈ 0, seega**lubab**täisresolutsiooniga sim-emit andmeid sisse, kuigi 25 MB andmepakett ületab 13,5 MB ringi mahutavust. Kui panna samad neli kaamerat**USB**10GbE-adapteri taha, on väljastuskiirus 200 MB/s, mitte 1063 — sissetulev andmevoog ületab väljastuskiirust ning kadu ilmneb pigem vigaste raamidena kui madalama kaadrisagedusena. 1 GbE hostil paneb kaamerate 31,25 MB/s DLThr alampiir põhjustab, et saabuv andmevoog ületab väljundvõimsuse → see**blokeerib**(selle tüüpi blokeerimise puhul vähendage ROI-d või kasutage binningut ≥ 2). Sissepääs on üks**kahest** ühendusväravast – teine on allpool kirjeldatud koondatud ülemärkimise kontroll.
- **Prognoositav kaadrisagedus (fps) on konservatiivne järjestikuse väljavõtte ülempiir.**Host-arvuti haaramistsükkel võtab praegu iga kaamera puhvri**serial**(~üks kaamera kohta väljastusaken), seega on tsükkel piiratud `max(readout+emit, N × emit)`-ga, kus kaamera kohta väljastus on piiratud kaamera**juurdepääsulinkiga**(1 GbE ≈ 80 MB/s), mitte hosti üleslingiga. 4kaameraga täisresolutsioonilise 12-bitise massiivi puhul on see**~2,8 kaadrit sekundis**, mis vastab mõõdetud ~2,7–3,0-le. Kaadrisagedus on tahtlikult**säriajast sõltumatu**, mistõttu võib hämarates stseenides tegelik kiirus ekspositsiooni pikenemisel langeda veidi alla ülemmäära. Põhinev kiirus on tegelik kaadrisageduse piiraja; selle paralleelistamine tõstaks ülemmäära ühe kaamera andmeedastuskiiruse suunas.
- **Kogusumma ülemäärane kasutamine on tõsine ühenduse takistus.**Kaamera kohta eraldatava ribalaiuse alampiir on**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), seega kui alampiir saavutatakse, võib koondnõudlus (`per_cam × N`) ületada**kokkupõrkevaba ühenduse ülempiiri**(`sustained × sim_emit_factor`). Praktilised täisresolutsiooni ülempiirid 1 GbE-võrgus:**6 kaamerat 1500 MTU-ga, 9 jumbo-raamidega**. See ülempiir sõltub ainult ühendusliini ja alampiirist – see on**raami suurusest sõltumatu**, seega**andmete koondamine ja väiksem huvipiirkond (ROI) EI AITA** (need vähendavad baitide arvu *raami* kohta, mitte GevSCPD-tempo järgi baitide arvu *sekund*); ainsad lahendused on vähem kaameraid, jumbo-raamid algusest lõpuni või kiirem võrgukaart. Sümptomiks oleks GVSP-pakettide kadu, mitte sujuv kaadrisageduse vähenemine, seega nullib `analyze-array` saavutatava kaadrisageduse näitajad ja väljastab `**OVER-SUBSCRIBED**` ning `array-connect` koos fikseeritud resolutsiooniga **keeldub ühendusest** (muidu „walk-down“ sorteerib kaadrid madalamasse kategooriasse, mis samuti ei kõrvalda seda tüüpi blokeeringut). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` alandab keeldumise valjuks hoiatuseks töölaua jaoks — vaata [Keskkonnamuutujad](#environment-variables).

### Massiivi tervis — milline alamsüsteem kaotab kaadreid

Ühendatud massiivi `GET /api/camera/array/<array_id>/capability` kannab reaalajas
`health`-plokki, mida hinnatakse uuesti jooksvas **10-sekundilise** ajavahemiku jooksul. See jagab kaadrite kaotuse
kaheks põhjuseks, mis vajavad vastupidiseid parandusi, selle asemel, et teatada ühest „ebatäielikust“
määrast, mis ei nimeta kumbagi:

| Väli | Mida see tähendab | Milline alamsüsteem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (seerianumbri kaupa) | Kaader **saabus, kuid oli struktuuriliselt vigane**— GVSP-pakettide kadu. |**Võrk**: ühenduse läbilaskevõime, tempo, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (seerianumbri kaupa) | Raam **ei jõudnud üldse kohale**— kaamera ei käivitatud või sellest ei väljunud midagi. |**Trigger / sünkroniseerimine**: M8-kaabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Iga kaamera halvim edastuskiirus. | — |
| `per_cam_rate_pct` | Kaamera kohta kombineeritud puudulikkuse määr (mõlemad põhjused kokku). | — |
| `stable_for_seconds` | Kui kaua on iga kaamera püsinud alla 0,01 %. | — |

Üle 5 % registreerib tagapõhi `[array-health <id>] WARN`-rea, milles nimetatakse jagunemist —
esimesel rikkumisel, tõsiduse vahemiku muutumisel, kord minutis, kuni see püsib, ja üks kord, kui
olukord normaliseerub. Rikkis pool väljastab `[gvsp-corrupt <SN>]` iga kaamera ja
põhjuse esimese tabamise korral, seejärel koondandmed iga 60 sekundi järel. Iga hindamine jõuab ikkagi tagapõhja logifaili;
loendurid liiguvad edasi iga puhvri puhul, sõltumata sellest, mis väljastatakse.

Samas kirjes on esitatud ka number, millest kogu eraldis sõltub:

| Väli | Mida see tähendab |
| --- | --- |
| `wire_ceiling_mbps` | Hosti kehtiv püsiv andmeedastusmaht, MB/s. |
| `wire_ceiling_source` | Kust see number pärineb, sõnadega — nt `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` või `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, kui `--wire-ceiling-mbps` (või graafilise kasutajaliidese **Wire Budget**-väli) seda seadistab. |
| `nic_is_usb` | `true` USB-Ethernet-adapteri puhul — vt eespool mainitud 200 MB/s piirangut. |

**Tõlgendus:** kui `gvsp_corrupt_rate_pct` on nullist erinev ja `never_arrived_rate_pct` on 0
tähendab, et triggereerimine ja kaabli sünkroniseerimine on täiuslikud ning 100 % kaotusest on võrguteel
— vähendage `--wire-ceiling-mbps` väärtust ja ühendage uuesti. Vastupidine muster viitab hoopis
sünkroniseerimiskaablile või triggereerimisliinile.

> **`--target-fps` ei ole rikutud raamide puhul otsustav tegur.** GevSCPD-i takt on kirjutatud
> ühenduse loomisel üks kord, seega triggereerimissageduse alandamine muudab töötsükli, mitte
> samaaegse saatmise pursesagedust. Mõõdetud 5-kordne nõudluse vähendamine ei toonud kaasa parandust;
> juhtme ülempiiri alandamine 240-lt 200 MB/s-ni viis sama seadme vigaste raamide määra 10,4 %-lt
> 0,00 %-ni.

> **TRI032S-i püsivara puhul ei ole voo keskel toimuv automaatne kokkutõmbumine saadaval.** Töötav massiiv
> ei suuda seda ise parandada; katkestage ühendus ja ühendage uuesti, et ühendusaja valija saaks
> uue ülempiiriga uuesti planeerida.

### Sümptom → lahendus

| Sümptom (Massiivi seaded / ühendus / `analyze_array_network`) | Põhjus | Lahendus |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` taastamine väärtuseks 32 (tavaliselt pärast draiveri uuendamist) | Määrake `ReceiveBufferLen`→256, `PendingReceives`→64; avage paneel uuesti (käivitage tagapõhi uuesti, kui see on vanad ringisuurused vahemällu salvestanud) |
| Taaskäivitus/shutdown hangub; hiljem `Invalid class` WMI-vead, võrgukaart ei lülitu sisse, USB-draivid puuduvad | Vana 2016. aasta Realtek USB 10GbE draiver → BSOD `0x9F` → sunnitud väljalülitumised | Uuendage adapteri draiver versioonile ≥ v10.67 (2026), seejärel rakendage uuesti eespool nimetatud vastuvõturõnga |
| Ühendamine õnnestub, kuid tagastab sub-native resolutsiooni | Smart-prep vähendas raami automaatselt, et see mahuks kaabli läbi | Uuenda ühendust / aktsepteeri vähendamist / `--force-tier slip-emit-and-capture` |
| Massiiv näitab korrektset siht-fps-i, kuid edastab sellest vaid murdosa; `health.gvsp_corrupt_rate_pct` mittenullist erinev, `never_arrived_rate_pct` 0 | Hosti järeldatud ühenduse läbilaskevõime ületab tegelikku võimsust (tüüpiline USB-Etherneti adapteri, kitsa PCIe-liini või jagatud võrgustiku puhul) | Ühenda uuesti madalama `--wire-ceiling-mbps`-iga ja kontrollige terviseblokki uuesti. **Mitte** `--target-fps` — GevSCPD-i taktsagedus on ühendamisel fikseeritud |
| Kaamerad puuduvad avaldatud rühmadest; `health.never_arrived_rate_pct` ei ole null, `gvsp_corrupt_rate_pct` 0 | Triggeri / sünkroniseerimistee — kaamerad ei käivitu, ei ole võrguprobleem | Kontrollige M8 sünkroniseerimiskaablit ja `--line`; veenduge, et kõik liikmed on sisse lülitatud (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` ületatud väärtusega `analyze-array` või ühenduse keeldumine fikseeritud resolutsiooniga (`array over-subscribes the wire`) | Kaamerate kogunõudlus (8 MB/s miinimum × N kaamerat) ületab kokkupõrkeohutu ühenduse ülempiiri — 6 kaamerat täisresolutsiooniga 1 GbE-võrgus @1500 MTU, 9 kaamerat jumbo-raamidega | Vähem kaameraid, jumbo-raamid-to-end või kiirem võrgukaart. **ROI/binning EI aita** (piirang ei sõltu raami suurusest). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` tühistab piirangu katsetingimustes (aktsepteerib pakettide kadu) |

---

## `chloros-cli daq`

Spektraalsensori käsud. Kaks klassi:
- **`pool-*`**— õhukesed HTTP kliendid, mis juhivad andurit tagapõhja püsiva reservi kaudu.**See on toetatud tee ja ainus, mis on kaasasolevas CLI-is olemas.** Tagapõhi omab transporti, seega jagavad GUI, skriptid „CLI“ ja „SDK“ ühte aktiivset käepidet, selle asemel et võidelda saripordi pärast.
- **Kõik muu**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — otsene riistvarale juurdepääs, täielikkuse huvides allpool dokumenteeritud. Nende jaoks on vaja `daq` Python paketti, mis**ei kuulu ühegi tarnitud artefakti koosseisu**: kompileeritud CLI ei sisalda seda (`scripts/Build-CLI.ps1` määrab selle väärtuseks `--nofollow-import-to=daq` ning sellega kaasnevad transpordid `pyserial` / `bleak` / `zeroconf` seda ei sisalda), samuti ei sisalda seda PyPI-SDKi pakett. Need töötavad ainult allikakoodi väljavõttest, seega käsitle neid pigem MAPIR -sisese arendusteena kui midagi, mille poole püüelda.
- **`discover` / `list`** asuvad kahe vahel: need on allikakoodist otseselt käivitatavad riistvarakäsud, kuid väljalaskekompilatsioonis kasutatakse nende asemel `pool-discover`-i ja tagapõhi teostab skaneerimise. Seega toimib skannimine kõikjal — mis on oluline, sest see on ainus viis DAQ-M-i BLE MAC-i teada saada.

> **`chloros-cli daq --help`** (ja `-h` / `help`) loetleb `pool-*` alamkäsklusi – abi suunatakse tahtlikult pool-kliendile, et see kajastaks tegelikult käivitatavaid käske. Kui käivitad tarnitud versioonis otsese riistvarakäsu, lõpetab programm töö selge veateatega, milles nimetatakse puuduvat paketti ja suunatakse sind tagasi `pool-*`-i juurde; midagi ei ebaõnnestu märkamatult. (`discover` / `list` on erand — need suunatakse ümber `pool-discover`-ile ja lihtsalt töötavad.)
>
> **Kõik, mida klient vajab, on kättesaadav `pool-*` kaudu** – ühendamine, andmevoog, kalibreeritud `.daq`-failide salvestamine ja kap-profiilide vahetamine. DAQ-seadet saab juhtida ka veebiaadressilt Python, kasutades `chloros_sdk.connect_daq_sensor()`-i abil, mis kasutab sama ühist teekonda.

### DAQ-anduri esmakordse ühendamise töövoog

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` viide

| Alakäsk | Otstarve |
| --- | --- |
| `daq pool-connect` (smart-detect) | Avab anduri tagapoolse koondpools. |
| `daq pool-connect --port PORT` | DAQ-U konkreetsel jadaportil. |
| `daq pool-connect --ble` | DAQ-M BLE kaudu, MAC-aadressi automaatne skannimine. |
| `daq pool-connect --mac MAC` | DAQ-M BLE kaudu teadaoleva MAC-aadressiga (eeldab `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E Etherneti kaudu teadaoleval hostil. |
| `daq pool-connect --eth` | DAQ-E Etherneti kaudu, host automaatselt avastatud (mDNS + ARP varuvariant; töötab tühja ARP-vahemäluga veebisaitidel Windows ja Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Integreerimisakna / AE-oleku häälestamine. |
| `daq pool-connect --no-stream` | Ühendus loodud, kuid voogedastust veel ei alustata (jätka `pool-stream --start`-ga). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-korrektsiooni profiil. Tagapõhiselt on vaikimisi seadistatud `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skaneeri iga transporti, et leida andureid, millega saaksid ühendust luua, ilma ühendamata. **Nii leiad DAQ-M-i BLE MAC-aadressi.** `daq discover` / `daq list` suunatakse siia automaatselt tarnitud versioonides. Poolis juba avatud andureid ei näidata — ühendatud DAQ-M lõpetab reklaamimise — seega kasuta nende jaoks `pool-list`-i. |
| `daq pool-list` | Näita kõiki andureid tagapõhja andurite kogumis. |
| `daq pool-disconnect --sensor-id ID [--all]` | Vabasta. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Viimasedhiljutist N spektri kaadrit. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Jätka / peata voogedastus. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Alusta / lõpeta .daq-salvestus. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Vaheta cap-korrektsiooniprofiili käitamise ajal. |

### Otse riistvarale suunatud alamkäskud (ainult lähtekoodi väljavõte — ei kuulu tarnitud versioonidesse)

> Loetletud täielikkuse huvides. Need nõuavad `daq` Python paketti ning `pyserial` / `bleak` / `zeroconf`, millest ükski ei kuulu kompileeritud versioonidesse CLI ega PyPI-sse SDK — need töötavad ainult allalaaditud lähtekoodist MAPIR. **Kui kasutate avaldatud versiooni Chloros, kasutage selle asemel ülaltoodud `pool-*` käske**; need hõlmavad ühendamist, voogedastust, salvestamist ja kapti valikut.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Avab, ühendub ja juhib salvestatud Chloros projekti (kaust, mis sisaldab `cameras.json` + `sensors.json` + `project.json`). Kõik suunatakse tagapõhja kaudu, nii et graafiline kasutajaliides ja CLI kuvavad identse riistvara seisundi.

### Alakäskude viide

| Alakäsk | Otstarve |
| --- | --- |
| `project open PATH` | Prindi projekti seadmete manifest (kaamerad, massiivid, andurid). |
| `project devices PATH [--reconnect]` | Näita loendit või käivita avastamine uuesti. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Ühendab kõik salvestatud kaamerad / massiivid / andurid. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Üksikpilt nimetatud kaamerast või massiivist. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | N-kaadriline pildiseeria nimetatud kaamerast või kaamerarühmast (`-n/--count` vaikimisi 5; `-i/--interval` kaadritevaheline aeg sekundites, vaikimisi 0). Kaamerarühma pildiseeriad eemaldavad dubleeritud sünkroniseeritud rühmad (vananenud andmete järelevalve), nii et osaliselt tsükliline kaamerarühm ei saa tagastada ühe kaadri N koopiat; väljastab-iteratsiooni tulemused. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Andmevoo salvestamine kettale tagapõhjaülesande kaudu. `--poll-interval` = sekundid `/stats` küsitluste vahel (vaikimisi 2,0). |
| `project sensor read PATH NAME [--json]` | Viimane spektriraam. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Salvestab .daq-faili. |
| `project run PATH RECIPE.yaml` | YAML/JSON-salvestusretsepti käivitamine. `--dry-run` valideerib ilma käivitamiseta. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Arvutab massiivi joonduse — vt [allpool olevat liputabelit](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Prindi praegune joondamisprofiil. |
| `project align clear PATH NAME` | Tühista vahemällu salvestatud profiil. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Ühe orja teisendust nihutada. |
| `project align export PATH NAME --to FILE` | Profiili salvestada faili „JSON“. |
| `project align import PATH NAME --from FILE [--no-validate]` | Salvestatud profiili laadida. |

#### `project align calibrate` valikud

| Lipuk | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Joondamismeetod. **Need kirjapildid erinevad `lattice align-calibrate`**-st, mis võtab lühivorme `orb` / `akaze` / `phase`; need kaks käsku selle lipu puhul omavahel asendamatud. |
| `--model {translation, rigid, affine, homography}` | `affine` | Muuda mudelit sobivaks. |
| `--frames N` | `1` | Sünkroniseeritud kaadripildid keskmiseks. |
| `--reference SN` | põhikaamera | Viitekaamera seerianumber; kõik ülejäänud liikmed moonutatakse selle järgi. |
| `--max-features N` | `5000` | ORB-tunnuste arvu ülempiir. |
| `--ratio-threshold F` | `0.75` | Lowe&#x27;i suhtarvu test. |
| `--ransac-threshold-px F` | `3.0` | RANSACi sisepunktide künnis. |
| `--min-matches N` | `15` | **Kvaliteedikontroll** — lükka lahendus tagasi, kui siseväärtuste kokkulangevuste arv on väiksem kui see. |
| `--max-reproj-err-px F` | `4.0` | **Kvaliteedikontroll** — lükka lahendus tagasi, kui RMS-ülekandeviga ületab selle väärtuse. |
| `--checkerboard RxC` | — | Plaadi geomeetria `--method checkerboard`, nt `9x6`. |
| `--name PROFILE` | tühi | Salvestatud failis „JSON“ sisalduv profiili nimi. **Mitte massiivi nimi** — see on positsiooniline `NAME`. |

Need kaks kvaliteedikontrolli on põhjuseks, miks kalibreerimine võib lahendamisel õnnestuda, kuid
salvestamine siiski ebaõnnestuda: profiil, mis ei vasta ühelegi neist, registreeriks vaikimisi valesti iga
hilisemat salvestust, mistõttu see lükatakse tagasi, mitte ei salvestata.

### Näited

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

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

`project run RECIPE.yaml` aktsepteerib YAML- või JSON-faili, mis kirjeldab tegevuste jada:

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
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
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

Toetatud tegevused: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Tegevus `burst` võtab vastu `name` (nõutav), `count` (vaikimisi 5), `interval` (sekundites, vaikimisi 0), `output`, `format` ja `settings` (sama kaamerapõhine seadistus nagu `apply`); masspildistamine kasutab sama äsja sünkroniseeritud rühma järelevalvet kui `project burst`.

Käivita see:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Keskkonnamuutujad

| Muutuja | Mõju |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Asendab tagapõhja URL (vaikimisi `http://127.0.0.1:5000`) — **arvestatakse ainult käskude perekondades `lattice`, `project` ja `daq pool-*`.** Põhikäsud (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) seavad `http://127.0.0.1:<port>` ja ignoreerivad seda muutujat (IPv4-literaal möödub Windows `localhost`→`::1` ~2 s karistus iga päringu kohta), mistõttu suunatakse need alati kohalikule masinale. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` alandab massiivi ülemäärase tellimuse ühenduse keeldumise (kogunõudlus kaamera kohta &gt; kokkupõrkeohutu ühenduse ülempiir koos `pin_resolution`-ga) valjuks hoiatuseksja jätkab tööd, aktsepteerides GVSP-pakettide kadu. Ainult katsetamiseks — vt [Massiivi kaadrisagedus ja purskemudel](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Määrab CLI ise; käsib tagapõhjal võimaldada paralleelset töötlemist. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` jätab GVSP varuproovi vahele (ainult ICMP-tulemused). **See lülitab jumbo välja, see ei vaigista lihtsalt logi** — kaamera vastab DF-pingidele igal marsruudil vaid kuni 1500-ni, seega on see proov ainus viis, millega saab jumbo-pakette tuvastada. Säästab ~1 s kaamera kohta ühenduse kohta; kulutab ~1,45× võrgu maksimaalset läbilaskevõimet, kui võrk *oleks* suutnud jumbo-pakette edastada. SDK hoiatab, kui seda seadistad. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Kinnitab GVSP-paketi suuruse N baiti; jätab proovimise täielikult vahele. Eelista käsupõhist (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) püsiva seadistuse asemel: fikseeritud suurus ei kohane enam eesoleva võrguga ning 9000-i fikseerimine teel, mis ei suuda edastada jumbo-pakette, põhjustab **iga** salvestamise ajal ajapiirangu ületamise veaga `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Ületab Nuitka onefile’i väljavõtukataloogi. CLI kasutab automaatselt `/mnt/ssd/tmp`, kui see on olemas. |

---

## Väljumiskoodid

| Kood | Tähendus |
| --- | --- |
| `0` | Edukas. |
| `1` | Üldine viga (enamik alamkäskude vigu). |
| `2` | Argumendiviga. |
| `130` | Katkestatud klahvikombinatsiooniga Ctrl+C. |

---

## Vihjeid veaotsinguks

- **&quot;Login required&quot;** → Käivita sellel arvutil üks kord käsk `chloros-cli login EMAIL PASSWORD`.
- **„backend unreachable“** → Käivita töölaua rakendus „Chloros“ või käivita backend-binaar otse (`chloros-backend`) või kontrolli `CHLOROS_BACKEND_URL`, kui tegemist on kaugjuurdepääsuga.
- **`lattice`-käskude täitmine ebaõnnestub veateatega „LATTICE kaameradraivereid ei leitud”** → Arena SDK jooksuaeg ei ole installitud; CLI on saadaval koos `win32api`-iga aadressil Windows, kuid C-jooksuaeg on osa graafilise kasutajaliidese installijast.
- **Array connect / Array Settings näitab „FRAMES WILL DROP” või „Reduce ROI to enable”** → Host-võrgukaardi vastuvõturõngas on liiga väike (tavaliselt taastatakse see pärast võrgukaardi draiveri uuendamist väärtusele 32). Vaata [Host-võrgukaardi seadistamine ja häälestamine](#host-nic-setup--tuning-lattice-arrays) — seadista `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Arvuti hangub taaskäivitamisel/väljalülitamisel, seejärel WMI `Invalid class` / NIC-kaarti ei saa aktiveerida / USB-draivid puuduvad** → Vananenud USB 10GbE adapteri draiver põhjustab `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Uuendage adapteri draiverit — vt [Host NIC Setup &amp; Tuning](#host-nic-setup--tuning-lattice-arrays).
- **Jetsoni vahetusmäluga seotud hoiatus** → Lisa failipõhine vahetusruum; käsk „CLI“ väljastab täpsed `fallocate` / `swapon` käsud.
- **DAQ otsesed käsud puuduvad** → Oodatav: kaasasolev `chloros-cli` jätab tahtlikult välja `daq` paketi, seega on olemas ainult `pool-*` (PyPI SDK ei paku seda samuti). Kasutage `pool-*`, mis juhib sama andurit tagapõhja kaudu, või `chloros_sdk.connect_daq_sensor()` aadressilt Python.

---

## Vaata ka

- [Python SDK viide](sdk-reference.md) — iga CLI käsu programmeerimislik ekvivalent.
- [DAQ-andurite juhend](../daq/README.md) — andurispetsiifiline ühendamine + kalibreerimine.
- Veebidokumendid: `https://mapir.gitbook.io/chloros/cli`
