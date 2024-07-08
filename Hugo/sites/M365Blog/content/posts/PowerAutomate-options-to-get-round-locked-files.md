---
title: "PowerAutomate Solutions For Office File Is Locked Error In Power Automate"
date: 2024-05-28T09:53:05+01:00
tags: ["Power Automate","MultiLine Text field"]
featured_image: '/posts/images/PowerAutomate-options-to-get-round-locked-files/changesetting.png'
omit_header_text: true
draft: true
---

# Workaround

## Issue

```
The file "https://ppfonline.sharepoint.com/teams/d-team-playground/ControlledDocuments/Sample Document 10.docx" is locked for shared use by srv_p_tncs@ppf.co.uk.
clientRequestId: c22d1dbd-a9eb-4be2-8065-67f9a0a0cada
serviceRequestId: 443d3aa1-c0d7-9000-4dab-908580287932
```

## Workaround

site address :

uri _api/web/lists/getbytitle('Controlled Documents')/items(@{triggerBody()?['entity']?['ID']})/validateUpdateListItem()

Headers
accept : application/json; odata=verbose
Content-Type : application/json; odata=verbose

body
{
"formValues":[{ "FieldName": "ApprovalStatus", "FieldValue": "Pending Approval" },{ "FieldName": "LastReviewDate", "FieldValue":"@{formatDateTime(utcNow(), 'dd-MM-yyyy')}"},
{"FieldName": "Editor",
 "FieldValue":"[{'Key':'@{body('Get_file_properties')?['Editor']?['Claims']}'}]"}
],
 "bNewDocumentUpdate":false
}

There are few workarounds mainly around looping through updates until successful updates

[4 Solutions For Excel File Is Locked Error In Power Automate](https://www.matthewdevaney.com/4-solutions-for-excel-file-is-locked-error-in-power-automate/#:~:text=The%20condition%20action%20checks%20the,Otherwise%2C%20the%20loop%20will%20end.)

• Solution #1 - Loop Until The Excel File Is Unlocked
• Solution #2 - Delete The Excel File & Bypass The Shared Use Lock
• Solution #3 - Copy Excel File To Temp Folder To Read The Rows
• Solution #4 - Write A Filled-In Excel Template To Another Folder


