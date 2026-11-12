# Kaardimärgid

Vahekaart „Kaart“ kuvab teie pildid interaktiivsel 2D-kaardil nende GPS-koordinaatide alusel. See annab teile pildistamissessiooni geograafilise ülevaate ja on kohe pärast importimist kiireim viis eemaldada pildid, mida te töödelda ei soovi.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Kaardi vahelehe avamine

1. Avage või looge projekt Chloros-is
2. Importige pildid, mis sisaldavad GPS-metadata
3. Klõpsake vasakul külgribal **Kaart**-<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line">-vahelehel
4. Kaardil kuvatakse märk iga pildi GPS-asukohas

{% hint style="info" %}
**GPS on vajalik**: kaardil kuvatakse ainult need pildid, mille EXIF-metandmetes on GPS-koordinaadid. Koordinaatideta pilt jääb ikkagi projekti ja töödeldakse tavapäraselt – sellel lihtsalt puudub märk.
{% endhint %}

***

## Piltide kohandamine vahekaardil „Kaart“

Vahekaardil **„Kaart“**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> on samad nuppud „Lisa“ <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> ja „Eemalda“ <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> nagu vahekaardil [**„Failibrauser“**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Seal kuvatakse sama projektifailide nimekiri koos geograafiliste veergudega:

| Veerg        | Sisu                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Nimi**      | Failinimi sellisena, nagu see kaamerast saadi                             |
| **Laiuskraad**  | Kümnendkraadid, kuus kümnendkohta                                |
| **Pikkuskraad** | Desimaalkraadid, kuus kümnendkohta                                |
| **Kõrgus**  | Meetrid, üks kümnendkoht — `-`, kui pildil puudub kõrgus |

{% hint style="info" %}
Klõpsake mis tahes veeru pealkirjal, et seda järgi sorteerida; klõpsake uuesti, et järjestust pöörata.
{% endhint %}

{% hint style="warning" %}
**Kõrgus on kõrgus merepinnast, mitte maapinnast.** Väärtus pärineb pildi EXIF-märgistusest `GPSAltitude`, mis viitab keskmisele merepinnale. See ei ole lennukõrgus maapinna suhtes ning Chloros ei arvuta selle põhjal maapinna proovivahekaugust – 300 m merepinnast kõrgemal asuva põllu kohal registreerib 100 m AGL kõrgusel lendav droon siin ligikaudu 400 m. Kasutage seda veergu hälvete tuvastamiseks ja lennukõrguse järjepidevuse kontrollimiseks, mitte aga AGL-mõõtmiseks.
{% endhint %}

***

## Pildimärgised

Iga GPS-andmetega pilt saab oma koordinaatidele märgise.

### Märkide kuvamine

* Märgid asuvad iga pildi jaoks salvestatud täpsetel koordinaatidel
* Lähedal asuvad märgid võivad väiksema suumiga visuaalselt üksteisega kattuda – suumige sisse, et neid eraldada
* Valitud ja esile tõstetud märgid kuvatakse teiste kohal

### Eelvaade hiirega üle libistamisel

* **Liiguta hiirt** mis tahes märgi kohale, et avada selle pildi pisipilt koos failinimega
* **Klõpsake**märgile, et valida pilt ja**kinnitada** hüpikaken avatuks — see jääb avatuks, kuni klõpsate mujale. Kui hüpikaken on kinnitatud, ei võta teiste märkide peale viimine seda ära
* See on kiire viis leida üks konkreetne kaader suures sessioonis, ilma kaardilt lahkumata

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Vahekaardil „Kaart“ kuvatakse kõik projekti geotagitud pildid</p></figcaption></figure>### Super-suum

{% hint style="success" %}
**SUPER-SUUM**: kui jõuad maksimaalsele suumitasemele, mille jaoks kaarditükkide pakkujal on pildimaterjali olemas, suurendab edasine suumimine kaarditükke edasi, selle asemel et peatuda, nii et saad eristada märgiseid, mis asuvad peaaegu üksteise peal.
{% endhint %}

* Super-zoom aktiveerub ainult siis, kui olete **jõudnud** teenusepakkuja poolt selle asukoha jaoks määratud maksimaalse suumini ja kaarditükid on lõpetanud laadimise. Sellest madalamal tasemel toimib suumimine tavapäraselt
* Suumivahemik on **1× kuni 32×** teenusepakkuja enda maksimaalse suumi peal
* Nurgas asuv indikaator näitab praegust superzoomi protsendina ning selle kõrval olev **×**-nupp viib ühe klõpsuga tagasi tavalise suumimise juurde
* Väljasuumimine toimub alati kaardil endal, seega ei saa te kunagi superzoomi juurde kinni jääda
* Suurendamise ja nihutamise ajal superzoomis kantakse tekkinud nihke kaardile üle, nii et keskpunktist eemale nihutatud ala jätkab ruutude laadimist, mitte ei muutu tühjaks
* Märgid joonistatakse vektorkujul, mitte rasterdatuna, mistõttu jäävad need teravaks igal superzoomi tasemel

***

## Kaardiplaadi pakkujad

{% hint style="success" %}
**Automaatne valik**: Chloros valib selle plaaditeenuse, mis pakub parimat suumitaset sinu piltide asukoha jaoks. Saad igal ajal käsitsi vahetada.
{% endhint %}

| Pakkuja        | Märkused                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Laiaulatuslik ülemaailmne katvus; toetab kõiki nelja kaarditüüpi                                                                                                            |
| **Esri ArcGIS**| Sageli kõrgem resolutsiooniga õhufotod teatud piirkondades. Kaarditüüp**Terrain** ei ole Esri jaoks saadaval ja selle nupp on keelatud, kui Esri on valitud |

***

## Kaardiplaadiliigid

Valige kaardikihti tüüp nuppude abil (vasakult paremale):

![](<../.gitbook/assets/image (14).png>)

| Tüüp                 | Näitab                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Maastik**          | Kõrguste varjutus koos kaardidetailidega (teed, sildid). Ainult Google       |
| **Kaart**              | Standardse tänavakaardi plaadid — madalaima ribalaiusega valik              |
| **Satelliit**        | Detailne satelliidipilt, ilma siltideta — suurima andmeedastuskiirusega valik |
| **Hübriid** (vaikimisi) | Satelliidipilt, millele on joonistatud teed ja sildid                |

Kaardi vahekaart avatakse **Hübriid**-režiimis. Teie valik mõjutab teenusepakkuja vahetust, kui teenusepakkuja seda toetab.

***

## Kaardil navigeerimine

* **Suurendamine**: hiire kerimisratas või kaardil olevad suurendamisnupud
* **Liikumine**: klõpsake ja lohistage
* **Täisekraan**: täisekraani nupp laiendab kaarti kogu akna suuruseks***

## Kasutusjuhtumid

### Lennutrajektoori ülevaatamine

* Vaata ühe pilguheitega droonilennu katteala
* Leia lüngad, kus mõni läbimine jäi vahele
* Veendu, et lend järgis planeeritud marsruuti

### Maapinna uuringu läbivaatamine

* Vaata, kuidas on jaotunud maapinnalt tehtud pildid
* Leia kalibreerimise sihtmärkide raamid uuringuala suhtes
* Otsusta, kus on vaja täiendavaid pilte

### Kvaliteedikontroll

* Leia ootamatutes kohtades pildistatud kaadrid ja eemalda need enne töötlemist
* Sorteeri kõrguse järgi, et leida valel kõrgusel pildistatud kaadrid või need, kus GPS-signaal oli nõrk
* Võrdle piltide asukohti välitöö märkmetega

***

## Veaotsing

### Märgiseid ei kuvata

**Võimalikud põhjused**

* Piltidel puuduvad GPS-metandmed
* Pildistamise ajal oli kaameras GPS välja lülitatud
* EXIF-andmed eemaldati enne importimist mõne teise tarkvara abil

**Mida teha**: veendu, et kaameras on GPS sisse lülitatud, ja impordi originaalfailid uuesti. Saad kontrollida, kas konkreetsel failil on koordinaadid, otsides seda vahekaardi „Kaart” failitabelist – pildil, millel koordinaate pole, puudub seal rida.

### Märgid asuvad vales kohas

**Võimalikud põhjused**: halb satelliitpositsioon pildistamise ajal või GPS-i hälve seansi jooksul.**Mida teha**: tegemist on pildistamise ajal tekkinud probleemiga, mida Chloros ei saa tagantjärele parandada. Täpseks tööks kasutage PPK/RTK GPS-töövoogu – vaadake seadet**Kohalda PPK-parandusi** [Projekti seaded](../project-settings/project-settings.md).

### Kaart on tühi või kaarditükkide laadimine peatub

Kaartiplaadipakkujad on veebipõhised teenused. Kui kaartiplaadid ei lae enam, kontrollige seadme võrguühendust ja proovige seejärel vahetada pakkujat. Kui olite väga suurelt suumitud, vajutage **×** taastamisnuppu, et naasta tavalisele suumitasemele ja lasta kaardil kaartiplaadid uuesti taotleda.

***

## Seotud leheküljed

* [**Pildivõrk**](image-grid.md) — sama pildikogum kui pisipildid
* [**Pildi avamine täisekraanil**](opening-an-image-full-screen.md) — ühe pildi üksikasjalik uurimine
* [**Failide lisamine projektile**](../processing-images-gui/adding-files-to-a-project.md) — selle vahekaardi failide lisamise ja eemaldamise nupud
