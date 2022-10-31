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

**Results:**

| sales_before | sales_after | change    | percent_change |
| ------------ | ----------- | --------- | -------------- |
| 2345878357   | 2318994169  | -26884188 | -1.15          |


Now I will look compare the four weeks before and after week by week.

**Query:**

```sql
WITH running_change as (
        SELECT week_number,
               sum(sales) as tot_sales,
               LAG(sum(sales)) OVER(ORDER BY week_number) as prev_week_sales
          FROM clean_weekly_sales
         WHERE calendar_year = 2020 and week_number between 21 and 28
      GROUP BY week_number
      ORDER BY week_number
                       )
  SELECT week_number,
         tot_sales,
         prev_week_sales,
         round(((tot_sales::numeric-prev_week_sales::numeric)/prev_week_sales::numeric)*100,2) as percent_change
    FROM running_change
ORDER BY week_number;
```

**Results:**

| week_number | tot_sales | prev_week_sales | percent_change |
| ----------- | --------- | --------------- | -------------- |
| 21          | 585008090 |                 |                |
| 22          | 589120804 | 585008090       | 0.70           |
| 23          | 585466073 | 589120804       | -0.62          |
| 24          | 586283390 | 585466073       | 0.14           |
| 25          | 570025348 | 586283390       | -2.77          |
| 26          | 583242828 | 570025348       | 2.32           |
| 27          | 575390599 | 583242828       | -1.35          |
| 28          | 590335394 | 575390599       | 2.60           |

----------------------------------------
