# Azure Databricks Data Engineering – ADLS Gen2 to Medallion Architecture

## 📌 Project Overview

This project demonstrates an **Azure Databricks data engineering workflow using Azure Data Lake Storage Gen2 (ADLS Gen2)** and a **Medallion Architecture**.

The workflow starts with raw **AdventureWorks CSV datasets** stored in the **Bronze** layer of ADLS Gen2. Azure Databricks and PySpark are then used to read, inspect, transform, combine, and persist the data into the **Silver** layer. Finally, curated dimensional and fact datasets are organized in the **Gold** layer and exposed through **Databricks Catalog**.

The project demonstrates a practical flow:

```text
ADLS Gen2
   │
   ▼
Bronze / Raw
   │
   │  Azure Databricks + PySpark
   │  Read • Inspect • Transform • Combine
   ▼
Silver / Processed
   │
   ▼
Gold / Curated
   │
   ▼
Databricks Catalog
   │
   ▼
Analytics / BI
```

---

## 🎯 Project Objectives

- Connect Azure Databricks to **ADLS Gen2**.
- Read raw AdventureWorks CSV files from the Bronze layer.
- Inspect source datasets using PySpark DataFrames.
- Apply basic data transformation and standardization.
- Combine yearly sales datasets into one consolidated Sales DataFrame.
- Persist processed datasets into the Silver layer.
- Organize curated data into dimensions and facts in the Gold layer.
- Register and inspect Bronze, Silver, and Gold datasets through Databricks Catalog.
- Demonstrate a practical **Bronze → Silver → Gold** data engineering workflow.

---

## 🏗️ Architecture

```text
                         ┌──────────────────────┐
                         │     ADLS Gen2        │
                         │   Storage Account    │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │   BRONZE / RAW       │
                         │                      │
                         │ Calendar             │
                         │ Customers            │
                         │ Products             │
                         │ Product Categories   │
                         │ Subcategories        │
                         │ Returns              │
                         │ Sales 2015–2017      │
                         │ Territories           │
                         └──────────┬───────────┘
                                    │
                                    │ Read with Spark
                                    ▼
                    ┌──────────────────────────────┐
                    │     AZURE DATABRICKS         │
                    │                              │
                    │  PySpark DataFrames          │
                    │  Data Inspection             │
                    │  Transformations             │
                    │  concat_ws / col             │
                    │  unionByName                 │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                         ┌──────────────────────┐
                         │   SILVER / PROCESSED │
                         │                      │
                         │ Calendar             │
                         │ Customers            │
                         │ Products             │
                         │ Product Subcategories│
                         │ Returns              │
                         │ Sales                │
                         │ Territories           │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │    GOLD / CURATED    │
                         │                      │
                         │ dim_customer         │
                         │ dim_date             │
                         │ dim_product          │
                         │ dim_territory        │
                         │ fact_returns         │
                         │ fact_sales           │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │  DATABRICKS CATALOG  │
                         │ Bronze / Silver / Gold│
                         └──────────────────────┘
```

---

## 🥉 Bronze Layer – Raw Data

The Bronze layer is the landing area for the original source files.

The project uses AdventureWorks datasets including:

- `AdventureWorks_Calendar.csv`
- `AdventureWorks_Customers.csv`
- `AdventureWorks_Product_Categories.csv`
- `AdventureWorks_Product_Subcategories.csv`
- `AdventureWorks_Products.csv`
- `AdventureWorks_Returns.csv`
- `AdventureWorks_Sales_2015.csv`
- `AdventureWorks_Sales_2016.csv`
- `AdventureWorks_Sales_2017.csv`
- `AdventureWorks_Territories.csv`

The Bronze layer keeps the source data available before processing and modelling.

---

## ⚙️ Azure Databricks Processing

Azure Databricks is used as the processing engine.

The demonstrated notebook workflow includes:

### 1. Configure ADLS Gen2 Access

The Databricks notebook establishes access to the ADLS Gen2 storage account and reads source files from the Bronze path.

> **Security:** Credentials should not be hard-coded in production notebooks. Use Azure Key Vault, Databricks secret scopes, managed identities, or another approved secure authentication mechanism.

### 2. Read CSV Files with PySpark

Example pattern:

```python
df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("<bronze-path>")
```

The project uses this approach to load the AdventureWorks datasets.

---

## 👥 Customer Data Transformation

The customer dataset is loaded from:

```text
AdventureWorks_Customers.csv
```

The project creates a standardized `full_name` field by combining:

```text
Prefix + FirstName + LastName
```

Using PySpark functions such as:

```python
from pyspark.sql.functions import concat_ws, col

df_customers = df_customers.withColumn(
    "full_name",
    concat_ws(" ", col("Prefix"), col("FirstName"), col("LastName"))
)
```

The transformed customer dataset is then written to the Silver layer.

---

## 📦 Product Data

The project reads:

```text
AdventureWorks_Products.csv
AdventureWorks_Product_Subcategories.csv
```

The product data is inspected using Spark DataFrames before being prepared for the processed layer.

Important product attributes shown in the project include:

- `ProductKey`
- `ProductSubcategoryKey`
- `ProductSKU`
- `ProductName`
- `ModelName`
- `ProductDescription`

---

## 🛒 Sales Data Consolidation

The project contains separate yearly sales files:

```text
AdventureWorks_Sales_2015.csv
AdventureWorks_Sales_2016.csv
AdventureWorks_Sales_2017.csv
```

Each file is loaded as a Spark DataFrame and combined using:

```python
sales = sales_2015.unionByName(sales_2016).unionByName(sales_2017)
```

This creates a consolidated Sales dataset containing records across the three years.

The demonstrated Sales dataset includes fields such as:

- `OrderDate`
- `StockDate`
- `OrderNumber`
- `ProductKey`
- `CustomerKey`
- `TerritoryKey`
- `OrderLineItem`
- `OrderQuantity`

---

## 🥈 Silver Layer – Processed Data

After processing, the Silver container contains datasets such as:

```text
silver/
├── Calendar.csv
├── Customers.csv
├── Products.csv
├── Returns.csv
├── Sales.csv
├── Territories.csv
└── product_subcategories/
```

The Silver layer represents the cleaned and standardized data used as the foundation for downstream modelling.

---

## 🥇 Gold Layer – Curated Data

The Gold layer contains analytics-oriented dimensions and facts.

```text
gold/
├── dim_customer/
├── dim_date/
├── dim_product/
├── dim_territory/
├── fact_returns/
└── fact_sales/
```

### Dimensions

| Dataset | Purpose |
|---|---|
| `dim_customer` | Customer-related analytical attributes |
| `dim_date` | Date/calendar information |
| `dim_product` | Product information |
| `dim_territory` | Territory information |

### Facts

| Dataset | Purpose |
|---|---|
| `fact_sales` | Sales transactions |
| `fact_returns` | Return transactions |

This structure provides a curated data model suitable for downstream analytics and BI workloads.

---

## 🗂️ Databricks Catalog

The project also demonstrates organizing the datasets through Databricks Catalog.

### Bronze Catalog

The Bronze schema contains source-oriented tables such as:

```text
calendar
customers
product_categories
product_subcategories
products
returns
territories
```

### Silver Catalog

The Silver schema contains processed tables such as:

```text
calendar
customer
product_categories
product_subcategories
products
returns
sales
territories
```

### Gold Catalog

The Gold schema contains curated analytical tables:

```text
dim_customer
dim_date
dim_product
dim_territory
fact_returns
fact_sales
```

The Catalog provides a structured way to discover and access the data assets rather than relying only on raw storage paths.

---

## 🔄 End-to-End Workflow

```text
1. Create / access ADLS Gen2 storage
             │
             ▼
2. Verify Bronze source files
             │
             ▼
3. Configure Databricks access
             │
             ▼
4. Read AdventureWorks datasets
             │
             ▼
5. Inspect source DataFrames
             │
             ▼
6. Apply PySpark transformations
             │
             ▼
7. Combine yearly Sales datasets
             │
             ▼
8. Write processed datasets to Silver
             │
             ▼
9. Build / organize Gold dimensions & facts
             │
             ▼
10. Register / inspect data through Catalog
             │
             ▼
11. Use curated data for analytics
```

---

## 🛠️ Technologies Used

| Technology | Usage |
|---|---|
| **Azure Data Lake Storage Gen2** | Cloud data storage |
| **Azure Databricks** | Data processing and engineering |
| **Apache Spark** | Distributed data processing |
| **PySpark** | Data transformation |
| **Python** | Notebook programming |
| **Databricks Catalog** | Data organization and discovery |
| **CSV** | Source and demonstrated storage format |
| **Medallion Architecture** | Bronze, Silver, Gold data organization |
| **AdventureWorks Dataset** | Sample retail/business data |

---
## 📸 Documentation

A detailed screenshot-based implementation guide is included with this project.

The guide covers **13 steps**, from verifying the ADLS Gen2 containers through inspecting the Bronze, Silver, and Gold schemas in Databricks Catalog. fileciteturn5file0L3-L13 fileciteturn5file0L132-L141

The documented workflow includes:

- ADLS Gen2 container verification
- Bronze source-data validation
- Databricks ADLS access
- Calendar ingestion
- Customer ingestion
- Customer name standardization
- Silver-layer persistence
- Product ingestion
- Multi-year Sales consolidation
- Silver-layer verification
- Gold-layer verification
- Bronze Catalog inspection
- Silver and Gold Catalog inspection

The source guide documents the Sales consolidation using the 2015, 2016, and 2017 files with `unionByName`. fileciteturn5file0L94-L103

---

## ✅ Project Outcome

This project demonstrates an end-to-end cloud data engineering workflow in which:

**Raw AdventureWorks data**

→ is stored in **ADLS Gen2 Bronze**

→ is processed with **Azure Databricks + PySpark**

→ is standardized and persisted in **Silver**

→ is organized into **Gold dimensions and facts**

→ and is exposed through **Databricks Catalog** for downstream analytics.

The final documented state includes the Gold dimensional and fact structures `dim_customer`, `dim_date`, `dim_product`, `dim_territory`, `fact_returns`, and `fact_sales`. fileciteturn5file0L158-L167

---

## 📚 Key Data Engineering Concepts Demonstrated

- Cloud data lake
- ADLS Gen2
- Azure Databricks
- Apache Spark
- PySpark
- DataFrame operations
- CSV ingestion
- Data inspection
- Data transformation
- `concat_ws`
- Column selection
- `unionByName`
- Bronze / Silver / Gold architecture
- Dimensional modelling
- Fact and dimension datasets
- Data Catalog
- Data discovery
- Cloud data security

---

## ⭐ Summary

This project demonstrates how **Azure Databricks and ADLS Gen2 can be combined to build a structured data engineering pipeline using the Medallion Architecture**.

The project moves data through:

```text
BRONZE
Raw AdventureWorks Data
        ↓
SILVER
Processed & Standardized Data
        ↓
GOLD
Curated Dimensions & Facts
        ↓
DATABRICKS CATALOG
Organized & Discoverable Data Assets
```

It provides a practical foundation for understanding modern cloud data engineering with Azure.

