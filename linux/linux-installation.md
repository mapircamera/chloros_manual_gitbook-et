# Linux paigaldamine

Chloros levitatakse Linux jaoks `.deb` pakettidena, mis paigaldavad CLI ja tagapõhjaserveri. Python SDK on eraldi pip-pakett (samuti on see lisatud `.deb`-i versiooniga sobiva wheel-failina).

Paketi failinimed sisaldavad versiooni ja arhitektuuri: `chloros_1.2.0_amd64.deb` x86_64 jaoks ning `chloros_1.2.0_arm64_jp6.deb` JetPack 6 Jetsoni koostude jaoks. Asenda allpool olevates käskudes tegelikult alla laetud faili nimega.

***

## Linux amd64 (x86_64)

### Süsteeminõuded

| Nõue | Miinimum | Soovitatav |
| --- | --- | --- |
| **Distributsioon** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Protsessor** | x86_64 (Intel/AMD) | Intel Core i7 või parem |
| **Mälu (RAM)** | 8 GB | 16 GB või rohkem |
| **Graafikakaart** | Puudub (töötleb protsessor) | NVIDIA GPU, millel on vähemalt 4 GB VRAM-i (12 GB või rohkem avab `GPU_PARALLEL`, 7 GB või rohkem hoiab Texture Aware&#x27;i ühe pildi teelt eemal) |
| **Salvestusruum** | 2 GB vaba ruumi | SSD, millel on vähemalt 10 GB vaba ruumi |
| **Python** | Python 3.7+ (SDK jaoks) | Python 3.10+ |

> **Ubuntu 20.04 ja Debian 11 ei ole toetatud.** `.deb` sõltuvuste nimekiri on
> tuletatud sellest, millega Chloros tagapõhi tegelikult seotud on, ja see hõlmab
> `libc6 (>= 2.34)`. Nii Focal kui ka bullseye sisaldavad glibc 2.31, seega `apt` keeldub
> paigaldamisest kohe, selle asemel et lasta sellel hiljem käivitamisel ebaõnnestuda.

### Paigaldamine

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` ei suuda sõltuvusi lahendada. Kui see teatab puuduvatest pakettidest, viib `sudo apt-get install -f` (või `sudo apt --fix-broken install`) paigaldamise lõpule — see on normaalne protsess, mitte viga.
{% endhint %}

Kontrollige paigaldust:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Süsteeminõuded

| Nõue | Miinimum | Soovituslik |
| --- | --- | --- |
| **Platvorm** | NVIDIA Jetson koos JetPack 6-ga | Jetson Orin NX 16 GB või AGX Orin |
| **JetPack** | JetPack 6.x | Uusim JetPack 6 |
| **Mälu (RAM)** | 8 GB (jagatud GPU/CPU) | 16 GB+ jagatud (12 GB+ on paralleelsete GPU-töötajate künnis) |
| **Salvestusruum** | 2 GB vaba ruumi | NVMe SSD, millel on 10 GB+ vaba ruumi |
| **Python** | Python 3.7+ (mudeli SDK jaoks) | Python 3.10+ |

### Paigaldamine

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Sama ülesehitus kui amd64-versioonil `.deb`, kuid CUDA-koostis on häälestatud Jetson Orin / Orin NX / Orin Nano jaoks. Jetsoni mälu, soojusjuhtivuse ja välitingimustes kasutamise kohta vaata [NVIDIA Jetsoni juhendit](nvidia-jetson-guide.md).

***

## Python SDK paigaldamine (kõik Linux)

SDK on backendile mõeldud puhas Python HTTP-klient, seega töötab sama pakett nii amd64- kui ka arm64-platvormidel. Kaks allikat:**PyPI-st** — avaldatud stabiilne versioon:

```bash
pip install chloros-sdk
```

**Komplektis olevast wheel-failist** — sobib kindlasti äsja installitud CLI/backendiga (kasutage seda, kui teie `.deb` on uuem kui PyPI-s olev):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668 distributsioonid** (Ubuntu 23.10+, Debian 12+) ei luba süsteemiüleseid pip-paigaldusi. Kasutage `pip install --user …`, virtuaalkeskkonda või `sudo pip install --break-system-packages …`. Paketi paigaldaja ei paigalda kunagi automaatselt SDK-i teie süsteemi Python — see valik jääb teie enda teha.
{% endhint %}

Valikulised lisad:

| Lisand | Käsk | Lisab |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` reaalajas edusammude voogedastuseks |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` BLE (DAQ-M) andmeedastuseks |

Kontrollige SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` installib Chloros, CLI ja tagapõhja. Python, SDK suhtleb selle backendiga kohaliku HTTP API (`http://127.0.0.1:5000`) kaudu ja käivitab selle vajaduse korral automaatselt. Kasutage alati otsest IPv4-aadressi, mitte `localhost` — `localhost` võib lahendada `::1`-iks ja kulutada umbes kaks sekundit iga päringu kohta.
{% endhint %}

***

## Esmane seadistamine

### 1. Logi sisse

CLI ja SDK juurdepääs eeldab tasulist Chloros+ taset (**Copper** või kõrgem), mida rakendatakse serveri poolel: välja logitud kasutaja saab `401 AUTH_REQUIRED` ja tasuta taseme (Iron) kasutaja saab `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Autentimisandmed salvestatakse vahemällu `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Peate pärast iga installimist või uuendamist uuesti sisse logima.** Paketi skript `prerm` tühistab tahtlikult `~/.chloros/user_session.json` ja vahemällu salvestatud litsentsi kõigi arvutis olevate kasutajate puhul, nii et uus versioon valideerib litsentsi alati uuesti, selle asemel et tugineda aegunud vahemällule.
{% endhint %}

### 2. Kontrollige oma litsentsi staatust

```bash
chloros-cli status
```

`chloros-cli status` töötab igal tasemel (sh tasuta versioonis), seega saate alati näha, miks juurdepääs on või ei ole saadaval.

### 3. Käivitage süsteemidiagnostika

```bash
chloros-cli selftest
```

Seitset kontrolli käivitatakse järjest ning kui mõni neist ebaõnnestub, lõpetab käsk töö nullist erineva tulemusega:

| # | Kontroll | Mida see tõestab |
| --- | --- | --- |
| 1 | **Versioon** | CLI teatab oma versiooni (`v1.2.0`). |
| 2 | **Port vaba** | Port 5000 on vaba *või* sellele vastab juba töökorras Chloros-tagapõhi (mis loetakse edukaks). |
| 3 | **Tagapõhi käivitub** | Tagapõhi käivitatakse. |
| 4 | **API test (`/api/test`)** | Tagapõhi vastab `status: ok`. |
| 5 | **Süsteemiinfo** | Väljastab `GPU: <name>, CUDA: <bool>, PyTorch: <version>` alates `/api/system-info`-st. |
| 6 | **Müra vähendamise mudelid** | Leiab `*.pth.enc` mudeleid (Linux-is: `/usr/lib/chloros/models`). |
| 7 | **CUDA + müra vähendaja**| Tekstuuritundlik funktsioon on tegelikult kasutatav — vajab CUDA-d**ja** vähemalt ühte mudelifaili. |

Käivitus lõpeb mudeliga `N/7 checks passed`, loetledes kõik ebaõnnestumised nimepidi.

### 4. Töötle oma esimest andmekogumit

```bash
chloros-cli process ~/datasets/flight001
```

***

## Failid ja kataloogid

### Kasutaja kohta

Chloros hoiab oma autentimisandmeid ja CLI konfiguratsiooni ühes platvormiüleses kataloogis, **`~/.chloros/`** (Windows-is, `%USERPROFILE%\.chloros\`-is). Kaks Linux-spetsiifilist vahemälud järgivad selle asemel XDG-konventsioone — need järgivad `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, kui need on määratud.

| Tee | Otstarve |
| --- | --- |
| `~/.chloros/user_session.json` | `chloros-cli login` poolt kirjutatud sisselogimissessiooni vahemälu (tühjendatakse iga paketi installimise/uuendamise korral) |
| `~/.chloros/working_directory.txt` | Projektikausta vaikimisi asukoha ümberkirjutamine (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI keele-eelistus (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Keelesätted, mida jagatakse Windows kasutajaliidesega — siin on `language`-il eelisõigus `cli_language.json` ees |
| `~/.chloros/update_cache.json` | Ühetunnine vahemälu Linux/Jetsoni käivitamisel toimuva värskenduste kontrolli jaoks |
| `~/.chloros/backend.log` | Tagapõhja logi, kui tagapõhi käivitati CLI poolt |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Kaamerate kaupa vahemällu salvestatud LATTICE kalibreerimispakid, indekseeritud seerianumbri ja paketi hash-väärtuse järgi |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Valikulised kasutaja ümberkirjutused DAQ-i kaap-korrektsiooniprofiilidele |
| `~/.config/chloros/system_config.json` | „Dynamic Compute Adaptation“i poolt salvestatud riistvaraprofiil – selle kustutamine sunnib süsteemi uuesti riistvara tuvastama |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-serveri logid, üks fail iga käivitamise kohta |
| `~/Chloros Projects/` | Vaikimisi projektikausta, kui ümberkirjutust ei ole määratud |

### Süsteemipõhine

| Tee | Otstarve |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-skript — määrab `LD_LIBRARY_PATH` komplektis olevatele natiivraamatukogudele ning käivitab seejärel tegeliku binaarfaili |
| `/usr/bin/chloros-backend` | Wrapper-skript — sama, pluss `CHLOROS_PRODUCTION=1`, et tagapõhja autentimislüüs ei saaks end kunagi vaikselt välja lülitada |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Kompileeritud binaarfailid |
| `/usr/lib/chloros/arena_runtime/` | LATTICE-kaamerate jaoks vajalik Arena SDK-i käitamiskeskkond |
| `/usr/lib/chloros/models/*.pth.enc` | Texture Aware debayeri poolt kasutatavad krüpteeritud müra eemaldamise mudelid |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK – täpselt sellele koostisele vastav ratas |
| `/usr/lib/chloros/exiftool` | Kaasasolev exiftool (sümboliline link `/usr/local/bin/exiftool`-ile ainult juhul, kui süsteemis exiftool puudub) |
| `/etc/chloros/update.conf` | Uuenduskanali konfiguratsioon, mida loeb `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Seadistab `net.ipv4.ip_unprivileged_port_start = 319` nii, et tagapõhi saaks PTP-porte siduda ilma root-õigusteta |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Suunab dünaamilise laadija `/usr/lib/chloros/arena_runtime`-ile |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Annab sisseloginud kasutajale juurdepääsu DAQ-U USB-jadaliidesele (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Lülita sisse pidevalt töötav taustateenus (paigaldatud, **ei ole aktiveeritud**) |
| `/usr/share/applications/chloros-cli.desktop` | Rakendusmenüü kirje „Chloros CLI”, mis avab terminali |

## Tagapõhja käivitatava faili asukoht

CLI ja SDK tuvastavad tagapõhja automaatselt:

| Komponent | Tee |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Backendi asukoha saab ümber määrata lipuga `--backend-exe` CLI või konstruktori parameetriga `backend_exe` SDK, ning porti lipuga `--port` (vaikimisi `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` suunab **`lattice`**,**`project`**ja**`daq pool-*`** käskude perekondadele kaug-tagapõhisüsteemis. Põhikäskudeks on (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignoreerivad seda tahtlikult ja suunavad alati käsu `http://127.0.0.1:<port>`-ile.
{% endhint %}

***

## LATTICE-kaamerad ja DAQ-valgusandurid seadmel Linux

Kõik live-hardware-käskude perekonnad töötavad Linux-il (amd64 ja Jetson):

* **`chloros-cli lattice`** — LATTICE-kaamerate ja sünkroniseeritud massiivide avastamine, ühendamine, konfigureerimine ja pildistamine. `.deb` sisaldab neile vajalikku Arena SDK-i jooksuaega ja registreerib selle dünaamilise laadijaga.
* **`chloros-cli daq pool-*`** — ühendab DAQ-U/M/E valgussensorid tagapõhja-pooli kaudu, edasta kalibreeritud spektreid ja salvestada `.daq`-faile. Kompileeritud CLI sisaldab ainult `pool-*`-perekonda: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — käivita salvestatud projekt (selle kaamerad, andurid ja töötlemisseaded) ilma kasutajaliideseta.
* **`chloros-cli time-sync`** — kontrolli PTP-grandmasterit, mille jaoks Chloros-tagapõhi töötab LATTICE-kaamerate ja DAQ-E-andurite jaoks.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` on vajalik `pool-latest`, `pool-stream`, `pool-record` ja `pool-set-cap`; `pool-list` näitab hetkel reservis olevaid ID-sid.

{% hint style="info" %}
**Mitme võrguliidesega arvutis soovitame esimeseks DAQ-E-ühenduseks kasutada `--eth-host`.** Automaatne avastamine otsib mDNS-i kaudu ja võib tühjast ARP-vahemälust anduri liidese leidmata jätta, mistõttu võib esimene `pool-connect --eth` pärast süsteemi käivitamist ebaõnnestuda isegi siis, kui andur on täiesti töökorras. Anduri IP-aadressi või hostinime edastamine jätab avastamise täielikult vahele.
{% endhint %}

**DAQ-U seeriaõigusi** haldab paigaldatud udev-reegel (`uaccess` + grupp `dialout`). Kui juba ühendatud andur jääb kättesaamatuks, laadige reeglid uuesti või ühendage andur uuesti:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Kogu käskude loetelu leiate [CLI-viitest](../CLI.md).

### Pidevalt töötav PTP peata hostide jaoks

Esmakordsel installimisel luuakse systemd-üksus `chloros-backend.service`, kuid see **ei ole aktiveeritud**. Kui kasutate ekraanita Jetsonit või serverit, kus PTP-ajastamine peab DAQ-E-andurite ja LATTICE-kaamerate jaoks pidevalt töötama, aktiveerige see:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Ilma selleta töötab PTP ainult siis, kui tagapõhi Chloros töötab — st aktiivse CLI/SDK-seansi ajal.

Seade seob taustaprogrammi seadme sisemiste `127.0.0.1:5000` (seadme sisemised keskkonnaseaded `CHLOROS_HOST` / `CHLOROS_PORT`; ülekirjutage sellega `sudo systemctl edit chloros-backend.service`) ja taaskäivitab selle rikke korral 5 sekundi pärast.

**Kuidas PTP oma pordid saab.** PTP kasutab UDP-porte 319/320, mis mõlemad asuvad tavapärase 1024-portilise privileegitud pordide alampiiri all. Paketi `postinst` kirjutab `/etc/sysctl.d/60-chloros-ptp.conf`-i koos `net.ipv4.ip_unprivileged_port_start = 319`-iga, mis võimaldab tagapõhjal neid siduda, töötades teie kasutajana. Samuti rakendab see `setcap cap_net_bind_service,cap_net_raw=+ep` tagapõhja binaarile lisameetmena – seetõttu on `libcap2-bin` paketi deklareeritud sõltuvus.***

## Bash-skriptide näited

{% hint style="info" %}
**Skriptimiseks sobivad väljumiskoodid.**`chloros-cli process` väljub `0`-iga edu korral ja**nullist erineva väärtusega ebaõnnestumise korral – sealhulgas juhul, kui käivitamisel taotleti pilditooteid, kuid ühtegi ei kirjutatud** (see väljastab koodi `Processing finished but wrote no image products.` ning nimetab projektikausta ja tavapärased põhjused). Edukad käivitused teatavad, kui palju pilditooteid salvestati (`Image products written: N`). Lõpetamiskoodid: `0` – edukas, `1` – ebaõnnestunud, `2` – argumentide viga, `130` – katkestatud.
{% endhint %}

### Mitme andmekogumi töötlemine

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Töötlemine kohandatud seadete abil

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Kehtivaid `--format` väärtusi on täpselt neli ja need sisaldavad tühikuid — pane need alati jutumärkidesse:

| `--format` väärtus | Väljundkaust |
| --- | --- |
| `TIFF (16-bit)` *(vaikimisi)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` aktsepteerib `standard` (vaikimisi) või `texture-aware` (Chloros+).

### Automaatne töötlemine Croni abil

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Näide

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Veatõrje

### CLI ei leitud pärast paigaldamist

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Juurdepääs keelatud

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### „setcap ebaõnnestus” paigaldamise ajal

`.deb` rakendab `cap_net_bind_service`-i `/usr/lib/chloros/chloros-backend`-ile, et see saaks siduda PTP-porte 319/320 ilma root-õigusteta. Kui `libcap2-bin` puudus paigaldamise ajal, jäetakse see käsk vahele. Paigalda see ja paigalda pakett uuesti:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP ei käivitu / ei saa seostada porti 319

Veenduge, et piiranguteta portide alampiir on alandatud, ja kui see pole veel tehtud, rakendage seda uuesti käesoleva käivituse jaoks:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Seejärel kontrollige grandmasterit:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### „LATTICE kaameradraivereid ei leitud”

Arena SDK käitamiskeskkonda ei suudeta lahendada. Veenduge, et paketi poolt kirjutatud laadija konfiguratsioon on olemas ja värskendatud:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Tagapõhja käivitamine ebaõnnestus

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Ebaõnnestunud käivitamise tagapõhja logid asuvad failis `~/.cache/chloros/logs/`.

### CUDA-d ei tuvastatud

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` teatab samast asjast ühes reas: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Puuduvad jagatud raamatukogud

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Aeglane käivitumine SD-kaardiga süsteemidel

Kompileeritud binaarfailid pakivad end iga käivitamise korral ajutisse kataloogi lahti. Kui `/mnt/ssd/tmp` on olemas, kasutab Chloros seda automaatselt; muul juhul seadista `TMPDIR` kiirele failisüsteemile:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Chloros-i uuendamine Linux-is

Käsk `update` on saadaval ainult Linux/Jetsonil. See kontrollib `/etc/chloros/update.conf`-is konfigureeritud uuenduskanalis avaldatud versiooni ja pakub sobiva `.deb` allalaadimist ja installimist:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Linux/Jetsonil teostab CLI iga käivitumise korral ka blokeerimatu uuenduste kontrolli (tulemus salvestatakse `~/.chloros/update_cache.json`-is üheks tunniks) ning kuvab teate `Update available: vX.Y.Z`, kui uuem versioon on olemas. Teie seaded ja projektid säilivad uuenduse järel; pärast seda peate uuesti sisse logima.

## Eemaldamine

```bash
sudo apt remove chloros
```

Eemaldamine peatab `chloros-backend.service`, taastab vaikimisi määratud õigusteta portide alampiiri (1024), eemaldab komplektis oleva exiftool-sümbolilinki ja Arena laadija konfiguratsiooni ning tühistab vahemällu salvestatud autentimisandmed. Teie projektid ja `~/.chloros/` andmefailid jäävad puutumata.

***

## Järgmised sammud

* [NVIDIA Jetsoni juhend](nvidia-jetson-guide.md) — Jetsonile spetsiifiline optimeerimine ja kasutuselevõtt
* [CLI : käsurea](../CLI.md) — CLI juhend
* [API : Python SDK](../api-python-sdk.md) — juhend SDK
* [CLI viide](../reference/cli-reference.md) ja [SDK viide](../reference/sdk-reference.md) — täielikud käskude/API loendid versioonile 1.2.0
* [Dünaamiline arvutusvõimsuse kohandamine](../processing-architecture/dynamic-compute-adaptation.md) — kuidas Chloros kohandub teie riistvaraga
