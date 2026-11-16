# Phase 2 — Source Architecture

## Source Systems

The Retail Business Intelligence Data Platform integrates three source
systems.

### Source 1 — Microsoft Wide World Importers

Purpose:
Core retail and business transaction data.

Source type:
SQL Server OLTP database.

Primary data domains:
- Customers
- Orders
- Order Lines
- Invoices
- Invoice Lines
- Stock Items
- Locations

### Source 2 — Open-Meteo Historical Weather API

Purpose:
Environmental context for analyzing weather-related changes in retail
sales.

Source type:
Public REST API.

Primary data:
- Daily maximum temperature
- Daily minimum temperature
- Precipitation
- Rainfall
- Maximum wind speed

### Source 3 — Nager.Date Public Holiday API

Purpose:
Calendar context for identifying potential holiday-related changes in
retail sales.

Source type:
Public REST API.

Primary data:
- Holiday date
- Holiday name
- Holiday type
- Country
- Global/local holiday indicator

## High-Level Flow

Wide World Importers
        |
        v
   Retail Sales Data
        |
        |
Open-Meteo ---------> Environmental Context
        |
        |
Nager.Date ----------> Calendar Context
        |
        v
     Azure Data Platform
        |
        v
   Analytics-Ready Data

## Target Architecture

Source Systems
     |
     v
ADF Ingestion
     |
     v
ADLS Gen2 Landing
     |
     v
Databricks
     |
     v
Bronze → Silver → Gold
     |
     v
Power BI