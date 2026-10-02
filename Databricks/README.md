# Azure Databricks Data Engineering

## 📌 Overview

This folder contains **Azure Databricks data engineering work** focused on processing data stored in **Azure Data Lake Storage Gen2 (ADLS Gen2)** using **Apache Spark / PySpark**.

The main demonstrated project uses **AdventureWorks retail data** and follows the **Medallion Architecture**:

```text
                    Azure Data Lake Storage Gen2
                               │
                               ▼
                    ┌─────────────────────┐
                    │   🥉 BRONZE / RAW   │
                    │                     │
                    │ AdventureWorks CSV  │
                    │ source datasets     │
                    └──────────┬──────────┘
                               │
                               │ PySpark
                               ▼
                    ┌─────────────────────┐
                    │ 🥈 SILVER / PROCESSED│
                    │                     │
                    │ Cleaned /           │
                    │ standardized data  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  🥇 GOLD / CURATED  │
                    │                     │
                    │ Dimensions + Facts  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ DATABRICKS CATALOG  │
                    │                     │
                    │ Bronze / Silver /   │
                    │ Gold schemas        │
                    └─────────────────────┘
```

This type of Bronze → Silver → Gold organization is commonly used to separate raw ingestion, transformation, and curated analytical data in lakehouse architectures. citeturn0search5turn0search11

---

## 🎯 Project Objectives

The Databricks work in this folder demonstrates how to:

- Connect Azure Databricks to **ADLS Gen2**.
- Access raw files stored in the Bronze layer.
- Read CSV datasets using PySpark.
- Inspect source DataFrames.
- Perform data transformations.
- Standardize customer information.
- Combine multiple yearly Sales datasets.
- Persist processed data into the Silver layer.
- Organize analytical dimensions and facts in the Gold layer.
- Explore datasets through **Databricks Catalog**.
- Understand the complete flow from raw data to analytics-ready data.

---

# 🏗️ Project Architecture

```text
┌──────────────────────────────────────────────────────────┐
│                  Azure Data Lake Storage Gen2             │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│                    BRONZE / RAW                           │
│                                                          │
│  Calendar                                                │
│  Customers                                               │
│  Product Categories                                      │
│  Product Subcategories                                   │
│  Products                                                │
│  Returns                                                 │
│  Sales 2015 / 2016 / 2017                                │
│  Territories                                             │
└────────────────────────────┬─────────────────────────────┘
                             │
                             │ Read with Spark
                             ▼
┌──────────────────────────────────────────────────────────┐
│                 AZURE DATABRICKS                         │
│                                                          │
│  PySpark                                                 │
│  DataFrame Processing                                    │
│  Data Inspection                                         │
│  Data Transformation                                     │
│  Data Consolidation                                      │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│                   SILVER / PROCESSED                     │
│                                                          │
│  Calendar                                                │
│  Customers                                               │
│  Products                                                │
│  Product Subcategories                                   │
│  Returns                                                 │
│  Sales                                                   │
│  Territories                                             │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│                    GOLD / CURATED                        │
│                                                          │
│  dim_customer                                            │
│  dim_date                                                │
│  dim_product                                             │
│  dim_territory                                           │
│  fact_returns                                            │
│  fact_sales                                              │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
                    Databricks Catalog
```

---

# 📂 Data Source

The demonstrated project uses the **AdventureWorks** dataset.

### Bronze source files

```text
AdventureWorks_Calendar.csv
AdventureWorks_Customers.csv
AdventureWorks_Product_Categories.csv
AdventureWorks_Product_Subcategories.csv
AdventureWorks_Products.csv
AdventureWorks_Returns.csv
AdventureWorks_Sales_2015.csv
AdventureWorks_Sales_2016.csv
AdventureWorks_Sales_2017.csv
AdventureWorks_Territories.csv
```

These files are initially stored in the **Bronze** container of ADLS Gen2.

---

# 🥉 Bronze Layer

The Bronze layer is the raw-data landing layer.

The project first verifies the ADLS Gen2 storage structure and confirms that the required Bronze source files are available.

```text
bronze/
├── Calendar
├── Customers
├── Product Categories
├── Product Subcategories
├── Products
├── Returns
├── Sales 2015
├── Sales 2016
├── Sales 2017
└── Territories
```

The Bronze layer is intentionally close to the original source structure before transformation.

---

# ⚡ Databricks + PySpark Processing

Azure Databricks is used as the processing environment.

The demonstrated workflow includes:

1. Configure access to ADLS Gen2.
2. Read source files from Bronze.
3. Create Spark DataFrames.
4. Inspect the data.
5. Apply transformations.
6. Combine related datasets.
7. Write processed results to Silver.
8. Prepare curated structures for Gold.

---

## 🔐 ADLS Gen2 Connection

The Databricks notebook establishes access to the ADLS Gen2 storage account and reads data from the Bronze path.

A typical PySpark CSV read follows this pattern:

```python
df = (
    spark.read
    .format("csv")
    .option("header", "true")
    .option("inferSchema", "true")
    .load("<bronze-path>")
)
```

### Security

For a production implementation:

- Do not hard-code storage keys or passwords.
- Use **Azure Key Vault** or Databricks secret management.
- Prefer Managed Identity / service-principal based authentication where supported.
- Never commit secrets to GitHub.

---

# 👥 Customer Data Processing

The customer dataset is read from:

```text
AdventureWorks_Customers.csv
```

Important customer fields include:

```text
CustomerKey
Prefix
FirstName
LastName
BirthDate
MaritalStatus
Gender
EmailAddress
```

A standardized `full_name` column is created by combining customer name fields.

Example:

```python
from pyspark.sql.functions import concat_ws, col

df_customers = df_customers.withColumn(
    "full_name",
    concat_ws(
        " ",
        col("Prefix"),
        col("FirstName"),
        col("LastName")
    )
)
```

This produces a cleaner representation of customer names for downstream processing.

---

# 📦 Product Processing

The project reads product-related datasets including:

```text
AdventureWorks_Product_Categories.csv
AdventureWorks_Product_Subcategories.csv
AdventureWorks_Products.csv
```

Product information demonstrated in the notebook includes fields such as:

```text
ProductKey
ProductSubcategoryKey
ProductSKU
ProductName
ModelName
ProductDescription
```

The data is inspected before being persisted into the processed layer.

---

# 🛒 Sales Data Processing

The project contains Sales data split across three yearly files:

```text
AdventureWorks_Sales_2015.csv
AdventureWorks_Sales_2016.csv
AdventureWorks_Sales_2017.csv
```

Each file is read into a Spark DataFrame.

The yearly datasets are then combined into one consolidated Sales dataset using `unionByName`.

Example:

```python
sales = (
    sales_2015
    .unionByName(sales_2016)
    .unionByName(sales_2017)
)
```

Important Sales fields include:

```text
OrderDate
StockDate
OrderNumber
ProductKey
CustomerKey
TerritoryKey
OrderLineItem
OrderQuantity
```

This allows the three yearly datasets to be processed as one logical Sales dataset.

---

# 🥈 Silver Layer

After processing, the Silver container contains processed datasets.

The demonstrated structure includes:

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

The Silver layer is used as the cleaned and standardized foundation for downstream modelling.

---

# 🥇 Gold Layer

The Gold layer contains curated analytical structures.

```text
gold/
├── dim_customer/
├── dim_date/
├── dim_product/
├── dim_territory/
├── fact_returns/
└── fact_sales/
```

## Dimensions

| Table | Description |
|---|---|
| `dim_customer` | Customer dimension |
| `dim_date` | Calendar/date dimension |
| `dim_product` | Product dimension |
| `dim_territory` | Territory dimension |

## Facts

| Table | Description |
|---|---|
| `fact_sales` | Sales transactions |
| `fact_returns` | Return transactions |

The Gold layer provides a curated structure for analytics and downstream BI workloads.

---

# 🗂️ Databricks Catalog

The project also demonstrates viewing the data through **Databricks Catalog**.

## Bronze Schema

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

## Silver Schema

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

## Gold Schema

The Gold schema contains the curated analytical model:

```text
dim_customer
dim_date
dim_product
dim_territory
fact_returns
fact_sales
```

This makes the datasets easier to discover and organize within the Databricks environment.

---

# 🔄 End-to-End Data Flow

```text
AdventureWorks CSV Files
          │
          ▼
Azure Data Lake Storage Gen2
          │
          ▼
     Bronze Layer
          │
          │
          ▼
   Azure Databricks
          │
          ├── Read
          ├── Inspect
          ├── Transform
          └── Combine
          │
          ▼
     Silver Layer
          │
          ▼
   Curated Modelling
          │
          ▼
      Gold Layer
          │
          ▼
   Databricks Catalog
          │
          ▼
     Analytics / BI
```

---

# 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| **Azure Databricks** | Data engineering and Spark processing |
| **Azure Data Lake Storage Gen2** | Cloud data lake storage |
| **Apache Spark** | Distributed data processing |
| **PySpark** | Data transformation |
| **Python** | Notebook development |
| **Databricks Catalog** | Data discovery and organization |
| **CSV** | Source data format |
| **AdventureWorks** | Retail/business sample dataset |
| **Medallion Architecture** | Bronze/Silver/Gold organization |

Azure Databricks projects commonly combine Spark/PySpark with Azure storage and layered lakehouse architectures. citeturn0search0turn0search3

---

# 📁 Suggested Databricks Folder Structure

Use the following structure if the Databricks folder contains notebooks, documentation, and screenshots:

```text
Databricks/
│
├── README.md
│
├── Notebooks/
│   ├── AdventureWorks/
│   └── Data_Transformation/
│
├── Screenshots/
│
└── Documentation/
    └── Azure_Databricks_ADLS_Medallion_Step_by_Step_Guide.pdf
```

If your current GitHub folder uses different filenames, keep those actual filenames rather than renaming them only for the README.

---

# 📚 Documentation

A detailed screenshot-based guide accompanies this project.

The guide documents **13 implementation steps**, beginning with ADLS Gen2 container verification and continuing through Bronze, Silver, Gold, and Databricks Catalog validation.

The documented steps include:

- ADLS Gen2 container verification
- Bronze source-data verification
- Databricks access configuration
- Calendar ingestion
- Customer ingestion
- Customer name standardization
- Silver persistence
- Product processing
- Sales 2015–2017 consolidation
- Silver verification
- Gold verification
- Bronze Catalog inspection
- Silver Catalog inspection
- Gold Catalog inspection

The guide specifically documents the consolidation of the three yearly Sales files into a single dataset using `unionByName`.

---

# 🔒 Security Guidelines

Before pushing Databricks notebooks or screenshots to a public GitHub repository:

### Never commit

```text
Storage Account Keys
Client Secrets
Passwords
Access Tokens
Connection Strings containing secrets
```

### Recommended approach

```text
Azure Key Vault
       │
       ▼
Secure Secret / Identity
       │
       ▼
Azure Databricks
       │
       ▼
ADLS Gen2
```

The project documentation also identifies credential configuration in the demonstrated notebook and recommends using Key Vault / Databricks secret management rather than hard-coded credentials.

---

# 🎓 Key Data Engineering Concepts

This project provides practical exposure to:

- Azure Data Lake Storage Gen2
- Azure Databricks
- Apache Spark
- PySpark
- DataFrames
- CSV ingestion
- Data transformation
- Data standardization
- `concat_ws`
- `unionByName`
- Medallion Architecture
- Bronze Layer
- Silver Layer
- Gold Layer
- Dimensional modelling
- Fact tables
- Dimension tables
- Databricks Catalog
- Cloud data security

---

# ✅ Project Outcome

The demonstrated workflow successfully shows how raw AdventureWorks data can move through a cloud data engineering architecture:

```text
RAW DATA
   ↓
ADLS GEN2 — BRONZE
   ↓
DATABRICKS + PYSPARK
   ↓
ADLS GEN2 — SILVER
   ↓
CURATED DIMENSIONS & FACTS
   ↓
ADLS GEN2 — GOLD
   ↓
DATABRICKS CATALOG
   ↓
ANALYTICS-READY DATA
```

The final curated model includes:

```text
dim_customer
dim_date
dim_product
dim_territory
fact_returns
fact_sales
```

## ⭐ Summary

This Databricks project demonstrates a practical **Azure cloud data engineering workflow** using:

**ADLS Gen2 + Azure Databricks + PySpark + Medallion Architecture + Databricks Catalog**

It shows the complete journey from raw AdventureWorks files to processed and curated data structures that can support downstream analytics and business intelligence.

