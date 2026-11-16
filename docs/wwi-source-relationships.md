# Wide World Importers Source Relationships

## Purpose

This document records the validated relationships between the Wide World
Importers source tables used by the Retail Business Intelligence Data
Platform.

The relationships were validated directly against the restored
`WideWorldImporters` SQL Server database.

---

## Sales Relationships

### Customers → Orders

```text
Sales.Customers.CustomerID
            ↓
Sales.Orders.CustomerID

Sales.Customers
      │
      ├──────────────→ Sales.Orders
      │                      │
      │                      └──→ Sales.OrderLines
      │
      └──→ Application.Cities
                   │
                   └──→ Application.StateProvinces
                                │
                                └──→ Application.Countries


Sales.Orders
      │
      └──→ Sales.Invoices
                   │
                   └──→ Sales.InvoiceLines
                                │
                                └──→ Warehouse.StockItems
                                           │
                                           ├──→ StockItemHoldings
                                           │
                                           └──→ StockItemTransactions
                                                        │
                                                        └──→ TransactionTypes