---
title: "Internal References in SharePoint Column Formatting : Name and Folder fields"
date: 2024-01-15T16:49:18Z
tags: ["libraries", "List Formatting", "Name", "Folder"]
featured_image: '/posts/images/ListFormatting-libraries-use-title-and-folder/Screenshot.png'
draft: false
---

# Internal References in SharePoint Column Formatting : Name and Folder fields

Some internal names of properties like $FileLeafRef and $FileRef can be used within column formatting. 

$FileLeafRef: Represents the server-relative URL of the file.
$FileRef: Signifies the name of the file.

In my initial attempts, I explored incorporating the $FileSystemObjectType to identify whether the object is a folder.

Internal names 
$FileLeafRef - Name of the file
$FileRef  - Server Relative Url of the file

I first tried to use the internal name $FileSystemObjectType to hide the link if it is not a folder, unfortunately $FileSystemObjectType can't be used within column formatting. 

```json
  "display":"=if(indexOf([$FileRef],'Payment Processing')>0 && [$FileSystemObjectType]!='Folder','display-inline','none')"
```

The property $ContentTypeId property can be used instead ensuring that it starts with '0x0120' for a folder if anything condition needs to be applied.

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
"txtContent": Specifies the text content for the span element. The text is generated using a conditional statement (if), which checks if the FileLeafRef column starts with '['. If true, it extracts the text between '[' and ']' using substring and indexOf. Otherwise, it sets the text to an empty string.

"style": Specifies the CSS styles for the a element. The display property is set based on a complex conditional statement. It checks if the FileRef column contains 'Folder_10', the ContentTypeId column does not start with '0x0120', and the FileLeafRef column starts with '['. If all conditions are met, it sets the display property to 'display-inline', otherwise, it sets it to 'none'.

"href": Specifies the URL for the link. It concatenates the current web URL, a fixed path, and dynamic parameters based on the FileLeafRef column.

![ColumnFormattingInAction](../images/ListFormatting-libraries-use-title-and-folder/Sample.gif)



Additional properties that can be used , e.g. @thumbnail.medium, please see an example [File Thumbnails](https://github.com/pnp/List-Formatting/tree/master/column-samples/file-thumbnail)

## References

[Formatting syntax reference](https://learn.microsoft.com/en-us/sharepoint/dev/declarative-customization/formatting-syntax-reference)