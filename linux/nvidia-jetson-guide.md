# NVIDIA Jetsoni juhend

Chloros võimaldab NVIDIA Jetsonil multispektraalset pilditöötlust servaseadmetes – välitingimustes, droonidel ja kaugpaigaldistes. Chloros 1.2.0 tuvastab käivitamisel teie Jetson-mudeli ja optimeerib töötlemisstrateegia vastavalt leitud riistvarale. **Käsitsi häälestamist ei ole vaja.**

***

## Toetatud Jetson-mudelid

| Mudel                | RAM            | Töötlusstrateegia                                     | Soovitatav kasutus                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB jagatud | `GPU_PARALLEL` (2 töötajat)                              | Maksimaalne jõudlus, suured andmekogumid                      |
| **Jetson Orin NX**   | 8–16 GB jagatud | `GPU_PARALLEL` (2 töötajat, 16 GB) / `GPU_SINGLE` (8 GB)   | Esmane soovitus õhust ja välitingimustes kasutamiseks |
| **Jetson Orin Nano** | 8 GB jagatud     | `GPU_SINGLE` (1 tööprotsess, järjestikune)                     | Algastme servaarvutus                                 |

{% hint style="info" %}
Pakett Linux arm64 nõuab **JetPack 6**, mis on saadaval Jetson Orin-seerias. Vanemad mudelid (Nano, TX2, Xavier NX) ei saa JetPack 6-t käivitada ja neid ei toeta praegune pakett.
{% endhint %}

***

## Nõuded

* **JetPack 6.x** (soovitatav on uusim versioon)
* **NVIDIA CUDA** (kuulub JetPacki juurde)
* **Tasuline Chloros+ pakett** — Copper-tase või kõrgem (nõutav kõigi CLI/SDK juurdepääsude jaoks; rakendatakse serveri poolel)

## Paigaldamine

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Üldiste Linux paigaldusjuhiste, failide asukohtade ja veaotsingute kohta vaata [Linux paigaldusjuhendit](linux-installation.md).

{% hint style="info" %}
**Paigutage lahtipakkimiskataloog kiirele andmekandjale.** Kompileeritud binaarfailid pakivad end iga käivitamise ajal ajutisse kataloogi lahti — SD-kaardilt on see piinavalt aeglane. Chloros kasutab automaatselt `/mnt/ssd/tmp`-i, kui see on olemas; muul juhul seadista `TMPDIR` oma NVMe-kettal asuvale teekonnale (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dünaamiline arvutusvõimsuse kohandamine Jetsonil

### Kuidas see toimib

Käivitamisel koostab Chloros teie süsteemi profiili:

1. **Tuvastab Jetsoni mudeli** `/proc/device-tree/model` kaudu
2. **Loeb kättesaadavat jagatud GPU/CPU mälu** (Jetson kasutab ühtset mälu)
3. **Valib töötlemisstrateegia** (`GPU_PARALLEL`, `GPU_SINGLE` või `CPU_PARALLEL`)
4. **Määrab töötajate arvu, töötlemisliini tüübi ja mälueralduse** automaatselt

Otsus sõltub **jagatud RAM-i kogumahust**, mitte mudeli nimest:

* **Kogumahuga alla 12 GB RAM-i**(kõik 8 GB Jetsonid): `GPU_SINGLE`**ühe töötajaga — sihipärane järjestikune töötlemine**. Mälu on samaaegsete töötajate jaoks liiga piiratud, seega töödeldakse pilte ükshaaval.**8 GB või vähem** mäluga Jetsonitel jätab niit 3 töötajate rühma täielikult vahele ja teostab iga pildi töötlemise protsessi käigus.
* **12 GB või rohkem**(Orin NX 16 GB, AGX Orin): ühtne mälu vastab `GPU_PARALLEL` nõuetele, kuid töötajate arv on**Jetsonil piiratud kahega** — GPU, tööprotsesside RAM ja iga töötaja CUDA-kontekstid kasutavad kõik sama jagatud mälupooli, mistõttu rohkemate töötajate korral on oht, et tekivad mälupuudusest tingitud vead.

Automaatset valikut saab ümber kirjutada keskkonnamuutujaga `CHLOROS_STRATEGY` — vt [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Mudelipõhine käitumine

| Jetsoni mudel                | Strateegia       | Tööprotsessid | Täitmine                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Järjestikune protsessisisene tsükkel (`tiled_gpu` mälupuuduse korral) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Järjestikune protsessisisene tsükkel                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Paralleelsed tööprotsessid, `fused_gpu` tee  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Paralleelsed tööprotsessid, `fused_gpu` tee  |

Platvormide peamine erinevus on **mälu**. 8 GB Jetson peab suure koormuse korral töötlema pilte ükshaaval, kasutades mälu säästvat tiled-lähenemist, samas kui 16 GB+ Orin suudab GPU kaudu töötleda samaaegselt 2 pilti, kasutades suurema läbilaskevõimega fused pipeline&#x27;i.

### GPU eelarve mudeli kohta

Iga Jetson-mudelil on ka riistvaraprofiil, mis piirab, kui palju jagatud mälupoolist töötlemine võib hõivata, ning skaleerib partiide suurusi:

| Mudel | GPU eelarve ülempiir | Partiisuuruse kordaja | Süsteemile/ekraanile reserveeritud |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

Profiili kohandab tuvastatud RAM-mälu maht: kui Jetson teatab **16 GB või rohkem**, suurendatakse selle partii kordajat ×1,2 võrra. Kordajate kohaldamise eelne partii baasmaht on 8 pilti.

Täieliku arvutusvõimsuse kohandamise juhendi leiate [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

***

## GPU sageduse piirang Texture Aware&#x27;i jaoks Nano ja Orin Nano mudelitel

Texture Aware&#x27;i debayer kasutab GPU neurovõrgu järeldamist, mis võib madala võimsusega Jetson-mudelitel (10–15 W klass) täis-GPU taktsagedusel põhjustada **ülevoolu hoiatusi**. Enne Texture Aware&#x27;i töötlemist**Jetson Nano või Orin Nano**seadmel kontrollib Chloros GPU maksimaalset sagedust ja piirab selle**510 MHz**-ni (510000000), kui see on hetkel kõrgem:

* Kui CLI suudab kirjutada GPU sageduse sysfs-sõlme, **rakendub piirang automaatselt** ja kuvatakse kinnitus.
* Kui mitte (vajab root-õigusi), kuvab CLI täpse `sudo`-käsu piirangu käsitsi rakendamiseks, ootab hetke, et saaksite selle läbi lugeda, ja jätkab seejärel — töötlemine jätkub, kuid võib kuvada ülevoolu hoiatusi.

Piirangu käsitsi rakendamiseks enne töötlemist:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Suurema võimsusega mudelid (Orin NX 25W, AGX Orin 60W) töötavad täis-GPU-kiirusel; piirangut ei rakendata. Standardne debayer ei käivita piirangut ühelgi mudelil.

{% hint style="info" %}
**„Texture Aware” töötab Jetsonil alati ühe pildiga korraga.** Iga töötaja vajaks oma CUDA-konteksti (~1 GB) ning oma koopiat müra eemaldamise mudelist, mida ühtne mälu ei suuda endale lubada — seega on Jetsonil Texture Aware&#x27;i töötlemistee kinnitatud ühele töötajale, kusjuures GPU-le juurdepääs on serialiseeritud. Eeldage, et Texture Aware on igal Jetsonil märkimisväärselt aeglasem kui Standard.
{% endhint %}

***

## Soojusjuhtimine

Jetson-seadmetel on piiratud soojusvaru, eriti suletud ruumides või õhus kasutamisel. Chloros jälgib SoC temperatuuri ja reguleerib partiide suurust automaatselt:

| Temperatuur         | Tegevus                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normaalne töö — täis töötlemiskiirus          |
| **70 °C** (hoiatus)  | Partii suurus väheneb järk-järgult (100% → 50% vahemikus 70 °C kuni 80 °C) |
| **80 °C** (kriitiline) | Agressiivne piiramine (50% → 0% vahemikus 80 °C kuni 90 °C) |
| **90 °C** (väljalülitamine) | GPU töötlemine peatatakse täielikult — vajalik jahutamine |

{% hint style="warning" %}
**Tagage piisav ventilatsioon ja soojuse ärajuhtimine** pikaajalise töötlemise tagamiseks, eriti suletud välitingimustes kasutatavates korpustes või õhusõidukites. Soojuspiirang vähendab töötlemisvõimsust, et kaitsta riistvara.
{% endhint %}

***

## Mäluhaldus

Jetson-seadmed kasutavad **ühtset mälu** — GPU ja CPU jagavad sama füüsilist RAM-i. Näidatud VRAM (nt ~15,3 GB Orin NX 16 GB-l) ei ole GPU-le eraldatud mälu; see on sama RAM, mida kasutavad operatsioonisüsteem ja kõik muud protsessid.

### Vahetusmälu hoiatus ja soovitused

Enne töötlemist Jetsonil loendab CLI teie sisendkaustas olevaid RAW-pilte (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-eelvaateid ei loeta), hindab töötlemiseks vajaliku mälu tippmahtu ning **hoiatab enne käivitamist**, kui RAM + vahemälu on tõenäoliselt ebapiisav. Hoiatuse pealkirjaks on `LOW MEMORY WARNING - Jetson Detected`, see kuvab piltide arvu, RAM-i, praeguse vahemälumahu ja hinnangulise tippmahu, seejärel annab täpsed `fallocate` / `chmod` / `mkswap` / `swapon` käsud, mis on suuruselt kohandatud teie projektile (mitte kunagi väiksemad kui 8 GB). Programm peatub mõneks sekundiks, et sõnum ei kaoks kerimisajal, seejärel jätkub töötlemine.**Hoiatuses kasutatud mäluhinnangud:**

| Debayer-režiim | Baas | Pildi kohta |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Tekstuuriteadlik | ~2,5 GB (mudel + Python-i käitamisaja mälu) | ~15 MB |

Hoiatus käivitub, kui hinnanguline tippväärtus ületab RAM-i + vahemälu miinus 1 GB ohutusvaru, ning arvestatakse ainult **failipõhist** vahemälu — ka ainult zram-i kasutav seadistus märgistatakse ikkagi.

Vahemälu käsitsi lisamiseks (näide: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

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
```### OOM (Out of Memory) käsitlemine

Töötlemise ajal jälgib Chloros GPU mälu kasutust ja vähendab jõudlust sujuvalt, selle asemel et krahhi minna:

1. Kui GPU mälu kasutamine ületab **85%**, vähendatakse partiide suurust ennetavalt
2. Kui mälu puudumise sündmus ikkagi esineb, **vähendatakse** partiide suurust poole võrra ning iga järgneva mälu puudumise korral veel poole võrra; iga järgnev edukas parti viib selle karistuse ühe sammu võrra tagasi
3. Pideva koormuse korral lülitub töötlemisahel `fused_gpu`-ilt üle mälu säästvale `tiled_gpu`-teele ning viimase abinõuna CPU-töötlemisele

***

## Kasutuselevõtt

### Võimsusega seotud kaalutlused

| Jetsoni mudel     | Tüüpiline võimsustarbimine | Märkused                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC-barrel-pistik          |
| Jetson Orin NX   | 10–25 W             | DC-barrel-pistik          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD või barrel-pistik |

Planeerige toitevaru pikaajaliseks töötlemiseks — suurim voolutarbimine esineb GPU-mahuka Thread 3 (töötlemine) käigus.

### Soovitused salvestusruumi kohta

* **NVMe SSD** on arm64-rakenduste puhul väga soovitatav
* SD-kaardid on töötlemiseks liiga aeglased — kasutage neid ainult käivitusandmekandjana
* Arvestage töödeldud väljundi jaoks 2–3 korda suurema mahuga kui toores pildifail

### Peata töö SSH kaudu

Chloros CLI on ideaalne ekraanita Jetson-lahenduste jaoks:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Pidevalt töötav tagapõhi LATTICE / DAQ-E ajasünkroniseerimiseks

Kui teie Jetson juhib LATTICE-kaameraid või DAQ-E valgusandureid ilma kuvarita, lülitage sisse backend-süsteemi teenus systemd, et PTP-grandmaster töötaks pidevalt (komponent on paigaldatud, kuid vaikimisi välja lülitatud):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Vaadake [Linux paigaldusjuhendit](linux-installation.md#always-on-ptp-for-headless-hosts) täpsemate üksikasjade saamiseks, sealhulgas selle kohta, kuidas pakett muudab PTP-pordid 319/320 siduvaks ilma root-õigusteta.

### Automaatne töötlemine systemd abil

Looge automaatse töötlemise jaoks systemd-teenus:

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

`chloros-cli process` lõpetab töö nullist erineva tulemusega, kui tooteid taotlenud käivitus ei kirjuta ühtegi pilti, seega on systemd-i veatase seire seisukohast oluline.

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
    --indices NDVI NDRE GNDVI
```

### Python ja SDK Jetsonil

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

### Mitme lennu partii töötlemine

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Soovitatavad Jetson-süsteemid välitingimustes kasutamiseks

Välitingimustes ja õhus kasutamiseks kaaluge järgmisi Jetson Orin NX 16 GB kandjaplaadi variante:

* **Õhus/droon**: Vibratsioonikindlad (MIL-STD), kerged (alla 300 g) ja passiivse jahutusega süsteemid
* **Kestvad välitingimustes**: IP67/IP69K veekindlad korpused, millel on PoE GigE kaamera ühenduvus
* **Minimaalne/eelarvesõbralik**: Arendajate komplektid lisakorpustega

Võtke ühendust [MAPIR tugiteenistusega](https://www.mapir.camera/community/contact), et saada konkreetseid riistvarasoovitusi teie kasutusolukorra jaoks.

***

## Järgmised sammud

* [Linux paigaldamine](linux-installation.md) — üldised Linux paigaldamise üksikasjad
* [Dünaamiline arvutusvõimsuse kohandamine](../processing-architecture/dynamic-compute-adaptation.md) — Täielik arvutusstrateegia juhend
* [Töötlemispipeline](../processing-architecture/processing-pipeline.md) — 4-niitilise pipelini mõistmine
* [CLI : käsurida](../CLI.md) — CLI juhend
* [API : Python SDK](../api-python-sdk.md) — SDK juhend
* [CLI viide](../reference/cli-reference.md) ja [SDK viide](../reference/sdk-reference.md) — Täielikud käskude/API loetelud versioonile 1.2.0
