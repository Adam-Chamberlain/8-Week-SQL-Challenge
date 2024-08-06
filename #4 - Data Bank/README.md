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
WITH cte AS (SELECT
customer_id,
MONTH(txn_date) AS monthnum,
SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) +
SUM(CASE WHEN txn_type IN('purchase', 'withdrawal') THEN (-1 * txn_amount) ELSE 0 END) AS total
FROM customer_transactions
GROUP BY customer_id, monthnum
ORDER BY customer_id, monthnum)

SELECT
customer_id,
monthnum,
CASE WHEN monthnum = 1 THEN '1/31/2020'
WHEN monthnum = 2 THEN '2/29/2020'
WHEN monthnum = 3 THEN '3/31/2020'
WHEN monthnum = 4 THEN '4/30/2020' ELSE NULL END AS endmonth,
total
FROM cte
```

Work in Progress

### 5. What is the percentage of customers who increase their closing balance by more than 5%?
## C. Data Allocation Challenge
To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

Option 1: data is allocated based off the amount of money at the end of the previous month
Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
Option 3: data is updated real-time
For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

running customer balance column that includes the impact each transaction
customer balance at the end of each month
minimum, average and maximum values of the running balance for each customer
Using all of the data available - how much data would have been required for each option on a monthly basis?
