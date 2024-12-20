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

<br>

## 2. Data Exploration

### Q1. What day of the week is used for each week_date value?
```sql
SELECT TO_CHAR(week_date, 'Day') AS day_of_week
	, COUNT(*)
FROM data_mart.clean_weekly_sales
GROUP BY day_of_week
;
```
|day_of_week|count|
|-----------|-----|
|Monday     |17117|

<br>

### Q2. What range of week numbers are missing from the dataset?
```sql
WITH A AS (	SELECT extract('week' FROM week_date) AS week_number
			FROM data_mart.clean_weekly_sales
			GROUP BY extract('week' FROM week_date)
			ORDER BY week_number
        	)

SELECT CONCAT('1 - ', CAST((MIN(week_number)-1) AS TEXT)) AS lower_range
	, CONCAT(CAST((MAX(week_number)+ 1) AS TEXT), ' - 52') AS higher_range
FROM A
;
```
|missing_lower_range|missing_higher_range|
|-------------------|--------------------|
|1 - 12             |37 - 52             |

<br>

### Q3. How many total transactions were there for each year in the dataset?
```sql
SELECT calendar_year
	, SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year
;
```
|calendar_year|total_transactions|
|-------------|------------------|
|2018         |346406460         |
|2019         |365639285         |
|2020         |375813651         |

<br>

### Q4. What is the total sales for each region for each month?
```sql
SELECT region
	, month_number
	, SUM(sales) AS total_sales
FROM clean_weekly_sales
GROUP BY region
	, month_number
ORDER BY region
	, month_number
;
```
|region|month_number|total_sales|
|------|------------|-----------|
|AFRICA|3           |567767480  |
|AFRICA|4           |1911783504 |
|AFRICA|5           |1647244738 |
|AFRICA|6           |1767559760 |
|AFRICA|7           |1960219710 |
|AFRICA|8           |1809596890 |
|AFRICA|9           |276320987  |
|ASIA  |3           |529770793  |
|ASIA  |4           |1804628707 |
|ASIA  |5           |1526285399 |
|ASIA  |6           |1619482889 |
|ASIA  |7           |1768844756 |
|ASIA  |8           |1663320609 |
|ASIA  |9           |252836807  |
|CANADA|3           |144634329  |
|CANADA|4           |484552594  |
|CANADA|5           |412378365  |
|CANADA|6           |443846698  |
|CANADA|7           |477134947  |
|CANADA|8           |447073019  |
|CANADA|9           |69067959   |
|EUROPE|3           |35337093   |
|EUROPE|4           |127334255  |
|EUROPE|5           |109338389  |
|EUROPE|6           |122813826  |
|EUROPE|7           |136757466  |
|EUROPE|8           |122102995  |
|EUROPE|9           |18877433   |
|OCEANIA|3           |783282888  |
|OCEANIA|4           |2599767620 |
|OCEANIA|5           |2215657304 |
|OCEANIA|6           |2371884744 |
|OCEANIA|7           |2563459400 |
|OCEANIA|8           |2432313652 |
|OCEANIA|9           |372465518  |
|SOUTH AMERICA|3           |71023109   |
|SOUTH AMERICA|4           |238451531  |
|SOUTH AMERICA|5           |201391809  |
|SOUTH AMERICA|6           |218247455  |
|SOUTH AMERICA|7           |235582776  |
|SOUTH AMERICA|8           |221166052  |
|SOUTH AMERICA|9           |34175583   |
|USA   |3           |225353043  |
|USA   |4           |759786323  |
|USA   |5           |655967121  |
|USA   |6           |703878990  |
|USA   |7           |760331754  |
|USA   |8           |712002790  |
|USA   |9           |110532368  |

<br>

### Q5. What is the total count of transactions for each platform
```sql
SELECT platform
	, SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY platform
ORDER BY total_transactions DESC
;
```
|platform|total_transactions|
|--------|------------------|
|Retail  |1081934227        |
|Shopify |5925169           |

<br>

### Q6. What is the percentage of sales for Retail vs Shopify for each month?
```sql
WITH A AS ( SELECT platform
				, month_number
				, SUM(sales) AS sales
			FROM clean_weekly_sales
			GROUP BY platform
				, month_number
			ORDER BY platform
				, month_number
           )
           
SELECT month_number
	,  platform
	,  ROUND(
       sales
       /
       SUM(sales) OVER (PARTITION BY month_number ORDER BY month_number) * 100
       , 2) AS percentage_of_sales 
FROM A
;
```
|month_number|platform|percentage_of_sales|
|------------|--------|-------------------|
|3           |Retail  |97.54              |
|3           |Shopify |2.46               |
|4           |Retail  |97.59              |
|4           |Shopify |2.41               |
|5           |Retail  |97.30              |
|5           |Shopify |2.70               |
|6           |Retail  |97.27              |
|6           |Shopify |2.73               |
|7           |Retail  |97.29              |
|7           |Shopify |2.71               |
|8           |Retail  |97.08              |
|8           |Shopify |2.92               |
|9           |Retail  |97.38              |
|9           |Shopify |2.62               |

<br>

### Q7. What is the percentage of sales by demographic for each year in the dataset?
```sql
WITH A AS ( SELECT demographic
				, calendar_year
				, SUM(sales) AS sales
			FROM clean_weekly_sales
			GROUP BY demographic
				, calendar_year
			ORDER BY demographic
				, calendar_year
           )
           
SELECT calendar_year
	,  demographic
	,  ROUND(
       sales
       /
       SUM(sales) OVER (PARTITION BY calendar_year ORDER BY calendar_year) * 100
       , 2) AS percentage_of_sales 
FROM A
;
```
|calendar_year|demographic|percentage_of_sales|
|-------------|-----------|-------------------|
|2018         |Families   |31.99              |
|2018         |Couples    |26.38              |
|2018         |unknown    |41.63              |
|2019         |Families   |32.47              |
|2019         |Couples    |27.28              |
|2019         |unknown    |40.25              |
|2020         |Couples    |28.72              |
|2020         |unknown    |38.55              |
|2020         |Families   |32.73              |

### Q8. Which age_band and demographic values contribute the most to Retail sales?
```sql
WITH demographic_table
		
        AS (SELECT platform
            	, demographic
				, SUM(sales) AS dem_sales
			FROM clean_weekly_sales
            WHERE platform = 'Retail'
			GROUP BY demographic, platform
			),
            
      age_band_table
        AS (SELECT platform
            	, age_band
				, SUM(sales) AS age_sales
			FROM clean_weekly_sales
            WHERE platform = 'Retail'
			GROUP BY age_band, platform
			),
            
      max_sales
      	AS (SELECT demographic
                , dem_sales
                , MAX(dem_sales) OVER () as max_dem_sales
                , age_band
                , age_sales
                , MAX(age_sales) OVER () as max_age_sales		
            FROM demographic_table
            INNER JOIN age_band_table
                ON demographic_table.platform = age_band_table.platform 
            )

SELECT demographic
	, age_band
FROM max_sales
WHERE dem_sales = max_dem_sales AND age_sales = max_age_sales
;
```
|demographic|age_band|
|-----------|--------|
|unknown    |unknown |

<br>

### Q9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

```sql
SELECT calendar_year
	, platform
    , ROUND(avg(avg_transaction)) AS avg_transaction_size_using_avg_transaction    
	, ROUND(sum(sales)/sum(transactions)) AS avg_transaction_size
FROM data_mart.clean_weekly_sales
GROUP BY calendar_year
	, platform
ORDER BY calendar_year
	, platform
;
```
|calendar_year|platform|avg_transaction_size_using_avg_transaction|avg_transaction_size|
|-------------|--------|------------------------------------------|--------------------|
|2018         |Retail  |43                                        |37                  |
|2018         |Shopify |188                                       |192                 |
|2019         |Retail  |42                                        |37                  |
|2019         |Shopify |178                                       |183                 |
|2020         |Retail  |41                                        |37                  |
|2020         |Shopify |175                                       |179                 |

<br>
Note: avg_transaction column cannot be used as it will produce inaccurate values because the calculation was made at the row level. But for more accurate result we need to sum up the sales and transactions at the year and platform level and then make calaculation.

## 3. Before and After Analysis

This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.
<br>

Taking the ***week_date*** value of ***2020-06-15*** as the baseline week where the Data Mart sustainable packaging changes came into effect.
<br>

We would include all week_date values for ***2020-06-15*** as the start of the period ***after*** the change and the previous ***week_date*** values would be ***before***
<br>

Using this analysis approach - answer the following questions:

### 1. What is the total sales for the 4 weeks before and after ***2020-06-15***? What is the growth or reduction rate in actual values and percentage of sales?
```sql
WITH A AS (SELECT CASE 
		  WHEN week_date <'2020-06-15'::DATE THEN '4 Weeks Before'
		  ELSE '4 Weeks After'
           	  END AS Threshold
                , sum(sales) AS total_sales
                , sum(transactions) AS total_transaction
            FROM data_mart.clean_weekly_sales
            WHERE (week_date >= '2020-06-15'::DATE - INTERVAL '4 week') AND
		  (week_date <= '2020-06-15'::DATE + INTERVAL '3 week')
            GROUP BY week_date
            ORDER BY week_date
            ),
            
    B AS (SELECT SUM(CASE
		     WHEN threshold = '4 Weeks Before' THEN total_sales
       		     ELSE 0 
       		     END) AS "_4_weeks_before_sales"
               	, SUM(CASE
                      WHEN threshold = '4 Weeks After' THEN total_sales
                      ELSE 0 
                      END) AS "_4_weeks_after_sales"
	  FROM A
    	  )

SELECT *
    , _4_weeks_after_sales - _4_weeks_before_sales AS growth_reduction_value
    , ROUND((_4_weeks_after_sales - _4_weeks_before_sales)/_4_weeks_before_sales * 100, 2) AS growth_reduction_percent
FROM B
;
```
|_4_weeks_before_sales|_4_weeks_after_sales|growth_reduction_value|growth_reduction_percent|
|---------------------|--------------------|----------------------|------------------------|
|2345878357           |2318994169          |-26884188             |-1.15                   |

<br>


### 2. What about the entire 12 weeks before and after?
```sql
WITH A AS (SELECT CASE 
		  WHEN week_date <'2020-06-15'::DATE THEN '12 Weeks Before'
		  ELSE '12 Weeks After'
           	  END AS Threshold
                , sum(sales) AS total_sales
                , sum(transactions) AS total_transaction
            FROM data_mart.clean_weekly_sales
            WHERE (week_date >= '2020-06-15'::DATE - INTERVAL '12 week') AND
		  (week_date <= '2020-06-15'::DATE + INTERVAL '11 week')
            GROUP BY week_date
            ORDER BY week_date
            ),
            
    B AS (SELECT SUM(CASE
		     WHEN threshold = '12 Weeks Before' THEN total_sales
       		     ELSE 0 
       		     END) AS "_12_weeks_before_sales"
               	, SUM(CASE
                      WHEN threshold = '12 Weeks After' THEN total_sales
                      ELSE 0 
                      END) AS "_12_weeks_after_sales"
	  FROM A
    	  )

SELECT *
    , _12_weeks_after_sales - _12_weeks_before_sales AS growth_reduction_value
    , ROUND((_12_weeks_after_sales - _12_weeks_before_sales)/_12_weeks_before_sales * 100, 2) AS growth_reduction_percent
FROM B
;
```
|_12_weeks_before_sales|_12_weeks_after_sales|growth_reduction_value|growth_reduction_percent|
|----------------------|---------------------|----------------------|------------------------|
|7126273147            |6973947753           |-152325394            |-2.14                   |

<br>

### 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

#### 4 weeks before and after
```sql
WITH A AS (SELECT calendar_year
		, week_number
		, CASE 
		  WHEN week_number < 25 THEN '4 Weeks Before'
		  ELSE '4 Weeks After'
		  END AS Threshold
                , sum(sales) AS total_sales
                , sum(transactions) AS total_transaction
            FROM data_mart.clean_weekly_sales
            WHERE week_number >= (25 - 4) AND (week_number <= (25 + 3))
            GROUP BY week_date, week_number, calendar_year
            ORDER BY week_date
            ),
            
    B AS (SELECT calendar_year
		, SUM(CASE
		  WHEN threshold = '4 Weeks Before' THEN total_sales
		  ELSE 0 
		  END) AS "_4_weeks_before_sales"
               	, SUM(CASE
                  WHEN threshold = '4 Weeks After' THEN total_sales
                  ELSE 0 
                  END) AS "_4_weeks_after_sales"
	   FROM A
           GROUP BY calendar_year
           )

SELECT *
    , _4_weeks_after_sales - _4_weeks_before_sales AS growth_reduction_value
    , ROUND((_4_weeks_after_sales - _4_weeks_before_sales)/_4_weeks_before_sales * 100, 2) AS growth_reduction_percent
FROM B
;
```
|calendar_year|_4_weeks_before_sales|_4_weeks_after_sales|growth_reduction_value|growth_reduction_percent|
|-------------|---------------------|--------------------|----------------------|------------------------|
|2018         |2125140809           |2129242914          |4102105               |0.19                    |
|2019         |2249989796           |2252326390          |2336594               |0.10                    |
|2020         |2345878357           |2318994169          |-26884188             |-1.15                   |

<br>

#### 12 weeks before and after
```sql
WITH A AS (SELECT calendar_year
		, week_number
		, CASE 
		  WHEN week_number < 25 THEN '12 Weeks Before'
		  ELSE '12 Weeks After'
		  END AS Threshold
                , sum(sales) AS total_sales
                , sum(transactions) AS total_transaction
            FROM data_mart.clean_weekly_sales
            WHERE week_number >= (25 - 12) AND (week_number <= (25 + 11))
            GROUP BY week_date, week_number, calendar_year
            ORDER BY week_date
            ),
            
    B AS (SELECT calendar_year
		, SUM(CASE
		  WHEN threshold = '12 Weeks Before' THEN total_sales
		  ELSE 0 
		  END) AS "_12_weeks_before_sales"
               	, SUM(CASE
                  WHEN threshold = '12 Weeks After' THEN total_sales
                  ELSE 0 
                  END) AS "_12_weeks_after_sales"
	   FROM A
           GROUP BY calendar_year
           )

SELECT *
    , _12_weeks_after_sales - _12_weeks_before_sales AS growth_reduction_value
    , ROUND((_12_weeks_after_sales - _12_weeks_before_sales)/_12_weeks_before_sales * 100, 2) AS growth_reduction_percent
FROM B
;
```
|calendar_year|_12_weeks_before_sales|_12_weeks_after_sales|growth_reduction_value|growth_reduction_percent|
|-------------|----------------------|---------------------|----------------------|------------------------|
|2018         |6396562317            |6500818510           |104256193             |1.63                    |
|2019         |6883386397            |6862646103           |-20740294             |-0.30                   |
|2020         |7126273147            |6973947753           |-152325394            |-2.14                   |

<br>

## 4. Bonus Questions

Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

- region
- platform
- age_band
- demographic
- customer_type

#### Region
```sql
WITH A AS (SELECT region
		, week_number
		, CASE 
		  WHEN week_number < 25 THEN '12 Weeks Before'
		  ELSE '12 Weeks After'
		  END AS Threshold
                , sum(sales) AS total_sales
                , sum(transactions) AS total_transaction
            FROM data_mart.clean_weekly_sales
            WHERE week_number >= (25 - 12) AND (week_number <= (25 + 11))
            GROUP BY week_date, week_number, region
            ORDER BY week_date
            ),
            
    B AS (SELECT region
		, SUM(CASE
		  WHEN threshold = '12 Weeks Before' THEN total_sales
		  ELSE 0 
		  END) AS "_12_weeks_before_sales"
               	, SUM(CASE
                  WHEN threshold = '12 Weeks After' THEN total_sales
                  ELSE 0 
                  END) AS "_12_weeks_after_sales"
	   FROM A
           GROUP BY region
           )

SELECT region
    , _12_weeks_after_sales - _12_weeks_before_sales AS growth_reduction_value
    , ROUND((_12_weeks_after_sales - _12_weeks_before_sales)/_12_weeks_before_sales * 100, 2) AS growth_reduction_percent
FROM B
ORDER BY growth_reduction_percent
;
```
|region|growth_reduction_value|growth_reduction_percent|
|------|----------------------|------------------------|
|ASIA  |-61315418             |-1.33                   |
|OCEANIA|-58341540             |-0.87                   |
|CANADA|-10637499             |-0.85                   |
|USA   |-7257385              |-0.37                   |
|SOUTH AMERICA|-2075531              |-0.34                   |
|AFRICA|54539249              |1.10                    |
|EUROPE|16278629              |4.96                    |

<br>

Asia has the highest negative impact with 1.33% reduction in sales. On the other hand Europe sales increased by 4.96%.

<br>

#### Platform
```sql
WITH A AS (SELECT platform
		, week_number
		, CASE 
		  WHEN week_number < 25 THEN '12 Weeks Before'
		  ELSE '12 Weeks After'
		  END AS Threshold
                , sum(sales) AS total_sales
                , sum(transactions) AS total_transaction
            FROM data_mart.clean_weekly_sales
            WHERE week_number >= (25 - 12) AND (week_number <= (25 + 11))
            GROUP BY week_date, week_number, platform
            ORDER BY week_date
            ),
            
    B AS (SELECT platform
		, SUM(CASE
		  WHEN threshold = '12 Weeks Before' THEN total_sales
		  ELSE 0 
		  END) AS "_12_weeks_before_sales"
               	, SUM(CASE
                  WHEN threshold = '12 Weeks After' THEN total_sales
                  ELSE 0 
                  END) AS "_12_weeks_after_sales"
	   FROM A
           GROUP BY platform
           )

SELECT platform
    , _12_weeks_after_sales - _12_weeks_before_sales AS growth_reduction_value
    , ROUND((_12_weeks_after_sales - _12_weeks_before_sales)/_12_weeks_before_sales * 100, 2) AS growth_reduction_percent
FROM B
ORDER BY growth_reduction_percent
;
```

|platform|growth_reduction_value|growth_reduction_percent|
|--------|----------------------|------------------------|
|Retail  |-117464107            |-0.59                   |
|Shopify |48654612              |9.35                    |

<br>

Retail sales decresead by 0.59%. Increase in Shopify by 9.35%.

<br>

#### Age Band
```sql
WITH A AS (SELECT age_band
		, week_number
		, CASE 
		  WHEN week_number < 25 THEN '12 Weeks Before'
		  ELSE '12 Weeks After'
		  END AS Threshold
                , sum(sales) AS total_sales
                , sum(transactions) AS total_transaction
            FROM data_mart.clean_weekly_sales
            WHERE week_number >= (25 - 12) AND (week_number <= (25 + 11))
            GROUP BY week_date, week_number, age_band
            ORDER BY week_date
            ),
            
    B AS (SELECT age_band
		, SUM(CASE
		  WHEN threshold = '12 Weeks Before' THEN total_sales
		  ELSE 0 
		  END) AS "_12_weeks_before_sales"
               	, SUM(CASE
                  WHEN threshold = '12 Weeks After' THEN total_sales
                  ELSE 0 
                  END) AS "_12_weeks_after_sales"
	   FROM A
           GROUP BY age_band
           )

SELECT age_band
    , _12_weeks_after_sales - _12_weeks_before_sales AS growth_reduction_value
    , ROUND((_12_weeks_after_sales - _12_weeks_before_sales)/_12_weeks_before_sales * 100, 2) AS growth_reduction_percent
FROM B
ORDER BY growth_reduction_percent
;
```
|age_band|growth_reduction_value|growth_reduction_percent|
|--------|----------------------|------------------------|
|unknown |-44645418             |-0.55                   |
|Middle Aged|-7143725              |-0.22                   |
|Young Adults|-4861910              |-0.21                   |
|Retirees|-12158442             |-0.18                   |

<br>

Sligthy affected all band age groups in similar proportion of decrease percenta sales of about 0.2%. 

<br>

#### Demographic
```sql
WITH A AS (SELECT demographic
		, week_number
		, CASE 
		  WHEN week_number < 25 THEN '12 Weeks Before'
		  ELSE '12 Weeks After'
		  END AS Threshold
                , sum(sales) AS total_sales
                , sum(transactions) AS total_transaction
            FROM data_mart.clean_weekly_sales
            WHERE week_number >= (25 - 12) AND (week_number <= (25 + 11))
            GROUP BY week_date, week_number, demographic
            ORDER BY week_date
            ),
            
    B AS (SELECT demographic
		, SUM(CASE
		  WHEN threshold = '12 Weeks Before' THEN total_sales
		  ELSE 0 
		  END) AS "_12_weeks_before_sales"
               	, SUM(CASE
                  WHEN threshold = '12 Weeks After' THEN total_sales
                  ELSE 0 
                  END) AS "_12_weeks_after_sales"
	   FROM A
           GROUP BY demographic
           )

SELECT demographic
    , _12_weeks_after_sales - _12_weeks_before_sales AS growth_reduction_value
    , ROUND((_12_weeks_after_sales - _12_weeks_before_sales)/_12_weeks_before_sales * 100, 2) AS growth_reduction_percent
FROM B
ORDER BY growth_reduction_percent
;
```
|demographic|growth_reduction_value|growth_reduction_percent|
|-----------|----------------------|------------------------|
|unknown    |-44645418             |-0.55                   |
|Couples    |-16524711             |-0.29                   |
|Families   |-7639366              |-0.12                   |

<br>

Sligthy affected both demographuc groups.

<br>

#### Customer Type

```sql
WITH A AS (SELECT customer_type
		, week_number
		, CASE 
		  WHEN week_number < 25 THEN '12 Weeks Before'
		  ELSE '12 Weeks After'
		  END AS Threshold
                , sum(sales) AS total_sales
                , sum(transactions) AS total_transaction
            FROM data_mart.clean_weekly_sales
            WHERE week_number >= (25 - 12) AND (week_number <= (25 + 11))
            GROUP BY week_date, week_number, customer_type
            ORDER BY week_date
            ),
            
    B AS (SELECT customer_type
		, SUM(CASE
		  WHEN threshold = '12 Weeks Before' THEN total_sales
		  ELSE 0 
		  END) AS "_12_weeks_before_sales"
               	, SUM(CASE
                  WHEN threshold = '12 Weeks After' THEN total_sales
                  ELSE 0 
                  END) AS "_12_weeks_after_sales"
	   FROM A
           GROUP BY customer_type
           )

SELECT customer_type
    , _12_weeks_after_sales - _12_weeks_before_sales AS growth_reduction_value
    , ROUND((_12_weeks_after_sales - _12_weeks_before_sales)/_12_weeks_before_sales * 100, 2) AS growth_reduction_percent
FROM B
ORDER BY growth_reduction_percent
;
```
|customer_type|growth_reduction_value|growth_reduction_percent|
|-------------|----------------------|------------------------|
|Existing     |-51510403             |-0.51                   |
|Guest        |-35202995             |-0.46                   |
|New          |17903903              |0.69                    |

<br>

The change affected the sales in existing customer and guest category with reduction in sale by 0.51%. The sales increased in new customery category by 0.69%

<br>







