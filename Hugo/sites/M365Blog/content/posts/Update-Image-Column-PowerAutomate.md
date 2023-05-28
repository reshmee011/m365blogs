---
title: "Add or Update Image Column in SharePoint with Power Automate: Download and Use Images from the Web"
date: 2023-05-28T06:56:03+01:00
author: "Reshmee Auckloo"
githubname: reshmee011
categories: ["Post"]
images:
- images:Update-Image-Column-PowerAutomate/images/PowerAutomate.png
tags: ["Image, PowerAutomate"]
type: "regular"
draft: false
---


# Add or Update Image Column in SharePoint with Power Automate: Download and Use Images from the Web

The blog post titled [Update image in SharePoint/Microsoft Lists Image columns using Power Automate](https://ganeshsanapblogs.wordpress.com/2022/10/20/update-image-in-sharepoint-microsoft-lists-image-columns-using-power-automate/) by Ganesh Sanap rovides a guide on how to update an image column in SharePoint or Microsoft Lists using Power Automate. The native "Create Item" and "Update Item" actions in Power Automate do not directly support adding or updating image columns. Instead, Ganesh suggests using the "Send an HTTP Request to SharePoint" action to accomplish this.This blog post covers how to download an image from an image url and add/update the image column.

This blog post focuses on downloading an image from a URL and subsequently adding or updating the image column. 

## Create a list with the following columns

To begin, either use an existing list or create a new list with the following columns:

| Column | Type            |
| --------- | ----------------- |
| title     | Multiple lines of text  |
| mediaUrl    | Multiple lines of text    |
| media    | image    |

![SharePoint List](../images/Update-Image-Column-PowerAutomate/SharePointList.png)

Make sure to populate both the title and mediaUrl fields. In the provided example, Instagram image URLs are utilised.

## Ensure Assets library Creation with PnP PowerShell

To ensure the creation of the Assets library, execute the following PnP PowerShell script:

```powerShell
Connect-PnPOnline -Url https://contoso.sharepoint.com/sites/<siteName>/ -Interactive
$web= Get-PnPWeb
$web.Lists.EnsureSiteAssetsLibrary()
Invoke-PnPQuery
```

## Power Automate flow 

Navigate to [Power Automate](https://make.powerautomate.com) and create a new Instant cloud flow using **Manually trigger a flow** trigger.

1. Add the **Get Items** action to retrieve all items from the SharePoint List, specifying the **Site Address** and **List Name**.

2. To upload the image to the **Site Assets** library, use the **Create File** action to save the image obtained from previous step. Set the **Folder** to  **SiteAssets/lists/<listId>** and the **FileName** to the title of the list item suffixed with **.png**. 

![Create File](../images/Update-Image-Column-PowerAutomate/CreateFile.png)

To obtain the **listId**, navigate to the list settings and copy the list id from the URL.

![ListId](../images/Update-Image-Column-PowerAutomate/listId.png)

Disable **Allow Chunking** for the **Create File** action to prevent flow failure if the file already exists.
![Create File settings](../images/Update-Image-Column-PowerAutomate/CreateFileSettings.png)

3. To save the image to column, use the **Send an HTTP request to SharePoint** action and configure the properties as follows

Method : POST
Uri : _api/web/lists('<listId>')/items(@{items('Apply_to_each')?['ID']})/validateUpdateListItem()
Headers: 
    content-type : application/json;odata=verbose
    accept : application/json;odata=verbose

Body: 
```json
{"formValues":[{"FieldName":"media","FieldValue":"{\"type\":\"thumbnail\",\"fileName\":\"@{items('Apply_to_each')?['Title']}.png\",\"fieldName\":\"media\",\"serverUrl\":\"https://contoso.sharepoint.com\",\"serverRelativeUrl\":\"/sites/siteName@{outputs('Create_file')?['body/Path']}\"}","HasException":false,"ErrorMessage":null}],"bNewDocumentUpdate":false,"checkInComment":null}
```
![Save image to column](../images/Update-Image-Column-PowerAutomate/SaveImageToColumn.png)

The final flow looks like like this.

![Update Media Column using Power Automate](../images/Update-Image-Column-PowerAutomate/UpdateMediaColumnFlow.png)

Save and test the flow. After the flow completes, the image column of all items will be populated from the URL specified from mediaUrl field

![PopulatedMedia](../images/Update-Image-Column-PowerAutomate/PopulatedMedia.png)


