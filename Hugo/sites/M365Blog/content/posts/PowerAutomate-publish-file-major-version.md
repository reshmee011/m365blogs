---
title: "PowerAutomate Publish File Major Version"
date: 2024-05-28T09:53:05+01:00
tags: ["Power Automate","File","Major Version"]
featured_image: '/posts/images/PowerAutomate-options-to-get-round-locked-files/changesetting.png'
omit_header_text: true
draft: true
---

# Title

I am using the trigger for a selected file for the Power Automate flow.  To publish the office document (word, excel or powerpoint) as a major version after the document is approved with an approval task, I initially used the checkin endpoint to no avail 

## Solution check in endpoint

![Check in Endpoint](../images/PowerAutomate-options-to-get-round-locked-files/CheckedIn_endpoint_Version.png)

_api/web/GetFileByServerRelativePath(DecodedUrl='@{body('Get_site_server_relative_url')?['d']?['ServerRelativeUrl']}/@{body('Get_file_properties')?['{FullPath}']}')/CheckIn(comment='COMMENTS%20FOR%20PUBLISH',checkintype=1)

The endpoint will work only if the file is checked out otherwise the message will be rendered. 

{
  "status": 423,
  "message": "The file \"ControlledDocuments/Sample Document 10.docx\" is not checked out.\r\nclientRequestId: 455056b3-ad8b-4da4-bbf4-6f7d3cb7fd71\r\nserviceRequestId: d8473aa1-4086-9000-5f10-bcce28eea893",
  "source": "https://ppfonline.sharepoint.com/teams/d-team-playground/_api/web/GetFileByServerRelativePath(DecodedUrl='/teams/d-team-playground/ControlledDocuments/Sample%20Document%2010.docx')/CheckIn(comment='COMMENTS%20FOR%20PUBLISH',checkintype=1)",
  "errors": [
    "-2147024738",
    "Microsoft.SharePoint.SPFileCheckOutException"
  ]
}

Please check [How to Publish Major Version in SharePoint using Microsoft Power Automate](https://powerusers.microsoft.com/t5/Power-Apps-Community-Blog/How-to-Publish-Major-Version-in-SharePoint-using-Microsoft-Power/ba-p/1622788) how to checkout and checkin a file if it is required in your scenerio

## Solution Publish endpoint

The alternative option is to use the publish endpoint.

![Publish endpoint](../images/PowerAutomate-options-to-get-round-locked-files/PublishFileMajorVersion.png)

To work out the serverrelativeUrl of the site use endpoint /_api/web/ServerRelativeUrl

![Get site relative url](../images/PowerAutomate-options-to-get-round-locked-files/GetSiteRelativeUrl.png)

**Send an HTTP request to SharePoint** Action

site address :

Method: Post
uri _api/web/GetFileByServerRelativePath(DecodedUrl='@{body('Get_site_server_relative_url')?['d']?['ServerRelativeUrl']}/@{body('Get_file_properties')?['{FullPath}']}')/Publish()


Headers
accept : application/json; odata=verbose
Content-Type : application/json; odata=verbose

body: 

Published version
![Published Version](../images/PowerAutomate-options-to-get-round-locked-files/PublishedVersion.png)

## Issue(s)
2. {
  "status": 400,
  "message": "serverRelativeUrl\r\nParameter name: Specified value is not supported for the serverRelativeUrl parameter.\r\nclientRequestId: def6bc2b-b78a-4cfb-83ab-1de90f5b1e91\r\nserviceRequestId: 0b3f3aa1-10cd-9000-35f1-9f406a04d79d",
  "source": "https://ppfonline.sharepoint.com/teams/d-team-playground/_api/web/GetFileByServerRelativeUrl('ControlledDocuments/Sample%20Document%2010.docx')/CheckIn(comment='COMMENTS%20FOR%20PUBLISH',checkintype=1)",
  "errors": [
    "-2147024809",
    "System.ArgumentException"
  ]
}

1. {
  "status": 400,
  "message": "Cannot deserialize data for type Microsoft.SharePoint.SPResourcePath.\r\nclientRequestId: 76bf623f-bfca-4aad-a036-6dfd04bdc779\r\nserviceRequestId: 5a443aa1-a016-9000-5f10-b1a2e6f5b6de",
  "source": "https://ppfonline.sharepoint.com/teams/d-team-playground/_api/web/GetFileByServerRelativePath('/teams/d-team-playground/ControlledDocuments/Sample%20Document%2010.docx')/Publish('Major%20publish')",
  "errors": [
    "-1",
    "Microsoft.SharePoint.Client.InvalidClientQueryException"
  ]
}

Ensure the decodedurl is used within GetFileByServerRelativePath

```

## Workaround



There are few workarounds mainly around looping through updates until successful updates

[4 Solutions For Excel File Is Locked Error In Power Automate](https://www.matthewdevaney.com/4-solutions-for-excel-file-is-locked-error-in-power-automate/#:~:text=The%20condition%20action%20checks%20the,Otherwise%2C%20the%20loop%20will%20end.)

• Solution #1 - Loop Until The Excel File Is Unlocked
• Solution #2 - Delete The Excel File & Bypass The Shared Use Lock
• Solution #3 - Copy Excel File To Temp Folder To Read The Rows
• Solution #4 - Write A Filled-In Excel Template To Another Folder


