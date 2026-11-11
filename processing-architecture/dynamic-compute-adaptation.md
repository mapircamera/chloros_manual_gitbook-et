# Dünaamiline arvutusvõimsuse kohandamine

Chloros 1.3.0 kasutab riistvara tuvastamist ja töötlemisstrateegia automaatset valikut. Töötlemismootor kohandub teie riistvaraga – alates Jetson Orin Nanost kuni mitme GPU-ga tööjaamani – ilma igasuguse käsitsi konfigureerimiseta.

***

## Kuidas see toimib

Kui Chloros käivitub, koostab ta teie süsteemi profiili:

1. **Tuvastab operatsioonisüsteemi** – Windows või Linux
2. **Määrab kindlaks CPU tuumad ja RAM-mälu kogumahu**

3.**Tuvastab GPU olemasolu** – NVIDIA CUDA toetuse, VRAM-i, mudeli
4. **Määrab kindlaks Jetsoni mudeli** (kui see on olemas) – `/proc/device-tree/model` kaudu
5. **Kontrollib soojusandureid** (Jetson) – temperatuuri arvestava töötlemise jaoks
6. **Valib arvutusstrateegia** – kõigi tuvastatud riistvarade põhjal
7. **Konfigureerib töötajate arvu, töötlemisjärjekorra tüübi ja mälu eraldamise** automaatselt

Tuvastatud profiil salvestatakse seansi jaoks vahemällu ja kettale, et hilisemad käivitused algaksid kiiremini:

| Platvorm | Vahemällu salvestatud profiil |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (võtab arvesse `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Kustutage see fail, et sundida süsteemi profiili uuesti tuvastama – see on kasulik pärast GPU või täiendava RAM-i lisamist. Chloros tuvastab profiili automaatselt uuesti ka juhul, kui vahemälu on kirjutatud ühilduvuse puudumisega vanema versiooni poolt.

***

## Arvutusstrateegiad

Chloros valib teie riistvara põhjal ühe kolmest arvutusstrateegiast:

| Strateegia | Valitakse, kui | Töötajad | Täitja | Töötlemisahel |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA-graafikakaart, mis näitab**12 GB+ VRAM**(Jetsoni ühtsel mälul, nõuab ka vähemalt 12 GB jagatud RAM-i) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, vähemalt 2 —**Jetsonil piiratud 2-ga** | Tööprotsessid (eraldi CUDA-kontekstid) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA GPU**2–12 GB VRAM-iga**| 3 (I/O kattuvus; GPU-le juurdepääs on semaforiga järjestatud).**1 (järjekorras) Jetsonitel, mille RAM-mälu on alla 12 GB** | Tööprotsessid (eraldi CUDA-kontekstid); järjekorras protsessisiseselt väikese RAM-mäluga Jetsonitel | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | CUDA-graafikaprotsessor puudub või vähem kui 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

`GPU_PARALLEL` töötaja valemi näited: 12 GB VRAM → 2 töötajat, 16 GB → 3 töötajat, 20 GB+ → 4 töötajat. Iga töötajale on eraldatud 3,5 GB, mis on tekstuuriteadliku ülesande mõõdetud vajadus, jäädes 90% piiresse VRAM-ist, mis jääb alles pärast 0,5 GB reservi.

{% hint style="info" %}
**GPU-strateegiad vajavad PyTorchi, mis suudab kasutada CUDA-d.** Seade, mille PyTorchil puudub kasutatav CUDA-seade, saab `CPU_PARALLEL`, olenemata selle GPU-st. Jetsoni 1.3.0 pakett kasutab JetPack 6 CUDA-versiooni PyTorchist, seega vastab Jetson nõuetele, kui JetPacki CUDA- ja cuDNN-teegid on paigaldatud. Alla 12 GB RAM-iga Jetson töötleb alati ühte pilti korraga, protsessi käigus (logitakse kui `GPU_SINGLE`).
{% endhint %}

GPU-strateegiad käitavad tööprotsesside kogumit, millest igaüks on eraldi operatsioonisüsteemi protsess oma CUDA-kontekstiga, mille Chloros käivitab, käivitades uuesti oma installitud tagapõhiprogrammi (mitte Python'i `multiprocessing` kaudu, mille tööprotsesside käivitamine ei tööta pakendatud rakenduse sees). CPU-strateegia kasutab niidipooli (Python i `ThreadPoolExecutor`). Chloros ei kasuta ühtegi kolmanda osapoole hajutatud raamistikku (nagu Ray).

### Töötlemisliinide tüübid

* **`fused_gpu`** — Täielik GPU-töötlemisliin. Debayeri, korrektsiooni ja indeksimise operatsioonid käivitatakse GPU-l ühe ühendatud läbimisega. Kõrgeim läbilaskevõime, vajab kõige rohkem VRAM-i.
* **`tiled_gpu`** — Mälu säästev GPU-töötlusviis. Töötleb pilte ruutudena, et need mahuksid piiratud GPU-mälu sisse. Madalam läbilaskevõime, kuid töötab mälupiirangutega seadmetel.
* **`cpu_fallback`** — Ainult CPU-põhine töötlemine, kasutades mitmeprotsessilist paralleelsust. Kasutatakse juhul, kui NVIDIA GPU pole saadaval, ning viimase abinõuna, kui mõlemad GPU-töötlemisteed ebaõnnestuvad.

Käitamisaja varuvariandi ahel on alati `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Strateegia käsitsi ümberkirjutamine

Määrake keskkonnamuutuja `CHLOROS_STRATEGY`, et sundida kasutama kindlat strateegiat — see on ekspertide varuplaan juhuks, kui automaatne tuvastamine valib teie olukorrale sobimatu lahenduse (näiteks GPU vabana hoidmine muude ülesannete jaoks):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Muutuja sobitamine toimub suurtähti arvestamata; kõike, mis ei kuulu kolme nimetatud nime hulka, ignoreeritakse ja automaatne tuvastamine jätkub tavapäraselt. Ülekirjutamise korral valib Chloros töötajate arvu ikkagi teie eest:

| Ülekirjutamine | Kasutatud töötajate arv |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Ülekirjutamine `GPU_SINGLE` või `GPU_PARALLEL` masinal, millel puudub kasutatav CUDA-seade, lükatakse tagasi ja logisse kirjutatakse rida `[STRATEGY]`, ning käivitamisel kasutatakse `CPU_PARALLEL`-i.

Soovitav on seadistada seda iga käsu jaoks eraldi, mitte püsivalt, et tavapärased käivitamised saaksid jätkata automaatset kohandamist.

***

## Platvormispetsiifiline käitumine

| Platvorm | Strateegia | Töötajad | Töötlemisahel | Märkused |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (järjekorras) | Mälu säästev režiim, üks pilt korraga |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (järjekorras) | Alla 12 GB jagatud RAM-mälu sunnib töötlema järjekorras |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (paralleelne) | Soovitatav servaseade — Jetsonil on piiratud 2 töötajaga |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (paralleelne) | Maksimaalne servaseadme jõudlus (ka Jetsonil on piiratud 2 töötajaga) |
| **Töölauaarvuti 8 GB GPU-ga** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 töötaja I/O kattub, samal ajal kui semafor järjestab GPU-le juurdepääsu |
| **Töölauaarvuti 12 GB+ GPU-ga** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (paralleelselt) | Optimaalne lauaarvuti jõudlus: 12 GB → 2 töötajat, 16 GB → 3, 20 GB+ → 4 |
| **Ainult CPU-põhine süsteem** | `CPU_PARALLEL` | füüsilised tuumad − 1 (min 2) | `cpu_fallback` | GPU-d ei ole vaja, kasutab niidipooli |

{% hint style="info" %}
**Jetsoni ühtne mälu**: Jetsoni seadmed jagavad GPU ja CPU mälu. Jetson Orin NX 16 GB näitab ~15,3 GB VRAM-i, kuid see on sama füüsiline RAM, mida kasutavad operatsioonisüsteem ja CPU protsessid. Seetõttu vastavad 16 GB ja suurema mälumahuga Jetsonid `GPU_PARALLEL` nõuetele nagu 12 GB ja suurema mälumahuga lauaarvuti GPU, kuid neil on piiratud 2 töötajaga – GPU, tööprotsessid ja nende töötaja-põhised CUDA-kontekstid kasutavad kõik sama jagatud reservi.
{% endhint %}

### GPU eelarve VRAM-i järgi (eraldiseisvad GPU-d)

x86_64-põhistes hostides, millel on eraldiseisev NVIDIA GPU, määrab tuvastatud VRAM ka selle, kui palju kaardi töötlemisvõimsust võib kasutada ja kui suured võivad partiid olla:

| Tuvastatud VRAM | GPU eelarve ülempiir | Partii suuruse kordaja |
| --- | --- | --- |
| **8 GB+** | 90% | ×2,0 |
| **6–8 GB** | 85% | ×1,75 |
| **3,5–6 GB** | 80% | ×1,5 |
| **2–3,5 GB** | 75% | ×1,25 |
| **Alla 2 GB** | 70% | ×1,0 |

Eraldiseisvad GPU-d reserveerivad süsteemile vaid 0,5 GB, kuna nad ei jaga süsteemi RAM-i. Jetsoni profiilid reserveerivad palju rohkem ja piiravad vähem — vt [NVIDIA Jetsoni juhendit](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dünaamiline GPU-mälu eraldamine

Chloros kasutab [4-niitlist töötlemispipeliini](processing-pipeline.md):

* **Niit 1** (avastamine) — pildi laadimine, EXIF-andmete analüüs, sihtmärgi avastamine
* **Niit 2** (kalibreerimine) — peegelduskalibreerimise arvutamine
* **Lõng 3** (töötlemine) — GPU-põhine debayer, vinjeti korrigeerimine, indeksi arvutamine
* **Lõng 4** (eksport) — failide kirjutamine, metaandmete lisamine

Lõngad 1, 2 ja 4 tarbivad GPU-võimsust vähe; 3. niit on kõige ressursimahukam. Kui varasemad töötlusjärgu niidid lõpetavad töö, **jaotatakse nende GPU ressursid ümber ülejäänud aktiivsetele niitidele**, mistõttu saab 3. niit töötluse edenedes järjest rohkem mälu.

### Jaotamisetapid

| Etapp | Aktiivsed niidid | GPU mälu jaotus |
| --- | --- | --- |
| **Algus** | 1, 2, 3, 4 | Jaguneb kõigi niitide vahel, enamik sellest niidile 3 |
| **Alguse keskel** | 2, 3, 4 | Niidi 1 osa jaotatakse ümber |
| **Keskmine-lõpp** | 3, 4 | Niidid 1 ja 2 osad lähevad niididele 3 ja 4 |
| **Lõpp** | 3 või 4 | Viimane aktiivne niit saab oma maksimaalse eraldise |

Numbreid reguleerivad kaks reeglit:

* **Ainus** aktiivne niit saab oma profiili maksimaalse eraldise.
* Kui aktiivne on rohkem kui üks *raske* GPU-ülesanne, jagatakse iga raske ülesande baaseraldis nende vahel (mitte kunagi alla konfigureeritud miinimumväärtuse).

Tegelikult kasutatakse käitamisel **väiksemat** väärtust platvormiprofiili eraldise ja GPU mälimonitori reaalajas soovituse vahel, seega võidab koormatud kaart alati optimistliku profiili üle.***

## Tekstuuriteadlik töötlemine

Tekstuuriteadlik debayer (**ainult Chloros+** — `--debayer texture-aware`) käitab AI/ML-müra eemaldamise mudelit, mis vajab ühe koopia kohta ligikaudu 1,75 GB VRAM-i FP16-režiimis, seega kasutab see palju rohkem GPU-mälu kui standardmeetod:

* Süsteemid, millel on **alla 7 GB VRAM-i**, töötlevad tekstuuriteadlikku töötlemist**sünkroonses tsüklis, üks pilt korraga** — mitu mudeli koopiat ei mahu sinna ja töötajate rühm tekitaks vaid konkurentsi
* Süsteemid, millel on **7 GB või rohkem VRAM-i**, suudavad Texture Aware'i töödelda paralleelselt, kuigi töötajate arv on Standard-meetodiga võrreldes väiksem
* **Jetsonil** on Texture Aware alati seotud ühe töötajaga
* Kui **kasutatavat CUDA-GPU-d pole**, kasutab käivitus Standard-debayerit ja tagapõhi logib ühe rea `[DEBAYER] … needs a CUDA GPU …`, milles selgitatakse põhjust
* Madala võimsusega Jetsonitel (Nano, Orin Nano) rakendab CLI GPU sageduse piirangut, kui taotled Texture Aware'i kasutamist — vaata [NVIDIA Jetsoni juhendit](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Soojusjuhtimine (Jetson)

Jetson-seadmetel on soojuspiirangud, eriti suletud ruumides või õhus kasutamisel. Chloros jälgib Jetsoni sisseehitatud temperatuuriandureid ja skaleerib partiide suurust automaatselt:

| Temperatuur | Reaktsioon |
| --- | --- |
| **< 70 °C** | Normaalne töö — täiskiirus |
| **70 °C** (hoiatus) | Partii suurus väheneb järk-järgult (100% → 50% vahemikus 70 °C kuni 80 °C) |
| **80 °C** (kriitiline) | Agressiivne võimsuse piiramine (50% → 0% vahemikus 80 °C kuni 90 °C) |
| **90 °C** (seiskamine) | GPU töötlemine peatatakse täielikult |

Piisava jahutusega lauaarvutites käivitub soojuspiirang harva.

***

## Mälu koormuse haldamine

 Chloros jälgib töötlemise ajal pidevalt GPU mälu ja reageerib kolmel tasandil.

**Partii suuruse määramine.** Partii algab 8 pildist, mis on korrutatud ülaltoodud tabelites esitatud platvormi kordajaga. Seejärel kontrollib Chloros vaba VRAM-i, reserveerib sellest 20% PyTorchi enda halduskoormuseks ja eeldab umbes 100 MB GPU mälu iga 12 MP pildi kohta – partii suurus on väiksem neist kahest: mälust tulenev piirang või platvormi baasväärtus. See ei lange kunagi alla 1.

**Ennetav vähendamine.**Kui**VRAM-i kasutamine ületab 85%**, vähendatakse partiide suurust enne, kui midagi ebaõnnestub.**Eraldise kärpimine iga niidi kohta.** Reaalajas kasutuse suurenemisel vähendatakse iga niidi GPU-eelarvet: ×0,75 üle 80% kasutuse juures, ×0,5 üle 90% juures. Seirevahemikud on 70% (konservatiivne), 85% (normaalne töötamispiir) ja 95% (OOM-risk).**OOM-tagasitõmbumine ja taastumine.** Kui mälu ammendumise sündmus ikkagi toimub:

* vähendatakse partii suurust **poole võrra** ning iga järgneva mälupuuduse korral veel kord poole võrra — iga järgnev edukas partii vähendab seda karistust ühe astme võrra
* aktiivsete niitide eraldised vähendatakse 70%ni nende praegusest väärtusest ning eraldaja lülitub üle konservatiivsele strateegiale, leevendades piiranguid taas pärast edukate eraldiste seeriat
* tugeva koormuse korral lülitub töötlemisahel tagasi `fused_gpu`-ilt `tiled_gpu`-ile ning viimase abinõuna `cpu_fallback`-ile

**Host-RAM (Jetson).** Enne töötlemist hindab CLI i pildi arvu ja debayer-režiimi põhjal hosti mälu tippmahtu ning hoiatab, kui RAM-mälu ja failipõhine vahetusmälu tõenäoliselt ei piisa, kuvades täpsed käsud vahetusmälu lisamiseks — vt [NVIDIA Jetsoni juhendit](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Arvutusvõimsuse kohandamise jälgimine

### Süsteemidiagnostika

`chloros-cli selftest` on kiireim viis kontrollida, mida arvutuskiht näeb:

```bash
chloros-cli selftest
```

Selle 7 kontrolli hõlmavad versiooni, pordi kättesaadavust, tagapõhja käivitamist, `/api/test`, süsteemiinfot, müra vähendaja mudeli olemasolu ning CUDA ja müra vähendaja valmisolekut. Kontroll 5 kuvab otse riistvararida:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

7. kontroll väljastab `CUDA: <bool>, Denoiser: <bool>` — mõlemad peavad olema tõesed, et Texture Aware'i saaks üldse kasutada.

### Tagapõhja logid

Strateegia ja töötajate arv valitakse backendis iga käivituse alguses — neid ei teavita ükski CLI-bänner. Kui midagi käitub ootamatult (GPU-tee tagasipöördumine, OOM, müra vähendaja, mis ei lae), ilmub see selle seansi backend-logisse:

| Platvorm | Logi asukoht |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (üks fail iga käivituse kohta) |
| **Linux, CLI kaudu käivitatud backend** | samuti `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Reaalajas edenemine

Käivituse ajal näitab CLI reaalajas iga niidi edenemist (avastamine, analüüsimine, töötlemine, eksportimine), mida edastatakse Server-Sent Eventsi kaudu — see annab praktilise ülevaate sellest, kas niit 3 on pudelikaelaks. Vaata [töötlemispipeliini](processing-pipeline.md).

***

## Järgmised sammud

* [Töötlemispipeline](processing-pipeline.md) — 4-niitilise töötlemispipeline'i arhitektuuri mõistmine
* [NVIDIA Jetsoni juhend](../linux/nvidia-jetson-guide.md) — Jetsonile spetsiifiline kasutuselevõtt ja optimeerimine
* [CLI: käsurea](../CLI.md) — CLI juhend
* [CLI viide](../reference/cli-reference.md) — Täielik käskude loetelu versioonile 1.3.0
