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

