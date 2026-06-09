# Dialstream Communications — Data Product Lifecycle Guidelines

## Overview

All data products published to Dialstream's internal data marketplace must follow this lifecycle. These stages ensure quality, discoverability, and compliance before data products are consumed by other teams.

---

## Lifecycle Stages

```
Placeholder → Work in Progress → MVP Created → Published → Temporary Non-Compliant → Archive → Delete
```

### 1. Placeholder
- An empty shell record on the marketplace
- Increases awareness of work in progress
- Used to signal intent and gather organic demand from potential consumers
- **No data is served at this stage**

### 2. Work in Progress
- The data product is being actively developed
- Sources are being sent requirements
- Code and pipelines are being built
- Future consumers are helping shape outputs and schema
- **Not available for consumption**

### 3. MVP Created
- The data product exists on the marketplace
- Meets the MVP requirements defined by the data product owner
- Schema is stable enough for early feedback
- **Available for preview/testing by select consumers**

### 4. Published
- All minimal requirements have been met:
  - ✅ Schema documented
  - ✅ Owner assigned
  - ✅ SLA defined (freshness, availability)
  - ✅ Data quality checks passing
  - ✅ Access controls configured
  - ✅ Compliance checks validated
- Data product is discoverable on the marketplace
- Can be consumed by any authorized team
- **This is the standard operational state**

### 5. Temporary Non-Compliant
- A published data product has not adjusted to the latest compliance requirements
- A **grace period** is in effect to maintain business continuity
- Alerts are issued to downstream consumers
- **Usable by existing connections only** — no new consumers can subscribe
- Compliance must be restored or the product will be archived
- Grace period: **30 days maximum**

### 6. Archive
- The data product is being retired due to:
  - Limited use (fewer than 3 active consumers)
  - A newer, more relevant product has replaced it
  - Domain ownership has changed
- Retention rules for any new data apply
- Existing consumers are notified 14 days before archival
- **Read-only access for 30 days after archival**

### 7. Delete
- After the retention period, the archived data product is permanently deleted
- All metadata, access records, and pipeline configurations are removed
- **Irreversible**

---

## Data Product Requirements (for "Published" status)

| Requirement | Description |
|-------------|-------------|
| **Owner** | Named individual or team responsible for the product |
| **Schema** | Documented schema with field descriptions and data types |
| **SLA** | Defined freshness (e.g., "updated every 4 hours") and availability target |
| **Quality** | Automated data quality checks (completeness, uniqueness, validity) |
| **Access Control** | Fine-grained permissions — who can read, at what granularity |
| **Lineage** | Documented upstream sources and transformation logic |
| **Classification** | Data sensitivity level (public, internal, confidential, restricted) |
| **Compliance** | Validated against applicable regulations (GDPR, CCPA, industry-specific) |

---

## Governance Principles

1. **Domain ownership** — Each data product is owned by the domain that produces it, not a central team
2. **Federated standards** — Domains follow shared standards for schema, quality, and access, but manage their own pipelines
3. **Self-serve infrastructure** — Platform team provides tools; domain teams build and publish independently
4. **Discoverability** — Every published product must be findable through the marketplace catalog
5. **Interoperability** — Products must use agreed-upon formats and identifiers to enable cross-domain joins

---

## Using These Guidelines in Your Design

When completing **Step 4 (Data Mesh Domain Model)**, reference these lifecycle stages in your data product catalog. Each product you define should include:
- Its current lifecycle stage
- Criteria for advancing to the next stage
- Owner and SLA commitments
- Quality checks that must pass before publishing
