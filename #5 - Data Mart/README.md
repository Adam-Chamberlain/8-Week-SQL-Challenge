# [Case Study #5: Data Mart](https://8weeksqlchallenge.com/case-study-5/)

This case study looks at an online supermarket that has data from 2018 through 2020 within one large table. The table needs to be cleaned quite a bit, and once that is done, there are many questions based on the contents of the data, especially with identifying how a business change that took place in June of 2020 affected sales.

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

```sql
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

```sql
SELECT DISTINCT
WEEKDAY(week_date) AS wkday
FROM clean
```
This only returns the value 0, meaning it is only showing Mondays. The dates with numbers 1-6 (Tuesday through Sunday) is missing.

### 3. How many total transactions were there for each year in the dataset?

```sql
SELECT
calendar_year,
SUM(transactions) AS count
FROM clean
GROUP BY calendar_year
```
![image](https://github.com/user-attachments/assets/70aa369a-e715-4da1-aab1-238cfcedf7d5)

### 4. What is the total sales for each region for each month?

```sql
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

```sql
SELECT
platform,
SUM(transactions) AS total_transactions
FROM clean
GROUP BY platform
```
![image](https://github.com/user-attachments/assets/2b33410a-516c-464c-b47b-df7595a59616)

### 6. What is the percentage of sales for Retail vs Shopify for each month?

```sql
cte AS (
  SELECT
    c.calendar_year,
    c.month_number,
    SUM(sales) AS total_sales,
    r_sales,
    s_sales
  FROM clean c
  JOIN (
    SELECT
      calendar_year,
      month_number,
      SUM(sales) AS r_sales
    FROM clean
    WHERE platform = 'Retail'
    GROUP BY calendar_year, month_number) r
  ON c.calendar_year = r.calendar_year AND c.month_number = r.month_number
  JOIN (
    SELECT
      calendar_year,
      month_number,
      SUM(sales) AS s_sales
    FROM clean
    WHERE platform = 'Shopify'
    GROUP BY calendar_year, month_number) s
  ON c.calendar_year = s.calendar_year AND c.month_number = s.month_number
  GROUP BY c.calendar_year, c.month_number, r_sales, s_sales)

SELECT
  calendar_year,
  month_number,
  ROUND(r_sales / total_sales * 100, 2) AS retail,
  ROUND(s_sales / total_sales * 100, 2) AS shopify
FROM cte
ORDER BY calendar_year, month_number
```
This CTE pulls sales data from each month in each year within each platform and puts it all in one table so that it can easily be divided to find the percentages. The data goes up to August 2020, but only 2018/19 are shown on the below image.

![image](https://github.com/user-attachments/assets/3154609b-da18-4934-8141-3800071a7d1a)

### 7. What is the percentage of sales by demographic for each year in the dataset?

```sql
cte AS (
  SELECT
    cl.calendar_year,
    SUM(sales) AS total_sales,
    c_sales,
    f_sales
  FROM clean cl
  JOIN (
    SELECT
      calendar_year,
      SUM(sales) AS c_sales
    FROM clean
    WHERE demographic = 'Couples'
    GROUP BY calendar_year) c
  ON cl.calendar_year = c.calendar_year
  JOIN (
    SELECT
      calendar_year,
      SUM(sales) AS f_sales
    FROM clean
    WHERE demographic = 'Families'
    GROUP BY calendar_year) f
  ON cl.calendar_year = f.calendar_year
  GROUP BY cl.calendar_year, c_sales, f_sales)
  
  SELECT
  calendar_year,
  ROUND(c_sales / total_sales * 100, 2) AS couples,
  ROUND(f_sales / total_sales * 100, 2) AS families,
  ROUND((1 - ((c_sales + f_sales) / total_sales)) * 100, 2) AS unknown
  FROM cte
  ORDER BY calendar_year
```
I used a similar solution to question 6 for this one. I also calculated the percentage that is unknown, meaning they are not identified as couples or families.

![image](https://github.com/user-attachments/assets/99daf389-340b-4dad-ad88-b3a67320f352)

### 8. Which age_band and demographic values contribute the most to Retail sales?

```sql
SELECT
age_band,
demographic,
SUM(sales) AS total
FROM clean
WHERE platform = 'Retail'
GROUP BY age_band, demographic
ORDER BY total DESC
```
![image](https://github.com/user-attachments/assets/351bd59d-487c-436a-87b8-4cafe849c8a2)

### 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

```sql
SELECT
calendar_year,
platform,
ROUND(AVG(avg_transaction), 2) AS average1,
ROUND(SUM(sales) / SUM(transactions), 2) AS average2
FROM clean
GROUP BY calendar_year, platform
ORDER BY calendar_year, platform
```
`average1` uses the `avg_transaction` column, and `average2` uses an alternative way. The results are slightly difference because 1 takes the average from each row and finds the average of the averages, while 2 sums up the totals and finds the average that way. Option 2 is more accurate because of this.

![image](https://github.com/user-attachments/assets/87df4cb3-9862-44aa-bf29-b98f79f904ec)

## 3. Before & After Analysis
This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.

Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before

Using this analysis approach - answer the following questions:

### 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?

```sql
data AS (
SELECT
SUM(CASE WHEN week_number BETWEEN 20 AND 23 THEN sales ELSE NULL END) AS before_change,
SUM(CASE WHEN week_number BETWEEN 25 AND 28 THEN sales ELSE NULL END) AS after_change
FROM clean
WHERE calendar_year = 2020)

SELECT
after_change - before_change AS difference,
ROUND(100 * (after_change - before_change) / before_change, 2) AS percent
FROM data
```
The week of June 15 is Week 24, so for this, 20-23 are the four weeks before, and 25-28 are the four weeks after. I basically find the total number of sales for each periods and calculate the difference and percentage difference between the two.

![image](https://github.com/user-attachments/assets/aeb56a6a-ae81-4062-bebf-b6c61cc3c4af)

### 2. What about the entire 12 weeks before and after?

```sql
data AS (
SELECT
SUM(CASE WHEN week_number BETWEEN 12 AND 23 THEN sales ELSE NULL END) AS before_change,
SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN sales ELSE NULL END) AS after_change
FROM clean
WHERE calendar_year = 2020)

SELECT
after_change - before_change AS difference,
ROUND(100 * (after_change - before_change) / before_change, 2) AS percent
FROM data
```
I used the same query with different week ranges to answer this question. There is a much larger difference in these two ranges.

![image](https://github.com/user-attachments/assets/52afdc6c-d696-4d26-a186-bb59b42c0841)

### 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

Again I used a similar query, but instead of `WHERE calendar_year = 2020` I added `calendar_year` as a column and grouped the sums by each year. There is a large drop in sales each year when looking at 12 weeks before and after, but it has grown by roughly 2% each year. Looking at 4 weeks before and after, 2020 was the first year it went in the negative.

**4 weeks Before and After**

![image](https://github.com/user-attachments/assets/beded384-b5d6-451f-bb27-520fc604ff0e)

**12 weeks Before and After**

![image](https://github.com/user-attachments/assets/01c0ddb1-04d3-468e-8c67-0f3ca576f843)

## 4. Bonus Question
Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

```sql
data AS (
SELECT
region,
SUM(CASE WHEN week_number BETWEEN 12 AND 23 THEN sales ELSE NULL END) AS before_change,
SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN sales ELSE NULL END) AS after_change
FROM clean
WHERE calendar_year = 2020
GROUP BY region)

SELECT
region,
after_change - before_change AS difference,
ROUND(100 * (after_change - before_change) / before_change, 2) AS percent
FROM data
ORDER BY difference
```
Here, I use the same base query for each area, with the `region` column being swapped out for various columns. 

- region

![image](https://github.com/user-attachments/assets/4c9cdf4d-cd3c-4c13-84c1-fb9724a50776)

- platform

![image](https://github.com/user-attachments/assets/0f3af52d-70c9-474c-aee1-36fac7bd2b5e)

- age_band

![image](https://github.com/user-attachments/assets/177cfa20-84e0-4137-bf93-501605bcf391)

- demographic

![image](https://github.com/user-attachments/assets/c725a0a8-5ced-434e-a4f1-67c845992a65)

- customer_type

![image](https://github.com/user-attachments/assets/0d6a1c08-ec39-452a-bd01-2093534df61f)

