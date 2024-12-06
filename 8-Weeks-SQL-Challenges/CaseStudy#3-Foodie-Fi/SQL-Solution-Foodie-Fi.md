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

B.10


























































