# Simulatore di convenienza economica: AI locale (open-weight) vs Cloud + API

Applicazione web statica, in un unico file HTML senza dipendenze esterne, per confrontare il costo totale di possesso (TCO) e l'adeguatezza di **due** alternative di implementazione di un sistema di AI generativa:

- **A — Locale / on-premise**: modello open-weight eseguito su GPU di proprietà, con data warehouse (DWH) e integrazioni gestite internamente.
- **B — Cloud + API**: modello accessibile via API a consumo, con DWH e piattaforma gestiti dal provider.

Il simulatore non tratta soluzioni ibride. Produce un output in 12 parti (più una tabella riepilogativa degli input) che termina con una **raccomandazione condizionata** del tipo "A conviene se…; B conviene se…", e si astiene dal scegliere quando mancano dati ad alta sensitività.

> Tutti i valori predefiniti (prezzi GPU, tariffe API, tariffe orarie, ecc.) sono **ipotesi di partenza modificabili**, non riferimenti di mercato. Ogni sezione di input consente di indicare fonte, data e livello di confidenza del dato.

---

## Indice

- [Struttura del repository](#struttura-del-repository)
- [Avvio e deploy](#avvio-e-deploy)
- [Flusso di lavoro](#flusso-di-lavoro)
- [Fase 1 — Pre-valutazione rapida (15 variabili)](#fase-1--pre-valutazione-rapida-15-variabili)
- [Fase 2 — Analisi completa e disaggregata](#fase-2--analisi-completa-e-disaggregata)
- [Modello di calcolo](#modello-di-calcolo)
- [Output: le 15 card di risultato](#output-le-15-card-di-risultato)
- [Import / export JSON](#import--export-json)
- [Sistema visivo](#sistema-visivo)
- [Test](#test)
- [Limiti noti](#limiti-noti)
- [Licenza](#licenza)

---

## Struttura del repository

```
simulatore/
├── index.html     # applicazione completa (HTML + CSS + JS, ~860 righe, nessuna dipendenza)
├── guida.html     # guida all'uso, formule, glossario (~50 termini), formato JSON
├── esempi/        # tre JSON sintetici per la pre-valutazione rapida + README
├── design.md      # specifica dell'interfaccia (v1, v2, v2.1, sistema visivo)
└── test/          # harness jsdom (dom2.js, quick.js, …) — non necessario per il deploy
```

Per la pubblicazione sono sufficienti `index.html`, `guida.html` e, opzionalmente, `esempi/`.

## Avvio e deploy

Nessuna build, nessun server applicativo, nessuna chiamata di rete a runtime.

- **Locale**: aprire `index.html` con un browser moderno (Chrome, Edge, Firefox, Safari), oppure `python3 -m http.server 8080` nella cartella e visitare `http://localhost:8080/`.
- **Netlify / GitHub Pages / qualsiasi hosting statico**: pubblicare la cartella così com'è (publish directory = `simulatore`, build command vuoto).

I dati inseriti restano nel browser dell'utente; nulla viene inviato a server esterni.

## Flusso di lavoro

1. **Pre-valutazione rapida** (facoltativa): 15 variabili di alto livello → esito euristico in pochi secondi, con eventuale trasferimento automatico dei valori nella fase 2.
2. **Analisi completa**: 10 schede di input (A–J, circa 100 variabili + catalogo task + integrazioni) → pulsante **Calcola** → 15 card di risultato con grafici interattivi.
3. **Revisione**: la card 13b elenca ogni input con il flag *dato/ipotesi*, la fonte e la confidenza dichiarate; la card 13 indica i dati mancanti più influenti sul risultato.
4. **Export**: l'intero stato (input, task, integrazioni, metadati) è esportabile e reimportabile in JSON.

## Fase 1 — Pre-valutazione rapida (15 variabili)

Confronto preliminare basato su euristiche di build-vs-buy diffuse nella letteratura di settore (differenziazione, vantaggio dati, sovranità, time-to-value, capacità del team, utilizzazione dell'hardware). Le variabili sono volutamente diverse, in parte, da quelle della fase 2:

| Variabile | Tipo | Uso nell'euristica |
|---|---|---|
| `use_case` | scelta (5 archetipi) | preimposta profilo di token, latenza e tool call |
| `active_users` | numero | plausibilità del carico |
| `tasks_per_day` | numero | carico medio → utilizzazione GPU |
| `peak_to_average_ratio` | numero | dimensionamento a picco |
| `annual_growth_pct` | % | proiezione del carico sull'orizzonte |
| `model_parameters_b` | miliardi di parametri | memoria GPU, throughput, costo API |
| `quality_gap` | Nessuno / Piccolo / Medio / Grande | gap qualitativo open-weight vs frontier |
| `data_sensitivity` | Bassa / Media / Alta / Critica | *Critica* esclude il cloud (vincolo eliminatorio) |
| `data_tb` | TB | costo DWH e concorrenza CPU/IO in locale |
| `integrations_count` | numero | ore di integrazione, latenza tool call |
| `total_budget` | valuta | vincolo eliminatorio sul CAPEX iniziale |
| `horizon_years` | anni | ammortamento e NPV |
| `deadline_months` | mesi | procurement GPU vs scadenza go-live (eliminatorio) |
| `internal_it_capability` | Nessuna / Base / Buona / Forte | FTE operations e rischio esecuzione |
| `annual_benefit` | valuta | valore netto e costo del ritardo |

L'esito riporta: alternativa suggerita o "non determinabile", vincoli eliminatori scattati, utilizzazione GPU stimata, motivazioni e pulsante **Trasferisci alla valutazione completa**.

Input possibile via form, via JSON incollato o via file (`esempi/`).

## Fase 2 — Analisi completa e disaggregata

Schede di input (pannello laterale):

| Scheda | Contenuto principale |
|---|---|
| **A · Contesto** | orizzonte, tasso di sconto, valuta, giorni/ore di esercizio, finestra di punta, picco stagionale, SLA/RTO/RPO, scadenza go-live |
| **B · Task** | catalogo di task editabile: n/giorno, token input/cache/output, latenza massima, interattivo, tool call, TB scansionati, quota QA |
| **C · Modello & GPU** | parametri totali/attivi, quantizzazione, contesto, tensor parallelism, catalogo GPU (H100/H200/A100/L40S/…) con VRAM, banda, TFLOPS, TDP, prezzo — tutti modificabili |
| **D · Qualità** | accuratezza attesa per alternativa, costo di revisione umana, costo dell'errore, downtime |
| **E · Dati & DWH** | volume, crescita, DWH co-locato (nodi CPU) in locale vs lakehouse a consumo in cloud, vector DB |
| **F · Integrazioni** | selezione da catalogo (DB, ERP, CRM, e-commerce, pagamenti, IAM, SIEM, WAF, osservabilità…) con ore e latenza per chiamata |
| **G · Locale €** | costi server, rete, storage, energia (PUE), spazio, manutenzione, FTE, procurement, sostituzione hardware |
| **H · Cloud €** | prezzo per M token input/cache/output, sconti volume, piattaforma, egress, DWH per TB, FTE servizi gestiti |
| **I · Sviluppo** | ore per attività × tariffa, markup fornitore separato, guadagno da coding agent (simmetrico), formazione |
| **J · Vincoli & Fit** | vincoli eliminatori (sovranità, budget, scadenza, SLA), punteggi 1–5 su 6 assi qualitativi per alternativa |

Per le schede B–I si registrano **fonte, data e confidenza** (alta/media/bassa) dei valori.

## Modello di calcolo

Sintesi delle regole implementate (dettaglio e formule in `guida.html`):

- **Vincoli eliminatori prima dell'economia**: un'alternativa esclusa da sovranità dati, budget, scadenza di procurement o SLA non irraggiungibile non viene confrontata sul TCO.
- **Tempo per task a parità di task**: `prefill` (token input non in cache / throughput di prefill) + `decode` (token output / token/s per stream) + `tool call` (n. chiamate × latenza media delle integrazioni selezionate). Identico modello per le due alternative; cambiano i parametri di throughput.
- **Numero di GPU** = max(vincolo di memoria: pesi quantizzati + KV cache; vincolo di throughput nell'ora di punta × moltiplicatore stagionale; vincolo di disponibilità: ridondanza per SLA). Il piano modello e il piano dati (DWH, CPU, IO) sono dimensionati separatamente; in locale il DWH co-locato compete con l'inferenza.
- **Costo cloud** = token/anno per tipo × prezzo per milione, meno sconti, più piattaforma, egress, DWH a consumo, servizi gestiti.
- **Costi comuni a entrambe** (ma con parametri distinti): sviluppo (ore × tariffa, markup separato), integrazioni, revisione umana, errori, downtime, compliance, FTE.
- **Metriche finanziarie**: TCO in valore attuale netto (NPV) sull'orizzonte, costo annuo equivalente, costo del ritardo (benefit annuo × mesi di ritardo del go-live), valore netto = benefit attualizzato − TCO − costo del ritardo.
- **Break-even**: punto di utilizzazione GPU / volume di task in cui le due curve di costo cumulato si incrociano; indicatori di congruenza del carico con ciascuna alternativa.
- **Scenari**: prudente / base / favorevole, con variazione congiunta di volume, prezzi, qualità e tempi.
- **Sensitività**: 10 variabili variate di ±20 % (o range specifico), grafico tornado, top-3 driver e **soglia di inversione** (valore della variabile al quale la raccomandazione cambia).
- **Fit qualitativo e confidenza** su assi separati dal TCO: radar 1–5 su capacità operativa, indipendenza dalla rete, ciclo di aggiornamento, sicurezza/IR, flessibilità modelli, verificabilità; confidenza per sezione di input.

## Output: le 15 card di risultato

| # | Card | Visualizzazione |
|---|---|---|
| 1 | Caso d'uso e perimetro | KPI, tabella piano per piano (applicazione, modello, dati, integrazioni, operazioni) |
| 2 | Vincoli eliminatori | tabella esito per alternativa |
| 3 | Tempo per task (prefill + decode + tool call) | barre affiancate per task, verifica latenza massima |
| 4 | Dimensionamento locale | GPU per vincolo (memoria/throughput/disponibilità), DWH, storage, barre capacità vs domanda |
| 5 | Dimensionamento cloud | token/anno, costo per componente |
| 6 | Integrazioni, sicurezza e compliance | ore e costi per integrazione |
| 7 | Sviluppo e gestione | ore × tariffa, markup, effetto coding agent |
| 8 | TCO NPV, costo annuo equivalente, ritardo, valore netto | barre impilate per voce, KPI |
| 9 | Scenari prudente / base / favorevole | barre raggruppate |
| 10 | Congruenza dell'utilizzo e break-even | curve cumulate, utilizzazione GPU, soglia di volume |
| 11 | Sensitività e soglie di inversione | tornado, top-3 driver, valori di flip |
| 12 | Fit qualitativo e confidenza | radar a 6 assi, barre di confidenza per sezione |
| 13 | Limiti e dati da raccogliere | elenco ordinato per impatto |
| 13b | Tabella degli input | dato/ipotesi, fonte, confidenza per ogni variabile |
| 14 | Raccomandazione condizionata | "A conviene se…; B conviene se…", oppure "non determinabile" con motivazione |

Ogni elemento grafico ha un tooltip con valore, quota e descrizione; le voci non attive vengono attenuate al passaggio del mouse.

## Import / export JSON

- **Stato completo**: `Esporta JSON` / `Importa JSON` nella barra superiore salvano e ripristinano input, task, integrazioni e metadati (fonte/data/confidenza).
- **Pre-valutazione**: oggetto `{ "quick": { … } }` con le 15 chiavi elencate sopra; i file possono essere parziali (le chiavi assenti restano ai valori predefiniti). Schema ed esempi in `esempi/README.md`.

## Sistema visivo

- Palette neutra chiara; il colore codifica solo l'alternativa: **antracite = Locale**, **blu = Cloud + API**. Palette sequenziali (grigi / blu) per le barre impilate; beige sfumato per i livelli di confidenza.
- Nessuna icona o emoji; gerarchia affidata a tipografia, spaziatura e numeri tabellari.
- Card, KPI e pulsanti con spessore laterale in grigio metallizzato; righe di tabella in rilievo inverso.
- Layout responsivo (pannello input laterale fisso su desktop, impilato sotto i 1100 px); stili di stampa dedicati.

## Test

Harness minimale in `test/` basato su [jsdom](https://github.com/jsdom/jsdom):

```bash
cd test && npm install jsdom && cd ..
node test/dom2.js    # attesi: "cards: 15 | err card: false"
node test/quick.js   # attesi: "errors: []" sui tre esempi JSON
```

## Limiti noti

- Il simulatore è uno strumento di **stima**, non di preventivazione: i valori predefiniti vanno sostituiti con quotazioni reali e la confidenza va dichiarata.
- Il modello di throughput GPU è analitico (banda di memoria, TFLOPS, quantizzazione, tensor parallelism), non misurato su benchmark specifici del modello scelto.
- Non sono modellati: soluzioni ibride, fine-tuning/training, GPU in noleggio cloud come terza via, effetti fiscali (ammortamenti, IVA), variazioni di cambio.
- La pre-valutazione rapida è euristica; il suo esito può differire dall'analisi completa, che ha la precedenza.
- Lo stato non è salvato automaticamente nel browser: usare l'export JSON.

## Licenza

Da definire dal proprietario del repository.
