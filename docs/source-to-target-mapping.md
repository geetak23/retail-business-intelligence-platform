# Source-to-Target Mapping

## Sales.InvoiceLines → FactInvoiceLine

### Target Grain

One row per invoice line.

### Business Purpose

Represents invoiced retail activity and provides the primary source
for sales amount, tax, quantity, and line-profit analysis.

### Column Mapping

| Source Table | Source Column | Target Column | Transformation | Business Meaning |
|---|---|---|---|---|
| `Sales.InvoiceLines` | `InvoiceLineID` | `InvoiceLineID` | Direct | Unique invoice-line identifier |
| `Sales.InvoiceLines` | `InvoiceID` | `InvoiceID` | Direct | Invoice associated with the line |
| `Sales.InvoiceLines` | `StockItemID` | `StockItemID` | Direct | Product associated with the line |
| `Sales.InvoiceLines` | `Quantity` | `Quantity` | Direct | Quantity invoiced |
| `Sales.InvoiceLines` | `UnitPrice` | `UnitPrice` | Direct | Selling price per unit before tax |
| `Sales.InvoiceLines` | `TaxRate` | `TaxRate` | Direct | Tax rate applied to the line |
| `Sales.InvoiceLines` | `TaxAmount` | `TaxAmount` | Direct | Tax amount for the line |
| `Sales.InvoiceLines` | `ExtendedPrice` | `ExtendedPrice` | Direct | Extended line amount including tax |
| `Sales.InvoiceLines` | `LineProfit` | `LineProfit` | Direct | Profit associated with the invoice line |
| `Sales.InvoiceLines` | `LastEditedWhen` | `SourceLastEditedWhen` | Direct | Source modification timestamp |

### Derived Columns

| Target Column | Derivation | Purpose |
|---|---|---|
| `NetLineAmountBeforeTax` | `ExtendedPrice - TaxAmount` | Line amount before tax |

### Financial Interpretation

Based on validation of the source data:

```text
Quantity × UnitPrice
        =
NetLineAmountBeforeTax

NetLineAmountBeforeTax + TaxAmount
        =
ExtendedPrice

## Sales.Invoices → FactInvoice

### Target Grain

One row per invoice.

### Business Purpose

Represents the invoice-level business event associated with invoiced
retail activity.

The invoice fact provides invoice-level dates and business status,
while `FactInvoiceLine` provides the detailed financial measures.

### Column Mapping

| Source Table | Source Column | Target Column | Transformation | Business Meaning |
|---|---|---|---|---|
| `Sales.Invoices` | `InvoiceID` | `InvoiceID` | Direct | Unique invoice identifier |
| `Sales.Invoices` | `CustomerID` | `CustomerID` | Direct | Customer associated with the invoice |
| `Sales.Invoices` | `BillToCustomerID` | `BillToCustomerID` | Direct | Customer receiving the invoice |
| `Sales.Invoices` | `OrderID` | `OrderID` | Direct | Order associated with the invoice |
| `Sales.Invoices` | `InvoiceDate` | `InvoiceDate` | Direct | Date of the invoice |
| `Sales.Invoices` | `IsCreditNote` | `IsCreditNote` | Direct | Indicates whether the invoice is a credit note |
| `Sales.Invoices` | `LastEditedWhen` | `SourceLastEditedWhen` | Direct | Source modification timestamp |

### Optional Source Fields

The following fields may be included later if required by reporting
or operational analysis:

- `TotalDryItems`
- `TotalChillerItems`
- `CreditNoteReason`

### Source Fields Intentionally Not Included Initially

The following fields are not required for the initial analytical fact:

- `DeliveryMethodID`
- `ContactPersonID`
- `AccountsPersonID`
- `SalespersonPersonID`
- `PackedByPersonID`
- `CustomerPurchaseOrderNumber`
- `Comments`
- `DeliveryInstructions`
- `InternalComments`
- `DeliveryRun`
- `RunPosition`
- `ReturnedDeliveryData`
- `LastEditedBy`

These fields can be incorporated later when a specific business
requirement justifies them.

### Important Source Finding

The validated source contains:

- 70,510 invoices
- 70,510 invoices with invoice lines
- 0 invoices without invoice lines

The source also currently contains:

- `IsCreditNote = 0` for all 70,510 invoices.

The `IsCreditNote` field will nevertheless be retained because it is
part of the source business model and should not be discarded merely
because the current sample contains no credit notes.

### Analytical Role

`FactInvoice` represents invoice-level events.

`FactInvoiceLine` represents the detailed financial activity associated
with each invoice.

The two facts will remain separate rather than duplicating invoice-level
attributes across every invoice line.

## Sales.Orders → FactOrder

### Target Grain

One row per order.

### Business Purpose

Represents the operational order event before invoicing.

This fact preserves order activity separately from invoiced financial
activity.

### Column Mapping

| Source Table | Source Column | Target Column | Transformation | Business Meaning |
|---|---|---|---|---|
| `Sales.Orders` | `OrderID` | `OrderID` | Direct | Unique order identifier |
| `Sales.Orders` | `CustomerID` | `CustomerID` | Direct | Customer associated with the order |
| `Sales.Orders` | `SalespersonPersonID` | `SalespersonPersonID` | Direct | Salesperson associated with the order |
| `Sales.Orders` | `OrderDate` | `OrderDate` | Direct | Date the order was placed |
| `Sales.Orders` | `ExpectedDeliveryDate` | `ExpectedDeliveryDate` | Direct | Expected delivery date |
| `Sales.Orders` | `IsUndersupplyBackordered` | `IsUndersupplyBackordered` | Direct | Indicates whether undersupply was backordered |
| `Sales.Orders` | `LastEditedWhen` | `SourceLastEditedWhen` | Direct | Source modification timestamp |

### Source Fields Intentionally Not Included Initially

The following fields are excluded from the initial analytical fact:

- `PickedByPersonID`
- `ContactPersonID`
- `BackorderOrderID`
- `CustomerPurchaseOrderNumber`
- `Comments`
- `DeliveryInstructions`
- `InternalComments`
- `PickingCompletedWhen`
- `LastEditedBy`

These can be added later if a specific analytical requirement
justifies them.

### Important Source Finding

The validated source contains:

- 73,595 orders
- 73,595 orders with at least one order line
- 70,510 orders with an associated invoice
- 3,085 orders without an associated invoice

Orders without invoices will remain in `FactOrder`.

They will not be artificially converted into invoice or sales-line
records.

### Analytical Role

`FactOrder` represents the order lifecycle and operational activity.

`FactOrderLine` represents detailed ordered-item activity.

`FactInvoice` represents invoice-level activity.

`FactInvoiceLine` represents detailed invoiced financial activity.

These facts represent different business grains and should remain
separate.
## Sales.OrderLines → FactOrderLine

### Target Grain

One row per order line.

### Business Purpose

Represents detailed operational order activity at the product level.

This fact supports analysis of ordered quantities and order behavior
before invoicing.

### Column Mapping

| Source Table | Source Column | Target Column | Transformation | Business Meaning |
|---|---|---|---|---|
| `Sales.OrderLines` | `OrderLineID` | `OrderLineID` | Direct | Unique order-line identifier |
| `Sales.OrderLines` | `OrderID` | `OrderID` | Direct | Order associated with the line |
| `Sales.OrderLines` | `StockItemID` | `StockItemID` | Direct | Product associated with the line |
| `Sales.OrderLines` | `Quantity` | `Quantity` | Direct | Quantity ordered |
| `Sales.OrderLines` | `UnitPrice` | `UnitPrice` | Direct | Unit selling price |
| `Sales.OrderLines` | `TaxRate` | `TaxRate` | Direct | Tax rate applied to the line |
| `Sales.OrderLines` | `PickedQuantity` | `PickedQuantity` | Direct | Quantity picked |
| `Sales.OrderLines` | `LastEditedWhen` | `SourceLastEditedWhen` | Direct | Source modification timestamp |

### Source Fields Intentionally Not Included Initially

The following fields are excluded from the initial analytical fact:

- `Description`
- `PackageTypeID`
- `PickingCompletedWhen`
- `LastEditedBy`

These can be added later if required by a business use case.

### Important Design Decision

`FactOrderLine` must not be treated as the same financial fact as
`FactInvoiceLine`.

An order line represents ordered activity.

An invoice line represents invoiced activity.

An order may exist without an invoice, as demonstrated by the
validated source relationship.

### Analytical Use

`FactOrderLine` will support:

- Units ordered
- Product demand analysis
- Order-line volume
- Ordered versus invoiced analysis
- Operational order analysis

Financial sales metrics will primarily use `FactInvoiceLine` rather
than assuming that every order represents completed invoiced sales.

## Warehouse.StockItems → DimProduct

### Target Grain

One row per stock item/product.

### Business Purpose

Provides product attributes used to analyze sales, profitability,
inventory, and product performance.

### Column Mapping

| Source Table | Source Column | Target Column | Transformation | Business Meaning |
|---|---|---|---|---|
| `Warehouse.StockItems` | `StockItemID` | `ProductKey` | Surrogate key generated in Gold; source ID retained as `SourceStockItemID` | Product identifier |
| `Warehouse.StockItems` | `StockItemName` | `ProductName` | Direct | Product name |
| `Warehouse.StockItems` | `SupplierID` | `SupplierID` | Direct | Supplier associated with the product |
| `Warehouse.StockItems` | `ColorID` | `ColorID` | Direct | Product color |
| `Warehouse.StockItems` | `Brand` | `Brand` | Direct | Product brand |
| `Warehouse.StockItems` | `Size` | `Size` | Direct | Product size |
| `Warehouse.StockItems` | `LeadTimeDays` | `LeadTimeDays` | Direct | Supplier/product lead time |
| `Warehouse.StockItems` | `IsChillerStock` | `IsChillerStock` | Direct | Indicates whether the product requires chilled storage |
| `Warehouse.StockItems` | `Barcode` | `Barcode` | Direct | Product barcode |
| `Warehouse.StockItems` | `TaxRate` | `TaxRate` | Direct | Product tax rate |
| `Warehouse.StockItems` | `UnitPrice` | `UnitPrice` | Direct | Product unit price |
| `Warehouse.StockItems` | `RecommendedRetailPrice` | `RecommendedRetailPrice` | Direct | Recommended retail price |
| `Warehouse.StockItems` | `TypicalWeightPerUnit` | `TypicalWeightPerUnit` | Direct | Typical weight per unit |

### Source Fields Intentionally Not Included Initially

The following fields are excluded from the initial analytical
dimension:

- `UnitPackageID`
- `OuterPackageID`
- `QuantityPerOuter`
- `MarketingComments`
- `InternalComments`
- `Photo`
- `CustomFields`
- `Tags`
- `SearchDetails`
- `LastEditedBy`

Package-related fields can be added later if packaging analysis becomes
a requirement.

### System-Versioned Source

`Warehouse.StockItems` is system-versioned and contains `ValidFrom` and
`ValidTo`.

The initial implementation will use the current product records.

A future enhancement may implement a Slowly Changing Dimension Type 2
strategy if historical product attribute changes need to be preserved.

### Analytical Use

`DimProduct` will support:

- Product sales analysis
- Product profitability analysis
- Inventory analysis
- Brand analysis
- Chiller/non-chiller analysis
- Product-level reporting

### Key Design Decision

The source `StockItemID` will be preserved as the business/source key.

A separate surrogate `ProductKey` may be generated in the Gold layer
to support dimensional modeling and future historical versions.

## Sales.Customers → DimCustomer

### Target Grain

One row per current customer.

### Business Purpose

Provides customer attributes used for customer-level sales and
geographic analysis.

### Column Mapping

| Source Table | Source Column | Target Column | Transformation | Business Meaning |
|---|---|---|---|---|
| `Sales.Customers` | `CustomerID` | `CustomerKey` | Surrogate key generated in Gold; source ID retained as `SourceCustomerID` | Customer identifier |
| `Sales.Customers` | `CustomerName` | `CustomerName` | Direct | Customer name |
| `Sales.Customers` | `CustomerCategoryID` | `CustomerCategoryID` | Direct | Customer category |
| `Sales.Customers` | `BuyingGroupID` | `BuyingGroupID` | Direct | Buying group associated with the customer |
| `Sales.Customers` | `DeliveryCityID` | `DeliveryCityID` | Direct | Customer delivery city |
| `Sales.Customers` | `AccountOpenedDate` | `AccountOpenedDate` | Direct | Date the customer account was opened |
| `Sales.Customers` | `StandardDiscountPercentage` | `StandardDiscountPercentage` | Direct | Standard discount percentage |

### Source Fields Intentionally Not Included Initially

The following fields are excluded from the initial customer dimension:

- `BillToCustomerID`
- `PrimaryContactPersonID`
- `AlternateContactPersonID`
- `DeliveryMethodID`
- `PostalCityID`
- `CreditLimit`
- `IsStatementSent`
- `IsOnCreditHold`
- `PaymentDays`
- Phone and fax fields
- Address fields
- `DeliveryLocation`
- `LastEditedBy`

These fields can be added later when a specific analytical requirement
justifies them.

### System-Versioned Source

`Sales.Customers` is system-versioned and contains `ValidFrom` and
`ValidTo`.

The initial implementation will use the current customer records.

A future enhancement may implement Slowly Changing Dimension Type 2 if
historical customer attribute changes need to be preserved.

### Analytical Use

`DimCustomer` will support:

- Customer sales analysis
- Customer profitability analysis
- Customer segmentation
- Buying-group analysis
- Geographic customer analysis

### Key Design Decision

The source `CustomerID` will be preserved as the business/source key.

A separate surrogate `CustomerKey` may be generated in the Gold layer
to support dimensional modeling and future historical versions.

## Warehouse.StockItemHoldings → FactInventorySnapshot

### Target Grain

One row per stock item representing the current inventory position.

### Business Purpose

Represents the latest available inventory position for each product.

This fact is a **current-state snapshot**, not a historical inventory
movement fact.

### Column Mapping

| Source Table | Source Column | Target Column | Transformation | Business Meaning |
|---|---|---|---|---|
| `Warehouse.StockItemHoldings` | `StockItemID` | `ProductKey` | Resolve to `DimProduct` | Product associated with inventory |
| `Warehouse.StockItemHoldings` | `QuantityOnHand` | `QuantityOnHand` | Direct | Current quantity on hand |
| `Warehouse.StockItemHoldings` | `BinLocation` | `BinLocation` | Direct | Warehouse bin/location |
| `Warehouse.StockItemHoldings` | `LastStocktakeQuantity` | `LastStocktakeQuantity` | Direct | Quantity recorded during the last stocktake |
| `Warehouse.StockItemHoldings` | `LastCostPrice` | `LastCostPrice` | Direct | Latest available inventory cost |
| `Warehouse.StockItemHoldings` | `ReorderLevel` | `ReorderLevel` | Direct | Inventory level requiring replenishment |
| `Warehouse.StockItemHoldings` | `TargetStockLevel` | `TargetStockLevel` | Direct | Target inventory level |
| `Warehouse.StockItemHoldings` | `LastEditedWhen` | `SourceLastEditedWhen` | Direct | Source modification timestamp |

### Derived Measure

```text
InventoryValue =
    QuantityOnHand × LastCostPrice

    ## Warehouse.StockItemTransactions → FactInventoryTransaction

### Target Grain

One row per stock-item inventory transaction.

### Business Purpose

Represents historical inventory movements associated with products.

This fact provides the historical movement view that complements the
current-state inventory snapshot in `FactInventorySnapshot`.

### Column Mapping

| Source Table | Source Column | Target Column | Transformation | Business Meaning |
|---|---|---|---|---|
| `Warehouse.StockItemTransactions` | `StockItemTransactionID` | `StockItemTransactionID` | Direct | Unique inventory transaction identifier |
| `Warehouse.StockItemTransactions` | `StockItemID` | `ProductKey` | Resolve to `DimProduct` | Product associated with the transaction |
| `Warehouse.StockItemTransactions` | `TransactionTypeID` | `TransactionTypeKey` | Resolve to `DimTransactionType` | Type of inventory transaction |
| `Warehouse.StockItemTransactions` | `CustomerID` | `CustomerKey` | Resolve to `DimCustomer` when applicable | Customer associated with the transaction |
| `Warehouse.StockItemTransactions` | `InvoiceID` | `InvoiceID` | Direct | Invoice associated with the transaction when applicable |
| `Warehouse.StockItemTransactions` | `SupplierID` | `SupplierID` | Direct | Supplier associated with the transaction when applicable |
| `Warehouse.StockItemTransactions` | `PurchaseOrderID` | `PurchaseOrderID` | Direct | Purchase order associated with the transaction when applicable |
| `Warehouse.StockItemTransactions` | `TransactionOccurredWhen` | `TransactionOccurredWhen` | Direct | Date and time the transaction occurred |
| `Warehouse.StockItemTransactions` | `Quantity` | `Quantity` | Direct | Quantity associated with the inventory movement |
| `Warehouse.StockItemTransactions` | `LastEditedWhen` | `SourceLastEditedWhen` | Direct | Source modification timestamp |

### Analytical Use

`FactInventoryTransaction` will support:

- Inventory movement analysis
- Stock receipt analysis
- Stock issue analysis
- Stock transfer analysis
- Stock adjustment analysis
- Customer-related inventory activity
- Supplier-related inventory activity

### Transaction Type Interpretation

The transaction type will be resolved through
`DimTransactionType`.

The source transaction type should not be hard-coded in downstream
transformations.

Primary inventory-related categories include:

- Stock Transfer
- Stock Issue
- Stock Receipt
- Stock Adjustment at Stocktake

Customer and supplier invoice/credit-note transactions may also be
used when analyzing inventory-related business activity.

### Optional Relationships

The following source relationships are optional because their values
depend on the transaction type:

- `CustomerID`
- `InvoiceID`
- `SupplierID`
- `PurchaseOrderID`

These fields must not be treated as universally populated.

### Important Design Decision

`FactInventoryTransaction` is a **movement fact**.

It must remain separate from:

```text
FactInventorySnapshot
    → current inventory position

    ## Application.TransactionTypes → DimTransactionType

### Target Grain

One row per transaction type.

### Business Purpose

Provides the business meaning and analytical classification of
inventory and related transactions.

### Column Mapping

| Source Table | Source Column | Target Column | Transformation | Business Meaning |
|---|---|---|---|---|
| `Application.TransactionTypes` | `TransactionTypeID` | `TransactionTypeKey` | Surrogate key generated in Gold; source ID retained as `SourceTransactionTypeID` | Transaction type identifier |
| `Application.TransactionTypes` | `TransactionTypeName` | `TransactionTypeName` | Direct | Business name of the transaction type |

### Analytical Classification

The analytical category will be derived from the source transaction
type:

| Transaction Type | Analytical Category |
|---|---|
| Customer Invoice | Sales |
| Customer Credit Note | Sales Adjustment |
| Customer Payment Received | Financial |
| Customer Refund | Financial |
| Supplier Invoice | Purchasing |
| Supplier Credit Note | Purchasing Adjustment |
| Supplier Payment Issued | Financial |
| Supplier Refund | Financial |
| Stock Transfer | Inventory |
| Stock Issue | Inventory |
| Stock Receipt | Inventory |
| Stock Adjustment at Stocktake | Inventory |
| Customer Contra | Financial / Other |

### Important Design Decision

The source transaction type name remains the authoritative business
definition.

The analytical category is a downstream classification used to make
reporting and filtering easier.

Transaction type names should not be hard-coded into the
`FactInventoryTransaction` transformation.

### Analytical Use

`DimTransactionType` will support:

- Inventory movement analysis
- Transaction-category analysis
- Stock receipt analysis
- Stock issue analysis
- Stock transfer analysis
- Stock adjustment analysis
- Sales-related inventory activity analysis

### Load Strategy

This is a small reference dataset.

The initial implementation will use a full load rather than an
incremental extraction strategy.

### System-Versioned Source

`Application.TransactionTypes` is system-versioned and contains
`ValidFrom` and `ValidTo`.

The initial implementation will use the current transaction-type
records.

Historical versions may be incorporated later if the business requires
historical tracking of transaction-type definitions.

## Application.Cities + Application.StateProvinces + Application.Countries → DimGeography

### Target Grain

One row per city.

### Business Purpose

Provides the geographic hierarchy required to analyze retail activity
from customer delivery location through city, state/province, sales
territory, and country.

### Source Relationships

```text
Sales.Customers.DeliveryCityID
        ↓
Application.Cities.CityID
        ↓
Application.StateProvinces.StateProvinceID
        ↓
Application.Countries.CountryID

## Open-Meteo → DimWeather

### Target Grain

One row per weather location per calendar date.

### Business Purpose

Provides daily weather context that can be joined with retail sales
activity by calendar date.

The initial implementation treats Dallas-area weather as contextual
environmental data. It does not imply that every customer or transaction
was directly affected by the weather conditions.

### Column Mapping

| Source Field | Target Column | Transformation | Business Meaning |
|---|---|---|---|
| `daily.time` | `WeatherDate` | Normalize daily array into rows | Calendar date of the weather observation |
| `daily.temperature_2m_max` | `TemperatureMax` | Direct | Maximum daily temperature |
| `daily.temperature_2m_min` | `TemperatureMin` | Direct | Minimum daily temperature |
| `daily.precipitation_sum` | `Precipitation` | Direct | Total daily precipitation |
| `daily.rain_sum` | `Rainfall` | Direct | Total daily rainfall |
| `daily.wind_speed_10m_max` | `WindSpeedMax` | Direct | Maximum daily wind speed |
| API `latitude` | `Latitude` | Direct | Weather observation latitude |
| API `longitude` | `Longitude` | Direct | Weather observation longitude |
| API `timezone` | `Timezone` | Direct | Source timezone |

### Source Units

The initial API request uses:

- Temperature: °F
- Precipitation: mm
- Rainfall: mm
- Wind speed: mph

These units should be preserved consistently in the analytical layer.

### JSON Normalization

The Open-Meteo response contains parallel arrays under `daily`.

Example:

```text
daily.time
daily.temperature_2m_max
daily.temperature_2m_min
daily.precipitation_sum
daily.rain_sum
daily.wind_speed_10m_max

## Nager.Date → DimHoliday

### Target Grain

One row per holiday occurrence and applicable
jurisdiction/type combination.

### Business Purpose

Provides holiday and calendar context for analyzing potential changes
in retail activity around public holidays.

### Column Mapping

| Source Field | Target Column | Transformation | Business Meaning |
|---|---|---|---|
| `date` | `HolidayDate` | Direct | Holiday date |
| `localName` | `LocalHolidayName` | Direct | Local holiday name |
| `name` | `HolidayName` | Direct | Standard holiday name |
| `countryCode` | `CountryCode` | Direct | Country code |
| `fixed` | `IsFixed` | Direct | Indicates whether the holiday date is fixed |
| `global` | `IsGlobal` | Direct | Indicates whether the holiday applies globally |
| `counties` | `ApplicableRegions` | Normalize array | Applicable regional/jurisdiction codes |
| `launchYear` | `LaunchYear` | Direct | Year the holiday was introduced, when provided |
| `types` | `HolidayTypes` | Normalize array | Holiday classifications |

### Holiday Applicability

The source may contain multiple records for the same calendar date.

Therefore, the transformation must preserve the source records rather
than assuming one holiday record per date.

Texas applicability will be determined using:

- `IsGlobal`
- `ApplicableRegions`

The transformation must not assume that every US holiday applies to
Texas.

### Analytical Use

`DimHoliday` will support:

- Holiday versus non-holiday sales comparison
- Holiday sales patterns
- Holiday-type analysis
- Texas-specific holiday analysis
- Calendar-based sales analysis

### Historical Coverage

The retail source covers:

```text
2013-01-01 through 2016-05-31

## Generated Calendar → DimDate

### Target Grain

One row per calendar date.

### Business Purpose

Provides a standardized calendar dimension for time-based analysis
across sales, inventory, weather, and holiday data.

### Date Range

The initial dimension will cover:

```text
2013-01-01 through 2016-05-31



