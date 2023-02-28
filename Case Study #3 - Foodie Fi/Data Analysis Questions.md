**Question 1**

How many customers has Foodie-Fi ever had?

```sql
SELECT COUNT(DISTINCT customer_id) as num_of_customers
  FROM subscriptions;
```

I counted the distinct number of customers that had Foodie-Fi had had.

Results:

| num_of_customers |
| ---------------- |
| 1000             |

* Foodie-Fi has had 1000 customers

-------------------------------


**Question 2**

What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```sql
  SELECT to_char(start_date, 'month')as month,count(plan_id) as num_of_trial
    FROM subscriptions
   WHERE plan_id = 0
GROUP BY date_trunc('month',start_date),to_char(start_date, 'month')
ORDER BY date_trunc('month',start_date);
```

I counted the number of trials and grouped by each month then I used `to_char` to extract the name of each month.

Results :

| month   | num_of_trial |
| --------- | ------------ |
| january   | 88           |
| february  | 68           |
| march     | 94           |
| april     | 81           |
| may       | 88           |
| june      | 79           |
| july      | 89           |
| august    | 88           |
| september | 87           |
| october   | 79           |
| november  | 75           |
| december  | 84           |

There were;
* 88 trials in January
* 68 trials in February
* 94 trials in March
* 81 trials in April
* 88 trials in May
* 79 trials in June
* 89 trials in July 
* 88 trials in August
* 87 trials in September
* 79 trials in October
* 75 trials in November
* 84 trials in December

---------------------------------

**Question 3**

What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```sql
  SELECT p.plan_name, COUNT(s.plan_id)
    FROM subscriptions s
    JOIN plans p
      ON s.plan_id = p.plan_id
   WHERE start_date > '2020-12-31'
GROUP BY 1;
```
I counted the number of plans after 31st December 2020 and grouped by the plan name

Results:

| plan_name     | count |
| ------------- | ----- |
| pro annual    | 63    |
| churn         | 71    |
| pro monthly   | 60    |
| basic monthly | 8     |

After 2020 there were;

* 63 pro annual plans 
* 71 churns
* 60 pro monthly plans
* 8 basic monthly plans

----------------------------------

**Question 4**

What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```sql
    WITH churn_cte as (
             SELECT COUNT(DISTINCT customer_id) as num_of_churn
               FROM subscriptions
              WHERE plan_id = 4)

  SELECT num_of_churn, 
         num_of_churn/COUNT(distinct customer_id) :: double precision *100 as churn_percent
    FROM churn_cte,subscriptions
GROUP BY 1;
```
I used a CTE to get the number of customer's who churned and divided it by the number of total number of customers. 

Results:
| num_of_churn | churn_percent |
| ------------ | ------------- |
| 307          | 30.7          |

* Approximately 30.7 percent of the customers churned.

---------------------------------------------------

**Question 5**

How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```sql
WITH 
--ranks the plans for each customer
plan_ranking AS (
  SELECT customer_id,
         plan_id,
         DENSE_RANK() OVER (
           PARTITION BY customer_id ORDER BY plan_id) as plan_ranks
    FROM subscriptions)

SELECT COUNT(pr.customer_id) as churn_after_trial_count, 
       ROUND(COUNT(pr.customer_id)/ (
          SELECT COUNT(DISTINCT customer_id) --total number of customers
            FROM subscriptions)::double precision
                 ) * 100) as churn_after_trial_percentage
  FROM plan_ranking pr
 WHERE pr.plan_id = 4 --4 is id for churn plan
   AND pr.plan_ranks = 2; --rank of 2 means plan after trial
```

I used a CTE to rank the plan ids for each customer according to the plan ID. The rationale is that for customers who churned after trial, the first rank would be the trial ID and the second rank would be the churn ID which is 4. Therefore i queried the CTE for the count of customers whose second rank was 2 and the plan id was 4.

Results:
| churn_after_trial_count | churn_after_trial_percentage |
| ----------------------- | ------------------- |
| 92                      | 9                   |

* 92 customers churned straight after trial making 9 percent of the total customers.

----------------------

**Question 6**

What is the number and percentage of customer plans after their initial free trial?

```sql
WITH plan_after_trial AS (
              SELECT customer_id,
                     plan_id,
                     plan_name,
                     start_date,
                     dense_rank() over (partition by customer_id order by plan_id) as rank
                FROM subscriptions s
                JOIN plans p
                  ON s.plan_id = p.plan_id ),
       total_customers AS (
              SELECT COUNT(DISTINCT customer_id) as total_number 
                FROM subscriptions)

  SELECT plan_name, 
         COUNT(customer_id) as num_of_customers, 
         round((COUNT(customer_id)/total_number::numeric *100), 1) as percentage
    FROM plan_after_trial,total_customers
   WHERE rank = 2
GROUP BY 1,total_number;
```
I used a CTE(`plan_after_trial`) to rank the plan id for each customer and another CTE(`total_customers`) to get the total number of customers. I then queried the two CTEs for the plan name that had a rank of 2, counted the number of customers for each plan and calculated the percentage of the total.

Results:
| plan_name     | num_of_customers | percentage |
| ------------- | ---------------- | ----- |
| basic monthly | 546              | 54.6  |
| churn         | 92               | 9.2   |
| pro annual    | 37               | 3.7   |
| pro monthly   | 325              | 32.5  |

After trial;
* 546 customers continued with the basic monthly
* 92 customers churned
* 37 customers moved to the pro annual plan
* 325 customers moved to the 325 pro monthly 

-------------------------------

**Question 7**

What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```sql
WITH end_of_year AS (
       SELECT customer_id,
              plan_name,
              s.plan_id, 
              DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY start_date desc) as rank
         FROM subscriptions s
         JOIN plans p
           ON s.plan_id = p.plan_id
        WHERE start_date <= '2020-12-31' )
SELECT plan_name,COUNT(eoy.customer_id) as num_of_customers, COUNT(eoy.plan_name)*100/customers::double precision as percent_of_total
FROM end_of_year eoy,
(SELECT count(distinct customer_id) as customers FROM subscriptions) as total_cust
WHERE rank = 1
GROUP BY plan_name,total_cust.customers;
```
I used a CTE(`end_of_year`) to rank the plans in 2020 in descending order by start date so that the last plans were ranked first because that was the plan the customer
was on by the end of the year. I counted the number of customers and percentage of total for each plan name that had a rank of 1.

Results:
| plan_name     | num_of_customers | percent_of_total |
| ------------- | ---------------- | ---------------- |
| basic monthly | 224              | 22.4             |
| churn         | 236              | 23.6             |
| pro annual    | 195              | 19.5             |
| pro monthly   | 326              | 32.6             |
| trial         | 19               | 1.9              |

By the end of 2020,
* 224 customers were on the basic monthly
* 236 customers had churned
* 195 customers were on the pro annual plan
* 326 customers were on the pro monthly plan
* 19 customers were on trial

----------------------------

**Question 8**

How many customers have upgraded to an annual plan in 2020?

```sql
SELECT COUNT(customer_id) as annual_plan_count
FROM subscriptions s
WHERE plan_id = 3 and start_date <= '2020-12-31';
```

Technically all pro annual plans in 2020 were upgrades so I counted the number of customers on the pro annual plan in 2020

Results:
| annual_plan_count |
| ----------------- |
| 195               |

* 195 customers upgraded to the pro annual plan in 2020

------------------------

**Question 9**

How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

```sql
SELECT round(avg(s2.start_date-s1.start_date)) as average_days
  FROM subscriptions s1
  JOIN subscriptions s2
    ON s1.customer_id = s2.customer_id
 WHERE s1.plan_id = 0 and s2.plan_id = 3;
```
I joined the subscriptions table to itself and calculated the days between the date when a customer started the trial plan and the data they upgraded to a pro annual plan

Results:
| average_days |
| ----- |
| 105   |

* On average, it took 106 days for customers to upgrade to the pro annual plan from the start of their trial when they join Foodie-Fi

------------------------------

**Question 10**

Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

```sql
WITH upgrade_days as (SELECT
     (CASE
          WHEN (s2.start_date-s1.start_date) <= 30 THEN '0-30 days'
          WHEN (s2.start_date-s1.start_date) <= 60 THEN '31-60 days'
          WHEN (s2.start_date-s1.start_date) <= 90 THEN '61-90 days'
          WHEN (s2.start_date-s1.start_date) <= 120 THEN '91-120 days'
          WHEN (s2.start_date-s1.start_date) <= 150 THEN '121-150 days'
          WHEN (s2.start_date-s1.start_date) <= 180 THEN '151-180 days'
          WHEN (s2.start_date-s1.start_date) <= 210 THEN '181-210 days'
          WHEN (s2.start_date-s1.start_date) <= 240 THEN '211-240 days'
          WHEN (s2.start_date-s1.start_date) <= 270 THEN '241-270 days'
          WHEN (s2.start_date-s1.start_date) <= 300 THEN '271-300 days'
          WHEN (s2.start_date-s1.start_date) <= 330 THEN '301-330 days'
          WHEN (s2.start_date-s1.start_date) <= 360 THEN '331-360 days'END ) days_to_upgrade,
       round(avg(s2.start_date-s1.start_date)) as average_upgrade_days,
       count(s2.start_date-s1.start_date) as num_of_customers
     FROM subscriptions s1
     JOIN subscriptions s2
       ON s1.customer_id = s2.customer_id
    WHERE s1.plan_id = 0 and s2.plan_id = 3
 GROUP BY 1)

  SELECT days_to_upgrade, num_of_customers, average_upgrade_days
    FROM upgrade_days
   WHERE days_to_upgrade is not null 
GROUP BY 1,2,3
ORDER BY 3;
```
I used a CTE(`upgrade_days`) to get calculate the days between when a customer joined and when they upgraded to the pro annual plan. Then I used case when statements to break down the days into 30 days into a new column. I then queried the CTE for the days it took to upgrade, the number of customers and the average days it took the customers in that category to upgrade.

Results:

| days_to_upgrade | num_of_customers | average_upgrade_days |
| --------------- | ---------------- | -------------------- |
| 0-30 days       | 49               | 10                   |
| 31-60 days      | 24               | 42                   |
| 61-90 days      | 34               | 71                   |
| 91-120 days     | 35               | 101                  |
| 121-150 days    | 42               | 133                  |
| 151-180 days    | 36               | 162                  |
| 181-210 days    | 26               | 191                  |
| 211-240 days    | 4                | 224                  |
| 241-270 days    | 5                | 257                  |
| 271-300 days    | 1                | 285                  |
| 301-330 days    | 1                | 327                  |
| 331-360 days    | 1                | 346                  |

After joining Foodie-Fi:
* 49 customers upgraded to the pro annual plan between 0-30 days averaging 10 days
* 24 customers upgraded to the pro annual plan between 31-60 days averaging 42 days
* 34 customers upgraded to the pro annual plan between 61-90 days averaging 71 days
* 35 customers upgraded to the pro annual plan between 91-120 days averaging 101 days
* 42 customers upgraded to the pro annual plan between 121-150 days averaging 133 days
* 36 customers upgraded to the pro annual plan between 151-180 days averaging 162 days
* 26 customers upgraded to the pro annual plan between 181-210 days averaging 191 days
* 4 customers upgraded to the pro annual plan between 211-240 days averaging 224 days
* 5 customers upgraded to the pro annual plan between 241-270 days averaging 257 days
* 1 customer upgraded to the pro annual plan between 271-300 days averaging 285 days
* 1 customer upgraded to the pro annual plan between 301-330 days averaging 327 days
* 1 customer upgraded to the pro annual plan between 331-360 days averaging 346 days

---------------------------

**Question 11**

How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

```sql
WITH basic_monthly as (SELECT *
                         FROM subscriptions
                        WHERE plan_id = 2
                          AND start_date BETWEEN '2020-01-01' AND '2020-12-31'),
       pro_monthly as (SELECT *
                         FROM subscriptions
                        WHERE plan_id = 1
                          AND start_date BETWEEN '2020-01-01' AND '2020-12-31')

SELECT count(bm.customer_id) as num_of_customers
  FROM basic_monthly bm
  JOIN pro_monthly pm
    ON bm.customer_id = pm.customer_id
 WHERE pm.start_date > bm.start_date
```
I used a CTE(`basic_monthly`) to get all information for customers on the basic monthly plan and another CTE(`pro_monthly`) to get the same for customers on the pro monthly plan. If a customer downgraded to a basic monthly plan from a pro montly plan then they would have a later start date for basic monthly comapared to their start date for pro monthly. I queried the CTEs for basic monthly start dates that were later than pro monthly start dates for each customer that had been on both plans

Results:

| num_of_customers |
| ----- |
| 0     |

* No customer downgraded to a basic monthly plan from a pro monthly plan in 2020
 

-------------------
   
















