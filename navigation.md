# Kasutajaliides: navigeerimine

Kui käivitad Chloros ja Chloros (brauser) esimest korda, käivitub nende taustsüsteem. Kui see on valmis, ilmub vasakule ülesossa peamenüü ikoon <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Ülemine päis sisaldab vasakult paremale järgmist:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Peamenüü

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Peamenüüst saate:

* **Uus projekt** — luua uue projekti
* **Ava projekt** — avada olemasoleva projekti
* **Ava projektikausta** — avada projektikausta failibrauseris
* **Lisa faile** — lisada üksikuid pildifaile praegusesse projekti _(nähtav pärast projekti avamist)_
* **Lisa kaust** — lisada piltide kaust praegusesse projekti _(nähtav pärast projekti avamist)_
* **Alusta töötlemist / Peata töötlemine** — alustada või peatada piltide töötlemise protsessi _(aktiveeritud pärast failide lisamist)_

{% hint style="info" %}
**Ainult Windows**: Chloros töölaua kasutajaliides on saadaval Windows-is. Linux kasutajad peaksid nägema [CLI](CLI.md) ja [Python SDK](api-python-sdk.md) dokumentatsiooni peata töötlemise kohta.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Esita/Käivita nupp

Kui see on aktiveeritud, käivitab töötlemise käivitusnupp pilditöötlemise protsessi.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Edusammude riba <img src=".gitbook/assets/image (5).png" alt="" data-size="line">Tasuta režiimis Chloros, mis töötleb kõiki faile järjest, näitab edenemisriba kahte etappi: sihtmärgi tuvastamine ja töötlemine.

Tasulises litsentsitud režiimis Chloros+, mis töötleb kõiki faile samaaegselt, näitab edenemisriba nelja etappi: tuvastamine, analüüsimine, kalibreerimine, eksportimine. Kui viite hiirekursori Chloros+ edastusribale, avaneb laiendatud 4-etapiline edastusriba paneel, et saaksite protsessi jälgida. Ülemise edastusriba klõpsamine külmutab rippmenüü paneeli, uuesti klõpsamine vabastab selle.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Külgmenüü

Vasakul külgribal asuv menüü sisaldab mitmesuguseid ikoone, millega saab suhelda:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projekti seaded](project-settings/project-settings.md)

Projekti seaded-vahekaardil saate kohandada projekti üldisi ja töötlemise seadeid. Kohandage neid enne failide töötlemise alustamist.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Failibrauser

Lisage faile/kaustu ja eemaldage faile projektist. Duplikaatfaile ignoreeritakse. Märkige sihtpildi jaoks sihtveeru kastike ning töötlemine vaatab sihtpiltide puhul ainult märgitud pilte, kiirendades oluliselt töötlemisaega. Kasutage nuppu „Pilt/Metadata”, et vahetada valitud pildi pisipiltide rasteri ja üksikasjaliku metadatatabeli vahel.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Pildivaataja](image-viewer-gui/opening-an-image-full-screen.md)

Kui peamises pildivaatajas klõpsatakse pilti, avatakse see täisekraanil vahekaardil „Pildivaataja”.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Kaart](image-viewer-gui/map-markers.md)

Vaadake oma pilte interaktiivsel 2D-kaardil nende GPS-koordinaatide alusel. Toetab Google Mapsi ja ESRI kaarditükkide pakkujaid, valides automaatselt teie asukohale parima teenuse. Viige hiir märkide kohale, et näha piltide pisipiltide eelvaateid.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Veakoodi logi

Vaadake logi veakoodide jaoks, kui tekib probleeme. Kopeerige/laadige logi alla ja saatke see [MAPIR tugiteenistusele](https://www.mapir.camera/community/contact) abi saamiseks.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Kasutaja sisselogimine](chloros+-login.md)

Kasutaja sisselogimise külgribal saate sisse logida oma Chloros+ kontole, et avada täiustatud funktsioonid. Samuti saate vaadata rakenduse praegust versiooni ning muuta Chloros GUI ja CLI kuvatava teksti keelt.
