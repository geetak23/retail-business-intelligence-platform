# Data Quality Rules

## 1. Data Quality Principles

Data quality will be validated throughout the data pipeline rather than
only at the final reporting layer.

The platform will apply data-quality checks across the Bronze, Silver,
and Gold layers.

### Core Principles

1. **Completeness**
   Required fields must be present and not unexpectedly null.

2. **Validity**
   Values must conform to the expected data type, format, and business
   rules.

3. **Uniqueness**
   Primary and business keys must not contain unexpected duplicates.

4. **Referential Integrity**
   Foreign-key relationships must resolve to the appropriate parent
   records.

5. **Consistency**
   Related datasets must remain logically consistent with each other.

6. **Accuracy of Derived Measures**
   Calculated measures must follow the documented source-to-target
   definitions.

7. **Timeliness**
   Data must be processed according to the expected ingestion schedule
   and source availability.

8. **Traceability**
   Raw source data must be preserved so that downstream records can be
   traced back to their source.

### Data Quality Handling

Records that fail critical validation rules will not silently enter the
analytics-ready Gold layer.

Depending on the rule, invalid records will either:

- Be quarantined for investigation.
- Be rejected from the current processing batch.
- Be flagged for review.
- Be allowed through when the issue is non-critical and appropriately
  documented.

Data-quality results will eventually be captured in the platform's audit
and monitoring framework.

## 2. Sales.InvoiceLines — FactInvoiceLine

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `InvoiceLineID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `InvoiceLineID` must be unique | Critical | Quarantine |
| Invoice relationship | `InvoiceID` must resolve to an invoice | Critical | Quarantine |
| Product relationship | `StockItemID` must resolve to a product | Critical | Quarantine |
| Quantity completeness | `Quantity` must not be null | Critical | Quarantine |
| Unit price completeness | `UnitPrice` must not be null | Critical | Quarantine |
| Tax rate completeness | `TaxRate` must not be null | Critical | Quarantine |
| Tax amount completeness | `TaxAmount` must not be null | Critical | Quarantine |
| Extended price completeness | `ExtendedPrice` must not be null | Critical | Quarantine |
| Line profit completeness | `LineProfit` must not be null | Critical | Quarantine |

### Financial Consistency Rules

The following relationship should hold within an acceptable numerical
tolerance:

```text
ExtendedPrice ≈ (Quantity × UnitPrice) + TaxAmount

## 3. Sales.Invoices — FactInvoice

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `InvoiceID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `InvoiceID` must be unique | Critical | Quarantine |
| Customer relationship | `CustomerID` must resolve to a customer | Critical | Quarantine |
| Invoice date completeness | `InvoiceDate` must not be null | Critical | Quarantine |
| Order relationship | When `OrderID` is populated, it must resolve to an order | Critical | Quarantine |

### Business Rules

- `InvoiceDate` must be a valid date.
- `InvoiceDate` must fall within the expected source data period.
- `IsCreditNote` must contain a valid Boolean value.
- `CustomerID` must exist in the customer source.
- `OrderID` may be null because not every invoice relationship should be
  assumed to be mandatory without source validation.

### Credit Note Handling

`IsCreditNote` will be retained even if the current source data does not
contain credit notes.

If credit-note records appear in future source loads, downstream
transformations must preserve the indicator rather than treating those
records as ordinary positive sales.

### Referential Integrity

The following relationship will be validated:

```text
FactInvoice.CustomerID
        ↓
DimCustomer.SourceCustomerID

## 4. Sales.Orders — FactOrder

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `OrderID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `OrderID` must be unique | Critical | Quarantine |
| Customer relationship | `CustomerID` must resolve to a customer | Critical | Quarantine |
| Order date completeness | `OrderDate` must not be null | Critical | Quarantine |

### Business Rules

- `OrderDate` must be a valid date.
- `ExpectedDeliveryDate`, when populated, must be a valid date.
- `ExpectedDeliveryDate` should not be earlier than `OrderDate`.
- `CustomerID` must exist in the customer source.
- `IsUndersupplyBackordered` must contain a valid Boolean value.

### Relationship Validation

The order-to-order-line relationship will be monitored:

```text
FactOrder.OrderID
       ↓
FactOrderLine.OrderID

## 5. Sales.OrderLines — FactOrderLine

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `OrderLineID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `OrderLineID` must be unique | Critical | Quarantine |
| Order relationship | `OrderID` must resolve to an order | Critical | Quarantine |
| Product relationship | `StockItemID` must resolve to a product | Critical | Quarantine |
| Quantity completeness | `Quantity` must not be null | Critical | Quarantine |
| Unit price completeness | `UnitPrice` must not be null | Critical | Quarantine |

### Business Rules

- `Quantity` should be greater than zero for normal order lines.
- `UnitPrice` should not be negative.
- `TaxRate` should not be negative.
- `PickedQuantity` should not be negative.
- `PickedQuantity` should not normally exceed `Quantity`.
- `OrderID` must exist in the order source.
- `StockItemID` must exist in the product source.

### Relationship Validation

The following relationship will be monitored:

```text
FactOrderLine.OrderID
       ↓
FactOrder.OrderID

## 6. Sales.Customers — DimCustomer

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `CustomerID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `CustomerID` must be unique in the current customer dataset | Critical | Quarantine |
| Customer name completeness | `CustomerName` must not be null or blank | Critical | Quarantine |
| Delivery city relationship | `DeliveryCityID` must resolve to a city | Critical | Quarantine |

### Business Rules

- `CustomerName` must not be an empty string.
- `AccountOpenedDate`, when populated, must be a valid date.
- `StandardDiscountPercentage` must not be negative.
- `StandardDiscountPercentage` should remain within a reasonable
  percentage range.
- `CustomerCategoryID`, when populated, must resolve to the appropriate
  source reference.
- `BuyingGroupID`, when populated, must resolve to the appropriate
  source reference.

### Geographic Relationship

The customer delivery location must resolve through the source
geographic hierarchy:

```text
Sales.Customers.DeliveryCityID
        ↓
Application.Cities.CityID
        ↓
Application.StateProvinces.StateProvinceID
        ↓
Application.Countries.CountryID

## 7. Warehouse.StockItems — DimProduct

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `StockItemID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `StockItemID` must be unique in the current product dataset | Critical | Quarantine |
| Product name completeness | `StockItemName` must not be null or blank | Critical | Quarantine |
| Unit price validity | `UnitPrice` must not be negative | Critical | Quarantine |

### Business Rules

- `StockItemName` must not be an empty string.
- `LeadTimeDays` must not be negative.
- `TaxRate` must not be negative.
- `QuantityPerOuter`, when populated, must not be negative.
- `TypicalWeightPerUnit`, when populated, must not be negative.
- `RecommendedRetailPrice`, when populated, must not be negative.
- `IsChillerStock` must contain a valid Boolean value.
- `SupplierID`, when populated, must resolve to the appropriate supplier
  source if supplier validation is included.

### Referential Integrity

Product references from transactional facts must resolve to the product
dimension:

```text
FactInvoiceLine.StockItemID
        ↓
DimProduct.SourceStockItemID

## 8. Warehouse.StockItemHoldings — FactInventorySnapshot

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Product key completeness | `StockItemID` must not be null | Critical | Quarantine |
| Product relationship | `StockItemID` must resolve to a product | Critical | Quarantine |
| Quantity completeness | `QuantityOnHand` must not be null | Critical | Quarantine |
| Cost validity | `LastCostPrice` must not be negative | Critical | Quarantine |

### Business Rules

- `QuantityOnHand` should not be negative unless a negative inventory
  position is explicitly supported by the source business process.
- `LastStocktakeQuantity` should not be negative.
- `LastCostPrice` should not be negative.
- `ReorderLevel` should not be negative.
- `TargetStockLevel` should not be negative.
- `BinLocation`, when populated, should be a valid source value.

### Inventory Consistency Checks

The following derived value will be monitored:

```text
InventoryValue =
    QuantityOnHand × LastCostPrice

    ## 9. Warehouse.StockItemTransactions — FactInventoryTransaction

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `StockItemTransactionID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `StockItemTransactionID` must be unique | Critical | Quarantine |
| Product relationship | `StockItemID` must resolve to a product | Critical | Quarantine |
| Transaction type relationship | `TransactionTypeID` must resolve to a transaction type | Critical | Quarantine |
| Transaction timestamp completeness | `TransactionOccurredWhen` must not be null | Critical | Quarantine |
| Quantity completeness | `Quantity` must not be null | Critical | Quarantine |

### Business Rules

- `StockItemID` must exist in the product source.
- `TransactionTypeID` must exist in the transaction-type source.
- `TransactionOccurredWhen` must be a valid timestamp.
- `Quantity` must be a valid numeric value.
- `InvoiceID`, when populated, must resolve to an invoice.
- `CustomerID`, when populated, must resolve to a customer.
- `SupplierID`, when populated, must resolve to the appropriate supplier
  source when supplier validation is implemented.
- `PurchaseOrderID`, when populated, must resolve to the appropriate
  purchase-order source when purchase-order validation is implemented.

### Transaction-Type Validation

The transaction type must be resolved through
`DimTransactionType`.

The pipeline must not hard-code transaction type names.

```text
FactInventoryTransaction.TransactionTypeKey
                ↓
       DimTransactionType

       ## 10. Application.TransactionTypes — DimTransactionType

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Primary key completeness | `TransactionTypeID` must not be null | Critical | Quarantine |
| Primary key uniqueness | `TransactionTypeID` must be unique | Critical | Quarantine |
| Transaction name completeness | `TransactionTypeName` must not be null or blank | Critical | Quarantine |

### Business Rules

- `TransactionTypeID` must be a valid integer.
- `TransactionTypeName` must not be an empty string.
- Each source transaction type should have one corresponding analytical
  classification.
- Analytical classifications must be documented and reproducible.

### Referential Integrity

Every transaction in
`Warehouse.StockItemTransactions` must resolve to a valid transaction
type:

```text
FactInventoryTransaction.TransactionTypeKey
                ↓
       DimTransactionType

       ## 11. Geography — DimGeography

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| City key completeness | `CityID` must not be null | Critical | Quarantine |
| City key uniqueness | `CityID` must be unique | Critical | Quarantine |
| City name completeness | `CityName` must not be null or blank | Critical | Quarantine |
| State relationship | `StateProvinceID` must resolve to a state/province | Critical | Quarantine |
| Country relationship | `CountryID` must resolve to a country | Critical | Quarantine |

### Business Rules

- `CityName` must not be an empty string.
- `StateProvinceName` must not be null or blank.
- `CountryName` must not be null or blank.
- `StateProvinceCode`, when populated, should be valid for the source
  record.
- Country identifiers must remain consistent with their associated
  state/province records.

### Geographic Relationship Validation

The complete hierarchy must resolve:

```text
City
 ↓
State / Province
 ↓
Country

## 12. Open-Meteo — DimWeather

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Date completeness | `WeatherDate` must not be null | Critical | Quarantine |
| Date uniqueness | One record per location/date combination | Critical | Quarantine |
| Maximum temperature validity | `TemperatureMax` must be numeric when provided | Critical | Quarantine |
| Minimum temperature validity | `TemperatureMin` must be numeric when provided | Critical | Quarantine |
| Precipitation validity | `Precipitation` must be numeric and non-negative | Critical | Quarantine |
| Rainfall validity | `Rainfall` must be numeric and non-negative | Critical | Quarantine |
| Wind speed validity | `WindSpeedMax` must be numeric and non-negative | Critical | Quarantine |

### Business Rules

- `WeatherDate` must be a valid calendar date.
- `TemperatureMax` should not be lower than `TemperatureMin`.
- `Precipitation` must not be negative.
- `Rainfall` must not be negative.
- `WindSpeedMax` must not be negative.
- `Latitude` and `Longitude` must remain consistent for the configured
  weather location.
- `Timezone` must remain consistent with the configured API request.

### JSON Normalization Validation

The source response contains parallel arrays under `daily`.

The pipeline must verify that all required daily arrays contain the same
number of elements before converting them into rows.

For example:

```text
daily.time
daily.temperature_2m_max
daily.temperature_2m_min
daily.precipitation_sum
daily.rain_sum
daily.wind_speed_10m_max

## 13. Nager.Date — DimHoliday

### Critical Rules

| Rule | Validation | Severity | Action |
|---|---|---|---|
| Holiday date completeness | `HolidayDate` must not be null | Critical | Quarantine |
| Holiday name completeness | `HolidayName` must not be null or blank | Critical | Quarantine |
| Country completeness | `CountryCode` must not be null or blank | Critical | Quarantine |
| Date validity | `HolidayDate` must be a valid calendar date | Critical | Quarantine |

### Business Rules

- `CountryCode` must contain a valid country code.
- `IsFixed` must contain a valid Boolean value.
- `IsGlobal` must contain a valid Boolean value.
- `LaunchYear`, when populated, must be a valid year.
- `HolidayTypes` must be handled as an array and must not be silently
  discarded.
- `ApplicableRegions`, when populated, must be handled as an array.

### Duplicate Handling

Multiple holiday records may legitimately exist for the same date.

Therefore:

```text
Same HolidayDate
        ≠
Automatic duplicate
## 14. Cross-Dataset Data Quality

The following validations will be performed across datasets to ensure
that the analytical model remains consistent.

### Referential Integrity

The following relationships must resolve successfully:

```text
FactOrder.CustomerKey
        ↓
DimCustomer

FactOrderLine.OrderID
        ↓
FactOrder

FactOrderLine.ProductKey
        ↓
DimProduct

FactInvoice.CustomerKey
        ↓
DimCustomer

FactInvoice.OrderID
        ↓
FactOrder

FactInvoiceLine.InvoiceID
        ↓
FactInvoice

FactInvoiceLine.ProductKey
        ↓
DimProduct

FactInventorySnapshot.ProductKey
        ↓
DimProduct

FactInventoryTransaction.ProductKey
        ↓
DimProduct

FactInventoryTransaction.TransactionTypeKey
        ↓
DimTransactionType

