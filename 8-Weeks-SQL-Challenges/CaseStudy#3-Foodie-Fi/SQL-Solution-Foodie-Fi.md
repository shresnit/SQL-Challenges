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





























































