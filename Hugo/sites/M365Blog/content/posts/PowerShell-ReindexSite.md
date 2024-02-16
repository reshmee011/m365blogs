---
title: "Pnp Powershell Reindex all sites"
date: 2024-02-16T16:16:55Z
tags: ["SharePoint","PnP","PowerShell","CLI for M365" ,"Audit Logs","DLP", "Exchange", "AzureDirectory"]
featured_image: '/posts/images/PowerShell_PnPUnifiedLog/Sample.png'
draft: true
---

After applying auto sensitivity labelling policies to all SPO sites, all sites needed to be reindex to indentify which documents had a particular property so that policies are aware of the policies. 
 

```PowerShell
#Set Parameters
$AdminCenterURL="https://contoso-admin.sharepoint.com"
#$SiteURL = "https://contoso.sharepoint.com/teams/d-team-site"
Connect-PnPOnline -Url $AdminCenterURL -Interactive

$m365Sites = Get-PnPTenantSite -Detailed | Where-Object {($_.Url -like '*/teams-*' -or $_.Template -eq 'TEAMCHANNEL#1') -and $_.Template -ne 'RedirectSite#0' } 
$m365Sites | ForEach-Object {
    Connect-PnPOnline -Url $siteUrl -Interactive
    #Get the Web
    $Web = Get-PnPWeb
    #Request Reindex
    Request-PnPReIndexWeb
    Write-host $Web.Url
}
# Export the result array to CSV file
$filesCollection | sort-object "File Url" |Export-CSV $OutPutView -Force -NoTypeInformation
```

Run simulation against auto-labelling policies. 
