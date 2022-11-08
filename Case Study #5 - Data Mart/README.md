<div align="center">
 <h1>Data Mart 🏬:</h1>
</div>

<p align="center">
    <img src="https://8weeksqlchallenge.com/images/case-study-designs/5.png" width="400" height="400">
</p>

Data Mart specialises in fresh produce. In June 2020, Data Mart introduced large scale supply changes. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

The owner of Data Mart wants the the impact of the change on sales performance quantified.

The key business question he wants you to help him answer are the following:

* What was the quantifiable impact of the changes introduced in June 2020?
* Which platform, region, segment and customer types were the most impacted by this change?
* What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?

The tasks include:

* **[Data Cleansing](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/A.Data%20Cleansing.md)** - Cleaning the data provided by Data Mart and extracting additional columns from the data.
* **[Data Exploration](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/B.Data%20Exploration.md)** - Analyzing overall sales for Data Mart.
* **[Before&After Analysis](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/C.Before%20%26%20After%20Analysis.md)** - Analyzing sales performance 4 and 12 weeks before and after the change. Comparing the 12 weeks before and after periods to the sales performance in the past years.
* **[Impact Analysis](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/D.Impact%20Analysis.md)** - Analyzing which region, demographic, platform, customer type and age band felt the most impact of the change in terms of sales performance.

**Available Data**
--------------

**Column Dictionary**

The columns are pretty self-explanatory based on the column names but here are some further details about the dataset:

* Data Mart has international operations using a multi-`region` strategy
* Data Mart has both, a retail and online `platform` in the form of a Shopify store front to serve their customers
* Customer `segment` and `customer_type` data relates to personal age and demographics information that is shared with Data Mart
* `transactions` is the count of unique purchases made through Data Mart and sales is the actual dollar amount of purchases
Each record in the dataset is related to a specific aggregated slice of the underlying sales data rolled up into a week_date value which represents the start of the sales week.

**Example Rows**

10 random rows are shown in the table output below from data_mart.weekly_sales:

week_date|	region|	platform	|segment|	customer_type	|transactions	|sales
--------|-------|-----------|--------|----------|------------|------------
9/9/20|	OCEANIA|	Shopify|	C3	|New	|610	|110033.89
29/7/20|	AFRICA|	Retail	|C1	|New|	110692|	3053771.19
22/7/20|	EUROPE|	Shopify	|C4	|Existing	|24	|8101.54
13/5/20|	AFRICA|	Shopify	|null|	Guest|	5287|	1003301.37
24/7/19|	ASIA	|Retail|	C1	|New	|127342	|3151780.41
10/7/19|	CANADA|	Shopify|	F3	|New|	51	|8844.93
26/6/19|	OCEANIA|	Retail	|C3	|New	|152921	|5551385.36
29/5/19|	SOUTH AMERICA|	Shopify|	null|	New	|53|	10056.2
22/8/18|	AFRICA	|Retail	|null	|Existing	|31721	|1718863.58
25/7/18|	SOUTH AMERICA|	Retail|	null|	New|	2136|	8
