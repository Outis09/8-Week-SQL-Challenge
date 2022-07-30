**QUESTION 1**

If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if 
there are no delivery fees?

```sql
SELECT SUM(CASE WHEN oct.pizza_id= 1 THEN order_count*12
						    WHEN oct.pizza_id = 2 THEN order_count*10
						    END) sales_in_dollars
FROM (SELECT c.pizza_id, COUNT(c.order_id) as order_count
      FROM customer_orders_temp c
      JOIN runner_orders_temp r
      ON c.order_id = r.order_id
      WHERE r.cancellation = ''
      GROUP BY 1) sq
```

I used a subquery(`sq`) to find the number of pizzas that were delivered for each pizza type. Then I used `CASE WHEN` to assign the prices as stated in the question.
So if the pizza Id was 1 then the number of orders for that pizza was multiplied by 12 to get the sales from that pizza. If the pizza ID was 2 then the 
number of orders for that pizza was multiplied by 10. Then i summed the results.

Results:

| sales_in_dollars |
| ---------------- |
| 138              |

* Pizza Runner made a total of $138

---------------------------------

**QUESTION 2**

What if there was an additional $1 charge for any pizza extras?
* Add cheese is $1 extra

```sql
WITH standard_sales AS (
          SELECT SUM(CASE WHEN oct.pizza_id= 1 THEN order_count*12
						              WHEN oct.pizza_id = 2 THEN order_count*10
						              END) sales_in_dollars
          FROM (SELECT c.pizza_id, COUNT(c.order_id) as order_count
                FROM customer_orders_temp c
                JOIN runner_orders_temp r
                ON c.order_id = r.order_id
                WHERE r.cancellation = ''
                GROUP BY 1) sq1
         JOIN pizza_names p
         ON sq1.pizza_id = p.pizza_id),
extras AS (
          SELECT COUNT(extras) *1 AS extras_sales_in_dollars
          FROM (SELECT c.order_id, c.pizza_id ,(TRIM(' ' FROM UNNEST(string_to_array(extras, ',') )) :: int) AS extras
                FROM customer_orders_temp c
                JOIN runner_orders_temp r
                ON c.order_id = r.order_id 
                WHERE r.cancellation = '') sq2)

SELECT sales_in_dollars + extras_sales_in_dollars AS total_sales_in_dollars
FROM standard_sales, extras;

I created two CTEs. In the first CTE(`standard_sales`) I calculated for the total amount os sales minus the extras. In the second CTE(`extras`) I converted the 
strings in the `extras` column to an array and unnested it. I then counted the number of extras and multiplied by 1($1). Then i added the total sales minus extras and 
exclusions(`sales_in_dollars`) to the total cost of extras(`extras_sales_in_dollars).

Results:
| total_sales_in_dollars |
| ---------------------- |
| 142                    |

* Pizza Runner made $142 dollars for charging extra $1 on extras .

---------------------------------

**QUESTION 3**

The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table 
for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

```sql
DROP TABLE IF EXISTS rating_system;
CREATE TABLE rating_system(
order_id INT NOT NULL,
customer_id INT NOT NULL,
runner_id INT NOT NULL,
rating INT NOT NULL);

INSERT INTO rating_system(order_id, customer_id,runner_id,rating)
VALUES (1, 101, 1, 3),
		(2, 101, 1, 4 ),
		(3, 102, 1, 5),
		(4, 103, 2, 5),
		(5, 104, 3, 5),
		(7, 105, 2, 2),
		(8, 102, 2, 4),
		(10, 104, 1, 5);
		
SELECT *
FROM rating_system;
```

I created a table with 4 columns. The columns included the `rating` column where the customer rated the runners.

Results:

| order_id | customer_id | runner_id | rating |
| -------- | ----------- | --------- | ------ |
| 1        | 101         | 1         | 3      |
| 2        | 101         | 1         | 4      |
| 3        | 102         | 1         | 5      |
| 4        | 103         | 2         | 5      |
| 5        | 104         | 3         | 5      |
| 7        | 105         | 2         | 2      |
| 8        | 102         | 2         | 4      |
| 10       | 104         | 1         | 5      |


------------------------------

**QUESTION 4**

Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
* customer_id
* order_id
* runner_id
* rating
* order_time
* pickup_time
* Time between order and pickup
* Delivery duration
* Average speed
* Total number of pizzas

```sql
SELECT c.customer_id, r.order_id,
       r.runner_id, r.rating,
       c.order_time,rot.pickup_time,
       sq.duration AS order_prep_dur,rot.duration,
       ROUND((rot.distance/rot.duration)*60) AS avg_speed_KmH, COUNT(c.pizza_id)
FROM rating_system r
JOIN customer_orders_temp c
ON r.order_id = c.order_id
JOIN runner_orders_temp rot
ON r.order_id = rot.order_id
JOIN (SELECT c.order_id, c.customer_id,r.runner_id, DATE_PART('minute',DATE_TRUNC('minute',r.pickup_time-c.order_time)) as duration
      FROM customer_orders_temp c
      JOIN runner_orders_temp r
      ON c.order_id = r.order_id
      WHERE r.pickup_time IS NOT NULL
      GROUP BY r.runner_id,r.pickup_time,c.order_time,c.order_id,c.customer_id
      ORDER BY r.runner_id) sq
ON r.order_id = sq.order_id
GROUP BY 1,r.order_id,3,4,5,6,rot.distance,rot.duration,sq.duration
ORDER BY 1;
```

I used a subquery(`sq`) to calculate the difference between `order_time` and `pickup_time` and later aliased it as `order_prep_dur`. I selected the remaining
columns from the their respective tables by joining  the tables and selecting the columns I needed.

Results:

| customer_id | order_id | runner_id | rating | order_time               | pickup_time              | order_prep_dur | duration | avg_speed_kmh | count |
| ----------- | -------- | --------- | ------ | ------------------------ | ------------------------ | -------------- | -------- | ------------- | ----- |
| 101         | 1        | 1         | 3      | 2020-01-01T18:05:02 | 2020-01-01T18:15:34 | 10             | 32       | 38            | 1     |
| 101         | 2        | 1         | 4      | 2020-01-01T19:00:52 | 2020-01-01T19:10:54 | 10             | 27       | 44            | 1     |
| 102         | 3        | 1         | 5      | 2020-01-02T23:51:23 | 2020-01-03T00:12:37 | 21             | 20       | 40            | 2     |
| 102         | 8        | 2         | 4      | 2020-01-09T23:54:33 | 2020-01-10T00:15:02 | 20             | 15       | 94            | 1     |
| 103         | 4        | 2         | 5      | 2020-01-04T13:23:46 | 2020-01-04T13:53:03 | 29             | 40       | 35            | 3     |
| 104         | 5        | 3         | 5      | 2020-01-08T21:00:29 | 2020-01-08T21:10:57 | 10             | 15       | 40            | 1     |
| 104         | 10       | 1         | 5      | 2020-01-11T18:34:49 | 2020-01-11T18:50:20 | 15             | 10       | 60            | 2     |
| 105         | 7        | 2         | 2      | 2020-01-08T21:20:29 | 2020-01-08T21:30:45 | 10             | 25       | 60            | 1     |

-------------------------------

**QUESTION 5**

If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how
much money does Pizza Runner have left over after these deliveries?

```sql
WITH sales AS (
          SELECT SUM(CASE WHEN oct.pizza_id= 1 THEN order_count*12
						              WHEN oct.pizza_id = 2 THEN order_count*10
						              END) sales_in_dollars
          FROM (SELECT c.pizza_id, COUNT(c.order_id) as order_count
                FROM customer_orders_temp c
                JOIN runner_orders_temp r
                ON c.order_id = r.order_id
                WHERE r.cancellation = ''
                GROUP BY 1) oct
          JOIN pizza_names p
          ON oct.pizza_id = p.pizza_id),
runners_cost AS (
            SELECT SUM(distance) * 0.30 AS total_runners_cost
            FROM runner_orders_temp)
SELECT (sales_in_dollars - total_runners_cost) AS net_profit
FROM sales,runners_cost;
```


I created two CTEs. In the first CTE(`sales`) I calculated the total sales Pizza Runner made from selling Meatlovers $12 and Vegetarian $10 and not charging for
extras.
In the second CTE(`runners_cost`), i multiplied 0.30 by the total distance all the runners covered. Then i subtracted that cost from the total sales.

Results:

| net_sales |
| ---------- |
| 94.44      |

* Pizza Runner had $94.44 dollars after paying for deliveries.

---------------------------------------------------------------------

**BONUS QUESTIONS**

If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen 
if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

```sql
INSERT INTO pizza_recipes(pizza_id, toppings)
VALUES (3,'1,2,3,4,5,6,7,8,9,10,11,12');

INSERT INTO pizza_names(pizza_id, pizza_name)
VALUES (3, 'Supreme');

SELECT pr.pizza_id, pn.pizza_name, pr.toppings
FROM pizza_recipes pr
JOIN pizza_names pn
ON pr.pizza_id = pn.pizza_id;
```

Results:

| pizza_id | pizza_name | toppings                   |
| -------- | ---------- | -------------------------- |
| 1        | Meatlovers | 1, 2, 3, 4, 5, 6, 8, 10    |
| 2        | Vegetarian | 4, 6, 7, 9, 11, 12         |
| 3        | Supreme    | 1,2,3,4,5,6,7,8,9,10,11,12 |

-------------------------------------------------------------------------
