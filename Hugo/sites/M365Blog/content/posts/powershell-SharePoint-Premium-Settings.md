---
title: "Manage SharePoint Premium Settings using PowerShell to help overcome oversharing for Copilot for M365 rollout"
date: 2024-05-18T15:45:36+01:00
tags: ["SharePoint","Sharing","Tenant","Sites","PowerShell","Copilot for M365","Information Governance","Sensitivity Label","Conditional Access", "SharePoint Premium", "SharePoint Advanced Management","IG"]
featured_image: '/posts/images/powershell-SharePoint-Premium-Settings/SharePointPayAsYouGoBilling.png'
draft: false
---

# Manage SharePoint Premium Settings using PowerShell to help overcome oversharing for Copilot for M365 rollout

SharePoint Premium provides features to help with oversharing which will help towards Copilot for M365 rollout to prevent accidental data leaks.

Read [Microsoft Syntex - SharePoint Advanced Management overview](https://learn.microsoft.com/en-us/sharepoint/advanced-management?wt.mc_id=MVP_308367) for more info.

SharePoint Premium offers Pay as you go billing. I followed the post [Enable Pay-as-You-Go Licensing for Syntex aka SharePoint Premium.](https://www.divyaakula.com/sharepointpremium/2024/05/01/enabling-sharepoint-syntex-with-pay-as-you-go.html) to enable SharePoint Premium using my Visual Studio Azure subscription.

## Tenant level settings

### Enable Restricted Access Control

**SPO PowerShell**

```powershell
set-spotenant -EnableRestrictedAccessControl
```

**PnP PowerShell**

```powershell
set-pnptenant -EnableRestrictedAccessControl
```

Enables site-level access restriction for your organization. It might take up to one hour for command to take effect.

If appropriate license is missing, the following error message will appear.

**set-spotenant : This operation can't be performed as the tenant doesn't have the required license. Refer https://aka.ms/RACPolicyForSites to learn more**


### [Restrict OneDrive access by security group](https://learn.microsoft.com/en-us/sharepoint/limit-access?wt.mc_id=MVP_308367)

Access and sharing of OneDrive content can be restricted to users in specified Microsoft Entra ID security groups. Users outside the specified groups won't have access to their own OneDrive content once this policy is in effect.

Read more from [Restrict OneDrive access by security group](https://learn.microsoft.com/en-us/sharepoint/limit-access?wt.mc_id=MVP_308367).

The only way to update this setting is to use REST method to _api/SPOInternalUseOnly.Tenant

```powershell
connect-PnPONline -Url https://reshmeeauckloo-admin.sharepoint.com -Interactive

# Prepare the REST API endpoint URL
$apiUrl = "/_api/SPOInternalUseOnly.Tenant"

# Prepare the JSON payload
$payload = @{
    AllowSelectSGsInODBListInTenant = @('c:0t.c|tenant|a50cce6c-f979-4a1a-93dd-5b8487ada96b', 'c:0t.c|tenant|c6b80bfb-2d70-424b-a3e0-7cff04dc0387')
} | ConvertTo-Json

# Invoke the REST API method
Invoke-PnPSPRestMethod -Method Post -Url $apiUrl -ContentType "application/json;odata=verbose" -Content $payload
```

### DisableDocumentLibraryDefaultLabeling

Turns off the feature that supports a default sensitivity label for SharePoint document libraries. If enabled, default sensitivity label can be set to a library to all new / modified files inside a document library.

See [Configure a default sensitivity label for a SharePoint document library](https://learn.microsoft.com/en-us/purview/sensitivity-labels-sharepoint-default-label) for more info.

```powershell
set-spotenant -DisableDocumentLibraryDefaultLabeling $false #enables default sensitivity label for SharePoint document libraries
```

**PnP PowerShell**

```powershell
set-pnptenant -DisableDocumentLibraryDefaultLabeling $false #enables default sensitivity label for SharePoint document libraries
```

## Site level settings

### [Block download policy for SharePoint sites and OneDrive](https://learn.microsoft.com/en-us/sharepoint/block-download-from-sites?WT.mc_id=365AdminCSH_spo&?wt.mc_id=MVP_308367)

**BlockDownloadPolicy**

Block users from downloading, printing, or syncing files from specified SharePoint sites or OneDrive accounts to reduce the risk of data loss. User will have browser-only access and won't be able to access content through apps, including Microsoft Office desktop apps.

Configure certain sites to not allow
- Download
- Print
- Or sync of files

[block download policy](https://learn.microsoft.com/en-us/sharepoint/block-download-from-sites?wt.mc_id=MVP_308367)

Can only be configured at site level and at not at tenant wide.

This is a SharePoint Premium feature and since being in GA, you may need to consider licensing costs to use it.

**ExcludeBlockDownloadPolicySiteOwners**

#Exempt site owners from this policy and allow them to download any content
for the site
Blocks the download of files from SharePoint sites or OneDrive except the site owners.

**ExcludedBlockDownloadGroupIds**

Exempt users from the mentioned groups from this policy and allow them to download any content for the site

Provide comma separated group Ids.

**ExcludeBlockDownloadSharePointGroups**
Exempts users from the mentioned SharePoint groups from this policy and they can fully download any content for the site

Provide comma separated group Ids.

**ReadOnlyForBlockDownloadPolicy**

Marks the site as read-only in addition to preventing downloads.

If you don't have the appropriate license, you may get the error message

**Set-SPOSite : You do not have required licenses to perform this operation. Please read here for licensing related requirements : https://learn.microsoft.com/en-US/sharepoint/block-download-from-sites**

![Licence not activated](../images/powershell-SharePoint-Premium-Settings/BlockDownloadPolicyLicense.png)

Follow the post [Enable Pay-as-You-Go Licensing for Syntex aka SharePoint Premium.](https://www.divyaakula.com/sharepointpremium/2024/05/01/enabling-sharepoint-syntex-with-pay-as-you-go.html) to find details how to enable SharePoint Premium.

* SPO PowerShell

```powershell
## Block download policy 
Set-SPOSite -Identity https://contoso.sharepoint.com/sites/SharingTest `
            -BlockDownloadPolicy $true `
            -ExcludeBlockDownloadPolicySiteOwners $true `
            -ExcludedBlockDownloadGroupIds 21af775d-17b3-4637-94a4-2ba8625277cb,5cecaf4f-df68-414f-8029-a4e7e37df66e `
            -ExcludeBlockDownloadSharePointGroups 6c59372e-ebe5-4620-8a4c-588f2fc0c506,a50cce6c-f979-4a1a-93dd-5b8487ada96b `
            -ReadOnlyForBlockDownloadPolicy $true 
```

* PnP PowerShell

```powershell
## Block download policy 
Set-PnPTenantSite -Identity https://contoso.sharepoint.com/sites/SharingTest ` 
            -BlockDownloadPolicy $true `
            -ExcludeBlockDownloadPolicySiteOwners $true `
            -ExcludedBlockDownloadGroupIds 21af775d-17b3-4637-94a4-2ba8625277cb,5cecaf4f-df68-414f-8029-a4e7e37df66e `
            <#ExcludeBlockDownloadSharePointGroups and ReadOnlyForBlockDownloadPolicy can't be set using PnP PowerShell yet#>
            ##-ExcludeBlockDownloadSharePointGroups 6c59372e-ebe5-4620-8a4c-588f2fc0c506,a50cce6c-f979-4a1a-93dd-5b8487ada96b `
            ##-ReadOnlyForBlockDownloadPolicy $true 
```

### Restrict grouped connected and non grouped connected SharePoint sites

**Enable site access restriction**

Before using the property , enables tenant-level access restriction for your organization first setting the property EnableRestrictedAccessControl described above.

For group connected sites if RestrictedAccessControl is enabled, SharePoint site permissions are limited to users part of the Microsoft 365 Group. Site owners won't be able to share anything with members outside the Microsoft 365 Group.

For non grouped sites, site access can be restricted to members of up to 10 specified Entra ID security groups or Microsoft 365 Groups.

* SPO PowerShell

```PowerShell
Set-SPOSite -Identity <siteurl> -RestrictedAccessControl $true
```

* PnP PowerShell

```PowerShell
Set-PnPTenantSite -Identity <siteurl> -RestrictedAccessControl $true
```

**Reset site access restriction**

* SPO PowerShell

```PowerShell
Set-SPOSite -Identity <siteurl> -ClearRestrictedAccessControl
```

* PnP PowerShell

```PowerShell
Set-PnPTenantSite -Identity <siteurl> -ClearRestrictedAccessControl 
```

The following message will be displayed on resetting the site access restriction.

**Restricted access control has been disabled on the site https://reshmeeauckloo.sharepoint.com/sites/SharingTest. It is recommended to review the site permissions and remove users who no longer need access to the site.**

### Restrict non grouped connected SharePoint sites and OneDrive access with Microsoft 365 groups and Entra security groups more settings

Read more [Restrict SharePoint site access with Microsoft 365 groups and Entra security groups](https://learn.microsoft.com/en-us/sharepoint/restricted-access-control)

Read more from [Restrict access to a user's OneDrive content to people in a group](https://learn.microsoft.com/en-us/sharepoint/onedrive-site-access-restriction?wt.mc_id=MVP_308367)

Access and sharing of individual OneDrive's and SharePoint site content can be restricted to users in specified Microsoft Entra ID security groups using OneDrive access restriction policy. Users not in the specified group won't be able to access the content, even if they had prior permissions or shared link.

Using the above cmdlets on a group connected site will throw the following error message

**The <site URL> is not updated. This site is a Microsoft 365 group connected site, refer https://aka.ms/RACPolicyForSites to learn more.**

If license is missing and restricted access is not enabled , the error message and warning message will appear

```PowerShell
WARNING: To apply restricted access control, enable the policy on the site https://reshmeeauckloo.sharepoint.com/sites/Contosoinc-hr-Finance. Refer https://aka.ms/RACPolicyForSites to learn more.  
Set-SPOSite : You need a SharePoint Advanced Management license to perform this action.
```

**Add group**

* SPO PowerShell

```PowerShell
Set-SPOSite -Identity <siteurl> -AddRestrictedAccessControlGroups 21af775d-17b3-4637-94a4-2ba8625277cb,5cecaf4f-df68-414f-8029-a4e7e37df66e 
```

* PnP PowerShell*

```PowerShell
Set-PnPTenantSite -Identity <siteurl> -AddRestrictedAccessControlGroups 21af775d-17b3-4637-94a4-2ba8625277cb,5cecaf4f-df68-414f-8029-a4e7e37df66e
```

**Edit group**	

* SPO PowerShell

```PowerShell
Set-SPOSite -Identity <siteurl> -RestrictedAccessControlGroups 21af775d-17b3-4637-94a4-2ba8625277cb,5cecaf4f-df68-414f-8029-a4e7e37df66e
```

* PnP PowerShell*

```PowerShell
Set-PnPTenantSite -Identity <siteurl> -RestrictedAccessControlGroups 21af775d-17b3-4637-94a4-2ba8625277cb,5cecaf4f-df68-414f-8029-a4e7e37df66e
```

**View group**	

* SPO PowerShell

```PowerShell
Get-SPOSite -Identity <siteurl> | Select RestrictedAccessControl, RestrictedAccessControlGroups
```

* PnP PowerShell

```PowerShell
Get-PnPTenantSite -Identity <siteurl> | Select RestrictedAccessControl, RestrictedAccessControlGroups
```

**Remove group**

* SPO PowerShell

```PowerShell
Set-SPOSite -Identity <siteurl> -RemoveRestrictedAccessControlGroups 21af775d-17b3-4637-94a4-2ba8625277cb,5cecaf4f-df68-414f-8029-a4e7e37df66e
```

* PnP PowerShell

```PowerShell
Get-PnPTenantSite -Identity <siteurl> -RemoveRestrictedAccessControlGroups 21af775d-17b3-4637-94a4-2ba8625277cb,5cecaf4f-df68-414f-8029-a4e7e37df66e
```


### Conditional Access Policy

Blocks or limits access to SharePoint and OneDrive content from un-managed devices.

Requires SharePoint Advanced Management feature to use since it is in GA. 

Possible values:

* AllowFullAccess: Allows full access from desktop apps, mobile apps, and the web.
* AllowLimitedAccess: Allows limited, web-only access.
* BlockAccess: Blocks Access.

Refer to [Control access from unmanaged devices](https://learn.microsoft.com/en-us/sharepoint/control-access-from-unmanaged-devices?wt.mc_id=MVP_308367)

```PowerShell
Set-SPOSite -Identity https://contoso.sharepoint.com/sites/SharingTest ` 
            -ConditionalAccessPolicy AllowLimitedAccess  #PnP PowerShell does not have that option yet
```

### Default Sensitivity label for sites and libraries for security

To use the feature, the tenant setting EnableAIPIntegration needs to be enabled.

```PowerShell
Set-SPOTenant -EnableAIPIntegration $true
```

see [Enable sensitivity labels for Office files in SharePoint and OneDrive](https://learn.microsoft.com/en-us/microsoft-365/compliance/sensitivity-labels-sharepoint-onedrive-files?wt.mc_id=MVP_308367) for more info.

#### Set Sensitivity label can be set at the site level

* SPO PowerShell

```powershell
set-sposite -identity <site url>  -SensitivityLabel b160962f-aee6-431d-891c-8f1e31ccdba3
```

* PnP PowerShell

```powershell
Get-PnPTenantSite -Identity <siteurl>  -SensitivityLabel b160962f-aee6-431d-891c-8f1e31ccdba3
```

#### Set Sensitivity label can be set at the library level

Default sensitivity label can be set to a library to all new / modified files inside a document library. This ensures that documents are protected even if a user forgets to specify the label. It can apply a higher priority label if a user has a default label applied.

![Library sensitivity label](../images/powershell-SharePoint-Premium-Settings/LibraryDefaultSensitivityLabel.png)

* PnP PowerShell

```powershell
Set-PnPList -Identity "Demo List" -DefaultSensitivityLabelForLibrary "Business"
```

See [Configure a default sensitivity label for a SharePoint document library](https://learn.microsoft.com/en-us/purview/sensitivity-labels-sharepoint-default-label) for more info