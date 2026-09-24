# 🚲 Bike Data Lakehouse

An end-to-end **Data Lakehouse project** built using Databricks and PySpark, implementing a **Bronze → Silver → Gold Medallion Architecture**.

The project ingests CRM and ERP source data, performs data cleansing and cross-system integration, builds a dimensional model, and orchestrates the complete pipeline using Databricks Jobs.

![Lakehouse Architecture](docs/architecture/lakehouse_architecture.png)

---

## 🏗️ Architecture

```text
CRM / ERP CSV Sources
        ↓
Bronze Layer
Raw Delta Tables
        ↓
Silver Layer
Cleaned & Integrated Data
        ↓
Gold Layer
Dimensional Model
        ↓
Databricks Job
End-to-End Orchestration
```

### Pipeline

```text
bronze_ingestion
       ↓
silver_orchestration
       ↓
gold_orchestration
```

---

## 🥉 Bronze Layer

Raw source data is ingested from CRM and ERP CSV files into Delta tables.

### CRM

- `crm_cust_info`
- `crm_prd_info`
- `crm_sales_details`

### ERP

- `erp_cust_az12`
- `erp_loc_a101`
- `erp_px_cat_g1v2`

---

## 🥈 Silver Layer

The Silver layer cleans, standardizes, validates, and integrates the source data.

### CRM

- Customer cleansing and deduplication
- Product standardization
- Sales date conversion and validation
- Business-key validation
- Customer and product referential-integrity checks

### ERP

- Customer gender standardization
- Invalid/future birth-date handling
- Country normalization
- Product-category standardization

Cross-system identifiers between CRM and ERP were normalized to establish relationships between datasets.

---

## 🥇 Gold Layer

The Gold layer provides analytics-ready dimensional tables.

### `dim_customers`

Customer master dimension combining CRM and ERP attributes.

### `dim_products`

Product dimension enriched with ERP category and maintenance information.

### `fact_sales`

Sales transaction fact table at the grain of:

```text
order_number + product_key
```

### Final Gold Tables

| Table | Rows |
|---|---:|
| `dim_customers` | 18,484 |
| `dim_products` | 397 |
| `fact_sales` | 60,398 |

---

## 🔄 Orchestration

Silver transformations are executed through:

```text
silver_orchestration
├── CRM Customer
├── CRM Product
├── CRM Sales
├── ERP Customer
├── ERP Location
└── ERP Product Category
```

Gold transformations are executed through:

```text
gold_orchestration
├── dim_customers
├── dim_products
└── fact_sales
```

The complete pipeline is orchestrated using **Databricks Jobs**:

```text
Bronze
  ↓
Silver
  ↓
Gold
```

The end-to-end Job has been successfully executed.

---

## 🔍 Data Quality

The project includes validation for:

- NULL values
- Duplicate records
- Business-key uniqueness
- Date validity
- Date relationships
- Referential integrity
- Cross-system identifier matching
- Product-key reconciliation
- Post-transformation row counts

---

## 🛠️ Technology Stack

- **Databricks**
- **PySpark**
- **Delta Lake**
- **Unity Catalog**
- **Python**
- **Databricks Jobs**
- **Git / GitHub**
- **Draw.io**

---

## 📁 Repository Structure

```text
bike-data-lakehouse/
│
├── README.md
│
├── docs/
│   └── architecture/
│       └── lakehouse_architecture.png
│
├── bronze/
│   └── bronze_ingestion.ipynb
│
├── silver/
│   ├── silver_orchestration.ipynb
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

## 📚 Credits

This project was developed as a hands-on implementation and learning project based on the **Data Engineering concepts and project guidance from Data with Baraa**.

Credit and appreciation to **Data with Baraa** for the educational material and project inspiration.

---

## 👨‍💻 Author

**Adaka Sivanagaraju**

Data Engineering | SQL | Python | PySpark | Databricks | Delta Lake

- GitHub: [SivaNR373](https://github.com/SivaNR373)
- LinkedIn: [sivan373](https://www.linkedin.com/in/sivan373/)
- Email: `sivanagaraju373@gmail.com`

For questions, feedback, or collaboration, feel free to reach out.

---

⭐ If you find this project useful, feel free to explore the notebooks and architecture.
