Using a single SQL query - create a new output table which has the following details:

* How many times was each product viewed?
* How many times was each product added to cart?
* How many times was each product added to a cart but not purchased (abandoned)?
* How many times was each product purchased?

**Query:**
```sql
WITH 
abandon as (
    SELECT page_name,
           count(event_type) as abandoned
      FROM events
      JOIN page_hierarchy
     USING (page_id)
     WHERE event_type = 2 and visit_id NOT IN (SELECT visit_id FROM events where event_type =3)
  GROUP BY page_name 
             ),

purchased as (
    SELECT page_name,
           count(event_type) as purchases
      FROM events
      JOIN page_hierarchy
     USING (page_id)
     WHERE event_type = 2 and visit_id IN (SELECT DISTINCT visit_id FROM events WHERE event_type =3)
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

Use your 2 new output tables - answer the following questions:

**Question 1**
Which product had the most views, cart adds and purchases?
-----
**Most views:**

**Query:**
```sql
  SELECT product,views
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
  SELECT product, cart_adds
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
  SELECT product,purchases
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
  SELECT product,abandoned
    FROM product_performance
ORDER BY abandoned DESC
   LIMIT 1;
```

**Results:**
| product        | abandoned |
| -------------- | --------- |
| Russian Caviar | 249       |

----------


Which product had the highest view to purchase percentage?
What is the average conversion rate from view to cart add?
What is the average conversion rate from cart add to purchase?




