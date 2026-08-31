# Chloros Python SDK Viide

**Versioon:**

1.2.0**Koostatud:**

29.07.2026 kell 19:19 ·**Muudetud:**

30.08.2026**Pakett:** `chloros-sdk` (PyPI)**Sihtrühm:** Optimeeritud LLM-i kasutamiseks; inimesele loetav.**Kohaldamisala:** Kõik `import chloros_sdk` poolt avalikustatud avalikud klassid, funktsioonid ja abifunktsioonid, koos kopeeritavate ja kleebitavate näidetega, mis hõlmavad pilditöötlust, ühe kaamera juhtimist, sünkroniseeritud kaameramassiive, DAQ-andureid ja projekti automatiseerimist.

Kui soovite tutvuda vaid olulisemate teemadega, minge otse järgmistesse osadesse:
- [Paigaldamine ja kiirstart](#installation)
- [Smart-Connect LATTICE-kaameramassiivide jaoks](#smart-connect-for-lattice-cameras)
- [DAQ-andurite sessioonid](#daq-sensor-sessions)
- [Projekti automatiseerimine](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arhitektuur 60 sekundiga

SDK on õhuke Python kiht Chloros tagapõhja peal (sama Flask-server, mida kasutavad ka töölaua graafiline kasutajaliides ja CLI). Automaatika jaoks impordite `chloros_sdk` ja kutsute välja kõrgetasemelisi meetodeid; sisuliselt muutub iga kutsumine HTTP-päringuks kohalikule backendile pordil 5000 — `http://127.0.0.1:5000/api/...` (tahtlikult mitte `localhost`, mis suunatakse esmalt aadressile Windows ja maksab umbes 2 sekundit päringu kohta, kui backend toetab ainult IPv4-aadresse). Tagapõhjal on riistvarapool – kaamerad, DAQ-andurid, joondamisprofiilid, kaadripuhvrid –, mistõttu saavad skriptid SDK eksisteerida koos graafilise kasutajaliidesega, ilma et nad peaksid võitlema jadaportide või võrgukaardi ribalaiuse pärast.

Kasutate kolme kasutajaliidest:

1. **`ChlorosLocal` + vabad funktsioonid** (`process_folder`, `process_lattice_capture`) – pilditöötluse töötlusketi. Kogu kausta kalibreerimine, debayeri töötlus ja indeksite eksportimine ühe Python-kõnega.
2. **Smart-connect-käepidemed** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Avage püsiv tagapõhja-seanss reaalajas töötavale riistvarale. Sama „smart-prep” voog nagu graafilises kasutajaliideses: võrgu-proov, taseme automaatvalik, PTP, AE algandmed, GPIO-käivitaja konfiguratsioon.
3. **`ChlorosProject` / `open_project`** — Laadige salvestatud projekt (kaust, mis sisaldab `cameras.json` + `sensors.json` + `project.json`), ühenda kõik korraga ja salvestab andmeid nimeliste käepidemete abil.

Pinnad 1 ja 2 **käivitavad automaatselt kohaliku tagapõhja**, kui see veel ei ole kuulamisrežiimis (sama komplektis olev binaar, mida käivitavad GUI ja CLI) — seega töötab lihtne skript värskest shellist, ilma et peaksite backendit esmalt käivitama. Edastage `auto_start_backend=False`, et sellest loobuda (nt kui viitate kaug-backendile, mida kunagi ei käivitatud). Vaadake [Backendi automaatne käivitamine](#backend-auto-start). Surface 3 käitub teisiti: `open_project()` ei võta vastu parameetrit `auto_start_backend` ning `connect_all()` ei käivita tagapõhja kunagi — see proovib `http://127.0.0.1:5000`-i üks kord ja, kui keegi ei vasta, lülitub ta märkamatult tagasi otsesele (tagapõhja-vabale) `lattice_sdk` seadme juhtimisele. Ainult `proj.process()` ja `stream(..., overlays=True)` loovad viivitusega `ChlorosLocal()`-i (mis teeb automaatsetkäivitumist).

Kõik kolm on autentimispõhised: käivita masinal üks kord `chloros-cli login` või logi sisse töölaua graafilise kasutajaliidese kaudu. SDK-kutsed ilma kehtiva sessioonita tekitavad veateate `ChlorosAuthenticationError`.

Nõuded:
- Python 3.7+ (nagu pakettis märgitud; arendatud/testitud versioonil 3.10)
- Kohalikult installitud „Chloros“ töölaua rakendus (tagapõhja binaar on paigaldaja sees)
- Aktiivne sisselogimine aadressil Chloros+. Minimaalne tasand SDK / CLI on **Copper**või kõrgem (Copper / Bronze / Silver / Gold); tasuta**Iron**tasandil puudub juurdepääs SDK / CLI. Seda rakendatakse**serveripoolelt**: iga SDK / CLI märgistusega päring peab sisaldama nii aktiivset sessiooni kui ka tasulist paketti, vastasel juhul tagastab tagapõhi koodi `403` koos koodiga `error_code: PLAN_UPGRADE_REQUIRED` (mis kuvatakse `ChlorosLicenseError`-na abifunktsioonide poolt `ChlorosLocal` ja `ChlorosConnectError`-na abifunktsioonide poolt `connect_*`). Välja logitud kasutaja saab selle asemel koodi `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) – need kaks on erinevad, sest `chloros-cli login` uuesti käivitamine parandab esimese, kuid ei suuda parandada teist.
- Offline-kasutust toetatakse plaani armuaegade jooksul: taset loetakse serveri valideerimise vahemälust (5 minutit) või allkirjastatud, seadmega seotud litsentsi vahemälust (kuuplaanide puhul 30 päeva, aastaplaanide puhul kuni tellimuse lõppemiseni). Kui see ajapikendus lõpeb, muutub pakett tasuta versiooniks ja juurdepääs aadressidele SDK / CLI peatatakse, kuni seade suudab serveriga ühendust luua. `chloros-cli status` (`GET /api/license-status`) jääb tasuta tasemel kättesaadavaks, mistõttu on põhjus nähtav — see on ainus SDK / CLI marsruut, mis on taseme piirangust vabastatud.
- Windows 10/11 64-bitine, **Ubuntu 22.04 LTS või uuem**või Jetson (JetPack 6). Ubuntu 20.04**ei** toetatud: `.deb` sõltuvused tulenevad sellest, millega backend on seotud, sealhulgas `libc6 (>= 2.34)`, ning Focal tarnib glibc 2.31.

---

## Paigaldamine

Python SDK on õhuke Python kiht Chloros tagapõhja peal. Kõigi töövoogude jaoks, mis ulatuvad kaugemale mõnest üksnes andmekogumisele (DAQ) pühendatud töövoolust, peab olema **kohalikult paigaldatud Chlorosi töölaua pakett** (Windows paigaldaja või Linux `.deb`) — see pakett sisaldab backend-binaarfaili, Arena SDK-i käivituskeskkonda LATTICE-kaamerate jaoks ning kalibreerimispakette.

Viimased allalaadimised: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### 1. samm — Paigalda platvormipakett „Chloros“

#### Windows (.exe)

1. Lae alla `Chloros-Setup-x.y.z.exe` allalaadimislehelt.
2. Käivita paigaldaja ja järgi paigaldusviisardi juhiseid. Vaikimisi paigalduskataloog on `C:\Program Files\MAPIR\Chloros\`.
3. Käivita vähemalt üks kord veebiaadress Chloros ja logi sisse oma Chloros+ kontoga.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### 2. samm — Paigalda „Python” SDK

**Chlorosi paigaldaja sisaldab sobivat „SDK” wheel-faili.** Iga Windows paigaldaja ja Linux .deb-fail paigutab kettale faili `chloros_sdk-X.Y.Z-py3-none-any.whl`, mis vastab täpselt GUI / CLI / backend-versioonile. Sa ei pea PyPI-d jälgima, et versioonid oleksid sünkroonis.

#### Windows

Paigaldaja käivitab automaatselt `pip install`-i koos kaasasoleva wheel-failiga, kasutades teie süsteemi Python (eelistatakse käivitajat `py.exe`, tagavaraks kasutatakse `python -m pip`). Tegevust ei ole vaja – `import chloros_sdk` töötab pärast edukat paigaldamist teie Python keskkonnas. Kui arvutis puudub Python, jätab paigaldaja selle sammu vaikimisi vahele ning graafiline kasutajaliides ja CLI jätkavad töötamist.

#### Linux (.deb)

.deb-fail paigutab wheel-faili asukohta `/usr/lib/chloros/sdk/`. `postinst` kuvab täpse käsu – PEP 668 distributsioonid keelavad vaikimisi globaalsed pip-kirjutused, seega me ei installi seda automaatselt:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Air-gapped Jetsoni kasutuselevõtude puhul toimub see täielikult offline-režiimis – wheel on juba kettal.

#### Avalik PyPI

Ainult pip-i kasutavate hostide puhul (Chlorosi töölaua pakett pole installitud; kaugjuhtimisega-backend või ainult DAQ-põhised töövood):

```bash
pip install chloros-sdk
```

PyPI uuendatakse iga uue versiooni installeri koostamisel, seega vastab avaldatud wheel viimasele stabiilsele versioonile. Arendusversioonid (nt `1.1.4.dev1`) tarnitakse ainult komplektis oleva installiprogrammi wheel-faili kaudu.

#### Kontrollige

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Vajalik on Chloros+ tellimus.** Kõik SDK-kutsed nõuavad aktiivset Chloros+ sisselogimist. Käivita `chloros-cli login user@example.com 'YourPassword'` üks kord iga masina kohta; kasutajatunnused salvestatakse vahemällu `~/.chloros/`.

### Kas mul on vaja töölaua paketti?

Enamiku töövoogude jaoks **ei** piisa ainult pip-paketist. Siin on loetletud, mida iga SDK pind vajab:

| SDK pind | Kas vajab töölaua paketti? | Miks |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Jah** | Käivitab automaatselt tagapõhja binaarfaili aadressil `/usr/lib/chloros/chloros-backend` (Linux) või `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Jah**(kohalik)**/ Ei**(kaugjuhtimine) | Puhtad HTTP kliendid tagapõhja kaudu. Kohalik tagapõhi → vajalik töölaua pakett. Kaugjuhtimise tagapõhi → `backend_url=`**tunneli kaudu** (vt kaugjuhtimise tagapõhi režiim — kaasasolevad tagapõhjad seovad ainult loopbacki). |
| `ChlorosProject` / `open_project` | **Jah** | Juhib salvestatud projekte tagapõhja kaudu. |
| Otsesed LATTICE-klassid (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Jah** | Vajab Arena SDK natiivset käitamiskeskkonda, mis on kaasas töölaua paketiga. Muidu on `CAMERA_AVAILABLE` importimisel sama mis `False`. |
| Otsesed DAQ-klassid (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Ei** | Puhas „Python” pyserial/bleak/zeroconf kaudu. Ainult pip-keskkond suudab DAQ-seadmeid läbi viia algusest lõpuni. |

### Kaug-tagapõhja režiim (ainult pip-põhine host, tunneli kaudu)

> **Kaasasolev backend ei ole LAN-võrgus kättesaadav.** Tootmisversioonid
> seovad end ainult loopback-iga (mõlemad loopback-perekonnad) ja keelavad kategooriliselt
> ainsa mitte-loopback-režiimi (`CHLOROS_CLOUD_MODE`), seega
> `backend_url="http://<lan-ip>:5000"` **ei saa töötada koos installitud
> Chloros**-ga — see skeem töötas alati ainult source/dev
> backendiga. Teisel masinal asuva backendi juhtimiseks suuna selle loopback-
> port ise edasi ja suuna SDK tunneli poole:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Peata / CI / robootika-hostid võivad hoida ühte masinat täieliku töölaua installatsiooniga kui „Chloros-serverit” ja kõikjal mujal `pip install chloros-sdk` — kuid nende vaheline andmeedastus toimub eespool nimetatud kasutaja poolt korraldatud tunneli kaudu, mitte otsese LAN-URLi kaudu.

> **Teadaolev piirang — `ChlorosLocal` ei toeta ainult pip-i.** `ChlorosLocal(backend_url=BACKEND)` tuvastab praegu oma konstruktoris kohaliku tagapõhja binaarfaili *enne* URL-i kontrollimist ja tekitab vea `ChlorosBackendError` („Chloros-tagapõhja ei leitud…“), kui ühtegi töölaua paketti pole installitud — isegi kui kaugtagapõhi on kättesaadav. Ainult eespool nimetatud smart-connect-liides (`connect_camera` / `connect_array` / `connect_daq_sensor`, pluss `analyze_array_network` ja abiprogrammid `list_*` / `discover_*`) töötab ainult pip-hostilt.

### Ainult DAQ-i kasutav töövoog (ainult pip-i kasutav host)

Kui vajate ainult DAQ-andureid ega kasuta LATTICE-kaameraid ega pilditöötlust, on pip-pakett iseseisev:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Otse riistvaraga DAQ-tööks ei ole vaja tagapõhja, .deb-faili ega sisselogimist aadressil Chloros+.

---

## Kiirstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
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

## Ülemise taseme „API” indeks

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
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Pilditöötlus — `ChlorosLocal`

Peamine töötlusklass. Käivitab esmakordsel kasutamisel tagapõhja, loob ja konfigureerib projekte, jälgib protsessi kulgu ning tagastab töötlemise järel kokkuvõtted.

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
| `import_images(folder_path, recursive=False)` | RAW/TIF/JPG/DNG-piltide **ja `.daq` valgusanduri salvestusi**. Tagastab `count` (pildid) ja `scan_count` (salvestused). Annab hoiatuse ainult juhul, kui kaustas pole kumbagi. |
| `export_light_sensor(daq=True, csv=True)` | Kirjutab kalibreeritud `.daq` + `.csv` iga projekti valgusanduri salvestuse kohta faili `<project>/Light Sensor/`. Vaata [Valgusanduri salvestused](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Seadista töötlemise parameetrid. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Käivita töötlemisahel. Tagastab `{"status": "complete", "async": False}` ning lisaks `summary` võtme, kui tagapõhi seda pakub — vt [Käivitusejärgne kokkuvõte ja näpunäited](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Kontrolli tagapõhja seisundit. |
| `logout()` | Tühjenda vahemällu salvestatud autentimisandmed. |
| `shutdown_backend()` | Lõpeta tagapõhi (kui see on käivitatud käsuga `SDK`). |
| `discover_cameras()` | Leia LATTICE-kaamerad **selle instantsi tagapõhja kaudu** (`/api/camera/discover`). Tagastab sõnastikute nimekirja (`serial`, `model`, `ip`, …) — sama kujuga, mida näevad GUI ja CLI. Tühi nimekiri, kui ühtegi ei leitud või backend ei ole kättesaadav. |
| `camera_capture(output_dir, format="tiff", **settings)` | Salvestab ühe kaadri**backendi kaudu**(käivitatakse automaatselt selle käepideme poolt), nii et see saab sama ettevalmistuse kui GUI ja CLI (vaikimisi 12-bitine, mäluploki taaskasutus, sisseehitatud kalibreerimismetadandid). Määrake sihtmärk kindlaks käsuga `serial=` või `device_index=`; edastage `exposure`/`gain`/`pixel_format`/`preset` edasi kui `**settings`. Tagastab vana metaandmete sõnastiku (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Saagikuse ülekannekoondatud kaamerast saadud eelvaatekaadrid – kerge MJPEG-klient tagapõhja `/api/camera/<serial>/stream-annotated` marsruudi kaudu (zebra / võre / ristjoon / histogramm / tippväärtus / serveri poolt joonistatud täpppoolel). `decode=True` annab BGR-massiive; `False` annab töötlemata JPEG baiti. Samuti kättesaadav projektipõhiselt kui `ChlorosProject.stream(overlays=True)`. |

Kasutamine kontekstihaldurina tagatud puhastamiseks:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Valgusanduri salvestused — kalibreeritud `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E on võimalik salvestada **ilma** selle kalibreerimispaketita. See tähendab,
teevad avalikud [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
salvestid (`record_daq.py`) vaikimisi: nad kirjutavad salvestusse anduri toorandmed ja märgistavad
faili nii, et Chloros laadib selle anduritehase kalibreerimise **seerianumbri järgi** – kõigepealt kohalikust vahemälust
ja seejärel MAPIR pilvest – ning rakendab seda importimisel.

Chloros kirjutab tulemuse tagasi kahe tootena salvestuse kohta, nimetusega
`<project>/Light Sensor/`:

| Toode | Mis see on |
| --- | --- |
| `<name>_calibrated.daq` | Ümbertöödeldav arhiiv — sama skeem nagu reaalajas salvestusel, nüüd deklareerides selle loonud paketi. Selle uuesti importimine **ei** kalibreeri seda teist korda. |
| `<name>_calibrated.csv` | Spektraalne kiirgustihedus ühikutes W/m²/nm anduri enda lainepikkuste võrgustikus, üks rida iga mõõtmistulemuse kohta, lisaks fotomeetrilised veerud (koguvõimsus, fotopiline/skotopiline luks, PPFD ja selle sinine/roheline/punane jaotus, tipplainepikkus). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Ainult pakettideta andurid (DAQ-A).** Anduri toores spektraalne loend – *mitte* kiirgustihedus. Vaata allpool. |

`process()` teostab seda eksporti ühe oma etappidest. See **ei** vaja pilte:
eraldi lennutatud valgussensor on esmaklassiline töövoog ning sellisel projektil pole
konstruktsiooni tõttu ühtegi pilti.

**DAQ-A salvestised eksporditakse toorloenditena.** DAQ-A-seeria on vanem kui seerianumbri-põhine
kimpude süsteem ja sellel pole kimpu, mida alla laadida — selle asemel kalibreeritakse seda välitingimustes
peegeldusmäära sihtmärgi abil, millepärast pole seda kunagi vaja olnud. Need salvestused eksporditakse
tüübiga `_raw`, mitte `_calibrated`: teistsugune failinimi, mitte lipuk
faili sees, sest andmed peavad säilima ka siis, kui need saadetakse edasi e-kirjas pelgalt failinimena. Pealkiri
`.csv` näitab `raw spectral sensor counts (NOT irradiance)` ja hoiatab, et
väärtused on võrreldavad **selle** faili piires — just seda, milleks sihtmärgipõhine kalibreerimine
neid kasutab — ja mitte andurite vahel. Võimsusest sõltuvad fotomeetrilised veerud (koguvõimsus,
fotopiline/skotopiline luks, PPFD) annavad tulemuseks **NULL**, mitte loendustest integreeritud väärtused.

DAQ-U / DAQ-M / DAQ-E, mille paketti lihtsalt ei õnnestunud alla laadida, **jätatakse ikkagi vahele**,
mitte toorandmetena salvestatud: seal on pakett olemas ja „taasühendamine ja uuesti töötlemine” on tõeline soovitus.

Vana versiooni **v1.01 / v1.02** salvestused (neid kirjutab DAQ-A-SD) ei sisalda lugemipõhist ajastust,
vaid ainult faili kirjutamise aja. Pildi↔allapoole suunatud andmete sobitaja lükkab need endiselt tagasi –
kaadri sobitamine kirjutamisajaga oleks märkamatult vale –, kuid eksportija loeb neid, ja
fail CSV väljastab `clock=daq_created_on`, nii et tulemus näitab, millisel kellal see põhineb.

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
kalibreerimisandmed puuduvad failist), kajastatakse koodiga `skipped` **koos põhjendusega**. Seda ei
kirjutata kunagi välja „kalibreeritud“ failina, mis sisaldab töötlemata loendusi – ühenduge internetiga ja
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

### Käsitlusejärgne kokkuvõte ja näpunäited

Lõpetamisel laadib `process()` alla `GET /api/processing-summary` ja lisab selle sisu failina `result["summary"]`. Laadimine toimub parima püüdluse põhimõttel ega takista kunagi edukat tagasipöördumist – kui kokkuvõte pole kättesaadav, kasutab `process()` tagavarana lihtsat `{"status": "complete", "async": False}`-vormi. Iga kirje `summary["hints"]`-is – täielikud laused koos soovitatud parandusmeetmetega, nt miks käivitus ei andnud tulemust — edastatakse samuti uuesti kujul Python `UserWarning`, seega on nullväljundiga käivitused isediagnostiseerivad isegi siis, kui sõnastikku kunagi ei vaadata:

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
| `models` | Jooksu kaamerarühmad. |
| `images_in_groups` | Nende rühmade allikapildid. |
| `targets_found` | Avastatud peegeldusmäärade sihtmärgid. |
| `images_calibrated` | Jooksu käigus kalibreeritud pildid. |
| `exported_files` | **Käivituse käigus loodud pildifailid.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Valgusandurite salvestused, mida on tahtlikult eraldi loendatud — need pärinevad teisest etapist ja eksisteerivad ka selliste käivituste puhul, kus pilte üldse ei ole, seega nende kaasamine muudaks ainult andmekogumisega käivituse selliseks, nagu oleks see pilte eksportinud. |

Nende kõrval: `summary["output_dirs"]` (iga kataloog, kuhu kirjutati),
`summary["light_sensor_export"]`, `summary["stopped"]` (tõene, kui kasutaja katkestas
jooksu, nii et osalised loendid ei näita seda kui lõpetatud jooksu, mille toodang jäi väiksemaks) ning
`summary["groups"]` (jaotus rühmade kaupa).

`exported_files` salvestatakse torujuhtme poolt **kirjutamise ajal**, mitte skannitakse hiljem
projekti pildiobjektidest välja. Paralleel- ja GPU-strateegiad loovad oma pildiobjektid
(GPU-teede puhul töötajate alamprotsessides), seega vana skannimine näitas
`0 file(s) written` iga sellise käivituse kohta ja väljastas seejärel null-eksportide vihje – käivituste puhul,
kus kõik oli töötanud. Kui te loote skripti selle numbri põhjal, näitab nüüd korralik paralleelkäivitus
nullist erinevat arvu.

Vahelejätmiste aruanded näitavad põhjust, mille lugeja iga faili puhul tegelikult kindlaks tegi –
loetamatu skeem, puuduv kimp, kirjutusviga – **duplikaatidest puhastatuna**, nii et kakskümmend faili,
mida ühe põhjuse tõttu vahele jäeti, kuvatakse ühe põhjusena, mitte selle kahekümne kordusena.

> **`process()` ei tekita veateadet, kui käivitamine ei tooda pilte.** See on ainus koht, kus SDK ja
> CLI erinevad teadlikult: `chloros-cli process` käsitleb &quot;tooteid taotleti, kuid ühtegi ei
> kirjutatud” kui ebaõnnestumist ja lõpetab töötamise nullist erineva tulemusega, samas kui SDK lõpetab töötamise normaalselt ja teatab
> olukorrast `summary` / hints kaudu. Kui teie töötlemisahel peaks tühja käivituse korral peatuma, kontrollige seda
> ise üle – kontrollige `summary`-i (või loendage projektikausta all olevaid faile), selle asemel et tugineda
> erandite puudumisele. Tavapärased põhjused on sisendkaust, mida ei tunnistatud
> salvestusena, ja tooted, mis jäeti vahele, kuna need ei olnud kohaldatavad olemasolevate kaamerate jaoks (nt kiirgus RGB -ainult
> kaameratelt).

### Mugavusfunktsioonid

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
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

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Toetatud väärtused

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
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

#### Radiomeetriline väljund (LATTICE multispektraalne töötlemisahel)

`process` töötlemisahela LATTICE multispektraalne (M3C/M3M) eksporditase — `reflectance` (vaikimisi), `radiance`, `sensor-response` või `all` (iga pildi puhul kõik sobivad režiimid) — vastab projekti töötlemisseadistusele **„Radiomeetriline väljund”**. `configure()`-il on selle jaoks eraldi märksõna:

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

Täpsem pääsetee — projekti `"Radiometric output"` võtme kirjutamine läbi `custom_settings` — toimib endiselt, kuid pidage meeles, et see asendab kogu seadete ploki (vaadake allpool olevat hoiatust):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (vaikimisi) jagab kaamera kiirgust **ajamärgiga sobitatud DAQ-i allapoole suunatud kiirgusega**, mis tuletatakse automaatselt salvestatud `.daq` (DAQ-U/M/E)**või DAQ-M-i omast `.csv`**, mis on leitud piltide juurest; kõik kohapeal puuduvad kaamera- või DAQ-kalibreerimispakid**laetakse esmakordsel kasutamisel automaatselt AWS-ist**. CLIis kuvatakse see tootetüübi kaupa lülititena `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **asendab** kogu arvutatud seadete ploki (see jätab kavandatud viisil `configure()` muud märksõnad ja valideerimise vahele). Kui seda kasutate, lisage kõik teile olulised `Project Settings`-võtmed, nagu eespool toodud näites.

---

## Smart-Connect LATTICE-kaameratele

Püsivad tagapõhja-seansid reaalajas töötavale riistvarale. Kasutatakse samu lõpppunkte, mida kasutab ka graafiline kasutajaliides (GUI), seega on käitumine identne veebisaidil SDK / CLI / GUI.

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

#### `connect_camera()` allkiri

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

#### `CameraSession` Meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Loeb GenICam-sõlmi; tagastab `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Kirjutab sõlmed sõbraliku nime järgi (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Salvestab **ühe** kaadri. Tagastab ühe elemendiga loendi, mis sisaldab kaadri metaandmete sõnastikke. (Seeria- ja mitmekaadriline salvestamine on eemaldatud — kui vajate seeriat, kutsuge tsüklis välja `capture()`.) |
| `disconnect()` | Vabastab ressursi reservist. Ei tee midagi, kui oleme ühendunud juba avatud sessiooniga. |

`capture()` ekspordikontrollid (sama mudel nagu massiiv + graafiline kasutajaliides):

- `processing` / `levels` — `processing="all"` salvestab kõik sobivad eksporditüübid; `levels=["raw","radiance"]` salvestab ainult need (asendab `processing`). Jäta mõlemad välja, et kasutada tagapõhja vaikimisi seadeid.
- `force_daq=True` — salvestab määratud DAQ/DLS-näidu `.daq`-sidecarina isegi puhta toorandme korral, et kaadrit saaks hiljem ümber töödelda peegeldusvõime/indeksiks. Ei tee midagi, kui DAQ-seadet pole seotud.

### Sünkroniseeritud massiiv — `ArraySession` (Smart-Prep)

`connect_array` on **soovitatav alguspunkt** mitme kaameraga seadistuste puhul. See käivitab taustal täieliku GUI smart-prep-protsessi:

1. **Võrguanalüüs** (`/api/camera/array/recommend`) — leiab suurima kaadri suuruse, mis mahub sim-emit-tasemele ilma kaadrite kaotamiseta.
2. **Taseme automaatne valik** — `sim-capture-sim-emit`, kui ühendus seda toetab; vastasel juhul `sim-capture-ftd-stagger` või `slip-emit-and-capture`.
3. **Automaatne vähendamine**— vähendab vaikselt raami suurust / suurendab binningut, kui ühendus ei suuda taotletud resolutsiooni toetada.**See turvavõrk ei hõlma koondatud ülemärkimist**: ühendusele liiga palju kaameraid ei saa lahendada raamide vähendamisega — vt [Ülemärkimine](#over-subscription-kaamera-kohta-alampiir).
4. **PTP on vaikimisi sisse lülitatud**vaikimisi — kaameratevahelised ajamärgid langevad ühele ühisele kellale**~1 ms**täpsusega. Samaaegne säriaeg tuleneb M8 riistvaratriggerist (**&lt; 100 µs** moodulitevaheliselt), mitte PTP-st: PTP joondab *ajamärke*, mitte säriaegu.
5. **Kaamera-põhine pikseliformaadi automaatvalik** — RGB-kaamerad → `BayerRG8`, multispektraalsed → `BayerRG12`.
6. **AE algväärtustamine** — salvestab iga kaamera hetke AE-seisundi, et ühendamine ei nulliks eksponeerimisaega töötamise käigus.
7. **GPIO-trigeri konfiguratsioon** — `connect_array` aktiveerib kõik kaamerad (`TriggerMode=On`, `TriggerSource=Line2`), nii et master-seadme impulss juhib slave-seadmeid M8-kaabli kaudu. See on ainult massiivi jaoks mõeldud samm: üksik kaamera, mis on avatud käskuga `LatticeCamera`, töötab selle asemel vabalt.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` allkiri

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier` väärtused:
- `"sim-capture-sim-emit"` — tõeline samaaegsus (kõik kaamerad käivituvad sama taktsignaali serval).
- `"sim-capture-ftd-stagger"` — paindlik-domeeni nihutus (kamid saadavad andmeid veidi nihutatud aegadel, nii et paketid järjestuvad võrgus).
- `"slip-emit-and-capture"` — järjestikune salvestamine kammi kohta (ajaliselt sünkroniseerimata; ainus valik, kui ükski raami suurus ei sobi sünkroniseerimisega).

`wire_ceiling_mbps` asendab **host&#x27;i püsiva andmeside eelarve** MB/s-is — see on ainus
arv, millest kogu massiivi eraldus sõltub. Jätke see väärtuseks `None`, et kasutada automaatselt tuvastatud
väärtust. Vähendage seda, kui massiiv teatab GVSP-riknenud raamidest: automaatne väärtus tuletatakse
võrgukaardi poolt teatatud ühenduskiirusest, mis ülehinnab
USB-adaptereid, kitsaid PCIe-kanaleid jaUSB-adaptereid, kitsaid PCIe-kanaleid ja
hõivatud jagatud struktuure – ning see ülehindamine avaldub pigem vigastena raamidena kui
nähtavalt aeglase ühendusena. Väärtus salvestatakse projekti massiivi salvestusplokki, seega
taasavamisel või hilisema `connect_array` käivitamisel taastatakse see nagu iga teine massiivi seade.
Vaata [Massiivi tervis](#array-health--which-subsystem-is-losing-frames).

#### Ülemäärane kasutamine (kaamera kohta kehtiv alampiir)

Sim-emit-tempo jaotab igale kaamerale osa kokkupõrkeohutust andmeside eelarvest, mille alampiiriks on **8 MB/s kaamera kohta**(`per_cam_floor_bps`). Kui `N × floor` ületab kokkupõrkeohutu ülempiiri, siis massiiv**ületab võrgu läbilaskevõime**— rikke tüüp on GVSP-pakettide kadu, mitte madalam kaadrisagedus — ning kaadri suuruse muutmine ei aita:**binning ja ROI vähendavad baitide arvu kaadri kohta, mitte sekundi kohta jaotatud baitide arvu**, mida koondkontroll võrdleb. Praktilised täisresolutsiooni ülempiirid 1 GbE hostil:**6 kaamerat @ 1500 MTU, 9 koos jumbo-kaadritega** (analüüsivastuses `max_cams_collision_safe` näitab teie ühenduse ülempiiri). Lahendused: vähem kaameraid, jumbo-kaadrid algusest lõpuni või kiirem võrgukaart.

- Vastused `analyze_array_network()` ja `/api/camera/array/connect` sisaldavad `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` ja `per_cam_floor_bps`. Kui `oversubscribed` on tõene, siis projektsioon **nullib fps-väljad** (`achievable_fps_max` / `fps_bright` / `fps_dark`) nullib, selle asemel et näidata eksitavat aeglast, kuid-töötava kiiruse kohta.
- `POST /api/camera/array/connect` aktsepteerib `pin_resolution` keha parameetrit (**ainult HTTP – mitte SDK-i kwarg**; `connect_array` seda ei avalda). Kinnitamine eemaldab binningu järkjärgulise vähendamise turvavõrgu, mistõttu ülemäära koormatud ühendus, mille puhul on määratud `pin_resolution`,**lükatakse kindlalt tagasi** veateatega, milles nimetatakse kõik võimalikud lahendused. Ilma kinnitamiseta jätkub ühenduse loomine järkjärgulise vähendamisega, kuid hoiatatakse, et vähendamine ei suuda kogumahtu tühjendada.
- Testkeskkonna pääsetee: seadista tagapõhja keskkonnas `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`, et alandada keeldumist valjuks hoiatuseks — sa ühendud niikuinii ja aktsepteerid pakettide kadu.

#### Massiivi tervis — milline alamsüsteem kaotab raame

`GET /api/camera/array/<array_id>/capability` kannab aktiivset `health`-blokki
ühendatud massiivis, mida hinnatakse uuesti jooksvas **10-sekundilise** akna jooksul. See jagab raamikaotuse
kaheks põhjuseks, mis vajavad vastupidiseid parandusi, selle asemel, et näidata üht „ebatäieliku” määra, mis
mõlemat ei nimetagi:

| Väli | Mida see tähendab | Milline alamsüsteem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (seerianumbri kaupa) | Raam **jõudis kohale, kuid oli struktuuriliselt vigane**— GVSP-pakettide kadu. |**Võrk**: kaabli läbilaskevõime, tempotamine, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (iga seerianumbri kohta) | Raam **ei jõudnud üldse kohale**— kaamera ei käivitatud või sellest ei väljunud midagi. |**Trigger / sünkroniseerimine**: M8-kaabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Iga kaameramäär iga kaamera kohta. | — |
| `per_cam_rate_pct` | Kaamera kohta arvutatud kombineeritud puudulikkuse määr (mõlemad põhjused kokku). | — |
| `stable_for_seconds` | Kui kaua on iga kaamera püsinud alla 0,01 %. | — |

Koos `health`-iga näitab sama kirje ka kogu eraldise mahu:

| Väli | Mida see tähendab |
| --- | --- |
| `wire_ceiling_mbps` | Hosti kehtiv püsiv andmeside eelarve, MB/s. |
| `wire_ceiling_source` | Selle arvu allikas, sõnadega — nt `USB-capped 200 MB/s (was theoretical 1062; …)` või `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, kui `wire_ceiling_mbps=` selle määras. |
| `nic_is_usb` | `true` USB-Ethernet-adapteri jaoks. |

Selle lõpppunkti jaoks ei ole SDK-ümbrist – loe seda otse:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Lugemine:** nullist erinev `gvsp_corrupt_rate_pct` koos väärtusega 0 `never_arrived_rate_pct` tähendab, et
trigger ja kaabli sünkroniseerimine on täiuslikud ning 100 % kaotusest on võrguteel — vähendage
`wire_ceiling_mbps` väärtust ja ühendage uuesti. Vastupidine muster viitab pigem sünkroniseerimiskaablile või
triggeriliinile.

> **`target_fps` ei ole vigaste raamide põhjuseks.** GevSCPD-sagedus kirjutatakse sisse üks kord
> ühendamisel, seega triggergiiruse alandamine muudab töötsüklit, mitte aga
> samaaegse saatmise purske sagedust. Mõõdetud 5-kordne nõudluse vähendamine ei andnud mingit parandust, samas kui
> kaabli ülempiiri langetamine 240-lt 200 MB/s-ni viis sama seadme vigaste raamide määra 10,4 %-lt
> 0,00 %-ni.

> **TRI032S-i püsivara puhul ei ole voo keskel toimuv automaatne vähendamine saadaval.** Töötav massiiv ei saa
> seda ise parandada; katkesta ühendus ja ühenda uuesti, et ühendusaja valija planeeriks uuesti vastavalt
> uuele ülempiirile.

**USB-Etherneti adapteri maksimaalne kiirus on 200 MB/s** sõltumata selle
nimikilbil märgitud andmetest: tõhusustabel, mis teisendab ühenduse kiiruse püsivaks näitajaks, on
põhineb PCIe-l, ning USB-võrgukaart edastab oma Etherneti ühenduse kiiruse, olles samal ajal piiratud
USB-siini ja selle draiveri poolt. Piirang on absoluutne, mitte suhteline – USB 1 GbE-adapter
saab ~80 MB/s ja see ei muutu.

#### `ArraySession` meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `status(timeout=10.0)` | Reaalajas `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Üks sünkroniseeritud salvestusgrupp. Tagastab `CaptureResult` (kaadrite sõnastikute nimekirja + `.skipped`). Ekspordikontrollid allpool. |
| `capture(..., smart=True)` | **Nutikas salvestus** — ootab, kuni AE on kõigil kaameratel stabiliseerunud, ja käivitab seejärel salvestuse. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Kiireim salvestus: ainult toorandmed + määratud DAQ-näit (+ vaba kombineeritud indeks). Peegeldab kasutajaliidese nuppu „Kiireim salvestamine“. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Ühekordne / Pidev / Intervall ühes piiratud tsüklis. Tagastab `list[CaptureResult]`.**Nõuab `count` ja/või `duration_s`**, et see lõpetataks (SDK&#x27;il puudub Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Alusta reaalajas kombineeritud indeksivaate salvestamist videoks/GIF-iks → `RecorderHandle`. Üks komposiitsalvesti iga massiivi kohta. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Alusta kõrgekaadrisagedusega toores-Bayer-pildiseeria → `RecorderHandle`. Töötle offline-režiimis uuesti funktsiooniga `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Töötle salvestatud toores pildiseeria offline-režiimis kalibreeritud videoks(deks). Blokeerib kuni lõpetamiseni (`wait=True`) ja tagastab `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Küsib offline-koostamistöö staatust: `{running, result, error, burst_dir}`. |
| `disconnect()` | Vabastab kogu massiivi. |

`capture()` ekspordikontrollid (sama lõpppunkt, mida kasutavad GUI/CLI):

- `processing` / `levels` — `processing="all"` (või `levels=["raw","radiance",…]`) salvestab iga kaamera kohta kõik kohaldatavad eksporditüübid; üks `processing` väärtus salvestab vaid selle taseme.
- `aligned=True` — moonutab iga liikme mitte-toorekspordi massiivi[joondamisprofiili](#array-alignment) järgi (koosregistreeritud); töötlemata andmed jäävad joondamata, kuid sisaldavad teisendust metaandmetes. Kui massiivil puudub profiil, kasutatakse tagavarana joondamata andmeid (koos hoiatusega tulemuse `alignment`-is).
- `render_index=False` — jätab vahelekaamera taimestikuindeksi pealekandmist; vaikimisi renderdatakse see seal, kus see on konfigureeritud.
- `force_daq=True` — salvestab määratud DAQ/DLS-näidu `.daq`-sidecarina isegi siis, kui ükski valitud tase seda ei vaja.

**TIFF-pakkimine (ainult HTTP-nupp):**`ArraySession.capture()` ei saada `compression`-võtit, seega kehtib tagapõhja vaikimisi seade — `POST /api/camera/array/capture` loeb `compression` body parameetrit, vaikimisi `"deflate"` (kaotuseta zlib L1 + horisontaalne ennustaja, ~4,1 MB täisresolutsioonilise kaadri kohta). `"none"` kirjutab tihendamata (~6,3 MB/kaader) ja**~5× kiirema kirjutuskiirusega** — mõlemad on kadudeta ja loetakse importimisel identselt. SDK ei paku selle jaoks ühtegi kwarg&#x27;i; pääseteeks on `chloros-cli lattice array-capture --compression none` või toores HTTP. DEFLATE hoiab samuti Python GIL-i, seega kompresseeritud kirjutamised ei toimu paralleelselt kaamera-põhiste kirjutajate niitide vahel — püsiv 8-kaameraga täisresolutsiooniga salvestamine anduri kiirusel nõuab `compression: "none"`-i. Täpsemalt: [CLI Viide → array-capture](cli-reference.md).**Liikme-põhised ekspordi ümberkirjutused (ainult HTTP):**sama lõpppunkt aktsepteerib ka `exclude_serials` (nimekiri — salvestatud kogumist liikmete eemaldamine; massiiv käivitub endiselt ühe sünkroniseeritud rühmana ja väljajäetud liikmed tagastatakse `excluded`-is), `serial_levels` (`{serial: [level tokens]}` kaamera-põhised ümberkirjutused) ja `serial_index` (`{serial: bool}` kaamera-põhised indeks-ülekatte ümberkirjutused). Need on GUI-pariteediga põhiparameetrid ja**SDKi kwargid veel mitte**; kaartidest puuduvad liikmed kasutavad tagavarana massiiviüleseid `levels` / `render_index`.

##### Vahelejäänud kaamerate kontrollimine — `CaptureResult.skipped`

`ArraySession.capture()` tagastab `CaptureResult`, mis on `list` alamklass: iteratsiooni tegemisel, indekseerimisel `len()` — kõik olemasolevad mustrid toimivad endiselt. Uus kood saab kontrollida atribuuti `.skipped`, et näha, millised kaadrid jäeti välja ja miks. Kõige tavalisem juhtum on RGB kaadrid segafiltreid sisaldavas massiivis, kui küsitakse `processing="radiance"` või `"reflectance"` — lairibaanduri puhul on Bayeri-põhine kiirgusvõimsus mõttetu, seega jätab tagapõhi need kaamerad vahele, selle asemel et toota mõttetut tulemust.

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

Põhjuse märgid järgivad mustrit `<level>-not-applicable-to-rgb-cam` (üks kanne iga vahelejäetud taseme kohta, millest igaüks sisaldab `level`). Peegeldusastmestspetsiifilised vahelejätmised on `reflectance-skipped-no-fresh-dls` (puudub uus allapoole suunatud mõõtmistulemus), `reflectance-skipped-bound-daq-unavailable (…)` (seotud DAQ-seadmele ei saadud ühendust) ja `dls-uncalibrated-band-<nm>` — lainepikkus jääb enamasti väljapoole DAQ-i valgusanduri radiomeetriliselt kalibreeritud vahemikku (~374–974 nm), mistõttu DAQ-põhine absoluutne peegeldusvõime jagamine lükatakse tagasi ja kaader langetatakse selgelt anduri vastuse tasemele. Müügil olevatest toodetest põhjustab seda ainult F988; selle kaamera toetatud töövoog on peegelduspaneli töövoog.

`processing` tasemed:

| Tase | Väljund |
| --- | --- |
| `"raw"` | Ühekanaliline Bayer (mustvalged kaamerad: ühekanaliline) otse andurist. |
| `"debayered"` *(vaikimisi SDK)* | 3-kanaliline BGR bilineaarse demosaikimise kaudu (mustvalged kaamerad: 1-kanaliline halltoon). |
| `"radiance"` | float32 W/m²/sr/nm täieliku radiomeetrilise ahela kaudu. Ainult multispektraalne — RGB kaamerad on vahele jäetud. |
| `"reflectance"` | uint16 0..32768 (Pix4D-valmis); nõuab reaalajas andmekogumise (DAQ) sidumist absoluutse viite saamiseks. Ainult multispektraalne. |
| `"display"` | Täielik ahel, mis vastab GUI-eelvaatele (CCM + WB + gamma vastavalt kaamera profiilile). |
| `"all"` | **Iga kaamera kohta üks fail iga kohaldatava taseme kohta** (vastab kasutajaliidese vaikimisi seadistusele „Capture All“ / „CLI“). Tagastatud `CaptureResult` sisaldab seejärel ühte kaadri sõnastikku iga `(cam, level)` kohta, kusjuures iga sõnastikus on tasand; kohaldamata tasandid ilmuvad `.skipped`-is. Iga peegeldusraami jaoks kasutatud DAQ-näit salvestatakse `.daq`-i lisafailina. |

> **Märkus — vaikimisi väärtus erineb failist „CLI“.** `ArraySession.capture()` vaikimisi väärtus on `processing="debayered"`; käsu `chloros-cli lattice array-capture` vaikimisi väärtus on `processing="all"`. Edastage `processing="all"` selgesõnaliselt käsust „SDK”, et peegeldada CLI /GUI mitmetasandilist salvestamist.

### Salvestusrežiimid ja salvestid

Massiivi pind peegeldab GUI salvestuspaneeli: ühekordne / pidev / intervall / kiireim katikurežiim, lisaks kaks salvestajat (reaalajas komposiitvideo ja toores pildiseeria → offline-töötlus).

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

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**on „SDK” pidev/intervall-tsükkel. Kuna skriptis puudub `Ctrl+C`, millega seda katkestada,**peate** edastama `count` võivõi `duration_s` (see peatub, kui kumbki neist on saavutatud). `interval_s` mõõdetakse iga läbiviimise algusest (vastavalt graafilisele kasutajaliidesele). Ülejäänud kwargid edastatakse otse `capture()`-ile.
- **`record`** on *jälgimisklassi*: see salvestab reaalajas kuvatavat kombineeritud indeksi komposiiti, seega peab kombineeritud voog olema avatud, et kaadrid jõuaksid kohale. Üks komposiitsalvesti iga massiivi kohta (tekitab vea, kui üks juba töötab).
- **`burst` → `build_video`** on *analüüsitasemel*: `burst` kirjutab toorkaadrid + kaadri-põhise manifesti + ühe `.daq` iga eraldi DLS-lugemise kohta `<output>/bursts/<base>/` all salvestuslõigu täiskiirusel (ilma ahelata, ilma exiftoolita, ilma reaalaja vaatamiseta). `build_video` sobitab iga kaadri aja järgi lähima `.daq`-iga ja käivitab uuesti importimisprotsessi kiirguse/peegelduse/indeksi ahela. `products` on nimekiri `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`-ist (vaikimisi: kombineeritud indeks). `burst().stop()` käivitab automaatselt ka parima võimaliku kombineeritud indeksi koostamise, mille tulemusena tagastatakse `build_job` lõpetamistulemusena.

#### `RecorderHandle`

Tagastatakse `ArraySession.record()` ja `ArraySession.burst()` poolt. Kasutage seda kontekstihaldurina, et peatada töö automaatselt ulatuse lõppemisel, või juhtige seda käsitsi.

| Liige | Kirjeldus |
| --- | --- |
| `job_id` | Tagapõhja töö ID (str). |
| `kind` | `"composite"` (alates `record`) või `"raw"` (alates `burst`). |
| `start_stats` | `start`-kõne poolt tagastatud sõnastik. |
| `result` | `None` käivitamise ajal; lõplik peatamistulemuse sõnastik pärast peatamist. |
| `stats(timeout=10.0)` | Töö reaalajas statistika (kirjutatud kaadrid, tegelik kaadrisagedus, kulunud aeg). |
| `stop(timeout=60.0)` | Peatab salvestaja; tagastab ja salvestab vahemällu lõpptulemuse. Idempotentne (teine kutse tagastab vahemällu salvestatud tulemuse). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Ühendamine juba ühendatud massiivile — `attach_array`

Kui massiiv on juba käivitatud (GUI avas selle või eelmine SDK-seanss kutsus välja `connect_array`), kasuta selle asemel `attach_array`, et saada sellele käepide, selle asemel et uuesti ühenduda. `connect_array` annab <sn><id>selles olukorras</id></sn> alati veateate „Kaamera  on<sn> juba massiivis<id>”, kuna POST-kõne `/array/connect` </id>massiivi<id> liikme-in-pooli POST-kõne ei ole idempotentne; `attach_array` loeb `/api/camera/array/list`-i ja teeb vasteid kas array_id või serials alusel.

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

Muster: SDK skriptid, mis töötavad koos-üürnikuga töölaua graafilise kasutajaliidesega peaksid esmalt proovima `attach_array` ja kui pooli pole veel ühtegi massiivi, siis kasutama `connect_array`.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Oluline — context-manageri sulgemine katkestab ühenduse.**`ArraySession.disconnect()` saadab alati POST-päringu `/array/disconnect`-ile; puudub „attached-not-owned” kaitsemehhanism, nagu on olemas `CameraSession` / `DAQSensorSession` puhul. Kui kasutate seda koos GUI-ga ja ei soovi massiivi ulatuse lõpetamisel maha lammutada,**ärge kasutage `with`-plokki** — hoidke käepide tavalises muutujas ja jätke vahele selgesõnaline `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Võrguanalüüsi abivahend

Kasulik enne massiivi avamist — prognoosib, kas teie kavandatud seaded sobivad:

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
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` on üks järgmistest: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` hulka (muidu `error`). `auto_capped_fps` tähendab, et taotletud resolutsioon sobib RX-ringiga ainult piiratud käivitussageduse juures — säilita resolutsioon ja edasta `target_fps=result["recommended"]["recommended_target_fps"]` `connect_array`-ile (vt [Näide 6](#6-võimekuse-kontroll-enne-4-kaamera-massiivi-ühendamist)).

**Kuidas projektsiooni lugeda** (sama mudel kui GUI-paneelis „Array Settings”):

- **Burst (`frame_bytes_total`) summeeritakse iga kaamera puhul vastavalt selle tegelikule pikseliformaadile.**Mono**M3M**kaamerad edastavad Mono12 (2 B/px) sõltumata sellest, millise `pixel_format` väärtuse edastad, seega on 4-kaameralise täisresolutsiooniga kaadri suurus kolme monokaameraga**~25 MB**, mitte ~12,6 MB, nagu eeldab-8-bitise eelduse puhul. Tagapõhi määrab iga kaamera formaadi kindlaks selle mudeli järgi.
- **Admittance (`burst_fits_nic_ring`) arvestab andmevoo tühjenemisega**, mitte kogu-burst-vs-ring: sim-emit sobib, kui host tühjendab RX-ringi kiiremini, kui kaamerad seda täidavad. 10G host + 1 GbE kaamerad**võtavad vastu** täisresolutsiooniga kaadreid isegi siis, kui burst ületab ringi mahtu; 1 GbE host blokeerib (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` on konservatiivne järjestikuse andmete väljavõtmise ülempiir** — `max(readout+emit, N×emit)`, kus iga kaamera väljastus on piiratud 1 GbE kaameralingiga, valgustamisest sõltumatu. Näiteks ~2,8 kaadrit sekundis 4-kaameralise täisresolutsioonilise 12-bitise massiivi puhul (vastab käitamisaja mõõdetud ~2,7–3,0-le). Täielik mudel: [CLI Viide → Massiivi kaadrisagedus ja purskemudel](cli-reference.md#array-fps--burst-model).
- **Ülekoormus (`oversubscribed: true`) tähendab, et N × kaamera kohta kehtiv alampiir ületab kokkupõrkeohutu ülempiiri** — kaadrisageduse väljad (`achievable_fps_max` / `fps_bright` / `fps_dark`) näitavad väärtust 0 ning automaatne vähendamine/binning ei suuda seda probleemi lahendada (need vähendavad baitide arvu kaadri kohta, mitte sekundi kohta edastatavate baitide arvu). Lahendused on kaamerate arvu vähendamine, jumbo-raamid või kiirem võrgukaart; `max_cams_collision_safe` näitab ülempiiri (6 täisresolutsiooniga kaamerat 1 GbE-l @ 1500 MTU, 9 jumbo-raamidega). Vastuses on ka `aggregate_demand_bps`, `collision_safe_ceiling_bps` ja `per_cam_floor_bps` (8 MB/s). Vaata [Ülemäärane kasutamine](#over-subscription-the-per-cam-floor).

### Avastamine ja loetlemine

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

LATTICE-massiivid käivitavad taustal pideva automaatse ekspositsiooni (AE) niipea, kui need on ühendatud, kuid äsja suunatud stseeni konvergeerumine võtab hetke aega. **Smart-Capture** on mugavuslahendus: see küsib iga kaamera säriaega, ootab, kuni massiiv on akna ulatuses stabiilne, ja käivitab seejärel pildistamise. See vastab graafilisele kasutajaliidesele: töölaua rakenduse „smart” pildistamisnupp kutsub välja sama tagapõhja lõpppunkti.

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

Kui kasutate `ChlorosProject`-i (järgmine osa), saate rohkem reguleerimisvõimalusi:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Smart-AE-poliitika on vaikimisi konservatiivne. Karmistage `exposure_tolerance_pct`-i nõudlike radiomeetriliste tööde jaoks; leevendage seda kiiresti muutuvate stseenide puhul, kus soovite lihtsalt „piisavalt lähedast“ tulemust.

---

## DAQ-andurite seansid

Püsiv tagapõhja-pool spektraalanduritele (DAQ-U USB-ühenduse kaudu, DAQ-M BLE-ühenduse kaudu, DAQ-E Etherneti kaudu). Peegeldab kaamera pinda: nutikas tuvastamine, pooli taaskasutamine, idempotentne ühendamine.

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

Prioriteet: Ethernet → BLE → USB. Edasta ükskõik milline selgesõnaline vihje, et määrata kindlaks andmeedastusviis.

### Kinnitatud andmeedastusviis

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession` meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `status(timeout=10.0)` | Puli kirje kokkuvõte (voogedastus-/salvestusolek, lainepikkuse vahemik, kalibreerimise SHA, integratsiooniaeg, frame_avg, AE-olek). |
| `latest(n=1, timeout=10.0)` | Tagastab kuni N viimast-hiljutist spektriraami. |
| `stream_start()` / `stream_stop()` | Jätka / peata voogedastus (käepide jääb avatuks). |
| `record_start(output_dir=None, device_name=None)` | Alusta .daq-faili salvestamine. Tagastab faili asukoha. Keeldub DAQ-U/M-i puhul, millel puudub AWS-kalibreerimispakett (DAQ-E on erand). |
| `record_stop()` | Lõpetab salvestamise. Tagastab `{path, rows}`. |
| `disconnect()` | Vabastab . Ei tee midagi, kui käepidemed on kinnitatud, kuid ei kuulu kasutajale. |

> **Piirangukorrektsiooniprofiilid (`cap_id`) ei ole „SDK” nupp.** `connect_daq_sensor()` / `DAQSensorSession` ei avalda `cap_id` parameetrit ega `set_cap` meetodit. Valige laevastiku piirangukorrektsiooniprofiil kas CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) või backendi `/api/daq`-HTTPi marsruutide (`/api/daq/connect` ja `/api/daq/<id>/cap-id` aktsepteerivad `cap_id`) kaudu.

### Avastamine — ühendamiseks vajaliku aadressi leidmine

`discover_daq_sensors()` otsib USB-st / BLE-st / ETH-st andureid, mida *võiks* avada. See on DAQ-seadme vastand `discover_lattice_cameras()`-ile ja ainus viis saada **DAQ-M-i BLE MAC-aadress** – DAQ-E-l on hostinimi ja DAQ-U-l COM-port, kuid MAC-aadressi ei ole seadmele trükitud ega ole seda loetletud ka operatsioonisüsteemis.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

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
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` või `None`, kui skannimine ei suuda porti tuvastada (USB-seriaaliadaptereid ei ole ilma sondita võimalik eristada, seega kuvatakse tundmatud seadmed, mitte ei peeta neid varjatuks). |
| `extra` | Transpordiviisi kohta käivad üksikasjad (BLE reklaamitud nimi, USB tootja, DAQ-E IP/fw/…). Tühjad väärtused jäetakse välja. |

| Parameeter | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `transports` | kõik kolm | Skaneerimist piirav jada (või CSV-string). Tasub edastada, kui teate, mida soovite — BLE on aeglane osa. |
| `scan_timeout` | 5 | Transpordiviisi skannimisaken sekundites; tagapõhi piirab seda vahemikku 1–20. |
| `timeout` | 60,0 | kogu kõne jaoks kehtiv „HTTP“ ülempiir (nagu ka mujal „SDK“is). |
| `auto_start_backend` | `True` | Käivitab kohaliku backend&#x27;i, kui ükski ei ole käimas. Kaug-`backend_url` puhul ei käivita kunagi. |

> **Poolis juba avatud andurid ei kuvata.** Ühendatud BLE-perifeeria lõpetab reklaamimise ja avatud COM-porti ei saa tuvastada, seega loetleb avastamine seda, mis on *ühenduseks saadaval*. Vahetult pärast seadme ühendamist on oodata tühja tulemust — kasuta `list_daq_sensors()`-i selle jaoks, mis sul juba olemas on. Transpordid, mille skaneerimine ei saa töötada (puudub bleak / zeroconf) jäetakse vahele, mitte ei tekita viga, seega saab Bluetoothita seade ikkagi oma USB- ja ETH-vastused.

### Loetelu

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Ühiskasutus GUI-ga / CLI

Kui GUI-s on juba andur avatud, siis `connect_daq_sensor(port="COM3")`-i kutsumine aadressilt Python tagastab käepideme, mis on märgistatud `already_connected=True`-iga. Seansi `disconnect()` on seejärel tühitoiming, mistõttu teie skript SDK ei eemalda andurit GUI-st, kui programm lõpetatakse.

### Otsesed riistvaraklassid (ilma tagapõhja)

`daq_sdk` eksporditakse uuesti `chloros_sdk` kaudu, mistõttu saate andureid juhtida ka protsessi käigus algusest lõpuni ilma tagapõhja abita:

> **Kättesaadavus:**`daq_sdk` on kaasas „Chloros” töölaua installiga,**mitte** koos PyPI-paketiga — `pip install chloros-sdk` annab teile `lattice_sdk`, kuid jätab välja `chloros_sdk.DAQ_AVAILABLE == False`. Kontrollige seda lipikut enne nende klasside kasutamist; ainult pip-i kasutaval host-kettal kasutage selle asemel andurit läbi [`connect_daq_sensor()`](#daq-sensor-sessions), mis ei vaja kohalikke transpordiraamatukogusid.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Eelistage smart-connect-teed (`connect_daq_sensor`), kui soovite jagada omandiõigust graafilise kasutajaliidesega; kasutage otseseid klasse peata skriptide puhul, mis omavad andurit ainuõiguslikult.

---

## Projekti automatiseerimine — `ChlorosProject`

Salvestatud Chloros projekt on kaust, mis sisaldab `cameras.json` + `sensors.json` + `project.json`. `open_project` laadib manifesti ja `connect_all` viib kõik salvestatud seadmed võrku koos nende salvestatud seadistustega – sama riistvaraseisundiga, mida graafiline kasutajaliides tekitaks.

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

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Või kontekstihaldurina:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Leiab ja ühendab kõik salvestatud seadmed. Tagastab klassipõhise ühendusaruande. Kasutab töötavat tagapõhja, kui see kuulab `127.0.0.1:5000`-i; muidu lülitub märkamatult üle otsesele (tagapõhja-vaba) `lattice_sdk` seadmejuhtimisele — see ei käivita kunagi tagapõhja. |
| `disconnect_all()` | Lõpetab kõik ühendused. |
| `capture_all(output_dir=".")` | Üks kaader iga kaamerast + massiiv + spekter igast sensorist. |
| `stream(camera, overlays=False, fps=10.0)` | Generaator, mis toodab BGR-kaadreid nimetatud kaamerast (või massiivist). `overlays=False` on otsene `lattice_sdk` pildivõtutsükkel (massiivid annavad `{serial: frame}`-sõnastikke). `overlays=True` suunab läbi `ChlorosLocal.camera_stream()` → tagapõhja `/api/camera/<serial>/stream-annotated` MJPEG-voo, kusjuures kaamera salvestatud `ui.overlay`-plokk edastatakse päringuparameetritena. Nõuab tagapõhja režiimi ja **iseseisvat kaamerat**: otserežiimis kaamera tekitab veatunnuse `RuntimeError` (tagapõhi ei saa haarata kaamerat, mis kuulub sellele protsessile) ja massiiv tekitab veatunnuse `NotImplementedError` (koostab kaamerate kaupa komposiitpildi – edastab liikme nime järgi). Ühekordse käivituse ekvivalent: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Käivita joondamine igal hetkel ühendatud massiivil. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Käivita kalibreerimis- ja indekseerimispipeline projekti piltide puhul (hõlmab `ChlorosLocal.process`; need neli on **ainukesed** aktsepteeritavad kwargid — `indices=` jne. tekitavad veatunnuse `TypeError`; indeksite seadmine toimub `ChlorosLocal.configure()` kaudu). Loob viivitusega `ChlorosLocal()`, mis käivitab automaatselt tagapõhja. |

Atribuudid:
- `proj.cameras` — `Dict[str, CameraHandle]`, mille võtmesõnadeks on nimi JA seerianumber.
- `proj.arrays` — `Dict[str, ArrayHandle]`, mille võtmesõnadeks on nimi JA array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, mille võtmesõnadeks on nimi JA slot_id järgi.
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
tokeni ning ahel on kumulatiivne — iga tase käivitab kõik sellest kõrgemal olevad tasemed:

| Tase | Väljund | Märkused |
| --- | --- | --- |
| `raw` | 1-kanaliline Bayer, anduri omapärane | Demosaiki ei toimu. Selles tasemes ei ole ülekatteid saadaval. |
| `debayered` | 3-kanaliline BGR (**vaikimisi**) | Bilineaarne demosaik. Ainus tase, mis töötab ilma backend-režiimita. |
| `radiance` | float32, W/m²/sr/nm | Täielik radiomeetriline ahel: demosaik + 3×3 segunemise eemaldamine (multispektraalne) + DSNU + tasandus + NIST-skaala, kus ekspositsioon × võimendus on välja jagatud, nii et väärtused on absoluutsed. |
| `reflectance` | uint16, 32768 = 1,0 | Kiirgusjõud jagatud allapoole suunatud kiirgusvoo tihedusega (ρ = π·L/E). Vajab DLS/DAQ-näitu — vt allpool olevat märkust. |
| `display` | 8-bitine sRGB-laadne | GUI-ekvivalentne renderdamine: CCM + valgebalanss + gamma kaamera aktiivse värviprofiili kaudu. |

Kõik muud kui `debayered` nõuavad backend-režiimi; otserežiimis kaamera tekitab
`NotImplementedError`. `reflectance` vajab kasutatavat allapoole suunatud kiirguse näitu — kaadri lõpppunkt tõmbab
koondatud DAQ automaatselt kaamera DLS-pesasse, kuid kui DAQ-d pole seotud, keeldub ahel
peegeldusvõime väljundist ja märgib alandamise ausalt tagastatud metaandmetesse, selle asemel et vaikselt
tagastada madalama kvaliteediga tulemust.

> **Peegeldusvõime DN-skaala – ärge kodeerige seda kõvaks.** LATTICE peegeldusvõime kasutab `32768` = ρ 1,0 ja märgib
> XMP `Chloros:PixelScale=32768`; Survey3 peegeldusvõime kasutab `65535` = ρ 1,0 ja ei sisalda
> `Chloros:*`-märgiseid. Loe märgis välja ja jaga sellega. See on defineeritud uint16-domeenis, seega jääb see
> `32768` igas formaadis, mis teeb ümberskaalamist (16-bitine TIFF, 8-bitine PNG /JPG, 32-bitine protsent) — normaliseeri
> salvestatud andmetüüp esmalt tagasi uint16-ks (8-bitisest ×257, ×65535 float-ist). Ainus erand:
> 8-bitise allikaga salvestus, mis on kirjutatud 8-bitisena TIFF, *lõigatakse ära*, mitte skaleeritakse ümber, seega ei kirjeldata seda ühegi skaalaga
> — Chloros jätab sel juhul `PixelScale` ja MicaSense&#x27;i tupla täielikult välja. Käsitle puuduvat
> sildit LATTICE-peegeldusfailis kui „puudub kehtiv skaala”, mitte kui vaikimisi väärtust.

> **EXIF kantakse eksportimisel üle.** `process()` kopeerib allika pildistuse GPS-ploki
> **ja selle ExifIFD** igasse tootesse, seega sisaldavad ekspordid `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` ja `CameraSerialNumber` ning samuti
> georeferentsi. `FocalLength` on see, mille alusel Pix4D arvutab maapinna proovivahekauguse – ilma selleta
> langeb rekonstruktsioon äärmiselt valele mõõtkavale (ühel mõõdetud juhul muutus 411 m suurune ala
> 47,8 km suuruseks). Koopia ei ole tahtlikult `-all:all`: IFD0 struktuurimärgised rikuvad
> LATTICE väljundit, ning `ExifImageWidth`/`Height` on välja jäetud, kuna need kirjeldavad allika
> pildistamist, mitte eksporditud rasterit.

Salvestusetapi alamlipud (kehtivad radiomeetriliste tasemete kohta — `radiance`, `reflectance`, `display`):

| Lipu | Vaikimisi | Tähendus |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + tasandusvälja + 3x3 segunemise eemaldamine + NIST-i radiomeetriline skaala. |
| `apply_white_balance` | `True` | WB LUT. DLS-toetus, kui DAQ on kaameraga seotud. |
| `apply_index` | `False` | Taimestikuindeksi hindamine. |
| `index_expression` | `None` | Valemi ümberkirjutamine. Kui väärtus ei ole tühi, aktiveeritakse indeks automaatselt. |
| `annotated` | `False` | GUI-dekoratsioonide (zebra/võrgustik/piik) pealekandmine. Ei ole saadaval `raw` puhul. |

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

> **Tagasitulevatüüp on `CapturePathMap`, mitte `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` on `Dict[str, Union[str, List[str]]]`: ühetasandiline
> `processing` annab igale seerianumbrile ühe tee, samas kui mitmetasandiline (`"all"`, või
> selgesõnaline `levels` nimekiri) annab sellele selle
> kaamera jaoks salvestatud iga toote **järjestatud nimekirja**. Reaalajas kombineeritud komposiit, kui selline voogedastatakse, saabub täiendava
> `"combined"` võtme alla, mitte seerianumbri alla. Kood, mis eeldab `str`-i, annab vea
> nimekirja vormi puhul, ilma et tüübikontrollija sellele vastu oleks – märkus näitas `Dict[str, str]`
> veel mõnda aega pärast nimekirja vormi kasutuselevõttu, mistõttu see alias ka olemas on. Normaliseeri
> siis, kui soovid lamedat vormi:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Massiivi joondamine

`ArrayHandle` avab kogu joondamispinna. Profiilid on vaikimisi seansipõhised — püsivaks salvestamiseks kutsu selgesõnaliselt välja `export_alignment()`.

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

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Ühendamisel toimuv joondamine

`connect_all(align=...)` suudab ühendamisel iga massiivi automaatselt joondada:

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

Kui seda ei ole määratud, kasutatakse tagavaravõimalusena `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Otsene riistvara (tagapõhja-vaba)

Kui soovid täielikku sõltumatust tagapõhjast (CI, peata robotid, sisseehitatud süsteemid), impordige otse `lattice_sdk` ja `daq_sdk` — mõlemad eksporditakse uuesti `chloros_sdk` kaudu. Hoiatus seoses `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` on PyPI-paketis (kuid vajab Arena SDK käitamisaja olemasolu), samas kui `daq_sdk` on saadaval ainult töölaua versiooniga.

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

Neljast eelseadistusest kolm on **vabajooks**: kaamera eksponeerib pidevalt ja
`capture()` tagastab järgmise kaadri. `triggered` on erand — see paneb
kaamera valmis riistvaralise serva jaoks liinil 2, seega ei jäädvusta see midagi enne, kui see saabub.

| Eelseade | Trigger | Kasutada, kui |
| --- | --- | --- |
| `default` | vabajooks | üldine kasutus |
| `high_speed` | vabajooks | 8-bitine, piirang 60 kaadrit sekundis, lühike säriaeg |
| `high_quality` | vabajooks | 12-bitine, kaadrisageduse piiranguta — tavaline valik fotode jaoks |
| `triggered` | **valmis, liin 2** | kaamera on ühendatud M8 sünkroniseerimiskaabliga ja midagi muud käivitab selle |

Kui valite `triggered` (või seadistate ise `trigger_mode="On"`), ilma et midagi
juhtiks liini 2, siis iga `capture()` aegub — õigustatult, kuna te palusite
kaameral oodata. „SDK“ selgitab seda, kui see juhtub; vaata
[SC_ERR_TIMEOUT salvestamise ajal](#direct-hardware-backend-free).

> **Märkus — „GVSP probe” / `SC_ERR_TIMEOUT -1011`-sõnumid ühenduse loomisel ei ole vead.**&gt; Ühenduse loomisel üritab SDK läbirääkimisi**jumbo-raamide** (9000-baidilised GVSP-paketid) üle, et saavutada suuremat läbilaskevõimet. Otsese punktist-punkt-punkt-võrgukaardi ühendusel (nt link-local `169.254.x.x` aadress) ei suuda võrk tavaliselt jumbo-raame edastada, mistõttu see proov aegub ja logib selliseid ridu nagu:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> See on **ette nähtud tagavarameede**: SDK&#x27;i lülitub automaatselt tagasi standardsetele 1500-baitistele pakettidele ja kaamera jätkab ühenduse loomist tavapäraselt (järgnevad `[chunk-enable …]`-read on osa tavalisest ühendusjärjestusest). Salvestamine toimib endiselt.
>
> Võite selle proovi vahele jätta, kuid **see ei ole lihtsalt logide summutaja — see lülitab jumbo-raamid välja.** Kaamera vastab „Don&#x27;t-Fragment”-pingidele ainult kuni 1500 baiti, olenemata sellest, kui hea on teie võrk, seega ei suuda ping-test üksi kunagi jumbo-raame tuvastada; see proov on ainus, mis suudab seda teha. Lülitage see välja ja kaamera kasutab igas võrgus igavesti tavalisi 1500-baidiseid pakette:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> See tasub end ära ainult võrgus, mille kohta te *teate*, et see ei toeta jumbo-pakette, kus see säästab ühendusaega umbes ühe sekundi kaamera kohta. Kuna tegemist on tegeliku, mitte kosmeetilise muudatusega, märgib „SDK” seda nüüd, kui seda kasutate:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Jätke see rahule, kui teil pole selleks põhjust.** Kui see on sisse lülitatud, mõõdab iga ühendus uuesti teie tegelikku võrku: ühendage end jumbo-pakette toetava lülitiga ja järgmine ühendus võtab jumbo-pakette ise kasutusele, ilma et midagi tuleks seadistada ega süsteemi taaskäivitada.
>
> Kui te *soovite* jumbo-pakettide läbilaskevõimet, lülitage sisse jumbo-pakettide läbiviimine kogu võrgus (võrgukaardi MTU 9000 + lüliti, mis neid läbi laseb) või fikseeri see parameetriga `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, kui tead, et ühendus seda toetab — kuigi eelista käsukohast `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` püsiseadistuse asemel, kuna fikseeritud suurus jätab proovi vahele ja lõpetab kohanemise eelneva võrguga. **Iga** seadme teel peab toetama jumbo-pakette – sealhulgas kõik PoE-jagajad ja -injektorid, mis on tavaliselt põhjuseks, miks muidu jumbo-pakette toetav seadistus neid edastada ei suuda.

> **`SC_ERR_TIMEOUT -1011` ajal, mil `capture()` / `grab*()` on teine probleem – see on tõeline viga.**&gt; Eespool toodud märkus puudutab ainult**connect-time proovi* poolt logitud `-1011`-i* poolt logitud. Sama viga, mis tekib **salvestuse** käigus, tähendab, et kaamera ühendus on korras, kuid see ei saada ühtegi pilti:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Selle tunnuseks on kaamera, mille *juhtimiskanal* on korras — avastamine toimib, seaded ja `[chunk-enable …]`-kirjutused õnnestuvad kõik — samas kui *iga* kaadri puhul aegub ooteaeg.
>
> **Tavaliselt on põhjuseks see, et kaamera on seadistatud riistvaralisele päästikule.** `trigger_mode="On"` ja `trigger_source="Line2"` korral ei saada kaamera midagi enne, kui M8 sünkroniseerimiskaablile jõuab elektriline signaali serv. Kui seda liini, ootab iga kaadri salvestamine igavesti. Kaamera ei ole katki ja võrk töötab korralikult — see teeb täpselt seda, mida talle on kästud.
>
> `CameraSettings()` ning eelseadistused `default` / `high_speed` / `high_quality` töötavad vabajooksuna, ja käivitamine, mille ooteaeg lõpeb, kui seade on aktiveeritud selgitab olukorda ise, selle asemel et kuvada lihtsalt `-1011`. `PRESETS["triggered"]` aktiveerib Line2, nagu ette nähtud.
>
> Kaamera vabajooksu sundimiseks:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Kui ajapiirang lõpeb ikkagi `trigger_mode="Off"` korral, siis kaamera tõepoolest andmeid ei edasta — saatke meile logi ja `ip link show`.

#### Värviprofiilid (RGB reaalajas eelvaade) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` valib ekraani värviprofiili **reaalajas eelvaate** RGB-kaameratel (multispektraalsed kaamerad ignoreerivad seda seadet):

| Profiil | Tähendus |
| --- | --- |
| `raw` | Jäta radiomeetriline ahel täielikult vahele. |
| `linear` | DSNU + flat + WB, ilma CCM-ita, ilma gammata. |
| `natural` | Lineaarne + mõõdetud CCM + sRGB gamma, ainult odava viimistlusega (kroma silumine + heledate alade desaturatsioon) — realistlik vaikimisi seade. |
| `enhanced` | `natural` pluss täielik hub-parity viimistlus (värvijoonte eemaldamine, värvikus, CLAHE kohalik kontrast). Rikkam välimus umbes **kahekordse kaadri kohta kuluga**, seega madalam LIVE kaadrisagedus. |
| `custom_temp` | `natural`, kuid valgebalanss fikseeritud `custom_cct_k` Kelvini väärtusele (DLS ignoreeritakse; piiratud 2000–10 000 K-ni tagapõhja poolel). |

Profiil on **ainult reaalajas eelvaate jaoks** mõeldud kiiruse/välimuse reguleerimisnupp: salvestatud kaadrid saavad alati täieliku rikkaliku viimistluse, sõltumata valitud profiilist, seega `natural` valimine kaadri aja kokkuhoiuks ei alanda kettale salvestatava materjali kvaliteeti kettale salvestatakse. Tundmatu profiil suurendab `ValueError` väärtust; kui chloros-tagapõhi on kättesaadav, saadetakse muudatus ka sinna POST-päringuga, nii et järgmine eelvaatekaader peegeldab seda (direct-SDKi kasutajad, kellel tagapõhi puudub, saavad ikkagi seadete muudatuse).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M) kaamerad ja `Calibration`

Mono **M3M** kaamera (`M3M-<lens>-F<wavelength>`) on ühe sagedusribaga: üks halltoonide tasand, ilma Bayeri mosaiigita, ilma 3×3 spektraalse ristkõne maatriksita. `Calibration` tunnistab seda ja kuvab `is_mono` lipu. Peegeldus kehtib endiselt ribapõhise radiomeetrilise kaardina (lahutamine on identiteetmaatriks), kuid mitme-ribaline matemaatika ühe kaamera puhul annab pigem tulemuse kui toob välja mõttetusi:

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

Et luua taimestikuindeks monokromaatilisest riistvarast, kombineeri mitu erinevatel lainepikkustel töötavat M3M-kaamerat joondatud mitmeribalisse pildistakki (vt [Pildistakkide joondamine](#array-alignment)) ja arvutage indeks kogu selle pildistiku põhjal, mitte ühe kaamera põhjal.

DAQ otserežiim:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` aktsepteeritud võtmed**— täpselt `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; kasutuselt kõrvaldatud, asendatud `cap_id`-ga), `filter_model` (DAQ-M) ja `cap_id` (kõik DAQ-tüübid; `None`/`""`/`"none"` = paljas andur, ilma kate-korrektsioonita). Tundmatud võtmed**jätatakse märkamatult tähelepanuta** — nt `{"integration_time": 64}` ei tee midagi (peab olema `integration_time_ms`). Tagastab `{"applied": [...], "errors": {...}}` ja ei tekita kunagi viga.

`chloros_sdk` ekspordib uuesti ainult eespool kasutatud põhipinda. Täielik avalik `daq_sdk`-APIm (22 nime) lisab järgmised – importige need otse `daq_sdk`-ist:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Erandid

Püüa kinni baasklass, et käsitleda „kõiki Chloros vigu”:

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

> `ChlorosAuthenticationError` ja `ChlorosConfigurationError` eksporditakse ülejäänutega koos tipptasandil; neid saab ka importida`chloros_sdk.exceptions`-ist, nagu näidatud.

Hierarhia:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

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
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Reaalajas LATTICE-maatriks → peegeldusvõime + DAQ-viide

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

### 5. Peata otsene riistvarapõhine (ilma tagapõhjata) salvestamisskript

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
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Salvestusretsepti ekvivalent (puhas Python)

CLIi retsepti DSL-il on otsene Python ekvivalent:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

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

## Tagapõhja automaatne käivitamine

Smart-Connecti sisenemispunktid — `connect_camera`, `connect_array`, `connect_daq_sensor` ja `discover_lattice_cameras` — on kerged HTTP kliendid, mis eeldavad, et backend kuulab aadressil `127.0.0.1:5000` (smart-connect-liidese vaikimisi URL). Kui graafiline kasutajaliides või CLI juba töötab, on üks neist käivitatud. Tühja skripti puhul võib see aga puududa — seega käivitavad need funktsioonid **käivitavad komplektis oleva backend-binaarfaili automaatselt** (aknata, samamoodi nagu `ChlorosLocal`) enne esimest kutsumist ja ootavad seejärel kuni `backend_startup_timeout`, kuni see käivitub.

Reeglid:

- **Käivitatakse ainult kohalik URL.** Lubatud on `backend_url`, mis viitab `localhost` / `127.0.0.1` / `[::1]`-ile; mis tahes muu host eeldatakse kuuluvat kellegi teisearvutiks ja seda ei käivitatagi kunagi.
- **Tagapõhi jäetakse taaskasutamiseks töötama** (nagu ka CLI puhul) — skripti lõpetamisel ei toimu vaikimisi sulgemist. Skripti uuesti käivitamisel kasutatakse uuesti töötavat tagapõhja.
- **Loobumine toimub `auto_start_backend=False` abil** mis tahes neist kutsumistest (nt kui oledviidanud kaug-backendile või haldate backend&#x27;i elutsüklit ise).

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

Kui kaasasolevat binaarfaili ei õnnestu leida või käivitada, tekitab järgnev HTTP-kõne tegevusele kutsuva, **platvormist sõltuva** `ChlorosConnectError` , mitte lihtsalt ühenduse keeldumise jälje – veebisaidil Windows suunab see sind töölaua rakenduse või käsu `chloros-cli` juurde; veebisaidil Linux (ilma graafilise kasutajaliideseta) suunab see sind käsu `chloros-cli` või `.deb`-ile.

---

## Keskkond ja päised

SDK märgistab iga tagapõhja HTTP-kõne koodiga `X-Chloros-Client: sdk`. Tagapõhi rakendab SDK / CLI litsentsieeskirju (vajalik on sisselogimine **ja** tasuline Chloros+ pakett), mitte graafilise kasutajaliidese tasuta taseme reegleid. See seadistatakse automaatselt importimise ajal — te ei pea midagi tegema.

`http://localhost` ja `http://127.0.0.1` tuvastatakse kohaliku backendina. Kõned teistele hostidele (nt teie enda analüütikateenusele) jäävad muutmata.

Tagapõhja URL saab ümber kirjutada, edastades `backend_url=` (või `api_url=`, kui `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Mitte-loopback-tüüpi `backend_url` jõuab ainult allika/seadme tagapõhja – kaasasolevad tagapõhjad seovad ainult loopbacki; tunneli mustri kohta vaata jaotist „Kaugtagapõhja režiim”.)

---

## Versioonid ja ühilduvus

- Versioon „SDK” kuvatakse kui `chloros_sdk.__version__`.
- „SDK” seob käitumise kaasasoleva backend-versiooniga. Vanema „SDK” ja uuema backendi segamine toimib tavaliselt (edasiühilduvad lõpppunktid), kuid uuema SDK kombineerimine vanema backendiga võib uutes lõpppunktides põhjustada `404`-vigu — uuendage töölaua rakendus vastavalt.
- Smart-Connecti liides (`connect_camera` / `connect_array` / `connect_daq_sensor`) ja võrguanalüüsi lõpppunkt tagastavad stabiilseid JSON skeeme; uued väljad on täiendavad.

---

## Veaotsinguvihjed

- **`ChlorosAuthenticationError: Login required`** → Käivita sellel arvutil üks kord `chloros-cli login EMAIL PASSWORD` või logi sisse Chloros töölaua rakenduse kaudu.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-Connecti kutsed käivitavad automaatselt kohaliku taustprotsessi, seega ilmub see teade ainult siis, kui komplektis olevat binaarfaili ei leita või seda ei saa käivitada (ntnt pip-põhine host, millel puudub töölaua pakett). Sõnum on platvormispetsiifiline: Windows avage töölaua rakendus või käivitage mis tahes `chloros-cli`-käsk; Linux käivitage `chloros-cli`-käsk (GUI puudub) või installige `.deb`. Kaugbackendi puhul edasta `backend_url=` (ja `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** importimisel → `lattice_sdk` laadimine ebaõnnestus (tavaliselt pole Arena SDKi käitamisaja DLL-failid paigaldatud). Kaameraväline pind töötab endiselt.
- **Array connect tagastab sub-native resolutsiooni**→ tagapõhja smart-prep vähendab kaadri suurust automaatselt, et see mahuks juhtme pikkusesse. Kasutage `analyze_array_network()`, et näha põhjust, seejärel uuendage linki, aktsepteerige vähendamist või edastage `force_tier="slip-emit-and-capture"` järjestikuliseks salvestamiseks. Vähendamise turvavõrk**ei** kata koondatud ülemärkimist (`oversubscribed: true`, fps-väljad 0): liiga paljude kaamerate korral ühendusele ei saa seda binningu/ROI abil parandada — vähendage kaamerate arvu, lubage jumbo-kaadrid või vahetage kiirema võrgukaardi vastu (vt [Ülemärkimine](#ülemärkimine-kaamera-põhja-piir)).
- **`analyze_array_network()` teatab, et võrgukaardi vastuvõturing on väga väike (~0,26 MB) / ühendusväravad näitavad „FRAMES WILL DROP”** → Host-võrgukaardi vastuvõturing on vaikimisi seadistuses (sageli taastatakse see pärast võrgukaardi draiveri uuendamist väärtusele 32). Realtek USB 10GbE-adapteri puhul seadistage `ReceiveBufferLen=256` ja `PendingReceives=64` (kõrgendatud), seejärel taaskäivitage backend, et see loeks ringi uuesti sisse. Täielik juhend: [CLI Viide → Host-võrgukaardi seadistamine ja häälestamine](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host hangub taaskäivitamisel/väljalülitamisel, hiljem ilmnevad WMI-vead `Invalid class` / võrgukaart ei aktiveeru** → Vananenud USB 10GbE draiver põhjustab veateate `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Uuendage adapteri draiver viimase versioonini (≥ 2026) ja rakendage uuesti vastuvõturõnga seaded. Vaadake [CLI Viide → Host-võrgukaardi seadistamine ja häälestamine](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Peegeldusvõime keelati** → Reaalajas andmekogumisüsteem (DAQ) peab olema seotud kaameraga (või kaamerate massiiviga) absoluutskaala peegeldusvõime saamiseks. Seostage kas graafilise kasutajaliidese kaudu või kasutage `processing="radiance"` (W/m²/sr/nm), mis ei vaja paarisandurit.
- **`smart=True` andmete salvestamine võtab oodatust kauem aega** → AE konvergents sõltub stseeni dünaamikast; kui soovite kiiremat (vähem stabiilset) triggert, suurendage `exposure_tolerance_pct` väärtust või lühendage `stability_window_s` väärtust.

---

## Vaata ka

- [CLI Viide](cli-reference.md) — iga CLI alamkäsk peegeldab SDK kutsumist.
- [DAQ-andurite juhend](../daq/README.md) — andurispetsiifilised ühendus-, kalibreerimis- ja salvestusreeglid.
- Veebidokumendid: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
