---
title: "PnP Batch updating a large list"
date: 2023-07-20T14:49:19+01:00
tags: ["PnPBatch", "list","SharePoint", "Update"]
draft: true
---

# PnP Batch updating a large list of around 60 k

## Throttling

If your site is being throttled , you might see the following message while navigating to a SharePoint site

![Throttle](../images/PnPBatch-Update-BigList-SharePoint/Throttle_image.png)

We had around 60k items to update on a test environment and still was taking more than 12 hours to update one item at a time. Using **PnP- Batch** helps to half the time with fewer requests sent , however it was prone to throttling which meant without exception handling and retry mechanism might have to start the update script manually.
 
 
```PowerShell
$siteUrl = "https://contoso.sharepoint.com/teams/u-app-ar"
##azure function #Sharepoint App
Connect-PnPOnline –Url $siteUrl -interactive
function UpdateMatchType($MatchTypeColumn,$list){
do {
try {
$batch = New-PnPBatch
$index = 1; 
$itemId = 0; 
$listItems = Get-PnPListItem -List $list  -PageSize 500 | Where {$_.FieldValues.$MatchTypeColumn -ne $null }

 

$totalCount =  $listItems.Count

 

$listItems| ForEach-Object {
    $itemId = $_.Id
   Set-PnPListItem -List $list -Identity $_.Id -Values @{$MatchTypeColumn = $null;} -UpdateType SystemUpdate -Batch $batch

 

if($index % 100 -eq 0 -or $index -eq $listItems.Count){
  write-host "Updating batch starting $index out of $totalCount on library $list"
  Invoke-PnPBatch $batch
  $batch = New-PnPBatch
}
  $index+=1;
}

 

 

Write-Host "Job completed"
$Stoploop = $true

 

}
catch {
if ($Retrycount -gt 3){
Write-Host "Could not send Information after 3 retrys.$itemId after number of item  processed $index"
$Stoploop = $true
}
else {
  Write-Host "Could not send Information retrying in 30 seconds...{$itemId} after number of item  processed {$index}"
  Start-Sleep -Seconds 30
  Connect-PnPOnline –Url $siteUrl -interactive
  $Retrycount = $Retrycount + 1
  }
}
}
While ($Stoploop -eq $false)

 

write-host $("End time " + (Get-Date) + " Updating column: " +  $MatchTypeColumn + "from list " + $listName )
}

 

 

UpdateMatchType "PaymentMatchType" "Remittances" 
UpdateMatchType "UPMMatchType" "Remittances"
UpdateMatchType "MatchType" "Payments"
UpdateMatchType "MatchType" "Recoveries"
UpdateMatchType "MatchType" "UPM"
```

## Embed individual instagram post 

To embed an individual post , use the following embed code below specifying scrolling and frameborder properties. Replace <postRef> with instagram post reference.

