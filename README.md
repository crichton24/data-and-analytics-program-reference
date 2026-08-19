# Data and Analytics Reference Framework

Hub for components of a boilerplate data and analytics program including:  Governance, Data, Analytics, Data Science, and AI.

## Purpose
This is a series of repos containing working reference architectures for an enterprise grade Data & Analytics program.  It stis at the intersectino of technology and oeprations, serving as an example of deploying infrastructure with best practices and governance for enterprise deploymentes.  It's intent is to cover broadly with key focus on _program_ centric items over performance.  So it focuses on things like tagging, architecture decisions, and standardization.  

## Audience
The core audience is for owners of data programs to be refined for their own deployments.

## How to Use
Program owners should start by reviewing the technology selections and [Governance](https://github.com/crichton24/data-analytics-governance-standards) documents and compare those to your own internal demands.

Managers and team leads of each of the below functions should breifly review the components of their respective repos then the governance sections.

## Content

- `DONE`[Overview](https://github.com/crichton24/data-and-analytics-program-reference)
- `DONE`[Governance](https://github.com/crichton24/data-analytics-governance-standards):  standards, norms, and principles
- `DONE`[Data Management](https://github.com/crichton24/data-management-reference):  architecture and cdoe
- `WIP`Analytics:  dashboards for KPIs, governance adherence, and data performance
- `WIP` Data Science:  time forecasts, segmentation, and prediction
- `WIP` AI:  co-pilot, co-work, and autonomous deployments


## Technology
Technology has been chosen to balance low-cost with best-in-breed and trasnferrability

- `DONE`**Data Warehouse:  Databricks**:  free version, can easily swap with Snowflake, Redshift, or other OLAP warehouses.
- `DONE`**Data Storage:  AWS S3**:  inexpensive and broadly accessible.
- `PARTIAL`**Data Orchestration / Ingestion**
    - `DONE`**Batch:  Apache Airflow** -- opensource and commonly deployed.  Excellent for lineage, error tracing, and monitoring.
    - `OPEN`**Stream:  Apache Kafka** --
- `DONE`**Data Transformation:  dbt** -- best in class for managed and governed data transformation.
- `OPEN`**Analytics:  <tbd>** --
- `OPEN`**Data Science: <tbd>** --
- `WIP`**AI:  Claude** -- excellent code writing co-pilot.

**Supporting Technology**
- `DONE`**Version Control:  Github** -- also excellent for ensforcing standards on commit
- `DONE`**Hosting:  Docker** -- local image purely for cost control.  While secrets are managed, this is not an enterprise grade solution.
- `DONE`**Coding:  VS Code** -- feature rich components for dbt, GitHub, and docker.

![Application Architecture](images/Application_architecture.png)
_Note:  Data Warehouse and Visualization tools can be exchanged for other enterprise-grade tools._

## Disclaimers
- This is a reference and boilerplate only.  Deployments will vary based on organizational constraints and requirements
- While the framework is a working deployment, these repos will fail to deploy as they rely on local & hidden security controls.
- Code contains no sensitive information or intellectual property


