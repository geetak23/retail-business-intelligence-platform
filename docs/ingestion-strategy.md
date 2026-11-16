# Ingestion Strategy

## 1. Ingestion Principles

The ingestion layer is responsible for extracting data from the defined source systems and loading the raw source data into Azure Data Lake Storage Gen2 (ADLS Gen2).

Azure Data Factory (ADF) serves as the primary ingestion service, while Azure Databricks handles downstream Bronze, Silver, and Gold transformations.

### Core Principles

1. **Preserve raw source data**

   Source data is stored in the Landing layer before downstream transformation.

2. **Separate ingestion from transformation**

   ADF is responsible for source extraction, data movement, parameterization, and pipeline orchestration.

   Databricks performs the primary Bronze, Silver, and Gold transformations.

3. **Use appropriate load strategies**

   The platform design supports different ingestion patterns based on source characteristics:

   - Full loads
   - Incremental loads
   - Parameterized API loads

4. **Parameterize reusable pipelines**

   Source-specific values such as dates, years, API parameters, and watermarks should be configurable rather than unnecessarily hard-coded.

5. **Maintain source traceability**

   Source keys and relevant metadata should be preserved so that downstream records can be traced back to their originating systems.

6. **Design for repeatable execution**

   Pipelines should support controlled reruns without unintentionally creating duplicate analytical records.

7. **Fail visibly**

   Critical ingestion failures should cause the pipeline to report a failure rather than silently producing incomplete data.

8. **Preserve failed or invalid records**

   Records rejected during downstream validation should be routed to the quarantine area rather than silently discarded.

9. **Secure source connectivity**

   Credentials and secrets should not be hard-coded in pipeline definitions.

   Azure Key Vault and managed identities are used where appropriate.

10. **Maintain auditability**

    The platform should capture sufficient metadata to understand what was processed, when it was processed, and whether ingestion succeeded.

11. **Prefer simplicity initially**

    The implementation focuses on a clear, source-specific ingestion design before introducing advanced metadata-driven frameworks or unnecessary optimization.

### Source-Specific Strategy

The three source systems require different ingestion approaches:

```text
Wide World Importers
        ↓
SQL Server Extraction
        ↓
Full / Incremental Load Strategy


Open-Meteo
        ↓
Parameterized REST API
        ↓
Date-Range Ingestion


Nager.Date
        ↓
Parameterized REST API
        ↓
Year-Based Ingestion
```

---

## 2. Wide World Importers Ingestion

### Source

Microsoft Wide World Importers (WWI), running on SQL Server.

The source provides the core retail transaction, customer, product, inventory, and geographic data used by the platform.

### Ingestion Service

Azure Data Factory extracts WWI data and writes the raw source records to ADLS Gen2 Landing.

```text
Wide World Importers
        ↓
SQL Server
        ↓
Azure Data Factory
        ↓
ADLS Gen2 Landing
```

### Ingestion Approach

The ingestion design supports full and incremental extraction patterns depending on the dataset.

- Full extraction can be used for initial historical loads and reference datasets.
- Incremental extraction can be introduced for datasets with suitable change-tracking or watermark columns.
- Source identifiers should be retained for downstream traceability.

The actual extraction behavior should be determined by the source query and pipeline configuration. A full extraction should not be described as incremental unless a filter or change-tracking mechanism is implemented.

---

## 3. Open-Meteo Ingestion

### Source

Open-Meteo Historical Weather API.

The source provides daily weather observations for a configured geographic location and date range.

### Ingestion Service

Azure Data Factory calls the REST API and stores the raw API response in ADLS Gen2 Landing.

```text
Open-Meteo
    ↓
REST API
    ↓
Azure Data Factory
    ↓
ADLS Gen2 Landing
```

### Ingestion Approach

- Use configurable latitude and longitude values.
- Define the requested historical date range through parameters.
- Preserve the original API response in the Landing layer.
- Perform JSON normalization and transformation in Databricks.
- Handle missing measurements and unexpected response structures through downstream validation.

---

## 4. Nager.Date Ingestion

### Source

Nager.Date Public Holiday API.

The source provides public holiday information for a specified country and year.

### Ingestion Service

Azure Data Factory calls the REST API and stores the raw API response in ADLS Gen2 Landing.

```text
Nager.Date
    ↓
REST API
    ↓
Azure Data Factory
    ↓
ADLS Gen2 Landing
```

### Ingestion Approach

- Use configurable country and year parameters.
- Retrieve holiday information for the required historical years.
- Preserve the original API response.
- Normalize the JSON structure in Databricks.
- Validate holiday dates, country codes, and duplicate records during downstream processing.

---

## 5. ADLS Gen2 Landing Structure

The Landing layer is the raw ingestion boundary between the source systems and downstream processing layers.

Data in Landing should remain as close as practical to its original source format.

### Source-Based Organization

The Landing layer separates data by source system.

```text
landing/
├── wwi/
│   ├── orders/
│   ├── orderlines/
│   └── customers/
│
├── weather/
│
└── holidays/
```

The folder structure provides logical separation and supports source-specific processing.

Additional dataset folders can be introduced as new source entities are onboarded.

---

## 6. Load Strategy and Watermarks

The ingestion strategy uses different load patterns depending on the characteristics of each source dataset.

The platform design supports:

- Full loads
- Incremental loads
- Parameterized API loads

### 6.1 Full Load

A full load extracts the complete available dataset from the source.

Full loads are suitable for:

- Initial historical ingestion.
- Small reference datasets.
- Datasets where incremental extraction is not required or has not been configured.

Examples of reference entities include:

```text
Application.TransactionTypes
Application.Countries
Application.StateProvinces
Application.Cities
```

### 6.2 Incremental Load

An incremental load extracts records that have changed or been added since the previous successful load.

A typical watermark-based approach uses a source column such as:

- `LastModifiedWhen`
- `ModifiedDate`
- Another suitable source change-tracking column

Conceptually:

```text
Previous Successful Watermark
              ↓
       Source Extraction
              ↓
Records Modified After Watermark
              ↓
         Landing Layer
              ↓
      Validate Load Results
              ↓
     Update Watermark
```

A watermark should only advance after the corresponding ingestion is confirmed successful.

> **Implementation note:** Incremental loading is part of the ingestion design. It should only be described as implemented for a dataset when its source query, watermark handling, and successful-load behavior are present in the actual pipeline configuration.

### 6.3 API-Based Loads

External API ingestion uses configurable parameters to control the requested data.

| Source | Parameters | Load Pattern |
|---|---|---|
| Open-Meteo | Latitude, longitude, start date, end date | Date-range API request |
| Nager.Date | Country code, year | Year-based API request |

The original API response should be retained in Landing before normalization.

---

## 7. Idempotency and Duplicate Prevention

The ingestion design aims to make pipeline reruns safe and predictable.

This is particularly important when:

- A pipeline is retried after a failure.
- A historical date range is reprocessed.
- An incremental load is rerun.
- A failed downstream transformation requires reprocessing.

### Idempotency Principle

A pipeline run should produce the same logical result when the same source data and processing parameters are executed again.

```text
Same Source Data
      +
Same Parameters
      ↓
Repeat Processing
      ↓
Same Logical Dataset
```

### Duplicate Prevention

Duplicate prevention should be handled according to the source and target data grain.

Possible approaches include:

- Using stable source primary keys.
- Applying deterministic deduplication rules.
- Using Delta Lake `MERGE` operations where appropriate.
- Maintaining checkpoints for incremental file processing.
- Separating ingestion metadata from business data.

A repeated file or record should not automatically be treated as a new business record.

---

## 8. Error Handling, Retry, and Failure Management

The ingestion pipelines should distinguish between temporary technical failures and permanent data or configuration failures.

The objective is to make failures visible, diagnosable, and recoverable.

### 8.1 Transient Failures

Temporary failures may include:

- Network interruptions.
- Temporary API availability issues.
- SQL connection interruptions.
- Temporary Azure service errors.
- Request timeouts.

These failures can use controlled retry behavior where supported by Azure Data Factory.

Conceptually:

```text
Pipeline Activity
       ↓
     Failure
       ↓
      Retry
