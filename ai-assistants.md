# Chlorosi kasutamine koos tehisintellekti abilistega

Käesolev juhend on mõeldud kahele sihtrühmale: inimestele ja tehisintellekti assistentidele, kelle kaudu inimesed üha enam töötavad. Igal lehel on esitatud täpsed väärtused, vaikimisi seaded ja kopeeritavad-kleebitavad käsud, et assistent (Claude, ChatGPT, Copilot, programmeerimisagent jne) saaks juba esimesel katsel kirjutada toimiva Chloros-automaatika.

Chlorosi versioon: **

1.3.0**. CLI- ja SDK-platvormid: Windows 10/11 x64 ja Linux (x86\_64 / Jetson aarch64).

## Mida assistendile edastada

| Ressurss | URL | Milleks see on |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Käesoleva juhendi iga lehekülje masinloetav indeks. |
| **Käsurea (CLI) viide** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Täielik `chloros-cli` käsurea liides: kõik käsud, lipud, vaikimisi seaded, väljumiskoodid ja väljundkausta reeglid. Koostatud LLM-i jaoks. |
| **SDK-viide** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Täielik `chloros_sdk` Python-API: klassid, signatuurid, erandid ja näited. Koostatud LLM-i jaoks. |
| **Iga lehekülg toores Markdown-vormingus** | lisage `.md` lehekülje URL-ile | nt `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` tagastab lehekülje toores Markdown-vormingus — ideaalne kontekstiaknasse kleepimiseks või agendilt hankimiseks. |

Juhendisisesed lingid: [CLI-viide](reference/cli-reference.md) · [SDK-viide](reference/sdk-reference.md).

{% hint style="info" %}
Need kaks viitelehte on iseseisvad: assistent, kes on ühe neist läbi lugenud, ei vaja käsiraamatu ülejäänud osa, et kirjutada korrektne skript.
{% endhint %}

## Laske oma assistendil käivitadChloros'i otse: `chloros-mcp`

Kõik eespool nimetatu eeldab, et teie assistent *kirjutab* skripti või käsu. `chloros-mcp` abil juhib [Model Context Protocol](https://modelcontextprotocol.io) (MCP) keelt kõnelev assistent, näiteks Claude Desktop, Claude Code või Cursor, Chloros'i ise. Te esitate küsimuse lihtsas keeles ja assistent kutsub välja serveri tööriistad:

* **Piltide töötlemine:** looge projekt, importige pildikaadrite kaust, valige seaded, töötlege ja esitage aruanne selle kohta, mida käivitamine eksportis.
* **Riistvara käitamine:** ühendab LATTICE-kaamerad, maatriksid ja DAQ-valgusandurid, näitab eelvaadet, seadistab säri ja värvi, teeb pildistusi, salvestab ja uuendab püsivara.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Serveri tööriistad hõlmavad seda, mida teeb [Pythoni SDK](api-python-sdk.md). Need paigaldatakse koos SDK-ga, seega pole tavaliselt vaja midagi muud paigaldada: Windowsis paigutab Chlorosi paigaldaja `chloros-mcp.exe` teie Pythoni `Scripts` kausta, Linuxi puhul paigutab `.deb` faili `chloros-mcp` kausta `/usr/bin`.

**Claude'i kood:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**lisage see faili `claude_desktop_config.json` (**Seaded > Arendaja > Konfiguratsiooni redigeerimine**avab selle), seejärel taaskäivitage Claude Desktop.**Kursor** võtab sama kande failist `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Kui klient teatab, et serverit ei õnnestunud käivitada, ei ole `chloros-mcp` tõenäoliselt selles `PATH`-is, mida ta näeb: sisesta selle asemel täielik tee (`where chloros-mcp` Windowsis, `which chloros-mcp` Linuxis).

Samale arvutile peab olema installitud Chloros ja seal peab olema sisselogitud – kas rakenduses või `chloros-cli login` kaudu – mõnel tasulisel Chloros+ tasemel (Copper või kõrgem). Server ei saa teie eest sisse logida.

{% hint style="info" %}
Iga rakenduse, valikute, tööriistade ja käivitamise käitumise kliendi seadistamine: [MCP-server (AI-assistendid)](reference/sdk-reference.md#mcp-server-ai-assistants) SDK-viites.
{% endhint %}

## Käskude näidised

Kopeeri, täida `<placeholders>` ja kleebi oma assistenti.

### 1. Lennukausta töötlemine NDVI-ks

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Jälgi pildistatud kaustat partiina

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Ühenda LATTICE-maatriks ja tee pildistamine

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. DAQ-valgusanduri spektrite salvestamine

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
DAQ-skriptide käivitamine käsurealt toimub alati `daq pool-*`-perekonna kaudu (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Muud `daq` alamkäskud, mida teie assistent võib välja mõelda, ei ole tarnitud versioonides saadaval ja põhjustavad vea.
{% endhint %}

## Miks AI poolt kirjutatud skriptid töötavad hästi koos Chlorosega

Kõik need on Chloros 1.3.0 tegelikud ja kinnitatud käitumismallid – need kõrvaldavad masinapõhise automatiseerimise klassikalised ebaõnnestumisviisid:

* **Ei mingeid keerulisi seadistusi.**SDK nutikad ühendusabivahendid (`connect_camera`, `connect_array`, `connect_daq_sensor`) ja töötlemise sisenemispunktid (`ChlorosLocal`, `process_folder`)**käivitavad kohaliku tagapõhja automaatselt**. Genereeritud skript ei vaja avatud graafilist kasutajaliidest ega käsitsi käivitatud serverit – vaja on vaid töölaua- või CLI-paketi installimist.
* **Kogu töötlemisahel on üks käsk.** `chloros_sdk.process_folder("path", indices=["NDVI"])` käivitab algusest lõpuni impordi → kalibreerimise → peegeldusvõime mõõtmise → indeksi ekspordi. Vähem pinda, vähem kohti, kus genereeritud skript võib viga tekitada.
* **Väljundita käivitused teostavad enesediagnoosi.** Pärast `process()` lisatakse tulemusele käivituse kokkuvõte ning iga töötlemisega seotud vihje (nt *miks* käivitus väljundit ei andnud) edastatakse uuesti Pythoni `UserWarning`-ina — nii et isegi skript, mis tulemuste sõnastikku kunagi ei kontrolli, kuvab diagnoosi.
* **CLI-liides annab selgelt teada ebaõnnestumisest.**`chloros-cli process`-käivitus, mis nõudis väljundeid, kuid ei kirjutanud ühtegi, väljastab `Processing finished but wrote no image products.` ja**lõpetab töötamise nullist erineva väljundkoodiga**, nii et shell-skriptid ja CI tuvastavad selle lihtsa väljundkoodi kontrolliga. Edukad käivitused näitavad koodi `Image products written: N`, millele lisandub `(plus M clamp masks)`, kui maskid kirjutati. Ka salvestuskäsud nimetavad iga faili, mida nad kirjutada ei suutnud, ja lõpetavad töö koodiga `1`.
* **Suhtelised teed tähendavad seda, mida skript ootab.** Kohaliku tagapõhja puhul määratakse suhteline väljundtee (`-o out/` CLI-s, `output_dir="out/"` SDK-s) vastavalt kaustale, kust käsk või skript käivitatakse.

Üks asümmeetria, mida assistent peaks teadma: SDK-i `process()` ei tekita tahtlikult **viga** nulltulemusega käivituse korral — selle asemel annab see teavet kokkuvõtte/näpunäidete kaudu. Kui Python-torujuht peab tühja käivituse korral peatuma, kontrollige kokkuvõtet (retsept 2 teeb seda).

## Hoiatused

* **Nõutav on Chloros+ sisselogimine.**CLI ja SDK nõuavad**tasulist** Chloros+ taset, mida rakendatakse serveri poolel: päringud ebaõnnestuvad veaga `401 AUTH_REQUIRED`, kui sisse pole logitud, ja veaga `403 PLAN_UPGRADE_REQUIRED` tasuta tasemel. Käivita `chloros-cli login` üks kord iga masina kohta enne genereeritud skriptide käivitamist. Vaata [Chloros+ sisselogimine](chloros+-login.md).
* **Salvestuskäsud juhivad reaalset riistvara.** `lattice` / `daq` / `project`-käskudega, SDK-seansiobjektidega ja `chloros-mcp`-i tööriistadega ühendatakse, edastatakse andmeid ja käivitatakse füüsilisi kaameraid ja andureid. Vaadake genereeritud skripti enne selle esimest käivitamist läbi ja käivitage see riistvara juuresolekul.
* **Kontrollige väljundeid pisteliselt.** Enne tulemuste avaldamist kontrollige tootekaustu ja mõningaid pikselväärtusi. Eelkõige on peegeldusvõime TIFF-failid skaalatud allika järgi – lugege `Chloros:PixelScale` XMP-märget (LATTICE-failid sisaldavad seda: 32768 = peegeldusvõime 1,0; Survey3-failid seda ei sisalda ja kasutavad 65535 = 1,0) selle asemel, et eeldada jagajat. Mõlemal viitelehel on see dokumenteeritud jaotises „Peegeldusvõime pikslite lugemine”.
* **Väikesed lõksud, mis segavad genereeritud koodi tööd:**`pool-record` kirjutab**tagapõhja hosti** failisüsteemi (vaikimisi `~/Documents/DAQ Live View/`); masinatel, millel on mitu võrguliidest, eelistage `daq pool-connect --eth-host <ip-or-hostname>` automaatsele tuvastamisele; kasutage `http://127.0.0.1:5000` (mitte kunagi `localhost`) kõikjal, kus esineb tagapõhja URL; skript, mis läbib projekti väljundpuud, peaks `clamp_masks/`-kaustad vahele jätma (nende `*_clampmask.tif`-failid on maskid, mitte pildid — vt [Väljundpildi formaadid](output-image-formats.md#clamp-masks)); ning Windowsis, kus pikad teed on välja lülitatud, hoidke projektikaustad lühikesed – projektikaust, mille nimi on pikem kui umbes 130 tähemärki, lükatakse tagasi enne töötlemise algust.
