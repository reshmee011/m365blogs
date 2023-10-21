---
title: "ListFormatting_MultiLineTextField"
date: 2023-10-18T13:40:12+01:00
draft: true
---

```JSON
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/column-formatting.schema.json",
  "elmType": "div",
  "children": [
    {
      "elmType": "span",
      "txtContent": "@currentField",
      "style": {
        "display": "=if(@currentField == '', 'block', 'none')"
      }
    },
    {
      "elmType": "button",
      "attributes": {
        "class": "ms-fontColor-themePrimary ms-fontColor-themeDarker--hover ms-Button"
      },
      "style": {
        "border": "none",
        "background-color": "transparent",
        "cursor": "pointer",
        "display": "=if(@currentField == '', 'none', 'flex')",
        "flex-direction": "row",
        "justify-content": "left",
        "align-items": "center",
        "flex-wrap": "wrap",
        "position": "relative"
      },
      "children": [
        {
          "elmType": "div",
          "txtContent": "=if(@currentField == '', '', substring(@currentField, 0, 101)+'...')",
          "style": {
            "flex-grow": "1",
            "word-break": "break-all"
          }
        },
        {
          "elmType": "span",
          "attributes": {
            "iconName": "comment"
          },
          "style": {
            "padding": "4px",
            "position": "absolute",
            "bottom": "0",
            "right": "0"
          },
          "customCardProps": {
            "formatter": {
              "elmType": "div",
              "txtContent": "@currentField",
              "style": {
                "width": "400px",
                "padding": "15px",
                "word-break": "break-all"
              }
            },
            "openOnEvent": "click",
            "directionalHint": "bottomCenter",
            "isBeakVisible": true,
            "beakStyle": {
              "backgroundColor": "white"
            }
          }
        }
      ]
    }
  ]
}
```