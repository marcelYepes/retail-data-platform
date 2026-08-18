# Olist Dataset Inventory

## Purpose

This document records the local source-data inspection completed before pipeline implementation. It defines the initial file-to-entity mapping and highlights source characteristics that must be handled deliberately in the Medallion layers.

The full dataset is stored locally under `data/raw/` and is excluded from Git.

## Source inventory

Row counts exclude the CSV header.

| Source file | Rows | Size | Bronze table | Candidate key |
| --- | ---: | ---: | --- | --- |
| `olist_customers_dataset.csv` | 99,441 | 8.62 MB | `retail_data_platform.bronze.customers` | `customer_id` |
| `olist_orders_dataset.csv` | 99,441 | 16.84 MB | `retail_data_platform.bronze.orders` | `order_id` |
| `olist_order_items_dataset.csv` | 112,650 | 14.72 MB | `retail_data_platform.bronze.order_items` | `order_id`, `order_item_id` |
| `olist_order_payments_dataset.csv` | 103,886 | 5.51 MB | `retail_data_platform.bronze.payments` | `order_id`, `payment_sequential` |
| `olist_products_dataset.csv` | 32,951 | 2.27 MB | `retail_data_platform.bronze.products` | `product_id` |
| `olist_sellers_dataset.csv` | 3,095 | 0.17 MB | `retail_data_platform.bronze.sellers` | `seller_id` |
| `product_category_name_translation.csv` | 71 | less than 0.01 MB | `retail_data_platform.bronze.product_categories` | `product_category_name` |

All candidate keys were unique and non-null during the initial inspection.

## Source columns

### Customers

`customer_id`, `customer_unique_id`, `customer_zip_code_prefix`, `customer_city`, `customer_state`

### Orders

`order_id`, `customer_id`, `order_status`, `order_purchase_timestamp`, `order_approved_at`, `order_delivered_carrier_date`, `order_delivered_customer_date`, `order_estimated_delivery_date`

### Order items

`order_id`, `order_item_id`, `product_id`, `seller_id`, `shipping_limit_date`, `price`, `freight_value`

### Payments

`order_id`, `payment_sequential`, `payment_type`, `payment_installments`, `payment_value`

### Products

`product_id`, `product_category_name`, `product_name_lenght`, `product_description_lenght`, `product_photos_qty`, `product_weight_g`, `product_length_cm`, `product_height_cm`, `product_width_cm`

### Sellers

`seller_id`, `seller_zip_code_prefix`, `seller_city`, `seller_state`

### Product categories

`product_category_name`, `product_category_name_english`

## Expected relationships

| Child field | Parent field | Initial inspection |
| --- | --- | --- |
| `orders.customer_id` | `customers.customer_id` | No orphan rows |
| `order_items.order_id` | `orders.order_id` | No orphan rows |
| `order_items.product_id` | `products.product_id` | No orphan rows |
| `order_items.seller_id` | `sellers.seller_id` | No orphan rows |
| `payments.order_id` | `orders.order_id` | No orphan rows |
| `products.product_category_name` | `product_categories.product_category_name` | 13 rows across 2 non-null category values have no translation |

## Observed nulls

Only columns with observed nulls are listed.

| Entity | Column | Null rows |
| --- | --- | ---: |
| Orders | `order_approved_at` | 160 |
| Orders | `order_delivered_carrier_date` | 1,783 |
| Orders | `order_delivered_customer_date` | 2,965 |
| Products | `product_category_name` | 610 |
| Products | `product_name_lenght` | 610 |
| Products | `product_description_lenght` | 610 |
| Products | `product_photos_qty` | 610 |
| Products | `product_weight_g` | 2 |
| Products | `product_length_cm` | 2 |
| Products | `product_height_cm` | 2 |
| Products | `product_width_cm` | 2 |

These nulls require business-aware validation in Silver. Delivery timestamps, for example, may legitimately be absent for orders that were canceled or not yet delivered.

## Known source quirks

* `product_category_name_translation.csv` begins with a UTF-8 byte order mark (BOM). CSV reading must handle it without retaining the BOM in the first column name.
* The products source spells `length` as `lenght` in `product_name_lenght` and `product_description_lenght`.
* Thirteen product rows use two non-null category values that are absent from the translation file.

Bronze should preserve the source data with minimal changes. Source naming corrections, type enforcement, null handling, and category-translation rules belong in Silver.

## Scope boundary

This inventory is a discovery artifact only. No source files have been uploaded to Databricks, no Bronze tables have been created, and no ingestion pipeline has been implemented yet.
