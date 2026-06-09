# Designing an Enterprise Data Lakehouse for Dialstream Communications

## Background Story

You've just been hired as the Senior Data Architect at **Dialstream Communications**, a mid-size enterprise in the customer engagement and contact center space. Dialstream serves over 200 B2B clients and processes millions of customer interactions daily across voice, chat, and email channels.

Over the past three years, Dialstream's analytics platform grew organically — teams spun up data warehouses, relational databases, object storage buckets, and specialized AI services to solve immediate needs. There was no unified strategy. The result: a sprawling, expensive, and fragile data layer with duplicated pipelines, inconsistent metrics, and no clear ownership.

**The breaking point:** Dialstream's user base has doubled in the last two quarters. Product teams need real-time analytics dashboards. The AI team wants to expand call analytics models but can't get clean training data. Finance is asking why the cloud bill jumped 40% while query performance actually got worse. Leadership has also signed a partnership that will bring in a new SaaS data source (CRM platform) within 90 days — and there's nowhere clean to land it.

Your VP of Engineering has given you a clear mandate:

> "Design us a modern data lakehouse that simplifies what we have, costs less to run, performs better under load, and can actually scale with our growth. We need a plan we can start executing in 90 days."

---

## Your Mission

Assess the current state, design a governed enterprise data lakehouse, define domain ownership through a data mesh model, and deliver a phased migration roadmap that keeps the business running while you modernize underneath it.

---

## Repository Structure

```
cd0034-project-starter/
├── starter/                        # 📖 Read these first
│   ├── current-state-architecture.md
│   ├── service-inventory.md
│   └── data-product-lifecycle.md
├── solution_documents/             # 📝 Your deliverables (Jupyter notebooks)
│   ├── 1-current-state-assessment.ipynb
│   ├── 2-target-state-architecture.ipynb
│   ├── 3-medallion-layer-design.ipynb
│   ├── 4-data-product-catalog.ipynb
│   ├── 5-saas-integration-plan.ipynb
│   ├── 6-migration-roadmap.ipynb
│   └── 7-architecture-decision-record.ipynb
└── diagrams/                       # 🖼️ Your architecture diagrams
```

---

## Getting Started

1. **Review the starter materials** in the `starter/` folder — these describe Dialstream's current platform, every service in use, and the company's data product lifecycle standards.

2. **Complete the solution notebooks** in `solution_documents/` — they are numbered in the order you should work through them. Each notebook corresponds to a step covered in the classroom.

3. **Create diagrams** using a diagramming tool (draw.io, Lucidchart, or similar) and place them in the `diagrams/` folder. Reference them from your notebooks.

---

## Starter Materials

| Document | What It Contains |
|----------|-----------------|
| [Current-State Architecture](starter/current-state-architecture.md) | Dialstream's existing platform diagram, data flows, and known pain points |
| [Service Inventory](starter/service-inventory.md) | Every service — its role, data it handles, costs, and issues |
| [Data Product Lifecycle](starter/data-product-lifecycle.md) | Company standards for publishing and governing data products |

---

## Tools Required

- Architecture diagramming tool (draw.io, Lucidchart, or similar)
- Jupyter Notebook environment
- No cloud account or spend required

---

## Submission Format

Submit a single `.zip` file containing:

```
submission/
├── 1-current-state-assessment.ipynb
├── 2-target-state-architecture.ipynb
├── 3-medallion-layer-design.ipynb
├── 4-data-product-catalog.ipynb
├── 5-saas-integration-plan.ipynb
├── 6-migration-roadmap.ipynb
├── 7-architecture-decision-record.ipynb
└── diagrams/
    ├── current-state-annotated.png (or .svg, .pdf)
    ├── target-state-architecture.png (or .svg, .pdf)
    ├── medallion-layers.png (or .svg, .pdf)
    ├── domain-model.png (or .svg, .pdf)
    └── saas-integration.png (or .svg, .pdf)
```

**Requirements:**
- Solution documents: Jupyter Notebook (`.ipynb`)
- Diagrams: PNG, SVG, or PDF — must be legible at standard zoom
- All diagrams must also be embedded/referenced in their corresponding notebook
- File names must match the structure above
