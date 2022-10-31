This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.

Taking the `week_date` value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before

Using this analysis approach - answer the following questions:

---------------

1.What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
-----
First I found the week number of the baseline week which is 2020-06-15.

**Query:**

```sql
SELECT DISTINCT week_number as baseline_week
FROM clean_weekly_sales
WHERE week_date = '2020-06-15';
```

**Results:**

| baseline_week |
| ------------- |
| 25            |

The baseline week number if 25. So four weeks before the change will be week numbers 21,22,23 and 24. The four week numbers after are 25,26,27, and 28.

Now I will look at the change and percent change between the two periods.

**Query:**

```sql
WITH impact as(
    SELECT SUM(CASE WHEN week_number BETWEEN 25 and 28 THEN sales ELSE 0 END)as sales_after,
           SUM(CASE WHEN week_number BETWEEN 21 and 24 THEN sales ELSE 0 END)as sales_before
      FROM clean_weekly_sales 
     WHERE calendar_year=2020
              )
SELECT sales_before,
       sales_after,
       sales_after-sales_before as change,
       round(((sales_after::numeric-sales_before::numeric)/sales_before::numeric)*100,2) as percent_change
 FROM impact;
```
