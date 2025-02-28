---
title: "Set processing options (Reporting Services in SharePoint integrated mode)| Microsoft Docs"
description: In SQL Server Reporting Services in SharePoint integrated mode, specify when data processing occurs, a time-out value, and other options.
author: maggiesMSFT
ms.author: maggies
ms.date: 09/25/2024
ms.service: reporting-services
ms.subservice: report-server-sharepoint
ms.topic: conceptual
ms.custom:
  - updatefrequency5
monikerRange: ">=sql-server-2016 <=sql-server-2016"
---
# Set processing options (Reporting Services in SharePoint integrated mode)

[!INCLUDE[ssrs-appliesto](../../includes/ssrs-appliesto.md)] [!INCLUDE[ssrs-appliesto-2016](../../includes/ssrs-appliesto-2016.md)] [!INCLUDE[ssrs-appliesto-sharepoint-2013-2016i](../../includes/ssrs-appliesto-sharepoint-2013-2016.md)] [!INCLUDE[ssrs-appliesto-not-pbirsi](../../includes/ssrs-appliesto-not-pbirs.md)]

[!INCLUDE [ssrs-previous-versions](../../includes/ssrs-previous-versions.md)]

  You can set processing options on a Reporting Services report to determine when data processing occurs. You can also set a time-out value for report processing, and options that determine whether report history is enabled for the current report.  
  
-   You can run a report as a report snapshot to prevent the report from being run at arbitrary times (for example, during a scheduled backup). A report snapshot is usually created and then refreshed on a schedule, allowing you to time exactly when report and data processing occurs. If a report is based on queries that take a long time to run, or on queries that use data from a data source that you prefer no one access during certain hours, you should run the report as a snapshot.  
  
-   A report server can cache a copy of a processed report and return that copy when a user opens the report. Caching is a performance-enhancement technique. Caching can shorten the time required to retrieve a report if the report is large or accessed frequently.  
  
-   Report history is a collection of previously run copies of a report. You can use report history to maintain a record of a report over time. Report history isn't intended for reports that contain confidential or personal data. For this reason, report history can include only those reports that query a data source using a single set of credentials (either stored credentials or credentials used for unattended report execution) that are available to all users who run a report.  

    Reporting Services integration with SharePoint uses the check out and check in content management features of SharePoint to save updates to Reporting Services content types. This includes the creation of report snapshots. Therefore if you enable versioning on a document library, you see the report version updated when a new report history snapshot is created. This is a side-effect of updating snapshots. When a snapshot is updated, it causes the LastExecution property of the report to change and that causes a change in the version of the report.  

-   You can specify time-out values to set limits on how system resources are used.  

> [!NOTE]
> Reporting Services integration with SharePoint is no longer available after SQL Server 2016.

## Set data refresh options
  
1.  Point to a report in the library.  
  
2.  Select the down arrow, and choose **Manage processing options**.  
  
3.  In **Data Refresh Options**, select **Use snapshot data**. If you see "This report can not run from a snapshot because one or more of the data sources credentials are not stored," the report isn't configured to run unattended, and you must modify the data sources to use stored credentials before setting this option.  
  
4.  In **Data Snapshot Options**, select **Schedule data processing**.  
  
5.  Select **On a shared schedule** if you have an existing shared schedule that you want to use, otherwise select **On a custom schedule**, and then choose **Configure**.  
  
6.  Select frequency, schedule, and start and end dates, and then choose **OK**.  
  
7.  In **Data Snapshot Options**, select **Create or update the snapshot when this page is saved** if you want to immediately create snapshot data to use with the report, without waiting for the scheduled data processing to occur.  
  
## Set report caching options
  
1.  Point to a report in the library.  
  
2.  Select the down arrow, and choose **Manage processing options**.  
  
3.  In **Data Refresh Options**, select **Use cached data**. If you see "This report can not be cached because one or more of the data sources credentials are not stored," the report isn't configured to run unattended and you must modify the data sources to use stored credentials before setting this option.  
  
4.  In **Cache Options**, specify how the cache expires:  
  
    -   Enter a number of minutes after which the cache expires.  
  
    -   Use a shared schedule to clear the cache at times specified in the schedule.  
  
    -   Create a custom schedule to clear the cache at a time that you specify.  
  
## Set processing time-out values
  
1.  Point to a report in the library.  
  
2.  Select the down arrow, and choose **Manage processing options**.  
  
3.  In **Processing Time-out**, select **Use site default setting** if you want to use the value specified at the report server level. Otherwise, select **Do not time out report processing** or **Limit report processing in seconds** if you want to override that value with no time-out or different time-out values.  
  
## Set report history options and limits
  
1.  Point to a report in the library.  
  
2.  Select the down arrow, and choose **Manage processing options**.  
  
3.  In **History Snapshot Options**, specify how and when data processing occurs and is saved.  
  
4.  In **History Snapshot Limits**, select **Use site default setting** if you want to use the value specified at the report server level. Otherwise, select **Do not limit the number of snapshots** or **Limit number of snapshots** to specify a custom value.  
  
## Set database timeout
  
*  Use Windows PowerShell to set the database timeout of a SharePoint report server. For more information, see the "Get and set Properties of the Reporting Service Application Database" section of [PowerShell cmdlets for Reporting Services SharePoint Mode](../../reporting-services/report-server-sharepoint/powershell-cmdlets-for-reporting-services-sharepoint-mode.md).  
  
## Related content

- [Set Report Processing properties](../../reporting-services/report-server/set-report-processing-properties.md)
- [Caching reports](../../reporting-services/report-server/caching-reports-ssrs.md)
- [Setting time-out values for report and shared dataset processing](../../reporting-services/report-server/setting-time-out-values-for-report-and-shared-dataset-processing-ssrs.md)
- [Try asking the Reporting Services forum](https://go.microsoft.com/fwlink/?LinkId=620231)
