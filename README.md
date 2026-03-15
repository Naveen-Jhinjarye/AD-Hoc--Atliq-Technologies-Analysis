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
