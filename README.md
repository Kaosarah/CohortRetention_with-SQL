# Cohort Retention Analysis (using SQL and Tableau)

## Introduction
This is a transnational data set of transactions occurring between 01/12/2010 and 09/12/2011 for a UK-based and registered non-store online retail.The company mainly sells unique all-occasion gifts and many of their customers are wholesalers.

## Data source 
This data set was gotten from UCI Machine Learning Repository, access through the link [here](
https://archive.ics.uci.edu/ml/machine-learning-databases/00352/).

The data has 7 columns and 541909 rows.

## Business Objective
The company is interested in knowing the revenue generated from 01/12/2010 and 09/12/2011 and their best customers, the organization also what to know if they have been able to retain most of their customers or lost them in order to improve their marketing strategies.

## Data Cleaning
The data was cleaned using SQL

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
 
----Top (10) customers
  SELECT TOP 10 [CustomerID], ROUND(SUM ([Quantity]* [UnitPrice]),0) AS Sales
  FROM #Cleandata
  GROUP BY [CustomerID]
  ORDER BY Sales DESC 
  
  --- Sales by country
  
   SELECT [Country], ROUND(SUM ([Quantity]* [UnitPrice]),0) AS Sales
  FROM #Cleandata
  GROUP BY [Country]
  ORDER BY Sales DESC
  
 --- Getting the first day of purchase and months of purchase of each customer
 
  SELECT
   DISTINCT [CustomerID],
   MIN(InvoiceDate) AS FirstPurchase,---first day of purchase
   DATEFROMPARTS(YEAR(
   MIN(InvoiceDate) 
   ), MONTH(MIN(InvoiceDate)), 1) AS PurchaseMonth --- month of purchase of each customer
   INTO #Cohort
   FROM #Cleandata
   GROUP BY [CustomerID]

   SELECT *
   FROM #Cohort;
   
 --- Getting the Cohort Index
 
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

  ----Pivot cohort table 
  
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

## Cohort retention analysis result 

![Screenshot (110)](https://user-images.githubusercontent.com/109418747/187843368-10c8a48f-b6ab-468d-ba06-9d99f19dfb9e.png)


![Screenshot (121)](https://user-images.githubusercontent.com/109418747/187861423-f20ac9da-e9dd-41a4-ae04-83b4359b16ac.png)


![Screenshot (120)](https://user-images.githubusercontent.com/109418747/187861138-c70f26d9-2cdc-4180-a577-d7bc213a5fa1.png)
  
  
![Screenshot (114)](https://user-images.githubusercontent.com/109418747/187849882-bf34470b-6b37-437f-8afb-6e80f2a16d7d.png)



![Screenshot (115)](https://user-images.githubusercontent.com/109418747/187849920-2ac467f8-41f0-46ab-a244-734df7cbde1d.png)


## Findings and Recommendations

**$8,878,811** was generated from 01/12/2010 and 09/12/2011, the company had their best sales in the month of November,2011 and lowest sales in February,2011. There has been a positive trend in sales over time.

The company make more sales from United Kingdom and least sales from Saudi Arabia.

Out of all of the new customers during this time range 01/12/2010-09/12/2011 (4,338 users), **22.5%** users are retained on the first month, 21.72% on the second month, 21.37%, as at the 9th month 11.73% was retained amounting to half of customers that were retained in the first month and at the business's last transaction month **5.42%** of the customers where retained. This indicate there is a high churn rate.

## Recommendations
The company needs to communicate with customers to know how they can improve their services and ask the churned customers why they left through various communication channels. Much attention to their complains and comments will help determine why customer churns and how to resolve it.

All customers should be educated about the products.

Valuable customers should be separated from the rest and go an extra mile to make sure that at least they are getting what they have signed up for likewise customers that are most likely to churn.

The marketing strategies should be review and improve putting into consideration what the customers complained about and also countries with low sales like Saudi Arabia, Brazil, Czech republic and others. The country should also consider giving discount on goods bought.
 
**Link to interactive pages [here](
https://public.tableau.com/app/profile/lawal.kaosarah/viz/Cohort_retention/COHORTRETENTION#1)**.



