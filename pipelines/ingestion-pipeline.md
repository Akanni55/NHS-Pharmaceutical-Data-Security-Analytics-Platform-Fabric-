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

## 4. Parameterised Ingestion Design

The pipeline design supports reusable ingestion logic by dynamically supplying source file names and destination table names.

This reduces the need to create and maintain separate Copy Data activities for every dataset.

The target design includes:

1. A list of source datasets.
2. A ForEach activity to iterate through the list.
3. A Copy Data activity inside the loop.
4. Dynamic source file and destination table configuration.
5. Pipeline execution monitoring and error handling.

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
