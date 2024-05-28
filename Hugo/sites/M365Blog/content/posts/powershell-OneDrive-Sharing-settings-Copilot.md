---
title: "Empowering Secure Collaboration: Configuring OneDrive Tenant and Site Settings"
date: 2024-05-19T06:51:10Z
tags: ["SharePoint","Sharing","Tenant","Sites","PowerShell","OneDrive","Copilot for M365","Information Governance","IG"]
featured_image: '/posts/images/powershell-sharePoint-sharing-permissions-copilot/TenantSharingOptions_filefolderdomain.png'
draft: false
---

# Empowering Secure Collaboration: Configuring OneDrive Tenant Settings with PowerShell

OneDrive makes it easy to collaborate by sharing files and folders with others. OneDrive is the storage space for personal productivity and not meant for collaboration. Data stored within OneDrive are 
 
- Files shared to chats within Teams
- Files shared with end user by other OneDrive users.
- OneNote
- Personal lists and Document storage
- Shortcuts to SharePoint sites/libraries
- Favourites
- Loops within chats
- Streams

In this article, we'll explore into how PowerShell can empower SharePoint administrators to configure OneDrive sharing settings proactively at the tenant level, addressing oversharing concerns for the Copilot for M365 rollout.

## Tenant Level Sharing Settings

The SharePoint settings influencing OneDrive include: SharingCapability, ShowAllUsersClaim, ShowEveryoneClaim, ShowEveryoneExceptExternalUsersClaim, RequireAnonymousLinksExpireInDays, NotifyOwnersWhenItemsReshared, DefaultSharingLinkType, PreventExternalUsersFromResharing, ShowPeoplePickerSuggestionsForGuestUsers, FileAnonymousLinkType, FolderAnonymousLinkType, DefaultLinkPermission, RequireAcceptingAccountMatchInvitedAccount, SharingAllowedDomainList, SharingBlockedDomainList, SharingDomainRestrictionMode, ExternalUserExpirationRequired, ExternalUserExpireInDays, and EnableRestrictedAccessControl.

For targeted OneDrive information governance, consider these specific settings:


Refer to the post which covers some of the SharePoint level settings which applies for OneDrive too
[Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell to prevent oversharing](https://reshmeeauckloo.com/posts/powershell-sharepoint-sharing-permissions-copilot/)


### OneDriveSharingCapability

This setting adjusts the sharing capabilities for OneDrive, aligning with tenant-wide options.

Valid options include:

- **ExternalUserAndGuestSharing** (default): Enables both external user sharing (via email) and guest link sharing, allowing file and folder sharing with links that don't require sign-in.
- **Disabled**: Disables both external user sharing and guest link sharing, restricting sharing to internal organization members only.
- **ExternalUserSharingOnly**: Allows external user sharing via email, but disables guest link sharing, requiring new and existing guests to sign in or provide a verification code.
- **ExistingExternalUserSharingOnly**: Limits sharing to guests already in your organization's directory.

For changing a user's OneDrive external sharing settings, visit [Change the external sharing setting for a user's OneDrive](https://learn.microsoft.com/en-us/sharepoint/user-external-sharing-settings?wt.mc_id=MVP_308367).

### ProvisionSharedWithEveryoneFolder

This feature is deprecated. Attempting to enable it results in an error:

```powerShell
Set-SPOTenant : Provisioning of the SharedWithEveryone Folder within OneDrive for Business is no longer supported.
At line:1 char:1
+ Set-SPOTenant -ProvisionSharedWithEveryoneFolder $true
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Set-SPOTenant], ServerException
    + FullyQualifiedErrorId : Microsoft.SharePoint.Client.ServerException,Microsoft.Online.SharePoint.PowerShell.SetTenant
```

If previously enabled, a "Shared with Everyone" folder is created in users' OneDrive libraries to facilitate collaboration. Instead, consider creating a shared folder or document library on a SharePoint site with appropriate permissions for internal sharing.

### OneDriveDefaultShareLinkScope

Determines the default sharing link scope for OneDrive files, with options:

* Anyone
* Organization
* SpecificPeople
* Uninitialized

Attempting to set this to "Anyone" without enabling "Anonymous access links" at the tenant level triggers a warning:

**WARNING: Anonymous access links aren’t enabled for your organization. You must first enable them by running the command “Set-SPOTenant -SharingCapability ExternalUserAndGuestSharing” before you can set the DefaultSharingLinkType parameter to AnonymousAccess. We will not set the value in this case.**


### OneDriveDefaultShareLinkRole

Configures the default permission level for sharing links created for files in OneDrive.

**Note**: Despite multiple options being visible in PowerShell, currently, only "View" or "Edit" can be actively set.

Available options:

* Edit
* None
* Owner
* RestrictedView
* Review
* View

### OneDriveLoopDefaultSharingLinkScope

Determines the default audience scope for sharing links for Microsoft Loop components stored on OneDrive.

Options include:

- **Anyone**: Shareable with anyone, including external users.
- **Organization**: Limits sharing within your organization.
- **SpecificPeople**: Restricts sharing to specific individuals.
- **Uninitialized**: Default state before an explicit setting is applied.

**Error Handling**: Attempting to set the scope to 'Anyone' without enabling "Anyone access links" at the tenant level will result in an error. To resolve, execute:

```PowerShell
Set-SPOTenant -OneDriveSharingCapability ExternalUserAndGuestSharing
```

### OneDriveLoopDefaultSharingLinkRole

Sets the default permission level for sharing links for Loop and Whiteboard files in OneDrive.

Valid options:

* Edit
* None
* Owner
* RestrictedView
* Review
* View


### OneDriveDefaultLinkToExistingAccess

When enabled, the default sharing link will grant access to individuals who already have access, preserving existing permissions. This ensures that sharing links do not alter the original permission settings.

![linkSettings](../images/powershell-sharePoint-sharing-permissions-copilot/linktoexistingaccess.png)

### OneDriveBlockGuestsAsSiteAdmin

This experimental feature prevents guests from being assigned as site administrators, enhancing security by limiting administrative privileges.

It is still an experimental feature at the time of writing the article.

### OneDriveRequestFilesLinkEnabled

Enable or disable the "Request Files" feature across all OneDrive sites. By default, the "Request Files" link is available only for OneDrive sites that permit "Anyone" links.

The "Request Files" feature streamlines the process of collecting files from external parties such as suppliers and clients, securely and without granting them additional permissions. It allows external users to upload files to a designated OneDrive folder via a link, eliminating the need for email attachments or manual file transfers. This simplifies file collection, ensures secure and controlled access, fosters seamless collaboration, centralizes file management, and automates notifications.

For more insights, see [How to use the “Request Files” Feature in OneDrive for Business?](https://www.sharepointdiary.com/2021/06/request-files-feature-in-onedrive-for-business.html) and [Enable File Requests in SharePoint or OneDrive](https://learn.microsoft.com/en-us/sharepoint/enable-file-requests?wt.mc_id=MVP_308367).

### OneDriveRequestFilesLinkExpirationInDays

Defines the lifespan, in days, of a "Request Files" link across all OneDrive sites, with a permissible range of 0 to 730 days. Setting this value to zero (0) effectively removes any expiration constraint for the link.

For additional details, refer to [Enable File Requests in SharePoint or OneDrive](https://learn.microsoft.com/en-us/sharepoint/enable-file-requests?wt.mc_id=MVP_308367).

### OwnerAnonymousNotification

Toggles email notifications for OneDrive for Business owners when anonymous links are created or modified, enhancing security and oversight.

### OrphanedPersonalSitesRetentionPeriod

Determines the retention period, in days, for OneDrive content after a user's Entra ID account is deleted, updating the retention policy for all orphaned OneDrive for Business sites. The range is from 30 to 3650 days, with a default setting of 30 days.

Learn more about OneDrive retention and deletion policies at [OneDrive retention and deletion](https://learn.microsoft.com/en-us/sharepoint/retention-and-deletion?wt.mc_id=MVP_308367).

![retention](../images/powershell-OneDrive-Sharing-settings-Copilot/retention.png)

### OneDriveStorageQuota

Sets the default storage quota for new OneDrive for Business sites within the tenant. This can be used to limit the storage available to OneDrive for Business sites to below the entitlement level of the user's license, such as setting a default quota of 10 gigabytes (GB).

If the value is set to 0, it will have no effect. Similarly, if the value exceeds the maximum allowed quota for OneDrive for Business, it will be disregarded.

![storage](../images/powershell-OneDrive-Sharing-settings-Copilot/storage.png)

### OneDriveForGuestsEnabled

Allows or disallows guest users from creating OneDrive for Business sites. Guest users must be assigned OneDrive for Business licenses prior to site creation.

Valid values are:

- `true` - Enables OneDrive access for guest users, contingent on license assignment.
- `false` - Disables OneDrive access for guest users.

Activating this setting prompts a confirmation to ensure intentional enabling for guest users.

![Warning Message Enabling Guests to OneDrive](../images/powershell-OneDrive-Sharing-settings-Copilot/WarningEnablingGuestToHaveOneDrive.png)

### DisableAddShortcutsToOneDrive

Controls the ability to add new shortcuts to "My files" in OneDrive. When disabled (`$true`), the option to add shortcuts is removed, though existing shortcuts remain accessible.

A warning message appears when this setting is enabled, indicating the change in functionality.

**WARNING: Users in your organization will no longer be able to add new shortcuts to their OneDrive. Existing shortcuts will remain functional.**

For more information, see [Teams Real Simple with Pictures: Disabling Add Shortcut to OneDrive](https://microsoft365pro.co.uk/2022/04/11/teams-real-simple-with-pictures-disabling-add-shortcut-to-onedrive/).

This adjustment affects how users can link to other storage locations, such as SharePoint libraries or folders, within OneDrive.
Enable or disable the Request files link on the OneDrive partition for all OneDrive sites. If this value is not set, the Request files link will only show for OneDrives with Anyone links enabled.

### DefaultOneDriveInformationBarrierMode

Configures the default information barrier mode for all OneDrive sites, enhancing security and compliance by controlling how information is shared and accessed within the organization.

Valid values include:

- **Open**: No restrictions on information sharing or access.
- **Explicit**: Users can only access information if explicitly granted permission.
- **Implicit**: Users have implicit access but can be restricted based on policies.
- **OwnerModerated**: Site owners moderate access requests.
- **Mixed**: A combination of the above modes, tailored to specific organizational needs.

If information barriers are not enabled in your organization, attempting to set this property will result in an error:

**Set-SPOTenant : Information barriers haven't been enabled in your organization.**

To enable information barriers in SharePoint and OneDrive, execute:

```PowerShell
Set-SPOTenant -InformationBarriersSuspension $false
```
For more details, visit [Use information barriers with SharePoint](https://learn.microsoft.com/en-us/purview/information-barriers-sharepoint?wt.mc_id=MVP_308367)

### ContentTypeSyncSiteTemplatesList

Controls whether the Content Type Hub pushes content types to OneDrive for Business sites. By default, new content types from the Content Type Hub are not automatically pushed to OneDrive for Business sites.

To enable syncing of content types to OneDrive for Business sites, use:

```PowerShell
Set-SPOTenant -ContentTypeSyncSiteTemplatesList MySites
```

To disable this syncing, execute:

```powershell
Set-SPOTenant -ContentTypeSyncSiteTemplatesList MySites -ExcludeSiteTemplate
```

### BlockUserInfoVisibilityInOneDrive

Prevents specific users from accessing User Info in OneDrive if they only have Limited Access permission. This setting enhances privacy and security by limiting information visibility based on user permissions.

The valid options for controlling user access to User Info in OneDrive based on Limited Access permission are:

- **ApplyToNoUsers** (default): No restrictions are imposed on users. Everyone can access User Info, even with Limited Access permission.
- **ApplyToAllUsers**: Restricts all users, whether internal or external, from accessing User Info when they only have Limited Access permission.
- **ApplyToGuestAndExternalUsers**: Specifically targets external or guest users, preventing them from accessing User Info if their permission level is Limited Access only.
- **ApplyToInternalUsers**: Applies restrictions solely to internal users, barring them from User Info access under Limited Access permission.

### NotificationsInOneDriveForBusinessEnabled

This setting enables or disables notifications within OneDrive for Business, allowing for more granular control over the communication flow related to file sharing and collaboration activities.

To manage notifications, follow these steps:

1. Navigate to the **SharePoint Admin Center**.
2. Select **Settings** > **OneDrive Notifications**.
3. Enable the option for **Allow notifications** to activate notifications.

![OneDrive Notifications](../images/powershell-OneDrive-Sharing-settings-Copilot/Notifications.png)

For detailed guidance, refer to [Control notifications in OneDrive](https://docs.microsoft.com/en-us/onedrive/turn-on-external-sharing-notifications#allow-or-block-notifications?wt.mc_id=MVP_308367) and [Turn on OneDrive file activity notifications](https://learn.microsoft.com/en-us/sharepoint/turn-on-external-sharing-notifications?WT.mc_id=365AdminCSH_spo?wt.mc_id=MVP_308367)

### NotifyOwnersWhenInvitationsAccepted

Setting this parameter to **true** activates email notifications for OneDrive for Business owners whenever an external user accepts an invitation to access resources within their OneDrive. This feature enhances the visibility of external user engagement and resource access.

For more information on configuring external sharing notifications, consult [Configure notifications for external sharing in OneDrive for Business](https://docs.microsoft.com/en-us/onedrive/turn-on-external-sharing-notifications#configure-notifications-for-external-sharing?wt.mc_id=MVP_308367).

### NotifyOwnersWhenItemsReshared

Enabling this feature by setting the parameter to **$true** ensures that OneDrive for Business owners receive email notifications when a document within their OneDrive is re-shared by another user. This functionality aids in tracking the dissemination of shared documents and maintaining oversight over document access.

For additional details on setting up notifications for file activity and external sharing, visit [Turn on OneDrive file activity notifications](https://learn.microsoft.com/en-us/sharepoint/turn-on-external-sharing-notifications?&wt.mc_id=MVP_308367).

### ODBAccessRequests

This setting controls the ability to send access requests and share requests in OneDrive for Business.

**Valid values:**

- **On**: Enables users without sharing permissions to send requests to the OneDrive owner when attempting to share. Similarly, users lacking access to a specific file or folder can send access requests to the owner.
- **Off**: Disables the functionality for access and share requests in OneDrive for Business.
- **Unspecified**: Delegates the decision to enable or disable access and share requests to individual OneDrive for Business owners.

### ODBMembersCanShare

Determines the re-sharing permissions for users with access to OneDrive for Business.

**Valid values:**

- **On**: Allows users with edit permissions to re-share items.
- **Off**: Restricts sharing capabilities to the OneDrive for Business owner only. The `ODBAccessRequests` setting determines if a share request is sent to the owner.
- **Unspecified**: Gives OneDrive for Business owners the autonomy to set their re-sharing policies.

For detailed instructions on configuring access requests and sharing settings, visit [Configure Access Requests and Sharing Settings in OneDrive for Business Sites](https://www.sharepointdiary.com/2019/09/configure-onedrive-for-business-access-requests-settings.html#ixzz8ZWETbmmi?&wt.mc_id=MVP_308367).

### Set-SPOTenantSyncClientRestriction

Administrators can restrict the types of files that users are allowed to sync to their OneDrive for Business for security and compliance reasons. This section provides guidance on how to implement such restrictions using the OneDrive admin center and PowerShell.

For a step-by-step guide on blocking specific file types from syncing in OneDrive for Business, refer to [OneDrive for Business: Block Syncing of Specific File Types](https://www.sharepointdiary.com/2019/09/onedrive-for-business-block-syncing-specific-file-types.html?wt.mc_id=MVP_308367).

![sync](../images/powershell-OneDrive-Sharing-settings-Copilot/sync.png)

### Add Additional Site Collection Admin to Site

By default, the primary site collection administrator or the OneDrive Owner rights are assigned to the MySite owner. There are scenarios, such as a user leaving the company, where it may be necessary to grant another user access to a OneDrive for Business site for backup, compliance, delegation, or security reasons.

For instructions on how to grant another user access to a OneDrive for Business site, read [How to Grant Access to another User’s OneDrive in Office 365](https://www.sharepointdiary.com/2017/04/gain-admin-permission-to-onedrive-for-business-sites-using-powershell.html#ixzz8ZWGLSb3I).

```powershell
#Add Site Collection Admin to the OneDrive
Set-SPOUser -Site $OneDriveSiteUrl -LoginName $SiteCollAdmin -IsSiteCollectionAdmin $True
```
### HideSyncButtonOnTeamSite : Turn off OneDrive sync for SharePoint libraries

Users have two options when syncing files in SharePoint libraries and Teams. They can

[Add shortcuts to libraries and folders to their OneDrive.](https://support.microsoft.com/office/d66b1347-99b7-4470-9360-ffc048d35a33?wt.mc_id=MVP_308367))
[Use the Sync button in the document library.](https://support.microsoft.com/office/6de9ede8-5b6e-4503-80b2-6190f3354a88?wt.mc_id=MVP_308367))


Adding OneDrive shortcuts allows content to be accessed on all devices, whereas sync is related to a specific device. Additionally, OneDrive shortcuts offer improved performance versus using the sync button.
Microsoft recommends using OneDrive shortcuts as the more versatile option. The sync can be turned off at the tenant level.
[Turn off OneDrive sync for SharePoint libraries](https://learn.microsoft.com/en-us/sharepoint/sharepoint-sync?wt.mc_id=MVP_308367)

```powershell
Set-SPOTenant -HideSyncButtonOnTeamSite $true
```
The following warning will be displayed on disabling the **sync** option.

**WARNING: Users in your organization will no longer be able to use the "Sync" button to sync TeamSite content. However, existing synced content will remain functional, and users can still sync content via their OneDrive using the Add shortcut command.**

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
            ,NotifyOwnersWhenItemsReshared `
            ,HideSyncButtonOnTeamSite
```

**SPO PowerShell to update OneDrive specific properties**

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
            -OneDriveLoopDefaultSharingLinkScope Organization `
            -OneDriveDefaultLinkToExistingAccess $true `
            -OwnerAnonymousNotification $true `
            -OneDriveStorageQuota 1048576 `
            -OneDriveRequestFilesLinkExpirationInDays 60 `
            -BlockUserInfoVisibilityInOneDrive ApplyToNoUsers `
            -NotificationsInOneDriveForBusinessEnabled $true `
            -NotifyOwnersWhenInvitationsAccepted $true `
            -NotifyOwnersWhenItemsReshared $true `
            -HideSyncButtonOnTeamSite $true

            # -DefaultOneDriveInformationBarrierMode   ` dependent on InformationBarriersSuspension being on
            # -OneDriveBlockGuestsAsSiteAdmin On ` experimental feature
            Set-SPOTenant -OneDriveForGuestsEnabled $false #click yes or no for the setting to update
            Set-SPOTenant -OrphanedPersonalSitesRetentionPeriod 30 #click yes or no for the setting to update 

            Set-SPOTenantSyncClientRestriction -ExcludedFileExtensions "exe;mp3;mp4"

```

**PnP PowerShell**

**Get Tenant OneDrive settings**

```powershell
Get-PnPTenant | select-object -property ODBAccessRequests `
            ,ODBMembersCanShare `
            ,OneDriveRequestFilesLinkEnabled `
            ,ProvisionSharedWithEveryoneFolder `
            ,OneDriveLoopDefaultSharingLinkRole `
            ,OwnerAnonymousNotification `
            ,DisableAddToOneDrive `
            ,OrphanedPersonalSitesRetentionPeriod `
            ,OneDriveStorageQuota `
            ,OneDriveRequestFilesLinkExpirationInDays `
            ,OneDriveForGuestsEnabled `
            ,DefaultOneDriveInformationBarrierMode `
            ,BlockUserInfoVisibilityInOneDrive `
            ,NotificationsInOneDriveForBusinessEnabled `
            ,NotifyOwnersWhenInvitationsAccepted `
            ,NotifyOwnersWhenItemsReshared `
            ,HideSyncButtonOnTeamSite

<#PnP PowerShell can't be used to return the following properties at the time of writing this article
            ,DisableAddShortCutsToOneDrive  ` - use DisableAddToOneDrive
            OneDriveSharingCapability
            ,OneDriveDefaultShareLinkScope `
            ,OneDriveDefaultShareLinkRole `
            ,OneDriveDefaultLinkToExistingAccess `
            ,OneDriveBlockGuestsAsSiteAdmin `
            ,ContentTypeSyncSiteTemplatesList `
#>
```

**Update Tenant OneDrive settings**

```PowerShell
connect-pnponline -url https://contoso-admin.sharepoint.com -interactive
Set-PnPTenant -ODBAccessRequests Off `
            -ODBMembersCanShare On `
            -OneDriveRequestFilesLinkEnabled $true `
            -DisableAddToOneDrive $true  `
            -OneDriveLoopDefaultSharingLinkRole Edit `
            -OneDriveLoopDefaultSharingLinkScope Organization `
            -OwnerAnonymousNotification $true `
            -OrphanedPersonalSitesRetentionPeriod 30 `
            -OneDriveStorageQuota 1048576 `
            -OneDriveRequestFilesLinkExpirationInDays 60 `
            -OneDriveForGuestsEnabled $false `
            -BlockUserInfoVisibilityInOneDrive ApplyToNoUsers `
            -NotificationsInOneDriveForBusinessEnabled $true `
            -NotifyOwnersWhenInvitationsAccepted $true `
            -NotifyOwnersWhenItemsReshared $true `
            -HideSyncButtonOnTeamSite $true
#Set blocked File types
Set-PnPTenantSyncClientRestriction -ExcludedFileExtensions "exe;mp3;mp4"
```

## Site-Level Configuration for OneDrive for Business

For managing OneDrive for Business site collections, administrators can utilize a range of parameters including `ConditionalAccessPolicy`, `DefaultLinkPermission`, `DefaultSharingLinkType`, `DisableCompanyWideSharingLinks`, `LimitedAccessFileType`, `SharingAllowedDomainList`, `SharingBlockedDomainList`, `SharingCapability`, `SharingDomainRestrictionMode`, and `ShowPeoplePickerSuggestionsForGuestUsers`.

For a comprehensive guide on configuring these settings to enhance security and prevent data oversharing, refer to [Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell](https://reshmeeauckloo.com/posts/powershell-sharepoint-sharing-permissions-copilot/).

## Additional Considerations

### Leveraging SharePoint Premium Features

To further enhance your SharePoint environment, consider exploring SharePoint Premium features. Detailed instructions on managing these features via PowerShell can be found at [Manage SharePoint Premium Settings using PowerShell](https://reshmeeauckloo.com/posts/powershell-sharepoint-premium-settings/).

## Conclusion

The strategic management of sharing settings plays a pivotal role in safeguarding data security and ensuring regulatory compliance within SharePoint and OneDrive environments. Leveraging PowerShell for these tasks offers a powerful and efficient means to audit and adjust settings as needed.
