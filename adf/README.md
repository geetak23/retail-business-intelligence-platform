# Azure Data Factory assets

This folder contains the ADF datasets, linked services and ingestion pipeline
exports available in the project snapshot.

## Included

- **Datasets:** ADLS Gen2 landing datasets, SQL Server source datasets, and
  HTTP datasets for Open-Meteo and Nager.Date.
- **Linked services:** ADLS Gen2, Open-Meteo and Nager.Date.
- **Pipelines:** WWI orders, order lines, customers, historical weather and
  public holidays.

These are source exports for review and version control. They are not a
complete, one-click deployment package. The project currently has individual
ingestion pipelines; a master/orchestration pipeline and triggers are not
included.

## SQL Server linked service

The SQL Server linked-service export from the working folder contained
`[REDACTED]` placeholders in fields such as server, username and encrypted
credential. Those placeholders made the file invalid JSON, so it is
intentionally excluded from this repository.

To configure the connection in your own ADF environment:

1. Create or update a SQL Server linked service named
   `LS_local_SqlServer_wwi_db`.
2. Configure the SQL Server host and `WideWorldImporters` database.
3. Select the self-hosted integration runtime appropriate to your network
   (`shIR-retail-bi-dev` in the original development setup).
4. Configure authentication securely in ADF. Do not commit passwords,
   encrypted credentials, connection strings or tokens.
5. Test the linked service in the target environment before running a pipeline.

The SQL source datasets refer to this linked-service name. The connection must
exist in the target factory before those datasets and pipelines can run.

## Review notes

See [`../docs/adf-review.md`](../docs/adf-review.md) for static validation
results and items to reconcile before deployment.

Environment-specific storage endpoints, linked-service authentication,
integration-runtime configuration and permissions must be reviewed in the
target Azure environment. Do not assume that importing these JSON files alone
will reproduce the original Azure setup.
