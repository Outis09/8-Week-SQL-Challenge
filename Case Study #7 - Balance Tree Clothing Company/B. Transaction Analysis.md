**Question 1:**
How many unique transactions were there?
-----

**Query:**

```sql
SELECT count(distinct(txn_id)) as num_of_txns
  FROM sales;
```

**Results:**

|num_of_txns|
|---------|
|2500|
