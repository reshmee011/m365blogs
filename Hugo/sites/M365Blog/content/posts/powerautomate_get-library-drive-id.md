---
title: "Powerautomate_get Library Drive Id"
date: 2024-05-17T16:54:24+01:00
tags: ["Power Automate","DriveId","REST","Solution","Excel for Business"]
featured_image: '/posts/images/powerautomate_get-library-drive-id/div_email_owa.PNG'
draft: true
---

# How to get DriveId of library for Excel for Business

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


## Send HTTP Request to SharePoint to get Drive Id  

![REST API](../images/powerautomate_get-library-drive-id/Select_RESTAPI.png)

![Send HTTP Request to SharePoint to get Drive Id ](../images/powerautomate_get-library-drive-id/SendHTTPRequestToSharePointToGetDriveId.png)

The REST API does not allow using the $filter parameter

![filter array action](../images/powerautomate_get-library-drive-id/filterarrayaction_name.png)
![Libray connection](../images/powerautomate_get-library-drive-id/libraryconnection.png)

### Handling Locked files

https://sharepains.com/2024/01/05/system-update-sharepoint-power-automate/#Locked_files

#### Send an HTTP request to SharePoint get driveid to ValidateUpdateListItem
/_api/web/Lists/GetbyTitle('Data Import')/items(triggerOutputs()?['body']['ID'])/ValidateUpdateListItem()

{
"formValues":[{"FieldName": "Status", "FieldValue": "In Progress"},
{"FieldName": "Editor",
 "FieldValue":"[{'Key':'triggerOutputs()?['body/Editor/Claims']'}]"}],
 "bNewDocumentUpdate":true
}

### Send an HTTP request to SharePoint get driveid

filter does not work
_api/v2.0/drives?filter=name%20eq%20%27data%20import%27

Use only select parameter only
_api/v2.0/drives?$select=name,id

Headers
Content-Type application/json
Accept  application/json

### filter array to get drive id

equals(item()['name'],'Data Import')

### Compose - get driveId

outputs('Filter_array_to_get_drive_id')?['body']?[0]?['id']

### List rows present in a table
outputs('Compose_-_get_driveId')


## Reference
[Get drive id for Doc Library selection](https://powerusers.microsoft.com/t5/Building-Flows/Get-drive-id-for-Doc-Library-selection/m-p/857384#M120179)
[System Update trick to avoid locked file issue](https://sharepains.com/2024/01/05/system-update-sharepoint-power-automate/#Locked_files)
[Hyperlink Column JSON](https://tomriha.com/how-to-update-sharepoint-hyperlink-column-in-power-automate/)
[Storing a link into Sharepoint List](https://powerusers.microsoft.com/t5/Building-Flows/Storing-a-link-into-Sharepoint-List-Hyperlink-Column/m-p/1636264#M181989) 