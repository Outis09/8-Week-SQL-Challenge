The `pizza_recipes` table looks like this:

| pizza_id | toppings                |
| -------- | ----------------------- |
| 1        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 2        | 4, 6, 7, 9, 11, 12      |

Running queries with the table in this format requires long queries therefore I opted to reformat the table but still maintain the information. This makes it easier and faster to query the table for data.

```sql
CREATE TEMPORARY TABLE pizza_recipes_temp AS (
                                    SELECT pizza_id, 
                                           TRIM(' ' FROM unnest(string_to_array(toppings, ',')))::int as toppings
  				    FROM pizza_recipes);
```

I created a temporary table (`pizza_recipes_temp`). The datatype of the `toppings` column in the original column is string or text. So I converted it to an array and 
unnested the array created using "," as the seperator so that the individual values were retrieved. However, the individual values had white spaces attached to them so I
used  `TRIM` to remove the white spaces. i then casted the datatype to `int` so that i could perform numeric operations on them.

The resulting table looks like this:

| pizza_id | toppings |
| -------- | -------- |
| 1        | 1        |
| 1        | 2        |
| 1        | 3        |
| 1        | 4        |
| 1        | 5        |
| 1        | 6        |
| 1        | 8        |
| 1        | 10       |
| 2        | 4        |
| 2        | 6        |
| 2        | 7        |
| 2        | 9        |
| 2        | 11       |
| 2        | 12       |

------------------------------------------------

**QUESTION 1**

What are the standard ingredients for each pizza?

```sql
SELECT pn.pizza_name,
       pt.topping_name
FROM pizza_names pn
JOIN pizza_recipes_temp prt
ON pn.pizza_id = prt.pizza_id
JOIN pizza_toppings pt
ON prt.toppings = pt.topping_id
ORDER BY pn.pizza_name;
```

The `pizza_recipes_temp` table contained just the numbers representing the pizzas and the ingredients. For this query I needed the names so I joined the `pizza_names` 
table and the `pizza_toppings` table. I selected the pizza name and the name of the toppings for each pizza.

Results:
| pizza_name | topping_name |
| ---------- | ------------ |
| Meatlovers | BBQ Sauce    |
| Meatlovers | Pepperoni    |
| Meatlovers | Cheese       |
| Meatlovers | Salami       |
| Meatlovers | Chicken      |
| Meatlovers | Bacon        |
| Meatlovers | Mushrooms    |
| Meatlovers | Beef         |
| Vegetarian | Tomato Sauce |
| Vegetarian | Cheese       |
| Vegetarian | Mushrooms    |
| Vegetarian | Onions       |
| Vegetarian | Peppers      |
| Vegetarian | Tomatoes     |

* The Meatlovers pizza's standard ingredients are BBQ Sauce, Pepperoni, Cheese, Salami, Chicken, Bacon, Mushrooms and Beef
* The Vegetarian pizza's standard ingredients are Tomato Sauce, Cheese, Mushrooms, Onions, Peppers and Tomatoes.

----------------------------------------------

**QUESTION 2**

What was the most commonly added extra?

```sql
WITH common_extras AS (
                  SELECT order_id,
                         unnest(string_to_array(extras, ','))::int as extras
		  FROM customer_orders_temp
                      )
SELECT pt.topping_name AS extras, COUNT(order_id)
FROM common_extras cex
JOIN pizza_toppings pt
ON cex.extras = pt.topping_id
GROUP BY extras, pt.topping_name
ORDER BY COUNT(order_id) DESC
LIMIT 1;
```

The `extras` column in the `customer_orders_temp` table some rows with two numbers as strings. I created a CTE (`common_extras`) and converted those strings to an array 
and unnested the array to get the individual numbers. Then i casted the datatype of the numbers to `int` because they were `strings`. I joined the `pizza_toppings` table 
so i could extract the name of the topping using the ID. I then counted the orders and grouped by the extras and ordered by the count in descending order so that
the highest was displayed first. Then i limited the results to one row so that only the highest was displayed.

Results:

| extras | count |
| ------ | ----- |
| Bacon  | 4     |

* The most commonly added extras is Bacon and it was added as an extra to 4 different orders

----------------------------------

**QUESTION 3**

What was the most common exclusion?

```sql
WITH common_exclusions AS (
                        SELECT order_id,
                               unnest(string_to_array(exclusions, ','))::int as exclusions
						            FROM customer_orders_temp
                        )
SELECT pt.topping_name AS exclusion, COUNT(order_id)
FROM common_exclusions ce
JOIN pizza_toppings pt
ON ce.exclusions = pt.topping_id
GROUP BY pt.topping_name
ORDER BY COUNT(order_id) DESC
LIMIT 1;
```

The `exclusions` column also had two numbers in rows as strings so i created a CTE(`common_exclusions`) and converted the strings to an array and unnested the array.
Then i casted the datatype to an `int`. I joined the `pizza_toppings` table so i could extract the name of the exclusion. I selected the name of the toppings, counted 
the number of orders and grouped by exclusions so that for each exclusions i could get the number of orders on which the exclsuions were made. I ordered by the count
in descending order and limited the results to one row so that only the highest count was displayed.

Results:

| exclusions | count |
| ------ | ----- |
| Cheese | 4     |

* The most commom exclusionis Cheese and it was excluded on 4 different orders

--------------------------------------

**QUESTION 4**

Generate an order item for each record in the customers_orders table in the format of one of the following:
* Meat Lovers
* Meat Lovers - Exclude Beef
* Meat Lovers - Extra Bacon
* Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```sql
SELECT c.order_id,
       c.customer_id, 
		   CASE WHEN pizza_id = 1 AND exclusions = '' AND extras = '' THEN 'Meat Lovers'
		        WHEN pizza_id = 2 AND exclusions = '' AND extras = '' THEN  'Vegetarian'
		        WHEN pizza_id = 1 AND exclusions = '4' AND  extras = '' THEN 'Meat Lovers - Exclude Cheese'
		        WHEN pizza_id = 2 AND exclusions = '4' AND  extras = '' THEN 'Vegetarian - Exclude Cheese'
		        WHEN pizza_id =1 AND exclusions = '' AND extras = '1' THEN 'Meat Lovers - Extra Bacon'
		        WHEN pizza_id = 2 AND exclusions = '' AND extras = '1' THEN 'Vegetarian - Extra Bacon'
		        WHEN pizza_id = 1 AND exclusions = '4' AND extras = '1, 5' THEN 'Meat Lovers - Exclude Cheese-  Extra Bacon and Chicken'
		        WHEN pizza_id = 1 AND exclusions = '2, 6' AND extras = '1, 4' THEN 'Meat Lovers - Exclude BBQ Sauce and Mushrooms - Extra Bacon and Cheese'
            END AS Orders
FROM customer_orders_temp c;
```

I used a lot of `CASE WHEN` statements for this query. I used them to identify each unique condition in the `customer_orders_temp` table.

Results:

| order_id | customer_id | orders                                                                 |
| -------- | ----------- | ---------------------------------------------------------------------- |
| 1        | 101         | Meat Lovers                                                            |
| 2        | 101         | Meat Lovers                                                            |
| 3        | 102         | Meat Lovers                                                            |
| 3        | 102         | Vegetarian                                                             |
| 4        | 103         | Meat Lovers - Exclude Cheese                                           |
| 4        | 103         | Meat Lovers - Exclude Cheese                                           |
| 4        | 103         | Vegetarian - Exclude Cheese                                            |
| 5        | 104         | Meat Lovers - Extra Bacon                                              |
| 6        | 101         | Vegetarian                                                             |
| 7        | 105         | Vegetarian - Extra Bacon                                               |
| 8        | 102         | Meat Lovers                                                            |
| 9        | 103         | Meat Lovers - Exclude Cheese-  Extra Bacon and Chicken                 |
| 10       | 104         | Meat Lovers                                                            |
| 10       | 104         | Meat Lovers - Exclude BBQ Sauce and Mushrooms - Extra Bacon and Cheese |

------------------------------------

**QUESTION 5**

Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: 
* "Meat Lovers: 2xBacon, Beef, ... , Salami"

```sql
SELECT c.order_id, c.customer_id,
		CASE WHEN pizza_id = 1 AND exclusions = '' AND extras = '' THEN 'Meat Lovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms,Pepperoni and Salami'
			   WHEN pizza_id = 2 AND exclusions = '' AND extras = '' THEN 'Vegetarian: Cheese,Mushrooms, Onions, Peppers, Tomatoes and Tomato Sauce'
			   WHEN pizza_id = 1 AND exclusions = '4' AND extras = '' THEN 'Meat Lovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms,Pepperoni and Salami'
			   WHEN pizza_id = 2 AND exclusions = '4' AND extras = '' THEN 'Vegetarian: Mushrooms, Onions, Peppers, Tomatoes and Tomato Sauce'
			   WHEN pizza_id = 1 AND exclusions = '' AND extras = '1' THEN 'Meat Lovers: 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms,Pepperoni and Salami'
			   WHEN pizza_id = 2 AND exclusions = '' AND extras = '1' THEN 'Vegetarian: Bacon, Cheese,Mushrooms, Onions, Peppers, Tomatoes and Tomato Sauce'
			   WHEN pizza_id = 1 AND exclusions = '4' AND extras = '1, 5' THEN 'Meat Lovers: 2x Bacon, BBQ Sauce, Beef, 2x Chicken, Mushrooms,Pepperoni and Salami ' 
			   WHEN pizza_id = 1 AND exclusions = '2, 6' AND extras = '1, 4' THEN 'Meat Lovers: 2x Bacon, Beef, 2x Cheese, Chicken,Pepperoni and Salami'
		     END AS Orders
FROM customer_orders_temp c;
```

I used `CASE WHEN` statments to specify the unique conditions in the table.

Results:

| order_id | customer_id | orders                                                                                  |
| -------- | ----------- | --------------------------------------------------------------------------------------- |
| 1        | 101         | Meat Lovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms,Pepperoni and Salami    |
| 2        | 101         | Meat Lovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms,Pepperoni and Salami    |
| 3        | 102         | Meat Lovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms,Pepperoni and Salami    |
| 3        | 102         | Vegetarian: Cheese,Mushrooms, Onions, Peppers, Tomatoes and Tomato Sauce                |
| 4        | 103         | Meat Lovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms,Pepperoni and Salami            |
| 4        | 103         | Meat Lovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms,Pepperoni and Salami            |
| 4        | 103         | Vegetarian: Mushrooms, Onions, Peppers, Tomatoes and Tomato Sauce                       |
| 5        | 104         | Meat Lovers: 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms,Pepperoni and Salami |
| 6        | 101         | Vegetarian: Cheese,Mushrooms, Onions, Peppers, Tomatoes and Tomato Sauce                |
| 7        | 105         | Vegetarian: Bacon, Cheese,Mushrooms, Onions, Peppers, Tomatoes and Tomato Sauce         |
| 8        | 102         | Meat Lovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms,Pepperoni and Salami    |
| 9        | 103         | Meat Lovers: 2x Bacon, BBQ Sauce, Beef, 2x Chicken, Mushrooms,Pepperoni and Salami      |
| 10       | 104         | Meat Lovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms,Pepperoni and Salami    |
| 10       | 104         | Meat Lovers: 2x Bacon, Beef, 2x Cheese, Chicken,Pepperoni and Salami                    |

----------------------------------

**QUESTION 6**

What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

```sql
WITH 
ingredients_cte AS (
          SELECT topping_name,
                 COUNT(sq1.pizza_id) as ingredients
          FROM pizza_toppings pt
          JOIN (SELECT c.pizza_id, p.toppings
                FROM customer_orders_temp c
                JOIN pizza_recipes_temp p
                ON c.pizza_id = p.pizza_id
                JOIN runner_orders_temp r
                ON c.order_id = r.order_id
                WHERE cancellation = '') sq1
         ON pt.topping_id = sq1.toppings
        GROUP BY 1
        ),
exclusions_cte AS (
          SELECT topping_name, COUNT(exclusions) as exclusions
          FROM pizza_toppings p
          JOIN (SELECT c.order_id,
                       c.pizza_id ,
                       TRIM(' ' FROM UNNEST(string_to_array(exclusions, ',') )) :: int AS exclusions
                FROM customer_orders_temp c
                JOIN runner_orders_temp r
                ON c.order_id = r.order_id 
                WHERE r.cancellation = '') sq2
          ON p.topping_id = sq2.exclusions
          GROUP BY 1),
extras_cte AS (
        SELECT  topping_name, COUNT(order_id) AS extras
        FROM pizza_toppings p
        JOIN (SELECT c.order_id, 
                     c.pizza_id ,
                     TRIM(' ' FROM UNNEST(string_to_array(extras, ',') )) :: int AS extras
              FROM customer_orders_temp c
              JOIN runner_orders_temp r
              ON c.order_id = r.order_id 
              WHERE r.cancellation = '') sq3
        ON p.topping_id = sq3.extras
        GROUP BY 1)

SELECT int.topping_name, int.ingredients,exc.exclusions , extras, CASE WHEN exclusions IS NULL AND extras IS NULL THEN ingredients
																	WHEN exclusions IS NOT NULL AND extras IS NULL THEN ingredients - exclusions
																	WHEN exclusions IS NULL AND extras IS NOT NULL THEN ingredients + extras
																	WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN ingredients + extras - exclusions
															END AS eppme
FROM ingredients_cte int
LEFT JOIN exclusions_cte exc
ON int.topping_name = exc.topping_name
LEFT JOIN extras_cte ext
ON int.topping_name = ext.topping_name
ORDER BY eppme DESC;
```

I created 3 CTEs. 

I created the first CTE(`ingredients_cte`) to get the ingredients and the number of times they had been used to prepare different pizzas. In the subquery(`sq1`), I selected `pizza_id` and 
the standard ingredients used to prepare each pizza each time it was orders. Then I filtered to include only those that were not cancelled which means they
were delivered. The subquery returns the IDs of the pizza and the ingredients. In the CTE `t1` I selected the name of the ingredients from the `pizza_toppings` table.
I then counted all the pizzas from the results of the subquery so that for each ingredient, I got the number of times it was used to prepare delivered pizzas.

In the second CTE(`exclusions_cte`) I wanted to get exclusion and the number of times it was excluded on delivered pizzas. In the subquery(`sq2`) in this CTE, i converted the exclusions
column to an array then i unnested the array and casted the datatype to an int and filtered to only include delivered orders. The results of this subquery would be 
the orders with exclusions, the pizza contained in those orders and the exclusions. Because I wanted to use the name, I joined the `pizza_toppings` table and selected 
the name of the ingredients being the exclusions. I then counted the pizzas and grouped by the name of the exclusion. The result of this CTE is the name of the 
exclusion and the number of times it was excluded on delivered pizzas.

In the third query(`extras_cte`) I wanted to get the extras and the number of times they were added to delivered pizzas. I wrote a subquery(`sq3`) to get the orders which were delivered, the pizzas in those orders and the ingredients that were added as extras. In the CTE i used the `pizza_toppings` table to get the name of the extras and counted the number of delivered pizzas that had these extras.

I combined the results of these CTE and used `CASE WHEN` statements to specify conditions. For each ingredient, the number of times it was used as `extras` was added 
to and the number of times it was used as `exclusions` was subtracted from the number of times it was supposed to have been used as a standard ingredient.

Results:

| topping_name | ingredients | exclusions | extras | eppme |
| ------------ | ----------- | ---------- | ------ | ----- |
| Bacon        | 9           |            | 3      | 12    |
| Mushrooms    | 12          | 1          |        | 11    |
| Cheese       | 12          | 3          | 1      | 10    |
| Pepperoni    | 9           |            |        | 9     |
| Chicken      | 9           |            |        | 9     |
| Salami       | 9           |            |        | 9     |
| Beef         | 9           |            |        | 9     |
| BBQ Sauce    | 9           | 1          |        | 8     |
| Tomato Sauce | 3           |            |        | 3     |
| Onions       | 3           |            |        | 3     |
| Tomatoes     | 3           |            |        | 3     |
| Peppers      | 3           |            |        | 3     |

* Bacon was used a total of 12 times in delivered pizzas.
* Mushrooms were used 11 times 
* Cheese was used 10 times
* Pepperoni, Chicken, Salami and Beef were used 9 times each
* BBQ Sauce was used 8 times
* Tomato Sauce, Onions, Tomatoes and Peppers were used 3 times each

------------------------------------------------------------


