# Case Study #2 - Pizza Runner 
![2 (Custom)](https://github.com/user-attachments/assets/72dd5cda-6170-4ab8-86ed-80c3214b5e31) 
<br>
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

#### A1. How many pizzas were ordered?
```sql
    SELECT COUNT(*) AS no_of_pizzas_ordered
    FROM customer_orders
    ;
```
|no_of_pizzas_ordered|
|--------------------|
|14                  |

<br>

#### A2. How many unique customer orders were made?
```sql
    SELECT COUNT(DISTINCT order_id) AS unique_customer_orders
    FROM customer_orders
    ;
```
|unique_customer_orders|
|----------------------|
|10                    |

<br>

#### A3. How many successful orders were delivered by each runner?
```sql
    SELECT runner_id
    	, COUNT(*) AS number_of_orders_delivered
    FROM runner_orders
    WHERE pickup_time IS NOT NULL
    GROUP BY runner_id
    ORDER BY runner_id
    ;
```
|runner_id|number_of_orders_delivered|
|---------|--------------------------|
|1        |4                         |
|2        |4                         |
|3        |2                         |

<br>

#### A3. How many successful orders were delivered by each runner?
```sql
    SELECT runner_id
    	, COUNT(*) AS number_of_orders_delivered
    FROM runner_orders
    WHERE pickup_time != 'null'
    GROUP BY runner_id
    ORDER BY runner_id
    ;
```
|runner_id|number_of_orders_delivered|
|---------|--------------------------|
|1        |4                         |
|2        |3                         |
|3        |1                         |

<br>

#### A4. How many of each type of pizza was delivered?
```sql
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
```
|pizza_id|pizza_name|count|
|--------|----------|-----|
|1       |Meatlovers|9    |
|2       |Vegetarian|3    |

<br>

#### A5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
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
```
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

#### A6. What was the maximum number of pizzas delivered in a single order?
```sql
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
```
<br>

#### A7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
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
```
|customer_id|changes          |no_of_deliveredpizzas|
|-----------|-----------------|---------------------|
|101        |No Change        |2                    |
|102        |No Change        |3                    |
|103        |At Least 1 Change|3                    |
|104        |No Change        |1                    |
|104        |At Least 1 Change|2                    |
|105        |At Least 1 Change|1                    |

<br>

#### A8. How many pizzas were delivered that had both exclusions and extras?
```sql
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
```
|no_of_deliveredpizzas_both|
|--------------------------|
|1                         |

<br>


#### A9. What was the total volume of pizzas ordered for each hour of the day?
```sql
    SELECT EXTRACT(HOUR FROM order_time) hour_of_the_day
    	, COUNT(*) AS volume_of_pizzas
    FROM customer_orders
    GROUP BY hour_of_the_day
    ORDER BY hour_of_the_day
    ;
```
|hour_of_the_day|volume_of_pizzas|
|---------------|----------------|
|11             |1               |
|13             |3               |
|18             |3               |
|19             |1               |
|21             |3               |
|23             |3               |

<br>

#### A10. What was the volume of orders for each day of the week?
```sql
    SELECT TO_CHAR(order_time, 'DAY') AS day_of_the_week
    	, COUNT(*) AS volume_of_pizzas
    FROM customer_orders
    GROUP BY day_of_the_week
    ;
```
|day_of_the_week|volume_of_pizzas|
|---------------|----------------|
|WEDNESDAY      |5               |
|THURSDAY       |3               |
|FRIDAY         |1               |
|SATURDAY       |5               |

<br>

## B. Runner and Customer Experience

#### B1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
    SELECT FLOOR((registration_date - DATE '2021-01-01') / 7) + 1 AS week
    	, COUNT(runner_id) AS no_of_runners
    FROM runners
    GROUP BY week
    ORDER BY week
    ;
```
|week|no_of_runners|
|----|-------------|
|1   |2            |
|2   |1            |
|3   |1            |

<br>

#### B2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
    SELECT runner_id
    	, ROUND(AVG(minutes)) AS average_time_minutes
    FROM (SELECT B.runner_id
        	, EXTRACT(EPOCH FROM (TO_TIMESTAMP(B.pickup_time, 'YYYY-MM-DD HH24:MI:SS')
                  	- A.order_time)) / 60 AS minutes
    	  FROM customer_orders AS A
    	  LEFT JOIN runner_orders AS B
    			ON A.order_id = B.order_id
    	  WHERE pickup_time != 'null') AS SQ
    GROUP BY runner_id
    ORDER BY runner_id
    ;
```
|runner_id|average_time_minutes|
|---------|--------------------|
|1        |16                  |
|2        |24                  |
|3        |10                  |

<br>

#### B3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
    SELECT no_of_pizzas_in_orders
    	, ROUND(AVG(avg_prep_duration)) AS avg_prep_duration
        
    FROM (SELECT COUNT(order_id) AS no_of_pizzas_in_orders
              , ROUND(AVG(minutes)) as avg_prep_duration
    
          FROM (SELECT A.order_id 
                  , B.runner_id
                  , EXTRACT(EPOCH FROM (TO_TIMESTAMP(B.pickup_time, 'YYYY-MM-DD HH24:MI:SS')
                          - A.order_time)) / 60 AS minutes
                FROM customer_orders AS A
                LEFT JOIN runner_orders AS B
                      ON A.order_id = B.order_id
                WHERE pickup_time != 'null') AS SQ1
    
          GROUP BY order_id) AS SQ2
    GROUP BY no_of_pizzas_in_orders
    ORDER BY avg_prep_duration DESC
    ;
```
|no_of_pizzas_in_orders|avg_prep_duration|
|----------------------|-----------------|
|3                     |29               |
|2                     |18               |
|1                     |12               |

*Note: More the number of pizzas in an order more the preparation time*

<br>

#### B4. What was the average distance travelled for each customer?
```sql
    SELECT customer_id
    	, AVG(total_distance) AS avg_distance_travelled_minutes
        
    FROM (SELECT A.customer_id
              , A.order_id 
              , MIN(CAST(REGEXP_REPLACE(B.distance, '[A-Za-z]', '', 'g') AS FLOAT)) AS total_distance
          FROM customer_orders AS A
          INNER JOIN runner_orders AS B
              ON A.order_id = B.order_id
          WHERE pickup_time != 'null'
          GROUP BY A.order_id, A.customer_id
          ORDER BY A.customer_id, A.order_id) AS SQ
          
    GROUP BY customer_id
    ORDER BY customer_id
    ;
```
|customer_id|avg_distance_travelled_minutes|
|-----------|----------------------|
|101        |20                    |
|102        |18.4                  |
|103        |23.4                  |
|104        |10                    |
|105        |25                    |

<br>

#### B5. What was the difference between the longest and shortest delivery times for all orders?
```sql
    SELECT MAX(total_duration) - MIN(total_duration) AS difference_longest_shortest_delivery_minutes
    
    FROM  (SELECT  A.order_id 
              , MIN(CAST(REGEXP_REPLACE(B.duration, '[A-Za-z]', '', 'g') AS INT)) AS total_duration
          FROM customer_orders AS A
          INNER JOIN runner_orders AS B
              ON A.order_id = B.order_id
          WHERE pickup_time != 'null'
          GROUP BY A.order_id, A.customer_id) AS SQ
    ;
```
|difference_longest_shortest_delivery_minutes|
|--------------------------------------------|
|30                                          |

<br>

#### B6. What was the average speed for each runner for each delivery and do you notice any trend for these values?SELEC
```sql
    SELECT runner_id
    	, order_id
        , ROUND(AVG(CAST(REGEXP_REPLACE(distance, '[A-Za-z]', '', 'g') AS FLOAT)
          	   		/
    	  	  		(CAST(REGEXP_REPLACE(duration, '[A-Za-z]', '', 'g') AS FLOAT) / 60)
          	  		)) AS avg_speed_km_per_hr
    FROM runner_orders
    WHERE pickup_time != 'null'
    GROUP BY runner_id, order_id
    ORDER BY runner_id, order_id
```
|ner_id|order_id|avg_speed_km_per_hr|
|------|--------|-------------------|
|1     |1       |38                 |
|1     |2       |44                 |
|1     |3       |40                 |
|1     |10      |60                 |
|2     |4       |35                 |
|2     |7       |60                 |
|2     |8       |94                 |
|3     |5       |40                 |

<br>

#### B7. What is the successful delivery percentage for each runner?
```sql
    SELECT runner_id
    	, CAST(delivery_flag AS FLOAT) / CAST(total_orders AS FLOAT) * 100 AS successful_delivery_percentage
        
    FROM (SELECT runner_id
            , SUM(CASE
              WHEN pickup_time = 'null' THEN 0 ELSE 1
              END) AS delivery_flag
            , COUNT(*) AS total_orders
    	 FROM runner_orders
         GROUP BY runner_id
    	 ORDER BY runner_id) AS SQ
    ;
```
|runner_id|successful_delivery_percentage|
|---------|------------------------------|
|1        |100                           |
|2        |75                            |
|3        |50                            |

<br>

## C. Ingredient Optimisation

#### C1. What are the standard ingredients for each pizza?
```sql
    SELECT A.pizza_id
    	, pizza_name
    	, topping_id
        , topping_name
    FROM (SELECT pizza_id
    		 , CAST(UNNEST(STRING_TO_ARRAY(toppings, ',')) AS INT) AS toppings -- split value to rows
    	  FROM pizza_recipes) AS A
    LEFT JOIN pizza_toppings AS B
    	ON A.toppings = B.topping_id
    INNER JOIN pizza_names AS C
    	ON A.pizza_id = C.pizza_id
    ORDER BY A.pizza_id, topping_id
    ;
```
|pizza_id|pizza_name|topping_id|topping_name|
|--------|----------|----------|------------|
|1       |Meatlovers|1         |Bacon       |
|1       |Meatlovers|2         |BBQ Sauce   |
|1       |Meatlovers|3         |Beef        |
|1       |Meatlovers|4         |Cheese      |
|1       |Meatlovers|5         |Chicken     |
|1       |Meatlovers|6         |Mushrooms   |
|1       |Meatlovers|8         |Pepperoni   |
|1       |Meatlovers|10        |Salami      |
|2       |Vegetarian|4         |Cheese      |
|2       |Vegetarian|6         |Mushrooms   |
|2       |Vegetarian|7         |Onions      |
|2       |Vegetarian|9         |Peppers     |
|2       |Vegetarian|11        |Tomatoes    |
|2       |Vegetarian|12        |Tomato Sauce|

<br>

#### C2. What was the most commonly added extra?
```sql
    SELECT topping_name AS most_common_extra
    	,COUNT(*) AS COUNT
    FROM (SELECT CAST(UNNEST(STRING_TO_ARRAY(extras, ',')) AS INT) AS extras
    	  FROM customer_orders
    	  WHERE extras NOT IN ('null', '')) AS A
    INNER JOIN pizza_toppings AS B
    	ON A.extras = B.topping_id
    GROUP BY topping_name
    ORDER BY COUNT DESC
    LIMIT 1
    ;
```
|most_common_extra|count|
|-----------------|-----|
|Bacon            |4    |

<br>

#### C3. What was the most common exclusion?
```sql
    SELECT topping_name AS most_common_exclusion
    	,COUNT(*) AS COUNT
    FROM (SELECT CAST(UNNEST(STRING_TO_ARRAY(exclusions, ',')) AS INT) AS exclusions
    	  FROM customer_orders
    	  WHERE exclusions NOT IN ('null', '')) AS A
    INNER JOIN pizza_toppings AS B
    	ON A.exclusions = B.topping_id
    GROUP BY topping_name
    ORDER BY COUNT DESC
    LIMIT 1
    ;
```
|most_common_exclusion|count|
|---------------------|-----|
|Cheese               |4    |


#### C4. Generate an order item for each record in the customers_orders table in the format of one of the following:
<br>
-- Meat Lovers
<br>
-- Meat Lovers - Exclude Beef
<br>
-- Meat Lovers - Extra Bacon
<br>
-- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Pepper
<br>

```sql
    WITH  A AS (SELECT ROW_NUMBER() OVER() AS id
              , order_id
              , customer_id
              , pizza_id
              , CASE
                WHEN exclusions IN ('', 'null', NULL) THEN '0' 
                ELSE exclusions
                END AS exclusions 
              , CASE
                WHEN extras IN ('', 'null', NULL) THEN '0' 
                ELSE extras
                END AS extras
          FROM customer_orders),
           
          B AS (SELECT id, order_id, customer_id, pizza_id
        		, CAST(UNNEST(STRING_TO_ARRAY(exclusions, ',')) AS INT) AS exclusions
        		, CAST(UNNEST(STRING_TO_ARRAY(extras, ',')) AS INT) AS extras
     		FROM A ),
            
	      C AS (SELECT B.id
              , B.order_id
              , customer_id
              , pizza_name
              , P.topping_name AS exclusions
              , extras
          FROM B
          LEFT JOIN pizza_toppings AS P
              ON (B.exclusions = P.topping_id)
          INNER JOIN pizza_names AS N
            	ON B.pizza_id = N.pizza_id
          ORDER BY id),
          
          D AS (SELECT C.id
                  , C.order_id
                  , C.customer_id
                  , C.pizza_name
                  , C.exclusions
                  , Q.topping_name AS extras
            FROM C
            LEFT JOIN pizza_toppings AS Q
                ON (C.extras = Q.topping_id)
            ORDER BY id),
            
          E AS  (SELECT id
                    , order_id
                    , customer_id
                    , pizza_name
                    , STRING_AGG(exclusions, ', ' ORDER BY exclusions ) AS exclusions
                    , STRING_AGG(extras, ', ' ORDER BY extras ) AS extras
            FROM D
            GROUP BY id, order_id, customer_id, pizza_name)
            
    SELECT order_id
	, customer_id
    , CASE
      WHEN COALESCE(exclusions, 'Unknown') = 'Unknown'
      	AND COALESCE(extras, 'Unknown') = 'Unknown' THEN pizza_name
      WHEN COALESCE(exclusions, 'Unknown') = 'Unknown'
      	AND COALESCE(extras, 'Unknown') != 'Unknown' THEN CONCAT(pizza_name, ' - ', 'Extra ', extras)
      WHEN COALESCE(exclusions, 'Unknown') != 'Unknown'
      	AND COALESCE(extras, 'Unknown') = 'Unknown' THEN CONCAT(pizza_name, ' - ', 'Exclude ', exclusions)
      ELSE CONCAT(pizza_name, ' - ', 'Exclude ', exclusions,' - ', 'Extra ', extras)
      END AS order_item
    FROM E
    ORDER BY order_id
    ;
```
|order_id|customer_id|order_item                                                         |
|--------|-----------|---------------------------------------------------------------|
|1       |101        |Meatlovers                                                     |
|2       |101        |Meatlovers                                                     |
|3       |102        |Meatlovers                                                     |
|3       |102        |Vegetarian                                                     |
|4       |103        |Meatlovers - Exclude Cheese                                    |
|4       |103        |Meatlovers - Exclude Cheese                                    |
|4       |103        |Vegetarian - Exclude Cheese                                    |
|5       |104        |Meatlovers - Extra Bacon                                       |
|6       |101        |Vegetarian                                                     |
|7       |105        |Vegetarian - Extra Bacon                                       |
|8       |102        |Meatlovers                                                     |
|9       |103        |Meatlovers - Exclude Cheese - Extra Bacon, Chicken             |
|10      |104        |Meatlovers                                                     |
|10      |104        |Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese|

<br>

#### C5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients.
<br>
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

```sql
	WITH  A AS (SELECT ROW_NUMBER() OVER() AS id
	              , order_id
	              , customer_id
	              , pizza_id
	              , CASE
	                WHEN exclusions IN ('', 'null', NULL) THEN '0' 
	                ELSE exclusions
	                END AS exclusions 
	              , CASE
	                WHEN extras IN ('', 'null', NULL) THEN '0' 
	                ELSE extras
	                END AS extras
	        	FROM customer_orders),
	      
	      B AS (SELECT pizza_id
	              , CAST(UNNEST(STRING_TO_ARRAY(toppings, ',')) AS INT) AS topping_id
	            FROM pizza_recipes
	            ORDER BY pizza_id, topping_id ),
	           
	      C AS (SELECT A.id
	           		, A.order_id
	            	, A.customer_id
	            	, A.pizza_id
	            	, A.exclusions
	            	, A.extras
	            	, B.topping_id
	            FROM A
	            LEFT JOIN B
	            	ON A.pizza_id = B.pizza_id
	            ORDER BY A.id, A.order_id, A.customer_id
	            ), 
	       
	       D AS (SELECT id
	                , order_id
	                , customer_id
	                , pizza_id
	                , topping_id
	                , CAST(UNNEST(STRING_TO_ARRAY(exclusions, ',')) AS INT) AS exclusions
	                , CAST(UNNEST(STRING_TO_ARRAY(extras, ',')) AS INT) AS extras
	            FROM C
	            ),
	            
	       E AS (SELECT id
	                , order_id
	                , customer_id
	                , pizza_id
	                , topping_id
	                , (CASE
	                   WHEN topping_id = exclusions THEN 1
	                   WHEN topping_id = extras THEN 2
	                   ELSE 0
	                   END) AS Calc
	                , exclusions
	                , extras
	            FROM D
	            ),
	            
	       F AS (SELECT id
	                , order_id
	                , customer_id
	                , pizza_name
	                , topping_name
	                , MAX(calc) as Flag
	            FROM E
	            LEFT JOIN pizza_names
	                ON E.pizza_id = pizza_names.pizza_id
	            LEFT JOIN pizza_toppings
	                ON E.topping_id = pizza_toppings.topping_id
	            GROUP BY id
	                , order_id
	                , customer_id
	                , pizza_name
	                , topping_name
	            ORDER BY id
	                , order_id
	                , customer_id
	                , pizza_name
	                , topping_name
	             ),
	                
	        G AS (SELECT id
	               , order_id
	               , customer_id
	               , pizza_name
	               , CASE
	                  WHEN flag = 2 THEN CONCAT(' ', CAST(flag AS TEXT), 'X',topping_name) 
	                  WHEN flag = 0 THEN CONCAT(' ',topping_name)
	                  ELSE ''
	                  END AS relevant_ingredients
	            FROM F
                WHERE flag IN (2, 0)  
	            )
	
	SELECT order_id
	    , customer_id
	    , CONCAT('"', pizza_name,':', STRING_AGG(relevant_ingredients, ','), '"') AS relevant_ingredients
	FROM G
	GROUP BY id
	    , order_id
	    , customer_id
	    , pizza_name
    ;
```
|order_id|customer_id|relevant_ingredients                                                               |
|--------|-----------|-----------------------------------------------------------------------------------|
|1       |101        |Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
|2       |101        |Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
|3       |102        |Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
|3       |102        |Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes             |
|4       |103        |Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami          |
|4       |103        |Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami          |
|4       |103        |Vegetarian: Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                     |
|5       |104        |Meatlovers: BBQ Sauce, 2XBacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami|
|6       |101        |Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes             |
|7       |105        |Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes             |
|8       |102        |Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
|9       |103        |Meatlovers: BBQ Sauce, 2XBacon, Beef, 2XChicken, Mushrooms, Pepperoni, Salami      |
|10      |104        |Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
|10      |104        |Meatlovers: 2XBacon, Beef, 2XCheese, Chicken, Pepperoni, Salami                    |


<br>

#### C6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
```sql
	WITH  A AS (SELECT ROW_NUMBER() OVER() AS id
	              , order_id
	              , customer_id
	              , pizza_id
	              , CASE
	                WHEN exclusions IN ('', 'null', NULL) THEN '0' 
	                ELSE exclusions
	                END AS exclusions 
	              , CASE
	                WHEN extras IN ('', 'null', NULL) THEN '0' 
	                ELSE extras
	                END AS extras
	        	FROM customer_orders),
	      
	      B AS (SELECT pizza_id
	              , CAST(UNNEST(STRING_TO_ARRAY(toppings, ',')) AS INT) AS topping_id
	            FROM pizza_recipes
	            ORDER BY pizza_id, topping_id ),
	           
	      C AS (SELECT A.id
	           		, A.order_id
	            	, A.customer_id
	            	, A.pizza_id
	            	, A.exclusions
	            	, A.extras
	            	, B.topping_id
	            FROM A
	            LEFT JOIN B
	            	ON A.pizza_id = B.pizza_id
	            ORDER BY A.id, A.order_id, A.customer_id
	            ), 
	       
	       D AS (SELECT id
	                , order_id
	                , customer_id
	                , pizza_id
	                , topping_id
	                , CAST(UNNEST(STRING_TO_ARRAY(exclusions, ',')) AS INT) AS exclusions
	                , CAST(UNNEST(STRING_TO_ARRAY(extras, ',')) AS INT) AS extras
	            FROM C
	            ),
	            
	       E AS (SELECT id
	                , order_id
	                , customer_id
	                , pizza_id
	                , topping_id
	                , (CASE
	                   WHEN topping_id = exclusions THEN 1
	                   WHEN topping_id = extras THEN 2
	                   ELSE 0
	                   END) AS Calc
	                , exclusions
	                , extras
	            FROM D
	            ),
	            
	       F AS (SELECT id
	                , order_id
	                , customer_id
	                , pizza_name
                 	, E.topping_id
	                , topping_name
	                , MAX(calc) as Flag
	            FROM E
	            LEFT JOIN pizza_names
	                ON E.pizza_id = pizza_names.pizza_id
	            LEFT JOIN pizza_toppings
	                ON E.topping_id = pizza_toppings.topping_id
	            GROUP BY id
	                , order_id
	                , customer_id
	                , pizza_name
                 	, E.topping_id
	                , topping_name
	            ORDER BY id
	                , order_id
	                , customer_id
	                , pizza_name
                 	, E.topping_id
	                , topping_name
	             )
	
	SELECT topping_name
		, SUM(CASE WHEN flag = 0 THEN 1 ELSE flag END) AS total_quantity
	FROM F
	INNER JOIN runner_orders
		ON F.order_id = runner_orders.order_id
	WHERE pickup_time != 'null' AND flag IN (0, 2)
	GROUP BY topping_name
	ORDER BY total_quantity DESC
    ;
```      
|topping_name|total_quantity|
|------------|--------------|
|Bacon       |11            |
|Mushrooms   |11            |
|Cheese      |10            |
|Pepperoni   |9             |
|Chicken     |9             |
|Salami      |9             |
|Beef        |9             |
|BBQ Sauce   |8             |
|Tomato Sauce|3             |
|Onions      |3             |
|Tomatoes    |3             |
|Peppers     |3             |

<br>

## D. Pricing and Ratings

#### D1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```sql
	SELECT CONCAT('$', CAST(SUM(CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END) AS TEXT)) AS total_revenue
	FROM customer_orders AS A
	LEFT JOIN runner_orders AS B
		ON A.order_id = B.order_id
	WHERE B.pickup_time != 'null'
	;
```
|total_revenue|
|-------------|
|$138         |

<br>

#### D2. What if there was an additional $1 charge for any pizza extras?
<br>
Add cheese is $1 extra

```sql
	WITH base AS	(SELECT A.order_id
	            		, SUM(CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END) base_price
	                FROM customer_orders AS A
	                LEFT JOIN runner_orders AS B
	                    ON A.order_id = B.order_id
	                WHERE B.pickup_time != 'null'
	                GROUP BY A.order_id
	                ),
	
	     extra AS   (SELECT order_id
	                    , SUM(COUNT) AS extra_price
	                FROM (
	                      SELECT A.order_id
	                          , UNNEST(STRING_TO_ARRAY(
			   		  (CASE WHEN extras IN ('', 'null') THEN NULL ELSE extras END)
	  				   , ',')) AS extras
	                          , 1 AS COUNT
	                      FROM customer_orders AS A
	                      LEFT JOIN runner_orders AS B
	                          ON A.order_id = B.order_id
	                      WHERE B.pickup_time != 'null') AS SQ
	                GROUP BY order_id
	                 )
	
	SELECT CONCAT('$', CAST(SUM((base_price + COALESCE(extra_price, '0'))) AS TEXT)) AS total_revenue_with_extra
	FROM base
	LEFT JOIN extra
		ON base.order_id = extra.order_id
	;
```
|total_revenue_with_extra|
|------------------------|
|$142                    |



<br>

#### D3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```sql
	DROP TABLE IF EXISTS ratings;
	CREATE TABLE ratings (
	  "order_id" INTEGER,
	  "rating" INTEGER
	);
	INSERT INTO ratings
	  ("order_id", "rating")
	VALUES
	  (1, '4'),
	  (2, '5'),
	  (3, '5'),
	  (4, '4'),
	  (5, '4'),
	  (6, NULL),
	  (7, '3'),
	  (8, '5'),
	  (9, NULL),
	  (10, '4')
	  ;
```
Note: Table Created
|order_id|rating|
|--------|------|
|1       |4     |
|2       |5     |
|3       |5     |
|4       |4     |
|5       |4     |
|6       |null  |
|7       |3     |
|8       |5     |
|9       |null  |
|10      |4     |

<br>

#### D4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
<br>
customer_id
<br>
order_id
<br>
runner_id
<br>
rating
<br>
order_time
<br>
pickup_time
<br>
Time between order and pickup
<br>
Delivery duration
<br>
Average speed
<br>
Total number of pizzas

```sql
	WITH X AS   (SELECT A.order_id
	                , customer_id
	             	, MAX(order_time) AS order_time
	                , COUNT(*) AS number_of_pizzas
	                , B.rating
	            FROM customer_orders AS A
	            LEFT JOIN ratings AS B
	                ON A.order_id = B.order_id
	            GROUP BY A.order_id
	                , customer_id
	                , B.rating
	            ORDER BY A.order_id
	             )
	             
	SELECT customer_id
		, X.order_id AS order_id
	    , runner_id
	    , rating
	    , order_time
	    , CAST(pickup_time AS TIMESTAMP) AS pickup_time
	    , ROUND(
	      EXTRACT(EPOCH FROM (CAST(pickup_time AS TIMESTAMP) - order_time)) / 60
	      ) AS time_btw_order_pickup_minutes
	    , CAST(REGEXP_REPLACE(duration, '[A-Za-z]', '', 'g') AS INT) AS duration_minutes
	    , ROUND((CAST(REGEXP_REPLACE(distance, '[A-Za-z]', '', 'g') AS FLOAT)
	      	   		/
	    		(CAST(REGEXP_REPLACE(duration, '[A-Za-z]', '', 'g') AS FLOAT) / 60))
	       	   ) AS avg_speed_km_per_hr
		 , number_of_pizzas AS total_number_of_pizzas
	FROM X
	LEFT JOIN runner_orders
		ON X.order_id = runner_orders.order_id
	WHERE pickup_time != 'null'
	;
```
|customer_id|order_id|runner_id|rating|order_time         |pickup_time        |time_btw_order_pickup_minutes|duration_minutes|avg_speed_km_per_hr|total_number_of_pizzas|
|-----------|--------|---------|------|-------------------|-------------------|-----------------------------|----------------|-------------------|----------------------|
|101        |1       |1        |4     |2020-01-01 18:05:02|2020-01-01 18:15:34|11                           |32              |38                 |1                     |
|101        |2       |1        |5     |2020-01-01 19:00:52|2020-01-01 19:10:54|10                           |27              |44                 |1                     |
|102        |3       |1        |5     |2020-01-02 23:51:23|2020-01-03 00:12:37|21                           |20              |40                 |2                     |
|103        |4       |2        |4     |2020-01-04 13:23:46|2020-01-04 13:53:03|29                           |40              |35                 |3                     |
|104        |5       |3        |4     |2020-01-08 21:00:29|2020-01-08 21:10:57|10                           |15              |40                 |1                     |
|105        |7       |2        |3     |2020-01-08 21:20:29|2020-01-08 21:30:45|10                           |25              |60                 |1                     |
|102        |8       |2        |5     |2020-01-09 23:54:33|2020-01-10 00:15:02|20                           |15              |94                 |1                     |
|104        |10      |1        |4     |2020-01-11 18:34:49|2020-01-11 18:50:20|16                           |10              |60                 |2                     |

<br>

#### D5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```sql
	WITH X AS   (SELECT A.order_id
	                , customer_id
	             	, MAX(order_time) AS order_time
	                , SUM(CASE WHEN A.pizza_id = 1 THEN 12 ELSE 10 END) AS pizza_price 
	                , B.rating
	            FROM customer_orders AS A
	            LEFT JOIN ratings AS B
	                ON A.order_id = B.order_id
	            GROUP BY A.order_id
	                , customer_id
	                , B.rating
	            ORDER BY A.order_id
	             )
	             
	SELECT CONCAT('$',
	                   CAST (ROUND(
	                   SUM(
	                   pizza_price -
	                   CAST(REGEXP_REPLACE(distance, '[A-Za-z]', '', 'g') AS FLOAT) * 0.3 
	                   )) AS TEXT)) AS amount_leftover
	FROM X
	LEFT JOIN runner_orders
		ON X.order_id = runner_orders.order_id
```
|amount_leftover|
|---------------|
|$94            |

<br>

## E. Bonus Questions
If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?


