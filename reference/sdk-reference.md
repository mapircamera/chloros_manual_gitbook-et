# Chloros Python SDK Viide

**Versioon:**

1.2.0**Koostatud:**

29.07.2026 kell 19:19 ·**Muudetud:**

30.08.2026**Pakett:** `chloros-sdk` (PyPI)**Sihtrühm:** Optimeeritud LLM-i kasutamiseks; inimesele loetav.**Kohaldamisala:** Kõik `import chloros_sdk` poolt avalikustatud avalikud klassid, funktsioonid ja abifunktsioonid, koos kopeeritavate ja kleebitavate näidetega, mis hõlmavad pilditöötlust, ühe kaamera juhtimist, sünkroniseeritud massiive, DAQ-andureid ja projekti automatiseerimist.

Kui vajate ainult olulisimat, minge otse järgmistesse osadesse:
- [Paigaldamine ja kiirstart](#installation)
- [Smart-Connect LATTICE-kaamerate jaoks](#smart-connect-for-lattice-cameras)
- [DAQ-andurite seansid](#daq-sensor-sessions)
- [Projekti automatiseerimine](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arhitektuur 60 sekundiga

SDK on õhuke Python kiht Chloros tagapõhja peal (sama Flask-server, mida kasutavad töölaua graafiline kasutajaliides ja CLI kasutavad). Automatiseerimiseks impordite `chloros_sdk` ja kutsute välja kõrgetasemelisi meetodeid; sisemiselt muutub iga kutse HTTP-päringuks kohalikule tagapõhjale pordil 5000 — `http://127.0.0.1:5000/api/...` (tahtlikult mitte `localhost`, mis suunatakse esmalt `::1`-ile Windows-is ja kulutab ~2 s iga päringu kohta ainult IPv4-tagantpoolt). Tagantpool omab riistvarapooli — kaamerad, DAQ-andurid, joondamisprofiilid, kaadripuhvrid — nii et SDK skriptid saavad GUI-ga koos eksisteerida, ilma et nad peaksid võitlema jadaportide või võrgukaardi ribalaiuse pärast.

Kasutada on kolm liidest:

1. **`ChlorosLocal` + vabad funktsioonid** (`process_folder`, `process_lattice_capture`) — Pilditöötluse töötlusvoog. Kogu kausta töötlemine kalibreerimise, debayeri ja indeksite ekspordi kaudu ühe Python-kõne abil.
2. **Smart-connect-käepidemed** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — avage püsiv tagapõhja-seanss reaalajas töötavale riistvarale. Sama „smart-prep“ voog nagu graafilises kasutajaliideses: võrguproov, taseme automaatvalik, PTP, AE algväärtustamine, GPIO-trigeri konfiguratsioon.
3. **`ChlorosProject` / `open_project`** — Laadige salvestatud projekt (kaust, mis sisaldab `cameras.json` + `sensors.json` + `project.json`), ühendab kõik korraga ja juhib salvestamist nimeliste käepidemete abil.

Pinnad 1 ja 2 **käivita automaatselt kohalik tagapõhi**, kui see veel ei ole kuulamisrežiimis (sama komplektis olev binaar, mida käivitab GUI/CLI) — nii töötab lihtne skript värskes shellis, ilma et peaksid backendit esmalt käivitama. Edasta `auto_start_backend=False`, et sellest loobuda (nt kui viidatakse kaug-backendile, mida kunagi ei käivitatud). Vaata [Backendi automaatne käivitamine](#backend-auto-start). Surface 3 käitub teisiti: `open_project()` ei võta vastu parameetrit `auto_start_backend`, ja `connect_all()` ei käivita kunagi backendit — see proovib `http://127.0.0.1:5000`-i üks kord ja kui keegi ei vasta, lülitub vaikselt tagasi otsesele (backendita) `lattice_sdk` seadme juhtimisele. Ainult `proj.process()` ja `stream(..., overlays=True)` loovad viivitusega `ChlorosLocal()`-i (mis teostab automaatse käivitamise).

Kõik kolm on autentimispõhised: käivita `chloros-cli login` masinal üks kord või logi sisse töölaua graafilise kasutajaliidese kaudu. SDK-i kutsed ilma kehtiva sessioonita tekitavad vea `ChlorosAuthenticationError`.

Nõuded:
- Python 3.7+ (nagu pakendis märgitud; arendatud/testitud versioonil 3.10)
- Chloros Desktop peab olema kohalikult installitud (tagapõhja binaar on paigaldaja sees)
- Aktiivne Chloros+ sisselogimine. SDK/CLI-tase on **Copper**või kõrgem (Copper / Bronze / Silver / Gold); tasuta**Iron**-tasemel puudub SDK/CLI juurdepääs. Seda rakendatakse**serveri poolel**: iga SDK/CLI-märgistusega päring peab sisaldama nii aktiivset seanssi kui ka tasulist paketti, vastasel juhul tagastab tagapõhi `403` koos `error_code: PLAN_UPGRADE_REQUIRED` (mida `ChlorosLocal` kuvab kui `ChlorosLicenseError` ja `connect_*` abifunktsioonid kui `ChlorosConnectError`). välja logitud helistaja saab selle asemel veakoodid `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — need kaks on erinevad, sest `chloros-cli login` uuesti käivitamine parandab esimese, kuid ei suuda parandada teist.
- Offline-kasutust toetatakse paketi armuaegade jooksul: taseme andmed loetakse serveri valideerimise vahemälust (5 minutit) või allkirjastatud, seadmega seotud litsentsi vahemälust (30 päeva kuupakettide puhul, aastapakettide puhul kuni tellimuse lõppemiseni). Kui see ajapikendus lõpeb, muutub pakett tasuta versiooniks ja juurdepääs SDK/CLI-ile peatatakse, kuni seade suudab serveriga ühendust saada. `chloros-cli status` (`GET /api/license-status`) jääb tasuta tasemel kättesaadavaks, seega on põhjus nähtav – see on ainus SDK/CLI marsruut, mis on taseme piirangust vabastatud.
- Windows 10/11 64-bitine, **Ubuntu 22.04 LTS või uuem**või Jetson (JetPack 6). Ubuntu 20.04**ei** toetatud: `.deb` sõltuvused tulenevad sellest, millega tagapõhi on seotud, sealhulgas `libc6 (>= 2.34)`, ning Focal tarnib glibc 2.31.

---

## Paigaldamine

Python SDK on õhuke Python kiht Chloros backend&#x27;i peal. Kõigi ainult DAQ-le keskenduvat töövoolu, peab olema **kohalikult installitud Chloros töölaua pakett** (Windows installija või Linux `.deb`) – see pakub backend-binaari, Arena SDK-i käivituskeskkonda LATTICE-kaameratele ning kalibreerimispakette.

Viimased allalaadimised: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### 1. samm — Paigalda platvormipakett Chloros

#### Windows (.exe)

1. Laadige alla `Chloros-Setup-x.y.z.exe` allalaadimislehelt.
2. Käivitage paigaldaja ja järgige paigaldusviisardi juhiseid. Vaikimisi paigalduskataloog on `C:\Program Files\MAPIR\Chloros\`.
3. Käivita Chloros vähemalt üks kord ja logi sisse oma Chloros+ kontoga.

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

### 2. samm — Paigalda Python SDK

**Chloros paigaldaja sisaldab sobivat SDK wheel-faili.** Iga Windows paigaldaja ja Linux .deb-fail paigutab kettale `chloros_sdk-X.Y.Z-py3-none-any.whl`-i, mis vastab täpselt GUI / CLI / backend-versiooniga. Te ei pea PyPI-d jälgima, et versioonid sünkroonis püsiksid.

#### Windows

Paigaldaja käivitab automaatseltkäivitab `pip install` komplektis oleva wheel-faili abil, kasutades teie süsteemi Python (eelistatakse `py.exe` käivitajat, tagavaraks on `python -m pip`). Tegevust ei ole vaja – `import chloros_sdk` töötab teie Python keskkonnas pärast edukat paigaldamist. Kui arvutis puudub Python, jätab paigaldaja selle sammu vaikimisi vahele ning graafiline kasutajaliides ja CLI töötavad edasi.

#### Linux (.deb)

.deb-fail paigutab ratta kataloogi `/usr/lib/chloros/sdk/`. `postinst` kuvab täpse käsu — PEP 668 distributsioonid keelavad vaikimisi globaalsed pip-kirjutused, seega me ei paigalda seda automaatselt:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Air-gapped Jetsoni kasutuselevõtude puhul toimub see täielikult offline-režiimis – wheel on juba kettal.

#### Avalik PyPI

Ainult pip-i kasutavate hostide puhul (Chloros töölaua pakett pole paigaldatud; remote-backend või ainult DAQ-põhised töövood):

```bash
pip install chloros-sdk
```

PyPI uuendatakse versiooniväljalaske installeri koostamisel, seega vastab avaldatud wheel viimasele stabiilsele versioonile. Arendajaversioonid (nt `1.1.4.dev1`) levitatakse ainult komplektis oleva installeri wheel’i kaudu.

#### Kontrolli

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Vajalik on Chloros+ tellimus.** Kõik SDK-kutsed nõuavad aktiivset Chloros+ kasutajatunnust. Käivita `chloros-cli login user@example.com 'YourPassword'` üks kord iga masina kohta; kasutajatunnused salvestatakse vahemällu `~/.chloros/`.

### Kas mul on vaja töölaua paketti?

Ainult pip-pakett ei ole **ei** piisa enamiku töövoogude jaoks. Siin on loetletud, mida iga SDK-pind vajab:

| SDK-pind | Kas vajab töölaua paketti? | Miks |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Jah** | Käivitab taustaprogrammi automaatselt `/usr/lib/chloros/chloros-backend` (Linux) või `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Jah**(kohalik)**/ Ei**(kaugjuurdepääs) | Puhtad HTTP-kliendid tagapõhja kaudu. Kohalik tagapõhi → vajalik töölaua pakett. Kaugtagapõhi → `backend_url=`**tunneli kaudu** (vt kaugtagapõhi režiim — kaasasolevad tagapõhjad seovad ainult loopbacki). |
| `ChlorosProject` / `open_project` | **Jah** | Juhib salvestatud projekte tagapõhja kaudu. |
| Otsesed LATTICE-klassid (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Jah** | Vajab Arena SDK natiivset käitamiskeskkonda, mis on kaasas töölaua paketis. Muidu on `CAMERA_AVAILABLE` importimisel sama mis `False`. |
| Otsesed DAQ-klassid (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Ei** | Puhas Python pyserial/bleak/zeroconf kaudu. Ainult pip-põhine keskkond suudab DAQ-seadmeid läbi kogu ahela juhtida. |

### Kaug-tagapõhja režiim (ainult pip-i kasutav host, tunneli kaudu)

> **Kaasasolev tagapõhi ei ole LAN-i kaudu kättesaadav.** Tootmisversioonid
> seovad ainult loopbacki (mõlemad loopbacki perekonnad) ja keelavad kategooriliselt
> ainsa mitte-loopbacki režiimi (`CHLOROS_CLOUD_MODE`), seega
> `backend_url="http://<lan-ip>:5000"` **ei saa töötada koos installitud
> Chloros** — see skeem töötas alati ainult source/dev
> tagapõhja puhul. Teisel masinal asuva tagapõhja juhtimiseks suuna selle loopback-
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

Peata / CI / robootika-hostid võivad hoida ühte masinat täieliku töölaua installatsiooniga „Chloros serverina” ja `pip install chloros-sdk` kõikjal mujal — kuid nende vaheline andmeedastus toimub eespool kirjeldatud kasutaja-korraldatud tunneli kaudu, mitte otsese LAN-i kaudu URL.

> **Teadaolev piirang — `ChlorosLocal` ei toeta ainult pip-i.** `ChlorosLocal(backend_url=BACKEND)` lahendab praegu oma konstruktoris kohaliku tagapõhja binaarfaili *enne* URL-i kontrollimist ning tekitab veateate `ChlorosBackendError` (&quot;Chloros tagapõhja ei leitud…”), kui ühtegi töölaua paketti pole installitud — isegi juhul, kui kaugbackend on kättesaadav. Ainult ülaltoodud smart-connect-liides (`connect_camera` / `connect_array` / `connect_daq_sensor`, pluss `analyze_array_network` ja abiprogrammid `list_*` / `discover_*`) töötab ainult pip-i kasutaval hostil.

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

Otse riistvaraga andmete kogumiseks ei ole vaja tagapõhja, .deb-faili ega Chloros+ sisselogimist.

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

## API pealehekülg

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

Peamine töötlusklass. Käivitab backend&#x27;i esmakordsel kasutamisel, loob ja konfigureerib projekte, jälgib protsessi kulgu ning tagastab töötlemise järel kokkuvõtted.

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
| `import_images(folder_path, recursive=False)` | RAW/TIF/JPG/DNG-pildid **ja `.daq` valgusanduri salvestised**. Tagastab `count` (pildid) ja `scan_count` (salvestised). Hoiatab ainult juhul, kui kaustas pole kumbagi. |
| `export_light_sensor(daq=True, csv=True)` | Kirjutab kalibreeritud `.daq` + `.csv` iga projekti valgusanduri salvestuse kohta kausta `<project>/Light Sensor/`. Vaata [Valgusanduri salvestused](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Määra töötlemise parameetrid. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Käivita töötlemisahel. Tagastab `{"status": "complete", "async": False}` ning lisaks `summary` võtme, kui tagapõhi seda pakub — vt [Käivitusejärgne kokkuvõte ja näpunäited](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Kontrolli tagapõhja seisundit. |
| `logout()` | Tühjenda vahemällu salvestatud autentimisandmed. |
| `shutdown_backend()` | Lõpeta tagapõhi (kui see on käivitatud SDK-ga). |
| `discover_cameras()` | Leia LATTICE-kaamerad **selle instantsi tagapõhja kaudu** (`/api/camera/discover`). Tagastab sõnastikute loendi (`serial`, `model`, `ip`, …) – sama kujuga, mida näeb GUI/CLI. Tühi nimekiri, kui ühtegi ei leitud või tagapõhi pole kättesaadav. |
| `camera_capture(output_dir, format="tiff", **settings)` | Salvestab ühe kaadri**tagapõhja kaudu**(käivitatakse automaatselt selle käepideme abil), nii et see saab sama ettevalmistuse kui GUI/CLI (vaikimisi 12-bitine, mäluploki taaskasutus, sisseehitatud kalibreerimismetadandid). Lahendab sihtmärgi `serial=` või `device_index=` abil; edasta `exposure`/`gain`/`pixel_format`/`preset` edasi kui `**settings`. Tagastab vana metaandmete sõnastiku (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Annab ülekattega koostatud eelvaatekaadrid koondatud kaamera — õhuke MJPEG-klient tagapõhja `/api/camera/<serial>/stream-annotated` marsruudi kaudu (zebra / võre / sihtjoon / histogramm / tipp / serveripoolne joonistatud täpp). `decode=True` toodab BGR-massiive; `False` toodab toorandmeid JPEG baitides. Samuti kättesaadav-projekti kaudu kui `ChlorosProject.stream(overlays=True)`. |

Kasutada kontekstihaldurina tagatud puhastamiseks:

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

DAQ-U / DAQ-M / DAQ-E-d on võimalik salvestada **ilma** nende kalibreerimispakettideta. See tähendab,
teevad avalikud [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
salvestid (`record_daq.py`) vaikimisi: nad kirjutavad salvestusse andurite toorandmed ja märgivad
faili, nii et Chloros laadib selle anduri tehase kalibreerimisandmed **seerianumbri järgi** – esmalt kohalikust vahemälust,
seejärel MAPIR pilvest – ja rakendab neid importimisel.

Chloros kirjutab tulemuse tagasi kahe tootena salvestuse kohta, allpool
`<project>/Light Sensor/`:

| Toode | Mis see on |
| --- | --- |
| `<name>_calibrated.daq` | Ümbertöödeldav arhiiv — sama skeem kui reaalajasalvestusel, nüüd koos selle loonud paketi deklaratsiooniga. Selle uuesti importimisel ****ei** kalibreeri seda teist korda. |
| `<name>_calibrated.csv` | Spektraalne kiirgustihedus ühikutes W/m²/nm anduri enda lainepikkuste võrgustikus, üks rida iga mõõtmistulemuse kohta, lisaks fotomeetrilised veerud (koguvõimsus, fotopiline/skotopiline luks, PPFD ja selle sinine/roheline/punane jaotus, tipp-lainepikkus). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Ainult pakettideta andurid (DAQ-A).** Andurite toorespektraalsed loendid — *mitte* kiirgustihedus. Vaata allpool. |

`process()` teostab seda eksporti ühe oma etapina. See **ei** vaja pilte:
eraldi lennutatud valgusandur on esmaklassiline töövoog ning sellisel projektil puuduvad
pildid.

**DAQ-A salvestused eksporditakse toorloenditena.** DAQ-A seeria on varasem kui seerianumbri
põhine kimpude süsteem ja sellel pole kimpu, mida alla laadida — see kalibreeritakse välitingimustes
peegeldusmäära sihtmärgi abil, mistõttu pole seda kunagi vaja olnud. Need salvestised eksporditakse
tüübiga `_raw`, mitte `_calibrated`: tegemist on erineva failinimega, mitte lipuga
faili sees, sest nimetus peab säilima ka siis, kui seda edastatakse e-kirjas pelgalt nimena. Pealkirjas
`.csv` on märgitud `raw spectral sensor counts (NOT irradiance)` ja hoiatatakse, et
väärtused on võrreldavad **faili sees** — just selleks, milleks sihtmärgipõhine kalibreerimine
neid kasutab — ja mitte andurite vahel. Võimsusest sõltuvad fotomeetrilised veerud (koguvõimsus,
fotopiline/skotopiline luks, PPFD) kuvatakse **NULL**, mitte loendustest integreerituna.

DAQ-U / DAQ-M / DAQ-E, mille paketti lihtsalt ei õnnestunud alla laadida, **jäetakse ikkagi vahele**,
mitte kirjutatakse toorandmetena: seal pakett eksisteerib ja „taasühenda ja töötle uuesti“ on tõeline nõuanne.

Vana versiooni **v1.01 / v1.02** salvestused (neid kirjutab DAQ-A-SD) ei sisalda lugemise-põhiseid ajavahemikke,
vaid ainult faili kirjutamise aja. Pildi↔allapoole suunatud valguse sobitaja lükkab need endiselt tagasi — kaadri
sobitamine kirjutamise ajaga oleks märkamatult vale — kuid eksportija loeb neid, ja
CSV väljastab `clock=daq_created_on`, nii et tulemus näitab, millisel kellal see põhineb.

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

Salvestus, mille kalibreerimispaketti ei ole võimalik alla laadida (offline-režiimis või andur, mille kohta
failis puudub kalibreerimine), kajastatakse `skipped` all **koos põhjendusega**. Seda ei
kirjutata kunagi välja „kalibreeritud” failina, mis sisaldab töötlemata loendusi – ühenduge internetiga ja
käivitage uuesti, ning eksportimine lõpeb.

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

### Järelanalüüs ja näpunäited

Lõpetamisel laadib `process()` alla `GET /api/processing-summary` ja lisab selle sisu `result["summary"]`-na. Allalaadimine on väheja ei takista kunagi edukat tagasipöördumist — kui kokkuvõte pole kättesaadav, kasutab `process()` tagavarana lihtsat `{"status": "complete", "async": False}`-vormi. Iga kirje `summary["hints"]`-is — täielikud laused koos soovitatud parandusmeetmetega, nt miks käivitus andis nullväljundi — edastatakse samuti uuesti kujul Python `UserWarning`, nii et nullväljundiga käivitused on isediagnoosivad isegi siis, kui sa sõnastikku kunagi ei vaata:

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
| `images_calibrated` | Käivituse käigus kalibreeritud pildid. |
| `exported_files` | **Käivituse käigus loodud pilditootefailid.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Valgusanduriandurite salvestused, mida loendatakse tahtlikult eraldi — need pärinevad teisest etapist ja esinevad ka jooksudes, kus pilte üldse pole, seega nende kaasamine muudaks ainult andmete kogumisele pühendatud jooksu selliseks, nagu oleks see eksportinud pilte. |

Nende kõrval: `summary["output_dirs"]` (iga kataloog, kuhu on kirjutatud),
`summary["light_sensor_export"]`, `summary["stopped"]` (tõene, kui kasutaja katkestas
jooksu, nii et osalised loendid ei näita seda kui lõpetatud jooksu, mille toodang oli puudulik) ja
`summary["groups"]` (jaotus rühmade kaupa).

`exported_files` salvestatakse töötlusketi poolt **kirjutamise ajal**, mitte skannitakse hiljem
projektipildiobjektidest hiljem skaneerides. Paralleel- ja GPU-strateegiad loovad oma pildiobjektid
(GPU-teede puhul töötajate alamprotsessides), seega näitas vana skaneerimine
`0 file(s) written` iga sellise käivituse puhul ja andis seejärel välja null-eksportimise vihje — käivituste puhul,
, kus kõik oli töötanud. Kui te loote skripti selle numbri põhjal, näitab nüüd tõrgeteta paralleelkäivitus
nullist erinevat arvu.

Vahelejätmiste põhjused näitavad põhjust, mille lugeja iga faili puhul tegelikult kindlaks tegi –
loetamatu skeem, puuduv kimp, kirjutamisviga – **duplikaatidest puhastatuna**, nii et kakskümmend faili,
mis ühe põhjuse tõttu vahele jäid, kuvatakse ühe põhjusena, mitte selle kahekümne kordusena.

> **`process()` ei teki, kui käivitamine ei tooda ühtegi pilti.** See on ainus koht, kus SDK ja
> CLI teadlikult erinevad: `chloros-cli process` käsitleb olukorda „tooteid taotleti, kuid ühtegi ei
> kirjutatud” veana ja lõpetab töötamise nullist erineva tulemusega, samas kui SDK lõpetab töötamise normaalselt ja teatab
> olukorrast `summary` / vihjete kaudu. Kui teie töötlemisahel peaks tühja käivituse korral peatuma, kontrollige seda
> ise üle – kontrollige `summary`-i (või loendage projekti kausta all olevaid faile), selle asemel et tugineda
> erandite puudumisele. Tavapärased põhjused on sisendkaust, mida ei tunnistatud
> salvestusena, ning tooted, mis jäeti vahele, kuna need ei olnud olemasolevate kaamerate jaoks kohaldatavad (nt kiirgus ainult RGB-kaameratelt
> ).

### Abifunktsioonid

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

#### Radiomeetriline väljund (LATTICE multispektraalne töötlusprotsess)

`process` töötlemisahela LATTICE multispektraalne (M3C/M3M) eksporditase — `reflectance` (vaikimisi), `radiance`, `sensor-response` või `all` (iga pildi puhul vastav režiim) — vastab projekti **„Radiomeetriline väljund“** töötlemisseadistusele. `configure()`-il on selle jaoks spetsiaalne märksõna:

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

Täpsem pääsetee — projekti `"Radiometric output"` võtme kirjutamine läbi `custom_settings` — toimib endiselt, kuid pidage meeles, et see asendab kogu seadeteploki (vaadake allpool olevat hoiatust):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (vaikimisi) jagab kaamera kiirgust **ajamärgiga sobitatud DAQ-i allapoole suunatud kiirgusega**, mida määratakse automaatselt salvestatud `.daq` (DAQ-U/M/E)**või piltide juures leitud DAQ-M-i omapärase `.csv`**alusel; kõik kohapeal puuduvad kaamera- või DAQ-kalibreerimispaketid**laetakse automaatselt AWS-ist** esmakordsel kasutamisel. CLI kuvab seda tüübipõhiste tootevalikutena `chloros-cli process`-is: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **asendab** kogu arvutatud seadete ploki (see jätab disaini tõttu `configure()` muud märksõnad ja valideerimise vahele). Kui seda kasutate, lisage kõik teile olulised `Project Settings`-võtmed, nagu eespool toodud näites.

---

## Smart-Connect LATTICE-kaameratele

Püsivad tagapõhja-seansid reaalajas töötava riistvara jaoks. Kasutatakse samu lõpppunkte, mida kasutab graafiline kasutajaliides, seega on käitumine identne SDK / CLI / graafilise kasutajaliidese vahel.

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

#### `CameraSession` meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Loeb GenICam-sõlmi; tagastab `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Kirjutab sõlmed sõbraliku nime järgi (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Salvestab **ühe** kaadri. Tagastab ühe elemendiga loendi, mis sisaldab kaadri metaandmete sõnastikke. (Seeria- ja mitmekaadriline salvestamine on eemaldatud — kui vajate seeriat, kutsuge silmuses välja `capture()`.) |
| `disconnect()` | Vabastab ressursi reservist. Ei tee midagi, kui oleme ühendatud juba avatud sessiooniga. |

`capture()` ekspordikontrollid (sama mudel nagu massiiv + graafiline kasutajaliides):

- `processing` / `levels` — `processing="all"` salvestab kõik kohaldatavad eksporditüübid; `levels=["raw","radiance"]` salvestab ainult need (asendab `processing`). Jäta mõlemad välja, et kasutada tagapõhja vaikimisi seadeid.
- `force_daq=True` — salvestab määratud DAQ/DLS-näidu `.daq`-sidecarina isegi ainult toorandmete kogumisel, et raami saaks hiljem ümber töödelda peegeldusvõime/indeksiks. Ei tee midagi, kui DAQ-i pole seotud.

### Sünkroniseeritud massiiv — `ArraySession` (Smart-Prep)

`connect_array` on **soovitatav alguspunkt** mitme kaameraga seadistuste jaoks. See käivitab taustal täieliku GUI smart-prep-voo:

1. **Võrguanalüüs** (`/api/camera/array/recommend`) — leiab suurima raami suuruse, mis mahub sim-emit-tasemele ilma raamide kaotamiseta.
2. **Taseme automaatvalik** — `sim-capture-sim-emit`, kui ühendus seda toetab; muidu `sim-capture-ftd-stagger` või `slip-emit-and-capture`.
3. **Automaatne vähendamine**— vähendab vaikselt kaadri suurust / suurendab binningut, kui ühendus ei suuda nõutud resolutsiooni tagada.**See turvameede ei hõlma koondatud ülemärkimist**: liiga palju kaameraid ühendusliini jaoks ei saa kaadrite vähendamisega parandada — vt [Ülekoormus](#over-subscription-the-per-cam-floor).
4. **PTP on vaikimisi sisse lülitatud** — kaameratevahelised ajamärgid on võrreldavad mikrosekundite täpsusega.
5. **Kaamera-põhine pikseliformaadi automaatvalik** — RGB kaamerad → `BayerRG8`, multispektraalsed → `BayerRG12`.
6. **AE algväärtustamine** — salvestab iga kaamera hetke AE-seisundi, et ühendamine ei nulliks säritust töö käigus.
7. **GPIO-trigeri konfiguratsioon** — `connect_array` aktiveerib kõik kaamerad (`TriggerMode=On`, `TriggerSource=Line2`), nii et peakaamera impulss juhib allkaameraid M8-kaabli kaudu. See samm kehtib ainult kaamerate massiivi puhul: üksik kaamera, mis on avatud käsuga `LatticeCamera`, töötab selle asemel vabalt.

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
- `"sim-capture-sim-emit"` — tõeline samaaegsus (kõik kaamerad vallanduvad samal taktsignaalil).
- `"sim-capture-ftd-stagger"` — paindlik ajadomeeni nihutus (kaamerad edastavad andmeid veidi nihutatud aegadel, nii et paketid järjestuvad võrgus).
- `"slip-emit-and-capture"` — järjestikune salvestamine kaamera kaupa (ajaliselt sünkroniseerimata; ainus valik, kui ükski raami suurus ei sobi sünkroonsusega).

`wire_ceiling_mbps` ületab **host&#x27;i püsiva andmeside eelarve** MB/s-is — see on ainus
arv, millest sõltub kogu massiivi eraldus. Jätke see väärtuseks `None`, et kasutada automaatselt tuvastatud
väärtust. Vähendage seda, kui massiiv teatab GVSP-rikkis raamidest: automaatne väärtus tuletatakse
võrgukaardi teatatud ühenduskiirusest, mis ülehinnab USB-adaptereid, kitsaid PCIe-kanaleid ja
hõivatud jagatud võrgustikke – ning see ülehinnang avaldub pigem vigastena raamidena kui
nähtavalt aeglase ühendusena. Väärtus salvestatakse projekti massiivi salvestusplokki, seega
taasavamisel või hiljem `connect_array` taastab selle nagu iga muu massiivi seade.
Vaata [Massiivi tervis](#array-health--which-subsystem-is-losing-frames).

#### Ülemäärane eraldamine (kaamera kohta kehtiv alampiir)

Sim-emit-tempo eraldab igale kaamerale osa kokkupõrkeohutust andmeside eelarvest, mille alampiir on **8 MB/s kaamera kohta**(`per_cam_floor_bps`). Kui `N × floor` ületab kokkupõrkeohutu ülempiiri, siis massiiv**ülekoormab võrgu**— rikkeviisiks on GVSP-pakettide kadu, mitte madalam kaadrisagedus — ning kaadri suuruse osas ei ole lahendust:**koondamine ja ROI vähendavad baitide arvu kaadri kohta, mitte sekundi kohta jaotatud baite**, mida koondkontroll võrdleb. Praktilised täisresolutsiooni ülempiirid 1 GbE-hostil:**6 kaamerat @ 1500 MTU, 9 neist jumbo-kaadritega** (analüüsivastuses `max_cams_collision_safe` näitab teie ühenduse piirmäära). Lahendused: vähem kaameraid, jumbo-kaadrid kogu ühenduse ulatuses või kiirem võrgukaart.

- `analyze_array_network()` ja `/api/camera/array/connect` vastused sisaldavad `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` ja `per_cam_floor_bps`. Kui `oversubscribed` on tõene, **nullib projektsioon fps-väljad** (`achievable_fps_max` / `fps_bright` / `fps_dark`) nullib, selle asemel et näidata eksitavat aeglast, kuid toimivat kiirust.
- `POST /api/camera/array/connect` aktsepteerib `pin_resolution` keha parameetrit (**ainult HTTP – mitte SDK-i kwarg**; `connect_array` ei avalda seda). Kinnitamine eemaldab binningu järkjärgulise vähendamise turvavõrgu, mistõttu ülemäära koormatud ühendus, kus on seatud `pin_resolution`,**lükatakse kindlalt tagasi** koos veateatega, milles nimetatakse kõik võimalikud lahendused. Ilma kinnitamiseta jätkub ühendus järkjärgulise vähendamisega, kuid hoiatab, et vähendamine ei suuda kogumit tühjendada.
- Test-töö pääsetee: seadista tagapõhja keskkonnas `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`, et alandada keeldumist valjuhäälseks hoiatuseks – sa ühendud niikuinii ja aktsepteerid pakettide kadu.

#### Massiivi tervis — milline alamsüsteem kaotab raame

`GET /api/camera/array/<array_id>/capability` kannab reaalajas `health`-plokki
ühendatud massiivis, mida-hinnatakse jooksvas **10-sekundilises** aknas. See jagab raamikaotuse
kaheks põhjuseks, mis vajavad vastupidiseid parandusi, selle asemel, et kasutada üht „ebatäielikku“ määra, mis
mõlemat ei nimeta:

| Väli | Mida see tähendab | Milline alamsüsteem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (seerianumbri kaupa) | Kaader **saabus, kuid oli struktuuriliselt vigane**— GVSP-pakettide kadu. |**Võrk**: kaabli läbilaskevõime, tempo, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (seerianumbri kaupa) | Raam **ei jõudnud üldse kohale**— kaamera ei käivitanud või sellest ei tulnud midagi välja. |**Trigger / sünkroniseerimine**: M8-kaabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Iga kaamera halvim edastussagedus. | — |
| `per_cam_rate_pct` | Kaamera kohta arvutatud kombineeritud puudulikkuse määr (mõlemad põhjused kokku). | — |
| `stable_for_seconds` | Kui kaua on iga kaamera püsinud alla 0,01 % taseme. | — |

Koos `health`-iga näitab sama kirje ka kogu eraldise kasutamata osa suurust:

| Väli | Mida see tähendab |
| --- | --- |
| `wire_ceiling_mbps` | Hosti kehtiv püsiv võrguühenduse maht, MB/s. |
| `wire_ceiling_source` | Selle numbri päritolu, sõnadega — nt `USB-capped 200 MB/s (was theoretical 1062; …)` või `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, kui `wire_ceiling_mbps=` selle seadis. |
| `nic_is_usb` | `true` USB-Ethernet-adapteri puhul. |

Selle lõpppunkti jaoks puudub SDK-ümbris – loe seda otse:

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

**Lugemine:** nullist erinev `gvsp_corrupt_rate_pct` koos väärtusega 0 tähendab, et
trigger ja kaabli sünkroniseerimine on täiuslikud ning 100 % kaotusest on võrguühenduses — alandage
`wire_ceiling_mbps` ja ühendage uuesti. Vastupidine muster viitab pigem sünkroniseerimiskaablile või
triggeriliinile.

> **`target_fps` ei ole rikutud raamide puhul reguleerimise võti.** GevSCPD-i taktsagedus kirjutatakse ühe korra
> ühendamisel, seega triggerisageduse alandamine muudab töötsüklit, mitte
> samaaegse edastamise purske kiirust. Mõõdetud 5-kordne nõudluse vähendamine ei toonud parandust, samas kui
> kaabli ülempiiri langetamine 240-lt 200 MB/s-le viis sama seadme vigaste raamide määra 10,4 %-lt
> 0,00 %-ni.

> **TRI032S-i püsivara ei toeta andmevoo keskel toimuvat automaatset vähendamist.** Töötav massiiv ei suuda
> seda ise parandada; katkesta ühendus ja ühenda uuesti, et ühendusaja valija koostaks uue plaani vastavalt
> uuele ülempiirile.

**USB-Etherneti-adapteri kiirus on sondi poolt piiratud 200 MB/s**-ni, sõltumata selle
nimikilbil märgitud andmetest: tõhusustabel, mis teisendab ühenduse kiiruse püsivaks näitajaks, on
PCIe-põhine, ning USB-võrgukaart edastab oma Etherneti-ühenduse kiiruse, olles samas piiratud
USB-siini ja selle draiveri poolt. Piirang on absoluutne, mitte suhteline – USB 1 GbE-adapter
saavutab ~80 MB/s ja see ei mõjuta seda.

#### `ArraySession` meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Üks sünkroniseeritud salvestusgrupp. Tagastab `CaptureResult` (raamistikute nimekirja kaadrite sõnastikest + `.skipped`). Ekspordikontrollid allpool. |
| `capture(..., smart=True)` | **Nutikas salvestamine** — ootab, kuni AE on kõigil kaameratel stabiliseerunud, ja käivitub seejärel. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Kiireim salvestamine: ainult toorandmed + määratud DAQ-näidud (+ vaba kombineeritud indeks). Vastab GUI-s olevale nupule „Kiireim salvestamine”. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Ühekordne / Pidev / Intervall ühes piiratud tsüklis. Tagastab `list[CaptureResult]`.**Nõuab `count` ja/või `duration_s`**, et see lõpetataks (SDK-il puudub Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Alustab kombineeritud indeksivaate salvestamist videoks/GIF-iks → `RecorderHandle`. Üks komposiitsalvesti iga massiivi kohta. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Alusta kõrge kaadrisagedusega toores Bayer-pildiseeria salvestamist → `RecorderHandle`. Töötle offline-režiimis uuesti `build_video()` abil. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Töötle salvestatud toores seeria offline-režiimis ümber kalibreeritud videoks/videodeks. Blokeerib kuni lõpetamiseni (`wait=True`) ja tagastab `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Küsitle offline-koostamistööd: `{running, result, error, burst_dir}`. |
| `disconnect()` | Vabasta kogu massiiv. |

`capture()` ekspordikontrollid (sama lõpppunkt, mida kasutab GUI/CLI kasutamine):

- `processing` / `levels` — `processing="all"` (või `levels=["raw","radiance",…]`) salvestab iga kaamera kohta kõik kohaldatavad eksporditüübid; üks `processing` väärtus salvestab vaid selle taseme.
- `aligned=True` — kohandab iga elemendi mitte-toorekspordi massiivi [joondusprofiiliga](#array-alignment) (ko-registreeritud); tooreksport jääb kohandamata, kuid sisaldab teisendust metandmetes. Kui massiivil puudub profiil, kasutatakse tagavarana joondamata versiooni (koos hoiatusega tulemuse `alignment`-is).
- `render_index=False` — jätta vahele kaamera-põhine taimestikuindeksi pealekandmine; vaikimisi renderdatakse see seal, kus see on konfigureeritud.
- `force_daq=True` — salvestab määratud DAQ/DLS-näidu `.daq`-sidecarina isegi siis, kui ükski valitud tase seda ei vaja.

**TIFF-pakkimine (ainult HTTP-regulaator):**`ArraySession.capture()` ei saada `compression`-võtit, seega kehtib tagapõhja vaikimisi seade — `POST /api/camera/array/capture` loeb `compression`-keha parameetrit, `"deflate"` vaikimisi (kaotuseta zlib L1 + horisontaalne ennustaja, ~4,1 MB täisresolutsioonilise kaadri kohta). `"none"` kirjutab tihendamata (~6,3 MB/kaader)**~5× kiirema kirjutuskiirusega** — mõlemad on kadudeta ja loetakse importimisel identselt. SDK ei paku selle jaoks ühtegi kwarg&#x27;i; pääseteeks on `chloros-cli lattice array-capture --compression none` või toores HTTP. DEFLATE hoiab kinni ka Python GIL-i, seega kompresseeritud kirjutamised ei toimu paralleelselt kaamera-põhiste kirjutamislõimede vahel — püsiv 8-kaameraga täisresolutsiooniga salvestamine anduri kiirusel vajab `compression: "none"`-i. Täpsemalt: [CLI Viide → massiivi salvestamine](cli-reference.md).**Liikmekohased ekspordi ümberkirjutused (ainult HTTP):**sama lõpppunkt aktsepteerib ka `exclude_serials` (loetelu — liikmete eemaldamine salvestatud hulgast; massiiv käivitub endiselt ühe sünkroniseeritud rühmana ja väljajäetud liikmed tagastatakse `excluded`-is), `serial_levels` (`{serial: [level tokens]}` kaamera tasandi ümberkirjutused) ning `serial_index` (`{serial: bool}`kaamera-indeksiga ülekatte ülekirjutused). Need on GUI-pariteediga põhiparameetrid ja**veel mitte SDK-i kwargid**; kaartidelt puuduvad liikmed kasutavad tagavarana massiiviüle kogu massiivi ulatuvale `levels` / `render_index`-le.

##### Vahelejäänud kaamerate kontrollimine — `CaptureResult.skipped`

`ArraySession.capture()` tagastab `CaptureResult`, mis on `list` alamklass: iteratsiooni korral indekseeri seda, `len()` seda — kõik olemasolevad mustrid toimivad endiselt. Uus kood saab kontrollida `.skipped` atribuuti, et näha, millised kaamerad jäeti välja ja miks. Kõige tavalisem juhtum on RGB kaamerasegafiltreid sisaldavas massiivis, kui küsitakse `processing="radiance"` või `"reflectance"` — lairibaanduri puhul on Bayeri-põhine kiirgusvõimsus mõttetu, seega jätab tagapõhi need kaamerad vahele, selle asemel et toota mõttetut tulemust.

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

Põhjuse märgid järgivad mustrit `<level>-not-applicable-to-rgb-cam` (üks kanne iga vahelejäetud taseme kohta, millest igaühel on `level`). Peegeldusele spetsiifilised vahelejätmised on `reflectance-skipped-no-fresh-dls` (puudub värske allapoole suunatud mõõtmistulemus ), `reflectance-skipped-bound-daq-unavailable (…)` (seotud andmekogumisega ei õnnestunud ühendust saada) ja `dls-uncalibrated-band-<nm>` – lainepikkus jääb peamiselt väljapoole andmekogumise valgusanduri radiomeetriliselt kalibreeritud vahemikku (~374–974 nm), mistõttu DAQ-põhine absoluutne peegeldusvõime jagamine lükatakse tagasi ja kaader langetatakse selgelt anduri vastuse tasemele. Müügil olevatest toodetest põhjustab seda ainult F988; selle kaamera toetatud töövoog on peegeldusvõime-paneeli töövoog.

`processing`-tasemed:

| Tase | Väljund |
| --- | --- |
| `"raw"` | Ühekanaliline Bayer (mustvalged kaamerad: ühekanaliline) otse andurist. |
| `"debayered"` *(SDK vaikimisi)* | 3-kanaliline BGR bilineaarse demosaikimise kaudu (mustvalged kaamerad: 1-kanaliline halltoon). |
| `"radiance"` | float32 W/m²/sr/nm täieliku radiomeetrilise ahela kaudu. Ainult multispektraalne — RGB kaamerad jäetakse vahele. |
| `"reflectance"` | uint16 0..32768 (Pix4D-valmis); nõuab reaalajas andmekogumise ühendust absoluutse võrdluspunktina. Ainult multispektraalne. |
| `"display"` | Täielik ahel, mis vastab kasutajaliidese eelvaatele (CCM + WB + gamma vastavalt kaamera profiilile). |
| `"all"` | **Üks fail iga asjakohase taseme kohta** iga kaamera kohta (vastab GUI-le „Capture All“ / CLI vaikimisi seadistusele). Tagastatud `CaptureResult` sisaldab seejärel ühte kaadri sõnastikku iga `(cam, level)` kohta, kusjuures iga sõnastikus on tasand; mittekohaldatavad tasemed kuvatakse `.skipped`-is. Iga peegeldusraami jaoks kasutatud DAQ-näit salvestatakse `.daq`-i lisafailina. |

> **Märkus — vaikimisi seade erineb CLI-ist.** `ArraySession.capture()`-i vaikimisi väärtus on `processing="debayered"`; `chloros-cli lattice array-capture`-i käsu vaikimisi väärtus on `processing="all"`. Edastage `processing="all"` selgesõnaliselt SDK-ist, et peegeldada CLI/GUI mitmetasandilist salvestamist.

### Salvestusrežiimid ja salvestid

Massiivi pind peegeldab GUI salvestuspaneeli: ühekordne / pidev / intervall / kiireim katikurežiim, lisaks kaks salvestajat (reaalajas komposiitvideo ja töötlemata pildiseeria → offline-taasesitamine).

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

- **`capture_repeated`**on SDK-i pidev/intervall-tsükkel. Kunapole `Ctrl+C`-i, millega seda skriptist katkestada,**peate** edastama `count` ja/või `duration_s` (tsükkel peatub, kui jõutakse kummagi väärtuseni). `interval_s` mõõdetakse iga läbimise algusest (vastavalt kasutajaliidesele). Ülejäänud kwargid edastatakse otse `capture()`-ile.
- **`record`** on *jälgimiskvaliteediga*: see salvestab ekraanil kuvatava reaalajas kombineeritud indeksi komposiidi, seega peab kombineeritud voog olema avatud, et kaadrid jõuaksid sinna. Üks komposiitsalvesti iga massiivi kohta (tekitab vea, kui üks juba töötab).
- **`burst` → `build_video`** on *analüüsitaseme*: `burst` kirjutab toorkaadrid + igakaadri manifesti + ühe `.daq` iga eraldi DLS-lugemise kohta `<output>/bursts/<base>/` all salvestusringi täiskiirusel (ilma ahelata, ilma exiftoolita, ilma reaalajas vaatamiseta). `build_video` sobitab iga kaadri aja järgi lähima `.daq`-iga ja käivitab uuesti impordipipeliini kiirguse/peegelduse/indeksi ahela. `products` on nimekiri `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`-idest (vaikimisi: kombineeritud indeks). `burst().stop()` käivitab automaatselt ka parima võimaliku kombineeritud indeksi koostamise, mille tulemusena tagastatakse `build_job` lõpetamistulemusena.

#### `RecorderHandle`

Tagastatakse `ArraySession.record()` ja `ArraySession.burst()` poolt. Kasuta seda kontekstihaldurina, et lõpetada töö automaatselt ulatuse lõppemisel, või juhenda seda käsitsi.

| Liige | Kirjeldus |
| --- | --- |
| `job_id` | Tagapõhja töö ID (str). |
| `kind` | `"composite"` (pärit `record`-ist) või `"raw"` (pärit `burst`). |
| `start_stats` | `start`-kõne poolt tagastatud sõnastik. |
| `result` | `None` töötamise ajal; lõplik peatamistulemuse sõnastik pärast peatamist. |
| `stats(timeout=10.0)` | Töö käigus kuvatavad statistilised andmed (kirjutatud kaadrid, tegelik kaadrisagedus, kulunud aeg). |
| `stop(timeout=60.0)` | Peata salvestaja; tagastab ja salvestab vahemällu lõpptulemuse. Idempotentne (teine kutsumine tagastab vahemällu salvestatud tulemuse). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Ühendamine juba ühendatud massiiviga — `attach_array`

Kui massiiv on juba käivitatud (GUI avas selle või eelmine SDK-istung kutsus välja `connect_array`), kasuta selle asemel uuesti ühendamise asemel `attach_array`, et saada sellele käepide. `connect_array` annab selles olukorras alati vea „Kaamera  on<sn> juba massiivis <id>“, sest `/array/connect`-i POST-kõne massiivi liikme jaoks ei ole idempotentne; `attach_array` loeb `/api/camera/array/list`-i ja teeb vasteid kas array_id või seerianumbri alusel.

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

Muster: SDK skriptid, mis töötavad koos töölaua graafilise kasutajaliidesega, peaksid kõigepealt proovima `attach_array` ja kui massiivis pole veel ühtegi massiivi, siis kasutama `connect_array`.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Oluline — context-manageri väljumine katkestab ühenduse.**`ArraySession.disconnect()` saadab alati POST-päringu `/array/disconnect`-ile; siin puudub „attached-not-owned” kaitse, nagu on olemas `CameraSession` / `DAQSensorSession` puhul. Kui tekasutad GUI-d ja ei soovi massiivi ulatuse lõpetamisel maha võtta,**ära kasuta `with`-plokki** — hoia käepide tavalises muutujas ja jäta selgesõnaline `disconnect()` vahele:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Võrguanalüüsi abivahend

Kasulik enne massiivi avamist — ennustab, kas sinu kavandatud seaded sobivad:

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

`status` on üks järgmistest: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (muidu `error`). `auto_capped_fps` tähendab, et soovitud resolutsioon sobib RX-ringiga ainult piiratud triggereerimissageduse juures — säilita resolutsioon ja mine edasi `target_fps=result["recommended"]["recommended_target_fps"]` edasi `connect_array`-iks (vt [Näide 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Kuidas projektsiooni lugeda** (sama mudel nagu GUI-paneelis „Array Settings”):

- **Pildiseeria (`frame_bytes_total`) summeeritakse iga kaamera puhul vastavalt kaamera tegelikule pikseliformaadile.**Mono**M3M**kaamerad edastavad Mono12 (2 B/px) sõltumata sellest, millise `pixel_format` väärtuse edastad, seega on 4-kaameralise täisresolutsiooniga kaadri suurus**~25 MB** suurune, mitte ~12,6 MB, nagu eeldaks täielikult 8-bitine mudel. Tagapõhi tuvastab iga kaamera formaadi selle mudeli järgi.
- **Admittance (`burst_fits_nic_ring`) arvestab andmevoo läbilaskevõimet**, mitte kogu-burst-vs-ring: sim-emit sobib, kui host tühjendab RX-ringi kiiremini, kui kaamerad seda täidavad. 10G host + 1 GbE kaamerad**võimaldavad** täisresolutsiooni isegi siis, kui burst ületab ringi mahtu; 1 GbE host blokeerib (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` on konservatiivne järjestikuse andmete väljavõtmise ülempiir** — `max(readout+emit, N×emit)`, kus kaamera kohta edastatav andmehulk on piiratud 1 GbE kaameraliiniga, sõltumata ekspositsioonist. Näiteks ~2,8 kaadrit sekundis 4-kaameralise täisresolutsioonilise 12-bitise massiivi puhul (vastab käitamisaja mõõdetud ~2,7–3,0). Täielik mudel: [CLI Viide → Maatriksi kaadrisageduse ja sarivõtte mudel](cli-reference.md#array-fps--burst-model).
- **Ülekoormus (`oversubscribed: true`) tähendab, et N × kaamera kohta kehtiv miinimum ületab kokkupõrkeohutu maksimumi** — kaadrisageduse väljad (`achievable_fps_max` / `fps_bright` / `fps_dark`) näitavad väärtust 0 ning automaatne-kahanemine/binning ei suuda seda parandada (need vähendavad baitide arvu kaadri kohta, mitte sekundi kohta edastatavate baitide arvu). Lahendused on kaamerate arvu vähendamine, jumbo-kaadrid või kiirem võrgukaart; `max_cams_collision_safe` näitab ülempiiri (6 täisresolutsiooniga kaamerat 1 GbE-võrgus @ 1500 MTU, 9 jumbo-raamidega). Vastuses on ka `aggregate_demand_bps`, `collision_safe_ceiling_bps` ja `per_cam_floor_bps` (8 MB/s). Vaata [Ülemäärane kasutamine](#over-subscription-the-per-cam-floor).

### Avastamine ja loetlemine

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

LATTICE-massiivid käivitavad taustal pideva automaatse ekspositsiooni (AE) niipea, kui nad on ühendatud, kuid äsja suunatud stseeni puhul kulub konvergeerumiseks hetk aega. **Smart-Capture** on mugavuslahendus: see küsib iga kaamera säriaega, ootab, kuni massiiv on kogu akna ulatuses stabiilne, ja käivitab seejärel pildistamise. See on GUI-ekvivalent: töölaua rakenduse „smart” pildistamisnupp kutsub välja sama tagapõhja-lõpppunkti.

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

Kui kasutate `ChlorosProject`-i (järgmine jaotis), saate kasutada rohkem seadistusi:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Smart-AE-režiim on vaikimisi konservatiivne. Karmistage `exposure_tolerance_pct`-i nõudlike radiomeetriliste tööde jaoks; leevendage seda kiiresti muutuvate stseenide puhul, kus piisab „piisavalt lähedasest“ tulemusest.

---

## DAQ-andurite seansid

Püsiv tagapõhja-pool spektraalanduritele (DAQ-U USB-liidese kaudu, DAQ-M BLE kaudu, DAQ-E Etherneti kaudu). Peegeldab kaamera pinda: nutikas tuvastamine, pooli taaskasutamine, idempotentne ühendamine.

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

Prioriteet: Ethernet → BLE → USB. Edasta ükskõik milline selgesõnaline vihje, et fikseerida andmeedastusviis.

### Fikseeritud andmeedastusviis

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
| `status(timeout=10.0)` | Pooli kande kokkuvõte (voogedastus-/salvestusolek, lainepikkuse vahemik, kalibreerimise SHA, integratsiooniaeg, frame_avg, AE-olek). |
| `latest(n=1, timeout=10.0)` | Tagastab kuni N viimast spektriraami. |
| `stream_start()` / `stream_stop()` | Jätkab / peatab voogedastust (käepide jääb avatuks). |
| `record_start(output_dir=None, device_name=None)` | Alustab .daq-faili salvestamist. Tagastab faili asukoha. Keeldub DAQ-U/M-i puhul, millel puudub AWS-kalibreerimispakett (DAQ-E on erand). |
| `record_stop()` | Salvestamise lõpetamine. Tagastab `{path, rows}`. |
| `disconnect()` | Vabastamine reservist. Ei tee midagi ühendatud-mitte-omanduses olevate käepidemete puhul ei toimu mingit operatsiooni. |

> **Kap-korrektsiooniprofiilid (`cap_id`) ei ole SDK-i reguleerimisvõimalus.** `connect_daq_sensor()` / `DAQSensorSession` ei avalda ühtegi `cap_id` parameetrit ega `set_cap` meetodit. Valige laevastiku piirangute korrigeerimisprofiil CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) või tagapõhja `/api/daq` ja HTTP marsruutide kaudu (`/api/daq/connect` ja `/api/daq/<id>/cap-id` aktsepteerivad `cap_id`).

### Avastamine — ühendamiseks vajaliku aadressi leidmine

`discover_daq_sensors()` otsib USB-st / BLE-st / ETH-st andureid, mida *võiks* avada. See on DAQ-i vastand `discover_lattice_cameras()`-ile ja ainus viis saada **DAQ-M-i BLE MAC-aadressi** — DAQ-E-l on hostinimi ja DAQ-U-l COM-port, kuid MAC-aadressi ei ole seadmele trükitud ega ole see operatsioonisüsteemis loetletud.

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
| `address` | COM-port / BLE MAC / hostinimi — edastatakse `connect_daq_sensor`-ile kui `port=` / `mac=` / `eth_host=`. |
| `display` | Inimloetav silt. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` või `None`, kui skanner ei suuda porti tuvastada (USB-jadaportiadaptereid ei ole ilma sondita võimalik eristada, seega kuvatakse tundmatud seadmed, mitte ei peidetud). |
| `extra` | Transpordiviisi kaupa üksikasjad (BLE-i reklaamitud nimi, USB-tootja, DAQ-E IP/fw/…). Tühjad väärtused jäetakse välja. |

| Parameeter | Vaikimisi | Kirjeldus |
| --- | --- | --- |
| `transports` | kõik kolm | Skaneerimist piirav jada (või CSV-string). Tasub edastada, kui teate, mida soovite — BLE on aeglane osa. |
| `scan_timeout` | 5 | Skannimisaken transpordiliigi kohta sekundites; tagapõhi piirab väärtuse vahemikku 1–20. |
| `timeout` | 60,0 | HTTP kogu kõne ülempiir (nagu ka mujal SDK-is). |
| `auto_start_backend` | `True` | Käivita kohalik backend, kui ükski ei ole käimas. Kaug-backendit ei käivita kunagi. `backend_url`. |

> **Poolis juba avatud andurid ei kuvata.** Ühendatud BLE-perifeeria lõpetab reklaamimise ja avatud COM-porti ei saa uurida, seega loetleb avastamine seda, mis on *ühendamiseks saadaval*. Kohe pärast seadme ühendamist on oodata tühja tulemust — kasuta `list_daq_sensors()` selle jaoks, mis sul juba olemas on. Transpordid, mille skaneerimine ei saa töötada (bleak / zeroconf pole installitud), jäetakse vahele, mitte ei tekitata viga, seega saab seade, millel pole Bluetoothi, ikkagi oma USB- ja ETH-vastused.

### Loend

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Ühiskasutus GUI-ga / CLI

Kui GUI-s on juba andur avatud, siis Python-ist `connect_daq_sensor(port="COM3")`-i kutsumine tagastab käepideme, mis on märgitud `already_connected=True`. Seansi `disconnect()` on seejärel tühitoiming, nii et teie skript SDK ei eemalda andurit GUI alt, kui skopeerimisaknast väljutakse.

### Otsesed riistvaraklassid (ilma tagapõhja)

`daq_sdk` eksporditakse uuesti funktsiooni `chloros_sdk` poolt, nii et saate andureid juhtida ka protsessi sees otsest lõpppunkti, ilma tagapõhja abita:

> **Kättesaadavus:**`daq_sdk` on kaasas Chloros töölaua installiga,**mitte** koos PyPI-paketiga — `pip install chloros-sdk` pakub teile `lattice_sdk`-i, kuid jätab välja `chloros_sdk.DAQ_AVAILABLE == False`-i. Kontrollige seda lipukest enne nende klasside kasutamist; ainult pip-i kasutaval host-kõvakettal kasutage selle asemel andurit [`connect_daq_sensor()`](#daq-sensor-sessions) kaudu, mis ei vaja kohalikke transpordiraamatukogusid.

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

Salvestatud Chloros projekt on kaust, mis sisaldab `cameras.json` + `sensors.json` + `project.json`. `open_project` laadib manifesti ja `connect_all` lülitab kõik salvestatud seadmed salvestatud seadistustega võrku – sama riistvaraseisund, mille graafiline kasutajaliides tekitaks.

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

### `ChlorosProject`-i meetodid

| Meetod | Kirjeldus |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Leiab ja ühendab kõik salvestatud seadmed. Tagastab klassipõhise ühendusaruande. Kasutab töötavat tagapõhja, kui see kuulab `127.0.0.1:5000`-i; muidu lülitub vaikselt üle otsesele (tagapõhja-vaba) `lattice_sdk` seadmejuhtimisele — see ei käivita kunagi tagapõhja. |
| `disconnect_all()` | Lõpetab kõik ühendused. |
| `capture_all(output_dir=".")` | Üks raam iga kaamerast + massiiv + spektrum igast andurist. |
| `stream(camera, overlays=False, fps=10.0)` | Generaator, mis toodab BGR-kaadreid nimetatud kaamerast (või massiivist). `overlays=False` on otsene `lattice_sdk` kaadrite kogumise tsükkel (massiivid annavad `{serial: frame}` sõnastikud). `overlays=True` suunab läbi `ChlorosLocal.camera_stream()` → tagapõhja `/api/camera/<serial>/stream-annotated` MJPEG-voo, kusjuures kaamera salvestatud `ui.overlay`-plokk edastatakse päringuparameetritena. Nõuab tagapõhja režiimi ja **iseseisvat kaamerat**: otserežiimis kaamera tekitab veatüübi `RuntimeError` (tagapõhi ei saa haarata kaamerat, mis kuulub sellele protsessile) ja massiiv tekitab veatüübi `NotImplementedError` (kaamerate kaupa komposiitülekanne — voogedastab liikme nime järgi). Ühekordse käivituse ekvivalent: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Käivita joondamine igal hetkel ühendatud massiivil. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Käivita kalibreerimis- ja indekseerimispipeline projekti piltide puhul (hõlmab `ChlorosLocal.process`; need neli on **ainukesed** aktsepteeritavad kwargid — `indices=` jne. tekitavad veatüübi `TypeError`; indeksid määratakse `ChlorosLocal.configure()` kaudu). Loob viivitusega `ChlorosLocal()`, mis käivitab automaatselt tagapõhja. |

Atribuudid:
- `proj.cameras` — `Dict[str, CameraHandle]`, mille võtmeks on nimi JA seerianumber.
- `proj.arrays` — `Dict[str, ArrayHandle]`, mille võtmesõnadeks on nimi JA array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, mille võtmesõnadeks on nimi JA slot_id.
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
tokeni ning ahel on kumulatiivne — iga tase käivitab kõik sellest kõrgemal asuvad tasemed:

| Tase | Väljund | Märkused |
| --- | --- | --- |
| `raw` | 1-kanaliline Bayer, anduri omapärane | Demosaikimist ei toimu. Selles tasemes pole ülekatteid saadaval. |
| `debayered` | 3-kanaliline BGR (**vaikimisi**) | Billineaarne demosaik. Ainus tase, mis töötab ilma backend-režiimita. |
| `radiance` | float32, W/m²/sr/nm | Täielik radiomeetriline ahel: demosaik + 3×3 segunemise eemaldamine (multispektraalne) + DSNU + tasandus + NIST-skaala, kus ekspositsioon × võimendus on välja jagatud, nii et väärtused on absoluutsed. |
| `reflectance` | uint16, 32768 = 1,0 | Kiirgusjõud jagatud allapoole suunatud kiirgusintensiivsusega (ρ = π·L/E). Vajab DLS/DAQ-näitu — vt allpool olevat märkust. |
| `display` | 8-bitine sRGB-laadne | GUI-ekvivalentne renderdamine: CCM + valgebalanss + gamma kaamera aktiivse värviprofiili kaudu. |

Kõik muud kui `debayered` nõuavad backend-režiimi; otserežiimis kaamera tekitab
`NotImplementedError`. `reflectance` vajab kasutatavat allapoole suunatud mõõtmistulemust — raami lõpp-punkt tõmbab
koondatud DAQ automaatselt kaamera DLS-pesasse, kuid kui DAQ-d pole seotud, keeldub ahel
peegeldusvõime väljundist ja märgib alandamise ausalt tagastatud metaandmetesse, selle asemel et vaikselt
tagastada madalama kvaliteediga tulemust.

> **Peegeldusvõime DN-skaala – ära koodi seda kõvasti sisse.** LATTICE peegeldusvõime kasutab `32768` = ρ 1,0 ja märgib
> XMP `Chloros:PixelScale=32768`; Survey3 peegeldusvõime kasutab `65535` = ρ 1,0 ja ei sisalda
> `Chloros:*`-märgiseid. Loe märgist ja jaga sellega. See on defineeritud uint16-domeenis, seega jääb see
> `32768`-iks igas formaadis, mis muudab skaalat (16-bitine TIFF, 8-bitine PNG/JPG, 32-bitine protsent) — normaliseeri
> salvestatud andmetüüp esmalt tagasi uint16-ks (×257 8-bitisest, ×65535 ujukomast). Üks erand:
> 8-bitineallikast pärit salvestus, mis on kirjutatud 8-bitisena TIFF, *lõigatakse ära*, mitte ei skaalata ümber, seega ei kirjeldata seda ühegi skaalaga
> seda — Chloros jätab sel juhul `PixelScale` ja MicaSense&#x27;i tupli täielikult välja. Käsitle puuduvat
> märget LATTICE peegeldusfailis kui „kehtivat skaalat pole“, mitte kui vaikimisi väärtust.

> **EXIF-andmed kanti ekspordisse üle.** `process()` kopeerib iga toote puhul allika pildistamise GPS-ploki
> **ja selle ExifIFD** igale tootele, seega sisaldavad ekspordid ka `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` ja `CameraSerialNumber`, samuti
> georeferentsi. `FocalLength` on see, mille alusel Pix4D arvutab maapinna proovide vahekauguse – ilma selleta
> langeb rekonstruktsioon tagasi äärmiselt valesse mõõtkavasse (ühel mõõdetud juhul muutus 411 m suurune ala
> 47,8 km suuruseks). Koopia ei ole tahtlikult `-all:all`: IFD0 struktuurimärgised rikuvad
> LATTICE väljundit ning `ExifImageWidth`/`Height` on välja jäetud, kuna need kirjeldavad allika
> pildistamist, mitte eksporditud rasterit.

Salvestusetapi alamlipud (kehtivad radiomeetriliste tasemete kohta — `radiance`, `reflectance`, `display`):

| Lipu | Vaikimisi | Tähendus |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + tasandus + 3x3 segunemise eemaldamine + NIST-i radiomeetriline skaala. |
| `apply_white_balance` | `True` | WB LUT. DLS-tundlik, kui DAQ on kaameraga seotud. |
| `apply_index` | `False` | Taimestikuindeksi hindamine. |
| `index_expression` | `None` | Ülekirjutamisvalem. Kui väärtus pole tühi, lülitatakse indeks automaatselt sisse. |
| `annotated` | `False` | GUI-dekoratsioonide (zebra/võrk/piigid) pealekandmine. Ei ole saadaval `raw` jaoks. |

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

> **Tagastatav tüüp on `CapturePathMap`, mitte `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` on `Dict[str, Union[str, List[str]]]`: ühetasandiline
> `processing` annab igale seerianumbrile ühe tee, samas kui mitmetasandiline (`"all"` või
> selgesõnaline `levels` nimekiri) annab selle **järjestatud nimekirja** kõigist selle selle
> kaamera jaoks salvestatud toodete **järjestatud nimekirja**. Reaalajas kombineeritud komposiit, kui seda voogedastati, saabub lisavõtme
> `"combined"` all, mitte seerianumbri all. Kood, mis eeldab `str`-i, annab vea
> nimekirja vormis, ilma et tüübikontrollija sellele vastu oleks – märge näitas `Dict[str, str]`
> veel mõnda aega pärast nimekirja vormi väljalaskmist, mistõttu see alias eksisteeribki. Normaliseeri
> kui soovid lamevormi:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Massiivi joondamine

`ArrayHandle` avab kogu joondamispinna. Profiilid on vaikimisi seansipõhised — säilitamiseks kutsu selgesõnaliselt välja `export_alignment()`.

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

`connect_all(align=...)` suudab iga massiivi ühendamisel automaatselt joondada:

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

Kui seda ei ole määratud, kasutatakse tagavaravarianti `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Otsene riistvara (tagapõhja vaba)

Kui soovite täielikku sõltumatust tagapõhjast (CI, peata robotid, sisseehitatud), impordi otse `lattice_sdk` ja `daq_sdk` — mõlemad eksporditakse uuesti `chloros_sdk` kaudu. Kaitsemehhanism `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` on PyPI-paketis (kuid vajab Arena SDK-i töökeskkonna olemasolu), samas kui `daq_sdk` on saadaval ainult töölaua versiooniga.

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

##### Eelseadistused ja käivitaja

Neljast eelseadistusest kolm on **vabajooks**: kaamera pildistab pidevalt ja
`capture()` tagastab järgmise kaadri. `triggered` on erand — see aktiveerib
kaamera 2. liini riistvaralisele servale, seega ei jäädvusta see midagi enne, kui see saabub.

| Eelseadistus | Käivitaja | Kasutada, kui |
| --- | --- | --- |
| `default` | vabajooks | üldine kasutus |
| `high_speed` | vabajooks | 8-bitine, maksimaalselt 60 kaadrit sekundis, lühike eksponeerimisaeg |
| `high_quality` | vabajooks | 12-bitine, kaadrisageduse piiranguta — tavaline valik fotode jaoks |
| `triggered` | **valmis, liin 2** | kaamera on ühendatud M8 sünkroniseerimiskaabliga ja midagi muud käivitab selle |

Kui valite `triggered` (või seadistad ise `trigger_mode="On"`), kusjuures midagi
ei juhita liini 2 kaudu, siis iga `capture()` aegub — õigustatult, kuna sa palusid
kaameral oodata. SDK selgitab seda, kui see juhtub; vaata
[SC_ERR_TIMEOUT salvestamise ajal](#direct-hardware-backend-free).

> **Märkus — „GVSP probe” / `SC_ERR_TIMEOUT -1011`-sõnumid ühenduse loomisel ei ole vead.**&gt; Ühenduse loomisel püüab SDK läbirääkimiste teel saavutada**jumbo-raame** (9000-baidised GVSP-paketid) suurema läbilaskevõime saavutamiseks. Otsese punkt-punkt-võrgukaardi ühenduse korral (nt-kohalik `169.254.x.x`-aadress) ei suuda võrk tavaliselt jumbo-raame edastada, mistõttu see proov aegub ja logib järgmisi ridu:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> See on **ette nähtud tagavaralahendus**: SDK naaseb automaatselt standardse 1500-baidise paketi kasutamisele ja kaamera jätkab ühenduse loomist tavapäraselt (järgnevad `[chunk-enable …]`-read on osa tavalisest ühendusjärjestusest). Salvestamine toimib endiselt.
>
> Saate selle proovi vahele jätta, kuid **see ei ole lihtsalt logide summutaja – see lülitab jumbo-raamid välja.** Kaamera vastab „Don&#x27;t-Fragment pingidele vastab vaid kuni 1500 baiti pikkustele pakettidele, olenemata sellest, kui hea on teie võrk, seega ping-test üksi ei suuda kunagi jumbo-raame tuvastada; see proov on ainus, mis seda suudab. Lülitage see välja ja kaamera kasutab igas võrgus igavesti standardseid 1500-baitiseid pakette:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> See tasub end ära ainult võrgus, mille kohta te *tead*, et see ei toeta jumbo-pakette, kus see säästab ühenduse loomisel umbes ühe sekundi kaamera kohta. Kuna tegemist on reaalsete, mitte kosmeetiliste muudatustega, näitab SDK seda nüüd ka kasutamisel:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Jätke see rahule, kui teil pole selleks põhjust.** Kui see on sisse lülitatud, mõõdab iga ühendus uuesti teie tegelikku võrku: ühendage kaamera jumbo-pakette toetavasse lülitisse ja järgmine ühendus võtab jumbo-pakettide toetuse ise kasutusele, ilma et midagi tuleks konfigureerida ega süsteemi taaskäivitada.
>
> Kui *soovid* jumbo-andmeedastuskiirust, lülita sisse läbiv jumbo-režiim (võrgukaardi MTU 9000 + lüliti, mis neid edastab)või fikseeri see parameetriga `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, kui tead, et ühendus seda toetab – kuigi soovitame kasutada iga käsu jaoks eraldi parameetrit `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …`, mitte seda püsivalt seadistada, kuna fikseeritud suurus jätab proovi vahele ja lõpetab kohanemise eesoleva võrguga. **Iga** seadme teel peab jumbo-pakette edastama — sealhulgas kõik PoE-jagajad või -injektorid, mis on tavaline põhjus, miks muidu jumbo-pakette toetav seadistus neid edastada ei suuda.

> **`SC_ERR_TIMEOUT -1011` ajal, kui `capture()` / `grab*()` on tegemist teistsuguse probleemiga – see on tegelik viga.**&gt; Eespool toodud märkus puudutab ainult**connect-time probe**poolt logitud `-1011`-i. Sama viga, mis tekkis**capture** käigus, tähendab, et kaamera ühendus õnnestus, kuid see ei saada ühtegi pilti:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Selle tunnuseks on kaamera, mille *juhtimiskanal* on korras – avastamine toimib, seadistused ja `[chunk-enable …]`-kirjutused õnnestuvad kõik –, kuid *iga* kaadri puhul aegub ooteaeg.
>
> **Tavaliselt on põhjuseks see, et kaamera on seadistatud riistvaralisele käivitajale.** `trigger_mode="On"` ja `trigger_source="Line2"` korral ei saada kaamera midagi, kuni M8 sünkroniseerimiskaablile jõuab elektriline signaal. Kui sellel liinil pole ühtegi kaablit, ootab iga kaadri salvestamine lõputult. Kaamera ei ole rikkis ja võrk toimib korralikult – see teeb täpselt seda, mida talle on kästud.
>
> `CameraSettings()` ning `default` / `high_speed` / `high_quality` eelseadistused töötavad vabajooksurežiimis ning ajastatud pildivõtt, mis aegub aktiveeritud olekus, annab selgituse, selle asemel et kuvada lihtsalt `-1011`. `PRESETS["triggered"]` aktiveerib Line2, nagu ette nähtud.
>
> Mis tahes kaamera vabajooksu sundimiseks:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Kui ajapiirang ikkagi lõpeb `trigger_mode="Off"`-i puhul, siis kaamera tõepoolest andmeid ei edasta — saatke meile logi ja `ip link show`.

#### Värviprofiilid (RGB reaalajas eelvaade) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` valib ekraani värviprofiili **reaalajas eelvaate** jaoks kaameratel RGB kaamerate puhul (multispektraalsed kaamerad ignoreerivad seda seadet):

| Profiil | Tähendus |
| --- | --- |
| `raw` | Jäta radiomeetriline ahel täielikult vahele. |
| `linear` | DSNU + flat + valgebalanss, ilma CCM-ita, ilma gammata. |
| `natural` | Lineaarne + mõõdetud CCM + sRGB-gamma, ainult odava viimistlusega (kroma silumine + heledate alade desaturatsioon) — realistlik vaikimisi seade. |
| `enhanced` | `natural` pluss täielik hub-parity viimistlus (defringe, vibrance, CLAHE kohalik kontrast). Rikkam välimus, kuid kaadri kohta ligikaudu **kaks korda suurem viimistluskulu**, seega madalam reaalajas kaadrisagedus. |
| `custom_temp` | `natural`, kuid valgebalanss fikseeritud `custom_cct_k` Kelvinile (DLS-i ignoreeritakse; tagapoolel piiratud vahemikuga 2000–10000 K). |

Profiil on **ainult reaalajas eelvaateks mõeldud** kiiruse/välimuse reguleerimisnupp: salvestatud kaadrid saavad alati täieliku rikkaliku viimistluse, sõltumata valitud profiilist, seega `natural` valimine kaadriaja kokkuhoiuks ei vähenda kettale salvestatava materjali kvaliteeti. Tundmatu profiil tõstab `ValueError` väärtust; kui chloros-tagapõhi on kättesaadav, saadetakse muudatus ka sinna POST-päringu kaudu, nii et järgmine eelvaatekaader peegeldab seda (otsese SDK kasutajad, kellel tagapõhja pole, saavad seadete muutuse ikkagi kätte).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M) kaamerad ja `Calibration`

Mono **M3M**-kaamera (`M3M-<lens>-F<wavelength>`) on ühe lainepikkusega: üks halltoonide tasand, ilma Bayeri mosaiigita, ilma 3×3 spektraalse ristkõne maatriksita. `Calibration` tuvastab selle ja kuvab lipu `is_mono`. Peegeldus kehtib endiselt ribapõhise radiomeetrilise kaardina (lahutamine on identiteetmaatriks), kuid ühe kaamera puhul mitmeribaline matemaatika annab pigem tulemuse kui mõttetut väljundit:

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

Et luua taimestiku indeks ühekanalilise riistvara abil, kombineerige mitu erinevatel lainepikkustel töötavat M3M-kaamerat joondatud mitmekanalilisse pildistakki (vt [Array Alignment](#array-alignment)) ja arvutage indeks kogu selle pildistiku põhjal, mitte ühe kaamera põhjal.

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

> **`apply_sensor_settings` aktsepteeritud võtmed**— täpselt `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; kasutuselt kõrvaldatud, asendatud `cap_id`), `filter_model` (DAQ-M) ja `cap_id` (kõik DAQ-tüübid; `None`/`""`/`"none"` = paljas andur, ilma kondensaatori korrigeerimiseta). Tundmatuid võtmeid**ignoreeritakse märkamatult** – nt `{"integration_time": 64}` ei tee midagi (peab olema `integration_time_ms`). Tagastab `{"applied": [...], "errors": {...}}` ja ei tekita kunagi viga.

`chloros_sdk` ekspordib uuesti ainult eespool kasutatud põhipinda. Täielik `daq_sdk` avalik API (22 nime) lisab järgmise — importige need otse `daq_sdk`-ist:

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

Püüa kinni baasklass, et käsitleda „kõiki Chloros-iga seotud vigu”:

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

> `ChlorosAuthenticationError` ja `ChlorosConfigurationError` eksporditakse koos ülejäänutega tipptasandil; neid saab importida ka `chloros_sdk.exceptions`-ist, nagu näidatud.

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

### 4. Mitme kaamera kaadrivoog → NumPy töötlusketi

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

### 5. Peata otsene riistvarapõhine (ilma tagapõhja) salvestamisskript

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

CLI-i retsepti DSL-il on otsene Python-i ekvivalent:

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

Smart-Connecti sisenemispunktid — `connect_camera`, `connect_array`, `connect_daq_sensor` ja `discover_lattice_cameras` — on kerged HTTP-kliendid, mis eeldavad, et backend kuulab aadressil `127.0.0.1:5000` (smart-connect-liidese vaikimisi URL). Kui graafiline kasutajaliides või CLI juba töötab, siis üks neist töötab. Lihtsa skripti puhul võib see aga puududa — seega käivitavad need funktsioonid **automaatseltkäivitavad komplektis oleva tagapõhja binaarfaili** (aknata, samamoodi nagu seda teeb `ChlorosLocal`) enne esimest kutsumist ja ootavad seejärel kuni `backend_startup_timeout`, kuni see käivitub.

Reeglid:

- **Ainult kohalik URL käivitatakse.** Sobiv on `backend_url`, mis viitab `localhost`-ile / `127.0.0.1`-ile / `[::1]`-ile; mis tahes muu host eeldatakse olevat kellegi teisearvutiks ja seda ei käivitatagi kunagi.
- **Tagapõhi jäetakse taaskasutamiseks töötama** (nagu CLI puhul) — skripti lõpetamisel ei toimu automaatset sulgemist. Kui-skripti käivitamisel kasutatakse uuesti juba töötavat backendit.
- **Loobuge sellest `auto_start_backend=False` abil** mis tahes nimetatud kutsumise korral (nt kui olete suunanud kaugbackendile või haldate backendi elutsüklit ise).

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

Kui komplektis olevat binaarfaili ei õnnestu leida või käivitada, tekitab järgnev HTTP-kõne tegevusele kutsuva, **platvormist sõltuva** `ChlorosConnectError`-i, mitte lihtsalt ühenduse keeldumise jälje – Windows juhib teid töölaua rakenduse või `chloros-cli`-käsu juurde; Linux puhul (ilma graafilise kasutajaliideseta) suunab see sind `chloros-cli`-käskluse või `.deb` juurde.

---

## Keskkond ja päised

SDK märgistab iga tagapõhja HTTP-kõne `X-Chloros-Client: sdk`-ga. Tagapõhi rakendab SDK/CLI litsentsimiseeskirju (vajalik on sisselogimine **ja** nõutav on tasuline Chloros+ pakett), mitte GUI tasuta taseme teed. See seatakse automaatselt importimise ajal — te ei pea midagi tegema.

`http://localhost` ja `http://127.0.0.1` tuvastatakse kohaliku tagapõhjana. Kõned teistele hostidele (nt teie enda analüütikateenus) jäävad muutmata.

Tagapõhja URL saab ümber kirjutada, edastades `backend_url=` (või `api_url=`, kui kasutatakse `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Mitte-loopback-tüüpi `backend_url` jõuab ainult allika/seadme backendini — kaasasolevad backendid seovad ainult loopbacki; tunneli mustri kohta vaata Remote-Backend Mode.)

---

## Versioonid ja ühilduvus

- Versioon SDK on kättesaadav versioonina `chloros_sdk.__version__`.
- SDK seob käitumise kaasasoleva tagapõhja versiooniga. Vanema SDK ja uuema tagapõhja kombineerimine toimib tavaliselt (edasiühilduvad lõpppunktid), kuid uuema SDK-i segamine vanema backendiga võib uutes lõpppunktides põhjustada `404`-vigu – uuendage töölaua rakendus vastavaks.
- Nutikas-connect-liides (`connect_camera` / `connect_array` / `connect_daq_sensor`) ja võrguanalüüsi lõpppunkt tagastavad stabiilseid JSON skeeme; uued väljad on täiendavad.

---

## Veaotsinguvihjed

- **`ChlorosAuthenticationError: Login required`** → Käivita `chloros-cli login EMAIL PASSWORD` sellel arvutil üks kord või logi sisse Chloros töölaua rakenduse kaudu.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-Connecti kutsed käivitavad automaatselt kohaliku tagapõhja, seega ilmub see teade ainult siis, kui kaasasolevat binaarfaili ei leita või seda ei saa käivitada (nt ainult pip-i kasutav host, millel puudub töölauarakendus). Teade sõltub platvormist: Windows-is avage töölaua rakendus või käivitage mis tahes `chloros-cli`-käsk; Linux-is käivitage `chloros-cli`-käsk (graafilist kasutajaliidest ei ole) või paigaldage `.deb`. Kaugbackendi puhul edastage `backend_url=` (ja `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** importimisel → `lattice_sdk` laadimine ebaõnnestus (tavaliselt pole Arena SDK-i käitamisajaga DLL-failid paigaldatud). Kaameraväline pind töötab endiselt.
- **Array connect tagastab sub-native resolutsiooni**→ Backendi smart-ettevalmistus vähendab kaadri suurust automaatselt, et see mahuks ühendusse. Kasutage `analyze_array_network()`, et näha põhjust, seejärel uuendage ühendust, aktsepteerige vähendamist või edastage `force_tier="slip-emit-and-capture"` järjestikuliseks salvestamiseks. Vähendamise turvavõrk**ei** kata koondülekoormust (`oversubscribed: true`, fps-väljad 0): liiga palju kaameraid ühendusele ei saa parandada piltide koondamise/ROI abil — vähenda kaamerate arvu, lülita sisse jumbo-raamid või vaheta kiirema võrgukaardi vastu (vaata [Ülemäärane kasutamine](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` teatab, et võrgukaardi vastuvõturing on väga väike (~0,26 MB) / ühendusväravad koos &quot;FRAMES WILL DROP&quot;** → Host-võrgukaardi vastuvõturõngas on vaikimisi seadistuses (sageli taastatakse väärtuseks 32 pärast võrgukaardi draiveri uuendamist). Realtek USB 10GbE adapteril seadistage `ReceiveBufferLen=256` ja `PendingReceives=64` (kõrgendatud) ning seejärel taaskäivitage tagapõhi, et see loeks ringi uuesti sisse. Täielik juhend: [CLI Viide → Host-NICi seadistamine ja häälestamine](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host hangub taaskäivitamisel/väljalülitamisel, hiljem ilmnevad WMI `Invalid class` vead / võrgukaart ei lülitu sisse** → Aegunud USB 10GbE draiver põhjustab `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Uuendage adapteri draiver viimase versioonini (≥ 2026) ja rakendage uuesti vastuvõturingi seaded. Vaadake [CLI Viide → Host-võrgukaardi seadistamine ja häälestamine](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Peegeldusvõime keeldus** → Absoluutse skaala peegeldusvõime saamiseks peab kaameraga (või andurite massiiviga) olema seotud aktiivne andmekogumissüsteem (DAQ). Seo see kas graafilise kasutajaliidese kaudu või kasuta `processing="radiance"` (W/m²/sr/nm), mis ei vaja paarisandurit.
- **`smart=True` andmete kogumine võtab oodatust kauem aega** → AE konvergents sõltub stseeni dünaamikast; kui soovite kiiremat (vähem stabiilset) päästikku.

---

## Vaata ka

- [CLI viide](cli-reference.md) — iga CLI alamkäsk peegeldab SDK-kõnet.
- [DAQ-andurite juhend](../daq/README.md) — andurispetsiifilised ühendus-, kalibreerimis- ja salvestamiseeskirjad.
- Veebidokumendid: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
