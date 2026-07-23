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

3. What is the running total of revenue per branch ordered by date?
SELECT
    territory,
    orderdate::date AS order_date,
    ROUND(SUM(sales::numeric), 2) AS daily_revenue,
    ROUND(SUM(SUM(sales::numeric)) OVER (
        PARTITION BY territory
        ORDER BY orderdate::date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ), 2) AS running_total
FROM sales
WHERE territory IS NOT NULL
GROUP BY territory, orderdate::date
ORDER BY territory, order_date;

4. Which branch crossed the $100,000 revenue milestone first?
select 
		territory,
		round(sum(sales::numeric), 2) as revenue
from sales
group by 1
order by 2 desc

Q5 — Average revenue per order for each deal size
with order_totals as (
select 
		ordernumber,
		dealsize,
		sum(sales::numeric) as order_revenue
from sales 
group by 1, 2
)

select 
		dealsize,
		count(order_revenue) as total_orders,
		ROUND(AVG(order_revenue), 2) AS avg_revenue_per_order,
		ROUND(MIN(order_revenue), 2) AS min_order_revenue,
		ROUND(MAX(order_revenue), 2) AS max_order_revenue
FROM order_totals
GROUP BY dealsize
ORDER BY avg_revenue_per_order DESC;

Q6 — Which deal size generates more revenue per product line
Q7 — Rank customers by total spend within each territory
Q8 — Percentage of total revenue per deal size
Q9 — Which product line sells most units per order
Q10 — Product lines above and below average revenue
Q11 — Which product line has most distinct customers
Q12 — Month over month revenue change per product line
Q13 — Which month consistently generates highest revenue
Q14 — Which quarter drives most revenue per territory
Q15 — Year over year revenue growth percentage
Q16 — 3 month rolling average of revenue
Q17 — Percentage of cancelled or on hold orders per territory
Q18 — Which product line has highest cancellation rate
Q19 — Which country has highest average order value
Q20 — Top 5 customers by revenue in each territory
21.  Do Member customers rate their experience higher than Normal customers on average — broken down by branch?
