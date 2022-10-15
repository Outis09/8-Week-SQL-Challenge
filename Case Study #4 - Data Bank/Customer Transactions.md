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

**Results:**

| txn_type   | num_of_unique_txn | total_amount |
| ---------- | ----------------- | ------------ |
| deposit    | 500               | 1359168      |
| withdrawal | 439               | 793003       |
| purchase   | 448               | 806537       |

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
