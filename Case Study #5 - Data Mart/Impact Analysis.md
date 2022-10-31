Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

* region
* platform
* age_band
* demographic
* customer_type


Region
-----

**Query:**
```sql
WITH region_sales as (
SELECT region,
       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	   sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
FROM clean_weekly_sales
GROUP BY region)

SELECT region,
       tot_sales_bfr,
	   tot_sales_aft,
	   round(((tot_sales_aft::numeric-tot_sales_bfr::numeric)/tot_sales_bfr::numeric)*100,2) as percent_change
FROM region_sales
ORDER BY percent_change;
```

**Results:**
| region        | tot_sales_bfr | tot_sales_aft | percent_change |
| ------------- | ------------- | ------------- | -------------- |
| ASIA          | 4613242689    | 4551927271    | -1.33          |
| OCEANIA       | 6698586333    | 6640244793    | -0.87          |
| CANADA        | 1244662705    | 1234025206    | -0.85          |
| USA           | 1967554887    | 1960297502    | -0.37          |
| SOUTH AMERICA | 611056923     | 608981392     | -0.34          |
| AFRICA        | 4942976910    | 4997516159    | 1.10           |
| EUROPE        | 328141414     | 344420043     | 4.96           |

---------------------------
