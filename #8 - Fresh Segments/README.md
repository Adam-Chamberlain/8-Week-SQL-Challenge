# [Case Study #8: Fresh Segments](https://8weeksqlchallenge.com/case-study-8/)

This case study looks at a digital marketing agency and tracks monthly data on how much interest their clients receive. There were some questions that were a bit confusing, so I have not completed every single question in this case study, but I will likely return to them at a later date.

## A. Data Exploration and Cleansing
### 1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month

```sql
ALTER TABLE interest_metrics
MODIFY COLUMN month_year VARCHAR(10);

UPDATE interest_metrics
SET month_year = CONCAT(RIGHT(month_year, 4), '-', LEFT(month_year, 2), '-01');

ALTER TABLE interest_metrics
MODIFY COLUMN month_year DATE;
```
The `month_year` column can only be seven characters long, so I first updated it to be up to ten so that I could add the day. I do so using CONCAT, and then I change the format of the column again to the date format.

### 2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?

```sql
SELECT
  month_year,
  COUNT(*) AS amount
FROM interest_metrics
GROUP BY month_year
ORDER BY month_year
```
![image](https://github.com/user-attachments/assets/a81f85d1-0429-4e21-9f49-d63087a79f04)

### 3. What do you think we should do with these null values in the fresh_segments.interest_metrics

Rows with null values exclude the date and interest ID. The remaining values don't really mean much without the interest ID showing. Out of curiosity, I checked to see how much the average of all the columns were with or without the null values.

With

![image](https://github.com/user-attachments/assets/7a4eb21b-9d1a-4f14-b9e0-fa05a6726244)

Without

![image](https://github.com/user-attachments/assets/46b53bfc-c8fe-4e97-b2e1-54dd515b672a)

None of the values vary enough to be meaningful, and the null values do not take up very many rows, it would probably be best to delete them all.

```sql
DELETE FROM interest_metrics
WHERE month_year IS NULL
```

### 4. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?

```sql
SELECT
  DISTINCT map.id,
  met.interest_id
FROM interest_map map
LEFT JOIN interest_metrics met
  ON map.id = met.interest_id
WHERE met.interest_id IS NULL
```
By joining the two tables with a LEFT JOIN, the amount of unique IDs that do not show up in `interest_metrics` can be counted. Alternatively, the second and third line can be changed to `COUNT(DISTINCT map.id)` to show the actual number of missing IDs.

![image](https://github.com/user-attachments/assets/251894b5-e469-48be-ab70-55debfc4a5fc)

```sql
SELECT
  DISTINCT map.id,
  met.interest_id
FROM interest_map map
RIGHT JOIN interest_metrics met
  ON map.id = met.interest_id
WHERE map.id IS NULL
```
Doing it the other way around, there are no missing IDs on the `interest_map` table.

### 5. Summarise the id values in the fresh_segments.interest_map by its total record count in this table

```sql
SELECT
  id,
  interest_name,
  COUNT(interest_name) AS amount
FROM interest_map map
JOIN interest_metrics met
  ON map.id = met.interest_id
GROUP BY id, interest_name
ORDER BY amount DESC, id
```
I'm not entirely sure what "this table" is referring to, so I am assuming it means `interest_metrics`. Each ID only appears once on the `interest_map` table. The table below shows the first few rows, ordered by the highest amount.

![image](https://github.com/user-attachments/assets/1c8cd823-04a1-48f2-9768-e0d9ac20a7a4)

### 6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.

INNER JOIN should be used so that there are not NULL values for the seven unused IDs.

### 7. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?

```sql
SELECT
  month_year,
  id,
  interest_name,
  created_at
FROM interest_map map
JOIN interest_metrics met
  ON map.id = met.interest_id
WHERE created_at > month_year
```
There are many instances where this happened, which is likely due to the fact that the `month_year` column always defaults to the first day of the month. If they were created before the `month_year` value, it is likely that the customer gained interest at that point instead.

![image](https://github.com/user-attachments/assets/144a8936-944a-453a-9dfa-98adb48de6f4)

To double-check that all of these instances are in the same month, I changed the last line of the query to this:

```sql
WHERE created_at > DATE(month_year + 30)
```
This adds 30 days to the `month_year` value to see if there were any instances where the interest ID was created a month or more later, which there was not.

## B. Interest Analysis
### 1. Which interests have been present in all month_year dates in our dataset?

There are 14 months included in the dataset, so any ID that is counted 14 times has been present in every month.

```sql
SELECT
  COUNT(*) AS amount
FROM (
  SELECT
    interest_id,
    COUNT(*) AS amount
  FROM interest_metrics
  GROUP BY interest_id
  ORDER BY amount DESC, interest_id) a
WHERE amount = 14
```
The subquery lists all interest IDs with how many times they appeared, and the main query counts how many of those IDs appeared 14 times.

![image](https://github.com/user-attachments/assets/eac738b3-14f2-4508-9673-e87d278a9f5a)

### 2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?

I have no idea what this question is asking, and after a bit of looking around, it looks like nobody else understands it either. I am skipping the remaining questions based off of this one for now.

### 3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?
### 4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.
### 5. After removing these interests - how many unique interests are there for each month?
## C. Segment Analysis
### 1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year

```sql
SELECT
  im.month_year,
  im.interest_id,
  im.composition
FROM interest_metrics im
JOIN (
  SELECT
    interest_id,
    COUNT(*) AS amount
  FROM interest_metrics
  GROUP BY interest_id) a
  ON im.interest_id = a.interest_id
WHERE a.amount >= 6
ORDER BY composition DESC
LIMIT 10
```
I created a subquery that counts the amount of times each interest ID appears, and then I joined it with the original `interest_metrics` table. This allowed me to filter out interest IDs that appear less than 6 times. With that, I sorted it to show the top composition values.

![image](https://github.com/user-attachments/assets/c9a074cb-fd7a-4296-8606-60778b1cb8e5)

Removing DESC from the ORDER BY clause shows the lowest values:

![image](https://github.com/user-attachments/assets/2de14dc3-2358-4ade-9999-2573773c9dc6)

### 2. Which 5 interests had the lowest average ranking value?

```sql
SELECT
  interest_id,
  AVG(ranking) AS average
FROM interest_metrics
GROUP BY interest_id
ORDER BY average DESC
LIMIT 5
```
The best, or highest, rankings are the smallest numbers, so the five with the highest average are shown.

![image](https://github.com/user-attachments/assets/be51a10e-44a9-4725-a5cd-c49a63542841)

### 3. Which 5 interests had the largest standard deviation in their percentile_ranking value?

```sql
SELECT
  interest_id,
  ROUND(STDDEV(percentile_ranking), 2) AS stdev
FROM interest_metrics
GROUP BY interest_id
ORDER BY stdev DESC
LIMIT 5
```
![image](https://github.com/user-attachments/assets/e91b3dd4-6214-4135-bd95-094aa34f7755)

### 4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?

```sql
SELECT
  a.interest_id,
  a.stdev,
  a.max,
  im1.month_year AS max_month,
  a.min,
  im2.month_year AS min_month
FROM
  (SELECT
    interest_id,
    ROUND(STDDEV(percentile_ranking), 2) AS stdev,
    MAX(percentile_ranking) AS max,
    MIN(percentile_ranking) AS min
  FROM interest_metrics
  GROUP BY interest_id) a
JOIN interest_metrics im1
  ON a.interest_id = im1.interest_id AND a.max = im1.percentile_ranking
JOIN interest_metrics im2
  ON a.interest_id = im2.interest_id AND a.min = im2.percentile_ranking
ORDER BY stdev DESC
LIMIT 5
```
I basically took the prior question's query and expanded on it for this. I found the min and max values in a subquery and matched them with the respective `month_year` date as long as the interest ID also matched. To do this, I joined the subquery with two the original `interest_metrics` table twice, once to find the max value's month and year, and once for the min value.

![image](https://github.com/user-attachments/assets/5321c4ea-3187-4b59-a073-e79a2363d0ff)

### 5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?
## D. Index Analysis

The index_value is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

### 1. What is the top 10 interests by the average composition for each month?

```sql
SELECT
  interest_id,
  month_year,
  composition,
  index_value,
  ROUND(composition / index_value, 2) AS avg_comp
FROM interest_metrics
ORDER BY avg_comp DESC
LIMIT 10
```
![image](https://github.com/user-attachments/assets/c2aa1135-b902-4972-b77f-0193d32116a6)

### 2. For all of these top 10 interests - which interest appears the most often?

The ID `21057` appears six times and has the top six average composition values. `6324` also appears twice.

### 3. What is the average of the average composition for the top 10 interests for each month?

```sql
SELECT
month_year,
ROUND(AVG(avg_comp), 2) AS average
FROM
(SELECT
RANK() OVER(PARTITION BY month_year ORDER BY ROUND(composition / index_value, 2) DESC) AS ranking,
interest_id,
month_year,
composition,
index_value,
ROUND(composition / index_value, 2) AS avg_comp
FROM interest_metrics
ORDER BY month_year, ranking) a
WHERE ranking BETWEEN 1 AND 10
GROUP BY month_year
```
The subquery creates the `avg_comp` column and also ranks each average composition value by each month. This allows me to only pull the top 10 from each month in the outer query.

![image](https://github.com/user-attachments/assets/a0432883-57f0-4825-a09a-347f68ef7bb6)

### 4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.

Again, this question is a little confusing, and I am not entirely sure how to solve it properly. I will return to this at a later date.

### 5. Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?
