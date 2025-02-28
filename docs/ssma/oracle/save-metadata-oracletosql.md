---
title: "Save Metadata  (OracleToSQL)"
description: "Save Metadata  (OracleToSQL)"
author: cpichuka
ms.author: cpichuka
ms.date: "01/19/2017"
ms.service: sql
ms.subservice: ssma
ms.topic: conceptual
ms.collection:
  - sql-migration-content
---
# Save Metadata  (OracleToSQL)
The **Save Metadata** dialog box prompts you to load metadata into your SSMA project before saving it. This lets you have a complete project file that you can use offline and send to other people, such as technical support personnel.  
  
To access the **Save Metadata** dialog box, save the project. If any metadata is missing, SSMA will display the **Save Metadata** dialog box.  
  
## Options  
**Name**  
The name of each database in the project.  
  
**Status**  
Indicates if metadata is loaded into the SSMA project, or if metadata is missing.  
  
SSMA loads metadata into the project as necessary. Metadata is loaded automatically when you browse metadata and convert schemas.  
  
**Select All**  
Selects all listed databases.  
  
**Clear**  
Clears the check box for all databases with missing metadata. You cannot clear the check box if a metadata has been loaded.  
  
**Save**  
Saves the project, loading metadata for selected databases that have missing metadata.  
  
**Cancel**  
Cancels the save operation. Missing metadata is not loaded into the project.  
  
