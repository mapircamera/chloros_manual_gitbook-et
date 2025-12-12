# Sihtpiltide valimine

Kalibreerimise sihtmärkide sisaldavate piltide märkimine on oluline samm, mis kiirendab oluliselt Chloros töötlemisprotsessi. Sihtpiltide eelvalikuga ei pea Chloros skannima kõiki andmekogus olevaid pilte kalibreerimise sihtmärkide leidmiseks.

## Miks märkida sihtpilte?

### Töötlemise kiirus

Sihtpilte märkimata peab Chloros:

* Skaneerima iga projekti pildi
* Käivitama sihtmärgi tuvastamise algoritmid iga pildi puhul
* Kontrollima sadu või tuhandeid pilte asjatult

**Tulemus**: Töötlemine võib võtta oluliselt kauem aega, eriti suurte andmekogumite puhul.

### Märgitud sihtmärgi piltidega

Kui märgite sihtmärgi veerus konkreetsed pildid:

* Chloros skaneerib sihtmärke ainult märgitud piltidel
* Sihtmärgi tuvastamine toimub palju kiiremini
* Üldine töötlemisaeg lüheneb oluliselt

{% vihje style=&quot;success&quot; %}
**Kiiruse paranemine**: 500 pildist koosnevas andmekogus 2–3 sihtpildi märkimine võib sihtmärgi tuvastamise aega lühendada 30+ minutilt alla 1 minuti.
{% endhint %}

***

## Sihtmärgi piltide märkimine

### 1. samm: Sihtmärgi piltide tuvastamine

Vaadake failibrauseris imporditud pilte ja tuvastage, millised pildid sisaldavad kalibreerimise sihtmärke.

**Tavalised stsenaariumid:**

* **Eelnevalt jäädvustatud sihtmärk**: jäädvustatud enne seansi algust.
* **Järeljäädvustatud sihtmärk**: jäädvustatud pärast seansi lõppu.
* **Väljas olevad sihtmärgid**: sihtmärgid, mis asuvad jäädvustamisalas.
* **Mitmed sihtmärgid**: 2–3 sihtmärgi pilti seansi kohta (soovitatav).

### 2. samm: kontrollige sihtmärgi veergu

Iga kalibreerimise sihtmärki sisaldava pildi puhul:

1. Leidke pilt failibrauseri tabelist.
2. Leidke **Sihtmärk** veerg (parempoolseim veerg).
3. Klõpsake selle pildi sihtmärgi veerus olevale märkeruudule.
4. Korrake seda kõigi sihtmärke sisaldavate piltide puhul.

### 3. samm: kontrollige oma valikut

Enne töötlemist kontrollige veel kord:

* [ ] Kõik kalibreerimise sihtmärgiga pildid on märgitud
* [ ] Ükski sihtmärgita pilt ei ole kogemata märgitud
* [ ] Sihtmärgid on märgitud piltidel selgelt nähtavad

***

## Sihtmärgi piltide parimad tavad

### Sihtmärgi pildistamise juhised

**Aeg:**

* Pildistage sihtmärgi pildid vahetult enne pildistamise seanssi ja selle käigus
* Samades valgustingimustes kui DAQ valgussensor
* Parimate tulemuste saavutamiseks pildistage sihtmärke võimalikult tihti. Muidu kasutatakse kalibreerimise kohandamiseks aja jooksul valgussensori andmeid.

**Kaamera asend:**

* Hoia kaamera sihtmärgi kohal nii, et see oleks keskel ja täidaks umbes 40–60% pildi keskosast.
* Hoia kaamera sihtmärgi pinnaga paralleelselt/nadiril

**Valgustus:**

* Sama ümbritsev valgus kui teie DAQ valgussensoril.
* Vältige varjusid sihtmärgi pinnal.
* Ärge varjake valgusallikat oma kehaga, sõidukiga või taimestikuga.
* Pilvine ilm annab kõige ühtlasemad tulemused.

**Sihtmärgi seisund:**

* Hoidke sihtmärgi paneelid puhtad ja kuivad.
* Kõik 4 paneeli peavad olema selgelt nähtavad ja takistusteta.
* Sihtmärgid võimaluse korral risti/nadir valgusallikaga.

### Kui palju sihtmärgi pilte?

**Minimaalselt:** 1 sihtmärgi pilt seansi kohta. **Soovitatav:** 3–5 sihtmärgi pilti seansi kohta.

**Parim praktika:**

* 3–5 pilti, mis on tehtud vahetult pärast valgussensori salvestamise algust
* Parimate tulemuste saamiseks pöörake kaamerat pildistamise vahel
* Valikuliselt: perioodiliselt seansi keskel, kui valgustusolud muutuvad pidevalt

***

## Töö mitme kaameraga

### Kahe kaameraga seadistused

Kui kasutate kahte MAPIR kaamerat korraga (nt Survey3W RGN + Survey3N OCN):

1. Pildistage sihtmärgid **mõlema kaameraga** samaaegselt.
2. Kasutage mõlema kaamera jaoks **sama füüsilist sihtmärki**.
3. Märgistage sihtmärgid **mõlema kaameratüübi** jaoks failibrauseris.
4. Chloros kasutab iga kaamera kalibreerimiseks sobivaid sihtmärke.

### Kaamera mudeli veerg

Veerg **Kaamera mudel** aitab kindlaks teha, millised pildid on pärit millisest kaamerast:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* jne.

Kasutage seda veergu, et kontrollida, kas olete oma projektis märkinud sihtmärgid iga kaameratüübi jaoks.

***

## Sihtmärgi tuvastamise seaded

### Tuvastamise tundlikkuse reguleerimine

Kui Chloros ei tuvasta teie sihtmärke õigesti, reguleerige neid seadeid [Projekti seaded](adjusting-project-settings.md):

**Minimaalne kalibreerimise valimi pindala:**

* **Vaikimisi**: 25 pikslit
* **Suurendage**, kui väikeste artefaktide puhul tekivad valed tuvastused.
* **Vähendage**, kui sihtmärke ei tuvastata.

**Minimaalne sihtmärkide klastrite arv:**

* **Vaikimisi**: 60
* **Suurendage**, kui sihtmärgid jagunevad mitmeks tuvastamiseks
* **Vähendage**, kui värvitooniga sihtmärgid ei tuvastata täielikult

***

## Tavapärased sihtmärgi pildi probleemid

### Probleem: sihtmärke ei tuvastata

**Võimalikud põhjused:**

* Sihtmärgi pildid ei ole failibrauseris märgitud
* Sihtmärk on raamis liiga väike (&lt; 30% pildist)
* Halb valgustatus (varjud, päikese peegeldus)
* Sihtmärgi tuvastamise seaded on liiga ranged

**Lahendused:**

1. Kontrollige, et sihtmärgi veerg on märgitud õigete piltide jaoks
2. Vaadake sihtmärgi pildi kvaliteeti eelvaates
3. Kui kvaliteet on halb, pildistage sihtmärgid uuesti
4. Vajadusel kohandage sihtmärgi tuvastamise seadeid

### Probleem: vale sihtmärgi tuvastamine

**Võimalikud põhjused:**

* Valged hooned, sõidukid või maapind, mida ekslikult peetakse sihtmärkideks
* Heledad laigud taimestikus
* Liiga madal tuvastamise tundlikkus

**Lahendused:**

1. Märkige ainult tegelikud sihtmärgi pildid, et piirata tuvastamise ulatust
2. Suurendage minimaalse kalibreerimise valimi pindala
3. Suurendage minimaalse sihtmärgi klastri väärtust
4. Veenduge, et sihtmärgi pildid näitavad ainult sihtmärki (minimaalne taustakülg)

***

## Kontrollnimekiri

Enne töötlemise alustamist kontrollige oma sihtmärgi piltide valikut:

* [ ] Iga seansi kohta on märgitud vähemalt 1 sihtmärgi pilt.
* [ ] Kõigi sihtmärgi piltide puhul on märgitud sihtmärgi veeru valikukastid.
* [ ] Sihtmärgi pildid on tehtud sama ajavahemiku jooksul kui uuring.
* [ ] Sihtmärgid on eelvaates selgelt nähtavad, kui neile klõpsata.
* [ ] Kõik 4 kalibreerimispaneeli on nähtavad igal sihtmärgi pildil.
* [ ] Sihtmärkidel ei ole varjusid ega takistusi
* [ ] Kahe kaamera puhul: sihtmärgid on märgitud mõlema kaameratüübi jaoks

***

## Sihtmärkideta töötlemine

### Töötlemine ilma kalibreerimise sihtmärkideta

Kuigi teadustööks ei ole see soovitatav, võite töötleda ilma sihtmärkideta:

1. Jätke kõik sihtmärgi veeru valikukastid märkimata
2. **Keelake** projekti seadetest „Peegelduskalibreerimine”
3. Vignette&#x27;i korrigeerimine jääb endiselt kehtima.
4. Väljundit ei kalibreerita absoluutse peegeldusvõime jaoks.

{% vihje style=&quot;warning&quot; %}
**Ei soovitata**: ilma peegeldusvõime kalibreerimiseta esindavad pikseliväärtused ainult suhtelist heledust, mitte teaduslikke peegeldusvõime mõõtmisi. Täpsete ja korratavate tulemuste saamiseks kasutage kalibreerimise sihtmärke.
{% endhint %}

***

## Järgmised sammud

Kui olete märkinud oma sihtpildid:

1. **Vaadake oma seaded üle** – vt [Projekti seadeid kohandamine](adjusting-project-settings.md)
2. **Alustage töötlemist** – vt [Töötlemise alustamine](starting-the-processing.md)
3. **Jälgige protsessi kulgu** – vt [Töötlemise jälgimine](monitoring-the-processing.md)

Lisateavet kalibreerimise sihtmärkide kohta leiate jaotisest [Kalibreerimise sihtmärgid](../calibration-targets.md).
