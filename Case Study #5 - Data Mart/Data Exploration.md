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
