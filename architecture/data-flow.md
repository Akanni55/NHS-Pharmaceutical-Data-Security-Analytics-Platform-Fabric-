## End-to-End Data Flow

```mermaid
flowchart TD
    A["Pharmaceutical Source Data"] --> B["Microsoft Fabric Data Pipeline"]
    B --> C["Raw Lakehouse Tables - dbo"]
    C --> D["PySpark Transformation Notebook"]
    D --> E["HMAC-SHA-256 Pseudonymisation"]
    E --> F["Cross-Column Identifier Detection"]
    F --> G["Curated Delta Tables - psy"]
    F --> H["Processing Audit Log - metadata"]
    G --> I["Power BI Semantic Model"]
    I --> J["Power BI Reports"]
    J --> K["Power BI App - Stakeholder Access"]
```
