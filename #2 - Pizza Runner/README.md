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
WITH clean AS (
SELECT
    *,
    CASE WHEN cancellation LIKE 'null' or cancellation LIKE '' THEN null ELSE cancellation END as cancellations
FROM pizza_runner.runner_orders)

SELECT
    runner_id,
    COUNT(order_id) AS delivered
FROM clean
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
WITH clean AS (
SELECT
    c.order_id
FROM pizza_runner.customer_orders c
JOIN pizza_runner.runner_orders r
    ON c.order_id = r.order_id
WHERE CASE WHEN r.cancellation LIKE 'null' or r.cancellation LIKE '' THEN null ELSE r.cancellation END IS NULL)

SELECT 
    order_id,
    COUNT(order_id) AS amount
FROM clean
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
WITH clean AS (
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
FROM clean
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
WITH clean AS (
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
FROM clean
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
WITH clean AS (
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
FROM clean
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

```
WITH clean AS (
SELECT
  r.runner_id,
  c.order_time,
  CASE WHEN r.pickup_time LIKE 'null' THEN null ELSE CAST(r.pickup_time AS TIMESTAMP) END AS pickup
  FROM pizza_runner.runner_orders r
JOIN pizza_runner.customer_orders c
ON r.order_id = c.order_id)

SELECT
  runner_id,
  DATE_PART('minute', AVG(pickup - order_time)) AS average_time
  FROM clean
  GROUP BY runner_id
```
This took a lot of cleaning yet again; the `pickup_time` had "null" text and was not in the timestamp format because of that. I used a CASE statement to convert null text to actual null values and then converted the times to timestamps. This allowed me to subtract the order time from it to get the exact time difference between when the order was placed and when the order was picked up. I used DATE_PART to only pull the amount of minutes rather than including seconds and milliseconds, as the question only asks for minutes.

| runner_id | average_time |
| --------- | ------------ |
| 3         | 10           |
| 2         | 23           |
| 1         | 15           |

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```
WITH clean AS (
SELECT
  c.order_id,
  c.order_time,
  CASE WHEN r.pickup_time LIKE 'null' THEN null ELSE CAST(r.pickup_time AS TIMESTAMP) END AS pickup
  FROM pizza_runner.runner_orders r
JOIN pizza_runner.customer_orders c
ON r.order_id = c.order_id)

SELECT
  order_id,
  COUNT(order_id) AS volume,
  DATE_PART('minute', pickup - order_time) AS time
  FROM clean
  GROUP BY order_id, pickup, order_time
  ORDER BY order_id
```
For this one, I used the same CTE to pull relevant cleaned information, the only difference being the order id being pulled instead of the runner id. I then counted the amount of pizzas ordered and grouped it by the order id to get the total number of pizzas ordered per order id. This chart shows a clear trend; most orders with one pizza take about 10 minutes, with the exception of order 8. Orders with two pizzas take longer, with one taking 15 minutes and one taking 21 minutes. Order 4, which had 3 pizzas, took the longest, at 29 minutes.

| order_id | volume | time |
| -------- | ------ | ---- |
| 1        | 1      | 10   |
| 2        | 1      | 10   |
| 3        | 2      | 21   |
| 4        | 3      | 29   |
| 5        | 1      | 10   |
| 6        | 1      |      |
| 7        | 1      | 10   |
| 8        | 1      | 20   |
| 9        | 1      |      |
| 10       | 2      | 15   |

### 4. What was the average distance travelled for each customer?

```
WITH clean AS(
SELECT
DISTINCT c.order_id,
c.customer_id,
CASE WHEN r.distance LIKE '%km' THEN TRIM('%km' FROM r.distance)
WHEN r.distance LIKE 'null' THEN null ELSE r.distance END AS distance
FROM pizza_runner.customer_orders c
JOIN pizza_runner.runner_orders r
ON c.order_id = r.order_id
WHERE CASE WHEN r.distance LIKE 'null' THEN null ELSE r.distance END IS NOT NULL)

SELECT
customer_id,
AVG(CAST(distance AS DECIMAL)) AS avg_distance
FROM clean
GROUP BY customer_id
ORDER BY customer_id
```
The `distance` column had inconsistencies with how the data was inputted, so it required lots of cleaning. In the CTE, I removed "km" from all the numbers that included it using TRIM and changed the null text to the actual null value. I then used CAST to convert the numbers to decimals and added up the average for each customer.

| customer_id | avg_distance        |
| ----------- | ------------------- |
| 101         | 20.0000000000000000 |
| 102         | 18.4000000000000000 |
| 103         | 23.4000000000000000 |
| 104         | 10.0000000000000000 |
| 105         | 25.0000000000000000 |

### 5. What was the difference between the longest and shortest delivery times for all orders?

```
WITH clean AS (
SELECT
CAST(CASE WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
WHEN duration LIKE 'null' THEN NULL ELSE duration END AS INTEGER) AS delivery_duration
FROM pizza_runner.runner_orders)

SELECT
MAX(delivery_duration) - MIN(delivery_duration) AS difference
FROM clean
```
The `duration` tab needed a lot of cleaning, as it was a mix of just numbers, x mins, x minutes, and x minute. I used a long CASE statement to clean up all inconsistencies, and I used CAST around that to convert them all to integers. I then subtracted the max value from the min value, getting 30 minutes as the difference.

| difference |
| ---------- |
| 30         |

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

```
WITH clean AS(
SELECT
runner_id,
CAST(CASE WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
  WHEN distance LIKE 'null' THEN NULL ELSE distance END AS DECIMAL) AS distance,
CAST(CASE WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
WHEN duration LIKE 'null' THEN NULL ELSE duration END AS DECIMAL) AS duration
FROM pizza_runner.runner_orders)

SELECT
runner_id,
ROUND(distance / duration * 60, 2) AS avg_km_per_hour
FROM clean
WHERE distance IS NOT NULL
ORDER BY runner_id, avg_km_per_hour DESC
```
The CTE cleaned up the `distance` and `duration` columns, removing inconsistancies and converting them to decimal values. Then, they are calculated into the average KM per hour per order. Runner 1's average fluctuates between 37.5 and 60 KM/h, while runner 2's goes from 35.1 to a whopping 93.6 KM/h.

| runner_id | avg_km_per_hour |
| --------- | --------------- |
| 1         | 60.00           |
| 1         | 44.44           |
| 1         | 40.20           |
| 1         | 37.50           |
| 2         | 93.60           |
| 2         | 60.00           |
| 2         | 35.10           |
| 3         | 40.00           |

### 7. What is the successful delivery percentage for each runner?

```
WITH clean AS (
SELECT
runner_id,
CASE WHEN cancellation LIKE 'null' or cancellation LIKE '' OR cancellation IS NULL THEN 1 ELSE NULL END AS cancellation
FROM pizza_runner.runner_orders)

SELECT
runner_id,
100 * COUNT(cancellation) / COUNT(*) AS avg
FROM clean
GROUP BY runner_id
ORDER BY runner_id
```
The CTE basically flips all of the contents of the `cancellation` column, making non-null values null and making null values 1. This allowed me to easily count up the amount of orders that were successfully delivered. I found the average by dividing the number of orders that were not cancelled by the total amount of orders, and then I grouped them per runner.

| runner_id | avg |
| --------- | --- |
| 1         | 100 |
| 2         | 75  |
| 3         | 50  |

## C. Ingredient Optimization
### 1. What are the standard ingredients for each pizza?

```
WITH clean AS (
SELECT
pn.pizza_name,
CAST(UNNEST(STRING_TO_ARRAY(pr.toppings, ',')) AS INTEGER) AS topping_id
FROM pizza_runner.pizza_names pn
JOIN pizza_runner.pizza_recipes pr
ON pn.pizza_id = pr.pizza_id)

SELECT
c.pizza_name,
pt.topping_name
FROM clean c
JOIN pizza_runner.pizza_toppings pt
ON c.topping_id = pt.topping_id
ORDER BY c.pizza_name, pt.topping_id
```
Since the pizza ingredients are all in one column and separated by commas, they needed to be separated. I did so using UNNEST. Using STRING_TO_ARRAY, it found all of the ingredients separated by commas and put them in separate rows. I then converted them to integers so that they could easily be joined with the `pizza_toppings` table in order to easily identify which number corresponds to which ingredient.

| pizza_name | topping_name |
| ---------- | ------------ |
| Meatlovers | Bacon        |
| Meatlovers | BBQ Sauce    |
| Meatlovers | Beef         |
| Meatlovers | Cheese       |
| Meatlovers | Chicken      |
| Meatlovers | Mushrooms    |
| Meatlovers | Pepperoni    |
| Meatlovers | Salami       |
| Vegetarian | Cheese       |
| Vegetarian | Mushrooms    |
| Vegetarian | Onions       |
| Vegetarian | Peppers      |
| Vegetarian | Tomatoes     |
| Vegetarian | Tomato Sauce |

### 2. What was the most commonly added extra?

```
WITH clean AS (
SELECT
order_id,
CASE WHEN extras LIKE '' OR extras LIKE 'null' THEN null ELSE extras END AS extras
FROM pizza_runner.customer_orders),

separate AS (
SELECT
order_id,
CAST(UNNEST(STRING_TO_ARRAY(extras, ',')) AS INTEGER) AS topping_id
FROM clean)

SELECT
topping_name,
COUNT(order_id) AS amount
FROM separate s
JOIN pizza_runner.pizza_toppings pt
ON s.topping_id = pt.topping_id
GROUP BY topping_name
ORDER BY amount DESC
```
Since this required lots of cleaning, I used two CTEs. The first one fixes all the inconsistent null values in the `extras` column. The second one separates the topping ids, since they are in the same column and separated by commas, similar to the previous question. The main query joins it with the pizza toppings table, matches the topping ids with their name, and groups them based on how many times they are counted.

| topping_name | amount |
| ------------ | ------ |
| Bacon        | 4      |
| Chicken      | 1      |
| Cheese       | 1      |

### 3. What was the most common exclusion?

```
WITH clean AS (
SELECT
order_id,
CASE WHEN exclusions LIKE '' OR exclusions LIKE 'null' THEN null ELSE exclusions END AS exclusions
FROM pizza_runner.customer_orders),

separate AS (
SELECT
order_id,
CAST(UNNEST(STRING_TO_ARRAY(exclusions, ',')) AS INTEGER) AS topping_id
FROM clean)

SELECT
topping_name,
COUNT(order_id) AS amount
FROM separate s
JOIN pizza_runner.pizza_toppings pt
ON s.topping_id = pt.topping_id
GROUP BY topping_name
ORDER BY amount DESC
```
This uses the exact same queries as the prior question but with the first two queries referencing the `exclusions` column instead of `extras`.

| topping_name | amount |
| ------------ | ------ |
| Cheese       | 4      |
| Mushrooms    | 1      |
| BBQ Sauce    | 1      |

### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```
WITH clean AS (
SELECT
ROW_NUMBER() OVER() AS row,
order_id,
pizza_id,
CASE WHEN exclusions LIKE '' OR exclusions LIKE 'null' THEN null ELSE exclusions END AS exclusions,
  CASE WHEN extras LIKE '' OR extras LIKE 'null' THEN null ELSE extras END AS extras
FROM pizza_runner.customer_orders),

temp2 AS (SELECT
row,
CAST(UNNEST(STRING_TO_ARRAY(exclusions, ',')) AS INTEGER) AS exclusions,
CAST(UNNEST(STRING_TO_ARRAY(extras, ',')) AS INTEGER) AS extras
FROM clean),

temp3 AS (SELECT
row,
STRING_AGG(topping_name, ', ') AS exclusions
FROM temp2 t2
JOIN pizza_runner.pizza_toppings pt
ON t2.exclusions = pt.topping_id
GROUP BY row),

temp4 AS (SELECT
row,
STRING_AGG(topping_name, ', ') AS extras
FROM temp2 t2
JOIN pizza_runner.pizza_toppings pt
ON t2.extras = pt.topping_id
GROUP BY row)

SELECT
c.order_id,
CONCAT(
  CASE WHEN pn.pizza_name LIKE 'Meatlovers' THEN 'Meat Lovers' ELSE pn.pizza_name END,
  CASE WHEN t3.exclusions IS NOT NULL THEN ' - Exclude ' ELSE NULL END, t3.exclusions,
  CASE WHEN t4.extras IS NOT NULL THEN ' - Extra ' ELSE NULL END, t4.extras) AS order
FROM clean c
LEFT JOIN temp3 t3
ON c.row = t3.row
LEFT JOIN temp4 t4
ON c.row = t4.row
JOIN pizza_runner.pizza_names pn
ON c.pizza_id = pn.pizza_id
```
This took a LOT of CTE tables, but it works!
- clean - Cleans data and pulls relevant information. I also added a row number column, which is important for grouping ingredients back together later on.
- temp2 - Separates all exclusions and extras into separate columns so that they can be properly identified with the `pizza_toppings` table.
- temp3 - Matches ingredient numbers from the `exclusions` column and groups them back together based on the initial row number in the case where there are multiple exclusions.
- temp4 - Does the same thing as temp3 but with the `extras` column instead.
- Main Query - Where it all comes together. Exclusions are pulled from temp3, and extras are pulled from temp4. The previous queries already formatted properly with commas separating them. A CONCAT function brings it all together, using CASE statements to add hyphens and the "Exclude/Extra" text if there are any in the given order.

| order_id | order                                                            |
| -------- | ---------------------------------------------------------------- |
| 1        | Meat Lovers                                                      |
| 2        | Meat Lovers                                                      |
| 3        | Meat Lovers                                                      |
| 3        | Vegetarian                                                       |
| 4        | Meat Lovers - Exclude Cheese                                     |
| 4        | Meat Lovers - Exclude Cheese                                     |
| 4        | Vegetarian - Exclude Cheese                                      |
| 5        | Meat Lovers - Extra Bacon                                        |
| 6        | Vegetarian                                                       |
| 7        | Vegetarian - Extra Bacon                                         |
| 8        | Meat Lovers                                                      |
| 9        | Meat Lovers - Exclude Cheese - Extra Bacon, Chicken              |
| 10       | Meat Lovers                                                      |
| 10       | Meat Lovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

```
WITH clean AS (
SELECT
ROW_NUMBER() OVER() AS row,
co.order_id,
co.pizza_id,
CASE WHEN exclusions LIKE '' OR exclusions LIKE 'null' THEN null ELSE exclusions END AS exclusions,
CASE WHEN extras LIKE '' OR extras LIKE 'null' THEN null ELSE extras END AS extras,
pr.toppings
FROM pizza_runner.customer_orders co
  JOIN pizza_runner.pizza_recipes pr
  ON co.pizza_id = pr.pizza_id
ORDER BY order_id),
  
organize AS (SELECT
row,
c.order_id,
pn.pizza_name,
topping_name,
CASE WHEN topping_id IN (
  SELECT CAST(UNNEST(STRING_TO_ARRAY(pr.toppings, ',')) AS INTEGER)) THEN 1 ELSE NULL END AS toppings,
CASE WHEN topping_id IN (
  SELECT CAST(UNNEST(STRING_TO_ARRAY(exclusions, ',')) AS INTEGER)) THEN 1 ELSE NULL END AS exclusions,
CASE WHEN topping_id IN (
  SELECT CAST(UNNEST(STRING_TO_ARRAY(extras, ',')) AS INTEGER)) THEN 1 ELSE NULL END AS extras
FROM
clean c,
pizza_runner.pizza_toppings pt,
pizza_runner.pizza_recipes pr
JOIN pizza_runner.pizza_names pn ON pr.pizza_id = pn.pizza_id
WHERE c.pizza_id = pr.pizza_id
ORDER BY row),

filter AS (SELECT
row,
order_id,
CASE WHEN pizza_name LIKE 'Meatlovers' THEN 'Meat Lovers' ELSE pizza_name END AS pizza_name,
topping_name,
toppings,
extras,
exclusions,
CASE WHEN toppings = 1 AND extras = 1 THEN CONCAT('2x',topping_name)
WHEN exclusions = 1 THEN NULL
WHEN toppings = 1 OR extras = 1 THEN topping_name ELSE NULL END AS topping
FROM organize)

SELECT
order_id,
CONCAT(pizza_name, ': ', STRING_AGG(topping, ', ' ORDER BY topping ASC)) AS order
FROM filter
GROUP BY pizza_name, order_id, row
ORDER BY order_id
```
This one was quite complicated, and I did use some outside help for the subqueries used in the `organize` CTE on this question and the next question. The first CTE cleans the data. The `organize` CTE lists out every pizza ordered along with the right type of pizza, and each pizza has a row for each ingredient, whether it's included or not. The subqueries mark whether each topping is included as a standard topping, exclusion, or extra. If they are, they are marked with a 1. The `filter` CTE filters out toppings based on where 1 values are included; if they are in extras and toppings, then it adds '2x'. Finally, they are brought together with STRING_AGG function separating each ingredient by commas.

| order_id | order                                                                                |
| -------- | ------------------------------------------------------------------------------------ |
| 1        | Meat Lovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 2        | Meat Lovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 3        | Meat Lovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 3        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes               |
| 4        | Meat Lovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| 4        | Meat Lovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| 4        | Vegetarian: Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                       |
| 5        | Meat Lovers: 2xBacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes               |
| 7        | Vegetarian: Bacon, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes        |
| 8        | Meat Lovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 9        | Meat Lovers: 2xBacon, 2xChicken, BBQ Sauce, Beef, Mushrooms, Pepperoni, Salami       |
| 10       | Meat Lovers: 2xBacon, 2xCheese, Beef, Chicken, Pepperoni, Salami                     |
| 10       | Meat Lovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |

### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

```
WITH clean AS (SELECT
ROW_NUMBER() OVER() AS row,
co.order_id,
co.pizza_id,
CASE WHEN exclusions LIKE '' OR exclusions LIKE 'null' THEN null ELSE exclusions END AS exclusions,
CASE WHEN extras LIKE '' OR extras LIKE 'null' THEN null ELSE extras END AS extras,
pr.toppings
FROM pizza_runner.customer_orders co
  JOIN pizza_runner.pizza_recipes pr
  ON co.pizza_id = pr.pizza_id
JOIN pizza_runner.runner_orders ro
  ON co.order_id = ro.order_id
WHERE CASE WHEN ro.cancel lation LIKE 'null' or ro.cancellation LIKE '' THEN null ELSE ro.cancellation END IS NULL
ORDER BY order_id),

data AS (
SELECT
row,
order_id,
topping_name,
CASE WHEN topping_id IN (
  SELECT CAST(UNNEST(STRING_TO_ARRAY(exclusions, ',')) AS INTEGER)) THEN 0
ELSE CASE WHEN topping_id IN (
  SELECT CAST(UNNEST(STRING_TO_ARRAY(toppings, ',')) AS INTEGER)) THEN COUNT(topping_name)
ELSE 0 END END AS topping_count,
CASE WHEN topping_id IN (
  SELECT CAST(UNNEST(STRING_TO_ARRAY(extras, ',')) AS INTEGER)) THEN COUNT(topping_name)
ELSE 0 END AS extra_count
FROM clean c, pizza_runner.pizza_toppings pt
GROUP BY c.row, pt.topping_name, c.exclusions, pt.topping_id, c.toppings, c.extras, c.order_id)

SELECT
topping_name,
SUM(topping_count) + SUM(extra_count) AS total
FROM data
GROUP BY topping_name
ORDER BY total DESC
```
This was pretty tricky and I honestly had to get some outside help for the subqueries in the `data` CTE, but I was able to figure it out. As always, the `clean` CTE cleans and joins all the data. The `data` CTE is where it all comes together. Basically, each pizza has a row for each ingredient. For example, pizza '1' has a row for Bacon, BBQ Sauce, Cheese, and so on. The CASE statements with subqueries search for if the topping id exists in the list of exclusions, extras, or standard pizza toppings for each pizza. If it does, it counts them and adds it to a column, with the exception of `exclusions`, where it removes the value instead. With that, they are then summed and grouped by topping name to show the total amount of each topping used.

| topping_name | total |
| ------------ | ----- |
| Bacon        | 12    |
| Mushrooms    | 11    |
| Cheese       | 10    |
| Pepperoni    | 9     |
| Chicken      | 9     |
| Salami       | 9     |
| Beef         | 9     |
| BBQ Sauce    | 8     |
| Tomato Sauce | 3     |
| Onions       | 3     |
| Tomatoes     | 3     |
| Peppers      | 3     |

## D. Pricing and Ratings
### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

```
SELECT
    SUM(CASE WHEN pn.pizza_name LIKE 'Meatlovers' THEN 12
    WHEN pn.pizza_name LIKE 'Vegetarian' THEN 10 ELSE 0 END) AS total
FROM pizza_runner.customer_orders co
JOIN pizza_runner.pizza_names pn
    ON co.pizza_id = pn.pizza_id
JOIN pizza_runner.runner_orders ro
ON co.order_id = ro.order_id
WHERE CASE WHEN ro.cancellation LIKE 'null' or ro.cancellation LIKE '' THEN null ELSE ro.cancellation END IS NULL
```
Pretty simple query. If an order is 'Meatlovers' it assigns $12 to the total column. If it's 'Vegetarian', it becomes $10. The values of the column are added up to give the total amount made from all orders. A WHERE clause is also used to filter out orders that were not successfully delivered.

| total |
| ----- |
| 138   |

### 2. What if there was an additional $1 charge for any pizza extras?
- Add cheese is $1 extra

```
WITH prices AS (
SELECT
    SUM(CASE WHEN pn.pizza_name LIKE 'Meatlovers' THEN 12
    WHEN pn.pizza_name LIKE 'Vegetarian' THEN 10 ELSE 0 END) AS total
FROM pizza_runner.customer_orders co
JOIN pizza_runner.pizza_names pn
    ON co.pizza_id = pn.pizza_id
JOIN pizza_runner.runner_orders ro
ON co.order_id = ro.order_id
WHERE CASE WHEN ro.cancellation LIKE 'null' or ro.cancellation LIKE '' THEN null ELSE ro.cancellation END IS NULL),

extra_cost AS (
SELECT
    UNNEST(STRING_TO_ARRAY(CASE WHEN extras LIKE '' OR extras LIKE 'null' THEN null ELSE extras END, ',')) AS extras
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro
ON co.order_id = ro.order_id
WHERE CASE WHEN ro.cancellation LIKE 'null' or ro.cancellation LIKE '' THEN null ELSE ro.cancellation END IS NULL)

SELECT
    total + COUNT(extras) AS total
FROM prices, extra_cost
GROUP BY total
```
This uses two CTE tables - the first one is just like the prior question and grabs the total from pizzas delivered. The second one separates all the extras into one column, which is then counted in the main query to get the total amount; since each extra is $1, COUNT works great here.

| total |
| ----- |
| 142   |

### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

```
DROP TABLE IF EXISTS ratings;
CREATE TABLE ratings (
  "order_id" INTEGER,
  "rating" INTEGER
);
INSERT INTO ratings
  ("order_id", "rating")
VALUES
  (1, 5),
  (2, 5),
  (3, 3),
  (4, 4),
  (5, 4),
  (7, 5),
  (8, 2),
  (10, 3);
```
To do this, I simply created the `ratings` table using `order_id` and `rating` as the columns. `order_id` would match with other tables, allowing them to easily be joined. I then inserted values for each successful order (6 and 9 were cancelled) and added random ratings.

| order_id | rating |
| -------- | ------ |
| 1        | 5      |
| 2        | 5      |
| 3        | 3      |
| 4        | 4      |
| 5        | 4      |
| 7        | 5      |
| 8        | 2      |
| 10       | 3      |

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

```
WITH clean AS (
SELECT
order_id,
runner_id,
CASE WHEN pickup_time LIKE 'null' THEN null ELSE CAST(pickup_time AS TIMESTAMP) END AS pickup_time,
CAST(CASE WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
  WHEN distance LIKE 'null' THEN NULL ELSE distance END AS DECIMAL) AS distance,
CAST(CASE WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
WHEN duration LIKE 'null' THEN NULL ELSE duration END AS DECIMAL) AS duration
FROM pizza_runner.runner_orders)
  
SELECT 
co.customer_id,
co.order_id,
c.runner_id,
r.rating,
co.order_time,
c.pickup_time,
DATE_PART('minute', c.pickup_time - co.order_time) AS time_from_order_and_pickup,
c.duration AS delivery_duration,
ROUND(c.distance / c.duration * 60, 2) AS avg_speed,
COUNT(co.customer_id) AS pizza_count
FROM pizza_runner.customer_orders co
LEFT JOIN pizza_runner.ratings r
ON co.order_id = r.order_id
LEFT JOIN clean c
ON co.order_id = c.order_id
GROUP BY co.order_id, customer_id, order_time, r.rating, c.runner_id, c.pickup_time, c.duration, c.distance
ORDER BY co.order_id
```
The CTE cleans the `runner_orders` table, and the rest of the information is pulled from other tables with the necessary information. Many of these formulas were able to be taken from prior questions.

| customer_id | order_id | runner_id | rating | order_time               | pickup_time              | time_from_order_and_pickup | delivery_duration | avg_speed | pizza_count |
| ----------- | -------- | --------- | ------ | ------------------------ | ------------------------ | -------------------------- | ----------------- | --------- | ----------- |
| 101         | 1        | 1         | 5      | 2020-01-01T18:05:02.000Z | 2020-01-01T18:15:34.000Z | 10                         | 32                | 37.50     | 1           |
| 101         | 2        | 1         | 5      | 2020-01-01T19:00:52.000Z | 2020-01-01T19:10:54.000Z | 10                         | 27                | 44.44     | 1           |
| 102         | 3        | 1         | 3      | 2020-01-02T23:51:23.000Z | 2020-01-03T00:12:37.000Z | 21                         | 20                | 40.20     | 2           |
| 103         | 4        | 2         | 4      | 2020-01-04T13:23:46.000Z | 2020-01-04T13:53:03.000Z | 29                         | 40                | 35.10     | 3           |
| 104         | 5        | 3         | 4      | 2020-01-08T21:00:29.000Z | 2020-01-08T21:10:57.000Z | 10                         | 15                | 40.00     | 1           |
| 101         | 6        | 3         |        | 2020-01-08T21:03:13.000Z |                          |                            |                   |           | 1           |
| 105         | 7        | 2         | 5      | 2020-01-08T21:20:29.000Z | 2020-01-08T21:30:45.000Z | 10                         | 25                | 60.00     | 1           |
| 102         | 8        | 2         | 2      | 2020-01-09T23:54:33.000Z | 2020-01-10T00:15:02.000Z | 20                         | 15                | 93.60     | 1           |
| 103         | 9        | 2         |        | 2020-01-10T11:22:59.000Z |                          |                            |                   |           | 1           |
| 104         | 10       | 1         | 3      | 2020-01-11T18:34:49.000Z | 2020-01-11T18:50:20.000Z | 15                         | 10                | 60.00     | 2           |

### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

```
WITH pizza_price AS (SELECT
    SUM(CASE WHEN pn.pizza_name LIKE 'Meatlovers' THEN 12
    WHEN pn.pizza_name LIKE 'Vegetarian' THEN 10 ELSE 0 END) AS total
FROM pizza_runner.customer_orders co
JOIN pizza_runner.pizza_names pn
    ON co.pizza_id = pn.pizza_id
JOIN pizza_runner.runner_orders ro
ON co.order_id = ro.order_id
WHERE CASE WHEN ro.cancellation LIKE 'null' or ro.cancellation LIKE '' THEN null ELSE ro.cancellation END IS NULL)

SELECT
total - SUM(CAST(CASE WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
  WHEN distance LIKE 'null' THEN NULL ELSE distance END AS DECIMAL)) * 0.3 AS balance
FROM pizza_runner.runner_orders, pizza_price
GROUP BY total
```
The CTE uses the same query as question 1 to find the amount made from successfully delivered pizzas. It then finds the total amount of kilometers travelled, multiplies it by 0.3 (since it's $0.30 per km) and subtracts that total from the pizza sales total.

| balance |
| ------- |
| 94.44   |

## E. Bonus Questions
### If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

```
INSERT INTO pizza_runner.pizza_names
  ("pizza_id", "pizza_name")
VALUES
  (3, 'Supreme');

INSERT INTO pizza_runner.pizza_recipes
  ("pizza_id", "toppings")
VALUES
  (3, '1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12');
```
Rows would be inserted into the `pizza_names` and `pizza_recipes` tables to account for the new type of pizza. Nothing else would need to be modified.
