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

**Question 3**
What is the unique number of visits by all users per month?
-----

**Query:**

```sql
  SELECT to_char(event_time,'month') as month,count(distinct(visit_id))
    FROM events
GROUP BY 1,date_trunc('month',event_time)
ORDER BY date_trunc('month',event_time);
```

**Results:**

| month     | count |
| --------- | ----- |
| january   | 876   |
| february  | 1488  |
| march     | 916   |
| april     | 248   |
| may       | 36    |

--------------------


**Question 4**
What is the number of events for each event type?
-----

**Query:**

```sql
  SELECT event_name, count(visit_id) as num_of_visits
    FROM events e
    JOIN event_identifier ei
      ON e.event_type = ei.event_type
GROUP BY event_name;
```

**Results:**

| event_name    | num_of_visits |
| ------------- | ------------- |
| Purchase      | 1777          |
| Ad Impression | 876           |
| Add to Cart   | 8451          |
| Page View     | 20928         |
| Ad Click      | 702           |

-----------------------------------------------------

**Question 5**
What is the percentage of visits which have a purchase event?
-----

**Query:**

```sql
SELECT round((count(distinct(visit_id))::numeric /
             (select count(distinct(visit_id)) from events)::numeric)*100,2)
  FROM events
 WHERE event_type = 3;
```

**Results:**

| round |
| ----- |
| 49.86 |

--------------------------------------------------------

**Question 6:**
What is the percentage of visits which view the checkout page but do not have a purchase event?
------

**Query:**

```sql
--gets an array of page id and event type for each visit id
with arrays as (
           SELECT visit_id,
	            array_agg(page_id) as pages,
	            array_agg(event_type) as events
             FROM events
         GROUP BY visit_id
                )
         
SELECT count(visit_id) as num_of_visits,
       round(100*count(visit_id)/(select count(distinct(visit_id)) FROM events)::numeric,2) as prcnt_of_all_vsts,
       round(100*count(visit_id)/(select count(visit_id) from events where page_id=12)::numeric,2) as prcnt_of_all_chckots
FROM arrays
WHERE 12 = ANY(pages::int[]) and 3 != ALL(events::int[]);
```

**Results:**

| num_of_visits | prcnt_of_all_vsts | prcnt_of_all_chckots |
| ------------- | ----------------- | -------------------- |
| 326           | 9.15              | 15.50                |

-----------------------------------------------
