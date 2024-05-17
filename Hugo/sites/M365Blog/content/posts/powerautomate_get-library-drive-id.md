---
title: "Powerautomate_get Library Drive Id"
date: 2024-05-17T16:54:24+01:00
draft: true
---

Error with malformed drive id if using document as data source

The provided drive id appears to be malformed, or does not represent a valid drive.
clientRequestId: 6842e526-7afc-46a2-84f5-e40794457816
serviceRequestId: 6ffaacd1-6b2d-41d7-b3a6-963ab3e2dcbd

### Handling Locked files

https://sharepains.com/2024/01/05/system-update-sharepoint-power-automate/#Locked_files

#### Send an HTTP request to SharePoint get driveid to ValidateUpdateListItem
/_api/web/Lists/GetbyTitle('Data Import')/items(triggerOutputs()?['body']['ID'])/ValidateUpdateListItem()

{
"formValues":[{"FieldName": "Status", "FieldValue": "In Progress"}],
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
()[https://sharepains.com/2024/01/05/system-update-sharepoint-power-automate/#Locked_files]
