# Data-Engineering-Sales-Analytics-Using-Microsoft-Fabric-Project
### 🛒 SA Retail Analytics - Microsoft Fabric End-to-End Data Engineering Project

> **A production-grade data engineering portfolio project using Microsoft Fabric, built on real South African retail data.**
> Covers Bronze → Silver → Gold medallion architecture, PySpark transformations, and Power BI dashboards.

---

## 📸 Architecture Overview

```
JSON API (SA Retail Data)
        │
        ▼
┌───────────────────┐
│   BRONZE LAYER    │  ← Dataflow Gen2 (raw ingestion)
│  customer.csv     │
│  product.csv      │
│  orders.csv       │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│   SILVER LAYER    │  ← PySpark Notebook (cleaning + joins)
│ silver_customer   │
│ silver_product    │
│ silver_orders     │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│    GOLD LAYER     │  ← PySpark Notebook (aggregations)
│  gold_customer    │
│  gold_product     │
│  gold_segment     │
│  gold_category    │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│    POWER BI       │  ← Dashboard (DirectLake)
│  Sales Dashboard  │
└───────────────────┘
         │
    All orchestrated by
    Sales Pipeline (auto-scheduled)
```

---

## 🇿🇦 Business Context

This project simulates a South African multi-store retail analytics platform. The dataset includes:

- **Customers** across major SA cities (Johannesburg, Cape Town, Durban, Pretoria, etc.)
- **Products** from well-known SA retail brands (Sasko, Koo, Clover, Simba, etc.)
- **Orders** placed at SA retailers (Pick n Pay, Checkers, Woolworths, Shoprite, Spar, Clicks)
- **Currency**: ZAR (South African Rand)
- **Customer segments**: Premium, Regular, Budget

---

## 🛠️ Tech Stack

| Component | Tool |
|---|---|
| Storage | Microsoft Fabric OneLake (Lakehouse) |
| Ingestion | Dataflow Gen2 |
| Transformation | PySpark Notebooks |
| Orchestration | Microsoft Fabric Pipeline |
| Visualisation | Power BI (DirectLake mode) |
| Data Format | JSON → Delta Tables |

---

## 📁 Repository Structure

```
sa-retail-analytics/
├── README.md                        ← You are here
├── IMPLEMENTATION_PLAN.md           ← Step-by-step build guide
│
├── data/
│   ├── sales_data.json              ← Source dataset (SA retail)
│   └── generate_data.py             ← Script to regenerate/extend data
│
├── notebooks/
│   ├── 01_silver_cleaning.py        ← Silver layer PySpark notebook
│   └── 02_gold_analytics.py         ← Gold layer PySpark notebook
│
├── pipeline/
│   └── pipeline_config.json         ← Pipeline definition reference
│
├── powerbi/
   └── POWERBI_GUIDE.md             ← Power BI setup & DAX measures guide

```

---

## 🚀 Quick Start

### Prerequisites

- Microsoft Fabric workspace (free trial at [fabric.microsoft.com](https://fabric.microsoft.com))
- Power BI Desktop (free download)
- Python 3.9+ (to regenerate data locally)

### 1. Clone the repo

```bash
git clone https://github.com/Lehlohonolo-Saohatse/Data-Engineering-Sales-Analytics-Using-Microsoft-Fabric-Project.git
cd Data-Engineering-Sales-Analytics-Using-Microsoft-Fabric-Project
```

### 2. Follow the implementation plan

See [`IMPLEMENTATION_PLAN.md`](./IMPLEMENTATION_PLAN.md) for the complete step-by-step guide from workspace setup to Power BI dashboard.

### 3. Regenerate data (optional)

```bash
cd data
python generate_data.py
```

---

## 📊 Gold Layer Outputs

| Table | Description |
|---|---|
| `gold_customer` | Total orders, revenue & items per customer |
| `gold_product` | Quantity sold & revenue per product |
| `gold_segment` | Orders & revenue by customer segment (Premium/Regular/Budget) |
| `gold_category` | Performance by product category |

---

## 🗺️ Data Model

### customers
| Column | Type | Description |
|---|---|---|
| customer_id | STRING | Unique customer ID |
| name | STRING | Full name |
| email | STRING | Email address |
| phone | STRING | SA mobile number |
| city | STRING | City of residence |
| province | STRING | SA province |
| segment | STRING | Premium / Regular / Budget |

### products
| Column | Type | Description |
|---|---|---|
| product_id | STRING | Unique product ID |
| name | STRING | Product name |
| category | STRING | Product category |
| price | DOUBLE | Price in ZAR |
| stock | INT | Available stock units |

### orders
| Column | Type | Description |
|---|---|---|
| order_id | STRING | Unique order ID |
| customer_id | STRING | FK to customers |
| order_date | DATE | Date of order |
| store | STRING | Retail store name |
| payment_method | STRING | Card / Cash / EFT / SnapScan / Zapper |
| product_id | STRING | FK to products |
| product_name | STRING | Product name (denormalised) |
| category | STRING | Product category |
| quantity | INT | Units ordered |
| unit_price | DOUBLE | Unit price in ZAR |
| line_total | DOUBLE | quantity × unit_price in ZAR |
| order_total | DOUBLE | Total order value in ZAR |

---

## 📄 Licence

MIT - free to use for portfolio projects, learning, and commercial use.
