# Cohort Retention Analysis (using SQL and Tableau)

## Introduction
This is a transnational data set of transactions occurring between 01/12/2010 and 09/12/2011 for a UK-based and registered non-store online retail.The company mainly sells unique all-occasion gifts and many of their customers are wholesalers.

## Data source 
This data set was gotten from UCI Machine Learning Repository, access through the link below;
https://archive.ics.uci.edu/ml/machine-learning-databases/00352/
The data has 7 columns and 541909 rows.

## Business Objective
The company is interested in knowing the revenue generated from 01/12/2010 and 09/12/2011 and their best customers, The organization also what to know if they have been able to retain most of their customers or lost them in order to improve their marketing strategies.

## Data Cleaning
The data was using SQL, 
```

---All transaction(541,909)
SELECT *
FROM [Kaggle].['Online Retail$']

---Transaction with CustomerID(406,829)
SELECT *
FROM [Kaggle].['Online Retail$']
WHERE [CustomerID] IS NOT NULL;

--Transaction excluding returned items(397,884)
WITH retails AS( SELECT *
FROM [Kaggle].['Online Retail$']
WHERE [CustomerID] IS NOT NULL
AND [Quantity] > 0
AND [UnitPrice] >0),

---Duplicate check

Duplicate AS (
SELECT *,
ROW_NUMBER() OVER(PARTITION BY [InvoiceNo],[StockCode],[Quantity] ORDER BY[InvoiceDate] ) dupNo
FROM retails
)
SELECT *
INTO #Cleandata
FROM Duplicate
where dupNo =1;

 SELECT * 
 FROM #Cleandata
 ```
 
 ## Data Analysis using SQL and Tableau
 
 ```
 ---- Revenue generated from 01/12/2010 and 09/12/2011
 SELECT SUM ([Quantity]* [UnitPrice])
 FROM #Cleandata

 --Cohort
  
  SELECT
   DISTINCT [CustomerID],
   MIN(InvoiceDate) AS FirstPurchase,
   DATEFROMPARTS(YEAR(
   MIN(InvoiceDate) 
   ), MONTH(MIN(InvoiceDate)), 1) AS PurchaseMonth
   INTO #Cohort
   FROM #Cleandata
   GROUP BY [CustomerID]

   SELECT *
   FROM #Cohort;

  -- Cohort Analysis
 WITH analysis AS ( SELECT 
  cd.[InvoiceNo],
  cd.[StockCode], 
  cd.[Quantity],
  cd.[InvoiceDate],
  cd.[UnitPrice],
  cd.[Country],
  cd.[CustomerID],
  c.PurchaseMonth,
  c.FirstPurchase,
  YEAR(cd.[InvoiceDate])- YEAR( FirstPurchase) AS Year_dif,
  MONTH(cd.[InvoiceDate]) - MONTH(c.FirstPurchase) AS Month_dif
  FROM #Cleandata cd
  LEFT JOIN  #Cohort c
  ON cd.CustomerID = c.CustomerID),
  Cohort_Index AS (
  SELECT *, 
   Year_dif*12 + Month_dif + 1 AS Cohort_index
  FROM analysis)
  SELECT *
  into #Cohort_table
  FROM Cohort_Index
  --Cohort_table with cohort_index
  SELECT *
  FROM #Cohort_table

  --Total Number of Cohort index(13)
  SELECT DISTINCT Cohort_index
  FROM #Cohort_table
  ORDER BY 1 ;

  ----Pivot
  SELECT *
  into #cohort_analys
  from(
  SELECT DISTINCT 
      [PurchaseMonth],
       CustomerID,
       Cohort_index
  FROM #Cohort_table )AS CT
   PIVOT(
   COUNT ([CustomerID])
   FOR Cohort_index IN (
   [1],
   [2],
   [3],
   [4],
   [5],
   [6],
   [7],
   [8],
   [9],
   [10],
   [11],
   [12],
   [13])
   ) AS Pivotcohort_table
   ORDER BY PurchaseMonth
   

   ---Cohort retention result
   SELECT *
   FROM #cohort_analys
   ORDER BY PurchaseMonth
   
```

  

