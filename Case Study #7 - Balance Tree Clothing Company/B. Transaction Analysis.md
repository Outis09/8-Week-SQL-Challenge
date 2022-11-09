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

--------------------------

**Question 2:**
What is the average unique products purchased in each transaction?
---

**Query:**

```sql
--gets number of products for each transaction
WITH avg_prods as (
    SELECT txn_id,
           count(distinct(prod_id)) as uniq_prods
      FROM sales
  GROUP BY 1
  )
  
SELECT round(avg(uniq_prods)) as avg_uniq_prods
  FROM avg_prods;
```

**Results:**

|avg_uniq_prods|
|---------|
|6|
