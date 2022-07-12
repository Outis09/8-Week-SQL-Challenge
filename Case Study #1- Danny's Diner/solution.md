```sql
SET search_path TO dannys_diner;
```
-------------


**QUESTION 1:** What is the total amount each customer spent at the restaurant?


```sql
SELECT s.customer_id, 
       SUM(m.price) as total_spent
FROM sales s
JOIN menu m
  ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 2 DESC;
```

customer_id	|total_spent
---------|---------
A	|76
B	|74
C	|36

* Customer A spent $76
* Customer B spent spent $74
* Customer C spent $36 

**I ordered by the total amount spent to show the highest spender first.**

Customer A spent the most money.

-------------

**QUESTION 2:** How many days has each customer visited the restaurant?

```sql
SELECT customer_id, 
       COUNT(DISTINCT(order_date)) AS days_visited
FROM sales 
GROUP BY 1;
```

I used `distinct` to count unique days .
customer_id|	days_visited
---------|--------
A	|4
B	|6
C	|2

* Customer A visited the restaurant on 4 different days
* Customer B visited on 6 different days
* Customer C visited on 2 different days.

------------

**QUESTION 3:** What was the first item from the menu purchased by each customer?

```sql
WITH first_item AS (
	SELECT customer_id, 
	       product_name,
	       DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS ranks
	FROM sales
	JOIN menu
        USING (product_id) )
SELECT customer_id, product_name
FROM first_item
WHERE ranks = 1
GROUP BY 1,2;
```
I created a CTE (`first_item`) and used the dense rank windows function to rank all orders for each customer by partititioning by `customer_id` and ordering by `order_date`. The earliest orders are ranked as 1 and the rest follow. Then I join the menu table to get the name of the products in customer's first purchases. For customer A, two items were purchased on the first day and there is no time of order therefore this query returns two items as first purchase for customer A.

customer_id|	product_name
--------|-------
A	|curry
A	|sushi
B	|curry
C	|ramen

* Customer A's first purchase was curry and sushi
* Customer B's first item purchased was curry
* Customer C's first item purchased was ramen 

However, it is possible to limit customer A's first order to one item by adding s.product_id to the `ORDER BY` in the windows function. Even though the following query returns one item, it is not certain if that is actually the first order because the time of purchase is what can show exactly which item was purchased first and the data does not provide the time of purchase.

```sql
WITH t1 AS (
	SELECT customer_id, 
	       product_name,
	       DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date, s.product_id) AS ranks
        FROM sales s
        JOIN menu
        USING (product_id) )
SELECT customer_id, product_name
FROM t1
WHERE ranks = 1
GROUP BY 1,2;
```
customer_id	|product_name
--------|--------
A	|sushi
B	|curry
C	|ramen

------------

**QUESTION 4:** What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT  m.product_name as fav_prod,
	COUNT(s.*) as order_count
FROM sales s 
JOIN menu m 
ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

fav_prod |order_count
------|-------
ramen|	8

* The most purchased item is `ramen` and it was purchased 8 times.

-----------

**QUESTION 5:** Which item was the most popular for each customer?

```sql
WITH fav_item AS (
		SELECT s.customer_id,
		       m.product_name, 
		       COUNT(s.product_id) AS quantity,
		       DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC ) AS ranks
		FROM sales s
	        JOIN menu m
		   ON s.product_id = m.product_id
	        GROUP BY 1,2)
SELECT customer_id,product_name,quantity
FROM fav_item
WHERE ranks = 1;
```

I created a CTE (fav_item) to create a temporary table that contained `customer_id`, `product_name`, a count of purchases and I used `dense_rank()` to rank the count of `product_id` in descending order for each customer. I joined the `menu` table so I could use the name associated with the `product_id`.


customer_id |product_name |quantity
--------|--------|--------
A	|ramen	|3
B	|sushi	|2
B	|curry	|2
B	|ramen	|2
C	|ramen	|3


* Customer A's most popular item is ramen and it was purchased 3 times. 
* Customer B purchased all items equally.
* Customer C's most popular item was ramen and it was ordered 3 times.


-----------
I assumed that any order placed on `join_date` was placed after the customer had become a member.

------------
**QUESTION 6:** Which item was purchased first by the customer after they became a member?

```sql
WITH purchase_after_membership AS (
				SELECT s.customer_id,
				       s.order_date,
				       m.product_name,
				       me.join_date,
				       DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS ranks
				FROM sales s
				JOIN members me
				ON s.customer_id = me.customer_id
				JOIN menu m 
				ON s.product_id = m.product_id
				WHERE s.order_date >= me.join_date)
SELECT customer_id, product_name, order_date, join_date
FROM purchase_after_membership
WHERE ranks = 1;
```


I put `customer_id`, `order_date`, `product_name` and `join_date` into a CTE(`purchase_after_membership`) and I ranked orders using the `order_date` for each customer. I had to join all three tables in the CTE so i could access different elements from the tables and put it into one table I filtered the query so that only orders that were made after `join_date` were displayed. Then I selected from the CTE all the orders that had a rank of 1.


customer_id	|product_name	|order_date	|join_date
---------|-------|------|-------
A|	curry	|07/01/2021	|07/01/2021
B|	sushi|	11/01/2021	|09/01/2021

After becoming a member, 
* Customer A first purchased `curry` 
* Customer B purchased `sushi`.

-----------


**QUESTION 7:** Which item was purchased just before the customer became a member?

```sql
WITH purchase_before_membership AS (
				SELECT s.customer_id,
					s.order_date,
					m.product_name,
					me.join_date,
					DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS ranks
				FROM sales s
				JOIN members me
				    ON s.customer_id = me.customer_id
				JOIN menu m 
				    ON s.product_id = m.product_id
				WHERE s.order_date < me.join_date)
SELECT customer_id, product_name, order_date, join_date
FROM purchase_before_membership
WHERE ranks = 1;
```

This query is similar to the query is question 6 except I filtered to display results where `order_date` was before `join_date`. In the windows functions, i ordered in descending order because I wanted the ranking to start from the date closest to `join_date`.

customer_id	|product_name	|order_date	|join_date
---------|-------|-------|---------
A	|sushi	|01/01/2021	|07/01/2021
A	|curry	|01/01/2021	|07/01/2021
B	|sushi	|04/01/2021	|09/01/2021

Just before becominga member,
* Customer A purchased `sushi` and `curry`
* Customer B ordered `sushi`

-------------


**QUESTION 8:** What is the total items and amount spent for each member before they became a member?

```sql
SELECT s.customer_id,
       COUNT(s.product_id) order_count,
       SUM(m.price) AS amount_spent
FROM sales s
JOIN menu m
   ON s.product_id = m.product_id
JOIN members me
   ON s.customer_id = me.customer_id
WHERE s.order_date < me.join_date
GROUP BY 1;
```

customer_id	|order_count	|amount_spent
------|-------|-------
B	|3	|40
A	|2	|25

Before becoming a member, 
* Customer B purchased 3 items and spent $40 
* Customer A ordered 2 items and spent $25

-------------

**QUESTION 9:** If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT customer_id, SUM(points) AS points
FROM (SELECT s.customer_id,
	     CASE WHEN s.product_id = 1 THEN m.price * 20
		  ELSE price*10
		  END AS points 
      FROM sales s
      JOIN menu m
	ON s.product_id = m.product_id) points
GROUP BY 1 
ORDER BY 2 DESC;
```

I used a subquery to  calculate the points for each customer according to the given criteria. In the main query, I displayed the data from the subquery.
I ordered the table with `points` in descending order to display the customer with the most points first.

customer_id	|points
-------|-----
B	|940
A	|860
C	|360


* Customer B, with the most points, had 940 points.
* Customer A  had 860 points 
* Customer C had 360 points.

--------------

**QUESTION 10:** In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


```sql
WITH points_cte AS (SELECT s.customer_id, s.order_date, s.product_id,
			CASE WHEN s.product_id = 1 THEN price * 20
     			     WHEN s.order_date >= mem.join_date AND s.order_date < (mem.join_date + interval '7 days') THEN price*20
			     ELSE price *10 END AS points
		    FROM sales s
		    JOIN members mem
			ON s.customer_id = mem.customer_id
		    JOIN menu m
			ON s.product_id = m.product_id )
SELECT customer_id, SUM(points) AS points
FROM points_cte
GROUP BY customer_id;
```

I used a CTE(`points_cte`) to calculate the points for each customer based on the given criteria. I used the `interval` data type to specify 7 days which is a week so i could give points according to the criteria. Then i queried the results from the CTE.

customer_id	|points
-----|-----
A	|1370
B	|940

* Customer A had the most points with 1370 points 
* Customer B had 940 points

---------------

**BONUS QUESTIONS**

**JOIN ALL THINGS**

```sql
SELECT s.customer_id, 
       s.order_date, 
       m.product_name,m.price, 
       CASE WHEN order_date >= join_date THEN 'Y' ELSE 'N' END AS member
FROM sales s
LEFT JOIN menu m
ON s.product_id = m.product_id
LEFT JOIN members mem
ON s.customer_id = mem.customer_id;
```

The resulting table is expected to look like a table which can be found at [case study.md](case-study.md)

customer_id	|order_date	|product_name	|price	|member
---------|---------|-------|-------|--------
A	|07/01/2021	|curry	|15	|Y
A	|11/01/2021	|ramen	|12	|Y
A	|11/01/2021	|ramen	|12	|Y
A	|10/01/2021	|ramen	|12	|Y
A	|01/01/2021	|sushi	|10	|N
A	|01/01/2021	|curry	|15	|N
B	|04/01/2021	|sushi	|10	|N
B	|11/01/2021	|sushi	|10	|Y
B	|01/01/2021	|curry	|15	|N
B	|02/01/2021	|curry	|15	|N
B	|16/01/2021	|ramen	|12	|Y
B	|01/02/2021	|ramen	|12	|Y
C	|01/01/2021	|ramen	|12	|N
C	|01/01/2021	|ramen	|12	|N
C	|07/01/2021	|ramen	|12	|N


-------------------

**RANK ALL THINGS**

```sql
WITH members AS (SELECT s.customer_id, 
			s.order_date, 
			m.product_name,m.price, 
			CASE WHEN order_date >= join_date THEN 'Y' ELSE 'N' END AS member
		FROM sales s
		LEFT JOIN menu m
			ON s.product_id = m.product_id
		LEFT JOIN members mem
			ON s.customer_id = mem.customer_id )
SELECT customer_id, 
       order_date, 
       product_name,
       price,
       member,
       CASE WHEN member = 'Y' THEN RANK() OVER (PARTITION BY customer_id,member ORDER BY order_date) ELSE NULL END AS ranking
FROM members;
```

The resulting table is expected to look like a table which can be found at [case study.md](case-study.md)

customer_id	|order_date	|product_name	|price	|member	|ranking
-------|--------|--------|--------|----------|----------
A	|01/01/2021	|sushi	|10	|N	|NULL
A	|01/01/2021	|curry	|15	|N	|NULL
A	|07/01/2021	|curry	|15	|Y	|1
A	|10/01/2021	|ramen	|12	|Y	|2
A	|11/01/2021	|ramen	|12	|Y	|3
A	|11/01/2021	|ramen	|12	|Y	|3
B	|01/01/2021	|curry	|15	|N	|NULL
B	|02/01/2021	|curry	|15	|N	|NULL
B	|04/01/2021	|sushi	|10	|N	|NULL
B	|11/01/2021	|sushi	|10	|Y	|1
B	|16/01/2021	|ramen	|12	|Y	|2
B	|01/02/2021	|ramen	|12	|Y	|3
C	|01/01/2021	|ramen	|12	|N	|NULL
C	|01/01/2021	|ramen	|12	|N	|NULL
C	|07/01/2021	|ramen	|12	|N	|NULL

