# Graafiline kasutajaliides: navigeerimine

Kui käivitad Chloros'i esimest korda, alustab see oma töötlemisbackendi käivitamist. Kui backend on valmis, ilmub vasakule ülesossa peamenüü ikoon <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> ning vasakus külgribas avanevad vahekaardid „Kaamerad” ja „Valgusandurid” (seni on need hallid).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Ülemine päis sisaldab vasakult paremale järgmist:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Peamenüü

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Peamenüüst saate:

* **Uus projekt**— luua uue projekti. Kui olete salvestanud projektimalle, ilmub rippmenüü**Vali mall**, et uus projekt alustaks malli seadetest.
* **Ava projekt**— avada olemasolev projekt. Loendis on nupp**Ava projektikausta**, mis avab projektikausta teie failihalduris.
* **Kopeeri projekt** — kopeerib hetkel avatud projekti uue nimega (soovitatakse vaba nime, näiteks „MyProject (2)”) ja avab koopia. _(nähtav pärast projekti avamist)_
* **Lisa faile** — lisa üksikuid pildifaile praegusesse projekti _(nähtav pärast projekti avamist)_
* **Lisa kaust** — lisa üks või mitu pildikaustu praegusesse projekti _(nähtav pärast projekti avamist)_
* **Alusta töötlemist / Peata töötlemist** — alusta või peata pilditöötluse protsess _(saadaval pärast failide lisamist)_
* **Ühenda kaameraga** — mine [Kaamerad-vahelehele](lattice/), et ühendada LATTICE-kaamera või -maatriks. Töötab ka ilma avatud projektita.
* **Ühenda valgusanduriga** — hüppa [Valgusandurite vahelehele](daq/), et ühendada DAQ-valgusandur. Töötab ka ilma avatud projektita.

{% hint style="info" %}
**Ainult Windows**: Chloros i töölaua kasutajaliides on saadaval Windowsis. Linux i kasutajad peaksid tutvuma [CLI](CLI.md) ja [Python SDK](api-python-sdk.md) dokumentatsiooniga, et teostada töötlemist ilma graafilise kasutajaliideseta.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Esitus-/käivitusnupp

Kui see on aktiveeritud, käivitab töötlemise alustamise nupp pilditöötluse protsessi.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Edusammude riba <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



Tasuta Chloros-režiimis, mis töötleb kõiki faile järjestikku, kuvab edusammude riba kahte etappi: sihtmärgi tuvastamine ja töötlemine.

Tasulises Chloros+ litsentsitud režiimis, mis töötleb kõiki faile samaaegselt, kuvab edusammude riba nelja etappi: tuvastamine, analüüsimine, kalibreerimine, eksportimine. Kui viite hiirekursori „Chloros+“ edenemisriba kohale, avaneb laiendatud neljaosaline edenemisriba paneel, mille abil saate protsessi jälgida. Ülemise edenemisriba klõpsamine külmutab rippmenüü paneeli, uuesti klõpsamine vabastab selle.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Külgmenüü

Vasakpoolses külgmenüüs on mitmesugused ikoonid, millega saab suhelda, järgmises järjekorras ülevalt alla:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projekti seaded](project-settings/project-settings.md)

Vahekaardil „Projekti seaded“ saate kohandada projekti üldseadeid ja töötlemisseadeid. Reguleerige neid enne failide töötlemise alustamist.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Failibrauser

Lisage faile/kaustu ja eemaldage faile projektist. Duplikaatfaile ignoreeritakse. Märkige sihtpildi jaoks sihtveeru kastike ning töötlemisel vaadatakse sihtpiltide puhul ainult märgitud pilte, mis kiirendab oluliselt töötlemisaega. Kasutage lülitit „Pilt/Metadata“, et vahetada valitud pildi pisipiltide risti vaate ja üksikasjaliku metadatatabeli vahel.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Pildivaataja](image-viewer-gui/opening-an-image-full-screen.md)

Kui peamises pildivaatajas klõpsatakse pilti, avatakse see täisekraanil vahekaardil „Pildivaataja”.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Kaardivaataja](image-viewer-gui/map-markers.md)

Vaadake oma pilte interaktiivsel 2D-kaardil nende GPS-koordinaatide alusel. Toetab Google Mapsi ja ESRI kaarditükkide pakkujaid, valides automaatselt teie asukohale parima teenuse. Viige hiir märkide kohale, et näha piltide pisipiltide eelvaateid.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kaamerad](lattice/)

Ühendage ja juhtige LATTICE-kaameraid reaalajas — ükshaaval või sünkroniseeritud mitme kaamera rühmadena. Sellel vahekaardil kuvatakse reaalajas eelvaatekaadrid koos kihtide ja histogrammidega, kaamera- ja süsteemipõhised seaded ning salvestusseaded, millega saab valida, milliseid kaameraid ja eksporditüüpe funktsioon „Capture All“ kasutab. Saadaval siis, kui tagapõhi on valmis; täieliku juhendi leiate [LATTICE-jaotisest](lattice/).

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Valgusandurid](daq/)

Ühendage DAQ-valgusandurid — DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet) — ning vaadake nende kalibreeritud spektridiagramme reaalajas ühikutes W/m²/nm. Siit saate salvestada `.daq`-faile avatud projekti, andureid ümber nimetada, valida kap-korrektsiooniprofiile ja uuendada DAQ-E püsivara. Saadaval, kui tagapõhi on valmis; täieliku juhendi leiate [DAQ-jaotisest](daq/).

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Veakiri

Kui tekib probleeme, vaadake veakirjast veateateid. Kopeerige/laadige veakiri alla ja saatke see abi saamiseks [MAPIRi tugiteenistusele](https://www.mapir.camera/community/contact).

Uus versioonis 1.3.0: logi vahekaardi ülaosas asuv veaikoon (**Teata veast**) saadab MAPIRile aruande, mis sisaldab teie kirjeldust, kuni 10 pilti, Chloros-logisid ja süsteemi andmeid. Saamiseks peate olema sisse logitud. Vaata [Vea teatamine logi vahekaardilt](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Kasutaja sisselogimine](chloros+-login.md)

Kasutaja sisselogimise külgriba võimaldab teil sisse logida oma Chloros+ kontole, et avada täiustatud funktsioonid. Samuti saate vaadata rakenduse praegust versiooni ning kohandada Chlorosi graafilise kasutajaliidese ja CLI kuvatava teksti keelt.
