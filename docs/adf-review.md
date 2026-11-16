# Azure Data Factory Implementation Review

**Review Type:** Static review of repository JSON exports  

## 1. Overview

Azure Data Factory is used in this project to ingest retail transactional data from WideWorldImporters SQL Server and external data from weather and public-holiday APIs into Azure Data Lake Storage Gen2.

The implementation follows a modular pipeline design, with separate ingestion pipelines for individual source entities and external data sources.

This review documents the available ADF configurations, validates repository artifacts and dataset references, and identifies configuration considerations for future deployment and execution.

The review also provides technical documentation of the current implementation and establishes a clear reference for extending the solution in the future.

## 2. Export Inventory

The repository contains the following Azure Data Factory artifacts:

| Artifact Type | Included |
|---|---:|
| Datasets | 6 |
| Linked Services | 3 valid JSON exports |
| Pipelines | 5 |
| Copy Activities | 5 |
| Master Pipeline | Not included in current export |
| Triggers | Not included in current export |

The SQL Server linked service is referenced by two SQL datasets but is not included as a deployable JSON file. The supplied source export contained redaction placeholders and was invalid JSON. The file was excluded to avoid committing an incomplete or misleading configuration.

Refer to [ADF Setup Notes](../adf/README.md) for additional configuration details.

## 3. Pipeline Implementation Summary

The project includes five individual ingestion pipelines:

| Pipeline | Source | Destination |
|---|---|---|
| `pl_ingest_wwi_orders` | SQL Server `Sales.Orders` | `landing/wwi/orders/orders.csv` |
| `pl_ingest_wwi_orderlines` | SQL Server `Sales.OrderLines` | `landing/wwi/orderlines/orderlines.csv` |
| `pl_ingest_wwi_customers` | SQL Server `Sales.Customers` | `landing/wwi/customers/customers.csv` |
| `pl_ingest_historical_weather` | Open-Meteo archive API | `landing/weather/weather.json` |
| `pl_ingest_holiday` | Nager.Date, US, 2025 | `landing/holiday/holidays.json` |

Each pipeline currently contains one Copy activity.

The three WideWorldImporters pipelines use SQL Server as the source and ADLS Gen2 as the destination.

The weather and holiday pipelines use HTTP sources and JSON-based landing destinations.

This modular approach separates source ingestion responsibilities and provides a foundation for independent pipeline maintenance and future orchestration.

## 4. Configuration Validation

The following static validation checks were completed:

- All included ADF JSON files parse successfully.
- Dataset references used by the five pipelines resolve to dataset files in the repository.
- HTTP and ADLS linked-service references resolve to included linked-service exports.
- SQL datasets correctly reference `LS_local_SqlServer_wwi_db`; its deployable configuration is intentionally excluded because the supplied export was invalid.
- Dataset parameters referenced by the pipelines are present in their corresponding dataset definitions.
- The supplied export does not contain a master pipeline or trigger definitions.

These checks establish that the available repository artifacts are structurally consistent, subject to the configuration considerations documented below.

## 5. Technical Review and Configuration Considerations

### 5.1 SQL Server Linked Service Configuration

The supplied `LS_local_SqlServer_wwi_db.json` contains `[REDACTED]` placeholders for connection fields, making it invalid JSON.

Rather than including an incomplete configuration, the linked service has been excluded from the deployable repository artifacts.

For deployment in another environment, configure the SQL Server linked service in the target ADF factory using:

- Correct SQL Server instance and database.
- Approved authentication mechanism.
- Appropriate self-hosted Integration Runtime.
- Secure credential management through ADF or an approved secret store.

This keeps environment-specific connection details and credentials separate from source-controlled project artifacts.

### 5.2 Orders Source Schema Alignment

The ADF `DS_SqlLocal_wwidb_orders` dataset defines the source columns in the following order:

1. `OrderID`
2. `CustomerID`
3. `SalespersonPersonID`
4. `PickedByPersonID`
5. `ContactPersonID`
6. `BackorderOrderID`
7. `OrderDate`
8. `ExpectedDeliveryDate`
9. `CustomerPurchaseOrderNumber`
10. `IsUndersupplyBackordered`
11. `Comments`
12. `DeliveryInstructions`
13. `InternalComments`
14. `PickingCompletedWhen`
15. `LastEditedBy`
16. `LastEditedWhen`

The Auto Loader `orders_schema` in `databricks/00_Auto_Loader_Bronze_ingestion.ipynb` currently uses a different positional schema.

It omits `BackorderOrderID` and `LastEditedBy`, while including `TotalDryItems` and `TotalChillerItems`.

Because the landing dataset has `firstRowAsHeader: false` and Auto Loader also reads with `header=false`, the source column ordering and ingestion schema need to be aligned before relying on a fresh ingestion.

This is an important source-contract validation consideration when working with headerless CSV files.

The existing notebook implementation was preserved during this repository review.

### 5.3 CSV File Naming and Extension Consistency

The WWI landing datasets use filenames ending in `.csv`, while the customers and order-lines Copy sinks specify `fileExtension: ".txt"`.

The Orders pipeline also uses the `.txt` sink extension setting.

The actual output naming behavior should be confirmed in the target ADF environment, and the following configurations should be aligned:

- Dataset filename.
- Copy activity sink extension.
- Auto Loader file discovery pattern.

This will help maintain consistent file discovery and ingestion behavior across the transactional datasets.

### 5.4 SQL Extraction Strategy

The exported Orders pipeline currently uses:

`select * from sales.orders;`

The Customers and OrderLines pipelines use parameterized table datasets.

The supplied exports do not show watermark parameters or incremental SQL filters.

Accordingly, these configurations are documented as full-table extraction patterns based on the available repository snapshot.

Incremental extraction using watermark-based filtering can be considered as a future enhancement to improve scalability and reduce unnecessary data movement.

### 5.5 Landing Storage and Security Configuration

The landing layer provides the entry point for source data entering the data lake.

Before deploying the solution in another environment, review:

- Storage linked-service authentication.
- ADLS Gen2 permissions and access controls.
- Self-hosted Integration Runtime availability and connectivity.
- Azure Key Vault references and permissions.
- Environment-specific security settings.

Customer source data may contain personal information, so approved development data and appropriate access restrictions should be maintained throughout the ingestion process.

### 5.6 Pipeline Orchestration and Scheduling

The current repository contains five independent pipeline definitions.

A master pipeline, dependency ordering, schedule/event triggers and failure-notification workflow are not included in the supplied export.

The modular pipeline structure provides a foundation for introducing centralized orchestration and scheduling when required.

These capabilities are documented as future enhancements rather than implemented functionality.

## 6. Deployment Considerations

For deployment into another Azure environment, the following activities provide a practical configuration checklist:

- Configure the SQL Server linked service and Integration Runtime.
- Validate connectivity between the source SQL Server and ADF.
- Configure ADLS Gen2 access and permissions.
- Confirm Key Vault integration and secure credential references.
- Align the Orders source schema with the Auto Loader schema.
- Confirm file naming and extension behavior.
- Validate source extraction strategy and dataset parameters.
- Review pipeline dependencies and execution order if orchestration is introduced.

These steps support a controlled deployment process while keeping environment-specific configuration separate from the portfolio repository.

## 7. Review Conclusion

The Azure Data Factory implementation demonstrates a modular source-ingestion approach using SQL Server and external HTTP-based data sources, with ADLS Gen2 serving as the landing destination.

The repository contains five individual ingestion pipelines, six datasets, and the associated linked-service configurations available for static technical review.

The review also validates the structure of the supplied JSON exports and documents important engineering considerations involving source schemas, file formats, secure connectivity, extraction strategies and orchestration.

The existing implementation provides a practical foundation for an Azure Data Engineering workflow and demonstrates familiarity with ADF components and their integration with the broader Databricks medallion architecture.

The current configurations and findings are preserved as part of the project documentation. Additional deployment validation and enhancements can be undertaken independently when required.

**Review Scope:** This document records static configuration findings and implementation considerations. No ADF pipeline or Databricks transformation logic was modified or executed as part of this review.
