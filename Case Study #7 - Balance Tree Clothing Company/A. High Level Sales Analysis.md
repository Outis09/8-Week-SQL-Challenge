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

-------------------------------------

**Question 2:**
What is the total generated revenue for all products before discounts?
---------------

**Query:**

```sql
SELECT sum(price*qty) as tot_sales
  FROM sales;
```
**Results:**

|tot_sales|
|-------|
|1289453|


-----------------------------------------------

**Question 3:**
What was the total discount amount for all products?
-----

**Query:**

```sql
SELECT round(sum((price*qty)*(discount::numeric/100))) as tot_discount
  FROM sales;
```

**Results:**

|tot_discount|
|-------|
|156229|

--------------------------------------------
