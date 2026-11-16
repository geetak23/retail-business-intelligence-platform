# Retail Business Intelligence Data Platform

## Project Overview

This project demonstrates a practical Azure data engineering solution built using retail data and external data sources. It follows a medallion architecture to ingest, process, transform, and organize data into analytics-ready datasets that support business reporting and decision-making.

The solution integrates transactional data from WideWorldImporters SQL Server with external weather and public-holiday data. Azure Data Factory handles data ingestion, Azure Data Lake Storage Gen2 provides the storage layers, and Azure Databricks with PySpark and Delta Lake supports data transformation and processing.

This is an active portfolio project. The core Azure resources, ingestion components, and primary Bronze, Silver, and Gold notebooks have been developed. The repository provides an overview of the implementation, architecture, and current development status.

## Technology Stack

- **Source System:** WideWorldImporters (SQL Server)
- **External Data Sources:** Open-Meteo API, Nager.Date API
- **Orchestration and Ingestion:** Azure Data Factory (ADF)
- **Cloud Storage:** Azure Data Lake Storage Gen2 (ADLS Gen2)
- **Data Processing:** Azure Databricks, PySpark, Delta Lake
- **Data Governance:** Unity Catalog
- **Security:** Azure Managed Identity and Azure Key Vault
- **Analytics:** Power BI (planned)

## Architecture and Data Flow

The solution follows a Medallion Architecture with Landing, Bronze, Silver, and Gold layers.

```text
WideWorldImporters SQL Server ─┐
Open-Meteo API ────────────────┼──> Azure Data Factory
Nager.Date API ────────────────┘             |
                                              v
                                      ADLS Gen2 Landing
                                              |
                                              v
                                           Bronze
                                              |
                                              v
                                           Silver
                                              |
                                              v
                                            Gold
                                              |
                                              v
                                    Analytics / Power BI
                                         (Planned)
```

### Medallion Architecture

- **Landing:** Stores source data ingested from SQL Server and external APIs.
- **Bronze:** Maintains raw ingested data using incremental ingestion patterns.
- **Silver:** Applies data cleansing, standardization, validation, and transformation.
- **Gold:** Contains business-oriented dimensional models and aggregated datasets for analytical consumption.

The Databricks notebooks implement Auto Loader ingestion into Bronze, WideWorldImporters transformations into Silver, weather and holiday transformations, Gold model creation, and data-quality validation.

## Current Implementation

### Gold Data Model

The current Gold layer contains the following datasets:

| Table | Description |
|---|---|
| `FactSales` | Sales fact table containing one record per order line. |
| `DimDate` | Date dimension with calendar attributes covering the order-date range. |
| `DimCustomer` | Customer dimension containing the current customer representation. |
| `AggDailySales` | Daily sales aggregation for reporting and analysis. |

### Azure Data Factory Pipelines

The repository includes modular Azure Data Factory ingestion pipelines for:

- WideWorldImporters orders
- WideWorldImporters order lines
- WideWorldImporters customers
- Historical weather data
- Public-holiday data

The pipelines are organized as independent, reusable components.

Environment-specific configuration, including the SQL Server linked service and Integration Runtime, is completed during deployment to the target ADF environment.

Master pipeline orchestration and trigger-based scheduling are planned as future enhancements.

### Development Results

The development phase successfully produced the following data volumes:

| Dataset | Records |
|---|---:|
| Orders | 73,595 |
| Order Lines | 231,412 |
| Customers | 663 |
| FactSales | 231,412 |
| AggDailySales | 1,069 |

These results provide a baseline of the data processing outcomes achieved during development.

## Repository Structure

```text
retail-business-intelligence-platform/
│
├── adf/
│   ├── datasets/          # ADF dataset exports
│   ├── linked_services/   # Non-secret linked-service exports
│   ├── pipelines/         # Individual ingestion pipelines
│   └── README.md          # ADF configuration and setup
│
├── databricks/            # Bronze, Silver, Gold and DQ notebooks
│
├── data/                  # Sample API payloads
│
├── docs/                  # Architecture, mappings and project documentation
│
├── powerbi/               # Power BI-related assets
│
├── scripts/               # Supporting scripts
│
├── sql/                   # SQL scripts
│
├── tests/                 # Validation and testing assets
│
├── README.md
└── SECURITY.md
```

## Project Documentation

For additional implementation details, refer to the following documentation:

- [Project Status and Remaining Work](docs/project-status.md)
- [ADF Export Review](docs/adf-review.md)
- [Architecture and Source Inventory](docs/source-inventory.md)
- [Source-to-Target Mapping](docs/source-to-target-mapping.md)
- [Data Quality Rules](docs/data-quality-rules.md)
- [Ingestion Strategy](docs/ingestion-strategy.md)
- [CI/CD Plan](docs/ci-cd-plan.md)
- [ADF Setup Notes](adf/README.md)
- [Security Guidance](SECURITY.md)

## Deployment and Configuration

This repository provides the developed Azure Data Factory configurations, Databricks notebooks, and supporting documentation as a foundation for technical review and further development.

To deploy the solution in another environment, configure the following environment-specific components:

1. Azure Data Factory linked services and Integration Runtime.
2. ADLS Gen2 storage permissions and access configuration.
3. Azure Key Vault access and secret references.
4. Databricks catalog, storage locations, and notebook paths.
5. Environment-specific parameters and resource configurations.

Refer to [ADF Setup Notes](adf/README.md) for additional configuration details before deployment.

## Project Status

The repository distinguishes implemented capabilities from planned enhancements to provide a transparent view of the current solution.

The core data ingestion components, medallion processing notebooks, Gold data model, and development validation results are available for technical review.

Further enhancements include master pipeline orchestration, trigger-based scheduling, additional deployment automation, and Power BI reporting integration.

## Conclusion

This project demonstrates hands-on experience designing and implementing an Azure-based retail data engineering solution using Azure Data Factory, ADLS Gen2, Azure Databricks, PySpark, Delta Lake and Unity Catalog.

It provides a practical example of working with multiple data sources, implementing medallion architecture, transforming transactional data into analytical models, and applying data-quality validation.

The repository serves as a technical portfolio demonstrating the Azure Data Engineering concepts, services and implementation skills applied throughout the project.
