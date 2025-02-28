---
title: Microsoft Entra-only authentication
titleSuffix: Azure SQL Database & Azure SQL Managed Instance & Azure Synapse Analytics
description: This article provides information on the Microsoft Entra-only authentication feature with Azure SQL.
author: VanMSFT
ms.author: vanto
ms.reviewer: wiassaf, vanto, mathoma
ms.date: 09/23/2024
ms.service: azure-sql
ms.subservice: security
ms.topic: conceptual
ms.custom:
  - devx-track-azurecli
  - devx-track-azurepowershell
monikerRange: "=azuresql||=azuresql-db||=azuresql-mi"
---

# Microsoft Entra-only authentication with Azure SQL

[!INCLUDE [appliesto-sqldb-sqlmi-asa-dedicated-only](../includes/appliesto-sqldb-sqlmi-asa-dedicated-only.md)]

Microsoft Entra-only authentication is a feature within [Azure SQL](../azure-sql-iaas-vs-paas-what-is-overview.md) that allows the service to only support Microsoft Entra authentication, and is supported for [Azure SQL Database](sql-database-paas-overview.md) and [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md). 

[!INCLUDE [entra-id](../includes/entra-id.md)]

Microsoft Entra-only authentication is also available for dedicated SQL pools (formerly SQL DW) in standalone servers. Microsoft Entra-only authentication can be enabled for the Azure Synapse workspace. For more information, see [Microsoft Entra-only authentication with Azure Synapse workspaces](/azure/synapse-analytics/sql/active-directory-authentication).

SQL authentication is disabled when enabling Microsoft Entra-only authentication in the Azure SQL environment, including connections from SQL server administrators, logins, and users. Only users using [Microsoft Entra authentication](authentication-aad-overview.md) are authorized to connect to the server or database.

Microsoft Entra-only authentication can be enabled or disabled using the Azure portal, Azure CLI, PowerShell, or REST API. Microsoft Entra-only authentication can also be configured during server creation with an Azure Resource Manager (ARM) template.

For more information on Azure SQL authentication, see [Authentication and authorization](logins-create-manage.md#authentication-and-authorization).

## Feature description

When enabling Microsoft Entra-only authentication, [SQL authentication](logins-create-manage.md#authentication-and-authorization) is disabled at the server or managed instance level and prevents any authentication based on any SQL authentication credentials. SQL authentication users won't be able to connect to the [logical server](logical-servers.md) for Azure SQL Database or managed instance, including all of its databases. Although SQL authentication is disabled, new SQL authentication logins and users can still be created by Microsoft Entra accounts with proper permissions. Newly created SQL authentication accounts won't be allowed to connect to the server. Enabling Microsoft Entra-only authentication doesn't remove existing SQL authentication login and user accounts. The feature only prevents these accounts from connecting to the server, and any database created for this server.

You can also force servers to be created with Microsoft Entra-only authentication enabled using Azure Policy. For more information, see [Azure Policy for Microsoft Entra-only authentication with Azure SQL](authentication-azure-ad-only-authentication-policy.md).

## Permissions

Microsoft Entra-only authentication can be enabled or disabled by Microsoft Entra users who are members of high privileged [Microsoft Entra built-in roles](/azure/role-based-access-control/built-in-roles), such as Azure subscription [Owners](/azure/role-based-access-control/built-in-roles#owner), [Contributors](/azure/role-based-access-control/built-in-roles#contributor), and [Global Administrators](/azure/active-directory/roles/permissions-reference#global-administrator). Additionally, the role [SQL Security Manager](/azure/role-based-access-control/built-in-roles#sql-security-manager) can also enable or disable the Microsoft Entra-only authentication feature.

The [SQL Server Contributor](/azure/role-based-access-control/built-in-roles#sql-server-contributor) and [SQL Managed Instance Contributor](/azure/role-based-access-control/built-in-roles#sql-managed-instance-contributor) roles won't have permissions to enable or disable the Microsoft Entra-only authentication feature. This is consistent with the [Separation of Duties](security-best-practice.md#implement-separation-of-duties) approach, where users who can create an Azure SQL server or create a Microsoft Entra admin, can't enable or disable security features.

### Actions required

The following actions are added to the [SQL Security Manager](/azure/role-based-access-control/built-in-roles#sql-security-manager) role to allow management of the Microsoft Entra-only authentication feature.

- Microsoft.Sql/servers/azureADOnlyAuthentications/*
- Microsoft.Sql/servers/administrators/read - required only for users accessing the Azure portal **Microsoft Entra ID** menu
- Microsoft.Sql/managedInstances/azureADOnlyAuthentications/*
- Microsoft.Sql/managedInstances/read

The above actions can also be added to a custom role to manage Microsoft Entra-only authentication. For more information, see [Create and assign a custom role in Microsoft Entra ID](/azure/active-directory/roles/custom-create).

<a name='managing-azure-ad-only-authentication-using-apis'></a>

<a id="managing-microsoft-entra-only-authentication-using-apis"></a>

## Manage Microsoft Entra-only authentication using APIs

> [!IMPORTANT]
> The Microsoft Entra admin must be set before enabling Microsoft Entra-only authentication.

# [Azure CLI](#tab/azure-cli)

You must have Azure CLI version **2.14.2** or higher.

`name` corresponds to the prefix of the server or instance name (for example, `myserver`) and `resource-group` corresponds to the resource the server belongs to (for example, `myresource`).

## Azure SQL Database

For more information, see [az sql server ad-only-auth](/cli/azure/sql/server/ad-only-auth).

### Enable or disable in SQL Database

**Enable**

```azurecli
az sql server ad-only-auth enable --resource-group myresource --name myserver
```

**Disable**

```azurecli
az sql server ad-only-auth disable --resource-group myresource --name myserver
```

### Check the status in SQL Database

```azurecli
az sql server ad-only-auth get --resource-group myresource --name myserver
```

## Azure SQL Managed Instance

For more information, see [az sql mi ad-only-auth](/cli/azure/sql/mi/ad-only-auth).

**Enable**

```azurecli
az sql mi ad-only-auth enable --resource-group myresource --name myserver
```

**Disable**

```azurecli
az sql mi ad-only-auth disable --resource-group myresource --name myserver
```

### Check the status in SQL Managed Instance

```azurecli
az sql mi ad-only-auth get --resource-group myresource --name myserver
```

# [PowerShell](#tab/azure-powershell)

[Az.Sql 2.10.0](https://www.powershellgallery.com/packages/Az.Sql/2.10.0) module or higher is required.

`ServerName` or `InstanceName` correspond to the prefix of the server name (for example, `myserver` or `myinstance`) and `ResourceGroupName` corresponds to the resource the server belongs to (for example, `myresource`).

## Azure SQL Database

### Enable or disable in SQL Database

**Enable**

For more information, see [Enable-AzSqlServerActiveDirectoryOnlyAuthentication](/powershell/module/az.sql/enable-azsqlserveractivedirectoryonlyauthentication). You can also run `get-help Enable-AzSqlServerActiveDirectoryOnlyAuthentication -full`.

```powershell
Enable-AzSqlServerActiveDirectoryOnlyAuthentication -ServerName myserver -ResourceGroupName myresource
```

You can also use the following command:

```powershell
Get-AzSqlServer -ServerName myserver | Enable-AzSqlServerActiveDirectoryOnlyAuthentication
```

**Disable**

For more information, see [Disable-AzSqlServerActiveDirectoryOnlyAuthentication](/powershell/module/az.sql/disable-azsqlserveractivedirectoryonlyauthentication).

```powershell
Disable-AzSqlServerActiveDirectoryOnlyAuthentication -ServerName myserver -ResourceGroupName myresource
```

### Check the status in SQL Database

```powershell
Get-AzSqlServerActiveDirectoryOnlyAuthentication  -ServerName myserver -ResourceGroupName myresource
```

You can also use the following command:

```powershell
Get-AzSqlServer -ServerName myserver | Get-AzSqlServerActiveDirectoryOnlyAuthentication
```

## Azure SQL Managed Instance

### Enable or disable in SQL Managed Instance

**Enable**

For more information, see [Enable-AzSqlInstanceActiveDirectoryOnlyAuthentication](/powershell/module/az.sql/enable-azsqlinstanceactivedirectoryonlyauthentication).

```powershell
Enable-AzSqlInstanceActiveDirectoryOnlyAuthentication -InstanceName myinstance -ResourceGroupName myresource
```

You can also use the following command:

```powershell
Get-AzSqlInstance -InstanceName myinstance | Enable-AzSqlInstanceActiveDirectoryOnlyAuthentication
```

For more information on these PowerShell commands, run `get-help  Enable-AzSqlInstanceActiveDirectoryOnlyAuthentication -full`.

**Disable**

For more information, see [Disable-AzSqlInstanceActiveDirectoryOnlyAuthentication](/powershell/module/az.sql/disable-azsqlinstanceactivedirectoryonlyauthentication).

```powershell
Disable-AzSqlInstanceActiveDirectoryOnlyAuthentication -InstanceName myinstance -ResourceGroupName myresource
```

### Check the status in SQL Managed Instance

```powershell
Get-AzSqlInstanceActiveDirectoryOnlyAuthentication -InstanceName myinstance -ResourceGroupName myresource
```

You can also use the following command:

```powershell
Get-AzSqlInstance -InstanceName myinstance | Get-AzSqlInstanceActiveDirectoryOnlyAuthentication
```

# [REST API](#tab/rest-api)

The following parameters need to be defined:

- `<subscriptionId>` can be found by navigating to **Subscriptions** in the Azure portal.
- `<myserver>` correspond to the prefix of the server or instance name (for example, `myserver`).
- `<myresource>` corresponds to the resource the server belongs to (for example, `myresource`)

To use latest MSAL, download it from https://www.powershellgallery.com/packages/MSAL.PS.

```rest
$subscriptionId = '<subscriptionId>'
$serverName = "<myserver>"
$resourceGroupName = "<myresource>"
```

## Azure SQL Database

For more information, see the [Server Azure AD Only Authentications](/rest/api/sql/server-azure-ad-only-authentications) REST API documentation.

### Enable or disable in SQL Database

**Enable**

```rest
$body = @{ properties = @{ azureADOnlyAuthentication = 1 } } | ConvertTo-Json
Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Sql/servers/$serverName/azureADOnlyAuthentications/default?api-version=2020-02-02-preview" -Method PUT -Headers $authHeader -Body $body -ContentType "application/json"
```

**Disable**

```rest
$body = @{ properties = @{ azureADOnlyAuthentication = 0 } } | ConvertTo-Json
Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Sql/servers/$serverName/azureADOnlyAuthentications/default?api-version=2020-02-02-preview" -Method PUT -Headers $authHeader -Body $body -ContentType "application/json"
```

### Check the status in SQL Database

```rest
Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Sql/servers/$serverName/azureADOnlyAuthentications/default?api-version=2020-02-02-preview" -Method GET -Headers $authHeader  | Format-List
```

## Azure SQL Managed Instance

For more information, see the [Azure SQL Managed Instance Azure AD Only Authentications](/rest/api/sql/managed-instance-azure-ad-only-authentications) REST API documentation.

### Enable or disable in SQL Managed Instance

**Enable**

```rest
$body = @{   properties = @{  azureADOnlyAuthentication = 1 }  } | ConvertTo-Json 
Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Sql/managedInstances/$serverName/azureADOnlyAuthentications/default?api-version=2020-02-02-preview" -Method PUT -Headers $authHeader -Body $body -ContentType "application/json"
```

**Disable**

```rest
$body = @{   properties = @{  azureADOnlyAuthentication = 0 }  } | ConvertTo-Json 
Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Sql/managedInstances/$serverName/azureADOnlyAuthentications/default?api-version=2020-02-02-preview" -Method PUT -Headers $authHeader -Body $body -ContentType "application/json"
```

### Check the status in SQL Managed Instance

```rest
Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Sql/managedInstances/$serverName/azureADOnlyAuthentications/default?api-version=2020-02-02-preview" -Method GET -Headers $authHeader  | Format-List
```

# [ARM Template](#tab/arm)

- Input the Microsoft Entra admin for the deployment. You will find the user Object ID by going to the [Azure portal](https://portal.azure.com) and navigating to your **Microsoft Entra ID** resource. Under **Manage**, select **Users**. Search for the user you want to set as the Microsoft Entra admin for your Azure SQL server. Select the user, and under their **Profile** page, you will see the **Object ID**.
- The Tenant ID can be found in the **Overview** page of your **Microsoft Entra ID** resource.

## Azure SQL Database

### Enable

The following ARM Template enables Microsoft Entra-only authentication in your Azure SQL Database. To disable Microsoft Entra-only authentication, set the `azureADOnlyAuthentication` property to `false`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.1",
    "parameters": {
        "sqlServer_name": {
            "type": "String"
        },
        "aad_admin_name": {
            "type": "String"
        },
        "aad_admin_objectid": {
            "type": "String"
        },
        "aad_admin_tenantid": {
            "type": "String"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Sql/servers/administrators",
            "apiVersion": "2020-02-02-preview",
            "name": "[concat(parameters('sqlServer_name'), '/ActiveDirectory')]",
            "properties": {
                "administratorType": "ActiveDirectory",
                "login": "[parameters('aad_admin_name')]",
                "sid": "[parameters('aad_admin_objectid')]",
                "tenantId": "[parameters('aad_admin_tenantId')]"
            }
        },        
        {
            "type": "Microsoft.Sql/servers/azureADOnlyAuthentications",
            "apiVersion": "2020-02-02-preview",
            "name": "[concat(parameters('sqlServer_name'), '/Default')]",
            "dependsOn": [
                "[resourceId('Microsoft.Sql/servers/administrators', parameters('sqlServer_name'), 'ActiveDirectory')]"
            ],
            "properties": {
                "azureADOnlyAuthentication": true
            }
        }
    ]
}
```

For more information, see [Microsoft.Sql servers/azureADOnlyAuthentications](/azure/templates/microsoft.sql/servers/azureadonlyauthentications).

## Azure SQL Managed Instance

### Enable

The following ARM Template enables Microsoft Entra-only authentication in your Azure SQL Managed Instance. To disable Microsoft Entra-only authentication, set the `azureADOnlyAuthentication` property to `false`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.1",
    "parameters": {
        "instance": {
            "type": "String"
        },
        "aad_admin_name": {
            "type": "String"
        },
        "aad_admin_objectid": {
            "type": "String"
        },
        "aad_admin_tenantid": {
            "type": "String"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Sql/managedInstances/administrators",
            "apiVersion": "2020-02-02-preview",
            "name": "[concat(parameters('instance'), '/ActiveDirectory')]",
            "properties": {
                "administratorType": "ActiveDirectory",
                "login": "[parameters('aad_admin_name')]",
                "sid": "[parameters('aad_admin_objectid')]",
                "tenantId": "[parameters('aad_admin_tenantId')]"
            }
        },
        {
            "type": "Microsoft.Sql/managedInstances/azureADOnlyAuthentications",
            "apiVersion": "2020-02-02-preview",
            "name": "[concat(parameters('instance'), '/Default')]",
            "dependsOn": [
                "[resourceId('Microsoft.Sql/managedInstances/administrators', parameters('instance'), 'ActiveDirectory')]"
            ],
            "properties": {
                "azureADOnlyAuthentication": true
            }
        }
    ]
}

```

For more information, see [Microsoft.Sql managedInstances/azureADOnlyAuthentications](/azure/templates/microsoft.sql/managedinstances/azureadonlyauthentications).

---

<a name='checking-azure-ad-only-authentication-using-t-sql'></a>

<a id="checking-microsoft-entra-only-authentication-using-t-sql"></a>

### Check Microsoft Entra-only authentication using T-SQL

The [SERVERPROPERTY](/sql/t-sql/functions/serverproperty-transact-sql) `IsExternalAuthenticationOnly` has been added to check if Microsoft Entra-only authentication is enabled for your server or managed instance. `1` indicates that the feature is enabled, and `0` represents the feature is disabled.

```sql
SELECT SERVERPROPERTY('IsExternalAuthenticationOnly') 
```

## Remarks

- A [SQL Server Contributor](/azure/role-based-access-control/built-in-roles#sql-server-contributor) can set or remove a Microsoft Entra admin, but can't set the **Microsoft Entra authentication only** setting. The [SQL Security Manager](/azure/role-based-access-control/built-in-roles#sql-security-manager) can't set or remove a Microsoft Entra admin, but can set the **Microsoft Entra authentication only** setting. Only accounts with higher Azure RBAC roles or custom roles that contain both permissions can set or remove a Microsoft Entra admin and set the **Microsoft Entra authentication only** setting. One such role is the [Contributor](/azure/role-based-access-control/built-in-roles#contributor) role.
- After enabling or disabling **Microsoft Entra authentication only** in the Azure portal, an **Activity log** entry can be seen in the **SQL server** menu.
    :::image type="content" source="media/authentication-azure-ad-only-authentication/azure-ad-only-authentication-portal-sql-server-activity-log.png" alt-text="Screenshot from the Azure portal of the activity log entry." lightbox="media/authentication-azure-ad-only-authentication/azure-ad-only-authentication-portal-sql-server-activity-log.png":::
- The **Microsoft Entra authentication only** setting can only be enabled or disabled by users with the right permissions if the **Microsoft Entra admin** is specified. If the Microsoft Entra admin isn't set, the **Microsoft Entra authentication only** setting remains inactive and cannot be enabled or disabled. Using APIs to enable Microsoft Entra-only authentication will also fail if the Microsoft Entra admin hasn't been set.
- Changing a Microsoft Entra admin when Microsoft Entra-only authentication is enabled is supported for users with the appropriate permissions.
- Changing a Microsoft Entra admin and enabling or disabling Microsoft Entra-only authentication is allowed in the Azure portal for users with the appropriate permissions. Both operations can be completed with one **Save** in the Azure portal. The Microsoft Entra admin must be set in order to enable Microsoft Entra-only authentication.
- Removing a Microsoft Entra admin when the Microsoft Entra-only authentication feature is enabled isn't supported. Using an API to remove a Microsoft Entra admin will fail if Microsoft Entra-only authentication is enabled.
    - If the **Microsoft Entra authentication only** setting is enabled, the **Remove admin** button is inactive in the Azure portal.
- Removing a Microsoft Entra admin and disabling the **Microsoft Entra authentication only** setting is allowed, but requires the right user permission to complete the operations. Both operations can be completed with one **Save** in the Azure portal.
- Microsoft Entra users with proper permissions can impersonate existing SQL users.
    - Impersonation continues working between SQL authentication users even when the Microsoft Entra-only authentication feature is enabled.

<a name='limitations-for-azure-ad-only-authentication-in-sql-database'></a>

### Limitations for Microsoft Entra-only authentication in SQL Database

When Microsoft Entra-only authentication is enabled for SQL Database, the following features aren't supported:

- [Azure SQL Database server roles for permission management](security-server-roles.md) are supported for [Microsoft Entra server principals](authentication-azure-ad-logins.md), but not if the Microsoft Entra login is a group.
- [Elastic jobs in Azure SQL Database](elastic-jobs-overview.md)
- [SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md)
- [Change data capture (CDC)](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) - If you create a database in Azure SQL Database as a Microsoft Entra user and enable change data capture on it, a SQL user will not be able to disable or make changes to CDC artifacts. However, another Microsoft Entra user will be able to enable or disable CDC on the same database. Similarly, if you create an Azure SQL Database as a SQL user, enabling or disabling CDC as a Microsoft Entra user won't work
- [Transactional replication with Azure SQL Managed Instance](../managed-instance/replication-transactional-overview.md) - Since SQL authentication is required for connectivity between replication participants, when Microsoft Entra-only authentication is enabled, transactional replication is not supported for SQL Database for scenarios where transactional replication is used to push changes made in an Azure SQL Managed Instance, on-premises SQL Server, or an Azure VM SQL Server instance to a database in Azure SQL Database
- [SQL Insights (preview)](/azure/azure-monitor/insights/sql-insights-overview)
- `EXEC AS` statement for Microsoft Entra group member accounts

<a name='limitations-for-azure-ad-only-authentication-in-managed-instance'></a>

### Limitations for Microsoft Entra-only authentication in Azure SQL Managed Instance

When Microsoft Entra-only authentication is enabled for SQL Managed Instance, the following features aren't supported:

- [Transactional replication with Azure SQL Managed Instance](../managed-instance/replication-transactional-overview.md) 
- [Automate management tasks using SQL Agent jobs in Azure SQL Managed Instance](../managed-instance/job-automation-managed-instance.md) supports Microsoft Entra-only authentication. However, the Microsoft Entra user who is a member of a Microsoft Entra group that has access to the managed instance cannot own SQL Agent Jobs.
- [SQL Insights (preview)](/azure/azure-monitor/insights/sql-insights-overview)
- `EXEC AS` statement for Microsoft Entra group member accounts

For more limitations, see [T-SQL differences between SQL Server & Azure SQL Managed Instance](../managed-instance/transact-sql-tsql-differences-sql-server.md#logins-and-users).

## Related content

- [Tutorial: Enable Microsoft Entra-only authentication with Azure SQL](authentication-azure-ad-only-authentication-tutorial.md)
- [Create server with Microsoft Entra-only authentication enabled in Azure SQL](authentication-azure-ad-only-authentication-create-server.md)
