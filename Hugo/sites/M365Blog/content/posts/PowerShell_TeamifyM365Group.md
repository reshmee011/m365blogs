---
title: "Teamifying an existing m365 group"
date: 2024-07-09T14:49:19+01:00
tags: ["PowerShell", "M365", "PnP","Teamify"]
featured_image: '/posts/images/PnPPowerShell-not-digitally-signed/Notdigitallysigned.png'
omit_header_text: true
draft: true
---

```PowerShell
$AdminCenterURL="https://contoso-admin.sharepoint.com/"
Connect-PnPOnline -Url $AdminCenterURL -Interactive
 
$siteurl = "https://ppfonline.sharepoint.com/teams/TEAM-PMO/"
$siteinfo = Get-PnPtenantSite -Identity $siteurl -Detailed
 
$GroupId = $SiteInfo.GroupId
 
New-PnPTeamsTeam -GroupId $GroupId
```
