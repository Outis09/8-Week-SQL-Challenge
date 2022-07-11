```sql
SET search_path TO dannys_diner;
```

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

Customer A spent $76, customer B spent spent $74, customer C spent $36. I ordered by the total amount spent to show the highest spender first.

Customer A spent the most money.

**QUESTION 2:** How many days has each customer visited the restaurant?

```sql
SELECT customer_id, 
       COUNT(DISTINCT(order_date)) AS days_visited
FROM sales 
GROUP BY 1;
```

customer_id|	days_visited
---------|--------
A	|4
B	|6
C	|2

Customer A visited the restaurant on 4 different days, customer B visited on 6 different days, and customer C visited on 2 different days.

**QUESTION 3:** What was the first item from the menu purchased by each customer?

```sql
WITH t1 AS (SELECT customer_id, 
					product_name,
					DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS ranks
		   FROM sales
		   JOIN menu
		   USING (product_id) )
SELECT customer_id, product_name
FROM t1
WHERE ranks = 1
GROUP BY 1,2;
```
I created a CTE and used the dense rank windows function to rank all orders for each customer by partititioning by `customer_id` and ordering by `order_date`. The earliest orders are ranked as 1 and the rest follow. Then I join the menu table to get the name of the products in customer's first purchases. For customer A, two items were purchased on the first day and there is no time of order therefore this query returns two items as first purchase for customer A.

customer_id|	product_name
--------|-------
A	|curry
A	|sushi
B	|curry
C	|ramen

However, it is possible to limit customer A's first order to one item by adding s.product_id to the `ORDER BY` in the windows function. Even though the following query returns one item, it is not certain if that is actually the first order because time is what can show exactly which item was purchased first and the data lacks time.

```sql
WITH t1 AS (SELECT customer_id, 
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






