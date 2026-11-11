# Töötlemisprotsess

Chloros versioonis 1.3.0 kasutatakse nelja niidiga töötlemisprotsessi, mis toimib nagu etapiviisiline konveierliin. Iga niit tegeleb töövoo eraldi etapiga, mistõttu võib korraga olla töötlemisel mitu pilti erinevates etappides.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Töötlemisprotsessi arhitektuur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Iga pilt läbib järjest kõik neli niiti. Chloros+ mitme niidiga töötlemise puhul hõivavad mitu pilti samaaegselt erinevaid niite – kui niit 3 töötleb ühte pilti, võib niit 1 tuvastada järgmist, niit 2 kalibreerida veel ühte ja niit 4 kirjutada valmis pilti kettale.

Töö edenemist kajastatakse iga niidi kohta eraldi ning see edastatakse Server-Sent Eventsi kaudu (tagapõhi avaldab need `/api/events`-is). CLI reaalajas edenemise näidikus on neli etappi märgistatud **Tuvastamine, Analüüsimine, Töötlemine, Eksportimine**.***

## Niiditeave

### Niit 1: Tuvastamine

**Eesmärk**: Piltide laadimine ja kalibreerimise sihtmärkide tuvastamine.

* Loeb pildifaile kettalt — Survey3i `.raw`+`.jpg` paarid, LATTICE `.tif`/`.tiff` pildid ning `.dng`
* Eraldab EXIF-metadata (GPS, kaameramudel, ajamärgid, säriaeg)
* Tuvastab kalibreerimismärgid: ArUco-märgistatud sihtmärkide geomeetriad LATTICE-pildistuste jaoks ning klassikalise paneelidetektori Survey3 kalibreerimismärkide fotode jaoks
* Väljundid: pildid + metaandmed + sihtmärkide tuvastamise tulemused

Peamiselt sisend-väljund- ja protsessorikoormusega seotud protsess.

### Niit 2: Kalibreerimine

**Eesmärk**: Arvutada kalibreerimisparaameetrid tuvastatud sihtmärkide põhjal.

* Arvutab sihtmärkide piltidest peegelduskalibreerimise koefitsiendid
* Arvutab vinjeti korrigeerimise parameetrid
* Määrab kindlaks iga lainepikkuse jaoks kalibreerimiskõverad
* Väljundid: iga pildi kalibreerimisparaameetrid

CPU-mahukas arvutusniit. Survey3- või segakäituse korral ootab niit 3 seda, kui peegelduskalibreerimine on sisse lülitatud, nii et selle koefitsiendid on valmis enne ühegi pildi töötlemist. Ainult LATTICE-käituse korral töötab niit 3 tuvastamise ajal: iga kaader algab, kui selle sihtmärgi ankur on lõplik. Kaader võtab ajaliselt lähima sihtmärgi mõlemalt poolt, seega ootab see, kuni on skaneeritud kõik selle kaamera kaadrid, mis on ajaliselt lähemal kui selle praegune lähim sihtmärk. Tulemused on samad, ainult varem valmis.

### Niit 3: Töötlemine (GPU)

**Eesmärk**: rakendada parandusi ja arvutada taimestiku indeksid.**See on arvutusmahukaim niit.*** **Debayering**: teisendab RAW-Bayer-andmed mitmekanalilisteks piltideks
  * Standard (kiire, keskmine kvaliteet) — vaikimisi, `--debayer standard`
  * Tekstuuritundlik (aeglane, kõrgeim kvaliteet) — ainult Chloros+, `--debayer texture-aware`, kasutab AI/ML müra eemaldamise mudelit
  * LATTICE mono (M3M) pildid on ühekanalilised: nende puhul jäetakse demosaik ja valgebalansi etappid vahele (koos üherealise logisõnumiga), samas kui samas töökäigus olevad M3C/Bayer-pildid läbivad need ikkagi
* **Vignettkorrektsioon**: rakendab kogu pildile objektiivi vignettkorrektsiooni
* **Peegelduskalibreerimine**: rakendab kalibreerimiskoefitsiente, et teisendada väärtused peegeldusväärtusteks
* **Indeksi arvutamine**: arvutab taimestiku indeksid (NDVI, NDRE, GNDVI, …)
* Väljundid: eksportimiseks valmis töödeldud pildid

See protsessisaatja saab GPU kiirendusest kõige enam kasu ning just seda protsessisaatjat häälestab [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Protsess 4: Eksport

**Eesmärk**: kirjutada töödeldud pildid kettale.

* Kirjutab väljundfailid valitud formaadis — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Lisab väljundfailidesse metaandmed (GPS, ajamärgised, töötlemisparameetrid) — sealhulgas `Chloros:*` töötlemismärgised, mida alates versioonist 1.3.0 kirjutatakse igas töötlemisrežiimis, nii Survey3-s kui ka LATTICE-s
* Korraldab väljundfailid projektikausta alla nimega `<camera>/<format>/<Product>_Images/` — näiteks `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Eksporditud failid säilitavad allikafaili nime; kaust identifitseerib toote.**
* LATTICE-pildistamiste puhul võib üks allikakaader jaguneda mitmeks tooteks (Debayered, Preview, Radiance, Reflectance, Index), millest igaüks asub oma tootekausis. Indeksitooted arvutatakse töötluse käigus loodud peegeldusvõime toote põhjal (kui seda pole, siis mõne teise töötluse käigus loodud toote, näiteks kiirguse põhjal). Multispektraalsete M3C-kaamerate kiirguse ja peegeldusvõime tooted võivad saada ka pikselipõhise piiramismaski failis `<format>/clamp_masks/` (vt [Väljundpildi formaadid](../output-image-formats.md#clamp-masks))
* Väljundid: lõplikud failid kettal

Peamiselt sisend-väljundiga seotud niit — SSD-salvestus parandab selle jõudlust märgatavalt.

***

## Sisemine toimimine: täitjad

Niidis 3 on pildipõhine töö paralleelistatud järgmiselt:

* **GPU-strateegiad**(`GPU_SINGLE`, `GPU_PARALLEL`) kasutavad**tööprotsesside** kogumit, millest igaüks on eraldi operatsioonisüsteemi protsess oma CUDA-kontekstiga. Chloros käivitab iga tööprotsessi, käivitades uuesti selle enda installitud tagapõhiprogrammi – mitte Pythoni `multiprocessing` kaudu, mille tööprotsesside käivitamine pakendatud rakenduse sees ei tööta
* **`CPU_PARALLEL`** kasutab niidipooli (Pythoni `ThreadPoolExecutor`) — NumPy ja OpenCV vabastavad GIL-i, seega piisab niitidest
* Jetson-seadmed, millel on 8 GB või vähem jagatud RAM-i, jätavad täitja täielikult vahele ja töötlevad protsessi sees järjestikku
* Texture Aware töötab ka GPU-l, millel on alla 7 GB VRAM-i, järjestikku — müra eemaldaja mudel ei mahu sinna rohkem kui üks kord

Chloros ei kasuta ühtegi kolmanda osapoole hajutatud raamistikku (nagu Ray). Vaata [Dynamic Compute Adaptation](dynamic-compute-adaptation.md), et teada saada, kuidas valitakse strateegia ja töötajate arv.

***

## Järjestikune vs. torustatud töötlemine

### Vaba režiim (järjestikune)

Chlorosi tasuta versioonis töödeldakse pilte **ükshaaval** järjestikku kõigi nelja etapi kaudu:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI kuvab vabarežiimis lihtsustatud edenemisriba; selle järjestikused faasid on märgitud kui **Sihtmärgi tuvastamine**ja seejärel**Töötlemine**.

### Chloros+ režiim (torustikupõhine)

Chloros+ litsentsi korral töötavad kõik neli niiti **paralleelselt** erinevate piltidega:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI edastusribal kuvatakse neli etappi; viige hiir selle kohale, et näha iga niidi edusamme. CLI-s kuvatakse samad neli etappi reaalajas järgmiselt: **Detecting, Analyzing, Processing, Exporting**.

{% hint style="info" %}
**Üks märge, kaks nime.** CLI-s nimetatakse 3. etappi _Töötlemine_. Backendi premium-režiimi edastatav edusammude voog – see, mida graafiline kasutajaliidese edusammuriba kuvab – märgistab sama etapi _Kalibreerimine_. Tegemist on sama protsessiga, mis teeb sama tööd (protsess 3: debayer, parandused, indeksid).
{% endhint %}

{% hint style="success" %}
**Chloros+ abil toimuv torutöötlus** võib olla 3–5 korda kiirem kui järjestikune töötlus, sõltuvalt teie riistvarast ja andmekogumi suurusest. Kiiruse kasv on suurim süsteemides, kus on kiired GPU-d ja SSD-d.
{% endhint %}

***

## 4. protsessikeere ekspordi edenemine

Ekspordi protsessikeerel on oma edenemise jälgimine, mida saate eraldi kontrollida:**KäskureaCLI**:

```bash
chloros-cli export-status
```

**SDK**:

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Töötlemine on lõppenud, kui niit 4 jõuab 100%ni.

{% hint style="info" %}
**Käivitus, mille käigus pilte ei kirjutata, on ebaõnnestunud.**Eduka tulemuse korral teatab `chloros-cli process`, kui palju pilditooteid kirjutati (`Image products written: N`). Kui tooteid taotleti, kuid**ühtegi**ei kirjutatud — ainult `project.json` ja `calibration_data.json` —, kuvab CLI `Processing finished but wrote no image products.` ja**lõpetab töötamise nullist erineva tulemusega**, nimetades projektikausta ja tavalised põhjused (sisendkaustat ei tunnistatud salvestusena – kontrollige paigutust ja `--input-level` – või kõik taotletud tooted olid nende kaamerate jaoks sobimatud). Skriptid võivad tugineda väljumiskoodile.
{% endhint %}

***

## Seos dünaamilise arvutusadapteerimisega

[Dünaamiline arvutusadapteerimine](dynamic-compute-adaptation.md) mõjutab peamiselt **niiti 3 (töötlemine)**:

* **`GPU_PARALLEL`**: 3. niit töötleb GPU kaudu samaaegselt mitut pilti, kasutades `fused_gpu`-töötlusliini
* **`GPU_SINGLE`**: 3. niit järjestab GPU-le juurdepääsu semafori abil, samal ajal kui tööprotsessid teevad I/O-operatsioone kattuvalt, kasutades `fused_gpu` või mälusäästlikku `tiled_gpu` torujuhet
* **`CPU_PARALLEL`**: 3. niit kasutab CPU-põhist töötlemist mitme niidiga paralleelsusega

Niidi 3 GPU-mälu eraldus suureneb ka siis, kui niidid 1 ja 2 lõpetavad töö — vt [Dynaamiline GPU-mälu eraldus](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Järgmised sammud

* [Dünaamiline arvutusvõimsuse kohandamine](dynamic-compute-adaptation.md) — Kuidas Chloros valib teie riistvarale optimaalse strateegia
* [NVIDIA Jetsoni juhend](../linux/nvidia-jetson-guide.md) — Platvormispetsiifiline töötlusvoo käitumine Jetsonil
* [Töötlemise jälgimine](../processing-images-gui/monitoring-the-processing.md) — GUI kaudu töötlemise kulgemise jälgimine
* [Käsurea CLI-viide](../reference/cli-reference.md) — `process`, `export-status`, väljumiskoodid ja väljundi paigutus
