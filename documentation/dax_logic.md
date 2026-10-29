# DAX Logic

All measures are stored in the `_Measures` table for cleanliness.

## Core Revenue & Profit Measures

```dax
Product Revenue =
SUMX(
    FILTER(
        EcomOrdersTable,
        EcomOrdersTable[Order_Status] <> "Cancelled"
            && EcomOrdersTable[Quantity] > 0
    ),
    EcomOrdersTable[Quantity]
        * RELATED(EcomProductTable[List_Price])
        * (1 - EcomOrdersTable[Coupon_Discount_Pct])
)
```

```dax
Shipping Revenue =
CALCULATE(
    SUM(EcomOrdersTable[Shipping_Fee]),
    EcomOrdersTable[Order_Status] <> "Cancelled",
    EcomOrdersTable[Quantity] > 0
)
```

```dax
Net Revenue = [Product Revenue] + [Shipping Revenue]
```

```dax
COGS =
SUMX(
    FILTER(
        EcomOrdersTable,
        EcomOrdersTable[Order_Status] <> "Cancelled"
            && EcomOrdersTable[Quantity] > 0
    ),
    EcomOrdersTable[Quantity] * RELATED(EcomProductTable[Unit_Cost])
)
```

```dax
Gross Profit = [Net Revenue] - [COGS]
```

```dax
Gross Margin % = DIVIDE([Gross Profit], [Net Revenue], 0)
```

```dax
Average Order Value = DIVIDE([Net Revenue], [Valid Orders], 0)
```

```dax
Valid Orders =
CALCULATE(
    DISTINCTCOUNT(EcomOrdersTable[Order_ID]),
    EcomOrdersTable[Order_Status] <> "Cancelled",
    EcomOrdersTable[Quantity] > 0
)
```

---

## Refund Measures

```dax
Refunded Orders =
CALCULATE(
    DISTINCTCOUNT(EcomRefundsTable[Order_ID]),
    EcomRefundsTable[Refund_Status] IN { "Refunded", "Approved" }
)
```

```dax
Refund Rate % = DIVIDE([Refunded Orders], [Valid Orders], 0)
```

```dax
Refund Value =
SUMX(
    FILTER(
        EcomRefundsTable,
        EcomRefundsTable[Refund_Status] IN { "Refunded", "Approved" }
    ),
    EcomRefundsTable[Refund_Qty] * RELATED(EcomOrdersTable[Net Unit Price])
)
```

```dax
Adjusted Revenue = [Net Revenue] - [Refund Value]
```

---

## Customer Measures

```dax
Repeat Customer Rate % =
VAR CustomerOrderTable =
    ADDCOLUMNS(
        VALUES(EcomOrdersTable[Customer_ID]),
        "@OrderCount", CALCULATE([Valid Orders])
    )
VAR TotalCustomers =
    COUNTROWS(FILTER(CustomerOrderTable, [@OrderCount] >= 1))
VAR RepeatCustomers =
    COUNTROWS(FILTER(CustomerOrderTable, [@OrderCount] >= 2))
RETURN
    DIVIDE(RepeatCustomers, TotalCustomers, 0)
```

```dax
New Customers =
CALCULATE(
    DISTINCTCOUNT(EcomOrdersTable[Customer_ID]),
    FILTER(
        VALUES(EcomOrdersTable[Customer_ID]),
        CALCULATE(DISTINCTCOUNT(EcomOrdersTable[Order_ID])) = 1
    )
)
```

```dax
Repeat Customers =
CALCULATE(
    DISTINCTCOUNT(EcomOrdersTable[Customer_ID]),
    FILTER(
        VALUES(EcomOrdersTable[Customer_ID]),
        CALCULATE(DISTINCTCOUNT(EcomOrdersTable[Order_ID])) >= 2
    )
)
```

---

## Marketing & Traffic Measures

```dax
Marketing Spend = SUM(CampaignPerformanceTable[Marketing_Spend])
```

```dax
ROAS = DIVIDE([Adjusted Revenue], [Marketing Spend], 0)
```

```dax
Order Conversion Rate % =
DIVIDE([Valid Orders], SUM(WebTrafficTable[Sessions]), 0)
```

---

## Time Intelligence

```dax
Revenue YTD =
TOTALYTD([Net Revenue], Dim_Date[Date])
```

```dax
Revenue LY =
CALCULATE(
    [Net Revenue],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```

```dax
Revenue YoY % =
DIVIDE([Net Revenue] - [Revenue LY], [Revenue LY], 0)
```

---

## Notes

- All revenue / profit measures respect the **Valid Order** rule.
- ROAS uses **Adjusted Revenue** (after refunds) for a more realistic view of return.
- `Net Unit Price` is a calculated column used inside the Refund Value measure.
