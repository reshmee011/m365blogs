---
title: "Effective permissions of end user using across sites PowerShell"
date: 2024-07-10T16:16:09Z
tags: ["SharePoint","PnP","PowerShell","Effective permissions" ,"Security","Permissions"]
featured_image: '/posts/images/PowerShell_PnPUnifiedLog/Sample.png'
omit_header_text: true
draft: true
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
              "ApprovalStatus": "In-Review"
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
    },
    {
      "elmType": "button",
      "customRowAction": {
        "action": "executeFlow",
        "actionParams": "{\"id\":\"93f5e4c1-02e8-44d5-827d-3eaa5e389625\", \"headerText\":\"Submit for Approval\",\"runFlowButtonText\":\"Submit for Approval\"}"
      },
      "attributes": {
        "class": "ms-fontColor-themePrimary ms-fontColor-themeDarker--hover"
      },
      "style": {
        "border": "none",
        "background-color": "transparent",
        "cursor": "pointer",
        "text-align": "left",
        "display": "=if([$ApprovalStatus]=='In-Review','block','none')"
      },
      "children": [
        {
          "elmType": "span",
          "attributes": {
            "iconName": "Flow"
          },
          "style": {
            "padding-right": "6px"
          }
        },
        {
          "elmType": "span",
          "txtContent": "Submit for Approval"
        }
      ]
    },
    {
      "elmType": "button",
      "customRowAction": {
        "action": "executeFlow",
        "actionParams": "{\"id\":\"8ddeebff-84f0-4047-a3a2-e544bebcf3cf\", \"headerText\":\"Please enter the Last Approval Details\",\"runFlowButtonText\":\"Set to Approved\"}"
      },
      "attributes": {
        "class": "ms-fontColor-themePrimary ms-fontColor-themeDarker--hover"
      },
      "style": {
        "border": "none",
        "background-color": "transparent",
        "cursor": "pointer",
        "text-align": "left",
        "display": "=if([$ApprovalStatus] == 'In-Review' && ([$LastApprovedBy]=='' || [$LastApprovedDate]==''),'block','none')"
      },
      "children": [
        {
          "elmType": "span",
          "attributes": {
            "iconName": "Flow"
          },
          "style": {
            "padding-right": "6px"
          }
        },
        {
          "elmType": "span",
          "txtContent": "Set to Approved"
        }
      ]
    },
    {
      "elmType": "div",
      "children": [
        {
          "elmType": "span",
          "txtContent": "='This item is ' + toLowerCase([$ApprovalStatus])",
          "style": {
            "display": "=if([$ApprovalStatus] == 'Pending Approval','block','none')",
            "padding-left": "5px",
            "word-break": "keep-all"
          }
        }
      ]
    }
  ]
}
```