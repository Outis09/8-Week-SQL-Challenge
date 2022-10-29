**Question 1:**
What day of the week is used for each week_date value?
-----
**Query:**

```sql
SELECT DISTINCT(to_char(week_date,'day')) as dow
  FROM clean_weekly_sales;
```

**Results:**

| dow       |
| --------- |
| monday    |

---------------------------------------------------

**Question 2:**
What range of week numbers are missing from the dataset?
-----

**Query:**

```sql
   SELECT array_agg(id) as missing_week_numbers
     FROM generate_series(1,52,1) gs(id)
LEFT JOIN clean_weekly_sales
       ON week_number = id
    WHERE week_number is null;
```

**Results:**

| missing_week_numbers                                                       |
| -------------------------------------------------------------------------- |
| 1,2,3,4,5,6,7,8,9,10,11,12,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52 |

---------------------------------------------------------------------------------
