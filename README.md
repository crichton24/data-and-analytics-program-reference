# Data and Analytics Reference Framework

Hub for components of a boilerplate data and analytics program including:  Governance, Data, Analytics, Data Science, and AI.

## Purpose
This is a series of repos containing working reference architectures for an enterprise grade Data & Analytics program.  It stis at the intersectino of technology and oeprations, serving as an example of deploying infrastructure with best practices and governance for enterprise deploymentes.  It's intent is to cover broadly with key focus on _program_ centric items over performance.  So it focuses on things like tagging, architecture decisions, and standardization.  

## Audience
The core audience is for owners of data programs to be refined for their own deployments.

## Content

- 


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


## Disclaimers
- This is a reference and boilerplate only.  Deployments will vary based on organizational constraints and requirements
- While the framework is a working deployment, these repos will fail to deploy as they rely on local & hidden security controls.
- Code contains no sensitive information or intellectual property

outline:
- intro and purpose
    - low cost, open source tech as examples of data program architecture (replace with your own, enterprise versions or different data warehouse / analytics tool (snowflake, redshift / Sigma, Tableau, PowerBI, etc))
    - demonstrates intersectino of technology and process for best practices (e.g. s3 for staging storage, airflow for ingestion, dbt for transformation)

- how to use:  
- content
    - links to each repo with purpose & state
    - Governance:  boiler plate guidelines for compliance and team standardization
    - data:  governed data life cycle with tags from ingestion (source --> s3 --> databricks) to formal data products (dbt:  raw->transform->formal).
    - analytics:  
        - 'hub' style deployment:  Corporate KPI dashboard to dept dashboards to functional reports.
        - governance reporting
        - D&A performance
    - data science:  forecasting, segmentation, and prediction
    - AI:  how used in this deployment:  enabler & fully autonomous


