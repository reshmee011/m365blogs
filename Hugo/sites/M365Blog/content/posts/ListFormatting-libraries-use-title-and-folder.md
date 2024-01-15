---
title: "ListFormatting_LibrariesProps_Folder_Title"
date: 2024-01-15T16:49:18Z
tages:
draft: true
---

# Column formatting referring to the Name and Folder column
Internal names 
$FileLeafRef - Server Relative Url of the file
$FileRef  - Name of the file

I first tried to use the FileSystemObjectType

## Sample 

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
        "display":"=if(indexOf([$FileRef],'Manual Load')>0 && [$FileSystemObjectType]!='Folder','display-inline','none')"
      },
      "attributes": {
        "iconName": "View",
        "class": "ms-fontColor-themePrimary ms-fontColor-themeDark--hover",
        "title": "View Remittance Document",
        "target": "_blank",
        "href": =@currentWeb +'/RemittanceDocs/Forms/AllItems.aspx?q=listitemid%3A' + substring([$FileLeafRef],1,indexOf([$FileLeafRef],']'))  +'&view=7'
      }
    }
  ]
}

```

The trick is to use the [$ContentTypeId] ensuring it starts with '0x0120'
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/column-formatting.schema.json",
  "elmType": "div",
  "children": [
    {
      "elmType": "span",
      "style": {
        "padding-right": "8px"
      },
      "txtContent": ""=if(startsWith([$FileLeafRef],'['),substring([$FileLeafRef],1,indexOf([$FileLeafRef],']')),'')"
    },
    {
      "elmType": "a",
      "style": {
        "display":"=if(indexOf([$FileRef],'Manual Load')> 0 && indexOf([$ContentTypeId],'0x0120') < 0,'display-inline','none')"
      },
      "attributes": {
        "iconName": "View",
        "class": "ms-fontColor-themePrimary ms-fontColor-themeDark--hover",
        "title": "View Remittance Document",
        "target": "_blank",
        "href": =@currentWeb +'/RemittanceDocs/Forms/AllItems.aspx?q=listitemid%3A' + substring([$FileLeafRef],1,indexOf([$FileLeafRef],']'))  +'&view=7'
      }
    }
  ]
}


Other properties that can be used,[] @thumbnail.medium

[File Thumbnails](https://github.com/pnp/List-Formatting/tree/master/column-samples/file-thumbnail)
