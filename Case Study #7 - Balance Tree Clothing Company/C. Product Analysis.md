**Question 1:**
What are the top 3 products by total revenue before discount?
-----

**Query:**
```sql
  SELECT product_name,
         sum(qty*s.price) as revenue
    FROM product_details pd
    JOIN sales s
      ON pd.product_id = s.prod_id
GROUP BY product_name
ORDER BY revenue DESC
   LIMIT 3;
```

**Results:**

| product_name                 | revenue |
| ---------------------------- | ------- |
| Blue Polo Shirt - Mens       | 217683  |
| Grey Fashion Jacket - Womens | 209304  |
| White Tee Shirt - Mens       | 152000  |

--------------------------------

**Question 2:**
What is the total quantity, revenue and discount for each segment?
-----

**Query:**

```sql
  SELECT segment_name,
         sum(qty) as tot_qty,
         sum(qty*s.price) as tot_revenue,
         round(sum((qty*s.price)*(discount::numeric/100)),2)as tot_discount
    FROM product_details pd
    JOIN sales s
      ON pd.product_id = s.prod_id
GROUP BY segment_name;
```

**Results:**

|segment_name|tot_qty|tot_revenue|tot_discount|
|------------|-------|-----------|------------|
|Shirt|11265|406143|49594.27|
|Jeans|11349|208350|25343.97|
|Jacket|11385|366983|44277.46|
|Socks|11217|307977|37013.44|

--------------------------------------------------------------------

**Question 3:**
What is the top selling product for each segment?
---

**Query:**
```sql
WITH seg_prod_rank as (
    SELECT segment_name,
           product_name,
           sum(qty*s.price) as revenue,
           RANK() OVER (PARTITION BY segment_name ORDER BY SUM(qty*s.price) DESC)
      FROM product_details pd
      JOIN sales s
        ON pd.product_id = s.prod_id
  GROUP BY 1,2)

SELECT segment_name,
       product_name,
	     revenue
  FROM seg_prod_rank
 WHERE rank = 1;
```

**Results:**

|segment_name|product_name|revenue|
|------------|------------|------|
|Jacket|Grey Fashion Jacket - Womens|209304
|Jeans|Black Straight Jeans - Womens|121152|
|Shirt|Blue Polo Shirt - Mens|217683|
|Socks|Navy Solid Socks - Mens|136512|


Alternative

**Query:**

```sql
WITH seg_prod_rank as (
      SELECT segment_name,
             product_name,
             sum(qty) as tot_qty,
             RANK() OVER (PARTITION BY segment_name ORDER BY sum(qty) DESC)
        FROM product_details pd
        JOIN sales s
          ON pd.product_id = s.prod_id
    GROUP BY 1,2)

SELECT segment_name,
       product_name,
	     tot_qty
  FROM seg_prod_rank
 WHERE rank = 1;
```

**Results:**

|segment_name|product_name|tot_qty|
|------------|------------|------|
|Jacket|Grey Fashion Jacket - Womens|3876
|Jeans|Navy Oversized Jeans - Womens|3856|
|Shirt|Blue Polo Shirt - Mens|3819|
|Socks|Navy Solid Socks - Mens|3792|

----------------------------

