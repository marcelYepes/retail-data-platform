# Bronze Ingestion Contract

## Purpose

This contract defines how the initial Olist CSV snapshot will land in Databricks and how it will be represented in Bronze. It is intentionally defined before pipeline implementation so storage, naming, schema, and reload behavior are explicit.

## Landing layout

The managed Unity Catalog Volume is:

`/Volumes/retail_data_platform/bronze/source_files/`

The seven original CSV files will be placed under one dataset directory:

```text
/Volumes/retail_data_platform/bronze/source_files/
└── olist/
    ├── olist_customers_dataset.csv
    ├── olist_order_items_dataset.csv
    ├── olist_order_payments_dataset.csv
    ├── olist_orders_dataset.csv
    ├── olist_products_dataset.csv
    ├── olist_sellers_dataset.csv
    └── product_category_name_translation.csv
```

Original filenames are preserved so each landed file remains traceable to the published dataset. Date partitions are not needed for version one because Olist is a fixed historical snapshot rather than a recurring source feed.

## Source-to-table mapping

| Source file | Bronze table |
| --- | --- |
| `olist_customers_dataset.csv` | `retail_data_platform.bronze.customers` |
| `olist_orders_dataset.csv` | `retail_data_platform.bronze.orders` |
| `olist_order_items_dataset.csv` | `retail_data_platform.bronze.order_items` |
| `olist_order_payments_dataset.csv` | `retail_data_platform.bronze.payments` |
| `olist_products_dataset.csv` | `retail_data_platform.bronze.products` |
| `olist_sellers_dataset.csv` | `retail_data_platform.bronze.sellers` |
| `product_category_name_translation.csv` | `retail_data_platform.bronze.product_categories` |

## CSV read contract

All files use the following read settings:

| Setting | Value |
| --- | --- |
| Header | Present |
| Delimiter | Comma |
| Quote character | Double quote |
| Encoding | UTF-8, with BOM-safe handling |
| Multiline records | Disabled unless source inspection proves otherwise |
| Schema inference | Disabled |

Each table will use an explicit schema. All source columns will initially be loaded as strings so Bronze preserves source representation and Silver owns type conversion and validation.

The source column names listed in [DATASET_INVENTORY.md](DATASET_INVENTORY.md) are the Bronze schema contract. The original product columns containing `lenght` remain unchanged in Bronze. The BOM in the category translation header is an encoding artifact and must not become part of the Bronze column name.

## Ingestion metadata

Each Bronze table will add three columns that do not exist in the source CSV:

| Column | Type | Purpose |
| --- | --- | --- |
| `_source_file` | `STRING` | Unity Catalog Volume path of the source file |
| `_ingested_at` | `TIMESTAMP` | UTC timestamp when the table load runs |
| `_ingestion_run_id` | `STRING` | Identifier shared by all tables loaded in one run |

Metadata columns use a leading underscore to distinguish operational metadata from source fields.

## Load behavior

Version one uses a full-refresh load for each Bronze table:

1. Validate that the expected source file exists.
2. Read it with the explicit CSV and column contract.
3. Add ingestion metadata.
4. Replace the target Bronze table atomically.
5. Validate the resulting row count and required key fields.

Full refresh is appropriate because the public dataset is a small, fixed snapshot. Incremental ingestion, Auto Loader, and streaming would add complexity without a recurring source feed.

Rerunning the same input must replace the existing table rather than append duplicate rows. The load therefore remains idempotent.

## Bronze quality boundary

Bronze checks are limited to failures that indicate an unusable load:

* expected file exists
* header matches the explicit source contract
* file can be parsed
* row count is greater than zero
* candidate key fields are not null
* loaded row count matches the parsed source row count

Business transformations belong in Silver, including:

* casting dates, timestamps, numeric values, and identifiers
* correcting `lenght` to `length` in column names
* interpreting source nulls
* validating timestamp relationships
* handling untranslated product categories
* validating monetary values and entity relationships

## Scope boundary

This document is a design contract. It does not create tables, upload files, or implement the ingestion workflow.
