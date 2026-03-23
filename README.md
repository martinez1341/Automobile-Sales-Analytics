# Automobile-Sales-Analytics
This project demonstrates a complete business intelligence workflow — from raw data ingestion to an interactive executive dashboard
# 🚗 Automobile Sales Analytics — Power BI Dashboard
---

## 📌 Project Overview

This project demonstrates a complete business intelligence workflow — from raw data ingestion to an interactive executive dashboard — using **Power BI Desktop**. The dataset contains aggregated automobile sales data spanning multiple months, customers, and product SKUs.

The goal was to answer four key business questions:

| # | Business Question |
|---|-------------------|
| 1 | What is our overall revenue performance and average order value? |
| 2 | Which months drive peak and low sales, and what is the seasonal pattern? |
| 3 | Which customers generate the most revenue (top customer concentration)? |
| 4 | Which product SKUs are our highest revenue contributors? |

---

## 📊 Dashboard Preview

| Page | Description |
|------|-------------|
| **Executive Summary** | KPI cards — Total Sales, Orders, Qty, Customers, Avg Order Value |
| **Monthly Trend** | Line / bar chart of monthly sales seasonality |
| **Customer Revenue** | Ranked bar chart of Top 5 customers by revenue |
| **Product Performance** | Ranked bar chart of Top 10 SKUs by revenue |

> 📁 See [`/screenshots`](./screenshots/) for full dashboard images.

---

## 🗂️ Repository Structure

```
automobile-powerbi-project/
│
├── README.md                        ← You are here
├── Automobile_Dataset.xlsx          ← Source data file
│
├── data/
│   └── data_dictionary.md           ← Field definitions & data notes
│
├── dax/
│   ├── measures.dax                 ← All DAX measure definitions
│   └── calculated_columns.dax       ← Calculated column formulas
│
├── docs/
│   └── project_documentation.md     ← Full methodology & design decisions
│
└── screenshots/
    ├── 01_executive_summary.png
    ├── 02_monthly_trend.png
    ├── 03_customer_revenue.png
    └── 04_product_performance.png
```

---

## 📦 Dataset Summary

The source file (`Automobile_Dataset.xlsx`) is a pre-aggregated Excel workbook containing **four pivot summary tables**:

| Table Section | Contents |
|---------------|----------|
| **KPI Summary** | Total Sales ($9.76M), Order Count (2,747), Total Qty (96,428), Unique Customers (89), Avg Order Value ($3,553) |
| **Monthly Sales** | Sum of Sales by calendar month (Jan–Dec) |
| **Top Customers** | Top 5 customers by cumulative sales revenue |
| **Top Products** | Top 10 product SKUs by cumulative sales revenue |

### Key Metrics at a Glance

| Metric | Value |
|--------|-------|
| 💰 Total Sales | **$9,760,221.71** |
| 🧾 Total Orders | **2,747** |
| 📦 Total Qty Ordered | **96,428 units** |
| 👥 Unique Customers | **89** |
| 📈 Avg Order Value | **$3,553.05** |

---

## 🔍 Key Insights

### 1. Strong Q4 Seasonality
October and November dominate sales, contributing **$2.61M (~26.7%)** of annual revenue. This classic automotive retail pattern suggests year-end promotions, fleet procurement cycles, or model-year transitions driving bulk purchasing.

| Month | Sales | % of Total |
|-------|-------|------------|
| November | $1,434,766 | 14.7% |
| October | $1,176,388 | 12.1% |
| January | $936,965 | 9.6% |
| February | $951,180 | 9.7% |
| July | $490,103 | **5.0% ← Lowest** |

### 2. Customer Revenue Concentration Risk
The **top 5 customers** account for a disproportionate share of revenue. Customer **CUS-22** alone contributes **$912,294 (9.3%)**, suggesting the business has a long-tail customer base with a few high-value anchor accounts — a concentration risk to monitor.

| Customer | Revenue | Rank |
|----------|---------|------|
| CUS-22 | $912,294 | 🥇 1st |
| CUS-38 | $654,858 | 🥈 2nd |
| CUS-9  | $200,995 | 🥉 3rd |
| CUS-72 | $197,737 | 4th |
| CUS-13 | $180,125 | 5th |

### 3. Product SKU Leadership
Product **S18-3232** leads at **$284,249** in revenue — more than **2× the lowest-ranked** top-10 SKU, indicating a flagship product line driving outsized returns.

---

## 🛠️ Power BI Build Guide

### Step 1: Load Data

1. Open **Power BI Desktop** → `Get Data` → `Excel Workbook`
2. Select `Automobile_Dataset.xlsx`
3. In the Navigator, check all sheets → `Transform Data`

### Step 2: Power Query Transformations (M Code)

Since the source is a pivoted Excel file, the data must be reshaped:

```m
// Monthly Sales Table — Extract and promote headers
let
    Source = Excel.Workbook(File.Contents("Automobile_Dataset.xlsx"), null, true),
    Sheet1 = Source{[Item="Sheet1",Kind="Sheet"]}[Data],
    
    // Monthly Sales block starts at row 11 (0-indexed)
    MonthlySales = Table.Range(Sheet1, 11, 12),
    Promoted = Table.PromoteHeaders(MonthlySales),
    Renamed = Table.RenameColumns(Promoted, {{"Row Labels", "Month"}, {"Sum of SALES", "Sales"}}),
    TypedTable = Table.TransformColumnTypes(Renamed, {{"Month", type text}, {"Sales", type number}})
in
    TypedTable
```

> See [`/dax/measures.dax`](./dax/measures.dax) for the full set of DAX measures.

### Step 3: Data Model

Create the following tables in the Power BI model:

```
[KPI_Summary]        ← Single-row KPI table
[Monthly_Sales]      ← 12 rows (Jan–Dec)
[Customer_Revenue]   ← Top 5 customers
[Product_Revenue]    ← Top 10 SKUs
[Month_Order]        ← Helper table for correct month sort order
```

**Relationships:**
- `Monthly_Sales[Month]` → `Month_Order[Month]` (Many-to-One, Single filter direction)

### Step 4: DAX Measures

```dax
// Core KPIs
Total Sales = SUM(KPI_Summary[Total_Sales])

Total Orders = SUM(KPI_Summary[Order_Count])

Avg Order Value = DIVIDE([Total Sales], [Total Orders])

// Month-over-Month Change
MoM Sales Change % = 
VAR CurrentSales = [Total Sales]
VAR PrevSales = CALCULATE([Total Sales], DATEADD(Month_Order[MonthNum], -1, MONTH))
RETURN DIVIDE(CurrentSales - PrevSales, PrevSales, 0)

// Customer Concentration (Top N share)
Top5 Customer Share % = 
DIVIDE(
    CALCULATE(SUM(Customer_Revenue[Sales]), TOPN(5, Customer_Revenue, Customer_Revenue[Sales])),
    SUM(Customer_Revenue[Sales])
)
```

> Full DAX library: [`/dax/measures.dax`](./dax/measures.dax)

### Step 5: Visuals Layout

| Page | Visual Type | Fields Used |
|------|------------|-------------|
| Executive Summary | Card (×5) | Total Sales, Orders, Qty, Customers, Avg Order Value |
| Monthly Trend | Clustered Bar + Line Combo | Month, Sales (Bar); MoM Change % (Line) |
| Customer Revenue | Horizontal Bar | Customer_ID, Sales — sorted descending |
| Product Performance | Horizontal Bar | ProductCode, Sales — sorted descending |

### Step 6: Formatting & Design

- **Theme:** Custom corporate theme (`#1F3864` primary, `#F2C811` accent — Power BI yellow)
- **Font:** Segoe UI throughout
- **KPI Cards:** Conditional formatting — green if above average, amber below
- **Slicers:** None needed (pre-aggregated data); add a `Year` slicer if raw data is appended

---

## ✅ Business Recommendations

Based on the analysis, three actionable recommendations emerge:

1. **Capitalize on Q4 momentum** — allocate 30–35% of marketing budget to Sep–Nov campaigns to amplify the natural demand spike.

2. **Reduce customer concentration risk** — with CUS-22 representing ~9.3% of revenue, develop a customer diversification strategy targeting mid-tier accounts (CUS-10 through CUS-50 range).

3. **Invest in top SKU line extensions** — S18-3232's 2× revenue premium over peers signals strong brand affinity for this model family; prioritize inventory and bundling strategies around this SKU.

---

## 🧰 Tools & Technologies

| Tool | Purpose |
|------|---------|
| **Power BI Desktop** | Dashboard development, DAX, data modeling |
| **Power Query (M)** | Data transformation & shaping |
| **Microsoft Excel** | Source data format |
| **DAX** | Measures, KPIs, calculated columns |

---

## 👤 Author

**Martin Chukwu**  
Data Analyst  
📧 chuksmart814@gmail.com  
🔗 [LinkedIn](https://linkedin.com/in/chukwumartin) | [Portfolio](https://martinez1341.github.io)

---
