Requirements
-----

1. In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:

2. Convert the week_date to a DATE format

3. Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

4. Add a month_number with the calendar month for each week_date value as the 3rd column

5. Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values

6. Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

|segment|	age_band
-------|--------
1|	Young Adults
2	|Middle Aged
3 |or 4	Retirees

7. Add a new demographic column using the following mapping for the first letter in the segment values:

segment|	demographic
------|-------
C	|Couples
F|	Families

8. Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns

9. Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record


Query
-----

```sql
SET search_path to data_mart;

SELECT week_date :: date,
       extract('week' from week_date::date) as week_number,
       extract('month' from week_date::date) as month_number,
       extract('year' from week_date::date) as calendar_year,
       region,
       platform,
       CASE WHEN segment = 'null' THEN 'unknown'
            ELSE segment
            END AS segment,
       CASE WHEN right(segment,1) = '1' THEN 'Young Adults'
            WHEN right(segment,1) = '2' THEN 'Middle Aged'
            WHEN right(segment,1) = '3' OR right(segment,1) = '4' THEN 'Retirees'
            ELSE 'unknown'
            END AS age_band,
       CASE WHEN left(segment,1) ='C' THEN 'Couples'
            WHEN left(segment,1) = 'F' THEN 'Families'
            ELSE 'unknown'
            END AS demographic,
       customer_type,
       transactions,
       sales,
       round((sales::numeric/transactions::numeric),2) as avg_transaction
  INTO clean_weekly_sales --new table
  FROM weekly_sales;

SELECT *
  FROM clean_weekly_sales 
 LIMIT 20;
```	   
	   
  
