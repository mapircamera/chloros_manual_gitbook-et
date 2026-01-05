# CLI : käsurea

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** pakub võimast käsurea juurdepääsu Chloros pilditöötlusmootorile, võimaldades automatiseerimist, skriptimist ja peata töötamist teie pilditöötlusvoogudes.

### Peamised omadused

* 🚀 **Automatiseerimine** – mitme andmekogumi skriptide kogumitöötlus
* 🔗 **Integreerimine** – integreerimine olemasolevatesse töövoogudesse ja torujuhtmetesse
* 💻 **Peata töö** – töötab ilma graafilise kasutajaliideseta
* 🌍 **Mitmekeelsus** – toetab 38 keelt
* ⚡ **Paralleelne töötlemine** – skaleerub dünaamiliselt teie CPU-le (kuni 16 paralleelset töötajat)

### Nõuded

| Nõue          | Detailid                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operatsioonisüsteem** | Windows 10/11 (64-bitine)                                              |
| **Litsents**          | Chloros+ ([tasuline pakett nõutav](https://cloud.mapir.camera/pricing)) |
| **Mälu**           | Minimaalselt 8 GB RAM-i (soovitatav 16 GB)                                  |
| **Internet**         | Vajalik litsentsi aktiveerimiseks                                     |
| **Kettaruum**       | Sõltub projekti suurusest                                              |

{% vihje style=&quot;warning&quot; %}
**Litsentsinõuded**: CLI nõuab tasulist Chloros+ tellimust. Standard (tasuta) paketid ei sisalda CLI juurdepääsu. Uuendamiseks külastage [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Kiirstart

### Paigaldamine

CLI on automaatselt kaasas Chloros paigaldajaga:

1. Lae alla ja käivita **Chloros paigaldaja.exe**

2. Viige paigaldusviisard lõpule
3. CLI installitud: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Installija lisab automaatselt `chloros-cli` teie süsteemi PATH-i. Pärast installimist taaskäivitage terminal.
{% endhint %}

### Esmane seadistamine

Enne CLI kasutamist aktiveerige oma Chloros+ litsents:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Põhiline kasutus

Käsitlege kausta vaikimisi seadetega:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Käskude viited

### Üldine süntaks

```
chloros-cli [global-options] <command> [command-options]
```

***

## Käskud

### `process` – piltide töötlemine

Käsitle pildid kaustas kalibreerimisega.

**Süntaks:**

```bash
chloros-cli process <input-folder> [options]
```

**Näide:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Käsitlemiskäsu valikud

| Valik                | Tüüp    | Vaikimisi        | Kirjeldus                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Tee    | _Nõutav_     | RAW/JPG multispektraalsete piltide kaust                                         |
| `-o, --output`        | Tee    | Sama kui sisend  | Töödeldud piltide väljundkaust                                                     |
| `-n, --project-name`  | String  | Automaatselt genereeritud | Kohandatud projekti nimi                                                                    |
| `--vignette`          | Lipuke    | Lubatud        | Luba vinjeti korrigeerimine                                                             |
| `--no-vignette`       | Lipuke    | -              | Keela vinjeti korrigeerimine                                                            |
| `--reflectance`       | Lipuke    | Lubatud        | Peegelduskalibreerimise lubamine                                                         |
| `--no-reflectance`    | Lipuke    | -              | Peegelduskalibreerimise keelamine                                                        |
| `--ppk`               | Lipuke    | Keelatud       | Rakenda PPK-korrektsioone .daq valgussensori andmetest                                      |
| `--format`            | Valik  | TIFF (16-bitine)  | Väljundvorming: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Täisarv | Auto           | Kalibreerimispaneeli tuvastamise minimaalne sihtmärgi suurus pikslites                          |
| `--target-clustering` | Täisarv | Auto           | Sihtmärgi klastrite künnis (0–100)                                                    |
| `--exposure-pin-1`    | String  | Puudub           | Kaamera mudeli ekspositsiooni lukustamine (pin 1)                                                 |
| `--exposure-pin-2`    | String  | Puudub           | Kaamera mudeli ekspositsiooni lukustamine (pin 2)                                                 |
| `--recal-interval`    | Täisarv | Auto           | Kalibreerimise intervall sekundites                                                      |
| `--timezone-offset`   | Täisarv | 0              | Ajavööndi nihke tundides                                                               |

***

### `login` – konto autentimine

Logige sisse oma Chloros+ kasutajatunnusega, et võimaldada CLI töötlemist.

**Süntaks:**

```bash
chloros-cli login <email> <password>
```

**Näide:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Erimärgid**: Kasutage ülakomaid paroolide ümber, mis sisaldavad märke nagu `$`, `!` või tühikuid.
{% endhint %}

**Väljund:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Krediidiandmete kustutamine

Kustutage salvestatud krediidiandmed ja logige oma kontolt välja.

**Süntaks:**

```bash
chloros-cli logout
```

**Näide:**

```powershell
chloros-cli logout
```

**Väljund:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style=&quot;info&quot; %}
**SDK kasutajad**: Python SDK pakub ka programmilist `logout()` meetodit volituste tühistamiseks Python skriptides. Täpsema teabe saamiseks vaadake [Python SDK dokumentatsiooni](api-python-sdk.md#logout).
{% endhint %}

***

### `status` – kontrolli litsentsi staatust

Kuva praegune litsentsi ja autentimise staatus.

**Süntaks:**

```bash
chloros-cli status
```

**Näide:**

```powershell
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

### `export-status` – ekspordi edenemise kontrollimine

Jälgib Thread 4 ekspordi edenemist töötlemise ajal või pärast seda.

**Süntaks:**

```bash
chloros-cli export-status
```

**Näide:**

```powershell
chloros-cli export-status
```

**Kasutusjuhtum:** Kutsu käsk välja töötlemise käigus, et kontrollida ekspordi edenemist.***

### `language` – Liidese keele haldamine

Vaata või muuda CLI liidese keelt.

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

```powershell
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

| Kood    | Keel              | Emakeel      |
| ------- | --------------------- | ---------------- |
| `en`    | Inglise keel               | English          |
| `es`    | Hispaania keel               | Español          |
| `pt`    | Portugali keel            | Português        |
| `fr`    | Prantsuse keel                | Français         |
| `de`    | Saksa keel                | Deutsch          |
| `it`    | Itaalia keel               | Italiano         |
| `ja`    | Jaapani keel              | 日本語              |
| `ko`    | Korea keel                | 한국어              |
| `zh`    | Hiina keel (lihtsustatud)  | 简体中文             |
| `zh-TW` | Hiina keel (traditsiooniline) | 繁體中文             |
| `ru`    | Vene               | Русский          |
| `nl`    | Hollandi                 | Nederlands       |
| `ar`    | Araabia                | العربية          |
| `pl`    | Poola                | Polski           |
| `tr`    | türgi keel               | Türkçe           |
| `hi`    | hindi keel                 | हिंदी            |
| `id`    | indoneesia keel            | Bahasa Indonesia |
| `vi`    | Vietnami keel            | Tiếng Việt       |
| `th`    | Tai keel                  | ไทย              |
| `sv`    | Rootsi keel               | Svenska          |
| `da`    | Taani keel                | Dansk            |
| `no`    | Norra keel             | Norsk            |
| `fi`    | Soome keel               | Suomi            |
| `el`    | Kreeka keel                 | Ελληνικά         |
| `cs`    | Tšehhi                 | Čeština          |
| `hu`    | Ungari             | Magyar           |
| `ro`    | Rumeenia              | Română           |
| `uk`    | Ukraina             | Українська       |
| `pt-BR` | Brasiilia portugali keel  | Português Brasileiro |
| `zh-HK` | Kantoni keel             | 粵語             |
| `ms`    | Malai keel                 | Bahasa Melayu    |
| `sk`    | Slovaki keel                | Slovenčina       |
| `bg`    | Bulgaaria keel             | Български        |
| `hr`    | Horvaadi keel              | Hrvatski         |
| `lt`    | Leedu keel            | Lietuvių         |
| `lv`    | Läti keel               | Latviešu         |
| `et`    | Eesti              | Eesti            |
| `sl`    | Sloveenia             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Automaatne püsivus**: Teie keele-eelistus salvestatakse `~/.chloros/cli_language.json` ja püsib kõigis sessioonides.
{% endhint %}

***

### `set-project-folder` - Vaikimisi projektikausta seadmine

Muuda vaikimisi projektikausta asukohta (jagatud GUI-ga).

**Süntaks:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Näide:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` – Projekti kausta kuvamine

Kuva praegune vaikimisi projektikausta asukoht.

**Süntaks:**

```bash
chloros-cli get-project-folder
```

**Näide:**

```powershell
chloros-cli get-project-folder
```

**Väljund:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Taasta vaikimisi

Taasta projekti kausta vaikimisi asukoht.

**Süntaks:**

```bash
chloros-cli reset-project-folder
```

***

## Globaalsed valikud

Need valikud kehtivad kõikide käskude puhul:

| Valik          | Tüüp    | Vaikimisi       | Kirjeldus                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Tee    | Automaatselt tuvastatud | Tee tagapõhja käivitatavale failile                       |
| `--port`        | Täisarv | 5000          | Tagapõhja API pordi number                          |
| `--restart`     | Lipuke    | -             | Sundida taustaprogrammi taaskäivitamist (lõpetab olemasolevad protsessid) |
| `--version`     | Lipuke    | -             | Näita versiooni teavet ja välju                |
| `--help`        | Lipuke    | -             | Näita abi teavet ja välju                   |

**Näide globaalsete valikutega:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Töötlemise seadete juhend

### Paralleelne töötlemine

Chloros+ CLI **skaleerib automaatselt**paralleelset töötlemist vastavalt teie arvuti võimsusele:**Kuidas see toimib:**

* Tuvastab teie CPU tuumad ja RAM-i
* Jaotab töötajad: **2× CPU tuumad** (kasutab hüperthreadingut)
* **Maksimum: 16 paralleelset töötajat** (stabiilsuse tagamiseks)**Süsteemi tasemed:**

| Süsteemi tüüp   | CPU        | RAM      | Töötajad  | Jõudlus     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Kõrgetasemeline**  | 16+ tuuma  | 32+ GB   | Kuni 16 | Maksimaalne kiirus   |
| **Keskklassi** | 8–15 tuuma | 16–31 GB | 8–16     | Suurepärane kiirus |
| **Madala klassi**   | 4–7 tuuma  | 8–15 GB  | 4–8      | Hea kiirus      |

{% hint style=&quot;success&quot; %}
**Automaatne optimeerimine**: CLI tuvastab automaatselt teie süsteemi spetsifikatsioonid ja konfigureerib optimaalse paralleelse töötlemise. Käsitsi konfigureerimine pole vajalik!
{% endhint %}

### Debayer-meetodid

CLI kasutab vaikimisi ja soovitatava debayer-algoritmina **kõrge kvaliteeti (kiirem)**:

| Meetod                      | Kvaliteet | Kiirus | Kirjeldus                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Kõrge kvaliteet (kiirem)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Servatundlik algoritm (vaikimisi, soovitatav) |

### Vignette&#x27;i korrigeerimine

**Mida see teeb:** Korrigeerib valguse langust pildi servades (kaamerapiltidel tavalised tumedamad nurgad).

* **Vaikimisi sisse lülitatud** – enamik kasutajaid peaks selle sisse lülitatuna jätma.
* Kasutage `--no-vignette`, et see välja lülitada.

{% hint style=&quot;success&quot; %}
**Soovitus**: lülitage vinjetikorrektsioon alati sisse, et tagada ühtlane heledus kogu kaadris.
{% endhint %}

### Peegelduskalibreerimine

Muudab tooresensorite väärtused kalibreerimispaneelide abil standardiseeritud peegeldusprotsentideks.

* **Vaikimisi aktiveeritud** – oluline taimestiku analüüsimiseks.
* Nõuab kalibreerimise sihtpaneele piltidel.
* Kasutage `--no-reflectance`, et deaktiveerida.

{% vihje stiil=&quot;info&quot; %}
**Nõuded**: Tagage, et kalibreerimispaneelid oleksid piltidel õigesti eksponeeritud ja nähtavad, et tagada täpne peegeldusvõime konverteerimine.
{% endhint %}

### PPK parandused

**Funktsioon:** Rakendab DAQ-A-SD logiandmeid kasutades järelkäsitletud kinemaatilisi parandusi, et parandada GPS-i täpsust.

* **Vaikimisi keelatud**
* Kasutamiseks aktiveerige `--ppk`
* Nõuab .daq faile projektikausta MAPIR DAQ-A-SD valgussensorist.

### Väljundvormingud

<table><thead><tr><th width="197">Vorming</th><th width="130.20001220703125">Biti sügavus</th><th width="116.5999755859375">Faili suurus</th><th>Sobib kõige paremini</th></tr></thead><tbody><tr><td><strong>TIFF (16-bitine)</strong> ⭐</td><td>16-bitine täisarv</td><td>Suur</td><td>GIS-analüüs, fotogramm-meetria (soovitatav)</td></tr><tr><td><strong>TIFF (32-bitine, protsent)</strong></td><td>32-bitine ujukomaarv</td><td>Väga suur</td><td>Teaduslik analüüs, uurimistöö</td></tr><tr><td><strong>PNG (8-bitine)</strong></td><td>8-bitine täisarv</td><td>Keskmine</td><td>Visuaalne kontroll, veebis jagamine</td></tr><tr><td><strong>JPG (8-bitine)</strong></td><td>8-bitine täisarv</td><td>Väike</td><td>Kiire eelvaade, kompresseeritud väljund</td></tr></tbody></table>***

## Automatiseerimine ja skriptimine

### PowerShell&#x27;i kogumitöötlus

Mitme andmekogumi kausta automaatne töötlemine:

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

### Windows kogumiskript

Lihtne tsükkel kogumitöötluseks:

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

### Python automatiseerimisskript

Täiustatud automatiseerimine veakäsitlusega:

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

1. **Sisend**: kaust, mis sisaldab RAW/JPG pildipaare
2. **Avastamine**: CLI skannib automaatselt toetatud pildifaile
3. **Töötlemine**: Paralleelrežiim skaleerub vastavalt teie CPU tuumadele (Chloros+)
4. **Väljund**: Loob kaameramudeli alamkaustad töödeldud piltidega

### Väljundi struktuuri näide

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

| Režiim              | Aeg      | Riistvara                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Paralleelrežiim** | 5–10 min  | i7/Ryzen 7, 16 GB RAM, SSD (kuni 16 töötajat) |
| **Paralleelrežiim** | 10–15 min | i5/Ryzen 5, 8 GB RAM, HDD (kuni 8 töötajat)   |

{% vihje style=&quot;info&quot; %}
**Jõudluse näpunäide**: Töötlemisaeg sõltub piltide arvust, resolutsioonist ja arvuti spetsifikatsioonidest.
{% endhint %}

***

## Veaotsing

### CLI ei leitud

**Viga:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Lahendused:**

1. Kontrollige installimise asukohta:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Kasutage täielikku teekonda, kui see ei ole PATH-is:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Lisage PATH-i käsitsi:
   * Avage süsteemi omadused → keskkonnamuutujad
   * Muutke PATH-i muutujat
   * Lisage: `C:\Program Files\Chloros\resources\cli`
   * Käivitage terminal uuesti.

***

### Backend ei õnnestunud käivitada.**Viga:**

```

Backend failed to start within 30 seconds
```

**Lahendused:**

1. Kontrollige, kas backend juba töötab (sulge see esmalt).
2. Kontrollige, et Windows tulemüür ei blokeeri.
3. Proovige teist porti:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Sundige backendi taaskäivitamine:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Litsentsi-/autentimise probleemid**Viga:**

```

Chloros+ license required for CLI access
```

**Lahendused:**

1. Veenduge, et teil on aktiivne Chloros+ tellimus.
2. Logige sisse oma kasutajatunnuse ja parooliga:

```powershell
chloros-cli login user@example.com 'password'
```

3. Kontrollige litsentsi staatust:

```powershell
chloros-cli status
```

4. Võtke ühendust klienditoega: info@mapir.camera

***

### Pilte ei leitud**Viga:**

```

No images found in the specified folder
```

**Lahendused:**

1. Veenduge, et kaust sisaldab toetatud formaate (.RAW, .TIF, .JPG).
2. Kontrollige, et kausta tee on õige (kasutage tühikuga teede puhul jutumärke).
3. Veenduge, et teil on kausta lugemisõigused.
4. Kontrollige, et faililaiendid on õiged.

***

### Töötlemine seiskub või hangub**Lahendused:**

1. Kontrollige vaba kettaruumi (veenduge, et väljundiks on piisavalt ruumi).
2. Sulgege teised rakendused, et vabastada mälu.
3. Vähendage piltide arvu (töötlege partiidena).

***

### Port on juba kasutusel**Viga:**

```

Port 5000 is already in use
```

**Lahendus:**

Määrake teine port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## KKK

### K: Kas ma vajan litsentsi CLI jaoks?

**V:**Jah! CLI nõuab tasulist**Chloros+ litsentsi**.

* ❌ Standardne (tasuta) pakett: CLI on keelatud
* ✅ Chloros+ (tasuline) paketid: CLI täielikult aktiveeritud

Telli aadressil: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### K: Kas ma saan kasutada CLI serveris, millel pole graafilist kasutajaliidest?**V:** Jah! CLI töötab täielikult ilma graafilise kasutajaliideseta. Nõuded:

* Windows Server 2016 või uuem
* Visual C++ Redistributable installitud
* Piisav RAM-mälu (minimaalselt 8 GB, soovitatavalt 16 GB)
* Ühekordne GUI litsentsi aktiveerimine mis tahes arvutil

***

### K: Kuhu salvestatakse töödeldud pildid?**V:**Vaikimisi salvestatakse töödeldud pildid**samasse kausta kui sisend** kaameramudeli alamkausta (nt `Survey3N_RGN/`).

Kasutage `-o` valikut, et määrata teine väljundkataloog:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### K: Kas ma saan töödelda mitut kataloogi korraga?**V:** Ühe käsuga otseselt mitte, kuid saate kasutada skripte, et töötada kaustu järjestikku. Vaadake jaotist [Automatiseerimine ja skriptimine](CLI.md#automation--scripting).***

### K: Kuidas salvestada CLI väljund logifaili?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Paki:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### K: Mis juhtub, kui ma töötlemise ajal vajutan Ctrl+C?**V:** CLI teeb järgmist:

1. Lõpetab töötlemise sujuvalt
2. Sulgeb tagapõhja
3. Lõpetab koodiga 130

Osaliselt töödeldud pildid võivad jääda väljundkausta.

***

### K: Kas ma saan CLI töötlemist automatiseerida?**V:** Muidugi! CLI on loodud automatiseerimiseks. Vaadake [Automatiseerimine ja skriptimine](CLI.md#automation--scripting) PowerShell, Batch ja Python näidete jaoks.***

### K: Kuidas kontrollida CLI versiooni?**V:**

```powershell
chloros-cli --version
```

**Väljund:**

```

Chloros CLI 1.0.2
```

***

## Abi saamine

### Käsuviiva abi

Vaadake abiinfo otse CLI-is:

```powershell
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Abikanalid

* **E-post**: info@mapir.camera
* **Veebisait**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Hinnad**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Täielikud näited

### Näide 1: põhiline töötlemine

Töötlemine vaikimisi seadetega (vinjett, peegeldusvõime):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Näide 2: kõrgekvaliteediline teaduslik tulemus

32-bitine ujukomaarv TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Näide 3: kiire eelvaate töötlemine

8-bitine PNG ilma kalibreerimiseta kiireks ülevaatamiseks:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Näide 4: PPK-korrigeeritud töötlemine

PPK-korrektsioonide rakendamine peegeldusvõimega:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Näide 5: Kohandatud väljundi asukoht

Töötlemine teise draivi spetsiifilises formaadis:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Näide 6: Autentimise töövoog

Täielik autentimise voog:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Näide 7: Mitmekeelne kasutamine

Muuda liidese keelt:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
