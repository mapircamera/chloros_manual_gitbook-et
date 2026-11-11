# Chloros+ sisselogimine

## Sisselogimine graafilise kasutajaliidese kaudu

Kasutaja <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> külgriba menüü võimaldab sul oma Chloros+ kontosse sisse logida ja avada täiendavaid funktsioone.

**Igal arvutil tuleb sisse logida vaid üks kord.** GUI, CLI ja Python SDK kasutavad sama vahemällu salvestatud sessiooni – sisselogimine töölaua GUI kaudu aktiveerib ka CLI ja SDK selle arvuti peal (ja vastupidi, kasutades `chloros-cli login`).

Sisselogimisel kuvatakse teie konto andmed:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Pakettide tasemed

| Pakett | `plan_id` | Tüüp |
| --- | --- | --- |
| Iron | `0` | Tasuta |
| Copper | `1` | Tasuline (Chloros

+) |
| Bronze | `2` | Tasuline (Chloros

+) |
| Silver | `3` | Tasuline (Chloros

+) |
| Kuld | `4` | Tasuline (Chloros

+) |

Vaata [pakette ja hindu](https://cloud.mapir.camera/pricing), et teada saada, mida iga tasuline pakett sisaldab.

### CLI

-le / SDK

-le juurdepääs eeldab tasulist paketti

CLI

-le ja Python

i SDK

-le juurdepääs eeldab **mis tahes tasulist Chloros

+ taset (Copper või kõrgem)**. Seda rakendatakse**serveri poolel** — iga CLI

/SDK

-päring peab sisaldama nii aktiivset sessiooni kui ka tasulist paketti:

| HTTP

-staatus | `error_code` | Tähendus | Lahendus |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Selles seadmes pole sisse logitud | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Sisse logitud, kuid paketi tase on liiga madal (tasuta Iron-tase) | Uuendage mis tahes tasulisele Chloros

+ paketile |

`chloros-cli status` jääb tasuta tasemel kättesaadavaks, seega saate alati vaadata oma praegust paketti ja põhjust, miks juurdepääs keelati.

### Ühendatud seadmete piirangud paketi kohta

Iga pakett piirab, kui palju LATTICE-kaameraid ja DAQ-valgusandureid saab korraga reaalajas ühendada:

| Pakett | LATTICE-kaamerad | DAQ-valgusandurid |
| --- | --- | --- |
| Iron (tasuta / sisselogimata) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI

-sisselogimine

Logige sisse oma Chloros

+ kasutajatunnustega, et võimaldada CLI

-töötlemist. Linux

is (ilma graafilise kasutajaliideseta) on see ainus viis oma litsentsi aktiveerimiseks.

**Süntaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK

-kasutajad**: Python

i SDK

pakub ka programmilist meetodit `logout()` vahemällu salvestatud kasutajatunnuste tühjendamiseks. Täpsemad üksikasjad leiate [SDK

-viitekogust](reference/sdk-reference.md).
{% endhint %}

**Näide:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Erimärgid**: Kasutage ühekordseid jutumärke paroolide ümber, mis sisaldavad märke nagu `$`, `!` või tühikuid.
{% endhint %}

**Väljund:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Autentimisandmete salvestamine

Vahemällu salvestatud autentimisandmed ja konfiguratsioon salvestatakse **kõikidel platvormidel** teie kasutaja kodukataloogi kausta `.chloros`:

| Platvorm | Autentimisandmete vahemälutee |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Uue versiooni paigaldamine olemasoleva peale säilitab teie sisselogimisandmed nii Windows

is (paigaldaja) kui ka Linux

is (kataloog `.deb`, sealhulgas `chloros-cli update`). Chloros

i desinstallimine — või Linux

i paketi eemaldamine — kustutab vahemällu salvestatud sisselogimisandmed, mistõttu peate pärast uue versiooni installimist uuesti sisse logima.

### Paketi kehtivusaja lõpp ja offline-armuaeg

GUI-s kuvatav paketi kehtivusaja lõpp näitab, millal teie litsents kaotab kehtivuse. Korduvate kuumaksete puhul lõpeb kehtivusaeg kuu lõpus; aastamaksete puhul on see aasta pärast tellimuse algust.

Chloros

valideerib teie litsentsi võrgus, kuid võrguühenduseta töötamine on toetatud ajapikenduse jooksul:

* Edukad serverivalideerimised salvestatakse vahemällu **5 minutiks**, seega tavapärase kasutuse korral tehakse litsentsipäringuid väga vähe.
* Allkirjastatud, seadmega seotud litsentsi vahemälu katab pikemaid võrguühenduseta perioode: **30 päeva kuupakettide puhul**ja**aastapakettide puhul kuni teie tellimuse kehtivusaja lõpuni (kõige rohkem 365 päeva)**.
* Kui ajapikendusperiood lõpeb, lülitub pakett üle tasuta Iron-tasemele, kuni seade suudab litsentsiserveriga ühendust saada; juurdepääs taastub järgmise edukalt läbitud kontrolli järel.

### Seadmete piirang

Iga „Chloros

+“ pakett pakub erinevat arvu registreeritud seadmeid. Iga seade, millesse logite sisse „Chloros

+“ kontoga, loetakse registreeritud seadmete hulka. Seadme nime saate muuta ja seadme eemaldada oma „MAPIR

Cloud“ kontolehel.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ pakett</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Toetatud seadmed</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Teie konto täpne seadmete limiit on näidatud teie MAPIRi pilvekonto lehel. Seadmest väljalogimine vabastab selle koha kindlalt ning juba registreeritud seade saab alati uuesti sisse logida, isegi kui konto on jõudnud seadmete limiidini.
