---
title: "Empowering Secure Collaboration: Configuring Microsoft Loop Sharing Tenant and Site Settings with PowerShell"
date: 2024-05-13T06:51:10Z
tags: ["SharePoint","Sharing","Tenant","Sites","PowerShell","Copilot for M365","Governance","Microsoft Loop"]
featured_image: '/posts/images/powershell-loop-Sharing-settings-Copilot/loopsharing.png'
draft: false
---

# Empowering Secure Collaboration: Configuring Microsoft Loop Sharing SharePoint Tenant and Site Settings with PowerShell

Guest users can be invited to collaborate within Microsoft Loop in the tenant. Refer [How to work with guest users using Microsoft Loop ](https://mymetaverseday.com/2024/05/08/loop-with-guests/?utm_source=collab365&utm_medium=collab365today&utm_campaign=daily_digest) how sharing within Loop works.

This post focuses on using PowerShell to control the Microsoft Loop sharing settings to help securing data especially with Copilot for M365 which can expose data not previously accessible by other means.

## Tenant Level Microsoft Loop Sharing Settings

Refer to the post which covers some of the SharePoint level settings which applies for Microsoft Loop too for external sharing.
[Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell to prevent oversharing](https://reshmeeauckloo.com/posts/powershell-sharepoint-sharing-permissions-copilot/)

The SharePoint settings that apply to Loop are: SharingCapability, ShowAllUsersClaim, ShowEveryoneClaim, ShowEveryoneExceptExternalUsersClaim,RequireAnonymousLinksExpireInDays, DefaultSharingLinkType, PreventExternalUsersFromResharing, ShowPeoplePickerSuggestionsForGuestUsers, FileAnonymousLinkType, FolderAnonymousLinkType, DefaultLinkPermission, RequireAcceptingAccountMatchInvitedAccount, SharingAllowedDomainList, SharingBlockedDomainList, SharingDomainRestrictionMode, ExternalUserExpirationRequired, ExternalUserExpireInDays,EnableAzureADB2BIntegration

For more info, refer to the two posts

* [Change the organization-level external sharing setting](https://learn.microsoft.com/en-gb/sharepoint/turn-external-sharing-on-or-off#change-the-organization-level-external-sharing-setting?wt.mc_id=MVP_308367)
* [SharePoint and OneDrive integration with Microsoft Entra B2B](https://learn.microsoft.com/en-gb/sharepoint/sharepoint-azureb2b-integration?wt.mc_id=MVP_308367)

Loop components can be used across different M365 services: [Teams](https://mymetaverseday.com/2021/11/10/using-microsoft-loop-components-in-teams/), [Outlook, Whiteboard and Word Online](https://mymetaverseday.com/2023/02/09/loop-whiteboard/). 

Depending on where loop content was originally created it is stored in a different location. Refer to [Loop Storage](https://learn.microsoft.com/en-us/microsoft-365/loop/loop-compliance-summary?view=o365-worldwide#loop-storage&wt.mc_id=MVP_308367) for more info.

![Loop Storage](https://learn.microsoft.com/en-us/microsoft-365/loop/media/loop-files-sharepoint.png)

* Created in the Loop app ➡️️ SharePoint Embedded
* Created outside the Loop app in places that have dedicated shared storage (e.g. Teams channels) ➡️️ SharePoint
* Created outside the Loop app in all other places that don't have tightly associated collaborative storage (e.g. Teams chat, Outlook email, Word for the web, Whiteboard) ➡️️ OneDrive

### IsLoopEnabled

Enables Loop experiences in M365 Apps supporting Loop.

## IsCollabMeetingNotesFluidEnabled

 Enable/disable integration in Communication app (Outlook, Teams) to enable view and create Loop files in Outlooks

View [Manage Loop components in OneDrive and SharePoint](https://learn.microsoft.com/en-us/microsoft-365/loop/loop-components-configuration?view=o365-worldwide&wt.mc_id=MVP_308367) for more details.

**To disable Loop everywhere**

```powershell

Set-SPOTenant -IsLoopEnabled $false  `
     -IsCollabMeetingNotesFluidEnabled $false
```

### OneDriveLoopDefaultSharingLinkScope

Gets or sets default share link scope for Microsoft Loop on OneDrive sites.

The valid values are:

* Anyone
* Organization
* SpecificPeople
* Uninitialized

### OneDriveLoopDefaultSharingLinkRole

Gets or sets the default sharing link role for Microsoft Loop within OneDrive sites

The values View or Edit can only be set for the time being.

### CoreLoopDefaultSharingLinkScope

Gets or sets the default sharing link scope for Microsoft Loop and Whiteboard files on SharePoint sites.

The valid values are:

* Anyone
* Organization
* SpecificPeople
* Uninitialized

### CoreLoopDefaultSharingLinkRole

Gets or sets the default sharing link role for Microsoft Loop and Whiteboard files on SharePoint sites.

The values View or Edit can only be set for the time being.

### AllowAnonymousMeetingParticipantsToAccessWhiteboards

When a whiteboard is shared in a Teams meeting, Whiteboard creates a sharing link. This link is accessible by anyone within the organization. Whiteboards are shared using company-shareable links, regardless of the default setting.

There's more capability for temporary collaboration by external and shared device accounts during a Teams meeting. Users can temporarily view and collaborate on whiteboards that are shared in a meeting, in a similar way to PowerPoint Live sharing.In this case, Whiteboard provides temporary viewing and collaboration on the whiteboard during the Teams meeting only. A share link isn't created and Whiteboard doesn't grant access to the file.

If external sharing is enabled for OneDrive for Business, no further action is required.

If external sharing is restricted for OneDrive for Business, it can be kept restricted, and enable this  setting in order for external and shared device accounts to work. For more information, see [Manage sharing for Microsoft Whiteboard](https://learn.microsoft.com/microsoft-365/whiteboard/manage-sharing-organizations?wt.mc_id=MVP_308367).

This setting applies only to whiteboards and replaces the previously shared settings: **OneDriveLoopSharingCapability** and **CoreLoopSharingCapability**. Those settings are no longer applicable and can be disregarded.

### OneDriveLoopSharingCapability and CoreLoopSharingCapability

Warning thrown while setting OneDriveLoopSharingCapability or CoreLoopSharingCapability. Use **AllowAnonymousMeetingParticipantsToAccessWhiteboards** instead as **OneDriveLoopSharingCapability** or **CoreLoopSharingCapability** are redundant.

```powershell
Set-PnPTenant: Loop sharing capability settings are disabled. Please use AllowAnonymousMeetingParticipantsToAccessWhiteboards instead.
```

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
```

**PnP PowerShell**

#### To get Microsoft Loop settings

```PowerShell
#currently unable to retrieve below properties using PnP PowerShell, I have created PR https://github.com/pnp/powershell/pull/3948 to enable retrieval of these values
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
```
 
## Microsoft Loop site level sharing settings

Sharing settings for Microsoft Loop can be set at the site level to provide more granular control.

### LoopDefaultSharingLinkScope

Gets or sets default share link scope for Microsoft Loop on SharePoint or OneDrive site.

The valid values are:

* Anyone
* Organization
* SpecificPeople
* Uninitialized

### LoopDefaultSharingLinkRole

Gets or sets the default sharing link role for Microsoft Loop within OneDrive sites

The values View or Edit can only be set for the time being.

### PowerShell script to update  Microsoft Loop site sharing settings

```PowerShell

Connect-SPOService -Url https://contoso-admin.sharepoint.com

Set-SPOSite -Identity https://contoso.sharepoint.com/sites/SharingTest `
            -LoopDefaultSharingLinkScope Organization `
            -LoopDefaultSharingLinkRole Edit 

Get-SPOSite -Identity https://contoso.sharepoint.com/sites/SharingTest | select-object -Property `
            LoopDefaultSharingLinkScope `
            LoopDefaultSharingLinkRole             
```

**PnP PowerShell**

```
connect-pnponline -url https://contoso.sharepoint.com/sites/SharingTest  -interactive

Set-PnPSite -Identity https://contoso.sharepoint.com/sites/SharingTest `
            -LoopDefaultSharingLinkScope Organization `
            -LoopDefaultSharingLinkRole Edit 

Get-PnPSite -Identity https://contoso.sharepoint.com/sites/SharingTest | select-object -Property `
            LoopDefaultSharingLinkScope `
            LoopDefaultSharingLinkRole 
```

## Other settings to consider 

1. Sensitivity Labels
[How to protect sensitive information in SharePoint Online using Purview Sensitivity Labels](https://www.sharepointeurope.com/how-to-protect-sensitive-information-in-sharepoint-online-using-purview-sensitivity-labels/?utm_source=collab365&utm_medium=collab365today&utm_campaign=daily_digest)

2. Retention Policies
3. Data Loss Prevention

View [Empowering Secure Collaboration: Configuring SharePoint Sharing Tenant and Site Settings with PowerShell to prevent oversharing](https://reshmeeauckloo.com/posts/powershell-sharepoint-sharing-permissions-copilot/) for more info 

## Conclusion

Effective management of sharing settings is crucial for maintaining data security across M365 apps including Microsoft Loop. PowerShell offers unparalleled flexibility and control in configuring these settings, ensuring that collaboration remains both seamless and secure.

## References

[Microsoft Copilot for Microsoft 365 - best practices with SharePoint](https://learn.microsoft.com/en-us/SharePoint/sharepoint-copilot-best-practices?wt.mc_id=MVP_308367)

[Manage sharing for Microsoft Whiteboard](https://learn.microsoft.com/microsoft-365/whiteboard/manage-sharing-organizations?wt.mc_id=MVP_308367)

[How to work with guest users using Microsoft Loop ](https://mymetaverseday.com/2024/05/08/loop-with-guests/?utm_source=collab365&utm_medium=collab365today&utm_campaign=daily_digest)

[Change the organization-level external sharing setting](https://learn.microsoft.com/en-gb/sharepoint/turn-external-sharing-on-or-off#change-the-organization-level-external-sharing-setting?wt.mc_id=MVP_308367)

[SharePoint and OneDrive integration with Microsoft Entra B2B](https://learn.microsoft.com/en-gb/sharepoint/sharepoint-azureb2b-integration?wt.mc_id=MVP_308367)

[Loop in Teams](https://mymetaverseday.com/2021/11/10/using-microsoft-loop-components-in-teams/)

[Outlook, Whiteboard and Word Online](https://mymetaverseday.com/2023/02/09/loop-whiteboard/)

[Loop Storage](https://learn.microsoft.com/en-us/microsoft-365/loop/loop-compliance-summary?view=o365-worldwide#loop-storage&wt.mc_id=MVP_308367)
