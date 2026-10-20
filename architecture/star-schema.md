## Pharmaceutical Sales Star Schema

```mermaid
erDiagram
    DimDate ||--o{ FactSales : DateKey
    DimPharmacy ||--o{ FactSales : PharmacyID
    DimProduct ||--o{ FactSales : ProductID

    DimDate {
        int DateKey PK
        date Date
        int Year
        int Quarter
        int MonthNumber
        string MonthName
        string YearMonth
    }

    DimPharmacy {
        string PharmacyID PK
        string PharmacyName
        string Country
        string Region
        string City
        string PharmacyType
    }

    DimProduct {
        string ProductID PK
        string ProductName
        string Category
        string Brand
        decimal ListPriceEUR
        decimal StandardCostEUR
    }

    FactSales {
        string SalesID PK
        int DateKey FK
        string PharmacyID FK
        string ProductID FK
        int UnitsSold
        decimal RevenueEUR
        decimal CostEUR
        decimal MarginEUR
        string PromoFlag
    }
```

**Model design:** FactSales is the central transaction table. DimDate, DimPharmacy and DimProduct provide descriptive attributes for analysis. Each dimension has a one-to-many relationship with FactSales, using single-direction filtering.

**Security consideration:** PharmacyID is pseudonymised consistently in the curated fact and pharmacy dimension tables so that their relationship can be maintained.

*The diagram shows logical data types for illustration; authoritative Spark types must be verified in Fabric.*
