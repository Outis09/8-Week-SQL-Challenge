Generate a table that has 1 single row for every unique visit_id record and has the following columns:

* user_id
* visit_id
* visit_start_time: the earliest event_time for each visit
* page_views: count of page views for each visit
* cart_adds: count of product cart add events for each visit
* purchase: 1/0 flag if a purchase event exists for each visit
* campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
* impression: count of ad impressions for each visit
* click: count of ad clicks for each visit
* (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

**Query**
```sql
--gets the products added to cart ordered by seqience number
WITH sequence_products as (
	SELECT user_id,
	       visit_id,
	       string_agg(page_name,', ' ORDER BY sequence_number) as cart_products 
	  FROM events e
     LEFT JOIN page_hierarchy ph
         USING (page_id)
     LEFT JOIN users
         USING (cookie_id)
	 WHERE event_type = 2
      GROUP BY 1,2
      ORDER BY 1)

        SELECT u.user_id,
	       e.visit_id,
	       MIN(event_time) as visit_start_time,
	       sum(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) as page_views,
	       sum(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) as cart_addds,
	       CASE WHEN e.visit_id IN (SELECT DISTINCT visit_id FROM events WHERE event_type=3 ) THEN 1 ELSE 0 END AS purchase,
	       CASE WHEN MIN(event_time) BETWEEN ci.start_date AND ci.end_date THEN campaign_name END AS campaign_name,
	       sum(CASE WHEN event_type = 4 THEN 1 ELSE 0 END) as impression,
	       sum(CASE WHEN event_type = 5 THEN 1 ELSE 0 END) as click,
	       cart_products
	  INTO campaign_analysis
	  FROM events e 
	  JOIN users u
	 USING (cookie_id)
	  JOIN  campaign_identifier ci
           ON  e.event_time BETWEEN ci.start_date AND ci.end_date
     LEFT JOIN sequence_products sp
         USING e.visit_id = sp.visit_id
      GROUP BY 1,2,6,ci.start_date,ci.end_date,ci.campaign_name,sp.cart_products
      ORDER BY 1,2,3;
```

```sql
SELECT *
  FROM campaign_analysis;
```

**Results:**

| user_id | visit_id | visit_start_time         | page_views | cart_addds | purchase | campaign_name                     | impression | click | cart_products                                                                         |
| ------- | -------- | ------------------------ | ---------- | ---------- | -------- | --------------------------------- | ---------- | ----- | ------------------------------------------------------------------------------------- |
| 1       | 02a5d5   | 2020-02-26T16:57:26.260Z | 4          | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 1       | 0826dc   | 2020-02-26T05:58:37.918Z | 1          | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 1       | 0fc437   | 2020-02-04T17:49:49.602Z | 10         | 6          | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Tuna, Russian Caviar, Black Truffle, Abalone, Crab, Oyster                            |
| 1       | 30b94d   | 2020-03-15T13:12:54.023Z | 9          | 7          | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Abalone, Lobster, Crab                        |
| 1       | 41355d   | 2020-03-25T00:11:17.860Z | 6          | 1          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster                                                                               |
| 1       | ccf365   | 2020-02-04T19:16:09.182Z | 7          | 3          | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster, Crab, Oyster                                                                 |
| 1       | eaffde   | 2020-03-25T20:06:32.342Z | 10         | 8          | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster           |
| 1       | f7c798   | 2020-03-15T02:23:26.312Z | 9          | 3          | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Russian Caviar, Crab, Oyster                                                          |
| 2       | 0635fb   | 2020-02-16T06:42:42.735Z | 9          | 4          | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Salmon, Kingfish, Abalone, Crab                                                       |
| 2       | 1f1198   | 2020-02-01T21:51:55.078Z | 1          | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 2       | 3b5871   | 2020-01-18T10:16:32.158Z | 9          | 6          | 1        | 25% Off - Living The Lux Life     | 1          | 1     | Salmon, Kingfish, Russian Caviar, Black Truffle, Lobster, Oyster                      |
| 2       | 49d73d   | 2020-02-16T06:21:27.138Z | 11         | 9          | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster |
| 2       | 910d9a   | 2020-02-01T10:40:46.875Z | 8          | 1          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Abalone                                                                               |
| 2       | c5c0ee   | 2020-01-18T10:35:22.765Z | 1          | 0          | 0        | 25% Off - Living The Lux Life     | 0          | 0     |                                                                                       |
| 2       | d58cbd   | 2020-01-18T23:40:54.761Z | 8          | 4          | 0        | 25% Off - Living The Lux Life     | 0          | 0     | Kingfish, Tuna, Abalone, Crab                                                         |
| 2       | e26a84   | 2020-01-18T16:06:40.907Z | 6          | 2          | 1        | 25% Off - Living The Lux Life     | 0          | 0     | Salmon, Oyster                                                                        |
| 3       | 25502e   | 2020-02-21T11:26:15.353Z | 1          | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 3       | 9a2f24   | 2020-02-21T03:19:10.032Z | 6          | 2          | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Kingfish, Black Truffle                                                               |
| 3       | bf200a   | 2020-03-11T04:10:26.708Z | 7          | 2          | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Salmon, Crab                                                                          |
| 3       | eb13cd   | 2020-03-11T21:36:37.222Z | 1          | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 4       | 07a950   | 2020-03-19T17:56:24.610Z | 6          | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 4       | 4c0ce3   | 2020-02-22T19:42:45.498Z | 1          | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 4       | 7caba5   | 2020-02-22T17:49:37.646Z | 5          | 2          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Tuna, Lobster                                                                         |
| 4       | b90e25   | 2020-03-19T11:01:58.182Z | 9          | 4          | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Tuna, Black Truffle, Lobster, Crab                                                    |
| 5       | 05c52a   | 2020-02-11T12:30:33.479Z | 9          | 8          | 0        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab         |
| 5       | 4bffe1   | 2020-02-26T16:03:10.377Z | 8          | 1          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Russian Caviar                                                                        |
| 5       | 580bf6   | 2020-02-11T04:05:42.307Z | 8          | 6          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster                         |
| 5       | b45feb   | 2020-02-01T07:47:45.025Z | 1          | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 5       | f61ed7   | 2020-02-01T06:30:39.766Z | 8          | 2          | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster, Crab                                                                         |
| 5       | fa70cb   | 2020-02-26T11:12:20.361Z | 1          | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |

The full table contains over 3000 rows so I displayed the results for the first 5 user ids.

------------------------------------------------

**Analyzing Campaigns**
---------------------------
The campaigns did not last for an equal number of days. The first two lasted for 13 days and the last campaign lasted for 59 days. Therefore comparing total visits for the campaigns is not a good metric. So I will rather calculate the average visits in a day for each campaign and compare.

**Query:**
```sql
--gets the number of visits and days for each campaign
WITH visits as (
	SELECT ca.campaign_name,
	       count(visit_id) as num_of_visits,
	       ci.end_date - ci.start_date as camp_days 
	  FROM campaign_analysis ca
     LEFT JOIN campaign_identifier ci
            ON ca.campaign_name = ci.campaign_name
      GROUP BY ca.campaign_name,ci.end_date,ci.start_date)

  SELECT campaign_name,
         round(num_of_visits/LEFT(camp_days::varchar,2)::numeric) as avg_daily_visits
    FROM visits
ORDER BY avg_daily_visits DESC;
```
I used a CTE,`visits`, to get the number of visits and days for each campaign. In the main query, I divided the number of visits by the number of days to get the number of daily visits during each company.

**Results:**

|campign_name|avg_daily_visits|
|-----------|-------|
|Half Off - Treat Your Shellf(ish)|40|
|25% Off - Living The Lux Life|31|
|BOGOF - Fishing For Compliments|20|

The Half Off campaign had the highest daily visitors with an average of 40 visitors a day. This is followed by the 25% Off campaign which had 31 daily visits. The BOGOF had the least daily visitors with an average of 20 visits.
