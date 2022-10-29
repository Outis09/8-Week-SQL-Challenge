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

**Question 3:**
How many total transactions were there for each year in the dataset?
-----

**Query:**

```sql
  SELECT calendar_year as year,
         count(transactions) as num_of_txns
    FROM clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year;
```

**Results:**

| year | num_of_txns |
| ---- | ----------- |
| 2018 | 5698        |
| 2019 | 5708        |
| 2020 | 5711        |

---------------------------------------

**Question 4:**
What is the total sales for each region for each month?
-----

**Query:**

```sql
SELECT region,
       calendar_year,
	   to_char(week_date,'month') as month,
	   sum(sales) AS tot_sales
FROM clean_weekly_sales
GROUP BY region,calendar_year,month_number,month
ORDER BY region,calendar_year,month_number;
```

**Results:**

| region        | calendar_year | month     | tot_sales |
| ------------- | ------------- | --------- | --------- |
| AFRICA        | 2018          | march     | 130542213 |
| AFRICA        | 2018          | april     | 650194751 |
| AFRICA        | 2018          | may       | 522814997 |
| AFRICA        | 2018          | june      | 519127094 |
| AFRICA        | 2018          | july      | 674135866 |
| AFRICA        | 2018          | august    | 539077371 |
| AFRICA        | 2018          | september | 135084533 |
| AFRICA        | 2019          | march     | 141619349 |
| AFRICA        | 2019          | april     | 700447301 |
| AFRICA        | 2019          | may       | 553828220 |
| AFRICA        | 2019          | june      | 546092640 |
| AFRICA        | 2019          | july      | 711867600 |
| AFRICA        | 2019          | august    | 564497281 |
| AFRICA        | 2019          | september | 141236454 |
| AFRICA        | 2020          | march     | 295605918 |
| AFRICA        | 2020          | april     | 561141452 |
| AFRICA        | 2020          | may       | 570601521 |
| AFRICA        | 2020          | june      | 702340026 |
| AFRICA        | 2020          | july      | 574216244 |
| AFRICA        | 2020          | august    | 706022238 |
| ASIA          | 2018          | march     | 119180883 |
| ASIA          | 2018          | april     | 603716301 |
| ASIA          | 2018          | may       | 472634283 |

------------------------------------------

**Question 5:**
What is the total count of transactions for each platform
-----

**Query:**
```sql
SELECT platform,
       count(transactions)
FROM clean_weekly_sales
GROUP BY platform;
```

**Results:**

| platform | count |
| -------- | ----- |
| Shopify  | 8549  |
| Retail   | 8568  |

--------------------

