# Chloros+ sisselogimine

## Chloros ja Chloros (brauser) sisselogimine

Kasutaja <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> külgribamenüü võimaldab teil sisse logida oma Chloros+ kontosse ja avada lisafunktsioone.

Sisse logimisel kuvatakse teie konto andmed:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLI sisselogimine

Logige sisse oma Chloros+ kasutajatunnustega, et võimaldada CLI töötlemist.

**Süntaks:**

```bash
chloros-cli login <email> <password>
```

**Näide:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Erimärgid**: Kasutage ülakomaid paroolide ümber, mis sisaldavad märke nagu `$`, `!` või tühikuid.
{% endhint %}

**Väljund:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### Paketi kehtivusaeg

Paketi kehtivusaeg GUI-s näitab, millal teie litsents kehtivuse kaotab. Kuu jooksul korduvate tellimuste puhul on kehtivusaeg kuu lõpus. Aastaste tellimuste puhul on kehtivusaeg aasta pärast tellimuse algust. Litsentsi kontrollimiseks on vaja igakuist internetiühendust, mille puhul on 30-päevane ajapikendus.

### Seadmete piirang

Iga Chloros+ plaan pakub erinevat arvu registreeritud seadmeid. Iga seade, millesse logite sisse Chloros+ kontoga, loetakse registreeritud seadmete hulka. Seadme nime saab muuta ja seadme saab eemaldada MAPIR pilvekonto lehel.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ pakett</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">HÕBE</th><th align="center">KULD</th></tr></thead><tbody><tr><td align="right">Toetatud seadmed</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
