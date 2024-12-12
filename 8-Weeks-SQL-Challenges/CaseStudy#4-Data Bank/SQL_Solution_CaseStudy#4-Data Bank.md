# Case Study #4 - Data Bank
![4 (Custom)](https://github.com/user-attachments/assets/72c327cf-6485-4ab1-aa37-9710759fc0b0)
<br>
Challenge Source: https://8weeksqlchallenge.com/case-study-4/
<br>

### Introduction
There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.
<br>

Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!
<br>

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!
<br>

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!
<br>

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will nee
<br>

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!
<br>
<br>

### Available Data
The Data Bank team have prepared a data model for this case study as well as a few example rows from the complete dataset below to get you familiar with their tables.
<br>

![image](https://github.com/user-attachments/assets/941c328d-6816-4da5-ba59-c00d4f375d00)

<br>

## A. Customer Nodes Exploration

#### A1. How many unique nodes are there on the Data Bank system?

```sql
SELECT COUNT (DISTINCT node_id) AS no_unique_nodes
FROM customer_nodes
```
|no_unique_nodes|
|---------------|
|5              |

<br>

#### A2.  What is the number of nodes per region?

```sql
SELECT region_name
	, COUNT(DISTINCT node_id) AS number_of_nodes
FROM regions AS R
INNER JOIN customer_nodes AS N
	ON R.region_id = N.region_id
GROUP BY region_name
;
```
|region_name |number_of_nodes|
|------------|---------------|
|Africa      |5              |
|America     |5              |
|Asia        |5              |
|Australia   |5              |
|Europe      |5              |

<br>

#### A3. How many customers are allocated to each region?
```sql
SELECT region_name
	, COUNT(DISTINCT customer_id) AS number_of_customers
FROM regions AS R
INNER JOIN customer_nodes AS N
	ON R.region_id = N.region_id
GROUP BY region_name
;
```
|region_name |number_of_customers|
|------------|-------------------|
|Africa      |102                |
|America     |105                |
|Asia        |95                 |
|Australia   |110                |
|Europe      |88                 |

<br>

#### A4. How many days on average are customers reallocated to a different node?

```sql
SELECT  ROUND(AVG((end_date - start_date)+1)) AS average_days
FROM customer_nodes AS N
WHERE end_date != '9999-12-31'
;
```
|average_days|
|------------|
|16          |

<br>

#### A5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

```sql
WITH A AS	(SELECT region_name
                , (end_date - start_date)+1 AS average_days
                , NTILE(100) OVER  (PARTITION BY region_name ORDER BY (end_date - start_date)+1) AS percentile
            FROM customer_nodes AS N
            INNER JOIN regions AS R
                ON R.region_id = N.region_id
            WHERE end_date != '9999-12-31'
            ),

	 B AS	(SELECT region_name
    			, percentile
				, ROUND(AVG(average_days)) AS value
            FROM A
            WHERE percentile IN (50, 80, 95)
            GROUP BY region_name, percentile
             )

SELECT region_name
	, SUM (CASE WHEN percentile = 50 THEN value END) AS Median
    , SUM (CASE WHEN percentile = 80 THEN value END) AS "80thPercentile"
    , SUM (CASE WHEN percentile = 95 THEN value END) AS "95thPercentile"
FROM B
GROUP BY region_name
;
```
|region_name|median|80thPercentile|95thPercentile|
|-----------|------|--------------|--------------|
|Africa     |16    |25            |29            |
|America    |17    |24            |29            |
|Asia       |16    |24            |29            |
|Australia  |17    |25            |29            |
|Europe     |17    |25            |29            |

<br>

## B. Customer Transactions

#### B1. What is the unique count and total amount for each transaction type?
```sql
SELECT txn_type
	, COUNT(DISTINCT txn_type) AS unique_count
    , SUM(txn_amount) AS total_amount
FROM customer_transactions
GROUP BY txn_type
;
```
|txn_type |unique_count|total_amount|
|---------|------------|------------|
|deposit  |1           |1359168     |
|purchase |1           |806537      |
|withdrawal|1          |793003      |

<br>

#### B2. What is the average total historical deposit counts and amounts for all customers?
```sql
WITH A AS	(SELECT customer_id
				, COUNT(*) AS deposit_counts
				, SUM(txn_amount) AS amounts
			FROM customer_transactions
			WHERE txn_type = 'deposit'
			GROUP BY customer_id
        	)
        
SELECT ROUND(AVG(deposit_counts)) AS historical_deposit_counts_avg
	, SUM(amounts) AS total_amount
FROM A
;
```
|historical_deposit_counts_avg|total_amount|
|-----------------------------|------------|
|5                            |1359168     |

<br>

#### B3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
WITH A AS	(SELECT customer_id
             	, EXTRACT(MONTH from txn_date) as month
             	, SUM((CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END)) AS deposits
             	, SUM((CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END)) AS purchase
             	, SUM((CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END)) AS withdrawal
			FROM customer_transactions
            GROUP BY customer_id, month
            ORDER BY customer_id
        	)
        
SELECT COUNT(DISTINCT customer_id) AS number_of_customers
FROM A
WHERE deposits > 1 AND (purchase >= 1 OR withdrawal >= 1)
;
```
|number_of_customers|
|-------------------|
|347                |

<br>

#### B4. What is the closing balance for each customer at the end of the month?
```sql
/*
Assigning negative txt_amount for withdraw and purchase and then performing running total to calculate available balance
as per the transaction date.
Then identifying end_of_month flag for each customer
*/

WITH 	A AS	(SELECT customer_id
                 	, txn_date
                 	, txn_type
                 	, txn_amount
                	, SUM (CASE 
                 	  	   WHEN txn_type = 'deposit' THEN txn_amount
                 	  	   ELSE -(txn_amount)
                 	  	   END)
                 	  OVER (PARTITION BY customer_id ORDER BY txn_date) AS available_balance
                 	, TO_CHAR(txn_date, 'Month') AS month_name
                    , ROW_NUMBER()
                 	  OVER (PARTITION BY customer_id
                                     	, EXTRACT(MONTH FROM txn_date) 
                            ORDER BY txn_date DESC) AS end_of_month_flag
                 FROM customer_transactions
                 )
        
SELECT customer_id
	, month_name AS month
    , available_balance AS endofmonth_closing_balance
FROM A
WHERE end_of_month_flag = 1
LIMIT 20 -- Limiting to first 20 rows in the output for display purpose
;
```
|customer_id|month   |endofmonth_closing_balance|
|-----------|--------|--------------------------|
|1          |January |312                       |
|1          |March   |-640                      |
|2          |January |549                       |
|2          |March   |610                       |
|3          |January |144                       |
|3          |February|-821                      |
|3          |March   |-1222                     |
|3          |April   |-729                      |
|4          |January |848                       |
|4          |March   |655                       |
|5          |January |954                       |
|5          |March   |-1923                     |
|5          |April   |-2413                     |
|6          |January |733                       |
|6          |February|-52                       |
|6          |March   |340                       |
|7          |January |964                       |
|7          |February|3173                      |
|7          |March   |2533                      |
|7          |April   |2623                      |

<br>

#### B5. What is the percentage of customers who increase their closing balance by more than 5%?

```sql
/*
As the question is not clear I am assuming that comparasion is between latest month to previous month closing balance.
*/

-- Creating a table with negative and positive amount as per the txn type
-- Then Using ROW_NUMBER function to flag the end of the month transaction for each customer
-- Using SUM window function to create running total for closing balance after each transaction

WITH 	A AS  (SELECT customer_id
		 , txn_date
		 , txn_type
		 , CAST(txn_amount AS FLOAT) AS txt_amount
		 , SUM (CASE 
			WHEN txn_type = 'deposit' THEN txn_amount
			ELSE -(txn_amount)
			END)
		   OVER (PARTITION BY customer_id ORDER BY txn_date) AS available_balance
		 , TO_CHAR(txn_date, 'Month') AS month_name
		 , EXTRACT(MONTH FROM txn_date) AS month_number
	 	 , ROW_NUMBER()
		   OVER (PARTITION BY customer_id , EXTRACT(MONTH FROM txn_date) 
 			 ORDER BY txn_date DESC) AS end_of_month_flag
                 FROM customer_transactions
                 ),

-- Filtering to only end of month balance and using Row Number to rank latest month to previous months balances

	B AS (SELECT customer_id
		, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY txn_date DESC) AS month_order
		, month_name AS month
		, available_balance AS closing_balance
              FROM A
              WHERE end_of_month_flag = 1
              ), 

--Including only latest month and previous month balances
--Using LEAD window function to bring previous month closing balance to latest month row for calculation purpose

	C AS (SELECT *
               , LEAD(closing_balance) OVER (PARTITION BY  customer_id ORDER BY month_order) AS previous_month_closing_balance
              FROM B
              WHERE month_order IN (1,2)
              ),


-- Calculating percent difference based on to latest month closing balance and previous month closing balance

	D As (SELECT *
		, ROUND((closing_balance - previous_month_closing_balance) 
		/ (CASE WHEN previous_month_closing_balance = 0 THEN NULL ELSE previous_month_closing_balance END)::NUMERIC  
		* 100, 2) AS percent_difference
              FROM C
              )
              
--Determing the customers with more than 5% closing balance increase and making calculation

SELECT  ROUND(SUM(CASE WHEN percent_difference > 5 THEN 1 ELSE 0 END)
		/ COUNT(*)::NUMERIC
        * 100, 1) AS percent_of_customers
FROM D
WHERE month_order = 1
;
```
|percent_of_customers|
|--------------------|
|46.4                |

<br>

## C. Data Allocation Challenge

To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:
<br>
<br>
Option 1: data is allocated based off the amount of money at the end of the previous month
<br>
Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
<br>
Option 3: data is updated real-time
<br>
<br>
For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:
<br>
- running customer balance column that includes the impact each transaction
- customer balance at the end of each month
- minimum, average and maximum values of the running balance for each customer
<br>
Using all of the data available - how much data would have been required for each option on a monthly basis?
<br>
<br>
#### Option 1: If Customer A has $100 at the end of the previous month then 100 GB of cloud storage will be allocated to the customer for the current month. 

```sql

/*
Approach
Since some customers had 1 or 2 transactions over the time period, we need balance amount for
each month for every customers.
Here I am using concept of data scaffolding to create these month rows for every customer and
making the calculation.

*/

--Identifying minimum and maximum transaction date in the data and trunc them in month

WITH A AS (SELECT DATE_TRUNC('month', MIN(txn_date))::DATE AS min_date
				, DATE_TRUNC('month', MAX(txn_date))::DATE AS max_date
			FROM customer_transactions
          ),

--Generating month level rows from the min and max date and assigning them to each customers using cross join

	 B AS (SELECT customer_id
				, GENERATE_SERIES(min_date, max_date, INTERVAL '1 month')::DATE  AS month_date
		  FROM A
		  CROSS JOIN (SELECT DISTINCT customer_id FROM customer_transactions) AS SQ
		  ORDER BY customer_id, month_date
          ),


-- Calculating closing balance for each month and every customers

	 C AS (SELECT customer_id
                 , DATE_TRUNC('month', txn_date)::DATE AS month_date
                 , SUM (CASE 
                 	  	WHEN txn_type = 'deposit' THEN txn_amount
                 	  	ELSE -(txn_amount)
                 	  	END) AS balance
          FROM customer_transactions
          GROUP BY customer_id, month_date
          ),
          

-- Assigning the balance of a specific month to the newly generate table B.
-- if there are no transaction in a particular month,  NULL is assigned as we are using left join
-- As a result we have every month level balance for every customers

	 D AS (SELECT B.customer_id
				, B.month_date
    			, balance
		   FROM B
		   LEFT JOIN C
			ON B.customer_id = C.customer_id
    		AND B.month_date = C.month_date
           ),

-- Now, using SUM window function for running total of balance.
-- As a result customer having a NULL in a particular month is assigned balance from previous month

	E AS (SELECT customer_id
				, month_date 
    			, balance
				, SUM(balance) OVER (PARTITION BY customer_id ORDER BY month_date) AS closing_balance
		 FROM D
          ),

-- As the data allocation for current month is based on previous month balance, 
-- we are using LAG window function bring the value to current row from previous row
-- As nothing is allocated for balace with negative value replacing them with 0

	F AS (SELECT *
       			, CASE
          		  WHEN COALESCE(LAG(closing_balance) OVER (PARTITION BY customer_id ORDER BY month_date), 0)
				< 0 THEN 0
          		  ELSE COALESCE(LAG(closing_balance) OVER (PARTITION BY customer_id ORDER BY month_date), 0)
          		  END AS data_allocation       
		 FROM E
         )


--Finally, summing up all the data allocation numbers of all customers at month level

SELECT month_date
	, SUM(data_allocation) AS "data_allocation (GB)"
FROM F
GROUP BY month_date
ORDER BY month_date
;
```
|month_date|data_allocation (GB)|
|----------|--------------------|
|2020-01-01|0                   |
|2020-02-01|235595              |
|2020-03-01|261508              |
|2020-04-01|260971              |

<br>
#### Option 2: If Customer A has $100 on an average in previous 30 days then 100 GB of cloud storage will be allocated to the customer for the current day. 

```sql
/*
Approach
Data scaffolding at day level. Assigning running total balance for each customer at each day from min and max
date of the dataset. This will allow to figure what was the balance every day before 30 days from start of month
*/

--Identifying minimum and maximum transaction date in the data and trunc them in days

WITH 	 A AS (SELECT DATE_TRUNC('month', MIN(txn_date))::DATE AS min_date
				, MAX(txn_date)  AS max_date
	       FROM customer_transactions
          ),

--Generating day level rows from the min and max date and assigning them to each customers using cross join

	 B AS (SELECT customer_id
				, GENERATE_SERIES(min_date, max_date, INTERVAL '1 day')::DATE  AS day_date
	      FROM A
	      CROSS JOIN (SELECT DISTINCT customer_id FROM customer_transactions) AS SQ
	      ORDER BY customer_id, day_date
	      ),

 -- Calculating closing balance for every customers at day level. This also includes summarizing transaction at day.

	 C AS (SELECT customer_id
           		 , txn_date
                 , SUM (CASE 
                 	  	WHEN txn_type = 'deposit' THEN txn_amount
                 	  	ELSE -(txn_amount)
                 	  	END) AS balance
 	       FROM customer_transactions
               GROUP BY customer_id, txn_date
              ),
 

-- Assigning the transactions to  to the newly generate table B. So every customers have a balance for a every day
-- if there are no trasaction in a particular day then 0 is assigned
-- As a result we have every day level balance for every customers

	 D AS (SELECT B.customer_id
               , B.day_date
           	   , DATE_TRUNC('month', B.day_date)::DATE + INTERVAL '1 month' AS start_of_month 
           	   , COALESCE(balance, 0) AS balance -- Assigning 0 if null
              FROM B
              LEFT JOIN C
              ON B.customer_id = C.customer_id
              AND B.day_date = C.txn_date
              ),

-- Using SUM window function for running total of balance.
-- As a result we have running total of balance for each customer at day level from start to enddate of the dataset
	E AS (SELECT customer_id
			, day_date
          		, start_of_month::DATE AS start_of_month
          		, (start_of_month - INTERVAL '30 days' )::DATE AS previous_30_days_date
			, SUM(balance) OVER (PARTITION BY customer_id ORDER BY day_date) AS closing_balance
	      FROM D
 	      ),

-- Now, lets allocate the data based on following logic
-- "data is allocated on the average amount of money kept in the account in the previous 30 days"
-- Let's considers day 1 of month AS current date and look 30 days back 

	F AS (SELECT customer_id
          		, start_of_month
			, ROUND(AVG(closing_balance)) AS average_balance_prev_30days
	      FROM E
	      WHERE (day_date < start_of_month AND day_date >= previous_30_days_date::DATE)
	      GROUP BY customer_id, start_of_month
  	     )

--Allocating data to the customers and summarizing at start of month level

SELECT start_of_month
	, SUM(CASE
      	  WHEN average_balance_prev_30days < 0 THEN 0
      	  ELSE average_balance_prev_30days
	  END) AS "data_allocation (GB)"
FROM F
GROUP BY start_of_month
;
```
|start_of_month|data_allocation (GB)|
|--------------|--------------------|
|2020-04-01    |257336              |
|2020-03-01    |245923              |
|2020-05-01    |256593              |
|2020-02-01    |128447              |
<br>
#### Option 3: If the cloud data storage is updated dynamically in real-time based on the customer’s current account balance 

```sql


```




























































