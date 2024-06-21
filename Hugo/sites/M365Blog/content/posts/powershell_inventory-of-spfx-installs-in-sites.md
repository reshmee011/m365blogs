---
title: "Powershell inventory of Spfx Installed in Site Collection and Tenant App Catalog"
date: 2024-06-21T07:17:21+01:00
tags: ["PowerShell", "Inventory","SPFX"]
featured_image: '/posts/images/powershell_inventory-of-spfx-installs-in-sites/example.png'
draft: false
---

```PowerShell
$AdminCenterURL = "https://contosoonline-admin.sharepoint.com"
$tenantAppCatalogUrl = "https://contosoonline.sharepoint.com/sites/appcatalog"
$hubSiteUrl = "https://contosoonline.sharepoint.com"
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "\InventorySPFx-" + $dateTime + ".csv"
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
$istenantApp = ""

$associatedSites | select url | ForEach-Object {
  $Site = Get-PnPTenantSite $_.url -Connection $adminConnection
  Connect-PnPOnline -Url $Site.url -Interactive
  $siteConnection  = Get-PnPConnection   

  try{
  foreach($package in $packageFiles)
  {  
     $packageName = $package.PSChildName
     $appTitle = $null;
      #Find Name of app from installed package
      $RestMethodUrl = '/_api/web/lists/getbytitle(''Apps%20for%20SharePoint'')/items?$select=Title,LinkFilename'
      if((Get-PnPSiteCollectionAppCatalog -CurrentSite)){
        $apps = (Invoke-PnPSPRestMethod -Url $RestMethodUrl -Method Get -Connection $siteConnection).Value
        $appTitle = ($apps | where-object {$_.LinkFilename -eq $packageName} | select Title).Title
        $istenantApp = "Site Collection App Catalog"
      }
      if(!$appTitle)
      {
        $apps = (Invoke-PnPSPRestMethod -Url $RestMethodUrl -Method Get -Connection $appCatConnection).Value
        $appTitle = ($apps | where-object {$_.LinkFilename -eq $packageName} | select Title).Title
        $istenantApp = "Tenant App Catalog"
      }
  
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
        $ExportVw | Add-Member -MemberType NoteProperty -name "Is Tenant" -value $istenantApp
      $SiteAppUpdateCollection += $ExportVw
    }
  }
}
catch{
  write-host -f Red $_.Exception.Message
 }
}

#Export the result Array to CSV file
$SiteAppUpdateCollection | Export-CSV $OutPutView -Force -NoTypeInformation
```