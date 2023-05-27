---
title: 'PnP Batch Add /Delete 300k items from a SharePoint Online list'
date: Sun, 12 Sep 2021 09:09:39 +0000
draft: false
tags: ['PnP', 'SharePoint Online', 'Uncategorized']
---

I was tasked to delete more 300k items from a SharePoint list generated as part of testing. I tried the script https://vladilen.com/office-365/spo/fastest-way-to-delete-all-items-in-a-large-list/

```
Get-PnPListItem -List $list -Fields "ID" -PageSize 100 -ScriptBlock { Param($items) $items | Sort-Object -Property Id -Descending | ForEach-Object{ $\_.DeleteObject() } } 
```

However the script kept failing at irregular intervals with exception messages like "A task was cancelled", "The collection has not been initialised" and "operation time out". It was a long processing task which I need to keep monitoring and resumed the script manually to continue the deletion many times to my dismay. What I thought was a simple task ended being so tedious. I still needed a script which can delete 300k items without stopping each time an error is encountered due to throttling issue, token expiring or network connectivity issues.

[![](https://reshmeeauckloo.files.wordpress.com/2021/09/operationsonlargelisterrors.png?w=990)](https://reshmeeauckloo.files.wordpress.com/2021/09/operationsonlargelisterrors.png)

I used the [PnPBatch](https://pnp.github.io/powershell/articles/batching.html) to add/delete 1 k items at one time as there is a limit on the number of operations that can be passed as part of PnPBatch

```
$action = Read-Host "Enter the action you want to perform, e.g. Add or Delete"
$siteUrl = "https://contoso.sharepoint.com/sites/Team1"
$listName = "TestDemo" 
$ErrorActionPreference="Stop"
Connect-PnPOnline –Url $siteUrl -interactive


$Stoploop = $false

[int]$Retrycount = "0"


write-host $("Start time " + (Get-Date))
do {
try {

if($action -eq "Add")
{   $lst = Get-PnPList -Identity $listName
    
    if($lst.ItemCount -lt 300000)
    {
       $startInc = $lst.ItemCount
       while($lst.ItemCount -lt 300000)
       {
      
       $batch = New-PnPBatch
        #perform in increment of 1000 until 300k is reached 
       if($startInc+1000 -gt 300000)
        {
         $endNu = 300000
        } 
        else
        {
        $endNu = $startInc+1000
        }
        for($i=$startInc;$i -lt ($endNu);$i++)
        {
            Add-PnPListItem -List $listName -Values @{"Title"="Test $i"} -Batch $batch
        }
        Invoke-PnPBatch -Batch $batch
         $lst = Get-PnPList -Identity $listName
       }
    }
}

if($action -eq "Delete")
{
 $listItems= Get-PnPListItem -List $listName -Fields "ID" -PageSize 1000  
$itemIds = $lisItems | Foreach {$_.Id}

$itemCount = $listItems.Count
while($itemCount -gt 0)
{
    $batch = New-PnPBatch
    #delete in batches of 1000, if itemcount is less than 1000 , all will be deleted 

    if($itemCount -lt 1000)
    {
     $noDeletions = 0
    }
    else
    {
     $noDeletions = $itemCount -1000
    }

    for($i=$itemCount-1;$i -ge $noDeletions;$i--)
    {
        Remove-PnPListItem -List $listName -Identity $listItems[$i].Id -Batch $batch 
    }
    Invoke-PnPBatch -Batch $batch
    $itemCount = $itemCount-1000
}

}

Write-Host "Job completed"

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
write-host $("End time " + (Get-Date))
```

The script took nearly 4 hours to create 300k items in a SharePoint list with calling retry once.

[![](https://reshmeeauckloo.files.wordpress.com/2021/09/add_longoperation.png?w=601)](https://reshmeeauckloo.files.wordpress.com/2021/09/add_longoperation.png)

The script took 7.5 hours to delete 300k items calling the retry twice.

[![](https://reshmeeauckloo.files.wordpress.com/2021/09/delete_longoperation.png?w=619)](https://reshmeeauckloo.files.wordpress.com/2021/09/delete_longoperation.png)