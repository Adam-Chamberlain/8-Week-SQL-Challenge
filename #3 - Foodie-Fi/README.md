# [Case Study #3: Foodie-Fi](https://8weeksqlchallenge.com/case-study-3/)

In this case study, a streaming service based around cooking shows is wanting to use collected data to answer important business questions to ensure future growth. It uses two tables: `plans`, which lists the type of purchasable plans and their price, and `subscriptions`, which lists each customer, when they bought or upgraded a plan, and on what date.

Since there were many more rows of data in this case study, I decided to use MySQL instead of the DB Fiddle database that was provided. 

### Entity Relationship Diagram
![129744449-37b3229b-80b2-4cce-b8e0-707d7f48dcec](https://github.com/user-attachments/assets/4403d92f-7759-4931-846a-ccaf9eb63d85)

## A. Customer Journey
### Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey. Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

```
SELECT
  s.customer_id,
  s.start_date,
  p.plan_id,
  p.plan_name,
  p.price
FROM subscriptions s
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
    (SELECT
      COUNT(DISTINCT customer_id)
    FROM subscriptions
    WHERE plan_id = 4) AS churn,
ROUND(
    (SELECT
      COUNT(DISTINCT customer_id)
    FROM subscriptions
    WHERE plan_id = 4)
  / COUNT(DISTINCT customer_id) * 100, 1) AS percent
FROM subscriptions
```
This uses subqueries to find the exact number of customers that have churned, and it then divides that number by the total number of customers to get the percentage.

|churned|percent|
|---|---|
|307|0.3|

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```
WITH ranked AS (SELECT 
  customer_id, 
  plan_id, 
  ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS number
FROM subscriptions)
    
SELECT
  COUNT(CASE WHEN number = 2 AND plan_id = 4 THEN 1 ELSE NULL END) AS churn_count,
  ROUND(COUNT(CASE WHEN number = 2 AND plan_id = 4 THEN 1 ELSE NULL END) / COUNT(DISTINCT customer_id) * 100, 0) AS percentage
FROM ranked
```
This uses a CTE to first put numbers next to each customer's plan purchase. The PARTITION BY makes it start counting again every time a new customer shows up; the first time they appear, it counts 1, and the second time, it counts 2, etc. Every single customer starts with a free trial, which is 1, so if they churn immediately after, the row will be 2.

With this, it counts the rows where the ranking is equal to 2 and the plan ID is equal to 4, which is the ID for churn.

|churned|percent|
|---|---|
|92|9|

### 6. What is the number and percentage of customer plans after their initial free trial?

```
WITH ranked AS (SELECT 
    customer_id, 
    s.plan_id,
    plan_name,
	  ROW_NUMBER() OVER (
      PARTITION BY customer_id 
      ORDER BY start_date) AS number
  FROM subscriptions s
  JOIN plans p
  ON s.plan_id = p.plan_id)
    
SELECT
  plan_id,
  plan_name,
  COUNT(number) AS count,
  ROUND(COUNT(number) / (SELECT COUNT(DISTINCT customer_id) FROM ranked) * 100, 0) AS percentage
FROM ranked
WHERE number = 2
GROUP BY plan_name, plan_id
ORDER BY plan_id
```
This uses a similar CTE as the last question to pull relevant info and add rankings to each customer. Again, since everyone starts with a free trial, the values with the number 2 are key here. 

|plan_id|plan_name|churned|percent|
|---|---|---|---|
|1|basic monthly|546|55|
|2|pro monthly|546|33|
|3|pro annual|546|4|
|4|churn|92|9|

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```
WITH ranked AS (SELECT 
    customer_id, 
    s.plan_id,
    plan_name,
	  ROW_NUMBER() OVER (
      PARTITION BY customer_id 
      ORDER BY start_date DESC) AS number
  FROM subscriptions s
  JOIN plans p
  ON s.plan_id = p.plan_id
  WHERE YEAR(start_date) < 2021)
    
SELECT
  plan_id,
  plan_name,
  COUNT(number) AS count,
  ROUND(COUNT(number) / (SELECT COUNT(DISTINCT customer_id) FROM ranked) * 100, 0) AS percentage
FROM ranked
WHERE number = 1
GROUP BY plan_name, plan_id
ORDER BY plan_id
```
This only needed a few modifications from the previous query. First, the ranking was ordered in descending order instead to mark the most recent subscription per customer as 1. I also added a filter to only include subscriptions from before 2021.

|plan_id|plan_name|churned|percent|
|---|---|---|---|
|0|trial|19|2|
|1|basic monthly|224|22|
|2|pro monthly|326|33|
|3|pro annual|195|20|
|4|churn|236|24|

### 8. How many customers have upgraded to an annual plan in 2020?

```
SELECT
  COUNT(customer_id) AS count
FROM subscriptions
WHERE plan_id = 3 AND YEAR(start_date) = 2020
```

|count|
|-----|
|195 |

### 9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?

```
WITH annual AS (SELECT
  customer_id,
  plan_id,
  start_date
FROM subscriptions
WHERE plan_id = 3),

trial AS (SELECT
  customer_id,
  plan_id,
  start_date
FROM subscriptions
WHERE plan_id = 0)

SELECT
  ROUND(AVG(DATEDIFF(a.start_date, t.start_date)), 0) AS average
FROM annual a
JOIN trial t
ON a.customer_id = t.customer_id
```
This uses two CTEs: one to only pull annual subscriptions, and one to pull trial subscriptions. They are then JOINed, the difference between the two signup dates are found, and the average is found of those differences.

|average|
|-----|
|105 |

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

I'm honestly not too sure what this question is exactly asking, but I will assume that it is asking for how many customers fall into each category.

```
WITH annual AS (SELECT
  customer_id,
  plan_id,
  start_date
FROM subscriptions
WHERE plan_id = 3),

trial AS (SELECT
  customer_id,
  plan_id,
  start_date
FROM subscriptions
WHERE plan_id = 0)

SELECT
  CASE WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 0 AND 30 THEN 1
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 31 AND 60 THEN 2
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 61 AND 90 THEN 3
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 91 AND 120 THEN 4
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 121 AND 150 THEN 5
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 151 AND 180 THEN 6
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 181 AND 210 THEN 7
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 211 AND 240 THEN 8
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 241 AND 270 THEN 9
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 271 AND 300 THEN 10
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 301 AND 330 THEN 11
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 331 AND 360 THEN 12
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 361 AND 390 THEN 13
  ELSE NULL END AS num,
  CASE WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 0 AND 30 THEN '0-30 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 31 AND 60 THEN '31-60 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 61 AND 90 THEN '61-90 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 91 AND 120 THEN '91-120 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 121 AND 150 THEN '121-150 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 151 AND 180 THEN '151-180 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 181 AND 210 THEN '181-210 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 211 AND 240 THEN '211-240 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 241 AND 270 THEN '241-270 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 271 AND 300 THEN '271-300 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 301 AND 330 THEN '301-330 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 331 AND 360 THEN '331-360 days'
    WHEN DATEDIFF(a.start_date, t.start_date) BETWEEN 361 AND 390 THEN '361-390 days'
 ELSE NULL END AS amount,
  COUNT(DATEDIFF(a.start_date, t.start_date)) AS upgrade_time
FROM annual a
JOIN trial t
  ON a.customer_id = t.customer_id
GROUP BY amount, num
ORDER BY num
```
Using the same two CTEs, I used CASE statements to put different values depending on how long the difference was between each customer's first trial and their purchase of an annual plan. It is duplicated for sorting purposes; although it's a bit messy, it seemed like the easiest way to effectively sort the data.

|num|amount      |upgrade_time|
|---|------------|------------|
|1  |0-30 days   |49          |
|2  |31-60 days  |24          |
|3  |61-90 days  |34          |
|4  |91-120 days |35          |
|5  |121-150 days|42          |
|6  |151-180 days|36          |
|7  |181-210 days|26          |
|8  |211-240 days|4           |
|9  |241-270 days|5           |
|10 |271-300 days|1           |
|11 |301-330 days|1           |
|12 |331-360 days|1           |

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

```
WITH pro AS (SELECT
customer_id,
plan_id,
start_date
FROM subscriptions
WHERE plan_id = 2 AND YEAR(start_date) = 2020),

basic AS (SELECT
customer_id,
plan_id,
start_date
FROM subscriptions
WHERE plan_id = 1 AND YEAR(start_date) = 2020)

SELECT
COUNT(DATEDIFF(p.start_date, b.start_date)) AS amount
FROM pro p
JOIN basic b
ON p.customer_id = b.customer_id
WHERE b.start_date > p.start_date
```
Using similar CTE tables to instead pull instances with pro monthly and basic montly plan purchases, I had it count the amount of instances where customers specifically downgraded from pro monthly to basic monthly, which was actually none.

## C. Challenge Payment Question
### The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:
- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

I will return to this question at a later date.

## D. Outside The Box Questions
### 1. How would you calculate the rate of growth for Foodie-Fi?

There are a few ways this can be done. The table in question 2 shows how many new customers use the product for the first time throughout the year. But that doesn't paint the whole picture; customers are only new once, and many subscribe long-term, so it can be better to look at how much revenue is brought in on a monthly or yearly basis as well as how many active customers there are each month.

### 2. What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?

Like mentioned before, tracking monthly and annual revenue is key to see if the company is actually growing or not. It is also important to track how many customers they have each month as well as how many new customers join and how many churn. This can give a good idea of whether they are gaining more customers than they are losing, or vise versa.

### 3. What are some key customer journeys or experiences that you would analyse further to improve customer retention?

It would probably be best to look at two different points: what customers do after the free trial ends, and what causes customers to churn. The reasonings that people purchase subscriptions after the free trial can likely be found with data, but not the data that is supplied. Demographics, location and wealth likely drive a consumer's decision as well as what shows they watched during the trial. The same data points could be used to see why people churn, but it is also worth looking at if new content is keeping people on the platform.

### 4. If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?

- What caused you to make the decision to cancel your subscription?

This would give the company an idea of why people cancelled, whether it be money related, issues with the service, lack of new content, or something else.
- Did you have any issues using the service?

It's important for companies to keep their consumers happy and satisfied so that they continue using the product. This can give the company an idea of what they need to improve on.
- What other streaming services do you use?

This can give the company an idea of their main competition, especially if they are taking consumers away from them. If one is identified, they can do in-depth research into the company to see how they can more effectively compete.

### 5. What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?

It'd be important to see why customers decide to churn, especially ones that are long-term customers and not ones that churn immediately after the free trial ends. Lots of research could be conducted to see what shows these users watched as well as how frequently they used the service. As mentioned before, demographics, wealth, and location likely play a role in what causes customers to subscribe, so there could be more targeted marketing towards the demographics that are most likely to use the product.
