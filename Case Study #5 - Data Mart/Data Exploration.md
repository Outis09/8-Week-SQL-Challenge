**Question 1:**
What day of the week is used for each week_date value?
-----
**Query:**

```sql
SELECT DISTINCT(to_char(week_date,'day')) as dow
  FROM clean_weekly_sales;
```
I used `to_char` to get the name of the day from `week_date`. Then I selected the distinct day name.

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
There are 52 weeks in a year so I generated a series from 1 to 52. Using `LEFT JOIN`, I joined the `clean_weekly_sales` table. For numbers on the generated series that were not in the `week_number` column, the left join will return null so i filtered for that. In the select statement, I converted the results to an array.

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
I selected the calendar year and counted the transactions which I grouped by the calendar year.

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
         calendar_year as year,
	 to_char(week_date,'month') as month,
	 sum(sales) AS tot_sales
    FROM clean_weekly_sales
GROUP BY region,calendar_year,month_number,month
ORDER BY region,calendar_year,month_number;
```
I selected the regions, the calendar year, month names by using `to_char` and the sum of sales. then I grouped by region, year and month.

**Results:**

| region        | year | month     | tot_sales |
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

The full results contained 140 rows so I displayed only the first few rows here.

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

**Question 6:**
What is the percentage of sales for Retail vs Shopify for each month?
-----

**Query:**

```sql
--gets total sales,total retail sales and total shopify sales
WITH platform_sales as(
SELECT calendar_year as year,
       to_char(week_date,'month') as month,
	   sum(sales) as monthly_sales,
	   sum(CASE WHEN platform = 'Retail' THEN sales ELSE 0 end)as retail_sales,
	   sum(CASE WHEN platform = 'Shopify' THEN sales ELSE 0 end) as shopify_sales
FROM clean_weekly_sales
GROUP BY 1,month_number,2
ORDER BY 1,month_number)

SELECT year,
       month,
	   round(((retail_sales::numeric/monthly_sales::numeric)*100),2) as retail_sales_percentage,
	   round(((shopify_sales::numeric/monthly_sales::numeric)*100),2) as shopify_sales_percentage
FROM platform_sales;
```

**Results:**

| year | month     | retail_sales_percentage | shopify_sales_percentage |
| ---- | --------- | ----------------------- | ------------------------ |
| 2018 | march     | 97.92                   | 2.08                     |
| 2018 | april     | 97.93                   | 2.07                     |
| 2018 | may       | 97.73                   | 2.27                     |
| 2018 | june      | 97.76                   | 2.24                     |
| 2018 | july      | 97.75                   | 2.25                     |
| 2018 | august    | 97.71                   | 2.29                     |
| 2018 | september | 97.68                   | 2.32                     |
| 2019 | march     | 97.71                   | 2.29                     |
| 2019 | april     | 97.80                   | 2.20                     |
| 2019 | may       | 97.52                   | 2.48                     |
| 2019 | june      | 97.42                   | 2.58                     |
| 2019 | july      | 97.35                   | 2.65                     |
| 2019 | august    | 97.21                   | 2.79                     |
| 2019 | september | 97.09                   | 2.91                     |
| 2020 | march     | 97.30                   | 2.70                     |
| 2020 | april     | 96.96                   | 3.04                     |
| 2020 | may       | 96.71                   | 3.29                     |
| 2020 | june      | 96.80                   | 3.20                     |
| 2020 | july      | 96.67                   | 3.33                     |
| 2020 | august    | 96.51                   | 3.49                     |

------------------------------------------

**Question 7**
What is the percentage of sales by demographic for each year in the dataset?
-----

**Query:**

```sql
WITH demo_sales as (
SELECT calendar_year as year,
       sum(sales) as total_sales,
	   sum(CASE WHEN demographic = 'Couples' THEN sales ELSE 0 END)as couples_sales,
	   sum(CASE WHEN demographic = 'Families' THEN sales ELSE 0 END) as families_sales,
	   sum(CASE WHEN demographic = 'unknown' THEN sales ELSE 0 END) as unknown_demo_sales
FROM clean_weekly_sales
GROUP BY 1
					)
SELECT year,
       round(((couples_sales::numeric/total_sales::numeric)*100),2) as couples_sales_percent,
	   round(((families_sales::numeric/total_sales::numeric)*100),2) as fam_sales_percent,
	   round(((unknown_demo_sales::numeric/total_sales::numeric)*100),2) as unknown_sales_percent
FROM demo_sales
ORDER BY year;
```

**Results:**

| year | couples_sales_percent | fam_sales_percent | unknown_sales_percent |
| ---- | --------------------- | ----------------- | --------------------- |
| 2018 | 26.38                 | 31.99             | 41.63                 |
| 2019 | 27.28                 | 32.47             | 40.25                 |
| 2020 | 28.72                 | 32.73             | 38.55                 |

----------------------

**Question 8:**
Which age_band and demographic values contribute the most to Retail sales?
------

**Query:**

```sql
SELECT age_band,
       demographic,
       sum(sales) as retail_sales,
       round(((sum(sales)::numeric/(SELECT sum(sales) FROM clean_weekly_sales WHERE platform='Retail')::numeric)*100),2) as percentage
FROM clean_weekly_sales
WHERE platform = 'Retail'
GROUP BY age_band, demographic
ORDER BY retail_sales DESC;
```

**Results:**

| age_band     | demographic | retail_sales | percentage |
| ------------ | ----------- | ------------ | ---------- |
| unknown      | unknown     | 16067285533  | 40.52      |
| Retirees     | Families    | 6634686916   | 16.73      |
| Retirees     | Couples     | 6370580014   | 16.07      |
| Middle Aged  | Families    | 4354091554   | 10.98      |
| Young Adults | Couples     | 2602922797   | 6.56       |
| Middle Aged  | Couples     | 1854160330   | 4.68       |
| Young Adults | Families    | 1770889293   | 4.47       |

------------------------------------

**Question 9:**
Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
-----

**Query:**

```sql
SELECT calendar_year,
       platform,
	   round(avg(avg_transaction),2) as avg_transaction_row,
	   round((sum(sales)::numeric/sum(transactions)::numeric),2) as avg_transaction_group
FROM clean_weekly_sales
GROUP BY calendar_year,platform
ORDER BY calendar_year;
```

**Results:**

| calendar_year | platform | avg_transaction_row | avg_transaction_group |
| ------------- | -------- | ------------------- | --------------------- |
| 2018          | Retail   | 42.91               | 36.56                 |
| 2018          | Shopify  | 188.28              | 192.48                |
| 2019          | Retail   | 41.97               | 36.83                 |
| 2019          | Shopify  | 177.56              | 183.36                |
| 2020          | Shopify  | 174.87              | 179.03                |
| 2020          | Retail   | 40.64               | 36.56                 |

-------------------------------
