# [Case Study #7: Balanced Tree Clothing Co.](https://8weeksqlchallenge.com/case-study-7/)

## A. High Level Sales Analysis
### 1. What was the total quantity sold for all products?

```sql
SELECT
  product_name,
  SUM(qty) AS amount
FROM sales s
JOIN product_details pd
  ON s.prod_id = pd.product_id
GROUP BY product_name
```
![image](https://github.com/user-attachments/assets/c0015d17-8330-4817-b5bd-b191f670a32d)

### 2. What is the total generated revenue for all products before discounts?

```sql
SELECT
  product_name,
  SUM(qty * s.price) AS amount
FROM sales s
JOIN product_details pd
  ON s.prod_id = pd.product_id
GROUP BY product_name
```
Same as question 1 but also multiplying quantity by price before summing them up.

![image](https://github.com/user-attachments/assets/bfa1ee51-5b4f-412b-802c-51c4e18d96fc)

### 3. What was the total discount amount for all products?

```sql
SELECT
  product_name,
  ROUND(SUM(qty * s.price * discount/100), 2) AS amount
FROM sales s
JOIN product_details pd
  ON s.prod_id = pd.product_id
GROUP BY product_name
```
Same as question 2, but the total is now multiplied by the percent to show how much money is being lost from discounts.

![image](https://github.com/user-attachments/assets/fc397392-3447-4cbd-a93b-d434bd49a89e)

## B. Transaction Analysis
### 1. How many unique transactions were there?

```sql
SELECT
  COUNT(DISTINCT txn_id) AS transactions
FROM sales
```
![image](https://github.com/user-attachments/assets/2ccf8524-3dd6-4c54-9fed-8e5870fb0e6f)

### 2. What is the average unique products purchased in each transaction?

```sql
SELECT
  COUNT(txn_id) / COUNT(DISTINCT txn_id) AS average
FROM sales
```
Since each row with the same ID has different products, I can count the total number of rows and divide it by the number of unique IDs in the table.

![image](https://github.com/user-attachments/assets/57c4cfad-afc1-4290-9c24-c1332dc68ea6)

### 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?

```sql
WITH sorted AS (
  SELECT
    ROW_NUMBER() OVER() AS rownum,
    txn_id,
    total
  FROM (
    SELECT
      txn_id,
      SUM(qty * price) AS total
    FROM sales
  GROUP BY txn_id
  ORDER BY total) a)

SELECT
  (SELECT total FROM sorted WHERE rownum = ROUND(COUNT(*)*0.25,0)) AS 25th,
  (SELECT total FROM sorted WHERE rownum = ROUND(COUNT(*)*0.5,0)) AS 50th,
  (SELECT total FROM sorted WHERE rownum = ROUND(COUNT(*)*0.75,0)) AS 75th
FROM sorted
```
For this, I am assuming discounts are not included in revenue. The CTE sorts all transactions by revenue and numbers them. This allows the main query to identify specific rows by taking the total amount of rows and multiplying it by specific percentages. For example, 2,500 (the total amount of rows) multiplied by 50% gives 1,250, so it pulls the 1,250th row.

![image](https://github.com/user-attachments/assets/33c5d7ba-7b8b-4dd8-ab91-5b1ebbfe4061)

### 4. What is the average discount value per transaction?

```sql
SELECT
ROUND(SUM(qty * price * discount/100) / COUNT(DISTINCT txn_id),2) AS average
FROM sales
```
This takes the total discount amount and divides it by the total number of unique transactions.

![image](https://github.com/user-attachments/assets/d21d3dc1-f900-4cf6-8c60-4aaf95fafb98)

### 5. What is the percentage split of all transactions for members vs non-members?

```sql
SELECT
  ROUND((SELECT COUNT(DISTINCT txn_id) FROM sales WHERE member = 't') / COUNT(DISTINCT txn_id) * 100, 2) AS members,
  ROUND((SELECT COUNT(DISTINCT txn_id) FROM sales WHERE member = 'f') / COUNT(DISTINCT txn_id) * 100, 2) AS nonmembers
FROM sales
```
I use subqueries to count the number of transactions by members and non-members, which are divided by the total amount of transactions, to get the percentages.

![image](https://github.com/user-attachments/assets/0cf8ec94-4623-42ef-907d-5e53fa19c453)

### 6. What is the average revenue for member transactions and non-member transactions?

```sql
WITH totals AS (
  SELECT
    txn_id,
    member,
    SUM(qty * price) / COUNT(DISTINCT txn_id) AS total
  FROM sales
  GROUP BY txn_id, member)

SELECT
  ROUND((SELECT AVG(total) FROM totals WHERE member = 't'), 2) AS avg_mem,
  ROUND((SELECT AVG(total) FROM totals WHERE member = 'f'), 2) AS avg_non
```
The CTE pulls the total revenue from all transactions, and the main query gets the average from both members and non-members.

![image](https://github.com/user-attachments/assets/5b54cfbb-e3fd-49e5-b8d8-49dae74a3cc0)

## C. Product Analysis
### 1. What are the top 3 products by total revenue before discount?

```sql
SELECT
  product_name,
  SUM(qty * s.price) AS amount
FROM sales s
JOIN product_details pd
  ON s.prod_id = pd.product_id
GROUP BY product_name
ORDER BY amount DESC
LIMIT 3
```
![image](https://github.com/user-attachments/assets/04a02d79-690c-4fab-ae6f-09d76f9d6ec7)

### 2. What is the total quantity, revenue and discount for each segment?

```sql
SELECT
  segment_id,
  segment_name,
  SUM(qty) AS quantity,
  SUM(qty * s.price) AS total_rev,
  ROUND(SUM(qty * s.price * discount/100), 2) AS discount
FROM sales s
JOIN product_details pd
  ON s.prod_id = pd.product_id
GROUP BY segment_id, segment_name
ORDER BY segment_id
```
![image](https://github.com/user-attachments/assets/f0ffd6bb-c626-4883-b4fb-15238953f0e8)

### 3. What is the top selling product for each segment?

```sql
SELECT
  segment_name,
  product_name,
  amount
FROM
  (SELECT
    segment_name,
    product_name,
    SUM(qty) AS amount,
    RANK() OVER (PARTITION BY segment_name ORDER BY SUM(qty) DESC) AS ranking
  FROM sales s
  JOIN product_details pd
    ON s.prod_id = pd.product_id
  GROUP BY product_name, segment_name
  ORDER BY segment_name ASC, amount DESC) a
WHERE ranking = 1
```
This ranks each segment so that the top selling product in each is ranked "1", allowing me to pull only the top sellers with the WHERE clause.

![image](https://github.com/user-attachments/assets/7ce59105-56b4-49c3-bc23-ac95c6a56b6b)

### 4. What is the total quantity, revenue and discount for each category?
### 5. What is the top selling product for each category?
### 6. What is the percentage split of revenue by product for each segment?
### 7. What is the percentage split of revenue by segment for each category?
### 8. What is the percentage split of total revenue by category?
### 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
### 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
