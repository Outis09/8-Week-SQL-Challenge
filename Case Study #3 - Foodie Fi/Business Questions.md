**Question:** How would you calculate the rate of growth for Foodie-Fi?
-----

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

**Question:** What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?
-----

The key metrics to track to assess performance over time include:
* number of new customers for each month in a year
* number of churns after trial for each month
* number of upgrades from basic monthly to pro monthly or pro annual plans
* revenue from each month for each year
* number of customers for each month

These metrics are suggested in months however at the end of the each year, the same metric can be used to compare one year to the previous year and assess performnce.

---------------------------------------------------------

**Question:** What are some key customer journeys or experiences that you would analyse further to improve customer retention?
-----

To improve customer retention I would analyze:
* customers who churned after trial
* customers who downgraded
* customers who churned after subscribing to any of the plans after trial
* customers who upgraded their subscriptions
* customers who upgraded from trial to pro annual

By grouping and analyzing these customers, I can identify trends and areas that can be leveraged to retain new customers who patronize the business and whose 
characteristics align with the trend of any group.

------------------------------------------------------

**Question:** If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?
-----

* Why did you sign up for the service?
* Was the service what you expected?
* Have you tried a similar service? How does our service compare?
* What would it take for you to consider re-subscribing to our service?
* How can we improve?

-----------------------------------

**Question:** What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?
------

##### Analyzing customers who churn

By analyzing customers who churn, foodie-Fi can identify traits that are common amongst these customers. After identifying these traits, Foodie-fi can use them to identify new customers who are at risk of churning and find ways to retain them. These ways could be:

* offering discounts
* offering an option to pause subscriptions
* offering special packages engaging them

To evaluate this, Foodie-Fi would have to monitor customers who are most likely to churn, and after using various means to retain the, calculate the percentage that was retained and for those who still churned, how long it took for them to churn.

##### Identifying and targeting the right audience. 

Customers could be churning because they do not like the service and there is little to nothing that Foodie-Fi can do to convince these customers. So to reduce churns and improve customer retention, Foodie-Fi can identify the right audience and make them the target market. This can be done by analyzing customers that have upgraded, customers who subscribed after trial and customers who have been subscribed for a period of time. The common traits these customers share can be analyzed and used as the basis for targeting a particular market. For example, if these customers are students or newly wedded couples or belong to a particular geographical location, Foodie-Fi could then target them because they have  a higher probability of being retained. This can be evaluated by analysing customers gained after this target marketing and calculating the retention rate.


--------------------------------------------------


