---
title: "Update Choice values of List Items in SharePoint List"
date: 2024-07-13T06:39:48Z
tags: ["PowerShell","Choice","SharePoint","List"]
featured_image: '/posts/images/powershell-update-choice-value-from-old-to-new/csv.png'
omit_header_text: true
draft: false
---

# Update Choice values of List Items in SharePoint List

Maintaining up-to-date list items, especially when dealing with choice fields, can be a daunting task sepcially after choice field values are updated. We need a way to update these old values to the correct new ones.

## The Challenge

SharePoint lists use choice fields to categorise items. Over time, the need to update these choice values can arise, whether due to changes in terminology, business processes, or error correction. Manually updating these values across numerous list items is not only time-consuming but also prone to human error.

## The PowerShell Solution

The PowerShell script automates the update of choice values in SharePoint list items. 

## Solution : PowerShell Script

To use the script prepare a csv file with mapping of old value to new value

![csv](../images/powershell-update-choice-value-from-old-to-new/csv.png)

```PowerShell
$SiteCollectionUrl = Read-Host -Prompt "Enter site collection URL";
$csvPath = Read-Host -Prompt "Enter csv path ";
$list = Read-Host -Prompt "Enter list name ";
$fieldName = Read-Host -Prompt "Enter field name which has the choice values ";

connect-pnponline -url $SiteCollectionUrl -Interactive

$ids = [System.Collections.ArrayList]@();

if(Test-Path $csvPath){
  Import-Csv -Path $csvPath | foreach-object{
  $row = $_
  $q = @"
<View>
    <Query>
        <Where>
           <Eq><FieldRef Name=$fieldName/><Value Type='Text'>$($row.OldValue)</Value></Eq>
        </Where>
    </Query>
    <ViewFields>
        <FieldRef Name='Id' />
        <FieldRef Name=$fieldName />
    </ViewFields>
</View>
"@
 
$items = Get-PnPListItem -Connection $Connection -List $list -Query $q  | Select-Object Id, Category
$items | ForEach-Object{
$ids.Add([PSCustomObject]@{
    Id = $_.Id
    OldValue = $row.OldValue
    NewValue = $row.NewValue
}) | out-null;
 
   Set-PnPListItem -List $list -Identity $_.Id -Values @{$fieldName= $row.NewValue;} -UpdateType SystemUpdate
  }
 
  }
} 
$ids | Export-Csv -Path "C:\temp\id_toupdate.csv" -NoTypeInformation -Force -Delimiter "|"
```

The script :

1. **Connect to SharePoint Online**: The script starts by prompting the user for the site collection URL and establishing a connection using `Connect-PnPOnline`.

    ```PowerShell
    $SiteCollectionUrl = Read-Host -Prompt "Enter site collection URL: ";
    connect-pnponline -url $SiteCollectionUrl -Interactive
    ```

2. **Read from CSV**: It reads a CSV file specified by the user, which contains the old and new choice values for the update.

    ```PowerShell
    $csvPath = Read-Host -Prompt "Enter timesheet csv path ";
    ```

3. **Query List Items**: For each row in the CSV, the script constructs a CAML query to find list items with the old choice value from the specified list and column

```PowerShell
$fieldName = Read-Host -Prompt "Enter field name which has the choice values ";

  $q = @"
<View>
    <Query>
        <Where>
           <Eq><FieldRef Name=$fieldName/><Value Type='Text'>$($row.OldValue)</Value></Eq>
        </Where>
    </Query>
    <ViewFields>
        <FieldRef Name='Id' />
        <FieldRef Name=$fieldName />
    </ViewFields>
</View>
"@
```

4. **Update List Items**: Each identified list item is updated with the new choice value using `Set-PnPListItem`.

    ```PowerShell
    Set-PnPListItem -List $list -Identity $_.Id -Values @{$fieldName= $row.NewValue;} -UpdateType SystemUpdate
    ```

5. **Export Updated Item IDs**: Finally, the script exports the IDs of the updated items to a CSV file, providing a clear record of the changes made.

    ```PowerShell
    $ids | Export-Csv -Path "C:\temp\timesheet_id_toupdate.csv" -NoTypeInformation -Force -Delimiter "|"
    ```

## Conclusion

This PowerShell script simplifies the process of updating choice values in SharePoint list items. By automating this task, organizations can ensure that their SharePoint lists remain accurate and up-to-date with minimal effort. 
