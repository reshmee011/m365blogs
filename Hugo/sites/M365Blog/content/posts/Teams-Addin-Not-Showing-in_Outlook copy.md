---
title: "Teams Add-in Not Showing Within Outlook"
date: 2023-07-20T14:49:19+01:00
tags: ["Outlook", "Teams","Add-in"]
draft: false
---

# Teams Add-in Not Showing Within Outlook

The add-in for setting up a teams meeting within the outlook application is no longer showing, It does not show in disabled items. 
Teams has been reinstalled for both affected users and still not showing. 

dentified a missing reg key within the registry which has been attached. 

The Fix we completed is as followed. We identified the key by searching the 3 locations for any missing key. 

HKEY_CURRENT_USER\Software\Microsoft\Office\Outlook\Addins\TeamsAddin.FastConnect
HKEY_CLASSES_ROOT\TeamsAddin.FastConnect.1
HKEY_CLASSES_ROOT\TeamsAddin.FastConnect.1\CLSID

Once we identified the missing key we took this from a working computer and then moved this and imported this into the damaged computer. Once Outlook had been restarted the add-in was then available. 

The reg key has been attached the ticket. Please remove the .txt part of the file name as unable to upload .reg file.

The fault was diagnosed by Microsoft Support as being an issue with a previous upgrade which removed the key.

There are 2 workarounds available. 1. Use Team calendar to create meeting, 2. Use OWA to create meetings
If your site is being throttled , you might see the following message while navigating to a SharePoint site

![Throttle](../images/PnPBatch-Update-BigList-SharePoint/Throttle_image.png)

We had around 60k items to update on a test environment and still was taking more than 12 hours to update one item at a time. Using **PnP- Batch** helps to half the time with fewer requests sent , however it was prone to throttling which meant without exception handling and retry mechanism might have to start the update script manually.
 
 
```PowerShell
$siteUrl = "https://contoso.sharepoint.com/teams/test"

$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "\GroupRefUpdateReport-" + $dateTime + ".csv"
$OutPutView = $directorypath + $fileName
$fileName = "\GroupRefReport-" + $dateTime + ".csv"
$OutPutGroup = $directorypath + $fileName

$list = "Receipts"

Connect-PnPOnline –Url $siteUrl -interactive
$listItems = Get-PnPListItem -List $list  -PageSize 500 | Where {$_.FieldValues.GroupingReference -ne $null } #-and $_.FieldValues.ParentId -eq $null
$DataCollection = @()
$UpdateLog = @()

ForEach($Item in $ListItems)
{
    #Collect data       
    $Data = New-Object PSObject -Property @{
            GroupingRef  = $Item["GroupingReference"]
            ParentId  = $Item["ParentId"].LookupValue
           # Fund  = $Item["MemberType"]
            ID = $Item.Id
        }
    $DataCollection += $Data
}
$Groups = $DataCollection | Sort-Object -Property ID | Group-Object -Property GroupingRef  | Where-Object {$_.Count -gt 1} 
#| Select-Object -ExpandProperty Group

$Groups | Export-CSV $OutPutGroup -Force -NoTypeInformation

$arraylist = New-Object System.Collections.ArrayList
$index= 1;

function UpdateItems($items){
$Stoploop = $false

[int]$Retrycount = "0"
do {
try {
#itemId,$parentId,$list
#Update item
  $batch = New-PnPBatch
$items | foreach-object {
Set-PnPListItem -List $_.list -Identity $_.id -Values @{ParentId = $_.parentId;} -UpdateType SystemUpdate -Batch $batch

}
Write-Host "Updating $index"
  Invoke-PnPBatch $batch
$Stoploop = $true

}
catch {
if ($Retrycount -gt 3){
Write-Host "Could not send Information after 3 retrys."
$Stoploop = $true
}
else {
  Write-Host "Could not send Information retrying in 30 seconds..."
  Start-Sleep -Seconds 30
  Connect-PnPOnline –Url $siteUrl -interactive
  $Retrycount = $Retrycount + 1
  }
}
}
While ($Stoploop -eq $false)
}

ForEach($Group in $Groups) 
{
   $ParenId = ($Group.Group | Select-Object -first 1).ID 
   $log = New-Object PSObject -Property @{
       GroupingRef  = ($Group.Group | Select-Object -first 1).GroupingRef
       CountOfRecordsUpdated  =  $Group.Count - 1 
       StartTime = (Get-Date).toString("dd-MM-yyyy HH:mm:ss")

     }
    $Group.Group | Select-Object -Skip 1 | ForEach-Object {
    ##logic to update rest of records albeit the first on
    #Write-Host $("Set Parent Id to " + $ParenId + " for item id " + $_.ID)
    ##send in batches of 100 to update
    $itemtoUpdate = "" | Select-Object id, parentId,list
     $itemtoUpdate.id = $_.ID
     $itemtoUpdate.parentId = $ParenId
     $itemtoUpdate.list = $list
    $arraylist.Add($itemtoUpdate) | Out-Null 

    if($index % 100 -eq 0 ){
     UpdateItems $arraylist
     $arraylist = New-Object System.Collections.ArrayList
     }
     $index+=1

    }  
    $log | Add-Member -MemberType NoteProperty -name "EndTime" -value (Get-Date).toString("dd-MM-yyyy HH:mm:ss")
    $UpdateLog += $log
}
#in case there are remaining items to update 
if($index % 100 -ne 0){
  UpdateItems $arraylist
}

 

$UpdateLog | Export-CSV $OutPutView -Force -NoTypeInformation
```

## Embed individual instagram post 

To embed an individual post , use the following embed code below specifying scrolling and frameborder properties. Replace <postRef> with instagram post reference.

