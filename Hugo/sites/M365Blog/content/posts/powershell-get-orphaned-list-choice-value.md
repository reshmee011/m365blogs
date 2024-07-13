---
title: "Get orphaned choice list values"
date: 2024-07-11T06:39:48Z
tags: ["PowerShell","choice","Orphaned Values"]
featured_image: '/posts/images/powershell-restore-previous-version/example.png'
omit_header_text: true
draft: true
---

```powershell

$list = "Timesheet"
$ExportFileDirectory = (get-location).ToString();
$field = Get-PnPField -List $list -Identity "Category"
$catChoices = $field.choices
 
$sitesWithTimesheet = @(
    "https://contoso.sharepoint.com/teams/TEAM-PMO"
    "https://contoso.sharepoint.com/sites/Infrastructure"
    "https://contoso.sharepoint.com/teams/TEAM-Security"
    "https://contoso.sharepoint.com/teams/TEAM-DevTest"
    "https://contoso.sharepoint.com/teams/TEAM-SD"
)
 
if(Test-Path $ExportFileDirectory){
    $sitesWithTimesheet | foreach {
    connect-pnponline -url $_ -Interactive
    $items = Get-PnPListItem -List $list -PageSize 500 -Fields Category
 
 
    ##get distinct choices within column
 
$type = [System.Collections.ArrayList]@();
$items | foreach-object {
    if($_.FieldValues.Category){
    if($catChoices -notcontains $_.FieldValues.Category){
        if($type.Name -notcontains $_.FieldValues.Category){
            $type.Add([PSCustomObject]@{
                Name = $_.FieldValues.Category
            }) | out-null;
            write-host $_.FieldValues.Category;
        }
    }
  }
 }
 }
}
$type | Export-Csv -Path "C:\temp\all_timesheetmissingcategories.csv" -NoTypeInformation -Force -Delimiter "|"
```