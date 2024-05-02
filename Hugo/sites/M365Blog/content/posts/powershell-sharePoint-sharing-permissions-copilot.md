---
title: "Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell to prevent oversharing"
date: 2024-04-16T06:51:10Z
tags: ["SharePoint","Sharing","Tenant","Sites","PowerShell","Copilot for M365","Governance"]
featured_image: '/posts/images/powershell-sharePoint-sharing-permissions-copilot/TenantSharingOptions_filefolderdomain.png'
draft: false
---

# Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell to prevent oversharing

Sharing lies at the heart of collaboration within SharePoint, facilitating seamless communication and teamwork. However, effective management of sharing settings is crucial to maintain data security and prevent unintended exposure. This is particularly important in light of tools like Copilot for M365, which help identify and mitigate instances of oversharing that can compromise data security. Let's explore how PowerShell can empower you to configure some sharing settings effectively at both the tenant and site levels.

An extract from [Announcing SharePoint advanced management innovations for the AI and Copilot era](https://techcommunity.microsoft.com/t5/sharepoint-premium-blog/announcing-sharepoint-advanced-management-innovations-for-the-ai/ba-p/4126366?WT.mc_id=5005104&ck_subscriber_id=2673998245)

```powershell
With Copilot and AI, security has become a concern. Not because Copilot allows people to access anything more than they could previously; it just allows them to find information they have access to faster. A term used sometimes in SharePoint is "Security by obscurity"; hide stuff and hope people don't find it. That doesn't work as well anymore with Copilot. It surfaces data more broadly and quickly. 
```

In this article, we'll delve into how PowerShell can empower SharePoint administrators to configure sharing settings proactively at both the tenant and site levels. By leveraging PowerShell scripts, you can streamline the process of enforcing security measures to safeguard sensitive information and mitigate the risk of oversharing.

Let's explore the key strategies and PowerShell commands you can implement to reinforce the security of your SharePoint environment, ensuring that collaboration remains productive while minimizing the potential for data breaches.

## Tenant Level Sharing Settings

### SharingCapability

This setting governs the sharing capability for the entire tenant, allowing you to control external user sharing and guest link sharing. 

The valid values are:

ExternalUserAndGuestSharing (default) - External user sharing (share by email) and guest link sharing are both enabled: Anyone - users can share files and folders using links that don't require sign-in

Disabled - External user sharing (share by email) and guest link sharing are both disabled - Only people in your organization - no external sharing allowed

ExternalUserSharingOnly - External user sharing (share by email) is enabled, but guest link sharing is disabled. New and existing guests - guests must sign in or provide a verification code

ExistingExternalUserSharingOnly - Existing guests only - only guests already in your organization's directory

![SharingCapability](../images/powershell-sharePoint-sharing-permissions-copilot/SharingCapability.png)

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

### ShowEveryoneExceptExternalUsersClaim

Enables or disables the display of the "Everyone except external users" claim in the People Picker. When users share an item with "Everyone except external users", it is accessible to all organisation members except external users in the tenant's Azure Active Directory.

The valid values are:
True(default) - The Everyone except external users is displayed in People Picker.
False - The Everyone except external users claim is not visible in People Picker.

### MySiteSharingCapability

Configures sharing capability for mysite (onedrive), offering options similar to those for the overall tenant.

The valid values are:

ExternalUserAndGuestSharing (default) - External user sharing (share by email) and guest link sharing are both enabled: Anyone - users can share files and folders using links that don't require sign-in

Disabled - External user sharing (share by email) and guest link sharing are both disabled - Only people in your organization - no external sharing allowed

ExternalUserSharingOnly - External user sharing (share by email) is enabled, but guest link sharing is disabled. New and existing guests - guests must sign in or provide a verification code

ExistingExternalUserSharingOnly - Existing guests only - only guests already in your organization's directory

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

Sets the expiration period for all anonymous links, enhancing security by limiting access duration.

### DefaultSharingLinkType

Allows administrators to choose what type of link appears is selected in the 'Get a link' sharing dialog box in OneDrive for Business and SharePoint Online.

Values can be set to None, Direct, Internal or AnonymousAccess

If set to AnonymousAccess you may get the following warning message to set `SharingCapability` to `ExternalUserAndGuestSharing`.

**WARNING**: `Anonymous access links aren’t enabled for your organization. You must first enable them by running the command "Set-PnPTenant -SharingCapability ExternalUserAndGuestSharing" before you can set the DefaultSharingLinkType parameter to AnonymousAccess. We will not set the value in this case.`


### PreventExternalUsersFromResharing

Allows or denies external users from resharing.

### ShowPeoplePickerSuggestionsForGuestUsers

Show or hide the guest users claim in the People Picker when set to true or false respectively.

### FileAnonymousLinkType

Configures anonymous link types for files. Allowed values are `View`, `Edit` or `None`

### FolderAnonymousLinkType

Configures anonymous link types for folders. Allowed values are `View`, `Edit` or `None`.

### NotifyOwnersWhenItemsReshared

When this parameter is set to true and another user re-shares a document from a user's OneDrive for Business, the OneDrive for Business owner is notified by e-mail.

### DefaultLinkPermission

Specifies the link permission. Valid values are `View`, `Edit` or `None`.

### RequireAcceptingAccountMatchInvitedAccount

Ensures that an external user can only accept an external sharing invitation with an account matching the invited email address. 
Valid values are 
- False (default) - When a document is shared with an external user, bob@contoso.com, it can be accepted by any user with access to the invitation link in the original e-mail.
- True - User must accept this invitation with bob@contoso.com.

If external sharing using domains is enabled you may be presented with the warning message

**WARNING**: `We automatically enabled RequireAcceptingAccountMatchInvitedAccount because you selected to limit external sharing using domains.`

### SharingAllowedDomainList

Specifies the list of allowed domains that users can share with.

### SharingBlockedDomainList

Specifies the list of blocked domains that users cannot share with.

### SharingDomainRestrictionMode

Available values are AllowList, BlockList, None


**WARNING**: `We automatically enabled RequireAcceptingAccountMatchInvitedAccount because you selected to limit external sharing using domains.`

**WARNING**: `You must set SharingDomainRestrictionMode to AllowList in order to have the list of domains you configured for SharingAllowedDomainList to take effect.`

**WARNING**: `You must set SharingDomainRestrictionMode to BlockList in order to have the list of domains you configured for SharingBlockedDomainList to take effect.`

### ExternalUserExpirationRequired

Enable guest access to a site or OneDrive to expire.

### ExternalUserExpireInDays

Specifies number of days for guest Access links to expire.

Refer to [External User Access Expiration in SharePoint Online and OneDrive for Business](https://www.sharepointdiary.com/2021/08/guest-user-access-expiration-in-sharepoint-online-onedrive.html#ixzz8XVgAFD56)

### -AllowEveryoneExceptExternalUsersClaimInPrivateSite
When this parameter is true, the "Everyone except external users" claim is available in the People Picker of a private site. Set it to false to disable this feature.

The valid values are:

True - The "Everyone except external users" claim is available in People Picker of a private site.
False (default) - The "Everyone except external users" claim is not available in People Picker of a private site.

### Sample script to amend tenant level sharing settings 

**SPO PowerShell**

```PowerShell
connect-SPOService -Url https://contoso-admin.sharepoint.com

Set-SPOTenant -SharingCapability ExternalUserAndGuestSharing `
            -ShowEveryoneClaim $false  `
            -ShowAllUsersClaim $false `
            -ShowEveryoneExceptExternalUsersClaim $true `
            -ProvisionSharedWithEveryoneFolder $false `
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
            -NotifyOwnersWhenItemsReshared $true `
            -DefaultLinkPermission "View" `
            -RequireAcceptingAccountMatchInvitedAccount $false `
            -ExternalUserExpirationRequired  $false `
            -AllowEveryoneExceptExternalUsersClaimInPrivateSite $true `
            -ExternalUserExpireInDays 60 
```

**PnP PowerShell**

```PowerShell
connect-pnponline -url https://contoso-admin.sharepoint.com -interactive
Set-PnPTenant -SharingCapability ExternalUserAndGuestSharing `
            -ShowEveryoneClaim $false  `
            -ShowAllUsersClaim $false `
            -ShowEveryoneExceptExternalUsersClaim $true `
            -ProvisionSharedWithEveryoneFolder $false `
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
            -NotifyOwnersWhenItemsReshared $true `
            -DefaultLinkPermission "View" `
            -RequireAcceptingAccountMatchInvitedAccount $false `
            -ExternalUserExpirationRequired  $false `
            -AllowEveryoneExceptExternalUsersClaimInPrivateSite $true `
            -ExternalUserExpireInDays 60 
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

This parameter prevents non-owners from sharing to new users to the site/library/file.

![Site Sharing Settings](../images/powershell-sharePoint-sharing-permissions-copilot/sitesharingsettings.png)

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

The default link type for the site collection

Possible values are 

None - Respect the organization default sharing link type
AnonymousAccess - Sets the default sharing link for this site to an Anonymous Access or Anyone link
Internal - Sets the default sharing link for this site to the "organization" link or company shareable link
Direct - Sets the default sharing link for this site to the "Specific people" link

### DefaultLinkPermission

The default link permission for the site collection

Possible values are

None - Respect the organization default link permission
View - Sets the default link permission for the site to "view" permissions
Edit - Sets the default link permission for the site to "edit" permissions

### AnonymousLinkExpirationInDays

Specifies all anonymous/anyone links that have been created (or will be created) will expire after the set number of days. Only applies if OverrideTenantAnonymousLinkExpirationPolicy is set to true.

### OverrideTenantExternalUserExpirationPolicy

Choose whether to override the external user expiration policy on this site

Possible values:

None: Respect the organization-level policy for external user expiration.
False: Respect the organization-level policy for external user expiration.
True: Override the organization-level policy for external user expiration (can be more or less restrictive).

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
            -AnonymousLinkExpirationInDays 60 
```

**PnP PowerShell**

```
connect-pnponline -url https://contoso.sharepoint.com/sites/SharingTest  -interactive
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

1. Sensitivity Labels
[How to protect sensitive information in SharePoint Online using Purview Sensitivity Labels](https://www.sharepointeurope.com/how-to-protect-sensitive-information-in-sharepoint-online-using-purview-sensitivity-labels/?utm_source=collab365&utm_medium=collab365today&utm_campaign=daily_digest)

2. Retention Policies
3. Data Loss Prevention

## Conclusion

Effective management of sharing settings is crucial for maintaining data security and compliance in SharePoint. PowerShell offers unparalleled flexibility and control in configuring these settings, ensuring that collaboration remains both seamless and secure.

## References

[ShowEveryoneExceptExternalUsersClaim](https://pnp.github.io/powershell/cmdlets/Set-PnPTenant.html#-showeveryoneexceptexternalusersclaim)

[SPOSharingSettings](https://microsoft365dsc.com/resources/sharepoint/SPOSharingSettings/)

[Use information barriers with SharePoint](https://learn.microsoft.com/en-gb/purview/information-barriers-sharepoint#information-barriers-modes-and-sharepoint-sites?wt.mc_id=MVP_308367)

[External User Access Expiration in SharePoint Online and OneDrive for Business](https://www.sharepointdiary.com/2021/08/guest-user-access-expiration-in-sharepoint-online-onedrive.html#ixzz8XVgAFD56)

[Enable auto-acceleration](https://learn.microsoft.com/en-us/sharepoint/enable-auto-acceleration??wt.mc_id=MVP_308367)

[Set-SPOTenant](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/set-spotenant?view=sharepoint-ps&wt.mc_id=MVP_308367)

[Set-SPOSite](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/set-sposite?view=sharepoint-ps&wt.mc_id=MVP_308367)

[Limit Sharing](https://learn.microsoft.com/en-us/microsoft-365/solutions/microsoft-365-limit-sharing?view=o365-worldwide#people-in-your-organization-sharing-links&wt.mc_id=MVP_308367)

[How to protect sensitive information in SharePoint Online using Purview Sensitivity Labels](https://www.sharepointeurope.com/how-to-protect-sensitive-information-in-sharepoint-online-using-purview-sensitivity-labels/?utm_source=collab365&utm_medium=collab365today&utm_campaign=daily_digest)
