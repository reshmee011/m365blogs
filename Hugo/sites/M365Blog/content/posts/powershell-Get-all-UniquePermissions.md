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
param(
    [Parameter(Mandatory)]
    [string]$SiteUrl
)
 
if(!$siteUrl)
{
    $siteUrl = Read-Host -Prompt "Enter the site collection";
}
 
 
$dateTime = (Get-Date).toString("dd-MM-yyyy-hh-ss")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "UniquePermissions-" + $dateTime + ".csv"
$ReportOutput = $directorypath + "\Logs\"+ $fileName
 
 
 
#Exclude certain libraries
$ExcludedLists = @("Access Requests", "App Packages", "appdata", "appfiles","Apps for SharePoint" ,"Apps in Testing", "Cache Profiles", "Composed Looks", "Content and Structure Reports", "Content type publishing error log", "Converted Forms",
    "Device Channels", "Form Templates", "fpdatasources", "Get started with Apps for Office and SharePoint", "List Template Gallery", "Long Running Operation Status", "Maintenance Log Library", "Images", "site collection images"
    , "Master Docs", "Master Page Gallery", "MicroFeed", "NintexFormXml", "Quick Deploy Items", "Relationships List", "Reusable Content", "Reporting Metadata", "Reporting Templates", "Search Config List", "Site Assets", "Preservation Hold Library",
    "Site Pages", "Solution Gallery", "Style Library", "Suggested Content Browser Locations", "Theme Gallery", "TaxonomyHiddenList", "User Information List", "Web Part Gallery", "wfpub", "wfsvc", "Workflow History", "Workflow Tasks", "Pages")
 
#$m365Sites = Get-PnPTenantSite| Where-Object { ( $_.Url -like '*/sites/*') -and $_.Template -ne 'RedirectSite#0' }
#$m365Sites | ForEach-Object {
#$siteUrl = $_.Url;    
Connect-PnPOnline -Url $siteUrl -Interactive
write-host $("Start time " + (Get-Date))
 
Write-Host "Processing site $siteUrl"  -Foregroundcolor "Red";
$siteID = (Get-PnPSite -Includes Id).Id
#getSharingLink $ctx $web "site" $siteUrl "";
$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists } #$_.BaseType -eq "DocumentLibrary"
  Write-Host "Number of lists $($ll.Count)";
 
  foreach($list in $ll)
  {
   
    $ListItems = Get-PnPListItem -List $list -PageSize 2000
 
       ForEach($item in $ListItems)
        {
             $HasUniquePermissions = Get-PnPProperty -ClientObject $Item -Property "HasUniqueRoleAssignments"
            #Check if the Item has unique permissions
           if($HasUniquePermissions){
 
                   
 
            }
        }
    }
 
  #Export-CSV $ReportOutput -NoTypeInformation
  write-host $("End time " + (Get-Date)) 
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

```PowerShell
param(
    [Parameter(Mandatory)]
    [string]$SiteUrl
)
 
if(!$siteUrl)
{
    $siteUrl = Read-Host -Prompt "Enter the site collection";
}
 
 
#Parameters
$dateTime = (Get-Date).toString("dd-MM-yyyy-hh-ss")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "SharedLinksDeletion-" + $dateTime + ".csv"
$ReportOutput = $directorypath + "\Logs\"+ $fileName
 
#Exclude certain libraries
$ExcludedLists = @("Access Requests", "App Packages", "appdata", "appfiles","Apps for SharePoint" ,"Apps in Testing", "Cache Profiles", "Composed Looks", "Content and Structure Reports", "Content type publishing error log", "Converted Forms",
    "Device Channels", "Form Templates", "fpdatasources", "Get started with Apps for Office and SharePoint", "List Template Gallery", "Long Running Operation Status", "Maintenance Log Library", "Images", "site collection images"
    , "Master Docs", "Master Page Gallery", "MicroFeed", "NintexFormXml", "Quick Deploy Items", "Relationships List", "Reusable Content", "Reporting Metadata", "Reporting Templates", "Search Config List", "Site Assets", "Preservation Hold Library",
    "Site Pages", "Solution Gallery", "Style Library", "Suggested Content Browser Locations", "Theme Gallery", "TaxonomyHiddenList", "User Information List", "Web Part Gallery", "wfpub", "wfsvc", "Workflow History", "Workflow Tasks", "Pages")
 
#$m365Sites = Get-PnPTenantSite| Where-Object { ( $_.Url -like '*/sites/*') -and $_.Template -ne 'RedirectSite#0' }
#$m365Sites | ForEach-Object {
#$siteUrl = $_.Url;    
Connect-PnPOnline -Url $siteUrl -Interactive
write-host $("Start time " + (Get-Date))
Write-Host "Processing site $siteUrl"  -Foregroundcolor "Red";
 
$siteID = (Get-PnPSite -Includes Id).Id
#getSharingLink $ctx $web "site" $siteUrl "";
$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists } #$_.BaseType -eq "DocumentLibrary"
  Write-Host "Number of lists $($ll.Count)";
 
  foreach($list in $ll)
  {
    $listUrl = $list.RootFolder.ServerRelativeUrl;    
    $listID = (Get-PnPList $list.Title).Id
   
    $token = Get-PnPappauthaccesstoken
    $selectFields = "ID,HasUniqueRoleAssignments,FileRef,FileLeafRef,FileSystemObjectType"
    $headers = @{"Accept" = "application/json;odata=verbose"
    "Authorization" = "Bearer $token"}
   
$Url = $siteUrl + '/_api/web/lists/getbytitle(''' + $($list.Title) + ''')/items?$select=' + $($selectFields)
$nextLink = $Url
$ListItems = @()
$Stoploop =$true
while($nextLink){  
    do{
    try {
        $response = invoke-pnpsprestmethod -Url $nextLink -Method Get
        $Stoploop =$true
 
    }
    catch {
        write-host "An error occured: $_  : Retrying" -ForegroundColor Red
        $Stoploop =$true
        Start-Sleep -Seconds 30
    }
 }
 While ($Stoploop -eq $false)
 
 
   
    $ListItems += $response.value | where-object{$_.HasUniqueRoleAssignments -eq $true}
    if($response.'odata.nextlink'){
        $nextLink = $response.'odata.nextlink'
    }    else{
        $nextLink = $null
    }
 
}
        ForEach($item in $ListItems)
        {
           
        }
    }
 
write-host $("End time " + (Get-Date))
``

![Graph does not have property HasUniqueRoleAssignments](../images/powershell_getallitemswithUniquePermissions/MSGraph_HasUniqueRoleAssignments_Does_NotExist.png)

## References

[Invoke-PnPSPRestMethod](https://pnp.github.io/powershell/cmdlets/Invoke-PnPSPRestMethod.html)

[SharePoint REST API to retrieve ACLs of a SharePoint object](https://learn.microsoft.com/en-us/answers/questions/208656/sharepoint-rest-api-to-retrieve-acls-of-a-sharepoi)

[Get list of SharePoint files with unique permissions](https://community.powerplatform.com/forums/thread/details/?threadid=ee28b49a-4c5e-4882-b370-32ff84795ffd)