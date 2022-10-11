**Question:**

The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
once a customer churns they will no longer make payments
Example outputs for this table might look like the following:

| customer_id | plan_id | plan_name     | payment_date             | amount | payment_order |
| ----------- | ------- | ------------- | ------------------------ | ------ | ------------- |
| 1           | 1       | basic monthly | 2020-08-08  | 9.90   | 1             |
| 1           | 1       | basic monthly | 2020-09-08  | 9.90   | 2             |
| 1           | 1       | basic monthly | 2020-10-08  | 9.90   | 3             |
| 1           | 1       | basic monthly | 2020-11-08  | 9.90   | 4             |
| 1           | 1       | basic monthly | 2020-12-08  | 9.90   | 5             |
| 2           | 3       | pro annual    | 2020-09-27  | 199.00 | 1             |
| 13          | 1       | basic monthly | 2020-12-22  | 9.90   | 1             |
| 15          | 2       | pro monthly   | 2020-03-24  | 19.90  | 1             |
| 15          | 2       | pro monthly   | 2020-04-24  | 19.90  | 2             |
| 16          | 1       | basic monthly | 2020-06-07  | 9.90   | 1             |
| 16          | 1       | basic monthly | 2020-07-07  | 9.90   | 2             |
| 16          | 1       | basic monthly | 2020-08-07  | 9.90   | 3             |
| 16          | 1       | basic monthly | 2020-09-07  | 9.90   | 4             |
| 16          | 1       | basic monthly | 2020-10-07  | 9.90   | 5             |
| 16          | 3       | pro annual    | 2020-10-21  | 189.10 | 6             |
| 18          | 2       | pro monthly   | 2020-07-13  | 19.90  | 1             |
| 18          | 2       | pro monthly   | 2020-08-13  | 19.90  | 2             |
| 18          | 2       | pro monthly   | 2020-09-13  | 19.90  | 3             |
| 18          | 2       | pro monthly   | 2020-10-13  | 19.90  | 4             |
| 18          | 2       | pro monthly   | 2020-11-13  | 19.90  | 5             |
| 18          | 2       | pro monthly   | 2020-12-13  | 19.90  | 6             |
| 19          | 2       | pro monthly   | 2020-06-29  | 19.90  | 1             |
| 19          | 2       | pro monthly   | 2020-07-29  | 19.90  | 2             |
| 19          | 3       | pro annual    | 2020-08-29  | 199.00 | 3             |

-----------------------------------------------------------------------------------------

**SOLUTION**

```sql
set search_path to foodie_fi;
	 
CREATE TEMP TABLE payments_date AS (
SELECT customer_id,
       plan_id,
       generate_series( 
              start_date :: timestamp,
              CASE WHEN plan_id = 3 THEN start_date
                   WHEN plan_id = 4 THEN null
                   WHEN LEAD(start_date) over(partition by customer_id order by start_date) IS NOT NULL 
                        THEN LEAD(start_date)over(partition by customer_id order by start_date)
                   ELSE '2021-01-01' end,
               '1 month'+ '1 minute'::interval
                        )::date as payment_date
FROM subscriptions 
WHERE start_date < '2021-01-01' and plan_id != 0
);

CREATE TABLE payments AS (	
SELECT  customer_id, 
        pd.plan_id,
        plan_name,
        payment_date,
        CASE WHEN pd.plan_id != 1 and payment_date - LAG(payment_date) OVER(PARTITION BY customer_id ORDER BY payment_date ) < 30 
                  THEN price - LAG(price) OVER (PARTITION BY customer_id ORDER BY payment_date)
             ELSE price
             END AS amount,
        RANK() OVER(PARTITION BY  customer_id ORDER BY payment_date) as payment_order
   FROM payments_date as pd
   JOIN plans as p
     ON pd.plan_id = p.plan_id
ORDER BY customer_id,pd.plan_id,p.plan_name,pd.payment_date
);

SELECT *
  FROM payments
 WHERE customer_id in (1,2,13,15,16,18,19)
```
I wrote three different queries. 

First query returns a temporary table which was used to create and store the payment dates using the conditions given by the Foodie-Fi team. 

In the second query, the `payments` table contains the information in the temporary table and  a new column to calculate the amount a customer spent on the subscription. The `plans` table is joined to the `payments` table so that the names of subscriptions can be displayed. The `RANK()` function was used to rank the subscriptions for each customer according to the start date. 

In the third query, information on a  select customer IDs were displayed to look like the preview table given by the Foodie-Fi team to make for easy comparison. The full table contains over 4000 rows. 

Results:

| customer_id | plan_id | plan_name     | payment_date             | amount | payment_order |
| ----------- | ------- | ------------- | ------------------------ | ------ | ------------- |
| 1           | 1       | basic monthly | 2020-08-08  | 9.90   | 1             |
| 1           | 1       | basic monthly | 2020-09-08  | 9.90   | 2             |
| 1           | 1       | basic monthly | 2020-10-08  | 9.90   | 3             |
| 1           | 1       | basic monthly | 2020-11-08  | 9.90   | 4             |
| 1           | 1       | basic monthly | 2020-12-08  | 9.90   | 5             |
| 2           | 3       | pro annual    | 2020-09-27  | 199.00 | 1             |
| 13          | 1       | basic monthly | 2020-12-22  | 9.90   | 1             |
| 15          | 2       | pro monthly   | 2020-03-24  | 19.90  | 1             |
| 15          | 2       | pro monthly   | 2020-04-24  | 19.90  | 2             |
| 16          | 1       | basic monthly | 2020-06-07  | 9.90   | 1             |
| 16          | 1       | basic monthly | 2020-07-07  | 9.90   | 2             |
| 16          | 1       | basic monthly | 2020-08-07  | 9.90   | 3             |
| 16          | 1       | basic monthly | 2020-09-07  | 9.90   | 4             |
| 16          | 1       | basic monthly | 2020-10-07  | 9.90   | 5             |
| 16          | 3       | pro annual    | 2020-10-21  | 189.10 | 6             |
| 18          | 2       | pro monthly   | 2020-07-13  | 19.90  | 1             |
| 18          | 2       | pro monthly   | 2020-08-13  | 19.90  | 2             |
| 18          | 2       | pro monthly   | 2020-09-13  | 19.90  | 3             |
| 18          | 2       | pro monthly   | 2020-10-13  | 19.90  | 4             |
| 18          | 2       | pro monthly   | 2020-11-13  | 19.90  | 5             |
| 18          | 2       | pro monthly   | 2020-12-13  | 19.90  | 6             |
| 19          | 2       | pro monthly   | 2020-06-29  | 19.90  | 1             |
| 19          | 2       | pro monthly   | 2020-07-29  | 19.90  | 2             |
| 19          | 3       | pro annual    | 2020-08-29  | 199.00 | 3             |
