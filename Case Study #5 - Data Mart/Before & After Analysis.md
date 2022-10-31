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
--gets total sales for the four weeks before and after the baseline week
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
I used a CTE,`impact`, to get the sum of sales for the four weeks before and after period. In the main query, I calculated the difference between the sales in the two periods and the percent change.

**Results:**

| sales_before | sales_after | change    | percent_change |
| ------------ | ----------- | --------- | -------------- |
| 2345878357   | 2318994169  | -26884188 | -1.15          |

There was a decrease in sales by 1.15% four weeks after the change was introduced as compared to four weeks before the change was introduced.


Now I will look compare the four weeks before and after week by week.

**Query:**

```sql
--gets week sales and previous week sales
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
I used a CTE,`running_change`, to get the total sales for each week in the 8 week period and the sales of the previous week. In the main query, I compared the sales for each week to the sales for the previous week in percentages.

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

The highest decrease in the 8 week period was recorded in the 25th week whihc is the week the change was introduced. The next highest decrease was recorded in the 27th week. These two percentage decreases are significantly higher than the decrease that was recorded in the four weeks before period which is 0.62.

----------------------------------------

2.What about the entire 12 weeks before and after?
----

**Query:**
```sql
--gets total sales for 12 weeks before and after the change
WITH impact as(
        SELECT SUM(CASE WHEN week_number BETWEEN 25 and 36 THEN sales ELSE 0 END)as sales_after,
               SUM(CASE WHEN week_number BETWEEN 13 and 24 THEN sales ELSE 0 END)as sales_before
          FROM clean_weekly_sales 
         WHERE calendar_year=2020
              )
SELECT sales_before,
       sales_after,
       sales_after-sales_before as change,
       round(((sales_after::numeric-sales_before::numeric)/sales_before::numeric)*100,2) as percent_change
  FROM impact;
```
I used a CTE,`impact`, to get the sales for the two 12 week periods. In the main query, I calculated the change and the change percentage.
**Results:**
| sales_before | sales_after | change     | percent_change |
| ------------ | ----------- | ---------- | -------------- |
| 7126273147   | 6973947753  | -152325394 | -2.14          |

There was a percentage decrease of 2.14% in sales in the 12 weeks after the change was introduced compared to the period before the change was introduced.

I will compare the weeks in the 24 week period.

**Query:**

```sql
WITH running_change as (
	SELECT week_number,
	       sum(sales) as tot_sales,
	       LAG(sum(sales)) OVER(ORDER BY week_number) as prev_week_sales
	  FROM clean_weekly_sales
	 WHERE calendar_year = 2020 and week_number between 13 and 36
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
| 13          | 638197506 |                 |                |
| 14          | 600898723 | 638197506       | -5.84          |
| 15          | 585551930 | 600898723       | -2.55          |
| 16          | 606143272 | 585551930       | 3.52           |
| 17          | 571874343 | 606143272       | -5.65          |
| 18          | 589783021 | 571874343       | 3.13           |
| 19          | 601398593 | 589783021       | 1.97           |
| 20          | 586547402 | 601398593       | -2.47          |
| 21          | 585008090 | 586547402       | -0.26          |
| 22          | 589120804 | 585008090       | 0.70           |
| 23          | 585466073 | 589120804       | -0.62          |
| 24          | 586283390 | 585466073       | 0.14           |
| 25          | 570025348 | 586283390       | -2.77          |
| 26          | 583242828 | 570025348       | 2.32           |
| 27          | 575390599 | 583242828       | -1.35          |
| 28          | 590335394 | 575390599       | 2.60           |
| 29          | 585936402 | 590335394       | -0.75          |
| 30          | 580874115 | 585936402       | -0.86          |
| 31          | 576349635 | 580874115       | -0.78          |
| 32          | 565709853 | 576349635       | -1.85          |
| 33          | 580024393 | 565709853       | 2.53           |
| 34          | 581075406 | 580024393       | 0.18           |
| 35          | 593379892 | 581075406       | 2.12           |
| 36          | 591603888 | 593379892       | -0.30          |

There were relatively high percentage decreases of 5.84%, 2.55%, 5.65% and 2.47% before the change was introduced. In the 12 weeks after the change was introduced, there were relatively high percentage decreases of 2.77%, 1.35%, and 1.85%. The percentage decreases in the 12 week period before the change are higher than the 12 week period after the change was introduced. Is the change the cause of the decrease? Are there extraneous variables affecting sales that have not been accounted for? More information is needed to answer these questions and the data does not provide enough information.

----------------------------------------------
