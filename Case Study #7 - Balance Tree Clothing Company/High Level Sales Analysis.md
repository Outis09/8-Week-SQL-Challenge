**Question 1:**
What was the total quantity sold for all products?
------

**Query:**

```sql
SELECT sum(qty) as tot_qty
FROM sales;
```

**Results:**

|tot_qty|
|-------|
|45216|
