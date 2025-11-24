                 ┌──────────────────────────────┐
                 │  UST – Universal Tokens      │
                 │  (typed semantic units)      │
                 └───────────────┬──────────────┘
                                 │
                                 ▼
                 ┌────────────────────────────────┐
                 │  USR – Semantic Runtime Layer  │
                 │--------------------------------│
                 │ • Resolve references           │
                 │ • Enforce invariants           │
                 │ • Normalize semantic state     │
                 │ • Select execution profiles    │
                 │ • Route to proper engine path  │
                 └───────────────┬────────────────┘
                                 │
                                 ▼
                 ┌────────────────────────────────┐
                 │  USE – Semantic Executor       │
                 │--------------------------------│
                 │ • Translate USTs into          │
                 │   executable semantic ops      │
                 │ • Build operation graph        │
                 │ • Prepare CE contract          │
                 └───────────────┬────────────────┘
                                 │
                                 ▼
                 ┌────────────────────────────────┐
                 │  CE – Cognition Engine         │
                 │--------------------------------│
                 │ • Orient → Evaluate → Model    │
                 │ • Select → Justify → Finalize  │
                 │ • Produce reasoning unit       │
                 └───────────────┬────────────────┘
                                 │
                                 ▼
                 ┌────────────────────────────────┐
                 │  Domain Engine (SynCE, FinCE)  │
                 │--------------------------------│
                 │ • Domain-specific knowledge    │
                 │ • Domain-specific operations   │
                 │ • Final inference + validation │
                 └───────────────┬────────────────┘
                                 │
                                 ▼
                        ┌─────────────────┐
                        │    Output       │
                        └─────────────────┘
