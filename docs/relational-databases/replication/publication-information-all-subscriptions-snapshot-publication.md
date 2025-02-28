---
title: "All Subscriptions (Snapshot - SSMS)"
description: Describes the 'All Subscriptions' tab for a selected Snapshot Publication in SQL Server Management Studio (SSMS).
author: "MashaMSFT"
ms.author: "mathoma"
ms.date: 09/25/2024
ms.service: sql
ms.subservice: replication
ms.topic: ui-reference
ms.custom:
  - updatefrequency5
f1_keywords:
  - "sql13.rep.monitor.publicationinfo.allsubscriptions.snapshot.f1"
monikerRange: "=azuresqldb-mi-current||>=sql-server-2016"
---
# Publication Information, All Subscriptions (Snapshot Publication)
[!INCLUDE [SQL Server SQL MI](../../includes/applies-to-version/sql-asdbmi.md)]
  The **All Subscriptions** tab displays information on all subscriptions to the selected snapshot publication.  
  
## Options  
 For more detailed information and tasks related to a subscription, right-click the row for that subscription, and then click an option on the shortcut menu. To change the way that the grid displays data, right-click the grid, and then click one of the following options:  
  
-   **Sort**: Sort on one or more columns in the **Sort Columns** dialog box.  
  
-   **Choose Columns to Show**: Select which columns to display and the order in which to display them in the **Choose Columns** dialog box.  
  
-   **Filter**: Filter rows in the grid based on column values in the **Filter Settings** dialog box.  
  
-   **Clear Filter**: Clear any filter settings for the grid.  
  
 Filter settings are specific to each grid. Column selection and sorting are applied to all grids of the same type, such as the publications grid for each Publisher.  
  
 **Show**  
 [!INCLUDE[msCoName](../../includes/msconame-md.md)] [!INCLUDE[ssVersion2005](../../includes/ssversion2005-md.md)] and later versions only. Select the subscription states to display for the selected subscription type. For example, you can select to display only those subscriptions that have an error.  
  
 **Status**  
 The status of each subscription, which is determined by the status of the Snapshot Agent or the Distribution Agent (the higher priority status is displayed).  
  
 By default, the grid containing subscription information is sorted by the **Status** column. The following list shows the possible status values and the sort order for the values (for example, errors are always shown at the top of the grid).  
  
-   Error  
  
-   Expiring soon/Expired ([!INCLUDE[ssVersion2005](../../includes/ssversion2005-md.md)] and later versions only)  
  
-   Uninitialized subscription ([!INCLUDE[ssVersion2005](../../includes/ssversion2005-md.md)] and later versions only)  
  
-   Retrying failed command  
  
-   Synchronizing  
  
-   Not synchronizing  
  
 The sort order also determines which value is displayed if a given subscription is in more than one state. For example, if a subscription has an error and is expiring soon, the **Status** column displays **Error**.  
  
 The status values **Expiring soon/Expired** and **Uninitialized subscription** are warnings. When a warning is displayed, the **Status** column also displays if an agent is running. For example, the status could be **Running, Expiring soon/Expired**.  
  
 The status value **Expiring soon/Expired** is displayed only if a threshold is set. For information on setting thresholds, see [Set Thresholds and Warnings in Replication Monitor](../../relational-databases/replication/monitor/set-thresholds-and-warnings-in-replication-monitor.md).  
  
 **Subscription**  
 The name of each subscription, in the form: *SubscriberName: SubscriptionDatabaseName*.  
  
 **Last Synchronization**  
 The time at which the Distribution Agent last ran. If synchronization is in progress, **In progress** is displayed.  
  
## Related content

- [Start the Replication Monitor](../../relational-databases/replication/monitor/start-the-replication-monitor.md)
- [View information and perform tasks using Replication Monitor](../../relational-databases/replication/monitor/view-information-and-perform-tasks-replication-monitor.md)
- [Monitoring Replication](../../relational-databases/replication/monitor/monitoring-replication.md)
