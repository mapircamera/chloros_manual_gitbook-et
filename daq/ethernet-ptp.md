# DAQ-E võrgustik ja ajasünkroniseerimine

> Anduri füüsiline võrguseadistus – kaabeldus, PoE, IP-aadressi määramine ja seadme enda võrguseaded – on kirjeldatud **[DAQ-kasutusjuhendis](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Käesolev lehekülg käsitleb Chloros-i: ühendamine, aja sünkroniseerimine ja mida teha, kui seadme avastamine ei õnnestu.

DAQ-E on DAQ-seeria Ethernet-seade: see saab toidet PoE kaudu, seda leitakse mDNS-i kaudu (teenus `_daq-e._tcp`) ning sellele saab viidata anduri ID-st tuletatud hostinimega — `daq-e-<6 hex>.local`, nt `daq-e-def330.local`. Käesolev lehekülg käsitleb, kuidas seade edastab andmeid võrgus ja osaleb PTP-aja sünkroniseerimises.

## Ülekanderežiimid

| Režiim | Lõpppunkt | Tarbijad | Märkused |
| --- | --- | --- | --- |
| **Multicast** (vaikimisi) | UDP `239.10.10.10:5002` | Iga seade samas LAN-võrgus võtab vastu sama voo | Iga datagramm on CRC-16/CCITT-ga valideeritud |
| **Raw** | TCP-port `5000` | Täpselt üks klient (eksklusiivne) | Vahetult ühilduv DAQ-U-ga baitide tasandil |

Chloros kasutab vaikimisi multisaadet, mis võimaldab graafilisel kasutajaliidesel, CLI-il ja SDK-il jälgida ühte andurit korraga.

## Võrgu nõuded

* **Sama leviala.** Chloros-i käitav arvuti peab asuma samas L2-võrgusegmendis kui andur — mDNS-otsing ei läbi ruutereid.
* **Windows tulemüüri küsimus: nõustu sellega.** Kui Chloros seob esimest korda multisaatepistikupesad, küsib Windows Defender seda üks kord. Selle lubamine hõlmab DAQ-E andmeid (UDP 5002), mDNS-i (UDP 5353) ja PTP-d (UDP 319/320). Linux puhul toimub see märkamatult.
* **PoE-toide, puudub oleku-LED.** DAQ-E-l puudub oma LED – kontrollige toidet lüliti või injektori pordi link/PoE-indikaatori kaudu ning oodake pärast sisselülitamist paar sekundit, kuni seade käivitub ja ühineb võrguga.

## Ühendamine

**GUI:** vahekaart „Valgusandurid“ → „Ühenda andur“ → seadmetüüp „DAQ-E (Ethernet)“. Avastamine toimub ainult siis, kui ühendusdialoog on ekraanil (mDNS-otsing ja ARP-skaneerimine aadressil Windows), kordudes iga 15 sekundi järel; nupp „Värskenda“ käivitab uue skaneerimise kohe. Avastatud andurid ilmuvad rippmenüüsse; esimene avastatud andur valitakse automaatselt.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (taustaprogramm töötab):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Mitme võrgukaardiga arvutid ja esimene ühendus pärast käivitamist

Hostidel, millel on rohkem kui üks aktiivne võrguliides, võib **esimene** `pool-connect --eth` pärast käivitamist jääda tühjaks isegi siis, kui andur on töökorras — avastamisotsing võib jätta märkamata liidese, millel andur asub, kui ARP-vahemälu on veel külm. Usaldusväärne lahendus on avastamine vahele jätta ja aadress selgesõnaliselt edastada:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` aktsepteerib mDNS-i hostinime või IP-aadressi, suunab alati õige anduri poole ja on soovitatav vorm skriptide ja peata installatsioonide jaoks. Graafilises kasutajaliideses kasuta ühendusdialoogi nuppu „Värskenda“ ja lase skannimistsükkel uuesti läbi.

## Seadme seaded ja püsivara

Andur ise hoiab võrguseadeid – staatiline IP vs. DHCP + link-local aadressimine, seadme nimi, automaatne voog käivitamisel, OTA-parool. Neid seadmepoolseid seadeid ei ole tarnitud CLI-is käskudena avatud; neid hallatakse Chloros-i graafilise kasutajaliidese kaudu, kus need on kuvatud, või MAPIR-i toe abil.

**Püsivara uuendused on sisse ehitatud kasutajaliidesesse.**Kui ühendatud DAQ-E-seadmel on vanem püsivara kui teie Chloros-koostisega kaasas olev pilt, kuvatakse selle anduriribal kollane**Värskendus saadaval** ikoon ning hammasratta-seadete aknas on<version>

nupp</version> „Värskenda versioonile<version>

“. Uuendus laaditakse võrgu kaudu umbes 30 sekundiga; andur taaskäivitub ja ühendub automaatselt uuesti ning ülekande katkemisel jääb praegune püsivara muutmata.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-aja sünkroniseerimine

DAQ-E püsivara versioon 1.2.0+ osaleb IEEE 1588 PTPv2-s tavalise (ainult orjana toimiva) kellana. **Chloros-i hosti tagapõhi on PTP-grandmaster** — iga DAQ-E ja iga LATTICE-kaamera LAN-võrgus on selle orjad domeenis 0, hoides kõikide seadmete ajamärke ~1 ms tolerantsi piires. Just see ühine kell võimaldab DAQ-i näidud ajamärgistega kaamera säriaegadega kokku viia (vt [Salvestamine ja .daq-vorming](recording.md)).

Kontrollige sünkroniseerimist CLI-ist:

| Käsk | Näitab |
| --- | --- |
| `chloros-cli time-sync status` | Põhiserveri (grandmaster) olek, BMCA prioriteedid, kella identiteet |
| `chloros-cli time-sync peers` | Kõik tuvastatud allseadmed (DAQ-E andurid + LATTICE-kaamerad) |
| `chloros-cli time-sync cameras` | Iga kaamera PTP-töökorrasolek (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Grandmaster-protsessi taaskäivitamine |

Graafilises kasutajaliideses kuvatakse DAQ-E seadete aknas reaalajas **PTP Sync** rida, mis näitab anduri hetke PTP-seisundit.

Täpsed juhised ranget sünkroniseerimist nõudvatele tarbijatele:

* Igal edastataval datagrammil on lipuväli; **bit 2 on seatud raamides, mille ajamärge on PTP-sünkroniseeritud**. Torustikud, mis nõuavad kaamera ja andmekogumise seadme ranget sünkroniseerimist, peaksid töötama selle biti alusel.
* Enne sünkroniseeritud salvestamist veenduge, et andur on loetletud `chloros-cli time-sync peers`-is. (MAPIR-i sisemine otsene riistvaraline tööriist võib salvestamist PTP-lukustuse alusel piirata ka `--wait-ptp`-i lipuga, mis ootab kuni 15 sekundit, kuni andur jõuab SLAVE-olekusse; see tööriist ei kuulu tarnitud CLI-i komplekti.)
* Kui PTP töötab aktiivselt orjana, keeldub andur käsitsi kella sünkroniseerimisest („PTP annab kella”). See on nii kavandatud – usaldage PTP-d.

## Linux märkused

* **PTP vajab paigaldamisel `libcap2-bin`-i.** `.deb` postinst annab `cap_net_bind_service=+ep`-ile õigused `/usr/lib/chloros/chloros-backend`-ile, et see saaks siduda PTP-pordid 319/320 ilma root-õigusteta. Kui `libcap2-bin` puudub, jäetakse see samm vahele ja PTP ei õnnestu käivitada. Lahendus:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Headless Jetson / Raspberry Pi:** esimesel installimisel luuakse systemd-üksus `chloros-backend.service`, kuid seda ei aktiveerita. PTP (ja DAQ-i kättesaadavuse) pidevaks töötamiseks ilma graafilise kasutajaliideseta:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Ilma selleta töötab PTP ainult siis, kui graafiline kasutajaliides Chloros on avatud.

## Veaotsing: „DAQ-E-seadmeid ei leitud”

| Kontroll | Detail |
| --- | --- |
| Toide | Anduril ei põle LED-tuli — kontrollige lüliti/injektori pordi PoE- ja ühenduse indikaatoreid; oodake pärast sisselülitamist mõned sekundid |
| Levidomeen | Host ja andur asuvad samas L2-segmendis; mDNS ei suuna |
| Windows tulemüür | Nõustu Defenderi küsitlusega esmakordsel käivitamisel (UDP 5002, 5353, 319/320) |
| Mitme võrgukaardiga host | Esimene avastamine pärast käivitamist võib andurit mitte leida — ühenda `--eth-host <ip-or-hostname>`-iga |
| GUI uuesti skannimine | Avastamine toimub ainult siis, kui ühendusdialoog on avatud; kasuta nuppu „Värskenda“ |</version>
