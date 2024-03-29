**Question 1:**
How many unique transactions were there?
-----

**Query:**

```sql
SELECT count(distinct(txn_id)) as num_of_txns
  FROM sales;
```

Each transactio has a unique transaction ID so I counted the number of unique IDs in the sales table.
**Results:**

|num_of_txns|
|---------|
|2500|

* There are 2500 unique transactions

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

In the CTE,`avg_prods`, I counted the distinct number of products for each transaction ID. In the main query, I calculated the average number of unique products and rounded to the next whole number.

**Results:**

|avg_uniq_prods|
|---------|
|6|

* The average unique products in each transaction is 6.

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

The CTE,`txn_revenue`, gets the revenue for each transaction. NB: my calculation involved discount. Therefore if discount was 17% then revenue after discount was 83%.
In the main query, I calculated the 25th, 50th and 75th percentile values for revenue.
**Results:**

| percentile_25 | percentile_50 | percentile_75 |
| ------------- | ------------- | ------------- |
| 326           | 441           | 573           |

* The 25th percentile for revenue is $326
* The 50th percentile for revenue is $441
* The 75th percentile for revenue is $573


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

The CTE,`avg_discount`,gets the discount for each transaction by multiplying the discount percentage with the results of the price * quantity for each product in each transaction. In the main query, I calculated the average discount and rounded to two decimal places.
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


-------------------------------------

**Question 6**
What is the average revenue for member transactions and non-member transactions?
-----

**Query:**

```sql
WITH 
--gets revenue for members
mem_rev as (
	SELECT ROUND((sum((price*qty)*((100-discount::numeric)/100)))/count(distinct(txn_id))) as avg_member_revenue
	  FROM sales
	 WHERE member = 't'),
--gets revenue for non-members
non_mem_rev as (
	SELECT ROUND((sum((price*qty)*((100-discount::numeric)/100)))/count(distinct(txn_id))) as avg_non_member_revenue
	  FROM sales
	 WHERE member = 'f')

SELECT avg_member_revenue,avg_non_member_revenue
FROM mem_rev, non_mem_rev;
```

**Results:**

| avg_member_revenue | avg_non_member_revenue |
| ------------------ | ---------------------- |
| 454                | 452                    |

--------------
