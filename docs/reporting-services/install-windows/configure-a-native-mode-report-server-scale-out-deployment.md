---
title: "Configure a Native mode report server scale-out deployment"
description: "Configure a Native Mode Report Server Scale-Out Deployment"
author: maggiesMSFT
ms.author: maggies
ms.date: 09/25/2024
ms.service: reporting-services
ms.subservice: report-server
ms.topic: conceptual
ms.custom:
  - updatefrequency5
helpviewer_keywords:
  - "report servers [Reporting Services], deployments"
  - "deploying [Reporting Services], scale-out deployment model"
  - "scale-out deployments [Reporting Services]"
---

# Configure a Native mode report server scale-out deployment

[!INCLUDE [ssrs-appliesto](../../includes/ssrs-appliesto.md)] [!INCLUDE [ssrs-appliesto-2017-and-later-enterprise](../../includes/ssrs-appliesto-2017-and-later-enterprise.md)] [!INCLUDE [ssrs-appliesto-pbirs](../../includes/ssrs-appliesto-pbirs.md)]

Reporting Services native mode supports a scale-out deployment model that allows you to run multiple report server instances that share a single report server database. Scale-out deployments are used to increase scalability of report servers to handle more concurrent users and larger report execution loads. It can also be used to dedicate specific servers to process interactive or scheduled reports.

> [!IMPORTANT]
> For Power BI Report Server, you need to configure client affinity (sometimes called sticky sessions or persistence) on the load balancer for any scale-out environment to ensure proper performance and consistent Power BI (PBIX) report functionality.
  
For SQL Server 2016 Reporting Services and earlier, SharePoint mode report servers utilize the SharePoint products infrastructure for scale-out. SharePoint mode scale-out is performed by adding more SharePoint mode report servers to the SharePoint farm. For information on scale-out in SharePoint mode, see [Add another report server to a farm &#40;SSRS scale-out&#41;](../../reporting-services/install-windows/add-an-additional-report-server-to-a-farm-ssrs-scale-out.md).  

> [!NOTE]
> Reporting Services integration with SharePoint is no longer available after SQL Server 2016.
 
  A *scale-out deployment* is used in the following scenarios:  
  
-   As a prerequisite for load balancing multiple report servers in a server cluster. Before you can load balance multiple report servers, you must first configure them to share the same report server database.  
  
-   To segment report server applications on different computers, by using one server for interactive report processing and a second server for scheduled report processing. In this scenario, each server instance processes different types of requests for the same report server content stored in the shared report server database.  
  
 **Scale-out deployments consist of:**  
  
-   Two or more report server instances sharing a single report server database.  
  
-   Optionally, a network load-balanced (NLB) cluster to spread interactive user load across the report server instances.  
  
 When deploying Reporting Services on an NLB cluster, you need to ensure the NLB virtual server name is used in the configuration of report server URLs and that servers are configured to share the same view state.  
  
 Reporting Services doesn't participate in Microsoft Cluster Services clusters. However, you can create the report server database on a Database Engine instance that is part of a failover cluster.  
  
 **To plan, install, and configure a scale-out deployment, follow these steps:**  
  
- Review [Install SQL Server from the Installation Wizard &#40;Setup&#41;](../../database-engine/install-windows/install-sql-server-from-the-installation-wizard-setup.md) for instructions on how to install report server instances.   
- If you're planning to host the scale-out deployment on a network load balanced (NLB) cluster, you should configure the NLB cluster before you configure the scale-out deployment. For more information, see [Configure a report server on a network load balancing cluster](../../reporting-services/report-server/configure-a-report-server-on-a-network-load-balancing-cluster.md).  
  
- Review the procedures in this article for instructions on how to share a report server database and join report servers to a scale-out.  
  
     The procedures explain how to configure a two-node report server scale-out deployment. Repeat the steps described in this article to add more report server nodes to the deployment.  
  
    - Use Setup to install each report server instance that you want to join to the scale-out deployment.  
  
         To avoid database compatibility errors when connecting the server instances to the shared database, be sure that all instances are the same version. For example, if you create the report server database using a SQL Server 2016 report server instance, all other instances in the same deployment must also be SQL Server 2016.  
  
    - Use the Report Server Configuration Manager to connect each report server to the shared database. You can only connect to and configure one report server at a time.
    - Use the Reporting Services Configuration tool to complete the scale-out by joining new report server instances to the first report server instance already connected to the report server database.  
    - Use SQL Server Reporting Services Enterprise Edition. See [SQL Server Reporting Services features supported by editions](../reporting-services-features-supported-by-the-editions-of-sql-server-2016.md) for details.

## Install a SQL Server instance to host the report server databases  
  
1.  Install a [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] instance on a computer that you want to host report server databases. At a minimum, install [!INCLUDE[ssDEnoversion](../../includes/ssdenoversion-md.md)] and [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)].  
  
2.  If necessary, enable the report server for remote connections. Some versions of [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] don't enable remote TCP/IP and Named Pipes connections by default. To confirm whether remote connections are allowed, use [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] Configuration Manager and view the network configuration settings of the target instance. If the remote instance is also a named instance, verify that the [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] Browser service is enabled and running on the target server. [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] Browser provides the port number that is used to connect to the named instance. 

> [!NOTE]
> Configurable named instances are not available in SQL Server Reporting Services 2017 and later or Power BI Report Server. SQL Server Reporting Services 2017 and later will always use the instance name **SSRS**. Power BI Report Server will always be the instance name **PBIRS**.

## Service accounts

The service accounts used for the Reporting Services instance are important when dealing with a scale-out deployment. You should do one of the following options when deploying your Reporting Services instances.

**Option 1:** All of the Reporting Services instances should be configured with the same domain user account for the service account.

**Option 2:** Each individual service account, domain account or not, need to be granted dbadmin permissions within the SQL Server database instance that is hosting the ReportServer catalog database.

If you chose a different configuration than either of the above options, you might encounter intermittent failures of modifying tasks with SQL Agent. These failures show up as an error in both the Reporting Services log and on the web portal when editing a report subscription.

```
An error occurred within the report server database.  This may be due to a connection failure, timeout or low disk condition within the database.
``` 

The issue intermittenly occurs, because only the server that creates the SQL Agent task has rights to view, delete or edit the item. If you don't do one of the above options, the operations only succeed when the load balancer sends all of your requests for that subscription to the server that creates the SQL Agent task. 
  
## Install the first report server instance  
  
1.  Install the first report server instance that is part of the deployment. When you install [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)], choose the **Install but do not configure server** option on the Report Server Installation Options page.  
  
2.  Start the [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)] Configuration tool.  
  
3.  Configure the Report Server Web service URL, Web Portal URL, and the report server database. For more information, see [Configure a report server &#40;Reporting Services Native mode&#41;](../../reporting-services/report-server/configure-a-report-server-reporting-services-native-mode.md)
  
4.  Verify that the report server is operational. For more information, see [Verify a Reporting Services installation](../../reporting-services/install-windows/verify-a-reporting-services-installation.md)  
  
## Install and configure the second report server instance  
  
1.  Run Setup to install a second instance of [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)] on a different computer or as a named instance on the same computer. When you install Reporting Services, choose the **Install but do not configure server** option on the Report Server Installation Options page.  
  
2.  Start the [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)] Configuration tool and connect to the new instance you installed.  
  
3.  Connect the report server to the same database you used for the first report server instance:  
  
    1.  Select **Database** to open the Database page.  
  
    2.  Select **Change Database**.  
  
    3.  Select **Choose an existing report server database**.  
  
    4.  Enter the server name of the SQL Server Database Engine instance that hosts the report server database you want to use. This name must be the same server that you connected to in the previous set of the instructions.  
  
    5.  Select **Test Connection**, and then choose **Next**.  
  
    6.  In **Report Server Database**, select the database you created for the first report server, and then choose **Next**. The default name is ReportServer. Don't select ReportServerTempDB. It's used only for storing temporary data when processing reports. If the database list is empty, repeat the previous four steps to establish a connection to the server.  
  
    7.  In the Credentials page, select the type of account and credentials that the report server you want uses to connect to the report server database. You can use the same credentials as the first report server instance or different credentials. Select **Next**.  
  
    8.  Select **Summary** and then choose **Finish**.  
  
4.  Configure the Report Server **Web service URL**. Don't test the URL yet. It can't resolve until the report server joins to the scale-out deployment.  
  
5.  Configure the **Web Portal URL**. Don't test the URL yet or try to verify the deployment. The report server is unavailable until the report server joins to the scale-out deployment.  
  
## Join the second report server instance to the scale-out deployment  
  
1.  Open the [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)] Configuration tool, and reconnect to the first report server instance. The first report server is already initialized for reversible encryption operations, so it can be used to join more report server instances to the scale-out deployment.  
  
2.  Select **Scale-out Deployment** to open the Scale-out Deployment page. You should see two entries, one for each report server instance that is connected to the report server database. The first report server instance should be joined. The second report server should be "Waiting to join." If you don't see similar entries for your deployment, verify you're connected to the first report server that is already configured and initialized to use the report server database.  

    :::image type="content" source="../../reporting-services/install-windows/media/scaloutscreen.gif" alt-text="Screenshot that partially shows the scale-out deployment page.":::

 
  
3.  On the Scale-out Deployment page, select the report server instance that is waiting to join the deployment, and choose **Add Server**.  
  
    > [!NOTE]  
    >  **Issue:** When you attempt to join a Reporting Services report server instance to the scale-out deployment, you may experience error messages similar to 'Access Denied'.  
    >   
    >  **Workaround:** Back up the [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)] encryption key from the first [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)] instance and restore the key to the second [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)] report server. Then try to join the second server to the [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)] scale-out deployment.  
  
4.  You should now be able to verify that both report server instances are operational. To verify the second instance, you can use the Reporting Services Configuration tool to connect to the report server and select the **Web Service URL** or the **Web Portal URL**.  
  
 If you plan to run the report servers in a load-balanced server cluster, extra configuration is required. For more information, see [Configure a report server on a network load balancing cluster](../../reporting-services/report-server/configure-a-report-server-on-a-network-load-balancing-cluster.md).  

## Related content

- [Configure a service account](configure-the-report-server-service-account-ssrs-configuration-manager.md)
- [Configure a URL](../../reporting-services/install-windows/configure-a-url-ssrs-configuration-manager.md)
- [Create a Native mode report server database](../../reporting-services/install-windows/ssrs-report-server-create-a-native-mode-report-server-database.md)
- [Configure report server URLs](../../reporting-services/install-windows/configure-report-server-urls-ssrs-configuration-manager.md)
- [Configure a report server database connection](../../reporting-services/install-windows/configure-a-report-server-database-connection-ssrs-configuration-manager.md)
- [Add and remove encryption keys for scale-out deployment](../../reporting-services/install-windows/add-and-remove-encryption-keys-for-scale-out-deployment.md)
- [Manage a Reporting Services Native mode report server](../../reporting-services/report-server/manage-a-reporting-services-native-mode-report-server.md)
- [Try asking the Reporting Services forum](/answers/search.html?c=&f=&includeChildren=&q=ssrs+OR+reporting+services&redirect=search%2fsearch&sort=relevance&type=question+OR+idea+OR+kbentry+OR+answer+OR+topic+OR+user)
