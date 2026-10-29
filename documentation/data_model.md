# Data Model

## Overview

The model follows a **star schema** design with `EcomOrdersTable` as the central fact table.

```
                    ┌─────────────────────┐
                    │   Dim_Date          │
                    └──────────┬──────────┘
                               │
┌──────────────────┐    ┌──────▼──────────────┐    ┌─────────────────────┐
│ EcomCustomerTable│◄───│  EcomOrdersTable    │───►│ EcomProductTable    │
└──────────────────┘    └──────┬──────────────┘    └─────────────────────┘
                               │
                    ┌──────────┼──────────┐
                    │          │          │
          ┌─────────▼──┐  ┌────▼────┐  ┌──▼──────────────────┐
          │EcomRefunds │  │Campaign │  │ WebTrafficTable     │
          │Table       │  │Perf.    │  │                     │
          └────────────┘  └─────────┘  └─────────────────────┘
```

## Tables

| Table                        | Role          | Grain                          | Key Columns |
|------------------------------|---------------|--------------------------------|-------------|
| EcomOrdersTable              | Fact          | One row per Order_ID           | Order_ID, Order_Date, Customer_ID, Product_ID, Quantity, Coupon_Discount_Pct, Shipping_Fee, Order_Status, Channel, Device |
| EcomProductTable             | Dimension     | One row per Product_ID         | Product_ID, Product_Name, Category, Brand, List_Price, Unit_Cost |
| EcomCustomerTable            | Dimension     | One row per Customer_ID        | Customer_ID, Customer_Name, Region, Customer_Tier, Signup_Date |
| EcomRefundsTable             | Fact / Bridge | One row per Refund_ID          | Refund_ID, Order_ID, Refund_Date, Refund_Qty, Refund_Status |
| CampaignPerformanceTable     | Fact          | Month + Channel                | Month, Marketing_Channel, Marketing_Spend, Impressions, Clicks |
| WebTrafficTable              | Fact          | Month + Channel                | Month, Channel, Sessions, Add_to_Cart |
| Dim_Date                     | Date Dimension| One row per day                | Date, Year, Month, Month Name, Quarter |
| dim_Marketing_Channel        | Dimension     | One row per Channel            | Marketing_Channel |
| _Measures                    | Measure table | —                              | All DAX measures live here |

## Relationships

| From                    | To                       | Type     | Notes |
|-------------------------|--------------------------|----------|-------|
| EcomOrdersTable[Order_Date] | Dim_Date[Date]        | Many-to-One | Active, for time intelligence |
| EcomOrdersTable[Customer_ID] | EcomCustomerTable[Customer_ID] | Many-to-One | |
| EcomOrdersTable[Product_ID] | EcomProductTable[Product_ID] | Many-to-One | |
| EcomRefundsTable[Order_ID] | EcomOrdersTable[Order_ID] | Many-to-One | Used for refund calculations |
| CampaignPerformanceTable[Month] | Dim_Date[Date]     | Many-to-One | Month-level |
| WebTrafficTable[Month]  | Dim_Date[Date]           | Many-to-One | Month-level |
| Channel fields          | dim_Marketing_Channel    | Many-to-One | For consistent channel filtering |

## Important Modelling Rules

1. **Calendar table** (`Dim_Date`) is marked as a Date table.
2. Revenue metrics only consider **Valid Orders** (Order_Status ≠ "Cancelled" **and** Quantity > 0).
3. Campaign and Web Traffic are at **Month + Channel** grain — relationships must respect this grain to avoid double-counting.
4. Customer Region is taken from `EcomCustomerTable` (canonical source).
5. Exact duplicate Order_ID and Refund_ID rows were removed in Power Query.

## Data Cleaning Applied (Power Query)

- Removed exact duplicate Order_ID and Refund_ID rows.
- Standardised Marketing Channel, Device, and Order Status (case, aliases, trailing spaces).
- Handled blank Channel / Device / Customer Tier with documented rules (usually mapped to “Unknown” or left as blank with a note).
- Corrected data types (dates, numbers, percentages).
