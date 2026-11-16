# Retail Business Intelligence Data Platform

## Business Objective

Build a cloud-based data platform that integrates retail transactional
data with external weather and holiday data to produce
analytics-ready datasets for business users.

The platform will use Microsoft Wide World Importers as the core retail
source and enrich the retail data with weather and public-holiday
context.

The goal is to create a reliable, scalable, and production-style data
engineering solution using Azure, Azure Data Factory, Azure Data Lake
Storage, and Databricks.

## Primary Business Questions

The platform should support the following business questions:

1. What is the daily sales performance?
2. How is sales performance changing over time?
3. Which products generate the most sales and profit?
4. Which customers generate the most sales?
5. Which geographic areas perform best?
6. What is the current inventory position?
7. How does inventory movement change over time?
8. How do sales compare with previous periods?
9. Are there observable sales patterns around holidays?
10. Are there observable relationships between weather conditions and sales?
11. What is the status and data quality of the data pipelines?

## Business Users

The primary consumers of the platform are:

- Business analysts
- Sales and operations teams
- Inventory and supply-chain teams
- Data analysts
- Business stakeholders and management

These users will consume curated, analytics-ready data through
reporting and business intelligence tools such as Power BI.

## Key Business Metrics

The platform should support calculation of the following metrics:

### Sales Metrics

- Daily sales amount
- Sales amount by product
- Sales amount by customer
- Sales amount by geographic area
- Units sold
- Line profit
- Average order value

### Inventory Metrics

- Quantity on hand
- Inventory value based on available cost data
- Inventory movements
- Reorder level
- Target stock level

### Contextual Metrics

- Daily maximum temperature
- Daily minimum temperature
- Precipitation
- Rainfall
- Maximum wind speed
- Holiday indicator
- Holiday type
- Texas-specific holiday applicability

## Data Scope

### Core Retail Data

The platform will initially use the following Wide World Importers
data domains:

- Customers
- Orders
- Order Lines
- Invoices
- Invoice Lines
- Products
- Inventory
- Geography

### External Context Data

The platform will integrate:

- Historical weather data from Open-Meteo
- Public holiday data from Nager.Date

### Historical Period

The initial retail analysis will use the available Wide World Importers
data from:

- Start date: 2013-01-01
- End date: 2016-05-31

External weather and holiday data will be aligned to the applicable
retail analysis dates.

## Business Requirements Summary

The platform must:

1. Ingest reliable retail data from Wide World Importers.
2. Ingest historical weather data from Open-Meteo.
3. Ingest public holiday data from Nager.Date.
4. Preserve raw source data for traceability.
5. Transform and validate data through Bronze, Silver, and Gold layers.
6. Apply data-quality rules to identify invalid or incomplete records.
7. Support incremental processing where appropriate.
8. Produce analytics-ready datasets for Power BI.
9. Support analysis of sales, products, customers, geography, inventory,
   holidays, and weather.
10. Provide sufficient monitoring and audit information to understand
    pipeline execution and data quality.

## Out of Scope for Initial Version

The initial version will not include:

- Real-time streaming
- Customer-level weather attribution
- Predictive sales forecasting
- Machine-learning models
- Automated business decision-making
- Additional external data sources beyond the three defined sources

These capabilities may be considered as future enhancements.