---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# KKK

<details>

<summary>Kas saan Chloros-iga töödelda ka muude tootjate kaamerate pilte peale MAPIR-i?</summary>

Ei, Chloros toetab ainult MAPIR kaamerate piltide töötlemist — Survey3 ja LATTICE seeriaid. Lisateabe saamiseks vaadake [toetatud kaameramudelite](supported-cameras.md) nimekirja. Pakume teiste kaamerate töötlemist MAPIR Cloudis, täieliku nimekirja leiate [siit](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Kas Chloros toetab LATTICE-kaameraid?</summary>

Jah. Chloros 1.2.0 toetab LATTICE M3C ja M3M kaameramooduleid terviklikult: **reaalajas juhtimine**— avastamine, ühendamine, eelvaade ja salvestamine GUI kaamerate vahekaardilt, `chloros-cli lattice` või Python SDK kaudu, sealhulgas sünkroniseeritud mitme kaamera massiivid PTP-ajasünkroniseerimisega — ning salvestiste**täielik radiomeetriline töötlemine** (toores → debayered → kiirgus → peegeldus → indeks). Vaata [Toetatud kaamerad](supported-cameras.md) ja [LATTICE juhendit](lattice/README.md).

</details>

<details>

<summary>Kas ma saan oma pilte peegeldusvõime jaoks kalibreerida ilma kalibreerimismärgita?</summary>

**Survey3:** Ei. Ilma kalibreerimismärgi pildita, mis on pildistatud samal ajal, kui pildistatakse muid pilte, ei ole võimalik seostada pildi pikselväärtusi teadaoleva peegeldusvõime protsendiga. Kui te ei lisa ka MAPIR valgusanduri logi, siis ümbritseva valguse spektrit ei mõõdeta ja peegeldusvõime tulemused ei ole täpsed.**LATTICE:** Jah. Peegeldusvõimet saab seostada DAQ-valgusanduri poolt mõõdetud allapoole suunatud kiirgusega paneeli asemel (ρ = π·L/E). Kui kaadris *on* kvaliteedikontrolli läbinud sihtmärk, muutub see vaikimisi absoluutseks võrdluspunktiks (`--reflectance-source auto`). Üks erand: „F988 peegeldusvõime kalibreeritakse stseeni sees oleva peegeldusvõime paneeli abil: sagedusala jääb väljapoole DAQ-valgusanduri kalibreeritud vahemikku, seega kasutab Chloros teie viimast paneeli salvestust ja säilitab seda paneeli vaatamiste vahel.” Vaata [Kalibreerimismärklaud](calibration-targets.md).

</details>

<details>

<summary>Kas mul on vaja DAQ-valgusandurit?</summary>

Kiirguse puhul mitte: LATTICE kiirguse ekspordid põhinevad iga kaamera tehase radiomeetrilisel kalibreerimisel ning ei vaja ei DAQ-andurit ega sihtmärki. **Peegeldusvõime**puhul on vaja ümbritseva valguse võrdluspunkti – kas DAQ-valgusanduri allapoole suunatud mõõtmist või kaadris olevat kalibreerimissihmärki. DAQ-anduri abil saate kalibreeritud peegeldusvõime väärtusi**ilma, et peaksite stseeni mingeid paneele paigutama**. Salvestatud `.daq`-failid sobitatakse teie piltidega automaatselt ajamärgise järgi. Vaadake [kalibreerimismärke](calibration-targets.md) ja [CLI viidet](reference/cli-reference.md).

</details>

<details>

<summary>Kas ma saan kasutada Chloros-i koos AI-abimehega (Claude, ChatGPT jne)?</summary>

Jah – käesolev kasutusjuhend ning CLI/SDK on selleks loodud:

* Kasutusjuhendi täielik sisukord on kättesaadav aadressil `https://mapir.gitbook.io/chloros/llms.txt`, et AI-abistajad saaksid leida iga lehekülje.
* Iga lehe algupärane Markdown-kood on kättesaadav vastava lehe väiketähelisel aadressil URL, millele on lisatud `.md` (näiteks `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI viide](reference/cli-reference.md) ja [SDK viide](reference/sdk-reference.md) on koostatud LLM-i jaoks: täpsed lipud, vaikimisi seaded, väljumissemantika ja kopeeritavad-kleebitavad käsud.

Vaata [AI-abistajad](ai-assistants.md), et teada saada, kuidas suunata oma abistaja Chloros-ile.

</details>

<details>

<summary>Kuhu salvestatakse minu töödeldud väljundfailid?</summary>

Tulemused salvestatakse projekti kausta alla, rühmitatuna kaamera järgi ja seejärel failivormingu järgi:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kaamera-kaust** — `LATT-<sensor>-<lens>-F<filter>` kaamera LATTICE jaoks, `<model>_<filter>` (nt `Survey3N_RGN`) kaamera Survey3 jaoks
* **vormingu-kaust** — `tiff16`, `tiff8`, `png8`, `jpg8` või `tiff32`
* **tootekausta** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (alati kaustas `tiff32`), `<INDEX>_Index_Images/`**Eksporditud failid säilitavad allikafaili nime — toodet identifitseerib kaust, mitte failinime laiend.**CLI puhul luuakse projektikaust sisendkausta kõrvale, kui te ei edasta parameetrit `-o`. Pange tähele, et `chloros-cli process` käivitus, mis taotles tooteid, kuid ühtegi ei kirjutanud, väljastab `Processing finished but wrote no image products.` ja**lõpetab töötamise nullist erineva väärtusega**, nii et skriptid suudavad seda tuvastada. Vaata [Väljundpildi formaadid](output-image-formats.md) ja [CLI-i viite](reference/cli-reference.md).

</details>

<details>

<summary>Kas ma saan oma pilte enne töötlemist Chloros-is redigeerida?</summary>

Ei. Chloros eeldab, et sisendandmeid ei ole muudetud. Ärge muutke failinimesid.

</details>

<details>

<summary>Kas ma saan seadistada oma MAPIR ja Survey3 kaamerad automaatsele säritusele ja töödelda pilte Chloros-is?</summary>

Ei. Survey3 pildikogumitel peab olema fikseeritud/lukustatud ekspositsioon, seega ei tohi kasutada automaatset säriaega ega automaatset ISO-väärtust. Kõikidel sama kaameramudeli piltidel peab olema identne säriaeg ja ISO (eksponatsioon).

LATTICE-kaameratel seda piirangut ei ole: Chloros juhib nende ekspositsiooni reaalajas (Smart AE) ning iga pildistamisel salvestatakse tegelikult kasutatud ekspositsioon ja võimendus, mida radiomeetriline töötlemisahel arvesse võtab.

</details>

<details>

<summary>Kas Chloros suudab töödelda või analüüsida ortomosaiikpilte?</summary>

Ei. Toetatakse ainult üksikuid MAPIR kaamera pilte, mitte ühendatud pilte, nagu ortomosaiikkaart.

</details>

<details>

<summary>Kuidas saan kiirendada Chloros-i sihtmärkide tuvastamise etappi?</summary>

Kui valite failibrauseri tabelis parempoolses veerus eelnevalt sihtmärgipildid, annab see Chloros-ile teada, et kalibreerimissihte tuleb otsida ainult nendest piltidest, mis kiirendab töötlemist märkimisväärselt.

</details>

<details>

<summary>Kui ma laadin oma pildid üles <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>i, kas peaksin neid enne üleslaadimist töötlema Chloros-is?</summary>

Kui kavatsete pilte üles laadida meie veebipõhisele töötlusplatvormile [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), siis ärge pilte enne üleslaadimist redigeerige. Cloud teostab kõik samad töötlustoimingud ja veelgi enam.

</details>

<details>

<summary>Kas MAPIR hakkab kunagi toetama funktsiooni X? Sooviksin väga, et MAPIR pakuks funktsiooni X.</summary>

Oleme alati huvitatud tagasisidest meie toodete kohta. Kui leiate meie toodetes vea või teil on ettepanekuid, kuidas saaksime oma tooteid parandada, palun [VÕTKE MEiega ÜHENDUST](https://www.mapir.camera/community/contact), et oma mõtteid jagada. Suurem osa meie uurimis- ja arendustööst lähtub klientide peamiste vajaduste kuulamisest.

</details>

<details>

<summary>Kas Chloros on saadaval Linux-ile?</summary>

Jah! Chloros 1.2.0 toetab Linux amd64 (x86_64) ja arm64 (NVIDIA Jetson JetPack 6) kaudu `.deb`-i pakettide kaudu. CLI ja Python SDK on Linux-il täielikult toetatud, sealhulgas LATTICE-kaamera ja DAQ-andurite reaalajas juhtimine. Linux-i jaoks puudub graafiline kasutajaliides — kogu suhtlus toimub [CLI](CLI.md) või [Python SDK](api-python-sdk.md) kaudu. Täpsema teabe leiate [Linux ülevaatest](linux/linux-overview.md).

</details>

<details>

<summary>Kas ma saan Chloros-i kasutada NVIDIA Jetsonil?</summary>

Jah! Chloros toetab NVIDIA Jetsoni platvorme, sealhulgas Jetson Nano, Orin Nano, Orin NX ja AGX Orin, millel töötab JetPack 6. Chloros tuvastab automaatselt teie Jetsoni mudeli ja optimeerib selle töötlemisstrateegiat. Seadistamise ja kasutuselevõtu juhised leiate [NVIDIA Jetsoni juhendist](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Kas Chloros optimeerib automaatselt minu riistvara jaoks?</summary>

Jah! Chloros sisaldab [dünaamilist arvutusvõimsuse kohandamist](processing-architecture/dynamic-compute-adaptation.md), mis tuvastab automaatselt teie CPU, GPU, RAM-i ja (Jetsonil) temperatuuriandurid. Seejärel valib see optimaalse töötlemisstrateegia – alates `GPU_PARALLEL`-ist suure mälumahuga süsteemides kuni `GPU_SINGLE`-ini piiratud ressurssidega seadmetes ja `CPU_PARALLEL`-ini süsteemides, millel puudub NVIDIA GPU. Käsitsi konfigureerimist ei ole vaja.

</details>

<details>

<summary>Mis on 4-niitline töötlemispipeline?</summary>

Chloros kasutab Chloros+ kasutajate jaoks 4-niitilist toruarhitektuuri: 1. niit (avastamine) laadib pildid ja tuvastab kalibreerimise sihtmärgid, 2. niit (kalibreerimine) arvutab peegelduskalibreeringu, 3. niit (töötlemine) teostab GPU-kiirendatud debayeringut ja indeksarvutust ning 4. niit (eksport) kirjutab väljundfailid. Maksimaalse läbilaskevõime saavutamiseks võivad mitu pilti olla samaaegselt erinevates niitides. Täpsema teabe saamiseks vaadake [Töötlemispipeliin](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Kuidas saan oma Chloros-installatsioonil diagnostikat käivitada?</summary>

Kasutage käsku `selftest`, et käivitada 7-etapiline kiirtest: versioon, pordi kättesaadavus, tagapõhja käivitamine, API ühenduvus (`/api/test`), süsteemiinfo (`/api/system-info` — GPU/CUDA/PyTorch), müra vähendaja mudeli olemasolu ning CUDA ja müra vähendaja valmisolek:

```bash
chloros-cli selftest
```

See on eriti kasulik Linux/Jetson-süsteemidel GPU ja CUDA seadistuse kontrollimiseks.

</details>
