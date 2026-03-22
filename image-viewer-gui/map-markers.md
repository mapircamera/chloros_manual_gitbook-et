# Kaardimärgid

Vahekaardil „Kaart“ kuvatakse teie pildid interaktiivsel 2D-kaardil nende GPS-koordinaatide alusel. See annab geograafilise ülevaate teie pildistamissessioonist ja aitab visualiseerida ruumilist katvust. See on kasulik ka piltide esmakordsel importimisel, et kiiresti eemaldada pildid, mida pole vaja töödelda.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Kaardi vahelehe avamine

1. Avage või looge projekt Chloros-is
2. Importige pildid, mis sisaldavad GPS-metadata
3. Klõpsake vasakul külgribal **Kaart** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> vasakul külgribal
4. Kaart kuvab märgid iga pildi GPS-asukohas

{% hint style="info" %}
**GPS on vajalik**: Kaardil kuvatakse ainult need pildid, mille EXIF-metadata sisaldab GPS-koordinaate. Veenduge, et pildistamise ajal on teie kaameras GPS sisse lülitatud.
{% endhint %}

***

## Piltide kohandamine kaardivahekaardilt**Kaart**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> on sama lisamise  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  ja eemaldamise  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  nupud nagu [**Failibrauser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Samuti kuvatakse seal sama projektifailide tabel, kuid teiste veerupealkirjadega:

### Failinimi

* Kaamera originaalfailinimi
* Säilitab kaamera nimetamiskonventsiooni (nt IMG\_0001.RAW)

### Laiuskraad

* Pildi laiuskraad

### Pikkuskraad

* Pildi pikkuskraad

### Kõrgus

* Pildi kõrgus

{% hint style="info" %}
Tabelite veerupealkirjade klõpsamine sorteerib ka ridade andmeid
{% endhint %}

***

## Pildi märgid

Iga GPS-andmetega pilt on kaardil esindatud märgiga:

### Märkide kuvamine

* Märgid näitavad täpseid GPS-koordinaate, kus iga pilt on tehtud
* Kui kaarti vähendada, võivad märgid rühmituda
* Suurendage kaarti, et näha iga pildi asukohta eraldi

{% hint style="success" %}
SUPER-ZOOM: Kui jõuate kaarditahvli pakkuja maksimaalse suumitasemeni, suurendatakse tahvlit edasise suumimise korral, võimaldades teil näha märgid, mis asuvad üksteise lähedal.
{% endhint %}

### Eelvaade hiirega üle libistamisel

* **Libistage hiirt** mis tahes märgi kohale, et näha selle pildi pisipilti
* See võimaldab kiiret visuaalset tuvastamist ilma kaardivaatest lahkumata
* Kasulik konkreetsete piltide leidmiseks suure pildistamissessiooni raames

***

## Kaarditahvlite pakkujad

{% hint style="success" %}
**Automaatne valik**: Chloros valib automaatselt kaarditahvli teenuse, mis pakub parimat suumitaset teie praeguse kaardiasukoha jaoks. Soovi korral saate teenusepakkujate vahel käsitsi vahetada.
{% endhint %}

Kaardi vahekaart toetab kahe teenusepakkuja taustakaardipilte:

### Google Maps

* Google&#x27;i standardne satelliit- ja kaardipilt
* Sobib kõige paremini üldiseks ülemaailmseks katvuseks

### ESRI

* ESRI ArcGIS-i satelliit- ja õhupildid
* Pakub teatud piirkondades sageli kõrgemahulisi pilte

***

## Kaarditüüpide valik

Võite valida kaardikihti tüübi (vasakult paremale):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Maastik

Näitab kõrgusprofiile ja kaardiplaate koos detailidega (teed jne)

### Kaart

Näitab standardseid (madalama ribalaiusega) kaardiplaate koos detailidega (teed jne)

### Satelliit

Näitab detailseid (kõrgema ribalaiusega) satelliitkaardiplaate

### Hübriid

Näitab satelliitkaardiplaate koos lisatud detailidega (teed jne)

***

## Kaardil navigeerimine

### Suumimise juhtnupud

* **Suumimine sisse/välja**: Kasutage hiire kerimisratast või suumimisnuppe
* **Täisekraan**: Kaardi kuvamine täisekraanil

### Paanimise juhtnupud

* **Paanimine**: Klõpsake ja lohistage, et kaardil liikuda***

## Kasutusjuhtumid

### Lennutrajektoori visualiseerimine

* Vaadake drooniga pildistamise seansside katvust
* Tuvastage lüngad pildikatvuses
* Kontrollige lennutrajektoori täitmist

### Maapinna uuringu ülevaatamine

* Vaadake maapinnal tehtud piltide ruumilist jaotust
* Leidke kalibreerimise sihtmärkide pildid suhtes uuringualaga
* Planeerige täiendavaid pildistamiskohti

### Kvaliteedikontroll

* Ootamatutes kohtades pildistatud piltide kiire tuvastamine
* GPS-i täpsuse kontrollimine kogu andmekogus
* Piltide asukohtade võrdlemine välitöö märkmetega

***

## Veaotsing

### Märgid ei kuvata

**Võimalikud põhjused:**

* Pildid ei sisalda GPS-i metaandmeid
* GPS oli pildistamise ajal kaameras välja lülitatud
* EXIF-andmed on välise tarkvara poolt eemaldatud

**Lahendus**: Kontrollige, kas GPS on kaameras sisse lülitatud, ja importige originaalfailid uuesti

### Märgised on vales asukohas

**Võimalikud põhjused:**

* Kaamera GPS-i satelliitide signaal oli nõrk
* GPS-i nihkumine pildistamise ajal

**Lahendus**: Tegu on tavaliselt pildistamise ajal tekkinud probleemiga; kaaluge täpsust nõudvate rakenduste puhul PPK/RTK GPS-i kasutamist
