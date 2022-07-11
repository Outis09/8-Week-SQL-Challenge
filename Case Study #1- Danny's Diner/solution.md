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

Customer A spent $76, customer B spent spent $74, customer C spent $36
