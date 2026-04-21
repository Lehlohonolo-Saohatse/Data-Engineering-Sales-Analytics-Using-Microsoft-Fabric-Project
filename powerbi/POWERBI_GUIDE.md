# 📊 Power BI Guide — SA Retail Analytics Dashboard

This guide covers connecting Power BI to your Fabric Lakehouse, building the data model, writing DAX measures, and designing all 4 dashboard pages.

---

## Step 1 — Connect to Fabric Lakehouse (DirectLake)

1. Open **Power BI Desktop** (download from powerbi.microsoft.com)
2. **Home** → **Get Data** → **Microsoft Fabric** → **Lakehouse**
3. Sign in with the Microsoft account you used for Fabric
4. Navigate: `SA Retail Analytics` workspace → `SalesStorage`
5. In the navigator, expand **Tables** — select all gold tables:
   - gold_customer
   - gold_product
   - gold_segment
   - gold_category
   - gold_monthly_trend
   - gold_store_performance
   - gold_payment_methods
6. Click **Load**

> **DirectLake mode** means Power BI reads directly from OneLake Delta tables — no import, no duplication, always fresh data.

---

## Step 2 — Data Model Relationships

Switch to **Model view** (left sidebar icon). Create these relationships by dragging fields:

| From Table | From Column | → | To Table | To Column | Cardinality |
|---|---|---|---|---|---|
| gold_customer | customer_id | → | gold_orders | customer_id | 1:many |
| gold_product | product_id | → | gold_orders | product_id | 1:many |

---

## Step 3 — DAX Measures

Create a blank table called **Measures** (Enter Data → leave empty → Load), then add all measures to it.

### KPI Measures

```dax
-- Total Revenue in ZAR
Total Revenue = 
SUMX(gold_customer, gold_customer[total_revenue])

-- Total Orders
Total Orders = 
SUMX(gold_customer, gold_customer[total_orders])

-- Total Customers
Total Customers = 
COUNTROWS(gold_customer)

-- Average Order Value
Avg Order Value = 
DIVIDE([Total Revenue], [Total Orders], 0)

-- Average Items Per Order
Avg Items Per Order = 
DIVIDE(SUMX(gold_customer, gold_customer[total_items]), [Total Orders], 0)
```

### Segment Measures

```dax
-- Premium Revenue Share %
Premium Revenue % = 
VAR PremRevenue = CALCULATE(SUM(gold_segment[total_revenue]), gold_segment[segment] = "PREMIUM")
RETURN DIVIDE(PremRevenue, [Total Revenue], 0) * 100

-- Budget Customer Count
Budget Customers = 
CALCULATE(SUM(gold_segment[total_customers]), gold_segment[segment] = "BUDGET")
```

### Trend Measures

```dax
-- Monthly Revenue (for line chart)
Monthly Revenue = SUM(gold_monthly_trend[monthly_revenue])

-- Revenue MoM % Change
Revenue MoM % = 
VAR CurrentPeriod = MAX(gold_monthly_trend[month_label])
VAR PrevPeriod = CALCULATE(
    SUM(gold_monthly_trend[monthly_revenue]),
    PREVIOUSMONTH(gold_monthly_trend[month_label])
)
VAR CurrRevenue = SUM(gold_monthly_trend[monthly_revenue])
RETURN IF(ISBLANK(PrevPeriod), BLANK(), DIVIDE(CurrRevenue - PrevPeriod, PrevPeriod) * 100)

-- Cumulative Revenue YTD
Revenue YTD = 
CALCULATE(
    SUM(gold_monthly_trend[monthly_revenue]),
    DATESYTD(gold_monthly_trend[month_label])
)
```

### Product Measures

```dax
-- Top Product by Revenue
Top Product = 
FIRSTNONBLANK(
    TOPN(1, VALUES(gold_product[product_name]), SUM(gold_product[total_revenue]), DESC),
    1
)

-- Top Category by Revenue
Top Category = 
FIRSTNONBLANK(
    TOPN(1, VALUES(gold_category[category]), SUM(gold_category[total_revenue]), DESC),
    1
)
```

---

## Step 4 — Dashboard Pages

### Page 1: Executive Overview

| Visual | Type | Fields |
|---|---|---|
| Total Revenue (ZAR) | KPI Card | [Total Revenue] |
| Total Orders | KPI Card | [Total Orders] |
| Average Order Value | KPI Card | [Avg Order Value] |
| Total Customers | KPI Card | [Total Customers] |
| Revenue by Province | Clustered Bar | gold_customer[province], [Total Revenue] |
| Monthly Revenue Trend | Line Chart | gold_monthly_trend[month_label], [Monthly Revenue] |
| Revenue by Segment | Donut Chart | gold_segment[segment], gold_segment[total_revenue] |

**Formatting tips:**
- Currency format: `R #,##0.00` (South African Rand)
- Use brand colours: green (#00843D) for positive, red (#E2001A) for negative KPI trends
- Add a slicer for `order_year` to filter by year

---

### Page 2: Product Performance

| Visual | Type | Fields |
|---|---|---|
| Top 10 Products | Horizontal Bar | gold_product[product_name], gold_product[total_revenue] → TopN filter = 10 |
| Revenue by Category | Treemap | gold_category[category], gold_category[total_revenue] |
| Category Comparison | Column Chart | gold_category[category], gold_category[total_units_sold], gold_category[total_revenue] |
| Product Detail Table | Table | product_name, category, total_units_sold, total_revenue, product_rank |

---

### Page 3: Customer Analysis

| Visual | Type | Fields |
|---|---|---|
| Top Customers | Table | name, city, segment, total_orders, total_revenue, revenue_rank |
| Revenue by City | Map (ArcGIS or Bing) | gold_customer[city], gold_customer[total_revenue] |
| Customers by Segment | Stacked Bar | segment, total_customers, total_revenue |
| Customer Scatter | Scatter Chart | X: total_orders, Y: total_revenue, Size: total_items, Legend: segment |

> For the SA map to work, ensure city names include province or append `", South Africa"` to the city column in Power Query.

---

### Page 4: Store & Payments

| Visual | Type | Fields |
|---|---|---|
| Revenue by Store | Bar Chart | gold_store_performance[store], gold_store_performance[total_revenue] |
| Payment Methods Split | Donut Chart | gold_payment_methods[payment_method], gold_payment_methods[total_orders] |
| Store Performance Table | Matrix | store, total_orders, unique_customers, avg_basket_size, total_revenue |
| Payment × Revenue | Clustered Bar | payment_method, total_revenue, total_orders |

---

## Step 5 — Formatting & Design

### Colour Theme
Use the South African-inspired palette below in **View → Themes → Customise**:

```json
{
  "name": "SA Retail Theme",
  "dataColors": [
    "#00843D", "#1C6B32", "#FFB81C", "#E2001A",
    "#005A9C", "#6E2B8B", "#00A6A0", "#F47920"
  ],
  "background": "#FFFFFF",
  "foreground": "#333333",
  "tableAccent": "#00843D"
}
```

### Recommended Fonts
- Title: Segoe UI Semibold, 16pt
- Body: Segoe UI, 12pt
- Numbers: Segoe UI, 14pt, bold

---

## Step 6 — Publish to Fabric

1. **File** → **Publish** → **Publish to Power BI**
2. Select workspace: `SA Retail Analytics`
3. Name: `SA Retail Sales Dashboard`
4. Once published, open in Fabric portal
5. Pin key visuals to a new **Dashboard** for at-a-glance monitoring

### Scheduled Refresh
Since you used DirectLake, data refreshes automatically when the pipeline writes new Delta table versions — no separate refresh schedule needed!

---

## Bonus: Row-Level Security (RLS)

To restrict data by province (e.g. only show Western Cape data to WC managers):

1. In Power BI Desktop → **Modelling** → **Manage Roles**
2. Create role: `Western Cape Manager`
3. Filter: `gold_customer[province] = "Western Cape"`
4. Test via **View As Role**
5. After publishing, assign users to roles in the Fabric portal under **Security**
