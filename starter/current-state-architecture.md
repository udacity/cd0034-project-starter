# Dialstream Communications — Current-State Architecture

## Platform Overview

Dialstream's analytics platform was built incrementally over three years by different teams solving immediate business needs. There is no unified data strategy, no single source of truth, and no formal governance model.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                                      │
├─────────────────────────────────────────────────────────────────────────┤
│  Voice Calls  │  Chat Logs  │  Email  │  CRM (legacy)  │  Billing      │
└──────┬────────┴──────┬──────┴────┬────┴───────┬────────┴───────┬───────┘
       │               │           │            │                │
       ▼               ▼           ▼            ▼                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     INGESTION & PROCESSING                               │
├─────────────────────────────────────────────────────────────────────────┤
│  Kinesis Streams  │  Glue ETL Jobs  │  Custom Lambda Pipelines          │
│  (real-time)      │  (batch, daily) │  (ad-hoc, write directly to       │
│                   │                 │   Redshift and S3)                 │
└──────┬────────────┴────────┬────────┴──────────┬────────────────────────┘
       │                     │                   │
       ▼                     ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        STORAGE LAYER                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────┐  ┌──────────────────────┐  ┌────────────────────────┐│
│  │  Amazon S3   │  │  Redshift            │  │  PostgreSQL (RDS)      ││
│  │  (raw data,  │  │  (single cluster,    │  │  (operational          ││
│  │   transcripts,│  │   no schema design,  │  │   analytics,           ││
│  │   logs)      │  │   direct writes from │  │   cross-source joins,  ││
│  │              │  │   Lambda + Glue,     │  │   ad-hoc queries)      ││
│  │              │  │   no sort/dist keys) │  │                        ││
│  └──────────────┘  └──────────────────────┘  └────────────────────────┘│
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
       │                     │                   │
       ▼                     ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     QUERY & ANALYTICS                                    │
├─────────────────────────────────────────────────────────────────────────┤
│  Athena (available but rarely used — few staff know SQL/Athena)         │
│  Redshift SQL (direct queries from BI tools and consumers)              │
│  Power BI (dashboards, connected to Redshift)                           │
│  Smartsheets (manual exports and scheduled data pulls)                  │
│  OpenSearch (application log search and troubleshooting)                │
└─────────────────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     AI / ML SERVICES                                     │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────┐  ┌──────────────────────────────────────┐│
│  │  Bedrock                 │  │  Neptune                             ││
│  │  (call analytics,        │  │  (POC only, stalled,                 ││
│  │   summarization)         │  │   no production use case)            ││
│  └──────────────────────────┘  └──────────────────────────────────────┘│
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow Summary

1. **Ingestion**: Raw interaction data (voice transcripts, chat logs, emails) streams into S3 via Kinesis. Batch sources (billing, CRM exports) land in S3 via scheduled Glue jobs. Lambda functions write processed data directly into both S3 and Redshift with no coordination layer.

2. **Storage**: Data sits in three disconnected systems:
   - **S3** — raw files, no consistent folder structure or partitioning strategy
   - **Redshift** — single cluster receiving direct writes from Lambda functions and Glue jobs. No schema design, no sort keys, no distribution keys, no workload management. Tables created ad-hoc by different teams.
   - **PostgreSQL (RDS)** — operational analytics, cross-source joins for product teams

3. **Query & Analytics**: Consumers query Redshift directly with unoptimized SQL — full table scans, no filters, complex joins. The schema was never designed; it grew through accumulation of materialized views stacked on top of each other. Power BI connects to Redshift for dashboards. Some teams export data manually into Smartsheets. Athena is available but underutilized — only 2–3 engineers in the org know how to use it effectively. Most teams default to Redshift or manual exports.

4. **AI/ML**: Bedrock processes call transcripts for summarization and sentiment — pulls directly from raw S3 data with no curated input layer. Neptune was set up as a POC for graph-based interaction patterns but never moved to production (still running and costing money). OpenSearch is used for application log aggregation and troubleshooting — not for semantic search or embeddings despite having the capability.

---

## Known Pain Points

| Category | Issue |
|----------|-------|
| **Cost** | Redshift cluster runs 24/7 but has no optimization — no sort keys, no distribution strategy, no vacuum/analyze schedules. Duplicate data across S3, Redshift, and PostgreSQL. Multiple Glue jobs and Lambda functions write the same data to different targets. |
| **Performance** | Redshift is slow due to unoptimized table design (no sort/dist keys, wide tables with no compression). Direct writes from Lambda create small-block inserts that fragment storage. Consumers query Redshift directly with unoptimized SQL (full table scans, complex joins). Hundreds of materialized views layered on each other add refresh overhead and confusion. Athena queries on S3 are slow due to small files and no partitioning. PostgreSQL struggles with analytical workloads. |
| **Governance** | No data catalog or lineage tracking. No access control beyond IAM roles at the bucket/database level. No data quality checks in pipelines. Teams don't know what data exists or who owns it. |
| **Duplication** | Same customer data exists in 3+ places with different schemas and refresh cadences. Lambda functions and Glue jobs write overlapping data to Redshift without coordination. No single source of truth. |
| **BI & Reporting** | Power BI connects directly to Redshift, competing for cluster resources with ETL writes. Smartsheets relies on manual exports — data is stale and error-prone. No semantic layer or governed metrics. |
| **Scalability** | Adding new data sources requires building custom pipelines from scratch. No standardized ingestion pattern. The upcoming CRM integration has no clear landing zone. |
| **Skills Gap** | Athena is underutilized because most staff lack SQL/cloud query skills. Knowledge is concentrated in 2–3 engineers. Teams default to manual exports or direct Redshift queries. |
| **AI/ML** | Bedrock pulls raw unvalidated data directly from S3. Neptune is a stalled POC still incurring cost with no production value. OpenSearch is only used for app logs — semantic search and embedding capabilities are untapped. No curated ML-ready data layer exists. |

---

## What's Coming

- **New CRM SaaS platform** (partnership signed, data must be integrated within 90 days)
- **2x user growth** projected over the next year
- **Executive mandate** to reduce cloud spend by 25% while improving query performance
