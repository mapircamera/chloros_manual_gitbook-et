# CLI-i kiirjuhend (pool-*)

Kaasasolev `chloros-cli` juhib DAQ-andureid **`daq pool-*`**-käskude perekonna kaudu — need on lihtsad HTTP-kliendid, mis juhivad andurit Chloros-tagapõhja püsiva andurite reservi kaudu. Backend haldab andmeedastust, mistõttu jagavad GUI, CLI ja SDK-skriptid ühte aktiivset käepidet, selle asemel et võidelda pordi pärast. Kõik, mida klient vajab, on kättesaadav `pool-*` kaudu: ühenduse loomine, andmevoo edastamine, kalibreeritud `.daq`-failide salvestamine ja kap-profiilide vahetamine.

`pool-*` on väljalastud versioonides toetatud andmekogumispind. `chloros-cli daq --help` loetleb `pool-*` alamkäskud ning `daq discover` / `daq list` käivitavad sama skaneerimise kui `pool-discover`. Kui otsene riistvarapõhine DAQ-alamkäsk pole saadaval, lõpetab programm töö selge veateatega, milles nimetatakse puuduvat paketti ja suunatakse kasutaja tagasi käsu `pool-*` juurde – midagi ei ebaõnnestu märkamatult. (Ka `pip install chloros-sdk` ei paku otsese riistvaraga seotud käske.)

***

## Eeltingimused

* **Chlorosi taustprotsess peab töötama** — `pool-*`-käskud on HTTP-kliendid, mitte riistvaradraiverid. Windowsis käivita Chlorosi töölaua rakendus (see käivitab taustprotsessi). Kuvataustata Linux/Jetsonil aktiveerige teenus oma kasutajana: `sudo systemctl enable --now chloros-backend@$USER.service` (vt [Linuxi paigaldamine](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Chloros+ (tasuline tase) sisselogimine**: käivita esmalt `chloros-cli login`. Jõustamine toimub serveri poolel — ilma sisselogimiseta ebaõnnestuvad käsud `401 AUTH_REQUIRED`-iga; tasuta (Iron) tasemel ebaõnnestuvad need `403 PLAN_UPGRADE_REQUIRED`-iga.
* Käskudele on vaikimisi määratud sihtmärgiks `http://127.0.0.1:5000`; `daq pool-*`-perekond arvestab keskkonnamuutujat `CHLOROS_BACKEND_URL`, kui teie tagapõhi töötab mujal.

***

## Viieminutiline seanss

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — ava andur andurite kogumis

| Variant | Tähendus |
| --- | --- |
| `daq pool-connect` | Nutikas tuvastamine: leia kõik selle masina DAQ-seadmed. |
| `daq pool-connect --port PORT` | DAQ-U kindlal jadaportil (nt `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M BLE kaudu, MAC-aadressi automaatne skaneerimine. |
| `daq pool-connect --mac MAC` | DAQ-M teadaoleva BLE MAC-aadressiga (tähendab `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E teadaoleva hostinime või IP-aadressiga — **usaldusväärne tee**. |
| `daq pool-connect --eth` | DAQ-E automaatse avastamisega (mDNS, ARP-varuvõimalusega). Vaata allpool olevat hoiatust. |

Häälestuslipud, kõik valikulised:

| Lipu | Tähendus |
| --- | --- |
| `--integration-time MS` / `-t MS` | Käsitsi määratud integratsiooniaeg millisekundites. |
| `--frame-avg N` / `-f N` | Raamide keskmine arv iga teatatud spektri kohta. |
| `--no-ae` | Lülita automaatne säritus välja (AE on vaikimisi sisse lülitatud). |
| `--no-stream` | Ühendus ilma voo käivitamiseta (jätka hiljem käsuga `pool-stream --start`). |
| `--cap-id CAP` | Cap-korrektsiooni profiil; tagapõhja vaikimisi väärtus on `sunshine_cosine`. Vaata [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Juba poolis oleva DAQ-E ühendamine mõne teise selle aadressi kaudu — kas selle `daq-e-<id>.local` hostinime või IP-aadressi kaudu — tagastab olemasoleva pooli kande, selle asemel et seadet teist korda avada.

{% hint style="warning" %}
**`--eth` automaatse avastamise hoiatus.** Mitme võrguliidesega hostil (rohkem kui üks aktiivne võrguliides) võib *esimene* `pool-connect --eth` pärast käivitamist jääda tühjaks isegi siis, kui andur on töökorras – avastamisotsing võib anduri liidese vahele jätta, kui ARP-vahemälu on tühjaks jäänud. Kui `--eth` midagi ei leia, proovi uuesti või jäta avastamine üldse vahele, kasutades `--eth-host <ip-or-hostname>`-i, mis on mitme võrguliidesega masinatel usaldusväärne lahendus. DAQ-E-seadme hostinimi on `daq-e-<id>.local` (nt `daq-e-def330.local`); ka selle tavaline IP-aadress sobib.
{% endhint %}

## `pool-discover` — skannimine ühenduseta

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Otsib andureid, millega saaksid ühenduse luua, ja näitab ühenduseks vajalikku aadressi, ilma et ühendust loodaks. `--only` võtab komadega eraldatud alamhulga `usb,ble,eth`-ist; `--timeout SEC` määrab kindlaks skannimise kestuse ühe transpordi kohta (vaikimisi 5 s, piiratud vahemikuga 1–20 s). Poolis juba avatud andureid ei kuvata (ühendatud DAQ-M lõpetab reklaamimise) — nende jaoks kasuta `pool-list`. Tarnitud versioonis käivitavad `chloros-cli daq discover` ja `daq list` sama skaneerimise.

## `pool-list` — vaata, mis on ühendatud

Näitab kõiki backend-poolis olevaid andureid, sealhulgas `sensor_id`-i, mida kõik teised käsud vajavad:

| Mudel | `sensor_id`-i vorm | Näide |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktettiline, sidekriipsuga | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

DAQ-E, mille PTP on välja lülitatud (seade on salvestatud seadmesse), on märgitud `pool-list`-is koos `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.`-iga ja käskuga, mis lülitab selle uuesti sisse — vt [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — spektri raamide lugemine

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Tagastab viimase raami või viimased `--recent N` raamid; `--json` väljastab skriptimiseks masinloetavat väljundit. Raamid on radiomeetriliselt kalibreeritud spektraalne kiirgustihedus (W/m²/nm) 135-punktilisel, 340–1010 nm võrgustikul, millele on juba rakendatud anduri katteprofiil. Kvantitatiivsete kiirgustiheduse väärtuste saamiseks arvutage vähemalt 15 sekundi pikkuste kaadrite keskmine – see on seadme omadus, mitte defekt.

## `pool-stream` — voo peatamine või jätkamine

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — salvestage `.daq`-fail

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Lipuk | Vaikimisi | Tähendus |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Salvestuse pikkus sekundites; `0` tähendab, et salvestamine jätkub kuni käskluse `--stop` andmiseni. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Väljundkataloog. Kui tagapõhi asub selles arvutis, määratakse suhteline tee kindlaks selle kausta suhtes, kus käsk käivitati; kaugtagapõhi puhul määratakse tee kindlaks **tagapõhja käitaval arvutil**. |
| `--device-name NAME` | — | Salvestusega koos salvestatud silt. |
| `--stop` | — | Peata käimasolev salvestus. |
| `--confirm-after SEC` | `20` | Pärast käivitamist oodatakse kuni selle aja jooksul esimese rea kirjutamist ning kui seda ei toimu, antakse vali hoiatus (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`). `0` lülitab kontrolli välja. |
| `--heartbeat SEC` | `30` | `--duration` abil trükita ridade arv iga SEC sekundi järel välja ja anna hoiatus, kui see enam ei kasva. `0` lülitab selle välja. |

{% hint style="info" %}
Salvestamine toimub tagapõhiselt, seega salvestatakse fail `.daq` **tagapõhise masina** failisüsteemi — vaikimisi asub see seal, mitte tingimata seal, kus sa CLI-d käivitasid. Failinimed sisaldavad anduri ID-d ja ajamärget.
{% endhint %}

## `pool-set-cap` — paigaldatud korgi deklareerimine

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Katte ID valib tehases mõõdetud korrigeerimisprofiili, mida rakendatakse igale spektrile, ning see **peab vastama andurile füüsiliselt paigaldatud katele** — ei andur ega tarkvara suuda katet ise tuvastada ning valik salvestatakse igasse `.daq`-faili. Vaikimisi on kõikjal kasutusel `sunshine_cosine` (iga DAQ-seade tarnitakse koos paigaldatud Sunshine'i kosinus-korrektoriga, mille kavandatud sumbumine on ~12× — deklareerimata kate vahetus korrigeerib spektreid valesti ligikaudu sama teguriga).

| `--cap-id` | Saadaval |
| --- | --- |
| `sunshine_cosine` (vaikimisi) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | ainult DAQ-U |
| `none` | ainult DAQ-E — vt märkust |

Anduri komplektist väljaspool olev kate ID lükatakse ühendamisel selge veateatega tagasi. `none` (DAQ-E) tähendab, et kate on füüsiliselt eemaldatud — see rakendab endiselt DAQ-E süvendatud klaasdiffuusori tehasegeomeetriaprofiili, seega ei ole tegemist tühitoiminguga, ning paljas DAQ-E on laborikonfiguratsioon, mitte toetatud välitingimustes kasutatav konfiguratsioon. (Paljas DAQ-U on tõeliselt paljas ja ei vaja üldse korrigeerimisprofiili; DAQ-M-i kasutatakse koos selle Sunshine-korgiga.)

## `pool-disconnect` — andurite vabastamine

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` vabastab anduri, kes iganes selle avas, seega vabastab see ka anduri, mille kukkunud skript on jätnud andurite reservi (kui salvestamine sellel on esmalt lõpetatud). Anduril, mis ei ole andurite reservis, pole midagi vabastada: käsk väljastab `<sensor-id>: already released (not in the pool)` ja lõpetab töö `0`. Vabastamine, mida tagapõhi keeldus teostamast või mida ta ei kinnitanud, lõpetab töö `1`.

## `pool-enable-ptp` — lülita DAQ-E PTP uuesti sisse

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

DAQ-E säilitab oma PTP sisse-/väljalülitamise seadistuse oma mälus, seega seade, mille PTP oli välja lülitatud, jääb välja lülitatuks — ja selle ajamärgised ei ole PTP-ga sünkroniseeritud — kuni see uuesti sisse lülitatakse. `pool-list` märgistab sellise seadme ja kuvab selle jaoks käesoleva käsu. Chloros ei lülita seadet kunagi ise tagasi sisse, kuna väljalülitamine võib olla tahtlik. Uus versioonis 1.3.0.

## DAQ-E seadme seaded

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` näitab anduri püsivara (ja seda, kas käesolev Chlorosi versioon sisaldab uuemat), selle aadressi ja selle saamise viisi, salvestatud võrguseadistusi, kui need erinevad hetkel kasutatavatest, sisselülitamisel toimuvat andmeedastust ja PTP-d. `pool-set-network` taaskäivitab anduri, et muudatus jõustuks, ja ühendab selle uuesti varasemate seadistustega; `--no-reboot` salvestab need vaid. Sellel arvutil peab olema staatiline aadress, vastasel juhul lükkab Chloros selle tagasi; `--force` tühistab selle piirangu, kuid ainult arvuti puhul, millele määratakse uus aadress või mis suunatakse sellele. Taaskäivitamine ei ole lubatud, kui andur salvestab andmeid. `pool-update-firmware` on graafilise kasutajaliidese (GUI) käsk **Update to \<version\>** vastand käskureale ning viis DAQ-E uuendamiseks Linuxis, kus graafilist kasutajaliidest pole. Uus versioonis 1.3.0.

***

## Käskude kokkuvõte

| Käsk | Otstarve |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Avab anduri tagapõhja-poolis. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Otsib andureid ühendamata (leiab DAQ-M-i BLE MAC-aadressi). |
| `daq pool-list` | Näita kõiki andureid anduripoolis koos nende `sensor_id`-iga. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Viimased N kalibreeritud spektriraami. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Jätka / peata voogedastus. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Alusta / lõpeta `.daq` salvestamine (tagapõhja poolel). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Vaheta cap-korrektsiooni profiili käitamise ajal. |
| `daq pool-disconnect --sensor-id ID [--all]` | Vabasta üks andur või kõik andurid. |
| `daq pool-enable-ptp --sensor-id ID` | Lülita PTP uuesti sisse DAQ-E-seadmel, mille PTP on välja lülitatud. |
| `daq pool-info --sensor-id ID [--json]` | DAQ-E-seadme püsivara, aadress, salvestatud võrguseaded, sisselülitamisel algav voogedastus ja PTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP või staatiline aadress; taaskäivitab anduri ja ühendab selle uuesti. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Andmete edastamine sisselülitamisel. |
| `daq pool-reboot --sensor-id ID` | Käivita DAQ-E uuesti ja ühenda see uuesti. |
| `daq pool-update-firmware --sensor-id ID` | Lae selle versiooni DAQ-E püsivara vanemale andurile. |

***

## Esimese DAQ-E ühenduse veaotsing

1. DAQ-E-l puudub oleku-LED — kontrollige toite olemasolu lüliti või injektori pordi PoE/link-indikaatori abil ning oodake pärast sisselülitamist mõni sekund, kuni seade käivitub ja ühineb võrguga.
2. Tagapõhjaarvuti peab asuma **samal levidomeenil** kui andur — mDNS ei läbi ruutereid.
3. Tulemüür peab lubama läbi mDNS-i (UDP 5353), DAQ-E andmeid (UDP 5002) ja PTP-d (UDP 319/320). Windowsi paigaldaja lisab selleks reegli ning Linuxi `.deb` avab need `ufw` / `firewalld`-is, mis oli paigaldamise ajal aktiivne; kolmanda osapoole tulemüür või hiljem aktiveeritud tulemüür peab neid lubama.
4. Ikka veel pole `--eth`-ist midagi kuulda? Kasutage `--eth-host` koos seadme hostinimega (`daq-e-<id>.local`) või IP-aadressiga, mis jätab avastamise vahele.

***
{% hint style="info" %}
**Nõuanne AI-abilistele.** Kõik käesoleva juhendi leheküljed on esitatud toores Markdown-vormingus — lisage `.md` lehekülje väiketähelisele URL-slugile (käesolev lehekülg: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); masinloetav indeks on `https://mapir.gitbook.io/chloros/llms.txt`. `chloros-cli daq` ja kõigi teiste käskude perekondade täieliku liputaseme dokumentatsiooni saamiseks laadige alla [CLI-viide](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Pythoni tee on `chloros_sdk.connect_daq_sensor()` [SDK-viites](../reference/sdk-reference.md).
{% endhint %}
