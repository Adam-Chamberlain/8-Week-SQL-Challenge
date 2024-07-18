## A. Pizza Metrics
### 1. How many pizzas were ordered?

```
SELECT
    COUNT(*) AS orders
FROM pizza_runner.customer_orders;
```
The first question is pretty simple; I just count the amount of orders that took place on the `customer_orders` table.

| orders |
| ------ |
| 14     |

### 2. How many unique customer orders were made?

```
SELECT
    COUNT(DISTINCT order_id) AS unique_orders
FROM pizza_runner.customer_orders
```
This uses the same query as the one above, but instead of counting all rows, it counds the amount of unique order IDs instead.

| unique_orders |
| ------------- |
| 10            |

### 3. How many successful orders were delivered by each runner?

```
WITH temp AS (
SELECT
    *,
    CASE WHEN cancellation LIKE 'null' or cancellation LIKE '' THEN null ELSE cancellation END as cancellations
FROM pizza_runner.runner_orders)

SELECT
    runner_id,
    COUNT(order_id) AS delivered
FROM temp
WHERE cancellations IS NULL
GROUP BY runner_id
```
This one required some data cleaning beforehand. The null values in the `cancellation` column was a mix of actual null values, the text 'null', and a singular space. The above CASE statement converts the text and spaces to actual null values, and the lower query filters out only orders where that column equals null, meaning the order was successfully delivered and not cancelled.

| runner_id | delivered |
| --------- | --------- |
| 1         | 4         |
| 2         | 3         |
| 3         | 1         |

### 4. How many of each type of pizza was delivered?

```
SELECT
p.pizza_name,
COUNT(c.pizza_id) AS amount
FROM pizza_runner.customer_orders c
JOIN pizza_runner.pizza_names p
ON c.pizza_id = p.pizza_id
JOIN pizza_runner.runner_orders r
ON c.order_id = r.order_id
WHERE CASE WHEN r.cancellation LIKE 'null' or r.cancellation LIKE '' THEN null ELSE r.cancellation END IS NULL
GROUP BY p.pizza_name
```
Again, this required some cleaning. I joined the `customer_orders` and `pizza_names` tables to pull the accurate pizza names, and I then joined it with the `runner_orders` table to exclude cancellations. I used the same CASE statement in the WHERE clause to only include null values, meaning they were successfully delivered.

| pizza_name | amount |
| ---------- | ------ |
| Meatlovers | 9      |
| Vegetarian | 3      |

### 5. How many Vegetarian and Meatlovers were ordered by each customer?

```
SELECT
c.customer_id,
p.pizza_name,
COUNT(c.pizza_id) AS amount
FROM pizza_runner.customer_orders c
JOIN pizza_runner.pizza_names p
ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY customer_id
```
For this one, I simply had to count the amount of orders and group it by the customer id and the pizza name. This shows how many of each pizza was ordered by each customer.

| customer_id | pizza_name | amount |
| ----------- | ---------- | ------ |
| 101         | Meatlovers | 2      |
| 101         | Vegetarian | 1      |
| 102         | Meatlovers | 2      |
| 102         | Vegetarian | 1      |
| 103         | Meatlovers | 3      |
| 103         | Vegetarian | 1      |
| 104         | Meatlovers | 3      |
| 105         | Vegetarian | 1      |

### 6. What was the maximum number of pizzas delivered in a single order?

```
WITH temp AS (
SELECT
    c.order_id
FROM pizza_runner.customer_orders c
JOIN pizza_runner.runner_orders r
    ON c.order_id = r.order_id
WHERE CASE WHEN r.cancellation LIKE 'null' or r.cancellation LIKE '' THEN null ELSE r.cancellation END IS NULL)

SELECT 
    order_id,
    COUNT(order_id) AS amount
FROM temp
GROUP BY order_id
ORDER BY amount DESC
```
I first joined the necessary tables and pulled the relevant information with unsuccessful orders filtered out, again using the same cleaning method as before. Then, the orders were counted, grouped by the order id, and ordered to show the ones with the most pizzas ordered at the top.

| order_id | amount |
| -------- | ------ |
| 4        | 3      |
| 3        | 2      |
| 10       | 2      |
| 7        | 1      |
| 1        | 1      |
| 5        | 1      |
| 2        | 1      |
| 8        | 1      |

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```
WITH temp AS (
SELECT
    c.customer_id,
    c.pizza_id,
  	CASE WHEN exclusions LIKE 'null' or exclusions LIKE '' THEN null ELSE exclusions END AS exclusions,
  CASE WHEN extras LIKE 'null' or extras LIKE '' THEN null ELSE extras END AS extras
FROM pizza_runner.customer_orders c
JOIN pizza_runner.runner_orders r
    ON c.order_id = r.order_id
WHERE CASE WHEN r.cancellation LIKE 'null' or r.cancellation LIKE '' THEN null ELSE r.cancellation END IS NULL)

SELECT
customer_id,
SUM(CASE WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN 1 ELSE 0 END) AS changes,
SUM(CASE WHEN exclusions IS NULL AND extras IS NULL THEN 1 ELSE 0 END) AS no_changes
FROM temp
GROUP BY customer_id
ORDER BY customer_id
```
This one was a bit more complicated and required lots of cleaning. Like the `cancellation` column, the `exclusions` and `extras` columns had inconsistent null values too, so I had to make all three of them consistent. Using the cleaned data, I used CASE statements to find which pizzas had either exclusions, extras, or both to create the `changes` column. I used another CASE statement to only find pizzas that have neither, creating the `no_changes` column.

| customer_id | changes | no_changes |
| ----------- | ------- | ---------- |
| 101         | 0       | 3          |
| 102         | 0       | 3          |
| 103         | 4       | 0          |
| 104         | 2       | 1          |
| 105         | 1       | 0          |

### 8. How many pizzas were delivered that had both exclusions and extras?

```
WITH temp AS (
SELECT
    c.customer_id,
    c.pizza_id,
  	CASE WHEN exclusions LIKE 'null' or exclusions LIKE '' THEN null ELSE exclusions END AS exclusions,
  CASE WHEN extras LIKE 'null' or extras LIKE '' THEN null ELSE extras END AS extras
FROM pizza_runner.customer_orders c
JOIN pizza_runner.runner_orders r
    ON c.order_id = r.order_id
WHERE CASE WHEN r.cancellation LIKE 'null' or r.cancellation LIKE '' THEN null ELSE r.cancellation END IS NULL)

SELECT
    SUM(CASE WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN 1 ELSE 0 END) AS both
FROM temp
```
This one just required a few changes from the last one. Instead of using OR in the CASE statement, it uses AND to check if it meets both criteria. I also removed the extra columns that are not relevant for this question. Although there are actually two orders that have both exclusions and extras, one was cancelled, so it was filtered out in the first query.

| both |
| ---- |
| 1    |

### 9. What was the total volume of pizzas ordered for each hour of the day?

```
SELECT 
EXTRACT(HOUR FROM order_time) AS hour,
COUNT(order_id) AS amount
FROM pizza_runner.customer_orders
GROUP BY hour
ORDER BY hour
```
For this, the hour of the day is extracted from the order time, and with that, it counts the number of pizzas ordered per hour of the day, regardless of what day the order was placed. If it only asked for orders that were successfully delivered, a WHERE clause could easily be added to filter out cancelled orders.

| hour | amount |
| ---- | ------ |
| 11   | 1      |
| 13   | 3      |
| 18   | 3      |
| 19   | 1      |
| 21   | 3      |
| 23   | 3      |

### 10. What was the volume of orders for each day of the week?

```
WITH temp AS (
SELECT 
    EXTRACT(isodow FROM order_time) AS day,
    COUNT(order_id) AS amount
FROM pizza_runner.customer_orders
GROUP BY day
ORDER BY day)

SELECT
    CASE WHEN day = 1 THEN 'Monday'
    WHEN day = 2 THEN 'Tuesday'
    WHEN day = 3 THEN 'Wednesday'
    WHEN day = 4 THEN 'Thursday'
    WHEN day = 5 THEN 'Friday'
    WHEN day = 6 THEN 'Saturday'
    WHEN day = 7 THEN 'Sunday' ELSE null END AS day,
    amount
FROM temp
```
Here, "isodow" is extracted from `order_time`, which is the day of the week (1 = Monday, 2 = Tuesday, etc). The first query groups the amount of pizzas ordered per day of the week, and the second one changes the numbers to the actual name of the weekday using a long CASE statement.

| day       | amount |
| --------- | ------ |
| Wednesday | 5      |
| Thursday  | 3      |
| Friday    | 1      |
| Saturday  | 5      |

## B. Runner and Customer Experience
### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```
SELECT
    CASE WHEN registration_date BETWEEN '2021-01-01' AND '2021-01-07' THEN 'Week 1: 1/1-1/7'
    WHEN registration_date BETWEEN '2021-01-08' AND '2021-01-14' THEN 'Week 2: 1/8-1/14'
    WHEN registration_date BETWEEN '2021-01-15' AND '2021-01-21' THEN 'Week 3: 1/15-1/21' ELSE null END AS week,
    COUNT(runner_id) AS signups
FROM pizza_runner.runners
GROUP BY week
ORDER BY week
```
I had to use a CASE statement to number each week period. If I used EXTRACT(WEEK) instead, it would start each week period on Monday, which is not the case here, since January 1, 2021 was on a Friday. I labeled each week within the query to avoid confusion.

| week              | signups |
| ----------------- | ------- |
| Week 1: 1/1-1/7   | 2       |
| Week 2: 1/8-1/14  | 1       |
| Week 3: 1/15-1/21 | 1       |

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
### 4. What was the average distance travelled for each customer?
### 5. What was the difference between the longest and shortest delivery times for all orders?
### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
### 7. What is the successful delivery percentage for each runner?
## C. Ingredient Optimization
### 1. What are the standard ingredients for each pizza?
### 2. What was the most commonly added extra?
### 3. What was the most common exclusion?
### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
## D. Pricing and Ratings
### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
### 2. What if there was an additional $1 charge for any pizza extras?
- Add cheese is $1 extra
### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
###  4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas
### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
## E. Bonus Questions
### If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?
