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

--------------------------------------------------------

**Question 2**
How many cookies does each user have on average?
-----

**Query:**
```sql
WITH cookie as (
      SELECT user_id,count(cookie_id) as cookies
        FROM users
    GROUP BY user_id
    ORDER BY user_id
              )
    
SELECT round(avg(cookies))as avg_cookies
FROM cookie;
```

**Results:**

| avg_cookies        |
| ------------------ |
| 4 |

----------------------------------------------------------




