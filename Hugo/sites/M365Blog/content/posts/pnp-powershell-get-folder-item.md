---
title: "Pnp Powershell Get Folder Item"
date: 2024-02-16T16:16:55Z
draft: true
---

Get-PnPFolderItem does not work with large libraries with message 
```powershell
Get-PnPFolderItem : The attempted operation is prohibited because it exceeds the list view threshold.
```
 Unfortunately there is no -PageSize for the cmdlet.
 
Get-PnPListItem can be used to get files from a large library within a particular folder 

```PowerShell
$SiteUrl = "https://contoso.sharepoint.com/sites/test"
Connect-PnPOnline -url $SiteUrl -Interactive
$FolderSiteRelativeURL = "*Shared Documents/folder/subfolder-folder/subfolder-subfolder-folder*"
$list = Get-PnPList "Shared Documents"
# $items = Get-PnPFolderItem -FolderSiteRelativeURL $FolderSiteRelativeURL -Recursive -ItemType File
$global:counter = 0
$items = Get-PnPListItem -List "Shared Documents" -PageSize 500 -Fields FileLeafRef,FileRef,PPF_Comments -ScriptBlock `
      { Param($items) $global:counter += $items.Count; Write-Progress -PercentComplete `
    ($global:Counter / ($List.ItemCount) * 100) -Activity "Getting folders from List:" -Status "Processing Items $global:Counter to $($List.ItemCount)";} `
    | Where {$_.FileSystemObjectType -ne "Folder" -and $_.FieldValues.FileRef -like $FolderSiteRelativeURL}
 
$type = [System.Collections.ArrayList]@();
 
$items | foreach-object {
    if($_.FieldValues.PPF_Comments){
        if($type -notcontains $_.FieldValues.PPF_Comments){
            $type.Add([PSCustomObject]@{
                Name = $_.FieldValues.FileLeafRef
            });
            write-host $_.FieldValues.FileLeafRef;
        }
   }
}
 
$type | Export-Csv -Path "C:\temp\paymentprocessingcategories.csv" -NoTypeInformation -Force -Delimiter "|"
```

