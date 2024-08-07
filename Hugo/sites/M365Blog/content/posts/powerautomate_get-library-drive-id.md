---
title: "Powerautomate_get Library Drive Id"
date: 2024-08-07T16:54:24+01:00
tags: ["Power Automate","DriveId","REST","Solution","Excel for Business"]
featured_image: '/posts/images/powerautomate_get-library-drive-id/div_email_owa.PNG'
omit_header_text: true
draft: true
---


If a library is referenced within a Excel for Business action with a Power Automate, it automatically figures out the driveid internally. However if ever a solution is used for a Power Automate to be deployed across different environments and a Library as a datasource environment variable is created. Referencing the variable in the Power Automate flow, it will throw an error message failing to work our the drive id of the library.

![Data source library](../images/powerautomate_get-library-drive-id/DataSourceLibrary.png)

Error with malformed drive id if using document as data source

![Error with data source library](../images/powerautomate_get-library-drive-id/errorwithlibrarydatasource.png)

```dotnetcli
The provided drive id appears to be malformed, or does not represent a valid drive.
clientRequestId: 6842e526-7afc-46a2-84f5-e40794457816
serviceRequestId: 6ffaacd1-6b2d-41d7-b3a6-963ab3e2dcbd
```
## HTTP with Microsoft Entra ID to get Drive Id

If the action **HTTP with Microsoft Entra ID** is blocked in your environment you might get the following error message

![HTTP with Microsoft Entra ID](../images/powerautomate_get-library-drive-id/HTTPWithMIcrosoftEntraID.png)

Error
![Invoke Entra ID error](../images/powerautomate_get-library-drive-id/InvokeEntraId_error.png)

It leaves the option to use the REST endpoint 

## Send HTTP Request to SharePoint to get Drive Id  

filter does not work
_api/v2.0/drives?filter=name%20eq%20%27data%20import%27

![REST API](../images/powerautomate_get-library-drive-id/filter_does_not_work.png) 

Use only select parameter only
_api/v2.0/drives?$select=name,id


![REST API](../images/powerautomate_get-library-drive-id/Select_RESTAPI.png)

![Send HTTP Request to SharePoint to get Drive Id ](../images/powerautomate_get-library-drive-id/SendHTTPRequestToSharePointToGetDriveId.png)

### Properties

|Property|Value|
|---|---|
|Site Address|_The SharePoint site URL_|
|Method|GET|
|Uri|`_api/v2.0/drives?$select=name,id`|
|Headers|_See **Headers** table below_|
|Body|_See **Body** below_|

### Headers

|Name|Value|
|---|---|
|Accept|`application/json; odata=nometadata`|
|Content-Type|`application/json; odata=verbose`|


The REST API does not allow using the $filter parameter unfortunately. After all drives are returned from SharePoint, the filter action is used to query  the drive if of the library. 

## filter array to get drive id
![filter array action](../images/powerautomate_get-library-drive-id/filterarrayaction_name.png)


equals(item()['name'],'Data Import')

## Compose - get driveId

outputs('Filter_array_to_get_drive_id')?['body']?[0]?['id']

![Compose action](../images/powerautomate_get-library-drive-id/Compose_GetDriveid.png)


## List rows present in a table

![List rows present in a table](../images/powerautomate_get-library-drive-id/ListRowspresentinatable.png)

If the file being created is triggering the flow, the file can be referenced by `triggerBody()?['{Identifier}']`

|Property|Value|
|---|---|
|Location|_The SharePoint site URL_|
|Document Library|outputs('Compose_-_get_driveId')|
|File|triggerBody()?['{Identifier}']|
|Table|<tablename>|

The drive Id is populated
![Drive Id](../images/powerautomate_get-library-drive-id/libraryconnection.png)

## Reference
[Get drive id for Doc Library selection](https://powerusers.microsoft.com/t5/Building-Flows/Get-drive-id-for-Doc-Library-selection/m-p/857384#M120179)
[System Update trick to avoid locked file issue](https://sharepains.com/2024/01/05/system-update-sharepoint-power-automate/#Locked_files)
