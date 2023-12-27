~~~ SQL
/*Cancelled Trannactions*/
SELECT 
  COUNT(*) AS Cancelled_Transcations
FROM `single-being-353600.E_Commerce_Data.Sales_Data`
WHERE Quantity < 0; 
-- Negative values are indicative of cancelled transactions
--8,585 Cancelled Transcations

/*Completed Transactions*/
SELECT 
  COUNT(*) AS Completed_Transcations
FROM `single-being-353600.E_Commerce_Data.Sales_Data`
WHERE Quantity > 0; 
-- 527,765 completed transactions

CREATE TEMP TABLE t1 AS
SELECT *,
ROUND(CAST((price * quantity) AS numeric),2) AS Total_Price 
FROM `single-being-353600.E_Commerce_Data.Sales_Data`
WHERE Quantity > 0 ; -- filtering out all cancelled transcations

/*Data Range*/
SELECT 
  MIN(Date) AS Min_Date,
  MAX(Date) AS Max_Date
FROM t1;
-- Date Range: December 1,2018 to December 9,2019 

/*Unit Price Range*/
SELECT 
  MIN(Price) AS Min_Unit_Price,
  MAX(Price) AS Max_Unit_Price
FROM t1;
-- Min price: $5:13
-- Max price: $660.62 

/*Total Price Range*/
SELECT 
  MIN(Total_Price) AS Min_Total_Price,
  MAX(Total_Price) AS Max_Total_Price
FROM t1;
-- Min total price: $5.13
-- Max total price: $1,002,718.1

/*Total Revenue*/
SELECT 
  ROUND(SUM(price * quantity),2) AS Total_Revenue
FROM t1;
-- Total Revenue $62,965,974.34

/*Unique Customers*/
SELECT
  COUNT(DISTINCT CustomerNo) AS Unique_Customers
FROM t1;
-- Unique Customers: 4,719

/* Unique Country Count*/
SELECT
  COUNT(DISTINCT Country) AS Unique_Country_Count
FROM t1;
-- Unique Country Count: 38

/*Unique Product Count*/
SELECT
COUNT(DISTINCT t1.productname) AS Unique_Product_Ct
FROM t1;
-- 3,753 unique products sold in the dataset

/*Total Price Quartiles and Average*/
SELECT 
  DISTINCT PERCENTILE_CONT(t1.Total_Price, 0.25) OVER () AS Total_Price_25,
  PERCENTILE_CONT(t1.Total_Price, 0.50) OVER () AS Total_Price_Median,
  PERCENTILE_CONT(t1.Total_Price, 0.75) OVER () AS Total_Price_75,
  ROUND(AVG(t1.Total_Price) OVER(),2) AS Avg_Total_Price
FROM t1;

--Total Price Metris
-- 25th Percentile: $17.17
-- Median: $43.83
-- 75th: $119.4
-- Average: 1$19.31

/*Distribution*/
SELECT 
  DISTINCT sub.Rounded_Total_Price,
  COUNT(sub.Rounded_Total_Price) AS Cnt
FROM
(SELECT 
  ROUND(t1.Total_Price,-2) AS Rounded_Total_Price
FROM t1) AS sub
GROUP BY sub.Rounded_Total_Price
ORDER BY sub.Rounded_Total_Price
-- Majority of transactions,283,519,were less than $100 

~~~
