# Case Study #3 - Foodie-Fi
Challenge Source: https://8weeksqlchallenge.com/case-study-3/
<br>

### Introduction
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!
<br>

Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!
<br>

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.
<br>
<br>

### Available Data
Danny has shared the data design for Foodie-Fi and also short descriptions on each of the database tables - our case study focuses on only 2 tables but there will be a challenge to create a new table for the Foodie-Fi team.
<br>

All datasets exist within the foodie_fi database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.
<br>

![image](https://github.com/user-attachments/assets/2b1ebd4d-2eae-4def-98b5-940e52465f15)


<br>
<br>


## A. Customer Journey

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.
<br>
Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

```sql
    WITH A AS (SELECT customer_id
                  , plan_name
                  , start_date
                  , price
               	  , CASE
                    WHEN plan_name = 'trial' THEN CONCAT('Started a ', plan_name, ' on ', CAST(start_date AS TEXT) )
                    WHEN plan_name = 'basic monthly' THEN CONCAT('Switched to ', plan_name, ' on ', CAST(start_date AS TEXT) )
                    WHEN plan_name = 'pro monthly' THEN CONCAT('Switched to ', plan_name, ' on ', CAST(start_date AS TEXT) )
                    WHEN plan_name = 'pro annual' THEN CONCAT('Switched to ', plan_name, ' on ', CAST(start_date AS TEXT) )
                    ELSE CONCAT('Cancelled the subscription on ', CAST(start_date AS TEXT) ) 
                    END AS Description
              FROM subscriptions AS S
              LEFT JOIN plans AS P
                  ON S.plan_id = P.plan_id
              WHERE customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)
              ORDER BY customer_id, start_date)
    
    SELECT customer_id
    	, STRING_AGG(description, ', ') AS description
        
    FROM A
    GROUP BY customer_id
    ;
```

|customer_id|description                                                                                                   |
|-----------|--------------------------------------------------------------------------------------------------------------|
|1          |Started a trial on 2020-08-01, Switched to basic monthly on 2020-08-08                                        |
|2          |Started a trial on 2020-09-20, Switched to pro annual on 2020-09-27                                           |
|11         |Started a trial on 2020-11-19, Cancelled the subscription on 2020-11-26                                       |
|13         |Started a trial on 2020-12-15, Switched to basic monthly on 2020-12-22, Switched to pro monthly on 2021-03-29 |
|15         |Started a trial on 2020-03-17, Switched to pro monthly on 2020-03-24, Cancelled the subscription on 2020-04-29|
|16         |Started a trial on 2020-05-31, Switched to basic monthly on 2020-06-07, Switched to pro annual on 2020-10-21  |
|18         |Started a trial on 2020-07-06, Switched to pro monthly on 2020-07-13                                          |
|19         |Started a trial on 2020-06-22, Switched to pro monthly on 2020-06-29, Switched to pro annual on 2020-08-29    |

<br>

## B. Data Analysis Questions

B1. How many customers has Foodie-Fi ever had?

```sql
    SELECT COUNT(DISTINCT customer_id) AS customers_ever_had
    FROM subscriptions
    ;
```
|customers_ever_had|
|------------------|
|1000              |

<br>

B2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```sql
WITH A AS (SELECT customer_id
               , plan_name
                , TO_CHAR(start_date, 'Month') as Month
             	  , EXTRACT(MONTH FROM start_date) AS Month_No
                , price
           FROM subscriptions AS S
           LEFT JOIN plans AS P
               ON S.plan_id = P.plan_id
           ORDER BY customer_id, start_date)  
SELECT month
    	, COUNT(*) no_of_trial_plans
FROM A
WHERE plan_name = 'trial'
GROUP BY month, month_no
ORDER BY month_no
;
```

|month|no_of_trial_plans|
|-----|-----------------|
|January|88               |
|February|68               |
|March|94               |
|April|81               |
|May  |88               |
|June |79               |
|July |89               |
|August|88               |
|September|87               |
|October|79               |
|November|75               |
|December|84               |

<br>

B3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name?
```sql
    WITH A AS (SELECT customer_id
                  , plan_name
                  , start_date
              FROM subscriptions AS S
              LEFT JOIN plans AS P
                  ON S.plan_id = P.plan_id
              WHERE EXTRACT(YEAR FROM start_date) > 2020 AND plan_name != 'churn'
              ORDER BY customer_id, start_date)
    
    SELECT plan_name
    	, COUNT(*) AS count_of_events
    FROM A
    GROUP BY plan_name
    ;
```
|plan_name|count_of_events|
|---------|---------------|
|pro annual|63             |
|pro monthly|60             |
|basic monthly|8              |

<br>

B4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```sql
WITH A AS (SELECT customer_id
                  , SUM(CASE WHEN plan_name = 'churn' THEN 1 ELSE 0 END) AS churn_count
               	  , COUNT(DISTINCT customer_id) AS count
	FROM subscriptions AS S
	LEFT JOIN plans AS P
	ON S.plan_id = P.plan_id
	GROUP BY customer_id)
    
SELECT SUM(churn_count) AS churn_count
    	, ROUND(SUM(churn_count) / SUM(count) * 100, 1) AS churn_percentage
FROM A
;
```
|churn_count|churn_percentage|
|-----------|----------------|
|307        |30.7            |

<br>

B5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```sql
WITH A AS (SELECT ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY customer_id, start_date) AS row_no
           		, CASE
           		  WHEN ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY customer_id, start_date) = 2
           				AND plan_name = 'churn' THEN 1 
           		  ELSE 0 
           		  END AS churn_after_trial_flag
           		, customer_id
                , plan_name
               	, start_date
	        FROM subscriptions AS S
	        LEFT JOIN plans AS P
	          ON S.plan_id = P.plan_id),
    
    B AS (SELECT customer_id
				, SUM(churn_after_trial_flag) AS churn_after_trial_flag
				, COUNT(DISTINCT customer_id) AS count
		 FROM A
		 GROUP BY 1)

SELECT SUM(churn_after_trial_flag) AS churn_after_trial_count
    	, ROUND(SUM(churn_after_trial_flag) / SUM(count) * 100) AS churn_percentage
FROM B
;
```
|churn_after_trial_count|churn_percentage|
|-----------------------|----------------|
|92                     |9               |

<br>

B6. What is the number and percentage of customer plans after their initial free trial?

```sql
WITH A AS (SELECT ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY customer_id, start_date) AS row_no
           		, customer_id
                , plan_name
               	, start_date
	        FROM subscriptions AS S
	        LEFT JOIN plans AS P
	          ON S.plan_id = P.plan_id),
    
     B AS (SELECT plan_name
   				, COUNT (DISTINCT customer_id) AS number_of_customer_plans
   		   FROM A
           WHERE row_no = 2
           GROUP BY plan_name)
           
SELECT plan_name
	, number_of_customer_plans
    , ROUND(
      		number_of_customer_plans / SUM(number_of_customer_plans) OVER() * 100
      		, 1) AS percentage_of_customer_plans
FROM B
;
```
|plan_name|number_of_customer_plans|percentage_of_customer_plans|
|---------|------------------------|----------------------------|
|basic monthly|546                     |54.6                        |
|churn    |92                      |9.2                         |
|pro annual|37                      |3.7                         |
|pro monthly|325                     |32.5                        |

<br>

B7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```sql
/*
Approached the solution based on case study logic
"If customers downgrade from a pro plan or cancel their subscription - the higher plan will
remain in place until the period is over"
"When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher
plan will take effect straightaway."
*/


/*
Identifying the latest subscription using row_number and using LAG window function to bring previous
plan in the same row of latest subscription plan and performing end_date calculation
*/

WITH A AS (SELECT ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY customer_id, start_date DESC) AS row_no
            , customer_id
            , S.plan_id
            , LAG(S.plan_id) OVER (PARTITION BY customer_id ORDER BY  start_date) AS previous_plan_id
            , plan_name
            , LAG(plan_name) OVER (PARTITION BY customer_id ORDER BY  start_date) AS previous_plan_name
            , start_date
            , LAG(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS previous_plan_start_date
            , CAST ((CASE
                     WHEN S.plan_id = 0 THEN start_date + INTERVAL '6 days'
                     WHEN S.plan_id = 1 THEN start_date + INTERVAL '1 month' - INTERVAL '1 day'
                     WHEN S.plan_id = 2 THEN start_date + INTERVAL '1 month' - INTERVAL '1 day'
                     WHEN S.plan_id = 3 THEN start_date + INTERVAL '12 months' - INTERVAL '1 day'
                     ELSE NULL
                     END) AS DATE) AS end_date
          FROM subscriptions AS S
          LEFT JOIN plans AS P
                      ON S.plan_id = P.plan_id
          WHERE start_date <= '2020-12-31' ),
 
 -- Identiying previous_plan end date 
 
     B AS (SELECT *
		, LAG(end_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS previous_plan_end_date
	   FROM A
	   ORDER BY customer_id, row_no),
  
-- Making calculation based on case study logic

     C AS  (SELECT * 
              , CASE
                WHEN plan_id = 0 THEN 1
                WHEN plan_id = 1 THEN 1
                WHEN (plan_id = 2 AND previous_plan_id = 3) AND (previous_plan_end_date < '2020-12-31') THEN 1
                WHEN plan_id = 2 THEN 1
                WHEN (plan_id = 4 AND previous_plan_id IN(0,1,2,3)) AND (previous_plan_end_date < '2020-12-31') THEN 1
                WHEN plan_id = 3 THEN 1
                ELSE 0
                END AS active_plan_flag
          FROM B
          WHERE row_no = 1 -- Selecting only latest subscription plan rows
          ORDER BY customer_id, start_date),

-- replacing the plan name if the case study logic exist

     D AS (SELECT CASE
                 WHEN active_plan_flag = 0 THEN previous_plan_name
                 ELSE plan_name
                 END AS plan_name
                 , (SELECT COUNT(*) FROM C) AS total_customer
          FROM C)

-- Final Calculation

SELECT plan_name
	, COUNT(*) AS no_of_customers
    , ROUND(COUNT(*) / AVG(total_customer) * 100, 1) AS percentage
FROM D
GROUP BY plan_name
ORDER BY percentage DESC
```
|plan_name    |no_of_customers|percentage|
|-------------|---------------|----------|
|pro monthly  |327            |32.7      |
|churn        |234            |23.4      |
|basic monthly|225            |22.5      |
|pro annual   |195            |19.5      |
|trial        |19             |1.9       |

<br>

B8. How many customers have upgraded to an annual plan in 2020?

```sql
/*
Use LAG window function to identify the previous_plan of the customer and applying case fuction to flag
the records upgraded to an annual plan in 2020
*/

WITH A AS	(SELECT * 
              , CASE
                WHEN S.plan_id = 3 AND LAG(S.plan_id) OVER (PARTITION BY customer_id ORDER BY  start_date) < 3 THEN 1
                ELSE 0
                END AS annual_plan_upgrade_flag
          	FROM subscriptions AS S
         	 LEFT JOIN plans AS P
              ON S.plan_id = P.plan_id
         	 WHERE EXTRACT( YEAR FROM start_date) = 2020
          	ORDER BY customer_id, start_date)
            
SELECT COUNT(customer_id) AS customers_upgraded_annual_plan
FROM A
WHERE annual_plan_upgrade_flag = 1
;
```
|customers_upgraded_annual_plan|
|------------------------------|
|195                           |

<br>

B9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

```sql
/*
Approach
Limit the records to customers who started trial and later upgraded to annual plan.
Assigning start date of annual plan in same row of trial using LEAD window function and then calculating the days difference
*/

WITH A AS	(SELECT customer_id
             	, S.plan_id
             	, start_date
             	, MIN(S.plan_id) OVER (PARTITION BY customer_id ORDER BY customer_id) AS min_plan_id --identfying the lowest plan (trial for sure)
				, MAX(S.plan_id) OVER (PARTITION BY customer_id ORDER BY customer_id) AS max_plan_id -- identfying the highest plan

          	FROM subscriptions AS S
         	LEFT JOIN plans AS P
              ON S.plan_id = P.plan_id
         	WHERE S.plan_id != 4 -- removing churn first
			ORDER BY customer_id, start_date
            ),
 
 -- Bringing start date of annual plan in same row of trial and then calculating the days difference
      B AS  (SELECT customer_id
                , plan_id
                , (LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY  start_date) - start_date) AS days    
            FROM A
            WHERE (min_plan_id = 0 AND max_plan_id = 3) AND (plan_id IN (0,3)) -- Limiting to customers with trial and annual plan
             )

SELECT ROUND(AVG(days)) AS days_on_average
FROM B
WHERE plan_id = 0 --Limiting to each customer record where calculation was performed
;
```
|days_on_average|
|---------------|
|105            |

<br> 

B.10 Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```sql
WITH A AS	(SELECT customer_id
             	, S.plan_id
             	, start_date
             	, MIN(S.plan_id) OVER (PARTITION BY customer_id ORDER BY customer_id) AS min_plan_id --identfying the lowest plan (trial for sure)
				, MAX(S.plan_id) OVER (PARTITION BY customer_id ORDER BY customer_id) AS max_plan_id -- identfying the highest plan

          	FROM subscriptions AS S
         	LEFT JOIN plans AS P
              ON S.plan_id = P.plan_id
         	WHERE S.plan_id != 4 -- removing churn first
			ORDER BY customer_id, start_date
            ),
 
 -- Bringing start date of annual plan in same row of trial and then calculating the days difference
      B AS  (SELECT customer_id
                , plan_id
                , (LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY  start_date) - start_date) AS days    
            FROM A
            WHERE (min_plan_id = 0 AND max_plan_id = 3) AND (plan_id IN (0,3)) -- Limiting to customers with trial and annual plan
             ),

	  C	AS	(SELECT  CASE
                     WHEN days <= 30 THEN '0-30 days'
                     WHEN days > 30 AND days <= 60 THEN '30-60 days'
                     WHEN days > 60 AND days <= 90 THEN '60-90 days'
                     WHEN days > 90 AND days <= 120 THEN '90-120 days'
                     WHEN days > 120 AND days <= 150 THEN '120-150 days'
                     WHEN days > 150 AND days <= 180 THEN '150-180 days'
                     WHEN days > 180 AND days <= 210 THEN '180-210 days'
                     WHEN days > 210 AND days <= 240 THEN '210-240 days'
                     WHEN days > 240 AND days <= 270 THEN '240-270 days'
                     WHEN days > 270 AND days <= 300 THEN '270-300 days'
                     ELSE '>300 days'
                     END AS period
                     , days
              FROM B
              WHERE plan_id = 0 --Limiting to each customer record where calculation was performed
             )

SELECT period
	, ROUND(AVG(days)) AS days_on_average
FROM C
GROUP BY period
;
```
|period      |days_on_average|
|------------|---------------|
|0-30 days   |10             |
|120-150 days|133            |
|150-180 days|162            |
|180-210 days|191            |
|210-240 days|224            |
|240-270 days|257            |
|270-300 days|285            |
|30-60 days  |42             |
|60-90 days  |71             |
|90-120 days |101            |
|>300 days   |337            |

<br>

B11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

```sql
/*
Approach
Limit the records to customers who had instances of basic and pro monthly plans
Identify customers with basic monthly plan who has pro monthly as previous plan
*/

WITH A AS	(SELECT customer_id
             	, S.plan_id
             	, start_date
          	FROM subscriptions AS S
         	LEFT JOIN plans AS P
              ON S.plan_id = P.plan_id
         	WHERE S.plan_id NOT IN (0, 3, 4) -- removing trial, annual and churn
             	  AND EXTRACT(YEAR FROM start_date) = 2020 
			ORDER BY customer_id, start_date
            ),

	 B AS 	(SELECT *
				, LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS previous_plan_id 
			FROM A
             )
SELECT *
FROM B 
WHERE previous_plan_id = 2
;
```
|      |
|------------|
|There are no records where customers downgraded from a pro monthly to a basic monthly plan in 2020|

<br>

## C. Challenge Payment Question
The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

```sql
/*
Approach
First, using LEAD and LAG windows functions to bring start date of the next subscription plan for every record.
Then, generating monthly payment schedule for basic and pro monthly plan, generating annual payment plans and a record for the churns.
Then, removing records that overlaps with the help of next subscription plan start_date
Finally, applying calculationsm based on the cast study question logic
*/


-- Joining the subscriptions and plans table and removing records of trial
-- LEAD LAG function to determine previous and next plan information
WITH A AS 	(SELECT ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS row_number
             	, customer_id
             	, S.plan_id
             	, P.plan_name
             	, start_date
             	, LAG(S.plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS previous_plan_id
             	, LEAD(S.plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_plan_id
             	, COALESCE(LEAD(S.start_date) OVER (PARTITION BY customer_id ORDER BY start_date),
			   '2020-12-31') AS next_plan_start_date
             	, price
             	, LAG(P.price) OVER (PARTITION BY customer_id ORDER BY start_date) AS previous_plan_price
                , CAST('2020-12-31' AS DATE) AS end_date
          	FROM subscriptions AS S
         	LEFT JOIN plans AS P
              ON S.plan_id = P.plan_id
         	WHERE S.plan_id NOT IN (0) -- removing trial
             	  AND EXTRACT(YEAR FROM start_date) = 2020
			ORDER BY customer_id, start_date
            ),

-- Creating monthly payment schedule for subscription who started with plan 1 or 2 (monthly)
	B AS 	(SELECT *
            	FROM A,
            	GENERATE_SERIES(start_date, end_date, INTERVAL '1 month') AS p_date
            	WHERE  plan_id IN (1,2) --AND customer_id = 193
            	ORDER BY customer_id, plan_id, p_date
             	),

-- Creating annual payment schedule for subscription who started with plan 3 (annual)
	C AS 	(SELECT *
            	FROM A,
            	GENERATE_SERIES(start_date, end_date, INTERVAL '1 year') AS p_date
            	WHERE  plan_id IN (3) --AND customer_id = 193
            	ORDER BY customer_id, plan_id, p_date
             	),

-- Combining  Table A and C
	D AS	(SELECT * FROM B UNION ALL
             	SELECT * FROM C
            	) 
            
--Final Calculation             
SELECT customer_id
	, plan_id
    	, plan_name
    	, CAST(p_date AS DATE) AS payment_date
    	, CASE
     	  WHEN plan_id = 3 AND previous_plan_id = 1 THEN (price - previous_plan_price)
      	  ELSE price
     	  END AS amount
    	, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY p_date) AS payment_order
FROM D
WHERE customer_id IN (1, 2, 13 ,15, 16, 18, 19)
      AND -- Limiting to customer_id as shown in the desired output
      p_date < next_plan_start_date -- removing records that overlaps based on case study logic
;
```
|customer_id |plan_id|plan_name    |payment_date|amount|payment_order|
|------------|-------|-------------|------------|------|-------------|
|1           |1      |basic monthly|2020-08-08  |9.90  |1            |
|1           |1      |basic monthly|2020-09-08  |9.90  |2            |
|1           |1      |basic monthly|2020-10-08  |9.90  |3            |
|1           |1      |basic monthly|2020-11-08  |9.90  |4            |
|1           |1      |basic monthly|2020-12-08  |9.90  |5            |
|2           |3      |pro annual   |2020-09-27  |199.00|1            |
|13          |1      |basic monthly|2020-12-22  |9.90  |1            |
|15          |2      |pro monthly  |2020-03-24  |19.90 |1            |
|15          |2      |pro monthly  |2020-04-24  |19.90 |2            |
|16          |1      |basic monthly|2020-06-07  |9.90  |1            |
|16          |1      |basic monthly|2020-07-07  |9.90  |2            |
|16          |1      |basic monthly|2020-08-07  |9.90  |3            |
|16          |1      |basic monthly|2020-09-07  |9.90  |4            |
|16          |1      |basic monthly|2020-10-07  |9.90  |5            |
|16          |3      |pro annual   |2020-10-21  |189.10|6            |
|18          |2      |pro monthly  |2020-07-13  |19.90 |1            |
|18          |2      |pro monthly  |2020-08-13  |19.90 |2            |
|18          |2      |pro monthly  |2020-09-13  |19.90 |3            |
|18          |2      |pro monthly  |2020-10-13  |19.90 |4            |
|18          |2      |pro monthly  |2020-11-13  |19.90 |5            |
|18          |2      |pro monthly  |2020-12-13  |19.90 |6            |
|19          |2      |pro monthly  |2020-06-29  |19.90 |1            |
|19          |2      |pro monthly  |2020-07-29  |19.90 |2            |
|19          |3      |pro annual   |2020-08-29  |199.00|3            |




























































