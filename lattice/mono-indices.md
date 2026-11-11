# Monokroomkaamerad ja taimestiku indeksid

## Üks kaamera = üks lainepikkus

**M3M**-kaamera on Bayeri**M3C**-kaamera monokroomne versioon: ühe kitsasribalise interferentsfiltri taga asub monokroomne IMX265-andur. Mudelijärjekord määrab lainepikkuse – `M3M-<lens>-F<wavelength>`, nt `M3M-L87-F685` (Chloros es kuvatakse see kui `LATT-M3M-L87-F685`). Andur annab **ühe halltoonide sagedusriba** ilma Bayeri mosaiigita: pole midagi demosaikida, pole kanalitevahelist ülekandumist, mida lahti segada, ega valgebalanssi, mida seadistada.

Mono-süsteemi planeerimisel tasub teada järgmisi asjaolusid:

* **Kiirgus ja peegeldus on iga riba puhul täielikult määratletud.**Tegemist on ribapõhiste radiomeetriliste kaartidega, seega toodab üks M3M-kaamera kalibreeritud float32 kiirgust (W/m²/sr/nm) ja uint16 peegeldust (`32768` = ρ 1,0) täpselt samamoodi nagu M3C-riba. Mono-kaadrid sisaldavad**identiteetset** anduri vastusmaatriksit – 3×3 segamise eemaldamist ei ole vaja ega rakendata.
* **Üks mono-kaamera ei suuda toota taimestiku indeksit.** NDVI, NDRE ja teised sarnased indeksid vajavad vähemalt kahte laineala. Indeksite arvutamiseks mono-riistvaraga tuleb kombineerida mitu M3M-kaamerat – vt allpool.
* M3M-kaamerad edastavad **Mono12** (12-bitine: 2 baiti/piksel andmeside kaudu või 1,5 baiti/piksel, kui maatriks kasutab [pakitud 12-bitist edastust](arrays.md#packed-12-bit-transport)), mis on oluline [maatriksi ribalaiuse eelarvestamisel](arrays.md#bandwidth-how-many-cameras-fit).

## Mida Chloros mono puhul vahele jätab — ja kuidas ta sellest teavitab

Värvipipeliini etapid lihtsalt ei kehti ühe sagedusriba anduri puhul. Chloros **jätab need vahele üherealise teatega**, selle asemel et veateadet kuvada, ning käivitab need samas sessioonis ikkagi tavapäraselt mis tahes M3C (Bayer) kaamera puhul:

| Etapp | Mono (M3M) käitumine | M3C käitumine |
| --- | --- | --- |
| Demosaic / debayer | Vahelejäetud — `debayered` eksporditase on 1-kanaliline halltoonipilt. | 3-kanaliline demosaic. |
| Valgebalanss (`lattice white-balance`) | Vahelet jäetud koos üherealise teatega. | Töötab normaalselt. |
| Värviprofiil (`lattice color-profile`) | Vahelet jäetud koos üherealise teatega. | Töötab normaalselt. |
| Küllastus/kontrast (`lattice color`) | Vahelet jäetud üherealise teatega. | Töötab normaalselt. |
| Spektraalne ristkõne eraldamine | Identiteet (ei ole 3×3 maatriksit). | Rakendatakse kaamera-põhist 3×3 maatriksit. |
| Kiirgus / peegeldusvõime | **Töötab** — iga sagedusriba kohta, täielikult kalibreeritud. | Töötab iga sagedusriba kohta. |

GUI rakendab sama filtreerimist: monokaamera puhul peidab kaamera-põhine seadete paneel ainult RGB-ridu (valgebalanss, gamma, värviprofiil, küllastus, kontrast, kanalite jagamine) ning reaalajas histogramm on lukustatud ühele **MONO** joonele. Kogu pildistacki eristajaks on mudeli stringis olev märge `M3M`, mis kuvatakse kasutajaliideses/SDK-s kui `is_mono`.

## Indeksid vajavad ≥ 2 laineala: joondamine → pildistack → indeks

Monoindeksi töövoog koosneb alati samast kolmest sammust:

1. **Joondamine** — suunake mitu M3M-kaamerat erinevatele lainepikkustele (nt F650 „Red” ja F850 „NIR”), ühendage need [mitme kaamera massiiviks](arrays.md) ja laske Chloros el arvutada kaameratevaheline ko-registreerimise moonutus.
2. **Kuhjamine** — joondatud kaadrid muutuvad üheks mitme lainepikkusega pildiks (iga kaamera annab ühe nimega lainepikkuse).
3. **Indeks** — arvutage indeksivalem pildi kihtide kohta, renderdades seda soovi korral LUT-i kaudu.

Graafilises kasutajaliideses on kogu see ahel **Combined Cameras**massiivi kuvarežiim: reaalajas komposiitpilt on juba joondatud ning massiivi indeksikalkulaator (allpool) määrab selle renderdamise valemi. Salvestatud ekspordid saab**Aligned** salvestusvalikuga moonutada samasse joondusse.

## Indeksiarvuti

Indeksiarvuti koostab indeksiväljendi, mida kasutatakse reaalajas vaates ja kaamerapõhistes indeksekspordides. See on üks ühine pind, mida saab avada kahe koha kaudu vahekaardil „Cameras”:

* **Kaamera kaupa**— Reaalaja eelvaade →**Indeks** hammasratas (ainult RGN-, OCN- ja NGB-Bayer-kaamerad; ühel mono-kaameral puudub indeksikontroll, kuna ühe lainepikkusega ei saa indeksit luua).
* **Iga maatriksi kohta**— maatriksi seaded → Reaalaja eelvaade →**Indeks**hammasratas. See on monokroomne tee: lainepikkuste nimekiri hõlmab**kõiki kuuluvate kaamerate** lainepikkusi, seega annab monokroomne paar siin oma kaks lainepikkust.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Selle juhtelemendid, ülevalt alla:

* **Lainepikkuste kiibid** („Lainepikkused — klõpsa, et lisada avaldisse”) — üks nupp iga saadaval oleva lainepikkuse kohta, millel on märgitud värvinimi + lainepikkus nm-des (korduvad värvinimed eristatakse, nt „Värv 850”). Klõpsates lisatakse lainepikkuse märk kursori asukohta. Lained kaameratest, mis ei suuda toota lainepõhist kiirgust (nt RGB/FRGB), filtreeritakse välja.
* **Operaatorite ja funktsioonide nupud** — `+ - * / ( ) ^ ,` ja `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Väljendi tekstiväli** — vabalt sisestatav valem; asendusmärk näitab klassikalist NDVI-vormi `(NIR - Red) / (NIR + Red)`. Selle kohal asuv ainult lugemiseks mõeldud tokeniseeritud eelvaade kuvab ribade kiibid, numbrid ja lipud tundmatute tokenitena.
* **Kehtivuse bänner**— hall „Tühi — indeksit ei rakendata“; roheline „Kehtiv avaldis“; punane koos konkreetse analüüsiveaga (tundmatu laineala, mitme kaamera poolt jäädvustatud ebaselge laineala, puuduv sulg, …); või kollane, kui avaldis on kehtiv, kuid**konstant** (nt `X/X` või NDVI-nimetaja, mis on sisestatud `−` asemel `+`) — konstant kaardistab kogu kaadri ühe värviga.
* Eraldi kollane hoiatus ilmub, kui kasutatud väljend on korrektne, kuid **reaalajas kaader on ühtlane** (ühtlane või küllastunud stseen) — histogrammi kokkuvarisemine tuvastatakse teie eest.
* **LUTi rakendamine**(vaikimisi sisse lülitatud; välja lülitatud = halltoonide venitamine),**Tase**2/3/5/7-stop (vaikimisi 7-stop) ja**Min / Max**sisendid, mis asuvad gradiendiriba kõrval. Min on vaikimisi**0,2**— see suumib värviskaala taimestikuga seotud vahemikku, samas kui sellest madalamad väärtused kuvatakse halltoonidena; seadke Min väärtuseks −1, et saada kogu indeksivahemik (nupp**Reset** taastab vahemiku −1…+1). Max on vaikimisi 1.
* **Reaalajas histogramm** indeksi jaotuse kohta — ruutjuurega skaalitud tulbad, merevaigukollased p2/p98 protsentiili jooned, valge mediaanjoon ja vahemikust väljaspool olevad näidud („◀ N% < lo“ / „hi < N% ▶”), mis muutuvad merevaigukollaseks üle 1 % taseme, andes märku Min/Max-vahemiku laiendamise vajadusest.
* **Rakenda** rakendab avaldist reaalajas voogule; LUT-i muudatused rakenduvad reaalajas ilma nupule „Rakenda” vajutamata. Kui projekt on avatud, salvestatakse avaldis ja LUT-i seaded koos projektiga ning taastatakse projekti uuesti avamisel; projekti puudumisel kehtivad need ainult selle seansi jooksul.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## CLI-tee

Sama joondamine → virnastamine → indekseerimine ahel, skriptitav algusest lõpuni:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` kasutab ühendatud massiivi, mis hoiab oma kaameraid, või avab need ise ühe käivituse jaoks. `index --live` avab kaamerad alati ise (ajutine massiiv (../reference/cli-reference.md#multi-camera-transient-sessions)) ja tal puudub `--serials`: see avab kõik võrgus olevad kaamerad. Seega vabastage kõik kaamerad, mida Chloros enne seda hoiab — GUI, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): hoitud kaamera teistkordne avamine võib peatada selle hoidja voo või põhjustada tagapõhja kokkujooksmise. Kaamera, mis ei edasta ühtegi kaadrit, põhjustab käivituse ebaõnnestumise ning virna ega indeksit ei kirjutata.

`--channel` seostab eelseadistuse sümbolid virna sagedusribade nimedega. Kaks reeglit aitavad vältida ebaõnnestunud käivitust:

* **Sümbolites arvestatakse suur- ja väiketähti** ning need peavad täpselt vastama eelseadistuse kanalite nimedele — eelseadistustes kasutatakse väiketähti (nt NDVI puhul on need `red`, `nir`; vaata `--list-presets`). `--channel red=Red_660` töötab; `--channel RED=660` ebaõnnestub veaga `channel_map missing entries`.
* Lindi poolel peab olema nimetatud joondatud pildistiku lint (`lattice align-info --profile align.json` loetleb need). Offline-režiimis aktsepteeritakse ka 0-põhiseid lindiindekseid, nt `--channel red=0 --channel nir=1`.

`lattice index` töötab ka offline-režiimis salvestatud joondatud mitme ribaga TIFF-faili puhul. Iga vorm töötab tagapõhiselt, seega peab see töötama ja olema sisse logitud tasulise Chloros+ taseme all; suhteline `--input` või `--output` viitab kaustale, kust käsk käivitatakse:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indeksi eelseadistused

`lattice index --preset` (ja vahekaardi „Pilt” all olev [Indeksi/LUT-liivakast](../image-viewer-gui/index-lut-sandbox.md), mis kasutab sama mootorit) sisaldab järgmisi **22 eelseadistust**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Käivitage `chloros-cli lattice index --list-presets` iga eelseadistuse valemi ja kanalisümbolite vaatamiseks ning `--list-gradients` kättesaadavate värvigradientide vaatamiseks. Kohandatud valemite puhul kasutage `--formula EXPR`-i, mille süntaks on sama kui indeksikalkulaatoril. Pange tähele, et see eelseadete nimekiri kehtib spetsiaalselt LATTICE indeksimootori puhul — imporditud piltide puhul on projekti seadete töötlemise rippmenüüs teine nimekiri (vt [Multispektraalsed indeksivalemid](../project-settings/multispectral-index-formulas.md)).

Täielik lipukomplekt (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, joondamise ja moonutamise nupud `--live` jaoks jm) on dokumenteeritud [CLI-viite § Indeks / Taimestiku matemaatika](../reference/cli-reference.md#index--vegetation-maths); SDK-vasted on toodud [SDK-viites](../reference/sdk-reference.md).

## Indeksitoodete salvestamine monomaatriksist

Kui maatriks on ühendatud ja indeksaväljend rakendatud, salvestab `array-capture` (või GUI **Capture All**) salvestab kaamera-põhised eksporditasemed *ja* indeksrenderduse — `--index`/`--no-index` lülitab selle CLI-s sisse ning salvestab vaikimisi kõik kohaldatavad tasemed. Mono-kaamera panus igasse salvestusgruppi on selle üks sagedusriba toorandmete/debayereeritud (halltoonid)/kiirguse/peegelduse tasanditel, millele lisandub ühine kombineeritud indeksiga komposiitpilt, kui massiiv töötab kombineeritud režiimis. Vaata [Mitme kaameraga massiivid § Salvestamine](arrays.md#capturing-monitoring-vs-analysis).
