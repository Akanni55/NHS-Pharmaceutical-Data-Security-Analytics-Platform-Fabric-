# Solution Architecture

## 1. Architecture Overview

This project implements an end-to-end pharmaceutical data engineering and analytics solution using Microsoft Fabric and Power BI.

The architecture separates raw data ingestion, secure transformation, curated storage and analytical reporting.

The solution uses a Lakehouse-based architecture to support scalable data processing, consistent pseudonymisation and controlled access to pharmaceutical analytics.

## 2. Data Flow

```text
Pharmaceutical Source Data
            |
            v
Microsoft Fabric Data Pipeline
            |
            v
Lakehouse - Raw Layer (dbo)
            |
            v
PySpark Transformation Notebook
    | HMAC-SHA-256 Pseudonymisation
    | Cross-Column Identifier Detection
    | Embedded Identifier Replacement
            |
            +------> Metadata Processing Log
            |        (metadata schema)
            v
Lakehouse - Curated Layer (psy)
            |
            v
Power BI Semantic Model
            |
            v
Power BI Reports and App
            |
            v
Authorised Reporting Stakeholders
```

## 3. Architecture Components

| Component | Purpose |
|---|---|
| Fabric Data Pipeline | Automates ingestion of source datasets into the Lakehouse |
| Raw Layer (`dbo`) | Stores original source data for controlled processing |
| PySpark Notebook | Performs deterministic HMAC-SHA-256 pseudonymisation |
| Curated Layer (`psy`) | Stores transformed reporting-ready data |
| Metadata Layer (`metadata`) | Records pseudonymisation processing activity |
| Power BI Semantic Model | Maintains relationships, calculations and reporting logic |
| Power BI App | Provides controlled report distribution |

## 4. Data Security Design

The solution applies pseudonymisation before data reaches the reporting layer.

The transformation process generates consistent HMAC-SHA-256 pseudonyms for sensitive identifiers across fact and dimension tables.

The notebook also scans string columns for sensitive identifiers that may appear as standalone values or embedded within other text.

Detected identifiers are replaced using the same pseudonymisation mapping.

The curated tables preserve relational consistency without retaining original sensitive identifiers in the transformed fields.

Access to the raw layer must be restricted separately.

## 5. Data Processing and Monitoring

Processing activity is recorded in a dedicated metadata table.

The intended log records the source table, processed column, number of identifier replacements, execution timestamp and processing run identifier.

This provides visibility into the pseudonymisation process and supports operational monitoring and validation.

## 6. Reporting Layer

Power BI connects to the curated Lakehouse tables through the Fabric SQL analytics endpoint.

The semantic model preserves relationships between pharmaceutical transactions, pharmacies, products and dates.

Reports provide pharmaceutical analytics while limiting exposure of sensitive identifiers.

Power BI App audiences and security groups can be used to manage stakeholder access.

## 7. Architecture Benefits

- Separation of raw and curated data.
- Automated ingestion and transformation.
- Consistent pseudonymisation across related datasets.
- Detection of sensitive identifiers across multiple columns.
- Preserved analytical relationships.
- Centralised processing metadata.
- Controlled Power BI reporting access.

## 8. Implementation Note

This is a personal portfolio demonstration using dummy pharmaceutical data. Production deployment would require secure secret management, scalable identifier matching, validation controls and an approved Information Governance assessment.
