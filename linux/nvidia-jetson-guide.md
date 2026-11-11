# NVIDIA Jetsoni juhend NVIDIA Jetsonil töötav

Chloros
võimaldab multispektraalset pilditöötlust äärearvutuses – välitingimustes, droonidel ja kaugpaigaldistes. Chloros
1.3.0 tuvastab käivitamisel teie Jetsoni mudeli ja optimeerib töötlusstrateegia vastavalt leitud riistvarale. **Käsitsi seadistamist ei ole vaja.**
{% hint style="info" %}
**Jetsoni pakett versioonis 1.3.0 töötleb andmeid Jetsoni GPU-l.** `chloros_1.3.0_arm64_jp6.deb` on loodud JetPack 6 jaoks ja kasutab JetPack 6 CUDA-versiooni PyTorchist, seega töötab kogu töötlemine, sealhulgas tekstuuritundlik debayer, Orini GPU-l. Pakett ei sisalda CUDA-t ennast: see kasutab CUDA- ja cuDNN-teeke, mille JetPack 6 installib (`sudo apt install nvidia-jetpack` pildil, millel need puuduvad). Kui CUDA-graafikaprotsessorit ei ole võimalik kasutada, toimub töötlemine CPU-l ja Texture Aware kasutab Standard-debayerit.
{% endhint %}

***

## Toetatud Jetson-mudelid

| Mudel                | RAM            | Töötlemisstrateegia                                     | Soovitatav kasutus                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB jagatud | `GPU_PARALLEL` (2 töötajat)                              | Maksimaalne jõudlus, suured andmekogumid                      |
| **Jetson Orin NX**   | 8–16 GB jagatud | `GPU_PARALLEL` (2 töötajat, 16 GB) / `GPU_SINGLE` (8 GB)   | Esmane soovitus välitingimustes kasutamiseks              |
| **Jetson Orin Nano** | 8 GB jagatud     | `GPU_SINGLE` (1 tööjaam, järjestikune)                     | Algastme servaarvutus                                 |

{% hint style="info" %}
Linux
i arm64-pakett nõuab **JetPack 6**-i, mis on saadaval Jetson Orin-seerias. Vanemad mudelid (Nano, TX2, Xavier NX) ei saa JetPack 6-i käivitada ja neid praegune pakett ei toeta.
{% endhint %}

***

## Nõuded

* **JetPack 6.x** (soovitatav on uusim versioon)
* **NVIDIA CUDA ja cuDNN** JetPack 6-st (`sudo apt install nvidia-jetpack`, kui need puuduvad teie pildist)
* **Tasuline Chloros
+ pakett** — Copper-tase või kõrgem (nõutav kogu CLI
- ja SDK
-juurdepääsu jaoks; rakendatakse serveri poolel)

## Paigaldamine

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` paigaldab süsteemile `python3` selle koostisega sobiva Python
i SDK
. Ainult virtualenv vajab komplektis oleva wheeli paigaldamist: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (täielikult offline-režiimis). Üldiste Linux
i paigaldamise üksikasjade, failide asukohtade ja veaotsingute kohta vaata [Linux
i paigaldamine](linux-installation.md).

{% hint style="info" %}
**Käivitamisel ei toimu enam lahtipakkimist.** Alates versioonist 1.3.0 paigaldatakse tagapõhi ja CLI
lahtipakitud kujul kataloogi `/usr/lib/chloros/`, seega ei pakita need end enam iga käivitamise korral ajutisse kataloogi. Ajutised failid salvestatakse endiselt automaatselt kataloogi `/mnt/ssd/tmp`, kui see kataloog on olemas; vastasel juhul saate määrata `TMPDIR`-ile tee oma NVMe-le (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dünaamiline arvutusvõimsuse kohandamine Jetsonil

### Kuidas see toimib

Käivitamisel koostab Chloros
teie süsteemi profiili:

1. **Tuvastab Jetsoni mudeli** `/proc/device-tree/model` kaudu
2. **Loeb kättesaadavat jagatud GPU/CPU mälu** (Jetson kasutab ühtset mälu)
3. **Küsib PyTorchilt, kas see saab kasutada CUDA-t** — kui JetPack 6 CUDA ja cuDNN on installitud, siis saab
4. **Valib töötlemisstrateegia** (`GPU_PARALLEL`, `GPU_SINGLE` või `CPU_PARALLEL`) ja töötajate arvu automaatselt

Otsus sõltub **kogumahust jagatud RAM-ist**, mitte mudeli nimest:

* **Kogumahuga alla 12 GB RAM**(kõik 8 GB Jetsonid):**1 töötaja — tahtlik järjestikune töötlemine**. Mälu on samaaegsete töötajate jaoks liiga piiratud, seega töödeldakse pilte ükshaaval. Jetsonitel, millel on**8 GB või vähem**, jätab Thread 3 töötajate rühma täielikult vahele ja teostab pildipõhiseid ülesandeid protsessi sees. Tagapõhja logis nimetatakse seda strateegiat `GPU_SINGLE`.
* **12 GB või rohkem**(Orin NX 16 GB, AGX Orin): ühtne mälu vastab `GPU_PARALLEL` tingimustele, kuid töötajate arv on**Jetsonil piiratud kahega** — GPU, tööprotsesside RAM-mälu ja iga tööprotsessi CUDA-kontekstid kasutavad kõik sama jagatud mälupooli, mistõttu suurema tööprotsesside arvu korral on oht, et tekivad mälupuudusest tingitud vead.
* **CUDA-d ei saa kasutada** (JetPacki CUDA-teegid puuduvad): `CPU_PARALLEL` 12 GB või rohkem, allpool üks pilt korraga.

Automaatset valikut saab ümber kirjutada keskkonnamuutuja `CHLOROS_STRATEGY` abil — vt [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Mudelipõhine käitumine

| Jetsoni mudel                | Strateegia       | Töötajad | Täitmine                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Järjestikune protsessisisene tsükkel (`tiled_gpu` mälukoormuse korral) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Järjestikune protsessisisene tsükkel                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Paralleelsed tööprotsessid, `fused_gpu` tee  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Paralleelsed tööprotsessid, `fused_gpu` tee  |

Platvormide peamine erinevus on **mälu**. 8 GB Jetson peab suure koormuse korral töötlema pilte ükshaaval, kasutades mälu säästvat tiled-lähenemist, samas kui 16 GB+ Orin suudab töötada GPU kaudu samaaegselt kahte pilti, kasutades suurema läbilaskevõimega fused pipeline&#x27;i.

### GPU-ressursid mudeli kohta

Igal Jetsoni mudelil on ka riistvaraprofiil, mis piirab, kui palju jagatud mälupoolist töötlemine võib hõivata, ning skaalab partiide suurusi:

| Mudel | GPU-ressursside ülempiir | Partiisuuruse kordaja | Süsteemile/ekraanile reserveeritud |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

Profiili kohandatakse vastavalt tuvastatud RAM-ile: kui Jetson teatab **16 GB või rohkem**, suurendatakse selle partii kordajat ×1,2 võrra. Kordajate kohaldamise eelne partii baassuurus on 8 pilti.

Täieliku arvutusvõimsuse kohandamise juhendi leiate [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

***

## GPU sageduse piirang Texture Aware&#x27;i jaoks Nano ja Orin Nano mudelitel

Texture Aware&#x27;i debayer kasutab GPU neurovõrgu järeldamist, mis võib madala võimsusega Jetson-mudelitel (10–15 W klass) täis-GPU taktsagedusel põhjustada **ülevoolu hoiatusi**. Kui edastate `--debayer texture-aware`**Jetson Nano või Orin Nano**, kontrollib CLI
GPU maksimaalset sagedust ja piirab selle **510 MHz**-ni (510000000), kui see on hetkel kõrgem:

* Kui CLI
-l on õigus kirjutada GPU sageduse sysfs-sõlme, **rakendatakse** piirang automaatselt ja kuvatakse kinnitus.
* Kui mitte (vajab root-õigusi), kuvab CLI
täpse käsu `sudo`, et piirangut käsitsi rakendada, ootab hetke, et saaksite selle läbi lugeda, ja jätkab seejärel — töötlemine jätkub, kuid võib kuvada ülevoolu hoiatusi.

Piirangu käsitsi rakendamiseks enne töötlemist:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Suurema võimsusega mudelid (Orin NX 25W, AGX Orin 60W) töötavad täis-GPU-kiirusel; piirangut ei rakendata. Standardne debayer ei käivita piirangut ühelgi mudelil.

{% hint style="info" %}
**„Texture Aware“ töötab Jetsonil alati ühe pildiga korraga.** Iga töötaja vajaks oma CUDA-konteksti (~1 GB) ning oma koopiat müra eemaldamise mudelist, mida ühtne mälu ei suuda toetada — seega on Jetsonil Texture Aware&#x27;i töötlemistee seotud ühe töötajaga, kelle GPU-juurdepääs on serialiseeritud. Eeldage, et Texture Aware on igal Jetsonil märkimisväärselt aeglasem kui Standard. Texture Aware kehtib Survey3
-piltide puhul: LATTICE-pildid töödeldakse alati Standard-debayeriga. Kasutatava CUDA-GPU puudumisel kasutatakse töötlemisel Standard-režiimi ning tagapõhi logib ühe `[DEBAYER] … needs a CUDA GPU …`-rea, milles selgitatakse põhjust (näidatud koos `chloros-cli --verbose`-ga).
{% endhint %}

***

## Soojusjuhtimine

Jetson-seadmetel on piiratud soojusvaru, eriti suletud ruumides või õhus kasutamisel. Chloros
jälgib SoC-i temperatuuri ja piirab partiide suurust automaatselt:

| Temperatuur         | Tegevus                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normaalne töö — täis töötlemiskiirus          |
| **70 °C** (hoiatus)  | Partii suurus väheneb järk-järgult (100% → 50% vahemikus 70 °C kuni 80 °C) |
| **80 °C** (kriitiline) | Agressiivne piiramine (50% → 0% vahemikus 80 °C kuni 90 °C) |
| **90 °C** (väljalülitamine) | GPU töötlemise täielik peatamine — vajalik jahutamine |

{% hint style="warning" %}
**Tagage piisav ventilatsioon ja soojuse ärajuhtimine** pikaajaliseks töötlemiseks, eriti suletud välitingimustes kasutatavates korpustes või õhusõidukites. Soojuspiirang vähendab töötlemisvõimsust riistvara kaitsmiseks.
{% endhint %}

***

## Mäluhaldus

Jetson-seadmed kasutavad **ühtset mälu** — GPU ja CPU jagavad sama füüsilist RAM-mälu. Näidatud VRAM (nt ~15,3 GB Orin NX 16 GB-l) ei ole GPU-le eraldatud mälu; see on sama RAM-mälu, mida kasutavad operatsioonisüsteem ja kõik muud protsessid.

### Vahetusmälu hoiatus ja soovitused

Enne töötlemist Jetsonil loendab CLI
teie sisendkaustas olevaid RAW-pilte (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-eelvaateid ei arvestata), hindab käivitamiseks vajaliku mälu tippmahtu ning **hoiatab enne käivitamist**, kui RAM + vahemälu on tõenäoliselt ebapiisav. Hoiatuse pealkirjaks on `LOW MEMORY WARNING - Jetson Detected`, see kuvab piltide arvu, RAM-i, praeguse vahemälumahu ja hinnangulise tippmahu ning annab seejärel täpsed käskud `fallocate` / `chmod` / `mkswap` / `swapon`, mis on mõõdetud just (mitte kunagi väiksem kui 8 GB). Süsteem peatub mõneks sekundiks, et sõnum ei kaoks kerimisajal, seejärel jätkub töötlemine.**Hoiatuses kasutatud mäluhinnangud:**

| Debayer-režiim | Baas | Pildi kohta |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Tekstuuritundlik | ~2,5 GB (mudel + Python-käitamiskeskkond) | ~15 MB |

Hoiatus käivitub, kui hinnanguline tippväärtus ületab RAM-i + vahetusmälu miinus 1 GB ohutusvaru, ning arvestatakse ainult **failipõhist** vahetusmälu — ka ainult zram-i kasutav seadistus märgistatakse ikkagi.

Vahemälu käsitsi lisamiseks (näide: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### OOM (Out of Memory) käsitlemine

Töötlemise ajal jälgib Chloros
GPU mälu kasutust ja vähendab jõudlust sujuvalt, selle asemel et krahhi minna:

1. Kui GPU mälu kasutamine ületab **85%**, vähendatakse partiide suurust ennetavalt
2. Kui mälu puudumise juhtum ikkagi esineb, **vähendatakse** partiide suurust poole võrra ning iga järgneva OOM-juhtumi korral veel kord poole võrra; iga järgnev edukas parti viib selle karistuse ühe sammu võrra tagasi
3. Pideva koormuse korral lülitub töötlemisahel `fused_gpu`-ilt mälu säästvale `tiled_gpu`-rajale ning viimase abinõuna CPU-töötlemisele

Kuna GPU jagab Jetsoni RAM-mälu, piirab RAM koos vahemäluga endiselt töökäiku — eespool mainitud vahemäluhoiatus on märk, millele tuleb tähelepanu pöörata.

***

## Kasutuselevõtt välitingimustes

### Võimsusega seotud kaalutlused

| Jetsoni mudel     | Tüüpiline võimsustarbimine | Märkused                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC-pistik          |
| Jetson Orin NX   | 10–25 W             | DC-pistik          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD või DC-pistik |

Planeerige toitevaru pikaajaliseks töötlemiseks — suurim võimsustarbimine esineb GPU-mahuka Thread 3 (töötlemine) käigus.

### Salvestusruumi soovitused

* **NVMe SSD** on arm64-rakenduste puhul väga soovitatav
* SD-kaardid on töötlemiseks liiga aeglased — kasutage neid ainult käivitusandmekandjana
* Arvestage töödeldud väljundi jaoks 2–3 korda suurema ruumiga kui toores pildimahu suurus

### Peata töö SSH
kaudu

Chloros
-CLI
on ideaalne peata Jetsoni kasutusjuhtumite jaoks:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Pidevalt töötav tagapõhi LATTICE / DAQ-E ajasünkroniseerimiseks

Kui teie Jetson juhib LATTICE-kaameraid või DAQ-E valgussensoreid ilma kuvarita, aktiveerige backend-systemd-teenus **oma kasutajana**, et PTP-grandmaster töötaks pidevalt ja SDK
-skriptid leiaksid töötava backendi (seade on paigaldatud, kuid vaikimisi mitte aktiveeritud):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Seejärel töötab backend teie nimel: selle poolt salvestatud andmed kuuluvad teile ja see kasutab teie `chloros-cli login`-i. Vaata [Linux
i paigaldusjuhendit](linux-installation.md#always-on-ptp-for-headless-hosts) täpsemate üksikasjade kohta, sealhulgas root-kasutaja `chloros-backend.service` ja kuidas pakett muudab PTP-pordid 319/320 siduvaks ilma root-õigusteta.

### Automatiseeritud töötlemine systemd abil

Looge automaatseks töötlemiseks systemd-teenus:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

`chloros-cli process` lõpetab töötamise nullist erineva tulemusega, kui tooteid taotlenud käivitus ei salvesta ühtegi pilti, seega on systemd-i veatase seire seisukohalt oluline.

Kasutage koos systemd-ajastiga planeeritud töötlemiseks:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Näidistöövoogud

### Jetsoni põhitöötlus

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### Python
i SDK
Jetsonil

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Mitme lennu kogumitöötlus

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Soovitatavad Jetson-süsteemid välitingimustes kasutamiseks

Välitingimustes ja õhus kasutamiseks kaaluge järgmisi Jetson Orin NX 16 GB kandjaplaadi variante:

* **Õhus/droon**: vibratsioonikindlad (MIL-STD) süsteemid, kerged (alla 300 g), passiivse jahutusega
* **Karmid välitingimused**: IP67/IP69K veekindlad korpused, millel on PoE GigE kaamera ühenduvus
* **Minimaalne/eelarvesõbralik**: arendajate komplektid lisakorpustega

Võtke ühendust [MAPIR
i tugiteenistusega](https://www.mapir.camera/community/contact), et saada konkreetseid riistvarasoovitusi just teie kasutusolukorra jaoks.

***

## Järgmised sammud

* [Linux
i paigaldamine](linux-installation.md) — üldised Linux
i paigaldamise üksikasjad
* [Dünaamiline arvutusvõimsuse kohandamine](../processing-architecture/dynamic-compute-adaptation.md) — Täielik arvutusstrateegia juhend
* [Töötlemispipeline](../processing-architecture/processing-pipeline.md) — 4-niitilise töötlemispipeline&#x27;i mõistmine
* [CLI: käsurea](../CLI.md) — CLI
juhend
* [API
: Python
SDK
](../api-python-sdk.md) — SDK
juhend
* [CLI
-juhend](../reference/cli-reference.md) ja [SDK
-juhend](../reference/sdk-reference.md) — Kõik käskud ja API
-d versioonis 1.3.0
