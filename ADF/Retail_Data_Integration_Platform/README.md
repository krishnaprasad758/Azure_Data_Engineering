# Retail Data Integration Platform — Azure Data Factory

## 📌 Project Overview

The **Retail Data Integration Platform (RDIP)** is an Azure Data Factory-based data integration project that brings together retail data from multiple sources and converts it into a clean, business-ready dataset for reporting.

The solution integrates:

- **Orders** from an Azure SQL Database
- **Product Catalog** from a vendor REST API
- **Store Master** from CSV files in Azure Blob Storage

The data is processed through a **Bronze → Silver → Gold** architecture:

1. **Bronze / Raw Zone** – stores source data after ingestion.
2. **Silver / Processed Zone** – cleans, joins and aggregates the data.
3. **Gold / Curated Zone** – stores the final business-ready `SalesSummary` data in Azure SQL Database.

The complete workflow is orchestrated through a master Azure Data Factory pipeline.

> **Project scope:** The implementation uses Azure Data Factory for ingestion, transformation and orchestration. No external compute engine such as Databricks or Synapse Spark is used.

---

## 🏗️ Architecture

```text
                         ┌──────────────────────┐
                         │      Source Systems  │
                         └──────────┬───────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
          ┌─────────────┐    ┌─────────────┐    ┌──────────────┐
          │ Azure SQL   │    │ Vendor REST │    │ Store Master │
          │   Orders    │    │ Product API │    │     CSV      │
          └──────┬──────┘    └──────┬──────┘    └──────┬───────┘
                 │                  │                  │
                 └──────────────────┼──────────────────┘
                                    ▼
                    ┌────────────────────────────┐
                    │ PL_01_Ingest_RawZone       │
                    │ Metadata-driven ingestion  │
                    │ Lookup → ForEach → Switch  │
                    └──────────────┬─────────────┘
                                   ▼
                    ┌────────────────────────────┐
                    │       BRONZE / RAW         │
                    │ Azure Data Lake Storage    │
                    │ Products / Stores / Orders │
                    └──────────────┬─────────────┘
                                   │
                                   ▼
                    ┌────────────────────────────┐
                    │ PL_02_Transform_Processed │
                    │      Mapping Data Flow     │
                    └──────────────┬─────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
          Filter               Derived              Joins
       Invalid rows          Standardize         Orders + Products
                                                  + Stores
                                   │
                                   ▼
                              Aggregate
                           Daily Sales KPIs
                                   │
                                   ▼
                    ┌────────────────────────────┐
                    │      SILVER / PROCESSED    │
                    │ Curated aggregated dataset │
                    └──────────────┬─────────────┘
                                   │
                                   ▼
                    ┌────────────────────────────┐
                    │ PL_03_Load_CuratedZone     │
                    │      Upsert to SQL         │
                    └──────────────┬─────────────┘
                                   │
                                   ▼
                    ┌────────────────────────────┐
                    │       GOLD / CURATED       │
                    │ Azure SQL - SalesSummary   │
                    └──────────────┬─────────────┘
                                   │
                                   ▼
                              Power BI
```

---

## 🎯 Project Objectives

The project was designed to:

- Automate ingestion from multiple retail data sources.
- Remove manual data consolidation.
- Implement incremental loading using a watermark mechanism.
- Store raw source data separately from processed data.
- Clean and standardize retail order data.
- Join Orders, Products and Stores into a single dataset.
- Generate daily sales-level aggregates.
- Load curated data into Azure SQL Database.
- Prevent duplicate records during reruns using upsert logic.
- Validate the final row count after loading.
- Provide failure notifications through a webhook.
- Orchestrate the complete workflow from one master pipeline.

---

## ☁️ Azure Resources

The implementation uses the following Azure resources:

| Resource | Purpose |
|---|---|
| Azure Data Factory | Pipeline orchestration and data integration |
| Azure SQL Database | Source Orders data and target curated SalesSummary |
| Azure SQL Server | SQL hosting environment |
| Azure Storage Account / ADLS | Bronze and Silver data zones |
| REST API | Product Catalog source |
| Azure Blob Storage | Store Master CSV source |

The build documentation shows the Azure resource group containing the SQL database, Azure Data Factory, SQL server and storage account. fileciteturn1file7

---

# 🔗 Linked Services

The ADF implementation contains linked services for connecting to the different data sources and targets:

| Linked Service | Purpose |
|---|---|
| `AzureDataLakeStorage1` | Connects to Bronze/Silver storage |
| `LS_SQL_SOURCE` | Connects to Orders source database |
| `LS_SQL_TARGET` | Connects to curated target database |
| `RestService1` | Connects to Product Catalog REST API |

These connections allow the pipelines to move data between SQL, REST API and Azure Storage. fileciteturn1file0

---

# 🗄️ Data Zones

## 🥉 Bronze — Raw Zone

The Bronze zone stores the ingested source data.

Example files generated during the implementation:

```text
Products.csv
Stores.csv
dbo.Orders.csv
```

The ingestion pipeline successfully landed these files in the raw container. fileciteturn1file7

**Purpose:**

- Preserve source data after ingestion.
- Separate ingestion from transformation.
- Provide a raw layer for troubleshooting and reprocessing.

---

## 🥈 Silver — Processed Zone

The Silver zone contains the cleaned, joined and aggregated dataset.

Transformations include:

- Null filtering
- Date standardization
- Load date creation
- Orders + Products join
- Orders + Stores join
- Daily sales aggregation

The processed output contains business-oriented fields such as:

```text
ProductID
StoreID
OrderDate
ProductName
StoreName
Category
Ordercount
TotalSales
TotalQuantity
```

The build successfully wrote the processed output to the Silver zone. fileciteturn1file8

---

## 🥇 Gold — Curated Zone

The Gold layer is represented by the curated `SalesSummary` table in Azure SQL Database.

The final dataset contains business-ready sales information such as:

```text
SalesDate
StoreID
StoreName
ProductID
ProductName
Category
TotalQuantity
TotalSales
```

The completed build produced **62 rows** of curated sales data ready for reporting/Power BI consumption. fileciteturn2file5

---

# 🔄 ADF Pipelines

The solution contains four major pipelines.

## 1. `PL_00_Master_Orchestrator`

The master pipeline controls the complete end-to-end workflow.

```text
PL_01_Ingest_RawZone
          │
       Success
          ▼
PL_02_Transform_ProcessedZone
          │
       Success
          ▼
PL_03_Load_CuratedZone
```

`Wait on completion` is enabled so that each stage completes before the next stage begins. The build successfully executed all three pipelines in sequence. fileciteturn2file0

---

## 2. `PL_01_Ingest_RawZone`

This is the ingestion pipeline.

### Step 1 — Lookup Active Sources

The pipeline reads the `WatermarkControl` table:

```sql
SELECT *
FROM WatermarkControl
WHERE IsActive = 1;
```

The Lookup returns the active source records.

### Step 2 — ForEach

The ForEach iterates over each active source:

```text
@activity('Lookup1').output.value
```

### Step 3 — Source-Based Routing

A Switch activity evaluates:

```text
@item().SourceType
```

and routes the source to the appropriate ingestion logic.

### SQL Source

For SQL:

```text
Copy Data
    ↓
Orders → Bronze
    ↓
SP_UpdateWatermark
```

The stored procedure updates the watermark after a successful load.

### REST Source

The REST branch copies Product Catalog data into the Bronze zone.

### Blob Source

The Blob branch uses Get Metadata to check whether the Store Master CSV exists before processing it.

The implementation successfully completed the ingestion iterations and landed the Products, Stores and Orders files in Bronze. fileciteturn1file7

---

# 3. `PL_02_Transform_ProcessedZone`

This pipeline contains the Mapping Data Flow used to transform the Bronze datasets.

## Transformation Flow

```text
Orders
   │
   ▼
Filter
   │
   ▼
Derived Column
   │
   ├──────────────┐
   ▼              │
Products ──► Join Orders + Products
                     │
                     ▼
Stores ─────────► Join with Stores
                     │
                     ▼
                 Aggregate
                     │
                     ▼
                  Silver
```

### Filter

Rows with missing critical fields are removed:

- `OrderID`
- `ProductID`
- `StoreID`
- `OrderDate`

### Derived Column

The pipeline standardizes `OrderDate` and adds:

```text
LoadDate = currentDate()
```

The build preview showed clean records after the filtering and derived-column transformations. fileciteturn1file8

### Join — Products

Orders are joined with Products using:

```text
ProductID
```

### Join — Stores

The enriched Orders + Products dataset is then joined with Stores using:

```text
StoreID
```

### Aggregate

Daily sales are aggregated using:

```text
Ordercount  = count(OrderID)
TotalSales  = sum(UnitPrice)
TotalStock  = sum(Quantity)
```

The transformation preview produced **62 aggregated rows** summarizing sales by store and product. fileciteturn2file6

---

# 4. `PL_03_Load_CuratedZone`

This pipeline loads the processed data into Azure SQL.

## Upsert Logic

The Copy Data activity uses **Upsert** behavior.

Business key columns include:

```text
StoreID
ProductID
```

This allows existing records to be updated instead of creating duplicates when the pipeline is rerun. fileciteturn1file5

---

# ✅ Data Quality Validation

After the SQL load, the pipeline validates the number of rows in `SalesSummary`.

The Lookup executes:

```sql
SELECT COUNT(*) AS [RowCount]
FROM SalesSummary;
```

A Fail activity is configured with:

```text
Error Code:
ROW_COUNT_VALIDATION_FAILED
```

If the row count falls below the configured threshold, the pipeline can fail rather than silently completing with an incomplete dataset. fileciteturn2file3

---

# 🚨 Failure Notification

Failure notification is implemented in the master orchestration layer.

Web activities are attached to failure paths:

```text
WEB_Notify_PL01_F
WEB_Notify_PL02_F
WEB_Notify_PL03_F
```

The notification payload includes information such as:

- Failed pipeline name
- Pipeline run ID
- Failure time

The build uses a GitHub webhook endpoint for the notification path. fileciteturn1file6

---

# 🔐 Incremental Loading & Watermark

The project uses a metadata-driven `WatermarkControl` table.

The table allows the ingestion pipeline to:

1. Identify active sources.
2. Store the last successful load timestamp.
3. Use the timestamp for incremental ingestion.
4. Update the watermark after a successful source load.

The stored procedure used in the implementation is:

```text
dbo.SP_UpdateWatermark
```

It receives the source name and the new watermark value after a successful SQL ingestion. fileciteturn1file0

This design makes the ingestion framework reusable instead of creating separate hardcoded pipeline logic for every source.

---

# 📊 Final Dataset

The final `SalesSummary` dataset is designed for business reporting.

Example structure:

| Column | Description |
|---|---|
| `SalesDate` | Date of sales |
| `StoreID` | Store identifier |
| `StoreName` | Store name |
| `ProductID` | Product identifier |
| `ProductName` | Product name |
| `Category` | Product category |
| `TotalQuantity` | Aggregated quantity |
| `TotalSales` | Aggregated sales value |

The completed implementation returned 62 rows from the curated table during validation. fileciteturn2file5

---

# 🧪 Validation & Testing

The project was tested at multiple stages.

### Ingestion Validation

Verified that:

```text
Products.csv
Stores.csv
dbo.Orders.csv
```

were successfully written to Bronze.

### Transformation Validation

Verified:

- Null records were filtered.
- Dates were standardized.
- Product information was joined.
- Store information was joined.
- Aggregations were generated.
- Silver output was created.

### Curated Load Validation

Verified:

- Data was loaded into `SalesSummary`.
- Upsert behavior was configured.
- Row count validation succeeded.
- Final curated data was available for reporting.

### End-to-End Validation

The final master pipeline execution successfully completed:

```text
PL_01_Ingest_RawZone
        ↓
PL_02_Transform_ProcessedZone
        ↓
PL_03_Load_CuratedZone
```

confirming the complete Bronze → Silver → Gold workflow. fileciteturn2file0

---

# 🧰 Technologies Used

| Technology | Usage |
|---|---|
| Microsoft Azure | Cloud platform |
| Azure Data Factory | ETL/ELT and orchestration |
| Azure Data Lake Storage | Bronze/Silver storage |
| Azure SQL Database | Source and curated target |
| Azure Blob Storage | Store Master file ingestion |
| REST API | Product Catalog ingestion |
| SQL | Source/target queries and validation |
| ADF Mapping Data Flow | Data cleansing and transformation |
| Web Activity | Failure notification |
| Stored Procedure | Watermark management |
| Power BI | Intended reporting/consumption layer |

---

# 📁 Suggested GitHub Repository Structure

```text
Retail-Data-Integration-Platform/
│
├── README.md
│
├── adf/
│   ├── pipelines/
│   │   ├── PL_00_Master_Orchestrator.json
│   │   ├── PL_01_Ingest_RawZone.json
│   │   ├── PL_02_Transform_ProcessedZone.json
│   │   └── PL_03_Load_CuratedZone.json
│   │
│   ├── dataflows/
│   │   └── DF_CleanAndJoin.json
│   │
│   ├── datasets/
│   │   └── ...
│   │
│   ├── linkedServices/
│   │   └── ...
│   │
│   └── triggers/
│       └── ...
│
├── sql/
│   ├── create_tables.sql
│   ├── watermark_procedure.sql
│   └── validation_queries.sql
│
├── docs/
│   ├── Retail_Data_Integration_Platform_ADF_Build.pdf
│   └── BRD_Retail_Data_Integration_ADF.docx
│
└── screenshots/
    └── ...
```

> **Note:** Do not commit passwords, connection strings, API keys, webhook secrets or other credentials to GitHub.

---

# 🚀 How the Pipeline Works

The complete process can be summarized as:

```text
1. Trigger Master Pipeline
          ↓
2. Read Active Sources
          ↓
3. Ingest SQL / REST / Blob
          ↓
4. Store Raw Data in Bronze
          ↓
5. Clean and Standardize
          ↓
6. Join Orders + Products + Stores
          ↓
7. Aggregate Daily Sales
          ↓
8. Write Processed Data to Silver
          ↓
9. Upsert into SalesSummary
          ↓
10. Validate Row Count
          ↓
11. Notify on Failure
          ↓
12. Data Ready for Power BI
```

---

# 💡 Key Data Engineering Concepts Demonstrated

This project demonstrates practical implementation of:

- **ETL / ELT**
- **Azure Data Factory**
- **Metadata-driven pipelines**
- **Incremental loading**
- **Watermark-based ingestion**
- **Bronze / Silver / Gold architecture**
- **Data cleansing**
- **Data standardization**
- **Data joining**
- **Data aggregation**
- **Idempotent / Upsert loading**
- **Pipeline orchestration**
- **Data quality validation**
- **Failure handling**
- **Webhook-based alerting**
- **Azure SQL**
- **Azure Data Lake Storage**
- **REST API integration**
- **Mapping Data Flows**

---

# 📈 Project Outcome

The completed implementation demonstrates an end-to-end retail data integration workflow using Azure Data Factory.

The pipeline successfully:

- Ingested data from SQL, REST API and Blob sources.
- Stored source data in the Bronze zone.
- Cleaned and enriched the datasets.
- Aggregated sales data.
- Wrote the processed dataset to Silver.
- Loaded the curated `SalesSummary` table into Azure SQL.
- Validated the final row count.
- Configured failure notifications.
- Executed the complete Bronze → Silver → Gold workflow from the master orchestrator.

The final curated dataset contained **62 business-ready rows** during the documented validation run. fileciteturn2file5

---

# 👨‍💻 Project Role

**Role:** Data Engineering / Azure Data Factory Project

### Responsibilities demonstrated

- Azure resource setup
- Azure Data Factory pipeline development
- Linked service configuration
- SQL table and stored procedure setup
- Metadata-driven ingestion
- Incremental loading
- Mapping Data Flow development
- Data cleansing and transformation
- Data joining and aggregation
- SQL upsert implementation
- Data quality validation
- Pipeline orchestration
- Failure notification configuration
- End-to-end testing

---

# 📚 Documentation

The repository can include the detailed implementation document:

- `Retail_Data_Integration_Platform_ADF_Build.pdf` — step-by-step ADF build and validation documentation.


The BRD defines the project as an Azure Data Factory-only implementation and describes the Bronze/Silver/Gold architecture, metadata-driven ingestion, incremental loading, curated Sales Summary and reporting objectives. fileciteturn1file1

---

## ⭐ Project Summary

**Retail Data Integration Platform** is a complete Azure Data Factory data engineering project that demonstrates how multiple heterogeneous retail data sources can be integrated, transformed, validated and delivered as a curated dataset for analytics.

**SQL + REST API + CSV → ADF → Bronze → Silver → Gold/Azure SQL**

---

