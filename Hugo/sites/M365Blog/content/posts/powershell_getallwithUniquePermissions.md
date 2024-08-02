---
title: "Get all Unique Permissions PowerShell - Options"
date: 2023-08-02T08:42:13+01:00
tags: ["SharePoint", "PowerShell","Unique Permissions"]
omit_header_text: true
draft: true
---

## PnP PowerShell

### Fields Parameter `HasUniqueRoleAssignments`
Specifying the Fields parameter to field `HasUniqueRoleAssignments` does not load the values of properties `HasUniqueRoleAssignments`


 ```powerShell
 $ListItems = Get-PnPListItem -List $list -PageSize 2000 -Fields "HasUniqueRoleAssignments"
 ```

### CAML Query

Using the CAML query to return 'SharedWithUsers', 'SharedUserDetails' or 'HasUniqueRoleAssignments' did not work

```powershell

```

```powershell
    

    $ListItems = Get-PnPListItem -List $list -PageSize 2000 -Fields "HasUniqueRoleAssignments"
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

param(
    [Parameter(Mandatory)]
    [string]$SiteUrl
)
 
        #Exclude certain libraries
$ExcludedLists = @("Access Requests", "App Packages", "appdata", "appfiles","Apps for SharePoint" ,"Apps in Testing", "Cache Profiles", "Composed Looks", "Content and Structure Reports", "Content type publishing error log", "Converted Forms",
    "Device Channels", "Form Templates", "fpdatasources", "Get started with Apps for Office and SharePoint", "List Template Gallery", "Long Running Operation Status", "Maintenance Log Library", "Images", "site collection images"
    , "Master Docs", "Master Page Gallery", "MicroFeed", "NintexFormXml", "Quick Deploy Items", "Relationships List", "Reusable Content", "Reporting Metadata", "Reporting Templates", "Search Config List", "Site Assets", "Preservation Hold Library",
    "Site Pages", "Solution Gallery", "Style Library", "Suggested Content Browser Locations", "Theme Gallery", "TaxonomyHiddenList", "User Information List", "Web Part Gallery", "wfpub", "wfsvc", "Workflow History", "Workflow Tasks", "Pages")
 
#$m365Sites = Get-PnPTenantSite| Where-Object { ( $_.Url -like '*/sites/*') -and $_.Template -ne 'RedirectSite#0' }
#$m365Sites | ForEach-Object {
#$siteUrl = $_.Url;    
Connect-PnPOnline -Url $siteUrl -Interactive
 
Write-Host "Processing site $siteUrl"  -Foregroundcolor "Red";
$siteID = (Get-PnPSite -Includes Id).Id
#getSharingLink $ctx $web "site" $siteUrl "";
$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists } #$_.BaseType -eq "DocumentLibrary"
  Write-Host "Number of lists $($ll.Count)";
 
  foreach($list in $ll)
  {
    $listUrl = $list.RootFolder.ServerRelativeUrl;    
    $listID = (Get-PnPList $list.Title).Id
    #Get all list items in batches
    #foreach-object{Get-PnPListItem -Id $_.Id-List $list -Fields HasUniqueRoleAssignments} |
    #$ListItems = Get-PnPListItem -List $list -Query "<View><ViewFields><FieldRef Name='HasUniqueRoleAssignments'/><FieldRef Name='FileRef'/><FieldRef Name='FileSystemObjectType'/><FieldRef Name='FileLeafRef'/></ViewFields><Query></Query></View>"
# -Fields HasUniqueRoleAssignments
    $token = Get-PnPappauthaccesstoken
    $selectFields = "ID,HasUniqueRoleAssignments,FileRef,FileLeafRef,FileSystemObjectType"
    $headers = @{"Accept" = "application/json;odata=verbose"
    "Authorization" = "Bearer $token"}
   
$Url = $siteUrl + '_api/web/lists/getbytitle(''' + $($list.Title) + ''')/items?$select=' + $($selectFields)
$nextLink = $Url
$ListItems = @()
while($nextLink){  
    $response = invoke-pnpsprestmethod -Url $nextLink -Method Get
   
    $ListItems += $response.value | where-object{$_.HasUniqueRoleAssignments -eq $true}
    if($response.'odata.nextlink'){
        $nextLink = $response.'odata.nextlink' -replace "$siteUrl/_api/",""
    }    else{
        $nextLink = $null
    }
 
}
        ForEach($item in $ListItems)
        {
            #Check if the Item has unique permissions
           #if($item.HasUniqueRoleAssignments){
                #Get Shared Links
                if($list.BaseType -eq "DocumentLibrary")
                {
                    $type= $item.FileSystemObjectType;
                }
               
                getSharingLink $item $type $siteUrl $listUrl;
            #}
        }
    }
 
```

## REST API 

```md
/_api/web/HasUniqueRoleAssignments  
/_api/web/lists/getbytitle('list title')/HasUniqueRoleAssignments  
/_api/web/lists/getbytitle('list title')/items(id)/HasUniqueRoleAssignments  
```

```md
https://reshmeeauckloo.sharepoint.com/sites/Company311/_api/web/lists/getbytitle('Documents')/items(5)?$Select=ID,HasUniqueRoleAssignments
```


![Graph does not have property HasUniqueRoleAssignments](../images/powershell_getallitemswithUniquePermissions/MSGraph_HasUniqueRoleAssignments_Does_NotExist.png)

## References

[Invoke-PnPSPRestMethod](https://pnp.github.io/powershell/cmdlets/Invoke-PnPSPRestMethod.html)

[SharePoint REST API to retrieve ACLs of a SharePoint object](https://learn.microsoft.com/en-us/answers/questions/208656/sharepoint-rest-api-to-retrieve-acls-of-a-sharepoi)