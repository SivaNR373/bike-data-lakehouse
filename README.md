# 🚲 Bike Data Lakehouse

A modern **Data Lakehouse** built with **Databricks, PySpark, Delta Lake, Unity Catalog, and GitHub**, implementing a complete **Bronze → Silver → Gold Medallion Architecture**.

The project ingests customer, product, sales, location, and product-category data from CRM and ERP source systems, performs data-quality transformations, builds an analytical dimensional model, and orchestrates the complete pipeline using **Databricks Jobs**.

---

## 📌 Project Overview

This project demonstrates an end-to-end Data Engineering workflow:

```text
CRM / ERP CSV Sources
        │
        ▼
   BRONZE LAYER
   Raw Delta Tables
        │
        ▼
   SILVER LAYER
Cleaned & Standardized Data
        │
        ▼
    GOLD LAYER
Dimensional Data Model
        │
        ▼
Databricks Job Orchestration
```

The pipeline is designed to demonstrate practical Data Engineering concepts including:

- Data ingestion
- Delta Lake
- Medallion Architecture
- Data cleansing
- Data quality validation
- Cross-system data integration
- Referential integrity
- Dimensional modeling
- Fact and dimension tables
- PySpark transformations
- Databricks notebook orchestration
- Databricks Jobs
- Unity Catalog
- Git/GitHub version control

---

# 🏗️ Architecture

```text
                         ┌─────────────────────┐
                         │    CRM CSV Sources  │
                         └──────────┬──────────┘
                                    │
                                    │
                         ┌──────────▼──────────┐
                         │    ERP CSV Sources  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   BRONZE LAYER      │
                         │   Raw Delta Tables  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                    ┌───────────────────────────────┐
                    │      SILVER ORCHESTRATION     │
                    └───────────────┬───────────────┘
                                    │
                ┌───────────────────┼───────────────────┐
                ▼                   ▼                   ▼
          CRM Transformations   ERP Transformations   Data Quality
                │                   │                   │
                └───────────────────┼───────────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    GOLD LAYER       │
                         │ Dimensional Model   │
                         └──────────┬──────────┘
                                    │
                   ┌────────────────┼────────────────┐
                   ▼                ▼                ▼
             dim_customers    dim_products      fact_sales
                                   
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Databricks Job    │
                         │  End-to-End Run     │
                         └─────────────────────┘
```

---

# 🥉 Bronze Layer

The Bronze layer stores the source data as Delta tables with minimal transformation.

### CRM

| Source | Bronze Table |
|---|---|
| `cust_info.csv` | `bronze.crm_cust_info` |
| `prd_info.csv` | `bronze.crm_prd_info` |
| `sales_details.csv` | `bronze.crm_sales_details` |

### ERP

| Source | Bronze Table |
|---|---|
| `CUST_AZ12.csv` | `bronze.erp_cust_az12` |
| `LOC_A101.csv` | `bronze.erp_loc_a101` |
| `PX_CAT_G1V2.csv` | `bronze.erp_px_cat_g1v2` |

### Bronze responsibilities

- Read CSV source files
- Infer source schema
- Preserve source-level information
- Write data as Delta tables
- Provide a reliable starting point for downstream transformations

---

# 🥈 Silver Layer

The Silver layer contains cleaned, standardized, and integrated data.

Six Silver transformation notebooks process the Bronze tables.

### CRM

```text
silver/crm/
├── silver_crm_cust_info
├── silver_crm_prd_info
└── silver_crm_sales_details
```

### ERP

```text
silver/erp/
├── silver_erp_cust_az12
├── silver_erp_loc_a101
└── silver_erp_px_cat_g1v2
```

### Key transformations

#### Customer data

- Removed records with missing business identifiers
- Trimmed names
- Resolved duplicate customer records
- Standardized gender values
- Standardized customer attributes
- Validated business-key uniqueness

#### Product data

- Trimmed product attributes
- Preserved historical product versions
- Standardized column names
- Validated product identifiers

Product keys were intentionally **not deduplicated**, because multiple records can represent different product versions over time.

#### Sales data

- Converted `yyyyMMdd` source dates to proper date types
- Handled invalid source dates safely
- Validated order/ship/due-date relationships
- Checked duplicate order-line business keys
- Validated customer referential integrity
- Validated product relationships across CRM/ERP key formats
- Preserved source financial values where the correct business interpretation was uncertain

#### ERP customer data

- Standardized gender values
- Converted invalid future birth dates to NULL
- Validated customer identifier uniqueness

#### ERP location data

- Standardized country names
- Consolidated values such as:

```text
US
USA
United States
```

into:

```text
United States
```

- Converted missing/blank countries to `Unknown`

#### ERP product categories

- Standardized column names
- Validated category identifiers
- Preserved the source category structure

---

# 🥇 Gold Layer

The Gold layer provides business-oriented analytical tables using a dimensional model.

## Star Schema

```text
                     ┌──────────────────┐
                     │  dim_customers   │
                     └────────┬─────────┘
                              │
                              │
┌──────────────────┐          │          ┌──────────────────┐
│   dim_products   │──────────┼──────────│    fact_sales    │
└──────────────────┘          │          └──────────────────┘
                              │
                              │
                       Sales Line Grain
```

### Gold tables

| Table | Purpose |
|---|---|
| `gold.dim_customers` | Customer master dimension |
| `gold.dim_products` | Product and category dimension |
| `gold.fact_sales` | Sales transaction fact |

---

## `dim_customers`

Combines customer information from CRM and ERP sources.

Includes:

- Customer ID
- Customer key
- First name
- Last name
- Marital status
- Gender
- Create date
- Birth date
- Country

CRM customer records are used as the primary customer spine, with ERP attributes integrated through cross-system identifier normalization.

---

## `dim_products`

Combines CRM product information with ERP product-category metadata.

Includes:

- Product ID
- Product key
- Product name
- Product cost
- Product line
- Start date
- End date
- Category
- Subcategory
- Maintenance requirement

Historical product versions are retained rather than deduplicating on product key.

---

## `fact_sales`

The sales fact table represents:

> **One row per `order_number + product_key` sales line.**

Includes:

- Order number
- Product key
- Customer ID
- Order date
- Ship date
- Due date
- Sales amount
- Quantity
- Unit price

The fact table retains business keys rather than joining directly to historical dimension versions, avoiding accidental row multiplication.

---

# 🔍 Data Quality & Validation

Data quality checks were performed throughout the pipeline rather than only at the end.

Examples include:

- NULL checks
- Duplicate checks
- Business-key validation
- Date validation
- Date relationship validation
- Referential integrity
- Cross-system identifier normalization
- Product-key reconciliation
- Numeric anomaly analysis
- Row-count validation
- Post-transformation sanity checks

### Final verified Gold row counts

| Gold Table | Rows |
|---|---:|
| `dim_customers` | 18,484 |
| `dim_products` | 397 |
| `fact_sales` | 60,398 |

### Referential integrity

The final Silver/Gold validation confirmed:

- Sales → Customer unmatched records: **0**
- Sales → Product unmatched records: **0**
- CRM Customer → ERP Customer unmatched records: **0**
- CRM Customer → ERP Location unmatched records: **0**

---

# 🔄 Pipeline Orchestration

The project uses Databricks notebook orchestration with:

```python
dbutils.notebook.run()
```

### Silver orchestration

The Silver parent notebook executes all six Silver transformations sequentially:

```text
silver_orchestration
│
├── silver_crm_cust_info
├── silver_crm_prd_info
├── silver_crm_sales_details
├── silver_erp_cust_az12
├── silver_erp_loc_a101
└── silver_erp_px_cat_g1v2
```

### Gold orchestration

```text
gold_orchestration
│
├── dim_customers
├── dim_products
└── fact_sales
```

---

# ⚙️ Databricks Job

The complete pipeline is orchestrated using a Databricks Job.

```text
┌──────────────────────┐
│   bronze_ingestion   │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ silver_orchestration │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  gold_orchestration  │
└──────────────────────┘
```

The end-to-end Job was successfully executed with all three tasks completing successfully.

---

# 🗂️ Repository Structure

```text
bike-data-lakehouse/
│
├── bronze/
│   └── bronze_ingestion.ipynb
│
├── silver/
│   ├── silver_orchestration.ipynb
│   │
│   ├── crm/
│   │   ├── silver_crm_cust_info.ipynb
│   │   ├── silver_crm_prd_info.ipynb
│   │   └── silver_crm_sales_details.ipynb
│   │
│   └── erp/
│       ├── silver_erp_cust_az12.ipynb
│       ├── silver_erp_loc_a101.ipynb
│       └── silver_erp_px_cat_g1v2.ipynb
│
└── gold/
    ├── gold_orchestration.ipynb
    ├── dim_customers.ipynb
    ├── dim_products.ipynb
    └── fact_sales.ipynb
```

---

# 🛠️ Technology Stack

| Technology | Purpose |
|---|---|
| **Databricks Free Edition** | Lakehouse development platform |
| **PySpark** | Data transformation |
| **Delta Lake** | Storage/table format |
| **Unity Catalog** | Data and schema organization |
| **Python** | Notebook development |
| **Databricks Serverless Compute** | Execution |
| **Databricks Jobs** | Pipeline orchestration |
| **GitHub** | Version control |
| **Draw.io** | Architecture and dimensional modeling |

---

# 🚀 Execution Flow

The pipeline can be executed through the Databricks Job:

```text
1. Bronze ingestion
        ↓
2. Silver transformations
        ↓
3. Gold dimensional model
```

The Job manages task dependencies so downstream layers execute only after the preceding layer succeeds.

---

# 🧠 Engineering Decisions

### 1. Preserve uncertain source values

Where the source contained unusual sales or price values, values were not arbitrarily corrected because the correct business interpretation could not be established from the available source data.

### 2. Preserve product history

Product keys can occur across multiple product versions. Therefore, product records were not deduplicated solely by product key.

### 3. Normalize cross-system identifiers

CRM and ERP systems use different identifier formats.

For example:

```text
CRM:
11000

ERP Customer:
NASAW00011000

ERP Location:
AW-00011000
```

The pipeline normalizes the numeric identifier suffix to establish the cross-system relationship.

### 4. Avoid fact-table multiplication

The product dimension contains historical versions. Joining the entire product dimension directly into the sales fact could multiply rows.

The fact therefore retains the business product key while the dimensional relationship is validated separately.

---

# 📈 Current Project Status

| Component | Status |
|---|---|
| Architecture | ✅ Complete |
| Unity Catalog setup | ✅ Complete |
| Bronze ingestion | ✅ Complete |
| Silver transformations | ✅ Complete |
| Gold dimensional model | ✅ Complete |
| Silver orchestration | ✅ Complete |
| Gold orchestration | ✅ Complete |
| Databricks Job | ✅ Complete |
| End-to-end pipeline run | ✅ Successful |
| GitHub version control | ✅ Complete |
| README documentation | 🔄 In progress |

---

# 🔮 Future Improvements

The current implementation focuses on a clean **V1 end-to-end lakehouse pipeline**.

Potential production hardening includes:

- Parameterized ingestion framework
- Reusable transformation functions
- Incremental ingestion
- Metadata-driven pipelines
- Schema evolution handling
- Audit columns and batch/run IDs
- Automated data-quality gates
- Retry and failure handling
- Monitoring and alerting
- Automated testing
- CI/CD
- Data lineage
- Performance optimization
- Pipeline scheduling

---

# 👨‍💻 Author

**Adaka Siva Nagaraju**

Data Engineering | Python | SQL | PySpark | Databricks | Delta Lake | Data Warehousing

GitHub: [@SivaNR373](https://github.com/SivaNR373)

---

## ⭐ Project

If you find this project useful, feel free to explore the notebooks and architecture.
