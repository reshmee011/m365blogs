---
title: "SharePoint Portals Site collections Inventory"
date: 2024-07-19T09:53:05+01:00
tags: ["SharePoint", "Site Collections", "Portals","PowerShell","Inventory"]
featured_image: '/posts/images/SharePoint-Portal-sitecollections-Inventory/Script_result.png'
omit_header_text: true
draft: false
---

As part of the Copilot for M365 rollout , questions were raised on the `/portals/Community` and `/portals/hub` as both 'Everyone Except External Users' was granted access raising concerns on the content on the site. These are legacy sites and are currently inaccessible. Historically, these sites were accessible via the legacy SharePoint admin centre, however, they are not available through the modern SharePoint admin centre interface.

Thanks to Gregory Zelfond's blog post [What are all these site collections in SharePoint?](https://sharepointmaven.com/site-collections-sharepoint/) I found why these sites exist. 

## /portals/Community

This site collection is a dedicated space with the Community features enabled. It allows Administrators to manage discussions in a virtual environment. Read about this site collection [here](https://support.office.com/en-us/article/Create-a-community-8b6bb936-7ebc-4e60-b8ab-2d4897499af9).

## /portals/hub 

This is a site collection that contains all the videos posted to Office 365 Video Portal. Check it out [here](https://sharepointmaven.com/how-to-add-a-video-in-sharepoint/). Office 365 Video Portal is like an internal YouTube for your organization. Now Stream is the preferred option for videos

## Contents

The sites can't be navigated from the UI

![Site can't be reached from UI](../images/SharePoint-Portal-sitecollections-Inventory/Portals_Community_Inaccessible.png)

PowerShell came to the rescue to monitor contents within the portals site.

```PowerShell
$SiteStats= @()
#Delete the Output Report, if exists
If (Test-Path $OutPutView) { Remove-Item $OutPutView }
Start-Transcript
    $siteFolderCount = 0;
    $siteFileCount = 0;
    $siteItemCount = 0;
    Try{
        $siteUrl = Read-Host "Enter site url";
        Connect-PnPOnline -Url $siteUrl -Interactive
        $siteconnection = Get-PnPConnection
        #Get All Lists
        $Lists = Get-PnPList -Includes RootFolder -connection $siteconnection| Where-Object {$_.Hidden -eq $False  }
       
        ForEach($List in $Lists)
        {
            $ListURL =  $List.RootFolder.ServerRelativeUrl
 
            $folders = Get-PnPListItem -List "$($List.Title)" -PageSize 500 -connection $siteconnection -ScriptBlock `
                { Param($items) $global:counter += $items.Count;} `
                | Where-Object {$_.FileSystemObjectType -eq "Folder"}
            $filesItems = Get-PnPListItem -List "$($List.Title)" -PageSize 500 -connection $siteconnection -ScriptBlock `
                { Param($items) $global:counter += $items.Count;} `
            | Where-Object {$_.FileSystemObjectType -ne "Folder"}
           
            $siteFolderCount+=$folders.Count
            if($List.BaseTemplate -eq 101){  
              $siteFileCount +=$filesItems.Count
            }
            else {
               $siteItemCount += $filesItems.Count
            }
 
            Get-PnPListItem -List "$($List.Title)" -PageSize 500 -connection $siteconnection | ForEach-Object{
                write-host  "file  $($_.fieldValues.FileLeafRef)  in List $($List.Title)"
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
 
Write-host "Total count within site"
$SiteStats
Stop-Transcript
```

![Script inventoryresult](../images/SharePoint-Portal-sitecollections-Inventory/Script_result.png)

File count for both sites are 6 if no additional file was added to the sites

**Details of files**

* file  PointPublishing.aspx  in List AppPages
* file  VideoEmbedHost.aspx  in List AppPages
* file  1_.000  in List Hub Settings
* file  Media Player  in List Style Library
* file  AlternateMediaPlayer.xaml  in List Style Library
* file  AudioPreview.png  in List Style Library
* file  VideoPreview.png  in List Style Library
* file  MediaWebPartPreview.png  in List Style Library



## Conclusion

All these sites are Out of the Box sites which came with the tenant.  Impact will need to be assessed before deleting those.