---
title: "Power Automate : Update Author and Editor of a Page"
date: 2024-07-19T09:53:05+01:00
tags: ["Power Automate","Update Author",""Update Editor","REST API","SharePoint"]
featured_image: '/posts/images/powerauomate-get-user-details/GetAuthorDetails.png'
omit_header_text: true
draft: false
---

This post outlines how to use Power Automate to update author and editor of a Page within SharePoint using the `Send an Http request to SharePoint action`.

Within a Power Automate flow, follow the following steps

1. Add `Send an Http request to SharePoint` renamed to `Update Author`

* Site Address: siteUrl
* Method: POST
* Uri : _api/web/lists/GetByTitle('Site%20Pages')/items(1)/ValidateUpdateListItem()

* Headers : content-type: application/json;odata=verbose
            accept : application/json;odata=verbose

* Body :

```json
{"formValues":[{"FieldName":"Author","FieldValue":"[{'key':'@{body('Parse_JSON_Author_Details')?['d']?['LoginName']}'}]"},{"FieldName":"Editor","FieldValue":"[{'key':'body('Parse_JSON_Author_Details')?['d']?['LoginName']}']"}],"bNewDocumentUpdate":true,"checkInComment":null}
```

```json
example of body compiled: {"formValues":[{"FieldName":"Author","FieldValue":"[{'key':'i:0#.f|membership|Reshmee.Auckloo@contoso.co.uk'}]"},{"FieldName":"Editor","FieldValue":"[{'key':'i:0#.f|membership|Reshmee.Auckloo@contoso.co.uk'}]"}],"bNewDocumentUpdate":true,"checkInComment":null}
```

![Update Author](../images/powerauomate-get-user-details/UpdateAuthorDetails.png)

By following these steps, you can update the author and editor fields in a SharePoint list item using Power Automate. This is particularly useful for automating the management of SharePoint content and ensuring that the correct user details are recorded.