# Azure Data Factory — Data Engineering Projects

## 📌 Overview

This folder contains **Azure Data Factory (ADF) data engineering work**, including pipeline development, data ingestion, transformation, orchestration, triggers, validation and monitoring.

The implementations focus on practical Azure Data Engineering patterns such as:

- ETL / ELT pipelines
- Azure SQL integration
- Azure Data Lake Storage / Blob Storage integration
- REST API ingestion
- Metadata-driven ingestion
- Incremental loading
- Watermark-based processing
- Mapping Data Flows
- Bronze → Silver → Gold architecture
- Pipeline orchestration
- Schedule and event-based triggers
- Data quality validation
- Failure handling and notifications
- Azure Data Factory monitoring

Microsoft's Azure Data Factory repository similarly organizes ADF examples around reusable data integration solutions and samples. 
---

# 🏗️ Azure Data Factory Architecture

A typical workflow represented by the projects in this folder is:

```text
                    SOURCE SYSTEMS
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
      Azure SQL       REST API       CSV / Blob
          │              │              │
          └──────────────┼──────────────┘
                         ▼
              ┌─────────────────────┐
              │ Azure Data Factory  │
              │     Pipelines       │
              └──────────┬──────────┘
                         │
                         ▼
                 🥉 BRONZE / RAW
                  Azure Storage
                         │
                         ▼
                 🥈 SILVER / CLEAN
              Mapping Data Flow
                         │
                         ▼
                  🥇 GOLD / CURATED
                   Azure SQL DB
                         │
                         ▼
                    Power BI
```

The Bronze/Silver/Gold pattern separates raw ingestion from transformation and business-ready data, a common organization pattern in Azure data engineering projects. 

---

# 🔧 Technologies

| Technology | Purpose |
|---|---|
| **Azure Data Factory** | Data integration, ETL/ELT and orchestration |
| **Azure SQL Database** | Relational source/target and validation |
| **Azure Data Lake Storage Gen2** | Data lake storage |
| **Azure Blob Storage** | File-based ingestion |
| **REST API** | API-based data ingestion |
| **ADF Mapping Data Flow** | Data transformation |
| **SQL** | Queries, validation and control logic |
| **Azure Key Vault** | Secure credential management |
| **GitHub** | Source control and project documentation |
| **Power BI** | Analytics/reporting consumption |

---

# 📂 What This ADF Work Demonstrates

## 1. Data Ingestion

ADF Copy Data activities can be used to ingest data from different source types.

Examples include:

```text
Azure SQL
    ↓
Azure Data Factory
    ↓
Azure Storage
```

```text
REST API
    ↓
Azure Data Factory
    ↓
Azure Storage
```

```text
CSV / Blob Storage
    ↓
Azure Data Factory
    ↓
Processing Layer
```

---

# 2. Metadata-Driven Pipelines

A metadata-driven design allows one pipeline to process multiple sources based on configuration rather than creating a separate pipeline for every source.

A typical pattern is:

```text
Lookup Control Table
        ↓
     ForEach
        ↓
 SourceType / Configuration
        ↓
 ┌──────┼────────┐
 ▼      ▼        ▼
SQL    REST     Blob
```

This approach makes the pipeline easier to extend and maintain.

---

# 3. Incremental Loading

Incremental loading processes only new or changed data instead of repeatedly loading the entire source.

A typical watermark pattern is:

```text
Last Successful Watermark
          ↓
Source Query
          ↓
New / Changed Records
          ↓
Load Target
          ↓
Update Watermark
```

For example:

```sql
WHERE ModifiedDate > @LastLoadTimestamp
```

A control table can store:

```text
SourceName
SourceType
LastLoadTimestamp
IsActive
```

This allows the pipeline to determine what should be processed during the next run.

---

# 4. Mapping Data Flows

ADF Mapping Data Flow is used for transformation operations such as:

- Filtering invalid records
- Removing null keys
- Standardizing dates
- Casting data types
- Joining datasets
- Creating derived columns
- Aggregating data
- Writing processed datasets

Example:

```text
Orders
   │
   ▼
Filter
   │
   ▼
Derived Columns
   │
   ▼
Join Products
   │
   ▼
Join Stores
   │
   ▼
Aggregate
   │
   ▼
Processed Dataset
```

---

# 5. Pipeline Orchestration

Multiple pipelines can be connected through a master pipeline.

```text
PL_00_Master_Orchestrator
            │
            ▼
PL_01_Ingest
            │
         Success
            ▼
PL_02_Transform
            │
         Success
            ▼
PL_03_Load
```

The **Execute Pipeline** activity can be used to invoke child pipelines and coordinate their execution.

---

# 6. Triggers

The ADF work also covers different ways of starting pipelines.

### Schedule Trigger

Runs a pipeline according to a defined schedule.

```text
Schedule
   ↓
Pipeline
   ↓
ETL Processing
```

### Storage Event Trigger

Starts a pipeline when a file arrives in configured Azure Storage.

```text
New File
   ↓
Blob Event
   ↓
ADF Pipeline
   ↓
Processing
```

### Tumbling Window Trigger

Runs processing over defined recurring time windows and is useful for time-window-based batch processing.

---

# 7. Data Quality Validation

Data quality checks can be incorporated into the pipeline before declaring a successful run.

Example:

```sql
SELECT COUNT(*) AS RowCount
FROM SalesSummary;
```

The pipeline can then evaluate the result:

```text
Row Count
    │
    ▼
Meets Threshold?
   /       \
 Yes       No
  │         │
  ▼         ▼
Success    Fail
```

This prevents incomplete loads from silently reaching the reporting layer.

---

# 8. Failure Handling

ADF pipelines can use failure paths to trigger notification activities.

Example:

```text
Pipeline
   │
   ├── Success → Next Pipeline
   │
   └── Failure
          ↓
      Web Activity
          ↓
     Notification
```

Useful information for an operational notification includes:

- Pipeline name
- Pipeline run ID
- Failure time
- Failed activity
- Error information

---

# 🔐 Security

Credentials and secrets should **not** be hardcoded inside pipeline definitions.

Recommended pattern:

```text
Azure Key Vault
       │
       ▼
ADF Linked Service
       │
       ▼
Pipeline
       │
       ▼
Source / Target
```

When publishing this repository publicly, remove or parameterize:

- Passwords
- API keys
- Connection strings
- Access keys
- SAS tokens
- Webhook secrets
- Subscription-specific secrets

---

# 📊 Example End-to-End Retail Data Integration

One of the implementations documented in this Azure Data Engineering work is a **Retail Data Integration Platform**.

It integrates:

```text
Orders
  └── Azure SQL

Products
  └── Vendor REST API

Stores
  └── CSV / Blob Storage
```

The data is processed through:

```text
SQL + REST API + CSV
          ↓
        ADF
          ↓
       Bronze
          ↓
     Clean / Join
          ↓
       Silver
          ↓
      Aggregate
          ↓
        Gold
          ↓
    SalesSummary
          ↓
       Power BI
```

The documented implementation uses four main pipelines:

| Pipeline | Purpose |
|---|---|
| `PL_00_Master_Orchestrator` | Controls the complete workflow |
| `PL_01_Ingest_RawZone` | Ingests source data into Bronze |
| `PL_02_Transform_ProcessedZone` | Cleans, joins and aggregates data |
| `PL_03_Load_CuratedZone` | Loads/upserts curated data |

The documented Retail Data Integration Platform specifically uses a metadata-driven ingestion framework and a Bronze/Silver/Gold architecture, with the final curated `SalesSummary` dataset intended for reporting. 

---

# 🧠 Key ADF Concepts Covered

This folder is intended to demonstrate practical knowledge of:

### Data Integration

- Linked Services
- Datasets
- Copy Data
- REST API integration
- SQL integration
- Blob/ADLS integration

### Pipeline Development

- Lookup
- ForEach
- If Condition
- Switch
- Execute Pipeline
- Get Metadata
- Stored Procedure
- Web Activity
- Delete Activity
- Fail Activity
- Variables

### Data Transformation

- Mapping Data Flow
- Filter
- Derived Column
- Join
- Aggregate
- Sink

### Incremental Processing

- Watermarks
- Control tables
- Incremental queries
- Metadata-driven ingestion

### Operations

- Schedule triggers
- Storage event triggers
- Tumbling window triggers
- Pipeline monitoring
- Activity monitoring
- Failure handling
- Row-count validation

---

# 📁 Recommended Folder Structure

For a clean GitHub organization, the ADF directory can follow this structure:

```text
ADF/
│
├── README.md
│
├── Pipelines/
│   ├── PL_00_Master_Orchestrator.json
│   ├── PL_01_Ingest_RawZone.json
│   ├── PL_02_Transform_ProcessedZone.json
│   └── PL_03_Load_CuratedZone.json
│
├── Dataflows/
│   └── DF_CleanAndJoin.json
│
├── Datasets/
│   └── ...
│
├── LinkedServices/
│   └── ...
│
├── Triggers/
│   └── ...
│
├── SQL/
│   ├── Tables/
│   ├── StoredProcedures/
│   └── ValidationQueries/
│
└── Documentation/
    ├── Architecture/
    ├── Screenshots/
    └── ProjectGuides/
```

This type of separation is also commonly used in public ADF repositories to keep pipelines, datasets, dataflows, linked services, triggers and documentation organized. citeturn0search11turn0search8

---

# 🚀 How to Use This Folder

## Step 1 — Open Azure Data Factory

Open your Azure Data Factory instance through the Azure portal.

## Step 2 — Review Linked Services

Verify that the required source and target connections are available.

## Step 3 — Review Datasets

Check the datasets used by the pipelines and confirm their parameters and paths.

## Step 4 — Review Pipelines

Start with the master/orchestrator pipeline when one is available.

## Step 5 — Validate Parameters

Replace environment-specific values such as:

```text
<storage-account>
<container>
<database>
<server>
<api-url>
```

with your own Azure resources.

## Step 6 — Configure Credentials

Use Azure Key Vault or secure ADF connection mechanisms rather than committing secrets to GitHub.

## Step 7 — Run the Pipeline

Use ADF Debug/Trigger Now during development.

## Step 8 — Monitor the Run

Open:

```text
ADF Studio
   ↓
Monitor
   ↓
Pipeline Runs
   ↓
Activity Runs
```

Review successful and failed activities.

---

# 🧪 Testing Checklist

Before considering a pipeline complete, verify:

- [ ] Source connection works
- [ ] Target connection works
- [ ] Dataset paths are correct
- [ ] Pipeline parameters are populated
- [ ] Copy activity succeeds
- [ ] Data Flow succeeds
- [ ] Incremental logic returns the expected records
- [ ] Watermark is updated after successful ingestion
- [ ] Duplicate records are handled
- [ ] Row-count validation passes
- [ ] Failure path works
- [ ] Trigger executes correctly
- [ ] Pipeline run appears in Monitor
- [ ] No secrets are committed to GitHub

---

# 📚 Learning Outcomes

This ADF folder demonstrates practical experience with:

**Azure Cloud → Data Integration → ETL → Incremental Processing → Data Transformation → Orchestration → Monitoring**

It is intended as a portfolio/reference collection for Azure Data Engineering work.

---

# 🔒 Security Notice

This repository should contain **configuration and implementation examples, not secrets**.

Never commit:

```text
passwords
API keys
access keys
SAS tokens
client secrets
private keys
database connection strings containing credentials
webhook secrets
```

Use Azure Key Vault, Managed Identity or secure parameterization where appropriate.

## ⭐ Summary

This ADF folder demonstrates how **Azure Data Factory can be used to build reusable data integration pipelines**, from source ingestion through transformation, orchestration, validation and monitoring.

### Core flow

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
VALIDATE
  ↓
GOLD
  ↓
REPORTING
```

**Azure Data Factory | Azure SQL | ADLS Gen2 | Blob Storage | REST API | SQL | Mapping Data Flow | Incremental Loading | Pipeline Orchestration**

