**Question 1:**
Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month
--------------
 ```sql
 ALTER TABLE interest_metrics
ALTER COLUMN month_year SET DATA TYPE DATE USING month_year :: DATE;
```

I used the ALTER function to alter the table and to change the data type of the `month_year` column to date by first casting the column to date.
