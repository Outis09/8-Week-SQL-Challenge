**Question 1**
How many users are there?
-----

**Query:**

```sql
SELECT COUNT(DISTINCT(user_id)) as customers
FROM users;
```

**Results:**

|customers|
|---------|
|500|


