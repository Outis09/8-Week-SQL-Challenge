**Question 1**

How many customers has Foodie-Fi ever had?

```sql
SELECT COUNT(DISTINCT customer_id) as num_of_customers
  FROM subscriptions;
```

I counted the distinct number of customers that had Foodie-Fi had had.

Results:

| num_of_customers |
| ---------------- |
| 1000             |

* Foodie-Fi has had 1000 customers

-------------------------------


**Question 2**

What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```sql
  SELECT to_char(start_date, 'month'),count(plan_id) as num_of_trial
    FROM subscriptions
   WHERE plan_id = 0
GROUP BY date_trunc('month',start_date),to_char(start_date, 'month')
ORDER BY date_trunc('month',start_date);
```

I counted the number of trials and grouped by each month then I used `to_char` to extract the name of each month.

Results :

| to_char   | num_of_trial |
| --------- | ------------ |
| january   | 88           |
| february  | 68           |
| march     | 94           |
| april     | 81           |
| may       | 88           |
| june      | 79           |
| july      | 89           |
| august    | 88           |
| september | 87           |
| october   | 79           |
| november  | 75           |
| december  | 84           |

There were;
* 88 trials in January
* 68 trials in February
* 94 trials in March
* 81 trials in April
* 88 trials in May
* 79 trials in June
* 89 trials in July 
* 88 trials in August
* 87 trials in September
* 79 trials in October
* 75 trials in November
* 84 trials in December

---------------------------------

**Question 3**

What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```sql
  SELECT p.plan_name, COUNT(s.plan_id)
    FROM subscriptions s
    JOIN plans p
      ON s.plan_id = p.plan_id
   WHERE start_date > '2020-12-31'
GROUP BY 1;
```
