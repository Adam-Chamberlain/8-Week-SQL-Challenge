# [Case Study 6: Clique Bait](https://8weeksqlchallenge.com/case-study-6/)

This case study looks at website and visit data for an online seafood seller. Lots of information is provided, including individual steps that are taken during each individual user's visit to the website. There are five initial tables, which are shown in the first question:

## 1. Enterprise Relationship Diagram
Using the following DDL schema details to create an ERD for all the Clique Bait datasets.

```sql
Table event_identifier {
  event_type integer
  event_name integer
}

Table campaign_identifier {
  campaign_id integer
  products varchar
  campaign_name varchar
  start_date timestamp
  end_date timestamp
}

Table page_hierarchy {
  page_id integer
  page_name varchar
  product_category varchar
  product_id integer
}

Table users {
  user_id integer
  cookie_id varchar
  start_date timestamp
}

Table events {
  visit_id integer
  cookie_id varchar
  page_id integer
  event_type integer
  sequence_number integer
  event_time timestamp
}

Ref: event_identifier.event_type > events.event_type
Ref: users.cookie_id > events.cookie_id
Ref: page_hierarchy.page_id > events.page_id
```
With the following code, I was able to create this entity relationship diagram using [this website.](https://dbdiagram.io/d)

![image](https://github.com/user-attachments/assets/5d7de0ec-2f09-4bff-a984-e1ae1976b745)

## 2. Digital Analysis

### 1. How many users are there?

```sql
SELECT
COUNT(DISTINCT user_id) AS usercount
FROM users
```
![image](https://github.com/user-attachments/assets/48ee27ea-423d-4b6e-af1a-b4d203c1ab29)

### 2. How many cookies does each user have on average?

```sql
SELECT
COUNT(cookie_id) / COUNT(DISTINCT user_id) AS average
FROM users
```
This counts the total amount of rows with cookies and divides it by the unique number of user IDs to get the average.

![image](https://github.com/user-attachments/assets/0a090fb8-ed03-46ca-9855-1c4411c25064)

### 3. What is the unique number of visits by all users per month?

```sql
SELECT
MONTH(event_time) AS month,
COUNT(DISTINCT visit_id) AS visits
FROM events
GROUP BY month
ORDER BY month
```
Each visit ID adds a new row when a new page is viewed, so it needs to show distinct visit IDs for this to work accurately. 

![image](https://github.com/user-attachments/assets/380712f6-eb66-467a-8d58-c74ea3ad03aa)

### 4. What is the number of events for each event type?

```sql
SELECT
e.event_type,
event_name,
COUNT(e.event_type) AS amount
FROM events e
JOIN event_identifier ei
ON e.event_type = ei.event_type
GROUP BY e.event_type, event_name
```
![image](https://github.com/user-attachments/assets/3cfdcbc8-b7e0-46ea-8f6c-950082064653)

### 5. What is the percentage of visits which have a purchase event?

```sql
SELECT
  ROUND((SELECT COUNT(DISTINCT visit_id) FROM events WHERE event_type = 3)
  / COUNT(DISTINCT visit_id), 2) AS percent
FROM events
```
This uses a subquery to find how many unique visit IDs have a purchase event (the value 3), which is then divided by the total amount of unique visits to get the percentage.

![image](https://github.com/user-attachments/assets/4f05745c-babd-408b-81d4-73057e7ca1ec)

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?

```sql
WITH cte AS (
  SELECT
    a.visit_id,
    page_id,
    event_type
  FROM(
    SELECT
      DISTINCT visit_id,
      page_id
    FROM events
    WHERE page_id = 12) a
  LEFT JOIN(
    SELECT
      DISTINCT visit_id,
      event_type
    FROM events
    WHERE event_type = 3) b
  ON a.visit_id = b.visit_id)

SELECT
  (SELECT COUNT(visit_id) FROM cte WHERE event_type IS NULL) / COUNT(DISTINCT visit_id) AS percentage
FROM events
```
This CTE creates a table that JOINs the `events` table twice to show every unique visit ID that viewed the checkout page (12), whether or not they also purchased an item (3), as shown below:

![image](https://github.com/user-attachments/assets/ad9615fc-316d-47ad-b939-16238ade0ace)

The rows with NULL values in `event_type` are the ones that did not purchase an item but still viewed the checkout page at some point. I counted all the rows where the value was NULL and divided it by the total number of unique visit IDs to get the final percentage.

![image](https://github.com/user-attachments/assets/641edf13-e5b5-40ec-8f37-86e0f0b125b6)

### 7. What are the top 3 pages by number of views?

```sql
SELECT
  page_name,
  COUNT(e.page_id) AS amount
FROM events e
JOIN page_hierarchy ph
  ON e.page_id = ph.page_id
WHERE e.event_type = 1
GROUP BY page_name
ORDER BY amount DESC
LIMIT 3
```
`event_type` has to equal 1 (page view) since the question is asking for number of views only.

![image](https://github.com/user-attachments/assets/45b51dad-45fd-4d2c-91e4-dc2764385efb)

### 8. What is the number of views and cart adds for each product category?

```sql
SELECT
  product_category,
  COUNT(CASE WHEN event_type = 1 THEN 1 ELSE NULL END) AS views,
  COUNT(CASE WHEN event_type = 2 THEN 1 ELSE NULL END) AS cart_adds
FROM events e
JOIN page_hierarchy ph
  ON e.page_id = ph.page_id
WHERE product_category IS NOT NULL
GROUP BY product_category
ORDER BY views DESC
```
![image](https://github.com/user-attachments/assets/bbd885c5-dfea-456b-bfc3-113974d0750c)

### 9. What are the top 3 products by purchases?

```sql
WITH cte AS (SELECT
    a.visit_id,
    a.page_id AS purchases,
    b.page_id AS checkout
  FROM(
    SELECT
      DISTINCT visit_id,
      page_id
    FROM events
    WHERE event_type = 2) a
  LEFT JOIN(
    SELECT
      DISTINCT visit_id,
      page_id
    FROM events
    WHERE page_id = 13) b
  ON a.visit_id = b.visit_id
  WHERE b.page_id IS NOT NULL)
  
SELECT
  page_name,
  COUNT(purchases) AS amount
FROM cte c
JOIN page_hierarchy ph
  ON c.purchases = ph.page_id
GROUP BY page_name
ORDER BY amount DESC
 LIMIT 3
```
The first subquery pulls all purchases (instances where `event_type` is 2) and the second subquery pulls all instances where the user checked out and actually purchased the items (where `page_id` is 13. The two subqueries are joined together using a left join to show all purchases, regardless of if they checked out or not. If they did not check out, the `checkout` column would show NULL, which is filtered out with the WHERE clause at the end.

![image](https://github.com/user-attachments/assets/3408aadd-eefe-4644-b514-3ed4118db22b)

Finally, the purchases are grouped by the name of the page.

![image](https://github.com/user-attachments/assets/213d7509-df19-406b-910d-e60f80dd34ca)

## 3. Product Funnel Analysis

Using a single SQL query - create a new output table which has the following details:

- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?

```sql
WITH purchases AS (SELECT
    a.visit_id,
    a.page_id AS purchases,
    CASE WHEN b.page_id IS NOT NULL THEN 1 ELSE 0 END AS checkout
  FROM(
    SELECT
      DISTINCT visit_id,
      page_id
    FROM events
    WHERE event_type = 2) a
  LEFT JOIN(
    SELECT
      DISTINCT visit_id,
      page_id
    FROM events
    WHERE page_id = 13) b
  ON a.visit_id = b.visit_id),
  
viewed AS (
  SELECT
  page_id,
  COUNT(page_id) AS views
  FROM events
  WHERE event_type = 1 AND page_id NOT IN(1,2,12,13)
  GROUP BY page_id),
  
cart AS (
  SELECT
  page_id,
  COUNT(page_id) AS in_cart
  FROM events
  WHERE event_type = 2
  GROUP BY page_id)
  
SELECT
  page_name,
  views,
  in_cart,
  SUM(checkout) AS purchased,
  SUM(CASE WHEN checkout = 1 THEN 0 ELSE 1 END) AS not_purchased
FROM purchases p
JOIN page_hierarchy ph
  ON p.purchases = ph.page_id
  JOIN viewed v
ON p.purchases = v.page_id
  JOIN cart ca
ON p.purchases = ca.page_id
GROUP BY page_name, views, in_cart
```
The `purchases` CTE works similarly to the one in the prior question to identify which products were purchased. Purchases are given the value 1, and ones that were not purchased are given 0. To get the total amount that was not purchased, I used a CASE statement to invert these numbers.

The other two CTEs show the amount of times each product was viewed or added to the cart. They were then joined with the other CTE to put it all together. I saved this table as the name `products`.

![image](https://github.com/user-attachments/assets/3e34f123-66cf-4eb4-b2ef-6fff9147f1ec)

### Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

```sql
combined AS (
SELECT
product_category,
SUM(views) AS views,
SUM(in_cart) AS in_cart
FROM viewed v
JOIN cart c
ON v.page_id = c.page_id
JOIN page_hierarchy ph
ON v.page_id = ph.page_id
GROUP BY product_category)
  
SELECT
  ph.product_category,
  views,
  in_cart,
  SUM(checkout) AS purchased,
  SUM(CASE WHEN checkout = 1 THEN 0 ELSE 1 END) AS not_purchased
FROM purchases p
JOIN page_hierarchy ph
  ON p.purchases = ph.page_id
  JOIN combined c
  ON ph.product_category = c.product_category
GROUP BY product_category, views, in_cart
```
To do this, I added one more CTE called `combined` to combine the `views` and `in_cart` columns and convert them to product categories. With that, I used a similar final query. I saved this table as `categories`.

![image](https://github.com/user-attachments/assets/2dc78f93-31f9-45b7-86e3-544d71204b0f)

Use your 2 new output tables - answer the following questions:

### 1. Which product had the most views, cart adds and purchases?

Views: Oyster. Cart Adds: Lobster. Purchases: Lobster.

### 2. Which product was most likely to be abandoned?

```sql
SELECT
page_name,
not_purchased / in_cart AS percent
FROM products
ORDER BY percent
```
I divided the amount that were in the cart but not purchased by the total amount in the cart to see the percent of each product that did not get purchased. Russian Caviar is the most likely to be abandoned.

![image](https://github.com/user-attachments/assets/59f115f8-c10a-4d94-ac2d-c71fef2c796e)

### 3. Which product had the highest view to purchase percentage?

```sql
SELECT
page_name,
purchased / views AS ratio
FROM products
ORDER BY ratio DESC
```
Similar to the prior question, I divided the amount of purchases by the amount of views. Lobster has the highest view-to-purchase percentage.

![image](https://github.com/user-attachments/assets/f7708951-127b-40dd-b67b-aefa692367b5)

### 4. What is the average conversion rate from view to cart add?

```sql
SELECT
page_name,
in_cart / views AS ratio
FROM products
ORDER BY ratio DESC
```
![image](https://github.com/user-attachments/assets/32dd1ffe-6103-41e9-b708-c9cb1aff620f)

### 5. What is the average conversion rate from cart add to purchase?

```sql
SELECT
page_name,
purchased / in_cart AS ratio
FROM products
ORDER BY ratio DESC
```
![image](https://github.com/user-attachments/assets/8d091239-2329-4acc-9795-a5269f7fa52e)

### 4. Campaigns Analysis

Generate a table that has 1 single row for every unique visit_id record and has the following columns:

- user_id
- visit_id
- visit_start_time: the earliest event_time for each visit
- page_views: count of page views for each visit
- cart_adds: count of product cart add events for each visit
- purchase: 1/0 flag if a purchase event exists for each visit
- campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- impression: count of ad impressions for each visit
- click: count of ad clicks for each visit
- (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

```sql
WITH starttime AS (
SELECT
visit_id,
event_time
FROM events
WHERE sequence_number = 1)

SELECT
user_id,
e.visit_id,
st.event_time AS visit_start_time,
SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS page_views,
SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS cart_adds,
SUM(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase,
campaign_name,
SUM(CASE WHEN event_type = 4 THEN 1 ELSE 0 END) AS impression,
SUM(CASE WHEN event_type = 5 THEN 1 ELSE 0 END) AS click,
GROUP_CONCAT(CASE WHEN event_type = 2 THEN page_name ELSE NULL END ORDER BY sequence_number SEPARATOR ', ') AS cart_products
FROM events e

JOIN starttime st
ON e.visit_id = st.visit_id

LEFT JOIN users u
ON e.cookie_id = u.cookie_id

LEFT JOIN campaign_identifier ci
ON e.event_time BETWEEN ci.start_date AND ci.end_date

JOIN page_hierarchy ph
ON e.page_id = ph.page_id

GROUP BY e.visit_id, st.event_time, user_id, campaign_name
ORDER BY user_id
```
I was able to add the `user_id` column by joining the table with the `users` table. I used a CTE to get the start time from each visit, which was also joined. To get the number of page views, cart adds, etc, I used CASE statements to count how many times each appeared per visit. To identify ongoing campaigns, I joined the `campaign_identifier` table with the start time to see if it is between the start and end date of any campaigns. This uses a LEFT JOIN, since some visits were not during campaigns, and they would not show otherwise. Lastly, I used a GROUP_CONCAT to list all cart adds in one column.

![image](https://github.com/user-attachments/assets/1a2edcd1-ebd6-4df9-988a-97e68b129931)
