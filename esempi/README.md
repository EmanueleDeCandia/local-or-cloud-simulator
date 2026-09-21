# Esempi JSON per la pre-valutazione rapida (15 variabili)

Caricare da **⚡ Pre-valutazione rapida → Importa file JSON** (oppure incollare nel riquadro e "Applica JSON").
I file possono essere parziali: le variabili assenti restano ai valori predefiniti.

| File | Scenario sintetico | Esito atteso |
|---|---|---|
| `prevalutazione_pmi_ecommerce.json` | PMI e-commerce, 4.000 task/g, picco ×4, modello 32B, team base, 4 mesi, 600 k€ | Cloud + API (locale escluso: procurement > scadenza; utilizzazione GPU ~1%) |
| `prevalutazione_sanita_regolata.json` | Sanità, 20.000 documenti/g, dati con divieto di trasferimento, 70B, team forte, 5 anni | Locale (cloud escluso dal vincolo di sovranità) |
| `prevalutazione_industria_dwh.json` | Manifattura, reporting su DWH 40 TB, modello 120B, 600 task/g | Cloud + API (carico basso: economia e fit concordano) |

## Schema
```json
{ "quick": {
  "use_case": "Assistente clienti / e-commerce | Automazione documentale / back-office | Analisi dati e reporting su DWH | Coding / knowledge assistant interno | Agente con azioni su ERP/CRM/pagamenti",
  "active_users": 2000,
  "tasks_per_day": 3000,
  "peak_to_average_ratio": 3,
  "annual_growth_pct": 30,
  "model_parameters_b": 70,
  "quality_gap": "Nessuno | Piccolo | Medio | Grande",
  "data_sensitivity": "Bassa | Media | Alta | Critica",
  "data_tb": 5,
  "integrations_count": 4,
  "total_budget": 800000,
  "horizon_years": 3,
  "deadline_months": 6,
  "internal_it_capability": "Nessuna | Base | Buona | Forte",
  "annual_benefit": 500000
} }
```
Per le variabili a scelta basta la prima parola (es. `"Alta"`, `"Forte"`).
