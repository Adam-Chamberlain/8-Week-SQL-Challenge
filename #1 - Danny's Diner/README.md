# [Case Study #1: Danny's Diner](https://8weeksqlchallenge.com/case-study-1/)

In this case study, a restaurant has provided various data on its customers, sales, and menu, and has asked to find specific pieces of data using complex SQL queries. This database uses three tables: `sales`, which shows what customer bought what product on what day, `members`, which lists customers as well as the date they became a member, and `menu`, which lists all of the menu items and their prices.

### Entity Relationship Diagram
![127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8](https://github.com/user-attachments/assets/c25db0b9-00d4-4c4b-9d1c-1062a9cbd381)

### 1. What is the total amount each customer spent at the restaurant?

```
SELECT
  s.customer_id,
  SUM(m.price) AS total
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY customer_id;
```
To solve this question, I first used JOIN to pull relevant data from both the `sales` and `menu` tables. I then found the sum of the price of all orders and grouped it by `customer_id` to show the sum of the amounts each customer spent.

| customer_id | total |
| ----------- | ----- |
| A           | 76    |
| B           | 74    |
| C           | 36    |

### 2. How many days has each customer visited the restaurant?

```
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS days_visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```
This question just required the `sales` table. Since `order_date` only showed the date and not exact times, I was able to use DISTINCT to only pull unique dates. I counted the number of unique dates and grouped them by each customer, showing how many unique days each one visited the restaurant.

| customer_id | date |
| ----------- | ---- |
| A           | 4    |
| B           | 6    |
| C           | 2    |

### 3. What was the first item from the menu purchased by each customer?

```
WITH ranked AS
SELECT DISTINCT
  s.customer_id,
  s.order_date,
  m.product_name,
  DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS ranking
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
ORDER BY ranking, s.order_date)

SELECT *
FROM ranked
WHERE ranking = 1
```
Once again, joining the `sales` and `menu` tables was required for this question. I used SELECT DISTINCT to ensure there were not any duplicate entries present. In order to find the first purchase from each customer, I used DENSE_RANK. The PARTITION BY clause ensured that each customer id has its own ranking, making the first order date for each customer #1.

I used this data as a CTE (Common Table Expression), which allows me to use it as a temporary table to use in a second query. With this, I ran another query to only pull the orders where the ranking was equal to 1, meaning it was the customer's first order. Even though all of the first orders would be at the top of the table regardless, this extra step removes the later orders, which are not relevant to the question

| customer_id | order_date               | product_name | ranking |
| ----------- | ------------------------ | ------------ | ------- |
| A           | 2021-01-01T00:00:00.000Z | curry        | 1       |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 1       |
| B           | 2021-01-01T00:00:00.000Z | curry        | 1       |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 1       |

Customer A visited twice on January 1, buying both curry and sushi. It is unclear which one was bought first with the given data. Customer B's first item was curry, and Customer C's was ramen.

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```
SELECT
  m.product_name,
  COUNT(s.product_id) AS amount
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY amount DESC
```
This query joins the `sales` and `menu` tables, counts the number of each product that was purchased, and groups them together by the product's name. It then orders them in descending order, showing the items that were purchased the most at the top. Ramen was the most purchased product, which was bought eight times.

| product_name | amount |
| ------------ | ------ |
| ramen        | 8      |
| curry        | 4      |
| sushi        | 3      |

### 5. Which item was the most popular for each customer?

```
WITH ranked AS (
SELECT
  s.customer_id,
  m.product_name,
  COUNT(s.product_id) AS amount,
  DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS ranking
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name
ORDER BY ranking, s.customer_id)

SELECT *
FROM ranked
WHERE ranking = 1
```
Similar to question 3, I used a CTE to get these results. This question also required two GROUP BY values, since it needed to see how many times each customer bought each product rather than the amount of times each product was bought or the amount of times each customer bought anything. Similar to question 3, I used DENSE_RANK to add rankings to each customer id, ordered in descending order to give the highest ranking to the product that was bought the most by each customer.

I then used that table to create a second query to only pull results where the ranking was 1.

| customer_id | product_name | amount | ranking |
| ----------- | ------------ | ------ | ------- |
| A           | ramen        | 3      | 1       |
| B           | ramen        | 2      | 1       |
| B           | curry        | 2      | 1       |
| B           | sushi        | 2      | 1       |
| C           | ramen        | 3      | 1       |

A and C's favorite product is ramen, which they each bought 3 times. B has a 3-way tie for their favorite product, buying each of them twice.

### 6. Which item was purchased first by the customer after they became a member?

```
WITH clean AS (
SELECT
    s.customer_id,
    s.order_date,
    ms.join_date,
    m.product_name
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
JOIN dannys_diner.members ms
    ON s.customer_id = ms.customer_id
WHERE join_date < order_date),

ranked AS(
SELECT *,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS ranking
FROM clean)

SELECT
    customer_id,
    product_name
FROM ranked
WHERE ranking = 1
```
This one took two CTEs. The first one, called `clean`, joins all the tables and filters out data so that only visits after the customer becomes a member are included. The second one, `ranked`, takes that first set of data and ranks them with DENSE_RANK, giving the earliest visit per customer rank 1. Lastly, using the data with the rankings from the `ranked` CTE, I filtered data to only show what each customer ordered on the rows with the ranking 1, which are their earliest visits *after* they became a member.

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

Customer A's first purchase after becoming a member was ramen, and customer B's was sushi. Customer C is not included because they are not a member.

### 7. Which item was purchased just before the customer became a member?

```
WITH clean AS (
SELECT
    s.customer_id,
    s.order_date,
    ms.join_date,
    m.product_name
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
JOIN dannys_diner.members ms
    ON s.customer_id = ms.customer_id
WHERE join_date > order_date),

ranked AS(
  SELECT *,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date DESC) AS ranking
FROM clean)

SELECT
    customer_id,
    product_name
FROM ranked
WHERE ranking = 1
```
This uses the same query as question 6, but with two changes. The join date should be before becoming a member instead of after, so the sign in the WHERE clause in the first CTE was flipped. In the DENSE_RANK clause, instead of ordering it in ascending order, it should be descending instead, since it is asking for the most recent date instead of the earliest date. Customer C is not included because they are not a member.

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

Customer A ordered both sushi and curry on the same day, though it is unsure which was the most recent before becoming a member, as time information is not given. Customer B's last order before becoming a member was sushi.

### 8. What is the total items and amount spent for each member before they became a member?

```
WITH before AS(
SELECT
    s.customer_id,
    s.order_date,
    ms.join_date,
    m.price
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
JOIN dannys_diner.members ms
    ON s.customer_id = ms.customer_id
WHERE join_date > order_date)

SELECT
customer_id,
COUNT(customer_id) AS total_items,
SUM(price) AS total_price
FROM before
GROUP BY customer_id
ORDER BY customer_id
```
Again, this uses a CTE called `before` to only pull the orders that took place before each customer became a member. It then counts the amount of orders and sums up the cost of those orders and groups them together. Like the above tables, customer C is not included because they are not a member.

| customer_id | total_items | total_price |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 3           | 40          |

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```
WITH point_calc AS (
SELECT
    s.customer_id,
    m.product_name,
    m.price,
    m.price * 10 + CASE WHEN m.product_name = 'sushi' THEN m.price * 10 ELSE 0 END AS points
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id)

SELECT
    customer_id,
    SUM(points) AS point_total
FROM point_calc
GROUP BY customer_id
ORDER BY customer_id
```
Here, a CASE statement is used to find the accurate amount of points per product. It starts by multiplying the cost by 10, since each dollar is equal to 10 points. It then checks if the product is sushi; if it is, it multiplies the price by 10 again and adds the two, getting the price multiplied by 20. If it's not sushi, it does nothing else, only multiplying it by 10.

| customer_id | point_total |
| ----------- | ----------- |
| A           | 860         |
| B           | 940         |
| C           | 360         |

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```
WITH point_calc AS (
SELECT
    s.customer_id,
    m.price * 10 +
    (CASE WHEN s.order_date BETWEEN ms.join_date AND ms.join_date + 6 THEN m.price * 10
     WHEN m.product_name = 'sushi' THEN m.price * 10 ELSE 0 END) AS points
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
JOIN dannys_diner.members ms
    ON s.customer_id = ms.customer_id
WHERE s.order_date < '2021-02-01')
  
SELECT
    customer_id,
    SUM(points) AS point_total
FROM point_calc
GROUP BY customer_id
ORDER BY customer_id
```

The CTE again pulls relevant data from each table and has a long CASE statement to show the accurate amount of points per order. All orders start by being multiplied by 10. It then checks if the order was placed within a week of the customer becoming a member *or* if the order was sushi. If either of those criteria are true, it multiplies the price by 10 again and adds the two, multiplying it by 20 in total.

Some additional notes for this question:
- It does not specify whether sushi should get multiplied a second time, making it 4x. For this scenario, I assumed it did not, so it only multiplies by 20 even if both criteria are met.
- It does not specify whether orders before customers become a member count towards points. I assumed all orders count for points regardless, so I did not exclude orders from before the customer became a member. If I wanted to, I could add a second WHERE clause to only show orders that took place after the customer became a member.

| customer_id | point_total |
| ----------- | ----------- |
| A           | 1220        |
| B           | 820         |

### Bonus Question 1 - Join All The Things

For this question, I had to recreate the table below:

| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-04T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

I did so using the following query:

```
SELECT
    s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE WHEN s.order_date >= ms.join_date THEN 'Y' ELSE 'N' END AS member
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members ms
    ON s.customer_id = ms.customer_id
ORDER BY s.customer_id ASC, s.order_date ASC, m.price DESC
```

I used a CASE to check whether the order date was after or equal to the date the customer became a member. If it was the case, it put "Y", and if not, it put "N". Also note that I used a LEFT JOIN with the members table. If I did not, it would not show customer C's orders, since they are not a member.

### Bonus Question 2 - Rank All The Things

For this question, I had to recreate the table below:

| customer_id | order_date               | product_name | price | member | ranking |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |         |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |         |
| A           | 2021-01-04T00:00:00.000Z | curry        | 15    | N      |         |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      | 1       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |         |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |         |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |         |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |         |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |         |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |         |

I did so using the following query:

```
WITH cte AS (
SELECT
    s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE WHEN s.order_date >= ms.join_date THEN 'Y' ELSE 'N' END AS member
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members ms
    ON s.customer_id = ms.customer_id
ORDER BY s.customer_id ASC, s.order_date ASC, m.price DESC)

SELECT *,
    CASE WHEN member = 'N' THEN null ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date) END AS ranking
FROM cte
```

I started with the exact same query as the prior bonus question, as it used the same columns in the same order. To get the final column, I put the initial query's results in a CTE and created a CASE statement to make all instances where the `member` column has "N" return null. Otherwise, it ranks the remaining orders, with the earliest ones having the highest ranking. It was partitioned by both `customer_id` and `member` so that it did not include the null instances in the rankings; it sorted them separately, but those rankings are not shown since they are overwritten by null values.
