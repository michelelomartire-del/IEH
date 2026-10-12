# Studio del repository `morluto/rea` (REA — Reverse Engineer Anything)

> **Metadati dello studio**
> Repository: `https://github.com/morluto/rea` (pubblico, licenza MIT, sito `https://rea.tools`)
> Commit studiato: `5190843f8f09ac17cbf79561bd1f708c509cda78` (`main`, 2026-10-12)
> Clone locale: `/home/user/rea` (91 MB di working tree, 51 MB di git)
> Data dello studio: 2026-10-12
> Numeri GitHub al momento dello studio: ⭐ 132.262 · 🍴 29.560 · 414 watcher · 129 issue aperte · 1.424 PR totali · 710 issue totali · ≥100 contributor
> Ultima release: `rea-agents` v6.4.0 (2026-10-11) — pacchetto npm `rea-agents`, 31 versioni pubblicate dal 2026-07-12

---

## 1. TL;DR — che cosa è questo repository

REA è un **server MCP + CLI** che espone a un agente (Claude Code, Codex, Cursor, Gemini CLI, Copilot CLI, OpenCode, Pi, Hermes, dsh, …) un insieme molto ampio di strumenti per il **reverse engineering locale**: binari nativi, app JavaScript/Electron, assembly .NET, siti web, capture di rete, firmware, APK Android, device ADB, progetti JEB, WASM e bytecode EVM.

Punti che definiscono il progetto:

1. **MCP-first, ma con parità CLI.** Lo stesso workflow applicativo serve `tools/call` MCP e i comandi `rea …` (`src/application/**` è condiviso; `src/contracts/**` definisce l'inventario canonico).
2. **Evidence come cittadino di prima classe.** Ogni risultato è un record di *Evidence* immutabile con provenienza (provider, digest del soggetto, profilo di analisi), `confidence` (`observed` / `derived` / `inferred`), `authority`, `limitations`, link ad altra Evidence e un registro esplicito di **unknown residui**. Non esistono "risposte secche": esistono osservazioni con provenienza e limiti.
3. **Local-only, BYO engine.** Nessun upload del target. Gli engine di analisi profonda (Hopper, Ghidra, IDA, JEB) sono portati dall'utente; REA *non* installa né aggiorna runtime non correlati (niente Homebrew/Node/Java/Ghidra); Hopper può essere installato solo con approvazione esplicita e piano mostrato.
4. **Ownership rigorosa delle risorse.** Ogni processo avviato è tracciato (PID + parent + process group + discendenti, "run token" su macOS, Job Objects su Windows) e REA non uccide mai un processo di cui non può provare il possesso. I cleanup falliti **mantengono** l'ownership per un retry.
5. **Scala e disciplina di processo inusuali.** ~470k righe di TypeScript, 915 file di test, 173 tool MCP pubblici con audit degli effetti, 130 comandi CLI, 19 workflow GitHub Actions, 35 release in ~3 mesi.

---

## 2. Storia, ritmo e comunità

| Data | Evento |
| --- | --- |
| 2026-04-14 | Primo commit: *"betterBinaryMCP - Enhanced MCP server for Hopper Disassembler"* (progetto Python, focus Hopper) |
| 2026-07-12 | Rimozione dell'implementazione Python legacy, tooling Node.js, prime release (`rea-0.2.0` → `rea-agents-0.5.0`), primo publish su npm |
| 2026-07 → 2026-10 | Riscrittura/espansione completa in TypeScript; Ghidra, IDA, browser/CDP, JavaScript, .NET, Android, firmware, EVM, WASM, ADB, process capture |
| 2026-10-05 → 10-11 | Serie di release molto rapida (v4.0.1 … v6.4.0 in sei giorni) |
| 2026-10-12 | HEAD studiato: correzioni su setup/uninstall, provider managed, plist binari, WASM, MCP |

- **2.491 commit** su `main`, **35 tag di release**, changelog generato (2.467 righe in `CHANGELOG.md`).
- Top contributor: `morluto` (~1.439 commit) — il progetto è a fortissima concentrazione autoriale, con `Kaoru0822`, `N0zoM1z0`, `Ysopwnable` e altre decine di contributor esterni citati nei ringraziamenti di release.
- La numerazione issue/PR condivisa arriva a ~#2140 → ~1.424 PR + ~710 issue in 6 mesi.
- C'è un canale Discord, un sito VitePress e casi di studio pubblici (DX-Ball, Notion/Elettron, TH04) con repository di ricostruzione terzi.

**Lettura critica:** la crescita (132k stelle in 6 mesi) è quella di un progetto "viral" guidato da un autore principale molto attivo e da un flusso continuo di contribuzioni esterne. Il rischio principale non è la qualità del codice — che è sorprendentemente disciplinata — ma la **sostenibilità**: 173 tool, decine di provider e un changelog torrenziale implicano un carico di manutenzione e verifica molto alto, oggi sostenuto da poche persone chiave.

---

## 3. Mappa del repository

```
/home/user/rea
├── src/                 # 1.568 file .ts, ~324k righe (incl. ~87k di test colocati)
│   ├── cli.ts, main.ts  # entry point: CLI one-shot e server MCP (stdio)
│   ├── cli/             # adapter CLI (130 comandi, opzioni, output JSON/stream)
│   ├── server/          # adapter MCP (createServer, registrazione tool, delivery)
│   ├── composition/     # factory: costruiscono provider/servizi senza avviare engine
│   ├── application/     # workflow condivisi CLI+MCP (Setup, Doctor, sessioni, analisi)
│   ├── contracts/       # schemi zod canonici dei tool e degli input/output
│   ├── domain/          # logica pura (Evidence, Result, target, diff, grafi)
│   ├── process/         # supervisione processi, ownership, deadline, capture
│   ├── <provider>/      # hopper, ghidra, ida, jeb, browser, dotnet, adb, android,
│   │                    # apktool, firmware, evm, wasm, go, javascript, artifacts, native…
│   ├── config/          # parsing env con zod
│   └── evaluation/      # valutazione delle sessioni agente a fine release
├── bridge/              # codice che gira DENTRO gli engine: Python (Hopper, mitmproxy,
│                        # pwntools/pwndbg, LLDB), Java (Ghidra, JADX, NativeAOT), Swift
├── native/windows/      # addon Node-API in C++ (NTFS, DACL, Job Objects)
├── tests/               # 7.0 MB: boundary (461 file), acceptance, conformance, composition…
├── docs/                # 49 guide Markdown + .vitepress + releases/
├── website/             # sito VitePress + assets, deploy Cloudflare Workers
├── scripts/             # 100+ script di build, generazione, verifica, CI
├── third_party/         # 5 submodule (binwalk, unblob, wakaru, jadx-headless-mcp, ghidra-nativeaot)
├── licenses/            # 17 testi di licenza di terze parti incorporati
└── .github/workflows/   # 19 workflow (1 CI principale + 18 lane "real-*" e docs/release)
```

Distribuzione del codice (righe): `src` ~324k (di cui test colocati ~87k), `tests` ~148k, `docs` 2,6 MB, `website` 11 MB.

---

## 4. Architettura a livelli

Il diagramma ufficiale è `docs/architecture.mermaid` (152 righe). La regola dichiarata in `AGENTS.md`:

> *"Dependencies flow inward: domain semantics and contracts, provider adapters, shared application workflows, then CLI/MCP adapters. Keep engine protocols and provider-specific code out of domain/application layers."*

Livelli effettivi:

| Livello | Directory | Responsabilità | Vincoli |
| --- | --- | --- | --- |
| Entry point | `scripts/rea.mjs` → `src/cli.ts`, `src/main.ts` | dispatcher `rea` / `rea mcp`; CLI one-shot vs server stdio | nessuna logica di dominio |
| Adapter | `src/cli/`, `src/server/` | traduzione richieste, registrazione tool, delivery, progress, cancellazione | i tool devono essere registrati *per nome di contratto*, mai per ordine di catalogo |
| Composition | `src/composition/` | factory tipizzate; binding di provider e sessione | non avviano engine né acquisiscono target; i fallimenti opzionali non devono abbattere i peer |
| Application | `src/application/` | workflow condivisi (Setup, Doctor, sessioni, analisi, investigazioni, comparison) | usa *port* stretti; nessun accesso diretto a protocolli engine |
| Provider | `src/hopper`, `src/ghidra`, `src/ida`, `src/jeb`, `src/browser`, `src/dotnet`, `src/android`, `src/adb`, `src/apktool`, `src/firmware`, `src/evm`, `src/wasm`, `src/go`, `src/native`, `src/artifacts`, `src/javascript`, `src/inspector` | protocolli e parsing specifici | confinati qui |
| Process foundation | `src/process/`, `src/windows/`, `native/windows/` | process group posseduti, root privati, deadline, letture bounded, cleanup | riusabile, nessuna semantica di dominio |
| Contracts | `src/contracts/` | schemi zod canonici, effetti, annotazioni, esempi, digest di catalogo | binding per nome |
| Domain | `src/domain/` (486 file) | puro, side-effect free: Evidence, `Result`, target, diff, grafi applicativi | nessun I/O |
| Config | `src/config/` | parsing env validato con zod, default, origini consentite | fallimento esplicito su valori invalidi |

Guardie eseguibili: `npm run verify:module-boundaries` (incluso nel lint) e `npm run verify:architecture-guards` (incluso in `check`/`check:ci`) verificano che i confini di import e le regole architetturali restino rispettati.

### Flusso tipico MCP

```
client MCP
  └─ tools/call "open_binary"        → src/server/registerSessionTools → src/application/** → composition → provider (Hopper/Ghidra/IDA)
  └─ tools/call "binary_overview"    → prima query: avvia l'engine, attende import + auto-analisi + bridge + health
  └─ tools/call "procedure_pseudo_code" / "xrefs" / "trace_call_path" …
  └─ tools/call "close_binary"       → drena le richieste, (opz.) snapshot, chiude provider e record
```

Le **sessioni** (`open_binary`, `binary_session`, `close_binary`) sono l'unica modalità "stateful": una sola binding immutabile per target/profilo; `SessionProviderRouter` + `AnalysisProviderRegistry` scelgono il provider in modo **deterministico e senza fallback** (auto-selezione con errore tipizzato `ambiguous` / `provider_unavailable` se la scelta non è univoca o l'engine è rotto). Sui target senza sessione, gli adapter ausiliari (browser, JavaScript, .NET, ADB, firmware…) sono instradati in modo disgiunto.

---

## 5. Superficie pubblica

### 5.1 MCP — 173 tool

Inventario canonico: `src/contracts/toolContracts.ts` aggrega 25 array di contratti (`officialToolContracts`, `enhancedToolContracts`, `nativeToolContracts`, `sessionToolContracts`, `adb/adbToolContracts`, …). Il test `src/contracts/toolEffects.test.ts` **impone biiezione** tra i contratti e `TOOL_EFFECTS` (173 chiavi verificate al commit studiato).

Famiglie principali:

| Famiglia | Esempi |
| --- | --- |
| Sessione/Evidence | `open_binary`, `close_binary`, `binary_session`, `export_evidence_bundle`, `get_evidence_bundle`, `get_navigation_context`, `import_evidence_bundle`, `list_unknowns`, `record_unknown`, `update_unknown`, `verify_unknown_resolution` |
| Nativi (Hopper/Ghidra/IDA) | `binary_overview`, `analyze_function`, `procedure_pseudo_code`, `procedure_callers/callees`, `xrefs`, `search_strings`, `list_segments`, `set_address_name`, `annotate_native_function`, `inspect_native_instruction/data_type/load_image`, `resolve_native_call_targets`, `read_bytes`, `address_to_file_offset` |
| Runtime/processo | `capture_process_scenario`, `compare_process_captures`, `observe_native_ui`, `trace_native_values`, `capture_browser_scenario`, `observe_web_session`, `observe_javascript_runtime`, `reconcile_javascript_runtime` |
| Applicazioni | `analyze_javascript_application`, `trace_feature`, `trace_application_feature`, `trace_javascript_semantics`, `compare_application_versions`, `compare_source_to_bundle`, `project_*_application_graph`, `build_reconstruction_obligation_ledger`, `evaluate_reconstruction_coverage`, `verify_reconstruction` |
| Web/Electron | `inspect_web_page`, `analyze_web_bundle`, `export_web_scripts`, `trace_web_source_location`, `trace_web_module_imports`, `inspect_web_event_listeners`, `compare_web_captures`, `capture_web_screenshot`, `compare_web_screenshots`, `discover_webmcp_tools` |
| Altri formati | `inspect_managed_*` (.NET), `inspect_android_*`/`decode_android_resources`/`search_android_classes` (APK), 25 tool ADB, `inspect_jeb_*`, `inspect_firmware_regions`, `extract_firmware`, `inspect_wasm_artifact`, `inspect_evm_interface`, `inspect_go_binary`, `inspect_binary_layout`, `inspect_recorded_crash`, `inspect_artifact` |
| Artefatti/comparison | `inspect_macho`, `inspect_signature`, `inspect_plist`, `list_architectures`, `demangle_swift`, `inspect_keyed_archive`, `decode_interface_builder`, `compare_artifacts`, `compare_functions`, `compare_bundles`, `find_changed_behavior`, `build_call_path`, `correlate_static_and_runtime` |

**Design dei tool** (documentato in `docs/tool-design.md`): forme ricorrenti `inspect` / `search`·`list` / `trace` / `compare` / `workflow` / `observe`·`capture`; si estende un tool esistente quando intent e contratto coincidono, se ne aggiunge uno per un esito o un'autorità diversi.

Ogni contratto dichiara:
- `inputSchema` / `outputSchema` zod (con esempi letterali, senza riferimenti ricorsivi raggiungibili);
- **effetti veri** (`mutatesTarget`, `mutatesSession`, `writesFilesystem`, `launchesProcess`, `accessesNetwork`, `changesUiState`, `mayDiscardData`, `executesForeignCode`, `mutatesDevice`, `idempotent`) in `src/contracts/toolEffects.ts`;
- **annotazioni MCP derivate** (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`) calcolate *solo* dagli effetti — e un test che verifica che la prosa delle descrizioni non contraddica gli effetti dichiarati.

`tools/list` è sempre **completo**: i tool non disponibili restano nel catalogo e la disponibilità è riportata da `binary_session.tool_availability` (in gruppi con requirement per client). Nessun `notifications/tools/list_changed`: il catalogo non cambia durante la sessione.

Casi limite curati in modo notevole:
- **Profilo schema compatto** (`REA_MCP_INPUT_SCHEMA_PROFILE=compact`) per client con cap sulla dimensione del JSON Schema per tool (Moonshot/kimi-cli, #1484): budget di 13.312 byte, con sei schemi presentati in forma ridotta ma **validazione canonica lato server invariata**.
- **Digest di catalogo** (`src/catalogIdentity.ts`): `tools_sha256`, `prompts_sha256`, `combined_sha256` calcolati su JSON canonico, utilizzabili da `binary_session` per confrontare la sessione viva con le attese del chiamante.
- **Errori** sempre come diagnostica canonica `{ error: … }` in `content` con `isError: true`, senza `structuredContent`; errori oversize trattenuti come Evidence con riferimento di export.

### 5.2 CLI — 130 comandi

`src/cliCommandNames.ts` è l'elenco canonico (usato anche nel digest di catalogo). Coprono le stesse famiglie dei tool (kebab-case), più comandi di vita del progetto: `setup`, `doctor`, `update`, `uninstall`, `capabilities`, `providers`, `evidence-import`/`evidence-export`, `compare`, `capture-process`.

Output: testo leggibile o `--json` (con variante streamed); exit code documentati in `docs/cli.md`; `setup --dry-run` pianifica senza scrivere; cancellazione via SIGINT/SIGTERM tradotta nello stesso `AbortSignal` dei provider.

### 5.3 Skill e prompt per agenti

- `.agents/skills/reverse-engineer-anything/SKILL.md` + reference (`native-and-artifacts.md`, `javascript-applications.md`, `runtime-observation.md`, `evidence-workflows.md`, `connection-and-recovery.md`) sono il **sorgente autoriale**; `skills/` e `docs/public/product-catalog.json` sono **output generati e gitignorati** (rigenerati con `npm run mcp-catalog:generate`, `npm run docs:generate`).
- `src/contracts/promptContracts.ts` definisce i prompt MCP guidati (es. investigazioni) e finisce anch'esso nel digest del catalogo.

---

## 6. Il modello di Evidence (il cuore del progetto)

`src/domain/evidence.ts` definisce il record immutabile; `src/application/investigation/EvidenceLedger.ts` (480 righe) e `UnknownEvidence.ts` lo governano.

Campi principali di un record:

| Campo | Note |
| --- | --- |
| `evidence_id` | prefisso `ev_` + digest (identità del contenuto) |
| `subject` | nome, `sha256`, formato (`mach-o`, `elf`, `pe`, `apk`, `hap`, `zip`, `dos-mz`…), architettura, path locale |
| `provider` | identità concreta dell'engine (Hopper/Ghidra/IDA/…), versione |
| `analysis_profile` | profilo di analisi, verificato coerente con il provider |
| `predicate_type` / `operation` / `parameters` | cosa è stato chiesto |
| `raw_result` / `normalized_result` | output grezzo del produttore e sua normalizzazione |
| `confidence` | `observed` \| `derived` \| `inferred` |
| `authority` | vocabolario chiuso (analisi statica, inferenza dell'analista, ecc.) |
| `environment` | piattaforma, architettura, isolamento (`none`/`process`/`container`/`vm`) |
| `limitations[]` | limiti dichiarati, obbligatori e non decorativi |
| `relationship_coverage` | copertura delle relazioni osservate: `complete` significa "restituito senza limiti noti", **mai** "tutte le possibili relazioni runtime" |
| `locations[]`, `evidence_links[]` | riferimenti incrociati e ancore al target |

Proprietà che il codice difende attivamente:
- **Idempotenza**: `ledger.record()` accetta duplicati che *concordano* e rifiuta (`EvidenceIntegrityError`) record con `evidence_id` uguale ma contenuto diverso.
- **Atomicità**: `ledger.import()` valida un intero bundle prima di fondere i record; delta di import riportato (`recordsAdded`, `unknownsAdded`, `changed`, `metadataChanged`).
- **Unknown residui di prima classe**: `record_unknown` / `update_unknown` / `verify_unknown_resolution`, con revisioni e "head" nel ledger; esistono test dedicati (`EvidenceLedger.residualUnknown.test.ts`).
- **Unknown ≠ assente**: la documentazione di design lo ripete più volte ("Missing coverage is unknown, not absence"); le prove di *possesso* sono tenute separate da validità/modificabilità/readiness.
- **Frozen/immutabile**: snapshot JSON congelati (`freezeJsonSnapshot`, `isImmutableJsonSnapshot`) per evitare che consumer mutino Evidence condivisa.

`InspectEvidenceBundle` + `docs/mcp-contracts.md` definiscono anche l'**import/export** dei bundle (formati di persistenza, import atomico, `published`/`published_path`/`bytes` come ricevute di pubblicazione quando la pulizia dello staging fallisce).

**Perché è il cuore:** questo modello è ciò che distingue REA da un insieme di wrapper attorno a Hopper/Ghidra. L'agente non riceve "il risultato", riceve un'osservazione con provenienza, incertezza e limiti — e può quindi collegarla, confrontarla e conservarla. È anche ciò che rende possibile la parte più ambiziosa del prodotto: i workflow di *ricostruzione verificata* (`build_reconstruction_obligation_ledger`, `evaluate_reconstruction_coverage`, `verify_reconstruction`) dove l'agente deve chiudere obblighi di ricostruzione con prove.

---

## 7. Provider e target supportati

| Target | Cosa restituisce | Requisiti | Guida |
| --- | --- | --- | --- |
| Binari nativi | pseudocodice, assembly, stringhe, simboli, call/xref | Hopper, Ghidra o IDA | `docs/installation.md#hopper`, `docs/ida-provider.md` |
| ELF offline | sezioni, segmenti, simboli/relocazioni originali, candidate mitigazioni | pwntools caller-supplied, Linux x64 | `docs/binary-diagnostics.md` |
| Go | versione compilatore, moduli, replacements, build settings | built-in (ELF/PE/thin Mach-O) | `docs/go-binaries.md` |
| WASM | validazione, sezioni, import/export WAT esatti | WABT 1.0.42 caller-supplied | `docs/wasm-artifacts.md` |
| EVM bytecode | dispatch selector, offset, argomenti inferiti | raw/hex carrier Linux x64 | `docs/evm-bytecode.md` |
| Crash registrati | registri/segnali di ogni thread, candidate mapping | pwntools (+ GDB/pwndbg opz.) | `docs/recorded-crashes.md` |
| JavaScript/Electron | moduli, import, source map, route, IPC, addon nativi | Node.js + npm | `docs/javascript-application-graph.md` |
| Siti web | struttura pagina, script, rete, screenshot | browser Chrome-family | `docs/browser-observation.md` |
| Capture di rete | richieste/risposte, payload esposti, source locations | HAR; mitmdump su Linux | `docs/web-network-captures.md` |
| .NET | metadata, CIL, dipendenze native dichiarate | nessuno (statico) | `docs/managed-code-analysis.md` |
| APK Android | manifest, classi, metodi decompilati | JADX headless + JDK completo | `docs/android-analysis.md` |
| Device Android | device, package, process, log, screenshot, trasferimenti file | adb caller-supplied | `docs/adb-device-analysis.md` |
| Progetti JEB | unit, digest, pseudocodice | client JEB avviato dall'utente (`REA_JEB_MCP_URL`) | `docs/jeb-analysis.md` |
| Risorse Android | manifest decodificato, stringhe, locale | Apktool caller-supplied | `docs/apktool-resource-analysis.md` |
| Firmware | regioni, estrazioni, handoff all'analisi nativa | Binwalk/Unblob su Linux | `docs/firmware-analysis.md` |
| Package/risorse | inventari, digest, plist, bundle Apple | nessuno | `docs/apple-application-analysis.md` |
| Processi | output terminale, interazioni, exit, effetti su filesystem, confronti | PTY nativo su Linux/macOS | `docs/process-capture.md` |

Complessità specifiche per engine (tutte documentate): Ghidra con database **effimeri** (nessuna mutazione dei byte, nessun controllo della GUI), supporto DOS/MIPS/NativeAOT/PE, Windows Ghidra "P0" read-only; IDA in sola lettura (mai salvare/chiudere un DB GUI collegato); Hopper avviato da REA quando serve (con dialogo di prima esecuzione demo/licenza da completare).

Pattern tecnico rilevante: i **bridge** (`bridge/**`) vivono *dentro* l'engine — `hopper_bridge.py` (socket Unix), `ReaGhidraBridge.java` + estensioni Ghidra (HeadlessScript), `ReaJadxBridge.java`, `capture.py` per mitmproxy, Swift per UI nativa e per il "run token reader", Python per pwntools/pwndbg/LLDB.

---

## 8. Processi, ownership e cleanup

`src/process/` (59 file) è un sotto-sistema a sé, con test molto frammentati (`ProcessOwnership.test.ts`, `.identity.`, `.observation.`, `.runId.`, `.validation.`, `part2/3`):

- **Ownership provata**: `ProcessOwnership` + `ProcessRunTokenObservations` + `ProcessStartIdentity`; su macOS un helper Swift (`bridge/process/ProcessRunTokenReader.swift`) legge il token di avvio; su Windows `WindowsAuthority` + `native/windows` (addon Node-API in C++ con Job Objects, DACL protette, ammissione NTFS).
- **Deadline e diagnosi**: `ProviderDeadline` (Hopper 120 s, Ghidra 330 s di default, override `REA_HOPPER_STARTUP_TIMEOUT_MS` / `REA_GHIDRA_STARTUP_TIMEOUT_MS` con validazione), `ProviderDiagnosticRedaction` (redazione credenziali di trasporto), `boundedUtf8Text`, `BoundedFileBytes`.
- **Cancellazione ≠ timeout**; il fallimento di cleanup produce `cleanup_incomplete` con l'elenco **solo** delle risorse possedute rimaste; se una pubblicazione atomica è già avvenuta, la ricevuta (`published: true`, `published_path`, `bytes`) resta nel risultato per distinguerla da una scrittura mai commessa.
- **Osservazioni parziali**: `details.partial_observation` conserva ciò che è stato osservato anche quando l'esecuzione fallisce (call tracing nativo, process capture con `elapsed_ms`/`exit_code` null se nessun exit osservato).
- **Process capture**: `src/process/capture/` (filesystem snapshot, journal, progress, terminal renderer, sampling, timer) con marcatori espliciti — "historical snapshots, not live process inventories" e "does not claim that no short-lived descendant existed".

Questo è, insieme a Evidence, la parte più "da manuale" del repository: le regole sono enunciate in `AGENTS.md` e poi *verificate da codice*, non solo dichiarate.

---

## 9. Contratti, schemi e compatibilità con i client

Problemi reali risolti in modo interessante in `src/contracts/` e `src/server/`:

1. **Advertised vs canonical**: il client SDK converte gli schemi zod in JSON Schema; i test validano lo schema *dopo* la conversione, contro il dialetto dichiarato e contro chiamate reali (`advertisedInputExamples.test.ts`, `compactInputSchema.test.ts`, `toolContractExamples`).
2. **Union di oggetti appiattite** per compatibilità con API modello che rifiutano `anyOf/oneOf/allOf` alla radice: la proiezione fonde proprietà e descrive i gruppi; la validazione canonica resta quella esatta.
3. **Esempi mai con Evidence inline gigante**: si usa il riferimento `{"kind":"retained-evidence","evidence_id":"ev_…"}`; gli esempi non pubblicizzabili restano nei test.
4. **`$id` content-bound** sugli output schema: cambiare schema cambia identità, permettendo cache dei validator.
5. **Profilo compatto** per client con cap di dimensione (vedi §5.1).
6. **Admission**: `src/server/analysisAdmission.ts` + `admittedToolOperation.ts` serializzano le operazioni sul provider attivo; `sessionAvailabilityPolicy.ts` decide la disponibilità per host/client; le operazioni di lifecycle (`close_binary`) drenano le richieste precedenti.

---

## 10. Testing e verifica

`docs/testing.md` (1.418 righe) è il documento più lungo del repo e definisce **profondità comportamentali** con collocazione vincolante:

| Livello | Dove | Cosa prova |
| --- | --- | --- |
| Module | `src/**/*.test.ts` (87k righe) | un dominio/servizio/adapter tramite la sua superficie pubblica |
| Composition | `tests/composition/` | wiring sessione/registry; niente filesystem, processi, socket |
| Boundary | `tests/boundary/` (461 file di test) | esattamente **un** confine di produzione (fs, processo, rete, browser, CLI, provider) |
| MCP boundary | `tests/boundary/mcp/` | trasporto MCP e contratti, sessioni isolate con cleanup esplicito |
| Acceptance | `tests/acceptance/` | journey CLI/MCP compilati |
| Conformance | `tests/conformance/` | contratti provider condivisi, parametrizzati per capability dichiarate |
| Evaluation | `tests/evaluation/` | parsing/scoring deterministico del valutatore |
| Process-global | `tests/process-global/` (fork isolati) | dipendenze da stato globale di processo |

Comandi principali: `npm run test:local -- PATH`, `test:focused`, `test:changed`, `test:ci:shard` (sharding in CI), `test:boundary`, `test:mcp`, `test:acceptance`, `test:coverage`.

Le affermazioni sui **provider reali** non passano da Vitest: vivono in **82** script `npm run verify:*` (`verify:hopper`, `verify:hopper:fat64`, `verify:ghidra:mips`, `verify:ghidra:nativeaot`, `verify:ida`, `verify:jeb`, `verify:android`, `verify:adb`, `verify:browser:*`, `verify:electron`, `verify:managed`, `verify:firmware`, `verify:evm:interface`, `verify:recorded:crash`, `verify:published`, `verify:package(:windows)`, `verify:architecture-guards`, `verify:agent`, `verify:readiness`, …) e in lane per **client agent reale** (`verify:claude-client`, `verify:gemini-client`, `verify:qwen-client`, `verify:pi-client`, `verify:hermes-client`, `verify:opencode-client`, `verify:grok-client`, `verify:copilot-client`, `verify:deepseek-client`, `verify:commandcode-client`, `verify:omp-client`). Queste lane configurano profili usa-e-getta, verificano backup/idempotenza del setup, cataloghi completi e cleanup, e **dichiarano esplicitamente ciò che non provano** (es. "Token counts are synthetic; live Google API compatibility remains unverified").

CI (`.github/workflows/ci.yml`):
- job `pr-title` impone **titoli Conventional Commits** (necessari a release-please);
- job `changes` esegue `scripts/ci/plan.mjs`, che **classifica il diff** e attiva solo le lane pertinenti (`code`, `runtime`, `package`, `apple`, `fixtures`, `docs`, `browser`, `evm`, `nativeaot`, …), con etichetta `ci:full` per forzare tutto;
- build → test shardati → merge dei report; job dedicati per fixture di conformità, build nativa Windows, e workflow separati `real-*.yml` su runner self-hosted (macOS con Hopper, Ghidra, browser, ecc.);
- `pages.yml`/`website-pages.yml` per il sito, `release.yml` per la pubblicazione npm.

`npm run scan:todos`, `jscpd` (duplicazione), `knip` (export/codice morto), `oxlint`, `oxfmt`, `metadata:check` (generati vs sorgenti) completano la rete di controllo.

---

## 11. Build, tooling e release

- Toolchain pinnata: `.nvmrc` + `packageManager: npm@11.16.0`; `engines.node: ^22.19 || ^24.11 || >=26`.
- Orchestrazione: **Turborepo** (`turbo.json`, task `compile`, `typecheck`, `lint`, `lint:built`, `format:check`, `knip`, `metadata:check`, `artifacts:skills`, `test:run`, `mcp-catalog:check`, `docs:check`) con cache e input dichiarati; `scripts/run-exclusive.mjs artifacts …` per serializzare ciò che non è parallelizzabile.
- Build: `scripts/bundle-runtime.mjs` — dopo `tsc`, **rolldown** raggruppa gli entry point (`mcp/main.js`, `cli-runtime/{cli,cliOutput,…}`) con dipendenze esterne e import dinamici preservati, più un plugin (`@babel/parser`) che conserva le posizioni dei moduli compilati; `artifacts:skills` copia/valida la skill; `prepack.mjs` per il pacchetto npm; `install.sh` per l'installazione rapida. Toolchain TS dichiarata: `typescript 7.0.2`.
- Release: **release-please** (`release-please-config.json`, `versioning: always-bump-minor`), changelog con sezioni Highlights/Changes/Fixes, tag `rea-agents-X.Y.Z`, publish su npm di `rea-agents`, verifica `verify:mcp-registry-metadata` (server.json per il registry MCP).
- Documentazione: VitePress in `docs/`, generazione di `docs/public/product-catalog.json` e della catalogo errori (`docs/error-contract.schema.json`, `docs/user-visible-errors.csv`), `docs:check` in PR; sito pubblico servito da `website/public/` su Cloudflare Workers, più un host GitHub Pages opzionale.
- Igiene dei generati: `skills/`, `docs/public/product-catalog.json`, `docs/verification/managed-conformance-*.json` sono **output ignorati**; si modificano i sorgenti e si rigenera (`AGENTS.md`).

---

## 12. Convenzioni e "costituzione" del repository

`AGENTS.md` (7 KB) è scritto per gli agenti che lavorano *dentro* il repo e contiene regole forti, verificate da test:

- autorità: REA è local-only; Ghidra con DB effimeri; IDA read-only; setup additivo/idempotente con backup e approvazione; nessuna installazione di software non correlato;
- privacy: preservare input/paths/digest dell'utente, **redigere** credenziali di trasporto e valori marcati sensibili, non inferire segreti dal nome, non persistere l'ambiente per il solo fatto che un figlio lo eredita;
- pulizia: "Retain ownership of resources whose cleanup failed"; un PID + path non bastano per stabilire ownership dopo l'uscita o il riuso;
- anti-brittleness: quando si corregge un bug, guardare il **confine produttore** e i consumer, fare il refactor correlato e rimuovere il codice superato; un test di regressione verde non prova che la fragilità sia risolta;
- errori: algebra di errori taggata + `Result` (`src/domain/result.ts`, errori tipizzati in `src/domain/*Error*.ts`), preservando fatti parziali e cause;
- commit: Conventional Commits, breaking con `!` o footer `BREAKING CHANGE:`.

`CONTRIBUTING.md` aggiunge: giustificare il costo di review, non frammentare PR cosmetiche, eseguire `verify:test-discovery` e `verify:module-boundaries` quando si toccano test/import, e la regola "keep provider protocols in adapters".

---

## 13. Cosa vale la pena copiare (punti di forza)

1. **Contratti canonici + proiezione pubblicitaria separata**, con test che validano *ciò che il client riceve davvero* (dopo la conversione SDK) e non solo lo schema sorgente.
2. **Effetti dichiarati e annotazioni derivate**, con test che confrontano la prosa dei tool con gli effetti: previene la classe di bug "il tool dice di non eseguire codice ma lo esegue".
3. **Evidence con incertezza e unknown residui**, invece del solito JSON di risultato: rende l'agente onesto e verificabile.
4. **Ownership e cleanup come requisito di prima classe**, con osservazioni di possesso storiche e conservazione delle risorse non ripulite.
5. **Separazione netta dei livelli con guardie eseguibili** (`verify:module-boundaries`, `verify:architecture-guards`), non solo linee guida.
6. **Verifica reale nei lane dedicati**, con dichiarazione esplicita di ciò che resta non verificato (ricorrente in tutta la documentazione: "does not establish…", "unverified").
7. **CI che classifica il diff** e attiva solo le lane necessarie, con lane self-hosted per engine commerciali.
8. **Generati fuori dal repo** e `metadata:check`/`docs:check` in CI: deriva tra sorgenti e artefatti generati individuata subito.

## 14. Rischi, limiti e osservazioni critiche

1. **Superficie enorme.** 173 tool + 130 comandi + 17 famiglie di target: il costo di apprendimento e la scoperta dei tool da parte di un modello sono un problema reale (il repo lo ammette: per questo esistono `binary_session.tool_availability`, il profilo `compact`, l'evaluation delle sessioni agente e `docs/tool-design.md`). La pressione per aggiungere tool è il rischio strutturale principale.
2. **Dipendenze commerciali/esterne.** Hopper, IDA, JEB sono richiesti per le parti più profonde; molte lane di verifica girano solo su runner self-hosted con licenze. Chi clona il repo non può riprodurre la verifica reale senza attrezzatura specifica (Windows Ghidra "P0" è read-only e con copertura parziale).
3. **Documentazione ≠ codice.** 49 guide molto dettagliate sono anche un debito: `docs/mcp-contracts.md` descrive comportamenti di timeout/profile che devono restare sincronizzati col codice; esiste `npm run check-doc-facts` e `docs:check`, ma la superficie testuale è gigantesca (il README esiste in 19 lingue, ~20 KB ciascuno).
4. **Concentrazione autoriale.** Un autore domina la storia (`morluto`, ~58% dei commit): bus factor basso, nonostante i molti contributori esterni.
5. **Ritmo di release estremo** (v4.0.1 → v6.4.0 in 6 giorni). Il changelog è alto/basso per release: utile, ma rende difficile seguire le regressioni e suggerisce forte pressione sulle correzioni.
6. **Dimensioni del risultato.** `analyze_javascript_application` può produrre Evidence da centinaia di MB (esempio documentato: 334 MB per Obsidian): il progetto lo affronta con viste, paginazione e export bundle, ma il modello dati resta pesante.
7. **Nessuna verifica eseguita in questo studio.** Non ho eseguito `npm ci`/`npm run build`/test in sandbox (installazione grande, artefatti nativi Windows, lane che richiedono engine esterni): le affermazioni sul comportamento runtime derivano da codice, contratti e documentazione, non da esecuzione.
8. **Numeri GitHub e npm** sono snapshot al 2026-10-12 e non vanno usati come indicatori di qualità (le 132k stelle sono in parte un fenomeno virale).

---

## 15. Dove mettere le mani: mappa per attività tipiche

| Attività | File da toccare (ordine consigliato) |
| --- | --- |
| Aggiungere un tool MCP | `src/contracts/<famiglia>ToolContracts.ts` → `src/contracts/toolEffects.ts` → `src/contracts/toolContracts.ts` (aggregazione) → `src/application/**` (workflow) → `src/server/register*Tools.ts` → test di contratto/boundary → `npm run mcp-catalog:generate` + `docs:generate` |
| Aggiungere un comando CLI | `src/cliCommandNames.ts` → `src/cli/*Commands.ts` → registration in `src/cli.ts` → rigenerare catalogo (`catalogIdentity` dipende da `CLI_COMMAND_NAMES`) |
| Nuovo provider di analisi profonda | `src/<provider>/` (client + capabilities + profile) → `src/composition/*.ts` → `src/application/AnalysisProviderRegistry.ts` / `SessionProviderRouter` → `src/application/Doctor*` → `verify:<provider>` lane + `docs/<provider>-provider.md` |
| Cambiare il modello di Evidence | `src/domain/evidence.ts`, `evidenceBundle.ts`, `residualUnknown.ts` → `src/application/investigation/*` → schemi di sessione `src/contracts/sessionToolContracts.ts` → migrazioni doc (`docs/migration-*.md`, `docs/reconstruction-obligation-ledgers.md`) |
| Supportare un nuovo client agente | `src/application/SupportedClients.ts` (+ `SetupClients.ts`, `ClientConfigurationFiles.ts`) → `scripts/verify-*-client.mjs` → tabella in `docs/installation.md` |
| Tuning di timeout/limiti provider | `src/process/ProviderDeadline.ts`, `src/config/*`, variabili `REA_*` documentate in `docs/mcp-contracts.md` e `docs/installation.md` |
| Setup/installazione | `src/application/Setup*.ts`, `src/application/Uninstall.ts`, `src/cli/interactiveSetup.ts`, `install.sh` (regola: additivo, idempotente, backup, approvazione) |
| Sito e guide | `docs/**` + `docs/.vitepress/config.ts` (link relativi!), `website/**` per il deploy Cloudflare |

Comandi utili in locale:

```bash
npm ci
npm run check:fast                     # typecheck+lint cached + pre-push check
npm run test:focused -- src/config.test.ts
npm run build:cached                   # runtime + skill pacchettizzata
npm run check:pr                       # gate completo (opzionale, lungo)
npm run docs:dev                       # sito VitePress
npm run verify:module-boundaries       # confini di import
npm run verify:test-discovery          # dopo aver aggiunto/spostato test
```

---

## 16. Riferimenti interni più utili (lettura consigliata)

1. `README.md` + `docs/installation.md` — orientamento e requisiti.
2. `AGENTS.md` — le regole che il repo considera non negoziabili.
3. `docs/architecture.mermaid` — mappa completa dei livelli e delle frecce.
4. `docs/mcp-contracts.md` (570 righe) — semantica runtime di sessione, Evidence, progress, timeout Ghidra, profilo compatto.
5. `docs/tool-design.md` (92 righe) — come decidere se e come aggiungere un tool.
6. `docs/testing.md` (1.418 righe) — livelli, lane reali, criteri di pruning.
7. `src/domain/evidence.ts` + `src/application/investigation/EvidenceLedger.ts` — il modello di prova.
8. `src/process/ProcessOwnership*.ts` — il modello di possesso delle risorse.
9. `src/contracts/toolEffects.ts` + `src/contracts/toolContracts.ts` — inventario e autorità dichiarata.
10. `.github/workflows/ci.yml` + `scripts/ci/plan.mjs` — come il progetto decide cosa verificare.

---

## 17. Domande aperte (per il passo successivo)

1. **Perché questo studio?** Se l'obiettivo è un fork o un prodotto derivato (il repo di sessione `michelelomartire-del/IEH` è oggi vuoto) conviene decidere da subito se si eredita la disciplina di Evidence/sessioni o se si vuole un sottoinsieme (es. solo JavaScript/Electron, o solo nativi).
2. **Serve una verifica eseguibile?** Posso provare `npm ci` + `npm run build:cached` + `npm run check:fast` in sandbox (niente engine commerciali: le lane `verify:hopper`/`verify:ghidra`/`verify:ida` resterebbero non eseguibili).
3. **Serve un confronto mirato** tra REA e ciò che vuoi costruire in IEH (funzionalità, modello di evidenza, contratti MCP, setup multi-agente, packaging npm)?
4. **Serve una lettura guidata di un'area specifica** (es. `src/browser/**` CDP, `src/ghidra/**` lifecycle, `src/application/Setup*` per l'integrazione con 18 client)?
