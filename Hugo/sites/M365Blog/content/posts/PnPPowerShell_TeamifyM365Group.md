---
title: "Resolving the 'PnP PowerShell Not Digitally Signed' Issue"
date: 2024-06-15T14:49:19+01:00
tags: ["PnP PowerShell", "Digital Signature", "Execution Policy"]
featured_image: '/posts/images/PnPPowerShell-not-digitally-signed/Notdigitallysigned.png'
omit_header_text: true
draft: true
---

$AdminCenterURL="https://contoso-admin.sharepoint.com/"
Connect-PnPOnline -Url $AdminCenterURL -Interactive
 
$siteurl = "https://ppfonline.sharepoint.com/teams/TEAM-PMO/"
$siteinfo = Get-PnPtenantSite -Identity $siteurl -Detailed
 
$GroupId = $SiteInfo.GroupId
 
New-PnPTeamsTeam -GroupId $GroupId
