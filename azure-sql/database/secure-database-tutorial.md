---
title: Secure a database
description: This tutorial teaches you the about techniques and features to secure an Azure SQL Database, whether it's a single database, or pooled.
author: VanMSFT
ms.author: vanto
ms.reviewer: wiassaf, mathoma
ms.date: 01/26/2024
ms.service: azure-sql-database
ms.subservice: security
ms.topic: tutorial
---
# Tutorial: Secure a database in Azure SQL Database
[!INCLUDE [appliesto-sqldb](../includes/appliesto-sqldb.md)]

In this tutorial you learn how to:

> [!div class="checklist"]
>
> - Create server-level and database-level firewall rules
> - Configure a Microsoft Entra administrator
> - Manage user access with SQL authentication, Microsoft Entra authentication, and secure connection strings
> - Enable security features, such as Microsoft Defender for SQL, auditing, data masking, and encryption

[!INCLUDE [entra-id](../includes/entra-id.md)]

Azure SQL Database secures data by allowing you to:

- Limit access using firewall rules
- Use authentication mechanisms that require identity
- Use authorization with role-based memberships and permissions
- Enable security features

> [!NOTE]
> Azure SQL Managed Instance is secured using network security rules and private endpoints as described in [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) and [connectivity architecture](../managed-instance/connectivity-architecture-overview.md).

To learn more, see the [Azure SQL Database security overview](./security-overview.md) and [capabilities](security-overview.md) articles.

> [!TIP]
> This free Learn module shows you how to [Secure your database in Azure SQL Database](/training/modules/secure-your-azure-sql-database/).

## Prerequisites

To complete the tutorial, make sure you have the following prerequisites:

- [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms)
- A [server](logical-servers.md) and a single database
  - Create them with the [Azure portal](single-database-create-quickstart.md), [CLI](az-cli-script-samples-content-guide.md), or [PowerShell](powershell-script-content-guide.md)

If you don't have an Azure subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.

## Sign in to the Azure portal

For all steps in the tutorial, sign in to the [Azure portal](https://portal.azure.com/)

## Create firewall rules

Databases in SQL Database are protected by firewalls in Azure. By default, all connections to the server and database are rejected. To learn more, see [server-level and database-level firewall rules](firewall-configure.md).

Set **Allow access to Azure services** to **OFF** for the most secure configuration. Then, create a [reserved IP (classic deployment)](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip) for the resource that needs to connect, such as an Azure VM or cloud service, and only allow that IP address access through the firewall. If you're using the [Resource Manager](/azure/virtual-network/ip-services/public-ip-addresses) deployment model, a dedicated public IP address is required for each resource.

> [!NOTE]
> SQL Database communicates over port 1433. If you're trying to connect from within a corporate network, outbound traffic over port 1433 might not be allowed by your network's firewall. If so, you can't connect to the server unless your administrator opens port 1433.

### Set up server-level firewall rules

Server-level IP firewall rules apply to all databases within the same server.

To set up a server-level firewall rule:

1. In the Azure portal, select **SQL databases** from the left-hand menu, and select your database on the **SQL databases** page.

    :::image type="content" source="media\design-first-database-tutorial\server-name.png" alt-text="Screenshot of the Azure portal page for a logical SQL database, highlighting the server name." lightbox="media\design-first-database-tutorial\server-name.png":::

    > [!NOTE]
    > Be sure to copy your fully qualified server name (such as *yourserver.database.windows.net*) for use later in the tutorial.

1. Select **Networking** under **Settings**. Choose the **Public Access** tab, and then select **Selected networks** under **Public network access** to display the **Firewall rules** section. 

   :::image type="content" source="media\design-first-database-tutorial\server-firewall-rule.png" alt-text="Screenshot of the Azure portal Networking page for a logical SQL Server, showing the server-level IP firewall rule." lightbox="media\design-first-database-tutorial\server-firewall-rule.png":::

1. Select **Add client IP** on the toolbar to add your current IP address to a new IP firewall rule. An IP firewall rule can open port 1433 for a single IP address or a range of IP addresses.
1. Select **OK** to save your firewall settings. 

You can now connect to any database in the server with the specified IP address or IP address range.

### Setup database firewall rules

Database-level firewall rules only apply to individual databases. The database will retain these rules during a server failover. Database-level firewall rules can only be configured using Transact-SQL (T-SQL) statements, and only after you've configured a server-level firewall rule.

To set up a database-level firewall rule:

1. Connect to the database, for example using [SQL Server Management Studio](connect-query-ssms.md).

1. In **Object Explorer**, right-click the database and select **New Query**.

1. In the query window, add this statement and modify the IP address to your public IP address:

    ```sql
    EXECUTE sp_set_database_firewall_rule N'Example DB Rule','0.0.0.4','0.0.0.4';
    ```

1. On the toolbar, select **Execute** to create the firewall rule.

> [!NOTE]
> You can also create a server-level firewall rule in SSMS by using the [sp_set_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-set-firewall-rule-azure-sql-database?view=azuresqldb-current&preserve-view=true) command, though you must be connected to the *master* database.

<a name='create-an-azure-ad-admin'></a>

## Create a Microsoft Entra admin

Make sure you're using the appropriate Microsoft Entra ID ([formerly Azure Active Directory](/entra/fundamentals/new-name)) managed domain. To select your domain, use the upper-right corner of the Azure portal. This process confirms the same subscription is used for both Microsoft Entra ID and the logical server hosting your database or data warehouse.

   :::image type="content" source="media\secure-database-tutorial\choose-directory-subscription.png" alt-text="Screenshot of the Azure portal showing the Directory + subscription filter page, where you would choose the directory.":::

To set the Microsoft Entra administrator:

1. In the Azure portal, on the **SQL server** page, select **Microsoft Entra ID** from the resource menu, then select **Set admin** to open the **Microsoft Entra ID** pane.

    :::image type="content" source="media\secure-database-tutorial\admin-settings.png" alt-text="Screenshot of the Azure portal Microsoft Entra ID page for a logical server." lightbox="media\secure-database-tutorial\admin-settings.png":::

    > [!IMPORTANT]
    > You need to be a "Global Administrator" to perform this task.

1. On the **Microsoft Entra ID** pane, search and select the Microsoft Entra user or group and choose **Select**. All members and groups of your Microsoft Entra organization are listed, and entries grayed out are [not supported as Microsoft Entra administrators](authentication-aad-overview.md#microsoft-entra-administrator). 

    :::image type="content" source="media\secure-database-tutorial\admin-select.png" alt-text="Screenshot of the Azure portal page to add a Microsoft Entra admin.":::

    > [!IMPORTANT]
    > Azure role-based access control (Azure RBAC) only applies to the portal and isn't propagated to SQL Server.

1. At the top of the **Microsoft Entra admin** page, select **Save**.

    The process of changing an administrator might take several minutes. The new administrator will appear in the **Microsoft Entra admin** field.

> [!NOTE]
> When setting a Microsoft Entra admin, the new admin name (user or group) cannot exist as a login or user in the *master* database. If present, the setup will fail and roll back changes, indicating that such an admin name already exists. Since the server login or user is not part of Microsoft Entra ID, any effort to connect the user using Microsoft Entra authentication fails.

For information about configuring Microsoft Entra ID, see:

- [Integrate your on-premises identities with Microsoft Entra ID](/azure/active-directory/hybrid/whatis-hybrid-identity)
- [Add your own domain name to Microsoft Entra ID](/azure/active-directory/fundamentals/add-custom-domain)
- [Federation with Microsoft Entra ID](/azure/active-directory/hybrid/connect/whatis-fed)
- [Administer your Microsoft Entra directory](/azure/active-directory/fundamentals/active-directory-whatis)
- [Manage Microsoft Entra ID using PowerShell](/powershell/azure/)
- [Hybrid identity required ports and protocols](/azure/active-directory/hybrid/reference-connect-ports)

## Manage database access

Manage database access by adding users to the database, or allowing user access with secure connection strings. Connection strings are useful for external applications. To learn more, see [Manage logins and user accounts](logins-create-manage.md) and [Microsoft Entra authentication](authentication-aad-overview.md).

To add users, choose the database authentication type:

- **SQL authentication**, use a username and password for logins and are only valid in the context of a specific database within the server

- **Microsoft Entra authentication**, use identities managed by Microsoft Entra ID

### SQL authentication

To add a user with SQL authentication:

1. Connect to the database, for example using [SQL Server Management Studio](connect-query-ssms.md).

1. In **Object Explorer**, right-click the database and choose **New Query**.

1. In the query window, enter the following command:

    ```sql
    CREATE USER ApplicationUser WITH PASSWORD = 'YourStrongPassword1';
    ```

1. On the toolbar, select **Execute** to create the user.

1. By default, the user can connect to the database, but has no permissions to read or write data. To grant these permissions, execute the following commands in a new query window:

    ```sql
    ALTER ROLE db_datareader ADD MEMBER ApplicationUser;
    ALTER ROLE db_datawriter ADD MEMBER ApplicationUser;
    ```

> [!NOTE]
> Create non-administrator accounts at the database level, unless they need to execute administrator tasks like creating new users.

<a name='azure-ad-authentication'></a>

### Microsoft Entra authentication

Because Azure SQL Database doesn't support Microsoft Entra server principals (logins), database users created with Microsoft Entra accounts are created as contained database users. A contained database user is not associated to a login in the `master` database, even if there exists a login with the same name. The Microsoft Entra identity can either be for an individual user or a group. For more information, see [Contained database users, make your database portable](/sql/relational-databases/security/contained-database-users-making-your-database-portable) and review the [Microsoft Entra tutorial](authentication-aad-configure.md) on how to authenticate using Microsoft Entra ID.

> [!NOTE]
> Database users (excluding administrators) cannot be created using the Azure portal. Microsoft Entra roles do not propagate to SQL servers, databases, or data warehouses. They are only used to manage Azure resources and do not apply to database permissions.
>
> For example, the *SQL Server Contributor* role does not grant access to connect to a database or data warehouse. This permission must be granted within the database using T-SQL statements.

> [!IMPORTANT]
> Special characters like colon `:` or ampersand `&` are not supported in user names in the T-SQL `CREATE LOGIN` and `CREATE USER` statements.

To add a user with Microsoft Entra authentication:

1. Connect to your server in Azure using a Microsoft Entra account with at least the *ALTER ANY USER* permission.

1. In **Object Explorer**, right-click the database and select **New Query**.

1. In the query window, enter the following command and modify `<Azure_AD_principal_name>` to the principal name of the Microsoft Entra user or the display name of the Microsoft Entra group:

   ```sql
   CREATE USER [<Azure_AD_principal_name>] FROM EXTERNAL PROVIDER;
   ```

> [!NOTE]
> Microsoft Entra users are marked in the database metadata with type `E (EXTERNAL_USER)` and type `X (EXTERNAL_GROUPS)` for groups. For more information, see [sys.database_principals](/sql/relational-databases/system-catalog-views/sys-database-principals-transact-sql).

### Secure connection strings

To ensure a secure, encrypted connection between the client application and SQL Database, a connection string must be configured to:

- Request an encrypted connection
- Not trust the server certificate

The connection is established using Transport Layer Security (TLS) and reduces the risk of a man-in-the-middle attack. Connection strings are available per database and are pre-configured to support client drivers such as ADO.NET, JDBC, ODBC, and PHP. For information about TLS and connectivity, see [TLS considerations](connect-query-content-reference-guide.md#tls-considerations-for-database-connectivity).

To copy a secure connection string:

1. In the Azure portal, select **SQL databases** from the left-hand menu, and select your database on the **SQL databases** page.

1. On the **Overview** page, select **Show database connection strings**.

1. Select a driver tab and copy the complete connection string.

    :::image type="content" source="media\secure-database-tutorial\connection.png" alt-text="Screenshot of the Azure portal showing the connection strings page. The ADO.NET tab is selected and the ADO.NET (SQL authentication) connection string is displayed.":::

## Enable security features

Azure SQL Database provides security features that are accessed using the Azure portal. These features are available for both the database and server, except for data masking, which is only available on the database. To learn more, see [Microsoft Defender for SQL](azure-defender-for-sql.md), [Auditing](./auditing-overview.md), [Dynamic data masking](dynamic-data-masking-overview.md), and [Transparent data encryption](transparent-data-encryption-tde-overview.md).

### Microsoft Defender for SQL

The Microsoft Defender for SQL feature detects potential threats as they occur and provides security alerts on anomalous activities. Users can explore these suspicious events using the auditing feature, and determine if the event was to access, breach, or exploit data in the database. Users are also provided a security overview that includes a vulnerability assessment and the data discovery and classification tool.

> [!NOTE]
> An example threat is SQL injection, a process where attackers inject malicious SQL into application inputs. An application can then unknowingly execute the malicious SQL and allow attackers access to breach or modify data in the database.

To enable Microsoft Defender for SQL:

1. In the Azure portal, select **SQL databases** from the left-hand menu, and select your database on the **SQL databases** page.

1. On the **Overview** page, select the **Server name** link. The server page will open.

1. On the **SQL server** page, find the **Security** section and select **Defender for Cloud**.

   1. Select **ON** under **Microsoft Defender for SQL** to enable the feature. Choose a storage account for saving vulnerability assessment results. Then select **Save**.

      :::image type="content" source="media\secure-database-tutorial\threat-settings.png" alt-text="Screenshot of the Azure portal Navigation pane for threat detection settings.":::

      You can also configure emails to receive security alerts, storage details, and threat detection types.

1. Return to the **SQL databases** page of your database and select **Defender for Cloud** under the **Security** section. Here you'll find various security indicators available for the database.

    :::image type="content" source="media\secure-database-tutorial\threat-status.png" alt-text="Screenshot of the Azure portal Threat status page showing pie charts for Data Discovery & Classification, Vulnerability Assessment, and Threat Detection." lightbox="media\secure-database-tutorial\threat-status.png":::

If anomalous activities are detected, you receive an email with information on the event. This includes the nature of the activity, database, server, event time, possible causes, and recommended actions to investigate and mitigate the potential threat. If such an email is received, select the **Azure SQL Auditing Log** link to launch the Azure portal and show relevant auditing records for the time of the event.

   :::image type="content" source="media\secure-database-tutorial\threat-email.png" alt-text="Screenshot of a sample email from Azure, indicating a Potential Sql Injection Threat detection. A link in the body of the email to Azure SQL DB Audit Logs is highlighted.":::

### Auditing

The auditing feature tracks database events and writes events to an audit log in either Azure storage, Azure Monitor logs, or to an event hub. Auditing helps maintain regulatory compliance, understand database activity, and gain insight into discrepancies and anomalies that could indicate potential security violations.

To enable auditing:

1. In the Azure portal, select **SQL databases** from the left-hand menu, and select your database on the **SQL databases** page.

1. In the **Security** section, select **Auditing**.

1. Under **Auditing** settings, set the following values:

   1. Set **Auditing** to **ON**.

   1. Select **Audit log destination** as any of the following:

       - **Storage**, an Azure storage account where event logs are saved and can be downloaded as *.xel* files

          > [!TIP]
          > Use the same storage account for all audited databases to get the most from auditing report templates.

       - **Log Analytics**, which automatically stores events for query or further analysis

           > [!NOTE]
           > A **Log Analytics workspace** is required to support advanced features such as analytics, custom alert rules, and Excel or Power BI exports. Without a workspace, only the query editor is available.

       - **Event Hub**, which allows events to be routed for use in other applications

   1. Select **Save**.

      :::image type="content" source="media\secure-database-tutorial\audit-settings.png" alt-text="Screenshot of the Azure portal Audit settings page. The Save button is highlighted. Audit log destination fields are highlighted.":::

1. Now you can select **View audit logs** to view database events data.

    :::image type="content" source="media\secure-database-tutorial\audit-records.png" alt-text="Screenshot of the Azure portal page showing Audit records for a SQL database." lightbox="media\secure-database-tutorial\audit-records.png":::

> [!IMPORTANT]
> See [SQL Database auditing](./auditing-overview.md) on how to further customize audit events using PowerShell or REST API.

### Dynamic data masking

The data masking feature will automatically hide sensitive data in your database.

To enable data masking:

1. In the Azure portal, select **SQL databases** from the left-hand menu, and select your database on the **SQL databases** page.

1. In the **Security** section, select **Dynamic Data Masking**.

1. Under **Dynamic data masking** settings, select **Add mask** to add a masking rule. Azure will automatically populate available database schemas, tables, and columns to choose from.

    :::image type="content" source="media\secure-database-tutorial\mask-settings.png" alt-text="Screenshot of the Azure portal page to Save or Add Dynamic Data Mask fields. Recommended fields to mask display schema, table, and columns of tables.":::

1. Select **Save**. The selected information is now masked for privacy.

    :::image type="content" source="media\secure-database-tutorial\mask-query.png" alt-text="Screenshot of SQL Server Management Studio (SSMS) showing a simple INSERT and SELECT statement. The SELECT statement displays masked data in the LastName column.":::

### Transparent data encryption

The encryption feature automatically encrypts your data at rest, and requires no changes to applications accessing the encrypted database. For new databases, encryption is on by default. You can also encrypt data using SSMS and the [Always encrypted](always-encrypted-certificate-store-configure.md) feature.

To enable or verify encryption:

1. In the Azure portal, select **SQL databases** from the left-hand menu, and select your database on the **SQL databases** page.

1. In the **Security** section, select **Transparent data encryption**.

1. If necessary, set **Data encryption** to **ON**. Select **Save**.

    :::image type="content" source="media\secure-database-tutorial\encryption-settings.png" alt-text="Screenshot of the Azure portal page to enable Transparent Data Encryption.":::

> [!NOTE]
> To view encryption status, connect to the database using [SSMS](connect-query-ssms.md) and query the `encryption_state` column of the [sys.dm_database_encryption_keys](/sql/relational-databases/system-dynamic-management-views/sys-dm-database-encryption-keys-transact-sql) view. A state of `3` indicates the database is encrypted.

> [!NOTE]
> Some items considered customer content, such as table names, object names, and index names, might be transmitted in log files for support and troubleshooting by Microsoft.

## Related content

- [Try Azure SQL Database for free (preview)](free-offer.md)
- [What's new in Azure SQL Database?](doc-changes-updates-release-notes-whats-new.md)
- [Configure and manage content reference - Azure SQL Database](how-to-content-reference-guide.md)
- [Plan and manage costs for Azure SQL Database](cost-management.md)

> [!TIP]
> **Ready to start developing an .NET application?** This free Learn module shows you how to [Develop and configure an ASP.NET application that queries an Azure SQL Database](/training/modules/develop-app-that-queries-azure-sql/), including the creation of a simple database.

## Next step

Advance to the next tutorial to learn how to implement geo-distribution.

> [!div class="nextstepaction"]
> [Tutorial: Implement a geo-distributed database (Azure SQL Database)](geo-distributed-application-configure-tutorial.md)
