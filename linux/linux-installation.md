# Linux paigaldamine

Chloros levitatakse Linux jaoks `.deb` pakettidena, mis paigaldavad CLI ja tagapõhja. Python SDK installitakse eraldi pipi kaudu.

***

## Linux amd64 (x86_64)

### Süsteeminõuded

| Nõue | Minimaalne | Soovitatav |
| --- | --- | --- |
| **Distributsioon** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Protsessor** | x86_64 (Intel/AMD) | Intel Core i7 või parem |
| **Mälu (RAM)** | 8 GB | 16 GB või rohkem |
| **Graafikakaart** | Puudub (töötleb CPU) | NVIDIA GPU 4 GB+ VRAM-iga |
| **Salvestusruum** | 2 GB vaba ruumi | SSD 10 GB+ vaba ruumiga |
| **Python** | Python 3.7+ (SDK jaoks) | Python 3.10+ |

### Paigaldamine

Laadige alla `.deb` pakett ja paigaldage:

```bash
sudo dpkg -i chloros-amd64.deb
```

Kontrollige paigaldamist:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Süsteeminõuded

| Nõue | Minimaalne | Soovitatav |
| --- | --- | --- |
| **Platvorm** | NVIDIA Jetson koos JetPack 6-ga | Jetson Orin NX 16 GB või AGX Orin |
| **JetPack** | JetPack 6.x | Viimane JetPack 6 |
| **Mälu (RAM)** | 8 GB (jagatud GPU/CPU) | 16 GB+ jagatud |
| **Salvestusruum** | 2 GB vaba ruumi | NVMe SSD, millel on 10 GB+ vaba ruumi |
| **Python** | Python 3.7+ (SDK jaoks) | Python 3.10+ |

### Paigaldamine

Laadige alla JetPack 6 `.deb` pakett ja paigaldage:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Kontrollige paigaldamist:

```bash
chloros-cli --version
```

Jetsoni üksikasjaliku seadistamise, sealhulgas soojusjuhtimise ja välitingimustes kasutuselevõtu kohta vaadake [NVIDIA Jetsoni juhendit](nvidia-jetson-guide.md).

***

## Python SDK paigaldamine (kõik Linux)

Python SDK paigaldatakse eraldi pipi kaudu ja töötab nii amd64 kui ka arm64 platvormidel:

```bash
pip install chloros-sdk
```

Valikulise edusammude voogedastuse toe lisamiseks:

```bash
pip install chloros-sdk[progress]
```

Kontrollige SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Pakett `.deb` installib Chloros CLI ja backend. Python SDK on eraldi pip-pakett, mis suhtleb backendiga kohaliku HTTP API kaudu.
{% endhint %}

***

## Konfiguratsioonikataloogid

Chloros Linux-is järgib [XDG baaskataloogi spetsifikatsiooni](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Otstarve | Linux Tee | Windows Ekvivalent |
| --- | --- | --- |
| **Konfiguratsioon** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Andmed / Projektid** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Vahemälu / Autentimisandmed** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Tagapõhja käivitatavate failide asukohad

Pakett `.deb` installib tagapõhja standardasukohta. Paketid CLI ja SDK tuvastavad tagapõhja asukoha automaatselt:

| Paigaldusmeetod | Tagapõhja asukoht |
| --- | --- |
| `.deb` pakett | `/usr/lib/chloros/chloros-backend` |
| Käsitsi / kohandatud | `/opt/mapir/chloros/backend/chloros-backend` |

Saate backend-kataloogi asendada lipuga `--backend-exe` CLI või konstruktori parameetriga `backend_exe` SDK.

***

## Esmane seadistamine

### 1. Aktiveerige oma litsents

Chloros+ litsents on vajalik CLI ja SDK juurdepääsuks:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Kontrollige oma litsentsi staatust

```bash
chloros-cli status
```

### 3. Töötlege oma esimest andmekogumit

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Käivitage süsteemidiagnostika

Veenduge, et teie süsteem on õigesti konfigureeritud:

```bash
chloros-cli selftest
```

See käivitab 7 diagnostilist kontrolli, sealhulgas versiooni, tagapõhja käivitamise, API ühenduvuse ja CUDA/GPU kättesaadavuse kontrolli.

***

## Bash-skriptide näited

### Töötle mitut andmekogumit

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Töötle kohandatud seadetega

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automaatne töötlemine Croniga

Lisage oma crontabisse (`crontab -e`), et töötada uusi andmekogumeid automaatselt:

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

## Veahaldus

### CLI ei leitud pärast installimist

Kui `chloros-cli` ei leita pärast `.deb` paketi installimist:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Juurdepääs keelatud

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
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

### CUDA-d ei tuvastatud

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Puuduvad jagatud raamatukogud

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Chloros uuendamine Linux-is

Kasutage sisseehitatud uuenduskäsku uuenduste otsimiseks ja installimiseks:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Järgmised sammud

* [NVIDIA Jetsoni juhend](nvidia-jetson-guide.md) — Jetsonile spetsiifiline optimeerimine ja kasutuselevõtt
* [CLI : käsurea](../CLI.md) — Täielik CLI käskude viide
* [API : Python SDK](../api-python-sdk.md) — Täielik SDK viide
* [Dünaamiline arvutusvõimsuse kohandamine](../processing-architecture/dynamic-compute-adaptation.md) — Kuidas Chloros kohandub teie riistvaraga
