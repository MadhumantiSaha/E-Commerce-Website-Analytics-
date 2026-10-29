# Dashboard Guide

How to navigate and interpret the **Ecommerce Data Analysis Dashboard**.

## Pages Overview

### 1. Executive Commerce

**Purpose**: One-page executive view of commercial performance.

**Key components**:

| Visual                        | What it shows                              | How to use it |
|-------------------------------|--------------------------------------------|---------------|
| KPI Cards                     | Net Revenue, Gross Profit, Gross Margin %, AOV, Refund Rate %, Valid Orders | Quick health check |
| Monthly Revenue & Profit Trend| Net Revenue (columns) + Gross Profit (line) over time | Spot seasonality and growth/decline |
| Revenue by Region             | Contribution of East / West / North / South | Identify strongest and weakest regions |
| Revenue by Category           | Contribution of each product category      | Understand category mix |
| Refund Rate by Category       | Refund pressure by category                | Flag high-risk categories |
| Slicers                       | Date, Region, Channel, Category, Device    | Filter the entire page |

**Typical questions answered**:
- How is overall revenue and profit trending?
- Which region or category is driving results?
- Where is refund risk concentrated?

---

### 2. Marketing & Customers

**Purpose**: Deep dive into marketing efficiency and customer behaviour.

**Key components**:

| Visual                          | What it shows                                      | How to use it |
|---------------------------------|----------------------------------------------------|---------------|
| Channel Revenue vs Spend        | Net Revenue and Marketing Spend side-by-side       | See efficiency at a glance |
| ROAS by Channel                 | Return on Ad Spend per channel                     | Rank channels by efficiency |
| Order Conversion Rate %         | Valid Orders ÷ Sessions by channel                 | Measure traffic quality |
| New vs Repeat Customers         | Split of customer base                             | Understand loyalty |
| Top 10 Customers                | Highest Adjusted Revenue customers + key metrics   | Identify VIP customers |
| Tooltip / Drill-through         | Full order history of a selected customer          | Investigate individual behaviour |
| Slicers                         | Same as Executive page (synced)                    | Keep context consistent |

**Typical questions answered**:
- Which channels give the best return on marketing spend?
- How well does traffic convert into orders?
- Who are our most valuable customers and what do they buy?

---

### 3. Tooltip – Customer Orders

This is a **report-page tooltip** (or drill-through page).  
It appears when you hover (or right-click → Drill through) on a customer in the Top Customers visual.

It shows the detailed order history for that customer (Order ID, Date, Product, Revenue, Status, etc.).

---

## Recommended Analysis Flow

1. Start on **Executive Commerce** → understand overall performance and trends.
2. Use slicers to focus on a specific period, region, or channel.
3. Move to **Marketing & Customers** → evaluate ROAS and conversion.
4. Drill into Top Customers and use the tooltip for order-level detail.
5. Return to Executive page to communicate findings to leadership.

## Design Notes

- Consistent colour theme and fonts across both pages.
- Slicers are synchronised so filters carry over.
- All revenue and profit measures exclude Cancelled orders and Quantity ≤ 0.
- Adjusted Revenue = Net Revenue − Refund Value.
