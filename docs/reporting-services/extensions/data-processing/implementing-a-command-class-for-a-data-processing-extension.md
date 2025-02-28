---
title: "Implement a Command class for a data processing extension"
description: Learn how to implement a Command class for a data processing extension so that the extension can formulate requests and pass them on to the data source.
author: maggiesMSFT
ms.author: maggies
ms.date: 09/25/2024
ms.service: reporting-services
ms.subservice: extensions
ms.topic: reference
ms.custom:
  - updatefrequency5
helpviewer_keywords:
  - "data processing extensions [Reporting Services], commands"
  - "Command class"
  - "commands [Reporting Services]"
---
# Implement a Command class for a data processing extension
  The **Command** object formulates a request and passes it on to the data source. The command text can take many different syntactical forms, including text and XML. If results are returned, the **Command** object returns results as a **DataReader** object.  
  
 To create a **Command** class, implement <xref:Microsoft.ReportingServices.DataProcessing.IDbCommand>. Implement the <xref:Microsoft.ReportingServices.DataProcessing.IDbCommand.ExecuteReader%2A> method to return a result set as a **DataReader** object. The <xref:Microsoft.ReportingServices.DataProcessing.IDbCommand.ExecuteReader%2A> method of your **Command** class should include an implementation that takes a <xref:Microsoft.ReportingServices.DataProcessing.CommandBehavior> enumeration as an argument. If you deploy your data processing extension to Report Designer, provide an implementation that handles a <xref:Microsoft.ReportingServices.DataProcessing.CommandBehavior.SchemaOnly> case in the <xref:Microsoft.ReportingServices.DataProcessing.IDbCommand.ExecuteReader%2A> method. A schema-only implementation is used to supply Report Designer with a fields list. The **DataReader** object returned by the <xref:Microsoft.ReportingServices.DataProcessing.IDbCommand.ExecuteReader%2A> method needs to contain type and name information for the fields, or columns, in your result set.  
  
 Optionally, your **Command** class can implement <xref:Microsoft.ReportingServices.DataProcessing.IDbCommandAnalysis>. This interface enables an implementing class to analyze a query and return a list of parameters in the query. The functionality of the <xref:Microsoft.ReportingServices.DataProcessing.IDbCommandAnalysis> interface is only used in Report Designer. When you implement <xref:Microsoft.ReportingServices.DataProcessing.IDbCommandAnalysis>, you enable users of Report Designer to be prompted for parameters whenever a report is run in preview mode. In addition, you can view the parameters in the **Parameters** tab of the **Data Set** dialog.  
  
> [!NOTE]  
>  You should not implement <xref:Microsoft.ReportingServices.DataProcessing.IDbCommandAnalysis> if your custom data processing extension does not support parameters.  
  
 For a sample **Command** class implementation, see [SQL Server Reporting Services Product Samples](https://go.microsoft.com/fwlink/?LinkId=177889).  
  
## Related content

- [Reporting Services extensions](../../../reporting-services/extensions/reporting-services-extensions.md)
- [Implement a data processing extension](../../../reporting-services/extensions/data-processing/implementing-a-data-processing-extension.md)
- [Reporting Services extension library](../../../reporting-services/extensions/reporting-services-extension-library.md)
