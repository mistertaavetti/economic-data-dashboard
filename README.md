# Economic Data Dashboard

> Case study: an economic-data platform ingesting, refreshing, and visualizing Nigerian economic indicators and market data for a national newsroom audience.
> Built for **Nairametrics — Nigeria's largest business newsroom**.
> **Code is proprietary; this repo documents the build** — architecture, decisions, and outcomes.

---

## Context

| | |
|---|---|
| **Client** | Nairametrics (Infometrics economic data platform) |
| **Role** | Full-stack — data pipeline, backend API, dashboard frontend |
| **Timeline** | 2024 — present (ongoing platform; the newsroom's P0 data deliverable) |
| **Live API** | `api.infometrics.ng` |

## Problem

A business newsroom sits on large amounts of economic data — inflation, FX rates, monetary indicators, market series — scattered across spreadsheets and third-party sources. The goal was a single platform where editors and analysts could upload and refresh data, and where readers could explore interactive charts and dashboards driven by that data, at newsroom scale.

## Architecture

```
┌──────────────────────────┐       ┌──────────────────────────┐
│  Next.js monorepo (TS)   │  v1   │  FastAPI (Python)        │
│  Dashboards, admin       │ ◀───▶ │  SQLAlchemy + Alembic    │
│  console, charts         │  API  │  PostgreSQL, pandas      │
└──────────────────────────┘       │  CSV ingestion, JWT auth │
                                   └───────────┬──────────────┘
                                     Data refresh pipeline
                                     (CSV upload / provider)
```

### Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js (monorepo), TypeScript, interactive charting |
| Backend | FastAPI 0.115, Pydantic 2, SQLAlchemy 2.0, Alembic migrations |
| Data | PostgreSQL, pandas for ingestion and transformation, openpyxl for Excel interchange |
| Auth | JWT (python-jose / PyJWT), bcrypt |
| Infra | Uvicorn, pytest test suite |

### Data pipeline

- **CSV-first admin ingestion:** an admin console lets the newsroom's non-engineers upload and update economic series via CSV — the pipeline normalizes and loads them without a developer in the loop.
- **Migration to the v1 API contract:** the frontend was migrated onto the `api.infometrics.ng` v1 contract behind a **swappable data provider** — local fixture data in development, live API in production — so the frontend never hard-codes a backend.
- **Refresh discipline:** a data refresh pipeline keeps indicators current, with pandas doing the transformation work between source formats and the database schema.
- **Schema evolution:** Alembic migrations version every change to the data model — on a platform where historical economic series are the product, schema drift is not an option.
- **Interchange:** openpyxl handles Excel sources the way analysts actually produce them; the backend also runs a pytest suite around API and pipeline behavior.

### Reader-facing dashboard

Interactive charts and dashboards for economic indicators and market data, with cross-comparison views and a query interface for asking questions of the data. Planned analytical modules include a natural-language "ask engine" over the indicators and a commercial-banking data module, being ported onto the v1 stack.

### Admin console

The admin dashboard for uploading/updating economic data via CSV shipped as a planned milestone, giving the newsroom self-service control over the data that drives the public charts. The console continues to evolve — admin UI improvements and pipeline documentation are active workstreams, and lazy-loading of heavy series keeps the public charts responsive as the dataset grows.

## Key Technical Decisions

| Decision | Rationale | Trade-off |
|---|---|---|
| FastAPI over Django for this service | Lightweight, Pydantic-typed contracts, ideal for a data-shaped API | Fewer batteries than Django — admin/auth had to be built deliberately |
| Swappable provider layer in the frontend | Decouples development from API availability; migration to v1 became a provider swap | An extra abstraction layer to maintain |
| SQLAlchemy 2.0 + Alembic | Explicit, versioned schema evolution on a data-heavy platform | Migration discipline overhead |
| CSV as the ingestion format | The newsroom's data lives in spreadsheets; meet users where they are | CSV validation edge cases — schemas, encodings, date formats — need robust handling |
| Monorepo for the frontend | Shared components across dashboard, admin console, and chart modules | Build complexity grows with the monorepo |

## Outcomes

- The newsroom's **P0 data deliverable**: the economic data platform became the flagship data product, live behind `api.infometrics.ng`.
- Non-technical staff upload and refresh the data themselves via the admin console — the pipeline removed the engineer bottleneck from data updates.
- Frontend migration to the v1 API contract completed behind the provider abstraction without a big-bang rewrite.

## What I'd Do Differently

- **Validation reports in the admin console.** CSV ingestion failures surface as errors; a row-level validation report would let editors fix bad rows themselves instead of round-tripping through a developer.
- **Provider contract tests from the start.** The swappable provider pattern proved its worth during the v1 migration, but the local/live parity was maintained by hand until late — contract tests should have guarded it from day one.
- **Automated data-refresh monitoring.** Refresh failures are currently discovered operationally; alerting on stale series would close the loop.

## Evidence

- **Live platform:** available on request.
- **API:** https://api.infometrics.ng
- **Screenshots:** available on request.

---

*Case study by David Aloba. [code.davidaloba.com](https://code.davidaloba.com) · [LinkedIn](https://linkedin.com/in/mistertaavetti)*
