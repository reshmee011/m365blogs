---
title: "Retracting SPFx Solutions from Tenant Sites using PnP PowerShell"
date: 2024-03-19T08:25:29Z
tags: ["App Catalog","SPFx","PowerShell", "Uninstall"]
featured_image: '/posts/images/powershell-retract-spfx-from-sites-tenant/retract.png'
draft: true
---


# Retracting SPFx Solutions from Tenant Sites using PnP PowerShell

SharePoint Framework (SPFx) solutions are a powerful tool for extending and customizing SharePoint sites. However, managing these solutions across multiple sites in a SharePoint tenant can be a daunting task. Fortunately, PowerShell provides automation capabilities that can streamline these operations and ensure consistency across the tenant.

In this blog post, we'll explore a PowerShell script designed to retract SPFx solutions from sites within a SharePoint tenant. We'll discuss each section of the script, its purpose, and how it contributes to the overall goal of solution management.

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

Understanding the Script:
Let's break down the PowerShell script provided into its key components:

Variables Initialization: The script begins by initializing variables such as the admin center URL, tenant app catalog URL, hub site URL, and others. These variables hold essential information needed throughout the script execution.

Connecting to SharePoint Online: Using the Connect-PnPOnline cmdlet, the script establishes connections to both the tenant app catalog and the SharePoint admin center. These connections are necessary for accessing and managing sites and solutions within the SharePoint environment.

Retrieving Associated Sites: The script retrieves all sites associated with a hub site. This step ensures that solutions are retracted from all relevant sites within the tenant, including the hub site itself and its associated sites.

Iterating Through Sites: For each associated site, the script performs the following tasks:

Connects to the site using Connect-PnPOnline.
Retrieves a list of SPFx solution packages from a specified folder.
Checks if each package is installed on the site.
Uninstalls the package if it's installed.
Exporting Results: As the script iterates through sites and solutions, it collects information about the retraction process, such as the site URL and the names of removed packages. This information is then exported to a CSV file for record-keeping and analysis.

Cleanup and Disconnection: Finally, the script disconnects from the SharePoint Online sessions using Disconnect-PnPOnline, ensuring clean termination of connections.

## Conclusion:
Automating the management of SPFx solutions in a SharePoint tenant is essential for maintaining consistency, security, and efficiency. PowerShell provides a robust platform for implementing such automation, allowing administrators to streamline repetitive tasks and focus on higher-value activities.

The script discussed in this blog post demonstrates how PowerShell can be used to retract SPFx solutions from multiple sites within a SharePoint tenant. By understanding and adapting this script to specific organizational needs, administrators can effectively manage their SharePoint environments and ensure optimal performance and security.

As with any automation script, it's essential to test thoroughly in a non-production environment before deploying it in a production environment. Additionally, staying informed about updates and best practices in SharePoint development and administration will help maximize the effectiveness of PowerShell automation.

In future posts, we'll explore additional PowerShell scripts and techniques for managing SharePoint environments, further empowering administrators to unleash the full potential of their SharePoint deployments. Stay tuned for more insights and practical tips on SharePoint administration and development with PowerShell.