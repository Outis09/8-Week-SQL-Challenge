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
--gets total sales for the 12 weeks before and after the change for each region
WITH region_sales as (
	SELECT region,
	       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	       sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
	  FROM clean_weekly_sales
      GROUP BY region
                     )

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
--gets total sales before and after the change on each Platform
WITH platform_sales as (
	SELECT platform,
	       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	       sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
	  FROM clean_weekly_sales
      GROUP BY platform
                        )

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
--gets total sales before and after the change for each age band
WITH age_band_sales as (
	SELECT age_band,
	       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	       sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
	  FROM clean_weekly_sales
      GROUP BY age_band
                        )

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
--gets total sales before and after the change for each demographic
WITH demo_sales as (
	SELECT demographic,
	       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	       sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
	  FROM clean_weekly_sales
      GROUP BY demographic
                    )

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
--gets total sales before and after the change for each customer type
WITH customer_sales as (
	SELECT customer_type,
	       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	       sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
	  FROM clean_weekly_sales
      GROUP BY customer_type
                        )

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

More detailed impact analysis
----

**Query:**
```sql
WITH overall_impact as (
	SELECT region,
	       platform,
	       age_band,
	       demographic,
	       customer_type,
	       sum(CASE WHEN week_number between 13 and 24 then sales else 0 end) as tot_sales_bfr,
	       sum(CASE WHEN week_number between 25 and 36 then sales else 0 end) as tot_sales_aft
	  FROM clean_weekly_sales
      GROUP BY region,platform,age_band,demographic,customer_type
                      )

  SELECT region,
         platform,
	 age_band,
	 demographic,
	 customer_type,
	 tot_sales_bfr,
	 tot_sales_aft,
	 round(((tot_sales_aft::numeric-tot_sales_bfr::numeric)/tot_sales_bfr::numeric)*100,2) as percent_change
    FROM overall_impact
ORDER BY percent_change;
```

**Results:**
| region        | platform | age_band     | demographic | customer_type | tot_sales_bfr | tot_sales_aft | percent_change |
| ------------- | -------- | ------------ | ----------- | ------------- | ------------- | ------------- | -------------- |
| SOUTH AMERICA | Retail   | unknown      | unknown     | Existing      | 609879        | 456009        | -25.23         |
| SOUTH AMERICA | Retail   | Retirees     | Families    | New           | 506115        | 418849        | -17.24         |
| SOUTH AMERICA | Retail   | Retirees     | Couples     | New           | 1728704       | 1443074       | -16.52         |
| SOUTH AMERICA | Shopify  | Retirees     | Families    | New           | 16159         | 13809         | -14.54         |
| SOUTH AMERICA | Retail   | Retirees     | Couples     | Existing      | 2850110       | 2441740       | -14.33         |
| SOUTH AMERICA | Retail   | Middle Aged  | Families    | New           | 587106        | 546593        | -6.90          |
| OCEANIA       | Retail   | unknown      | unknown     | Existing      | 64639774      | 60489208      | -6.42          |
| SOUTH AMERICA | Retail   | unknown      | unknown     | New           | 2278628       | 2132544       | -6.41          |
| ASIA          | Retail   | unknown      | unknown     | Existing      | 44046987      | 41414914      | -5.98          |
| SOUTH AMERICA | Retail   | Middle Aged  | Couples     | New           | 867493        | 826106        | -4.77          |
| AFRICA        | Retail   | unknown      | unknown     | Existing      | 51046331      | 48692221      | -4.61          |
| SOUTH AMERICA | Retail   | Retirees     | Families    | Existing      | 2137107       | 2052881       | -3.94          |
| SOUTH AMERICA | Retail   | Young Adults | Families    | New           | 385293        | 370786        | -3.77          |
| SOUTH AMERICA | Retail   | Middle Aged  | Couples     | Existing      | 1179377       | 1136223       | -3.66          |
| ASIA          | Retail   | Retirees     | Couples     | Existing      | 530994020     | 514415643     | -3.12          |
| OCEANIA       | Retail   | Retirees     | Couples     | Existing      | 808760538     | 790476515     | -2.26          |
| SOUTH AMERICA | Retail   | Middle Aged  | Families    | Existing      | 1857659       | 1820469       | -2.00          |
| ASIA          | Retail   | unknown      | unknown     | Guest         | 1775661158    | 1740139393    | -2.00          |
| OCEANIA       | Retail   | Young Adults | Families    | Existing      | 260629668     | 255435893     | -1.99          |
| CANADA        | Retail   | Retirees     | Couples     | New           | 49089251      | 48142714      | -1.93          |
| USA           | Retail   | Young Adults | Families    | Existing      | 82781745      | 81190833      | -1.92          |
| ASIA          | Retail   | Young Adults | Families    | Existing      | 157677724     | 154662536     | -1.91          |
| USA           | Retail   | Middle Aged  | Families    | Existing      | 196112024     | 192378351     | -1.90          |
| CANADA        | Retail   | unknown      | unknown     | New           | 23030516      | 22650948      | -1.65          |
| CANADA        | Retail   | Retirees     | Couples     | Existing      | 153404993     | 150899981     | -1.63          |
| EUROPE        | Shopify  | Retirees     | Families    | Existing      | 1052355       | 1035608       | -1.59          |
| CANADA        | Retail   | Young Adults | Families    | Existing      | 47131088      | 46394112      | -1.56          |
| CANADA        | Retail   | unknown      | unknown     | Guest         | 423154544     | 416816837     | -1.50          |
| CANADA        | Retail   | Middle Aged  | Families    | New           | 22426694      | 22093589      | -1.49          |
| OCEANIA       | Retail   | Middle Aged  | Couples     | Existing      | 213825220     | 210688314     | -1.47          |
| CANADA        | Retail   | Young Adults | Families    | New           | 9370393       | 9239332       | -1.40          |
| OCEANIA       | Retail   | Young Adults | Couples     | Existing      | 310991546     | 306815447     | -1.34          |
| ASIA          | Retail   | Middle Aged  | Families    | Existing      | 365269341     | 360586807     | -1.28          |
| OCEANIA       | Retail   | Middle Aged  | Families    | Existing      | 627202943     | 619362101     | -1.25          |
| ASIA          | Retail   | Young Adults | Couples     | Existing      | 201182062     | 198732892     | -1.22          |
| OCEANIA       | Retail   | unknown      | unknown     | New           | 113173781     | 111832204     | -1.19          |
| OCEANIA       | Retail   | unknown      | unknown     | Guest         | 2339877889    | 2312766536    | -1.16          |
| USA           | Retail   | unknown      | unknown     | Existing      | 20129299      | 19900411      | -1.14          |
| CANADA        | Retail   | Young Adults | Couples     | Existing      | 82227474      | 81303656      | -1.12          |
| CANADA        | Retail   | Middle Aged  | Couples     | Existing      | 32983055      | 32615541      | -1.11          |
| CANADA        | Retail   | Young Adults | Couples     | New           | 28447732      | 28151202      | -1.04          |
| ASIA          | Retail   | Middle Aged  | Couples     | Existing      | 156499725     | 154904713     | -1.02          |
| ASIA          | Retail   | Retirees     | Families    | Existing      | 667276596     | 660971431     | -0.94          |
| USA           | Retail   | Young Adults | Couples     | Existing      | 82496435      | 81746129      | -0.91          |
| USA           | Retail   | Retirees     | Couples     | Existing      | 262402128     | 260372752     | -0.77          |
| USA           | Retail   | unknown      | unknown     | Guest         | 625697241     | 620864831     | -0.77          |
| USA           | Retail   | Middle Aged  | Couples     | New           | 30555060      | 30344320      | -0.69          |
| OCEANIA       | Retail   | Middle Aged  | Families    | New           | 110041595     | 109282211     | -0.69          |
| ASIA          | Retail   | Middle Aged  | Families    | New           | 77518015      | 77020890      | -0.64          |
| OCEANIA       | Retail   | Retirees     | Families    | Existing      | 1016091505    | 1009706233    | -0.63          |
| CANADA        | Retail   | Retirees     | Families    | New           | 23542227      | 23396299      | -0.62          |
| AFRICA        | Retail   | Young Adults | Families    | Existing      | 185860538     | 184761609     | -0.59          |
| CANADA        | Retail   | unknown      | unknown     | Existing      | 11933842      | 11866524      | -0.56          |
| OCEANIA       | Retail   | Young Adults | Families    | New           | 49069024      | 48798359      | -0.55          |
| ASIA          | Retail   | Retirees     | Couples     | New           | 173745585     | 172895594     | -0.49          |
| CANADA        | Retail   | Middle Aged  | Families    | Existing      | 114398815     | 113886975     | -0.45          |
| ASIA          | Retail   | Young Adults | Families    | New           | 34914806      | 34763017      | -0.43          |
| USA           | Retail   | Middle Aged  | Families    | New           | 37855947      | 37695652      | -0.42          |
| SOUTH AMERICA | Retail   | unknown      | unknown     | Guest         | 578964403     | 576643105     | -0.40          |
| USA           | Retail   | Middle Aged  | Couples     | Existing      | 69070814      | 68811945      | -0.37          |
| SOUTH AMERICA | Retail   | Young Adults | Families    | Existing      | 1102740       | 1098853       | -0.35          |
| USA           | Retail   | Young Adults | Families    | New           | 16488501      | 16435759      | -0.32          |
| ASIA          | Retail   | unknown      | unknown     | New           | 82789550      | 82534151      | -0.31          |
| EUROPE        | Retail   | Middle Aged  | Families    | Existing      | 30908363      | 30834464      | -0.24          |
| USA           | Retail   | Retirees     | Families    | Existing      | 291926709     | 291248150     | -0.23          |
| AFRICA        | Retail   | Middle Aged  | Couples     | New           | 59821149      | 59742497      | -0.13          |
| AFRICA        | Retail   | Young Adults | Families    | New           | 30347873      | 30318272      | -0.10          |
| EUROPE        | Retail   | Young Adults | Families    | Existing      | 14391142      | 14383305      | -0.05          |
| OCEANIA       | Retail   | Retirees     | Couples     | New           | 231720270     | 231646054     | -0.03          |
| AFRICA        | Retail   | Middle Aged  | Families    | Existing      | 517883517     | 518043552     | 0.03           |
| CANADA        | Retail   | Retirees     | Families    | Existing      | 175730918     | 176000858     | 0.15           |
| AFRICA        | Retail   | unknown      | unknown     | New           | 88568584      | 88728186      | 0.18           |
| AFRICA        | Retail   | Middle Aged  | Couples     | Existing      | 158118057     | 158450244     | 0.21           |
| SOUTH AMERICA | Retail   | Young Adults | Couples     | Existing      | 2743800       | 2752136       | 0.30           |
| OCEANIA       | Retail   | Middle Aged  | Couples     | New           | 97190076      | 97510743      | 0.33           |
| CANADA        | Retail   | Middle Aged  | Couples     | New           | 15623979      | 15689592      | 0.42           |
| AFRICA        | Retail   | Young Adults | Couples     | Existing      | 230192757     | 231177566     | 0.43           |
| ASIA          | Retail   | Retirees     | Families    | New           | 94702143      | 95142214      | 0.46           |
| ASIA          | Retail   | Middle Aged  | Couples     | New           | 74098189      | 74515346      | 0.56           |
| EUROPE        | Retail   | Young Adults | Couples     | Existing      | 23061250      | 23199911      | 0.60           |
| USA           | Retail   | unknown      | unknown     | New           | 35198248      | 35439608      | 0.69           |
| AFRICA        | Retail   | Middle Aged  | Families    | New           | 81278572      | 81853804      | 0.71           |
| EUROPE        | Retail   | Middle Aged  | Couples     | Existing      | 15929501      | 16061845      | 0.83           |
| AFRICA        | Retail   | Retirees     | Couples     | Existing      | 676223054     | 683144645     | 1.02           |
| USA           | Retail   | Retirees     | Families    | New           | 36678806      | 37095037      | 1.13           |
| AFRICA        | Retail   | unknown      | unknown     | Guest         | 1645007794    | 1663747923    | 1.14           |
| OCEANIA       | Retail   | Retirees     | Families    | New           | 125957219     | 127405669     | 1.15           |
| USA           | Retail   | Young Adults | Couples     | New           | 34013222      | 34430982      | 1.23           |
| AFRICA        | Retail   | Retirees     | Families    | Existing      | 757446611     | 766906475     | 1.25           |
| EUROPE        | Retail   | unknown      | unknown     | Existing      | 4522840       | 4579975       | 1.26           |
| SOUTH AMERICA | Shopify  | Young Adults | Couples     | New           | 88498         | 89629         | 1.28           |
| ASIA          | Retail   | Young Adults | Couples     | New           | 92536334      | 93958087      | 1.54           |
| OCEANIA       | Retail   | Young Adults | Couples     | New           | 126824379     | 128822307     | 1.58           |
| USA           | Retail   | Retirees     | Couples     | New           | 81219398      | 82531106      | 1.62           |
| AFRICA        | Retail   | Young Adults | Couples     | New           | 81340508      | 82684259      | 1.65           |
| EUROPE        | Retail   | Retirees     | Families    | Existing      | 38977265      | 39735155      | 1.94           |
| SOUTH AMERICA | Shopify  | Retirees     | Couples     | New           | 149191        | 152441        | 2.18           |
| CANADA        | Shopify  | unknown      | unknown     | Existing      | 436570        | 446532        | 2.28           |
| CANADA        | Shopify  | Middle Aged  | Couples     | New           | 437214        | 448220        | 2.52           |
| AFRICA        | Retail   | Retirees     | Families    | New           | 83057127      | 85410945      | 2.83           |
| SOUTH AMERICA | Retail   | Young Adults | Couples     | New           | 1741041       | 1796294       | 3.17           |
| AFRICA        | Retail   | Retirees     | Couples     | New           | 181364756     | 187111330     | 3.17           |
| EUROPE        | Shopify  | Middle Aged  | Families    | New           | 60772         | 62947         | 3.58           |
| EUROPE        | Retail   | unknown      | unknown     | New           | 5251479       | 5451671       | 3.81           |
| USA           | Shopify  | Retirees     | Couples     | New           | 1010992       | 1051122       | 3.97           |
| OCEANIA       | Shopify  | unknown      | unknown     | Existing      | 3023515       | 3145887       | 4.05           |
| AFRICA        | Shopify  | Retirees     | Families    | New           | 546748        | 569784        | 4.21           |
| USA           | Shopify  | Retirees     | Couples     | Existing      | 9391176       | 9863689       | 5.03           |
| USA           | Shopify  | Retirees     | Families    | Existing      | 6214792       | 6547462       | 5.35           |
| AFRICA        | Shopify  | Middle Aged  | Families    | New           | 1302157       | 1374706       | 5.57           |
| EUROPE        | Shopify  | Retirees     | Couples     | New           | 89359         | 94332         | 5.57           |
| EUROPE        | Shopify  | Young Adults | Couples     | Existing      | 661484        | 698799        | 5.64           |
| OCEANIA       | Shopify  | Middle Aged  | Couples     | Existing      | 23227723      | 24543795      | 5.67           |
| EUROPE        | Shopify  | Middle Aged  | Couples     | Existing      | 1279372       | 1352198       | 5.69           |
| EUROPE        | Shopify  | Middle Aged  | Families    | Existing      | 1492147       | 1577586       | 5.73           |
| CANADA        | Shopify  | Retirees     | Couples     | New           | 470910        | 498078        | 5.77           |
| USA           | Shopify  | Middle Aged  | Families    | New           | 811812        | 859109        | 5.83           |
| EUROPE        | Shopify  | Middle Aged  | Couples     | New           | 86317         | 91478         | 5.98           |
| OCEANIA       | Shopify  | Retirees     | Couples     | Existing      | 27609962      | 29278946      | 6.04           |
| EUROPE        | Shopify  | Retirees     | Families    | New           | 19581         | 20772         | 6.08           |
| USA           | Shopify  | Young Adults | Couples     | New           | 641631        | 682520        | 6.37           |
| SOUTH AMERICA | Shopify  | Retirees     | Couples     | Existing      | 606926        | 646565        | 6.53           |
| CANADA        | Shopify  | Middle Aged  | Couples     | Existing      | 3207968       | 3419150       | 6.58           |
| AFRICA        | Shopify  | Retirees     | Couples     | Existing      | 15983040      | 17039490      | 6.61           |
| CANADA        | Shopify  | Retirees     | Couples     | Existing      | 4257470       | 4540930       | 6.66           |
| CANADA        | Shopify  | Retirees     | Families    | Existing      | 3392591       | 3619709       | 6.69           |
| USA           | Shopify  | Middle Aged  | Families    | Existing      | 11919775      | 12731248      | 6.81           |
| EUROPE        | Retail   | unknown      | unknown     | Guest         | 127899706     | 136655861     | 6.85           |
| OCEANIA       | Shopify  | Retirees     | Couples     | New           | 2784904       | 2977003       | 6.90           |
| EUROPE        | Retail   | Young Adults | Couples     | New           | 4616369       | 4937020       | 6.95           |
| CANADA        | Shopify  | Retirees     | Families    | New           | 153129        | 163764        | 6.95           |
| ASIA          | Shopify  | Retirees     | Couples     | New           | 1293007       | 1383676       | 7.01           |
| CANADA        | Shopify  | Young Adults | Couples     | New           | 331622        | 355115        | 7.08           |
| SOUTH AMERICA | Shopify  | Young Adults | Couples     | Existing      | 347200        | 371918        | 7.12           |
| AFRICA        | Shopify  | Young Adults | Couples     | New           | 968647        | 1037712       | 7.13           |
| USA           | Shopify  | Young Adults | Families    | New           | 652300        | 700012        | 7.31           |
| SOUTH AMERICA | Shopify  | Middle Aged  | Couples     | Existing      | 713073        | 765241        | 7.32           |
| EUROPE        | Retail   | Middle Aged  | Couples     | New           | 3422831       | 3678375       | 7.47           |
| USA           | Shopify  | Young Adults | Families    | Existing      | 7849042       | 8435701       | 7.47           |
| USA           | Shopify  | Young Adults | Couples     | Existing      | 4054128       | 4357418       | 7.48           |
| AFRICA        | Shopify  | Retirees     | Couples     | New           | 1619557       | 1741265       | 7.51           |
| CANADA        | Shopify  | Young Adults | Families    | New           | 298191        | 321274        | 7.74           |
| OCEANIA       | Shopify  | Young Adults | Couples     | Existing      | 12726172      | 13715298      | 7.77           |
| EUROPE        | Retail   | Retirees     | Couples     | Existing      | 36284156      | 39171680      | 7.96           |
| AFRICA        | Shopify  | Young Adults | Couples     | Existing      | 7117333       | 7690852       | 8.06           |
| EUROPE        | Retail   | Young Adults | Families    | New           | 1387112       | 1498980       | 8.06           |
| EUROPE        | Shopify  | Retirees     | Couples     | Existing      | 1454010       | 1572516       | 8.15           |
| OCEANIA       | Shopify  | Retirees     | Families    | Existing      | 20727122      | 22416589      | 8.15           |
| USA           | Shopify  | Middle Aged  | Couples     | Existing      | 7672945       | 8298978       | 8.16           |
| CANADA        | Shopify  | Middle Aged  | Families    | New           | 402421        | 435279        | 8.17           |
| CANADA        | Shopify  | Young Adults | Couples     | Existing      | 2339490       | 2530552       | 8.17           |
| ASIA          | Shopify  | Retirees     | Couples     | Existing      | 11019481      | 11955018      | 8.49           |
| AFRICA        | Shopify  | Middle Aged  | Couples     | New           | 1441150       | 1565664       | 8.64           |
| AFRICA        | Shopify  | Retirees     | Families    | Existing      | 12982215      | 14116121      | 8.73           |
| SOUTH AMERICA | Shopify  | Young Adults | Families    | Existing      | 386990        | 420850        | 8.75           |
| EUROPE        | Shopify  | Young Adults | Families    | Existing      | 926664        | 1010496       | 9.05           |
| OCEANIA       | Shopify  | Young Adults | Families    | Existing      | 21853180      | 23832030      | 9.06           |
| OCEANIA       | Shopify  | unknown      | unknown     | New           | 2433650       | 2656395       | 9.15           |
| OCEANIA       | Shopify  | Middle Aged  | Families    | Existing      | 34259519      | 37398479      | 9.16           |
| CANADA        | Shopify  | Middle Aged  | Families    | Existing      | 6375563       | 6964887       | 9.24           |
| ASIA          | Shopify  | Middle Aged  | Couples     | Existing      | 9869565       | 10781927      | 9.24           |
| OCEANIA       | Shopify  | Young Adults | Couples     | New           | 1638633       | 1790951       | 9.30           |
| EUROPE        | Shopify  | Young Adults | Families    | New           | 41246         | 45119         | 9.39           |
| OCEANIA       | Shopify  | Young Adults | Families    | New           | 1527848       | 1672927       | 9.50           |
| OCEANIA       | Shopify  | unknown      | unknown     | Guest         | 44847164      | 49138238      | 9.57           |
| SOUTH AMERICA | Shopify  | Middle Aged  | Couples     | New           | 210578        | 230771        | 9.59           |
| AFRICA        | Shopify  | Middle Aged  | Couples     | Existing      | 11961976      | 13132147      | 9.78           |
| EUROPE        | Shopify  | unknown      | unknown     | Guest         | 1775045       | 1949239       | 9.81           |
| AFRICA        | Shopify  | Young Adults | Families    | New           | 812378        | 892657        | 9.88           |
| USA           | Shopify  | unknown      | unknown     | Guest         | 11713657      | 12880033      | 9.96           |
| OCEANIA       | Shopify  | Middle Aged  | Couples     | New           | 3004957       | 3307291       | 10.06          |
| CANADA        | Shopify  | Young Adults | Families    | Existing      | 3556087       | 3916466       | 10.13          |
| CANADA        | Shopify  | unknown      | unknown     | Guest         | 6076453       | 6729767       | 10.75          |
| AFRICA        | Shopify  | Middle Aged  | Families    | Existing      | 23233355      | 25737984      | 10.78          |
| USA           | Shopify  | Retirees     | Families    | New           | 272355        | 302084        | 10.92          |
| AFRICA        | Shopify  | Young Adults | Families    | Existing      | 12424216      | 13842795      | 11.42          |
| ASIA          | Shopify  | Middle Aged  | Couples     | New           | 1435089       | 1599838       | 11.48          |
| AFRICA        | Shopify  | unknown      | unknown     | Guest         | 22118705      | 24670235      | 11.54          |
| ASIA          | Shopify  | Retirees     | Families    | New           | 437320        | 489446        | 11.92          |
| AFRICA        | Shopify  | unknown      | unknown     | Existing      | 1538823       | 1722673       | 11.95          |
| ASIA          | Shopify  | Young Adults | Couples     | Existing      | 4961648       | 5562055       | 12.10          |
| USA           | Shopify  | unknown      | unknown     | New           | 815655        | 914319        | 12.10          |
| ASIA          | Shopify  | Middle Aged  | Families    | New           | 938958        | 1052861       | 12.13          |
| ASIA          | Shopify  | Retirees     | Families    | Existing      | 8656392       | 9734203       | 12.45          |
| ASIA          | Shopify  | Young Adults | Families    | New           | 761864        | 858475        | 12.68          |
| SOUTH AMERICA | Shopify  | unknown      | unknown     | New           | 287010        | 323608        | 12.75          |
| CANADA        | Shopify  | unknown      | unknown     | New           | 431505        | 487323        | 12.94          |
| ASIA          | Shopify  | Middle Aged  | Families    | Existing      | 13201609      | 14968923      | 13.39          |
| ASIA          | Shopify  | Young Adults | Families    | Existing      | 8989650       | 10218558      | 13.67          |
| USA           | Shopify  | Middle Aged  | Couples     | New           | 1055022       | 1199374       | 13.68          |
| OCEANIA       | Shopify  | Retirees     | Families    | New           | 890879        | 1012883       | 13.69          |
| OCEANIA       | Shopify  | Middle Aged  | Families    | New           | 2035678       | 2320287       | 13.98          |
| ASIA          | Shopify  | Young Adults | Couples     | New           | 824215        | 943401        | 14.46          |
| EUROPE        | Retail   | Retirees     | Families    | New           | 2407327       | 2759045       | 14.61          |
| SOUTH AMERICA | Shopify  | unknown      | unknown     | Guest         | 8164166       | 9390622       | 15.02          |
| EUROPE        | Retail   | Middle Aged  | Families    | New           | 2485626       | 2859879       | 15.06          |
| ASIA          | Shopify  | unknown      | unknown     | Existing      | 1323144       | 1522610       | 15.08          |
| USA           | Shopify  | unknown      | unknown     | Existing      | 854028        | 988567        | 15.75          |
| SOUTH AMERICA | Shopify  | unknown      | unknown     | Existing      | 97372         | 112852        | 15.90          |
| SOUTH AMERICA | Shopify  | Middle Aged  | Families    | New           | 46844         | 54538         | 16.42          |
| EUROPE        | Shopify  | Young Adults | Couples     | New           | 66236         | 77176         | 16.52          |
| SOUTH AMERICA | Shopify  | Retirees     | Families    | Existing      | 169199        | 197145        | 16.52          |
| ASIA          | Shopify  | unknown      | unknown     | Guest         | 19395814      | 22758124      | 17.34          |
| SOUTH AMERICA | Shopify  | Middle Aged  | Families    | Existing      | 198045        | 232575        | 17.44          |
| AFRICA        | Shopify  | unknown      | unknown     | New           | 1369382       | 1608546       | 17.47          |
| ASIA          | Shopify  | unknown      | unknown     | New           | 1222698       | 1440528       | 17.82          |
| EUROPE        | Retail   | Retirees     | Couples     | New           | 7369862       | 8730361       | 18.46          |
| SOUTH AMERICA | Shopify  | Young Adults | Families    | New           | 36217         | 43166         | 19.19          |
| EUROPE        | Shopify  | unknown      | unknown     | Existing      | 138498        | 175398        | 26.64          |
| EUROPE        | Shopify  | unknown      | unknown     | New           | 83499         | 118852        | 42.34          |
