---
title: "Harness sharing capabilities within your tenant and SharePoint site using PowerShell"
date: 2024-03-25T06:51:10Z
draft: true
---

# Harness sharing capabilities within your tenant and SharePoint site


## Tenant Level Settings
### ShowEveryoneExceptExternalUsersClaim
Enables the administrator to hide the Everyone except external users claim in the People Picker.

**SPO PowerShell**
``` 
Set-SPOTenant SharingCapability                        = 'ExternalUserSharingOnly'
            ShowEveryoneClaim                          = $false
            ShowAllUsersClaim                          = $false
            ShowEveryoneExceptExternalUsersClaim       = $true
            ProvisionSharedWithEveryoneFolder          = $false
            EnableGuestSignInAcceleration              = $false
            BccExternalSharingInvitations              = $false
            BccExternalSharingInvitationsList          = ""
            RequireAnonymousLinksExpireInDays          = 730
            SharingAllowedDomainList                   = @("contoso.com")
            SharingBlockedDomainList                   = @("contoso.com")
            SharingDomainRestrictionMode               = "None"
            DefaultSharingLinkType                     = "AnonymousAccess"
            PreventExternalUsersFromResharing          = $false
            ShowPeoplePickerSuggestionsForGuestUsers   = $false
            FileAnonymousLinkType                      = "Edit"
            FolderAnonymousLinkType                    = "Edit"
            NotifyOwnersWhenItemsReshared              = $true
            DefaultLinkPermission                      = "View"
            RequireAcceptingAccountMatchInvitedAccount = $false
```
**PnP PowerShell**
```
Set-PnPTenant ShowEveryoneClaim                          = $false
            ShowAllUsersClaim                          = $false
            ShowEveryoneExceptExternalUsersClaim       = $true
            ProvisionSharedWithEveryoneFolder          = $false
            EnableGuestSignInAcceleration              = $false
            BccExternalSharingInvitations              = $false
            BccExternalSharingInvitationsList          = ""
            RequireAnonymousLinksExpireInDays          = 730
            SharingAllowedDomainList                   = @("contoso.com")
            SharingBlockedDomainList                   = @("contoso.com")
            SharingDomainRestrictionMode               = "None"
            DefaultSharingLinkType                     = "AnonymousAccess"
            PreventExternalUsersFromResharing          = $false
            ShowPeoplePickerSuggestionsForGuestUsers   = $false
            FileAnonymousLinkType                      = "Edit"
            FolderAnonymousLinkType                    = "Edit"
            NotifyOwnersWhenItemsReshared              = $true
            DefaultLinkPermission                      = "View"
            RequireAcceptingAccountMatchInvitedAccount = $false
```

### ShowAllUsersClaim
Enables the administrator to hide the All Users claim groups in People Picker.

### ShowEveryoneClaim
	Enables the administrator to hide the Everyone claim in the People Picker.
### MySiteSharingCapability
	Configures sharing capability for mysite (onedrive)
 ExistingExternalUserSharingOnly, ExternalUserAndGuestSharing, Disabled, ExternalUserSharingOnly
### SharingCapability
Configures sharing capability for SharePoint
ExistingExternalUserSharingOnly, ExternalUserAndGuestSharing, Disabled, ExternalUserSharingOnly

### ProvisionSharedWithEveryoneFolder
Creates a Shared with Everyone folder in every user's new OneDrive for Business document library.

### EnableGuestSignInAcceleration
Accelerates guest-enabled site collections as well as member-only site collections when the SignInAccelerationDomain parameter is set.

### RequireAnonymousLinksExpireInDays
Specifies all anonymous links that have been created (or will be created) will expire after the set number of days.

### DefaultSharingLinkType
Lets administrators choose what type of link appears is selected in the 'Get a link' sharing dialog box in OneDrive for Business and SharePoint Online

None, Direct, Internal, AnonymousAccess

### PreventExternalUsersFromResharing
Allow or deny external users re-sharing

### ShowPeoplePickerSuggestionsForGuestUsers
Enables the administrator to hide the guest users claim in the People Picker

### FileAnonymousLinkType
Configures anonymous link types for files. View, Edit

### FolderAnonymousLinkType
Configures anonymous link types for folders. View, Edit

### NotifyOwnersWhenItemsReshared
When this parameter is set to $true and another user re-shares a document from a userâs OneDrive for Business, the OneDrive for Business owner is notified by e-mail.

### DefaultLinkPermission
Specifies the link permission on the tenant level. Valid values to set are View and Edit. A value of None will be set to Edit as its the default value. None, View, Edit

### RequireAcceptingAccountMatchInvitedAccount
Ensures that an external user can only accept an external sharing invitation with an account matching the invited email address.Administrators who desire increased control over external collaborators should consider enabling this feature. False (default) - When a document is shared with an external user, bob@contoso.com, it can be accepted by any user with access to the invitation link in the original e-mail.True - User must accept this invitation with bob@contoso.com.

### ExternalUserExpirationRequired
Enable Guest access to a site or Onedrive to expire after

### ExternalUserExpireInDays
	Specifies Number of days for Guest Access links to expire.

 
## Site Level Settings

### DisableCompanyWideSharingLinks
Disabled , Enabled

### DefaultLinkToExistingAccess 
boolean

### DisableSharingForNonOwners
boolean

### ExternalSharingTipsEnabled

### ExternalUserExpirationInDays

### ShareByEmailEnabled

### ShareByLinkEnabled

### ShowPeoplePickerSuggestionsForGuestUsers

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

### OverrideTenantExternalUserExpirationPolicy

### SharingLockDownEnabled

### SharingLockDownCanBeCleared

### LoopDefaultSharingLinkScope

### LoopDeafualtSharingLinkRole

### DefaultShareLinkScope

### DefaultShareLinkRole

### BlockGuestsAsSiteAdmin

### RestrictedAccessControl

### RestrictedAccessControlGroups 

Set-PnPSite -DisableCompanyWideSharingLinks Disabled -DefaultLinkToExistingAccess $true -DisableSharingForNonOwners | Out-Null  

```powerShell
Enables the administrator to hide the "Everyone except external users" claim in the People Picker. When users share an item with "Everyone except external users", it is accessible to all organization members in the tenant's Azure Active Directory, but not to any users who have previously accepted invitations.

The valid values are: True(default) - The Everyone except external users is displayed in People Picker. False - The Everyone except external users claim is not visible in People Picker.
```

There are other tenant wide level settings but I think it would have been great to be able to set those at the library or site level and have site power users have greater control over them 


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
https://danielanderson.io/?ck_subscriber_id=2437811072
[ShowEveryoneExceptExternalUsersClaim](https://pnp.github.io/powershell/cmdlets/Set-PnPTenant.html#-showeveryoneexceptexternalusersclaim)
[SPOSharingSettings](https://microsoft365dsc.com/resources/sharepoint/SPOSharingSettings/)
