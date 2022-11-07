Using a single SQL query - create a new output table which has the following details:

* How many times was each product viewed?
* How many times was each product added to cart?
* How many times was each product added to a cart but not purchased (abandoned)?
* How many times was each product purchased?

**Query:**
```sql
WITH 
--gets number of time a product was added to cart but not purchased
abandon as (
    SELECT page_name,
           count(event_type) as abandoned
      FROM events
      JOIN page_hierarchy
     USING (page_id)
     WHERE event_type = 2 
       AND visit_id NOT IN (SELECT visit_id FROM events where event_type =3)
  GROUP BY page_name 
             ),
--gets number of times each product was purchased
purchased as (
    SELECT page_name,
           count(event_type) as purchases
      FROM events
      JOIN page_hierarchy
     USING (page_id)
     WHERE event_type = 2 
       AND visit_id IN (SELECT DISTINCT visit_id FROM events WHERE event_type =3)
  GROUP BY page_name
             )
	
    SELECT ph.page_name as product,
           sum(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) as views,
           sum(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) as cart_adds,
           abandoned,
           purchases
      INTO product_performance
      FROM page_hierarchy ph
      JOIN events e
        ON ph.page_id = e.page_id
      JOIN abandon ab
        ON ph.page_name = ab.page_name 
      JOIN purchased pc
        ON ph.page_name = pc.page_name 
  GROUP BY ph.page_name,abandoned,purchases;
```

```sql
SELECT *
  FROM product_performance
```

**Results:**
| product        | views | cart_adds | abandoned | purchases |
| -------------- | ----- | --------- | --------- | --------- |
| Oyster         | 1568  | 943       | 217       | 726       |
| Salmon         | 1559  | 938       | 227       | 711       |
| Abalone        | 1525  | 932       | 233       | 699       |
| Tuna           | 1515  | 931       | 234       | 697       |
| Black Truffle  | 1469  | 924       | 217       | 707       |
| Russian Caviar | 1563  | 946       | 249       | 697       |
| Lobster        | 1547  | 968       | 214       | 754       |
| Kingfish       | 1559  | 920       | 213       | 707       |
| Crab           | 1564  | 949       | 230       | 719       |

------------------------------------------

Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

**Query:**
```sql
WITH 
--gets number of times product from each category was added to cart and not purchased 
abandon as (
	SELECT product_category,
	       count(event_type) as abandoned
	  FROM events
	  JOIN page_hierarchy
	 USING (page_id)
	 WHERE event_type = 2 
	   AND visit_id NOT IN (SELECT visit_id FROM events where event_type =3)
      GROUP BY product_category),
--gets number of times products were purchased from each category
purchased as (
	SELECT product_category,
	       count(event_type) as purchases
	  FROM events
	  JOIN page_hierarchy
	 USING (page_id)
	 WHERE event_type = 2 
	   AND visit_id IN (SELECT DISTINCT visit_id FROM events WHERE event_type =3)
      GROUP BY product_category)
	
        SELECT ph.product_category,
	       sum(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) as views,
	       sum(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) as cart_adds,
	       abandoned,
	       purchases
	  INTO product_category_perf
	  FROM page_hierarchy ph
	  JOIN events e
	    ON ph.page_id = e.page_id
	  JOIN abandon ab
	    ON ph.product_category = ab.product_category 
	  JOIN purchased pc
	    ON ph.product_category = pc.product_category 
      GROUP BY ph.product_category,abandoned,purchases;
```

```sql
SELECT *
  FROM product_category_perf;
```

**Results:**
| product_category | views | cart_adds | abandoned | purchases |
| ---------------- | ----- | --------- | --------- | --------- |
| Fish             | 4633  | 2789      | 674       | 2115      |
| Shellfish        | 6204  | 3792      | 894       | 2898      |
| Luxury           | 3032  | 1870      | 466       | 1404      |

--------------------------------------------


Use your 2 new output tables - answer the following questions:

**Question 1**
Which product had the most views, cart adds and purchases?
-----
**Most views:**

**Query:**
```sql
  SELECT product,
         views
    FROM product_performance
ORDER BY views DESC
   LIMIT 1;
```

**Results:**
| product | views |
| ------- | ----- |
| Oyster  | 1568  |


**Most cart adds:**

**Query:**
```sql
  SELECT product, 
         cart_adds
    FROM product_performance
ORDER BY cart_adds DESC
   LIMIT 1;
```

**Results:**
| product | cart_adds |
| ------- | --------- |
| Lobster | 968       |

**Most purchases:**

**Query:**
```sql
  SELECT product,
         purchases
    FROM product_performance
ORDER BY purchases DESC
   LIMIT 1;
```

**Results:**
| product | purchases |
| ------- | --------- |
| Lobster | 754       |


**Question 2:**
Which product was most likely to be abandoned?
-----

**Query:**
```sql
  SELECT product,
         abandoned
    FROM product_performance
ORDER BY abandoned DESC
   LIMIT 1;
```

**Results:**
| product        | abandoned |
| -------------- | --------- |
| Russian Caviar | 249       |

----------

I created the report below in Power BI. It contains
* total number of views
* total number of cart adds
* total number of purchases
* total number of abandoned products
* top 3 products by views
* top 3 products by cart adds
* top 3 products by puchases
* top 3 abandoned products

![clique2](https://user-images.githubusercontent.com/104911707/200275536-49e63f29-28ff-4782-82e3-6bc2b58098b9.png)

-----------------------------------




**Question 3:**
Which product had the highest view to purchase percentage?
-------

**Query:**
```sql
  SELECT product,
         views,
	 purchases,
	 round((100*purchases::numeric/views),2) as prchs_vw_percent
    FROM product_performance
ORDER BY prchs_vw_percent DESC;
```

**Results:**

| product        | views | purchases | prchs_vw_percent |
| -------------- | ----- | --------- | ---------------- |
| Lobster        | 1547  | 754       | 48.74            |
| Black Truffle  | 1469  | 707       | 48.13            |
| Oyster         | 1568  | 726       | 46.30            |
| Tuna           | 1515  | 697       | 46.01            |
| Crab           | 1564  | 719       | 45.97            |
| Abalone        | 1525  | 699       | 45.84            |
| Salmon         | 1559  | 711       | 45.61            |
| Kingfish       | 1559  | 707       | 45.35            |
| Russian Caviar | 1563  | 697       | 44.59            |

-------------------------------

**Question 4:**
What is the average conversion rate from view to cart add?
-------

**Query:**
```sql
SELECT round(100*sum(cart_adds)::numeric/sum(views),2) as avg_cnvrsn_rate
  FROM product_performance;
```
**Results:**
| avg_cnvrsn_rate |
| ----- |
| 60.93 |

-------------------------

**Question 5:**
What is the average conversion rate from cart add to purchase?
------

**Query:**
```sql
SELECT round(100*sum(purchases)::numeric/sum(cart_adds),2) as avg_cnvrsn_rate
FROM product_performance;
```

**Results:**
| avg_cnvrsn_rate |
| ----- |
| 75.93 |

----------------------------





