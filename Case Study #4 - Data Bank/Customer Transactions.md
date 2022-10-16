**QUESTION 1:**
What is the unique count and total amount for each transaction type?
-----

```sql
  SELECT DISTINCT(txn_type),
         COUNT(DISTINCT(customer_id)) as num_of_unique_txn,
         SUM(txn_amount) as total_amount
    FROM customer_transactions
GROUP BY 1;
```
I selected the distinct transaction types which i used to group the count of the distinct number of customers and total amount. So the query will display the distinct transaction type, the number of transactions from distinct customers and the sum of transactions for each transaction type.

**Results:**

| txn_type   | num_of_unique_txn | total_amount |
| ---------- | ----------------- | ------------ |
| deposit    | 500               | 1359168      |
| withdrawal | 439               | 793003       |
| purchase   | 448               | 806537       |

* 500 unique deposits with a total amount of US$1,359,168
* 439 unique withdrawals with a total sum of US$793,003
* 448 unique purchases with a total sum of US$806,537
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

**Results:**

| avg_deposit_count | avg_deposit_amt |
| ----------------- | --------------- |
| 5                 | 2718            |

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
**Results:**

| month                    | num_of_customers |
| ------------------------ | ---------------- |
| 2020-01-01    | 168              |
| 2020-02-01    | 181              |
| 2020-03-01    | 192              |
| 2020-04-01    | 70               |


