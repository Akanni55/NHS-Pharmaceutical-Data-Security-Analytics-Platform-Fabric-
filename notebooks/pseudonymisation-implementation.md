# PySpark Pseudonymisation Implementation

## 1. Overview

This notebook demonstrates deterministic pseudonymisation of pharmaceutical identifiers using PySpark and HMAC-SHA-256 within Microsoft Fabric.

The solution transforms raw Lakehouse data into curated Delta tables while preserving relationships required for analytical reporting.

This is a personal, NHS-inspired security demonstration using fictional pharmaceutical data.

## 2. Source and Destination Tables

| Source Table | Curated Table |
|---|---|
| `dbo.PharmacyFactSales` | `psy.PharmacyFactSales_Curated` |
| `dbo.PharmacyDimPharmacy` | `psy.PharmacyDimPharmacy_Curated` |

The Date and Product dimensions are not pseudonymised by this notebook.

## 3. Deterministic HMAC-SHA-256 Pseudonymisation

The notebook generates a pseudonym for each pharmacy identifier using HMAC-SHA-256.

The same identifier produces the same pseudonym when processed using the same secret key and normalisation rules.

This supports consistent relationships between the sales fact table and pharmacy dimension.

HMAC pseudonymisation does not make data anonymous. The curated data must still be governed appropriately.

## 4. Cross-Column Identifier Detection

The implementation goes beyond replacing identifiers in the primary `PharmacyID` column.

It scans string columns for identifiers that may have been copied into other fields, including identifiers embedded within longer text.

A regular-expression replacement mechanism identifies known pharmacy IDs and substitutes their corresponding pseudonyms.

For example, a fictional identifier embedded within a comment can be replaced without discarding the surrounding text.

## 5. Curated Delta Table Creation

The transformed DataFrames are written as Delta tables within the `psy` schema.

The notebook uses overwrite mode to regenerate the curated datasets.

This provides a repeatable transformation process for the portfolio demonstration.

## 6. Processing Audit Log

The Fact Sales processing stage includes an audit logging mechanism.

The log records:

- Execution run identifier.
- Source table.
- Column name.
- Number of identifier occurrences replaced.
- Processing timestamp.

The log is written to `metadata.PseudonymisationProcessingLog`.

Current notebook coverage is limited to the Fact Sales processing stage. Extending the audit mechanism consistently across all transformed tables is a future enhancement.

## 7. Validation

The implementation includes validation of pseudonymised identifiers and their consistency between the fact and pharmacy dimension tables.

Validation should also cover null handling, identifier uniqueness, record counts and detection of residual sensitive identifiers.

## 8. Security Considerations

The public notebook does not contain a usable HMAC secret.

For production deployment, secret material should be obtained from an approved secrets-management service, with appropriate access controls and rotation procedures.

The raw Lakehouse tables must remain access-restricted.

Pseudonymisation is one security control and does not replace role-based access, data minimisation or information governance requirements.

## 9. Limitations and Future Improvements

The demonstration builds an in-memory identifier mapping using `collect()`. This approach is suitable only for small datasets and should be replaced with a scalable distributed design for larger workloads.

Additional improvements include:

- Using an authoritative identifier mapping across all tables.
- Introducing boundary-aware identifier detection to reduce false-positive replacements.
- Extending audit logging to all curated tables.
- Implementing automated data quality tests.
- Managing HMAC secrets through an approved secure service.

## 10. Implementation Status

The notebook demonstrates HMAC-based pseudonymisation, cross-column identifier replacement and curated Delta table generation.

The GitHub version has been sanitised for public documentation and requires secure runtime configuration before execution.
