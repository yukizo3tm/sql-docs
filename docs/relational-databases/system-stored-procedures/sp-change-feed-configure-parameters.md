---
title: "sys.sp_change_feed_configure_parameters (Transact-SQL)"
description: "The sys.sp_change_feed_configure_parameters system stored procedure is used to increase the batch size with higher transactions."
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: imotiwala
ms.date: 06/03/2024
ms.service: fabric
ms.subservice: system-objects
ms.topic: "reference"
f1_keywords:
  - "sys.sp_change_feed_configure_parameters_TSQL"
  - "sys.sp_change_feed_configure_parameters"
  - "sp_change_feed_configure_parameters_TSQL"
  - "sp_change_feed_configure_parameters"
helpviewer_keywords:
  - "sp_change_feed_configure_parameters"
dev_langs:
  - "TSQL"
monikerRange: ">=sql-server-ver16 || =azuresqldb-current || =fabric || =azure-sqldw-latest"
---
# sys.sp_change_feed_configure_parameters (Transact-SQL)

[!INCLUDE [sqlserver2022-asdb-asa-fabric](../../includes/applies-to-version/sqlserver2022-asdb-asa-fabric.md)]

Used to reduce latency by decreasing change batch size with `@maxtrans`, or to reduce the cost by increasing the batch size. As the batch size increases, less IO operation will be performed.

This system stored procedure is used to fine tune the operational performance for:

- The Azure Synapse Link feature for SQL Server instances and Azure SQL Database. For more information, see [Manage Azure Synapse Link for SQL Server and Azure SQL Database](../../sql-server/synapse-link/synapse-link-sql-server-change-feed-manage.md).
- The Fabric Mirrored Database feature for Azure SQL Database. For more information, see [What is Mirroring in Fabric?](/fabric/database/mirrored-database/overview)

## Syntax

:::image type="icon" source="../../includes/media/topic-link-icon.svg" border="false"::: [Transact-SQL syntax conventions](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)

```syntaxsql
sys.sp_change_feed_configure_parameters
    [ [ @maxtrans = ] max_trans ]
    [ , [ @pollinterval = ] polling_interval ]
[ ; ]
```

## Arguments

#### [ @maxtrans = ] *max_trans*

Data type is **int**. Indicates the maximum number of transactions to process in each scan cycle.

- For Azure Synapse Link, the default value if not specified is `10000`. If specified, the value must be a positive integer.
- For Fabric mirroring, this value is dynamically determined and automatically set.

#### [ @pollinterval = ] *polling_interval*

Data type is **int**. Describes the frequency that the log is scanned for any new changes, in seconds.

- For Azure Synapse Link, the default interval if not specified is 5 seconds. The value must be `5` or larger.
- For Fabric mirroring, this value is dynamically determined and automatically set.

## Returns

`0` (success) or `1` (failure).

## Permissions

A user with [CONTROL database permissions](../security/permissions-database-engine.md), **db_owner** database role membership, or **sysadmin** server role membership can execute this procedure.

## Related content

- [sys.sp_help_change_feed (Transact-SQL)](sp-help-change-feed.md)
- [sys.sp_help_change_feed_table (Transact-SQL)](sp-help-change-feed-table.md)
- [sys.sp_help_change_feed_table_groups (Transact-SQL)](sp-help-change-feed-table-groups.md)
- [sys.sp_help_change_feed_settings (Transact-SQL)](sp-help-change-feed-settings.md)
- [sys.dm_change_feed_log_scan_sessions (Transact-SQL)](../system-dynamic-management-views/sys-dm-change-feed-log-scan-sessions.md)
- [sys.dm_change_feed_errors (Transact-SQL)](../system-dynamic-management-views/sys-dm-change-feed-errors.md)

**For Microsoft Fabric mirrored databases**:

- [What is Mirroring in Fabric?](/fabric/database/mirrored-database/overview)
- [Monitor Fabric mirrored database replication](/fabric/database/mirrored-database/monitor)
- [Explore data in your Mirrored database using Microsoft Fabric](/fabric/database/mirrored-database/explore)

**For Azure Synapse Link**:

- [What is Azure Synapse Link for SQL?](/azure/synapse-analytics/synapse-link/sql-synapse-link-overview)
- [Manage Azure Synapse Link for SQL Server and Azure SQL Database](../../sql-server/synapse-link/synapse-link-sql-server-change-feed-manage.md)
- [Troubleshoot: Azure Synapse Link for SQL initial snapshot issues](/azure/synapse-analytics/synapse-link/troubleshoot/troubleshoot-sql-snapshot-issues)
