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
--gets the revenue for each product in each segment
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
--gets the quantity for each ptoduct in each segment
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

**Question 4:**
What is the total quantity, revenue and discount for each category?
-----

**Query:**

```sql
  SELECT category_name,
         sum(qty) as tot_qty,
	 sum(qty*s.price) as tot_revenue,
	 round(sum((qty*s.price)*(discount::numeric/100)),2)as tot_discount
    FROM product_details pd
    JOIN sales s
      ON pd.product_id =s.prod_id
GROUP BY category_name;
```
**Results:**

|category_name|tot_qty|tot_revenue|tot_discount|
|-------------|-------|------------|-----------|
|Mens|22482|714120|86607.71|
|Womens|22734|575333|69621.43|

------------------------------------

**Question 5:**
What is the top selling product for each category?
-----

**Query:**

```sql
--gets the revenue for each product in each category
WITH cat_prod_rank as (
	SELECT category_name,
	       product_name,
	       sum(qty*s.price) as revenue,
	       RANK() OVER (PARTITION BY category_name ORDER BY SUM(qty*s.price) DESC)
	  FROM product_details pd
	  JOIN sales s
	    ON pd.product_id = s.prod_id
      GROUP BY 1,2)

SELECT category_name,
       product_name,
       revenue
  FROM cat_prod_rank
 WHERE rank = 1;
```

**Results:**

|category_name|product_name|revenue|
|-------------|----------|----------|
|Mens|Blue Polo Shirt - Mens|217683|
|Womens|Grey Fashion Jacket - Womens|209304|

Alternative:

**Query:**

```sql
--gets the quantity of each product in each category
WITH cat_prod_rank as (
	SELECT category_name,
	       product_name,
	       sum(qty) as tot_qty,
	       RANK() OVER (PARTITION BY category_name ORDER BY sum(qty) DESC)
	  FROM product_details pd
	  JOIN sales s
	    ON pd.product_id = s.prod_id
      GROUP BY 1,2
                         )

SELECT category_name,
       product_name,
       tot_qty
  FROM cat_prod_rank
  WHERE rank = 1;
```
**Results:**

|category_name|product_name|revenue|
|-------------|----------|----------|
|Mens|Blue Polo Shirt - Mens|3819|
|Womens|Grey Fashion Jacket - Womens|3876|

----------------------------------

**Question 6:**
What is the percentage split of revenue by product for each segment?
-----

**Query:**

```sql
--gets the revenue and percentage of revenue for each product in each segment
WITH seg_percent as (
	SELECT segment_name,
	       product_name,
	       sum(qty*s.price) as revenue,
	       100*sum(qty*s.price)/sum(sum(qty*s.price)) OVER (PARTITION BY segment_name) as percent_of_seg_rev
	  FROM product_details pd
	  JOIN sales s
	    ON pd.product_id = s.prod_id
      GROUP BY 1,2)

SELECT segment_name as segment,
       product_name as product,
       revenue,
       round(percent_of_seg_rev,2) as percent_of_seg_rev
  FROM seg_percent;
```

**Results:**

| segment | product                          | revenue | percent_of_seg_rev |
| ------- | -------------------------------- | ------- | ----- |
| Jacket  | Indigo Rain Jacket - Womens      | 71383   | 19.45 |
| Jacket  | Khaki Suit Jacket - Womens       | 86296   | 23.51 |
| Jacket  | Grey Fashion Jacket - Womens     | 209304  | 57.03 |
| Jeans   | Navy Oversized Jeans - Womens    | 50128   | 24.06 |
| Jeans   | Black Straight Jeans - Womens    | 121152  | 58.15 |
| Jeans   | Cream Relaxed Jeans - Womens     | 37070   | 17.79 |
| Shirt   | White Tee Shirt - Mens           | 152000  | 37.43 |
| Shirt   | Blue Polo Shirt - Mens           | 217683  | 53.60 |
| Shirt   | Teal Button Up Shirt - Mens      | 36460   | 8.98  |
| Socks   | Navy Solid Socks - Mens          | 136512  | 44.33 |
| Socks   | White Striped Socks - Mens       | 62135   | 20.18 |
| Socks   | Pink Fluro Polkadot Socks - Mens | 109330  | 35.50 |

------------------------------------

**Question 7:**
What is the percentage split of revenue by segment for each category?
-----

**Query:**
```sql
--gets revenue and revenue percentage for each segment in each category
WITH cat_percent as (
	SELECT category_name,
	       segment_name,
	       sum(qty*s.price) as revenue,
	       100*sum(qty*s.price)/sum(sum(qty*s.price)) OVER (PARTITION BY category_name) as percent_of_cat_rev
	  FROM product_details pd
	  JOIN sales s
	    ON pd.product_id = s.prod_id
      GROUP BY 1,2)

SELECT category_name as category,
       segment_name as segment,
       revenue,
       round(percent_of_cat_rev,2)
  FROM cat_percent;
```

**Results:**

| category | segment | revenue | round |
| -------- | ------- | ------- | ----- |
| Mens     | Socks   | 307977  | 43.13 |
| Mens     | Shirt   | 406143  | 56.87 |
| Womens   | Jeans   | 208350  | 36.21 |
| Womens   | Jacket  | 366983  | 63.79 |

--------------------------

**Question 8:**
What is the percentage split of total revenue by category?
-----

**Query:**
```sql
--gets the revenue and revenue percentage for each category
WITH cat_rev as (
	SELECT category_name,
	       sum(qty*s.price) as revenue,
	       100*sum(qty*s.price)/sum(sum(qty*s.price)) OVER () as rev_percent
	  FROM product_details pd
	  JOIN sales s 
	    ON pd.product_id = s.prod_id
      GROUP BY 1)

SELECT category_name as category,
       revenue,
       round(rev_percent,2) as rev_percent
  FROM cat_rev;
```

**Results:**
| category | revenue | rev_percent |
| -------- | ------- | ----- |
| Mens     | 714120  | 55.38 |
| Womens   | 575333  | 44.62 |

-----------------------------
