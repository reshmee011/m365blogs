---
title: "Get Folder Item properties using PnP PowerShell : Get-PnPFolderItem versus Get-PnPListItem"
date: 2024-04-01T09:00:00Z
tags: ["SharePoint","PnP","PowerShell", "lists","libraries"]
featured_image: '/posts/images/Pnp-Powershell-Get-Folder-Item/sample.png'
omit_header_text: true
draft: false
---

# Get Folder Item properties using PnP PowerShell : Get-PnPFolderItem versus Get-PnPListItem

## Introduction

In this blog post, we will explore an alternative approach to retrieving folder item properties using PnP PowerShell. We will discuss the limitations of the `Get-PnPFolderItem` cmdlet and demonstrate how to use `Get-PnPListItem` to overcome those limitations.

## The Limitations of Get-PnPFolderItem

The `Get-PnPFolderItem` cmdlet is not suitable for working with large libraries. When attempting to retrieve items from a large library, you may encounter the following error message:
 
```PowerShell
Get-PnPFolderItem : The attempted operation is prohibited because it exceeds the list view threshold.
```

Regrettably, this cmdlet lacks a -PageSize parameter, complicating the handling of large datasets.

## Leveraging Get-PnPListItem
To circumvent the limitations of Get-PnPFolderItem, Get-PnPListItem can be used. This cmdlet enables efficient retrieval of files from large libraries, especially within specific folders, along with their associated properties.


```PowerShell
$SiteUrl = "https://contoso.sharepoint.com/sites/test"
Connect-PnPOnline -url $SiteUrl -Interactive
$FolderSiteRelativeURL = "*Shared Documents/folder/subfolder-folder/subfolder-subfolder-folder*"
$list = Get-PnPList "Shared Documents"
$global:counter = 0
$items = Get-PnPListItem -List "Shared Documents" -PageSize 500 -Fields FileLeafRef,FileRef,PPF_Comments -ScriptBlock `
      { Param($items) $global:counter += $items.Count; Write-Progress -PercentComplete `
    ($global:Counter / ($List.ItemCount) * 100) -Activity "Getting folders from List:" -Status "Processing Items $global:Counter to $($List.ItemCount)";} `
    | Where {$_.FileSystemObjectType -ne "Folder" -and $_.FieldValues.FileRef -like $FolderSiteRelativeURL}
 
$type = [System.Collections.ArrayList]@();
 
$items | foreach-object {
    if($_.FieldValues.Issue_Comments){
        if($type -notcontains $_.FieldValues.Issue_Comments){
            $type.Add([PSCustomObject]@{
                Name = $_.FieldValues.Issue_Comments
            });
            write-host $_.FieldValues.Issue_Comments;
        }
   }
}
 
$type | Export-Csv -Path "C:\temp\categories.csv" -NoTypeInformation -Force -Delimiter "|"
```

In the above script distinct field values from a custom column "Issue_Comments" are retrieved from the sub folder.
