# RETAIL SALES & INVENTORY ANALYTICS PLATFORM

## 📌 Project Overview

The **Retail Sales & Inventory Analytics Platform** is an end-to-end retail data engineering and analytics project built around a layered data architecture.

The project brings retail data from multiple source files into a data lake environment, processes the data through **Bronze, Silver, and Gold layers**, creates product-level business KPIs, and makes the curated data available in **Power BI** for reporting and analysis.

The overall flow is:

```text
                 SOURCE DATA
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
    Inventory      Orders       Returns
       JSON          CSV          XLSX
        │            │            │
        └────────────┼────────────┘
                     ▼
              DATA INGESTION
                     │
                     ▼
             🥉 BRONZE LAYER
              Raw / Ingested Data
                     │
                     ▼
             DATA PROCESSING
             PySpark / Notebook
                     │
                     ▼
             🥈 SILVER LAYER
              Cleaned Data
                     │
                     ▼
             🥇 GOLD LAYER
          Business-ready KPIs
                     │
                     ▼
                 POWER BI
              Retail Analytics
```

---

# 🎯 Business Objective

The project is designed to provide a consolidated view of retail sales and inventory information.

The project addresses the need to:

- Bring data from multiple sources into a centralized data platform.
- Maintain a raw copy of incoming data.
- Clean and standardize data before analytics.
- Create business-ready product KPIs.
- Analyze orders, customers, returns, revenue, inventory and product performance.
- Provide a Power BI dashboard for business users.

The project documentation describes the broader requirement as building an end-to-end retail data pipeline and bringing data from multiple sources into a data lake. The source systems include transaction, store and product data, along with customer data supplied in JSON format.

---

# 🏗️ Architecture

```text
┌─────────────────────────────────────────────────────────┐
│                    SOURCE SYSTEMS                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  inventory_data.json     orders_data.csv                │
│  returns_data.xlsx                                      │
│                                                         │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│                DATA INGESTION PIPELINE                  │
│                                                         │
│     Orders Copy  →  Inventory Copy  →  Returns Copy    │
│                                                         │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│                    🥉 BRONZE                            │
│                                                         │
│  Raw / ingested datasets                                │
│  inventory_data.parquet                                 │
│  orders_data.parquet                                    │
│  returns_data.xlsx.parquet                              │
│                                                         │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│                 NOTEBOOK / PYSPARK                      │
│                                                         │
│  • Read source data                                     │
│  • Inspect DataFrames                                   │
│  • Clean and standardize data                           │
│  • Prepare analytical datasets                          │
│                                                         │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│                    🥈 SILVER                            │
│                                                         │
│  Cleaned and standardized datasets                      │
│  Example: silver_inventory                              │
│                                                         │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│                     🥇 GOLD                             │
│                                                         │
│                  gold_product_kpis                       │
│                                                         │
│  Orders • Customers • Returns • Revenue • Cost          │
│                                                         │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│                     POWER BI                            │
│                                                         │
│              Retail Sales Dashboard                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

# 📥 Source Data

The ingestion stage shown in the project uses three source files:

| Source | Format | Purpose |
|---|---|---|
| `inventory_data.json` | JSON | Inventory information |
| `orders_data.csv` | CSV | Retail order transactions |
| `returns_data.xlsx` | Excel | Return information |

The ingestion pipeline contains separate copy activities for:

```text
Orders
Inventory
Returns
```

The pipeline run shown in the project completes these ingestion activities successfully.

---

# 🔄 Data Ingestion

The first stage is responsible for moving source data into the data-lake environment.

### Pipeline flow

```text
Orders Source
     │
     ▼
Copy Data
     │
     ▼
Bronze

Inventory Source
     │
     ▼
Copy Data
     │
     ▼
Bronze

Returns Source
     │
     ▼
Copy Data
     │
     ▼
Bronze
```

The pipeline execution is monitored to confirm that the individual copy activities complete successfully.

---

# 🥉 Bronze Layer

The Bronze layer contains the ingested/raw representation of the source data.

The project screenshot shows the following files:

```text
Bronze/
│
├── inventory_data.parquet
├── orders_data.parquet
└── returns_data.xlsx.parquet
```

The purpose of this layer is to retain the incoming data before applying the main cleaning and transformation logic.

### Why Bronze?

The Bronze layer provides:

- A raw landing area.
- A recoverable copy of source data.
- Separation between ingestion and transformation.
- A consistent starting point for downstream processing.

---

# ⚙️ Databricks / Notebook Processing

The project uses a notebook-based processing stage to read and inspect the ingested data.

The notebook demonstrates loading the inventory dataset and displaying it as a table.

The inventory data contains fields such as:

```text
product_id
productName
stock
last_stocked
warehouse
cost_price
available
```

A typical processing workflow is:

```text
Bronze Data
     │
     ▼
Read DataFrame
     │
     ▼
Inspect Schema / Records
     │
     ▼
Clean / Standardize
     │
     ▼
Silver Dataset
```

---

# 🧹 Silver Layer

The Silver layer contains processed data.

The project demonstrates a cleaned inventory dataset named:

```text
silver_inventory
```

The Silver table contains standardized fields including:

```text
product_id
ProductName
LastStocked
CostPrice
```

The Silver layer is the stage where raw data is prepared for reliable analytical processing.

### Typical Silver responsibilities

- Data cleaning
- Column standardization
- Data-type handling
- Date standardization
- Removing or handling invalid values
- Preparing datasets for business transformations

---

# 🥇 Gold Layer

The Gold layer contains business-ready analytical data.

The main analytical dataset demonstrated in this project is:

```text
gold_product_kpis
```

This dataset brings together product-level metrics that can be consumed by Power BI.

---

# 📊 Gold Product KPIs

The Gold dataset contains business metrics including:

| KPI | Description |
|---|---|
| `Product_Name` | Product being analyzed |
| `Total_Orders` | Total number of orders associated with the product |
| `Unique_Customers` | Number of distinct customers |
| `Total_Returns` | Total returns associated with the product |
| `Return_Rate_Percent` | Return rate percentage |
| `Total_Return_Amount` | Total value associated with returns |
| `Total_Revenue` | Revenue generated by the product |
| `Avg_Order_Value` | Average order value |
| `Avg_Cost` | Average product cost |

The Gold layer therefore converts processed data into metrics that are directly useful for business reporting.

---

# 📈 Power BI Reporting

The final stage connects the Gold dataset to Power BI.

The project contains a **Retail Sales Dashboard** based on the curated product KPI dataset.

The dashboard provides an analytical interface for exploring product-level performance.

The report includes:

- Product filtering
- Product KPI information
- Revenue analysis
- Return analysis
- Product-level comparisons
- Business performance visualizations

### Reporting flow

```text
gold_product_kpis
        │
        ▼
Power BI Semantic Model
        │
        ▼
Retail Sales Dashboard
```

---

# 🔍 End-to-End Workflow

```text
1. Receive source data
          │
          ▼
2. Ingest Orders / Inventory / Returns
          │
          ▼
3. Store ingested data in Bronze
          │
          ▼
4. Read Bronze data using notebook
          │
          ▼
5. Inspect and transform the data
          │
          ▼
6. Store cleaned data in Silver
          │
          ▼
7. Generate product-level KPIs
          │
          ▼
8. Store curated data in Gold
          │
          ▼
9. Connect Gold data to Power BI
          │
          ▼
10. Analyze retail performance
```

---

# 🧱 Medallion Architecture

This project follows the basic Medallion Architecture pattern.

## 🥉 Bronze

**Purpose:** Raw / ingested data

```text
Source → Bronze
```

Data is retained close to its incoming structure.

## 🥈 Silver

**Purpose:** Cleaned and standardized data

```text
Bronze → Cleaning → Silver
```

Data is prepared for reliable downstream analysis.

## 🥇 Gold

**Purpose:** Business-ready analytical data

```text
Silver → Business KPIs → Gold
```

The Gold layer contains metrics designed for reporting and decision support.

---

# 🛠️ Technologies Used

| Technology | Role |
|---|---|
| **Microsoft Fabric / Data Factory** | Data ingestion and pipeline orchestration |
| **Azure Data Lake / Lakehouse storage** | Data storage |
| **Notebook / PySpark** | Data processing and transformation |
| **Parquet** | Processed data storage |
| **JSON** | Inventory source format |
| **CSV** | Orders source format |
| **Excel/XLSX** | Returns source format |
| **Power BI** | Reporting and visualization |
| **Medallion Architecture** | Bronze/Silver/Gold organization |

---

### Bronze validation

```text
Did the ingestion pipeline create the expected files?
```

### Silver validation

```text
Is the data cleaned and structured correctly?
```

### Gold validation

```text
Are the required product KPIs available?
```

### Power BI validation

```text
Can the Gold dataset be consumed successfully by the dashboard?
```

---

# 🎓 Key Data Engineering Concepts Demonstrated

This project provides practical exposure to:

- Microsoft Fabric
- Data ingestion
- Data pipelines
- Data lake / lakehouse concepts
- Bronze layer
- Silver layer
- Gold layer
- Medallion Architecture
- PySpark
- DataFrames
- JSON ingestion
- CSV ingestion
- Excel ingestion
- Parquet
- Data cleaning
- Data transformation
- KPI generation
- Power BI
- Business intelligence
- Retail analytics

---

# 📌 Project Outcome

The project creates an end-to-end analytical flow:

```text
MULTIPLE SOURCES
       ↓
DATA INGESTION
       ↓
BRONZE
Raw Data
       ↓
SILVER
Cleaned Data
       ↓
GOLD
Product KPIs
       ↓
POWER BI
Retail Sales Analytics
```

The final Gold dataset provides product-level metrics covering:

```text
Orders
Customers
Returns
Revenue
Cost
Return Rate
Average Order Value
```

These metrics form the analytical foundation for the Retail Sales Dashboard.

## ⭐ Summary

**Retail Sales & Inventory Analytics Platform** demonstrates how retail data from multiple source formats can be ingested, processed through a layered data architecture, converted into business KPIs, and consumed through Power BI.

### Core pipeline

```text
SOURCE
  ↓
INGEST
  ↓
BRONZE
  ↓
TRANSFORM
  ↓
SILVER
  ↓
CURATE
  ↓
GOLD
  ↓
POWER BI
```

The project provides a practical example of building a retail analytics solution using modern cloud data engineering concepts.

