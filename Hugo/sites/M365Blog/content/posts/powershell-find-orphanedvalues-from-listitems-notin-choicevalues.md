---
title: "Find orphaned values not available within choice values"
date: 2024-02-09T16:16:09Z
tags: ["SharePoint","PnP","PowerShell","Choice","SharePoint Online"]
featured_image: '/posts/images/PowerShell_PnPUnifiedLog/Sample.png'
omit_header_text: true
draft: false
---

```PowerShell
#$SiteCollectionUrl = Read-Host -Prompt "Enter site collection URL: ";
 
$list = "Timesheet"
$ExportFileDirectory = (get-location).ToString();
$field = Get-PnPField -List $list -Identity "Category"
$catChoices = $field.choices
 
$sitesWithTimesheet = @(
    "https://contoso.sharepoint.com/teams/PMO"
    "https://contoso.sharepoint.com/sites/In"
    "https://contoso.sharepoint.com/teams/Security"
    "https://contoso.sharepoint.com/teams/Dev"
    "https://contoso.sharepoint.com/teams/SD"
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