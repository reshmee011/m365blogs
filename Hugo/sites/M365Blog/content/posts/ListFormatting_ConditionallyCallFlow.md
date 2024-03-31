
---
title: "Conditionally calls different flows based on conditions"
date: 2023-10-16T09:57:15+01:00
tags: ["ListFomatting","JSON","executeFlow"]
draft: true
---


×

List/Library

TRIGGER FROM SHAREPOINT LIST

. You will need the GUID of your flow. Once your flow is created and
saved you can get the GUID from the URL
. Select the flow and copy the URL of the flow detail screen, specifically
the part shown below in red. That is the GUID
https: //us.flow.microsoft.com/manage/environments/Default-9d3ccee1-5cf8-4e08-
887c-53b2a210967b/flows/de138f88-1e85-5d0f-a3ad-fa9b0c9e5ac5/details

TRIGGER

. You'll also need to add run-only
permissions so others can run it.
. Users and groups can be added
. A SharePoint list or library can be added


Manage run-only permissions

Users and groups SharePoint

Invite a SharePoint list or library
Let others run this flow and see the results, but not edit in any way.
Site

+

Add

Currently shared with
This flow has not been shared with any SharePoint Lists. Add one and see it

# Conditionally calls different flows based on conditions
```JSON
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/column-formatting.schema.json",
  "elmType": "button",
  "customRowAction": {
    "action": "executeFlow",
    "actionParams": "='{\"id\": \"' + if([$Status]=='New','9cf8fe40-3686-48a9-948f-69c2d2e2194f',if([$Status]=='Pending for Approval','10gd7cf9-1695-4743-bfc0-88ef98bf6180',if([$Status]=='Authorised' && [$ProcessingDate],'779gg0ab-c1b1-4625-8b87-98c15ada599f',''))) + '\"}'"
  },
  "attributes": {
    "class": "ms-fontColor-themePrimary ms-fontColor-themeDarker--hover"
  },
  "style": {
    "border": "none",
    "background-color": "transparent",
    "cursor": "pointer",
    "text-align": "left",
    "display": "=if([$Status]=='New','block',if([$Status]=='Pending for Approval','block',if([$Status]=='Approved' && [$ProcessingDate],'block','none')))"
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
      "txtContent": "=if([$Status]=='New','Submit for Validation',if([$Status]=='Pending for Approval','Submit for Approval',if([$Status]=='Approved' && [$ProcessingDate],'Submit for Processing','')))"
    }
  ]
}
```

DISPLAY BUTTON COND

. This button should only be visible if the
content approval status is "Pending".
. Use this JSON code in the Format Column
panel
. Update the color-coded pieces of the code
· Red - my column's system name is
ModerationStatus
. Blue - the value of the status is pending
· In the syntax, a question mark (?) operator
is used to create a condition, like an IF
statement.
. In the "visibility" section of the code
· IF ModerationStatus equals ( == ) Pending,
then "visible", otherwise "hidden"