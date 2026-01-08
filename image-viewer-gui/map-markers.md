# Kaardimärgid

Kaardi vahekaardil kuvatakse teie pildid interaktiivsel 2D-kaardil nende GPS-koordinaatide alusel. See annab geograafilise ülevaate teie pildistamissessioonist ja aitab visualiseerida ruumilist katvust. See on kasulik ka piltide esmakordsel importimisel, et kiiresti eemaldada pildid, mida pole vaja töödelda.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Kaardi vahekaardi avamine

1. Avage või looge projekt Chloros-is.
2. Importige pildid, mis sisaldavad GPS-metadata.
3. Klõpsake vasakul külgribal **Kaart** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> vahekaarti vasakul külgribal
4. Kaardil kuvatakse märgid iga pildi GPS-asukoha juures

{% hint style="info" %}
**GPS vajalik**: Kaardil kuvatakse ainult need pildid, mille EXIF-metadata sisaldab GPS-koordinaate. Veenduge, et teie kaamera GPS on pildistamise ajal sisse lülitatud.
{% endhint %}

***

## Piltide kohandamine kaardi vahekaardilt

**Kaardi** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> kaardil on samad lisamise  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  ja eemaldamise  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  failide nuppe nagu [**Failibrauser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Samuti kuvatakse sama projektifailide tabel, kuid erinevate veeru pealkirjadega:

### Failinimi

* Kaamera originaalne failinimi
* Säilitab kaamera nimetamiskonventsiooni (nt IMG\_0001.RAW)

### Laiuskraad

* Pildi laiuskraad

### Pikkuskraad

* Pildi pikkuskraad

### Kõrgus

* Pildi kõrgus

{% hint style="info" %}
Tabelis veeru pealkirjade klõpsamine sorteerib ka rea andmed
{% endhint %}

***

## Pildi märgised

Iga GPS-andmetega pilt on kaardil esindatud märgisega:

### Märgise kuvamine

* Märgid näitavad täpsed GPS-koordinaadid, kus iga pilt on tehtud.
* Klastriteks koondunud märgid võivad väiksemal suumil kokku rühmituda.
* Suumige sisse, et näha üksikute piltide asukohti.

{% hint style="success" %}
SUPER-ZOOM: Kui jõuate kaardikilpide pakkuja maksimaalse suumitasemeni, suumitakse kilp edasi, võimaldades teil näha lähedal asuvaid märke.
{% endhint %}

### Eelvaade hiirega üle libistades

* **Libistage hiirega** mis tahes märgi üle, et näha selle pildi pisipilti.
* See võimaldab kiiret visuaalset identifitseerimist kaardivaatest lahkumata.
* Kasulik konkreetsete piltide leidmiseks suure pildistamissessiooni jooksul.

***

## Kaardikilpide pakkujad

{% hint style="success" %}
**Automaatne valik**: Chloros valib automaatselt kaardikilpide teenuse, mis pakub parimat suumitaset teie praeguse kaardikoha jaoks. Soovi korral saate teenusepakkujate vahel käsitsi vahetada.
{% endhint %}

Kaardi vahekaart toetab kahte kaardikilpide pakkujat taustakaardi piltide jaoks:

### Google Maps

* Google&#x27;i standardne satelliit- ja kaardipilt
* Parim üldise ülemaailmse katvuse jaoks

### ESRI

* ESRI ArcGIS satelliit- ja õhupildid
* Pakub tihti teatud piirkondades kõrgemate resolutsiooniga pilte

***

## Kaardikilpide tüübid

Võite valida kaardikihi tüübi (vasakult paremale):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="line">### Maastik

Näitab kõrgusprofiile ja kaardikahvleid detailidega (teed jne)

### Kaart

Näitab standardseid (madalama ribalaiusega) kaardikahvleid detailidega (teed jne)

### Satelliit

Näitab detailseid (kõrgema ribalaiusega) satelliitkaardikahvleid

### Hübriid

Näitab satelliitkaardikahvleid lisatud detailidega (teed jne)

***

## Kaardi navigeerimine

### Suumikontrollid

* **Suumimine/väljasuumimine**: kasutage hiire rullikku või suumiknuppe.
* **Täisekraan**: kaardi täisekraan.

### Panoraamikontrollid

* **Panoraam**: klõpsake ja lohistage, et kaardil liikuda.

***

## Kasutusjuhtumid

### Lennutrajektoori visualiseerimine

* Vaadake drooniga pildistatud alade katvust
* Leidke pildistatud alade katvuse lüngad
* Kontrollige lennutrajektoori täitmist

### Maapinna uuringu ülevaade

* Vaadake maapinnal pildistatud alade ruumilist jaotust
* Leidke kalibreerimise sihtmärgi pildid uuringualaga seotud kohtades
* Planeerige täiendavad pildistamiskohad

### Kvaliteedikontroll

* Ootamatutes kohtades pildistatud piltide kiire tuvastamine
* GPS-i täpsuse kontrollimine kogu andmekogus
* Piltide asukohtade ristviited välitöö märkmetega

***

## Probleemide lahendamine

### Märgistused ei kuvata

**Võimalikud põhjused:**

* Pildid ei sisalda GPS-i metaandmeid
* GPS oli pildistamise ajal kaameras välja lülitatud
* EXIF-andmed on eemaldatud välise tarkvara abil

**Lahendus**: Kontrollige, kas kaamera GPS on sisse lülitatud, ja importige originaalfailid uuesti.

### Märgid vales asukohas

**Võimalikud põhjused:**

* Kaamera GPS-il oli halb satelliidi signaal
* GPS-i kõrvalekalle pildistamise ajal

**Lahendus**: Tavaliselt on tegemist pildistamise ajal tekkinud probleemiga; kaaluge täpsete rakenduste jaoks PPK/RTK GPS-i kasutamist.
