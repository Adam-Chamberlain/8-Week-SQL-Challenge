# [Case Study #4: Data Bank](https://8weeksqlchallenge.com/case-study-4/)

This case study looks at a digital bank that stores money as well as data. Customers are given a specific amount of cloud storage depending on how much money they have stored within the bank. This set of data uses three tables: `Regions`, which pairs region names with region IDs, `Customer Nodes`, which lists all customers with their respective regions and their nodes, which are randomly distributed, and `Customer Transactions`, which lists all transactions that have taken place for each customer, including deposits, withdrawals, and purchases.

### Entity Relationship Diagram
![case-study-4-erd](https://github.com/user-attachments/assets/01be7bc7-1389-4c91-9eb8-cc3b086d060e)


## A. Customer Nodes Exploration
### 1. How many unique nodes are there on the Data Bank system?

```
SELECT
  COUNT(DISTINCT node_id) AS node_count
FROM customer_nodes
```
|node_count|
|---|
|5|

### 2.What is the number of nodes per region?

```
SELECT
region_name,
COUNT(DISTINCT node_id) AS nodes
FROM regions r
JOIN customer_nodes c
ON r.region_id = c.region_id
GROUP BY region_name
```

|region_name|nodes|
|---|---|
|Africa|5|
|America|5|
|Asia|5|
|Australia|5|
|Europe|5|

### 3. How many customers are allocated to each region?

```
SELECT
region_name,
COUNT(customer_id) AS customers
FROM regions r
JOIN customer_nodes c
ON r.region_id = c.region_id
GROUP BY region_name
ORDER BY region_name
```

|region_name|customers|
|---|---|
|Africa|714|
|America|735|
|Asia|665|
|Australia|770|
|Europe|616|

### 4. How many days on average are customers reallocated to a different node?

```
WITH cte AS ( SELECT
customer_id,
node_id,
DATEDIFF(end_date, start_date) AS diff
FROM customer_nodes
WHERE end_date <> '9999-12-31'
ORDER BY customer_id)

SELECT
ROUND(AVG(diff),0) AS average
FROM cte
```

|average|
|---|
|15|

### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

```
WITH cte AS (SELECT
ROW_NUMBER() OVER() AS rownumber,
customer_id,
node_id,
DATEDIFF(end_date, start_date) AS diff
FROM customer_nodes
WHERE end_date <> '9999-12-31'
ORDER BY diff)

SELECT
diff AS median
FROM cte
WHERE rownumber = (SELECT ROUND(COUNT(*)/2,0) FROM cte)
```
Since there's no median function, I assigned a row number to each row, organized from shortest to longest amount of reallocation days, and then pulled the number from the row exactly in the middle.

|median|
|---|
|15|

For the percentile values, the final WHERE clause can be changed like this: 

- 80th: `WHERE rownumber = (SELECT ROUND(COUNT(*)*0.8,0) FROM cte)` - **23**
- 95th: `WHERE rownumber = (SELECT ROUND(COUNT(*)*0.95,0) FROM cte)` - **28**

## B. Customer Transactions
### 1. What is the unique count and total amount for each transaction type?

```
SELECT
txn_type,
COUNT(txn_type) AS amount
FROM customer_transactions
GROUP BY txn_type
```

|txn_type|amount|
|---|---|
|deposit|2671|
|withdrawal|1580|
|purchase|1617|

### 2. What is the average total historical deposit counts and amounts for all customers?

```
WITH cte AS (
SELECT
  customer_id,
  COUNT(txn_type) AS type,
  AVG(txn_amount) AS amount
FROM customer_transactions
WHERE txn_type = 'deposit'
GROUP BY customer_id)

SELECT
  ROUND(AVG(type),0) AS avg_count,
  ROUND(AVG(amount),0) AS avg_amount
FROM cte
```

|avg_count|avg_amount|
|---|---|
|5|509|

### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

```
WITH cte AS (SELECT
MONTH(txn_date) AS monthnum,
customer_id,
SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposits,
SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchases,
SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawals
FROM customer_transactions
GROUP BY monthnum, customer_id)

SELECT
monthnum,
COUNT(customer_id) AS amount
FROM cte
WHERE deposits > 1 AND (purchases >= 1 OR withdrawals >= 1)
GROUP BY monthnum
ORDER BY monthnum
```
This separates deposits, purchases, and withdrawals to separate columns so that they can be counted for each customer in each month. I can then use WHERE to look for specific instances where customers make more than one deposit and at least one purchase or withdrawal within a month.

|monthnum|amount|
|---|---|
|1|168|
|2|181|
|3|192|
|4|70|

### 4. What is the closing balance for each customer at the end of the month?

```
CREATE TABLE temp_table (
num INT PRIMARY KEY,
monthdate DATE);

INSERT INTO temp_table
  (num, monthdate)
VALUES
  (1, '2020-01-31'),
  (2, '2020-02-29'),
  (3, '2020-03-31'),
  (4, '2020-04-30');
```

I first created a table to identify end dates and create a row for each month for each customer id, even if they did not have any transactions in a specific month. With this, I used many CTE tables to complete this question.

```
WITH cte AS (
SELECT
  customer_id,
  MONTH(txn_date) AS monthnum,
  SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) +
  SUM(CASE WHEN txn_type IN('purchase', 'withdrawal') THEN (-1 * txn_amount) ELSE 0 END) AS total
FROM customer_transactions
GROUP BY customer_id, monthnum
ORDER BY customer_id, monthnum),
```
This CTE pulls the total for each customer in each month they made transactions. Months where they did not make transactions are not included yet.

```
alldates AS (
SELECT DISTINCT
  customer_id,
  num AS monthnum,
  monthnum AS num,
  monthdate,
  total
FROM cte, temp_table
ORDER BY customer_id, num),
```
This merges the prior CTE with the temporary table to create the below table. Although it looks messy, the next CTE cleans it up further. (`monthnum` and `num` are also swapped to be more accurate for the remainder of the query; `monthnum` now corresponds with `monthdate`)

![image](https://github.com/user-attachments/assets/a64a6bee-c6ea-4d01-9e53-acf1ef74d825)

```
clean AS (
SELECT
  ad.customer_id,
  ad.monthnum,
  ad.num,
  ad.monthdate AS end_month,
  c.total,
  SUM(ad.total) OVER (PARTITION BY ad.customer_id, ad.monthnum) AS end_balance
FROM alldates ad
LEFT JOIN cte c
  ON ad.monthnum = c.monthnum AND ad.customer_id = c.customer_id
WHERE ad.monthnum >= ad.num)
```
This CTE does lots of things. First, it joins the two prior CTEs to get an accurate total for each month, as seen in the new `total` column. It is also filtered to remove instances where `monthnum` is less than `num` so that totals that have not taken place yet do not show. This allows the PARTITION BY statement to accurately show the total for that specific month.

(For example: For customer 1 in Month 3, it pulls the total values from the prior table that corresponds with the right values. It finds 312 and -952, so they are added to make 640. In Month 2, however, the WHERE clause removes the row with -952, since that had not taken place yet, so the total for Month 2 is 312.

![image](https://github.com/user-attachments/assets/fd02defe-c7bf-4020-9fc5-9bda6e750a58)

```
SELECT DISTINCT
  customer_id,
  end_month,
  CASE WHEN total IS NULL THEN 0 ELSE total END AS monthly_change,
  end_balance
FROM clean
```
Finally, duplicate values are filtered out so that each customer only has one row per month.

![image](https://github.com/user-attachments/assets/ced0741e-9165-4cd4-bfad-c48ea8e9d1e1)

### 5. What is the percentage of customers who increase their closing balance by more than 5%?

```
final AS (SELECT DISTINCT
  a.customer_id,
  a.end_balance AS jan,
  b.end_balance AS apr,
  b.end_balance - a.end_balance AS difference,
  ROUND(CASE WHEN a.end_balance < 0 THEN -1 ELSE 1 END *
    (b.end_balance / a.end_balance -1) * 100, 2) AS percent
FROM (SELECT * FROM clean WHERE monthnum = 1) a
JOIN (SELECT * FROM clean WHERE monthnum = 4) b
  ON a.customer_id = b.customer_id)

SELECT
  ROUND((SELECT COUNT(percent) FROM final WHERE percent >= 5) / COUNT(percent) * 100,2) AS percent
FROM final
```
Using all of the CTEs from Question 4, I was able to solve this with just one more CTE. Basically, it allows me to pull the percent difference between January's balance and April's balance. I did this by merging the same table with itself to put each month's balance on separate rows so that they could be compared.

![image](https://github.com/user-attachments/assets/d6cf8ed3-4ee5-4008-b705-d1e0bfc4012c)

With that, I could simply filter out instances where the percentage was less than five, divide it by the total, and get the answer.

|percent|
|---|
|33.20|

## C. Data Allocation Challenge
To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

- Option 1: data is allocated based off the amount of money at the end of the previous month
- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
- Option 3: data is updated real-time

For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

- running customer balance column that includes the impact each transaction
- customer balance at the end of each month
- minimum, average and maximum values of the running balance for each customer

Using all of the data available - how much data would have been required for each option on a monthly basis?

## D. Extra Challenge
Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?

Special notes:
- Data Bank wants an initial calculation which does not allow for compounding interest, however they may also be interested in a daily compounding interest calculation so you can try to perform this calculation if you have the stamina!

I may return to these last two questions at a later point. I am instead prioritizing other projects and the remaining case studies.
