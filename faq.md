---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# KKK

<details>

<summary>Kas saan Chlorosega töödelda ka muude tootjate kui MAPIR kaamerate pilte?</summary>

Ei, Chloros toetab ainult MAPIR-kaamerate piltide töötlemist — Survey3- ja LATTICE-seeria kaameraid. Lisateabe saamiseks vaadake [toetatud kaameramudelite](supported-cameras.md) nimekirja. Pakume teiste kaamerate piltide töötlemist MAPIRi pilves, täieliku nimekirja leiate [siit](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Kas Chloros toetab LATTICE-kaameraid?</summary>

Jah. Chloros 1.3.0 toetab LATTICE M3C ja M3M kaameramooduleid terviklikult: **reaalajas juhtimine**— kaamerate avastamine, ühendamine, eelvaade ja pildistamine GUI-liidese kaamerate vahekaardilt, `chloros-cli lattice`, või Pythoni SDK kaudu, sealhulgas sünkroniseeritud mitme kaamera massiivid PTP-ajasünkroniseerimisega — ning**pildistatud materjali täielik radiomeetriline töötlemine** (toores → debayered → kiirgus → peegeldus → indeks). Vaata [Toetatud kaamerad](supported-cameras.md) ja [LATTICE juhendit](lattice/README.md).

</details>

<details>

<summary>Kas ma saan oma pilte peegeldusvõime järgi kalibreerida ilma kalibreerimismärgita?</summary>

**Survey3:** Ei. Ilma kalibreerimismärgi pildita, mis on jäädvustatud samal ajal kui muud pildid, ei ole võimalik seostada pildi pikselväärtusi teadaoleva peegeldusprotsendiga. Kui te ei lisa ka MAPIR-valgusanduri logi, siis ümbritseva valguse spektrit ei mõõdeta ja peegeldusvõime tulemused ei ole täpsed.

**LATTICE:** Jah. Peegeldusvõimet saab seostada DAQ-valgusanduri poolt mõõdetud allapoole suunatud kiirgusintensiivsusega paneeli asemel (ρ = π·L/E). Kui kaadris *on* olemas kvaliteedikontrolli läbinud sihtmärk, muutub see vaikimisi absoluutseks võrdlusaluseks (`--reflectance-source auto`), välja arvatud juhul, kui see läbib kontrolli lipuga, mil sihtmärk ja DAQ-ahelad konkureerivad mõõdetud tulemuse osas. Üks erand: „F988 peegeldusvõimet kalibreeritakse stseeni sees oleva peegeldusvõime paneeli abil: sagedusala jääb väljapoole DAQ-valgusanduri kalibreeritud vahemikku, seega kasutab Chloros i teie viimast paneeli salvestust ja säilitab seda paneeli vaatluste vahel.“ Vaata [Kalibreerimissihte](calibration-targets.md).

</details>

<details>

<summary>Kas mul on vaja DAQ-valgusandurit?</summary>

Kiirguse puhul mitte: LATTICE’i kiirguse ekspordid põhinevad iga kaamera tehase radiomeetrilisel kalibreerimisel ega vaja ei DAQ-andurit ega sihtmärki. **Peegeldusvõime**puhul on vaja ümbritseva valguse võrdlusväärtust – kas DAQ-valgusanduri allapoole suunatud mõõtmist või kaadris olevat kalibreerimissihtmärki. DAQ-anduri abil saate kalibreeritud peegeldusvõimet**ilma, et peaksite stseeni mingeid paneele paigutama**. Salvestatud `.daq`-failid sobitatakse teie piltidega automaatselt ajamärgise järgi. Vaadake [kalibreerimismärke](calibration-targets.md) ja [CLI-viidet](reference/cli-reference.md).

</details>

<details>

<summary>Kas ma saan kasutadChloros'i koos AI-assistendiga (Claude, ChatGPT jne)?</summary>

Jah — käesolev kasutusjuhend ning CLI ja SDK on selleks loodud:

* Kasutusjuhendi täielik sisukord on kättesaadav aadressil `https://mapir.gitbook.io/chloros/llms.txt`, et AI-assistendid saaksid leida iga lehekülje.
* Iga lehe toores Markdown-kood on kättesaadav selle väiketähelisel lehe URL-il, millele on lisatud `.md` (näiteks `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-viide](reference/cli-reference.md) ja [SDK-viide](reference/sdk-reference.md) on koostatud suurte keelemudelite (LLM) jaoks: täpsed lipud, vaikimisi seaded, väljumissemantika ja kopeeritavad-kleebitavad käsud.

Vaata [AI-assistendid](ai-assistants.md), et teada saada, kuidas suunata oma assistent Chlorose poole.

</details>

<details>

<summary>Kuhu salvestatakse minu töödeldud väljundfailid?</summary>

Tulemused salvestatakse projekti kausta alla, rühmitatuna kaamera ja seejärel failivormingu järgi:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kaamera-kaust** — `LATT-<sensor>-<lens>-F<filter>` (LATTICE), `<model>_<filter>` (nt `Survey3N_RGN`) (Survey3)
* **vormingu-kaust** — `tiff16`, `tiff8`, `png8`, `jpg8` või `tiff32`
* **tootekaustad** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (alati kaustas `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — ainult LATTICE multispektraalne: piksli-põhised maskid tootekaustade kõrval, vt allpool**Eksporditud failid säilitavad allikafaili nime — toodet identifitseerib kaust, mitte failinime laiend.**CLI-i kasutamisel luuakse projektikausta teie vaikimisi projektikausta alla (`Documents/MAPIR/Chloros Projects`, kui te seda pole muutnud), välja arvatud juhul, kui te edastate `-o`; suhteline `-o` võetakse suhteliselt kausta suhtes, kust käsk käivitatakse. Pange tähele, et käsk `chloros-cli process`, mis taotles tooteid, kuid ühtegi ei kirjutanud, väljastab `Processing finished but wrote no image products.` ja**lõpetab töötamise nullist erineva väärtusega**, nii et skriptid saavad seda tuvastada. Vaata [Väljundpildi formaadid](output-image-formats.md) ja [CLI-viite](reference/cli-reference.md).

</details>

<details>

<summary>Mis on clamp_masks-kaustad ja *_clampmask.tif-failid?</summary>

Uus versioonis 1.3.0. Multispektraalse LATTICE M3C kaamera kiirguse ja peegeldusvõime tooted läbivad 3×3-ribalise segunemise eemaldamise ning kui mõnes ribas tekib negatiivne väärtus, kärbib Chloros selle väärtuse nullini. Toode, millel on vähemalt üks kärbitud piksel, saab ühekanalilise `uint8`-maski, mis märgib, millised pikslid ja kanalid kärbiti (bit 0 = toote R-kanal, bit 1 = G, bit 2 = B). Maskid salvestatakse `clamp_masks/`-kausta **kõrval** tootekastidesse — näiteks `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — ning toote `Chloros:UnmixClampMask` XMP-märge nimetab selle maski (või märgib `none`).

Need ei ole stseeni pildid: Chlorosi enda importimisfunktsioon jätab need vahele ning kuna need ei asu kunagi tootekausta sees, võite tootekausta edastada Pix4D-le või Metashape’ile sellisena, nagu see on. Vaadake [Clamp-maske](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Miks Chloros teatab: „Projektikausta asukoht on selle arvuti jaoks liiga sügav“?</summary>

See kehtib ainult Windowsi puhul. Windowsis on pikad teed vaikimisi välja lülitatud ning seetõttu ei tohi ükski failitee olla pikem kui 260 tähemärki. Chlorosi tooted asuvad kuni umbes 130 tähemärki projektikausta allpool, seega keeldub Chloros enne töötlemist sügavamat projektikausta – selle asemel, et käivituda ja midagi kirjutamata jätta, nagu võis juhtuda enne versiooni 1.3.0. Teade näitab pikimat sobivat projektikausta.

Parandage seda ühel kahest viisist:

* Kasutage lühemat projektikausta (**Töökataloog** projekti seadetest või `-o` / `chloros-cli set-project-folder` CLI-s).
* Lülita sisse Windowsi pikad teed ja käivita Chloros uuesti. Chlorosi installija valik **Enable Windows long paths** (vaikimisi märgitud) teeb seda; käivita installija uuesti, kui oled selle märgistuse eemaldanud.

Vaata [Töötlemise alustamine](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Kas ma saan oma pilte enne töötlemist Chloroses redigeerida?</summary>

Ei. Chloros eeldab, et sisendandmeid ei ole muudetud. Ärge muutke failinimesid.

</details>

<details>

<summary>Kas ma saan seadistada oma MAPIRi Survey3-kaamerad automaatsele säritusele ja töödelda pilte Chloroses?</summary>

Ei. Survey3 pildikogumitel peab olema fikseeritud/lukustatud säri, seega ei tohi kasutada automaatset säriaega ega automaatset ISO-väärtust. Kõikidel sama kaameramudeli piltidel peab olema identne säriaeg ja ISO (säri).

LATTICE-kaameratel seda piirangut ei ole: Chloros juhib nende säritust reaalajas (Smart AE) ning iga pildistamisel salvestatakse tegelikult kasutatud säritus ja võimendus, mida radiomeetriline töötlusarvutus arvesse võtab.

</details>

<details>

<summary>Kas Chlorosega saab töödelda või analüüsida ortomoosaikpilte?</summary>

Ei. Toetatakse ainult üksikuid MAPIR-kaamera pilte, mitte ühendatud pilte, nagu ortomoosaikkaart.

</details>

<details>

<summary>Kuidas saan kiirendadChlorosi sihtmärkide tuvastamise etappi?</summary>

Kui valite failibrauseri tabelis paremas veerus eelnevalt sihtmärgipildid, annab see Chlorosele teada, et kalibreerimissihte tuleb otsida ainult nendest piltidest, mis kiirendab töötlemist märkimisväärselt.

</details>

<details>

<summary>Kui ma laadin oma pildid üles <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR</a>i<a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription"> pilve</a>, kas peaksin neid enne üleslaadimist Chloroses töötlema?</summary>

Kui kavatsete pilte üles laadida meie veebipõhisele töötlemisplatvormile [MAPIR i pilv](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), ärge pilte enne üleslaadimist redigeerige. Pilv teostab kogu sama töötlemise ja veelgi enam.

</details>

<details>

<summary>Kas MAPIR hakkab kunagi toetama X-funktsiooni? Sooviksin väga, et MAPIR pakuks X-funktsiooni.</summary>

Oleme alati huvitatud tagasisidest meie toodete kohta. Kui leiate meie toodetes mõne probleemi või teil on ettepanekuid, kuidas saaksime oma tooteid parandada, palun [VÕTKE MEiega ÜHENDUST](https://www.mapir.camera/community/contact), et oma mõtteid jagada. Suurem osa meie uurimis- ja arendustööst lähtub klientide suurimate vajaduste kuulamisest.

</details>

<details>

<summary>Kas Chloros on saadaval Linuxile?</summary>

Jah! Chloros 1.3.0 toetab Linuxi amd64 (x86_64) ja arm64 (NVIDIA Jetson JetPack 6) platvorme `.deb` pakettide kaudu. CLI ja Python SDK on Linuxis täielikult toetatud, sealhulgas LATTICE-kaamera ja DAQ-andurite reaalajas juhtimine. Linuxi jaoks ei ole graafilist kasutajaliidest – kogu suhtlus toimub [CLI](CLI.md) või [Python SDK](api-python-sdk.md) kaudu. Täpsema teabe saamiseks vaadake [Linuxi ülevaadet](linux/linux-overview.md).

</details>

<details>

<summary>Kas ma saan kasutadChloros'i NVIDIA Jetsonil?</summary>

Jah! Chloros toetab NVIDIA Jetsoni **Orin-seeria** seadmeid — Orin Nano, Orin NX ja AGX Orin —, millel töötab JetPack 6. Vanemad mudelid (Jetson Nano, TX2, Xavier NX) ei toeta JetPack 6-t ja neid ei toeta praegune pakett. Chloros tuvastab automaatselt teie Jetson-mudeli ja optimeerib selle töötlemisstrateegiat. Jetsoni pakett kasutab JetPack 6 CUDA-põhist PyTorchi versiooni, mistõttu töötlemine toimub Jetsoni GPU-l. Seadistamise ja kasutuselevõtu juhiseid leiate [NVIDIA Jetsoni juhendist](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Kas Chloros optimeerib automaatselt minu riistvara jaoks?</summary>

Jah! Chloros sisaldab [dünaamilist arvutusvõimsuse kohandamist](processing-architecture/dynamic-compute-adaptation.md), mis tuvastab automaatselt teie CPU, GPU, RAM-i ja (Jetsonil) temperatuuriandurid. Seejärel valib see optimaalse töötlemisstrateegia — alates `GPU_PARALLEL`-ist suure mälumahuga GPU-de puhul kuni `GPU_SINGLE`-ini väiksemate puhul ning `CPU_PARALLEL`-ini süsteemides, kus puudub CUDA GPU, mida PyTorch saaks kasutada. Käsitsi konfigureerimist ei ole vaja.

</details>

<details>

<summary>Mis on 4-niitline töötlemispipeline?</summary>

Chloros kasutab Chloros+ kasutajate jaoks 4-niitlist toruarhitektuuri: 1. niit (avastamine) laadib pildid ja tuvastab kalibreerimise sihtmärgid, 2. niit (kalibreerimine) arvutab peegelduskalibreeringu, 3. niit (töötlemine) teostab GPU-kiirendatud debayeringut ja indeksi arvutamist ning 4. niit (eksport) kirjutab väljundfailid. Maksimaalse läbilaskevõime saavutamiseks võivad mitu pilti olla samaaegselt erinevates niitides. Täpsemad üksikasjad leiate jaotisest [Töötlemispipeline](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Kuidas saan oma Chloros installatsiooni diagnostikat käivitada?</summary>

Kasutage käsku `selftest`, et käivitada 7-etapiline kiirtest: versioon, portide kättesaadavus, tagapõhja käivitamine, API-ühendus (`/api/test`), süsteemiinfo (`/api/system-info` — GPU/CUDA/PyTorch), müra vähendaja mudeli olemasolu ning CUDA ja müra vähendaja valmisolek:

```bash
chloros-cli selftest
```

See on eriti kasulik Linux-süsteemides GPU ja CUDA seadistuse kontrollimiseks. Jetsonil tähendab `CUDA: False` tavaliselt, et JetPack 6 CUDA ja cuDNN raamatukogud puuduvad: paigalda need käsuga `sudo apt install nvidia-jetpack`.

</details>
