**Question 1**
How many users are there?
-----

**Query:**

```sql
SELECT COUNT(DISTINCT(user_id)) as customers
FROM users;
```

I counted the distinct number of user IDs in the database.

**Results:**

|customers|
|---------|
|500|

* There are 500 users

--------------------------------------------------------

**Question 2**
How many cookies does each user have on average?
-----

**Query:**
```sql
--gets the number of cookies for each user
WITH cookie as (
      SELECT user_id,
             count(cookie_id) as cookies
        FROM users
    GROUP BY user_id
    ORDER BY user_id
              )
    
SELECT round(avg(cookies))as avg_cookies
FROM cookie;
```

I used a CTE,`cookie`, to get the number of cookies for each user. In the main query, i calculated the average number of cookies and rounded to the nearest whole number

**Results:**

| avg_cookies        |
| ------------------ |
| 4 |

* On the average, each user has 4 cookies
----------------------------------------------------------

**Question 3**
What is the unique number of visits by all users per month?
-----

**Query:**

```sql
  SELECT to_char(event_time,'Month') as month,count(distinct(visit_id))
    FROM events
GROUP BY 1,date_trunc('month',event_time)
ORDER BY date_trunc('month',event_time);
```
I used `to_char` to extract month names from the `event_time` column. Then I counted the number of distinct visits and grouped by month.

**Results:**

| month     | count |
| --------- | ----- |
| January   | 876   |
| February  | 1488  |
| March     | 916   |
| April     | 248   |
| May       | 36    |


--------------------


**Question 4**
What is the number of events for each event type?
-----

**Query:**

```sql
  SELECT event_name, 
         count(visit_id) as num_of_visits
    FROM events e
    JOIN event_identifier ei
      ON e.event_type = ei.event_type
GROUP BY event_name;
```
I joined the `events` table to the `event_identifier`. I selected `event_name` and counted the number of visit IDs. I grouped the count by event name.

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
I counted the number of unique visit IDs that had a purchase and divided by the total number of unique visit IDs in the database. I multiplied by 100 and rounded to two decimal places

**Results:**

| round |
| ----- |
| 49.86 |

* 49.86 of visits had a purchase event
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
I used a CTE,`arrays`, to get the pages and events for each visit in an array. In the main query, i filtered for visit IDs that had been to the checkout page but not purchased. That is, those with 12 in the page array but no 3 in the event array. I counted the number of visit IDs, divided by the unique number of visits in the database and multiplied by 100. I also divided the count of visits which met the condition by the number of visits that had viewed the checkout page and multiplied by 100.
**Results:**

| num_of_visits | prcnt_of_all_vsts | prcnt_of_all_chckots |
| ------------- | ----------------- | -------------------- |
| 326           | 9.15              | 15.50                |

* There were 326 visits which viewed the checkout page but did not purchase anything
* 9.15 percent of all visits viewed the checkout page but did not purchase anything
* 15.50 percent of all visits that viewed the checkout page did not purchase anything
-----------------------------------------------

**Question 7**
What are the top 3 pages by number of views?
--------

**Query :**

```sql
  SELECT page_name,
         count(event_type) as num_of_views
    FROM events e
    JOIN page_hierarchy ph
      ON e.page_id = ph.page_id
   WHERE event_type = 1
GROUP BY page_name
ORDER BY num_of_views DESC
   LIMIT 3;
```
I joined the `page_hierarchy` table to the `events` table. I selected the page name and counted the number of events then grouped by page name. I ordered in descending order of the count. I limited the results to 3 so that only the top 3 were displayed.

**Results:**

| page_name    | num_of_views |
| ------------ | ------------ |
| All Products | 3174         |
| Checkout     | 2103         |
| Home Page    | 1782         |

--------------------------------------------

**Question 8:**
What is the number of views and cart adds for each product category?
-----

```sql
  SELECT product_category,
         sum(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) as num_of_views,
         sum(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) as num_of_cart_adds
    FROM page_hierarchy ph
    JOIN events e
      ON ph.page_id = e.page_id
   WHERE product_category IS NOT NULL
GROUP BY product_category
ORDER BY num_of_views DESC;
```

**Results:**
| product_category | num_of_views | num_of_cart_adds |
| ---------------- | ------------ | ---------------- |
| Shellfish        | 6204         | 3792             |
| Fish             | 4633         | 2789             |
| Luxury           | 3032         | 1870             |

-------------------------------------------

**Question 9:**
What are the top 3 products by purchases?
--------

**Query:**

```sql
WITH sequence as (
	SELECT visit_id,
	       page_id,
	       page_name,
	       event_type,
	       event_name
	  FROM events
	  JOIN page_hierarchy
	 USING (page_id)
	  JOIN event_identifier
	 USING (event_type)
	 WHERE visit_id IN (SELECT visit_id FROM events WHERE event_type = 3)
      GROUP BY 1,2,3,4,5,event_time
      ORDER BY visit_id)

  SELECT page_name,
         count(event_name) as purchases
    FROM sequence 
   WHERE event_type = 2
GROUP BY page_name
ORDER BY purchases DESC
   LIMIT 3;
```

**Results:**

| page_name | purchases |
| --------- | --------- |
| Lobster   | 754       |
| Oyster    | 726       |
| Crab      | 719       |

----------------------------------------
