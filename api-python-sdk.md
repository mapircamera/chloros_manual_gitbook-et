# API : Python SDK

**Chloros Python SDK** pakub programmilist juurdepääsu Chloros pilditöötlusmootorile, võimaldades automatiseerimist, kohandatud töövooge ja sujuvat integratsiooni teie Python rakenduste ja uurimisprotsessidega.

### Peamised omadused

* 🐍 **Native Python** – puhas, Pythoni stiilis API pilditöötluseks
* 🔧 **Täielik API juurdepääs** – täielik kontroll Chloros töötlemise üle
* 🚀 **Automatiseerimine** – looge kohandatud partii töötlemise töövood
* 🔗 **Integreerimine** – integreerige Chloros olemasolevatesse Python rakendustesse
* 📊 **Teadustööks valmis** – ideaalne teaduslike analüüsiprotsesside jaoks
* ⚡ **Paralleelne töötlemine** – Skaleerub vastavalt teie CPU tuumadele (Chloros+)

### Nõuded

| Nõue          | Detailid                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros installitud** | Windows: töölaua installija; Linux: `.deb` pakett                  |
| **Litsents**          | Chloros+ ([vajalik tasuline pakett](https://cloud.mapir.camera/pricing)) |
| **Operatsioonisüsteem** | Windows 10/11 (64-bitine), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 või uuem                                                |
| **Mälu**           | Minimaalselt 8 GB RAM-i (soovitatavalt 16 GB)                                  |
| **Internet**         | Vajalik litsentsi aktiveerimiseks                                     |

{% hint style="warning" %}
**Litsentsinõuded**: Python SDK nõuab Chloros+ tasulist tellimust, et saada juurdepääs API-ile. Standard (tasuta) paketid ei anna juurdepääsu API/SDK-ile. Uuendamiseks külasta [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Kiirstart

### Paigaldamine

Paigaldage pipi kaudu:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Esmane seadistamine**: Enne SDK kasutamist aktiveerige oma Chloros+ litsents, avades Chloros, Chloros (brauser) või Chloros CLI ja logige sisse oma kasutajatunnuse ja parooliga. Seda tuleb teha ainult üks kord. Linux-is (ilma graafilise kasutajaliideseta) kasutage: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Põhikasutus

Töötle kaust vaid mõne reaga:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Platvormiüleseid teid**: Selle lehe koodinäited kasutavad Windows-stiilis teid (nt `C:\\DroneImages\\Flight001`). Linux-is kasutage selle asemel Linux-tüüpi teid (nt `/home/user/drone_images/flight001` või `~/drone_images/flight001`). SDK töötab mõlemal platvormil ühtmoodi.
{% endhint %}

### Täielik kontroll

Täpsemate töövoogude jaoks:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Paigaldusjuhend

### Eeltingimused

Enne SDK paigaldamist veenduge, et teil on:

1. **Chloros on paigaldatud** — Windows: töölaua paigaldaja ([alla laadida](download.md)); Linux: `.deb` pakett ([Linux paigaldamine](linux/linux-installation.md))
2. **Python 3.7+** on installitud ([python.org](https://www.python.org))
3. **Aktiivne Chloros+ litsents** ([uuendamine](https://cloud.mapir.camera/pricing))

### Paigaldamine pipi kaudu

**Standardpaigaldus:**

```bash
pip install chloros-sdk
```

**Progressi jälgimise toega:**

```bash
pip install chloros-sdk[progress]
```

**Arenduspaigaldus:**

```bash
pip install chloros-sdk[dev]
```

### Paigalduse kontrollimine

Kontrollige, kas SDK on õigesti paigaldatud:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Esmane seadistamine

### Litsentsi aktiveerimine

SDK kasutab sama litsentsi kui Chloros, Chloros (brauser) ja Chloros CLI. Aktiveerige üks kord graafilise kasutajaliidese või CLI kaudu:**Windows:**Avage**Chloros või Chloros (brauser)** ja logige sisse vahekaardil „Kasutaja” <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> või kasutage CLI.**Linux:** Kasutage CLI (graafilist kasutajaliidest pole saadaval):

```bash
chloros-cli login user@example.com 'your_password'
```

Litsents salvestatakse kohalikku vahemällu ja säilib ka pärast süsteemi taaskäivitamist.

{% hint style="success" %}
**Ühekordne seadistamine**: Pärast sisselogimist graafilise kasutajaliidese või CLI kaudu kasutab SDK automaatselt vahemällu salvestatud litsentsi. Täiendavat autentimist ei ole vaja!
{% endhint %}

{% hint style="info" %}
**Väljumine**: SDK kasutajad saavad vahemällu salvestatud autentimisandmeid programmiliselt kustutada, kasutades meetodit `logout()`. Vaadake [logout() meetodit](#logout) API viites.
{% endhint %}

### Ühenduse testimine

Kontrollige, kas SDK suudab ühenduda Chloros-iga:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API viide

### ChlorosLocal klass

Peamine klass kohaliku Chloros pilditöötluse jaoks.

#### Konstruktor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parameetrid:**

| Parameeter                 | Tüüp | Vaikimisi                   | Kirjeldus                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL kohaliku Chloros tagapõhja          |
| `auto_start_backend`      | bool | `True`                    | Käivita backend vajadusel automaatselt |
| `backend_exe`             | str  | `None` (automaatne tuvastamine)      | Backendi käivitatava faili tee            |
| `timeout`                 | int  | `30`                      | Päringu ooteaeg sekundites            |
| `backend_startup_timeout` | int  | `60`                      | Backendi käivitamise ooteaeg (sekundites) |

**Näited:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Platvormiülene automaatne tuvastamine**: SDK proovib automaatselt teie platvormile sobivat tagapõhja teekonda:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (käsitsi)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Meetodid

#### `create_project(project_name, camera=None)`

Loo uus Chloros projekt.

**Parameetrid:**

| Parameeter      | Tüüp | Nõutav | Kirjeldus                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Jah      | Projekti nimi                                     |
| `camera`       | str  | Ei       | Kaamera mall (nt &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Tagastab:** `dict` - Projekti loomise vastus**Näide:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Impordi pildid kaustast.

**Parameetrid:**

| Parameeter     | Tüüp     | Nõutav | Kirjeldus                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Jah      | Pildikausta asukoht         |
| `recursive`   | bool     | Ei       | Otsi alamkaustadest (vaikimisi: False) |

**Tagastab:** `dict` - Impordi tulemused koos failide arvuga**Näide:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Töötlemise seadete konfigureerimine.

**Parameetrid:**

| Parameeter                 | Tüüp | Vaikimisi                 | Kirjeldus                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Standard (kiire, keskmine kvaliteet)&quot; | Debayer-meetod            |
| `vignette_correction`     | bool | `True`                  | Luba vinjettkorrektsiooni      |
| `reflectance_calibration` | bool | `True`                  | Luba peegelduskalibreerimist  |
| `indices`                 | list | `None`                  | Arvutatavad taimestiku indeksid |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Väljundvorming                   |
| `ppk`                     | bool | `False`                 | Luba PPK-korrektsioone          |
| `custom_settings`         | dict | `None`                  | Täpsemad kohandatud seaded        |

**Ekspordivormingud:**

* `"TIFF (16-bit)"` – soovitatav GIS-i/fotogramm-meetria jaoks
* `"TIFF (32-bit, Percent)"` – teaduslik analüüs
* `"PNG (8-bit)"` – visuaalne kontroll
* `"JPG (8-bit)"` – pakitud väljund

**Saadaval olevad indeksid:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 jm.**Näide:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Töötle projekti pilte.

**Parameetrid:**

| Parameeter           | Tüüp     | Vaikimisi      | Kirjeldus                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Töötlemisrežiim: &quot;parallel&quot; või &quot;serial&quot;   |
| `wait`              | bool     | `True`       | Oota lõpetamist                       |
| `progress_callback` | callable | `None`       | Edusammude tagasisidefunktsioon (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Edusammude küsitlusintervall (sekundites)   |

**Tagastab:** `dict` - Töötlemise tulemused

{% hint style="warning" %}
**Paralleelrežiim**: Nõuab Chloros+ litsentsi. Skaalub automaatselt vastavalt teie CPU tuumadele (kuni 16 töötajat).
{% endhint %}

**Näide:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Hangi praegune projekti konfiguratsioon.

**Tagastab:** `dict` - Praegune projekti konfiguratsioon**Näide:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Hangi tagapõhja staatuse teave, sealhulgas iga niidi töötlemise edenemine.

**Tagastab:** `dict` – Tagapõhja staatus järgmise struktuuriga:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Näide:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Lülita backend välja (kui see on käivitatud käskuga SDK).

**Näide:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Kustuta vahemällu salvestatud autentimisandmed kohalikust süsteemist.

**Kirjeldus:**

Logib programmiliselt välja, eemaldades vahemällu salvestatud autentimisandmed. See on kasulik:
* Erinevate Chloros+ kontode vahel vahetamiseks
* Autentimisandmete tühjendamiseks automatiseeritud keskkondades
* Turvalisuse eesmärgil (nt autentimisandmete eemaldamine enne desinstallimist)

**Tagastab:** `dict` - Väljumise operatsiooni tulemus**Näide:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Vajalik uuesti autentimine**: Pärast `logout()` kutsumist peate uuesti sisse logima Chloros, Chloros (brauser) või Chloros CLI kaudu, enne kui kasutate SDK.
{% endhint %}

***

### Mugavusfunktsioonid

#### `process_folder(folder_path, **options)`

Üherealine mugavusfunktsioon kausta töötlemiseks.

**Parameetrid:**

| Parameeter                 | Tüüp     | Vaikimisi         | Kirjeldus                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Nõutav        | Pildikausta asukoht     |
| `project_name`            | str      | Automaatselt genereeritud  | Projekti nimi                   |
| `camera`                  | str      | `None`          | Kaamera mall                |
| `indices`                 | list     | `["NDVI"]`      | Arvutamiseks vajalikud indeksid           |
| `vignette_correction`     | bool     | `True`          | Vignettkorrektsiooni lubamine     |
| `reflectance_calibration` | bool     | `True`          | Peegelduskalibreerimise lubamine |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Väljundvorming                  |
| `mode`                    | str      | `"parallel"`    | Töötlemisrežiim                |
| `progress_callback`       | kutsutav | `None`          | Edusammude tagasiside              |

**Tagastab:** `dict` - Töötlemise tulemused**Näide:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Kontekstihalduri tugi

SDK toetab kontekstihaldureid automaatseks puhastamiseks:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Täielikud näited

{% hint style="info" %}
**Linux kasutajad**: Kõik allpool toodud näited kasutavad Windows teid. Asendage `C:\\...` teed oma Linux teedega (nt `/home/user/...` või `~/...`). Kõik SDK funktsioonid on platvormide lõikes identsed.
{% endhint %}

### Näide 1: Põhitöötlus

Kausta töötlemine vaikimisi seadetega:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Näide 2: Kohandatud töövoog

Täielik kontroll töötlusprotsessi üle:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Näide 3: Mitme kausta kogumitöötlus

Töötle mitut lennuandmestikku:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Näide 4: Uurimistöö voo integreerimine

Integreeri Chloros andmete analüüsiga:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Näide 5: Kohandatud protsessi jälgimine

Täiustatud protsessi jälgimine logimisega:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Näide 6: Veakäsitlus

Tugev veakäsitlus tootmiskasutuseks:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Näide 7: Kontode haldamine ja väljalogimine

Kontoandmete haldamine programmeeritult:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Näide 8: Käsurea tööriist

Kohandatud CLI tööriista loomine SDK abil:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Kasutamine:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Erandite käsitlemine

SDK pakub erinevate veatüüpide jaoks spetsiifilisi erandiklasse:

### Erandite hierarhia

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Erandite näited

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Täpsemad teemad

### Tagapõhja kohandatud konfiguratsioon

Kasutage tagapõhja kohandatud asukohta või konfiguratsiooni:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Blokeerimata töötlemine

Alustage töötlemist ja jätkake teiste ülesannetega:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Mäluhaldus

Suurte andmekogumite puhul töötle partiidena:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Veaotsing

### Tagapõhi ei käivitu

**Probleem:** SDK ei suuda tagapõhja käivitada**Lahendused:**

1. Kontrollige, kas Chloros on installitud:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Kontrollige tulemüüri (Windows) või pordi kättesaadavust (Linux: `lsof -i :5000`)
3. Proovige käsitsi sisestada tagapõhja tee:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Litsentsi ei tuvastatud**Probleem:** SDK hoiatab puuduva litsentsi kohta**Lahendused:**

1. Avage Chloros, Chloros (brauser) või Chloros CLI ja logige sisse.
2. Veenduge, et litsents on vahemällu salvestatud:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Kui tekib probleeme kasutajatunnustega, tühjendage vahemällu salvestatud kasutajatunnused ja logige uuesti sisse:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Võtke ühendust tugiteenistusega: info@mapir.camera

***

### Importimisvead**Probleem:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Lahendused:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Töötlemise aegumine**Probleem:** Töötlemise aegumine**Lahendused:**

1. Suurendage aegumist:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Töötle väiksemaid partiisid
3. Kontrolli vaba kettaruumi
4. Jälgi süsteemiressursse

***

### Port on juba kasutusel**Probleem:** Tagapõhja port 5000 on hõivatud**Lahendused:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Või leia ja sulge konflikti tekitav protsess:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Jõudluse näpunäited

### Optimeeri töötlemiskiirust

1. **Kasuta paralleelrežiimi** (nõuab Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Vähenda väljundi resolutsiooni** (kui see on vastuvõetav)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Lülita välja mittevajalikud indeksid**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Töötle SSD-l** (mitte HDD-l)***

### Mälu optimeerimine

Suurte andmekogumite puhul:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Taustatöötlus

Vabasta Python muude ülesannete jaoks:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Integreerimise näited

### Django integreerimine

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## KKK

### K: Kas SDK vajab internetiühendust?

**V:** Ainult esialgse litsentsi aktiveerimiseks. Pärast sisselogimist Chloros, Chloros (brauser) või Chloros CLI kaudu salvestatakse litsents kohalikult ja töötab offline-režiimis 30 päeva.***

### K: Kas ma saan kasutada SDK-i serveril, millel puudub graafiline kasutajaliides?**V:** Jah! SDK töötab ilma graafilise kasutajaliideseta nii Windows-i kui ka Linux-i serveritel.**Linux (soovitatav graafilise kasutajaliidese puudumisel):**
* Paigaldage `.deb` paketi kaudu
* Aktiveerige litsents: `chloros-cli login user@example.com 'password'`

**Windows Server:**
* Windows Server 2016 või uuem
* Chloros installitud (ühekordne)
* Litsents aktiveeritud CLI kaudu või mis tahes arvutil

***

### K: Mis vahe on Desktopil, CLI-il ja SDK-il?

| Omadus         | Desktop GUI | CLI käsurea | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Liides**   | Klõpsuga | Käsk | Python API  |
| **Sobib kõige paremini**    | Visuaalne töö | Skriptimine        | Integreerimine |
| **Automatiseerimine**  | Piiratud     | Hea             | Suurepärane   |
| **Paindlikkus** | Põhiline       | Hea             | Maksimaalne     |
| **Litsents**     | Chloros+    | Chloros+         | Chloros+    |***

### K: Kas ma saan levitada SDK-iga loodud rakendusi?**V:** SDK koodi saab integreerida teie rakendustesse, kuid:

* Lõppkasutajatel peab olema installitud Chloros
* Lõppkasutajatel peab olema aktiivne Chloros+ litsents
* Kommertslevitamiseks on vaja OEM-litsentsi

OEM-alaste küsimuste korral võtke ühendust info@mapir.camera-iga.

***

### K: Kuidas uuendada SDK-i?

```bash
pip install --upgrade chloros-sdk
```

***

### K: Kuhu salvestatakse töödeldud pildid?

Vaikimisi projekti kausta:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### K: Kas ma saan töödelda pilte Python skriptidest, mis töötavad ajakava järgi?**V:** Jah! Kasutage oma operatsioonisüsteemi ajastajat koos Python skriptidega:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Planeerige ülesande planeerija abil igapäevaseks käivitamiseks.**Linux:** Planeerige croni abil:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### K: Kas SDK toetab async/await-funktsioone?**V:** Praegune versioon on sünkroonne. Asünkroonse käitumise jaoks kasutage `wait=False` või käivitage eraldi protsessis:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### K: Kuidas vahetada erinevate Chloros+ kontode vahel?**V:** Kasutage meetodit `logout()`, et tühjendada vahemällu salvestatud autentimisandmed, seejärel logige uue kontoga uuesti sisse:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Pärast väljalogimist autentigeerige uue kontoga GUI, brauseri või CLI kaudu, enne kui kasutate SDK uuesti.

***

## Abi saamine

### Dokumentatsioon

* **API viide**: See lehekülg

### Tugikanalid

* **E-post**: info@mapir.camera
* **Veebisait**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Hinnad**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Näidiskood

Kõik siin loetletud näited on testitud ja valmis kasutuseks. Kopeerige ja kohandage neid oma vajadustele.

***

## Litsents**Omanditarkvara** – Autoriõigus (c) 2025 MAPIR Inc.

SDK nõuab aktiivset Chloros+ tellimust. Ebaseaduslik kasutamine, levitamine või muutmine on keelatud.
