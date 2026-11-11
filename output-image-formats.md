---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Väljundpildi formaadid

Chloros ekspordib töödeldud tooteid neljas failiformaadis. Valige formaat projekti seadetest (GUI), kasutades `--format` (CLI) või `export_format` (SDK). CLI ja SDK aktsepteerivad täpselt allpool toodud stringe.

| Formaadistring | Laiend | Pikselitüüp | Pikselivahemik | Märkused |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(vaikimisi)* | `.tif` | uint16 digitaalarv | 0 – 65535 | Soovitatav fotogramm-meetria / GIS-i jaoks. |
| `TIFF (8-bit)` | `.tif` | uint8 digitaalarv | 0 – 255 | Kõikidele 8-bitistele pildistamistele, mis pakuvad ainult 8-bitiseid formaate: Chloros ei suurenda bittisügavust. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100% peegeldusvõime. Mõned rakendused ei suuda lugeda ujukomaga TIFF-faile; failid on suuremad. |
| `PNG (8-bit)` | `.png` | uint8 digitaalarv | 0 – 255 | Kaotuseta pakkimine, sobib veebis vaatamiseks ja visualiseerimiseks. |
| `JPG (8-bit)` | `.jpg` | uint8 digitaalarv | 0 – 255 | Kaotusega pakkimine, kõige väiksemad failid. |

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

Kaamera kaust on LATTICE puhul `LATT-<sensor>-<lens>-F<filter>` ja Survey3 puhul `<model>_<filter>` (nt `Survey3N_RGN`). **Iga eksporditud toode säilitab allikafaili nime — toodet identifitseerib kaust, mitte failinime laiend.** Täielike reeglite kohta vaata [Väljundite asukoht](reference/cli-reference.md) CLI-viites.

## LATTICE-tooted (salvestus- ja eksporditasemed)

Üks LATTICE-toorkaader jaguneb ühe läbimisega kõikideks taotletud toodeteks. Igal tootetüübil on oma lüliti (GUI-valikukastid või CLI-käskudega `--debayered` / `--preview` / `--radiance` / `--reflectance`, kõik vaikimisi sisse lülitatud):

| Tase | Sisu | Andmetüüp |
| --- | --- | --- |
| `raw` | Otse sensorist pärit Bayeri andmed (mustvalged kaamerad: üks lainepikkus). Töötlemine algab alati toorandmetest. | Nagu salvestatud |
| `debayered` | Lineaarne demosaik — M3C puhul 3-kanaliline, M3M puhul 1-kanaliline halltoonide skaala: kalibreerimispaketi tume nihke on eemaldatud, seejärel toimub servadest lähtuv demosaik, ilma lineaarsuse korrigeerimise, tasase välja või segunemise eemaldamiseta. `process`-eksport salvestab igas töötlemisrežiimis sama faili, mida salvestab ka pildistamine sellel tasemel. | Lineaarne DN |
| `radiance` | Absoluutne spektraalne kiirgusvõimsus täielikust radiomeetrilisest ahelast, ühikutes **W/m²/sr/nm**. Salvestatakse alati 32-bitise TIFF-failina (`tiff32/Radiance_Images/`), sõltumata valitud ekspordivormingust. | float32 |
| `reflectance` | Peegeldusvõime ρ, kus **DN 32768 = ρ 1,0 (100%)**, varuga kuni ρ 2,0. Pix4D-valmis. | uint16 |
| `preview` | Ekraanivalmis renderdus: RGB = valgebalanss + gamma; multispektraalne = valevärvide venitus. | 8-bitine ekraan |

### Suund

`raw` ja `debayered` failid jäävad alati anduri **algse** orientatsiooni juurde; kaamera pööramise/pöörlemise seade rakendub ainult teistele toodetele (eelvaade, kiirgus, peegeldus, indeks). Massiivi salvestamisel säilitatakse iga elemendi failid algses orientatsioonis, kui te ei määra orientatsiooni (`--orient` CLI-s, `orient=True` SDK-s). Iga fail salvestab oma pikslitele kuuluva orientatsiooni XMP-vormingus — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` – ning `ChlorosOrientationApplied` näitab, kas piksleid on sellega varustatud (`false` toorfailides ja debayereeritud failides). Chlorosi pildivaataja teisendab RAW- ja debayered-faile selle kirje alusel ainult kuvamiseks; faile endid ei kirjutata kunagi ümber. Fail, millel kirje puudub (kirjutatud enne versiooni 1.3.0 või mõne teise tööriistaga), kuvatakse sellisena, nagu see on salvestatud.

## Klambermaskid

Multispektraalse M3C-kaamera kiirguse ja peegeldusvõime andmed töödeldakse 3×3-ribalise segamise eemaldamisega. Kui segamise eemaldamine annab mõnes ribas negatiivse väärtuse, kärbitakse see väärtus nullini (reaalajas vaated kärbitakse täpselt samamoodi). Uus versioonis 1.3.0: toode, milles vähemalt üks piksel on kärbitud, saab kaasasoleva **kärbimismaski**, mis näitab täpselt, millised pikslid ja lainepikkused on kärbitud:

* **Kus:**kaustas `clamp_masks/`**tootekastide kõrval**, mitte kunagi nende sees: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` ja `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Mis:** ühe lainepikkusega `uint8`-TIFF-fail toote enda pikselvõrgustikus (koos orientatsiooni, uuritava piirkonna (ROI), binningu ja massiivi joondamisega). Bit 0 (`mask & 1`) on seatud seal, kus toote R-kanali laine oli kärbitud, bit 1 (`& 2`) G-kanalil, bit 2 (`& 4`) B-kanalil. Lõigatud pikslite osakaal iga sagedusriba kohta on tootele märgitud ka kui `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` ja `Chloros:BandBUnmixClampFraction`.
* **Kuidas toode seda väljendab:** XMP-märgis `Chloros:UnmixClampMask` sisaldab maski asukohta toote kausta suhtes (`../clamp_masks/<product folder>/<name>_clampmask.tif`, alati `/`-ga eraldatud), `none`, kui ühtegi pikslit ei lõigatud (maski ei kirjutatud), või `unavailable`, kui maski ei olnud võimalik kirjutada. Sildita toode ei ole kunagi läbinud värvide eraldamist (RGB- ja monokaamerad, toored ja debayered tasemed).

Kuna üheski tootekausta ei ole maski, võite edastada tootekausta, näiteks `Reflectance_Calibrated_Images/`, Pix4D-le, Metashape’ile või muule fotogramm-meetria tarkvarale sellisena, nagu see on, ka alamkaustadega – edastage tootekausta, mitte selle kohal asuvat formaatikausta. Chlorose enda importimisfunktsioon jätab klambrimaskid vahele. CLI loendab neid eraldi: `Image products written: N (plus M clamp masks)`.

## Indeksitooted

Iga indeks paigutatakse oma `<INDEX>_Index_Images/`-kausta (nt `NDVI_Index_Images/`).

* **Täisarvformaadid**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) salvestavad**pildi-põhise lineaarse venituse**, mitte −1…1 kaarti: DN 0 on selle kaadri madalaim indeksväärtus ja täisskaala on kõrgeim. Fail sisaldab `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` ja `Chloros:IndexMax`, seega ka `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Kuna igal kaadril on oma otspunktid, võrdle kaadreid väärtustena, mitte DN-na.
* **`TIFF (32-bit, Percent)`** salvestab indeksväärtused ise float32-tüübina, ilma ümberskaalimiseta — see on parim valik kvantitatiivseks tööks kogu lennu ulatuses.

Uus versioonis 1.3.0 kirjeldab indeksitoode **iseennast**, selle asemel et korrata allika toorandmete sildid. Selle GPS- ja kaamerasildid kopeeritakse tootest, mille põhjal see arvutati (peegeldusvõime toode, kui see on olemas), ja seejärel:

| Silt | Väärtus |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (seega faili ei impordita kunagi uuesti pildina) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / kasutatud valem |
| `Chloros:IndexLevel` | `reflectance` või `radiance` — millest indeks arvutati (puudub, kui kumbki ei kehti) |
| `Chloros:IlluminationInvariant` | `True`, kui indeks on arvutatud peegeldusvõime põhjal, `False`, kui kiirguse või anduri väärtuste põhjal |
| `Chloros:ChlorosPipelineVersion` | indeksit kirjutanud Chloros-versioon |

`Chloros:PixelScale` on indeksitoodetest eemaldatud — kasuta ülaltoodud `IndexMin` / `IndexMax`. Täielik tagasilugemise juhend on jaotises „Täisarvuline indeksraster on pildi-põhine kontrasti venitamine” [CLI-viites](reference/cli-reference.md).

## Peegeldusvõime pikselväärtuste lugemine

Peegeldusvõime salvestatakse täisarvulise digitaalarvuna ning **DN, mis tähistab ρ = 1,0 (100% peegeldusvõime), sõltub allikakaamerast**:

| Allikakaamera | ρ = 1,0 on DN | Kuidas kindlaks teha |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (varu kuni ρ 2,0) | Failile on lisatud XMP-märge `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (piiratud ρ 1,0 juures) | `Chloros:PixelScale`-märget pole – selle puudumine ongi märk. (Alates versioonist 1.3.0 sisaldavad Survey3-tooted igas töötlemisrežiimis ka teisi `Chloros:*` töötlemismärke, näiteks `Chloros:CalibrationApplied`.) |

**Loe `Chloros:PixelScale` XMP-märget ja jaga sellega**, selle asemel et eeldada konstanti. Silt on defineeritud uint16-domeenis, seega jääb see `32768`-iks ka väljundvormingutes, mis teevad ümberskaalamist — normaliseerige salvestatud andmetüüp esmalt tagasi uint16-ks (8-bitisest ×257, float32-st ×65535).

{% hint style="warning" %}
**8-bitised salvestused.** 8-bitise salvestuse eksporditud andmed on 8-bitised. Selle peegeldusvõime skaleeritakse ümber ja märgistatakse `Chloros:PixelScale=32768`-iga, nii et skaala säilib; 8-bitised DN-tooted ja enne versiooni 1.3.0 salvestatud peegeldusvõimefailid on kärbitud vahemikku 0–255 ja neil puudub skaalamärge. Kui LATTICE peegeldusvõimefailil puudub see märge, ärge eeldage skaalat; eksportige see asemel uuesti 16-bitisena või 32-bitisena.
{% endhint %}

Täielikud reeglid (sh MicaSense’iga ühilduvad märgised) leiate jaotisest **„Peegeldusvõime pikslite lugemine”** [CLI-viites](reference/cli-reference.md).
