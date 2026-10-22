# PySpark Pseudonymisation Implementation

> **Portfolio demonstration:** NHS-inspired pharmaceutical analytics using fictional data. This project is independent of, and not endorsed by, the NHS. No real patient records are included.

## 1. Overview

This page combines the explanatory documentation with the **complete Python source code from the public, sanitised notebook**, in its original execution order. It demonstrates deterministic HMAC-SHA-256 pseudonymisation in Microsoft Fabric PySpark, transforming raw Lakehouse tables into curated Delta tables while preserving joinable pharmacy identifiers.

**[View the original Jupyter notebook](NB_NHS_PHARMACY_%20PSEUDONYMISATION.ipynb)** — the code blocks below are copied verbatim from its four code cells; only the surrounding Markdown commentary has been added.

## 2. Source and destination tables

| Raw source | Curated output |
| --- | --- |
| `dbo.PharmacyDimPharmacy` | `psy.PharmacyDimPharmacy_Curated` |
| `dbo.PharmacyFactSales` | `psy.PharmacyFactSales_Curated` |

The Date and Product dimensions are not pseudonymised by this notebook. The notebook uses `spark` and `display` as provided by a compatible Microsoft Fabric notebook runtime. Its public version requires an appropriately configured secret and existing tables/schemas before execution.


## 3. Imports, secret configuration and deterministic HMAC

The notebook imports Python cryptography libraries and Spark UDF helpers, reads `PHARMACY_HMAC_KEY` from the runtime environment, and stops if no key is configured. The `pseudonymise_id` function normalises input by removing spaces and hyphens before calculating an HMAC-SHA-256 digest. The function is registered as a Spark UDF. The key is deliberately **not** committed to GitHub.

```python
# 1. PSEUDONYMISATION SETUP

import hmac
import hashlib
from pyspark.sql.functions import udf, col
from pyspark.sql.types import StringType

# TEST ONLY - production secret will come from Azure Key Vault
import os
SECRET_KEY = os.environ.get("PHARMACY_HMAC_KEY")
if not SECRET_KEY:
    raise RuntimeError("Set PHARMACY_HMAC_KEY securely before running this notebook")

def pseudonymise_id(value):
    if value is None:
        return None

    # Standardise the identifier before pseudonymising
    cleaned = str(value).replace(" ", "").replace("-", "").strip()

    return hmac.new(
        SECRET_KEY.encode("utf-8"),
        cleaned.encode("utf-8"),
        hashlib.sha256
    ).hexdigest()

# Make the function available to Spark
pseudonymise_udf = udf(pseudonymise_id, StringType())
```


## 4. Pharmacy dimension: discovery, replacement and Delta output

The dimension stage reads `dbo.PharmacyDimPharmacy`, creates an original-to-pseudonym mapping from distinct non-null `PharmacyID` values, and gathers it into a Python dictionary. A regular expression searches every string column for known identifiers, including IDs embedded within longer text. The transformed data is written to `psy.PharmacyDimPharmacy_Curated` in Delta overwrite mode.

```python
# PSEUDONYMISE PHARMACY DIM_PHARMACY (Including standalone and embedded PharmacyIDs across all string columns)

from pyspark.sql.functions import col, udf
from pyspark.sql.types import StringType
import re

# 1. Reading my original table
df_pharmacy = spark.read.table("dbo.PharmacyDimPharmacy")

# 2. Create a mapping of original IDs to HMAC value and Reuse existing HMAC UDF for consistency
df_id_mapping = (
    df_pharmacy
    .select("PharmacyID")
    .where(col("PharmacyID").isNotNull())
    .distinct()
    .withColumn("PseudoID", pseudonymise_udf(col("PharmacyID")))
)

# Convert the mapping into a Python dictionary
id_mapping = {
    row["PharmacyID"]: row["PseudoID"]
    for row in df_id_mapping.collect()
}

# 3. Create pattern to detect IDs anywhere within text, Longest IDs first to avoid overlapping matches
if id_mapping:

    pattern = re.compile(
        "|".join(
            re.escape(k)
            for k in sorted(id_mapping, key=len, reverse=True)
        )
    )

    def replace_pharmacy_ids(value):
        if value is None:
            return None

        return pattern.sub(
            lambda match: id_mapping[match.group(0)],
            value
        )

    replace_ids_udf = udf(replace_pharmacy_ids, StringType())

    # 4. Scan every string column
    df_pharmacy_curated = df_pharmacy


    for field in df_pharmacy.schema.fields:
        if isinstance(field.dataType, StringType):
            df_pharmacy_curated = df_pharmacy_curated.withColumn(
                field.name,
                replace_ids_udf(col(field.name))
            )

else:
    df_pharmacy_curated = df_pharmacy

# 5. Save curated table
df_pharmacy_curated.write \
    .format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable("psy.PharmacyDimPharmacy_Curated")

# 6. Display results
display(df_pharmacy_curated)
```


## 5. Sales fact: replacement, curated output and audit log

The fact stage reads `dbo.PharmacyFactSales`, constructs a mapping using the same HMAC function, and replaces matching identifiers across string columns. It writes `psy.PharmacyFactSales_Curated` as a Delta table. This cell also creates a unique processing run ID and UTC timestamp, counts regex matches in the original fact-table columns, and appends a per-column processing summary to `metadata.PseudonymisationProcessingLog`. **Audit logging is implemented for Fact Sales only**, not the pharmacy dimension.

```python
# PSEUDONYMISE PHARMACY FACT_SALES (Including standalone and embedded PharmacyIDs across all string columns)

from pyspark.sql.functions import col, udf
from pyspark.sql.types import StringType
import re

# 1. Reading my original Fact Sales table
df_sales = spark.read.table("dbo.PharmacyFactSales")

# 2. Create a mapping of original IDs to HMAC value and Reuse existing HMAC UDF for consistency
df_id_mapping = (
    df_sales
    .select("PharmacyID")
    .where(col("PharmacyID").isNotNull())
    .distinct()
    .withColumn("PseudoID", pseudonymise_udf(col("PharmacyID")))
)

# Convert the mapping into a Python dictionary
id_mapping = {
    row["PharmacyID"]: row["PseudoID"]
    for row in df_id_mapping.collect()
}

# 3. Create pattern to detect IDs anywhere within text, Longest IDs first to avoid overlapping matches
if id_mapping:

    pattern = re.compile(
        "|".join(
            re.escape(k)
            for k in sorted(id_mapping, key=len, reverse=True)
        )
    )

    def replace_pharmacy_ids(value):
        if value is None:
            return None

        return pattern.sub(
            lambda match: id_mapping[match.group(0)],
            value
        )

    replace_ids_udf = udf(replace_pharmacy_ids, StringType())

    # 4. Scan every string column
    df_sales_curated = df_sales

    for field in df_sales.schema.fields:
        if isinstance(field.dataType, StringType):
            df_sales_curated = df_sales_curated.withColumn(
                field.name,
                replace_ids_udf(col(field.name))
            )

else:
    df_sales_curated = df_sales

# 5. Save curated table
df_sales_curated.write \
    .format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable("psy.PharmacyFactSales_Curated")

# 6. PROCESSING AUDIT LOG - FACT SALES

from pyspark.sql import Row
from pyspark.sql.types import StringType
from datetime import datetime, timezone
import uuid

# Unique ID and timestamp for this processing run
run_id = str(uuid.uuid4())
processed_at = datetime.now(timezone.utc)

log_rows = []

# Scan each original string column and count actual ID occurrences
for field in df_sales.schema.fields:

    column_name = field.name
    replacement_count = 0

    if isinstance(field.dataType, StringType) and id_mapping:

        # Count every matching PharmacyID occurrence
        values = df_sales.select(column_name).toLocalIterator()

        for row in values:
            value = row[column_name]

            if value is not None:
                replacement_count += len(
                    pattern.findall(value)
                )

    log_rows.append(
        Row(
            RunID=run_id,
            SourceTable="dbo.PharmacyFactSales",
            ColumnName=column_name,
            ReplacementsCount=replacement_count,
            ProcessedAt=processed_at
        )
    )

# 7. Create audit DataFrame
df_processing_log = spark.createDataFrame(log_rows)

# 8. Append log records rather than overwriting previous runs
df_processing_log.write \
    .format("delta") \
    .mode("append") \
    .saveAsTable("metadata.PseudonymisationProcessingLog")

# 9. Display processing summary
display(df_processing_log)

# 10. Display curated data
display(df_sales_curated)
```


## 6. Relationship validation

The final code cell selects distinct pseudonymised `PharmacyID` values from both curated tables and performs an inner join. It prints the number of matching distinct IDs and displays the matches. This checks key overlap, but is **not** a complete referential-integrity or data-quality test.

```python
# VALIDATE THAT THE PSEUDONYMISED KEY STILL JOINS AS A RELATIONSHIP

validation = (
    df_sales_curated
    .select("PharmacyID")
    .distinct()
    .join(
        df_pharmacy_curated.select("PharmacyID").distinct(),
        on="PharmacyID",
        how="inner"
    )
)

print("Matching pseudonymised PharmacyIDs across both tables:")
print(validation.count())

display(validation)
```


## 7. Processing audit: what is recorded

The Fact Sales stage writes one audit row per source column, including `RunID`, `SourceTable`, `ColumnName`, `ReplacementsCount` and `ProcessedAt`. The `ReplacementsCount` value is calculated by searching the **original** string-column values for known pharmacy identifiers; it represents counted regex occurrences, not an independently reconciled count of persisted changes. The audit Delta table uses append mode so multiple runs can be retained.

## 8. Security considerations

- **Deterministic, not anonymous:** the same normalised identifier and HMAC secret yield the same pseudonym. Pseudonymisation does not make the data anonymous or remove governance obligations.
- **No committed key:** the notebook expects `PHARMACY_HMAC_KEY` from the environment. In production, retrieve secrets from an approved secrets-management mechanism such as Azure Key Vault, with access controls and rotation procedures.
- **Raw data restrictions:** access to raw Lakehouse tables and mapping material must remain appropriately restricted. Pseudonymisation complements, rather than replaces, least-privilege access and data minimisation.
- **Embedded identifiers:** the regex matches known identifiers wherever they occur in string values. It is not boundary-aware, so it can replace matching substrings inside unrelated strings; further safeguards would be needed for production.

## 9. POC limitations and improvements

1. **Driver-side mapping:** `collect()` constructs the ID mapping on the Spark driver. This is suitable for the small POC dataset, not large-scale distributed processing.
2. **Separate mappings:** dimension and fact mappings are built independently from each table's observed identifiers. The shared HMAC function makes matching source IDs consistent, but a production design should use a validated authoritative identifier domain.
3. **Normalisation versus matching:** the HMAC function removes spaces and hyphens before hashing, but the regex looks for the literal original IDs gathered from each table. Variants embedded in free text may therefore remain undetected.
4. **Partial auditing:** the dimension stage does not create processing audit records. The fact stage iterates through original values on the driver to count matches, which may be costly at scale.
5. **Validation coverage:** the inner join confirms overlapping distinct pseudonymised keys. It does not detect unmatched fact IDs, duplicate dimension IDs, null keys, unchanged embedded identifiers or unintended replacements. These should become explicit automated tests.
6. **Repeatability:** curated tables are overwritten each run; audit rows are appended. Production orchestration should define run-level failure handling, idempotency and retention.

## 10. Implementation status

The notebook implements HMAC-SHA-256 pseudonymisation, cross-column regex-based identifier replacement, Delta curated table writes, Fact Sales audit logging and a distinct-key overlap check. It is a **sanitised portfolio POC**, not a production-ready privacy or security control. The original notebook remains available for inspection; the Python blocks above reproduce its source without changing the implementation.
