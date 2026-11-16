# Wide World Importers Data Dictionary

## Source Database

**Database:** WideWorldImporters

**Source Type:** SQL Server OLTP

**Purpose:** Core retail transactional, customer, product, inventory,
and geographic data.

## Sales Domain

The initial sales model uses the following tables:

- `Sales.Orders`
- `Sales.OrderLines`
- `Sales.Invoices`
- `Sales.InvoiceLines`

## Customer Domain

- `Sales.Customers`

## Product Domain

- `Warehouse.StockItems`

## Inventory Domain

- `Warehouse.StockItemHoldings`
- `Warehouse.StockItemTransactions`

## Reference and Geography Domains

- `Application.TransactionTypes`
- `Application.Cities`
- `Application.StateProvinces`
- `Application.Countries`

## Data Coverage

Primary retail transaction data currently validated:

- Start date: `2013-01-01`
- End date: `2016-05-31`


## Sales.Orders

### Purpose

Represents customer orders in the Wide World Importers operational
system.

### Grain

One row per customer order.

### Primary Key

`OrderID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `OrderID` | Primary Key | Unique order identifier |
| `CustomerID` | Foreign Key | Customer associated with the order |
| `SalespersonPersonID` | Foreign Key | Salesperson associated with the order |
| `OrderDate` | Business Date | Date the order was placed |
| `ExpectedDeliveryDate` | Business Date | Expected delivery date |
| `IsUndersupplyBackordered` | Flag | Indicates whether an undersupply was backordered |
| `LastEditedWhen` | Audit/Incremental Candidate | Last modification timestamp |

### Validated Data Coverage

- Earliest `OrderDate`: `2013-01-01`
- Latest `OrderDate`: `2016-05-31`
- Row count: `73,595`

### Incremental Load Candidate

`LastEditedWhen` is the initial candidate for incremental extraction
because it records when the source row was last edited.

The incremental strategy will be finalized during ingestion design.

## Sales.OrderLines

### Purpose

Represents the individual product lines associated with customer orders.

### Grain

One row per order line.

### Primary Key

`OrderLineID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `OrderLineID` | Primary Key | Unique order-line identifier |
| `OrderID` | Foreign Key | Associated customer order |
| `StockItemID` | Foreign Key | Product associated with the order line |
| `Quantity` | Measure | Quantity ordered |
| `UnitPrice` | Measure | Unit selling price |
| `TaxRate` | Measure | Applicable tax rate |
| `PickedQuantity` | Measure | Quantity picked |
| `LastEditedWhen` | Audit/Incremental Candidate | Last modification timestamp |

### Validated Data Coverage

- Earliest `LastEditedWhen`: `2013-01-01 11:00:00`
- Latest `LastEditedWhen`: `2016-05-31 12:00:00`
- Row count: `231,412`
- Distinct orders represented: `73,595`

### Relationship

```text
Sales.Orders
     |
     | OrderID
     v
Sales.OrderLines

## Sales.Invoices

### Purpose

Represents invoices generated for customer transactions.

### Grain

One row per invoice.

### Primary Key

`InvoiceID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `InvoiceID` | Primary Key | Unique invoice identifier |
| `CustomerID` | Foreign Key | Customer associated with the invoice |
| `BillToCustomerID` | Foreign Key | Customer responsible for billing |
| `OrderID` | Foreign Key | Associated order when applicable |
| `InvoiceDate` | Business Date | Date of the invoice |
| `IsCreditNote` | Flag | Indicates whether the invoice is a credit note |
| `CreditNoteReason` | Attribute | Reason associated with a credit note |
| `LastEditedWhen` | Audit/Incremental Candidate | Last modification timestamp |

### Validated Data Coverage

- Earliest `InvoiceDate`: `2013-01-01`
- Latest `InvoiceDate`: `2016-05-31`
- Row count: `70,510`

### Relationship to Orders

Validated relationship:

- Total orders: `73,595`
- Orders with an invoice: `70,510`
- Orders without an invoice: `3,085`

Therefore, an order should **not** automatically be treated as an
invoice.

Approximately 4.19% of orders in the validated dataset do not have an
associated invoice.

### Credit Note Observation

The validated dataset currently contains:

- `IsCreditNote = 0`: 70,510
- `IsCreditNote = 1`: 0

The `IsCreditNote` field will be retained in the source model even
though no credit notes are currently present in the dataset.

### Incremental Load Candidate

`LastEditedWhen` is the initial candidate for incremental extraction.
The final incremental strategy will be defined during ingestion design.


## Sales.InvoiceLines

### Purpose

Represents the individual product lines included on customer invoices.

### Grain

One row per invoice line.

### Primary Key

`InvoiceLineID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `InvoiceLineID` | Primary Key | Unique invoice-line identifier |
| `InvoiceID` | Foreign Key | Associated invoice |
| `StockItemID` | Foreign Key | Product associated with the invoice line |
| `Quantity` | Measure | Quantity invoiced |
| `UnitPrice` | Measure | Unit selling price |
| `TaxRate` | Measure | Applicable tax rate |
| `TaxAmount` | Measure | Tax amount for the invoice line |
| `LineProfit` | Measure | Profit associated with the invoice line |
| `ExtendedPrice` | Measure | Extended line amount including tax |
| `LastEditedWhen` | Audit/Incremental Candidate | Last modification timestamp |

### Validated Data Coverage

- Earliest `LastEditedWhen`: `2013-01-01 12:00:00`
- Latest `LastEditedWhen`: `2016-05-31 12:00:00`
- Row count: `228,265`
- Distinct invoices represented: `70,510`

### Relationship

```text
Sales.Invoices
      |
      | InvoiceID
      v
Sales.InvoiceLines

## Sales.Customers

### Purpose

Represents customers associated with retail orders and invoices.

### Grain

One row per customer in the current customer dataset.

### Primary Key

`CustomerID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `CustomerID` | Primary Key | Unique customer identifier |
| `CustomerName` | Attribute | Customer name |
| `CustomerCategoryID` | Foreign Key | Customer category |
| `BuyingGroupID` | Foreign Key | Buying group associated with the customer |
| `DeliveryCityID` | Foreign Key | Customer delivery city |
| `AccountOpenedDate` | Business Date | Date the customer account was opened |
| `StandardDiscountPercentage` | Measure/Attribute | Standard customer discount percentage |

### Important Characteristics

`Sales.Customers` is a system-versioned temporal table.

The source contains:

- `ValidFrom`
- `ValidTo`

These fields can support historical customer-state analysis in a
future enhancement.

For the initial version, the platform will focus on the current
customer records rather than implementing full historical
Slowly Changing Dimension behavior.

### Geography Relationship

```text
Sales.Customers
      |
      | DeliveryCityID
      v
Application.Cities
      |
      | StateProvinceID
      v
Application.StateProvinces
      |
      | CountryID
      v
Application.Countries

## Warehouse.StockItems

### Purpose

Represents products/items available for sale in the Wide World
Importers system.

### Grain

One row per stock item.

### Primary Key

`StockItemID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `StockItemID` | Primary Key | Unique product identifier |
| `StockItemName` | Attribute | Product name |
| `SupplierID` | Foreign Key | Supplier associated with the product |
| `ColorID` | Foreign Key | Product color |
| `Brand` | Attribute | Product brand |
| `Size` | Attribute | Product size |
| `LeadTimeDays` | Measure | Supplier lead time in days |
| `IsChillerStock` | Flag | Indicates whether the item requires chilled storage |
| `Barcode` | Attribute | Product barcode |
| `TaxRate` | Measure | Applicable product tax rate |
| `UnitPrice` | Measure | Product unit price |
| `RecommendedRetailPrice` | Measure | Recommended retail price |
| `TypicalWeightPerUnit` | Measure | Typical weight per unit |

### Important Characteristics

`Warehouse.StockItems` is a system-versioned temporal table and contains:

- `ValidFrom`
- `ValidTo`

The initial version will use the current product records.

Historical product changes may be incorporated later as a Slowly
Changing Dimension implementation.

### Analytical Use

Product data will support:

- Sales by product
- Sales by brand
- Sales by size
- Sales by product category where available
- Inventory analysis
- Chiller/non-chiller analysis
- Product profitability analysis


## Warehouse.StockItemHoldings

### Purpose

Represents the current inventory position for each stock item.

### Grain

One row per stock item.

### Primary Key

`StockItemID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `StockItemID` | Primary Key / Foreign Key | Product associated with the inventory record |
| `QuantityOnHand` | Measure | Current quantity available on hand |
| `BinLocation` | Attribute | Warehouse bin/location |
| `LastStocktakeQuantity` | Measure | Quantity recorded during the last stocktake |
| `LastCostPrice` | Measure | Cost price associated with the inventory |
| `ReorderLevel` | Measure | Inventory level at which replenishment is required |
| `TargetStockLevel` | Measure | Target inventory level |
| `LastEditedWhen` | Audit/Incremental Candidate | Last modification timestamp |

### Inventory Interpretation

`QuantityOnHand` represents the current inventory position.

A derived inventory value may be calculated as:

```text
InventoryValue = QuantityOnHand × LastCostPrice

## Warehouse.StockItemTransactions

### Purpose

Represents historical inventory movements for stock items.

### Grain

One row per stock-item inventory transaction.

### Primary Key

`StockItemTransactionID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `StockItemTransactionID` | Primary Key | Unique inventory transaction identifier |
| `StockItemID` | Foreign Key | Stock item associated with the transaction |
| `TransactionTypeID` | Foreign Key | Type of inventory transaction |
| `CustomerID` | Foreign Key | Customer associated with the transaction when applicable |
| `InvoiceID` | Foreign Key | Invoice associated with the transaction when applicable |
| `SupplierID` | Foreign Key | Supplier associated with the transaction when applicable |
| `PurchaseOrderID` | Foreign Key | Purchase order associated with the transaction when applicable |
| `TransactionOccurredWhen` | Business Timestamp | Date and time the transaction occurred |
| `Quantity` | Measure | Quantity associated with the inventory movement |
| `LastEditedWhen` | Audit/Incremental Candidate | Last modification timestamp |

### Relationship

```text
Warehouse.StockItemTransactions
          |
          | StockItemID
          v
Warehouse.StockItems

## Application.TransactionTypes

### Purpose

Provides the business meaning of inventory and financial transaction
types recorded in the Wide World Importers system.

### Grain

One row per transaction type.

### Primary Key

`TransactionTypeID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `TransactionTypeID` | Primary Key | Unique transaction type identifier |
| `TransactionTypeName` | Attribute | Business name of the transaction type |

### Observed Transaction Types

| ID | Transaction Type | Analytical Category |
|---:|---|---|
| 1 | Customer Invoice | Sales |
| 2 | Customer Credit Note | Sales Adjustment |
| 3 | Customer Payment Received | Financial |
| 4 | Customer Refund | Financial |
| 5 | Supplier Invoice | Purchasing |
| 6 | Supplier Credit Note | Purchasing Adjustment |
| 7 | Supplier Payment Issued | Financial |
| 8 | Supplier Refund | Financial |
| 9 | Stock Transfer | Inventory |
| 10 | Stock Issue | Inventory |
| 11 | Stock Receipt | Inventory |
| 12 | Stock Adjustment at Stocktake | Inventory |
| 13 | Customer Contra | Financial / Other |

### Analytical Use

This reference table will be used to interpret
`Warehouse.StockItemTransactions`.

For inventory movement analysis, the primary transaction categories
are:

- Stock Transfer
- Stock Issue
- Stock Receipt
- Stock Adjustment at Stocktake

Customer and supplier invoice/credit-note transactions may also be
used when analyzing inventory-related business activity.

### Important Design Decision

Transaction type descriptions will be sourced from
`Application.TransactionTypes` rather than hard-coded in downstream
Databricks transformations.

This keeps the analytical model aligned with the source system's
business definitions.

### Incremental Load

This is a small reference table and will initially be treated as a
full-load reference dataset.

## Application.Cities

### Purpose

Provides city-level geographic reference data used to associate
customers with their delivery locations.

### Grain

One row per city.

### Primary Key

`CityID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `CityID` | Primary Key | Unique city identifier |
| `CityName` | Attribute | City name |
| `StateProvinceID` | Foreign Key | State or province associated with the city |
| `LatestRecordedPopulation` | Attribute | Latest population value recorded by the source |

### Relationship

```text
Sales.Customers
      |
      | DeliveryCityID
      v
Application.Cities
      |
      | StateProvinceID
      v
Application.StateProvinces

## Application.StateProvinces

### Purpose

Provides state and province reference data used to build the geographic
hierarchy for customers and sales analysis.

### Grain

One row per state or province.

### Primary Key

`StateProvinceID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `StateProvinceID` | Primary Key | Unique state or province identifier |
| `StateProvinceCode` | Attribute | State or province code |
| `StateProvinceName` | Attribute | State or province name |
| `CountryID` | Foreign Key | Country associated with the state or province |
| `SalesTerritory` | Attribute | Sales territory assigned by the source system |

### Relationship

```text
Sales.Customers
      |
      | DeliveryCityID
      v
Application.Cities
      |
      | StateProvinceID
      v
Application.StateProvinces
      |
      | CountryID
      v
Application.Countries

## Application.Countries

### Purpose

Provides country-level geographic reference data used to complete the
geographic hierarchy for customer and sales analysis.

### Grain

One row per country.

### Primary Key

`CountryID`

### Key Columns

| Column | Role | Description |
|---|---|---|
| `CountryID` | Primary Key | Unique country identifier |
| `CountryName` | Attribute | Country name |
| `IsoAlpha3Code` | Attribute | ISO alpha-3 country code |
| `IsoNumericCode` | Attribute | ISO numeric country code |
| `Continent` | Attribute | Continent associated with the country |
| `Region` | Attribute | Geographic region |
| `Subregion` | Attribute | Geographic subregion |

### Relationship

```text
Sales.Customers
      |
      | DeliveryCityID
      v
Application.Cities
      |
      | StateProvinceID
      v
Application.StateProvinces
      |
      | CountryID
      v
Application.Countries