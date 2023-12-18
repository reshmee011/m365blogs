$AdminCenterURL="https://contoso-admin.sharepoint.com/"
Connect-PnPOnline -Url $AdminCenterURL -Interactive
 
$siteurl = "https://ppfonline.sharepoint.com/teams/TEAM-PMO/"
$siteinfo = Get-PnPtenantSite -Identity $siteurl -Detailed
 
$GroupId = $SiteInfo.GroupId
 
New-PnPTeamsTeam -GroupId $GroupId
