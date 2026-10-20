# Security Design Evaluation: HMAC Pseudonymisation vs Dynamic Data Masking

## 1. Objective

Evaluate two approaches to protecting sensitive pharmaceutical identifiers within a Microsoft Fabric and Power BI analytics solution.

The assessment focused on:

- Protecting sensitive identifiers in reporting datasets.
- Maintaining relationships between fact and dimension tables.
- Preventing unintended identifier exposure to report consumers.
- Supporting controlled report distribution.

## 2. Options Evaluated

| Area | Option A: HMAC-SHA-256 | Option B: Dynamic Data Masking |
|---|---|---|
| Approach | Transform identifiers into deterministic pseudonyms | Mask query results based on database permissions |
| Stored values | Curated tables contain pseudonyms | Original values remain stored |
| Relationship consistency | Preserved using consistent HMAC inputs and key | Can be disrupted when distinct identifiers receive the same mask |
| Power BI Import behaviour | Imports pseudonymised identifiers | Imports values visible to the refresh identity |
| Exposure considerations | Requires secure key management and raw-layer access controls | Masking may not carry through to downstream imported models |
| POC decision | **Selected** | Not selected |

## 3. Option A — Deterministic Pseudonymisation

A PySpark notebook applies HMAC-SHA-256 to pharmacy identifiers and writes the transformed data to curated Delta tables.

The same source identifier produces a consistent pseudonym across the fact and dimension tables when the same secret and normalisation rules are used.

Testing demonstrated that the transformed identifiers maintained relationships between the relevant datasets.

The notebook was also extended to detect identifiers embedded in other string columns.

### Advantages

- Sensitive identifiers are transformed before reporting.
- Consistent pseudonyms support fact-to-dimension relationships.
- Power BI consumes curated values rather than relying on viewer-specific SQL masking.
- The transformation can be incorporated into an automated Fabric pipeline.

### Limitations

- The HMAC secret requires secure management.
- Raw source data remains sensitive and must be access-controlled.
- Pseudonymised information is not automatically anonymous.
- Identifier detection requires additional validation to prevent missed matches or unintended replacements.

## 4. Option B — Dynamic Data Masking

Dynamic Data Masking was tested using separate masked tables and the Fabric SQL analytics endpoint.

Masking rules were applied to the `PharmacyID` column.

### Test 1 — SQL Endpoint Permissions

A privileged developer identity could retrieve the original identifier.

A restricted test identity received masked values.

**Finding:** SQL masking behaved differently according to the querying identity, as intended.

### Test 2 — Power BI Import and App Access

A Power BI model imported using a privileged connection retained the original identifier values.

Restricting an end user's access to the Power BI App did not retroactively mask those imported values.

**Finding:** Source-level SQL masking was not sufficient to protect identifiers already imported into the semantic model.

### Test 3 — Restricted Identity and Model Relationships

When the Power BI source connection was changed to a restricted identity, the query returned masked identifiers.

However, multiple distinct pharmacy identifiers became the same masked value.

This caused a uniqueness problem in the pharmacy dimension, preventing the existing one-to-many relationship from refreshing successfully.

**Finding:** Default masking can interfere with relational model integrity when applied to a relationship key.

## 5. Security Risks Identified

The POC highlighted three key limitations of relying on Dynamic Data Masking alone:

1. **Refresh identity dependency:** Power BI Import stores the values returned to the account performing the refresh.
2. **Relationship key collisions:** Multiple identifiers can collapse into the same masked value.
3. **Incomplete protection boundary:** SQL masking does not replace the original underlying data or universally protect every access path.

## 6. Final Design Decision

**Option A — HMAC-SHA-256 pseudonymisation was selected for the POC.**

This approach better met the requirement to transform identifiers before exposing data through the reporting layer while retaining consistent analytical relationships.

Dynamic Data Masking remains useful for appropriate SQL access-control scenarios, but it was not suitable as the primary protection mechanism for this Power BI Import-based design.

## 7. Additional Reporting Controls

The proposed reporting security model includes:

- Power BI App distribution to authorised audiences.
- Security-group-based access management.
- Restricted workspace access.
- Limiting data exports to summarised information where supported.
- Avoiding unnecessary Build permissions for report consumers.

These controls complement pseudonymisation and require configuration and validation for each deployment.

## 8. Conclusion

The evaluation demonstrates the importance of testing security controls across the complete data lifecycle, not only at the source database.

The selected architecture applies deterministic pseudonymisation before the Power BI semantic model consumes the data.

This is a proof-of-concept security evaluation using fictional pharmaceutical data and does not represent a formal NHS security approval.
