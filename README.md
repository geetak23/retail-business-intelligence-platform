# Retail Business Intelligence Data Platform

I built this project to practice and demonstrate a practical Azure data
engineering workflow using retail data and a couple of external data sources.
The goal is to take source data from ingestion through a medallion-style lake
architecture and shape it into datasets that can support business reporting.

This is an active portfolio project. The core Azure resources and the main
Bronze, Silver and Gold notebooks have been created, but some engineering
reviews and deployment work are still open. The repository is useful for
reviewing the implementation; it is not a one-click production deployment.

## What the project uses

- **WideWorldImporters on SQL Server** for customers, orders and order lines.
- **Open-Meteo** for historical weather data.
- **Nager.Date** for US public-holiday data.
- **Azure Data Factory** for source ingestion.
- **Azure Data Lake Storage Gen2** for the lake storage layers.
- **Azure Databricks, PySpark and Delta Lake** for data processing.
- **Unity Catalog and Azure identity-based access** for data governance and
  storage access.
- **Azure Key Vault** for secret-management integration.

## How the data flows

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
                                  Analytics / Power BI (planned)
```

The Databricks notebooks cover Auto Loader ingestion into Bronze, WWI
transformations into Silver, weather and holiday transformations, Gold model
creation, and data-quality checks.

## Current implementation

The current Gold model contains:

- `FactSales` — one row per order line in the implemented model.
- `DimDate` — date attributes covering the order-date range.
- `DimCustomer` — the current customer representation.
- `AggDailySales` — daily sales aggregations.

The ADF export includes individual ingestion pipelines for WWI orders, order
lines and customers, plus weather and holiday API ingestion. The exported
factory configuration is partial: the SQL Server linked service must be
configured in the target ADF environment, and there is no master pipeline or
trigger in this repository.

Previously recorded development results include 73,595 orders, 231,412 order
lines, 663 customers, 231,412 FactSales rows and 1,069 daily sales aggregate
rows. These are historical project results, not a fresh validation performed
when this repository was prepared.

## Repository layout

```text
adf/
  datasets/          ADF dataset exports
  linked_services/   Non-secret linked-service exports
  pipelines/         Individual ingestion pipeline exports
databricks/           Databricks notebooks for Bronze, Silver, Gold and DQ
data/                 Small sample API payloads
docs/                 Architecture, source contracts, mappings and project notes
```

See the documentation for more detail:

- [Project status and remaining work](docs/project-status.md)
- [ADF export review](docs/adf-review.md)
- [Architecture and source inventory](docs/source-inventory.md)
- [Source-to-target mapping](docs/source-to-target-mapping.md)
- [Data quality rules](docs/data-quality-rules.md)
- [Ingestion strategy](docs/ingestion-strategy.md)
- [CI/CD plan](docs/ci-cd-plan.md)
- [ADF setup notes](adf/README.md)
- [Security guidance](SECURITY.md)

## What is still in progress

A few items need attention before I would describe this as production-ready:

- Reconcile the Orders CSV column order with the explicit Auto Loader schema.
- Review the SQL Server linked-service setup and integration-runtime
  configuration in the target ADF environment.
- Verify the ADF sink file naming/extension behavior for the CSV landing files.
- Resolve the outstanding Silver transformation review items, including the
  holiday dataframe flow and deterministic customer deduplication.
- Add reconciliation for order lines that do not match an order before Gold
  fact creation.
- Improve data-quality failure handling and run-history retention.
- Add a non-deploying CI validation workflow, then define controlled ADF and
  Databricks deployment steps.

Weather and holiday samples currently cover January 2025, while the available
WideWorldImporters sales data covers 2013 through 2016. Those periods do not
overlap, so the current project does not claim to measure weather or holiday
effects on historical sales.

The Power BI dashboard has been deferred. Gold tables were connected and
visible during development, but a completed dashboard is not included here.

## Running and deployment

The notebooks and ADF JSON are provided for technical review and further
development. Before running them in another environment, configure the
environment-specific linked services, storage permissions, Key Vault access,
catalog and paths. Review the notes in `adf/README.md` first.

I am keeping this repository focused on the work that is actually present.
Future enhancements are tracked separately rather than represented as completed
features.
