---
title: "Empowering Secure Collaboration: Configuring SharePoint Tenant and Site Settings with PowerShell to prevent oversharing"
date: 2024-05-03T06:51:10Z
tags: ["SharePoint","Sharing","Tenant","Sites","PowerShell","Copilot for M365","Governance","Sensitivity Label", "Data Loss Prevention","Retention Policies","RSS", "DLP", "Restricted SharePoint Search"]
featured_image: '/posts/images/powershell-sharePoint-sharing-permissions-copilot/TenantSharingOptions_filefolderdomain.png'
draft: false
---

# Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell to prevent Oversharing

Sharing lies at the heart of collaboration within SharePoint, facilitating seamless communication and teamwork. However, effective management of sharing settings is crucial to maintain data security and prevent unintended exposure. This is particularly important in light of tools like Copilot for M365.

An extract from [Announcing SharePoint advanced management innovations for the AI and Copilot era](https://techcommunity.microsoft.com/t5/sharepoint-premium-blog/announcing-sharepoint-advanced-management-innovations-for-the-ai/ba-p/4126366?WT.mc_id=5005104&ck_subscriber_id=2673998245)


"With Copilot and AI, security has become a concern. Not because Copilot allows people to access anything more than they could previously; it just allows them to find information they have access to faster. A term used sometimes in SharePoint is "Security by obscurity"; hide stuff and hope people don't find it. That doesn't work as well anymore with Copilot. It surfaces data more broadly and quickly."


In this article, we'll delve into how PowerShell can empower SharePoint administrators to configure sharing and other governance settings proactively at both the tenant and site levels to protect data.

![SharingSettings](../images/powershell-sharePoint-sharing-permissions-copilot/TenantSharingOptions_filefolderdomain.png)

There are links to additional resources related to Sensitivity Labels , Data Loss Prevention and Retention Policies which also help protecting your data.

## Tenant Level Sharing Settings

### SharingCapability

This setting governs the sharing capability for the entire tenant, allowing you to control external user sharing and guest link sharing.
For example if set to **Disabled**, a.k.a **Only people in your organisation**, The default behaviour in the share dialogue in SharePoint and in office documents is **Only people in your organisation** with option to choose to share with specific people to enable ad-hoc sharing.

The valid values are:

ExternalUserAndGuestSharing (default) - External user sharing (share by email) and guest link sharing are both enabled: Anyone - users can share files and folders using links that don't require sign-in

Disabled - External user sharing (share by email) and guest link sharing are both disabled - Only people in your organisation - no external sharing allowed

ExternalUserSharingOnly - External user sharing (share by email) is enabled, but guest link sharing is disabled. New and existing guests - guests must sign in or provide a verification code

ExistingExternalUserSharingOnly - Existing guests only - only guests already in your organisation's directory

![SharingCapability](../images/powershell-sharePoint-sharing-permissions-copilot/SharingCapability.png)

Microsoft recommends configuring the default sharing links to be most restrictive (for example, from company-wide to specific people) and making sure that any access request goes to the SharePoint site owners to help against oversharing.

### ShowAllUsersClaim

Determines whether the "All Users" claim group is displayed in the People Picker.

The valid values are: 
True(default) - "All users" is displayed in People Picker. When users share an item with "All Users (x)" it is accessible to all organization members in the tenant that used NTLM to authentication with SharePoint.
False - "All users" claim is not visible in People Picker.

The All Users group is equivalent to the Everyone claim, and shows as Everyone. 

![SharingOptions](../images/powershell-sharePoint-sharing-permissions-copilot/SharingOptions.png)

### ShowEveryoneClaim

Determines whether the "Everyone" claim group is displayed in the People Picker.

The valid values are: 
True(default) - The "Everyone" is displayed in People Picker. When users share an item with "Everyone", it is accessible to all authenticated users including any active external users
False - The "Everyone" claim is not visible in People Picker. It is the default for new tenants.

[Grant the Everyone claim to external users in Microsoft 365](https://learn.microsoft.com/en-us/microsoft-365/troubleshoot/access-management/grant-everyone-claim-to-external-users)

### ShowEveryoneExceptExternalUsersClaim

Enables or disables the display of the "Everyone except external users" claim in the People Picker within the tenant. When users share an item with "Everyone except external users", it is accessible to all organisation members except external users in the tenant's Azure Active Directory.

The valid values are:
True(default) - The Everyone except external users is displayed in People Picker.
False - The Everyone except external users claim is not visible in People Picker.

Microsoft recommends to review site-level sharing controls, and remove “Everyone Except External Users” from the people picker.
The alternative to consider is the **AllowEveryoneExceptExternalUsersClaimInPrivateSite** which applies the setting only to private sites.

### CoreSharingCapability 

Sets the level of sharing available for SharePoint sites excluding OneDrive sites.

The valid values are:

- ExternalUserAndGuestSharing (default) - External user sharing (share by email) and guest link sharing are both enabled: Anyone  - users can share files and folders using links that don't require sign-in

- Disabled - External user sharing (share by email) and guest link sharing are both disabled - Only people in your organisation no external sharing allowed

- ExternalUserSharingOnly - External user sharing (share by email) is enabled, but guest link sharing is disabled. New and existing guests - guests must sign in or provide a verification code

- ExistingExternalUserSharingOnly - Existing guests only - only guests already in your organisation's directory

### ProvisionSharedWithEveryoneFolder

Automatically creates a "Shared with Everyone" folder in users' OneDrive document libraries, enhancing collaboration.

### EnableGuestSignInAcceleration

Enables auto-acceleration on sites that have external sharing enabled. 

If the SignInAcceleration Domain is not set , the setting cannot be changed and you will be presented with message

`Set-SPOTenant : This setting cannot be changed until you set the SignInAcceleration Domain.`

Run the command to enable auto-acceleration on sites that aren't shared externally
```
Set-SPOTenant -SignInAccelerationDomain "contoso.com"
```

If set to $true, you will be prompted with the message

`Make sure that your federated sign-in supports guest users. If it doesn't, your guest users will no longer be able to sign in after you set EnableGuestSignlnAcceleration to $true.`

[Enable auto-acceleration](https://learn.microsoft.com/en-us/sharepoint/enable-auto-acceleration??wt.mc_id=MVP_308367)

### RequireAnonymousLinksExpireInDays

Sets the expiration period for all anonymous links, enhancing security by limiting access duration for external users and preventing accidental or malicous leaks.  

### DefaultSharingLinkType

Allows administrators to choose what type of link appears is selected in the 'Get a link' sharing dialog box in OneDrive for Business and SharePoint Online.

Values can be set to None, Direct, Internal or AnonymousAccess

If set to AnonymousAccess you may get the following warning message to set `SharingCapability` to `ExternalUserAndGuestSharing`.

**WARNING**: `Anonymous access links aren’t enabled for your organization. You must first enable them by running the command "Set-PnPTenant -SharingCapability ExternalUserAndGuestSharing" before you can set the DefaultSharingLinkType parameter to AnonymousAccess. We will not set the value in this case.`

### PreventExternalUsersFromResharing

Allows or denies external users from resharing files, folders, and sites that they do not own to prevent unauthorized disclosures of information.

### ShowPeoplePickerSuggestionsForGuestUsers

Show or hide the guest users claim in the People Picker when set to true or false respectively.

### FileAnonymousLinkType

Configures anonymous link types for files to only view or view with edit. Allowed values are `View`, `Edit` or `None`

### FolderAnonymousLinkType

Configures anonymous link types for folders to only view or view with edit. Allowed values are `View`, `Edit` or `None`.

### DefaultLinkPermission

Specifies the link permission. Valid values are `View`, `Edit` or `None`.

For example if set to `Edit`, the default behaviour in the sharing dialogue is set to Edit with option to the user to choose to apply “view only” and “block download” controls if they wish.

Check [Sharing files, folders, and list items](https://support.microsoft.com/en-gb/office/sharing-files-folders-and-list-items-74cab0bf-39c6-4112-a63f-88ee121722d0)

### RequireAcceptingAccountMatchInvitedAccount

Ensures guests must sign in using the same account to which sharing invitations are sent. This ensures that only the intended party is opening the shared document. 

Possible values are
  
- False (default) - When a document is shared with an external user, bob@contoso.com, it can be accepted by any user with access to the invitation link in the original e-mail.
- True - User must accept this invitation with bob@contoso.com.

If external sharing using domains is enabled you may be presented with the warning message

**WARNING**: `We automatically enabled RequireAcceptingAccountMatchInvitedAccount because you selected to limit external sharing using domains.`

### SharingAllowedDomainList : Limits external sharing by domain

Specifies the list of allowed domains that users can share with.

Refer to [restricted-domains-sharing](https://learn.microsoft.com/en-gb/sharepoint/restricted-domains-sharing?redirectSourcePath=%252farticle%252frestricted-domains-sharing-in-sharepoint-online-and-onedrive-for-business-5d7589cd-0997-4a00-a2ba-2320ec49c4e9) for more info.

### SharingBlockedDomainList

Specifies the list of blocked domains that users cannot share with.

### SharingDomainRestrictionMode

Available values are AllowList, BlockList, None


**WARNING**: `We automatically enabled RequireAcceptingAccountMatchInvitedAccount because you selected to limit external sharing using domains.`

**WARNING**: `You must set SharingDomainRestrictionMode to AllowList in order to have the list of domains you configured for SharingAllowedDomainList to take effect.`

**WARNING**: `You must set SharingDomainRestrictionMode to BlockList in order to have the list of domains you configured for SharingBlockedDomainList to take effect.`

### ExternalUserExpirationRequired

Enable guest access to a site or OneDrive to expire. Once the configured expiration data is reached, external users will lose access to the site or OneDrive.

### ExternalUserExpireInDays

Specifies number of days for guest Access links to expire. Guest access to a site or OneDrive will expire automatically after this many days.

Refer to [External User Access Expiration in SharePoint Online and OneDrive for Business](https://www.sharepointdiary.com/2021/08/guest-user-access-expiration-in-sharepoint-online-onedrive.html#ixzz8XVgAFD56?wt.mc_id=MVP_308367)

To set ExternalUserExpirationRequired and ExternalUserExpireInDays from the UI login to SharePoint Admin Center site at: https://tenant-admin.sharepoint.com > click on the "Policies" and then the "Sharing" link > expand the “More external sharing setting” and enable the checkbox “Guest access to a site or OneDrive will expire automatically after this many days” and then set the number of days. 

Read more: https://www.sharepointdiary.com/2021/08/guest-user-access-expiration-in-sharepoint-online-onedrive.html#ixzz8ZW8W9x2f

### AllowEveryoneExceptExternalUsersClaimInPrivateSite

Enables or disables the "Everyone except external users" claim in the People Picker of a private site. 
The valid values are:

True - The "Everyone except external users" claim is available in People Picker of a private site.
False (default) - The "Everyone except external users" claim is not available in People Picker of a private site.

### EnableAIPIntegration

This parameter enables support for sensitivity labels within SharePoint. Enabling this feature also results in SharePoint and OneDrive being able to process the contents of Office files and optionally, PDF documents that have been encrypted by using a sensitivity label. Until feature is enabled, these services can't process encrypted files, which means that coauthoring, eDiscovery, data loss prevention, search, and other collaborative features won't work for these files.For more information, see [Enable sensitivity labels for Office files in SharePoint and OneDrive](https://learn.microsoft.com/en-us/microsoft-365/compliance/sensitivity-labels-sharepoint-onedrive-files?wt.mc_id=MVP_308367).

### EnableSensitivityLabelforPDF

Enables support of sensitivity labels on PDF files

### MarkNewFilesSensitiveByDefault BlockExternalSharing

Prevents sharing until DLP (Data Loss P) has processed content.
 
When new files are added to SharePoint or OneDrive in M365, it may take some time for them to be crawled and indexed. It takes additional time for the Office Data Loss Prevention (DLP) policy to scan the content and apply rules to help protect sensitive information. If external sharing is turned on, sensitive content could be shared and accessed by guests before the Office DLP rule finishes processing. This setting ensures that documents are protected until DLP scans and marks them as safe to share. 

```powershell
Set-SPOTenant -MarkNewFilesSensitiveByDefault BlockExternalSharing 
```

Refer to [Mark new files as sensitive by default](https://docs.microsoft.com/en-us/sharepoint/sensitive-by-default?wt.mc_id=MVP_308367) for more details.

### AllowGuestUserShareToUsersNotInSiteCollection

Sets whether to allow guests to share to users not in the site.

The valid values are:

False (default) - Guest users will only be able to share to users that exist within the current site.
True - Guest users will be able to find user accounts in the directory by typing in the exact email address match.

### ConditionalAccessPolicy

Blocks or limits access to SharePoint and OneDrive content from un-managed devices.

Requires SharePoint Advanced Management feature to use since it is in GA. While in preview 

Possible values:

* AllowFullAccess: Allows full access from desktop apps, mobile apps, and the web.
* AllowLimitedAccess: Allows limited, web-only access.
* BlockAccess: Blocks Access.

Refer to [Control access from unmanaged devices](https://learn.microsoft.com/en-us/sharepoint/control-access-from-unmanaged-devices?wt.mc_id=MVP_308367)

### EnableAzureADB2BIntegration and SyncAadB2BManagementPolicy

Enables Azure AD B2B integration for SharePoint and OneDrive to establish a trusted connection between their SharePoint environment and external tenants allowing secure collaboration and access between different tenants to on SharePoint sites, documents, and other resources.

Cross-tenant collaboration is facilitated by this integration by streamlining the invitation and authentication process for external users ensuring they are properly authenticated and authorised to collaborate within the SharePoint environment. 

[SharePoint and OneDrive integration with Microsoft Entra B2B](https://docs.microsoft.com/en-us/sharepoint/sharepoint-azureb2b-integration?wt.mc_id=MVP_308367)

One-time passcode authentication, as described in [Email one-time passcode authentication](https://learn.microsoft.com/en-us/entra/external-id/one-time-passcode?wt.mc_id=MVP_308367) must be configured before enabling this integration.

### DisableDocumentLibraryDefaultLabeling

Turns off the feature that supports a default sensitivity label for SharePoint document libraries. If enabled, default sensitivity label can be set to a library to all new / modified files inside a document library.

See [Configure a default sensitivity label for a SharePoint document library](https://learn.microsoft.com/en-us/purview/sensitivity-labels-sharepoint-default-label) for more info.

### Sample script to amend tenant level sharing settings 

**SPO PowerShell**

```PowerShell
connect-SPOService -Url https://contoso-admin.sharepoint.com
##SharePoint specific settings
Set-SPOTenant -SharingCapability ExternalUserAndGuestSharing `
             -CoreSharingCapability ExternalUserAndGuestSharing `
            -ShowEveryoneClaim $false  `
            -ShowAllUsersClaim $false `
            -ShowEveryoneExceptExternalUsersClaim $true `
            -EnableGuestSignInAcceleration $false `
            -BccExternalSharingInvitations $false `
            -BccExternalSharingInvitationsList "" `
            -RequireAnonymousLinksExpireInDays 730 `
            -SharingAllowedDomainList "contoso.com" `
            -SharingBlockedDomainList "contoso.com" `
            -SharingDomainRestrictionMode AllowList `
            -DefaultSharingLinkType  AnonymousAccess `
            -PreventExternalUsersFromResharing  $false `
            -ShowPeoplePickerSuggestionsForGuestUsers $false `
            -FileAnonymousLinkType Edit  `
            -FolderAnonymousLinkType Edit `
            -DefaultLinkPermission "View" `
            -RequireAcceptingAccountMatchInvitedAccount $false `
            -ExternalUserExpirationRequired  $false `
            -AllowEveryoneExceptExternalUsersClaimInPrivateSite $true `
            -EnableAIPIntegration $true `
            -MarkNewFilesSensitiveByDefault BlockExternalSharing `
            -AllowGuestUserShareToUsersNotInSiteCollection $false `
            -ConditionalAccessPolicy AllowLimitedAccess `
            -EnableAzureADB2BIntegration $true `
            -DisableDocumentLibraryDefaultLabeling $false `
            -SyncAadB2BManagementPolicy $true `
            -EnableSensitivityLabelforPDF $true ` 
            -ExternalUserExpireInDays 60 

```

**PnP PowerShell**

```PowerShell
connect-pnponline -url https://contoso-admin.sharepoint.com -interactive
Set-PnPTenant -SharingCapability ExternalUserAndGuestSharing `
            -CoreSharingCapability ExternalUserAndGuestSharing `
            -ShowEveryoneClaim $false  `
            -ShowAllUsersClaim $false `
            -ShowEveryoneExceptExternalUsersClaim $true `
            -EnableGuestSignInAcceleration $false `
            -BccExternalSharingInvitations $false `
            -BccExternalSharingInvitationsList "" `
            -RequireAnonymousLinksExpireInDays 730 `
            -SharingAllowedDomainList "contoso.com" `
            -SharingBlockedDomainList "contoso.com" `
            -SharingDomainRestrictionMode AllowList `
            -DefaultSharingLinkType  AnonymousAccess `
            -PreventExternalUsersFromResharing  $false `
            -ShowPeoplePickerSuggestionsForGuestUsers $false `
            -FileAnonymousLinkType Edit  `
            -FolderAnonymousLinkType Edit `
            -DefaultLinkPermission "View" `
            -RequireAcceptingAccountMatchInvitedAccount $false `
            -ExternalUserExpirationRequired  $false `
            -AllowEveryoneExceptExternalUsersClaimInPrivateSite $true ` 
            -EnableAIPIntegration $true ` 
            -MarkNewFilesSensitiveByDefault BlockExternalSharing ` 
            -EnableAzureADB2BIntegration $true `
            -ConditionalAccessPolicy AllowLimitedAccess `
            -DisableDocumentLibraryDefaultLabeling $false `
            -ExternalUserExpireInDays 60 

            #-SyncAadB2BManagementPolicy $true ` #Only available through SPO PowerShell 
            #-EnableSensitivityLabelforPDF $true ` #Only available through SPO PowerShell
```
 
## Site Level Settings

Sharing settings can be set at the site level to provide more granular control.

### DisableCompanyWideSharingLinks

Disables People in your organization links. For more information, see People in your organization sharing links. Possible values

Disabled: If disabled, users sharing files in the site may need to use Specific people links, which can be shared with a maximum of 50 people.
NotDisabled: Users can choose option `People in <tenantName>` while sharing

![linkSettings](../images/powershell-sharePoint-sharing-permissions-copilot/linkSettings.png)

It can help to prevent accidental sharing with everyone authenticated within the tenant by disabling it.

### DefaultLinkToExistingAccess

When set to true, default sharing link will a People with Existing Access link (which does not modify permissions) maintaining permission integrity.
![linkSettings](../images/powershell-sharePoint-sharing-permissions-copilot/linktoexistingaccess.png)

When set to false (the default), the default sharing link type is controlled by the DefaultSharingLinkType parameter.

### DisableSharingForNonOwners

This parameter prevents non-owners from sharing to new users to the site/library/file. View [restrict members from sharing](https://learn.microsoft.com/en-us/microsoft-365/solutions/microsoft-365-limit-sharing#sharing-with-specific-people)

![Site Sharing Settings](../images/powershell-sharePoint-sharing-permissions-copilot/sitesharingsettings.png)

### Default user access request

_api/web/SetUseAccessRequestDefaultAndUpdate 


### ExternalUserExpirationInDays

Specifies all external user expiration which will expire after the set number of days. Only applies if OverrideTenantExternalUserExpirationPolicy is set to true.

### ShowPeoplePickerSuggestionsForGuestUsers

Enables search functionality for existing guest users at the site collection level when this parameter is set to $true.

The following warning message may display when enabled on site level when the setting is not enabled at the tenant level.

**WARNING**: `Warning: This setting won't take effect because showPeoplePickerSuggestionsForGuestUsers is currently disabled for your organization. Run the command Set-SPOTenant -showPeoplePickerSuggestionsForGuestUsers $true to enable this setting for your organization first.`

### SharingDomainRestrictionMode

Specifies the sharing mode for external domains.

Possible values are:

None - Do not restrict sharing by domain

AllowList - Sharing is allowed only with external users that have account on domains specified within -SharingAllowedDomainList

BlockList - Sharing is allowed with external users in all domains except in domains specified within -SharingBlockedDomainList

### SharingAllowedDomainList

Specifies a list of email domains that is allowed for sharing with the external collaborators. Use the space character as the delimiter for entering multiple values. For example, "contoso.com fabrikam.com".

### SharingBlockedDomainList

Specifies a list of email domains that is blocked or prohibited for sharing with the external collaborators. Use space character as the delimiter for entering multiple values. For example, "contoso.com fabrikam.com".

### DefaultSharingLinkType

Sets the default link type for the site collection

Possible values are 

None - Respect the organization default sharing link type
AnonymousAccess - Sets the default sharing link for this site to an Anonymous Access or Anyone link
Internal - Sets the default sharing link for this site to the "organization" link or company shareable link
Direct - Sets the default sharing link for this site to the "Specific people" link

### DefaultLinkPermission

Sets the default link permission for the site collection

Possible values are

None - Respect the organization default link permission
View - Sets the default link permission for the site to "view" permissions
Edit - Sets the default link permission for the site to "edit" permissions

### AnonymousLinkExpirationInDays

Specifies all anonymous/anyone links that have been created (or will be created) will expire after the set number of days. Only applies if OverrideTenantAnonymousLinkExpirationPolicy is set to true. This allows external users to access the shared content for a limited period and prevents unauthorized access if the link is shared accidentally or maliciously.

#Read more: https://www.sharepointdiary.com/2017/09/set-anonymous-link-expiration-in-sharepoint-online.html#ixzz8ZW6CchJS

### OverrideTenantExternalUserExpirationPolicy

Choose whether to override the external user expiration policy on this site

Possible values:

None: Respect the organization-level policy for external user expiration.
False: Respect the organization-level policy for external user expiration.
True: Override the organization-level policy for external user expiration (can be more or less restrictive).

### ReadOnlyForUnmanagedDevices

To set this parameter the **ConditionalAccessPolicy** needs to be set to **AllowLimitedAccess** and configures the site to be readonly on unmanaged devices meaning users can view the content on these devices, but they cannot make any changes. This helps to protect the data from potential security threats. 

Unmanaged devices include personal laptops, smartphones, or tablets that employees use to access company resources. These devices typically do not have the same security controls present on managed devices like firewalls, automatic updates, etc.. posing security risks.

**To set this parameter, you need to set the Set-SPOSite -ConditionalAccessPolicy to AllowLimitedAccess. <Would you like to set it now?>**

![Prompt to set read only for unmanaged devices](../images/powershell-sharePoint-sharing-permissions-copilot/ReadOnlyForUnmanagedDevices.png)

### PowerShell script to update site sharing settings

```PowerShell

Connect-SPOService -Url https://contoso-admin.sharepoint.com

Set-SPOSite -Identity https://contoso.sharepoint.com/sites/SharingTest ` -DisableSharingForNonOwners

Set-SPOSite -Identity https://contoso.sharepoint.com/sites/SharingTest `
            -ShowPeoplePickerSuggestionsForGuestUsers $false `
            -SharingCapability ExistingExternalUserSharingOnly `
            -ExternalUserExpirationInDays 60 `
            -SharingAllowedDomainList "contoso.com" `
            -SharingBlockedDomainList "contoso.com" `
            -SharingDomainRestrictionMode AllowList -OverrideTenantExternalUserExpirationPolicy $false `
            -DefaultSharingLinkType None `
            -DefaultLinkPermission None `
            -DefaultLinkToExistingAccess $true `
            -DisableCompanyWideSharingLinks Disabled `
            -AnonymousLinkExpirationInDays 60 `
           
```

**PnP PowerShell**

```PowerShell

Connect-pnponline -url https://contoso.sharepoint.com/sites/SharingTest  -interactive
Set-PnPSite  -DisableSharingForNonOwners

Set-PnPTenantSite -Identity https://contoso.sharepoint.com/sites/SharingTest `
            -ShowPeoplePickerSuggestionsForGuestUsers $false `
            -SharingCapability ExistingExternalUserSharingOnly `
            -ExternalUserExpirationInDays 60 `
            -SharingAllowedDomainList "contoso.com" `
            -SharingBlockedDomainList "contoso.com" `
            -SharingDomainRestrictionMode AllowList -OverrideTenantExternalUserExpirationPolicy $false `
            -DefaultSharingLinkType None `
            -DefaultLinkPermission None `
            -DefaultLinkToExistingAccess $true `
            -DisableCompanyWideSharingLinks Disabled `
            -AnonymousLinkExpirationInDays 60 
```

## Other settings to consider 

### SharePoint Restricted Search

Until the right controls are in place, up to 100 sites can be allowed to the list of sites to be part of the Organisation wide and Copilot experience using the SharePoint Restricted Search. 

View post [Introducing Restricted SharePoint Search to help you get started with Copilot for Microsoft 365](https://techcommunity.microsoft.com/t5/copilot-for-microsoft-365/introducing-restricted-sharepoint-search-to-help-you-get-started/ba-p/4071060) for more details.

### Sensitivity Labels

Sensitivity labels provides an additional layer of security to protect your data. Make sure the correct label is used for your data so that sensitive data get the correct restrictions , e.g. to prevent it from being shared externally via email or be shared with people outside a certain team.

Refer to [Use PowerShell for sensitivity labels and their policies](https://learn.microsoft.com/en-us/purview/create-sensitivity-labels?view=o365-worldwide#use-powershell-for-sensitivity-labels-and-their-policies&wt.mc_id=MVP_308367) to manage sensitivity labels.

[How to protect sensitive information in SharePoint Online using Purview Sensitivity Labels](https://www.sharepointeurope.com/how-to-protect-sensitive-information-in-sharepoint-online-using-purview-sensitivity-labels/?utm_source=collab365&utm_medium=collab365today&utm_campaign=daily_digest&wt.mc_id=MVP_308367)
 
[Sensitivity labels for Microsoft Teams for further information](https://docs.microsoft.com/en-us/microsoftteams/sensitivity-labels?wt.mc_id=MVP_308367). 

For more information on how sensitivity labels work across Microsoft Teams, SharePoint, and OneDrive, visit these links: 

[Enable sensitivity labels for Office files in SharePoint and OneDrive](https://docs.microsoft.com/en-us/microsoft-365/compliance/sensitivity-labels-sharepoint-onedrive-files?wt.mc_id=MVP_308367) 

[Apply a sensitivity label to content automatically](https://docs.microsoft.com/en-us/microsoft-365/compliance/apply-sensitivity-label-automatically?wt.mc_id=MVP_308367)

[Use sensitivity labels to protect content in Microsoft Teams, Microsoft 365 groups, and SharePoint sites](https://learn.microsoft.com/en-us/purview/sensitivity-labels-teams-groups-sites?wt.mc_id=MVP_308367)

The link [Configure a team with security isolation](https://docs.microsoft.com/en-us/microsoft-365/solutions/secure-teams-security-isolation?view=o365-worldwide&wt.mc_id=MVP_308367) describes how to use sensitivity labels to create an isolated team that can be used to provide a collaboration space for sensitive projects with protection that travels with the files that are stored in the team. 

The [How to enable sensitivity labels for containers and synchronize labels](https://learn.microsoft.com/en-us/microsoft-365/compliance/sensitivity-labels-teams-groups-sites?view=o365-worldwide#:~:text=External%20sharing%20and-,Conditional%20Access,-settings%2C%20now%20configure&wt.mc_id=MVP_308367) describes container sensitivity labels when combined with CA rules, provide context aware access to the container and the data it stores. For example the data that resides in that container can Only be accessed by your internal users connecting from a corporate (or managed) device or allow external guests connecting from a guest device (that is not one of your corporate devices joined to your tenant). 

[Enable sensitivity labels for Office files in SharePoint and OneDrive ](https://compliance.microsoft.com/informationprotection?viewid=autolabeling&wt.mc_id=MVP_308367)

[Apply a sensitivity label to content automatically](https://docs.microsoft.com/en-us/microsoft-365/compliance/apply-sensitivity-label-automatically?view=o365-worldwide#how-to-configure-auto-labeling-policies-for-sharepoint-onedrive-and-exchange&wt.mc_id=MVP_308367)

### Retention Policies

Retention policies can help with stale old data as well as to preserve documents from deletion. Retention policies can be configured to automatically remove data from a site or library or based on metadata/content type after a certain amount of time. These policies can help to ensure data is not deleted based on some metadata or where they are stored.

Refer to [Managing and applying Purview retention labels using code](https://www.blimped.nl/managing-and-applying-purview-retention-labels-using-code/) how to manage and apply purview rerention lebels using code

### Data Loss Prevention (DLP)

One example of DLP is to reducing sharing of Documents marked as Internal Only 
[Preventing and educating users from sharing sensitive documents externally](https://microsoft.github.io/ComplianceCxE/notes/mip-dlp/DLP-policy-externalshare?wt.mc_id=MVP_308367)

Refer to [How to Create and Manage DLP policies using PowerShell](https://www.jorgebernhardt.com/create-manage-dlp-policies/) for PowerShell scripts to manage DLP.

## Conclusion

Effective management of sharing settings is crucial for maintaining data security and compliance in SharePoint. PowerShell offers unparalleled flexibility and control in configuring these settings, ensuring that collaboration remains both seamless and secure.

## References

[Microsoft Purview data security and compliance protections for Microsoft Copilot](https://learn.microsoft.com/en-gb/purview/ai-microsoft-purview?wt.mc_id=MVP_308367)

[Microsoft Copilot for Microsoft 365 - best practices with SharePoint](https://learn.microsoft.com/en-us/SharePoint/sharepoint-copilot-best-practices?wt.mc_id=MVP_308367)

[Microsoft Purview data security and compliance protections for Microsoft Copilot](https://learn.microsoft.com/en-us/purview/ai-microsoft-purview?wt.mc_id=MVP_308367)

[ShowEveryoneExceptExternalUsersClaim](https://pnp.github.io/powershell/cmdlets/Set-PnPTenant.html#-showeveryoneexceptexternalusersclaim)

[SPOSharingSettings](https://microsoft365dsc.com/resources/sharepoint/SPOSharingSettings/)

[Use information barriers with SharePoint](https://learn.microsoft.com/en-gb/purview/information-barriers-sharepoint#information-barriers-modes-and-sharepoint-sites?wt.mc_id=MVP_308367)

[External User Access Expiration in SharePoint Online and OneDrive for Business](https://www.sharepointdiary.com/2021/08/guest-user-access-expiration-in-sharepoint-online-onedrive.html#ixzz8XVgAFD56)

[Enable auto-acceleration](https://learn.microsoft.com/en-us/sharepoint/enable-auto-acceleration?wt.mc_id=MVP_308367)

[Set-SPOTenant](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/set-spotenant?view=sharepoint-ps&wt.mc_id=MVP_308367)

[Set-SPOSite](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/set-sposite?view=sharepoint-ps&wt.mc_id=MVP_308367)

[Limit Sharing](https://learn.microsoft.com/en-us/microsoft-365/solutions/microsoft-365-limit-sharing?view=o365-worldwide#people-in-your-organization-sharing-links&wt.mc_id=MVP_308367)

[How to protect sensitive information in SharePoint Online using Purview Sensitivity Labels](https://www.sharepointeurope.com/how-to-protect-sensitive-information-in-sharepoint-online-using-purview-sensitivity-labels/?utm_source=collab365&utm_medium=collab365today&utm_campaign=daily_digest?wt.mc_id=MVP_308367)

[Sharing and permissions in the SharePoint modern experience](https://learn.microsoft.com/en-us/sharepoint/modern-experience-sharing-permissions?wt.mc_id=MVP_308367)
