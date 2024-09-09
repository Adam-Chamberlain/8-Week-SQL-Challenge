This page showcases the problems within all eight case studies that were the most challenging to solve and required the most complex and creative SQL queries. I also explained my thought process in each query much more in-depth.

# Table of Contents

### **[🍜 Case Study 1: Danny's Diner](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#1--dannys-diner)**
- **[Question 1 - Member Point Multiplier]([https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#2-in-the-first-week-after-a-customer-becomes-a-member-including-their-join-date-they-earn-2x-points-on-all-items-each-1-spent-equates-to-10-points-and-sushi-has-a-permanent-2x-point-multiplier-how-many-points-do-customers-a-and-b-have-at-the-end-of-january](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/edit/main/Highlights.md#1-in-the-first-week-after-a-customer-becomes-a-member-including-their-join-date-they-earn-2x-points-on-all-items-each-1-spent-equates-to-10-points-and-sushi-has-a-permanent-2x-point-multiplier-how-many-points-do-customers-a-and-b-have-at-the-end-of-january))**
### **[🍕 Case Study 2: Pizza Runner](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#2--pizza-runner)**
- **[Question 1 - List of Exclusions/Extras per Order](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#1-generate-an-order-item-for-each-record-in-the-customers_orders-table-in-the-format-of-one-of-the-following)**
- **[Question 2 - List of Ingredients per Order](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#2-generate-an-alphabetically-ordered-comma-separated-ingredient-list-for-each-pizza-order-from-the-customer_orders-table-and-add-a-2x-in-front-of-any-relevant-ingredients)**
### **[📺 Case Study 3: Foodie-Fi](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#-3-foodie-fi)**
### **[🏦 Case Study 4: Data Bank](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#-4-data-bank)**
### **[🏪 Case Study 5: Data Mart](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#-5-data-mart)**
### **[🦞 Case Study 6: Clique Bait](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#-6-clique-bait)**
### **[🧥 Case Study 7: Balanced Tree Clothing Co.](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#-7-balanced-tree-clothing-co)**
### **[🖱️ Case Study 8: Fresh Segments](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#%EF%B8%8F-8-fresh-segments)**

# 1. 🍜 Danny's Diner

This case study looks at a restaurant that has small amounts of data on customers, whether or not they are a member, and what they have ordered from the limited menu. As this was the first case study of eight, the questions were much more simple. Note that I used the provided DB Fiddle for the first two case studies, and I switched to MySQL for the remaining six.

**[Case Study Website](https://8weeksqlchallenge.com/case-study-1/)**

**[All of my Solutions](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/%231%20-%20Danny's%20Diner/README.md)**

## 1. In the first week after a customer becomes a member (including their join date), they earn 2x points on all items. Each $1 spent equates to 10 points, and sushi has a permanent 2x point multiplier. How many points do customers A and B have at the end of January?

This has a lot of variables taken into account when figuring out how many points each order gives. Also, it does not specify whether customers gain points BEFORE they become a member (I assumed it did not) or whether sushi multipliers stack for a 4x multiplier. (again, I assumed it did not)

```sql
WITH point_calc AS (
  SELECT
    s.customer_id,
    m.product_name,
    m.price,
    m.price * 10 +
    (CASE WHEN s.order_date BETWEEN ms.join_date AND ms.join_date + 6 THEN m.price * 10
      WHEN m.product_name = 'sushi' THEN m.price * 10 ELSE 0 END) AS points,
    s.order_date,
    ms.join_date
  FROM dannys_diner.sales s
  JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
  JOIN dannys_diner.members ms
    ON s.customer_id = ms.customer_id
  WHERE s.order_date < '2021-02-01'
    AND s.order_date >= ms.join_date)

SELECT
  customer_id,
  SUM(points) AS point_total
FROM point_calc
GROUP BY customer_id
ORDER BY customer_id
```

**point_calc**: This uses a CASE statement with multiple criteria: If the order took place on the day they became a member or any of the six days after, the points get multiplied 2x. Alternatively, if the order is sushi, it gets the same 2x multiplier. If neither happen, they are not multiplied. Orders after January are also filtered out as well as orders before each customer became a member.

Only the `customer_id` and `points` columns are necessary here, but I added the rest to give a better understanding of how it works.
| customer_id | product_name | price | points | order_date               | join_date                |
| ----------- | ------------ | ----- | ------ | ------------------------ | ------------------------ |
| B           | sushi        | 10    | 200    | 2021-01-11T00:00:00.000Z | 2021-01-09T00:00:00.000Z |
| A           | curry        | 15    | 300    | 2021-01-07T00:00:00.000Z | 2021-01-07T00:00:00.000Z |
| A           | ramen        | 12    | 240    | 2021-01-11T00:00:00.000Z | 2021-01-07T00:00:00.000Z |
| A           | ramen        | 12    | 240    | 2021-01-11T00:00:00.000Z | 2021-01-07T00:00:00.000Z |
| A           | ramen        | 12    | 240    | 2021-01-10T00:00:00.000Z | 2021-01-07T00:00:00.000Z |
| B           | ramen        | 12    | 120    | 2021-01-16T00:00:00.000Z | 2021-01-09T00:00:00.000Z |


Lastly, the main query puts it together by summing up the points.
| customer_id | point_total |
| ----------- | ----------- |
| A           | 1020        |
| B           | 320         |

# 2. 🍕 Pizza Runner

This case study focuses on a pizza delivery company. There are multiple questions that I am going to focus on from here - despite only being the second case study, it had some of the hardest challenges overall! Note that I used the provided DB Fiddle for the first two case studies, and I switched to MySQL for the remaining six.

**[Case Study Website](https://8weeksqlchallenge.com/case-study-2/)**

**[All of my Solutions](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/%232%20-%20Pizza%20Runner/README.md)**

## 1. Generate an order item for each record in the customers_orders table in the format of one of the following:
- **Meat Lovers**
- **Meat Lovers - Exclude Beef**
- **Meat Lovers - Extra Bacon**
- **Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers**

For context, this is what the `customer_orders` table looks like initially. In order to solve this, the exclusions and extras need to be separated, identified using the `pizza_toppings` table, and put back together with specific additions in the text if criteria is met.
| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        | null       | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        | null       | null   | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        | null       | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        | null       | null   | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        | null       | null   | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |

And here is the full SQL query.
```sql
WITH clean AS (
SELECT
  ROW_NUMBER() OVER() AS row,
  order_id,
  pizza_id,
  CASE WHEN exclusions LIKE '' OR exclusions LIKE 'null' THEN null ELSE exclusions END AS exclusions,
    CASE WHEN extras LIKE '' OR extras LIKE 'null' THEN null ELSE extras END AS extras
FROM pizza_runner.customer_orders),

separate AS (SELECT
  row,
  CAST(UNNEST(STRING_TO_ARRAY(exclusions, ',')) AS INTEGER) AS exclusions,
  CAST(UNNEST(STRING_TO_ARRAY(extras, ',')) AS INTEGER) AS extras
FROM clean)

SELECT
  c.order_id,
  CONCAT(
    CASE WHEN pn.pizza_name LIKE 'Meatlovers' THEN 'Meat Lovers' ELSE pn.pizza_name END,
    CASE WHEN a.exclusions IS NOT NULL THEN ' - Exclude ' ELSE NULL END, a.exclusions,
    CASE WHEN b.extras IS NOT NULL THEN ' - Extra ' ELSE NULL END, b.extras) AS order
FROM clean c
LEFT JOIN (
  SELECT
    row,
    STRING_AGG(topping_name, ', ') AS exclusions
  FROM separate s
  JOIN pizza_runner.pizza_toppings pt
    ON s.exclusions = pt.topping_id
  GROUP BY row) a
    ON c.row = a.row
LEFT JOIN (
    SELECT
    row,
    STRING_AGG(topping_name, ', ') AS extras
  FROM separate s
  JOIN pizza_runner.pizza_toppings pt
    ON s.extras = pt.topping_id
  GROUP BY row) b
  ON c.row = b.row
JOIN pizza_runner.pizza_names pn
  ON c.pizza_id = pn.pizza_id
```
**clean**: The original table has many inconsistencies that need to be cleaned before it is ready for use. Notably, some NULL values are blank, and some are the text "null" rather than the value. I used CASE statements to replace these so that they are all consistent. I also added a row number so that each individual pizza is identified with a number; this becomes important when putting the pieces back together.
| row | order_id | pizza_id | exclusions | extras |
| --- | -------- | -------- | ---------- | ------ |
| 1   | 1        | 1        |            |        |
| 2   | 2        | 1        |            |        |
| 3   | 3        | 1        |            |        |
| 4   | 3        | 2        |            |        |
| 5   | 4        | 1        | 4          |        |
| 6   | 4        | 1        | 4          |        |
| 7   | 4        | 2        | 4          |        |
| 8   | 5        | 1        |            | 1      |
| 9   | 6        | 2        |            |        |
| 10  | 7        | 2        |            | 1      |
| 11  | 8        | 1        |            |        |
| 12  | 9        | 1        | 4          | 1, 5   |
| 13  | 10       | 1        |            |        |
| 14  | 10       | 1        | 2, 6       | 1, 4   |

**separate**: here, the `exclusions` and `extras` columns are split apart by comma values, since some have multiple exclusions or extras. This is used for the subqueries in the final query. Note that there are now multiple of the same `row` values, since one pizza's toppings are in multiple rows now.
| row | exclusions | extras |
| --- | ---------- | ------ |
| 5   | 4          |        |
| 6   | 4          |        |
| 7   | 4          |        |
| 8   |            | 1      |
| 10  |            | 1      |
| 12  | 4          | 1      |
| 12  |            | 5      |
| 14  | 2          | 1      |
| 14  | 6          | 4      |

**subqueries**: In the main query, two subqueries are used (labeled `a` and `b`) and joined with the clean dataset. `a` focuses on exclusions, and `b` focuses on extras. They identify the corresponding topping based on the number and combine back together based on the initial row number.
| row | exclusions           |
| --- | -------------------- |
| 5   | Cheese               |
| 6   | Cheese               |
| 14  | BBQ Sauce, Mushrooms |
| 7   | Cheese               |
| 12  | Cheese               |

They are joined based on the row number so that each pizza has the right exclusions and extras. Without the CONCAT function, the table would look like this:
| order_id | pizza_name | exclusions           | extras         |
| -------- | ---------- | -------------------- | -------------- |
| 1        | Meatlovers |                      |                |
| 2        | Meatlovers |                      |                |
| 3        | Meatlovers |                      |                |
| 3        | Vegetarian |                      |                |
| 4        | Meatlovers | Cheese               |                |
| 4        | Meatlovers | Cheese               |                |
| 4        | Vegetarian | Cheese               |                |
| 5        | Meatlovers |                      | Bacon          |
| 6        | Vegetarian |                      |                |
| 7        | Vegetarian |                      | Bacon          |
| 8        | Meatlovers |                      |                |
| 9        | Meatlovers | Cheese               | Bacon, Chicken |
| 10       | Meatlovers |                      |                |
| 10       | Meatlovers | BBQ Sauce, Mushrooms | Bacon, Cheese  |

Lastly, the CONCAT at the start of the main query puts it together. It is organized as such:
- Pizza Name (If it's Meatlovers, the text is changed to Meat Lovers to be consistant with the requested format)
- "- Exclude" - This text is only added if the `exclusions` column is not NULL. It wouldn't make sense to include it if there are no exclusions.
- Exclusions
- "- Extra" - Like exclusions, this is only added if the pizza has extras added.
- Extras

Final output
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

## 2. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients.
**(For example: "Meat Lovers: 2xBacon, Beef, ... , Salami")**

This is an extremely complex challenge that requires lots of steps. First off, all of the necessary data is separated into many tables. The `pizza_recipes` table shows each pizza and their toppings:

| pizza_id | toppings                |
| -------- | ----------------------- |
| 1        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 2        | 4, 6, 7, 9, 11, 12      |

However, the pizza's names are on the `pizza_names` table:

| pizza_id | pizza_name |
| -------- | ---------- |
| 1        | Meatlovers |
| 2        | Vegetarian |

And toppings are listed on the `pizza_toppings` table:

| topping_id | topping_name |
| ---------- | ------------ |
| 1          | Bacon        |
| 2          | BBQ Sauce    |
| 3          | Beef         |
| 4          | Cheese       |
| 5          | Chicken      |
| 6          | Mushrooms    |
| 7          | Onions       |
| 8          | Pepperoni    |
| 9          | Peppers      |
| 10         | Salami       |
| 11         | Tomatoes     |
| 12         | Tomato Sauce |

To do this, the topping numbers must be separated so that they can be identified with the corresponding topping name, and extras and exclusions need to be taken into account as well.

```sql
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
JOIN pizza_runner.pizza_names pn
  ON pr.pizza_id = pn.pizza_id
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
**clean**: Like the last challenge, the data must be cleaned first, since there are many inconsistent NULL values. Additionally, the `customer_orders` table was joined with the `pizza_recipes` table to get the list of ingredients from the selected pizza. Lastly, a row number was added so that each pizza could be put back together at the end.
| row | order_id | pizza_id | exclusions | extras | toppings                |
| --- | -------- | -------- | ---------- | ------ | ----------------------- |
| 7   | 1        | 1        |            |        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 2   | 2        | 1        |            |        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 3   | 3        | 1        |            |        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 11  | 3        | 2        |            |        | 4, 6, 7, 9, 11, 12      |
| 14  | 4        | 2        | 4          |        | 4, 6, 7, 9, 11, 12      |
| 8   | 4        | 1        | 4          |        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 9   | 4        | 1        | 4          |        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 10  | 5        | 1        |            | 1      | 1, 2, 3, 4, 5, 6, 8, 10 |
| 13  | 6        | 2        |            |        | 4, 6, 7, 9, 11, 12      |
| 12  | 7        | 2        |            | 1      | 4, 6, 7, 9, 11, 12      |
| 4   | 8        | 1        |            |        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 5   | 9        | 1        | 4          | 1, 5   | 1, 2, 3, 4, 5, 6, 8, 10 |
| 1   | 10       | 1        | 2, 6       | 1, 4   | 1, 2, 3, 4, 5, 6, 8, 10 |
| 6   | 10       | 1        |            |        | 1, 2, 3, 4, 5, 6, 8, 10 |

**organize**: This is where it gets a bit complex; this query lists each pizza ordered, the corresponding pizza type, and every single ingredient, whether it is on the pizza or not. If the ingredient is included as a standard topping, exclusion, or extra, the column gains the value 1; otherwise, it stays empty.

This pulls from three unjoined tables; every `row` value (each pizza) has 24 columns, since there are two pizza types and 12 ingredients. (12 for Meatlovers and each of the 12 ingredients, and 12 more for Vegetarian). `pizza_recipes` pulls the pizza IDs (which are named via joining the table with `pizza_names`) and ingredients are pulled from `pizza_toppings`. Since each pizza ordered already has an ID, I used a WHERE clause to filter out the other type of pizza, since those rows are not necessary; this way, there are just 12 rows per pizza, each corresponding with the right pizza type.

The CASE statements with subqueries decide whether or not the ingredient is included or not. It lists all the corresponding ingredients from the pizza in a given row. If there is a match, it is given the value 1. For example, Bacon has the value "1", and the "Meatlovers" pizza includes that "1" in its toppings, so bacon is included and is given the value of 1. Similarly, it checks the `exclusions` and `extras` columns for matching values.

Only the first two pizzas are shown, since the table is quite long at this point.
| row | order_id | pizza_name | topping_name | toppings | exclusions | extras |
| --- | -------- | ---------- | ------------ | -------- | ---------- | ------ |
| 1   | 10       | Meatlovers | Bacon        | 1        |            | 1      |
| 1   | 10       | Meatlovers | BBQ Sauce    | 1        | 1          |        |
| 1   | 10       | Meatlovers | Beef         | 1        |            |        |
| 1   | 10       | Meatlovers | Cheese       | 1        |            | 1      |
| 1   | 10       | Meatlovers | Chicken      | 1        |            |        |
| 1   | 10       | Meatlovers | Mushrooms    | 1        | 1          |        |
| 1   | 10       | Meatlovers | Onions       |          |            |        |
| 1   | 10       | Meatlovers | Pepperoni    | 1        |            |        |
| 1   | 10       | Meatlovers | Peppers      |          |            |        |
| 1   | 10       | Meatlovers | Salami       | 1        |            |        |
| 1   | 10       | Meatlovers | Tomatoes     |          |            |        |
| 1   | 10       | Meatlovers | Tomato Sauce |          |            |        |
| 2   | 2        | Meatlovers | Bacon        | 1        |            |        |
| 2   | 2        | Meatlovers | BBQ Sauce    | 1        |            |        |
| 2   | 2        | Meatlovers | Beef         | 1        |            |        |
| 2   | 2        | Meatlovers | Cheese       | 1        |            |        |
| 2   | 2        | Meatlovers | Chicken      | 1        |            |        |
| 2   | 2        | Meatlovers | Mushrooms    | 1        |            |        |
| 2   | 2        | Meatlovers | Onions       |          |            |        |
| 2   | 2        | Meatlovers | Pepperoni    | 1        |            |        |
| 2   | 2        | Meatlovers | Peppers      |          |            |        |
| 2   | 2        | Meatlovers | Salami       | 1        |            |        |
| 2   | 2        | Meatlovers | Tomatoes     |          |            |        |
| 2   | 2        | Meatlovers | Tomato Sauce |          |            |        |

**filter**: This query takes the `toppings`, `extras`, and `exclusions` columns and puts it together to decide what the output is. If the value 1 is shown on both toppings and extras, "2x" is added to the topping name. If all three rows are blank OR the toppings and exclusions columns have 1, it is left blank. (This is because the ingredient starts on the pizza and is then removed) This gives a list of each ingredient present on each pizza, which can be combined using one last query.

Again, only the first two pizzas are shown.
| row | order_id | pizza_name  | topping_name | toppings | extras | exclusions | topping      |
| --- | -------- | ----------- | ------------ | -------- | ------ | ---------- | ------------ |
| 1   | 10       | Meat Lovers | Bacon        | 1        | 1      |            | 2xBacon      |
| 1   | 10       | Meat Lovers | BBQ Sauce    | 1        |        | 1          |              |
| 1   | 10       | Meat Lovers | Beef         | 1        |        |            | Beef         |
| 1   | 10       | Meat Lovers | Cheese       | 1        | 1      |            | 2xCheese     |
| 1   | 10       | Meat Lovers | Chicken      | 1        |        |            | Chicken      |
| 1   | 10       | Meat Lovers | Mushrooms    | 1        |        | 1          |              |
| 1   | 10       | Meat Lovers | Onions       |          |        |            |              |
| 1   | 10       | Meat Lovers | Pepperoni    | 1        |        |            | Pepperoni    |
| 1   | 10       | Meat Lovers | Peppers      |          |        |            |              |
| 1   | 10       | Meat Lovers | Salami       | 1        |        |            | Salami       |
| 1   | 10       | Meat Lovers | Tomatoes     |          |        |            |              |
| 1   | 10       | Meat Lovers | Tomato Sauce |          |        |            |              |
| 2   | 2        | Meat Lovers | Bacon        | 1        |        |            | Bacon        |
| 2   | 2        | Meat Lovers | BBQ Sauce    | 1        |        |            | BBQ Sauce    |
| 2   | 2        | Meat Lovers | Beef         | 1        |        |            | Beef         |
| 2   | 2        | Meat Lovers | Cheese       | 1        |        |            | Cheese       |
| 2   | 2        | Meat Lovers | Chicken      | 1        |        |            | Chicken      |
| 2   | 2        | Meat Lovers | Mushrooms    | 1        |        |            | Mushrooms    |
| 2   | 2        | Meat Lovers | Onions       |          |        |            |              |
| 2   | 2        | Meat Lovers | Pepperoni    | 1        |        |            | Pepperoni    |
| 2   | 2        | Meat Lovers | Peppers      |          |        |            |              |
| 2   | 2        | Meat Lovers | Salami       | 1        |        |            | Salami       |
| 2   | 2        | Meat Lovers | Tomatoes     |          |        |            |              |
| 2   | 2        | Meat Lovers | Tomato Sauce |          |        |            |              |

**Final query**: With all the hard work done, the list of toppings just need to be combined. This is done with a CONCAT to pull the pizza name, add a colon, then list each ingredient, alphabatized and separated by columns. This is able to work because of the `row` column given to each pizza in the first query.

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

# 📺 3. Foodie-Fi

# 🏦 4. Data Bank

# 🏪 5. Data Mart

# 🦞 6. Clique Bait

# 🧥 7. Balanced Tree Clothing Co.

# 🖱️ 8. Fresh Segments
