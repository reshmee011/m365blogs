---
title: "Optimising Large List Updates with PnP Batch: Handling Throttling and Enhancing Efficiency"
date: 2023-07-25T14:49:19+01:00
tags: ["PnPBatch", "list","SharePoint", "SystemUpdate"]
draft: false
---

# Optimising Large List Updates with PnP Batch: Handling Throttling and Enhancing Efficiency

In this article, we explore how to efficiently update a large SharePoint list containing approximately 60,000 items using **PnP-Batch**. Updating such a substantial number of items individually can be time-consuming and prone to throttling issues. Prior to using PnP Batch , it was taking more than 12 hours to update 60k one by one. The article highlights the benefits of using PnP Batch, which significantly reduces the time taken for updates by sending fewer requests. However, it also addresses the throttling challenge and demonstrates the importance of implementing exception handling and retry mechanisms to ensure a smooth update process. The article provides insights into handling throttling and showcases the impact on performance, ultimately improving the update process for large SharePoint lists.

The dreaded throttling message when accessing a site if tenant is being throttled which may mean running the script out of office hours or over the weekend.  

![Throttle](../images/PnPBatch-Update-BigList-SharePoint/Throttle_image.png)

Or you may get 429 http response from API calls as behind the scenes, throttling happens based on resource availability on the servers. You can reach out to Microsoft via a case to identify the exact cause of throttling in your tenant if it is affecting general usage of SharePoint sites.

 
```PowerShell
$siteUrl = "https://contoso.sharepoint.com/teams/app-test"
##azure function #Sharepoint App
Connect-PnPOnline –Url $siteUrl -interactive
function UpdateType($TypeColumn,$list){
do {
try {
$StopLoop = $false
$batch = New-PnPBatch
$index = 1; 
$itemId = 0; 
$listItems = Get-PnPListItem -List $list  -PageSize 500 | Where {$_.FieldValues.$TypeColumn -ne $null }
$totalCount =  $listItems.Count

$listItems| ForEach-Object {
    $itemId = $_.Id
   Set-PnPListItem -List $list -Identity $_.Id -Values @{$TypeColumn = $null;} -UpdateType SystemUpdate -Batch $batch

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

write-host $("End time " + (Get-Date) + " Updating column: " +  $TypeColumn + "from list " + $listName )
}

UpdateType "Type" "List1" 
```

The script defines a function called UpdateType responsible for updating the "Type" column in the specified list. It contains a do-while loop that executes the main update logic until the $Stoploop variable becomes true. This loop is used for error handling and retrying the updates if something goes wrong. Inside the loop, the function sets up a batch using New-PnPBatch cmdlet. A batch is a way of grouping multiple operations into a single request to SharePoint, which can improve performance.

The list items from the specified list are retrieved using the Get-PnPListItem cmdlet with the -PageSize parameter set to 500 (meaning it retrieves up to 500 items at a time) until all items are returned. For each list item retrieved, the function proceeds to update the "TypeColumn" to null using the Set-PnPListItem cmdlet. It does this as a system update to prevent triggering any workflows or versioning that might be configured for the list.

After updating each item, the function checks if the current index is a multiple of 100 or if it has reached the total count of list items. If so, it writes a progress message to the console, executes the batch using Invoke-PnPBatch cmdlet, and then creates a new batch for the next set of updates.

The loop continues until all list items have been processed.

If an error occurs during the update process, the function catches the exception, and if the $Retrycount is greater than 3 (meaning it has retried more than three times), it stops the loop. Otherwise, it writes an error message to the console, waits for 30 seconds, and then tries to connect to the SharePoint site again before incrementing the $Retrycount and trying the update process again.

![Script Update](../images/PnPBatch-Update-BigList-SharePoint/ThrottlingIssue.png)

This script seems to be designed to handle errors gracefully and ensure that all items in the list are processed, even if there are temporary connection issues with SharePoint. In the above screenshot the script attempted retry at random interval 1880 to 3422 and the update rate was around 10k items per hour. T
