****Case Study #1: Danny's Dinner****
_________________________________
Challenge Source: https://8weeksqlchallenge.com/case-study-1/
<br>

**Introduction**
<br>
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.
<br>

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.
<br>

**Problem Statement**
<br>
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.
<br>
He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.
<br>

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

<br>

Danny has shared with you 3 key datasets for this case study:
<br>

sales
<br>
menu
<br>
members
<br>
<br>
**Solution**
____________
Q1. What is the total amount each customer spent at the restaurant?
 ```sql
    SELECT S.customer_id
    	, SUM(M.price) AS amount_spent
    FROM sales AS S
    INNER JOIN menu AS M
    	ON S.product_id = M.product_id
    GROUP BY customer_id
    ORDER BY customer_id
    ;
```
| customer_id | amount_spent |
| ----------- | --- |
| A           | 76  |
| B           | 74  |
| C           | 36  |

<br>
<br>

Q2. How many days has each customer visited the restaurant?
```sql
    
    SELECT customer_id 
    	, COUNT(order_date) AS days_visited
     FROM (SELECT S.customer_id
          		, S.order_date	
          FROM sales AS S
          INNER JOIN menu AS M
              ON S.product_id = M.product_id
          GROUP BY customer_id, S.order_date
          ) AS SQ
    GROUP BY customer_id
    ORDER BY customer_id
    ;
```
| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

<br>
<br>

Q3. What was the first item from the menu purchased by each customer?
```sql    
    SELECT customer_id
    	, product_name AS first_item
    FROM (SELECT S.customer_id
            , M.product_name
            , S.order_date
            , ROW_NUMBER() OVER (PARTITION BY S.customer_id ORDER BY M.product_name, S.order_date) AS rank
        FROM sales AS S
        INNER JOIN menu AS M
            ON S.product_id = M.product_id) AS SQ
    WHERE rank = 1
    ;
```
| customer_id | first_item |
| ----------- | ---------- |
| A           | curry      |
| B           | curry      |
| C           | ramen      |

<br>
<br>

Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?
 ```sql
        SELECT customer_id
        		, product_name AS most_purchased_item
                , COUNT(*) AS times_purchased
        FROM sales AS S
        INNER JOIN menu AS M
        	ON S.product_id = M.product_id
       
        WHERE product_name = 
                  (SELECT product_name 
                  FROM sales AS S
                  INNER JOIN menu AS M
                      ON S.product_id = M.product_id
                  GROUP BY product_name
                  ORDER BY COUNT(product_name)  DESC
                  LIMIT 1) 
     	GROUP BY customer_id, product_name
        ORDER BY times_purchased DESC
        ;
```
| customer_id | most_purchased_item | times_purchased |
| ----------- | ------------------- | --------------- |
| A           | ramen               | 3               |
| C           | ramen               | 3               |
| B           | ramen               | 2               |

<br>
<br>

Q5. Which item was the most popular for each customer?
```sql
    SELECT customer_id
    	, product_name
        , times_purchased
    FROM
      (SELECT customer_id
    	    , product_name
          , times_purchased
          , MAX(times_purchased) OVER (PARTITION BY customer_id ORDER BY customer_id) AS high_purchase
      FROM (
          SELECT customer_id
              , product_name
              , COUNT(*) AS times_purchased
          FROM sales AS S
          INNER JOIN menu AS M
              ON S.product_id = M.product_id
          GROUP BY customer_id, product_name
          ORDER BY customer_id, times_purchased DESC
      		) AS SQ1 ) AS SQ2
    WHERE times_purchased = high_purchase
```
| customer_id | product_name | times_purchased |
| ----------- | ------------ | --------------- |
| A           | ramen        | 3               |
| B           | ramen        | 2               |
| B           | curry        | 2               |
| B           | sushi        | 2               |
| C           | ramen        | 3               |

<br>
<br>

Q6. Which item was purchased first by the customer after they became a member?
 ```sql   
    SELECT customer_id
    		, product_name
            , first_order_date
    
    FROM (SELECT S.customer_id
                  , M.product_name
                  , S.order_date
                  , MIN(order_date) OVER (PARTITION BY S.customer_id ORDER BY order_date) AS first_order_date
          FROM sales AS S
          INNER JOIN menu AS M
              ON S.product_id = M.product_id
          INNER JOIN members AS Mem
              ON S.customer_id = Mem.customer_id
          WHERE order_date >= join_date
          ORDER BY S.customer_id, order_date) AS SQ
          
    WHERE order_date = first_order_date
    ;
```
| customer_id | product_name | first_order_date |
| ----------- | ------------ | ---------------- |
| A           | curry        | 2021-01-07       |
| B           | sushi        | 2021-01-11       |

<br>
<br>


Q7. Which item was purchased just before the customer became a member?
```sql    
    SELECT customer_id
    		, product_name
            , last_order_date
    FROM (SELECT S.customer_id
                  , M.product_name
                  , S.order_date
                  , Mem.join_date
                  , MAX(order_date) OVER (PARTITION BY S.customer_id ORDER BY order_date DESC) AS last_order_date
          FROM sales AS S
          INNER JOIN menu AS M
              ON S.product_id = M.product_id
          INNER JOIN members AS Mem
              ON S.customer_id = Mem.customer_id
          WHERE order_date < join_date
          ORDER BY S.customer_id, order_date) AS SQ
    WHERE order_date = last_order_date
    ;
```
| customer_id | product_name | last_order_date |
| ----------- | ------------ | --------------- |
| A           | sushi        | 2021-01-01      |
| A           | curry        | 2021-01-01      |
| B           | sushi        | 2021-01-04      |

<br>
<br>

Q8. What is the total items and amount spent for each member before they became a member?
```sql    
    SELECT customer_id
    	, count(DISTINCT product_name) AS total_items
        , SUM(price) AS amount_spent
    FROM (SELECT S.customer_id
                  , M.product_name
                  , M.price
                  
          FROM sales AS S
          INNER JOIN menu AS M
              ON S.product_id = M.product_id
          INNER JOIN members AS Mem
              ON S.customer_id = Mem.customer_id
          WHERE order_date < join_date) AS SQ
     GROUP BY customer_id
    ;
```
| customer_id | total_items | amount_spent |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 2           | 40           |

<br>
<br>

Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql    
    SELECT customer_id
    	, SUM((CASE product_name WHEN 'sushi' THEN 2 ELSE 1 END) *  price * 10) AS points
            
    FROM sales AS S
    INNER JOIN menu AS M
    	ON S.product_id = M.product_id
    GROUP BY customer_id
    ORDER BY customer_id
    ;
```
| customer_id | points |
| ----------- | ------ |
| A           | 860     |
| B           | 940     |
| C           | 360     |

<br>
<br>

Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql    
    SELECT S.customer_id
               , SUM(CASE
                  	WHEN order_date < join_date AND product_name = 'sushi' THEN 2
                    WHEN order_date >= join_date AND (order_date < (join_date + INTERVAL '6 days')) THEN 2
                    WHEN (order_date > (join_date + INTERVAL '6 days')) AND product_name = 'sushi' THEN 2
                    ELSE 1
                    END * price * 10) AS points
                  
          FROM sales AS S
          INNER JOIN menu AS M
              ON S.product_id = M.product_id
          INNER JOIN members AS Mem
              ON S.customer_id = Mem.customer_id
    WHERE order_date <= '2021-01-31'
    GROUP BY S.customer_id
```
| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 820    |














































