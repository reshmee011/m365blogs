---
title: "Deploying SPFx Packages from Tenant App Catalog to Hub Site and Associated Sites"
date: 2024-03-17T07:06:22Z
tags: ["App Catalog","SPFx","PowerShell", "Deployment"]
featured_image: '/posts/images/PowerShell_SPFXUpgradeFromTenant/Update.png'
omit_header_text: true
draft: false
---

# Deploying SharePoint Framework (SPFx) Packages from Tenant App Catalog to Hub Site and Associated Sites

There is the blog post how [Deploying and Installing SharePoint Framework (SPFx) solutions using PnP PowerShell to Hub Site and Associated Sites using site collection app catalog](https://pnp.github.io/blog/post/deploy-spfx-in-hub-site-and-associated-sites/). This post covers how to perform same objective but using the tenant level app catalog if the SPFx packages have not been added to all sites globally during deployment in the tenant level app catalog and instead need targeted deployment or upgrades on specific sites. 

From the UI when uploading the SPFx package , the **enable app only** is chosen.

![Enable App Only](../images/PowerShell_SPFXUpgradeFromTenant/EnableAppOnly.png)

Using PnP PowerShell if the option -skipfeaturedeployment is omitted the spfx packages is not added to all sites.

```PowerShell
   Add-pnpapp -Path ("{0}/{1}" -f $sppkgFolder , $package.PSChildName) -Scope Tenant -Overwrite -Publish
```

![Not Added to all sites](../images/PowerShell_SPFXUpgradeFromTenant/Enabled_NotAddedToAllSites.png)

Below is a PowerShell script snippet that demonstrates how to manage the deployment and upgrades of SPFx packages from the tenant app catalog to a hub site and associated sites.

## PnP PowerShell Script

Before running the script, ensure to update the following variables 
- SharePoint admin centre Url
```PowerShell
$AdminCenterURL = "https://contoso-admin.sharepoint.com"
```
- Tenant App Catalog site collection Url
```PowerShell
$tenantAppCatalogUrl = "https://contoso.sharepoint.com/sites/apps"
```
- Path where the spfx packages are stored
```PowerShell
$sppkgFolder = "./packages"
```
- Name and Path of the log files
```PowerShell
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "\Log_Tenant-" + $dateTime + ".csv"
$OutPutView = $directorypath + $fileName
```

### Script

```PowerShell
$AdminCenterURL="https://contoso-admin.sharepoint.com"
$tenantAppCatalogUrl = "https://contoso.sharepoint.com/sites/apps"
$hubSiteUrl = "https://contoso.sharepoint.com"
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "\Log_Tenant-" + $dateTime + ".csv"
$OutPutView = $directorypath + $fileName
 
$sppkgFolder = "./packages"

cd $PSScriptRoot
$packageFiles = Get-ChildItem $sppkgFolder

Connect-PnPOnline $tenantAppCatalogUrl -Interactive
$appCatConnection = Get-PnPConnection

Connect-PnPOnline $AdminCenterURL -Interactive
$adminConnection = Get-PnPConnection

$SiteAppUpdateCollection = @()

$associatedSites = Get-PnPHubSiteChild -Identity $hubSiteUrl -Connection $adminConnection

foreach ($package in $packageFiles) {
    $packageName = $package.PSChildName
    Write-Host ("Installing {0}..." -f $packageName) -ForegroundColor Yellow
    # deploy sppkg assuming app catalog is already configured
    Add-pnpapp -Path ("{0}/{1}" -f $sppkgFolder, $package.PSChildName) -Scope Tenant -Overwrite -Publish
}

$associatedSites += $hubSiteUrl # Add the hub site to the list of associated sites

# Get all site collections associated with the hub site
# To Test with updated changes
$associatedSites | ForEach-Object {
    $Site = Get-PnPTenantSite $_ -Connection $adminConnection
    Connect-PnPOnline -Url $Site.url -Interactive
    $siteConnection = Get-PnPConnection
    foreach ($package in $packageFiles) {
        $ExportVw = New-Object PSObject
        $ExportVw | Add-Member -MemberType NoteProperty -name "Site URL" -value $Site.url
        $packageName = $package.PSChildName

        Write-Host "Deploying packages $packageName to $($Site.url)" -ForegroundColor Yellow

        $ExportVw | Add-Member -MemberType NoteProperty -name "Package Name" -value $packageName
        # Find Name of app from installed package
        $RestMethodUrl = '/_api/web/lists/getbytitle(''Apps%20for%20SharePoint'')/items?$select=Title,LinkFilename'
        $apps = (Invoke-PnPSPRestMethod -Url $RestMethodUrl -Method Get -Connection $appCatConnection).Value
        $appTitle = ($apps | where-object { $_.LinkFilename -eq $packageName } | select Title).Title

        $currentPackage = Get-PnPApp -Identity $appTitle -scope Tenant

        # Install App to the Site if not already installed
        $web = Get-PnPWeb -Includes AppTiles -Connection $siteConnection
        $app = $web.AppTiles | where-object { $_.Title -eq $currentPackage.Title }
        if (!$app) {
            Install-PnPApp -Identity $currentPackage.Id -Connection $siteConnection

        } else {
            # Action when all if and elseif conditions are false
            $currentPackage = Get-PnPApp -Identity $appTitle -Connection $siteConnection
            Write-Host "Current package version on site $($site.Url): $($currentPackage.InstalledVersion)"

            Write-Host "Latest package version: $($currentPackage.AppCatalogVersion)"

            # Update the package to the latest version
            if ($currentPackage.InstalledVersion -ne $currentPackage.AppCatalogVersion) {
                Write-Host "Upgrading package on site $($site.Url) to latest version..."
                Update-PnPApp -Identity $currentPackage.Id
                $currentPackage = Get-PnPApp -Identity $appTitle -Connection $siteConnection
                $ExportVw | Add-Member -MemberType NoteProperty -name "Package Version" -value $currentPackage.AppCatalogVersion
            } else {
                Write-Host "Package already up-to-date on site $($site.Url)."
            }
        }
    }
    $SiteAppUpdateCollection += $ExportVw
}

# Export the result Array to CSV file
$SiteAppUpdateCollection | Export-CSV $OutPutView -Force -NoTypeInformation

start-sleep -Seconds 30
foreach ($package in $packageFiles) {
    $packageName = $package.PSChildName
    Write-Host ("Approving {0}..." -f $packageName) -ForegroundColor Yellow
    $RestMethodUrl = '/_api/web/lists/getbytitle(''Apps%20for%20SharePoint'')/items?$select=Title,LinkFilename'
    $apps = (Invoke-PnPSPRestMethod -Url $RestMethodUrl -Method Get -Connection $appCatConnection).Value
    $appTitle = ($apps | where-object { $_.LinkFilename -eq $packageName } | select Title).Title

    $permRequests = Get-PnPTenantServicePrincipalPermissionRequests | where-object { $_.PackageName -eq $appTitle }

    $permRequests | ForEach-Object {
        Write-Host "Approving permission request for $($_.Resource) at scope $($_.Scope) and package $appTitle..."
        Approve-PnPTenantServicePrincipalPermissionRequest -RequestId $_.Id.Guid -Force -ErrorAction Ignore
    }
}

 Disconnect-PnPOnline
```

### Output of run with two SPFx packages across a hub site with associated sites

![Example](../images/PowerShell_SPFXUpgradeFromTenant/Update.png)

## Explanation

This script snippet automates the deployment and potential upgrades of SPFx packages across associated sites of a hub site, leveraging the tenant-level app catalog. It navigates through each associated site, checks for existing packages, installs or upgrades them as needed, and records the deployment details for reporting purposes.
This approach offers flexibility and control over SPFx solutions, ensuring targeted deployment across SharePoint environment.

Remember to adjust the specific URLs, paths, and configurations based on your SharePoint environment and requirements before executing the script.
