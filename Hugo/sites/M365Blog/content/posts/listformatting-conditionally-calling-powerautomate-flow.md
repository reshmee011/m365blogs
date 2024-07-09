


```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/column-formatting.schema.json",
  "elmType": "button",
  "customRowAction": {
    "action": "executeFlow",
    "actionParams": "='{\"id\": \"' + if([$TrackingStatus]=='New','1811db3a-d75e-40c4-9c5f-6b303e4800c5',if([$RequestValidation]=='Validated' && [$RequestAuthorisation]=='Pending','99fc6be7-1695-4743-bfc0-77de87ae5079',if([$RequestAuthorisation]=='Authorised' && [$FinanceProcessingDate] && [$TrackingStatus]!='Completed','668ff0ab-c0b0-4524-8a86-87c15ada588e',''))) + '\"}'"
  },
  "attributes": {
    "class": "ms-fontColor-themePrimary ms-fontColor-themeDarker--hover"
  },
  "style": {
    "border": "none",
    "background-color": "transparent",
    "cursor": "pointer",
    "text-align": "left",
    "display": "=if([$TrackingStatus]=='New','block',if([$RequestValidation]=='Validated' && [$RequestAuthorisation]=='Pending','block',if([$RequestAuthorisation]=='Authorised' && [$FinanceProcessingDate] && [$TrackingStatus]!='Completed','block','none')))"
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
      "txtContent": "=if([$TrackingStatus]=='New','Submit for Validation',if([$RequestValidation]=='Validated' && [$RequestAuthorisation]=='Pending','Submit for Authorisation',if([$RequestAuthorisation]=='Authorised' && [$FinanceProcessingDate] && [$TrackingStatus]!='Completed','Submit for Sign-Off','')))"
    }
  ]
}
```