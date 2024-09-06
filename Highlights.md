This page showcases the problems within all eight case studies that were the most difficult and required the most complex SQL queries. I also explained my thought process in each query much more in-depth.

## 2. Pizza Runner

This case study focuses on a pizza delivery company. There are multiple questions that I am going to focus on from here. I also used DB Fiddle for this and the first case study, which was provided on that website, but I switched to MySQL for case studies 3-8.

### Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

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
