# Case Study #4 - Data Bank
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
<br>
## A. Customer Nodes Exploration
A1. How many unique nodes are there on the Data Bank system?

```sql
SELECT COUNT (DISTINCT node_id) AS no_unique_nodes
FROM customer_nodes
```
|no_unique_nodes|
|---------------|
|5              |

<br>

A2.  What is the number of nodes per region?

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

A3. How many customers are allocated to each region?
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

A4. How many days on average are customers reallocated to a different node?

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

A5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?




