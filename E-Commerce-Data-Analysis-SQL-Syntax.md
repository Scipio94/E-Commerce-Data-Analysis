~~~ SQL
/* Analysis Questions*/

-- 1. What was the total revenue for all completed transactions?
-- 2. What product had the highest revenue?
-- 3. What country had the highest revenue?
-- 4. What month had the highest revenue?
-- 5. What month had the highest number of customers served?

CREATE TEMP TABLE t1 AS
SELECT *,
ROUND(CAST((price * quantity) AS numeric),2) AS Total_Price 
FROM `single-being-353600.E_Commerce_Data.Sales_Data`
WHERE Quantity > 0 ; -- filtering out all cancelled transcations

-- Total Revenue
SELECT 
  ROUND(SUM(t1.total_price),2) AS Total_Revenue
FROM t1;
-- Total Revenue $62,965,974.34

-- Product with the least revenue
SELECT 
  DISTINCT t1.productname,
  ROUND(SUM(t1.total_price),2) AS Product_Revenue
FROM t1
GROUP BY t1.productname
ORDER BY Product_Revenue
LIMIT 5;
-- Set 10 Cards Snowy Robin 17099 and Crochet Lilac/Red Bear Keyring were both tied for the lowest revenue, $6.19

-- Product with the highest revenue
SELECT 
  DISTINCT t1.productname,
  ROUND(SUM(t1.total_price),2) AS Product_Revenue
FROM t1
GROUP BY t1.productname
ORDER BY Product_Revenue DESC
LIMIT 5;
-- Paper Craft Little Birdie was the product with the highest revenue, $100,2718.10

-- Country with the lowest revenue
SELECT
  DISTINCT t1.country,
  ROUND(SUM(t1.total_price),2) AS Country_Revenue
FROM t1
GROUP BY t1.country
ORDER BY Country_Revenue
LIMIT 5;
-- Saudi Arabia was the country with the lowest revenue, $969.5

-- Country with the highest revenue
SELECT
  DISTINCT t1.country,
  ROUND(SUM(t1.total_price),2) AS Country_Revenue
FROM t1
GROUP BY t1.country
ORDER BY Country_Revenue DESC
LIMIT 5;
-- United Kingdom was the country with the highest revenue, $52,524,658.47

-- Product in the United Kingdom with the most revenue
SELECT
  DISTINCT t1.country,
  t1.productname,
  ROUND(SUM(t1.total_price),2) AS UK_Prod_Most_Revenue
FROM t1
WHERE t1.country = 'United Kingdom' -- filteriung for the UK
GROUP BY t1.country, t1.productname
ORDER BY UK_Prod_Most_Revenue DESC
LIMIT 5;
-- Paper Craft Little Birdie was the product that generated the most revenue, $1,002,718.1, in the UK

-- Month with the lowest revenue
SELECT
  DATE_TRUNC(t1.date, month) AS Month, -- Creating month column
  ROUND(SUM(t1.total_price),2) AS Month_Revenue
FROM t1
GROUP BY Month 
ORDER BY Month_Revenue 
LIMIT 5;
-- December 2019 was the month with the lowest revenue

-- Month with the highest revenue
SELECT
  DATE_TRUNC(t1.date, month) AS Month, -- Creating month column
  ROUND(SUM(t1.total_price),2) AS Month_Revenue
FROM t1
GROUP BY Month 
ORDER BY Month_Revenue DESC
LIMIT 5;
-- November 2019 was the month with the highest revenue, 

-- MoM Revenue Difference
WITH month AS 
(SELECT
  CAST(DATE_TRUNC(t1.date, month) AS date) AS month,
  ROUND(SUM(t1.Total_Price),2) AS month_revenue
FROM t1
GROUP BY month
ORDER BY month),

month_lag AS
(SELECT 
  sub.month,
  LAG(sub.Month_Revenue,1) OVER (ORDER BY sub.month) AS month_revenue_lag -- creating lag month_revenue value
FROM
  (SELECT
  CAST(DATE_TRUNC(t1.date, month) AS date) AS month,
  ROUND(SUM(t1.Total_Price),2) AS Month_Revenue
FROM t1
GROUP BY month
ORDER BY month) AS sub
)

SELECT 
  m.month,
  m.month_revenue,
  COALESCE(ml.month_revenue_lag,0) AS month_revenue_lag,
  COALESCE(ROUND(m.month_revenue - ml.month_revenue_lag,2),0) AS MoM_Revenue_Diff,
  COALESCE(ROUND((m.month_revenue - ml.month_revenue_lag)/ m.month_revenue,2),0) AS MoM_Revenue_Diff_Pct
FROM month AS m
LEFT JOIN month_lag AS ml 
USING (month)
ORDER BY month;

-- MoM Revenue Average
WITH month AS 
(SELECT
  CAST(DATE_TRUNC(t1.date, month) AS date) AS month,
  ROUND(SUM(t1.Total_Price),2) AS month_revenue
FROM t1
GROUP BY month
ORDER BY month),

month_lag AS
(SELECT 
  sub.month,
  LAG(sub.Month_Revenue,1) OVER (ORDER BY sub.month) AS month_revenue_lag -- creating lag month_revenue value
FROM
  (SELECT
  CAST(DATE_TRUNC(t1.date, month) AS date) AS month,
  ROUND(SUM(t1.Total_Price),2) AS Month_Revenue
FROM t1
GROUP BY month
ORDER BY month) AS sub
)

SELECT
  DISTINCT ROUND(AVG(sub.MoM_Revenue_Diff) OVER(),1) AS MoM_Revenue_Diff_Avg,
  ROUND(AVG(sub.MoM_Revenue_Diff_Pct) OVER(),1) AS MoM_Revenue_Diff_Pct_Avg
FROM
(SELECT 
  m.month,
  m.month_revenue,
  COALESCE(ml.month_revenue_lag,0) AS month_revenue_lag,
  COALESCE(ROUND(m.month_revenue - ml.month_revenue_lag,2),0) AS MoM_Revenue_Diff,
  COALESCE(ROUND((m.month_revenue - ml.month_revenue_lag)/ m.month_revenue,2),0) AS MoM_Revenue_Diff_Pct
FROM month AS m
LEFT JOIN month_lag AS ml 
USING (month)
ORDER BY month) AS sub;

-- On average there was a decrease of -$146,189.70, 10% per month

-- Month Revenue Distribution
WITH month_metrics AS 
(SELECT
  CAST(DATE_TRUNC(t1.date, month) AS date) AS month,
  ROUND(SUM(t1.Total_Price),2) AS month_revenue
FROM t1
GROUP BY month
ORDER BY month)

SELECT 
  DISTINCT PERCENTILE_CONT(month_metrics.month_revenue,0.25) OVER() AS Month_Revenue_25th,
  PERCENTILE_CONT(month_metrics.month_revenue,0.50) OVER() AS Median,
  PERCENTILE_CONT(month_metrics.month_revenue,0.75) OVER() AS Month_Revenue_75th,
  ROUND(AVG(month_metrics.month_revenue) OVER(),2) AS Average
FROM month_metrics;

-- Monthly Customers Greatest
SELECT
   DATE_TRUNC(t1.date, month) AS month,
   COUNT(t1.customerno) AS Month_Customer_Count
FROM t1
GROUP BY month
ORDER BY Month_Customer_Count DESC
LIMIT 1;
-- November 2019 had the greatest number of customers, 83,042

-- Monthly Customers Least
SELECT
   DATE_TRUNC(t1.date, month) AS month,
   COUNT(t1.customerno) AS Month_Customer_Count
FROM t1
GROUP BY month
ORDER BY Month_Customer_Count
LIMIT 1;
-- December had the least number of customers 25,012

-- Monthly Customers MoM Change
WITH month_customers AS
(SELECT
   DATE_TRUNC(t1.date, month) AS month,
   COUNT(t1.customerno) AS Month_Customer_Count
FROM t1
GROUP BY month
ORDER BY month),
month_customers_lag AS
(SELECT 
   month_customers.month,
   LAG(Month_Customer_Count,1) OVER (ORDER BY month_customers.month) AS Month_Customer_Count_Lag
FROM month_customers
ORDER BY month_customers.month)

SELECT 
   month,
   Month_Customer_Count,
   COALESCE(Month_Customer_Count - Month_Customer_Count_Lag,0) AS MoM_Customer_Diff,
   COALESCE(ROUND((Month_Customer_Count - Month_Customer_Count_Lag)/Month_Customer_Count,2),0) AS MoM_Customer_Diff_Pct

FROM month_customers AS mc
LEFT JOIN month_customers_lag AS mcl
USING(month)
ORDER BY Month_Customer_Count DESC
LIMIT 1;
-- November 2019 was the month the highest number of total customers. 

-- MoM Unique Customer Change
WITH month_customers_unique AS
(SELECT
   DATE_TRUNC(t1.date, month) AS month,
   COUNT(DISTINCT t1.customerno) AS Month_Customer_Unique_Count
FROM t1
GROUP BY month
ORDER BY month),

month_customers_unique_lag AS
(SELECT 
   month_customers_unique.month,
   LAG(Month_Customer_Unique_Count,1) OVER (ORDER BY month_customers_unique.month) AS Month_Customer_Unique_Count_Lag
FROM month_customers_unique)

SELECT 
   month,
   Month_Customer_Unique_Count,
   COALESCE(Month_Customer_Unique_Count - Month_Customer_Unique_Count_Lag,0) AS MoM_Customer_Unique_Diff,
   COALESCE(ROUND((Month_Customer_Unique_Count - Month_Customer_Unique_Count_Lag)/Month_Customer_Unique_Count,2),0) AS MoM_Customer_Un_Diff_Pct
FROM month_customers_unique AS mc
LEFT JOIN month_customers_unique_lag AS mcl
USING(month)
ORDER BY Month_Customer_Unique_Count DESC
LIMIT 1
-- November 2019 has the highest number of unique customers.

~~~
