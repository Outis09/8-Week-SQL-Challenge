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


**Question 3:**
What are the 25th, 50th and 75th percentile values for the revenue per transaction?
-----

**Query:**

```sql
--gets revenue for each transaction
WITH txn_revenue as (
      SELECT txn_id,
             ROUND(sum((price*qty)*((100-discount::numeric)/100))) as revenue
        FROM sales
    GROUP BY 1
           )

SELECT percentile_cont(0.25) WITHIN GROUP(ORDER BY revenue) as percentile_25,
       percentile_cont(0.5) WITHIN GROUP(ORDER BY revenue) as percentile_50,
	     percentile_cont(0.75) WITHIN GROUP(ORDER BY revenue) as percentile_75
  FROM txn_revenue;
```

**Results:**

| percentile_25 | percentile_50 | percentile_75 |
| ------------- | ------------- | ------------- |
| 326           | 441           | 573           |

--------------------------------

**Question 4:**
What is the average discount value per transaction?
-----

**Query:**

```sql
--gets discount for each transaction
WITH avg_discount as (
	SELECT txn_id,
	       sum((price*qty)*(discount::numeric/100)) as discount
	  FROM sales
      GROUP BY 1)

SELECT round(avg(discount),2) as avg_discount
  FROM avg_discount;
```

**Results:**

| avg_discount |
| ------------ |
| 62.49        |

----------------------------

**Question 5:**
What is the percentage split of all transactions for members vs non-members?
-----

**Query:**

```sql
SELECT round(100*sum(CASE WHEN member = 't' THEN 1 ELSE 0 END)::numeric/count(txn_id)) as member_percentage,
       round(100*sum(CASE WHEN member = 'f' THEN 1 ELSE 0 END)::numeric/count(txn_id)) as non_member_percentage
  FROM sales;
```

**Results:**

| member_percentage | non_member_percentage |
| ----------------- | --------------------- |
| 60                | 40                    |



