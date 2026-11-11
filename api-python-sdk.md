# API: Python SDK

{% hint style="info" %}
**Otsite täielikku API-d?** See lehekülg on praktiline juhend. Kõik avalikud klassid, meetodid, täpsed signatuurid ja kopeeritavad näited on toodud [SDK-viites](reference/sdk-reference.md), mis on optimeeritud AI-assistendile.**Kasutad AI-assistenti?** Kleebi see URL vestlusaknasse, et saada kätte täielik ja ajakohane Chloros 1.3.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Käesoleva juhendi iga lehekülg on kättesaadav toores Markdown-vormingus aadressil, mis koosneb väiketähtedest koosnevast slugist + `.md`, ning kogu juhend on indekseeritud aadressil `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chlorosi Python SDK** (`chloros-sdk` PyPI-s) juhib kõiki töölaua rakenduse funktsioone Pythonist: piltide kogumitöötlust, LATTICE-kaamera ja maatriksi reaalajas juhtimist, DAQ-valgusanduri seansse ning salvestatud projektide automatiseerimist. Tegemist on õhukese kihiga, mis asub sama kohaliku tagapõhja peal, mida kasutavad nii graafiline kasutajaliides kui ka CLI (HTTP aadressil `127.0.0.1:5000`), mistõttu on käitumine kõigil kolmel kasutajaliidesel identne.

## Paigaldamine

SDK-ga kaasneb Chloros-i töölaua pakett, mis sisaldab ka töötlemise tagapõhja ja riistvara käivituskeskkondi ning mille versioon vastab alati SDK-i versioonile.

**

1. samm — Chlorosi paigaldamine.** Windows: käivita töölaua paigaldaja (vaikimisi asukoht `C:\Program Files\Chloros\`) lehelt [Allalaadimine](download.md). Linux: paigalda pakett `.deb` koos `sudo apt install ./chloros_1.3.0_<arch>.deb`-iga ([Linuxi paigaldamine](linux/linux-installation.md)).**

2. samm — Kontrollige SDK-i** (Python 3.7+). Tavaliselt pole midagi paigaldada:

* **Windows:** paigaldaja paigaldab komplektis oleva SDK-i wheel-faili teie süsteemi Pythonisse (kui Python pole paigaldatud, vahetab ta selle sammu üle).
* **Linux:** `.deb` paigaldab süsteemi `python3` jaoks mõeldud SDK-i `/usr/lib/python3/dist-packages`-i. Pakett soovitab järgmisi nõudeid: `python3-numpy` ja `python3-requests`; `apt install` sisaldab neid, `dpkg -i` aga mitte.
* **Virtualenv** ei näe süsteemi pakette; paigalda komplektis olev wheel sinna: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` Linuxis või `chloros_sdk-*.whl` Windowsi installikausta `sdk`-kaustas (vaikimisi `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Arvutis, kus puudub Chloros-pakett ja mille SDK juhib teise arvuti tagapõhja (vt [Mida annab ainult pip-i abil tehtud install](#what-a-pip-only-install-gets)), installige see PyPI-st: `pip install chloros-sdk`. PyPI uuendatakse iga uue versiooni väljalaskmisel, seega vastab see viimasele stabiilsele versioonile.

**

3. samm — Logi sisse üks kord igal arvutil:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Sisselogimisandmed salvestatakse vahemällu `~/.chloros/` (mõlemal platvormil). Windowsis saate sisse logida ka töölaua rakenduse vahekaardil „Kasutaja” <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">. SDK nõuab tasulist Chloros+ paketti — vaadake allpool [Litsentsinõuded](#license-requirement).

| Nõue | Üksikasjad |
| --- | --- |
| **Chloros on installitud** | Windows: töölaua installija; Linux: `.deb` pakett (sisaldab backend-binaarfaili ja SDK-i) |
| **Python** | 3.7 või uuem (arendatud/testitud versioonil 3.10) |
| **Operatsioonisüsteem** | Windows 10/11 64-bitine, Ubuntu 22.04 LTS või uuem või NVIDIA Jetson (JetPack 6) |
| **Litsents** | Aktiivne Chloros+ sisselogimine, mis tahes tasuline pakett (Copper või kõrgem) |

## 60-sekundiline võit

Ühe käsklusega luuakse projekt, imporditakse kaust, konfigureeritakse töötlemine ja käivitatakse töötlemisahel — tagapõhi käivitatakse automaatselt, kui see veel ei tööta:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(Linuxis kasuta Linuxi teekondi: `/home/user/drone_images/flight001`. SDK töötab mõlemal platvormil ühtmoodi.)

Töötled LATTICE&#x27;i salvestuskausta? Kasutage LATTICE-ga ühilduvat wrapperit – see rakendab õigeid vaikimisi seadeid (ei tuvasta paneeli sihtmärki, kasutab standardset debayeri):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` – täielik kontroll töötlemisahela üle

Kõigi üherealistest käskudest keerulisemate ülesannete jaoks kasutage `ChlorosLocal`. See kasutab juba töötavat tagapõhja (töölaua rakenduse oma) või käivitab esmakordsel kasutamisel installitud tagapõhja (`auto_start_backend=True`; Windowsis Chlorosi installikausta kaustast `backend`, vaikimisi `C:\Program Files\Chloros\backend\`), loob ja konfigureerib projekte, jälgib töö kulgu ning esitab pärast käivitamist kokkuvõtte.

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
Jätke vaikimisi `http://127.0.0.1:5000`, mitte asendage seda `localhost`-iga — Windowsis lahendatakse `localhost` esmalt `::1`-iks ja see võtab ~2 sekundit iga päringu kohta, kui kasutatakse ainult IPv4-tagapõhja.
{% endhint %}

Kasutage seda kontekstihaldurina. Blokist väljumine lõpetab selle instantsi poolt tagapõhja kasutamise, mitte tagapõhja enda töö: see jääb järgmise skripti jaoks töötama ja peatub ise 30 minuti pärast, kui seda ei kasutata. `shutdown_backend()` peatab ühe käivitatud instantsi.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` aktsepteerib järgmisi märksõnu: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` ja `custom_settings`. Peamised väärtused:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

LATTICE-spetsiifilised reguleerimisnupud (`input_level`, `radiometric_output`, `array_alignment*`-perekond) on koos täielike väärtustabelitega dokumenteeritud [SDK-viites](reference/sdk-reference.md#supported-values).

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

### Töökäigu järelkokkuvõtte lugemine — ja tühjade töökäikude tuvastamine

Lõpetamisel lisab `process()` tagapõhja töötlemise kokkuvõtte failina `result["summary"]`. Iga kirje failis `summary["hints"]` on täielik lause, mis selgitab kõiki märkimisväärseid asjaolusid — näiteks, miks jooks ei andnud tulemust — ning iga vihje edastatakse ka uuesti Pythoni vormingus `UserWarning`, mistõttu tühjad jooksud on iseenesest diagnoositavad isegi siis, kui sa sõnastikku kunagi ei vaata:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` ei tekita viga, kui käivitamine ei anna ühtegi pilti.** See on ainus koht, kus SDK ja CLI erinevad teadlikult: `chloros-cli process` käsitleb olukorda „tooteid taotleti, kuid ühtegi ei salvestatud” veana ja lõpetab töötamise nullist erineva tulemusega, samas kui SDK lõpetab töötamise normaalselt ja teatab sellest olukorrast `summary` / vihjete kaudu. Kui teie töötlemisahel peaks tühja käivituse korral peatuma, kontrollige seda ise – uurige `summary`-i (või loendage projektikausta all olevaid faile), selle asemel et tugineda erandile.
{% endhint %}

## Smart Connect — reaalajas töötav riistvara

Kolm abiprogrammi avavad püsivad seansid tagapõhja riistvarapoolis — samas poolis, mida kasutavad ka CLI, muud skriptid ja GUI kaameravahekaardid, nii et SDK-skriptid saavad nendega koos eksisteerida ilma, et nad võitleksid jadaportide või võrgu ribalaiuse pärast. Kõik kolm käivitavad automaatselt kohaliku tagapõhja, kui ükski neist ei ole käivitatud. GUI DAQ-paneel ei kuulu sellesse reservi: sulge valgussensor GUI-s enne, kui avad selle skriptist.

### Üksik LATTICE-kaamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Suhtelised teed viivad sinna, kus teie skript töötab (muudetud versioonis 1.3.0).** Kohaliku tagapõhja korral suhteline `output_dir` (sealhulgas vaikimisi `"output"`) — ja muud teed, mida tagapõhi avab (`burst_dir`, `profile_path`, `save_path`, `mask_path`) – lahendatakse teie skripti töökataloogi suhtes, mistõttu `cam.capture("output/")` kirjutab `<current folder>/output/`-i. Versioonis 1.2.0 lahendas installitud tagapõhi need oma installikausta suhtes ja andis veateate „Juurdepääs keelatud”. Kaugtagapõhi puhul saadetakse teed täpselt nii, nagu need on sisestatud.
{% endhint %}

### Sünkroniseeritud massiiv — `connect_array`

`connect_array` on soovitatav alguspunkt mitme kaameraga seadmetele. See käivitab sama smart-prep-voo kui graafiline kasutajaliides: võrguanalüüs, sünkroniseerimistaseme automaatvalik, PTP-ajasünkroniseerimine, pikseliformaadi valik iga kaamera jaoks, AE algväärtustamine ja GPIO-päästiku aktiveerimine. **Esimene seerianumber on master** (see käivitab riistvaralise triggimpulsi); ülejäänud on slave&#x27;id.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Lisage `smart=True` mis tahes massiivi salvestamisele, et oodata, kuni automaatne säri on kõigil kaameratel stabiliseerunud, enne kui päästikku vabastatakse.

Uus versioonis 1.3.0 pakub `ArraySession` teile sünkroniseeritud kaadreid otse, ilma faile salvestamata:

* `arr.grab()` tagastab ühe kaadri iga kaamera kohta, kõik ühe triggereerimise tulemusena, numpy-massiividena (`{serial: array}`); `.group_id` on ühine käivitus ja `.errors` nimetab kõik välja jäänud kaamerad.
* `arr.frame_stream(fps=5.0)` annab ühe sellise komplekti iga takti kohta, et saaksite neid oma töötlusketis reaalajas töödelda.
* `arr.continuous("output/")` käivitab backendi enda pideva salvestamise (GUI pidev režiim) ja tagastab `ContinuousHandle` koos `status()`, `wait()` ja `stop()` – kasutage seda salvestussilmuse asemel, kui vajate iga triggereid salvestamist suure sagedusega.

Salvestusrežiimide (ühekordne / pidev / intervall / kiireim), salvestite, burst-to-video ja massiivi joondamise kohta vaadake [SDK-viidet](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ valgussensor — `connect_daq_sensor`

Ilma argumentideta tuvastab `connect_daq_sensor()` automaatselt andmeedastusviisi (eelistuse järjekord: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Iga raam sisaldab 135-punktilist `spectrum` (kalibreerimisel W/m²/nm), `is_saturated` lipukest ning CIE `x`, `y`, `z`. Konkreetse anduri või andmeedastusviisi määramiseks edastage üks selgesõnaline vihje:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Pange tähele, et cap-korrektsiooniprofiilid (`cap_id`) **ei** ole SDK-i reguleeritavad parameetrid — valige need selle asemel `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` kaudu.

### Salvestatud projektid — `open_project`

Salvestatud Chloros-projekt säilitab ühendatud riistvara (`cameras.json` + `sensors.json` koos `project.json`-iga), ning `chloros_sdk.open_project(path)` suudab oma kaamerad, andurite massiivid ja valgussensorid korraga uuesti ühendada ning käivitada salvestusi seadme nime järgi. Valgusandurid ühenduvad uuesti töötava Chlorosi tagapõhja kaudu, säilitades oma salvestatud seaded ja pildistamispiirangud. Vaata viites [Projekti automatiseerimine](reference/sdk-reference.md#project-automation--chlorosproject).

## Mida annab ainult pip-i kaudu tehtud install

Enne riistvarapindade kasutamist kontrollige moodulitasandi kättesaadavusmärgiseid:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Hostis, kus on **ainult** `pip install chloros-sdk` ja puudub Chlorosi töölaua pakett:

* `ChlorosLocal`, `process_folder` ja `process_lattice_capture` **ei** tööta — need vajavad töölaua paigaldaja sees kaasasolevat tagapõhja binaarfaili.
* Smart-connect-abiprogrammid (`connect_camera`, `connect_array`, `connect_daq_sensor`) on puhtad HTTP-kliendid, seega töötavad nad teisel masinal asuva tagapõhja vastu — kuid kaasasolevad backendid seovad end ainult loopbackiga, seega peate porti ise edastama (nt `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) ja edastama `backend_url="http://127.0.0.1:5000"` koos `auto_start_backend=False`-iga. Vaata [Kaug-tagapõhja režiim](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Otse riistvaraga seotud LATTICE-klassid (`LatticeCamera`, `CameraPool`, …) imporditakse, kuid vajavad töölaua paketist pärit Arena SDK-i käivituskeskkonda — ilma selleta on `CAMERA_AVAILABLE` sama mis `False`.
* DAQ-andureid juhitakse ainult (tunnelitud) tagapõhja kaudu — `connect_daq_sensor()` või projekti `proj.sensors`; tagapõhi avab anduri oma masinas.

## AI-assistendid — `chloros-mcp`

SDK sisaldab `chloros-mcp`-i, MCP-serverit, mis võimaldab Claude Desktopil, Claude Code’il, Cursoril ja teistel MCP-klientidel juhtida Chloros’i: töödelda pilte ning käitada LATTICE-kaameraid, andurimassiive ja DAQ-valgusandureid. Selle tööriistad hõlmavad kõiki selle SDK funktsioone. See installitakse koos SDK-ga, seega on Claude Code&#x27;i seadistamine vaid ühe rea pikkune:

```bash
claude mcp add chloros -- chloros-mcp
```

Muud kliendid: [Chloros&#x27;i kasutamine AI-abistajatega](ai-assistants.md). Valikud ja kõik tööriistad: [MCP-server (AI-assistendid)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Litsentsinõuded

SDK-le juurdepääsuks on vaja aktiivset Chloros+ sisselogimist mis tahes tasulisel tasemel — **Copper või kõrgem**(Copper / Bronze / Silver / Gold); tasuta tasemel „Iron“ puudub juurdepääs SDK-le ja CLI-le. Nõude täitmist kontrollitakse**serveri poolel**: iga SDK-päring peab sisaldama nii aktiivset seanssi kui ka tasulist paketti, vastasel juhul tagastab backend veakoodid `403` / `PLAN_UPGRADE_REQUIRED` (mida `ChlorosLocal` tekitab kui `ChlorosLicenseError` ja `connect_*` abifunktsioonid kui `ChlorosConnectError`). Välja logitud helistaja saab selle asemel veakoodid `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — `chloros-cli login` uuesti käivitamine lahendab esimese juhtumi, kuid mitte teist.

Offline-kasutamine toimib paketi armuaja jooksul: taset loetakse serveri valideerimise vahemälust (5 minutit) või allkirjastatud, seadmega seotud litsentsi vahemälust (kuupakettide puhul 30 päeva; aastapakettide puhul kuni tellimuse lõppemiseni). Kui ajapikendus lõpeb, muutub pakett tasuta versiooniks ja SDK-juurdepääs peatub, kuni seade ühendub serveriga. `chloros-cli status` jääb tasuta tasemel kättesaadavaks, seega on põhjus alati nähtav. Vaata [Chloros+ sisselogimine](chloros+-login.md).

## Erandid

Püüa kinni baasklass, et käsitleda „kõiki Chlorosi vigu”:

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

Kõik torujuhtme erandid (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) pärinevad `ChlorosError`-ist. Üks nüanss: `ChlorosConnectError` — tekitatakse ainult `connect_camera` / `connect_array` / `connect_daq_sensor` poolt — pärineb tavalisest `Exception`-ist, **mitte** `ChlorosError`-ist, seega `except ChlorosError` seda ei tuvasta. Täielik hierarhia on esitatud [SDK-viites](reference/sdk-reference.md#exceptions).

## Vaata ka

* [SDK-viide](reference/sdk-reference.md) — täielik API-liides, mis on optimeeritud AI-assistendidele.
* [CLI-viide](reference/cli-reference.md) — iga CLI-alakäsk peegeldab SDK-kõnet.
* [Chlorose kasutamine AI-abistajatega](ai-assistants.md) — `chloros-mcp` seadistamise ja käskude näited.
* [Allalaadimine](download.md) — paigaldusprogrammid Windowsile ja Linuxile.
