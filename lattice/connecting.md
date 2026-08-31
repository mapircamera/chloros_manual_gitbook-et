# Kaamerate ühendamine

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Vahekaart „Kaamerad“ enne ühendamist</p></figcaption></figure>Chloros tuvastab LATTICE-kaamerad ühenduses automaatselt — kasutajaliidese vahekaardilt „Kaamerad”, `chloros-cli lattice`-ist või Python SDK kaudu. Kaamera mudelistring juhib kõiki järgnevaid toiminguid: Chloros määrab kindlaks anduri profiili, sagedusriba paigutuse ja tehase kalibreeringu kaamera `DeviceUserID` + `DeviceSerialNumber` andmete põhjal, seega **pole vaja ühtegi kaamerat eraldi konfigureerida**.

Enne ühendamist veenduge, et võrguhost on seadistatud – link-local-aadressimine, jumbo-raamid ja massiivide puhul võrgukaardi vastuvõtupuhvri seaded. See on riistvarapoolne seadistamine ja on kirjeldatud LATTICE&#x27;i kasutusjuhendis: [**Võrgu seadistamine**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Ühendamine graafilise kasutajaliidese kaudu

Avage Chloros külgribal vahekaart **Kaamerad**(riistvara vahekaardid ilmuvad, kui taustsüsteem on käivitumise lõpetanud) või kasutage peamenüüd →**Ühenda kaameraga**. Mõlemad avavad dialoogakna**Ühenda kaamera(d)**.

### Dialoogaken „Ühenda kaamera(d)”

Dialoogaken skaneerib võrku kohe, kui see avatakse („Võrgu skaneerimine...”), ja loetleb kõik leitud kaamerad. Igas reas on näidatud kaamera **mudel**(nt `LATT-M3M-L41-F550`),**seerianumber**ja**IP-aadress**.

* **Klõpsake rida, et see valida**(roheline esiletõstmine). Võite valida**mitu kaamerat** ja ühendada need korraga — Chloros ühendab need järjest.
* Ridad, millel on märge **„Ühendatud“**, on juba ühendatud ja neid ei saa uuesti valida.
* Ridad, millel on märge **„Massiivis“**, kuuluvad praegu ühendatud kaameramassiivi. Lõigake massiiv esmalt lahti, et kasutada seda kaamerat eraldi.
* **Ühenda** — ühendab valitud kaamera(d); kui on valitud rohkem kui üks kaamera, kuvab nupp arvu, nt „Ühenda (3)”.
* **Uuesti otsi** — käivitab kaamerate otsimise uuesti.
* **Sulge** — sulgeb dialoogakna.
* Kui otsing lõpeb tulemusteta, kuvatakse dialoogaknas **„Võrgust ei leitud ühtegi kaamerat“** — vaata allpool [Vigade kõrvaldamine](connecting.md#troubleshooting).

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Dialoogaken „Ühenda kaamera(d)” — siin näidatud olukorras, kus võrgus pole ühtegi kaamerat</p></figcaption></figure>### Esimene ühendamine: kalibreerimispaketi allalaadimine

Kui antud kaamera ühendatakse seadmega **esimest korda**, laadib Chloros kaamera tehase kalibreerimispaketi (\~3,8 MB) kaamerast endast GigE-võrgu kaudu alla. Selle käivitumise ajal kuvab dialoogaken rohelist paneeli**„Laadime kalibreerimisandmeid kaamerast alla“**koos seerianumbri kaupa jagatud edenemisribaga – arvestage umbes**70 sekundit** kaamera kohta. Pakett salvestatakse hosti vahemällu, mistõttu sama kaamera hilisemad ühendused jätavad allalaadimise täielikult vahele (ja paneeli ei kuvata kunagi).

### Süsteemi analüüs

Dialoogakna nupp **Süsteemi analüüs** kontrollib hosti ja võrku (protsessi ajal kuvatakse silt „Analüüsitakse...”) ning koostab diagnostikaruande:

* **Host** — protsessori tuumad ja RAM; GPU nimi ja mälu või „GPU: ei tuvastatud”.
* **Võrguliidesed** — iga võrgukaardi nimi, ühenduse kiirus, MTU (koos märkega „jumbo”, kui see on aktiivne), üles-/alla-olek ja kas see asub USB-bussil.
* **Kaamerad**— seerianumber, mudel, IP-aadress ja**millisel võrgukaardil iga kaamera asub**.
* **Jõudlus** — kaamera praegune ja ideaalne kaadrisagedus (fps) vastava pikseliformaadi puhul; kui ideaalne väärtus ületab praegust, kuvatakse roheline rida „Potentsiaal: võimalik N-kordne parandus”.
* **Hoiatused ja nummerdatud soovitused** — või „Süsteem toimib hästi praeguse kaamerate arvu juures”, kui midagi parandada pole.

Käivita see alati, kui kaamerate tuvastamine või voogedastus käitub ootamatult — see tuvastab enamiku võrgukaardi-poolseid probleeme (vale MTU, kaamera valel liidesel, USB-adapteri piirangud) dialoogist lahkumata.

### Kaamerate rühma ühendamine

Kahe või enama kaamera ühendamiseks **sünkroniseeritud rühmaks**kasutage selle asemel rühma ühendamise viisardit (**Connect Camera Array**): see juhendab teid läbi peremehe/orja valiku (eelnevalt täidetud GPIO ühendusproovi abil), kuvamisrežiimi valiku (eraldi vs. kombineeritud ruudud) ja massiivi seadete ekraani, kus enne kinnitamist kuvatakse reaalajas saavutatav kaadrisagedus (fps) ja ühenduse ribalaius. Viisardit ja kaamerarühma töövooge käsitletakse käesoleva juhendi mitme kaameraga kaamerarühmade jaotises; CLI vastavuseks on „LATTICE kaamera esmakordse ühendamise töövoog” [CLI viites](../reference/cli-reference.md).

## Ühendamine CLI ja SDK

CLI ja SDK juurdepääs eeldab tasulist Chloros+ taset ja sisselogimist; seda rakendatakse serveri poolel (`401 AUTH_REQUIRED`, kui pole sisse logitud, `403 PLAN_UPGRADE_REQUIRED` tasuta tasemel).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Täielikud allkirjad, valikud ja salvestamise töövood: [CLI Viide](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK Viide](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Kuidas kalibreerimine ühendamisel toimub

Igal LATTICE-kaameral on oma tehase kalibreerimispakett **kaameras** ning Chloros kontrollib ka MAPIR pilve, kui kaamera ühendub:

| Olukord   | Mida Chloros kasutab                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Võrgus**|**Selle seerianumbri jaoks avaldatud uusim kalibreerimine** — pilvekopeerimine võidab kaameras oleva koopia. Seega uuendab kaamera, mida on MAPIR abil uuesti kalibreeritud või uuendatud, end automaatselt; kasutaja sekkumist ei ole vaja. |
| **Offline**|**Kaameras olev pakett**, sellisena nagu see on. Täielikult offline-töövood toimivad edasi; need lihtsalt ei võta uuemaid kalibreeringuid kasutusele enne, kui kaamera on kord võrgus olnud (või tehases uuesti flashitud).                                                  |

Pildistamise ajal „külmutatakse“ tegelikult rakendatud koefitsiendid **iga pildi XMP-metadata sisse**. Hilisem kalibreerimise uuendus ei muuda kunagi märkamatult juba pildistatud pilte — vana pildi uuesti töötlemisel kasutatakse selle XMP-s salvestatud koefitsiente, mitte neid, mis on täna kõige uuemad.

## Veaotsing

* **„Võrgust ei leitud ühtegi kaamerat&quot;**— kontrollige link-local seadistust [Võrguseadistuses](https://mapir.gitbook.io/lattice-camera/setup/network-setup): hosti võrgukaardi staatiline aadress `169.254.x.x/16`, kaamerad samal lingil, DHCP-d ega väravat ei eeldata. Seejärel kasutage ühendusdialoogis funktsiooni**Süsteemi analüüs**, et kontrollida, millisel võrgukaardil iga kaamera on (või ei ole) nähtav.**Skannige uuesti** pärast igasugust kaabelduse või võrgukaardi vahetust.
* **Varem töötanud süsteem keeldub ühendusest** (massiivipaneeli väravad koos `FRAMES WILL DROP` / `Reduce ROI to enable`) — võrgukaardi draiveri uuendus on vaikimisi lähtestanud vastuvõturingi seaded. Kohaldage neid uuesti või käivitage `chloros-cli lattice network --fix` kõrgendatud õigustega terminalis; vt [Võrgu seadistamine](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kaamera kuvab „In Array“** — see kuulub ühendatud massiivi seanssi. Lõpetage massiivi ühendus, et kasutada kaamerat eraldi.
