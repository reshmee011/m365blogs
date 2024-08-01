---
title: "Power Automate : Move File bypassing locked issue"
date: 2024-07-31T09:53:05+01:00
tags: ["Power Automate","Update Author","Update Editor","REST API","SharePoint","BypassSharedLock"]
featured_image: '/posts/images/powerautomate-movefile-bypassinglock/rest_api_createcopyjobs.png'
omit_header_text: true
draft: false
---

The  `SharePoint - Move file` action can be used to move files, however the file can't be moved if the file was accessed by the current Power Automate flow for any processing reslutng in a locked file error.

! [../images/powerautomate-movefile-bypassinglock/movefile_networkerror.png]

> {
  "status": 400,
  "message": "File 'Shared Documents/Attendances/To Be Processed/Attendance -16072024.xlsx' cannot be moved because it is in locked mode.\r\nclientRequestId: d6df7566-881f-4f14-8548-c5fac1eda46d\r\nserviceRequestId: 606841a1-40f0-9000-9c69-507df9b21720"
}

## Handling Locked Files

There are two options to handlelocked files when using `SharePoint - Move file` action:

1. **Handle Locked file in a Loop**: Attempt to move the file in a loop until it is no longer locked.

[../images/powerautomate-movefile-bypassinglock/loop_MoveFileUntilUnlocked.png]

2. **Add Delay action**: Add a `Delay` action for at least 5-6 minutes to wait until the Power Automate automatically releases the lock.

However, the time Power Automate takes to release the lock is unpredictable and can sometimes exceed 5-6 minutes, causing the flow to run longer.

## Using REST API to Move Files

I attempted to use the endpoint `_api/SP.MoveCopyUtil.MoveFileByPath(overwrite=@a1)?@a1=true` as described in [Moving Files with SharePoint Online REST APIs in Microsoft Flow](https://gist.github.com/zplume/9f4c1a658517802701deff3473f23a60), but encountered issues..

>{
  "status": 400,
  "message": "{\"odata.error\":{\"code\":\"-1, Microsoft.Data.OData.ODataException\",\"message\":{\"lang\":\"en-US\",\"value\":\"The property '__metadata' does not exist on type 'SP.ResourcePath'. Make sure to only use property names that are defined by the type.\"}}}\r\nclientRequestId: 652f3666-c218-4e12-8091-ea6398864295\r\nserviceRequestId: d9a541a1-707f-9000-9682-4373dfbba747",
  "source": "https://ppfonline.sharepoint.com/teams/d-team-playground/_api/SP.MoveCopyUtil.MoveFileByPath(overwrite=@a1)?@a1=true",
  "errors": []
}

![../images/powerautomate-movefile-bypassinglock/movefilebypath_badrequest.png]

The above posts guided me to dig out the API being called from the network tab from developer tools in the browser to confirm the API called.

### Inspecting SharePoint API traffic

To understand how SharePoint Online's modern UI uses SharePoint REST APIs, you can inspect the network traffic using the Network tab of your browser developer tools or a debugging proxy like [Fiddler](https://www.telerik.com/fiddler). This is helpful to discover what endpoints are called during operations.

### CreateCopyJobs to copy or move file

When moving a file using the 'Move To' button in a SharePoint Online document library, , the following REST request is made:

```md
https://tenant.sharepoint.com/_api/site/CreateCopyJobs
```

## Using Power Automate to Move Files
 
To move a file within SharePoint using the `Send an Http request to SharePoint` action in Power Automate, follow these steps:


1. Add `Send an Http request to SharePoint` and rename it to `Move file`

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

### Body

```json
{
    "exportObjectUris":["https://contoso.sharepoint.com/teams/test/Shared%20Documents/To%20Be%20Processed/test.xlsx"],
    "destinationUri":"https://contoso.sharepoint.com/teams/test/Shared%20Documents/Completed",
    "options":{
        "IgnoreVersionHistory":true,
        "AllowSchemaMismatch":true,
        "BypassSharedLock":true,
        "IsMoveMode":true,
        "IncludeItemPermissions":true,
        "SameWebCopyMoveOptimization":true
      }
}
```

**Key Parameters**

There are several parameters in the body of the request to control the behavior of the move operation. Here are the key parameters:

* BypassSharedLock

True:  Bypass any shared locks on the file.
False: Do not bypass any shared locks on the file.

* IncludeItemPermissions

True:  Ensures that item-level permissions are included when moving or copying the file.
False: Item-level permissions are not included when moving or copying the file.

* SameWebCopyMoveOptimization

True:  Can improve performance and reduce the time required for the operation when moving or copying files within the same site.
False: Does not have any improvement during move of the file.

* IgnoreVersionHistory:

True: Ignores the version history of the file.
False: Retains the version history of the file.

* IsMoveMode:

True: Moves the file.
False: Copies the file.

* AllowSchemaMismatch:

True: Allows the move even if the content types of the source and destination libraries do not match.
False: Prevents the move if there is a schema mismatch.

* NameConflictBehavior:

1 (Replace): Overwrites the existing file in the destination.
2 (KeepBoth): Keeps both files by renaming the new file.

![Move Action](../images/powerautomate-movefile-bypassinglock/rest_api_createcopyjobs.png)

By following these steps, files can be moved within SharePoint using Power Automatee, bypassing locked file issues.