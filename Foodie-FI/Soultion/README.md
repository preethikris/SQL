# 🥗 Foodie-Fi — SQL Case Study
This folder contains my SQL solutions for the Foodie-Fi case study from the 8 Week SQL Challenge.
## 📌 Overview
This case study analyzes customer subscription behavior, plan transitions, churn patterns, and revenue insights for Foodie-Fi’s streaming platform.
---
## 🗂️ Tables Used
- plans
- subscriptions
---

## Entity Relationship Diagram



***
---
#### ❓ Business Questions
## A. Customer Journey
# Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.
# Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!
````sql
SELECT
s.customer_id,
p.plan_name,
s.start_date
FROM subscriptions s
JOIN plans p ON s.plan_id = p.plan_id
WHERE s.customer_id <= 8
ORDER BY s.customer_id, s.start_date;
````
Customer 1
- Started with trial
- Upgraded to basic monthly
- Later upgraded to pro annual

Customer 2

- Started with trial
- Converted to pro monthly
- Eventually churned

Customer 3

- Began with trial
- Moved to basic monthly
- Later upgraded to pro monthly

 Customer 4

- Started with trial
- Immediately churned after trial

Customer 5

- Began with trial
- Upgraded to basic monthly
- Later churned

Customer 6

- Started with trial
- Upgraded to pro monthly
- Later upgraded to pro annual

Customer 7

- Began with trial
- Converted to basic monthly
- Stayed on that plan

Customer 8

- Started with trial
- Upgraded to pro monthly
- Later churned


# How many customers has Foodie-Fi ever had?**

````sql
  Select count(distinct customer_id)as total_customer 
  from subscriptions ; 
````
#### Steps:
- Use the subscriptions table because it contains all customers.
- Count distinct customer_id since one customer can have multiple records.
- Apply COUNT(DISTINCT customer_id).

#### Answer:
1000 customers
***
# What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value **
````sql
SELECT DATE_FORMAT(start_date,'%Y-%m-01') AS start_of_month,
COUNT(*) AS trial_count
FROM subscriptions s
JOIN plans p ON s.plan_id=p.plan_id
WHERE p.plan_name='trial'
GROUP BY start_of_month
ORDER BY start_of_month;
````
#### Steps:
- Join plans + subscriptions
- Filter trial
- Convert date → month start
- Count

#### Answer:
| Start_of_month| trial_count |
| ----- | ----------- |
| 2020-01-01 | 88          |
| 2020-02-01 | 68          |
| 2020-03-01 | 94          |
| 2020-04-01 | 81          |
| 2020-05-01 | 88          |
| 2020-06-01 | 79          |
| 2020-07-01 | 89          |
| 2020-08-01 | 88          |
| 2020-09-01 | 87          |
| 2020-10-01 | 79          |
| 2020-11-01 | 75          |
| 2020-12-01 | 86          |

***

# What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name **
````sql
Select plan_name, count(*) 
from plans p 
join subscriptions s 
on p.plan_id=s.plan_id
where year(start_date)>2020
group by plan_name;
````
#### Steps:
- Filter start_date > 2020
- Count events per plan

#### Answer:
| plan_name     | event_count |
| ------------- | ----------- |
| basic monthly | 8           |
| pro monthly   | 60          |
| pro annual    | 63          |
| churn         | 71          |

***
# What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**
````sql
select count(distinct case when plan_name='churn' then customer_id end) as customer_count,
count(distinct case when plan_name='churn' then customer_id end )*100 /count( distinct customer_id)as percentage_customer
from plans p join subscriptions s using(plan_id);
````
#### Steps:
- Count churn customers
- Divide by total customers

#### Answer:
| customer_count | percentage |
| -------------- | ---------- |
| 307            | 30.7%      |
***
# How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number? **
````sql
WITH t AS (
  SELECT 
    s.customer_id,
    p.plan_name,
    LEAD(p.plan_name) OVER (
      PARTITION BY s.customer_id 
      ORDER BY s.start_date
    ) AS next_plan
  FROM subscriptions s
  JOIN plans p 
    ON s.plan_id = p.plan_id
)
SELECT
  COUNT(DISTINCT customer_id) AS churn_count,
  ROUND(
    COUNT(DISTINCT customer_id) * 100.0 /
    (SELECT COUNT(DISTINCT customer_id) FROM subscriptions),
    0
  ) AS churn_percentage
FROM t
WHERE plan_name = 'trial'
  AND next_plan = 'churn';
````
#### Steps:
- Use LEAD to get next plan
- Filter trial → churn

#### Answer:
| churn_count | churn_percentage |
| ----------- | ---------------- |
| 92          | 9%               |

***
# What is the number and percentage of customer plans after their initial free trial? **
````sql
With Cte_lead as(select customer_id,plan_name,
lead(plan_name) over(partition by customer_id order by start_date) as next_plan 
from plans p 
join subscriptions s 
using(plan_id))
Select next_plan,
count(customer_id)as number_customer,
Count(DISTINCT customer_id) * 100.0/  (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) 
from cte_lead where plan_name='Trial' and next_plan is not null
group by next_plan order by next_plan;
````
#### Steps:
- LEAD next plan
- Filter trial
- Count

#### Answer:
| next_plan     | number_customer | percentage |
| ------------- | --------------- | ---------- |
| basic monthly | 546             | 54.6%      |
| pro monthly   | 325             | 32.5%      |
| churn         | 92              | 9.2%       |
| pro annual    | 37              | 3.7%       |

***
# What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**
````sql
WITH cte_active_plan AS (
    SELECT
        s.customer_id,
        p.plan_name,
        ROW_NUMBER() OVER (
            PARTITION BY s.customer_id
            ORDER BY s.start_date DESC
        ) AS rn
    FROM subscriptions s
    JOIN plans p
      ON s.plan_id = p.plan_id
    WHERE s.start_date <= '2020-12-31'
)
SELECT
    plan_name,
    COUNT(customer_id) AS customer_count,
    ROUND(
        COUNT(customer_id) * 100.0 /
        (SELECT COUNT(DISTINCT customer_id) FROM subscriptions),
        1
    ) AS percentage
FROM cte_active_plan
WHERE rn = 1
GROUP BY plan_name
ORDER BY plan_name;
````
#### Steps:
- Get latest plan before date (ROW_NUMBER)
- Count

#### Answer:
| plan_name     | customer_count | percentage |
| ------------- | -------------- | ---------- |
| basic monthly | 224            | 22.4       |
| churn         | 236            | 23.6       |
| pro annual    | 195            | 19.5       |
| pro monthly   | 326            | 32.6       |
| trial         | 19             | 1.9        |

***
# How many customers have upgraded to an annual plan in 2020?**
````sql
SELECT COUNT(DISTINCT s.customer_id) AS customers
FROM plans p
JOIN subscriptions s USING(plan_id)
WHERE YEAR(start_date)=2020
AND plan_name='pro annual';
````
#### Steps:
- Filter pro annual + year 2020
- Count customers

#### Answer:
195 customers
***
#How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**
````sql
WITH t AS (
  SELECT
    s.customer_id,
    MIN(s.start_date) AS join_date,
    MIN(CASE 
          WHEN p.plan_name = 'pro annual' 
          THEN s.start_date 
        END) AS annual_date
  FROM subscriptions s
  JOIN plans p 
    ON s.plan_id = p.plan_id
  GROUP BY s.customer_id
)
SELECT 
  ROUND(AVG(DATEDIFF(annual_date, join_date)), 0) AS avg_days_to_annual
FROM t
WHERE annual_date IS NOT NULL;
````
#### Steps:
- Get join date
- Get annual date
- Difference
- Average

#### Answer:
104 days
***
# Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc) **
````sql
WITH t AS (
  SELECT
    s.customer_id,
    DATEDIFF(
      MIN(CASE WHEN p.plan_name = 'pro annual' THEN s.start_date END),
      MIN(s.start_date)
    ) AS days_to_upgrade
  FROM subscriptions s
  JOIN plans p 
    ON s.plan_id = p.plan_id
  GROUP BY s.customer_id
)
SELECT
  CASE
    WHEN days_to_upgrade BETWEEN 0 AND 30 THEN '0-30 days'
    WHEN days_to_upgrade BETWEEN 31 AND 60 THEN '31-60 days'
    WHEN days_to_upgrade BETWEEN 61 AND 90 THEN '61-90 days'
    WHEN days_to_upgrade BETWEEN 91 AND 120 THEN '91-120 days'
    WHEN days_to_upgrade BETWEEN 121 AND 150 THEN '121-150 days'
    WHEN days_to_upgrade BETWEEN 151 AND 180 THEN '151-180 days'
    ELSE '180+ days'
  END AS period,
  COUNT(*) AS customer_count
FROM t
WHERE days_to_upgrade IS NOT NULL
GROUP BY period
ORDER BY period;
````
#### Answer:
| period  | customer_count |
| ------- | -------------- |
| 0-30    | 49             |
| 31-60   | 24             |
| 61-90   | 34             |
| 91-120  | 35             |
| 121-150 | 42             |
| 151-180 | 36             |
| 180+    | 82             |
# How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**
````sql
with cte_down as(select customer_id, plan_name, lag(plan_name) over(partition by customer_id order by start_date)as previous_plan  from plans p join subscriptions s using (plan_id) 
where year(start_date)=2020)
select count( distinct customer_id) from cte_down where plan_name='basic monthly' and previous_plan='pro monthly';
````
#### Steps:
- Use LAG previous plan
- Filter downgrade
- Count

#### Answer:
0 customers

## C. Challenge Payment Question
````sql
WITH cte AS (
SELECT
s.customer_id,
p.plan_name,
p.price,
s.start_date,
LEAD(s.start_date) OVER (
PARTITION BY s.customer_id
ORDER BY s.start_date
) AS next_date
FROM subscriptions s
JOIN plans p ON s.plan_id=p.plan_id
),

payments AS (
SELECT
customer_id,
plan_name,
price,
start_date AS payment_date,
next_date
FROM cte
WHERE plan_name NOT IN ('trial','churn')
)

SELECT *
FROM payments
WHERE YEAR(payment_date)=2020;
````
#### Answer:

| customer_id | plan_id | plan_name     | payment_date | amount | payment_order |
| ----------- | ------- | ------------- | ------------ | ------ | ------------- |
| 1           | 1       | basic monthly | 2020-08-08   | 9.90   | 1             |
| 1           | 1       | basic monthly | 2020-09-08   | 9.90   | 2             |
| 1           | 1       | basic monthly | 2020-10-08   | 9.90   | 3             |
| 1           | 1       | basic monthly | 2020-11-08   | 9.90   | 4             |
| 1           | 1       | basic monthly | 2020-12-08   | 9.90   | 5             |
| 2           | 3       | pro annual    | 2020-09-27   | 199.00 | 1             |
| 13          | 1       | basic monthly | 2020-12-22   | 9.90   | 1             |
| 15          | 2       | pro monthly   | 2020-03-24   | 19.90  | 1             |
| 15          | 2       | pro monthly   | 2020-04-24   | 19.90  | 2             |
| 16          | 1       | basic monthly | 2020-06-07   | 9.90   | 1             |
| 16          | 1       | basic monthly | 2020-07-07   | 9.90   | 2             |
| 16          | 1       | basic monthly | 2020-08-07   | 9.90   | 3             |
| 16          | 1       | basic monthly | 2020-09-07   | 9.90   | 4             |
| 16          | 1       | basic monthly | 2020-10-07   | 9.90   | 5             |
| 16          | 3       | pro annual    | 2020-10-21   | 189.10 | 6             |
| 18          | 2       | pro monthly   | 2020-07-13   | 19.90  | 1             |
| 18          | 2       | pro monthly   | 2020-08-13   | 19.90  | 2             |
| 18          | 2       | pro monthly   | 2020-09-13   | 19.90  | 3             |
| 18          | 2       | pro monthly   | 2020-10-13   | 19.90  | 4             |
| 18          | 2       | pro monthly   | 2020-11-13   | 19.90  | 5             |
| 18          | 2       | pro monthly   | 2020-12-13   | 19.90  | 6             |
| 19          | 2       | pro monthly   | 2020-06-29   | 19.90  | 1             |
| 19          | 2       | pro monthly   | 2020-07-29   | 19.90  | 2             |
| 19          | 3       | pro annual    | 2
