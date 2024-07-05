---
title: "Get Total Count of SharePoint Files, Folders, and Items with PnP PowerShell"
date: 2024-07-05T07:17:21+01:00
tags: ["PowerShell", "Inventory","SharePoint Online","Libraries","Lists"]
featured_image: '/posts/images/powershell-get-totalcount-of-files-folders-items-in-tenant/example.png'
draft: false
---

# Get Total Count of SharePoint Files, Folders, and Items with PnP PowerShell

This PowerShell script powered by PnP PowerShell can help to get total count of files, folders, and list items across SharePoint tenant. This script is invaluable for administrators looking to perform audits, verify data migrations, or simply keep tabs on the content sprawl within their environments. The use case of this script was to get a total number of items that would be ingested into a third party application Records365 (provided by RecordPoint) to ensure the number tally for compliance purposes and identify any gaps.

## Script Overview

The script connects to SharePoint Online tenant and iterates over each site collection, excluding specific system sites. It tallies the number of files, folders, and list items, excluding items from system lists and libraries. The results are compiled into a CSV report for easy analysis.

### Prerequisites

- SharePoint Online Management Shell
- PnP PowerShell Module

### Script Breakdown


```PowerShell
#Parameters
$SiteURL="https://contosoonline-admin.sharepoint.com/"

$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "SiteStats-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
$Excludedsites =@("https://contosoonline.sharepoint.com/sites/ACT-TrunkDemo-OpenAccess","https://contosoonline.sharepoint.com/teams/TrunkDemo","https://contosoonline.sharepoint.com/sites/Test-OpenAccess"
,"https://contosoonline.sharepoint.com/teams/Test1","https://contosoonline.sharepoint.com/sites/d-intranet",
 "https://contosoonline.sharepoint.com/sites/d-intranet-aboutus","https://contosoonline.sharepoint.com/sites/d-intranet-community", "https://contosoonline.sharepoint.com/sites/d-intranet-directorate",
 "https://contosoonline.sharepoint.com/sites/d-intranet-directorate-comms","https://contosoonline.sharepoint.com/sites/d-intranet-employeehub", "https://contosoonline.sharepoint.com/sites/d-intranet-news","https://contosoonline.sharepoint.com/teams/d-app-caspr-portal","https://contosoonline.sharepoint.com/teams/d-app-app");

$global:foldercounter = 0 
$global:fileCounter = 0 
$global:ItemCounter = 0 
$SiteStats= @()
#Delete the Output Report, if exists
If (Test-Path $OutPutView) { Remove-Item $OutPutView }
Start-Transcript

    #Connect to the Site       
    Connect-PnPOnline -Url $SiteURL -Interactive
    $adminconnection = Get-PnPConnection
    #Array to Skip System Lists and Libraries
    $SystemLists = @("Converted Forms", "Master Page Gallery", "Customized Reports", "Form Templates", "List Template Gallery", "Theme Gallery","Apps for SharePoint",
    "Reporting Templates", "Solution Gallery", "Style Library", "Web Part Gallery","Site Assets", "wfpub", "Site Pages", "Images", "MicroFeed","Pages")
    #Get All Site collections - Exclude: Seach Center, Redirect site, Mysite Host, App Catalog, Content Type Hub, eDiscovery and Bot Sites
    $m365Sites = Get-PnPTenantSite | Where-Object -Property Url -NotIn $Excludedsites  | Where-Object -Property Template -NotIn ("PWA#0","SRCHCEN#0", "REDIRECTSITE#0", "SPSMSITEHOST#0", "APPCATALOG#0", "POINTPUBLISHINGHUB#0", "POINTPUBLISHINGTOPIC#0","EDISC#0", "STS#-1") 
    #($_.Url -like '*/TEAM-*' -or  $_.Url -like '*/ACT-*' -or  $_.Url -like '*/PROJ-*' -or $_.Template -eq 'TEAMCHANNEL#1') -and
    #$m365Sites = Get-PnPTenantSite -Detailed | Where-Object {($_.Url -like '*Comm*') -and $_.Template -ne 'RedirectSite#0' }
    $m365Sites | ForEach-Object {
        Write-host -f Yellow "Scanning site:" $_.Url " Template" $_.Template
        $siteFolderCount = 0;
        $siteFileCount = 0;
        $siteItemCount = 0;
    Try{
        $siteUrl = $_.Url;     
        Connect-PnPOnline -Url $siteUrl -Interactive
        $siteconnection = Get-PnPConnection
                #Get All Lists
        $Lists = Get-PnPList -Includes RootFolder -connection $siteconnection| Where-Object {$_.Hidden -eq $False  }
        #-and $SystemLists -notcontains $_.Title -and $_.BaseTemplate -eq 101
        #Get content types of each list from the web
        
        ForEach($List in $Lists)
        {
            $ListURL =  $List.RootFolder.ServerRelativeUrl

            $folders = Get-PnPListItem -List "$($List.Title)" -PageSize 500 -connection $siteconnection -ScriptBlock `
                { Param($items) $global:counter += $items.Count;} `
                | Where-Object {$_.FileSystemObjectType -eq "Folder"}
            $filesItems = Get-PnPListItem -List "$($List.Title)" -PageSize 500 -connection $siteconnection -ScriptBlock `
                { Param($items) $global:counter += $items.Count;} `
            | Where-Object {$_.FileSystemObjectType -ne "Folder"}
            
            $global:folderCounter += $folders.Count
            $siteFolderCount+=$folders.Count
            if($List.BaseTemplate -eq 101){
              $global:fileCounter += $filesItems.Count
              $siteFileCount +=$filesItems.Count
            }
            else {
               $global:ItemCounter += $filesItems.Count
               $siteItemCount += $filesItems.Count
            }
        }
        $TotalFileCount = Get-PnPFolderStorageMetric -FolderSiteRelativeUrl $siteurl -connection $siteconnection| Select -ExpandProperty TotalFileCount  
        #Collect data
        $Data = [PSCustomObject][ordered]@{
            SiteURL  = $siteUrl
            FilesCount = $siteFileCount
            FolderCount = $siteFolderCount
            ItemCount = $siteItemCount
            SiteFileCount = $TotalFileCount #includes checked out files 
        }
        $SiteStats+= $Data
  }

  Catch {
    write-host -f Red "Error Generating Stats Report!" $_.Exception.Message
  }
  $Data = [PSCustomObject][ordered]@{
    SiteURL  = "tenant"
    FilesCount = $global:filecounter
    FolderCount = $global:foldercounter
    ItemCount = $global:itemcounter
    SiteFileCount = ($global:filecounter + $global:itemcounter)#excludes checked out files  
}
}

$FolderStats | Export-Csv -Path $OutPutView -NoTypeInformation


Stop-Transcript
```

[example output](../images/powershell-get-totalcount-of-files-folders-items-in-tenant/example.png)

This script provides a detailed snapshot of your SharePoint content, helping you manage your digital estate more effectively. Whether you're conducting a routine audit or preparing for a migration, this tool simplifies the process of inventorying your SharePoint assets.