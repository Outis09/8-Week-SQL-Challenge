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
