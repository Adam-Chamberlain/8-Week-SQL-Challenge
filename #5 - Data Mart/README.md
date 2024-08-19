# [Case Study #5: Data Mart](https://8weeksqlchallenge.com/case-study-5/)

## 1. Data Cleansing Steps
In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:

- Convert the week_date to a DATE format
- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a month_number with the calendar month for each week_date value as the 3rd column
- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value
- Add a new demographic column using the following mapping for the first letter in the segment values:
- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record

```
SELECT
  clean_date AS week_date,
  WEEK(clean_date) AS week_number,
  MONTH(clean_date) AS month_number,
  YEAR(clean_date) AS calendar_year,
  region,
  platform,
  CASE WHEN segment LIKE 'null' THEN NULL ELSE segment END AS segment,
  CASE WHEN RIGHT(segment, 1) = 1 THEN 'Young Adults'
    WHEN RIGHT(segment, 1) = 2 THEN 'Middle Aged'
    WHEN RIGHT(segment, 1) IN(3, 4) THEN 'Retirees' ELSE NULL END AS age_band,
  CASE WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
    WHEN LEFT(segment, 1) = 'F' THEN 'Families' ELSE NULL END AS demographic,
  customer_type,
  transactions,
  sales,
  ROUND(sales / transactions, 2) AS avg_transaction
FROM (
  SELECT *,
    CAST(CONCAT(
      RIGHT(week_date, 2), '/',
      CASE WHEN MID(week_date, 3, 1) = '/' AND MID(week_date, 6, 1) = '/' THEN MID(week_date, 4, 2)
      WHEN MID(week_date, 3, 1) = '/' AND MID(week_date, 5, 1) = '/' THEN MID(week_date, 4, 1)
      WHEN MID(week_date, 2, 1) = '/' AND MID(week_date, 5, 1) = '/' THEN MID(week_date, 3, 2)
      ELSE MID(week_date, 3, 1) END, '/',
      CASE WHEN MID(week_date, 3, 1) = '/' THEN LEFT(week_date, 2) ELSE LEFT(week_date, 1) END)
    AS DATE) AS clean_date
  FROM weekly_sales) clean
```

To clean the `week_date` column, I used a CONCAT to rearrange it. Since the length of the strings were inconsistent, I used CASE statements to identify how many numbers were in the month and date by finding where the slashes were, and I pulled specific characters from those results.

For all future questions, I use this cleaned data as a CTE named `clean`.

![image](https://github.com/user-attachments/assets/32ab7187-216a-494e-a3ba-caa35e261fd3)


## 2. Data Exploration
### 1. What day of the week is used for each week_date value?
### 2. What range of week numbers are missing from the dataset?

```
SELECT DISTINCT
WEEKDAY(week_date) AS wkday
FROM clean
```
This only returns the value 0, meaning it is only showing Mondays. The dates with numbers 1-6 (Tuesday through Sunday) is missing.

### 3. How many total transactions were there for each year in the dataset?

```
SELECT
calendar_year,
SUM(transactions) AS count
FROM clean
GROUP BY calendar_year
```
![image](https://github.com/user-attachments/assets/70aa369a-e715-4da1-aab1-238cfcedf7d5)

### 4. What is the total sales for each region for each month?

```
SELECT
month_number,
region,
SUM(sales) AS total_sales
FROM clean
GROUP BY region, month_number
ORDER BY month_number, region
```
The data goes from March (3) to September (9) in years 2018 to 2020, but 2020 only goes to August. The image just shows the first few months.

![image](https://github.com/user-attachments/assets/ceab6f1f-1282-42d3-95eb-3ae6d29ab82b)

### 5. What is the total count of transactions for each platform

```
SELECT
platform,
SUM(transactions) AS total_transactions
FROM clean
GROUP BY platform
```
![image](https://github.com/user-attachments/assets/2b33410a-516c-464c-b47b-df7595a59616)

### 6. What is the percentage of sales for Retail vs Shopify for each month?
### 7. What is the percentage of sales by demographic for each year in the dataset?
### 8. Which age_band and demographic values contribute the most to Retail sales?
### 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

## 3. Before & After Analysis
This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.

Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before

Using this analysis approach - answer the following questions:

What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
What about the entire 12 weeks before and after?
How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
4. Bonus Question
Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

region
platform
age_band
demographic
customer_type
Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?
