---

copyright:
  years: 2020, 2023
lastupdated: "2023-12-14"

keywords: aspera, data lake, object storage, csv, monitoring

subcollection: sql-query

---

{{site.data.keyword.attribute-definition-list}}

# Monitoring
{: #metrics}

{{site.data.keyword.sqlquery_full}} is deprecated. As of 18 February 2024 you can't create new instances, and access to free instances will be removed. Existing Standard plan instances are supported until 18 January 2025. Any instances that still exist on that date will be deleted.
{: deprecated}

{{site.data.keyword.mon_full}} is a cloud-native, and container-intelligence management system that you can include as part of your {{site.data.keyword.cloud_notm}} architecture. Use it to gain operational visibility into the performance and health of your applications, services, and platforms. It offers administrators, DevOps teams, and developers full stack telemetry with advanced features to monitor and troubleshoot, define alerts, and design custom dashboards.
{: shortdesc}

## Opting in to and enabling {{site.data.keyword.sqlquery_short}} metrics
{: #opt_in_metrics}

Before you can start to use {{site.data.keyword.sqlquery_full}} metrics, you must first opt in, and then enable platform metrics by completing the following steps.

1. Enable platform metrics for {{site.data.keyword.sqlquery_short}}. For more information, see [Enabling platform metrics](/docs/Monitoring-with-Sysdig?topic=Monitoring-with-Sysdig-platform_metrics_enabling){: external}. The owner of the account has full access to the metrics data. For more information about managing access for other users, see [Getting started tutorial for {{site.data.keyword.mon_full}}](/docs/Monitoring-with-Sysdig?topic=Monitoring-with-Sysdig-getting-started){: external}.

2. To navigate from the {{site.data.keyword.sqlquery_short}} instance page to the {{site.data.keyword.mon_full_notm}} dashboard, click the overflow menu (three vertical dots) of the instance page (**Service instance options**) and select **Monitoring**.

   On your first usage, you might see a welcome wizard. To advance to the dashboard selection menu, select **Next** and then **Skip** on the **Choosing an installation method** page. Accept the prompts that follow. You can then select the **{{site.data.keyword.sqlquery_short}}** dashboard.

   Dashboards are available only after metrics start to be recorded, which might take a few minutes to initialize.
   {: note}

## {{site.data.keyword.sqlquery_short}} metrics cost information
{: #metric_costs}

Before you opt in to using {{site.data.keyword.mon_full}} metrics, be aware of the cost of doing so. Every instance of {{site.data.keyword.sqlquery_short}} makes four metrics available, each time series has a monthly cost of $0.08, which leads to a monthly cost of $0.32 per instance.

For more information, see [{{site.data.keyword.mon_full_notm}} pricing ![External link icon](../../icons/launch-glyph.svg "External link icon")](/docs/Monitoring-with-Sysdig?topic=Monitoring-with-Sysdig-pricing_plans).

## {{site.data.keyword.sqlquery_short}} metrics details
{: #metric_details}

The following tables describe the specific metrics that are provided by {{site.data.keyword.sqlquery_short}}.

| Metric name |
|-----------|
| [Number of bytes read by completed jobs](#ibm_sqlquery_bytes_read) |
| [Number of completed jobs](#ibm_sqlquery_completed_jobs) |
| [Number of failed jobs](#ibm_sqlquery_failed_jobs) |
| [Number of jobs in progress](#ibm_sqlquery_jobs_in_progress) |
{: caption="Available metrics." caption-side="top"}

### Number of bytes read by completed jobs
{: #ibm_sqlquery_bytes_read}

The following table shows the cumulative number of bytes read by all {{site.data.keyword.sqlquery_short}} jobs that completed successfully.

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_sqlquery_bytes_read`|
| `Metric type` | `gauge` |
| `Value type`  | `byte` |
| `Segment by` | `Service instance, Service instance name` |
{: caption="Number of bytes read by completed jobs metric metadata." caption-side="top"}

### Number of completed jobs
{: #ibm_sqlquery_completed_jobs}

The following table shows the number of {{site.data.keyword.sqlquery_short}} jobs that were completed successfully.

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_sqlquery_completed_jobs`|
| `Metric type` | `gauge` |
| `Value type`  | `none` |
| `Segment by` | `Service instance, Service instance name` |
{: caption="Number of completed jobs metric metadata." caption-side="top"}

### Number of failed jobs
{: #ibm_sqlquery_failed_jobs}

The following table shows the number of {{site.data.keyword.sqlquery_short}} jobs that failed.

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_sqlquery_failed_jobs`|
| `Metric type` | `gauge` |
| `Value type`  | `none` |
| `Segment by` | `Service instance, Service instance name` |
{: caption="Number of failed jobs metric metadata." caption-side="top"}

### Number of jobs in progress
{: #ibm_sqlquery_jobs_in_progress}

The following table shows the number of submitted {{site.data.keyword.sqlquery_short}} jobs that are in progress (queued or running).

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_sqlquery_jobs_in_progress`|
| `Metric type` | `gauge` |
| `Value type`  | `none` |
| `Segment by` | `Service instance, Service instance name` |
{: caption="Number of jobs in progress metric metadata." caption-side="top"}

## Attributes for segmentation
{: #attributes}

### Global attributes
{: #global-attributes}

The following attributes are available for segmenting all of the listed metrics.

| Attribute | Attribute name | Attribute description |
|-----------|----------------|-----------------------|
| `Cloud type` | `ibm_ctype` | The cloud type is a value of public, dedicated, or local. |
| `Location` | `ibm_location` | The location of the monitored resource, which can be a region, a data center, or global. |
| `Resource group` | `ibm_resource_group_name` | The resource group where the service instance was created. |
| `Scope` | `ibm_scope` | The scope is the account, organization, or space GUID associated with this metric. |
| `Service name` | `ibm_service_name` | Name of the service that generates this metric. |
{: caption="Global attributes." caption-side="top"}

### More attributes
{: #more-attributes}

The following attributes are available for segmenting one or more attributes as described in the preceding reference. See the individual metrics for segmentation options.

| Attribute | Attribute name | Attribute description |
|-----------|----------------|-----------------------|
| `Service instance` | `ibm_service_instance` | The service instance segment identifies the instance that the metric is associated with. |
| `Service instance name` | `ibm_service_instance_name` | The service instance name provides the user-provided name of the service instance that isn't necessarily a unique value, depending on the name provided by the user. |
{: caption="More attributes." caption-side="top"}

## Next Steps
{: #monitoring-next}

You can manage the data that the Monitoring instance collects for you. From management to setting alerts, you can [get started ![External link icon](../../icons/launch-glyph.svg "External link icon")](/docs/Monitoring-with-Sysdig?topic=Monitoring-with-Sysdig-getting-started) monitoring your data quickly and efficiently.
