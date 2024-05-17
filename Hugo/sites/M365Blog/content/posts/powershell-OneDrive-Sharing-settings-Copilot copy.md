---
title: "Empowering Secure Collaboration: Configuring OneDrive Sharing Tenant and Site Settings with PowerShell to prevent oversharing"
date: 2024-05-16T06:51:10Z
tags: ["SharePoint","Sharing","Tenant","Sites","PowerShell","OneDrive","Copilot for M365","Information Governance","IG"]
featured_image: '/posts/images/powershell-sharePoint-sharing-permissions-copilot/TenantSharingOptions_filefolderdomain.png'
draft: true
---

# Empowering Secure Collaboration: Configuring OneDrive Sharing Tenant Settings with PowerShell to help with oversharing

OneDrive makes it easy to collaborate by sharing files and folders with others.
OneDrive is the storage space for personal productivity and not meant for collaboration and is critical for usage for other M365 Apps.  
- Files shared to chats within Teams
- OneNote
- Personal lists and Document storage
- Shortcuts to SharePoint sites/libraries
- Favourites 
- Loops in chats
- Streams

In this article, we'll delve into how PowerShell can empower SharePoint administrators to configure OneDrive sharing settings proactively at both the tenant and site levels. By leveraging PowerShell scripts, you can streamline the process of enforcing security measures to safeguard sensitive information and mitigate the risk of oversharing.

## Tenant Level Sharing Settings

Refer to the post which covers some of the SharePoint level settings which applies for OneDrive too
[Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell to prevent oversharing](https://reshmeeauckloo.com/posts/powershell-sharepoint-sharing-permissions-copilot/)

The SharePoint settings that apply to OneDrive are: SharingCapability, ShowAllUsersClaim, ShowEveryoneClaim, ShowEveryoneExceptExternalUsersClaim,RequireAnonymousLinksExpireInDays, NotifyOwnersWhenItemsReshared, DefaultSharingLinkType, PreventExternalUsersFromResharing, ShowPeoplePickerSuggestionsForGuestUsers, FileAnonymousLinkType, FolderAnonymousLinkType, DefaultLinkPermission, RequireAcceptingAccountMatchInvitedAccount, SharingAllowedDomainList, SharingBlockedDomainList, SharingDomainRestrictionMode, ExternalUserExpirationRequired, ExternalUserExpireInDays

There are specific OneDrive settings you may look into for information governance.

### OneDriveSharingCapability

Configures sharing capability for mysite (onedrive), offering options similar to those for the overall tenant.

The valid values are:

* ExternalUserAndGuestSharing (default) - External user sharing (share by email) and guest link sharing are both enabled: Anyone - users can share files and folders using links that don't require sign-in

* Disabled - External user sharing (share by email) and guest link sharing are both disabled - Only people in your organization - no external sharing allowed

* ExternalUserSharingOnly - External user sharing (share by email) is enabled, but guest link sharing is disabled. New and existing guests - guests must sign in or provide a verification code

* ExistingExternalUserSharingOnly - Existing guests only - only guests already in your organization's directory

### ProvisionSharedWithEveryoneFolder

Automatically creates a "Shared with Everyone" folder in users' OneDrive document libraries, enhancing collaboration.

### OneDriveDefaultShareLinkScope

Gets or sets default share link scope for Loop and Whiteboard files on OneDrive sites.

The valid values are:

Anyone
Organization
SpecificPeople
Uninitialized


### OneDriveDefaultShareLinkRole

Gets or sets default share link role for Loop and Whiteboard files on OneDrive sites.

Note: Although the values below may be viewable in Powershell, only View OR Edit may be set at this time.

The valid values are:

Edit
View
LimitedEdit
LimitedView
ManageList
None
Owner
RestrictedView
Review
Submit


### OneDriveLoopDefaultSharingLinkRole

Gets or sets default share link role for Loop and Whiteboard files on OneDrive sites.

Note: Although the values below may be viewable in Powershell, only View OR Edit may be set at this time.

The valid values are:

Edit
View
LimitedEdit
LimitedView
ManageList
None
Owner
RestrictedView
Review
Submit


### OneDriveDefaultLinkToExistingAccess
: False

### OneDriveBlockGuestsAsSiteAdmin
: Unspecified

### OneDriveRequestFilesLinkEnabled

Enable or disable the Request files link on the OneDrive partition for all OneDrive sites. If this value is not set, the Request files link will only show for OneDrives with Anyone links enabled.

[How to use the “Request Files” Feature in OneDrive for Business?](https://www.sharepointdiary.com/2021/06/request-files-feature-in-onedrive-for-business.html) mentions some benefits of this feature. It makes it easy to collect files from external users like suppliers, clients, etc.. securely without providing them any extra permissions. The ability for the external user to access the dedicated OneDrive Folder to upload a file by the end user via a link without the need of email attachments or manual file transfers and access to the OneDrive simplifies file collection, provides secure and controlled access, seamless collaboration, centralised file management and automated notifications.

### OwnerAnonymousNotification

Enables or disables owner anonymous notification. If enabled, an email notification will be sent to the OneDrive for Business owners when an anonymous links are created or changed.

PARAMVALUE: $true | $false

### OrphanedPersonalSitesRetentionPeriod

Specifies the number of days after a user's Active Directory account is deleted that their OneDrive for Business content will be deleted.

The value range is in days, between 30 and 3650. The default value is 30.

### OneDriveStorageQuota

Sets a default OneDrive for Business storage quota for the tenant. It will be used for new OneDrive for Business sites created.

A typical use will be to reduce the amount of storage associated with OneDrive for Business to a level below what the License entitles the users. For example, it could be used to set the quota to 10 gigabytes (GB) by default.

If value is set to 0, the parameter will have no effect.

If the value is set larger than the Maximum allowed OneDrive for Business quota, it will have no effect.

### OneDriveRequestFilesLinkExpirationInDays

Specifies the number of days before a Request files link expires for all OneDrive sites.

The value can be from 0 to 730 days.

To remove the expiration requirement, set the value to zero (0).

### OneDriveForGuestsEnabled

Lets OneDrive for Business creation for administrator managed guest users. Administrator managed Guest users use credentials in the resource tenant to access the resources.

The valid values are:

$true - Administrator managed Guest users can be given OneDrives, provided needed licenses are assigned.
$false - Administrator managed Guest users can't be given OneDrives as functionality is turned off.

DisableAddShortcutsToOneDrive
When the feature is disabled ($true), the option Add shortcut to My files will be removed; any folders that have already been added will remain on the user's computer.


### DefaultOneDriveInformationBarrierMode 

The DefaultOneDriveInformationBarrierMode sets the information barrier mode for all OneDrive sites.

The valid values are:

Open
Explicit
Implicit
OwnerModerated
Mixed

(Use information barriers with SharePoint)[https://learn.microsoft.com/en-us/purview/information-barriers-sharepoint]

### ContentTypeSyncSiteTemplatesList [String[]] [-ExcludeSiteTemplate]

By default Content Type Hub will no longer push content types to OneDrive for Business sites (formerly known as MySites). 
 Enables Content Type Hub to push content types to all OneDrive for Business sites

-ContentTypeSyncSiteTemplatesList MySites -ExcludeSiteTemplate

stops publishing content types to OneDrive for Business sites.

### BlockUserInfoVisibilityInOneDrive

The valid values are:

* ApplyToNoUsers (default) - No users are prevented from accessing User Info when they have Limited Access permission only.

* ApplyToAllUsers - All users (internal or external) are prevented from accessing User Info if they have Limited Access permission only.

* ApplyToGuestAndExternalUsers - Only external or guest users are prevented from accessing User Info if they have Limited Access permission only.

* ApplyToInternalUsers - Only internal users are prevented from accessing User Info if they have Limited Access permission only.

### NotificationsInOneDriveForBusinessEnabled

Enables or disables notifications in OneDrive for Business.

PARAMVALUE: $true | $false

Follow the guidance in the link, using the settings from below: [Control notifications - OneDrive | Microsoft Docs](https://docs.microsoft.com/en-us/onedrive/turn-on-external-sharing-notifications#allow-or-block-notifications?wt.mc_id=MVP_308367)

From the UI 

* SharePoint Admin Center > Settings > OneDrive Notifications 

* Tick "Allow notifications"


### NotifyOwnersWhenInvitationsAccepted

When this parameter is set to $true and when an external user accepts an invitation to a resource in a user's OneDrive for Business, the OneDrive for Business owner is notified by e-mail.

For additional information about how to configure notifications for external sharing, see Configure notifications for external sharing for OneDrive for Business.

The valid values are $true and $false.


### NotifyOwnersWhenItemsReshared

When this parameter is set to $true and another user re-shares a document from a user's OneDrive for Business, the OneDrive for Business owner is notified by e-mail.

For additional information about how to configure notifications for external sharing, see Configure notifications for external sharing for OneDrive for Business.

The valid values are $true and $false.

### ODBAccessRequests 

Enables/Disables access requests to OneDrive for Business.

The valid values are:

On - Users without permission to share can trigger sharing requests to the OneDrive for Business owner when they attempt to share. Also, users without permission to a file or folder can trigger access requests to the OneDrive for Business owner when they attempt to access an item they do not have permissions to.

Off - Prevent access requests and requests to share on OneDrive for Business.
Unspecified - Let each OneDrive for Business owner enable or disable access requests and requests to share on their OneDrive.

### ODBMembersCanShare

Sets policy on re-sharing behavior for those who have access in OneDrive for Business.

The valid values are:

On - Users with edit permissions can re-share.
Off - Only OneDrive for Business owner can share. The value of ODBAccessRequests defines whether a request to share gets sent to the owner.
Unspecified - Let each OneDrive for Business owner enable or disable re-sharing behavior on their OneDrive.

[Configure Access Requests and Sharing Settings in OneDrive for Business Sites](https://www.sharepointdiary.com/2019/09/configure-onedrive-for-business-access-requests-settings.html#ixzz8ZWETbmmi?&wt.mc_id=MVP_308367)

### Set-SPOTenantSyncClientRestriction

By default, Users can upload all types of files in their OneDrive for Business folder. However, for security and compliance reasons, we can restrict users from syncing specific file types in OneDrive for Business. This guide will explain how to block specific file types in OneDrive for Business using the OneDrive admin center and PowerShell.


[OneDrive for Business: Block Syncing of Specific File Types](https://www.sharepointdiary.com/2019/09/onedrive-for-business-block-syncing-specific-file-types.html?wt.mc_id=MVP_308367)


### Add additional site collection admin to site

 By default, the primary site collection administrator or the OneDrive Owner rights is set to the user who owns the MySite. If user's OneDrive for Business site or need share OneDrive with another user. E.g. when the user leaves the company, the manager of the user may need access or for reasons : backup, compliance, delegation, security, etc.

Read more: [How to Grant Access to another User’s OneDrive in Office 365](https://www.sharepointdiary.com/2017/04/gain-admin-permission-to-onedrive-for-business-sites-using-powershell.html#ixzz8ZWGLSb3I)

```powershell
#Add Site Collection Admin to the OneDrive
Set-SPOUser -Site $OneDriveSiteUrl -LoginName $SiteCollAdmin -IsSiteCollectionAdmin $True
```

### Sample script to amend tenant level sharing settings 

**SPO PowerShell**

```PowerShell
connect-SPOService -Url https://contoso-admin.sharepoint.com
$SiteCollAdmin = "admin@contoso.sharepoint.com"
##SharePoint specific settings
Set-SPOTenant -ODBAccessRequests Off ` 
            -ODBMembersCanShare On ` 
            -OneDriveRequestFilesLinkEnabled $true `
            -DisableAddShortCutsToOneDrive $true  `
            -OneDriveSharingCapability `
            -ProvisionSharedWithEveryoneFolder `
            -OneDriveDefaultShareLinkScope `
            -OneDriveDefaultShareLinkRole `
            -OneDriveLoopDefaultSharingLinkRole `
            -OneDriveDefaultLinkToExistingAccess `
            -OneDriveBlockGuestsAsSiteAdmin `
            -OneDriveRequestFilesLinkEnabled `
            -OwnerAnonymousNotification `
            -OrphanedPersonalSitesRetentionPeriod `
            -OneDriveStorageQuota `
            -OneDriveRequestFilesLinkExpirationInDays `
            -OneDriveForGuestsEnabled `
            -DefaultOneDriveInformationBarrierMode `
            -ContentTypeSyncSiteTemplatesList `
            -BlockUserInfoVisibilityInOneDrive `    
            -NotificationsInOneDriveForBusinessEnabled `
            -NotifyOwnersWhenInvitationsAccepted `
            -NotifyOwnersWhenItemsReshared `
            -ODBAccessRequests `
            -ODBMembersCanShare `
        
Set-SPOTenantSyncClientRestriction -ExcludedFileExtensions "exe;mp3;mp4"

#Add Site Collection Admin to the OneDrive
Set-SPOUser -Site $OneDriveSiteUrl -LoginName $SiteCollAdmin -IsSiteCollectionAdmin $True
```

**PnP PowerShell**

```PowerShell
connect-pnponline -url https://contoso-admin.sharepoint.com -interactive
Set-PnPTenant -SharingCapability ExternalUserAndGuestSharing `
            -CoreSharingCapability ExternalUserAndGuestSharing ` # I have created a PR to include into PnP PowerShell, not yet released
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
            -AllowEveryoneExceptExternalUsersClaimInPrivateSite $true ` # I have created a PR to include into PnP PowerShell, not yet released
            -ExternalUserExpireInDays 60 
#Set blocked File types
Set-PnPTenantSyncClientRestriction -ExcludedFileExtensions "exe;mp3;mp4"
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

[How to protect sensitive information in SharePoint Online using Purview Sensitivity Labels](https://www.sharepointeurope.com/how-to-protect-sensitive-information-in-sharepoint-online-using-purview-sensitivity-labels/?utm_source=collab365&utm_medium=collab365today&utm_campaign=daily_digest)

[Sharing and permissions in the SharePoint modern experience](https://learn.microsoft.com/en-us/sharepoint/modern-experience-sharing-permissions)