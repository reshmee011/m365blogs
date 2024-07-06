---
title: "Delete Redirect site using PowerSehll"
date: 2024-04-21T17:32:36Z
tags: ["psm1","PowerShell","PnP","PowerShell Script Module"]
featured_image: '/posts/images/PowerShell-editing-psm1-reload/script.png'
omit_header_text: true
draft: true
---


I am unable to delete https://contoso.sharepoint.com/teams/PROJ-hubtest because it is a hub site.
Unregistering the site as hub site failed too. Please help deleting the site.

PS C:\Users\RAuckloo] Remove-PnPTenantSite -Url https://contoso.sharepoint.com/teams/PROJ-hubtest -Force
Remove-PnPTenantSite: Hub site https://contoso.sharepoint.com/teams/PROJ-hubtest cannot be deleted, please unregister it first.

Trying to unregister a redirect site 


Hebatallah Raslan (Convergys International Europe)

Troubleshooting steps:
 
1.	The  site is a redirect site for a Hub site.

2.	Removed it: 
Unregister-SPOHubSite -Identity https://contoso.sharepoint.com/teams/PROJ-hubtest
  
3.	Tried to delete the site and it is deleted using PowerShell.
Remove-SPOSite -Identity https://constoso.sharepoint.com/teams/PROJ-hubtest
4.	Run the below commands: 
Get-sposite -Identity https://contoso.sharepoint.com/teams/PROJ-hubtest | FL
Set-SPOSite -Identity "https://contoso.sharepoint.com/teams/PROJ hubtest" -LockState "Unlock"
Set-SPOUser -Site https://contoso.sharepoint.com/sites/marketing -LoginName melissa.kerr@contoso.com -IsSiteCollectionAdmin $true
Then
Unregister-SPOHubSite -Identity https://contoso.sharepoint.com/teams/PROJ-hubtest
Remove-SPOSite -Identity https://contoso.sharepoint.com/teams/PROJ-hubtest
 
Outcome: Resolved
 
## References

[Unregister-SPOHubSite (Microsoft.Online.SharePoint.PowerShell)](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/unregister-spohubsite?view=sharepoint-ps)

[Manage site redirects - SharePoint in Microsoft 365](https://learn.microsoft.com/en-us/sharepoint/manage-site-redirects)

[Manage Lock Status](https://learn.microsoft.com/en-us/sharepoint/manage-lock-status) 

[set-spouser]https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/set-spouser?view=sharepoint-ps

[Lock and Unlock SharePoint Sites in Microsoft 365 Using PowerShell](https://o365reports.com/2023/10/20/lock-and-unlock-sharepoint-sites-in-microsoft-365-using-powershell/)