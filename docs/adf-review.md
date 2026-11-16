# Azure Data Factory export review

**Review type:** Static review of the repository JSON exports  
**Review date:** 13 November 2026  
**Execution:** No ADF pipeline was run as part of this review.

## Export inventory

| Artifact type | Included |
|---|---:|
| Datasets | 6 |
| Linked services | 3 valid JSON exports |
| Pipelines | 5 |
| Activities | 5 Copy activities |
| Master pipeline | Not present |
| Triggers | Not present in this export |

The SQL Server linked service is referenced by two SQL datasets but is not
included as a deployable JSON file. Its source export contained redaction
placeholders and was invalid JSON. See [`../adf/README.md`](../adf/README.md).

## Pipeline summary

| Pipeline | Source | Destination |
|---|---|---|
| `pl_ingest_wwi_orders` | SQL Server `Sales.Orders` | `landing/wwi/orders/orders.csv` |
| `pl_ingest_wwi_orderlines` | SQL Server `Sales.OrderLines` | `landing/wwi/orderlines/orderlines.csv` |
| `pl_ingest_wwi_customers` | SQL Server `Sales.Customers` | `landing/wwi/customers/customers.csv` |
| `pl_ingest_historical_weather` | Open-Meteo archive API | `landing/weather/weather.json` |
| `pl_ingest_holiday` | Nager.Date, US, 2025 | `landing/holiday/holidays.json` |

Each exported pipeline currently contains one Copy activity. The three WWI
pipelines use the SQL Server source and ADLS Gen2 sink. Weather and holiday
pipelines use HTTP sources and JSON sinks.

## Checks performed

- All included ADF JSON files parse successfully.
- Dataset references used by the five pipelines resolve to dataset files in
  this repository.
- HTTP and ADLS linked-service references resolve to included linked-service
  exports.
- SQL datasets refer to `LS_local_SqlServer_wwi_db`; that linked service is
  intentionally not included because its supplied export was invalid.
- Dataset parameters used by the pipelines are present in the corresponding
  dataset definitions.
- No master pipeline or trigger definition was found in the supplied export.

## Items to resolve before deployment

### 1. SQL Server linked service is not deployable from this repository

The supplied `LS_local_SqlServer_wwi_db.json` contains bare `[REDACTED]`
placeholders for connection fields, which makes it invalid JSON. It has been
excluded rather than converted into a misleading deployable artifact.

Create the linked service in the target ADF factory using the correct server,
database, authentication and self-hosted integration runtime. Keep credentials
in ADF secure configuration or an approved secret store.

### 2. Orders source column order does not match the Auto Loader schema

The ADF `DS_SqlLocal_wwidb_orders` dataset lists the source columns in this
order:

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

The Auto Loader `orders_schema` in
`databricks/00_Auto_Loader_Bronze_ingestion.ipynb` does not use the same
positional schema. It omits `BackorderOrderID` and `LastEditedBy`, and includes
`TotalDryItems` and `TotalChillerItems` instead. Since the landing dataset has
`firstRowAsHeader: false` and Auto Loader also reads with `header=false`, this
is a material positional-schema mismatch that must be corrected before
relying on a fresh ingestion.

The notebook source was not changed during this repository preparation.

### 3. CSV file extension needs confirmation

The WWI landing dataset receives filenames ending in `.csv`, while the
customers and order-lines Copy sinks specify `fileExtension: ".txt"`. Confirm
the actual output naming behavior in ADF and align the dataset filename,
sink extension and Auto Loader file discovery pattern. The Orders pipeline
also uses the `.txt` sink extension setting.

### 4. SQL extraction strategy is currently full-table

The exported orders pipeline uses `select * from sales.orders;`. The customers
and order-lines pipelines use parameterized table datasets. The exports do not
show watermark parameters or incremental filters. Treat these as full extracts
unless the deployed pipeline contains additional configuration not present in
this snapshot.

### 5. Landing and security configuration must be checked per environment

Review the storage linked service authentication, ADLS permissions, self-hosted
integration runtime availability, Key Vault references and access controls
before deployment. Customer source data may contain personal information;
only use approved development data and protect the landing zone accordingly.

### 6. No master orchestration or triggers are included

The five pipelines are separate definitions. A master pipeline, dependency
ordering, schedule/event triggers and failure-notification workflow are not
present in the supplied export. Do not describe these as implemented.

## Review conclusion

The updated export is materially more complete than the earlier repository
snapshot and is useful for GitHub source control and technical review. It is
not yet a complete deployment package. The SQL Server linked service and the
Orders positional schema are the main blockers to treating the ADF-to-Bronze
path as ready for a fresh run.

This review records findings only. It does not modify ADF pipeline logic or
Databricks transformation logic.
