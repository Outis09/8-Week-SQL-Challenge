```sql
SET search_path TO dannys_diner;
```

**QUESTION 1:** What is the total amount each customer spent at the restaurant?


```sql
SELECT s.customer_id, SUM(m.price) as total_spent
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

Customer A spent $76, customer B spent spent $74, customer C spent $36.

Customer A spent the most money.

**QUESTION 2:** How many days has each customer visited the restaurant?

```sql
SELECT customer_id, COUNT(DISTINCT(order_date)) AS days_visited
FROM sales 
GROUP BY 1;
```

customer_id|	days_visited
---------|--------
A	|4
B	|6
C	|2

Customer A visited the restaurant on 4 different days, customer B visited on 6 different days, and customer C visited on 2 different days.



