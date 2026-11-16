# Data Quality Rules

## 1. Purpose and Implementation Scope

This document defines the data-quality principles and validation rules for the Retail Business Intelligence Data Platform.

Data quality is considered throughout the pipeline rather than only at the final reporting layer. The framework covers validation across the Bronze, Silver, and Gold layers.

> **Implementation scope:** The currently implemented Gold model consists of `FactSales`, `DimDate`, `DimCustomer`, and `AggDailySales`. The rules for these tables describe the current analytical model. The additional fact and dimension tables in this document represent proposed analytical model extensions. Including a rule here does not mean that the corresponding table or validation has already been implemented or executed.

All rules should be confirmed against the actual source schema, target table definitions, notebook transformations, and business requirements before being treated as automated controls.

## 2. Data Quality Principles

### Core Principles

1. **Completeness**  
   Required fields must be present and not unexpectedly null.

2. **Validity**  
   Values must conform to expected data types, formats, and business rules.

3. **Uniqueness**  
   Primary and business keys must not contain unexpected duplicates.

4. **Referential Integrity**  
   Foreign-key relationships must resolve to the appropriate parent records.

5. **Consistency**  
   Related datasets must remain logically consistent with each other.

6. **Accuracy of Derived Measures**  
   Calculated measures must follow documented source-to-target definitions.

7. **Timeliness**  
   Data must be processed according to the expected ingestion schedule and source availability.

8. **Traceability**  
   Source data and ingestion metadata should support tracing downstream records back to their origin.

### Data Quality Severity Levels

| Severity | Description | Typical Handling |
|---|---|---|
| Critical | A failure affects record identity, required data, or a mandatory relationship | Quarantine or reject affected records |
| Warning | A potential issue requires review but does not necessarily prevent processing | Flag the record and capture the issue |
| Informational | An expected condition or useful operational observation | Record for monitoring |

### Data Quality Handling

Records that fail critical validation rules should not silently enter the analytics-ready Gold layer.

Depending on the rule and its business impact, records may be:

- Quarantined for investigation.
- Rejected from the current processing batch.
- Flagged for review.
- Allowed through when the issue is non-critical, expected, and documented.

The severity and handling approach should reflect the source contract and business requirements. For example, a missing primary key is generally critical, while a missing optional reference may be acceptable.

### Validation Results and Auditability

Data-quality results can be captured in the platform's audit and monitoring framework.

Useful attributes include:

- Pipeline or notebook name.
- Source dataset and target table.
- Rule identifier and description.
- Execution timestamp or batch identifier.
- Number of records evaluated.
- Number of records passed and failed.
- Failure severity and handling outcome.
- Source file or ingestion metadata, where available.

---

## 3. Sales.InvoiceLines — FactInvoiceLine

*Proposed analytical model extension.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `InvoiceLineID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `InvoiceLineID` must be unique at the target grain | Critical | Quarantine duplicates |
| Invoice relationship | `InvoiceID` must resolve to an invoice | Critical | Quarantine unresolved records |
| Product relationship | `StockItemID` must resolve to a product | Critical | Quarantine unresolved records |
| Quantity completeness | `Quantity` must not be null | Critical | Quarantine |
| Unit price completeness | `UnitPrice` must not be null | Critical | Quarantine |
| Tax rate completeness | `TaxRate` must be populated when required by the source contract | Warning or Critical | Flag or quarantine |
| Tax amount completeness | `TaxAmount` must be populated when required by the source contract | Warning or Critical | Flag or quarantine |
| Extended price completeness | `ExtendedPrice` must not be null | Critical | Quarantine |
| Line profit completeness | `LineProfit` must be populated when required by the source contract | Warning or Critical | Flag or quarantine |

### Financial Consistency Rules

The following financial relationship should hold within an acceptable numerical tolerance:

```text
NetLineAmountBeforeTax = Quantity × UnitPrice

ExtendedPrice ≈ NetLineAmountBeforeTax + TaxAmount
```

Validation should account for the source system's decimal precision and rounding behavior.

Additional checks:

- `Quantity` should be greater than zero for normal invoice lines. Credit notes and adjustments should be evaluated according to source business rules.
- `UnitPrice` should not be negative unless a documented source process permits it.
- `TaxRate` should fall within the valid range defined by the source.
- `TaxAmount` and `ExtendedPrice` must be numeric.

---

## 4. Sales.Invoices — FactInvoice

*Proposed analytical model extension.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `InvoiceID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `InvoiceID` must be unique | Critical | Quarantine duplicates |
| Customer relationship | `CustomerID` must resolve to a customer | Critical | Quarantine unresolved records |
| Invoice date completeness | `InvoiceDate` must not be null | Critical | Quarantine |
| Order relationship | When `OrderID` is populated, it must resolve to an order | Critical | Quarantine unresolved records |

### Business Rules

- `InvoiceDate` must be a valid date.
- `InvoiceDate` should fall within the expected source data period.
- `IsCreditNote` must conform to the source data type and accepted Boolean representation.
- `CustomerID` must resolve to a customer.
- The nullability and mandatory status of `OrderID` should be confirmed against the actual source schema before treating it as optional or required.

### Credit Note Handling

`IsCreditNote` should be retained even if the current source data does not contain credit notes.

If credit-note records appear in future source loads, downstream transformations should preserve the indicator rather than automatically treating those records as ordinary positive sales.

### Referential Integrity

The proposed source-to-dimension relationship is:

```text
FactInvoice.CustomerID
        ↓
DimCustomer.SourceCustomerID
```

If a surrogate customer key is generated in the Gold model, validate the source-to-surrogate key mapping separately.

---

## 5. Sales.Orders — FactOrder

*Proposed analytical model extension.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `OrderID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `OrderID` must be unique | Critical | Quarantine duplicates |
| Customer relationship | `CustomerID` must resolve to a customer | Critical | Quarantine unresolved records |
| Order date completeness | `OrderDate` must not be null | Critical | Quarantine |

### Business Rules

- `OrderDate` must be a valid date.
- `ExpectedDeliveryDate`, when populated, must be a valid date.
- `ExpectedDeliveryDate` should not be earlier than `OrderDate`; exceptions should be reviewed.
- `CustomerID` must resolve to a customer.
- `IsUndersupplyBackordered` must conform to the source data type and accepted Boolean representation.

### Relationship Validation

The order-to-order-line relationship should be validated:

```text
FactOrder.OrderID
       ↓
FactOrderLine.OrderID
```

Every order line should reference a valid order.

---

## 6. Sales.OrderLines — FactOrderLine

*Proposed analytical model extension.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `OrderLineID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `OrderLineID` must be unique | Critical | Quarantine duplicates |
| Order relationship | `OrderID` must resolve to an order | Critical | Quarantine unresolved records |
| Product relationship | `StockItemID` must resolve to a product | Critical | Quarantine unresolved records |
| Quantity completeness | `Quantity` must not be null | Critical | Quarantine |
| Unit price completeness | `UnitPrice` must not be null | Critical | Quarantine |

### Business Rules

- `Quantity` should be greater than zero for normal order lines.
- `UnitPrice` should not be negative unless a documented source process permits it.
- `TaxRate` should follow the valid range defined by the source.
- `PickedQuantity` should not be negative.
- `PickedQuantity` should not normally exceed `Quantity`; exceptions should be reviewed against source business processes.
- `OrderID` must resolve to an order.
- `StockItemID` must resolve to a product.

### Relationship Validation

```text
FactOrderLine.OrderID
       ↓
FactOrder.OrderID
```

---

## 7. Sales.Customers — DimCustomer

*Current Gold model: implemented dimension. Confirm exact target column names against the notebook output.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Customer key completeness | Source `CustomerID` must not be null | Critical | Quarantine |
| Customer key uniqueness | Source `CustomerID` must be unique at the selected dimension grain | Critical | Resolve duplicates |
| Customer name completeness | `CustomerName` must not be null or blank | Critical | Quarantine or flag |
| Delivery city relationship | `DeliveryCityID` must resolve to a city when required by the model | Critical | Quarantine or flag |

### Business Rules

- `CustomerName` must not be an empty string.
- `AccountOpenedDate`, when populated, must be a valid date.
- `StandardDiscountPercentage` must not be negative.
- `StandardDiscountPercentage` should remain within the range defined by the source business rules.
- `CustomerCategoryID`, when populated, should resolve to the appropriate source reference.
- `BuyingGroupID`, when populated, should resolve to the appropriate source reference.

### Geographic Relationship

When geographic validation is included in the model, the customer delivery location should resolve through the source geographic hierarchy:

```text
Sales.Customers.DeliveryCityID
        ↓
Application.Cities.CityID
        ↓
Application.StateProvinces.StateProvinceID
        ↓
Application.Countries.CountryID
```

Optional geographic relationships should be handled according to the source schema and business requirements.

### Key Design

Retain the source customer identifier for traceability. If a surrogate key is generated in Gold, document and validate the mapping between the source identifier and the surrogate key.

---

## 8. Warehouse.StockItems — DimProduct

*Proposed analytical model extension.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `StockItemID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `StockItemID` must be unique | Critical | Quarantine duplicates |
| Product name completeness | `StockItemName` must not be null or blank | Critical | Quarantine |
| Unit price validity | `UnitPrice` must follow the source-defined valid range | Warning or Critical | Flag or quarantine |

### Business Rules

- `StockItemName` must not be an empty string.
- `LeadTimeDays` should not be negative.
- `TaxRate` should follow the valid range defined by the source.
- `QuantityPerOuter`, when populated, should not be negative.
- `TypicalWeightPerUnit`, when populated, should not be negative.
- `RecommendedRetailPrice`, when populated, should not be negative.
- `IsChillerStock` must conform to the source data type and accepted Boolean representation.
- `SupplierID`, when populated, should resolve to the appropriate supplier source if supplier validation is included.

### Referential Integrity

Product references from transactional facts should resolve to the product dimension:

```text
FactInvoiceLine.StockItemID
        ↓
DimProduct.SourceStockItemID
```

---

## 9. Warehouse.StockItemHoldings — FactInventorySnapshot

*Proposed analytical model extension.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Product key completeness | `StockItemID` must not be null | Critical | Quarantine |
| Product relationship | `StockItemID` must resolve to a product | Critical | Quarantine |
| Quantity completeness | `QuantityOnHand` must not be null | Critical | Quarantine |
| Cost validity | `LastCostPrice` must not be negative | Warning or Critical | Flag or quarantine |

### Business Rules

- `QuantityOnHand` should not be negative unless a negative inventory position is explicitly supported by the source business process.
- `LastStocktakeQuantity` should not be negative unless permitted by the source.
- `LastCostPrice` should not be negative.
- `ReorderLevel` should not be negative.
- `TargetStockLevel` should not be negative.
- `BinLocation`, when populated, should follow the expected source format.

### Inventory Consistency Checks

The following derived value can be monitored when it is part of the analytical model:

```text
InventoryValue = QuantityOnHand × LastCostPrice
```

The valuation definition and treatment of null values should be documented.

---

## 10. Warehouse.StockItemTransactions — FactInventoryTransaction

*Proposed analytical model extension.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `StockItemTransactionID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `StockItemTransactionID` must be unique | Critical | Quarantine duplicates |
| Product relationship | `StockItemID` must resolve to a product | Critical | Quarantine |
| Transaction type relationship | `TransactionTypeID` must resolve to a transaction type | Critical | Quarantine |
| Transaction timestamp completeness | `TransactionOccurredWhen` must not be null | Critical | Quarantine |
| Quantity completeness | `Quantity` must not be null | Critical | Quarantine |

### Business Rules

- `StockItemID` must resolve to the product source.
- `TransactionTypeID` must resolve to the transaction-type source.
- `TransactionOccurredWhen` must be a valid timestamp.
- `Quantity` must be numeric.
- `InvoiceID`, when populated, should resolve to an invoice.
- `CustomerID`, when populated, should resolve to a customer.
- `SupplierID`, when populated, should resolve to the appropriate supplier source when supplier validation is implemented.
- `PurchaseOrderID`, when populated, should resolve to the appropriate purchase-order source when purchase-order validation is implemented.

### Transaction-Type Validation

Transaction types should be resolved through `DimTransactionType`. Transaction type names should not be hard-coded into transformation logic.

```text
FactInventoryTransaction.TransactionTypeID
                ↓
       DimTransactionType.TransactionTypeID
```

If the target model uses a surrogate transaction-type key, validate the source-to-surrogate mapping separately.

---

## 11. Application.TransactionTypes — DimTransactionType

*Proposed analytical model extension.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `TransactionTypeID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `TransactionTypeID` must be unique | Critical | Quarantine duplicates |
| Transaction name completeness | `TransactionTypeName` must not be null or blank | Critical | Quarantine |

### Business Rules

- `TransactionTypeID` must conform to the source data type.
- `TransactionTypeName` must not be an empty string.
- Each source transaction type should have one corresponding analytical classification when classification is part of the model.
- Analytical classifications must be documented and reproducible.

### Referential Integrity

Every transaction in `Warehouse.StockItemTransactions` should resolve to a valid transaction type.

```text
FactInventoryTransaction.TransactionTypeID
                ↓
       DimTransactionType.TransactionTypeID
```

---

## 12. Geography — DimGeography

*Proposed analytical model extension.*

Source entities:

- `Application.Cities`
- `Application.StateProvinces`
- `Application.Countries`

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| City key completeness | `CityID` must not be null | Critical | Quarantine |
| City key uniqueness | `CityID` must be unique | Critical | Quarantine duplicates |
| City name completeness | `CityName` must not be null or blank | Critical | Quarantine |
| State relationship | `StateProvinceID` must resolve to a state or province when required | Critical | Quarantine |
| Country relationship | `CountryID` must resolve to a country when required | Critical | Quarantine |

### Business Rules

- `CityName` must not be an empty string.
- `StateProvinceName` must not be null or blank when required.
- `CountryName` must not be null or blank when required.
- `StateProvinceCode`, when populated, should conform to the expected source format.
- Country identifiers must remain consistent with their associated state or province records.

### Geographic Relationship Validation

The geographic hierarchy should resolve consistently:

```text
City
  ↓
State / Province
  ↓
Country
```

---

## 13. Open-Meteo — DimWeather

*External data source. Validate against the API response contract and the fields actually retained in Silver or Gold.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Date completeness | `WeatherDate` must not be null | Critical | Quarantine |
| Date uniqueness | The location/date combination must be unique at the selected grain | Critical | Resolve duplicates |
| Maximum temperature validity | `TemperatureMax` must be numeric when provided | Warning | Flag invalid values |
| Minimum temperature validity | `TemperatureMin` must be numeric when provided | Warning | Flag invalid values |
| Precipitation validity | `Precipitation` must be numeric and non-negative when provided | Warning | Flag invalid values |
| Rainfall validity | `Rainfall` must be numeric and non-negative when provided | Warning | Flag invalid values |
| Wind speed validity | `WindSpeedMax` must be numeric and non-negative when provided | Warning | Flag invalid values |

### Business Rules

- `WeatherDate` must be a valid calendar date.
- `TemperatureMax` should not be lower than `TemperatureMin` when both are populated.
- `Precipitation` and `Rainfall` should not be negative.
- `WindSpeedMax` should not be negative.
- `Latitude` and `Longitude` should remain consistent with the configured weather location.
- `Timezone` should remain consistent with the configured API request.
- Missing individual weather measurements should be handled as nullable values when permitted by the API contract.

### JSON Normalization Validation

The source response contains parallel arrays under `daily`.

Before converting the arrays into rows, the pipeline should verify that the required arrays have compatible lengths.

Examples include:

```text
daily.time
daily.temperature_2m_max
daily.temperature_2m_min
daily.precipitation_sum
daily.rain_sum
daily.wind_speed_10m_max
```

The normalization process should preserve the positional relationship between each date and its corresponding measurements.

---

## 14. Nager.Date — DimHoliday

*External data source. Validate against the API response contract and the fields actually retained in Silver or Gold.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Holiday date completeness | `HolidayDate` must not be null | Critical | Quarantine |
| Holiday name completeness | `HolidayName` must not be null or blank when required | Critical | Quarantine |
| Country completeness | `CountryCode` must not be null or blank | Critical | Quarantine |
| Date validity | `HolidayDate` must be a valid calendar date | Critical | Quarantine |

### Business Rules

- `CountryCode` must conform to the expected country-code format.
- `IsFixed` must conform to the API's Boolean representation.
- `IsGlobal` must conform to the API's Boolean representation.
- `LaunchYear`, when populated, must be a valid year.
- `HolidayTypes` should be handled as an array and must not be silently discarded if required by the model.
- `ApplicableRegions`, when populated, should be handled according to the source API structure.

### Duplicate Handling

Multiple holiday records may legitimately exist for the same date. Therefore, a matching date alone does not establish that records are duplicates.

```text
Same HolidayDate
        ≠
Automatic duplicate
```

Duplicate detection should consider the country, date, holiday identity, and any applicable regional attributes.

---

## 15. Generated Calendar — DimDate

*Current Gold model: implemented dimension. Confirm exact target column names against the notebook output.*

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Date key completeness | Date key must not be null | Critical | Reject invalid output |
| Date uniqueness | Each calendar date must appear once | Critical | Resolve duplicates |
| Date validity | Each date must be a valid calendar date | Critical | Reject invalid output |
| Calendar continuity | Dates should be continuous across the configured range | Warning | Flag missing dates |

### Business Rules

- Year, quarter, month, and day attributes must be consistent with the date key.
- Month and weekday labels should use a consistent naming convention.
- Fiscal attributes, if included, must follow a documented fiscal calendar.
- The calendar range should cover the date range required by the fact data and reporting use cases.

---

## 16. Current Gold Model — FactSales

*Currently implemented Gold fact table. Confirm exact column names and grain against the Gold notebook and table schema.*

### Grain and Key Rules

- The intended grain is one row per sales order line.
- The source order-line identifier, or documented equivalent, should uniquely identify a fact row.
- Required customer and date references should resolve to their corresponding dimensions.
- Duplicate fact rows should be investigated using the source business key, not removed solely because displayed values happen to match.

### Completeness and Validity Rules

- Required order-line identifiers must not be null.
- Required customer and order/date references must be populated.
- Quantity and unit-price measures must be numeric.
- Derived financial measures must follow the documented calculation and source precision.
- Date keys must resolve to `DimDate`.
- Customer keys or source customer identifiers must resolve to `DimCustomer`, according to the actual Gold model design.

### Reconciliation

- Compare the Gold fact row count with the expected source order-line population after accounting for documented filters and exclusions.
- Reconcile selected totals, such as quantity and sales amount, with the Silver source data using consistent inclusion rules.
- Record expected differences caused by filtering, deduplication, or business transformations.

---

## 17. Current Gold Model — AggDailySales

*Currently implemented Gold aggregate table.*

### Grain and Key Rules

- The intended grain should be one row per defined aggregation key, such as sales date and any additional grouping dimensions included in the table.
- The aggregation key combination must be unique.
- Date and customer references, where included, must resolve to their corresponding dimensions.

### Aggregation Rules

- Aggregate measures must be calculated from the corresponding `FactSales` records.
- Sales totals should use the same financial definition and filters as the underlying fact.
- Counts and quantities must use documented aggregation logic.
- Null grouping keys should be handled consistently and should not silently create misleading aggregate categories.

### Reconciliation

- Recalculate selected daily totals from `FactSales` and compare them with `AggDailySales`.
- Confirm that the aggregate covers the expected fact date range.
- Investigate differences caused by missing keys, filtering, or inconsistent calculation logic.

---

## 18. Cross-Dataset Data Quality

The following validations help ensure consistency across the analytical model.

### Referential Integrity

For the proposed extended model, the following relationships should be validated using the actual source identifiers or documented surrogate-key mappings:

```text
FactOrder.CustomerID
        ↓
DimCustomer.SourceCustomerID

FactOrderLine.OrderID
        ↓
FactOrder.OrderID

FactOrderLine.StockItemID
        ↓
DimProduct.SourceStockItemID

FactInvoice.CustomerID
        ↓
DimCustomer.SourceCustomerID

FactInvoice.OrderID
        ↓
FactOrder.OrderID

FactInvoiceLine.InvoiceID
        ↓
FactInvoice.InvoiceID

FactInvoiceLine.StockItemID
        ↓
DimProduct.SourceStockItemID

FactInventorySnapshot.StockItemID
        ↓
DimProduct.SourceStockItemID

FactInventoryTransaction.StockItemID
        ↓
DimProduct.SourceStockItemID

FactInventoryTransaction.TransactionTypeID
        ↓
DimTransactionType.TransactionTypeID
```

If the Gold model uses surrogate keys, the source-to-surrogate mappings should be validated separately. Source identifiers and surrogate keys should not be treated as interchangeable.

### Ingestion and File Validation

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Source availability | Expected source or file is available for the ingestion run | Warning or Critical | Log or fail according to pipeline requirements |
| File traceability | Source file name or source identifier is retained where practical | Warning | Record missing lineage metadata |
| File format | File content matches the expected format and parsing configuration | Critical | Quarantine malformed input |
| Schema compatibility | Incoming schema is compatible with the expected source contract | Critical or Warning | Fail, quarantine, or review schema changes |
| Incremental processing | Checkpoint or watermark behavior prevents unintended reprocessing or data loss | Critical | Stop and investigate |
| Duplicate ingestion | Repeated source files or records are handled according to the ingestion design | Critical or Warning | Deduplicate or flag using stable identifiers |

### Layer-to-Layer Validation

- Compare source, Bronze, Silver, and Gold record counts where the transformation is expected to preserve records.
- Document expected count differences caused by filtering, deduplication, invalid-record handling, or aggregation.
- Confirm that required columns and data types are preserved or intentionally transformed.
- Verify that source lineage metadata is available where required.
- Ensure quarantined records are excluded from downstream analytical outputs unless explicitly remediated.

### Pipeline Operational Checks

- Capture pipeline and notebook execution status.
- Record processing timestamps and relevant batch identifiers.
- Track input and output record counts.
- Record failed activities and error details.
- Monitor unexpected changes in data volume, schema, or processing duration.
- Do not interpret a successful pipeline status alone as proof that business data is correct.

---

## 19. Recorded Validation Outcomes

The following are recorded development results for the current project. These are historical results and are not a claim that the pipeline was freshly rerun during this documentation review.

| Dataset / Table | Recorded Count |
|---|---:|
| Orders | 73,595 |
| OrderLines | 231,412 |
| Customers | 663 |
| DimDate | 1,247 |
| FactSales | 231,412 |
| AggDailySales | 1,069 |
| Weather Silver | 5 |
| Holiday Silver | 17 |
| Holiday Bronze | 34 |
| Data-quality result rows | 12 |

Recorded checks reported zero issues for the checked categories:

- Duplicate records.
- Null values in the checked required fields.
- Invalid values covered by the checks.
- Foreign-key violations covered by the checks.

> These results apply only to the validation checks that were actually executed and recorded. They do not establish that every rule in this document has been implemented or tested.

---

## 20. Implementation and Maintenance Guidance

Before implementing a rule as an automated check:

1. Confirm the source column name, data type, nullability, and constraints.
2. Confirm the target table's actual grain and key design.
3. Confirm whether the target uses source identifiers, surrogate keys, or both.
4. Define the business reason for the rule and its severity.
5. Define the expected response to a validation failure.
6. Test the rule against representative valid and invalid data.
7. Record the rule identifier, result counts, and execution context.
8. Review the rule when source schemas or business requirements change.

### Important Modeling Considerations

- Source identifiers and Gold surrogate keys are not interchangeable.
- Retain source identifiers where practical for traceability.
- If a surrogate key is generated, document how it maps to the source identifier.
- Confirm actual source nullability before treating a field as mandatory or optional.
- Avoid treating all negative quantities or missing optional attributes as invalid without considering source business processes.
- Apply deterministic deduplication rules rather than selecting arbitrary records.
- Keep proposed model extensions separate from the currently implemented Gold model.

The proposed extensions include `FactInvoiceLine`, `FactInvoice`, `FactOrder`, `FactOrderLine`, `DimProduct`, `FactInventorySnapshot`, `FactInventoryTransaction`, `DimTransactionType`, `DimGeography`, `DimWeather`, and `DimHoliday`.

---

## 21. Summary

This framework establishes a consistent approach to validating completeness, validity, uniqueness, referential integrity, financial consistency, and pipeline traceability.

It distinguishes the current Gold implementation from proposed analytical model extensions and separates recorded development outcomes from validation rules that have not necessarily been executed.

The framework provides a foundation for maintaining data quality and extending automated validation as the platform evolves.
