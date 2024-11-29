# Case Study #2 - Pizza Runner
Challenge Source: https://8weeksqlchallenge.com/case-study-2/
<br>

### Introduction
Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”
<br>

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!
<br>

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.
<br>

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.
<br>
<br>

### Available Data
Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.
<br>

He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.
<br>

All datasets exist within the pizza_runner database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.
<br>

![image](https://github.com/user-attachments/assets/090d81e5-0997-4cbc-ad50-b26483e27202)

<br>
<br>


## A. Pizza  Metrics

#### Q1. How many pizzas were ordered?

    SELECT COUNT(*) AS no_of_pizzas_ordered
    FROM customer_orders
    ;
|no_of_pizzas_ordered|
|--------------------|
|14                  |

<br>

#### Q2. How many unique customer orders were made?

    SELECT COUNT(DISTINCT order_id) AS unique_customer_orders
    FROM customer_orders
    ;
|unique_customer_orders|
|----------------------|
|10                    |

<br>

#### Q3. How many successful orders were delivered by each runner?

    SELECT runner_id
    	, COUNT(*) AS number_of_orders_delivered
    FROM runner_orders
    WHERE pickup_time IS NOT NULL
    GROUP BY runner_id
    ORDER BY runner_id
    ;

|runner_id|number_of_orders_delivered|
|---------|--------------------------|
|1        |4                         |
|2        |4                         |
|3        |2                         |

<br>

#### Q3. How many successful orders were delivered by each runner?

    SELECT runner_id
    	, COUNT(*) AS number_of_orders_delivered
    FROM runner_orders
    WHERE pickup_time != 'null'
    GROUP BY runner_id
    ORDER BY runner_id
    ;

|runner_id|number_of_orders_delivered|
|---------|--------------------------|
|1        |4                         |
|2        |3                         |
|3        |1                         |

<br>

#### Q4. How many of each type of pizza was delivered?

    SELECT A. pizza_id
    	, C.pizza_name
       , COUNT(A. order_id)
    FROM customer_orders AS A
    INNER JOIN runner_orders AS B
    	ON A.order_id = B.order_id
    INNER JOIN pizza_names AS C
    	ON A.pizza_id = C.pizza_id
    WHERE B.pickup_time != 'null'
    GROUP BY A.pizza_id, C.pizza_name
    ;

|pizza_id|pizza_name|count|
|--------|----------|-----|
|1       |Meatlovers|9    |
|2       |Vegetarian|3    |

<br>

#### Q5. How many Vegetarian and Meatlovers were ordered by each customer?

    SELECT A.customer_id
    	, C.pizza_name
       	, COUNT(A. order_id) AS no_of_pizzas
    FROM customer_orders AS A
    INNER JOIN runner_orders AS B
    	ON A.order_id = B.order_id
    INNER JOIN pizza_names AS C
    	ON A.pizza_id = C.pizza_id
    GROUP BY A.customer_id, C.pizza_name
    ORDER BY customer_id
    ;

|customer_id|pizza_name|no_of_pizzas|
|-----------|----------|-----|
|101        |Meatlovers|2    |
|101        |Vegetarian|1    |
|102        |Meatlovers|2    |
|102        |Vegetarian|1    |
|103        |Meatlovers|3    |
|103        |Vegetarian|1    |
|104        |Meatlovers|3    |
|105        |Vegetarian|1    |

<br>

#### Q6. What was the maximum number of pizzas delivered in a single order?

    SELECT COUNT(A. order_id) AS max_pizzas_delivered_in_single_order
    FROM customer_orders AS A
    INNER JOIN runner_orders AS B
    	ON A.order_id = B.order_id
    INNER JOIN pizza_names AS C
    	ON A.pizza_id = C.pizza_id
    WHERE B.pickup_time != 'null' 
    GROUP BY A.order_id
    ORDER BY max_pizzas_delivered_in_single_order DESC
    LIMIT 1
    ;
|max_pizzas_delivered_in_single_order|
|------------------------------------|
|3                                   |

<br>

#### Q7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

    SELECT A.customer_id
    	,CASE
         WHEN COALESCE(A.exclusions, '') IN ('', 'null') AND COALESCE(A.extras, '') IN ('', 'null')	THEN 'No Change'
         ELSE 'At Least 1 Change'
         END AS changes
        ,COUNT(*) AS No_of_DeliveredPizzas 
    FROM customer_orders AS A
    INNER JOIN runner_orders AS B
    	ON A.order_id = B.order_id
    INNER JOIN pizza_names AS C
    	ON A.pizza_id = C.pizza_id
    WHERE B.pickup_time != 'null' 
    GROUP BY changes, A.customer_id
    ORDER BY A.customer_id
    ;
|customer_id|changes          |no_of_deliveredpizzas|
|-----------|-----------------|---------------------|
|101        |No Change        |2                    |
|102        |No Change        |3                    |
|103        |At Least 1 Change|3                    |
|104        |No Change        |1                    |
|104        |At Least 1 Change|2                    |
|105        |At Least 1 Change|1                    |

<br>

#### Q8. How many pizzas were delivered that had both exclusions and extras?

    SELECT SUM(no_of_deliveredpizzas) AS no_of_deliveredpizzas_both
    FROM (SELECT A.customer_id
            ,CASE
             WHEN COALESCE(A.exclusions, '') NOT IN ('', 'null') AND COALESCE(A.extras, '') NOT IN ('', 'null')	THEN 'Both'
             ELSE 'Not Both'
             END AS changes
            ,COUNT(*) AS No_of_DeliveredPizzas 
          FROM customer_orders AS A
          INNER JOIN runner_orders AS B
              ON A.order_id = B.order_id
          INNER JOIN pizza_names AS C
              ON A.pizza_id = C.pizza_id
          WHERE B.pickup_time != 'null' 
          GROUP BY changes, A.customer_id) AS SQ
    WHERE changes = 'Both'
    ;

|no_of_deliveredpizzas_both|
|--------------------------|
|1                         |

<br>


#### Q9. What was the total volume of pizzas ordered for each hour of the day?

    SELECT EXTRACT(HOUR FROM order_time) hour_of_the_day
    	, COUNT(*) AS volume_of_pizzas
    FROM customer_orders
    GROUP BY hour_of_the_day
    ORDER BY hour_of_the_day
    ;
|hour_of_the_day|volume_of_pizzas|
|---------------|----------------|
|11             |1               |
|13             |3               |
|18             |3               |
|19             |1               |
|21             |3               |
|23             |3               |

<br>

#### Q10. What was the volume of orders for each day of the week?

    SELECT TO_CHAR(order_time, 'DAY') AS day_of_the_week
    	, COUNT(*) AS volume_of_pizzas
    FROM customer_orders
    GROUP BY day_of_the_week
    ;

|day_of_the_week|volume_of_pizzas|
|---------------|----------------|
|WEDNESDAY      |5               |
|THURSDAY       |3               |
|FRIDAY         |1               |
|SATURDAY       |5               |

<br>


