QUESTION 1
-----------

How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
WITH runner_registration AS (
                        SELECT DENSE_RANK() OVER (ORDER BY DATE_PART('week',registration_date ) DESC) as week_number,
                               registration_date
                        FROM runners)
SELECT week_number, COUNT(registration_date) AS num_of_registrations
FROM runner_registration
GROUP BY week_number
ORDER BY week_number;
```
I created a CTE named `runner_registration`. In this CTE I used the `dense_rank()` windows function to rank by the `week` in `registration_date`. I named the results `week_number` and selected `registration_date` as well. I queried `runner_registration` to get the `week_number` and the count of `registration_date`. This counts the number of registrations for each 1 week because I grouped by `week_number`.

**Results**

| week_number | num_of_registrations |
| ----------- | -------------------- |
| 1           | 2                    |
| 2           | 1                    |
| 3           | 1                    |

* In the first week startting 2021-01-01, 2 runners signed up
* In the second week 1 runner signed up
* In the third week 1 runner signed up

------------------------------------------

QUESTION 2
---------------

What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
WITH duration_cte AS (
                  SELECT r.runner_id, (r.pickup_time-c.order_time) as duration
                  FROM customer_orders_temp c
                  JOIN runner_orders_temp r
                  ON c.order_id = r.order_id
                  WHERE r.pickup_time IS NOT NULL
                  GROUP BY r.runner_id,r.pickup_time,c.order_time
                  ORDER BY r.runner_id)
SELECT runner_id, DATE_TRUNC('minute',AVG(duration)) AS avg_duration
FROM duration_cte
GROUP BY runner_id;
```

I created a CTE and named it `duration_cte`. In this CTE, I selected `runner_id`, and subtracted `order_time` from `pickup_time` as `duration`. This is the time it took the runner to arrive at Pizza Runner HQ to pick up the order. I filtered to only values where the `pickup_time` column was not NULL because the cancelled orders had NULL values there. I grouped by `runner_id` so that orders with more than pizzas will not be counted as different orders. I then queried `duration_cte` to get each `runner_id` and calculate the average of `duration`.I used `DATE_TRUNC()` to cut down the time to just the minutes so that the results will be more absolute.

**Results**

| runner_id | avg_duration    |
| --------- | --------------- |
| 1         | 00:14:00 |
| 2         | 00:20:00 |
| 3         | 00:10:00 |

On the average it took,
* `runner_id` 1 14 minutes to pick up the order
*  `runner_id` 2 20 minutes to pick up the order
*  `runner_id` 3 10 minutes to pick up the order

----------------------------

QUESTION 3
--------------

Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
SELECT num_of_pizza,AVG(interval) as avg_prep_time
FROM (SELECT c.order_id, 
             COUNT(pizza_id) AS num_of_pizza,
             DATE_TRUNC('minute',r.pickup_time-c.order_time) AS interval
	    FROM customer_orders_temp c
	    JOIN runner_orders_temp r
	    ON c.order_id = r.order_id
	    WHERE r.pickup_time IS NOT NULL
	    GROUP BY 1, c.order_time, r.pickup_time
	    ORDER BY 1) correlation
GROUP BY 1
ORDER BY 1;
```
To check for a relationship between the number of pizzas and the time taken to prepare, i had to derive a table containing number of pizzas in an order and how many minutes it took to prepare. I assumed the time between the `order_time` and `pickup_time` is the time it took to prepare each order.

I used a subquery.In the subquery (`correlation`), I selected `order_id`, counted the number of pizzas and used `DATE_TRUNC()` to get the minute from the time it took to prepare the order(`pickup_time` - `order_time`). I filtered to exclude rows where `pickup_time` was NULL because that meant the order was cancelled. I grouped by `order_id` so that the count of `pizza_id` was done for each `order_id`. From the subquery i selected `num_of_pizza` and calculated the average time it took to prepare each order. I grouped by `num_of_pizza` so that the average was calculated for each number of pizzas in an order. The results make this a litle clearer.

**Results**

| num_of_pizza | avg_prep_time   |
| ------------ | --------------- |
| 1            | 00:12:00 |
| 2            | 00:18:00 |
| 3            | 00:23:00 |

* In orders that contained only 1 pizza, it took an average of 12 minutes to prepare
* In orders that contained 2 pizzas, it took and average of 18 minutes to prepare
* In orders that contained 3 pizzas, it took an average of 23 minutes to prepare

From the results we can tell that the higher the number of pizzas in an order, the more time it took to prepare the order. This means that there is a positive relation ship between the number of pizzas and the time it took to prepare an order.

-----------------------------

QUESTION 4
----------

What was the average distance travelled for each customer?

```sql
SELECT c.customer_id, ROUND(avg(r.distance)) as avg_distance_km
FROM customer_orders_temp c
JOIN runner_orders_temp r
ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY c.customer_id
ORDER BY c.customer_id;
```
I joined the `customer_orders_temp` and `runner_orders_temp` tables for this query. I selected the `customer_id` and calculated the average of the distance which i then rounded to eliminate the many decimals. I grouped by `customer-id` so that the avearage distance was calculated for each customer. I filtered to include only orders which had been delivered.

**Results**
| customer_id | avg_distance_km |
| ----------- | --------------- |
| 101         | 20              |
| 102         | 17              |
| 103         | 23              |
| 104         | 10              |
| 105         | 25              |

The average distance travelled for;

* `Customer_id` 1 is 20km
* `Customer_id` 2 is 17km
* `Customer_id` 3 is 23km
* `Customer_id` 4 is 10km
* `Customer_id` 5 is 25km

----------------------------

QUESTION 5
--------------

What was the difference between the longest and shortest delivery times for all orders?

```sql
SELECT (MAX(duration) - MIN(duration)) as range
FROM runner_orders_temp
```

**Results**

| range |
| ----- |
| 30    |

* The difference between the longest and shortest delivery times for all orders is 30 minutes

-------------------------

QUESTION 6
---------------

What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql

