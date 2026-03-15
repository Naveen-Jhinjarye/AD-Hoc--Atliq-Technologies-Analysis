# 💼 Ad-Hoc Business Analysis — AtliQ Hardwares

![MySQL](https://img.shields.io/badge/MySQL-005C84?style=flat&logo=mysql&logoColor=white)
![Jira](https://img.shields.io/badge/Jira-0052CC?style=flat&logo=jira&logoColor=white)
![Kanban](https://img.shields.io/badge/Methodology-Kanban-brightgreen?style=flat)
![Stored Procedures](https://img.shields.io/badge/SQL-Stored%20Procedures-blue?style=flat)
![Views & Joins](https://img.shields.io/badge/SQL-Views%20%26%20Joins-4A90D9?style=flat)
![Ad-hoc Analytics](https://img.shields.io/badge/Analysis-Ad--hoc%20Analytics-orange?style=flat)
![Kanban](https://img.shields.io/badge/Methodology-Kanban-brightgreen?style=flat)
![Window Functions](https://img.shields.io/badge/SQL-Window%20Functions-9B59B6?style=flat)
![CTEs](https://img.shields.io/badge/SQL-CTEs-27AE60?style=flat)
![Functions](https://img.shields.io/badge/SQL-Functions-E74C3C?style=flat)

---

## 🏢 About the Company

AtliQ Hardwares is a computer hardware company that sells products worldwide
through channels like Amazon, Croma, and Best Buy. They work with both
businesses and individual customers across many markets.

---

## 🔍 Problem Statement

AtliQ was running everything on Excel. As the business grew, their Excel files
got too large and slow to handle. One day, a critical business file crashed and
could not be recovered.

This was the turning point. AtliQ moved to a **MySQL database** and hired a data
analytics team to answer important business questions using SQL — helping
the company make smarter, faster decisions.

---

## 🎯 Project Objectives

| # | Objective | Description |
|---|-----------|-------------|
| 01 | **Ad-hoc SQL Analysis** | Answer real business questions directly from the database |
| 02 | **Automate Repetitive Reporting** | Use stored procedures to eliminate manual querying |
| 03 | **Build Reusable Query Assets** | Views and modular queries for rapid report generation |
| 04 | **Enable Data-Driven Decisions** | Empower the Product Owner with timely, accurate insights |

---

## 🗂️ Project Management Approach

### 🔷 Jira
All tasks tracked as Jira tickets — each SQL analysis request logged, assigned,
and closed with documented output.

### 🟩 Kanban Methodology
Work visualized on a Kanban board — tasks moved across
`To Do → In Progress → Done` for clear, continuous delivery tracking.

---

## 📊 Business Focus Areas

- 📈 Sales Trends
- 🌍 Market Performance
- 📦 Product Demand
- 🎯 Forecast Accuracy
- 🛒 Customer & Channel Strategy

---

## 🛠️ Tech Stack & Tools

| Category | Tool / Technology |
|----------|------------------|
| Database | MySQL |
| IDE | MySQL Workbench |
| Project Management | Jira + Kanban Board |
| SQL Concepts | Joins, CTEs, Views, Stored Procedures, Functions,Window Functions |
| Domain | Consumer Electronics / Hardware |

---

> **Type:** Advanced Course Project &nbsp;|&nbsp; **Methodology:** Kanban via Jira

---

## 🗄️ Dataset Overview

The database `gdb0041` contains the following tables used across all analyses:

### 📋 Dimension Tables
| Table | Description |
|-------|-------------|
| `dim_customer` | Customer details — customer_code,customer,platform,channel,market,subregion,region|
| `dim_product` | Product details — product_code,segment, division, product,category, variant |
| `dim_date` | Custom fiscal calendar (Sep–Aug cycle) · Range: `2017-09-01` to `2021-12-01` |

### 📦 Fact Tables
| Table | Description |
|-------|-------------|
| `fact_sales_monthly` | Monthly sales quantity per customer & product |
| `fact_pre_invoice_deductions` | Pre-invoice discount % per customer per fiscal year |
| `fact_post_invoice_deductions` | Post-invoice & other deductions |
| `fact_gross_price` | Gross price per product per fiscal year |
| `fact_freight_cost` | Freight & other cost % per market per fiscal year |
| `fact_manufacturing_cost` | Manufacturing cost per product per fiscal year |
| `fact_forecast_monthly` | Monthly forecast quantity per customer & product |

### 🔧 Custom Tables Created
| Table | Description |
|-------|-------------|
| `dim_date` | Manually created fiscal date table — maps each date to fiscal month, quarter & year (fiscal year: Sep–Aug) |
| `fact_act_est` | Combined actual sales + forecast quantity in one place — built for supply chain & forecast accuracy analysis |

---

## 📋 Ad-Hoc Requests & SQL Solutions

---

### 🔖 Request 01 — Croma India Product-Wise Sales Report

> **As a Product Owner**, I want a monthly product-level sales report for
> Croma India (FY 2021) to track individual product performance and run
> further analysis in Excel.

**Output Fields:** Month · Product Name · Variant · Sold Quantity · Gross Price Per Item · Gross Price Total

---

#### ⚙️ Step 1 — Created a Reusable Function

To handle AtliQ's **Sep–Aug fiscal year**, a custom MySQL function was built
that converts any calendar date into its correct fiscal year.
```sql
CREATE FUNCTION `get_fiscal_year`(Calender_date DATE)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE fiscal_year INT;
    SET fiscal_year = YEAR(DATE_ADD(Calender_date, INTERVAL 4 MONTH));
    RETURN fiscal_year;
END
```

> 💡 **Logic:** Adding 4 months to any date shifts September → January,
> making `YEAR()` return the correct fiscal year automatically.

---

#### 🔍 Step 2 — Sales Report Query
```sql
SELECT
    fsm.date,
    fsm.product_code,
    p.product,
    p.variant,
    fsm.sold_quantity,
    g.gross_price,
    ROUND(sold_quantity * gross_price, 2) AS gross_price_total
FROM fact_sales_monthly fsm
JOIN dim_product p
    ON fsm.product_code = p.product_code
JOIN fact_gross_price g
    ON g.product_code = fsm.product_code
    AND g.fiscal_year = get_fiscal_year(fsm.date)
WHERE fsm.customer_code = 90002002
    AND get_fiscal_year(fsm.date) = 2021
ORDER BY fsm.date ASC
LIMIT 1000000;
```

**Tables Used:**
| Table | Purpose |
|-------|---------|
| `fact_sales_monthly` | Monthly sold quantity per customer & product |
| `dim_product` | Product name & variant details |
| `fact_gross_price` | Gross price per product per fiscal year |

---

#### 📊 Result

![Query Result](https://github.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/blob/main/query%20code%20and%20result%20image/Screenshot%20(627).png)

---
---

### 🔖 Request 02 — Monthly Gross Sales Report (Stored Procedure)

> **As a Data Analyst**, I want a stored procedure for monthly gross sales
> so that any user with limited DB access can generate this report
> independently — without involving the analytics team every time.

**Output Fields:** Month · Total Gross Sales

---

#### ⚙️ Solution — Stored Procedure

Instead of running a manual query each time, a reusable **Stored Procedure**
was created. It accepts any customer code as input and returns their
monthly gross sales automatically.
```sql
CREATE PROCEDURE `get_monthly_gross_sales_for_customers`(
    in_customer_code TEXT
)
BEGIN
    SELECT
        sm.date,
        ROUND(SUM(gp.gross_price * sm.sold_quantity), 2) AS total_gross_sales
    FROM fact_sales_monthly sm
    JOIN fact_gross_price gp
        ON sm.product_code = gp.product_code
        AND gp.fiscal_year = get_fiscal_year(sm.date)
    WHERE FIND_IN_SET(sm.customer_code, in_customer_code) > 0
    GROUP BY sm.date;
END
```
![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(646).png)

> 💡 **Why `FIND_IN_SET()`?** It allows passing **multiple customer codes**
> as a comma-separated string — making the procedure flexible for both
> single and multiple customers in one call.

---

#### 🚀 How to Use
```sql
-- Single customer
CALL get_monthly_gross_sales_for_customers('90002002');

-- Multiple customers
CALL get_monthly_gross_sales_for_customers('90002002,90002003,90002004');
```

---

#### 🔑 Key Benefit

| Without Stored Proc | With Stored Proc |
|---------------------|-----------------|
| Manual query edit every time | One `CALL` does the job |
| Only analysts can run it | Any DB user can execute |
| Error-prone & time-consuming | Fast, reusable & reliable |

---

**Tables Used:**
| Table | Purpose |
|-------|---------|
| `fact_sales_monthly` | Monthly sold quantity per customer & product |
| `fact_gross_price` | Gross price per product per fiscal year |

---

#### 📊 Result

![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(647).png)

---

---

### 🔖 Request 04 — Top Markets, Products & Customers by Net Sales

> **As a Product Owner**, I want a report for top markets, products, and
> customers by net sales for a given fiscal year — to get a holistic view
> of AtliQ's financial performance and make better business decisions.

---

#### 🏗️ Architecture — How It Was Built

Before writing stored procedures, a **layered view approach** was used to
keep the logic clean, reusable, and easy to maintain.
```
fact_sales_monthly
       ↓
  Gross Sales View          ← gross price × sold quantity
       ↓
  Pre-Invoice Deductions View    ← apply pre-invoice discount %
       ↓
  Post-Invoice Deductions View   ← apply post-invoice discount %
       ↓
  ✅ net_sales_view              ← final clean net sales table
```

> 💡 **Performance Note:** Query performance was explored using `dim_date`
> and an optimized generated column was added directly in
> `fact_sales_monthly` — reducing repeated function calls and speeding
> up query execution significantly.

---

#### 👁️ Step 1 — Net Sales View

Combines all deductions into one final clean view with `Net_Sales` ready
for analysis.
```sql
CREATE VIEW `net_sales_view` AS
    SELECT
        date, fiscal_year,
        product_code, customer_code,
        customer, market,
        product, variant,
        sold_quantity, gross_price,
        gross_price_total,
        pre_invoice_discount_pct,
        Net_invoice_Sales,
        post_invoice_deductions_pct,
        ROUND(
            (1 - post_invoice_deductions_pct) * Net_invoice_Sales
        , 2) AS Net_Sales
    FROM net_post_invoice_deductions_table_view;
```

---
#### 📊 Result

![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(757).png)

#### ⚙️ Step 2 — Three Stored Procedures

---

**🏆 Top N Products by Net Sales**
```sql
CREATE PROCEDURE `top_n_products_by_net_sales`(
    IN in_fiscalYear INT,
    IN in_top_p      INT
)
BEGIN
    SELECT
        product,
        ROUND(SUM(net_sales) / 1000000, 2) AS net_sales_mln
    FROM net_sales_view
    WHERE fiscal_year = in_fiscalYear
    GROUP BY product
    ORDER BY net_sales_mln DESC
    LIMIT in_top_p;
END
```

---
#### 📊 Result

![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(667).png)

**🌍 Top N Markets by Net Sales**
```sql
CREATE PROCEDURE `top_n_market_by_net_sales`(
    IN in_fY     INT,
    IN in_top_n  INT
)
BEGIN
    SELECT
        market,
        ROUND(SUM(net_sales) / 1000000, 2) AS net_sales_mln
    FROM net_sales_view
    WHERE fiscal_year = in_fY
    GROUP BY market
    ORDER BY net_sales_mln DESC
    LIMIT in_top_n;
END
```

---
#### 📊 Result

![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(665).png)


**🤝 Top N Customers by Net Sales**
```sql
CREATE PROCEDURE `top_n_customer_by_net_sales`(
    IN in_market     VARCHAR(45),
    IN in_fiscalYear INT,
    IN in_top_c      INT
)
BEGIN
    SELECT
        customer,
        ROUND(SUM(net_sales) / 1000000, 2) AS net_sales_mln
    FROM net_sales_view
    WHERE fiscal_year = in_fiscalYear
        AND market = in_market
    GROUP BY customer
    ORDER BY net_sales_mln DESC
    LIMIT in_top_c;
END
```

---
#### 📊 Result

![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(668).png)


#### 🚀 How to Use
```sql
-- Top 5 products in FY 2021
CALL top_n_products_by_net_sales(2021, 5);

-- Top 3 markets in FY 2021
CALL top_n_market_by_net_sales(2021, 3);

-- Top 3 customers in India for FY 2021
CALL top_n_customer_by_net_sales("India", 2021, 3);
```

---

#### 🔑 Why Three Separate Procedures?

| Procedure | Filters By | Groups By |
|-----------|-----------|-----------|
| `top_n_products_by_net_sales` | Fiscal Year | Product |
| `top_n_market_by_net_sales` | Fiscal Year | Market |
| `top_n_customer_by_net_sales` | Fiscal Year + Market | Customer |

> 💡 **Net Sales values divided by 1,000,000** — reported in **millions (MLN)**
> for cleaner, more readable output.

---

**Views Used:**
| View | Purpose |
|------|---------|
| `net_sales_view` | Final net sales after all deductions |
| `net_post_invoice_deductions_table_view` | Base view feeding into net sales |

---

---

### 🔖 Request 05 — Net Sales % Share by Region

> **As a Product Owner**, I want a region-wise percentage breakdown of net
> sales by customer (APAC, EU, LTAM, etc.) for any given fiscal year —
> to analyze AtliQ's financial performance across global regions.

**Output:** % Net Sales share per customer within each region · Visualized as Pie Charts

---

#### ⚙️ Solution — CTE + Window Function

This query uses a **CTE** to first calculate net sales per customer per
region, then applies a **Window Function** to compute each customer's
% share within their region — all in one clean pass.
```sql
WITH cte1 AS (
    SELECT
        c.customer,
        c.region,
        ROUND(SUM(net_sales) / 1000000, 2) AS net_sales_mln
    FROM net_sales_view n
    JOIN dim_customer c
        ON c.customer_code = n.customer_code
    WHERE fiscal_year = 2021
    GROUP BY c.customer, c.region
)
SELECT
    *,
    ROUND(
        net_sales_mln * 100 / SUM(net_sales_mln) OVER (PARTITION BY region)
    , 2) AS pct_share
FROM cte1
ORDER BY region, net_sales_mln DESC;
```

---

![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(679).png)

#### 🧠 Query Breakdown

| Concept | Usage |
|---------|-------|
| `CTE (cte1)` | Aggregates net sales per customer & region |
| `SUM() OVER (PARTITION BY region)` | Calculates total sales within each region |
| `net_sales_mln * 100 / regional_total` | Converts to % share per customer |
| `ORDER BY region, net_sales_mln DESC` | Groups regions & ranks customers by sales |

> 💡 **Why Window Function?** `PARTITION BY region` computes the regional
> total without collapsing rows — so each customer keeps their own row
> while still seeing the full regional context.

---

#### 🔁 Reusable — Change Fiscal Year Anytime
```sql
-- FY 2020
WHERE fiscal_year = 2020

-- FY 2019
WHERE fiscal_year = 2019
```

Simply swap the year — the query handles the rest automatically.

---

**Views & Tables Used:**
| Asset | Purpose |
|-------|---------|
| `net_sales_view` | Final net sales after all deductions |
| `dim_customer` | Customer name & region mapping |

---

#### 📊 Result — Top 10 Customers by Net Sales % per Region (FY 2021)

![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(678).png)

---
---

### 🔖 Request 06 — Net Sales % Share Global (Top 10 Customers)

> **As a Product Owner**, I want a global view of top 10 customers by
> % net sales contribution for FY 2021 — to understand which customers
> are driving the most revenue worldwide.

**Output:** Top 10 Customers · % Net Sales Share · Bar Chart in Excel

---

#### ⚙️ Solution — CTE + Window Function
```sql
WITH net_sales_pct AS (
    SELECT
        customer,
        fiscal_year,
        ROUND(SUM(net_sales) / 1000000, 2) AS net_sales_mln
    FROM net_sales_view
    WHERE fiscal_year = 2021
    GROUP BY customer
)
SELECT
    *,
    ROUND(
        net_sales_mln * 100 / SUM(net_sales_mln) OVER()
    , 2) AS pct_share
FROM net_sales_pct
ORDER BY net_sales_mln DESC;
```

---
![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(680).png)

#### 🧠 Request 05 vs Request 06 — Key Difference

| | Request 05 | Request 06 |
|--|-----------|-----------|
| **Scope** | % share within each region | % share across global total |
| **Window** | `PARTITION BY region` | No partition — full global sum |
| **View** | Region-wise breakdown | Single global ranking |
| **Output** | Multiple regional charts | One global Top 10 bar chart |

> 💡 **No `PARTITION BY` here** — `SUM() OVER()` without partition runs
> across the **entire result set**, giving each customer their share of
> the global total rather than a regional total.

---

#### 📊 Visualization — Excel Bar Chart

SQL output was exported to Excel and a **Top 10 Customers Bar Chart**
was built to visually rank customers by their global net sales % share.

---

**View Used:**
| View | Purpose |
|------|---------|
| `net_sales_view` | Final net sales after all deductions |

---

#### 📊 Result

![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(675).png)

---

---

### 🔖 Request 07 — Top N Products per Division by Quantity Sold

> **As a Product Owner**, I want a stored procedure that returns the top N
> products within each division ranked by total quantity sold — for any
> given fiscal year.

**Output Fields:** Division · Product · Total Quantity · Rank

---

#### ⚙️ Solution — Stored Procedure with CTE + Dense Rank
```sql
CREATE PROCEDURE `top_n_products_by_division_by_qty_sold`(
    IN in_fiscal_year INT,
    IN in_top_n       INT
)
BEGIN
    WITH top_n_division_n_product AS (
        SELECT
            p.division,
            n.product,
            SUM(n.sold_quantity) AS total_quantity
        FROM net_sales_view n
        JOIN dim_product p
            ON n.product_code = p.product_code
        WHERE n.fiscal_year = in_fiscal_year
        GROUP BY p.division, n.product
    ),
    cte_2 AS (
        SELECT *,
            DENSE_RANK() OVER (
                PARTITION BY division
                ORDER BY total_quantity DESC
            ) AS rnk
        FROM top_n_division_n_product
    )
    SELECT * FROM cte_2
    WHERE rnk <= in_top_n;
END
```

---

#### 🧠 Query Breakdown

| Concept | Usage |
|---------|-------|
| `CTE 1` | Aggregates total sold quantity per product per division |
| `CTE 2` | Applies `DENSE_RANK()` within each division |
| `PARTITION BY division` | Ranks products independently inside each division |
| `ORDER BY total_quantity DESC` | Highest selling product gets rank 1 |
| `WHERE rnk <= in_top_n` | Filters only top N products per division |

> 💡 **Why `DENSE_RANK()` over `ROW_NUMBER()`?** If two products have the
> exact same quantity, `DENSE_RANK()` gives them the same rank — no
> product is unfairly excluded from the top N.

---

#### 🚀 How to Use
```sql
-- Top 3 products per division for FY 2021
CALL top_n_products_by_division_by_qty_sold(2021, 3);

-- Top 5 products per division for FY 2020
CALL top_n_products_by_division_by_qty_sold(2020, 5);
```


**Assets Used:**
| Asset | Purpose |
|-------|---------|
| `net_sales_view` | Final net sales with sold quantity |
| `dim_product` | Product name & division mapping |

---

#### 📊 Result


![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(706).png)

---
---

### 🔖 Request 08 — Forecast Accuracy Report (Supply Chain)

> **As a Product Owner**, I want an aggregate forecast accuracy report for
> all customers in a given fiscal year — to track how well demand forecasts
> matched actual sales and improve future supply chain planning.

**Output Fields:** Customer Code · Customer Name · Market · Sold Qty ·
Forecast Qty · Net Error · Net Error % · Abs Error · Abs Error % · Forecast Accuracy %

---

#### 🏗️ Step 1 — Helper Table: `fact_act_estimate`

Before writing the forecast query, a **combined helper table** was created
that merges actual sales and forecast data into one unified table — making
supply chain analysis clean and efficient.
```sql
CREATE TABLE fact_act_estimate (
    SELECT
        s.date              AS date,
        s.fiscal_year       AS fiscal_year,
        s.product_code      AS product_code,
        s.customer_code     AS customer_code,
        s.sold_quantity     AS sold_quantity,
        f.forecast_quantity AS forecast_quantity
    FROM fact_sales_monthly s
    LEFT JOIN fact_forecast_monthly f
        USING (date, product_code, customer_code)

    UNION

    SELECT
        f.date              AS date,
        f.fiscal_year       AS fiscal_year,
        f.product_code      AS product_code,
        f.customer_code     AS customer_code,
        s.sold_quantity     AS sold_quantity,
        f.forecast_quantity AS forecast_quantity
    FROM fact_forecast_monthly f
    LEFT JOIN fact_sales_monthly s
        USING (date, product_code, customer_code)
);
```

> 💡 **Why two LEFT JOINs with UNION?**
> - First join → captures all **actual sales** records, even if no forecast exists
> - Second join → captures all **forecast** records, even if no actual sale occurred
> - Together they ensure **zero data loss** — a full outer join equivalent in MySQL

---

#### ⚙️ Step 2 — Stored Procedure: Forecast Accuracy Report
```sql
CREATE PROCEDURE `get_forecast_accuracy_report`(
    IN in_fiscal_year INT
)
BEGIN
    WITH fcst_err_t AS (
        SELECT
            customer_code,
            SUM(sold_quantity)                    AS sold_quantity,
            SUM(forecast_quantity)                AS forecast_quantity,
            SUM(forecast_quantity - sold_quantity) AS net_err,
            ROUND(
                SUM(forecast_quantity - sold_quantity) * 100
                / SUM(forecast_quantity)
            , 2)                                  AS net_pct_err,
            SUM(ABS(forecast_quantity - sold_quantity)) AS abs_err,
            ROUND(
                SUM(ABS(forecast_quantity - sold_quantity)) * 100
                / SUM(forecast_quantity)
            , 2)                                  AS abs_pct_err
        FROM fact_act_estimate a
        WHERE a.fiscal_year = in_fiscal_year
        GROUP BY customer_code
    )
    SELECT
        e.customer_code,
        c.customer,
        c.market,
        e.sold_quantity,
        e.forecast_quantity,
        e.net_err,
        e.net_pct_err,
        e.abs_err,
        e.abs_pct_err,
        IF(abs_pct_err > 100, 0, 100 - abs_pct_err) AS forecast_accuracy
    FROM fcst_err_t e
    JOIN dim_customer c
        USING (customer_code)
    ORDER BY forecast_accuracy DESC;
END
```

---

#### 🧠 Forecast Accuracy Metrics Explained

| Metric | Formula | What It Tells Us |
|--------|---------|-----------------|
| `net_err` | `SUM(forecast - actual)` | Overall over/under forecasting direction |
| `net_pct_err` | `net_err × 100 / forecast` | Net error as a % of total forecast |
| `abs_err` | `SUM(ABS(forecast - actual))` | Total error magnitude — ignores direction |
| `abs_pct_err` | `abs_err × 100 / forecast` | Absolute error as a % of total forecast |
| `forecast_accuracy` | `100 - abs_pct_err` | Final accuracy % — higher is better |

> 💡 **Why `IF(abs_pct_err > 100, 0, ...)`?**
> If absolute error exceeds 100% of forecast, accuracy would go negative —
> which is meaningless. This condition floors it at `0%` instead, keeping
> results clean and interpretable.

---

#### 🚀 How to Use
```sql
-- Forecast accuracy for FY 2021
CALL get_forecast_accuracy_report(2021);

-- Forecast accuracy for FY 2020
CALL get_forecast_accuracy_report(2020);
```

---

**Assets Used:**
| Asset | Purpose |
|-------|---------|
| `fact_act_estimate` | Combined actual + forecast helper table |
| `dim_customer` | Customer name & market mapping |

---

#### 📊 Result


![SQL Query Result](https://raw.githubusercontent.com/Naveen-Jhinjarye/AD-Hoc--Atliq-Technologies-Analysis/main/query%20code%20and%20result%20image/Screenshot%20(717).png)

---

## 🎯 Project Summary

| # | Request | Key SQL Concept |
|---|---------|----------------|
| 01 | Croma India Product Sales Report | JOIN + Custom Function |
| 02 | Monthly Gross Sales Report | Stored Procedure |
| 03 | Market Badge Classification | Stored Proc + IN/OUT Parameters |
| 04 | Top Markets, Products & Customers | Layered Views + 3 Stored Procedures |
| 05 | Net Sales % Share by Region | CTE + Window Function (Partition) |
| 06 | Net Sales % Share Global | CTE + Window Function (Global) |
| 07 | Top N Products per Division | CTE + Dense Rank |
| 08 | Forecast Accuracy Report | Helper Table + Error Formula + Stored Proc |

---

> *Domain: Consumer Electronics · Database: MySQL · Project Type: Advanced Course Project · Methodology: Kanban via Jira*
