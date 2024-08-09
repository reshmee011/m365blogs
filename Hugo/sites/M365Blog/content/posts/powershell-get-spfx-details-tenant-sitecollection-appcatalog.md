---
title: "Retrieve SPFx Details from Tenant and Site Collection App Catalogs Using PowerShell"
date: 2024-08-08T07:17:21+01:00
tags: ["PowerShell", "SPFx","SharePoint Online","App Catalog","Inventory"]
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