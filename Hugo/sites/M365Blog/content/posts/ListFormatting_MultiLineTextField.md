---
title: "ListFormatting Expand Multi Line Text"
date: 2023-10-18T13:40:12+01:00
tags: ["List formatting","JSON","HTML","CSS", "MultiLineText"]
featured_image: '/posts/images/PowerAutomate_EmailHtmlGotchas/div_email_owa.PNG'
draft: true
---

# Expand Multi Line Text using column formatting

## Summary

The following sample shows how you can view more of a truncated multi line text column on hover. 

![Screenshot of sample](../images/ListFormatting_MultiLineTextField/MultiLineViewMore.png)

## View requirements

- This format can be applied to multi line text column.


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

## Caveats

1. In List Experience the edit/view form does not render properly with the "See More" link stop to work for multi line text fields
2. The custom hover card does not display in the list form
3. If user had read only access to the view, they can't see the full text and users with edit rights can see the full text in edit mode.
4. Multi line text fields have to be plain fields, won't work on rich text 

## References

[PR discussion for multi line text field ](https://github.com/pnp/List-Formatting/pull/739)
