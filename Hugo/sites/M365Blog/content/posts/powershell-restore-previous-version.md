---
title: "Restore previous version of list in SharePoint List"
date: 2024-02-11T06:39:48Z
tags: ["PowerShell","version","SharePoint","List"]
featured_image: '/posts/images/powershell-restore-previous-version/example.png'
omit_header_text: true
draft: true
---


```PowerShell
using module PnP.PowerShell
$ItemsRestored = [System.Collections.ArrayList]@();
 
function ListVersions-Data {
Param(
    [Parameter(Mandatory = $true)][string] $List,
    [Parameter(Mandatory = $true)][string] $Key,
    [Parameter(Mandatory = $true)][string] $user,
    [Parameter(Mandatory = $true)][string] $date
 
)
 
$q = @"
<View>
    <Query>
        <Where>
           <Eq><FieldRef Name='Key' /><Value Type='Text'>$Key</Value></Eq>
        </Where>
    </Query>
    <ViewFields>
        <FieldRef Name='Id' />
    </ViewFields>
</View>
"@
$counter = 0;
$items = Get-PnPListItem -Connection $Connection -List $List -Query $q  | Select-Object Id, ObjectVersion
 
  $items|ForEach-Object{
    if($_.ObjectVersion -ne 1){
      $listVersions = Get-PnPListItemVersion -List $List -Identity $_.Id
      $itemId= $_.Id      
      if ($listVersions) {
        if ($listVersions.Count -gt 1) {
        $listVersions| foreach-object{
         if($_.Values.Editor.LookupValue -eq $user -and $_.Created.ToString("yyyy-MM-dd") -eq $date){
          $informationMessage = "Edited by $($_.Values.Editor.LookupValue)  on list $List with ID $itemId on $($_.Created.ToString("yyyy-MM-dd")) with version $($_.VersionLabel)"
          Restore-PnPListItemVersion -Connection $Connection -List $List -Identity $itemId -Version $($_.VersionLabel - 1).ToString(".0")  -Force:$Force
          $informationMessage = "Restored item $itemId to version $(($_.VersionLabel - 1).ToString(".0")) for filter $FilterKey."
 
          Write-Host $informationMessage -ForegroundColor yellow
       
          $ItemsRestored.Add([PSCustomObject]@{
            ItemId =  $itemId
            list = $list
            RestoredVersion = $($_.VersionLabel - 1).ToString(".0")
            FilterKey = $FilterKey  
        });
          $counter+=1;
         }
        }
      }
    }
  }
 }
 Write-Host "Total items updated on $List - $counter for $Key" -ForegroundColor Yellow
}
 
Start-Transcript
     # Generate a unique log file name using today's date and time
     $todayDate = Get-Date -Format "yyyy-MM-dd_HHmm"
 
     $restoredVersion = "restored-file-versions_$todayDate.csv"
     $restoredVersionFilePath = Join-Path -Path $PSScriptRoot -ChildPath $restoredVersion
 
    # Tenant URL
    $siteUrl = Read-Host -Prompt "Enter site collection URL " 
 
    Connect-PnPOnline -Url $siteUrl -Interactive
    $PaymentsList = "Payments"
 
    ListVersions-ARData -List $PaymentsList -Key "202401-Ms-Software" "System Account" "2024-04-29"  
 
 
$ItemsRestored | Export-Csv -Path $restoredVersionFilePath -NoTypeInformation -Force -Delimiter ","
Stop-Transcript
```