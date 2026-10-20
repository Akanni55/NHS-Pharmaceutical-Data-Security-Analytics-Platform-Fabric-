# Microsoft Fabric Data Ingestion Pipeline

## 1. Overview

This component demonstrates automated ingestion of pharmaceutical source datasets into a Microsoft Fabric Lakehouse using Fabric Data Pipelines.

The solution uses a parameterised approach to reduce repetitive development and simplify the ingestion of multiple datasets.

## 2. Business Requirement

The pharmaceutical analytics platform requires four core datasets:

- Pharmacy sales transactions
- Pharmacy information
- Product information
- Calendar dates

These datasets are ingested into the raw Lakehouse layer before downstream transformation and reporting.

## 3. Pipeline Architecture

**Source Data → Fabric Data Pipeline → Lakehouse Raw Layer (`dbo`) → PySpark Transformation**

The ingestion process is designed to preserve source information before pseudonymisation and analytical transformations.

## 4. Pipeline Implementation

The Microsoft Fabric pipeline orchestrates source data ingestion and downstream pseudonymisation within a single workflow.

#### ForEach Configuration

The `ForEach_File_available` activity is configured with sequential execution enabled.

**Items expression:**

```text
@createArray('PharmacyDimDate', 'PharmacyDimProduct', 'PharmacyDimPharmacy', 'PharmacyFactSales')
```

The expression defines four source datasets to be processed by the reusable `Copy_each_file_into_table` activity.

Each iteration supplies one dataset name to the Copy Data activity. Sequential execution ensures that the iterations run one after another.

This design avoids duplicating Copy Data activities and makes it easier to extend the pipeline with additional datasets.

#### Dynamic Source and Destination Configuration

The `Copy_each_file_into_table` activity uses dynamic expressions to ingest multiple CSV files into separate Microsoft Fabric Lakehouse tables.

**Source configuration**

- Connection: `LH_PLICS_POC`
- Root folder: `Files`
- File path type: `File path`
- File format: `DelimitedText`
- Recursive search: Enabled

Dynamic source filename:

```text
@concat(item(),'.csv')
```

**Destination configuration**

- Connection: `LH_PLICS_POC`
- Root folder: `Tables`
- Schema: `dbo`
- Table action: `Overwrite`

Dynamic destination table name:

```text
@item()
```

**Source-to-destination mapping**

| Source file | Lakehouse table |
|---|---|
| `PharmacyDimDate.csv` | `dbo.PharmacyDimDate` |
| `PharmacyDimProduct.csv` | `dbo.PharmacyDimProduct` |
| `PharmacyDimPharmacy.csv` | `dbo.PharmacyDimPharmacy` |
| `PharmacyFactSales.csv` | `dbo.PharmacyFactSales` |

The ForEach activity processes each dataset sequentially. The source expression appends `.csv` to the current item, while the destination expression uses the same item as the target table name.

The overwrite configuration replaces the destination table contents during each ingestion run. This is a full-refresh ingestion pattern rather than incremental loading.

### 4.3 Notebook Execution

**Activity name:** `Run_pseudonymisation`

A Microsoft Fabric Notebook activity is connected to the ForEach activity using a success dependency.

This ensures that the pseudonymisation notebook is triggered only after the ForEach activity completes successfully.

The notebook forms the transformation stage of the architecture, preparing pseudonymised datasets for downstream analytics.

### 4.4 Pipeline Execution Evidence

A successful pipeline execution was recorded on **20 October 2026**.

The execution screenshot confirms:

- Overall pipeline status: Succeeded.
- ForEach activity status: Succeeded.
- ForEach activity duration: 1 minute 46 seconds.
- Multiple successful Copy Data executions.
- A downstream notebook activity connected through a success dependency.

![Successful Fabric Pipeline Execution](fabric-ingestion-pipeline.png)

### 4.5 Implementation Status

The pipeline orchestration and successful execution have been verified through Fabric monitoring evidence.

Further documentation will cover the exact parameter array, dynamic expressions, source and destination mappings, and pseudonymisation notebook configuration.

## 5. Raw Lakehouse Tables

| Source Dataset | Destination Table |
|---|---|
| Pharmacy Fact Sales | `dbo.PharmacyFactSales` |
| Pharmacy Dimension | `dbo.PharmacyDimPharmacy` |
| Product Dimension | `dbo.PharmacyDimProduct` |
| Date Dimension | `dbo.PharmacyDimDate` |

## 6. Data Quality Considerations

The ingestion process should validate:

- Source file availability and naming conventions.
- Successful table creation and schema mapping.
- Record counts between source and destination.
- Data type compatibility.
- Pipeline execution status and failures.

## 7. Implementation Evidence

The following evidence will be added after reviewing the completed Fabric pipeline:

- Pipeline activity diagram.
- ForEach configuration.
- Dynamic expressions.
- Copy Data source and destination configuration.
- Successful execution results.
- Error handling and troubleshooting notes.

## 8. Security Considerations

Source datasets must contain synthetic or appropriately authorised data.

Raw Lakehouse tables should have restricted access, and sensitive identifiers must be transformed before use in the curated reporting layer.

No connection strings, authentication tokens, secrets or internal organisational details should be published in this repository.

---

**Implementation status:** This document describes the intended ingestion pattern. Pipeline configuration and successful execution evidence will be added after verification.
