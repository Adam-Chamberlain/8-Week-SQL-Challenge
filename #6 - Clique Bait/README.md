# [Case Study 6: Clique Bait](https://8weeksqlchallenge.com/case-study-6/)

## 1. Enterprise Relationship Diagram
Using the following DDL schema details to create an ERD for all the Clique Bait datasets.

```
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

```
SELECT
COUNT(DISTINCT user_id) AS usercount
FROM users
```
![image](https://github.com/user-attachments/assets/48ee27ea-423d-4b6e-af1a-b4d203c1ab29)

### 2. How many cookies does each user have on average?

```
SELECT
COUNT(cookie_id) / COUNT(DISTINCT user_id) AS average
FROM users
```
This counts the total amount of rows with cookies and divides it by the unique number of user IDs to get the average.

![image](https://github.com/user-attachments/assets/0a090fb8-ed03-46ca-9855-1c4411c25064)

### 3. What is the unique number of visits by all users per month?

```
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

```
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

```
SELECT
  ROUND((SELECT COUNT(DISTINCT visit_id) FROM events WHERE event_type = 3)
  / COUNT(DISTINCT visit_id), 2) AS percent
FROM events
```
This uses a subquery to find how many unique visit IDs have a purchase event (the value 3), which is then divided by the total amount of unique visits to get the percentage.

![image](https://github.com/user-attachments/assets/4f05745c-babd-408b-81d4-73057e7ca1ec)

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?

```
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

```
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

```
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

## 3. Product Funnel Analysis

Using a single SQL query - create a new output table which has the following details:

- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?

Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables - answer the following questions:

1. Which product had the most views, cart adds and purchases?
2. Which product was most likely to be abandoned?
3. Which product had the highest view to purchase percentage?
4. What is the average conversion rate from view to cart add?
5. What is the average conversion rate from cart add to purchase?

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

Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.

Some ideas you might want to investigate further include:

- Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
- Does clicking on an impression lead to higher purchase rates?
- What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
- What metrics can you use to quantify the success or failure of each campaign compared to eachother?
