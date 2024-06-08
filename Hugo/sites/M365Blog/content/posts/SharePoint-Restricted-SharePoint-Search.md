---
title: "Sharepoint Restricted Search"
date: 2024-05-28T09:55:41+01:00
tags: ["SharePoint Restricted Search","Copilot for M365","Governance","PowerShell","PnP PowerShell"]
featured_image: '/posts/images/SharePoint-Restricted-SharePoint-Search/rss_enabled.png'
draft: false
---

### Remove certain SharePoint sites from search

Excluding certain SharePoint sites from search would mean the contents from the excluded sites won't be available to M365 search and Copilot and should be used as a last resort. 

Until the right controls are in place, sensitive sites may be excluded using [PowerShell Scripts for Restricted SharePoint Search](https://learn.microsoft.com/en-us/sharepoint/restricted-sharepoint-search-admin-scripts?wt.mc_id=MVP_308367).

1. Enable Restricted Search Mode

```PowerShell
Set-SPOTenantRestrictedSearchMode -Mode Enabled 
```

2. Add the sites to the allowed list via csv file or in list string

Sample csv to use with no headers and only one column for the site url 

```csv
https://reshmeeauckloo.sharepoint.com/sites/Company311
https://reshmeeauckloo.sharepoint.com/sites/contosoportal
```

```Powershell
Add-SPOTenantRestrictedSearchAllowedList  -SitesListFileUrl C:\Users\admin\Downloads\UrlList.csv
```

```Powershell
Add-SPOTenantRestrictedSearchAllowedList -SitesList  @("https://reshmeeauckloo.sharepoint.com/sites/Company311","https://reshmeeauckloo.sharepoint.com/sites/contosoportal") 
```

3. Retrieves existing list of URLs in the allowed list

```powershell
 Get-SPOTenantRestrictedSearchAllowedList
```

4. Removes sites from the allow list

```powershell
Remove-SPOTenantRestrictedSearchAllowedList -SitesList <List[string]> [<CommonParameters>]
```

## Potential issues

If you don't have license 

![No License](../images/SharePoint-Restricted-SharePoint-Search/rss-licenserequired.png)


If the Url is specified as [https://reshmeeauckloo.sharepoint.com/sites/Company311](https://reshmeeauckloo.sharepoint.com/sites/Company311), keep it simple with just the URL

Add-PnPTenantRestrictedSearchAllowedList: Could not find site with URL [https://reshmeeauckloo.sharepoint.com/sites/Company311](https://reshmeeauckloo.sharepoint.com/sites/Company311). Verify if the site exists and URL is valid.