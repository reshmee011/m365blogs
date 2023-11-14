---
title: "Deploying SPFx Packages from Tenant App Catalog to Hub Site and Associated Sites"
date: 2023-11-14T07:06:22Z
tags: ["App Catalog","SPFx","PowerShell", "Deployment"]
draft: true
---

# Deploying SPFx Packages from Tenant App Catalog to Hub Site and Associated Sites

There is the blog post how [Deploying and Installing SharePoint Framework (SPFx) solutions using PnP PowerShell to Hub Site and Associated Sites](https://pnp.github.io/blog/post/deploy-spfx-in-hub-site-and-associated-sites/) using site collection app catalog.  This method is particularly beneficial when SPFx packages shouldn't be globally available during deployment in the tenant level app catalog and instead need targeted deployment or upgrades on specific sites, such as those associated with a hub site in an intranet setup.

Below is a PowerShell script snippet that demonstrates how to manage the deployment and upgrades of SPFx packages from the tenant app catalog to associated sites:

```powershell
$AdminCenterURL=https://contoso-admin.sharepoint.com
$tenantAppCatalogUrl = "https://contoso.sharepoint.com/sites/appcatalog"
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "\IntranetUpgradeSPFx-" + $dateTime + ".csv"
$OutPutView = $directorypath + $fileName
# directory where the SPFx packages are stored
$sppkgFolder = "./packages"
cd $PSScriptRoot

$packageFiles = Get-ChildItem $sppkgFolder
Connect-PnPOnline $AdminCenterURL -Interactive
$adminConnection  = Get-PnPConnection
Connect-PnPOnline $tenantAppCatalogUrl -Interactive
$appCatConnection  = Get-PnPConnection

$ViewCollection = @()
$HubSiteID = (Get-PnPTenantSite  https://contoso.sharepoint.com).HubSiteId

#Get associated sites with hub
$associatedSites = Get-PnPTenantSite -Detailed | Where-Object { $_.HubSiteId -eq $HubSiteID }
#Get all site collections associated with the hub site
#TO Test with updated changes
$associatedSites | select url | ForEach-Object {
    $Site = Get-PnPTenantSite $_.url
     Connect-PnPOnline -Url $Site.url -Interactive
 
      Write-Host ("Deploying packages to {0}..." -f $Site.url) -ForegroundColor Yellow
       foreach($package in $packageFiles)
       {
          $ExportVw = New-Object PSObject
          $ExportVw | Add-Member -MemberType NoteProperty -name "Site URL" -value $Site.url
          $packageName = $package.PSChildName
          Write-Host ("Installing {0}..." -f $packageName) -ForegroundColor Yellow
          $ExportVw | Add-Member -MemberType NoteProperty -name "Package Name" -value $packageName
           #deploy sppkg assuming app catalog is already configured
           Add-pnpapp -Path ("{0}/{1}" -f $sppkgFolder , $package.PSChildName) -Scope Tenant -Overwrite -Publish
           Start-Sleep -Seconds 10

           #Find Name of app from installed package
           $RestMethodUrl = '/_api/web/lists/getbytitle(''Apps%20for%20SharePoint'')/items?$select=Title,LinkFilename'
           $apps = (Invoke-PnPSPRestMethod -Url $RestMethodUrl -Method Get -Connection $appCatConnection).Value
           $appTitle = ($apps | where-object {$_.LinkFilename -eq $packageName} | select Title).Title

      # Get the current version of the SPFx package
    $currentPackage = Get-PnPApp -Identity  $appTitle
    Write-Host "Current package version on site $($site.Url): $($currentPackage.InstalledVersion)"

    # Get the latest version of the SPFx package

    Write-Host "Latest package version: $($currentPackage.AppCatalogVersion)"

    # Update the package to the latest version
    if ($currentPackage.InstalledVersion -ne $currentPackage.AppCatalogVersion) {
        Write-Host "Upgrading package on site $($site.Url) to latest version..."
        Update-PnPApp -Identity $currentPackage.Id
        $currentPackage = Get-PnPApp -Identity $appTitle
        $ExportVw | Add-Member -MemberType NoteProperty -name "Package Version" -value $currentPackage.AppCatalogVersion
        $ViewCollection += $ExportVw
    } else {
        Write-Host "Package already up-to-date on site $($site.Url)."
    }
  }
}

#Export the result Array to CSV file
$ViewCollection | Export-CSV $OutPutView -Force -NoTypeInformation

#Disconnect-PnPOnline
```

## Explanation

This script snippet automates the deployment and potential upgrades of SPFx packages across associated sites of a hub site, leveraging the tenant-level app catalog. It navigates through each associated site, checks for existing packages, installs or upgrades them as needed, and records the deployment details for reporting purposes.
This approach offers flexibility and control over SPFx solutions, ensuring targeted deployment and version management across your SharePoint environment.

Remember to adjust the specific URLs, paths, and configurations based on your SharePoint environment and requirements before executing the script.

