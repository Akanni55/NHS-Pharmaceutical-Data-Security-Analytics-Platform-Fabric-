# NHS-Pharmaceutical Data Security & Analytics Platform

### An NHS-Inspired Microsoft Fabric Data Engineering Project

**Technologies:** Microsoft Fabric | PySpark | Python | Delta Lake | HMAC-SHA-256 | SQL | Power BI

## Project Overview

This project demonstrates the design and implementation of a secure, end-to-end pharmaceutical data engineering and analytics platform using Microsoft Fabric and Power BI.

The solution is based on a fictional NHS-inspired healthcare scenario in which pharmaceutical transaction data is collected from multiple pharmacy locations and used to support financial reporting, operational monitoring and data-driven decision-making.

A key challenge is ensuring that sensitive identifiers remain protected while maintaining consistent relationships across datasets and preserving analytical functionality.

The platform addresses this challenge through automated data ingestion, deterministic HMAC-SHA-256 pseudonymisation, cross-column sensitive identifier detection, processing audit logs and secure Power BI reporting.
## 📊 Live Interactive Power BI Dashboard

Explore the **Pharmaceutical Intelligence Dashboard**, an interactive analytics report built with Power BI using fictional pharmaceutical sales data.

### [🚀 View Live Interactive Dashboard](https://app.fabric.microsoft.com/view?r=eyJrIjoiY2FjM2IxMDMtNDY5OS00NDk5LTg4ZWQtMzYxODZhNzJkNzBkIiwidCI6IjYyODhhOGI0LWYxN2ItNGMzOC04ODkzLTE1MTNkMmM5MTVmZSJ9&embedImagePlaceholder=true)

**Dashboard capabilities**
- Revenue, cost, margin and sales performance KPIs.
- Monthly revenue and sales trends.
- Revenue comparisons across countries.
- Interactive filters and cross-filtering.
- Consistent pseudonymised pharmacy identifiers that preserve fact-to-dimension relationships.

**Data security:** This public dashboard is a separate portfolio demonstration using fictional data and independently generated pseudonyms. It illustrates the analytical output of the NHS-inspired Microsoft Fabric architecture without providing access to the original NHS test environment.

*This is an independent portfolio project and is not affiliated with or endorsed by the NHS.*
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

## Solution Architecture

The platform follows a layered Microsoft Fabric architecture, separating raw ingestion, pseudonymisation, curated storage, processing audit logs and Power BI reporting.

### Architecture Diagram

[View the End-to-End Data Flow](architecture/data-flow.md)

### Technical Documentation

[Read the Solution Architecture Overview](architecture/architecture-overview.md)

The architecture demonstrates how pharmaceutical data moves through the platform while sensitive identifiers are protected before being made available for analytics.

## Data Disclaimer

This is an independent personal portfolio project using a dummy NHS pharmaceutical data.

It is inspired by healthcare data security and analytics requirements but is not an official NHS project, does not use real NHS patient records, and is not affiliated with or endorsed by the NHS.

All architecture, code and documentation published in this repository are intended for demonstration and educational purposes.
