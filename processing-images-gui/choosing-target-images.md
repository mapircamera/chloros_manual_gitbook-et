# Sihtpiltide valimine

Kalibreerimissihte sisaldavate piltide märkimine on oluline samm, mis kiirendab märkimisväärselt Chloros töötlemisprotsessi. Sihtpiltide eelvalikuga vältite olukorda, kus Chloros peab andmekogus olevatest piltidest kalibreerimissihte otsima.

## Miks sihtpilte märkida?

### Töötlemiskiirus

Kui sihtpilte ei märgistata, peab Chloros:

* Skaneerima iga projekti pildi
* Käivitama sihtmärkide tuvastamise algoritme igal pildil
* Kontrollima tarbetult sadu või tuhandeid pilte

**Tulemus**: Töötlemine võib võtta märkimisväärselt kauem aega, eriti suurte andmekogumite puhul.

### Märgistatud sihtpiltidega

Kui märgistate sihtpiltide veerus konkreetsed pildid:

* Chloros skaneerib sihtmärke ainult märgitud piltidelt
* Sihtmärkide tuvastamine lõpeb palju kiiremini
* Üldine töötlemisaeg lüheneb oluliselt

{% hint style="success" %}
**Kiiruse paranemine**: 2–3 sihtmärgi pildi märkimine 500-pildilises andmekogus võib lühendada sihtmärkide tuvastamise aega 30+ minutilt alla 1 minuti.
{% endhint %}

***

## Kuidas märgistada sihtmärkpilte

### 1. samm: Määrake kindlaks oma sihtmärkpildid

Vaadake failibrauseris läbi oma imporditud pildid ja määrake kindlaks, millised pildid sisaldavad kalibreerimise sihtmärke.

**Tavalised stsenaariumid:*** **Enne pildistamist märgitud sihtmärk**: Märgitud enne seansi algust
* **Pärast pildistamist märgitud sihtmärk**: Märgitud pärast seansi lõppu
* **Välitingimustes asuvad sihtmärgid**: Pildistamispiirkonnas paigutatud sihtmärgid
* **Mitmed sihtmärgid**: 2–3 sihtmärgi pilti seansi kohta (soovitatav)

### 2. samm: Kontrollige sihtmärgi veergu

Iga kalibreerimissihte sisaldava pildi puhul:

1. Leidke pilt failibrauseri tabelis
2. Leidke veerg **Sihtmärk** (kõige parempoolne veerg)
3. Klõpsake selle pildi juures veerus „Sihtmärk” asuvale märkeruudule
4. Korrake seda kõigi sihtmärke sisaldavate piltide puhul

### 3. samm: Kontrollige oma valikut

Enne töötlemist kontrollige veel kord:

* [ ] Kõik kalibreerimissihte sisaldavad pildid on märgitud
* [ ] Ükski sihtmärgita pilt ei ole kogemata märgitud
* [ ] Sihtmärgid on märgitud piltidel selgelt nähtavad

***

## Sihtmärgipiltide parimad tavad

### Sihtmärgi pildistamise juhised

**Aeg:**

* Pildistage sihtmärgipilte vahetult enne pildistamissessiooni algust ja selle käigus
* Samades valgustingimustes kui teie DAQ valgussensor
* Parimate tulemuste saamiseks pildistage sihtmärke ideaalis võimalikult tihti. Muidu kasutatakse kalibreerimise kohandamiseks aja jooksul valgussensori andmeid.

**Kaamera asend:**

* Hoidke kaamerat sihtmärgi kohal nii, et see oleks keskel ja täidaks umbes 40–60% pildi keskosast.
* Hoidke kaamera paralleelselt/nadirina sihtmärgi pinnaga

**Valgustus:**

* Sama ümbritsev valgustus kui DAQ valgusanduril
* Vältige varju sihtmärgi pinnal
* Ärge varjake valgusallikat oma kehaga, sõiduki või taimestikuga
* Pilves ilmastik tagab kõige ühtlasemad tulemused

**Sihtmärgi seisukord:**

* Hoidke sihtmärgi paneelid puhtad ja kuivad
* Kõik 4 paneeli peavad olema selgelt nähtavad ja takistusteta
* Sihtmärgid võimaluse korral risti/nadiriga valgusallikaga

### Kui palju sihtmärgi pilte?

**Minimaalselt:**1 sihtmärgi pilt seansi kohta.**Soovitatavalt:** 3–5 sihtmärgi pilti seansi kohta.**Parim praktika ajakava:**

* 3–5 pilti, mis on jäädvustatud vahetult pärast valgusanduri salvestamise algust
* Parimate tulemuste saamiseks vahetage kaamera asendit pildistamiste vahel
* Valikuliselt: perioodiliselt seansi keskel, kui valgustingimused muutuvad pidevalt

***

## Töö mitme kaameraga

### Kahe kaamera seadistused

Kui kasutate kahte MAPIR kaamerat samaaegselt (nt Survey3W RGN + Survey3N OCN):

1. Salvestage sihtmärgi pildid **mõlema kaameraga** samaaegselt
2. Kasutage mõlema kaamera jaoks **sama füüsilist sihtmärki**

3. Märgistage failibrauseris**mõlema kaameratüübi** sihtmärgi pildid
4. Chloros kasutab iga kaamera kalibreerimiseks sobivaid sihtmärke

### Veerg „Kaameramudel”

Veerg **„Kaameramudel”** aitab kindlaks teha, millised pildid pärinevad millisest kaamerast:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* jne.

Kasutage seda veergu, et kontrollida, kas olete oma projektis märkinud sihtmärgid iga kaameratüübi jaoks.

***

## Sihtmärkide tuvastamise seaded

### Tuvastamise tundlikkuse reguleerimine

Kui Chloros ei tuvasta teie sihtmärke õigesti, reguleerige neid seadeid [Projekti seaded](adjusting-project-settings.md):**Minimaalne kalibreerimisproovi pindala:*** **Vaikimisi**: 25 pikslit
* **Suurendage**, kui väikeste objektide puhul tekivad valed tuvastused
* **Vähendage**, kui sihtmärke ei tuvastata**Minimaalne sihtmärkide klastrite arv:*** **Vaikimisi**: 60
* **Suurendage**, kui sihtmärgid jagunevad mitmeks tuvastuseks
* **Vähendage**, kui värvimuutustega sihtmärke ei tuvastata täielikult***

## Sihtmärgi pildi tavalised probleemid

### Probleem: sihtmärke ei tuvastata

**Võimalikud põhjused:**

* Sihtmärgi pilte ei ole failibrauseris märgitud
* Sihtmärk on kaadris liiga väike (&lt; 30% pildist)
* Halb valgustus (varjud, peegeldused)
* Sihtmärgi tuvastamise seaded on liiga ranged

**Lahendused:**

1. Kontrollige, kas veerus „Sihtmärk” on märgitud õiged pildid
2. Vaadake sihtmärgi pildi kvaliteeti eelvaates
3. Pildistage sihtmärgid uuesti, kui kvaliteet on halb
4. Reguleerige vajadusel sihtmärgi tuvastamise seadeid

### Probleem: vale sihtmärgi tuvastamine

**Võimalikud põhjused:**

* Valged hooned, sõidukid või maapinna kattega alad, mida peetakse sihtmärkideks
* Heledad laigud taimestikus
* Tuvastamise tundlikkus on liiga madal

**Lahendused:**

1. Märgistage ainult tegelikud sihtmärgi pildid, et piirata tuvastamise ulatust
2. Suurendage minimaalse kalibreerimisproovi pindala
3. Suurendage minimaalse sihtmärgi klastrite arvu
4. Veenduge, et sihtmärgi piltidel on näha ainult sihtmärk (minimaalne taustakohin)

***

## Kontrollnimekiri

Enne töötlemise alustamist kontrollige oma sihtmärgi piltide valikut:

* [ ] Iga seansi kohta on märgistatud vähemalt 1 sihtmärgi pilt
* [ ] Kõigi sihtmärkide piltide puhul on märgitud sihtmärgi veeru valikukastid
* [ ] Sihtmärkide pildid on jäädvustatud sama ajavahemiku jooksul kui uuring
* [ ] Sihtmärgid on eelvaates selgelt nähtavad, kui neile klõpsatakse
* [ ] Kõik 4 kalibreerimispaneeli on nähtavad igal sihtmärgi pildil
* [ ] Sihtmärkidel ei ole varju ega takistusi
* [ ] Kahe kaamera puhul: sihtmärgid on märgitud mõlema kaameratüübi jaoks

***

## Sihtmärkideta töötlemine

### Töötlemine ilma kalibreerimissihte

Kuigi seda ei soovitata teadustööks, on võimalik töötada ilma sihtideta:

1. Jätke kõik sihtide veeru valikukastid märkimata
2. **Lülitage välja** „Peegelduskalibreerimine” projekti seadetest
3. Vignettkorrektsiooni rakendatakse ikkagi
4. Väljundit ei kalibreerita absoluutse peegelduse jaoks

{% hint style="warning" %}
**Ei soovitata**: Ilma peegelduskalibreerimiseta esindavad pikselväärtused ainult suhtelist heledust, mitte teaduslikke peegeldusmõõtmisi. Kasutage kalibreerimistargetit täpsete ja korratavate tulemuste saamiseks.
{% endhint %}

***

## Järgmised sammud

Kui olete oma target-pildid märkinud:

1. **Vaadake oma seaded üle** – vt [Projekti seadete kohandamine](adjusting-project-settings.md)
2. **Alustage töötlemist** – vt [Töötlemise alustamine](starting-the-processing.md)
3. **Jälgige töötlemise kulgu** – vt [Töötlemise jälgimine](monitoring-the-processing.md)

Lisateavet kalibreerimissihte kohta leiate jaotisest [Kalibreerimissihid](../calibration-targets.md).
