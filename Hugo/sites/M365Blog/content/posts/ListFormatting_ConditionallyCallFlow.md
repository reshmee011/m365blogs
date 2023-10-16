
---
title: "PowerShell_EnsureOwnersAreMembersM365Group"
date: 2023-10-16T09:57:15+01:00
tags: ["ListFomatting","JSON","executeFlow"]
draft: true
---

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