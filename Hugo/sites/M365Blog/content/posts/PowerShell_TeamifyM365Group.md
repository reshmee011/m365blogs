---
title: "Teamifying an Existing M365 Group with PowerShell"
date: 2024-07-09T14:49:19+01:00
tags: ["PowerShell", "M365 Group", "PnP PowerShell", "Teams"]
omit_header_text: true
draft: false
---

# Teamifying an Existing M365 Group with PowerShell

Within M365 , SharePoint and Teams together provides a rich collaboaration platform. When a team site is created from SharePoint admin centre, it is not associated with a `Teams` despite a M365 group is created in the background. To extend SharePoint collaboration features , there is a need to "teamify" an existing Microsoft 365 Group, essentially attaching a new Teams instance to it to allow use of channels and other apps. This post covers how the an existing M365 group can be teamified. 

> This action can't be undone.

## Prerequisites

- **PnP PowerShell Module**: Ensure you have the latest version of the PnP PowerShell module installed. This module provides a rich set of cmdlets that simplify administrative tasks in SharePoint and Microsoft 365.
- **Administrator Credentials**: You must have administrative privileges to the SharePoint admin center and the Microsoft 365 Group you intend to teamify.

## PowerShell Script

```PowerShell
$AdminCenterURL="https://contoso-admin.sharepoint.com/"
Connect-PnPOnline -Url $AdminCenterURL -Interactive
 
$siteurl = "https://ppfonline.sharepoint.com/teams/TEAM-PMO/"
$siteinfo = Get-PnPtenantSite -Identity $siteurl -Detailed
 
$GroupId = $SiteInfo.GroupId
 
New-PnPTeamsTeam -GroupId $GroupId
```

## Conclusion

The script can automate the steps to teamify an existing M365 group saving time.
