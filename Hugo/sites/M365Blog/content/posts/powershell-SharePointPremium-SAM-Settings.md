---
title: "Manage SharePoint Premium - SharePoint Advanced Management Settings Using PowerShell to protect data in Copilot for M365 Rollout"
date: 2024-05-18T15:45:36+01:00
tags: ["SharePoint Premium","Syntex","oversharing","Tenant","Sites","PowerShell","Copilot for M365","Information Governance","Sensitivity Label","Conditional Access", "SharePoint Premium", "SharePoint Advanced Management","IG"]
featured_image: '/posts/images/powershell-SharePointPremium-SAM-Settings/SharePointAdvancedManagementSettings.png'
draft: false
---

# Manage SharePoint Premium - SharePoint Advanced Management Settings Using PowerShell to protect data in Copilot for M365 Rollout

SharePoint Premium - SharePoint Advanced Management offers features to help prevent data oversharing and accidental leaks, which is crucial for a successful rollout of Copilot for M365. This guide will show you how to manage these settings using PowerShell.

For an overview, read the [Microsoft SharePoint Premium - SharePoint Advanced Management overview](https://learn.microsoft.com/en-us/sharepoint/advanced-management?wt.mc_id=MVP_308367).

This post covers how to manage **SharePoint Premium - SharePoint Advanced Management** settings at both the tenant and site level using PowerShell.

## Enabling SharePoint Advanced Management - SAM

SharePoint Premium provides pay-as-you-go billing. You can start a trial by navigating to:
Microsoft 365 Admin Center > Billing > Purchase Services > Search for SharePoint Advanced Management > Scroll Down > Click on Details > Click on Start Free Trial.

![Enable Trial](../images/powershell-SharePointPremium-SAM-Settings/SharePointAdvancedManagementFeatures.png)

Once enabled SAM (SharePoint Advanced Management) settings like **Restrict SharePoint site access with Microsoft 365 groups and Entra security groups**,  **Restrict OneDrive content access**,**Restrict OneDrive service access**, **Conditional access policy for SharePoint sites and OneDrive**, **Data access governance reports for SharePoint sites** and **Restricted Content Discoverability (RCD)**  can be managed using PowerShell.

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

This setting enables site-level access restriction for your organization. It may take up to an hour for the command to take effect. If the appropriate license is missing, you will receive an error message.

**set-spotenant : This operation can't be performed as the tenant doesn't have the required license. Refer https://aka.ms/RACPolicyForSites to learn more**

### [Restrict OneDrive access by security group](https://learn.microsoft.com/en-us/sharepoint/limit-access?wt.mc_id=MVP_308367)

Restrict access and sharing of OneDrive content to users in specified Microsoft Entra ID security groups. Use the REST method to update this setting through PowerShell using REST method to _api/SPOInternalUseOnly.Tenant

```powershell
connect-PnPOnline -Url https://reshmeeauckloo-admin.sharepoint.com -Interactive

# Prepare the REST API endpoint URL
$apiUrl = "/_api/SPOInternalUseOnly.Tenant"

# Prepare the JSON payload
$payload = @{
    AllowSelectSGsInODBListInTenant = @('c:0t.c|tenant|a50cce6c-f979-4a1a-93dd-5b8487ada96b', 'c:0t.c|tenant|c6b80bfb-2d70-424b-a3e0-7cff04dc0387')
}

# Invoke the REST API method
Invoke-PnPSPRestMethod -Method Patch -Url $apiUrl -ContentType "application/json;odata.metadata=minimal" -Content $payload
```

Read [Restrict OneDrive access by security group](https://learn.microsoft.com/en-us/sharepoint/limit-access?wt.mc_id=MVP_308367) for more info.

![OneDrive Access Restriction](../images/powershell-SharePointPremium-SAM-Settings/OneDriveAccessRestriction.png)

### DisableDocumentLibraryDefaultLabeling

This setting turns off the default sensitivity label for SharePoint document libraries. If enabled, default sensitivity label can be set to a library to all new / modified files inside a document library.

For more information, see  [Configure a default sensitivity label for a SharePoint document library](https://learn.microsoft.com/en-us/purview/sensitivity-labels-sharepoint-default-label?wt.mc_id=MVP_308367).

**SPO PowerShell**

```powershell
set-spotenant -DisableDocumentLibraryDefaultLabeling $false #enables default sensitivity label for SharePoint document libraries
```

**PnP PowerShell**

```powershell
set-pnptenant -DisableDocumentLibraryDefaultLabeling $false #enables default sensitivity label for SharePoint document libraries
```

## Site level settings

### Restricted Content Discoverability (RCD)

Restricted Content Discoverability (RCD) is another SharePoint Advanced Management (SAM) feature which can be an alternative to [Restricted SharePoint Search - RSS](https://reshmeeauckloo.com/posts/sharepoint-restricted-sharepoint-search/) to use to restrict content discoverability via Copilot and Org-wide search if oversharing is a concern.

At the time of writing this blog post , this setting is still in private preview. Again use it judiciously as less data will be available to Copilot for responses, hence less intelligent response and affects findability/search.

**SPO PowerShell**

```PowerShell
Set-SPOSite -Identity <site-url> -RestrictContentOrgWideSearch $true
```

**PnP PowerShell**

This setting is dependent on the PR getting merged [4024](https://github.com/pnp/powershell/pull/4024) being merged into **PnP PowerShell**.

```PowerShell
Set-PnPSite -Identity <site-url> -RestrictContentOrgWideSearch $true
```

### [Block download policy for SharePoint sites and OneDrive](https://learn.microsoft.com/en-us/sharepoint/block-download-from-sites?WT.mc_id=365AdminCSH_spo&wt.mc_id=MVP_308367)

**BlockDownloadPolicy**

Block users from downloading, printing, or syncing files from specified SharePoint sites or OneDrive accounts to reduce the risk of data loss. User will have browser-only access and won't be able to access content through apps, including Microsoft Office desktop apps.

For more information, see [block download policy](https://learn.microsoft.com/en-us/sharepoint/block-download-from-sites?wt.mc_id=MVP_308367)

This can only be configured at site level and at not at tenant wide.

**ExcludeBlockDownloadPolicySiteOwners**

Exempt site owners from this policy and allow them to download any content for the site

**ExcludedBlockDownloadGroupIds**

Exempt users from the mentioned groups from this policy and allow them to download any content for the site

**ExcludeBlockDownloadSharePointGroups**

Exempts users from the mentioned SharePoint groups from this policy and they can fully download any content for the site

**ReadOnlyForBlockDownloadPolicy**

Marks the site as read-only in addition to preventing downloads.

If you don't have the appropriate license, you may get the error message

**Set-SPOSite : You do not have required licenses to perform this operation. Please read here for licensing related requirements : https://learn.microsoft.com/en-US/sharepoint/block-download-from-sites**

![Licence not activated](../images/powershell-SharePointPremium-SAM-Settings/BlockDownloadPolicyLicense.png)

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

For group-connected sites, this setting limits permissions to users in the Microsoft 365 Group. For non-grouped sites, access can be restricted to up to 10 specified Entra ID security groups or Microsoft 365 Groups.

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

Read more [Restrict access to a user's OneDrive content to people in a group](https://learn.microsoft.com/en-us/sharepoint/onedrive-site-access-restriction?wt.mc_id=MVP_308367)

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

This policy blocks or limits access to SharePoint and OneDrive content from unmanaged devices.

Possible values:

**AllowFullAccess**: Allows full access from desktop apps, mobile apps, and the web.
**AllowLimitedAccess**: Allows limited, web-only access.
**BlockAccess**: Blocks Access.

Refer to [Control access from unmanaged devices](https://learn.microsoft.com/en-us/sharepoint/control-access-from-unmanaged-devices?wt.mc_id=MVP_308367).

```PowerShell
Set-SPOSite -Identity https://contoso.sharepoint.com/sites/SharingTest ` 
            -ConditionalAccessPolicy AllowLimitedAccess  #PnP PowerShell does not have that option yet
```

### Default Sensitivity label for sites and libraries for security

To use this feature, enable the tenant setting EnableAIPIntegration.

```PowerShell
Set-SPOTenant -EnableAIPIntegration $true
```

For more information, see [Enable sensitivity labels for Office files in SharePoint and OneDrive](https://learn.microsoft.com/en-us/microsoft-365/compliance/sensitivity-labels-sharepoint-onedrive-files?wt.mc_id=MVP_308367) for more info.

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

![Library sensitivity label](../images/powershell-SharePointPremium-SAM-Settings/LibraryDefaultSensitivityLabel.png)

* PnP PowerShell

```powershell
Set-PnPList -Identity "Demo List" -DefaultSensitivityLabelForLibrary "Business"
```

For more information, see [Configure a default sensitivity label for a SharePoint document library](https://learn.microsoft.com/en-us/purview/sensitivity-labels-sharepoint-default-label?wt.mc_id=MVP_308367)

## Other SharePoint Avanced Management features

![SharePoint Avanced Management features](../images/powershell-SharePointPremium-SAM-Settings/SharePointAdvancedManagementSettings.png)

* Change history : Find who made particular site or organization setting changes and when

For more information, see [Create change history reports](https://learn.microsoft.com/en-us/sharepoint/change-history-report?wt.mc_id=MVP_308367)

* Data access governance reports : Discover potential oversharing and keep track of sites that have sensitive files

For more information, see [Data access governance reports for SharePoint sites](https://learn.microsoft.com/en-us/sharepoint/data-access-governance-reports?wt.mc_id=MVP_308367)

![Data Governance](../images/powershell-SharePointPremium-SAM-Settings/DataGovernance.png)

* Recent actions : Review recent site changes you made

For more information, see [Review your recent changes to SharePoint site properties](https://learn.microsoft.com/en-us/sharepoint/recent-actions-panel?WT.mc_id=365AdminCSH_spo?wt.mc_id=MVP_308367)

* Site lifecycle management : Automate tasks across the life cycle of your sites

For more information, see [Manage site lifecycle policies](https://learn.microsoft.com/en-us/sharepoint/site-lifecycle-management?wt.mc_id=MVP_308367)
 
## Additional AI driven SharePoint Premium Features

I followed the post [Enable Pay-as-You-Go Licensing for Syntex aka SharePoint Premium.](https://www.divyaakula.com/sharepointpremium/2024/05/01/enabling-sharepoint-syntex-with-pay-as-you-go.html) to enable SharePoint Premium using my Visual Studio Azure subscription.

### Content Processing

* Autofill columns
* Taxonomy tagging
* Content query
* Translation
* Annotations
* Automated classification and security

![Syntex Settings](../images/powershell-SharePointPremium-SAM-Settings/SyntexSettings.png)
