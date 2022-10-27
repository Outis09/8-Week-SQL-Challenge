**Question 1:**
How many unique nodes are there on the Data Bank system?
-----

```sql
SELECT count(distinct(node_id)) as num_of_nodes
  FROM customer_nodes;
```
I counted the distinct number of nodes in the entire Data Bank system.

**Results:**

| num_of_nodes |
| ----- |
| 5     |

* There are 5 unique nodes in the Data Bank system

-------------------------------------------

**Question 2:**
What is the number of nodes per region?
-----

```sql
  SELECT region_name as region ,count(node_id) as number_of_nodes
    FROM customer_nodes as cn
    JOIN regions as r
      ON cn.region_id = r.region_id
GROUP BY 1;
```
I joined the `customer_nodes` table to the `regions` table using `region_id`. I selected the region name and counted the number of nodes then grouped by the regions.

**Results:**

| region    | number_of_nodes |
| --------- | --------------- |
| America   | 735             |
| Australia | 770             |
| Africa    | 714             |
| Asia      | 665             |
| Europe    | 616             |

There are:
* 735 nodes in America
* 770 nodes in Australia
* 714 nodes in Africa
* 665 nodes in Asia
* 616 nodes in Europe

-----------------------------

**Question 3:**
How many customers are allocated to each region?
-----

```sql
  SELECT region_name as region, count(distinct(customer_id)) as num_of_customers
    FROM customer_nodes as cn
    JOIN regions as r
      ON cn.region_id = r.region_id
GROUP BY 1;
```

I counted the number of customers and grouped by the regions.

**Results:**

| region    | num_of_customers |
| --------- | ---------------- |
| Africa    | 102              |
| America   | 105              |
| Asia      | 95               |
| Australia | 110              |
| Europe    | 88               |

There are:
* 102 customers in Africa
* 105 customers in America
* 95 customers in Asia
* 110 customers in Australia
* 88 customers in Europe

---------------------------------

**Question 4:**
How many days on average are customers reallocated to a different node?
-----

```sql
--reallocation days
WITH reallocation_cte AS (
        SELECT customer_id,
               node_id,
               start_date,
               CASE WHEN 
                    LAG(node_id) OVER(PARTITION BY customer_id ORDER BY start_date) != node_id
                    THEN start_date - LAG(start_date) OVER(PARTITION BY customer_id ORDER BY start_date) 
                    ELSE null
                    END AS reallocation_days
         FROM customer_nodes
                        )

SELECT round(avg(reallocation_days),1) as average_reallocation_days
  FROM reallocation_cte;
```
I created a CTE (`reallocation_cte`) where I selected customer ID, node ID and start date. Using a case when statement, I specified a condition that for each row for each customer, if node ID was not equal to the previous node ID then the start date of the previous node should be subtracted from the start date of the current node so that it resulted in the number of days it took for the reallocation to happen. If the node ID on one row was equal to the node on the previous row then it should return null. I named the column `reallocation_days`. I then queried the CTE for the average of the reallocation days and rounded to 1 decimal point.

**Results:**

| average_reallocation_days |
| ------------------------- |
| 15.6      |

* On the average, it takes approximately 16 days for customers to be reallocated to a different node

----------------------------------------

**Question 5:**
What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
-----

```sql
--reallocation days
WITH reallocation_cte AS (
        SELECT customer_id,
	       region_name as region,
               node_id,
               start_date,
               CASE WHEN 
                    LAG(node_id) OVER(PARTITION BY customer_id ORDER BY start_date) != node_id
                    THEN start_date - LAG(start_date) OVER(PARTITION BY customer_id ORDER BY start_date) 
                    ELSE null
                    END AS reallocation_days
         FROM customer_nodes cn
	 JOIN regions r
	   ON cn.region_id = r.region_id
                        )
							
  SELECT region,
         percentile_cont(0.5) within group(order by reallocation_days) as median,
         percentile_cont(0.8) within group(order by reallocation_days) as eightieth_percentile,
	 percentile_cont(0.95) within group(order by reallocation_days) as ninetyfifith_percentile
    FROM reallocation_cte
GROUP BY regions;
```
I used the same CTE I used in the previous business question. I then queried the CTE for the median, 80th percentile and 95th percentile using the `percentile_cont` function.

**Results:**
| region    | median | eightyfifth_percentile | ninetyfifith_percentile |
| --------- | ------ | ---------------------- | ----------------------- |
| Africa    | 16     | 25                     | 29                      |
| America   | 16     | 24                     | 29                      |
| Asia      | 15     | 24                     | 29                      |
| Australia | 15.5   | 24                     | 29                      |
| Europe    | 16     | 26                     | 29                      |



