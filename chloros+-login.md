# Chloros+ sisselogimine

## Chloros ja Chloros (brauser) sisselogimine

Kasutaja <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> külgriba menüü võimaldab teil sisse logida oma Chloros+ kontole ja avada täiendavad funktsioonid.

Kui olete sisse logitud, kuvatakse teie konto andmed:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI sisselogimine

Logige sisse oma Chloros+ kasutajatunnustega, et võimaldada CLI töötlemist. Linux-is (ilma graafilise kasutajaliideseta) on see ainus viis oma litsentsi aktiveerimiseks.

**Süntaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK kasutajad**: Python SDK pakub ka programmilist `logout()` meetodit vahemällu salvestatud kasutajatunnuste kustutamiseks. Täpsema teabe saamiseks vaadake [Python SDK dokumentatsiooni](api-python-sdk.md#logout).
{% endhint %}

**Näide:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Erimärgid**: Kasutage ühekordseid jutumärke paroolide ümber, mis sisaldavad märke nagu `$`, `!` või tühikuid.
{% endhint %}

**Väljund:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Autentimisandmete salvestamine

Vahemällu salvestatud autentimisandmed salvestatakse platvormispetsiifilisse asukohta:

| Platvorm | Autentimisandmete vahemälu asukoht |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Paketi kehtivusaja lõpp

GUI-s kuvatav paketi kehtivusaja lõpp näitab, millal teie litsents kaotab kehtivuse. Korduvate igakuiste tellimuste puhul on kehtivusaja lõpp kuu lõpus. Aastaste tellimuste puhul on see aasta pärast tellimuse algust. Litsentsi kontrollimiseks on vaja igakuist internetiühendust, kusjuures kehtib 30-päevane ajapikendus.

### Seadmete piirang

Iga Chloros+ pakett pakub erinevat arvu registreeritud seadmeid. Iga seade, millesse sisse logite Chloros+ kontoga, loetakse registreeritud seadmete hulka. Seadme nime saate muuta ja seadme eemaldada oma MAPIR pilvekonto lehel.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ pakett</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">HÕBE</th><th align="center">KULD</th></tr></thead><tbody><tr><td align="right">Toetatud seadmed</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
