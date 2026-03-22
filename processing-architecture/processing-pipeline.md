# Töötlemisprotsess

Chloros 1.1.0 kasutab neljatoalist töötlemisprotsessi, mis toimib etapiviisilise konveierina. Iga protsessitoiming tegeleb töötlemisvoo eraldi etapiga, võimaldades mitme pildi samaaegset töötlemist erinevates etappides.

***

## Protsessi arhitektuur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Iga pilt läbib järjest kõik neli niiti. Chloros+ mitme niidiga töötlemise abil võivad mitu pilti olla samaaegselt erinevates niitides – kui niit 3 töötleb ühte pilti, võib niit 1 tuvastada järgmist, niit 2 kalibreerida teist ja niit 4 kirjutada eelnevalt töödeldud pilti kettale.

***

## Niitide üksikasjad

### Lõng 1: Tuvastamine

**Eesmärk**: Laadida pilte ja tuvastada kalibreerimise sihtmärke.

* Loeb pildifaile kettalt (RAW, JPG)
* Ekstraktib EXIF-metadata (GPS, kaameramudel, ajamärgid, säri)
* Tuvastab ArUco kalibreerimise sihtmärgid märgistatud sihtmärgipiltidel
* Väljundid: pildid + metadata + sihtmärkide tuvastamise tulemused

See on peamiselt I/O- ja CPU-sõltuv niit.

### Niit 2: Kalibreerimine

**Eesmärk**: Arvutada kalibreerimise parameetrid tuvastatud sihtmärkide põhjal.

* Arvutab sihtmärkide piltidest peegelduskalibreerimise koefitsiendid
* Arvutab vinjeti korrigeerimise parameetrid
* Määrab kindlaks iga lainepikkuse kalibreerimiskõverad
* Väljundid: kalibreerimise parameetrid iga pildi jaoks

See on CPU-mahukas arvutusniit.

### Niit 3: Töötlemine (GPU)

**Eesmärk**: Rakendada parandusi ja arvutada taimestiku indeksid.**See on kõige arvutusmahukam niit.*** **Debayering**: teisendab RAW-vormingus olevad Bayeri mustri andmed mitmekanalilisteks piltideks
  * Standard (kiire, keskmine kvaliteet) — vaikimisi
  * Tekstuuritundlik (aeglane, kõrgeim kvaliteet) — ainult Chloros+, kasutab AI/ML müra eemaldamist
* **Vignette-korrektsioon**: rakendab objektiivi vignette-korrektsiooni kogu pildile
* **Peegelduskalibreerimine**: rakendab kalibreerimiskoefitsiente peegeldusväärtuste teisendamiseks
* **Indeksi arvutamine**: arvutab taimestiku indeksid (NDVI, NDRE, GNDVI jne)
* Väljundid: eksportimiseks valmis töödeldud pildid

See protsess saab GPU kiirendusest kõige enam kasu. [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) süsteem optimeerib peamiselt selle protsessi käitumist.

### Protsess 4: Eksport

**Eesmärk**: Kirjutada töödeldud pildid kettale.

* Kirjutab väljundfailid valitud formaadis (TIFF 16-bitine, TIFF 32-bitine %, PNG, JPG)
* Lisab väljundfailidesse EXIF-metadata (GPS, ajamärgid, töötlemisparameetrid)
* Korraldab väljundi kaameramudelite alamkaustadesse
* Väljundid: lõplikud failid kettal

See on peamiselt sisend-väljundiga seotud niit. SSD-salvestusruum parandab oluliselt niidi 4 jõudlust.

***

## Järjestikune vs. torustatud töötlemine

### Vaba režiim (järjestikune)

Chloros vabaversioonis töödeldakse pilte **ükshaaval**, järjestikku läbi kõigi nelja etapi:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI-i edenemisribal on näha 2 etappi: sihtmärgi tuvastamine ja töötlemine.

### Chloros+ režiim (torustatud)

Chloros+ litsentsiga töötavad kõik neli protsessisäitu **paralleelselt** erinevate piltidega:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI-i edenemisribal on näha 4 etappi: tuvastamine, analüüsimine, kalibreerimine, eksportimine. Viige hiir edenemisribale, et näha iga protsessisäiu edenemist.

{% hint style="success" %}
**Järjekorras töötlemine Chloros+ abil** võib olla 3–5 korda kiirem kui järjestikune töötlemine, sõltuvalt teie riistvarast ja andmekogumi suurusest. Kiiruse kasv on suurim süsteemides, millel on kiired GPU-d ja SSD-d.
{% endhint %}

***

## 4. niidi ekspordi edenemine

Chloros versioonis 1.1.0 on ekspordi niidil (niit 4) oma spetsiaalne edenemise jälgimise süsteem. Ekspordi edenemist saate jälgida eraldi:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Töötlemine on lõpetatud, kui niit 4 jõuab 100%ni.

***

## Seos dünaamilise arvutusadapteerimisega

[Dünaamilise arvutusadapteerimise](dynamic-compute-adaptation.md) süsteem mõjutab peamiselt **niiti 3 (töötlemine)**:

* **`GPU_PARALLEL`** strateegia: niit 3 töötleb GPU kaudu mitut pilti korraga, kasutades `fused_gpu` torujuhet
* **`GPU_SINGLE`** strateegia: Thread 3 töötleb ühe pildi korraga, kasutades mälu säästvat `tiled_gpu` torujuhet
* **`CPU_PARALLEL`** strateegia: Thread 3 kasutab CPU-põhist töötlemist mitmeprotsessilise paralleelsusega

Thread 3 GPU mälu eraldamine muutub dünaamiliselt ka siis, kui Thread 1 ja 2 lõpetavad töö — vaata [Dynamic GPU Memory Allocation](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Järgmised sammud

* [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) — Kuidas Chloros valib teie riistvarale optimaalse strateegia
* [NVIDIA Jetsoni juhend](../linux/nvidia-jetson-guide.md) — Platvormispetsiifiline töötlemisprotsessi käitumine Jetsonil
* [Töötlemise jälgimine](../processing-images-gui/monitoring-the-processing.md) — GUI-põhine protsessi jälgimine
