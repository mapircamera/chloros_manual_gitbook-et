---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Väljundpildi formaadid

Chloros ekspordib töödeldud tulemusi neljas failiformaadis. Valige formaat projekti seadetest (GUI), kasutades `--format` (CLI) või `export_format` (SDK). CLI ja SDK aktsepteerivad allpool toodud täpseid stringe.

| Vormingustring | Laiend | Pikselitüüp | Pikselivahemik | Märkused |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(vaikimisi)* | `.tif` | uint16 digitaalarv | 0 – 65535 | Soovitatav fotogramm-meetria / GIS jaoks. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100% peegeldusvõime. Mõned rakendused ei suuda lugeda ujukomaarvulisi TIFF-faile; failid on suuremad. |
| `PNG (8-bit)` | `.png` | uint8 digitaalarv | 0 – 255 | Kaotuseta pakkimine, sobib veebis vaatamiseks ja visualiseerimiseks. |
| `JPG (8-bit)` | `.jpg` | uint8-tüüpi digitaalarv | 0 – 255 | Kaotusega pakkimine, kõige väiksemad failid. |

## Väljundfailide asukoht

Tooted salvestatakse projekti kausta alla, rühmitatuna kaamera ja seejärel failivormingu järgi:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kaamera kaust on LATTICE puhul `LATT-<sensor>-<lens>-F<filter>` ja Survey3 puhul `<model>_<filter>` (nt `Survey3N_RGN`). **Iga eksporditud toode säilitab allikafaili nime – toodet identifitseerib kaust, mitte failinime laiend.** Täielikud reeglid leiate CLI viite jaotisest [Kuhu väljundid salvestatakse](reference/cli-reference.md).

## LATTICE-tooted (salvestus- ja eksporditasemed)

Üks LATTICE-toorkaader jaguneb ühe läbimisega kõikideks taotletud toodeteks. Igal tootetüübil on oma lüliti (GUI-valikukastid või CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, kõik vaikimisi sisse lülitatud):

| Tase | Sisu | Andmetüüp |
| --- | --- | --- |
| `raw` | Otse sensorist pärit Bayeri andmed (mustvalged kaamerad: ühe lainepikkusega). Töötlemine algab alati toorandmetest. | Nagu salvestatud |
| `debayered` | Lineaarne demosaik – M3C puhul 3-kanaliline, M3M puhul 1-kanaliline halltoonide skaala. | Lineaarne DN |
| `radiance` | Absoluutne spektraalne kiirgusvõimsus täielikust radiomeetrilisest ahelast, ühikutes **W/m²/sr/nm**. Kirjutatakse alati 32-bitisena TIFF (`tiff32/Radiance_Images/`), sõltumata valitud ekspordiformaadist. | float32 |
| `reflectance` | Peegeldusvõime ρ, kus **DN 32768 = ρ 1,0 (100%)**, varuga kuni ρ 2,0. Pix4D-valmis. | uint16 |
| `preview` | Ekraanivalmis renderdus: RGB = valgebalanss + gamma; multispektraalne = valevärvide venitus. | 8-bitine kuvamine |

## Peegeldusvõime pikselväärtuste lugemine

Peegeldusvõime salvestatakse täisarvulise digitaalarvuna ning **DN, mis tähendab ρ = 1,0 (100% peegeldusvõime), sõltub allikakaamerast**:

| Allikakaamera | ρ = 1,0 on DN | Kuidas teada saada |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (varu kuni ρ 2,0) | Failile on märgitud XMP-märge `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (lõigatud väärtusel ρ 1,0) | XMP-märgiseid `Chloros:*` ei ole – see puudumine ongi märk. |

**Loe `Chloros:PixelScale` XMP-märget ja jaga sellega**, selle asemel et eeldada konstanti. Silt on määratletud uint16-domeenis, seega jääb see `32768`-iks ka väljundvormingutes, mis skaalavad ümber — normaliseeri salvestatud andmetüüp esmalt tagasi uint16-ks (8-bitisest ×257, float32-st ×65535).

{% hint style="warning" %}
**Üks juhtum ei sisalda skaalat, see on nii kavandatud.** Kui 8-bitine allikas (BayerRG8) salvestatakse 8-bitisena TIFF, siis töötlemisahel kärbib väärtused vahemikku 0–255, selle asemel et neid ümberskaalata, mistõttu failil puudub skaala — Chloros jätab seal teadlikult `Chloros:PixelScale` välja. Kui LATTICE-peegeldusfailis puudub see silt, ärge eeldage skaalat; eksportige fail hoopis uuesti 16-bitise või 32-bitisena.
{% endhint %}

Täielikud reeglid (sealhulgas MicaSense’iga ühilduvad märgised) leiate jaotisest **„Peegelduspikslite lugemine”** [CLI viitejuhendis](reference/cli-reference.md).
