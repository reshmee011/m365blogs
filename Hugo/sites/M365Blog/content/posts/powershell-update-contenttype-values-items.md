---
title: "Update Content Type of List Items in SharePoint List"
date: 2024-07-11T06:39:48Z
tags: ["PowerShell","Content Type","SharePoint","Data Management"]
featured_image: '/posts/images/powershell-update-contenttype-values-items/ContentTypeAmended.png'
omit_header_text: true
draft: false
---

# Update Content Type of List Items in SharePoint List

A content type in SharePoint is a reusable collection of metadata (columns) allowing to organise, manage and handle content in a consistent way. If the content type of files and items need to be updated because of required changes how particular content have to be managed, it can be a daunting laborious task if done manually. PowerShell can help automating the update of content types and corresponding metadata (columns).

## Script

Before running the script please update the variables $siteUrl, $list, $oldContentType and $newContentType to the correct siteurl, list, oldcontenttype and newcontenttype values. Also review the column names.

In my scenerio, the additional Year column part of the new content type needed to be updated to the folder name which was the year value, i.e. 2022, 2024, 2023. However the files could be within nested folders in the folder named after the year.

 `[regex]::Match($filePath, "(\d{4})")` searches the string contained in $filePath for the first sequence of exactly four digits and captures that sequence. This is commonly used to find a year in a string, assuming the year is represented by four consecutive digits.

```powershell

$siteUrl =  "https://contoso.sharepoint.com/teams/TEAM-TEST"
$list = "Risk Factors" 
$oldContentType = 'Test Document'
$newContentType = 'Risk Document'

Connect-PnPOnline -url $siteUrl -Interactive
 
$listItems = Get-PnPListItem -List $list  -PageSize 500  -IncludeContentType | Where {$_.ContentType.Name -eq $oldContentType } | ForEach-Object {

$filePath = $_['FileRef']
#Requirement was based on the folder which was year value, i.e. 2022, 2023, 2024 to update the Year field to the folder name

 $match = [regex]::Match($filePath, "/(\d{4})/") 
    if ($match.Success) {
        $year = $match.Groups[1].Value
        Write-Host "Item $($_.Id) Year extracted from the file path: $year"
        Set-PnPListItem -List $list -Identity $_.Id -ContentType $newContentType -Values @{Year = $year;Activity ='P1 Factors'} -UpdateType SystemUpdate
    } else {
        Write-Host "Item $($_.Id) path: $($filePath) Year not found in the file path."
        Set-PnPListItem -List $list -Identity $_.Id -ContentType 'Risk Document' -Values @{Activity ='P2 Factors'} -UpdateType SystemUpdate
    }
}

```

## Conclusion

This PowerShell script can help to automate the process of updating SharePoint list items or files, saving time and reducing the potential for errors. Feel free to amend to suit your needs. 