# Töötlemisprotsess

Chlorosversioonis 1.2.0 kasutatakse nelja niidiga töötlemisprotsessi, mis toimib nagu etapiviisiline konveierliin. Iga niit tegeleb töövoo eraldi etapiga, mistõttu võib korraga olla töös mitu pilti erinevates etappides.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Töötlemisprotsessi arhitektuur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Iga pilt läbib järjest kõik neli niiti. Chloros+ mitme niidiga töötlemise puhul hõivavad mitu pilti samaaegselt erinevaid niite – kui niit 3 töötleb ühte pilti, võib niit 1 tuvastada järgmist, niit 2 kalibreerida teist ja niit 4 kirjutada valmis pilti kettale.

Töö edenemist kajastatakse iga niidi kohta eraldi ning andmed edastatakse Server-Sent Eventsi kaudu (tagapõhi avaldab need `/api/events`-is). CLI reaalajas edenemise kuvamisel on neli etappi märgistatud **Tuvastamine, Analüüsimine, Töötlemine, Eksportimine**.***

## Niiditeave

### Niit 1: Avastamine

**Eesmärk**: Laadida pildid ja tuvastada kalibreerimise sihtmärgid.

* Loeb pildifaile kettalt — Survey3 `.raw`+`.jpg` paarid, LATTICE `.tif`/`.tiff` pildid ja `.dng`
* Eraldab EXIF-metadata (GPS, kaamera mudel, ajamärgid, säriaeg)
* Tuvastab kalibreerimismärgid: ArUco-märgistatud sihtmärkide geomeetriad LATTICE-pildistamiste puhul ning klassikalise paneeldetektori puhul Survey3 kalibreerimismärkide fotode korral
* Väljundid: pildid + metaandmed + sihtmärkide tuvastamise tulemused

Peamiselt sisend-väljund- ja protsessorikoormusega seotud niit.

### Protsess 2: Kalibreerimine

**Eesmärk**: Arvutada kalibreerimisparaameetrid tuvastatud sihtmärkide põhjal.

* Arvutab sihtmärkide piltidest peegelduskalibreerimise koefitsiendid
* Arvutab vinjeti korrigeerimise parameetrid
* Määrab kindlaks iga laineala kalibreerimiskõverad
* Väljundid: iga pildi kalibreerimise parameetrid

Protsessoriga seotud arvutusniit. Niit 3 ootab selle lõpetamist, kui peegelduskalibreerimine on sisse lülitatud, et koefitsiendid oleksid valmis enne piltide töötlemist.

### Niit 3: Töötlemine (GPU)

**Eesmärk**: Rakendada korrektsioone ja arvutada taimestiku indeksid.**See on arvutusmahukaim niit.*** **Debayering**: teisendab RAW-Bayer-andmed mitmekanalilisteks piltideks
  * Standard (kiire, keskmine kvaliteet) — vaikimisi, `--debayer standard`
  * Tekstuuritundlik (aeglane, kõrgeim kvaliteet) — ainult Chloros+, `--debayer texture-aware`, kasutab AI/ML müra eemaldamise mudelit
  * LATTICE mono (M3M) pildid on ühekanalilised: nende puhul jäetakse demosaik ja valgebalansi etappid vahele (koos üherealise logisõnumiga), samas kui samas seerias olevad M3C/Bayer-pildid läbivad need ikkagi
* **Vignettkorrektsioon**: rakendab kogu pildile objektiivi vignettkorrektsiooni
* **Peegeldusvõime kalibreerimine**: rakendab kalibreerimiskoefitsiente, et teisendada väärtused peegeldusvõimeks
* **Indeksi arvutamine**: arvutab taimestiku indeksid (NDVI, NDRE, GNDVI, …)
* Väljundid: eksportimiseks valmis töödeldud pildid

See protsessisaat saab kõige enam kasu GPU kiirendusest ning just seda protsessisaat [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optimeerib.

### Protsessisaat 4: Eksport

**Eesmärk**: kirjutada töödeldud pildid kettale.

* Kirjutab väljundfailid valitud formaadis — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Lisab väljundfailidesse metaandmed (GPS, ajamärgid, töötlemisparameetrid)
* Korraldab väljundid projektikausta alla nimedega `<camera>/<format>/<Product>_Images/` – näiteks `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Eksporditud failid säilitavad allikafaili nime; toodet identifitseerib kaust.**
* LATTICE-pildistamiste puhul võib üks allikakaader jaguneda mitmeks tooteks (Debayered, Preview, Radiance, Reflectance, Index), millest igaüks asub oma tootekausis
* Väljundid: lõplikud failid kettal

Peamiselt sisend-väljundiga seotud niit — SSD-salvestusruum parandab selle jõudlust märgatavalt.

***

## Sisemine toimimine: täitjad

Niidis 3 on pildipõhine töö paralleelistatud „Python” standardiga `concurrent.futures`:

* **GPU-strateegiad**(`GPU_SINGLE`, `GPU_PARALLEL`) kasutavad `ProcessPoolExecutor`-i koos**spawn** — iga töötaja on eraldi protsess, millel on oma CUDA-kontekst (`fork` päriks vanema algseadistatud CUDA-seisundi ja rikuks järglaste seisundi)
* **`CPU_PARALLEL`** kasutab `ThreadPoolExecutor`-i — NumPy ja OpenCV vabastavad GIL-i, seega piisab niitidest
* Jetson-seadmed, millel on 8 GB või vähem jagatud RAM-i, jätavad täitja täielikult vahele ja töötlevad protsessi sees järjestikku
* Texture Aware töötab ka GPU-l, millel on alla 7 GB VRAM-i, järjestikku — müra eemaldamise mudel ei mahu sinna rohkem kui üks kord

Chlorosei kasuta ühtegi kolmanda osapoole hajutatud raamistikku (nagu Ray). Vaata [Dynamic Compute Adaptation](dynamic-compute-adaptation.md), et teada saada, kuidas valitakse strateegia ja töötajate arv.

***

## Järjestikune vs. torustikupõhine töötlemine

### Vaba režiim (järjestikune)

Chlorosi tasuta versioonis töödeldakse pilte **ükshaaval** järjestikku kõigi nelja etapi kaudu:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI kuvab vabarežiimis lihtsustatud edenemisriba; selle järjestikused etapid on märgitud kui **Sihtmärgi tuvastamine**ja seejärel**Töötlemine**.

### „Chloros+“ režiim (järkjärguline)

„Chloros+“ litsentsi korral töötavad kõik neli niiti **paralleelselt** erinevate piltidega:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI edastusribal on näha neli etappi; viige hiir selle peale, et näha iga niidi edusamme. „CLI’is” kuvatakse samad neli etappi reaalajas järgmiselt: **Detecting, Analyzing, Processing, Exporting**.

{% hint style="info" %}
**Üks märge, kaks nime.** „CLI” nimetab 3. etappi _Töötlemine_. Backendi premium-režiimi edastusvoog – see, mida GUI edastusriba kuvab – märgib sama etappi _Kalibreerimine_. Tegemist on sama protsessiga, mis teeb sama tööd (protsess 3: debayer, parandused, indeksid).
{% endhint %}

{% hint style="success" %}
**Chloros-ga** teostatav torustatud töötlemine võib olla 3–5 korda kiirem kui järjestikune töötlemine, sõltuvalt teie riistvarast ja andmekogumi suurusest. Kiiruse kasv on suurim süsteemides, kus on kiired GPU-d ja SSD-d.
{% endhint %}

***

## 4. niidi ekspordi edenemine

Ekspordi niidil on oma edenemise jälgimissüsteem, mida saate eraldi küsida:**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Töötlemine on lõppenud, kui niit 4 jõuab 100%ni.

{% hint style="info" %}
**Käivitus, mille käigus pilte ei kirjutata, on ebaõnnestunud.**Eduka tulemuse korral teatab `chloros-cli process`, kui palju pilditooteid kirjutati (`Image products written: N`). Kui tooteid taotleti, kuid**ühtegi**ei kirjutatud — ainult `project.json` ja `calibration_data.json` —, siis väljastab CLI `Processing finished but wrote no image products.` ja**lõpetab töötamise nullist erineva väärtusega**, nimetades projekti kausta ja tavalised põhjused (sisendkausta ei tunnistatud salvestusena – kontrollige paigutust ja `--input-level` – või kõik taotletud tooted olid nende kaamerate jaoks sobimatud). Skriptid võivad tugineda väljumiskoodile.
{% endhint %}

***

## Seos dünaamilise arvutusadapteerimisega

[Dünaamiline arvutusadapteerimine](dynamic-compute-adaptation.md) mõjutab peamiselt **niiti 3 (töötlemine)**:

* **`GPU_PARALLEL`**: 3. niit töötleb GPU kaudu samaaegselt mitut pilti, kasutades `fused_gpu`-töötlusliini
* **`GPU_SINGLE`**: Thread 3 järjestab GPU-le juurdepääsu semafori abil, samal ajal kui tööprotsessid teevad I/O-operatsioone üheaegselt, kasutades `fused_gpu` või mälusäästlikku `tiled_gpu` torujuhet
* **`CPU_PARALLEL`**: 3. niit kasutab CPU-põhist töötlemist mitme niidiga paralleelsusega

Niidi 3 GPU-mälu eraldus suureneb ka siis, kui niidid 1 ja 2 lõpetavad töö — vt [Dynamic GPU Memory Allocation](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Järgmised sammud

* [Dünaamiline arvutusvõimsuse kohandamine](dynamic-compute-adaptation.md) — Kuidas „Chloros“ valib teie riistvarale optimaalse strateegia
* [NVIDIA Jetsoni juhend](../linux/nvidia-jetson-guide.md) — Platvormispetsiifiline töötlemisprotsessi käitumine Jetsonil
* [Töötlemise jälgimine](../processing-images-gui/monitoring-the-processing.md) — Tööde edenemise jälgimine graafilise kasutajaliidese kaudu
* [„CLI“ viide](../reference/cli-reference.md) — `process`, `export-status`, väljumiskoodid ja väljundi paigutus
