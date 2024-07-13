---
title: "Update Content Type of List Items in SharePoint List"
date: 2024-02-11T06:39:48Z
tags: ["PowerShell","ContentType","SharePoint","Libraries"]
featured_image: '/posts/images/powershell-update-contenttype-values-items/TestDataSuccessfullyGenerated.png'
omit_header_text: true
draft: true
---

```powershell

$siteUrl =  "https://contoso.sharepoint.com/teams/TEAM-TEST"
$list = "Risk Factors"
 
Connect-PnPOnline -url $siteUrl -Interactive
 
$listItems = Get-PnPListItem -List $list  -PageSize 500  -IncludeContentType | Where {$_.ContentType.Name -eq 'Test Document' } | ForEach-Object {

$filePath = $_['FileRef']
#Requirement was based on the folder which was year value, i.e. 2022, 2023, 2024 to update the Year field to the folder name

 $match = [regex]::Match($filePath, "/(\d{4})/") 
    if ($match.Success) {
        $year = $match.Groups[1].Value
        Write-Host "Item $($_.Id) Year extracted from the file path: $year"
        Set-PnPListItem -List $list -Identity $_.Id -ContentType 'Risk Document' -Values @{Year = $year;Activity ='P1 Factors'} -UpdateType SystemUpdate
    } else {
        Write-Host "Item $($_.Id) path: $($filePath) Year not found in the file path."
        Set-PnPListItem -List $list -Identity $_.Id -ContentType 'Risk Document' -Values @{Activity ='P2 Factors'} -UpdateType SystemUpdate
    }
}

```