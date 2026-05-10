# Changelog

## v1.9.0
- NEW: `list_cost_centers` — lista i centri di costo/ricavo configurati in FattureInCloud
- NEW: `get_received_document` — dettaglio fattura passiva per ID
- NEW: `create_received_document` — crea documento passivo (expense / credit_note) con fornitore, importi, categoria, descrizione, opzionale `cost_center`
- NEW: parametro opzionale `revenue_center` su `create_invoice`, `create_credit_note`, `create_proforma`, `convert_proforma_to_invoice`, `update_document`, `duplicate_invoice` (validato contro `list_cost_centers`; convert e duplicate ereditano dal documento sorgente se non passato)
- NEW: parametro opzionale `cost_center` su `create_received_document`
- NEW: cache locale file-based per fetch di anagrafica clienti e centri di costo (key per `company_id`, TTL 24h). Configurabile via `FIC_CACHE_DIR` (default `~/.fattureincloud-mcp/cache/`) e disattivabile via `FIC_CACHE_DISABLED=1`. Riduce le chiamate API ridondanti durante il flusso di creazione fatture (la stessa anagrafica veniva fetchata due volte, ora una sola)
- CHANGE: `get_invoice`, `list_invoices`, `list_received_documents` espongono `revenue_center`/`cost_center` nei risultati quando presenti
- KNOWN ISSUE: la duplicazione fatture può fallire per configurazioni cliente specifiche (workaround: duplicare manualmente dal pannello web FIC). Tracciato in `docs/KNOWN_ISSUES.md`
- NOTE: il modulo `cache.py` è collocato a top level; sarà spostato in `server/cache.py` durante un futuro refactor strutturale del modulo `server`

## v1.8.0
- NEW: `convert_proforma_to_invoice` — converte proforma in fattura elettronica (elimina proforma di default, `keep_proforma=True` per mantenerla)
- NEW: `get_pdf_url` — restituisce URL PDF e link web del documento
- NEW: `create_client` — crea nuovo cliente in anagrafica
- NEW: `update_client` — aggiorna dati cliente esistente
- FIX: `get_situation` — ora sottrae le NDC dal fatturato (fatturato_netto = fatture - NDC) e supporta filtro per cliente
- REMOVED: `mark_payment_paid` / `mark_payment_unpaid` / `list_payment_accounts` — l'API FIC richiede un conto di saldo obbligatorio non recuperabile in modo affidabile via SDK. La marcatura pagamenti va eseguita direttamente dal pannello FIC.

## v1.6.4
- FIX: `create_credit_note` — prezzi e payment positivi, FIC inverte internamente per type=credit_note
- FIX: `update_document` — stessa logica per NDC

## v1.6.3
- FIX: tentativo NDC senza payments_list (non funzionava)

## v1.6.2
- FIX: tentativo NDC con payment negativo (non funzionava)

## v1.6.1
- FIX: tentativo abs() su payment per NDC (non funzionava)

## v1.6.0
- NEW: `update_document` — modifica parziale di qualsiasi documento bozza (fattura, NDC, proforma): data, oggetto, righe, giorni pagamento. Carica l'originale e applica solo i campi passati.

## v1.5.0
- NEW: `create_credit_note` — crea nota di credito; importi positivi in input, negativi automaticamente; `source_invoice_id` opzionale
- NEW: `create_proforma` — crea proforma, non inviabile allo SDI
- CHANGE: `list_invoices` accetta parametro `type`: `invoice` (default), `credit_note`, `proforma`

## v1.4.0
- NEW: `list_invoices` accetta parametro `type`

## v1.3.0
- FIX: `create_invoice` include `ei_code` dall'anagrafica
- FIX: `duplicate_invoice` aggiorna `ei_code`
- NEW: `check_numeration`

## v1.2.0
- NEW: `delete_invoice`
- NEW: `payment_days` in `duplicate_invoice`

## v1.1.0
- Release iniziale
