# [Case Study #8: Fresh Segments](https://8weeksqlchallenge.com/case-study-8/)


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
### 3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?
### 4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.
### 5. After removing these interests - how many unique interests are there for each month?
## C. Segment Analysis
### 1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year
### 2. Which 5 interests had the lowest average ranking value?
### 3. Which 5 interests had the largest standard deviation in their percentile_ranking value?
### 4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?
### 5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?
## D. Index Analysis
The index_value is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

What is the top 10 interests by the average composition for each month?
For all of these top 10 interests - which interest appears the most often?
What is the average of the average composition for the top 10 interests for each month?
What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.
Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?
