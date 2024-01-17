---
title: "Internal References in SharePoint Column Formatting for Document libraries: leveraging Name and Folder properties"
date: 2024-01-15T16:49:18Z
tags: ["libraries", "List Formatting", "Name", "Folder"]
featured_image: '/posts/images/ListFormatting-libraries-use-title-and-folder/Screenshot.png'
draft: false
---

# SharePoint Column Formatting for Document libraries: Leveraging Name and Folder properties

While referencing [$Title] in lists is straightforward, document libraries demand a nuanced approach. Instead of [$Name] or [$Title], delve into the intricacies of internal names like $FileLeafRef and $FileRef when working with column formatting.

**$FileLeafRef**: Denotes the name of the file.
**$FileRef**: Represents the server-relative URL of the file.

Initial attempts to incorporate $FileSystemObjectType for folder identification might seem challenging. 

```json
  "display":"=if(indexOf([$FileRef],'Payment Processing')>0 && [$FileSystemObjectType]!='Folder','display-inline','none')"
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
T
"txtContent": Specifies the text content for the span element. The text is generated using a conditional statement (if), which checks if the FileLeafRef column starts with '['. If true, it extracts the text between '[' and ']' using substring and indexOf. Otherwise, it sets the text to an empty string.

"style": Specifies the CSS styles for the a element. The display property is set based on a complex conditional statement. It checks if the FileRef column contains 'Folder_10', the ContentTypeId column does not start with '0x0120', and the FileLeafRef column starts with '['. If all conditions are met, it sets the display property to 'display-inline', otherwise, it sets it to 'none'.

"href": Specifies the URL for the link. It concatenates the current web URL, a fixed path, and dynamic parameters based on the FileLeafRef column.

![ColumnFormattingInAction](../images/ListFormatting-libraries-use-title-and-folder/Sample.gif)

Additional properties that can be used , e.g. @thumbnail.medium, please see an example [File Thumbnails](https://github.com/pnp/List-Formatting/tree/master/column-samples/file-thumbnail)

## References

[Formatting syntax reference](https://learn.microsoft.com/en-us/sharepoint/dev/declarative-customization/formatting-syntax-reference)