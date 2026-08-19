# Data and Analytics Reference Framework

Hub for a boilerplate data and analytics program spanning Governance, Data,
Analytics, Data Science, and AI.

## Purpose

A series of repositories containing working reference architectures for an
enterprise-grade data and analytics program. It sits at the intersection of
technology and operations, demonstrating infrastructure deployed with the
governance and standards an enterprise would require.

The emphasis is deliberately broad rather than deep, and **program-centric
rather than performance-centric** — tagging, architecture decisions, naming,
and standardization over throughput and optimization.

## Audience

Owners of data programs looking for a boiler-plate starter or comparative reference for their own deployments.

## How to use

**Program owners** should start with the technology selections below and the
[Governance](https://github.com/crichton24/governance-reference)
standards, comparing both against their own internal requirements.

**Managers and team leads** should review the component repository for their
function, then the governance sections that apply to it.

## Components

| Status | Component | Contents |
|---|---|---|
| ✅ | [Overview](https://github.com/crichton24/data-and-analytics-program-reference) | This repository |
| ✅ | [Governance](https://github.com/crichton24/governance-reference) | Standards, norms, and principles |
| ✅ | [Data Management](https://github.com/crichton24/data-management-reference) | Architecture and code |
| 🚧 | Analytics | Dashboards for KPIs, governance adherence, and data performance |
| 🚧 | Data Science | Forecasting, segmentation, and prediction |
| 🚧 | AI | Co-pilot, co-work, and autonomous deployments |

✅ Complete · 🚧 In progress · ⬜ Not started

## Technology

Selected to balance low cost against best-in-breed capability and
transferability to other tools.

| Status | Layer | Selection | Rationale |
|---|---|---|---|
| ✅ | Data Warehouse | **Databricks** | Free edition. Swappable for Snowflake, Redshift, or another OLAP warehouse |
| ✅ | Data Storage | **AWS S3** | Inexpensive and broadly accessible |
| ✅ | Batch Orchestration | **Apache Airflow** | Open source and widely deployed. Strong lineage, error tracing, and monitoring |
| 🚧 | Stream Ingestion | **Apache Kafka** | Durable log with replay, decoupling producers from consumers |
| ✅ | Transformation | **dbt** | Best in class for governed, tested transformation |
| ⬜ | Analytics | *To be determined* | |
| ⬜ | Data Science | *To be determined* | |
| 🚧 | AI | **Claude** | Code authoring and review |

**Supporting**

| Status | Purpose | Selection | Rationale |
|---|---|---|---|
| ✅ | Version Control | **GitHub** | Also enforces standards at commit and on pull request |
| ✅ | Hosting | **Docker** | Local images for cost control. Not an enterprise-grade deployment |
| ✅ | Development | **VS Code** | Strong extension support for dbt, GitHub, and Docker |

![Application Architecture](images/Application_architecture.png)

*The data warehouse and visualization tools are interchangeable with other
enterprise-grade equivalents.*

## Disclaimers

**This is a reference, not a product.** Deployments will vary with
organizational constraints and requirements.

**These repositories will not deploy as-is.** They depend on local
credentials and security controls that are deliberately excluded.

**Security choices favor low-cost deployment.** Free tiers and local Docker
drive several decisions — personal access tokens rather than service
principals, long-lived credentials rather than assumed roles — that an
enterprise deployment would make differently. These trade-offs are documented
in the relevant architecture decision records rather than left implicit.

**No sensitive information or intellectual property is contained in this
code.**