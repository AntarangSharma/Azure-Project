# Spotify Lakehouse — Azure + Databricks

A production-style data lakehouse built on **Azure** and **Databricks**, ingesting Spotify data through a medallion architecture (Silver → Gold) with Delta Live Tables, Unity Catalog, and automated orchestration via Azure Data Factory.

---

## Architecture

```
Spotify API / Raw Files
        │
        ▼
  Azure Data Lake Storage Gen2
        │
        ├── Silver Layer  ←  DLT pipeline (cleaning, dedup, typing)
        │
        └── Gold Layer    ←  DLT pipeline (aggregations, business metrics)
                │
                ▼
        Unity Catalog (spotify_cata)
                │
                ▼
        Databricks SQL / BI Tools
```

Orchestration: **Azure Data Factory** triggers Databricks jobs on a schedule.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Cloud Platform | Microsoft Azure |
| Compute | Databricks (Apache Spark / PySpark) |
| Storage | Azure Data Lake Storage Gen2 |
| Table Format | Delta Lake |
| Pipelines | Delta Live Tables (DLT) |
| Catalog | Unity Catalog (`spotify_cata`) |
| Orchestration | Azure Data Factory |
| IaC / Deployment | Databricks Asset Bundles |
| Language | Python (PySpark) |

---

## Project Structure

```
spotify_dab/                  # Databricks Asset Bundle root
├── databricks.yml            # Bundle config (targets: dev / prod)
├── resources/
│   ├── spotify_dab_etl.pipeline.yml   # DLT pipeline definition
│   └── sample_job.job.yml             # Databricks Job definition
└── src/
    └── dlt/
        ├── silver_pipeline.py         # Silver-layer DLT notebook
        └── gold_pipeline.py           # Gold-layer DLT notebook
```

---

## Prerequisites

- **Databricks CLI** ≥ 0.200 with a configured profile (`~/.databrickscfg`)
- **Azure subscription** with a Databricks workspace and ADLS Gen2 account
- **Unity Catalog** metastore attached to the workspace (`spotify_cata` catalog provisioned)
- Python 3.10+

---

## Deployment

```bash
# Validate the bundle locally
databricks bundle validate

# Deploy to dev target
databricks bundle deploy --target dev

# Deploy to production
databricks bundle deploy --target prod

# Run the ETL job manually
databricks bundle run spotify_dab_etl --target dev
```

---

## Resources

| Resource | Type | Description |
|---|---|---|
| `spotify_dab_etl` | DLT Pipeline | Silver → Gold ETL with Delta Live Tables |
| `sample_job` | Databricks Job | Wrapper job that triggers the DLT pipeline |

---

## Branches

| Branch | Purpose |
|---|---|
| `main` | Stable, production-ready code |
| `Antarang` | ADF collaboration branch (Azure Data Factory source) |
| `adf_publish` | ADF ARM template publish target (auto-managed by ADF) |

> **Note:** `Antarang` and `adf_publish` are managed by Azure Data Factory's git integration — do not manually rebase or delete them.

---

## Possible Improvements

- Add a Bronze layer for raw ingest (full medallion: Bronze → Silver → Gold)
- Parameterise the Spotify API credentials via Databricks Secrets
- Add DLT data quality expectations (`@dlt.expect_or_drop`)
- Wire ADF pipeline to CI/CD via GitHub Actions
