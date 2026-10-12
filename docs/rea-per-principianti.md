# REA spiegato semplice (per chi non ci capisce niente)

> **Cos'è questo file.** Una guida in italiano, senza gergo, a cosa serve REA, cosa ti serve per usarlo e come si comincia. È scritta per chi non è un programmatore esperto.
> I risultati citati con numeri **li ho eseguiti davvero** il 2026-10-12 in una macchina Linux (Debian 12, Node 22.22.3), installando REA da npm. Quando una cosa non l'ho provata, lo dico.
> Documento tecnico di riferimento: `docs/rea-repository-study.md` (stesso repo).

---

## 1. Cos'è, in una frase

**REA è un traduttore tra il tuo agente AI (o il tuo terminale) e gli strumenti che servono a "smontare" i programmi degli altri.**

Non è un programma che usi con il mouse cliccando pulsanti. È un **server di strumenti**: l'agente AI (Claude Code, Cursor, Copilot, Gemini CLI…) lo interroga, e REA risponde con dati, codice, prove.

### L'analogia del perito meccanico

Immagina di avere davanti un motore chiuso, senza manuale (un'app, un programma, un file binario).

- Senza REA: il tuo agente AI può solo *leggere file* e *guardare*. Davanti a un `.exe`, un `.dll` o un'app dentro un `.asar`, si ferma.
- Con REA: l'agente ha un **perito meccanico** che smonta il motore, elenca pezzi e viti, ti dice come sono collegati e — soprattutto — **ti consegna le ricevute**: "questo l'ho visto davvero", "questo l'ho dedotto", "questo non lo so".

La parte importante è l'ultima: REA **non inventa**. Ogni risposta arriva con la sua provenienza e i suoi limiti.

---

## 2. Per cosa ti serve (casi concreti)

| Se ti riconosci in questa frase… | REA ti serve per… |
| --- | --- |
| "Questa app fa una cosa che vorrei nella mia" | capire come è fatta quella funzione, e rifarla nella tua (i casi di studio pubblici fanno esattamente questo: DX-Ball, Notion, un gioco PC-98) |
| "Devo far parlare il mio software con un altro programma senza documentazione" | trovare dove sono i punti di contatto: comandi, indirizzi, canali IPC, formati di file |
| "Ho un file binario/file di dati e non so cosa contiene" | ispezionare struttura, sezioni, stringhe, firme, plist, archivi, firmware, APK |
| "Sto facendo CTF / sicurezza / studio di malware" | pseudocodice, assembly, chiamate, riferimenti, protezioni attive |
| "Devo capire cosa fa un sito web quando lo apri" | vedere script caricati, richieste di rete, listener, screenshot |
| "Ho un'app Electron e voglio capire cosa passa tra interfaccia e sistema" | mappa di moduli, preload, API esposte, canali IPC |

Esempi **veri** che ho eseguito mentre scrivevo questa guida (dettagli al §8):

- su una libreria JavaScript (Express) ha ricostruito la mappa del codice in **4,8 secondi**;
- su una piccola app Electron inventata da me ha riconosciuto **1 finestra, 1 preload, 4 API esposte, 8 operazioni IPC** (4 canali, tutti "accoppiati" tra interfaccia e main process);
- su `/bin/ls` (programma di sistema Linux) ha elencato **31 sezioni, 13 segmenti, 127 simboli, 329 rilocazioni** e le protezioni attive (PIE, NX, stack canary, RELRO parziale).

---

## 3. Per cosa **NON** serve (evitiamo equivoci)

1. **Non è un programma da usare "a mano" con il mouse.** È fatto per essere usato *dall'agente AI* o da riga di comando. Chi si aspetta una finestra con il grafico da esplorare resterà deluso (per quello esistono Hopper, Ghidra, IDA aperti direttamente).
2. **Non "buca" niente da solo.** Non è un attrezzo per entrare in sistemi altrui, e non aggira protezioni. La licenza MIT e il disclaimer lo dicono chiaro: si usa solo su software che possiedi o che sei autorizzato a esaminare.
3. **Non ti dispensa dal capire.** Ti dà materiale e prove; le conclusioni le trae l'agente (o tu).
4. **Non serve per il TUO codice sorgente.** Se hai già i sorgenti, usa l'editor e i test. REA serve quando i sorgenti **non ci sono**.
5. **Non sostituisce Hopper/Ghidra/IDA.** Per l'analisi profonda dei binari REA *guida* quegli strumenti, non li rimpiazza.
6. **Non è un servizio online.** Gira sul tuo computer; non carica i tuoi file da nessuna parte (l'agente, però, poi manda i risultati al suo modello: vale la politica del tuo provider AI).

---

## 4. Cosa ti serve per iniziare (fai la somma di questi pezzi)

| Per analizzare… | Ti serve | Costo |
| --- | --- | --- |
| App JavaScript / Electron, .NET, archivi, plist, HAR (traffico di rete salvato) | solo **Node.js** (versione 22.x ≥22.19, 24.x ≥24.11 o 26+) e npm | **gratis** |
| Siti web | Node.js + un browser Chrome/Chromium installato | gratis |
| Binari nativi (pseudocodice, assembly…) | Node.js + un **motore**: Ghidra (gratis, richiede Java), oppure Hopper (a pagamento, c'è la demo), oppure IDA Pro (molto costoso) | da 0 € a migliaia di € |
| ELF su Linux (sezioni/simboli/mitigazioni) | Node.js + un ambiente Python con `pwntools` (REA **non** lo installa: glielo devi indicare tu) | gratis |
| APK Android | Node.js + JADX headless + un JDK | gratis |
| Firmware | Node.js + Binwalk o Unblob (Linux) | gratis |
| Device Android | Node.js + `adb` | gratis |

**In una frase:** la parte JavaScript/web/archivi funziona *subito* e gratis; la parte "binari nativi profondi" richiede un motore esterno.

Nota: alcune cose sono specifiche di un sistema operativo. Su Linux, per esempio, i comandi per file Apple (`inspect-plist`, `demangle-swift`) rispondono "serve macOS" — l'ho verificato, vedi §8.

---

## 5. Come si usa — Modo 1: con il tuo agente AI (il modo consigliato)

### Passo 1 — installa (una volta sola)

Apri il terminale e scrivi:

```bash
npx rea-agents setup
```

Cosa succede: REA ti chiede **quali agenti** hai (Claude Code, Cursor, Codex, Gemini CLI, Copilot CLI, OpenCode, Windsurf…), ti **mostra il piano** delle modifiche ai file di configurazione, fa un **backup** di quelli esistenti e **aspetta la tua approvazione** prima di scrivere. Non installa Node, Java, Homebrew e simili. Hopper lo installa solo se glielo concedi esplicitamente.

Se vuoi solo *vedere* cosa farebbe, senza toccare niente: `npx rea-agents setup --dry-run`.

### Passo 2 — riavvia l'agente

Chiudi e riapri il tuo agente AI. Ora ha gli strumenti REA disponibili (170 nel pacchetto 6.4.0; vedi §8) e le istruzioni di lavoro ("skill") installate.

### Passo 3 — parla con l'agente come parleresti a una persona

Esempi di richieste che funzionano bene:

> "Ho questa app Electron nella cartella `/Users/…/MyApp.app`. Spiegami come fa a leggere i file di configurazione e mostrami le prove che trovi."

> "Analizza `/percorso/app.asar`: quali moduli ci sono, quali canali IPC esistono e quali di questi non hanno un controllo di sicurezza. Dimmi anche cosa non sei riuscito a determinare."

> "Questo eseguibile legge un file proprietario. Ricostruisci il formato del file e scrivimi una funzione Python che lo legge."

Tre abitudini che sfruttano il punto forte di REA:

- **chiedi le prove**: "mostrami l'evidenza e i limiti";
- **chiedi gli "unknown"**: "cosa non siamo riusciti a determinare?" (REA tiene un registro apposta);
- **chiedi il passo successivo**: "quale strumento dovremmo provare adesso e perché?".

### Cosa NON devi fare

- Non lanciare REA "a mano" per ogni domanda: ci pensa l'agente.
- Non aspettarti il primo risultato su un binario grande in 2 secondi: la prima analisi con Ghidra può richiedere **minuti** (import + analisi automatica). Se l'agente va in timeout, la guida dice di riprovare dopo `close_binary`/`open_binary` (nel dubbio: chiedi all'agente, sa leggere la guida).

---

## 6. Come si usa — Modo 2: dal terminale (copia-e-incolla)

Serve quando vuoi un risultato tuo, salvabile, senza agente.

```bash
# 1) Provare senza installare nulla (una riga)
npx -y rea-agents@latest analyze-javascript-application /percorso/della/tua/app --json

# 2) Installare il comando `rea` in modo permanente
npm install --global rea-agents
rea --help                 # elenco dei comandi (130)
rea <comando> --help       # spiegazione e opzioni del singolo comando

# 3) Aggiornare, quando esce una versione nuova
rea update
```

Comandi utili per iniziare (tutti con `--json` se vuoi il risultato leggibile da altri programmi):

```bash
# Cosa ho sotto mano? (struttura di un archivio/pacchetto: file, digest, formati)
rea inspect-artifact /percorso/file.bin --json

# App JavaScript/Electron: mappa dei moduli, preload, IPC
rea analyze-javascript-application /percorso/app --json > evidenza.json

# Da un risultato grande, estrai solo un riassunto leggibile
jq -c '{source:{kind:"inline",evidence:.},view:{kind:"summary"}}' evidenza.json > vista.json
rea inspect-analysis-view vista.json

# Binario Linux: sezioni, segmenti, simboli, mitigazioni (serve pwntools, vedi §8)
REA_PWNTOOLS_PYTHON=/percorso/venv/bin/python rea inspect-binary-layout /bin/ls --json

# .NET (nessun motore esterno richiesto)
rea inspect-managed-artifact /percorso/Assembly.dll --json

# Diagnostica: "perché non funziona?"
rea doctor                 # dice cosa c'è, cosa manca e cosa fare
```

Opzioni trasversali comode (le trovi con `--help` su ogni comando): `--format json|yaml|md|toon|jsonl`, `--filter-output chiave.sottochiave` per vedere solo una parte del risultato, `--token-count` per sapere quanto è grande l'output, `--schema` per la descrizione esatta degli input, `--llms` per un manifest pensato per gli agenti.

---

## 7. La cosa più importante da capire: i risultati sono "prove", non risposte

Quando REA risponde, ogni risultato è un **record di evidenza** con:

| Campo | In parole semplici |
| --- | --- |
| `subject` | **cosa** è stato analizzato: nome file, impronta digitale (sha256), formato |
| `provider` | **chi** l'ha analizzato: es. "REA pwntools ELF adapter 4.15.0" |
| `confidence` | **quanto è solido**: `observed` = visto direttamente · `derived` = dedotto dal contenuto · `inferred` = ipotesi |
| `authority` | **da dove viene**: file come spedito, esecuzione, documentazione… |
| `limitations` | **cosa non copre** quel risultato (elenco esplicito, non fumo) |
| `evidence_links` | riferimenti ad altre prove collegate |
| unknowns | domande ancora aperte, tenute in un registro apposta |

Esempio reale (dalla mia prova su `/bin/ls`): le protezioni risultano marcate come **`inferred`** — cioè "indicate dai flag del file, non provate eseguendo". È esattamente il tipo di onestà che serve per non costruirci sopra conclusioni sbagliate.

**Conseguenza pratica:** quando l'agente ti risponde "non lo so" o "non determinabile", non è un difetto. È REA che si rifiuta di inventare.

---

## 8. Cosa ho eseguito io (e cosa ha risposto)

Tutto su Linux Debian 12, Node 22.22.3, REA 6.4.0 installato con `npm install rea-agents`. Tempi reali.

**a) Libreria JavaScript (Express, quella di npm).** Comando:
`rea analyze-javascript-application ./node_modules/express --json`
→ **4,8 secondi**, file di evidenza da **7,0 MB**. Dentro: 8 file rilevanti, 7 file JavaScript analizzati, 6.074 nodi di sintassi esaminati, 172 osservazioni, **4.261 nodi** nel grafo semantico. `confidence: derived`, `authority: shipped-artifact`, e **5 limitazioni** dichiarate (tra cui: "il codice è stato letto come testo inerte, mai eseguito").

**b) Piccola app Electron scritta da me** (finestra + preload + 4 canali IPC). Comando:
`rea analyze-javascript-application ./mini-app --json`
→ **470 KB** di evidenza. Riassunto: `browser_windows: 1`, `preload_entrypoints: 1`, `context_bridge_apis: 1`, `exposed_api_members: 4`, IPC: **8 operazioni, 4 canali letterali, 4 handler nel main, 4 trasmissioni dal renderer, 4 accoppiate, 0 ambigue, 0 non accoppiate**. È esattamente la mappa che avrei scritto a mano.

**c) Binario di sistema (`/bin/ls`).** Prima ho provato **senza** pwntools:
`rea inspect-binary-layout /bin/ls --json` → errore `provider_unavailable` con spiegazione e rimedio: *"Imposta REA_PWNTOOLS_PYTHON su un Python fornito da te con pwntools 4.15.0… REA non installa mai Python"*. Ho creato un ambiente Python e installato pwntools, poi:
`REA_PWNTOOLS_PYTHON=./.venv/bin/python rea inspect-binary-layout /bin/ls --json`
→ **2,2 secondi**, **177 KB**, `confidence: observed`, provider `pwntools@4.15.0;pyelftools@0.33;unicorn@2.1.2`: ELF x86-64 `ET_DYN`, entry point `0x61d0` (con `execution_status: unknown`), **31 sezioni, 13 segmenti, 127 simboli, 329 rilocazioni**, librerie collegate `libselinux.so.1`, `libc.so.6`, e mitigazioni candidate: PIE sì, NX sì, stack canary sì, **RELRO parziale**, stack eseguibile no.

**d) Il server MCP vero e proprio.** Ho avviato `rea mcp` e fatto l'handshake come farebbe un agente: il server si presenta come `rea 6.4.0`, protocollo MCP `2025-06-18`, ed espone **170 tool**. (Nel repository su `main` i contratti sono **173**: i tre in più — `inspect_go_binary`, `inspect_wasm_artifact`, `project_harmony_application_graph` — sono stati aggiunti *dopo* la release 6.4.0, quindi non sono ancora nel pacchetto npm. Verificato confrontando l'elenco live con il codice.)

**e) Cose che su Linux non funzionano (giusto che sia così).** `rea inspect-plist` e `rea demangle-swift` hanno risposto `capability_unavailable: "Native macOS utilities require macOS."` REA non finge di poterle fare.

**f) Diagnostica.** `rea doctor` ha riportato `environment_healthy: false` con spiegazioni oneste: Node 22.22.3 ok, host debian 12, `GHIDRA_INSTALL_DIR is not set`, "IDA MCP registration is not configured", "Installed REA skill identity is missing". **Non è un errore**: significa che su quella macchina non c'è nessun motore di analisi configurato. È il primo comando da lanciare quando qualcosa non va.

---

## 9. Errori tipici, tradotti

| Quello che vedi | Cosa significa | Cosa fare |
| --- | --- | --- |
| `provider_unavailable` + `REA_PWNTOOLS_PYTHON` | manca pwntools (per ELF Linux) | crea un ambiente Python tu, come ho fatto io: `python3 -m venv .venv && .venv/bin/pip install pwntools==4.15.0 pyelftools==0.33 unicorn==2.1.2`, poi indica quel Python |
| `provider_unavailable` su un binario nativo | manca Hopper/Ghidra/IDA, oppure non è configurato | installa Ghidra (gratis) o Hopper, poi `rea doctor` |
| `capability_unavailable: requires macOS` | stai su Windows/Linux e hai chiesto qualcosa di Apple | usa una macchina macOS, oppure ignora: non è il caso d'uso |
| `VALIDATION_ERROR` con `path: ["symbols"]` | hai dato gli argomenti sbagliati (esempio reale: `demangle-swift` vuole `path` + simboli) | leggi `rea <comando> --help` (o `--schema`) |
| `environment_healthy: false` in `rea doctor` | manca un motore o la skill non è installata | segui le voci elencate: sono una lista di cose da fare, non un guasto |
| l'agente "non vede" REA | non è stato riavviato dopo `setup`, oppure la registrazione è vecchia | riapri l'agente; se persiste, `npx rea-agents@latest setup` e `rea doctor --client <nome>` |
| la prima analisi non finisce | import + analisi automatica del motore (Ghidra può richiedere minuti) | aspetta; per i binari grandi si aumenta il tempo massimo d'avvio con `REA_GHIDRA_STARTUP_TIMEOUT_MS` |

---

## 10. Rischi, soldi e legalità (da leggere, sono 6 righe)

- **REA è gratis** (licenza MIT). Gratis non è la parte nativa profonda: **Hopper** (a pagamento, demo gratuita), **IDA Pro** (molto costoso), **JEB** (a pagamento) restano tuoi, con le loro licenze. **Ghidra è gratuito.**
- **Devi avere il diritto di analizzare ciò che analizzi.** Software tuo, o per cui hai autorizzazione. Il disclaimer del progetto è esplicito: no a copie craccate, no a usi illeciti.
- **L'analisi gira in locale**, ma l'agente manda i risultati al suo modello AI: se il bersaglio è sensibile, tieni conto della politica del tuo provider.
- **Attenzione ai comandi "runtime"** (`capture-*`, `observe-*`, ADB, process capture): lì REA **esegue o interagisce** con il programma. Fallo solo su cose tue o in una macchina virtuale.
- **Per il lavoro/azienda:** verifica le policy interne prima di installare strumenti di reverse engineering sui computer aziendali.
- **REA cambia in fretta** (35 release in 3 mesi): aggiorna (`rea update`) e, se qualcosa si rompe, prova prima l'aggiornamento.

---

## 11. Glossario minimo

| Parola | Significato in una riga |
| --- | --- |
| **MCP** | Il "linguaggio" con cui un agente AI chiama strumenti esterni; REA è un server MCP |
| **CLI** | Uso da riga di comando (terminale) |
| **Agente AI** | Claude Code, Cursor, Copilot CLI, Gemini CLI… il programma con cui parli |
| **Skill** | Istruzioni di lavoro che `setup` installa nell'agente, così sa *come* usare REA |
| **Evidence / evidenza** | Il risultato strutturato di un'analisi, con provenienza e limiti |
| **Provider** | Il motore che fa il lavoro (Hopper, Ghidra, IDA, pwntools, il parser .NET…) |
| **Binario nativo** | Un programma compilato (.exe, .dll, file ELF/Mach-O): qui serve un motore |
| **Pseudocodice** | Codice "leggibile" ricostruito da un decompilatore (utile ma non è il sorgente originale) |
| **ASAR** | Contenitore di file usato dalle app Electron |
| **IPC** | I canali con cui le parti di un'app si scambiano messaggi (es. interfaccia ↔ processo principale) |
| **HAR** | File in cui il browser salva il traffico di rete di una sessione |
| **Unknown residuo** | Una domanda rimasta senza risposta, registrata e riutilizzabile più tardi |

---

## 12. Da dove partire domani (checklist)

1. **Prova a costo zero.** Su una macchina qualsiasi con Node installato: `npx -y rea-agents@latest analyze-javascript-application /percorso/di/una/app --json | head -c 2000`.
2. **Scegli un bersaglio utile a te** e scrivi la domanda in una frase ("come fa X a fare Y?").
3. **Collega l'agente:** `npx rea-agents setup`, riavvia l'agente, fai la domanda.
4. **Se ti serve l'analisi nativa:** installa Ghidra (gratis) e verifica con `rea doctor`.
5. **Tieni il risultato:** salva l'evidenza su file (`> evidenza.json`), così puoi rileggerla, confrontarla ed esportarla.

---

## 13. Cheat sheet (da tenere a portata)

```bash
npx rea-agents setup            # collega REA ai tuoi agenti AI (con piano + approvazione)
npx rea-agents setup --dry-run  # ...senza cambiare niente
rea --help                      # elenco comandi
rea <comando> --help            # come si usa quel comando
rea doctor                      # diagnosi: cosa manca e come rimediare
rea update                      # aggiorna
rea capabilities                # cosa è disponibile su questa macchina
rea analyze-javascript-application <percorso> --json   # app JS/Electron
rea inspect-artifact <percorso> --json                 # cosa c'è dentro un file/archivio
rea inspect-binary-layout <percorso> --json            # ELF (serve pwntools)
rea inspect-managed-artifact <file.dll> --json         # .NET, senza motori esterni
rea inspect-analysis-view <vista.json>                 # riassunto di un'evidenza enorme
```

**La regola d'oro:** chiedi sempre *"mostrami le prove e i limiti"*. Se te lo dice REA, fidati. Se te lo dice un modello senza prove, no.
