# AGENTS.md

## Project

This repository contains a portfolio project called **Retail Data Platform**.

Before making changes, read:

* `README.md`
* `docs/PROJECT_CONTEXT.md`

## Goal

Build a production-style end-to-end retail data platform using Databricks Free Edition.

The project should demonstrate practical Data Engineering skills without introducing technologies only for complexity.

## Core stack

* Databricks
* Python
* SQL
* Unity Catalog
* Databricks Asset Bundles
* Databricks Workflows / Jobs
* Databricks Apps
* Git / GitHub

## Engineering principles

* Prefer simple, maintainable implementations.
* Use Python and SQL as the primary languages.
* Do not introduce PySpark, Kafka, Airflow, dbt, Terraform, or other technologies unless there is a clear project requirement.
* Use Medallion Architecture: Bronze → Silver → Gold.
* Keep raw data separate from source code.
* Do not commit the full Olist dataset to GitHub.
* Prefer Unity Catalog objects and modern Databricks patterns.
* Keep the project compatible with Databricks Free Edition.
* Use clear naming and small modular components.
* Add tests or data-quality checks where they provide meaningful value.

## Collaboration

Before implementing a substantial change:

1. Inspect the existing repository.
2. Explain the proposed approach briefly.
3. Reuse existing patterns where possible.
4. Avoid unnecessary refactors unrelated to the current task.
5. Update documentation when architecture or project decisions change.
