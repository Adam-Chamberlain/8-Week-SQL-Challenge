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
To solve this question, I first used JOIN to pull relevant data from both the "sales" and "menu" tables. I then pulled the SUM of the price of all orders, getting the total amount of money spent, and grouped it by "customer_id" to show the sum of the amounts each customer spent.

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
This question just required the sales table. Since `order_date` only showed the date and not exact times, I was able to use DISTINCT to only pull unique dates. I counted the number of unique dates and grouped them by each customer, showing how many unique days each one visited the restaurant.

| customer_id | date |
| ----------- | ---- |
| A           | 4    |
| B           | 6    |
| C           | 2    |

### 3. What was the first item from the menu purchased by each customer?

```
SELECT DISTINCT
  s.customer_id,
  s.order_date,
  m.product_name
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON s.product_id = m.product_id
ORDER BY order_date
```
Once again, joining the `sales` and `menu` tables was required for this question. I simply pulled all of the orders and ordered them by date to easily find the first order for each customer. If this were a much larger database with more customers and a longer timeframe, this would require a much more complex query, but for this scenario, the above query works great.

| customer_id | order_date               | product_name |
| ----------- | ------------------------ | ------------ |
| A           | 2021-01-01T00:00:00.000Z | curry        |
| A           | 2021-01-01T00:00:00.000Z | sushi        |
| B           | 2021-01-01T00:00:00.000Z | curry        |
| C           | 2021-01-01T00:00:00.000Z | ramen        |
| B           | 2021-01-02T00:00:00.000Z | curry        |
| B           | 2021-01-04T00:00:00.000Z | sushi        |
| A           | 2021-01-07T00:00:00.000Z | curry        |
| C           | 2021-01-07T00:00:00.000Z | ramen        |
| A           | 2021-01-10T00:00:00.000Z | ramen        |
| A           | 2021-01-11T00:00:00.000Z | ramen        |
| B           | 2021-01-11T00:00:00.000Z | sushi        |
| B           | 2021-01-16T00:00:00.000Z | ramen        |
| B           | 2021-02-01T00:00:00.000Z | ramen        |

Customer A visited twice on 1/1, buying both curry and sushi. It is unclear which one was bought first with the given data. Customer B's first item was curry, and Customer C's was ramen.

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
### 5. Which item was the most popular for each customer?
### 6. Which item was purchased first by the customer after they became a member?
### 7. Which item was purchased just before the customer became a member?
### 8. What is the total items and amount spent for each member before they became a member?
### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

