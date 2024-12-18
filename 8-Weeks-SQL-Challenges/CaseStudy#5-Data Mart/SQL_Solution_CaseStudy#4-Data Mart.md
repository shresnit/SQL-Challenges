# Case Study #5 - Data Mart
![5 (Custom)](https://github.com/user-attachments/assets/2880807a-ba01-4e1b-b93d-2a143bc992f1)
<br>
Challenge Source: https://8weeksqlchallenge.com/case-study-5/
<br>

### Introduction
Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh produce - Danny is asking for your support to analyse his sales performance.
<br>

In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.
<br>

Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas.
<br>

The key business question he wants you to help him answer are the following:
<br>

- What was the quantifiable impact of the changes introduced in June 2020?
- Which platform, region, segment and customer types were the most impacted by this change?
- What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?

### Available Data
For this case study there is only a single table: data_mart.weekly_sales
<br>

The Entity Relationship Diagram is shown below with the data types made clear, please note that there is only this one table - hence why it looks a little bit lonely!

![image](https://github.com/user-attachments/assets/fd9b1f65-d22c-4fe2-9874-ea4ccb44d82e)


<br>
Test the Queries here:
https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/8
<br>

## 1. Data Cleansing Steps

- In a single query, perform the following operations and generate a new table in the ***data_mart*** schema named ***clean_weekly_sales***:
- Convert the ***week_date*** to a ***DATE*** format
- Add a ***week_number*** as the second column for each ***week_date*** value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a ***month_number*** with the calendar month for each ***week_date*** value as the 3rd column
- Add a ***calendar_year*** column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called ***age_band*** after the original ***segment*** column using the following mapping on the number inside the ***segment*** value

<div align="center">
  
|segment|age_band    |
|-------|------------|
|1      |Young Adults|
|2      |Middle Aged |
|3 or 4 |Retirees    |

</div>

- Add a new ***demographic*** column using the following mapping for the first letter in the ***segment*** values:

<div align="center">
  
|segment|demographic |
|-------|------------|
|C      |Couples     |
|F      |Families    |

</div>
<br>

- Ensure all ***null*** string values with an ***"unknown"*** string value in the original ***segment*** column as well as the new ***age_band*** and ***demographic*** columns
- Generate a new ***avg_transaction*** column as the ***sales*** value divided by ***transactions*** rounded to 2 decimal places for each record
<br>
<br>

**Solution**
<br>

```sql
DROP TABLE IF EXISTS data_mart.clean_weekly_sales
;
  
CREATE TABLE data_mart.clean_weekly_sales (
  "week_date" DATE,
  "week_number" INTEGER,
  "month_number" INTEGER,
  "calendar_year" INTEGER,
  "region" VARCHAR(13),
  "platform" VARCHAR(7),
  "segment" VARCHAR(10),
  "age_band" VARCHAR(12),
  "demographic" VARCHAR(8),
  "customer_type" VARCHAR(8),
  "transactions" INTEGER,
  "sales" INTEGER,
  "avg_transaction" NUMERIC
);

INSERT INTO	data_mart.clean_weekly_sales ("week_date" 
  	, "week_number" 
  	, "month_number" 
  	, "calendar_year"
  	, "region" 
  	, "platform"
  	, "segment" 
  	, "age_band"
  	, "demographic"
  	, "customer_type"
  	, "transactions"
  	, "sales"
  	, "avg_transaction" 
)    
SELECT TO_DATE(week_date, 'DD/MM/YY') AS week_date
	, EXTRACT('week' FROM TO_DATE(week_date, 'DD/MM/YY') ) AS week_number
    , EXTRACT('month' FROM TO_DATE(week_date, 'DD/MM/YY') ) AS month_number
    , EXTRACT('year' FROM TO_DATE(week_date, 'DD/MM/YY') ) AS calendar_year
    , region
    , platform
    , CASE
      WHEN segment = 'null' THEN 'unknown'
      ELSE segment
      END AS segment
    , CASE
      WHEN REGEXP_REPLACE(segment, '\D+', '', 'g') = '1' THEN 'Young Adults'
      WHEN REGEXP_REPLACE(segment, '\D+', '', 'g')  = '2' THEN 'Middle Aged'
      WHEN REGEXP_REPLACE(segment, '\D+', '', 'g') IN ('3', '4') THEN 'Retirees'
      ELSE 'unknown'
      END AS age_band
    , CASE
      WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
      WHEN LEFT(segment, 1) = 'F' THEN 'Families'
      ELSE 'unknown'
      END AS demographic
    , customer_type
    , transactions
    , sales
    , ROUND((CAST(sales AS NUMERIC)) / (CAST(transactions AS NUMERIC)), 2) avg_transaction
FROM data_mart.weekly_sales
;
```
New Table clean_weekly_sales (First 10 rows)
|week_date|week_number |month_number|calendar_year|region|platform|segment|age_band    |demographic|customer_type|transactions|sales   |avg_transaction|
|---------|------------|------------|-------------|------|--------|-------|------------|-----------|-------------|------------|--------|---------------|
|2020-08-31|36          |8           |2020         |ASIA  |Retail  |C3     |Retirees    |Couples    |New          |120631      |3656163 |30.31          |
|2020-08-31|36          |8           |2020         |ASIA  |Retail  |F1     |Young Adults|Families   |New          |31574       |996575  |31.56          |
|2020-08-31|36          |8           |2020         |USA   |Retail  |unknown|unknown     |unknown    |Guest        |529151      |16509610|31.20          |
|2020-08-31|36          |8           |2020         |EUROPE|Retail  |C1     |Young Adults|Couples    |New          |4517        |141942  |31.42          |
|2020-08-31|36          |8           |2020         |AFRICA|Retail  |C2     |Middle Aged |Couples    |New          |58046       |1758388 |30.29          |
|2020-08-31|36          |8           |2020         |CANADA|Shopify |F2     |Middle Aged |Families   |Existing     |1336        |243878  |182.54         |
|2020-08-31|36          |8           |2020         |AFRICA|Shopify |F3     |Retirees    |Families   |Existing     |2514        |519502  |206.64         |
|2020-08-31|36          |8           |2020         |ASIA  |Shopify |F1     |Young Adults|Families   |Existing     |2158        |371417  |172.11         |
|2020-08-31|36          |8           |2020         |AFRICA|Shopify |F2     |Middle Aged |Families   |New          |318         |49557   |155.84         |
|2020-08-31|36          |8           |2020         |AFRICA|Retail  |C3     |Retirees    |Couples    |New          |111032      |3888162 |35.02          |





















