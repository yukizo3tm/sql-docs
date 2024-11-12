---
title: Known issues in SQL Server Management Studio
description: Learn about known issues in SQL Server Management Studio (SSMS).
author: erinstellato-ms
ms.author: erinstellato
ms.reviewer: randolphwest, maghan
ms.date: 11/12/2024
ms.service: sql
ms.subservice: ssms
ms.topic: troubleshooting-general
---
# Known issues in SQL Server Management Studio 21 Preview

[!INCLUDE [sql-asdb-asdbmi-asa](../../includes/applies-to-version/sql-asdb-asdbmi-asa.md)]

This page lists known issues for [!INCLUDE [ssms-21-md](../includes/ssms-21-md.md)].

| Feature | Details | Workaround |
| --- | --- | --- |
| Analysis Services | There's currently no support for Analysis Services in [!INCLUDE [ssms-21-md](../includes/ssms-21-md.md)]. | Use SSMS 20.2 to connect to Analysis Services, we'll bring back Analysis Services in a future preview. |
| Arm64 | There's no support for Arm64, and SSMS can't be installed on Arm64 devices. | Run SSMS on a device that isn't Arm64. |
| Connection | If SSMS is configured to open Object Explorer, Query Editor, or Activity Monitor on startup, in **Tools** > **Options** > **Startup**, the connection dialog might appear while SSMS is still loading. | Configure SSMS to open to an empty environment. |
| Installer | The **Product version** listed on the **Details** page within the **Properties** of the bootstrapper `vs_ssms.exe` is incorrect. | There's no current workaround, we'll address this in a future preview. |
| Integration Services | There's currently no support for Integration Services in [!INCLUDE [ssms-21-md](../includes/ssms-21-md.md)]. | Use SSMS 20.2 to connect to Integration Services, we'll bring back Integration Services in a future preview. |
| License | The "Microsoft Software License Terms" link in the Visual Studio Install splash screen doesn't access a valid URL. | Access the license for SSMS 21 from the "license" link within the Visual Studio Installer, available when customizing the SSMS 21 installation. |
| Maintenance Plans | There's currently no support for creating or editing maintenance plans in [!INCLUDE [ssms-21-md](../includes/ssms-21-md.md)]. | Use SSMS 20.2 to create or edit maintenance plans, we'll bring back Integration Services in a future preview. |
| Menu | Opening a folder from **File** > **Recent Projects and Solutions** generates the error "An exception of type NullReferenceException has been encountered." if opening the folder also opens one or more files that were open in the editor when the folder was last closed. | After closing the error, work can continue. Alternatively, close all files in the editor before closing a folder. |
| Menu | The **File** > **Add** > **New Project or Existing Project** menu option isn't available. | Use SSMS 20.2 to create a new project or open an existing one. |
| Menu | The **File** > **File** menu option isn't available. | Use **File** > **New Query with current connection** or the New Query button to open a new query editor. |
| Menu | The **File** > **Open** > **Project/Solution** menu option isn't available. | Use SSMS 20.2 to open an existing one. |
| Object Explorer | Selecting "Filter" in Object Explorer for any object causes the error "Could not load file or assembly 'Microsoft.DataWarehouse, Version=16.2.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91' or one of its dependencies. The system can't find the file specified. | Use SSMS 20.2 to filter. |
| Query Editor | Within the query editor in SSMS 21, there's a new file health indicator in the bottom left of the toolbar, which displays errors and warnings. The errors and warnings aren't intuitive for SQL users. | To remove the indicator, go to **Tools**> **Options** > **Text Editor** and uncheck **Show file health indicator**. |
| Settings | There's no ability to import settings from an earlier version of SSMS when you first launch SSMS 21. | There's no current workaround. We plan to address this in a future preview. |
| Results Grid | Tabs and script splitter bar aren't themed. | These will be properly themed in the next preview. |
| Icon | If the SSMS 21 icon is pinned to the Windows task bar, another entry appears in the task bar when SSMS is launched. | Don't pin SSMS 21 to the task bar. |

## Related content

- [Install SQL Server Management Studio 21 Preview](../install/install.md)
