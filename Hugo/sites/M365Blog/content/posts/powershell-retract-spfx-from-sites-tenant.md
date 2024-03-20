---
title: "Retracting SPFx Solutions from Tenant Sites using PnP PowerShell"
date: 2024-03-19T08:25:29Z
tags: ["App Catalog","SPFx","PowerShell", "Uninstall"]
featured_image: '/posts/images/powershell-retract-spfx-from-sites-tenant/retract.png'
draft: false
---

# Retracting SPFx Solutions from Hub Site and associated sites using PnP PowerShell

SharePoint Framework (SPFx) solutions are a powerful tool for extending and customizing SharePoint sites. However, managing these solutions across multiple sites in a SharePoint tenant can be a daunting task. Fortunately, PnP PowerShell provides automation capabilities that can streamline these operations and ensure consistency across the tenant.

The blog post [Deploying SharePoint Framework (SPFx) Packages from Tenant App Catalog to Hub Site and Associated Sites](https://reshmeeauckloo.com/posts/powershell_spfxdeploytohubfromtenant/) covers how to deploy SPFx solutions across a hub site and associated sites. This current post covers how to retract SPFx solutions from a hub site and associated sites within a tenant. 

## Script

```PowerShell
$AdminCenterURL="https://contoso-admin.sharepoint.com"
$tenantAppCatalogUrl = "https://contoso.sharepoint.com/sites/apps"
$hubSiteUrl = "https://contoso.sharepoint.com/sites/Contosoinc-intranet"
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "\Log_Tenant_Removing_Solution" + $dateTime + ".csv"
$OutPutView = $directorypath + $fileName
 
$sppkgFolder = "./packages"
 
cd $PSScriptRoot
$packageFiles = Get-ChildItem $sppkgFolder
 
Connect-PnPOnline $tenantAppCatalogUrl -Interactive
$appCatConnection  = Get-PnPConnection
 
Connect-PnPOnline $AdminCenterURL -Interactive
$adminConnection  = Get-PnPConnection
 
$SiteAppUpdateCollection = @()

$associatedSites = Get-PnPHubSiteChild -Identity $hubSiteUrl -Connection $adminConnection
$associatedSites += $hubSiteUrl #Add the hub site to the list of associated sites
#Get all site collections associated with the hub site
$associatedSites | ForEach-Object {
    $Site = Get-PnPTenantSite $_ -Connection $adminConnection
     Connect-PnPOnline -Url $Site.url -Interactive
     $siteConnection  = Get-PnPConnection
       foreach($package in $packageFiles)
       {
        $ExportVw = New-Object PSObject
          $ExportVw | Add-Member -MemberType NoteProperty -name "Site URL" -value $Site.url
          $packageName = $package.PSChildName

          Write-Host `Removing packages $packageName from $Site.url` -ForegroundColor Yellow       
           #Find Name of app from installed package
           $RestMethodUrl = '/_api/web/lists/getbytitle(''Apps%20for%20SharePoint'')/items?$select=Title,LinkFilename'
           $apps = (Invoke-PnPSPRestMethod -Url $RestMethodUrl -Method Get -Connection $appCatConnection).Value
           $appTitle = ($apps | where-object {$_.LinkFilename -eq $packageName} | select Title).Title
        
           $currentPackage = Get-PnPApp -Identity $appTitle -scope Tenant
#check is app is installed on site
           $web = Get-PnPWeb -Includes AppTiles -Connection $siteConnection
           $app = $web.AppTiles  |  where-object {$_.Title -eq $currentPackage.Title }
        if($app)
        {
            Uninstall-PnPApp -Identity $currentPackage.Id -Connection $siteConnection
            $ExportVw | Add-Member -MemberType NoteProperty -name "Package Name" -value $packageName
        }
        $SiteAppUpdateCollection += $ExportVw     
      }     
  }

foreach($package in $packageFiles)
{
  $packageName = $package.PSChildName
   Write-Host ("Removing {0}..." -f $packageName) -ForegroundColor Yellow
   $RestMethodUrl = '/_api/web/lists/getbytitle(''Apps%20for%20SharePoint'')/items?$select=Title,LinkFilename'
   $apps = (Invoke-PnPSPRestMethod -Url $RestMethodUrl -Method Get -Connection $appCatConnection).Value
   $appTitle = ($apps | where-object {$_.LinkFilename -eq $packageName} | select Title).Title
 
}
#Export the result Array to CSV file
$SiteAppUpdateCollection | Export-CSV $OutPutView -Force -NoTypeInformation
 
Disconnect-PnPOnline
```

Let's break down the PowerShell script provided into its key components:

### Variables

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
$fileName = "\Log_Tenant_Removing_Solution" + $dateTime + ".csv"
$OutPutView = $directorypath + $fileName
```

The script retrieves all sites associated with a hub site. It navigates through each associated site, checks for existing packages, uninstalls them before removing the package from tenant app catalog.

## Conclusion:

The script discussed in this blog post demonstrates how PowerShell can be used to retract SPFx solutions from multiple sites within a SharePoint tenant. By understanding and adapting this script to specific organizational needs, administrators can effectively manage their SharePoint environments and ensure optimal performance and security.

 PnP PowerShell or alternative CLI for M365 provide a robust platform for implementing such automation, allowing administrators to streamline repetitive tasks and focus on higher-value activities.

