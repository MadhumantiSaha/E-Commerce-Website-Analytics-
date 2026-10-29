# KPI Definitions & Business Logic

This document provides standardized business definitions, calculation logic, and operational rules for every Key Performance Indicator (KPI) used across the Ecommerce Power BI dashboard.

---

## Table of Contents
- [Global Business Rules](#global-business-rules)
- [Metric Relationship Hierarchy](#metric-relationship-hierarchy)
- [1. Sales & Revenue KPIs](#1-sales--revenue-kpis)
- [2. Cost & Profitability KPIs](#2-cost--profitability-kpis)
- [3. Returns & Refund KPIs](#3-returns--refund-kpis)
- [4. Customer & Retention KPIs](#4-customer--retention-kpis)
- [5. Marketing & Acquisition KPIs](#5-marketing--acquisition-kpis)

---

## Global Business Rules

All measures and visual reports must strictly adhere to the following core business logic:

> [!IMPORTANT]
> 1. **Order Validation:** Cancelled orders (`Order_Status = "Cancelled"`) and invalid transaction rows (`Quantity <= 0`) are excluded from all revenue, profit, and order-count metrics.
> 2. **Canonical Customer Attributes:** Customer attributes (such as `Region` and `Customer_Tier`) must always be derived from the Customer Master (`EcomCustomerTable`).
> 3. **Attribute Standardization:** Dimensions including Marketing `Channel`, `Device`, and `Order_Status` were cleaned and standardized in Power Query prior to model ingestion.
> 4. **Refund Status Scope:** Refund calculations exclusively evaluate refund records with status **"Refunded"** or **"Approved"** (pending or rejected requests are excluded).

---

## Metric Relationship Hierarchy

```
[Qty] × [List_Price] × (1 − [Coupon_Discount_Pct])
                     │
                     ▼
             Product Revenue  +  Shipping Revenue
                     │
                     ▼
     ┌────────── Net Revenue ──────────┐
     │                                 │
  (-) COGS                     (-) Refund Value
     │                                 │
     ▼                                 ▼
Gross Profit                   Adjusted Revenue
     │                                 │
(÷) Net Revenue               (÷) Marketing Spend
     │                                 │
     ▼                                 ▼
Gross Margin %                       ROAS
```

---

## 1. Sales & Revenue KPIs

Core top-line volume and value measures derived from `EcomOrdersTable`.

| KPI | Business Definition | Formula / DAX Logic | Scope & Notes |
| :--- | :--- | :--- | :--- |
| **Valid Orders** | Total distinct orders successfully placed (excluding cancellations and non-positive quantities). | `DISTINCTCOUNT(Order_ID)`<br>_where `Order_Status ≠ "Cancelled"` & `Quantity > 0`_ | Foundation for order-level ratios (AOV, Conversion Rate). |
| **Product Revenue** | Total revenue generated from product sales after applied discounts, before shipping charges. | `SUMX(Orders, Quantity × List_Price × (1 − Coupon_Discount_Pct))` | Valid orders only. |
| **Shipping Revenue** | Total delivery fees collected from customers. | `SUM(Shipping_Fee)` | Valid orders only. |
| **Net Revenue** | Primary top-line revenue representing combined product and shipping revenue. | `[Product Revenue] + [Shipping Revenue]` | Primary top-line benchmark. |
| **Average Order Value (AOV)** | Mean Net Revenue generated per valid customer order. | `[Net Revenue] ÷ [Valid Orders]` | Evaluates basket size and transaction quality. |
| **Revenue YTD** | Year-to-date cumulative Net Revenue for the current reporting year. | `TOTALYTD([Net Revenue], Dim_Date[Date])` | Requires active relationship with marked `Dim_Date` table. |
| **Revenue YoY %** | Percentage growth in Net Revenue compared against the equivalent prior-year period. | `DIVIDE([Net Revenue] − [Prior Year Net Revenue], [Prior Year Net Revenue])` | Tracks year-over-year expansion pace. |

---

## 2. Cost & Profitability KPIs

Margin and unit-cost metrics tracking retail operating efficiency.

| KPI | Business Definition | Formula / DAX Logic | Scope & Notes |
| :--- | :--- | :--- | :--- |
| **COGS** (Cost of Goods Sold) | Direct wholesale cost of products sold to customers. | `SUMX(Orders, Quantity × RELATED(Product[Unit_Cost]))` | Valid orders only. |
| **Gross Profit** | Retained gross earnings after accounting for direct inventory costs. | `[Net Revenue] − [COGS]` | Evaluates unit-level profitability. |
| **Gross Margin %** | Proportion of Net Revenue retained as gross profit. | `DIVIDE([Gross Profit], [Net Revenue])` | Displayed as percentage (%). |

---

## 3. Returns & Refund KPIs

Post-purchase return volumes and financial adjustments from `EcomRefundsTable`.

| KPI | Business Definition | Formula / DAX Logic | Scope & Notes |
| :--- | :--- | :--- | :--- |
| **Refunded Orders** | Count of unique orders that had an approved or completed refund. | `DISTINCTCOUNT(Refund[Order_ID])`<br>_where `Refund_Status ∈ {"Refunded", "Approved"}`_ | Evaluates order defect frequency. |
| **Refund Rate %** | Proportion of valid orders that resulted in an approved refund. | `DIVIDE([Refunded Orders], [Valid Orders])` | Key product quality and satisfaction indicator. |
| **Refund Value** | Total financial value reimbursed to customers for returned units. | `SUMX(Refunds, Refund_Qty × Net_Unit_Price)` | Evaluated using net purchase unit price. |
| **Adjusted Revenue** | Net revenue realized after deducting the value of approved refunds. | `[Net Revenue] − [Refund Value]` | Realistic revenue realization figure. |

---

## 4. Customer & Retention KPIs

Customer acquisition, repeat purchasing, and loyalty metrics from `EcomCustomerTable`.

| KPI | Business Definition | Formula / DAX Logic | Scope & Notes |
| :--- | :--- | :--- | :--- |
| **New Customers** | Customers who have placed exactly one valid lifetime order. | `COUNTROWS(FILTER(Customers, [Customer Order Count] = 1))` | Indicator of new customer acquisition. |
| **Repeat Customers** | Customers who have placed two or more valid lifetime orders. | `COUNTROWS(FILTER(Customers, [Customer Order Count] ≥ 2))` | Core driver of customer lifetime value (LTV). |
| **Repeat Customer Rate %** | Share of customer base with repeat purchase history. | `DIVIDE([Repeat Customers], [Total Transacting Customers])` | Strong indicator of loyalty and retention. |

---

## 5. Marketing & Acquisition KPIs

Efficiency, spend, and conversion metrics across acquisition channels (`CampaignPerformanceTable` & `WebTrafficTable`).

| KPI | Business Definition | Formula / DAX Logic | Scope & Notes |
| :--- | :--- | :--- | :--- |
| **Marketing Spend** | Total advertising and campaign expenditures across channels and periods. | `SUM(CampaignPerformance[Marketing_Spend])` | Aggregated at Month + Channel grain. |
| **ROAS** (Return on Ad Spend) | Revenue generated per dollar of marketing capital deployed. | `DIVIDE([Adjusted Revenue], [Marketing Spend])` | Higher indicates greater advertising efficiency. |
| **Order Conversion Rate %** | Share of total website browsing sessions that successfully converted into a valid order. | `DIVIDE([Valid Orders], SUM(WebTraffic[Sessions]))` | Evaluates site checkout efficiency and traffic quality. |
