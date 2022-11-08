<div align="center">
 <h1>Pizza Runner :pizza:</h1>
</div>

<p align="center">
    <img src="https://8weeksqlchallenge.com/images/case-study-designs/2.png" width="400" height="400">
</p>

Pizza Runner is an _uberized_ pizza delivery business. Runners pick up orders from the Pizza Runner headquarters and deliver to customers. The owner has collected data on the orders, runners and customers and would like the data analyzed so he can optimize his business.

However, the data is messy and will need some cleaning before analysis. The queries used to clean the data are contained in the **[Data Cleaning](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232-%20Pizza%20Runner/Data_Cleaning.md)** file. 

The owner's business questions have been grouped into four:
* **[Pizza Metrics](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232-%20Pizza%20Runner/Solutions/Pizza%20Metrics.md)** - questions on number of unique orders, orders for each pizza type and breakdown of orders by hour of day.
* **[Runner and Customer Experience](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232-%20Pizza%20Runner/Solutions/Runner%20and%20Customer%20Experience.md)** - questions on runners, cancelled orders and delivery times.
* **[Ingredient Optimisation](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232-%20Pizza%20Runner/Solutions/Ingredients%20Optimization.md)** - questions on ingredients used in pizza preparations.
* **[Pricing and Ratings](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232-%20Pizza%20Runner/Solutions/Pricing%20and%20Ratings.md)** - questions on revenue, and delivery costs.


The [schema](schema.md) contains the code to create the database in SQL.

This case study tests :
-----------
  
  * Data Cleaning with SQL
  * Temporary Tables
  * Common Table Expressions
  * Dealing with NULL values
  * Table Joins 
  * Casting
  
  **Available Data**
  ---------
  
  **Entity Relationship Diagram**
  
  ![Pizza Runner-2](https://user-images.githubusercontent.com/104911707/200674892-a2667146-6204-4177-98ce-5065e7d8fbb9.png)

**Table 1: runners**

The runners table shows the `registration_date` for each new runner

|runner_id|	registration_date
----------|---------
1	|2021-01-01
2	|2021-01-03
3	|2021-01-08
4	|2021-01-15

**Table 2: customer_orders**

Customer pizza orders are captured in the `customer_orders` table with 1 row for each individual pizza that is part of the order.

The `pizza_id` relates to the type of pizza which was ordered whilst the exclusions are the `ingredient_id` values which should be removed from the pizza and the extras are the `ingredient_id` values which need to be added to the pizza.

Note that customers can order multiple pizzas in a single order with varying `exclusions` and `extras` values even if the pizza is the same type!

The `exclusions` and `extras` columns will need to be cleaned up before using them in your queries.

order_id	|customer_id|	pizza_id	|exclusions	|extras|	order_time
---------|----------|----------|---------|------|---------
1	|101|	1	| | 	 	|2021-01-01 18:05:02
2	|101	|1| | |	 	 	|2021-01-01 19:00:52
3	|102	|1| | 	 	 	|2021-01-02 23:51:23
3	|102|	2| |	 	NaN|	2021-01-02 23:51:23
4	|103	|1|	4| | |	 	2021-01-04 13:23:46
4	|103	|1	|4| | |	 	2021-01-04 13:23:46
4	|103	|2	|4| | |	 	2021-01-04 13:23:46
5	|104	|1	|null	|1	|2021-01-08 21:00:29
6	|101	|2	|null|	null	|2021-01-08 21:03:13
7	|105	|2	|null|	1|	2021-01-08 21:20:29
8	|102	|1	|null	|null	|2021-01-09 23:54:33
9	|103	|1	|4	1, |5	|2021-01-10 11:22:59
10|104	|1	|null	|null	|2021-01-11 18:34:49
10	|104|	1	|2, 6	|1, 4	|2021-01-11 18:34:49
  
**Table 3: runner_orders**

After each orders are received through the system - they are assigned to a runner - however not all orders are fully completed and can be cancelled by the restaurant or the customer.

The `pickup_time` is the timestamp at which the runner arrives at the Pizza Runner headquarters to pick up the freshly cooked pizzas. The `distance` and `duration` fields are related to how far and long the runner had to travel to deliver the order to the respective customer.

There are some known data issues with this table so be careful when using this in your queries - make sure to check the data types for each column in the schema SQL!

Sample:

order_id|	runner_id|	pickup_time	|distance|	duration	|cancellation
----------|------|---------|-------|--------|--------------
1|	1	|2021-01-01 18:15:34	|20km	|32 minutes	| | 
2	|1	|2021-01-01 19:10:54	|20km	|27 minutes	| | 
3	|1	|2021-01-03 00:12:37	|13.4km|	20 mins|	NaN|
4	|2	|2021-01-04 13:53:03	|23.4	|40	|NaN|
5	|3	|2021-01-08 21:10:57	|10	|15	|NaN
6	|3	|null|	null|	null|	Restaurant Cancellation
7	|2	|2020-01-08 21:30:45	|25km	|25mins|	null
8	|2	|2020-01-10 00:15:02	|23.4 km	|15 minute	null
9	|2	|null	|null	|null	|Customer Cancellation
10	|1|	2020-01-11 18:50:20|	10km|	10minutes	|null
  
   
**Table 4: pizza_names**

At the moment - Pizza Runner only has 2 pizzas available the Meat Lovers or Vegetarian!

pizza_id|	pizza_name
--------|-----------
1	|Meat Lovers
2	|Vegetarian


**Table 5: pizza_recipes**

Each `pizza_id` has a standard set of `toppings` which are used as part of the pizza recipe.

pizza_id	|toppings
------------|-------------
1|	1, 2, 3, 4, 5, 6, 8, 10
2	|4, 6, 7, 9, 11, 12


**Table 6: pizza_toppings**

This table contains all of the `topping_name` values with their corresponding `topping_id` value

topping_id|	topping_name
-----------|-----------------
1	|Bacon
2	|BBQ Sauce
3	|Beef
4	|Cheese
5	|Chicken
6	|Mushrooms
7	|Onions
8	|Pepperoni
9	|Peppers
10|	Salami
11|	Tomatoes
12|	Tomato Sauce
