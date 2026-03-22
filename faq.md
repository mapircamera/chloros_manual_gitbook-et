---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# KKK

<details>

<summary>Kas ma saan Chloros-iga töödelda ka muude tootjate kaamerate pilte?</summary>

Ei, Chloros toetab ainult MAPIR kaamerate piltide töötlemist. Lisateabe saamiseks vaadake [toetatud kaameramudelite](supported-cameras.md) nimekirja. Pakume teiste kaamerate töötlemist MAPIR Cloudis, vaadake täielikku nimekirja [siit](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Kas ma saan kalibreerida oma pilte peegeldusvõime jaoks ilma kalibreerimismärgi abita?</summary>

Ei. Ilma kalibreerimismärgi pildita, mis on jäädvustatud samal ajal kui pildid, mis ei sisalda märki, ei ole võimalik seostada pildi pikselväärtusi teadaoleva peegeldusprotsendiga. Kui te ei lisa ka MAPIR valgussensori logi, siis ümbritseva valguse spektrit ei mõõdeta ja peegeldustulemused ei ole täpsed.

</details>

<details>

<summary>Kas ma saan oma pilte enne töötlemist Chloros-is redigeerida?</summary>

Ei. Chloros eeldab, et sisendandmeid ei ole muudetud. Ärge muutke failinimesid.

</details>

<details>

<summary>Kas ma saan seadistada oma MAPIR ja Survey3 kaamerad automaatsele säritusele ja töödelda pilte Chloros-is?</summary>

Ei. Survey3 pildikogumitel peab olema fikseeritud/lukustatud säri, seega ei tohi kasutada automaatset säriaega ega automaatset ISO-väärtust. Kõikidel sama kaameramudeli piltidel peab olema identne säriaeg ja ISO (säri).

</details>

<details>

<summary>Kas Chloros suudab töödelda või analüüsida ortomosaiikpilte?</summary>

Ei. Toetatakse ainult üksikuid MAPIR kaamerapilte, mitte ühendatud pilte, nagu ortomosaiikkaart.

</details>

<details>

<summary>Kuidas saan kiirendada Chloros sihtmärgi tuvastamise etappi?</summary>

Kui valite failibrauseri tabelis paremas veerus eelnevalt sihtmärgipildid, annab see Chloros-ile teada, et kalibreerimissihte tuleb otsida ainult nendest piltidest, mis kiirendab töötlemist märkimisväärselt.

</details>

<details>

<summary>Kui ma laadin oma pildid üles <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>i, kas ma peaksin neid enne üleslaadimist töötlema Chloros-is?</summary>

Kui kavatsete pilte üles laadida meie veebipõhisele töötlemisplatvormile [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), siis ärge pilte enne üleslaadimist redigeerige. Cloud teostab kõik samad töötlemistoimingud ja veelgi enam.

</details>

<details>

<summary>Kas MAPIR hakkab kunagi toetama X-funktsiooni? Sooviksin väga, et MAPIR pakuks X-funktsiooni.</summary>

Oleme alati huvitatud tagasisidest meie toodete kohta. Kui leiate meie toodetes probleemi või teil on ettepanekuid, kuidas saaksime oma tooteid parandada, palun [VÕTKE MEiega ÜHENDUST](https://www.mapir.camera/community/contact), et oma mõtteid jagada. Suurem osa meie uurimis- ja arendustööst lähtub klientide suurimate vajaduste kuulamisest.

</details>

<details>

<summary>Kas Chloros on saadaval Linux-ile?</summary>

Jah! Chloros 1.1.0 toetab Linux amd64 (x86_64) ja arm64 (NVIDIA Jetson JetPack 6) kaudu `.deb` pakette. CLI ja Python SDK on Linux-is täielikult toetatud. Linux-ile pole graafilist kasutajaliidest — kogu suhtlus toimub [CLI](CLI.md) või [Python SDK](api-python-sdk.md) kaudu. Täpsema info saamiseks vaadake [Linux ülevaadet](linux/linux-overview.md).

</details>

<details>

<summary>Kas ma saan Chloros-i kasutada NVIDIA Jetsonil?</summary>

Jah! Chloros 1.1.0 toetab NVIDIA Jetson platvorme, sealhulgas Jetson Nano, Orin Nano, Orin NX ja AGX Orin, millel töötab JetPack 6. Chloros tuvastab automaatselt teie Jetson-mudeli ja optimeerib selle töötlemisstrateegiat. Seadistamise ja kasutuselevõtu juhised leiate [NVIDIA Jetsoni juhendist](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Kas Chloros optimeerib automaatselt minu riistvara jaoks?</summary>

Jah! Chloros 1.1.0 sisaldab [dünaamilist arvutusvõimsuse kohandamist](processing-architecture/dynamic-compute-adaptation.md), mis tuvastab automaatselt teie CPU, GPU, RAM-i ja (Jetsonil) temperatuuriandurid. Seejärel valib see optimaalse töötlemisstrateegia – alates `GPU_PARALLEL`-ist suure mälumahuga süsteemidel kuni `GPU_SINGLE`-ini piiratud seadmetel ja `CPU_PARALLEL`-ini süsteemidel, millel puudub NVIDIA GPU. Käsitsi konfigureerimine ei ole vajalik.

</details>

<details>

<summary>Mis on 4-niitline töötlemispipeline?</summary>

Chloros 1.1.0 kasutab Chloros+ kasutajate jaoks 4-niitlist toruarhitektuuri: 1. niit (avastamine) laadib pildid ja tuvastab kalibreerimise sihtmärgid, 2. niit (kalibreerimine) arvutab peegelduskalibreeringu, 3. niit (töötlemine) teostab GPU-kiirendatud debayeringut ja indeksarvutust ning 4. niit (eksport) kirjutab väljundfailid. Maksimaalse läbilaskevõime saavutamiseks võib mitu pilti olla samaaegselt erinevates niitides. Täpsema info saamiseks vaadake [Töötlemispipeliin](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Kuidas saan oma Chloros installatsioonil diagnostikat käivitada?</summary>

Kasutage käsku `selftest`, et käivitada 7 süsteemidiagnostikat, sealhulgas versiooni kontroll, pordi kättesaadavus, tagapõhja käivitamine, API ühenduvus, süsteemiinfo, müra vähendamise mudelid ja CUDA kättesaadavus:

```bash
chloros-cli selftest
```

See on eriti kasulik Linux/Jetson süsteemidel, et kontrollida GPU ja CUDA seadistust.

</details>
