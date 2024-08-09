---
title: "Retrieve SPFx Details from Tenant and Site Collection App Catalogs Using PowerShell"
date: 2024-08-08T07:17:21+01:00
tags: ["PowerShell", "SPFx","SharePoint Online","App Catalog","Inventory","REST API"]
featured_image: '/posts/images/powershell-get-spfx-details-tenant-sitecollection-appcatalog/example.png'
omit_header_text: true
draft: false
---

Have you ever needed to gather detailed information about SPFx solutions installed in your SharePoint environment, such as API permissions, for auditing, inventory, or compliance purposes? The PowerShell script below helps you retrieve these details from both the tenant-level and site collection app catalogs.

To execute this script, you must have Global Administrator or SharePoint Administrator roles.

**Classic View Interface**
![App Catalog](../images/powershell-get-spfx-details-tenant-sitecollection-appcatalog/AppCatalogList.png)

**Modern View Interface**
![Tenant App Catalog](../images/powershell-get-spfx-details-tenant-sitecollection-appcatalog/TenantAppCatalog.png)

Both interfaces don't display all details e.g. API Permissions making difficult to know after installations which permissions were granted to each SPFx app. 

The `Get-PnPApp` returns limited properties
![Tenant App Catalog](../images/powershell-get-spfx-details-tenant-sitecollection-appcatalog/GetPnPApp.png)

The alternative is to use the REST API endpoint `/_api/web/lists/getbytitle(''Apps%20for%20SharePoint'')/items?$select=Title` to retrieve specific properties of the apps within the app catalog which is a list under the hood.

## PowerShell script

```PowerShell
$AdminCenterURL= Read-Host -Prompt "Enter admin tenant collection URL";


$tenantAppCatalogUrl = Get-PnPTenantAppCatalogUrl
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "\InventorySPFx-" + $dateTime + ".csv"
$OutPutView = $directorypath + $fileName
 
cd $PSScriptRoot
 
Connect-PnPOnline $tenantAppCatalogUrl -Interactive
$appCatConnection  = Get-PnPConnection
 
Connect-PnPOnline $AdminCenterURL -Interactive
$adminConnection  = Get-PnPConnection
 
$appsDetails = @()
 
#Get associated sites with hub
$sites = Get-PnPTenantSite -Detailed -Connection $adminConnection  | Where-Object -Property Template -NotIn ("PWA#0","SRCHCEN#0", "REDIRECTSITE#0", "SPSMSITEHOST#0", "APPCATALOG#0", "POINTPUBLISHINGHUB#0", "POINTPUBLISHINGTOPIC#0","EDISC#0", "STS#-1") 
$RestMethodUrl = '/_api/web/lists/getbytitle(''Apps%20for%20SharePoint'')/items?$select=Title,LinkFilename,SkipFeatureDeployment,ContainsTeamsManifest,ContainsVivaManifest,SupportsTeamsTabs,WebApiPermissionScopesNote,ContainsTenantWideExtension,IsolatedDomain,PackageDefaultSkipFeatureDeployment,IsClientSideSolutionCurrentVersionDeployed,ExternalContentDomains,IsClientSideSolutionDeployed,IsClientSideSolution,AppPackageErrorMessage,IsValidAppPackage,SharePointAppCategory,AppDescription,AppShortDescription'

$apps = (Invoke-PnPSPRestMethod -Url $RestMethodUrl -Method Get -Connection $appCatConnection).Value
#export details of apps
$apps| foreach-object{
    $app = $_
    $app  | Add-Member -MemberType NoteProperty -name "Site Url" -value $tenantAppCatalogUrl
    $appsDetails += $app
}

$sites | select url | ForEach-Object {
  write-host "Processing Site:" $_.url -f Yellow
    $Site = Get-PnPTenantSite $_.url -Connection $adminConnection
  Connect-PnPOnline -Url $Site.url -Interactive
  $siteConnection  = Get-PnPConnection   

  try{
      if((Get-PnPSiteCollectionAppCatalog -CurrentSite)){
        $apps = (Invoke-PnPSPRestMethod -Url $RestMethodUrl -Method Get -Connection $siteConnection).Value
        $apps| foreach-object{
            $app = $_
            $app  | Add-Member -MemberType NoteProperty -name "Site Url" -value $Site.url
            $appsDetails += $app
        }
      }
     }
 catch{
  write-host -f Red $_.Exception.Message
 }
}
#Export the result Array to CSV file
$appsDetails | Export-CSV $OutPutView -Force -NoTypeInformation
```

This script automates the process of collecting and exporting SPFx solution details, which is essential for maintaining an up-to-date inventory and ensuring compliance across your SharePoint environment.

- **Title**: The name of the SPFx solution or app.

- **LinkFilename**: The filename of the app package (.sppkg file).

- **SkipFeatureDeployment**: Indicates whether the app skips feature deployment, meaning it doesn't require activation on individual sites.

- **ContainsTeamsManifest**: Indicates whether the app contains a Microsoft Teams manifest, making it available in Teams.

- **ContainsVivaManifest**: Indicates whether the app contains a Microsoft Viva manifest, allowing integration with Viva.

- **SupportsTeamsTabs**: Shows if the app supports being used as a Teams tab.

- **WebApiPermissionScopesNote**: Lists any API permissions granted to the app, often important for security and compliance reviews.

- **ContainsTenantWideExtension**: Indicates if the app contains a tenant-wide extension, which can affect all sites across the tenant.

- **IsolatedDomain**: Shows whether the app runs in an isolated domain for enhanced security, often used in sandboxed solutions.

- **PackageDefaultSkipFeatureDeployment**: Indicates the default setting of the app package regarding feature deployment skipping.

- **IsClientSideSolutionCurrentVersionDeployed**: Confirms if the currently deployed version of the client-side solution is up-to-date.

- **ExternalContentDomains**: Lists any external content domains the app might interact with, important for understanding potential external dependencies.

- **IsClientSideSolutionDeployed**: Indicates whether the client-side solution is actively deployed in the environment.

- **IsClientSideSolution**: Confirms whether the app is a client-side solution, typically referring to SPFx solutions.

- **AppPackageErrorMessage**: Contains any error messages related to the app package, useful for troubleshooting deployment issues.

- **IsValidAppPackage**: Indicates if the app package is valid and free from errors.

- **SharePointAppCategory**: Specifies the category assigned to the app within SharePoint, often used for organizing and searching for apps.

- **AppDescription**: A description of the app, providing a summary of its functionality and purpose.

- **AppShortDescription**: A shorter version of the app's description, typically used in user interfaces where space is limited.
