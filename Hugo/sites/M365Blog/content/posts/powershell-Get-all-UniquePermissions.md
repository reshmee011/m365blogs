---
title: "Optimizing PowerShell Scripts to Retrieve Unique Permissions in SharePoint: REST API vs. Get-PnPListItem"
date: 2024-08-03T08:42:13+01:00
tags: ["SharePoint", "PowerShell","Unique Permissions","PnP PowerShell","REST","HasUniqueRoleAssignments"]
featured_image: '/posts/images/powershell-Get-all-UniquePermissions/output.png'
omit_header_text: true
draft: false
---

When working with large SharePoint sites, retrieving unique permissions can be a time-consuming task. This blog post explores methods to optimize PowerShell scripts for fetching unique permissions, including using PnP PowerShell and the SharePoint REST API. We compare their performance and highlight the advantages and limitations of each approach.

## Using PnP PowerShell

PnP PowerShell provides an efficient way to interact with SharePoint Online and retrieve list items to check for unique permissions. Here's a script that demonstrates how to use PnP PowerShell to achieve this:

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
 
# Ensure the logs folder exists
$logsFolder = Split-Path -Path $ReportOutput -Parent
if (-not (Test-Path -Path $logsFolder)) {
    New-Item -Path $logsFolder -ItemType Directory
}

# Ensure the file exists
if (-not (Test-Path -Path $ReportOutput)) {
    New-Item -Path $ReportOutput -ItemType File
} 
 
#Exclude certain libraries
$ExcludedLists = @("Access Requests", "App Packages", "appdata", "appfiles","Apps for SharePoint" ,"Apps in Testing", "Cache Profiles", "Composed Looks", "Content and Structure Reports", "Content type publishing error log", "Converted Forms",
    "Device Channels", "Form Templates", "fpdatasources", "Get started with Apps for Office and SharePoint", "List Template Gallery", "Long Running Operation Status", "Maintenance Log Library", "Images", "site collection images"
    , "Master Docs", "Master Page Gallery", "MicroFeed", "NintexFormXml", "Quick Deploy Items", "Relationships List", "Reusable Content", "Reporting Metadata", "Reporting Templates", "Search Config List", "Site Assets", "Preservation Hold Library",
    "Site Pages", "Solution Gallery", "Style Library", "Suggested Content Browser Locations", "Theme Gallery", "TaxonomyHiddenList", "User Information List", "Web Part Gallery", "wfpub", "wfsvc", "Workflow History", "Workflow Tasks", "Pages")
 
Connect-PnPOnline -Url $siteUrl -Interactive
write-host $("Start time " + (Get-Date))
 
Write-Host "Processing site $siteUrl"  -Foregroundcolor "Red";
$siteID = (Get-PnPSite -Includes Id).Id
$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists }
  Write-Host "Number of lists $($ll.Count)";
 
  foreach($list in $ll)
  {
   
    $ListItems = Get-PnPListItem -List $list -PageSize 2000
 
       ForEach($item in $ListItems)
        {
             $HasUniquePermissions = Get-PnPProperty -ClientObject $Item -Property "HasUniqueRoleAssignments"
            #Check if the Item has unique permissions
           if($HasUniquePermissions){
 
             # add items to the report to export
             $item | Select-Object @{Name="SiteURL";Expression={$siteUrl}},@{Name="ListTitle";Expression={$list.Title}},@{Name="ListUrl";Expression={$listUrl}},@{Name="ItemID";Expression={$item.ID}},@{Name="ItemURL";Expression={$siteUrl + $item.FieldValues['FileRef']}},@{Name="ItemName";Expression={$item.FieldValues['FileLeafRef']}} | Export-Csv -Path $ReportOutput -Append -NoTypeInformation      
 
            }
        }
    }
 
  #Export-CSV $ReportOutput -NoTypeInformation
  write-host $("End time " + (Get-Date)) 
```

For a site with 1600 items, this script takes approximately 200 seconds. For larger sites with over 100,000 items, it takes over 2 hours, which is not optimal.

![output of PnP](../images/powershell-Get-all-UniquePermissions/Output_PnP.png)

### Using parameter Fields `HasUniqueRoleAssignments` with Get-PnPListItem

Specifying the Fields parameter to return values field `HasUniqueRoleAssignments` does not load the values of properties `HasUniqueRoleAssignments` without specifying the Id parameter.

 ```powerShell
 $ListItems = Get-PnPListItem -List $list -PageSize 2000 -Fields "HasUniqueRoleAssignments"
 ```

### Attempts to Optimize with CAML Query

Using CAML query to specify fields did not significantly improve performance and encountered list view threshold errors for lists with more than 5000 items.

```powershell
$ListItems = Get-PnPListItem -List $list -Query "<View><ViewFields><FieldRef Name='HasUniqueRoleAssignments'/><FieldRef Name='FileRef'/><FieldRef Name='FileSystemObjectType'/><FieldRef Name='FileLeafRef'/></ViewFields><Query></Query></View>"
```

### Using REST API

An alternative approach involves using the SharePoint REST API to query the `HasUniqueRoleAssignments` property. 

#### REST API Endpoints

The following endpoints can be used to retrieve the `HasUniqueRoleAssignments` property:

```md
/_api/web/HasUniqueRoleAssignments  
/_api/web/lists/getbytitle('list title')/HasUniqueRoleAssignments  
/_api/web/lists/getbytitle('list title')/items(id)/HasUniqueRoleAssignments  
```

The query from the browser returns the `HasUniqueRoleAssignments` value.

```md
https://reshmeeauckloo.sharepoint.com/sites/Company311/_api/web/lists/getbytitle('Documents')/items(5)?$Select=ID,HasUniqueRoleAssignments
```

![HasUniqueRoleAssignments value returned from REST](../images/powershell-Get-all-UniquePermissions/REST_API.png)

#### Example REST API Script
Here's a PowerShell script leveraging the REST API to retrieve unique permissions:

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
$fileName = "UniquePermissions_Rest-" + $dateTime + ".csv"
$ReportOutput = $directorypath + "\Logs\"+ $fileName

# Ensure the logs folder exists
$logsFolder = Split-Path -Path $ReportOutput -Parent
if (-not (Test-Path -Path $logsFolder)) {
    New-Item -Path $logsFolder -ItemType Directory
}

# Ensure the file exists
if (-not (Test-Path -Path $ReportOutput)) {
    New-Item -Path $ReportOutput -ItemType File
}

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

function Get-ListItems{
    param(
        [Parameter(Mandatory)]
        [Microsoft.SharePoint.Client.List]$List
    )
    
    $token = Get-PnPappauthaccesstoken
    $selectFields = "ID,HasUniqueRoleAssignments,FileRef,FileLeafRef,FileSystemObjectType"
    $headers = @{"Accept" = "application/json;odata=verbose"
    "Authorization" = "Bearer $token"}
   
    $Url = $siteUrl + '/_api/web/lists/getbytitle(''' + $($list.Title) + ''')/items?$select=' + $($selectFields)
    $nextLink = $Url
    $listItems = @()
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
    
        $listItems += $response.value | where-object{$_.HasUniqueRoleAssignments -eq $true}
        if($response.'odata.nextlink'){
            $nextLink = $response.'odata.nextlink'
        }    else{
            $nextLink = $null
        }
    }

    return $listItems
}

$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists } #$_.BaseType -eq "DocumentLibrary"
  Write-Host "Number of lists $($ll.Count)";
 
  foreach($list in $ll)
  {
    $listUrl = $list.RootFolder.ServerRelativeUrl;     
       $ListItems = Get-ListItems -List $list 
       ForEach($item in $ListItems)
        {
         # add items to the report to export
          $item | Select-Object @{Name="SiteURL";Expression={$siteUrl}},@{Name="ListTitle";Expression={$list.Title}},@{Name="ListUrl";Expression={$listUrl}},@{Name="ItemID";Expression={$item.ID}},@{Name="ItemURL";Expression={$siteUrl + $item.FileRef}},@{Name="ItemName";Expression={$item.FileLeafRef}} | Export-Csv -Path $ReportOutput -Append -NoTypeInformation
        }
    }
 write-host $("End time " + (Get-Date))
```

For a sites around 1600 items, the script took 8 secs to complete.

![Output REST](../images/powershell_getallitemswithUniquePermissions/Output_REST.png)

For a site with more than 100000 items, the retry logic helped to handle throttling or transient network errors taking less than 11 minutes to complete compared to the PnP PowerShell cmdlets taking over 2 hours. 

![REST Unique Permissions with retired](../images/powershell_getallitemswithUniquePermissions/RESTAPI_UniquePermissions.png)

![Graph does not have property HasUniqueRoleAssignments](../images/powershell_getallitemswithUniquePermissions/MSGraph_HasUniqueRoleAssignments_Does_NotExist.png)

## Microsoft Graph does not return the property HasUniqueRoleAssignments

Microsoft Graph does not return the property HasUniqueRoleAssignments

![Graph does not have property HasUniqueRoleAssignments](../images/powershell_getallitemswithUniquePermissions/MSGraph_HasUniqueRoleAssignments_Does_NotExist.png)


## Differences in timings

|**Endpoint Type**|**Number of items**|**Timing**|
|---|---|---|
|REST API|1600|8 secs|
|PnP PowerShell without any batch|1600|200 secs|
|REST API|100,000|11 mins|
|PnP PowerShell without any batch|100,000|2 hours|
|Microsoft Graph|`N/A`|`N/A`|

## References

[Invoke-PnPSPRestMethod](https://pnp.github.io/powershell/cmdlets/Invoke-PnPSPRestMethod.html)

[SharePoint REST API to retrieve ACLs of a SharePoint object](https://learn.microsoft.com/en-us/answers/questions/208656/sharepoint-rest-api-to-retrieve-acls-of-a-sharepoi)

[Get list of SharePoint files with unique permissions](https://community.powerplatform.com/forums/thread/details/?threadid=ee28b49a-4c5e-4882-b370-32ff84795ffd)