---
title: "Export a paginated report to a PDF file (Report Builder)"
description: In Report Builder, the PDF rendering extension renders paginated reports to files that can be opened in Adobe Acrobat and other PDF viewers.
author: maggiesMSFT
ms.author: maggies
ms.date: 09/25/2024
ms.service: reporting-services
ms.subservice: report-builder
ms.topic: conceptual
ms.custom:
  - updatefrequency5
---
# Export a paginated report to a PDF file (Report Builder)

[!INCLUDE[ssrs-appliesto](../../includes/ssrs-appliesto.md)] [!INCLUDE [ssrs-appliesto-ssrs-rb](../../includes/ssrs-appliesto-ssrs-rb.md)] [!INCLUDE [ssrs-appliesto-pbi-rb](../../includes/ssrs-appliesto-pbi-rb.md)] [!INCLUDE [ssrb-applies-to-ssdt-yes](../../includes/ssrb-applies-to-ssdt-yes.md)]

The PDF rendering extension renders paginated reports to files that can be opened in Adobe Acrobat and other non-Microsoft PDF viewers that support PDF 1.3. Although PDF 1.3 is compatible with Adobe Acrobat 4.0 and later versions, [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)] supports Adobe Acrobat 11.0 or later. The rendering extension doesn't require Adobe software to render the report. However, PDF viewers such as Adobe Acrobat are required to view or print a report in PDF format.

The PDF rendering extension supports ANSI characters and can translate Unicode characters from Japanese, Korean, Traditional Chinese, Simplified Chinese, Cyrillic, Hebrew, and Arabic with certain limitations. For more information about the limitations, see [Export reports (Report Builder and SSRS)](../../reporting-services/report-builder/export-reports-report-builder-and-ssrs.md). The PDF rendering extension also conforms to  ISO 14289-1 (PDF/UA) standards for Accessible PDF. See [PDF rendering extension conformance to ISO 14289-1, Power BI Report Server & SSRS](/power-bi/report-server/rendering-extension-support) for details.

The PDF renderer is a physical page renderer and, therefore, has pagination behavior that differs from other renderers such as HTML and Excel. This article provides PDF renderer-specific information and describes exceptions to the rules.

> [!NOTE]  
> [!INCLUDE[ssRBRDDup](../../includes/ssrbrddup-md.md)]

## <a id="FontRequirements"></a> Embed fonts

When possible, the PDF rendering extension embeds the subset of each font needed to display the report in the PDF file. Fonts that are used in the report must be installed on the report server. When the report server generates a report in PDF format, it uses the information stored in the font referenced by the report to create character mappings within the PDF file. If the referenced font isn't installed on the report server, the resulting PDF file might not contain the correct mappings and might not display correctly when viewed.

Fonts are embedded in the PDF file when the following conditions apply:

- The font author grants font embedding privileges. Installed fonts include a property that indicates whether the font author intends to allow embedding a font in a document. If the property value is `EMBED_NOEMBEDDING`, the font isn't embedded in the PDF file. For more information, search for "TTGetEmbeddingType" on `msdn.microsoft.com`.

- The font is `TrueType`.

- Visible items reference fonts in a report. If a font is referenced by an item that has the Hidden property set to **True**, the font isn't needed to display rendered data and isn't included in the file. Fonts are embedded only when they're needed to display the rendered report data.

If all of these conditions are met for a font, the font is embedded in the PDF file. If one or more of these conditions isn't met, the font isn't embedded in the PDF file.

> [!NOTE]  
> Although the conditions are met, there is one circumstance under which fonts are not embedded in the PDF file. If the fonts used are the ones in the PDF specification that are commonly known as standard type 1 fonts or the base fourteen fonts, then fonts aren't embedded for ANSI content.

### Fonts on the client computer

When a font is embedded in the PDF file, the computer that is used to view the report doesn't need to have the font installed for the report to display correctly.

When a font isn't embedded in the PDF file, the client computer must have the correct font installed for the report to display correctly. If the font isn't installed on the client computer, the PDF file displays a question mark character (`?`) for unsupported characters.

### Verify fonts in a PDF file

Differences in PDF output occur most often when a font that doesn't support non-Latin characters is used in a report and then non-Latin characters are added to the report. You should test the PDF rendering output on both the report server and the client computers to verify that the report renders correctly.

Don't rely on viewing the report in Preview or exporting to HTML. The report looks correct due to automatic font substitution performed by Report Builder or by the browser, respectively. If Unicode Glyphs are missing on the server, you might see characters replaced with a question mark (`?`). If a font is missing on the client, you might see characters replaced with boxes (`□`).

The fonts that are embedded in the PDF file are included in the `Fonts` property that is saved with the file, as metadata.

Windows 10 and 11 introduced a recommended Universal Windows Platform (UWP) font set that's common across all editions that support UWP, including Desktop, Server, and Xbox. Check this list for supported fonts: [Font list Windows 11 - typography](/typography/fonts/windows_11_font_list#introduction).

> [!IMPORTANT]  
> When using paginated reports in the Power BI service and exporting to a PDF file, the only fonts supported are those included in the **Introduction** font list of [Font list Windows 11 - typography](/typography/fonts/windows_11_font_list#introduction).

## <a id="Metadata"></a> Metadata

In addition to the report layout, the PDF rendering extension writes the following metadata to the PDF Document Information Dictionary.

| PDF property | Created from |
| --- | --- |
| **Title** | The **Name** attribute of the **Report** RDL element. |
| **Author** | The **Author** RDL element. |
| **Subject** | The **Description** RDL element. |
| **Creator** | [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)] product name and version. |
| **Producer** | Rendering extension name and version. |
| **CreationDate** | Report execution time in PDF **datetime** format. |

## <a id="Interactivity"></a> Interactivity

Some interactive elements are supported in PDF. The following section is a description of specific behaviors.

### Show and hide

Dynamic show and hide elements aren't supported in PDF. The PDF document is rendered to match the current state of any items in the report. For example, if the item is displayed when the report is run initially, then the item is rendered. Images that can be toggled aren't rendered, if they're hidden when the report is exported.

### Document map

If there are any document map labels present in the report, a document outline is added to the PDF file. Each document map label appears as an entry in the document outline in the order that it appears in the report. In Acrobat, a target bookmark is added to the document outline only if the page it's on is rendered.

If only a single page is rendered, no document outline is added. The document map is arranged hierarchically to reflect the level of nesting in the report. The document outline is accessible in Acrobat under the **Bookmarks** tab. Selecting an entry within the document outline causes the document to go to the bookmarked location.

### Bookmarks

Bookmarks aren't supported in PDF rendering.

### Drillthrough links

Drillthrough links aren't supported in PDF rendering. The drillthrough links aren't rendered as selectable links and drillthrough reports can't connect to the target of the drillthrough.

### Hyperlinks

Hyperlinks in reports are rendered as selectable links in the PDF file. When selected, Acrobat opens the default client browser and navigates to the hyperlink URL.

## <a id="Compression"></a> Compression

Image compression is based on the original file type of the image. The PDF rendering extension compresses PDF files by default.

To preserve any compression for images included in the PDF file when possible, JPEG images are stored as JPEG and all other image types are stored as BMP.

> [!NOTE]  
> - PDF files don't support embedding PNG images.
> - Reporting Services PDF exports don't support CMYK color format images.

## <a id="DeviceInfo"></a> Device information settings

You can change some default settings for this renderer by changing the device information settings. For more information, see [PDF device information settings](../../reporting-services/pdf-device-information-settings.md).

## Related content

- [Pagination in Reporting Services (Report Builder  and SSRS)](../../reporting-services/report-design/pagination-in-reporting-services-report-builder-and-ssrs.md)
- [Renderer behaviors (Report Builder  and SSRS)](../../reporting-services/report-design/rendering-behaviors-report-builder-and-ssrs.md)
- [Interactive functionality for different report rendering extensions (Report Builder and SSRS)](../../reporting-services/report-builder/interactive-functionality-different-report-rendering-extensions.md)
- [Render report items (Report Builder and SSRS)](../../reporting-services/report-design/rendering-report-items-report-builder-and-ssrs.md)
- [Tables, matrices, and lists (Report Builder and SSRS)](../../reporting-services/report-design/tables-matrices-and-lists-report-builder-and-ssrs.md)
