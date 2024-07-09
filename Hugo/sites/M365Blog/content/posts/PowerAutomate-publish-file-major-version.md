---
title: "Publishing Major Versions of Files in SharePoint with Power Automate"
date: 2024-07-07T09:53:05+01:00
tags: ["Power Automate","File","Major Version","SharePoint","REST","Send an HTTP request to SharePoint"]
featured_image: '/posts/images/PowerAutomate-publish-file-major-version/PublishFileMajorVersion.png'
omit_header_text: true
draft: false
---

# Publishing Major Versions of Files in SharePoint with Power Automate

This post covers how to publish major versions for Office documents (Word, Excel, or PowerPoint) in SharePoint using Power Automate, especially after an approval task using the checkin and publish REST endpoints. This applies to libraries having minor versions enabled

Just for context, a Power Automate approval flow with trigger `for a selected file` needed publishing for the selected file as major version after being approved.

## checkin REST endpoint

The checkin endpoint can be used to publish as major version using **Send an HTTP request to SharePoint** action.

{{< warning >}}
 _api/web/GetFileByServerRelativePath(DecodedUrl='@{body('Get_site_server_relative_url')?['d']?['ServerRelativeUrl']}/@{body('Get_file_properties')?['{FullPath}']}')/CheckIn(comment='COMMENTS%20FOR%20PUBLISH',checkintype=1)
{{< /warning >}}

![checkin endpoint](../images/PowerAutomate-publish-file-major-version/CheckedIn_endpoint_Version.png)

However it can lead to issues if the file isn't already checked out, leading to errors and a failure to publish the document as a major version.

{{< warning >}}
 {
  "status": 423,
  "message": "The file \"ControlledDocuments/Sample Document 10.docx\" is not checked out.\r\nclientRequestId: 455056b3-ad8b-4da4-bbf4-6f7d3cb7fd71\r\nserviceRequestId: d8473aa1-4086-9000-5f10-bcce28eea893",
  "source": "https://contoso.sharepoint.com/teams/d-team/_api/web/GetFileByServerRelativePath(DecodedUrl='/teams/d-team-playground/ControlledDocuments/Sample%20Document%2010.docx')/CheckIn(comment='COMMENTS%20FOR%20PUBLISH',checkintype=1)",
  "errors": [
    "-2147024738",
    "Microsoft.SharePoint.SPFileCheckOutException"
  ]
}
{{< /warning >}}


For a detailed guide on how to check out and check in a file as required, refer to [How to Publish Major Version in SharePoint using Microsoft Power Automate](https://powerusers.microsoft.com/t5/Power-Apps-Community-Blog/How-to-Publish-Major-Version-in-SharePoint-using-Microsoft-Power/ba-p/1622788)

## Publish REST endpoint

The alternative option is to use the publish endpoint, which circumvents the need for checkout and checkin processes.

![Publish endpoint](../images/PowerAutomate-publish-file-major-version/PublishFileMajorVersion.png)

Add and configure the **Send an HTTP request to SharePoint** Action as follows

* Site address : Choose SharePoint site
* Method: Post
* uri:  _api/web/GetFileByServerRelativePath(DecodedUrl='body('Get_site_server_relative_url')?['d']?['ServerRelativeUrl']/body('Get_file_properties')?['{FullPath}']')/Publish()
Headers
    Accept : application/json; odata=verbose
    Content-Type : application/json; odata=verbose

* body: 

 A message parameter can be passed to the Publish endpoint as follows: 

> _api/web/GetFileByServerRelativePath(DecodedUrl='body('Get_site_server_relative_url')?['d']?['ServerRelativeUrl']/body('Get_file_properties')?['{FullPath}']')/Publish('Publish post approval')


To work out the server relative Url of the site use endpoint `_api/web/ServerRelativeUrl` using **Send an HTTP request to SharePoint** Action

![Get site relative url](../images/PowerAutomate-publish-file-major-version/GetSiteRelativeUrl.png)

Published version
![Published Version](../images/PowerAutomate-publish-file-major-version/PublishedVersion.png)

## Issues and Solutions

When working with the publish endpoint, you might encounter errors related to the serverRelativeUrl parameter or issues with deserializing data. These errors often stem from incorrect formatting of the URL or issues with the SharePoint API.

### Incorrect parameter format for GetFileByServerRelativeUrl
> {
  "status": 400,
  "message": "serverRelativeUrl\r\nParameter name: Specified value is not supported for the serverRelativeUrl parameter.\r\nclientRequestId: def6bc2b-b78a-4cfb-83ab-1de90f5b1e91\r\nserviceRequestId: 0b3f3aa1-10cd-9000-35f1-9f406a04d79d",
  "source": "https://contoso.sharepoint.com/teams/d-team/_api/web/GetFileByServerRelativeUrl('ControlledDocuments/Sample%20Document%2010.docx')/CheckIn(comment='COMMENTS%20FOR%20PUBLISH',checkintype=1)",
  "errors": [
    "-2147024809",
    "System.ArgumentException"
  ]
}

Ensure the correct full relative path is specified including the site relative url.

### Cannot deserialize data

 > {
  "status": 400,
  "message": "Cannot deserialize data for type Microsoft.SharePoint.SPResourcePath.\r\nclientRequestId: 76bf623f-bfca-4aad-a036-6dfd04bdc779\r\nserviceRequestId: 5a443aa1-a016-9000-5f10-b1a2e6f5b6de",
  "source": "https://contoso.sharepoint.com/teams/d-team/_api/web/GetFileByServerRelativePath('/teams/d-team/ControlledDocuments/Sample%20Document%2010.docx')/Publish('Major%20publish')",
  "errors": [
    "-1",
    "Microsoft.SharePoint.Client.InvalidClientQueryException"
  ]
}

Ensure the DecodedUrl is used within GetFileByServerRelativePath

## Conclusion

The automatic publish of files as major version in SharePoint using Power Automate for document management processes, especially post-approval can streamline the process.