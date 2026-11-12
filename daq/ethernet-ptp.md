# DAQ-E võrgustik ja ajasünkroniseerimine

> Anduri füüsiline võrguseadistus – kaabeldus, PoE, IP-aadressi määramine ja seadme enda võrguseaded – on kirjeldatud **[DAQ-kasutusjuhendis](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Käesolev lehekülg käsitleb Chloros i poolt: ühendamine, ajasünkroniseerimine ja mida teha, kui seadme avastamine ei õnnestu.

DAQ-E on DAQ-perekonna Ethernet-liige: toidetakse PoE kaudu, leitakse mDNS-i kaudu (teenus `_daq-e._tcp`) ja sellele saab viidata anduri ID-st tuletatud hostinimega — `daq-e-<6 hex>.local`, nt `daq-e-def330.local`. Käesolev lehekülg käsitleb, kuidas seade edastab andmeid võrgus ja kuidas osaleb PTP-ajasünkroniseerimises.

## Ülekanderežiimid

| Režiim | Lõpppunkt | Tarbijad | Märkused |
| --- | --- | --- | --- |
| **Multicast** (vaikimisi) | UDP `239.10.10.10:5002` | Iga sama LAN-võrgus olev seade võtab vastu sama voogu | Iga datagramm on CRC-16/CCITT-ga valideeritud |
| **Raw** | TCP-port `5000` | Täpselt üks klient (eksklusiivne) | Baiditasandil ühilduv DAQ-U-ga |

Chloros kasutab vaikimisi multisaadet, mis võimaldab graafilisel kasutajaliidesel (GUI), käsurealiidesel (CLI) ja arendajate tööriistakomplektil (SDK) jälgida ühte andurit korraga.

## Võrgunõuded

* **Sama levidomeen.** Chloros'i käitav arvuti peab asuma samas L2-võrgusegmendis kui andur — mDNS-otsing ei läbi ruutereid.
* **Tulemüür.** Windowsis lisab installija igasse võrguprofiili backendile tulemüürireegli **Chloros Backend (UDP-In)**, mis hõlmab DAQ-E andmeid (UDP 5002), mDNS-i (UDP 5353) ja PTP-d (UDP 319/320), ning**„Chloros Backend (TCP-In)“** reegli, mis on piiratud kohaliku alamvõrguga ja on mõeldud ühenduseks, mille DAQ-E loob arvutiga tagasi püsivara uuendamise ajal. Linuxis avab `.deb` UDP-pordid `ufw`-is või `firewalld`-is, kui see tulemüür on paigaldamise ajal aktiivne. Kolmanda osapoole tulemüür või hiljem aktiveeritud tulemüür vajab samade portide lubamist.
* **PoE-toide, puudub oleku-LED.** DAQ-E-l puudub oma LED – kontrollige toite olemasolu lüliti või injektori pordi link/PoE-indikaatori abil ning oodake pärast sisselülitamist mõni sekund, kuni seade käivitub ja ühineb võrguga.

## Ühendamine

**GUI:** Vahekaart „Valgusandurid“ → „Ühenda andur“ → Seadmetüüp „DAQ-E (Ethernet)“. Andurite otsimine toimub ainult siis, kui ühendusdialoog on ekraanil (mDNS-otsing ja ARP-skaneerimine Windowsis), kordudes iga 15 sekundi järel; nupp „Värskenda“ käivitab uue skaneerimise kohe. Leitud andurid kuvatakse rippmenüüs; esimene tuvastatud andur valitakse automaatselt.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (tagapõhi töötab):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Juba andmepoolis olev DAQ-E tuvastatakse nii selle hostinime kui ka IP-aadressi järgi: selle uuesti ühendamine teise aadressi all toob tagasi olemasoleva andmepooli kande, mitte ei ava seadet teist korda.

### Mitme võrgukaardiga hostid ja esimene ühendus pärast käivitamist

Hostidel, millel on rohkem kui üks aktiivne võrguliides, võib **esimene** `pool-connect --eth` pärast käivitamist jääda tühjaks isegi siis, kui andur on töökorras — avastamisotsing võib jätta märkamata liidese, millel andur asub, kui ARP-vahemälu on veel tühjaks jäänud. Usaldusväärne lahendus on avastamine vahele jätta ja aadress selgesõnaliselt edastada:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` aktsepteerib mDNS-i hostinime või IP-aadressi, suunab alati õigele andurile ning on soovitatav vorm skriptide ja peata installatsioonide puhul. Graafilises kasutajaliideses kasutage ühendusdialoogi nuppu „Värskenda” ja laske skannimistsükkel uuesti läbi.

## Seadme seaded ja püsivara

Anduril endal on oma võrguseaded: DHCP (tehase seade, mis kasutab tagavarana 169.254.x.x link-local-aadressi, kui ükski DHCP-server ei vasta) või staatiline aadress. Samuti on seal määratud, kas andur alustab andmete edastamist kohe sisselülitamisel. Muutke neid graafilises kasutajaliideses (anduri hammasratta ikooniga seaded, **Võrk**:**Muuda…**,**Andmevoog sisselülitamisel**,**Anduri taaskäivitamine**; vt [vahekaart „DAQ”](gui.md)) või CLI- ja SDK-liidese kaudu käskudega `pool-set-network`, `pool-set-auto-stream` ja `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; vt [CLI-juhendit](../reference/cli-reference.md#daq-e-device-settings)). Andur taaskäivitub võrgumuudatuse rakendamiseks ja Chloros ühendab selle uuesti. Chloros i käitava arvuti võrgus peab olema staatiline aadress: Chloros lükkab tagasi aadressi, milleni ta pärast taaskäivitamist ei pääse. Anduri ID ja selle `daq-e-<id>.local` hostinimi ei muutu kunagi. Uus versioonis 1.3.0.

**Püsivara uuendused on integreeritud graafilisse kasutajaliidesesse.** Kui ühendatud DAQ-E-seadmel on vanem püsivara kui teie Chloros-koostisega kaasas olev püsivara, kuvatakse selle anduri real kollane **Uuendus saadaval**-ikoon ning hammasratta-seadete aknas on nupp „Uuenda versioonile \<version\>“. Uuendus laaditakse võrgu kaudu umbes 30 sekundiga; andur taaskäivitub ja ühendub automaatselt uuesti ning katkestatud ülekande korral jääb praegune püsivara muutumatuks. CLI-st või SDK-st ning Linux-s, kus kasutajaliidest pole, käivitage käsk `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-aja sünkroniseerimine

DAQ-E püsivara versioon v1.2.0+ osaleb IEEE 1588 PTPv2-s tavalise (ainult orjana toimiva) kellana. **Chloros-i hosti backend on PTP grandmaster** — iga DAQ-E ja iga LATTICE-kaamera LAN-võrgus on selle orjad domeenis 0, hoides kõigi seadmete ajamärke ~1 ms tolerantsi piires. Just see ühine kell võimaldab DAQ-i näidud ajamärgistega kaamera säriaegadega kokku viia (vt [Salvestamine ja .daq-vorming](recording.md)).

Sünkroniseerimise kontrollimine CLI kaudu:

| Käsk | Näitab |
| --- | --- |
| `chloros-cli time-sync status` | Põhiserveri seisund, BMCA prioriteedid, kella identiteet ning `health`: `OK` või `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` koos `health_detail`-iga, mis selgitab, miks selle paketid võrku ei jõua |
| `chloros-cli time-sync peers` | Kõik tuvastatud allseadmed (DAQ-E andurid + LATTICE kaamerad) |
| `chloros-cli time-sync cameras` | Iga kaamera PTP-töökorrasolek (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Grandmaster-protsessi taaskäivitamine |

Graafilises kasutajaliideses kuvab DAQ-E seadete aken reaalajas **PTP Sync** rida, milles on näha anduri hetke PTP-seisund.**DAQ-E, mille PTP on välja lülitatud, jääb välja lülitatuks.** Seade säilitab oma PTP sisse-/väljalülitamise seadistuse oma mälus, seega seade, mille PTP oli välja lülitatud, jääb välja lülitatuks ka pärast uuesti ühendamist või toite väljalülitamist ja sisselülitamist, ning selle ajamärgised ei ole PTP-ga sünkroniseeritud. `chloros-cli daq pool-list` märgistab sellise seadme (`!! PTP is switched OFF on this unit …`) ja kuvab selle parandamiseks vajaliku käsu:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros ei lülita seadet kunagi ise tagasi sisse, kuna väljalülitatud olek võib olla tahtlik.

Täpsem teave ranget sünkroniseerimist nõudvatele kasutajatele:

* Igal edastataval datagrammil on lipuväli; **bit 2 on seatud raamides, mille ajamärge on PTP-sünkroniseeritud**. Torustikud, mis nõuavad ranget kaamera/DAQ-i sünkroniseerimist, peaksid selle biti alusel andmevoogu filtreerima.
* Enne sünkroniseeritud salvestamist veenduge, et andur kuvatakse `chloros-cli time-sync peers`-is ja et `chloros-cli daq pool-list` ei märgi selle PTP-d väljalülitatuks. (MAPIRi sisemine otse riistvarale suunatud tööriist võib salvestamist PTP-lukustuse alusel filtreerida ka `--wait-ptp` lipuga, mis ootab kuni 15 sekundit, kuni andur jõuab SLAVE-olekusse; see tööriist ei kuulu tarnitud CLI-i koosseisu.)
* Kui PTP töötab aktiivselt orjana, keeldub andur käsitsi kella sünkroniseerimisest („PTP pakub kella”). See on nii kavandatud – usaldage PTP-d.

## Linuxi märkused

* **PTP-pordid ilma root-õigusteta.** PTP seob end UDP-portidega 319/320. `.deb` kirjutab `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), seega seob tagapõhi need teie kasutajana töötades. Alates versioonist 1.3.0 ei määra pakett backendile ühtegi failiõigust (`setcap`) ega vaja `libcap2-bin`-i; ärge lisage õigust ise, sest siis ei käivitu lahtipakitud backend. Kui PTP ei suuda seostada porti 319, vaadake [Linuxi paigaldamine](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Headless Jetson / Linux-host:** pakett installib backend-teenused, kuid ei aktiveeri ühtegi neist. PTP pidevaks töötamiseks (ja DAQ kättesaadavuseks) ilma, et keegi käsku käivitaks, aktiveerige teenus oma kasutajana:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Ilma selleta töötab PTP ainult siis, kui midagi on tagapõhja käivitanud — käsk `chloros-cli` või SDK-skript. Vaata [Linuxi installimist](../linux/linux-installation.md#always-on-ptp-for-headless-hosts), et saada teavet root-kasutaja `chloros-backend.service` kohta ja selle kohta, kuidas sellest välja lülitada.

## Veaotsing: „DAQ-E-seadmeid ei leitud”

| Kontroll | Detail |
| --- | --- |
| Toide | Anduril ei põle LED – kontrollige lüliti/injektori pordi PoE- ja ühenduse indikaatoreid; oodake pärast sisselülitamist paar sekundit |
| Levidomeen | Host ja andur asuvad samas L2-segmendis; mDNS ei suuna |
| Tulemüür | Windows: installiprogrammi reegel **Chloros Backend (UDP-In)** lubab sisse DAQ-E-voogu (UDP 5002), mDNS-i (5353) ja PTP-d (319/320); kolmanda osapoole tulemüür peab tegema sama. Linux: `.deb` avab need pordid `ufw` / `firewalld`-is ainult juhul, kui tulemüür oli paigaldamise ajal aktiivne — lubage need ise hiljem sisse lülitatud tulemüüris |
| Mitme võrgukaardiga host | Avastamine otsib mDNS-i, kasutades ARP-d varuvõimalusena; `--eth-host <ip-or-hostname>` jätab avastamise vahele ja ühendub otse |
| GUI uuestiotsing | Avastamine töötab ainult siis, kui ühendusdialoog on avatud; kasutage nuppu „Värskenda“ |
