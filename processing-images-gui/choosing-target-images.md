# Sihtpiltide valimine

Kui märgite, millised pildid sisaldavad kalibreerimissihte, annab see programmile „Chloros” täpse teabe, kust neid otsida. Kui veerus „Target” on märgitud vähemalt üks pilt, skaneerib „Chloros” **ainult märgitud pilte** – seega kiirendab sihtide märkimine töötlemist ja aitab vältida, et uuringupilte ekslikult sihtideks peetaks.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Miks märgistada sihtmärgipilte?

### Märgistamine juhib skaneerimist

Kui märgistate veerus „Sihtmärk“ konkreetsed pildid:

* Chloros skaneerib sihtmärke otsides ainult märgistatud pilte
* Sihtmärkide tuvastamine toimub palju kiiremini
* Uuringupildid ei saa põhjustada valesid sihtmärkide tuvastamisi

Kui **ühtegi** pilti ei ole märgitud, skannib „Chloros“ vaikimisi kõiki projekti pilte:

* Sihtmärkide tuvastamise algoritmid töötavad iga pildi puhul
* Sadu või tuhandeid pilte kontrollitakse asjatult
* Töötlemine võtab märkimisväärselt kauem aega, eriti suurte andmekogumite puhul

{% hint style="success" %}
**Kiiruse parandamine**: 500 pildist koosnevas andmekogus 2–3 sihtmärgi pildi märkimine võib lühendada sihtmärgi tuvastamise aega 30+ minutilt alla 1 minuti.
{% endhint %}

***

## Kuidas märgistada sihtmärgi pilte

### 1. samm: Sihtmärgi piltide tuvastamine

Vaadake failibrauseris läbi imporditud pildid ja tuvastage, millised pildid sisaldavad kalibreerimise sihtmärke.

**Tavalised stsenaariumid:*** **Enne pildistamist märgistatud sihtmärk**: Märgistatud enne seansi algust
* **Pärast pildistamist pildistatud sihtmärk**: pildistatud pärast seansi lõppu
* **Välitingimustes asuvad sihtmärgid**: pildistamispiirkonnas paigutatud sihtmärgid
* **Mitmed sihtmärgid**: 2–3 sihtmärgi pilti seansi kohta (soovitatav)

### 2. samm: Kontrollige veergu „Target” (<img src="../.gitbook/assets/image (33).png" alt="" data-size="original">)

Iga kalibreerimise sihtmärki sisaldava pildi puhul:

1. Leidke pilt failibrauseri tabelis
2. Leidke veerg **Target** (kõige parempoolsem veerg)
3. Klõpsake selle pildi puhul veerus „Target” asuvat märkeruutu
4. Korda seda kõigi sihtmärke sisaldavate piltide puhul

### 3. samm: Kontrolli oma valikut

Enne töötlemist kontrolli veel kord:

* [ ] Kõik kalibreerimissihtmärke sisaldavad pildid on märgitud
* [ ] Ühtegi sihtmärki mittesisaldavat pilti ei ole kogemata märgitud
* [ ] Sihtmärgid on märgitud piltidel selgesti nähtavad

***

## LATTICE: sihtmärgid on valikulised, kui DAQ salvestab

LATTICE multispektraalsete kaamerate puhul on kaadris olev kalibreerimissihtmärk **üks kahest** võimalikust peegeldusvõime võrdluspunktist:

* **Kaadris olev sihtmärk**: kui märgitud sihtmärgi pilt läbib „Chloros” kvaliteedikontrolli (QA), muutub sihtmärk ümbritseva pildi**absoluutseks peegeldusvõime võrdluspunktiks**.
* **DAQ-i allapoole suunatud kiirgus**: kui sihtmärki pole (või kvaliteedikontroll ebaõnnestub), arvutab Chloros peegeldusvõime hoopis DAQ-i valgusanduri allapoole suunatud kiirguse põhjal (ρ = π·L/E). Kui teie salvestusi hõlmab `.daq` või DAQ-M `.csv` salvestus, saate kalibreeritud peegeldusvõime**ilma ühegi sihtmärgi pildita**.

See automaatne käitumine on vaikimisi sisse lülitatud. CLIis / SDK vastab see `--reflectance-source auto`-ile; võite ka sundida kasutama `target`-i (rangelt — ilma DAQ-asendamiseta) või `daq`-i (DAQ-põhine). Vaata [CLI viidet](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**LATTICE sihtmärkide geomeetriad**: lisaks klassikalisele paneelidetekteerimisele, mida kasutatakse Survey3 puhul, toetab LATTICE-töötlus**ArUco-märgistatud sihtmärke**,**fikseeritud ROI-sihtmärke**ja**ribasihtmärke**, mis konfigureeritakse projekti kaupa. Ühiku kohta**mõõdetud** sihtmärgi peegeldusvõime skaneeringuid saab esitada seerianumbri järgi (CLI: `--target-reflectance-dir`, üks `<serial>.csv` iga sihtmärgiühiku kohta), kusjuures varuvariandina kasutatakse nominaalseid T3/T4P spektreid.

{% hint style="info" %}
**F988-moodul**: F988 peegeldusvõimet kalibreeritakse stseeni sees oleva peegeldusvõime paneeli abil: sagedusriba jääb väljapoole DAQ-valgusanduri kalibreeritud vahemikku, mistõttu rakendus „Chloros“ kasutab teie viimast paneeli salvestust ja säilitab selle paneeli vaatluste vahel. Kui F988-moodulit töödeldakse ainult DAQ-andmete põhjal, lükkab „Chloros” selle lainepikkuse puhul DAQ-põhise peegeldusvõime tagasi (vahelejätmise põhjus `dls-uncalibrated-band-988`) – toetatud meetodiks on paneeli kasutamine.
{% endhint %}

***

## Sihtmärkide piltide parimad tavad

### Sihtmärkide pildistamise juhised

**Aeg:**

* Pildistage sihtmärke vahetult enne pildistamissessiooni algust ja kogu selle vältel
* Samades valgustingimustes kui DAQ-valgusandur
* Parimate tulemuste saavutamiseks pildistage sihtmärki ideaalis võimalikult tihti. Muidu kasutatakse valgusanduri andmeid kalibreerimise kohandamiseks aja jooksul.

**Kaamera asend:**

* Hoidke kaamerat sihtmärgi kohal nii, et see oleks keskel ja täidaks umbes 40–60% pildi keskosast.
* Hoidke kaamera sihtmärgi pinnaga paralleelselt või selle suhtes nadir-asendis

**Valgustus:**

* Sama ümbritsev valgustus kui teie DAQ-valgusanduril
* Vältige varjude tekkimist sihtmärgi pindadel
* Ärge varjake valgusallikat oma kehaga, sõidukiga ega taimestikuga
* Pilves ilmastik tagab kõige ühtlasemad tulemused

**Sihtmärgi seisukord:**

* Hoidke sihtmärgi paneelid puhtad ja kuivad
* Kõik sihtmärgi paneelid (nt kõik 4 T4-l) peavad olema selgelt nähtavad ja takistusteta
* Sihtmärk peaks olema võimaluse korral valgusallikaga risti või selle suhtes nadir-asendis

### Kui palju sihtmärgi pilte?

**Minimaalselt:**1 sihtmärgi pilt seansi kohta.**Soovitatavalt:** 3–5 sihtmärgi pilti seansi kohta.**Parim praktika ajakava:**

* 3–5 pilti, mis on jäädvustatud vahetult pärast valgusanduri salvestamise algust
* Parimate tulemuste saamiseks pöörake kaamerat pildistamiste vahel
* Valikuliselt: perioodiliselt seansi keskel, kui valgustingimused muutuvad pidevalt

***

## Töö mitme kaameraga

### Kahe kaamera seadistused

Kui kasutate samaaegselt kahte „MAPIR“ kaamerat (nt Survey3W RGN + Survey3N OCN):

1. Pildistage sihtmärkide pilte **mõlema kaameraga** samaaegselt
2. Kasutage mõlema kaamera jaoks **sama füüsilist sihtmärki**

3. Märgistage failibrauseris**mõlema kaameratüübi** sihtmärgipildid
4. Chloros kasutab iga kaamera kalibreerimiseks sobivaid sihtmärke

### Veerg „Kaameramudel”

Veerg **„Kaameramudel”** aitab kindlaks teha, millised pildid pärinevad millisest kaamerast:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* jne.

Kasutage seda veergu, et kontrollida, kas olete oma projektis märkinud sihtmärgid iga kaameratüübi jaoks.

***

## Sihtmärkide tuvastamise seaded

### Tuvastamise tundlikkuse reguleerimine

Kui „Chloros” ei tuvasta teie sihtmärke õigesti, reguleerige neid seadeid [Projekti seaded](adjusting-project-settings.md) all:**Minimaalne kalibreerimisproovi pindala (px):*** **Vaikimisi**: 25 pikslit
* **Suurendage**, kui väikeste objektide puhul tekivad valed tuvastused
* **Vähendage**, kui sihtmärke ei tuvastata**Minimaalne sihtmärkide klastrite arv (0–100):*** **Vaikimisi**: 60
* **Suurendage**, kui sihtmärgid jagunevad mitmeks tuvastuseks
* **Vähendage**, kui värvimuutustega sihtmärke ei tuvastata täielikult

{% hint style="info" %}
**Näpunäide CLI'ile**: `chloros-cli process` toetab samu reguleerimisnuppe (`--min-target-size`, `--target-clustering`), ning selle lipukesed `--target`/`--targets` märgivad kogu sisendkausta sihtmärkide paneeliks. Vaata [CLI viidet](../reference/cli-reference.md).
{% endhint %}

***

## Tavalised sihtpildi probleemid

### Probleem: sihtmärke ei tuvastatud

**Võimalikud põhjused:**

* Sihtmärgipilte ei ole failibrauseris märgitud
* Sihtmärk on kaadris liiga väike (< 30% pildist)
* Halb valgustus (varjud, peegeldused)
* Sihtmärgi tuvastamise seaded on liiga ranged

**Lahendused:**

1. Veenduge, et õigete piltide juures on märgitud veerg „Sihtmärk“
2. Kontrollige sihtmärgi pildi kvaliteeti eelvaates
3. Kui kvaliteet on halb, pildistage sihtmärgid uuesti
4. Vajaduse korral kohandage sihtmärkide tuvastamise seadeid

### Probleem: valed sihtmärkide tuvastamised

**Võimalikud põhjused:**

* Valged hooned, sõidukid või maapinna kattekiht, mida peetakse ekslikult sihtmärkideks
* Heledad laigud taimestikus
* Tuvastamise tundlikkus on liiga madal

**Lahendused:**

1. Märgistage ainult tegelikud sihtmärgi pildid — skannitakse ainult märgistatud pilte
2. Suurendage kalibreerimise minimaalse proovipinna suurust
3. Suurendage sihtmärkide klastrite minimaalse arvu väärtust
4. Veenduge, et sihtmärgi piltidel oleks näha ainult sihtmärk (taustal võimalikult vähe segavaid elemente)

***

## Kontrollnimekiri

Enne töötlemise alustamist kontrollige sihtmärgi piltide valikut:

* [ ] Iga seansi kohta on märgitud vähemalt 1 sihtmärgi pilt (või LATTICE puhul seansi hõlmav `.daq`/`.csv` salvestus)
* [ ] Kõigi sihtmärgi piltide puhul on sihtmärgi veeru valikukastid märgitud
* [ ] Sihtmärgi pildid on jäädvustatud sama ajavahemiku jooksul kui uuring
* [ ] Sihtmärgid on eelvaates selgelt nähtavad, kui neile klõpsata
* [ ] Kõik kalibreerimispaneelid on igal sihtmärgi pildil nähtavad
* [ ] Sihtmärkidel ei ole varje ega takistusi
* [ ] Kahe kaamera puhul: sihtmärgid on märgitud mõlema kaameratüübi jaoks

***

## Sihtmärgita töötlemine

### LATTICE: DAQ-salvestusega

Kui DAQ-valgusandur salvestas LATTICE-pildistamise ajal allapoole suunatud kiirgustihedust, ei ole sihtmärki vaja:

1. Impordi piltidega fail `.daq` (või DAQ-M `.csv`)
2. Jäta veerg „Sihtmärk” märkimata
3. Peegeldusvõime arvutatakse automaatselt DAQ-i allapoole suunatud viitepõhjal
4. Kiirgusvõimsus ei vaja kunagi sihtmärki ega DAQ-i – see tuleneb ainult kaamera tehase radiomeetrilisest kalibreerimisest

### Töötlemine ilma viiteta

Võite töötleda ka ilma sihtmärkideta ja ilma DAQ-ita:

1. Jätke kõik „Sihtmärk” veeru valikukastid märkimata
2. **Lülita välja** „Peegeldusvõime kalibreerimine / valgebalanss” projekti seadetest – sihtpunktide tuvastamine jäetakse sel juhul täielikult vahele
3. Vignettkorrektsiooni rakendatakse ikkagi
4. Väljundit ei kalibreerita absoluutse peegeldusvõime järgi (LATTICE multispektraalne ekspordib endiselt debayered-, eelvaate- ja kiirgustiheduse tooteid)

{% hint style="warning" %}
**Survey3i teadustööks ei soovitata**: ilma peegeldusvõime kalibreerimiseta esindavad Survey3 pikseliväärtused ainult suhtelist heledust, mitte teaduslikke peegeldusvõime mõõtmisi. Täpsete ja korratavate tulemuste saamiseks kasutage kalibreerimissihte (või LATTICE puhul DAQ-valgusandurit).
{% endhint %}

***

## Järgmised sammud

Kui olete oma sihtpildid märgistanud:

1. **Vaadake oma seaded üle** – vt [Projekti seadete kohandamine](adjusting-project-settings.md)
2. **Alustage töötlemist** – vt [Töötlemise alustamine](starting-the-processing.md)
3. **Jälgige töötlemise kulgu** – vt [Töötlemise jälgimine](monitoring-the-processing.md)

Lisateavet kalibreerimismärkide kohta leiate jaotisest [Kalibreerimismärgid](../calibration-targets.md).
