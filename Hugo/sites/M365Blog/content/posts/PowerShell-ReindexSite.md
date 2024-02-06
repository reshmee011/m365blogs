

```PowerShell
#Set Parameters
$AdminCenterURL="https://contoso-admin.sharepoint.com"
#$SiteURL = "https://contoso.sharepoint.com/teams/d-team-site"
Connect-PnPOnline -Url $AdminCenterURL -Interactive

$m365Sites = Get-PnPTenantSite -Detailed | Where-Object {($_.Url -like '*/teams-*' -or $_.Template -eq 'TEAMCHANNEL#1') -and $_.Template -ne 'RedirectSite#0' } 
$m365Sites | ForEach-Object {
$siteUrl = $_.Url;     
Connect-PnPOnline -Url $siteUrl -Interactive
#Get the Web
$Web = Get-PnPWeb
#Request Reindex
Request-PnPReIndexWeb -Web $Web
}
# Export the result array to CSV file
$filesCollection | sort-object "File Url" |Export-CSV $OutPutView -Force -NoTypeInformation
```
