---
title: "Get all Unique Permissions PowerShell - Options"
date: 2023-08-02T08:42:13+01:00
tags: ["SharePoint", "PowerShell","Unique Permissions"]
omit_header_text: true
draft: true
---

## PnP PowerShell

 $ListItems = Get-PnPListItem -List $list -PageSize 2000 
```powershell
    
    $ListItems = Get-PnPListItem -List $list -PageSize 2000 
 #   getSharingLink $ctx $list "list/library" $siteUrl $listUrl;
        #Iterate through each list item
        ForEach($item in $ListItems)
        {
            #Check if the Item has unique permissions
            $HasUniquePermissions = Get-PnPProperty -ClientObject $Item -Property "HasUniqueRoleAssignments"
            If($HasUniquePermissions){
           write-host 
             }
        }
```

## REST API 

```md
https://reshmeeauckloo.sharepoint.com/sites/Company311/_api/web/lists/getbytitle('Documents')/items(5)?$Select=ID,HasUniqueRoleAssignments
```


![Graph does not have property HasUniqueRoleAssignments](../images/powershell_getallitemswithUniquePermissions/MSGraph_HasUniqueRoleAssignments_Does_NotExist.png)
