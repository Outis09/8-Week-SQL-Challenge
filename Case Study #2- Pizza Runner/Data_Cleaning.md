# Pizza Runner 🍕

The data in the `runner_orders` and `customer_orders` tables contain some `'null'` and `NaN` values therefore the data has to be clean so that we can query the data from those tables to solve Danny's problems. 
To clean the data, i used temporary tables to create a copy of the `runner_orders` and `customer_orders` tables and used `CASE WHEN` statements to clean the data into those temporary tables.

Cleaning `runner_orders`
-------------

```sql
DROP TABLE IF EXISTS runner_orders_temp;
CREATE TEMP TABLE runner_orders_temp AS (
                      SELECT order_id,
                          runner_id,
                         -- converting `'null'` values into actual `NULL` values
                         CASE WHEN pickup_time IS NULL OR pickup_time LIKE '%null%' THEN NULL ELSE pickup_time END AS pickup_time,
                          CASE WHEN distance IS NULL OR distance LIKE '%null%' THEN NULL 
	                             WHEN distance LIKE '%km%' THEN trim('km' from distance) ELSE distance END AS distance,
                          CASE WHEN duration IS NULL OR duration LIKE '%null%' THEN NULL
	                             WHEN duration LIKE '%mins%' THEN TRIM('mins' FROM duration)
	                             WHEN duration LIKE '%minutes%' THEN TRIM('minutes' FROM duration)
	                             WHEN duration LIKE '%minute%' THEN TRIM( 'minute' FROM duration)
	                             ELSE duration END AS duration,
                          CASE WHEN cancellation IS NULL OR cancellation LIKE '%null%' THEN '' ELSE cancellation END AS cancellation
                      FROM runner_orders
                      );
```

I used `CREATE TEMP TABLE` syntax to create a temporary table called `runner_orders_temp`. The `order_id` and `runner_id` do not cleaning so I selected them into the temporary table just as they were in the original table.
The `pickup_time` column had two `'null'` values so I converted those values into actual `NULL` values.

The `distance` column also has some `'null'` values so I converted those values to actual `NULL` values. However, the other values had `km` attached to the numbers so I had to remove to enable me convert the column to an `int` later.
I did this by identifying the values with `km` and using the `TRIM()` function to trim `km` from the nnumbers.

The duration column also has `'null'` values so I used `CASE WHEN` statement to convert them to actual `NULL` values. The other values have `'mins'`, `'minutes'` and `'minute'`
attached to their numeric values so I had to remove them to enable me convert the column to an `int` later. I used the `WHEN` statement to identify the values with  `'mins'`, `'minutes'` and `'minute'` then I used the `TRIM()` function to remove those words from the 
numbers.

The `cancellation` column has `'null'` and `Nan` values. PostgreSQL recognizes `NaN` values as `NULL` values. Converting the `'null'` values to `NULL` values will make the table clumsy
so instead I converted them to `''`. Unlike most RDMS, PostgreSQL does not recognize `''` as NULL.

The resulting table is below.

`runner_order_temp`
--------------

| order_id | runner_id | pickup_time              | distance | duration | cancellation            |
| -------- | --------- | ------------------------ | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         | NULL                | NULL     |NULL      | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         | NULL                |NULL      |NULL      | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |


Even though this 
