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

6. What is the average revenue per transaction for each customer type (Member vs Normal)?
   
7. Which customer type generates more revenue per product line — and is the pattern consistent across branches?
   
8.  Rank customers by total spend within each branch using a window function (RANK() OVER PARTITION BY)
9.  What percentage of total revenue comes from Member customers vs Normal customers?
10.  Which product line sells the most units on average per transaction?
11.  Identify product lines performing above and below the store average in revenue (CASE WHEN + subquery)
12.   Which product line has the highest return visit rate — meaning customers buy it across multiple visits?
13.  What is the month-over-month revenue change per product line?
14.  Which time of day (morning, afternoon, evening) generates the most revenue per branch?
15.  Which day of the week consistently has the highest average transaction value?
16.  Is there a significant revenue difference between weekdays and weekends?
17.  What is the 4-week rolling average of daily revenue across all branches? (rolling window)
18.  Is there a correlation between customer rating and gross income per transaction?
19.  Which product line receives the highest average rating — and does it also generate the most revenue?
20.  Which branch has the most consistently high ratings — lowest standard deviation in ratings? (STDDEV)
21.  Do Member customers rate their experience higher than Normal customers on average — broken down by branch?
