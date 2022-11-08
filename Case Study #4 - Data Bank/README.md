<div align="center">
 <h1>Data Bank :bank:</h1>
</div>

<p align="center">
    <img src="https://8weeksqlchallenge.com/images/case-study-designs/4.png" width="400" height="400">
</p>

Data Bank is a digital bank but it is not only for banking activities. They have the world's most secured data storage platform.Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. 

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments.

Data Banks' business problems and questions have been divided into the following sections:

* **[Customer Nodes Exploration](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%234%20-%20Data%20Bank/Customer%20Nodes%20Exploration.md)** - Questions regarding unique nodes, number of nodes in each Region and descriptive statistics of node reallocation days.
* **[Customer Transactions](https://github.com/Outis09/8-Week-SQL-Challenge/blob/main/Case%20Study%20%234%20-%20Data%20Bank/Customer%20Transactions.md)** - Questions regarding transactions, transaction types and opening and closing balances.

**Available Data**
----------------

**Entity Relationship Diagram**

<p align="center">
    <img src="https://8weeksqlchallenge.com/images/case-study-4-erd.png">
</p>

**Table 1: Regions**

Just like popular cryptocurrency platforms - Data Bank is also run off a network of nodes where both money and data is stored across the globe. In a traditional banking sense - you can think of these nodes as bank branches or stores that exist around the world.

This `regions` table contains the `region_id` and their respective `region_name` values

region_id|	region_name
----------|--------
1	|Africa
2	||America
3	|Asia
4	|Europe
5	|Oceania


**Table 2: Customer Nodes**

Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data.

This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!

Below is a sample of the top 10 rows of the `data_bank.customer_nodes`

customer_id|	region_id	|node_id	|start_date	|end_date
------------|---------|-------|----------|-----------
1|	3|	4	|2020-01-02|	2020-01-03
2|	3	|5	|2020-01-03|	2020-01-17
3	|5	|4	|2020-01-27|2020-02-18
4	|5	|4	|2020-01-07	|2020-01-19
5	|3	|3	|2020-01-15	|2020-01-23
6	|1	|1	|2020-01-11	|2020-02-06
7	|2	|5	|2020-01-20	|2020-02-04
8	|1	|2	|2020-01-15	|2020-01-28
9	|4	|5	|2020-01-21	|2020-01-25
10|	3|	4	|2020-01-13|	2020-01-14

**Table 3: Customer Transactions**

This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

customer_id	|txn_date|	txn_type|	txn_amount
----------|--------|-----------|----------
429	|2020-01-21|	deposit	|82
155|	2020-01-10	|deposit	|712
398|	2020-01-01	|deposit	|196
255|	2020-01-14	|deposit	|563
185|	2020-01-29	|deposit	|626
309|	2020-01-13	|deposit	|995
312|	2020-01-20	|deposit	|485
376|	2020-01-03	|deposit	|706
188	|2020-01-13	|deposit	|601
138	|2020-01-11	|deposit	|520
