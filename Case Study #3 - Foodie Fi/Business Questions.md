**Question:** How would you calculate the rate of growth for Foodie-Fi?

One way to look at growth would be to calculate the number of trials in each month. This tells us the number of new customers that tried the service.
The query below returns a table that shows this and calculates the percentage change as well.

```sql
WITH trial_cte AS (
  SELECT date_trunc('month',start_date) as month,
         count(customer_id) as num_of_trials,
         lag(count(customer_id)) over (order by date_trunc('month',start_date)) as previous_num
    FROM subscriptions
   WHERE plan_id = 0
GROUP BY 1
ORDER BY 1)

  SELECT month,
         num_of_trials,
         ((num_of_trials - previous_num )*100)/previous_num as percentage_change
   FROM trial_cte
```

**Results:**

| month                    | num_of_trials | percentage_change |
| ------------------------ | ------------- | ----------------- |
| 2020-01-01   | 88            |                   |
| 2020-02-01   | 68            | -22               |
| 2020-03-01   | 94            | 38                |
| 2020-04-01   | 81            | -13               |
| 2020-05-01   | 88            | 8                 |
| 2020-06-01   | 79            | -10               |
| 2020-07-01   | 89            | 12                |
| 2020-08-01   | 88            | -1                |
| 2020-09-01   | 87            | -1                |
| 2020-10-01   | 79            | -9                |
| 2020-11-01   | 75            | -5                |
| 2020-12-01   | 84            | 12                |

The table shows the month, the number of trials and the percentage change relative to the previous month. For example, in February 2020, there was a 22% decrease 
in number of trials. However, in March 2020, there was a 38% increase in number of trials.
The table only includes months in 2020 because there hadn't been any new trials in 2021 at the time the data was collected. Since the last subscription recorded was 
in April 2021, and no new trials had been recorded in 2021 by then, there is the need to investigate what the reasons might be but the data collected is limited and 
does not allow for that analysis.

Other ways to calculate growth include calculating the number of churns for each month and comparing it to the previous year or analyzing the number of upgrades to 
pro monthly and pro annual plans. By year's end in 2021, growth can be calculated by using the metrics stated to compare 2020 and 2021.

----------------------------------------------------

**Question:**
