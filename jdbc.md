---

copyright:
  years: 2020, 2023
lastupdated: "2023-12-14"

keywords: jdbc, data engine

subcollection: sql-query

---

{{site.data.keyword.attribute-definition-list}}

# JDBC driver
{: #jdbc}

## Driver download
{: #driver_download}

[ibmcloudsql-jdbc-jar]: <> "lines=1 search=\[.*\](.*) replace=exp:[`${VERSION}`](https://us.sql-query.cloud.ibm.com/download/jdbc/ibmcloudsql-jdbc-${VERSION}.jar)"

Download the latest version: [`2.7.35`](https://us.sql-query.cloud.ibm.com/download/jdbc/ibmcloudsql-jdbc-2.7.35.jar)

[previous-ibmcloudsql-jdbc-jar]: <> "lines=1 search=\[.*\](.*) replace=ref:ibmcloudsql-jdbc-jar:link"
Here you find the previous version for reference: [`2.7.34`](https://us.sql-query.cloud.ibm.com/download/jdbc/ibmcloudsql-jdbc-2.7.34.jar)

## JDBC driver class and URL format
{: #jdbc_class}

{{site.data.keyword.sqlquery_full}} is deprecated. As of 18 February 2024 you can't create new instances, and access to free instances will be removed. Existing Standard plan instances are supported until 18 January 2025. Any instances that still exist on that date will be deleted.
{: deprecated}

The Java™ class name for the JDBC driver is `com.ibm.cloud.sql.jdbc.Driver`.

The JDBC URL needs to include the CRN of an {{site.data.keyword.sqlquery_short}} instance, and must match the following schema:

`jdbc:ibmcloudsql:{instance-crn}[?{key1}={value1}&{key2}={value2}...]`

The CRN can be obtained from the service dashboard in the {{site.data.keyword.cloud}} console. The following example shows a JDBC URL:

`jdbc:ibmcloudsql:crn:v1:bluemix:public:sql-query:eu-de:a/290ec9931c0737248f3dc2aa57187d14%3AAf86d20c4-7646-441e-ad4b-182c57008471::?targetcosurl=cos://us-geo/sample-result-bucket/results/`

You can also easily obtain the JDBC URL from the {{site.data.keyword.sqlquery_short}} UI from the **Connect** tab.

Connection properties (except for the CRN) can be specified as part of the URL, separated by `&`, or through the Java connection properties object. The following properties are supported:

- `password` (required): {{site.data.keyword.cloud_notm}} API key for running the queries. The property name must start with a lowercase letter.
- `user` (optional): A username is not required and is ignored if given. It must start with a lowercase letter.
- `targetcosurl` (optional, but usually needed): Cloud {{site.data.keyword.cos_short}} URI in SQL query style, where the results are stored. If this property is not specified, you cannot run queries that return a JDBC result set. The JDBC connection can still be used to retrieve database metadata and run DDL and [ETL-type statements](#etl-type-statements). It must start with a lowercase letter.
- `loggerFile` (optional, default none): file to write driver logs to.
- `loggerLevel` (optional, default set by JDK: `java.util.logging` level for the driver. JDK default is usually `INFO`.
    - `DEBUG/FINER` or `TRACE/FINEST` are the most useful values.
- `filterType` (optional, default none):
    - Only tables are returned if `filterType` value is set to `table`.
    - Only views are returned if `filterType` value is set to `view`.
- `appendInto` (optional, default *true*):
    - If it is set to false, no `INTO` clause is appended, and results are not available through the driver. It is used with [ETL-type statements](#etl-type-statements), where the INTO options are provided as part of the statement.

## Driver functions
{: #driver_functionality}

This driver is designed as a facade to JVM applications for easy access to {{site.data.keyword.sqlquery_short}} service. The driver uses the REST API of the service to run queries, stores the results in Cloud {{site.data.keyword.cos_short}}, and makes these results accessible through JDBC interfaces.

The JDBC driver supports Java 17. To use of the JDBC driver in a Java 17 JVM, you must pass the following option to the JVM:

```
 --add-opens java.base/java.lang=ALL-UNNAMED
 ```

The driver does not implement full JDBC-compliant functions, but only the parts of the API that are most commonly used by clients. The following functions are explicitly tested:

- Retrieving query results with the following primitive SQL types: `STRING` or `VARCHAR`, `INTEGER`, `LONG`, `FLOAT`, `DOUBLE`, `DECIMAL`, `DATE`, `TIME`, `TIMESTAMP`. `TIMESTAMP` columns are handled as Coordinated Universal Time (UTC).
- Inspecting result columns and types through the JDBC ResultSetMetaData interface.
- Running DDL statements `CREATE` or `DROP TABLE` to manage objects in the [{{site.data.keyword.sqlquery_short}} catalog](/docs/sql-query?topic=sql-query-hivemetastore).
- Accessing a list of tables and table columns through the JDBC DatabaseMetaData interface. The schema information is retrieved from the {{site.data.keyword.sqlquery_short}} catalog.

The SQL syntax itself is the full syntax that is supported by {{site.data.keyword.sqlquery_short}} except for the `INTO` clause. An `INTO` clause is implicitly added by the driver based on the Cloud {{site.data.keyword.cos_short}} location that is given with the `targetcosurl` connection property.

Results in the `targetcosurl` location are never deleted by the driver. Use [expiration rules](/docs/cloud-object-storage?topic=cloud-object-storage-expiry) to clean up your results on the {{site.data.keyword.cloud_notm}} automatically.

## Basic limitations
{: #basic_limitations}

The following limitations are implied by the use of {{site.data.keyword.sqlquery_short}}, which is not a full-featured database:

- The driver is based on an asynchronous REST API and does not maintain persistent *connections* to the backend.
- {{site.data.keyword.sqlquery_short}} is not designed for interactive performance on small data. Even tiny queries usually take several seconds to run.
- Query results are returned through results in Cloud {{site.data.keyword.cos_short}}. A `SELECT *` query creates a full copy of the selected table (or Cloud {{site.data.keyword.cos_short}}) in the `targetcosurl` location.
- Streaming of query results cannot start until the execution fully completed and results were written to Cloud {{site.data.keyword.cos_short}}.
- {{site.data.keyword.sqlquery_short}} works on read-only data in Cloud {{site.data.keyword.cos_short}}, so the following functions that are related to data updates is not supported:
    - Transactions are not supported. `commit()` and `rollback()` are no-ops.
    - Result sets that can be updated are not supported.
- SQL statements cannot be canceled.

## Driver-specific limitations
{: #driver_limitations}

See the following technical limitations of the driver:

- Only primitive SQL types are supported in the result. Types like `STRUCT`, `ARRAY`, `LOB`, or `ROWID` cannot be retrieved from the result.
- When you access the result, use the getter method, matching the result column type, for example, `getInt()` to retrieve an INTEGER column. Some implicit conversions are supported, for example, accessing an INTEGER column with `getLong()`, but not all conversions are implemented.
- All result sets are `FORWARD_ONLY` and can be iterated with the `ResultSet.next()` method only. Determining the cursor position and explicitly moving the cursor is not supported.
- Prepared statements are not supported.
- Batch execution with the `Statement.addBatch()` and `executeBatch()` methods is not supported.
- Various other uncommon JDBC methods are not implemented.

## ETL-type statements
{: #etl_statements}

You can use the JDBC driver to run transform-type statements on your data in Cloud {{site.data.keyword.cos_short}}, where you do not want to retrieve the data in the JDBC client itself. See the following example of such a statement:

```sql
SELECT ...
FROM cos://source1 JOIN cos://source2 ON ... JOIN ...
INTO cos://target STORED AS ... PARTITIONED BY ...
```

You can run this type of statement (with a user-defined INTO clause) by the `Statement.executeUpdate` method. This statement does not give you a result set in the JDBC client (and avoids all overhead for result fetching), but creates a target object with the specific format and partitioning that you need.

You cannot run statements with an `INTO` clause by using the generic `Statement.execute()` method because the driver cannot detect to treat these statements as an update, not a query. Therefore, the driver generates a second `INTO` clause, leading to a syntax error.

## JDBC driver logging
{: #jdbc_driver_logging}

JDBC driver logging uses the java.util.logging framework, similar to the [postgresql JDBC driver](https://jdbc.postgresql.org/documentation/logging/). However, since driver version 2.5.30, logging is turned off by default to avoid unexpected console output in a default logging configuration. To turn on logging, use the connection property `loggerLevel`:

Set `loggerLevel` to an explicit log level name, such as `INFO`, to configure that log level for the driver base logger `com.ibm.cloud.sql.jdbc`.
Set `loggerLevel` to the value default to respect the `existing java.util.logging` configuration without overriding the base logger level. Java™ SE Development Kit default is usually to log to the console at `INFO` level, but you can [set log levels and log destinations with a configuration file](https://docs.oracle.com/javase/8/docs/api/java/util/logging/LogManager.html). The file name must be specified as Java system property `-D java.util.logging.config.file=<path>`.
Not setting the `loggerLevel` property is equivalent to setting the value to `OFF` explicitly.
For convenience, and in cases where JVM system properties are not under your control, use the connection property `loggerFile` to control the log destination. This property installs a file handler for the driver base logger. For example, append `&loggerLevel=debug&loggerFile=/tmp/sqlquery.log` to the JDBC URL to create detailed logging output in files `/tmp/sqlquery.log.<n>`.

If you set `loggerLevel` higher than `INFO` (for example, `DEBUG`), it has no effect, unless you also configure `loggerFile` or install a log handler because the default console handler suppresses all messages with a log level higher than `INFO`.

## Using the driver with Tableau Desktop
{: #using_tableau}

[Tableau Desktop](https://www.tableau.com/products/desktop) is a BI reporting tool that connects to a rich set of data sources. You can connect to any custom JDBC driver by using the generic JDBC connector offered by Tableau. Download Tableau Desktop version 2020.2 or later.

To make sure that Tableau generates SQL that is supported by a specific JDBC driver only, you must specify the supported and unsupported SQL capabilities of the driver. Tableau generates appropriate SQL statements dependent on this specification.

The following steps describe how to make Tableau Desktop for Windows work with the {{site.data.keyword.sqlquery_short}} JDBC driver:

1. Install Tableau Desktop for Windows.
2. Download the {{site.data.keyword.sqlquery_short}} JDBC driver and copy it to the installation directory of Tableau.
    - For **Windows**: `C:\Program Files\Tableau\Drivers\ibmcloudsql-jdbc-<version>.jar`
    - For **Mac**: `~/Library/Tableau/Drivers/ibmcloudsql-jdbc-<version>.jar`
3. Create a Tableau data source customization file (*.tdc) with the following content:

    ```sql
    <connection-customization class='genericjdbc' enabled='true' version='2020.2'>
    <vendor name='genericjdbc' />
        <driver name='ibmcloudsql' />
            <customizations>
            <customization name='CAP_CREATE_TEMP_TABLES' value='no'/>
            <customization name='CAP_SELECT_TOP_INTO' value='no'/>
            <customization name='CAP_QUERY_TOP_N' value='yes'/>
            <customization name='CAP_QUERY_TOPSTYLE_TOP' value='no'/>
            <customization name='CAP_QUERY_TOPSTYLE_ROWNUM' value='no'/>
            <customization name='CAP_QUERY_TOPSTYLE_LIMIT' value='yes'/>
            <customization name='CAP_QUERY_SUBQUERIES_WITH_TOP' value='no'/>
            <customization name='CAP_QUERY_GROUP_BY_DEGREE' value='no'/>`
            <customization name='CAP_JDBC_SUPPRESS_ENUMERATE_DATABASES' value='yes'/>
            <customization name='CAP_JDBC_SUPPRESS_ENUMERATE_SCHEMAS' value='yes'/>
            </customizations>
    </connection-customization>
    ```

    - Store the content in a `*.tdc` file in the following folder:
      **Windows**: `C:\Documents\My Tableau Repository\Datasources\ibmcloudsql-jdbc.tdc`
      **Mac**: `~/My Tableau Repository/Datasources/ibmcloudsql-jdbc.tdc`

      If further customization is needed in future, check out [capabilities](https://help.tableau.com/current/pro/desktop/en-us/jdbc_capabilities.htm) that can be turned on and off.
4. Start Tableau Desktop. Go to **Connect > To a Server > More**.
5. On the next page, you see a list of supported connectors. Select **Other Databases (JDBC)**.
6. On the raised input form enter the following information:
   - URL: `jdbc:ibmcloudsql:{CRN of your {{site.data.keyword.sqlquery_short}} service instance}?targetcosurl={COS location for results}`
   - Dialect: `SQL 92`
   - User: `apikey`
   - Password: `{your IBM Cloud API Key}`
7. Click **Sign in**.

As Tableau does not support complex data types, such as `struct`, if a table contains one or more columns with complex data type, perform the following actions:

- Create a flattened view on the table.
- You can set the `filterType` option to `view`. This option effectively hides all tables and reveals views only to Tableau. To set the option with the JDBC URL, use the following URL:

    `jdbc:ibmcloudsql:{CRN of your {{site.data.keyword.sqlquery_short}} service instance}?targetcosurl={COS location for results}&filterType=view`

## Using the driver with Cognos Analytics
{: #using_cognos}

See the [Cognos Analytics Dynamic Query](https://www.ibm.com/docs/en/cognos-analytics/11.2.0?topic=administration-support-cloud-sql-query) documentation to learn how to create a data server connection to {{site.data.keyword.sqlquery_short}} through the {{site.data.keyword.sqlquery_short}} JDBC driver.

## ODBC connectivity that uses the JDBC driver with Progress Data Direct HDP
{: #odbc_connectivity}

The JDBC driver enables BI Tools to integrate data on Cloud {{site.data.keyword.cos_short}} into business reports.
However, some BI tools exist that provide only ODBC connectors but no JDBC connectors.
If you use such business intelligence tools, you need another way to connect to your data.

An easy way to set up ODBC connectivity to {{site.data.keyword.sqlquery_short}} is to use the [Progress DataDirect Hybrid Data Pipeline](https://www.progress.com/cloud-and-hybrid-data-integration) product.

![ODBC connectivity to {{site.data.keyword.sqlquery_short}}.](images/ODBC_image_DE.svg "ODBC connectivity to {{site.data.keyword.sqlquery_short}}"){: caption="ODBC connectivity to {{site.data.keyword.sqlquery_short}}" caption-side="bottom"}

A [step-by-step tutorial](https://www.ibm.com/cloud/blog/odbc-connectivity-to-ibm-cloud-sql-query) explains how easily {{site.data.keyword.sqlquery_short}} and Hybrid Data Pipeline (HDP) can complement one another to achieve this task.
