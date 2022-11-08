<div align="center">
 <h1>FOODIE-FI 🥑</h1>
</div>

<p align="center">
    <img src="https://8weeksqlchallenge.com/images/case-study-designs/3.png" width="400" height="400">
</p>

Foodie-Fi is a subscription based streaming service that has only food content. Kind of like Netflix but for food-related content. The owner sells monthly and annual subscriptions and has collected data on customers and their subscriptions. The owner has important business questions that can be answered using the data he has provided.

The owner's business questions have been divided into four:

* **[Customer Journey](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%233%20-%20Foodie%20Fi/Customer%20Journey.md)** - The owner wants to know the subscription journey of 8 select customers out of the 1000 customers.
* **[Data Analysis Questions](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%233%20-%20Foodie%20Fi/Data%20Analysis%20Questions.md)** - Specific business questions spanning customers, churns, and subscription upgrades.
* **[Challenge Payment Question](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%233%20-%20Foodie%20Fi/Challenge%20Payments%20Question.md)** - The Foodie-Fi teams wants a new table to contain the payment date and amounts based on certain conditions.
* **[Business Questions](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%233%20-%20Foodie%20Fi/Business%20Questions.md)** - Business Recommendations.

The [schema](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%233%20-%20Foodie%20Fi/Schema.md) file contains the codes to create the database.

**NB:** PostgreSQL was used in the analysis. 

The full case study can be accessed [here](https://8weeksqlchallenge.com/case-study-3/).

**Available Data**
-------

**Entity Relationship Diagram**

<p align="center">
    <img src="https://8weeksqlchallenge.com/images/case-study-3-erd.png">
</p>

**Table 1: plans**

Customers can choose which plans to join Foodie-Fi when they first sign up.

Basic plan customers have limited access and can only stream their videos and is only available monthly at $9.90

Pro plan customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.

Customers can sign up to an initial 7 day free trial will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.

When customers cancel their Foodie-Fi service - they will have a `churn` plan record with a `null` price but their plan will continue until the end of the billing period.

plan_id	|plan_name	|price
------------|---------|------------
0	|trial|	0
1	|basic monthly|	9.90
2	|pro monthly	|19.90
3	|pro annual	|199
4	|churn	|null


**Table 2: subscriptions**

Customer subscriptions show the exact date where their specific `plan_id` starts.

If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the `start_date` in the `subscriptions` table will reflect the date that the actual plan changes.

When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher plan will take effect straightaway.

When customers churn - they will keep their access until the end of their current billing period but the `start_date` will be technically the day they decided to cancel their service.

customer_id|	plan_id	|start_date
----------|----------|-----------
1|	0	|2020-08-01
1|	1	|2020-08-08
2|	0|	2020-09-20
2	|3	|2020-09-27
11|	0|	2020-11-19
11|	4	|2020-11-26
13|	0	|2020-12-15
13	|1	|2020-12-22
13	|2	|2021-03-29
15	|0	|2020-03-17
15	|2|	2020-03-24
15	|4	|2020-04-29
16	|0|	2020-05-31
16	|1	|2020-06-07
16	|3	|2020-10-21
18	|0	|2020-07-06
18	|2	|2020-07-13
19	|0	|2020-06-22
19	|2	|2020-06-29
19	|3	|2020-08-29

