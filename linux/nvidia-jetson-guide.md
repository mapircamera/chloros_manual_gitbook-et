# NVIDIA Jetsoni juhend

Chloros võimaldab NVIDIA Jetsonil töötada multispektraalsete piltidega äärearvutites – välitingimustes, droonidel ja kaugpaikades. Chloros tuvastab automaatselt teie Jetsoni mudeli ja optimeerib töötlemisstrateegia vastavalt teie riistvarale.

***

## Toetatud Jetson-mudelid

| Mudel                | RAM            | Töötlemisstrateegia                                   | Soovitatav kasutus                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB jagatud | `GPU_PARALLEL` (4 töötajat)                            | Maksimaalne jõudlus, suured andmekogumid                      |
| **Jetson Orin NX**   | 8–16 GB jagatud  | `GPU_PARALLEL` (3 töötajat, 16 GB) / `GPU_SINGLE` (8 GB) | Esmane soovitus õhust ja välitingimustes kasutamiseks |
| **Jetson Orin Nano** | 8 GB jagatud     | `GPU_SINGLE` (1 töötaja)                               | Algastme servaarvutus                                 |
| **Jetson Nano**      | 4–8 GB jagatud   | `GPU_SINGLE` (1 töötaja)                               | Algastme, piiratud mäluga                          |

{% hint style="info" %}
**Vanemad Jetson-mudelid** (TX2, TX1, Xavier NX) ei pruugi olla toetatud. Jõudlus sõltub kättesaadavast GPU-mälust ja CUDA-võimalustest.
{% endhint %}

***

## Nõuded

* **JetPack 6.x** (soovitatav on uusim versioon)
* **NVIDIA CUDA** (kuulub JetPacki juurde)
* **Chloros+ litsents** (vajalik CLI/SDK juurdepääsuks)

## Paigaldamine

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Üldised Linux paigaldamise üksikasjad leiate [Linux paigaldamisjuhendist](linux-installation.md).

***

## Dünaamiline arvutusvõimsuse kohandamine Jetsonil

Chloros tuvastab automaatselt teie Jetson-mudeli ja valib optimaalse töötlemisstrateegia. **Käsitsi häälestamine ei ole vajalik.**

### Kuidas see töötab

Käivitamisel koostab Chloros teie süsteemi profiili:

1. **Tuvastab Jetson-mudeli** `/proc/device-tree/model` kaudu
2. **Loeb kättesaadava GPU/jagatud mälu**

3.**Valib töötlemisstrateegia** (`GPU_PARALLEL`, `GPU_SINGLE` või `CPU_PARALLEL`)
4. **Määrab töötajate arvu, torujuhtme tüübi ja mälu eraldamise** automaatselt

### Mudeli-põhine käitumine

| Jetson mudel                | Strateegia       | Töötajad | Torujuht                       | Paralleelsus |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (mälu-efektiivne) | Järjestatud  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Järjestatud  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serialiseeritud  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (täielik GPU-tee)    | Paralleelne  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Paralleelne  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** on ideaalne valik servarakenduste jaoks – see kasutab strateegiat `GPU_PARALLEL` koos 3 samaaegse töötajaga, pakkudes tõelist paralleelset GPU-töötlust kompaktse vormi juures.
{% endhint %}

Platvormide peamine erinevus on **mälu**. 8 GB jagatud mäluga Jetson Nano peab töötlema pilte ükshaaval, kasutades mälusäästlikku tiled-lähenemist, samas kui 16 GB mäluga Orin NX suudab töötada GPU-s samaaegselt 3 pilti, kasutades suurema läbilaskevõimega fused pipeline&#x27;i.

Täieliku arvutusliku kohandamise viite leiate [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

***

## Soojusjuhtimine

Jetson-seadmetel on piiratud soojusvaru, eriti suletud või õhus kasutamisel. Chloros sisaldab automaatset soojuse jälgimist ja piiramist:

| Temperatuur         | Tegevus                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normaalne töö — täis töötlemiskiirus          |
| **70 °C** (hoiatus)  | Vähendab automaatselt partii suurust                   |
| **80 °C** (kriitiline) | Agressiivne piiramine — madalam samaaegsus         |
| **90 °C** (väljalülitamine) | Peatab GPU töötlemise täielikult — vajalik jahutamine |

{% hint style="warning" %}
**Tagage piisav ventilatsioon ja jahutus** püsiva töötlemise tagamiseks, eriti suletud välistingimustes või õhusüsteemides. Termiline piiramine vähendab töötlemise läbilaskevõimet, et kaitsta riistvara.
{% endhint %}

***

## Mäluhaldus

Jetson-seadmed kasutavad **ühtset mälu** — GPU ja CPU jagavad sama füüsilist RAM-i. See tähendab, et näidatud VRAM (nt 15,3 GB Orin NX 16 GB-l) ei ole GPU-le pühendatud mälu; seda jagatakse operatsioonisüsteemi ja teiste protsessidega.

### Swap-soovitused

Suurte andmekogumite või tekstuuritundliku debayeri töötlemise puhul võib Chloros soovitada swap-ruumi loomist:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Mälukasutuse hinnangud ühe pildi kohta:**

* Standardne debayer: ~10 MB ühe pildi kohta
* Tekstuuritundlik debayer: ~15 MB ühe pildi kohta

Chloros arvutab vajaliku mälu automaatselt andmekogumi suuruse põhjal ja hoiatab, kui vahetusmälu on soovitatav.

### OOM (mälu puudus) varuplaan

Kui töötlemise käigus tuvastatakse mälu puudus:

1. Chloros vähendab automaatselt GPU töötajate arvu
2. Vahetab `fused_gpu`-i torustiku vastu `tiled_gpu`-i torustiku (mälu säästlikum)
3. Jätkab töötlemist vähendatud läbilaskevõimega, selle asemel et kokku jooksma

***

## Kasutuselevõtt

### Võimsusega seotud kaalutlused

| Jetson mudel     | Tüüpiline võimsustarbimine | Märkused                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 W              | USB-C või barrel-pistik    |
| Jetson Orin Nano | 7–15 W              | DC barrel-pistik          |
| Jetson Orin NX   | 10–25 W             | DC barrel-pistik          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD või barrel-pistik |

Planeerige oma võimsuse eelarve püsivaks töötlemiseks — suurim võimsustarbimine toimub GPU-mahuka Thread 3 (töötlemine) ajal.

### Soovitused salvestusruumi kohta

* **NVMe SSD** on arm64-rakenduste jaoks väga soovitatav
* SD-kaardid on töötlemiseks liiga aeglased — kasutage neid ainult käivitusandmekandjana
* Planeerige töödeldud väljundi jaoks 2–3 korda suurem ruum kui toores pildifaili suurus

### Peata töö SSH kaudu

Chloros CLI on ideaalne peata Jetsoni rakenduste jaoks:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Automatiseeritud töötlemine systemd abil

Loo automatiseeritud töötlemiseks systemd-teenus:

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

Ühenda systemd-ajastiga planeeritud töötlemiseks:

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

## Näite-töövoogud

### Jetsoni põhitöötlus

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK Jetsonil

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
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
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Soovitatavad Jetson-süsteemid välitingimustes kasutamiseks

Välitingimustes ja õhus kasutamiseks kaaluge järgmisi Jetson Orin NX 16 GB kandjaplaadi valikuid:

* **Õhus/droon**: vibratsioonikindlad (MIL-STD) süsteemid, kerged (alla 300 g), passiivne jahutus
* **Karm välitingimused**: IP67/IP69K veekindlad korpused PoE GigE kaamera ühenduvusega
* **Minimaalne/eelarveline**: Arendajate komplektid lisakorpustega

Võtke ühendust [MAPIR tugiteenistusega](https://www.mapir.camera/community/contact), et saada konkreetseid riistvarasoovitusi teie kasutusolukorra jaoks.

***

## Järgmised sammud

* [Linux paigaldamine](linux-installation.md) — Üldised Linux paigaldamise üksikasjad
* [Dünaamiline arvutusvõimsuse kohandamine](../processing-architecture/dynamic-compute-adaptation.md) — Täielik arvutusstrateegia viide
* [Töötlemispipeline](../processing-architecture/processing-pipeline.md) — 4-niitilise pipelini mõistmine
* [CLI : käsurea](../CLI.md) — Täielik CLI viide
* [API : Python SDK](../api-python-sdk.md) — Täielik SDK viide
