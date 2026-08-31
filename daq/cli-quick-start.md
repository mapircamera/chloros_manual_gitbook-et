# CLI Kiirjuhend (pool-*)

Kaasasolev `chloros-cli` juhib DAQ-andureid **`daq pool-*`** käskude perekonna kaudu — need on kerged HTTP-kliendid, mis juhivad andurit Chloros-tagapõhja püsiva andurite reservi kaudu. Tagapõhi haldab andmeedastust, mistõttu jagavad graafiline kasutajaliides (GUI), skriptid CLI ja SDK ühte aktiivset käepidet, selle asemel et võidelda pordi pärast. Kõik, mida klient vajab, on kättesaadav `pool-*`i kaudu: ühendamine, andmevoog, kalibreeritud `.daq`-failide salvestamine ja kap-profiilide vahetamine.

`pool-*` on ka **ainus** DAQ-pind väljalastud versioonides. `chloros-cli daq --help` loetleb `pool-*` alamkäsklusi, ning kui käivitada väljalastud versioonis otsene riistvarapõhine DAQ-alamkäsk, lõpetatakse töö selge veateatega, milles nimetatakse puuduvat paketti ja suunatakse teid tagasi `pool-*` juurde – midagi ei ebaõnnestu märkamatult. (Otse riistvarale suunatud käsud töötavad ainult MAPIR lähtekoodi väljavõttest; ka `pip install chloros-sdk` neid ei paku.)

***

## Eeltingimused

* **Chloros-tagapõhi peab töötama** — `pool-*`-käskud on HTTP-kliendid, mitte riistvaradraiverid. Windows-is käivita töölaua rakendus Chloros (see käivitab backend&#x27;i). Kuvarieta Linux/Jetsonil aktiveerige teenus: `sudo systemctl enable --now chloros-backend.service`.
* **Chloros+ (tasuline tasand) sisselogimine**: käivita esmalt `chloros-cli login`. Järelevalve toimub serveri poolel – ilma sisselogimiseta ebaõnnestuvad käsud `401 AUTH_REQUIRED`-iga; tasuta (Iron) tasemel ebaõnnestuvad need `403 PLAN_UPGRADE_REQUIRED`-iga.
* Käskudele on vaikimisi määratud sihtmärgiks `http://127.0.0.1:5000`; `daq pool-*`-perekond arvestab keskkonnamuutujat `CHLOROS_BACKEND_URL`, kui teie tagapõhi töötab mujal.

***

## Viieminutiline seanss

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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

## `pool-connect` — ava andur andurite kogumist

| Variant | Tähendus |
| --- | --- |
| `daq pool-connect` | Smart-detect: leia kõik selle masina DAQ-seadmed. |
| `daq pool-connect --port PORT` | DAQ-U kindlal jadaportil (nt `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M BLE kaudu, MAC-aadress automaatselt skannitud. |
| `daq pool-connect --mac MAC` | DAQ-M teadaoleva BLE MAC-aadressiga (tähendab `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E teadaoleva hostinime või IP-aadressiga — **usaldusväärne tee**. |
| `daq pool-connect --eth` | DAQ-E automaatse avastamisega (mDNS, ARP-varuvõimalusega). Vaata allpool esitatud hoiatust. |

Häälestuslipud, kõik valikulised:

| Lipu kood | Tähendus |
| --- | --- |
| `--integration-time MS` / `-t MS` | Käsitsi määratud integratsiooniaeg millisekundites. |
| `--frame-avg N` / `-f N` | Keskmistatud kaadrite arv iga teatatud spektri kohta. |
| `--no-ae` | Lülita automaatne säritus välja (AE on vaikimisi sisse lülitatud). |
| `--no-stream` | Ühendus ilma voo käivitamiseta (jätka hiljem käsuga `pool-stream --start`). |
| `--cap-id CAP` | Cap-korrektsiooni profiil; tagapõhja vaikimisi väärtus on `sunshine_cosine`. Vaata [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**`--eth` automaatse avastamise hoiatus.** Mitme võrguliidesega hostil (rohkem kui üks aktiivne võrguliides) võib *esimene* `pool-connect --eth` pärast käivitamist jääda tühjaks isegi siis, kui andur on töökorras — avastamisotsing võib anduri liidese vahele jätta, kui ARP-vahemälu on tühi. Kui `--eth` midagi ei leia, proovi uuesti või jäta avastamine täielikult vahele, kasutades `--eth-host <ip-or-hostname>`-i, mis on mitme võrguliidesega masinatel usaldusväärne lahendus. DAQ-E hostinimi on `daq-e-<id>.local` (nt `daq-e-def330.local`); ka selle tavaline IP-aadress sobib.
{% endhint %}

## `pool-list` — vaata, mis on ühendatud

Näitab kõiki tagapoolse puli andureid, sealhulgas `sensor_id`-i, mida kõik muud käsud vajavad:

| Mudel | `sensor_id` vorm | Näide |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktettiline, sidekriipsuga | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — spektri raamide lugemine

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Tagastab viimase kaadri või viimased `--recent N` kaadrid; `--json` väljastab masinloetavat väljundit skriptide jaoks. Raamid on radiomeetriliselt kalibreeritud spektraalne kiirgustihedus (W/m²/nm) 135-punktilisel, 340–1010 nm võrgustikul, millele on juba rakendatud anduri kateprofiil. Kvantitatiivsete kiirgustiheduste saamiseks arvutage vähemalt 15 sekundi pikkuste kaadrite keskmine – see on seadme omadus, mitte defekt.

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
| `--duration SEC` / `-d SEC` | `0` | Salvestuse pikkus sekundites; `0` tähendab, et käivita kuni käskluse `--stop` andmiseni. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Väljundkataloog, määratakse kindlaks **tagapõhja käitaval masinal**. |
| `--device-name NAME` | — | Salvestusega koos salvestatud silt. |
| `--stop` | — | Peatab käimasoleva salvestuse. |

{% hint style="info" %}
Salvestamine toimub backendis, seega salvestatakse fail **backend-masina** failisüsteemi — vaikimisi asub see seal, mitte tingimata seal, kus te CLI käivitasite. Failinimed sisaldavad anduri ID-d ja ajamärget.
{% endhint %}

## `pool-set-cap` — paigaldatud kate deklareerimine

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Katte ID valib tehases mõõdetud korrigeerimisprofiili, mida rakendatakse igale spektrile, ning see **peab vastama andurile füüsiliselt paigaldatud katele** — ei andur ega tarkvara suuda katet ise tuvastada ning valik salvestatakse igasse `.daq`-faili. Kõikjal on vaikimisi seadistuseks `sunshine_cosine` (iga DAQ tarnitakse koos paigaldatud Sunshine&#x27;i kosinus-korrektoriga, mille disainiga on ette nähtud umbes 12-kordne sumbumine — deklareerimata kate vahetus korrigeerib spektreid valesti ligikaudu selle teguri võrra).

| `--cap-id` | Saadaval |
| --- | --- |
| `sunshine_cosine` (vaikimisi) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | ainult DAQ-U |
| `none` | ainult DAQ-E — vt märkust |

Anduri komplektist väljaspool olev kate ID lükatakse ühendamisel selge veateatega tagasi. `none` (DAQ-E) tähendab, et kate on füüsiliselt eemaldatud — see rakendab endiselt tehase geomeetriaprofiili DAQ-E süvendatud klaasdifuusori jaoks, seega ei ole tegemist tühitoiminguga ning paljas DAQ-E on laborikonfiguratsioon, mitte toetatud välitingimustes kasutatav konfiguratsioon. (Paljas DAQ-U on tõeliselt paljas ja ei vaja üldse korrigeerimisprofiili; DAQ-M-i kasutatakse koos selle Sunshine-korgiga.)

## `pool-disconnect` — andurite vabastamine

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Käskude kokkuvõte

| Käsk | Otstarve |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Ava andur tagapoolse andurite kogumis. |
| `daq pool-list` | Näita kõiki andureid anduripoolis koos nende `sensor_id`-iga. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Viimased N kalibreeritud spektriraami. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Jätka / peata voogedastust. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Alusta / lõpeta `.daq` salvestamine (tagapõhja poolel). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Vaheta cap-korrektsiooniprofiili töötamise ajal. |
| `daq pool-disconnect --sensor-id ID [--all]` | Vabasta üks andur või kõik andurid. |

***

## Esimese DAQ-E ühenduse veaotsing

1. DAQ-E-l puudub oleku-LED — kontrollige toite olemasolu lüliti või injektori pordi PoE/link-indikaatori kaudu ning oodake pärast sisselülitamist mõni sekund, kuni seade käivitub ja ühineb võrguga.
2. Tagapõhjaarvuti peab asuma **samal levidomeenil** kui andur — mDNS ei läbi ruutereid.
3. Seadmel Windows tuleb esmakordsel käivitamisel nõustuda Defender-tulemüüri küsimusega (mDNS UDP 5353, DAQ-E andmed UDP 5002, PTP UDP 319/320).
4. `--eth`-ist ikka veel midagi ei tule? Kasutage `--eth-host`-i koos seadme hostinimega (`daq-e-<id>.local`) või IP-aadressiga – see on usaldusväärsem viis, eriti mitme ühendusega hostide puhul.

***{% hint style="info" %}**Nõuanne AI-abilistele.** Käesoleva juhendi iga lehekülg on esitatud toores Markdown-vormingus — lisage `.md` lehekülje väiketähelisele slugile URL (käesolev lehekülg: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); masinloetav indeks on `https://mapir.gitbook.io/chloros/llms.txt`. `chloros-cli daq` ja kõigi teiste käskude perekondade täieliku liputaseme dokumentatsiooni saamiseks laadige alla [CLI viide](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Python asukoht on `chloros_sdk.connect_daq_sensor()` [SDK viites](../reference/sdk-reference.md).
{% endhint %}
