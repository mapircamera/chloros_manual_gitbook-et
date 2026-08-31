# Graafiline kasutajaliides: navigeerimine

Kui käivitate Chloros-i esimest korda, käivitub selle töötlemisbackend. Kui backend on valmis, ilmub vasakule ülemisse nurka peamenüü ikoon <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> ning vasakus külgribas avanevad vahekaardid „Kaamerad” ja „Valgusandurid” (seni on need hallid).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Ülemine päis sisaldab vasakult paremale järgmisi elemente:

### Peamenüü „<img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">“

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Peamenüüst saate:

* **Uus projekt**— luua uue projekti. Kui olete salvestanud projektimalle, ilmub rippmenüü**Vali mall**, et uus projekt alustaks malli seadetest.
* **Ava projekt**— avada olemasolev projekt. Loendis on nupp**Ava projekti kaust**, mis avab projektide kausta teie failihalduris.
* **Kopeeri projekt** — kopeerib hetkel avatud projekti uue nimega (soovitatakse vaba nime, näiteks „MinuProjekt (2)”) ja avab koopia. _(nähtav pärast projekti avamist)_
* **Lisa faile** — lisa üksikuid pildifaile praegusesse projekti _(nähtav pärast projekti avamist)_
* **Lisa kaust** — lisa üks või mitu pildikaustu praegusesse projekti _(nähtav pärast projekti avamist)_
* **Töötlemise alustamine / peatamine** — alusta või peata pilditöötlusprotsess _(saadaval pärast failide lisamist)_
* **Ühenda kaameraga** — mine vahekaardile [Kaamerad](lattice/), et ühendada LATTICE-kaamera või -massiiv. Töötab ka ilma avatud projektita.
* **Ühenda valgusanduriga** — hüppa [Valgusandurite vahelehele](daq/), et ühendada DAQ-valgusandur. Töötab ka ilma avatud projektita.

{% hint style="info" %}
**Ainult Windows**: Chloros töölaua kasutajaliides on saadaval Windows-is. Linux-kasutajad peaksid tutvuma [CLI](CLI.md) ja [Python SDK](api-python-sdk.md) dokumentatsiooniga, mis käsitleb peata töötlemist.
{% endhint %}

###<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Esitus-/käivitusnupp

Kui see on aktiveeritud, käivitab töötlemise alustamise nupp pilditöötluse protsessi.

###<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Edusammude riba<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

Tasuta Chloros režiimis, mis töötleb kõiki faile järjest, kuvab edusammude riba kahte etappi: sihtmärgi tuvastamine ja töötlemine.

Tasulises Chloros+ litsentsitud režiimis, mis töötleb kõiki faile samaaegselt, näitab edenemisriba nelja etappi: tuvastamine, analüüsimine, kalibreerimine, eksportimine. Kui viite hiirekursori Chloros+ edenemisriba kohale, avaneb laiendatud neljaetapiline edenemisriba paneel, mille abil saate protsessi jälgida. Ülemise edenemisriba klõpsamine külmutab rippmenüü paneeli, uuesti klõpsamine vabastab selle.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Külgmenüü

Vasakpoolses külgmenüüs on erinevad ikoonid, millega saab suhelda, järgmises järjekorras ülevalt alla:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projekti seaded](project-settings/project-settings.md)

Vahekaardil „Projekti seaded“ saate kohandada projekti üldseadeid ja töötlemisseadeid. Reguleerige neid enne failide töötlemise alustamist.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Failibrauser

Lisage faile/kaustu ja eemaldage faile projektist. Duplikaatfaile ignoreeritakse. Märkige sihtpildi veerus asuv kastike ja töötlemisel vaadatakse sihtpiltide puhul ainult märgitud pilte, mis kiirendab oluliselt töötlemisaega. Kasutage lülitit „Pilt/Metadandid“, et vahetada valitud pildi pisipiltide võrgustiku ja üksikasjaliku metadatatabeli vahel.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Pildivaataja](image-viewer-gui/opening-an-image-full-screen.md)

Kui peamise pildivaataja aknas klõpsatakse pilti, avatakse see täisekraanil vahekaardil „Image Viewer“.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Kaardivaataja](image-viewer-gui/map-markers.md)

Vaadake oma pilte interaktiivsel 2D-kaardil nende GPS-koordinaatide alusel. Toetab Google Mapsi ja ESRI kaarditükkide pakkujaid, valides automaatselt teie asukohale parima teenuse. Viige hiir märkide kohale, et näha piltide pisipiltide eelvaateid.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kaamerad](lattice/)

Ühendage ja juhtige LATTICE-kaameraid reaalajas — ükshaaval või sünkroniseeritud mitme kaamera rühmadena. Sellel vahekaardil kuvatakse reaalajas eelvaatekaadrid koos kihtide ja histogrammidega, kaamera- ja süsteemipõhised seaded ning salvestusseaded, millega saab valida, milliseid kaameraid ja eksporditüüpe funktsioon „Capture All“ kasutab. Saadaval, kui tagapõhi on valmis; täieliku juhendi leiate [LATTICE-jaotisest](lattice/).

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Valgusandurid](daq/)

Ühendage DAQ-valgusandurid — DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet) — ning vaadake nende kalibreeritud spektridiagramme reaalajas ühikutes W/m²/nm. Siit saate salvestada `.daq`-faile avatud projekti, anda anduritele uued nimed, valida kap-korrektsiooniprofiile ja uuendada DAQ-E püsivara. Saadaval siis, kui tagapõhi on valmis; täieliku juhendi leiate [DAQ-jaotisest](daq/).

#### „<img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">“ vealogi

Kui tekib probleeme, vaadake logist veateateid. Kopeerige või laadige logi alla ja saatke see [MAPIR tugiteenistusele](https://www.mapir.camera/community/contact), et saada abi.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Kasutaja sisselogimine](chloros+-login.md)

Kasutaja sisselogimise külgriba võimaldab teil sisse logida oma Chloros+ kontole, et avada täiustatud funktsioonid. Samuti saate vaadata rakenduse praegust versiooni ning muuta Chloros kasutajaliidese ja CLI kuvatava teksti keelt.
