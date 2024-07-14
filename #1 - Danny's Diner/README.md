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
To solve this question, I first used JOIN to pull relevant data from both the `sales` and `menu` tables. I then found the sum of the price of all orders, getting the total amount of money spent, and grouped it by `customer_id` to show the sum of the amounts each customer spent.

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
WITH first_purchase AS(SELECT DISTINCT
  s.customer_id,
  s.order_date,
  m.product_name,
  DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS ranking
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON s.product_id = m.product_id
ORDER BY ranking, s.order_date)

SELECT *
FROM first_purchase
WHERE ranking = 1
```
Once again, joining the `sales` and `menu` tables was required for this question. I used SELECT DISTINCT to ensure there were not any duplicate entries present. In order to find the first purchase from each customer, I used DENSE_RANK. The PARTITION BY clause ensured that each customer id has its own ranking, making the first order date for each customer #1.

I used this info as a temporary table and ran another query to only pull the orders where the ranking was equal to 1, meaning it was the customer's first order. Even though all of the first orders would be at the top of the table regardless, this extra step removes the later orders, which are not relevant to the question

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
WITH most_popular AS(SELECT
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
FROM most_popular
WHERE ranking = 1
```
Similar to question 3, I used a temporary table to get these results. This question also required two GROUP BY values, since it needed to see how many times each customer bought each product rather than the amount of times each product was bought or the amount of times each customer bought anything. Similar to question 3, I used DENSE_RANK to add rankings to each customer id, ordered in descending order to give the highest ranking to the product that was bought the most by each customer.

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
### 7. Which item was purchased just before the customer became a member?
### 8. What is the total items and amount spent for each member before they became a member?
### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

