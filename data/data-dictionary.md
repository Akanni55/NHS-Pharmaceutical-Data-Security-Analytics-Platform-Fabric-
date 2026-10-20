# Data Dictionary

## Dataset overview

This personal portfolio project uses a fictional European pharmacy-chain distributor dataset to demonstrate Microsoft Fabric ingestion, pseudonymisation, data modelling and Power BI reporting. It is **not NHS data** and is **not affiliated with the NHS**.

The record counts below are supplied in the source workbook and have not been independently verified against the current Lakehouse.

| Source table | Rows (source workbook) | Primary key |
|---|---:|---|
| DimDate | 731 | `DateKey` |
| DimPharmacy | 120 | `PharmacyID` |
| DimProduct | 220 | `ProductID` |
| FactSales | 62,139 | `SalesID` |

## Source-to-Lakehouse table mapping

| Dataset table | Fabric raw table | Curated table (where applicable) |
|---|---|---|
| DimDate | `dbo.PharmacyDimDate` | Not specified |
| DimPharmacy | `dbo.PharmacyDimPharmacy` | `psy.PharmacyDimPharmacy_Curated` |
| DimProduct | `dbo.PharmacyDimProduct` | Not specified |
| FactSales | `dbo.PharmacyFactSales` | `psy.PharmacyFactSales_Curated` |

The workbook documents business definitions and example values, **not authoritative Spark/SQL data types or nullability**. Verify those separately against the Lakehouse before adding schema types.

## DimDate

| Column | Example value | Business definition |
|---|---|---|
| `DateKey` | 20250115 | A unique key for each day. |
| `Date` | 2025-01-15 | The calendar date. |
| `Year` | 2025 | Year number. |
| `Quarter` | 1 | Quarter number (1–4). |
| `MonthNumber` | 1 | Month number (1–12). |
| `MonthName` | January | Month name. |
| `YearMonth` | 2025-01 | Month bucket label. |

## DimPharmacy

| Column | Example value | Business definition |
|---|---|---|
| `PharmacyID` | PH0042 | Unique store ID. |
| `PharmacyName` | Paris HealthPoint #042 | Store display name. |
| `Country` | France | Country where the store is. |
| `Region` | Île-de-France | Region inside a country. |
| `City` | Paris | City where the store is. |
| `PharmacyType` | Urban | Whether the store is urban/suburban/rural. |
| `OpenDate` | 2019-06-10 | When the store opened. |
| `StoreSizeBand` | M | Small/Medium/Large store size. |
| `Latitude` | 48.8571 | Map coordinate (north/south). |
| `Longitude` | 2.3509 | Map coordinate (east/west). |

## DimProduct

| Column | Example value | Business definition |
|---|---|---|
| `ProductID` | PR0107 | Unique product ID. |
| `ProductName` | Medica Ibuprofen 400 mg | Display name of the product. |
| `Category` | OTC | High-level product group. |
| `Brand` | Medica | Brand family. |
| `IsGeneric` | Yes | Whether product is generic. |
| `PackSize` | 20 tablets | Package format. |
| `ListPriceEUR` | 12.99 | Base list price (before country/promo effects). |
| `StandardCostEUR` | 9.1 | Base cost (before country effects). |
| `LaunchDate` | 2020-03-15 | When the product became available. |
| `IsDiscontinued` | No | Whether product has been discontinued. |
| `DiscontinuedDate` | 2025-05-01 | Date after which product is no longer sold. |

## FactSales

| Column | Example value | Business definition |
|---|---|---|
| `SalesID` | S0001234 | Unique row ID for a sales record. |
| `DateKey` | 20250115 | The day of the sale. |
| `PharmacyID` | PH0042 | Which store sold it. |
| `ProductID` | PR0107 | Which product was sold. |
| `UnitsSold` | 6 | How many units were sold in that row. |
| `RevenueEUR` | 54.3 | Total revenue for that row. |
| `CostEUR` | 40.1 | Total cost for that row. |
| `MarginEUR` | 14.2 | Profit for that row (Revenue − Cost). |
| `PromoFlag` | Yes | Whether the row was sold on promo. |

## Data model relationships

| Dimension key | Fact foreign key | Cardinality | Filter direction |
|---|---|---|---|
| `DimDate.DateKey` | `FactSales.DateKey` | 1 to many | Single |
| `DimPharmacy.PharmacyID` | `FactSales.PharmacyID` | 1 to many | Single |
| `DimProduct.ProductID` | `FactSales.ProductID` | 1 to many | Single |

## Suggested reporting hierarchies

- Geography: Country → Region → City → PharmacyName
- Product: Category → Brand → ProductName
- Time: Year → Quarter → MonthName → Date

## Data quality and business rules

- Coordinates are WGS84 latitude/longitude with slight jitter for map visualisation.
- `MarginEUR` = `RevenueEUR` − `CostEUR`, rounded to two decimal places.
- Pharmacies opened during 2024–2025 should have no sales before their `OpenDate`.
- Discontinued products should have no sales after their `DiscontinuedDate`.

## Security and pseudonymisation

- `PharmacyID` is the demonstrator identifier used for deterministic HMAC-SHA-256 pseudonymisation across fact and pharmacy dimension tables. It is a fictional store identifier, **not a real NHS number or patient identifier**.
- The POC also tests detection and replacement of identifiers embedded within other text fields.
- Pseudonymisation does not, by itself, guarantee anonymity. Production use requires secure key management, governance and testing.
- Processing-log design and implementation status should be validated before claiming production-ready auditing.

## Source and limitations

- Prepared from the supplied `Data Dictionary.xlsx` workbook (sheets: Data Dictionary and README).
- The workbook does not specify a public redistribution licence; confirm rights before publishing the original Excel file or underlying row-level data.
- Examples are illustrative, not evidence of actual records in the deployed Lakehouse.
