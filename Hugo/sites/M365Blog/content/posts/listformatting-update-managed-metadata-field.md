---
title: "Update managed metadata field using list formatting"
date: 2024-08-05T16:49:18Z
tags: ["Libraries", "List Formatting", "Column Formatting", "Managed Metadata", "fileref", "file type"]
featured_image: '/posts/images/listformatting-update-managed-metadata-field/Screenshot.png'
omit_header_text: true
draft: true
---

Using column formatting, you are able to update other fields's value using the inline function `setvalue` allowing to build functionaly around lists/libraries without extensive development work and hence might help not to build digital debt.

However updating a managed metadata field is not straight with only specifying the display value.

## Column Fomatting

The column formatting is a combination of online CSS, HTML with JSON. The JSON renders a hyperlink with label 'Set in Review' and on click updates the field 'Approval Status' to 'In-Review' and 'Status' to 'Draft'. 

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/column-formatting.schema.json",
  "elmType": "div",
  "style": {
    "flex-directon": "column",
    "justify-content": "left",
    "align-items": "center",
    "flex-wrap": "nowrap",
    "display": "flow"
  },
 "children": [
    {
      "elmType": "div",
      "style": {
        "display": "=if([$ApprovalStatus] == 'Approved','block','none')",
        "flex-directon": "row",
        "justify-content": "left",
        "align-items": "center",
        "flex-wrap": "wrap"
      },
  "children": [
        {
          "elmType": "button",
          "customRowAction": {
            "action": "setValue",
            "actionInput": {
              "ApprovalStatus": "In-Review",
              "Status": "Draft|997ddd6e-c086-4730-adf0-0ca8f9eeac48;"
            }
          },
          "attributes": {
            "class": "ms-fontColor-themePrimary ms-fontColor-themeDarker--hover"
          },
          "style": {
            "border": "none",
            "background-color": "transparent",
            "cursor": "pointer",
            "display": "flex",
            "flex-directon": "row",
            "justify-content": "left",
            "align-items": "center",
            "flex-wrap": "wrap"
          },
          "children": [
            {
              "elmType": "span",
              "attributes": {
                "iconName": "SkypeCircleCheck"
              },
              "style": {
                "padding": "4px"
              }
            },
            {
              "elmType": "span",
              "txtContent": "Set In Review",
              "style": {
                "word-break": "keep-all"
              }
            }
          ]
        }
      ]
    }
   ]
 }
 ```

The JSON renders an inline button

 ![Set in Review button](../images/listformatting-update-managed-metadata-field/Screenshot.png)

OnVlick it updates 