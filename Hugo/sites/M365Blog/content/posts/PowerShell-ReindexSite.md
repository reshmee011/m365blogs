---
title: "Automating Site Reindexing with PowerShell"
date: 2024-02-19T16:16:55Z
tags: ["SharePoint","PnP","PowerShell","Reindex","Sites"]
featured_image: '/posts/images/powerShell-reindexSites/ReindexSiteWarning.png'
draft: false
---

## Automating Site Reindexing with PowerShell

## Introduction

Keeping your SharePoint environment up-to-date is crucial, especially after making schema changes. One important aspect of maintaining accurate and relevant search results is to regularly reindex sites, libraries, or lists. In this blog post, we will explore a streamlined approach using PnP PowerShell to automate the reindexing process. By leveraging this script, you can ensure that your search results reflect the latest changes in your SharePoint environment.

### PnP PowerShell
```PowerShell
#Set Parameters
$AdminCenterURL="https://contoso-admin.sharepoint.com"
Connect-PnPOnline -Url $AdminCenterURL -Interactive

$m365Sites = Get-PnPTenantSite -Detailed | Where-Object {($_.Url -like '*/teams-*' -or $_.Template -eq 'TEAMCHANNEL#1') -and $_.Template -ne 'RedirectSite#0' } #filter to exclude redirect sites and to include team channel sites in the list
$m365Sites | ForEach-Object {
    Connect-PnPOnline -Url $siteUrl -Interactive
    #Get the Web
    $Web = Get-PnPWeb
    #Request Reindex
    Request-PnPReIndexWeb
    Write-host $Web.Url
}
```

### CLI for M365
```PowerShell
#Get Credentials to connect
$m365Status = m365 status
if ($m365Status -match "Logged Out") {
    m365 login
}

#Get SharePoint sites
$m365Sites = m365 spo site list | ConvertFrom-Json | Where-Object {($_.Url -like '*/teams-*' -or $_.Template -eq 'TEAMCHANNEL#1') -and $_.Template -ne 'RedirectSite#0'} #filter to include sites with "/sites/" managed path and to exclude the redirect sites 

$m365Sites | ForEach-Object {
	Write-host "Requesting reindexing for: " $_.Url
	
    #Request Re-indexing of SharePoint site
	m365 spo web reindex --url $_.Url
}

#Disconnect SharePoint online connection
m365 logout
```
## Remarks

From the UI when Reindex is clicked from **Site Settings >Search and Offline Availability** the following warning pops up

![LinkedM365GroupOwners.png](../images/powerShell-reindexSites/ReindexSiteWarning.png)

> [!Note]
> 'Site reindexing may add stress to the search system and take an extended amount of time to
complete. Recrawls of sites with more than one million items can potentially take weeks to process.
Subsequent index updates could be delayed until the reindexing is complete, leading to out-of-date
search results. Make sure to only initiate a site reindex after making changes that require all items to
be reindexed.`

I ran the script over a weekend to minimise impact hence use the script judiciously.

## References 

[Manually request crawling and reindexing of a site, a library or a list](https://learn.microsoft.com/en-us/sharepoint/crawl-site-content?wt.mc_id=MVP_308367)

[Enable content on a site to be searchable](https://learn.microsoft.com/en-us/sharepoint/make-site-content-searchable?wt.mc_id=MVP_308367)
