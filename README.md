---
metaLinks: {}
---

# Alustamine

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros on [MAPIR](https://www.mapir.camera) tarkvararakendus piltide ja muude andurite andmete töötlemiseks.

***{% hint style="success" %}**Uued funktsioonid versioonis Chloros 1.1.0**: Linux-i natiivne tugi (amd64 ja arm64), NVIDIA Jetson servaarvutus, dünaamiline arvutusadapteerimine, 4-niitline töötlemispipeline, uued CLI-i käsud ja valikud. Vaata [Allalaadimine](download.md) täieliku muudatuste loendi jaoks.
{% endhint %}

Chloros on saadaval 3 rakendusrežiimis:

## Chloros: Töölaua GUI-rakendus

Eraldi aken kõigi funktsioonidega. _Ainult Windows._

## [Chloros CLI: Käsurealiides](CLI.md)

Käsurea kaudu toimuv pakettide töötlemine. Sobib ideaalselt automatiseerimiseks, skriptide kirjutamiseks ja peata tööks. Saadaval **Windows, Linux amd64 ja Linux arm64 (NVIDIA Jetson)**. _CLI-le juurdepääsuks on vaja Chloros+ litsentsi._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmeeritav Python liides automatiseerimiseks ja kohandatud töövoogudeks. Sobib ideaalselt uurimistöö protsessidele, integreerimiseks olemasolevate Python rakendustega ja kohandatud tööriistade loomiseks. Saadaval **kõikidel platvormidel** `pip install chloros-sdk` kaudu. _API-le juurdepääsuks on vaja Chloros+ litsentsi._***

## Toetatud platvormid

| Platvorm | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Jah | Jah | Jah |
| **Linux amd64 (x86_64)** | Ei | Jah | Jah |
| **Linux arm64 (NVIDIA Jetson)** | Ei | Jah | Jah |

Linux paigaldusjuhiseid vaadake jaotisest [Linux ja servaarvutus](linux/linux-overview.md).

***

## Chloros+

Kuigi Chloros on enamiku ülesannete jaoks tasuta, võite avastada, et soovite rohkem. Siis võib Chloros+ tasuline litsents teile kasuks tulla. Chloros+ litsentsiga saate avada uusi funktsioone, nagu:

* **Mitmeprotsessiline töötlemine**: kiirendage oluliselt suuremate projektide pilditöötlust, töötledes pilte töötlusketis samaaegselt.
* **GPU (CUDA) kiirendus**: kasutage ära tänapäeva suurema GPU mälu mahtu, et veelgi kiirendada pilditöötluse töötlemisprotsessi. Parimate tulemuste saavutamiseks soovitame 4 GB või rohkem VRAM-i.
* **Chloros+**[**CLI**](CLI.md)**Juurdepääs**: käivitage Chloros+ käsurealt, et automatiseerida ja integreerida see oma tarkvarasse.
* **Chloros+**[**API**](api-python-sdk.md)**Juurdepääs:** käivitage Chloros+ programmist Python, et saavutada programmiline kontroll, mis võimaldab sujuvat integreerimist teie uurimisprotsesside, andmete analüüsi töövoogude ja kohandatud rakendustega.
* **Mitme seadme kasutamine**: iga Chloros+ litsents võimaldab registreerida 2+ seadet. Kasutage oma MAPIR pilvekontot registreeritud seadmete haldamiseks. Lisage tugi rohkematele seadmetele, uuendades oma Chloros+ litsentsi.
* **Täiustatud tekstuuritundlik debayer-meetod:** kõrgekvaliteediline servatundlik debayer, mis on kombineeritud AI/ML müra vähendamise mudeliga, mis eemaldab peaaegu kogu debayering-müra. 
* **Kohandatud multispektraalsed indeksivalemid:** sisestage kohandatud multispektraalsed indeksid Chloros rasterarvutitesse, nii töötlemiseks kui ka pildi vaatamise sandboxi jaoks.
* **Linux ja servarvutamine:** käivitage Chloros platvormidel Linux x86\_64 ja ARM64, sealhulgas NVIDIA Jetson, välitöötluseks ja servatöötluseks. Vaata [Linux ülevaadet](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ hinnad ja registreerimine</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
