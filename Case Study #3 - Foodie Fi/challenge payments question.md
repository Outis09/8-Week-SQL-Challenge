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
