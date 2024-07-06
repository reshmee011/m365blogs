---
title: "Find SharePoint Framework (SPFx) Packages with PowerShell in Tenant and Site Collection App Catalogs"
date: 2024-07-04T07:17:21+01:00
tags: ["PowerShell", "Inventory","SharePoint Framework (SPFx)","SPFx"]
featured_image: '/posts/images/powershell_find-spfx-installs-in-tenant-sitecollection-appcatalog/example.png'
omit_header_text: true
draft: false
---

# Find SharePoint Framework (SPFx) Packages with PowerShell in Tenant and Site Collection App Catalogs

This post covers a PowerShell script to generate an inventory of SPFx installations within your SharePoint Online environment which will help you maintain oversight of your SPFx solutions, ensuring they are up-to-date and compliant. The script was particularly useful in pinpointing sites within the tenant where third-party applications, specifically an analytics SPFx component, were deployed. This was crucial for ensuring that data collection was confined to designated sites, such as the intranet in my case study. Despite the analytics dashboard aggregating data from all tenant sites, it was challenging to discern the sources of data collection. Therefore, this script was developed to clearly identify the sites from which data were being collected.

## Script

```PowerShell
# Parameters
$AdminCenterURL = "https://contosoonline-admin.sharepoint.com"
$tenantAppCatalogUrl = "https://contosoonline.sharepoint.com/sites/appcatalog"
$sppkgFolder = "./packages"
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$fileName = "\InventorySPFx-" + $dateTime + ".csv"

$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$OutPutView = $directorypath + $fileName

 
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

The PowerShell scripts generates an inventory of SPFx packages installed in both site collection app catalogs and the tenant app catalog exported to a CSV file.
Specific targeted sites are excluded based on the template namely : "PWA#0","SRCHCEN#0", "REDIRECTSITE#0", "SPSMSITEHOST#0", "APPCATALOG#0", "POINTPUBLISHINGHUB#0", "POINTPUBLISHINGTOPIC#0","EDISC#0" and "STS#-1". Please amend the list as appropriate as per your needs.

### Prerequisites

* SharePoint Administrator or Global administrator right to the tenant
* PowerShell PnP module installed

### How to use

1. Copy and paste the above script
2. Amend the values of the variables for the $AdminCenterURL, $tenantappcatalogURL, $sppkgFolder, and filenames to store the output

### How it works

The script iterates through each site collection, checking for SPFx packages in both the site collection app catalog and tenant app catalog. It checks whether the package is installed in the site collection app catalog before checking the tenant app catalog. This is because if the package is installed at both the site collection app catalog and tenant app catalog, the site collection app catalog takes precedence and the package installed in the site will be from site collection app catalog. 

For each SPFx package found, the script collects details such as the site URL, package name, version, and whether it's installed in the tenant app catalog or a site collection app catalog. The inventory data is exported into a CSV file, providing a clear view of SPFx installations across the environment.

![Output](../images/powershell_find-spfx-installs-in-tenant-sitecollection-appcatalog/example.png)

## Conclusion

This PowerShell script can be used by SharePoint administrators and developers looking to audit their SPFx installation efficiently. Feel free to customize the script to fit your specific needs and enhance your SharePoint governance strategy.

Call to Action: Share your experiences using the script.