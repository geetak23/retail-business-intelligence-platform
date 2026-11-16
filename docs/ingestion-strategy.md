# Ingestion Strategy

## 1. Ingestion Principles

The ingestion layer is responsible for extracting data from the defined
source systems and reliably loading the raw source data into Azure Data
Lake Storage Gen2.

Azure Data Factory (ADF) will be the primary ingestion and orchestration
service.

### Core Principles

1. **Preserve raw source data**

   Source data will be preserved in the Landing layer before downstream
   transformation.

2. **Separate ingestion from transformation**

   ADF will focus on extraction, loading, parameterization, and
   orchestration.

   Databricks will perform the primary Bronze, Silver, and Gold
   transformations.

3. **Use the appropriate load strategy**

   The platform will support:

   - Full loads
   - Incremental loads
   - Parameterized API loads

4. **Parameterize reusable pipelines**

   Source-specific values such as dates, years, API parameters, and
   watermarks should be configurable rather than hard-coded.

5. **Maintain source traceability**

   Source keys and relevant source metadata will be preserved so that
   downstream records can be traced back to their originating source.

6. **Design for repeatable execution**

   Pipelines should be safely rerunnable without unintentionally
   creating duplicate analytical records.

7. **Fail visibly**

   Critical ingestion failures must cause the pipeline to report a
   failure rather than silently producing incomplete data.

8. **Preserve failed or invalid records**

   Records rejected during downstream validation should be routed to
   the quarantine area rather than silently discarded.

9. **Secure source connectivity**

   Credentials and secrets must not be hard-coded in pipeline
   definitions.

   Azure Key Vault and managed identities will be used where
   appropriate.

10. **Maintain auditability**

    The platform will capture sufficient metadata to understand what
    was processed, when it was processed, and whether the ingestion
    succeeded.

11. **Prefer simplicity initially**

    The first implementation will focus on a reliable end-to-end
    pipeline before introducing advanced metadata-driven frameworks or
    unnecessary optimization.

### Source-Specific Strategy

The three sources require different ingestion approaches:

```text
Wide World Importers
        ↓
SQL Server extraction
        ↓
Full + incremental ingestion


Open-Meteo
        ↓
Parameterized REST API
        ↓
Date-range ingestion


Nager.Date
        ↓
Parameterized REST API
        ↓
Year-based ingestion

## 2. Wide World Importers Ingestion

### Source

Microsoft Wide World Importers running on SQL Server.

The source provides the core retail transaction, customer, product,
inventory, and geographic data used by the platform.

### Ingestion Service

Azure Data Factory will extract the WWI data and write the raw source
records to ADLS Gen2 Landing.

```text
Wide World Importers
        ↓
SQL Server
        ↓
Azure Data Factory
        ↓
ADLS Gen2 Landing

## 3. Open-Meteo Ingestion

### Source

Open-Meteo Historical Weather API.

The source provides daily weather observations for a configured
geographic location and date range.

### Ingestion Service

Azure Data Factory will call the REST API and store the raw response in
ADLS Gen2 Landing.

```text
Open-Meteo
    ↓
REST API
    ↓
Azure Data Factory
    ↓
ADLS Gen2 Landing

## 4. Nager.Date Ingestion

### Source

Nager.Date Public Holiday API.

The source provides public holiday information for a specified country
and year.

### Ingestion Service

Azure Data Factory will call the REST API and store the raw API response
in ADLS Gen2 Landing.

```text
Nager.Date
    ↓
REST API
    ↓
Azure Data Factory
    ↓
ADLS Gen2 Landing

## 5. ADLS Gen2 Landing Structure

The Landing layer is the raw ingestion boundary between the source
systems and the downstream data processing layers.

Data in Landing should remain as close as practical to the original
source format.

### Source-Based Organization

The Landing layer will separate data by source system.

```text
landing/
├── wwi/
├── weather/
└── holidays/

## 6. Load Strategy and Watermarks

The ingestion strategy will use different load patterns depending on the
characteristics of each source dataset.

The initial implementation will support:

- Full loads
- Incremental loads
- Parameterized API loads

### 6.1 Full Load

A full load extracts the complete available dataset from the source.

Full loads will initially be used for:

- Initial historical ingestion
- Small reference datasets
- Datasets where incremental extraction is not yet required

Examples include:

```text
Application.TransactionTypes
Application.Countries
Application.StateProvinces
Application.Cities

## 7. Idempotency and Duplicate Prevention

The ingestion pipelines must be designed so that rerunning a pipeline
does not unintentionally create duplicate analytical records.

This is particularly important when:

- A pipeline is retried after a failure.
- A historical date range is reprocessed.
- An incremental load is rerun.
- A failed downstream transformation requires reprocessing.

### Idempotency Principle

A pipeline run should produce the same logical result when the same
source data and processing parameters are executed again.

Conceptually:

```text
Same Source Data
      +
Same Parameters
      ↓
Repeat Pipeline
      ↓
Same Logical Dataset

## 8. Error Handling, Retry, and Failure Management

The ingestion pipelines must distinguish between temporary technical
failures and permanent data or configuration failures.

The goal is to make failures visible, diagnosable, and recoverable.

### 8.1 Transient Failures

Temporary failures may include:

- Network interruptions
- Temporary API availability issues
- SQL connection interruptions
- Temporary Azure service errors
- Request timeouts

These failures should use controlled retry behavior where supported by
Azure Data Factory.

Conceptually:

```text
Pipeline Activity
       ↓
    Failure
       ↓
   Retry
       ↓
 ┌─────┴─────┐
Success     Failure
              ↓
          Report Failure

          ## 9. Audit and Monitoring Metadata

The ingestion framework will capture operational metadata for each
pipeline execution.

The purpose is to provide visibility into what was processed, when it
was processed, how many records were handled, and whether the execution
succeeded.

### 9.1 Pipeline Run ID

Each pipeline execution will have a unique run identifier.

This identifier will be used to correlate:

- ADF pipeline execution
- Source extraction
- Landing files
- Downstream Databricks processing
- Data-quality results
- Errors and quarantine records

Conceptually:

```text
Pipeline Run ID
      │
      ├── ADF execution
      ├── Landing data
      ├── Bronze processing
      ├── Silver processing
      ├── Data-quality results
      └── Errors / quarantine

      ## 10. Security and Connectivity

The ingestion layer must securely connect Azure Data Factory to the
defined source systems without exposing credentials in pipeline
definitions.

### 10.1 Azure Key Vault

Secrets and credentials required by the ingestion framework will be
stored in Azure Key Vault rather than hard-coded in:

- ADF pipelines
- Linked services
- Source code
- Configuration files

```text
Azure Data Factory
        ↓
Azure Key Vault
        ↓
Secrets / Credentials

## 11. Ingestion Pipeline Organization

Azure Data Factory pipelines will be organized by source and ingestion
responsibility.

The initial implementation will favor clear, source-specific pipelines
rather than introducing a complex metadata-driven framework.

### 11.1 WWI Pipelines

The WWI ingestion process will be separated logically by dataset type.

The initial pipeline organization will support:

```text
WWI
├── Transactional data
├── Master data
├── Reference data
└── Inventory data

## 12. Ingestion Execution Order

The ingestion process will follow a controlled sequence to establish
the raw data foundation before downstream transformation begins.

### 12.1 Initial Execution Sequence

The initial execution will follow this general order:

```text
Source Connectivity Validation
            ↓
WWI Historical Load
            ↓
Weather Historical Load
            ↓
Holiday Historical Load
            ↓
Landing Data Validation
            ↓
Ready for Databricks Processing

## 13. Ingestion Strategy Summary

The ingestion layer establishes a reliable and traceable path from the
defined source systems into Azure Data Lake Storage Gen2.

### Source Strategy

| Source | Ingestion Pattern | Initial Scope |
|---|---|---|
| Wide World Importers | Full + incremental SQL extraction | 2013-01-01 → 2016-05-31 |
| Open-Meteo | Parameterized REST API | 2013-01-01 → 2016-05-31 |
| Nager.Date | Parameterized year-based REST API | 2013–2016 |

### Core Architecture

```text
                    Source Systems
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       SQL Server     Open-Meteo     Nager.Date
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                 Azure Data Factory
                         │
                         ↓
                  ADLS Gen2 Landing
                         │
                         ↓
                      Bronze
                         │
                         ↓
                      Silver
                         │
                         ↓
                       Gold
                         │
                         ↓
                     Power BI

                     