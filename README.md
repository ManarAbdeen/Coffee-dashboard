# ☕ Grid Sales Pricing Dashboard

A Power BI dashboard for analyzing coffee/grocery product sales, pricing, and profitability across regions and time — built on top of a unified SQL view that consolidates multi-year order data.

![Dashboard Preview](Screenshot%202026-08-20%20175215.png)

## 📊 Overview

This dashboard provides a consolidated view of sales performance, allowing stakeholders to track revenue, profit, and customer trends across product categories, regions, and time periods.

**Key metrics tracked:**
- 💰 **Revenue** — $871.3K
- 👥 **Customers** — 4.432K
- 📦 **Quantity** — 11K units
- 📈 **Profit** — $481.9K

## 🧩 Features

- **Year filter** (2023 / 2024 / 2025) to slice all visuals dynamically
- **Revenue by Product** — matrix breakdown by product name and region (East, North, South, West, Unassigned)
- **Revenue by Product Category** — donut chart showing category share (Grinders & Brewers, Subscriptions, Accessories, Merchandise, Consumables)
- **Revenue by Year, Quarter, and Region** — stacked area chart showing seasonal and regional trends over time

## 🗃️ Data Model

The dashboard is powered by a single SQL Server view, `vw_PowerBI_Pricing_Analysis`, which:

1. Combines order data from three yearly tables (`Orders_2023`, `Orders_2024`, `Orders_2025`) using `UNION ALL`
2. Joins in customer data (`Customers`) to enrich orders with region and join date
3. Joins in product data (`Products`) to enrich orders with product name, category, and pricing
4. Cleans missing revenue values by recalculating `Price × Quantity` when `Revenue` is `NULL`
5. Derives a `Profit` column (`CleanedRevenue - COGS`) to avoid nulls flowing into Power BI visuals

```sql
CREATE OR ALTER VIEW vw_PowerBI_Pricing_Analysis AS

WITH all_orders AS (
    SELECT OrderID, CustomerID, ProductID, OrderDate, Quantity, Revenue, COGS 
    FROM Orders_2023
    UNION ALL
    SELECT OrderID, CustomerID, ProductID, OrderDate, Quantity, Revenue, COGS 
    FROM Orders_2024
    UNION ALL
    SELECT OrderID, CustomerID, ProductID, OrderDate, Quantity, Revenue, COGS 
    FROM Orders_2025
)

SELECT 
    a.OrderID,
    a.CustomerID,
    ISNULL(c.Region, 'Unassigned') AS Region,
    a.ProductID,
    a.OrderDate,
    c.CustomerJoinDate,
    a.Quantity,
    a.Revenue,
    CASE 
        WHEN a.Revenue IS NULL THEN p.Price * a.Quantity 
        ELSE a.Revenue 
    END AS CleanedRevenue,
    (CASE WHEN a.Revenue IS NULL THEN p.Price * a.Quantity ELSE a.Revenue END) - a.COGS AS Profit,
    a.COGS,
    p.ProductName,
    p.ProductCategory,
    p.Price,
    p.Base_Cost

FROM all_orders a
LEFT JOIN Customers c ON a.CustomerID = c.CustomerID
LEFT JOIN Products p  ON a.ProductID = p.ProductID;
```

The full version of this view with Arabic comments is available in [`vw_PowerBI_Pricing_Analysis.sql`](./vw_PowerBI_Pricing_Analysis.sql).

### Source Tables

| Table | Description |
|---|---|
| `Orders_2023`, `Orders_2024`, `Orders_2025` | Yearly order/transaction records |
| `Customers` | Customer master data (region, join date) |
| `Products` | Product master data (name, category, price, base cost) |

## 🚀 Setup

1. **Deploy the SQL view** to your SQL Server database:
   ```sql
   -- Run vw_PowerBI_Pricing_Analysis.sql against your database
   ```
2. **Connect Power BI** to the database and import `vw_PowerBI_Pricing_Analysis` as a data source.
3. **Open the `.pbix` file** (if included in this repo) or rebuild the visuals using the metrics and columns described above.
4. Refresh the dataset to pull the latest data from the view.

## 🛠️ Tech Stack

- **SQL Server** — data consolidation and cleaning via a database view
- **Power BI** — dashboard visualization and interactivity

## 📁 Repository Structure

```
.
├── README.md
├── vw_PowerBI_Pricing_Analysis.sql        # SQL view powering the dashboard
└── Screenshot 2026-08-20 175215.png       # Dashboard screenshot
```

## 🔗 Connect

**Manar Abdeen** — [LinkedIn](https://www.linkedin.com/in/manar-abdeen/)
