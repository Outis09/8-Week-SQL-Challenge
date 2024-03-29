# Pizza Runner 🍕

The data in the `runner_orders` and `customer_orders` tables contain some `'null'` and `NaN` values therefore the data has to be clean so that we can query the data from those tables to solve Danny's problems. 
To clean the data, I used temporary tables to create a copy of the `runner_orders` and `customer_orders` tables and used `CASE WHEN` statements to clean the data into those temporary tables.

--------

Cleaning `runner_orders`
-------------
* The original `runner_orders` table can be accessed [here](case_study.md)

```sql
DROP TABLE IF EXISTS runner_orders_temp;
CREATE TEMP TABLE runner_orders_temp AS (
                      SELECT order_id,
                          runner_id,
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

I used `CREATE TEMP TABLE` syntax to create a temporary table called `runner_orders_temp`. The `order_id` and `runner_id` columns do not need cleaning so I selected them into the temporary table just as they were in the original table.
The `pickup_time` column had two `'null'` values so I converted those values into actual `NULL` values.

The `distance` column also has some `'null'` values so I converted those values to actual `NULL` values. However, the other values had `km` attached to the numbers so I had to remove to enable me convert the column to an `int` later.
I did this by identifying the values with `km` and using the `TRIM()` function to trim `km` from the numbers.

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


Even though the table looks clean now, there are still come changes to be made. Below is how the table looks on pgAdmin4

    
 ![Screenshot (3)](https://user-images.githubusercontent.com/104911707/179404674-4ddd9254-a2b2-4b29-b41f-d2bc1c9e8eb6.png)
 
 From the table we can tell that the data type for the `pickup_time`, `distance` and  `duration` columns is `character varying`. If the columns remain like this we cannot perform the appropriate date and numeric functions on these columns. Therefore I altered the table to alter the columns stated.
 
 ```SQL
 ALTER TABLE runner_orders_temp
	ALTER COLUMN pickup_time TYPE timestamp USING pickup_time::timestamp without time zone,
	ALTER COLUMN distance TYPE double precision USING distance::double precision,
	ALTER COLUMN duration TYPE int USING duration::int;
```

I used the query above to alter the temporary table. I casted the `pickup_time` column to  a `timestamp without time zone`. I casted the `distance` column to a `double precision` which is another name for `float` because it had some decimal values. Lastly, I casted the `duration` column to an `int`.

The resulting table:

![Screenshot (4)](https://user-images.githubusercontent.com/104911707/179405848-c7ec2e6e-87ae-4aa2-95d1-9ab6dc475721.png)


From the new table above, the columns have now been converted to the appropriate data types. We can now work on the columns. `runner_orders_temp` is the clean version of `runner_orders` therefore I will be using `runner_orders_temp` to solve Danny's problems. 

--------------

 Cleaning `customer_orders`
 ----------------
 * The original `customer_orders` table can be accessed [here](case_study.md)
 
 The `customer_orders` table also needs to be cleaned because it has some `'null'` values. Converting all the `'null'` values to actual `NULL` values will make the table clumsy so once again I converted them to `''`.
 
 ```sql
 CREATE TEMP TABLE customer_orders_temp AS(
 				SELECT order_id,
					       customer_id,
					       pizza_id,
					       CASE WHEN exclusions LIKE '%null%' THEN ''  ELSE exclusions END AS exclusions,
					       CASE WHEN extras LIKE '%null%' THEN ''
				                    WHEN extras IS NULL THEN ''
						    ELSE extras END AS extras,
					       order_time
				        FROM customer_orders
					);
```

I created a temporary table called `customer_orders_temp`.The `order_id`, `pizza_id`, `customer_id` and `order_time` columns were clean so I just had to clean the `exclusions` and `extras` columns. I used `CASE WHEN` statements to convert `'null'` and `NULL` values to `''`. 

Below is the resulting table.

| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        |            | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        |            |        | 2020-01-08 21:03:13 |
| 7        | 105         | 2        |            | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        |            |        | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        |            |        | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |

The above table is more suitable for writing queries than the original `customer_orders` therefore I used `customer_orders_temp` so solve Danny's problems.
