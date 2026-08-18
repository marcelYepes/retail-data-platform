# Retail Data Platform — Project Context

## Purpose

This is a public portfolio project designed to demonstrate practical Data Engineering skills.

The project should resemble a real-world engineering project while remaining small enough to understand, maintain, and explain clearly during a technical interview.

The objective is not to showcase as many technologies as possible. Every technology used should have a clear reason.

## Developer profile

The project owner currently works as a Data Engineer and has professional experience primarily with:

* Databricks
* Python
* SQL
* Unity Catalog
* Databricks Asset Bundles
* Lakebase
* Databricks Apps
* Git / GitHub
* CI/CD

The project should therefore emphasize these technologies rather than artificially adding tools that are not part of the developer's primary hands-on experience.

## Project

**Name:** Retail Data Platform

**Repository:** `retail-data-platform`

**Dataset:** Brazilian E-Commerce Public Dataset by Olist

**Platform:** Databricks Free Edition

## Business scenario

The project models a retail/e-commerce company that receives operational data about:

* customers
* orders
* order items
* products
* payments
* sellers
* product categories

The platform transforms these raw operational datasets into curated analytical datasets that can be consumed by a business-facing application.

## Dataset files

Initial scope:

* `olist_customers_dataset.csv`
* `olist_orders_dataset.csv`
* `olist_order_items_dataset.csv`
* `olist_products_dataset.csv`
* `olist_order_payments_dataset.csv`
* `olist_sellers_dataset.csv`
* `product_category_name_translation.csv`

Out of scope for the first version:

* reviews
* geolocation

These may be introduced later if they provide meaningful value.

## Architecture

High-level flow:

```text
Olist CSV files
      ↓
Raw ingestion
      ↓
Bronze
      ↓
Silver
      ↓
Gold
      ↓
Databricks App
```

The project follows Medallion Architecture.

### Bronze

Preserve source data with minimal transformations.

Expected entities:

* customers
* orders
* order_items
* products
* payments
* sellers
* product_categories

Example table:

`retail_data_platform.bronze.orders`

### Silver

Clean and validate data.

Typical responsibilities:

* correct data types
* normalize columns
* remove or handle duplicates
* handle null values
* validate identifiers
* standardize timestamps
* enforce meaningful data-quality rules

Example:

`retail_data_platform.silver.orders`

### Gold

Create business-oriented datasets.

Initial Gold datasets:

* `daily_sales`
* `product_performance`
* `customer_metrics`

Example:

`retail_data_platform.gold.daily_sales`

## Business questions

The Gold layer and Databricks App should initially answer three questions.

### Sales

How are sales and order volumes evolving over time?

### Products

Which product categories generate the most revenue and orders?

### Customers

Which customers contribute the most value and how frequently do they purchase?

## Unity Catalog structure

Target catalog:

`retail_data_platform`

Schemas:

```text
retail_data_platform.bronze
retail_data_platform.silver
retail_data_platform.gold
```

Additional schemas may be added only if there is a clear reason.

## Repository structure

Target structure:

```text
retail-data-platform/
├── src/
│   ├── ingestion/
│   ├── bronze/
│   ├── silver/
│   └── gold/
├── app/
├── resources/
├── tests/
├── docs/
│   ├── BRONZE_INGESTION_CONTRACT.md
│   ├── DATASET_INVENTORY.md
│   └── PROJECT_CONTEXT.md
├── data/
│   └── sample/
├── databricks.yml
├── AGENTS.md
├── README.md
└── .gitignore
```

The structure can evolve if implementation requirements justify it.

## Data storage

The complete Olist dataset must not be committed to GitHub.

`data/sample/` may contain small representative samples if useful for local tests.

Raw production-style data is stored within the Databricks environment using Unity Catalog-compatible storage.

The raw-file landing location is the managed Unity Catalog Volume:

`/Volumes/retail_data_platform/bronze/source_files/`

The managed Volume uses the catalog's Databricks default storage. This keeps file access governed by Unity Catalog and avoids introducing external cloud-storage configuration that the project does not require.

## Databricks Asset Bundles

The project should use Databricks Asset Bundles to define and deploy relevant Databricks resources.

The bundle should eventually include the jobs/workflows required to execute the pipeline.

Avoid adding resources before they are needed.

## Databricks App

The final project should include a small Databricks App consuming the Gold layer.

The App exists to demonstrate how engineered datasets can be exposed to end users.

It should remain secondary to the Data Engineering architecture; this is not primarily a frontend project.

## Quality

The project should demonstrate meaningful data-quality practices.

Examples:

* required IDs are not null
* order IDs are unique where expected
* monetary values are non-negative
* timestamps have valid relationships
* foreign-key relationships can be validated where useful

Avoid adding dozens of arbitrary checks merely to increase the apparent complexity of the project.

## Portfolio objective

The final GitHub repository should allow a recruiter or engineer to quickly understand:

1. the business problem
2. the architecture
3. how data moves through the platform
4. the engineering decisions
5. how the project is deployed
6. how quality is validated
7. what the final data product provides

The repository should eventually include:

* professional README
* architecture diagram
* setup/deployment instructions
* pipeline explanation
* data model
* screenshots of the Databricks App
* relevant technical decisions

## Current status

Completed:

* project concept selected
* Olist dataset selected
* high-level architecture defined
* Bronze/Silver/Gold scope defined
* business questions defined
* GitHub repository created and connected
* Olist source files downloaded and inspected locally
* Databricks Free Edition access verified
* local Databricks CLI authentication configured
* minimal Databricks Asset Bundle initialized and deployed
* Unity Catalog catalog and Bronze/Silver/Gold schemas created
* managed Unity Catalog Volume created for raw source files
* source inventory and initial data profile documented
* raw-file layout and Bronze ingestion contract defined
* seven source CSV files uploaded to the managed Volume and verified by byte size

Immediate next steps:

1. Define the implementation structure for the Bronze loader and its tests.
2. Implement the explicit Bronze schemas and idempotent full-refresh loader.
3. Add meaningful Bronze load checks and a Databricks Workflow resource.

The environment, deployment workflow, and raw source landing are working. Pipeline implementation has not started yet.
