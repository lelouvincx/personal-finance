# AGENTS.md

Personal Finance Analytics Project.

## Architecture

![architecture](https://media.secondbrain.lelouvincx.com/2026/02/56cc15cbc9fe24d3ef49463c76400836.png)

```
Google Drive (.mmbak weekly)
  → Dagster (orchestration, VPS-hosted)
    → Python ingestion asset (SQLite → MotherDuck)
      → dbt (staging → intermediate → marts)
        → Semantic context doc (auto-compiled from DBML + dbt YAML)
          → Vanna.ai text-to-SQL (schema-aware, trainable)
            → Next.js frontend (chat + dashboards)
```

### Key architectural decisions

1. **Dagster over Airflow/Prefect** — Asset-centric model maps 1:1 to data assets (raw tables, dbt models, semantic docs). `dagster-dbt` integration treats each dbt model as a software-defined asset. Lightweight enough for a personal VPS.
2. **MotherDuck over BigQuery/local DuckDB** — DuckDB query speed (~50-100ms) keeps AI chat interactive; BigQuery's 2-5s cold start kills UX. Cloud-hosted for accessibility from deployed frontend. Free tier covers this project.
3. **Docs-as-code semantic layer (DBML + dbt YAML) over Cube.js/MetricFlow** — Only one consumer (AI agent). Runtime semantic layer adds unnecessary query translation hop. DBML provides physical schema + relationships (JOINs); dbt YAML provides business semantics (metrics, descriptions). Both version-controlled, zero-runtime. Compiled into a single context doc for LLM consumption.
4. **Vanna.ai over WrenAI/raw prompting** — Library (not platform), composable, no UI/modeling format lock-in. Trainable on DDL + example queries — accuracy improves over time. WrenAI alternative if batteries-included is preferred (but heavier, AGPL, own modeling format).
5. **LLM for chart reasoning** — Structured output prompt: given data shape → generate ECharts config JSON. Few-shot examples sufficient for ~10 chart types. No chart recommendation library needed.
6. **Next.js + shadcn/ui + Tremor + ECharts** — Thin rendering shell. Tremor for dashboard KPIs/tables, ECharts for complex finance charts (Sankey, treemap), Vercel AI SDK for streaming chat, v0.dev to scaffold layouts. **Note:** Frontend stack is not finalized — will be decided after the data foundation (ingestion → transform → semantic → AI) is solid. Could be Next.js, SvelteKit, or anything else.

## Structure

```
personal-finance/                       # Monorepo root
├── orchestration/                      # Dagster project (Python)
│   ├── pyproject.toml                  # Dagster + dbt deps
│   ├── definitions.py                  # Assets: gdrive_download, raw_tables, semantic_doc
│   ├── resources.py                    # MotherDuck connection, GDrive client
│   └── transform/                      # dbt project (nested, for dagster-dbt)
│       ├── dbt_project.yml
│       ├── profiles.yml
│       ├── models/
│       │   ├── staging/                # stg_* — clean + rename + cast
│       │   ├── intermediate/           # int_* — joins, category tree
│       │   └── marts/                  # fct_*/dim_* — analytics-ready
│       └── seeds/
├── semantic/                           # Docs-as-code semantic layer
│   ├── money_manager.dbml              # Physical schema + Ref: + Note:
│   └── context.md                      # Auto-generated: unified ref for AI
├── app/                                # Frontend (TBD stack)
│   ├── package.json
│   └── src/
├── data/                               # Raw .mmbak backups (gitignored)
│   └── MMAuto*.mmbak
├── architecture.d2
├── AGENTS.md
└── README.md
```

## Key Data Model

- `INOUTCOME` — Main transactions table. `ZDATE` is millisecond Unix timestamp. `DO_TYPE`: 0=income, 1=expense, 3=transfer_out, 4=transfer_in. `ZMONEY` is VARCHAR, cast to DECIMAL. Filter `IS_DEL != 1`.
- `ASSETS` — Accounts (Cash, Momo, BIDV, CAKE, etc.). Linked via `uid`.
- `ZCATEGORY` — Hierarchical categories via `pUid`. `TYPE`: 0=income, 1=expense.
- All tables use `uid` (TEXT) as logical foreign key, not integer PKs.

## Conventions

- SQL: lowercase keywords, snake_case aliases, CTEs over subqueries, always filter soft deletes (`IS_DEL`/`isDel`/`C_IS_DEL`).
- dbt: staging models prefix `stg_`, intermediate `int_`, marts `fct_`/`dim_`. Metrics defined in YAML `meta:` block.
- DBML: keep `money_manager.dbml` in sync when schema changes. Include `Ref:` for all foreign keys and `Note:` for business context.
- Currency is VND (₫) primary. All amounts should be normalized to VND using `CURRENCY.RATE`.
