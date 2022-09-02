Subscriptions (sample)
-----------
customer_id|	plan_id	|start_date
---------|---------|--------
1|	0	|2020-08-01
1|	1	|2020-08-08
2|	0	|2020-09-20
2|	3	|2020-09-27
11|	0	|2020-11-19
11|	4	|2020-11-26
13|	0	|2020-12-15
13|	1	|2020-12-22
13|	2	|2021-03-29
15|	0	|2020-03-17
15|	2	|2020-03-24
15|	4	|2020-04-29
16|	0	|2020-05-31
16|	1	|2020-06-07
16|	3	|2020-10-21
18|	0	|2020-07-06
18|	2	|2020-07-13
19|	0	|2020-06-22
19|	2	|2020-06-29
19|	3	|2020-08-29

QUESTION
----------
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

---------------

Customer 1 
-----------
```sql
SELECT s.customer_id,p.plan_name,s.start_date
  FROM subscriptions s
  JOIN plans p
    ON s.plan_id = p.plan_id
 WHERE s.customer_id = 1;
```

#### Results 


| customer_id | plan_name     | start_date               |
| ----------- | ------------- | ------------------------ |
| 1           | trial         | 2020-08-01 |
| 1           | basic monthly | 2020-08-08 |

#### Analysis

Customer 1 signed up for the initial 7 day free trial on 1st August 2020.The trial plan ended on a week later and instead of the automatic subscription
to `pro monthly`, Customer 1 downgraded to the `basic monthly` subscription.

-----------

Customer 2
----------
```sql
SELECT s.customer_id,p.plan_name,s.start_date
  FROM subscriptions s
  JOIN plans p
    ON s.plan_id = p.plan_id
 WHERE s.customer_id = 2;
```

#### Results

| customer_id | plan_name  | start_date               |
| ----------- | ---------- | ------------------------ |
| 2           | trial      | 2020-09-20 |
| 2           | pro annual | 2020-09-27 |

#### Analysis

Customer 2 signed up for the initial 7 day free trial on 20th september,2020. A week later when the free trial ended Customer 2 subscribed to the 
`pro annual` plan instead of the automatic `pro monthly` subscription.

---------------------

Customer 11
---------
```sql
SELECT s.customer_id,p.plan_name,s.start_date
  FROM subscriptions s
  JOIN plans p
    ON s.plan_id = p.plan_id
 WHERE s.customer_id = 11;
```

#### Results

| customer_id | plan_name | start_date               |
| ----------- | --------- | ------------------------ |
| 11          | trial     | 2020-11-19 |
| 11          | churn     | 2020-11-26 |

#### Analysis

Customer 11 signed up for the 7 day free trial on 19 November,2022. A week later when the free trial ended, Customer 11 cancelled the plan (`churn`).

-------------------------

Customer 13
-------

```sql
SELECT s.customer_id,p.plan_name,s.start_date
  FROM subscriptions s
  JOIN plans p
    ON s.plan_id = p.plan_id
 WHERE s.customer_id = 13;
```

#### Results

| customer_id | plan_name     | start_date               |
| ----------- | ------------- | ------------------------ |
| 13          | trial         | 2020-12-15 |
| 13          | basic monthly | 2020-12-22 |
| 13          | pro monthly   | 2021-03-29 |

#### Analysis

Customer 13 signed up for the 7 day free trial on 15th December 2020 and a week later when the trial ended, Customer 13 downgraded 
to a basic monthly subscription. Customer 13 continued this subcription for 3 months and upgraded to the pro montly subscription.

------------------------------

Customer 15
--------

```sql
SELECT s.customer_id,p.plan_name,s.start_date
  FROM subscriptions s
  JOIN plans p
    ON s.plan_id = p.plan_id
 WHERE s.customer_id = 15;
```

#### Results

| customer_id | plan_name   | start_date               |
| ----------- | ----------- | ------------------------ |
| 15          | trial       | 2020-03-17 |
| 15          | pro monthly | 2020-03-24 |
| 15          | churn       | 2020-04-29 |

#### Analysis

Customer 15 started the 7 day free trial on 17th March 2020 and after it ended they subcribed to the pro monthly plan. A week after the first plan ended,
they cancelled it.however because they cancelled after a week after the first plan ended,it is likely they subscribed for a second month.therefore even though 
the plan was cancelled, the second subscription will run till 2020-05-24

-----------------------------------------------------------------

Customer 16
------------

```sql
SELECT s.customer_id,p.plan_name,s.start_date
  FROM subscriptions s
  JOIN plans p
    ON s.plan_id = p.plan_id
 WHERE s.customer_id = 16;
```

#### Results

| customer_id | plan_name     | start_date               |
| ----------- | ------------- | ------------------------ |
| 16          | trial         | 2020-05-31 |
| 16          | basic monthly | 2020-06-07 |
| 16          | pro annual    | 2020-10-21 |


#### Analysis

Customer 16 started the 7 day free trial on 31st May 2020. when the trial ended they subscribed to the basic monthly plan.The basic montly plan continued for 4 
months.they then upgraded to the pro monthly plan.However they upgraded after the 4th month's subscription had ended but didnt cancel.So it is implies that they 
were on their subcription for basic monthly for the fifth month when they upgraded.

-----------------------------------

