# ShopFlow Analytics

> **Hands-on Snowflake data warehouse project · SnowPro Core aligned · Built on real data**

[![Live Site](https://img.shields.io/badge/Live%20Site-bh00t.github.io%2FShopFlow--Analytics-29B5E8?style=flat-square&logo=github)](https://bh00t.github.io/ShopFlow-Analytics/)
[![Dataset](https://img.shields.io/badge/Dataset-Olist%20Brazilian%20E--Commerce-orange?style=flat-square&logo=kaggle)](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
[![Snowflake](https://img.shields.io/badge/Platform-Snowflake-29B5E8?style=flat-square)](https://www.snowflake.com/)
[![SnowPro Core](https://img.shields.io/badge/Cert%20Goal-SnowPro%20Core-blue?style=flat-square)](https://learn.snowflake.com/en/certifications/snowpro-core/)

---

## What is ShopFlow?

ShopFlow is a production-grade Snowflake data warehouse built on the **Olist Brazilian E-Commerce dataset** — 9 CSV files, ~1.6 million rows, real data donated by Olist.

The project was designed to do two things at once:

1. **Master Snowflake by doing,** through 100% hands-on learning — no passive video watching
2. **Build a portfolio project** that demonstrates real data engineering patterns used in production

The architecture follows the **Medallion pattern** (Bronze → Silver → Gold), implemented in Snowflake as three schemas:

```
SHOPFLOW_RAW  →  SHOPFLOW_STAGED  →  SHOPFLOW_ANALYTICS
 (landing)         (cleaning)           (star schema)
```

---

## Live Project Website

**[ShopFlow-Analytics](https://bh00t.github.io/ShopFlow-Analytics/)**

The site is a single-file interactive learning hub that includes:

- Phase-by-phase project roadmap with step cards (Phases 00–06)
- SQL and Python reference with common error fixes for every step
- 15 random questions quiz each session
- Direct `.ipynb` notebook downloads for all 7 phases

---

## Dataset

[Olist Brazilian E-Commerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — free on Kaggle

| File | RAW Table | Rows |
|---|---|---|
| olist_orders_dataset.csv | RAW_ORDERS | ~99,441 |
| olist_order_items_dataset.csv | RAW_ORDER_ITEMS | ~112,650 |
| olist_customers_dataset.csv | RAW_CUSTOMERS | ~99,441 |
| olist_sellers_dataset.csv | RAW_SELLERS | ~3,095 |
| olist_products_dataset.csv | RAW_PRODUCTS | ~32,951 |
| olist_order_payments_dataset.csv | RAW_PAYMENTS | ~103,886 |
| olist_order_reviews_dataset.csv | RAW_REVIEWS | ~99,224 |
| olist_geolocation_dataset.csv | RAW_GEOLOCATION | ~1,000,163 |
| product_category_name_translation.csv | RAW_CATEGORY_TRANSLATION | ~71 |

---

## Architecture

```
Kaggle CSV files (local machine)
        │
        ▼  Snowsight UI file upload
@OLIST_STAGE  (Snowflake internal stage)
        │
        ▼  COPY INTO
SHOPFLOW_RAW  ── all VARCHAR · _loaded_at · _source_file
        │
        ▼  CTAS + TRY_CAST + ROW_NUMBER + QUALIFY
SHOPFLOW_STAGED  ── typed · deduplicated · NULLs handled
        │
        ▼  Star schema modelling
SHOPFLOW_ANALYTICS  ── FACT_ORDERS + 4 DIM tables
        │
        ▼  Streams + Tasks + Sharing
Automation pipeline · CDC · nightly aggregation · live data share
        │
        ▼  Visual Analytics (Snowpark Python + Plotly + Streamlit)
Interactive charts · session.sql().to_pandas() · st.plotly_chart()
```

### Snowflake objects created

| Object | Name | Notes |
|---|---|---|
| Virtual warehouse | SHOPFLOW_WH | X-Small, AUTO_SUSPEND=60 |
| Database | SHOPFLOW_DB | DATA_RETENTION=1 day |
| Role | SHOPFLOW_ROLE | project-scoped, least privilege |
| Schema | SHOPFLOW_RAW | landing zone |
| Schema | SHOPFLOW_STAGED | cleaning room |
| Schema | SHOPFLOW_ANALYTICS | star schema + automation objects |
| File format | OLIST_CSV_FORMAT | CSV, UTF8, SKIP_HEADER=1 |
| Internal stage | OLIST_STAGE | loading dock for CSV files |

---

## Phases

### ✅ Phase 00 — Snowflake Setup
*One-time · Snowsight UI*

Warehouse · database · role · 3 schemas · file format · internal stage. Every object created with the right privilege scope from scratch.

**Key concepts:** compute vs storage separation · object hierarchy · RBAC · internal vs external stages · named file formats · TIMESTAMP_NTZ

---

### ✅ Phase 01 — Load RAW Data
*~1 hour · Snowsight UI + SQL notebook*

Upload 9 CSVs via Snowsight UI → `COPY INTO` all 9 RAW tables → verify row counts → audit via `LOAD_HISTORY`.

**Key concepts:** `COPY INTO` syntax · `ON_ERROR` options · `PURGE` · idempotency · `ERROR_ON_COLUMN_COUNT_MISMATCH` · `INFORMATION_SCHEMA.LOAD_HISTORY`

---

### ✅ Phase 02 — Build STAGED Layer
*~2 hours · SQL notebook*

Cast all VARCHAR columns to proper types · fix column name typos · deduplicate geolocation · add English category names via `LEFT JOIN` · compute `delivery_delay_days`.

**Key concepts:** `TRY_CAST` vs `CAST` · `TRY_TO_TIMESTAMP` · `CTAS` pattern · `ROW_NUMBER + QUALIFY` · `COALESCE` · `TIMESTAMP_NTZ` · referential integrity checks

---

### ✅ Phase 03 — ANALYTICS Star Schema
*~3 hours · SQL notebook*

Build `FACT_ORDERS` (one row per order item) · `DIM_CUSTOMERS` · `DIM_PRODUCTS` · `DIM_SELLERS` · `DIM_DATE` (generated via `GENERATOR`) · write the 4 core business queries.

**Key concepts:** star schema design · fact vs dimension tables · `DIM_DATE` generation · `GENERATOR` function · clustering keys · micro-partition pruning · result cache

---

### ✅ Phase 04 — Performance + Security
*~2 hours · SQL notebook*

Clustering keys on `FACT_ORDERS` · result cache demonstration · RBAC with 3 project roles · dynamic data masking on PII · row access policy for seller-level isolation.

**Key concepts:** clustering + `SYSTEM$CLUSTERING_INFORMATION` · 3 result cache types · `CREATE MASKING POLICY` · `CREATE ROW ACCESS POLICY` · `GRANT` hierarchy

---

### ✅ Phase 05 — Streams, Tasks & Sharing
*~2 hours · SQL notebook*

Stream on `FACT_ORDERS` for CDC · nightly task with CRON schedule · `SYSTEM$STREAM_HAS_DATA` for conditional execution · secure data share with a reader account.

**Key concepts:** streams · append-only streams · task DAGs · `USING CRON` · serverless vs warehouse tasks · `CREATE SHARE` · zero-copy sharing

---

### ✅ Phase 06 — Visual Analytics
*~2 hours · Python notebook*

Plotly charts built directly on the star schema using Snowpark Python inside Snowflake Notebooks. Monthly GMV trend · review score vs delivery delay scatter · orders by Brazilian state · seller performance bubble chart.

**Key concepts:** `get_active_session()` · `session.sql().to_pandas()` · lazy Snowpark evaluation · `st.plotly_chart()` · `numpy.polyfit()` trendlines · Snowflake Notebook runtime limitations

**Gotchas covered:**

| Issue | Root cause | Fix |
|---|---|---|
| `fig.show()` crashes | `nbformat` not installed | Use `st.plotly_chart(fig, use_container_width=True)` |
| `trendline='lowess'` crashes | `statsmodels` not installed | Use `numpy.polyfit()` + `numpy.poly1d()` |
| `urllib` fetch fails | No outbound internet in Snowflake Notebooks | Load data via `COPY INTO`, chart from Snowflake |
| Column names not found | `.to_pandas()` returns UPPERCASE column names | Use `df['MONTH']` not `df['month']` |
| `session` not in scope | Auto-injection not guaranteed | Always import and call `get_active_session()` explicitly |

---

## Notebooks

Download and import directly into Snowflake Notebooks via **Snowsight → Projects → Notebooks → Import .ipynb**

| Phase | Notebook |
|---|---|
| Phase 00 | [SHOPFLOW_PHASE0_SETUP.ipynb](./SHOPFLOW_PHASE0_SETUP.ipynb) |
| Phase 01 | [SHOPFLOW_PHASE1_COPYINTO.ipynb](./SHOPFLOW_PHASE1_COPYINTO.ipynb) |
| Phase 02 | [SHOPFLOW_PHASE2_STAGED.ipynb](./SHOPFLOW_PHASE2_STAGED.ipynb) |
| Phase 03 | [SHOPFLOW_PHASE3_ANALYTICS.ipynb](./SHOPFLOW_PHASE3_ANALYTICS.ipynb) |
| Phase 04 | [SHOPFLOW_PHASE4_SECURITY.ipynb](./SHOPFLOW_PHASE4_SECURITY.ipynb) |
| Phase 05 | [SHOPFLOW_PHASE5_STREAMS_TASKS.ipynb](./SHOPFLOW_PHASE5_STREAMS_TASKS.ipynb) |
| Phase 06 | [SHOPFLOW_PHASE6_VISUAL_ANALYTICS.ipynb](./SHOPFLOW_PHASE6_VISUAL_ANALYTICS.ipynb) |

---

## Full Project Pipeline

```
Kaggle CSVs
    ↓  Phase 00 — Snowflake setup
@OLIST_STAGE
    ↓  Phase 01 — COPY INTO
SHOPFLOW_RAW (9 tables, all VARCHAR)
    ↓  Phase 02 — TRY_CAST, QUALIFY, COALESCE
SHOPFLOW_STAGED (9 tables, typed + clean)
    ↓  Phase 03 — Star schema
SHOPFLOW_ANALYTICS
    ├── FACT_ORDERS + 4 DIM tables
    ├── 4 answered business queries
    └── Clustering key on FACT_ORDERS
    ↓  Phase 04 — Security
    ├── 3 RBAC roles
    ├── Dynamic data masking
    └── Row access policy
    ↓  Phase 05 — Automation
    ├── ORDERS_STREAM (CDC)
    ├── NIGHTLY_SELLER_SUMMARY task
    ├── DAILY_SELLER_SUMMARY table
    └── BRAND_PARTNER_SHARE
    ↓  Phase 06 — Visual Analytics
    ├── session.sql().to_pandas() pattern
    ├── Monthly GMV trend (Plotly bar)
    ├── Review score vs delivery delay (scatter + numpy trendline)
    ├── Orders by Brazilian state (bar chart)
    └── st.plotly_chart() — Streamlit rendering in Notebooks
```

---

## SnowPro Core Coverage

This project covers the majority of SnowPro Core exam domains hands-on across 7 phases.

| Domain | Phases | Status |
|---|---|---|
| Snowflake Architecture | 00 | ✅ |
| Account Access & Security | 00, 04 | ✅ |
| Data Movement | 01 | ✅ |
| Data Transformation | 02 | ✅ |
| Data Modeling & Storage | 03 | ✅ |
| Performance & Optimization | 03, 04 | ✅ |
| Data Sharing & Collaboration | 05 | ✅ |
| Snowflake Automation | 05 | ✅ |
| Snowpark & Python | 06 | ✅ |

---

## Who is this for?

- Data engineers with SQL and cloud experience (AWS/GCP/Azure) looking to add Snowflake
- Anyone preparing for the **SnowPro Core** certification who wants hands-on practice over passive video watching
- Portfolio builders who want a real end-to-end DWH project on structured data

**Prerequisites:** SQL (intermediate) · basic Python · any cloud data warehouse experience

---

## Repo Structure

```
ShopFlow-Analytics/
├── index.html                              # Interactive project website (GitHub Pages)
├── SHOPFLOW_PHASE0_SETUP.ipynb             # Snowflake setup notebook
├── SHOPFLOW_PHASE1_COPYINTO.ipynb          # RAW data loading notebook
├── SHOPFLOW_PHASE2_STAGED.ipynb            # STAGED layer notebook
├── SHOPFLOW_PHASE3_ANALYTICS.ipynb         # Star schema notebook
├── SHOPFLOW_PHASE4_SECURITY.ipynb          # Performance + security notebook
├── SHOPFLOW_PHASE5_STREAMS_TASKS.ipynb     # Streams, tasks & sharing notebook
└── SHOPFLOW_PHASE6_VISUAL_ANALYTICS.ipynb  # Visual analytics (Snowpark + Plotly)
```

---

## Getting Started

1. Create a [free Snowflake trial account](https://signup.snowflake.com/) ($400 credit)
2. Download the [Olist dataset from Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
3. Open the [live site](https://bh00t.github.io/ShopFlow-Analytics/) and follow Phase 00
4. Download each notebook and import into Snowflake Notebooks via Snowsight
5. Run cells one at a time — every cell has an explanation of what it does and why

---

*ShopFlow Analytics · Built hands-on · SnowPro Core aligned · 7 phases · 530 quiz questions*
