---
title: "Update managed metadata field using list formatting"
date: 2024-08-05T16:49:18Z
tags: ["Libraries", "List Formatting", "Column Formatting", "Managed Metadata"]
featured_image: '/posts/images/listformatting-update-managed-metadata-field/SetInReview.png'
omit_header_text: true
draft: false
---

Using column formatting in SharePoint, you can update the values of other fields using the `setValue` function. This approach allows you to create dynamic functionality in your lists and libraries without extensive development, helping to avoid accumulating technical debt.

Updating a Managed Metadata field, however, is not as straightforward as simply specifying the display value. The field expects a specific format that includes the term's ID (termId).

## Understanding Column Formatting
Column formatting in SharePoint involves using a combination of inline CSS, HTML, and JSON to control how fields in lists and libraries are displayed. In this post, I'll walk you through how to render a button that, when clicked, updates the "Approval Status" field to "In-Review" and the "Status" field (a Managed Metadata field) to "Draft."

## The JSON Configuration
The following JSON configuration creates a button labeled "Set in Review." When clicked, it updates the "Approval Status" to "In-Review" and the "Status" (a Managed Metadata field) to "Draft."

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

## How It Works

**Conditionally Displaying the Button**:
The button is displayed only if the "Approval Status" is set to "Approved". This is controlled by the display style in the JSON, using the if statement.

**Updating Fields**:
When the button is clicked, the setValue action triggers, updating the "Approval Status" to "In-Review" and the "Status" field to "Draft." Notice that the "Status" field is a Managed Metadata field. Therefore, its value is updated using a specific format: "Draft|997ddd6e-c086-4730-adf0-0ca8f9eeac48;", where the term ID is crucial.

## Visuals

The JSON configuration renders the following inline button:

![Set in Review button](../images/listformatting-update-managed-metadata-field/SetInReview.png)


## The Managed Metadata Field

Updating Managed Metadata fields via column formatting requires specifying the term ID. This ensures that the field is correctly populated with the corresponding metadata value.

Here is an example of the Managed Metadata field before the update:

 ![MM Field](../images/listformatting-update-managed-metadata-field/MMStatusField.png)

After clicking the 'Set in Review' button, the Managed Metadata field "Status" is updated to "Draft":

![Status change to draft](../images/listformatting-update-managed-metadata-field/SetToDraft.png)

## Conclusion

Using column formatting to update Managed Metadata fields in SharePoint can significantly enhance your list or library's functionality without requiring custom development. 

Feel free to customize the JSON further based on your requirements, and don't forget to test thoroughly in a non-production environment before deploying.