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

Platform
-----

**Query:**
```sql
WITH platform_sales as (
SELECT platform,
       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	   sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
FROM clean_weekly_sales
GROUP BY platform)

SELECT platform,
       tot_sales_bfr,
	   tot_sales_aft,
	   round(((tot_sales_aft::numeric-tot_sales_bfr::numeric)/tot_sales_bfr::numeric)*100,2) as percent_change
FROM platform_sales
ORDER BY percent_change;
```

**Results:**
| platform | tot_sales_bfr | tot_sales_aft | percent_change |
| -------- | ------------- | ------------- | -------------- |
| Retail   | 19886040272   | 19768576165   | -0.59          |
| Shopify  | 520181589     | 568836201     | 9.35           |

----------------------------

Age Band
---------

**Query:**
```sql
WITH age_band_sales as (
SELECT age_band,
       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	   sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
FROM clean_weekly_sales
GROUP BY age_band)

SELECT age_band,
       tot_sales_bfr,
	   tot_sales_aft,
	   round(((tot_sales_aft::numeric-tot_sales_bfr::numeric)/tot_sales_bfr::numeric)*100,2) as percent_change
FROM age_band_sales
ORDER BY percent_change;
```

**Results:**
| age_band     | tot_sales_bfr | tot_sales_aft | percent_change |
| ------------ | ------------- | ------------- | -------------- |
| unknown      | 8191628826    | 8146983408    | -0.55          |
| Middle Aged  | 3276892347    | 3269748622    | -0.22          |
| Young Adults | 2290835366    | 2285973456    | -0.21          |
| Retirees     | 6646865322    | 6634706880    | -0.18          |

-----------------------------

Demographic
-----

**Query:**

```sql
WITH demo_sales as (
SELECT demographic,
       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	   sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
FROM clean_weekly_sales
GROUP BY demographic)

SELECT demographic,
       tot_sales_bfr,
	   tot_sales_aft,
	   round(((tot_sales_aft::numeric-tot_sales_bfr::numeric)/tot_sales_bfr::numeric)*100,2) as percent_change
FROM demo_sales
ORDER BY percent_change;
```

**Results:**
| demographic | tot_sales_bfr | tot_sales_aft | percent_change |
| ----------- | ------------- | ------------- | -------------- |
| unknown     | 8191628826    | 8146983408    | -0.55          |
| Couples     | 5608866131    | 5592341420    | -0.29          |
| Families    | 6605726904    | 6598087538    | -0.12          |

-------------

Customer Type
-----

**Query:**
```sql
WITH customer_sales as (
SELECT customer_type,
       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	   sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
FROM clean_weekly_sales
GROUP BY customer_type)

SELECT customer_type,
       tot_sales_bfr,
	   tot_sales_aft,
	   round(((tot_sales_aft::numeric-tot_sales_bfr::numeric)/tot_sales_bfr::numeric)*100,2) as percent_change
FROM customer_sales
ORDER BY percent_change;
```

**Results:**

| customer_type | tot_sales_bfr | tot_sales_aft | percent_change |
| ------------- | ------------- | ------------- | -------------- |
| Existing      | 10168877642   | 10117367239   | -0.51          |
| Guest         | 7630353739    | 7595150744    | -0.46          |
| New           | 2606990480    | 2624894383    | 0.69           |

---------------------------
