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

AtliQ Hardwares is a global computer hardware manufacturer distributing products
through leading retail channels including Amazon, Croma, and Best Buy. Operating
across multiple markets, the company serves both B2B and B2C segments with a
diverse product portfolio.

---

## 🔍 Problem Statement

As sales and operations grew, AtliQ's complete dependence on Excel became a
serious liability. Business-critical planning files swelled beyond manageable
limits — and a catastrophic file crash became the final trigger for change.

AtliQ responded by migrating to a structured **MySQL database** and building a
dedicated analytics team. The team's mandate: transform raw transactional data
into clear, timely business intelligence — answering real questions from
stakeholders across product, sales, and supply chain.

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
| SQL Concepts | Joins, CTEs, Views, Stored Procedures, Window Functions |
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
