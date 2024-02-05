---
title: "Internal References in SharePoint Column Formatting for Document libraries: leveraging Name, Folder and Path properties"
date: 2024-01-20T16:49:18Z
tags: ["libraries", "List Formatting", "Name", "fileleafref", "fileref", "file type"]
featured_image: '/posts/images/ListFormatting-libraries-use-title-and-folder/Screenshot.png'
draft: false
---

# SharePoint Column Formatting for Document libraries: Leveraging Name, Folder and Path properties

While referencing [$Title] in lists is straightforward, document libraries demand a nuanced approach. Instead of [$Name], had to delve into the intricacies of internal names like $FileLeafRef and $FileRef when working with column formatting.

**$FileLeafRef**: Denotes the name of the file.

**$FileRef**: Represents the server-relative URL of the file.

Initial attempts to incorporate $FileSystemObjectType for folder identification was challenging, indicating that not all document library internal column names are supported.

```json
  "display":"=if(indexOf([$FileRef],'Processing')>0 && [$FileSystemObjectType]!='Folder','display-inline','none')"
```

Another option would be to use **$File_x0020_Type** property which is the file type (Word Doc, Excel Sheet, etc.) in case you might want to apply any logic based on it.

```json
  "display":"=if([$File_x0020_Type] != '' && indexOf([$FileRef],'Processing') >0 ,'display-inline','none')"
```

However, a more reliable approach involves using the $ContentTypeId property. Validate whether it commences with '0x0120' to effectively discern folders in conditional checks.

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/column-formatting.schema.json",
  "elmType": "div",
  "children": [
    {
      "elmType": "span",
      "style": {
        "padding-right": "8px"
      },
      "txtContent": "=if(startsWith([$FileLeafRef],'['),substring([$FileLeafRef],1,indexOf([$FileLeafRef],']')),'')"
    },
    {
      "elmType": "a",
      "style": {
        "display":"=if(indexOf([$FileRef],'Folder_10')> 0 && indexOf([$ContentTypeId],'0x0120') < 0 && startsWith([$FileLeafRef],'[') ,'display-inline','none')"
      },
      "attributes": {
        "iconName": "View",
        "class": "ms-fontColor-themePrimary ms-fontColor-themeDark--hover",
        "title": "View Remittance Document",
        "target": "_blank",
        "href": "=@currentWeb +'/paymentprocessing/Forms/AllItems.aspx?q=listitemid%3A' + substring([$FileLeafRef],1,indexOf([$FileLeafRef],']'))  +'&view=7'"
      }
    }
  ]
}
```

The above JSON extracts an ID from the Name property between '[' and ']' using functions substring and indexOf to [$FileLeafRef] with a View icon link to the SharePoint file within the library. 

"txtContent" specifies the text content to be displayed. The text is generated using a conditional statement (if), which checks if the FileLeafRef column starts with '['. If true, it extracts the text between '[' and ']' using substring and indexOf. Otherwise, it sets the text to an empty string.

"style" specifies the CSS styles for the a element. The display property is set based on a complex conditional statement. It checks if the FileRef column contains 'Folder_10', the ContentTypeId column does not start with '0x0120', and the FileLeafRef column starts with '['. If all conditions are met, it sets the display property to 'display-inline', otherwise, it sets it to 'none'.

"href": Specifies the URL for the link. It concatenates the current web URL, a fixed path, and dynamic parameters based on the ID extracted from the FileLeafRef column.

![ColumnFormattingInAction](../images/ListFormatting-libraries-use-title-and-folder/Sample.gif)

Additional properties that can be used , e.g. @thumbnail.medium, please see an example [File Thumbnails](https://github.com/pnp/List-Formatting/tree/master/column-samples/file-thumbnail)

## References

[Formatting syntax reference](https://learn.microsoft.com/en-us/sharepoint/dev/declarative-customization/formatting-syntax-reference?wt.mc_id=MVP_308367)

[Open a document in View mode by default- SharePoint](https://answers.microsoft.com/en-us/msoffice/forum/all/open-a-document-in-view-mode-by-default-sharepoint/dc13abe1-97f7-4d1d-83c2-4d97e65d802e?wt.mc_id=MVP_308367)

[File Thumbnails](https://github.com/pnp/List-Formatting/tree/master/column-samples/file-thumbnail?wt.mc_id=MVP_308367)