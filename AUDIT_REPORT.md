# Audit Report — fattureincloud-mcp v1.8.0

> **Generato:** 2026-05-10
> **Branch:** `main` (HEAD `02834af`)
> **Scopo:** baseline pre-pianificazione v1.9.0 (rinominata da "v1.3.5" del BRIEF originale, ormai obsoleto).
> **Modalità:** sola lettura. Nessuna modifica al codice.

---

## TL;DR

- Repo a **v1.8.0** (PyPI + codice). BRIEF.md descrive uno stato **v1.3** ormai superato di 5 minor.
- **Drift README/codice (Task A0 del BRIEF) GIÀ RISOLTO**: README e codice espongono entrambi 20 tool con nomi allineati.
- **Cache: assente.** Nessuna implementazione locale, nessun riferimento `cache` in `server.py`.
- **Cost/revenue centers: assenti.** Nessun tool dedicato, nessun parametro su documenti emessi/ricevuti.
- **Test: assenti.** Nessuna directory `tests/`, copertura 0%.
- **Tool annotations MCP (`readOnlyHint`/`destructiveHint`/...): assenti.** I tool sono dichiarati con `name`/`description`/`inputSchema` soltanto.
- **Architettura: monolitica.** Tutto in `server.py` (1182 righe), SDK MCP raw (non FastMCP).
- **Nessun git tag** in repo. PyPI ha pubblicato solo 5 versioni (1.1.0, 1.2.0, 1.3.0, 1.6.4, 1.8.0); le altre del CHANGELOG non sono mai arrivate al registry.
- **Bug noto duplicate-fattura: nessuna traccia nel codice** (no TODO/FIXME, no issue GitHub).

Lo scope tematico v1.3.5 (cache + cost centers + test) **è ancora interamente da fare** sulla v1.8 di partenza. Il delta è quasi solo di numerazione: il rebranding a `v1.9.0` è coerente; valutare se serve anche un mini-bump intermedio per alcuni delivery.

---

## 1. Versione PyPI e tag git

| Campo | Valore |
|---|---|
| Versione PyPI corrente | **1.8.0** |
| Versione `pyproject.toml` | **1.8.0** |
| Versione header `server.py` | **1.8.0** |
| Tag git nel repo | **nessuno** |
| Versioni storiche su PyPI | `1.1.0`, `1.2.0`, `1.3.0`, `1.6.4`, `1.8.0` |
| Default branch | `main` |
| Ultimo commit | `02834af` ("v1.8.0: rimuovi mark_payment_paid/unpaid/list_payment_accounts, aggiungi convert_proforma, get_pdf_url, create/update_client") |
| Last push | 2026-02-20 |
| Stars / forks / watchers | 10 / 6 / 10 |
| Contributor unico | `aringad` (Giuliano Delfino, Mediaform) |

**Note critiche:**

1. **Nessun git tag.** Le versioni sono tracciate solo nei messaggi di commit e nel CHANGELOG. Da v1.9.0 in avanti raccomandato `git tag vX.Y.Z` su ogni release.
2. **Drift PyPI vs CHANGELOG.** v1.4.0, v1.5.0, v1.6.0–v1.6.3, v1.7.0–v1.7.2 sono presenti nel CHANGELOG/commit ma **NON pubblicati su PyPI**. Solo 1.6.4 e 1.8.0 hanno superato il salto. v1.7.x è stata effettivamente sviluppata e poi parzialmente rimossa: v1.8.0 elimina `mark_payment_paid`/`mark_payment_unpaid`/`list_payment_accounts` introdotti in v1.7.0–7.2.
3. **CHANGELOG.md non riporta v1.7.0–v1.7.2.** Quei tre commit esistono in git log ma sono "saltati" nel CHANGELOG, che passa direttamente da v1.6.4 a v1.8.0.

---

## 2. CHANGELOG da v1.3 a v1.8

Estrazione testuale dal CHANGELOG.md attuale (più cosa il git log aggiunge per la finestra v1.7 mancante).

### v1.3.0 (PyPI)
- FIX: `create_invoice` include `ei_code` dall'anagrafica
- FIX: `duplicate_invoice` aggiorna `ei_code`
- NEW: `check_numeration`

### v1.4.0 (NON pubblicato su PyPI)
- NEW: `list_invoices` accetta parametro `type`

### v1.5.0 (NON pubblicato su PyPI)
- NEW: `create_credit_note` (NDC; importi positivi in input, negativi automatici; `source_invoice_id` opzionale)
- NEW: `create_proforma` (proforma, non inviabile a SDI)
- CHANGE: `list_invoices` accetta parametro `type`: `invoice` (default), `credit_note`, `proforma`

### v1.6.0 (NON pubblicato su PyPI)
- NEW: `update_document` (modifica parziale di qualsiasi documento bozza: data, oggetto, righe, giorni pagamento)

### v1.6.1 / v1.6.2 / v1.6.3 (NON pubblicati su PyPI)
- Tre tentativi falliti di FIX per NDC (abs su payment, payment negativo, senza payments_list)

### v1.6.4 (PyPI)
- FIX: `create_credit_note` — prezzi e payment positivi (FIC inverte internamente per `type=credit_note`)
- FIX: `update_document` — stessa logica per NDC

### v1.7.0 — v1.7.2 (solo git log, NON in CHANGELOG, NON su PyPI)
Da git log:
- v1.7.0: aggiunti `mark_payment_paid`, `mark_payment_unpaid`, `convert_proforma`, `get_pdf_url`, `create/update_client`, `get_situation` migliorato
- v1.7.1: fix `mark_payment_paid` con `payment_account` obbligatorio, aggiunto `list_payment_accounts`
- v1.7.2: rimossa `PaymentAccountsApi` (non nel SDK), uso REST diretto per conti

### v1.8.0 (PyPI corrente)
- NEW: `convert_proforma_to_invoice` (elimina proforma originale di default; `keep_proforma=True` per mantenerla)
- NEW: `get_pdf_url` (URL PDF + link web)
- NEW: `create_client`
- NEW: `update_client`
- FIX: `get_situation` ora sottrae le NDC dal fatturato (`fatturato_netto = fatture - NDC`) e supporta filtro per cliente
- REMOVED: `mark_payment_paid`, `mark_payment_unpaid`, `list_payment_accounts` — l'API FIC richiede un conto di saldo obbligatorio non recuperabile in modo affidabile via SDK

**Implicazioni per v1.9.0:**
- La numerazione PyPI vs codice è coerente; la storia è "saltellante" ma l'utente che fa `pip install fattureincloud-mcp` ottiene 1.8.0 con tutto lo scope corretto.
- Il debito tecnico storico più visibile: tre minor di tentativi NDC (v1.6.1–6.3) + un'intera linea v1.7 abortita. Conferma che FIC API ha edge case scivolosi su NDC e payment accounts. Da considerare per la roadmap futura (no `mark_payment_paid` finché l'API non lo permette).

---

## 3. Tool esposti oggi (20 totali)

Estratto da `server.py` `@app.list_tools()` (righe 176–468). Ogni `Tool()` ha `name`, `description`, `inputSchema`. **Nessuno** ha `annotations`.

| # | Nome tool | Read-only? | Mutativo? | Descrizione | Annotations attuali |
|---|---|---|---|---|---|
| 1 | `list_invoices` | ✓ | | Lista documenti emessi (anno/mese, type=invoice/credit_note/proforma) | nessuna |
| 2 | `get_invoice` | ✓ | | Dettaglio documento per ID | nessuna |
| 3 | `get_pdf_url` | ✓ | | URL PDF e link web | nessuna |
| 4 | `list_clients` | ✓ | | Lista clienti (filtro opzionale) | nessuna |
| 5 | `get_company_info` | ✓ | | Info azienda collegata | nessuna |
| 6 | `create_client` | | ✓ | Crea cliente | nessuna |
| 7 | `update_client` | | ✓ | Aggiorna cliente | nessuna |
| 8 | `create_invoice` | | ✓ | Crea fattura (bozza) | nessuna |
| 9 | `create_credit_note` | | ✓ | Crea NDC (bozza) | nessuna |
| 10 | `create_proforma` | | ✓ | Crea proforma (bozza) | nessuna |
| 11 | `convert_proforma_to_invoice` | | ✓ | Converte proforma → fattura (elimina sorgente di default) | nessuna |
| 12 | `update_document` | | ✓ | Modifica parziale documento bozza | nessuna |
| 13 | `duplicate_invoice` | | ✓ | Duplica fattura con nuova data | nessuna |
| 14 | `delete_invoice` | | ✓ (distruttivo) | Elimina documento bozza | nessuna |
| 15 | `send_to_sdi` | | ✓ (irreversibile) | Invia documento a SDI | nessuna |
| 16 | `get_invoice_status` | ✓ | | Stato e-invoice/SDI | nessuna |
| 17 | `send_email` | | ✓ (effetto esterno) | Invia copia cortesia via email | nessuna |
| 18 | `list_received_documents` | ✓ | | Lista fatture passive | nessuna |
| 19 | `get_situation` | ✓ | | Dashboard finanziaria (fatturato netto, incassato, costi, margine; filtro cliente) | nessuna |
| 20 | `check_numeration` | ✓ | | Verifica continuità numerica fatture | nessuna |

**Conteggio:** 11 read-only, 9 mutativi (1 distruttivo `delete_invoice`, 1 irreversibile `send_to_sdi`, 1 con effetti esterni email `send_email`).

**Mappa annotations target (per fase MCPB):** vedi BRIEF Appendice D, da rivedere completamente perché alcuni nomi citati lì non esistono più (`*_issued_document`, `delete_client`, ecc.).

---

## 4. Sistema di cache

**Esito:** **NON ESISTE.**

- `grep -niE 'cache' server.py` → 0 match.
- Nessun modulo `cache.py` nel repo.
- `requirements.txt` non include alcuna libreria di caching.
- Nessuna env var `*_CACHE_*` documentata.

**Conseguenza:** ogni invocazione di tool che ha bisogno di anagrafica (es. `create_invoice` chiama `get_client_by_id` e `get_ei_code_for_client` per costruire l'entity) effettua una chiamata API live a FattureInCloud. Più precisamente:

- `build_entity_from_client(client_id)` chiama `get_client_by_id(client_id)` → API `clients_api.get_client(...)` (server.py:54).
- `get_ei_code_for_client(client_id)` chiama anche lui `get_client_by_id` (server.py:62) → seconda chiamata per lo stesso client_id.
- Su `create_invoice` la stessa anagrafica può essere fetchata **due volte** in un singolo flusso.

Tutto lo scope cache del BRIEF originale è ancora valido e applicabile alla v1.8 senza adattamenti. La reference implementation in BRIEF Appendice E si può integrare 1:1.

---

## 5. Cost / revenue centers

**Esito:** **NON ESPOSTI.**

- `grep -niE 'cost_cent|revenue_cent' server.py` → 0 match.
- Nessun tool `list_cost_centers`.
- `create_invoice`, `create_credit_note`, `create_proforma`, `convert_proforma_to_invoice`, `update_document`: **nessun parametro** `revenue_center`.
- `list_received_documents` (l'unico tool sui documenti passivi): **nessun parametro** `cost_center`. Inoltre **mancano** `get_received_document` e `create_received_document` — il flow received è solo "list".

Lo scope cost centers del BRIEF è interamente da implementare. Confermato candidato per v1.9.0.

---

## 6. Drift README ↔ codice (Task A0 BRIEF)

**Esito:** **GIÀ RISOLTO.**

- Codice: 20 tool (vedi § 3).
- README.md sezione "Funzionalità (20 tool)" e "Features (20 tools)": entrambe elencano gli stessi 20 nomi del codice.
- Nessuna discrepanza di naming, descrizione corta o presenza/assenza.

Il BRIEF cita un drift "11 vs 20" che era reale alla v1.3 ma è stato sanato in (almeno) v1.8.0. Il Task A0 può essere chiuso.

**Da rifare comunque in v1.9.0:** quando si aggiungeranno i tool/parametri cost-center, README e CHANGELOG vanno aggiornati in lockstep per evitare nuovo drift.

---

## 7. SDK MCP

| Campo | Valore |
|---|---|
| Pacchetto | `mcp` (SDK ufficiale low-level Python) |
| Pin in `requirements.txt` | `mcp>=1.0.0` |
| Pattern utilizzato | Server raw + decoratori `@app.list_tools()` / `@app.call_tool()` |
| Import | `from mcp.server import Server`, `from mcp.server.stdio import stdio_server`, `from mcp.types import Tool, TextContent` |
| FastMCP | non usato |

**Versione installata:** non determinabile in questo audit (no venv attivo, requirements solo con floor `>=1.0.0`). Da verificare in fase venv pre-v1.9 con `pip show mcp`.

**Considerazioni roadmap:**
- L'SDK low-level espone già il campo `annotations` su `Tool` (parte della spec MCP 2024-11+). L'aggiunta delle annotations richieste dalla submission Anthropic è un cambio puramente dichiarativo, no rifattorizzazione.
- Migrazione a FastMCP **non necessaria** per v1.9 né per il bundle MCPB. Il pattern attuale è valido.

---

## 8. Test coverage

**Esito:** **0%.**

- Nessuna directory `tests/`.
- Nessun file `test_*.py` o `*_test.py` nel repo.
- `requirements.txt` non include `pytest`/`pytest-cov`/altro framework di test.
- Nessun workflow CI in `.github/`.

**Implicazioni per v1.9.0:**
- Lo scope test del BRIEF (`test_cache.py`, regressione drop-chiamate-API durante `create_invoice`, e2e cost-center) richiede creazione completa della struttura test da zero.
- Serve setup minimale: `pytest`, `pytest-cov` in `requirements-dev.txt` o in extras `pyproject.toml`, fixture per mock `clients_api`/`issued_api`.
- Target proposto BRIEF: copertura ≥80% sui moduli nuovi (`cache.py`, tool cost-related). Realistico se si testano i nuovi e si lasciano i vecchi senza copertura per ora.

---

## 9. Issue GitHub aperte

| # | Stato | Titolo | Rilevanza per v1.9 |
|---|---|---|---|
| 3 | open | "Add page and per_page in list_invoices tool and multi page in get_situation" | **MEDIA** — paginazione utile su account con molti documenti. Non blocca lo scope cache+cost-centers ma è candidata in scope se Giuliano vuole una "v1.9 anagrafiche & efficienza" più larga. |
| 2 | open | "🎨 PREA was here — discovered fattureincloud-mcp" | nessuna — saluto/discovery, non actionable. |

**Pull Request aperte:** nessuna.

---

## 10. Architettura e struttura repo

```
fattureincloud-mcp/
├── .env.example          # FIC_ACCESS_TOKEN, FIC_COMPANY_ID, FIC_SENDER_EMAIL
├── .gitignore            # include .env, venv/, .venv/, __pycache__, build/, dist/, ecc.
├── CHANGELOG.md          # storia v1.1.0 → v1.8.0 (gap v1.7.x non documentato)
├── LICENSE               # MIT
├── README.md             # bilingue IT/EN, ~280 righe, 20 tool documentati allineati al codice
├── pyproject.toml        # version=1.8.0
├── requirements.txt      # fattureincloud-python-sdk>=2.0.0, mcp>=1.0.0, python-dotenv>=1.0.0
├── server.json           # presente (da verificare contenuto: probabilmente metadati MCP registry)
└── server.py             # 1182 righe, monolitico
```

**Nota su `server.json`:** non in BRIEF. Da indagare in fase BRIEF-realignment.

**Funzioni in `server.py`:**

```
44   get_total_from_doc
52   get_client_by_id
60   get_ei_code_for_client      (chiama get_client_by_id → potenziale doppio fetch)
75   build_entity_from_client    (chiama get_client_by_id + get_ei_code_for_client)
99   build_items_list
116  build_issued_document
177  list_tools (decorator MCP)
472  call_tool (decorator MCP)   → 700 righe di if/elif name == "..." per dispatch
1175 main (entry point asyncio)
```

**Pattern dispatch:** `call_tool` è uno switch monolitico su `name`. Per il refactor MCPB (Fase 4 BRIEF) potrebbe valere la pena spezzare in moduli `tools/*.py`, ma è opzionale per v1.9.0 (cache + cost centers stanno bene anche dentro il monolite).

---

## 11. Convenzioni env var (drift vs BRIEF)

| Variabile | Codice (server.py:27-29) | BRIEF v1.3.5 prevedeva |
|---|---|---|
| Token API | `FIC_ACCESS_TOKEN` | `FATTUREINCLOUD_API_TOKEN` |
| Company ID | `FIC_COMPANY_ID` | `FATTUREINCLOUD_COMPANY_ID` |
| Email mittente | `FIC_SENDER_EMAIL` | (non previsto nel BRIEF) |
| Cache dir | (assente) | `FATTUREINCLOUD_MCP_CACHE_DIR` |
| Cache disable | (assente) | `FATTUREINCLOUD_MCP_CACHE_DISABLED` |

Il BRIEF è disallineato. Il prefisso reale `FIC_*` è già stabilito su PyPI v1.8 e nel README — cambiarlo ora romperebbe l'installazione di chiunque sta già usando il pacchetto. Raccomandazione: **mantenere `FIC_*`** e aggiornare BRIEF/Appendici A (manifest), B (README), C (privacy), E (cache) di conseguenza. Le nuove env var di cache useranno `FIC_CACHE_DIR` e `FIC_CACHE_DISABLED` per coerenza.

---

## 12. Bug noto duplicate-fattura

- `grep -niE 'TODO|FIXME|XXX|HACK' server.py` → **0 match**. Nessun marker nel codice.
- `duplicate_invoice` è definito a server.py:365 (Tool) e server.py:867 (handler).
- v1.3.0 CHANGELOG: "FIX: `duplicate_invoice` aggiorna `ei_code`" (correzione passata).
- Nessuna issue GitHub aperta o chiusa che descrive il bug "cliente specifico" del BRIEF Parking lot.

**Conclusione:** il bug citato nel BRIEF non ha tracce documentali nel repo. Senza dati cliente concreti (anagrafica, ID fattura sorgente, errore esatto, log) la riproduzione non è realistica. Conferma: parking lot mantenuto per future v1.9.x bugfix se i dati arrivano dal cliente.

---

## 13. Confronto BRIEF v1.3.5 ↔ realtà v1.8.0

| Voce BRIEF | Stato attuale | Azione per v1.9 |
|---|---|---|
| Cache file-based (Appendice E) | Assente | **Da implementare** ex novo |
| Wrap fetch anagrafica con `@cached` | Assente | **Da implementare** (target: `get_client_by_id`, `get_ei_code_for_client`) |
| Tool `list_cost_centers` | Assente | **Da implementare** |
| Param `revenue_center` su issued docs | Assente | **Da implementare** su create_invoice/credit_note/proforma/convert/update_document/duplicate_invoice |
| Param `cost_center` su received docs | Assente, e mancano anche `get_received_document` / `create_received_document` | **Da implementare** (allargare scope se vogliamo received completi, oppure limitarsi al param sui list) |
| Test end-to-end cost centers | Assente (no test affatto) | **Da implementare** stack pytest + e2e |
| Test regressione cache (drop chiamate API) | Assente | **Da implementare** |
| CHANGELOG aggiornato | Allineato a v1.8 | Aggiungere entry v1.9.0 a fine ciclo |
| README aggiornato | Allineato a v1.8 (no drift) | Aggiungere sezioni Caching, Cost centers, Known issues a fine ciclo |
| Tag git `v1.9.0` | Mai usato in storia | **Da introdurre** (anche retroattivo per `v1.8.0` se utile) |
| Refactor MCPB (Fase 4) | Non avviato | Posticipato a v2.0.0 (post-PyPI v1.9.0), come da BRIEF |
| Bug duplicate-fattura | Nessuna traccia | Parking lot — invariato |

**Voci BRIEF non più valide:**

- Tool annotation matrix Appendice D cita `*_issued_document` / `delete_client` / `list_suppliers` / `list_products` / `get_einvoice_xml` / `schedule_email` / `get_received_document` / `create_received_document`: **nomi inesistenti** nel codice o comunque non presenti su PyPI. Vanno sostituiti con i nomi reali di § 3.
- Citazione "drift README/codice 11 vs 20" (Task A0 BRIEF § 1.2): **risolto, da rimuovere o mettere come done**.
- Versioni `v1.3.5` ovunque nel BRIEF e CLAUDE.md: **da rinominare a v1.9.0** in lockstep.
- Env var prefix `FATTUREINCLOUD_*` nel BRIEF: **da sostituire con `FIC_*`** dove non già fatto.

---

## 14. Raccomandazioni per la pianificazione v1.9.0

Da discutere insieme prima di scrivere il nuovo BRIEF:

1. **Numerazione**: confermare `v1.9.0` come minor bump (cache + cost centers + test = scope tematico significativo, giustifica minor). Alternativa: `v1.8.5` se vogliamo mantenere "1.8 series" e poi saltare a 2.0 con MCPB. Suggerito v1.9.0.
2. **Scope deciso**:
   - **In:** modulo cache + decorator + integrazione anagrafiche (BRIEF § 6.1–6.2), tool `list_cost_centers` (§ 6.3), `revenue_center` su issued (§ 6.4 prima metà), `cost_center` come parametro opzionale dove sensato.
   - **Da decidere:** completamento received documents (`get_received_document`, `create_received_document`) — è un'estensione coerente con cost centers ma estende lo scope. Pro: API simmetrica e completa. Contro: aumenta superficie di test ed errori.
   - **Da decidere:** issue #3 (paginazione `list_invoices` / `get_situation`). Pro: utile a utenti con molti dati. Contro: nuovo asse, fuori tema "anagrafiche & efficienza".
   - **Out:** annotations MCPB (rinviate a v2.0 con bundle), refactor modulare (rinviato a v2.0), `mark_payment_*` (bloccato da limiti API).
3. **Stack test**: pytest + pytest-cov + ruff. `requirements-dev.txt` separato (più pulito). Target ≥80% sui nuovi moduli, nessun obbligo retroattivo.
4. **Tagging**: introdurre `git tag vX.Y.Z` da v1.9.0; opzionalmente retroattivo per `v1.8.0` (per documentare lo storico).
5. **Privacy / trademark / submission Anthropic**: tutto rinviato a v2.0 (post-PyPI v1.9). Non toccare in questa fase.

---

## Appendice — Checklist pre-pianificazione v1.9 (per Giuliano)

Decisioni da prendere prima di riscrivere il BRIEF:

- [ ] Conferma numerazione `v1.9.0` (vs `v1.8.5`)
- [ ] Scope received documents: solo `cost_center` su `list_received_documents`, oppure anche `get_received_document`/`create_received_document`?
- [ ] Issue #3 (paginazione): in scope o fuori?
- [ ] Conferma prefisso env var `FIC_*` (mantenuto, BRIEF da aggiornare)
- [ ] Nuove env var cache: `FIC_CACHE_DIR` / `FIC_CACHE_DISABLED` o nomi diversi?
- [ ] Tag git retroattivo per `v1.8.0`: sì/no?
- [ ] Account FIC per testing: produzione Mediaform (come v1.0–1.8), nuovo account test, oppure entrambi (test per CI, prod per smoke locale)?

---

*Audit prodotto in modalità sola-lettura. Nessuna modifica al codice. Nessun branch creato. Nessun commit oltre questo report.*
