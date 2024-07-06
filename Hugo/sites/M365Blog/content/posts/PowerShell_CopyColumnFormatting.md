---
title: "Copy Column View Formatting to different environment using PnP PowerShell"
date: 2023-11-06T11:52:05Z
draft: false
tags: ["PnP","Formatting","Columns","Views", "Permissions"]
omit_header_text: true
featured_image: '/posts/images/PowerShell_CopyColumnFormatting/Output_ColF.PNG'
---

# Copy Column and View Formatting using PnP PowerShell

There is a great sample script how to [backup all column, view and content type formatting](https://github.com/pnp/script-samples/tree/main/scripts/spo-export-all-customformatting) by Dan Toft. If you need to copy column and view formatting across different environments, such as from Dev to Test, UAT, and Prod, especially when dealing with multiple lists/libraries, the script may help you.

## PnP PowerShell Script

```PowerShell
param (
    [Parameter(Mandatory=$false)]
    [string]$SourceSiteUrl = "https://contoso.sharepoint.com/teams/d-app-test",
    [Parameter(Mandatory=$false)]
    [string]$DestinationSiteUrl = "https://contoso.sharepoint.com/teams/t-app-test"
)
$VerbosePreference = 'SilentlyContinue'
#Array of Lists and Libraries
 $Lists = @("list1", "list2", "list3", "lib1", "lib2", "lib3")
# Connect to the source and destination SharePoint sites
Connect-PnPOnline -Url $SourceSiteUrl -Interactive
$SourceConn  = Get-PnPConnection
Connect-PnPOnline -Url $DestinationSiteUrl -Interactive
$DestConn  = Get-PnPConnection
$sWeb = Get-PnPWeb -Connection $SourceConn
$dWeb = Get-PnPWeb -Connection $DestConn
 
# Generate a unique log file name using today's date
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "CopyFieldFormattingReport-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
 
$fieldsGroupCollection = @()
    Write-Host "Starting copy of column and view formatting" -ForegroundColor Yellow;
    Get-PnPList  -Includes Id, Title, Views, Fields -Connection $SourceConn | Where-Object { -not $_.Hidden -and $Lists -contains $_.Title } | ForEach-Object{
        $list = $_
   
        $list.Fields | Where-Object { $null -ne $_.CustomFormatter  -and $_.CustomFormatter -ne "" }| ForEach-Object {
            $field = $_
            try{
                Write-Host "List '$($list.Title)' > field: '$($field.Title)'";
                $dField= Get-PnPField -List $list.Title -Identity $field.Title -Connection $DestConn -ErrorAction Ignore
                if($dField){
                    Set-pnpField -List $list.Title -Identity $field.Title -Connection $DestConn -Values @{CustomFormatter= $field.CustomFormatter}                    
                    $ExportVw = New-Object PSObject
                    $ExportVw | Add-Member -MemberType NoteProperty -name "Source Site Name" -value $sWeb.Title
                    $ExportVw | Add-Member -MemberType NoteProperty -name "Desctination Site Name" -value $dWeb.Title
                    $ExportVw | Add-Member -MemberType NoteProperty -name "List Name" -value $list.Title
                    $ExportVw | Add-Member -MemberType NoteProperty -name "Type" -value "Field"
                    $ExportVw | Add-Member -MemberType NoteProperty -name "Name" -value $field.Title
                    $ExportVw | Add-Member -MemberType NoteProperty -name "Old Json" -value $dField.CustomFormatter
                    $ExportVw | Add-Member -MemberType NoteProperty -name "New Json" -value $field.CustomFormatter
                    $fieldsGroupCollection += $ExportVw
                }
            }
            catch{
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red;
            }
        }
       
        $list.Views | Where-Object { $null -ne $_.CustomFormatter -and $_.CustomFormatter -ne "" }| ForEach-Object {
        $view  = $_
            try {
      
                $NView= Get-PnPView -List $list.Title -Identity $view.Title -Connection $DestConn -ErrorAction Ignore
                if($NView){
                    Set-PnPView -List $list.Title -Identity $view.Title -Connection $DestConn -Values @{CustomFormatter= $view.CustomFormatter} | out-null
                    $ExportVw = New-Object PSObject
                    $ExportVw | Add-Member -MemberType NoteProperty -name "Source Site Name" -value $sWeb.Title
                    $ExportVw | Add-Member -MemberType NoteProperty -name "Desctination Site Name" -value $dWeb.Title
                    $ExportVw | Add-Member -MemberType NoteProperty -name "List Name" -value $list.Title
                    $ExportVw | Add-Member -MemberType NoteProperty -name "Type" -value "View"
                    $ExportVw | Add-Member -MemberType NoteProperty -name "Name" -value $view.Title
                    $ExportVw | Add-Member -MemberType NoteProperty -name "Old Json" -value $NView.CustomFormatter
                    $ExportVw | Add-Member -MemberType NoteProperty -name "New Json" -value $view.CustomFormatter
                    $fieldsGroupCollection += $ExportVw
                }            
            }
            catch {
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red;
            }
     }
    }
    $fieldsGroupCollection | sort-object "List Name" |Export-CSV $OutPutView -Force -NoTypeInformation
```

This script connects to both the source and destination SharePoint sites, and then proceeds to copy the column and view formatting.

## Output

The script generates a CSV report containing details about the copied formatting, such as source site name, destination site name, list name, type (field or view), name of the field/view, old JSON formatting, and new JSON formatting.

Here's an example of the output:
![Output](../images/PowerShell_CopyColumnFormatting/Output_ColF.PNG)

## References

[Export All Formatting](https://github.com/pnp/script-samples/tree/main/scripts/spo-export-all-customformatting)