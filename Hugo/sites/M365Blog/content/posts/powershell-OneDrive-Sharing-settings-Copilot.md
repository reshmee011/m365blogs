---
title: "Empowering Secure Collaboration: Configuring OneDrive Sharing Tenant and Site Settings with PowerShell to prevent oversharing"
date: 2024-05-16T06:51:10Z
tags: ["SharePoint","Sharing","Tenant","Sites","PowerShell","OneDrive","Copilot for M365","Information Governance","IG"]
featured_image: '/posts/images/powershell-sharePoint-sharing-permissions-copilot/TenantSharingOptions_filefolderdomain.png'
draft: true
---

# Empowering Secure Collaboration: Configuring OneDrive Sharing Tenant Settings with PowerShell

OneDrive makes it easy to collaborate by sharing files and folders with others.
OneDrive is the storage space for personal productivity and not meant for collaboration and is critical for usage for other M365 Apps.  
- Files shared to chats within Teams
- OneNote
- Personal lists and Document storage
- Shortcuts to SharePoint sites/libraries
- Favourites 
- Loops in chats
- Streams

In this article, we'll delve into how PowerShell can empower SharePoint administrators to configure OneDrive sharing settings proactively at the tenant levels to help with Copilor for M365 rollout to help with oversharing.

## Tenant Level Sharing Settings

Refer to the post which covers some of the SharePoint level settings which applies for OneDrive too
[Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell to prevent oversharing](https://reshmeeauckloo.com/posts/powershell-sharepoint-sharing-permissions-copilot/)

The SharePoint settings that apply to OneDrive are: SharingCapability, ShowAllUsersClaim, ShowEveryoneClaim, ShowEveryoneExceptExternalUsersClaim,RequireAnonymousLinksExpireInDays, NotifyOwnersWhenItemsReshared, DefaultSharingLinkType, PreventExternalUsersFromResharing, ShowPeoplePickerSuggestionsForGuestUsers, FileAnonymousLinkType, FolderAnonymousLinkType, DefaultLinkPermission, RequireAcceptingAccountMatchInvitedAccount, SharingAllowedDomainList, SharingBlockedDomainList, SharingDomainRestrictionMode, ExternalUserExpirationRequired, ExternalUserExpireInDays, EnableRestrictedAccessControl

There are specific OneDrive settings you may look into for information governance.

### OneDriveSharingCapability

Configures sharing capability for mysite (onedrive), offering options similar to those for the overall tenant.

The valid values are:

* ExternalUserAndGuestSharing (default) - External user sharing (share by email) and guest link sharing are both enabled: Anyone - users can share files and folders using links that don't require sign-in

* Disabled - External user sharing (share by email) and guest link sharing are both disabled - Only people in your organization - no external sharing allowed

* ExternalUserSharingOnly - External user sharing (share by email) is enabled, but guest link sharing is disabled. New and existing guests - guests must sign in or provide a verification code

* ExistingExternalUserSharingOnly - Existing guests only - only guests already in your organization's directory

[Change the external sharing setting for a user's OneDrive](https://learn.microsoft.com/en-us/sharepoint/user-external-sharing-settings?wt.mc_id=MVP_308367)

### ProvisionSharedWithEveryoneFolder

It is an unsupported feature. Trying to set it to $true will throw the following error message

```powerShell
Set-SPOTenant : Provisioning of the SharedWithEveryone Folder within OneDrive for Business is no longer supported.
At line:1 char:1
+ Set-SPOTenant -ProvisionSharedWithEveryoneFolder $true
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Set-SPOTenant], ServerException
    + FullyQualifiedErrorId : Microsoft.SharePoint.Client.ServerException,Microsoft.Online.SharePoint.PowerShell.SetTenant
```

It is already set to $true, it will automatically createa a "Shared with Everyone" folder in users' OneDrive document libraries, enhancing collaboration.

Use other methods to create a shared folder to share with all internal users within the organization. For example, a shared folder or Document library can be created within in a SharePoint site with appropriate permission settings to ensure only the right people have access. 

### OneDriveDefaultShareLinkScope

Gets or sets default share link scope for files on OneDrive sites.

The valid values are:

* Anyone
* Organization
* SpecificPeople
* Uninitialized


### OneDriveDefaultShareLinkRole

Gets or sets default share link role for files on OneDrive sites.

Note: Although the values below may be viewable in Powershell, only View OR Edit may be set at this time.

The valid values are:

* Edit
* View
* LimitedEdit
* LimitedView
* ManageList
* None
* Owner
* RestrictedView
* Review
* Submit

### OneDriveLoopDefaultSharingLinkScope

Gets or sets default share link scope for Microsoft Loop on OneDrive sites.

The valid values are:

* Anyone
* Organization
* SpecificPeople
* Uninitialized


### OneDriveLoopDefaultSharingLinkRole

Gets or sets default share link role for Loop and Whiteboard files on OneDrive sites.

Note: Although the values below may be viewable in Powershell, only View OR Edit may be set at this time.

The valid values are:

* Edit
* View
* LimitedEdit
* LimitedView
* ManageList
* None
* Owner
* RestrictedView
* Review
* Submit


### OneDriveDefaultLinkToExistingAccess

When set to true, default sharing link will a People with Existing Access link (which does not modify permissions) maintaining permission integrity.
![linkSettings](../images/powershell-sharePoint-sharing-permissions-copilot/linktoexistingaccess.png)

### OneDriveBlockGuestsAsSiteAdmin

Prevents Guests being set up as site administrators

It is still an experimental feature at the time of writing the article.

### OneDriveRequestFilesLinkEnabled

Enable or disable the Request files link on the OneDrive partition for all OneDrive sites. If this value is not set, the Request files link will only show for OneDrives with Anyone links enabled.

[How to use the “Request Files” Feature in OneDrive for Business?](https://www.sharepointdiary.com/2021/06/request-files-feature-in-onedrive-for-business.html) mentions some benefits of this feature. It makes it easy to collect files from external users like suppliers, clients, etc.. securely without providing them any extra permissions. The ability for the external user to access the dedicated OneDrive Folder to upload a file by the end user via a link without the need of email attachments or manual file transfers and access to the OneDrive simplifies file collection, provides secure and controlled access, seamless collaboration, centralised file management and automated notifications.

Read more from [Enable File Requests in SharePoint or OneDrive](https://learn.microsoft.com/en-us/sharepoint/enable-file-requests?wt.mc_id=MVP_308367)

### OneDriveRequestFilesLinkExpirationInDays

Specifies the number of days before a Request files link expires for all OneDrive sites.

The value can be from 0 to 730 days.

To remove the expiration requirement, set the value to zero (0).

Read more from [Enable File Requests in SharePoint or OneDrive](https://learn.microsoft.com/en-us/sharepoint/enable-file-requests?wt.mc_id=MVP_308367)

### OwnerAnonymousNotification

Enables or disables owner anonymous notification. If enabled, an email notification will be sent to the OneDrive for Business owners when an anonymous links are created or changed.

PARAMVALUE: $true | $false

### OrphanedPersonalSitesRetentionPeriod

Specifies the number of days after a user's Entra ID account is deleted that their OneDrive content will be deleted.

The value range is in days, between 30 and 3650. The default value is 30.

[OneDrive retention and deletion](https://learn.microsoft.com/en-us/sharepoint/retention-and-deletion?wt.mc_id=MVP_308367)

### OneDriveStorageQuota

Sets a default OneDrive for Business storage quota for the tenant. It will be used for new OneDrive for Business sites created.

A typical use will be to reduce the amount of storage associated with OneDrive for Business to a level below what the License entitles the users. For example, it could be used to set the quota to 10 gigabytes (GB) by default.

If value is set to 0, the parameter will have no effect.

If the value is set larger than the Maximum allowed OneDrive for Business quota, it will have no effect.

### OneDriveForGuestsEnabled

Lets OneDrive for Business creation for administrator managed guest users. Administrator managed Guest users use credentials in the resource tenant to access the resources.

The valid values are:

* true - Administrator managed Guest users can be given OneDrives, provided needed licenses are assigned.
* false - Administrator managed Guest users can't be given OneDrives as functionality is turned off.

If set to $true, you will be prompted with the following promtp. Click on Yes to proceed if it's the intention to set it to true.

**This will enable all users, including guests, to create OneDrive for Business sites. You must first assign OneDrive for Business licenses to the guests before they can create their OneDrive for Business sites.**

![Warning Message Enabling Guests to OneDrive](../images/powershell-OneDrive-Sharing-settings-Copilot/WarningEnablingGuestToHaveOneDrive.png)

### DisableAddShortcutsToOneDrive

When the feature is disabled ($true), the option Add shortcut to My files will be removed; any folders that have already been added will remain on the user's computer.

The following warning message will display when set to $true.

**WARNING: Users in your organization will no longer be able to add new shortcuts to their OneDrive. Existing shortcuts will remain functional.**

Read more from [Teams Real Simple with Pictures: Disabling Add Shortcut to OneDrive](https://microsoft365pro.co.uk/2022/04/11/teams-real-simple-with-pictures-disabling-add-shortcut-to-onedrive/)

Having shortcuts within OneDrive links to other storage areas, i.e. SharePoint libraries or folders.

### DefaultOneDriveInformationBarrierMode

The DefaultOneDriveInformationBarrierMode sets the information barrier mode for all OneDrive sites.

The valid values are:

* Open
* Explicit
* Implicit
* OwnerModerated
* Mixed

If not set within the tenant , you may get the following message 
**Set-SPOTenant : Information barriers haven't been enabled in your organization.**

Run the following to enable information barriers in SharePoint and OneDrive

```PowerShell
 Set-SPOTenant -InformationBarriersSuspension $false 
```

[Use information barriers with SharePoint](https://learn.microsoft.com/en-us/purview/information-barriers-sharepoint?wt.mc_id=MVP_308367)

### ContentTypeSyncSiteTemplatesList [String[]] [-ExcludeSiteTemplate]

By default Content Type Hub will no longer push content types to OneDrive for Business sites (formerly known as MySites). 
 Enables Content Type Hub to push content types to all OneDrive for Business sites

-ContentTypeSyncSiteTemplatesList MySites -ExcludeSiteTemplate

stops publishing content types to OneDrive for Business sites.

### BlockUserInfoVisibilityInOneDrive

Blocks users from accessing User Info if they have Limited Access permission only to the OneDrive. 

The valid values are:

* ApplyToNoUsers (default) - No users are prevented from accessing User Info when they have Limited Access permission only.
* ApplyToAllUsers - All users (internal or external) are prevented from accessing User Info if they have Limited Access permission only.
* ApplyToGuestAndExternalUsers - Only external or guest users are prevented from accessing User Info if they have Limited Access permission only.
* ApplyToInternalUsers - Only internal users are prevented from accessing User Info if they have Limited Access permission only.

### NotificationsInOneDriveForBusinessEnabled

Enables or disables notifications in OneDrive for Business.

Follow the guidance in the link, using the settings from below: [Control notifications - OneDrive | Microsoft Docs](https://docs.microsoft.com/en-us/onedrive/turn-on-external-sharing-notifications#allow-or-block-notifications?wt.mc_id=MVP_308367)

From the UI 

* SharePoint Admin Center > Settings > OneDrive Notifications 
* Tick "Allow notifications"

### NotifyOwnersWhenInvitationsAccepted

When this parameter is set to true and when an external user accepts an invitation to a resource in a user's OneDrive for Business, the OneDrive for Business owner is notified by e-mail.

For additional information about how to configure notifications for external sharing, see Configure notifications for external sharing for OneDrive for Business.

### NotifyOwnersWhenItemsReshared

When this parameter is set to $true and another user re-shares a document from a user's OneDrive for Business, the OneDrive for Business owner is notified by e-mail.

For additional information about how to configure notifications for external sharing, see Configure notifications for external sharing for OneDrive for Business.

The valid values are $true and $false.

[Turn on OneDrive file activity notifications](https://learn.microsoft.com/en-us/sharepoint/turn-on-external-sharing-notifications?&wt.mc_id=MVP_308367)

### ODBAccessRequests 

Enables/Disables access requests to OneDrive for Business.

The valid values are:

On - Users without permission to share can trigger sharing requests to the OneDrive for Business owner when they attempt to share. Also, users without permission to a file or folder can trigger access requests to the OneDrive for Business owner when they attempt to access an item they do not have permissions to.

Off - Prevent access requests and requests to share on OneDrive for Business.
Unspecified - Let each OneDrive for Business owner enable or disable access requests and requests to share on their OneDrive.

### ODBMembersCanShare

Sets policy on re-sharing behavior for those who have access in OneDrive for Business.

The valid values are:

* On - Users with edit permissions can re-share.
* Off - Only OneDrive for Business owner can share. The value of ODBAccessRequests defines whether a request to share gets sent to the owner.
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

**SPO PowerShell to get OneDrive Specific properties**

```powershell
get-SPOTenant | select-object -property ODBAccessRequests `
            ,ODBMembersCanShare `
            ,OneDriveRequestFilesLinkEnabled `
            ,DisableAddShortCutsToOneDrive  `
            ,OneDriveSharingCapability `
            ,ProvisionSharedWithEveryoneFolder `
            ,OneDriveDefaultShareLinkScope `
            ,OneDriveDefaultShareLinkRole `
            ,OneDriveLoopDefaultSharingLinkRole `
            ,OneDriveDefaultLinkToExistingAccess `
            ,OneDriveBlockGuestsAsSiteAdmin `
            ,OwnerAnonymousNotification `
            ,OrphanedPersonalSitesRetentionPeriod `
            ,OneDriveStorageQuota `
            ,OneDriveRequestFilesLinkExpirationInDays `
            ,OneDriveForGuestsEnabled `
            ,DefaultOneDriveInformationBarrierMode `
            ,ContentTypeSyncSiteTemplatesList `
            ,BlockUserInfoVisibilityInOneDrive `
            ,NotificationsInOneDriveForBusinessEnabled `
            ,NotifyOwnersWhenInvitationsAccepted `
            ,NotifyOwnersWhenItemsReshared
```

**SPO PowerShell to update OneDrive Specific properties**

```PowerShell
connect-SPOService -Url https://contoso-admin.sharepoint.com
##OneDrive specific settings
Set-SPOTenant -ODBAccessRequests Off `
            -ODBMembersCanShare On `
            -OneDriveRequestFilesLinkEnabled $true `
            -DisableAddShortCutsToOneDrive $true  `
            -OneDriveSharingCapability ExternalUserAndGuestSharing `
            -OneDriveDefaultShareLinkScope Organization `
            -OneDriveDefaultShareLinkRole Edit `
            -OneDriveLoopDefaultSharingLinkRole Edit `
            -OneDriveLoopDefaultSharingLinkScope Anyone `
            -OneDriveDefaultLinkToExistingAccess $true `
            -OwnerAnonymousNotification $true `
            -OrphanedPersonalSitesRetentionPeriod 30 `
            -OneDriveStorageQuota 1048576 `
            -OneDriveRequestFilesLinkExpirationInDays 60 `
            -OneDriveForGuestsEnabled $false `
            -ContentTypeSyncSiteTemplatesList `
            -BlockUserInfoVisibilityInOneDrive ApplyToNoUsers `
            -NotificationsInOneDriveForBusinessEnabled $true `
            -NotifyOwnersWhenInvitationsAccepted $true `
            -NotifyOwnersWhenItemsReshared $true

            # -DefaultOneDriveInformationBarrierMode   ` dependent on InformationBarriersSuspension being on
            # -OneDriveBlockGuestsAsSiteAdmin On ` Experimental feature
Set-SPOTenantSyncClientRestriction -ExcludedFileExtensions "exe;mp3;mp4"

```

**PnP PowerShell**

```PowerShell
connect-pnponline -url https://contoso-admin.sharepoint.com -interactive
Set-PnPTenant -ODBAccessRequests Off `
            -ODBMembersCanShare On `
            -OneDriveRequestFilesLinkEnabled $true `
            -DisableAddShortCutsToOneDrive $true  `
            -OneDriveSharingCapability ExternalUserAndGuestSharing `
            -OneDriveDefaultShareLinkScope Organization `
            -OneDriveDefaultShareLinkRole Edit `
            -OneDriveLoopDefaultSharingLinkRole Edit `
            -OneDriveLoopDefaultSharingLinkScope Anyone `
            -OneDriveDefaultLinkToExistingAccess $true `
            -OneDriveBlockGuestsAsSiteAdmin On `
            -OwnerAnonymousNotification $true `
            -OrphanedPersonalSitesRetentionPeriod 30 `
            -OneDriveStorageQuota 1048576 `
            -OneDriveRequestFilesLinkExpirationInDays 60 `
            -OneDriveForGuestsEnabled $false `
            -ContentTypeSyncSiteTemplatesList `
            -BlockUserInfoVisibilityInOneDrive ApplyToNoUsers `
            -NotificationsInOneDriveForBusinessEnabled $true `
            -NotifyOwnersWhenInvitationsAccepted $true `
            -NotifyOwnersWhenItemsReshared $true
#Set blocked File types
Set-PnPTenantSyncClientRestriction -ExcludedFileExtensions "exe;mp3;mp4"
```
 
## Site Level Settings

For OneDrive for Business site collection, the valid parameters are  ConditionalAccessPolicy, DefaultLinkPermission, DefaultSharingLinkType, DisableCompanyWideSharingLinks, LimitedAccessFileType, SharingAllowedDomainList, SharingBlockedDomainList, SharingCapability, SharingDomainRestrictionMode and ShowPeoplePickerSuggestionsForGuestUsers

Review [Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell to prevent oversharing](https://reshmeeauckloo.com/posts/powershell-sharepoint-sharing-permissions-copilot/) for more details about the above sharing settings.

## Other features to consider

### SharePoint Premium features

Read [Manage SharePoint Premium Settings using PowerShell](https://reshmeeauckloo.com/posts/powershell-sharepoint-premium-settings/)

## Conclusion

Effective management of sharing settings is crucial for maintaining data security and compliance in SharePoint and OneDrive. PowerShell can help review and set those settings.
