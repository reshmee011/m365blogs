---
title: "Handling Locked Office Files issue In Power Automate"
date: 2024-07-09T09:53:05+01:00
tags: ["Power Automate","Locked Files","REST","validateUpdateListItem"]
featured_image: '/posts/images/PowerAutomate-options-to-get-round-locked-files/FileLockedIssue.png'
omit_header_text: true
draft: true
---

# Handling Locked Office Files issue In Power Automate

Inspired by the workaround described by Pieter Veenstra [System Updates in SharePoint from Power Automate using the ValidateUpdateListItem endpoint](https://sharepains.com/2024/01/05/system-update-sharepoint-power-automate) to help with the locked file issue, this post covers file version creation as well.

## File Locked Issue

File lock issues in Power Automate can occur due to:

* The file being opened by a user.
* The file being updated by a Power Automate flow, which may take up to 6 minutes to release the lock. Recently, it has taken even longer, increasing the flow completion time, especially if updates are made in a loop while waiting for the file to be unlocked. For more details, see  [Speed of SharePoint updates](https://powerusers.microsoft.com/t5/Building-Flows/Speed-of-Sharepoint-updates/td-p/2557990).

### File Locked Error
 
The error within Power Automate flow is 

[locked file issue](../images/PowerAutomate-options-to-get-round-locked-files/FileLockedIssue.png)

{{< warning >}}
The file "https://contoso.sharepoint.com/teams/d-team/ControlledDocuments/Sample Document 10.docx" is locked for shared use by srv_p@.co.uk.
clientRequestId: c22d1dbd-a9eb-4be2-8065-67f9a0a0cada
serviceRequestId: 443d3aa1-c0d7-9000-4dab-908580287932
{{< /warning >}}

## Solution : Use the validateUpdateListItem() REST Endpoint

To resolve the locked file issue, use the **Send an HTTP request to SharePoint** action to update the file as follows:

* site address : <SharePoint site url>
* uri: _api/web/lists/getbytitle('Controlled Documents')/items(triggerBody()?['entity']?['ID'])/validateUpdateListItem()

* Headers
accept : application/json; odata=verbose
Content-Type : application/json; odata=verbose

body
```json
{
"formValues":[{ "FieldName": "ApprovalStatus", "FieldValue": "Pending Approval" },{ "FieldName": "LastReviewDate", "FieldValue":"formatDateTime(utcNow(), 'dd-MM-yyyy')"},
{"FieldName": "Editor",
 "FieldValue":"[{'Key':'body('Get_file_properties')?['Editor']?['Claims']'}]"}
],
 "bNewDocumentUpdate":false
}
```

[Http with Editor](../images/PowerAutomate-options-to-get-round-locked-files/UseHttp_for_SharePoint_SpecifyingEditor.png)

This action updates a file selected by the Power Automate trigger `for a selected file` with the status "Pending Approval" and sets LastReviewDate to the current date and time.

> Include the Editor , otherwise the file locked error message will be thrown

The `bNewDocumentUpdate` parameter determines whether new versions are created. If set to true, no new versions are created. However, for auditing purposes, versions were required even if changes were made by the flow. Therefore, `bNewDocumentUpdate` is set to false.

[versions saved](../images/PowerAutomate-options-to-get-round-locked-files/VersionsSave.png)

## Conclusion

Handling locked files in Power Automate is essential for ensuring smooth flow operations. This workaround not only helps resolve the locked file issue but also ensures that file versions are created and maintained for auditing purposes.