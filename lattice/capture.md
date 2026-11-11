# Pildistamise seaded ja režiimid

Pildistamine vahekaardil „Kaamerad” toimub ühe punase nupu **Capture All**ja ühe paneeli**Capture Settings** abil, mis määrab, milliseid tulemusi see nupp annab: millised kaamerad osalevad, milliseid eksporditüüpe iga kaamera salvestab ning kas katik vallandub üks kord, pidevalt või kindla intervalliga. Käesoleval lehel on kirjeldatud kogu protsess – seadistamine, pildistamine ise, failide salvestamise koht kettal ning kuidas neid hiljem kalibreeritud toodeteks ümber töödelda. Kaamera- ja massiivi juhtimisfunktsioonid asuvad lehel [Kaamera seaded](camera-settings.md).

{% hint style="info" %}
**Salvestamiseks peab projekt olema avatud.** Nupud „Capture All“ ja „Capture Settings“ on keelatud, kuni projekt on avatud („Loo või ava projekt, et salvestada pildistamised“). Iga pildistamine salvestatakse projekti kausta alla `captures/`.
{% endhint %}

## Pildistamise seaded

Avage see külgriba kaamerate loendis asuva **„Kõik pildistada“ kõrval oleva hammasratta**abil või mis tahes kaamera seadeakna allosas asuva**„Ava pildistamise seaded…“** nupu abil. Pealkirjas on kirjas „Pildistamise seaded“ koos ← tagasi-nupuga.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Siin tehtud valikud – kaasatud kaamerad, tüübipõhised valikukastid ja salvestusrežiim – salvestatakse **projekti kaupa** ja taastatakse projekti uuesti avamisel.

### Salvestusrežiimid

Paneeli ülaosas on kolm režiiminuppu:

| Režiim | Mida see teeb | Alaseaded (vaikimisi) |
| --- | --- | --- |
| **Ühekordne** *(vaikimisi)* | Üks pildistamine kõigi valitud kaameratega. | — |
| **Järjepidev**| Pildistamine järjest, kuni peatamistingimus täitub. | Peatamine**pildistamiste arvu** järgi (vaikimisi 1) *või* **pildistamise kestuse** järgi (vaikimisi 10 s; ühikud: sekundid / minutid / tunnid / päevad). |
| **Intervall**(ajavahemik) | Pildistamissarjad ajastatud intervallidega. |**Pildistamised / intervall**(vaikimisi 1) ·**Iga**N ühiku järel (vaikimisi 5 s) ·**Kestusega** N ühikut (vaikimisi 1 m). |

Pidevas või intervallrežiimis muutub nupp „Kõik pildistada” töötamise ajal nupuks **Peata (N)**, loendades pildistamisi nende saabumise järgi. Täis mälukaart peatab töötamise (vt [„Kõik pildistada“ voog](#the-capture-all-flow)).**Järjepidev režiim massiivides töötab tagapõhiselt (uus alates versioonist 1.3.0).** Kui iga nimekirjas olev kaamera kuulub massiivi (eraldiseisvad kaamerad on peidetud või välistatud), ei saada pidevas režiimis enam ühte päringut iga salvestuse kohta: Chloros i tagapõhi käitab salvestussilmust ise, kirjutades ühe sünkroniseeritud rühma samal ajal, kui valib järgmise, ning nupp küsib selle arvu. Kui nimekirjas on eraldiseisvad kaamerad, salvestab Chloros nagu varemgi pass-haaval.

{% hint style="warning" %}
**Kiire salvestamine Windowsis: jäta Chloros i backend Microsoft Defenderist välja.** Iga faili reaalajas skaneerimine, mida kiire salvestamine kirjutab, võtab MAPIR i testide kohaselt umbes 30 % salvestuskiirusest (sadu faile sekundis väikese huvipiirkonna puhul). Windows i paigaldaja pakub valikut **„Jäta Chloros i backend Microsoft Defenderi skaneerimisest välja (palju kiirem kaamera salvestamine; soovitatav)“**, mis on vaikimisi märgitud, ning eemaldab selle väljajätmise desinstallimisel. Kui te selle valiku tühistasite, käivitage paigaldaja uuesti, jättes selle valiku märgitud, või lisage Defenderi protsesside välistuste nimekirja `chloros-backend.exe` (Chloros i paigalduskausta kaustas `backend`). Kui Defender kasutab massiivi pideva salvestamise ajal ühte või enamat CPU tuuma, märgib Chloros i backend seda üks kord oma logisse, pakkudes samu kahte lahendust.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Kaamerate ja eksporditüüpide valimine

Paneeli abitekst võtab selle kokku: valige, milliseid kaameraid ja eksporditüüpe „Capture All“ toodab – vaikimisi on kõik sisse lülitatud ja valikud salvestatakse koos selle projektiga.

* Nupud **„Vali kõik” / „Ära vali midagi”** lülitavad korraga sisse või välja kõikide kaamerate valikukastid.
* **Eksporditüüpide masslülitid**(kaks rida nuppe):**Kõik Raw / Kõik Debayered / Kõik eelvaade / Kõik Radiance / Kõik Reflectance / Kõik Index**. Igal lülitil on kolm värvi: roheline ✓ = sisse lülitatud kõigi seda toetavate kaamerate puhul, merevaigukollane – = sisse lülitatud mõnede puhul, hall = puudub. Lüliti on keelatud, kui ükski ühendatud kaamera seda tüüpi ei toeta. Kõik lülitid muutuvad halliks, kui funktsioon „Kiireim pildistamine“ on sisse lülitatud.
* **Kaamerapõhised read**: valikukast „Kaasa võtta“ ning laienev (▸/▾) nimekiri selle kaamera sobivatest eksporditüüpidest, millel on eraldi valikukastid. Ridas on näidatud sisselülitatud kaamerate arv, näiteks „4/6“.

### Eksporditüübid ja millised kaamerad neid toetavad

Eksporditüüpe on kuus: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Iga kaamera real kuvatakse ainult asjakohased tüübid:

| Eksporditüüp | Sisu | RGB (FRGB) | Bayeri multispektraalne (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mosaiik (mono: üks riba) otse sensorist | ✓ | ✓ | ✓ |
| **Debayered** | Lineaarne demosaik: kalibreerimispaketi tume nihke (DSNU) eemaldatud toorandmetest, seejärel servadega arvestav demosaik — ilma lineaarsuse korrigeerimise, tasase välja või segamise eemaldamiseta. Salvestatud 3-kanalilise RGB-TIFF-failina (mono: 1-kanaliline halltoon) | ✓ | ✓ | ✓ |
| **Eelvaade** | Täielik kuvamisahel (valgebalanss + gamma vastavalt kaamera profiilile, lisaks RGB-kaamera „Highlights“ seadistus; multispektraalne: valevärvide venitus) | ✓ | ✓ | ✓ |
| **Kiirgus** | float32 W/m²/sr/nm täieliku radiomeetrilise ahela kaudu | — (ei pakuta) | ✓ | ✓ |
| **Peegeldusvõime** | uint16 ρ (32768 = 1,0) | — (ei pakuta) | ✓ — kuvatakse ainult siis, kui kaameral on DAQ-valgusandur (oma või massiivilt pärandatud) | sama mis multispektraalne |
| **Indeks** | Taimestikuindeks (LUT) render | — | ✓ — nõuab kaameral aktiveeritud, mittetühja indeksaväljendit ning ei ole saadaval kombineeritud massiivi liikmetele (massiivil on üks ühine indeks) | — (indeks vajab ≥2 laineala; vt [Mono-kaamerad ja taimestiku indeksid](mono-indices.md)) |

RGB-kaamerate puhul ei pakuta kunagi kiirgust ega peegeldust — lairiba fotomeetrilise anduri puhul ei ole Bayeri-põhine kiirgus mõttekas.

### Kiireim salvestamine

Lüliti **⚡ Kiireim salvestamine — ainult RAW**(sisse lülitatuna oranž) asendab kõik ekspordivalikud**ainult RAW-vorminguga** — lisaks tasuta kombineeritud indeksiga komposiit massiivide jaoks — nii et kaader salvestatakse võimalikult kiiresti: kiirguse/peegeldusvõime/kuvamise arvutused jäetakse salvestamise ajal täielikult vahele.

{% hint style="info" %}
**`.daq` salvestatakse ikkagi.** Kui on määratud valgussensor, salvestab „Kiireim salvestamine“ ikkagi DAQ-i allapoole suunatud mõõtmistulemuse (salvestuste `daq/`-kausta) — nii saab kiirgust, peegeldust ja indeksit käsitlevaid tooteid hiljem uuesti töötlemise teel luua (vt [Salvestuste ümbertöötlemine](#re-processing-captures-into-calibrated-products)). „Fastest Capture“ ei mõjuta ka teie valikukastide sätteid: lülitage see välja ja need taastuvad.
{% endhint %}

### Massiivi-põhised juhtimisfunktsioonid

Igal ühendatud massiivil on paneelis oma rühmakort:

* **Valikukast „Include“** (kolme seisundiga liikmete lõikes) ja massiivi nimi koos kuvamisrežiimiga: „(combined | separate)“.
* **Joondatud**valikukast (vaikimisi**sisse**): joondab liikmete ekspordid massiivi joondamisprofiili järgi, nii et ekspordid on kaamerate vahel pikselitäpselt joondatud. Toores materjal jääb moonutamata, kuid kannab transformatsiooni oma metaandmetes. (Profiil ise arvutatakse välja [massiivi seadeaknas](camera-settings.md#alignment-co-registration).) Massiivi liikmete failid salvestatakse iga kaamera algses orientatsioonis, kusjuures peegeldus- ja pööramisseaded salvestatakse faili, mitte ei rakendata.
* Kaameraelementide read on kaardi sees pesastatud.

Massiivi kaart sisaldab ka kahte salvestajat. Mõelge neile kui **jälgimisele ja analüüsile**:

| Salvestaja | Tase | Mida salvestab |
| --- | --- | --- |
| **● Salvestab indeksvideot / ■ Peatab salvestamise** *(ainult kombineeritud massiivid)* | **Jälgimine** | Reaalajas kombineeritud indeks-komposiitvideot kiirusega 10 kaadrit sekundis — 8-bitine, eelvaate resolutsioon, sisseehitatud LUT. Vajab avatud projekti ja reaalajas voogedastust. Näitab kaadreid ja salvestamise kestust. |
| **⦿ Salvestab toorkaadrite seeria / ■ Toores pildiseeria salvestamise lõpetamine** *(mis tahes kaameramassiiv)* | **Analüüs**| Toored Bayer-kaadrid reaalajas salvestamise kiirusel (töötlemata) koos kaadrihaaval manifestiga ja `.daq` näitudega, salvestatuna failina `captures/bursts/`. Pärast sarivõtet ilmub nupp**Loo video**: see töötleb sarivõtte offline-režiimis ümber kalibreeritud videoks — kombineeritud indeks ja/või kaamera-põhine kiirgus / peegeldus / indeks — pluss valikulised TIFF-failid. Kombineeritud indeksiga video loomine algab automaatselt, kui peatate sarivõtte. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## „Capture All“ töövoog

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->



Vajuta külgribal asuvas kaamerate loendis nuppu **„Capture All“**:

1. Iga kaasatud, nähtav ja pausile panemata kaamera salvestab valitud eksporditüüpide järgi. **Kaamerarühmad käivituvad ühe sünkroniseeritud päästikuna** (ühtne sünkroniseeritud rühm kõigi liikmete vahel — vt [Mitme kaameraga rühmad](arrays.md)); eraldiseisvad kaamerad salvestavad individuaalselt.
2. Peidetud (silma) või pausile pandud kaamerad jäetakse vahele. Kaamerarühm blokeeritakse täielikult alles siis, kui *kõik* selle liikmed on peidetud või pausile pandud.
3. Kui valgussensor on määratud, salvestatakse vastav DAQ-i allapoole suunatud valguse mõõtmistulemus `.daq`-failina salvestuste `daq/`-kausta — isegi ainult toorandmetena salvestatud materjalide puhul —, et radiomeetrilisi tulemusi saaks alati hiljem tuletada.
4. Nupp näitab reaalajas edenemist – „Salvestamine… valmis/kokku“ – ning pidevas/intervallrežiimis muutub see **Stop (N)**-iks. Igal salvestuselemendil on 300-sekundiline ooteaeg.
5. Kui salvestusvoor lõpeb, kuvab tulemuste teade **„Salvestatud N faili”**või**„Salvestatud N, F ebaõnnestus”**, lisaks „(S peidetud/peatatud/vahele jäetud)”, kui kaamerad jäeti vahele. Kui kõvaketas täitub, peatub töö ja kuvatakse teade**„Peatatud: kõvaketas täis. Salvestatud N, F ebaõnnestus”** — iga fail, mida ei suudetud kirjutada, loetakse ebaõnnestunuks.

## Kuhu salvestused paigutatakse

Salvestused salvestatakse avatud projekti alla kataloogi `<project>/captures/`. Iga eksporditüüp paigutatakse oma **alakataloogi**, seega mitmetasandilises salvestuses ei segune tüübid kunagi:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` on salvestuse ajamärge ja `<serial>` kaamera seerianumber. Eraldi salvestused nimetatakse `capture_<ts>_SN<serial>_<level>`; ühe sünkroniseeritud päästiku abil tehtud masssalvestused nimetatakse `sync_<ts>_SN<serial>_<level>` ja **neil on ühine ajamärge kõigi rühma kaamerate jaoks** (taseme sufiks jäetakse ära, kui kaamera salvestab ainult ühe taseme).
* **Üks erand, mida tuleb teada:** kuvatase salvestatakse kausta nimega `preview/`, samas kui failide nimedes säilitatakse `_display` — kausta ja sufiksi erinevus kehtib ainult selle taseme puhul.
* Tundmatud tasemed salvestatakse oma nime kandvasse kausta; kui alamkausta ei ole võimalik luua, salvestatakse fail salvestiste juurkataloogi, selle asemel et see kaoks.
* Salvestatud TIFF-failid on vaikimisi kadudeta pakitud (DEFLATE) ning sisaldavad kogu kalibreerimis- ja töötlemismetadata **faili XMP-s** — pildid on iseloomustavad ega vaja muid lisafaile peale `.daq`-i (ja selle `.daq.cameras.json`-i).
* **Klambermaskid (uus versioonis 1.3.0).**Kui 3×3 spektraalne eraldamine pidi vähemalt ühel pikselil sagedusriba 0-ni klamberdama, saavad kiirguse, peegeldusvõime ja indeksi tooted pikselipõhise maski, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = toote R-kanali sagedusala, bit 1 = G, bit 2 = B); toote `Chloros:UnmixClampMask` XMP-märge viitab sellele. Maskid asuvad**toote kaustade kõrval**, mitte kunagi nende sees, seega saab sellist tootekaustat nagu `reflectance/` edastada fotogramm-meetria tarkvarale sellisena, nagu see on. Chloros e enda importimisfunktsioon jätab maskid vahele.
* **Orienteerimine (1.3.0).** Eraldi kaamera eelvaate-, kiirguse-, peegeldus- ja indeksitooted pööratakse vastavalt selle peegeldus-/pööramisseadistusele; RAW- ja debayered-failid jäävad selliseks, nagu andur need luges. Kõik massiivi liikmefailid jäävad algupäraseks. Iga fail salvestab seadistuse oma XMP-sse, mistõttu uuesti töötlemisel rakendatakse seda samamoodi.
* **Millisele kaamerale kuulub `.daq` (1.3.0).** Pildistamise `.daq` sobitatakse töötlemise ajal ainult nende kaamerate piltidega, mille nimed on märgitud selle `.daq.cameras.json`-is. Seetõttu ei kasuta eraldiseisev kaamera, millel puudub oma valgusandur, enam teise kaamera näitu – seostage sellega andur või salvestage see vahekaardil „Valgusandurid”. Valgusanduri salvestus (ja iga `.daq`, millel seda faili pole, sealhulgas vanemad salvestused) teenib endiselt kõiki kaameraid.

See on sama struktuur, mille `chloros-cli lattice capture` / `array-capture` kirjutavad oma `-o` kataloogi – dokumenteeritud [CLI-viite § Kuidas näeb välja salvestuste kaust](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Salvestiste ümbertöötlemine kalibreeritud toodeteks

Salvestatud töötlemata kaadrid ja salvestatud `.daq` on kõik, mida töötlemisprotsess vajab – seetõttu on „Fastest Capture“ reaalseks tööks ohutu.

* **GUI**: lisage salvestuste kaust projektile ([Failide lisamine projektile](../processing-images-gui/adding-files-to-a-project.md)) ja töötlege nagu tavaliselt.
* **CLI**: suunake `process` **salvestuste juurkaustale**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` impordib tavaliselt ainult nimetatud kausta, kuid kui selles kaustas pole pilte ja seal on alamkaustad, liigub programm automaatselt allapoole — nii et alamkaustade tasemed ja `daq/` kaust võetakse ühe korraga sisse. Iga salvestus imporditakse **ühe pildina**, mille juurde on lisatud teised tasemed vaatamisrežiimidena, mitte ühe pildina iga taseme kohta.

Taseme alamkausta otsene nimetamine (nt `…/captures/raw/`) toimib samuti, kuid jätab kausta `daq/` kõrvale – kopeerige see kõrvale, kui te- tuletate radiomeetrilist toodet kaustast `raw/`, vastasel juhul ei ole ajamärgise vastavusel millegi suhtes lahendada.

{% hint style="warning" %}
**Töötlemine algab alati kaustast `raw`.** Iga salvestuse puhul on töötlemisahela allikaks töötlemata kaader; `debayered`, `radiance`, `reflectance`, ja `preview` on vaatamisrežiimid, kuid neid ei suunata kunagi tagasi töötlusvoo kaudu — tuletatud toote uuesti töötlemine rakendaks uuesti vinjeti, värvi ja kiirguse matemaatikat, mis on juba pikslitesse sisse põletatud, seetõttu loobub Chloros sellest, selle asemel et teha topelt-töötlust. Renderdusi `index/` ja `composite/` ei töödeldagi üldse (need on väljundid, mitte salvestused). Salvestatud kaadrite kaust, milles **puuduvad** toorandmed, kuvatakse normaalselt, kuid `process` jätab selle vahele ja teatab sellest; `--input-level {raw,debayered,processed}` on tahtlik pääsetee, mis sunnib sissepääsupunkti kasutama. Täpsed vahelejätmisteated leiate [CLI-viitest](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Veel kaks käitumist, mida tasub teada skriptide abil ümbertöötlemisel:

* `chloros-cli process` käivitus, mis taotles väljundeid, kuid **ei kirjutanud ühtegi pildiväljundit, ebaõnnestub märgatavalt ja lõpetab töötamise nullist erineva tulemusega** — vaikset tühja käivitust ei toimu kunagi. Edukad käivitused teatavad väljundite arvu. (Tahtlikult ainult metaandmetele suunatud käivitus loetakse ikkagi edukaks.)
* Uuesti imporditud töödeldud ekspordid ei võta kunagi pildi toorandmete pesa — algne toorandmete fail jääb alati töötlemisprotsessi allikaks.

## CLI-ekvivalendid

Kõiki sellel lehel kirjeldatud toiminguid saab teostada graafilise kasutajaliidese (GUI) kasutamata. GUI pildistamisrežiimid vastavad otseselt järgmistele -käivitustele:

| GUI | CLI |
| --- | --- |
| Ühekordne | `chloros-cli lattice array-capture` |
| Pidev | `array-capture --continuous [--count N] [--duration S]` |
| Intervall | `array-capture --interval S [--duration S]` |
| Kiireim salvestamine | `array-capture --fastest` |
| Joondatud valikukast | `--aligned / --no-aligned` |
| Eksporditüübi valikukastid | `--processing LEVEL` või `--levels L1,L2,…` (vaikimisi `all`) |
| Salvestada indeksvideo | `chloros-cli lattice array-record` |
| Salvestada töötlemata pildiseeria / Koostada video | `chloros-cli lattice array-burst` / `array-build-video` |

Täielikud liputabelid, nutika automaatse säri (smart-AE) püsiv pildistamise valik (`--smart`) ja püsiva kiirusega mudel on kirjeldatud [CLI-viite jaotises „Pildistamisrežiimid, salvestid ja offline-töötlemine”](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
