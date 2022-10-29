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

**Results:**

| to_date                  | week_number | month_number | calendar_year | region        | platform | segment | age_band     | demographic | customer_type | transactions | sales    | avg_transaction |
| ------------------------ | ----------- | ------------ | ------------- | ------------- | -------- | ------- | ------------ | ----------- | ------------- | ------------ | -------- | --------------- |
| 2020-08-31     | 36          | 8            | 2020          | ASIA          | Retail   | C3      | Retirees     | Couples     | New           | 120631       | 3656163  | 30.31           |
| 2020-08-31     | 36          | 8            | 2020          | ASIA          | Retail   | F1      | Young Adults | Families    | New           | 31574        | 996575   | 31.56           |
| 2020-08-31     | 36          | 8            | 2020          | USA           | Retail   | unknown | unknown      | unknown     | Guest         | 529151       | 16509610 | 31.20           |
| 2020-08-31     | 36          | 8            | 2020          | EUROPE        | Retail   | C1      | Young Adults | Couples     | New           | 4517         | 141942   | 31.42           |
| 2020-08-31     | 36          | 8            | 2020          | AFRICA        | Retail   | C2      | Middle Aged  | Couples     | New           | 58046        | 1758388  | 30.29           |
| 2020-08-31     | 36          | 8            | 2020          | CANADA        | Shopify  | F2      | Middle Aged  | Families    | Existing      | 1336         | 243878   | 182.54          |
| 2020-08-31     | 36          | 8            | 2020          | AFRICA        | Shopify  | F3      | Retirees     | Families    | Existing      | 2514         | 519502   | 206.64          |
| 2020-08-31     | 36          | 8            | 2020          | ASIA          | Shopify  | F1      | Young Adults | Families    | Existing      | 2158         | 371417   | 172.11          |
| 2020-08-31     | 36          | 8            | 2020          | AFRICA        | Shopify  | F2      | Middle Aged  | Families    | New           | 318          | 49557    | 155.84          |
| 2020-08-31     | 36          | 8            | 2020          | AFRICA        | Retail   | C3      | Retirees     | Couples     | New           | 111032       | 3888162  | 35.02           |
| 2020-08-31     | 36          | 8            | 2020          | USA           | Shopify  | F1      | Young Adults | Families    | Existing      | 1398         | 260773   | 186.53          |
| 2020-08-31     | 36          | 8            | 2020          | OCEANIA       | Shopify  | C2      | Middle Aged  | Couples     | Existing      | 4661         | 882690   | 189.38          |
| 2020-08-31     | 36          | 8            | 2020          | SOUTH AMERICA | Retail   | C2      | Middle Aged  | Couples     | Existing      | 1029         | 38762    | 37.67           |
| 2020-08-31     | 36          | 8            | 2020          | SOUTH AMERICA | Shopify  | C4      | Retirees     | Couples     | New           | 6            | 917      | 152.83          |
| 2020-08-31     | 36          | 8            | 2020          | EUROPE        | Shopify  | F3      | Retirees     | Families    | Existing      | 115          | 35215    | 306.22          |
| 2020-08-31     | 36          | 8            | 2020          | OCEANIA       | Retail   | F3      | Retirees     | Families    | Existing      | 551905       | 30371770 | 55.03           |
| 2020-08-31     | 36          | 8            | 2020          | ASIA          | Shopify  | C3      | Retirees     | Couples     | Existing      | 1969         | 374327   | 190.11          |
| 2020-08-31     | 36          | 8            | 2020          | AFRICA        | Retail   | F1      | Young Adults | Families    | Existing      | 97604        | 5185233  | 53.13           |
| 2020-08-31     | 36          | 8            | 2020          | OCEANIA       | Retail   | C2      | Middle Aged  | Couples     | New           | 111219       | 2980673  | 26.80           |
| 2020-08-31     | 36          | 8            | 2020          | USA           | Retail   | F1      | Young Adults | Families    | New           | 11820        | 463738   | 39.23           |
	   
  
