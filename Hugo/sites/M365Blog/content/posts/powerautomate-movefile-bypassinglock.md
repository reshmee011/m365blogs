---
title: "Power Automate : Move File bypassing locked issue"
date: 2024-07-31T09:53:05+01:00
tags: ["Power Automate","Update Author","Update Editor","REST API","SharePoint"]
featured_image: '/posts/images/powerautomate-movefile-bypassinglock/UpdateAuthorDetails.png'
omit_header_text: true
draft: true
---

The  `SharePoint - Move file` action can be used , however the file can't be moved if the file was accessed by the current Power Automate flow for any processing throwing the locked file error.

[../images/powerautomate-movefile-bypassinglock/movefile_networkerror.png]

> {
  "status": 400,
  "message": "File 'Shared Documents/Staff Attendances/To Be Processed/Staff Attendance -16072024.xlsx' cannot be moved because it is in locked mode.\r\nclientRequestId: d6df7566-881f-4f14-8548-c5fac1eda46d\r\nserviceRequestId: 606841a1-40f0-9000-9c69-507df9b21720"
}

There are two options to use if `SharePoint - Move file` is used
- Handle locked file in a loop to attempt moving file until it is not locked

[../images/powerautomate-movefile-bypassinglock/loop_MoveFileUntilUnlocked.png]

- Add `Delay` action for at least 5-6 minutes to wait until the Power Automate release the lock to do any action with the file including move


However speed is key and I have noticed the amount of time Power Automate releases the lock is unpredictable sometimes taking longer than 5-6 minutes which may cause the flow to run longer.

I have used the endpoint `_api/SP.MoveCopyUtil.MoveFileByPath(overwrite=@a1)?@a1=true` in vain as described [Moving Files with SharePoint Online REST APIs in Microsoft Flow]https://gist.github.com/zplume/9f4c1a658517802701deff3473f23a60.

>{
  "status": 400,
  "message": "{\"odata.error\":{\"code\":\"-1, Microsoft.Data.OData.ODataException\",\"message\":{\"lang\":\"en-US\",\"value\":\"The property '__metadata' does not exist on type 'SP.ResourcePath'. Make sure to only use property names that are defined by the type.\"}}}\r\nclientRequestId: 652f3666-c218-4e12-8091-ea6398864295\r\nserviceRequestId: d9a541a1-707f-9000-9682-4373dfbba747",
  "source": "https://ppfonline.sharepoint.com/teams/d-team-playground/_api/SP.MoveCopyUtil.MoveFileByPath(overwrite=@a1)?@a1=true",
  "errors": []
}

[../images/powerautomate-movefile-bypassinglock/movefilebypath_badrequest.png]

The above posts guided me to dig out the API being called from the network tab from developer tools in the browser to confirm the API called.


## Inspecting SharePoint API traffic

If you're curious about how SharePoint Online's modern UI uses SharePoint REST APIs, you can inspect the Network traffic sent from the SharePoint page using either the Network tab of your browser developer tools, or a debugging proxy like Telerik's excellent (and free) [Fiddler](https://www.telerik.com/fiddler).

This can be a useful technique for discovering as-yet undocumented APIs or how to use APIs that are documented but not explained clearly enough with examples.

In this post I'll focus on using the Network tab of Chrome's developer tools, but the same principals should apply to other browser development tools.

To only show API requests, you'll want to filter traffic shown in the Network tab by clicking on the XHR heading.
You can additionally filter by a text value in the 'Filter' text box.

## SP.MoveCopyUtil in the modern UI

If you navigate to a SharePoint Online document library that's configured to show the modern UI and (while inspecting browser network traffic) use the 'Move To' button to move a file to a different folder in the same library, you'll see the following REST request:

```
https://tenant.sharepoint.com/_api/SP.MoveCopyUtil.MoveFileByPath()
```

This endpoint accepts source and destination URLs in the request **body** rather than the request **URL**, which means it doesn't have the drawbacks of the `getFileByServerRelativeUrl('/from/')/moveTo(newUrl='/to/')` approach. 🎉

If you try and move a file to a folder that already contains a file with that name, the API request will return an error (HTTP 400) and you'll see the prompt `A file with this name already exists` with an option to `replace the existing one`. If you click **Replace**, the request is sent again with an additional parameter:

```
https://tenant.sharepoint.com/_api/SP.MoveCopyUtil.MoveFileByPath(overwrite=@a1)?@a1=true
```

If you inspect the request body (in Chrome this is via Headers > Request payload), you'll see something like the following JSON:

```json
{
    "srcPath": {
        "__metadata": {
            "type": "SP.ResourcePath"
        },
        "DecodedUrl": "https://tenant.sharepoint.com/Shared Documents/FileName.docx"
    },
    "destPath": {
        "__metadata": {
            "type": "SP.ResourcePath"
        },
        "DecodedUrl": "https://tenant.sharepoint.com/Shared Documents/FolderName/FileName.docx"
    }
}
```

As you can see, the **DecodedUrl** values are absolute URLs that include the source and destination file names.
The other option is to use [https://www.cleverworkarounds.com/2021/02/21/how-to-clear-annoying-excel-file-locks-in-power-automate/comment-page-2/](https://www.cleverworkarounds.com/2021/02/21/how-to-clear-annoying-excel-file-locks-in-power-automate/comment-page-2/)

This post outlines how to use Power Automate to move a file within SharePoint using the `Send an Http request to SharePoint action`.

Within a Power Automate flow, follow the following steps

1. Add `Send an Http request to SharePoint` renamed to `Move file`

### Properties

|Property|Value|
|---|---|
|Site Address|_The SharePoint site URL_|
|Method|POST|
|Uri|`_api/site/CreateCopyJobs`|
|Headers|_See **Headers** table below_|
|Body|_See **Body** below_|

### Headers
|Name|Value|
|---|---|
|Accept|`application/json; odata=nometadata`|
|Content-Type|`application/json; odata=verbose`|

### Body :

```json
{"exportObjectUris":["https://contoso.sharepoint.com/teams/test/Shared%20Documents/To%20Be%20Processed/test.xlsx"],"destinationUri":"https://contoso.sharepoint.com/teams/test/Shared%20Documents/Completed","options":{"IgnoreVersionHistory":true,"AllowSchemaMismatch":true,"BypassSharedLock":true,"IsMoveMode":true,"IncludeItemPermissions":true,"SameWebCopyMoveOptimization":true}}
```


![Move Action](../images/powerautomate-movefile-bypassinglock/rest_api_createcopyjobs.png.png)

By following these steps, you can update the author and editor fields in a SharePoint list item using Power Automate. This is particularly useful for automating the management of SharePoint content and ensuring that the correct user details are recorded.