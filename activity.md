---

copyright:
  years: 2018, 2023
lastupdated: "2023-12-14"

keywords: activity tracker, activity, event

subcollection: sql-query

---

{{site.data.keyword.attribute-definition-list}}

# Activity Tracker events
{: #activitytracker}

{{site.data.keyword.sqlquery_full}} is deprecated. As of 18 February 2024 you can't create new instances, and access to free instances will be removed. Existing Standard plan instances are supported until 18 January 2025. Any instances that still exist on that date will be deleted.
{: deprecated}

Use the {{site.data.keyword.at_full}} service to track how users and applications interact with {{site.data.keyword.sqlquery_short}}.

The {{site.data.keyword.at_full_notm}} service records user-initiated activities that change the state of a service in {{site.data.keyword.cloud}}. For more information, see [Getting started with {{site.data.keyword.at_short}}](/docs/Activity-Tracker-with-LogDNA?topic=Activity-Tracker-with-LogDNA-getting-started).

You can use the {{site.data.keyword.sqlquery_short}} service to query {{site.data.keyword.at_short}} archive files that are stored in an {{site.data.keyword.cos_short}} bucket in your account. For more information, see [Searching archive data by using the {{site.data.keyword.sqlquery_short}} service](/docs/activity-tracker?topic=activity-tracker-sqlquery).

You can search activity tracker events with {{site.data.keyword.sqlquery_short}}.

## List of events
{: #events}

The following table lists the actions that generate an event:

Actions  |  Description
--- | ---
`sql-query.sql-job.create` |  An SQL query was submitted.
`sql-query.sql-job.list` |  List of jobs was retrieved.
`sql-query.sql-job.get` |  Details of a job were retrieved.
`sql-query.sql-job.disable` |  An SQL query streaming job was stopped.
`sql-query.sql-job.notify` |  An SQL query streaming job can't process all messages before it is rolled out of the `EventStreams` topic.
`sql-query.catalog-table.list` |  List of catalog tables was retrieved.
`sql-query.catalog-table.get` |  Details of a catalog table were retrieved.
`sql-query.catalog-table-partition.list`| List of partitions of a table was retrieved.
