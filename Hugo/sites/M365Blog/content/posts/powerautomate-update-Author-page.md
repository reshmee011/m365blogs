---
title: "Power Automate get user details"
date: 2024-07-23T09:53:05+01:00
tags: ["Power Automate","Get User Details","REST API","SharePoint"]
featured_image: '/posts/images/powerauomate-get-user-details/GetAuthorDetails.png'
omit_header_text: true
draft: true
---

1. `Send an Http request to SharePoint` renamed to `Update Author`

Site Address: siteUrl
Method: POST
Uri : _api/web/lists/GetByTitle('Site%20Pages')/items(1)/ValidateUpdateListItem()

Headers : content-type: application/json;odata=verbose
        accept : application/json;odata=verbose

Body :
{"formValues":[{"FieldName":"Author","FieldValue":"[{'key':'@{body('Parse_JSON_Author_Details')?['d']?['LoginName']}'}]"},{"FieldName":"Editor","FieldValue":"[{'key':'\@{body('Parse_JSON_Author_Details')?['d']?['LoginName']}'}]"}],"bNewDocumentUpdate":true,"checkInComment":null}


example of body compiled: {"formValues":[{"FieldName":"Author","FieldValue":"[{'key':'i:0#.f|membership|Reshmee.Auckloo@contoso.co.uk'}]"},{"FieldName":"Editor","FieldValue":"[{'key':'i:0#.f|membership|Reshmee.Auckloo@contoso.co.uk'}]"}],"bNewDocumentUpdate":true,"checkInComment":null}


![Update Author](../images/powerauomate-get-user-details/UpdateAuthorDetails.png)