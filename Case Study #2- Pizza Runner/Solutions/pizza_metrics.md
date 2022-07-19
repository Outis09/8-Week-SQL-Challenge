QUESTION 1:
-------

How many pizzas were ordered?

```sql
SELECT COUNT(order_id) AS num_of_orders
FROM customer_orders_temp;
```

**Results**
| num_of_orders |
| ------------- |
| 14            |

14 pizzas were ordered in all

----------------

QUESTION 2
----------

How many unique customer orders were made?

```sql
SELECT COUNT(DISTINCT(order_id)) AS unique_order_count
FROM customer_orders_temp;
```

To get the unique orders, I used `DISTINCT` to ensure that only unique orders were counted.

**Results**
| unique_order_count |
| ------------------ |
| 10                 |

10 unique customer orders were made

------------

QUESTION 3
--------------

How many successful orders were delivered by each runner?

```sql
SELECT runner_id, COUNT(order_id) AS num_of_deliveries
FROM runner_orders_temp
WHERE distance IS NOT NULL
GROUP BY runner_id
ORDER BY 2 DESC;
```
I selected `runner_id` and counted the `order_id` then i filtered to only include `order_id` where `distance` was not `NULL` because that would mean the order was not 
delivered. I then grouped by runner_id` so that the count would be done for each runner. Then finally i ordered by the count in descending order so that the runner 
with the most successful order deliveries would come first.

The same results can be achieved by replacing `distance` in the query with `duration` or `pickup_time`.

**Results**

| runner_id | num_of_deliveries |
| --------- | ----------------- |
| 1         | 4                 |
| 2         | 3                 |
| 3         | 1                 |

* Runner 1 delivered 4 successful orders
* Runner 2 delivered 3 successful orders
* Runner 3 delivered 1 successful order

-------------

QUESTION 4
--------

How many of each type of pizza was delivered?

```sql
SELECT p.pizza_name, COUNT(c.pizza_id) AS delivery_count
FROM customer_orders_temp c
JOIN runner_orders_temp r
ON c.order_id = r.order_id
JOIN pizza_names p
ON c.pizza_id = p.pizza_id
WHERE r.distance IS NOT NULL
GROUP BY p.pizza_name;
```

I joined 3 tables in this query. I used `customer_orders_temp` so I could match the `pizza_id` to the `pizza_name` in the `pizza_name` tables. I joined the 
`runner_orders_temp` table so i could filter to include only successful deliveries.

I selected the `pizza-name` and counted the `pizza_id`. I thenfiltered to inlcude only successful deliveries in the count by specifying that `distance` is not `NULL¬.
Then I grouped by `pizza_name` to make sure that the count was done for each pizza.

**Results**
| pizza_name | delivery_count |
| ---------- | -------------- |
| Meatlovers | 9              |
| Vegetarian | 3              |

* 9 Meatlovers pizzas were delivered
* 3 Vegetarian pizzas were delivered

----------------

QUESTION 5
----------

How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT c.customer_id, p.pizza_name, COUNT(c.order_id) AS order_count
FROM customer_orders_temp c
JOIN pizza_names p
ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id,p.pizza_name
ORDER BY c.customer_id;
```

I used the `customer_orders_temp` and `pizza_names` tables for this query. 

I selected the `customer_id` and the `pizza_name` then I counted `order_id`. I grouped by `customer_id` and `pizza_name` to make the count for each customer and for 
each customer, each pizza name. Not adding `pizza_name` will result in the count of orders just for each customer and not for each pizza name for each customer.

**Results**
| customer_id | pizza_name | order_count |
| ----------- | ---------- | ----------- |
| 101         | Meatlovers | 2           |
| 101         | Vegetarian | 1           |
| 102         | Meatlovers | 2           |
| 102         | Vegetarian | 1           |
| 103         | Meatlovers | 3           |
| 103         | Vegetarian | 1           |
| 104         | Meatlovers | 3           |
| 105         | Vegetarian | 1           |

* Customer 101 ordered 2 Meatlovers pizza and 1 Vegetarian pizza
* Customer 102 ordered 2 Meatlovers pizza and 1 Vegetarian pizza
* Customer 103 ordered 3 Meatlovers pizza and 1 Vegetarian pizza
* Customer 104 ordered 3 Meatlovers pizza 
* Customer 105 ordered 1 Vegetarian pizza

-------------

QUESTION 6
---------

What was the maximum number of pizzas delivered in a single order?

```sql
SELECT c.order_id, COUNT(c.pizza_id) max_pizza_delivered
FROM customer_orders_temp c
JOIN runner_orders_temp r
ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

To get the maximum number of pizza delivered in an order, I joined the `customer_orders_temp` table and `runner_orders-temp` table. I selected the `ordered_id` and counted the number of pizza to be grouped by the `order_id`. This would count the number of pizzas in each order. I then filtered to only include orders where `distance` was not NULL which signifies that the pizza had been delivered. I ordered in descending order of the count so that the highest appeared first then i limited the results to one so that only the highest was displayed.

**Results**

| order_id | max_pizza_delivered |
| -------- | ------------------- |
| 4        | 3                   |

* The maximum number of pizzas delivered in a single order is 3 and the order ID is 4.

------------

QUESTION 7
----------

For each customer, how many delivered pizzas had at least 1 change and how many had no changes?


```sql
SELECT c.customer_id, 
	   SUM(CASE WHEN c.exclusions = '' 
		   			AND c.extras = ''
		   		    THEN 1 ELSE 0 END) AS unchanged,
	   SUM(CASE WHEN c.exclusions != ''
		   			 OR c.extras != '' 
		   			 THEN 1 ELSE 0 END )AS changed
FROM customer_orders_temp c
JOIN runner_orders_temp r
ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY c.customer_id
ORDER BY c.customer_id;
```

I joined the `customer_orders_temp` table to the `runner_orders_temp` so i could filter by orders which had been delivered. I selected the `customer_id` and used `CASE WHEN` statements and `SUM` to count the number of changes or no changes for each customer. In the first `CASE WHEN` statement, if both exclusions and extras were empty then 1 was assigned and if not 0. Therefore 1 in the resulting column signifies that there was no change. In the second `CASE WHEN` statement, if either exclusions or extras were not empty then 1 was assigned otherwise 0 was assigned. Therefore in the resulting column, 1 signified at least one change in the order. I then summed the resuls from both `CASE WHEN` statements and grouped by each customer.

**Results**

| customer_id | unchanged | changed |
| ----------- | --------- | ------- |
| 101         | 2         | 0       |
| 102         | 3         | 0       |
| 103         | 0         | 3       |
| 104         | 1         | 2       |
| 105         | 0         | 1       |

* Customer ID 101 had 2 pizzas delivered with no changes
* Customer ID 102 had 3 pizzas delivered with no changes
* Customer ID 103 had 3 pizzas deivered with at least one change in each pizza
* Customer ID 104 had 1 pizza delivered with no chnages and 2 with at least one change
* Customer ID 105 had 1 pizza delivered with at least one change.

------------------

QUESTION 8
--------------

How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT count(c.order_id) as delivered_orders_w_exclusions_n_extras
FROM customer_orders_temp c
JOIN runner_orders_temp r
ON c.order_id = r.order_id
WHERE c.exclusions != '' AND c.extras != '' AND r.distance IS NOT NULL;
```

For this, I had to join the `runner_orders_temp` table to the `customer_orders_temp` table so I could filter for only delivered orders. I counted orders in the `customer_orders_temp` table and filtered for orders where `extras` was not equal to `''` AND `exclusions` was also not equal to `''` AND `distance` was not NULL. 

**Results**

| delivered_orders_w_exclusions_n_extras |
| -------------------------------------- |
| 1                                      |

* Only 1 order with `extras` and `exclusions` was delivered.

--------------------------

QUESTION 9
----------

What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT DATE_PART('hour', order_time) AS hour_of_order, COUNT(order_id) AS num_of_orders
FROM customer_orders_temp 
GROUP BY 1
ORDER BY 1;
```

I used the `DATE_PART` function to select the hour from the `order_time` then i counted the number of orders. I grouped by the hour to ensure that the count was doen for each hour in the `order_time`.

**Results**

| hour_of_order | num_of_orders |
| ------------- | ------------- |
| 11            | 1             |
| 13            | 3             |
| 18            | 3             |
| 19            | 1             |
| 21            | 3             |
| 23            | 3             |


* 1 order was placed within the hours of 11:00
* 3 orders were placed within the hours of 13:00
* 3 orders were placed within the hours of 18:00
* 1 order was placed within the hours of 19:00
* 3 orders were placed within the hours of 21:00
* 3 orders were placed within the hours of 23:00

------------------

QUESTION 10
------------

What was the volume of orders for each day of the week?

```sql
SELECT TO_CHAR(order_time,'Day') AS day, count(order_id) AS num_of_orders
FROM customer_orders_temp
GROUP BY 1
ORDER BY 2 DESC;
```

I wanted to extract the names of the days of the week so I used the `TO_CHAR` function to extract the names of the days from `order_time`. I then counted the order IDs and grouped by the days so that for each day, the number of orders that took place was counted. I ordered by the count of orders in descending order to show the highest count first. 

**Results**

| day       | num_of_orders |
| --------- | ------------- |
| Saturday  | 5             |
| Wednesday | 5             |
| Thursday  | 3             |
| Friday    | 1             |

* 5 orders were placed on Saturday
* 5 orders were placed on Wednesday
* 3 orders were placed on Thursday
* 1 order was placed on Friday

---------------------------




