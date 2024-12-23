# Case Study #6 - Clique Bait
![6 (Custom)](https://github.com/user-attachments/assets/ccdcb10d-ecfc-4ace-87df-7383a93b70fd)
<br>
Challenge Source: https://8weeksqlchallenge.com/case-study-6/
<br>

### Introduction
Clique Bait is not like your regular online seafood store - the founder and CEO Danny, was also a part of a digital data analytics team and wanted to expand his knowledge into the seafood industry!
<br>

In this case study - you are required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.
<br>

### Available Data
For this case study there is a total of 5 datasets which you will need to combine to solve all of the questions
<br>

<br>
Test the Queries here:
https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17
<br>

## 1. Enterprise Relationship Diagram
### Using the following DDL schema details to create an ERD for all the Clique Bait datasets.
<br>

Click [here](https://dbdiagram.io/d/67698808d16109b4009addb8) to access the DB Diagram tool to create the ERD.

```sql
TABLE event_identifier {
  event_type INTEGER
  event_name VARCHAR(13)
}

TABLE campaign_identifier {
  "campaign_id" INTEGER
  "products" VARCHAR(3)
  "campaign_name" VARCHAR(33)
  "start_date" TIMESTAMP
  "end_date" TIMESTAMP
}

TABLE page_hierarchy {
  "page_id" INTEGER
  "page_name" VARCHAR(14)
  "product_category" VARCHAR(9)
  "product_id" INTEGER
}

TABLE users  {
  "user_id" INTEGER
  "cookie_id" VARCHAR(6)
  "start_date" TIMESTAMP
}

TABLE events   {
  "visit_id" VARCHAR(6)
  "cookie_id" VARCHAR(6)
  "page_id" INTEGER
  "event_type" INTEGER
  "sequence_number" INTEGER
  "event_time" TIMESTAMP
}

Ref: events.cookie_id > users.cookie_id //many to one
Ref: events.page_id > page_hierarchy.page_id
Ref: events.event_type > event_identifier.event_type
Ref: campaign_identifier.products < page_hierarchy.product_id
```
The Entity Relationship Diagram is shown below with the data types
<br>
![image](https://github.com/user-attachments/assets/70038926-99da-4c24-bc81-edc5a4cbff18)

## 2. Digital Analysis
### 2.1 How many users are there?
```sql
SELECT COUNT (DISTINCT user_id) AS number_of_user
FROM clique_bait.users
;
```
|number_of_user|
|--------------|
|500           |

### 2.2 How many cookies does each user have on average?
```sql
SELECT ROUND(AVG(number_of_cookie)) AS avg_cookie_per_user
FROM (SELECT user_id
		, COUNT (DISTINCT cookie_id) AS number_of_cookie
	  FROM clique_bait.users
	  GROUP BY user_id) AS SQ
;
```
|avg_cookie_per_user|
|-------------------|
|4                  |

### 2.3 What is the unique number of visits by all users per month?
```sql
SELECT EXTRACT(MONTH FROM event_time) AS month
	, COUNT (DISTINCT visit_id) AS unique_visits
FROM clique_bait.users AS U
INNER JOIN clique_bait.events AS E
	ON U.cookie_id = E.cookie_id
GROUP BY 1
;
```
|month|unique_visits|
|-----|-------------|
|1    |876          |
|2    |1488         |
|3    |916          |
|4    |248          |
|5    |36           |


### 2.4 What is the number of events for each event type?
```sql
SELECT E.event_type
	, I.event_name
	, COUNT(*) AS number_of_events
FROM clique_bait.events AS E
INNER JOIN clique_bait.event_identifier AS I
	ON E.event_type = I.event_type
GROUP BY 1, 2
ORDER BY 1
;
```
|event_type|event_name   |number_of_events|
|----------|-------------|----------------|
|1         |Page View    |20928           |
|2         |Add to Cart  |8451            |
|3         |Purchase     |1777            |
|4         |Ad Impression|876             |
|5         |Ad Click     |702             |


### 2.6 What is the percentage of visits which have a purchase event?

### 2.7 Whqt is the percentage of visits which view the checkout page but do not have a purchase event?

### 2.8 What are the top 3 pages by number of views?

### 2.9 What is the number of views and cart adds for each product category?

### 2.10 What are the top 3 products by purchases?


