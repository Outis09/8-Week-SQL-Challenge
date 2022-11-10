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
