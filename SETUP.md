# Setup and Reproduction Guide

> **Independent NHS-inspired portfolio POC.** Use only fictional, self-created or otherwise authorised data and infrastructure. This guide documents how to reproduce the design; it does **not** mean the pipeline or notebook has been executed in a new environment.

## 1. Prerequisites

- Access to a Microsoft Fabric-enabled workspace with permissions to create a Lakehouse, Data Pipeline and Notebook, and run Spark workloads.
- Four compatible **synthetic** source datasets and permission to use them.
- A securely managed HMAC key available to the notebook runtime as `PHARMACY_HMAC_KEY`. **Never commit or print the key.**
- Power BI Desktop if you wish to recreate the semantic model and report.
- Familiarity with Fabric Data Pipelines, Lakehouse tables, Spark notebooks and Power BI.

**Hosting note:** Fabric capacity/trial access and Power BI publishing eligibility depend on your tenant and licence. The public dashboard in this repository is a separate portfolio demonstration, not a live connection to the POC Lakehouse.

## 2. Source datasets and schema

Prepare four files with the following base names and consistent foreign keys:

| Source dataset | Required key | Raw Lakehouse table |
| --- | --- | --- |
| `PharmacyDimDate` | `DateKey` | `dbo.PharmacyDimDate` |
| `PharmacyDimProduct` | `ProductID` | `dbo.PharmacyDimProduct` |
| `PharmacyDimPharmacy` | `PharmacyID` | `dbo.PharmacyDimPharmacy` |
| `PharmacyFactSales` | `SalesID`; foreign keys `DateKey`, `ProductID`, `PharmacyID` | `dbo.PharmacyFactSales` |

Use the [data dictionary](data/data-dictionary.md) for documented fields and business definitions. Its example counts and types are not guaranteed to match your own synthetic data.

**Source format distinction:** The documented [CSV ingestion pipeline](pipelines/ingestion-pipeline.md) uses `.csv` source files. A separate successful screenshot shows an activity named `Copy_Excel_To_Table`, but its configuration is not fully exported in this repository. The steps below reproduce the **documented CSV variant**; do not assume the same Copy Data source settings work for `.xlsx` files.

## 3. Create the Fabric workspace and Lakehouse

1. Open Microsoft Fabric and choose an authorised Fabric-enabled workspace.
2. Create a Lakehouse, e.g. `LH_Pharmacy_Portfolio`. The original POC documentation uses `LH_PLICS_POC`; that name is not required for a new deployment.
3. Upload the four synthetic `.csv` files to the Lakehouse **Files** area, using these exact filenames:
   - `PharmacyDimDate.csv`
   - `PharmacyDimProduct.csv`
   - `PharmacyDimPharmacy.csv`
   - `PharmacyFactSales.csv`
4. Check that the files are readable and have the expected header rows and delimiter.

**Do not** publish tenant IDs, SQL endpoints, workspace IDs, internal URLs or connection strings as part of your reproduction.

## 4. Configure the ingestion pipeline

Create a Fabric Data Pipeline with a **ForEach** activity and a **Copy Data** activity inside it.

**ForEach settings**

- Sequential execution: enabled.
- Items expression:

```text
@createArray('PharmacyDimDate', 'PharmacyDimProduct', 'PharmacyDimPharmacy', 'PharmacyFactSales')
```

**Copy Data source**

- Connection: your new Lakehouse.
- Root: `Files`.
- Format: delimited text / CSV.
- Dynamic filename:

```text
@concat(item(),'.csv')
```

**Copy Data destination**

- Connection: the same Lakehouse.
- Root: `Tables`.
- Schema: `dbo`.
- Dynamic table name:

```text
@item()
```

- Table action: **Overwrite** (full refresh, **not** incremental ingestion).
- Confirm headers, type mapping and CSV parsing against your actual files before running.

Run the pipeline's ingestion portion and verify that all four `dbo` tables exist and their row counts match the source files. The existing [pipeline documentation](pipelines/ingestion-pipeline.md) describes the expressions and orchestration.

## 5. Configure and run the PySpark notebook

1. Import or recreate the [sanitised notebook](notebooks/NB_NHS_PHARMACY_%20PSEUDONYMISATION.ipynb) in the same authorised Fabric workspace.
2. Attach the notebook to the Lakehouse that contains the four raw tables.
3. Ensure the target schemas `psy` and `metadata` are available in the Lakehouse. Create them using the appropriate Fabric-supported method if absent.
4. Arrange for `PHARMACY_HMAC_KEY` to be available **securely to the notebook's Python process** before the first code cell runs. The published code reads `os.environ.get("PHARMACY_HMAC_KEY")`. **It does not implement Azure Key Vault retrieval itself.** Your Fabric runtime may require additional approved integration to supply the variable; do not assume a local shell environment variable is automatically propagated to Spark.
5. Run the notebook cells in order. It reads `dbo.PharmacyDimPharmacy` and `dbo.PharmacyFactSales`, then writes:
   - `psy.PharmacyDimPharmacy_Curated`
   - `psy.PharmacyFactSales_Curated`
   - `metadata.PseudonymisationProcessingLog` (sales-fact processing log only)
6. Verify that the output tables exist and that no unhandled exception occurred.

The notebook **overwrites** curated Delta tables and **appends** audit-log records on each successful execution. Its identifier mapping uses driver-side `collect()` and is appropriate only for small POC datasets.

**Notebook dependency:** The notebook uses Fabric-provided `spark` and `display`. It is not a standalone Python script that can be run without a compatible Spark/Fabric runtime.

## 6. Orchestrate ingestion followed by pseudonymisation

Add a Notebook activity (e.g. `Run_pseudonymisation`) to the pipeline and connect it to the ForEach activity with an **On success** dependency.

Expected sequence:

```text
Four source CSVs
      |
      v
ForEach (sequential)
  Copy Data -> dbo tables
      |
      | on success
      v
Run_pseudonymisation
      |
      v
psy curated tables + metadata processing log
```

Run the complete pipeline and check each Copy Data iteration and the notebook activity in the monitoring output. A successful run in the original POC is documented with a [pipeline screenshot](pipelines/fabric-pipeline-execution.png), but that screenshot is not evidence of a run in your own environment.

## 7. Validate the output

At minimum, verify:

1. All four raw tables were loaded and have expected row counts.
2. Curated pharmacy and sales tables have the expected row counts and columns.
3. The same fictional `PharmacyID` generates the same pseudonym in both curated tables.
4. All fact `PharmacyID` values expected to join have a matching dimension key; check unmatched values explicitly.
5. Original identifiers are absent from fields intended to be pseudonymised; check both standalone and embedded occurrences.
6. The processing log has the expected `RunID`, `SourceTable`, `ColumnName`, `ReplacementsCount` and `ProcessedAt` fields.
7. Re-running does not unexpectedly change pseudonyms when the key and normalisation are unchanged.

See [pseudonymisation validation](docs/pseudonymisation-validation.md) and the [implementation walkthrough](notebooks/pseudonymisation-implementation.md).

**Important limitations:** The notebook's current inner-join check measures overlapping distinct keys, **not** complete referential integrity. Its regex matching is not boundary-aware, and audit logging covers only the sales fact. Treat the above as required manual verification until automated tests are added.

## 8. Build the Power BI semantic model

Create/import four reporting queries:

- `FactSales` from `psy.PharmacyFactSales_Curated`
- `DimPharmacy` from `psy.PharmacyDimPharmacy_Curated`
- `DimDate` from `dbo.PharmacyDimDate`
- `DimProduct` from `dbo.PharmacyDimProduct`

Configure single-direction, one-to-many dimension-to-fact relationships:

| Dimension | Fact |
| --- | --- |
| `DimDate[DateKey]` | `FactSales[DateKey]` |
| `DimProduct[ProductID]` | `FactSales[ProductID]` |
| `DimPharmacy[PharmacyID]` | `FactSales[PharmacyID]` |

Create reporting measures and visuals appropriate to your data. Consult the [semantic model documentation](powerbi/semantic-model.md). The public [interactive dashboard](https://app.fabric.microsoft.com/view?r=eyJrIjoiY2FjM2IxMDMtNDY5OS00NDk5LTg4ZWQtMzYxODZhNzJkNzBkIiwidCI6IjYyODhhOGI0LWYxN2ItNGMzOC04ODkzLTE1MTNkMmM5MTVmZSJ9&embedImagePlaceholder=true) is an **independently hosted, synthetic-data demonstration**, not a reproducible deployment artifact from this guide.

## 9. Troubleshooting

| Symptom | Check |
| --- | --- |
| File not found | Lakehouse **Files** path, extension and exact filename; `.csv` and `.xlsx` require different source configurations |
| Missing column in Power Query | Compare actual source headers to transformation steps; do not assume columns such as `Month` or `SubCategory` exist |
| `PHARMACY_HMAC_KEY` missing | Verify approved secret injection into the notebook process; do not paste a real key into source code |
| `psy` or `metadata` write fails | Verify target schema existence and Lakehouse permissions |
| Broken Power BI relationship | Confirm dimension keys are unique, types match, and fact/dimension pseudonyms were generated with the same key and normalisation |
| Unexpected identifier replacement | Review regex matching: the current POC matches substrings, not identifier boundaries |

## 10. Security, ownership and scope

- Use **fictional** data only, with rights to publish and process it.
- Do not copy employer-owned code, report designs, workspace details or data into a personal environment without authorisation.
- Keep raw data and HMAC secrets access-controlled; pseudonymisation is **not anonymisation**.
- Do not use **Publish to web** for confidential or restricted datasets: public reports and their exposed data can be accessed by anyone.
- This guide is for a **proof of concept**, not a production security deployment.

## 11. Related documentation

- [Architecture overview](architecture/architecture-overview.md)
- [End-to-end data flow](architecture/data-flow.md)
- [Ingestion pipeline](pipelines/ingestion-pipeline.md)
- [PySpark implementation with code](notebooks/pseudonymisation-implementation.md)
- [Data dictionary](data/data-dictionary.md)
- [Security design evaluation](docs/security-design-evaluation.md)
- [Power BI semantic model](powerbi/semantic-model.md)
