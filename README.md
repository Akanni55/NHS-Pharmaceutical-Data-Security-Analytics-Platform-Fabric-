# NHS-Pharmaceutical Data Security & Analytics Platform

### An NHS-Inspired Microsoft Fabric Data Engineering Project

**Technologies:** Microsoft Fabric | PySpark | Python | Delta Lake | HMAC-SHA-256 | SQL | Power BI

## Project Overview

This project demonstrates the design and implementation of a secure, end-to-end pharmaceutical data engineering and analytics platform using Microsoft Fabric and Power BI.

The solution is based on a fictional NHS-inspired healthcare scenario in which pharmaceutical transaction data is collected from multiple pharmacy locations and used to support financial reporting, operational monitoring and data-driven decision-making.

A key challenge is ensuring that sensitive identifiers remain protected while maintaining consistent relationships across datasets and preserving analytical functionality.

The platform addresses this challenge through automated data ingestion, deterministic HMAC-SHA-256 pseudonymisation, cross-column sensitive identifier detection, processing audit logs and secure Power BI reporting.

## Business Problem

Healthcare organisations require reliable pharmaceutical data to monitor expenditure, understand transaction patterns and support operational decisions.

However, different stakeholders have different data-access requirements. Finance teams may require aggregated financial performance, operational teams may need pharmacy-level insights, and authorised analysts may require more detailed information.

Exposing sensitive identifiers unnecessarily creates privacy and governance risks.

This project investigates how a modern data platform can support analytical requirements while applying appropriate data protection controls.

## Project Objectives

1. Build an automated data ingestion process using Microsoft Fabric Data Pipelines.
2. Store and transform pharmaceutical data using Fabric Lakehouse and Delta tables.
3. Apply deterministic HMAC-SHA-256 pseudonymisation to sensitive identifiers.
4. Detect and pseudonymise identifiers appearing in other columns, including embedded text.
5. Maintain consistent identifiers and relationships across fact and dimension tables.
6. Implement processing logs to monitor pseudonymisation activity.
7. Develop Power BI dashboards for pharmaceutical analytics.
8. Evaluate Dynamic Data Masking against pseudonymisation and document technical findings.

## Data Disclaimer

This is an independent personal portfolio project using a dummy NHS pharmaceutical data.

It is inspired by healthcare data security and analytics requirements but is not an official NHS project, does not use real NHS patient records, and is not affiliated with or endorsed by the NHS.

All architecture, code and documentation published in this repository are intended for demonstration and educational purposes.
