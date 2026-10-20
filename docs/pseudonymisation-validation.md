# Pseudonymisation Validation and Audit Logging

## 1. Objective

Validate that the Microsoft Fabric pseudonymisation process transforms pharmacy identifiers consistently, preserves analytical relationships and provides visibility into processing activity.

## 2. Validation Approach

The proof of concept includes checks for deterministic pseudonymisation, cross-table consistency and identifier replacement.

| Validation area | Expected outcome | POC evidence |
|---|---|---|
| Deterministic pseudonymisation | Same source identifier produces the same pseudonym | HMAC-SHA-256 transformation logic |
| Cross-table consistency | Matching pharmacy identifiers remain joinable | 120 matching distinct identifiers observed during testing |
| Embedded identifier detection | Known identifiers embedded in string values are replaced | Tested using synthetic values containing embedded pharmacy IDs |
| Curated table creation | Transformed datasets are written as Delta tables | `psy.PharmacyFactSales_Curated` and `psy.PharmacyDimPharmacy_Curated` |
| Processing audit logging | Replacement counts are recorded by column | Implemented for the Fact Sales processing stage |

## 3. Referential Integrity Validation

A central requirement was preserving the relationship between pharmacy sales and pharmacy dimension data.

The same HMAC-SHA-256 transformation is applied consistently to matching identifiers.

During the POC, a validation check identified **120 matching distinct pseudonymised pharmacy identifiers** across the relevant tables.

This provided evidence that the transformation preserved the matching keys needed for analytical joins.

It does not, by itself, prove that all records passed every referential-integrity check.

## 4. Cross-Column Identifier Replacement

The transformation scans string columns for known pharmacy identifiers.

Testing included synthetic values where an identifier appeared within other text, rather than occupying an entire field.

The replacement logic preserves surrounding text while substituting the detected identifier with its pseudonym.

This helps address accidental duplication of sensitive identifiers into descriptive fields.

## 5. Processing Audit Log

The notebook writes processing metadata to:

`metadata.PseudonymisationProcessingLog`

The audit structure includes:

| Field | Description |
|---|---|
| `RunID` | Unique identifier for the notebook execution |
| `SourceTable` | Source table being processed |
| `ColumnName` | Column examined |
| `ReplacementsCount` | Number of matched identifier occurrences |
| `ProcessedAt` | Processing timestamp |

The current implementation records processing activity for `dbo.PharmacyFactSales`.

Replacement counts represent matched occurrences rather than necessarily the number of distinct rows modified.

## 6. Limitations

The POC validation does not establish that every possible identifier occurrence has been detected.

The current identifier mapping is constructed from known source identifiers and uses an in-memory collection suitable for a small demonstration.

Further work should include:

- Automated source-to-curated record-count reconciliation.
- Validation of uniqueness and null handling.
- Checks for unmatched foreign keys.
- Tests for residual identifiers across all relevant columns.
- Verification of processing-log completeness.
- Audit logging across both transformed tables.
- Scalable identifier matching for larger datasets.

## 7. Conclusion

The POC demonstrated consistent deterministic pseudonymisation and successful matching of transformed identifiers across the pharmacy fact and dimension tables.

It also introduced cross-column identifier detection and processing metadata logging.

Additional automated testing and operational controls would be required before production deployment.
