In progress...


<div align="center">
 <h1>Fresh Segments :orange:</h1>
</div>

<p align="center">
    <img src="https://8weeksqlchallenge.com/images/case-study-designs/8.png" width="400" height="400">
</p>

**Problem Statement**
-------------------

Danny created Fresh Segments, a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.

Danny has asked for your assistance to analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests.

**Available Data**
------------------

For this case study there is a total of 2 datasets which you will need to use to solve the questions.

**Table 1: Interest Metrics**

This table contains information about aggregated interest metrics for a specific major client of Fresh Segments which makes up a large proportion of their customer base.

Each record in this table represents the performance of a specific interest_id based on the client’s customer base interest measured through clicks and interactions with specific targeted advertising content.

_month|	_year|	month_year|	interest_id|	composition|	index_value|	ranking|	percentile_ranking
|-----|-----|------------|----------|-----------|----------|------------|-------------|
7|	2018|	07-2018|	32486|	11.89|	6.19|	1	|99.86
7	|2018	|07-2018	|6106	|9.93	|5.31	|2	|99.73
7	|2018|	07-2018|	18923|	10.85|	5.29|	3|	99.59
7	|2018	|07-2018	|6344	|10.32	|5.1	|4	|99.45
7	|2018	|07-2018	|100	|10.77|	5.04|	5	|99.31
7	|2018	|07-2018	|69	|10.82	|5.03|	6	|99.18
7	|2018|	07-2018|	79|	11.21|	4.97|	7|	99.04
7	|2018	|07-2018	|6111	|10.71	|4.83	|8	|98.9
7	|2018	|07-2018	|6214	|9.71|	4.83|	8|	98.9
7	|2018	|07-2018	|19422	|10.11	|4.81	|10	|98.63

For example - let’s interpret the first row of the interest_metrics table together:

_month|	_year|	month_year|	interest_id|	composition|	index_value|	ranking|	percentile_ranking
|-----|-----|------------|----------|-----------|----------|------------|-------------|
7|	2018|	07-2018|	32486|	11.89|	6.19|	1	|99.86

In July 2018, the composition metric is 11.89, meaning that 11.89% of the client’s customer list interacted with the interest interest_id = 32486 - we can link interest_id to a separate mapping table to find the segment name called “Vacation Rental Accommodation Researchers”

The index_value is 6.19, means that the composition value is 6.19x the average composition value for all Fresh Segments clients’ customer for this particular interest in the month of July 2018.

The ranking and percentage_ranking relates to the order of index_value records in each month year.

**Table 2: Interest Map**

This mapping table links the interest_id with their relevant interest information. You will need to join this table onto the previous interest_details table to obtain the interest_name as well as any details about the summary information.

id|	interest_name|	interest_summary|	created_at|	last_modified|
---|-------------|-----------------|----------|---------------|
1	|Fitness Enthusiasts	|Consumers using fitness tracking apps and websites.|	2016-05-26 14:57:59	|2018-05-23 11:30:12
2	|Gamers|	Consumers researching game reviews and cheat codes.|	2016-05-26 14:57:59	|2018-05-23 11:30:12
3	|Car Enthusiasts|	Readers of automotive news and car reviews.|	2016-05-26 14:57:59|	2018-05-23 11:30:12
4	|Luxury Retail Researchers	|Consumers researching luxury product reviews and gift ideas.|	2016-05-26 14:57:59|	2018-05-23 11:30:12
5	|Brides & Wedding Planners|	People researching wedding ideas and vendors.|	2016-05-26 14:57:59	|2018-05-23 11:30:12
6	|Vacation Planners	|Consumers reading reviews of vacation destinations and accommodations.|	2016-05-26 14:57:59|	2018-05-23 11:30:13
7	|Motorcycle Enthusiasts|	Readers of motorcycle news and reviews.|	2016-05-26 14:57:59	|2018-05-23 11:30:13
8	|Business News Readers	|Readers of online business news content.|	2016-05-26 14:57:59	|2018-05-23 11:30:12
12|	Thrift Store Shoppers	|Consumers shopping online for clothing at thrift stores and researching locations.|	2016-05-26 14:57:59|	2018-03-16 13:14:00
13|	Advertising Professionals|	People who read advertising industry news.|	2016-05-26 14:57:59	|2018-05-23 11:30:12
