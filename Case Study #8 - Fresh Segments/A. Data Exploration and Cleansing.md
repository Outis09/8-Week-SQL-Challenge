**Question 1:**
Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a date data type with the start of the month
--------------
 ```sql
 ALTER TABLE interest_metrics
ALTER COLUMN month_year SET DATA TYPE DATE USING month_year :: DATE;
```

I used the ALTER function to alter the table and to change the data type of the `month_year` column to date by first casting the column to date.

-------------------------------------
**Question 2:**
What is count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest) with the null values appearing first?
-----

```sql
    SELECT month_year, count(*) as count_of_records
    FROM fresh_segments.interest_metrics
    GROUP BY 1
    ORDER BY month_year NULLS FIRST;
```

I selected the `month_year` column and counted the number of records then I grouped by the `month_year` column. In the `ORDER BY` clause, I specified `NULLS FIRST` so that the count of null values is displayed first.

**Results:**

| month_year | count_of_records |
| ---------- | ---------------- |
|       [null]     | 1194             |
| 01-2019    | 973              |
| 02-2019    | 1121             |
| 03-2019    | 1136             |
| 04-2019    | 1099             |
| 05-2019    | 857              |
| 06-2019    | 824              |
| 07-2018    | 729              |
| 07-2019    | 864              |
| 08-2018    | 767              |
| 08-2019    | 1149             |
| 09-2018    | 780              |
| 10-2018    | 857              |
| 11-2018    | 928              |
| 12-2018    | 995              |

---

