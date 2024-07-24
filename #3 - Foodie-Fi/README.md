https://8weeksqlchallenge.com/case-study-3/

In this case study, a streaming service based around cooking shows is wanting to use collected data to answer important business questions to ensure future growth.

Since there were many more rows of data in this case study, I decided to use MySQL instead of the DB Fiddle database that was provided. 

## A. Customer Journey
### Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey. Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

```
SELECT
s.customer_id,
s.start_date,
p.plan_id,
p.plan_name,
p.price
from subscriptions s
JOIN plans p
ON s.plan_id = p.plan_id
WHERE s.customer_id IN(1, 2, 11, 13, 15, 16, 18, 19)
```
The samples shown on the case study website are customers with the IDs 1, 2, 11, 13, 15, 16, 18, and 19. I joined the two tables to identify plan names and prices and then filtered out data to only show these 8 customers.

- **1:** Free Trial (7 days) -> Basic Monthly Plan
- **2:** Free Trial (7 days) -> Pro Annual Plan
- **11:** Free Trial (7 days) -> Churn
- **13:** Free Trial (7 days) -> Basic Monthly Plan (3 Months) -> Pro Monthly Plan
- **15:** Free Trial (7 days) -> Pro Monthly Plan (1 Month) -> Churn
- **16:** Free Trial (7 days) -> Basic Monthly Plan (4 Months) -> Pro Annual Plan
- **18:** Free Trial (7 days) -> Pro Monthly Plan
- **19:** Free Trial (7 days) -> Pro Monthly Plan (2 Months) -> Pro Annual Plan

|customer_id|start_date|plan_id|plan_name    |price |
|-----------|----------|-------|-------------|------|
|1          |2020-08-01|0      |trial        |0.00  |
|1          |2020-08-08|1      |basic monthly|9.90  |
|2          |2020-09-20|0      |trial        |0.00  |
|2          |2020-09-27|3      |pro annual   |199.00|
|11         |2020-11-19|0      |trial        |0.00  |
|11         |2020-11-26|4      |churn        |NULL  |
|13         |2020-12-15|0      |trial        |0.00  |
|13         |2020-12-22|1      |basic monthly|9.90  |
|13         |2021-03-29|2      |pro monthly  |19.90 |
|15         |2020-03-17|0      |trial        |0.00  |
|15         |2020-03-24|2      |pro monthly  |19.90 |
|15         |2020-04-29|4      |churn        |NULL  |
|16         |2020-05-31|0      |trial        |0.00  |
|16         |2020-06-07|1      |basic monthly|9.90  |
|16         |2020-10-21|3      |pro annual   |199.00|
|18         |2020-07-06|0      |trial        |0.00  |
|18         |2020-07-13|2      |pro monthly  |19.90 |
|19         |2020-06-22|0      |trial        |0.00  |
|19         |2020-06-29|2      |pro monthly  |19.90 |
|19         |2020-08-29|3      |pro annual   |199.00|

## B. Data Analysis Questions
### 1. How many customers has Foodie-Fi ever had?
```
SELECT
COUNT(DISTINCT customer_id) AS count
FROM subscriptions
```
Pretty straightforward; for this, all that is needed is to count the number of distinct customer IDs.

|count|
|-----|
|1000 |

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```
SELECT
MONTH(start_date) AS month,
MONTHNAME(start_date) AS month_name,
COUNT(CASE WHEN plan_name = 'trial' THEN 1 ELSE NULL END) AS trial
FROM subscriptions s
JOIN plans p
ON s.plan_id = p.plan_id
GROUP BY month, month_name
ORDER BY month
```
Basically, I used a CASE statement to count instances where there was a free trial, and then I had the total grouped by each month.

|month|month_name|trial|
|---|---|---|
|1|January|88|
|2|February|68|
|3|March|94|
|4|April|81|
|5|May|88|
|6|June|79|
|7|July|89|
|8|August|88|
|9|September|87|
|10|October|79|
|11|November|75|
|12|December|84|

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```
SELECT
s.plan_id,
plan_name,
COUNT(customer_id) AS amount
FROM subscriptions s
JOIN plans p
ON s.plan_id = p.plan_id
WHERE YEAR(start_date) = 2021
GROUP BY plan_name, s.plan_id
ORDER BY s.plan_id
```
This query counts the amount of plans that are signed up for in the year 2021, sorted by plan name.

|plan_id|plan_name|amount
|---|---|---|
1|basic monthly|8|
2|pro monthly|60|
3|pro annual|63|
4|churn|71|

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```
SELECT
ROUND((SELECT
COUNT(DISTINCT customer_id) AS churn
FROM subscriptions
WHERE plan_id = 4)
/ COUNT(DISTINCT customer_id),1) AS percent_churned
FROM subscriptions
```
This uses a subquery to find the exact number of customers that have churned, and it then divides that number by the total number of customers.

|percent_churned|
|---|
|0.3|

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
### 6. What is the number and percentage of customer plans after their initial free trial?
### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
### 8. How many customers have upgraded to an annual plan in 2020?
### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

## C. Challenge Payment Question
### The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:
- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

## D. Outside The Box Questions
### 1. How would you calculate the rate of growth for Foodie-Fi?
### 2. What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?
### 3. What are some key customer journeys or experiences that you would analyse further to improve customer retention?
### 4. If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?
### 5. What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?
