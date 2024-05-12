---
title: "Empowering Secure Collaboration: Configuring Loop Sharing Tenant and Site Settings with PowerShell"
date: 2024-05-11T06:51:10Z
tags: ["SharePoint","Sharing","Tenant","Sites","PowerShell","Copilot for M365","Governance","Microsoft Loop"]
featured_image: '/posts/images/powershell-sharePoint-sharing-permissions-copilot/TenantSharingOptions_filefolderdomain.png'
draft: true
---

# Empowering Secure Collaboration: Configuring Loop Sharing Tenant and Site Settings with PowerShell

Guest users can be invited to collaborate within Microsoft Loop in the tenant. Refer [How to work with guest users using Microsoft Loop ](https://mymetaverseday.com/2024/05/08/loop-with-guests/?utm_source=collab365&utm_medium=collab365today&utm_campaign=daily_digest) how sharing within Loop works. 

This post focuses on using PowerShell to control the Microsoft Loop external sharing to help securing data especially with Copilot for M365 which can expose data not previously accessible by other means.

## Tenant Level Sharing Settings

Refer to the post which covers some of the SharePoint level settings which applies for Microsoft Loop too for external sharing.
[Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell to prevent oversharing]()

The SharePoint settings that apply to Loop are: SharingCapability, ShowAllUsersClaim, ShowEveryoneClaim, ShowEveryoneExceptExternalUsersClaim,RequireAnonymousLinksExpireInDays, DefaultSharingLinkType, PreventExternalUsersFromResharing, ShowPeoplePickerSuggestionsForGuestUsers, FileAnonymousLinkType, FolderAnonymousLinkType, DefaultLinkPermission, RequireAcceptingAccountMatchInvitedAccount, SharingAllowedDomainList, SharingBlockedDomainList, SharingDomainRestrictionMode, ExternalUserExpirationRequired, ExternalUserExpireInDays,EnableAzureADB2BIntegration

For more info, refer to the two posts

* [Change the organization-level external sharing setting](https://learn.microsoft.com/en-gb/sharepoint/turn-external-sharing-on-or-off#change-the-organization-level-external-sharing-setting?wt.mc_id=MVP_308367)
* [SharePoint and OneDrive integration with Microsoft Entra B2B](https://learn.microsoft.com/en-gb/sharepoint/sharepoint-azureb2b-integration?wt.mc_id=MVP_308367)

Loop components can be used across different M365 services: [Teams](https://mymetaverseday.com/2021/11/10/using-microsoft-loop-components-in-teams/), [Outlook, Whiteboard and Word Online](https://mymetaverseday.com/2023/02/09/loop-whiteboard/). 

Depending on where loop content was originally created it is stored in a different location. Refer to [Loop Storage](https://learn.microsoft.com/en-us/microsoft-365/loop/loop-compliance-summary?view=o365-worldwide#loop-storage) for more info.

[Loop Storage](https://learn.microsoft.com/en-us/microsoft-365/loop/media/loop-files-sharepoint.png?view=o365-worldwide)

### IsLoopEnabled

Enables Loop experiences in M365 Apps supporting Loop.

## IsCollabMeetingNotesFluidEnabled

Enables Loop components everywhere, but Disable integration in Communication app (Outlook, Teams)

Gets or sets a value to specify whether CollabMeetingNotes Fluid Framework is enabled. If IsFluidEnabled disabled, IsCollabMeetingNotesFluidEnabled will be disabled automatically. If IsFluidEnabled enabled, IsCollabMeetingNotesFluidEnabled will be enabled automatically. IsCollabMeetingNotesFluidEnabled can be enabled only when IsFluidEnabled is already enabled.


View [Manage Loop components in OneDrive and SharePoint](https://learn.microsoft.com/en-us/microsoft-365/loop/loop-components-configuration?view=o365-worldwide) for more details.

**To disable Loop run**

```powershell

Set-SPOTenant -IsLoopEnabled $false  `
     -IsCollabMeetingNotesFluidEnabled $false
```

### OneDriveLoopDefaultSharingLinkScope
Gets or sets default share link scope for fluid on OneDrive sites.
Determines the default sharing link for Microsoft Loop within OneDrive.

The valid values are:

* Anyone
* Organization
* SpecificPeople
* Uninitialized

### OneDriveLoopDefaultSharingLinkRole

Determines the default sharing link role for Microsoft Loop within OneDrive. 

The values View or Edit can only be set for the time being.

### CoreLoopDefaultSharingLinkScope

Determines the default sharing link scope for Microsoft Loop and Whiteboard files on SharePoint sites.

Gets or sets default share link scope for fluid on SharePoint sites.

The valid values are:

* Anyone
* Organization
* SpecificPeople
* Uninitialized

### CoreLoopDefaultSharingLinkRole

Determines the default sharing link role for Microsoft Loop and Whiteboard files on SharePoint sites.

The values View or Edit can only be set for the time being.

### AllowAnonymousMeetingParticipantsToAccessWhiteboards

When you share a whiteboard in a Teams meeting, Whiteboard creates a sharing link. This link is accessible by anyone within the organization. The whiteboard is also shared with any in-tenant users in the meeting. Whiteboards are shared using company-shareable links, regardless of the default setting. Support for the default sharing link type is planned.

There's more capability for temporary collaboration by external and shared device accounts during a Teams meeting. Users can temporarily view and collaborate on whiteboards that are shared in a meeting, in a similar way to PowerPoint Live sharing.

In this case, Whiteboard provides temporary viewing and collaboration on the whiteboard during the Teams meeting only. A share link isn't created and Whiteboard doesn't grant access to the file.

If you have external sharing enabled for OneDrive for Business, no further action is required.

If you restrict external sharing for OneDrive for Business, you can keep it restricted, and just enable this new setting in order for external and shared device accounts to work. For more information, see [Manage sharing for Microsoft Whiteboard](https://learn.microsoft.com/microsoft-365/whiteboard/manage-sharing-organizations).


### OneDriveLoopSharingCapability and CoreLoopSharingCapability

Warning thrown while setting OneDriveLoopSharingCapability or CoreLoopSharingCapability Set-PnPTenant: Loop sharing capability settings are disabled. Please use AllowAnonymousMeetingParticipantsToAccessWhiteboards instead.
CoreLoopSharingCapability
Gets or sets collaboration type for fluid on core partition

When sharing a whiteboard in a Teams meeting, Whiteboard creates a sharing link that’s accessible by anyone within the organization and automatically shares the whiteboard with any in-tenant users in the meeting.

There’s an additional capability for temporary collaboration by external and shared device accounts during a meeting. This allows these users to temporarily view and collaborate on whiteboards when they’re shared in a Teams meeting, similar to PowerPoint Live sharing.

If you have the external sharing for OneDrive for Business allowed, no further action is required. If you have external sharing for OneDrive for Business disabled, you can leave it disabled but you must enable this new setting. The setting will not take effect until the SharingCapability 'ExternalUserAndGuestSharing' is also enabled at Tenant level. For more information, see [Enable Microsoft Whiteboard for your organization](https://support.microsoft.com/office/enable-microsoft-whiteboard-for-your-organization-1caaa2e2-5c18-4bdf-b878-2d98f1da4b24).

The valid values are:

- Disabled
- ExternalUserSharingOnly
- ExternalUserAndGuestSharing
- ExistingExternalUserSharingOnly


### Sample script to amend tenant level sharing settings for Microsoft Loop configuration

**SPO PowerShell**

#### To get Microsoft Loop settings

```PowerShell
 Get-SPOTenant | select-object -Property IsLoopEnabled `
        ,OneDriveLoopDefaultSharingLinkScope `
        ,OneDriveLoopSharingCapability `
        ,OneDriveLoopDefaultSharingLinkRole `
        ,CoreLoopSharingCapability `
        ,CoreLoopDefaultSharingLinkScope `
        ,CoreLoopDefaultSharingLinkRole `
        ,IsCollabMeetingNotesFluidEnabled `
        ,AllowAnonymousMeetingParticipantsToAccessWhiteboards
```

#### To set the Microsoft Loop settings

```PowerShell
connect-SPOService -Url https://contoso-admin.sharepoint.com
## Microsoft Loop specific settings
    Set-SPOTenant -IsLoopEnabled $true `
            -OneDriveLoopDefaultSharingLinkScope Organization `
            -OneDriveLoopDefaultSharingLinkRole Edit `
            -CoreLoopDefaultSharingLinkScope Organization `
            -CoreLoopDefaultSharingLinkRole Edit `
            -IsCollabMeetingNotesFluidEnabled $false `
            -AllowAnonymousMeetingParticipantsToAccessWhiteboards On `
        

        <# These two settings can't be set yet using SPO PowerShell, use PnP PowerShell instead 
         #-OneDriveLoopSharingCapability ` #Determines the level of sharing of Microsoft Loop within OneDrive
         #-CoreLoopSharingCapability #Determines the level of sharing for Microsoft Loop within SharePoint sites excluding OneDrive
        #>
```

**PnP PowerShell**

#### To get Microsoft Loop settings

```PowerShell
 Get-PnPTenant | select-object -Property IsLoopEnabled `
        ,OneDriveLoopDefaultSharingLinkScope `
        ,OneDriveLoopSharingCapability `
        ,OneDriveLoopDefaultSharingLinkRole `
        ,CoreLoopSharingCapability `
        ,CoreLoopDefaultSharingLinkScope `
        ,CoreLoopDefaultSharingLinkRole `
        ,IsCollabMeetingNotesFluidEnabled `
        ,AllowAnonymousMeetingParticipantsToAccessWhiteboards
```

#### To set the Microsoft Loop settings

```PowerShell
    connect-PnPOnline -Url https://contoso-admin.sharepoint.com -interactive
    ## Microsoft Loop specific settings
    Set-PnPTenant -IsLoopEnabled $true `
            -OneDriveLoopDefaultSharingLinkScope Organization `
            -OneDriveLoopDefaultSharingLinkRole Edit `
            -CoreLoopDefaultSharingLinkScope Organization `
            -CoreLoopDefaultSharingLinkRole Edit `
            -IsCollabMeetingNotesFluidEnabled $false `
            -AllowAnonymousMeetingParticipantsToAccessWhiteboards On

<#
            -OneDriveLoopSharingCapability ExternalUserSharingOnly `
            -CoreLoopSharingCapability ExternalUserSharingOnly `

Warning thrown while setting OneDriveLoopSharingCapability or CoreLoopSharingCapability Set-PnPTenant: Loop sharing capability settings are disabled. Please use AllowAnonymousMeetingParticipantsToAccessWhiteboards instead.
#>

```
 
## Microsoft Loop site level sharing settings

Sharing settings for Microsoft Loop can be set at the site level to provide more granular control.

### LoopDefaultSharingLinkRole

### DefaultSharingLinkType

The default link type for the site collection

Possible values are 

None - Respect the organization default sharing link type
AnonymousAccess - Sets the default sharing link for this site to an Anonymous Access or Anyone link
Internal - Sets the default sharing link for this site to the "organization" link or company shareable link
Direct - Sets the default sharing link for this site to the "Specific people" link

### PowerShell script to update site sharing settings for Microsoft Loop

```PowerShell

Connect-SPOService -Url https://contoso-admin.sharepoint.com

Set-SPOSite -Identity https://contoso.sharepoint.com/sites/SharingTest `
            -LoopDefaultSharingLinkRole $false `
            -DefaultSharingLinkType ExistingExternalUserSharingOnly 
            
```

**PnP PowerShell**

```
connect-pnponline -url https://contoso.sharepoint.com/sites/SharingTest  -interactive

Set-PnPTenantSite -Identity https://contoso.sharepoint.com/sites/SharingTest `
            -LoopDefaultSharingLinkRole $false `
            -DefaultSharingLinkType ExistingExternalUserSharingOnly 
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
