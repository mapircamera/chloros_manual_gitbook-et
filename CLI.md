# CLI : Käsurea

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** pakub võimsat käsurea juurdepääsu Chloros pilditöötlusmootorile, võimaldades automatiseerimist, skriptimist ja peata töötamist teie pilditöötlusvoogudes.

### Peamised omadused

* 🚀 **Automatiseerimine** – mitme andmekogumi skriptipõhine paketttöötlus
* 🔗 **Integreerimine** – integreerimine olemasolevatesse töövoogudesse ja protsessidesse
* 💻 **Käsitsi juhtimata töö** – töötab ilma graafilise kasutajaliideseta
* 🌍 **Mitmekeelne** – toetab 38 keelt
* ⚡ **Paralleelne töötlemine** – [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) optimeerib automaatselt teie riistvara jaoks

### Nõuded

| Nõue          | Üksikasjad                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operatsioonisüsteem** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Litsents**          | Chloros+ ([vajalik tasuline pakett](https://cloud.mapir.camera/pricing)) |
| **Mälu**           | Minimaalselt 8 GB RAM-i (soovitatavalt 16 GB)                                  |
| **Internet**         | Vajalik litsentsi aktiveerimiseks                                     |
| **Kettaruum**       | Sõltub projekti suurusest                                              |

{% hint style="warning" %}
**Litsentsinõuded**: CLI nõuab tasulist Chloros+ tellimust. Standard (tasuta) paketid ei anna juurdepääsu CLI-ile. Uuendamiseks külastage [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Kiirstart

### Paigaldamine

#### Windows

CLI on automaatselt lisatud Chloros paigaldajasse:

1. Laadige alla ja käivitage **Chloros Installer.exe**

2. Viige paigaldusviisard lõpule
3. CLI paigaldati: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Paigaldaja lisab automaatselt `chloros-cli` teie süsteemi PATH-i. Käivitage pärast paigaldamist terminal uuesti.
{% endhint %}

#### Linux

Paigaldage oma arhitektuurile sobiv `.deb` pakett:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Linux üksikasjaliku seadistamise kohta vaadake [Linux paigaldamine](linux/linux-installation.md).

### Esmane seadistamine

Enne CLI kasutamist aktiveerige oma Chloros+ litsents:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Põhikasutus

Kausta töötlemine vaikimisi seadetega:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Käskude viide

### Üldine süntaks

```
chloros-cli [global-options] <command> [command-options]
```

***

## Käskud

### `process` - Töötle pilte

Töötle kausta pilte kalibreerimisega.

**Süntaks:**

```bash
chloros-cli process <input-folder> [options]
```

**Näited:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Käskude töötlemise valikud

| Valik                | Tüüp    | Vaikimisi        | Kirjeldus                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Tee    | _Nõutav_     | Kausta, mis sisaldab RAW/JPG multispektraalpilte                                         |
| `-o, --output`        | Tee    | Sama kui sisend  | Töödeldud piltide väljundkausta                                                     |
| `-n, --project-name`  | String  | Automaatselt genereeritud | Kohandatud projekti nimi                                                                    |
| `--vignette`          | Lipuke    | Lubatud        | Luba vinjeti korrigeerimine                                                             |
| `--no-vignette`       | Lipuke    | -              | Keela vinjeti korrigeerimine                                                            |
| `--reflectance`       | Lipuke    | Lubatud        | Lubada peegelduskalibreerimist                                                         |
| `--no-reflectance`    | Lipuke    | -              | Keelata peegelduskalibreerimist                                                        |
| `--ppk`               | Lipuke    | Lülitatud välja       | Rakenda PPK-korrektsioone .daq valgusanduri andmetest                                      |
| `--format`            | Valik  | TIFF (16-bitine)  | Väljundvorming: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Täisarv | Auto           | Kalibreerimispaneeli tuvastamise minimaalne sihtmärgi suurus pikslites                          |
| `--target-clustering` | Täisarv | Auto           | Sihtmärkide klastrite künnis (0–100)                                                    |
| `--debayer`           | Valik  | `standard`     | Debayeri meetod: `standard` või `texture-aware` (ainult Chloros+)                          |
| `--target`, `--targets` | Lipuke  | Keelatud       | Otsi kalibreerimise sihtmärke ainult alamkaustast „target” või „targets” (kiirendab töötlemist) |
| `--indices`           | Nimekiri    | Puudub           | Arvutatavad taimestiku indeksid (nt `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | String  | Puudub           | Lukusta kaamera mudeli ekspositsioon (pin 1)                                                 |
| `--exposure-pin-2`    | String  | Puudub           | Kaamera mudeli säriaja lukustamine (pin 2)                                                 |
| `--recal-interval`    | Täisarv | Auto           | Kalibreerimise intervall sekundites                                                      |
| `--timezone-offset`   | Täisarv | 0              | Ajavööndi nihke tundides                                                               |

***

### `login` - Konto autentimine

Logige sisse oma Chloros+ kasutajatunnustega, et võimaldada CLI töötlemist.

**Süntaks:**

```bash
chloros-cli login <email> <password>
```

**Näide:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Erimärgid**: Kasutage ühekordseid jutumärke paroolide ümber, mis sisaldavad märke nagu `$`, `!` või tühikuid.
{% endhint %}

**Väljund:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` – Kustuta kasutajatunnused

Kustuta salvestatud kasutajatunnused ja logi oma kontolt välja.

**Süntaks:**

```bash
chloros-cli logout
```

**Näide:**

```bash
chloros-cli logout
```

**Väljund:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK kasutajad**: Python SDK pakub ka programmilist `logout()` meetodit kasutajatunnuste kustutamiseks Python skriptides. Täpsema teabe saamiseks vaadake [Python SDK dokumentatsiooni](api-python-sdk.md#logout).
{% endhint %}

***

### `status` – Litsentsi staatuse kontrollimine

Kuva praegune litsentsi- ja autentimise staatus.

**Süntaks:**

```bash
chloros-cli status
```

**Näide:**

```bash
chloros-cli status
```

**Väljund:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` – Ekspordi edenemise kontrollimine

Jälgib Thread 4 ekspordi edenemist töötlemise ajal või pärast seda.

**Süntaks:**

```bash
chloros-cli export-status
```

**Näide:**

```bash
chloros-cli export-status
```

**Kasutamisjuhtum:** Kutsuge seda käsku töötlemise käigus, et kontrollida ekspordi kulgu.***

### `language` – Liidese keele haldamine

Vaadake või muutke CLI liidese keelt.

**Süntaks:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Näited:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Toetatud keeled (kokku 38)

| Kood    | Keel              | Algne nimi      |
| ------- | --------------------- | ---------------- |
| `en`    | Inglise keel               | English          |
| `es`    | Hispaania keel               | Español          |
| `pt`    | Portugali keel            | Português        |
| `fr`    | Prantsuse keel                | Français         |
| `de`    | Saksa                | Deutsch          |
| `it`    | Itaalia               | Italiano         |
| `ja`    | Jaapani              | 日本語              |
| `ko`    | Korea                | 한국어              |
| `zh`    | Hiina (lihtsustatud)  | 简体中文             |
| `zh-TW` | Hiina (traditsiooniline) | 繁體中文             |
| `ru`    | Vene               | Русский          |
| `nl`    | Hollandi keel                 | Nederlands       |
| `ar`    | Araabia keel                | العربية          |
| `pl`    | Poola keel                | Polski           |
| `tr`    | Türgi keel               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indoneesia keel            | Bahasa Indonesia |
| `vi`    | Vietnami keel            | Tiếng Việt       |
| `th`    | Tai keel                  | ไทย              |
| `sv`    | Rootsi keel               | Svenska          |
| `da`    | Taani keel                | Dansk            |
| `no`    | Norra keel             | Norsk            |
| `fi`    | Soome keel               | Suomi            |
| `el`    | Kreeka keel                 | Ελληνικά         |
| `cs`    | Tšehhi keel                 | Čeština          |
| `hu`    | Ungari             | Magyar           |
| `ro`    | Rumeenia              | Română           |
| `uk`    | Ukraina             | Українська       |
| `pt-BR` | Brasiilia portugali  | Português Brasileiro |
| `zh-HK` | Kantoni keel             | 粵語             |
| `ms`    | Malai keel                 | Bahasa Melayu    |
| `sk`    | Slovaki keel                | Slovenčina       |
| `bg`    | Bulgaaria keel             | Български        |
| `hr`    | Horvaatia keel              | Hrvatski         |
| `lt`    | Leedu keel            | Lietuvių         |
| `lv`    | Läti keel               | Latviešu         |
| `et`    | Eesti keel              | Eesti            |
| `sl`    | Sloveeni keel             | Slovenščina      |

{% hint style="success" %}
**Automaatne püsivus**: Teie keele-eelistus salvestatakse faili `~/.chloros/cli_language.json` ja säilib kõigi sessioonide vältel.
{% endhint %}

***

### `set-project-folder` - Vaikimisi projektikausta määramine

Muuda vaikimisi projektikausta asukohta (jagatud GUI-ga failis Windows).

**Süntaks:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Näited:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` – Näita projektikausta

Kuva praeguse vaikimisi projektikausta asukoht.

**Süntaks:**

```bash
chloros-cli get-project-folder
```

**Näide:**

```bash
chloros-cli get-project-folder
```

**Väljund:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` – Taasta vaikimisi

Taasta projektikausta asukoht vaikimisi.

**Süntaks:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` – Süsteemi diagnostika käivitamine

Käivita 7 diagnostilist kontrolli, et kontrollida süsteemi konfiguratsiooni.

**Süntaks:**

```bash
chloros-cli selftest
```

**Teostatavad diagnostikad:**

1. Versiooni kontroll
2. Pordi kättesaadavus (5000)
3. Backendi käivitamine
4. API ühenduvuse test
5. Süsteemiinfo ja GPU tuvastamine
6. Müra vähendamise mudelite kontroll
7. CUDA kättesaadavuse kontroll

{% hint style="info" %}
**Kasulik veaotsinguks**: Käivitage pärast installimist `selftest`, et kontrollida, kas teie süsteem on õigesti konfigureeritud, eriti Linux/Jetsonil, kus GPU ja CUDA seadistust võib olla vaja kontrollida.
{% endhint %}

***

### `update` – Uuenduste otsimine (ainult Linux)

Otsige ja installige CLI uuendused Linux süsteemidele.

**Süntaks:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Valik    | Kirjeldus                        |
| --------- | ---------------------------------- |
| `--check` | Vaata ainult värskendusi, ära installeeri |

{% hint style="info" %}
See käsk on saadaval ainult Linux-süsteemides. Windows-süsteemides edastatakse värskendused paigaldaja kaudu.
{% endhint %}

***

## Üldised valikud

Need valikud kehtivad kõigi käskude puhul:

| Valik            | Tüüp    | Vaikimisi       | Kirjeldus                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Tee    | Automaatselt tuvastatud | Tagapõhja käivitatava faili tee                       |
| `--port`          | Täisarv | 5000          | Tagapõhja API pordi number                          |
| `--restart`       | Lipuk | -             | Sundida tagapõhja taaskäivitamist (lõpetab olemasolevad protsessid) |
| `--version`       | Lipuk | -             | Näita versiooniinfo ja välju                |
| `--help`          | Lipuk | -             | Näita abiinfo ja välju                   |

{% hint style="info" %}
**Tagapõhja automaatne tuvastamine**: `--backend-exe` tee tuvastatakse automaatselt vastavalt platvormile:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (käsitsi)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Näide globaalsete valikutega:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Töötlemise seadete juhend

### Paralleelne töötlemine ja dünaamiline arvutusvõimsuse kohandamine

Chloros 1.1.0 sisaldab [dünaamilist arvutusvõimsuse kohandamist](processing-architecture/dynamic-compute-adaptation.md) — töötlemismootor **tuvastab automaatselt teie riistvara** ja valib optimaalse strateegia:

| Platvorm | Strateegia | Töötajad | Töötlemisahel | Märkused |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Mälu-efektiivne, serialiseeritud |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Paralleelne GPU-töötlemine |
| **Lauaarvuti 8 GB GPU-ga** | `GPU_SINGLE` | 3 | `tiled_gpu` | Hea lauaarvuti jõudlus |
| **Töölauaarvuti 12 GB+ GPU-ga** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimaalne töölauaarvuti jõudlus |
| **Ainult CPU-ga süsteem** | `CPU_PARALLEL` | tuumad – 1 | `cpu_fallback` | GPU-d ei ole vaja |

{% hint style="success" %}
**Käsitsi konfigureerimist ei ole vaja!** Chloros tuvastab automaatselt teie CPU, GPU, RAM-i ja (Jetsonil) temperatuuriandurid ning konfigureerib seejärel automaatselt optimaalse töötlemisprotsessi.
{% endhint %}

### Debayer-meetodid

| Meetod | CLI Lipuk | Kvaliteet | Kiirus | Litsents |
| --- | --- | --- | --- | --- |
| **Standard (kiire, keskmine kvaliteet)** | `--debayer standard` | Hea | Kiire | Tasuta / Chloros+ |
| **Tekstuuritundlik (aeglane, kõrgeim kvaliteet)** | `--debayer texture-aware` | Kõrgeim | Aeglane | Ainult Chloros+ |

Vaikimisi kasutatav debayer-meetod on **Standard**.**Tekstuuritundlik** meetod kasutab AI/ML müra vähendamise mudelit, et saavutada kõrgeima kvaliteediga tulemus, kuid nõuab Chloros+ litsentsi ja NVIDIA GPU-d.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Vignette-korrektsioon

**Mida see teeb:** Korrigeerib valguse langust pildi servades (kaamerapiltidel tavaliselt esinevad tumedamad nurgad).

* **Vaikimisi sisse lülitatud** – enamik kasutajaid peaks selle sisse jätma
* Kasutage `--no-vignette` selle väljalülitamiseks

{% hint style="success" %}
**Soovitus**: Lülita vinjeti korrigeerimine alati sisse, et tagada ühtlane heledus kogu kaadri ulatuses.
{% endhint %}

### Peegelduskalibreerimine

Muudab tooresensorite väärtused kalibreerimispaneelide abil standardiseeritud peegeldusprotsentideks.

* **Vaikimisi sisse lülitatud** – oluline taimestiku analüüsiks
* Nõuab piltidel kalibreerimispaneele
* Kasutage `--no-reflectance`, et see välja lülitada

{% hint style="info" %}
**Nõuded**: Tagage, et kalibreerimispaneelid on piltidel õigesti valgustatud ja nähtavad, et tagada täpne peegeldusvõime teisendus.
{% endhint %}

### PPK-korrektsioonid

**Mida see teeb:** Rakendab järelkäsitletud kinemaatilisi korrektsioone, kasutades DAQ-A-SD logiandmeid GPS-i täpsuse parandamiseks.

* **Vaikimisi keelatud**
* Kasutage `--ppk` selle lubamiseks
* Nõuab .daq faile projektikausta MAPIR DAQ-A-SD valgusandurist.

### Väljundformaadid

<table><thead><tr><th width="197">Vorming</th><th width="130.20001220703125">Bitide sügavus</th><th width="116.5999755859375">Faili suurus</th><th>Sobib kõige paremini</th></tr></thead><tbody><tr><td><strong>TIFF (16-bitine)</strong> ⭐</td><td>16-bitine täisarv</td><td>Suur</td><td>GIS-analüüs, fotogramm-meetria (soovitatav)</td></tr><tr><td><strong>TIFF (32-bitine, protsent)</strong></td><td>32-bitine ujukomaarv</td><td>Väga suur</td><td>Teaduslik analüüs, uurimistöö</td></tr><tr><td><strong>PNG (8-bitine)</strong></td><td>8-bitine täisarv</td><td>Keskmine</td><td>Visuaalne kontroll, veebis jagamine</td></tr><tr><td><strong>JPG (8-bitine)</strong></td><td>8-bitine täisarv</td><td>Väike</td><td>Kiire eelvaade, pakitud väljund</td></tr></tbody></table>***

## Automaatika ja skriptimine

### PowerShelli kogumitöötlus (Windows)

Töötle mitut andmekogumite kausta automaatselt Windows-is:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows partiiskript (Windows)

Lihtne tsükkel partiitöötluseks Windows-is:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash-patsi töötlemine (Linux)

Töötle mitut andmekogumite kausta Linux-is:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python automatiseerimisskript (platvormiülene)

Täiustatud automatiseerimine veatöötlusega (töötab Windows ja Linux-is):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Töötlemise töövoog

### Standardne töövoog

1. **Sisend**: Kaust, mis sisaldab RAW/JPG pildipaare
2. **Avastamine**: CLI otsib automaatselt toetatud pildifaile
3. **Töötlemine**: Paralleelrežiim skaleerub vastavalt teie CPU tuumadele (Chloros+)
4. **Väljund**: Loob kaameramudeli alamkaustad töödeldud piltidega

### Näide väljundstruktuurist

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Töötlemise aja hinnangud

Tüüpilised töötlemisajad 100 pildi puhul (igaüks 12 MP):

| Platvorm | Režiim | Hinnanguline aeg | Märkused |
| --- | --- | --- | --- |
| **Töölauaarvuti 12 GB+ GPU** | `GPU_PARALLEL` | 5–10 min | Kiireim valik |
| **Töölauaarvuti 8 GB GPU** | `GPU_SINGLE` | 10–15 min | Hea jõudlus |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15–25 min | Äärtearvutus |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30–60 min | Piiratud mälu |
| **Ainult CPU** | `CPU_PARALLEL` | 20–40 min | GPU-d ei ole vaja |

{% hint style="info" %}
**Jõudluse näpunäide**: Töötlemisaeg sõltub piltide arvust, resolutsioonist, debayeri meetodist ja riistvarast. Tekstuuritundlik debayer võtab märkimisväärselt rohkem aega kui standardne. Vaata täpsemaid andmeid [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) alt.
{% endhint %}

***

## Veaotsing

### CLI ei leitud

**Windows viga:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Lahendused:**

1. Kontrollige installimiskohta:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Kasutage täielikku teekonda, kui see ei ole PATH-is:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Lisage PATH-i käsitsi:
   * Avage Süsteemi omadused → Keskkonnamuutujad
   * Muutke PATH-muutujat
   * Lisage: `C:\Program Files\Chloros\resources\cli`
   * Käivitage terminal uuesti

**Linux Viga:**

```
chloros-cli: command not found
```

**Linux Lahendused:**

1. Kontrolli installi:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Laadi oma shell uuesti:

```bash
source ~/.bashrc
```

3. Kontrolli õigusi:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Backend ei õnnestunud käivitada**Viga:**

```

Backend failed to start within 30 seconds
```

**Lahendused:**

1. Kontrollige, kas backend juba töötab (sulgege see esmalt)
2. Kontrollige, et tulemüür ei blokeeri (Windows) või kontrollige pordi kättesaadavust (Linux: `lsof -i :5000`)
3. Proovige teist porti:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Sundige backendit taaskäivitama:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Kontrollige Linux puhul, kas backend-i käivitatav fail on olemas:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Litsentsi- ja autentimise probleemid**Viga:**

```

Chloros+ license required for CLI access
```

**Lahendused:**

1. Veenduge, et teil on aktiivne Chloros+ tellimus
2. Logige sisse oma kasutajatunnuse ja parooliga:

```bash
chloros-cli login user@example.com 'password'
```

3. Kontrollige litsentsi staatust:

```bash
chloros-cli status
```

4. Võtke ühendust tugiteenistusega: info@mapir.camera

***

### Pilte ei leitud**Viga:**

```

No images found in the specified folder
```

**Lahendused:**

1. Veenduge, et kaust sisaldab toetatud formaate (.RAW, .TIF, .JPG)
2. Kontrollige, kas kausta tee on õige (kasutage jutumärke, kui tees on tühikuid)
3. Veenduge, et teil on kaustale lugemisõigused
4. Kontrollige, kas faililaiendid on õiged

***

### Töötlemine seisab või hangub**Lahendused:**

1. Kontrollige vaba kettaruumi (veenduge, et väljundiks on piisavalt ruumi)
2. Sulgege teised rakendused, et vabastada mälu
3. Vähendage piltide arvu (töötlege partiidena)

***

### Port on juba kasutusel**Viga:**

```

Port 5000 is already in use
```

**Lahendused:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## KKK

### K: Kas ma vajan CLI jaoks litsentsi?

**V:**Jah! CLI nõuab tasulist**Chloros+ litsentsi**.

* ❌ Standard (tasuta) pakett: CLI on keelatud
* ✅ Chloros+ (tasuline) pakett: CLI on täielikult lubatud

Telli aadressil: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### K: Kas ma saan kasutada CLI-i serveris, millel puudub graafiline kasutajaliides?**V:** Jah! CLI töötab täielikult ilma graafilise kasutajaliideseta. See on Linux peamine kasutusjuhtum.**Windows server:**
* Windows Server 2016 või uuem
* Visual C++ Redistributable on installitud

**Linux Server:**
* Ubuntu 20.04+ / Debian 11+ (amd64) või JetPack 6 (arm64)
* Paigaldamine `.deb` paketi kaudu

**Mõlemad platvormid:**
* Minimaalselt 8 GB RAM-i (soovitatavalt 16 GB)
* Ühekordne litsentsi aktiveerimine: `chloros-cli login user@example.com 'password'`

***

### K: Kuhu salvestatakse töödeldud pildid?**V:**Vaikimisi salvestatakse töödeldud pildid**samasse kausta kui sisend** kaameramudeli alamkaustadesse (nt `Survey3N_RGN/`).

Kasutage valikut `-o`, et määrata teine väljundkaust:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### K: Kas ma saan töödelda mitut kausta korraga?**V:** Ühe käsuga otseselt mitte, kuid skriptide abil saate kaustu järjestikku töödelda. Vaadake jaotist [Automatiseerimine ja skriptimine](CLI.md#automation--scripting).***

### K: Kuidas salvestada CLI väljund logifaili?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### K: Mis juhtub, kui vajutan töötlemise ajal klahvikombinatsiooni Ctrl+C?**V:** CLI teeb järgmist:

1. Peatab töötlemise sujuvalt
2. Sulgeb taustaprogrammi
3. Lõpetab töö koodiga 130

Osaliselt töödeldud pildid võivad jääda väljundkausta.

***

### K: Kas ma saan CLI töötlemist automatiseerida?**V:** Kindlasti! CLI on loodud automatiseerimiseks. Vaata [Automatiseerimine ja skriptimine](CLI.md#automation--scripting) PowerShelli (Windows), Batchi (Windows), Bash (Linux) ja Python (platvormiülene) näiteid.***

### K: Kuidas kontrollida CLI versiooni?**V:**

```bash
chloros-cli --version
```

**Väljund:**

```

Chloros CLI 1.1.0
```

***

## Abi saamine

### Käsurea abi

Vaadake abiinfo otse CLI-is:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Tugikanalid

* **E-post**: info@mapir.camera
* **Veebisait**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Hinnad**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Täielikud näited

### Näide 1: Põhitöötlus

Töötlus vaikimisi seadetega (vignett, peegeldus):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Näide 2: Kvaliteetne teaduslik väljund

32-bitine ujukomaarv TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Näide 3: Kiire eelvaate töötlemine

8-bitine PNG ilma kalibreerimiseta kiireks ülevaatamiseks:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Näide 4: PPK-korrigeeritud töötlemine

Rakenda PPK-korrektsioone peegeldusega:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Näide 5: Kohandatud väljundkoht

Töötle teise asukohta kindlas formaadis:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Näide 6: Autentimise töövoog

Täielik autentimise voog (sama kõikidel platvormidel):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Näide 7: Mitmekeelne kasutamine

Muuda liidese keelt (sama kõikidel platvormidel):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
