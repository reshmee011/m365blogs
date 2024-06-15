---
title: "Retrieving File Count and Size of a folder using PnP PowerShell"
date: 2024-06-15T14:49:19+01:00
tags: ["PnP PowerShell", "Get File count and size","SharePoint", "Get-PnPFolderStorageMetric",folder"]
featured_image: '/posts/images/PowerShell_GetFolderStorageMetric/example.png'
draft: false
---

# Retrieving File Count and Size of a folder within a SharePoint Library using PnP PowerShell

This post covers a PowerShell script that uses the PnP PowerShell module to retrieve the file count and total size of a specific folder within a SharePoint library.

```PowerShell
#Parameters
$SiteURL = "https://contoso.sharepoint.com/sites/company311"
$FolderSiteRelativeURL = "Shared Documents/Test1"
#Connect to PnP Online
Connect-PnPOnline -Url $SiteURL -Interactive
#Get the folder
$Folder = Get-PnPFolder -Url $FolderSiteRelativeURL -Includes ListItemAllFields
#Get the total Size of the folder - with versions
Write-host "Size of the Folder:" $([Math]::Round(($Folder.ListItemAllFields.FieldValues.SMTotalSize.LookupId/1KB),2))
 
 
#Get the folder size from storage metrics
$FolderSize= Get-PnPFolderStorageMetric -FolderSiteRelativeUrl $FolderSiteRelativeURL| Select -ExpandProperty TotalSize
$FolderSize = [Math]::Round($FolderSize/1MB, 2)
$FolderItemCount = Get-PnPFolderStorageMetric -FolderSiteRelativeUrl $FolderSiteRelativeURL| Select -ExpandProperty TotalFileCount
Write-host "Total size of the Folder: $FolderSize MB"
Write-host "Total File count: $FolderItemCount"
```

This script connects to a SharePoint site, retrieves a specific folder within a library, and calculates the total size and file count of the folder using the **Get-PnPFolderStorageMetric** cmdlet. It then prints these metrics to the console.

This script can be useful for SharePoint administrators who need to monitor the size and file count of specific folders within their libraries. 

![Sample Output](../images/PowerShell_GetFolderStorageMetric/example.png)
