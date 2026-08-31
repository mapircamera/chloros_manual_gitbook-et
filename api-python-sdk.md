# API : Python SDK

{% hint style="info" %}
**Otsite täielikku API-i?** See lehekülg on praktiline juhend. Kõik avalikud klassid, meetodid, täpsed signatuurid ja kopeeritavad näited on [SDK viites](reference/sdk-reference.md), mis on optimeeritud AI-assistendile.**Kasutad AI-assistenti?** Kleebi see URL vestlusaknasse, et sul oleks kättesaadav täielik ja ajakohane Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Käesoleva juhendi iga lehekülg on kättesaadav toores Markdown-vormingus selle väiketähelise slug’i + `.md` all ning kogu juhend on indekseeritud aadressil `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` PyPI-s) juhib kõiki töölaua rakenduse funktsioone alates Python-st: piltide hulgitöötlus, LATTICE-kaamera ja maatriksi reaalajas juhtimine, DAQ-valgusandurite seansid ning salvestatud projektide automatiseerimine. Tegemist on õhukese kihiga, mis on ehitatud sama kohaliku tagapõhja peale, mida kasutavad ka graafiline kasutajaliides ja CLI (HTTP `127.0.0.1:5000`-is), seega on käitumine kõigil kolmel platvormil identne.

## Paigaldamine

Paigaldamine toimub kahes etapis: kõigepealt paigaldatakse Chloros töölauapakett (see pakub töötlemise taustsüsteemi ja riistvara käitamiskeskkondi), seejärel Python pakett.

**

1. samm — Paigalda Chloros.** Windows: käivita töölaua paigaldaja (vaikimisi asukoht `C:\Program Files\MAPIR\Chloros\`) lehelt [Allalaadimine](download.md). Linux: paigalda pakett `.deb` ([Linux paigaldamine](linux/linux-installation.md)).**

2. samm — Paigalda SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Võib-olla ei ole sul isegi pip-i vaja: iga paigaldaja sisaldab vastavat SDK-i wheel-i. Windows-i paigaldaja paigaldab selle automaatselt sinu süsteemi Python; Linux `.deb` paigutab selle kataloogi `/usr/lib/chloros/sdk/` ja kuvab täpse käsu `pip install --user`. PyPI uuendatakse iga uue versiooni väljalaskmisel, seega vastab `pip install chloros-sdk` viimasele stabiilsele versioonile.

**

3. samm — Logi sisse üks kord iga arvuti puhul:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Autentimisandmed salvestatakse vahemällu `~/.chloros/` (mõlemal platvormil). Windows-is saad sisse logida ka töölaua rakenduse vahekaardil „Kasutaja<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">”. SDK nõuab tasulist Chloros+ paketti — vaata allpool [Litsentsinõuded](#license-requirement).

| Nõue | Üksikasjad |
| --- | --- |
| **Chloros on installitud** | Windows: töölaua installija; Linux: `.deb`-pakett (sisaldab tagapõhja binaarfaili) |
| **Python** | 3.7 või uuem (arendatud/testitud versioonil 3.10) |
| **Operatsioonisüsteem** | Windows 10/11 64-bitine, Ubuntu 22.04 LTS või uuem, või NVIDIA Jetson (JetPack 6) |
| **Litsents** | Aktiivne Chloros+ sisselogimine, mis tahes tasuline pakett (Copper või kõrgem) |

## 60-sekundiline võit

Ühe käsklusega luuakse projekt, imporditakse kaust, konfigureeritakse töötlemine ja käivitatakse töötlemisahel – tagapõhi käivitatakse automaatselt, kui see veel ei tööta:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Linux-is kasuta Linux-i teid: `/home/user/drone_images/flight001`. SDK töötab mõlemal platvormil ühtmoodi.)

Töötlete LATTICE’i salvestuskausta? Kasutage LATTICE-sõbralikku wrapperit — see rakendab õigeid vaikimisi seadeid (ei tuvasta paneeli sihtmärki, kasutab standardset debayerit):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — täielik töötlusprotsessi kontroll

Kõigi üherealise käskluse ületavate ülesannete puhul kasutage `ChlorosLocal`-i. See käivitab esmakordsel kasutamisel tagapõhja (`auto_start_backend=True`), loob ja konfigureerib projekte, jälgib töökäiku ning tagastab töötlemise järel kokkuvõtte.

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

{% hint style="info" %}
Jätke alles vaikimisi seadistus `http://127.0.0.1:5000`, selle asemel et asendada seda `localhost`-iga — käskus Windows `localhost` lahendub esmalt `::1`-iks ja kulub umbes 2 sekundit päringu kohta ainult IPv4-d toetava tagapõhja puhul.
{% endhint %}

Kasutage seda kontekstihaldurina, et tagada puhastamine:

```python
import chloros_sdk

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

`configure()` aktsepteerib järgmisi märksõnu: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` ja `custom_settings`. Peamised väärtused:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

LATTICE-spetsiifilised nupud (`input_level`, `radiometric_output`, `array_alignment*`-perekond) on koos täielike väärtustabelitega dokumenteeritud [SDK-viites](reference/sdk-reference.md#supported-values).

### Töö käigu jälgimine

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Tööjärgse kokkuvõtte lugemine — ja tühjade käivituste tuvastamine

Lõpetamisel lisab `process()` tagapõhja töötlemise kokkuvõtte failina `result["summary"]`. Iga kirje failis `summary["hints"]` on terviklik lause, mis selgitab kõiki märkimisväärseid asjaolusid — näiteks, miks jooks andis nullväljundi — ning iga vihje edastatakse uuesti failina Python `UserWarning`, mistõttu tühjad jooksud on isediagnoosivad isegi siis, kui sa sõnastikku kunagi ei vaata:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` ei teki, kui käivitus ei anna ühtegi pilti.** See on ainus koht, kus SDK ja CLI erinevad teadlikult: `chloros-cli process` käsitleb olukorda „tooteid taotleti, kuid ühtegi ei salvestatud” veana ja lõpetab töötamise nullist erineva tulemusega, samas kui SDK lõpetab töötamise normaalselt ja teatab sellest olukorrast `summary` kaudu / vihjete abil. Kui teie töötlemisahel peaks tühja käivituse korral peatuma, kontrollige seda ise — vaadake `summary`-i (või loendage projektikausta all olevaid faile), selle asemel et tugineda erandile.
{% endhint %}

## Smart Connect — reaalajas töötav riistvara

Kolm abiprogrammi avavad püsivad seansid tagapõhja riistvarapoolis – samas poolis, mida kasutab ka graafiline kasutajaliides, mistõttu skriptid SDK toimivad koos töölaua rakendusega, ilma et nad võitleksid jadaportide või võrgu ribalaiuse pärast. Kõik kolm käivitavad automaatselt kohaliku tagapõhja, kui ükski neist ei ole käivitatud.

### Üksik LATTICE-kaamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Sünkroniseeritud kaamerarühm — `connect_array`

`connect_array` on soovitatav alguspunkt mitme kaameraga süsteemidele. See käivitab sama smart-prep-voo kui graafiline kasutajaliides: võrguanalüüs, sünkroniseerimistaseme automaatne valik, PTP-aja sünkroniseerimine, pikseliformaadi valik iga kaamera jaoks, AE algväärtustamine ja GPIO-päästiku aktiveerimine. **Esimene seeria on juhtseade** (see käivitab riistvaralise päästikuimpulsi); ülejäänud on allseadmed.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Lisage `smart=True` mis tahes massiivi salvestusele, et oodata, kuni automaatne säri on kõigil kaameratel stabiliseerunud, enne käivitamist. Salvestusrežiimide (ühekordne / pidev / intervall / kiireim), salvestite, pildiseeriate salvestamise videoks ja massiivi joondamise kohta vaadake [SDK viitejuhendit](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ valgussensor — `connect_daq_sensor`

Ilma argumentideta tuvastab `connect_daq_sensor()` automaatselt andmeedastusviisi (eelistuse järjekord: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Iga raam sisaldab 135-punktilist `spectrum` (kalibreerimisel W/m²/nm), `is_saturated` lipukest ja CIE `x`, `y`, `z`. Konkreetse anduri või andmeedastusviisi kindlaksmääramiseks – usaldusväärne valik mitme võrguliidesega hostidel, kus Etherneti automaatne avastamine võib esimesel katsel töökõlbliku DAQ-E-seadme märkamata jätta – edastage üks selge vihje:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Pange tähele, et piirangute korrigeerimise profiilid (`cap_id`) **ei** ole SDK-i seadistus – valige need selle asemel `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` kaudu.

### Salvestatud projektid — `open_project`

Salvestatud Chloros projekt säilitab ühendatud riistvara (`cameras.json` + `sensors.json` koos `project.json`-ga), ning `chloros_sdk.open_project(path)` suudab kõik korraga uuesti ühendada ja salvestada andmeid seadme nime järgi. Vaata viites [Projekti automatiseerimine](reference/sdk-reference.md#project-automation--chlorosproject).

## Mida annab ainult pip-i kaudu tehtud install

Enne riistvarapindade kasutamist kontrollige moodulitasandi kättesaadavusmärgiseid:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Hostis, kus on **ainult** `pip install chloros-sdk` ja puudub Chloros töölaua pakett:

* `ChlorosLocal`, `process_folder` ja `process_lattice_capture` **ei** tööta — need vajavad töölaua paigaldaja sees kaasas olevat tagapõhja binaarfaili.
* Smart-Connecti abiprogrammid (`connect_camera`, `connect_array`, `connect_daq_sensor`) on puhtad HTTP-kliendid, seega töötavad nad teisel masinal asuva backendiga — kuid kaasasolevad backendid seovad end ainult loopbackiga, seega peate porti ise edasi suunama (nt `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) ja edastama `backend_url="http://127.0.0.1:5000"` koos `auto_start_backend=False`-ga. Vaata [Kaug-tagapõhja režiim](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Otseselt riistvaraga seotud LATTICE-klassid (`LatticeCamera`, `CameraPool`, …) on imporditavad, kuid vajavad töölaua paketist pärit Arena SDK-i käitamiskeskkonda — ilma selleta on `CAMERA_AVAILABLE` sama mis `False`.
* `daq_sdk` (otsese DAQ-klassid) kuulub lauaarvuti installi, mitte PyPI-paketti, seega on `DAQ_AVAILABLE` pip-põhises keskkonnas `False` – juhtige DAQ-andureid selle asemel `connect_daq_sensor()` kaudu (tunnelitud) tagapõhja vastu.

## Litsentsinõuded

SDK-ile juurdepääsuks on vaja aktiivset Chloros+ sisselogimist mõnel tasulisel tasemel — **Copper või kõrgem**(Copper / Bronze / Silver / Gold); tasuta tasemel „Iron“ puudub juurdepääs SDK/CLI-ile. Järelevalve toimub**serveri poolel**: iga SDK-päring peab sisaldama nii aktiivset seanssi kui ka tasulist paketti, vastasel juhul tagastab tagapõhi `403` / `PLAN_UPGRADE_REQUIRED` (mida `ChlorosLocal` tekitab kui `ChlorosLicenseError` ja `connect_*` abifunktsioonid kui `ChlorosConnectError`). Välja logitud helistaja saab selle asemel veakoodid `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — `chloros-cli login` uuesti käivitamine parandab esimese juhtumi, kuid mitte teist.

Offline-kasutamine toimib plaani armuaja jooksul: taset loetakse serveri valideerimise vahemälust (5 minutit) või allkirjastatud, masinaga seotud litsentsi vahemälust (kuuplaanide puhul 30 päeva; aastaplaanide puhul kuni tellimuse aegumiseni). Kui ajapikendus lõpeb, muutub pakett tasuta versiooniks ja juurdepääs koodile SDK peatatakse, kuni seade ühendub serveriga. Kood `chloros-cli status` jääb tasuta tasemel kättesaadavaks, seega on põhjus alati nähtav. Vaata [Chloros+ sisselogimine](chloros+-login.md).

## Erandid

Kasutage baasklassi, et käsitleda „kõiki Chloros-iga seotud vigu”:

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

Kõik torujuhtme erandid (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) tulenevad `ChlorosError`-ist. Üks nüanss: `ChlorosConnectError` — tekitatakse ainult `connect_camera` / `connect_array` / `connect_daq_sensor` poolt — tuleneb tavalisest `Exception`-ist, **mitte** `ChlorosError`-ist, seega `except ChlorosError` seda ei püüa kinni. Täielik hierarhia on esitatud [SDK viites](reference/sdk-reference.md#exceptions).

## Vaata ka

* [SDK viide](reference/sdk-reference.md) — täielik API pind, mis on optimeeritud AI-abistajate jaoks.
* [CLI viide](reference/cli-reference.md) — iga CLI alamkäsk peegeldab SDK-kõnet.
* [Allalaadimine](download.md) — paigaldusprogrammid Windows ja Linux jaoks.
