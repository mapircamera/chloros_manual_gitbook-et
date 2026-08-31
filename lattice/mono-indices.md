# Monokroomkaamerad ja taimestiku indeksid

## Üks kaamera = üks lainepikkus

**M3M**-kaamera on Bayeri**M3C**-kaamera monokroomne versioon: ühe kitsasribalise interferentsfiltri taga asub monokroomne IMX265-andur. Mudeli stringis on märgitud laine – `M3M-<lens>-F<wavelength>`, nt `M3M-L87-F685` (kuvatakse Chloros kujul kui `LATT-M3M-L87-F685`). Andur annab**ühe halltoonide sagedusriba** ilma Bayeri mosaiigita: pole midagi demosaikida, pole kanalitevahelist ülekandumist, mida lahti segada, ega valgebalanssi, mida seadistada.

Mono-süsteemi planeerimisel tasub teada järgmisi asjaolusid:

* **Kiirgus ja peegeldus on iga riba puhul täielikult määratletud.**Need on ribapõhised radiomeetrilised kaardid, seega toodab üks M3M-kaamera kalibreeritud float32 kiirgust (W/m²/sr/nm) ja uint16 peegeldust (`32768` = ρ 1,0) täpselt samamoodi nagu M3C-riba. Mono-kaadrid sisaldavad**identiteetset** anduri vastusmaatriksit – 3×3 segamise eemaldamist ei ole vaja ega rakendata.
* **Üks mono-kaamera ei suuda luua taimestikuindeksit.** NDVI, NDRE ja teised sarnased indeksid vajavad vähemalt kahte sagedusala. Indeksite arvutamiseks mono-riistvaraga tuleb kombineerida mitut M3M-kaamerat – vt allpool.
* M3M-kaamerad edastavad **Mono12**-andmevoogu (12-bitine, 2 baiti pikseli kohta), mis on oluline [andurite ribalaiuse eelarvestamisel](arrays.md#bandwidth-the-rules-of-thumb).

## Mida Chloros mono puhul vahele jätab – ja kuidas see sulle teatab

Värvipipeliini etapid lihtsalt ei kehti üheribalise anduri puhul. Chloros **jätab need ühe rea pikkuse sõnumiga vahele**, selle asemel et veateadet kuvada, ning käitab neid ikkagi tavapäraselt mis tahes M3C (Bayer) kaamera puhul samas sessioonis:

| Etapp | Mono (M3M) käitumine | M3C käitumine |
| --- | --- | --- |
| Demosaic / debayer | Vahelejäetud — `debayered` eksporditase on ühekanaliline halltoonipilt. | 3-kanaliline demosaic. |
| Valgebalanss (`lattice white-balance`) | Vahelet jäetud koos üherealise teatega. | Töötab normaalselt. |
| Värviprofiil (`lattice color-profile`) | Vahelet jäetud koos üherealise teatega. | Töötab normaalselt. |
| Küllastus/kontrast (`lattice color`) | Vahelet jäetud üherealise teatega. | Töötab normaalselt. |
| Spektraalne risthäirete eraldamine | Identiteet (puudub 3×3 maatriks). | Rakendatakse kaamera-põhist 3×3 maatriksit. |
| Kiirgus / peegeldusvõime | **Töötab** — iga sagedusriba kohta, täielikult kalibreeritud. | Töötab iga sagedusriba kohta. |

GUI rakendab sama filtreerimist: monokaamera puhul peidab kaamerapõhine seadete paneel ainult RGB-ile mõeldud read (valgebalanss, gamma, värviprofiil, küllastus, kontrast, kanalite jagamine), ning reaalajas histogramm on lukustatud ühele **MONO** joonele. Kogu pildistacki eristajaks on mudeli stringis olev `M3M`-märge, mis kuvatakse kasutajaliideses/SDK-is kui `is_mono`.

## Indeksid vajavad ≥ 2 riba: joondamine → pinu → indeks

Monoindeksi töövoog koosneb alati samadest kolmest sammust:

1. **Joondamine** — suunake mitu M3M-kaamerat erinevatele lainepikkustele (nt F650 „Red” ja F850 „NIR”), ühendage need [mitme kaamera massiiviks](arrays.md) ja laske Chloros-il arvutada kaameratevaheline ko-registreerimise moonutus.
2. **Kogum** — joondatud kaadrid moodustavad ühe mitme lainepikkusega pildi (iga kaamera annab ühe nimega lainepikkuse).
3. **Indeks** — arvutage indeksivalem kogumi lainepikkuste põhjal, renderdades seda soovi korral LUT-i kaudu.

Graafilises kasutajaliideses on kogu see ahel **Combined Cameras**massiivi kuvamisrežiim: reaalajas komposiitpilt on juba joondatud ning massiivi indeksikalkulaator (allpool) määrab kindlaks selle renderdamise valemi. Salvestatud ekspordid saab joondada samasse asendisse**Aligned** salvestusvaliku abil.

## Indeksiarvuti

Indeksiarvuti koostab indeksaväljendi, mida kasutatakse reaalajas vaates ja kaamerapõhistes indeksieksportides. See on üks ühine pind, mida saab avada kahe koha kaudu vahekaardil „Kaamerad”:

* **Kaamera kaupa**— Reaalaja eelvaade →**Indeks** hammasratas (ainult RGN/OCN/NGB Bayeri kaamerad; ühel mono-kaameral puudub indeksikontroll, kuna ühe lainepikkusega ei saa indeksit moodustada).
* **Iga massiivi kohta**— massiivi seaded → Reaalaja eelvaade →**Indeks**hammasratas. See on mono-tee: lainepikkuste nimekiri hõlmab**kõiki massiivi kuuluvaid kaameraid**, seega panustab mono-paar siia oma kaks lainepikkust.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Selle juhtelemendid ülevalt alla:

* **Lainepikkuste kiibid** („Lainepikkused — klõpsa, et lisada avaldisse”) — üks nupp iga kättesaadava lainepikkuse kohta, millel on märgitud värvinimi + lainepikkus nm-des (korduvad värvinimed eristatakse, nt „Värv 850”). Klõpsates lisatakse laineala märk kursori asukohta. Lainealad kaameratest, mis ei suuda toota laineala-põhist kiirgust (RGB/FRGB), filtreeritakse välja.
* **Operaatorite ja funktsioonide nupud** — `+ - * / ( ) ^ ,` ja `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Väljendi tekstiväli** — vabalt sisestatav valem; asendusmärk näitab klassikalist NDVI-vormi `(NIR - Red) / (NIR + Red)`. Selle kohal asuv ainult lugemiseks mõeldud tokeniseeritud eelvaade kuvab ribade tükid, numbrid ja lipud tundmatute tokenitena.
* **Kehtivuse bänner**— hall „Tühi — indeksit ei rakendata”; roheline „Kehtiv avaldis“; punane koos konkreetse analüüsiveaga (tundmatu sagedusriba, mitme kaamera poolt tuvastatud ebaselge sagedusriba, puuduv sulg, …); või kollane, kui avaldis on kehtiv, kuid**konstantne** (nt `X/X`, või nimetaja NDVI, mis on sisestatud `−` asemel `+`) — konstant kaardistab kogu kaadri ühe värviga.
* Eraldi merevaigukollane hoiatus ilmub, kui kasutatud avaldis on korrektne, kuid **reaalajas kaader on ühtlane** (ühtlane või küllastunud stseen) — histogrammi kokkuvarisemine tuvastatakse teie eest.
* **LUTi rakendamine**(vaikimisi sisse lülitatud; välja lülitatud = halltoonide venitamine),**Tase**2/3/5/7-stop (vaikimisi 7-stop) ja**Min / Max**sisendid, mis asuvad gradiendiriba kõrval. Min on vaikimisi**0,2**— see suurendab värviskaalat taimestikuga seotud vahemikku, samas kui sellest madalamad väärtused kuvatakse halltoonidena; seadke Min väärtuseks −1, et saada kogu indeksivahemik (nupp**Reset** taastab vahemiku −1…+1). Max on vaikimisi 1.
* **Reaalajas histogramm** indeksi jaotuse kohta — ruutjuurega skaalitud tulbad, merevaigukollased p2/p98 protsentiili jooned, valge mediaanjoon ja vahemikust väljaspool olevad näidud („◀ N% &lt; lo“ / „hi &lt; N% ▶”), mis muutuvad merevaigukollaseks üle 1 %-i, andes märku Min/Max-akna laiendamise vajadusest.
* **Rakenda**rakendab avaldist reaalajas voogule; LUT-i muudatused rakenduvad reaalajas ilma nupule „Rakenda“ vajutamata. Avaldised on tahtlikult**ainult sessioonipõhised** — neid ei säilita sessioonide vahel.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## CLI-teek

Sama joondamine → virn → indeksiahel, skriptitav algusest lõpuni:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` seostab eelseadistuse sümbolid virna ribanimedega. Kaks reeglit aitavad vältida ebaõnnestunud käivitamist:

* **Sümbolites arvestatakse suur- ja väiketähti** ning need peavad täpselt vastama eelseadistuse kanalite nimedele — eelseadistustes kasutatakse väiketähti (NDVI on `red`,`nir`; kontrollige `--list-presets`). `--channel red=Red_660` töötab; `--channel RED=660` ebaõnnestub veaga `channel_map missing entries`.
* Sagedusriba pool peab nimetama sagedusriba joondatud virnas (`lattice align-info --profile align.json` loetleb need). Offline-režiim aktsepteerib ka 0-põhiseid sagedusriba indekseid, nt `--channel red=0 --channel nir=1`.

`lattice index` töötab täielikult ka offline-režiimis salvestatud joondatud mitme sagedusribaga TIFF suhtes:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indeksi eelseadistused

`lattice index --preset` (ja vahekaardi „Image“ [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md), mis kasutab sama mootorit) sisaldab neid **22 eelseadistust**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Käivitage `chloros-cli lattice index --list-presets` iga eelseadistuse valemi ja kanalisümbolite vaatamiseks ning `--list-gradients` kättesaadavate värvigradientide vaatamiseks. Kohandatud valemite jaoks kasutage `--formula EXPR`-i, mille süntaks on sama kui indeksikalkulaatoril. Pange tähele, et see eelseadete nimekiri on spetsiifiline LATTICE indeksimootorile — imporditud piltide puhul on projekti seadete töötlemise rippmenüüs teine nimekiri (vt [Multispektraalsed indeksivalemid](../project-settings/multispectral-index-formulas.md)).

Täielik lipukomplekt (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, joondamise moonutusnupud `--live` jaoks, ja muud) on dokumenteeritud [CLI-viites § Indeks / Taimestiku matemaatika](../reference/cli-reference.md#index--vegetation-maths); SDK vasted on esitatud [SDK viites](../reference/sdk-reference.md).

## Indeksitoodete salvestamine mono-massiivist

Kui massiiv on ühendatud ja indeksavaldis rakendatud, salvestab `array-capture` (või GUI-funktsioon **Capture All**) iga kaamera eksporditasemed *ja* indeksrenderduse — `--index`/`--no-index` lülitab selle sisse CLI-is ning salvestab vaikimisi kõik sobivad tasemed. Ühe kaamera panus igasse salvestusgruppi on selle üks sagedusriba toores/debayered (halltoonides)/kiirguse/peegelduse tasemetel, millele lisandub jagatud kombineeritud indeksiga komposiit, kui massiiv töötab kombineeritud režiimis. Vaata [Mitme kaamera massiivid § Salvestamine](arrays.md#capturing-monitoring-vs-analysis).
