# Chloros-i kasutamine koos tehisintellekti abilistega

Käesolev juhend on mõeldud kahele sihtrühmale: inimestele ja tehisintellekti abilistele, kelle kaudu inimesed üha enam töötavad. Igal lehel on esitatud täpsed väärtused, vaikimisi seaded ja kopeeritavad-kleebitavad käsud, et assistent (Claude, ChatGPT, Copilot, programmeerimisagent jne) saaks juba esimesel katsel kirjutada toimiva Chloros-automaatika.

Chloros versioon: **

1.2.0**. CLI/SDK platvormid: Windows 10/11 x64 ja Linux (x86_64 / Jetson aarch64).

## Mida oma assistendile anda

| Ressurss | URL | Milleks see on |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Käesoleva juhendi iga lehekülje masinloetav indeks. |
| **CLI Viide** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Täielik `chloros-cli` käskude loend: kõik käsud, lipud, vaikimisi seaded, väljumiskoodid ja väljundkausta reeglid. Koostatud masinõppe (LLM) jaoks. |
| **SDK viide** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Täielik `chloros_sdk` Python API: klassid, signatuurid, erandid ja lahendatud näited. Kirjutatud LLM-i kasutamiseks. |
| **Iga lehekülg toores Markdown-vormingus** | lisa `.md` leheküljele URL | nt `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` tagastab lehe toores Markdown-vormingus — ideaalne kontekstiaknasse kleepimiseks või agendist hankimiseks. |

Juhendisisesed lingid: [CLI Viide](reference/cli-reference.md) · [SDK Viide](reference/sdk-reference.md).

{% hint style="info" %}
Need kaks viitelehte on iseseisvad: assistent, kes on ühe neist läbi lugenud, ei vaja käsiraamatu ülejäänud osa, et kirjutada õige skript.
{% endhint %}

## Kiirjuhendid

Kopeerige, täitke `<placeholders>` ja kleepige see oma assistenti.

### 1. Lennukausta töötlemine NDVI-iks

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Jälgi salvestuste kataloogi partiide kaupa

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Ühenda LATTICE-maatriks ja tee salvestus

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Salvestage DAQ-valgusanduri spektrid

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
DAQ-skriptide käivitamine käsurealt toimub alati `daq pool-*`-seeria kaudu (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Muud `daq` alamkäskud, mida teie assistent võib välja mõelda, ei ole väljastatud versioonides saadaval ja lõpetavad töö veateatega.
{% endhint %}

## Miks AI poolt kirjutatud skriptid töötavad hästi koos Chloros-iga

Kõik need on Chloros versiooni 1.2.0 tegelikud ja kinnitatud käitumismallid — need kõrvaldavad masinakirjutatud automatiseerimise klassikalised veamoodused:

* **Ei mingeid keerulisi seadistusi.**SDKi nutikad ühendusabimehhanismid (`connect_camera`, `connect_array`, `connect_daq_sensor`) ja töötlemise sisenemispunktid (`ChlorosLocal`, `process_folder`)**käivitavad kohaliku tagapõhja automaatselt**. Genereeritud skript ei vaja avatud graafilist kasutajaliidest ega käsitsi käivitatud serverit — vaja on ainult töölauale installitud paketti desktop/CLI.
* **Kogu töötlemisahel on üks käsk.** `chloros_sdk.process_folder("path", indices=["NDVI"])` käivitab algusest lõpuni järgmised toimingud: import → kalibreerimine → peegeldus → indeksi eksport. Vähem pinda, vähem kohti, kus genereeritud skript võib viga anda.
* **Väljundita käivitused teostavad enesediagnostikat.** Pärast `process()` lisatakse tulemusele käivituse kokkuvõte ning iga töötlemisvihje (nt *miks* käivitus tulemust ei andnud) edastatakse uuesti Python `UserWarning`-na — nii et isegi skript, mis tulemuse sõnastikku kunagi ei kontrolli, kuvab diagnoosi.
* **CLI ebaõnnestub märgatavalt.**`chloros-cli process`-käivitus, mis nõudis väljundeid, kuid ei kirjutanud ühtegi, väljastab `Processing finished but wrote no image products.` ja**lõpetab töötamise nullist erineva tulemusega**, nii et shell-skriptid ja CI tuvastavad selle lihtsa lõpetuskoodi kontrolliga. Edukad käivitused annavad tulemuseks `Image products written: N`.

Üks asümmeetria, mida assistent peaks teadma: SDK-i `process()` ei tekita tahtlikult **viga** nulltoote käivituse korral — selle asemel annab see teavet kokkuvõtte/vihjete kaudu. Kui Python-torujuht peab tühja käivituse korral peatuma, kontrollige kokkuvõtet (retsept 2 teeb seda).

## Hoiatused

* **Chloros+ sisselogimine on nõutav.**CLI ja SDK nõuavad**tasulist** Chloros+ taset, mida rakendatakse serveri poolel: päringud ebaõnnestuvad veaga `401 AUTH_REQUIRED`, kui pole sisse logitud, ja veaga `403 PLAN_UPGRADE_REQUIRED` tasuta tasemel. Käivita `chloros-cli login` üks kord iga masina kohta enne genereeritud skriptide käivitamist. Vaata [Chloros+ sisselogimist](chloros+-login.md).
* **Salvestamiskäsud juhivad tegelikku riistvara.** Käsud `lattice` / `daq` / `project` ja seansiobjektid SDK loovad ühenduse füüsiliste kaamerate ja anduritega, edastavad neilt voogu ja käivitavad neid. Vaadake loodud skripti enne selle esimest käivitamist läbi ja käivitage see riistvara juuresolekul.
* **Kontrollige väljundeid pisteliselt.** Enne tulemuste avaldamist kontrollige tootekaustaid ja mõningaid pikselväärtusi. Eelkõige skaalatakse peegeldusvõime TIFF-faile allika järgi – lugege `Chloros:PixelScale` XMP-märget (LATTICE: 32768 = peegeldusvõime 1,0; Survey3: 65535), selle asemel et eeldada jagajat. Mõlemal viitelehel on see dokumenteeritud jaotises „Peegelduspikslite lugemine”.
* **Väikesed lõksud, mis tekitavad probleeme genereeritud koodis:**`pool-record` kirjutab**tagapõhja hosti** failisüsteemi (vaikimisi `~/Documents/DAQ Live View/`); masinatel, millel on mitu võrguliidest, eelistage `daq pool-connect --eth-host <ip-or-hostname>`-i automaatse tuvastamise asemel; ning kasutage `http://127.0.0.1:5000` (mitte kunagi `localhost`) kõikjal, kus esineb backend URL.
