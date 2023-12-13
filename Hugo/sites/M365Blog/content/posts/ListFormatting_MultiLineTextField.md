---
title: "Enhance List Formatting for Multi-Line Text fields challenges"
date: 2023-12-13T13:40:12+01:00
tags: ["List formatting","JSON","HTML","CSS", "MultiLineText"]
featured_image: '/posts/images/ListFormatting_MultiLineTextField/MultiLineViewMore.PNG'
draft: false
---

# Enhance List Formatting for Multi-Line Text fields challenges

## Summary

The following sample shows how you can view more of a truncated multi line text column on hover. Column formatting in SharePoint is a powerful tool to customize how data is displayed. Multi-line text fields, however, present unique challenges when applying these formats. This guide explores a method to expand truncated multi-line text on hover and discusses its limitations.

![Screenshot of sample](../images/ListFormatting_MultiLineTextField/MultiLineViewMore.png)

## Understanding the Concept
The objective is to provide users with a preview of truncated text and enable expansion on hover. This can be achieved using JSON-based column formatting in SharePoint.

### Implementation

This format can be applied to multi line text plain text column.

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

### How It Works

A comment icon is displayed to allow view complete text on click with conditional display if the field is not empty.

## Challenges and limitations

1. User Experience Issues
Edit/View Form: In certain scenarios, the "See More" link may not function as intended within the List Experience.
![Screenshot of sample](../images/ListFormatting_MultiLineTextField/SeeMoreNotWorking.png)

2. Hover Card: The custom hover card might not display in list forms.
3. Read-Only Access: Users with read-only access face limitations in viewing the full text within the list form.
4. Rich Text Fields: Column formatting for multi-line text only supports plain fields and won't function with rich text.
5. Icon which look like Pop Up icon disappeared from the column after column formatting is applied

![popup icon not visible](../images/ListFormatting_MultiLineTextField/popupiconnotvisible.png)

## Conclusion

Enhancing multi-line text column formatting comes with inherent challenges due to the loss of certain out-of-the-box functionalities. While this technique offers expanded text previews, it's crucial to consider the limitations and user experience discrepancies.

## References

[GitHub Discussion: Review the discussion on enhancing multi-line text fields with Tetsuya and Federico Sapia](https://github.com/pnp/List-Formatting/pull/739)
