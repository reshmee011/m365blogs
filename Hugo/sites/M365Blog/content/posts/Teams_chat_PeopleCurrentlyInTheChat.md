---
title: "Teams Chats - default Sharing Links to 'People Currently in this Chat"
date: 2024-07-30T06:51:10Z
tags: ["OneDrive","Teams Chat","Tenant","Sites","PowerShell","OneDrive","Copilot for M365","Information Governance","IG", "PowerShell", "Tackling oversharing"]
featured_image: '/posts/images/Teams_chat_PeopleCurrentlyInTheChat/PeopleInChatCanEdit.png'
omit_header_text: true
draft: false
---

Have you ever wondered how to default the default sharing link in Teams chats to 'People currently in this chat'? By default, if "Anyone" is disabled within the tenant, the sharing link is set to "People in my organization". This can be a challenge when you want to limit access to only those in the current chat. All files uploaded to a Teams Chat are stored in the uploader's OneDrive for Business.

![MS Teams Chat Files](../images/Teams_chat_PeopleCurrentlyInTheChat/OneDriveMSTeamsChatFiles.png)

This is important to prevent "People in my organization" links being created within OneDrive which will help tackling oversharing.

## Setting Default Sharing Links

From the UI, you can set the `File and Folder default link` setting for SharePoint and OneDrive using the `DefaultSharingLinkType` property. However, for more specific control, you should use `OneDriveDefaultShareLinkScope` or `CoreDefaultShareLinkScope`.

* **OneDriveDefaultShareLinkScope**: Sets the default sharing link scope for OneDrive.
* **CoreDefaultShareLinkScope**: Sets the default sharing link scope for SharePoint sites.

![File and Folder link](../images/Teams_chat_PeopleCurrentlyInTheChat/FileAndFolderLinks.png)

Similarly, for setting default permissions, use `CoreDefaultShareLinkRole` or `OneDriveDefaultShareLinkRole` instead of `DefaultLinkPermission`.

* **CoreDefaultShareLinkRole**: Defines the default permission level for sharing links in SharePoint.
* **OneDriveDefaultShareLinkRole**: Defines the default permission level for sharing links in OneDrive.

## PowerShell Script

### PnP PowerShell
```powershell
set-pnptenant -OneDriveDefaultShareLinkScope SpecificPeople -OneDriveDefaultShareLinkRole View 
```

### SPO PowerShell 
```powershell
set-spotenant -OneDriveDefaultShareLinkScope SpecificPeople -OneDriveDefaultShareLinkRole View 
```

## Result

Running the script will configure the default sharing link to be restricted to 'Specific People' with 'View' permissions.

![People in chat can view](../images/Teams_chat_PeopleCurrentlyInTheChat/PeopleInChatCanView.png)


## Additional Considerations

You may also want to customize tenant-level settings for SharePoint and OneDrive using `CoreSharingCapability` and `OneDriveSharingCapability`. The options available are:

* Anyone: Shareable with anyone, including external users.
* Organization: Limits sharing within your organization.
* SpecificPeople: Restricts sharing to specific individuals.
* Uninitialized: Default state before an explicit setting is applied.

By configuring these settings, you can ensure that your Teams chats and file sharing are secure and tailored to your organization's needs.


## References  

[Teams file sharing "People currently in this chat" default option](https://answers.microsoft.com/en-us/msteams/forum/all/teams-file-sharing-people-currently-in-this-chat/3e2c40f1-2e9f-4a05-8ade-c79d31284114). 

[Set-SPOTenant](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/set-spotenant?view=sharepoint-ps)

[Set-PnPTenant](https://pnp.github.io/powershell/cmdlets/Set-PnPTenant.html)