---

copyright:
  years: 2020, 2023
lastupdated: "2023-12-14"

keywords: best practice, faq, sql query

subcollection: sql-query

---

{{site.data.keyword.attribute-definition-list}}

# FAQ
{: #faq}

## Is {{site.data.keyword.sqlquery}} transaction safe?
{: #transaction_safe}

{{site.data.keyword.sqlquery_full}} is deprecated. As of 18 February 2024 you can't create new instances, and access to free instances will be removed. Existing Standard plan instances are supported until 18 January 2025. Any instances that still exist on that date will be deleted.
{: deprecated}

{{site.data.keyword.sqlquery_short}} does not support INSERT, UPDATE, or DELETE operations, while it can be used to create {{site.data.keyword.cos_short}} objects by using the INTO clause. For the created objects in {{site.data.keyword.cos_short}}, the question arises if this operation is transaction safe and ACID compliant? In other words, if you create an object in {{site.data.keyword.cos_short}} by using {{site.data.keyword.sqlquery_short}}, can you assume that this creation is transaction safe? If anything happens before the creation is done, it is rolled back?

The answer is {{site.data.keyword.cos_full}} does not support transactions. However, an object is rolled back if you create it with a single SQL query job on a nonpartitioned object on {{site.data.keyword.cos_short}} because {{site.data.keyword.cos_short}} itself does object write in an atomic fashion. If you want to write complex hierarchies of objects with a single SQL job, two scenarios can happen: 1) the job fails half way through and only part of the objects are written, or 2) during job execution some objects are already visible while others are still in process to be written.
For the latter scenario, use Hive-style partitioned tables, which means that the SQL job writes a new set of objects into a new {{site.data.keyword.cos_short}} prefix location. This method does not affect anything in the Hive-style partitioned table. Only when you then also issue an `ALTER TABLE … ADD PARTITION` with the newly written object location, the data is made available in the Hive-style partitioned table. That ALTER TABLE DDL is in fact also an atomic operation.

## How to handle large time series data sets?
{: #large_ts}

When you use time-series queries, in general, the more time series that exist, the better the parallelism.

The following example query creates one time series per key, and partitions the data based on the key. Anytime a query is run against "ts", it is run in parallel across time series.

```sql
select 
	key, 
	ts
from table 
using time_series_format(key="key", timetick="time_tick", value="amount") in ts
```

The issue arises if "ts" is large and thus does not fit in the memory of one machine. To mitigate this issue, modify your key as follows.

First, add a date or any granularity of time to your key (to generate a smaller time series).

```sql
select
	concat(cast(to_date(time_tick) as str), "-", key) as date_and_key
	time_tick,
	amount
from table
into table2
```

Second, create your time series with the new `date_and_key` key.

```sql
select
	date_and_key,
	ts
from table2
using time_series_format(key="date_and_key", timetick="time_tick", value="amount") in ts
```

The final queries better use parallelism by providing a larger set of keys. In this case, you not only have the set of unique keys, but the dates associated with each key, as well. By doing so, each time series is much smaller, and has a better chance of fitting in memory.

## How to avoid exhausting allocated resources or out of memory errors (OOM)?
{: #oom}

The most common cause for exhausting allocated resources or out of memory errors is the structure of the query or the layout of the data. The following best practices help to avoid such issues:

- Prepare your data first. Transform input formats, such as .csv or .json to a compressed format, such as Parquet.
- Write results into [partitioned result sets](/docs/sql-query?topic=sql-query-sql-reference#partitionedClause). Start with a double digit partition value. An optimal single object size is about 128 MB. For more information, see [the following blog on big data layout](https://www.ibm.com/cloud/blog/big-data-layout).
- Use Parquet as the output format (STORED AS clause) instead of .csv or .json. Avoid working with `Gzip` objects. Depending on how a join clause is written, it can cause memory-intensive operations. Avoid to use functions in the join clause, such as trim. It is better to cleanse the data first with a preparation step and then to use the cleansed data in the join operation, without needing the functions.
- If you already follow the previous recommendations and still get the out of memory errors, try breaking down complex statements into multiple SQL statements.
- When you use time series functions, look at the following [FAQ](https://cloud.ibm.com/docs/sql-query?topic=sql-query-faq).

## How to work with large number of objects?
{: #many_objects}

If you have more than 150,000 objects in a single source location and you don't use a catalog table, it's possible that your query fails with an error, such as "Too many objects for a single query". If you don't use Hive-style partitioning or catalog tables, all objects in the source location must be listed as part of the query execution, even if only a single row from a single object is required for the query.

To successfully query the source location, use the following best practices.

- Create a table with the correct schema specified. If you don't specify the schema, schema inference causes the same error to be returned.
- If you use more than 10 UNION/JOIN constructs for different source URIs, try to lower the number of different sources.
- Depending on the number of table partitions, you can either add the partition manually one by one, or use the `ALTER TABLE … RECOVER PARTITION` command. Although the limit is 20,000 partitions per table by default, stay below 10,000 partitions for a single table.
- Lay your data out by using Hive-style partitioning and aim for an object size of 128 MB if possible.
