# Pildistamise seaded ja režiimid

Pildistamine vahekaardil „Kaamerad“ toimub ühe punase nupu **Capture All**ja ühe paneeli**Capture Settings** abil, mis määrab, milliseid tulemusi see nupp annab: millised kaamerad osalevad, milliseid eksporditüüpe iga kaamera salvestab ning kas katik vallandub üks kord, pidevalt või kindla intervalliga. Käesoleval lehel on kirjeldatud kogu protsess – seadistamine, pildistamine ise, failide salvestamise koht kettal ning kuidas neid hiljem kalibreeritud tulemusteks ümber töödelda. Kaamera- ja kaamerarivi juhtimisfunktsioonid asuvad lehel [Kaamera seaded](camera-settings.md).

{% hint style="info" %}
**Pildistamiseks peab projekt olema avatud.** Nupud „Pildista kõik“ ja „Pildistamise seaded“ (hammasratas) on blokeeritud, kuni projekt on avatud („Loo või ava projekt, et salvestada pildistatud materjal“). Iga pildistamine salvestatakse projekti kausta alla `captures/`.
{% endhint %}

## Pildistamise seaded

Avage see külgriba kaamerate loendis asuva **„Kõik pildistada“ kõrval oleva hammasratta**abil või mis tahes kaamerapõhiste seadeakna allosas asuva**„Ava pildistamise seaded…“** nupu abil. Pealkirjas on kirjas „Salvestusseaded“ koos ← tagasi-nupuga.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Siin tehtud valikud – kaasatud kaamerad, tüübipõhised valikukastid ja salvestusrežiim – salvestatakse **projekti kaupa** ja taastatakse projekti uuesti avamisel.

### Salvestusrežiimid

Paneeli ülaosas on kolm režiiminuppu:

| Režiim | Mida see teeb | Alaseaded (vaikimisi) |
| --- | --- | --- |
| **Üksik** *(vaikimisi)* | Üks salvestus kõigilt valitud kaameratelt. | — |
| **Järjepidev**| Pildistamine järjest, kuni peatamistingimus täitub. | Peatamine**pildistamiste arvu** järgi (vaikimisi 1) *või* **pildistamise kestuse** järgi (vaikimisi 10 s; ühikud: sekundid / minutid / tunnid / päevad). |
| **Intervall**(ajavahemik) | Pildistamine ajastuse järgi. |**Pildistamised / intervall**(vaikimisi 1) ·**Iga**N ühiku järel (vaikimisi 5 s) ·**Kestus** N ühikut (vaikimisi 1 m). |

Pidevas või intervallrežiimis muutub nupp „Pildista kõik“ töötamise ajal nupuks **Peata (N)**, loendades tehtud pilte nende salvestamise järgi.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Kaamerate ja eksporditüüpide valimine

Paneeli abitekst võtab selle kokku: vali, milliseid kaameraid ja eksporditüüpe nupp „Capture All“ toodab – vaikimisi on kõik sisse lülitatud ja valikud salvestatakse koos selle projektiga.

* Nupud **Valige kõik / Ära vali midagi** lülitavad korraga sisse või välja iga kaamera valikukasti.
* **Hulgekspordi tüübi lülitid**(kaks rida nuppe):**Kõik toored / Kõik debayered / Kõik eelvaated / Kõik kiirgus / Kõik peegeldus / Kõik indeks**. Iga lüliti on kolmeolekuline: roheline ✓ = sisse lülitatud kõikidel kaameratel, mis seda toetavad; merevaigukollane – = sisse lülitatud mõnel kaameral; hall = puudub. Lüliti on keelatud, kui ükski ühendatud kaamera seda tüüpi ei toeta. Kõik lülitid muutuvad halliks, kui funktsioon „Fastest Capture“ on sisse lülitatud.
* **Kaamerapõhised read**: valikukast „Kaasa arvestada” ning laienev (▸/▾) nimekiri selle kaamera jaoks sobivatest eksporditüüpidest, millel on eraldi valikukastid. Rida näitab aktiveeritud arvutust, näiteks „4/6”.

### Eksporditüübid ja millised kaamerad neid toetavad

Eksporditüüpe on kuus: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Iga kaamera real kuvatakse ainult asjakohased tüübid:

| Eksporditüüp | Sisu | RGB (FRGB) | Bayeri multispektraalne (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayeri mosaiik (mono: üks kanal) otse sensorist | ✓ | ✓ | ✓ |
| **Debayered** | Lineaarne demosaik (mono: 1-kanaliline halltoon) | ✓ | ✓ | ✓ |
| **Eelvaade** | Täielik kuvamisahel (valgebalanss + gamma vastavalt kaamera profiilile; multispektraalne: valevärvide venitus) | ✓ | ✓ | ✓ |
| **Kiirgus** | float32 W/m²/sr/nm täieliku radiomeetrilise ahela kaudu | — (ei pakuta) | ✓ | ✓ |
| **Peegeldusvõime** | uint16 ρ (32768 = 1,0) | — (ei pakuta) | ✓ — kuvatakse ainult siis, kui kaameral on DAQ-valgusandur (oma või massiivist pärit) | sama mis multispektraalne |
| **Indeks** | Taimestikuindeks (LUT) render | — | ✓ — nõuab kaameral aktiveeritud, mittetühja indeksavaldist ning seda ei pakuta kombineeritud maatriksi liikmetele (maatriksil on üks ühine indeks) | — (indeks vajab ≥2 laineala; vt [Monokaamerad ja taimestikuindeksid](mono-indices.md)) |

Kiirgust ja peegeldust ei pakuta kunagi RGB kaamerate puhul — Bayeri-põhine kiirgus ei ole laiaribalise fotomeetrilise anduri puhul mõttekas.

### Kiireim salvestamine

Lüliti **⚡ Kiireim salvestamine — ainult toorandmed**(sisse lülitatuna oranž) asendab kõik ekspordivalikud valikuga**ainult toorandmed** — lisaks tasuta kombineeritud indeksiga komposiit massiivide jaoks — nii et kaader jõuab sihtkohta võimalikult kiiresti: kiirguse/peegeldusvõime/kuvamise arvutused jäetakse pildistamise ajal täielikult vahele.

{% hint style="info" %}
**`.daq` salvestatakse ikkagi.** Kui on määratud valgussensor, kirjutab „Kiireim salvestamine“ endiselt DAQ-i allapoole suunatud näidu toorkaadrite kõrvale — nii saab kiirgust, peegeldust ja indeksit käsitlevaid tooteid hiljem uuesti töötlemise teel luua (vt [Salvestuste uuesti töötlemine](#re-processing-captures-into-calibrated-products)). Fastest Capture ei mõjuta ka teie valikute seadeid: lülitage see välja ja need taastuvad.
{% endhint %}

### Massiivi-põhised juhtimisfunktsioonid

Igal ühendatud massiivil on paneelis oma rühmakort:

* **Valikukast „Include“** (kolmeolekuga liikmete vahel) ja massiivi nimi koos kuvamisrežiimiga: „(combined | separate)“.
* **Joondatud**valikukast (vaikimisi**sees**): joondab liikmete ekspordid massiivi joondamisprofiili järgi, nii et ekspordid on kaamerate vahel pikselitäpselt joondatud. Toores andmestik jääb moonutamata, kuid sisaldab teisendust oma metaandmetes. (Profiil ise arvutatakse välja [massiivi seadeaknas](camera-settings.md#alignment-co-registration-combined-only).)
* Kaameraelementide read on kaardi sees pesastatud.

Massiivi kaardil on ka kaks salvestajat. Mõelge neile kui **jälgimisele vs analüüsile**:

| Salvestaja | Tase | Mida salvestab |
| --- | --- | --- |
| **● Salvestab indeksvideot / ■ Peatab salvestamise** *(ainult kombineeritud massiivid)* | **Jälgimine** | Reaalajas kombineeritud indeks-komposiitvideona kiirusega 10 kaadrit sekundis — 8-bitine, eelvaate resolutsioon, sisseehitatud LUT. Vajab avatud projekti ja reaalajas voogedastust. Näitab kaadreid ja salvestamise kestust. |
| **⦿ Salvestab toor-pildiseeria / ■ Toores seeria salvestamise peatamine** *(mis tahes kaameramassiiv)* | **Analüüs**| Toored Bayer-kaadrid reaalajas salvestamise kiirusel (töötlemata) koos kaadri-põhise manifesti ja `.daq` näitudega, salvestatuna failina `captures/bursts/`. Pärast sarivõtet ilmub nupp**„Build video“**: see töötleb sarivõtte offline-režiimis ümber kalibreeritud videoks — kombineeritud indeks ja/või kaamera kohta kiirgus / peegeldus / indeks — pluss valikulised TIFF-failid. Kombineeritud indeksiga koostamine algab automaatselt, kui peatate sarivõtte. |##

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

„Capture All“ töövoog

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

Vajuta külgribal asuvas kaamerate nimekirjas nuppu **„Capture All“**:

1. Iga kaasatud, nähtav ja pausile panemata kaamera salvestab valitud eksporditüüpidega. **Kaamerarühmad käivituvad ühe sünkroniseeritud päästikuna** (ühtne sünkroniseeritud rühm kõigi liikmete vahel — vt [Mitme kaameraga rühmad](arrays.md)); eraldiseisvad kaamerad salvestavad individuaalselt.
2. Peidetud (silmaikooniga) või pausile pandud kaamerad jäetakse vahele. Kaamerarühm blokeeritakse täielikult alles siis, kui *kõik* selle liikmed on peidetud või pausile pandud.
3. Kui valgussensor on määratud, salvestatakse vastav DAQ-i allapoole suunatud valguse mõõtmistulemus `.daq`-failina koos piltidega — isegi ainult toorandmetena salvestatud piltide puhul —, et radiomeetrilisi tooteid saaks alati hiljem tuletada.
4. Nupp näitab reaalajas edenemist – „Salvestamine… valmis/kokku“ – ning pidevas/intervallrežiimis muutub see **Stop (N)**-iks. Igal salvestuselemendil on 300-sekundiline ooteaeg.
5. Kui läbimine lõpeb, kuvab tulemuste teade **„Salvestatud N faili”**või**„Salvestatud N, F ebaõnnestus”**, lisaks „(S peidetud/peatatud/vahele jäetud)”, kui kaamerad jäeti vahele.

## Kuhu salvestused paigutatakse

Salvestused salvestatakse avatud projekti alla kataloogi `<project>/captures/`. Iga eksporditüüp paigutatakse oma **eraldi alamkataloogi**, seega mitmetasandilises salvestuses ei segune tüübid kunagi:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` on salvestuse ajamärge ja `<serial>` kaamera seerianumber. Eraldi salvestused nimetatakse `capture_<ts>_SN<serial>_<level>`; ühe sünkroniseeritud päästiku abil tehtud masssalvestused nimetatakse `sync_<ts>_SN<serial>_<level>` ja **neil on ühine ajamärge kõigi rühma kaamerate jaoks** (taseme sufiks jäetakse ära, kui kaamera salvestab ainult ühe taseme).
* **Üks erand, mida tuleb teada:** kuvamistasand salvestatakse kausta nimega `preview/`, samas kui failide nimedes säilitatakse `_display` – kausta nimi ja lõpp erinevad ainult selle tasandi puhul.
* Tundmatud tasemed salvestatakse oma nime kandvasse kausta; kui alamkausta ei ole võimalik luua, salvestatakse fail pildistuste juurkausta, selle asemel et see kaoks.
* Pildistatud TIFF-failid on vaikimisi kaotusvabalt pakitud (DEFLATE) ja sisaldavad kõiki kalibreerimis- ja töötlemismetandmeid **faili XMP-s** – pildid on isekirjeldavad ega vaja muid lisafaile peale `.daq`-faili.

See on sama struktuur, mille `chloros-cli lattice capture` / `array-capture` kirjutavad oma `-o` kataloogi — dokumenteeritud [CLI viites § Kuidas näeb välja salvestuste kaust](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Salvestuste ümbertöötlemine kalibreeritud toodeteks

Salvestatud töötlemata kaadrid ja salvestatud `.daq` on kõik, mida töötlemisahel vajab — seetõttu on „Fastest Capture“ reaalseks tööks ohutu.

* **GUI**: lisage salvestuste kaust projekti ([Failide lisamine projekti](../processing-images-gui/adding-files-to-a-project.md)) ja töötlege nagu tavaliselt.
* **CLI**: suuna `process`**pildistatud kaadrite juurkataloogile**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` impordib tavaliselt ainult nimetatud kausta, kuid kui selles kaustas pole pilte ja seal on alamkaustad, liigub programm automaatselt allapoole — nii et taseme alamkaustad ja juurkaustas `.daq` olevad failid võetakse ühekorraga sisse. Iga salvestus imporditakse **ühe pildina**, mille juurde on lisatud teised tasemed vaatamisrežiimidena, mitte ühe pildina iga taseme kohta.

Taseme alamkausta otsene nimetamine (nt `…/captures/raw/`) toimib samuti, kuid juurkaustad `.daq` jäävad välja – kopeerige need kõrvale, kui tuletate radiomeetrilist toodet uuesti `raw/`-ist, vastasel juhul ei ole ajamärgise vastavusel millegi vastu võrrelda.

{% hint style="warning" %}
**Töötlemine algab alati kaustast `raw`.**Iga salvestuse puhul on töötlemisahela allikaks töötlemata kaader; `debayered`, `radiance`, `reflectance` ja `preview` on vaatamisrežiimid, kuid neid ei suunata kunagi tagasi töötlusprotsessi — tuletatud toote uuesti töötlemine rakendaks uuesti vinjeti, värvi ja sära arvutusi, mis on juba piksleidesse sisse küpsetatud, seega loobub Chloros kahekordsest töötlemisest. Renderdusi `index/` ja `composite/` ei töödeldagi üldse (need on väljundid, mitte salvestused). Salvestatud kaadrite kaust, milles**puuduvad** RAW-failid, kuvatakse normaalselt, kuid `process` jätab selle vahele ja teatab sellest; `--input-level {raw,debayered,processed}` on tahtlik pääsetee, mis sunnib sissepääsupunkti kasutama. Täpsed vahelejätmisteated leiate [CLI-viitest](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Veel kaks käitumist, mida tasub teada skriptide abil ümbertöötlemisel:

* `chloros-cli process` käivitus, mis taotles väljundeid, kuid **ei kirjutanud ühtegi pildiväljundit, ebaõnnestub ilmselgelt ja lõpeb nullist erineva tulemusega** — vaikset tühja käivitust ei saa kunagi. Edukad käivitused teatavad oma väljundite arvu. (Tahtlikult ainult metandmetega käivitus loetakse ikkagi edukaks.)
* Uuesti imporditud töödeldud ekspordid ei võta kunagi pildi toorandmete pesa — algne toorandmestik jääb alati töötlemisprotsessi allikaks.

## CLI ekvivalendid

Kõiki sellel lehel kirjeldatud toiminguid saab teostada graafilise kasutajaliidese kasutamiseta. Graafilise kasutajaliidese pildistamisrežiimid vastavad otseselt `chloros-cli lattice array-capture`-ile:

| GUI | CLI |
| --- | --- |
| Ühekordne | `chloros-cli lattice array-capture` |
| Pidev | `array-capture --continuous [--count N] [--duration S]` |
| Intervall | `array-capture --interval S [--duration S]` |
| Kiireim salvestamine | `array-capture --fastest` |
| Joondatud valikukast | `--aligned / --no-aligned` |
| Eksporditüübi valikukastid | `--processing LEVEL` või `--levels L1,L2,…` (vaikimisi `all`) |
| Salvestada indeksvideo | `chloros-cli lattice array-record` |
| Salvestada töötlemata pildiseeria / Luua video | `chloros-cli lattice array-burst` / `array-build-video` |

Täielikud liputabelid, nutika automaatse ekspositsiooni (smart-AE) püsiv pildistamise valik (`--smart`) ja püsiva kiirusega mudel asuvad jaotises [CLI Viide § Pildistamisrežiimid, salvestid ja offline-töötlemine](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
