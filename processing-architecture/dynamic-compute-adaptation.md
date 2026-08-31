# Dünaamiline arvutusvõimsuse kohandamine

Chloros 1.2.0 kasutab riistvara tuvastamist ja töötlusstrateegia automaatset valikut. Töötlusmootor kohandub teie riistvaraga – alates Jetson Orin Nanost kuni mitme GPU-ga tööjaamani – ilma käsitsi konfigureerimiseta.

***

## Kuidas see töötab

Kui Chloros käivitub, koostab see teie süsteemi profiili:

1. **Tuvastab operatsioonisüsteemi** — Windows või Linux
2. **Määrab kindlaks CPU tuumad ja RAM-mälu kogumahu**

3.**Tuvastab GPU olemasolu** — NVIDIA CUDA toetust, VRAM-mälu, mudeli
4. **Määrab kindlaks Jetsoni mudeli** (kui see on olemas) — `/proc/device-tree/model`i kaudu
5. **Kontrollib temperatuuriandureid** (Jetson) — temperatuurist sõltuva töötlemise jaoks
6. **Valib arvutusstrateegia** — kõigi tuvastatud riistvarakomponentide põhjal
7. **Konfigureerib töötajate arvu, töötlemisjärjekorra tüübi ja mälu jaotuse** automaatselt

Tuvastatud profiil salvestatakse sessiooni jaoks mällu ja kettale, et hilisemad käivitamised algaksid kiiremini:

| Platvorm | Salvestatud profiil |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (järgib `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Kustutage see fail, et sundida süsteemi uuesti tuvastama — see on kasulik pärast GPU või täiendava RAM-i lisamist. Chloros tuvastab automaatselt uuesti ka juhul, kui vahemälu on kirjutatud ühilduvuse puudumisega vanema versiooni poolt.

***

## Arvutusstrateegiad

Chloros valib teie riistvara põhjal ühe kolmest arvutusstrateegiast:

| Strateegia | Valitakse, kui | Töötajad | Täitja | Torujuht |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA-graafikakaart, mis näitab**12 GB+ VRAM**(Jetsoni ühtses mälus, nõuab ka vähemalt 12 GB jagatud RAM-i) | `min(4, VRAM ÷ 4GB)`, vähemalt 2 —**Jetsonil piiratud 2-ga** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA GPU**2–12 GB VRAM-iga**| 3 (I/O-kattuvus; GPU-juurdepääs on semaforiga järjestatud).**1 (järjekorras) Jetsonitel, mille RAM-mälu on alla 12 GB** | `ProcessPoolExecutor` (käivitamine); järjekorras protsessi sees väikese RAM-iga Jetsonil | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Ilma CUDA GPU-ta või alla 2 GB VRAM-iga | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

`GPU_PARALLEL` töötaja valemi näited: 12 GB VRAM → 3 töötajat, 16 GB+ → 4 töötajat, mis tahes Jetson → 2 töötajat.

Paralleelsus on rakendatud Python standardse `concurrent.futures` abil: GPU-strateegiad kasutavad `ProcessPoolExecutor`-i koos **spawn**-käivitusmeetodiga (iga töötaja on eraldi protsess oma CUDA-kontekstiga — `fork` kopeeriks juba initsialiseeritud CUDA-seisundi ja rikuks alamprotsesse), ning CPU-strateegia kasutab `ThreadPoolExecutor`-i. Chloros ei kasuta ühtegi kolmanda osapoole hajutatud raamistikku (nagu Ray).

### Töötlemisliinide tüübid

* **`fused_gpu`** — Täielik GPU-töötlemisliin. Debayer, korrektsioon ja indekseerimine toimuvad GPU-l ühe ühendatud läbimisega. Kõrgeim läbilaskevõime, vajab kõige rohkem VRAM-i.
* **`tiled_gpu`** — Mälu säästev GPU-töötlusrada. Töötleb pilte plaadikestena, et need mahuksid piiratud GPU-mälu sisse. Läbilaskevõime on madalam, kuid töötab mälupiirangutega seadmetel.
* **`cpu_fallback`** — Ainult CPU-põhine töötlemine, kasutades mitmeprotsessilist paralleelsust. Kasutatakse juhul, kui NVIDIA GPU ei ole saadaval, ning viimase abinõuna, kui mõlemad GPU-töötlusviisid ebaõnnestuvad.

Käitamisaja varuvariandi ahel on alati `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Strateegia käsitsi ümberkirjutamine

Määrake keskkonnamuutuja `CHLOROS_STRATEGY`, et sundida kasutama kindlat strateegiat — see on ekspertide varuvariant juhuks, kui automaatne tuvastamine valib teie olukorrale sobimatu strateegia (näiteks GPU vabana hoidmine muude ülesannete jaoks):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Muutuja sobitamine toimub suurtähti arvestamata; kõike, mis ei kuulu kolme nimetatud nime hulka, ignoreeritakse ja automaatne tuvastamine jätkub tavapäraselt. Ülekirjutamise korral valib Chloros teie jaoks ikkagi töötajate arvu:

| Ülekirjutamine | Kasutatud töötajate arv |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Soovitan seda seadistada pigem käsu kaupa kui püsivalt, et tavapärased käivitused saaksid end automaatselt kohandada.

***

## Platvormispetsiifiline käitumine

| Platvorm | Strateegia | Töötajad | Töötlusliin | Märkused |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (järjekorras) | Mälu säästev režiim, üks pilt korraga |
| **Jetson Orin NX 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (järjekorras) | Alla 12 GB jagatud RAM-mälu sunnib töötlema järjekorras |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (paralleelne) | Soovitatav servaseade — Jetsonil on piiratud 2 töötajaga |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (paralleelne) | Maksimaalne servaseadme jõudlus (samuti Jetsoniga piiratud 2 töötajaga) |
| **Lauaarvuti 8 GB GPU-ga** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 töötaja I/O kattub, samal ajal kui semafor järjestab GPU-le juurdepääsu |
| **Töölauaarvuti 12 GB+ GPU-ga** | `GPU_PARALLEL` | 3–4 | `fused_gpu` (paralleelselt) | Optimaalne lauaarvuti jõudlus: 12 GB → 3 töötajat, 16 GB+ → 4 |
| **Ainult CPU-põhine süsteem** | `CPU_PARALLEL` | füüsilised tuumad − 1 (min 2) | `cpu_fallback` | GPU-d ei nõuta, kasutab niidipooli |

{% hint style="info" %}
**Jetsoni ühtne mälu**: Jetsoni seadmed jagavad GPU ja CPU mälu. Jetson Orin NX 16 GB näitab ~15,3 GB VRAM-i, kuid tegemist on sama füüsilise RAM-iga, mida kasutavad operatsioonisüsteem ja CPU-protsessid. Seetõttu vastavad 16 GB+ Jetsonid `GPU_PARALLEL`-i nõuetele nagu 12 GB+ lauaarvuti GPU, kuid nende töötajate arv on piiratud kahega – GPU, tööprotsessid ja nende töötaja-põhised CUDA-kontekstid kasutavad kõik sama jagatud mälupooli.
{% endhint %}

### GPU eelarve VRAM-i järgi (eraldiseisvad GPU-d)

x86_64-põhistes hostides, millel on eraldiseisev NVIDIA graafikakaart, määrab tuvastatud VRAM ka selle, kui palju kaardi töötlemisvõimsust võib kasutada ja kui suured võivad partiid olla:

| Tuvastatud VRAM | GPU eelarve ülempiir | Partiisuuruse kordaja |
| --- | --- | --- |
| **8 GB+** | 90% | ×2,0 |
| **6–8 GB** | 85% | ×1,75 |
| **3,5–6 GB** | 80% | ×1,5 |
| **2–3,5 GB** | 75% | ×1,25 |
| **Alla 2 GB** | 70% | ×1,0 |

Eraldi graafikakaardid reserveerivad süsteemile vaid 0,5 GB, kuna nad ei jaga süsteemi RAM-i. Jetsoni profiilid reserveerivad palju rohkem ja piiravad vähem — vaata [NVIDIA Jetsoni juhendit](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dünaamiline GPU-mälu eraldamine

Chloros kasutab [4-niitlist töötlemispipeliini](processing-pipeline.md):

* **Niit 1** (avastamine) — pildi laadimine, EXIF-andmete analüüs, sihtmärgi avastamine
* **Niit 2** (kalibreerimine) — peegelduskalibreerimise arvutamine
* **Niit 3** (töötlemine) — GPU-põhine debayer, vinjeti korrigeerimine, indeksi arvutamine
* **Niit 4** (eksport) — faili kirjutamine, metaandmete lisamine

Protsessid 1, 2 ja 4 tarbivad GPU-võimsust vähe; protsess 3 on kõige ressursimahukam. Kui varasemad protsessid lõpetavad töö, **jaotatakse nende GPU-võimsus ümber ülejäänud aktiivsetele protsessidele**, mistõttu saab protsess 3 töötamise käigus järjest rohkem mälu.

### Eraldamise etapid

| Etapp | Aktiivsed niidid | GPU-mälu jaotus |
| --- | --- | --- |
| **Algus** | 1, 2, 3, 4 | Jaguneb kõigi niidide vahel, enamik sellest läheb niidile 3 |
| **Varajane keskfaas** | 2, 3, 4 | Niidi 1 osa jaotatakse ümber |
| **Hiline keskfaas** | 3, 4 | Niidide 1 ja 2 osad lähevad niididele 3 ja 4 |
| **Lõpp** | 3 või 4 | Viimane aktiivne niit saab oma profiili maksimaalse eraldise |

Numbreid reguleerivad kaks reeglit:

* Niit, mis on **ainus** aktiivne, saab oma profiili maksimaalse eraldise.
* Kui aktiivne on rohkem kui üks *raske* GPU-ülesanne, jagatakse iga raske ülesande baaskvoot nende vahel (mitte kunagi alla konfigureeritud miinimumväärtuse).

Tegelikult kasutatakse käitamisel **väiksemat** väärtust platvormiprofiili eraldise ja GPU mälimonitori reaalajas soovituse vahel, seega võidab koormatud kaart alati optimistliku profiili üle.***

## Tekstuuriteadlik töötlemine

Tekstuuriteadlik debayer (**ainult Chloros+** — `--debayer texture-aware`) kasutab AI/ML-müra vähendamise mudelit, mis vajab ühe koopia kohta umbes 1,75 GB VRAM-i FP16-režiimis, seega kasutab see palju rohkem GPU-mälu kui standardmeetod:

* Süsteemid, millel on **alla 7 GB VRAM-i**, töötlevad tekstuuritundlikku töötlust**sünkroonses tsüklis, üks pilt korraga** — mitu mudeli koopiat ei mahu sinna ära ja töötajate rühm tekitaks vaid konkurentsi
* Süsteemid, millel on **7 GB või rohkem VRAM-i**, suudavad Texture Aware&#x27;i töödelda paralleelselt, kuigi töötajate arv on Standard-meetodiga võrreldes väiksem
* **Jetsonil** on Texture Aware alati seotud ühe töötajaga ning madala võimsusega mudelitel (Nano, Orin Nano) rakendub automaatselt ka GPU sageduse piirang — vt [NVIDIA Jetsoni juhendit](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Soojusjuhtimine (Jetson)

Jetson-seadmetel on soojuspiirangud, eriti suletud ruumides või õhus kasutamisel. Chloros jälgib Jetsoni sisseehitatud temperatuuriandureid ja skaleerib partiide suurust automaatselt:

| Temperatuur | Reaktsioon |
| --- | --- |
| **&lt; 70 °C** | Normaalne töö – täiskiirus |
| **70 °C** (hoiatus) | Partii suurus väheneb järk-järgult (100% → 50% vahemikus 70 °C kuni 80 °C) |
| **80 °C** (kriitiline) | Agressiivne jõudluse piiramine (50% → 0% vahemikus 80 °C kuni 90 °C) |
| **90 °C** (väljalülitamine) | GPU töötlemine peatatakse täielikult |

Piisava jahutusega lauaarvutites käivitub termiline piiramine harva.

***

## Mälukoormuse käsitlemine

Chloros jälgib töötlemise ajal pidevalt GPU mälu ja reageerib kolmel tasandil.

**Partii suurus.** Partii algab 8 pildist, millele korrutatakse eespool toodud tabelites esitatud platvormi kordaja. Chloros kontrollib seejärel vaba VRAM-i, reserveerib sellest 20% PyTorchi enda halduskoormuseks ning eeldab umbes 100 MB GPU mälu iga 12 MP pildi kohta — partii suurus on väiksem neist kahest: mälust tulenev piirang või platvormi baasväärtus. See ei lange kunagi alla 1.**Ennetav vähendamine.**Kui**VRAM-i kasutamine ületab 85%**, vähendatakse partiide suurust enne, kui midagi ebaõnnestub.**Eraldise kärpimine iga niidi kohta.** Reaalajas kasutuse suurenemisel vähendatakse iga niidi GPU-eelarvet: ×0,75 üle 80% kasutuse juures, ×0,5 üle 90% kasutuse juures. Seirevahemikud on 70% (konservatiivne), 85% (normaalne töötamispiir) ja 95% (mälupuuduse risk).**Mälupuuduse tagasitõmbumine ja taastumine.** Kui mälupuudus ikkagi tekib:

* vähendatakse partii suurust **poole võrra** ning iga järgneva mälupuuduse korral veel kord poole võrra — iga järgnev edukas partii vähendab seda karistust ühe astme võrra
* aktiivsete niitide eraldised vähendatakse 70%ni nende praegusest väärtusest ja eraldaja lülitub üle konservatiivsele strateegiale, leevendades piiranguid taas pärast edukate eraldiste seeriat
* tugeva koormuse korral lülitub töötlemisahel `fused_gpu`-ilt tagasi `tiled_gpu`-ile ning viimase abinõuna `cpu_fallback`-ile

**Host-RAM (Jetson).** Enne töötlemist hindab CLI piltide arvu ja debayer-režiimi põhjal host-mälu tippkoormust ning hoiatab, kui RAM ja failipõhine vahetusmälu tõenäoliselt ei piisa, kuvades täpsed käsud vahetusmälu lisamiseks — vt [NVIDIA Jetsoni juhend](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Arvutusvõimsuse kohandamise jälgimine

### Süsteemidiagnostika

`chloros-cli selftest` on kiireim viis kontrollida, mida arvutuskihis näeb:

```bash
chloros-cli selftest
```

Selle 7 kontrolli hõlmavad versiooni, pordi kättesaadavust, tagapõhja käivitamist, `/api/test`, süsteemiinfot, müra vähendaja mudeli olemasolu ning CUDA ja müra vähendaja valmisolekut. Kontroll 5 kuvab otse riistvarareaga:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Kontroll 7 kuvab `CUDA: <bool>, Denoiser: <bool>` — mõlemad peavad olema tõesed, et Texture Aware&#x27;i saaks üldse kasutada.

### Tagapõhja logid

Strateegia ja töötajate arv valitakse backendis iga käivituse alguses – neid ei teata mingi CLI-bänneriga. Kui midagi käitub ootamatult (GPU-tee tagasipöördumine, OOM, müra vähendaja, mis ei lae), ilmub see selle seansi backend-logis:

| Platvorm | Logi asukoht |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (üks fail iga käivituse kohta) |
| **Linux, CLI-started backend** | samuti `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Reaalajas edenemine

Käivituse ajal näitab CLI reaalajas iga niidi edenemist (avastamine, analüüsimine, töötlemine, eksportimine), mida edastatakse Server-Sent Eventsi kaudu — see annab praktilise ülevaate sellest, kas niit 3 on pudelikaelaks. Vaata [töötlemispipeliini](processing-pipeline.md).

***

## Järgmised sammud

* [Töötlemispipeliin](processing-pipeline.md) — 4-niitilise pipeliini arhitektuuri mõistmine
* [NVIDIA Jetsoni juhend](../linux/nvidia-jetson-guide.md) — Jetsonile spetsiifiline kasutuselevõtt ja optimeerimine
* [CLI : käsurea](../CLI.md) — CLI juhend
* [CLI viide](../reference/cli-reference.md) — Täielik käskude loetelu versioonile 1.2.0
