---
title: "Counting Files within SharePoint Libraries using PnP PowerShell"
date: 2024-06-15T14:49:19+01:00
tags: ["PnP PowerShell", "Get File count","SharePoint", "Library"]
featured_image: '/posts/images/PowerShell_GetFileCountWithinLibrary/example.png'
draft: false
---

# Counting Files within SharePoint Libraries using PnP PowerShell

This post shows a PowerShell script using PnP PowerShell module to count the number of files and folders within a SharePoint library. It might be useful doing an inventory of file count. One useful application was to check whether the number of items ingested from the SharePoint site in a third party application like RecordPoint tally.

```PowerShell
$SiteURL = "Counting Files within a SharePoint Library using PnP PowerShell"
 
# Generate a unique log file name using today's date
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "FileCountReport-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
$NumberOfFiles = 0
#Connect to SharePoint Online
Connect-PnPOnline $SiteURL -Interactive
 
$SystemLists = @("Converted Forms", "Master Page Gallery", "Customized Reports", "Form Templates", "List Template Gallery", "Theme Gallery","Apps for SharePoint","Reporting Templates", "Solution Gallery", "Style Library", "Web Part Gallery","Site Assets", "wfpub", "Site Pages", "Images", "MicroFeed","Pages")
$FolderStats = @()
#Get the list
Get-PnPList -Includes RootFolder | Where {$_.Hidden -eq $false -and $SystemLists -notcontains $_.Title -and $_.BaseTemplate -eq 101 } | ForEach-Object {
#Get Folders from the Library - with progress bar
$List = $_
$global:counter = 0
$FolderItems = Get-PnPListItem -List $List -PageSize 500 -Fields FileLeafRef -ScriptBlock { Param($items) $global:counter += $items.Count; Write-Progress -PercentComplete `
            ($global:Counter / ($List.ItemCount) * 100) -Activity "Getting folders from List:" -Status "Processing Items $global:Counter to $($List.ItemCount)";}  | Where {$_.FileSystemObjectType -eq "Folder"}
Write-Progress -Activity "Completed Retrieving Folders from List $ListName" -Completed
$NumberOfFiles = $List.ItemCount - $FolderItems.Count
 
 
 $Data = [PSCustomObject][ordered]@{
    ListName =     $list.Title
    ListUrl =     $list.RootFolder.ServerRelativeUrl
    FilesCount     = $NumberOfFiles
    FolderCount = $FolderItems.Count
    ListCount =    $List.ItemCount
}
 $FolderStats+= $Data
}
#Export the data to CSV
$FolderStats | Export-Csv -Path $OutPutView -NoTypeInformation
```

This script connects to a SharePoint site, retrieves all non-hidden libraries, and counts the number of files and folders in each library. It then exports this data to a CSV file.

![Sample Output](../images/PowerShell_GetFileCountWithinLibrary/example.png)
