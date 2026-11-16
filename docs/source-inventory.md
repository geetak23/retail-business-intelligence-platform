# Source Inventory

## Overview

The Retail Business Intelligence Data Platform integrates three primary
source systems.

| Source | Type | Purpose | Ingestion Method |
|---|---|---|---|
| Microsoft Wide World Importers | SQL Server OLTP | Core retail transactions, customers, products, inventory, geography | Azure Data Factory |
| Open-Meteo | REST API | Historical weather context | Azure Data Factory |
| Nager.Date | REST API | Public holiday context | Azure Data Factory |

---

## Source 1 — Microsoft Wide World Importers

### Source Type

SQL Server relational OLTP database.

### Purpose

Provides the core retail transactional and reference data used by the
platform.

### Primary Data Domains

- Customers
- Orders
- Order Lines
- Invoices
- Invoice Lines
- Products
- Inventory
- Geography
- Transaction Types

### Historical Coverage

2013-01-01 through 2016-05-31 for the primary sales transaction data.

### Key Tables

- `Sales.Customers`
- `Sales.Orders`
- `Sales.OrderLines`
- `Sales.Invoices`
- `Sales.InvoiceLines`
- `Warehouse.StockItems`
- `Warehouse.StockItemHoldings`
- `Warehouse.StockItemTransactions`
- `Application.TransactionTypes`
- `Application.Cities`
- `Application.StateProvinces`
- `Application.Countries`

### Initial Ingestion Approach

ADF will extract data from the SQL Server source and write the raw
source data to ADLS Gen2 Landing.

Incremental extraction will be considered for tables that contain
suitable modification timestamps.

---

## Source 2 — Open-Meteo

### Source Type

Public REST API.

### Purpose

Provides daily weather context for analysis of potential relationships
between weather conditions and retail sales.

### Location

Dallas, Texas.

### Key Data

- Weather date
- Maximum temperature
- Minimum temperature
- Precipitation
- Rainfall
- Maximum wind speed

### Expected Grain

One weather record per location per calendar date.

### Time Zone

America/Chicago.

### Initial Ingestion Approach

ADF will call the historical weather API and store the raw API response
in ADLS Gen2 Landing.

---

## Source 3 — Nager.Date

### Source Type

Public REST API.

### Purpose

Provides public holiday and jurisdiction information for analyzing
potential changes in retail sales around holidays.

### Country

United States (`US`).

### Key Data

- Holiday date
- Local holiday name
- Holiday name
- Country code
- Fixed-date indicator
- Global applicability indicator
- Regional applicability
- Holiday types

### Expected Grain

One record per holiday occurrence and applicable
jurisdiction/type combination.

### Initial Ingestion Approach

ADF will call the API by year and store the raw API response in
ADLS Gen2 Landing.

The initial retail analysis requires holiday data for 2013 through
2016.

---

## Source-to-Platform Flow

```text
Wide World Importers ───────┐
                            │
Open-Meteo ─────────────────┼──> ADF ──> ADLS Gen2 Landing
                            │
Nager.Date ────────────────┘


