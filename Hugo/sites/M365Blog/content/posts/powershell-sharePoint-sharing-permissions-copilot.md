---
title: "Harness sharing capabilities within your tenant and SharePoint site using PowerShell"
date: 2024-03-25T06:51:10Z
draft: true
---

# Harness sharing capabilities within your tenant and SharePoint site


## Tenant Level Settings

**SPO PowerShell**

``` 
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
            -ExternalUserExpireInDays 60 
```

**PnP PowerShell**

```
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
            -ExternalUserExpireInDays 60 
```

### SharingCapability

Configures sharing capability for the tenant.
The valid values are:

ExternalUserAndGuestSharing (default) - External user sharing (share by email) and guest link sharing are both enabled.
Disabled - External user sharing (share by email) and guest link sharing are both disabled.
ExternalUserSharingOnly - External user sharing (share by email) is enabled, but guest link sharing is disabled.
ExistingExternalUserSharingOnly - Only guests already in your organization's directory.

### ShowAllUsersClaim

Enables the administrator to hide the All Users claim groups in People Picker.

### ShowEveryoneClaim

Enables the administrator to hide the Everyone claim in the People Picker.

### ShowEveryoneExceptExternalUsersClaim

Enables the administrator to hide the "Everyone except external users" claim in the People Picker. When users share an item with "Everyone except external users", it is accessible to all organization members in the tenant's Azure Active Directory, but not to any users who have previously accepted invitations.

The valid values are: True(default) - The Everyone except external users is displayed in People Picker. False - The Everyone except external users claim is not visible in People Picker.

There are other tenant wide level settings but I think it would have been great to be able to set those at the library or site level and have site power users have greater control over them 


### MySiteSharingCapability

 Configures sharing capability for mysite (onedrive)
 ExistingExternalUserSharingOnly, ExternalUserAndGuestSharing, Disabled, ExternalUserSharingOnly

### ProvisionSharedWithEveryoneFolder

Creates a Shared with Everyone folder in every user's new OneDrive for Business document library.

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

Specifies all anonymous links that have been created (or will be created) will expire after the set number of days.

### DefaultSharingLinkType

Lets administrators choose what type of link appears is selected in the 'Get a link' sharing dialog box in OneDrive for Business and SharePoint Online

Default values are None, Direct, Internal, AnonymousAccess

### PreventExternalUsersFromResharing

Allow or deny external users re-sharing

### ShowPeoplePickerSuggestionsForGuestUsers

Enables the administrator to hide the guest users claim in the People Picker

### FileAnonymousLinkType

Configures anonymous link types for files. View, Edit, None

### FolderAnonymousLinkType

Configures anonymous link types for folders. View, Edit, None

### NotifyOwnersWhenItemsReshared

When this parameter is set to $true and another user re-shares a document from a userâs OneDrive for Business, the OneDrive for Business owner is notified by e-mail.

### DefaultLinkPermission

Specifies the link permission on the tenant level. Valid values to set are View and Edit. A value of None will be set to Edit as its the default value. None, View, Edit

### RequireAcceptingAccountMatchInvitedAccount

Ensures that an external user can only accept an external sharing invitation with an account matching the invited email address.Administrators who desire increased control over external collaborators should consider enabling this feature. False (default) - When a document is shared with an external user, bob@contoso.com, it can be accepted by any user with access to the invitation link in the original e-mail.True - User must accept this invitation with bob@contoso.com.

WARNING: We automatically enabled RequireAcceptingAccountMatchInvitedAccount because you selected to limit external sharing using domains.

### SharingAllowedDomainList

Specifies the list of allowed domains that users can share with.

### SharingBlockedDomainList

Specifies the list of blocked domains that users cannot share with.

### SharingDomainRestrictionMode

Available values are AllowList, BlockList, None

WARNING: We automatically enabled RequireAcceptingAccountMatchInvitedAccount because you selected to limit external sharing using domains.
WARNING: You must set SharingDomainRestrictionMode to AllowList in order to have the list of domains you configured for SharingAllowedDomainList to take effect.
WARNING: You must set SharingDomainRestrictionMode to BlockList in order to have the list of domains you configured for SharingBlockedDomainList to take effect

### ExternalUserExpirationRequired

Enable guest access to a site or OneDrive to expire.

### ExternalUserExpireInDays

Specifies number of days for guest Access links to expire.

Refer to [External User Access Expiration in SharePoint Online and OneDrive for Business](https://www.sharepointdiary.com/2021/08/guest-user-access-expiration-in-sharepoint-online-onedrive.html#ixzz8XVgAFD56)
 
## Site Level Settings

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

### DisableCompanyWideSharingLinks
Disabled , Enabled
Disables People in your organization links. For more information, see People in your organization sharing links. Possible values

Disabled
NotDisabled

### DefaultLinkToExistingAccess 
boolean

### DisableSharingForNonOwners
boolean

### ExternalUserExpirationInDays

### ShowPeoplePickerSuggestionsForGuestUsers
boolean

WARNING: Warning: This setting won't take effect because showPeoplePickerSuggestionsForGuestUsers is currently disabled for your organization. Run the command Set-SPOTenant -showPeoplePickerSuggestionsForGuestUsers $true to enabl
e this setting for your organization first.

### SharingDomainRestrictionMode

### SharingAllowedDomainList

### SharingBlockedDomainList

### DefaultSharingLinkType
None

### DefaultLinkPermission
None

### DefaultLinkToExistingAccess
Boolean

### AnonymousLinkExpirationInDays

### OverrideTenantExternalUserExpirationPolicy


### LoopDefaultSharingLinkScope

### LoopDefaultSharingLinkRole

### DefaultShareLinkScope

### DefaultShareLinkRole

### RestrictedAccessControl

### RestrictedAccessControlGroups 

Set-PnPSite -DisableCompanyWideSharingLinks Disabled -DefaultLinkToExistingAccess $true -DisableSharingForNonOwners | Out-Null  


This is an important topic to understand.

## At site level, disable default sharing 

## Restricts sharing to power users

## sharing links
SHARING LINKS from power automate

. Create a sharing link for a file or folder
· Specify:
. The Item ID
· Link Type (View and edit, View only)
· Link Scope (Anonymous, People in
your org)
· Link Expiration date (optional)
· Link is then available in Dynamic
Content to be able to send via email or
Teams
. Make sure that sharing is enabled from
the admin level in Microsoft 365.

## References

[ShowEveryoneExceptExternalUsersClaim](https://pnp.github.io/powershell/cmdlets/Set-PnPTenant.html#-showeveryoneexceptexternalusersclaim)

[SPOSharingSettings](https://microsoft365dsc.com/resources/sharepoint/SPOSharingSettings/)

[Use information barriers with SharePoint](https://learn.microsoft.com/en-gb/purview/information-barriers-sharepoint#information-barriers-modes-and-sharepoint-sites?wt.mc_id=MVP_308367)

[External User Access Expiration in SharePoint Online and OneDrive for Business](https://www.sharepointdiary.com/2021/08/guest-user-access-expiration-in-sharepoint-online-onedrive.html#ixzz8XVgAFD56)

[Enable auto-acceleration](https://learn.microsoft.com/en-us/sharepoint/enable-auto-acceleration??wt.mc_id=MVP_308367)

[Set-SPOTenant](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/set-spotenant?view=sharepoint-ps&wt.mc_id=MVP_308367)
[Set-SPOSite](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/set-sposite?view=sharepoint-ps&wt.mc_id=MVP_308367)