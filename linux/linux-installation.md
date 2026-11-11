# Linuxi paigaldamine

Chloros levitatakse Linuxi jaoks `.deb`-pakettidena, mis paigaldavad CLI-liidese, tagapõhja-serveri ja Pythoni SDK-i (süsteemi `python3` puhul lisaks versiooniga sobiv wheel virtuaalkeskkondadele).

Pakettide failinimed sisaldavad versiooni ja arhitektuuri: `chloros_1.3.0_amd64.deb` x86_64 jaoks, ja `chloros_1.3.0_arm64_jp6.deb` arm64 jaoks (NVIDIA Jetson, JetPack 6). Asenda allpool olevates käskudes tegelikult alla laetud faili nimega.

{% hint style="info" %}
**Uus versioonis 1.3.0: programmid tarnitakse lahtipakitud kujul.** Tagapõhi ja CLI paigaldatakse kaustadena, `/usr/lib/chloros/backend/` ja `/usr/lib/chloros/cli/`, mitte üksikute isepakkuvate failidena. Need ei paku end enam iga käivitamise korral ajutisse kausta lahti, mistõttu käivituvad need palju kiiremini (x86_64-l kulutas backend varem iga käivitamise korral lahtipakkimisele umbes 100 sekundit) ja ei vaja enam mitut GB vaba ruumi kataloogis `/tmp`. Amd64-versiooni `.deb` suurus on samuti umbes poole väiksem kui varem.
{% endhint %}

***

## Linuxi amd64 (x86_64)

### Süsteeminõuded

| Nõue | Miinimum | Soovituslik |
| --- | --- | --- |
| **Distributsioon** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Protsessor** | x86_64 (Intel/AMD) | Intel Core i7 või parem |
| **Mälu (RAM)** | 8 GB | 16 GB või rohkem |
| **Graafikakaart** | Puudub (töötlemine protsessoris) | NVIDIA GPU, millel on vähemalt 4 GB VRAM-i (12 GB või rohkem avab `GPU_PARALLEL`, 7 GB või rohkem hoiab Texture Aware'i ühe pildi teelt eemal) |
| **Salvestusruum** | 2 GB vaba ruumi | SSD, millel on vähemalt 10 GB vaba ruumi |
| **Python** | Python 3.7 või uuem (SDK jaoks) | Python 3.10 või uuem |

> **Ubuntu 20.04 ja Debian 11 ei ole toetatud.** `.deb`-i sõltuvuste nimekiri on
> tuletatud sellest, millega Chloros-i tagapõhi tegelikult seotud on, ja see hõlmab
> `libc6`, mis on uuem kui glibc 2.31, mida focal ja bullseye kaasa toovad, seega `apt` keeldub
> paigaldamisest kohe, selle asemel et lasta sellel hiljem käivitamisel ebaõnnestuda.

### Paigaldamine

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Kasutage `apt install ./…`, mitte `dpkg -i`.** `apt` lahendab paketi sõltuvused ja paigaldab ka soovitatud komponendid: Pythoni SDK nõuded, `python3-numpy` ja `python3-requests`. `dpkg -i` ei paigalda kumbagi. CLI ja backend on iseseisvad ning töötavad ilma nendeta, kuid `import chloros_sdk` vajab neid; paigaldamise väljund nimetab kõik puuduvad komponendid. Säilitage `./`, et `apt` installiks faili, selle asemel et otsida selle nimega paketti. Pakett deklareerib ka `libgl1`, mida komplektis olev OpenCV laadib ja mis võib minimaalsest serveri kujutisest puududa.
{% endhint %}

Kontrollige installi:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Süsteeminõuded

| Nõue | Minimaalne | Soovitatav |
| --- | --- | --- |
| **Platvorm** | NVIDIA Jetson koos JetPack 6-ga | Jetson Orin NX 16 GB või AGX Orin |
| **JetPack** | JetPack 6.x | Viimane JetPack 6 |
| **Mälu (RAM)** | 8 GB (jagatud GPU/CPU) | 16 GB+ jagatud (12 GB+ on paralleeltöötluse künnis) |
| **Salvestusruum** | 2 GB vaba ruumi | NVMe SSD, millel on vähemalt 10 GB vaba ruumi |
| **Python** | Python 3.7+ (SDK jaoks) | Python 3.10+ |

### Paigaldamine

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Sama ülesehitus kui amd64 `.deb`-il (sisaldab Pythoni SDK-i, vastuvõtupuhvreid ja tagapõhiteenuseid), mõeldud Jetson Orin / Orin NX / Orin Nano jaoks **JetPack 6**-l (L4T R36). Selle PyTorch on JetPack 6 CUDA-koostis Orini jaoks, seega töötab GPU-töötlus, sealhulgas Texture Aware debayer, Jetsoni GPU-l. Pakett ei sisalda CUDA-t ennast: see kasutab CUDA- ja cuDNN-teeke, mille JetPack 6 installib (`sudo apt install nvidia-jetpack` pildil, millel need puuduvad). Kui CUDA-GPU-d ei ole võimalik kasutada, toimub töötlemine CPU-l ja `--debayer texture-aware` kasutab Standard-debayerit (tagapõhi logib ühe `[DEBAYER] … needs a CUDA GPU …` rea, milles selgitatakse põhjust). Jetsoni mälu, soojusjuhtivuse ja välitingimustes kasutamise kohta vaadake [NVIDIA Jetsoni juhendit](nvidia-jetson-guide.md).***

## Pythoni SDK paigaldamine (kõik Linuxi versioonid)**`.deb` paigaldab SDK teie eest.** Mõlemad paketid (amd64 ja Jetson) paigaldavad selle süsteemi jaoks `python3`, seega töötab `import chloros_sdk` kohe pärast paigaldamist, ilma pip-ita ja võrguta. Selle paketid (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` ja projekti laadijad) paigutatakse paketi omandisse kuuluvasse `/usr/lib/python3/dist-packages`-i: uuendamine asendab need ning paketi eemaldamine eemaldab need. Versioon vastab alati teie poolt installitud CLI-le ja backendile.

SDK enda nõuded, `python3-numpy` ja `python3-requests`, on **soovitatavad, mitte kohustuslikud**: `sudo apt install ./chloros_<version>_<arch>.deb` installib need, `dpkg -i` aga mitte. Installimise väljund näitab SDK-versiooni, mida `python3` impordib, puuduvaid nõudeid ning kõik `chloros_sdk`-i versioonid, mis asuvad mujal `python3`-i teekonnas ja varjutavad paketi koopia (näiteks varem `pip install --user`-iga paigaldatud versioon).**Virtualenvis**, mis ei näe süsteemi pakette, installige komplektis olev wheel sinna (täielikult offline-režiimis):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**PyPI-st** — ainult arvutile, kuhu pole paigaldatud Chloros-paketti ja mille SDK juhib teisel arvutil asuvat tagapõhja (vt [Kaugtagapõhja režiim](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Valikulised lisad pip-paigaldamiseks (virtualenv või PyPI):

| Lisa | Käsk | Lisab |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` reaalajas edusammude voogedastuseks |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` BLE (DAQ-M) andmeedastuseks |

SDK kontrollimine:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python'i SDK suhtleb tagapõhjaga, mille `.deb` installib kohaliku HTTP-API kaudu (`http://127.0.0.1:5000`). `ChlorosLocal` ja ühendusabiprogrammid käivitavad selle vajaduse korral automaatselt; salvestatud projekti `connect_all()` seda kunagi ei tee, seega käivita tagapõhja [teenusena](#always-on-ptp-for-headless-hosts). Kasutage alati otsest IPv4-aadressi, mitte `localhost`-i — `localhost` võib lahenduda `::1`-iks ja kulutada umbes kaks sekundit iga päringu kohta.
{% endhint %}

***

## Esmane seadistamine

### 1. Sisselogimine

CLI-le ja SDK-le juurdepääs eeldab tasulist Chloros+ taset (**Copper** või kõrgem), mida rakendatakse serveri poolel: välja logitud kasutaja saab aadressi `401 AUTH_REQUIRED` ja tasuta taseme (Iron) kasutaja saab aadressi `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Autentimisandmed salvestatakse vahemällu `~/.chloros/user_session.json`.

{% hint style="info" %}
**Uuendamisel säilivad teie sisselogimisandmed; eemaldamisel kustutatakse need.** Uuema versiooni `.deb` paigaldamine olemasoleva installi peale (sh läbi `chloros-cli update`) säilitab `~/.chloros/user_session.json` ja vahemällu salvestatud litsentsi, mistõttu jätkavad töötamist ka graafikata CLI- ja SDK-ülesanded. Paketi (`sudo apt remove chloros`) eemaldamine kustutab salvestatud sisselogimisandmed ja litsentsi vahemälu kõigi selle arvuti kasutajate jaoks, seega on uue installi jaoks vaja uuesti `chloros-cli login`-i.
{% endhint %}

### 2. Kontrollige oma litsentsi staatust

```bash
chloros-cli status
```

`chloros-cli status` töötab igal tasemel (sh tasuta), seega saate alati näha, miks juurdepääs on või ei ole saadaval.

### 3. Käivita süsteemidiagnostika

```bash
chloros-cli selftest
```

Seitset kontrolli käivitatakse järjest ning kui mõni neist ebaõnnestub, lõpetab käsk töötamise nullist erineva tulemusega:

| # | Kontroll | Mida see tõestab |
| --- | --- | --- |
| 1 | **Versioon** | CLI näitab oma versiooni (`v1.3.0`). |
| 2 | **Port saadaval** | Port 5000 on vaba *või* sellele on juba vastanud töökorras Chlorosi tagapõhi (see loetakse edukaks). |
| 3 | **Tagapõhi käivitub** | Tagapõhi käivitatakse. |
| 4 | **API-test (`/api/test`)** | Tagapõhi vastab `status: ok`. |
| 5 | **Süsteemiinfo** | Väljastab `GPU: <name>, CUDA: <bool>, PyTorch: <version>` alates `/api/system-info`. |
| 6 | **Müra vähendamise mudelid** | Leiab `*.pth.enc` mudeleid (Linuxis: `/usr/lib/chloros/models`). |
| 7 | **CUDA + müra vähendaja**| Texture Aware on tegelikult kasutatav — vajab CUDA-d**ja** vähemalt ühte mudelifaili. |

Käivitus lõpeb `N/7 checks passed`-ga, loetledes kõik vead nimepidi.

### 4. Töötle oma esimest andmekogumit

```bash
chloros-cli process ~/datasets/flight001
```

***

## Failid ja kataloogid

### Kasutajapõhine

Chloros hoiab oma autentimisandmeid ja CLI-konfiguratsiooni ühes platvormiüleses kataloogis, **`~/.chloros/`** (Windowsis `%USERPROFILE%\.chloros\`). Kaks Linuxile omast vahemälud järgivad aga XDG-konventsioone – need kasutavad määramisel katalooge `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`.

| Tee | Otstarve |
| --- | --- |
| `~/.chloros/user_session.json` | `chloros-cli login` poolt kirjutatud sisselogimissessiooni vahemälu (säilitatakse paketi uuenduste käigus; tühjendatakse paketi eemaldamisel) |
| `~/.chloros/working_directory.txt` | Projektikausta vaikimisi asukoha ümberkirjutamine (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI-keele eelistus (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Keelesätted, mida jagatakse Windowsi graafilise kasutajaliidesega — siin on `language`-il eelisõigus `cli_language.json` ees |
| `~/.chloros/update_cache.json` | Ühetunnine vahemälu Linuxi/Jetsoni käivitamisel toimuva värskenduste kontrolli jaoks |
| `~/.chloros/backend.log` | Tagapõhja logi, kui tagapõhi käivitati CLI kaudu |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Kaamerate kaupa vahemällu salvestatud LATTICE kalibreerimispakid, indekseeritud seerianumbri ja paketi hash-väärtuse järgi |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Valikulised kasutaja ümberkirjutused DAQ-i kap-korrektsiooniprofiilidele |
| `~/.config/chloros/system_config.json` | Dünaamilise arvutusadapteerimise (Dynamic Compute Adaptation) vahemällu salvestatud riistvaraprofiil — selle kustutamine sunnib süsteemi uuesti riistvara tuvastama |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Tagapõhja serveri logid, üks fail iga käivitamise kohta |
| `~/Documents/MAPIR/Chloros Projects/` | Vaikimisi projektikaust, kui ümberkirjutust ei ole määratud |

### Süsteemitasandil

| Tee | Otstarve |
| --- | --- |
| `/usr/bin/chloros-cli` | Ümbritsev skript — määrab `LD_LIBRARY_PATH` komplektis olevate natiivteekide jaoks (ja `TMPDIR` väärtuseks `/mnt/ssd/tmp`, kui see kaust on olemas), seejärel käivitab tegeliku binaarfaili |
| `/usr/bin/chloros-backend` | Wrapper-skript — sama tagapõhja jaoks, mida see märgistab installitud tootmisversioonina |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Kompileeritud CLI ja tagapõhi, paigaldatakse lahtipakitud kujul (uus versioonis 1.3.0); programmid on `cli/chloros-cli` ja `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Kahe-realised käivitusskriptid, mis käivitavad eespool nimetatud tegelikud programmid — hoitakse nendes kataloogides, kuna SDK, CLI ja vanemad skriptid otsivad neid siit |
| `/usr/lib/chloros/arena_runtime/` | LATTICE-kaamerate jaoks vajalik Arena SDK-käivituskeskkond |
| `/usr/lib/chloros/models/*.pth.enc` | Texture Aware debayeri poolt kasutatavad krüpteeritud müra eemaldamise mudelid |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (ja `lattice_sdk`, `lattice_cam`, `arena_api` – projekti laadijad) | Pythoni SDK, mis on paigaldatud süsteemi `python3` jaoks ja kuulub paketile |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Täpselt selle koostisega Pythoni SDK-wheel virtuaalkeskkonna jaoks |
| `/usr/lib/chloros/exiftool` | Komplektis olev ExifTooli käivitaja (sümboliline link `/usr/local/bin/exiftool`-ile ainult juhul, kui süsteemis ei ole olemas exiftool-i). Alates versioonist 1.3.0 on ExifTooli skriptil kaasas oma Perl-teek `/usr/lib/chloros/exiftool-dist/`, mistõttu see töötab ilma süsteemipaketita `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | `chloros-cli update` poolt loetav uuenduskanali konfiguratsioon |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Seadistab `net.ipv4.ip_unprivileged_port_start = 319` nii, et tagapõhi saaks PTP-porte siduda ilma root-õigusteta |
| `/etc/sysctl.d/60-chloros-gige.conf` | Suurendab tuuma sokli vastuvõtupuhvreid (`net.core.rmem_max`, `net.core.rmem_default`) 32 MB-ni LATTICE-kaameravoogude jaoks ning rakendab neid paigaldamisel (juba suuremaks seatud väärtust ei muudeta). Tuuma vaikimisi seadistuse korral jäetakse kaamerapaketid tuumas kõrvale ning massiiv kaotab kaadreid ja kaadrisagedust; `chloros-cli lattice network` annab teada, kui puhvrid on tühjenemas. Vaata [Linuxi hostid — sokli vastuvõtupuhvrid](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Suunab dünaamilise laadija aadressile `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Annab sisseloginud kasutajale juurdepääsu DAQ-U USB-sarjasillale (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Lülita sisse alati töötav taustateenus, mis käitab taustasüsteemi sinu nimetatud kasutajana (paigaldatud, **ei ole aktiveeritud**) — see, mida tuleb aktiveerida, vaata [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Sama teenus, mis käitab taustateenust **root**-õigustes (paigaldatud,**ei ole aktiveeritud**), hoitakse alles, et masin, kus see on aktiveeritud, jätkaks töötamist |
| `/var/log/chloros/` | Root-teenuse üksikasjalik logi, mida saab lugeda rühm `adm` (kustutatakse paketi eemaldamisel) |
| `/usr/share/applications/chloros-cli.desktop` | „Chloros CLI” rakendusmenüü kirje, mis avab terminali |

**Tulemüür (uus versioonis 1.3.0).** Kui `ufw` või `firewalld` on paigaldamise ajal aktiivne, avab pakett UDP-pordid `5002` (DAQ-E andmed), `5353` (mDNS-avastamine), `319` ja `320` (PTP) ning eemaldab need reeglid, kui pakett eemaldatakse. Kui tulemüür on paigaldatud, kuid mitte aktiivne, ei lisa see midagi ning kuvab lubatavad pordid juhuks, kui peaksite selle hiljem sisse lülitama. See ei ava ühtegi TCP-porti: DAQ-E püsivara uuendamine, mille käigus andur ühendub TCP kaudu tagasi selle arvutiga pildi kogumiseks, ebaõnnestub veaga `… device never connected back on TCP <port> …`, kui aktiivne tulemüür blokeerib seda ühendust.

## Tagapõhja käivitatava faili asukoht

CLI ja SDK tuvastavad tagapõhja automaatselt:

| Komponent | Tee |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Tagapõhi | `/usr/lib/chloros/chloros-backend` (käivitusprogramm `/usr/lib/chloros/backend/chloros-backend`-ile) |

Backendi asukoha saab ümber määrata CLI-lipuga `--backend-exe` või SDK-konstruktori parameetriga `backend_exe`, ja porti lipuga `--port` (vaikimisi `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` suunab **`lattice`**,**`project`**ja**`daq pool-*`** käskude perekondadele kaug-tagapõhis. Põhikäsud (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignoreerivad seda tahtlikult ja suunavad alati käsu `http://127.0.0.1:<port>`-ile.
{% endhint %}

***

## LATTICE-kaamerad ja DAQ-valgusandurid Linuxis

Kõik live-hardware-käskude perekonnad töötavad Linuxis (amd64 ja Jetson):

* **`chloros-cli lattice`** — LATTICE-kaamerate ja sünkroniseeritud massiivide tuvastamine, ühendamine, konfigureerimine ja pildistamine. `.deb` koondab nende jaoks vajaliku Arena SDK-i käitamisaja ja registreerib selle dünaamilises laadijas.
* **`chloros-cli daq pool-*`** — ühendab DAQ-U/M/E valgussensoreid tagapõhja-pooli kaudu, edastab kalibreeritud spektreid, salvestab `.daq`-faile ja vahetab cap-profiile ning DAQ-E puhul muudab selle võrguseadeid, taaskäivitab selle ja uuendab selle püsivara. `chloros-cli daq discover` otsib andureid, millega saaksid ühendust luua, ja `chloros-cli daq --help` loetleb `pool-*` alamkäskud.
* **`chloros-cli project`** — käivitab salvestatud projekti (selle kaamerad, andurid ja töötlemisseaded) ilma kasutajaliideseta.
* **`chloros-cli time-sync`** — kontrollib PTP-grandmasteri, mida Chloros-tagapõhi kasutab LATTICE-kaamerate ja DAQ-E-andurite jaoks.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` on vajalik käskudele `pool-latest`, `pool-stream`, `pool-record` ja `pool-set-cap`; `pool-list` näitab hetkel reservis olevaid ID-sid.

{% hint style="info" %}
**`--eth-host` jätab avastamise vahele.** `pool-connect --eth` leiab DAQ-E mDNS-i abil, kasutades ARP-varuvõimalust, mis töötab tühjast ARP-vahemälust; anduri IP-aadressi või hostinime edastamine koos `--eth-host`-iga loob sellega otseühenduse.
{% endhint %}

**DAQ-U jadaliidese õigusi** haldab paigaldatud udev-reegel (`uaccess` + grupp `dialout`). Kui juba ühendatud andur jääb kättesaamatuks, laadige reeglid uuesti või ühendage andur uuesti:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Täieliku käskude loendi leiate [CLI-viitest](../CLI.md).

### Alati sisse lülitatud PTP peata hostide jaoks

Pakett installib tagapõhja jaoks kaks systemd-teenust, millest kumbki pole aktiveeritud. Kuvarieta Jetsonil või serveril, kus PTP-ajasünkroniseerimine peab DAQ-E-andurite ja LATTICE-kaamerate jaoks pidevalt töötama või kus SDK-skriptid ühendavad salvestatud projekte (`connect_all()` ei käivita kunagi tagapõhja), aktiveerige teenus **oma kasutajana**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` käitab backendit selle kasutajana: selle poolt salvestatud andmed kuuluvad sellele kasutajale, selle üksikasjalik logi asub selle kasutaja `~/.cache/chloros/logs`-is ning see kasutab selle kasutaja `chloros-cli login` (`~/.chloros`), seega logige esmalt korra sisse selle kasutajana. Paketi installimisel kuvatakse see käsk, millesse on sisestatud teie kasutajanimi.

Ilma teenuseta töötab tagapõhi (ja koos sellega ka PTP-grandmaster) ainult siis, kui keegi on selle käivitanud – kas käsuga „`chloros-cli`“ või SDK-skriptiga.

Seade seob tagapõhja `127.0.0.1:5000` (üksuse sisemised keskkonnaseaded `CHLOROS_HOST` / `CHLOROS_PORT`; ületage need `sudo systemctl edit chloros-backend@.service`-ga) ja taaskäivitab selle rikke korral 5 sekundi pärast.

**Root-teenus.**`chloros-backend.service` käitab tagapõhja**root**-õigustes. See on endiselt paigaldatud, et masin, kus see on aktiveeritud, jätkaks töötamist, kuid kõik, mida root-tagapõhi kirjutab, kuulub rootile, mistõttu kasutaja ei saa oma salvestusi kustutada; selle logi asub failis `/var/log/chloros`, mida saab lugeda rühm `adm`. Oma kasutajaks vahetamiseks:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Teenuse backend on jagatud ja klient ei saa seda kunagi peatada.** `chloros-cli` ja SDK-skriptid kasutavad seda sellisena, nagu nad selle leiavad: `process` töötab selle peal, laadides oma projekti viimase asemele. Kui CLImuidu taaskäivitaks backend'i (`--restart` või `process`, mille importimisel midagi ei leitud), nimetab ta üksuse, kuvab `sudo systemctl restart <unit>` ja lõpetab `1`, sest sel viisil peatatud tagapõhi jääks seisma. Käivita see ise uuesti käsuga `sudo systemctl restart chloros-backend@$USER.service`. Teenuse tagapõhi ei peatu kunagi ise ooterežiimis ja `chloros-cli backend stop` keeldub seda tegemast, nimetades käsku `systemctl stop`.**Kuidas PTP oma pordid saab.** PTP kasutab UDP-pordid 319 ja 320, mis mõlemad asuvad tavapärase 1024-portilise privileegitud pordide alampiiri all. Paketi `postinst` kirjutab `/etc/sysctl.d/60-chloros-ptp.conf`-i koos `net.ipv4.ip_unprivileged_port_start = 319`-iga, mis võimaldab tagapõhjal neid siduda, töötades teie kasutajana. Alates versioonist 1.3.0 ei määra pakett tagapõhja jaoks ühtegi failiõigust (`setcap`) ning `libcap2-bin` ei ole enam sõltuvus: lahtipakitud tagapõhjas takistaks failiõigus selle käivitumist, seega ärge lisage seda ise.***

## Bash-skriptide näited

{% hint style="info" %}
**Skriptimiseks sobivad väljumiskoodid.**`chloros-cli process` väljub `0`-ist õnnestumise korral ja**väljub nullist erineva väärtusega ebaõnnestumise korral — sealhulgas juhul, kui käivitamisel taotleti pilditooteid, kuid ühtegi ei kirjutatud** (see väljastab `Processing finished but wrote no image products.` ning nimetab projektikausta ja tavapärased põhjused). Edukad käivitused teatavad, kui palju pilditooteid salvestati (`Image products written: N`, pluss `(plus M clamp masks)`, kui salvestati maskid). Lõpetamiskoodid: `0` – edukas, `1` – ebaõnnestunud, `2` – argumentide viga, `130` – katkestatud.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Kehtivaid `--format` väärtusi on täpselt neli ja need sisaldavad tühikuid — märkige need alati jutumärkidesse:

| `--format` väärtus | Väljundkaust |
| --- | --- |
| `TIFF (16-bit)` *(vaikimisi)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` aktsepteerib `standard` (vaikimisi) või `texture-aware` (Chloros+).

### Automaatne töötlemine Croniga

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python'i SDK'i näide

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Veaotsing

### Pärast installimist ei leita CLI-d

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Tagapõhi ei käivitu pärast failiõiguste lisamist

Ärge käivitage `setcap` tagapõhis. Alates versioonist 1.3.0 ei määra pakett mingeid failivõimalusi ning PTP-i pordid 319/320 tulenevad seadistusest `sysctl`, mida on kirjeldatud jaotises [Kuidas PTP saab oma pordid](#always-on-ptp-for-headless-hosts). Failivõime paneb lahtipakitud backend'i turvalisse täitmisrežiimi, milles see ei leia oma raamatukogusid, mistõttu see ei käivitu. Kui olete selle lisanud, eemaldage see:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP ei käivitu / ei suuda seostuda pordiga 319

Veendu, et õigusteta portide alampiir on alandatud, ja kui seda pole tehtud, rakenda seda uuesti käesoleva käivituse jaoks:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Seejärel kontrollige grandmasterit:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice`-käskude täitmine ebaõnnestub veateatega „LATTICE kaameradraivereid ei leitud”

Tagapõhi ei suutnud laadida LATTICE-kaamera käitamiskeskkonda, millega see on paigaldatud. Iga `lattice`-käsk töötab tagapõhis; CLI-liides ise ei sisalda kaameradraivereid. Paigalda Chloros uuesti (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` installib käivituskeskkonda tagapõhja kõrvale, `/usr/lib/chloros/arena_runtime`, ja registreerib selle dünaamilises laadijas. Registreerimise kontrollimiseks:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Ebaõnnestunud käivitamise tagapõhja logid asuvad failis `~/.cache/chloros/logs/`. Kui port 5000 kuulub tagapõhja teenusele (`systemctl status chloros-backend@$USER.service`), siis taaskäivitage teenus, mitte lõpetage selle protsessi.

### CUDA-d ei tuvastatud

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` teatab samast asjast ühes reas: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Puuduvad jagatud raamatukogud

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Aeglane käivitumine SD-kaardiga süsteemidel

Alates versioonist 1.3.0 paigaldatakse tagapõhiteenus ja CLI pakkimata kujul, seega ei pakita need end enam iga käivitamise korral ajutisse kataloogi. Ajutised failid salvestatakse automaatselt kataloogi `/mnt/ssd/tmp`, kui see kataloog on olemas; muidu võite suunata need kiirele failisüsteemile:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Chlorosi uuendamine Linuxis

Käsk `update` on saadaval ainult Linuxis/Jetsonil. See kontrollib versiooni, mis on avaldatud `/etc/chloros/update.conf`-is konfigureeritud uuenduskanalis, ning pakub sobiva versiooni `.deb` allalaadimist ja installimist:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Linuxis/Jetsonis teostab CLI iga käivitumise korral ka blokeerimatu uuenduste kontrolli (tulemus salvestatakse üheks tunniks mällu parameetris `~/.chloros/update_cache.json`) ning kuvab teate `Update available: vX.Y.Z`, kui uuem versioon on olemas. Teie seaded, projektid ja sisselogimised säilivad uuenduse järel.

Paketi uuendamine peatab installimise ajaks kõik taustateenused, mistõttu kaotab töötav skript ühenduse kaamerate ja anduritega ning käivitab seejärel uuesti kõik need, mis on lubatud (käsitsi käivitatud, kuid kunagi lubamata jäänud teenus jääb peatatuks, nagu pärast taaskäivitamist).

## Eemaldamine

```bash
sudo apt remove chloros
```

Eemaldamine peatab ja lülitab välja kõik taustateenused, taastab vaikimisi määratud piirangu (1024), millest allpool asuvad pordid on õigusteta, eemaldab vastuvõtupuhvri seadistusfaili (käimasolevad väärtused jäävad alles kuni taaskäivitamiseni), komplektis oleva exiftool-sümbolilinki, Arena laadija konfiguratsiooni, paketi poolt lisatud tulemüüri reeglid ja selle poolt installitud Pythoni SDK ning tühistab vahemällu salvestatud autentimisandmed. Teie projektid ja `~/.chloros/` andmefailid jäävad puutumatuks. `sudo apt purge chloros` kustutab ka `/var/log/chloros`.

***

## Järgmised sammud

* [NVIDIA Jetsoni juhend](nvidia-jetson-guide.md) — Jetsonile spetsiifiline optimeerimine ja kasutuselevõtt
* [CLI: käsurea](../CLI.md) — CLI juhend
* [API: Python SDK](../api-python-sdk.md) — SDK juhend
* [CLI-viide](../reference/cli-reference.md) ja [SDK-viide](../reference/sdk-reference.md) — põhjalikud käskude/API-loendid versioonile 1.3.0
* [Dünaamiline arvutusvõimsuse kohandamine](../processing-architecture/dynamic-compute-adaptation.md) — kuidas Chloros kohandub teie riistvaraga
