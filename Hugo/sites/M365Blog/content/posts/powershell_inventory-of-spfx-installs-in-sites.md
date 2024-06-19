---
title: "Powershell_inventory of Spfx Installs in Sites"
date: 2024-06-19T19:56:15+01:00
draft: true
---

```PowerShell
$AdminCenterURL="https://contosoonline-admin.sharepoint.com"
$tenantAppCatalogUrl = "https://contosoonline.sharepoint.com/sites/appcatalog"
$hubSiteUrl = "https://contosoonline.sharepoint.com"
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "\ConnectOnlineInventorySPFx-" + $dateTime + ".csv"
$OutPutView = $directorypath + $fileName

$sppkgFolder = "./packages"
 
cd $PSScriptRoot
$packageFiles = Get-ChildItem $sppkgFolder
 
Connect-PnPOnline $tenantAppCatalogUrl -Interactive
$appCatConnection  = Get-PnPConnection
 
Connect-PnPOnline $AdminCenterURL -Interactive
$adminConnection  = Get-PnPConnection
 
$SiteAppUpdateCollection = @()
 
#Get associated sites with hub
$associatedSites = Get-PnPTenantSite -Detailed -Connection $adminConnection  | Where-Object -Property Template -NotIn ("PWA#0","SRCHCEN#0", "REDIRECTSITE#0", "SPSMSITEHOST#0", "APPCATALOG#0", "POINTPUBLISHINGHUB#0", "POINTPUBLISHINGTOPIC#0","EDISC#0", "STS#-1") 

foreach($package in $packageFiles)
{ 
 Write-Host ("finding packages to {0}..." -f $Site.url) -ForegroundColor Yellow

   $packageName = $package.PSChildName

    #Find Name of app from installed package
    $RestMethodUrl = '/_api/web/lists/getbytitle(''Apps%20for%20SharePoint'')/items?$select=Title,LinkFilename'
    $apps = (Invoke-PnPSPRestMethod -Url $RestMethodUrl -Method Get -Connection $appCatConnection).Value
    $appTitle = ($apps | where-object {$_.LinkFilename -eq $packageName} | select Title).Title

$associatedSites | select url | ForEach-Object {
try{

    $Site = Get-PnPTenantSite $_.url -Connection $adminConnection
     Connect-PnPOnline -Url $Site.url -Interactive
     $siteConnection  = Get-PnPConnection      
        $web = Get-PnPWeb -Includes AppTiles -Connection $siteConnection
        $app = $web.AppTiles  |  where-object {$_.Title -eq $appTitle }
        $currentPackage = Get-PnPApp -Identity  $appTitle -Connection $siteConnection
       if($currentPackage.InstalledVersion){
          Write-Host "Current package version on site $($site.Url): $($currentPackage.InstalledVersion)"
          $ExportVw = New-Object PSObject
          $ExportVw | Add-Member -MemberType NoteProperty -name "Site URL" -value $Site.url
          $ExportVw | Add-Member -MemberType NoteProperty -name "Package Name" -value $packageName
           $ExportVw | Add-Member -MemberType NoteProperty -name "Hub Site Name" -value  (get-pnphubsite -identity $Site.HubSiteId.Guid).title
           $ExportVw | Add-Member -MemberType NoteProperty -name "Is Hub Site" -value $Site.IsHubSite
           $ExportVw | Add-Member -MemberType NoteProperty -name "Version" -value $currentPackage.InstalledVersion
          $SiteAppUpdateCollection += $ExportVw
        }
     }
     catch{
        write-host -f Red $_.Exception.Message
    }
}
}

#Export the result Array to CSV file
$SiteAppUpdateCollection | Export-CSV $OutPutView -Force -NoTypeInformation
 

```