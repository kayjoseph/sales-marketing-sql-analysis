Walmart Sales Data — SQL Analysis

Tool: PostgreSQL 16
Dataset: Sample Sales Data — Kaggle
Database: walmart_db | Table: sales
Total Records: 2,823 rows | Columns: 24


Table of Contents

1. Project Overview
2. Dataset Columns
3. Analysis Questions & Queries

Project Overview
This project analyses a sales dataset covering multiple territories (NA, EMEA, APAC, Japan) across different product lines, customer types, and deal sizes. The goal is to extract business insights around revenue performance, product profitability, customer behaviour, and sales trends using intermediate to advanced PostgreSQL queries.

Key SQL concepts used in this project:

. Aggregations (SUM, AVG, COUNT, ROUND)
. Window functions (SUM() OVER, RANK() OVER, LAG())
. Common Table Expressions (WITH)
. Subqueries and CASE WHEN
. Date functions (EXTRACT, DATE_TRUNC)
. Rolling averages

Analysis Questions & Queries

1. What is the total revenue, COGS, and gross profit per territory?

SELECT 
    territory,
    ROUND(SUM(sales::numeric), 2) AS total_sales,
    ROUND(SUM(quantityordered::int * msrp::numeric), 2) AS total_cogs,
    ROUND(SUM(sales::numeric) - SUM(quantityordered::int * msrp::numeric), 2) AS gross_profit,
    ROUND(
        (SUM(sales::numeric) - SUM(quantityordered::int * msrp::numeric)) 
        * 100.0 / NULLIF(SUM(sales::numeric), 0)
    , 2) AS profit_margin_pct
FROM sales
GROUP BY 1
ORDER BY 3 DESC;

2. Which product line has the highest profit margin — and how does it vary across territories?


select 
	productline,
	territory,
	round(sum(sales::numeric), 2) as total_revenue,
	round(sum(quantityordered::numeric * msrp::numeric), 2) as total_cogs,
	round(sum(sales::numeric) - sum(quantityordered::numeric * msrp::numeric), 2) as gross_profit

from sales
group by 1,2 
order by 1,5

