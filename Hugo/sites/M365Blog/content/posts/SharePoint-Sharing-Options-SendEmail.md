---
title: "Sharing Options Updates - Send Email Button Greyed Out"
date: 2024-04-29T06:19:42Z
tags: ["SharePoint","Owners","M365", "PowerShell","Send Email", "Governance"]
featured_image: '/posts/images/SharePoint-Sharing-Options-SendEmail/SendLinkInOutlook.png'
omit_header_text: true
draft: False
---

# Sharing Options Updates - Send Email button greyed out

Starting from March 2024, there has been a notable change in SharePoint’s sharing options. If a user is not allowed to share on a SharePoint site (for instance, when a user is a member and only owners have sharing privileges), the Send Email button appears greyed out. Additionally, a warning message is displayed: **Sharing is limited on this item. You can only copy links for people who have existing access, and you can’t invite anyone new.** This occurs even when sharing with a user who already has existing access. 

The change of behaviour could be related to MC706173, please refer for more info[M365 Changelog: (Updated) Invite people you choose in the Share control in Microsoft 365 apps](https://petri.com/microsoft-changelog/m365-changelog-invite-people-you-choose-in-the-share-control-in-microsoft-365-apps/) though the full changes are not clear.

![Send email greyed out](../images/SharePoint-Sharing-Options-SendEmail/SendEmailGreyed.png) 

However, this issue does not affect owners who are permitted to share content.

![Sharing restricted to owners](../images/SharePoint-Sharing-Options-SendEmail/SiteSharingSettings_Owners.png) 

Interestingly, there is no official documentation from Microsoft regarding these sharing enhancements. Previously, the **Send Email** button was always enabled. Despite users being unable to share files, folders, or items, they could still select recipients who lacked **existing access.** This led to confusion when recipients received notifications about links they couldn’t access. To address this scenario, the **send email** option is now enabled only for those who can share files and folders. For users who cannot share, an alternative is to click the ellipsis at the top of the pop-up and select !Send Email to Outlook.

![Send Email to Outlook](../images/SharePoint-Sharing-Options-SendEmail/SendLinkInOutlook.png) 

Clicking **Send Email to Outlook** will open Outlook in OWA, allowing users to send the link to others.

![Email in OWA](../images/SharePoint-Sharing-Options-SendEmail/EmailinOWA.png) 

## References 

[sharing-is-limited-on-this-item](https://techcommunity.microsoft.com/t5/sharepoint/sharing-is-limited-on-this-item/m-p/4084439)

[Sharing files, folders, and list items](https://support.microsoft.com/en-gb/office/sharing-files-folders-and-list-items-74cab0bf-39c6-4112-a63f-88ee121722d0?wt.mc_id=MVP_308367)

[M365 Changelog: (Updated) Invite people you choose in the Share control in Microsoft 365 apps](https://petri.com/microsoft-changelog/m365-changelog-invite-people-you-choose-in-the-share-control-in-microsoft-365-apps/)