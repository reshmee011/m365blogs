---
title: "Update managed metadata field using list formatting"
date: 2024-08-05T16:49:18Z
tags: ["Libraries", "List Formatting", "Column Formatting", "Managed Metadata", "fileref", "file type"]
featured_image: '/posts/images/listformatting-update-managed-metadata-field/Screenshot.png'
omit_header_text: true
draft: false
---
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
              "PPF_Status": "Draft|997ddd6e-c086-4730-adf0-0ca8f9eeac48;"
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
 ```json
 