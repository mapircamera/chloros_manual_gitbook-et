# Chloros+ sisselogimine

## Sisselogimine graafilise kasutajaliidese kaudu

Kasutaja <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> külgriba menüü võimaldab teil sisse logida oma Chloros+ kontole ja avada täiendavaid funktsioone.

**Igal arvutil tuleb sisse logida vaid üks kord.** Graafiline kasutajaliides, CLI ja Python SDK kasutavad sama vahemällu salvestatud sessiooni — sisselogimine töölaua graafilise kasutajaliidese kaudu aktiveerib samal arvutil ka CLI ja SDK (ja vastupidi `chloros-cli login` kaudu).

Sisselogimisel kuvatakse teie konto andmed:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Pakettide tasemed

| Pakett | `plan_id` | Tüüp |
| --- | --- | --- |
| Iron | `0` | Tasuta |
| Copper | `1` | Tasuline (Chloros+) |
| Bronze | `2` | Tasuline (Chloros+) |
| Hõbe | `3` | Tasuline (Chloros+) |
| Kuld | `4` | Tasuline (Chloros+) |

Vaata [paketid ja hinnad](https://cloud.mapir.camera/pricing), et teada saada, mida iga tasuline tase sisaldab.

### CLI / SDK juurdepääs eeldab tasulise taseme kasutamist

CLI ja Python ning SDK juurdepääsuks on vaja **mis tahes tasulist Chloros+ taset (Copper või kõrgem)**. Seda rakendatakse**serveri poolel** — iga CLI/SDK päring peab sisaldama nii aktiivset seanssi kui ka tasulist paketti:

| HTTP staatus | `error_code` | Tähendus | Lahendus |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Selles seadmes pole sisse logitud | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Oled sisse logitud, kuid paketi tase on liiga madal (tasuta Iron-tase) | Uuenda mis tahes tasulisele Chloros+ paketile |

`chloros-cli status` jääb tasuta tasemel kättesaadavaks, seega saad alati vaadata oma praegust paketti ja põhjust, miks juurdepääs keelati.

### Ühendatud seadmete piirangud paketi kohta

Iga pakett piirab, kui palju LATTICE-kaameraid ja DAQ-valgusandureid saab korraga reaalajas ühendada:

| Pakett | LATTICE-kaamerad | DAQ-valgusandurid |
| --- | --- | --- |
| Iron (tasuta / sisselogimata) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI sisselogimine

Logige sisse oma Chloros+ kasutajatunnuse ja parooliga, et võimaldada CLI töötlemist. Linux-is (ilma graafilise kasutajaliideseta) on see ainus viis oma litsentsi aktiveerimiseks.

**Süntaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK kasutajad**: Python SDK pakub ka programmeerimislikku `logout()` meetodit vahemällu salvestatud autentimisandmete tühistamiseks. Täpsema teabe saamiseks vaadake [SDK viidet](reference/sdk-reference.md).
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
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Autentimisandmete salvestamine

Vahemällu salvestatud autentimisandmed ja konfiguratsioon salvestatakse **kõikidel platvormidel** teie kasutaja kodukataloogi kaustas `.chloros`:

| Platvorm | Autentimisandmete vahemälu asukoht |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Paketi kehtivusaja lõpp ja offline-armuaeg

GUI-s kuvatav paketi kehtivusaja lõpp näitab, millal teie litsents kehtivuse kaotab. Korduvate igakuiste tellimuste puhul lõpeb kehtivusaeg kuu lõpus; aastaste tellimuste puhul on see aasta pärast tellimuse algust.

Chloros valideerib teie litsentsi võrgus, kuid võrguvälist töötamist toetatakse ajapikenduse jooksul:

* Edukad serverivalideerimised salvestatakse vahemällu **5 minutiks**, seega tavapärase kasutamise korral tehakse litsentsikõnesid väga vähe.
* Allkirjastatud, seadmega seotud litsentsi vahemälu katab pikemaid võrguväliseid perioode: **30 päeva kuupakettide puhul**ja**aastapakettide puhul kuni teie abonemendi kehtivusaja lõpuni (kõige rohkem 365 päeva)**.
* Kui üleminekuperiood lõpeb, lülitub pakett üle tasuta Iron-tasemele, kuni seade suudab litsentsiserveriga ühendust luua; juurdepääs taastub järgmise edukalt läbitud kontrolli järel.

### Seadmete piirang

Iga Chloros+ pakett pakub erinevat arvu registreeritud seadmeid. Iga seade, millesse sisse logite Chloros+ kontoga, arvestatakse teie registreeritud seadmete hulka. Seadme nime saate muuta ja seadme eemaldada oma MAPIR Cloudi kontolehel.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ pakett</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Toetatud seadmed</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>Teie konto täpne seadmete kvoot on näidatud teie MAPIR Cloudi kontolehel. Seadmest väljalogimine vabastab selle koha kindlalt ning juba registreeritud seade saab alati uuesti sisse logida, isegi kui konto on jõudnud seadmete piirini.
