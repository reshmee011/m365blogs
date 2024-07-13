---
title: "Find orphaned choice values in SharePoint list/libraries using PowerShell"
date: 2024-07-12T16:16:09Z
tags: ["SharePoint","PnP","PowerShell","Choice type","SharePoint"]
featured_image: '/posts/images/powershell-find-orphanedvalues-from-listitems-notin-choicevalues/example.png'
omit_header_text: true
draft: true
---

# Find orphaned choice values in SharePoint list/libraries using PowerShell

One common issue is orphaned choice values in SharePoint lists which may affect subsequent update to the list item. This can occur when list items contain values that are no longer valid according to the list's choice field values. This post covers a PowerShell script to identify these orphaned choice values in SharePoint lists to either update them or add them back to the list of choice field values.

## PowerShell Script

The script below iterates through specified sites, to identify any choice values in list items that are not present in the choice field's options/values. 


```PowerShell
 
$list = Read-Host -Prompt "Enter list name ";
$fieldName = Read-Host -Prompt "Enter field name which has the choice values ";

$ExportFileDirectory = (get-location).ToString();
$field = Get-PnPField -List $list -Identity $fieldName
$catChoices = $field.choices
 
$sites = @(
    "https://contoso.sharepoint.com/teams/PMO"
    "https://contoso.sharepoint.com/sites/In"
    "https://contoso.sharepoint.com/teams/Security"
    "https://contoso.sharepoint.com/teams/Dev"
    "https://contoso.sharepoint.com/teams/SD"
)
 
if(Test-Path $ExportFileDirectory){
    $sites | foreach {
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
$type | Export-Csv -Path "C:\temp\all_missingvalues.csv" -NoTypeInformation -Force -Delimiter "|"
```

The scripts prompts the List and Field values. Update the list of SharePoint sites,i.e. `$sites` containing the list and field.

* Identify Orphaned Values: For each site, the script fetches items from the specified list and checks if any item's choice value is not present in the field's predefined choices.
* Export Findings: Finally, it exports the identified orphaned choice values to a CSV file for further review and action.

![outcome](/images/powershell-find-orphanedvalues-from-listitems-notin-choicevalues/example.png)

## Conclusion

This PowerShell script offers a straightforward solution for identifying and addressing orphaned choice values in SharePoint lists, 