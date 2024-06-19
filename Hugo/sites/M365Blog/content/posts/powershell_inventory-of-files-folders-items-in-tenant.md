---
title: "Powershell_inventory of Files Folders Items in Tenant"
date: 2024-06-19T20:09:11+01:00
draft: true
---

```PowerShell
#Parameters
$SiteURL="https://contosoonline-admin.sharepoint.com/"

$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "SiteStats-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
$Excludedsites =@("https://contosoonline.sharepoint.com/sites/ACT-TrunkDemo-OpenAccess","https://contosoonline.sharepoint.com/teams/ACT-TrunkDemo","https://contosoonline.sharepoint.com/sites/PROJ-AnilTest-OpenAccess"
,"https://contosoonline.sharepoint.com/teams/PROJ-AnilTest" ,"https://contosoonline.sharepoint.com/sites/D-PROJ-BakingCompetitionProject","https://contosoonline.sharepoint.com/sites/d-commsite-playground","https://contosoonline.sharepoint.com/sites/d-intranet",
 "https://contosoonline.sharepoint.com/sites/d-intranet-aboutus","https://contosoonline.sharepoint.com/sites/d-intranet-community", "https://contosoonline.sharepoint.com/sites/d-intranet-directorate",
 "https://contosoonline.sharepoint.com/sites/d-intranet-directorate-comms","https://contosoonline.sharepoint.com/sites/d-intranet-employeehub", "https://contosoonline.sharepoint.com/sites/d-intranet-news", "https://contosoonline.sharepoint.com/sites/d-app-littlefish",
 "https://contosoonline.sharepoint.com/teams/d-app-ar", "https://contosoonline.sharepoint.com/teams/d-app-caspr-portal","https://contosoonline.sharepoint.com/teams/d-app-conligo",
 "https://contosoonline.sharepoint.com/teams/d-app-iqm","https://contosoonline.sharepoint.com/teams/d-app-marvin","https://contosoonline.sharepoint.com/teams/d-team-playground","https://contosoonline.sharepoint.com/teams/d-TEAM-DemoTeam", 
 "https://contosoonline.sharepoint.com/teams/d-TEAM-DemoTeam-DemoPrivateChannel","https://contosoonline.sharepoint.com/teams/DevDemoTeam-DemoSharedChannel","https://contosoonline.sharepoint.com/teams/DevDemoTeam-SandboxSharedChannel",
 "https://contosoonline.sharepoint.com/teams/d-app-mars","https://contosoonline.sharepoint.com/teams/D-TEAM-Site","https://contosoonline.sharepoint.com/sites/D-TEAM-Site-OpenAccess","https://contosoonline.sharepoint.com/teams/d-app-software-catalogue",
 "https://contosoonline.sharepoint.com/sites/t-intranet","https://contosoonline.sharepoint.com/sites/t-intranet-aboutus","https://contosoonline.sharepoint.com/sites/t-intranet-community","https://contosoonline.sharepoint.com/sites/t-intranet-directorate",
  "https://contosoonline.sharepoint.com/sites/t-intranet-employeehub","https://contosoonline.sharepoint.com/sites/t-intranet-news","https://contosoonline.sharepoint.com/teams/t-app-ar","https://contosoonline.sharepoint.com/teams/t-app-conligo",
  "https://contosoonline.sharepoint.com/teams/t-app-iqm","https://contosoonline.sharepoint.com/teams/t-app-marvin","https://contosoonline.sharepoint.com/teams/t-app-mars","https://contosoonline.sharepoint.com/teams/t-planner-nickabraham","https://contosoonline.sharepoint.com/sites/u-intranet",
  "https://contosoonline.sharepoint.com/sites/u-intranet-aboutus","https://contosoonline.sharepoint.com/sites/u-intranet-community","https://contosoonline.sharepoint.com/sites/u-intranet-directorate","https://contosoonline.sharepoint.com/sites/u-intranet-employeehub",
   "https://contosoonline.sharepoint.com/sites/u-intranet-home","https://contosoonline.sharepoint.com/sites/u-intranet-home1","https://contosoonline.sharepoint.com/sites/U-Intranet-Home42","https://contosoonline.sharepoint.com/sites/u-Intranet-Home5",
"https://contosoonline.sharepoint.com/sites/u-intranet-news","https://contosoonline.sharepoint.com/sites/u-intranet-selfservice","https://contosoonline.sharepoint.com/teams/u-app-ar","https://contosoonline.sharepoint.com/teams/u-app-caspr-portal",
"https://contosoonline.sharepoint.com/teams/u-app-conligo","https://contosoonline.sharepoint.com/teams/u-app-iqm","https://contosoonline.sharepoint.com/teams/u-app-marvin","https://contosoonline.sharepoint.com/teams/u-app-mars",
"https://contosoonline.sharepoint.com/sites/Test","https://contosoonline.sharepoint.com/sites/Test964","https://contosoonline.sharepoint.com/sites/Test-CreatethecommsforUsers","https://contosoonline.sharepoint.com/sites/TestDT");

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
        
         <#   if(($filesItems.Count + $folders.Count) -ne $list.ItemCount){
                write-host "count don't match"-ForegroundColor red
            }#>
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