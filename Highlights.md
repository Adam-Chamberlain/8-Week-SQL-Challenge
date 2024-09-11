This page showcases the problems within all eight case studies that were the most challenging to solve and required the most complex and creative SQL queries. I also explained my thought process in each query much more in-depth.

I have chosen 1-2 of the hardest challenges from each case study. You can view my solutions to all of the challenges in the respective case study folders.

# Table of Contents

### **[🍜 Case Study 1: Danny's Diner](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#1--dannys-diner)**
- **[Question 1 - Member Point Multiplier](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#1-in-the-first-week-after-a-customer-becomes-a-member-including-their-join-date-they-earn-2x-points-on-all-items-each-1-spent-equates-to-10-points-and-sushi-has-a-permanent-2x-point-multiplier-how-many-points-do-customers-a-and-b-have-at-the-end-of-january)**
### **[🍕 Case Study 2: Pizza Runner](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#2--pizza-runner)**
- **[Question 1 - List of Exclusions/Extras per Order](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#1-generate-an-order-item-for-each-record-in-the-customers_orders-table-in-the-format-of-one-of-the-following)**
- **[Question 2 - List of Ingredients per Order](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#2-generate-an-alphabetically-ordered-comma-separated-ingredient-list-for-each-pizza-order-from-the-customer_orders-table-and-add-a-2x-in-front-of-any-relevant-ingredients)**
### **[📺 Case Study 3: Foodie-Fi](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#-3-foodie-fi)**
- **[Question 1 - Average Time between Subscription and Start Date](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#1-how-many-days-on-average-does-it-take-for-a-customer-to-upgrade-to-an-annual-plan-from-the-day-they-join-foodie-fi)**
### **[🏦 Case Study 4: Data Bank](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#-4-data-bank)**
- **[Question 1 - Monthly Closing Balance](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#1-what-is-the-closing-balance-for-each-customer-at-the-end-of-the-month)**
### **[🏪 Case Study 5: Data Mart](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#-5-data-mart)**
- **[Question 1 - Data Cleaning](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#1-data-cleaning)**
- **[Question 2 - % Sales by Demographic](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#2-what-is-the-percentage-of-sales-by-demographic-for-each-year-in-the-dataset)**
### **[🦞 Case Study 6: Clique Bait](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#-6-clique-bait)**
- **[Question 1 - Top 3 Products Purchased](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#1-what-are-the-top-3-products-by-purchases)**
### **[🧥 Case Study 7: Balanced Tree Clothing Co.](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#-7-balanced-tree-clothing-co)**
- **[Question 1 - Most Common Combinations of Three Products](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/blob/main/Highlights.md#1-what-is-the-most-common-combination-of-at-least-1-quantity-of-any-3-products-in-a-1-single-transaction)**
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

This case study looks at a food-based streaming service, and it has data on customers and what subscription plans they have used.

**[Case Study Website](https://8weeksqlchallenge.com/case-study-3/)**

**[All of my Solutions](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/tree/main/%233%20-%20Foodie-Fi)**

## 1. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?

In this database, every single customer starts with a free trial, and then they either upgrade to a basic monthly, pro monthly, or pro annual subscription, or choose to not get a subscription. With that information, I can find the difference between the day they become a pro annual member and the day they started their free trial.

```sql
SELECT
  ROUND(AVG(DATEDIFF(a.start_date, t.start_date)), 0) AS average
FROM (
  SELECT
    customer_id,
    plan_id,
    start_date
  FROM subscriptions
  WHERE plan_id = 3) a
JOIN (
  SELECT
    customer_id,
    plan_id,
    start_date
  FROM subscriptions
  WHERE plan_id = 0) t
  ON a.customer_id = t.customer_id
```
Each subquery pulls rows where a customer signs up for a specific subscription; 3 equals annual membership (shortened to a) and 0 equals free trial. (shortened to t) The two subqueries are joined to show all customer IDs that have done a free trial AND signed up for an annual subscription, as shown in the table below.

![image](https://github.com/user-attachments/assets/e6d42876-5a20-4117-9140-1359dcfff6f0)

With DATEDIFF, I can get the difference between the annual membership start date and the free trial start date for each customer. The average of these differences is the final result.

![image](https://github.com/user-attachments/assets/1b00c06b-262d-4236-9e07-92425991543a)

# 🏦 4. Data Bank

Data bank is a digital bank that stores money as well as data. Customers are given a specific amount of cloud storage depending on how much money they have stored.

**[Case Study Website](https://8weeksqlchallenge.com/case-study-4/)**

**[All of my Solutions](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/tree/main/%234%20-%20Data%20Bank)**

## 1. What is the closing balance for each customer at the end of the month?

This was probably the hardest challenge within all eight case studies. The `customer_transactions` table has a list of all transactions that took place within a 4 month timeframe, as shown in the image below. This includes purchases, deposits, and withdrawals. The question is asking for a closing balance for each customer in each month; this is tricky because not all customers have transactions within each month.

![image](https://github.com/user-attachments/assets/b8e9dd69-d0a2-4c9f-a1f4-baf98fb94f5d)

```sql
CREATE TABLE temp_table (
monthnum INT PRIMARY KEY,
monthdate DATE);

INSERT INTO temp_table
  (monthnum, monthdate)
VALUES
  (1, '2020-01-31'),
  (2, '2020-02-29'),
  (3, '2020-03-31'),
  (4, '2020-04-30');
  
WITH cte AS (
SELECT
  customer_id,
  MONTH(txn_date) AS totalnum,
  SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) +
  SUM(CASE WHEN txn_type IN('purchase', 'withdrawal') THEN (-1 * txn_amount) ELSE 0 END) AS total
FROM customer_transactions
GROUP BY customer_id, totalnum
ORDER BY customer_id, totalnum),

alldates AS (
SELECT DISTINCT
  customer_id,
  monthnum,
  totalnum,
  monthdate,
  total
FROM cte, temp_table
ORDER BY customer_id, monthnum),

clean AS (
SELECT
  ad.customer_id,
  ad.monthnum,
  ad.totalnum,
  ad.monthdate AS end_month,
  c.total AS monthly_total,
  ad.total,
  SUM(ad.total) OVER (PARTITION BY ad.customer_id, ad.monthnum) AS end_balance
FROM alldates ad
LEFT JOIN cte c
  ON ad.monthnum = c.totalnum AND ad.customer_id = c.customer_id
WHERE ad.monthnum >= ad.totalnum)

SELECT DISTINCT
  customer_id,
  end_month,
  CASE WHEN monthly_total IS NULL THEN 0 ELSE monthly_total END AS monthly_change,
  end_balance
FROM clean
```
**temp_table**: There's a lot to break down here. First of all, I created a temporary table to list the final date of each month. This allows me to create a row for each customer in each month, regardless of if they had any transactions in a given month.

**cte**: The first CTE creates a monthly total for each month that a customer has a transaction. CASE statements are used to determine what type of transaction it is; purchases and withdrawals are subtracted, and deposits are added. `totalnum` refers to the month that each transaction took place in; this will make more sense in the next two CTEs.

![image](https://github.com/user-attachments/assets/8b0642f9-f8b3-4c2e-97d5-2181c8b1967a)

**alldates**: The first CTE is then merged with the temporary table to create a corresponding month row for each existing row. For each month that had a transaction, there are now four rows, one for each month. While this seems messy, this is extremely important for the next CTE.

The `monthnum` column corresponds to `monthdate`, and the `totalnum` column corresponds to the month that the `total` is from.

![image](https://github.com/user-attachments/assets/25ea71d6-4098-4a7d-8c72-e6496fd082a7)

**clean**: The `alldates` CTE is merged with the original CTE (named `cte`) via a LEFT JOIN to pull the total for each month, even if there were no transactions in a given month. This is named `monthly_total`. The other `total` column from the `alldates` CTE is used to calculate the end balance.

`end_balance` is the tricky part. In the below example, there is only one `monthnum` for January and February, since the `totalnum` value for March is not included. (The WHERE clause removes that value, since it had not happened yet) Therefore, the end balance for both months is 312. For March, however, there is a `monthnum` row for both January's and March's transactions. These are added up using PARTITION BY to create an end balance of -640. The duplicate rows are removed in the final query.

![image](https://github.com/user-attachments/assets/de70c2c1-aeb2-4aee-81b9-df4579ea5796)

**Final Query**: Finally, the `clean` CTE has the duplicate rows removed and the NULL values in the monthly totals changed to 0.

![image](https://github.com/user-attachments/assets/c643c95f-a974-434e-9277-d35ce3297a14)

# 🏪 5. Data Mart

This case study looks at an online supermarket that has data from 2018 through 2020 within one large table. Unlike other case studies, this just has one large table, but it needed to be cleaned heavily beforehand.

**[Case Study Website](https://8weeksqlchallenge.com/case-study-5/)**

**[All of my Solutions](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/tree/main/%235%20-%20Data%20Mart)**

## 1. Data Cleaning

In a single query, perform the following operations:
- Convert the week_date to a DATE format
- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a month_number with the calendar month for each week_date value as the 3rd column
- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value: 1 - Young Adults, 2 - Middle Aged, 3 or 4 - Retirees
- Add a new demographic column using the following mapping for the first letter in the segment values: C - Couples, F - Families
- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record

Here is what the original table looks like:

![image](https://github.com/user-attachments/assets/5273019b-645b-4b19-89a3-220b293126b2)

```sql
SELECT
  clean_date AS week_date,
  WEEK(clean_date) AS week_number,
  MONTH(clean_date) AS month_number,
  YEAR(clean_date) AS calendar_year,
  region,
  platform,
  CASE WHEN segment LIKE 'null' THEN NULL ELSE segment END AS segment,
  CASE WHEN RIGHT(segment, 1) = 1 THEN 'Young Adults'
    WHEN RIGHT(segment, 1) = 2 THEN 'Middle Aged'
    WHEN RIGHT(segment, 1) IN(3, 4) THEN 'Retirees' ELSE NULL END AS age_band,
  CASE WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
    WHEN LEFT(segment, 1) = 'F' THEN 'Families' ELSE NULL END AS demographic,
  customer_type,
  transactions,
  sales,
  ROUND(sales / transactions, 2) AS avg_transaction
FROM (
  SELECT *,
    CAST(CONCAT(
      RIGHT(week_date, 2), '/',
      CASE WHEN MID(week_date, 3, 1) = '/' AND MID(week_date, 6, 1) = '/' THEN MID(week_date, 4, 2)
      WHEN MID(week_date, 3, 1) = '/' AND MID(week_date, 5, 1) = '/' THEN MID(week_date, 4, 1)
      WHEN MID(week_date, 2, 1) = '/' AND MID(week_date, 5, 1) = '/' THEN MID(week_date, 3, 2)
      ELSE MID(week_date, 3, 1) END, '/',
      CASE WHEN MID(week_date, 3, 1) = '/' THEN LEFT(week_date, 2) ELSE LEFT(week_date, 1) END)
    AS DATE) AS clean_date
  FROM weekly_sales) clean
```
First off, the date was formated as dd/mm/yy rather than yyyy/mm/dd, and it was not formatted in DATE format. I started off by creating a subquery that rearranges the date using LEFT, RIGHT, and MID. The problem is, the amount of characters was inconsistent between dates. (for example: 31/12/20 and 8/2/20) I used CASE statements to find the location of the backslashes, which would determine what characters would be pulled next. If it finds the first backslash at character 3 (when the date is 2 digits long) AND the second backslash at character 6 (when the month is also 2 digits long) it pulls the 4th and 5th digit as the date. I then used CAST to convert the results to the proper DATE format. This allowed me to format the first four columns.

For the rest, I used the `segment` column to get age and demographic data using LEFT and RIGHT. For example, the value C1 would mean the customers are a couple (C) and young adults (1). The NULL values also had to be changed, since it was the text "null" rather than the actual value.

![image](https://github.com/user-attachments/assets/aec043e1-c19d-41e1-a35d-1067309a4a94)

## 2. What is the percentage of sales by demographic for each year in the dataset?

With the cleaned dataset from the prior question, it is much easier to find key trends such as the percentage of sales by demographic. To solve this, I first find the total amount of sales per demographic in each year, and I can then use those numbers to find the percentages.

```sql
WITH cte AS (
  SELECT
    cl.calendar_year,
    SUM(sales) AS total_sales,
    c_sales,
    f_sales
  FROM clean cl
  JOIN (
    SELECT
      calendar_year,
      SUM(sales) AS c_sales
    FROM clean
    WHERE demographic = 'Couples'
    GROUP BY calendar_year) c
  ON cl.calendar_year = c.calendar_year
  JOIN (
    SELECT
      calendar_year,
      SUM(sales) AS f_sales
    FROM clean
    WHERE demographic = 'Families'
    GROUP BY calendar_year) f
  ON cl.calendar_year = f.calendar_year
  GROUP BY cl.calendar_year, c_sales, f_sales)
  
  SELECT
  calendar_year,
  ROUND(c_sales / total_sales * 100, 2) AS couples,
  ROUND(f_sales / total_sales * 100, 2) AS families,
  ROUND((1 - ((c_sales + f_sales) / total_sales)) * 100, 2) AS unknown
  FROM cte
  ORDER BY calendar_year
```
First, two subqueries calculate the total sales for each demographic in each year. This pulls from the `clean` table from the prior question. The two subqueries are joined to create one table that shows the total overall sales as well as the total sales for each demographic.

![image](https://github.com/user-attachments/assets/9db674d7-7397-498c-96f9-ad7ee5135109)

With that, each demographic's sales value is divided by total sales to find the percentage. Additionally, I calculated the percentage of sales where the demographic was unknown. Adding the two demographics and dividing it by total sales shows the total percentage of BOTH demographics, so subtracting that amount from 1 gives the remaining percentage.

![image](https://github.com/user-attachments/assets/4facfb5d-d07c-44dd-a34c-c82b31671e7f)

# 🦞 6. Clique Bait

This case study looks at website and visit data for an online seafood seller. Lots of information is provided, including individual steps that are taken during each individual user's visit to the website.

**[Case Study Website](https://8weeksqlchallenge.com/case-study-6/)**

**[All of my Solutions](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/tree/main/%236%20-%20Clique%20Bait)**

## 1. What are the top 3 products by purchases?

First, the tables must be understood to figure out how to properly handle this question. Below is the `events` table, which shows each step that each customer took when visiting the website in a given visit, ordered by sequence number.

![image](https://github.com/user-attachments/assets/2de76707-cd49-452b-964e-a23e2df6e28c)

Additionally, the `event_identifier` table shows what each event type means:

![image](https://github.com/user-attachments/assets/d8a8e2f4-980e-46c1-a54c-08e0799def67)

And `page_hierarchy` shows what each page ID means:

![image](https://github.com/user-attachments/assets/b7885bd5-2f84-4090-8e3e-b3fd34218e5a)

So with the example in the `events` table, the customer adds three products to their cart, and at the end, they check out. In some cases, however, the customer does not check out, so the purchase was not fully completed. These instances need to be filtered out, so we cannot just search for all products where `page_id` = 2.

```sql
WITH cte AS (SELECT
    a.visit_id,
    a.page_id AS purchases,
    b.page_id AS checkout
  FROM (
    SELECT
      visit_id,
      page_id
    FROM events
    WHERE event_type = 2) a
  JOIN (
    SELECT
      visit_id,
      page_id
    FROM events
    WHERE page_id = 13) b
  ON a.visit_id = b.visit_id)
  
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
First off, I used two subqueries. The first lists all items that were purchased, regardless of if it was a completed transaction. The page IDs get identified further at a later step.

![image](https://github.com/user-attachments/assets/8f303ce0-b48e-4263-8b6a-fe0f8785126a)

The other subquery pulls all rows where `page_id` equals 13, which is the confirmation screen that corresponds with a successful check-out.

![image](https://github.com/user-attachments/assets/cdefdba4-a128-4018-bbe0-b00a0721bc2e)

These two tables are combined using JOIN to list all purchases that took place when the customer also checked out on the visit. With that, I could then count the number of each products that were successfully purchased.

![image](https://github.com/user-attachments/assets/c65565d2-99a7-437e-893e-0e567921d1f5)

The final query counts the amount of each product that was purchased and identifies the name of each product based on the page ID.

![image](https://github.com/user-attachments/assets/8179c774-b01f-46ed-ad1c-3589f21202aa)

# 🧥 7. Balanced Tree Clothing Co.

This case study looks at data within an online clothing company. Clothes are broken down into categories and segments, and often times, multiple of the same type of clothing is bought in one transaction, so accurate financial info must be found by joining tables to identify categories and segments and using complex queries.

**[Case Study Website](https://8weeksqlchallenge.com/case-study-7/)**

**[All of my Solutions](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/tree/main/%237%20-%20Balanced%20Tree%20Clothing%20Co)**

## 1. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

This was one of the trickiest challenges overall. With 12 different products, many combinations are possible, and it becomes even trickier when some transactions purchase more than three products, creating multiple combinations in one transaction.

```sql
WITH products AS (
  SELECT
    txn_id,
    product_name
  FROM sales s
  JOIN product_details pd
    ON s.prod_id = pd.product_id)

SELECT
  p.product_name AS one,
  p2.product_name AS two,
  p3.product_name AS three,
  COUNT(*) AS amount
FROM products p
JOIN products p2
  ON p.txn_id = p2.txn_id
  AND p.product_name < p2.product_name
JOIN products p3
  ON p.txn_id = p3.txn_id
  AND p2.product_name < p3.product_name
GROUP BY p.product_name, p2.product_name, p3.product_name
ORDER BY amount DESC
```

First off, each transaction in the `sales` table shows a product ID rather than the product's name, so the CTE labels them by name. Here are all of the products bought by transaction ID 54f307.

![image](https://github.com/user-attachments/assets/a600d98b-7879-4bf0-a97a-5dc3d413a7f2)

The table is then joined three times in a way that creates unique combinations. They are joined by making sure the second product is GREATER than the first product. (meaning it is after the first product alphabetically) With the same transaction ID, there are six possible two-way combinations:

![image](https://github.com/user-attachments/assets/1ed41cbd-decc-4f58-aece-94bcd7abd6af)

Again, it is joined a third time using the same conditions. Product 1 has to be before product 2 alphabetically, and product 2 has to be before product 3. Here, there are four unique three-way combinations:

![image](https://github.com/user-attachments/assets/fcadcbee-1ba3-4b5d-859e-24c31a9090d3)

Finally, the query counts the amount of times each unique combination took place. It does not have to worry about the same combination being in a different order, since they were organized alphabetically.

![image](https://github.com/user-attachments/assets/95fc56b6-25a5-4de8-beb5-379437745263)

# 🖱️ 8. Fresh Segments

This case study looks at a digital marketing agency and tracks monthly data on how much interest their clients receive. 

**[Case Study Website](https://8weeksqlchallenge.com/case-study-8/)**

**[All of my Solutions](https://github.com/Adam-Chamberlain/8-Week-SQL-Challenge/tree/main/%238%20-%20Fresh%20Segments)**
