**Question 1:**
How many unique nodes are there on the Data Bank system?
-----

```sql
SELECT count(distinct(node_id)) as num_of_nodes
  FROM customer_nodes;
```

Results:

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

Results: 
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

Results:

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

Results:

| average_reallocation_days |
| ------------------------- |
| 15.6      |

* On the average, it takes approximately 16 days for customers to be reallocated to a different node

----------------------------------------

**Question 5:**
What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
-----

```sql
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
							
SELECT percentile_cont(0.5) within group(order by reallocation_days) as median,
       percentile_cont(0.8) within group(order by reallocation_days) as eightieth_percentile,
	     percentile_cont(0.95) within group(order by reallocation_days) as ninetyfifith_percentile
  FROM reallocation_cte;
```

Results:
| median | eightieth_percentile | ninetyfifith_percentile |
| ------ | ---------------------- | ----------------------- |
| 16     | 24                     | 29                      |

* The median allocation day is 16 days
* The 80th percentile is 24 days
* The 95th percentile is 29 days

