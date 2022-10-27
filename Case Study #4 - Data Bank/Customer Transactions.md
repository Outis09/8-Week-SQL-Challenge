**QUESTION 1:**
What is the unique count and total amount for each transaction type?
-----

```sql
  SELECT DISTINCT(txn_type),
         COUNT(txn_type) as num_of_txns,
         SUM(txn_amount) as total_amount
    FROM customer_transactions
GROUP BY 1;
```
I selected the distinct transaction types which I used to group the count of the distinct number of customers and total amount. So the query will display the distinct transaction type, the number of transactions from distinct customers and the sum of transactions for each transaction type.

**Results:**

| txn_type   | num_of_txns | total_amount |
| ---------- | ----------------- | ------------ |
| deposit    | 2671       | 1359168      |
| withdrawal | 1617              | 793003       |
| purchase   | 1580              | 806537       |

* 2671 deposits with a total amount of US$1,359,168
* 1617 withdrawals with a total sum of US$793,003
* 1580 purchases with a total sum of US$806,537
----------------------------------------------------------------------------------------------------

**QUESTION 2:**
What is the average total historical deposit counts and amounts for all customers?
-----

```sql
SELECT COUNT(txn_type)/COUNT(DISTINCT(customer_id)) as avg_deposit_count,
       SUM(txn_amount)/COUNT(DISTINCT(customer_id)) as avg_deposit_amt
  FROM customer_transactions
 WHERE txn_type = 'deposit';
```
I filtered for transactions that were only deposits. I then counted the number of transactions (deposits) and divided by the number of distinct customers. I also summed the total of transactions (deposits) and divided by the number of unique customers.

**Results:**

| avg_deposit_count | avg_deposit_amt |
| ----------------- | --------------- |
| 5                 | 2718            |

* The average deposit count for all customers is 5
* The average deposit amount for all customers is US$2,718
-------------------------------------------------------------------------------------------------------

**QUESTION 3:**
For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
-----

```sql
WITH dep_txn as (
          SELECT customer_id, 
                 DATE_TRUNC('month',txn_date) as txn_date,
                 CASE WHEN count(txn_type) > 1 THEN 1 ELSE null
                      END AS multi_depo
            FROM customer_transactions
           WHERE txn_type = 'deposit'
        GROUP BY 1,2
        ORDER BY 1,2
               ),

other_txns as (
          SELECT customer_id,
                 DATE_TRUNC('month',txn_date) as txn_date,
                 CASE WHEN count(txn_type) >= 1 THEN 1 ELSE null
                      END AS  other_txn
            FROM customer_transactions
           WHERE txn_type != 'deposit'
        GROUP BY 1,2
        ORDER BY 1,2
               )

  SELECT dt.txn_date as month,
         COUNT(dt.customer_id) as num_of_customers
    FROM dep_txn as dt
    JOIN other_txns as ot
      ON dt.txn_date = ot.txn_date and dt.customer_id = ot.customer_id
   WHERE multi_depo = 1 and other_txn = 1
GROUP BY 1
```
I created two CTEs. In the first CTE,`dep_txn`, I filtered for transactions that were deposits. Then, I selected the customer IDs and  truncated the transaction date to month. Using a `CASE WHEN` statement, I created a new column,`multi_depo`, where 1 was assigned to customers with more than 1 deposit in a month and null for those less than 1.

In the second CTE,`other_txns`, I filtered for transactions that were not deposits. Then, I selected the customer IDs and truncated the transaction date to month. I used a `CASE WHEN` statement to create a new column,`other_txn`, to assign 1 to customers who had at least 1 transaction for a given month. Because I had filtered to leave out deposits, this transaction could be either withdrawals or purchases.

In the main query, I joined the two CTEs and  filtered for customers who for a given month had more than 1 deposit and 1 withdrawal or deposit. That is, 1 in the `multi_depo` column and 1 in the `other_txn` column. I selected the truncated transaction dates, and counted the number of customers who met the condition. I grouped the results by the truncated date.

**Results:**

| month                    | num_of_customers |
| ------------------------ | ---------------- |
| 2020-01-01    | 168              |
| 2020-02-01    | 181              |
| 2020-03-01    | 192              |
| 2020-04-01    | 70               |

* In January 2020, 168 customers had more than 1 deposit and at least 1 purchase or withdrawal.
* In February 2020, 181 customers had more than 1 deposit and at least 1 purchase or withdrawal.
* In March 2020, 192 customers had more than 1 deposit and at least 1 purchase or withdrawal.
* In April 2020, 70 customers had more than 1 deposit and at least 1 purchase or withdrawal.

----------------------------------------------------

**QUESTION 4:**
What is the closing balance for each customer at the end of the month?
-----

```sql
--summing transactions for each month
WITH sum_txns as (
        SELECT customer_id,
               date_trunc('month',txn_date)+ interval '1 month' - interval '1 day' as eomonth,
               SUM(CASE WHEN txn_type = 'deposit' then txn_amount
                        ELSE -txn_amount
                        END) as amount
          FROM customer_transactions
      GROUP BY customer_id,eomonth
      ORDER BY customer_id,eomonth
	),
-- generate closing dates till end of April
closing_dates as (
        SELECT DISTINCT customer_id,
               to_date('2020-01-31','YYYY-MM-DD') + (generate_series(0,3,1)*interval '1 month') as month
          FROM customer_transactions
      ORDER BY customer_id,month)

   SELECT cd.customer_id,
          to_char(month,'month') as month,
          SUM(amount) OVER(PARTITION BY cd.customer_id ORDER BY month) as closing_balance
     FROM closing_dates cd
LEFT JOIN sum_txns st
       ON cd.customer_id = st.customer_id and cd.month = st.eomonth
 ORDER BY cd.customer_id,ROW_NUMBER() OVER(PARTITION BY cd.customer_id ORDER BY cd.month);
```

**Results:**

| customer_id | month     | closing_balance |
| ----------- | --------- | --------------- |
| 1           | january   | 312             |
| 1           | february  | 312             |
| 1           | march     | -640            |
| 1           | april     | -640            |
| 2           | january   | 549             |
| 2           | february  | 549             |
| 2           | march     | 610             |
| 2           | april     | 610             |
| 3           | january   | 144             |
| 3           | february  | -821            |
| 3           | march     | -1222           |
| 3           | april     | -729            |
| 4           | january   | 848             |
| 4           | february  | 848             |
| 4           | march     | 655             |
| 4           | april     | 655             |
| 5           | january   | 954             |
| 5           | february  | 954             |
| 5           | march     | -1923           |
| 5           | april     | -2413           |

**NB:** Only the first five customers are displayed because the full results contains 2000 rows.

-------------------------------------------------------------

**QUESTION 5:**
What is the percentage of customers who increase their closing balance by more than 5%?
------

```sql
WITH sum_txns as (
	SELECT customer_id,
	       date_trunc('month',txn_date)+ interval '1 month' - interval '1 day' as eomonth,
	       SUM(CASE WHEN txn_type = 'deposit' then txn_amount
	                ELSE -txn_amount
			END) as amount
	 FROM customer_transactions
     GROUP BY customer_id,eomonth
     ORDER BY customer_id,eomonth
	),
-- generate closing dates till end of April
closing_dates as (
	SELECT DISTINCT customer_id,
	       to_date('2020-01-31','YYYY-MM-DD') + (generate_series(0,3,1)*interval '1 month') as month
	  FROM customer_transactions
      ORDER BY customer_id,month),
--closing balance for each month
closing_balances as(
	SELECT cd.customer_id,
	       ROW_NUMBER() OVER(PARTITION BY cd.customer_id ORDER BY cd.month) as month_id,
	       to_char(month,'month') as month,
	       SUM(amount) OVER(PARTITION BY cd.customer_id ORDER BY month) as closing_balance
	  FROM closing_dates cd
     LEFT JOIN sum_txns st
            ON cd.customer_id = st.customer_id and cd.month = st.eomonth
      ORDER BY cd.customer_id,month_id
                     ),
--closing balances for the previous month
prev_balances as(
SELECT *,
       LAG(closing_balance) OVER(PARTITION BY customer_id ORDER BY month_id) as prev_closing_bal
  FROM closing_balances
	   )
SELECT round((count(distinct(customer_id))::numeric/(select count(distinct customer_id)::numeric from customer_transactions))*100,2) as percentage
  FROM prev_balances
 WHERE closing_balance > (105/100)*prev_closing_bal and prev_closing_bal::text not like '-%'
 ```
