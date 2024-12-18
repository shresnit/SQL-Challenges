# Case Study #4 - Data Mart
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





















