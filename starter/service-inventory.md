# Dialstream Communications — Service Inventory

## Overview

This document details every service in Dialstream's current data platform, including its role, data it handles, upstream/downstream dependencies, and known issues.

---

## Storage Services

### Object Storage (Amazon S3)
| Attribute | Detail |
|-----------|--------|
| **Role** | Primary landing zone for raw data |
| **Data** | Voice transcripts (JSON), chat logs (CSV), email archives, billing exports, Glue job outputs |
| **Volume** | ~15 TB total, growing ~500 GB/month |
| **Structure** | Inconsistent — mix of date-partitioned, team-named, and flat folder structures |
| **Access** | Multiple teams write directly; no naming conventions enforced |
| **Issues** | Small file problem (millions of <1MB files), no lifecycle policies, no versioning on critical buckets |

### Data Warehouse (Amazon Redshift)
| Attribute | Detail |
|-----------|--------|
| **Role** | Primary reporting warehouse, catch-all data store, and direct query endpoint for consumers |
| **Data** | Mix of raw, semi-processed, and reporting tables — no intentional schema design. Schema has grown organically through accumulation of materialized views created by consumers to work around performance issues. |
| **Cluster** | dc2.large, 4 nodes, running 24/7 |
| **Load Pattern** | Direct writes from Lambda functions (real-time inserts), Glue jobs (batch), and manual COPY commands. No coordinated load schedule. |
| **Table Design** | No sort keys, no distribution keys, no compression. Tables created ad-hoc by different teams. No vacuum or analyze schedules. Hundreds of materialized views layered on top of each other — some referencing other materialized views. |
| **Consumers** | BI team (Power BI dashboards), Finance (scheduled reports), Executives (weekly KPIs), product teams (direct SQL queries), Lambda functions (direct inserts). Consumers query Redshift directly with unoptimized queries — full table scans, no filters on partition columns, complex joins across unnormalized tables. |
| **Issues** | Severe performance degradation from unoptimized tables and small-block inserts from Lambda. Materialized view sprawl — nobody knows which views are still needed or what depends on what. Consumer queries are unoptimized and compete with ETL writes for resources. No workload management. Concurrency limits during peak hours. Estimated 40% cost waste from always-on provisioning with no optimization. |

### Relational Database (PostgreSQL on RDS)
| Attribute | Detail |
|-----------|--------|
| **Role** | Operational analytics and cross-source joins |
| **Data** | Denormalized views combining CRM, billing, and interaction data |
| **Instance** | db.r5.2xlarge, Multi-AZ |
| **Users** | Product teams (direct SQL), internal tools, API backends |
| **Issues** | Not designed for analytical workloads — full table scans on large joins. Growing contention between operational and analytical queries. No read replicas for analytics. |

---

## Ingestion & Processing Services

### Kinesis Data Streams
| Attribute | Detail |
|-----------|--------|
| **Role** | Real-time ingestion of interaction events |
| **Sources** | Voice platform events, chat session events, email receipt notifications |
| **Consumers** | Lambda functions writing to S3, direct Bedrock triggers |
| **Throughput** | ~5,000 records/second peak |
| **Issues** | No dead-letter queue configured. Consumer failures cause data loss silently. |

### AWS Glue (ETL)
| Attribute | Detail |
|-----------|--------|
| **Role** | Batch ETL — transforms raw S3 data and loads into Redshift/PostgreSQL |
| **Jobs** | 23 active jobs, 8 deprecated but still scheduled |
| **Schedule** | Nightly (most), hourly (3 jobs), ad-hoc (5 jobs triggered manually) |
| **Issues** | No dependency management between jobs. Failures require manual intervention. Multiple jobs process the same source data redundantly. No data quality validation. |

### Lambda Functions (Custom Pipelines)
| Attribute | Detail |
|-----------|--------|
| **Role** | Ad-hoc data movement and transformation |
| **Count** | 47 functions related to data processing |
| **Issues** | Undocumented. Built by individual engineers, some of whom have left. No monitoring or alerting on most functions. Timeout issues on large payloads. |

---

## Query & Analytics Services

### Athena
| Attribute | Detail |
|-----------|--------|
| **Role** | Ad-hoc SQL queries on S3 data |
| **Users** | Only 2–3 engineers use it regularly. Most staff are unaware it exists or lack the SQL skills to use it. |
| **Issues** | Slow due to unoptimized file formats (CSV/JSON instead of Parquet), no partitioning, small files. Costs unpredictable — full scans on large datasets. Severely underutilized due to skills gap. |

### Power BI
| Attribute | Detail |
|-----------|--------|
| **Role** | Primary BI dashboards and visualizations |
| **Data Source** | Connected directly to Redshift |
| **Users** | ~60 active dashboard consumers across BI, Finance, and Executive teams |
| **Issues** | Competes with ETL writes for Redshift cluster resources. Dashboard refresh is slow during peak load. No semantic layer — each report author writes their own SQL. Inconsistent metric definitions across dashboards. |

### Smartsheets
| Attribute | Detail |
|-----------|--------|
| **Role** | Operational tracking, project reporting, manual data sharing |
| **Data Source** | Manual CSV exports from Redshift and PostgreSQL, some scheduled email reports |
| **Users** | Operations, project managers, client success teams |
| **Issues** | Data is stale (exported manually, often days old). Error-prone copy/paste workflows. No audit trail. Used as a workaround because teams can't access BI tools or write SQL. |

---

## AI / ML Services

### Amazon Bedrock
| Attribute | Detail |
|-----------|--------|
| **Role** | Call transcript summarization, sentiment analysis, agent coaching insights |
| **Models** | Claude (summarization), Titan (embeddings) |
| **Data Source** | Pulls transcripts directly from S3 raw bucket |
| **Issues** | No curated input layer — model quality depends on raw data consistency. No versioning of prompts or outputs. |

### Neptune (Graph Database)
| Attribute | Detail |
|-----------|--------|
| **Role** | POC for customer interaction graph — mapping relationships between customers, agents, topics, outcomes |
| **Status** | **Proof of concept only** — never moved to production. Limited data loaded, no active consumers. |
| **Data Source** | Small subset fed by manual scripts from PostgreSQL |
| **Issues** | Running and incurring cost despite being a stalled POC. No production use case validated. No team actively maintaining it. |

### OpenSearch
| Attribute | Detail |
|-----------|--------|
| **Role** | Application log aggregation and search (operational logging) |
| **Data Source** | Application logs shipped from Lambda, ECS, and other services |
| **Usage** | DevOps and engineering teams use it for troubleshooting and log search |
| **Issues** | Used only for operational logs — not leveraged for semantic search, embeddings, or vector retrieval despite the capability. Index bloat from no retention policy. Storage costs growing. |

---

## Data Catalog & Governance

### AWS Glue Data Catalog
| Attribute | Detail |
|-----------|--------|
| **Role** | Metadata catalog for Athena queries |
| **Coverage** | Partial — only covers ~30% of S3 data. Redshift and PostgreSQL not cataloged. |
| **Issues** | Stale entries. No ownership metadata. No lineage. Not used as a governance tool. |

### IAM / Access Control
| Attribute | Detail |
|-----------|--------|
| **Current Model** | Broad IAM roles per team. S3 bucket-level policies. Redshift user/group permissions. |
| **Issues** | No fine-grained column or row-level access. No centralized policy management. PII data accessible to teams that don't need it. |

---

## Cost Summary (Monthly Estimates)

| Service | Monthly Cost | Notes |
|---------|-------------|-------|
| S3 | $3,200 | Growing with no lifecycle management |
| Redshift | $12,800 | 24/7 cluster, unoptimized, no sort/dist keys |
| RDS PostgreSQL | $4,500 | Oversized for mixed workload |
| Glue | $2,100 | Redundant jobs inflating cost |
| Kinesis | $1,800 | Appropriately sized |
| Lambda | $900 | Scattered, hard to attribute |
| Athena | $600 | Low usage due to skills gap |
| Bedrock | $3,400 | Usage growing with AI expansion |
| Neptune | $2,200 | Underutilized relative to cost |
| OpenSearch | $3,100 | Index bloat driving storage costs |
| Power BI | $1,200 | Pro licenses for ~60 users |
| Smartsheets | $500 | Business plan licensing |
| **Total** | **~$36,300/mo** | |

---

## Upcoming Changes

- **New CRM SaaS integration** — Salesforce-class platform, REST API available, ~2M records initial load, ~50K records/day incremental
- **User growth** — Platform must support 2x current query volume within 6 months
- **Cost target** — Reduce total platform cost by 25% ($36K → $27K/mo target)
